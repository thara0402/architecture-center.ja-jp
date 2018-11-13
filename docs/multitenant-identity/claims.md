---
title: マルチテナント アプリケーションで要求ベースの ID を操作する
description: 発行者の検証と承認に要求を使用する方法
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 46c43c9bfa4514f206b5e7eabd9223ad4c61628b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429374"
---
# <a name="work-with-claims-based-identities"></a><span data-ttu-id="4ea1b-103">要求ベースの ID を操作する</span><span class="sxs-lookup"><span data-stu-id="4ea1b-103">Work with claims-based identities</span></span>

<span data-ttu-id="4ea1b-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="4ea1b-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="claims-in-azure-ad"></a><span data-ttu-id="4ea1b-105">Azure AD の要求</span><span class="sxs-lookup"><span data-stu-id="4ea1b-105">Claims in Azure AD</span></span>
<span data-ttu-id="4ea1b-106">ユーザーがサインインすると、Azure AD は、ユーザーに関する要求セットを含む ID トークンを送信します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-106">When a user signs in, Azure AD sends an ID token that contains a set of claims about the user.</span></span> <span data-ttu-id="4ea1b-107">要求は、キーと値のペアで表される 1 つの情報です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-107">A claim is simply a piece of information, expressed as a key/value pair.</span></span> <span data-ttu-id="4ea1b-108">たとえば、`email`=`bob@contoso.com` です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-108">For example, `email`=`bob@contoso.com`.</span></span>  <span data-ttu-id="4ea1b-109">要求には、ユーザーを認証し、要求を作成するエンティティである発行者があります (ここでは Azure AD)。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-109">Claims have an issuer &mdash; in this case, Azure AD &mdash; which is the entity that authenticates the user and creates the claims.</span></span> <span data-ttu-id="4ea1b-110">発行者は、ユーザーを認証し、要求を作成するエンティティです。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-110">You trust the claims because you trust the issuer.</span></span> <span data-ttu-id="4ea1b-111">発行者を信頼していれば、要求も信頼することになります(逆に、信頼できない発行者の場合は、要求も信頼しないでください)。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-111">(Conversely, if you don't trust the issuer, don't trust the claims!)</span></span>

<span data-ttu-id="4ea1b-112">概要:</span><span class="sxs-lookup"><span data-stu-id="4ea1b-112">At a high level:</span></span>

1. <span data-ttu-id="4ea1b-113">ユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-113">The user authenticates.</span></span>
2. <span data-ttu-id="4ea1b-114">IDP から 1 セットの要求が送信されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-114">The IDP sends a set of claims.</span></span>
3. <span data-ttu-id="4ea1b-115">アプリで、要求の正規化または強化が行われます (省略可能)。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-115">The app normalizes or augments the claims (optional).</span></span>
4. <span data-ttu-id="4ea1b-116">アプリで、要求を使用して承認が決定されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-116">The app uses the claims to make authorization decisions.</span></span>

<span data-ttu-id="4ea1b-117">OpenID Connect では、取得する要求のセットは、認証要求の[スコープ パラメーター]によって制御されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-117">In OpenID Connect, the set of claims that you get is controlled by the [scope parameter] of the authentication request.</span></span> <span data-ttu-id="4ea1b-118">ただし、Azure AD が OpenID Connect を通して発行する要求のセットは限られています。[サポートされているトークンとクレームの種類]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-118">However, Azure AD issues a limited set of claims through OpenID Connect; see [Supported Token and Claim Types].</span></span> <span data-ttu-id="4ea1b-119">ユーザーの詳細情報を取得するには、Azure AD Graph API を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-119">If you want more information about the user, you'll need to use the Azure AD Graph API.</span></span>

<span data-ttu-id="4ea1b-120">次に、通常はアプリが処理するような AAD の要求を示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-120">Here are some of the claims from AAD that an app might typically care about:</span></span>

