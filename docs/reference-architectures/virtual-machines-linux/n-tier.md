---
title: "Azure で n 層アプリケーションの Linux VM を実行する"
description: "Microsoft Azure で N 層アーキテクチャの Linux VM を実行する方法について説明します。"
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 98814685e0f33f2a1258bf8307a86f92d8a81968
ms.sourcegitcommit: 583e54a1047daa708a9b812caafb646af4d7607b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/28/2017
---
# <a name="run-linux-vms-for-an-n-tier-application"></a><span data-ttu-id="67fc8-103">N 層アプリケーションの Linux VM を実行する</span><span class="sxs-lookup"><span data-stu-id="67fc8-103">Run Linux VMs for an N-tier application</span></span>

<span data-ttu-id="67fc8-104">この参照アーキテクチャでは、N 層アプリケーションの Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-104">This reference architecture shows a set of proven practices for running Linux virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="67fc8-105">**以下のソリューションをデプロイします**。</span><span class="sxs-lookup"><span data-stu-id="67fc8-105">**Deploy this solution**.</span></span>](#deploy-the-solution)  

<span data-ttu-id="67fc8-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="67fc8-106">![[0]][0]</span></span>

<span data-ttu-id="67fc8-107">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="67fc8-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="67fc8-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="67fc8-108">Architecture</span></span>

<span data-ttu-id="67fc8-109">N 層アーキテクチャを実装する方法は多数あります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="67fc8-110">図は、典型的な 3 層 Web アプリケーションを示しています。</span><span class="sxs-lookup"><span data-stu-id="67fc8-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="67fc8-111">このアーキテクチャは「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」に基づいて作成されています。</span><span class="sxs-lookup"><span data-stu-id="67fc8-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="67fc8-112">Web 層とビジネス層では、負荷分散された VM が使用されます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="67fc8-113">**可用性セット。**</span><span class="sxs-lookup"><span data-stu-id="67fc8-113">**Availability sets.**</span></span> <span data-ttu-id="67fc8-114">階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span>  <span data-ttu-id="67fc8-115">こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="67fc8-116">可用性セット内の単一の VM をデプロイできますが、単一の VM がすべての OS およびデータ ディスクに Azure Premium Storage を使用していない限り、その単一の VM は SLA 保証に対して適格ではありません。</span><span class="sxs-lookup"><span data-stu-id="67fc8-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="67fc8-117">**サブネット。**</span><span class="sxs-lookup"><span data-stu-id="67fc8-117">**Subnets.**</span></span> <span data-ttu-id="67fc8-118">階層ごとに個別のサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="67fc8-119">[CIDR] 表記を使用してアドレス範囲とサブネット マスクを指定します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="67fc8-120">**ロード バランサー**。</span><span class="sxs-lookup"><span data-stu-id="67fc8-120">**Load balancers.**</span></span> <span data-ttu-id="67fc8-121">[インターネットに接続するロード バランサー][load-balancer-external]を使用して着信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="67fc8-122">**ジャンプボックス。**</span><span class="sxs-lookup"><span data-stu-id="67fc8-122">**Jumpbox.**</span></span> <span data-ttu-id="67fc8-123">[要塞ホスト]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-123">Also called a [bastion host].</span></span> <span data-ttu-id="67fc8-124">管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="67fc8-124">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="67fc8-125">ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-125">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="67fc8-126">NSG は Secure Shell (SSH) トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-126">The NSG should permit secure shell (SSH) traffic.</span></span>
* <span data-ttu-id="67fc8-127">**監視。**</span><span class="sxs-lookup"><span data-stu-id="67fc8-127">**Monitoring.**</span></span> <span data-ttu-id="67fc8-128">[Nagios]、[Zabbix]、[Icinga] などの監視ソフトウェアを使用して、応答時間、VM の稼働時間、システムの全体的な正常性に関する洞察を得ることができます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-128">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="67fc8-129">個別の管理サブネットに配置されている VM 上に監視ソフトウェアをインストールします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-129">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="67fc8-130">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="67fc8-130">**NSGs.**</span></span> <span data-ttu-id="67fc8-131">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-131">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="67fc8-132">たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-132">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="67fc8-133">**Apache Cassandra データベース**。</span><span class="sxs-lookup"><span data-stu-id="67fc8-133">**Apache Cassandra database**.</span></span> <span data-ttu-id="67fc8-134">レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-134">Provides high availability at the data tier, by enabling replication and failover.</span></span>

