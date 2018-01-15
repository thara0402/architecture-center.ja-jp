---
title: "スケーラビリティと可用性のために Azure で負荷分散された VM を実行する"
description: "スケーラビリティと可用性のために Azure で複数の Windows VM を実行する方法。"
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 14e7e023afd7cb7cbe0e8db8e224ba777f6fe863
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2018
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a><span data-ttu-id="69000-103">スケーラビリティと可用性のために負荷分散された VM を実行する</span><span class="sxs-lookup"><span data-stu-id="69000-103">Run load-balanced VMs for scalability and availability</span></span>

<span data-ttu-id="69000-104">この参照アーキテクチャは、可用性とスケーラビリティを向上させるために、ロード バランサーの背後にあるスケール セット内で複数の Windows 仮想マシン (VM) を実行するための一連の実証済みの手法を示します。</span><span class="sxs-lookup"><span data-stu-id="69000-104">This reference architecture shows a set of proven practices for running multiple Windows virtual machines (VMs) in a scale set behind a load balancer, to improve availability and scalability.</span></span> <span data-ttu-id="69000-105">このアーキテクチャは任意のステートレス ワークロード (Web サーバーなど) に使用でき、N 層アプリケーションをデプロイするための基礎となります。</span><span class="sxs-lookup"><span data-stu-id="69000-105">This architecture can be used for any stateless workload, such as a web server, and is a foundation for deploying n-tier applications.</span></span> [<span data-ttu-id="69000-106">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="69000-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="69000-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="69000-107">![[0]][0]</span></span>

<span data-ttu-id="69000-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="69000-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="69000-109">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="69000-109">Architecture</span></span>

<span data-ttu-id="69000-110">このアーキテクチャは、[単一の VM 参照アーキテクチャ][single-vm]に基づいて構築されています。</span><span class="sxs-lookup"><span data-stu-id="69000-110">This architecture builds on the [Single VM reference architecture][single-vm].</span></span> <span data-ttu-id="69000-111">それらの推奨事項は、このアーキテクチャにも適用されます。</span><span class="sxs-lookup"><span data-stu-id="69000-111">Those recommendations also apply to this architecture.</span></span>

<span data-ttu-id="69000-112">このアーキテクチャでは、ワークロードが複数の VM インスタンスにわたって分散されます。</span><span class="sxs-lookup"><span data-stu-id="69000-112">In this architecture, a workload is distributed across multiple VM instances.</span></span> <span data-ttu-id="69000-113">単一のパブリック IP アドレスが存在し、ロード バランサーを使用してインターネット トラフィックが VM に分散されます。</span><span class="sxs-lookup"><span data-stu-id="69000-113">There is a single public IP address, and Internet traffic is distributed to the VMs using a load balancer.</span></span> <span data-ttu-id="69000-114">このアーキテクチャは、ステートレスな Web アプリケーションなどの、単一の層のアプリケーションに使用できます。</span><span class="sxs-lookup"><span data-stu-id="69000-114">This architecture can be used for a single-tier application, such as a stateless web application.</span></span>

<span data-ttu-id="69000-115">アーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="69000-115">The architecture has the following components:</span></span>

* <span data-ttu-id="69000-116">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="69000-116">**Resource group.**</span></span> <span data-ttu-id="69000-117">[リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="69000-117">[Resource groups][resource-manager-overview] are used to group resources so they can be managed by lifetime, owner, or other criteria.</span></span>
* <span data-ttu-id="69000-118">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="69000-118">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="69000-119">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="69000-119">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>
* <span data-ttu-id="69000-120">**Azure Load Balancer**。</span><span class="sxs-lookup"><span data-stu-id="69000-120">**Azure Load Balancer**.</span></span> <span data-ttu-id="69000-121">[ロード バランサー][load-balancer]は、受信インターネット要求を各 VM インスタンスに分散します。</span><span class="sxs-lookup"><span data-stu-id="69000-121">The [load balancer][load-balancer] distributes incoming Internet requests to the VM instances.</span></span> 
* <span data-ttu-id="69000-122">**パブリック IP アドレス**。</span><span class="sxs-lookup"><span data-stu-id="69000-122">**Public IP address**.</span></span> <span data-ttu-id="69000-123">パブリック IP アドレスは、ロード バランサーがインターネット トラフィックを受信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="69000-123">A public IP address is needed for the load balancer to receive Internet traffic.</span></span>
* <span data-ttu-id="69000-124">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="69000-124">**Azure DNS**.</span></span> <span data-ttu-id="69000-125">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="69000-125">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="69000-126">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="69000-126">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>
* <span data-ttu-id="69000-127">**VM スケール セット**。</span><span class="sxs-lookup"><span data-stu-id="69000-127">**VM scale set**.</span></span> <span data-ttu-id="69000-128">[VM スケール セット][vm-scaleset]は、ワークロードをホストするために使用される同一の VM のセットです。</span><span class="sxs-lookup"><span data-stu-id="69000-128">A [VM scale set][vm-scaleset] is a set of identical VMs used to host a workload.</span></span> <span data-ttu-id="69000-129">スケール セットにより、VM の数を手動でスケールインまたはスケールアウトしたり、定義済みの規則に基づいて自動的に設定したりできるようになります。</span><span class="sxs-lookup"><span data-stu-id="69000-129">Scale sets allow the number of VMs to be scaled in or out manually, or automatically based on predefined rules.</span></span>
* <span data-ttu-id="69000-130">**可用性セット**。</span><span class="sxs-lookup"><span data-stu-id="69000-130">**Availability set**.</span></span> <span data-ttu-id="69000-131">[可用性セット][availability-set]には VM が含まれ、VM がより高度な[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="69000-131">The [availability set][availability-set] contains the VMs, making the VMs eligible for a higher [service level agreement (SLA)][vm-sla].</span></span> <span data-ttu-id="69000-132">より高度な SLA を適用するためには、可用性セットには少なくとも 2 つの VM が含まれる必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-132">For the higher SLA to apply, the availability set must include a minimum of two VMs.</span></span> <span data-ttu-id="69000-133">可用性セットはスケール セットの中で暗黙的です。</span><span class="sxs-lookup"><span data-stu-id="69000-133">Availability sets are implicit in scale sets.</span></span> <span data-ttu-id="69000-134">スケール セットの外で VM を作成する場合は、可用性セットを個別に作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-134">If you create VMs outside a scale set, you need to create the availability set independently.</span></span>
* <span data-ttu-id="69000-135">**Managed Disks**。</span><span class="sxs-lookup"><span data-stu-id="69000-135">**Managed disks**.</span></span> <span data-ttu-id="69000-136">Azure Managed Disks では、VM ディスクの仮想ハード ディスク (VHD) ファイルが管理されます。</span><span class="sxs-lookup"><span data-stu-id="69000-136">Azure Managed Disks manage the virtual hard disk (VHD) files for the VM disks.</span></span> 
* <span data-ttu-id="69000-137">**ストレージ**。</span><span class="sxs-lookup"><span data-stu-id="69000-137">**Storage**.</span></span> <span data-ttu-id="69000-138">Azure Storage アカウントを作成して、VM の診断ログを保持します。</span><span class="sxs-lookup"><span data-stu-id="69000-138">Create an Azure Storage acount to hold diagnostic logs for the VMs.</span></span>

## <a name="recommendations"></a><span data-ttu-id="69000-139">Recommendations</span><span class="sxs-lookup"><span data-stu-id="69000-139">Recommendations</span></span>

<span data-ttu-id="69000-140">ユーザーの要件が、ここで説明されているアーキテクチャと完全には整合しない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="69000-140">Your requirements may not align completely with the architecture described here.</span></span> <span data-ttu-id="69000-141">これらの推奨事項は原案として使用してください。</span><span class="sxs-lookup"><span data-stu-id="69000-141">Use these recommendations as a starting point.</span></span> 

### <a name="availability-and-scalability-recommendations"></a><span data-ttu-id="69000-142">可用性とスケーラビリティの推奨事項</span><span class="sxs-lookup"><span data-stu-id="69000-142">Availability and scalability recommendations</span></span>

<span data-ttu-id="69000-143">可用性とスケーラビリティのための 1 つのオプションは、[仮想マシン スケール セット][vmss]を使用することです。</span><span class="sxs-lookup"><span data-stu-id="69000-143">An option for availability and scalability is to use a [virtual machine scale set][vmss].</span></span> <span data-ttu-id="69000-144">VM スケール セットを使用して、同一の VM のセットをデプロイして管理できます。</span><span class="sxs-lookup"><span data-stu-id="69000-144">VM scale sets help you to deploy and manage a set of identical VMs.</span></span> <span data-ttu-id="69000-145">スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="69000-145">Scale sets support autoscaling based on performance metrics.</span></span> <span data-ttu-id="69000-146">VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。</span><span class="sxs-lookup"><span data-stu-id="69000-146">As the load on the VMs increases, additional VMs are automatically added to the load balancer.</span></span> <span data-ttu-id="69000-147">VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。</span><span class="sxs-lookup"><span data-stu-id="69000-147">Consider scale sets if you need to quickly scale out VMs, or need to autoscale.</span></span>

<span data-ttu-id="69000-148">既定では、スケール セットでは "オーバープロビジョニング" が使用されます。これは、スケール セットが要求するよりも多くの VM を最初にプロビジョニングし、次に余分な VM を削除するという意味です。</span><span class="sxs-lookup"><span data-stu-id="69000-148">By default, scale sets use "overprovisioning," which means the scale set initially provisions more VMs than you ask for, then deletes the extra VMs.</span></span> <span data-ttu-id="69000-149">これにより、VM をプロビジョニングするときの全体的な成功率が向上します。</span><span class="sxs-lookup"><span data-stu-id="69000-149">This improves the overall success rate when provisioning the VMs.</span></span> <span data-ttu-id="69000-150">[管理ディスク](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)を使用していない場合は、ストレージ アカウントあたりのオーバープロビジョニングが有効な VM が 20 を超えず、またオーバープロビジョニングが無効な VM が 40 を超えないようにすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="69000-150">If you are not using [managed disks](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), we recommend no more than 20 VMs per storage account with overprovisioning enabled, and no more than 40 VMs with overprovisioning disabled.</span></span>

<span data-ttu-id="69000-151">スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。</span><span class="sxs-lookup"><span data-stu-id="69000-151">There are two basic ways to configure VMs deployed in a scale set:</span></span>

- <span data-ttu-id="69000-152">拡張機能を使用してプロビジョニングされた後に VM を構成します。</span><span class="sxs-lookup"><span data-stu-id="69000-152">Use extensions to configure the VM after it is provisioned.</span></span> <span data-ttu-id="69000-153">この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="69000-153">With this approach, new VM instances may take longer to start up than a VM with no extensions.</span></span>

- <span data-ttu-id="69000-154">カスタム ディスク イメージと共に [Managed Disk](/azure/storage/storage-managed-disks-overview) をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="69000-154">Deploy a [managed disk](/azure/storage/storage-managed-disks-overview) with a custom disk image.</span></span> <span data-ttu-id="69000-155">このオプションの方が早くデプロイできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="69000-155">This option may be quicker to deploy.</span></span> <span data-ttu-id="69000-156">ただし、イメージを最新の状態に保つ必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-156">However, it requires you to keep the image up to date.</span></span>

<span data-ttu-id="69000-157">詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="69000-157">For additional considerations, see [Design considerations for scale sets][vmss-design].</span></span>

> [!TIP]
> <span data-ttu-id="69000-158">自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。</span><span class="sxs-lookup"><span data-stu-id="69000-158">When using any autoscale solution, test it with production-level workloads well in advance.</span></span>

<span data-ttu-id="69000-159">スケール セットを使用しない場合は、少なくとも可用性セットの使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="69000-159">If you do not use a scale set, consider at least using an availability set.</span></span> <span data-ttu-id="69000-160">[Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-160">Create at least two VMs in the availability set to support the [availability SLA for Azure VMs][vm-sla].</span></span> <span data-ttu-id="69000-161">Azure Load Balancer でも、負荷分散された VM が同じ可用性セットに属する必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-161">The Azure load balancer also requires that load-balanced VMs belong to the same availability set.</span></span>

<span data-ttu-id="69000-162">各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。</span><span class="sxs-lookup"><span data-stu-id="69000-162">Each Azure subscription has default limits in place, including a maximum number of VMs per region.</span></span> <span data-ttu-id="69000-163">サポート リクエストを提出することで、制限値を上げることができます。</span><span class="sxs-lookup"><span data-stu-id="69000-163">You can increase the limit by filing a support request.</span></span> <span data-ttu-id="69000-164">詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="69000-164">For more information, see [Azure subscription and service limits, quotas, and constraints][subscription-limits].</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="69000-165">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="69000-165">Network recommendations</span></span>

<span data-ttu-id="69000-166">VM を同じサブネット内にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="69000-166">Deploy the VMs within the same subnet.</span></span> <span data-ttu-id="69000-167">VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。</span><span class="sxs-lookup"><span data-stu-id="69000-167">Do not expose the VMs directly to the Internet, but instead give each VM a private IP address.</span></span> <span data-ttu-id="69000-168">クライアントは、ロード バランサーのパブリック IP アドレスを使用して接続します。</span><span class="sxs-lookup"><span data-stu-id="69000-168">Clients connect using the public IP address of the load balancer.</span></span>

<span data-ttu-id="69000-169">ロード バランサーの背後にある VM にログインする必要がある場合は、ログインできるパブリック IP アドレスを使用して、単一の VM をジャンプボックス (要塞ホストとも呼ばれます) として追加することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="69000-169">If you need to log into the VMs behind the load balancer, consider adding a single VM as a jumpbox (also called a bastion host) with a public IP address you can log into.</span></span> <span data-ttu-id="69000-170">ジャンプボックスからロード バランサーの背後にある VM にログインします。</span><span class="sxs-lookup"><span data-stu-id="69000-170">And then log into the VMs behind the load balancer from the jumpbox.</span></span> <span data-ttu-id="69000-171">あるいは、ロード バランサーの受信ネットワーク アドレス変換 (NAT) 規則を構成できます。</span><span class="sxs-lookup"><span data-stu-id="69000-171">Alternatively, you can configure the load balancer's inbound network address translation (NAT) rules.</span></span> <span data-ttu-id="69000-172">ただし、N 層ワークロードまたは複数のワークロードをホストしている場合は、ジャンプボックスを使用する方がより優れたソリューションです。</span><span class="sxs-lookup"><span data-stu-id="69000-172">However, having a jumpbox is a better solution when you are hosting n-tier workloads or multiple workloads.</span></span>

### <a name="load-balancer-recommendations"></a><span data-ttu-id="69000-173">ロード バランサーの推奨事項</span><span class="sxs-lookup"><span data-stu-id="69000-173">Load balancer recommendations</span></span>

<span data-ttu-id="69000-174">可用性セット内のすべての VM をロード バランサーのバックエンド アドレス プールに追加します。</span><span class="sxs-lookup"><span data-stu-id="69000-174">Add all VMs in the availability set to the back-end address pool of the load balancer.</span></span>

<span data-ttu-id="69000-175">ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。</span><span class="sxs-lookup"><span data-stu-id="69000-175">Define load balancer rules to direct network traffic to the VMs.</span></span> <span data-ttu-id="69000-176">たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-176">For example, to enable HTTP traffic, create a rule that maps port 80 from the front-end configuration to port 80 on the back-end address pool.</span></span> <span data-ttu-id="69000-177">クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。</span><span class="sxs-lookup"><span data-stu-id="69000-177">When a client sends an HTTP request to port 80, the load balancer selects a back-end IP address by using a [hashing algorithm][load-balancer-hashing] that includes the source IP address.</span></span> <span data-ttu-id="69000-178">この方法で、クライアント要求がすべての VM に配布されます。</span><span class="sxs-lookup"><span data-stu-id="69000-178">In that way, client requests are distributed across all the VMs.</span></span>

<span data-ttu-id="69000-179">特定の VM にトラフィックをルーティングするには、NAT 規則を使用します。</span><span class="sxs-lookup"><span data-stu-id="69000-179">To route traffic to a specific VM, use NAT rules.</span></span> <span data-ttu-id="69000-180">たとえば、VM への RDP を有効にするには、VM ごとに個別の NAT 規則を作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-180">For example, to enable RDP to the VMs, create a separate NAT rule for each VM.</span></span> <span data-ttu-id="69000-181">各規則では、RDP の既定のポートであるポート 3389 に個別のポート番号を割り当てる必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-181">Each rule should map a distinct port number to port 3389, the default port for RDP.</span></span> <span data-ttu-id="69000-182">たとえば、"VM1" にはポート 50001 を、"VM2" にはポート 50002 を使用するなどです。</span><span class="sxs-lookup"><span data-stu-id="69000-182">For example, use port 50001 for "VM1," port 50002 for "VM2," and so on.</span></span> <span data-ttu-id="69000-183">VM 上の NIC に NAT 規則を割り当てます。</span><span class="sxs-lookup"><span data-stu-id="69000-183">Assign the NAT rules to the NICs on the VMs.</span></span>

### <a name="storage-account-recommendations"></a><span data-ttu-id="69000-184">ストレージ アカウントの推奨事項</span><span class="sxs-lookup"><span data-stu-id="69000-184">Storage account recommendations</span></span>

<span data-ttu-id="69000-185">[管理ディスク](/azure/storage/storage-managed-disks-overview)を [Premium ストレージ][premium]と共に使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="69000-185">We recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview) with [premium storage][premium].</span></span> <span data-ttu-id="69000-186">管理ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="69000-186">Managed disks do not require a storage account.</span></span> <span data-ttu-id="69000-187">単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="69000-187">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="69000-188">非管理対象ディスクを使用している場合は、ストレージ アカウントの 1 秒あたりの入出力操作 [(IOPS) の制限][vm-disk-limits]に達しないようにするために、仮想ハード ディスク (VHD) を保持するための個別の Azure ストレージ アカウントを VM ごとに作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-188">If you are using unmanaged disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the input/output operations per second [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span>

<span data-ttu-id="69000-189">診断ログ用のストレージ アカウントを 1 つ作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-189">Create one storage account for diagnostic logs.</span></span> <span data-ttu-id="69000-190">このストレージ アカウントは、すべての VM で共有できます。</span><span class="sxs-lookup"><span data-stu-id="69000-190">This storage account can be shared by all the VMs.</span></span> <span data-ttu-id="69000-191">これには、標準ディスクを使ったアンマネージ ストレージ アカウントを指定できます。</span><span class="sxs-lookup"><span data-stu-id="69000-191">This can be an unmanaged storage account using standard disks.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="69000-192">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="69000-192">Availability considerations</span></span>

<span data-ttu-id="69000-193">可用性セットは、計画済みおよび計画外の両方のメンテナンス イベントに対するアプリケーションの復元性を高めます。</span><span class="sxs-lookup"><span data-stu-id="69000-193">The availability set makes your application more resilient to both planned and unplanned maintenance events.</span></span>

* <span data-ttu-id="69000-194">*計画済みメンテナンス*は、Microsoft が基本のプラットフォームを更新したときに行われ、VM の再起動を伴う場合もあります。</span><span class="sxs-lookup"><span data-stu-id="69000-194">*Planned maintenance* occurs when Microsoft updates the underlying platform, sometimes causing VMs to be restarted.</span></span> <span data-ttu-id="69000-195">Azure は、可用性セット内の VM がすべて同時に再起動されないことを保証します。</span><span class="sxs-lookup"><span data-stu-id="69000-195">Azure makes sure the VMs in an availability set are not all restarted at the same time.</span></span> <span data-ttu-id="69000-196">他の VM が再起動されている間、少なくとも 1 つの VM は継続的に実行されます。</span><span class="sxs-lookup"><span data-stu-id="69000-196">At least one is kept running while others are restarting.</span></span>
* <span data-ttu-id="69000-197">*計画外のメンテナンス*は、ハードウェア障害が発生したときに行われます。</span><span class="sxs-lookup"><span data-stu-id="69000-197">*Unplanned maintenance* happens if there is a hardware failure.</span></span> <span data-ttu-id="69000-198">Azure では、可用性セット内の VM が複数のサーバー ラックにプロビジョニングされることを保証します。</span><span class="sxs-lookup"><span data-stu-id="69000-198">Azure makes sure that VMs in an availability set are provisioned across more than one server rack.</span></span> <span data-ttu-id="69000-199">これにより、ハードウェア障害、ネットワーク停止、電源の瞬停などの影響が小さくなります。</span><span class="sxs-lookup"><span data-stu-id="69000-199">This helps to reduce the impact of hardware failures, network outages, power interruptions, and so on.</span></span>

<span data-ttu-id="69000-200">詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="69000-200">For more information, see [Manage the availability of virtual machines][availability-set].</span></span> <span data-ttu-id="69000-201">また、[VM のスケール調整のために可用性セットを構成する方法][availability-set-ch9]のビデオにも可用性セットの適切な概要が示されています。</span><span class="sxs-lookup"><span data-stu-id="69000-201">The following video also provides a good overview of availability sets: [How Do I Configure an Availability Set to Scale VMs][availability-set-ch9].</span></span>

> [!WARNING]
> <span data-ttu-id="69000-202">VM をプロビジョニングするときに、必ず可用性セットを構成してください。</span><span class="sxs-lookup"><span data-stu-id="69000-202">Make sure to configure the availability set when you provision the VM.</span></span> <span data-ttu-id="69000-203">現時点では、VM がプロビジョニングされた後に、Resource Manager VM を可用性セットに追加する方法はありません。</span><span class="sxs-lookup"><span data-stu-id="69000-203">Currently, there is no way to add a Resource Manager VM to an availability set after the VM is provisioned.</span></span>

<span data-ttu-id="69000-204">ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。</span><span class="sxs-lookup"><span data-stu-id="69000-204">The load balancer uses [health probes][health-probes] to monitor the availability of VM instances.</span></span> <span data-ttu-id="69000-205">タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。</span><span class="sxs-lookup"><span data-stu-id="69000-205">If a probe cannot reach an instance within a timeout period, the load balancer stops sending traffic to that VM.</span></span> <span data-ttu-id="69000-206">ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。</span><span class="sxs-lookup"><span data-stu-id="69000-206">However, the load balancer will continue to probe, and if the VM becomes available again, the load balancer resumes sending traffic to that VM.</span></span>

<span data-ttu-id="69000-207">ロード バランサーの正常性プローブには、次のような推奨事項があります。</span><span class="sxs-lookup"><span data-stu-id="69000-207">Here are some recommendations on load balancer health probes:</span></span>

* <span data-ttu-id="69000-208">プローブでは、HTTP または TCP のいずれかをテストできます。</span><span class="sxs-lookup"><span data-stu-id="69000-208">Probes can test either HTTP or TCP.</span></span> <span data-ttu-id="69000-209">VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-209">If your VMs run an HTTP server, create an HTTP probe.</span></span> <span data-ttu-id="69000-210">それ以外の場合は、TCP プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="69000-210">Otherwise create a TCP probe.</span></span>
* <span data-ttu-id="69000-211">HTTP プローブでは、HTTP エンドポイントへのパスを指定します。</span><span class="sxs-lookup"><span data-stu-id="69000-211">For an HTTP probe, specify the path to an HTTP endpoint.</span></span> <span data-ttu-id="69000-212">プローブでは、このパスからの HTTP 200 の応答をチェックします。</span><span class="sxs-lookup"><span data-stu-id="69000-212">The probe checks for an HTTP 200 response from this path.</span></span> <span data-ttu-id="69000-213">これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。</span><span class="sxs-lookup"><span data-stu-id="69000-213">This can be the root path ("/"), or a health-monitoring endpoint that implements some custom logic to check the health of the application.</span></span> <span data-ttu-id="69000-214">エンドポイントでは、匿名の HTTP 要求を許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-214">The endpoint must allow anonymous HTTP requests.</span></span>
* <span data-ttu-id="69000-215">このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。</span><span class="sxs-lookup"><span data-stu-id="69000-215">The probe is sent from a [known IP address][health-probe-ip], 168.63.129.16.</span></span> <span data-ttu-id="69000-216">この IP アドレスとの間のトラフィックを、どのファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) 規則でもブロックしていないことを確認してください。</span><span class="sxs-lookup"><span data-stu-id="69000-216">Make sure you don't block traffic to or from this IP address in any firewall policies or network security group (NSG) rules.</span></span>
* <span data-ttu-id="69000-217">[正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。</span><span class="sxs-lookup"><span data-stu-id="69000-217">Use [health probe logs][health-probe-log] to view the status of the health probes.</span></span> <span data-ttu-id="69000-218">各ロード バランサーに対して Azure Portal のログ記録を有効にします。</span><span class="sxs-lookup"><span data-stu-id="69000-218">Enable logging in the Azure portal for each load balancer.</span></span> <span data-ttu-id="69000-219">ログは Azure Blob Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="69000-219">Logs are written to Azure Blob storage.</span></span> <span data-ttu-id="69000-220">ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。</span><span class="sxs-lookup"><span data-stu-id="69000-220">The logs show how many VMs on the back end are not receiving network traffic due to failed probe responses.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="69000-221">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="69000-221">Manageability considerations</span></span>

<span data-ttu-id="69000-222">複数の VM の場合、信頼性を高め、反復可能にするために、プロセスを自動化することが重要です。</span><span class="sxs-lookup"><span data-stu-id="69000-222">With multiple VMs, it is important to automate processes so they are reliable and repeatable.</span></span> <span data-ttu-id="69000-223">[Azure Automation][azure-automation] を使用して、デプロイ、OS パッチ、およびその他のタスクを自動化できます。</span><span class="sxs-lookup"><span data-stu-id="69000-223">You can use [Azure Automation][azure-automation] to automate deployment, OS patching, and other tasks.</span></span> <span data-ttu-id="69000-224">[Azure Automation][azure-automation] は、このために使用できる PowerShell に基づいたオートメーション サービスです。</span><span class="sxs-lookup"><span data-stu-id="69000-224">[Azure Automation][azure-automation] is an automation service based on PowerShell that can be used for this.</span></span> <span data-ttu-id="69000-225">オートメーション スクリプトの例は、[Runbook ギャラリー][runbook-gallery]から入手できます。</span><span class="sxs-lookup"><span data-stu-id="69000-225">Example automation scripts are available from the [Runbook Gallery][runbook-gallery].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="69000-226">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="69000-226">Security considerations</span></span>

<span data-ttu-id="69000-227">仮想ネットワークは、Azure のトラフィックの分離境界です。</span><span class="sxs-lookup"><span data-stu-id="69000-227">Virtual networks are a traffic isolation boundary in Azure.</span></span> <span data-ttu-id="69000-228">ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。</span><span class="sxs-lookup"><span data-stu-id="69000-228">VMs in one VNet cannot communicate directly with VMs in a different VNet.</span></span> <span data-ttu-id="69000-229">トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。</span><span class="sxs-lookup"><span data-stu-id="69000-229">VMs within the same VNet can communicate, unless you create [network security groups][nsg] (NSGs) to restrict traffic.</span></span> <span data-ttu-id="69000-230">詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="69000-230">For more information, see [Microsoft cloud services and network security][network-security].</span></span>

<span data-ttu-id="69000-231">インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。</span><span class="sxs-lookup"><span data-stu-id="69000-231">For incoming Internet traffic, the load balancer rules define which traffic can reach the back end.</span></span> <span data-ttu-id="69000-232">しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。</span><span class="sxs-lookup"><span data-stu-id="69000-232">However, load balancer rules don't support IP safe lists, so if you want to add certain public IP addresses to a safe list, add an NSG to the subnet.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="69000-233">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="69000-233">Deploy the solution</span></span>

<span data-ttu-id="69000-234">このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="69000-234">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="69000-235">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="69000-235">It deploys the following:</span></span>

  * <span data-ttu-id="69000-236">VM を含む、**Web** という名前の 1 つのサブネットを持つ仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="69000-236">A virtual network with a single subnet named **web** that contains the VMs.</span></span>
  * <span data-ttu-id="69000-237">Windows Server 2016 Datacenter Edition の最新バージョンを実行する VM を含んだ VM スケール セット。</span><span class="sxs-lookup"><span data-stu-id="69000-237">A VM scale set that contains VMs running the latest version of Windows Server 2016 Datacenter Edition.</span></span> <span data-ttu-id="69000-238">自動スケールが有効になっています。</span><span class="sxs-lookup"><span data-stu-id="69000-238">Autoscale is enabled.</span></span>
  * <span data-ttu-id="69000-239">VM スケール セットの前に配置されているロード バランサー。</span><span class="sxs-lookup"><span data-stu-id="69000-239">A load balancer that sits in front of the VM scale set.</span></span>
  * <span data-ttu-id="69000-240">VM スケール セットへの HTTP トラフィックを許可する受信規則を備えた NSG。</span><span class="sxs-lookup"><span data-stu-id="69000-240">An NSG with incoming rules that allow HTTP traffic to the VM scale set.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="69000-241">前提条件</span><span class="sxs-lookup"><span data-stu-id="69000-241">Prerequisites</span></span>

<span data-ttu-id="69000-242">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="69000-242">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="69000-243">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="69000-243">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="69000-244">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="69000-244">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="69000-245">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="69000-245">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="69000-246">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="69000-246">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="69000-247">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="69000-247">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="69000-248">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="69000-248">Deploy the solution using azbb</span></span>

<span data-ttu-id="69000-249">単一の VM ワークロードのサンプルをデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="69000-249">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="69000-250">上記の前提条件でダウンロードしたリポジトリの `virtual-machines\multi-vm\parameters\windows` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="69000-250">Navigate to the `virtual-machines\multi-vm\parameters\windows` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="69000-251">`multi-vm-v2.json` ファイルを開き、次に示すような引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="69000-251">Open the `multi-vm-v2.json` file and enter a username and password between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. <span data-ttu-id="69000-252">次に示すように、`azbb` を実行して VM をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="69000-252">Run `azbb` to deploy the VMs as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

<span data-ttu-id="69000-253">この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="69000-253">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "2 つの VM とロード バランサーを備えた可用性セットを構成する Azure 上のマルチ VM ソリューションのアーキテクチャ"
