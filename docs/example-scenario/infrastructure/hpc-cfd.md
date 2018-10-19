---
title: Azure での計算流体力学 (CFD) シミュレーションの実行
description: Azure で計算流体力学 (CFD) シミュレーションを実行します。
author: mikewarr
ms.date: 09/20/2018
ms.openlocfilehash: 5734e6fe707e3beb5e23f2ad2b4344ba289803bb
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818579"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a><span data-ttu-id="24433-103">Azure での計算流体力学 (CFD) シミュレーションの実行</span><span class="sxs-lookup"><span data-stu-id="24433-103">Running computational fluid dynamics (CFD) simulations on Azure</span></span>

<span data-ttu-id="24433-104">計算流体力学 (CFD) シミュレーションでは、膨大な計算時間と特殊なハードウェアが必要です。</span><span class="sxs-lookup"><span data-stu-id="24433-104">Computational Fluid Dynamics (CFD) simulations require significant compute time along with specialized hardware.</span></span> <span data-ttu-id="24433-105">クラスターの使用量が増えると、シミュレーションの時間と全体的なグリッドの使用が増え、予備の容量と長いキュー時間に関する問題が発生します。</span><span class="sxs-lookup"><span data-stu-id="24433-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="24433-106">物理ハードウェアの追加は高価であり、ビジネスで発生する使用量の山と谷に一致しない場合があります。</span><span class="sxs-lookup"><span data-stu-id="24433-106">Adding physical hardware can be expensive, and may not align to the usage peaks and valleys that a business goes through.</span></span> <span data-ttu-id="24433-107">Azure を利用することで、これらの課題の多くを設備投資なしで克服できます。</span><span class="sxs-lookup"><span data-stu-id="24433-107">By taking advantage of Azure, many of these challenges can be overcome with no capital expenditure.</span></span>

<span data-ttu-id="24433-108">Azure では、GPU および CPU 両方の仮想マシンで CFD ジョブを実行するために必要なハードウェアが提供されます。</span><span class="sxs-lookup"><span data-stu-id="24433-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="24433-109">RDMA (リモート ダイレクト メモリ アクセス) が有効な VM サイズには、低遅延 MPI (Message Passing Interface) 通信に対応する FDR InfiniBand ベースのネットワークがあります。</span><span class="sxs-lookup"><span data-stu-id="24433-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand-based networking which allows for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="24433-110">エンタープライズ規模のクラスター化されたファイル システムを提供する Avere vFXT と組み合わせると、Azure での読み取り操作に最大限のスループットを確保できます。</span><span class="sxs-lookup"><span data-stu-id="24433-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="24433-111">HPC クラスターの作成、管理、最適化を簡略化するには、Azure CycleCloud を使用して、クラスターをプロビジョニングし、ハイブリッド シナリオとクラウド シナリオ両方でのデータを調整できます。</span><span class="sxs-lookup"><span data-stu-id="24433-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="24433-112">保留中のジョブを監視することにより、CycleCloud は任意のワークロード スケジューラーに接続されたオンデマンド コンピューティングを自動的に起動し、支払いは使用した分だけで済みます。</span><span class="sxs-lookup"><span data-stu-id="24433-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="24433-113">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="24433-113">Relevant use cases</span></span>

<span data-ttu-id="24433-114">以下の業界で CFD アプリケーションを使用できる場合はこのシナリオを検討します。</span><span class="sxs-lookup"><span data-stu-id="24433-114">Consider this scenario for these industries where CFD applications could be used:</span></span>

* <span data-ttu-id="24433-115">航空</span><span class="sxs-lookup"><span data-stu-id="24433-115">Aeronautics</span></span>
* <span data-ttu-id="24433-116">自動車</span><span class="sxs-lookup"><span data-stu-id="24433-116">Automotive</span></span>
* <span data-ttu-id="24433-117">建築 HVAC</span><span class="sxs-lookup"><span data-stu-id="24433-117">Building HVAC</span></span>
* <span data-ttu-id="24433-118">石油、ガス</span><span class="sxs-lookup"><span data-stu-id="24433-118">Oil and gas</span></span>
* <span data-ttu-id="24433-119">ライフ サイエンス</span><span class="sxs-lookup"><span data-stu-id="24433-119">Life sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="24433-120">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="24433-120">Architecture</span></span>

![アーキテクチャ ダイアグラム][architecture]

