---
title: 'CAF: 中小企業 - マルチクラウドの進化'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 中小企業 - マルチクラウドの進化
author: BrianBlanchard
ms.openlocfilehash: 8bfe56f999ddef9d954fad9a157277301d81a98e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243663"
---
# <a name="small-to-medium-enterprise-multi-cloud-evolution"></a><span data-ttu-id="3b738-103">中小企業: マルチクラウドの進化</span><span class="sxs-lookup"><span data-stu-id="3b738-103">Small-to-medium enterprise: Multi-cloud evolution</span></span>

<span data-ttu-id="3b738-104">この記事では、マルチクラウドの導入のためにコントロールを追加することでストーリーを展開します。</span><span class="sxs-lookup"><span data-stu-id="3b738-104">This article evolves the narrative by adding controls for multi-cloud adoption.</span></span>

## <a name="evolution-of-the-narrative"></a><span data-ttu-id="3b738-105">ストーリーの進化</span><span class="sxs-lookup"><span data-stu-id="3b738-105">Evolution of the narrative</span></span>

<span data-ttu-id="3b738-106">Microsoft では、お客様が特定の目的で複数のクラウドを採用していることを認識しています。</span><span class="sxs-lookup"><span data-stu-id="3b738-106">Microsoft recognizes that customers are adopting multiple clouds for specific purposes.</span></span> <span data-ttu-id="3b738-107">この体験の架空の会社も例外ではありません。</span><span class="sxs-lookup"><span data-stu-id="3b738-107">The fictional customer in this journey is no exception.</span></span> <span data-ttu-id="3b738-108">Azure の導入体験と並行して、ビジネスの成功は、小規模ながらも補完的なビジネスの取得へとつながりました。</span><span class="sxs-lookup"><span data-stu-id="3b738-108">In parallel to the Azure adoption journey, the business success has led to the acquisition of a small, but complementary business.</span></span> <span data-ttu-id="3b738-109">そのビジネスは、別のクラウド プロバイダーですべての IT 操作を実行しています。</span><span class="sxs-lookup"><span data-stu-id="3b738-109">That business is running all of their IT operations on a different cloud provider.</span></span>

<span data-ttu-id="3b738-110">この記事では、新しい組織を統合するときに物事がどのように変化するかについて説明します。</span><span class="sxs-lookup"><span data-stu-id="3b738-110">This article describes how things change when integrating the new organization.</span></span> <span data-ttu-id="3b738-111">このストーリーでは、この会社はこの顧客体験で示されているガバナンスの各進化を完了しているものとします。</span><span class="sxs-lookup"><span data-stu-id="3b738-111">For purposes of the narrative, we assume this company has completed each of the governance evolutions outlined in this customer journey.</span></span>

### <a name="evolution-of-the-current-state"></a><span data-ttu-id="3b738-112">現在の状態の進化</span><span class="sxs-lookup"><span data-stu-id="3b738-112">Evolution of the current state</span></span>

<span data-ttu-id="3b738-113">このストーリーの前の段階で、この会社は CI/CD パイプラインを介して運用環境アプリケーションをクラウドに積極的に移行し始めました。</span><span class="sxs-lookup"><span data-stu-id="3b738-113">In the previous phase of this narrative, the company had begun actively pushing production applications to the cloud through CI/CD pipelines.</span></span>

<span data-ttu-id="3b738-114">その後、以下に示すように、ガバナンスに影響を与えるいくつかの変化がありました。</span><span class="sxs-lookup"><span data-stu-id="3b738-114">Since then, some things have changed that will affect governance:</span></span>

- <span data-ttu-id="3b738-115">ID は、Active Directory のオンプレミス インスタンスによって管理されている。</span><span class="sxs-lookup"><span data-stu-id="3b738-115">Identity is controlled by an on-premises instance of Active Directory.</span></span> <span data-ttu-id="3b738-116">Azure Active Directory へのレプリケーションによりハイブリッド ID が促進された。</span><span class="sxs-lookup"><span data-stu-id="3b738-116">Hybrid identity is facilitated through replication to Azure Active Directory.</span></span>
- <span data-ttu-id="3b738-117">IT の操作やクラウドの操作の多くは、Azure Monitor と関連する自動化によって管理されている。</span><span class="sxs-lookup"><span data-stu-id="3b738-117">IT Operations or Cloud Operations are largely managed by Azure Monitor and related automations.</span></span>
- <span data-ttu-id="3b738-118">ディザスター リカバリーとビジネス継続性は Azure コンテナー インスタンスによって制御されている。</span><span class="sxs-lookup"><span data-stu-id="3b738-118">Disaster Recovery / Business Continuity is controlled by Azure Vault instances.</span></span>
- <span data-ttu-id="3b738-119">Azure Security Center を使用して、セキュリティ違反や攻撃を監視している。</span><span class="sxs-lookup"><span data-stu-id="3b738-119">Azure Security Center is used to monitor security violations and attacks.</span></span>
- <span data-ttu-id="3b738-120">Azure Security Center と Azure Monitor を使用して、クラウドのガバナンスを監視している。</span><span class="sxs-lookup"><span data-stu-id="3b738-120">Azure Security Center and Azure Monitor are both used to monitor governance of the cloud.</span></span>
- <span data-ttu-id="3b738-121">Azure Blueprints、Azure Policy、および Azure 管理グループを使用して、ポリシーのコンプライアンスを自動化している。</span><span class="sxs-lookup"><span data-stu-id="3b738-121">Azure Blueprints, Azure Policy, and Azure management groups are used to automate compliance with policy.</span></span>

