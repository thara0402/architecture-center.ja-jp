---
title: CAF:コスト管理のサンプル ポリシー ステートメント
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 1/4/2019
description: コスト管理のサンプル ポリシー ステートメント
author: BrianBlanchard
ms.openlocfilehash: 1c8b94ae5285fa26cdf9760892beaced2487af8b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902057"
---
# <a name="cost-management-sample-policy-statements"></a><span data-ttu-id="a9430-103">コスト管理のサンプル ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="a9430-103">Cost Management sample policy statements</span></span>

<span data-ttu-id="a9430-104">個々のクラウド ポリシー ステートメントは、リスクの評価プロセスで識別された特定のリスクに対処するためのガイドラインとなります。</span><span class="sxs-lookup"><span data-stu-id="a9430-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="a9430-105">これらのステートメントでは、リスクとそれらのリスクに対処する計画の簡潔な概要を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="a9430-106">各ステートメント定義には、以下の情報を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="a9430-107">ビジネス リスク - このポリシーで対処されるリスクの概要。</span><span class="sxs-lookup"><span data-stu-id="a9430-107">Business risk - A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="a9430-108">ポリシー ステートメント - ポリシー要件の端的な概要説明。</span><span class="sxs-lookup"><span data-stu-id="a9430-108">Policy statement - A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="a9430-109">設計オプション - 実行可能な推奨事項、仕様、または IT チームおよび開発者がポリシーの実装時に使用できるその他のガイダンス。</span><span class="sxs-lookup"><span data-stu-id="a9430-109">Design options - Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="a9430-110">次のサンプル ポリシー ステートメントでは、コストに関連する複数の一般的なビジネス リスクに対処し、組織のニーズに対応する実際のポリシー ステートメントのドラフトを作成するときに参照する例として提供されます。</span><span class="sxs-lookup"><span data-stu-id="a9430-110">The following sample policy statements address a number of common cost-related business risks, and are provided as examples for you to reference when drafting actual policy statements addressing your own organization's needs.</span></span> <span data-ttu-id="a9430-111">これらの例は、規制的であることを意図しておらず、単一の特定されたリスクに対処するために複数のポリシー オプションが存在する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-111">These examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any single identified risk.</span></span> <span data-ttu-id="a9430-112">ビジネス チームおよび IT チームと密接に連携して、特定のコストに関するリスクに最適なポリシー ソリューションを識別します。</span><span class="sxs-lookup"><span data-stu-id="a9430-112">Work closely with business and IT teams to identify the best policy solutions for your particular cost-related risks.</span></span>  

## <a name="future-proofing"></a><span data-ttu-id="a9430-113">将来性</span><span class="sxs-lookup"><span data-stu-id="a9430-113">Future-proofing</span></span>

<span data-ttu-id="a9430-114">**ビジネス リスク**。</span><span class="sxs-lookup"><span data-stu-id="a9430-114">**Business risk**.</span></span> <span data-ttu-id="a9430-115">ガバナンス チームからコスト管理規範への投資が保証されない現在の条件。</span><span class="sxs-lookup"><span data-stu-id="a9430-115">Current criteria that don't warrant an investment in a Cost Management discipline from the governance team.</span></span> <span data-ttu-id="a9430-116">しかし、将来、そのような投資が予測されます。</span><span class="sxs-lookup"><span data-stu-id="a9430-116">However, you anticipate such an investment in the future.</span></span>

<span data-ttu-id="a9430-117">**ポリシー ステートメント**。</span><span class="sxs-lookup"><span data-stu-id="a9430-117">**Policy statement**.</span></span> <span data-ttu-id="a9430-118">クラウドにデプロイされるすべての資産を課金単位、アプリケーション/ワークロードに関連付け、名前付け標準を満たす必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-118">You should associate all assets deployed to the cloud with a billing unit, application/workload, and meet naming standards.</span></span> <span data-ttu-id="a9430-119">このポリシーにより、将来のコスト管理の取り組みが確実に効果的なものになります。</span><span class="sxs-lookup"><span data-stu-id="a9430-119">This policy will ensure that future Cost Management efforts will be effective.</span></span>

