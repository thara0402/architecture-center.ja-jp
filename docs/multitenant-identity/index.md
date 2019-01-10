---
title: マルチテナント アプリケーションの ID 管理
description: マルチテナント アプリケーションでの認証、承認、および ID 管理のベスト プラクティス。
author: MikeWasson
ms.date: 07/21/2017
ms.openlocfilehash: 864317cc98ee0211d4f4274253eda12b72beceed
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114149"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="8dd98-103">マルチテナント アプリケーションでの ID 管理</span><span class="sxs-lookup"><span data-stu-id="8dd98-103">Manage identity in multitenant applications</span></span>

<span data-ttu-id="8dd98-104">この一連の記事では、Azure AD を認証と ID 管理のために使用する際のマルチテナントのベスト プラクティスについて説明します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="8dd98-105">[![GitHub](../_images/github.png) サンプル コード][sample-application]</span><span class="sxs-lookup"><span data-stu-id="8dd98-105">[![GitHub](../_images/github.png) Sample code][sample-application]</span></span>

<span data-ttu-id="8dd98-106">マルチテナント アプリケーションを構築する場合、最初の課題の 1 つは、すべてのユーザーがテナントに属しているようになるために、ユーザー ID を管理することです。</span><span class="sxs-lookup"><span data-stu-id="8dd98-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="8dd98-107">例: </span><span class="sxs-lookup"><span data-stu-id="8dd98-107">For example:</span></span>

- <span data-ttu-id="8dd98-108">ユーザーは、その組織の資格情報でサインインします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-108">Users sign in with their organizational credentials.</span></span>
- <span data-ttu-id="8dd98-109">ユーザーは、その組織のデータへのアクセス権を持つ必要がありますが、他のテナントに属しているデータは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
- <span data-ttu-id="8dd98-110">組織は、アプリケーションにサインアップし、そのメンバーにアプリケーション ロールを割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="8dd98-111">Azure Active Directory (Azure AD) には、これらのシナリオのすべてをサポートするいくつかの優れた機能があります。</span><span class="sxs-lookup"><span data-stu-id="8dd98-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="8dd98-112">この一連の記事に付け加えるために、マルチテナント アプリケーションの完全な[エンドツーエンド実装][sample-application]を作成しました。</span><span class="sxs-lookup"><span data-stu-id="8dd98-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample-application] of a multitenant application.</span></span> <span data-ttu-id="8dd98-113">記事には、アプリケーションの作成プロセスで学習した内容が反映されています。</span><span class="sxs-lookup"><span data-stu-id="8dd98-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="8dd98-114">アプリケーションを開始するには、[「Run the Surveys application」][running-the-app] (Surveys アプリケーションの実行) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8dd98-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="8dd98-115">はじめに</span><span class="sxs-lookup"><span data-stu-id="8dd98-115">Introduction</span></span>

<span data-ttu-id="8dd98-116">たとえば、クラウドでホストされるエンタープライズ SaaS アプリケーションを作成するとします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="8dd98-117">当然ながら、アプリケーションにはユーザーがいます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-117">Of course, the application will have users:</span></span>

![ユーザー](./images/users.png)

<span data-ttu-id="8dd98-119">ただし、ユーザーは組織に所属しています。</span><span class="sxs-lookup"><span data-stu-id="8dd98-119">But those users belong to organizations:</span></span>

![組織のユーザー](./images/org-users.png)

<span data-ttu-id="8dd98-121">例:Tailspin は、SaaS アプリケーションへのサブスクリプションを販売しています。</span><span class="sxs-lookup"><span data-stu-id="8dd98-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="8dd98-122">Contoso と Fabrikam がアプリにサインアップします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="8dd98-123">Alice (`alice@contoso`) がサインインすると、アプリケーションは Alice が Contoso に属していることを認識します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

- <span data-ttu-id="8dd98-124">Alice には、Contoso データへのアクセス権が *付与されます* 。</span><span class="sxs-lookup"><span data-stu-id="8dd98-124">Alice *should* have access to Contoso data.</span></span>
- <span data-ttu-id="8dd98-125">Alice は Fabrikam データのアクセス権を持っている*べきではありません*。</span><span class="sxs-lookup"><span data-stu-id="8dd98-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="8dd98-126">このガイダンスでは、マルチテナント アプリケーションで [Azure Active Directory (Azure AD)](/azure/active-directory) を使用してサインインと認証を処理し、ユーザー ID を管理する方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory (Azure AD)](/azure/active-directory) to handle sign-in and authentication.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a><span data-ttu-id="8dd98-127">マルチテナントとは</span><span class="sxs-lookup"><span data-stu-id="8dd98-127">What is multitenancy?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="8dd98-128">*テナント* は、ユーザーのグループです。</span><span class="sxs-lookup"><span data-stu-id="8dd98-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="8dd98-129">SaaS アプリケーションのテナントとは、アプリケーションのサブスクライバーまたは顧客です。</span><span class="sxs-lookup"><span data-stu-id="8dd98-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="8dd98-130">*マルチテナント* は、複数のテナントがアプリの同じ物理インスタンスを共有するアーキテクチャです。</span><span class="sxs-lookup"><span data-stu-id="8dd98-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="8dd98-131">テナントは物理リソース (VM や記憶域など) を共有しますが、各テナントにはアプリの論理インスタンスが付与されます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="8dd98-132">通常、アプリケーション データは、テナント内のユーザー間で共有されますが、他のテナントとは共有されません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![マルチテナント](./images/multitenant.png)