<span data-ttu-id="24433-122">この図は、Azure でオンデマンド ノードのジョブ監視を提供する一般的なハイブリッド設計の概要を示したものです。</span><span class="sxs-lookup"><span data-stu-id="24433-122">This diagram shows a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="24433-123">Azure CycleCloud サーバーに接続してクラスターを構成します。</span><span class="sxs-lookup"><span data-stu-id="24433-123">Connect to the Azure CycleCloud server to configure the cluster.</span></span>
2. <span data-ttu-id="24433-124">MPI 用の RDMA 対応マシンを使用して、クラスターのヘッド ノードを構成して作成します。</span><span class="sxs-lookup"><span data-stu-id="24433-124">Configure and create the cluster head node, using RDMA enabled machines for MPI.</span></span>
3. <span data-ttu-id="24433-125">オンプレミスのヘッド ノードを追加して構成します。</span><span class="sxs-lookup"><span data-stu-id="24433-125">Add and configure the on-premises head node.</span></span>
4. <span data-ttu-id="24433-126">十分なリソースがない場合、Azure CycleCloud は Azure のコンピューティング リソースをスケールアップ (またはスケールダウン) します。</span><span class="sxs-lookup"><span data-stu-id="24433-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="24433-127">過剰な割り当てを防ぐため、あらかじめ設定された制限を定義できます。</span><span class="sxs-lookup"><span data-stu-id="24433-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="24433-128">実行ノードに割り当てられたタスク。</span><span class="sxs-lookup"><span data-stu-id="24433-128">Tasks allocated to the execute nodes.</span></span>
6. <span data-ttu-id="24433-129">オンプレミスの NFS サーバーから Azure にキャッシュされたデータ。</span><span class="sxs-lookup"><span data-stu-id="24433-129">Data cached in Azure from on-premises NFS server.</span></span>
7. <span data-ttu-id="24433-130">Azure キャッシュ用の Avere vFXT から読み取られたデータ。</span><span class="sxs-lookup"><span data-stu-id="24433-130">Data read in from the Avere vFXT for Azure cache.</span></span>
8. <span data-ttu-id="24433-131">Azure CycleCloud サーバーに中継されたジョブとタスクの情報。</span><span class="sxs-lookup"><span data-stu-id="24433-131">Job and task information relayed to the Azure CycleCloud server.</span></span>

### <a name="components"></a><span data-ttu-id="24433-132">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="24433-132">Components</span></span>

* <span data-ttu-id="24433-133">[Azure CycleCloud][cyclecloud] は、Azure で HPC とビッグ コンピューティング クラスターを作成、管理、運用、最適化するためのツールです。</span><span class="sxs-lookup"><span data-stu-id="24433-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC and Big Compute clusters in Azure.</span></span>
* <span data-ttu-id="24433-134">[Azure 上の Avere vFXT][avere] は、クラウド向けに構築されたエンタープライズ規模のクラスター化されたファイル システムを提供するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="24433-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud.</span></span>
* <span data-ttu-id="24433-135">[Azure Virtual Machines (VM)][vms] は、コンピューティング インスタンスの静的なセットの作成に使用されます。</span><span class="sxs-lookup"><span data-stu-id="24433-135">[Azure Virtual Machines (VMs)][vms] are used to create a static set of compute instances.</span></span>
* <span data-ttu-id="24433-136">[Virtual Machine Scale Sets (仮想マシン スケール セット)][vmss] は、Azure CycleCloud によってスケールアップまたはスケールダウンされる同じ VM のグループを提供します。</span><span class="sxs-lookup"><span data-stu-id="24433-136">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud.</span></span>
* <span data-ttu-id="24433-137">[Azure Storage アカウント](/azure/storage/common/storage-introduction)は、同期とデータ保持に使用されます。</span><span class="sxs-lookup"><span data-stu-id="24433-137">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
* <span data-ttu-id="24433-138">[Virtual Network](/azure/virtual-network/virtual-networks-overview) では、Azure Virtual Machine (VM) などのさまざまな種類の Azure リソースが、他の Azure リソース、インターネット、およびオンプレミスのネットワークと安全に通信することができます。</span><span class="sxs-lookup"><span data-stu-id="24433-138">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) enable many types of Azure resources, such as Azure Virtual Machines (VMs), to securely communicate with each other, the internet, and on-premises networks.</span></span>

### <a name="alternatives"></a><span data-ttu-id="24433-139">代替手段</span><span class="sxs-lookup"><span data-stu-id="24433-139">Alternatives</span></span>

