---
title: 'CAF: ID ベースライン規範の改良'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: ID ベースライン規範の改良
author: BrianBlanchard
ms.openlocfilehash: c96a638af549782fec22b2068c9b4943df4b943a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243253"
---
# <a name="identity-baseline-discipline-improvement"></a><span data-ttu-id="12b9a-103">ID ベースライン規範の改良</span><span class="sxs-lookup"><span data-stu-id="12b9a-103">Identity Baseline discipline improvement</span></span>

<span data-ttu-id="12b9a-104">ID ベースライン規範では、アプリケーションまたはワークロードをホストするクラウド プロバイダーには関係なく、ユーザー ID の一貫性と継続性を保証するポリシーを確立する方法に重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="12b9a-104">The Identity Baseline discipline focuses on ways of establishing policies that ensure consistency and continuity of user identities regardless of the cloud provider that hosts the application or workload.</span></span> <span data-ttu-id="12b9a-105">クラウド ガバナンスの 5 つの規範のうち、ID ベースラインには、[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)、ID リポジトリの評価および拡張、シングル サインオン (同じサインオン) の実装、承認されていない使用や悪意のあるアクターに対する監査および監視に関連した意思決定が含まれます。</span><span class="sxs-lookup"><span data-stu-id="12b9a-105">Within the Five Disciplines of Cloud Governance, Identity Baseline includes decisions regarding the [Hybrid Identity Strategy](../../decision-guides/identity/overview.md), evaluation and extension of identity repositories, implementation of single sign-on (same sign-on), auditing and monitoring for unauthorized use or malicious actors.</span></span> <span data-ttu-id="12b9a-106">場合によっては、複数の ID プロバイダーを最新化、整理、または統合するための意思決定も含まれる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-106">In some cases, it may also involve decisions to modernize, consolidate, or integrate multiple identity providers.</span></span>

<span data-ttu-id="12b9a-107">この記事では、会社が ID ベースライン規範をより適切に開発し、成熟させるために実行できるいくつかの潜在的なタスクの概要について説明します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-107">This article outlines some potential tasks your company can engage in to better develop and mature the Identity Baseline discipline.</span></span> <span data-ttu-id="12b9a-108">これらのタスクは、クラウド ソリューション実装の計画、構築、導入、および運用フェーズに分類できます。その後、それらが反復処理されて、[クラウド ガバナンスへの増分アプローチ](../journeys/overview.md#an-incremental-approach-to-cloud-governance)の開発が可能になります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-108">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![導入の 4 つのフェーズ](../../_images/adoption-phases.png)

<span data-ttu-id="12b9a-110">*図 1: クラウド ガバナンスへの増分型アプローチの導入の各フェーズ。*</span><span class="sxs-lookup"><span data-stu-id="12b9a-110">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="12b9a-111">1 つのドキュメントですべての企業の要件を説明することはできません。</span><span class="sxs-lookup"><span data-stu-id="12b9a-111">It's impossible for any one document to account for the requirements of all businesses.</span></span> <span data-ttu-id="12b9a-112">そのため、この記事ではガバナンス成熟プロセスのフェーズごとに、推奨される最小限のアクティビティと、考えられるアクティビティの例を示します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-112">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="12b9a-113">これらのアクティビティの最初の目標は、お客様が[ポリシーの MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) を構築し、漸進的にポリシーを進化させるためのフレームワークを確立できるように支援することです。</span><span class="sxs-lookup"><span data-stu-id="12b9a-113">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="12b9a-114">クラウド ガバナンス チームは、ID ベースライン ガバナンス機能を強化するために、これらのアクティビティにどれだけ投資するかを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-114">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Identity Baseline governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="12b9a-115">この記事で概要を説明する最小限のアクティビティと潜在的なアクティビティのどちらも、特定の企業ポリシーやサードパーティのコンプライアンス要件には整合していません。</span><span class="sxs-lookup"><span data-stu-id="12b9a-115">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="12b9a-116">このガイダンスは、両方の要件とクラウド ガバナンス モデルの整合に導く会話の促進に役立つように設計されています。</span><span class="sxs-lookup"><span data-stu-id="12b9a-116">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="12b9a-117">計画と準備状況</span><span class="sxs-lookup"><span data-stu-id="12b9a-117">Planning and readiness</span></span>

