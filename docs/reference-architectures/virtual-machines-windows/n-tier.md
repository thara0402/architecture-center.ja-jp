---
title: "N 層アーキテクチャの Windows VM を実行する"
description: "可用性、セキュリティ、スケーラビリティ、および管理容易性のセキュリティに特に注意して Azure で多層アーキテクチャを実装する方法について説明します。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0654239a5bbd966a2aa776415b7f15ae723ffd63
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a><span data-ttu-id="3860f-103">n 層アプリケーションの Windows VM を実行する</span><span class="sxs-lookup"><span data-stu-id="3860f-103">Run Windows VMs for an N-tier application</span></span>

<span data-ttu-id="3860f-104">この参照アーキテクチャでは、N 層アプリケーションの Windows 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。</span><span class="sxs-lookup"><span data-stu-id="3860f-104">This reference architecture shows a set of proven practices for running Windows virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="3860f-105">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="3860f-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="3860f-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="3860f-106">![[0]][0]</span></span>

<span data-ttu-id="3860f-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="3860f-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="3860f-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="3860f-108">Architecture</span></span> 

<span data-ttu-id="3860f-109">N 層アーキテクチャを実装する方法は多数あります。</span><span class="sxs-lookup"><span data-stu-id="3860f-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="3860f-110">図は、典型的な 3 層 Web アプリケーションを示しています。</span><span class="sxs-lookup"><span data-stu-id="3860f-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="3860f-111">このアーキテクチャは「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」に基づいて作成されています。</span><span class="sxs-lookup"><span data-stu-id="3860f-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="3860f-112">Web 層とビジネス層では、負荷分散された VM が使用されます。</span><span class="sxs-lookup"><span data-stu-id="3860f-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="3860f-113">**可用性セット。**</span><span class="sxs-lookup"><span data-stu-id="3860f-113">**Availability sets.**</span></span> <span data-ttu-id="3860f-114">階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="3860f-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="3860f-115">こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="3860f-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="3860f-116">可用性セット内の単一の VM をデプロイできますが、単一の VM がすべての OS およびデータ ディスクに Azure Premium Storage を使用していない限り、その単一の VM は SLA 保証に対して適格ではありません。</span><span class="sxs-lookup"><span data-stu-id="3860f-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="3860f-117">**サブネット。**</span><span class="sxs-lookup"><span data-stu-id="3860f-117">**Subnets.**</span></span> <span data-ttu-id="3860f-118">階層ごとに個別のサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="3860f-119">[CIDR] 表記を使用してアドレス範囲とサブネット マスクを指定します。</span><span class="sxs-lookup"><span data-stu-id="3860f-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="3860f-120">**ロード バランサー**。</span><span class="sxs-lookup"><span data-stu-id="3860f-120">**Load balancers.**</span></span> <span data-ttu-id="3860f-121">[インターネットに接続するロード バランサー][load-balancer-external]を使用して着信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="3860f-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="3860f-122">**ジャンプボックス。**</span><span class="sxs-lookup"><span data-stu-id="3860f-122">**Jumpbox.**</span></span> <span data-ttu-id="3860f-123">[要塞ホスト]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="3860f-123">Also called a [bastion host].</span></span> <span data-ttu-id="3860f-124">管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="3860f-124">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="3860f-125">ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="3860f-125">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="3860f-126">NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-126">The NSG should permit remote desktop (RDP) traffic.</span></span>
* <span data-ttu-id="3860f-127">**監視。**</span><span class="sxs-lookup"><span data-stu-id="3860f-127">**Monitoring.**</span></span> <span data-ttu-id="3860f-128">[Nagios]、[Zabbix]、[Icinga] などの監視ソフトウェアを使用して、応答時間、VM の稼働時間、システムの全体的な正常性に関する洞察を得ることができます。</span><span class="sxs-lookup"><span data-stu-id="3860f-128">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="3860f-129">個別の管理サブネットに配置されている VM 上に監視ソフトウェアをインストールします。</span><span class="sxs-lookup"><span data-stu-id="3860f-129">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="3860f-130">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="3860f-130">**NSGs.**</span></span> <span data-ttu-id="3860f-131">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="3860f-131">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="3860f-132">たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。</span><span class="sxs-lookup"><span data-stu-id="3860f-132">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="3860f-133">**SQL Server Always On 可用性グループ。**</span><span class="sxs-lookup"><span data-stu-id="3860f-133">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="3860f-134">レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。</span><span class="sxs-lookup"><span data-stu-id="3860f-134">Provides high availability at the data tier, by enabling replication and failover.</span></span>
* <span data-ttu-id="3860f-135">**Active Directory Domain Services (AD DS) サーバー。**</span><span class="sxs-lookup"><span data-stu-id="3860f-135">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="3860f-136">Windows Server 2016 に先立って、SQL Server Always On 可用性グループがドメインに参加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-136">Prior to Windows Server 2016, SQL Server Always On Availability Groups must be joined to a domain.</span></span> <span data-ttu-id="3860f-137">これは、可用性グループが Windows Server フェールオーバー クラスター (WSFC) テクノロジに依存するためです。</span><span class="sxs-lookup"><span data-stu-id="3860f-137">This is because Availability Groups depend on Windows Server Failover Cluster (WSFC) technology.</span></span> <span data-ttu-id="3860f-138">Windows Server 2016 では Active Directory なしでフェールオーバー クラスターを作成する機能が導入されました。この場合は AD DS サーバーはこのアーキテクチャには不要です。</span><span class="sxs-lookup"><span data-stu-id="3860f-138">Windows Server 2016 introduces the ability to create a Failover Cluster without Active Directory, in which case the AD DS servers are not required for this architecture.</span></span> <span data-ttu-id="3860f-139">詳細については、「[What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new]」(Windows Server 2016 でのフェールオーバー クラスタリングの新機能) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-139">For more information, see [What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new].</span></span>
* <span data-ttu-id="3860f-140">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="3860f-140">**Azure DNS**.</span></span> <span data-ttu-id="3860f-141">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="3860f-141">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="3860f-142">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="3860f-142">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="3860f-143">Recommendations</span><span class="sxs-lookup"><span data-stu-id="3860f-143">Recommendations</span></span>

