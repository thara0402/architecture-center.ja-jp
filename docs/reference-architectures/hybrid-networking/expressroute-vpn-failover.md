---
title: 高可用性ハイブリッド ネットワーク アーキテクチャの実装
description: Azure 仮想ネットワークと、 VPN ゲートウェイ フェールオーバー付きの ExpressRoute で接続されたオンプレミス ネットワークをカバーする、セキュアなサイト間ネットワーク アーキテクチャの実装方法。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 81298215c814cee805eff57fdc28f7c127148b5f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="672e9-103">VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する</span><span class="sxs-lookup"><span data-stu-id="672e9-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="672e9-104">このリファレンス アーキテクチャでは、サイト間仮想プライベート ネットワーク (VPN) をフェールオーバー接続として使用した ExpressRoute を使用して、オンプレミス ネットワークを Azure 仮想ネットワーク (VNet) に接続する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="672e9-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="672e9-105">オンプレミス ネットワークと Azure VNet 間のトラフィックは、ExpressRoute 接続を介して行き来します。</span><span class="sxs-lookup"><span data-stu-id="672e9-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="672e9-106">ExpressRoute 回線の接続が失われた場合、トラフィックは IPSec VPN トンネルを経由してルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="672e9-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> <span data-ttu-id="672e9-107">「[**Deploy this solution**](#deploy-the-solution)」(このソリューションのデプロイ)。</span><span class="sxs-lookup"><span data-stu-id="672e9-107">[**Deploy this solution**.](#deploy-the-solution)</span></span>

<span data-ttu-id="672e9-108">ExpressRoute 回線を使用できない場合、VPN ルートではプライベート ピアリング接続のみが処理されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="672e9-109">パブリック ピアリングと Microsoft ピアリングの接続は、インターネットを使用して処理されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="672e9-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="672e9-110">![[0]][0]</span></span>

<span data-ttu-id="672e9-111">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="672e9-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="672e9-112">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="672e9-112">Architecture</span></span> 

<span data-ttu-id="672e9-113">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="672e9-114">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="672e9-114">**On-premises network**.</span></span> <span data-ttu-id="672e9-115">組織内で運用されているプライベートのローカル エリア ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="672e9-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="672e9-116">**VPN アプライアンス**。</span><span class="sxs-lookup"><span data-stu-id="672e9-116">**VPN appliance**.</span></span> <span data-ttu-id="672e9-117">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービス。</span><span class="sxs-lookup"><span data-stu-id="672e9-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="672e9-118">VPN アプライアンスは、ハードウェア デバイスである場合もありますし、ソフトウェア ソリューション (Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) など) である場合もあります。</span><span class="sxs-lookup"><span data-stu-id="672e9-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="672e9-119">サポートされている VPN アプライアンスの一覧と、Azure への接続用に選択した VPN アプライアンスの構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="672e9-120">**ExpressRoute 回線**。</span><span class="sxs-lookup"><span data-stu-id="672e9-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="672e9-121">エッジ ルーターを介してオンプレミス ネットワークを Azure につなげる、レイヤー 2 またはレイヤー 3 の回線 (接続プロバイダーから提供されます)。</span><span class="sxs-lookup"><span data-stu-id="672e9-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="672e9-122">この回線では、接続プロバイダーによって管理されるハードウェア インフラストラクチャが使用されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="672e9-123">**ExpressRoute 仮想ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="672e9-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="672e9-124">ExpressRoute 仮想ネットワーク ゲートウェイは、オンプレミス ネットワークとの接続に使用される ExpressRoute 回線に、VNet が接続できるようにするためのものです。</span><span class="sxs-lookup"><span data-stu-id="672e9-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="672e9-125">**VPN 仮想ネットワーク ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="672e9-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="672e9-126">VPN 仮想ネットワーク ゲートウェイは、 VNet がオンプレミス ネットワーク内の VPN アプライアンスに接続できるようにするためのものです。</span><span class="sxs-lookup"><span data-stu-id="672e9-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="672e9-127">VPN 仮想ネットワーク ゲートウェイは、オンプレミス ネットワークからの要求を、VPN アプライアンスを通じてのみ受け入れるように構成されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="672e9-128">詳細については、「[オンプレミス ネットワークを Microsoft Azure 仮想ネットワークに接続する][connect-to-an-Azure-vnet]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="672e9-129">**VPN 接続**。</span><span class="sxs-lookup"><span data-stu-id="672e9-129">**VPN connection**.</span></span> <span data-ttu-id="672e9-130">この接続には、接続の種類 (IPSec) を指定するプロパティ と、オンプレミス VPN アプライアンスとの共有キー (トラフィックの暗号化用) を指定するプロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="672e9-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="672e9-131">**Azure 仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="672e9-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="672e9-132">各 VNet は 1 つの Azure リージョンに配置され、複数のアプリケーション層をホストできます。</span><span class="sxs-lookup"><span data-stu-id="672e9-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="672e9-133">アプリケーション層は、各 VNet 内でサブネットを使用してセグメント化できます。</span><span class="sxs-lookup"><span data-stu-id="672e9-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="672e9-134">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="672e9-134">**Gateway subnet**.</span></span> <span data-ttu-id="672e9-135">仮想ネットワーク ゲートウェイは、同じサブネット内に保持されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="672e9-136">**クラウド アプリケーション**。</span><span class="sxs-lookup"><span data-stu-id="672e9-136">**Cloud application**.</span></span> <span data-ttu-id="672e9-137">Azure でホストされるアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="672e9-137">The application hosted in Azure.</span></span> <span data-ttu-id="672e9-138">Azure ロード バランサーを介して接続された複数のサブネットを持つ、複数の層が含まれる場合があります。</span><span class="sxs-lookup"><span data-stu-id="672e9-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="672e9-139">アプリケーション インフラストラクチャの詳細については、「[Windows VM ワークロードの実行][windows-vm-ra]」と「[Linux VM ワークロードの実行][linux-vm-ra]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="672e9-140">Recommendations</span><span class="sxs-lookup"><span data-stu-id="672e9-140">Recommendations</span></span>

