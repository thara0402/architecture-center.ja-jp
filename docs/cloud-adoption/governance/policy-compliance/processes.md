---
title: CAF:ポリシーの準拠を監視し、強制する
description: 確立されたポリシーを確実に準拠する方法はありますか。
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
ms.openlocfilehash: 66df08b5ed66625c49907ac944f83d7af3ed1b53
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242473"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-processes-can-help-ensure-policy-adherence"></a><span data-ttu-id="a7f12-103">ポリシーに確実に準拠するにはどのようなプロセスが役に立ちますか。</span><span class="sxs-lookup"><span data-stu-id="a7f12-103">What processes can help ensure policy adherence?</span></span>

<!---
I've defined policies, I've provided an architecture guide. Now how do I monitor adherence to policy? If there is a violation, how do I enforce the policy?
--->

<span data-ttu-id="a7f12-104">クラウド ポリシー ステートメントを確立し、デザイン ガイドの草案を作成したら、ポリシー要件に従ってクラウドをデプロイするための戦略を作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-104">After establishing your cloud policy statements and drafting a design guide, you'll need to create a strategy for ensuring your cloud deployment stays in compliance with your policy requirements.</span></span> <span data-ttu-id="a7f12-105">この戦略では、クラウド ガバナンス チームの現行のレビューと通信のプロセスを網羅し、アクションを必要とするポリシー違反に関する基準と、違反を検出し修復アクションをトリガーさせる自動化された監視/コンプライアンス システムの要件を定義するための基準を確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-105">This strategy will need to encompass your Cloud Governance team's ongoing review and communication processes, establish criteria for when policy violations require action, and defining the requirements for automated monitoring and compliance systems that will detect violations and trigger remediation actions.</span></span>

<span data-ttu-id="a7f12-106">ポリシー準拠プロセスがクラウド ガバナンス プランに適合している例については、「[アクションにつながるガバナンス体験](../journeys/overview.md)」の企業ポリシーのセクションをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a7f12-106">See the corporate policy sections of the [actionable governance journeys](../journeys/overview.md) for examples of how policy adherence process fit into a cloud governance plan.</span></span>

## <a name="prioritize-policy-adherence-processes"></a><span data-ttu-id="a7f12-107">ポリシー準拠プロセスに優先順位を付ける</span><span class="sxs-lookup"><span data-stu-id="a7f12-107">Prioritize policy adherence processes</span></span>

<span data-ttu-id="a7f12-108">ポリシー目標に必要なプロセスの開発にはどのくらいの投資が必要になりますか。</span><span class="sxs-lookup"><span data-stu-id="a7f12-108">How much investment in developing processes is required to support your policy goals?</span></span> <span data-ttu-id="a7f12-109">コンプライアンスを支援するプロセスを確立するために必要な労力とその労力に付随するコストは、クラウド デプロイのサイズと成熟度によって大きく変わります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-109">Depending on the size and maturity of your cloud deployment, the effort required to establish processes that support compliance, and the costs associated with this effort, can vary widely.</span></span>

<span data-ttu-id="a7f12-110">開発リソースとテスト リソースからなる小規模のデプロイであれば、ポリシー要件がシンプルで、必要な専用リソースも少なくなる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-110">For small deployments consisting of development and test resources, policy requirements may be simple and require few dedicated resources to address.</span></span> <span data-ttu-id="a7f12-111">一方で、セキュリティとパフォーマンスの優先順位が高い、成熟したミッションクリティカルなクラウド デプロイであれば、場合によっては、スタッフからなるチーム、広範囲の内部プロセス、カスタムの監視ツールでポリシー目標を支援する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-111">On the other hand, a mature mission-critical cloud deployment with high-priority security and performance needs may require a team of staff, extensive internal processes, and custom monitoring tooling to support your policy goals.</span></span>

<span data-ttu-id="a7f12-112">ポリシー準拠戦略を定義する最初の手順として、下記のプロセスでポリシー要件をどのようにサポートできるかを評価します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-112">As a first step in defining your policy adherence strategy, evaluate how the processes discussed below can support your policy requirements.</span></span> <span data-ttu-id="a7f12-113">これらのプロセスへの投資にはどのくらいの労力がふさわしいか判断し、この情報を利用してニーズに合った現実的な予算計画と要員計画を立てます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-113">Determine how much effort is worth investing in these processes, and then use this information to establish realistic budget and staffing plans to meet these needs.</span></span>

## <a name="establish-cloud-governance-team-processes"></a><span data-ttu-id="a7f12-114">クラウド ガバナンス チーム プロセスを確立する</span><span class="sxs-lookup"><span data-stu-id="a7f12-114">Establish Cloud Governance team processes</span></span>