### <a name="evolution-of-the-future-state"></a><span data-ttu-id="3b738-122">将来の状態の進化</span><span class="sxs-lookup"><span data-stu-id="3b738-122">Evolution of the future state</span></span>

<span data-ttu-id="3b738-123">目標は、取得した会社を、可能な限り既存事業に統合することです。</span><span class="sxs-lookup"><span data-stu-id="3b738-123">The goal is to integrate the acquisition company into existing operations wherever possible.</span></span>

## <a name="evolution-of-tangible-risks"></a><span data-ttu-id="3b738-124">具体的なリスクの進化</span><span class="sxs-lookup"><span data-stu-id="3b738-124">Evolution of tangible risks</span></span>

<span data-ttu-id="3b738-125">**ビジネスの取得コスト**:新しいビジネスの取得は、約 5 年間で利益が出る予定です。</span><span class="sxs-lookup"><span data-stu-id="3b738-125">**Business acquisition cost**: Acquisition of the new business is slated to be profitable in approximately five years.</span></span> <span data-ttu-id="3b738-126">利益が出るまでに時間がかるため、取締役会は、できるだけ取得コストを管理することを希望しています。</span><span class="sxs-lookup"><span data-stu-id="3b738-126">Because of the slow rate of return, the board wants to control acquisition costs, as much as possible.</span></span> <span data-ttu-id="3b738-127">コスト管理と技術的統合が互いに競合するリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="3b738-127">There is a risk of cost control and technical integration conflicting with one another.</span></span>

<span data-ttu-id="3b738-128">このビジネス上のリスクは、いくつかの技術的リスクへと拡大する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-128">This business risk can be expanded into a few technical risks:</span></span>

- <span data-ttu-id="3b738-129">クラウドの移行によって追加の取得コストが生まれるリスクがある。</span><span class="sxs-lookup"><span data-stu-id="3b738-129">Cloud migration might produce additional acquisition costs</span></span>
- <span data-ttu-id="3b738-130">新しい環境は適切に管理されていない可能性があり、その結果、ポリシー違反が発生する可能性がある。</span><span class="sxs-lookup"><span data-stu-id="3b738-130">The new environment might not be properly governed which could result in policy violations.</span></span>

## <a name="evolution-of-the-policy-statements"></a><span data-ttu-id="3b738-131">ポリシー ステートメントの進化</span><span class="sxs-lookup"><span data-stu-id="3b738-131">Evolution of the policy statements</span></span>

<span data-ttu-id="3b738-132">ポリシーに対する次の変更は、新しいリスクを軽減して実装を導くのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="3b738-132">The following changes to policy will help mitigate the new risks and guide implementation.</span></span>

1. <span data-ttu-id="3b738-133">セカンダリ クラウド内のすべての資産は、既存の運用管理ツールとセキュリティ監視ツールを使用して監視する必要がある</span><span class="sxs-lookup"><span data-stu-id="3b738-133">All assets in a secondary cloud must be monitored through existing operational management and security monitoring tools</span></span>
2. <span data-ttu-id="3b738-134">すべての組織単位は、既存の ID プロバイダーに統合する必要がある</span><span class="sxs-lookup"><span data-stu-id="3b738-134">All Organization Units must be integrated into the existing identity provider</span></span>
3. <span data-ttu-id="3b738-135">プライマリ ID プロバイダーが、セカンダリ クラウド内の資産への認証を管理する</span><span class="sxs-lookup"><span data-stu-id="3b738-135">The primary identity provider should govern authentication to assets in the secondary cloud</span></span>

## <a name="evolution-of-the-best-practices"></a><span data-ttu-id="3b738-136">ベスト プラクティスの進化</span><span class="sxs-lookup"><span data-stu-id="3b738-136">Evolution of the best practices</span></span>

