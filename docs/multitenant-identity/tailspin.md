---
title: "Tailspin Surveys アプリケーションについて"
description: "Tailspin Surveys アプリケーションの概要"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="0010b-103">Tailspin シナリオ</span><span class="sxs-lookup"><span data-stu-id="0010b-103">The Tailspin scenario</span></span>

<span data-ttu-id="0010b-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="0010b-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="0010b-105">Tailspin は、Surveys という名前の SaaS アプリケーションを開発している仮の会社です。</span><span class="sxs-lookup"><span data-stu-id="0010b-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="0010b-106">このアプリケーションを使用すると、オンライン アンケートを作成および発行することができます。</span><span class="sxs-lookup"><span data-stu-id="0010b-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="0010b-107">組織はアプリケーションにサインアップできます。</span><span class="sxs-lookup"><span data-stu-id="0010b-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="0010b-108">組織がサインアップした後、ユーザーはその組織の資格情報でアプリケーションにサインインできます。</span><span class="sxs-lookup"><span data-stu-id="0010b-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="0010b-109">ユーザーはアンケートを作成、編集、および発行することができます。</span><span class="sxs-lookup"><span data-stu-id="0010b-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="0010b-110">アプリケーションを開始するには、「[Run the Surveys application (Surveys アプリケーションの実行)]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0010b-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="0010b-111">ユーザーがアンケートを作成、編集、および表示できる</span><span class="sxs-lookup"><span data-stu-id="0010b-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="0010b-112">認証されたユーザーは、自身が作成した、または自身が共同作成者の権限を持つすべてのアンケートを表示し、新しいアンケートを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="0010b-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="0010b-113">ユーザーが組織 ID `bob@contoso.com` でサインインしていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="0010b-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![アンケート アプリ](./images/surveys-screenshot.png)

<span data-ttu-id="0010b-115">このスクリーンショットには、[アンケートの編集] ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="0010b-115">This screenshot shows the Edit Survey page:</span></span>

![アンケートの編集](./images/edit-survey.png)

<span data-ttu-id="0010b-117">ユーザーは、同じテナント内で他のユーザーによって作成されたアンケートを表示することもできます。</span><span class="sxs-lookup"><span data-stu-id="0010b-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![テナントのアンケート](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="0010b-119">アンケートの所有者が共同作成者を招待できる</span><span class="sxs-lookup"><span data-stu-id="0010b-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="0010b-120">ユーザーがアンケートを作成する場合は、アンケートの共同作成者に他のユーザーを招待できます。</span><span class="sxs-lookup"><span data-stu-id="0010b-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="0010b-121">共同作成者はアンケートを編集することはできますが、削除または公開することはできません。</span><span class="sxs-lookup"><span data-stu-id="0010b-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![共同作成者の追加](./images/add-contributor.png)

<span data-ttu-id="0010b-123">ユーザーが他のテナントから共同作成者を追加できます。これにより、リソースのテナント間共有が可能になります。</span><span class="sxs-lookup"><span data-stu-id="0010b-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="0010b-124">このスクリーンショットでは、Bob (`bob@contoso.com`) が自身で作成したアンケートに、Alice (`alice@fabrikam.com`) を共同作成者として追加しています。</span><span class="sxs-lookup"><span data-stu-id="0010b-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="0010b-125">Alice がログインすると、[共同作成できるアンケート] の下にアンケートが表示されます。</span><span class="sxs-lookup"><span data-stu-id="0010b-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![アンケートの共同作成者](./images/contributor.png)

<span data-ttu-id="0010b-127">Alice が Contoso テナントのゲストとしてではなく、自分のテナントにサインインすることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="0010b-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="0010b-128">Alice に付与されているのは、そのアンケートの共同作成者のアクセス許可だけです &mdash; Contoso テナントから他のアンケートを表示することはできません。</span><span class="sxs-lookup"><span data-stu-id="0010b-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="0010b-129">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="0010b-129">Architecture</span></span>
<span data-ttu-id="0010b-130">Surveys アプリケーションは、Web フロント エンドおよび Web API バックエンドで構成されます。</span><span class="sxs-lookup"><span data-stu-id="0010b-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="0010b-131">両方とも [ASP.NET Core] を使用して実装されます。</span><span class="sxs-lookup"><span data-stu-id="0010b-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="0010b-132">Web アプリケーションでは、Azure Active Directory (Azure AD) を使用して、ユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="0010b-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="0010b-133">また、Web アプリケーションでは Azure AD を呼び出して、Web API の OAuth 2 アクセス トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="0010b-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="0010b-134">アクセス トークンは、Azure Redis Cache でキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="0010b-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="0010b-135">キャッシュを使用すると、複数のインスタンスで同じトークンのキャッシュ (たとえば、サーバー ファーム) を共有できます。</span><span class="sxs-lookup"><span data-stu-id="0010b-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![アーキテクチャ](./images/architecture.png)

<span data-ttu-id="0010b-137">[**次へ**][authentication]</span><span class="sxs-lookup"><span data-stu-id="0010b-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[Run the Surveys application (Surveys アプリケーションの実行)]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
