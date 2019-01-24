---
title: マルチテナント アプリケーションで要求ベースの ID を操作する
description: 発行者の検証と承認に要求を使用する方法。
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 8b8cbd2b857493d94103e80f53f187207feaa11e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486507"
---
# <a name="work-with-claims-based-identities"></a>要求ベースの ID を操作する

[![GitHub](../_images/github.png) サンプル コード][sample application]

## <a name="claims-in-azure-ad"></a>Azure AD の要求

ユーザーがサインインすると、Azure AD は、ユーザーに関する要求セットを含む ID トークンを送信します。 要求は、キーと値のペアで表される 1 つの情報です。 たとえば、`email`=`bob@contoso.com` です。  要求には、ユーザーを認証し、要求を作成するエンティティである発行者があります (ここでは Azure AD)。 発行者は、ユーザーを認証し、要求を作成するエンティティです。 発行者を信頼していれば、要求も信頼することになります(逆に、信頼できない発行者の場合は、要求も信頼しないでください)。

概要:

1. ユーザーを認証します。
2. IDP から 1 セットの要求が送信されます。
3. アプリで、要求の正規化または強化が行われます (省略可能)。
4. アプリで、要求を使用して承認が決定されます。

OpenID Connect では、取得する要求のセットは、認証要求の[スコープ パラメーター]によって制御されます。 ただし、Azure AD が OpenID Connect を通して発行する要求のセットは限られています。[サポートされているトークンとクレームの種類]に関する記事を参照してください。 ユーザーの詳細情報を取得するには、Azure AD Graph API を使用する必要があります。

次に、通常はアプリが処理するような AAD の要求を示します。

| ID トークンの要求の種類 | 説明 |
| --- | --- |
| aud |トークンがだれに対して発行されたか。 これは、アプリケーションのクライアント ID になります。 一般に、ミドルウェアが自動的に検証するため、この要求について気にする必要はありません。 例: `"91464657-d17a-4327-91f3-2ed99386406f"` |
| groups |ユーザーがメンバーである AAD グループの一覧。 例: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |OIDC トークンの [発行者] 。 例: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| name |ユーザーの表示名。 例: `"Alice A."` |
| oid |AAD でのユーザーのオブジェクト識別子。 この値は、ユーザーの変更不能な再利用できない識別子です。 ユーザーの一意識別子として、電子メールではなく、この値を使用します。電子メール アドレスは変更される可能性があります。 アプリで Azure AD Graph API を使用する場合は、オブジェクト ID がプロファイル情報のクエリを実行するために使用される値になります。 例: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| roles |ユーザーのアプリ ロールの一覧。    例: `["SurveyCreator"]` |
| tid |テナント ID。 この値は、Azure AD での テナントの一意識別子です。 例: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |人間が読み取り可能なユーザーの表示名。 例: `"alice@contoso.com"` |
| upn |ユーザー プリンシパル名。 例: `"alice@contoso.com"` |

この表は、ID トークンに表示される要求の種類の一覧です。 ASP.NET Coreでは、OpenID Connect ミドルウェアがユーザー プリンシパルの Claims コレクションを設定するときに、一部の要求の種類を変換します。

* oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`
* tid > `http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>要求の変換

認証フロー中に、IDP から取得する要求を変更できます。 ASP.NET Core では、OpenID Connect ミドルウェアからの **AuthenticationValidated** イベントの内部で要求変換を実行できます  (「[認証イベント]」を参照してください)。

**AuthenticationValidated** 中に追加された要求は、セッション認証 Cookie に保存されます。 Azure AD にプッシュバックされることはありません。

次に、要求変換の例をいくつか示します。

