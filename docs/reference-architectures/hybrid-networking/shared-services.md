---
title: ハブスポーク ネットワーク トポロジを実装する
titleSuffix: Azure Reference Architectures
description: Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装します。
author: telmosampaio
ms.date: 10/09/2018
ms.custom: seodec18
ms.openlocfilehash: 37ae02d8ef02f64329d5e5215e5a32df9f0f9491
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120460"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="2ab0e-103">Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="2ab0e-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="2ab0e-104">この参照アーキテクチャは、[ハブスポーク][guidance-hub-spoke]参照アーキテクチャに基づいて作成されており、すべてのスポークで利用できる共有サービスがハブに含まれます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="2ab0e-105">クラウドへのデータセンターの移行と[仮想データセンター]の構築に向けた最初の手順として、共有する必要がある最初のサービスが ID とセキュリティです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="2ab0e-106">この参照アーキテクチャは、Active Directory サービスをオンプレミスのデータセンターから Azure に拡張する方法と、ハブスポーク トポロジに、ファイアウォールとして動作可能なネットワーク仮想アプライアンス (NVA) を追加する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="2ab0e-107">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Azure での共有サービス トポロジ](./images/shared-services.png)

<span data-ttu-id="2ab0e-109">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="2ab0e-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="2ab0e-110">このトポロジの利点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-110">The benefits of this topology include:</span></span>

- <span data-ttu-id="2ab0e-111">**コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="2ab0e-112">**サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="2ab0e-113">**問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="2ab0e-114">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-114">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="2ab0e-115">DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="2ab0e-116">共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="2ab0e-117">相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="2ab0e-118">セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="2ab0e-119">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="2ab0e-119">Architecture</span></span>

<span data-ttu-id="2ab0e-120">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-120">The architecture consists of the following components.</span></span>

- <span data-ttu-id="2ab0e-121">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-121">**On-premises network**.</span></span> <span data-ttu-id="2ab0e-122">組織内で実行されているプライベートなローカル エリア ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-122">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="2ab0e-123">**VPN デバイス**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-123">**VPN device**.</span></span> <span data-ttu-id="2ab0e-124">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="2ab0e-125">VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="2ab0e-126">サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="2ab0e-127">**VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="2ab0e-128">仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="2ab0e-129">詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="2ab0e-130">この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="2ab0e-131">**ハブ VNet**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-131">**Hub VNet**.</span></span> <span data-ttu-id="2ab0e-132">ハブスポーク トポロジのハブとして使用する Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="2ab0e-133">ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="2ab0e-134">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-134">**Gateway subnet**.</span></span> <span data-ttu-id="2ab0e-135">複数の仮想ネットワーク ゲートウェイが同じサブネットに保持されます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-135">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="2ab0e-136">**共有サービス サブネット**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-136">**Shared services subnet**.</span></span> <span data-ttu-id="2ab0e-137">DNS や AD DS など、すべてのスポーク間で共有できるサービスをホストするために使用されるハブ VNet 内のサブネットです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

- <span data-ttu-id="2ab0e-138">**DMZ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-138">**DMZ subnet**.</span></span> <span data-ttu-id="2ab0e-139">ファイアウォールなどの、セキュリティ アプライアンスとして動作できる NVA をホストするために使用されるハブ VNet のサブネットです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

- <span data-ttu-id="2ab0e-140">**スポーク VNet**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-140">**Spoke VNets**.</span></span> <span data-ttu-id="2ab0e-141">ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="2ab0e-142">スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="2ab0e-143">各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="2ab0e-144">アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="2ab0e-145">**VNet ピアリング**。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-145">**VNet peering**.</span></span> <span data-ttu-id="2ab0e-146">[ピアリング接続][vnet-peering]を使用して、同じ Azure リージョン内の 2 つの VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="2ab0e-147">ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="2ab0e-148">ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="2ab0e-149">ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="2ab0e-150">この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="2ab0e-151">これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="2ab0e-152">Recommendations</span><span class="sxs-lookup"><span data-stu-id="2ab0e-152">Recommendations</span></span>

<span data-ttu-id="2ab0e-153">[ハブスポーク][guidance-hub-spoke]参照アーキテクチャのすべての推奨事項が、共有サービスの参照アーキテクチャにも適用されます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span>

