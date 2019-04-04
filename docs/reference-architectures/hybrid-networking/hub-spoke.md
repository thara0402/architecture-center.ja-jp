---
title: Azure にハブスポーク ネットワーク トポロジを実装する
titleSuffix: Azure Reference Architectures
description: Azure にハブスポーク ネットワーク トポロジを実装します。
author: telmosampaio
ms.date: 10/08/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: dbd2a9a8fbb18586e7b255873a9a503117deabcd
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489183"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="95f73-103">Azure にハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="95f73-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="95f73-104">この参照アーキテクチャでは、Azure にハブスポーク トポロジを実装する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="95f73-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="95f73-105">*ハブ*は、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="95f73-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="95f73-106">*スポーク*は、ハブとピアリングする VNet であり、ワークロードを分離するために使用できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="95f73-107">トラフィックは、ExpressRoute または VPN ゲートウェイ接続を経由してオンプレミスのデータセンターとハブの間を流れます。</span><span class="sxs-lookup"><span data-stu-id="95f73-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span> <span data-ttu-id="95f73-108">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="95f73-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="95f73-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="95f73-109">![[0]][0]</span></span>

<span data-ttu-id="95f73-110">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="95f73-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="95f73-111">このトポロジの利点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="95f73-111">The benefits of this toplogy include:</span></span>

- <span data-ttu-id="95f73-112">**コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。</span><span class="sxs-lookup"><span data-stu-id="95f73-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="95f73-113">**サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。</span><span class="sxs-lookup"><span data-stu-id="95f73-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="95f73-114">**問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。</span><span class="sxs-lookup"><span data-stu-id="95f73-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="95f73-115">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="95f73-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="95f73-116">DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。</span><span class="sxs-lookup"><span data-stu-id="95f73-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="95f73-117">共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="95f73-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="95f73-118">相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。</span><span class="sxs-lookup"><span data-stu-id="95f73-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="95f73-119">セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。</span><span class="sxs-lookup"><span data-stu-id="95f73-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="95f73-120">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="95f73-120">Architecture</span></span>

<span data-ttu-id="95f73-121">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="95f73-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="95f73-122">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="95f73-122">**On-premises network**.</span></span> <span data-ttu-id="95f73-123">組織内で実行されているプライベートなローカル エリア ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="95f73-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="95f73-124">**VPN デバイス**。</span><span class="sxs-lookup"><span data-stu-id="95f73-124">**VPN device**.</span></span> <span data-ttu-id="95f73-125">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。</span><span class="sxs-lookup"><span data-stu-id="95f73-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="95f73-126">VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。</span><span class="sxs-lookup"><span data-stu-id="95f73-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="95f73-127">サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95f73-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="95f73-128">**VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="95f73-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="95f73-129">仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="95f73-130">詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95f73-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="95f73-131">この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="95f73-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="95f73-132">**ハブ VNet**。</span><span class="sxs-lookup"><span data-stu-id="95f73-132">**Hub VNet**.</span></span> <span data-ttu-id="95f73-133">ハブスポーク トポロジのハブとして使用する Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="95f73-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="95f73-134">ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。</span><span class="sxs-lookup"><span data-stu-id="95f73-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="95f73-135">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="95f73-135">**Gateway subnet**.</span></span> <span data-ttu-id="95f73-136">仮想ネットワーク ゲートウェイは、同じサブネット内に保持されます。</span><span class="sxs-lookup"><span data-stu-id="95f73-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="95f73-137">**スポーク VNet**。</span><span class="sxs-lookup"><span data-stu-id="95f73-137">**Spoke VNets**.</span></span> <span data-ttu-id="95f73-138">ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。</span><span class="sxs-lookup"><span data-stu-id="95f73-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="95f73-139">スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="95f73-140">各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。</span><span class="sxs-lookup"><span data-stu-id="95f73-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="95f73-141">アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95f73-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="95f73-142">**VNet ピアリング**。</span><span class="sxs-lookup"><span data-stu-id="95f73-142">**VNet peering**.</span></span> <span data-ttu-id="95f73-143">[ピアリング接続][vnet-peering]を使用して、2 つの VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-143">Two VNets can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="95f73-144">ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。</span><span class="sxs-lookup"><span data-stu-id="95f73-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="95f73-145">ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。</span><span class="sxs-lookup"><span data-stu-id="95f73-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="95f73-146">ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。</span><span class="sxs-lookup"><span data-stu-id="95f73-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span> <span data-ttu-id="95f73-147">同じリージョンまたは異なるリージョンの仮想ネットワークをピアリングできます。</span><span class="sxs-lookup"><span data-stu-id="95f73-147">You can peer virtual networks in the same region, or different regions.</span></span> <span data-ttu-id="95f73-148">詳細については、「[要件と制約][vnet-peering-requirements]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-148">For more information, see [Requirements and constraints][vnet-peering-requirements].</span></span>

