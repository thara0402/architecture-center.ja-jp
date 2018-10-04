---
title: Azure での 3D ビデオのレンダリング
description: Azure Batch サービスを使用した、Azure でのネイティブ HPC ワークロードの実行
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 67dc8496656a75eab8f5d0ce45d00f8b1f4ea03f
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428059"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="feb98-103">Azure での 3D ビデオのレンダリング</span><span class="sxs-lookup"><span data-stu-id="feb98-103">3D video rendering on Azure</span></span>

<span data-ttu-id="feb98-104">3D レンダリングは時間がかかるプロセスで、完了までにかなりの CPU 時間が必要です。</span><span class="sxs-lookup"><span data-stu-id="feb98-104">3D rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span>  <span data-ttu-id="feb98-105">1 台のコンピューターで静的なアセットからビデオ ファイルを生成している場合、作成しているビデオの長さと複雑さよっては、そのプロセスに数時間から、数日かかる場合もあります。</span><span class="sxs-lookup"><span data-stu-id="feb98-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="feb98-106">多くの企業は、これらのタスクを実行するために高価なハイエンド デスクトップ コンピューターを購入するか、ジョブを送信できる大規模なレンダリング ファームに投資しています。</span><span class="sxs-lookup"><span data-stu-id="feb98-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="feb98-107">しかし、Azure Batch を使用すれば、その能力を必要なときだけ使用して、不要なときはシャットダウンできます。資本投資は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="feb98-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="feb98-108">Windows Server と Linux コンピューティングのどちらのノードを選択しても、Batch によって一貫したジョブの管理やスケジュール設定を実行できます。</span><span class="sxs-lookup"><span data-stu-id="feb98-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="feb98-109">Batch を使用すると、AutoDesk Maya、Blender など、既存の Windows または Linux アプリケーションを使用して、Azure で大規模なレンダリング ジョブを実行することができます。</span><span class="sxs-lookup"><span data-stu-id="feb98-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="feb98-110">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="feb98-110">Related use cases</span></span>

<span data-ttu-id="feb98-111">次のようなユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="feb98-112">3D モデリング</span><span class="sxs-lookup"><span data-stu-id="feb98-112">3D Modeling</span></span>
* <span data-ttu-id="feb98-113">Visual FX (VFX) レンダリング</span><span class="sxs-lookup"><span data-stu-id="feb98-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="feb98-114">ビデオ コード変換</span><span class="sxs-lookup"><span data-stu-id="feb98-114">Video transcoding</span></span>
* <span data-ttu-id="feb98-115">画像処理、色補正、およびサイズ変更</span><span class="sxs-lookup"><span data-stu-id="feb98-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="feb98-116">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="feb98-116">Architecture</span></span>

![Azure Batch を使用したクラウド ネイティブ HPC ソリューションに関与するコンポーネントのアーキテクチャの概要][architecture]

<span data-ttu-id="feb98-118">このサンプル シナリオでは、Azure Batch を使用するときのワークフローに対応します。データ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="feb98-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="feb98-119">入力ファイルとそのファイルを処理するアプリケーションを Azure Storage アカウントにアップロードする</span><span class="sxs-lookup"><span data-stu-id="feb98-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="feb98-120">Batch アカウント内のコンピューティング ノードの Batch プール、プールでワークロードを実行するジョブ、およびジョブ内のタスクを作成する。</span><span class="sxs-lookup"><span data-stu-id="feb98-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="feb98-121">入力ファイルとアプリケーションを Batch にダウンロードする</span><span class="sxs-lookup"><span data-stu-id="feb98-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="feb98-122">タスクの実行を監視する</span><span class="sxs-lookup"><span data-stu-id="feb98-122">Monitor task execution</span></span>
5. <span data-ttu-id="feb98-123">タスク出力をアップロードする</span><span class="sxs-lookup"><span data-stu-id="feb98-123">Upload task output</span></span>
6. <span data-ttu-id="feb98-124">出力ファイルをダウンロードする</span><span class="sxs-lookup"><span data-stu-id="feb98-124">Download output files</span></span>

