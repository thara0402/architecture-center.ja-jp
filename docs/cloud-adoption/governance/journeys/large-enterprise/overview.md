---
title: CAF:大企業のガバナンス体験
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 大企業のガバナンス体験
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901334"
---
# <a name="caf-large-enterprise-governance-journey"></a><span data-ttu-id="b2565-103">CAF:大企業のガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="b2565-103">CAF: Large enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="b2565-104">ベスト プラクティスの概要</span><span class="sxs-lookup"><span data-stu-id="b2565-104">Best practice overview</span></span>

<span data-ttu-id="b2565-105">このガバナンス体験は、ある架空の会社のガバナンスが成熟するさまざまな段階での経験に沿ったものです。</span><span class="sxs-lookup"><span data-stu-id="b2565-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="b2565-106">実際のお客様の体験に基づいています。</span><span class="sxs-lookup"><span data-stu-id="b2565-106">It is based on real customer journeys.</span></span> <span data-ttu-id="b2565-107">推奨されるベスト プラクティスは、架空の会社の制約とニーズに基づいています。</span><span class="sxs-lookup"><span data-stu-id="b2565-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="b2565-108">すぐに始められるように、この概要では、ベスト プラクティスに基づくガバナンスのための実用最小限の製品 (MVP) が定義されています。</span><span class="sxs-lookup"><span data-stu-id="b2565-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="b2565-109">また、新しいビジネスや技術的リスクが登場したときにベスト プラクティスをさらに追加するいくつかのガバナンス進化へのリンクも提供されています。</span><span class="sxs-lookup"><span data-stu-id="b2565-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="b2565-110">この MVP は、一連の想定に基づくベースラインの起点です。</span><span class="sxs-lookup"><span data-stu-id="b2565-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="b2565-111">この最小限の一連のベスト プラクティスも、独自のビジネス リスクとリスク許容範囲によってもたらされる企業のポリシーに基づいています。</span><span class="sxs-lookup"><span data-stu-id="b2565-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="b2565-112">これらの想定が自分にも当てはまるかどうかを確認するには、この記事の後にある[長い物語](./narrative.md)を読んでください。</span><span class="sxs-lookup"><span data-stu-id="b2565-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

### <a name="governance-best-practice"></a><span data-ttu-id="b2565-113">ガバナンスのベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="b2565-113">Governance best practice</span></span>

<span data-ttu-id="b2565-114">このベスト プラクティスは、組織が複数の Azure サブスクリプションに対して迅速かつ一貫してガバナンス ガードレールを追加するために使用できる基盤として機能します。</span><span class="sxs-lookup"><span data-stu-id="b2565-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="b2565-115">リソースの編成</span><span class="sxs-lookup"><span data-stu-id="b2565-115">Resource organization</span></span>

<span data-ttu-id="b2565-116">次の図では、リソースを編成するためのガバナンス MVP 階層を示します。</span><span class="sxs-lookup"><span data-stu-id="b2565-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![リソース編成の図](../../../_images/governance/resource-organization.png)

<span data-ttu-id="b2565-118">すべてのアプリケーションを管理グループ、サブスクリプション、リソース グループ階層の適切な領域にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b2565-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="b2565-119">デプロイ計画の間に、クラウド ガバナンス チームは、クラウド導入チームを支援するために必要なノードを階層に作成します。</span><span class="sxs-lookup"><span data-stu-id="b2565-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>

1. <span data-ttu-id="b2565-120">地理的な場所そして環境 (運用、非運用) が反映された詳細な階層を持つ各部署の管理グループ。</span><span class="sxs-lookup"><span data-stu-id="b2565-120">A management group for each business unit with a detailed hierarchy that reflects geography then environment type (Production, Non-Production).</span></span>
2. <span data-ttu-id="b2565-121">部署、地理的な場所、環境、および "アプリケーション分類" の一意の組み合わせごとのサブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="b2565-121">A subscription for each unique combination of business unit, geography, environment, and "Application Categorization."</span></span>
3. <span data-ttu-id="b2565-122">アプリケーションごとに個別のリソース グループ。</span><span class="sxs-lookup"><span data-stu-id="b2565-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="b2565-123">このグループ階層の各レベルで、一貫性のある用語体系を適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b2565-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

![大企業のリソース編成の図](../../../_images/governance/large-enterprise-resource-organization.png)

<span data-ttu-id="b2565-125">これらのパターンには、階層を必要以上に複雑にしないで成長に対応する余地があります。</span><span class="sxs-lookup"><span data-stu-id="b2565-125">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="b2565-126">ガバナンスの進化</span><span class="sxs-lookup"><span data-stu-id="b2565-126">Governance evolutions</span></span>

