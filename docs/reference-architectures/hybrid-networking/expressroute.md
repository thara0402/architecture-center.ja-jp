---
title: ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続
description: Azure ExpressRoute を使用して接続された Azure 仮想ネットワークとオンプレミス ネットワークにまたがる、セキュリティで保護されたサイト間ネットワーク アーキテクチャの実装方法。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 754542b37988e0cd2694ae84eb6b03d68c147243
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819178"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a><span data-ttu-id="d66ac-103">ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続</span><span class="sxs-lookup"><span data-stu-id="d66ac-103">Connect an on-premises network to Azure using ExpressRoute</span></span>

<span data-ttu-id="d66ac-104">この参照アーキテクチャでは、[Azure ExpressRoute][expressroute-introduction] を使用して、オンプレミス ネットワークを Azure の仮想ネットワークに接続する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-104">This reference architecture shows how to connect an on-premises network to virtual networks on Azure, using [Azure ExpressRoute][expressroute-introduction].</span></span> <span data-ttu-id="d66ac-105">ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-105">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="d66ac-106">プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-106">The private connection extends your on-premises network into Azure.</span></span> [<span data-ttu-id="d66ac-107">**以下のソリューションをデプロイします**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="d66ac-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="d66ac-108">![[0]][0]</span></span>

<span data-ttu-id="d66ac-109">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="d66ac-109">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="d66ac-110">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="d66ac-110">Architecture</span></span>

<span data-ttu-id="d66ac-111">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-111">The architecture consists of the following components.</span></span>

* <span data-ttu-id="d66ac-112">**オンプレミスの企業ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-112">**On-premises corporate network**.</span></span> <span data-ttu-id="d66ac-113">組織内で運用されているプライベートのローカル エリア ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="d66ac-113">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="d66ac-114">**ExpressRoute 回線**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-114">**ExpressRoute circuit**.</span></span> <span data-ttu-id="d66ac-115">エッジ ルーターを介してオンプレミス ネットワークを Azure につなげる、レイヤー 2 またはレイヤー 3 の回線 (接続プロバイダーから提供されます)。</span><span class="sxs-lookup"><span data-stu-id="d66ac-115">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="d66ac-116">この回線では、接続プロバイダーによって管理されるハードウェア インフラストラクチャが使用されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-116">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="d66ac-117">**ローカルのエッジ ルーター**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-117">**Local edge routers**.</span></span> <span data-ttu-id="d66ac-118">オンプレミス ネットワークを、プロバイダーによって管理されている回線に接続するルーター。</span><span class="sxs-lookup"><span data-stu-id="d66ac-118">Routers that connect the on-premises network to the circuit managed by the provider.</span></span> <span data-ttu-id="d66ac-119">接続のプロビジョニング方法によっては、ルーターが使用するパブリック IP アドレスを指定しなければならない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-119">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>
* <span data-ttu-id="d66ac-120">**Microsoft エッジ ルーター**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-120">**Microsoft edge routers**.</span></span> <span data-ttu-id="d66ac-121">アクティブ/アクティブの高可用性構成の 2 つのルーター。</span><span class="sxs-lookup"><span data-stu-id="d66ac-121">Two routers in an active-active highly available configuration.</span></span> <span data-ttu-id="d66ac-122">このルーターにより、接続プロバイダーが、その回線をデータセンターに直接接続できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-122">These routers enable a connectivity provider to connect their circuits directly to their datacenter.</span></span> <span data-ttu-id="d66ac-123">接続のプロビジョニング方法によっては、ルーターが使用するパブリック IP アドレスを指定しなければならない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-123">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>

