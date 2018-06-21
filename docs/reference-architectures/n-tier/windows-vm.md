---
title: Azure での Windows VM の実行
description: スケーラビリティ、回復性、管理容易性、およびセキュリティに注意しながら、Azure で Windows VM を実行する方法。
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: 748bcfcf599c7fa0e82691632d14e0f78fd78582
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2018
ms.locfileid: "33673604"
---
# <a name="run-a-windows-vm-on-azure"></a><span data-ttu-id="c78bd-103">Azure での Windows VM の実行</span><span class="sxs-lookup"><span data-stu-id="c78bd-103">Run a Windows VM on Azure</span></span>

<span data-ttu-id="c78bd-104">この記事では、Azure で Windows 仮想マシン (VM) を実行するための一連の実証済みの方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-104">This article describes a set of proven practices for running a Windows virtual machine (VM) on Azure.</span></span> <span data-ttu-id="c78bd-105">ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> [<span data-ttu-id="c78bd-106">**以下のソリューションをデプロイします。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-106">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="c78bd-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="c78bd-107">![[0]][0]</span></span>

## <a name="components"></a><span data-ttu-id="c78bd-108">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="c78bd-108">Components</span></span>

<span data-ttu-id="c78bd-109">Azure VM のプロビジョニングには、VM 自体の他に、ネットワーク リソース、ストレージ リソースなどの追加コンポーネントがいくつか必要です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-109">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

* <span data-ttu-id="c78bd-110">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-110">**Resource group.**</span></span> <span data-ttu-id="c78bd-111">[リソース グループ][resource-manager-overview]は、関連する Azure リソースを保持する論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="c78bd-111">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="c78bd-112">一般には、リソースの有効期間や管理者に基づいて、リソースをグループ化します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-112">In general, group resources based on their lifetime and who will manage them.</span></span> 

* <span data-ttu-id="c78bd-113">**VM**。</span><span class="sxs-lookup"><span data-stu-id="c78bd-113">**VM**.</span></span> <span data-ttu-id="c78bd-114">VM は、発行されたイメージの一覧や、Azure BLOB ストレージにアップロードされたカスタム管理されたイメージまたは仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-114">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span>

* <span data-ttu-id="c78bd-115">**Managed Disks**。</span><span class="sxs-lookup"><span data-stu-id="c78bd-115">**Managed Disks**.</span></span> <span data-ttu-id="c78bd-116">[Azure Managed Disks][managed-disks] は、ユーザーに代わってストレージを処理することでディスク管理を簡素化します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-116">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="c78bd-117">OS ディスクは [Azure Storage][azure-storage] に格納された VHD であるため、ホスト マシンが停止している場合でも維持されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-117">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="c78bd-118">また、[データ ディスク][data-disk]を 1 つ以上作成することもお勧めします。データ ディスクは、アプリケーション データ用に使用される永続的な VHD です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-118">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span>

* <span data-ttu-id="c78bd-119">**一時ディスク。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-119">**Temporary disk.**</span></span> <span data-ttu-id="c78bd-120">VM は一時ディスク (Windows の `D:` ドライブ) を使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-120">The VM is created with a temporary disk (the `D:` drive on Windows).</span></span> <span data-ttu-id="c78bd-121">このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-121">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="c78bd-122">Azure Storage には保存され*ない*ため、再起動やその他の VM ライフサイクル イベント中に削除される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c78bd-122">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="c78bd-123">ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-123">Use this disk only for temporary data, such as page or swap files.</span></span>

* <span data-ttu-id="c78bd-124">**仮想ネットワーク (VNet)。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-124">**Virtual network (VNet).**</span></span> <span data-ttu-id="c78bd-125">どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-125">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

* <span data-ttu-id="c78bd-126">**ネットワーク インターフェイス (NIC)**。</span><span class="sxs-lookup"><span data-stu-id="c78bd-126">**Network interface (NIC)**.</span></span> <span data-ttu-id="c78bd-127">NIC を使用すると、VM は仮想ネットワークと通信できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-127">The NIC enables the VM to communicate with the virtual network.</span></span>  

* <span data-ttu-id="c78bd-128">**パブリック IP アドレス。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-128">**Public IP address.**</span></span> <span data-ttu-id="c78bd-129">パブリック IP アドレスは、リモート デスクトップ (RDP) 経由などで VM &mdash; と通信するために必要です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-129">A public IP address is needed to communicate with the VM &mdash; for example, via remote desktop (RDP).</span></span>  

