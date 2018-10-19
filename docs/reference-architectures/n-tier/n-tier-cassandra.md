---
title: Apache Cassandra を使用する N 層アプリケーション
description: Microsoft Azure で N 層アーキテクチャの Linux VM を実行する方法について説明します。
author: MikeWasson
ms.date: 05/03/2018
ms.openlocfilehash: 2c5a80309e5d4d180cc83422de0b462c8dffcd90
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876921"
---
# <a name="n-tier-application-with-apache-cassandra"></a><span data-ttu-id="6f7fc-103">Apache Cassandra を使用する N 層アプリケーション</span><span class="sxs-lookup"><span data-stu-id="6f7fc-103">N-tier application with Apache Cassandra</span></span>

<span data-ttu-id="6f7fc-104">この参照アーキテクチャでは、Linux 上の Apache Cassandra をデータ層に使用して、N 層アプリケーション用に構成された VM と仮想ネットワークをデプロイする方法を示します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-104">This reference architecture shows how to deploy VMs and a virtual network configured for an N-tier application, using Apache Cassandra on Linux for the data tier.</span></span> [<span data-ttu-id="6f7fc-105">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="6f7fc-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="6f7fc-106">![[0]][0]</span></span>

<span data-ttu-id="6f7fc-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="6f7fc-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="6f7fc-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="6f7fc-108">Architecture</span></span> 

<span data-ttu-id="6f7fc-109">アーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-109">The architecture has the following components:</span></span>

* <span data-ttu-id="6f7fc-110">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-110">**Resource group.**</span></span> <span data-ttu-id="6f7fc-111">[リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-111">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>

* <span data-ttu-id="6f7fc-112">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-112">**Virtual network (VNet) and subnets.**</span></span> <span data-ttu-id="6f7fc-113">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-113">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span> <span data-ttu-id="6f7fc-114">階層ごとに個別のサブネットを作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-114">Create a separate subnet for each tier.</span></span> 

* <span data-ttu-id="6f7fc-115">**NSG。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-115">**NSGs.**</span></span> <span data-ttu-id="6f7fc-116">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-116">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="6f7fc-117">たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-117">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>

* <span data-ttu-id="6f7fc-118">**仮想マシン**。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-118">**Virtual machines**.</span></span> <span data-ttu-id="6f7fc-119">VM の構成に関する推奨事項については、「[Run a Windows VM on Azure (Azure での Windows VM の実行)](./windows-vm.md)」および「[Run a Linux VM on Azure (Azure での Linux VM の実行)](./linux-vm.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-119">For recommendations on configuring VMs, see [Run a Windows VM on Azure](./windows-vm.md) and [Run a Linux VM on Azure](./linux-vm.md).</span></span>

* <span data-ttu-id="6f7fc-120">**可用性セット。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-120">**Availability sets.**</span></span> <span data-ttu-id="6f7fc-121">階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-121">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="6f7fc-122">こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-122">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> 

* <span data-ttu-id="6f7fc-123">**VM スケール セット** (ここでは示されていません)。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-123">**VM scale set** (not shown).</span></span> <span data-ttu-id="6f7fc-124">[VM スケール セット][vmss]は可用性セットの代わりに使用します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-124">A [VM scale set][vmss] is an alternative to using an availability set.</span></span> <span data-ttu-id="6f7fc-125">スケール セットを使用すると、層内の VM を簡単にスケールアウトできます。スケールアウトは、手動で実行することも、定義済みの規則に基づいて自動的に実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-125">A scale sets makes it easy to scale out the VMs in a tier, either manually or automatically based on predefined rules.</span></span>

* <span data-ttu-id="6f7fc-126">**Azure Load Balancer。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-126">**Azure Load balancers.**</span></span> <span data-ttu-id="6f7fc-127">[ロード バランサー][load-balancer]は、受信インターネット要求を各 VM インスタンスに分散します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-127">The [load balancers][load-balancer] distribute incoming Internet requests to the VM instances.</span></span> <span data-ttu-id="6f7fc-128">[パブリック ロード バランサー][load-balancer-external]を使用して受信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-128">Use a [public load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>

