---
title: Azure での R モデルのバッチ スコアリング
description: Azure Batch と、小売店の売上予測に基づくデータ セットを使用して、R モデルでバッチ スコアリングを実行します。
author: njray
ms.date: 03/29/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 4fa57168c337b01c8e7d0fc86ba54fee59a7ae47
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887874"
---
# <a name="batch-scoring-of-r-machine-learning-models-on-azure"></a><span data-ttu-id="50137-103">Azure での R 機械学習モデルのバッチ スコアリング</span><span class="sxs-lookup"><span data-stu-id="50137-103">Batch scoring of R machine learning models on Azure</span></span>

<span data-ttu-id="50137-104">この参照アーキテクチャでは、Azure Batch を使用して R モデルでバッチ スコアリングを実行する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="50137-104">This reference architecture shows how to perform batch scoring with R models using Azure Batch.</span></span> <span data-ttu-id="50137-105">このシナリオは小売店の売上予測に基づいていますが、このアーキテクチャは、R モデルを使用した大規模な予測の生成を必要とするどのシナリオでも一般化することができます。</span><span class="sxs-lookup"><span data-stu-id="50137-105">The scenario is based on retail store sales forecasting, but this architecture can be generalized for any scenario requiring the generation of predictions on a large scaler using R models.</span></span> <span data-ttu-id="50137-106">このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="50137-106">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![アーキテクチャ ダイアグラム][0]

<span data-ttu-id="50137-108">**シナリオ**: あるスーパー マーケット チェーンで、次の四半期の商品売上を予測する必要があります。</span><span class="sxs-lookup"><span data-stu-id="50137-108">**Scenario**: A supermarket chain needs to forecast sales of products over the upcoming quarter.</span></span> <span data-ttu-id="50137-109">この予測により、企業はサプライ チェーンをより的確に管理して、各店舗の商品の需要に確実に対応することができます。</span><span class="sxs-lookup"><span data-stu-id="50137-109">The forecast allows the company to manage its supply chain better and ensure it can meet demand for products at each of its stores.</span></span> <span data-ttu-id="50137-110">この企業では、毎週、前の週の新しい売上データが利用可能になるたびに予測を更新して、次の四半期のマーケティング戦略が設定されます。</span><span class="sxs-lookup"><span data-stu-id="50137-110">The company updates its forecasts every week as new sales data from the previous week becomes available and the product marketing strategy for next quarter is set.</span></span> <span data-ttu-id="50137-111">個々の売上予測の不確実性を推測するために分位点予測が生成されます。</span><span class="sxs-lookup"><span data-stu-id="50137-111">Quantile forecasts are generated to estimate the uncertainty of the individual sales forecasts.</span></span>

<span data-ttu-id="50137-112">処理には次の手順が含まれます。</span><span class="sxs-lookup"><span data-stu-id="50137-112">Processing involves the following steps:</span></span>

1. <span data-ttu-id="50137-113">Azure Logic App が、週に 1 回、予測生成処理をトリガーします。</span><span class="sxs-lookup"><span data-stu-id="50137-113">An Azure Logic App triggers the forecast generation process once per week.</span></span>

1. <span data-ttu-id="50137-114">ロジック アプリは、スケジューラー Docker コンテナーを実行する Azure コンテナー インスタンスを開始し、それにより Batch クラスターでスコアリング ジョブがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="50137-114">The logic app starts an Azure Container Instance running the scheduler Docker container, which triggers the scoring jobs on the Batch cluster.</span></span>