> [!NOTE]
> <span data-ttu-id="95f73-149">この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="95f73-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="95f73-150">これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。</span><span class="sxs-lookup"><span data-stu-id="95f73-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="95f73-151">Recommendations</span><span class="sxs-lookup"><span data-stu-id="95f73-151">Recommendations</span></span>

<span data-ttu-id="95f73-152">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="95f73-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="95f73-153">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="95f73-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="95f73-154">リソース グループ</span><span class="sxs-lookup"><span data-stu-id="95f73-154">Resource groups</span></span>

<span data-ttu-id="95f73-155">ハブ VNet と各スポーク VNet は異なるリソース グループに実装でき、異なるサブスクリプションに実装することさえできます。</span><span class="sxs-lookup"><span data-stu-id="95f73-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions.</span></span> <span data-ttu-id="95f73-156">異なるサブスクリプションに属する仮想ネットワークをピアリングする場合、両方のサブスクリプションを同じまたは異なる Azure Active Directory テナントに関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="95f73-156">When you peer virtual networks in different subscriptions, both subscriptions can be associated to the same or different Azure Active Directory tenant.</span></span> <span data-ttu-id="95f73-157">これにより、各ワークロードを分散管理し、サービスの共有はハブ VNet で維持できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-157">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="95f73-158">VNet と GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="95f73-158">VNet and GatewaySubnet</span></span>

