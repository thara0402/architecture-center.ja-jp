---
title: 'CAF: セキュリティ ベースライン規範の改良'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: セキュリティ ベースライン規範の改良
author: BrianBlanchard
ms.openlocfilehash: 28a971f56c9f8ada1d184bdc1cb3dbb9a17c3507
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901902"
---
# <a name="security-baseline-discipline-improvement"></a><span data-ttu-id="9b6d1-103">セキュリティ ベースライン規範の改良</span><span class="sxs-lookup"><span data-stu-id="9b6d1-103">Security Baseline discipline improvement</span></span>

<span data-ttu-id="9b6d1-104">セキュリティ ベースライン規範では、ネットワーク、資産、および最も重要なものとしてクラウド プロバイダーのソリューションに存在するデータを保護するポリシーを確立する方法に重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-104">The Security Baseline discipline focuses on ways of establishing policies that protect the network, assets, and most importantly the data that will reside on a Cloud Provider's solution.</span></span> <span data-ttu-id="9b6d1-105">クラウド ガバナンスの 5 つの規範のうち、セキュリティ ベースラインにはデジタル資産とデータの分類が含まれます。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-105">Within the five disciplines of Cloud Governance, Security Baseline includes classification of the digital estate and data.</span></span> <span data-ttu-id="9b6d1-106">また、リスク、ビジネスの許容度、およびデータ、資産、ネットワークのセキュリティに関連付けられた軽減戦略のドキュメントも含まれます。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-106">It also includes documentation of risks, business tolerance, and mitigation strategies associated with the security of the data, assets, and network.</span></span> <span data-ttu-id="9b6d1-107">技術的な観点から、これには、[暗号化](../../decision-guides/encryption/overview.md)、[ネットワークの要件](../../decision-guides/software-defined-network/overview.md)、[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)、およびクラウド セキュリティ ベースライン ポリシーを開発するために使用される[プロセス](compliance-processes.md)に関連した決定への関与も含まれます。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-107">From a technical perspective, this also includes involvement in decisions regarding [encryption](../../decision-guides/encryption/overview.md), [network requirements](../../decision-guides/software-defined-network/overview.md), [hybrid identity strategies](../../decision-guides/identity/overview.md), and the [processes](compliance-processes.md) used to develop cloud Security Baseline policies.</span></span>

<span data-ttu-id="9b6d1-108">この記事では、会社がセキュリティ ベースライン規範をより適切に開発し、成熟させるために実行できるいくつかの潜在的なタスクの概要について説明します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-108">This article outlines some potential tasks your company can engage in to better develop and mature the Security Baseline discipline.</span></span> <span data-ttu-id="9b6d1-109">これらのタスクは、クラウド ソリューション実装の計画、構築、導入、および運用フェーズに分類できます。その後、それらが反復処理されて、[クラウド ガバナンスへの増分アプローチ](../journeys/overview.md#an-incremental-approach-to-cloud-governance)の開発が可能になります。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-109">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![導入の 4 つのフェーズ](../../_images/adoption-phases.png)

<span data-ttu-id="9b6d1-111">*図 1: クラウド ガバナンスへの増分アプローチの導入フェーズ。*</span><span class="sxs-lookup"><span data-stu-id="9b6d1-111">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="9b6d1-112">1 つのドキュメントですべてのビジネスの要件を説明することはできません。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-112">It's impossible for any one document to account for the requirements of all businesses.</span></span> <span data-ttu-id="9b6d1-113">そのため、この記事では、ガバナンス成熟プロセスのフェーズごとに推奨される最小限のアクティビティと潜在的なアクティビティの例の概要について説明します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-113">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="9b6d1-114">これらのアクティビティの初期の目標は、ユーザーが[ポリシー MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) を構築し、増分ポリシー進化のためのフレームワークを確立するのを支援することです。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-114">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="9b6d1-115">クラウド ガバナンス チームは、セキュリティ ベースライン ガバナンス機能を強化するために、これらのアクティビティにどれだけ投資するかを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-115">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Security Baseline governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="9b6d1-116">この記事で概要を説明する最小限のアクティビティと潜在的なアクティビティのどちらも、特定の企業ポリシーやサードパーティのコンプライアンス要件には整合していません。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-116">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="9b6d1-117">このガイダンスは、両方の要件とクラウド ガバナンス モデルの整合に導く会話の促進に役立つように設計されています。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-117">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="9b6d1-118">計画および準備状況</span><span class="sxs-lookup"><span data-stu-id="9b6d1-118">Planning and readiness</span></span>

