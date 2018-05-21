---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="fb5dc-103">Azure コンピューティング サービスのデシジョン ツリー</span><span class="sxs-lookup"><span data-stu-id="fb5dc-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="fb5dc-104">Azure では、複数の方法でお使いのアプリケーション コードをホストできます。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="fb5dc-105">"*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="fb5dc-106">次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="fb5dc-107">このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="fb5dc-108">**このフローチャートを原案として使用します。**</span><span class="sxs-lookup"><span data-stu-id="fb5dc-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="fb5dc-109">すべてのアプリケーションには固有の要件があるため、推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="fb5dc-110">その後、詳細な評価を実行し、次の点を確認します。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="fb5dc-111">機能セット</span><span class="sxs-lookup"><span data-stu-id="fb5dc-111">Feature set</span></span>
- [<span data-ttu-id="fb5dc-112">サービスの制限</span><span class="sxs-lookup"><span data-stu-id="fb5dc-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="fb5dc-113">コスト</span><span class="sxs-lookup"><span data-stu-id="fb5dc-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="fb5dc-114">SLA</span><span class="sxs-lookup"><span data-stu-id="fb5dc-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="fb5dc-115">リージョン別の提供状況</span><span class="sxs-lookup"><span data-stu-id="fb5dc-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="fb5dc-116">開発者のエコシステムとチームのスキル</span><span class="sxs-lookup"><span data-stu-id="fb5dc-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="fb5dc-117">コンピューティングの比較表</span><span class="sxs-lookup"><span data-stu-id="fb5dc-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="fb5dc-118">アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="fb5dc-119">完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="fb5dc-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

