---
title: Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する
description: Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する方法。
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 83367a3be2f7a1e33c2ef7018d42f70aae99104d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="5b427-103">Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="5b427-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="5b427-104">この参照アーキテクチャは、[ハブスポーク][guidance-hub-spoke]参照アーキテクチャに基づいて作成されており、すべてのスポークで利用できる共有サービスがハブに含まれます。</span><span class="sxs-lookup"><span data-stu-id="5b427-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="5b427-105">クラウドへのデータセンターの移行と[仮想データセンター]の構築に向けた最初の手順として、共有する必要がある最初のサービスが ID とセキュリティです。</span><span class="sxs-lookup"><span data-stu-id="5b427-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="5b427-106">この参照アーキテクチャは、Active Directory サービスをオンプレミスのデータセンターから Azure に拡張する方法と、ハブスポーク トポロジに、ファイアウォールとして動作可能なネットワーク仮想アプライアンス (NVA) を追加する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="5b427-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="5b427-107">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="5b427-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="5b427-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="5b427-108">![[0]][0]</span></span>

<span data-ttu-id="5b427-109">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="5b427-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="5b427-110">このトポロジの利点は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="5b427-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="5b427-111">**コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。</span><span class="sxs-lookup"><span data-stu-id="5b427-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="5b427-112">**サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。</span><span class="sxs-lookup"><span data-stu-id="5b427-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="5b427-113">**問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。</span><span class="sxs-lookup"><span data-stu-id="5b427-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="5b427-114">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="5b427-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="5b427-115">DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。</span><span class="sxs-lookup"><span data-stu-id="5b427-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="5b427-116">共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="5b427-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="5b427-117">相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。</span><span class="sxs-lookup"><span data-stu-id="5b427-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="5b427-118">セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。</span><span class="sxs-lookup"><span data-stu-id="5b427-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="5b427-119">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="5b427-119">Architecture</span></span>

<span data-ttu-id="5b427-120">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="5b427-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="5b427-121">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="5b427-121">**On-premises network**.</span></span> <span data-ttu-id="5b427-122">組織内で実行されているプライベートなローカル エリア ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="5b427-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="5b427-123">**VPN デバイス**。</span><span class="sxs-lookup"><span data-stu-id="5b427-123">**VPN device**.</span></span> <span data-ttu-id="5b427-124">オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。</span><span class="sxs-lookup"><span data-stu-id="5b427-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="5b427-125">VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。</span><span class="sxs-lookup"><span data-stu-id="5b427-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="5b427-126">サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="5b427-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="5b427-127">**VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="5b427-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="5b427-128">仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="5b427-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="5b427-129">詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="5b427-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="5b427-130">この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="5b427-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="5b427-131">**ハブ VNet**。</span><span class="sxs-lookup"><span data-stu-id="5b427-131">**Hub VNet**.</span></span> <span data-ttu-id="5b427-132">ハブスポーク トポロジのハブとして使用する Azure VNet。</span><span class="sxs-lookup"><span data-stu-id="5b427-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="5b427-133">ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。</span><span class="sxs-lookup"><span data-stu-id="5b427-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="5b427-134">**ゲートウェイ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="5b427-134">**Gateway subnet**.</span></span> <span data-ttu-id="5b427-135">複数の仮想ネットワーク ゲートウェイが同じサブネットに保持されます。</span><span class="sxs-lookup"><span data-stu-id="5b427-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="5b427-136">**共有サービス サブネット**。</span><span class="sxs-lookup"><span data-stu-id="5b427-136">**Shared services subnet**.</span></span> <span data-ttu-id="5b427-137">DNS や AD DS など、すべてのスポーク間で共有できるサービスをホストするために使用されるハブ VNet 内のサブネットです。</span><span class="sxs-lookup"><span data-stu-id="5b427-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="5b427-138">**DMZ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="5b427-138">**DMZ subnet**.</span></span> <span data-ttu-id="5b427-139">ファイアウォールなどの、セキュリティ アプライアンスとして動作できる NVA をホストするために使用されるハブ VNet のサブネットです。</span><span class="sxs-lookup"><span data-stu-id="5b427-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="5b427-140">**スポーク VNet**。</span><span class="sxs-lookup"><span data-stu-id="5b427-140">**Spoke VNets**.</span></span> <span data-ttu-id="5b427-141">ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。</span><span class="sxs-lookup"><span data-stu-id="5b427-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="5b427-142">スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="5b427-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="5b427-143">各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。</span><span class="sxs-lookup"><span data-stu-id="5b427-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="5b427-144">アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="5b427-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="5b427-145">**VNet ピアリング**。</span><span class="sxs-lookup"><span data-stu-id="5b427-145">**VNet peering**.</span></span> <span data-ttu-id="5b427-146">[ピアリング接続][vnet-peering]を使用して、同じ Azure リージョン内の 2 つの VNet を接続できます。</span><span class="sxs-lookup"><span data-stu-id="5b427-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="5b427-147">ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。</span><span class="sxs-lookup"><span data-stu-id="5b427-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="5b427-148">ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。</span><span class="sxs-lookup"><span data-stu-id="5b427-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="5b427-149">ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。</span><span class="sxs-lookup"><span data-stu-id="5b427-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="5b427-150">この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。</span><span class="sxs-lookup"><span data-stu-id="5b427-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="5b427-151">これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。</span><span class="sxs-lookup"><span data-stu-id="5b427-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="5b427-152">Recommendations</span><span class="sxs-lookup"><span data-stu-id="5b427-152">Recommendations</span></span>

