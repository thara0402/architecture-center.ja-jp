---
title: CAF:中小企業のガバナンス体験
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 中小企業のガバナンス体験
author: BrianBlanchard
ms.openlocfilehash: a3e078845038a12977e7be5affbf22708411069f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245163"
---
# <a name="small-to-medium-enterprise-governance-journey"></a><span data-ttu-id="483e1-103">中小企業のガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="483e1-103">Small-to-medium enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="483e1-104">ベスト プラクティスの概要</span><span class="sxs-lookup"><span data-stu-id="483e1-104">Best practice overview</span></span>

<span data-ttu-id="483e1-105">このガバナンス体験は、ある架空の会社のガバナンスが成熟するさまざまな段階での経験に沿ったものです。</span><span class="sxs-lookup"><span data-stu-id="483e1-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="483e1-106">実際のお客様の体験に基づいています。</span><span class="sxs-lookup"><span data-stu-id="483e1-106">It is based on real customer journeys.</span></span> <span data-ttu-id="483e1-107">推奨されるベスト プラクティスは、架空の会社の制約とニーズに基づいています。</span><span class="sxs-lookup"><span data-stu-id="483e1-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="483e1-108">すぐに始められるように、この概要では、ベスト プラクティスに基づくガバナンスのための実用最小限の製品 (MVP) が定義されています。</span><span class="sxs-lookup"><span data-stu-id="483e1-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="483e1-109">また、新しいビジネスや技術的リスクが登場したときにベスト プラクティスをさらに追加するいくつかのガバナンス進化へのリンクも提供されています。</span><span class="sxs-lookup"><span data-stu-id="483e1-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="483e1-110">この MVP は、一連の想定に基づくベースラインの起点です。</span><span class="sxs-lookup"><span data-stu-id="483e1-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="483e1-111">この最小限の一連のベスト プラクティスも、独自のビジネス リスクとリスク許容範囲によってもたらされる企業のポリシーに基づいています。</span><span class="sxs-lookup"><span data-stu-id="483e1-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="483e1-112">これらの想定が自分にも当てはまるかどうかを確認するには、この記事の後にある[長い物語](./narrative.md)を読んでください。</span><span class="sxs-lookup"><span data-stu-id="483e1-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

## <a name="governance-best-practice"></a><span data-ttu-id="483e1-113">ガバナンスのベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="483e1-113">Governance best practice</span></span>

<span data-ttu-id="483e1-114">このベスト プラクティスは、組織が複数の Azure サブスクリプションに対して迅速かつ一貫してガバナンス ガードレールを追加するために使用できる基盤として機能します。</span><span class="sxs-lookup"><span data-stu-id="483e1-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="483e1-115">リソースの編成</span><span class="sxs-lookup"><span data-stu-id="483e1-115">Resource organization</span></span>

<span data-ttu-id="483e1-116">次の図では、リソースを編成するためのガバナンス MVP 階層を示します。</span><span class="sxs-lookup"><span data-stu-id="483e1-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![リソース編成の図](../../../_images/governance/resource-organization.png)

<span data-ttu-id="483e1-118">すべてのアプリケーションを管理グループ、サブスクリプション、リソース グループ階層の適切な領域にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="483e1-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="483e1-119">デプロイ計画の間に、クラウド ガバナンス チームは、クラウド導入チームを支援するために必要なノードを階層に作成します。</span><span class="sxs-lookup"><span data-stu-id="483e1-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>  

1. <span data-ttu-id="483e1-120">環境の種類 (運用、開発、テストなど) ごとの管理グループ。</span><span class="sxs-lookup"><span data-stu-id="483e1-120">A management group for each type of environment (such as Production, Development, and Test).</span></span>
2. <span data-ttu-id="483e1-121">各 "アプリケーション分類" のサブスクリプション。</span><span class="sxs-lookup"><span data-stu-id="483e1-121">A subscription for each "application categorization".</span></span>
3. <span data-ttu-id="483e1-122">アプリケーションごとに個別のリソース グループ。</span><span class="sxs-lookup"><span data-stu-id="483e1-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="483e1-123">このグループ階層の各レベルで、一貫性のある用語体系を適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="483e1-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

<span data-ttu-id="483e1-124">このパターンの使用例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="483e1-124">Here is an example of this pattern in use:</span></span>

![中規模の会社のリソース組織の例](../../../_images/governance/mid-market-resource-organization.png)

