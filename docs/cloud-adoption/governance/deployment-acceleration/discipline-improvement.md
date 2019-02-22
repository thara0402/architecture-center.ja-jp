---
title: CAF:デプロイ加速規範の改良
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: デプロイ加速規範の改良
author: alexbuckgit
ms.openlocfilehash: 98192c777d8866efb01544737e8cabea6354c4d7
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902193"
---
# <a name="deployment-acceleration-discipline-improvement"></a><span data-ttu-id="f6f73-103">デプロイ加速規範の改良</span><span class="sxs-lookup"><span data-stu-id="f6f73-103">Deployment Acceleration discipline improvement</span></span>

<span data-ttu-id="f6f73-104">デプロイ加速規範では、リソースのデプロイと構成が一貫性を持って繰り返し実行され、それらのライフサイクル中、常にコンプライアンスが維持されるポリシーを確立することを重視しています。</span><span class="sxs-lookup"><span data-stu-id="f6f73-104">The Deployment Acceleration discipline focuses on establishing policies that ensure that resources are deployed and configured consistently and repeatably, and remain in compliance throughout their lifecycle.</span></span> <span data-ttu-id="f6f73-105">クラウド ガバナンスの 5 つの規範の中で、デプロイ加速規範には、デプロイの自動化、デプロイの成果物のソース管理、デプロイされたリソースの管理による目的の状態の維持、およびコンプライアンス問題の監査に関する決定が含まれています。</span><span class="sxs-lookup"><span data-stu-id="f6f73-105">Within the Five Disciplines of Cloud Governance, Deployment Acceleration includes decisions regarding automating deployments, source-controlling deployment artifacts, monitoring deployed resources to maintain desired state, and auditing any compliance issues.</span></span>

<span data-ttu-id="f6f73-106">この記事では、会社がデプロイ加速規範を適切に開発し、成熟させるために実行できるいくつかの潜在的なタスクの概要について説明します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-106">This article outlines some potential tasks your company can engage in to better develop and mature the Deployment Acceleration discipline.</span></span> <span data-ttu-id="f6f73-107">これらのタスクは、クラウド ソリューションを実装するための計画、構築、導入、および運用フェーズに分割でき、それらが反復処理されて、[クラウド ガバナンスに対する漸進的アプローチ](../journeys/overview.md#an-incremental-approach-to-cloud-governance)を発展させることができます。</span><span class="sxs-lookup"><span data-stu-id="f6f73-107">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![導入の 4 つのフェーズ](../../_images/adoption-phases.png)

<span data-ttu-id="f6f73-109">*図 1: クラウド ガバナンスに対する漸進的アプローチの導入フェーズ*</span><span class="sxs-lookup"><span data-stu-id="f6f73-109">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="f6f73-110">1 つのドキュメントですべての企業の要件を説明することはできません。</span><span class="sxs-lookup"><span data-stu-id="f6f73-110">It's impossible for any one document to account for the requirements of all businesses.</span></span> <span data-ttu-id="f6f73-111">そのため、この記事では、ガバナンス成熟プロセスのフェーズごとに推奨される最小限のアクティビティと潜在的なアクティビティの例の概要を説明します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-111">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="f6f73-112">これらのアクティビティの最初の目標は、お客様が[ポリシーの MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) を構築し、漸進的にポリシーを進化させるためのフレームワークを確立できるように支援することです。</span><span class="sxs-lookup"><span data-stu-id="f6f73-112">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="f6f73-113">お客様のクラウド ガバナンス チームは、ID ベースライン ガバナンス機能を強化するために、これらのアクティビティにどれだけ投資するかを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f6f73-113">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Identity Baseline governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="f6f73-114">この記事で概要を説明する最小限のアクティビティと潜在的なアクティビティのどちらも、特定の企業ポリシーやサードパーティのコンプライアンス要件には整合していません。</span><span class="sxs-lookup"><span data-stu-id="f6f73-114">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="f6f73-115">このガイダンスは、両方の要件とクラウド ガバナンス モデルの整合に導く会話の促進に役立つように設計されています。</span><span class="sxs-lookup"><span data-stu-id="f6f73-115">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="f6f73-116">計画と準備</span><span class="sxs-lookup"><span data-stu-id="f6f73-116">Planning and readiness</span></span>