<span data-ttu-id="a9430-120">**設計オプション**。</span><span class="sxs-lookup"><span data-stu-id="a9430-120">**Design options**.</span></span> <span data-ttu-id="a9430-121">将来性のある基盤の構築については、CAF ガイダンスの一部として含まれる、[実行可能な設計ガイド](../journeys/overview.md)に関するページのガバナンス MVP の作成に関する説明を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a9430-121">For information on establishing a future-proof foundation, see the discussions related to creating a governance MVP in the [actionable design guides](../journeys/overview.md) included as part of the CAF guidance.</span></span>

## <a name="budget-overruns"></a><span data-ttu-id="a9430-122">予算超過</span><span class="sxs-lookup"><span data-stu-id="a9430-122">Budget overruns</span></span>

<span data-ttu-id="a9430-123">**ビジネス リスク:** セルフ サービス デプロイには、予算オーバーというリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-123">**Business risk:** Self-service deployment creates a risk of overspending.</span></span>

<span data-ttu-id="a9430-124">**ポリシー ステートメント:** 承認済み予算と予算制限のメカニズムを使用して、請求単位にクラウド デプロイを割り当てる必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-124">**Policy statement:** Any cloud deployment must be allocated to a billing unit with approved budget and a mechanism for budgetary limits.</span></span>

<span data-ttu-id="a9430-125">**設計オプション:** Azure では、[Azure Cost Management](/azure/cost-management/manage-budgets) を使用して予算を制御することができます</span><span class="sxs-lookup"><span data-stu-id="a9430-125">**Design options:** In Azure, budget can be controlled with [Azure Cost Management](/azure/cost-management/manage-budgets)</span></span>

## <a name="underutilization"></a><span data-ttu-id="a9430-126">過小使用</span><span class="sxs-lookup"><span data-stu-id="a9430-126">Underutilization</span></span>

<span data-ttu-id="a9430-127">**ビジネス リスク:** 会社はクラウド サービスの料金を一括で払っているか、年間コミットメントを利用して一定額を支払っています。</span><span class="sxs-lookup"><span data-stu-id="a9430-127">**Business risk:** The company has prepaid for cloud services or has made an annual commitment to spend a specific amount.</span></span> <span data-ttu-id="a9430-128">合意された金額が使用されず、その結果、投資を損失するリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-128">There is a risk that the agreed upon amount won't be used, resulting in a lost investment.</span></span>

<span data-ttu-id="a9430-129">**ポリシー ステートメント:** クラウド予算が割り当てられた各請求単位を集め、年に一度、予算を設定し、四半期ごとに予算を調整し、毎月、計画された支出と実際の支出を確認するための時間を割り当てます。</span><span class="sxs-lookup"><span data-stu-id="a9430-129">**Policy statement:** Each billing unit with an allocated cloud budget will meet annually to set budgets, quarterly to adjust budgets, and monthly to allocate time for reviewing planned versus actual spending.</span></span> <span data-ttu-id="a9430-130">毎月、請求単位のリーダーと 20% を超える偏差について話し合います。</span><span class="sxs-lookup"><span data-stu-id="a9430-130">Discuss any deviations greater than 20% with the billing unit leader monthly.</span></span> <span data-ttu-id="a9430-131">追跡目的で、すべての資産を課金単位に割り当てます。</span><span class="sxs-lookup"><span data-stu-id="a9430-131">For tracking purposes, assign all assets to a billing unit.</span></span>

<span data-ttu-id="a9430-132">**設計オプション:**</span><span class="sxs-lookup"><span data-stu-id="a9430-132">**Design options:**</span></span>

