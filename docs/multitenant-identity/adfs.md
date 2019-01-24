---
title: 顧客の AD FS とのフェデレーション
description: マルチテナント アプリケーションで顧客の AD FS とのフェデレーションを行う方法。
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 82b39d77f1ee9af4063c4715a4688ef4b69bc477
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487859"
---
# <a name="federate-with-a-customers-ad-fs"></a><span data-ttu-id="f2248-103">顧客の AD FS とのフェデレーション</span><span class="sxs-lookup"><span data-stu-id="f2248-103">Federate with a customer's AD FS</span></span>

<span data-ttu-id="f2248-104">この記事では、顧客の AD FS とのフェデレーションを行うために、マルチテナント SaaS アプリケーションで Active Directory フェデレーション サービス (AD FS) を使用して認証をサポートする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="f2248-104">This article describes how a multi-tenant SaaS application can support authentication via Active Directory Federation Services (AD FS), in order to federate with a customer's AD FS.</span></span>

## <a name="overview"></a><span data-ttu-id="f2248-105">概要</span><span class="sxs-lookup"><span data-stu-id="f2248-105">Overview</span></span>

<span data-ttu-id="f2248-106">Azure Active Directory (Azure AD) を使用すると、Office365 や Dynamics CRM Online の顧客を含め、Azure AD テナントのユーザーによるサインインが簡単になります。</span><span class="sxs-lookup"><span data-stu-id="f2248-106">Azure Active Directory (Azure AD) makes it easy to sign in users from Azure AD tenants, including Office365 and Dynamics CRM Online customers.</span></span> <span data-ttu-id="f2248-107">では、企業のイントラネットでオンプレミスの Active Directory を使用する顧客についてはどうでしょうか。</span><span class="sxs-lookup"><span data-stu-id="f2248-107">But what about customers who use on-premise Active Directory on a corporate intranet?</span></span>

<span data-ttu-id="f2248-108">このような顧客には、 [Azure AD Connect]を使用してオンプレミスの AD と Azure AD を同期するという方法があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-108">One option is for these customers to sync their on-premise AD with Azure AD, using [Azure AD Connect].</span></span> <span data-ttu-id="f2248-109">ただし、企業の IT ポリシーやその他の理由により、この方法を使用できない顧客も存在します。</span><span class="sxs-lookup"><span data-stu-id="f2248-109">However, some customers may be unable to use this approach, due to corporate IT policy or other reasons.</span></span> <span data-ttu-id="f2248-110">そのような場合には、Active Directory フェデレーション サービス (AD FS) を使用してフェデレーションを行う方法もあります。</span><span class="sxs-lookup"><span data-stu-id="f2248-110">In that case, another option is to federate through Active Directory Federation Services (AD FS).</span></span>

<span data-ttu-id="f2248-111">このシナリオを実施するには:</span><span class="sxs-lookup"><span data-stu-id="f2248-111">To enable this scenario:</span></span>

* <span data-ttu-id="f2248-112">顧客がインターネットに接続された AD FS ファームを持っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-112">The customer must have an Internet-facing AD FS farm.</span></span>
* <span data-ttu-id="f2248-113">SaaS プロバイダーが独自の AD FS ファームをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="f2248-113">The SaaS provider deploys their own AD FS farm.</span></span>
* <span data-ttu-id="f2248-114">顧客と SaaS プロバイダーは、 [フェデレーションの信頼]を設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-114">The customer and the SaaS provider must set up [federation trust].</span></span> <span data-ttu-id="f2248-115">この設定は手動で行います。</span><span class="sxs-lookup"><span data-stu-id="f2248-115">This is a manual process.</span></span>

<span data-ttu-id="f2248-116">信頼関係には、3 つの主要なロールがあります。</span><span class="sxs-lookup"><span data-stu-id="f2248-116">There are three main roles in the trust relation:</span></span>