## <a name="recommendations"></a><span data-ttu-id="67fc8-135">Recommendations</span><span class="sxs-lookup"><span data-stu-id="67fc8-135">Recommendations</span></span>

<span data-ttu-id="67fc8-136">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-136">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="67fc8-137">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-137">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="67fc8-138">VNet/サブネット</span><span class="sxs-lookup"><span data-stu-id="67fc8-138">VNet / Subnets</span></span>

<span data-ttu-id="67fc8-139">VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-139">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="67fc8-140">[CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-140">Specify a subnet mask and a VNet address range large enough for the required IP addresses using [CIDR] notation.</span></span> <span data-ttu-id="67fc8-141">標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-141">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="67fc8-142">後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-142">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premises network later.</span></span> <span data-ttu-id="67fc8-143">VNet を作成した後は、アドレス範囲を変更できません。</span><span class="sxs-lookup"><span data-stu-id="67fc8-143">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="67fc8-144">機能とセキュリティの要件を念頭に置いてサブネットを設計します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-144">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="67fc8-145">同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-145">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="67fc8-146">VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-146">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="67fc8-147">各サブネットに対して、サブネットのアドレス空間を CIDR 表記で指定します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-147">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="67fc8-148">たとえば、"10.0.0.0/24" は IP アドレス 256 個の範囲を作成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-148">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="67fc8-149">VM ではこの内の 251 個を使用できます。5 個は予約されています。</span><span class="sxs-lookup"><span data-stu-id="67fc8-149">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="67fc8-150">アドレス範囲がサブネット間で重複しないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-150">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="67fc8-151">[Virtual Network に関する FAQ][vnet faq] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-151">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="67fc8-152">ネットワーク セキュリティ グループ</span><span class="sxs-lookup"><span data-stu-id="67fc8-152">Network security groups</span></span>

<span data-ttu-id="67fc8-153">NSG ルールを使用して階層間のトラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-153">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="67fc8-154">たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。</span><span class="sxs-lookup"><span data-stu-id="67fc8-154">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="67fc8-155">これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-155">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="67fc8-156">NSG を作成し、データベース層のサブネットに関連付けます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-156">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="67fc8-157">VNet からのすべての着信トラフィックを拒否するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-157">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="67fc8-158">(ルール内で `VIRTUAL_NETWORK` タグを使用します。)</span><span class="sxs-lookup"><span data-stu-id="67fc8-158">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="67fc8-159">ビジネス層のサブネットからの着信トラフィックを許可するルールを、より高い優先順位で追加します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-159">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="67fc8-160">このルールが前のルールを上書きし、ビジネス層がデータベース層と対話できるようになります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-160">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="67fc8-161">データベース層のサブネット自体からの着信トラフィックを許可するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-161">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="67fc8-162">このルールによって、データベースのレプリケーションやフェールオーバーに必要な、データベース層内の VM どうしの対話が可能になります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-162">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="67fc8-163">ジャンプボックスのサブネットからの SSH トラフィックを許可するルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-163">Add a rule that allows SSH traffic from the jumpbox subnet.</span></span> <span data-ttu-id="67fc8-164">このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-164">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="67fc8-165">NSG の[既定のルール][nsg-rules]では、VNet 内からの着信トラフィックがすべて許可されます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-165">An NSG has [default rules][nsg-rules] that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="67fc8-166">これらのルールは削除できませんが、より優先順位の高いルールを作成することで上書きできます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-166">These rules can't be deleted, but you can override them by creating higher-priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="67fc8-167">ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="67fc8-167">Load balancers</span></span>

<span data-ttu-id="67fc8-168">外部ロード バランサーは、インターネット トラフィックを Web 層に分散します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-168">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="67fc8-169">このロード バランサー用にパブリック IP アドレスを作成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-169">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="67fc8-170">[インターネットに接続するロード バランサーの作成][lb-external-create]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-170">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="67fc8-171">内部ロード バランサーは、Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-171">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="67fc8-172">このロード バランサーにプライベート IP アドレスを付与するには、フロントエンド IP 構成を作成し、ビジネス層のサブネットに関連付けます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-172">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="67fc8-173">[内部ロード バランサーの作成][lb-internal-create]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-173">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="cassandra"></a><span data-ttu-id="67fc8-174">Cassandra</span><span class="sxs-lookup"><span data-stu-id="67fc8-174">Cassandra</span></span>