| <span data-ttu-id="4ea1b-121">ID トークンの要求の種類</span><span class="sxs-lookup"><span data-stu-id="4ea1b-121">Claim type in ID token</span></span> | <span data-ttu-id="4ea1b-122">説明</span><span class="sxs-lookup"><span data-stu-id="4ea1b-122">Description</span></span> |
| --- | --- |
| <span data-ttu-id="4ea1b-123">aud</span><span class="sxs-lookup"><span data-stu-id="4ea1b-123">aud</span></span> |<span data-ttu-id="4ea1b-124">トークンがだれに対して発行されたか。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-124">Who the token was issued for.</span></span> <span data-ttu-id="4ea1b-125">これは、アプリケーションのクライアント ID になります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-125">This will be the application's client ID.</span></span> <span data-ttu-id="4ea1b-126">一般に、ミドルウェアが自動的に検証するため、この要求について気にする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-126">Generally, you shouldn't need to worry about this claim, because the middleware automatically validates it.</span></span> <span data-ttu-id="4ea1b-127">例: `"91464657-d17a-4327-91f3-2ed99386406f"`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-127">Example:  `"91464657-d17a-4327-91f3-2ed99386406f"`</span></span> |
| <span data-ttu-id="4ea1b-128">groups</span><span class="sxs-lookup"><span data-stu-id="4ea1b-128">groups</span></span> |<span data-ttu-id="4ea1b-129">ユーザーがメンバーである AAD グループの一覧。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-129">A list of AAD groups of which the user is a member.</span></span> <span data-ttu-id="4ea1b-130">例: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-130">Example: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span></span> |
| <span data-ttu-id="4ea1b-131">iss</span><span class="sxs-lookup"><span data-stu-id="4ea1b-131">iss</span></span> |<span data-ttu-id="4ea1b-132">OIDC トークンの [発行者] 。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-132">The [issuer] of the OIDC token.</span></span> <span data-ttu-id="4ea1b-133">例: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-133">Example: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span></span> |
| <span data-ttu-id="4ea1b-134">name</span><span class="sxs-lookup"><span data-stu-id="4ea1b-134">name</span></span> |<span data-ttu-id="4ea1b-135">ユーザーの表示名。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-135">The user's display name.</span></span> <span data-ttu-id="4ea1b-136">例: `"Alice A."`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-136">Example: `"Alice A."`</span></span> |
| <span data-ttu-id="4ea1b-137">oid</span><span class="sxs-lookup"><span data-stu-id="4ea1b-137">oid</span></span> |<span data-ttu-id="4ea1b-138">AAD でのユーザーのオブジェクト識別子。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-138">The object identifier for the user in AAD.</span></span> <span data-ttu-id="4ea1b-139">この値は、ユーザーの変更不能な再利用できない識別子です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-139">This value is the immutable and non-reusable identifier of the user.</span></span> <span data-ttu-id="4ea1b-140">ユーザーの一意識別子として、電子メールではなく、この値を使用します。電子メール アドレスは変更される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-140">Use this value, not email, as a unique identifier for users; email addresses can change.</span></span> <span data-ttu-id="4ea1b-141">アプリで Azure AD Graph API を使用する場合は、オブジェクト ID がプロファイル情報のクエリを実行するために使用される値になります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-141">If you use the Azure AD Graph API in your app, object ID is that value used to query profile information.</span></span> <span data-ttu-id="4ea1b-142">例: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-142">Example: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span></span> |
| <span data-ttu-id="4ea1b-143">roles</span><span class="sxs-lookup"><span data-stu-id="4ea1b-143">roles</span></span> |<span data-ttu-id="4ea1b-144">ユーザーのアプリ ロールの一覧。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-144">A list of app roles for the user.</span></span>    <span data-ttu-id="4ea1b-145">例: `["SurveyCreator"]`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-145">Example: `["SurveyCreator"]`</span></span> |
| <span data-ttu-id="4ea1b-146">tid</span><span class="sxs-lookup"><span data-stu-id="4ea1b-146">tid</span></span> |<span data-ttu-id="4ea1b-147">テナント ID。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-147">Tenant ID.</span></span> <span data-ttu-id="4ea1b-148">この値は、Azure AD での テナントの一意識別子です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-148">This value is a unique identifier for the tenant in Azure AD.</span></span> <span data-ttu-id="4ea1b-149">例: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-149">Example: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span></span> |
| <span data-ttu-id="4ea1b-150">unique_name</span><span class="sxs-lookup"><span data-stu-id="4ea1b-150">unique_name</span></span> |<span data-ttu-id="4ea1b-151">人間が読み取り可能なユーザーの表示名。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-151">A human readable display name of the user.</span></span> <span data-ttu-id="4ea1b-152">例: `"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-152">Example: `"alice@contoso.com"`</span></span> |
| <span data-ttu-id="4ea1b-153">upn</span><span class="sxs-lookup"><span data-stu-id="4ea1b-153">upn</span></span> |<span data-ttu-id="4ea1b-154">ユーザー プリンシパル名。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-154">User principal name.</span></span> <span data-ttu-id="4ea1b-155">例: `"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-155">Example: `"alice@contoso.com"`</span></span> |