* <span data-ttu-id="f2248-117">顧客の AD FS は、顧客の AD からのユーザーを認証し、ユーザー要求に応じてセキュリティ トークンを作成する[アカウント パートナー]です。</span><span class="sxs-lookup"><span data-stu-id="f2248-117">The customer's AD FS is the [account partner], responsible for authenticating users from the customer's AD, and creating security tokens with user claims.</span></span>
* <span data-ttu-id="f2248-118">SaaS プロバイダーの AD FS は、アカウント パートナーを信頼し、ユーザー要求を受信する[リソース パートナー]です。</span><span class="sxs-lookup"><span data-stu-id="f2248-118">The SaaS provider's AD FS is the [resource partner], which trusts the account partner and receives the user claims.</span></span>
* <span data-ttu-id="f2248-119">アプリケーションは、SaaS プロバイダーの AD FS で証明書利用者 (RP) として構成されます。</span><span class="sxs-lookup"><span data-stu-id="f2248-119">The application is configured as a relying party (RP) in the SaaS provider's AD FS.</span></span>
  
  ![フェデレーションの信頼](./images/federation-trust.png)

> [!NOTE]
> <span data-ttu-id="f2248-121">この記事では、アプリケーションで認証プロトコルとして OpenID Connect を使用することを想定しています。</span><span class="sxs-lookup"><span data-stu-id="f2248-121">In this article, we assume the application uses OpenID connect as the authentication protocol.</span></span> <span data-ttu-id="f2248-122">そのほか、WS-Federation を使用する方法もあります。</span><span class="sxs-lookup"><span data-stu-id="f2248-122">Another option is to use WS-Federation.</span></span>
>
> <span data-ttu-id="f2248-123">OpenID Connect を使用する場合、SaaS プロバイダーは、Windows Server 2016 で実行される AD FS 2016 を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-123">For OpenID Connect, the SaaS provider must use AD FS 2016, running in Windows Server 2016.</span></span> <span data-ttu-id="f2248-124">AD FS 3.0 は、OpenID Connect をサポートしていません。</span><span class="sxs-lookup"><span data-stu-id="f2248-124">AD FS 3.0 does not support OpenID Connect.</span></span>
>
> <span data-ttu-id="f2248-125">ASP.NET Core には、すぐに使用できる WS-Federation のサポートは含まれていません。</span><span class="sxs-lookup"><span data-stu-id="f2248-125">ASP.NET Core does not include out-of-the-box support for WS-Federation.</span></span>
>
>

<span data-ttu-id="f2248-126">ASP.NET 4 による WS-Federation の使用例については、[active-directory-dotnet-webapp-wsfederation サンプル][active-directory-dotnet-webapp-wsfederation]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f2248-126">For an example of using WS-Federation with ASP.NET 4, see the [active-directory-dotnet-webapp-wsfederation sample][active-directory-dotnet-webapp-wsfederation].</span></span>

## <a name="authentication-flow"></a><span data-ttu-id="f2248-127">Authentication flow</span><span class="sxs-lookup"><span data-stu-id="f2248-127">Authentication flow</span></span>

1. <span data-ttu-id="f2248-128">ユーザーが [サインイン] をクリックすると、アプリケーションにより、SaaS プロバイダーの AD FS の OpenID Connect エンドポイントにリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="f2248-128">When the user clicks "sign in", the application redirects to an OpenID Connect endpoint on the SaaS provider's AD FS.</span></span>
2. <span data-ttu-id="f2248-129">ユーザーが組織のユーザー名 ("`alice@corp.contoso.com`") を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-129">The user enters his or her organizational user name ("`alice@corp.contoso.com`").</span></span> <span data-ttu-id="f2248-130">AD FS がホーム領域検出を使用して顧客の AD FS (ユーザーが資格情報を入力する場所) にリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="f2248-130">AD FS uses home realm discovery to redirect to the customer's AD FS, where the user enters their credentials.</span></span>
3. <span data-ttu-id="f2248-131">顧客の AD FS が WF-Federation (または SAML) を使用して SaaS プロバイダーの AD FS にユーザー要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="f2248-131">The customer's AD FS sends user claims to the SaaS provider's AD FS, using WF-Federation (or SAML).</span></span>
4. <span data-ttu-id="f2248-132">要求は、OpenID Connect を使用して、AD FS からアプリケーションに渡されます。</span><span class="sxs-lookup"><span data-stu-id="f2248-132">Claims flow from AD FS to the app, using OpenID Connect.</span></span> <span data-ttu-id="f2248-133">これには、WS-Federation からのプロトコルの切り替えが必要です。</span><span class="sxs-lookup"><span data-stu-id="f2248-133">This requires a protocol transition from WS-Federation.</span></span>

## <a name="limitations"></a><span data-ttu-id="f2248-134">制限事項</span><span class="sxs-lookup"><span data-stu-id="f2248-134">Limitations</span></span>