<span data-ttu-id="a7f12-115">ポリシーのコンプライアンスの改善トリガーを定義する前に、チームで利用する全体的なプロセスと、IT スタッフとクラウド ガバナンス チームとの間で情報を共有し、担当者や上司に報告する方法を確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-115">Before defining triggers for policy compliance remediation, you need establish the overall processes that your team will use and how information will be shared and escalated between IT staff and the Cloud Governance team.</span></span>

### <a name="assign-cloud-governance-team-members"></a><span data-ttu-id="a7f12-116">クラウド ガバナンスのチーム メンバーを割り当てる</span><span class="sxs-lookup"><span data-stu-id="a7f12-116">Assign Cloud Governance team members</span></span>

<span data-ttu-id="a7f12-117">ポリシーのコンプライアンスに関する現行のガイダンスを提供し、クラウド アセットをデプロイし、操作しているときに発生したポリシー関連の問題を処理するのは誰ですか。</span><span class="sxs-lookup"><span data-stu-id="a7f12-117">Who will provide ongoing guidance on policy compliance and handle policy-related issues that emerge when deploying and operating your cloud assets?</span></span> <span data-ttu-id="a7f12-118">クラウド ガバナンス チームの規模と構成は、ポリシー要件の複雑性と、ポリシーのコンプライアンスに関連付けた予算と要員の優先順位によって異なります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-118">The size and composition of your Cloud Governance team will depend on the complexity of your policy requirements, and the budgeting and staffing priorities you've attached to policy compliance.</span></span>

<span data-ttu-id="a7f12-119">定義したポリシー ステートメントでカバーされる領域で専門知識を持つチーム メンバーを選びます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-119">Choose team members that have expertise in the areas covered by your defined policy statements.</span></span> <span data-ttu-id="a7f12-120">初回のテスト デプロイの場合、これはガバナンスの基礎の確立を担当する少数のシステム管理者に限定できます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-120">For initial test deployments this can be limited to a few system administrators responsible for establishing the basics of governance.</span></span> <span data-ttu-id="a7f12-121">デプロイが十分に進行し、ポリシーが一層複雑になり、さらに増えた企業ポリシー要件との統合の度合いが増すと、ますます複雑になるポリシー要件に対応するため、クラウド ガバナンス チームを変える必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-121">As your deployments mature and your policies become more complex and more integrated with your wider corporate policy requirements, your Cloud Governance team will need to change to support increasingly complicated policy requirements.</span></span>

<span data-ttu-id="a7f12-122">ガバナンス プロセスが十分に進行したら、最新のポリシー要件に適切に対処できるよう、クラウド ガバナンス チームのメンバーシップを定期的にレビューします。</span><span class="sxs-lookup"><span data-stu-id="a7f12-122">As your governance processes mature, review the cloud guidance team's membership regularly to ensure that you can properly address the latest policy requirements.</span></span> <span data-ttu-id="a7f12-123">ガバナンスの特定の領域で専門知識がある、あるいはその領域に関心を持っている IT スタッフを見つけ、固定のメンバーとしてチームに入れるか、必要に応じて臨時でチームに加えます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-123">Identify members of your IT staff with relevant experience or interest in specific areas of governance and include them in your teams on a permanent or ad-hoc basis as-needed.</span></span>

### <a name="reviews-and-policy-iteration"></a><span data-ttu-id="a7f12-124">レビューとポリシーのイテレーション</span><span class="sxs-lookup"><span data-stu-id="a7f12-124">Reviews and policy iteration</span></span>

<span data-ttu-id="a7f12-125">追加のリソースがデプロイされたら、クラウド ガバナンス チームは新しいワークロードまたはアセットをポリシー要件に確実に準拠させるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-125">As additional resources are deployed, the Cloud Governance team will need to ensure that new workloads or assets comply with policy requirements.</span></span> <span data-ttu-id="a7f12-126">新しいリソースのデプロイを担当するチームと会議の場を設け、デザイン ガイドに合わせた調整について話し合う計画を立てます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-126">Plan to meet with the teams responsible for deploying any new resources to discuss alignment with your design guides.</span></span>

<span data-ttu-id="a7f12-127">全体的なデプロイの規模が大きくなったら、新しい潜在的リスクを定期的に評価し、必要に応じてポリシー ステートメントとデザイン ガイドを更新します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-127">As your overall deployment grows, evaluate new potential risks regularly and update policy statements and design guides as needed.</span></span> <span data-ttu-id="a7f12-128">ポリシーが常に最新の状態であり、順守されるよう、5 つのガバナンス分野のそれぞれに対し、定期的なレビュー サイクルをスケジュールします。</span><span class="sxs-lookup"><span data-stu-id="a7f12-128">Schedule regular review cycles each of the five governance disciplines to ensure policy is up-to-date and being met.</span></span>