<span data-ttu-id="3860f-144">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-144">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="3860f-145">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-145">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="3860f-146">VNet/サブネット</span><span class="sxs-lookup"><span data-stu-id="3860f-146">VNet / Subnets</span></span>

<span data-ttu-id="3860f-147">VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。</span><span class="sxs-lookup"><span data-stu-id="3860f-147">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="3860f-148">[CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。</span><span class="sxs-lookup"><span data-stu-id="3860f-148">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="3860f-149">標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。</span><span class="sxs-lookup"><span data-stu-id="3860f-149">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="3860f-150">後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。</span><span class="sxs-lookup"><span data-stu-id="3860f-150">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="3860f-151">VNet を作成した後は、アドレス範囲を変更できません。</span><span class="sxs-lookup"><span data-stu-id="3860f-151">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="3860f-152">機能とセキュリティの要件を念頭に置いてサブネットを設計します。</span><span class="sxs-lookup"><span data-stu-id="3860f-152">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="3860f-153">同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。</span><span class="sxs-lookup"><span data-stu-id="3860f-153">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="3860f-154">VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-154">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="3860f-155">各サブネットに対して、サブネットのアドレス空間を CIDR 表記で指定します。</span><span class="sxs-lookup"><span data-stu-id="3860f-155">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="3860f-156">たとえば、"10.0.0.0/24" は IP アドレス 256 個の範囲を作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-156">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="3860f-157">VM ではこの内の 251 個を使用できます。5 個は予約されています。</span><span class="sxs-lookup"><span data-stu-id="3860f-157">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="3860f-158">アドレス範囲がサブネット間で重複しないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-158">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="3860f-159">[Virtual Network に関する FAQ][vnet faq] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-159">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="3860f-160">ネットワーク セキュリティ グループ</span><span class="sxs-lookup"><span data-stu-id="3860f-160">Network security groups</span></span>

<span data-ttu-id="3860f-161">NSG ルールを使用して階層間のトラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="3860f-161">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="3860f-162">たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。</span><span class="sxs-lookup"><span data-stu-id="3860f-162">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="3860f-163">これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-163">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="3860f-164">NSG を作成し、データベース層のサブネットに関連付けます。</span><span class="sxs-lookup"><span data-stu-id="3860f-164">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="3860f-165">VNet からのすべての着信トラフィックを拒否するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="3860f-165">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="3860f-166">(ルール内で `VIRTUAL_NETWORK` タグを使用します。)</span><span class="sxs-lookup"><span data-stu-id="3860f-166">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="3860f-167">ビジネス層のサブネットからの着信トラフィックを許可するルールを、より高い優先順位で追加します。</span><span class="sxs-lookup"><span data-stu-id="3860f-167">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="3860f-168">このルールが前のルールを上書きし、ビジネス層がデータベース層と対話できるようになります。</span><span class="sxs-lookup"><span data-stu-id="3860f-168">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="3860f-169">データベース層のサブネット自体からの着信トラフィックを許可するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="3860f-169">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="3860f-170">このルールによって、データベースのレプリケーションやフェールオーバーに必要な、データベース層内の VM どうしの対話が可能になります。</span><span class="sxs-lookup"><span data-stu-id="3860f-170">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="3860f-171">ジャンプボックスのサブネットからの RDP トラフィックを許可するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="3860f-171">Add a rule that allows RDP traffic from the jumpbox subnet.</span></span> <span data-ttu-id="3860f-172">このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。</span><span class="sxs-lookup"><span data-stu-id="3860f-172">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="3860f-173">NSG の既定のルールでは、VNet 内からの着信トラフィックがすべて許可されます。</span><span class="sxs-lookup"><span data-stu-id="3860f-173">An NSG has default rules that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="3860f-174">これらのルールは削除できませんが、より優先順位の高いルールを作成することで上書きできます。</span><span class="sxs-lookup"><span data-stu-id="3860f-174">These rules can't be deleted, but you can override them by creating higher priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="3860f-175">ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="3860f-175">Load balancers</span></span>

<span data-ttu-id="3860f-176">外部ロード バランサーは、インターネット トラフィックを Web 層に分散します。</span><span class="sxs-lookup"><span data-stu-id="3860f-176">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="3860f-177">このロード バランサー用にパブリック IP アドレスを作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-177">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="3860f-178">[インターネットに接続するロード バランサーの作成][lb-external-create]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-178">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="3860f-179">内部ロード バランサーは、Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="3860f-179">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="3860f-180">このロード バランサーにプライベート IP アドレスを付与するには、フロントエンド IP 構成を作成し、ビジネス層のサブネットに関連付けます。</span><span class="sxs-lookup"><span data-stu-id="3860f-180">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="3860f-181">[内部ロード バランサーの作成][lb-internal-create]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-181">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="3860f-182">SQL Server Always On 可用性グループ</span><span class="sxs-lookup"><span data-stu-id="3860f-182">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="3860f-183">SQL Server の高可用性のために [Always On 可用性グループ][sql-alwayson]の使用をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3860f-183">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="3860f-184">Windows Server 2016 に先立って、Always On 可用性グループはドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ AD ドメイン内にある必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-184">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="3860f-185">他の層は[可用性グループ リスナー][sql-alwayson-listeners]を使用してデータベースに接続します。</span><span class="sxs-lookup"><span data-stu-id="3860f-185">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="3860f-186">リスナーを使用することで、SQL クライアントは SQL Server の物理インスタンスの名前を知らなくても接続できます。</span><span class="sxs-lookup"><span data-stu-id="3860f-186">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="3860f-187">データベースにアクセスする VM はドメインに参加している必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-187">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="3860f-188">クライアント (ここでは、別の層) は、DNS を使用してリスナーの仮想ネットワーク名を IP アドレスに解決します。</span><span class="sxs-lookup"><span data-stu-id="3860f-188">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="3860f-189">SQL Server Always On 可用性グループを構成する手順は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="3860f-189">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="3860f-190">Windows Server フェールオーバー クラスタリング (WSFC) クラスター、SQL Server Always On 可用性グループ、プライマリ レプリカを作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-190">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="3860f-191">詳細については、「[AlwaysOn 可用性グループの概要][sql-alwayson-getting-started]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-191">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="3860f-192">静的プライベート IP アドレスを持つ内部ロード バランサーを作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-192">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="3860f-193">可用性グループ リスナーを作成し、リスナーの DNS 名を内部ロード バランサーの IP アドレスにマッピングします。</span><span class="sxs-lookup"><span data-stu-id="3860f-193">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="3860f-194">SQL Server リスニング ポート (既定ではTCP ポート 1433) に対するロード バランサーのルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-194">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="3860f-195">ロード バランサーのルールでは *Floating IP* (Direct Server Return とも呼ばれます) を有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-195">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="3860f-196">これにより VM が直接クライアントに応答でき、プライマリ レプリカへの直接接続が可能になります。</span><span class="sxs-lookup"><span data-stu-id="3860f-196">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="3860f-197">Floating IP が有効になっている場合は、フロントエンド ポート番号を、ロード バランサーのルール内のバックエンド ポート番号と同じにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-197">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
  > 
  > 

<span data-ttu-id="3860f-198">SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。</span><span class="sxs-lookup"><span data-stu-id="3860f-198">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="3860f-199">別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーはその後の要求を自動的に新しいプライマリ レプリカにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="3860f-199">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="3860f-200">詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成][sql-alwayson-ilb]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-200">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="3860f-201">フェールオーバー中は、既存のクライアント接続は閉じられます。</span><span class="sxs-lookup"><span data-stu-id="3860f-201">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="3860f-202">フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="3860f-202">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="3860f-203">アプリケーションが書き込みよりもはるかに多くの読み取りを行う場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。</span><span class="sxs-lookup"><span data-stu-id="3860f-203">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="3860f-204">「[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]」(リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-204">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="3860f-205">可用性グループの[手動フェールオーバーの強制][sql-alwayson-force-failover]によってデプロイをテストします。</span><span class="sxs-lookup"><span data-stu-id="3860f-205">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="3860f-206">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="3860f-206">Jumpbox</span></span>