<span data-ttu-id="5b427-153">[ハブスポーク][guidance-hub-spoke]参照アーキテクチャのすべての推奨事項が、共有サービスの参照アーキテクチャにも適用されます。</span><span class="sxs-lookup"><span data-stu-id="5b427-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="5b427-154">また、共有サービスのほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="5b427-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="5b427-155">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="5b427-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="5b427-156">ID</span><span class="sxs-lookup"><span data-stu-id="5b427-156">Identity</span></span>

<span data-ttu-id="5b427-157">ほとんどの企業組織のオンプレミス データセンターには Active Directory ディレクトリ サービス (ADDS) 環境があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="5b427-158">ADDS に依存しているオンプレミスのネットワークから Azure に移動される資産の管理を容易にするには、Azure で ADDS ドメイン コントローラーをホストすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="5b427-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="5b427-159">Azure とオンプレミスの環境を別々に管理するグループ ポリシー オブジェクトを使用する場合は、Azure リージョンごとに別の AD サイトを使用します。</span><span class="sxs-lookup"><span data-stu-id="5b427-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="5b427-160">依存するワークロードがアクセスできる中心の VNet (ハブ) に、ドメイン コントローラーを配置します。</span><span class="sxs-lookup"><span data-stu-id="5b427-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="5b427-161">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="5b427-161">Security</span></span>

<span data-ttu-id="5b427-162">オンプレミスの環境から Azure にワークロードを移動すると、これらのワークロードの一部を VM でホストする必要があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="5b427-163">コンプライアンス上の理由から、これらのワークロードを通過するトラフィックを制限することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="5b427-164">Azure では、ネットワーク仮想アプライアンス (NVA) を使用して、さまざまな種類のセキュリティおよびパフォーマンス サービスをホストできます。</span><span class="sxs-lookup"><span data-stu-id="5b427-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="5b427-165">現在、オンプレミスで特定のアプライアンスのセットを使い慣れている場合は、可能であれば、Azure で同じ仮想アプライアンスを使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="5b427-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="5b427-166">この参照アーキテクチャのデプロイ スクリプトでは、IP 転送が有効な Ubuntu VM を使用してネットワーク仮想アプライアンスを模倣します。</span><span class="sxs-lookup"><span data-stu-id="5b427-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="5b427-167">考慮事項</span><span class="sxs-lookup"><span data-stu-id="5b427-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="5b427-168">VNet ピアリングの制限の解消</span><span class="sxs-lookup"><span data-stu-id="5b427-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="5b427-169">Azure の [VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="5b427-170">制限を超える数のスポークを許可する場合は、ハブスポークハブスポーク トポロジの作成を検討してください。このトポロジでは、第 1 レベルのスポークもハブとして機能します。</span><span class="sxs-lookup"><span data-stu-id="5b427-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="5b427-171">この手法を次の図に示します。</span><span class="sxs-lookup"><span data-stu-id="5b427-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="5b427-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="5b427-172">![[3]][3]</span></span>

<span data-ttu-id="5b427-173">また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="5b427-174">たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。</span><span class="sxs-lookup"><span data-stu-id="5b427-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="5b427-175">このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="5b427-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="5b427-176">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="5b427-176">Deploy the solution</span></span>

<span data-ttu-id="5b427-177">このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="5b427-178">このデプロイでは、各 VNet の Ubuntu VM を使用して接続をテストします。</span><span class="sxs-lookup"><span data-stu-id="5b427-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="5b427-179">実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。</span><span class="sxs-lookup"><span data-stu-id="5b427-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="5b427-180">前提条件</span><span class="sxs-lookup"><span data-stu-id="5b427-180">Prerequisites</span></span>