<span data-ttu-id="12b9a-118">ガバナンス成熟のこのフェーズは、ビジネス成果とアクションにつながる戦略の間にあるギャップの橋渡しをします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-118">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="12b9a-119">このプロセス中に、リーダーシップ チームは具体的なメトリックを定義し、そのメトリックをデジタル資産にマップして、全体的な移行作業の計画を開始します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-119">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="12b9a-120">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-120">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="12b9a-121">[ID ツールチェーン](toolchain.md) オプションを評価し、組織に適したハイブリッド戦略を実装します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-121">Evaluate your [Identity toolchain](toolchain.md) options and implement a hybrid strategy that is appropriate to your organization.</span></span>
* <span data-ttu-id="12b9a-122">下書きのアーキテクチャ ガイドライン ドキュメントを開発し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-122">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="12b9a-123">アーキテクチャ ガイドラインの開発によって影響を受ける人びとやチームを教育し、関与させます。</span><span class="sxs-lookup"><span data-stu-id="12b9a-123">Educate and involve the people and teams affected by the development of architecture guidelines.</span></span>

<span data-ttu-id="12b9a-124">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-124">**Potential activities:**</span></span>

* <span data-ttu-id="12b9a-125">クラウド内の ID 管理とアクセス管理を管理するロールと割り当てを定義します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-125">Define roles and assignments that will govern identity and access management in the cloud.</span></span>
* <span data-ttu-id="12b9a-126">オンプレミス グループを定義し、対応するクラウド ベースのロールにマップします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-126">Define your on-premises groups and map to corresponding cloud-based roles.</span></span>
* <span data-ttu-id="12b9a-127">ID プロバイダー (カスタム アプリケーションによって使用されるデータベース主導の ID を含む) のインベントリを作成します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-127">Inventory identity providers (including database-driven identities used by custom applications).</span></span>
* <span data-ttu-id="12b9a-128">全体的な ID ソリューションを簡素化するために、重複が存在する場合は ID プロバイダーの整理または統合のためのオプションを検討します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-128">Consider options for consolidation or integration of identity providers where duplication exists, to simplify the overall identity solution.</span></span>
* <span data-ttu-id="12b9a-129">既存の ID プロバイダーのハイブリッド互換性を評価します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-129">Evaluate hybrid compatibility of existing identity providers.</span></span>
* <span data-ttu-id="12b9a-130">ハイブリッド互換性のない ID プロバイダーの場合は、整理または置き換えオプションを評価します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-130">For identity providers that are not hybrid compatible, evaluate consolidation or replacement options.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="12b9a-131">構築およびデプロイ前準備</span><span class="sxs-lookup"><span data-stu-id="12b9a-131">Build and pre-deployment</span></span>

<span data-ttu-id="12b9a-132">環境を適切に移行するには、いくつかの技術的な前提条件と技術面以外の前提条件が必要です。</span><span class="sxs-lookup"><span data-stu-id="12b9a-132">A number of technical and nontechnical prerequisites are required to successfully migrate an environment.</span></span> <span data-ttu-id="12b9a-133">このプロセスでは、移行を進める意思決定、準備状況、およびコア インフラストラクチャに重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="12b9a-133">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="12b9a-134">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-134">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="12b9a-135">[ID ツールチェーン](toolchain.md)を実装する前にパイロット テストを検討して、それによりユーザー エクスペリエンスができる限り簡素化されることを確認します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-135">Consider a pilot test before implementing your [Identity toolchain](toolchain.md), making sure it simplifies the user experience as much as possible.</span></span>
* <span data-ttu-id="12b9a-136">パイロット テストからデプロイ前準備にフィードバックを適用します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-136">Apply feedback from pilot tests into the pre-deployment.</span></span> <span data-ttu-id="12b9a-137">許容可能な結果が出るまで繰り返します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-137">Repeat until results are acceptable.</span></span>
* <span data-ttu-id="12b9a-138">デプロイとユーザー導入計画を含むようにアーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-138">Update the Architecture Guidelines document to include deployment and user adoption plans, and distribute to key stakeholders.</span></span>
* <span data-ttu-id="12b9a-139">早期導入者プログラムの確立、および制限された数のユーザーへのロールアウトを検討します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-139">Consider establishing an early adopter program and rolling out to a limited number of users.</span></span>
* <span data-ttu-id="12b9a-140">アーキテクチャ ガイドラインによって最も影響を受ける人びとやチームを引き続き教育します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-140">Continue to educate the people and teams most affected by the architecture guidelines.</span></span>

<span data-ttu-id="12b9a-141">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-141">**Potential activities:**</span></span>

