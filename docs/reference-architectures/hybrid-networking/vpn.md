---
title: VPN を使用した Azure へのオンプレミス ネットワークの接続
description: VPN を使用して接続された Azure 仮想ネットワークとオンプレミス ネットワークにまたがる、セキュリティで保護されたサイト間ネットワーク アーキテクチャの実装方法。
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: d0966791642ee174d020c960c9fc6e72d8768539
ms.sourcegitcommit: b38ba378c9d6110da2dfd50b4233fadd94604bb0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167440"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="1b321-103">VPN ゲートウェイを使用した Azure へのオンプレミス ネットワークの接続</span><span class="sxs-lookup"><span data-stu-id="1b321-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="1b321-104">この参照アーキテクチャでは、サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure に拡張する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="1b321-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="1b321-105">オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) 間のトラフィックは、IPSec VPN トンネルを介して行き来します。</span><span class="sxs-lookup"><span data-stu-id="1b321-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="1b321-106">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="1b321-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="1b321-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="1b321-107">![[0]][0]</span></span>

<span data-ttu-id="1b321-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="1b321-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="1b321-109">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="1b321-109">Architecture</span></span> 

<span data-ttu-id="1b321-110">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="1b321-111">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="1b321-111">**On-premises network**.</span></span> <span data-ttu-id="1b321-112">組織内で運用されているプライベートのローカル エリア ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="1b321-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="1b321-113">**VPN アプライアンス**。</span><span class="sxs-lookup"><span data-stu-id="1b321-113">**VPN appliance**.</span></span> <span data-ttu-id="1b321-114">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービス。</span><span class="sxs-lookup"><span data-stu-id="1b321-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="1b321-115">VPN アプライアンスは、ハードウェア デバイスである場合や、ソフトウェア ソリューション (Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) など) である場合があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="1b321-116">サポートされている VPN アプライアンスの一覧と、Azure VPN ゲートウェイに接続するためのその構成については、[サイト間 VPN Gateway 接続用の VPN デバイス][vpn-appliance]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="1b321-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="1b321-117">**仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="1b321-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="1b321-118">Azure VPN ゲートウェイのクラウド アプリケーションとコンポーネントは同じ [VNet][azure-virtual-network] に存在します。</span><span class="sxs-lookup"><span data-stu-id="1b321-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="1b321-119">**Azure VPN ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="1b321-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="1b321-120">[VPN ゲートウェイ][azure-vpn-gateway] サービスを使用すると、VNet を VPN アプライアンスを介してオンプレミス ネットワークに接続できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="1b321-121">詳細については、「[オンプレミス ネットワークを Microsoft Azure 仮想ネットワークに接続する][connect-to-an-Azure-vnet]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="1b321-122">VPN ゲートウェイには、次の要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="1b321-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="1b321-123">**仮想ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="1b321-123">**Virtual network gateway**.</span></span> <span data-ttu-id="1b321-124">VNet の仮想 VPN アプライアンスを提供するリソース。</span><span class="sxs-lookup"><span data-stu-id="1b321-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="1b321-125">オンプレミス ネットワークから VNet へのトラフィックのルーティングを行います。</span><span class="sxs-lookup"><span data-stu-id="1b321-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="1b321-126">**ローカル ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="1b321-126">**Local network gateway**.</span></span> <span data-ttu-id="1b321-127">オンプレミス VPN アプライアンスの抽象化。</span><span class="sxs-lookup"><span data-stu-id="1b321-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="1b321-128">クラウド アプリケーションからオンプレミス ネットワークへのネットワーク トラフィックは、このゲートウェイを介してルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="1b321-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="1b321-129">**接続**。</span><span class="sxs-lookup"><span data-stu-id="1b321-129">**Connection**.</span></span> <span data-ttu-id="1b321-130">この接続には、接続の種類 (IPSec) を指定するプロパティと、オンプレミス VPN アプライアンスとの共有キー (トラフィックの暗号化用) を指定するプロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="1b321-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="1b321-131">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="1b321-131">**Gateway subnet**.</span></span> <span data-ttu-id="1b321-132">仮想ネットワーク ゲートウェイは独自のサブネットで保持され、以下の推奨事項セクションで説明されているさまざまな要件の対象となります。</span><span class="sxs-lookup"><span data-stu-id="1b321-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="1b321-133">**クラウド アプリケーション**。</span><span class="sxs-lookup"><span data-stu-id="1b321-133">**Cloud application**.</span></span> <span data-ttu-id="1b321-134">Azure でホストされるアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="1b321-134">The application hosted in Azure.</span></span> <span data-ttu-id="1b321-135">Azure ロード バランサーを介して接続された複数のサブネットを持つ、複数の層が含まれる場合があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="1b321-136">アプリケーション インフラストラクチャの詳細については、「[Windows VM ワークロードの実行][windows-vm-ra]」と「[Linux VM ワークロードの実行][linux-vm-ra]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="1b321-137">**内部ロード バランサー**: </span><span class="sxs-lookup"><span data-stu-id="1b321-137">**Internal load balancer**.</span></span> <span data-ttu-id="1b321-138">VPN ゲートウェイからのネットワーク トラフィックは、内部ロード バランサーを介してクラウド アプリケーションにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="1b321-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="1b321-139">ロード バランサーは、アプリケーションのフロントエンドのサブネットに配置されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="1b321-140">Recommendations</span><span class="sxs-lookup"><span data-stu-id="1b321-140">Recommendations</span></span>