<span data-ttu-id="f2248-135">証明書利用者アプリケーションは、次の表に示すように、既定では id_token で利用可能な要求の固定セットだけを受け取ります。</span><span class="sxs-lookup"><span data-stu-id="f2248-135">By default, the relying party application receives only a fixed set of claims available in the id_token, shown in the following table.</span></span> <span data-ttu-id="f2248-136">AD FS 2016 では、OpenID Connect のシナリオで id_token をカスタマイズできます。</span><span class="sxs-lookup"><span data-stu-id="f2248-136">With AD FS 2016, you can customize the id_token in OpenID Connect scenarios.</span></span> <span data-ttu-id="f2248-137">詳細については、「[Custom ID Tokens in AD FS (AD FS でのカスタム ID トークン)](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f2248-137">For more information, see [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span></span>

| <span data-ttu-id="f2248-138">要求</span><span class="sxs-lookup"><span data-stu-id="f2248-138">Claim</span></span> | <span data-ttu-id="f2248-139">説明</span><span class="sxs-lookup"><span data-stu-id="f2248-139">Description</span></span> |
| --- | --- |
| <span data-ttu-id="f2248-140">aud</span><span class="sxs-lookup"><span data-stu-id="f2248-140">aud</span></span> |<span data-ttu-id="f2248-141">対象。</span><span class="sxs-lookup"><span data-stu-id="f2248-141">Audience.</span></span> <span data-ttu-id="f2248-142">要求の発行先のアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="f2248-142">The application for which the claims were issued.</span></span> |
| <span data-ttu-id="f2248-143">authenticationinstant</span><span class="sxs-lookup"><span data-stu-id="f2248-143">authenticationinstant</span></span> |<span data-ttu-id="f2248-144">[認証インスタント]。</span><span class="sxs-lookup"><span data-stu-id="f2248-144">[Authentication instant].</span></span> <span data-ttu-id="f2248-145">認証が発生した時刻。</span><span class="sxs-lookup"><span data-stu-id="f2248-145">The time at which authentication occurred.</span></span> |
| <span data-ttu-id="f2248-146">c_hash</span><span class="sxs-lookup"><span data-stu-id="f2248-146">c_hash</span></span> |<span data-ttu-id="f2248-147">コード ハッシュ値。</span><span class="sxs-lookup"><span data-stu-id="f2248-147">Code hash value.</span></span> <span data-ttu-id="f2248-148">これは、トークンの内容のハッシュです。</span><span class="sxs-lookup"><span data-stu-id="f2248-148">This is a hash of the token contents.</span></span> |
| <span data-ttu-id="f2248-149">exp</span><span class="sxs-lookup"><span data-stu-id="f2248-149">exp</span></span> |<span data-ttu-id="f2248-150">[期限切れ日時]。</span><span class="sxs-lookup"><span data-stu-id="f2248-150">[Expiration time].</span></span> <span data-ttu-id="f2248-151">この時刻の後、トークンは受け入れられなくなります。</span><span class="sxs-lookup"><span data-stu-id="f2248-151">The time after which the token will no longer be accepted.</span></span> |
| <span data-ttu-id="f2248-152">iat</span><span class="sxs-lookup"><span data-stu-id="f2248-152">iat</span></span> |<span data-ttu-id="f2248-153">発行時刻。</span><span class="sxs-lookup"><span data-stu-id="f2248-153">Issued at.</span></span> <span data-ttu-id="f2248-154">トークンが発行された日時。</span><span class="sxs-lookup"><span data-stu-id="f2248-154">The time when the token was issued.</span></span> |
| <span data-ttu-id="f2248-155">iss</span><span class="sxs-lookup"><span data-stu-id="f2248-155">iss</span></span> |<span data-ttu-id="f2248-156">発行者。</span><span class="sxs-lookup"><span data-stu-id="f2248-156">Issuer.</span></span> <span data-ttu-id="f2248-157">この要求の値は、常にリソース パートナーの AD FS です。</span><span class="sxs-lookup"><span data-stu-id="f2248-157">The value of this claim is always the resource partner's AD FS.</span></span> |
| <span data-ttu-id="f2248-158">name</span><span class="sxs-lookup"><span data-stu-id="f2248-158">name</span></span> |<span data-ttu-id="f2248-159">ユーザー名。</span><span class="sxs-lookup"><span data-stu-id="f2248-159">User name.</span></span> <span data-ttu-id="f2248-160">例: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="f2248-160">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="f2248-161">nameidentifier</span><span class="sxs-lookup"><span data-stu-id="f2248-161">nameidentifier</span></span> |<span data-ttu-id="f2248-162">[名前の識別子]。</span><span class="sxs-lookup"><span data-stu-id="f2248-162">[Name identifier].</span></span> <span data-ttu-id="f2248-163">トークンが発行されたエンティティの名前の識別子。</span><span class="sxs-lookup"><span data-stu-id="f2248-163">The identifier for the name of the entity for which the token was issued.</span></span> |
| <span data-ttu-id="f2248-164">nonce</span><span class="sxs-lookup"><span data-stu-id="f2248-164">nonce</span></span> |<span data-ttu-id="f2248-165">セッション nonce。</span><span class="sxs-lookup"><span data-stu-id="f2248-165">Session nonce.</span></span> <span data-ttu-id="f2248-166">再生攻撃を防ぐために AD FS によって生成される一意の値。</span><span class="sxs-lookup"><span data-stu-id="f2248-166">A unique value generated by AD FS to help prevent replay attacks.</span></span> |
| <span data-ttu-id="f2248-167">upn</span><span class="sxs-lookup"><span data-stu-id="f2248-167">upn</span></span> |<span data-ttu-id="f2248-168">ユーザー プリンシパル名 (UPN)。</span><span class="sxs-lookup"><span data-stu-id="f2248-168">User principal name (UPN).</span></span> <span data-ttu-id="f2248-169">例: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="f2248-169">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="f2248-170">pwd_exp</span><span class="sxs-lookup"><span data-stu-id="f2248-170">pwd_exp</span></span> |<span data-ttu-id="f2248-171">パスワードの有効期限。</span><span class="sxs-lookup"><span data-stu-id="f2248-171">Password expiration period.</span></span> <span data-ttu-id="f2248-172">ユーザーのパスワード、または PIN などのような認証のシークレットが期限切れに</span><span class="sxs-lookup"><span data-stu-id="f2248-172">The number of seconds until the user's password or a similar authentication secret, such as a PIN.</span></span> <span data-ttu-id="f2248-173">なるまでの秒数。</span><span class="sxs-lookup"><span data-stu-id="f2248-173">expires.</span></span> |

> [!NOTE]
> <span data-ttu-id="f2248-174">"Iss" 要求には、パートナーの AD FS が含まれています (通常、この要求は SaaS プロバイダーを発行者として識別します)。</span><span class="sxs-lookup"><span data-stu-id="f2248-174">The "iss" claim contains the AD FS of the partner (typically, this claim will identify the SaaS provider as the issuer).</span></span> <span data-ttu-id="f2248-175">顧客の AD FS は識別しません。</span><span class="sxs-lookup"><span data-stu-id="f2248-175">It does not identify the customer's AD FS.</span></span> <span data-ttu-id="f2248-176">UPN の一部として、顧客のドメインを見つけることができます。</span><span class="sxs-lookup"><span data-stu-id="f2248-176">You can find the customer's domain as part of the UPN.</span></span>

<span data-ttu-id="f2248-177">この記事の残りの部分で、RP (アプリケーション) とアカウント パートナー (顧客) の間に信頼関係を設定する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="f2248-177">The rest of this article describes how to set up the trust relationship between the RP (the app) and the account partner (the customer).</span></span>

## <a name="ad-fs-deployment"></a><span data-ttu-id="f2248-178">AD FS のデプロイ</span><span class="sxs-lookup"><span data-stu-id="f2248-178">AD FS deployment</span></span>

<span data-ttu-id="f2248-179">SaaS プロバイダーは、オンプレミスまたは Azure VM に AD FS をデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="f2248-179">The SaaS provider can deploy AD FS either on-premise or on Azure VMs.</span></span> <span data-ttu-id="f2248-180">セキュリティと可用性を確保するうえで、次のガイドラインが重要です。</span><span class="sxs-lookup"><span data-stu-id="f2248-180">For security and availability, the following guidelines are important:</span></span>

* <span data-ttu-id="f2248-181">AD FS サービスの可用性を最大限高めるために、AD FS サーバーと AD FS プロキシ サーバーを少なくとも 2 つずつデプロイします。</span><span class="sxs-lookup"><span data-stu-id="f2248-181">Deploy at least two AD FS servers and two AD FS proxy servers to achieve the best availability of the AD FS service.</span></span>
* <span data-ttu-id="f2248-182">ドメイン コントローラーと AD FS サーバーは直接インターネットに公開せず、これらに直接アクセスできる仮想ネットワーク内に設置する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-182">Domain controllers and AD FS servers should never be exposed directly to the Internet and should be in a virtual network with direct access to them.</span></span>
* <span data-ttu-id="f2248-183">Web アプリケーション プロキシ (以前の AD FS プロキシ) を使用して、インターネットに AD FS サーバーを公開する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-183">Web application proxies (previously AD FS proxies) must be used to publish AD FS servers to the Internet.</span></span>

<span data-ttu-id="f2248-184">このようなトポロジを Azure で設定するには、仮想ネットワーク、NSG、Azure VM、可用性セットを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-184">To set up a similar topology in Azure requires the use of Virtual networks, NSG’s, azure VM’s and availability sets.</span></span> <span data-ttu-id="f2248-185">詳細については、「 [Azure の仮想マシンでの Windows Server Active Directory のデプロイ ガイドライン][active-directory-on-azure]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f2248-185">For more details, see [Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][active-directory-on-azure].</span></span>

## <a name="configure-openid-connect-authentication-with-ad-fs"></a><span data-ttu-id="f2248-186">AD FS で OpenID Connect 認証を構成する</span><span class="sxs-lookup"><span data-stu-id="f2248-186">Configure OpenID Connect authentication with AD FS</span></span>

<span data-ttu-id="f2248-187">SaaS プロバイダーは、アプリケーションと AD FS の間の OpenID Connect を有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-187">The SaaS provider must enable OpenID Connect between the application and AD FS.</span></span> <span data-ttu-id="f2248-188">これを行うには、AD FS でアプリケーション グループを追加します。</span><span class="sxs-lookup"><span data-stu-id="f2248-188">To do so, add an application group in AD FS.</span></span>  <span data-ttu-id="f2248-189">詳細な手順については、この[ブログ記事]の「Setting up a Web App for OpenId Connect sign in AD FS (AD FS での OpenId Connect サインイン用 Web アプリの設定)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="f2248-189">You can find detailed instructions in this [blog post], under " Setting up a Web App for OpenId Connect sign in AD FS."</span></span>

<span data-ttu-id="f2248-190">次に、OpenID Connect ミドルウェアを構成します。</span><span class="sxs-lookup"><span data-stu-id="f2248-190">Next, configure the OpenID Connect middleware.</span></span> <span data-ttu-id="f2248-191">メタデータ エンドポイントは `https://domain/adfs/.well-known/openid-configuration` であり、ここでのドメインは SaaS プロバイダーの AD FS ドメインです。</span><span class="sxs-lookup"><span data-stu-id="f2248-191">The metadata endpoint is `https://domain/adfs/.well-known/openid-configuration`, where domain is the SaaS provider's AD FS domain.</span></span>

<span data-ttu-id="f2248-192">通常、このメタデータ エンドポイントとその他の OpenID Connect エンドポイント (AAD など) を組み合わせます。</span><span class="sxs-lookup"><span data-stu-id="f2248-192">Typically you might combine this with other OpenID Connect endpoints (such as AAD).</span></span> <span data-ttu-id="f2248-193">ユーザーが正しい認証エンドポイントに送られるように、2 つの異なるサインイン ボタンを使用するか、他の方法で 2 つのエンドポイントを区別する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-193">You'll need two different sign-in buttons or some other way to distinguish them, so that the user is sent to the correct authentication endpoint.</span></span>

## <a name="configure-the-ad-fs-resource-partner"></a><span data-ttu-id="f2248-194">AD FS リソース パートナーの構成</span><span class="sxs-lookup"><span data-stu-id="f2248-194">Configure the AD FS Resource Partner</span></span>

<span data-ttu-id="f2248-195">SaaS プロバイダーは、ADFS 経由で接続する顧客ごとに、次の操作を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-195">The SaaS provider must do the following for each customer that wants to connect via ADFS:</span></span>

1. <span data-ttu-id="f2248-196">要求プロバイダー信頼を追加します。</span><span class="sxs-lookup"><span data-stu-id="f2248-196">Add a claims provider trust.</span></span>
2. <span data-ttu-id="f2248-197">要求規則を追加します。</span><span class="sxs-lookup"><span data-stu-id="f2248-197">Add claims rules.</span></span>
3. <span data-ttu-id="f2248-198">ホーム領域検出を有効にします。</span><span class="sxs-lookup"><span data-stu-id="f2248-198">Enable home-realm discovery.</span></span>

<span data-ttu-id="f2248-199">詳細な手順を以下に示します。</span><span class="sxs-lookup"><span data-stu-id="f2248-199">Here are the steps in more detail.</span></span>

### <a name="add-the-claims-provider-trust"></a><span data-ttu-id="f2248-200">要求プロバイダー信頼の追加</span><span class="sxs-lookup"><span data-stu-id="f2248-200">Add the claims provider trust</span></span>

1. <span data-ttu-id="f2248-201">サーバー マネージャーで、**[ツール]** をクリックし、次に **[AD FS の管理]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-201">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="f2248-202">コンソール ツリーの **[AD FS]** で、**[要求プロバイダー信頼]** を右クリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-202">In the console tree, under **AD FS**, right click **Claims Provider Trusts**.</span></span> <span data-ttu-id="f2248-203">**[要求プロバイダー信頼の追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-203">Select **Add Claims Provider Trust**.</span></span>
3. <span data-ttu-id="f2248-204">**[開始]** をクリックしてウィザードを開始します。</span><span class="sxs-lookup"><span data-stu-id="f2248-204">Click **Start** to start the wizard.</span></span>
4. <span data-ttu-id="f2248-205">[オンラインまたはローカル ネットワークで公開されている要求プロバイダーについてのデータをインポートする] オプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-205">Select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="f2248-206">顧客のフェデレーション メタデータ エンドポイントの URI を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-206">Enter the URI of the customer's federation metadata endpoint.</span></span> <span data-ttu-id="f2248-207">(例: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`。)これを顧客から入手する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-207">(Example: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) You will need to get this from the customer.</span></span>
5. <span data-ttu-id="f2248-208">既定のオプションを使用してウィザードを完了します。</span><span class="sxs-lookup"><span data-stu-id="f2248-208">Complete the wizard using the default options.</span></span>

### <a name="edit-claims-rules"></a><span data-ttu-id="f2248-209">要求規則の編集</span><span class="sxs-lookup"><span data-stu-id="f2248-209">Edit claims rules</span></span>

1. <span data-ttu-id="f2248-210">新しく追加した要求プロバイダー信頼を右クリックし、 **[要求規則の編集]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-210">Right-click the newly added claims provider trust, and select **Edit Claims Rules**.</span></span>
2. <span data-ttu-id="f2248-211">**[規則の追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-211">Click **Add Rule**.</span></span>
3. <span data-ttu-id="f2248-212">[入力方向の要求をパス スルーまたはフィルター処理] を選択し、**[次へ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-212">Select "Pass Through or Filter an Incoming Claim" and click **Next**.</span></span>
   <span data-ttu-id="f2248-213">![変換要求規則の追加ウィザード](./images/edit-claims-rule.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-213">![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)</span></span>
4. <span data-ttu-id="f2248-214">規則の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-214">Enter a name for the rule.</span></span>
5. <span data-ttu-id="f2248-215">[入力方向の要求の種類] で、 **[UPN]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-215">Under "Incoming claim type", select **UPN**.</span></span>
6. <span data-ttu-id="f2248-216">[すべての要求値をパススルーする] を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-216">Select "Pass through all claim values".</span></span>
   <span data-ttu-id="f2248-217">![変換要求規則の追加ウィザード](./images/edit-claims-rule2.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-217">![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)</span></span>
7. <span data-ttu-id="f2248-218">**[完了]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-218">Click **Finish**.</span></span>
8. <span data-ttu-id="f2248-219">手順 2 - 7 を繰り返し、入力方向の要求の種類に **[アンカー要求の種類]** を指定します。</span><span class="sxs-lookup"><span data-stu-id="f2248-219">Repeat steps 2 - 7, and specify **Anchor Claim Type** for the incoming claim type.</span></span>
9. <span data-ttu-id="f2248-220">**[OK]** をクリックしてウィザードを完了します。</span><span class="sxs-lookup"><span data-stu-id="f2248-220">Click **OK** to complete the wizard.</span></span>

### <a name="enable-home-realm-discovery"></a><span data-ttu-id="f2248-221">ホーム領域検出の有効化</span><span class="sxs-lookup"><span data-stu-id="f2248-221">Enable home-realm discovery</span></span>

<span data-ttu-id="f2248-222">次の PowerShell スクリプトを実行します。</span><span class="sxs-lookup"><span data-stu-id="f2248-222">Run the following PowerShell script:</span></span>

```powershell
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

<span data-ttu-id="f2248-223">"name" は要求プロバイダー信頼のフレンドリ名、"suffix" は顧客の AD の UPN サフィックス ("corp.fabrikam.com" など) です。</span><span class="sxs-lookup"><span data-stu-id="f2248-223">where "name" is the friendly name of the claims provider trust, and "suffix" is the UPN suffix for the customer's AD (example, "corp.fabrikam.com").</span></span>

<span data-ttu-id="f2248-224">この構成では、エンド ユーザーが組織のアカウントを入力すると、AD FS によって対応する要求プロバイダーが自動的に選択されます。</span><span class="sxs-lookup"><span data-stu-id="f2248-224">With this configuration, end users can type in their organizational account, and AD FS automatically selects the corresponding claims provider.</span></span> <span data-ttu-id="f2248-225">詳細については、「 [AD FS サインイン ページのカスタマイズ]」の「特定の電子メール サフィックスを使用するための ID プロバイダーの構成」セクションを参照してください。</span><span class="sxs-lookup"><span data-stu-id="f2248-225">See [Customizing the AD FS Sign-in Pages], under the section "Configure Identity Provider to use certain email suffixes".</span></span>

## <a name="configure-the-ad-fs-account-partner"></a><span data-ttu-id="f2248-226">AD FS アカウント パートナーの構成</span><span class="sxs-lookup"><span data-stu-id="f2248-226">Configure the AD FS Account Partner</span></span>

<span data-ttu-id="f2248-227">顧客は次の操作を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f2248-227">The customer must do the following:</span></span>

1. <span data-ttu-id="f2248-228">証明書利用者 (RP) 信頼を追加します。</span><span class="sxs-lookup"><span data-stu-id="f2248-228">Add a relying party (RP) trust.</span></span>
2. <span data-ttu-id="f2248-229">要求規則を追加します。</span><span class="sxs-lookup"><span data-stu-id="f2248-229">Adds claims rules.</span></span>

### <a name="add-the-rp-trust"></a><span data-ttu-id="f2248-230">RP 信頼の追加</span><span class="sxs-lookup"><span data-stu-id="f2248-230">Add the RP trust</span></span>

1. <span data-ttu-id="f2248-231">サーバー マネージャーで、**[ツール]** をクリックし、次に **[AD FS の管理]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-231">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="f2248-232">コンソール ツリーの **[AD FS]** で、**[証明書利用者信頼]** を右クリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-232">In the console tree, under **AD FS**, right click **Relying Party Trusts**.</span></span> <span data-ttu-id="f2248-233">**[証明書利用者信頼の追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-233">Select **Add Relying Party Trust**.</span></span>
3. <span data-ttu-id="f2248-234">**[要求に対応する]** を選択して、**[開始]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-234">Select **Claims Aware** and click **Start**.</span></span>
4. <span data-ttu-id="f2248-235">**[データ ソースの選択]** ページで、[オンラインまたはローカル ネットワーク上で発行された要求プロバイダーに関するデータをインポートする] オプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-235">On the **Select Data Source** page, select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="f2248-236">SaaS プロバイダーのフェデレーション メタデータ エンドポイントの URI を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-236">Enter the URI of the SaaS provider's federation metadata endpoint.</span></span>
   <span data-ttu-id="f2248-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span></span>
5. <span data-ttu-id="f2248-238">**[表示名の指定]** ページで、任意の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-238">On the **Specify Display Name** page, enter any name.</span></span>
6. <span data-ttu-id="f2248-239">**[アクセス制御ポリシーの選択]** で、ポリシーを選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-239">On the **Choose Access Control Policy** page, choose a policy.</span></span> <span data-ttu-id="f2248-240">組織内のすべてのユーザーを許可することも、特定のセキュリティ グループを選択することもできます。</span><span class="sxs-lookup"><span data-stu-id="f2248-240">You could permit everyone in the organization, or choose a specific security group.</span></span>
   <span data-ttu-id="f2248-241">![証明書利用者信頼の追加ウィザード](./images/add-rp-trust2.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-241">![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)</span></span>
7. <span data-ttu-id="f2248-242">**[ポリシー]** ボックスに必要なパラメーターを入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-242">Enter any parameters required in the **Policy** box.</span></span>
8. <span data-ttu-id="f2248-243">**[次へ]** をクリックしてウィザードを完了します。</span><span class="sxs-lookup"><span data-stu-id="f2248-243">Click **Next** to complete the wizard.</span></span>

### <a name="add-claims-rules"></a><span data-ttu-id="f2248-244">要求規則の追加</span><span class="sxs-lookup"><span data-stu-id="f2248-244">Add claims rules</span></span>

1. <span data-ttu-id="f2248-245">新しく追加した証明書利用者信頼を右クリックし、 **[要求発行ポリシーの編集]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-245">Right-click the newly added relying party trust, and select **Edit Claim Issuance Policy**.</span></span>
2. <span data-ttu-id="f2248-246">**[規則の追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-246">Click **Add Rule**.</span></span>
3. <span data-ttu-id="f2248-247">[要求として LDAP 属性を送信] を選択し、 **[次へ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-247">Select "Send LDAP Attributes as Claims" and click **Next**.</span></span>
4. <span data-ttu-id="f2248-248">"UPN" など、規則の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-248">Enter a name for the rule, such as "UPN".</span></span>
5. <span data-ttu-id="f2248-249">**[属性ストア]** で、**[Active Directory]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-249">Under **Attribute store**, select **Active Directory**.</span></span>
   <span data-ttu-id="f2248-250">![変換要求規則の追加ウィザード](./images/add-claims-rules.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-250">![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)</span></span>
6. <span data-ttu-id="f2248-251">**[LDAP 属性のマッピング]** セクションで次の操作を実行します。</span><span class="sxs-lookup"><span data-stu-id="f2248-251">In the **Mapping of LDAP attributes** section:</span></span>
   * <span data-ttu-id="f2248-252">**[LDAP 属性]** で、**[ユーザー プリンシパル名]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-252">Under **LDAP Attribute**, select **User-Principal-Name**.</span></span>
   * <span data-ttu-id="f2248-253">**[出力方向の要求の種類]** で、**[UPN]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="f2248-253">Under **Outgoing Claim Type**, select **UPN**.</span></span>
     <span data-ttu-id="f2248-254">![変換要求規則の追加ウィザード](./images/add-claims-rules2.png)</span><span class="sxs-lookup"><span data-stu-id="f2248-254">![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)</span></span>
7. <span data-ttu-id="f2248-255">**[完了]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-255">Click **Finish**.</span></span>
8. <span data-ttu-id="f2248-256">もう一度 **[規則の追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-256">Click **Add Rule** again.</span></span>
9. <span data-ttu-id="f2248-257">[カスタムの規則を使用して要求を送信] を選択し、 **[次へ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-257">Select "Send Claims Using a Custom Rule" and click **Next**.</span></span>
10. <span data-ttu-id="f2248-258">"アンカー要求の種類" など、規則の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-258">Enter a name for the rule, such as "Anchor Claim Type".</span></span>
11. <span data-ttu-id="f2248-259">**[カスタムの規則]** で、次のように入力します。</span><span class="sxs-lookup"><span data-stu-id="f2248-259">Under **Custom rule**, enter the following:</span></span>

    ```console
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```

    <span data-ttu-id="f2248-260">この規則によって、種類が `anchorclaimtype` の要求が発行されます。</span><span class="sxs-lookup"><span data-stu-id="f2248-260">This rule issues a claim of type `anchorclaimtype`.</span></span> <span data-ttu-id="f2248-261">この要求は、証明書利用者にユーザーの不変 ID として UPN を使用するように指示します。</span><span class="sxs-lookup"><span data-stu-id="f2248-261">The claim tells the relying party to use UPN as the user's immutable ID.</span></span>
12. <span data-ttu-id="f2248-262">**[完了]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="f2248-262">Click **Finish**.</span></span>
13. <span data-ttu-id="f2248-263">**[OK]** をクリックしてウィザードを完了します。</span><span class="sxs-lookup"><span data-stu-id="f2248-263">Click **OK** to complete the wizard.</span></span>

<!-- links -->

[Azure AD Connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[フェデレーションの信頼]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[アカウント パートナー]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[リソース パートナー]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[認証インスタント]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[期限切れ日時]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Expiration time]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[名前の識別子]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[ブログ記事]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[blog post]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[AD FS サインイン ページのカスタマイズ]: https://technet.microsoft.com/library/dn280950.aspx
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
