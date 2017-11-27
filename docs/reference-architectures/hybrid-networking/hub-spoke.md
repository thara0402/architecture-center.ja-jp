---
title: "Azure にハブスポーク ネットワーク トポロジを実装する"
description: "Azure にハブスポーク ネットワーク トポロジを実装する方法。"
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="24f84-103">Azure にハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="24f84-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="24f84-104">この参照アーキテクチャでは、Azure にハブスポーク トポロジを実装する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="24f84-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="24f84-105">*ハブ*は、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="24f84-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="24f84-106">*スポーク*は、ハブとピアリングする VNet であり、ワークロードを分離するために使用できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="24f84-107">トラフィックは、ExpressRoute または VPN ゲートウェイ接続を経由してオンプレミスのデータセンターとハブの間を流れます。</span><span class="sxs-lookup"><span data-stu-id="24f84-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="24f84-108">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="24f84-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="24f84-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="24f84-109">![[0]][0]</span></span>

<span data-ttu-id="24f84-110">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="24f84-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="24f84-111">このトポロジの利点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="24f84-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="24f84-112">**コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。</span><span class="sxs-lookup"><span data-stu-id="24f84-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="24f84-113">**サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。</span><span class="sxs-lookup"><span data-stu-id="24f84-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="24f84-114">**問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。</span><span class="sxs-lookup"><span data-stu-id="24f84-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="24f84-115">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="24f84-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="24f84-116">DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。</span><span class="sxs-lookup"><span data-stu-id="24f84-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="24f84-117">共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="24f84-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="24f84-118">相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。</span><span class="sxs-lookup"><span data-stu-id="24f84-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="24f84-119">セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。</span><span class="sxs-lookup"><span data-stu-id="24f84-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="24f84-120">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="24f84-120">Architecture</span></span>

<span data-ttu-id="24f84-121">アーキテクチャは、次のコンポーネントで構成されています。</span><span class="sxs-lookup"><span data-stu-id="24f84-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="24f84-122">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="24f84-122">**On-premises network**.</span></span> <span data-ttu-id="24f84-123">組織内で実行されているプライベートなローカル エリア ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="24f84-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="24f84-124">**VPN デバイス**。</span><span class="sxs-lookup"><span data-stu-id="24f84-124">**VPN device**.</span></span> <span data-ttu-id="24f84-125">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。</span><span class="sxs-lookup"><span data-stu-id="24f84-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="24f84-126">VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。</span><span class="sxs-lookup"><span data-stu-id="24f84-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="24f84-127">サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="24f84-128">**VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="24f84-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="24f84-129">仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="24f84-130">詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="24f84-131">この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="24f84-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="24f84-132">**ハブ VNet**。</span><span class="sxs-lookup"><span data-stu-id="24f84-132">**Hub VNet**.</span></span> <span data-ttu-id="24f84-133">ハブスポーク トポロジのハブとして使用する Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="24f84-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="24f84-134">ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。</span><span class="sxs-lookup"><span data-stu-id="24f84-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="24f84-135">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="24f84-135">**Gateway subnet**.</span></span> <span data-ttu-id="24f84-136">複数の仮想ネットワーク ゲートウェイが同じサブネットに保持されます。</span><span class="sxs-lookup"><span data-stu-id="24f84-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="24f84-137">**共有サービス サブネット**。</span><span class="sxs-lookup"><span data-stu-id="24f84-137">**Shared services subnet**.</span></span> <span data-ttu-id="24f84-138">DNS や AD DS など、すべてのスポーク間で共有できるサービスをホストするために使用されるハブ VNet 内のサブネットです。</span><span class="sxs-lookup"><span data-stu-id="24f84-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="24f84-139">**スポーク VNet**。</span><span class="sxs-lookup"><span data-stu-id="24f84-139">**Spoke VNets**.</span></span> <span data-ttu-id="24f84-140">ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。</span><span class="sxs-lookup"><span data-stu-id="24f84-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="24f84-141">スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="24f84-142">各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。</span><span class="sxs-lookup"><span data-stu-id="24f84-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="24f84-143">アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="24f84-144">**VNet ピアリング**。</span><span class="sxs-lookup"><span data-stu-id="24f84-144">**VNet peering**.</span></span> <span data-ttu-id="24f84-145">[ピアリング接続][vnet-peering]を使用して、同じ Azure リージョン内の 2 つの VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="24f84-146">ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。</span><span class="sxs-lookup"><span data-stu-id="24f84-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="24f84-147">ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。</span><span class="sxs-lookup"><span data-stu-id="24f84-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="24f84-148">ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。</span><span class="sxs-lookup"><span data-stu-id="24f84-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="24f84-149">この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="24f84-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="24f84-150">これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。</span><span class="sxs-lookup"><span data-stu-id="24f84-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="24f84-151">Recommendations</span><span class="sxs-lookup"><span data-stu-id="24f84-151">Recommendations</span></span>

