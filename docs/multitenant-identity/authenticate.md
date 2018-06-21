---
title: マルチテナント アプリケーションでの認証
description: マルチテナント アプリケーションで Azure AD のユーザーを認証する方法
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: e85817626675cec4d126921c19a31a0983ecd62d
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2017
ms.locfileid: "26359257"
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a>Azure AD および OpenID Connect を使用して認証する

[![GitHub](../_images/github.png) サンプル コード][sample application]

Surveys アプリケーションでは、OpenID Connect (OIDC) プロトコルを使用して、Azure Active Directory (Azure AD) でユーザーを認証します。 Surveys アプリケーションでは、OIDC のビルトイン ミドルウェアがある ASP.NET Core を使用します。 次の図では、ユーザーがサインインしたときに行われる処理の概要を示します。

![Authentication flow](./images/auth-flow.png)

1. ユーザーがアプリの "サインイン" ボタンをクリックします。 このアクションは、MVC コントローラーによって処理されます。
2. MVC コントローラーが **ChallengeResult** アクションを返します。
3. ミドルウェアは **ChallengeResult** をインターセプトし、Azure AD サインイン ページにユーザーをリダイレクトする 302 応答を作成します。
4. ユーザーが Azure AD で認証を行います。
5. Azure AD が ID トークンをアプリケーションに送信します。
6. ミドルウェアが ID トークンを検証します。 この時点で、ユーザーはアプリケーションの内部で認証されます。
7. ミドルウェアがユーザーをアプリケーションにリダイレクトします。

## <a name="register-the-app-with-azure-ad"></a>アプリの Azure AD への登録
OpenID Connect を有効にするために、SaaS プロバイダーは、アプリケーションをその Azure AD テナント内で登録します。