* <span data-ttu-id="6f7fc-129">**パブリック IP アドレス**。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-129">**Public IP address**.</span></span> <span data-ttu-id="6f7fc-130">パブリック IP アドレスは、パブリック ロード バランサーがインターネット トラフィックを受信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-130">A public IP address is needed for the public load balancer to receive Internet traffic.</span></span>

* <span data-ttu-id="6f7fc-131">**ジャンプボックス。**</span><span class="sxs-lookup"><span data-stu-id="6f7fc-131">**Jumpbox.**</span></span> <span data-ttu-id="6f7fc-132">[要塞ホスト]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-132">Also called a [bastion host].</span></span> <span data-ttu-id="6f7fc-133">管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-133">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="6f7fc-134">ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-134">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="6f7fc-135">NSG では SSH トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-135">The NSG should permit ssh traffic.</span></span>

* <span data-ttu-id="6f7fc-136">**Apache Cassandra データベース**。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-136">**Apache Cassandra database**.</span></span> <span data-ttu-id="6f7fc-137">レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-137">Provides high availability at the data tier, by enabling replication and failover.</span></span>

* <span data-ttu-id="6f7fc-138">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-138">**Azure DNS**.</span></span> <span data-ttu-id="6f7fc-139">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-139">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="6f7fc-140">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-140">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="6f7fc-141">Recommendations</span><span class="sxs-lookup"><span data-stu-id="6f7fc-141">Recommendations</span></span>

<span data-ttu-id="6f7fc-142">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-142">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="6f7fc-143">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-143">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="6f7fc-144">VNet/サブネット</span><span class="sxs-lookup"><span data-stu-id="6f7fc-144">VNet / Subnets</span></span>

<span data-ttu-id="6f7fc-145">VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-145">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="6f7fc-146">[CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-146">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="6f7fc-147">標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-147">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="6f7fc-148">後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-148">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="6f7fc-149">VNet を作成した後は、アドレス範囲を変更できません。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-149">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="6f7fc-150">機能とセキュリティの要件を念頭に置いてサブネットを設計します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-150">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="6f7fc-151">同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-151">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="6f7fc-152">VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-152">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

### <a name="load-balancers"></a><span data-ttu-id="6f7fc-153">ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="6f7fc-153">Load balancers</span></span>

<span data-ttu-id="6f7fc-154">VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-154">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="6f7fc-155">クライアントは、パブリック ロード バランサーの IP アドレスを使用して接続します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-155">Clients connect using the IP address of the public load balancer.</span></span>

<span data-ttu-id="6f7fc-156">ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-156">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="6f7fc-157">たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-157">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="6f7fc-158">クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-158">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="6f7fc-159">この方法で、クライアント要求がすべての VM に配布されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-159">In that way, client requests are distributed across all the VMs.</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="6f7fc-160">ネットワーク セキュリティ グループ</span><span class="sxs-lookup"><span data-stu-id="6f7fc-160">Network security groups</span></span>

<span data-ttu-id="6f7fc-161">NSG ルールを使用して階層間のトラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-161">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="6f7fc-162">たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-162">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="6f7fc-163">これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-163">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="6f7fc-164">VNet からのすべての受信トラフィックを拒否します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-164">Deny all inbound traffic from the VNet.</span></span> <span data-ttu-id="6f7fc-165">(ルール内で `VIRTUAL_NETWORK` タグを使用します。)</span><span class="sxs-lookup"><span data-stu-id="6f7fc-165">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
2. <span data-ttu-id="6f7fc-166">ビジネス層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-166">Allow inbound traffic from the business tier subnet.</span></span>  
3. <span data-ttu-id="6f7fc-167">データベース層のサブネットからの受信トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-167">Allow inbound traffic from the database tier subnet itself.</span></span> <span data-ttu-id="6f7fc-168">このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-168">This rule allows communication between the database VMs, which is needed for database replication and failover.</span></span>
4. <span data-ttu-id="6f7fc-169">ジャンプボックスのサブネットからの SSH トラフィック (ポート 22) を許可します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-169">Allow ssh traffic (port 22) from the jumpbox subnet.</span></span> <span data-ttu-id="6f7fc-170">このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-170">This rule lets administrators connect to the database tier from the jumpbox.</span></span>

