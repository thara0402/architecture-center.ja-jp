---
title: マルチテナント アプリケーションにおけるバックエンド Web API のセキュリティ保護
description: バックエンド Web API をセキュリティで保護する方法。
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 517bdbb6e1a1063db9337b63905e2ff5f4bdd4d4
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114030"
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="452a1-103">バックエンド Web API をセキュリティで保護する</span><span class="sxs-lookup"><span data-stu-id="452a1-103">Secure a backend web API</span></span>

<span data-ttu-id="452a1-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="452a1-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="452a1-105">[Tailspin Surveys] アプリケーションは、バックエンド Web API を使用してアンケートに対する CRUD 操作を管理しています。</span><span class="sxs-lookup"><span data-stu-id="452a1-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="452a1-106">たとえば、ユーザーが [My Surveys] をクリックすると、Web アプリケーションから HTTP 要求が Web API に送信されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```http
GET /users/{userId}/surveys
```

<span data-ttu-id="452a1-107">Web API からは JSON オブジェクトが返されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-107">The web API returns a JSON object:</span></span>

```http
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

<span data-ttu-id="452a1-108">Web API は匿名要求を許可しないため、Web アプリは OAuth 2 ベアラー トークンを使用して自身を認証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="452a1-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="452a1-109">これはサーバー対サーバーのシナリオです。</span><span class="sxs-lookup"><span data-stu-id="452a1-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="452a1-110">アプリケーションは、ブラウザーから API に対して AJAX の呼び出しを実行しません。</span><span class="sxs-lookup"><span data-stu-id="452a1-110">The application does not make any AJAX calls to the API from the browser client.</span></span>

<span data-ttu-id="452a1-111">使用できる主なアプローチは 2 つです。</span><span class="sxs-lookup"><span data-stu-id="452a1-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="452a1-112">デリゲートされたユーザー ID。</span><span class="sxs-lookup"><span data-stu-id="452a1-112">Delegated user identity.</span></span> <span data-ttu-id="452a1-113">Web アプリケーションは、ユーザーの ID を使用して認証します。</span><span class="sxs-lookup"><span data-stu-id="452a1-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="452a1-114">アプリケーション ID。</span><span class="sxs-lookup"><span data-stu-id="452a1-114">Application identity.</span></span> <span data-ttu-id="452a1-115">Web アプリケーションは、OAuth2 クライアント資格情報フローで、自身のクライアント ID を使用して認証します。</span><span class="sxs-lookup"><span data-stu-id="452a1-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="452a1-116">Tailspin アプリケーションは、デリゲートされたユーザー ID を実装しています。</span><span class="sxs-lookup"><span data-stu-id="452a1-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="452a1-117">主な違いは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="452a1-117">Here are the main differences:</span></span>

<span data-ttu-id="452a1-118">**委任されたユーザー ID:**</span><span class="sxs-lookup"><span data-stu-id="452a1-118">**Delegated user identity:**</span></span>

* <span data-ttu-id="452a1-119">Web API に送信されるベアラー トークンには、ユーザー ID が含まれています。</span><span class="sxs-lookup"><span data-stu-id="452a1-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="452a1-120">Web API は、ユーザー ID に基づいて承認を決定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="452a1-121">ユーザーにアクションを実行する権限がない場合、Web アプリケーションは Web API から送信される 403 (Forbidden) エラーを処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="452a1-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="452a1-122">通常、Web アプリケーションは、UI に影響がある何らかの承認の決定を行います (UI 要素の表示、非表示など)。</span><span class="sxs-lookup"><span data-stu-id="452a1-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="452a1-123">Web API は、信用されていないクライアント (JavaScript アプリケーションや、ネイティブ クライアント アプリケーションなど) によって使用されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="452a1-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="452a1-124">**アプリケーション ID:**</span><span class="sxs-lookup"><span data-stu-id="452a1-124">**Application identity:**</span></span>