* **要求の正規化**またはユーザー間で一貫性のある要求にする。 これが特に関係するのは、複数の IDP から要求を取得している場合であり、類似する情報に対して異なる要求の種類が使用されている可能性があります。 たとえば、Azure AD は、ユーザーの電子メール アドレスを含む "upn" 要求を送信します。 その他の IDP は、"email" 要求を送信することがあります。 次のコードは、"upn" 要求を "email" 要求に変換します。

  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```

* 存在しない要求のための**既定の要求値**を追加します。たとえば、ユーザーに既定のロールを割り当てます。 これにより、承認ロジックを簡略化できる場合があります。
* ユーザーのアプリケーション固有の情報を使用して **カスタム要求の種類** を追加します。 たとえば、データベース内のユーザーに関する一部の情報を保存することができます。 この情報を含むカスタム要求を認証チケットに追加できます。 要求は Cookie に保存されるので、ログイン セッションごとに 1 度のみ、データベースから取得する必要があります。 一方で、過度に大きな Cookie を作成することを回避するには、Cookie のサイズとデータベース検索の妥協点を考慮する必要があります。

認証フローが完了したら、`HttpContext.User` で要求を使用できます。 その時点で、要求は、読み取り専用コレクションとして処理する必要があります。たとえば、承認に関する決定を行うために使用します。

## <a name="issuer-validation"></a>発行者の検証

OpenID Connect の発行者要求 ("iss") によって、ID トークンを発行した IDP が識別されます。 OIDC 認証フローの一部では、発行者要求が実際の発行者と一致することが検証されます。 この処理は、OIDC ミドルウェアが自動実行します。

Azure AD で発行者値は AD テナントごとに一意です (`https://sts.windows.net/<tenantID>`)。 そのため、発行者がアプリにサインインできるテナントであることを確認するには、アプリケーションで追加チェックを実行する必要があります。

シングルテナント アプリケーションの場合、発行者が自分のテナントであることを確認するだけです。 実際の処理では、既定で OIDC ミドルウェアが自動的に実行します。 マルチテナント アプリの場合、異なるテナントに対応する複数の発行者を許容する必要があります。 次に、利用できる一般的なアプローチを示します。

* OIDC ミドルウェア オプションで、 **ValidateIssuer** を false に設定します。 これで自動チェックが無効になります。
* テナントがサインアップしたら、テナント発行者をユーザー DB に保存します。
* ユーザーがサインインするたびに、データベース内の発行者を検索します。 発行元が見つからない場合、そのテナントがサインアップしていないことを意味します。 このような場合、サインアップ ページにリダイレクトすることができます。
* また、たとえばサブスクリプション料金を支払わない顧客など、特定のテナントをブラックリストに登録することもできます。

詳細については、[マルチテナント アプリケーションでのサインアップとテナントのオンボード][signup]に関するページを参照してください。

## <a name="using-claims-for-authorization"></a>承認の要求を使用する

要求では、ユーザーの ID は、一枚岩のエンティティではなくなりました。 たとえば、ユーザーは、電子メール アドレス、電話番号、誕生日、性別などを持つ場合があります。ユーザーの IDP には、これらの情報がすべて格納されている可能性があります。 ただし、ユーザーを認証するときは、通常はこれらの情報のサブセットを要求として取得します。 このモデルでは、ユーザーの ID は、単なる要求の束です。 ユーザーに関する承認の決定を行うときは、要求の特定のセットを探します。 つまり、「ユーザー X はアクション Y を実行できるか」という質問は、最終的には「ユーザー X は要求 Z を所有しているか」ということになります。

要求を要求する基本的なパターンを次に示します。

* 特定の値が設定された特定の要求を持つユーザーを確認する例を次に示します。

   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```

   このコードでは、ユーザーが "Admin" 値の Role 要求を持っているかどうかを確認します。 ユーザーが Role 要求を持っていない場合、または複数の Role 要求を持っている場合も正しく処理されます。
  
   **ClaimTypes** クラスでは、一般的に使用される要求の種類に対して定数を定義します。 ただし、要求の種類には任意の文字列値を使用できます。
* 最大で 1 つの値があると想定される要求の種類の場合、1 つの値を取得する例を次に示します。

  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```

* 要求の種類のすべての値を取得する例を次に示します。

  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

詳細については、[マルチテナント アプリケーションでのロールベースおよびリソースベースの承認][authorization]に関する記事を参照してください。

[**次へ**][signup]

<!-- links -->

[スコープ パラメーター]: https://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[サポートされているトークンとクレームの種類]: /azure/active-directory/active-directory-token-and-claims/
[発行者]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[認証イベント]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