- <span data-ttu-id="a9430-133">Azure では、計画された支出と実際の支出を、[Azure Cost Management](/azure/cost-management/quick-acm-cost-analysis) を使用して管理することができます</span><span class="sxs-lookup"><span data-stu-id="a9430-133">In Azure, planned versus actual spending can be managed via [Azure Cost Management](/azure/cost-management/quick-acm-cost-analysis)</span></span>
- <span data-ttu-id="a9430-134">課金単位でリソースをグループ化するためのオプションがいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-134">There are several options for grouping resources by billing unit.</span></span> <span data-ttu-id="a9430-135">Azure で、ガバナンス チームと協力して、[リソースの整合性モデル](../../decision-guides/resource-consistency/overview.md)を選択し、すべての資産に適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-135">In Azure, a [resource consistency model](../../decision-guides/resource-consistency/overview.md) should be chosen in conjunction with the governance team and applied to all assets.</span></span>

## <a name="overprovisioned-assets"></a><span data-ttu-id="a9430-136">オーバープロビジョニングの資産</span><span class="sxs-lookup"><span data-stu-id="a9430-136">Overprovisioned assets</span></span>

<span data-ttu-id="a9430-137">**ビジネス リスク:** 従来のオンプレミス データセンターでは、遠い将来の成長に備えて余分なキャパシティ プランニングを行って資産をデプロイするのが一般的な方法です。</span><span class="sxs-lookup"><span data-stu-id="a9430-137">**Business risk:** In traditional on-premises datacenters, it is common practice to deploy assets with extra capacity planning for growth in the distant future.</span></span> <span data-ttu-id="a9430-138">クラウドでは、従来の機器より迅速にスケーリングすることができます。</span><span class="sxs-lookup"><span data-stu-id="a9430-138">The cloud can scale more quickly than traditional equipment.</span></span> <span data-ttu-id="a9430-139">クラウド内の資産も技術的なキャパシティに基づいて課金されます。</span><span class="sxs-lookup"><span data-stu-id="a9430-139">Assets in the cloud are also priced based on the technical capacity.</span></span> <span data-ttu-id="a9430-140">クラウド支出を人為的に増やす古いオンプレミスの方法にはリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-140">There is a risk of the old on-premises practice artificially inflating cloud spending.</span></span>

<span data-ttu-id="a9430-141">**ポリシー ステートメント:** クラウドにデプロイされた資産は、使用率を監視し、使用率が 50% を超えるキャパシティを報告できるプログラムに登録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-141">**Policy statement:** Any asset deployed to the cloud must be enrolled in a program that can monitor utilization and report any capacity in excess of 50% of utilization.</span></span> <span data-ttu-id="a9430-142">クラウドにデプロイされた資産は、論理的な方法でグループ化またはタグ付けする必要があります。これにより、ガバナンス チームのメンバーは、オーバープロビジョニングの資産の最適化について、ワークロード所有者を関与させることができます。</span><span class="sxs-lookup"><span data-stu-id="a9430-142">Any asset deployed to the cloud must be grouped or tagged in a logical manner, so governance team members can engage the workload owner regarding any optimization of overprovisioned assets.</span></span>

<span data-ttu-id="a9430-143">**設計オプション:**</span><span class="sxs-lookup"><span data-stu-id="a9430-143">**Design options:**</span></span>

- <span data-ttu-id="a9430-144">Azure では、[Azure Advisor](/azure/advisor/advisor-cost-recommendations) により、最適化に関する推奨事項を提供することができます。</span><span class="sxs-lookup"><span data-stu-id="a9430-144">In Azure, [Azure Advisor](/azure/advisor/advisor-cost-recommendations) can provide optimization recommendations.</span></span>
- <span data-ttu-id="a9430-145">課金単位でリソースをグループ化するためのオプションがいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-145">There are several options for grouping resources by billing unit.</span></span> <span data-ttu-id="a9430-146">Azure で、ガバナンス チームと協力して、[リソースの整合性モデル](../../decision-guides/resource-consistency/overview.md)を選択し、すべての資産に適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-146">In Azure, a [resource consistency model](../../decision-guides/resource-consistency/overview.md) should be chosen in conjunction with the governance team and applied to all assets.</span></span>

## <a name="overoptimization"></a><span data-ttu-id="a9430-147">過剰な最適化</span><span class="sxs-lookup"><span data-stu-id="a9430-147">Overoptimization</span></span>