* <span data-ttu-id="452a1-125">Web API は、ユーザーに関する情報を取得しません。</span><span class="sxs-lookup"><span data-stu-id="452a1-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="452a1-126">Web API は、ユーザー ID に基づく承認を実行できません。</span><span class="sxs-lookup"><span data-stu-id="452a1-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="452a1-127">すべての承認は、Web アプリケーションが決定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="452a1-128">信頼されていないクライアント (JavaScript やネイティブ クライアント アプリケーション) は、Web API を使用できません。</span><span class="sxs-lookup"><span data-stu-id="452a1-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="452a1-129">このアプローチは、Web API に承認ロジックがないため、実装がやや簡単です。</span><span class="sxs-lookup"><span data-stu-id="452a1-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="452a1-130">いずれのアプローチにおいても、Web アプリケーションは Web API を呼び出すのに必要な資格情報であるアクセス トークンを取得する必要があります。</span><span class="sxs-lookup"><span data-stu-id="452a1-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="452a1-131">デリゲートされたユーザー ID の場合、トークンは、ユーザーの代わりにトークンを発行できる IDP から入手します。</span><span class="sxs-lookup"><span data-stu-id="452a1-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="452a1-132">クライアントの資格情報の場合、アプリケーションは IDP からトークンを取得するか、独自のトークン サーバーをホストします。</span><span class="sxs-lookup"><span data-stu-id="452a1-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="452a1-133">(ただし、トークン サーバーを最初から作成することはせずに、十分にテストされているフレームワーク ([IdentityServer4] など) を使用してください。)Azure AD で認証する場合、クライアントの資格情報フローでも、Azure AD からアクセス トークンを取得することを強くお勧めします。</span><span class="sxs-lookup"><span data-stu-id="452a1-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer4].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="452a1-134">以降、この記事では、アプリケーションが Azure AD を使用して認証している前提で説明します。</span><span class="sxs-lookup"><span data-stu-id="452a1-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![アクセス トークンを取得する](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="452a1-136">Azure AD に Web API を登録する</span><span class="sxs-lookup"><span data-stu-id="452a1-136">Register the web API in Azure AD</span></span>