<span data-ttu-id="f6f73-117">ガバナンス成熟のこのフェーズは、ビジネスの成果とすぐに実行できる戦略の間のギャップを橋渡しします。</span><span class="sxs-lookup"><span data-stu-id="f6f73-117">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="f6f73-118">このプロセス中に、リーダーシップ チームは、特定のメトリックを定義し、これらのメトリックをデジタル資産にマップして、移行作業全体の計画を開始します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-118">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="f6f73-119">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-119">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="f6f73-120">[デプロイ加速ツールチェーン](toolchain.md) オプションを評価し、ご自身の組織に適したハイブリッド戦略を実装します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-120">Evaluate your [Deployment Acceleration toolchain](toolchain.md) options and implement a hybrid strategy that is appropriate to your organization.</span></span>
- <span data-ttu-id="f6f73-121">アーキテクチャ ガイドライン ドキュメントのドラフトを作成し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-121">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
- <span data-ttu-id="f6f73-122">アーキテクチャ ガイドラインの作成によって影響を受ける人物とチームを教育し、関与させます。</span><span class="sxs-lookup"><span data-stu-id="f6f73-122">Educate and involve the people and teams affected by the development of Architecture Guidelines.</span></span>
- <span data-ttu-id="f6f73-123">開発チームと IT スタッフをトレーニングして、デプロイ加速規範における DevSecOps の原則と戦略、および完全に自動化されたデプロイの重要性を理解してもらいます。</span><span class="sxs-lookup"><span data-stu-id="f6f73-123">Train development teams and IT staff to understand DevSecOps principles and strategies and the importance of fully automated deployments in the Deployment Acceleration Discipline.</span></span>

<span data-ttu-id="f6f73-124">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-124">**Potential activities:**</span></span>

- <span data-ttu-id="f6f73-125">クラウドでのデプロイの加速を管理するロールと割り当てを定義します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-125">Define roles and assignments that will govern Deployment Acceleration in the cloud.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="f6f73-126">構築とデプロイ前の準備</span><span class="sxs-lookup"><span data-stu-id="f6f73-126">Build and pre-deployment</span></span>

<span data-ttu-id="f6f73-127">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-127">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="f6f73-128">新しいクラウドベースのアプリケーションでは、完全に自動化されたデプロイを早期の開発プロセスに導入します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-128">For new cloud-based applications, introduce fully automated deployments early in the development process.</span></span> <span data-ttu-id="f6f73-129">この投資によって、テスト プロセスの信頼性が向上し、開発、QA、および運用環境における一貫性が確保されます。</span><span class="sxs-lookup"><span data-stu-id="f6f73-129">This investment will improve the reliability of your testing processes and ensure consistency across your development, QA, and production environments.</span></span>
- <span data-ttu-id="f6f73-130">デプロイ テンプレートや構成スクリプトなどのデプロイのすべての成果物を、GitHub や Azure DevOps などのソース管理プラットフォームを使用して格納します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-130">Store all deployment artifacts such as deployment templates or configuration scripts using a source-control platform such as GitHub or Azure DevOps.</span></span>
- <span data-ttu-id="f6f73-131">ご自身の[デプロイ加速ツールチェーン](toolchain.md)を実装する前にパイロット テストの実施を検討し、デプロイが可能な限り合理化されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-131">Consider a pilot test before implementing your [Deployment Acceleration toolchain](toolchain.md), making sure it streamlines your deployments as much as possible.</span></span> <span data-ttu-id="f6f73-132">デプロイ前の段階で、必要に応じてパイロット テストからのフィードバックを繰り返し適用します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-132">Apply feedback from pilot tests during the pre-deployment phase, repeating as needed.</span></span>
- <span data-ttu-id="f6f73-133">アプリケーションの論理および物理アーキテクチャを評価し、他のクラウドベースのリソースを使用して、アプリケーションのリソースのデプロイを自動化したり、アーキテクチャの特定の部分を改良したりする機会を特定します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-133">Evaluate the logical and physical architecture of your applications, and identify opportunities to automate the deployment of application resources or improve portions of the architecture using other cloud-based resources.</span></span>
- <span data-ttu-id="f6f73-134">デプロイとユーザー導入計画を含むようにアーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-134">Update the Architecture Guidelines document to include deployment and user adoption plans, and distribute to key stakeholders.</span></span>
- <span data-ttu-id="f6f73-135">アーキテクチャ ガイドラインの影響を最も受ける人物とチームの教育を続行します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-135">Continue to educate the people and teams most affected by the architecture guidelines.</span></span>

<span data-ttu-id="f6f73-136">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-136">**Potential activities:**</span></span>

- <span data-ttu-id="f6f73-137">継続的インテグレーションと継続的デプロイ (CI/CD) パイプラインを定義して、開発、QA、および運用環境でのアプリケーションへの更新のリリースを完全に管理します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-137">Define a continuous integration and continuous deployment (CI/CD) pipeline to fully manage releasing updates to your application through your development, QA, and production environments.</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="f6f73-138">導入と移行</span><span class="sxs-lookup"><span data-stu-id="f6f73-138">Adopt and migrate</span></span>

<span data-ttu-id="f6f73-139">移行は、既存のデジタル資産内のアプリケーションまたはワークロードの移動、テスト、および導入に重点を置いた漸進的プロセスです。</span><span class="sxs-lookup"><span data-stu-id="f6f73-139">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="f6f73-140">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-140">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="f6f73-141">お使いの[デプロイ加速ツールチェーン](toolchain.md)を開発から運用に移行します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-141">Migrate your [Deployment Acceleration toolchain](toolchain.md) from development to production.</span></span>
- <span data-ttu-id="f6f73-142">アーキテクチャ ガイドライン ドキュメントを更新し、主な利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-142">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
- <span data-ttu-id="f6f73-143">開発と IT 導入の促進に役立つ教材とドキュメント、認知の伝達、インセンティブなどのプログラムを開発します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-143">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive developer and IT adoption.</span></span>

