---
title: Tailspin Surveys アプリケーションについて
description: Tailspin Surveys アプリケーションの概要。
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de2830b5c492e027c189a79e45ccc6634cab436b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247253"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="97b8a-103">Tailspin シナリオ</span><span class="sxs-lookup"><span data-stu-id="97b8a-103">The Tailspin scenario</span></span>

<span data-ttu-id="97b8a-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="97b8a-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="97b8a-105">Tailspin は、Surveys という名前の SaaS アプリケーションを開発している仮の会社です。</span><span class="sxs-lookup"><span data-stu-id="97b8a-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="97b8a-106">このアプリケーションを使用すると、オンライン アンケートを作成および発行することができます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="97b8a-107">組織はアプリケーションにサインアップできます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="97b8a-108">組織がサインアップした後、ユーザーはその組織の資格情報でアプリケーションにサインインできます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="97b8a-109">ユーザーはアンケートを作成、編集、および発行することができます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="97b8a-110">アプリケーションを開始するには、「[Surveys アプリケーションの実行]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="97b8a-110">To get started with the application, see [Run the Surveys application].</span></span>

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="97b8a-111">ユーザーがアンケートを作成、編集、および表示できる</span><span class="sxs-lookup"><span data-stu-id="97b8a-111">Users can create, edit, and view surveys</span></span>

<span data-ttu-id="97b8a-112">認証されたユーザーは、自身が作成した、または自身が共同作成者の権限を持つすべてのアンケートを表示し、新しいアンケートを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="97b8a-113">ユーザーが組織 ID `bob@contoso.com` でサインインしていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="97b8a-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![アンケート アプリ](./images/surveys-screenshot.png)

<span data-ttu-id="97b8a-115">このスクリーンショットには、[アンケートの編集] ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-115">This screenshot shows the Edit Survey page:</span></span>

![アンケートの編集](./images/edit-survey.png)

<span data-ttu-id="97b8a-117">ユーザーは、同じテナント内で他のユーザーによって作成されたアンケートを表示することもできます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![テナントのアンケート](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="97b8a-119">アンケートの所有者が共同作成者を招待できる</span><span class="sxs-lookup"><span data-stu-id="97b8a-119">Survey owners can invite contributors</span></span>

<span data-ttu-id="97b8a-120">ユーザーがアンケートを作成する場合は、アンケートの共同作成者に他のユーザーを招待できます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="97b8a-121">共同作成者はアンケートを編集することはできますが、削除または公開することはできません。</span><span class="sxs-lookup"><span data-stu-id="97b8a-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>

![共同作成者の追加](./images/add-contributor.png)

<span data-ttu-id="97b8a-123">ユーザーが他のテナントから共同作成者を追加できます。これにより、リソースのテナント間共有が可能になります。</span><span class="sxs-lookup"><span data-stu-id="97b8a-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="97b8a-124">このスクリーンショットでは、Bob (`bob@contoso.com`) が自身で作成したアンケートに、Alice (`alice@fabrikam.com`) を共同作成者として追加しています。</span><span class="sxs-lookup"><span data-stu-id="97b8a-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="97b8a-125">Alice がログインすると、[共同作成できるアンケート] の下にアンケートが表示されます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![アンケートの共同作成者](./images/contributor.png)

<span data-ttu-id="97b8a-127">Alice が Contoso テナントのゲストとしてではなく、自分のテナントにサインインすることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="97b8a-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="97b8a-128">Alice に付与されているのは、そのアンケートの共同作成者のアクセス許可だけです &mdash; Contoso テナントから他のアンケートを表示することはできません。</span><span class="sxs-lookup"><span data-stu-id="97b8a-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="97b8a-129">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="97b8a-129">Architecture</span></span>

<span data-ttu-id="97b8a-130">Surveys アプリケーションは、Web フロント エンドおよび Web API バックエンドで構成されます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="97b8a-131">両方とも [ASP.NET Core] を使用して実装されます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="97b8a-132">Web アプリケーションでは、Azure Active Directory (Azure AD) を使用して、ユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="97b8a-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="97b8a-133">また、Web アプリケーションでは Azure AD を呼び出して、Web API の OAuth 2 アクセス トークンを取得します。</span><span class="sxs-lookup"><span data-stu-id="97b8a-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="97b8a-134">アクセス トークンは、Azure Redis Cache でキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="97b8a-135">キャッシュを使用すると、複数のインスタンスで同じトークンのキャッシュ (たとえば、サーバー ファーム) を共有できます。</span><span class="sxs-lookup"><span data-stu-id="97b8a-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![アーキテクチャ](./images/architecture.png)

<span data-ttu-id="97b8a-137">[**次へ**][authentication]</span><span class="sxs-lookup"><span data-stu-id="97b8a-137">[**Next**][authentication]</span></span>

<!-- links -->

[authentication]: authenticate.md

[Surveys アプリケーションの実行]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