<span data-ttu-id="b2565-127">この MVP をデプロイした後は、追加のガバナンス レイヤーを環境にすばやく組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="b2565-127">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="b2565-128">特定のビジネス ニーズに合わせて MVP を進化させる方法をいくつか次に示します。</span><span class="sxs-lookup"><span data-stu-id="b2565-128">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="b2565-129">保護されたデータのセキュリティ ベースライン</span><span class="sxs-lookup"><span data-stu-id="b2565-129">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="b2565-130">ミッション クリティカルなアプリケーションのリソース構成</span><span class="sxs-lookup"><span data-stu-id="b2565-130">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="b2565-131">コスト管理の制御</span><span class="sxs-lookup"><span data-stu-id="b2565-131">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="b2565-132">マルチクラウドの進化の制御</span><span class="sxs-lookup"><span data-stu-id="b2565-132">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="b2565-133">このベスト プラクティスの役割</span><span class="sxs-lookup"><span data-stu-id="b2565-133">What does this best practice do?</span></span>

<span data-ttu-id="b2565-134">MVP では、企業ポリシーを迅速に適用するために、[デプロイ高速化](../../deployment-acceleration/overview.md)規範のプラクティスとツールが確立されます。</span><span class="sxs-lookup"><span data-stu-id="b2565-134">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="b2565-135">具体的には、MVP では、Azure Blueprints、Azure Policy、Azure 管理グループを使用して、この架空の会社の物語で定義されているいくつかの基本的な企業ポリシーが適用されます。</span><span class="sxs-lookup"><span data-stu-id="b2565-135">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="b2565-136">それらの企業ポリシーが Azure Resource Manager テンプレートと Azure ポリシーを使用して適用されて、ID とセキュリティのための非常に小さいベースラインが確立されます。</span><span class="sxs-lookup"><span data-stu-id="b2565-136">Those corporate policies are applied using Azure Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![増分ガバナンス MVP の例](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="b2565-138">ベスト プラクティスの進化</span><span class="sxs-lookup"><span data-stu-id="b2565-138">Evolving the best practice</span></span>

<span data-ttu-id="b2565-139">このガバナンス MVP は、時間をかけたガバナンス プラクティスの進化に使用されます。</span><span class="sxs-lookup"><span data-stu-id="b2565-139">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="b2565-140">導入が進むと、ビジネス上のリスクが増大します。</span><span class="sxs-lookup"><span data-stu-id="b2565-140">As adoption advances, business risk grows.</span></span> <span data-ttu-id="b2565-141">それらのリスクを軽減するため、CAF ガバナンス モデル内のさまざまな規範が進化します。</span><span class="sxs-lookup"><span data-stu-id="b2565-141">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="b2565-142">このシリーズの以降の記事では、架空の企業に影響を与える企業ポリシーの進化について説明します。</span><span class="sxs-lookup"><span data-stu-id="b2565-142">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="b2565-143">これらの進化は、3 つの規範で発生します。</span><span class="sxs-lookup"><span data-stu-id="b2565-143">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="b2565-144">ID ベースライン: 物語で移行の依存関係が進化するとき。</span><span class="sxs-lookup"><span data-stu-id="b2565-144">Identity Baseline, as migration dependencies evolve in the narrative</span></span>
- <span data-ttu-id="b2565-145">コスト管理: 導入が拡大するとき。</span><span class="sxs-lookup"><span data-stu-id="b2565-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="b2565-146">セキュリティ ベースライン: 保護されたデータがデプロイされるとき。</span><span class="sxs-lookup"><span data-stu-id="b2565-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="b2565-147">リソースの整合性: IT 運用でミッション クリティカルなワークロードのサポートが始まるとき。</span><span class="sxs-lookup"><span data-stu-id="b2565-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![増分ガバナンス MVP の例](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a><span data-ttu-id="b2565-149">次の手順</span><span class="sxs-lookup"><span data-stu-id="b2565-149">Next steps</span></span>

<span data-ttu-id="b2565-150">ガバナンス MVP について理解し、この後のガバナンスの進化についてわかったら、コンテキストについてさらに理解するのに役立つ物語を読んでください。</span><span class="sxs-lookup"><span data-stu-id="b2565-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="b2565-151">理解に役立つ物語を読む</span><span class="sxs-lookup"><span data-stu-id="b2565-151">Read the supporting narrative</span></span>](./narrative.md)
