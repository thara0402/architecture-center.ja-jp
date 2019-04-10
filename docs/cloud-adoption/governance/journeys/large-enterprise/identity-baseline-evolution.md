---
title: 'CAF: 大企業 – ID ベースラインの進化'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 大企業 – ID ベースラインの進化
author: BrianBlanchard
ms.openlocfilehash: 7eff9d8e8fd0fd40afa1d9815941fcce8d447579
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242583"
---
# <a name="large-enterprise-identity-baseline-evolution"></a><span data-ttu-id="b8f70-103">大企業:ID ベースラインの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-103">Large enterprise: Identity Baseline evolution</span></span>

<span data-ttu-id="b8f70-104">この記事では、ガバナンス MVP に ID ベースライン制御を追加することで物語を進化させます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-104">This article evolves the narrative by adding Identity Baseline controls to the governance MVP.</span></span>

## <a name="evolution-of-the-narrative"></a><span data-ttu-id="b8f70-105">物語の進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-105">Evolution of the narrative</span></span>

<span data-ttu-id="b8f70-106">2 つのデータセンターのクラウドへの移行に関する業務上の正当な理由は、CFO によって承認されました。</span><span class="sxs-lookup"><span data-stu-id="b8f70-106">The business justification for the cloud migration of the two datacenters was approved by the CFO.</span></span> <span data-ttu-id="b8f70-107">技術的実現可能性の調査時に、いくつかの障害が検出されました。</span><span class="sxs-lookup"><span data-stu-id="b8f70-107">During the technical feasibility study, several roadblocks were discovered:</span></span>

- <span data-ttu-id="b8f70-108">保護されたデータおよびミッション クリティカルなアプリケーションについては、2 つのデータセンターのワークロードが 25% と示されています。</span><span class="sxs-lookup"><span data-stu-id="b8f70-108">Protected data and mission-critical applications represent 25% of the workloads in the two datacenters.</span></span> <span data-ttu-id="b8f70-109">PII およびミッション クリティカルなアプリケーションに関する現在のガバナンス ポリシーが最新化されるまで、いずれも除去できません。</span><span class="sxs-lookup"><span data-stu-id="b8f70-109">Neither can be eliminated until the current governance policies regarding PII and mission-critical applications have been modernized.</span></span>
- <span data-ttu-id="b8f70-110">これらのデータセンター内の資産の 7% はクラウドと互換性はありません。</span><span class="sxs-lookup"><span data-stu-id="b8f70-110">7% of the assets in those datacenters are not cloud-compatible.</span></span> <span data-ttu-id="b8f70-111">これらは、データセンターの契約が終了する前に、代替データセンターに移動されます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-111">They will be moved to an alternate datacenter before termination of the datacenter contract.</span></span>
- <span data-ttu-id="b8f70-112">データセンター内の資産の 15% (750 台の仮想マシン) は、レガシ認証またはサード パーティの多要素認証に依存しています。</span><span class="sxs-lookup"><span data-stu-id="b8f70-112">15% of the assets in the datacenter (750 virtual machines) have a dependency on legacy authentication or third-party multi-factor authentication.</span></span>
- <span data-ttu-id="b8f70-113">既存のデータセンターと Azure を接続する VPN 接続では、2 年のタイムライン内で大量の資産を移行し、データセンターを廃止するために十分なデータ転送速度や待機時間は提供されません。</span><span class="sxs-lookup"><span data-stu-id="b8f70-113">The VPN connection that connects existing datacenters and Azure does not offer sufficient data transmission speeds or latency to migrate the volume of assets within the two-year timeline to retire the datacenter.</span></span>

<span data-ttu-id="b8f70-114">最初の 2 つの障害は並列に軽減されます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-114">The first two roadblocks are being mitigated in parallel.</span></span> <span data-ttu-id="b8f70-115">この記事では、3 番目と 4 番目の障害の解決策について説明します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-115">This article will address the resolution of the third and fourth roadblocks.</span></span>

### <a name="evolution-of-the-cloud-governance-team"></a><span data-ttu-id="b8f70-116">クラウド管理チームの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-116">Evolution of the Cloud Governance team</span></span>

<span data-ttu-id="b8f70-117">クラウド管理チームは拡大しています。</span><span class="sxs-lookup"><span data-stu-id="b8f70-117">The Cloud Governance team is expanding.</span></span> <span data-ttu-id="b8f70-118">ID 管理に関する追加サポートの必要性に応じて、既存のチーム メンバーが常に変化を認識するように、ID ベースライン チームのシステム管理者は現在、週 1 回の会議に出席しています。</span><span class="sxs-lookup"><span data-stu-id="b8f70-118">Given the need for additional support regarding identity management, a systems administrator from the Identity Baseline team now participates in a weekly meeting to keep the existing team members aware of changes.</span></span>