<span data-ttu-id="feb98-125">このプロセスを簡略化するために、[Maya および 3ds Max 用の Batch プラグイン][batch-plugins]を使用することもできます</span><span class="sxs-lookup"><span data-stu-id="feb98-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="feb98-126">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="feb98-126">Components</span></span>

<span data-ttu-id="feb98-127">Azure Batch は、次の Azure テクノロジに基づいています。</span><span class="sxs-lookup"><span data-stu-id="feb98-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="feb98-128">[リソース グループ][resource-groups]は、Azure リソースの論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="feb98-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="feb98-129">[仮想ネットワーク][vnet]は、ヘッド ノードとコンピューティング リソースの両方に使用されます</span><span class="sxs-lookup"><span data-stu-id="feb98-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="feb98-130">[ストレージ][storage] アカウントは、同期とデータ保有に使用されます</span><span class="sxs-lookup"><span data-stu-id="feb98-130">[Storage][storage] accounts are used for the synchronization and data retention</span></span>
* <span data-ttu-id="feb98-131">[仮想マシン スケール セット][vmss]は、CycleCloud によってコンピューティング リソース用に使用されます</span><span class="sxs-lookup"><span data-stu-id="feb98-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="feb98-132">考慮事項</span><span class="sxs-lookup"><span data-stu-id="feb98-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="feb98-133">Azure Batch で使用できるマシンのサイズ</span><span class="sxs-lookup"><span data-stu-id="feb98-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="feb98-134">レンダリング カスタマーのほとんどが高 CPU パワーのリソースを選択しますが、仮想マシンスケール セットを使用するワークロードでは VM を異なる方法で選択でき、そのワークロードはさまざまな要因に依存します。</span><span class="sxs-lookup"><span data-stu-id="feb98-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

* <span data-ttu-id="feb98-135">バインドされたメモリでアプリケーションが実行されているか。</span><span class="sxs-lookup"><span data-stu-id="feb98-135">Is the application being run memory bound?</span></span>
* <span data-ttu-id="feb98-136">アプリケーションが GPU を使用する必要があるか。</span><span class="sxs-lookup"><span data-stu-id="feb98-136">Does the application need to use GPUs?</span></span> 
* <span data-ttu-id="feb98-137">ジョブの種類が驚異的並列か、または密に結合されたジョブに対する InfiniBand 接続が必要か。</span><span class="sxs-lookup"><span data-stu-id="feb98-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
* <span data-ttu-id="feb98-138">コンピューティング ノードでストレージへの高速の I/O が必要か</span><span class="sxs-lookup"><span data-stu-id="feb98-138">Require fast I/O to storage on the compute Nodes</span></span>

<span data-ttu-id="feb98-139">Azure では、上記のアプリケーション要件のそれぞれに対処できる広範な VM サイズが必要で、その一部は HPC に固有ですが、最小のサイズを使用して、効果的なグリッド実装を提供することもできます。</span><span class="sxs-lookup"><span data-stu-id="feb98-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

* <span data-ttu-id="feb98-140">[HPC VM サイズ][compute-hpc]: CPU バインド型レンダリングの性質があるため、Microsoft では、通常、Azure H シリーズの VM を提案しています。</span><span class="sxs-lookup"><span data-stu-id="feb98-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span>  <span data-ttu-id="feb98-141">この種類の VM は、特にハイエンド コンピューティング ニーズ用に構築され、8 および 16 コア vCPU サイズが使用可能で、DDR4 メモリ、SSD 一時ストレージ、および Haswell E5 Intel テクノロジを備えています。</span><span class="sxs-lookup"><span data-stu-id="feb98-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
* <span data-ttu-id="feb98-142">[GPU VM サイズ][compute-gpu]: GPU 最適化済み VM サイズは、1 つまたは複数の NVIDIA GPU を備えた、特殊な用途に特化した仮想マシンです。</span><span class="sxs-lookup"><span data-stu-id="feb98-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="feb98-143">これらのサイズは、コンピューティング処理やグラフィック処理の負荷が高い視覚化ワークロードを意図して設計されています。</span><span class="sxs-lookup"><span data-stu-id="feb98-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
* <span data-ttu-id="feb98-144">NC、NCv2、NCv3、ND の各サイズは、CUDA および OpenCL ベースのアプリケーションやシミュレーション、AI、ディープ ラーニングなどの、コンピューティング集中型およびネットワーク集中型のアプリケーション、アルゴリズム用に最適化されています。</span><span class="sxs-lookup"><span data-stu-id="feb98-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="feb98-145">NV サイズは、リモートの視覚化、ストリーミング、ゲーム、エンコーディング、および OpenGL や DirectX などのフレームワークを使用する VDI シナリオ用に最適化されています。</span><span class="sxs-lookup"><span data-stu-id="feb98-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
* <span data-ttu-id="feb98-146">[メモリ最適化 VM サイズ][compute-memory]: さらに多くのメモリが必要な場合は、より高いメモリ対 CPU 比率を提供するメモリ最適化 VM サイズを使用します。</span><span class="sxs-lookup"><span data-stu-id="feb98-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
* <span data-ttu-id="feb98-147">[汎用 VM サイズ][compute-general]: バランスのとれた CPU 対メモリ比率を提供する汎用 VM サイズを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="feb98-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="feb98-148">代替手段</span><span class="sxs-lookup"><span data-stu-id="feb98-148">Alternatives</span></span>