<span data-ttu-id="6f7fc-171">最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-171">Create rules 2 &ndash; 4 with higher priority than the first rule, so they override it.</span></span>

### <a name="cassandra"></a><span data-ttu-id="6f7fc-172">Cassandra</span><span class="sxs-lookup"><span data-stu-id="6f7fc-172">Cassandra</span></span>

<span data-ttu-id="6f7fc-173">運用環境では [DataStax Enterprise][datastax] の使用をお勧めしますが、これらの推奨事項はすべての Cassandra エディションに適用されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-173">We recommend [DataStax Enterprise][datastax] for production use, but these recommendations apply to any Cassandra edition.</span></span> <span data-ttu-id="6f7fc-174">Azure での DataStax の実行の詳細については、「[Azure 用の DataStax Enterprise Deployment ガイド][cassandra-in-azure]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-174">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> 

<span data-ttu-id="6f7fc-175">Cassandra クラスター用の VM を可用性セット内に配置して、Cassandra レプリカが複数の障害ドメインおよびアップグレード ドメイン間に分散されることを保証します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-175">Put the VMs for a Cassandra cluster in an availability set to ensure that the Cassandra replicas are distributed across multiple fault domains and upgrade domains.</span></span> <span data-ttu-id="6f7fc-176">障害ドメインおよびアップグレード ドメインの詳細については、「[Virtual Machines の可用性管理][azure-availability-sets]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-176">For more information about fault domains and upgrade domains, see [Manage the availability of virtual machines][azure-availability-sets].</span></span> 

<span data-ttu-id="6f7fc-177">可用性セットごとに 3 個の障害ドメイン (最大数) と 18 個のアップグレード ドメインを構成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-177">Configure three fault domains (the maximum) per availability set and 18 upgrade domains per availability set.</span></span> <span data-ttu-id="6f7fc-178">これは、障害ドメイン間で引き続き均等に分散できるアップグレード ドメインの最大数を提供します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-178">This provides the maximum number of upgrade domains that can still be distributed evenly across the fault domains.</span></span>   

<span data-ttu-id="6f7fc-179">ラック認識モードでノードを構成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-179">Configure nodes in rack-aware mode.</span></span> <span data-ttu-id="6f7fc-180">`cassandra-rackdc.properties` ファイル内で障害ドメインをラックにマッピングします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-180">Map fault domains to racks in the `cassandra-rackdc.properties` file.</span></span>

<span data-ttu-id="6f7fc-181">クラスターの前にロード バランサーは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-181">You don't need a load balancer in front of the cluster.</span></span> <span data-ttu-id="6f7fc-182">クライアントはクラスターのノードに直接接続します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-182">The client connects directly to a node in the cluster.</span></span>

<span data-ttu-id="6f7fc-183">高可用性を確保するために、複数の Azure リージョンに Cassandra をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-183">For high availability, deploy Cassandra in more than one Azure region.</span></span> <span data-ttu-id="6f7fc-184">各リージョンのノードは、リージョン内の回復性を高めるために、障害ドメインとアップグレード ドメインを持つラック認識モードで構成されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-184">Within each region, nodes are configured in rack-aware mode with fault and upgrade domains, for resiliency inside the region.</span></span>


### <a name="jumpbox"></a><span data-ttu-id="6f7fc-185">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="6f7fc-185">Jumpbox</span></span>

<span data-ttu-id="6f7fc-186">アプリケーション ワークロードを実行する VM へのパブリック インターネットからの SSH アクセスを許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-186">Do not allow ssh access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="6f7fc-187">代わりに、これらの VM へのすべての SSH アクセスは、ジャンプボックスを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-187">Instead, all ssh access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="6f7fc-188">管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-188">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="6f7fc-189">ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの SSH トラフィックを許可します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-189">The jumpbox allows ssh traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="6f7fc-190">ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-190">The jumpbox has minimal performance requirements, so select a small VM size.</span></span> <span data-ttu-id="6f7fc-191">ジャンプボックス用に[パブリック IP アドレス]を作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-191">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="6f7fc-192">ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-192">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="6f7fc-193">ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ SSH 接続を許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-193">To secure the jumpbox, add an NSG rule that allows ssh connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="6f7fc-194">管理サブネットからの SSH トラフィックを許可するように、他のサブネットの NSG を構成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-194">Configure the NSGs for the other subnets to allow ssh traffic from the management subnet.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="6f7fc-195">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="6f7fc-195">Scalability considerations</span></span>

<span data-ttu-id="6f7fc-196">[VM スケール セット][vmss]を使用して、同一の VM のセットをデプロイして管理できます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-196">[VM scale sets][vmss] help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="6f7fc-197">スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-197">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="6f7fc-198">VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-198">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="6f7fc-199">VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-199">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="6f7fc-200">スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-200">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="6f7fc-201">拡張機能を使用してプロビジョニングされた後に VM を構成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-201">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="6f7fc-202">この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-202">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="6f7fc-203">カスタム ディスク イメージと共に[マネージド ディスク](/azure/storage/storage-managed-disks-overview)をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-203">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="6f7fc-204">このオプションの方が早くデプロイできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-204">This option may be quicker to deploy.</span></span> <span data-ttu-id="6f7fc-205">ただし、イメージを最新の状態に保つ必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-205">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="6f7fc-206">詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-206">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="6f7fc-207">自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-207">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="6f7fc-208">各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-208">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="6f7fc-209">サポート リクエストを提出することで、制限値を上げることができます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-209">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="6f7fc-210">詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-210">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="6f7fc-211">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="6f7fc-211">Availability considerations</span></span>

<span data-ttu-id="6f7fc-212">VM スケール セットを使用していない場合は、同じ層の VM を可用性セットに配置します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-212">If you are not using VM scale sets, put VMs in the same tier into an availability set.</span></span> <span data-ttu-id="6f7fc-213">[Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-213">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="6f7fc-214">詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-214">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> 

<span data-ttu-id="6f7fc-215">ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-215">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="6f7fc-216">タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-216">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="6f7fc-217">ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-217">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="6f7fc-218">ロード バランサーの正常性プローブには、次のような推奨事項があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-218">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="6f7fc-219">プローブでは、HTTP または TCP のいずれかをテストできます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-219">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="6f7fc-220">VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-220">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="6f7fc-221">それ以外の場合は、TCP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-221">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="6f7fc-222">HTTP プローブでは、HTTP エンドポイントへのパスを指定します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-222">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="6f7fc-223">プローブでは、このパスからの HTTP 200 の応答をチェックします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-223">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="6f7fc-224">これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-224">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="6f7fc-225">エンドポイントでは、匿名の HTTP 要求を許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-225">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="6f7fc-226">このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-226">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="6f7fc-227">この IP アドレスとの間のトラフィックを、どのファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) 規則でもブロックしていないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-227">Make sure you don't block traffic to or from this IP address in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="6f7fc-228">[正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-228">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="6f7fc-229">各ロード バランサーに対して Azure Portal のログ記録を有効にします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-229">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="6f7fc-230">ログは Azure Blob Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-230">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="6f7fc-231">ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-231">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