* <span data-ttu-id="12b9a-142">論理および物理アーキテクチャを評価し、[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)を決定します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-142">Evaluate your logical and physical architecture and determine a [Hybrid Identity Strategy](../../decision-guides/identity/overview.md).</span></span>
* <span data-ttu-id="12b9a-143">ID アクセス管理ポリシー (ログイン ID の割り当てなど) をマップし、Azure AD の適切な認証方法を選択します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-143">Map identity access management policies, such as login ID assignments, and choose the appropriate authentication method for Azure AD.</span></span>
  * <span data-ttu-id="12b9a-144">フェデレーションの場合は、管理者アカウントに対するテナント制限を有効にします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-144">If federated, enable tenant restrictions for administrative accounts.</span></span>
* <span data-ttu-id="12b9a-145">オンプレミス ディレクトリとクラウド ディレクトリを統合します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-145">Integrate your on-premises and cloud directories.</span></span>
* <span data-ttu-id="12b9a-146">次のアクセス モデルの使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-146">Consider using the following access models:</span></span>
  * <span data-ttu-id="12b9a-147">[最小特権アクセス](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) モデル</span><span class="sxs-lookup"><span data-stu-id="12b9a-147">[Least Privilege Access](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) model</span></span>
  * <span data-ttu-id="12b9a-148">[特権 ID ベースライン](/azure/active-directory/privileged-identity-management/pim-configure) アクセス モデル</span><span class="sxs-lookup"><span data-stu-id="12b9a-148">[Privileged Identity Baseline](/azure/active-directory/privileged-identity-management/pim-configure) access model</span></span>
* <span data-ttu-id="12b9a-149">統合前の詳細をすべて完成させ、[ID のベスト プラクティス](/azure/security/azure-security-identity-management-best-practices)をレビューします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-149">Finalize all pre-integration details and review [Identity Best Practices](/azure/security/azure-security-identity-management-best-practices).</span></span>
  * <span data-ttu-id="12b9a-150">単一の ID、シングル サインオン (SSO)、またはシームレス SSO を有効にする</span><span class="sxs-lookup"><span data-stu-id="12b9a-150">Enable single identity, single sign-on (SSO), or seamless SSO</span></span>
  * <span data-ttu-id="12b9a-151">管理者に対する多要素認証 (MFA) を構成する</span><span class="sxs-lookup"><span data-stu-id="12b9a-151">Configure multi-factor authentication (MFA) for administrators</span></span>
  * <span data-ttu-id="12b9a-152">必要な場合は、ID プロバイダーを整理または統合する</span><span class="sxs-lookup"><span data-stu-id="12b9a-152">Consolidate or integrate identity providers, where necessary</span></span>
  * <span data-ttu-id="12b9a-153">ID の管理を一元管理するために必要なツールを実装する</span><span class="sxs-lookup"><span data-stu-id="12b9a-153">Implement tooling necessary to centralize management of identities</span></span>
  * <span data-ttu-id="12b9a-154">Just-In-Time (JIT) アクセスおよびロール変更の通知を有効にする</span><span class="sxs-lookup"><span data-stu-id="12b9a-154">Enable just-in-time (JIT) access and role change alerting</span></span>
  * <span data-ttu-id="12b9a-155">組み込みロールに割り当てるための主な管理者アクティビティのリスク分析を実行する</span><span class="sxs-lookup"><span data-stu-id="12b9a-155">Conduct a risk analysis of key admin activities for assigning to built-in roles</span></span>
  * <span data-ttu-id="12b9a-156">すべてのユーザーへのより強力な認証の更新されたロールアウトを検討する</span><span class="sxs-lookup"><span data-stu-id="12b9a-156">Consider an updated rollout of stronger authentication for all users</span></span>
  * <span data-ttu-id="12b9a-157">追加の管理者ロールのための (期間限定アクティブ化を使用した) JIT の特権 ID ベースライン (PIM) を有効にする</span><span class="sxs-lookup"><span data-stu-id="12b9a-157">Enable Privileged Identity Baseline (PIM) for JIT (using time-limited activation) for additional administrative roles</span></span>
  * <span data-ttu-id="12b9a-158">ユーザー アカウントをグローバル管理者アカウントから分離する (管理者が、自分のグローバル管理者アカウントに関連付けられた電子メールを誤って開いたり、プログラムを実行したりしないようにするため)</span><span class="sxs-lookup"><span data-stu-id="12b9a-158">Separate user accounts from Global admin accounts (to make sure that administrators do not inadvertently open emails or run programs associated with their Global admin accounts)</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="12b9a-159">導入および移行</span><span class="sxs-lookup"><span data-stu-id="12b9a-159">Adopt and migrate</span></span>