<span data-ttu-id="3b738-137">記事のこのセクションでは、ガバナンス MVP の設計を進化させて、新しい Azure ポリシーと Azure Cost Management の実装を含めます。</span><span class="sxs-lookup"><span data-stu-id="3b738-137">This section of the article will evolve the governance MVP design to include new Azure policies and an implementation of Azure Cost Management.</span></span> <span data-ttu-id="3b738-138">これら 2 つの設計変更を組み合わせることで、会社の新しいポリシー ステートメントを実現します。</span><span class="sxs-lookup"><span data-stu-id="3b738-138">Together, these two design changes will fulfill the new corporate policy statements.</span></span>

1. <span data-ttu-id="3b738-139">ネットワークを接続する。</span><span class="sxs-lookup"><span data-stu-id="3b738-139">Connect the networks.</span></span> <span data-ttu-id="3b738-140">この手順は、クラウド ガバナンス チームの支援を受け、ネットワークおよび IT セキュリティ チームが実行します。</span><span class="sxs-lookup"><span data-stu-id="3b738-140">This step is executed by the Networking and IT Security teams, and supported by the Cloud Governance team.</span></span> <span data-ttu-id="3b738-141">MPLS/専用回線のプロバイダーから新しいクラウドに接続を追加することにより、ネットワークが統合されます。</span><span class="sxs-lookup"><span data-stu-id="3b738-141">Adding a connection from the MPLS/leased-line provider to the new cloud will integrate networks.</span></span> <span data-ttu-id="3b738-142">ルーティング テーブルとファイアウォールの構成を追加することにより、環境の間のアクセスとトラフィックを管理します。</span><span class="sxs-lookup"><span data-stu-id="3b738-142">Adding routing tables and firewall configurations will control access and traffic between the environments.</span></span>
2. <span data-ttu-id="3b738-143">ID プロバイダーを統合する。</span><span class="sxs-lookup"><span data-stu-id="3b738-143">Consolidate identity providers.</span></span> <span data-ttu-id="3b738-144">セカンダリ クラウドでホストされているワークロードに応じて、ID プロバイダーの統合に対してさまざまな選択肢があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-144">Depending on the workloads being hosted in the secondary cloud, there are a variety of options to identity provider consolidation.</span></span> <span data-ttu-id="3b738-145">以下に例を示します。</span><span class="sxs-lookup"><span data-stu-id="3b738-145">The following are a few examples:</span></span>
    1. <span data-ttu-id="3b738-146">OAuth 2 を使用して認証するアプリケーションの場合、セカンダリ クラウド内の Active Directory のユーザーは、単純に既存の Azure AD テナントにレプリケートできます。</span><span class="sxs-lookup"><span data-stu-id="3b738-146">For applications that authenticate using OAuth 2, users from Active Directory in the secondary cloud can simply be replicated to the existing Azure AD tenant.</span></span> <span data-ttu-id="3b738-147">これにより、すべてのユーザーがテナント内で認証されるようになります。</span><span class="sxs-lookup"><span data-stu-id="3b738-147">This ensures all users can be authenticated in the tenant.</span></span>
    2. <span data-ttu-id="3b738-148">一方、フェデレーションの場合は、OU をオンプレミスの Active Directory 経由で Azure AD インスタンスに送ることができます。</span><span class="sxs-lookup"><span data-stu-id="3b738-148">At the other extreme, federation allows OUs to flow into Active Directory on-premises, then into the Azure AD instance.</span></span>
3. <span data-ttu-id="3b738-149">Azure Site Recovery に資産を追加する。</span><span class="sxs-lookup"><span data-stu-id="3b738-149">Add assets to Azure Site Recovery.</span></span>
    1. <span data-ttu-id="3b738-150">Azure Site Recovery は、最初からハイブリッド/マルチクラウド ツールとして設計されました。</span><span class="sxs-lookup"><span data-stu-id="3b738-150">Azure Site Recovery was designed from the beginning as a hybrid/multi-cloud tool.</span></span>
    2. <span data-ttu-id="3b738-151">セカンダリ クラウド内の VM は、オンプレミスの資産の保護に使用されているのと同じ Azure Site Recovery プロセスによって保護できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-151">VMs in the secondary cloud might be able to be protected by the same Azure Site Recovery processes used to protect on-premises assets.</span></span>
4. <span data-ttu-id="3b738-152">Azure Cost Management に資産を追加する</span><span class="sxs-lookup"><span data-stu-id="3b738-152">Add assets to Azure Cost Management</span></span>
    1. <span data-ttu-id="3b738-153">Azure Cost Management は、最初からマルチクラウド ツールとして設計されました。</span><span class="sxs-lookup"><span data-stu-id="3b738-153">Azure Cost Management was designed from the beginning as a multi-cloud tool.</span></span>
    2. <span data-ttu-id="3b738-154">セカンダリ クラウド内の仮想マシンは、一部のクラウド プロバイダーの Azure Cost Management と互換性がある場合があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-154">Virtual machines in the secondary cloud may be compatible with Azure Cost Management for some cloud providers.</span></span> <span data-ttu-id="3b738-155">追加コストが適用される場合があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-155">Additional costs may apply.</span></span>
