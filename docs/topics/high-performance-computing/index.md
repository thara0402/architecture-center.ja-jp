---
title: Azure でのハイ パフォーマンス コンピューティング (HPC)
description: Azure 上で実行される HPC ワークロードを構築するためのガイド
author: adamboeglin
ms.date: 2/4/2019
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a><span data-ttu-id="183c0-103">Azure でのハイ パフォーマンス コンピューティング (HPC)</span><span class="sxs-lookup"><span data-stu-id="183c0-103">High Performance Computing (HPC) on Azure</span></span>

## <a name="introduction-to-hpc"></a><span data-ttu-id="183c0-104">HPC の概要</span><span class="sxs-lookup"><span data-stu-id="183c0-104">Introduction to HPC</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="183c0-105">"ビッグ コンピューティング" とも呼ばれるハイ パフォーマンス コンピューティング (HPC) では、CPU または GPU ベースのコンピューターを大量に使用して、複雑な数学的タスクを解決します。</span><span class="sxs-lookup"><span data-stu-id="183c0-105">High Performance Computing (HPC), also called "Big Compute", uses a large number of CPU or GPU-based computers to solve complex mathematical tasks.</span></span>

<span data-ttu-id="183c0-106">多くの業界では HPC を使用して、最も困難な問題の一部を解決しています。</span><span class="sxs-lookup"><span data-stu-id="183c0-106">Many industries use HPC to solve some of their most difficult problems.</span></span>  <span data-ttu-id="183c0-107">これらには、以下のようなワークロードがあります。</span><span class="sxs-lookup"><span data-stu-id="183c0-107">These include workloads such as:</span></span>

- <span data-ttu-id="183c0-108">Genomics</span><span class="sxs-lookup"><span data-stu-id="183c0-108">Genomics</span></span>
- <span data-ttu-id="183c0-109">石油およびガスのシミュレーション</span><span class="sxs-lookup"><span data-stu-id="183c0-109">Oil and gas simulations</span></span>
- <span data-ttu-id="183c0-110">Finance</span><span class="sxs-lookup"><span data-stu-id="183c0-110">Finance</span></span>
- <span data-ttu-id="183c0-111">半導体の設計</span><span class="sxs-lookup"><span data-stu-id="183c0-111">Semiconductor design</span></span>
- <span data-ttu-id="183c0-112">Engineering</span><span class="sxs-lookup"><span data-stu-id="183c0-112">Engineering</span></span>
- <span data-ttu-id="183c0-113">天気のモデリング</span><span class="sxs-lookup"><span data-stu-id="183c0-113">Weather modeling</span></span>

### <a name="how-is-hpc-different-on-the-cloud"></a><span data-ttu-id="183c0-114">クラウドでの HPC の違い</span><span class="sxs-lookup"><span data-stu-id="183c0-114">How is HPC different on the cloud?</span></span>

<span data-ttu-id="183c0-115">オンプレミスの HPC システムとクラウドのそれとの主な違いの 1 つは、必要に応じてリソースを動的に追加および削除できることです。</span><span class="sxs-lookup"><span data-stu-id="183c0-115">One of the primary differences between an on-premise HPC system and one in the cloud is the ability for resources to dynamically be added and removed as they're needed.</span></span>  <span data-ttu-id="183c0-116">動的スケーリングによって、コンピューター能力がボトルネットになることがなく、お客様はジョブの要件に応じてインフラストラクチャを適切にサイズ調整できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-116">Dynamic scaling removes compute capacity as a bottleneck and instead allow customers to right size their infrastructure for the requirements of their jobs.</span></span>

<span data-ttu-id="183c0-117">次の記事では、この動的スケーリング機能について詳しく説明します。</span><span class="sxs-lookup"><span data-stu-id="183c0-117">The following articles provide more detail about this dynamic scaling capability.</span></span>

