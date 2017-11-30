---
title: "Azure での Linux VM の実行"
description: "拡張性、回復性、管理容易性、セキュリティに注目しながら、Azure で Linux VM を実行する方法。"
author: telmosampaio
ms.date: 09/06/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f38c53db5df1681a1abc926df9f0b7f929ceb76b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="cd5ce-103">Azure での Linux VM の実行</span><span class="sxs-lookup"><span data-stu-id="cd5ce-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="cd5ce-104">この参照アーキテクチャでは、Azure で Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-104">This reference architecture shows a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="cd5ce-105">ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="cd5ce-106">このアーキテクチャは単一インスタンスを実行するために使用でき、n 層アプリケーションなどのさらに複雑なアーキテクチャの基礎となります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-106">This architecture can be used to run a single instance, and is the basis for more complex architectures such as n-tier applications.</span></span> [<span data-ttu-id="cd5ce-107">**このソリューションをデプロイします**。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="cd5ce-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="cd5ce-108">![[0]][0]</span></span>

<span data-ttu-id="cd5ce-109">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="cd5ce-109">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="cd5ce-110">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="cd5ce-110">Architecture</span></span>

<span data-ttu-id="cd5ce-111">Azure で VM をプロビジョニングする際は、VM 自体のみよりも、多くの変動的な部分があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-111">Provisioning a VM in Azure involves more moving parts than just the VM itself.</span></span> <span data-ttu-id="cd5ce-112">コンピューティング、ネットワーク、およびストレージの要素を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-112">There are compute, networking, and storage elements that you need to consider.</span></span>