<span data-ttu-id="8dd98-134">このアーキテクチャを、各テナントが専用の物理インスタンスを持つシングルテナント アーキテクチャと比較します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="8dd98-135">シングルテナント アーキテクチャでは、アプリの新しいインスタンスを起動してテナントを追加します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![シングル テナント](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="8dd98-137">マルチテナントと水平スケーリング</span><span class="sxs-lookup"><span data-stu-id="8dd98-137">Multitenancy and horizontal scaling</span></span>

<span data-ttu-id="8dd98-138">クラウドでスケーリングを達成するには、物理インスタンスを追加する方法が一般的です。</span><span class="sxs-lookup"><span data-stu-id="8dd98-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="8dd98-139">これは "*水平スケーリング*" または "*スケーリング アウト*" と呼ばれます。たとえば、Web アプリがあるとします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="8dd98-140">多くのトラフィックを処理する場合、サーバー VM を追加し、ロード バランサーの背後に配置します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="8dd98-141">各 VM では、別物理インスタンスの Web アプリを実行します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-141">Each VM runs a separate physical instance of the web app.</span></span>

![Web サイトの負荷分散](./images/load-balancing.png)

<span data-ttu-id="8dd98-143">任意の要求を任意のインスタンスにルーティングできます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="8dd98-144">その結果、システムは 1 つの論理インスタンスとして機能します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="8dd98-145">ユーザーに影響を与えることなく、VM を増減することができます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="8dd98-146">このアーキテクチャでは、各物理インスタンスがマルチ テナントであり、インスタンスを追加することで拡張できます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="8dd98-147">1 つのインスタンスが停止しても、テナントに影響は及びません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="8dd98-148">マルチテナント アプリの ID</span><span class="sxs-lookup"><span data-stu-id="8dd98-148">Identity in a multitenant app</span></span>

<span data-ttu-id="8dd98-149">マルチテナント アプリの場合、テナントのコンテキストでユーザーを考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dd98-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

### <a name="authentication"></a><span data-ttu-id="8dd98-150">Authentication</span><span class="sxs-lookup"><span data-stu-id="8dd98-150">Authentication</span></span>

- <span data-ttu-id="8dd98-151">ユーザーは、組織の資格情報を使用してアプリにサインインします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="8dd98-152">ユーザーがアプリの新しいユーザー プロファイルを作成する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-152">They don't have to create new user profiles for the app.</span></span>
- <span data-ttu-id="8dd98-153">同じ組織内のユーザーは、同じテナントに属します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-153">Users within the same organization are part of the same tenant.</span></span>
- <span data-ttu-id="8dd98-154">ユーザーがサインインすると、アプリケーションユーザーが属するテナントを認識します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

### <a name="authorization"></a><span data-ttu-id="8dd98-155">Authorization</span><span class="sxs-lookup"><span data-stu-id="8dd98-155">Authorization</span></span>

- <span data-ttu-id="8dd98-156">ユーザーのアクション (リソースの表示など) を承認する場合、アプリはユーザーのテナントを考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dd98-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
- <span data-ttu-id="8dd98-157">ユーザーには、アプリケーション内のロール ("Admin"、"Standard User" など) が割り当てられていることがあります。</span><span class="sxs-lookup"><span data-stu-id="8dd98-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="8dd98-158">ロールの割り当ては、SaaS プロバイダーではなく、顧客が管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dd98-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="8dd98-159">**例:**</span><span class="sxs-lookup"><span data-stu-id="8dd98-159">**Example.**</span></span> <span data-ttu-id="8dd98-160">Contoso の従業員である Alice は、ブラウザーでアプリケーションを開き、[ログイン] ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="8dd98-161">ログイン画面にリダイレクトされ、Alice は会社の資格情報 (ユーザー名とパスワード) を入力します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="8dd98-162">この時点で、 `alice@contoso.com`としてアプリにログインされます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="8dd98-163">アプリケーションは、Alice がこのアプリケーションの管理者であることも認識します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="8dd98-164">Alice は管理者なので、Contoso に属するすべてのリソースの一覧を表示できます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="8dd98-165">ただし、自分のテナント内でのみ管理者なので、Fabrikam のリソースは表示できません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="8dd98-166">このガイドでは、ID 管理のために Azure AD を使用することに注目します。</span><span class="sxs-lookup"><span data-stu-id="8dd98-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

- <span data-ttu-id="8dd98-167">ここでは、顧客が Azure AD (Office 365 テナントや Dynamics CRM テナントなど) にユーザー プロファイルを保存しているとします。</span><span class="sxs-lookup"><span data-stu-id="8dd98-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
- <span data-ttu-id="8dd98-168">オンプレミスの Active Directory (AD) がある顧客は、[Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) を使用してオンプレミスの AD を Azure AD と同期できます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="8dd98-169">オンプレミスの AD がある顧客が (会社の IT ポリシーなどの理由で) Azure AD Connect を使用できない場合、SaaS プロバイダーは、Active Directory Federation Services (AD FS) を介して顧客の AD と連携できます。</span><span class="sxs-lookup"><span data-stu-id="8dd98-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="8dd98-170">このオプションについては、「 [顧客の AD FS とのフェデレーション](adfs.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8dd98-170">This option is described in [Federating with a customer's AD FS](adfs.md).</span></span>

<span data-ttu-id="8dd98-171">このガイドでは、データのパーティション分割、テナントごとの構成など、マルチテナントの他の側面について考慮していません。</span><span class="sxs-lookup"><span data-stu-id="8dd98-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

[<span data-ttu-id="8dd98-172">**次へ**</span><span class="sxs-lookup"><span data-stu-id="8dd98-172">**Next**</span></span>](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
