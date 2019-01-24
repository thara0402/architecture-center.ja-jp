---
title: マルチテナント アプリケーションでの承認
description: マルチテナント アプリケーションで承認を行う方法。
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 1238f1cd3bc8e6f5f3d174ea5fa0c98b4da795f3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488615"
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="99cb6-103">ロールベースおよびリソースベースの承認</span><span class="sxs-lookup"><span data-stu-id="99cb6-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="99cb6-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="99cb6-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="99cb6-105">Microsoft の[参照実装]は、ASP.NET Core アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="99cb6-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="99cb6-106">この記事では、ASP.NET Core で作成された認証 API を使用した一般的な承認のアプローチを 2 つ説明します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="99cb6-107">**ロールベースの承認**。</span><span class="sxs-lookup"><span data-stu-id="99cb6-107">**Role-based authorization**.</span></span> <span data-ttu-id="99cb6-108">ユーザーに割り当てられたロールに基づいてアクションを承認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="99cb6-109">たとえば、一部の操作に管理者ロールを必須にします。</span><span class="sxs-lookup"><span data-stu-id="99cb6-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="99cb6-110">**リソースベースの承認**。</span><span class="sxs-lookup"><span data-stu-id="99cb6-110">**Resource-based authorization**.</span></span> <span data-ttu-id="99cb6-111">特定のリソースに基づいてアクションを承認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="99cb6-112">たとえば、すべてのリソースには所有者がいます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-112">For example, every resource has an owner.</span></span> <span data-ttu-id="99cb6-113">所有者はリソースを削除できますが、他のユーザーは削除できません。</span><span class="sxs-lookup"><span data-stu-id="99cb6-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="99cb6-114">一般的なアプリは両方のアプローチを採用します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="99cb6-115">たとえば、リソースを削除する場合、ユーザーはリソース所有者 *または* 管理者である必要があります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="99cb6-116">ロールベースの承認</span><span class="sxs-lookup"><span data-stu-id="99cb6-116">Role-Based Authorization</span></span>

<span data-ttu-id="99cb6-117">[Tailspin Surveys][Tailspin] アプリケーションでは次のロールが定義されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="99cb6-118">管理者。</span><span class="sxs-lookup"><span data-stu-id="99cb6-118">Administrator.</span></span> <span data-ttu-id="99cb6-119">そのテナントに属するすべてのアンケートに対してすべての CRUD 操作を実行できます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="99cb6-120">作成者。</span><span class="sxs-lookup"><span data-stu-id="99cb6-120">Creator.</span></span> <span data-ttu-id="99cb6-121">新しいアンケートを作成できます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-121">Can create new surveys</span></span>
* <span data-ttu-id="99cb6-122">閲覧者。</span><span class="sxs-lookup"><span data-stu-id="99cb6-122">Reader.</span></span> <span data-ttu-id="99cb6-123">そのテナントに属するすべてのアンケートを読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="99cb6-124">ロールは、アプリケーションの *ユーザー* に適用されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="99cb6-125">Surveys アプリケーションのユーザーは、管理者、作成者、または閲覧者です。</span><span class="sxs-lookup"><span data-stu-id="99cb6-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="99cb6-126">ロールを定義および管理する方法については、 [アプリケーション ロール]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="99cb6-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="99cb6-127">ロールを管理する方法にかかわらず、承認コードは同様になります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="99cb6-128">ASP.NET Core は、[承認ポリシー][policies]という抽象化を使用しています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="99cb6-129">この機能を使用してコードで承認ポリシーを定義し、そのポリシーをコントローラー アクションに適用します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="99cb6-130">ポリシーは、コントローラーから切り離されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="99cb6-131">ポリシーの作成</span><span class="sxs-lookup"><span data-stu-id="99cb6-131">Create policies</span></span>