<span data-ttu-id="452a1-137">Azure AD から Web API のベアラー トークンを発行するには、Azure AD で構成が必要です。</span><span class="sxs-lookup"><span data-stu-id="452a1-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="452a1-138">Azure AD に Web API を登録します。</span><span class="sxs-lookup"><span data-stu-id="452a1-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="452a1-139">Web アプリのクライアント ID を、Web API アプリケーション マニフェストの `knownClientApplications` プロパティに追加します。</span><span class="sxs-lookup"><span data-stu-id="452a1-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="452a1-140">[アプリケーション マニフェストの更新]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="452a1-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="452a1-141">Web API を呼び出すアクセス許可を Web アプリケーションに付与します。</span><span class="sxs-lookup"><span data-stu-id="452a1-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="452a1-142">Azure の管理ポータルでは、アプリケーション ID 向け (クライアントの資格情報フロー) の "アプリケーションのアクセス許可"、または委任されたユーザー ID 向けの "委任されたアクセス許可" という 2 種類のアクセス許可を設定できます。</span><span class="sxs-lookup"><span data-stu-id="452a1-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>

   ![デリゲートされたアクセス許可](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="452a1-144">アクセス トークンの取得</span><span class="sxs-lookup"><span data-stu-id="452a1-144">Getting an access token</span></span>

<span data-ttu-id="452a1-145">Web API を呼び出す前に、Web アプリケーションは Azure AD からアクセス トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="452a1-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="452a1-146">.NET アプリケーションで、[Azure AD Authentication Library (ADAL) for .NET][ADAL] を使用します。</span><span class="sxs-lookup"><span data-stu-id="452a1-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="452a1-147">OAuth 2 承認コード フローの場合、アプリケーションはアクセス トークンの承認コードを交換します。</span><span class="sxs-lookup"><span data-stu-id="452a1-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="452a1-148">次のコードでは、ADAL を使用してアクセス トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="452a1-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="452a1-149">このコードは、 `AuthorizationCodeReceived` イベント中に呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

<span data-ttu-id="452a1-150">必要な各種パラメーターを次に示します。</span><span class="sxs-lookup"><span data-stu-id="452a1-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="452a1-151">`authority`</span><span class="sxs-lookup"><span data-stu-id="452a1-151">`authority`.</span></span> <span data-ttu-id="452a1-152">サインインしたユーザーのテナント ID が元になります。</span><span class="sxs-lookup"><span data-stu-id="452a1-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="452a1-153">(SaaS プロバイダーのテナント ID ではありません)</span><span class="sxs-lookup"><span data-stu-id="452a1-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="452a1-154">`authorizationCode`</span><span class="sxs-lookup"><span data-stu-id="452a1-154">`authorizationCode`.</span></span> <span data-ttu-id="452a1-155">IDP から取得し直した認証コード。</span><span class="sxs-lookup"><span data-stu-id="452a1-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="452a1-156">`clientId`</span><span class="sxs-lookup"><span data-stu-id="452a1-156">`clientId`.</span></span> <span data-ttu-id="452a1-157">Web アプリケーションのクライアント ID。</span><span class="sxs-lookup"><span data-stu-id="452a1-157">The web application's client ID.</span></span>
* <span data-ttu-id="452a1-158">`clientSecret`</span><span class="sxs-lookup"><span data-stu-id="452a1-158">`clientSecret`.</span></span> <span data-ttu-id="452a1-159">Web アプリケーションのクライアント シークレット。</span><span class="sxs-lookup"><span data-stu-id="452a1-159">The web application's client secret.</span></span>
* <span data-ttu-id="452a1-160">`redirectUri`</span><span class="sxs-lookup"><span data-stu-id="452a1-160">`redirectUri`.</span></span> <span data-ttu-id="452a1-161">OpenID 接続用に設定したリダイレクト URI。</span><span class="sxs-lookup"><span data-stu-id="452a1-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="452a1-162">ここに、IDP がトークンでコールバックします。</span><span class="sxs-lookup"><span data-stu-id="452a1-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="452a1-163">`resourceID`</span><span class="sxs-lookup"><span data-stu-id="452a1-163">`resourceID`.</span></span> <span data-ttu-id="452a1-164">Azure AD で Web API を登録するときに作成した URI です。</span><span class="sxs-lookup"><span data-stu-id="452a1-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="452a1-165">`tokenCache`</span><span class="sxs-lookup"><span data-stu-id="452a1-165">`tokenCache`.</span></span> <span data-ttu-id="452a1-166">アクセス トークンをキャッシュするオブジェクト。</span><span class="sxs-lookup"><span data-stu-id="452a1-166">An object that caches the access tokens.</span></span> <span data-ttu-id="452a1-167">[トークンのキャッシュ]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="452a1-167">See [Token caching].</span></span>

<span data-ttu-id="452a1-168">`AcquireTokenByAuthorizationCodeAsync` が成功すると、ADAL はトークンをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="452a1-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="452a1-169">後でキャッシュからトークンを取得するには、次のように AcquireTokenSilentAsync を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="452a1-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="452a1-170">`userId` は、`http://schemas.microsoft.com/identity/claims/objectidentifier` 要求にあるユーザーのオブジェクト ID です。</span><span class="sxs-lookup"><span data-stu-id="452a1-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="452a1-171">アクセス トークンを使用して Web API を呼び出す</span><span class="sxs-lookup"><span data-stu-id="452a1-171">Using the access token to call the web API</span></span>

<span data-ttu-id="452a1-172">トークンを取得したら、HTTP 要求の Authorization ヘッダーで Web API に送信します。</span><span class="sxs-lookup"><span data-stu-id="452a1-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```http
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="452a1-173">Surveys アプリケーションの次の拡張メソッドで、 **HttpClient** クラスを使用して HTTP 要求の Authorization ヘッダーを設定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="452a1-174">Web API で認証する</span><span class="sxs-lookup"><span data-stu-id="452a1-174">Authenticating in the web API</span></span>

<span data-ttu-id="452a1-175">Web API は、ベアラー トークンを認証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="452a1-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="452a1-176">ASP.NET Core では、[Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] パッケージを使用できます。</span><span class="sxs-lookup"><span data-stu-id="452a1-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="452a1-177">このパッケージには、OpenID Connect ベアラー トークンをアプリケーションに渡すことができるミドルウェアが含まれています。</span><span class="sxs-lookup"><span data-stu-id="452a1-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="452a1-178">Web API の `Startup` クラスにミドルウェアを登録します。</span><span class="sxs-lookup"><span data-stu-id="452a1-178">Register the middleware in your web API `Startup` class.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });

    // ...
}
```

* <span data-ttu-id="452a1-179">**Audience**。</span><span class="sxs-lookup"><span data-stu-id="452a1-179">**Audience**.</span></span> <span data-ttu-id="452a1-180">Azure AD に Web API を登録したときに作成した Web API のアプリ ID URL に Audience を設定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="452a1-181">**Authority**。</span><span class="sxs-lookup"><span data-stu-id="452a1-181">**Authority**.</span></span> <span data-ttu-id="452a1-182">マルチテナント アプリケーションの場合、これを `https://login.microsoftonline.com/common/` に設定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="452a1-183">**TokenValidationParameters**。</span><span class="sxs-lookup"><span data-stu-id="452a1-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="452a1-184">マルチテナント アプリケーションの場合、**ValidateIssuer** を false に設定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="452a1-185">つまり、アプリケーションは発行者を検証します。</span><span class="sxs-lookup"><span data-stu-id="452a1-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="452a1-186">**イベント**は、**JwtBearerEvents** から派生するクラスです。</span><span class="sxs-lookup"><span data-stu-id="452a1-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="452a1-187">発行者の検証</span><span class="sxs-lookup"><span data-stu-id="452a1-187">Issuer validation</span></span>