### <a name="education"></a><span data-ttu-id="a7f12-129">教育</span><span class="sxs-lookup"><span data-stu-id="a7f12-129">Education</span></span>

<span data-ttu-id="a7f12-130">ポリシーのコンプライアンスには、IT スタッフと開発者がそれぞれの担当領域に影響を与えるポリシー要件を理解する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-130">Policy compliance requires IT staff and developers to understand the policy requirements that affect their areas of responsibility.</span></span> <span data-ttu-id="a7f12-131">意思決定や要件を記録するためのリソースを充て、ポリシー要件を支援するデザイン ガイドに関する知識を関連するすべてのチームに教育する計画を立てます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-131">Plan to devote resources to document decisions and requirements, and educate all relevant teams on the design guides that support your policy requirements.</span></span>

<span data-ttu-id="a7f12-132">ポリシーは変更されるので、定期的に文書や研修素材を作り直し、最新の要件やガイダンスがそれに関係する IT スタッフに確実に伝えられるように取り計らいます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-132">As policy changes, regularly update documentation and training materials, and ensure education efforts communicate updated requirements and guidance to relevant IT staff.</span></span>  

### <a name="establish-escalation-paths"></a><span data-ttu-id="a7f12-133">エスカレーション パスを確立する</span><span class="sxs-lookup"><span data-stu-id="a7f12-133">Establish escalation paths</span></span>

<span data-ttu-id="a7f12-134">あるリソースがポリシーには違反した場合、誰に通知しますか。</span><span class="sxs-lookup"><span data-stu-id="a7f12-134">If a resource goes out of compliance, who gets notified?</span></span> <span data-ttu-id="a7f12-135">IT スタッフがポリシーのコンプライアンスに関する問題を見つけた場合、誰に連絡しますか。</span><span class="sxs-lookup"><span data-stu-id="a7f12-135">If IT staff detect a policy compliance issue, who do they contact?</span></span> <span data-ttu-id="a7f12-136">クラウド ガバナンス チームへのエスカレーション プロセスを明確に定義します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-136">Make sure the escalation process to the Cloud Governance team is clearly defined.</span></span> <span data-ttu-id="a7f12-137">通信チャネルは常に最新の状態に維持し、スタッフや組織の変更を反映させます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-137">Ensure these communication channels are kept updated to reflect staff and organization changes.</span></span>

## <a name="violation-triggers-and-actions"></a><span data-ttu-id="a7f12-138">違反のトリガーとアクション</span><span class="sxs-lookup"><span data-stu-id="a7f12-138">Violation triggers and actions</span></span>

<span data-ttu-id="a7f12-139">クラウド ガバナンス チームとそのプロセスを定義したら、アクションをトリガーするコンプライアンス違反として見なされるものとそのアクションの内容を明示的に定義する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a7f12-139">After defining your Cloud Governance team and its processes, you need to explicitly define what qualifies as compliance violations that will triggers actions, and what those actions should be.</span></span>

### <a name="define-triggers"></a><span data-ttu-id="a7f12-140">トリガーを定義する</span><span class="sxs-lookup"><span data-stu-id="a7f12-140">Define triggers</span></span>

<span data-ttu-id="a7f12-141">ポリシー ステートメントごとに、ポリシー違反になる行為を判断する要件を確認します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-141">For each of your policy statements, review requirements to determine what constitutes a policy violation.</span></span> <span data-ttu-id="a7f12-142">ポリシー定義プロセスの一部として既に確立している情報を利用してトリガーを生成します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-142">Generate your triggers using the information you've already established as part of the policy definition process.</span></span>

* <span data-ttu-id="a7f12-143">リスク許容値 - [リスク許容値分析](risk-tolerance.md)の一環として確立したメトリックとインジケーターに基づき、違反トリガーを作成します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-143">Risk tolerance - Create violation triggers based on the metrics and risk indicators you established as part of your [risk tolerance analysis](risk-tolerance.md).</span></span>
* <span data-ttu-id="a7f12-144">定義されたポリシー要件 - ポリシー ステートメントには、サービス レベル アグリーメント (SLA)、事業継続とディザスター リカバリー (BRCD)、パフォーマンス要件など、コンプライアンス トリガーの基礎として使用するものを含めます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-144">Defined policy requirements - Policy statements may provide Service Level Agreement (SLA), Business continuity and disaster recovery (BRCD), or performance requirements that should be used as the basis for compliance triggers.</span></span>

### <a name="define-actions"></a><span data-ttu-id="a7f12-145">アクションを定義する</span><span class="sxs-lookup"><span data-stu-id="a7f12-145">Define actions</span></span>