* <span data-ttu-id="d66ac-124">**Azure 仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-124">**Azure virtual networks (VNets)**.</span></span> <span data-ttu-id="d66ac-125">各 VNet は 1 つの Azure リージョンに配置され、複数のアプリケーション層をホストできます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-125">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="d66ac-126">アプリケーション層は、各 VNet 内でサブネットを使用してセグメント化できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-126">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="d66ac-127">**Azure パブリック サービス**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-127">**Azure public services**.</span></span> <span data-ttu-id="d66ac-128">ハイブリッド アプリケーションで使用できる Azure サービス。</span><span class="sxs-lookup"><span data-stu-id="d66ac-128">Azure services that can be used within a hybrid application.</span></span> <span data-ttu-id="d66ac-129">このサービスはインターネット経由でも使用できますが、ExpressRoute 回線を使用してアクセスすると、トラフィックがインターネットを経由しないため、待ち時間が短縮され、パフォーマンスの予測可能性が向上します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-129">These services are also available over the Internet, but accessing them using an ExpressRoute circuit provides low latency and more predictable performance, because traffic does not go through the Internet.</span></span> <span data-ttu-id="d66ac-130">[パブリック ピアリング][expressroute-peering]を使用して接続が実行され、アドレスは、組織が所有するか、接続プロバイダーによって提供されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-130">Connections are performed using [public peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span>

* <span data-ttu-id="d66ac-131">**Office 365 サービス**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-131">**Office 365 services**.</span></span> <span data-ttu-id="d66ac-132">公開されている、Microsoft 提供の Office 365 アプリケーションとサービス。</span><span class="sxs-lookup"><span data-stu-id="d66ac-132">The publicly available Office 365 applications and services provided by Microsoft.</span></span> <span data-ttu-id="d66ac-133">[Microsoft ピアリング][expressroute-peering]を使用して接続が実行され、アドレスは、組織が所有するか、接続プロバイダーによって提供されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-133">Connections are performed using [Microsoft peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span> <span data-ttu-id="d66ac-134">また、Microsoft ピアリングを介して、Microsoft CRM Online に直接接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-134">You can also connect directly to Microsoft CRM Online through Microsoft peering.</span></span>

* <span data-ttu-id="d66ac-135">**接続プロバイダー** (非表示)。</span><span class="sxs-lookup"><span data-stu-id="d66ac-135">**Connectivity providers** (not shown).</span></span> <span data-ttu-id="d66ac-136">レイヤー 2 またはレイヤー 3 の接続を使用して、ご利用のデータセンターと Azure データセンターの間に接続を提供する企業。</span><span class="sxs-lookup"><span data-stu-id="d66ac-136">Companies that provide a connection either using layer 2 or layer 3 connectivity between your datacenter and an Azure datacenter.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d66ac-137">Recommendations</span><span class="sxs-lookup"><span data-stu-id="d66ac-137">Recommendations</span></span>

<span data-ttu-id="d66ac-138">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-138">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="d66ac-139">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-139">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="connectivity-providers"></a><span data-ttu-id="d66ac-140">接続プロバイダー</span><span class="sxs-lookup"><span data-stu-id="d66ac-140">Connectivity providers</span></span>

<span data-ttu-id="d66ac-141">自分の場所に適した ExpressRoute 接続プロバイダーを選択します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-141">Select a suitable ExpressRoute connectivity provider for your location.</span></span> <span data-ttu-id="d66ac-142">自分の場所で使用できる接続プロバイダーの一覧を取得するには、次の Azure PowerShell コマンドを使用します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-142">To get a list of connectivity providers available at your location, use the following Azure PowerShell command:</span></span>

```powershell
Get-AzureRmExpressRouteServiceProvider
```

<span data-ttu-id="d66ac-143">ExpressRoute 接続プロバイダーは、ご利用のデータセンターを次の方法で Microsoft に接続します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-143">ExpressRoute connectivity providers connect your datacenter to Microsoft in the following ways:</span></span>

* <span data-ttu-id="d66ac-144">**クラウド エクスチェンジにコロケーションされている**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-144">**Co-located at a cloud exchange**.</span></span> <span data-ttu-id="d66ac-145">クラウド エクスチェンジがある施設に併置されている場合、併置プロバイダーのイーサネット エクスチェンジ経由で Azure に仮想交差接続を要請できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-145">If you're co-located in a facility with a cloud exchange, you can order virtual cross-connections to Azure through the co-location provider’s Ethernet exchange.</span></span> <span data-ttu-id="d66ac-146">併置プロバイダーは、共有施設のインフラストラクチャと Azure の間に、レイヤー 2 交差接続と管理レイヤー 3 交差接続のいずれかを提供します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-146">Co-location providers can offer either layer 2 cross-connections, or managed layer 3 cross-connections between your infrastructure in the co-location facility and Azure.</span></span>
* <span data-ttu-id="d66ac-147">**ポイント ツー ポイントのイーサネット接続**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-147">**Point-to-point Ethernet connections**.</span></span> <span data-ttu-id="d66ac-148">オンプレミス データセンター/オフィスと Azure をポイント ツー ポイントのイーサネット リンクで接続できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-148">You can connect your on-premises datacenters/offices to Azure through point-to-point Ethernet links.</span></span> <span data-ttu-id="d66ac-149">ポイント ツー ポイントのイーサネットのプロバイダーは、サイトと Azure の間にレイヤー 2 接続と管理レイヤー 3 接続のいずれかを提供できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-149">Point-to-point Ethernet providers can offer layer 2 connections, or managed layer 3 connections between your site and Azure.</span></span>
* <span data-ttu-id="d66ac-150">**任意の環境間 (IPVPN) ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="d66ac-150">**Any-to-any (IPVPN) networks**.</span></span> <span data-ttu-id="d66ac-151">ワイド エリア ネットワーク (WAN) を Azure と統合できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-151">You can integrate your wide area network (WAN) with Azure.</span></span> <span data-ttu-id="d66ac-152">インターネット プロトコル仮想プライベート ネットワーク (IPVPN) プロバイダー (通常、マルチプロトコル ラベル スイッチング VPN) は、ブランチ オフィスとデータセンターの間に任意の環境間の接続を提供します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-152">Internet protocol virtual private network (IPVPN) providers (typically a multiprotocol label switching VPN) offer any-to-any connectivity between your branch offices and datacenters.</span></span> <span data-ttu-id="d66ac-153">Azure をご使用の WAN に相互接続し、ブランチ オフィスのように見せることができます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-153">Azure can be interconnected to your WAN to make it look just like any other branch office.</span></span> <span data-ttu-id="d66ac-154">通常、WAN プロバイダーは管理レイヤー 3 接続を提供します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-154">WAN providers typically offer managed layer 3 connectivity.</span></span>

<span data-ttu-id="d66ac-155">接続プロバイダーの詳細については、[ExpressRoute の概要][expressroute-introduction]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-155">For more information about connectivity providers, see the [ExpressRoute introduction][expressroute-introduction].</span></span>

### <a name="expressroute-circuit"></a><span data-ttu-id="d66ac-156">ExpressRoute 回線</span><span class="sxs-lookup"><span data-stu-id="d66ac-156">ExpressRoute circuit</span></span>

<span data-ttu-id="d66ac-157">Azure に接続するための [ExpressRoute 前提条件][expressroute-prereqs]を組織が満たしていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-157">Ensure that your organization has met the [ExpressRoute prerequisite requirements][expressroute-prereqs] for connecting to Azure.</span></span>

<span data-ttu-id="d66ac-158">`GatewaySubnet` という名前のサブネットを Azure VNet にまだ追加していない場合は追加し、Azure VPN ゲートウェイ サービスを使用して、ExpressRoute 仮想ネットワーク ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-158">If you haven't already done so, add a subnet named `GatewaySubnet` to your Azure VNet and create an ExpressRoute virtual network gateway using the Azure VPN gateway service.</span></span> <span data-ttu-id="d66ac-159">このプロセスの詳細については、「[回線のプロビジョニングと回線の状態の ExpressRoute ワークフロー][ExpressRoute-provisioning]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-159">For more information about this process, see [ExpressRoute workflows for circuit provisioning and circuit states][ExpressRoute-provisioning].</span></span>

<span data-ttu-id="d66ac-160">次のように、ExpressRoute 回線を作成します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-160">Create an ExpressRoute circuit as follows:</span></span>

1. <span data-ttu-id="d66ac-161">次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-161">Run the following PowerShell command:</span></span>
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. <span data-ttu-id="d66ac-162">新しい回線の `ServiceKey` をサービス プロバイダーに送信します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-162">Send the `ServiceKey` for the new circuit to the service provider.</span></span>

3. <span data-ttu-id="d66ac-163">プロバイダーによって回線がプロビジョニングされるのを待ちます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-163">Wait for the provider to provision the circuit.</span></span> <span data-ttu-id="d66ac-164">回線のプロビジョニング状態を確認するには、次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-164">To verify the provisioning state of a circuit, run the following PowerShell command:</span></span>
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="d66ac-165">回線の準備ができたら、出力の `Service Provider` セクションの `Provisioning state` フィールドが `NotProvisioned` から `Provisioned` に変わります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-165">The `Provisioning state` field in the `Service Provider` section of the output will change from `NotProvisioned` to `Provisioned` when the circuit is ready.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d66ac-166">レイヤー 3 接続を使用している場合、ルーティングはプロバイダーによって構成および管理されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-166">If you're using a layer 3 connection, the provider should configure and manage routing for you.</span></span> <span data-ttu-id="d66ac-167">プロバイダーが適切なルートを実装できるように、必要な情報を提供します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-167">You provide the information necessary to enable the provider to implement the appropriate routes.</span></span>
    > 
    > 

4. <span data-ttu-id="d66ac-168">レイヤー 2 接続を使用している場合:</span><span class="sxs-lookup"><span data-stu-id="d66ac-168">If you're using a layer 2 connection:</span></span>

    1. <span data-ttu-id="d66ac-169">実装するピアリングの種類ごとに、有効なパブリック IP アドレスで構成される 2 つの /30 サブネットを予約します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-169">Reserve two /30 subnets composed of valid public IP addresses for each type of peering you want to implement.</span></span> <span data-ttu-id="d66ac-170">これらの /30 サブネットは、回線で使用されるルーターに IP アドレスを提供するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-170">These /30 subnets will be used to provide IP addresses for the routers used for the circuit.</span></span> <span data-ttu-id="d66ac-171">プライベート、パブリック、および Microsoft ピアリングを実装する場合は、有効なパブリック IP アドレスを持つ 6 つの /30 サブネットが必要になります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-171">If you are implementing private, public, and Microsoft peering, you'll need 6 /30 subnets with valid public IP addresses.</span></span>     

    2. <span data-ttu-id="d66ac-172">ExpressRoute 回線用のルーティングを構成します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-172">Configure routing for the ExpressRoute circuit.</span></span> <span data-ttu-id="d66ac-173">構成するピアリングの種類 (プライベート、パブリック、および Microsoft) ごとに、次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-173">Run the following PowerShell commands for each type of peering you want to configure (private, public, and Microsoft).</span></span> <span data-ttu-id="d66ac-174">詳細については、[ExpressRoute 回線のルーティングの作成と変更][configure-expressroute-routing]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-174">For more information, see [Create and modify routing for an ExpressRoute circuit][configure-expressroute-routing].</span></span>
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. <span data-ttu-id="d66ac-175">パブリックおよび Microsoft ピアリングのネットワーク アドレス変換 (NAT) で使用するために、有効なパブリック IP アドレスのプールをもう 1 つ予約します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-175">Reserve another pool of valid public IP addresses to use for network address translation (NAT) for public and Microsoft peering.</span></span> <span data-ttu-id="d66ac-176">ピアリングごとに異なるプールを指定することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-176">It is recommended to have a different pool for each peering.</span></span> <span data-ttu-id="d66ac-177">接続プロバイダーに対してそのプールを指定します。これにより、プロバイダーはこれらの範囲の境界ゲートウェイ プロトコル (BGP) アドバタイズを構成できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-177">Specify the pool to your connectivity provider, so they can configure border gateway protocol (BGP) advertisements for those ranges.</span></span>

5. <span data-ttu-id="d66ac-178">次の PowerShell コマンドを実行して、プライベート VNet を ExpressRoute 回線にリンクします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-178">Run the following PowerShell commands to link your private VNet(s) to the ExpressRoute circuit.</span></span> <span data-ttu-id="d66ac-179">詳細については、[ExpressRoute 回線への仮想ネットワークのリンク][link-vnet-to-expressroute]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-179">For more information,see [Link a virtual network to an ExpressRoute circuit][link-vnet-to-expressroute].</span></span>

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

<span data-ttu-id="d66ac-180">すべての VNet と ExpressRoute 回線が、同じ地政学的リージョンに配置されている場合は、さまざまなリージョンにある複数の VNet を同じ ExpressRoute 回線に接続できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-180">You can connect multiple VNets located in different regions to the same ExpressRoute circuit, as long as all VNets and the ExpressRoute circuit are located within the same geopolitical region.</span></span>

### <a name="troubleshooting"></a><span data-ttu-id="d66ac-181">トラブルシューティング</span><span class="sxs-lookup"><span data-stu-id="d66ac-181">Troubleshooting</span></span> 

<span data-ttu-id="d66ac-182">オンプレミスまたはプライベート VNet 内で構成を変更していないにもかかわらず、以前に機能していた ExpressRoute 回線が接続できなくなった場合、接続プロバイダーに連絡し、協力して問題解決にあたらなければならない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-182">If a previously functioning ExpressRoute circuit now fails to connect, in the absence of any configuration changes on-premises or within your private VNet, you may need to contact the connectivity provider and work with them to correct the issue.</span></span> <span data-ttu-id="d66ac-183">次の PowerShell コマンドを実行して、ExpressRoute 回線がプロビジョニングされていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-183">Use the following Powershell commands to verify that the ExpressRoute circuit has been provisioned:</span></span>

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="d66ac-184">以下のように、このコマンドの出力には、`ProvisioningState`、`CircuitProvisioningState`、`ServiceProviderProvisioningState` など、回線のプロパティがいくつか示されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-184">The output of this command shows several properties for your circuit, including `ProvisioningState`, `CircuitProvisioningState`, and `ServiceProviderProvisioningState` as shown below.</span></span>

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

<span data-ttu-id="d66ac-185">新しい回線を作成しようとした後に、`ProvisioningState` が `Succeeded` に設定されていない場合は、次のコマンドを使用して回線を削除し、もう一度作成してみてください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-185">If the `ProvisioningState` is not set to `Succeeded` after you tried to create a new circuit, remove the circuit by using the command below and try to create it again.</span></span>

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="d66ac-186">プロバイダーによって既に回線がプロビジョニング済みであるのに、`ProvisioningState` が `Failed` に設定されているか、`CircuitProvisioningState` が `Enabled` でない場合は、対処方法についてプロバイダーにお問い合わせください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-186">If your provider had already provisioned the circuit, and the `ProvisioningState` is set to `Failed`, or the `CircuitProvisioningState` is not `Enabled`, contact your provider for further assistance.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d66ac-187">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d66ac-187">Scalability considerations</span></span>

<span data-ttu-id="d66ac-188">ExpressRoute 回線が提供するのは、ネットワーク間の高帯域幅のパスです。</span><span class="sxs-lookup"><span data-stu-id="d66ac-188">ExpressRoute circuits provide a high bandwidth path between networks.</span></span> <span data-ttu-id="d66ac-189">一般的に、高帯域幅であるほど、コストが高くなります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-189">Generally, the higher the bandwidth the greater the cost.</span></span> 

<span data-ttu-id="d66ac-190">ExpressRoute には、従量制課金プランと無制限データ プランの 2 つの[価格プラン][expressroute-pricing]が用意されています。</span><span class="sxs-lookup"><span data-stu-id="d66ac-190">ExpressRoute offers two [pricing plans][expressroute-pricing] to customers, a metered plan and an unlimited data plan.</span></span> <span data-ttu-id="d66ac-191">料金は、回線の帯域幅によって異なります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-191">Charges vary according to circuit bandwidth.</span></span> <span data-ttu-id="d66ac-192">使用できる帯域幅は、プロバイダーによって異なる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-192">Available bandwidth will likely vary from provider to provider.</span></span> <span data-ttu-id="d66ac-193">`Get-AzureRmExpressRouteServiceProvider` コマンドレットを使用すると、リージョンで使用できるプロバイダーと、そのプロバイダーが提供する帯域幅を確認できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-193">Use the `Get-AzureRmExpressRouteServiceProvider` cmdlet to see the providers available in your region and the bandwidths that they offer.</span></span>
 
<span data-ttu-id="d66ac-194">1 つの ExpressRoute 回線が、特定の数のピアリングと VNet リンクをサポートします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-194">A single ExpressRoute circuit can support a certain number of peerings and VNet links.</span></span> <span data-ttu-id="d66ac-195">詳細については、[ExpressRoute の制限](/azure/azure-subscription-service-limits)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-195">See [ExpressRoute limits](/azure/azure-subscription-service-limits) for more information.</span></span>

<span data-ttu-id="d66ac-196">追加料金を支払うことで、ExpressRoute Premium アドオンでいくつかの追加機能を利用できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-196">For an extra charge, the ExpressRoute Premium add-on provides some additional capability:</span></span>

* <span data-ttu-id="d66ac-197">パブリックおよびプライベート ピアリングのルート制限の増加。</span><span class="sxs-lookup"><span data-stu-id="d66ac-197">Increased route limits for public and private peering.</span></span> 
* <span data-ttu-id="d66ac-198">ExpressRoute 回線あたりの VNet リンク数の増加。</span><span class="sxs-lookup"><span data-stu-id="d66ac-198">Increased number of VNet links per ExpressRoute circuit.</span></span> 
* <span data-ttu-id="d66ac-199">サービスのグローバル接続。</span><span class="sxs-lookup"><span data-stu-id="d66ac-199">Global connectivity for services.</span></span>

<span data-ttu-id="d66ac-200">詳細については、「[ExpressRoute の価格][expressroute-pricing]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-200">See [ExpressRoute pricing][expressroute-pricing] for details.</span></span> 

<span data-ttu-id="d66ac-201">ExpressRoute 回線は、購入した帯域幅制限の 2 倍までの一時的ネットワーク バーストを無料で許容できるように設計されています。</span><span class="sxs-lookup"><span data-stu-id="d66ac-201">ExpressRoute circuits are designed to allow temporary network bursts up to two times the bandwidth limit that you procured for no additional cost.</span></span> <span data-ttu-id="d66ac-202">これは、冗長なリンクを使用することで実現します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-202">This is achieved by using redundant links.</span></span> <span data-ttu-id="d66ac-203">ただし、すべての接続プロバイダーが、この機能をサポートしているわけではありません。</span><span class="sxs-lookup"><span data-stu-id="d66ac-203">However, not all connectivity providers support this feature.</span></span> <span data-ttu-id="d66ac-204">この機能を使用する前に、接続プロバイダーによってこの機能が有効になっていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-204">Verify that your connectivity provider enables this feature before depending on it.</span></span>

<span data-ttu-id="d66ac-205">プロバイダーによっては帯域幅を変更できる場合もありますが、必ずニーズを上回る初期帯域幅を選択し、拡大に対応できるだけの余地を確保してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-205">Although some providers allow you to change your bandwidth, make sure you pick an initial bandwidth that surpasses your needs and provides room for growth.</span></span> <span data-ttu-id="d66ac-206">今後、帯域幅を増やす必要が出てきたとき、選択できるオプションは 2 つあります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-206">If you need to increase bandwidth in the future, you are left with two options:</span></span>

- <span data-ttu-id="d66ac-207">帯域幅を増やします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-207">Increase the bandwidth.</span></span> <span data-ttu-id="d66ac-208">このオプションはできるだけ避けてください。一部のプロバイダーでは、帯域幅を動的に拡大できません。</span><span class="sxs-lookup"><span data-stu-id="d66ac-208">You should avoid this option as much as possible, and not all providers allow you to increase bandwidth dynamically.</span></span> <span data-ttu-id="d66ac-209">それでも帯域幅を拡大する必要がある場合は、プロバイダーに連絡して、Powershell コマンドによる ExpressRoute 帯域幅プロパティの変更をサポートしていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-209">But if a bandwidth increase is needed, check with your provider to verify they support changing ExpressRoute bandwidth properties via Powershell commands.</span></span> <span data-ttu-id="d66ac-210">対応している場合は、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-210">If they do, run the commands below.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    <span data-ttu-id="d66ac-211">接続状態を維持したまま帯域幅を増やすことができます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-211">You can increase the bandwidth without loss of connectivity.</span></span> <span data-ttu-id="d66ac-212">帯域幅をダウン グレードすると、接続が中断されます。これは、回線を削除し、新しい構成で再作成する必要があるためです。</span><span class="sxs-lookup"><span data-stu-id="d66ac-212">Downgrading the bandwidth will result in disruption in connectivity, because you must delete the circuit and recreate it with the new configuration.</span></span>

- <span data-ttu-id="d66ac-213">価格プランを変更するか、Premium にアップグレードします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-213">Change your pricing plan and/or upgrade to Premium.</span></span> <span data-ttu-id="d66ac-214">これを行うには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-214">To do so, run the following commands.</span></span> <span data-ttu-id="d66ac-215">`Sku.Tier` プロパティには `Standard` または `Premium` を指定できます。`Sku.Name` プロパティには `MeteredData` または `UnlimitedData` を指定できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-215">The `Sku.Tier` property can be `Standard` or `Premium`; the `Sku.Name` property can be `MeteredData` or `UnlimitedData`.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="d66ac-216">`Sku.Name` プロパティが `Sku.Tier` および `Sku.Family` と一致することを確認します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-216">Make sure the `Sku.Name` property matches the `Sku.Tier` and `Sku.Family`.</span></span> <span data-ttu-id="d66ac-217">ファミリとレベルを変更しても、名前を変更しないと、接続が無効になります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-217">If you change the family and tier, but not the name, your connection will be disabled.</span></span>
    > 
    > 

    <span data-ttu-id="d66ac-218">SKU は中断なしでアップグレードできますが、無制限の価格プランを従量制課金に切り替えることはできません。</span><span class="sxs-lookup"><span data-stu-id="d66ac-218">You can upgrade the SKU without disruption, but you cannot switch from the unlimited pricing plan to metered.</span></span> <span data-ttu-id="d66ac-219">SKU をダウングレードする場合、帯域幅の消費は、標準 SKU の既定の制限内に維持する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-219">When downgrading the SKU, your bandwidth consumption must remain within the default limit of the standard SKU.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="d66ac-220">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d66ac-220">Availability considerations</span></span>

<span data-ttu-id="d66ac-221">ExpressRoute では、高可用性を実装するための、ホット スタンバイ ルーティング プロトコル (HSRP) や仮想ルーター冗長性プロトコル (VRRP) などのルーター冗長性プロトコルがサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="d66ac-221">ExpressRoute does not support router redundancy protocols such as hot standby routing protocol (HSRP) and virtual router redundancy protocol (VRRP) to implement high availability.</span></span> <span data-ttu-id="d66ac-222">代わりに、ピアリングごとに BGP セッションの冗長ペアが使用されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-222">Instead, it uses a redundant pair of BGP sessions per peering.</span></span> <span data-ttu-id="d66ac-223">ネットワークへの高可用性接続を容易にするために、Azure では、アクティブ/アクティブ構成の 2 つのルーター (Microsoft Edge の一部) に 2 つの冗長ポートがプロビジョニングされます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-223">To facilitate highly-available connections to your network, Azure provisions you with two redundant ports on two routers (part of the Microsoft edge) in an active-active configuration.</span></span>

<span data-ttu-id="d66ac-224">既定では、BGP セッションでは、60 秒のアイドル タイムアウト値を使用します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-224">By default, BGP sessions use an idle timeout value of 60 seconds.</span></span> <span data-ttu-id="d66ac-225">セッションが 3 回 (合計 180 秒) タイムアウトすると、ルーターは利用不可とマークされ、すべてのトラフィックが残りのルーターにリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-225">If a session times out three times (180 seconds total), the router is marked as unavailable, and all traffic is redirected to the remaining router.</span></span> <span data-ttu-id="d66ac-226">この 180 秒のタイムアウトは、クリティカルなアプリケーションにとっては長すぎる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-226">This 180-second timeout might be too long for critical applications.</span></span> <span data-ttu-id="d66ac-227">その場合は、オンプレミスのルーターで、BGP のタイムアウト設定をより小さな値に変更できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-227">If so, you can change your BGP time-out settings on the on-premises router to a smaller value.</span></span>

<span data-ttu-id="d66ac-228">Azure 接続の高可用性は、ご使用のプロバイダーの種類と、構成する ExpressRoute 回線および仮想ネットワーク ゲートウェイ接続の数に応じて、さまざまな方法で構成できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-228">You can configure high availability for your Azure connection in different ways, depending on the type of provider you use, and the number of ExpressRoute circuits and virtual network gateway connections you're willing to configure.</span></span> <span data-ttu-id="d66ac-229">可用性オプションを次にまとめます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-229">The following summarizes your availability options:</span></span>

* <span data-ttu-id="d66ac-230">レイヤー 2 接続を使用している場合は、アクティブ/アクティブ構成でオンプレミス ネットワークに冗長ルーターを配置します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-230">If you're using a layer 2 connection, deploy redundant routers in your on-premises network in an active-active configuration.</span></span> <span data-ttu-id="d66ac-231">プライマリ回線を一方のルーターに接続し、セカンダリ回線をもう一方に接続します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-231">Connect the primary circuit to one router, and the secondary circuit to the other.</span></span> <span data-ttu-id="d66ac-232">これにより、接続の両端で高可用性接続が実現します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-232">This will give you a highly available connection at both ends of the connection.</span></span> <span data-ttu-id="d66ac-233">これが必要なのは、ExpressRoute サービス レベル アグリーメント (SLA) を必要とする場合です。</span><span class="sxs-lookup"><span data-stu-id="d66ac-233">This is necessary if you require the ExpressRoute service level agreement (SLA).</span></span> <span data-ttu-id="d66ac-234">詳細については、[Azure ExpressRoute の SLA][sla-for-expressroute] に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-234">See [SLA for Azure ExpressRoute][sla-for-expressroute] for details.</span></span>

    <span data-ttu-id="d66ac-235">次の図は、冗長オンプレミス ルーターがプライマリ回線とセカンダリ回線に接続されている構成を示しています。</span><span class="sxs-lookup"><span data-stu-id="d66ac-235">The following diagram shows a configuration with redundant on-premises routers connected to the primary and secondary circuits.</span></span> <span data-ttu-id="d66ac-236">各回線によってパブリック ピアリングとプライベート ピアリングのトラフィックが処理されます (前のセクションで説明したように、各ピアリングに /30 アドレス空間のペアが指定されています)。</span><span class="sxs-lookup"><span data-stu-id="d66ac-236">Each circuit handles the traffic for a public peering and a private peering (each peering is designated a pair of /30 address spaces, as described in the previous section).</span></span>

    <span data-ttu-id="d66ac-237">![[1]][1]</span><span class="sxs-lookup"><span data-stu-id="d66ac-237">![[1]][1]</span></span>

* <span data-ttu-id="d66ac-238">レイヤー 3 接続を使用している場合は、自動的に可用性を処理する冗長 BGP セッションが提供されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-238">If you're using a layer 3 connection, verify that it provides redundant BGP sessions that handle availability for you.</span></span>

* <span data-ttu-id="d66ac-239">VNet を、さまざまなサービス プロバイダーによって提供される、複数の ExpressRoute 回線に接続します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-239">Connect the VNet to multiple ExpressRoute circuits, supplied by different service providers.</span></span> <span data-ttu-id="d66ac-240">この戦略により、追加の高可用性機能と、ディザスター リカバリー機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-240">This strategy provides additional high-availability and disaster recovery capabilities.</span></span>

* <span data-ttu-id="d66ac-241">ExpressRoute のフェールオーバー パスとしてサイト間 VPN を構成します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-241">Configure a site-to-site VPN as a failover path for ExpressRoute.</span></span> <span data-ttu-id="d66ac-242">このオプションの詳細については、「[VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する][highly-available-network-architecture]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-242">For more about this option, see [Connect an on-premises network to Azure using ExpressRoute with VPN failover][highly-available-network-architecture].</span></span>
 <span data-ttu-id="d66ac-243">このオプションは、プライベート ピアリングにのみ適用されます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-243">This option only applies to private peering.</span></span> <span data-ttu-id="d66ac-244">Azure と Office 365 サービスについては、インターネットが唯一のフェールオーバー パスです。</span><span class="sxs-lookup"><span data-stu-id="d66ac-244">For Azure and Office 365 services, the Internet is the only failover path.</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="d66ac-245">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d66ac-245">Manageability considerations</span></span>

<span data-ttu-id="d66ac-246">[Azure Connectivity Toolkit (AzureCT)][azurect] を使用すると、オンプレミスのデータセンターと Azure の間の接続を監視できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-246">You can use the [Azure Connectivity Toolkit (AzureCT)][azurect] to monitor connectivity between your on-premises datacenter and Azure.</span></span> 

## <a name="security-considerations"></a><span data-ttu-id="d66ac-247">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d66ac-247">Security considerations</span></span>

<span data-ttu-id="d66ac-248">Azure 接続のセキュリティ オプションは、セキュリティ上の懸念事項とコンプライアンス ニーズに応じて、さまざまな方法で構成できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-248">You can configure security options for your Azure connection in different ways, depending on your security concerns and compliance needs.</span></span> 

<span data-ttu-id="d66ac-249">ExpressRoute はレイヤー 3 で動作します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-249">ExpressRoute operates in layer 3.</span></span> <span data-ttu-id="d66ac-250">アプリケーション レイヤーの脅威を防止するには、トラフィックを正当なリソースに制限するネットワーク セキュリティ アプライアンスを使用します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-250">Threats in the application layer can be prevented by using a network security appliance that restricts traffic to legitimate resources.</span></span> <span data-ttu-id="d66ac-251">さらに、パブリック ピアリングを使用した ExpressRoute 接続は、オンプレミスからのみ開始できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-251">Additionally, ExpressRoute connections using public peering can only be initiated from on-premises.</span></span> <span data-ttu-id="d66ac-252">これにより、不正なサービスが、インターネットからオンプレミス データにアクセスして侵害するのを防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-252">This prevents a rogue service from accessing and compromising on-premises data from the Internet.</span></span>

<span data-ttu-id="d66ac-253">セキュリティを最大限に高めるには、オンプレミス ネットワークとプロバイダー エッジ ルーターの間に、ネットワーク セキュリティ アプライアンスを追加します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-253">To maximize security, add network security appliances between the on-premises network and the provider edge routers.</span></span> <span data-ttu-id="d66ac-254">これは、VNet から未承認トラフィックが流れ込んでくるのを制限するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-254">This will help to restrict the inflow of unauthorized traffic from the VNet:</span></span>

<span data-ttu-id="d66ac-255">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="d66ac-255">![[2]][2]</span></span>

<span data-ttu-id="d66ac-256">監査やコンプライアンスの目的で、VNet で実行されているコンポーネントからインターネットへの直接アクセスを禁止し、[強制トンネリング][forced-tuneling]を実装しなければならない場合があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-256">For auditing or compliance purposes, it may be necessary to prohibit direct access from components running in the VNet to the Internet and implement [forced tunneling][forced-tuneling].</span></span> <span data-ttu-id="d66ac-257">このような場合は、インターネット トラフィックを、オンプレミスで実行されているプロキシにリダイレクトして、監査できるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-257">In this situation, Internet traffic should be redirected back through a proxy running  on-premises where it can be audited.</span></span> <span data-ttu-id="d66ac-258">プロキシは、承認されていないトラフィックが流出するのをブロックし、悪意のある可能性がある受信トラフィックをフィルター処理するように構成できます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-258">The proxy can be configured to block unauthorized traffic flowing out, and filter potentially malicious inbound traffic.</span></span>

<span data-ttu-id="d66ac-259">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="d66ac-259">![[3]][3]</span></span>

<span data-ttu-id="d66ac-260">セキュリティを最大限に高めるためには、VM のパブリック IP アドレスは有効にしないでください。また、NSG を使用して、こうした VM にはパブリックにアクセスできないようにします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-260">To maximize security, do not enable a public IP address for your VMs, and use NSGs to ensure that these VMs aren't publicly accessible.</span></span> <span data-ttu-id="d66ac-261">VM は、内部 IP アドレスでのみ使用できるようにします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-261">VMs should only be available using the internal IP address.</span></span> <span data-ttu-id="d66ac-262">こうしたアドレスは ExpressRoute ネットワークを介してアクセス可能にできます。これにより、オンプレミスの DevOps スタッフが、構成やメンテナンスを行うことができます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-262">These addresses can be made accessible through the ExpressRoute network, enabling on-premises DevOps staff to perform configuration or maintenance.</span></span>

<span data-ttu-id="d66ac-263">VM の管理エンドポイントを外部ネットワークに公開する必要がある場合は、NSG またはアクセス制御リストを使用して、こうしたポートの可視性を、IP アドレスまたはネットワークのホワイトリストに制限します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-263">If you must expose management endpoints for VMs to an external network, use NSGs or access control lists to restrict the visibility of these ports to a whitelist of IP addresses or networks.</span></span>

> [!NOTE]
> <span data-ttu-id="d66ac-264">既定では、Azure Portal でデプロイされた Azure VM には、ログイン アクセスを提供するパブリック IP アドレスが含まれます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-264">By default, Azure VMs deployed through the Azure portal include a public IP address that provides login access.</span></span>  
> 
> 


## <a name="deploy-the-solution"></a><span data-ttu-id="d66ac-265">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="d66ac-265">Deploy the solution</span></span>

<span data-ttu-id="d66ac-266">**前提条件。**</span><span class="sxs-lookup"><span data-stu-id="d66ac-266">**Prequisites.**</span></span> <span data-ttu-id="d66ac-267">既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="d66ac-267">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="d66ac-268">ソリューションをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-268">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="d66ac-269">下記のボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-269">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="d66ac-270">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="d66ac-270">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="d66ac-271">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-er-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-271">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="d66ac-272">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-272">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="d66ac-273">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-273">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="d66ac-274">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-274">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="d66ac-275">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-275">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="d66ac-276">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-276">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="d66ac-277">下記のボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-277">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="d66ac-278">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="d66ac-278">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="d66ac-279">**[リソース グループ]** セクションで **[既存のものを使用]** を選択し、テキスト ボックスに「`ra-hybrid-er-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-279">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="d66ac-280">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="d66ac-280">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="d66ac-281">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="d66ac-281">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="d66ac-282">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-282">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="d66ac-283">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="d66ac-283">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="d66ac-284">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="d66ac-284">Wait for the deployment to complete.</span></span>


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャ"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "冗長ルーターと、ExpressRoute のプライマリ回路とセカンダリ回路の使用"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "オンプレミス ネットワークへのセキュリティ デバイスの追加"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "強制トンネリングを使用したインターネットへのトラフィックの監査"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "ExpressRoute 回線の ServiceKey の検索"  