<span data-ttu-id="5b427-181">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="5b427-182">[参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="5b427-182">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="5b427-183">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="5b427-184">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="5b427-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="5b427-185">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="5b427-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="5b427-186">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="5b427-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="5b427-187">シミュレートされたオンプレミスのデータセンターを azbb を使用してデプロイする</span><span class="sxs-lookup"><span data-stu-id="5b427-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="5b427-188">シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="5b427-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="5b427-189">上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\shared-services-stack\` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="5b427-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="5b427-190">`onprem.json` ファイルを開き、次に示す 45 行目と 46 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="5b427-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="5b427-191">次に示すように、`azbb` を実行して、シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="5b427-192">別のリソース グループ名 (`onprem-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="5b427-193">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="5b427-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="5b427-194">このデプロイでは、仮想ネットワーク、Windows を実行する仮想マシン、および VPN ゲートウェイを作成します。</span><span class="sxs-lookup"><span data-stu-id="5b427-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="5b427-195">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="5b427-196">Azure のハブ VNet</span><span class="sxs-lookup"><span data-stu-id="5b427-196">Azure hub VNet</span></span>

<span data-ttu-id="5b427-197">ハブ VNet をデプロイして、上記の手順で作成したシミュレートされたオンプレミスの VNet に接続するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5b427-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="5b427-198">`hub-vnet.json` ファイルを開き、次に示す 50 行目と 51 行目の引用符の間にユーザー名とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="5b427-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="5b427-199">52 行目の `osType` に `Windows` または `Linux` と入力し、ジャンプボックス用のオペレーティング システムとして Windows Server 2016 Datacenter または Ubuntu 16.04 のいずれかをインストールします。</span><span class="sxs-lookup"><span data-stu-id="5b427-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="5b427-200">次に示す 83 行目の引用符の間に共有キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="5b427-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="5b427-201">次に示すように、`azbb` を実行して、シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="5b427-202">別のリソース グループ名 (`hub-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="5b427-203">デプロイが完了するのを待機します。</span><span class="sxs-lookup"><span data-stu-id="5b427-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="5b427-204">このデプロイでは、仮想ネットワーク、仮想マシン、VPN ゲートウェイ、および前のセクションで作成したゲートウェイへの接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="5b427-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="5b427-205">VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="5b427-206">Azure 内の ADDS</span><span class="sxs-lookup"><span data-stu-id="5b427-206">ADDS in Azure</span></span>

<span data-ttu-id="5b427-207">Azure に ADDS ドメイン コントローラーをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5b427-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="5b427-208">`hub-adds.json` ファイルを開き、次に示す 14 行目と 15 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="5b427-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="5b427-209">次に示すように、`azbb` を実行して ADDS ドメイン コントローラーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="5b427-210">別のリソース グループ名 (`hub-adds-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

   > [!NOTE]
   > <span data-ttu-id="5b427-211">シミュレートされたオンプレミスのデータセンターでホストされているドメインに 2 つの VM を参加させ、その VM に AD DS をインストールする必要があるため、デプロイのこの部分は数分かかる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="5b427-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="5b427-212">NVA</span><span class="sxs-lookup"><span data-stu-id="5b427-212">NVA</span></span>

<span data-ttu-id="5b427-213">`dmz` サブネットに NVA をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5b427-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="5b427-214">`hub-nva.json` ファイルを開き、次に示す 13 行目と 14 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="5b427-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="5b427-215">`azbb` を実行して NVA VM とユーザー定義のルートをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="5b427-216">別のリソース グループ名 (`hub-nva-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="5b427-217">Azure のスポーク VNet</span><span class="sxs-lookup"><span data-stu-id="5b427-217">Azure spoke VNets</span></span>

<span data-ttu-id="5b427-218">スポーク VNet をデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5b427-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="5b427-219">`spoke1.json` ファイルを開き、次に示す 52 行目と 53 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="5b427-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="5b427-220">54 行目の `osType` に `Windows` または `Linux` と入力し、ジャンプボックス用のオペレーティング システムとして Windows Server 2016 Datacenter または Ubuntu 16.04 のいずれかをインストールします。</span><span class="sxs-lookup"><span data-stu-id="5b427-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="5b427-221">次に示すように、`azbb` を実行して、最初のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="5b427-222">別のリソース グループ名 (`spoke1-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="5b427-223">ファイル `spoke2.json` に対して上記の手順 1. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="5b427-223">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="5b427-224">次に示すように、`azbb` を実行して、2 番目のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="5b427-225">別のリソース グループ名 (`spoke2-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="5b427-226">Azure のスポーク VNet へのハブ VNet のピアリング</span><span class="sxs-lookup"><span data-stu-id="5b427-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="5b427-227">ハブ VNet からスポーク VNet にピアリング接続を作成するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5b427-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="5b427-228">`hub-vnet-peering.json` ファイルを開き、リソース グループ名、および 29 行目から始まる各仮想ネットワーク ピアリングの仮想ネットワーク名が正しいことを確認します。</span><span class="sxs-lookup"><span data-stu-id="5b427-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="5b427-229">次に示すように、`azbb` を実行して、最初のスポーク VNet 環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="5b427-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="5b427-230">別のリソース グループ名 (`hub-vnet-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。</span><span class="sxs-lookup"><span data-stu-id="5b427-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[0]: ./images/shared-services.png "Azure での共有サービス トポロジ"
[3]: ./images/hub-spokehub-spoke.svg "Hub-spoke-hub-spoke topology in Azure" (Azure のハブスポークハブスポーク トポロジ)
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
