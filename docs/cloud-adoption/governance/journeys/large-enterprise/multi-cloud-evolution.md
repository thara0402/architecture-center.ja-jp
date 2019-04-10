---
title: 'CAF: 大企業 – マルチクラウドの進化'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 大企業 – マルチクラウドの進化
author: BrianBlanchard
ms.openlocfilehash: c72040912a99ad232e367ae750e9bb2caa77cbf2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241089"
---
# <a name="large-enterprise-multi-cloud-evolution"></a><span data-ttu-id="82d4d-103">大企業:マルチクラウドの進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-103">Large enterprise: Multi-cloud evolution</span></span>

## <a name="evolution-of-the-narrative"></a><span data-ttu-id="82d4d-104">ストーリーの進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-104">Evolution of the narrative</span></span>

<span data-ttu-id="82d4d-105">Microsoft では、お客様が特定の目的で複数のクラウドを採用していることを認識しています。</span><span class="sxs-lookup"><span data-stu-id="82d4d-105">Microsoft recognizes that customers are adopting multiple clouds for specific purposes.</span></span> <span data-ttu-id="82d4d-106">この体験の架空の会社も例外ではありません。</span><span class="sxs-lookup"><span data-stu-id="82d4d-106">The fictional company in this journey is no exception.</span></span> <span data-ttu-id="82d4d-107">Azure の導入体験と並行して、ビジネスの成功は、小規模ながら補完的なビジネスの取得へとつながりました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-107">In parallel to the Azure adoption journey, the business success has led to the acquisition of a small, but complementary business.</span></span> <span data-ttu-id="82d4d-108">そのビジネスは、別のクラウド プロバイダーですべての IT 操作を実行しています。</span><span class="sxs-lookup"><span data-stu-id="82d4d-108">That business is running all of their IT operations on a different cloud provider.</span></span>

<span data-ttu-id="82d4d-109">この記事では、新しい組織を統合するときに物事がどのように変化するかについて説明します。</span><span class="sxs-lookup"><span data-stu-id="82d4d-109">This article describes how things change when integrating the new organization.</span></span> <span data-ttu-id="82d4d-110">このストーリーでは、この会社はこの顧客体験で示されているガバナンスの各進化を完了しているものとします。</span><span class="sxs-lookup"><span data-stu-id="82d4d-110">For purposes of the narrative, we assume this company has completed each of the governance evolutions outlined in this customer journey.</span></span>

### <a name="evolution-of-the-current-state"></a><span data-ttu-id="82d4d-111">現在の状態の進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-111">Evolution of the current state</span></span>

<span data-ttu-id="82d4d-112">このストーリーの前の段階では、会社は、クラウドの支出が会社の通常運用経費の一部になるときに、コスト管理とコスト監視の実装を開始しました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-112">In the previous phase of this narrative, the company had begun to implement cost controls and cost monitoring, as cloud spending becomes part of the company's regular operational expenses.</span></span>

<span data-ttu-id="82d4d-113">その後、以下に示すように、ガバナンスに影響を与えるいくつかの変化がありました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-113">Since then, some things have changed that will affect governance:</span></span>

