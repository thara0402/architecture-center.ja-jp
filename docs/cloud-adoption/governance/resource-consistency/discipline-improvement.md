---
title: 'CAF: リソースの整合性の規範の改良'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: リソースの整合性の規範の改良
author: BrianBlanchard
ms.openlocfilehash: bc81b894d46266c52291c53dba5532ab2ab6b860
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901342"
---
# <a name="resource-consistency-discipline-improvement"></a><span data-ttu-id="a3e47-103">リソースの整合性の規範の改良</span><span class="sxs-lookup"><span data-stu-id="a3e47-103">Resource Consistency discipline improvement</span></span>

<span data-ttu-id="a3e47-104">"リソースの整合性" という規範の焦点は、環境、アプリケーション、またはワークロードの運用管理に関連するポリシーを確立する方法です。</span><span class="sxs-lookup"><span data-stu-id="a3e47-104">The Resource Consistency discipline focuses on ways of establishing policies related to the operational management of an environment, application, or workload.</span></span> <span data-ttu-id="a3e47-105">クラウド ガバナンスの 5 つの規範のうち、"リソースの整合性" にはアプリケーション、ワークロード、および資産のパフォーマンスの監視が含まれます。</span><span class="sxs-lookup"><span data-stu-id="a3e47-105">Within the five disciplines of Cloud Governance, Resource Consistency includes monitoring of applications, workload, and asset performance.</span></span> <span data-ttu-id="a3e47-106">その他に含まれるものとしては、スケーリングが必要な場合の対処や、パフォーマンスのサービス レベル アグリーメント (SLA) 違反の修復、および SLA 違反を先回りして回避するための自動修復があります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-106">It also includes the tasks required to meet scale demands, remediate performance Service Level Agreement (SLA) violations, and proactively avoid SLA violations through automated remediation.</span></span>

<span data-ttu-id="a3e47-107">この記事では、会社の "リソースの整合性" 規範をより適切に発展させ成熟させていくために取り入れることができる、タスクの候補の概要を示します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-107">This article outlines some potential tasks your company can engage in to better develop and mature the Resource Consistency discipline.</span></span> <span data-ttu-id="a3e47-108">これらのタスクは、クラウド ソリューション実装のフェーズ (計画、構築、導入、運用) に分けることができます。これらを反復すると、[クラウド ガバナンスへの増分型アプローチ](../journeys/overview.md#an-incremental-approach-to-cloud-governance)が可能になります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-108">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![導入の 4 つのフェーズ](../../_images/adoption-phases.png)

<span data-ttu-id="a3e47-110">*図 1: クラウド ガバナンスへの増分型アプローチの導入の各フェーズ。*</span><span class="sxs-lookup"><span data-stu-id="a3e47-110">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="a3e47-111">あらゆる企業の要件を 1 つのドキュメントで網羅することは不可能です。</span><span class="sxs-lookup"><span data-stu-id="a3e47-111">It's impossible for any one document to account for the requirements of all businesses.</span></span> <span data-ttu-id="a3e47-112">そのため、この記事ではガバナンス成熟プロセスのフェーズごとに、推奨される最小限のアクティビティと、考えられるアクティビティの例を示します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-112">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="a3e47-113">これらのアクティビティの当初の目標は、[ポリシー MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) の構築と、増分型のポリシー進化のフレームワークの確立に役立てることです。</span><span class="sxs-lookup"><span data-stu-id="a3e47-113">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="a3e47-114">クラウド ガバナンス チームは、"リソースの整合性" のガバナンス能力を高めるためにこれらのアクティビティにどれだけ投資するかを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-114">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Resource Consistency governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="a3e47-115">この記事で示す "最小限" と "考えられる" のどちらのアクティビティも、特定の企業のポリシーや第三者のコンプライアンス要件に合わせたものではありません。</span><span class="sxs-lookup"><span data-stu-id="a3e47-115">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="a3e47-116">このガイダンスは、両方の要件とクラウド ガバナンス モデルの整合に導く会話の促進に役立つように設計されています。</span><span class="sxs-lookup"><span data-stu-id="a3e47-116">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="a3e47-117">計画と準備</span><span class="sxs-lookup"><span data-stu-id="a3e47-117">Planning and readiness</span></span>