<span data-ttu-id="99cb6-132">ポリシーを定義するには、まず `IAuthorizationRequirement`を実装するクラスを作成します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="99cb6-133">`AuthorizationHandler`から派生するのが最も簡単です。</span><span class="sxs-lookup"><span data-stu-id="99cb6-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="99cb6-134">`Handle` メソッドで、関連する要求を確認してください。</span><span class="sxs-lookup"><span data-stu-id="99cb6-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="99cb6-135">Tailspin Surveys アプリケーションの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="99cb6-136">このクラスは、新しいアンケートを作成するユーザーの要件を定義しています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="99cb6-137">ユーザーには SurveyAdmin または SurveyCreator ロールが割り当てられている必要があります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="99cb6-138">startup クラスで、1 つ以上の要件を含む名前付きポリシーを定義します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="99cb6-139">複数の要件がある場合、ユーザーが承認されるには、 *すべて* の要件を満たす必要があります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="99cb6-140">次のコードでは、2 つのポリシーを定義しています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="99cb6-141">このコードにより認証スキームも設定され、承認に失敗した場合は、どの認証ミドルウェアを実行すべきかを ASP.NET に伝えます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="99cb6-142">ここでは、Cookie 認証ミドルウェアを指定しますが、その理由は Cookie 認証ミドルウェアがユーザーを "禁止された" ページにリダイレクトできるためです。</span><span class="sxs-lookup"><span data-stu-id="99cb6-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="99cb6-143">禁止されたページの場所は、Cookie ミドルウェアの `AccessDeniedPath` オプションで設定されます。詳しくは、「[認証ミドルウェアの構成]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="99cb6-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="99cb6-144">コントローラー アクションを承認する</span><span class="sxs-lookup"><span data-stu-id="99cb6-144">Authorize controller actions</span></span>

<span data-ttu-id="99cb6-145">最後に、MVC コントローラーのアクションを承認するには、 `Authorize` 属性のポリシーを設定します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="99cb6-146">旧バージョンの ASP.NET では、この属性の **Roles** プロパティを設定します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="99cb6-147">このプロパティは ASP.NET Core でもサポートされていますが、承認ポリシーと比較すると短所があります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="99cb6-148">特定の要求の種類を前提としています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-148">It assumes a particular claim type.</span></span> <span data-ttu-id="99cb6-149">ポリシーであれば、任意の要求の種類を確認できます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-149">Policies can check for any claim type.</span></span> <span data-ttu-id="99cb6-150">ロールは 1 種類の要求のみです。</span><span class="sxs-lookup"><span data-stu-id="99cb6-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="99cb6-151">ロール名は、属性にハードコーディングされています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="99cb6-152">ポリシーによりすべての承認ロジックが 1 つの場所にあるため、構成設計の更新、読み込みが簡単にできます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="99cb6-153">ポリシーを使用すると、1 つのロール メンバーシップでは表現できない、より複雑な承認判断 (21 歳以上の年齢など) を実行できます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="99cb6-154">リソース ベースの承認</span><span class="sxs-lookup"><span data-stu-id="99cb6-154">Resource based authorization</span></span>

<span data-ttu-id="99cb6-155">*リソース ベースの承認* は、操作の影響を受ける特定のリソースに承認が依存している場合に常に発生します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="99cb6-156">Tailspin Surveys アプリケーションでは、すべてのアンケートに 1 人の所有者がいて、0 対多の共同作成者がいます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="99cb6-157">所有者は、アンケートの読み取り、更新、削除、発行、発行の取り消しを行うことができます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="99cb6-158">所有者は、アンケートに共同作成者を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="99cb6-159">共同作成者は、アンケートの読み取りと更新を行うことができます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="99cb6-160">"所有者" と "共同作成者" はアプリケーション ロールではなく、アンケートごとにアプリケーション データベースに格納されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="99cb6-161">ユーザーがアンケートを削除できるかどうかを確認するには、たとえば、アプリでそのユーザーがそのアンケートの所有者かどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="99cb6-162">ASP.NET Core で、**AuthorizationHandler** から派生させ、**ハンドル** メソッドをオーバーライドして、リソース ベースの承認を実装します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="99cb6-163">Survey オブジェクトのこのクラスは厳密に型指定されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="99cb6-164">開始時に DI のクラスを登録します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="99cb6-165">承認チェックを実行するには、**IAuthorizationService** インターフェイスを使用して、コントローラーに挿入します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="99cb6-166">次のコードを使用すると、ユーザーがアンケートを読み取れるかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="99cb6-167">ここでは `Survey` オブジェクトで渡すため、これにより `SurveyAuthorizationHandler` が呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="99cb6-168">承認コードでは、ユーザーのすべてのロールベースのアクセス許可とリソースベースのアクセス許可を集計し、目的の操作に対して集計セットを確認することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="99cb6-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="99cb6-169">Surveys アプリの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="99cb6-170">このアプリケーションには、いくつかのアクセス許可が定義されています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-170">The application defines several permission types:</span></span>

