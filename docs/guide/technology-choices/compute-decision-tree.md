---
title: Azure コンピューティング サービスのデシジョン ツリー
titleSuffix: Azure Application Architecture Guide
description: コンピューティング サービスを選択するためのフローチャート。
author: MikeWasson
ms.date: 11/03/2018
ms.custom: seojan19
ms.openlocfilehash: 905b9956c9dcddddb21a87ea588af0ad5160ae2a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114081"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="f2f56-103">Azure コンピューティング サービスのデシジョン ツリー</span><span class="sxs-lookup"><span data-stu-id="f2f56-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="f2f56-104">Azure では、複数の方法でお使いのアプリケーション コードをホストできます。</span><span class="sxs-lookup"><span data-stu-id="f2f56-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="f2f56-105">"*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。</span><span class="sxs-lookup"><span data-stu-id="f2f56-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="f2f56-106">次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="f2f56-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="f2f56-107">このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。</span><span class="sxs-lookup"><span data-stu-id="f2f56-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span>

<span data-ttu-id="f2f56-108">**このフローチャートを原案として使用します。**</span><span class="sxs-lookup"><span data-stu-id="f2f56-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="f2f56-109">すべてのアプリケーションには固有の要件があるため、推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="f2f56-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="f2f56-110">その後、詳細な評価を実行し、次の点を確認します。</span><span class="sxs-lookup"><span data-stu-id="f2f56-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>

- <span data-ttu-id="f2f56-111">機能セット</span><span class="sxs-lookup"><span data-stu-id="f2f56-111">Feature set</span></span>
- [<span data-ttu-id="f2f56-112">サービスの制限</span><span class="sxs-lookup"><span data-stu-id="f2f56-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="f2f56-113">コスト</span><span class="sxs-lookup"><span data-stu-id="f2f56-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="f2f56-114">SLA</span><span class="sxs-lookup"><span data-stu-id="f2f56-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="f2f56-115">リージョン別の提供状況</span><span class="sxs-lookup"><span data-stu-id="f2f56-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="f2f56-116">開発者のエコシステムとチームのスキル</span><span class="sxs-lookup"><span data-stu-id="f2f56-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="f2f56-117">コンピューティングの比較表</span><span class="sxs-lookup"><span data-stu-id="f2f56-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="f2f56-118">アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。</span><span class="sxs-lookup"><span data-stu-id="f2f56-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="f2f56-119">完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="f2f56-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="f2f56-120">Azure でコンテナーをホストする際のオプションの詳細については、[Azure コンテナー](https://azure.microsoft.com/overview/containers/)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="f2f56-120">For more information about your options for hosting containers in Azure, see [Azure Containers](https://azure.microsoft.com/overview/containers/).</span></span>

## <a name="flowchart"></a><span data-ttu-id="f2f56-121">フローチャート</span><span class="sxs-lookup"><span data-stu-id="f2f56-121">Flowchart</span></span>

![Azure コンピューティング サービスのデシジョン ツリー](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="f2f56-123">定義</span><span class="sxs-lookup"><span data-stu-id="f2f56-123">Definitions</span></span>

- <span data-ttu-id="f2f56-124">**リフト アンド シフト:** アプリケーションの再設計やコード変更なしで、ワークロードをクラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="f2f56-124">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="f2f56-125">"*リホスト*" とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="f2f56-125">Also called *rehosting*.</span></span> <span data-ttu-id="f2f56-126">詳細については、「[Azure Migration Center](https://azure.microsoft.com/migration/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="f2f56-126">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="f2f56-127">**クラウド用に最適化:** アプリケーションをリファクタリングすることでクラウドネイティブの機能を利用して、クラウドに移行する戦略です。</span><span class="sxs-lookup"><span data-stu-id="f2f56-127">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f2f56-128">次の手順</span><span class="sxs-lookup"><span data-stu-id="f2f56-128">Next steps</span></span>

<span data-ttu-id="f2f56-129">考慮する必要がある追加条件については、「[Azure コンピューティング サービスを選択するための条件](./compute-comparison.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="f2f56-129">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