<span data-ttu-id="a3e47-118">ガバナンス成熟のこのフェーズは、ビジネスの成果と、アクションにつながる戦略とのギャップを埋めるものです。</span><span class="sxs-lookup"><span data-stu-id="a3e47-118">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="a3e47-119">このプロセスの中で、リーダーシップ チームが具体的なメトリックを定義し、そのメトリックをデジタル資産にマッピングし、全体的な移行作業の計画を開始します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-119">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="a3e47-120">**最小限の推奨アクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-120">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="a3e47-121">自組織の[リソース整合性ツールチェーン](toolchain.md)のオプションを評価します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-121">Evaluate your [Resource Consistency toolchain](toolchain.md) options.</span></span>
* <span data-ttu-id="a3e47-122">自社のクラウド戦略に対するライセンス要件を理解します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-122">Understand the licensing requirements for your cloud strategy.</span></span>
* <span data-ttu-id="a3e47-123">「アーキテクチャ ガイドライン」ドキュメントの下書きを作成し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-123">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="a3e47-124">自組織のソリューションのためのリソースすべてを 1 つのグループとしてデプロイ、管理、監視するのに使用するリソース マネージャーを理解します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-124">Become familiar with the resource manager you use to deploy, manage, and monitor all the resources for your solution as a group.</span></span>
* <span data-ttu-id="a3e47-125">アーキテクチャ ガイドラインの開発によって影響を受ける人とチームを教育し、関与させます。</span><span class="sxs-lookup"><span data-stu-id="a3e47-125">Educate and involve the people and teams affected by the development of architecture guidelines.</span></span>
* <span data-ttu-id="a3e47-126">優先順位の高いリソース デプロイ タスクを移行バックログに追加します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-126">Add prioritized resource deployment tasks to your migration backlog.</span></span>

<span data-ttu-id="a3e47-127">**考えられるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-127">**Potential activities:**</span></span>

* <span data-ttu-id="a3e47-128">業務部門の利害関係者や社内のクラウド戦略チームと協力して、必要なクラウド会計アプローチを理解するとともに、ビジネス ユニット内と組織全体での原価計算手法を理解します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-128">Work with the business stakeholders and/or your cloud strategy team to understand the desired cloud accounting approach and cost accounting practices within your business units and organization as a whole.</span></span>
* <span data-ttu-id="a3e47-129">[監視とポリシー強制](compliance-processes.md)の要件を定義します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-129">Define your [monitoring and policy enforcement](compliance-processes.md) requirements.</span></span>
* <span data-ttu-id="a3e47-130">修復ポリシーと SLA の要件を定義するために、ビジネス価値と機能停止のコストを調べます。</span><span class="sxs-lookup"><span data-stu-id="a3e47-130">Examine the business value and cost of outage to define remediation policy and SLA requirements.</span></span>
* <span data-ttu-id="a3e47-131">リソースに対して[シンプルなワークロード](./governance-simple-workload.md)と[複数チーム](./governance-multiple-teams.md)のどちらのガバナンス戦略を展開するかを決定します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-131">Determine whether you'll deploy a [simple workload](./governance-simple-workload.md) or [multi-team](./governance-multiple-teams.md) governance strategy for your resources.</span></span>
* <span data-ttu-id="a3e47-132">予定しているワークロードに対するスケーラビリティの要件を特定します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-132">Determine scalability requirements for your planned workloads.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="a3e47-133">構築とデプロイ前の準備</span><span class="sxs-lookup"><span data-stu-id="a3e47-133">Build and pre-deployment</span></span>

<span data-ttu-id="a3e47-134">1 つの環境の移行を成功させるには、技術面とそれ以外の面での多数の前提条件を満たす必要があります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-134">A number of technical and nontechnical prerequisites are required to successful migrate an environment.</span></span> <span data-ttu-id="a3e47-135">このプロセスの焦点は、移行を前進させるための意思決定、準備、中核的インフラストラクチャです。</span><span class="sxs-lookup"><span data-stu-id="a3e47-135">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="a3e47-136">**最小限の推奨アクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-136">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="a3e47-137">[リソース整合性ツールチェーン](toolchain.md)を実装するために、デプロイ前フェーズでロールアウトします。</span><span class="sxs-lookup"><span data-stu-id="a3e47-137">Implement your [Resource Consistency toolchain](toolchain.md) by rolling out in a pre-deployment phase.</span></span>
* <span data-ttu-id="a3e47-138">「アーキテクチャ ガイドライン」ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-138">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="a3e47-139">優先順位の高い移行バックログに対してリソース デプロイのタスクを実装します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-139">Implement resource deployment tasks on your prioritized migration backlog.</span></span>
* <span data-ttu-id="a3e47-140">ユーザー受け入れを促進するために、教育用の資料やドキュメント、認識を高めるコミュニケーション、インセンティブ、その他のプログラムを作成します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-140">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="a3e47-141">**考えられるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-141">**Potential activities:**</span></span>