<span data-ttu-id="67fc8-175">運用環境では [DataStax Enterprise][datastax] の使用をお勧めしますが、これらの推奨事項はすべての Cassandra エディションに適用されます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-175">We recommend [DataStax Enterprise][datastax] for production use, but these recommendations apply to any Cassandra edition.</span></span> <span data-ttu-id="67fc8-176">Azure での DataStax の実行の詳細については、「[Azure 用の DataStax Enterprise Deployment ガイド][cassandra-in-azure]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-176">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> 

<span data-ttu-id="67fc8-177">Cassandra クラスター用の VM を可用性セット内に配置して、Cassandra レプリカが複数の障害ドメインおよびアップグレード ドメイン間に分散されることを保証します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-177">Put the VMs for a Cassandra cluster in an availability set to ensure that the Cassandra replicas are distributed across multiple fault domains and upgrade domains.</span></span> <span data-ttu-id="67fc8-178">障害ドメインおよびアップグレード ドメインの詳細については、「[Virtual Machines の可用性管理][azure-availability-sets]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-178">For more information about fault domains and upgrade domains, see [Manage the availability of virtual machines][azure-availability-sets].</span></span> 

<span data-ttu-id="67fc8-179">可用性セットごとに 3 個の障害ドメイン (最大数) と 18 個のアップグレード ドメインを構成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-179">Configure three fault domains (the maximum) per availability set and 18 upgrade domains per availability set.</span></span> <span data-ttu-id="67fc8-180">これは、障害ドメイン間で引き続き均等に分散できるアップグレード ドメインの最大数を提供します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-180">This provides the maximum number of upgrade domains that can still be distributed evenly across the fault domains.</span></span>   

<span data-ttu-id="67fc8-181">ラック認識モードでノードを構成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-181">Configure nodes in rack-aware mode.</span></span> <span data-ttu-id="67fc8-182">`cassandra-rackdc.properties` ファイル内で障害ドメインをラックにマッピングします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-182">Map fault domains to racks in the `cassandra-rackdc.properties` file.</span></span>

<span data-ttu-id="67fc8-183">クラスターの前にロード バランサーは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="67fc8-183">You don't need a load balancer in front of the cluster.</span></span> <span data-ttu-id="67fc8-184">クライアントはクラスターのノードに直接接続します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-184">The client connects directly to a node in the cluster.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="67fc8-185">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="67fc8-185">Jumpbox</span></span>

<span data-ttu-id="67fc8-186">ジャンプボックスのパフォーマンス要件は最小限に抑えられるため、ジャンプボックスには Standard A1 などの小さな VM のサイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-186">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="67fc8-187">ジャンプボックス用に[パブリック IP アドレス]を作成します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-187">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="67fc8-188">ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-188">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="67fc8-189">アプリケーション ワークロードを実行する VM へのパブリック インターネットからの SSH アクセスを許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-189">Do not allow SSH access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="67fc8-190">代わりに、これらの VM へのすべての SSH アクセスは、ジャンプボックスを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-190">Instead, all SSH access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="67fc8-191">管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-191">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="67fc8-192">ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの SSH トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-192">The jumpbox allows SSH traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="67fc8-193">ジャンプボックスをセキュリティで保護するには、NSG を作成してジャンプボックスのサブネットに適用します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-193">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="67fc8-194">安全な一連のパブリック IP アドレスからのみ SSH 接続を許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-194">Add an NSG rule that allows SSH connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="67fc8-195">NSG は、サブネットまたはジャンプボックスの NIC のいずれかに関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-195">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="67fc8-196">この場合は NIC に関連付けることをお勧めします。こうすることで、仮に同じサブネットに別の VM を追加した場合でも、SSH トラフィックはジャンプボックスのみに許可されます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-196">In this case, we recommend attaching it to the NIC, so SSH traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="67fc8-197">他のサブネットに対しても NSG を構成して、管理サブネットからの SSH トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-197">Configure the NSGs for the other subnets to allow SSH traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="67fc8-198">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="67fc8-198">Availability considerations</span></span>