<span data-ttu-id="1b321-141">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="1b321-142">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="1b321-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="1b321-143">VNet とゲートウェイ サブネット</span><span class="sxs-lookup"><span data-stu-id="1b321-143">VNet and gateway subnet</span></span>

<span data-ttu-id="1b321-144">必要なすべてのリソースにとって十分な大きさのアドレス空間を持つ Azure VNet を作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="1b321-145">追加の VM が今後必要になると思われる場合は、拡大に対応できるだけの領域を VNet アドレス空間に確保します。</span><span class="sxs-lookup"><span data-stu-id="1b321-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="1b321-146">VNet のアドレス空間は、オンプレミス ネットワークと重複させることはできません。</span><span class="sxs-lookup"><span data-stu-id="1b321-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="1b321-147">たとえば、上の図は、VNet に対してアドレス空間 10.20.0.0/16 を使用しています。</span><span class="sxs-lookup"><span data-stu-id="1b321-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="1b321-148">*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。</span><span class="sxs-lookup"><span data-stu-id="1b321-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="1b321-149">このサブネットは、仮想ネットワーク ゲートウェイで必要です。</span><span class="sxs-lookup"><span data-stu-id="1b321-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="1b321-150">このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="1b321-151">また、このサブネットは、アドレス空間の途中には配置しないでください。</span><span class="sxs-lookup"><span data-stu-id="1b321-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="1b321-152">ゲートウェイ サブネットのアドレス空間は、VNet アドレス空間の上端に設定することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="1b321-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="1b321-153">図の例では 10.20.255.224/27 を使用しています。</span><span class="sxs-lookup"><span data-stu-id="1b321-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="1b321-154">[CIDR] の簡単な計算手順を次に示します。</span><span class="sxs-lookup"><span data-stu-id="1b321-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="1b321-155">VNet のアドレス空間の変数ビットを、ゲートウェイ サブネットで使用されているビットまで 1 に設定し、残りのビットを 0 に設定します。</span><span class="sxs-lookup"><span data-stu-id="1b321-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="1b321-156">結果のビットを 10 進数に変換し、プレフィックス長をゲートウェイ サブネットのサイズに設定して、その 10 進数をアドレス空間として表現します。</span><span class="sxs-lookup"><span data-stu-id="1b321-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="1b321-157">たとえば、IP アドレス範囲が 10.20.0.0/16 の VNet については、上の手順 1. を適用すると、10.20.0b11111111.0b11100000 になります。</span><span class="sxs-lookup"><span data-stu-id="1b321-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="1b321-158">これを 10 進数に変換し、アドレス空間として表現すると、10.20.255.224/27 が生成されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="1b321-159">VM をゲートウェイ サブネットにデプロイしないでください。</span><span class="sxs-lookup"><span data-stu-id="1b321-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="1b321-160">また、NSG をこのサブネットに割り当てないでください。割り当てると、ゲートウェイが機能しなくなります。</span><span class="sxs-lookup"><span data-stu-id="1b321-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="1b321-161">仮想ネットワーク ゲートウェイ</span><span class="sxs-lookup"><span data-stu-id="1b321-161">Virtual network gateway</span></span>