- <span data-ttu-id="82d4d-114">ID は、Active Directory のオンプレミス インスタンスによって管理されている。</span><span class="sxs-lookup"><span data-stu-id="82d4d-114">Identity is controlled by an on-premises instance of Active Directory.</span></span> <span data-ttu-id="82d4d-115">Azure Active Directory へのレプリケーションによりハイブリッド ID が促進された。</span><span class="sxs-lookup"><span data-stu-id="82d4d-115">Hybrid Identity is facilitated through replication to Azure Active Directory.</span></span>
- <span data-ttu-id="82d4d-116">IT の操作やクラウドの操作の多くは、Azure Monitor および関連する自動化によって管理されている。</span><span class="sxs-lookup"><span data-stu-id="82d4d-116">IT Operations or Cloud Operations are largely managed by Azure Monitor and related automations.</span></span>
- <span data-ttu-id="82d4d-117">ディザスター リカバリーとビジネス継続性は Azure コンテナー インスタンスによって制御されている。</span><span class="sxs-lookup"><span data-stu-id="82d4d-117">Disaster Recovery / Business Continuity is controlled by Azure Vault instances.</span></span>
- <span data-ttu-id="82d4d-118">Azure Security Center を使用して、セキュリティ違反や攻撃を監視している。</span><span class="sxs-lookup"><span data-stu-id="82d4d-118">Azure Security Center is used to monitor security violations and attacks.</span></span>
- <span data-ttu-id="82d4d-119">Azure Security Center と Azure Monitor を使用して、クラウドのガバナンスを監視している。</span><span class="sxs-lookup"><span data-stu-id="82d4d-119">Azure Security Center and Azure Monitor are both used to monitor governance of the cloud.</span></span>
- <span data-ttu-id="82d4d-120">Azure Blueprints、Azure Policy、および管理グループを使用して、ポリシーのコンプライアンスを自動化している。</span><span class="sxs-lookup"><span data-stu-id="82d4d-120">Azure Blueprints, Azure Policy, and management groups are used to automate compliance to policy.</span></span>

### <a name="evolution-of-the-future-state"></a><span data-ttu-id="82d4d-121">将来の状態の進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-121">Evolution of the future state</span></span>

<span data-ttu-id="82d4d-122">目標は、取得した会社を、可能な限り既存事業に統合することです。</span><span class="sxs-lookup"><span data-stu-id="82d4d-122">The goal is to integrate the acquisition company into existing operations wherever possible.</span></span>

## <a name="evolution-of-tangible-risks"></a><span data-ttu-id="82d4d-123">具体的なリスクの進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-123">Evolution of tangible risks</span></span>

<span data-ttu-id="82d4d-124">**ビジネスの取得コスト**:新しいビジネスの取得は、約 5 年間で黒字化する見込みです。</span><span class="sxs-lookup"><span data-stu-id="82d4d-124">**Business Acquisition Cost**: Acquisition of the new business is slated to be profitable in approximately five years.</span></span> <span data-ttu-id="82d4d-125">利益が出るまでに時間がかるため、取締役会は、できるだけ取得コストを管理することを希望しています。</span><span class="sxs-lookup"><span data-stu-id="82d4d-125">Because of the slow rate of return, the board wants to control acquisition costs, as much as possible.</span></span> <span data-ttu-id="82d4d-126">コスト管理と技術的統合が互いに競合するというリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-126">There is a risk of cost control and technical integration conflicting with one another.</span></span>

<span data-ttu-id="82d4d-127">このビジネス上のリスクは、いくつかの技術的リスクに発展する可能性があります</span><span class="sxs-lookup"><span data-stu-id="82d4d-127">This business risk can be expanded into a few technical risks</span></span>

- <span data-ttu-id="82d4d-128">クラウドの移行によって追加の取得コストが生まれるリスクがある。</span><span class="sxs-lookup"><span data-stu-id="82d4d-128">There is risk of cloud migration producing additional acquisition costs.</span></span>
- <span data-ttu-id="82d4d-129">また、新しい環境が適切に管理されていなかったり、ポリシー違反が発生したりするリスクもある。</span><span class="sxs-lookup"><span data-stu-id="82d4d-129">There is also a risk of the new environment not being properly governed or resulting in policy violations.</span></span>

## <a name="evolution-of-the-policy-statements"></a><span data-ttu-id="82d4d-130">ポリシー ステートメントの進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-130">Evolution of the policy statements</span></span>

<span data-ttu-id="82d4d-131">ポリシーに対する次の変更は、新しいリスクを軽減して実装を導くのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-131">The following changes to policy will help mitigate the new risks and guide implementation.</span></span>

1. <span data-ttu-id="82d4d-132">セカンダリ クラウド内のすべての資産は、既存の運用管理ツールとセキュリティ監視ツールを使用して監視する必要がある。</span><span class="sxs-lookup"><span data-stu-id="82d4d-132">All assets in a secondary cloud must be monitored through existing operational management and security monitoring tools.</span></span>
2. <span data-ttu-id="82d4d-133">すべての組織単位は、既存の ID プロバイダーに統合する必要がある。</span><span class="sxs-lookup"><span data-stu-id="82d4d-133">All organizational units must be integrated into the existing identity provider.</span></span>
3. <span data-ttu-id="82d4d-134">プライマリ ID プロバイダーが、セカンダリ クラウド内の資産への認証を管理する。</span><span class="sxs-lookup"><span data-stu-id="82d4d-134">The primary identity provider should govern authentication to assets in the secondary cloud.</span></span>

