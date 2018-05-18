---
title: SQL Server を使用した n 層アプリケーション
description: 可用性、セキュリティ、スケーラビリティ、および管理容易性のために Azure で多層アーキテクチャを実装する方法について説明します。
author: MikeWasson
ms.date: 05/03/2018
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0f170f2fbcbbfeace53db199cb5d3949415b5546
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2018
---
# <a name="n-tier-application-with-sql-server"></a><span data-ttu-id="a5226-103">SQL Server を使用した n 層アプリケーション</span><span class="sxs-lookup"><span data-stu-id="a5226-103">N-tier application with SQL Server</span></span>

<span data-ttu-id="a5226-104">この参照アーキテクチャでは、Windows 上の SQL Server をデータ層に使用して、N 層アプリケーション用に構成された VM と仮想ネットワークをデプロイする方法を示します。</span><span class="sxs-lookup"><span data-stu-id="a5226-104">This reference architecture shows how to deploy VMs and a virtual network configured for an N-tier application, using SQL Server on Windows for the data tier.</span></span> [<span data-ttu-id="a5226-105">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="a5226-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="a5226-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="a5226-106">![[0]][0]</span></span>

<span data-ttu-id="a5226-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="a5226-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="a5226-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="a5226-108">Architecture</span></span> 

<span data-ttu-id="a5226-109">アーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="a5226-109">The architecture has the following components:</span></span>

* <span data-ttu-id="a5226-110">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="a5226-110">**Resource group.**</span></span> <span data-ttu-id="a5226-111">[リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="a5226-111">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>

* <span data-ttu-id="a5226-112">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="a5226-112">**Virtual network (VNet) and subnets.**</span></span> <span data-ttu-id="a5226-113">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="a5226-113">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span> <span data-ttu-id="a5226-114">階層ごとに個別のサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-114">Create a separate subnet for each tier.</span></span> 

* <span data-ttu-id="a5226-115">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="a5226-115">**NSGs.**</span></span> <span data-ttu-id="a5226-116">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="a5226-116">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="a5226-117">たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。</span><span class="sxs-lookup"><span data-stu-id="a5226-117">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>

* <span data-ttu-id="a5226-118">**仮想マシン**。</span><span class="sxs-lookup"><span data-stu-id="a5226-118">**Virtual machines**.</span></span> <span data-ttu-id="a5226-119">VM の構成に関する推奨事項については、「[Run a Windows VM on Azure (Azure での Windows VM の実行)](./windows-vm.md)」および「[Run a Linux VM on Azure (Azure での Linux VM の実行)](./linux-vm.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a5226-119">For recommendations on configuring VMs, see [Run a Windows VM on Azure](./windows-vm.md) and [Run a Linux VM on Azure](./linux-vm.md).</span></span>

* <span data-ttu-id="a5226-120">**可用性セット。**</span><span class="sxs-lookup"><span data-stu-id="a5226-120">**Availability sets.**</span></span> <span data-ttu-id="a5226-121">階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="a5226-121">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="a5226-122">こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="a5226-122">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> 

* <span data-ttu-id="a5226-123">**VM スケール セット** (ここでは示されていません)。</span><span class="sxs-lookup"><span data-stu-id="a5226-123">**VM scale set** (not shown).</span></span> <span data-ttu-id="a5226-124">[VM スケール セット][vmss]は可用性セットの代わりに使用します。</span><span class="sxs-lookup"><span data-stu-id="a5226-124">A [VM scale set][vmss] is an alternative to using an availability set.</span></span> <span data-ttu-id="a5226-125">スケール セットを使用すると、層内の VM を簡単にスケールアウトできます。スケールアウトは、手動で実行することも、定義済みの規則に基づいて自動的に実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="a5226-125">A scale sets makes it easy to scale out the VMs in a tier, either manually or automatically based on predefined rules.</span></span>

* <span data-ttu-id="a5226-126">**Azure Load Balancer。**</span><span class="sxs-lookup"><span data-stu-id="a5226-126">**Azure Load balancers.**</span></span> <span data-ttu-id="a5226-127">[ロード バランサー][load-balancer]は、受信インターネット要求を各 VM インスタンスに分散します。</span><span class="sxs-lookup"><span data-stu-id="a5226-127">The [load balancers][load-balancer] distribute incoming Internet requests to the VM instances.</span></span> <span data-ttu-id="a5226-128">[パブリック ロード バランサー][load-balancer-external]を使用して受信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="a5226-128">Use a [public load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>

* <span data-ttu-id="a5226-129">**パブリック IP アドレス**。</span><span class="sxs-lookup"><span data-stu-id="a5226-129">**Public IP address**.</span></span> <span data-ttu-id="a5226-130">パブリック IP アドレスは、パブリック ロード バランサーがインターネット トラフィックを受信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="a5226-130">A public IP address is needed for the public load balancer to receive Internet traffic.</span></span>

* <span data-ttu-id="a5226-131">**ジャンプボックス。**</span><span class="sxs-lookup"><span data-stu-id="a5226-131">**Jumpbox.**</span></span> <span data-ttu-id="a5226-132">[要塞ホスト]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="a5226-132">Also called a [bastion host].</span></span> <span data-ttu-id="a5226-133">管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="a5226-133">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="a5226-134">ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-134">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="a5226-135">NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-135">The NSG should permit remote desktop (RDP) traffic.</span></span>

* <span data-ttu-id="a5226-136">**SQL Server Always On 可用性グループ。**</span><span class="sxs-lookup"><span data-stu-id="a5226-136">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="a5226-137">レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。</span><span class="sxs-lookup"><span data-stu-id="a5226-137">Provides high availability at the data tier, by enabling replication and failover.</span></span>

* <span data-ttu-id="a5226-138">**Active Directory Domain Services (AD DS) サーバー。**</span><span class="sxs-lookup"><span data-stu-id="a5226-138">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="a5226-139">SQL Server Always On 可用性グループは、Windows Server フェールオーバー クラスター (WSFC) テクノロジでフェールオーバーできるように、ドメインに参加しています。</span><span class="sxs-lookup"><span data-stu-id="a5226-139">The SQL Server Always On Availability Groups are joined to a domain, to enable the Windows Server Failover Cluster (WSFC) technology for failover.</span></span> 

* <span data-ttu-id="a5226-140">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="a5226-140">**Azure DNS**.</span></span> <span data-ttu-id="a5226-141">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="a5226-141">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="a5226-142">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="a5226-142">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="a5226-143">Recommendations</span><span class="sxs-lookup"><span data-stu-id="a5226-143">Recommendations</span></span>

<span data-ttu-id="a5226-144">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-144">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="a5226-145">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-145">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="a5226-146">VNet/サブネット</span><span class="sxs-lookup"><span data-stu-id="a5226-146">VNet / Subnets</span></span>

<span data-ttu-id="a5226-147">VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。</span><span class="sxs-lookup"><span data-stu-id="a5226-147">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="a5226-148">[CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。</span><span class="sxs-lookup"><span data-stu-id="a5226-148">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="a5226-149">標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。</span><span class="sxs-lookup"><span data-stu-id="a5226-149">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="a5226-150">後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。</span><span class="sxs-lookup"><span data-stu-id="a5226-150">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="a5226-151">VNet を作成した後は、アドレス範囲を変更できません。</span><span class="sxs-lookup"><span data-stu-id="a5226-151">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="a5226-152">機能とセキュリティの要件を念頭に置いてサブネットを設計します。</span><span class="sxs-lookup"><span data-stu-id="a5226-152">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="a5226-153">同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。</span><span class="sxs-lookup"><span data-stu-id="a5226-153">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="a5226-154">VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-154">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

### <a name="load-balancers"></a><span data-ttu-id="a5226-155">ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="a5226-155">Load balancers</span></span>

<span data-ttu-id="a5226-156">VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。</span><span class="sxs-lookup"><span data-stu-id="a5226-156">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="a5226-157">クライアントは、パブリック ロード バランサーの IP アドレスを使用して接続します。</span><span class="sxs-lookup"><span data-stu-id="a5226-157">Clients connect using the IP address of the public load balancer.</span></span>

<span data-ttu-id="a5226-158">ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。</span><span class="sxs-lookup"><span data-stu-id="a5226-158">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="a5226-159">たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-159">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="a5226-160">クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。</span><span class="sxs-lookup"><span data-stu-id="a5226-160">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="a5226-161">この方法で、クライアント要求がすべての VM に配布されます。</span><span class="sxs-lookup"><span data-stu-id="a5226-161">In that way, client requests are distributed across all the VMs.</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="a5226-162">ネットワーク セキュリティ グループ</span><span class="sxs-lookup"><span data-stu-id="a5226-162">Network security groups</span></span>

<span data-ttu-id="a5226-163">NSG ルールを使用して階層間のトラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="a5226-163">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="a5226-164">たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。</span><span class="sxs-lookup"><span data-stu-id="a5226-164">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="a5226-165">これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-165">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="a5226-166">VNet からのすべての受信トラフィックを拒否します。</span><span class="sxs-lookup"><span data-stu-id="a5226-166">Deny all inbound traffic from the VNet.</span></span> <span data-ttu-id="a5226-167">(ルール内で `VIRTUAL_NETWORK` タグを使用します。)</span><span class="sxs-lookup"><span data-stu-id="a5226-167">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
2. <span data-ttu-id="a5226-168">ビジネス層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-168">Allow inbound traffic from the business tier subnet.</span></span>  
3. <span data-ttu-id="a5226-169">データベース層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-169">Allow inbound traffic from the database tier subnet itself.</span></span> <span data-ttu-id="a5226-170">このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。</span><span class="sxs-lookup"><span data-stu-id="a5226-170">This rule allows communication between the database VMs, which is needed for database replication and failover.</span></span>
4. <span data-ttu-id="a5226-171">ジャンプボックスのサブネットからの RDP トラフィック (ポート 3389) を許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-171">Allow RDP traffic (port 3389) from the jumpbox subnet.</span></span> <span data-ttu-id="a5226-172">このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。</span><span class="sxs-lookup"><span data-stu-id="a5226-172">This rule lets administrators connect to the database tier from the jumpbox.</span></span>

<span data-ttu-id="a5226-173">最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。</span><span class="sxs-lookup"><span data-stu-id="a5226-173">Create rules 2 &ndash; 4 with higher priority than the first rule, so they override it.</span></span>


### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="a5226-174">SQL Server Always On 可用性グループ</span><span class="sxs-lookup"><span data-stu-id="a5226-174">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="a5226-175">SQL Server の高可用性のために [Always On 可用性グループ][sql-alwayson]の使用をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="a5226-175">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="a5226-176">Windows Server 2016 に先立って、Always On 可用性グループはドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ AD ドメイン内にある必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-176">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="a5226-177">他の層は[可用性グループ リスナー][sql-alwayson-listeners]を使用してデータベースに接続します。</span><span class="sxs-lookup"><span data-stu-id="a5226-177">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="a5226-178">リスナーを使用することで、SQL クライアントは SQL Server の物理インスタンスの名前を知らなくても接続できます。</span><span class="sxs-lookup"><span data-stu-id="a5226-178">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="a5226-179">データベースにアクセスする VM はドメインに参加している必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-179">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="a5226-180">クライアント (ここでは、別の層) は、DNS を使用してリスナーの仮想ネットワーク名を IP アドレスに解決します。</span><span class="sxs-lookup"><span data-stu-id="a5226-180">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="a5226-181">SQL Server Always On 可用性グループを構成する手順は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="a5226-181">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="a5226-182">Windows Server フェールオーバー クラスタリング (WSFC) クラスター、SQL Server Always On 可用性グループ、プライマリ レプリカを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-182">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="a5226-183">詳細については、「[AlwaysOn 可用性グループの概要][sql-alwayson-getting-started]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-183">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="a5226-184">静的プライベート IP アドレスを持つ内部ロード バランサーを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-184">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="a5226-185">可用性グループ リスナーを作成し、リスナーの DNS 名を内部ロード バランサーの IP アドレスにマッピングします。</span><span class="sxs-lookup"><span data-stu-id="a5226-185">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="a5226-186">SQL Server リスニング ポート (既定ではTCP ポート 1433) に対するロード バランサーのルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-186">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="a5226-187">ロード バランサーのルールでは *Floating IP* (Direct Server Return とも呼ばれます) を有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-187">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="a5226-188">これにより VM が直接クライアントに応答でき、プライマリ レプリカへの直接接続が可能になります。</span><span class="sxs-lookup"><span data-stu-id="a5226-188">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="a5226-189">Floating IP が有効になっている場合は、フロントエンド ポート番号を、ロード バランサーのルール内のバックエンド ポート番号と同じにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-189">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
   > 
   > 

<span data-ttu-id="a5226-190">SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。</span><span class="sxs-lookup"><span data-stu-id="a5226-190">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="a5226-191">別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーはその後の要求を自動的に新しいプライマリ レプリカにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="a5226-191">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="a5226-192">詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成][sql-alwayson-ilb]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-192">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="a5226-193">フェールオーバー中は、既存のクライアント接続は閉じられます。</span><span class="sxs-lookup"><span data-stu-id="a5226-193">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="a5226-194">フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="a5226-194">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="a5226-195">アプリケーションが書き込みよりもはるかに多くの読み取りを行う場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。</span><span class="sxs-lookup"><span data-stu-id="a5226-195">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="a5226-196">「[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]」(リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-196">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="a5226-197">可用性グループの[手動フェールオーバーの強制][sql-alwayson-force-failover]によってデプロイをテストします。</span><span class="sxs-lookup"><span data-stu-id="a5226-197">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="a5226-198">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="a5226-198">Jumpbox</span></span>

<span data-ttu-id="a5226-199">アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="a5226-199">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="a5226-200">代わりに、これらの VM へのすべての RDP アクセスは、ジャンプボックスを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-200">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="a5226-201">管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="a5226-201">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="a5226-202">ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-202">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="a5226-203">ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="a5226-203">The jumpbox has minimal performance requirements, so select a small VM size.</span></span> <span data-ttu-id="a5226-204">ジャンプボックス用に[パブリック IP アドレス]を作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-204">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="a5226-205">ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。</span><span class="sxs-lookup"><span data-stu-id="a5226-205">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="a5226-206">ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="a5226-206">To secure the jumpbox, add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="a5226-207">他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="a5226-207">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="a5226-208">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="a5226-208">Scalability considerations</span></span>

<span data-ttu-id="a5226-209">[VM スケール セット][vmss]を使用して、同一の VM のセットをデプロイして管理できます。</span><span class="sxs-lookup"><span data-stu-id="a5226-209">[VM scale sets][vmss] help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="a5226-210">スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="a5226-210">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="a5226-211">VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。</span><span class="sxs-lookup"><span data-stu-id="a5226-211">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="a5226-212">VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-212">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="a5226-213">スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-213">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="a5226-214">拡張機能を使用してプロビジョニングされた後に VM を構成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-214">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="a5226-215">この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-215">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="a5226-216">カスタム ディスク イメージと共に [Managed Disk](/azure/storage/storage-managed-disks-overview) をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="a5226-216">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="a5226-217">このオプションの方が早くデプロイできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-217">This option may be quicker to deploy.</span></span> <span data-ttu-id="a5226-218">ただし、イメージを最新の状態に保つ必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-218">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="a5226-219">詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-219">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="a5226-220">自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。</span><span class="sxs-lookup"><span data-stu-id="a5226-220">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="a5226-221">各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-221">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="a5226-222">サポート リクエストを提出することで、制限値を上げることができます。</span><span class="sxs-lookup"><span data-stu-id="a5226-222">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="a5226-223">詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a5226-223">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="a5226-224">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="a5226-224">Availability considerations</span></span>

<span data-ttu-id="a5226-225">VM スケール セットを使用していない場合は、同じ層の VM を可用性セットに配置します。</span><span class="sxs-lookup"><span data-stu-id="a5226-225">If you are not using VM scale sets, put VMs in the same tier into an availability set.</span></span> <span data-ttu-id="a5226-226">[Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-226">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="a5226-227">詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-227">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> 

<span data-ttu-id="a5226-228">ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。</span><span class="sxs-lookup"><span data-stu-id="a5226-228">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="a5226-229">タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。</span><span class="sxs-lookup"><span data-stu-id="a5226-229">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="a5226-230">ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。</span><span class="sxs-lookup"><span data-stu-id="a5226-230">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="a5226-231">ロード バランサーの正常性プローブには、次のような推奨事項があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-231">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="a5226-232">プローブでは、HTTP または TCP のいずれかをテストできます。</span><span class="sxs-lookup"><span data-stu-id="a5226-232">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="a5226-233">VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-233">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="a5226-234">それ以外の場合は、TCP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5226-234">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="a5226-235">HTTP プローブでは、HTTP エンドポイントへのパスを指定します。</span><span class="sxs-lookup"><span data-stu-id="a5226-235">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="a5226-236">プローブでは、このパスからの HTTP 200 の応答をチェックします。</span><span class="sxs-lookup"><span data-stu-id="a5226-236">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="a5226-237">これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。</span><span class="sxs-lookup"><span data-stu-id="a5226-237">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="a5226-238">エンドポイントでは、匿名の HTTP 要求を許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-238">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="a5226-239">このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。</span><span class="sxs-lookup"><span data-stu-id="a5226-239">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="a5226-240">この IP アドレスとの間のトラフィックを、どのファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) 規則でもブロックしていないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-240">Make sure you don't block traffic to or from this IP address in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="a5226-241">[正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。</span><span class="sxs-lookup"><span data-stu-id="a5226-241">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="a5226-242">各ロード バランサーに対して Azure Portal のログ記録を有効にします。</span><span class="sxs-lookup"><span data-stu-id="a5226-242">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="a5226-243">ログは Azure Blob Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="a5226-243">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="a5226-244">ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。</span><span class="sxs-lookup"><span data-stu-id="a5226-244">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

<span data-ttu-id="a5226-245">[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、フェールオーバーに Azure Traffic Manager を使用して、2 つのリージョンでのアプリケーションのレプリケーションを検討します。</span><span class="sxs-lookup"><span data-stu-id="a5226-245">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, consider replication the application across two regions, using Azure Traffic Manager for failover.</span></span> <span data-ttu-id="a5226-246">詳細については、「[高可用性のためのマルチリージョン n 層アプリケーション][multi-dc]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-246">For more information, see [Multi-region N-tier application for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="a5226-247">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="a5226-247">Security considerations</span></span>

<span data-ttu-id="a5226-248">仮想ネットワークは、Azure のトラフィックの分離境界です。</span><span class="sxs-lookup"><span data-stu-id="a5226-248">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="a5226-249">ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。</span><span class="sxs-lookup"><span data-stu-id="a5226-249">VMs in one VNet cannot communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="a5226-250">トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。</span><span class="sxs-lookup"><span data-stu-id="a5226-250">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="a5226-251">詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a5226-251">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="a5226-252">インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。</span><span class="sxs-lookup"><span data-stu-id="a5226-252">For incoming Internet traffic, the load balancer rules define which traffic can reach the back end.</span></span> <span data-ttu-id="a5226-253">しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-253">However, load balancer rules don't support IP safe lists, so if you want to add certain public IP addresses to a safe list, add an NSG to the subnet.</span></span>

<span data-ttu-id="a5226-254">ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-254">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="a5226-255">NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。</span><span class="sxs-lookup"><span data-stu-id="a5226-255">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="a5226-256">詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-256">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

<span data-ttu-id="a5226-257">機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。</span><span class="sxs-lookup"><span data-stu-id="a5226-257">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="a5226-258">Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。</span><span class="sxs-lookup"><span data-stu-id="a5226-258">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="a5226-259">詳細については、[Azure VM 上の SQL Server 向け Azure Key Vault 統合の構成][sql-keyvault]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-259">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault].</span></span> <span data-ttu-id="a5226-260">データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="a5226-260">It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="a5226-261">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="a5226-261">Deploy the solution</span></span>

<span data-ttu-id="a5226-262">このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-262">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="a5226-263">前提条件</span><span class="sxs-lookup"><span data-stu-id="a5226-263">Prerequisites</span></span>

1. <span data-ttu-id="a5226-264">[参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="a5226-264">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="a5226-265">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-265">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="a5226-266">CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="a5226-266">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="a5226-267">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="a5226-267">Install the [Azure building blocks][azbb] npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="a5226-268">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="a5226-268">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="a5226-269">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="a5226-269">Deploy the solution using azbb</span></span>

<span data-ttu-id="a5226-270">N 層アプリケーションの参照アーキテクチャで Windows VM をデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="a5226-270">To deploy the Windows VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="a5226-271">上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-windows` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="a5226-271">Navigate to the `virtual-machines\n-tier-windows` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="a5226-272">このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。</span><span class="sxs-lookup"><span data-stu-id="a5226-272">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="a5226-273">参照アーキテクチャをデプロイする前に、これらを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-273">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="a5226-274">`n-tier-windows.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="a5226-274">Open the `n-tier-windows.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="a5226-275">このデプロイ中に実行される複数のスクリプトが **VirtualMachineExtension** オブジェクトと、一部の **VirtualMachine** オブジェクトの **extensions** 設定の両方に存在します。</span><span class="sxs-lookup"><span data-stu-id="a5226-275">There are multiple scripts that run during this deployment both in the  **VirtualMachineExtension** objects and in the **extensions** settings for some of the **VirtualMachine** objects.</span></span> <span data-ttu-id="a5226-276">これらのスクリプトのいくつかには、今変更した管理者ユーザー名とパスワードが必要です。</span><span class="sxs-lookup"><span data-stu-id="a5226-276">Some of these scripts require the administrator user name and password that you have just changed.</span></span> <span data-ttu-id="a5226-277">これらのスクリプトをレビューして、正しい資格情報を指定したことを確認することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="a5226-277">It's recommended that you review these scripts to ensure that you specified the correct credentials.</span></span> <span data-ttu-id="a5226-278">正しい資格情報を指定していない場合は、デプロイが失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5226-278">The deployment may fail if you have not specified the correct credentials.</span></span>
   > 
   > 

<span data-ttu-id="a5226-279">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="a5226-279">Save the file.</span></span>

3. <span data-ttu-id="a5226-280">次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="a5226-280">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
   ```

<span data-ttu-id="a5226-281">Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5226-281">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
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
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "Microsoft Azure を使用した N 層アーキテクチャ"
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