<span data-ttu-id="1b321-162">仮想ネットワーク ゲートウェイにパブリック IP アドレスを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="1b321-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="1b321-163">ゲートウェイ サブネットで仮想ネットワーク ゲートウェイを作成し、新しく割り当てられたパブリック IP アドレスを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="1b321-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="1b321-164">要件に最も合致し、VPN アプライアンスによって有効になっているゲートウェイの種類を使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="1b321-165">アドレス プレフィックスなどのポリシー基準に基づいて、要求がどのようにルーティングされるかを厳密に制御する必要がある場合、[ポリシー ベースのゲートウェイ][policy-based-routing]を作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="1b321-166">ポリシー ベースのゲートウェイは、静的ルーティングを使用し、サイト間接続でのみ機能します。</span><span class="sxs-lookup"><span data-stu-id="1b321-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="1b321-167">RRAS を使用してオンプレミス ネットワークに接続する場合、マルチサイトまたはリージョン間接続をサポートする場合、または VNet 間接続 (複数の VNet を横断するルートを含む) を実装する場合、[ルート ベースのゲートウェイ][route-based-routing]を作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="1b321-168">ルート ベースのゲートウェイは、ネットワーク間でトラフィックを転送する動的ルーティングを使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="1b321-169">これは代替ルートを試すことができるため、静的ルートよりもネットワーク パスの障害に耐性があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="1b321-170">また、ルート ベースのゲートウェイでは、ネットワーク アドレスが変わったときに、ルートを手動で更新する必要がない可能性があるため、管理オーバーヘッドを削減できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="1b321-171">サポートされている VPN アプライアンスの一覧については、[サイト間 VPN Gateway 接続の VPN デバイス][vpn-appliances]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="1b321-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="1b321-172">ゲートウェイを作成したら、ゲートウェイを削除して再作成しない限り、ゲートウェイの種類を変更することはできません。</span><span class="sxs-lookup"><span data-stu-id="1b321-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="1b321-173">ご自分のスループット要件に最も合致する Azure VPN ゲートウェイ SKU を選択します。</span><span class="sxs-lookup"><span data-stu-id="1b321-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="1b321-174">詳細については、「[ゲートウェイの SKU][azure-gateway-skus]」を参照してください</span><span class="sxs-lookup"><span data-stu-id="1b321-174">For more informayion, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="1b321-175">Basic SKU は、Azure ExpressRoute と互換性がありません。</span><span class="sxs-lookup"><span data-stu-id="1b321-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="1b321-176">ゲートウェイの作成後、[SKU を変更][changing-SKUs]できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="1b321-177">料金は、ゲートウェイがプロビジョニングされ、利用可能になっている時間に基づいて発生します。</span><span class="sxs-lookup"><span data-stu-id="1b321-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="1b321-178">「[VPN Gateway の価格][azure-gateway-charges]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="1b321-179">要求がアプリケーション VM に直接渡されるようにするのではなく、アプリケーションの受信トラフィックをゲートウェイから内部ロード バランサーに転送する、ゲートウェイ サブネットのルーティング ルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="1b321-180">オンプレミス ネットワーク接続</span><span class="sxs-lookup"><span data-stu-id="1b321-180">On-premises network connection</span></span>

<span data-ttu-id="1b321-181">ローカル ネットワーク ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-181">Create a local network gateway.</span></span> <span data-ttu-id="1b321-182">オンプレミスの VPN アプライアンスのパブリック IP アドレスと、オンプレミス ネットワークのアドレス空間を指定します。</span><span class="sxs-lookup"><span data-stu-id="1b321-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="1b321-183">オンプレミス VPN アプライアンスには、Azure VPN Gateway のローカル ネットワーク ゲートウェイがアクセスできるパブリック IP アドレスが必要であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="1b321-184">VPN デバイスはネットワーク アドレス変換 (NAT) デバイスの背後には配置できません。</span><span class="sxs-lookup"><span data-stu-id="1b321-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="1b321-185">仮想ネットワーク ゲートウェイおよびローカル ネットワーク ゲートウェイのサイト間接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="1b321-186">サイト間 (IPSec) 接続の種類を選択し、共有キーを指定します。</span><span class="sxs-lookup"><span data-stu-id="1b321-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="1b321-187">Azure VPN ゲートウェイによるサイト間の暗号化は IPSec プロトコルに基づいており、認証に事前共有キーを使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="1b321-188">Azure VPN ゲートウェイを作成するときにキーを指定します。</span><span class="sxs-lookup"><span data-stu-id="1b321-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="1b321-189">同じキーを使用して、オンプレミスで実行する VPN アプライアンスを構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="1b321-190">他の認証メカニズムは現在サポートされていません。</span><span class="sxs-lookup"><span data-stu-id="1b321-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="1b321-191">Azure VNet のアドレスに向けた要求が VPN デバイスに転送されるように、オンプレミス ルーティング インフラストラクチャが構成されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="1b321-192">オンプレミス ネットワークのクラウド アプリケーションに必要なポートを開きます。</span><span class="sxs-lookup"><span data-stu-id="1b321-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="1b321-193">接続をテストして、次の点を確認してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-193">Test the connection to verify that:</span></span>