<span data-ttu-id="2ab0e-154">また、共有サービスのほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-154">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="2ab0e-155">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="2ab0e-156">ID</span><span class="sxs-lookup"><span data-stu-id="2ab0e-156">Identity</span></span>

<span data-ttu-id="2ab0e-157">ほとんどの企業組織のオンプレミス データセンターには Active Directory ディレクトリ サービス (ADDS) 環境があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="2ab0e-158">ADDS に依存しているオンプレミスのネットワークから Azure に移動される資産の管理を容易にするには、Azure で ADDS ドメイン コントローラーをホストすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="2ab0e-159">Azure とオンプレミスの環境を別々に管理するグループ ポリシー オブジェクトを使用する場合は、Azure リージョンごとに別の AD サイトを使用します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="2ab0e-160">依存するワークロードがアクセスできる中心の VNet (ハブ) に、ドメイン コントローラーを配置します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="2ab0e-161">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="2ab0e-161">Security</span></span>

<span data-ttu-id="2ab0e-162">オンプレミスの環境から Azure にワークロードを移動すると、これらのワークロードの一部を VM でホストする必要があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="2ab0e-163">コンプライアンス上の理由から、これらのワークロードを通過するトラフィックを制限することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span>

<span data-ttu-id="2ab0e-164">Azure では、ネットワーク仮想アプライアンス (NVA) を使用して、さまざまな種類のセキュリティおよびパフォーマンス サービスをホストできます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="2ab0e-165">現在、オンプレミスで特定のアプライアンスのセットを使い慣れている場合は、可能であれば、Azure で同じ仮想アプライアンスを使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="2ab0e-166">この参照アーキテクチャのデプロイ スクリプトでは、IP 転送が有効な Ubuntu VM を使用してネットワーク仮想アプライアンスを模倣します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="2ab0e-167">考慮事項</span><span class="sxs-lookup"><span data-stu-id="2ab0e-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="2ab0e-168">VNet ピアリングの制限の解消</span><span class="sxs-lookup"><span data-stu-id="2ab0e-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="2ab0e-169">Azure の [VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="2ab0e-170">制限を超える数のスポークを許可する場合は、ハブスポークハブスポーク トポロジの作成を検討してください。このトポロジでは、第 1 レベルのスポークもハブとして機能します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="2ab0e-171">この手法を次の図に示します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-171">The following diagram shows this approach.</span></span>

![Azure のハブスポークハブスポーク トポロジ](./images/hub-spokehub-spoke.svg)

<span data-ttu-id="2ab0e-173">また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="2ab0e-174">たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="2ab0e-175">このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="2ab0e-176">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="2ab0e-176">Deploy the solution</span></span>

<span data-ttu-id="2ab0e-177">このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="2ab0e-178">デプロイによってサブスクリプション内に作成されるリソース グループは、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-178">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="2ab0e-179">hub-adds-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-179">hub-adds-rg</span></span>
- <span data-ttu-id="2ab0e-180">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-180">hub-nva-rg</span></span>
- <span data-ttu-id="2ab0e-181">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-181">hub-vnet-rg</span></span>
- <span data-ttu-id="2ab0e-182">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-182">onprem-vnet-rg</span></span>
- <span data-ttu-id="2ab0e-183">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-183">spoke1-vnet-rg</span></span>
- <span data-ttu-id="2ab0e-184">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="2ab0e-184">spoke2-vent-rg</span></span>

<span data-ttu-id="2ab0e-185">テンプレート パラメーター ファイルは、これらの名前を参照します。したがって、名前を変更する場合は、それに合わせてパラメーター ファイルも更新します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-185">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2ab0e-186">前提条件</span><span class="sxs-lookup"><span data-stu-id="2ab0e-186">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="2ab0e-187">シミュレートされたオンプレミスのデータセンターを azbb を使用してデプロイする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="2ab0e-188">この手順では、シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-188">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="2ab0e-189">GitHub リポジトリの `hybrid-networking\shared-services-stack\` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-189">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="2ab0e-190">`onprem.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-190">Open the `onprem.json` file.</span></span>

3. <span data-ttu-id="2ab0e-191">`UserName`、`adminUserName`、`Password`、および `adminPassword` のすべてのインスタンスを検索します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-191">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="2ab0e-192">パラメーターにユーザー名とパスワードの値を入力し、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-192">Enter values for the user name and password in the parameters and save the file.</span></span>

4. <span data-ttu-id="2ab0e-193">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-193">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```

5. <span data-ttu-id="2ab0e-194">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-194">Wait for the deployment to finish.</span></span> <span data-ttu-id="2ab0e-195">このデプロイでは、仮想ネットワーク、Windows を実行する仮想マシン、および VPN ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-195">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="2ab0e-196">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-196">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="2ab0e-197">ハブ VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-197">Deploy the hub VNet</span></span>

<span data-ttu-id="2ab0e-198">この手順では、ハブ VNet をデプロイして、シミュレートされたオンプレミス VNet に接続します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-198">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="2ab0e-199">`hub-vnet.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-199">Open the `hub-vnet.json` file.</span></span>

2. <span data-ttu-id="2ab0e-200">`adminPassword` を検索し、パラメーターにユーザー名とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-200">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="2ab0e-201">`sharedKey` のすべてのインスタンスを検索し、共有キーの値を入力します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-201">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="2ab0e-202">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-202">Save the file.</span></span>

   ```json
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="2ab0e-203">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-203">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="2ab0e-204">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-204">Wait for the deployment to finish.</span></span> <span data-ttu-id="2ab0e-205">このデプロイでは、仮想ネットワーク、仮想マシン、VPN ゲートウェイ、および前のセクションで作成したゲートウェイへの接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-205">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="2ab0e-206">VPN ゲートウェイの完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-206">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="2ab0e-207">Azure に AD DS をデプロイする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-207">Deploy AD DS in Azure</span></span>

<span data-ttu-id="2ab0e-208">この手順では、Azure に AD DS ドメイン コントローラーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-208">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="2ab0e-209">`hub-adds.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-209">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="2ab0e-210">`Password` および `adminPassword` のすべてのインスタンスを検索します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-210">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="2ab0e-211">パラメーターにユーザー名とパスワードの値を入力し、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-211">Enter values for the user name and password in the parameters and save the file.</span></span>

3. <span data-ttu-id="2ab0e-212">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-212">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```

<span data-ttu-id="2ab0e-213">シミュレートされたオンプレミスのデータセンターでホストされているドメインに 2 つの VM を参加させて、その VM に AD DS をインストールするため、このデプロイ手順は数分かかる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-213">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="2ab0e-214">スポーク VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-214">Deploy the spoke VNets</span></span>

<span data-ttu-id="2ab0e-215">この手順では、スポーク VNet をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-215">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="2ab0e-216">`spoke1.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-216">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="2ab0e-217">`adminPassword` を検索し、パラメーターにユーザー名とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-217">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="2ab0e-218">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-218">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="2ab0e-219">`spoke2.json` ファイルに対して手順 1. から 2. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-219">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="2ab0e-220">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-220">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="2ab0e-221">ハブ VNet をスポーク VNet にピアリングする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-221">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="2ab0e-222">ハブ VNet からスポーク VNet へのピアリング接続を作成するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-222">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="2ab0e-223">NVA をデプロイする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-223">Deploy the NVA</span></span>

<span data-ttu-id="2ab0e-224">この手順では、NVA を `dmz` サブネットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-224">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="2ab0e-225">`hub-nva.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-225">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="2ab0e-226">`adminPassword` を検索し、パラメーターにユーザー名とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-226">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="2ab0e-227">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-227">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="2ab0e-228">接続をテストする</span><span class="sxs-lookup"><span data-stu-id="2ab0e-228">Test connectivity</span></span>

<span data-ttu-id="2ab0e-229">シミュレートされたオンプレミスの環境からハブ VNet への接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-229">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="2ab0e-230">Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-230">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="2ab0e-231">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-231">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="2ab0e-232">`onprem.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-232">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="2ab0e-233">VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、ハブ VNet の ジャンプボックス VM に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-233">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="2ab0e-234">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-234">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="2ab0e-235">既定で、Windows Server VM では Azure の ICMP 応答が許可されていません。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-235">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="2ab0e-236">接続のテストに `ping` を使用する場合は、VM ごとに Windows の高度なファイアウォールで ICMP トラフィックを有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-236">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="2ab0e-237">スポーク VNet への接続をテストするには、同じ手順を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="2ab0e-237">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[仮想データセンター]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