<span data-ttu-id="4ea1b-156">この表は、ID トークンに表示される要求の種類の一覧です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-156">This table lists the claim types as they appear in the ID token.</span></span> <span data-ttu-id="4ea1b-157">ASP.NET Coreでは、OpenID Connect ミドルウェアがユーザー プリンシパルの Claims コレクションを設定するときに、一部の要求の種類を変換します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-157">In ASP.NET Core, the OpenID Connect middleware converts some of the claim types when it populates the Claims collection for the user principal:</span></span>

* <span data-ttu-id="4ea1b-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span></span>
* <span data-ttu-id="4ea1b-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span></span>
* <span data-ttu-id="4ea1b-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span></span>
* <span data-ttu-id="4ea1b-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span><span class="sxs-lookup"><span data-stu-id="4ea1b-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span></span>

## <a name="claims-transformations"></a><span data-ttu-id="4ea1b-162">要求の変換</span><span class="sxs-lookup"><span data-stu-id="4ea1b-162">Claims transformations</span></span>
<span data-ttu-id="4ea1b-163">認証フロー中に、IDP から取得する要求を変更できます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-163">During the authentication flow, you might want to modify the claims that you get from the IDP.</span></span> <span data-ttu-id="4ea1b-164">ASP.NET Core では、OpenID Connect ミドルウェアからの **AuthenticationValidated** イベントの内部で要求変換を実行できます </span><span class="sxs-lookup"><span data-stu-id="4ea1b-164">In ASP.NET Core, you can perform claims transformation inside of the **AuthenticationValidated** event from the OpenID Connect middleware.</span></span> <span data-ttu-id="4ea1b-165">(「[認証イベント]」を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-165">(See [Authentication events].)</span></span>

<span data-ttu-id="4ea1b-166">**AuthenticationValidated** 中に追加された要求は、セッション認証 Cookie に保存されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-166">Any claims that you add during **AuthenticationValidated** are stored in the session authentication cookie.</span></span> <span data-ttu-id="4ea1b-167">Azure AD にプッシュバックされることはありません。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-167">They don't get pushed back to Azure AD.</span></span>

<span data-ttu-id="4ea1b-168">次に、要求変換の例をいくつか示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-168">Here are some examples of claims transformation:</span></span>

* <span data-ttu-id="4ea1b-169">**要求の正規化**またはユーザー間で一貫性のある要求にする。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-169">**Claims normalization**, or making claims consistent across users.</span></span> <span data-ttu-id="4ea1b-170">これが特に関係するのは、複数の IDP から要求を取得している場合であり、類似する情報に対して異なる要求の種類が使用されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-170">This is particularly relevant if you are getting claims from multiple IDPs, which might use different claim types for similar information.</span></span>
  <span data-ttu-id="4ea1b-171">たとえば、Azure AD は、ユーザーの電子メール アドレスを含む "upn" 要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-171">For example, Azure AD sends a "upn" claim that contains the user's email.</span></span> <span data-ttu-id="4ea1b-172">その他の IDP は、"email" 要求を送信することがあります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-172">Other IDPs might send an "email" claim.</span></span> <span data-ttu-id="4ea1b-173">次のコードは、"upn" 要求を "email" 要求に変換します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-173">The following code converts the "upn" claim into an "email" claim:</span></span>
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* <span data-ttu-id="4ea1b-174">存在しない要求のための**既定の要求値**を追加します。たとえば、ユーザーに既定のロールを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-174">Add **default claim values** for claims that aren't present &mdash; for example, assigning a user to a default role.</span></span> <span data-ttu-id="4ea1b-175">これにより、承認ロジックを簡略化できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-175">In some cases this can simplify authorization logic.</span></span>
* <span data-ttu-id="4ea1b-176">ユーザーのアプリケーション固有の情報を使用して **カスタム要求の種類** を追加します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-176">Add **custom claim types** with application-specific information about the user.</span></span> <span data-ttu-id="4ea1b-177">たとえば、データベース内のユーザーに関する一部の情報を保存することができます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-177">For example, you might store some information about the user in a database.</span></span> <span data-ttu-id="4ea1b-178">この情報を含むカスタム要求を認証チケットに追加できます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-178">You could add a custom claim with this information to the authentication ticket.</span></span> <span data-ttu-id="4ea1b-179">要求は Cookie に保存されるので、ログイン セッションごとに 1 度のみ、データベースから取得する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-179">The claim is stored in a cookie, so you only need to get it from the database once per login session.</span></span> <span data-ttu-id="4ea1b-180">一方で、過度に大きな Cookie を作成することを回避するには、Cookie のサイズとデータベース検索の妥協点を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-180">On the other hand, you also want to avoid creating excessively large cookies, so you need to consider the trade-off between cookie size versus database lookups.</span></span>   

