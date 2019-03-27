---
title: CAF:Cost Management の目的とビジネス上のリスク
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Cost Management の目的とビジネス上のリスク
author: BrianBlanchard
ms.openlocfilehash: b9bf31a796ea1a7530c6a668a231d74b9b765509
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247473"
---
# <a name="cost-management-motivations-and-business-risks"></a><span data-ttu-id="3d832-103">Cost Management の目的とビジネス上のリスク</span><span class="sxs-lookup"><span data-stu-id="3d832-103">Cost Management motivations and business risks</span></span>

<span data-ttu-id="3d832-104">この記事では、クラウド ガバナンス戦略でお客様が Cost Management の規範を一般的に採用する理由について説明します。</span><span class="sxs-lookup"><span data-stu-id="3d832-104">This article discusses the reasons that customers typically adopt a Cost Management discipline within a cloud governance strategy.</span></span> <span data-ttu-id="3d832-105">ポリシー ステートメントを促進するビジネス リスクの例もいくつか提示します。</span><span class="sxs-lookup"><span data-stu-id="3d832-105">It also provides a few examples of business risks that drive policy statements.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="is-cost-management-relevant"></a><span data-ttu-id="3d832-106">Cost Management の関連性</span><span class="sxs-lookup"><span data-stu-id="3d832-106">Is Cost Management relevant?</span></span>

<span data-ttu-id="3d832-107">コストの管理の観点から見ると、クラウドの導入はパラダイム シフトをもたらします。</span><span class="sxs-lookup"><span data-stu-id="3d832-107">In terms of cost governance, cloud adoption creates a paradigm shift.</span></span> <span data-ttu-id="3d832-108">従来のオンプレミスの世界でのコスト管理は、更新サイクル、データ センターの取得、ホストの更新、および定期的なメンテナンスの問題に基づきます。</span><span class="sxs-lookup"><span data-stu-id="3d832-108">Management of cost in a traditional on-premises world is based on refresh cycles, datacenter acquisitions, host renewals, and recurring maintenance issues.</span></span> <span data-ttu-id="3d832-109">年間の設備投資予算に合わせて、これらの各コストを予測、計画、および調整できます。</span><span class="sxs-lookup"><span data-stu-id="3d832-109">You can forecast, plan, and refine each of these costs to align with annual capital expenditure budgets.</span></span>

<span data-ttu-id="3d832-110">クラウド ソリューションでは、多くの企業は Cost Management に対してより事後対応型の手法を採用する傾向があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-110">For cloud solutions, many businesses tend to take a more reactive approach to Cost Management.</span></span> <span data-ttu-id="3d832-111">多くの場合、企業は一定のクラウド サービスを事前購入するか、使用のコミットを行います。</span><span class="sxs-lookup"><span data-stu-id="3d832-111">In many cases, businesses will prepurchase, or commit to use, a set amount of cloud services.</span></span> <span data-ttu-id="3d832-112">このモデルでは、特定のクラウド ベンダーでの支出に関してビジネスが計画する金額に基づいて割引を最大化することで、事前対応の計画されたコスト サイクルが認識されると想定しています。</span><span class="sxs-lookup"><span data-stu-id="3d832-112">This model assumes that maximizing discounts, based on how much the business plans on spending with a specific cloud vendor, creates the perception of a proactive, planned cost cycle.</span></span> <span data-ttu-id="3d832-113">しかし、その認識が現実のものになるのは、ビジネスで、成熟した Cost Management の規範も実装している場合に限られます。</span><span class="sxs-lookup"><span data-stu-id="3d832-113">However, that perception will only become a reality if the business also implements mature Cost Management disciplines.</span></span>

<span data-ttu-id="3d832-114">クラウドでは、従来のオンプレミスのデータセンターでは前例のないセルフ サービス機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="3d832-114">The cloud offers self-service capabilities that were previously unheard of in traditional on-premises datacenters.</span></span> <span data-ttu-id="3d832-115">これらの新機能によって、ビジネスがより迅速に、より少ない制限で処理され、新しいテクノロジを採用しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="3d832-115">These new capabilities empower businesses to be more agile, less restrictive, and more open to adopt new technologies.</span></span> <span data-ttu-id="3d832-116">ただし、セルフ サービスの欠点は、エンド ユーザーが無意識のうちに割り当てられた予算を超える可能性があることです。</span><span class="sxs-lookup"><span data-stu-id="3d832-116">However, the downside of self-service is that end users can unknowingly exceed allocated budgets.</span></span> <span data-ttu-id="3d832-117">逆に、同じユーザーが計画の変更を経験し、クラウド サービスの予測金額を意外に使用しない可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="3d832-117">Conversely, the same users can experience a change in plans and unexpectedly not use the amount of cloud services forecasted.</span></span> <span data-ttu-id="3d832-118">どちらの方向にもシフトする可能性があるため、ガバナンス チーム内での Cost Management の規範に投資するだけの価値があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-118">The potential of shift in either direction justifies investment in a Cost Management discipline within the governance team.</span></span>