<span data-ttu-id="a7f12-146">違反トリガーにはそれぞれ、それに対応するアクションを与えます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-146">Each violation trigger should have a corresponding action.</span></span> <span data-ttu-id="a7f12-147">違反が発生したら、トリガーされたアクションによって担当の IT スタッフまたはクラウド ガバナンス チーム メンバーに常に通知されるようにします。</span><span class="sxs-lookup"><span data-stu-id="a7f12-147">Triggered actions should always notify an appropriate IT staff or Cloud Governance team member when a violation occurs.</span></span> <span data-ttu-id="a7f12-148">この通知の後、検出された違反の種類や重大度に基づき、コンプライアンス問題を手動でレビューするか、既に確立されている是正プロセスを開始します。</span><span class="sxs-lookup"><span data-stu-id="a7f12-148">This notification can lead to a manual review of the compliance issue or kickoff a pre-established remediation process depending on the type and severity of the detected violation.</span></span>

<span data-ttu-id="a7f12-149">違反のトリガーとアクションの例:</span><span class="sxs-lookup"><span data-stu-id="a7f12-149">Some examples of violation triggers and actions:</span></span>

| <span data-ttu-id="a7f12-150">クラウド ガバナンスの分野</span><span class="sxs-lookup"><span data-stu-id="a7f12-150">Cloud governance discipline</span></span> | <span data-ttu-id="a7f12-151">サンプル トリガー</span><span class="sxs-lookup"><span data-stu-id="a7f12-151">Sample trigger</span></span> | <span data-ttu-id="a7f12-152">サンプル アクション</span><span class="sxs-lookup"><span data-stu-id="a7f12-152">Sample action</span></span> |
|-----------------------------|----------------|---------------|
| <span data-ttu-id="a7f12-153">Cost Management</span><span class="sxs-lookup"><span data-stu-id="a7f12-153">Cost Management</span></span> | <span data-ttu-id="a7f12-154">月間のクラウド支出が予想より 20% 以上多い。</span><span class="sxs-lookup"><span data-stu-id="a7f12-154">Monthly cloud spending is more than 20% higher than expected.</span></span> | <span data-ttu-id="a7f12-155">請求チームの主任に連絡し、その主任がリソース利用状況のレビューを開始する。</span><span class="sxs-lookup"><span data-stu-id="a7f12-155">Notify billing unit leader who will begin a review of resource usage.</span></span> |
| <span data-ttu-id="a7f12-156">セキュリティ ベースライン</span><span class="sxs-lookup"><span data-stu-id="a7f12-156">Security Baseline</span></span> | <span data-ttu-id="a7f12-157">不審なユーザー ログイン行為を検出する。</span><span class="sxs-lookup"><span data-stu-id="a7f12-157">Detect suspicious user login activity.</span></span> | <span data-ttu-id="a7f12-158">IT セキュリティ チームに連絡し、疑わしいユーザー アカウントを無効にする。</span><span class="sxs-lookup"><span data-stu-id="a7f12-158">Notify IT security team and disable suspect user account.</span></span> |
| <span data-ttu-id="a7f12-159">リソースの整合性</span><span class="sxs-lookup"><span data-stu-id="a7f12-159">Resource Consistency</span></span> | <span data-ttu-id="a7f12-160">ワークロードの CPU 使用率が 90% を超えている。</span><span class="sxs-lookup"><span data-stu-id="a7f12-160">CPU utilization for workload is greater than 90%.</span></span> | <span data-ttu-id="a7f12-161">IT 運用チームに連絡し、リソースをスケールアウトして負荷を処理する。</span><span class="sxs-lookup"><span data-stu-id="a7f12-161">Notify the IT Operations team and scale out additional resources to handle load.</span></span> |

## <a name="monitoring-and-compliance-automation"></a><span data-ttu-id="a7f12-162">監視とコンプライアンスの自動化</span><span class="sxs-lookup"><span data-stu-id="a7f12-162">Monitoring and compliance automation</span></span>

<span data-ttu-id="a7f12-163">コンプライアンス違反のトリガーとアクションを定義したら、監視とポリシーのコンプライアンスの戦略を自動化する目的で、クラウド プラットフォームのログ記録ツール、報告ツール、その他の機能を最も効果的に利用する方法を計画できます。</span><span class="sxs-lookup"><span data-stu-id="a7f12-163">After you've defined your compliance violation triggers and actions, you can start planning how best to use the logging and reporting tools and other features of the cloud platform to help automate your monitoring and policy compliance strategy.</span></span>

<span data-ttu-id="a7f12-164">ご利用のデプロイに最適な監視パターンを選択する方法については、CAF ガイダンスの「[ログ記録とレポートの意思決定ガイド](../../decision-guides/log-and-report/overview.md)」トピックを参照してください。</span><span class="sxs-lookup"><span data-stu-id="a7f12-164">Consult the CAF [logging and reporting decision guide](../../decision-guides/log-and-report/overview.md) topic for guidance on choosing the best monitoring pattern for your deployment.</span></span>