* <span data-ttu-id="1b321-194">トラフィックが Azure VPN ゲートウェイを介してクラウド アプリケーションに到達するように、オンプレミス VPN アプライアンスによって正しくルーティングしている。</span><span class="sxs-lookup"><span data-stu-id="1b321-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="1b321-195">トラフィックがオンプレミス ネットワークに戻るように、VNet によって正しくルーティングされている。</span><span class="sxs-lookup"><span data-stu-id="1b321-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="1b321-196">両方向の禁止されたトラフィックが、正しくブロックされている。</span><span class="sxs-lookup"><span data-stu-id="1b321-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="1b321-197">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="1b321-197">Scalability considerations</span></span>

<span data-ttu-id="1b321-198">垂直スケーラビリティを制限するには、Basic または Standard の VPN Gateway SKU から High Performance VPN SKU に移行します。</span><span class="sxs-lookup"><span data-stu-id="1b321-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="1b321-199">大量の VPN トラフィックが想定される VNet については、さまざまなワークロードを小さな個別の VNet に分散し、それぞれに対して VPN ゲートウェイを構成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="1b321-200">VNet は、水平方向または垂直方向にパーティション分割できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="1b321-201">水平方向にパーティション分割するには、各階層の VM インスタンスをいくつか、新しい VNet のサブネットに移動します。</span><span class="sxs-lookup"><span data-stu-id="1b321-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="1b321-202">これにより、各 VNet の構造と機能が同じになります。</span><span class="sxs-lookup"><span data-stu-id="1b321-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="1b321-203">垂直方向にパーティション分割する場合は、機能がさまざまな論理領域 (注文処理、請求、顧客アカウントの管理など) に分割されるように各階層を再設計します。</span><span class="sxs-lookup"><span data-stu-id="1b321-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="1b321-204">その後、各機能領域を独自の VNet に配置できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="1b321-205">VNet でオンプレミスの Active Directory ドメイン コントローラーをレプリケートし、VNet に DNS を実装すると、オンプレミスからクラウドにフローするセキュリティ関連および管理トラフィックの削減に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="1b321-206">詳細については、[Azure への Active Directory Domain Services (AD DS) の拡張][adds-extend-domain]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="1b321-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="1b321-207">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="1b321-207">Availability considerations</span></span>

<span data-ttu-id="1b321-208">オンプレミス ネットワークを、Azure VPN ゲートウェイで引き続き使用できるようにする必要がある場合は、オンプレミス VPN ゲートウェイのフェールオーバー クラスターを実装します。</span><span class="sxs-lookup"><span data-stu-id="1b321-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="1b321-209">組織に複数のオンプレミス サイトがある場合は、1 つ以上の Azure VNet への[マルチサイト接続][vpn-gateway-multi-site]を作成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="1b321-210">このアプローチには、動的 (ルート ベース) のルーティングが必要です。したがって、オンプレミス VPN ゲートウェイがこの機能をサポートしていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="1b321-211">サービス レベル アグリーメントの詳細については、「[VPN Gateway のSLA][sla-for-vpn-gateway]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="1b321-212">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="1b321-212">Manageability considerations</span></span>