## <a name="business-risk"></a><span data-ttu-id="3d832-119">ビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="3d832-119">Business risk</span></span>

<span data-ttu-id="3d832-120">Cost Management の規範は、クラウド ベースのワークロードをホストする場合に伴う費用に関連するコア ビジネスのリスクに対処しようとします。</span><span class="sxs-lookup"><span data-stu-id="3d832-120">The Cost Management discipline attempts to address core business risks related to expenses incurred when hosting cloud-based workloads.</span></span> <span data-ttu-id="3d832-121">ビジネスとの連携により、これらのリスクを特定し、クラウドのデプロイを計画して実装したら、妥当性について各リスクを監視します。</span><span class="sxs-lookup"><span data-stu-id="3d832-121">Work with your business to identify these risks and monitor each of them for relevance as you plan for and implement your cloud deployments.</span></span>

<span data-ttu-id="3d832-122">リスクは組織によって異なりますが、クラウド ガバナンス チーム内でのディスカッションの開始点として使用できる一般的なコスト関連のリスクとして以下を利用できます。</span><span class="sxs-lookup"><span data-stu-id="3d832-122">Risks will differ between organization, but the following serve as common cost-related risks that you can use as a starting point for discussions within your Cloud Governance team:</span></span>

- <span data-ttu-id="3d832-123">**予算管理**。</span><span class="sxs-lookup"><span data-stu-id="3d832-123">**Budget control**.</span></span> <span data-ttu-id="3d832-124">予算を管理しないと、クラウド ベンダーとの間で過剰な支出が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-124">Not controlling budget can lead to excessive spending with a cloud vendor.</span></span>
- <span data-ttu-id="3d832-125">**使用の損失**。</span><span class="sxs-lookup"><span data-stu-id="3d832-125">**Utilization loss**.</span></span> <span data-ttu-id="3d832-126">使用されていない事前購入や事前コミットメントは、無駄な投資となる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-126">Prepurchases or precommitments that are not used can result in wasted investments.</span></span>
- <span data-ttu-id="3d832-127">**異常な支出**。いずれかの方向への予期せぬ急上昇は、不適切な使用のインジケーターとなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-127">**Spending anomalies**: Unexpected spikes in either direction can be indicators of improper usage.</span></span>
- <span data-ttu-id="3d832-128">**オーバープロビジョニングの資産**。</span><span class="sxs-lookup"><span data-stu-id="3d832-128">**Overprovisioned assets**.</span></span> <span data-ttu-id="3d832-129">資産がアプリケーションまたは仮想マシン (VM) のニーズを超える構成でデプロイされると、コストが増加して、資産が無駄になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3d832-129">When assets are deployed in a configuration that exceed the needs of an application or virtual machine (VM), they can increase costs and create waste.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3d832-130">次の手順</span><span class="sxs-lookup"><span data-stu-id="3d832-130">Next steps</span></span>

<span data-ttu-id="3d832-131">[クラウド管理テンプレート](./template.md)を使用して、現在のクラウド導入計画によって生じる可能性の高いビジネス リスクを文書化します。</span><span class="sxs-lookup"><span data-stu-id="3d832-131">Using the [Cloud Management Template](./template.md), document business risks that are likely to be introduced by the current cloud adoption plan.</span></span>

<span data-ttu-id="3d832-132">現実的なビジネス リスクについての理解が確実に得られたら、リスクに対する企業の許容範囲と、その許容範囲を監視するための指標と主なメトリックについて文書化することが、次の手順となります。</span><span class="sxs-lookup"><span data-stu-id="3d832-132">Once an understanding of realistic business risks is established, the next step is to document the business's tolerance for risk and the indicators and key metrics to monitor that tolerance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="3d832-133">指標、メトリック、リスクの許容範囲について理解する</span><span class="sxs-lookup"><span data-stu-id="3d832-133">Understand indicators, metrics, and risk tolerance</span></span>](./metrics-tolerance.md)
