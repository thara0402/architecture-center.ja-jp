---
title: "Azure にハブスポーク ネットワーク トポロジを実装する"
description: "Azure にハブスポーク ネットワーク トポロジを実装する方法。"
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 1a2855f0d4a903fc4d7a022aef20ea73fe763e2c
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="56db3-103">Azure にハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="56db3-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="56db3-104">この参照アーキテクチャでは、Azure にハブスポーク トポロジを実装する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="56db3-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="56db3-105">*ハブ*は、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="56db3-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="56db3-106">*スポーク*は、ハブとピアリングする VNet であり、ワークロードを分離するために使用できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="56db3-107">トラフィックは、ExpressRoute または VPN ゲートウェイ接続を経由してオンプレミスのデータセンターとハブの間を流れます。</span><span class="sxs-lookup"><span data-stu-id="56db3-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="56db3-108">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="56db3-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="56db3-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="56db3-109">![[0]][0]</span></span>

<span data-ttu-id="56db3-110">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="56db3-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="56db3-111">このトポロジの利点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="56db3-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="56db3-112">**コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。</span><span class="sxs-lookup"><span data-stu-id="56db3-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="56db3-113">**サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。</span><span class="sxs-lookup"><span data-stu-id="56db3-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="56db3-114">**問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。</span><span class="sxs-lookup"><span data-stu-id="56db3-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="56db3-115">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="56db3-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="56db3-116">DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。</span><span class="sxs-lookup"><span data-stu-id="56db3-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="56db3-117">共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="56db3-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="56db3-118">相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。</span><span class="sxs-lookup"><span data-stu-id="56db3-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="56db3-119">セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。</span><span class="sxs-lookup"><span data-stu-id="56db3-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="56db3-120">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="56db3-120">Architecture</span></span>

<span data-ttu-id="56db3-121">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="56db3-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="56db3-122">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="56db3-122">**On-premises network**.</span></span> <span data-ttu-id="56db3-123">組織内で実行されているプライベートなローカル エリア ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="56db3-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="56db3-124">**VPN デバイス**。</span><span class="sxs-lookup"><span data-stu-id="56db3-124">**VPN device**.</span></span> <span data-ttu-id="56db3-125">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。</span><span class="sxs-lookup"><span data-stu-id="56db3-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="56db3-126">VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。</span><span class="sxs-lookup"><span data-stu-id="56db3-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="56db3-127">サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="56db3-128">**VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="56db3-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="56db3-129">仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="56db3-130">詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="56db3-131">この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="56db3-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="56db3-132">**ハブ VNet**。</span><span class="sxs-lookup"><span data-stu-id="56db3-132">**Hub VNet**.</span></span> <span data-ttu-id="56db3-133">ハブスポーク トポロジのハブとして使用する Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="56db3-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="56db3-134">ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。</span><span class="sxs-lookup"><span data-stu-id="56db3-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="56db3-135">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="56db3-135">**Gateway subnet**.</span></span> <span data-ttu-id="56db3-136">仮想ネットワーク ゲートウェイは、同じサブネット内に保持されます。</span><span class="sxs-lookup"><span data-stu-id="56db3-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="56db3-137">**スポーク VNet**。</span><span class="sxs-lookup"><span data-stu-id="56db3-137">**Spoke VNets**.</span></span> <span data-ttu-id="56db3-138">ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。</span><span class="sxs-lookup"><span data-stu-id="56db3-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="56db3-139">スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="56db3-140">各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。</span><span class="sxs-lookup"><span data-stu-id="56db3-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="56db3-141">アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="56db3-142">**VNet ピアリング**。</span><span class="sxs-lookup"><span data-stu-id="56db3-142">**VNet peering**.</span></span> <span data-ttu-id="56db3-143">[ピアリング接続][vnet-peering]を使用して、同じ Azure リージョン内の 2 つの VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="56db3-144">ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。</span><span class="sxs-lookup"><span data-stu-id="56db3-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="56db3-145">ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。</span><span class="sxs-lookup"><span data-stu-id="56db3-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="56db3-146">ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="56db3-147">この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="56db3-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="56db3-148">これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。</span><span class="sxs-lookup"><span data-stu-id="56db3-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="56db3-149">Recommendations</span><span class="sxs-lookup"><span data-stu-id="56db3-149">Recommendations</span></span>

<span data-ttu-id="56db3-150">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="56db3-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="56db3-151">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="56db3-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="56db3-152">リソース グループ</span><span class="sxs-lookup"><span data-stu-id="56db3-152">Resource groups</span></span>