### <a name="evolution-of-the-current-state"></a><span data-ttu-id="b8f70-119">現在の状態の進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-119">Evolution of the current state</span></span>

<span data-ttu-id="b8f70-120">IT チームは、2 つのデータセンターを廃止する CIO と CFO の計画を進めるための承認を得ました。</span><span class="sxs-lookup"><span data-stu-id="b8f70-120">The IT team has approval to move forward with the CIO and CFO's plans to retire two datacenters.</span></span> <span data-ttu-id="b8f70-121">しかし、IT チームは、これらのデータセンター内の 750 台 (15%) の資産をクラウド以外の場所に移動する必要があることを懸念しています。</span><span class="sxs-lookup"><span data-stu-id="b8f70-121">However, IT is concerned that 750 (15%) of the assets in those datacenters will have to be moved somewhere other than the cloud.</span></span>

### <a name="evolution-of-the-future-state"></a><span data-ttu-id="b8f70-122">将来の状態の進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-122">Evolution of the future state</span></span>

<span data-ttu-id="b8f70-123">新たな将来の状態の計画には、レガシ認証要件がある 750 台の仮想マシンを移行するためのより堅牢な ID ベースライン ソリューションが必要です。</span><span class="sxs-lookup"><span data-stu-id="b8f70-123">The new future state plans require a more robust Identity Baseline solution to migrate the 750 virtual machines with legacy authentication requirements.</span></span> <span data-ttu-id="b8f70-124">これら 2 つのデータセンター以外にも同様の割合の資産があり、それらはこの課題による影響を受けることが予想されます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-124">Beyond these two datacenters, similar percentages of assets in other datacenters are expected to be affected by this challenge.</span></span>
<span data-ttu-id="b8f70-125">また、将来の状態では現在、クラウド プロバイダーから会社の MPLS/専用回線ソリューションへの接続が必要です。</span><span class="sxs-lookup"><span data-stu-id="b8f70-125">The future state now also requires a connection from the cloud provider to the company’s MPLS/leased-line solution.</span></span>

<span data-ttu-id="b8f70-126">現在および将来の状態が変わると、新しいポリシー ステートメントを必要とする新たなリスクが生じます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-126">The changes to current and future state expose new risks that will require new policy statements.</span></span>

## <a name="evolution-of-tangible-risks"></a><span data-ttu-id="b8f70-127">具体的なリスクの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-127">Evolution of tangible risks</span></span>

<span data-ttu-id="b8f70-128">**移行時の業務中断**。</span><span class="sxs-lookup"><span data-stu-id="b8f70-128">**Business interruption during migration**.</span></span> <span data-ttu-id="b8f70-129">クラウドへの移行では、管理可能な制御対象の期限付きリスクが生じます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-129">Migration to the cloud creates a controlled, time-bound risk that can be managed.</span></span> <span data-ttu-id="b8f70-130">古いハードウェアを世界の別の地域に移動することは、かなり高いリスクとなります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-130">Moving aging hardware to another part of the world is much higher risk.</span></span> <span data-ttu-id="b8f70-131">業務操作の中断を回避するには、軽減戦略が必要です。</span><span class="sxs-lookup"><span data-stu-id="b8f70-131">A mitigation strategy is needed to avoid interruptions to business operations.</span></span>

<span data-ttu-id="b8f70-132">**既存の ID の依存関係**。</span><span class="sxs-lookup"><span data-stu-id="b8f70-132">**Existing identity dependencies**.</span></span> <span data-ttu-id="b8f70-133">既存の認証と ID サービスの依存関係は、クラウドへの一部のワークロードの移行の遅延や妨げになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-133">Dependencies on existing authentication and identity services may delay or prevent the migration of some workloads to the cloud.</span></span> <span data-ttu-id="b8f70-134">2 つのデータセンターを予定どおりに戻せない場合、データセンターの数百万ドルのリース料金が発生することになります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-134">Failure to return the two datacenters on time will incur millions of dollars in datacenter lease fees.</span></span>

<span data-ttu-id="b8f70-135">このビジネス リスクは、いくつかの技術的リスクへと拡大する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-135">This business risk can be expanded into a few technical risks:</span></span>