<span data-ttu-id="3860f-207">ジャンプボックスのパフォーマンス要件は最小限に抑えられるため、ジャンプボックスには Standard A1 などの小さな VM のサイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="3860f-207">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="3860f-208">ジャンプボックス用に[パブリック IP アドレス]を作成します。</span><span class="sxs-lookup"><span data-stu-id="3860f-208">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="3860f-209">ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。</span><span class="sxs-lookup"><span data-stu-id="3860f-209">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="3860f-210">アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="3860f-210">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="3860f-211">代わりに、これらの VM へのすべての RDP アクセスは、ジャンプボックスを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-211">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="3860f-212">管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="3860f-212">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="3860f-213">ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="3860f-213">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="3860f-214">ジャンプボックスをセキュリティで保護するには、NSG を作成してジャンプボックスのサブネットに適用します。</span><span class="sxs-lookup"><span data-stu-id="3860f-214">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="3860f-215">安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="3860f-215">Add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="3860f-216">NSG は、サブネットまたはジャンプボックス NIC のいずれかに関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="3860f-216">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="3860f-217">この場合は NIC に関連付けることをお勧めします。こうすることで、仮に同じサブネットに別の VM を追加した場合でも、RDP トラフィックはジャンプボックスのみに許可されます。</span><span class="sxs-lookup"><span data-stu-id="3860f-217">In this case, we recommend attaching it to the NIC, so RDP traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="3860f-218">他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="3860f-218">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="3860f-219">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="3860f-219">Availability considerations</span></span>