<span data-ttu-id="672e9-141">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="672e9-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="672e9-142">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="672e9-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="672e9-143">VNet と GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="672e9-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="672e9-144">ExpressRoute 仮想ネットワーク ゲートウェイと VPN 仮想ネットワーク ゲートウェイは、同じ VNet 内に作成します。</span><span class="sxs-lookup"><span data-stu-id="672e9-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="672e9-145">つまり両者は、*GatewaySubnet* という同じサブネットを共有する必要があります。</span><span class="sxs-lookup"><span data-stu-id="672e9-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="672e9-146">*GatewaySubnet* という名前のサブネットが既に VNet に含まれている場合は、/27 以上のアドレス空間があることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="672e9-147">既存のサブネットが小さすぎる場合は、次の PowerShell コマンドを使用して、そのサブネットを削除します。</span><span class="sxs-lookup"><span data-stu-id="672e9-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="672e9-148">**GatewaySubnet** という名前のサブネットが VNet に含まれていない場合は、次の Powershell コマンドを使用して新しいサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="672e9-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="672e9-149">VPN および ExpressRoute ゲートウェイ</span><span class="sxs-lookup"><span data-stu-id="672e9-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="672e9-150">Azure に接続するための [ExpressRoute 前提条件][expressroute-prereq]が満たされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="672e9-151">Azure VNet 内に VPN 仮想ネットワーク ゲートウェイが既にある場合は、次の Powershell コマンドを使用して、そのゲートウェイを削除します。</span><span class="sxs-lookup"><span data-stu-id="672e9-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="672e9-152">「[Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute]」(Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャを実装する) の手順に従って、ExpressRoute 接続を確立します。</span><span class="sxs-lookup"><span data-stu-id="672e9-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="672e9-153">「[Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn]」(Azure とオンプレミス VPN を使用したハイブリッド ネットワーク アーキテクチャを実装する) の手順に従って、VPN 仮想ネットワーク ゲートウェイ接続を確立します。</span><span class="sxs-lookup"><span data-stu-id="672e9-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="672e9-154">仮想ネットワーク ゲートウェイ接続を確立したら、次の手順で環境をテストします。</span><span class="sxs-lookup"><span data-stu-id="672e9-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="672e9-155">オンプレミス ネットワークから Azure VNet に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="672e9-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="672e9-156">テストのために ExpressRoute 接続を停止するようプロバイダーに依頼します。</span><span class="sxs-lookup"><span data-stu-id="672e9-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="672e9-157">VPN 仮想ネットワーク ゲートウェイ接続を使用して、オンプレミス ネットワークから Azure VNet に接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="672e9-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="672e9-158">ExpressRoute 接続を再確立するよう、プロバイダーに依頼します。</span><span class="sxs-lookup"><span data-stu-id="672e9-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="672e9-159">考慮事項</span><span class="sxs-lookup"><span data-stu-id="672e9-159">Considerations</span></span>

<span data-ttu-id="672e9-160">ExpressRoute の考慮事項については、「[Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute]」(Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャを実装する) のガイダンスを参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="672e9-161">サイト間 VPN の考慮事項については、「[Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn]」(Azure とオンプレミス VPN を使用したハイブリッド ネットワーク アーキテクチャを実装する) のガイダンスを参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="672e9-162">Azure のセキュリティに関する全般的な考慮事項については、「[Microsoft クラウド サービスとネットワーク セキュリティ][best-practices-security]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="672e9-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="672e9-163">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="672e9-163">Deploy the solution</span></span>

<span data-ttu-id="672e9-164">**前提条件。**</span><span class="sxs-lookup"><span data-stu-id="672e9-164">**Prequisites.**</span></span> <span data-ttu-id="672e9-165">既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="672e9-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="672e9-166">ソリューションをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="672e9-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="672e9-167">下記のボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="672e9-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="672e9-168">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="672e9-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="672e9-169">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-er-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="672e9-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="672e9-170">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="672e9-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="672e9-171">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="672e9-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="672e9-172">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="672e9-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="672e9-173">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="672e9-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="672e9-174">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="672e9-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="672e9-175">下のボタンをクリックしてください。</span><span class="sxs-lookup"><span data-stu-id="672e9-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="672e9-176">Azure ポータルでリンクが開くのを待った後、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="672e9-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="672e9-177">**[リソース グループ]** セクションで **[既存のものを使用]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-er-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="672e9-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="672e9-178">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="672e9-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="672e9-179">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="672e9-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="672e9-180">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="672e9-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="672e9-181">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="672e9-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "ExpressRoute および VPN ゲートウェイを使用した高可用性ハイブリッド ネットワーク アーキテクチャ"