* <span data-ttu-id="cd5ce-113">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-113">**Resource group.**</span></span> <span data-ttu-id="cd5ce-114">[*リソース グループ*][resource-manager-overview]は、関連リソースを保持するコンテナーです。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-114">A [*resource group*][resource-manager-overview] is a container that holds related resources.</span></span> <span data-ttu-id="cd5ce-115">通常は、ユーザーが有効期限に基づいてソリューション内のリソースごとにリソース グループを作成し、リソースを管理します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-115">You usually create resource groups for different resources in a solution based on their lifetime, and who will manage the resources.</span></span> <span data-ttu-id="cd5ce-116">単一の VM ワークロードでは、すべてのリソースに対して単一のリソース グループを作成する場合があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-116">For a single VM workload, you may create a single resource group for all resources.</span></span>
* <span data-ttu-id="cd5ce-117">**VM**。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-117">**VM**.</span></span> <span data-ttu-id="cd5ce-118">Azure は、CentOS、Debian、Red Hat Enterprise、Ubuntu、FreeBSD など、現在普及しているさまざまな Linux ディストリビューションに対応します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-118">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="cd5ce-119">詳細については、「[Azure と Linux][azure-linux]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-119">For more information, see [Azure and Linux][azure-linux].</span></span> <span data-ttu-id="cd5ce-120">VM は、発行されたイメージの一覧、または Azure BLOB ストレージにアップロードした仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-120">You can provision a VM from a list of published images or from a virtual hard disk (VHD) file that you upload to Azure Blob storage.</span></span>
* <span data-ttu-id="cd5ce-121">**OS ディスク。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-121">**OS disk.**</span></span> <span data-ttu-id="cd5ce-122">OS ディスクは、[Azure Storage][azure-storage] に格納されている VHD です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-122">The OS disk is a VHD stored in [Azure Storage][azure-storage].</span></span> <span data-ttu-id="cd5ce-123">これは、ホスト コンピューターがダウンした場合でも VM が保持されることを意味します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-123">That means it persists even if the host machine goes down.</span></span> <span data-ttu-id="cd5ce-124">OS ディスクは `/dev/sda1`です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-124">The OS disk is `/dev/sda1`.</span></span>
* <span data-ttu-id="cd5ce-125">**一時ディスク。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-125">**Temporary disk.**</span></span> <span data-ttu-id="cd5ce-126">VM は一時ディスクを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-126">The VM is created with a temporary disk.</span></span> <span data-ttu-id="cd5ce-127">このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-127">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="cd5ce-128">Azure Storage には "*保存されない*" ため、再起動中や他の VM ライフサイクル イベント時に削除される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-128">It is *not* saved in Azure Storage and might be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="cd5ce-129">ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-129">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="cd5ce-130">一時ディスクは `/dev/sdb1` で、`/mnt/resource` または `/mnt` でマウントされます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-130">The temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>
* <span data-ttu-id="cd5ce-131">**データ ディスク。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-131">**Data disks.**</span></span> <span data-ttu-id="cd5ce-132">[データ ディスク][data-disk]は、アプリケーション データ用の永続的な VHD です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-132">A [data disk][data-disk] is a persistent VHD used for application data.</span></span> <span data-ttu-id="cd5ce-133">データ ディスクは、OS ディスクと同様に、Azure Storage に格納されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-133">Data disks are stored in Azure Storage, like the OS disk.</span></span>
* <span data-ttu-id="cd5ce-134">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-134">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="cd5ce-135">Azure 上のすべての VM は、さらに複数のサブネットに分かれる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-135">Every VM in Azure is deployed into a VNet that is further divided into subnets.</span></span>
* <span data-ttu-id="cd5ce-136">**パブリック IP アドレス。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-136">**Public IP address.**</span></span> <span data-ttu-id="cd5ce-137">パブリック IP アドレスは、SSH 経由などで VM と通信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-137">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>
* <span data-ttu-id="cd5ce-138">**ネットワーク インターフェイス (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-138">**Network interface (NIC)**.</span></span> <span data-ttu-id="cd5ce-139">NIC を使用すると、VM は仮想ネットワークと通信できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-139">The NIC enables the VM to communicate with the virtual network.</span></span>
* <span data-ttu-id="cd5ce-140">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-140">**Network security group (NSG)**.</span></span> <span data-ttu-id="cd5ce-141">[NSG][nsg] は、サブネットへのネットワーク トラフィックを許可または拒否するために使用します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-141">The [NSG][nsg] is used to allow/deny network traffic to the subnet.</span></span> <span data-ttu-id="cd5ce-142">NSG は、個々 の NIC またはサブネットに関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-142">You can associate an NSG with an individual NIC or with a subnet.</span></span> <span data-ttu-id="cd5ce-143">NSG をサブネットに関連付けると、そのサブネット内のすべての VM に NSG ルールが適用されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-143">If you associate it with a subnet, the NSG rules apply to all VMs in that subnet.</span></span>
* <span data-ttu-id="cd5ce-144">**[診断]。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-144">**Diagnostics.**</span></span> <span data-ttu-id="cd5ce-145">診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-145">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="recommendations"></a><span data-ttu-id="cd5ce-146">Recommendations</span><span class="sxs-lookup"><span data-stu-id="cd5ce-146">Recommendations</span></span>

<span data-ttu-id="cd5ce-147">このアーキテクチャでは、Azure で Linux VM を実行するためのベースラインの推奨事項が示されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-147">This architecture shows the baseline recommendations for running a Linux VM in Azure.</span></span> <span data-ttu-id="cd5ce-148">ただし、単一障害点ができてしまうため、ミッション クリティカルなワークロードに単一の VM を使用することは推奨されません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-148">However, we don't recommend using a single VM for mission critical workloads because it creates a single point of failure.</span></span> <span data-ttu-id="cd5ce-149">可用性を高めるには、複数の VM を[可用性セット][availability-set]にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-149">For higher availability, deploy multiple VMs in an [availability set][availability-set].</span></span> <span data-ttu-id="cd5ce-150">詳細については、[Azure での複数の VM の実行][multi-vm]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-150">For more information, see [Running multiple VMs on Azure][multi-vm].</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="cd5ce-151">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-151">VM recommendations</span></span>

<span data-ttu-id="cd5ce-152">Azure にはさまざまな仮想マシン サイズが用意されていますが、DS シリーズと GS シリーズをお勧めします。これらのマシン サイズでは [Premium Storage][premium-storage] がサポートされるためです。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-152">Azure offers many different virtual machine sizes, but we recommend the DS- and GS-series because these machine sizes support [Premium Storage][premium-storage].</span></span> <span data-ttu-id="cd5ce-153">ハイパフォーマンス コンピューティングなどの特殊なワークロードがない限り、これらのマシン サイズのいずれかを選択してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-153">Select one of these machine sizes unless you have a specialized workload such as high-performance computing.</span></span> <span data-ttu-id="cd5ce-154">詳細については、[仮想マシンのサイズ][virtual-machine-sizes]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-154">For details, see [virtual machine sizes][virtual-machine-sizes].</span></span>

<span data-ttu-id="cd5ce-155">既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-155">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="cd5ce-156">次に、CPU、メモリ、およびディスクの 1 秒あたりの入力/出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-156">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size if needed.</span></span> <span data-ttu-id="cd5ce-157">VM 用に複数の NIC が必要な場合は、NIC の最大数が [VM のサイズ][vm-size-tables]と関係することに注意してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-157">If you require multiple NICs for your VM, be aware that the maximum number of NICs is a function of the [VM size][vm-size-tables].</span></span>

<span data-ttu-id="cd5ce-158">VM および他のリソースをプロビジョニングする際は、リージョンを指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-158">When you provision the VM and other resources, you must specify a region.</span></span> <span data-ttu-id="cd5ce-159">一般的に、内部ユーザーや顧客に最も近いリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-159">Generally, choose a region closest to your internal users or customers.</span></span> <span data-ttu-id="cd5ce-160">ただし、すべてのリージョンですべての VM サイズを利用できるとは限りません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-160">However, not all VM sizes may be available in all region.</span></span> <span data-ttu-id="cd5ce-161">詳細については、[リージョン別のサービス][services-by-region]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-161">For details, see [Services by region][services-by-region].</span></span> <span data-ttu-id="cd5ce-162">特定のリージョンで利用できる VM サイズを一覧表示するには、次の Azure コマンド ライン インターフェイス (CLI) コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-162">To list the VM sizes available in a specific region, run the following Azure command-line interface (CLI) command:</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="cd5ce-163">発行された VM イメージの選択については、「[Azure CLI を使用した Linux VM イメージの選択][select-vm-image]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-163">For information about choosing a published VM image, see [Select Linux VM images with the Azure CLI][select-vm-image].</span></span>

<span data-ttu-id="cd5ce-164">基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-164">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="cd5ce-165">VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-165">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="cd5ce-166">詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-166">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

### <a name="disk-and-storage-recommendations"></a><span data-ttu-id="cd5ce-167">ディスクとストレージの推奨事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-167">Disk and storage recommendations</span></span>

<span data-ttu-id="cd5ce-168">最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-168">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="cd5ce-169">コストは、プロビジョニングされたディスクのサイズに基づいて決まります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-169">Cost is based on the size of the provisioned disk.</span></span> <span data-ttu-id="cd5ce-170">また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-170">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="cd5ce-171">[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用することもお勧めします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-171">We also recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="cd5ce-172">管理ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-172">Managed disks do not require a storage account.</span></span> <span data-ttu-id="cd5ce-173">ディスクのサイズと種類を指定するだけで、可用性の高い方法でデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-173">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span>

<span data-ttu-id="cd5ce-174">管理ディスクを使用しない場合は、ストレージ アカウントの IOPS 制限に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-174">If you are not using managed disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs) in order to avoid hitting the IOPS limits for storage accounts.</span></span>

<span data-ttu-id="cd5ce-175">1 つ以上のデータ ディスクを追加します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-175">Add one or more data disks.</span></span> <span data-ttu-id="cd5ce-176">作成した VHD は、フォーマットされていません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-176">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="cd5ce-177">その VM にログインしてディスクをフォーマットしてください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-177">Log in to the VM to format the disk.</span></span> <span data-ttu-id="cd5ce-178">Linux のシェルでは、データ ディスクは `/dev/sdc`、`/dev/sdd` などのように表示されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-178">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="cd5ce-179">`lsblk` を実行すると、ディスクなどのブロック デバイスの一覧を表示できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-179">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="cd5ce-180">データ ディスクを使用するには、パーティションとファイル システムを作成し、ディスクをマウントします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-180">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="cd5ce-181">For example:</span><span class="sxs-lookup"><span data-stu-id="cd5ce-181">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="cd5ce-182">[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用せず、データ ディスクの数が多い場合は、ストレージ アカウントの合計 I/O 制限に注意してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-182">If you are not using [managed disks](/azure/storage/storage-managed-disks-overview) and have a large number of data disks, be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="cd5ce-183">詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-183">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

<span data-ttu-id="cd5ce-184">データ ディスクを追加すると、ディスクに論理ユニット番号 (LUN) の ID が割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-184">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="cd5ce-185">LUN ID は必要に応じて指定できます。たとえば、ディスクを交換する際に同じ LUN ID を保持したい場合や、特定の LUN ID を検索するアプリケーションがある場合などに指定します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-185">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="cd5ce-186">ただし、ディスクごとに一意な LUN ID である必要があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-186">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="cd5ce-187">Premium Storage アカウントで使用する VM のディスクは SSD なので、I/O スケジューラを変更して、SSD のパフォーマンスを最適化することができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-187">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="cd5ce-188">一般的な推奨事項は、SSD に対して NOOP スケジューラを使用することですが、[iostat] などのツールを使用してワークロードのディスク I/O パフォーマンスを監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-188">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="cd5ce-189">最適なパフォーマンスを得るには、診断ログを保持するためのストレージ アカウントを別途作成します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-189">For best performance, create a separate storage account to hold diagnostic logs.</span></span> <span data-ttu-id="cd5ce-190">診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-190">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="cd5ce-191">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-191">Network recommendations</span></span>

<span data-ttu-id="cd5ce-192">パブリック IP アドレスは、動的でも静的でもかまいません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-192">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="cd5ce-193">既定では、動的アドレスになっています。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-193">The default is dynamic.</span></span>

* <span data-ttu-id="cd5ce-194">変化しない固定 IP アドレスが必要な場合 (たとえば、DNS に A レコードを作成する必要がある場合や IP アドレスをホワイトリストに登録する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-194">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="cd5ce-195">IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-195">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="cd5ce-196">これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-196">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="cd5ce-197">詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-197">For more information, see [create a fully qualified domain name in the Azure portal][fqdn].</span></span>

<span data-ttu-id="cd5ce-198">すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-198">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="cd5ce-199">既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-199">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="cd5ce-200">インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-200">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="cd5ce-201">SSH を有効にするには、TCP ポート 22 への着信トラフィックを許可する規則を NSG に追加します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-201">To enable SSH, add a rule to the NSG that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="cd5ce-202">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-202">Scalability considerations</span></span>

<span data-ttu-id="cd5ce-203">VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-203">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="cd5ce-204">水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-204">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="cd5ce-205">詳細については、「[Running multiple VMs on Azure for scalability and availability ][multi-vm]」 (スケーラビリティと可用性のために Azure で複数の VM を実行する) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-205">For details, see [Running multiple VMs on Azure for scalability and availability][multi-vm].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="cd5ce-206">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-206">Availability considerations</span></span>

<span data-ttu-id="cd5ce-207">可用性を高めるには、複数の VM を可用性セットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-207">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="cd5ce-208">こうすることにより、[サービス レベル アグリーメント][vm-sla] (SLA) も高くなります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-208">This also provides a higher [service level agreement][vm-sla]  (SLA).</span></span>

<span data-ttu-id="cd5ce-209">VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-209">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="cd5ce-210">[VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-210">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="cd5ce-211">VHD は [Azure Storage][azure-storage] に格納され、Azure Storage は持続性と可用性を確保するためにレプリケートされます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-211">VHDs are stored in [Azure storage][azure-storage], and Azure storage is replicated for durability and availability.</span></span>

<span data-ttu-id="cd5ce-212">通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-212">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="cd5ce-213">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-213">Manageability considerations</span></span>

<span data-ttu-id="cd5ce-214">**リソース グループ**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-214">**Resource groups.**</span></span> <span data-ttu-id="cd5ce-215">同じライフ サイクルを共有する密結合のリソースを同じ[リソース グループ][resource-manager-overview]に配置します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-215">Put tightly coupled resources that share the same life cycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="cd5ce-216">リソース グループを使用すると、グループとしてリソースをデプロイおよび監視し、リソース グループ別に請求コストをまとめることができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-216">Resource groups allow you to deploy and monitor resources as a group, and roll up billing costs by resource group.</span></span> <span data-ttu-id="cd5ce-217">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-217">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="cd5ce-218">リソースにはわかりやすい名前を付けます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-218">Give resources meaningful names.</span></span> <span data-ttu-id="cd5ce-219">これにより、特定のリソースを見つけて、その役割を理解することが簡単になります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-219">That makes it easier to locate a specific resource and understand its role.</span></span> <span data-ttu-id="cd5ce-220">「[Recommended Naming Conventions for Azure Resources][naming conventions]」(Azure リソースの推奨される名前付け規則) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-220">See [Recommended Naming Conventions for Azure Resources][naming conventions].</span></span>

<span data-ttu-id="cd5ce-221">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-221">**SSH**.</span></span> <span data-ttu-id="cd5ce-222">Linux VM を作成する前に、2048 ビット RSA 公開/秘密キー ペアを生成します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-222">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="cd5ce-223">VM を作成する場合は、公開キー ファイルを使用します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-223">Use the public key file when you create the VM.</span></span> <span data-ttu-id="cd5ce-224">詳細については、[Azure 上の Linux または Mac における SSH の使用方法][ssh-linux]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-224">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="cd5ce-225">**VM の停止。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-225">**Stopping a VM.**</span></span> <span data-ttu-id="cd5ce-226">Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-226">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="cd5ce-227">VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-227">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> 

<span data-ttu-id="cd5ce-228">Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-228">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="cd5ce-229">ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "*解除されない*" ため、引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-229">If you shut down through the OS while logged in, the VM is stopped but *not* deallocated, so you will still be charged.</span></span> 

<span data-ttu-id="cd5ce-230">**VM の削除。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-230">**Deleting a VM.**</span></span> <span data-ttu-id="cd5ce-231">VM を削除しても VHD は削除されません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-231">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="cd5ce-232">つまり、データを失うことなく安全に VM を削除できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-232">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="cd5ce-233">ただし、Storage に対して引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-233">However, you will still be charged for storage.</span></span> <span data-ttu-id="cd5ce-234">VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-234">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span>

<span data-ttu-id="cd5ce-235">誤って削除されないように、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、VM などの個々のリソースをロックします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-235">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as the VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="cd5ce-236">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="cd5ce-236">Security considerations</span></span>

<span data-ttu-id="cd5ce-237">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-237">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="cd5ce-238">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-238">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="cd5ce-239">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-239">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="cd5ce-240">「[Azure Security Center クイック スタート ガイド][security-center-get-started]」に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-240">Enable security data collection as described in [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="cd5ce-241">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-241">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="cd5ce-242">**更新プログラムの管理。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-242">**Patch management.**</span></span> <span data-ttu-id="cd5ce-243">有効な場合、セキュリティ センターは、セキュリティと重要な更新プログラムが不足しているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-243">If enabled, Security Center checks whether security and critical updates are missing.</span></span> 

<span data-ttu-id="cd5ce-244">**マルウェア対策。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-244">**Antimalware.**</span></span> <span data-ttu-id="cd5ce-245">有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-245">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="cd5ce-246">セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-246">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="cd5ce-247">**操作。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-247">**Operations.**</span></span> <span data-ttu-id="cd5ce-248">[ロールベースのアクセス制御][rbac] (RBAC) を使用して、デプロイする Azure リソースへのアクセスを制御します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-248">Use [role-based access control][rbac] (RBAC) to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="cd5ce-249">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-249">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="cd5ce-250">たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-250">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="cd5ce-251">一部の役割は、特定の Azure リソースの種類に固有です。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-251">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="cd5ce-252">たとえば、仮想マシン作成協力者ロールでは、VM の再起動または割り当て解除、管理者パスワードのリセット、新しい VM の作成などができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-252">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so forth.</span></span> <span data-ttu-id="cd5ce-253">このアーキテクチャで役立つ他の[組み込み RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワーク共同作成者][rbac-network]などがあります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-253">Other [built-in RBAC roles][rbac-roles] that might be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="cd5ce-254">ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-254">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="cd5ce-255">RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-255">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="cd5ce-256">これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-256">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="cd5ce-257">プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-257">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="cd5ce-258">**データの暗号化。**</span><span class="sxs-lookup"><span data-stu-id="cd5ce-258">**Data encryption.**</span></span> <span data-ttu-id="cd5ce-259">OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-259">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="cd5ce-260">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="cd5ce-260">Deploy the solution</span></span>

<span data-ttu-id="cd5ce-261">このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-261">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="cd5ce-262">次のものがデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-262">It deploys the following:</span></span>

  * <span data-ttu-id="cd5ce-263">VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-263">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="cd5ce-264">VM に対する SSH と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-264">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="cd5ce-265">Ubuntu 16.04.3 LTS の最新バージョンを実行する VM。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-265">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="cd5ce-266">Apache HTTP Server を Ubuntu VM にデプロイし、2 つのデータ ディスクをフォーマットするカスタム スクリプト拡張機能の例。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-266">A sample custom script extension that deploys Apache HTTP Server to the Ubuntu VM, and formats the two data disks.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="cd5ce-267">前提条件</span><span class="sxs-lookup"><span data-stu-id="cd5ce-267">Prerequisites</span></span>

<span data-ttu-id="cd5ce-268">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-268">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="cd5ce-269">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-269">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="cd5ce-270">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-270">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="cd5ce-271">CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-271">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="cd5ce-272">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-272">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="cd5ce-273">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-273">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="cd5ce-274">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="cd5ce-274">Deploy the solution using azbb</span></span>

<span data-ttu-id="cd5ce-275">単一の VM ワークロードのサンプルをデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-275">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="cd5ce-276">上記の前提条件でダウンロードしたリポジトリの `virtual-machines\single-vm\parameters\linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-276">Navigate to the `virtual-machines\single-vm\parameters\linux` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="cd5ce-277">`single-vm-v2.json` ファイルを開き、次に示すような引用符の間にユーザー名と SSH キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-277">Open the `single-vm-v2.json` file and enter a username and SSH key between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminsshPublicKey": "",
  ```

3. <span data-ttu-id="cd5ce-278">次に示すように、`azbb` を実行して VM のサンプルをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-278">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="cd5ce-279">この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-279">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="cd5ce-280">次のステップ</span><span class="sxs-lookup"><span data-stu-id="cd5ce-280">Next steps</span></span>

- <span data-ttu-id="cd5ce-281">[Azure の構成要素][azbbv2]について確認します。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-281">Learn about our [Azure building Blocks][azbbv2].</span></span>
- <span data-ttu-id="cd5ce-282">Azure で[複数の VM][multi-vm] をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="cd5ce-282">Deploy [multiple VMs][multi-vm] in Azure.</span></span>

<!-- links -->
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[multi-vm]: ../virtual-machines-linux/multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[OSPatching]: https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[readme]: https://github.com/mspnp/reference-architectures/blob/master/virtual-machines/single-vm/README.md
[0]: ./images/single-vm-diagram.png "Azure における単一の Linux VM アーキテクチャ"