1. <span data-ttu-id="50137-115">スコアリング ジョブは、Batch クラスターの複数のノードにわたって並列で実行されます。</span><span class="sxs-lookup"><span data-stu-id="50137-115">Scoring jobs run in parallel across the nodes of the Batch cluster.</span></span> <span data-ttu-id="50137-116">各ノードは以下のことを実行します。</span><span class="sxs-lookup"><span data-stu-id="50137-116">Each node:</span></span>

    1. <span data-ttu-id="50137-117">Docker Hub からワーカー Docker イメージをプルして、コンテナーを開始する。</span><span class="sxs-lookup"><span data-stu-id="50137-117">Pulls the worker Docker image from Docker Hub and starts a container.</span></span>

    1. <span data-ttu-id="50137-118">Azure Blob Storage から入力データと事前トレーニング済みの R モデルを読み取る。</span><span class="sxs-lookup"><span data-stu-id="50137-118">Reads input data and pre-trained R models from Azure Blob storage.</span></span>

    1. <span data-ttu-id="50137-119">データのスコアリングを行って予測を生成する。</span><span class="sxs-lookup"><span data-stu-id="50137-119">Scores the data to produce the forecasts.</span></span>

    1. <span data-ttu-id="50137-120">予測結果を Blob Storage に書き込む。</span><span class="sxs-lookup"><span data-stu-id="50137-120">Writes the forecast results to blob storage.</span></span>

<span data-ttu-id="50137-121">次の図は、ある店舗での 4 つの商品 (SKU) の予測売上を示しています。</span><span class="sxs-lookup"><span data-stu-id="50137-121">The figure below shows the forecasted sales for four products (SKUs) in one store.</span></span> <span data-ttu-id="50137-122">黒の線は売上の履歴、破線は中央値 (q50) の予測、ピンク色の部分は 25 パーセンタイルと 75 パーセン タイルを表しており、青色の部分は 5 パーセンタイルと 95 パーセンタイルを表しています。</span><span class="sxs-lookup"><span data-stu-id="50137-122">The black line is the sales history, the dashed line is the median (q50) forecast, the pink band represents the twenty-fifth and seventy-fifth percentiles, and the blue band represents the fifth and ninety-fifth percentiles.</span></span>

![売上予測][1]

## <a name="architecture"></a><span data-ttu-id="50137-124">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="50137-124">Architecture</span></span>

<span data-ttu-id="50137-125">このアーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="50137-125">This architecture consists of the following components.</span></span>

<span data-ttu-id="50137-126">[Azure Batch][batch] は、クラスターまたは仮想マシンで予測生成ジョブを並列に実行するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="50137-126">[Azure Batch][batch] is used to run forecast generation jobs in parallel on a cluster of virtual machines.</span></span> <span data-ttu-id="50137-127">予測は、R で実装されたトレーニング済みの機械学習モデルを使用して行われます。Azure Batch は、クラスターに送信されたジョブの数に基づいて VM の数を自動的にスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="50137-127">Predictions are made using pre-trained machine learning models implemented in R. Azure Batch can automatically scale the number of VMs based on the number of jobs submitted to the cluster.</span></span> <span data-ttu-id="50137-128">各ノードでは、R スクリプトが Docker コンテナー内で実行されてデータのスコアリングが行われ、予測が生成されます。</span><span class="sxs-lookup"><span data-stu-id="50137-128">On each node, an R script runs within a Docker container to score data and generate forecasts.</span></span>

<span data-ttu-id="50137-129">[Azure Blob Storage][blob] は、入力データ、事前トレーニング済みの機械学習モデル、および予測結果を格納するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="50137-129">[Azure Blob Storage][blob] is used to store the input data, the pre-trained machine learning models, and the forecast results.</span></span> <span data-ttu-id="50137-130">これは、このワークロードに必要なパフォーマンスに対して非常に費用効率の良いストレージを提供します。</span><span class="sxs-lookup"><span data-stu-id="50137-130">It delivers very cost-effective storage for the performance that this workload requires.</span></span>

<span data-ttu-id="50137-131">[Azure Container Instances][aci] は、オンデマンドでサーバーレス コンピューティングを提供します。</span><span class="sxs-lookup"><span data-stu-id="50137-131">[Azure Container Instances][aci] provide serverless compute on demand.</span></span> <span data-ttu-id="50137-132">このケースでは、スケジュールに基づいてコンテナー インスタンスがデプロイされ、予測を生成する Batch ジョブがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="50137-132">In this case, a container instance is deployed on a schedule to trigger the Batch jobs that generate the forecasts.</span></span> <span data-ttu-id="50137-133">Batch ジョブは、[doAzureParallel][doAzureParallel] パッケージを使用して R スクリプトからトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="50137-133">The Batch jobs are triggered from an R script using the [doAzureParallel][doAzureParallel] package.</span></span> <span data-ttu-id="50137-134">ジョブが完了すると、コンテナー インスタンスが自動的にシャットダウンされます。</span><span class="sxs-lookup"><span data-stu-id="50137-134">The container instance automatically shuts down once the jobs have finished.</span></span>

