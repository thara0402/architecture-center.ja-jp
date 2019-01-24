---
title: ビッグ コンピューティング アーキテクチャ スタイル
titleSuffix: Azure Application Architecture Guide
description: Azure のビッグ コンピューティング アーキテクチャのメリット、課題、ベスト プラクティスを説明します。
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, HPC
ms.openlocfilehash: 56bd2ce010b56880e769ada4c6397391a73bbd1e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485759"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="9bcf3-103">ビッグ コンピューティング アーキテクチャ スタイル</span><span class="sxs-lookup"><span data-stu-id="9bcf3-103">Big compute architecture style</span></span>

<span data-ttu-id="9bcf3-104">*ビッグ コンピューティング*という用語は、数百、数千という大量の数のコアを必要とする大規模なワークロードを指します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="9bcf3-105">シナリオには、イメージのレンダリング、流体力学、財務リスクのモデリング、石油探査、薬物設計、および工学応力分析などが含まれます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![ビッグ コンピューティング アーキテクチャ スタイルの論理図](./images/big-compute-logical.png)

<span data-ttu-id="9bcf3-107">ビッグ コンピューティング アプリケーションの一般的な特性を次に示します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-107">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="9bcf3-108">作業は、多くのコアで同時に実行できる個別のタスクに分割できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-108">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="9bcf3-109">各タスクは有限です。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-109">Each task is finite.</span></span> <span data-ttu-id="9bcf3-110">タスクはいくつかの入力を受け取り、いくつかの処理を行って、出力が生成されます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-110">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="9bcf3-111">アプリケーション全体は、一定時間 (数分から数日) 実行されます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-111">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="9bcf3-112">一般的なパターンは、バーストにより大量のコアをプロビジョニングし、アプリケーションが完了するとコア数はゼロまで下降します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-112">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span>
- <span data-ttu-id="9bcf3-113">アプリケーションを常時実行し続ける必要はありません。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-113">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="9bcf3-114">ただし、システムでは、ノードの障害またはアプリケーションのクラッシュを処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-114">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="9bcf3-115">一部のアプリケーションでは、タスクは独立しており、並列して実行できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-115">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="9bcf3-116">または、タスクが緊密に結合されているため、相互に交信または中間結果を交換する必要がある場合もあります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-116">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="9bcf3-117">この場合、InfiniBand やリモート ダイレクト メモリ アクセス (RDMA) などの高速ネットワーク テクノロジを使用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-117">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span>
- <span data-ttu-id="9bcf3-118">ワークロードに応じてコンピューティング集中型の VM サイズ (H16r、H16mr、および A9) を使用することができます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-118">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="9bcf3-119">このアーキテクチャを使用する条件</span><span class="sxs-lookup"><span data-stu-id="9bcf3-119">When to use this architecture</span></span>

- <span data-ttu-id="9bcf3-120">シミュレーション や計算などの大量の計算操作。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-120">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="9bcf3-121">計算負荷が高く、複数のコンピューター (10 ～ 1000 台) の CPU に分割する必要があるシミュレーション。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-121">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="9bcf3-122">1 台のコンピューター上の大量のメモリを必要とするシミュレーションでは、複数のコンピューターに分割する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-122">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="9bcf3-123">1 台のコンピューターでは計算に時間がかかりすぎる長い計算。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-123">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="9bcf3-124">モンテカルロ シミュレーションなどの、数百または数千回実行する必要がある小規模な計算。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-124">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="9bcf3-125">メリット</span><span class="sxs-lookup"><span data-stu-id="9bcf3-125">Benefits</span></span>

- <span data-ttu-id="9bcf3-126">"[驚異的並列][embarrassingly-parallel]" 処理による高いパフォーマンス。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-126">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="9bcf3-127">大きな問題を高速で解決するために何百、何千ものコンピューター コアを使用できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-127">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="9bcf3-128">専用の高速 InfiniBand ネットワークを使用した、特殊な高性能ハードウェアの使用。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-128">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="9bcf3-129">作業中、必要に応じて VM をプロビジョニングし、削除できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-129">You can provision VMs as needed to do work, and then tear them down.</span></span>