<span data-ttu-id="24433-140">お客様は、Azure CycleCloud を使用してグリッド全体を Azure 内に作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="24433-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span> <span data-ttu-id="24433-141">このセットアップでは、Azure CycleCloud サーバーは Azure サブスクリプション内で実行されます。</span><span class="sxs-lookup"><span data-stu-id="24433-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="24433-142">ワークロード スケジューラーの管理を必要としない最新のアプリケーション アプローチでは、[Azure Batch][batch] が役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="24433-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="24433-143">Azure Batch では、大規模な並列コンピューティングやハイパフォーマンス コンピューティング (HPC) のアプリケーションをクラウドで効率的に実行することができます。</span><span class="sxs-lookup"><span data-stu-id="24433-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="24433-144">Azure Batch を使用すると、インフラストラクチャを手動で構成または管理することなく、アプリケーションを並列で実行したり大規模に実行したりするように Azure コンピューティング リソースを定義できます。</span><span class="sxs-lookup"><span data-stu-id="24433-144">Azure Batch allows you to define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="24433-145">Azure Batch はコンピューティング集中型のタスクのスケジュールを設定し、要件に基づいてコンピューティング リソースを動的に追加または削除します。</span><span class="sxs-lookup"><span data-stu-id="24433-145">Azure Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements.</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="24433-146">スケーラビリティ、セキュリティ</span><span class="sxs-lookup"><span data-stu-id="24433-146">Scalability, and Security</span></span>

<span data-ttu-id="24433-147">Azure CycleCloud 上の実行ノードのスケーリングは、手動で、または自動スケールを使用して、実現できます。</span><span class="sxs-lookup"><span data-stu-id="24433-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or using autoscaling.</span></span> <span data-ttu-id="24433-148">詳しくは、[CycleCloud の自動スケール][cycle-scale]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24433-148">For more information, see [CycleCloud Autoscaling][cycle-scale].</span></span>

<span data-ttu-id="24433-149">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24433-149">For general guidance on designing secure solutions, see the [Azure security documentation][security].</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="24433-150">このシナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="24433-150">Deploy this scenario</span></span>

<span data-ttu-id="24433-151">Azure にデプロイする前に、いくつかの前提条件が必要です。</span><span class="sxs-lookup"><span data-stu-id="24433-151">Before deploying in Azure, some pre-requisites are required.</span></span> <span data-ttu-id="24433-152">Resource Manager テンプレートをデプロイする前に、次の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="24433-152">Follow these steps before deploying the Resource Manager template:</span></span>
1. <span data-ttu-id="24433-153">アプリ ID、表示名、名前、パスワード、テナントを取得するための[サービス プリンシパル][cycle-svcprin]を作成します。</span><span class="sxs-lookup"><span data-stu-id="24433-153">Create a [service principal][cycle-svcprin] for retrieving the appId, displayName, name, password, and tenant.</span></span>
2. <span data-ttu-id="24433-154">CycleCloud サーバーに安全にサインインするための [SSH キー ペア][cycle-ssh]を生成します。</span><span class="sxs-lookup"><span data-stu-id="24433-154">Generate an [SSH key pair][cycle-ssh] to sign in securely to the CycleCloud server.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. <span data-ttu-id="24433-155">[CycleCloud サーバーにログイン][cycle-login]して、新しいクラスターを構成および作成します。</span><span class="sxs-lookup"><span data-stu-id="24433-155">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster.</span></span>
4. <span data-ttu-id="24433-156">[クラスターを作成][cycle-create]します。</span><span class="sxs-lookup"><span data-stu-id="24433-156">[Create a cluster][cycle-create].</span></span>

<span data-ttu-id="24433-157">Avere Cache は、アプリケーション ジョブ データの読み取りスループットを大幅に高めることができるオプションのソリューションです。</span><span class="sxs-lookup"><span data-stu-id="24433-157">The Avere Cache is an optional solution that can drastically increase read throughput for the application job data.</span></span> <span data-ttu-id="24433-158">Avere vFXT for Azure は、オンプレミスまたは Azure Blob Storage に格納されているデータを利用しながら、これらのエンタープライズ HPC アプリケーションをクラウドで実行する問題を解決します。</span><span class="sxs-lookup"><span data-stu-id="24433-158">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="24433-159">オンプレミスのストレージとクラウド コンピューティングの両方を使用するハイブリッド インフラストラクチャを計画している場合、HPC アプリケーションは NAS デバイスに格納されているデータを使用して Azure に "介入" し、必要に応じて仮想 CPU を起動できます。</span><span class="sxs-lookup"><span data-stu-id="24433-159">For organizations that are planning for a hybrid infrastructure with both on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up virtual CPUs as needed.</span></span> <span data-ttu-id="24433-160">データ セットがクラウドに完全に移動されることはありません。</span><span class="sxs-lookup"><span data-stu-id="24433-160">The data set is never moved completely into the cloud.</span></span> <span data-ttu-id="24433-161">要求されたバイトは、処理の間に Avere クラスターを使用して一時的にキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="24433-161">The requested bytes are temporarily cached using an Avere cluster during processing.</span></span>

<span data-ttu-id="24433-162">Avere vFXT のインストールのセットアップと構成については、[Avere のセットアップと構成ガイド][avere]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24433-162">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration guide][avere].</span></span>