## <a name="evolution-of-the-best-practices"></a><span data-ttu-id="82d4d-135">ベスト プラクティスの進化</span><span class="sxs-lookup"><span data-stu-id="82d4d-135">Evolution of the best practices</span></span>

<span data-ttu-id="82d4d-136">記事のこのセクションでは、ガバナンス MVP の設計を進化させて、新しい Azure ポリシーと Azure Cost Management の実装を含めます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-136">This section of the article will evolve the governance MVP design to include new Azure policies and an implementation of Azure Cost Management.</span></span> <span data-ttu-id="82d4d-137">これら 2 つの設計変更を組み合わせることで、会社の新しいポリシー ステートメントを実現します。</span><span class="sxs-lookup"><span data-stu-id="82d4d-137">Together, these two design changes will fulfill the new corporate policy statements.</span></span>

1. <span data-ttu-id="82d4d-138">ネットワークを接続する。ガバナンスによってサポートされるネットワーキングと IT セキュリティによって実行されます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-138">Connect the networks - Executed by Networking and IT Security, supported by governance</span></span>
    1. <span data-ttu-id="82d4d-139">MPLS/専用回線の行のプロバイダーから新しいクラウドに接続を追加することにより、ネットワークが統合されます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-139">Adding a connection from the MPLS/Leased line provider to the new cloud will integrate networks.</span></span> <span data-ttu-id="82d4d-140">ルーティング テーブルとファイアウォールの構成を追加することにより、環境の間のアクセスとトラフィックを管理します。</span><span class="sxs-lookup"><span data-stu-id="82d4d-140">Adding routing tables and firewall configurations will control access and traffic between the environments.</span></span>
2. <span data-ttu-id="82d4d-141">ID プロバイダーを統合する。</span><span class="sxs-lookup"><span data-stu-id="82d4d-141">Consolidate Identity Providers.</span></span> <span data-ttu-id="82d4d-142">セカンダリ クラウドでホストされているワークロードに応じて、ID プロバイダーの統合に対してさまざまな選択肢があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-142">Depending on the workloads being hosted in the secondary cloud, there are a variety of options to identity provider consolidation.</span></span> <span data-ttu-id="82d4d-143">以下に例を示します。</span><span class="sxs-lookup"><span data-stu-id="82d4d-143">The following are a few examples:</span></span>
    1. <span data-ttu-id="82d4d-144">OAuth 2 を使用して認証するアプリケーションの場合、セカンダリ クラウド内の Active Directory のユーザーは、単純に既存の Azure AD テナントにレプリケートできます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-144">For applications that authenticate using OAuth 2, users in the Active Directory in the secondary cloud could simply be replicated to the existing Azure AD tenant.</span></span>
    2. <span data-ttu-id="82d4d-145">一方、2 つのオンプレミス ID プロバイダー間のフェデレーションにより、新しい Active Directory ドメインのユーザーを Azure にレプリケートすることができます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-145">On the other extreme, federation between the two on-premises identity providers, would allow users from the new Active Directory domains to be replicated to Azure.</span></span>
3. <span data-ttu-id="82d4d-146">Azure Site Recovery に資産を追加する</span><span class="sxs-lookup"><span data-stu-id="82d4d-146">Add assets to Azure Site Recovery</span></span>
    1. <span data-ttu-id="82d4d-147">Azure Site Recovery は、最初からハイブリッド クラウドまたはマルチクラウドのツールとして構築されました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-147">Azure Site Recovery was built as a hybrid/multi-cloud tool from the beginning.</span></span>
    2. <span data-ttu-id="82d4d-148">セカンダリ クラウド内の仮想マシンは、オンプレミスの資産の保護に使用されているのと同じ Azure Site Recovery プロセスによって保護できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-148">Virtual machines in the secondary cloud might be able to be protected by the same Azure Site Recovery processes used to protect on-premises assets.</span></span>
