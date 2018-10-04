---
title: SQL Server を使用した n 層アプリケーション
description: 可用性、セキュリティ、スケーラビリティ、および管理容易性のために Azure で多層アーキテクチャを実装する方法について説明します。
author: MikeWasson
ms.date: 07/19/2018
ms.openlocfilehash: d5f46570464916af892afcf492127402181a5df5
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429147"
---
# <a name="n-tier-application-with-sql-server"></a><span data-ttu-id="d13e6-103">SQL Server を使用した n 層アプリケーション</span><span class="sxs-lookup"><span data-stu-id="d13e6-103">N-tier application with SQL Server</span></span>

<span data-ttu-id="d13e6-104">この参照アーキテクチャでは、Windows 上の SQL Server をデータ層に使用して、N 層アプリケーション用に構成された VM と仮想ネットワークをデプロイする方法を示します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-104">This reference architecture shows how to deploy VMs and a virtual network configured for an N-tier application, using SQL Server on Windows for the data tier.</span></span> [<span data-ttu-id="d13e6-105">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="d13e6-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="d13e6-106">![[0]][0]</span></span>

<span data-ttu-id="d13e6-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="d13e6-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="d13e6-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="d13e6-108">Architecture</span></span> 

<span data-ttu-id="d13e6-109">アーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-109">The architecture has the following components:</span></span>

* <span data-ttu-id="d13e6-110">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-110">**Resource group.**</span></span> <span data-ttu-id="d13e6-111">[リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-111">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>

* <span data-ttu-id="d13e6-112">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-112">**Virtual network (VNet) and subnets.**</span></span> <span data-ttu-id="d13e6-113">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-113">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span> <span data-ttu-id="d13e6-114">階層ごとに個別のサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-114">Create a separate subnet for each tier.</span></span> 

* <span data-ttu-id="d13e6-115">**Application Gateway**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-115">**Application gateway**.</span></span> <span data-ttu-id="d13e6-116">[Azure Application Gateway](/azure/application-gateway/) はレイヤー 7 のロード バランサーです。</span><span class="sxs-lookup"><span data-stu-id="d13e6-116">[Azure Application Gateway](/azure/application-gateway/) is a layer 7 load balancer.</span></span> <span data-ttu-id="d13e6-117">このアーキテクチャでは、これによって HTTP 要求が Web フロント エンドにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-117">In this architecture, it routes HTTP requests to the web front end.</span></span> <span data-ttu-id="d13e6-118">Application Gateway には、一般的な脆弱性やその悪用からアプリケーションを保護する [Web アプリケーション ファイアウォール](/azure/application-gateway/waf-overview) (WAF) も用意されています。</span><span class="sxs-lookup"><span data-stu-id="d13e6-118">Application Gateway also provides a [web application firewall](/azure/application-gateway/waf-overview) (WAF) that protects the application from common exploits and vulnerabilities.</span></span> 

* <span data-ttu-id="d13e6-119">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-119">**NSGs.**</span></span> <span data-ttu-id="d13e6-120">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-120">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="d13e6-121">たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-121">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>

* <span data-ttu-id="d13e6-122">**仮想マシン**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-122">**Virtual machines**.</span></span> <span data-ttu-id="d13e6-123">VM の構成に関する推奨事項については、「[Run a Windows VM on Azure (Azure での Windows VM の実行)](./windows-vm.md)」および「[Run a Linux VM on Azure (Azure での Linux VM の実行)](./linux-vm.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-123">For recommendations on configuring VMs, see [Run a Windows VM on Azure](./windows-vm.md) and [Run a Linux VM on Azure](./linux-vm.md).</span></span>

* <span data-ttu-id="d13e6-124">**可用性セット。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-124">**Availability sets.**</span></span> <span data-ttu-id="d13e6-125">階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-125">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="d13e6-126">こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-126">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> 

