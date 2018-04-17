---
title: Azure での Linux VM の実行
description: 拡張性、回復性、管理容易性、セキュリティに注目しながら、Azure で Linux VM を実行する方法。
author: telmosampaio
ms.date: 04/03/2018
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 50e23b00dd898c0b8e6230730ecf27323ee50d14
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="b2f31-103">Azure での Linux VM の実行</span><span class="sxs-lookup"><span data-stu-id="b2f31-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="b2f31-104">この参照アーキテクチャでは、Azure で Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-104">This reference architecture shows a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="b2f31-105">ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="b2f31-106">このアーキテクチャは単一の VM インスタンスを実行するために使用でき、N 層アプリケーションなどのさらに複雑なアーキテクチャの基礎となります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-106">This architecture can be used to run a single VM instance, and is the basis for more complex architectures such as N-tier applications.</span></span> [<span data-ttu-id="b2f31-107">**以下のソリューションをデプロイします。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-107">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="b2f31-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="b2f31-108">![[0]][0]</span></span>

<span data-ttu-id="b2f31-109">*このアーキテクチャ ダイアグラムを含む [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="b2f31-109">*Download a [Visio file][visio-download] that contains this architecture diagram.*</span></span>

## <a name="architecture"></a><span data-ttu-id="b2f31-110">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="b2f31-110">Architecture</span></span>

<span data-ttu-id="b2f31-111">Azure VM のプロビジョニングには、VM 自体の他に、ネットワーク リソース、ストレージ リソースなどの追加コンポーネントがいくつか必要です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-111">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

* <span data-ttu-id="b2f31-112">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-112">**Resource group.**</span></span> <span data-ttu-id="b2f31-113">[リソース グループ][resource-manager-overview]は、関連する Azure リソースを保持する論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="b2f31-113">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="b2f31-114">一般には、リソースは有効期間や管理者に基づいてグループ化します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-114">In general, group resources based on their lifetime and who will manage them.</span></span> 

* <span data-ttu-id="b2f31-115">**VM**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-115">**VM**.</span></span> <span data-ttu-id="b2f31-116">VM は、発行されたイメージの一覧や、Azure BLOB ストレージにアップロードされたカスタム管理されたイメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-116">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="b2f31-117">Azure は、CentOS、Debian、Red Hat Enterprise、Ubuntu、FreeBSD など、現在普及しているさまざまな Linux ディストリビューションに対応します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-117">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="b2f31-118">詳細については、「[Azure と Linux][azure-linux]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-118">For more information, see [Azure and Linux][azure-linux].</span></span>

* <span data-ttu-id="b2f31-119">**Managed Disks**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-119">**Managed Disks**.</span></span> <span data-ttu-id="b2f31-120">[Azure Managed Disks][managed-disks] は、ユーザーに代わってストレージを処理することでディスク管理を簡素化します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-120">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="b2f31-121">OS ディスクは [Azure Storage][azure-storage] に格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-121">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="b2f31-122">Linux VM の場合、OS ディスクは `/dev/sda1` です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-122">For Linux VMs, the OS disk is `/dev/sda1`.</span></span> <span data-ttu-id="b2f31-123">また、[データ ディスク][data-disk]を 1 つ以上作成することもお勧めします。これは、アプリケーション データに使用される永続的な VHD です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-123">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span> 

* <span data-ttu-id="b2f31-124">**一時ディスク。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-124">**Temporary disk.**</span></span> <span data-ttu-id="b2f31-125">VM は一時ディスクを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-125">The VM is created with a temporary disk.</span></span> <span data-ttu-id="b2f31-126">このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-126">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="b2f31-127">Azure Storage には保存され*ない*ため、再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-127">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="b2f31-128">ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-128">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="b2f31-129">Linux VM の場合、一時ディスクは `/dev/sdb1` であり、`/mnt/resource` または `/mnt` でマウントされます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-129">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>

* <span data-ttu-id="b2f31-130">**仮想ネットワーク (VNet)。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-130">**Virtual network (VNet).**</span></span> <span data-ttu-id="b2f31-131">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-131">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

* <span data-ttu-id="b2f31-132">**ネットワーク インターフェイス (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-132">**Network interface (NIC)**.</span></span> <span data-ttu-id="b2f31-133">NIC を使用すると、VM は仮想ネットワークと通信できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-133">The NIC enables the VM to communicate with the virtual network.</span></span>

* <span data-ttu-id="b2f31-134">**パブリック IP アドレス。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-134">**Public IP address.**</span></span> <span data-ttu-id="b2f31-135">パブリック IP アドレスは、SSH 経由などで VM と通信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-135">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>

* <span data-ttu-id="b2f31-136">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-136">**Azure DNS**.</span></span> <span data-ttu-id="b2f31-137">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-137">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="b2f31-138">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-138">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

* <span data-ttu-id="b2f31-139">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-139">**Network security group (NSG)**.</span></span> <span data-ttu-id="b2f31-140">[ネットワーク セキュリティ グループ][nsg]は、VM へのネットワーク トラフィックを許可または拒否するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-140">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="b2f31-141">NSG は、サブネットまたは個々の VM インスタンスと関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-141">NSGs can be associated either with subnets or with individual VM instances.</span></span>

* <span data-ttu-id="b2f31-142">**[診断]。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-142">**Diagnostics.**</span></span> <span data-ttu-id="b2f31-143">診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-143">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="recommendations"></a><span data-ttu-id="b2f31-144">Recommendations</span><span class="sxs-lookup"><span data-stu-id="b2f31-144">Recommendations</span></span>

<span data-ttu-id="b2f31-145">このアーキテクチャでは、Azure で Linux VM を実行するためのベースラインの推奨事項が示されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-145">This architecture shows the baseline recommendations for running a Linux VM in Azure.</span></span> <span data-ttu-id="b2f31-146">ただし、単一障害点ができてしまうため、ミッション クリティカルなワークロードに単一の VM を使用することは推奨されません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-146">However, we don't recommend using a single VM for mission critical workloads because it creates a single point of failure.</span></span> <span data-ttu-id="b2f31-147">高可用性を実現するために、2 つ以上のロード バランサー VM をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-147">For higher availability, deploy two or more load-balanced VMs.</span></span> <span data-ttu-id="b2f31-148">詳細については、[Azure での複数の VM の実行][multi-vm]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-148">For more information, see [Running multiple VMs on Azure][multi-vm].</span></span>

### <a name="vm-recommendations"></a><span data-ttu-id="b2f31-149">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-149">VM recommendations</span></span>

<span data-ttu-id="b2f31-150">Azure は、さまざまな仮想マシン サイズを提供します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-150">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="b2f31-151">詳細については、[Azure の仮想マシンのサイズ][virtual-machine-sizes]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-151">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="b2f31-152">既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-152">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="b2f31-153">次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-153">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="b2f31-154">VM 用に複数の NIC が必要な場合は、[VM サイズ][vm-size-tables]ごとに NIC の最大数が定義されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-154">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="b2f31-155">一般的に、内部ユーザーや顧客に最も近い Azure リージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-155">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="b2f31-156">ただし、すべてのリージョンですべての VM サイズを使用できるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-156">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="b2f31-157">詳細については、「[リージョン別のサービス][services-by-region]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-157">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="b2f31-158">特定のリージョンで使用できる VM サイズの一覧を表示するには、Azure コマンド ライン インターフェイス (CLI) から次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-158">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="b2f31-159">発行された VM イメージの選択については、[Linux VM イメージの検索][select-vm-image]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-159">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="b2f31-160">基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-160">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="b2f31-161">VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-161">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="b2f31-162">詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-162">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

### <a name="disk-and-storage-recommendations"></a><span data-ttu-id="b2f31-163">ディスクとストレージの推奨事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-163">Disk and storage recommendations</span></span>

<span data-ttu-id="b2f31-164">最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-164">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="b2f31-165">コストは、プロビジョニングされたディスクの容量に基づきます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-165">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="b2f31-166">また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-166">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="b2f31-167">[Managed Disks][managed-disks] を使用することもお勧めします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-167">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="b2f31-168">管理ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-168">Managed disks do not require a storage account.</span></span> <span data-ttu-id="b2f31-169">単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-169">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="b2f31-170">1 つ以上のデータ ディスクを追加します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-170">Add one or more data disks.</span></span> <span data-ttu-id="b2f31-171">作成した VHD は、フォーマットされていません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-171">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="b2f31-172">その VM にログインしてディスクをフォーマットしてください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-172">Log into the VM to format the disk.</span></span> <span data-ttu-id="b2f31-173">Linux のシェルでは、データ ディスクは `/dev/sdc`、`/dev/sdd` などのように表示されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-173">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="b2f31-174">`lsblk` を実行すると、ディスクなどのブロック デバイスの一覧を表示できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-174">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="b2f31-175">データ ディスクを使用するには、パーティションとファイル システムを作成し、ディスクをマウントします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-175">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="b2f31-176">例: </span><span class="sxs-lookup"><span data-stu-id="b2f31-176">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="b2f31-177">データ ディスクを追加すると、ディスクに論理ユニット番号 (LUN) の ID が割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-177">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="b2f31-178">LUN ID は必要に応じて指定できます。たとえば、ディスクを交換する際に同じ LUN ID を保持したい場合や、特定の LUN ID を検索するアプリケーションがある場合などに指定します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-178">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="b2f31-179">ただし、ディスクごとに一意な LUN ID である必要があります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-179">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="b2f31-180">Premium Storage アカウントで使用する VM のディスクは SSD なので、I/O スケジューラを変更して、SSD のパフォーマンスを最適化することができます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-180">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="b2f31-181">一般的な推奨事項は、SSD に対して NOOP スケジューラを使用することですが、[iostat] などのツールを使用してワークロードのディスク I/O パフォーマンスを監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-181">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="b2f31-182">診断ログを保持するストレージ アカウントを作成します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-182">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="b2f31-183">診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-183">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="b2f31-184">Managed Disks を使用しない場合は、ストレージ アカウントの [(IOPS) 制限][vm-disk-limits]に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-184">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="b2f31-185">ストレージ アカウントの合計 I/O 制限に注意してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-185">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="b2f31-186">詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-186">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>


### <a name="network-recommendations"></a><span data-ttu-id="b2f31-187">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-187">Network recommendations</span></span>

<span data-ttu-id="b2f31-188">パブリック IP アドレスは、動的でも静的でもかまいません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-188">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="b2f31-189">既定では、動的アドレスになっています。</span><span class="sxs-lookup"><span data-stu-id="b2f31-189">The default is dynamic.</span></span>

* <span data-ttu-id="b2f31-190">変化しない固定 IP アドレスが必要な場合 (たとえば、DNS に A レコードを作成する必要がある場合や IP アドレスをホワイトリストに登録する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-190">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="b2f31-191">IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-191">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="b2f31-192">これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-192">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="b2f31-193">詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-193">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span> <span data-ttu-id="b2f31-194">[Azure DNS][azure-dns] または別の DNS サービスを使用できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-194">You can use [Azure DNS][azure-dns] or another DNS service.</span></span>

<span data-ttu-id="b2f31-195">すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b2f31-195">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="b2f31-196">既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-196">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="b2f31-197">インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-197">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="b2f31-198">SSH を有効にするには、TCP ポート 22 への受信トラフィックを許可する NSG 規則を追加します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-198">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="b2f31-199">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-199">Scalability considerations</span></span>

<span data-ttu-id="b2f31-200">VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-200">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="b2f31-201">水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-201">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="b2f31-202">詳細については、「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-202">For more information, see [Run load-balanced VMs for scalability and availability][multi-vm].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="b2f31-203">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-203">Availability considerations</span></span>

<span data-ttu-id="b2f31-204">可用性を高めるには、複数の VM を可用性セットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-204">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="b2f31-205">これにより、より高度な[サービス レベル アグリーメント][vm-sla] (SLA) も提供されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-205">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="b2f31-206">VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-206">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="b2f31-207">[VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-207">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="b2f31-208">通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-208">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="b2f31-209">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-209">Manageability considerations</span></span>

<span data-ttu-id="b2f31-210">**リソース グループ**</span><span class="sxs-lookup"><span data-stu-id="b2f31-210">**Resource groups.**</span></span> <span data-ttu-id="b2f31-211">同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ][resource-manager-overview]に配置します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-211">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="b2f31-212">リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-212">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="b2f31-213">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-213">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="b2f31-214">特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-214">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="b2f31-215">詳細については、「[Recommended naming conventions for Azure resources][naming-conventions]」(Azure リソースの推奨される名前付け規則) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-215">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="b2f31-216">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="b2f31-216">**SSH**.</span></span> <span data-ttu-id="b2f31-217">Linux VM を作成する前に、2048 ビット RSA 公開/秘密キー ペアを生成します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-217">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="b2f31-218">VM を作成する場合は、公開キー ファイルを使用します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-218">Use the public key file when you create the VM.</span></span> <span data-ttu-id="b2f31-219">詳細については、[Azure 上の Linux または Mac における SSH の使用方法][ssh-linux]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-219">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="b2f31-220">**VM の停止。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-220">**Stopping a VM.**</span></span> <span data-ttu-id="b2f31-221">Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-221">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="b2f31-222">VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-222">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="b2f31-223">Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-223">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="b2f31-224">ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "**解除されない**" ため、引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-224">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="b2f31-225">**VM の削除。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-225">**Deleting a VM.**</span></span> <span data-ttu-id="b2f31-226">VM を削除しても VHD は削除されません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-226">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="b2f31-227">つまり、データを失うことなく安全に VM を削除できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-227">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="b2f31-228">ただし、Storage に対して引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-228">However, you will still be charged for storage.</span></span> <span data-ttu-id="b2f31-229">VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-229">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="b2f31-230">誤って削除されないようにするために、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-230">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="b2f31-231">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="b2f31-231">Security considerations</span></span>

<span data-ttu-id="b2f31-232">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-232">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="b2f31-233">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-233">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="b2f31-234">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-234">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="b2f31-235">「[Azure Security Center クイック スタート ガイド][security-center-get-started]」の説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-235">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="b2f31-236">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-236">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="b2f31-237">**更新プログラムの管理。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-237">**Patch management.**</span></span> <span data-ttu-id="b2f31-238">有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-238">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> 

<span data-ttu-id="b2f31-239">**マルウェア対策。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-239">**Antimalware.**</span></span> <span data-ttu-id="b2f31-240">有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-240">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="b2f31-241">セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-241">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="b2f31-242">**操作。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-242">**Operations.**</span></span> <span data-ttu-id="b2f31-243">[ロールベースのアクセス制御 (RBAC)][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-243">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="b2f31-244">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-244">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="b2f31-245">たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-245">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="b2f31-246">一部の役割は、特定の Azure リソースの種類に固有です。</span><span class="sxs-lookup"><span data-stu-id="b2f31-246">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="b2f31-247">たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-247">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="b2f31-248">このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワークの共同作業者][rbac-network]が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-248">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="b2f31-249">ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-249">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="b2f31-250">RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。</span><span class="sxs-lookup"><span data-stu-id="b2f31-250">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="b2f31-251">これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。</span><span class="sxs-lookup"><span data-stu-id="b2f31-251">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="b2f31-252">プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-252">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="b2f31-253">**データの暗号化。**</span><span class="sxs-lookup"><span data-stu-id="b2f31-253">**Data encryption.**</span></span> <span data-ttu-id="b2f31-254">OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-254">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="b2f31-255">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="b2f31-255">Deploy the solution</span></span>

<span data-ttu-id="b2f31-256">このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-256">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="b2f31-257">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-257">It deploys the following:</span></span>

  * <span data-ttu-id="b2f31-258">VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="b2f31-258">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="b2f31-259">VM に対する SSH と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。</span><span class="sxs-lookup"><span data-stu-id="b2f31-259">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="b2f31-260">Ubuntu 16.04.3 LTS の最新バージョンを実行する VM。</span><span class="sxs-lookup"><span data-stu-id="b2f31-260">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="b2f31-261">2 つのデータ ディスクをフォーマットし、Apache HTTP Server を Ubuntu VM にデプロイするサンプルのカスタム スクリプト拡張機能。</span><span class="sxs-lookup"><span data-stu-id="b2f31-261">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="b2f31-262">前提条件</span><span class="sxs-lookup"><span data-stu-id="b2f31-262">Prerequisites</span></span>

1. <span data-ttu-id="b2f31-263">[参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-263">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="b2f31-264">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-264">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="b2f31-265">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-265">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="b2f31-266">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-266">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="b2f31-267">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のコマンドを入力して Azure アカウントにログインします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-267">From a command prompt, bash prompt, or PowerShell prompt, enter the following command to log into your Azure account.</span></span>

   ```bash
   az login
   ```

5. <span data-ttu-id="b2f31-268">SSH キー ペアを作成します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-268">Create an SSH key pair.</span></span> <span data-ttu-id="b2f31-269">詳細については、「[Azure に Linux VM 用の SSH 公開キーと秘密キーのペアを作成して使用する方法](/azure/virtual-machines/linux/mac-create-ssh-keys)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-269">For more information, see [How to create and use an SSH public and private key pair for Linux VMs in Azure](/azure/virtual-machines/linux/mac-create-ssh-keys).</span></span>

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="b2f31-270">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="b2f31-270">Deploy the solution using azbb</span></span>

<span data-ttu-id="b2f31-271">この参照アーキテクチャをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-271">To deploy this reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="b2f31-272">上の前提条件の手順でダウンロードしたリポジトリの `virtual-machines/single-vm/parameters/linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-272">Navigate to the `virtual-machines/single-vm/parameters/linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="b2f31-273">`single-vm-v2.json` ファイルを開き、引用符の間にユーザー名と使用している SSH 公開キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-273">Open the `single-vm-v2.json` file and enter a username and your SSH public key between the quotes, then save the file.</span></span>

   ```bash
   "adminUsername": "<your username>",
   "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
   ```

3. <span data-ttu-id="b2f31-274">次に示すように、`azbb` を実行して VM のサンプルをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-274">Run `azbb` to deploy the sample VM as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
   ```

<span data-ttu-id="b2f31-275">デプロイを確認するには、次の Azure CLI コマンドを実行して、VM のパブリック IP アドレスを見つけます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-275">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```bash
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="b2f31-276">Web ブラウザーでこのアドレスに移動すると、既定の Apache2 ホーム ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="b2f31-276">If you navigate to this address in a web browser, you should see the default Apache2 homepage.</span></span>

<span data-ttu-id="b2f31-277">このデプロイのカスタマイズについては、[GitHub リポジトリ][git]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b2f31-277">For information about customizing this deployment, visit our [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="b2f31-278">次の手順</span><span class="sxs-lookup"><span data-stu-id="b2f31-278">Next steps</span></span>

- <span data-ttu-id="b2f31-279">[Azure の構成要素][azbbv2]について確認します。</span><span class="sxs-lookup"><span data-stu-id="b2f31-279">Learn about our [Azure building Blocks][azbbv2].</span></span>
- <span data-ttu-id="b2f31-280">Azure で[複数の VM][multi-vm] をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="b2f31-280">Deploy [multiple VMs][multi-vm] in Azure.</span></span>

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
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure における単一の Linux VM アーキテクチャ"