* <span data-ttu-id="a3e47-142">[サブスクリプション設計戦略](../../decision-guides/subscriptions/overview.md)を決定し、組織とワークロードのニーズに最もよく適合するサブスクリプション パターンを選択します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-142">Decide on a [subscription design strategy](../../decision-guides/subscriptions/overview.md), choosing the subscription patterns that best fit your organization and workload needs.</span></span>
* <span data-ttu-id="a3e47-143">[リソース整合性](../../decision-guides/resource-consistency/overview.md)戦略を使用して、アーキテクチャ ガイドラインを徐々に適用します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-143">Use a [resource consistency](../../decision-guides/resource-consistency/overview.md) strategy to enforce architecture guidelines over time.</span></span>
* <span data-ttu-id="a3e47-144">リソースに対する[リソース命名とタグ付けの基準](../../decision-guides/resource-tagging/overview.md)を、組織と会計処理の要件に合わせて実装します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-144">Implement [resource naming, and tagging standards](../../decision-guides/resource-tagging/overview.md) for your resources to match your organizational and accounting requirements.</span></span>
* <span data-ttu-id="a3e47-145">先回り型の、特定時点でのガバナンスを作るには、デプロイ テンプレートと自動化を使用して、リソースやリソース グループをデプロイするときに共通の構成と統一されたグループ化構造を適用します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-145">To create proactive point-in-time governance, use deployment templates and automation to enforce common configurations and a consistent grouping structure when deploying resources and resource groups.</span></span>
* <span data-ttu-id="a3e47-146">最小特権アクセス許可モデルを確立します。このモデルでは、既定ではユーザーに何もアクセス許可を与えません。</span><span class="sxs-lookup"><span data-stu-id="a3e47-146">Establish a least privilege permissions model, where users have no permissions by default.</span></span>
* <span data-ttu-id="a3e47-147">組織内の誰が各ワークロードとアカウントを所有し、誰がこれらのリソースを保守または変更するためのアクセス権を必要とするかを特定します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-147">Determine who in your organization owns each workload and account, and who will need to access to maintain or modify these resources.</span></span> <span data-ttu-id="a3e47-148">クラウドでのロールと責任を、これらのニーズに合わせて定義し、このロールをアクセス制御の基礎として使用します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-148">Define cloud roles and responsibilities that match these needs and use these roles as the basis for access control.</span></span>
* <span data-ttu-id="a3e47-149">リソース間の依存関係を定義します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-149">Define dependencies between resources.</span></span>
* <span data-ttu-id="a3e47-150">計画段階で定義された要件に合わせて自動リソース スケーリングを実装します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-150">Implement automated resource scaling to match requirements defined in the Plan stage.</span></span>
* <span data-ttu-id="a3e47-151">アクセス パフォーマンスを実施して、利用したサービスの質を測定します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-151">Conduct access performance to measure the quality of services received.</span></span>
* <span data-ttu-id="a3e47-152">構成設定とリソース作成ルールを使用して SLA の適用を管理するための[ポリシー](/azure/governance/policy/overview)のデプロイを検討します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-152">Consider deploying [policy](/azure/governance/policy/overview) to manage SLA enforcement using configuration settings and resource creation rules.</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="a3e47-153">導入と移行</span><span class="sxs-lookup"><span data-stu-id="a3e47-153">Adopt and migrate</span></span>

<span data-ttu-id="a3e47-154">移行は増分型のプロセスであり、その焦点は既存のデジタル資産の中のアプリケーションまたはワークロードの移動、テスト、導入です。</span><span class="sxs-lookup"><span data-stu-id="a3e47-154">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="a3e47-155">**最小限の推奨アクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-155">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="a3e47-156">[リソース整合性ツールチェーン](toolchain.md)をデプロイ前準備から本稼働に移行します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-156">Migrate your [Resource Consistency toolchain](toolchain.md) from pre-deployment to production.</span></span>
* <span data-ttu-id="a3e47-157">「アーキテクチャ ガイドライン」ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-157">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="a3e47-158">ユーザー受け入れを促進するために、教育用の資料やドキュメント、認識を高めるコミュニケーション、インセンティブ、その他のプログラムを作成します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-158">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>
* <span data-ttu-id="a3e47-159">定義済みの SLA 要件をサポートする自動修復のためのスクリプトやツールが既にある場合は移行します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-159">Migrate any existing automated remediation scripts or tools to support defined SLA requirements.</span></span>