<span data-ttu-id="50137-135">[Azure Logic Apps][logic-apps] は、スケジュールに基づいてコンテナー インスタンスをデプロイして、ワークフロー全体をトリガーします。</span><span class="sxs-lookup"><span data-stu-id="50137-135">[Azure Logic Apps][logic-apps] trigger the entire workflow by deploying the container instances on a schedule.</span></span> <span data-ttu-id="50137-136">Logic Apps 内の Azure Container Instances コネクターにより、一連のトリガー イベントでインスタンスがデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="50137-136">An Azure Container Instances connector in Logic Apps allows an instance to be deployed upon a range of trigger events.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="50137-137">パフォーマンスに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="50137-137">Performance considerations</span></span>

### <a name="containerized-deployment"></a><span data-ttu-id="50137-138">コンテナー化されたデプロイ</span><span class="sxs-lookup"><span data-stu-id="50137-138">Containerized deployment</span></span>

<span data-ttu-id="50137-139">このアーキテクチャでは、すべての R スクリプトは [Docker](https://www.docker.com/) コンテナー内で実行されます。</span><span class="sxs-lookup"><span data-stu-id="50137-139">With this architecture, all R scripts run within [Docker](https://www.docker.com/) containers.</span></span> <span data-ttu-id="50137-140">これにより、スクリプトは毎回、一貫性のある環境で、同じ R バージョンとパッケージ バージョンを使用して実行されます。</span><span class="sxs-lookup"><span data-stu-id="50137-140">This ensures that the scripts run in a consistent environment, with the same R version and packages versions, every time.</span></span> <span data-ttu-id="50137-141">スケジューラー コンテナーとワーカー コンテナーには別々の Docker イメージが使用されます。これは、それぞれ異なる R パッケージ依存関係のセットを持っているからです。</span><span class="sxs-lookup"><span data-stu-id="50137-141">Separate Docker images are used for the scheduler and worker containers, because each has a different set of R package dependencies.</span></span>

<span data-ttu-id="50137-142">Azure Container Instances は、スケジューラー コンテナーを実行するためのサーバーレス環境を提供します。</span><span class="sxs-lookup"><span data-stu-id="50137-142">Azure Container Instances provides a serverless environment to run the scheduler container.</span></span> <span data-ttu-id="50137-143">スケジューラ コンテナーは、Azure Batch クラスターで実行される個々のスコアリング ジョブをトリガーする R スクリプトを実行します。</span><span class="sxs-lookup"><span data-stu-id="50137-143">The scheduler container runs an R script that triggers the individual scoring jobs running on an Azure Batch cluster.</span></span>

<span data-ttu-id="50137-144">Batch クラスターの各ノードは、スコアリング スクリプトを実行するワーカー コンテナーを実行します。</span><span class="sxs-lookup"><span data-stu-id="50137-144">Each node of the Batch cluster runs the worker container, which executes the scoring script.</span></span>

### <a name="parallelizing-the-workload"></a><span data-ttu-id="50137-145">ワークロードを並列化する</span><span class="sxs-lookup"><span data-stu-id="50137-145">Parallelizing the workload</span></span>

<span data-ttu-id="50137-146">R モデルでデータのバッチ スコアリングを行うときは、ワークロードを並列化することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="50137-146">When batch scoring data with R models, consider how to parallelize the workload.</span></span> <span data-ttu-id="50137-147">スコアリング操作を複数のクラスター ノードに分散できるように、入力データをパーティション分割する必要があります。</span><span class="sxs-lookup"><span data-stu-id="50137-147">The input data must be partitioned somehow so that the scoring operation can be distributed  across the cluster nodes.</span></span> <span data-ttu-id="50137-148">さまざまな方法を試して、ワークロードを分散するための最良の選択を見つけてください。</span><span class="sxs-lookup"><span data-stu-id="50137-148">Try different approaches to discover the best choice for distributing your workload.</span></span> <span data-ttu-id="50137-149">ケースごとに、次のことを検討してください。</span><span class="sxs-lookup"><span data-stu-id="50137-149">On a case-by-case basis, consider the following:</span></span>

- <span data-ttu-id="50137-150">1 つのノードのメモリで読み込んで処理できるデータの量。</span><span class="sxs-lookup"><span data-stu-id="50137-150">How much data can be loaded and processed in the memory of a single node.</span></span>
- <span data-ttu-id="50137-151">各 Batch ジョブの開始によるオーバーヘッド。</span><span class="sxs-lookup"><span data-stu-id="50137-151">The overhead of starting each batch job.</span></span>
- <span data-ttu-id="50137-152">R モデルの読み込みによるオーバーヘッド。</span><span class="sxs-lookup"><span data-stu-id="50137-152">The overhead of loading the R models.</span></span>

<span data-ttu-id="50137-153">この例で使用されているシナリオでは、モデル オブジェクトは大型で、個別の商品の予測を生成するためにかかる時間は数秒です。</span><span class="sxs-lookup"><span data-stu-id="50137-153">In the scenario used for this example, the model objects are large, and it takes only a few seconds to generate a forecast for individual products.</span></span> <span data-ttu-id="50137-154">この理由により、商品をグループ化して、ノードごとに 1 つの Batch ジョブを実行することができます。</span><span class="sxs-lookup"><span data-stu-id="50137-154">For this reason, you can group the products and execute a single Batch job per node.</span></span> <span data-ttu-id="50137-155">各ジョブ内のループにより、順番に商品の予測が生成されます。</span><span class="sxs-lookup"><span data-stu-id="50137-155">A loop within each job generates forecasts for the products sequentially.</span></span> <span data-ttu-id="50137-156">この方式が、結局はこの特定のワークロードを並列化する最も効率的な方法になります。</span><span class="sxs-lookup"><span data-stu-id="50137-156">This method turns out to be the most efficient way to parallelize this particular workload.</span></span> <span data-ttu-id="50137-157">この方式では、数多くの小さなバッチ ジョブを開始して、繰り返し R モデルを読み込むことによるオーバーヘッドが回避されます。</span><span class="sxs-lookup"><span data-stu-id="50137-157">It avoids the overhead of starting many smaller Batch jobs and repeatedly loading the R models.</span></span>

<span data-ttu-id="50137-158">別の方法は、商品ごとに 1 つのバッチ ジョブをトリガーするというものです。</span><span class="sxs-lookup"><span data-stu-id="50137-158">An alternative approach is to trigger one Batch job per product.</span></span> <span data-ttu-id="50137-159">Azure Batch は自動的にジョブのキューを形成し、ノードが使用可能になるとそれらをクラスターで実行するために送信します。</span><span class="sxs-lookup"><span data-stu-id="50137-159">Azure Batch automatically forms a queue of jobs and submits them to be executed on the cluster as nodes become available.</span></span> <span data-ttu-id="50137-160">[自動スケーリング][autoscale]を使用して、ジョブの数に応じてクラスター内のノードの数を調整します。</span><span class="sxs-lookup"><span data-stu-id="50137-160">Use [automatic scaling][autoscale] to adjust the number of nodes in the cluster depending on the number of jobs.</span></span> <span data-ttu-id="50137-161">この方法は、各スコアリング操作の完了に比較的長い時間がかかり、ジョブを開始してモデル オブジェクトを再読み込みすることによるオーバーヘッドが正当化される場合には理にかなっています。</span><span class="sxs-lookup"><span data-stu-id="50137-161">This approach makes more sense if it takes a relatively long time to complete each scoring operation, justifying the overhead of starting the jobs and reloading the model objects.</span></span> <span data-ttu-id="50137-162">また、この方法は実装が容易であり、合計ワークロードが事前にわからない場合に重要な考慮事項となる自動スケーリングを使用する柔軟性も得ることができます。</span><span class="sxs-lookup"><span data-stu-id="50137-162">This approach is also simpler to implement and gives you the flexibility to use automatic scaling—an important consideration if the size of the total workload is not known in advance.</span></span>

## <a name="monitoring-and-logging-considerations"></a><span data-ttu-id="50137-163">監視とログ記録に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="50137-163">Monitoring and logging considerations</span></span>

### <a name="monitoring-azure-batch-jobs"></a><span data-ttu-id="50137-164">Azure Batch ジョブの監視</span><span class="sxs-lookup"><span data-stu-id="50137-164">Monitoring Azure Batch jobs</span></span>

<span data-ttu-id="50137-165">Batch ジョブの監視と終了は、Azure portal の Batch アカウントの **[ジョブ]** ウィンドウから行います。</span><span class="sxs-lookup"><span data-stu-id="50137-165">Monitor and terminate Batch jobs from the **Jobs** pane of the Batch account in the Azure portal.</span></span> <span data-ttu-id="50137-166">個別のノードの状態を含め、バッチ クラスターを監視は、**[プール]** ウィンドウから行います。</span><span class="sxs-lookup"><span data-stu-id="50137-166">Monitor the batch cluster, including the state of individual nodes, from the **Pools** pane.</span></span>

### <a name="logging-with-doazureparallel"></a><span data-ttu-id="50137-167">doAzureParallel によるログ記録</span><span class="sxs-lookup"><span data-stu-id="50137-167">Logging with doAzureParallel</span></span>

<span data-ttu-id="50137-168">doAzureParallel パッケージは、Azure Batch で送信されたすべてのジョブのすべての stdout/stderr のログを自動的に収集します。</span><span class="sxs-lookup"><span data-stu-id="50137-168">The doAzureParallel package automatically collects logs of all stdout/stderr for every job submitted on Azure Batch.</span></span> <span data-ttu-id="50137-169">これらは、セットアップ時に作成されたストレージ アカウントにあります。</span><span class="sxs-lookup"><span data-stu-id="50137-169">These can be found in the storage account created at setup.</span></span> <span data-ttu-id="50137-170">それらを表示するには、[Azure Storage Explorer][storage-explorer] や Azure portal などのストレージ ナビゲーション ツールを使用します。</span><span class="sxs-lookup"><span data-stu-id="50137-170">To view them, use a storage navigation tool such as [Azure Storage Explorer][storage-explorer] or Azure portal.</span></span>

<span data-ttu-id="50137-171">開発中に Batch ジョブを素早くデバッグするには、doAzureParallel の [getJobFiles][getJobFiles] 関数を使用してローカル R セッションでログを表示します。</span><span class="sxs-lookup"><span data-stu-id="50137-171">To quickly debug Batch jobs during development, print logs in your local R session using the [getJobFiles][getJobFiles] function of doAzureParallel.</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="50137-172">コストに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="50137-172">Cost considerations</span></span>

<span data-ttu-id="50137-173">この参照アーキテクチャで使用されるコンピューティング リソースは、最も費用がかかるコンポーネントです。</span><span class="sxs-lookup"><span data-stu-id="50137-173">The compute resources used in this reference architecture are the most costly components.</span></span> <span data-ttu-id="50137-174">このシナリオでは、ジョブがトリガーされるたびに固定サイズのクラスターが作成され、ジョブが完了するとシャットダウンされます。</span><span class="sxs-lookup"><span data-stu-id="50137-174">For this scenario, a cluster of fixed size is created whenever the job is triggered and then shut down after the job has completed.</span></span> <span data-ttu-id="50137-175">費用が発生するのは、クラスター ノードの開始中、実行中、またはシャット ダウン中だけです。</span><span class="sxs-lookup"><span data-stu-id="50137-175">Cost is incurred only while the cluster nodes are starting, running, or shutting down.</span></span> <span data-ttu-id="50137-176">この方法は、予測を生成するために必要なコンピューティング リソースがどのジョブでも比較的一定であるシナリオに適しています。</span><span class="sxs-lookup"><span data-stu-id="50137-176">This approach is suitable for a scenario where the compute resources required to generate the forecasts remain relatively constant from job to job.</span></span>

<span data-ttu-id="50137-177">ジョブを完了するために必要なコンピューティング量が事前にわからないシナリオでは、自動スケーリングを使用する方が適している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="50137-177">In scenarios where the amount of compute required to complete the job is not known in advance, it may be more suitable to use automatic scaling.</span></span> <span data-ttu-id="50137-178">この方法では、クラスターのサイズはジョブのサイズに応じてスケールアップまたはスケールダウンされます。</span><span class="sxs-lookup"><span data-stu-id="50137-178">With this approach, the size of the cluster is scaled up or down depending on the size of the job.</span></span> <span data-ttu-id="50137-179">Azure Batch では、[doAzureParallel][doAzureParallel] API を使用してクラスターを定義するときに設定できる一連の自動スケーリング式をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="50137-179">Azure Batch supports a range of auto-scale formulae which you can set when defining the cluster using the [doAzureParallel][doAzureParallel] API.</span></span>

<span data-ttu-id="50137-180">一部のシナリオでは、ジョブ間の時間が、クラスターをシャット ダウンして起動するのに短すぎる場合があります。</span><span class="sxs-lookup"><span data-stu-id="50137-180">For some scenarios, the time between jobs may be too short to shut down and start up the cluster.</span></span> <span data-ttu-id="50137-181">このようなケースでは、適切な場合、ジョブとジョブとの間、クラスターを実行したままにしておきます。</span><span class="sxs-lookup"><span data-stu-id="50137-181">In these cases, keep the cluster running between jobs if appropriate.</span></span>

<span data-ttu-id="50137-182">Azure Batch と doAzureParallel では、優先順位の低い VM の使用がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="50137-182">Azure Batch and doAzureParallel support the use of low-priority VMs.</span></span> <span data-ttu-id="50137-183">これらの VM には大幅な割引がありますが、他の優先順位の高いワークロードによって割り当てられるリスクが伴います。</span><span class="sxs-lookup"><span data-stu-id="50137-183">These VMs come with a significant discount but risk being appropriated by other higher priority workloads.</span></span> <span data-ttu-id="50137-184">したがって、これらの VM の使用は、クリティカルな本番ワークロードには推奨されません。</span><span class="sxs-lookup"><span data-stu-id="50137-184">The use of these VMs are therefore not recommended for critical production workloads.</span></span> <span data-ttu-id="50137-185">ただし、これらは実験用ワークロードや開発用ワークロードには非常に有用です。</span><span class="sxs-lookup"><span data-stu-id="50137-185">However, they are very useful for experimental or development workloads.</span></span>

## <a name="deployment"></a><span data-ttu-id="50137-186">Deployment</span><span class="sxs-lookup"><span data-stu-id="50137-186">Deployment</span></span>

<span data-ttu-id="50137-187">この参照アーキテクチャをデプロイするには、[GitHub][github] リポジトリで説明されている手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="50137-187">To deploy this reference architecture, follow the steps described in the [GitHub][github] repo.</span></span>


[0]: ./_images/batch-scoring-r-models.png
[1]: ./_images/sales-forecasts.png
[aci]: /azure/container-instances/container-instances-overview
[autoscale]: /azure/batch/batch-automatic-scaling
[batch]: /azure/batch/batch-technical-overview
[blob]: /azure/storage/blobs/storage-blobs-introduction
[doAzureParallel]: https://github.com/Azure/doAzureParallel/blob/master/docs/32-autoscale.md
[getJobFiles]: /azure/machine-learning/service/how-to-train-ml-models
[github]: https://github.com/Azure/RBatchScoring
[logic-apps]: /azure/logic-apps/logic-apps-overview
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows