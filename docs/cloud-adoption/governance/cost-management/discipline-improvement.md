---
title: CAF:コスト管理規範の改善
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: コスト管理規範の改善
author: BrianBlanchard
ms.openlocfilehash: 34975d195a95b1a85ada96efe8c76a6138385ec1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902161"
---
# <a name="cost-management-discipline-improvement"></a><span data-ttu-id="2431f-103">コスト管理規範の改善</span><span class="sxs-lookup"><span data-stu-id="2431f-103">Cost Management discipline improvement</span></span>

<span data-ttu-id="2431f-104">コスト管理規範で、クラウドベースのワークロードをホストする場合に発生する費用に関連するコア ビジネス リスクへの対処を試みます。</span><span class="sxs-lookup"><span data-stu-id="2431f-104">The Cost Management discipline attempts to address core business risks related to expenses incurred when hosting cloud-based workloads.</span></span> <span data-ttu-id="2431f-105">クラウド ガバナンスの 5 つの規範の中で、コスト管理は、計画的なコスト サイクルを作成し維持することを目的として、クラウド リソースのコストと使用の管理に関わっています。</span><span class="sxs-lookup"><span data-stu-id="2431f-105">Within the five disciplines of Cloud Governance, Cost Management is involved in controlling cost and usage of cloud resources with the goal of creating and maintaining a planned cost cycle.</span></span>