<span data-ttu-id="3860f-220">データベース層に複数の VM を備えても、自動的にデータベースの可用性が高くなるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="3860f-220">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="3860f-221">リレーショナル データベースの場合は、通常、高可用性を実現するにはレプリケーションとフェールオーバーを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-221">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span> <span data-ttu-id="3860f-222">SQL Server の場合は、[Always On 可用性グループ][sql-alwayson]の使用をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3860f-222">For SQL Server, we recommend using [Always On Availability Groups][sql-alwayson].</span></span> 

<span data-ttu-id="3860f-223">[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、2 つのリージョンにアプリケーションをレプリケートし、Azure Traffic Manager を使用してフェールオーバーを行います。</span><span class="sxs-lookup"><span data-stu-id="3860f-223">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="3860f-224">詳細については、「[高可用性のために複数のリージョンで Windows VM を実行する][multi-dc]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-224">For more information, see [Run Windows VMs in multiple regions for high availability][multi-dc].</span></span>   

## <a name="security-considerations"></a><span data-ttu-id="3860f-225">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="3860f-225">Security considerations</span></span>

<span data-ttu-id="3860f-226">機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。</span><span class="sxs-lookup"><span data-stu-id="3860f-226">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="3860f-227">Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。</span><span class="sxs-lookup"><span data-stu-id="3860f-227">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="3860f-228">詳細については、「[Azure VM 上の SQL Server 向け Azure Key Vault 統合の構成][sql-keyvault]」を参照してください。アプリケーション シークレット (データベースの接続文字列など) も Key Vault に格納することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3860f-228">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault] It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="3860f-229">ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-229">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="3860f-230">NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。</span><span class="sxs-lookup"><span data-stu-id="3860f-230">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="3860f-231">詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-231">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="3860f-232">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="3860f-232">Scalability considerations</span></span>

<span data-ttu-id="3860f-233">ロード バランサーは、ネットワーク トラフィックを Web 層とビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="3860f-233">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="3860f-234">新しい VM インスタンスを追加することで水平方向にスケーリングします。</span><span class="sxs-lookup"><span data-stu-id="3860f-234">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="3860f-235">負荷に基づいて、Web 層とビジネス層を個別にスケーリングできることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-235">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="3860f-236">クライアント アフィニティを維持するために発生する可能性がある複雑さを減らすには、Web 層の VM はステートレスである必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-236">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="3860f-237">ビジネス ロジックをホストする VM もステートレスである必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-237">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="3860f-238">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="3860f-238">Manageability considerations</span></span>

<span data-ttu-id="3860f-239">[Azure Automation][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef]、[Puppet][puppet] などの集中管理ツールを使用することで、システム全体の管理を簡略化します。</span><span class="sxs-lookup"><span data-stu-id="3860f-239">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="3860f-240">これらのツールでは、複数の VM から取り込まれた診断情報と正常性情報を統合して、システムの全体像を提供できます。</span><span class="sxs-lookup"><span data-stu-id="3860f-240">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="3860f-241">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="3860f-241">Deploy the solution</span></span>

<span data-ttu-id="3860f-242">このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-242">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="3860f-243">前提条件</span><span class="sxs-lookup"><span data-stu-id="3860f-243">Prerequisites</span></span>

<span data-ttu-id="3860f-244">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-244">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="3860f-245">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="3860f-245">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="3860f-246">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-246">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="3860f-247">CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="3860f-247">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="3860f-248">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="3860f-248">Install the [Azure building blocks][azbb] npm package.</span></span>

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. <span data-ttu-id="3860f-249">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="3860f-249">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="3860f-250">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="3860f-250">Deploy the solution using azbb</span></span>

<span data-ttu-id="3860f-251">N 層アプリケーションの参照アーキテクチャで Windows VM をデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="3860f-251">To deploy the Windows VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="3860f-252">上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-windows` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="3860f-252">Navigate to the `virtual-machines\n-tier-windows` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="3860f-253">このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。</span><span class="sxs-lookup"><span data-stu-id="3860f-253">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="3860f-254">参照アーキテクチャをデプロイする前に、これらを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-254">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="3860f-255">`n-tier-windows.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="3860f-255">Open the `n-tier-windows.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="3860f-256">このデプロイ中に実行される複数のスクリプトが **VirtualMachineExtension** オブジェクトと、一部の **VirtualMachine** オブジェクトの **extensions** 設定の両方に存在します。</span><span class="sxs-lookup"><span data-stu-id="3860f-256">There are multiple scripts that run during this deployment both in the  **VirtualMachineExtension** objects and in the **extensions** settings for some of the **VirtualMachine** objects.</span></span> <span data-ttu-id="3860f-257">これらのスクリプトのいくつかには、今変更した管理者ユーザー名とパスワードが必要です。</span><span class="sxs-lookup"><span data-stu-id="3860f-257">Some of these scripts require the administrator user name and password that you have just changed.</span></span> <span data-ttu-id="3860f-258">これらのスクリプトをレビューして、正しい資格情報を指定したことを確認することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3860f-258">It's recommended that you review these scripts to ensure that you specified the correct credentials.</span></span> <span data-ttu-id="3860f-259">正しい資格情報を指定していない場合は、デプロイが失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3860f-259">The deployment may fail if you have not specified the correct credentials.</span></span>
  > 
  > 

<span data-ttu-id="3860f-260">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="3860f-260">Save the file.</span></span>

3. <span data-ttu-id="3860f-261">次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="3860f-261">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

<span data-ttu-id="3860f-262">Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3860f-262">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Microsoft Azure を使用した N 層アーキテクチャ"