<span data-ttu-id="483e1-126">これらのパターンには、階層を必要以上に複雑にしないで成長に対応する余地があります。</span><span class="sxs-lookup"><span data-stu-id="483e1-126">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="483e1-127">ガバナンスの進化</span><span class="sxs-lookup"><span data-stu-id="483e1-127">Governance evolutions</span></span>

<span data-ttu-id="483e1-128">この MVP をデプロイした後は、追加のガバナンス レイヤーを環境にすばやく組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="483e1-128">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="483e1-129">特定のビジネス ニーズに合わせて MVP を進化させる方法をいくつか次に示します。</span><span class="sxs-lookup"><span data-stu-id="483e1-129">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="483e1-130">保護されたデータのセキュリティ ベースライン</span><span class="sxs-lookup"><span data-stu-id="483e1-130">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="483e1-131">ミッション クリティカルなアプリケーションのリソース構成</span><span class="sxs-lookup"><span data-stu-id="483e1-131">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="483e1-132">コスト管理の制御</span><span class="sxs-lookup"><span data-stu-id="483e1-132">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="483e1-133">マルチクラウドの進化の制御</span><span class="sxs-lookup"><span data-stu-id="483e1-133">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="483e1-134">このベスト プラクティスの役割</span><span class="sxs-lookup"><span data-stu-id="483e1-134">What does this best practice do?</span></span>

<span data-ttu-id="483e1-135">MVP では、企業ポリシーを迅速に適用するために、[デプロイ高速化](../../deployment-acceleration/overview.md)規範のプラクティスとツールが確立されます。</span><span class="sxs-lookup"><span data-stu-id="483e1-135">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="483e1-136">具体的には、MVP では、Azure Blueprints、Azure Policy、Azure 管理グループを使用して、この架空の会社の物語で定義されているいくつかの基本的な企業ポリシーが適用されます。</span><span class="sxs-lookup"><span data-stu-id="483e1-136">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="483e1-137">それらの企業ポリシーが Resource Manager テンプレートと Azure ポリシーを使用して適用されて、ID とセキュリティのための非常に小さいベースラインが確立されます。</span><span class="sxs-lookup"><span data-stu-id="483e1-137">Those corporate policies are applied using Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![増分ガバナンス MVP の例](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="483e1-139">ベスト プラクティスの進化</span><span class="sxs-lookup"><span data-stu-id="483e1-139">Evolving the best practice</span></span>

<span data-ttu-id="483e1-140">このガバナンス MVP は、時間をかけたガバナンス プラクティスの進化に使用されます。</span><span class="sxs-lookup"><span data-stu-id="483e1-140">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="483e1-141">導入が進むと、ビジネス上のリスクが増大します。</span><span class="sxs-lookup"><span data-stu-id="483e1-141">As adoption advances, business risk grows.</span></span> <span data-ttu-id="483e1-142">それらのリスクを軽減するため、CAF ガバナンス モデル内のさまざまな規範が進化します。</span><span class="sxs-lookup"><span data-stu-id="483e1-142">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="483e1-143">このシリーズの以降の記事では、架空の企業に影響を与える企業ポリシーの進化について説明します。</span><span class="sxs-lookup"><span data-stu-id="483e1-143">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="483e1-144">これらの進化は、次の 3 つの規範で発生します。</span><span class="sxs-lookup"><span data-stu-id="483e1-144">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="483e1-145">コスト管理: 導入が拡大するとき。</span><span class="sxs-lookup"><span data-stu-id="483e1-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="483e1-146">セキュリティ ベースライン: 保護されたデータがデプロイされるとき。</span><span class="sxs-lookup"><span data-stu-id="483e1-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="483e1-147">リソースの整合性: IT 運用でミッション クリティカルなワークロードのサポートが始まるとき。</span><span class="sxs-lookup"><span data-stu-id="483e1-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![増分ガバナンス MVP の例](../../../_images/governance/governance-evolution.png)

## <a name="next-steps"></a><span data-ttu-id="483e1-149">次の手順</span><span class="sxs-lookup"><span data-stu-id="483e1-149">Next steps</span></span>

<span data-ttu-id="483e1-150">ガバナンス MVP について理解し、この後のガバナンスの進化についてわかったら、コンテキストについてさらに理解するのに役立つ物語を読んでください。</span><span class="sxs-lookup"><span data-stu-id="483e1-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="483e1-151">理解に役立つ物語を読む</span><span class="sxs-lookup"><span data-stu-id="483e1-151">Read the supporting narrative</span></span>](./narrative.md)