<span data-ttu-id="56db3-153">ハブ VNet と各スポーク VNet を異なるリソース グループに実装できます。同じ Azure リージョン内の同じ Azure Active Directory (Azure AD) テナントに属している限り、サブスクリプションが異なっていてもかまいません。</span><span class="sxs-lookup"><span data-stu-id="56db3-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="56db3-154">これにより、各ワークロードを分散管理し、サービスの共有はハブ VNet で維持できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="56db3-155">VNet と GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="56db3-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="56db3-156">*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。</span><span class="sxs-lookup"><span data-stu-id="56db3-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="56db3-157">このサブネットは、仮想ネットワーク ゲートウェイで必要です。</span><span class="sxs-lookup"><span data-stu-id="56db3-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="56db3-158">このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="56db3-159">ゲートウェイの設定について詳しくは、接続の種類に応じて、次の参照アーキテクチャをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="56db3-160">[ExpressRoute を使用するハイブリッド ネットワーク][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="56db3-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="56db3-161">[VPN ゲートウェイを使用するハイブリッド ネットワーク][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="56db3-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="56db3-162">高可用性を確保するために、ExpressRoute に加えてフェールオーバー用の VPN を使用できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="56db3-163">「[Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha]」(ExpressRoute と VPN フェールオーバーを使用してオンプレミス ネットワークを Azure に接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="56db3-164">オンプレミス ネットワークに接続する必要がない場合は、ゲートウェイを設定せずにハブスポーク トポロジを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="56db3-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="56db3-165">VNET ピアリング</span><span class="sxs-lookup"><span data-stu-id="56db3-165">VNet peering</span></span>

<span data-ttu-id="56db3-166">VNet ピアリングは、2 つの VNet 間の非推移的な関係です。</span><span class="sxs-lookup"><span data-stu-id="56db3-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="56db3-167">相互接続のためにスポークが必要な場合は、これらのスポーク間に個別のピアリング接続を追加することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="56db3-168">ただし、相互接続に必要なスポークが複数ある場合は、[VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]のために、有効なピアリング接続をすぐに使い果たします。</span><span class="sxs-lookup"><span data-stu-id="56db3-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="56db3-169">このシナリオでは、ユーザー定義ルート (UDR) を使用して、スポーク宛てのトラフィックをハブ VNet のルーターとして機能する NVA に強制的に送信することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="56db3-170">これにより、スポークを相互に接続できます。</span><span class="sxs-lookup"><span data-stu-id="56db3-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="56db3-171">リモート ネットワークと通信するハブ VNet ゲートウェイを使用するようにスポークを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="56db3-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="56db3-172">ゲートウェイ トラフィックをスポークからハブに流して、リモート ネットワークに接続するには、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="56db3-173">**ゲートウェイ トランジットを許可する**ようにハブで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="56db3-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="56db3-174">**リモート ゲートウェイを使用する**ように各スポークで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="56db3-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="56db3-175">**転送されたトラフィックを許可する**ようにすべての VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="56db3-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="56db3-176">考慮事項</span><span class="sxs-lookup"><span data-stu-id="56db3-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="56db3-177">スポーク接続</span><span class="sxs-lookup"><span data-stu-id="56db3-177">Spoke connectivity</span></span>

<span data-ttu-id="56db3-178">スポーク間の接続が必要な場合は、ハブでのルーティング用の NVA の実装、およびスポークでの UDR を使用したハブへのトラフィックの転送を検討してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="56db3-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="56db3-179">![[2]][2]</span></span>

<span data-ttu-id="56db3-180">このシナリオでは、**転送されたトラフィックを許可する**ようにピアリング接続を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="56db3-181">VNet ピアリングの制限の解消</span><span class="sxs-lookup"><span data-stu-id="56db3-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="56db3-182">Azure の [VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="56db3-183">制限を超える数のスポークを許可する場合は、ハブスポークハブスポーク トポロジの作成を検討してください。このトポロジでは、第 1 レベルのスポークもハブとして機能します。</span><span class="sxs-lookup"><span data-stu-id="56db3-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="56db3-184">この手法を次の図に示します。</span><span class="sxs-lookup"><span data-stu-id="56db3-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="56db3-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="56db3-185">![[3]][3]</span></span>

<span data-ttu-id="56db3-186">また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="56db3-187">たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。</span><span class="sxs-lookup"><span data-stu-id="56db3-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="56db3-188">このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="56db3-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="56db3-189">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="56db3-189">Deploy the solution</span></span>

<span data-ttu-id="56db3-190">このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="56db3-191">このデプロイでは、各 VNet の Ubuntu VM を使用して接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="56db3-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="56db3-192">実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。</span><span class="sxs-lookup"><span data-stu-id="56db3-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="56db3-193">前提条件</span><span class="sxs-lookup"><span data-stu-id="56db3-193">Prerequisites</span></span>

<span data-ttu-id="56db3-194">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="56db3-195">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="56db3-195">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="56db3-196">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="56db3-197">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="56db3-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="56db3-198">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="56db3-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="56db3-199">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="56db3-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="56db3-200">シミュレートされたオンプレミスのデータセンターを azbb を使用してデプロイする</span><span class="sxs-lookup"><span data-stu-id="56db3-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="56db3-201">シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="56db3-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="56db3-202">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="56db3-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="56db3-203">`onprem.json` ファイルを開き、次に示す 36 行目と 37 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="56db3-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="56db3-204">38 行目の `osType` に `Windows` または `Linux` と入力し、ジャンプボックス用のオペレーティング システムとして Windows Server 2016 Datacenter または Ubuntu 16.04 のいずれかをインストールします。</span><span class="sxs-lookup"><span data-stu-id="56db3-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="56db3-205">次に示すように、`azbb` を実行して、シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="56db3-206">別のリソース グループ名 (`onprem-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="56db3-207">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="56db3-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="56db3-208">このデプロイでは、仮想ネットワーク、仮想マシン、および VPN ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="56db3-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="56db3-209">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="56db3-210">Azure のハブ VNet</span><span class="sxs-lookup"><span data-stu-id="56db3-210">Azure hub VNet</span></span>

<span data-ttu-id="56db3-211">ハブ VNet をデプロイして、上記の手順で作成したシミュレートされたオンプレミスの VNet に接続するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="56db3-212">`hub-vnet.json` ファイルを開き、次に示す 39 行目と 40 行目の引用符の間にユーザー名とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="56db3-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="56db3-213">41 行目の `osType` に `Windows` または `Linux` と入力し、ジャンプボックス用のオペレーティング システムとして Windows Server 2016 Datacenter または Ubuntu 16.04 のいずれかをインストールします。</span><span class="sxs-lookup"><span data-stu-id="56db3-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="56db3-214">次に示す 72 行目の引用符の間に共有キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="56db3-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="56db3-215">次に示すように、`azbb` を実行して、シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="56db3-216">別のリソース グループ名 (`hub-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="56db3-217">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="56db3-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="56db3-218">このデプロイでは、仮想ネットワーク、仮想マシン、VPN ゲートウェイ、および前のセクションで作成したゲートウェイへの接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="56db3-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="56db3-219">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="56db3-220">(省略可能) オンプレミスからハブへの接続をテストする</span><span class="sxs-lookup"><span data-stu-id="56db3-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="56db3-221">Windows VM を使用して、シミュレートされたオンプレミスの環境からハブ VNet への接続をテストするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="56db3-222">Azure Portal から `onprem-jb-rg` リソース グループに移動し、`jb-vm1` 仮想マシン リソースをクリックします。</span><span class="sxs-lookup"><span data-stu-id="56db3-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="56db3-223">ポータルの VM ブレードの左上隅にある `Connect` をクリックし、メッセージの指示に従ってリモート デスクトップを使用して VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="56db3-224">必ず、`onprem.json` ファイルの 36 行目および 37 行目に指定したユーザー名とパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="56db3-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="56db3-225">VM で PowerShell コンソールを開き、次に示すように `Test-NetConnection` コマンドレットを使用して、ハブ ジャンプボックス VM に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="56db3-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
  ```
  > [!NOTE]
  > <span data-ttu-id="56db3-226">既定で、Windows Server VM では Azure の ICMP 応答が許可されていません。</span><span class="sxs-lookup"><span data-stu-id="56db3-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="56db3-227">接続のテストに `ping` を使用する場合は、VM ごとに Windows の高度なファイアウォールで ICMP トラフィックを有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="56db3-228">Linux VM を使用して、シミュレートされたオンプレミスの環境からハブ VNet への接続をテストするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="56db3-229">Azure Portal から `onprem-jb-rg` リソース グループに移動し、`jb-vm1` 仮想マシン リソースをクリックします。</span><span class="sxs-lookup"><span data-stu-id="56db3-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="56db3-230">ポータルの VM ブレードの左上隅にある `Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。</span><span class="sxs-lookup"><span data-stu-id="56db3-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="56db3-231">次に示すように、Linux プロンプトから `ssh` を実行し、上記の手順 2. でコピーした情報を使用して、シミュレートされたオンプレミス環境のジャンプボックスに接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

4. <span data-ttu-id="56db3-232">`onprem.json` ファイルの 37 行目に指定したパスワードを使用して、VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="56db3-233">次に示すように、`ping` コマンドを使用してハブ ジャンプボックスへの接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="56db3-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

  ```bash
  ping 10.0.0.68
  ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="56db3-234">Azure のスポーク VNet</span><span class="sxs-lookup"><span data-stu-id="56db3-234">Azure spoke VNets</span></span>

<span data-ttu-id="56db3-235">スポーク VNet をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="56db3-236">`spoke1.json` ファイルを開き、次に示す 47 行目と 48 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="56db3-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="56db3-237">49 行目の `osType` に `Windows` または `Linux` と入力し、ジャンプボックス用のオペレーティング システムとして Windows Server 2016 Datacenter または Ubuntu 16.04 のいずれかをインストールします。</span><span class="sxs-lookup"><span data-stu-id="56db3-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="56db3-238">次に示すように、`azbb` を実行して、最初のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="56db3-239">別のリソース グループ名 (`spoke1-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="56db3-240">ファイル `spoke2.json` に対して上記の手順 1. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="56db3-240">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="56db3-241">次に示すように、`azbb` を実行して、2 番目のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="56db3-242">別のリソース グループ名 (`spoke2-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="56db3-243">Azure のスポーク VNet へのハブ VNet のピアリング</span><span class="sxs-lookup"><span data-stu-id="56db3-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="56db3-244">ハブ VNet からスポーク VNet にピアリング接続を作成するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="56db3-245">`hub-vnet-peering.json` ファイルを開き、リソース グループ名、および 29 行目から始まる各仮想ネットワーク ピアリングの仮想ネットワーク名が正しいことを確認します。</span><span class="sxs-lookup"><span data-stu-id="56db3-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="56db3-246">次に示すように、`azbb` を実行して、最初のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="56db3-247">別のリソース グループ名 (`hub-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="56db3-248">接続をテストする</span><span class="sxs-lookup"><span data-stu-id="56db3-248">Test connectivity</span></span>

<span data-ttu-id="56db3-249">Windows VM を使用して、シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="56db3-250">Azure Portal から `onprem-jb-rg` リソース グループに移動し、`jb-vm1` 仮想マシン リソースをクリックします。</span><span class="sxs-lookup"><span data-stu-id="56db3-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="56db3-251">ポータルの VM ブレードの左上隅にある `Connect` をクリックし、メッセージの指示に従ってリモート デスクトップを使用して VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="56db3-252">必ず、`onprem.json` ファイルの 36 行目および 37 行目に指定したユーザー名とパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="56db3-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="56db3-253">VM で PowerShell コンソールを開き、次に示すように `Test-NetConnection` コマンドレットを使用して、ハブ ジャンプボックス VM に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="56db3-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
  Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
  ```

<span data-ttu-id="56db3-254">Linux VM を使用して、シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="56db3-255">Azure Portal から `onprem-jb-rg` リソース グループに移動し、`jb-vm1` 仮想マシン リソースをクリックします。</span><span class="sxs-lookup"><span data-stu-id="56db3-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="56db3-256">ポータルの VM ブレードの左上隅にある `Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。</span><span class="sxs-lookup"><span data-stu-id="56db3-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="56db3-257">次に示すように、Linux プロンプトから `ssh` を実行し、上記の手順 2. でコピーした情報を使用して、シミュレートされたオンプレミス環境のジャンプボックスに接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

5. <span data-ttu-id="56db3-258">`onprem.json` ファイルの 37 行目に指定したパスワードを使用して、VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="56db3-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

6. <span data-ttu-id="56db3-259">次に示すように、`ping` コマンドを使用して各スポークでジャンプボックス VM への接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="56db3-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

  ```bash
  ping 10.1.0.68
  ping 10.2.0.68
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="56db3-260">スポーク間に接続を追加する</span><span class="sxs-lookup"><span data-stu-id="56db3-260">Add connectivity between spokes</span></span>

<span data-ttu-id="56db3-261">スポークに相互接続を許可する場合は、ハブの仮想ネットワーク内のルーターとしてネットワーク仮想アプライアンス (NVA) を使用し、もう別のスポークへの接続を試みるときにスポークからルーターへのトラフィックを強制する必要があります。</span><span class="sxs-lookup"><span data-stu-id="56db3-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="56db3-262">1 つの VM として基本的なサンプル NVA と、2 つのスポーク VNet に接続を許可するために必要なユーザー定義のルートをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="56db3-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="56db3-263">`hub-nva.json` ファイルを開き、次に示す 13 行目と 14 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="56db3-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="56db3-264">`azbb` を実行して NVA VM とユーザー定義のルートをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="56db3-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="56db3-265">別のリソース グループ名 (`hub-nva-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="56db3-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
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