<span data-ttu-id="1b321-213">オンプレミスの VPN アプライアンスからの診断情報を監視します。</span><span class="sxs-lookup"><span data-stu-id="1b321-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="1b321-214">このプロセスは、VPN アプライアンスが提供する機能によって異なります。</span><span class="sxs-lookup"><span data-stu-id="1b321-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="1b321-215">たとえば、Windows Server 2012 でルーティングとリモート アクセス サービスを使用している場合は、[RRAS ログ記録][rras-logging]です。</span><span class="sxs-lookup"><span data-stu-id="1b321-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="1b321-216">[Azure VPN ゲートウェイ診断][gateway-diagnostic-logs]を使用して、接続の問題に関する情報を取得します。</span><span class="sxs-lookup"><span data-stu-id="1b321-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="1b321-217">こうしたログを使用すると、接続要求の要求元と要求先、使用されたプロトコル、接続の確立方法 (または、試行が失敗した理由) などの情報を追跡できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="1b321-218">Azure Portal で使用できる監査ログを使用して、Azure VPN ゲートウェイの操作ログを監視します。</span><span class="sxs-lookup"><span data-stu-id="1b321-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="1b321-219">ローカル ネットワーク ゲートウェイ、Azure ネットワーク ゲートウェイ、および接続に対して個別のログを使用できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="1b321-220">この情報は、ゲートウェイに対する変更の追跡に使用でき、前に機能していたゲートウェイの動作が何らかの理由で停止した場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="1b321-221">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="1b321-221">![[2]][2]</span></span>

<span data-ttu-id="1b321-222">接続を監視し、接続エラー イベントを追跡します。</span><span class="sxs-lookup"><span data-stu-id="1b321-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="1b321-223">この情報は、[Nagios][nagios] などの監視パッケージを使用して取得し、レポートすることができます。</span><span class="sxs-lookup"><span data-stu-id="1b321-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="1b321-224">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="1b321-224">Security considerations</span></span>

<span data-ttu-id="1b321-225">VPN ゲートウェイごとに別の共有キーを生成します。</span><span class="sxs-lookup"><span data-stu-id="1b321-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="1b321-226">強力な共有キーを使用すると、ブルート フォース攻撃に対抗するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="1b321-227">現在、Azure VPN ゲートウェイのキーを事前共有するために、Azure Key Vault を使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="1b321-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="1b321-228">オンプレミスの VPN アプライアンスによって、[Azure VPN ゲートウェイと互換性のある][vpn-appliance-ipsec]暗号化方式が使用されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="1b321-229">ポリシー ベースのルーティングでは、Azure VPN ゲートウェイは、AES256、AES128、および 3DES 暗号化アルゴリズムをサポートしています。</span><span class="sxs-lookup"><span data-stu-id="1b321-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="1b321-230">ルート ベースのゲートウェイは、AES256 および 3DES をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="1b321-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="1b321-231">オンプレミス VPN アプライアンスが境界ネットワーク (DMZ) にあり、その境界ネットワークとインターネットの間にファイアウォールが存在する場合は、サイト間 VPN 接続を許可するために、[追加のファイアウォール規則][additional-firewall-rules]を構成しなければならない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="1b321-232">VNet のアプリケーションがインターネットにデータを送信する場合は、インターネットに向かうすべてのトラフィックがオンプレミス ネットワークを介してルーティングされるように、[強制トンネリングを実装][forced-tunneling]することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="1b321-233">このアプローチを使用すると、アプリケーションによって行われた送信要求をオンプレミス インフラストラクチャから監査できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="1b321-234">強制トンネリングは、Azure サービス (Storage サービスなど) と Windows ライセンス マネージャーへの接続に影響を与える場合があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="1b321-235">トラブルシューティング</span><span class="sxs-lookup"><span data-stu-id="1b321-235">Troubleshooting</span></span> 