- <span data-ttu-id="b8f70-136">レガシ認証をクラウドで利用できなくなる可能性があり、これにより、一部のアプリケーションのデプロイが制限されます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-136">Legacy authentication might not be available in the cloud, limiting deployment of some applications.</span></span>
- <span data-ttu-id="b8f70-137">現在のサード パーティの MFA ソリューションをクラウドで利用できなくなる可能性があり、これにより、一部のアプリケーションのデプロイが制限されます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-137">The current third-party MFA solution might not be available in the cloud, limiting deployment of some applications.</span></span>
- <span data-ttu-id="b8f70-138">ツールの一新や移動により、サービスが停止し、コストが増える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-138">Retooling or moving either could create outages and add costs.</span></span>
- <span data-ttu-id="b8f70-139">VPN の速度と安定性が移行の妨げになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-139">The speed and stability of the VPN might impede migration.</span></span>
- <span data-ttu-id="b8f70-140">クラウドへのトラフィックにより、グローバル ネットワークの他の部分でセキュリティの問題が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-140">Traffic coming into the cloud could cause security issues in other parts of the global network.</span></span>

## <a name="evolution-of-the-policy-statements"></a><span data-ttu-id="b8f70-141">ポリシー ステートメントの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-141">Evolution of the policy statements</span></span>

<span data-ttu-id="b8f70-142">ポリシーに対する次の変更は、新しいリスクを軽減して実装を導くのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-142">The following changes to policy will help mitigate the new risks and guide implementation.</span></span>

1. <span data-ttu-id="b8f70-143">選択されたクラウド プロバイダーが、従来の方法による認証手段を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-143">The chosen cloud provider must offer a means of authenticating via legacy methods.</span></span>
2. <span data-ttu-id="b8f70-144">選択されたクラウド プロバイダーが、現在のサード パーティの MFA ソリューションによる認証手段を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-144">The chosen cloud provider must offer a means of authentication with the current third-party MFA solution.</span></span>
3. <span data-ttu-id="b8f70-145">クラウド プロバイダーをデータセンターのグローバル ネットワークに接続し、クラウド プロバイダーと会社の電気通信プロバイダー間で高速プライベート接続を確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-145">A high-speed private connection should be established between the cloud provider and the company’s telco provider, connecting the cloud provider to the global network of datacenters.</span></span>
4. <span data-ttu-id="b8f70-146">十分なセキュリティ要件が確立されるまで、インバウンド パブリック トラフィックでは、クラウド内でホストされている会社の資産にアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="b8f70-146">Until sufficient security requirements are established, no inbound public traffic may access company assets hosted in the cloud.</span></span> <span data-ttu-id="b8f70-147">グローバル WAN 外のソースからすべてのポートがブロックされます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-147">All ports are blocked from any source outside of the global WAN.</span></span>

## <a name="evolution-of-the-best-practices"></a><span data-ttu-id="b8f70-148">ベスト プラクティスの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-148">Evolution of the best practices</span></span>

<span data-ttu-id="b8f70-149">ガバナンス MVP 設定は、新しい Azure ポリシーと、仮想マシンでの Active Directory の実装を含めるために進化します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-149">The governance MVP design evolves to include new Azure policies and an implementation of Active Directory on a virtual machine.</span></span> <span data-ttu-id="b8f70-150">これら 2 つの設計変更を組み合わせることで、新しい企業ポリシー ステートメントを満たします。</span><span class="sxs-lookup"><span data-stu-id="b8f70-150">Together, these two design changes fulfill the new corporate policy statements.</span></span>

<span data-ttu-id="b8f70-151">新しいベスト プラクティスを以下に示します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-151">Here are the new best practices:</span></span>