<span data-ttu-id="f6f73-144">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-144">**Potential activities:**</span></span>

- <span data-ttu-id="f6f73-145">構築とデプロイ前フェーズ中に定義されたベスト プラクティスが正しく実行されていることを検証します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-145">Validate that the best practices defined during the build and pre-deployment phases are properly executed.</span></span>
- <span data-ttu-id="f6f73-146">リリースする前に、各アプリケーションまたはワークロードがデプロイ加速戦略と整合していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-146">Ensure that each application or workload aligns with the Deployment Acceleration strategy before release.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="f6f73-147">運用と実装後</span><span class="sxs-lookup"><span data-stu-id="f6f73-147">Operate and post-implementation</span></span>

<span data-ttu-id="f6f73-148">変換が完了したら、アプリケーションまたはワークロードの自然なライフサイクルに対して、ガバナンスと運用を続行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f6f73-148">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="f6f73-149">ガバナンス成熟のこのフェーズでは、ソリューションが実装され、変換サイクルが安定し始めた後に一般的に実行されるアクティビティに重点が置かれます。</span><span class="sxs-lookup"><span data-stu-id="f6f73-149">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="f6f73-150">**最小限の推奨されるアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-150">**Minimum suggested activities:**</span></span>

- <span data-ttu-id="f6f73-151">組織の変化する ID ニーズに対する変更に基づいて、お使いの[デプロイ加速ツールチェーン](toolchain.md)をカスタマイズします。</span><span class="sxs-lookup"><span data-stu-id="f6f73-151">Customize your [Deployment Acceleration toolchain](toolchain.md) based on changes to your organization’s changing identity needs.</span></span>
- <span data-ttu-id="f6f73-152">潜在的な構成の問題や悪意のある脅威が通知されるように、通知とレポートを自動化します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-152">Automate notifications and reports to alert you of potential configuration issues or malicious threats.</span></span>
- <span data-ttu-id="f6f73-153">アプリケーションとリソースの使用状況を監視して報告します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-153">Monitor and report on application and resource usage.</span></span>
- <span data-ttu-id="f6f73-154">デプロイ後のメトリックに関するレポートを作成し、利害関係者に配布します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-154">Report on post-deployment metrics and distribute to stakeholders.</span></span>
- <span data-ttu-id="f6f73-155">将来の導入プロセスのガイドとなるようにアーキテクチャ ガイドラインを改訂します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-155">Revise the Architecture Guidelines to guide future adoption processes.</span></span>
- <span data-ttu-id="f6f73-156">影響を受ける人物とチームとのやり取りを続け、トレーニングを定期的に実施して、アーキテクチャ ガイドラインへの順守が継続されるようにします。</span><span class="sxs-lookup"><span data-stu-id="f6f73-156">Continue to communicate with and train the affected people and teams on a regular basis to ensure ongoing adherence to Architecture Guidelines.</span></span>

<span data-ttu-id="f6f73-157">**潜在的なアクティビティ:**</span><span class="sxs-lookup"><span data-stu-id="f6f73-157">**Potential activities:**</span></span>

- <span data-ttu-id="f6f73-158">目的の状態の監視とレポート作成を行うツールを構成します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-158">Configure a desired state configuration monitoring and reporting tool.</span></span>
- <span data-ttu-id="f6f73-159">構成ツールとスクリプトを定期的に見直して、プロセスを改善し、共通する問題を識別します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-159">Regularly review configuration tools and scripts to improve processes and identify common issues.</span></span>
- <span data-ttu-id="f6f73-160">開発、運用、およびセキュリティ チームと協力して、DevSecOps プラクティスの成熟を支援し、非効率につながる組織のサイロ化を解消します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-160">Work with development, operations, and security teams to help mature DevSecOps practices and break down organizational silos that lead to inefficiencies.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f6f73-161">次の手順</span><span class="sxs-lookup"><span data-stu-id="f6f73-161">Next steps</span></span>

<span data-ttu-id="f6f73-162">クラウド ID ガバナンスの概念を理解したので、[ID ベースライン ツールチェーン](toolchain.md)を調べて、Azure プラットフォームの ID ベースライン ガバナンス規範を開発する際に必要になる Azure のツールと機能を識別します。</span><span class="sxs-lookup"><span data-stu-id="f6f73-162">Now that you understand the concept of cloud identity governance, examine the [Identity Baseline toolchain](toolchain.md) to identify Azure tools and features that you'll need when developing the Identity Baseline governance discipline on the Azure platform.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="f6f73-163">Azure の ID ベースライン ツールチェーン</span><span class="sxs-lookup"><span data-stu-id="f6f73-163">Identity Baseline toolchain for Azure</span></span>](toolchain.md)
