---
title: アプリケーション ロール
description: アプリケーション ロールを使用して承認を実行する方法について説明します。
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: 4a694eb65de717e6b5a7c65a2d6fb28f192dcdc5
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902512"
---
# <a name="application-roles"></a>アプリケーション ロール

[![GitHub](../_images/github.png) サンプル コード][sample application]

アプリケーション ロールは、ユーザーにアクセス許可を割り当てるために使用されます。 たとえば、[Tailspin Surveys][Tailspin] アプリケーションでは、次のロールが定義されています。

* 管理者。 そのテナントに属するすべてのアンケートに対してすべての CRUD 操作を実行できます。
* 作成者。 新しいアンケートを作成できます。
* 閲覧者。 そのテナントに属するすべてのアンケートを読み取ることができます。

[承認]処理で、最終的にロールはアクセス許可に変換されることがわかります。 ただ、最初の問題は、どのようにロールを割り当てて管理するかです。 ここでは、主に 3 つのオプションを指定しました。

* [Azure AD アプリ ロール](#roles-using-azure-ad-app-roles)
* [Azure AD セキュリティ グループ](#roles-using-azure-ad-security-groups)
* [アプリケーション ロール マネージャー](#roles-using-an-application-role-manager)

## <a name="roles-using-azure-ad-app-roles"></a>Azure AD アプリ ロールを使用したロール
これは Tailspin Surveys アプリで使用したアプローチです。

このアプローチでは、SaaS プロバイダーがアプリケーション マニフェストにアプリケーション ロールを追加して定義します。 顧客がサインアップした後に、顧客の AD ディレクトリ管理者がユーザーをロールに割り当てます。 ユーザーがサインインすると、ユーザーが割り当てたロールが要求として送信されます。

> [!NOTE]
> 顧客が Azure AD Premium を利用している場合、管理者はセキュリティ グループをロールに割り当て、グループのメンバーがそのアプリ ロールを継承します。 グループ所有者は AD 管理者になる必要はないため、これはロール管理する上で便利な方法です。
> 
> 

このアプローチの長所:

* 単純なプログラミング モデル。
* ロールはアプリケーションに固有です。 1 つのアプリケーションに対するロール要求は、別のアプリケーションに送信されません。
* 顧客が AD テナントからアプリケーションを削除すると、ロールの割り当ては解除されます。
* アプリケーションは、ユーザーのプロファイルの読み取り以外の追加の Active Directory アクセス許可を必要としません。

短所:

* Azure AD Premium を利用していない顧客は、セキュリティ グループをロールに割り当てることができません。 このような顧客の場合、AD 管理者がすべてのユーザーの割り当てを実行する必要があります。
* Web アプリとは別にバックエンド Web API がある場合、Web アプリのロール割り当ては Web API に適用されません。 この点の詳細については、 [バックエンド Web API のセキュリティ保護]に関するページを参照してください。

### <a name="implementation"></a>実装
**ロールを定義します**。 SaaS プロバイダーが、[アプリケーション マニフェスト]でアプリのロールを宣言します。 たとえば、Surveys アプリのマニフェスト エントリは次のとおりです。

```json
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

`value` プロパティはロール要求に含まれます。 `id` プロパティは、定義されたロールの一意の識別子です。 常に `id`の新しい GUID 値を生成します。

**ユーザーを割り当てます**。 新しい顧客がサインアップすると、アプリケーションは顧客の AD テナントに登録されます。 この時点で、そのテナントの AD 管理者はユーザーをロールに割り当てることができるようになります。

> [!NOTE]
> 前述のように、Azure AD Premium を利用している顧客は、セキュリティ グループをロールに割り当てることもできます。
> 
> 

Azure Portal の次のスクリーンショットは、Surveys アプリケーションのユーザーとグループを示しています。 管理者と作成者はグループであり、それぞれ SurveyAdmin ロール、SurveyCreator ロールに割り当てられています。 Alice は、SurveyAdmin ロールに直接割り当てられたユーザーです。 Bob と Charles は、ロールに直接割り当てられていないユーザーです。

![ユーザーとグループ](./images/running-the-app/users-and-groups.png)

次のスクリーンショットに示すように、Charles は管理者グループに属しているので、SurveyAdmin ロールを継承します。 Bob にはロールがまだ割り当てられていません。

![管理者グループのメンバー](./images/running-the-app/admin-members.png)


> [!NOTE]
> 別の方法として、アプリケーションで Azure AD Graph API を使用してプログラムでロールを割り当てることもできます。 ただし、この場合、アプリケーションは顧客の AD ディレクトリに対する書き込みアクセス許可を取得する必要があります。 これらのアクセス許可を持つアプリケーションは、さまざまな悪影響を及ぼす可能性がありますが、顧客はアプリを信頼しており、ディレクトリが台無しにされるとは思っていません。 多くの顧客は、このレベルのアクセス権を付与することを嫌う場合があります。
> 

**ロール要求を取得します**。 ユーザーがサインインすると、アプリケーションは種類が `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`の要求でユーザーに割り当てられたロールを受け取ります。  

ユーザーには、複数のロールが割り当てられている場合もあれば、ロールが割り当てられていない場合もあります。 承認コードでは、ユーザーがロール要求を 1 つだけ持っていることを前提としないでください。 代わりに、特定の要求の値が存在するかどうかを確認するコードを記述します。

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a>Azure AD セキュリティ グループを使用したロール
このアプローチでは、ロールは AD セキュリティ グループとして表されます。 アプリケーションは、セキュリティ グループ メンバーシップに基づいてアクセス許可をユーザーに割り当てます。

長所:

* Azure AD Premium を利用していない顧客の場合、このアプローチで、セキュリティ グループを使用してロールの割り当てを管理できます。

短所:

* 複雑。 各テナントが異なるグループ要求を送信するため、アプリは、各テナントについて、どのセキュリティ グループがどのアプリケーション ロールに対応するかを追跡する必要があります。
* 顧客が AD テナントからアプリケーションを削除しても、セキュリティ グループは AD ディレクトリに残ります。

### <a name="implementation"></a>実装
アプリケーション マニフェストで、 `groupMembershipClaims` プロパティを "SecurityGroup" に設定します。 この設定は、AAD からグループ メンバーシップ要求を取得するために必要です。

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

新しい顧客がサインアップすると、アプリケーションに必要なロールのセキュリティ グループを作成するように指示されます。 顧客は、グループのオブジェクト ID をアプリケーションに入力する必要があります。 この入力内容は、テナントごとにグループ ID をアプリケーション ロールに対応付けたテーブルに保存されます。

> [!NOTE]
> また、Azure AD Graph API を使用して、アプリケーションで自動的にグループを作成することもできます。  自動的に作成する方がエラーが少なくなります。 ただし、顧客の AD ディレクトリの "すべてのグループの読み取りおよび書き込み" アクセス許可をアプリケーションが取得する必要があります。 多くの顧客は、このレベルのアクセス権を付与することを嫌う場合があります。
> 
> 

ユーザーがサインインした場合:

1. アプリケーションは要求としてユーザーのグループを受け取ります。 各要求の値はグループのオブジェクト ID です。
2. Azure AD は、トークンで送信されるグループ数を制限しています。 グループ数がその制限を超えると、Azure AD から特殊な "過剰" 要求が送信されます。 この要求が存在する場合、アプリケーションから Azure AD Graph API にクエリして、そのユーザーが属するすべてのグループを取得する必要があります。 詳細については、「[Authorization in Cloud Applications using AD Groups (AD グループを使用したクラウド アプリケーションでの承認)]」の「Groups claim overage (グループの過剰要求)」をご覧ください。
3. アプリケーションはアプリケーションのデータベースからオブジェクト ID を検索し、対応するアプリケーション ロールを見つけて、ユーザーに割り当てます。
4. アプリケーションは、アプリケーション ロールを表すユーザー プリンシパルにカスタム要求値を追加します。 例: `survey_role` = "SurveyAdmin".

承認ポリシーには、グループ要求ではなく、カスタムロール要求を使用する必要があります。

## <a name="roles-using-an-application-role-manager"></a>アプリケーション ロール マネージャーを使用したロール
この方法では、アプリケーション ロールは Azure AD には保存されません。 代わりに、アプリケーションでは ASP.NET Identity の **RoleManager** クラスなどを使用して、各ユーザーのロールの割り当てを独自の DB に保存します。

長所:

* アプリはロールとユーザー割り当てを完全に制御できます。

短所:

* さらに複雑で、保守が困難になります。
* ロール割り当ての管理に AD セキュリティ グループを使用できません。
* アプリケーション データベースにユーザー情報が保存され、ユーザーが追加または削除されても、テナントの AD ディレクトリと同期されない可能性があります。   


[**次へ**][承認]

<!-- Links -->
[Tailspin]: tailspin.md

[承認]: authorize.md
[バックエンド Web API のセキュリティ保護]: web-api.md
[アプリケーション マニフェスト]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