* <span data-ttu-id="c78bd-130">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="c78bd-130">**Azure DNS**.</span></span> <span data-ttu-id="c78bd-131">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-131">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="c78bd-132">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-132">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

* <span data-ttu-id="c78bd-133">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="c78bd-133">**Network security group (NSG)**.</span></span> <span data-ttu-id="c78bd-134">[ネットワーク セキュリティ グループ][nsg]は、VM へのネットワーク トラフィックを許可または拒否するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-134">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="c78bd-135">NSG は、サブネットまたは個々の VM インスタンスと関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-135">NSGs can be associated either with subnets or with individual VM instances.</span></span> 

* <span data-ttu-id="c78bd-136">**[診断]。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-136">**Diagnostics.**</span></span> <span data-ttu-id="c78bd-137">診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-137">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="vm-recommendations"></a><span data-ttu-id="c78bd-138">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-138">VM recommendations</span></span>

<span data-ttu-id="c78bd-139">Azure は、さまざまな仮想マシン サイズを提供します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-139">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="c78bd-140">詳細については、[Azure の仮想マシンのサイズ][virtual-machine-sizes]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-140">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="c78bd-141">既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-141">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="c78bd-142">次に、CPU、メモリ、およびディスクの 1 秒あたりの入出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-142">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="c78bd-143">VM 用に複数の NIC が必要な場合は、[VM サイズ][vm-size-tables]ごとに NIC の最大数が定義されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-143">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="c78bd-144">一般的に、内部ユーザーや顧客に最も近い Azure リージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-144">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="c78bd-145">ただし、すべてのリージョンですべての VM サイズを使用できるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-145">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="c78bd-146">詳細については、「[リージョン別のサービス][services-by-region]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-146">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="c78bd-147">特定のリージョンで使用できる VM サイズの一覧を表示するには、Azure コマンド ライン インターフェイス (CLI) から次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-147">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="c78bd-148">発行された VM イメージの選択については、「[Find Windows VM images (Windows VM イメージを検索する)][select-vm-image]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-148">For information about choosing a published VM image, see [Find Windows VM images][select-vm-image].</span></span>

<span data-ttu-id="c78bd-149">基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-149">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="c78bd-150">VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-150">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="c78bd-151">詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-151">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

## <a name="disk-and-storage-recommendations"></a><span data-ttu-id="c78bd-152">ディスクとストレージの推奨事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-152">Disk and storage recommendations</span></span>

<span data-ttu-id="c78bd-153">最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-153">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="c78bd-154">コストは、プロビジョニングされたディスクの容量に基づきます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-154">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="c78bd-155">また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-155">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="c78bd-156">[Managed Disks][managed-disks] を使用することもお勧めします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-156">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="c78bd-157">管理ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-157">Managed disks do not require a storage account.</span></span> <span data-ttu-id="c78bd-158">単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-158">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="c78bd-159">1 つ以上のデータ ディスクを追加します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-159">Add one or more data disks.</span></span> <span data-ttu-id="c78bd-160">作成した VHD は、フォーマットされていません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-160">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="c78bd-161">その VM にログインしてディスクをフォーマットしてください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-161">Log into the VM to format the disk.</span></span> <span data-ttu-id="c78bd-162">可能であれば、OS ディスクではなく、データ ディスクにアプリケーションをインストールします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-162">When possible, install applications on a data disk, not the OS disk.</span></span> <span data-ttu-id="c78bd-163">一部のレガシー アプリケーションでは、C: ドライブへのコンポーネントのインストールが必要になることがあります。その場合は、PowerShell を使用して [OS ディスクのサイズを変更][resize-os-disk]できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-163">Some legacy applications might need to install components on the C: drive; in that case, you can [resize the OS disk][resize-os-disk] using PowerShell.</span></span>

<span data-ttu-id="c78bd-164">診断ログを保持するストレージ アカウントを作成します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-164">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="c78bd-165">診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-165">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="c78bd-166">Managed Disks を使用しない場合は、ストレージ アカウントの [(IOPS) 制限][vm-disk-limits]に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-166">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="c78bd-167">ストレージ アカウントの合計 I/O 制限に注意してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-167">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="c78bd-168">詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-168">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>


## <a name="network-recommendations"></a><span data-ttu-id="c78bd-169">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-169">Network recommendations</span></span>