5. <span data-ttu-id="3b738-156">Azure Monitor に資産を追加する。</span><span class="sxs-lookup"><span data-stu-id="3b738-156">Add assets to Azure Monitor.</span></span>
    1. <span data-ttu-id="3b738-157">Azure Monitor は、最初からハイブリッド クラウド ツールとして設計されました。</span><span class="sxs-lookup"><span data-stu-id="3b738-157">Azure Monitor was designed as a hybrid cloud tool from inception.</span></span>
    2. <span data-ttu-id="3b738-158">セカンダリ クラウド内の仮想マシンは、Azure Monitor エージェントと互換性があり、運用監視のために Azure Monitor に含めることができる場合があります。</span><span class="sxs-lookup"><span data-stu-id="3b738-158">Virtual machines in the secondary cloud may be compatible with Azure Monitor agents, allowing them to be included in Azure Monitor for operational monitoring.</span></span>
6. <span data-ttu-id="3b738-159">ガバナンス実施ツール:</span><span class="sxs-lookup"><span data-stu-id="3b738-159">Governance enforcement tools:</span></span>
    1. <span data-ttu-id="3b738-160">ガバナンス実施はクラウド固有です。</span><span class="sxs-lookup"><span data-stu-id="3b738-160">Governance enforcement is cloud-specific.</span></span>
    2. <span data-ttu-id="3b738-161">ガバナンス体験で設定した会社のポリシーはクラウド固有ではありません。</span><span class="sxs-lookup"><span data-stu-id="3b738-161">The corporate policies established in the governance journey are not cloud-specific.</span></span> <span data-ttu-id="3b738-162">実装はクラウドごとに異なる可能性がありますが、ポリシーはセカンダリ プロバイダーに適用できます。</span><span class="sxs-lookup"><span data-stu-id="3b738-162">While the implementation may vary from cloud to cloud, the policies can be applied to the secondary provider.</span></span>

<span data-ttu-id="3b738-163">マルチクラウドの導入が増えるにつれて、上記の設計の進化も成熟していきます。</span><span class="sxs-lookup"><span data-stu-id="3b738-163">As multi-cloud adoption grows, the design evolution above will continue to mature.</span></span>

## <a name="conclusion"></a><span data-ttu-id="3b738-164">まとめ</span><span class="sxs-lookup"><span data-stu-id="3b738-164">Conclusion</span></span>

<span data-ttu-id="3b738-165">この一連の記事では、この架空の会社のエクスペリエンスに沿って、ガバナンスのベスト プラクティスの進化について概要を説明しました。</span><span class="sxs-lookup"><span data-stu-id="3b738-165">This series of articles outlined the evolution of governance best practices, aligned with the experiences of this fictional company.</span></span> <span data-ttu-id="3b738-166">適切な基盤があれば、小規模から始めても、会社は迅速に移行し、適切なタイミングで適切な量のガバナンスを適用することができます。</span><span class="sxs-lookup"><span data-stu-id="3b738-166">By starting small, but with the right foundation, the company could move quickly and yet still apply the right amount of governance at the right time.</span></span> <span data-ttu-id="3b738-167">MVP 自体ではお客様は保護されませんでした。</span><span class="sxs-lookup"><span data-stu-id="3b738-167">The MVP by itself did not protect the customer.</span></span> <span data-ttu-id="3b738-168">その代わり、リスクを軽減し、保護を追加する基盤が作成されました。</span><span class="sxs-lookup"><span data-stu-id="3b738-168">Instead, it created the foundation to mitigate risk and add protections.</span></span> <span data-ttu-id="3b738-169">そこから、具体的なリスクを軽減するガバナンスの層が適用されました。</span><span class="sxs-lookup"><span data-stu-id="3b738-169">From there, layers of governance were applied to mitigate tangible risks.</span></span> <span data-ttu-id="3b738-170">ここで提示した体験は、あらゆるお客様の体験にそのまま 100% 合うものではありません。</span><span class="sxs-lookup"><span data-stu-id="3b738-170">The exact journey presented here won't align 100% with the experiences of any reader.</span></span> <span data-ttu-id="3b738-171">そうではなく、ガバナンスを段階的に進化させるためのひな形として利用できます。</span><span class="sxs-lookup"><span data-stu-id="3b738-171">Rather, it serves as a pattern for incremental governance.</span></span> <span data-ttu-id="3b738-172">これらのベスト プラクティスは、お客様固有の制約やガバナンス要件に合わせて調整することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3b738-172">The reader is advised to mold these best practices to fit their own unique constraints and governance requirements.</span></span>