4. <span data-ttu-id="82d4d-149">Azure Cost Management に資産を追加する</span><span class="sxs-lookup"><span data-stu-id="82d4d-149">Add assets to Azure Cost Management</span></span>
    1. <span data-ttu-id="82d4d-150">Azure Cost Management は、最初からマルチクラウド ツールとして構築されました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-150">Azure Cost Management was built as a multi-cloud tool from the beginning.</span></span>
    2. <span data-ttu-id="82d4d-151">セカンダリ クラウド内の仮想マシンは、一部のクラウド プロバイダーの Azure Cost Management と互換性がある場合があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-151">Virtual machines in the secondary cloud might be compatible with Azure Cost Management for some cloud providers.</span></span> <span data-ttu-id="82d4d-152">追加コストが適用される場合があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-152">Additional costs may apply.</span></span>
5. <span data-ttu-id="82d4d-153">Azure Monitor に資産を追加する</span><span class="sxs-lookup"><span data-stu-id="82d4d-153">Add assets to Azure Monitor</span></span>
    1. <span data-ttu-id="82d4d-154">Azure Monitor は、最初からハイブリッド クラウド ツールとして構築されました。</span><span class="sxs-lookup"><span data-stu-id="82d4d-154">Azure Monitor was built as a hybrid cloud tool from the beginning.</span></span>
    2. <span data-ttu-id="82d4d-155">セカンダリ クラウド内の仮想マシンは、Azure Monitor エージェントと互換性があり、運用監視のために Azure Monitor に含めることができる場合があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-155">Virtual machines in the secondary cloud might be compatible with Azure Monitor agents, allowing them to be included in Azure Monitor for operational monitoring.</span></span>
6. <span data-ttu-id="82d4d-156">ガバナンス実施ツール</span><span class="sxs-lookup"><span data-stu-id="82d4d-156">Governance enforcement tools</span></span>
    1. <span data-ttu-id="82d4d-157">ガバナンス実施はクラウド固有です。</span><span class="sxs-lookup"><span data-stu-id="82d4d-157">Governance enforcement is cloud-specific.</span></span>
    2. <span data-ttu-id="82d4d-158">ガバナンス体験で設定した会社のポリシーはそうではありません。</span><span class="sxs-lookup"><span data-stu-id="82d4d-158">The corporate policies established in the governance journey are not.</span></span> <span data-ttu-id="82d4d-159">実装はクラウドごとに異なる可能性がありますが、ポリシー ステートメントはセカンダリ プロバイダーに適用できます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-159">While the implementation may vary from cloud to cloud, the policy statements can be applied to the secondary provider.</span></span>

<span data-ttu-id="82d4d-160">マルチクラウドの導入が増えるにつれて、上記の設計の進化も成熟していきます。</span><span class="sxs-lookup"><span data-stu-id="82d4d-160">As multi-cloud adoption grows, the design evolution above will continue to mature.</span></span>

## <a name="next-steps"></a><span data-ttu-id="82d4d-161">次の手順</span><span class="sxs-lookup"><span data-stu-id="82d4d-161">Next steps</span></span>

<span data-ttu-id="82d4d-162">多くの大企業で、クラウド ガバナンスの 5 つの規範が導入の妨げになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="82d4d-162">In many large enterprises, the Five Dsciplines of Cloud Governance can be blockers to adoption.</span></span> <span data-ttu-id="82d4d-163">次の記事では、ガバナンスをチーム スポーツにして、クラウドでの長期的成功の確保に役立てるいくつかの追加的な考えをご紹介します。</span><span class="sxs-lookup"><span data-stu-id="82d4d-163">The next article has some additional thoughts on making governance a team sport to help ensure long-term success in the cloud.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="82d4d-164">複数レイヤーのガバナンス</span><span class="sxs-lookup"><span data-stu-id="82d4d-164">Multiple layers of governance</span></span>](./multiple-layers-of-governance.md)