<span data-ttu-id="6f7fc-232">Cassandra クラスターのために検討するフェールオーバー シナリオは、アプリケーションで使用される一貫性レベルと使用されるレプリカの数によって異なります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-232">For the Cassandra cluster, the failover scenarios to consider depend on the consistency levels used by the application, as well as the number of replicas used.</span></span> <span data-ttu-id="6f7fc-233">Cassandra での一貫性レベルと使用については、「[Configuring data consistency][cassandra-consistency]」(データの一貫性の構成) と「[Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage]」(Cassandra: Quorum を使用してトークできるノードの数) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-233">For consistency levels and usage in Cassandra, see [Configuring data consistency][cassandra-consistency] and [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage]</span></span> <span data-ttu-id="6f7fc-234">Cassandra でのデータの可用性は、アプリケーションで使用される一貫性レベルとレプリケーション メカニズムによって決まります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-234">Data availability in Cassandra is determined by the consistency level used by the application and the replication mechanism.</span></span> <span data-ttu-id="6f7fc-235">Cassandra のレプリケーションについては、「[Data Replication in NoSQL Databases Explained][cassandra-replication]」(NoSQL Database でのデータレプリケーションの説明) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-235">For replication in Cassandra, see [Data Replication in NoSQL Databases Explained][cassandra-replication].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="6f7fc-236">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="6f7fc-236">Security considerations</span></span>

<span data-ttu-id="6f7fc-237">仮想ネットワークは、Azure のトラフィックの分離境界です。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-237">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="6f7fc-238">ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-238">VMs in one VNet cannot communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="6f7fc-239">トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-239">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="6f7fc-240">詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-240">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="6f7fc-241">インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-241">For incoming Internet traffic, the load balancer rules define which traffic can reach the back end.</span></span> <span data-ttu-id="6f7fc-242">しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-242">However, load balancer rules don't support IP safe lists, so if you want to add certain public IP addresses to a safe list, add an NSG to the subnet.</span></span>

<span data-ttu-id="6f7fc-243">ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-243">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="6f7fc-244">NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-244">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="6f7fc-245">詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-245">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

<span data-ttu-id="6f7fc-246">機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-246">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="6f7fc-247">Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-247">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="6f7fc-248">データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-248">It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="6f7fc-249">[DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview) を有効にすることをお勧めします。これにより、VNet 内のリソースに対して DDoS の軽減策が追加されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-249">We recommend enabling [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), which provides additional DDoS mitigation for resources in a VNet.</span></span> <span data-ttu-id="6f7fc-250">Azure プラットフォームの一部として基本な DDoS 保護が自動的に有効になりますが、DDoS Protection Standard により、特に Azure Virtual Network リソース向けにチューニングされた軽減機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-250">Although basic DDoS protection is automatically enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure Virtual Network resources.</span></span>  

## <a name="deploy-the-solution"></a><span data-ttu-id="6f7fc-251">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="6f7fc-251">Deploy the solution</span></span>

<span data-ttu-id="6f7fc-252">このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-252">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="6f7fc-253">前提条件</span><span class="sxs-lookup"><span data-stu-id="6f7fc-253">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="6f7fc-254">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="6f7fc-254">Deploy the solution using azbb</span></span>

<span data-ttu-id="6f7fc-255">N 層アプリケーションの参照アーキテクチャで Linux VM をデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-255">To deploy the Linux VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="6f7fc-256">上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-256">Navigate to the `virtual-machines\n-tier-linux` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="6f7fc-257">このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-257">The parameter file specifies a default administrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="6f7fc-258">参照アーキテクチャをデプロイする前に、これらを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-258">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="6f7fc-259">`n-tier-linux.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-259">Open the `n-tier-linux.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>   <span data-ttu-id="6f7fc-260">ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-260">Save the file.</span></span>

3. <span data-ttu-id="6f7fc-261">次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-261">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

<span data-ttu-id="6f7fc-262">Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6f7fc-262">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: https://www.datastax.com/products/datastax-enterprise
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
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: https://www.zabbix.com/
[Icinga]: https://www.icinga.org/
[0]: ./images/n-tier-cassandra.png "Microsoft Azure を使用した N 層アーキテクチャ"

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