<span data-ttu-id="9b6d1-119">ガバナンス成熟のこのフェーズは、ビジネス成果とすぐに実行できる戦略の間のギャップを橋渡しします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-119">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="9b6d1-120">このプロセス中に、リーダーシップ チームは特定のメトリックを定義し、これらのメトリックをデジタル資産にマップして、全体的な移行作業の計画を開始します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-120">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="9b6d1-121">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-121">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="9b6d1-122">[セキュリティ ベースライン ツールチェーン](toolchain.md) オプションを評価します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-122">Evaluate your [Security Baseline toolchain](toolchain.md) options.</span></span>
- <span data-ttu-id="9b6d1-123">下書きのアーキテクチャ ガイドライン ドキュメントを開発し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-123">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
- <span data-ttu-id="9b6d1-124">アーキテクチャ ガイドラインの開発によって影響を受ける人びとやチームを教育し、関与させます。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-124">Educate and involve the people and teams affected by the development of architecture guidelines.</span></span>
- <span data-ttu-id="9b6d1-125">優先順位が付けられたセキュリティ タスクを移行バックログに追加します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-125">Add prioritized security tasks to your migration backlog.</span></span>

<span data-ttu-id="9b6d1-126">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-126">**Potential activities:**</span></span>

- <span data-ttu-id="9b6d1-127">データ分類スキーマを定義します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-127">Define a data classification schema.</span></span>
- <span data-ttu-id="9b6d1-128">ビジネス プロセスを強化し、運用をサポートしている現在の IT 資産のインベントリを作成するために、デジタル資産計画プロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-128">Conduct a digital estate planning process to inventory the current IT assets powering your business processes and supporting operations.</span></span>
- <span data-ttu-id="9b6d1-129">既存の企業 IT セキュリティ ポリシーを最新化するプロセスを開始し、既知のリスクに対応した MVP ポリシーを定義するために、[ポリシー レビュー](../../governance/policy-compliance/what-is-a-cloud-policy-review.md)を実行します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-129">Conduct a [policy review](../../governance/policy-compliance/what-is-a-cloud-policy-review.md) to begin the process of modernizing existing corporate IT security policies, and define MVP policies addressing known risks.</span></span>
- <span data-ttu-id="9b6d1-130">クラウド プラットフォームのセキュリティ ガイドラインをレビューします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-130">Review your cloud platform's security guidelines.</span></span> <span data-ttu-id="9b6d1-131">Azure の場合、これらは [Microsoft Service Trust Platform](https://www.microsoft.com/trustcenter/stp/default.aspx) にあります。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-131">For Azure these can be found in the [Microsoft Service Trust Platform](https://www.microsoft.com/trustcenter/stp/default.aspx).</span></span>
- <span data-ttu-id="9b6d1-132">セキュリティ ベースライン ポリシーに[セキュリティ開発ライフサイクル](https://www.microsoft.com/securityengineering/sdl/)が含まれているかどうかを判定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-132">Determine whether your Security Baseline policy includes a [Security Development Lifecycle](https://www.microsoft.com/securityengineering/sdl/).</span></span>
- <span data-ttu-id="9b6d1-133">次の 1 つから 3 つのリリースに基づいてネットワーク、データ、および資産関連のビジネス リスクを評価し、これらのリスクに対する組織の許容度を測定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-133">Evaluate network, data, and asset-related business risks based on the next one to three releases, and gauge your organization's tolerance for those risks.</span></span>
- <span data-ttu-id="9b6d1-134">現在のセキュリティ ランドスケープの概要を取得するために、Microsoft の[サイバーセキュリティにおける上位の傾向](https://www.microsoft.com/security/operations/security-intelligence-report)レポートをレビューします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-134">Review Microsoft's [top trends in cybersecurity](https://www.microsoft.com/security/operations/security-intelligence-report) report to get an overview of the current security landscape.</span></span>
- <span data-ttu-id="9b6d1-135">組織内の[セキュリティ DevOps](https://www.microsoft.com/en-us/securityengineering/devsecops) ロールの開発を検討します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-135">Consider developing a [Security DevOps](https://www.microsoft.com/en-us/securityengineering/devsecops) role in your organization.</span></span>

<!-- "en-us" location is required for the URL above. -->

## <a name="build-and-pre-deployment"></a><span data-ttu-id="9b6d1-136">構築およびデプロイ前準備</span><span class="sxs-lookup"><span data-stu-id="9b6d1-136">Build and pre-deployment</span></span>

<span data-ttu-id="9b6d1-137">環境を正常に移行するには、いくつかの技術的な前提条件および技術面以外の前提条件が必要です。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-137">A number of technical and nontechnical prerequisites are required to successful migrate an environment.</span></span> <span data-ttu-id="9b6d1-138">このプロセスでは、移行を進める意思決定、準備状況、およびコア インフラストラクチャに重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-138">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="9b6d1-139">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-139">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="9b6d1-140">デプロイ前フェーズでロールアウトすることによって、[セキュリティ ベースライン ツールチェーン](toolchain.md)を実装します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-140">Implement your [Security Baseline toolchain](toolchain.md) by rolling out in a pre-deployment phase.</span></span>
- <span data-ttu-id="9b6d1-141">アーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-141">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
- <span data-ttu-id="9b6d1-142">優先順位が付けられた移行バックログでセキュリティ タスクを実装します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-142">Implement security tasks on your prioritized migration backlog.</span></span>
- <span data-ttu-id="9b6d1-143">ユーザー導入の促進に役立つ教材やドキュメント、認知の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-143">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="9b6d1-144">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-144">**Potential activities:**</span></span>

- <span data-ttu-id="9b6d1-145">クラウドでホストされたデータのための組織の[暗号化](../../decision-guides/encryption/overview.md)戦略を決定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-145">Determine your organization's [encryption](../../decision-guides/encryption/overview.md) strategy for cloud-hosted data.</span></span>
- <span data-ttu-id="9b6d1-146">クラウド デプロイの [ID](../../decision-guides/identity/overview.md) 戦略を評価します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-146">Evaluate your cloud deployment's [identity](../../decision-guides/identity/overview.md) strategy.</span></span> <span data-ttu-id="9b6d1-147">クラウド ベースの ID ソリューションがオンプレミスの ID プロバイダーと共存または統合する方法を決定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-147">Determine how your cloud-based identity solution will coexist or integrate with on-premises identity providers.</span></span>
- <span data-ttu-id="9b6d1-148">[ソフトウェア定義ネットワーク (SDN)](../../decision-guides/software-defined-network/overview.md) 設計で仮想ネットワーク機能を確実にセキュリティで保護するためのネットワーク境界ポリシーを決定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-148">Determine network boundary policies for your [Software Defined Networking (SDN)](../../decision-guides/software-defined-network/overview.md) design to ensure secure virtualized networking capabilities.</span></span>
- <span data-ttu-id="9b6d1-149">組織の[最小特権アクセス](/azure/active-directory/users-groups-roles/roles-delegate-by-task) ポリシーを評価し、タスク ベースのロールを使用して特定のリソースへのアクセスを提供します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-149">Evaluate your organization's [least privilege access](/azure/active-directory/users-groups-roles/roles-delegate-by-task) policies, and use task-based roles to provide access to specific resources.</span></span>
- <span data-ttu-id="9b6d1-150">すべてのクラウド サービスと仮想マシンにセキュリティおよび監視メカニズムを適用します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-150">Apply security and monitoring mechanisms to for all cloud services and virtual machines.</span></span>
- <span data-ttu-id="9b6d1-151">可能な場合は、[セキュリティ ポリシー](../../decision-guides/policy-enforcement/overview.md)を自動化します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-151">Automate [security policies](../../decision-guides/policy-enforcement/overview.md) where possible.</span></span>
- <span data-ttu-id="9b6d1-152">セキュリティ ベースライン ポリシーをレビューし、[セキュリティ開発ライフサイクル](https://www.microsoft.com/securityengineering/sdl/)で概要が説明されているようなベスト プラクティス ガイダンスに従って計画を変更する必要があるかどうかを判定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-152">Review your Security Baseline policy and determine if you need to modify your plans according to best practices guidance such as those outlined in the [Security Development Lifecycle](https://www.microsoft.com/securityengineering/sdl/).</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="9b6d1-153">導入および移行</span><span class="sxs-lookup"><span data-stu-id="9b6d1-153">Adopt and migrate</span></span>

<span data-ttu-id="9b6d1-154">移行は、既存のデジタル資産内のアプリケーションまたはワークロードの移動、テスト、および導入に重点を置いた増分プロセスです。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-154">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="9b6d1-155">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-155">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="9b6d1-156">[セキュリティ ベースライン ツールチェーン](toolchain.md)をデプロイ前準備から運用環境に移行します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-156">Migrate your [Security Baseline toolchain](toolchain.md) from pre-deployment to production.</span></span>
- <span data-ttu-id="9b6d1-157">アーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-157">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
- <span data-ttu-id="9b6d1-158">ユーザー導入の促進に役立つ教材やドキュメント、認知の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-158">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption</span></span>

<span data-ttu-id="9b6d1-159">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-159">**Potential activities:**</span></span>

- <span data-ttu-id="9b6d1-160">新しいビジネス リスクを識別するために、最新のセキュリティ ベースラインおよび脅威情報をレビューします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-160">Review the latest Security Baseline and threat information to identify any new business risks.</span></span>
- <span data-ttu-id="9b6d1-161">発生する可能性のある新しいセキュリティ上のリスクを処理するための組織の許容度を測定します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-161">Gauge your organization's tolerance to handle new security risks that may arise.</span></span>
- <span data-ttu-id="9b6d1-162">ポリシーからの逸脱を識別し、修正を適用します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-162">Identify deviations from policy, and enforce corrections.</span></span>
- <span data-ttu-id="9b6d1-163">ポリシーのコンプライアンスを最大化するようにセキュリティとアクセス制御の自動化を調整します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-163">Adjust security and access control automation to ensure maximum policy compliance.</span></span>  
- <span data-ttu-id="9b6d1-164">構築/デプロイ前フェーズ中に定義されたベスト プラクティスが正しく実行されていることを検証します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-164">Validate that the best practices defined during the Build / Pre-deployment phases are properly executed.</span></span>
- <span data-ttu-id="9b6d1-165">最小特権アクセス ポリシーをレビューし、セキュリティを最大化するようにアクセス制御を調整します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-165">Review your least privilege access policies and adjust access controls to maximize security.</span></span>
- <span data-ttu-id="9b6d1-166">脆弱性をすべて識別して解決するために、セキュリティ ベースライン ツールチェーンをワークロードに対してテストします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-166">Test your Security Baseline toolchain against your workloads to identify and resolve any vulnerabilities.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="9b6d1-167">運用および実装後</span><span class="sxs-lookup"><span data-stu-id="9b6d1-167">Operate and post-implementation</span></span>

<span data-ttu-id="9b6d1-168">変換が完了したら、アプリケーションまたはワークロードの自然なライフサイクルに対してガバナンスと運用を続行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-168">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="9b6d1-169">ガバナンス成熟のこのフェーズでは、一般にソリューションが実装され、変換サイクルが安定し始めた後に実行されるアクティビティに重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-169">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="9b6d1-170">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-170">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="9b6d1-171">[セキュリティ ベースライン ツールチェーン](toolchain.md)を検証または改良します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-171">Validate and/or refine your [Security Baseline toolchain](toolchain.md).</span></span>
- <span data-ttu-id="9b6d1-172">潜在的なセキュリティの問題をユーザーに通知するように通知とレポートをカスタマイズします。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-172">Customize notifications and reports to alert you of potential security issues.</span></span>
- <span data-ttu-id="9b6d1-173">将来の導入プロセスのガイドとなるようにアーキテクチャ ガイドラインを改良します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-173">Refine the Architecture Guidelines to guide future adoption processes.</span></span>
- <span data-ttu-id="9b6d1-174">アーキテクチャ ガイドラインへの遵守を引き続き確実にするために、影響を受けるチームを定期的に伝達および教育します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-174">Communicate and educate the affected teams periodically to ensure ongoing adherence to architecture guidelines.</span></span>

<span data-ttu-id="9b6d1-175">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="9b6d1-175">**Potential activities:**</span></span>

- <span data-ttu-id="9b6d1-176">ワークロードのパターンや動作を検出し、異常なアクティビティ、アクセス、またはリソース使用状況をすべて識別してユーザーに通知するように監視およびレポート作成ツールを構成します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-176">Discover patterns and behavior for your workloads and configure your monitoring and reporting tools to identify and notify you of any abnormal activity, access, or resource usage.</span></span>
- <span data-ttu-id="9b6d1-177">最新の脆弱性、エクスプロイト、および攻撃を検出するように監視およびレポート作成ポリシーを継続的に更新します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-177">Continuously update your monitoring and reporting policies to detect the latest vulnerabilities, exploits, and attacks.</span></span>
- <span data-ttu-id="9b6d1-178">承認されていないアクセスをすばやく停止し、攻撃者によって侵害された可能性のあるリソースを無効にするための手順を確立します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-178">Have procedures in place to quickly stop unauthorized access and disable resources that may have been compromised by an attacker.</span></span>
- <span data-ttu-id="9b6d1-179">セキュリティに関する最新のベスト プラクティスを定期的にレビューし、可能な場合は、推奨事項をセキュリティ ポリシー、自動化、および監視機能に適用します。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-179">Regularly review the latest security best practices and apply recommendations to your security policy, automation, and monitoring capabilities where possible.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9b6d1-180">次の手順</span><span class="sxs-lookup"><span data-stu-id="9b6d1-180">Next steps</span></span>

<span data-ttu-id="9b6d1-181">ここまでで、クラウド セキュリティ ガバナンスの概念を理解したので、Azure に関して [Microsoft が提供するセキュリティおよびベスト プラクティス ガイダンス](azure-security-guidance.md)に関するページに進んでください。</span><span class="sxs-lookup"><span data-stu-id="9b6d1-181">Now that you understand the concept of cloud security governance, move on to learn more about [what security and best practices guidance Microsoft provides](azure-security-guidance.md) for Azure.</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="9b6d1-182">[Azure のセキュリティ ガイダンスについて](azure-security-guidance.md)
> [Azure セキュリティの概要](/azure/security/azure-security)
> [ログ、レポート、および監視について](../../decision-guides/log-and-report/overview.md)</span><span class="sxs-lookup"><span data-stu-id="9b6d1-182">[Learn about security guidance for Azure](azure-security-guidance.md)
[Introduction to Azure Security](/azure/security/azure-security)
[Learn about logging, reporting, and monitoring](../../decision-guides/log-and-report/overview.md)</span></span>