* <span data-ttu-id="d13e6-127">**VM スケール セット** (ここでは示されていません)。</span><span class="sxs-lookup"><span data-stu-id="d13e6-127">**VM scale set** (not shown).</span></span> <span data-ttu-id="d13e6-128">[VM スケール セット][vmss]は可用性セットの代わりに使用します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-128">A [VM scale set][vmss] is an alternative to using an availability set.</span></span> <span data-ttu-id="d13e6-129">スケール セットを使用すると、層内の VM を簡単にスケールアウトできます。スケールアウトは、手動で実行することも、定義済みの規則に基づいて自動的に実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-129">A scale sets makes it easy to scale out the VMs in a tier, either manually or automatically based on predefined rules.</span></span>

* <span data-ttu-id="d13e6-130">**ロード バランサー**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-130">**Load balancers.**</span></span> <span data-ttu-id="d13e6-131">[Azure Load Balancer][load-balancer] は、Web 層からビジネス層へ、ビジネス層から SQL Server へとネットワーク トラフィックを分散するために使用します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-131">Use [Azure Load Balancer][load-balancer] to distribute network traffic from the web tier to the business tier, and from the business tier to SQL Server.</span></span>

* <span data-ttu-id="d13e6-132">**パブリック IP アドレス**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-132">**Public IP address**.</span></span> <span data-ttu-id="d13e6-133">パブリック IP アドレスは、アプリケーションがインターネット トラフィックを受信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-133">A public IP address is needed for the application to receive Internet traffic.</span></span>

* <span data-ttu-id="d13e6-134">**ジャンプボックス。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-134">**Jumpbox.**</span></span> <span data-ttu-id="d13e6-135">[要塞ホスト]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-135">Also called a [bastion host].</span></span> <span data-ttu-id="d13e6-136">管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-136">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="d13e6-137">ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-137">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="d13e6-138">NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-138">The NSG should permit remote desktop (RDP) traffic.</span></span>

* <span data-ttu-id="d13e6-139">**SQL Server Always On 可用性グループ。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-139">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="d13e6-140">レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-140">Provides high availability at the data tier, by enabling replication and failover.</span></span> <span data-ttu-id="d13e6-141">これは、Windows Server フェールオーバー クラスター (WSFC) テクノロジを使用してフェールオーバーを行います。</span><span class="sxs-lookup"><span data-stu-id="d13e6-141">It uses Windows Server Failover Cluster (WSFC) technology for failover.</span></span>

* <span data-ttu-id="d13e6-142">**Active Directory Domain Services (AD DS) サーバー。**</span><span class="sxs-lookup"><span data-stu-id="d13e6-142">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="d13e6-143">フェールオーバー クラスターと、これに関連するクラスター化されたロールを表すコンピューター オブジェクトは、Active Directory Domain Services (AD DS) に作成されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-143">The computer objects for the failover cluster and its associated clustered roles are created in Active Directory Domain Services (AD DS).</span></span>

* <span data-ttu-id="d13e6-144">**クラウド監視**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-144">**Cloud Witness**.</span></span> <span data-ttu-id="d13e6-145">フェールオーバー クラスターでは、そのノードの半数以上が実行されている必要があり、"クォーラムに達している" と呼びます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-145">A failover cluster requires more than half of its nodes to be running, which is known as having quorum.</span></span> <span data-ttu-id="d13e6-146">クラスターにノードが 2 つしかない場合は、ネットワークのパーティションにより、各ノードは自身がマスター ノードであると認識する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-146">If the cluster has just two nodes, a network partition could cause each node to think it's the master node.</span></span> <span data-ttu-id="d13e6-147">この場合、"*監視*" によって優先順位を決定し、クォーラムを確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-147">In that case, you need a *witness* to break ties and establish quorum.</span></span> <span data-ttu-id="d13e6-148">監視は、クォーラムを確立する際の優先順位決定者として動作可能な、共有ディスクなどのリソースです。</span><span class="sxs-lookup"><span data-stu-id="d13e6-148">A witness is a resource such as a shared disk that can act as a tie breaker to establish quorum.</span></span> <span data-ttu-id="d13e6-149">クラウド監視は、Azure Blob Storage を使用する一種の監視です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-149">Cloud Witness is a type of witness that uses Azure Blob Storage.</span></span> <span data-ttu-id="d13e6-150">クォーラムの概念の詳細については、「[Understanding cluster and pool quorum (クラスターとプール クォーラムについて)](/windows-server/storage/storage-spaces/understand-quorum)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-150">To learn more about the concept of quorum, see [Understanding cluster and pool quorum](/windows-server/storage/storage-spaces/understand-quorum).</span></span> <span data-ttu-id="d13e6-151">クラウド監視の詳細については、「[Deploy a cloud witness for a Failover Cluster (フェールオーバー クラスター用にクラウド監視をデプロイする)](/windows-server/failover-clustering/deploy-cloud-witness)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-151">For more information about Cloud Witness, see [Deploy a Cloud Witness for a Failover Cluster](/windows-server/failover-clustering/deploy-cloud-witness).</span></span> 

