---
title: Azure での Linux VM の実行
titleSuffix: Azure Reference Architectures
description: Azure での Linux 仮想マシンの実行に関するベスト プラクティス。
author: telmosampaio
ms.date: 09/13/2018
ms.custom: seodec18
ms.openlocfilehash: b8e0f758765f45eabf5fe3480c99cc54bf5d0ee8
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120052"
---
# <a name="run-a-linux-virtual-machine-on-azure"></a><span data-ttu-id="52304-103">Azure で Linux 仮想マシンを実行する</span><span class="sxs-lookup"><span data-stu-id="52304-103">Run a Linux virtual machine on Azure</span></span>

<span data-ttu-id="52304-104">この記事では、Azure で Linux 仮想マシン (VM) を実行するための一連の実証済みの方法を示します。</span><span class="sxs-lookup"><span data-stu-id="52304-104">This article shows proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="52304-105">ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。</span><span class="sxs-lookup"><span data-stu-id="52304-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="52304-106">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="52304-106">[**Deploy this solution**](#deploy-the-solution).</span></span>

[0]: ./images/single-vm-diagram.png "Azure における単一の Linux VM"

## <a name="components"></a><span data-ttu-id="52304-108">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="52304-108">Components</span></span>

<span data-ttu-id="52304-109">Azure VM のプロビジョニングには、VM 自体の他に、ネットワーク リソース、ストレージ リソースなどの追加コンポーネントがいくつか必要です。</span><span class="sxs-lookup"><span data-stu-id="52304-109">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

- <span data-ttu-id="52304-110">**リソース グループ**。</span><span class="sxs-lookup"><span data-stu-id="52304-110">**Resource group**.</span></span> <span data-ttu-id="52304-111">[リソース グループ][resource-manager-overview]は、関連する Azure リソースを保持する論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="52304-111">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="52304-112">一般には、リソースの有効期間や管理者に基づいて、リソースをグループ化します。</span><span class="sxs-lookup"><span data-stu-id="52304-112">In general, group resources based on their lifetime and who will manage them.</span></span>

- <span data-ttu-id="52304-113">**VM**。</span><span class="sxs-lookup"><span data-stu-id="52304-113">**VM**.</span></span> <span data-ttu-id="52304-114">VM は、発行されたイメージの一覧や、Azure BLOB ストレージにアップロードされたカスタム管理されたイメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="52304-114">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="52304-115">Azure は、CentOS、Debian、Red Hat Enterprise、Ubuntu、FreeBSD など、現在普及しているさまざまな Linux ディストリビューションに対応します。</span><span class="sxs-lookup"><span data-stu-id="52304-115">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="52304-116">詳細については、「[Azure と Linux][azure-linux]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-116">For more information, see [Azure and Linux][azure-linux].</span></span>

- <span data-ttu-id="52304-117">**Managed Disks**。</span><span class="sxs-lookup"><span data-stu-id="52304-117">**Managed Disks**.</span></span> <span data-ttu-id="52304-118">[Azure Managed Disks][managed-disks] は、ユーザーに代わってストレージを処理することでディスク管理を簡素化します。</span><span class="sxs-lookup"><span data-stu-id="52304-118">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="52304-119">OS ディスクは [Azure Storage][azure-storage] に格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。</span><span class="sxs-lookup"><span data-stu-id="52304-119">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="52304-120">Linux VM の場合、OS ディスクは `/dev/sda1` です。</span><span class="sxs-lookup"><span data-stu-id="52304-120">For Linux VMs, the OS disk is `/dev/sda1`.</span></span> <span data-ttu-id="52304-121">また、[データ ディスク][data-disk]を 1 つ以上作成することもお勧めします。データ ディスクは、アプリケーション データ用に使用される永続的な VHD です。</span><span class="sxs-lookup"><span data-stu-id="52304-121">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span>