* <span data-ttu-id="99cb6-171">[Admin]</span><span class="sxs-lookup"><span data-stu-id="99cb6-171">Admin</span></span>
* <span data-ttu-id="99cb6-172">Contributor</span><span class="sxs-lookup"><span data-stu-id="99cb6-172">Contributor</span></span>
* <span data-ttu-id="99cb6-173">作成者</span><span class="sxs-lookup"><span data-stu-id="99cb6-173">Creator</span></span>
* <span data-ttu-id="99cb6-174">Owner</span><span class="sxs-lookup"><span data-stu-id="99cb6-174">Owner</span></span>
* <span data-ttu-id="99cb6-175">Reader</span><span class="sxs-lookup"><span data-stu-id="99cb6-175">Reader</span></span>

<span data-ttu-id="99cb6-176">また、アンケートに対して実行できる操作のセットも定義されています。</span><span class="sxs-lookup"><span data-stu-id="99cb6-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="99cb6-177">Create</span><span class="sxs-lookup"><span data-stu-id="99cb6-177">Create</span></span>
* <span data-ttu-id="99cb6-178">読み取り</span><span class="sxs-lookup"><span data-stu-id="99cb6-178">Read</span></span>
* <span data-ttu-id="99cb6-179">更新</span><span class="sxs-lookup"><span data-stu-id="99cb6-179">Update</span></span>
* <span data-ttu-id="99cb6-180">削除</span><span class="sxs-lookup"><span data-stu-id="99cb6-180">Delete</span></span>
* <span data-ttu-id="99cb6-181">発行</span><span class="sxs-lookup"><span data-stu-id="99cb6-181">Publish</span></span>
* <span data-ttu-id="99cb6-182">発行の取り消し</span><span class="sxs-lookup"><span data-stu-id="99cb6-182">Unpublish</span></span>

<span data-ttu-id="99cb6-183">次のコードは、特定のユーザーとアンケートのアクセス許可の一覧を作成します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="99cb6-184">このコードは、ユーザーのアプリ ロールと、アンケートの所有者/共同作成者フィールドの両方を確認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="99cb6-185">マルチ テナント アプリケーションでは、アクセス許可が別のテナント データに "リーク" しないことを確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="99cb6-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="99cb6-186">Surveys アプリでは、共同作成者のアクセス許可はテナント間で認められており、別のテナントのユーザーを共同作成者として割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contributor.</span></span> <span data-ttu-id="99cb6-187">その他のアクセス許可の種類は、そのユーザーのテナントに属するリソースに制限されます。</span><span class="sxs-lookup"><span data-stu-id="99cb6-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="99cb6-188">この要件を適用するには、アクセス許可を付与する前に、コードでテナント ID を確認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="99cb6-189">(アンケートの作成時に割り当てられた `TenantId` フィールドです。)</span><span class="sxs-lookup"><span data-stu-id="99cb6-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="99cb6-190">次の手順では、アクセス許可に対して操作 (読み取り、更新、削除など) を確認します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="99cb6-191">Surveys アプリは、関数のルックアップ テーブルを使用して、この手順を実装します。</span><span class="sxs-lookup"><span data-stu-id="99cb6-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="99cb6-192">[**次へ**][web-api]</span><span class="sxs-lookup"><span data-stu-id="99cb6-192">[**Next**][web-api]</span></span>

<!-- links -->

[Tailspin]: tailspin.md

[アプリケーション ロール]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[参照実装]: tailspin.md
[reference implementation]: tailspin.md
[認証ミドルウェアの構成]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
