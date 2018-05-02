---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="82592-103">Azure コンピューティング サービスのデシジョン ツリー</span><span class="sxs-lookup"><span data-stu-id="82592-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="82592-104">Azure では、複数の方法でお使いのアプリケーション コードをホストできます。</span><span class="sxs-lookup"><span data-stu-id="82592-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="82592-105">"*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。</span><span class="sxs-lookup"><span data-stu-id="82592-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="82592-106">次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="82592-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="82592-107">このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。</span><span class="sxs-lookup"><span data-stu-id="82592-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="82592-108">すべてのアプリケーションに固有の要件があるため、推奨事項は原案として使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="82592-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="82592-109">その後、詳細な分析を実行し、次の点を確認します。</span><span class="sxs-lookup"><span data-stu-id="82592-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="82592-110">機能セット</span><span class="sxs-lookup"><span data-stu-id="82592-110">Feature set</span></span>
- [<span data-ttu-id="82592-111">サービスの制限</span><span class="sxs-lookup"><span data-stu-id="82592-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="82592-112">コスト</span><span class="sxs-lookup"><span data-stu-id="82592-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="82592-113">SLA</span><span class="sxs-lookup"><span data-stu-id="82592-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="82592-114">リージョン別の提供状況</span><span class="sxs-lookup"><span data-stu-id="82592-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="82592-115">開発者のエコシステムとチームのスキル</span><span class="sxs-lookup"><span data-stu-id="82592-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="82592-116">コンピューティングの比較表</span><span class="sxs-lookup"><span data-stu-id="82592-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="82592-117">アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。</span><span class="sxs-lookup"><span data-stu-id="82592-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="82592-118">完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="82592-118">A complete solution may incorporate two or more compute services.</span></span>