アプリケーションを登録するには、「[Azure Active Directory とアプリケーションの統合](/azure/active-directory/active-directory-integrating-applications/)」の「[アプリケーションの追加](/azure/active-directory/active-directory-integrating-applications/#adding-an-application)」の手順に従います。

Surveys アプリケーションに固有の手順については、「[Run the Surveys application (Surveys アプリケーションの実行)](./run-the-app.md)」をご覧ください。 以下の点に注意してください。

- マルチ テナント アプリケーションの場合は、マルチ テナント オプションを明示的に構成する必要があります。 これにより、他の組織がアプリケーションにアクセスできるようになります。

- 応答 URL は、Azure AD が OAuth 2.0 応答を送信する URL です。 ASP.NET Core を使用する場合、認証ミドルウェアで構成するパスとこの URL が一致する必要があります (次のセクションをご覧ください)。 

## <a name="configure-the-auth-middleware"></a>認証ミドルウェアの構成
このセクションでは、OpenID Connect を使用したマルチテナント認証用に ASP.NET Core で認証ミドルウェアを構成する方法について説明します。

[スタートアップ クラス](/aspnet/core/fundamentals/startup)で、OpenID Connect ミドルウェアを追加します。

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectOptions {
    ClientId = configOptions.AzureAd.ClientId,
    ClientSecret = configOptions.AzureAd.ClientSecret, // for code flow
    Authority = Constants.AuthEndpointPrefix,
    ResponseType = OpenIdConnectResponseType.CodeIdToken,
    PostLogoutRedirectUri = configOptions.AzureAd.PostLogoutRedirectUri,
    SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    TokenValidationParameters = new TokenValidationParameters { ValidateIssuer = false },
    Events = new SurveyAuthenticationEvents(configOptions.AzureAd, loggerFactory),
});
```

設定の一部はランタイムの構成オプションから取られていることに注意してください。 ミドルウェア オプションの意味を次に示します。

* **ClientId**です。 Azure AD でアプリケーションを登録したときに取得した、アプリケーションのクライアント ID。
* **Authority**。 マルチテナント アプリケーションの場合、これを `https://login.microsoftonline.com/common/` に設定します。 これは、任意の Azure ADテナント からのユーザーがサインインできるようにする、Azure AD の共通 のエンドポイントの URL です。 共通のエンドポイントの詳細については、[このブログ記事](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/)をご覧ください。
* **TokenValidationParameters** で、**ValidateIssuer** を false に設定します。 つまり、ID トークンにおける発行者の値の検証をアプリが担当します。 (トークン自体はミドルウェアが引き続き検証します。)発行者の検証に関する詳細については、「[Issuer validation (発行者の検証)](claims.md#issuer-validation)」をご覧ください。
* **PostLogoutRedirectUri**。 サインアウトしたユーザーをリダイレクトする URL を指定します。これは、匿名の要求を許可するページ (通常はホーム ページ) です。
* **SignInScheme**。 `CookieAuthenticationDefaults.AuthenticationScheme`に設定します。 この設定は、ユーザーが承認された後、ユーザー要求がローカルの Cookie に保存されることを意味します。 この Cookie により、ブラウザー セッションでユーザーがログインした状態が保たれます。
* **Events**。 イベントのコールバックです。「[Authentication events (認証イベント)](#authentication-events)」をご覧ください。

さらに、Cookie 認証ミドルウェアをパイプラインに追加します。 このミドルウェアは、ユーザー要求を Cookie に書き込んだ後、以降のページ読み込み中に Cookie を読み取ります。

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions {
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    AccessDeniedPath = "/Home/Forbidden",
    CookieSecure = CookieSecurePolicy.Always,

    // The default setting for cookie expiration is 14 days. SlidingExpiration is set to true by default
    ExpireTimeSpan = TimeSpan.FromHours(1),
    SlidingExpiration = true
});
```

## <a name="initiate-the-authentication-flow"></a>認証フローの開始
ASP.NET MVC で認証フローを開始するには、次のように、コントローラーから **ChallengeResult** を返します。

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

これにより、ミドルウェアは、認証エンドポイントにリダイレクトする 302 (Found) 応答を返します。

## <a name="user-login-sessions"></a>ユーザー ログイン セッション
既に説明したように、ユーザーが初めてサインインするときに、Cookie 認証ミドルウェアがユーザー要求を Cookie に書き込みます。 それ以降、HTTP 要求は、Cookie を読み取ることによって認証されます。

既定では、Cookie ミドルウェアは[セッション Cookie][session-cookie] を書き込みます。セッション Cookie は、ユーザーがブラウザーを閉じると一度削除されます。 ユーザーは、次にそのサイトにアクセスする際、もう一度サインインする必要があります。 ただし、**ChallengeResult** で **IsPersistent** を true に設定すると、ミドルウェアは永続的な Cookie を書き込み、ユーザーはブラウザーを閉じた後もログインした状態のままとなります。 Cookie の有効期限を構成できます。[Cookie のオプションの制御][cookie-options]に関する記事をご覧ください。 永続的な Cookie はユーザーにとってより便利ですが、毎回ユーザーにサインインを求める必要がある一部のアプリケーション (銀行取引アプリケーションなど) には適切でない場合があります。

## <a name="about-the-openid-connect-middleware"></a>OpenID Connect ミドルウェアについて
ASP.NET の OpenID Connect ミドルウェアは、プロトコルの詳細のほとんどが公開されていません。 このセクションでは、プロトコル フローを理解するのに役立つ可能性のある、実装に関するいくつかの注意事項について説明します。

最初に、ASP.NET の観点から認証フローを見ていきましょう (アプリと Azure AD 間の OIDC プロトコル フローの詳細は無視します)。 そのプロセスを次の図に示します。

![Sign-in flow](./images/sign-in-flow.png)

この図では、2 つの MVC コントローラーがあります。 Account コントローラーはサインイン要求を処理し、Home コントローラーはホーム ページを提供します。

次に、認証プロセスを示します。

1. ユーザーが [サインイン] ボタンをクリックすると、ブラウザーが GET 要求を送信します。 たとえば、「 `GET /Account/SignIn/`」のように入力します。
2. Account コントローラーが `ChallengeResult`を返します。
3. OIDC ミドルウェアが、Azure AD にリダイレクトする HTTP 302 応答を返します。
4. ブラウザーが Azure AD に認証要求を送信します。
5. ユーザーが Azure AD にサインインすると、Azure AD から認証応答が返されます。
6. OIDC ミドルウェアが要求プリンシパルを作成して Cookie 認証ミドルウェアに渡します。
7. Cookie ミドルウェアが要求プリンシパルをシリアル化し、Cookie を設定します。
8. OIDC ミドルウェアにより、アプリケーションのコールバック URL にリダイレクトされます。
9. ブラウザーがそのリダイレクトに従い、要求で Cookie を送信します。
10. Cookie ミドルウェアが Cookie を要求プリンシパルに逆シリアル化し、要求プリンシパルと同じになるように `HttpContext.User` を設定します。 要求は MVC コントローラーにルーティングされます。

### <a name="authentication-ticket"></a>認証チケット
認証が成功すると、OIDC ミドルウェアは、ユーザーの要求を保持する要求プリンシパルが含まれている認証チケットを作成します。 **AuthenticationValidated** または **TicketReceived** イベント内のチケットにアクセスできます。

> [!NOTE]
> 認証フロー全体が完了するまで、`HttpContext.User` は認証されたユーザー**ではなく**、匿名のプリンシパルを引き続き保持します。 匿名プリンシパルには、空の要求コレクションがあります。 認証が完了し、アプリがリダイレクトされた後、Cookie ミドルウェアは、認証 Cookie を逆シリアル化し、 `HttpContext.User` を、認証されたユーザーを表す要求プリンシパルに設定します。
> 
> 

### <a name="authentication-events"></a>認証イベント
認証プロセス中、OpenID Connect ミドルウェアは、一連のイベントを発生させます。

* **RedirectToIdentityProvider**。 ミドルウェアが認証エンドポイントにリダイレクトされる直前に呼び出されます。 このイベントを使用すると、(たとえば、要求パラメーターを追加するために) リダイレクト URL を変更できます。 例については、「[Adding the admin consent prompt (管理者の同意プロンプトを追加する)](signup.md#adding-the-admin-consent-prompt)」をご覧ください。
* **AuthorizationCodeReceived**。 承認コードで呼び出されます。
* **TokenResponseReceived**。 ミドルウェアが IDP からアクセス トークンを取得した後、アクセス トークンが検証される前に呼び出されます。 承認コード フローにのみ当てはまります。
* **TokenValidated**。 ミドルウェアが ID トークンを検証した後に呼び出されます。 この時点で、アプリケーションには、ユーザーに関する一連の検証済みの要求があります。 このイベントを使用すると、要求に対する追加の検証を実行したり、要求を変換したりできます。 [要求の操作](claims.md)に関する記事をご覧ください。
* **UserInformationReceived**。 ミドルウェアがユーザー情報エンドポイントからユーザー プロファイルを取得する場合に呼び出されます。 承認コード フローのみ、およびミドルウェア オプションが `GetClaimsFromUserInfoEndpoint = true` の場合にのみ当てはまります。
* **TicketReceived**。 認証が完了したときに呼び出されます。 これは、認証が成功したことを想定した最後のイベントです。 このイベントが処理された後、ユーザーがアプリケーションにサインインします。
* **AuthenticationFailed**。 認証が失敗した場合に呼び出されます。 このイベントを使用して、エラー ページへのリダイレクトなどによって認証エラーを処理します。

これらのイベントのコールバックを指定するには、ミドルウェアで **Events** オプションを設定します。 イベント ハンドラーを宣言するには、ラムダを使用したインライン宣言と **OpenIdConnectEvents**から派生したクラスでの宣言という 2 つの方法があります。 イベントのコールバックに大量のロジックがあり、これらによってスタートアップ クラスが煩雑になるのを避ける場合は、2 番目のアプローチをお勧めします。 リファレンス実装では、このアプローチが使用されます。

### <a name="openid-connect-endpoints"></a>OpenID Connect のエンドポイント
Azure AD では [OpenID Connect 検出](https://openid.net/specs/openid-connect-discovery-1_0.html)がサポートされます。[既知のエンドポイント](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)から、ID プロバイダー (IDP) が JSON メタデータ ドキュメントを返します。 メタデータ ドキュメントには、次のような情報が含まれます。

* 承認エンドポイントの URL。 これは、アプリがユーザーの認証を行うためにリダイレクトされる場所です。
* アプリがユーザーをログアウトする "セッションの終了" エンドポイントの URL。
* IDP から取得した OIDC トークンの検証にクライアントが使用する署名キーを取得する URL。

既定では、OIDC ミドルウェアは、このメタデータを取り込む方法を認識しています。 ミドルウェアで **[機関]** オプションを設定すると、ミドルウェアはこのメタデータの URL を作成します。 (**MetadataAddress** オプションを設定して、メタデータ URL を上書きできます。)

### <a name="openid-connect-flows"></a>OpenID Connect のフロー
OIDC ミドルウェアは、既定で、フォーム ポスト応答モードによるハイブリッド フローを使用します。

* *ハイブリッド フロー* は、クライアントが承認サーバーへの同じラウンドトリップ内で ID トークンと承認コードを取得できることを意味します。
* *フォーム ポスト応答モード* は、承認サーバーが HTTP POST 要求を使用して ID トークンと承認コードをアプリに送信することを意味します。 値は form-urlencoded 形式 (content type = "application/x-www-form-urlencoded") です。

OIDC ミドルウェアが承認エンドポイントにリダイレクトされる場合、リダイレクト URL には、OIDC で必要なクエリ文字列パラメーターがすべて含まれます。 ハイブリッド フローのパラメーターは次のとおりです。

* client_id。 この値は、 **ClientId** オプションで設定します。
* スコープは "openid profile"、つまり、これは OIDC 要求であり、ユーザーのプロファイルが必要であるということを意味します。
* response_type は "id_token コード" です。 ハイブリッド フローを指定します。
* response_mode は "form_post" です。 フォーム ポスト応答を指定します。

別のフローを指定するには、オプションの **ResponseType** プロパティを設定します。 例: 

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

[**次へ**][claims]

[claims]: claims.md
[cookie-options]: /aspnet/core/security/authentication/cookie#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