<span data-ttu-id="452a1-188">**JwtBearerEvents.TokenValidated** イベントでトークン発行者を検証します。</span><span class="sxs-lookup"><span data-stu-id="452a1-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="452a1-189">発行者は "iss" 要求で送信されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="452a1-190">Surveys アプリケーションでは、Web API は [テナントのサインアップ]を処理しません。</span><span class="sxs-lookup"><span data-stu-id="452a1-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="452a1-191">そのため、発行者がアプリケーション データベース内に存在するかどうかのみを確認します。</span><span class="sxs-lookup"><span data-stu-id="452a1-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="452a1-192">存在しない場合は例外がスローされ、認証が失敗します。</span><span class="sxs-lookup"><span data-stu-id="452a1-192">If not, it throws an exception, which causes authentication to fail.</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

<span data-ttu-id="452a1-193">この例に示すように、**TokenValidated** イベントを使用して要求を変更することもできます。</span><span class="sxs-lookup"><span data-stu-id="452a1-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="452a1-194">要求は Azure AD から直接送信されることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="452a1-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="452a1-195">Web アプリケーションが取得した要求を変更する場合、それらの変更は Web API が受信するベアラー トークンでは表示されません。</span><span class="sxs-lookup"><span data-stu-id="452a1-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="452a1-196">詳細については、「[Claims transformations (要求の変換)][claims-transformation]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="452a1-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="452a1-197">Authorization</span><span class="sxs-lookup"><span data-stu-id="452a1-197">Authorization</span></span>

<span data-ttu-id="452a1-198">承認の一般的な説明については、「[Role-based and resource-based authorization (ロールベースおよびリソースベースの承認)][Authorization]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="452a1-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span>

<span data-ttu-id="452a1-199">JwtBearer ミドルウェアは、承認応答を処理します。</span><span class="sxs-lookup"><span data-stu-id="452a1-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="452a1-200">たとえば、認証されたユーザーに対するコントローラーのアクションを制御するには、**[承認]** 属性を使用して、認証スキームに **JwtBearerDefaults.AuthenticationScheme** を指定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="452a1-201">ユーザーが認証されていない場合、401 状態コードが返されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="452a1-202">承認ポリシーによってコントローラーのアクションを制限するには、**[承認]** 属性でポリシー名を指定します。</span><span class="sxs-lookup"><span data-stu-id="452a1-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="452a1-203">ユーザーが認証されていない場合は 401 状態コードが、ユーザーが認証されているが承認されていない場合は 403 状態コードが返されます。</span><span class="sxs-lookup"><span data-stu-id="452a1-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="452a1-204">スタートアップ時にポリシーを登録します。</span><span class="sxs-lookup"><span data-stu-id="452a1-204">Register the policy on startup:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });

    // ...
}
```

<span data-ttu-id="452a1-205">[**次へ**][token cache]</span><span class="sxs-lookup"><span data-stu-id="452a1-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer4]: https://github.com/IdentityServer/IdentityServer4
[アプリケーション マニフェストの更新]: ./run-the-app.md#update-the-application-manifests
[Update the application manifests]: ./run-the-app.md#update-the-application-manifests
[トークンのキャッシュ]: token-cache.md
[Token caching]: token-cache.md
[テナントのサインアップ]: signup.md
[tenant sign-up]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