- <span data-ttu-id="52304-122">**一時ディスク**。</span><span class="sxs-lookup"><span data-stu-id="52304-122">**Temporary disk**.</span></span> <span data-ttu-id="52304-123"> VM は一時ディスクを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="52304-123">The VM is created with a temporary disk.</span></span> <span data-ttu-id="52304-124">このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。</span><span class="sxs-lookup"><span data-stu-id="52304-124">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="52304-125">Azure Storage には保存され*ない*ため、再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="52304-125">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="52304-126">ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="52304-126">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="52304-127">Linux VM の場合、一時ディスクは `/dev/sdb1` であり、`/mnt/resource` または `/mnt` でマウントされます。</span><span class="sxs-lookup"><span data-stu-id="52304-127">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>

- <span data-ttu-id="52304-128">**仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="52304-128">**Virtual network (VNet)**.</span></span> <span data-ttu-id="52304-129">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="52304-129">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

- <span data-ttu-id="52304-130">**ネットワーク インターフェイス (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="52304-130">**Network interface (NIC)**.</span></span> <span data-ttu-id="52304-131">NIC を使用すると、VM は仮想ネットワークと通信できます。</span><span class="sxs-lookup"><span data-stu-id="52304-131">The NIC enables the VM to communicate with the virtual network.</span></span>

- <span data-ttu-id="52304-132">**パブリック IP アドレス**。</span><span class="sxs-lookup"><span data-stu-id="52304-132">**Public IP address**.</span></span> <span data-ttu-id="52304-133">パブリック IP アドレスは、SSH 経由などで VM と通信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="52304-133">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>

- <span data-ttu-id="52304-134">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="52304-134">**Azure DNS**.</span></span> <span data-ttu-id="52304-135">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="52304-135">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="52304-136">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="52304-136">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

- <span data-ttu-id="52304-137">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="52304-137">**Network security group (NSG)**.</span></span> <span data-ttu-id="52304-138">[ネットワーク セキュリティ グループ][nsg]は、VM へのネットワーク トラフィックを許可または拒否するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="52304-138">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="52304-139">NSG は、サブネットまたは個々の VM インスタンスと関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="52304-139">NSGs can be associated either with subnets or with individual VM instances.</span></span>

- <span data-ttu-id="52304-140">**診断**</span><span class="sxs-lookup"><span data-stu-id="52304-140">**Diagnostics**.</span></span> <span data-ttu-id="52304-141"> 診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="52304-141">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="vm-recommendations"></a><span data-ttu-id="52304-142">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="52304-142">VM recommendations</span></span>

<span data-ttu-id="52304-143">Azure は、さまざまな仮想マシン サイズを提供します。</span><span class="sxs-lookup"><span data-stu-id="52304-143">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="52304-144">詳細については、[Azure の仮想マシンのサイズ][virtual-machine-sizes]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="52304-144">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="52304-145">既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。</span><span class="sxs-lookup"><span data-stu-id="52304-145">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="52304-146">次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。</span><span class="sxs-lookup"><span data-stu-id="52304-146">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="52304-147">VM 用に複数の NIC が必要な場合は、[VM サイズ][vm-size-tables]ごとに NIC の最大数が定義されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="52304-147">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="52304-148">一般的に、内部ユーザーや顧客に最も近い Azure リージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="52304-148">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="52304-149">ただし、すべてのリージョンですべての VM サイズを使用できるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="52304-149">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="52304-150">詳細については、「[リージョン別のサービス][services-by-region]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="52304-150">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="52304-151">特定のリージョンで使用できる VM サイズの一覧を表示するには、Azure コマンド ライン インターフェイス (CLI) から次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="52304-151">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```azurecli
az vm list-sizes --location <location>
```

<span data-ttu-id="52304-152">発行された VM イメージの選択については、[Linux VM イメージの検索][select-vm-image]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-152">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="52304-153">基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。</span><span class="sxs-lookup"><span data-stu-id="52304-153">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="52304-154">VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。</span><span class="sxs-lookup"><span data-stu-id="52304-154">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="52304-155">詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-155">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>

## <a name="disk-and-storage-recommendations"></a><span data-ttu-id="52304-156">ディスクとストレージの推奨事項</span><span class="sxs-lookup"><span data-stu-id="52304-156">Disk and storage recommendations</span></span>

<span data-ttu-id="52304-157">最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="52304-157">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="52304-158">コストは、プロビジョニングされたディスクの容量に基づきます。</span><span class="sxs-lookup"><span data-stu-id="52304-158">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="52304-159">また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="52304-159">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span>

<span data-ttu-id="52304-160">[Managed Disks][managed-disks] を使用することもお勧めします。</span><span class="sxs-lookup"><span data-stu-id="52304-160">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="52304-161">マネージド ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="52304-161">Managed disks do not require a storage account.</span></span> <span data-ttu-id="52304-162">単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="52304-162">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="52304-163">1 つ以上のデータ ディスクを追加します。</span><span class="sxs-lookup"><span data-stu-id="52304-163">Add one or more data disks.</span></span> <span data-ttu-id="52304-164">作成した VHD は、フォーマットされていません。</span><span class="sxs-lookup"><span data-stu-id="52304-164">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="52304-165">その VM にログインしてディスクをフォーマットしてください。</span><span class="sxs-lookup"><span data-stu-id="52304-165">Log into the VM to format the disk.</span></span> <span data-ttu-id="52304-166">Linux のシェルでは、データ ディスクは `/dev/sdc`、`/dev/sdd` などのように表示されます。</span><span class="sxs-lookup"><span data-stu-id="52304-166">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="52304-167">`lsblk` を実行すると、ディスクなどのブロック デバイスの一覧を表示できます。</span><span class="sxs-lookup"><span data-stu-id="52304-167">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="52304-168">データ ディスクを使用するには、パーティションとファイル システムを作成し、ディスクをマウントします。</span><span class="sxs-lookup"><span data-stu-id="52304-168">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="52304-169">例: </span><span class="sxs-lookup"><span data-stu-id="52304-169">For example:</span></span>

```bash
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="52304-170">データ ディスクを追加すると、ディスクに論理ユニット番号 (LUN) の ID が割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="52304-170">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="52304-171">LUN ID は必要に応じて指定できます。たとえば、ディスクを交換する際に同じ LUN ID を保持したい場合や、特定の LUN ID を検索するアプリケーションがある場合などに指定します。</span><span class="sxs-lookup"><span data-stu-id="52304-171">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="52304-172">ただし、ディスクごとに一意な LUN ID である必要があります。</span><span class="sxs-lookup"><span data-stu-id="52304-172">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="52304-173">Premium Storage アカウントで使用する VM のディスクは SSD なので、I/O スケジューラを変更して、SSD のパフォーマンスを最適化することができます。</span><span class="sxs-lookup"><span data-stu-id="52304-173">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="52304-174">一般的な推奨事項は、SSD に対して NOOP スケジューラを使用することですが、[iostat] などのツールを使用してワークロードのディスク I/O パフォーマンスを監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="52304-174">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="52304-175">診断ログを保持するストレージ アカウントを作成します。</span><span class="sxs-lookup"><span data-stu-id="52304-175">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="52304-176">診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。</span><span class="sxs-lookup"><span data-stu-id="52304-176">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="52304-177">Managed Disks を使用しない場合は、ストレージ アカウントの [(IOPS) 制限][vm-disk-limits]に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。</span><span class="sxs-lookup"><span data-stu-id="52304-177">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="52304-178">ストレージ アカウントの合計 I/O 制限に注意してください。</span><span class="sxs-lookup"><span data-stu-id="52304-178">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="52304-179">詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-179">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

## <a name="network-recommendations"></a><span data-ttu-id="52304-180">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="52304-180">Network recommendations</span></span>

<span data-ttu-id="52304-181">パブリック IP アドレスは、動的でも静的でもかまいません。</span><span class="sxs-lookup"><span data-stu-id="52304-181">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="52304-182">既定では、動的アドレスになっています。</span><span class="sxs-lookup"><span data-stu-id="52304-182">The default is dynamic.</span></span>

- <span data-ttu-id="52304-183">変化しない固定 IP アドレスが必要な場合 (たとえば、DNS に A レコードを作成する必要がある場合や IP アドレスをホワイトリストに登録する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。</span><span class="sxs-lookup"><span data-stu-id="52304-183">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
- <span data-ttu-id="52304-184">IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="52304-184">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="52304-185">これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。</span><span class="sxs-lookup"><span data-stu-id="52304-185">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="52304-186">詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="52304-186">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span> <span data-ttu-id="52304-187">[Azure DNS][azure-dns] または別の DNS サービスを使用できます。</span><span class="sxs-lookup"><span data-stu-id="52304-187">You can use [Azure DNS][azure-dns] or another DNS service.</span></span>

<span data-ttu-id="52304-188">すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="52304-188">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="52304-189">既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。</span><span class="sxs-lookup"><span data-stu-id="52304-189">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="52304-190">インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="52304-190">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="52304-191">SSH を有効にするには、TCP ポート 22 への受信トラフィックを許可する NSG 規則を追加します。</span><span class="sxs-lookup"><span data-stu-id="52304-191">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="52304-192">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="52304-192">Scalability considerations</span></span>

<span data-ttu-id="52304-193">VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="52304-193">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="52304-194">水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="52304-194">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="52304-195">詳細については、[n 層参照アーキテクチャ](./n-tier-cassandra.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-195">For more information, see the [N-tier reference architecture](./n-tier-cassandra.md).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="52304-196">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="52304-196">Availability considerations</span></span>

<span data-ttu-id="52304-197">可用性を高めるには、複数の VM を可用性セットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="52304-197">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="52304-198">これにより、より高度な[サービス レベル アグリーメント][vm-sla] (SLA) も提供されます。</span><span class="sxs-lookup"><span data-stu-id="52304-198">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="52304-199">VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="52304-199">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="52304-200">[VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="52304-200">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="52304-201">通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。</span><span class="sxs-lookup"><span data-stu-id="52304-201">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="52304-202">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="52304-202">Manageability considerations</span></span>

<span data-ttu-id="52304-203">**リソース グループ**。</span><span class="sxs-lookup"><span data-stu-id="52304-203">**Resource groups**.</span></span> <span data-ttu-id="52304-204">同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ][resource-manager-overview]に配置します。</span><span class="sxs-lookup"><span data-stu-id="52304-204">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="52304-205">リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。</span><span class="sxs-lookup"><span data-stu-id="52304-205">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="52304-206">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="52304-206">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="52304-207">特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。</span><span class="sxs-lookup"><span data-stu-id="52304-207">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="52304-208">詳細については、「[Recommended naming conventions for Azure resources][naming-conventions]」(Azure リソースの推奨される名前付け規則) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-208">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="52304-209">**SSH**。</span><span class="sxs-lookup"><span data-stu-id="52304-209">**SSH**.</span></span> <span data-ttu-id="52304-210">Linux VM を作成する前に、2048 ビット RSA 公開/秘密キー ペアを生成します。</span><span class="sxs-lookup"><span data-stu-id="52304-210">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="52304-211">VM を作成する場合は、公開キー ファイルを使用します。</span><span class="sxs-lookup"><span data-stu-id="52304-211">Use the public key file when you create the VM.</span></span> <span data-ttu-id="52304-212">詳細については、[Azure 上の Linux または Mac における SSH の使用方法][ssh-linux]に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-212">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="52304-213">**VM の停止**。</span><span class="sxs-lookup"><span data-stu-id="52304-213">**Stopping a VM**.</span></span> <span data-ttu-id="52304-214">Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。</span><span class="sxs-lookup"><span data-stu-id="52304-214">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="52304-215">VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。</span><span class="sxs-lookup"><span data-stu-id="52304-215">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="52304-216">Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。</span><span class="sxs-lookup"><span data-stu-id="52304-216">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="52304-217">ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "**解除されない**" ため、引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="52304-217">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="52304-218">**VM の削除**。</span><span class="sxs-lookup"><span data-stu-id="52304-218">**Deleting a VM**.</span></span> <span data-ttu-id="52304-219"> VM を削除しても VHD は削除されません。</span><span class="sxs-lookup"><span data-stu-id="52304-219">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="52304-220">つまり、データを失うことなく安全に VM を削除できます。</span><span class="sxs-lookup"><span data-stu-id="52304-220">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="52304-221">ただし、Storage に対して引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="52304-221">However, you will still be charged for storage.</span></span> <span data-ttu-id="52304-222">VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。</span><span class="sxs-lookup"><span data-stu-id="52304-222">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="52304-223">誤って削除されないようにするために、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。</span><span class="sxs-lookup"><span data-stu-id="52304-223">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="52304-224">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="52304-224">Security considerations</span></span>

<span data-ttu-id="52304-225">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="52304-225">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="52304-226">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="52304-226">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="52304-227">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="52304-227">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="52304-228">「[Azure Security Center クイック スタート ガイド][security-center-get-started]」の説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="52304-228">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="52304-229">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="52304-229">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="52304-230">**更新プログラムの管理**。</span><span class="sxs-lookup"><span data-stu-id="52304-230">**Patch management**.</span></span> <span data-ttu-id="52304-231">有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。</span><span class="sxs-lookup"><span data-stu-id="52304-231">If enabled, Security Center checks whether any security and critical updates are missing.</span></span>

<span data-ttu-id="52304-232">**マルウェア対策**。</span><span class="sxs-lookup"><span data-stu-id="52304-232">**Antimalware**.</span></span> <span data-ttu-id="52304-233"> 有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="52304-233">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="52304-234">セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。</span><span class="sxs-lookup"><span data-stu-id="52304-234">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="52304-235">**操作**。</span><span class="sxs-lookup"><span data-stu-id="52304-235">**Operations**.</span></span> <span data-ttu-id="52304-236">[ロールベースのアクセス制御 (RBAC)][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御します。</span><span class="sxs-lookup"><span data-stu-id="52304-236">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="52304-237">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="52304-237">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="52304-238">たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。</span><span class="sxs-lookup"><span data-stu-id="52304-238">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="52304-239">一部の役割は、特定の Azure リソースの種類に固有です。</span><span class="sxs-lookup"><span data-stu-id="52304-239">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="52304-240">たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。</span><span class="sxs-lookup"><span data-stu-id="52304-240">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="52304-241">このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワークの共同作業者][rbac-network]が含まれます。</span><span class="sxs-lookup"><span data-stu-id="52304-241">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="52304-242">ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="52304-242">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="52304-243">RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。</span><span class="sxs-lookup"><span data-stu-id="52304-243">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="52304-244">これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。</span><span class="sxs-lookup"><span data-stu-id="52304-244">Those permissions are determined by the account type on the guest OS.</span></span>

<span data-ttu-id="52304-245">プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。</span><span class="sxs-lookup"><span data-stu-id="52304-245">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="52304-246">**データの暗号化**。</span><span class="sxs-lookup"><span data-stu-id="52304-246">**Data encryption**.</span></span> <span data-ttu-id="52304-247">OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。</span><span class="sxs-lookup"><span data-stu-id="52304-247">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span>

<span data-ttu-id="52304-248">**DDoS 保護**。</span><span class="sxs-lookup"><span data-stu-id="52304-248">**DDoS protection**.</span></span> <span data-ttu-id="52304-249">[DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview) を有効にすることをお勧めします。これにより、VNet 内のリソースに対して DDoS の軽減策が追加されます。</span><span class="sxs-lookup"><span data-stu-id="52304-249">We recommend enabling [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), which provides additional DDoS mitigation for resources in a VNet.</span></span> <span data-ttu-id="52304-250">Azure プラットフォームの一部として基本な DDoS 保護が自動的に有効になりますが、DDoS Protection Standard により、特に Azure Virtual Network リソース向けにチューニングされた軽減機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="52304-250">Although basic DDoS protection is automatically enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure Virtual Network resources.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="52304-251">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="52304-251">Deploy the solution</span></span>

<span data-ttu-id="52304-252">デプロイは、[GitHub][github-folder] で実行できます。</span><span class="sxs-lookup"><span data-stu-id="52304-252">A deployment is available on [GitHub][github-folder].</span></span> <span data-ttu-id="52304-253">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="52304-253">It deploys the following:</span></span>

- <span data-ttu-id="52304-254">VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="52304-254">A virtual network with a single subnet named **web** used to host the VM.</span></span>
- <span data-ttu-id="52304-255">VM に対する SSH と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。</span><span class="sxs-lookup"><span data-stu-id="52304-255">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
- <span data-ttu-id="52304-256">Ubuntu 16.04.3 LTS の最新バージョンを実行する VM。</span><span class="sxs-lookup"><span data-stu-id="52304-256">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
- <span data-ttu-id="52304-257">2 つのデータ ディスクをフォーマットし、Apache HTTP Server を Ubuntu VM にデプロイするサンプルのカスタム スクリプト拡張機能。</span><span class="sxs-lookup"><span data-stu-id="52304-257">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="52304-258">前提条件</span><span class="sxs-lookup"><span data-stu-id="52304-258">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="52304-259">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="52304-259">Deploy the solution using azbb</span></span>

1. <span data-ttu-id="52304-260">SSH キー ペアを作成します。</span><span class="sxs-lookup"><span data-stu-id="52304-260">Create an SSH key pair.</span></span> <span data-ttu-id="52304-261">詳細については、「[Azure に Linux VM 用の SSH 公開キーと秘密キーのペアを作成して使用する方法](/azure/virtual-machines/linux/mac-create-ssh-keys)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="52304-261">For more information, see [How to create and use an SSH public and private key pair for Linux VMs in Azure](/azure/virtual-machines/linux/mac-create-ssh-keys).</span></span>

2. <span data-ttu-id="52304-262">上の前提条件の手順でダウンロードしたリポジトリの `virtual-machines/single-vm/parameters/linux` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="52304-262">Navigate to the `virtual-machines/single-vm/parameters/linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

3. <span data-ttu-id="52304-263">`single-vm-v2.json` ファイルを開き、引用符の間にユーザー名と使用している SSH 公開キーを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="52304-263">Open the `single-vm-v2.json` file and enter a username and your SSH public key between the quotes, then save the file.</span></span>

    ```json
    "adminUsername": "<your username>",
    "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
    ```

4. <span data-ttu-id="52304-264">次に示すように、`azbb` を実行して VM のサンプルをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="52304-264">Run `azbb` to deploy the sample VM as shown below.</span></span>

    ```azurecli
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
    ```

<span data-ttu-id="52304-265">デプロイを確認するには、次の Azure CLI コマンドを実行して、VM のパブリック IP アドレスを見つけます。</span><span class="sxs-lookup"><span data-stu-id="52304-265">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```azurecli
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="52304-266">Web ブラウザーでこのアドレスに移動すると、既定の Apache2 ホーム ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="52304-266">If you navigate to this address in a web browser, you should see the default Apache2 homepage.</span></span>

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
[naming-conventions]: ../../best-practices/naming-conventions.md
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