* <span data-ttu-id="d13e6-152">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="d13e6-152">**Azure DNS**.</span></span> <span data-ttu-id="d13e6-153">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-153">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="d13e6-154">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-154">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d13e6-155">Recommendations</span><span class="sxs-lookup"><span data-stu-id="d13e6-155">Recommendations</span></span>

<span data-ttu-id="d13e6-156">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-156">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="d13e6-157">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-157">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="d13e6-158">VNet/サブネット</span><span class="sxs-lookup"><span data-stu-id="d13e6-158">VNet / Subnets</span></span>

<span data-ttu-id="d13e6-159">VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-159">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="d13e6-160">[CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-160">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="d13e6-161">標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-161">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="d13e6-162">後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-162">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="d13e6-163">VNet を作成した後は、アドレス範囲を変更できません。</span><span class="sxs-lookup"><span data-stu-id="d13e6-163">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="d13e6-164">機能とセキュリティの要件を念頭に置いてサブネットを設計します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-164">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="d13e6-165">同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-165">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="d13e6-166">VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-166">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

### <a name="load-balancers"></a><span data-ttu-id="d13e6-167">ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="d13e6-167">Load balancers</span></span>

<span data-ttu-id="d13e6-168">VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-168">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="d13e6-169">クライアントは、Application Gateway に関連付けられているパブリック IP アドレスを使用して接続します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-169">Clients connect using the public IP address  associated with the Application Gateway.</span></span>

<span data-ttu-id="d13e6-170">ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-170">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="d13e6-171">たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-171">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="d13e6-172">クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-172">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="d13e6-173">この方法で、クライアント要求がすべての VM に配布されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-173">In that way, client requests are distributed across all the VMs.</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="d13e6-174">ネットワーク セキュリティ グループ</span><span class="sxs-lookup"><span data-stu-id="d13e6-174">Network security groups</span></span>

<span data-ttu-id="d13e6-175">NSG ルールを使用して階層間のトラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-175">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="d13e6-176">たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。</span><span class="sxs-lookup"><span data-stu-id="d13e6-176">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="d13e6-177">これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-177">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="d13e6-178">VNet からのすべての受信トラフィックを拒否します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-178">Deny all inbound traffic from the VNet.</span></span> <span data-ttu-id="d13e6-179">(ルール内で `VIRTUAL_NETWORK` タグを使用します。)</span><span class="sxs-lookup"><span data-stu-id="d13e6-179">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
2. <span data-ttu-id="d13e6-180">ビジネス層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-180">Allow inbound traffic from the business tier subnet.</span></span>  
3. <span data-ttu-id="d13e6-181">データベース層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-181">Allow inbound traffic from the database tier subnet itself.</span></span> <span data-ttu-id="d13e6-182">このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-182">This rule allows communication between the database VMs, which is needed for database replication and failover.</span></span>
4. <span data-ttu-id="d13e6-183">ジャンプボックスのサブネットからの RDP トラフィック (ポート 3389) を許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-183">Allow RDP traffic (port 3389) from the jumpbox subnet.</span></span> <span data-ttu-id="d13e6-184">このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-184">This rule lets administrators connect to the database tier from the jumpbox.</span></span>