<span data-ttu-id="a9430-148">**ビジネス リスク:** 効果的なコスト管理では、実際に新たなリスクが生じる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-148">**Business risk:** Effective cost management can actually create new risks.</span></span> <span data-ttu-id="a9430-149">支出の最適化は、システム パフォーマンスとは反比例します。</span><span class="sxs-lookup"><span data-stu-id="a9430-149">Optimization of spending is inverse to system performance.</span></span> <span data-ttu-id="a9430-150">コストを削減すると、支出を抑え過ぎ、ユーザー エクスペリエンスが低下するリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-150">When reducing costs, there is a risk of overtightening spending and producing poor user experiences.</span></span>

<span data-ttu-id="a9430-151">**ポリシー ステートメント:** カスタマー エクスペリエンスに直接影響するすべての資産を、グループ化またはタグ付けによって識別する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-151">**Policy statement:** Any asset that directly affects customer experiences must be identified through grouping or tagging.</span></span> <span data-ttu-id="a9430-152">カスタマー エクスペリエンスに影響する資産を最適化する前に、クラウド ガバナンス チームは、90 日以上の使用率の傾向に基づいて、最適化を調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-152">Before optimizing any asset that affects customer experience, the Cloud Governance team must adjust optimization based on no fewer than 90 days of utilization trends.</span></span> <span data-ttu-id="a9430-153">資産を最適化する際に考慮される、季節またはイベント ドリブンの急増を文書化します。</span><span class="sxs-lookup"><span data-stu-id="a9430-153">Document any seasonal or event driven bursts considered when optimizing assets.</span></span>

<span data-ttu-id="a9430-154">**設計オプション:**</span><span class="sxs-lookup"><span data-stu-id="a9430-154">**Design options:**</span></span>

- <span data-ttu-id="a9430-155">Azure では、[Azure Monitor の Insights 機能](/azure/azure-monitor/insights/vminsights-performance)が、システム利用率の分析に役立つ場合があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-155">In Azure, [Azure Monitor's insights features](/azure/azure-monitor/insights/vminsights-performance) can help with analysis of system utilization.</span></span>
- <span data-ttu-id="a9430-156">ロールに基づいて、リソースをグループ化およびタグ付けするためのオプションがいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="a9430-156">There are several options for grouping and tagging resources based on roles.</span></span> <span data-ttu-id="a9430-157">Azure で、ガバナンス チームと協力して、[リソースの整合性モデル](../../decision-guides/resource-consistency/overview.md)を選択し、すべての資産にこれを適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a9430-157">In Azure, you should choose a [resource consistency model](../../decision-guides/resource-consistency/overview.md) in conjunction with the governance team and apply this to all assets.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a9430-158">次の手順</span><span class="sxs-lookup"><span data-stu-id="a9430-158">Next steps</span></span>

<span data-ttu-id="a9430-159">手始めに、この記事で説明されているサンプルを使用して、クラウドの導入計画に合致する特定のビジネス リスクに対処するポリシーを作成します。</span><span class="sxs-lookup"><span data-stu-id="a9430-159">Use the samples mentioned in this article as a starting point to develop policies that address specific business risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="a9430-160">コスト管理に関連する独自のカスタム ポリシー ステートメントの作成を開始するには、[コスト管理テンプレート](template.md)をダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="a9430-160">To begin developing your own custom policy statements related to Cost Management, download the [Cost Management template](template.md).</span></span>

<span data-ttu-id="a9430-161">この規範の導入を促進するには、ご使用の環境に最も合う[アクションにつながるガバナンス体験](../journeys/overview.md)を選択します。</span><span class="sxs-lookup"><span data-stu-id="a9430-161">To accelerate adoption of this discipline, choose the [Actionable Governance Journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="a9430-162">その後、設計を変更して、特定の企業ポリシーの決定を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="a9430-162">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="a9430-163">アクションにつながるガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="a9430-163">Actionable Governance Journeys</span></span>](../journeys/overview.md)