<span data-ttu-id="c78bd-170">パブリック IP アドレスは、動的でも静的でもかまいません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-170">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="c78bd-171">既定では、動的アドレスになっています。</span><span class="sxs-lookup"><span data-stu-id="c78bd-171">The default is dynamic.</span></span>

* <span data-ttu-id="c78bd-172">変化しない固定 IP アドレスが必要な場合 &mdash; (たとえば、DNS 'A' レコードを作成するか、またはセーフ リストに IP アドレスを追加する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-172">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create a DNS 'A' record or add the IP address to a safe list.</span></span>
* <span data-ttu-id="c78bd-173">IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-173">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="c78bd-174">これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-174">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="c78bd-175">詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-175">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span>

<span data-ttu-id="c78bd-176">すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="c78bd-176">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="c78bd-177">既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-177">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="c78bd-178">インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-178">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="c78bd-179">RDP を有効にするには、TCP ポート 3389 への着信トラフィックを許可する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-179">To enable RDP, add an NSG rule that allows inbound traffic to TCP port 3389.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="c78bd-180">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-180">Scalability considerations</span></span>

<span data-ttu-id="c78bd-181">VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-181">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="c78bd-182">水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-182">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="c78bd-183">詳細については、[n 層参照アーキテクチャ](./n-tier-sql-server.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-183">For more information, see the [N-tier reference architecture](./n-tier-sql-server.md).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="c78bd-184">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-184">Availability considerations</span></span>

<span data-ttu-id="c78bd-185">可用性を高めるには、複数の VM を可用性セットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-185">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="c78bd-186">これにより、より高度な[サービス レベル アグリーメント][vm-sla] (SLA) も提供されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-186">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="c78bd-187">VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c78bd-187">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="c78bd-188">[VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-188">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="c78bd-189">通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-189">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="c78bd-190">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-190">Manageability considerations</span></span>

<span data-ttu-id="c78bd-191">**リソース グループ**</span><span class="sxs-lookup"><span data-stu-id="c78bd-191">**Resource groups.**</span></span> <span data-ttu-id="c78bd-192">同じライフサイクルを共有する密接に関連付けられたリソースを同じ[リソース グループ][resource-manager-overview]に配置します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-192">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="c78bd-193">リソース グループを使用すると、リソースをグループとしてデプロイおよび監視したり、リソース グループ別に課金コストを追跡したりできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-193">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="c78bd-194">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-194">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="c78bd-195">特定のリソースの検索やその役割の理解を簡略化するために、意味のあるリソース名を割り当ててください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-195">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="c78bd-196">詳細については、「[Azure リソースの推奨される名前付け規則][naming-conventions]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-196">For more information, see [Recommended Naming Conventions for Azure Resources][naming-conventions].</span></span>

<span data-ttu-id="c78bd-197">**VM の停止。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-197">**Stopping a VM.**</span></span> <span data-ttu-id="c78bd-198">Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-198">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="c78bd-199">VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-199">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="c78bd-200">Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-200">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="c78bd-201">ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "**解除されない**" ため、引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-201">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="c78bd-202">**VM の削除。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-202">**Deleting a VM.**</span></span> <span data-ttu-id="c78bd-203">VM を削除しても VHD は削除されません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-203">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="c78bd-204">つまり、データを失うことなく安全に VM を削除できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-204">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="c78bd-205">ただし、Storage に対して引き続き課金されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-205">However, you will still be charged for storage.</span></span> <span data-ttu-id="c78bd-206">VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-206">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="c78bd-207">誤って削除されないようにするために、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、または個別のリソース (VM など) をロックします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-207">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="c78bd-208">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="c78bd-208">Security considerations</span></span>

<span data-ttu-id="c78bd-209">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-209">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="c78bd-210">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-210">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="c78bd-211">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-211">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="c78bd-212">「[Azure Security Center クイック スタート ガイド][security-center-get-started]」の説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-212">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="c78bd-213">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-213">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="c78bd-214">**更新プログラムの管理。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-214">**Patch management.**</span></span> <span data-ttu-id="c78bd-215">有効になっている場合、Security Center はセキュリティ更新プログラムや緊急更新プログラムが不足しているかどうかをチェックします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-215">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> <span data-ttu-id="c78bd-216">VM で[グループ ポリシー設定][group-policy]を使用して、システムの自動更新を有効にします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-216">Use [Group Policy settings][group-policy] on the VM to enable automatic system updates.</span></span>

<span data-ttu-id="c78bd-217">**マルウェア対策。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-217">**Antimalware.**</span></span> <span data-ttu-id="c78bd-218">有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-218">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="c78bd-219">セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-219">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="c78bd-220">**操作。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-220">**Operations.**</span></span> <span data-ttu-id="c78bd-221">[ロールベースのアクセス制御 (RBAC)][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-221">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="c78bd-222">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-222">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="c78bd-223">たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-223">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="c78bd-224">一部の役割は、特定の Azure リソースの種類に固有です。</span><span class="sxs-lookup"><span data-stu-id="c78bd-224">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="c78bd-225">たとえば、仮想マシンの共同作業者ロールは VM を再起動または割り当て解除したり、管理者パスワードをリセットしたり、新しい VM を作成したりできます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-225">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="c78bd-226">このアーキテクチャで役立つ可能性のあるその他の[組み込みの RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワークの共同作業者][rbac-network]が含まれます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-226">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="c78bd-227">ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-227">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="c78bd-228">RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。</span><span class="sxs-lookup"><span data-stu-id="c78bd-228">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="c78bd-229">これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。</span><span class="sxs-lookup"><span data-stu-id="c78bd-229">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="c78bd-230">プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-230">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="c78bd-231">**データの暗号化。**</span><span class="sxs-lookup"><span data-stu-id="c78bd-231">**Data encryption.**</span></span> <span data-ttu-id="c78bd-232">OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-232">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="c78bd-233">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="c78bd-233">Deploy the solution</span></span>

<span data-ttu-id="c78bd-234">このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-234">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="c78bd-235">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-235">It deploys the following:</span></span>

  * <span data-ttu-id="c78bd-236">VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。</span><span class="sxs-lookup"><span data-stu-id="c78bd-236">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="c78bd-237">VM に対する RDP と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。</span><span class="sxs-lookup"><span data-stu-id="c78bd-237">An NSG with two incoming rules to allow RDP and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="c78bd-238">Windows Server 2016 Datacenter Edition の最新バージョンを実行する VM。</span><span class="sxs-lookup"><span data-stu-id="c78bd-238">A VM running the latest version of Windows Server 2016 Datacenter Edition.</span></span>
  * <span data-ttu-id="c78bd-239">2 つのデータ ディスクをフォーマットするサンプル カスタム スクリプト拡張機能と、インターネット インフォメーション サービス (IIS) をデプロイする PowerShell DSC スクリプト。</span><span class="sxs-lookup"><span data-stu-id="c78bd-239">A sample custom script extension that formats the two data disks, and a PowerShell DSC script that deploys Internet Information Services (IIS).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="c78bd-240">前提条件</span><span class="sxs-lookup"><span data-stu-id="c78bd-240">Prerequisites</span></span>

1. <span data-ttu-id="c78bd-241">[参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-241">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="c78bd-242">Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-242">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="c78bd-243">CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-243">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="c78bd-244">[Azure の構成要素][azbb] npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-244">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="c78bd-245">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のコマンドを入力して Azure アカウントにログインします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-245">From a command prompt, bash prompt, or PowerShell prompt, enter the following command to log into your Azure account.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="c78bd-246">azbb を使用したソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="c78bd-246">Deploy the solution using azbb</span></span>

<span data-ttu-id="c78bd-247">この参照アーキテクチャをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-247">To deploy this reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="c78bd-248">上の前提条件の手順でダウンロードしたリポジトリの `virtual-machines\single-vm\parameters\windows` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-248">Navigate to the `virtual-machines\single-vm\parameters\windows` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="c78bd-249">`single-vm-v2.json` ファイルを開き、引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="c78bd-249">Open the `single-vm-v2.json` file and enter a username and password between the quotes, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. <span data-ttu-id="c78bd-250">次に示すように、`azbb` を実行して VM のサンプルをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="c78bd-250">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="c78bd-251">デプロイを確認するには、次の Azure CLI コマンドを実行して、VM のパブリック IP アドレスを見つけます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-251">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="c78bd-252">Web ブラウザーでこのアドレスに移動すると、既定の IIS ホーム ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="c78bd-252">If you navigate to this address in a web browser, you should see the default IIS homepage.</span></span>

<span data-ttu-id="c78bd-253">このデプロイのカスタマイズについては、[GitHub リポジトリ][git]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c78bd-253">For information about customizing this deployment, visit our [GitHub repository][git].</span></span>

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure での単一 Windows VM アーキテクチャ"