<span data-ttu-id="d13e6-185">最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-185">Create rules 2 &ndash; 4 with higher priority than the first rule, so they override it.</span></span>


### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="d13e6-186">SQL Server Always On 可用性グループ</span><span class="sxs-lookup"><span data-stu-id="d13e6-186">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="d13e6-187">SQL Server の高可用性のために [Always On 可用性グループ][sql-alwayson]の使用をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-187">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="d13e6-188">Windows Server 2016 に先立って、Always On 可用性グループはドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ AD ドメイン内にある必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-188">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="d13e6-189">他の層は[可用性グループ リスナー][sql-alwayson-listeners]を使用してデータベースに接続します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-189">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="d13e6-190">リスナーを使用することで、SQL クライアントは SQL Server の物理インスタンスの名前を知らなくても接続できます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-190">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="d13e6-191">データベースにアクセスする VM はドメインに参加している必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-191">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="d13e6-192">クライアント (ここでは、別の層) は、DNS を使用してリスナーの仮想ネットワーク名を IP アドレスに解決します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-192">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="d13e6-193">SQL Server Always On 可用性グループを構成する手順は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="d13e6-193">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="d13e6-194">Windows Server フェールオーバー クラスタリング (WSFC) クラスター、SQL Server Always On 可用性グループ、プライマリ レプリカを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-194">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="d13e6-195">詳細については、「[AlwaysOn 可用性グループの概要][sql-alwayson-getting-started]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-195">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="d13e6-196">静的プライベート IP アドレスを持つ内部ロード バランサーを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-196">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="d13e6-197">可用性グループ リスナーを作成し、リスナーの DNS 名を内部ロード バランサーの IP アドレスにマッピングします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-197">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="d13e6-198">SQL Server リスニング ポート (既定ではTCP ポート 1433) に対するロード バランサーのルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-198">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="d13e6-199">ロード バランサーのルールでは *Floating IP* (Direct Server Return とも呼ばれます) を有効にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-199">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="d13e6-200">これにより VM が直接クライアントに応答でき、プライマリ レプリカへの直接接続が可能になります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-200">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="d13e6-201">Floating IP が有効になっている場合は、フロントエンド ポート番号を、ロード バランサーのルール内のバックエンド ポート番号と同じにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-201">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
   > 
   > 

