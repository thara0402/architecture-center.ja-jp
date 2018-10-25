---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 35002b4840b80bcc35b5baf36ec8e414ed8f20be
ms.sourcegitcommit: 2ae794de13c45cf24ad60d4f4dbb193c25944eff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/25/2018
ms.locfileid: "50001900"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="dd7c9-103">Azure コンピューティング サービスのデシジョン ツリー</span><span class="sxs-lookup"><span data-stu-id="dd7c9-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="dd7c9-104">Azure では、複数の方法でお使いのアプリケーション コードをホストできます。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="dd7c9-105">"*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="dd7c9-106">次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="dd7c9-107">このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="dd7c9-108">**このフローチャートを原案として使用します。**</span><span class="sxs-lookup"><span data-stu-id="dd7c9-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="dd7c9-109">すべてのアプリケーションには固有の要件があるため、推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="dd7c9-110">その後、詳細な評価を実行し、次の点を確認します。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="dd7c9-111">機能セット</span><span class="sxs-lookup"><span data-stu-id="dd7c9-111">Feature set</span></span>
- [<span data-ttu-id="dd7c9-112">サービスの制限</span><span class="sxs-lookup"><span data-stu-id="dd7c9-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="dd7c9-113">コスト</span><span class="sxs-lookup"><span data-stu-id="dd7c9-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="dd7c9-114">SLA</span><span class="sxs-lookup"><span data-stu-id="dd7c9-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="dd7c9-115">リージョン別の提供状況</span><span class="sxs-lookup"><span data-stu-id="dd7c9-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="dd7c9-116">開発者のエコシステムとチームのスキル</span><span class="sxs-lookup"><span data-stu-id="dd7c9-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="dd7c9-117">コンピューティングの比較表</span><span class="sxs-lookup"><span data-stu-id="dd7c9-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="dd7c9-118">アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="dd7c9-119">完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="dd7c9-120">Azure でコンテナーをホストする際のオプションの詳細については、 https://azure.microsoft.com/overview/containers/ を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="dd7c9-121">フローチャート</span><span class="sxs-lookup"><span data-stu-id="dd7c9-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="dd7c9-122">定義</span><span class="sxs-lookup"><span data-stu-id="dd7c9-122">Definitions</span></span>

- <span data-ttu-id="dd7c9-123">**リフト アンド シフト:** アプリケーションの再設計やコード変更なしで、ワークロードをクラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-123">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="dd7c9-124">"*リホスト*" とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-124">Also called *rehosting*.</span></span> <span data-ttu-id="dd7c9-125">詳細については、「[Azure Migration Center](https://azure.microsoft.com/migration/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-125">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="dd7c9-126">**クラウド用に最適化:** アプリケーションをリファクタリングすることでクラウドネイティブの機能を利用して、クラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-126">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="dd7c9-127">次の手順</span><span class="sxs-lookup"><span data-stu-id="dd7c9-127">Next steps</span></span>

<span data-ttu-id="dd7c9-128">考慮する必要がある追加条件については、「[Azure コンピューティング サービスを選択するための条件](./compute-comparison.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd7c9-128">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