<span data-ttu-id="a3e47-160">**考えられるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-160">**Potential activities:**</span></span>

* <span data-ttu-id="a3e47-161">データの監視とレポートの機能を完成させ、テストします。</span><span class="sxs-lookup"><span data-stu-id="a3e47-161">Complete and test monitoring and reporting data.</span></span> <span data-ttu-id="a3e47-162">選択したオンプレミス、クラウド ゲートウェイ、またはハイブリッドのソリューションでテストします。</span><span class="sxs-lookup"><span data-stu-id="a3e47-162">with your chosen on-premises, cloud gateway, or hybrid solution.</span></span>
* <span data-ttu-id="a3e47-163">リソースの SLA や管理ポリシーについて変更が必要かどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-163">Determine if changes need to be made to SLA or management policy for resources.</span></span>
* <span data-ttu-id="a3e47-164">運用タスクを向上させるために、クエリ機能を実装します。自組織のクラウド資産全体でリソースを効率的に見つけるためです。</span><span class="sxs-lookup"><span data-stu-id="a3e47-164">Improve operations tasks by implementing query capabilities to efficiently find resource across your cloud estate.</span></span>
* <span data-ttu-id="a3e47-165">ビジネスのニーズやガバナンス要件の変化にリソースを整合させます。</span><span class="sxs-lookup"><span data-stu-id="a3e47-165">Align resources to changing business needs and governance requirements.</span></span>
* <span data-ttu-id="a3e47-166">仮想マシン、仮想ネットワーク、ストレージ アカウントに実際のリソース アクセス要件が反映されていることを各リリースの中で確認し、必要に応じて調整します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-166">Ensure that your virtual machines, virtual networks, and storage accounts reflect actual resource access needs during each release, and adjust as necessary.</span></span>
* <span data-ttu-id="a3e47-167">リソースの自動スケーリングがアクセスの要件を満たしていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-167">Verify automated scaling of resources meets access requirements.</span></span>
* <span data-ttu-id="a3e47-168">リソース、リソース グループ、Azure のサブスクリプションに対するユーザー アクセスを確認し、必要に応じてアクセス制御を調整します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-168">Review user access to resources, resource groups, and Azure subscriptions, and adjust access controls as necessary.</span></span>
* <span data-ttu-id="a3e47-169">リソース アクセス プランの変化を監視し、追加の承認が必要な場合は利害関係者に確認を取ります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-169">Monitor changes in resource access plans and validate with stakeholders if additional sign-offs are needed.</span></span>
* <span data-ttu-id="a3e47-170">アーキテクチャ ガイドラインのドキュメントを更新して実際のコストを反映します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-170">Update changes to the Architecture Guidelines document to reflect actual costs.</span></span>
* <span data-ttu-id="a3e47-171">自組織がビジネス ユニットの P&L への明確な財務的整合性を求めているかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-171">Determine whether your organization requires clearer financial alignment to P&Ls for business units.</span></span>
* <span data-ttu-id="a3e47-172">グローバルな組織の場合は、自組織の SLA コンプライアンスや主権性の要件を実装します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-172">For global organizations, implement your SLA compliance or sovereignty requirements.</span></span>
* <span data-ttu-id="a3e47-173">クラウド アグリゲーションの場合は、クラウド プロバイダーへのゲートウェイ ソリューションをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="a3e47-173">For cloud aggregation, deploy a gateway solution to a cloud provider.</span></span>
* <span data-ttu-id="a3e47-174">ツールがハイブリッドやゲートウェイのオプションに対応していない場合は、監視と運用監視ツールを密接に結合します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-174">For tools that don't allow for hybrid or gateway options, tightly couple monitoring with an operational monitoring tool.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="a3e47-175">運用と実装後</span><span class="sxs-lookup"><span data-stu-id="a3e47-175">Operate and post-implementation</span></span>