<span data-ttu-id="d13e6-202">SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-202">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="d13e6-203">別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーはその後の要求を自動的に新しいプライマリ レプリカにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-203">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="d13e6-204">詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成][sql-alwayson-ilb]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-204">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="d13e6-205">フェールオーバー中は、既存のクライアント接続は閉じられます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-205">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="d13e6-206">フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-206">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="d13e6-207">アプリケーションが書き込みよりもはるかに多くの読み取りを行う場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-207">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="d13e6-208">「[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]」(リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-208">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="d13e6-209">可用性グループの[手動フェールオーバーの強制][sql-alwayson-force-failover]によってデプロイをテストします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-209">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="d13e6-210">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="d13e6-210">Jumpbox</span></span>

<span data-ttu-id="d13e6-211">アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-211">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="d13e6-212">代わりに、これらの VM へのすべての RDP アクセスは、ジャンプボックスを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-212">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="d13e6-213">管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-213">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="d13e6-214">ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-214">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="d13e6-215">ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-215">The jumpbox has minimal performance requirements, so select a small VM size.</span></span> <span data-ttu-id="d13e6-216">ジャンプボックス用に[パブリック IP アドレス]を作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-216">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="d13e6-217">ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-217">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="d13e6-218">ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-218">To secure the jumpbox, add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="d13e6-219">他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-219">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d13e6-220">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d13e6-220">Scalability considerations</span></span>

<span data-ttu-id="d13e6-221">[VM スケール セット][vmss]を使用して、同一の VM のセットをデプロイして管理できます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-221">[VM scale sets][vmss] help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="d13e6-222">スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-222">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="d13e6-223">VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-223">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="d13e6-224">VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-224">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="d13e6-225">スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-225">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="d13e6-226">拡張機能を使用してプロビジョニングされた後に VM を構成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-226">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="d13e6-227">この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-227">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="d13e6-228">カスタム ディスク イメージと共に[マネージド ディスク](/azure/storage/storage-managed-disks-overview)をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-228">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="d13e6-229">このオプションの方が早くデプロイできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-229">This option may be quicker to deploy.</span></span> <span data-ttu-id="d13e6-230">ただし、イメージを最新の状態に保つ必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-230">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="d13e6-231">詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-231">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="d13e6-232">自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-232">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="d13e6-233">各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-233">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="d13e6-234">サポート リクエストを提出することで、制限値を上げることができます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-234">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="d13e6-235">詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-235">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="d13e6-236">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d13e6-236">Availability considerations</span></span>

<span data-ttu-id="d13e6-237">VM スケール セットを使用していない場合は、同じ層の VM を可用性セットに配置します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-237">If you are not using VM scale sets, put VMs in the same tier into an availability set.</span></span> <span data-ttu-id="d13e6-238">[Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-238">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="d13e6-239">詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-239">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> 

<span data-ttu-id="d13e6-240">ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-240">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="d13e6-241">タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-241">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="d13e6-242">ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-242">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="d13e6-243">ロード バランサーの正常性プローブには、次のような推奨事項があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-243">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="d13e6-244">プローブでは、HTTP または TCP のいずれかをテストできます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-244">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="d13e6-245">VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-245">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="d13e6-246">それ以外の場合は、TCP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-246">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="d13e6-247">HTTP プローブでは、HTTP エンドポイントへのパスを指定します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-247">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="d13e6-248">プローブでは、このパスからの HTTP 200 の応答をチェックします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-248">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="d13e6-249">これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-249">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="d13e6-250">エンドポイントでは、匿名の HTTP 要求を許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-250">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="d13e6-251">このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-251">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="d13e6-252">この IP アドレスとの間のトラフィックを、どのファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) 規則でもブロックしていないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-252">Make sure you don't block traffic to or from this IP address in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="d13e6-253">[正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-253">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="d13e6-254">各ロード バランサーに対して Azure Portal のログ記録を有効にします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-254">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="d13e6-255">ログは Azure Blob Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-255">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="d13e6-256">ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-256">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