<span data-ttu-id="4ea1b-181">認証フローが完了したら、`HttpContext.User` で要求を使用できます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-181">After the authentication flow is complete, the claims are available in `HttpContext.User`.</span></span> <span data-ttu-id="4ea1b-182">その時点で、要求は、読み取り専用コレクションとして処理する必要があります。たとえば、承認に関する決定を行うために使用します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-182">At that point, you should treat them as a read-only collection &mdash; e.g., use them to make authorization decisions.</span></span>

## <a name="issuer-validation"></a><span data-ttu-id="4ea1b-183">発行者の検証</span><span class="sxs-lookup"><span data-stu-id="4ea1b-183">Issuer validation</span></span>
<span data-ttu-id="4ea1b-184">OpenID Connect の発行者要求 ("iss") によって、ID トークンを発行した IDP が識別されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-184">In OpenID Connect, the issuer claim ("iss") identifies the IDP that issued the ID token.</span></span> <span data-ttu-id="4ea1b-185">OIDC 認証フローの一部では、発行者要求が実際の発行者と一致することが検証されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-185">Part of the OIDC authentication flow is to verify that the issuer claim matches the actual issuer.</span></span> <span data-ttu-id="4ea1b-186">この処理は、OIDC ミドルウェアが自動実行します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-186">The OIDC middleware handles this for you.</span></span>

<span data-ttu-id="4ea1b-187">Azure AD で発行者値は AD テナントごとに一意です (`https://sts.windows.net/<tenantID>`)。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-187">In Azure AD, the issuer value is unique per AD tenant (`https://sts.windows.net/<tenantID>`).</span></span> <span data-ttu-id="4ea1b-188">そのため、発行者がアプリにサインインできるテナントであることを確認するには、アプリケーションで追加チェックを実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-188">Therefore, an application should do an additional check, to make sure the issuer represents a tenant that is allowed to sign in to the app.</span></span>

<span data-ttu-id="4ea1b-189">シングルテナント アプリケーションの場合、発行者が自分のテナントであることを確認するだけです。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-189">For a single-tenant application, you can just check that the issuer is your own tenant.</span></span> <span data-ttu-id="4ea1b-190">実際の処理では、既定で OIDC ミドルウェアが自動的に実行します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-190">In fact, the OIDC middleware does this automatically by default.</span></span> <span data-ttu-id="4ea1b-191">マルチテナント アプリの場合、異なるテナントに対応する複数の発行者を許容する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-191">In a multi-tenant app, you need to allow for multiple issuers, corresponding to the different tenants.</span></span> <span data-ttu-id="4ea1b-192">次に、利用できる一般的なアプローチを示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-192">Here is a general approach to use:</span></span>

* <span data-ttu-id="4ea1b-193">OIDC ミドルウェア オプションで、 **ValidateIssuer** を false に設定します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-193">In the OIDC middleware options, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="4ea1b-194">これで自動チェックが無効になります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-194">This turns off the automatic check.</span></span>
* <span data-ttu-id="4ea1b-195">テナントがサインアップしたら、テナント発行者をユーザー DB に保存します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-195">When a tenant signs up, store the tenant and the issuer in your user DB.</span></span>
* <span data-ttu-id="4ea1b-196">ユーザーがサインインするたびに、データベース内の発行者を検索します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-196">Whenever a user signs in, look up the issuer in the database.</span></span> <span data-ttu-id="4ea1b-197">発行元が見つからない場合、そのテナントがサインアップしていないことを意味します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-197">If the issuer isn't found, it means that tenant hasn't signed up.</span></span> <span data-ttu-id="4ea1b-198">このような場合、サインアップ ページにリダイレクトすることができます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-198">You can redirect them to a sign up page.</span></span>
* <span data-ttu-id="4ea1b-199">また、たとえばサブスクリプション料金を支払わない顧客など、特定のテナントをブラックリストに登録することもできます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-199">You could also blacklist certain tenants; for example, for customers that didn't pay their subscription.</span></span>

