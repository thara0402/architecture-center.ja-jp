---
title: VPN を使用した Azure へのオンプレミス ネットワークの接続
titleSuffix: Azure Reference Architectures
description: VPN を使用して接続された Azure 仮想ネットワークとオンプレミス ネットワークにまたがる、セキュリティで保護されたサイト間ネットワーク アーキテクチャを実装します。
author: RohitSharma-pnp
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 515cd3f5d23957e0e153c9d25198b3cb98b92a5d
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487978"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="61c74-103">VPN ゲートウェイを使用した Azure へのオンプレミス ネットワークの接続</span><span class="sxs-lookup"><span data-stu-id="61c74-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="61c74-104">この参照アーキテクチャでは、サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure に拡張する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="61c74-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="61c74-105">オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) 間のトラフィックは、IPSec VPN トンネルを介して行き来します。</span><span class="sxs-lookup"><span data-stu-id="61c74-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> <span data-ttu-id="61c74-106">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="61c74-106">[**Deploy this solution**](#deploy-the-solution).</span></span>

![オンプレミスと Azure インフラストラクチャにまたがるハイブリッド ネットワーク](./images/vpn.png)

<span data-ttu-id="61c74-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="61c74-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="61c74-109">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="61c74-109">Architecture</span></span>

<span data-ttu-id="61c74-110">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="61c74-110">The architecture consists of the following components.</span></span>

- <span data-ttu-id="61c74-111">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="61c74-111">**On-premises network**.</span></span> <span data-ttu-id="61c74-112">組織内で運用されているプライベートのローカル エリア ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="61c74-112">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="61c74-113">**VPN アプライアンス**。</span><span class="sxs-lookup"><span data-stu-id="61c74-113">**VPN appliance**.</span></span> <span data-ttu-id="61c74-114">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービス。</span><span class="sxs-lookup"><span data-stu-id="61c74-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="61c74-115">VPN アプライアンスは、ハードウェア デバイスである場合や、ソフトウェア ソリューション (Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) など) である場合があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="61c74-116">サポートされている VPN アプライアンスの一覧と、Azure VPN ゲートウェイに接続するためのその構成については、[サイト間 VPN Gateway 接続用の VPN デバイス][vpn-appliance]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="61c74-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="61c74-117">**仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="61c74-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="61c74-118">Azure VPN ゲートウェイのクラウド アプリケーションとコンポーネントは同じ [VNet][azure-virtual-network] に存在します。</span><span class="sxs-lookup"><span data-stu-id="61c74-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

- <span data-ttu-id="61c74-119">**Azure VPN ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="61c74-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="61c74-120">[VPN ゲートウェイ][azure-vpn-gateway] サービスを使用すると、VNet を VPN アプライアンスを介してオンプレミス ネットワークに接続できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="61c74-121">詳細については、「[オンプレミス ネットワークを Microsoft Azure 仮想ネットワークに接続する][connect-to-an-Azure-vnet]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="61c74-122">VPN ゲートウェイには、次の要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="61c74-122">The VPN gateway includes the following elements:</span></span>

  - <span data-ttu-id="61c74-123">**仮想ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="61c74-123">**Virtual network gateway**.</span></span> <span data-ttu-id="61c74-124">VNet の仮想 VPN アプライアンスを提供するリソース。</span><span class="sxs-lookup"><span data-stu-id="61c74-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="61c74-125">オンプレミス ネットワークから VNet へのトラフィックのルーティングを行います。</span><span class="sxs-lookup"><span data-stu-id="61c74-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  - <span data-ttu-id="61c74-126">**ローカル ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="61c74-126">**Local network gateway**.</span></span> <span data-ttu-id="61c74-127">オンプレミス VPN アプライアンスの抽象化。</span><span class="sxs-lookup"><span data-stu-id="61c74-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="61c74-128">クラウド アプリケーションからオンプレミス ネットワークへのネットワーク トラフィックは、このゲートウェイを介してルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="61c74-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  - <span data-ttu-id="61c74-129">**接続**。</span><span class="sxs-lookup"><span data-stu-id="61c74-129">**Connection**.</span></span> <span data-ttu-id="61c74-130">この接続には、接続の種類 (IPSec) を指定するプロパティと、オンプレミス VPN アプライアンスとの共有キー (トラフィックの暗号化用) を指定するプロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="61c74-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  - <span data-ttu-id="61c74-131">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="61c74-131">**Gateway subnet**.</span></span> <span data-ttu-id="61c74-132">仮想ネットワーク ゲートウェイは独自のサブネットで保持され、以下の推奨事項セクションで説明されているさまざまな要件の対象となります。</span><span class="sxs-lookup"><span data-stu-id="61c74-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

- <span data-ttu-id="61c74-133">**クラウド アプリケーション**。</span><span class="sxs-lookup"><span data-stu-id="61c74-133">**Cloud application**.</span></span> <span data-ttu-id="61c74-134">Azure でホストされるアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="61c74-134">The application hosted in Azure.</span></span> <span data-ttu-id="61c74-135">Azure ロード バランサーを介して接続された複数のサブネットを持つ、複数の層が含まれる場合があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="61c74-136">アプリケーション インフラストラクチャの詳細については、「[Windows VM ワークロードの実行][windows-vm-ra]」と「[Linux VM ワークロードの実行][linux-vm-ra]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="61c74-137">**内部ロード バランサー**: </span><span class="sxs-lookup"><span data-stu-id="61c74-137">**Internal load balancer**.</span></span> <span data-ttu-id="61c74-138">VPN ゲートウェイからのネットワーク トラフィックは、内部ロード バランサーを介してクラウド アプリケーションにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="61c74-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="61c74-139">ロード バランサーは、アプリケーションのフロントエンドのサブネットに配置されます。</span><span class="sxs-lookup"><span data-stu-id="61c74-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="61c74-140">Recommendations</span><span class="sxs-lookup"><span data-stu-id="61c74-140">Recommendations</span></span>

<span data-ttu-id="61c74-141">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="61c74-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="61c74-142">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="61c74-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="61c74-143">VNet とゲートウェイ サブネット</span><span class="sxs-lookup"><span data-stu-id="61c74-143">VNet and gateway subnet</span></span>

<span data-ttu-id="61c74-144">必要なすべてのリソースにとって十分な大きさのアドレス空間を持つ Azure VNet を作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="61c74-145">追加の VM が今後必要になると思われる場合は、拡大に対応できるだけの領域を VNet アドレス空間に確保します。</span><span class="sxs-lookup"><span data-stu-id="61c74-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="61c74-146">VNet のアドレス空間は、オンプレミス ネットワークと重複させることはできません。</span><span class="sxs-lookup"><span data-stu-id="61c74-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="61c74-147">たとえば、上の図は、VNet に対してアドレス空間 10.20.0.0/16 を使用しています。</span><span class="sxs-lookup"><span data-stu-id="61c74-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="61c74-148">*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。</span><span class="sxs-lookup"><span data-stu-id="61c74-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="61c74-149">このサブネットは、仮想ネットワーク ゲートウェイで必要です。</span><span class="sxs-lookup"><span data-stu-id="61c74-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="61c74-150">このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="61c74-151">また、このサブネットは、アドレス空間の途中には配置しないでください。</span><span class="sxs-lookup"><span data-stu-id="61c74-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="61c74-152">ゲートウェイ サブネットのアドレス空間は、VNet アドレス空間の上端に設定することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="61c74-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="61c74-153">図の例では 10.20.255.224/27 を使用しています。</span><span class="sxs-lookup"><span data-stu-id="61c74-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="61c74-154">[CIDR] の簡単な計算手順を次に示します。</span><span class="sxs-lookup"><span data-stu-id="61c74-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="61c74-155">VNet のアドレス空間の変数ビットを、ゲートウェイ サブネットで使用されているビットまで 1 に設定し、残りのビットを 0 に設定します。</span><span class="sxs-lookup"><span data-stu-id="61c74-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="61c74-156">結果のビットを 10 進数に変換し、プレフィックス長をゲートウェイ サブネットのサイズに設定して、その 10 進数をアドレス空間として表現します。</span><span class="sxs-lookup"><span data-stu-id="61c74-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="61c74-157">たとえば、IP アドレス範囲が 10.20.0.0/16 の VNet については、上の手順 1. を適用すると、10.20.0b11111111.0b11100000 になります。</span><span class="sxs-lookup"><span data-stu-id="61c74-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="61c74-158">これを 10 進数に変換し、アドレス空間として表現すると、10.20.255.224/27 が生成されます。</span><span class="sxs-lookup"><span data-stu-id="61c74-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span>

> [!WARNING]
> <span data-ttu-id="61c74-159">VM をゲートウェイ サブネットにデプロイしないでください。</span><span class="sxs-lookup"><span data-stu-id="61c74-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="61c74-160">また、NSG をこのサブネットに割り当てないでください。割り当てると、ゲートウェイが機能しなくなります。</span><span class="sxs-lookup"><span data-stu-id="61c74-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
>

### <a name="virtual-network-gateway"></a><span data-ttu-id="61c74-161">仮想ネットワーク ゲートウェイ</span><span class="sxs-lookup"><span data-stu-id="61c74-161">Virtual network gateway</span></span>

<span data-ttu-id="61c74-162">仮想ネットワーク ゲートウェイにパブリック IP アドレスを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="61c74-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="61c74-163">ゲートウェイ サブネットで仮想ネットワーク ゲートウェイを作成し、新しく割り当てられたパブリック IP アドレスを割り当てます。</span><span class="sxs-lookup"><span data-stu-id="61c74-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="61c74-164">要件に最も合致し、VPN アプライアンスによって有効になっているゲートウェイの種類を使用します。</span><span class="sxs-lookup"><span data-stu-id="61c74-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="61c74-165">アドレス プレフィックスなどのポリシー基準に基づいて、要求がどのようにルーティングされるかを厳密に制御する必要がある場合、[ポリシー ベースのゲートウェイ][policy-based-routing]を作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="61c74-166">ポリシー ベースのゲートウェイは、静的ルーティングを使用し、サイト間接続でのみ機能します。</span><span class="sxs-lookup"><span data-stu-id="61c74-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="61c74-167">RRAS を使用してオンプレミス ネットワークに接続する場合、マルチサイトまたはリージョン間接続をサポートする場合、または VNet 間接続 (複数の VNet を横断するルートを含む) を実装する場合、[ルート ベースのゲートウェイ][route-based-routing]を作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="61c74-168">ルート ベースのゲートウェイは、ネットワーク間でトラフィックを転送する動的ルーティングを使用します。</span><span class="sxs-lookup"><span data-stu-id="61c74-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="61c74-169">これは代替ルートを試すことができるため、静的ルートよりもネットワーク パスの障害に耐性があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="61c74-170">また、ルート ベースのゲートウェイでは、ネットワーク アドレスが変わったときに、ルートを手動で更新する必要がない可能性があるため、管理オーバーヘッドを削減できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="61c74-171">サポートされている VPN アプライアンスの一覧については、[サイト間 VPN Gateway 接続の VPN デバイス][vpn-appliances]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="61c74-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="61c74-172">ゲートウェイを作成したら、ゲートウェイを削除して再作成しない限り、ゲートウェイの種類を変更することはできません。</span><span class="sxs-lookup"><span data-stu-id="61c74-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
>

<span data-ttu-id="61c74-173">ご自分のスループット要件に最も合致する Azure VPN ゲートウェイ SKU を選択します。</span><span class="sxs-lookup"><span data-stu-id="61c74-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="61c74-174">詳細については、「[ゲートウェイの SKU][azure-gateway-skus]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-174">For more information, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="61c74-175">Basic SKU は、Azure ExpressRoute と互換性がありません。</span><span class="sxs-lookup"><span data-stu-id="61c74-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="61c74-176">ゲートウェイの作成後、[SKU を変更][changing-SKUs]できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
>

<span data-ttu-id="61c74-177">料金は、ゲートウェイがプロビジョニングされ、利用可能になっている時間に基づいて発生します。</span><span class="sxs-lookup"><span data-stu-id="61c74-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="61c74-178">「[VPN Gateway の価格][azure-gateway-charges]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="61c74-179">要求がアプリケーション VM に直接渡されるようにするのではなく、アプリケーションの受信トラフィックをゲートウェイから内部ロード バランサーに転送する、ゲートウェイ サブネットのルーティング ルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="61c74-180">オンプレミス ネットワーク接続</span><span class="sxs-lookup"><span data-stu-id="61c74-180">On-premises network connection</span></span>

<span data-ttu-id="61c74-181">ローカル ネットワーク ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-181">Create a local network gateway.</span></span> <span data-ttu-id="61c74-182">オンプレミスの VPN アプライアンスのパブリック IP アドレスと、オンプレミス ネットワークのアドレス空間を指定します。</span><span class="sxs-lookup"><span data-stu-id="61c74-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="61c74-183">オンプレミス VPN アプライアンスには、Azure VPN Gateway のローカル ネットワーク ゲートウェイがアクセスできるパブリック IP アドレスが必要であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="61c74-184">VPN デバイスはネットワーク アドレス変換 (NAT) デバイスの背後には配置できません。</span><span class="sxs-lookup"><span data-stu-id="61c74-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="61c74-185">仮想ネットワーク ゲートウェイおよびローカル ネットワーク ゲートウェイのサイト間接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="61c74-186">サイト間 (IPSec) 接続の種類を選択し、共有キーを指定します。</span><span class="sxs-lookup"><span data-stu-id="61c74-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="61c74-187">Azure VPN ゲートウェイによるサイト間の暗号化は IPSec プロトコルに基づいており、認証に事前共有キーを使用します。</span><span class="sxs-lookup"><span data-stu-id="61c74-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="61c74-188">Azure VPN ゲートウェイを作成するときにキーを指定します。</span><span class="sxs-lookup"><span data-stu-id="61c74-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="61c74-189">同じキーを使用して、オンプレミスで実行する VPN アプライアンスを構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="61c74-190">他の認証メカニズムは現在サポートされていません。</span><span class="sxs-lookup"><span data-stu-id="61c74-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="61c74-191">Azure VNet のアドレスに向けた要求が VPN デバイスに転送されるように、オンプレミス ルーティング インフラストラクチャが構成されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="61c74-192">オンプレミス ネットワークのクラウド アプリケーションに必要なポートを開きます。</span><span class="sxs-lookup"><span data-stu-id="61c74-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="61c74-193">接続をテストして、次の点を確認してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-193">Test the connection to verify that:</span></span>

- <span data-ttu-id="61c74-194">トラフィックが Azure VPN ゲートウェイを介してクラウド アプリケーションに到達するように、オンプレミス VPN アプライアンスによって正しくルーティングしている。</span><span class="sxs-lookup"><span data-stu-id="61c74-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
- <span data-ttu-id="61c74-195">トラフィックがオンプレミス ネットワークに戻るように、VNet によって正しくルーティングされている。</span><span class="sxs-lookup"><span data-stu-id="61c74-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
- <span data-ttu-id="61c74-196">両方向の禁止されたトラフィックが、正しくブロックされている。</span><span class="sxs-lookup"><span data-stu-id="61c74-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="61c74-197">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="61c74-197">Scalability considerations</span></span>

<span data-ttu-id="61c74-198">垂直スケーラビリティを制限するには、Basic または Standard の VPN Gateway SKU から High Performance VPN SKU に移行します。</span><span class="sxs-lookup"><span data-stu-id="61c74-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="61c74-199">大量の VPN トラフィックが想定される VNet については、さまざまなワークロードを小さな個別の VNet に分散し、それぞれに対して VPN ゲートウェイを構成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="61c74-200">VNet は、水平方向または垂直方向にパーティション分割できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="61c74-201">水平方向にパーティション分割するには、各階層の VM インスタンスをいくつか、新しい VNet のサブネットに移動します。</span><span class="sxs-lookup"><span data-stu-id="61c74-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="61c74-202">これにより、各 VNet の構造と機能が同じになります。</span><span class="sxs-lookup"><span data-stu-id="61c74-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="61c74-203">垂直方向にパーティション分割する場合は、機能がさまざまな論理領域 (注文処理、請求、顧客アカウントの管理など) に分割されるように各階層を再設計します。</span><span class="sxs-lookup"><span data-stu-id="61c74-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="61c74-204">その後、各機能領域を独自の VNet に配置できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="61c74-205">VNet でオンプレミスの Active Directory ドメイン コントローラーをレプリケートし、VNet に DNS を実装すると、オンプレミスからクラウドにフローするセキュリティ関連および管理トラフィックの削減に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="61c74-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="61c74-206">詳細については、[Azure への Active Directory Domain Services (AD DS) の拡張][adds-extend-domain]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="61c74-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="61c74-207">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="61c74-207">Availability considerations</span></span>

<span data-ttu-id="61c74-208">オンプレミス ネットワークを、Azure VPN ゲートウェイで引き続き使用できるようにする必要がある場合は、オンプレミス VPN ゲートウェイのフェールオーバー クラスターを実装します。</span><span class="sxs-lookup"><span data-stu-id="61c74-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="61c74-209">組織に複数のオンプレミス サイトがある場合は、1 つ以上の Azure VNet への[マルチサイト接続][vpn-gateway-multi-site]を作成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="61c74-210">このアプローチには、動的 (ルート ベース) のルーティングが必要です。したがって、オンプレミス VPN ゲートウェイがこの機能をサポートしていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="61c74-211">サービス レベル アグリーメントの詳細については、「[VPN Gateway のSLA][sla-for-vpn-gateway]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="61c74-212">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="61c74-212">Manageability considerations</span></span>

<span data-ttu-id="61c74-213">オンプレミスの VPN アプライアンスからの診断情報を監視します。</span><span class="sxs-lookup"><span data-stu-id="61c74-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="61c74-214">このプロセスは、VPN アプライアンスが提供する機能によって異なります。</span><span class="sxs-lookup"><span data-stu-id="61c74-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="61c74-215">たとえば、Windows Server 2012 でルーティングとリモート アクセス サービスを使用している場合は、[RRAS ログ記録][rras-logging]です。</span><span class="sxs-lookup"><span data-stu-id="61c74-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="61c74-216">[Azure VPN ゲートウェイ診断][gateway-diagnostic-logs]を使用して、接続の問題に関する情報を取得します。</span><span class="sxs-lookup"><span data-stu-id="61c74-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="61c74-217">こうしたログを使用すると、接続要求の要求元と要求先、使用されたプロトコル、接続の確立方法 (または、試行が失敗した理由) などの情報を追跡できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="61c74-218">Azure Portal で使用できる監査ログを使用して、Azure VPN ゲートウェイの操作ログを監視します。</span><span class="sxs-lookup"><span data-stu-id="61c74-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="61c74-219">ローカル ネットワーク ゲートウェイ、Azure ネットワーク ゲートウェイ、および接続に対して個別のログを使用できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="61c74-220">この情報は、ゲートウェイに対する変更の追跡に使用でき、前に機能していたゲートウェイの動作が何らかの理由で停止した場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="61c74-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

![Azure portal の監査ログ](../_images/guidance-hybrid-network-vpn/audit-logs.png)

<span data-ttu-id="61c74-222">接続を監視し、接続エラー イベントを追跡します。</span><span class="sxs-lookup"><span data-stu-id="61c74-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="61c74-223">この情報は、[Nagios][nagios] などの監視パッケージを使用して取得し、レポートすることができます。</span><span class="sxs-lookup"><span data-stu-id="61c74-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="61c74-224">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="61c74-224">Security considerations</span></span>

<span data-ttu-id="61c74-225">VPN ゲートウェイごとに別の共有キーを生成します。</span><span class="sxs-lookup"><span data-stu-id="61c74-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="61c74-226">強力な共有キーを使用すると、ブルート フォース攻撃に対抗するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="61c74-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="61c74-227">現在、Azure VPN ゲートウェイのキーを事前共有するために、Azure Key Vault を使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="61c74-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
>

<span data-ttu-id="61c74-228">オンプレミスの VPN アプライアンスによって、[Azure VPN ゲートウェイと互換性のある][vpn-appliance-ipsec]暗号化方式が使用されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="61c74-229">ポリシー ベースのルーティングでは、Azure VPN ゲートウェイは、AES256、AES128、および 3DES 暗号化アルゴリズムをサポートしています。</span><span class="sxs-lookup"><span data-stu-id="61c74-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="61c74-230">ルート ベースのゲートウェイは、AES256 および 3DES をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="61c74-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="61c74-231">オンプレミス VPN アプライアンスが境界ネットワーク (DMZ) にあり、その境界ネットワークとインターネットの間にファイアウォールが存在する場合は、サイト間 VPN 接続を許可するために、[追加のファイアウォール規則][additional-firewall-rules]を構成しなければならない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="61c74-232">VNet のアプリケーションがインターネットにデータを送信する場合は、インターネットに向かうすべてのトラフィックがオンプレミス ネットワークを介してルーティングされるように、[強制トンネリングを実装][forced-tunneling]することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="61c74-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="61c74-233">このアプローチを使用すると、アプリケーションによって行われた送信要求をオンプレミス インフラストラクチャから監査できます。</span><span class="sxs-lookup"><span data-stu-id="61c74-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="61c74-234">強制トンネリングは、Azure サービス (Storage サービスなど) と Windows ライセンス マネージャーへの接続に影響を与える場合があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
>

## <a name="deploy-the-solution"></a><span data-ttu-id="61c74-235">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="61c74-235">Deploy the solution</span></span>

<span data-ttu-id="61c74-236">**前提条件**。</span><span class="sxs-lookup"><span data-stu-id="61c74-236">**Prerequisites**.</span></span> <span data-ttu-id="61c74-237">既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="61c74-237">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="61c74-238">ソリューションをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="61c74-238">To deploy the solution, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="61c74-239">下記のボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="61c74-239">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="61c74-240">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="61c74-240">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   - <span data-ttu-id="61c74-241">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="61c74-241">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   - <span data-ttu-id="61c74-242">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="61c74-242">Select the region from the **Location** drop down box.</span></span>
   - <span data-ttu-id="61c74-243">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="61c74-243">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   - <span data-ttu-id="61c74-244">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="61c74-244">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   - <span data-ttu-id="61c74-245">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="61c74-245">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="61c74-246">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="61c74-246">Wait for the deployment to complete.</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="61c74-247">接続のトラブルシューティングについては、[ハイブリッド VPN 接続のトラブルシューティング](./troubleshoot-vpn.md)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="61c74-247">To troubleshoot the connection, see [Troubleshoot a hybrid VPN connection](./troubleshoot-vpn.md).</span></span>

<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[forced-tunneling]: /azure/vpn-gateway/vpn-gateway-about-forced-tunneling
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[azure-cli]: /cli/azure/install-azure-cli
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