<span data-ttu-id="d13e6-257">[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、フェールオーバーに Azure Traffic Manager を使用して、2 つのリージョンでのアプリケーションのレプリケーションを検討します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-257">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, consider replication the application across two regions, using Azure Traffic Manager for failover.</span></span> <span data-ttu-id="d13e6-258">詳細については、「[高可用性のためのマルチリージョン n 層アプリケーション][multi-dc]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-258">For more information, see [Multi-region N-tier application for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="d13e6-259">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d13e6-259">Security considerations</span></span>

<span data-ttu-id="d13e6-260">仮想ネットワークは、Azure のトラフィックの分離境界です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-260">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="d13e6-261">ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。</span><span class="sxs-lookup"><span data-stu-id="d13e6-261">VMs in one VNet cannot communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="d13e6-262">トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-262">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="d13e6-263">詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-263">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="d13e6-264">ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-264">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="d13e6-265">NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。</span><span class="sxs-lookup"><span data-stu-id="d13e6-265">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="d13e6-266">詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-266">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

<span data-ttu-id="d13e6-267">機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-267">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="d13e6-268">Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-268">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="d13e6-269">詳細については、[Azure VM 上の SQL Server 向け Azure Key Vault 統合の構成][sql-keyvault]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-269">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault].</span></span> <span data-ttu-id="d13e6-270">データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-270">It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="d13e6-271">[DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview) を有効にすることをお勧めします。これにより、VNet 内のリソースに対して DDoS の軽減策が追加されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-271">We recommend enabling [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), which provides additional DDoS mitigation for resources in a VNet.</span></span> <span data-ttu-id="d13e6-272">Azure プラットフォームの一部として基本な DDoS 保護が自動的に有効になりますが、DDoS Protection Standard により、特に Azure Virtual Network リソース向けにチューニングされた軽減機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-272">Although basic DDoS protection is automatically enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure Virtual Network resources.</span></span>  

## <a name="deploy-the-solution"></a><span data-ttu-id="d13e6-273">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="d13e6-273">Deploy the solution</span></span>

<span data-ttu-id="d13e6-274">このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-274">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="d13e6-275">デプロイ全体を完了するには最大 2 時間かかる場合があります。これには、AD DS、Windows Server フェールオーバー クラスター、および SQL Server 可用性グループを構成するスクリプトの実行時間が含まれます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-275">Note that the entire deployment can take up to two hours, which includes running the scripts to configure AD DS, the Windows Server failover cluster, and the SQL Server availability group.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d13e6-276">前提条件</span><span class="sxs-lookup"><span data-stu-id="d13e6-276">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a><span data-ttu-id="d13e6-277">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="d13e6-277">Deploy the solution</span></span>

1. <span data-ttu-id="d13e6-278">次のコマンドを実行して、リソース グループを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-278">Run the following command to create a resource group.</span></span>

    ```bash
    az group create --location <location> --name <resource-group-name>
    ```

2. <span data-ttu-id="d13e6-279">次のコマンドを実行して、クラウド監視のストレージ アカウントを作成します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-279">Run the following command to create a Storage account for the Cloud Witness.</span></span>

    ```bash
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. <span data-ttu-id="d13e6-280">参照アーキテクチャ GitHub リポジトリの `virtual-machines\n-tier-windows` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-280">Navigate to the `virtual-machines\n-tier-windows` folder of the reference architectures GitHub repository.</span></span>

4. <span data-ttu-id="d13e6-281">`n-tier-windows.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-281">Open the `n-tier-windows.json` file.</span></span> 

5. <span data-ttu-id="d13e6-282">"witnessStorageBlobEndPoint" のすべてのインスタンスを検索し、プレース ホルダー テキストを手順 2. のストレージ アカウントの名前で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-282">Search for all instances of "witnessStorageBlobEndPoint" and replace the placeholder text with the name of the Storage account from step 2.</span></span>

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. <span data-ttu-id="d13e6-283">次のコマンドを実行して、ストレージ アカウントのアカウント キーを一覧表示します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-283">Run the following command to list the account keys for the storage account.</span></span>

    ```bash
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    <span data-ttu-id="d13e6-284">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-284">The output should look like the following.</span></span> <span data-ttu-id="d13e6-285">`key1` の値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-285">Copy the value of `key1`.</span></span>

    ```json
    [
    {
        "keyName": "key1",
        "permissions": "Full",
        "value": "..."
    },
    {
        "keyName": "key2",
        "permissions": "Full",
        "value": "..."
    }
    ]
    ```

7. <span data-ttu-id="d13e6-286">`n-tier-windows.json` ファイルで、"witnessStorageAccountKey" のすべてのインスタンスを検索し、アカウント キーに貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-286">In the `n-tier-windows.json` file, search for all instances of "witnessStorageAccountKey" and paste in the account key.</span></span>

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. <span data-ttu-id="d13e6-287">`n-tier-windows.json` ファイルで、`[replace-with-password]` と `[replace-with-sql-password]` のすべてのインスタンスを検索し、そのインスタンスを強力なパスワードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="d13e6-287">In the `n-tier-windows.json` file, search for all instances of `[replace-with-password]` and `[replace-with-sql-password]` replace them with a strong password.</span></span> <span data-ttu-id="d13e6-288">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="d13e6-288">Save the file.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d13e6-289">管理者ユーザー名を変更する場合は、JSON ファイルの `extensions` ブロックも更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d13e6-289">If you change the adminstrator user name, you must also update the `extensions` blocks in the JSON file.</span></span> 

9. <span data-ttu-id="d13e6-290">次のコマンドを実行して、アーキテクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d13e6-290">Run the following command to deploy the architecture.</span></span>

    ```bash
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

<span data-ttu-id="d13e6-291">Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d13e6-291">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
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
[Zabbix]: https://www.zabbix.com/
[Icinga]: https://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "Microsoft Azure を使用した N 層アーキテクチャ"
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