<span data-ttu-id="1b321-236">一般的な VPN 関連エラーのトラブルシューティングに関する全般的な情報については、「[Troubleshooting common VPN related errors (一般的な VPN 関連エラーのトラブルシューティング)][troubleshooting-vpn-errors]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-236">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="1b321-237">次の推奨事項は、オンプレミス VPN アプライアンスが正しく機能しているかどうかを判断するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-237">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="1b321-238">**VPN アプライアンスによって生成されたログ ファイルで、エラーや障害がないかどうかを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-238">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="1b321-239">これは、VPN アプライアンスが正しく機能しているかどうかを判断するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-239">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="1b321-240">この情報の場所は、アプライアンスによって異なります。</span><span class="sxs-lookup"><span data-stu-id="1b321-240">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="1b321-241">たとえば、Windows Server 2012 で RRAS を使用している場合は、次の PowerShell コマンドを使用して、RRAS サービスのエラー イベント情報を表示できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-241">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="1b321-242">各エントリの *Message* プロパティにエラーの説明が示されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-242">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="1b321-243">一般的な例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="1b321-243">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="1b321-244">次の PowerShell コマンドを使用して、RRAS サービスを介した接続試行に関するイベント ログ情報を入手することもできます。</span><span class="sxs-lookup"><span data-stu-id="1b321-244">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="1b321-245">接続で障害が発生した場合、このログには、次のようなエラーが含まれます。</span><span class="sxs-lookup"><span data-stu-id="1b321-245">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="1b321-246">**VPN ゲートウェイを介した接続とルーティングを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-246">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="1b321-247">VPN アプライアンスによって、Azure VPN Gateway を介したトラフィックが正しくルーティングされていない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-247">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="1b321-248">[PsPing][psping] などのツールを使用して、VPN ゲートウェイを介した接続とルーティングを確認してください。</span><span class="sxs-lookup"><span data-stu-id="1b321-248">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="1b321-249">たとえば、オンプレミスのマシンから VNet にある Web サーバーへの接続をテストするには、次のコマンドを実行します (`<<web-server-address>>` を Web サーバーのアドレスで置き換えてください)。</span><span class="sxs-lookup"><span data-stu-id="1b321-249">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="1b321-250">オンプレミスのマシンがトラフィックを Web サーバーにルーティングできる場合は、次のような出力が表示されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-250">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="1b321-251">オンプレミスのマシンが指定した接続先と通信できない場合は、次のようなメッセージが表示されます。</span><span class="sxs-lookup"><span data-stu-id="1b321-251">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="1b321-252">**オンプレミスのファイアウォールで VPN トラフィックの通過が許可されていること、また正しいポートが開いていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-252">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="1b321-253">**オンプレミスの VPN アプライアンスによって、[Azure VPN ゲートウェイと互換性のある][vpn-appliance]暗号化方式が使用されていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-253">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="1b321-254">ポリシー ベースのルーティングでは、Azure VPN ゲートウェイは、AES256、AES128、および 3DES 暗号化アルゴリズムをサポートしています。</span><span class="sxs-lookup"><span data-stu-id="1b321-254">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="1b321-255">ルート ベースのゲートウェイは、AES256 および 3DES をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="1b321-255">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="1b321-256">次の推奨事項は、Azure VPN ゲートウェイに問題があるかどうかを判断するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-256">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="1b321-257">**[Azure VPN ゲートウェイの診断ログ][gateway-diagnostic-logs]で潜在的な問題がないかどうかを調べる。**</span><span class="sxs-lookup"><span data-stu-id="1b321-257">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="1b321-258">**Azure VPN ゲートウェイとオンプレミス VPN アプライアンスが同じ共有認証キーで構成されていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-258">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="1b321-259">Azure VPN ゲートウェイによって保存されている共有キーを表示するには、次の Azure CLI コマンドを使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-259">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="1b321-260">ご使用のオンプレミス VPN アプライアンスに適したコマンドを使用して、そのアプライアンス用に構成された共有キーを表示します。</span><span class="sxs-lookup"><span data-stu-id="1b321-260">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="1b321-261">Azure VPN ゲートウェイを保持している *GatewaySubnet* サブネットが NSG に関連付けられていないことを確認します。</span><span class="sxs-lookup"><span data-stu-id="1b321-261">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="1b321-262">サブネットを詳細を表示するには、次の Azure CLI コマンドを使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-262">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="1b321-263">*Network Security Group id* という名前のデータ フィールドがないことを確認します。次の例は、割り当られた NSG (*VPN-Gateway-Group*) を持つ *GatewaySubnet* のインスタンスの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="1b321-263">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="1b321-264">この場合、この NSG に対してルールが定義されていると、ゲートウェイは正しく動作できなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-264">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="1b321-265">**Azure VNet の仮想マシンが VNet 外からの受信トラフィックを許可するように構成されていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-265">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="1b321-266">こうした仮想マシンを含むサブネットに関連付けられているすべての NSG ルールを確認します。</span><span class="sxs-lookup"><span data-stu-id="1b321-266">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="1b321-267">すべての NSG ルールを表示するには、次の Azure CLI コマンドを使用します。</span><span class="sxs-lookup"><span data-stu-id="1b321-267">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="1b321-268">**Azure VPN ゲートウェイが接続されていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-268">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="1b321-269">次の Azure PowerShell コマンドを使用すると、Azure VPN 接続の現在の状態を確認できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-269">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="1b321-270">`<<connection-name>>` パラメーターは、仮想ネットワーク ゲートウェイとローカル ゲートウェイをリンクする Azure VPN 接続の名前です。</span><span class="sxs-lookup"><span data-stu-id="1b321-270">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="1b321-271">次のスニペットは、ゲートウェイが接続されている場合 (最初の例)、および切断されている場合 (2 番目の例) に生成される出力を示しています。</span><span class="sxs-lookup"><span data-stu-id="1b321-271">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="1b321-272">次の推奨事項は、ホスト VM 構成、ネットワーク帯域幅の使用率、またはアプリケーションのパフォーマンスに問題があるかどうかを判断するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-272">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="1b321-273">**サブネットの Azure VM で実行されているゲスト オペレーティング システムのファイアウォールが、オンプレミスの IP 範囲からの許可されたトラフィックを許可するように正しく構成されていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-273">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="1b321-274">**トラフィックの量が、Azure VPN ゲートウェイで使用できる帯域幅の上限に近付いていないことを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-274">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="1b321-275">これを確認する方法は、オンプレミスで実行されている VPN アプライアンスによって異なります。</span><span class="sxs-lookup"><span data-stu-id="1b321-275">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="1b321-276">たとえば、Windows Server 2012 で RRAS を使用している場合は、パフォーマンス モニターを使用して、VPN 接続経由で送受信されているデータ量を追跡できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-276">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="1b321-277">*RAS Total* オブジェクトを使用して、*Bytes Received/Sec* と *Bytes Transmitted/Sec* のカウンターを選択します。</span><span class="sxs-lookup"><span data-stu-id="1b321-277">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="1b321-278">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="1b321-278">![[3]][3]</span></span>

    <span data-ttu-id="1b321-279">結果を、VPN ゲートウェイで使用できる帯域幅 (Basic および Standard SKU の場合は 100 Mbps、High Performance SKU の場合は 200 Mbps) と比較します。</span><span class="sxs-lookup"><span data-stu-id="1b321-279">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="1b321-280">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="1b321-280">![[4]][4]</span></span>

