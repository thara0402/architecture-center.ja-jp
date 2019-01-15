---
title: マルチテナント アプリケーションでの承認
description: マルチテナント アプリケーションで承認を行う方法。
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 6e406a7e80b77dea161db194a82ccae043bdc777
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110341"
---
# <a name="role-based-and-resource-based-authorization"></a>ロールベースおよびリソースベースの承認

[![GitHub](../_images/github.png) サンプル コード][sample application]

Microsoft の[参照実装]は、ASP.NET Core アプリケーションです。 この記事では、ASP.NET Core で作成された認証 API を使用した一般的な承認のアプローチを 2 つ説明します。

* **ロールベースの承認**。 ユーザーに割り当てられたロールに基づいてアクションを承認します。 たとえば、一部の操作に管理者ロールを必須にします。
* **リソースベースの承認**。 特定のリソースに基づいてアクションを承認します。 たとえば、すべてのリソースには所有者がいます。 所有者はリソースを削除できますが、他のユーザーは削除できません。

一般的なアプリは両方のアプローチを採用します。 たとえば、リソースを削除する場合、ユーザーはリソース所有者 *または* 管理者である必要があります。

## <a name="role-based-authorization"></a>ロールベースの承認

[Tailspin Surveys][Tailspin] アプリケーションでは次のロールが定義されます。

* 管理者。 そのテナントに属するすべてのアンケートに対してすべての CRUD 操作を実行できます。
* 作成者。 新しいアンケートを作成できます。
* 閲覧者。 そのテナントに属するすべてのアンケートを読み取ることができます。

ロールは、アプリケーションの *ユーザー* に適用されます。 Surveys アプリケーションのユーザーは、管理者、作成者、または閲覧者です。

ロールを定義および管理する方法については、 [アプリケーション ロール]に関するページを参照してください。

ロールを管理する方法にかかわらず、承認コードは同様になります。 ASP.NET Core は、[承認ポリシー][policies]という抽象化を使用しています。 この機能を使用してコードで承認ポリシーを定義し、そのポリシーをコントローラー アクションに適用します。 ポリシーは、コントローラーから切り離されます。

### <a name="create-policies"></a>ポリシーの作成

ポリシーを定義するには、まず `IAuthorizationRequirement`を実装するクラスを作成します。 `AuthorizationHandler`から派生するのが最も簡単です。 `Handle` メソッドで、関連する要求を確認してください。

Tailspin Surveys アプリケーションの例を次に示します。

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) ||
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

このクラスは、新しいアンケートを作成するユーザーの要件を定義しています。 ユーザーには SurveyAdmin または SurveyCreator ロールが割り当てられている必要があります。

startup クラスで、1 つ以上の要件を含む名前付きポリシーを定義します。 複数の要件がある場合、ユーザーが承認されるには、 *すべて* の要件を満たす必要があります。 次のコードでは、2 つのポリシーを定義しています。

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

このコードにより認証スキームも設定され、承認に失敗した場合は、どの認証ミドルウェアを実行すべきかを ASP.NET に伝えます。 ここでは、Cookie 認証ミドルウェアを指定しますが、その理由は Cookie 認証ミドルウェアがユーザーを "禁止された" ページにリダイレクトできるためです。 禁止されたページの場所は、Cookie ミドルウェアの `AccessDeniedPath` オプションで設定されます。詳しくは、「[認証ミドルウェアの構成]」をご覧ください。

### <a name="authorize-controller-actions"></a>コントローラー アクションを承認する

最後に、MVC コントローラーのアクションを承認するには、 `Authorize` 属性のポリシーを設定します。

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

旧バージョンの ASP.NET では、この属性の **Roles** プロパティを設定します。

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

このプロパティは ASP.NET Core でもサポートされていますが、承認ポリシーと比較すると短所があります。

* 特定の要求の種類を前提としています。 ポリシーであれば、任意の要求の種類を確認できます。 ロールは 1 種類の要求のみです。
* ロール名は、属性にハードコーディングされています。 ポリシーによりすべての承認ロジックが 1 つの場所にあるため、構成設計の更新、読み込みが簡単にできます。
* ポリシーを使用すると、1 つのロール メンバーシップでは表現できない、より複雑な承認判断 (21 歳以上の年齢など) を実行できます。

## <a name="resource-based-authorization"></a>リソース ベースの承認

*リソース ベースの承認* は、操作の影響を受ける特定のリソースに承認が依存している場合に常に発生します。 Tailspin Surveys アプリケーションでは、すべてのアンケートに 1 人の所有者がいて、0 対多の共同作成者がいます。

* 所有者は、アンケートの読み取り、更新、削除、発行、発行の取り消しを行うことができます。
* 所有者は、アンケートに共同作成者を割り当てることができます。
* 共同作成者は、アンケートの読み取りと更新を行うことができます。

"所有者" と "共同作成者" はアプリケーション ロールではなく、アンケートごとにアプリケーション データベースに格納されます。 ユーザーがアンケートを削除できるかどうかを確認するには、たとえば、アプリでそのユーザーがそのアンケートの所有者かどうかを確認します。

ASP.NET Core で、**AuthorizationHandler** から派生させ、**ハンドル** メソッドをオーバーライドして、リソース ベースの承認を実装します。

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Survey オブジェクトのこのクラスは厳密に型指定されます。  開始時に DI のクラスを登録します。

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

承認チェックを実行するには、**IAuthorizationService** インターフェイスを使用して、コントローラーに挿入します。 次のコードを使用すると、ユーザーがアンケートを読み取れるかどうかを確認できます。

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

ここでは `Survey` オブジェクトで渡すため、これにより `SurveyAuthorizationHandler` が呼び出されます。

承認コードでは、ユーザーのすべてのロールベースのアクセス許可とリソースベースのアクセス許可を集計し、目的の操作に対して集計セットを確認することをお勧めします。
Surveys アプリの例を次に示します。 このアプリケーションには、いくつかのアクセス許可が定義されています。

* [Admin]
* Contributor
* 作成者
* Owner
* Reader

また、アンケートに対して実行できる操作のセットも定義されています。

* Create
* 読み取り
* 更新
* 削除
* 発行
* 発行の取り消し

次のコードは、特定のユーザーとアンケートのアクセス許可の一覧を作成します。 このコードは、ユーザーのアプリ ロールと、アンケートの所有者/共同作成者フィールドの両方を確認します。

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

マルチ テナント アプリケーションでは、アクセス許可が別のテナント データに "リーク" しないことを確認する必要があります。 Surveys アプリでは、共同作成者のアクセス許可はテナント間で認められており、別のテナントのユーザーを共同作成者として割り当てることができます。 その他のアクセス許可の種類は、そのユーザーのテナントに属するリソースに制限されます。 この要件を適用するには、アクセス許可を付与する前に、コードでテナント ID を確認します。 (アンケートの作成時に割り当てられた `TenantId` フィールドです。)

次の手順では、アクセス許可に対して操作 (読み取り、更新、削除など) を確認します。 Surveys アプリは、関数のルックアップ テーブルを使用して、この手順を実装します。

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**次へ**][web-api]

<!-- links -->

[Tailspin]: tailspin.md

[アプリケーション ロール]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[参照実装]: tailspin.md
[認証ミドルウェアの構成]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