1. <span data-ttu-id="b8f70-152">DMZ のブループリント:DMZ のオンプレミス側を、次のソリューションとオンプレミスの Active Directory サーバー間の通信を許可するように構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-152">DMZ blueprint: The on-premises side of the DMZ should be configured to allow communication between the following solution and the on-premises Active Directory servers.</span></span> <span data-ttu-id="b8f70-153">このベスト プラクティスでは、DMZ でネットワーク境界を越えて Active Directory Domain Services を有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-153">This best practice requires a DMZ to enable Active Directory Domain Services across network boundaries.</span></span>
2. <span data-ttu-id="b8f70-154">Azure Resource Manager テンプレート:</span><span class="sxs-lookup"><span data-stu-id="b8f70-154">Azure Resource Manager templates:</span></span>
    1. <span data-ttu-id="b8f70-155">外部トラフィックをブロックし、内部トラフィックをホワイトリストに登録するように NSG を定義します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-155">Define an NSG to block external traffic and whitelist internal traffic.</span></span>
    1. <span data-ttu-id="b8f70-156">ゴールデン イメージに基づき、負荷分散ペアの 2 つの AD 仮想マシンをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="b8f70-156">Deploy two AD virtual machines in a load balanced pair based on a golden image.</span></span> <span data-ttu-id="b8f70-157">最初の起動時に、そのイメージで PowerShell スクリプトが実行され、ドメインへの参加とドメイン サービスへの登録が行われます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-157">On first boot, that image runs a PowerShell script to join the domain and register with domain services.</span></span> <span data-ttu-id="b8f70-158">詳細については、「[Extend Active Directory Domain Services (AD DS) to Azure](../../../../reference-architectures/identity/adds-extend-domain.md)」 (Azure への Active Directory Domain Services (AD DS) の拡張) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b8f70-158">For more information, see [Extend Active Directory Domain Services (AD DS) to Azure](../../../../reference-architectures/identity/adds-extend-domain.md).</span></span>
3. <span data-ttu-id="b8f70-159">Azure Policy:すべてのリソースに NSG を適用します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-159">Azure Policy: Apply the NSG to all resources.</span></span>
4. <span data-ttu-id="b8f70-160">Azure のブループリント</span><span class="sxs-lookup"><span data-stu-id="b8f70-160">Azure blueprint</span></span>
    1. <span data-ttu-id="b8f70-161">`active-directory-virtual-machines` という名前のブループリントを作成します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-161">Create a blueprint named `active-directory-virtual-machines`.</span></span>
    1. <span data-ttu-id="b8f70-162">ブループリントに、AD テンプレートおよびポリシーをそれぞれ追加します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-162">Add each of the AD templates and policies to the blueprint.</span></span>
    1. <span data-ttu-id="b8f70-163">適用可能な管理グループにブループリントを公開します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-163">Publish the blueprint to any applicable management group.</span></span>
    1. <span data-ttu-id="b8f70-164">ブループリントを、レガシまたはサード パーティの MFA 認証を必要とするサブスクリプションに適用します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-164">Apply the blueprint to any subscription requiring legacy or third-party MFA authentication.</span></span>
    1. <span data-ttu-id="b8f70-165">これで、Azure で実行されている AD のインスタンスを、オンプレミス AD ソリューションの拡張機能として使用でき、それを既存の MFA ツールと統合し、クレームベース認証を提供することができます。既存の Active Directory 機能を使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="b8f70-165">The instance of AD running in Azure can now be used as an extension of the on-premises AD solution, allowing it to integrate with the existing MFA tool and provide claims-based authentication, both through existing Active Directory functionality.</span></span>

## <a name="conclusion"></a><span data-ttu-id="b8f70-166">まとめ</span><span class="sxs-lookup"><span data-stu-id="b8f70-166">Conclusion</span></span>

<span data-ttu-id="b8f70-167">ガバナンス MVP へのこれらの変更の追加は、この記事に示した多くのリスクを軽減するのに役立ち、各クラウド導入チームはこの障害に迅速に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="b8f70-167">Adding these changes to the governance MVP helps mitigate many of the risks in this article, allowing each cloud adoption team to quickly move past this roadblock.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b8f70-168">次の手順</span><span class="sxs-lookup"><span data-stu-id="b8f70-168">Next steps</span></span>

<span data-ttu-id="b8f70-169">クラウド導入が進化し、ビジネス価値が高まる一方で、リスクやクラウド ガバナンスのニーズも高度化します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-169">As cloud adoption evolves and delivers additional business value, risks and cloud governance needs will also evolve.</span></span> <span data-ttu-id="b8f70-170">発生する可能性のあるいくつかの進化を以下に示します。</span><span class="sxs-lookup"><span data-stu-id="b8f70-170">The following are a few evolutions that may occur.</span></span> <span data-ttu-id="b8f70-171">この体験の架空の会社では、次のトリガーは、クラウド導入計画に保護されたデータを含めることです。</span><span class="sxs-lookup"><span data-stu-id="b8f70-171">For the fictional company in this journey, the next trigger is the inclusion of protected data in the cloud adoption plan.</span></span> <span data-ttu-id="b8f70-172">この変更には、追加のセキュリティ制御が必要です。</span><span class="sxs-lookup"><span data-stu-id="b8f70-172">This change will require additional security controls.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="b8f70-173">セキュリティ ベースラインの進化</span><span class="sxs-lookup"><span data-stu-id="b8f70-173">Security Baseline evolution</span></span>](./security-baseline-evolution.md)