- <span data-ttu-id="1b321-281">**アプリケーションの負荷に対して適切な数およびサイズの VM がデプロイされていることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-281">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="1b321-282">Azure VNet の仮想マシンで、実行速度が遅くなっているものがあるかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="1b321-282">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="1b321-283">ある場合は、それがオーバーロードになっている可能性があり、VM が少なすぎて負荷を処理できないか、ロード バランサーが正しく構成されていないことがあります。</span><span class="sxs-lookup"><span data-stu-id="1b321-283">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="1b321-284">これを確認するには、[診断情報を取得して、分析します][azure-vm-diagnostics]。</span><span class="sxs-lookup"><span data-stu-id="1b321-284">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="1b321-285">Azure Portal を使用して結果を調べることもできますが、パフォーマンス データについて詳細な情報を提供できるサード パーティ製ツールも多数あります。</span><span class="sxs-lookup"><span data-stu-id="1b321-285">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="1b321-286">**アプリケーションがクラウド リソースを効率的に使用していることを確認する。**</span><span class="sxs-lookup"><span data-stu-id="1b321-286">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="1b321-287">各 VM で実行されているアプリケーション コードをインストルメント化し、アプリケーションがリソースを最大限に利用しているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="1b321-287">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="1b321-288">[Application Insights][application-insights] などのツールを使用できます。</span><span class="sxs-lookup"><span data-stu-id="1b321-288">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="1b321-289">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="1b321-289">Deploy the solution</span></span>


<span data-ttu-id="1b321-290">**前提条件。**</span><span class="sxs-lookup"><span data-stu-id="1b321-290">**Prequisites.**</span></span> <span data-ttu-id="1b321-291">既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="1b321-291">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="1b321-292">ソリューションをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="1b321-292">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="1b321-293">下記のボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="1b321-293">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="1b321-294">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="1b321-294">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="1b321-295">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="1b321-295">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="1b321-296">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="1b321-296">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="1b321-297">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="1b321-297">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="1b321-298">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="1b321-298">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="1b321-299">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="1b321-299">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="1b321-300">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="1b321-300">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "オンプレミスと Azure インフラストラクチャにまたがるハイブリッド ネットワーク"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Azure Portal の監査ログ"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "VPN ネットワーク トラフィックを監視するパフォーマンス カウンター"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "VPN ネットワーク パフォーマンス グラフの例"