<span data-ttu-id="feb98-149">Azure のレンダリング環境をより細かく制御する必要がある場合、またはハイブリッド実装が必要な場合は、CycleCloud コンピューティングがクラウドでの IaaS グリッドの調整に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="feb98-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="feb98-150">基盤となる Azure テクノロジとして Azure Batch と同じものを使用すると、IaaS グリッドの作成および維持プロセスを効率化できます。</span><span class="sxs-lookup"><span data-stu-id="feb98-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="feb98-151">設計の原則の詳細については、次のリンクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="feb98-152">Azure で使用できるすべての HPC ソリューションの完全な概要については、「[Azure VM を使用した HPC、Batch、および Big Compute ソリューション][hpc-alt-solutions]」を参照してください</span><span class="sxs-lookup"><span data-stu-id="feb98-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="feb98-153">可用性</span><span class="sxs-lookup"><span data-stu-id="feb98-153">Availability</span></span>

<span data-ttu-id="feb98-154">さまざまなサービス、ツール、および API を利用して、Azure Batch のコンポーネントを監視できます。</span><span class="sxs-lookup"><span data-stu-id="feb98-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="feb98-155">監視の詳細については、「[Batch ソリューションの監視][batch-monitor]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="feb98-156">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="feb98-156">Scalability</span></span>

<span data-ttu-id="feb98-157">Azure Batch アカウント内のプールをスケーリングするには、手動で介入するか、Azure Batch メトリックに基づく数式を使用して自動的に行います。</span><span class="sxs-lookup"><span data-stu-id="feb98-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="feb98-158">スケーラビリティの詳細については、[Batch プール内のノードをスケーリングするための自動スケーリングの数式の作成][batch-scaling]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="feb98-159">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="feb98-159">Security</span></span>

<span data-ttu-id="feb98-160">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="feb98-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="feb98-161">回復性</span><span class="sxs-lookup"><span data-stu-id="feb98-161">Resiliency</span></span>

<span data-ttu-id="feb98-162">現在 Azure Batch にはフェールオーバー機能がありません。計画外の停止が発生した場合は、次の手順を使用して、可用性を確保することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="feb98-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