<span data-ttu-id="95f73-159">*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。</span><span class="sxs-lookup"><span data-stu-id="95f73-159">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="95f73-160">このサブネットは、仮想ネットワーク ゲートウェイで必要です。</span><span class="sxs-lookup"><span data-stu-id="95f73-160">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="95f73-161">このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-161">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="95f73-162">ゲートウェイの設定について詳しくは、接続の種類に応じて、次の参照アーキテクチャをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95f73-162">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="95f73-163">[ExpressRoute を使用するハイブリッド ネットワーク][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="95f73-163">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="95f73-164">[VPN ゲートウェイを使用するハイブリッド ネットワーク][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="95f73-164">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="95f73-165">高可用性を確保するために、ExpressRoute に加えてフェールオーバー用の VPN を使用できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-165">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="95f73-166">「[Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha]」(ExpressRoute と VPN フェールオーバーを使用してオンプレミス ネットワークを Azure に接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95f73-166">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="95f73-167">オンプレミス ネットワークに接続する必要がない場合は、ゲートウェイを設定せずにハブスポーク トポロジを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="95f73-167">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span>

### <a name="vnet-peering"></a><span data-ttu-id="95f73-168">VNET ピアリング</span><span class="sxs-lookup"><span data-stu-id="95f73-168">VNet peering</span></span>

<span data-ttu-id="95f73-169">VNet ピアリングは、2 つの VNet 間の非推移的な関係です。</span><span class="sxs-lookup"><span data-stu-id="95f73-169">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="95f73-170">相互接続のためにスポークが必要な場合は、これらのスポーク間に個別のピアリング接続を追加することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-170">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="95f73-171">ただし、相互接続に必要なスポークが複数ある場合は、[VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]のために、有効なピアリング接続をすぐに使い果たします。</span><span class="sxs-lookup"><span data-stu-id="95f73-171">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="95f73-172">このシナリオでは、ユーザー定義ルート (UDR) を使用して、スポーク宛てのトラフィックをハブ VNet のルーターとして機能する NVA に強制的に送信することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-172">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="95f73-173">これにより、スポークを相互に接続できます。</span><span class="sxs-lookup"><span data-stu-id="95f73-173">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="95f73-174">リモート ネットワークと通信するハブ VNet ゲートウェイを使用するようにスポークを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="95f73-174">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="95f73-175">ゲートウェイ トラフィックをスポークからハブに流して、リモート ネットワークに接続するには、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95f73-175">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

- <span data-ttu-id="95f73-176">**ゲートウェイ トランジットを許可する**ようにハブで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="95f73-176">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
- <span data-ttu-id="95f73-177">**リモート ゲートウェイを使用する**ように各スポークで VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="95f73-177">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
- <span data-ttu-id="95f73-178">**転送されたトラフィックを許可する**ようにすべての VNet ピアリング接続を構成します。</span><span class="sxs-lookup"><span data-stu-id="95f73-178">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="95f73-179">考慮事項</span><span class="sxs-lookup"><span data-stu-id="95f73-179">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="95f73-180">スポーク接続</span><span class="sxs-lookup"><span data-stu-id="95f73-180">Spoke connectivity</span></span>

<span data-ttu-id="95f73-181">スポーク間の接続が必要な場合は、ハブでのルーティング用の NVA の実装、およびスポークでの UDR を使用したハブへのトラフィックの転送を検討してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-181">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="95f73-182">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="95f73-182">![[2]][2]</span></span>

<span data-ttu-id="95f73-183">このシナリオでは、**転送されたトラフィックを許可する**ようにピアリング接続を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95f73-183">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

<span data-ttu-id="95f73-184">また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-184">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="95f73-185">たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。</span><span class="sxs-lookup"><span data-stu-id="95f73-185">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="95f73-186">このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="95f73-186">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="95f73-187">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="95f73-187">Deploy the solution</span></span>

<span data-ttu-id="95f73-188">このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95f73-188">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="95f73-189">ここでは、各 VNet の VM を使用して接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="95f73-189">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="95f73-190">実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。</span><span class="sxs-lookup"><span data-stu-id="95f73-190">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="95f73-191">デプロイによってサブスクリプション内に作成されるリソース グループは、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="95f73-191">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="95f73-192">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-192">hub-nva-rg</span></span>
- <span data-ttu-id="95f73-193">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-193">hub-vnet-rg</span></span>
- <span data-ttu-id="95f73-194">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-194">onprem-jb-rg</span></span>
- <span data-ttu-id="95f73-195">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-195">onprem-vnet-rg</span></span>
- <span data-ttu-id="95f73-196">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-196">spoke1-vnet-rg</span></span>
- <span data-ttu-id="95f73-197">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="95f73-197">spoke2-vent-rg</span></span>

<span data-ttu-id="95f73-198">テンプレート パラメーター ファイルは、これらの名前を参照します。したがって、名前を変更する場合は、それに合わせてパラメーター ファイルも更新します。</span><span class="sxs-lookup"><span data-stu-id="95f73-198">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="95f73-199">前提条件</span><span class="sxs-lookup"><span data-stu-id="95f73-199">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="95f73-200">シミュレートされたオンプレミスのデータセンターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="95f73-200">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="95f73-201">シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="95f73-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="95f73-202">参照アーキテクチャ リポジトリの `hybrid-networking/hub-spoke` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="95f73-202">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="95f73-203">`onprem.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-203">Open the `onprem.json` file.</span></span> <span data-ttu-id="95f73-204">`adminUsername` および `adminPassword` の値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="95f73-204">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="95f73-205">(省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。</span><span class="sxs-lookup"><span data-stu-id="95f73-205">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="95f73-206">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-206">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="95f73-207">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="95f73-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="95f73-208">このデプロイでは、仮想ネットワーク、仮想マシン、および VPN ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="95f73-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="95f73-209">VPN ゲートウェイの作成には約 40 分かかります。</span><span class="sxs-lookup"><span data-stu-id="95f73-209">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="95f73-210">ハブ VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="95f73-210">Deploy the hub VNet</span></span>

<span data-ttu-id="95f73-211">ハブ VNet をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-211">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="95f73-212">`hub-vnet.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-212">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="95f73-213">`adminUsername` および `adminPassword` の値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="95f73-213">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="95f73-214">(省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。</span><span class="sxs-lookup"><span data-stu-id="95f73-214">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="95f73-215">`sharedKey` の両方のインスタンスを見つけ、VPN 接続の共有キーを入力します。</span><span class="sxs-lookup"><span data-stu-id="95f73-215">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="95f73-216">値は一致する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95f73-216">The values must match.</span></span>

    ```json
    "sharedKey": "",
    ```

4. <span data-ttu-id="95f73-217">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-217">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="95f73-218">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="95f73-218">Wait for the deployment to finish.</span></span> <span data-ttu-id="95f73-219">このデプロイでは、仮想ネットワーク、仮想マシン、VPN ゲートウェイ、およびゲートウェイへの接続が作成されます。</span><span class="sxs-lookup"><span data-stu-id="95f73-219">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="95f73-220">VPN ゲートウェイの作成には約 40 分かかります。</span><span class="sxs-lookup"><span data-stu-id="95f73-220">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-to-the-hub-vnet-mdash-windows-deployment"></a><span data-ttu-id="95f73-221">ハブ VNet への接続をテストする &mdash; Windows デプロイ</span><span class="sxs-lookup"><span data-stu-id="95f73-221">Test connectivity to the hub VNet &mdash; Windows deployment</span></span>

<span data-ttu-id="95f73-222">Windows VM を使用して、シミュレートされたオンプレミスの環境からハブ VNet への接続をテストするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="95f73-222">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, follow these steps:</span></span>

1. <span data-ttu-id="95f73-223">Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="95f73-223">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="95f73-224">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-224">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="95f73-225">`onprem.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="95f73-225">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="95f73-226">VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、ハブ VNet の ジャンプボックス VM に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="95f73-226">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="95f73-227">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="95f73-227">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="95f73-228">既定で、Windows Server VM では Azure の ICMP 応答が許可されていません。</span><span class="sxs-lookup"><span data-stu-id="95f73-228">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="95f73-229">接続のテストに `ping` を使用する場合は、VM ごとに Windows の高度なファイアウォールで ICMP トラフィックを有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="95f73-229">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

### <a name="test-connectivity-to-the-hub-vnet-mdash-linux-deployment"></a><span data-ttu-id="95f73-230">ハブ VNet への接続をテストする &mdash; Linux デプロイ</span><span class="sxs-lookup"><span data-stu-id="95f73-230">Test connectivity to the hub VNet &mdash; Linux deployment</span></span>

<span data-ttu-id="95f73-231">Linux VM を使用して、シミュレートされたオンプレミスの環境からハブ VNet への接続をテストするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="95f73-231">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, follow these steps:</span></span>

1. <span data-ttu-id="95f73-232">Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="95f73-232">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="95f73-233">`Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。</span><span class="sxs-lookup"><span data-stu-id="95f73-233">Click `Connect` and copy the `ssh` command shown in the portal.</span></span>

3. <span data-ttu-id="95f73-234">Linux プロンプトから `ssh` を実行して、シミュレートされたオンプレミスの環境に接続します。</span><span class="sxs-lookup"><span data-stu-id="95f73-234">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="95f73-235">`onprem.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="95f73-235">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="95f73-236">`ping` コマンドを使用して、ハブ VNet のジャンプボックス VM への接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="95f73-236">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="95f73-237">スポーク VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="95f73-237">Deploy the spoke VNets</span></span>

<span data-ttu-id="95f73-238">スポーク VNet をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-238">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="95f73-239">`spoke1.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-239">Open the `spoke1.json` file.</span></span> <span data-ttu-id="95f73-240">`adminUsername` および `adminPassword` の値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="95f73-240">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="95f73-241">(省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。</span><span class="sxs-lookup"><span data-stu-id="95f73-241">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="95f73-242">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-242">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="95f73-243">`spoke2.json` ファイルに対して手順 1. ～ 2. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="95f73-243">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="95f73-244">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-244">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="95f73-245">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-245">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity-to-the-spoke-vnets-mdash-windows-deployment"></a><span data-ttu-id="95f73-246">スポーク VNet への接続をテストする &mdash; Windows デプロイ</span><span class="sxs-lookup"><span data-stu-id="95f73-246">Test connectivity to the spoke VNets &mdash; Windows deployment</span></span>

<span data-ttu-id="95f73-247">Windows VM を使用して、シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストするには、次の手順を行います。</span><span class="sxs-lookup"><span data-stu-id="95f73-247">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps:</span></span>

1. <span data-ttu-id="95f73-248">Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="95f73-248">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="95f73-249">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-249">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="95f73-250">`onprem.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="95f73-250">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="95f73-251">VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、スポーク VNet の ジャンプボックス VM に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="95f73-251">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

### <a name="test-connectivity-to-the-spoke-vnets-mdash-linux-deployment"></a><span data-ttu-id="95f73-252">スポーク VNet への接続をテストする &mdash; Linux デプロイ</span><span class="sxs-lookup"><span data-stu-id="95f73-252">Test connectivity to the spoke VNets &mdash; Linux deployment</span></span>

<span data-ttu-id="95f73-253">Linux VM を使用して、シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-253">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="95f73-254">Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="95f73-254">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="95f73-255">`Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。</span><span class="sxs-lookup"><span data-stu-id="95f73-255">Click `Connect` and copy the `ssh` command shown in the portal.</span></span>

3. <span data-ttu-id="95f73-256">Linux プロンプトから `ssh` を実行して、シミュレートされたオンプレミスの環境に接続します。</span><span class="sxs-lookup"><span data-stu-id="95f73-256">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="95f73-257">`onprem.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="95f73-257">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="95f73-258">`ping` コマンドを使用して、各スポーク内のジャンプボックス VM への接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="95f73-258">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="95f73-259">スポーク間に接続を追加する</span><span class="sxs-lookup"><span data-stu-id="95f73-259">Add connectivity between spokes</span></span>

<span data-ttu-id="95f73-260">この手順は省略可能です。</span><span class="sxs-lookup"><span data-stu-id="95f73-260">This step is optional.</span></span> <span data-ttu-id="95f73-261">スポークに相互接続を許可する場合は、ハブの VNet 内のルーターとしてネットワーク仮想アプライアンス (NVA) を使用し、別のスポークへの接続を試みるときにスポークからルーターへのトラフィックを強制する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95f73-261">If you want to allow spokes to connect to each other, you must use a network virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="95f73-262">1 つの VM としての基本的なサンプル NVA と、ユーザー定義のルート (UDR) をデプロイして、2 つのスポーク VNet の接続を許可するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-262">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="95f73-263">`hub-nva.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="95f73-263">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="95f73-264">`adminUsername` および `adminPassword` の値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="95f73-264">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="95f73-265">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="95f73-265">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

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
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Azure のハブスポーク トポロジ"
[1]: ./images/hub-spoke-gateway-routing.svg "Azure のハブスポーク トポロジと推移的なルーティング"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Azure のハブスポーク トポロジと NVA を使用した推移的なルーティング"
[3]: ./images/hub-spokehub-spoke.svg "Azure のハブスポークハブスポーク トポロジ"
