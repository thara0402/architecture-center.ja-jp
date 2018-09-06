---
title: ビッグ コンピューティング アーキテクチャ スタイル
description: Azure のビッグ コンピューティング アーキテクチャのメリット、課題、ベスト プラクティスを説明します。　
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: aca2221faf1fbf47de2fd81c8909dfe8aef46bea
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326175"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="1833a-103">ビッグ コンピューティング アーキテクチャ スタイル</span><span class="sxs-lookup"><span data-stu-id="1833a-103">Big compute architecture style</span></span>

<span data-ttu-id="1833a-104">*ビッグ コンピューティング*という用語は、数百、数千という大量の数のコアを必要とする大規模なワークロードを指します。</span><span class="sxs-lookup"><span data-stu-id="1833a-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="1833a-105">シナリオには、イメージのレンダリング、流体力学、財務リスクのモデリング、石油探査、薬物設計、および工学応力分析などが含まれます。</span><span class="sxs-lookup"><span data-stu-id="1833a-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="1833a-106">ビッグ コンピューティング アプリケーションの一般的な特性を次に示します。</span><span class="sxs-lookup"><span data-stu-id="1833a-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="1833a-107">作業は、多くのコアで同時に実行できる個別のタスクに分割できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="1833a-108">各タスクは有限です。</span><span class="sxs-lookup"><span data-stu-id="1833a-108">Each task is finite.</span></span> <span data-ttu-id="1833a-109">タスクはいくつかの入力を受け取り、いくつかの処理を行って、出力が生成されます。</span><span class="sxs-lookup"><span data-stu-id="1833a-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="1833a-110">アプリケーション全体は、一定時間 (数分から数日) 実行されます。</span><span class="sxs-lookup"><span data-stu-id="1833a-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="1833a-111">一般的なパターンは、バーストにより大量のコアをプロビジョニングし、アプリケーションが完了するとコア数はゼロまで下降します。</span><span class="sxs-lookup"><span data-stu-id="1833a-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="1833a-112">アプリケーションを常時実行し続ける必要はありません。</span><span class="sxs-lookup"><span data-stu-id="1833a-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="1833a-113">ただし、システムでは、ノードの障害またはアプリケーションのクラッシュを処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1833a-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="1833a-114">一部のアプリケーションでは、タスクは独立しており、並列して実行できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="1833a-115">または、タスクが緊密に結合されているため、相互に交信または中間結果を交換する必要がある場合もあります。</span><span class="sxs-lookup"><span data-stu-id="1833a-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="1833a-116">この場合、InfiniBand やリモート ダイレクト メモリ アクセス (RDMA) などの高速ネットワーク テクノロジを使用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="1833a-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="1833a-117">ワークロードに応じてコンピューティング集中型の VM サイズ (H16r、H16mr、および A9) を使用することができます。</span><span class="sxs-lookup"><span data-stu-id="1833a-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="1833a-118">このアーキテクチャを使用する条件</span><span class="sxs-lookup"><span data-stu-id="1833a-118">When to use this architecture</span></span>

- <span data-ttu-id="1833a-119">シミュレーション や計算などの大量の計算操作。</span><span class="sxs-lookup"><span data-stu-id="1833a-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="1833a-120">計算負荷が高く、複数のコンピューター (10 ～ 1000 台) の CPU に分割する必要があるシミュレーション。</span><span class="sxs-lookup"><span data-stu-id="1833a-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="1833a-121">1 台のコンピューター上の大量のメモリを必要とするシミュレーションでは、複数のコンピューターに分割する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1833a-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="1833a-122">1 台のコンピューターでは計算に時間がかかりすぎる長い計算。</span><span class="sxs-lookup"><span data-stu-id="1833a-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="1833a-123">モンテカルロ シミュレーションなどの、数百または数千回実行する必要がある小規模な計算。</span><span class="sxs-lookup"><span data-stu-id="1833a-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="1833a-124">メリット</span><span class="sxs-lookup"><span data-stu-id="1833a-124">Benefits</span></span>

- <span data-ttu-id="1833a-125">"[驚異的並列][embarrassingly-parallel]" 処理による高いパフォーマンス。</span><span class="sxs-lookup"><span data-stu-id="1833a-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="1833a-126">大きな問題を高速で解決するために何百、何千ものコンピューター コアを使用できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="1833a-127">専用の高速 InfiniBand ネットワークを使用した、特殊な高性能ハードウェアの使用。</span><span class="sxs-lookup"><span data-stu-id="1833a-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="1833a-128">作業中、必要に応じて VM をプロビジョニングし、削除できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="1833a-129">課題</span><span class="sxs-lookup"><span data-stu-id="1833a-129">Challenges</span></span>