<span data-ttu-id="12b9a-160">移行は増分型のプロセスであり、その焦点は既存のデジタル資産の中のアプリケーションまたはワークロードの移動、テスト、導入です。</span><span class="sxs-lookup"><span data-stu-id="12b9a-160">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="12b9a-161">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-161">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="12b9a-162">[ID ツールチェーン](toolchain.md)を開発環境から運用環境に移行します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-162">Migrate your [Identity toolchain](toolchain.md) from development to production.</span></span>
* <span data-ttu-id="12b9a-163">アーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-163">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="12b9a-164">ユーザー導入の促進に役立つ教材やドキュメント、認識の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-164">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="12b9a-165">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-165">**Potential activities:**</span></span>

* <span data-ttu-id="12b9a-166">構築/デプロイ前フェーズ中に定義されたベスト プラクティスが正しく実行されていることを検証します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-166">Validate that the best practices defined during the Build / Pre-deployment phases are properly executed.</span></span>
* <span data-ttu-id="12b9a-167">[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)を検証または改良します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-167">Validate and/or refine your [Hybrid Identity Strategy](../../decision-guides/identity/overview.md).</span></span>
* <span data-ttu-id="12b9a-168">リリースの前に、各アプリケーションまたはワークロードが引き続き ID 戦略に整合していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-168">Ensure that each application or workload continues to align with the identity strategy before release.</span></span>
* <span data-ttu-id="12b9a-169">シングル サインオン (SSO) とシームレス SSO がアプリケーションで期待どおりに機能していることを検証します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-169">Validate that single sign-on (SSO) and seamless SSO is working as expected for your applications.</span></span>
* <span data-ttu-id="12b9a-170">可能な場合は、代替 ID ストアの数を減らすか、またはなくします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-170">Reduce or eliminate the number of alternative identity stores, when possible.</span></span>
* <span data-ttu-id="12b9a-171">アプリ内またはデータベース内 ID ストアの必要性を綿密に検査します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-171">Scrutinize the need for any in-app or in-database identity stores.</span></span> <span data-ttu-id="12b9a-172">適切な ID プロバイダー (ファーストパーティまたはサードパーティ) から外れた ID は、アプリケーションやユーザーにとってリスクになる場合があります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-172">Identities that fall outside of a proper identity provider (first-party or third-party) can represent risk to the application and the users.</span></span>
* <span data-ttu-id="12b9a-173">[オンプレミスのフェデレーション アプリケーション](/azure/active-directory/active-directory-device-registration-on-premises-setup)への条件付きアクセスを有効にします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-173">Enable conditional access for [on-premises federated applications](/azure/active-directory/active-directory-device-registration-on-premises-setup).</span></span>
* <span data-ttu-id="12b9a-174">ID を複数のハブ内のグローバル リージョンにわたって分散させ、リージョン間の同期を維持します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-174">Distribute identity across global regions in multiple hubs with synchronization between regions.</span></span>
* <span data-ttu-id="12b9a-175">中央のロールベースのアクセス制御 (RBAC) フェデレーションを確立します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-175">Establish central role-based access control (RBAC) federation.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="12b9a-176">運用および実装後</span><span class="sxs-lookup"><span data-stu-id="12b9a-176">Operate and post-implementation</span></span>

<span data-ttu-id="12b9a-177">変換が完了したら、アプリケーションまたはワークロードの自然なライフサイクルに対してガバナンスと運用を続行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-177">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="12b9a-178">ガバナンス成熟のこのフェーズの焦点は、ソリューションが実装されて変換サイクルが安定し始めた後に一般的に行われるアクティビティです。</span><span class="sxs-lookup"><span data-stu-id="12b9a-178">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="12b9a-179">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-179">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="12b9a-180">組織の変化する ID ニーズに対する変更に基づいて、[ID ベースライン ツールチェーン](toolchain.md)をカスタマイズします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-180">Customize your [Identity Baseline toolchain](toolchain.md) based on changes to your organization’s changing identity needs.</span></span>
* <span data-ttu-id="12b9a-181">潜在的な悪意のある脅威をユーザーに通知するように、通知とレポートを自動化します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-181">Automate notifications and reports to alert you of potential malicious threats.</span></span>
* <span data-ttu-id="12b9a-182">システムの使用状況やユーザー導入の進行状況について監視および報告します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-182">Monitor and report on system usage and user adoption progress.</span></span>
* <span data-ttu-id="12b9a-183">デプロイ後のメトリックについて報告し、利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-183">Report on post-deployment metrics and distribute to stakeholders.</span></span>
* <span data-ttu-id="12b9a-184">将来の導入プロセスのガイドとなるようにアーキテクチャ ガイドラインを改良します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-184">Refine the Architecture Guidelines to guide future adoption processes.</span></span>
* <span data-ttu-id="12b9a-185">アーキテクチャ ガイドラインへの遵守を引き続き確実にするために、影響を受けるチームを定期的に伝達し、継続的に教育します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-185">Communicate and continually educate the affected teams on a periodic basis to ensure ongoing adherence to architecture guidelines.</span></span>

