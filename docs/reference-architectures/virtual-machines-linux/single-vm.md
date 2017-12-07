---
title: "Azure での Linux VM の実行"
description: "拡張性、回復性、管理容易性、セキュリティに注目しながら、Azure で Linux VM を実行する方法。"
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f538958be934ad2e9ea8d53791814b1e963c1a20
ms.sourcegitcommit: 115db7ee008a0b1f2b0be50a26471050742ddb04
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/17/2017
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="68360-103">Azure での Linux VM の実行</span><span class="sxs-lookup"><span data-stu-id="68360-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="68360-104">この参照アーキテクチャでは、Azure で Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。</span><span class="sxs-lookup"><span data-stu-id="68360-104">This reference architecture shows a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="68360-105">ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。</span><span class="sxs-lookup"><span data-stu-id="68360-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="68360-106">このアーキテクチャは単一の VM インスタンスを実行するために使用でき、N 層アプリケーションなどのさらに複雑なアーキテクチャの基礎となります。</span><span class="sxs-lookup"><span data-stu-id="68360-106">This architecture can be used to run a single VM instance, and is the basis for more complex architectures such as N-tier applications.</span></span> [<span data-ttu-id="68360-107">**このソリューションをデプロイします。**</span><span class="sxs-lookup"><span data-stu-id="68360-107">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="68360-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="68360-108">![[0]][0]</span></span>

<span data-ttu-id="68360-109">*このアーキテクチャ ダイアグラムを含む [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="68360-109">*Download a [Visio file][visio-download] that contains this architecture diagram.*</span></span>

## <a name="architecture"></a><span data-ttu-id="68360-110">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="68360-110">Architecture</span></span>

<span data-ttu-id="68360-111">Azure VM のプロビジョニングには、コンピューティング、ネットワーク、およびストレージ リソースなどの追加コンポーネントが必要です。</span><span class="sxs-lookup"><span data-stu-id="68360-111">Provisioning an Azure VM requires additional components, such as compute, networking, and storage resources.</span></span>

* <span data-ttu-id="68360-112">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="68360-112">**Resource group.**</span></span> <span data-ttu-id="68360-113">[*リソース グループ*][resource-manager-overview]は、関連リソースを保持するコンテナーです。</span><span class="sxs-lookup"><span data-stu-id="68360-113">A [*resource group*][resource-manager-overview] is a container that holds related resources.</span></span> <span data-ttu-id="68360-114">一般には、リソースの有効期間や、そのリソースを誰が管理するかに基づいてソリューション内のリソースをグループ化する必要があります。</span><span class="sxs-lookup"><span data-stu-id="68360-114">In general, you should group resources in a solution based on their lifetime and who will manage the resources.</span></span> <span data-ttu-id="68360-115">単一の VM ワークロードの場合は、すべてのリソースに対して単一のリソース グループを作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="68360-115">For a single VM workload, you may want to create a single resource group for all resources.</span></span>
* <span data-ttu-id="68360-116">**VM**。</span><span class="sxs-lookup"><span data-stu-id="68360-116">**VM**.</span></span> <span data-ttu-id="68360-117">VM は、発行されたイメージの一覧や、Azure BLOB ストレージにアップロードされたカスタム管理されたイメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="68360-117">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="68360-118">Azure は、CentOS、Debian、Red Hat Enterprise、Ubuntu、FreeBSD など、現在普及しているさまざまな Linux ディストリビューションに対応します。</span><span class="sxs-lookup"><span data-stu-id="68360-118">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="68360-119">詳細については、「[Azure と Linux][azure-linux]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-119">For more information, see [Azure and Linux][azure-linux].</span></span>
* <span data-ttu-id="68360-120">**OS ディスク。**</span><span class="sxs-lookup"><span data-stu-id="68360-120">**OS disk.**</span></span> <span data-ttu-id="68360-121">OS ディスクは [Azure Storage][azure-storage] に格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。</span><span class="sxs-lookup"><span data-stu-id="68360-121">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="68360-122">Linux VM の場合、OS ディスクは `/dev/sda1` です。</span><span class="sxs-lookup"><span data-stu-id="68360-122">For Linux VMs, the OS disk is `/dev/sda1`.</span></span>
* <span data-ttu-id="68360-123">**一時ディスク。**</span><span class="sxs-lookup"><span data-stu-id="68360-123">**Temporary disk.**</span></span> <span data-ttu-id="68360-124">VM は一時ディスクを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="68360-124">The VM is created with a temporary disk.</span></span> <span data-ttu-id="68360-125">このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。</span><span class="sxs-lookup"><span data-stu-id="68360-125">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="68360-126">Azure Storage には保存され*ない*ため、再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="68360-126">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="68360-127">ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="68360-127">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="68360-128">Linux VM の場合、一時ディスクは `/dev/sdb1` であり、`/mnt/resource` または `/mnt` でマウントされます。</span><span class="sxs-lookup"><span data-stu-id="68360-128">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>
* <span data-ttu-id="68360-129">**データ ディスク。**</span><span class="sxs-lookup"><span data-stu-id="68360-129">**Data disks.**</span></span> <span data-ttu-id="68360-130">[データ ディスク][data-disk]は、アプリケーション データ用の永続的な VHD です。</span><span class="sxs-lookup"><span data-stu-id="68360-130">A [data disk][data-disk] is a persistent VHD used for application data.</span></span> <span data-ttu-id="68360-131">データ ディスクは、OS ディスクと同様に、Azure Storage に格納されます。</span><span class="sxs-lookup"><span data-stu-id="68360-131">Data disks are stored in Azure Storage, like the OS disk.</span></span>
* <span data-ttu-id="68360-132">**仮想ネットワーク (VNet) とサブネット。**</span><span class="sxs-lookup"><span data-stu-id="68360-132">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="68360-133">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="68360-133">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>
* <span data-ttu-id="68360-134">**パブリック IP アドレス。**</span><span class="sxs-lookup"><span data-stu-id="68360-134">**Public IP address.**</span></span> <span data-ttu-id="68360-135">パブリック IP アドレスは、SSH 経由などで VM と通信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="68360-135">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>
* <span data-ttu-id="68360-136">**ネットワーク インターフェイス (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="68360-136">**Network interface (NIC)**.</span></span> <span data-ttu-id="68360-137">割り当てられた NIC により、VM は仮想ネットワークと通信できます。</span><span class="sxs-lookup"><span data-stu-id="68360-137">An assigned NIC enables the VM to communicate with the virtual network.</span></span>
* <span data-ttu-id="68360-138">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="68360-138">**Network security group (NSG)**.</span></span> <span data-ttu-id="68360-139">[NSG][nsg] は、ネットワーク リソースへのネットワーク トラフィックを許可または拒否するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="68360-139">[NSGs][nsg] are used to allow or deny network traffic to a network resource.</span></span> <span data-ttu-id="68360-140">NSG は、個々 の NIC またはサブネットに関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="68360-140">You can associate an NSG with an individual NIC or with a subnet.</span></span> <span data-ttu-id="68360-141">NSG をサブネットに関連付けると、そのサブネット内のすべての VM に NSG ルールが適用されます。</span><span class="sxs-lookup"><span data-stu-id="68360-141">If you associate it with a subnet, the NSG rules apply to all VMs in that subnet.</span></span>
* <span data-ttu-id="68360-142">**[診断]。**</span><span class="sxs-lookup"><span data-stu-id="68360-142">**Diagnostics.**</span></span> <span data-ttu-id="68360-143">診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="68360-143">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="recommendations"></a><span data-ttu-id="68360-144">Recommendations</span><span class="sxs-lookup"><span data-stu-id="68360-144">Recommendations</span></span>

<span data-ttu-id="68360-145">このアーキテクチャでは、Azure で Linux VM を実行するためのベースラインの推奨事項が示されます。</span><span class="sxs-lookup"><span data-stu-id="68360-145">This architecture shows the baseline recommendations for running a Linux VM in Azure.</span></span> <span data-ttu-id="68360-146">ただし、単一障害点ができてしまうため、ミッション クリティカルなワークロードに単一の VM を使用することは推奨されません。</span><span class="sxs-lookup"><span data-stu-id="68360-146">However, we don't recommend using a single VM for mission critical workloads because it creates a single point of failure.</span></span> <span data-ttu-id="68360-147">可用性を高めるには、複数の VM を[可用性セット][availability-set]にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="68360-147">For higher availability, deploy multiple VMs in an [availability set][availability-set].</span></span> <span data-ttu-id="68360-148">詳細については、[Azure での複数の VM の実行][multi-vm]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="68360-148">For more information, see [Running multiple VMs on Azure][multi-vm].</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="68360-149">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="68360-149">VM recommendations</span></span>

<span data-ttu-id="68360-150">Azure は、さまざまな仮想マシン サイズを提供します。</span><span class="sxs-lookup"><span data-stu-id="68360-150">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="68360-151">[Premium Storage][premium-storage] は高パフォーマンスと短い待機時間のために推奨され、[特定の VM サイズでサポートされます][premium-storage-supported]。</span><span class="sxs-lookup"><span data-stu-id="68360-151">[Premium Storage][premium-storage] is recommended due to its high performance and low latency, and is [supported by specific VM sizes][premium-storage-supported].</span></span> <span data-ttu-id="68360-152">ハイ パフォーマンス コンピューティングなどの特殊なワークロードがない限り、これらのサイズのいずれかを選択してください。</span><span class="sxs-lookup"><span data-stu-id="68360-152">Select one of these sizes unless you have a specialized workload such as high-performance computing.</span></span> <span data-ttu-id="68360-153">詳細については、[仮想マシンのサイズ][virtual-machine-sizes]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-153">For more information, see [virtual machine sizes][virtual-machine-sizes].</span></span>

<span data-ttu-id="68360-154">既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。</span><span class="sxs-lookup"><span data-stu-id="68360-154">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="68360-155">次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) に関して実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。</span><span class="sxs-lookup"><span data-stu-id="68360-155">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="68360-156">VM 用に複数の NIC が必要な場合は、[VM サイズ][vm-size-tables]ごとに NIC の最大数が定義されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="68360-156">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="68360-157">Azure リソースをプロビジョニングする場合は、リージョンを指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="68360-157">When you provision Azure resources, you must specify a region.</span></span> <span data-ttu-id="68360-158">一般的に、内部ユーザーや顧客に最も近いリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="68360-158">Generally, choose a region closest to your internal users or customers.</span></span> <span data-ttu-id="68360-159">ただし、すべてのリージョンですべての VM サイズを使用できるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="68360-159">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="68360-160">詳細については、[リージョン別のサービス][services-by-region]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-160">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="68360-161">特定のリージョンで使用できる VM サイズの一覧を表示するには、Azure コマンド ライン インターフェイス (CLI) から次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="68360-161">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="68360-162">発行された VM イメージの選択については、[Linux VM イメージの検索][select-vm-image]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-162">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="68360-163">基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。</span><span class="sxs-lookup"><span data-stu-id="68360-163">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="68360-164">VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。</span><span class="sxs-lookup"><span data-stu-id="68360-164">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="68360-165">詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-165">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

### <a name="disk-and-storage-recommendations"></a><span data-ttu-id="68360-166">ディスクとストレージの推奨事項</span><span class="sxs-lookup"><span data-stu-id="68360-166">Disk and storage recommendations</span></span>

<span data-ttu-id="68360-167">最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="68360-167">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="68360-168">コストは、プロビジョニングされたディスクの容量に基づきます。</span><span class="sxs-lookup"><span data-stu-id="68360-168">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="68360-169">また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="68360-169">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="68360-170">[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用することもお勧めします。</span><span class="sxs-lookup"><span data-stu-id="68360-170">We also recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="68360-171">管理ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="68360-171">Managed disks do not require a storage account.</span></span> <span data-ttu-id="68360-172">単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="68360-172">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="68360-173">非管理対象ディスクを使用している場合は、ストレージ アカウントの [ (IOPS) の制限][vm-disk-limits]に達しないようにするために、仮想ハード ディスク (VHD) を保持するための個別の Azure ストレージ アカウントを VM ごとに作成します。</span><span class="sxs-lookup"><span data-stu-id="68360-173">If you are using unmanaged disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span>

<span data-ttu-id="68360-174">1 つ以上のデータ ディスクを追加します。</span><span class="sxs-lookup"><span data-stu-id="68360-174">Add one or more data disks.</span></span> <span data-ttu-id="68360-175">作成した VHD は、フォーマットされていません。</span><span class="sxs-lookup"><span data-stu-id="68360-175">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="68360-176">その VM にログインしてディスクをフォーマットしてください。</span><span class="sxs-lookup"><span data-stu-id="68360-176">Log into the VM to format the disk.</span></span> <span data-ttu-id="68360-177">管理ディスクを使用せず、データ ディスクの数が多い場合は、ストレージ アカウントの合計 I/O 制限に注意してください。</span><span class="sxs-lookup"><span data-stu-id="68360-177">If you are not using managed disks and have a large number of data disks, be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="68360-178">詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-178">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

<span data-ttu-id="68360-179">Linux のシェルでは、データ ディスクは `/dev/sdc`、`/dev/sdd` などのように表示されます。</span><span class="sxs-lookup"><span data-stu-id="68360-179">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="68360-180">`lsblk` を実行すると、ディスクなどのブロック デバイスの一覧を表示できます。</span><span class="sxs-lookup"><span data-stu-id="68360-180">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="68360-181">データ ディスクを使用するには、パーティションとファイル システムを作成し、ディスクをマウントします。</span><span class="sxs-lookup"><span data-stu-id="68360-181">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="68360-182">For example:</span><span class="sxs-lookup"><span data-stu-id="68360-182">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="68360-183">データ ディスクを追加すると、ディスクに論理ユニット番号 (LUN) の ID が割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="68360-183">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="68360-184">LUN ID は必要に応じて指定できます。たとえば、ディスクを交換する際に同じ LUN ID を保持したい場合や、特定の LUN ID を検索するアプリケーションがある場合などに指定します。</span><span class="sxs-lookup"><span data-stu-id="68360-184">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="68360-185">ただし、ディスクごとに一意な LUN ID である必要があります。</span><span class="sxs-lookup"><span data-stu-id="68360-185">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="68360-186">Premium Storage アカウントで使用する VM のディスクは SSD なので、I/O スケジューラを変更して、SSD のパフォーマンスを最適化することができます。</span><span class="sxs-lookup"><span data-stu-id="68360-186">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="68360-187">一般的な推奨事項は、SSD に対して NOOP スケジューラを使用することですが、[iostat] などのツールを使用してワークロードのディスク I/O パフォーマンスを監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="68360-187">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="68360-188">パフォーマンスを最大化するには、診断ログを保持するための個別のストレージ アカウントを作成します。</span><span class="sxs-lookup"><span data-stu-id="68360-188">To maximize performance, create a separate storage account to hold diagnostic logs.</span></span> <span data-ttu-id="68360-189">診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。</span><span class="sxs-lookup"><span data-stu-id="68360-189">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="68360-190">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="68360-190">Network recommendations</span></span>

<span data-ttu-id="68360-191">パブリック IP アドレスは、動的でも静的でもかまいません。</span><span class="sxs-lookup"><span data-stu-id="68360-191">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="68360-192">既定では、動的アドレスになっています。</span><span class="sxs-lookup"><span data-stu-id="68360-192">The default is dynamic.</span></span>

* <span data-ttu-id="68360-193">変化しない固定 IP アドレスが必要な場合 (たとえば、DNS に A レコードを作成する必要がある場合や IP アドレスをホワイトリストに登録する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。</span><span class="sxs-lookup"><span data-stu-id="68360-193">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="68360-194">IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="68360-194">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="68360-195">これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。</span><span class="sxs-lookup"><span data-stu-id="68360-195">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="68360-196">詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-196">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span>

<span data-ttu-id="68360-197">すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="68360-197">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="68360-198">既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。</span><span class="sxs-lookup"><span data-stu-id="68360-198">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="68360-199">インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="68360-199">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="68360-200">SSH を有効にするには、TCP ポート 22 への受信トラフィックを許可する NSG 規則を追加します。</span><span class="sxs-lookup"><span data-stu-id="68360-200">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="68360-201">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="68360-201">Scalability considerations</span></span>

<span data-ttu-id="68360-202">VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="68360-202">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="68360-203">水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="68360-203">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="68360-204">詳細については、「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-204">For more information, see [Running multiple VMs on Azure for scalability and availability][multi-vm].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="68360-205">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="68360-205">Availability considerations</span></span>

<span data-ttu-id="68360-206">可用性を高めるには、複数の VM を可用性セットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="68360-206">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="68360-207">これにより、より高度な[サービス レベル アグリーメント][vm-sla] (SLA) も提供されます。</span><span class="sxs-lookup"><span data-stu-id="68360-207">This also provides a higher [service level agreement][vm-sla](SLA).</span></span>

<span data-ttu-id="68360-208">VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="68360-208">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="68360-209">[VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="68360-209">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="68360-210">VHD は [Azure Storage][azure-storage] に格納されます。</span><span class="sxs-lookup"><span data-stu-id="68360-210">VHDs are stored in [Azure storage][azure-storage].</span></span> <span data-ttu-id="68360-211">Azure Storage は、持続性と可用性を確保するためにレプリケートされます。</span><span class="sxs-lookup"><span data-stu-id="68360-211">Azure storage is replicated for durability and availability.</span></span>

<span data-ttu-id="68360-212">通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。</span><span class="sxs-lookup"><span data-stu-id="68360-212">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="68360-213">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="68360-213">Manageability considerations</span></span>

<span data-ttu-id="68360-214">**リソース グループ**</span><span class="sxs-lookup"><span data-stu-id="68360-214">**Resource groups.**</span></span> <span data-ttu-id="68360-215">同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ][resource-manager-overview]に配置します。</span><span class="sxs-lookup"><span data-stu-id="68360-215">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="68360-216">リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。</span><span class="sxs-lookup"><span data-stu-id="68360-216">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="68360-217">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="68360-217">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="68360-218">特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。</span><span class="sxs-lookup"><span data-stu-id="68360-218">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="68360-219">詳細については、「[Recommended naming conventions for Azure resources][naming-conventions]」(Azure リソースの推奨される名前付け規則) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-219">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="68360-220">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="68360-220">**SSH**.</span></span> <span data-ttu-id="68360-221">Linux VM を作成する前に、2048 ビット RSA 公開/秘密キー ペアを生成します。</span><span class="sxs-lookup"><span data-stu-id="68360-221">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="68360-222">VM を作成する場合は、公開キー ファイルを使用します。</span><span class="sxs-lookup"><span data-stu-id="68360-222">Use the public key file when you create the VM.</span></span> <span data-ttu-id="68360-223">詳細については、[Azure 上の Linux または Mac における SSH の使用方法][ssh-linux]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-223">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="68360-224">**VM の停止。**</span><span class="sxs-lookup"><span data-stu-id="68360-224">**Stopping a VM.**</span></span> <span data-ttu-id="68360-225">Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。</span><span class="sxs-lookup"><span data-stu-id="68360-225">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="68360-226">VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。</span><span class="sxs-lookup"><span data-stu-id="68360-226">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span>

<span data-ttu-id="68360-227">Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。</span><span class="sxs-lookup"><span data-stu-id="68360-227">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="68360-228">ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "*解除されない*" ため、引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="68360-228">If you shut down through the OS while logged in, the VM is stopped but *not* deallocated, so you will still be charged.</span></span>

<span data-ttu-id="68360-229">**VM の削除。**</span><span class="sxs-lookup"><span data-stu-id="68360-229">**Deleting a VM.**</span></span> <span data-ttu-id="68360-230">VM を削除しても VHD は削除されません。</span><span class="sxs-lookup"><span data-stu-id="68360-230">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="68360-231">つまり、データを失うことなく安全に VM を削除できます。</span><span class="sxs-lookup"><span data-stu-id="68360-231">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="68360-232">ただし、Storage に対して引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="68360-232">However, you will still be charged for storage.</span></span> <span data-ttu-id="68360-233">VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。</span><span class="sxs-lookup"><span data-stu-id="68360-233">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span>

<span data-ttu-id="68360-234">誤って削除されないようにするために、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。</span><span class="sxs-lookup"><span data-stu-id="68360-234">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="68360-235">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="68360-235">Security considerations</span></span>

<span data-ttu-id="68360-236">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="68360-236">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="68360-237">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="68360-237">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="68360-238">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="68360-238">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="68360-239">「[Azure Security Center クイック スタート ガイド][security-center-get-started]」の説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="68360-239">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="68360-240">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="68360-240">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="68360-241">**更新プログラムの管理。**</span><span class="sxs-lookup"><span data-stu-id="68360-241">**Patch management.**</span></span> <span data-ttu-id="68360-242">有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。</span><span class="sxs-lookup"><span data-stu-id="68360-242">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> 

<span data-ttu-id="68360-243">**マルウェア対策。**</span><span class="sxs-lookup"><span data-stu-id="68360-243">**Antimalware.**</span></span> <span data-ttu-id="68360-244">有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="68360-244">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="68360-245">セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。</span><span class="sxs-lookup"><span data-stu-id="68360-245">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="68360-246">**操作。**</span><span class="sxs-lookup"><span data-stu-id="68360-246">**Operations.**</span></span> <span data-ttu-id="68360-247">[ロールベースのアクセス制御 (RBAC)][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御します。</span><span class="sxs-lookup"><span data-stu-id="68360-247">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="68360-248">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="68360-248">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="68360-249">たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。</span><span class="sxs-lookup"><span data-stu-id="68360-249">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="68360-250">一部の役割は、特定の Azure リソースの種類に固有です。</span><span class="sxs-lookup"><span data-stu-id="68360-250">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="68360-251">たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。</span><span class="sxs-lookup"><span data-stu-id="68360-251">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="68360-252">このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワークの共同作業者][rbac-network]が含まれます。</span><span class="sxs-lookup"><span data-stu-id="68360-252">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="68360-253">ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="68360-253">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="68360-254">RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。</span><span class="sxs-lookup"><span data-stu-id="68360-254">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="68360-255">これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。</span><span class="sxs-lookup"><span data-stu-id="68360-255">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="68360-256">プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。</span><span class="sxs-lookup"><span data-stu-id="68360-256">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="68360-257">**データの暗号化。**</span><span class="sxs-lookup"><span data-stu-id="68360-257">**Data encryption.**</span></span> <span data-ttu-id="68360-258">OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。</span><span class="sxs-lookup"><span data-stu-id="68360-258">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="68360-259">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="68360-259">Deploy the solution</span></span>

<span data-ttu-id="68360-260">このアーキテクチャのデプロイについては、[GitHub][github-folder] をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="68360-260">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="68360-261">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="68360-261">It deploys the following:</span></span>

  * <span data-ttu-id="68360-262">VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="68360-262">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="68360-263">VM に対する SSH と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。</span><span class="sxs-lookup"><span data-stu-id="68360-263">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="68360-264">Ubuntu 16.04.3 LTS の最新バージョンを実行する VM。</span><span class="sxs-lookup"><span data-stu-id="68360-264">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="68360-265">2 つのデータ ディスクをフォーマットし、Apache HTTP Server を Ubuntu VM にデプロイするサンプルのカスタム スクリプト拡張機能。</span><span class="sxs-lookup"><span data-stu-id="68360-265">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="68360-266">前提条件</span><span class="sxs-lookup"><span data-stu-id="68360-266">Prerequisites</span></span>

<span data-ttu-id="68360-267">参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="68360-267">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="68360-268">[AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="68360-268">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="68360-269">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="68360-269">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="68360-270">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-270">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="68360-271">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="68360-271">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="68360-272">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="68360-272">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="68360-273">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="68360-273">Deploy the solution using azbb</span></span>

<span data-ttu-id="68360-274">単一の VM ワークロードのサンプルをデプロイするには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="68360-274">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="68360-275">上の前提条件の手順でダウンロードしたリポジトリの `virtual-machines\single-vm\parameters\linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="68360-275">Navigate to the `virtual-machines\single-vm\parameters\linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="68360-276">`single-vm-v2.json` ファイルを開き、次に示すように引用符の間にユーザー名と SSH 公開キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="68360-276">Open the `single-vm-v2.json` file and enter a username and SSH public key between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. <span data-ttu-id="68360-277">次に示すように、`azbb` を実行して VM のサンプルをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="68360-277">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="68360-278">この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="68360-278">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="68360-279">次のステップ</span><span class="sxs-lookup"><span data-stu-id="68360-279">Next steps</span></span>

- <span data-ttu-id="68360-280">[Azure の構成要素][azbbv2]について確認します。</span><span class="sxs-lookup"><span data-stu-id="68360-280">Learn about our [Azure building Blocks][azbbv2].</span></span>
- <span data-ttu-id="68360-281">Azure で[複数の VM][multi-vm] をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="68360-281">Deploy [multiple VMs][multi-vm] in Azure.</span></span>

<!-- links -->
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
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
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
[0]: ./images/single-vm-diagram.png "Azure における単一の Linux VM アーキテクチャ"