<span data-ttu-id="4ea1b-200">詳細については、[マルチテナント アプリケーションでのサインアップとテナントのオンボード][signup]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-200">For a more detailed discussion, see [Sign-up and tenant onboarding in a multitenant application][signup].</span></span>

## <a name="using-claims-for-authorization"></a><span data-ttu-id="4ea1b-201">承認の要求を使用する</span><span class="sxs-lookup"><span data-stu-id="4ea1b-201">Using claims for authorization</span></span>
<span data-ttu-id="4ea1b-202">要求では、ユーザーの ID は、一枚岩のエンティティではなくなりました。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-202">With claims, a user's identity is no longer a monolithic entity.</span></span> <span data-ttu-id="4ea1b-203">たとえば、ユーザーは、電子メール アドレス、電話番号、誕生日、性別などを持つ場合があります。ユーザーの IDP には、これらの情報がすべて格納されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-203">For example, a user might have an email address, phone number, birthday, gender, etc. Maybe the user's IDP stores all of this information.</span></span> <span data-ttu-id="4ea1b-204">ただし、ユーザーを認証するときは、通常はこれらの情報のサブセットを要求として取得します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-204">But when you authenticate the user, you'll typically get a subset of these as claims.</span></span> <span data-ttu-id="4ea1b-205">このモデルでは、ユーザーの ID は、単なる要求の束です。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-205">In this model, the user's identity is simply a bundle of claims.</span></span> <span data-ttu-id="4ea1b-206">ユーザーに関する承認の決定を行うときは、要求の特定のセットを探します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-206">When you make authorization decisions about a user, you will look for particular sets of claims.</span></span> <span data-ttu-id="4ea1b-207">つまり、「ユーザー X はアクション Y を実行できるか」という質問は、最終的には「ユーザー X は要求 Z を所有しているか」ということになります。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-207">In other words, the question "Can user X perform action Y" ultimately becomes "Does user X have claim Z".</span></span>

<span data-ttu-id="4ea1b-208">要求を要求する基本的なパターンを次に示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-208">Here are some basic patterns for checking claims.</span></span>

* <span data-ttu-id="4ea1b-209">特定の値が設定された特定の要求を持つユーザーを確認する例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-209">To check that the user has a particular claim with a particular value:</span></span>
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   <span data-ttu-id="4ea1b-210">このコードでは、ユーザーが "Admin" 値の Role 要求を持っているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-210">This code checks whether the user has a Role claim with the value "Admin".</span></span> <span data-ttu-id="4ea1b-211">ユーザーが Role 要求を持っていない場合、または複数の Role 要求を持っている場合も正しく処理されます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-211">It correctly handles the case where the user has no Role claim or multiple Role claims.</span></span>
  
   <span data-ttu-id="4ea1b-212">**ClaimTypes** クラスでは、一般的に使用される要求の種類に対して定数を定義します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-212">The **ClaimTypes** class defines constants for commonly-used claim types.</span></span> <span data-ttu-id="4ea1b-213">ただし、要求の種類には任意の文字列値を使用できます。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-213">However, you can use any string value for the claim type.</span></span>
* <span data-ttu-id="4ea1b-214">最大で 1 つの値があると想定される要求の種類の場合、1 つの値を取得する例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-214">To get a single value for a claim type, when you expect there to be at most one value:</span></span>
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* <span data-ttu-id="4ea1b-215">要求の種類のすべての値を取得する例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-215">To get all the values for a claim type:</span></span>
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

<span data-ttu-id="4ea1b-216">詳細については、[マルチテナント アプリケーションでのロールベースおよびリソースベースの承認][authorization]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4ea1b-216">For more information, see [Role-based and resource-based authorization in multitenant applications][authorization].</span></span>

<span data-ttu-id="4ea1b-217">[**次へ**][signup]</span><span class="sxs-lookup"><span data-stu-id="4ea1b-217">[**Next**][signup]</span></span>


<!-- Links -->

[スコープ パラメーター]: https://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[scope parameter]: https://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[サポートされているトークンとクレームの種類]: /azure/active-directory/active-directory-token-and-claims/
[Supported Token and Claim Types]: /azure/active-directory/active-directory-token-and-claims/
[発行者]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[issuer]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[認証イベント]: authenticate.md#authentication-events
[Authentication events]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