## <a name="challenges"></a><span data-ttu-id="9bcf3-130">課題</span><span class="sxs-lookup"><span data-stu-id="9bcf3-130">Challenges</span></span>

- <span data-ttu-id="9bcf3-131">VM インフラストラクチャの管理。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-131">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="9bcf3-132">大量の計算の管理</span><span class="sxs-lookup"><span data-stu-id="9bcf3-132">Managing the volume of number crunching</span></span>
- <span data-ttu-id="9bcf3-133">適切なタイミングで数千のコアをプロビジョニングする。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-133">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="9bcf3-134">緊密に結合されたタスクにコアを追加すると逆効果になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-134">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="9bcf3-135">実験を通して最適なコア数を特定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-135">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="9bcf3-136">Azure Batch を使用したビッグ コンピューティング</span><span class="sxs-lookup"><span data-stu-id="9bcf3-136">Big compute using Azure Batch</span></span>

<span data-ttu-id="9bcf3-137">[Azure Batc][batch] は、大規模な高パフォーマンス コンピューティング (HPC) のアプリケーションを実行するためのマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-137">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="9bcf3-138">Azure Batch を使用して、VM プールを構成し、アプリケーションとデータ ファイルをアップロードします。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-138">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="9bcf3-139">バッチ サービスにより VM がプロビジョニングされ、VM にタスクが割り当てられ、タスクが実行され、進行状況が監視されます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-139">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="9bcf3-140">バッチは、ワークロードに応じて VM を自動的にスケール アウトできます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-140">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="9bcf3-141">また、バッチは、ジョブのスケジューリングも提供します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-141">Batch also provides job scheduling.</span></span>

![Azure Batch を使用したビッグ コンピューティングの図](./images/big-compute-batch.png)

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="9bcf3-143">Virtual Machines で実行されるビッグ コンピューティング</span><span class="sxs-lookup"><span data-stu-id="9bcf3-143">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="9bcf3-144">[Microsoft HPC Pack][hpc-pack] を使用して VM のクラスターを管理して、HPC ジョブをスケジュールおよび監視できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-144">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="9bcf3-145">この方法では、ユーザーが VM およびネットワーク インフラストラクチャのプロビジョニングや管理を行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-145">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="9bcf3-146">既存の HPC ワークロードの一部またはすべてを Azure に移動する場合は、このアプローチを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-146">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="9bcf3-147">HPC クラスター全体を Azure に移動するか、HPC クラスターはオンプレミスにしたまま、バースト容量のために Azure を使用できます。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-147">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="9bcf3-148">詳細については、[大規模コンピューティング ワークロードのための Batch および HPC ソリューション][batch-hpc-solutions]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-148">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="9bcf3-149">Azure にデプロイされた HPC Pack</span><span class="sxs-lookup"><span data-stu-id="9bcf3-149">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="9bcf3-150">このシナリオでは、HPC クラスターすべてを Azure 内で作成します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-150">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![Azure にデプロイされた HPC Pack の図](./images/big-compute-iaas.png)

<span data-ttu-id="9bcf3-152">ヘッド ノードは、管理およびジョブ スケジューリング サービスをクラスターに提供します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-152">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="9bcf3-153">緊密に結合されたタスクの場合は、非常に高い帯域幅、低待機時間の VM 間通信を提供する RDMA ネットワークを使用します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-153">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="9bcf3-154">詳細については、「[Azure に HPC Pack 2016 クラスターをデプロイする][deploy-hpc-azure]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-154">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="9bcf3-155">Azure への HPC クラスターのバースト</span><span class="sxs-lookup"><span data-stu-id="9bcf3-155">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="9bcf3-156">このシナリオでは、組織は、HPC Pack をオンプレミスで実行しており、バースト容量のために Azure VM を使用します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-156">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="9bcf3-157">クラスターのヘッド ノードは、オンプレミスに設置されています。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-157">The cluster head node is on-premises.</span></span> <span data-ttu-id="9bcf3-158">ExpressRoute または VPN Gateway は、オンプレミス ネットワークを Azure VNet に接続します。</span><span class="sxs-lookup"><span data-stu-id="9bcf3-158">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![ハイブリッド ビッグ コンピューティング クラスターの図](./images/big-compute-hybrid.png)

<!-- links -->

[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029