<span data-ttu-id="67fc8-199">各層または各 VM ロールを、個別の可用性セットに配置します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-199">Put each tier or VM role into a separate availability set.</span></span> 

<span data-ttu-id="67fc8-200">データベース層に複数の VM を備えても、自動的にデータベースの可用性が高くなるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="67fc8-200">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="67fc8-201">リレーショナル データベースの場合は、通常、高可用性を実現するにはレプリケーションとフェールオーバーを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-201">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span>  

<span data-ttu-id="67fc8-202">[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、2 つのリージョンにアプリケーションをレプリケートし、Azure Traffic Manager を使用してフェールオーバーを行います。</span><span class="sxs-lookup"><span data-stu-id="67fc8-202">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="67fc8-203">詳細については、「[高可用性を得るために複数のリージョンで Linux VM を実行する][multi-dc]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-203">For more information, see [Run Linux VMs in multiple regions for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="67fc8-204">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="67fc8-204">Security considerations</span></span>

<span data-ttu-id="67fc8-205">ネットワーク仮想アプライアンス (NVA) を追加してパブリック インターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-205">Consider adding a network virtual appliance (NVA) to create a DMZ between the public Internet and the Azure virtual network.</span></span> <span data-ttu-id="67fc8-206">NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。</span><span class="sxs-lookup"><span data-stu-id="67fc8-206">NVA is a generic term for a virtual appliance that can perform network-related tasks such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="67fc8-207">詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-207">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="67fc8-208">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="67fc8-208">Scalability considerations</span></span>

<span data-ttu-id="67fc8-209">ロード バランサーは、ネットワーク トラフィックを Web 層とビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-209">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="67fc8-210">新しい VM インスタンスを追加することで水平方向にスケーリングします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-210">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="67fc8-211">負荷に基づいて、Web 層とビジネス層を個別にスケーリングできることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-211">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="67fc8-212">クライアント アフィニティを維持するために発生する可能性がある複雑さを減らすには、Web 層の VM はステートレスである必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-212">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="67fc8-213">ビジネス ロジックをホストする VM もステートレスである必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-213">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="67fc8-214">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="67fc8-214">Manageability considerations</span></span>

<span data-ttu-id="67fc8-215">[Azure Automation][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef]、[Puppet][puppet] などの集中管理ツールを使用することで、システム全体の管理を簡略化します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-215">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="67fc8-216">これらのツールでは、複数の VM から取り込まれた診断情報と正常性情報を統合して、システムの全体像を提供できます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-216">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="67fc8-217">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="67fc8-217">Deploy the solution</span></span>

<span data-ttu-id="67fc8-218">このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-218">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="67fc8-219">前提条件</span><span class="sxs-lookup"><span data-stu-id="67fc8-219">Prerequisites</span></span>

<span data-ttu-id="67fc8-220">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-220">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="67fc8-221">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-221">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="67fc8-222">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-222">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="67fc8-223">CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-223">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="67fc8-224">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-224">Install the [Azure building blocks][azbb] npm package.</span></span>

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. <span data-ttu-id="67fc8-225">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="67fc8-225">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="67fc8-226">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="67fc8-226">Deploy the solution using azbb</span></span>

<span data-ttu-id="67fc8-227">N 層アプリケーションの参照アーキテクチャで Linux VM をデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="67fc8-227">To deploy the Linux VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="67fc8-228">上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-228">Navigate to the `virtual-machines\n-tier-linux` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="67fc8-229">このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-229">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="67fc8-230">参照アーキテクチャをデプロイする前に、これらを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="67fc8-230">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="67fc8-231">`n-tier-linux.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="67fc8-231">Open the `n-tier-linux.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>   <span data-ttu-id="67fc8-232">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="67fc8-232">Save the file.</span></span>

3. <span data-ttu-id="67fc8-233">次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="67fc8-233">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
  ```

<span data-ttu-id="67fc8-234">Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="67fc8-234">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "Microsoft Azure を使用した N 層アーキテクチャ"