<span data-ttu-id="24f84-152">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="24f84-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="24f84-153">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="24f84-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="24f84-154">リソース グループ</span><span class="sxs-lookup"><span data-stu-id="24f84-154">Resource groups</span></span>

<span data-ttu-id="24f84-155">ハブ VNet と各スポーク VNet を異なるリソース グループに実装できます。同じ Azure リージョン内の同じ Azure Active Directory (Azure AD) テナントに属している限り、サブスクリプションが異なっていてもかまいません。</span><span class="sxs-lookup"><span data-stu-id="24f84-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="24f84-156">これにより、各ワークロードを分散管理し、サービスの共有はハブ VNet で維持できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="24f84-157">VNet と GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="24f84-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="24f84-158">*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。</span><span class="sxs-lookup"><span data-stu-id="24f84-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="24f84-159">このサブネットは、仮想ネットワーク ゲートウェイで必要です。</span><span class="sxs-lookup"><span data-stu-id="24f84-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="24f84-160">このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="24f84-161">ゲートウェイの設定について詳しくは、接続の種類に応じて、次の参照アーキテクチャをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="24f84-162">[ExpressRoute を使用するハイブリッド ネットワーク][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="24f84-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="24f84-163">[VPN ゲートウェイを使用するハイブリッド ネットワーク][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="24f84-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="24f84-164">高可用性を確保するために、ExpressRoute に加えてフェールオーバー用の VPN を使用できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="24f84-165">「[Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha]」(ExpressRoute と VPN フェールオーバーを使用してオンプレミス ネットワークを Azure に接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="24f84-166">オンプレミス ネットワークに接続する必要がない場合は、ゲートウェイを設定せずにハブスポーク トポロジを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="24f84-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="24f84-167">VNET ピアリング</span><span class="sxs-lookup"><span data-stu-id="24f84-167">VNet peering</span></span>

<span data-ttu-id="24f84-168">VNet ピアリングは、2 つの VNet 間の非推移的な関係です。</span><span class="sxs-lookup"><span data-stu-id="24f84-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="24f84-169">相互接続のためにスポークが必要な場合は、これらのスポーク間に個別のピアリング接続を追加することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="24f84-170">ただし、相互接続に必要なスポークが複数ある場合は、[VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]のために、有効なピアリング接続をすぐに使い果たします。</span><span class="sxs-lookup"><span data-stu-id="24f84-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="24f84-171">このシナリオでは、ユーザー定義ルート (UDR) を使用して、スポーク宛てのトラフィックをハブ VNet のルーターとして機能する NVA に強制的に送信することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="24f84-172">これにより、スポークを相互に接続できます。</span><span class="sxs-lookup"><span data-stu-id="24f84-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="24f84-173">リモート ネットワークと通信するハブ VNet ゲートウェイを使用するようにスポークを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="24f84-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="24f84-174">ゲートウェイ トラフィックをスポークからハブに流して、リモート ネットワークに接続するには、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="24f84-175">**ゲートウェイ トランジットを許可する**ようにハブで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="24f84-176">**リモート ゲートウェイを使用する**ように各スポークで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="24f84-177">**転送されたトラフィックを許可する**ようにすべての VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="24f84-178">考慮事項</span><span class="sxs-lookup"><span data-stu-id="24f84-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="24f84-179">スポーク接続</span><span class="sxs-lookup"><span data-stu-id="24f84-179">Spoke connectivity</span></span>

<span data-ttu-id="24f84-180">スポーク間の接続が必要な場合は、ハブでのルーティング用の NVA の実装、およびスポークでの UDR を使用したハブへのトラフィックの転送を検討してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="24f84-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="24f84-181">![[2]][2]</span></span>

<span data-ttu-id="24f84-182">このシナリオでは、**転送されたトラフィックを許可する**ようにピアリング接続を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="24f84-183">VNet ピアリングの制限の解消</span><span class="sxs-lookup"><span data-stu-id="24f84-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="24f84-184">Azure の [VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="24f84-185">制限を超える数のスポークを許可する場合は、ハブスポークハブスポーク トポロジの作成を検討してください。このトポロジでは、第 1 レベルのスポークもハブとして機能します。</span><span class="sxs-lookup"><span data-stu-id="24f84-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="24f84-186">この手法を次の図に示します。</span><span class="sxs-lookup"><span data-stu-id="24f84-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="24f84-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="24f84-187">![[3]][3]</span></span>

<span data-ttu-id="24f84-188">また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="24f84-189">たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。</span><span class="sxs-lookup"><span data-stu-id="24f84-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="24f84-190">このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="24f84-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="24f84-191">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="24f84-191">Deploy the solution</span></span>

<span data-ttu-id="24f84-192">このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24f84-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="24f84-193">このデプロイでは、各 VNet の Ubuntu VM を使用して接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="24f84-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="24f84-194">実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。</span><span class="sxs-lookup"><span data-stu-id="24f84-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="24f84-195">前提条件</span><span class="sxs-lookup"><span data-stu-id="24f84-195">Prerequisites</span></span>

<span data-ttu-id="24f84-196">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="24f84-197">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="24f84-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="24f84-198">Azure CLI を使用する方がよい場合は、Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="24f84-199">CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="24f84-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="24f84-200">PowerShell を使用する方がよい場合は、Azure 用の最新の PowerShell モジュールがコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="24f84-201">最新の Azure PowerShell モジュールをインストールするには、「[Install PowerShell for Azure][azure-powershell]」(Azure 用の PowerShell をインストールする) の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="24f84-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="24f84-202">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="24f84-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="24f84-203">シミュレートされたオンプレミスのデータセンターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="24f84-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="24f84-204">シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="24f84-205">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\onprem` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="24f84-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="24f84-206">`onprem.vm.parameters.json` ファイルを開き、次に示す 11 行目と 12 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="24f84-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="24f84-207">次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="24f84-208">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="24f84-209">別のリソース グループ名 (`ra-onprem-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="24f84-210">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="24f84-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="24f84-211">このデプロイでは、仮想ネットワーク、Ubuntu を実行する仮想マシン、および VPN ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="24f84-212">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="24f84-213">Azure のハブ VNet</span><span class="sxs-lookup"><span data-stu-id="24f84-213">Azure hub VNet</span></span>

<span data-ttu-id="24f84-214">ハブ VNet をデプロイして、上記の手順で作成したシミュレートされたオンプレミスの VNet に接続するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="24f84-215">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\hub` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="24f84-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="24f84-216">`hub.vm.parameters.json` ファイルを開き、次に示す 11 行目と 12 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="24f84-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="24f84-217">`hub.gateway.parameters.json` ファイルを開き、次に示す 23 行目の引用符の間に共有キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="24f84-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="24f84-218">この値をメモしておいてください。後からデプロイで使用します。</span><span class="sxs-lookup"><span data-stu-id="24f84-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="24f84-219">次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="24f84-220">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="24f84-221">別のリソース グループ名 (`ra-hub-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="24f84-222">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="24f84-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="24f84-223">このデプロイでは、仮想ネットワーク、Ubuntu を実行する仮想マシン、VPN ゲートウェイ、および前のセクションで作成したゲートウェイへの接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="24f84-224">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="24f84-225">オンプレミスからハブへの接続</span><span class="sxs-lookup"><span data-stu-id="24f84-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="24f84-226">シミュレートされたオンプレミスのデータセンターからハブ VNet に接続するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="24f84-227">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\onprem` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="24f84-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="24f84-228">`onprem.connection.parameters.json` ファイルを開き、次に示す 9 行目の引用符の間に共有キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="24f84-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="24f84-229">この共有キーの値は、以前にデプロイしたオンプレミスのゲートウェイで使用されるものと同じにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="24f84-230">次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="24f84-231">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="24f84-232">別のリソース グループ名 (`ra-onprem-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="24f84-233">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="24f84-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="24f84-234">このデプロイでは、オンプレミスのデータセンターのシミュレートに使用する VNet とハブ VNet 間の接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="24f84-235">Azure のスポーク VNet</span><span class="sxs-lookup"><span data-stu-id="24f84-235">Azure spoke VNets</span></span>

<span data-ttu-id="24f84-236">スポーク VNet をデプロイして、上記の手順で作成したハブ VNet に接続するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="24f84-237">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\spokes` フォルダーに切り替えます。</span><span class="sxs-lookup"><span data-stu-id="24f84-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="24f84-238">`spoke1.web.parameters.json` ファイルを開き、次に示す 53 行目と 54 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="24f84-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="24f84-239">`spoke2.web.parameters.json` ファイルについて前の手順を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="24f84-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="24f84-240">次の bash または PowerShell コマンドを実行して、最初のスポークをデプロイしてハブに接続します。</span><span class="sxs-lookup"><span data-stu-id="24f84-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="24f84-241">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="24f84-242">別のリソース グループ名 (`ra-spoke1-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="24f84-243">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="24f84-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="24f84-244">このデプロイでは、仮想ネットワーク、Ubuntu と Apache を実行する 3 つの仮想マシンを含むロード バランサー、VPN ゲートウェイ、および前のセクションで作成したハブ VNet への VNet ピアリング接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="24f84-245">このデプロイには 20 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="24f84-246">次の bash または PowerShell コマンドを実行して、最初のスポークをデプロイしてハブに接続します。</span><span class="sxs-lookup"><span data-stu-id="24f84-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="24f84-247">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="24f84-248">別のリソース グループ名 (`ra-spoke2-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="24f84-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="24f84-249">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="24f84-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="24f84-250">このデプロイでは、仮想ネットワーク、Ubuntu と Apache を実行する 3 つの仮想マシンを含むロード バランサー、VPN ゲートウェイ、および前のセクションで作成したハブ VNet への VNet ピアリング接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="24f84-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="24f84-251">このデプロイには 20 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="24f84-252">Azure のスポーク VNet へのハブ VNet のピアリング</span><span class="sxs-lookup"><span data-stu-id="24f84-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="24f84-253">ハブ VNet の VNet ピアリング接続をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="24f84-254">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\hub` フォルダーに切り替えます。</span><span class="sxs-lookup"><span data-stu-id="24f84-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="24f84-255">次の bash または PowerShell コマンドを実行して、最初のスポークにピアリング接続をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="24f84-256">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="24f84-257">次の bash または PowerShell コマンドを実行して、2 番目のスポークにピアリング接続をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="24f84-258">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="24f84-259">接続をテストする</span><span class="sxs-lookup"><span data-stu-id="24f84-259">Test connectivity</span></span>

<span data-ttu-id="24f84-260">オンプレミスのデータセンターに接続されているハブスポーク トポロジが機能していることを確認するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="24f84-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="24f84-261">[Azure Portal][ポータル] からサブスクリプションに接続し、`ra-onprem-rg` リソース グループの `ra-onprem-vm1` 仮想マシンに移動します。</span><span class="sxs-lookup"><span data-stu-id="24f84-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="24f84-262">`Overview` ブレードで、VM の`Public IP address`をメモします。</span><span class="sxs-lookup"><span data-stu-id="24f84-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="24f84-263">SSH クライアントを使用して、前の手順でメモした IP アドレスに接続します。この場合、デプロイ時に指定したユーザー名とパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="24f84-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="24f84-264">接続先の VM のコマンド プロンプトから次のコマンドを実行して、オンプレミスの VNet から Spoke1 VNet への接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="24f84-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="24f84-265">スポーク間に接続を追加する</span><span class="sxs-lookup"><span data-stu-id="24f84-265">Add connectivity between spokes</span></span>

<span data-ttu-id="24f84-266">スポークを相互に接続できるようにする場合は、他のスポーク宛てのトラフィックをハブ VNet のゲートウェイに転送する UDR を各スポークにデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="24f84-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="24f84-267">次の手順を実行して、現在はスポークから別のスポークに接続できないことを確認します。続いて、UDR をデプロイし、接続をもう一度テストします。</span><span class="sxs-lookup"><span data-stu-id="24f84-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="24f84-268">ジャンプボックス VM に接続されていない場合は、前のセクションの手順 1 ～ 4 を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="24f84-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="24f84-269">スポーク 1 でいずれかの Web サーバーに接続します。</span><span class="sxs-lookup"><span data-stu-id="24f84-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="24f84-270">スポーク 1 とスポーク 2 の間の接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="24f84-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="24f84-271">この接続は失敗します。</span><span class="sxs-lookup"><span data-stu-id="24f84-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="24f84-272">コンピューターのコマンド プロンプトに戻ります。</span><span class="sxs-lookup"><span data-stu-id="24f84-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="24f84-273">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\spokes` フォルダーに切り替えます。</span><span class="sxs-lookup"><span data-stu-id="24f84-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="24f84-274">次の bash または PowerShell コマンドを実行して、最初のスポークに UDR をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="24f84-275">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="24f84-276">次の bash または PowerShell コマンドを実行して、2 番目のスポークに UDR をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="24f84-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="24f84-277">値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="24f84-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="24f84-278">SSH ターミナルに戻ります。</span><span class="sxs-lookup"><span data-stu-id="24f84-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="24f84-279">スポーク 1 とスポーク 2 の間の接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="24f84-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="24f84-280">この接続は成功します。</span><span class="sxs-lookup"><span data-stu-id="24f84-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-spoke topology in Azure" (Azure のハブスポーク トポロジ)
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing" (Azure のハブスポーク トポロジと推移的なルーティング)
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing using an NVA" (Azure のハブスポーク トポロジと NVA を使用した推移的なルーティング)
[3]: ./images/hub-spokehub-spoke.svg "Hub-spoke-hub-spoke topology in Azure" (Azure のハブスポークハブスポーク トポロジ)
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