## <a name="pricing"></a><span data-ttu-id="24433-163">価格</span><span class="sxs-lookup"><span data-stu-id="24433-163">Pricing</span></span>

<span data-ttu-id="24433-164">CycleCloud サーバーを使用して HPC の実装を実行するコストは、さまざまな要因によって異なります。</span><span class="sxs-lookup"><span data-stu-id="24433-164">The cost of running an HPC implementation using CycleCloud server will vary depending on a number of factors.</span></span> <span data-ttu-id="24433-165">たとえば、CycleCloud は使用されたコンピューティング時間量によって課金され、Master サーバーと CycleCloud サーバーは通常常に割り当てられて実行しています。</span><span class="sxs-lookup"><span data-stu-id="24433-165">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="24433-166">Execute ノードを実行するコストは、稼働している時間と使用されたサイズによって決まります。</span><span class="sxs-lookup"><span data-stu-id="24433-166">The cost of running the Execute nodes will depend on how long these are up and running as well as what size is used.</span></span> <span data-ttu-id="24433-167">ストレージとネットワークに対する通常の Azure の料金も適用されます。</span><span class="sxs-lookup"><span data-stu-id="24433-167">The normal Azure charges for storage and networking also apply.</span></span>

<span data-ttu-id="24433-168">このシナリオでは CFD アプリケーションを Azure で実行する方法を示すので、マシンには RDMA 機能が必要であり、これは特定の VM サイズでのみ使用できます。</span><span class="sxs-lookup"><span data-stu-id="24433-168">This scenario shows how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="24433-169">次に示すのは、1 か月間継続して 1 日 8 時間割り当てられた、データ送信が 1 TB のスケール セットにかかるコストの例です。</span><span class="sxs-lookup"><span data-stu-id="24433-169">The following are examples of costs that could be incurred for a scale set that is allocated continuously for eight hours per day for one month, with data egress of 1 TB.</span></span> <span data-ttu-id="24433-170">Azure CycleCloud サーバーと、Azure インストール用 Avere vFXT の価格も含まれます。</span><span class="sxs-lookup"><span data-stu-id="24433-170">It also includes pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

* <span data-ttu-id="24433-171">リージョン: 北ヨーロッパ</span><span class="sxs-lookup"><span data-stu-id="24433-171">Region: North Europe</span></span>
* <span data-ttu-id="24433-172">Azure CycleCloud サーバー: 1 x Standard D3 (4 x CPU、14 GB メモリ、Standard HDD 32 GB)</span><span class="sxs-lookup"><span data-stu-id="24433-172">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="24433-173">Azure CycleCloud Master サーバー: 1 x Standard D12 v (4 x CPU、28 GB メモリ、Standard HDD 32 GB)</span><span class="sxs-lookup"><span data-stu-id="24433-173">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="24433-174">Azure CycleCloud ノード アレイ: 10 x Standard H16r (16 x CPU、112 GB メモリ)</span><span class="sxs-lookup"><span data-stu-id="24433-174">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
* <span data-ttu-id="24433-175">Azure クラスター上の Avere vFXT: 3 x D16s v3 (200 GB OS、Premium SSD 1-TB データ ディスク)</span><span class="sxs-lookup"><span data-stu-id="24433-175">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
* <span data-ttu-id="24433-176">データ送信: 1 TB</span><span class="sxs-lookup"><span data-stu-id="24433-176">Data Egress: 1 TB</span></span>

<span data-ttu-id="24433-177">上に示したハードウェアについてはこちらの[価格見積もり][pricing]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="24433-177">Review this [price estimate][pricing] for the hardware listed above.</span></span>

## <a name="next-steps"></a><span data-ttu-id="24433-178">次の手順</span><span class="sxs-lookup"><span data-stu-id="24433-178">Next Steps</span></span>

<span data-ttu-id="24433-179">サンプルを展開した後、[Azure CycleCloud][cyclecloud] の詳細を学習してください。</span><span class="sxs-lookup"><span data-stu-id="24433-179">Once you've deployed the sample, learn more about [Azure CycleCloud][cyclecloud].</span></span>

## <a name="related-resources"></a><span data-ttu-id="24433-180">関連リソース</span><span class="sxs-lookup"><span data-stu-id="24433-180">Related resources</span></span>

* <span data-ttu-id="24433-181">[RDMA 対応のマシン インスタンス][rdma]</span><span class="sxs-lookup"><span data-stu-id="24433-181">[RDMA Capable Machine Instances][rdma]</span></span>
* <span data-ttu-id="24433-182">[RDMA インスタンス VM のカスタマイズ][rdma-custom]</span><span class="sxs-lookup"><span data-stu-id="24433-182">[Customizing an RDMA Instance VM][rdma-custom]</span></span>

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