- <span data-ttu-id="1833a-130">VM インフラストラクチャの管理。</span><span class="sxs-lookup"><span data-stu-id="1833a-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="1833a-131">大量の計算の管理。</span><span class="sxs-lookup"><span data-stu-id="1833a-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="1833a-132">適切なタイミングで数千のコアをプロビジョニングする。</span><span class="sxs-lookup"><span data-stu-id="1833a-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="1833a-133">緊密に結合されたタスクにコアを追加すると逆効果になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="1833a-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="1833a-134">実験を通して最適なコア数を特定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1833a-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="1833a-135">Azure Batch を使用したビッグ コンピューティング</span><span class="sxs-lookup"><span data-stu-id="1833a-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="1833a-136">[Azure Batc][batch] は、大規模な高パフォーマンス コンピューティング (HPC) のアプリケーションを実行するためのマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="1833a-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="1833a-137">Azure Batch を使用して、VM プールを構成し、アプリケーションとデータ ファイルをアップロードします。</span><span class="sxs-lookup"><span data-stu-id="1833a-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="1833a-138">バッチ サービスにより VM がプロビジョニングされ、VM にタスクが割り当てられ、タスクが実行され、進行状況が監視されます。</span><span class="sxs-lookup"><span data-stu-id="1833a-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="1833a-139">バッチは、ワークロードに応じて VM を自動的にスケール アウトできます。</span><span class="sxs-lookup"><span data-stu-id="1833a-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="1833a-140">また、バッチは、ジョブのスケジューリングも提供します。</span><span class="sxs-lookup"><span data-stu-id="1833a-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="1833a-141">Virtual Machines で実行されるビッグ コンピューティング</span><span class="sxs-lookup"><span data-stu-id="1833a-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="1833a-142">[Microsoft HPC Pack][hpc-pack] を使用して VM のクラスターを管理して、HPC ジョブをスケジュールおよび監視できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="1833a-143">この方法では、ユーザーが VM およびネットワーク インフラストラクチャのプロビジョニングや管理を行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="1833a-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="1833a-144">既存の HPC ワークロードの一部またはすべてを Azure に移動する場合は、このアプローチを検討してください。</span><span class="sxs-lookup"><span data-stu-id="1833a-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="1833a-145">HPC クラスター全体を Azure に移動するか、HPC クラスターはオンプレミスにしたまま、バースト容量のために Azure を使用できます。</span><span class="sxs-lookup"><span data-stu-id="1833a-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="1833a-146">詳細については、[大規模コンピューティング ワークロードのための Batch および HPC ソリューション][batch-hpc-solutions]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1833a-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="1833a-147">Azure にデプロイされた HPC Pack</span><span class="sxs-lookup"><span data-stu-id="1833a-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="1833a-148">このシナリオでは、HPC クラスターすべてを Azure 内で作成します。</span><span class="sxs-lookup"><span data-stu-id="1833a-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="1833a-149">ヘッド ノードは、管理およびジョブ スケジューリング サービスをクラスターに提供します。</span><span class="sxs-lookup"><span data-stu-id="1833a-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="1833a-150">緊密に結合されたタスクの場合は、非常に高い帯域幅、低待機時間の VM 間通信を提供する RDMA ネットワークを使用します。</span><span class="sxs-lookup"><span data-stu-id="1833a-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="1833a-151">詳細については、「[Azure に HPC Pack 2016 クラスターをデプロイする][deploy-hpc-azure]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1833a-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="1833a-152">Azure への HPC クラスターのバースト</span><span class="sxs-lookup"><span data-stu-id="1833a-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="1833a-153">このシナリオでは、組織は、HPC Pack をオンプレミスで実行しており、バースト容量のために Azure VM を使用します。</span><span class="sxs-lookup"><span data-stu-id="1833a-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="1833a-154">クラスターのヘッド ノードは、オンプレミスに設置されています。</span><span class="sxs-lookup"><span data-stu-id="1833a-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="1833a-155">ExpressRoute または VPN Gateway は、オンプレミス ネットワークを Azure VNet に接続します。</span><span class="sxs-lookup"><span data-stu-id="1833a-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