* <span data-ttu-id="feb98-163">代替ストレージ アカウントを使用して代替の Azure の場所で Azure Batch アカウントを作成します</span><span class="sxs-lookup"><span data-stu-id="feb98-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="feb98-164">同じノード プールを、同じ名前で、ノードの割り当てなしで作成します</span><span class="sxs-lookup"><span data-stu-id="feb98-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
* <span data-ttu-id="feb98-165">アプリケーションが作成され、代替ストレージ アカウントに更新されていることを確認します</span><span class="sxs-lookup"><span data-stu-id="feb98-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="feb98-166">入力ファイルをアップロードし、ジョブを代替 Azure Batch アカウントに送信します</span><span class="sxs-lookup"><span data-stu-id="feb98-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="feb98-167">このシナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="feb98-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="feb98-168">Azure Batch アカウントとプールを手動で作成する</span><span class="sxs-lookup"><span data-stu-id="feb98-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="feb98-169">このサンプル シナリオでは、ご自身の顧客向けに開発できる Azure Batch ラボを SaaS ソリューションの例として紹介しながら、Azure Batch のしくみを学習するうえで役立ちます。</span><span class="sxs-lookup"><span data-stu-id="feb98-169">This sample scenario helps in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="feb98-170">[Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="feb98-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a><span data-ttu-id="feb98-171">Azure Resource Manager テンプレートを使用してサンプル シナリオをデプロイする</span><span class="sxs-lookup"><span data-stu-id="feb98-171">Deploying the sample scenario using an Azure Resource Manager template</span></span>

<span data-ttu-id="feb98-172">テンプレートによってデプロイされるもの:</span><span class="sxs-lookup"><span data-stu-id="feb98-172">The template will deploy:</span></span>

* <span data-ttu-id="feb98-173">新しい Azure Batch アカウント</span><span class="sxs-lookup"><span data-stu-id="feb98-173">A new Azure Batch account</span></span>
* <span data-ttu-id="feb98-174">ストレージ アカウント</span><span class="sxs-lookup"><span data-stu-id="feb98-174">A storage account</span></span>
* <span data-ttu-id="feb98-175">バッチ アカウントに関連付けられているノード プール</span><span class="sxs-lookup"><span data-stu-id="feb98-175">A node pool associated with the batch account</span></span>
* <span data-ttu-id="feb98-176">ノード プールは、A2 v2 VM Canonical Ubuntu イメージを使用するように構成されます</span><span class="sxs-lookup"><span data-stu-id="feb98-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
* <span data-ttu-id="feb98-177">ノード プールには最初は VM がなく、VM を追加して手動でスケーリングする必要があります</span><span class="sxs-lookup"><span data-stu-id="feb98-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="feb98-178">詳しくは、[Resource Manager テンプレート][azure-arm-templates]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="feb98-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="feb98-179">価格</span><span class="sxs-lookup"><span data-stu-id="feb98-179">Pricing</span></span>

<span data-ttu-id="feb98-180">Azure Batch の使用コストは、プールに使用されている VM のサイズと VM が割り当てられ実行されている期間によって異なります。Azure Batch アカウントの作成に関連するコストはありません。</span><span class="sxs-lookup"><span data-stu-id="feb98-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="feb98-181">ストレージとデータの送信を追加コストとして考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="feb98-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="feb98-182">あるジョブを 8 時間実行した場合に発生するコストの例を、サーバー数ごとに次に示します。</span><span class="sxs-lookup"><span data-stu-id="feb98-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

* <span data-ttu-id="feb98-183">100 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="feb98-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="feb98-184">100 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信</span><span class="sxs-lookup"><span data-stu-id="feb98-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="feb98-185">50 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="feb98-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="feb98-186">50 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信</span><span class="sxs-lookup"><span data-stu-id="feb98-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="feb98-187">10 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="feb98-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="feb98-188">10 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信</span><span class="sxs-lookup"><span data-stu-id="feb98-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="feb98-189">低優先度 VM の価格</span><span class="sxs-lookup"><span data-stu-id="feb98-189">Low-Priority VM Pricing</span></span>

<span data-ttu-id="feb98-190">Azure Batch では、ノード プールでの低優先度 VM\* の使用もサポートされます。これにより、大幅にコストを削減できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="feb98-190">Azure Batch also supports the use of Low-Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="feb98-191">標準 VM と低優先度 VM の価格の比較と、低優先度 VM の詳細については、「[Batch の価格][batch-pricing]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-191">For a price comparison between standard VMs and Low-Priority VMs, and to find out more about Low-Priority VMs, see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="feb98-192">\*低優先度 VM での実行に適しているのは、特定のアプリケーションとワークロードだけであることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="feb98-192">\* Please note that only certain applications and workloads will be suitable to run on Low-Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="feb98-193">関連リソース</span><span class="sxs-lookup"><span data-stu-id="feb98-193">Related Resources</span></span>

<span data-ttu-id="feb98-194">[Azure Batch の概要][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="feb98-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="feb98-195">[Azure Batch のドキュメント][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="feb98-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="feb98-196">[Azure Batch でのコンテナーの使用][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="feb98-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job