<span data-ttu-id="a3e47-176">変換が完了したら、アプリケーションやワークロードの自然ライフサイクルに対してガバナンスと運用を開始する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a3e47-176">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="a3e47-177">ガバナンス成熟のこのフェーズの焦点は、ソリューションが実装されて変換サイクルが安定し始めた後に一般的に行われるアクティビティです。</span><span class="sxs-lookup"><span data-stu-id="a3e47-177">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="a3e47-178">**最小限の推奨アクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-178">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="a3e47-179">[リソースの整合性のツールチェーン](toolchain.md)を、自組織のコスト管理のニーズの変化に基づいてカスタマイズします。</span><span class="sxs-lookup"><span data-stu-id="a3e47-179">Customize your [Resource Consistency toolchain](toolchain.md) based on updates to your organization’s changing Cost Management needs.</span></span>
* <span data-ttu-id="a3e47-180">実際のリソース使用状況を反映する通知やレポートの自動化を検討します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-180">Consider automating any notifications and reports to reflect actual resource usage.</span></span>
* <span data-ttu-id="a3e47-181">将来の導入プロセスの案内となるように、「アーキテクチャ ガイドライン」を改訂します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-181">Refine Architecture Guidelines to guide future adoption processes.</span></span>
* <span data-ttu-id="a3e47-182">アーキテクチャ ガイドラインに常に準拠しているようにするために、影響を受けるチームの教育を定期的に行います。</span><span class="sxs-lookup"><span data-stu-id="a3e47-182">Educate affected teams periodically to ensure ongoing adherence to the architecture guidelines.</span></span>

<span data-ttu-id="a3e47-183">**考えられるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="a3e47-183">**Potential activities:**</span></span>

* <span data-ttu-id="a3e47-184">四半期ごとにプランを調整して実際のリソースの変更を反映します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-184">Adjust plans quarterly to reflect changes to actual resources.</span></span>
* <span data-ttu-id="a3e47-185">将来のデプロイのときに、ガバナンスの要件を自動的に適用して強制します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-185">Automatically apply and enforce governance requirements during future deployments.</span></span>
* <span data-ttu-id="a3e47-186">使用率の低いリソースを評価し、利用を続行する価値があるかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-186">Evaluate underused resources and determine if they're worth continuing.</span></span>
* <span data-ttu-id="a3e47-187">リソース使用状況の計画と実績の間にある不整合や異常を検出します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-187">Detect misalignments and anomalies between planned and actual resource usage.</span></span>
* <span data-ttu-id="a3e47-188">クラウド導入チームとクラウド戦略チームがこれらの異常を理解して解決するのを手伝います。</span><span class="sxs-lookup"><span data-stu-id="a3e47-188">Assist the cloud adoption teams and the Cloud Strategy team in understanding and resolving these anomalies.</span></span>
* <span data-ttu-id="a3e47-189">料金請求や SLA に関して "リソースの整合性" に変更を加える必要があるかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-189">Determine if changes need to be made to Resource Consistency for billing and SLAs.</span></span>
* <span data-ttu-id="a3e47-190">ログ記録と監視のツールを評価し、自組織のオンプレミス、クラウド ゲートウェイ、またはハイブリッド ソリューションの調整が必要かどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-190">Evaluate logging and monitoring tools to determine whether your on-premises, cloud gateway, or hybrid solution needs adjusting.</span></span>
* <span data-ttu-id="a3e47-191">ビジネス ユニットや地理的に分散したグループについて、一元化したポリシーの適用と SLA 要件の達成のために、追加のクラウド管理機能 (たとえば [Azure 管理グループ](/azure/governance/management-groups/)) の使用を検討すべきかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="a3e47-191">For business units and geographically distributed groups, determine if your organization should consider using additional cloud management features (for example [Azure management groups](/azure/governance/management-groups/)) to better apply centralized policy and meet SLA requirements.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a3e47-192">次の手順</span><span class="sxs-lookup"><span data-stu-id="a3e47-192">Next steps</span></span>

<span data-ttu-id="a3e47-193">クラウド リソース ガバナンスの概念を理解したので、Azure での[リソース アクセスの管理方法](azure-resource-access.md)の詳細の学習に進み、[シンプルなワークロード](governance-simple-workload.md)または[複数チーム](governance-multiple-teams.md)のためのガバナンス モデル設計方法の学習に備えてください。</span><span class="sxs-lookup"><span data-stu-id="a3e47-193">Now that you understand the concept of cloud resource governance, move on to learn more about [how resource access is managed](azure-resource-access.md) in Azure in preparation for learning how to design a governance model for a [simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md).</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="a3e47-194">[Azure でのリソース アクセスについて学習する](azure-resource-access.md)
> [Azure の SLA について学習する](https://azure.microsoft.com/support/legal/sla/)
> [ログ記録、レポート、監視について学習する](../../decision-guides/log-and-report/overview.md)</span><span class="sxs-lookup"><span data-stu-id="a3e47-194">[Learn about resource access in Azure](azure-resource-access.md)
[Learn about SLAs for Azure](https://azure.microsoft.com/support/legal/sla/)
[Learn about logging, reporting, and monitoring](../../decision-guides/log-and-report/overview.md)</span></span>
