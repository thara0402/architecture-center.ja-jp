---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 689ec3f265e563273a75ad98268d03624a7b4536
ms.sourcegitcommit: ce2fa8ac2d310f7078317cade12f1b89db1ffe06
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/23/2018
ms.locfileid: "36338185"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="1753a-103">Azure コンピューティング サービスのデシジョン ツリー</span><span class="sxs-lookup"><span data-stu-id="1753a-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="1753a-104">Azure では、複数の方法でお使いのアプリケーション コードをホストできます。</span><span class="sxs-lookup"><span data-stu-id="1753a-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="1753a-105">"*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。</span><span class="sxs-lookup"><span data-stu-id="1753a-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="1753a-106">次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1753a-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="1753a-107">このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。</span><span class="sxs-lookup"><span data-stu-id="1753a-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="1753a-108">**このフローチャートを原案として使用します。**</span><span class="sxs-lookup"><span data-stu-id="1753a-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="1753a-109">すべてのアプリケーションには固有の要件があるため、推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="1753a-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="1753a-110">その後、詳細な評価を実行し、次の点を確認します。</span><span class="sxs-lookup"><span data-stu-id="1753a-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="1753a-111">機能セット</span><span class="sxs-lookup"><span data-stu-id="1753a-111">Feature set</span></span>
- [<span data-ttu-id="1753a-112">サービスの制限</span><span class="sxs-lookup"><span data-stu-id="1753a-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="1753a-113">コスト</span><span class="sxs-lookup"><span data-stu-id="1753a-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="1753a-114">SLA</span><span class="sxs-lookup"><span data-stu-id="1753a-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="1753a-115">リージョン別の提供状況</span><span class="sxs-lookup"><span data-stu-id="1753a-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="1753a-116">開発者のエコシステムとチームのスキル</span><span class="sxs-lookup"><span data-stu-id="1753a-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="1753a-117">コンピューティングの比較表</span><span class="sxs-lookup"><span data-stu-id="1753a-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="1753a-118">アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。</span><span class="sxs-lookup"><span data-stu-id="1753a-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="1753a-119">完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="1753a-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="1753a-120">Azure でコンテナーをホストする際のオプションの詳細については、https://azure.microsoft.com/overview/containers/ を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1753a-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="1753a-121">フローチャート</span><span class="sxs-lookup"><span data-stu-id="1753a-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="1753a-122">定義</span><span class="sxs-lookup"><span data-stu-id="1753a-122">Definitions</span></span>

- <span data-ttu-id="1753a-123">**未開発:** ゼロから構築された完全に新しいソフトウェア プロジェクトです。</span><span class="sxs-lookup"><span data-stu-id="1753a-123">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="1753a-124">レガシ コードは含まれません。</span><span class="sxs-lookup"><span data-stu-id="1753a-124">It does not include legacy code.</span></span> 

- <span data-ttu-id="1753a-125">**ブラウンフィールド:** 既存のアプリケーションに基づいて構築されたソフトウェア プロジェクトです。</span><span class="sxs-lookup"><span data-stu-id="1753a-125">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="1753a-126">レガシ コードまたはフレームワークが継承されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1753a-126">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="1753a-127">**リフト アンド シフト:** アプリケーションの再設計やコード変更なしで、ワークロードをクラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="1753a-127">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="1753a-128">"*リホスト*" とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="1753a-128">Also called *rehosting*.</span></span> <span data-ttu-id="1753a-129">詳細については、「[Azure Migration Center](https://azure.microsoft.com/migration/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1753a-129">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="1753a-130">**クラウド用に最適化:** アプリケーションをリファクタリングすることでクラウドネイティブの機能を利用して、クラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="1753a-130">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1753a-131">次の手順</span><span class="sxs-lookup"><span data-stu-id="1753a-131">Next steps</span></span>

<span data-ttu-id="1753a-132">考慮する必要がある追加条件については、「[Azure コンピューティング サービスを選択するための条件](./compute-comparison.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1753a-132">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