<span data-ttu-id="2431f-106">この記事では、コスト管理規範を作成し、成熟させるために会社が実行できる可能性があるタスクについて概要を説明します。</span><span class="sxs-lookup"><span data-stu-id="2431f-106">This article outlines potential tasks your company perform to develop and mature your Cost Management discipline.</span></span> <span data-ttu-id="2431f-107">これらのタスクは、クラウド ソリューション実装の計画、構築、導入、および運用フェーズに分類できます。これらのフェーズは繰り返され、[クラウド ガバナンスへの増分アプローチ](../journeys/overview.md#an-incremental-approach-to-cloud-governance)を開発できるようになります。</span><span class="sxs-lookup"><span data-stu-id="2431f-107">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![導入の 4 つのフェーズ](../../_images/adoption-phases.png)

<span data-ttu-id="2431f-109">*図 1: クラウド ガバナンスへの増分アプローチの導入フェーズ。*</span><span class="sxs-lookup"><span data-stu-id="2431f-109">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="2431f-110">1 つのドキュメントですべてのビジネスの要件を説明することはできません。</span><span class="sxs-lookup"><span data-stu-id="2431f-110">No single document can account for the requirements of all businesses.</span></span> <span data-ttu-id="2431f-111">そのため、この記事では、ガバナンス成熟プロセスのフェーズごとに推奨される最小限のアクティビティと潜在的なアクティビティの例について概要を説明します。</span><span class="sxs-lookup"><span data-stu-id="2431f-111">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="2431f-112">これらのアクティビティの初期の目標は、ユーザーが[ポリシー MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) を構築し、増分型のポリシー進化のためのフレームワークを確立できるように支援することです。</span><span class="sxs-lookup"><span data-stu-id="2431f-112">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="2431f-113">クラウド ガバナンス チームは、コスト管理ガバナンス機能を強化するために、これらのアクティビティにどれだけ投資するかを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2431f-113">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Cost Management governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="2431f-114">この記事で概要を説明する最小限のアクティビティと潜在的なアクティビティのどちらも、特定の企業ポリシーやサードパーティのコンプライアンス要件には整合していません。</span><span class="sxs-lookup"><span data-stu-id="2431f-114">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="2431f-115">このガイダンスは、両方の要件とクラウド ガバナンス モデルの整合に向けての話し合いを促進するために設計されています。</span><span class="sxs-lookup"><span data-stu-id="2431f-115">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="2431f-116">計画と準備状況</span><span class="sxs-lookup"><span data-stu-id="2431f-116">Planning and readiness</span></span>

<span data-ttu-id="2431f-117">ガバナンス成熟のこのフェーズは、ビジネス成果とアクションにつながる戦略の間にあるギャップの橋渡しをします。</span><span class="sxs-lookup"><span data-stu-id="2431f-117">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="2431f-118">このプロセス中に、リーダーシップ チームは具体的なメトリックを定義し、そのメトリックをデジタル資産にマップして、全体的な移行作業の計画を開始します。</span><span class="sxs-lookup"><span data-stu-id="2431f-118">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="2431f-119">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-119">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="2431f-120">[コスト管理ツールチェーン](toolchain.md)の選択肢を評価します。</span><span class="sxs-lookup"><span data-stu-id="2431f-120">Evaluate your [Cost Management toolchain](toolchain.md) options.</span></span>
* <span data-ttu-id="2431f-121">下書きのアーキテクチャ ガイドライン ドキュメントを作成し、主な関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="2431f-121">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="2431f-122">アーキテクチャ ガイドラインの開発によって影響を受ける人物とチームを教育し、関与させます。</span><span class="sxs-lookup"><span data-stu-id="2431f-122">Educate and involve the people and teams affected by the development of Architecture Guidelines.</span></span>

<span data-ttu-id="2431f-123">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-123">**Potential activities:**</span></span>

* <span data-ttu-id="2431f-124">クラウド戦略に対して業務の妥当性を裏付ける予算上の決定を確実に行います。</span><span class="sxs-lookup"><span data-stu-id="2431f-124">Ensure budgetary decisions that support the business justification for your cloud strategy.</span></span>
* <span data-ttu-id="2431f-125">資金配分の成功に関する報告に使用する学習メトリックを検証します。</span><span class="sxs-lookup"><span data-stu-id="2431f-125">Validate learning metrics that you use to report on the successful allocation of funding.</span></span>
* <span data-ttu-id="2431f-126">クラウド コストの計算方法に影響する、目的のクラウド アカウンティング モデルを理解します。</span><span class="sxs-lookup"><span data-stu-id="2431f-126">Understand the desired cloud accounting model that affects how cloud costs should be accounted for.</span></span>
* <span data-ttu-id="2431f-127">デジタル資産プランをよく理解し、正確な原価計算を検証します。</span><span class="sxs-lookup"><span data-stu-id="2431f-127">Become familiar with the digital estate plan and validate accurate costing expectations.</span></span>
* <span data-ttu-id="2431f-128">購入の選択肢を評価して、"従量課金制" にする方法と、エンタープライズ契約を購入して事前にコミットする方法のどちらが良いかを判断します。</span><span class="sxs-lookup"><span data-stu-id="2431f-128">Evaluate buying options to determine if it's better to "pay as you go" or to make a precommitment by purchasing an Enterprise Agreement.</span></span>
* <span data-ttu-id="2431f-129">計画された予算にビジネスの目標を合わせ、必要に応じて予算計画を調整します。</span><span class="sxs-lookup"><span data-stu-id="2431f-129">Align business goals with planned budgets and adjust budgetary plans as necessary.</span></span>
* <span data-ttu-id="2431f-130">各コスト サイクルの最後に、技術およびビジネスの関係者に通知するための目標と予算の報告メカニズムを作成します。</span><span class="sxs-lookup"><span data-stu-id="2431f-130">Develop a goals and budget reporting mechanism to notify technical and business stakeholders at the end of each cost cycle.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="2431f-131">構築とデプロイ前準備</span><span class="sxs-lookup"><span data-stu-id="2431f-131">Build and pre-deployment</span></span>

<span data-ttu-id="2431f-132">環境を適切に移行するには、いくつかの技術的な前提条件と技術面以外の前提条件が必要です。</span><span class="sxs-lookup"><span data-stu-id="2431f-132">A number of technical and nontechnical prerequisites are required to successfully migrate an environment.</span></span> <span data-ttu-id="2431f-133">このプロセスでは、移行を進める意思決定、準備状況、およびコア インフラストラクチャに重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="2431f-133">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="2431f-134">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-134">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="2431f-135">デプロイ前フェーズでロールアウトすることによって、[コスト管理ツールチェーン](toolchain.md)を実装します。</span><span class="sxs-lookup"><span data-stu-id="2431f-135">Implement your [Cost Management toolchain](toolchain.md) by rolling out in a pre-deployment phase.</span></span>
* <span data-ttu-id="2431f-136">アーキテクチャ ガイドライン ドキュメントを更新し、主な関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="2431f-136">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="2431f-137">ユーザー導入の促進に役立つ教材やドキュメント、認識の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="2431f-137">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>
* <span data-ttu-id="2431f-138">購入要件が予算と目標に沿っているかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="2431f-138">Determine if your purchase requirements align with your budgets and goals.</span></span>

<span data-ttu-id="2431f-139">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-139">**Potential activities:**</span></span>

* <span data-ttu-id="2431f-140">予算計画を、コア所有モデルを定義する[サブスクリプション戦略](../../decision-guides/subscriptions/overview.md)と合わせます。</span><span class="sxs-lookup"><span data-stu-id="2431f-140">Align your budgetary plans with the [Subscription Strategy](../../decision-guides/subscriptions/overview.md) that defines your core ownership model.</span></span>
* <span data-ttu-id="2431f-141">[リソース整合性戦略](../../decision-guides/resource-consistency/overview.md)を利用して、アーキテクチャとコストのガイドラインを徐々に適用します。</span><span class="sxs-lookup"><span data-stu-id="2431f-141">Leverage the [Resource Consistency Strategy](../../decision-guides/resource-consistency/overview.md) to enforce architecture and cost guidelines over time.</span></span>
* <span data-ttu-id="2431f-142">導入計画と移行計画に影響を与えるコストの異常がないかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="2431f-142">Determine if there are any cost anomalies that affect your adoption and migration plans.</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="2431f-143">導入と移行</span><span class="sxs-lookup"><span data-stu-id="2431f-143">Adopt and migrate</span></span>

<span data-ttu-id="2431f-144">移行は、既存のデジタル資産内のアプリケーションまたはワークロードの移動、テスト、および導入に重点を置いた増分プロセスです。</span><span class="sxs-lookup"><span data-stu-id="2431f-144">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="2431f-145">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-145">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="2431f-146">[コスト管理ツールチェーン](toolchain.md)をデプロイ前準備から運用環境に移行します。</span><span class="sxs-lookup"><span data-stu-id="2431f-146">Migrate your [Cost Management toolchain](toolchain.md) from pre-deployment to production.</span></span>
* <span data-ttu-id="2431f-147">アーキテクチャ ガイドライン ドキュメントを更新し、主な関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="2431f-147">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="2431f-148">ユーザー導入の促進に役立つ教材やドキュメント、認識の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="2431f-148">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="2431f-149">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-149">**Potential activities:**</span></span>

* <span data-ttu-id="2431f-150">クラウド アカウンティング モデルを実装します。</span><span class="sxs-lookup"><span data-stu-id="2431f-150">Implement your Cloud Accounting Model.</span></span>
* <span data-ttu-id="2431f-151">リリースごとに予算に実際の支出が反映されていることを確認し、必要に応じて調整します。</span><span class="sxs-lookup"><span data-stu-id="2431f-151">Ensure that your budgets reflect your actual spending during each release and adjust as necessary.</span></span>
* <span data-ttu-id="2431f-152">予算計画の変化を監視し、追加のサインオフが必要な場合は関係者に確認を取ります。</span><span class="sxs-lookup"><span data-stu-id="2431f-152">Monitor changes in budgetary plans and validate with stakeholders if additional sign-offs are needed.</span></span>
* <span data-ttu-id="2431f-153">アーキテクチャ ガイドラインのドキュメントを更新して実際のコストを反映します。</span><span class="sxs-lookup"><span data-stu-id="2431f-153">Update changes to the Architecture Guidelines document to reflect actual costs.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="2431f-154">運用と実装後</span><span class="sxs-lookup"><span data-stu-id="2431f-154">Operate and post-implementation</span></span>

<span data-ttu-id="2431f-155">変換が完了したら、アプリケーションまたはワークロードの自然なライフサイクルに対してガバナンスと運用を続行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2431f-155">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="2431f-156">ガバナンス成熟のこのフェーズでは、一般にソリューションが実装され、変換サイクルが安定し始めた後に実行されるアクティビティに重点を置きます。</span><span class="sxs-lookup"><span data-stu-id="2431f-156">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="2431f-157">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-157">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="2431f-158">組織のコスト管理のニーズの変化に基づいて[コスト管理ツールチェーン](toolchain.md)をカスタマイズします。</span><span class="sxs-lookup"><span data-stu-id="2431f-158">Customize your [Cost Management toolchain](toolchain.md) based on changes in your organization’s cost management needs.</span></span>
* <span data-ttu-id="2431f-159">通知やレポート (ある場合) を自動化して実際の支出を反映することを検討します。</span><span class="sxs-lookup"><span data-stu-id="2431f-159">Consider automating any notifications and reports to reflect actual spending.</span></span>
* <span data-ttu-id="2431f-160">将来の導入プロセスの案内となるように、アーキテクチャ ガイドラインを改良します。</span><span class="sxs-lookup"><span data-stu-id="2431f-160">Refine Architecture Guidelines to guide future adoption processes.</span></span>
* <span data-ttu-id="2431f-161">アーキテクチャ ガイドラインに常に準拠するように、影響を受けるチームの教育を定期的に行います。</span><span class="sxs-lookup"><span data-stu-id="2431f-161">Educate affected teams on a periodic basis to ensure ongoing adherence to the Architecture Guidelines.</span></span>

<span data-ttu-id="2431f-162">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="2431f-162">**Potential activities:**</span></span>

* <span data-ttu-id="2431f-163">四半期ごとにクラウド ビジネス レビューを実行し、ビジネスにもたらされる価値とそれに関連するコストを伝えます。</span><span class="sxs-lookup"><span data-stu-id="2431f-163">Execute a quarterly cloud business review to communicate value delivered to the business and associated costs.</span></span>
* <span data-ttu-id="2431f-164">四半期ごとにプランを調整して実際の支出への変更を反映します。</span><span class="sxs-lookup"><span data-stu-id="2431f-164">Adjust plans quarterly to reflect changes to actual spending.</span></span>
* <span data-ttu-id="2431f-165">事業部門のサブスクリプションについて、損益に対する財務上の調整を判断します。</span><span class="sxs-lookup"><span data-stu-id="2431f-165">Determine financial alignment to P&Ls for business unit subscriptions.</span></span>
* <span data-ttu-id="2431f-166">関係者の価値とコストの報告方法を毎月分析します。</span><span class="sxs-lookup"><span data-stu-id="2431f-166">Analyze stakeholder value and cost reporting methods on a monthly basis.</span></span>
* <span data-ttu-id="2431f-167">使用率の低い資産を修正し、継続する価値があるかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="2431f-167">Remediate underused assets and determine if they're worth continuing.</span></span>
* <span data-ttu-id="2431f-168">支出の計画と実績の間にある不整合や異常を検出します。</span><span class="sxs-lookup"><span data-stu-id="2431f-168">Detect misalignments and anomalies between the plan and actual spending.</span></span>
* <span data-ttu-id="2431f-169">クラウド導入チームとクラウド戦略チームがこのような異常を把握して解決できるように支援します。</span><span class="sxs-lookup"><span data-stu-id="2431f-169">Assist the cloud adoption teams and the Cloud Strategy team with understanding and resolving these anomalies.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2431f-170">次の手順</span><span class="sxs-lookup"><span data-stu-id="2431f-170">Next steps</span></span>

<span data-ttu-id="2431f-171">ここでは、クラウド ID ガバナンスの概念を理解しました。次は、[コスト管理ツールチェーン](toolchain.md)を検証し、Azure プラットフォーム上でコスト管理ガバナンス規範を開発するときに必要になる Azure のツールと機能を特定します。</span><span class="sxs-lookup"><span data-stu-id="2431f-171">Now that you understand the concept of cloud identity governance, examine the [Cost Management toolchain](toolchain.md) to identify Azure tools and features that you'll need when developing the Cost Management governance discipline on the Azure platform.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="2431f-172">Azure 向けコスト管理ツールチェーン</span><span class="sxs-lookup"><span data-stu-id="2431f-172">Cost Management toolchain for Azure</span></span>](toolchain.md)