<span data-ttu-id="12b9a-186">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="12b9a-186">**Potential activities:**</span></span>

* <span data-ttu-id="12b9a-187">ID ポリシーおよび遵守プラクティスの定期的な監査を実行します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-187">Conduct periodic audits of identity policies and adherence practices.</span></span>
* <span data-ttu-id="12b9a-188">悪意のあるアクターやデータ侵害 (特に、潜在的な管理者アカウント乗っ取りなど ID の不正アクセスに関連したもの) を定期的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-188">Scan for malicious actors and data breaches regularly, particularly those related to identity fraud, such as potential admin account takeovers.</span></span>
* <span data-ttu-id="12b9a-189">監視およびレポート作成ツールを構成します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-189">Configure a monitoring and reporting tool.</span></span>
* <span data-ttu-id="12b9a-190">セキュリティおよび不正アクセス防止システムとのより緊密な統合を検討します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-190">Consider integrating more closely with security and fraud-prevention systems.</span></span>
* <span data-ttu-id="12b9a-191">管理者特権ユーザーまたはロールのアクセス権を定期的にレビューします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-191">Regularly review access rights for elevated users or roles.</span></span>
  * <span data-ttu-id="12b9a-192">管理者特権をアクティブ化する資格があるすべてのユーザーを識別します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-192">Identify every user who is eligible to activate admin privilege.</span></span>
* <span data-ttu-id="12b9a-193">オンボーディング、オフボーディング、および資格情報の更新プロセスをレビューします。</span><span class="sxs-lookup"><span data-stu-id="12b9a-193">Review on-boarding, off-boarding, and credential update processes.</span></span>
* <span data-ttu-id="12b9a-194">ID アクセス管理 (IAM) モジュール間の自動化および通信のレベルの増加を調査します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-194">Investigate increasing levels of automation and communication between identity access management (IAM) modules.</span></span>
* <span data-ttu-id="12b9a-195">開発セキュリティ運用 (DevSecOps) アプローチの実装を検討します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-195">Consider implementing a development security operations (DevSecOps) approach.</span></span>
* <span data-ttu-id="12b9a-196">影響分析を実行して、コスト、セキュリティ、およびユーザー導入に関する結果を測定します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-196">Carry out an impact analysis to gauge results on costs, security, and user adoption.</span></span>
* <span data-ttu-id="12b9a-197">システムによって作成されるメトリックの変化を示す影響レポートを定期的に生成し、[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)のビジネスへの影響を見積もります。</span><span class="sxs-lookup"><span data-stu-id="12b9a-197">Periodically produce an impact report that shows the changes in metrics created by the system and estimate the business impacts of the [Hybrid Identity Strategy](../../decision-guides/identity/overview.md).</span></span>
* <span data-ttu-id="12b9a-198">[Azure Security Center](/azure/security-center/security-center-intro) によって推奨されている統合された監視を確立します。</span><span class="sxs-lookup"><span data-stu-id="12b9a-198">Establish integrated monitoring recommended by the [Azure Security Center](/azure/security-center/security-center-intro).</span></span>

## <a name="next-steps"></a><span data-ttu-id="12b9a-199">次の手順</span><span class="sxs-lookup"><span data-stu-id="12b9a-199">Next steps</span></span>

<span data-ttu-id="12b9a-200">ここまでで、クラウド ID ガバナンスの概念を理解したので、[ID ベースライン ツールチェーン](toolchain.md)を検証して、Azure プラットフォーム上で ID ベースライン ガバナンス規範を開発するときに必要になる Azure のツールおよび機能を識別してください。</span><span class="sxs-lookup"><span data-stu-id="12b9a-200">Now that you understand the concept of cloud identity governance, examine the [Identity Baseline toolchain](toolchain.md) to identify Azure tools and features that you'll need when developing the Identity Baseline governance discipline on the Azure platform.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="12b9a-201">Azure の ID ベースライン ツールチェーン</span><span class="sxs-lookup"><span data-stu-id="12b9a-201">Identity Baseline toolchain for Azure</span></span>](toolchain.md)