- [<span data-ttu-id="183c0-118">ビッグ コンピューティング アーキテクチャ スタイル</span><span class="sxs-lookup"><span data-stu-id="183c0-118">Big Compute Architecture Style</span></span>](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-119">自動スケーリングのベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="183c0-119">Autoscaling best practices</span></span>](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a><span data-ttu-id="183c0-120">実装チェックリスト</span><span class="sxs-lookup"><span data-stu-id="183c0-120">Implementation checklist</span></span>

<span data-ttu-id="183c0-121">独自の HPC ソリューションを Azure に実装しようとしている場合は、以下のトピックをご確認ください。</span><span class="sxs-lookup"><span data-stu-id="183c0-121">As you're looking to implement your own HPC solution on Azure, ensure you're reviewed the following topics:</span></span>

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - <span data-ttu-id="183c0-122">要件に基づいて適切な[アーキテクチャ](#infrastructure)を選択する</span><span class="sxs-lookup"><span data-stu-id="183c0-122">Choose the appropriate [architecture](#infrastructure) based on your requirements</span></span>
> - <span data-ttu-id="183c0-123">ワークロードに適した[コンピューティング](#compute) オプションを把握する</span><span class="sxs-lookup"><span data-stu-id="183c0-123">Know which [compute](#compute) options is right for your workload</span></span>
> - <span data-ttu-id="183c0-124">ニーズを満たす適切な[ストレージ](#storage) ソリューションを特定する</span><span class="sxs-lookup"><span data-stu-id="183c0-124">Identify the right [storage](#storage) solution that meets your needs</span></span>
> - <span data-ttu-id="183c0-125">すべてのリソースを[管理](#management)することになる方法を決定する</span><span class="sxs-lookup"><span data-stu-id="183c0-125">Decide how you're going to [manage](#management) all your resources</span></span>
> - <span data-ttu-id="183c0-126">クラウドに対して[アプリケーション](#hpc-applications)を最適化する</span><span class="sxs-lookup"><span data-stu-id="183c0-126">Optimize your [application](#hpc-applications) for the cloud</span></span>
> - <span data-ttu-id="183c0-127">インフラストラクチャを[セキュリティで保護](#security)する</span><span class="sxs-lookup"><span data-stu-id="183c0-127">[Secure](#security) your Infrastructure</span></span>

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a><span data-ttu-id="183c0-128">インフラストラクチャ</span><span class="sxs-lookup"><span data-stu-id="183c0-128">Infrastructure</span></span>

<span data-ttu-id="183c0-129">HPC システムの構築には、多数のインフラストラクチャ コンポーネントが必要です。</span><span class="sxs-lookup"><span data-stu-id="183c0-129">There are a number of infrastructure components necessary to build an HPC system.</span></span>  <span data-ttu-id="183c0-130">どのような方法で HPC ワークロードを管理することにしても、基礎となるコンポーネントを提供するのは、コンピューティング、ストレージ、およびネットワークです。</span><span class="sxs-lookup"><span data-stu-id="183c0-130">Compute, Storage, and Networking provide the underlying components, no matter how you choose to manage your HPC workloads.</span></span>

### <a name="example-hpc-architectures"></a><span data-ttu-id="183c0-131">HPC アーキテクチャの例</span><span class="sxs-lookup"><span data-stu-id="183c0-131">Example HPC architectures</span></span>

<span data-ttu-id="183c0-132">HPC アーキテクチャを設計して Azure に実装するには、さまざまな方法があります。</span><span class="sxs-lookup"><span data-stu-id="183c0-132">There are a number of different ways to design and implement your HPC architecture on Azure.</span></span>  <span data-ttu-id="183c0-133">HPC アプリケーションは、数千のコンピューティング コアにスケーリングしたり、オンプレミスのクラスターに拡張したり、100% クラウド ネイティブのソリューションとして実行したりできます。</span><span class="sxs-lookup"><span data-stu-id="183c0-133">HPC applications can scale to thousands of compute cores, extend on-premises clusters, or run as a 100% cloud native solution.</span></span>

<span data-ttu-id="183c0-134">次のシナリオでは、HPC ソリューションを構築する一般的な方法をいくつか説明します。</span><span class="sxs-lookup"><span data-stu-id="183c0-134">The following scenarios outline a few of the common ways HPC solutions are built.</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-135">Azure でのコンピューター支援エンジニアリング サービス</span><span class="sxs-lookup"><span data-stu-id="183c0-135">Computer-aided engineering services on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-136">Azure で、コンピューター支援エンジニアリング (CAE) に、サービスとしてのソフトウェア (SaaS) プラットフォームを提供します。</span><span class="sxs-lookup"><span data-stu-id="183c0-136">Provide a software-as-a-service (SaaS) platform for computer-aided engineering (CAE) on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-137">Azure での計算流体力学 (CFD) シミュレーション</span><span class="sxs-lookup"><span data-stu-id="183c0-137">Computational fluid dynamics (CFD) simulations on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-138">Azure で計算流体力学 (CFD) シミュレーションを実行します。</span><span class="sxs-lookup"><span data-stu-id="183c0-138">Execute computational fluid dynamics (CFD) simulations on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-139">Azure での 3D ビデオのレンダリング</span><span class="sxs-lookup"><span data-stu-id="183c0-139">3D video rendering on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-140">Azure Batch サービスを使用して、Azure でネイティブ HPC ワークロードを実行します。</span><span class="sxs-lookup"><span data-stu-id="183c0-140">Run native HPC workloads in Azure using the Azure Batch service</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a><span data-ttu-id="183c0-141">Compute</span><span class="sxs-lookup"><span data-stu-id="183c0-141">Compute</span></span>

<span data-ttu-id="183c0-142">Azure では、CPU の負荷が高いワークロードと GPU の負荷が高いワークロードの両方に対して最適化された幅広いサイズが提供されています。</span><span class="sxs-lookup"><span data-stu-id="183c0-142">Azure offers a range of sizes that are optimized for both CPU & GPU intensive workloads.</span></span>

#### <a name="cpu-based-virtual-machines"></a><span data-ttu-id="183c0-143">CPU ベースの仮想マシン</span><span class="sxs-lookup"><span data-stu-id="183c0-143">CPU-based virtual machines</span></span>

- [<span data-ttu-id="183c0-144">Linux VM</span><span class="sxs-lookup"><span data-stu-id="183c0-144">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="183c0-145">[Windows VM](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) の VM</span><span class="sxs-lookup"><span data-stu-id="183c0-145">[Windows VM's](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) VMs</span></span>
  
#### <a name="gpu-enabled-virtual-machines"></a><span data-ttu-id="183c0-146">GPU 対応仮想マシン</span><span class="sxs-lookup"><span data-stu-id="183c0-146">GPU-enabled virtual machines</span></span>

<span data-ttu-id="183c0-147">N シリーズ VM は、人工知能 (AI) の学習や視覚化などによりコンピューティングやグラフィック使用量が多いアプリケーションのために設計された NVIDIA GPU を採用しています。</span><span class="sxs-lookup"><span data-stu-id="183c0-147">N-series VMs feature NVIDIA GPUs designed for compute-intensive or graphics-intensive applications including artificial intelligence (AI) learning and visualization.</span></span>

- [<span data-ttu-id="183c0-148">Linux VM</span><span class="sxs-lookup"><span data-stu-id="183c0-148">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-149">Windows VM</span><span class="sxs-lookup"><span data-stu-id="183c0-149">Windows VMs</span></span>](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a><span data-ttu-id="183c0-150">Storage</span><span class="sxs-lookup"><span data-stu-id="183c0-150">Storage</span></span>

<span data-ttu-id="183c0-151">バッチ ワークロードや HPC ワークロードが大規模の場合には、従来のクラウド ファイル システムの容量を上回るデータ ストレージが必要になったり、データ アクセスが発生したりします。</span><span class="sxs-lookup"><span data-stu-id="183c0-151">Large-scale Batch and HPC workloads have demands for data storage and access that exceed the capabilities of traditional cloud file systems.</span></span>  <span data-ttu-id="183c0-152">Azure 上の HPC アプリケーションの速度ニーズと容量ニーズ、両方に対応できるソリューションが多数あります</span><span class="sxs-lookup"><span data-stu-id="183c0-152">There are a number of solutions to manage both the speed and capacity needs of HPC applications on Azure</span></span>

- <span data-ttu-id="183c0-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) (データ ストレージに対する処理速度とアクセス性を高め、エッジでハイパフォーマンス コンピューティングを実現)</span><span class="sxs-lookup"><span data-stu-id="183c0-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) for faster, more accessible data storage for high-performance computing at the edge</span></span>
- [<span data-ttu-id="183c0-154">BeeGFS</span><span class="sxs-lookup"><span data-stu-id="183c0-154">BeeGFS</span></span>](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [<span data-ttu-id="183c0-155">ストレージ最適化仮想マシン</span><span class="sxs-lookup"><span data-stu-id="183c0-155">Storage Optimized Virtual Machines</span></span>](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-156">Blob、Table、Queue Storage</span><span class="sxs-lookup"><span data-stu-id="183c0-156">Blob, table, and queue storage</span></span>](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-157">Azure SMB ファイル ストレージ</span><span class="sxs-lookup"><span data-stu-id="183c0-157">Azure SMB File storage</span></span>](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-158">Intel Cloud Edition Lustre</span><span class="sxs-lookup"><span data-stu-id="183c0-158">Intel Cloud Edition Lustre</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

<span data-ttu-id="183c0-159">Azure での Lustre、GlusterFS、BeeGFS の詳しい比較情報については、[Azure での並列ファイル システムに関する電子ブック](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-159">For more information comparing Lustre, GlusterFS, and BeeGFS on Azure, review the [Parallel Files Systems on Azure eBook](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span></span>

### <a name="networking"></a><span data-ttu-id="183c0-160">ネットワーク</span><span class="sxs-lookup"><span data-stu-id="183c0-160">Networking</span></span>

<span data-ttu-id="183c0-161">H16r、H16mr、A8、A9 の VM は、高スループットのバックエンド RDMA ネットワークに接続できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-161">H16r, H16mr, A8, and A9 VMs can connect to a high throughput back-end RDMA network.</span></span> <span data-ttu-id="183c0-162">このネットワークでは、Microsoft MPI または Intel MPI の下で実行される緊密に結合した並列アプリケーションのパフォーマンスを高めることができます。</span><span class="sxs-lookup"><span data-stu-id="183c0-162">This network can improve the performance of tightly coupled parallel applications running under Microsoft MPI or Intel MPI.</span></span>

- [<span data-ttu-id="183c0-163">RDMA 対応のインスタンス</span><span class="sxs-lookup"><span data-stu-id="183c0-163">RDMA Capable Instances</span></span>](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [<span data-ttu-id="183c0-164">Virtual Network</span><span class="sxs-lookup"><span data-stu-id="183c0-164">Virtual Network</span></span>](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-165">ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="183c0-165">ExpressRoute</span></span>](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a><span data-ttu-id="183c0-166">管理</span><span class="sxs-lookup"><span data-stu-id="183c0-166">Management</span></span>

### <a name="do-it-yourself"></a><span data-ttu-id="183c0-167">自作</span><span class="sxs-lookup"><span data-stu-id="183c0-167">Do-it-yourself</span></span>

<span data-ttu-id="183c0-168">Azure で一から HPC システムを構築すると、高度な柔軟性が得られるものの、多くの場合、メンテナンスの手間が非常に大きくなります。</span><span class="sxs-lookup"><span data-stu-id="183c0-168">Building an HPC system from scratch on Azure offers a significant amount of flexibility, but is often very maintenance intensive.</span></span>  

1. <span data-ttu-id="183c0-169">Azure 仮想マシンまたは[仮想マシンのスケール セット](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)に独自のクラスター環境を設定します。</span><span class="sxs-lookup"><span data-stu-id="183c0-169">Set up your own cluster environment in Azure virtual machines or [virtual machine scale sets](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>
2. <span data-ttu-id="183c0-170">Azure Resource Manager テンプレートを使って、業界をリードする[ワークロード マネージャー](#workload-managers)、インフラストラクチャ、および[アプリケーション](#hpc-applications)をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="183c0-170">Use Azure Resource Manager templates to deploy leading [workload managers](#workload-managers), infrastructure, and [applications](#hpc-applications).</span></span>
3. <span data-ttu-id="183c0-171">HPC および GPU の [VM サイズ](#compute)を選択します。これには、MPI ワークロードまたは GPU ワークロードのための特別なハードウェアとネットワーク接続が含まれます。</span><span class="sxs-lookup"><span data-stu-id="183c0-171">Choose HPC and GPU [VM sizes](#compute) that include specialized hardware and network connections for MPI or GPU workloads.</span></span>
4. <span data-ttu-id="183c0-172">I/O が集中するワークロードのための[高性能ストレージ](#storage)を追加します。</span><span class="sxs-lookup"><span data-stu-id="183c0-172">Add [high performance storage](#storage) for I/O-intensive workloads.</span></span>

### <a name="hybrid-and-cloud-bursting"></a><span data-ttu-id="183c0-173">ハイブリッドとクラウド バースティング</span><span class="sxs-lookup"><span data-stu-id="183c0-173">Hybrid and cloud Bursting</span></span>

<span data-ttu-id="183c0-174">Azure に接続したい既存のオンプレミス HPC システムがある場合、作業の開始に役立つリソースが多数あります。</span><span class="sxs-lookup"><span data-stu-id="183c0-174">If you have an existing on-premise HPC system that you'd like to connect to Azure, there are a number of resources to help get you started.</span></span>

<span data-ttu-id="183c0-175">最初に、ドキュメントの[オンプレミス ネットワークを Azure に接続するオプション](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)に関する記事を確認してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-175">First, review the [Options for connecting an on-premises network to Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) article in the documentation.</span></span>  <span data-ttu-id="183c0-176">そこで、以下の接続オプションに関する情報が必要になるでしょう。</span><span class="sxs-lookup"><span data-stu-id="183c0-176">From there, you may want information on these connectivity options:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-177">VPN ゲートウェイを使用した Azure へのオンプレミス ネットワークの接続</span><span class="sxs-lookup"><span data-stu-id="183c0-177">Connect an on-premises network to Azure using a VPN gateway</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-178">この参照アーキテクチャでは、サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure に拡張する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="183c0-178">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-179">ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続</span><span class="sxs-lookup"><span data-stu-id="183c0-179">Connect an on-premises network to Azure using ExpressRoute</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-180">ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。</span><span class="sxs-lookup"><span data-stu-id="183c0-180">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="183c0-181">プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</span><span class="sxs-lookup"><span data-stu-id="183c0-181">The private connection extends your on-premises network into Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-182">VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する</span><span class="sxs-lookup"><span data-stu-id="183c0-182">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-183">Azure 仮想ネットワークと、 VPN ゲートウェイ フェールオーバー付きの ExpressRoute で接続されたオンプレミス ネットワークをカバーする、可用性が高くセキュアなサイト間ネットワーク アーキテクチャを実装します。</span><span class="sxs-lookup"><span data-stu-id="183c0-183">Implement a highly available and secure site-to-site network architecture that spans an Azure virtual network and an on-premises network connected using ExpressRoute with VPN gateway failover.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

<span data-ttu-id="183c0-184">ネットワーク接続が安全に確立されたら、既存の[ワークロード マネージャー](#workload-managers)のバースティング機能と共に、オンデマンドでクラウド コンピューティング リソースの使用を開始できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-184">Once network connectivity is securely established, you can start using cloud compute resources on-demand with the bursting capabilities of your existing [workload manager](#workload-managers).</span></span>

### <a name="marketplace-solutions"></a><span data-ttu-id="183c0-185">Marketplace のソリューション</span><span class="sxs-lookup"><span data-stu-id="183c0-185">Marketplace solutions</span></span>

<span data-ttu-id="183c0-186">[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) では、多数のワークロード マネージャーが提供されています。</span><span class="sxs-lookup"><span data-stu-id="183c0-186">There are a number of workload managers offered in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/).</span></span>

- [<span data-ttu-id="183c0-187">RogueWave CentOS-based HPC</span><span class="sxs-lookup"><span data-stu-id="183c0-187">RogueWave CentOS-based HPC</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [<span data-ttu-id="183c0-188">SUSE Linux Enterprise Server for HPC</span><span class="sxs-lookup"><span data-stu-id="183c0-188">SUSE Linux Enterprise Server for HPC</span></span>](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [<span data-ttu-id="183c0-189">TIBCO Grid Server Engine</span><span class="sxs-lookup"><span data-stu-id="183c0-189">TIBCO Grid Server Engine</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [<span data-ttu-id="183c0-190">Windows および Linux 用 Azure データ サイエンス VM </span><span class="sxs-lookup"><span data-stu-id="183c0-190">Azure Data Science VM for Windows and Linux</span></span>](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-191">D3View</span><span class="sxs-lookup"><span data-stu-id="183c0-191">D3View</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [<span data-ttu-id="183c0-192">UberCloud</span><span class="sxs-lookup"><span data-stu-id="183c0-192">UberCloud</span></span>](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a><span data-ttu-id="183c0-193">Azure Batch</span><span class="sxs-lookup"><span data-stu-id="183c0-193">Azure Batch</span></span>

<span data-ttu-id="183c0-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) は、大規模な並列コンピューティングやハイ パフォーマンス コンピューティング (HPC) のアプリケーションをクラウドで効率的に実行するためのプラットフォーム サービスです。</span><span class="sxs-lookup"><span data-stu-id="183c0-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) is a platform service for running large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="183c0-195">大量のコンピューティングを要する作業を仮想マシンの管理されたプールで実行するようにスケジュールを設定し、ジョブのニーズに合わせてコンピューティング リソースを自動的にスケールできます。</span><span class="sxs-lookup"><span data-stu-id="183c0-195">Azure Batch schedules compute-intensive work to run on a managed pool of virtual machines, and can automatically scale compute resources to meet the needs of your jobs.</span></span>

<span data-ttu-id="183c0-196">SaaS のプロバイダーやデベロッパーは、Batch の SDK とツールを使って、HPC アプリケーションやコンテナー ワークロードを Azure に統合し、Azure にデータをステージングして、ジョブ実行パイプラインを作成できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-196">SaaS providers or developers can use the Batch SDKs and tools to integrate HPC applications or container workloads with Azure, stage data to Azure, and build job execution pipelines.</span></span>

### <a name="azure-cyclecloud"></a><span data-ttu-id="183c0-197">Azure CycleCloud</span><span class="sxs-lookup"><span data-stu-id="183c0-197">Azure CycleCloud</span></span>

<span data-ttu-id="183c0-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) は、Azure で任意のスケジューラ (Slurm、Grid Engine、HPC Pack、HTCondor、LSF、PBS Pro、Symphony など) を使用して HPC ワークロードを管理する最も簡単な方法を提供します</span><span class="sxs-lookup"><span data-stu-id="183c0-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Provides the simplest way to manage HPC workloads using any scheduler (like Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro, or Symphony), on Azure</span></span>

<span data-ttu-id="183c0-199">CycleCloud では以下を行えます。</span><span class="sxs-lookup"><span data-stu-id="183c0-199">CycleCloud allows you to:</span></span>

- <span data-ttu-id="183c0-200">スケジューラ、コンピューティング VM、ストレージ、ネットワーク、キャッシュなど、すべてのクラスターとその他のリソースをデプロイする</span><span class="sxs-lookup"><span data-stu-id="183c0-200">Deploy full clusters and other resources, including scheduler, compute VMs, storage, networking, and cache</span></span>
- <span data-ttu-id="183c0-201">ジョブ、データ、クラウドのワークフローを調整する</span><span class="sxs-lookup"><span data-stu-id="183c0-201">Orchestrate job, data, and cloud workflows</span></span>
- <span data-ttu-id="183c0-202">ジョブを実行できるユーザー、その実行場所、その実行にかかるコストを管理者が完全に制御できるようにする</span><span class="sxs-lookup"><span data-stu-id="183c0-202">Give admins full control over which users can run jobs, as well as where and at what cost</span></span>
- <span data-ttu-id="183c0-203">コスト管理、Active Directory の統合、監視、レポートなど、高度なポリシーおよび管理機能を使用して、クラスターをカスタマイズおよび最適化する</span><span class="sxs-lookup"><span data-stu-id="183c0-203">Customize and optimize clusters through advanced policy and governance features, including cost controls, Active Directory integration, monitoring, and reporting</span></span>
- <span data-ttu-id="183c0-204">変更せずに現在のジョブ スケジューラとアプリケーションを使用する</span><span class="sxs-lookup"><span data-stu-id="183c0-204">Use your current job scheduler and applications without modification</span></span>
- <span data-ttu-id="183c0-205">さまざまな HPC ワークロードや業界に対して、組み込みの自動スケーリングと実績が証明されている参照アーキテクチャを活用する</span><span class="sxs-lookup"><span data-stu-id="183c0-205">Take advantage of built-in autoscaling and battle-tested reference architectures for a wide range of HPC workloads and industries</span></span>

### <a name="workload-managers"></a><span data-ttu-id="183c0-206">ワークロード マネージャー</span><span class="sxs-lookup"><span data-stu-id="183c0-206">Workload managers</span></span>

<span data-ttu-id="183c0-207">Azure のインフラストラクチャで実行できるクラスターおよびワークロード マネージャーの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="183c0-207">The following are examples of cluster and workload managers that can run in Azure infrastructure.</span></span> <span data-ttu-id="183c0-208">Azure VM にスタンドアロンのクラスターを作成するか、オンプレミス クラスターから Azure VM にバーストします。</span><span class="sxs-lookup"><span data-stu-id="183c0-208">Create stand-alone clusters in Azure VMs or burst to Azure VMs from an on-premises cluster.</span></span>

- [<span data-ttu-id="183c0-209">Alces フライト コンピューティング</span><span class="sxs-lookup"><span data-stu-id="183c0-209">Alces Flight Compute</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [<span data-ttu-id="183c0-210">TIBCO DataSynapse GridServer</span><span class="sxs-lookup"><span data-stu-id="183c0-210">TIBCO DataSynapse GridServer</span></span>](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [<span data-ttu-id="183c0-211">Bright Cluster Manager</span><span class="sxs-lookup"><span data-stu-id="183c0-211">Bright Cluster Manager</span></span>](http://www.brightcomputing.com/technology-partners/microsoft)
- [<span data-ttu-id="183c0-212">IBM Spectrum Symphony および Symphony LSF</span><span class="sxs-lookup"><span data-stu-id="183c0-212">IBM Spectrum Symphony and Symphony LSF</span></span>](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [<span data-ttu-id="183c0-213">PBS Pro</span><span class="sxs-lookup"><span data-stu-id="183c0-213">PBS Pro</span></span>](http://pbspro.org)
- [<span data-ttu-id="183c0-214">Altair</span><span class="sxs-lookup"><span data-stu-id="183c0-214">Altair</span></span>](http://www.altair.com/)
- [<span data-ttu-id="183c0-215">Rescale</span><span class="sxs-lookup"><span data-stu-id="183c0-215">Rescale</span></span>](https://www.rescale.com/azure/)
- [<span data-ttu-id="183c0-216">Microsoft HPC Pack</span><span class="sxs-lookup"><span data-stu-id="183c0-216">Microsoft HPC Pack</span></span>](https://technet.microsoft.com/library/mt744885.aspx)
  - [<span data-ttu-id="183c0-217">Windows 用の HPC Pack</span><span class="sxs-lookup"><span data-stu-id="183c0-217">HPC Pack for Windows</span></span>](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [<span data-ttu-id="183c0-218">Linux 用の HPC Pack</span><span class="sxs-lookup"><span data-stu-id="183c0-218">HPC Pack for Linux</span></span>](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a><span data-ttu-id="183c0-219">Containers</span><span class="sxs-lookup"><span data-stu-id="183c0-219">Containers</span></span>

<span data-ttu-id="183c0-220">コンテナーは、一部の HPC ワークロードを管理するためにも使用できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-220">Containers can also be used to manage some HPC workloads.</span></span>  <span data-ttu-id="183c0-221">Azure Kubernetes Service (AKS) などのサービスを使用すると、マネージド Kubernetes クラスターを Azure 内に簡単にデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="183c0-221">Services like the Azure Kubernetes Service (AKS) makes it simple to deploy a managed Kubernetes cluster in Azure.</span></span>

- [<span data-ttu-id="183c0-222">Azure Kubernetes Service (AKS)</span><span class="sxs-lookup"><span data-stu-id="183c0-222">Azure Kubernetes Service (AKS)</span></span>](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-223">コンテナー レジストリ</span><span class="sxs-lookup"><span data-stu-id="183c0-223">Container Registry</span></span>](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a><span data-ttu-id="183c0-224">コスト管理</span><span class="sxs-lookup"><span data-stu-id="183c0-224">Cost management</span></span>

<span data-ttu-id="183c0-225">Azure での HPC コストの管理は、いくつかの異なる方法を通じて行えます。</span><span class="sxs-lookup"><span data-stu-id="183c0-225">Managing your HPC cost on Azure can be done through a few different ways.</span></span>  <span data-ttu-id="183c0-226">[Azure の購入オプション](https://azure.microsoft.com/pricing/purchase-options/)を確認して、ご自分の組織にとって最適な方法を見つけてください。</span><span class="sxs-lookup"><span data-stu-id="183c0-226">Ensure you've reviewed the [Azure purchasing options](https://azure.microsoft.com/pricing/purchase-options/) to find the method that works best for your organization.</span></span>

<span data-ttu-id="183c0-227">[低優先度 VM](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) を使うと、非常に低コストで未使用の容量を利用できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-227">[Low priority VMs](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) allow you to take advantage of our unutilized capacity at a significant cost savings.</span></span>

## <a name="security"></a><span data-ttu-id="183c0-228">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="183c0-228">Security</span></span>

<span data-ttu-id="183c0-229">Azure におけるセキュリティのベスト プラクティスの概要については、[Azure のセキュリティに関するドキュメント](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-229">For an overview of security best practices on Azure, review the [Azure Security Documentation](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>  

<span data-ttu-id="183c0-230">[クラウド バースティング](#hybrid-and-cloud-bursting)に関するセクションにあるネットワーク構成のほかに、ハブ/スポーク構成を実装して、コンピューティング リソースを分離することができます。</span><span class="sxs-lookup"><span data-stu-id="183c0-230">In addition to the network configurations available in the [Cloud Bursting](#hybrid-and-cloud-bursting) section, you may want to implement a hub/spoke configuration to isolate your compute resources:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-231">Azure にハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="183c0-231">Implement a hub-spoke network topology in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-232">ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="183c0-232">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="183c0-233">スポークは、ハブに対して配置される VNet であり、ワークロードを分離するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="183c0-233">The spokes are VNets that peer with the hub, and can be used to isolate workloads.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-234">Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="183c0-234">Implement a hub-spoke network topology with shared services in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-235">この参照アーキテクチャは、ハブスポーク参照アーキテクチャに基づいて作成されており、すべてのスポークで利用できる共有サービスがハブに含まれます。</span><span class="sxs-lookup"><span data-stu-id="183c0-235">This reference architecture builds on the hub-spoke reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a><span data-ttu-id="183c0-236">HPC アプリケーション</span><span class="sxs-lookup"><span data-stu-id="183c0-236">HPC applications</span></span>

<span data-ttu-id="183c0-237">Azure ではカスタム HPC アプリケーションや商用 HPC アプリケーションを実行できます。</span><span class="sxs-lookup"><span data-stu-id="183c0-237">Run custom or commercial HPC applications in Azure.</span></span> <span data-ttu-id="183c0-238">このセクションのさまざまな例で、VM やコンピューティング コアを追加して効率的にスケールできることが確認されています。</span><span class="sxs-lookup"><span data-stu-id="183c0-238">Several examples in this section are benchmarked to scale efficiently with additional VMs or compute cores.</span></span> <span data-ttu-id="183c0-239">すぐにデプロイ可能なソリューションについては、[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-239">Visit the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) for ready-to-deploy solutions.</span></span>

> [!NOTE]
> <span data-ttu-id="183c0-240">クラウドで実行するためのライセンスまたはその他の制限事項については、商用アプリケーションのベンダーに確認してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-240">Check with the vendor of any commercial application for licensing or other restrictions for running in the cloud.</span></span> <span data-ttu-id="183c0-241">すべてのベンダーが従量課金制ライセンスを提供しているとは限りません。</span><span class="sxs-lookup"><span data-stu-id="183c0-241">Not all vendors offer pay-as-you-go licensing.</span></span> <span data-ttu-id="183c0-242">ソリューション用にクラウド内にライセンス サーバーを用意したり、オンプレミスのライセンス サーバーに接続することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="183c0-242">You might need a licensing server in the cloud for your solution, or connect to an on-premises license server.</span></span>

### <a name="engineering-applications"></a><span data-ttu-id="183c0-243">エンジニアリング アプリケーション</span><span class="sxs-lookup"><span data-stu-id="183c0-243">Engineering applications</span></span>

- [<span data-ttu-id="183c0-244">Altair RADIOSS</span><span class="sxs-lookup"><span data-stu-id="183c0-244">Altair RADIOSS</span></span>](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [<span data-ttu-id="183c0-245">ANSYS CFD</span><span class="sxs-lookup"><span data-stu-id="183c0-245">ANSYS CFD</span></span>](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [<span data-ttu-id="183c0-246">MATLAB Distributed Computing Server</span><span class="sxs-lookup"><span data-stu-id="183c0-246">MATLAB Distributed Computing Server</span></span>](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-247">StarCCM+</span><span class="sxs-lookup"><span data-stu-id="183c0-247">StarCCM+</span></span>](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [<span data-ttu-id="183c0-248">OpenFOAM</span><span class="sxs-lookup"><span data-stu-id="183c0-248">OpenFOAM</span></span>](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a><span data-ttu-id="183c0-249">グラフィックとレンダリング</span><span class="sxs-lookup"><span data-stu-id="183c0-249">Graphics and rendering</span></span>

- <span data-ttu-id="183c0-250">Azure Batch での [Autodesk Maya、3ds Max、Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span><span class="sxs-lookup"><span data-stu-id="183c0-250">[Autodesk Maya, 3ds Max, and Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) on Azure Batch</span></span>

### <a name="ai-and-deep-learning"></a><span data-ttu-id="183c0-251">AI とディープ ラーニング</span><span class="sxs-lookup"><span data-stu-id="183c0-251">AI and deep learning</span></span>

- [<span data-ttu-id="183c0-252">Microsoft Cognitive Toolkit</span><span class="sxs-lookup"><span data-stu-id="183c0-252">Microsoft Cognitive Toolkit</span></span>](/cognitive-toolkit/cntk-on-azure)
- [<span data-ttu-id="183c0-253">ディープ ラーニング VM</span><span class="sxs-lookup"><span data-stu-id="183c0-253">Deep Learning VM</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [<span data-ttu-id="183c0-254">ディープ ラーニング用の Batch Shipyard レシピ</span><span class="sxs-lookup"><span data-stu-id="183c0-254">Batch Shipyard recipes for deep learning</span></span>](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a><span data-ttu-id="183c0-255">MPI プロバイダー</span><span class="sxs-lookup"><span data-stu-id="183c0-255">MPI Providers</span></span>

- [<span data-ttu-id="183c0-256">Microsoft MPI</span><span class="sxs-lookup"><span data-stu-id="183c0-256">Microsoft MPI</span></span>](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a><span data-ttu-id="183c0-257">リモート視覚化</span><span class="sxs-lookup"><span data-stu-id="183c0-257">Remote visualization</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="183c0-258">Citrix を使用した Linux 仮想デスクトップ</span><span class="sxs-lookup"><span data-stu-id="183c0-258">Linux virtual desktops with Citrix</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="183c0-259">Azure で Citrix を使用して Linux デスクトップ向けの VDI 環境を構築します。</span><span class="sxs-lookup"><span data-stu-id="183c0-259">Build a VDI environment for Linux Desktops using Citrix on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a><span data-ttu-id="183c0-260">パフォーマンス ベンチマーク</span><span class="sxs-lookup"><span data-stu-id="183c0-260">Performance Benchmarks</span></span>

- [<span data-ttu-id="183c0-261">コンピューティング ベンチマーク</span><span class="sxs-lookup"><span data-stu-id="183c0-261">Compute benchmarks</span></span>](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a><span data-ttu-id="183c0-262">顧客事例</span><span class="sxs-lookup"><span data-stu-id="183c0-262">Customer stories</span></span>

<span data-ttu-id="183c0-263">多くのお客様が、Azure を HPC ワークロードに使用して大きな成功を収めています。</span><span class="sxs-lookup"><span data-stu-id="183c0-263">There are a number of customers who have seen great success by using Azure for their HPC workloads.</span></span>  <span data-ttu-id="183c0-264">以下では、それらのお客様のケース スタディをいくつかご紹介します。</span><span class="sxs-lookup"><span data-stu-id="183c0-264">You can find a few of these customer case studies below:</span></span>

- [<span data-ttu-id="183c0-265">ANEO</span><span class="sxs-lookup"><span data-stu-id="183c0-265">ANEO</span></span>](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [<span data-ttu-id="183c0-266">AXA Global P&C</span><span class="sxs-lookup"><span data-stu-id="183c0-266">AXA Global P&C</span></span>](https://customers.microsoft.com/story/axa-global-p-and-c)
- [<span data-ttu-id="183c0-267">Axioma</span><span class="sxs-lookup"><span data-stu-id="183c0-267">Axioma</span></span>](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [<span data-ttu-id="183c0-268">d3View</span><span class="sxs-lookup"><span data-stu-id="183c0-268">d3View</span></span>](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [<span data-ttu-id="183c0-269">EFS</span><span class="sxs-lookup"><span data-stu-id="183c0-269">EFS</span></span>](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [<span data-ttu-id="183c0-270">Hymans Robertson</span><span class="sxs-lookup"><span data-stu-id="183c0-270">Hymans Robertson</span></span>](https://customers.microsoft.com/story/hymans-robertson)
- [<span data-ttu-id="183c0-271">MetLife</span><span class="sxs-lookup"><span data-stu-id="183c0-271">MetLife</span></span>](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [<span data-ttu-id="183c0-272">Microsoft Research</span><span class="sxs-lookup"><span data-stu-id="183c0-272">Microsoft Research</span></span>](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [<span data-ttu-id="183c0-273">Milliman</span><span class="sxs-lookup"><span data-stu-id="183c0-273">Milliman</span></span>](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [<span data-ttu-id="183c0-274">Mitsubishi UFJ Securities International</span><span class="sxs-lookup"><span data-stu-id="183c0-274">Mitsubishi UFJ Securities International</span></span>](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [<span data-ttu-id="183c0-275">NeuroInitiative</span><span class="sxs-lookup"><span data-stu-id="183c0-275">NeuroInitiative</span></span>](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [<span data-ttu-id="183c0-276">Schlumberger</span><span class="sxs-lookup"><span data-stu-id="183c0-276">Schlumberger</span></span>](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [<span data-ttu-id="183c0-277">Towers Watson</span><span class="sxs-lookup"><span data-stu-id="183c0-277">Towers Watson</span></span>](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a><span data-ttu-id="183c0-278">その他の重要な情報</span><span class="sxs-lookup"><span data-stu-id="183c0-278">Other important information</span></span>

- <span data-ttu-id="183c0-279">大規模なワークロードの実行を試行する前には、[vCPU クォータ](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)を引き上げておくようにしてください。</span><span class="sxs-lookup"><span data-stu-id="183c0-279">Ensure your [vCPU quota](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) has been increased before attempting to run large-scale workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="183c0-280">次の手順</span><span class="sxs-lookup"><span data-stu-id="183c0-280">Next steps</span></span>

<span data-ttu-id="183c0-281">最新のお知らせについては、以下を参照してください。</span><span class="sxs-lookup"><span data-stu-id="183c0-281">For the latest announcements, see:</span></span>

- [<span data-ttu-id="183c0-282">Microsoft HPC と Batch のチーム ブログ</span><span class="sxs-lookup"><span data-stu-id="183c0-282">Microsoft HPC and Batch team blog</span></span>](http://blogs.technet.com/b/windowshpc/)
- <span data-ttu-id="183c0-283">[Azure のブログ](https://azure.microsoft.com/blog/tag/hpc/)にアクセスしてください。</span><span class="sxs-lookup"><span data-stu-id="183c0-283">Visit the [Azure blog](https://azure.microsoft.com/blog/tag/hpc/).</span></span>

### <a name="microsoft-batch-examples"></a><span data-ttu-id="183c0-284">Microsoft Batch の例</span><span class="sxs-lookup"><span data-stu-id="183c0-284">Microsoft Batch Examples</span></span>

<span data-ttu-id="183c0-285">以下のチュートリアルでは、Microsoft Batch でのアプリケーションの実行について詳しく説明します</span><span class="sxs-lookup"><span data-stu-id="183c0-285">These tutorials will provide you with details on running applications on Microsoft Batch</span></span>

- [<span data-ttu-id="183c0-286">Batch を使った初めての開発</span><span class="sxs-lookup"><span data-stu-id="183c0-286">Get started developing with Batch</span></span>](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-287">Azure Batch のコード サンプルを使用する</span><span class="sxs-lookup"><span data-stu-id="183c0-287">Use Azure Batch code samples</span></span>](https://github.com/Azure/azure-batch-samples)
- [<span data-ttu-id="183c0-288">Batch で優先順位の低い VM を使用する</span><span class="sxs-lookup"><span data-stu-id="183c0-288">Use low-priority VMs with Batch</span></span>](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="183c0-289">Batch Shipyard を使ってコンテナー化した HPC ワークロードを実行する</span><span class="sxs-lookup"><span data-stu-id="183c0-289">Run containerized HPC workloads with Batch Shipyard</span></span>](https://github.com/Azure/batch-shipyard)
- [<span data-ttu-id="183c0-290">Batch で並列 R ワークロードを実行する</span><span class="sxs-lookup"><span data-stu-id="183c0-290">Run parallel R workloads on Batch</span></span>](https://github.com/Azure/doAzureParallel)
- [<span data-ttu-id="183c0-291">Batch でオンデマンド Spark ジョブを実行する</span><span class="sxs-lookup"><span data-stu-id="183c0-291">Run on-demand Spark jobs on Batch</span></span>](https://github.com/Azure/aztk)
- [<span data-ttu-id="183c0-292">Batch プールでコンピューティング集中型 VM を使用する</span><span class="sxs-lookup"><span data-stu-id="183c0-292">Use compute-intensive VMs in Batch pools</span></span>](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)