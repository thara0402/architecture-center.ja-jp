---
title: Azure Databricks での Spark モデルのバッチ スコアリング
description: Azure Databricks を使用して、スケジュールに従って Apache Spark 分類モデルをバッチ スコアリングするスケーラブルなソリューションを構築します。
author: njray
ms.date: 02/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 1b6f10edf098ed8d9fa14c16de113fc765372835
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58231414"
---
# <a name="batch-scoring-of-spark-models-on-azure-databricks"></a><span data-ttu-id="06a1a-103">Azure Databricks での Spark モデルのバッチ スコアリング</span><span class="sxs-lookup"><span data-stu-id="06a1a-103">Batch scoring of Spark models on Azure Databricks</span></span>

<span data-ttu-id="06a1a-104">この参照アーキテクチャでは、Azure 向けに最適化された Apache Spark ベースの分析プラットフォームである Azure Databricks を使用して、スケジュールに従って Apache Spark 分類モデルをバッチ スコアリングするスケーラブルなソリューションを構築する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-104">This reference architecture shows how to build a scalable solution for batch scoring an Apache Spark classification model on a schedule using Azure Databricks, an Apache Spark-based analytics platform optimized for Azure.</span></span> <span data-ttu-id="06a1a-105">このソリューションは、他のシナリオのために一般化できるテンプレートとして使用できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-105">The solution can be used as a template that can be generalized to other scenarios.</span></span>

<span data-ttu-id="06a1a-106">このアーキテクチャのリファレンス実装は、 [GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-106">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![Azure Databricks での Spark モデルのバッチ スコアリング](./_images/batch-scoring-spark.png)

<span data-ttu-id="06a1a-108">**シナリオ**: 資産が多い業界の企業が、予期しない機械の不具合に関連するコストとダウンタイムを最小化することを望んでいます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-108">**Scenario**: A business in an asset-heavy industry wants to minimize the costs and downtime associated with unexpected mechanical failures.</span></span> <span data-ttu-id="06a1a-109">機械から収集される IoT データを使用して、予知メンテナンス モデルを作成できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-109">Using IoT data collected from their machines, they can create a predictive maintenance model.</span></span> <span data-ttu-id="06a1a-110">企業は、このモデルを使用して、部品を積極的に保守し、不具合が発生する前にそれらを修繕できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-110">This model enables the business to maintain components proactively and repair them before they fail.</span></span> <span data-ttu-id="06a1a-111">機械部品を最大限使用できるようにすることで、コストを制御し、ダウンタイムを短縮できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-111">By maximizing mechanical component use, they can control costs and reduce downtime.</span></span>

<span data-ttu-id="06a1a-112">予知メンテナンス モデルでは、機械からデータが収集され、部品の不具合例の履歴が保持されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-112">A predictive maintenance model collects data from the machines and retains historical examples of component failures.</span></span> <span data-ttu-id="06a1a-113">その後、このモデルを使用して、部品の現在の状態を監視し、特定の部品で近い将来不具合が発生するかどうかを予測します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-113">The model can then be used to monitor the current state of the components and predict if a given component will fail in the near future.</span></span> <span data-ttu-id="06a1a-114">一般的なユース ケースとモデリング アプローチについては、「[予測メンテナンス ソリューションのための Azure AI ガイド][ai-guide]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="06a1a-114">For common use cases and modeling approaches, see [Azure AI guide for predictive maintenance solutions][ai-guide].</span></span>

<span data-ttu-id="06a1a-115">この参照アーキテクチャは、機械部品の新しいデータの存在によってトリガーされるワークロード用に設計されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-115">This reference architecture is designed for workloads that are triggered by the presence of new data from the component machines.</span></span> <span data-ttu-id="06a1a-116">処理には次の手順が含まれます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-116">Processing involves the following steps:</span></span>

1. <span data-ttu-id="06a1a-117">外部データ ストアから Azure Databricks データ ストアにデータを取り込みます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-117">Ingest the data from the external data store onto an Azure Databricks data store.</span></span>

2. <span data-ttu-id="06a1a-118">データをトレーニング データセットに変換した後、Spark MLlib モデルを構築して、機械学習モデルをトレーニングします。</span><span class="sxs-lookup"><span data-stu-id="06a1a-118">Train a machine learning model by transforming the data into a training data set, then building a Spark MLlib model.</span></span> <span data-ttu-id="06a1a-119">MLlib は、最も一般的な機械学習アルゴリズムと、Spark のデータ スケーラビリティ機能を活用するように最適化されたユーティリティで構成されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-119">MLlib consists of most common machine learning algorithms and utilities optimized to take advantage of Spark data scalability capabilities.</span></span>

3. <span data-ttu-id="06a1a-120">データをスコアリング データセットに変換することで、部品の不具合を予測 (分類) するトレーニング済みモデルを適用します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-120">Apply the trained model to predict (classify) component failures by transforming the data into a scoring data set.</span></span> <span data-ttu-id="06a1a-121">Spark MLLib モデルを使用してデータをスコア付けします。</span><span class="sxs-lookup"><span data-stu-id="06a1a-121">Score the data with the Spark MLLib model.</span></span>

4. <span data-ttu-id="06a1a-122">処理後の使用のために、Databricks データ ストアに結果を格納します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-122">Store results on the Databricks data store for post-processing consumption.</span></span>

<span data-ttu-id="06a1a-123">これらのタスクのそれぞれを実行するノートブックが  [GitHub][github] に提供されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-123">Notebooks are provided on [GitHub][github] to perform each of these tasks.</span></span>

## <a name="architecture"></a><span data-ttu-id="06a1a-124">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="06a1a-124">Architecture</span></span>

<span data-ttu-id="06a1a-125">アーキテクチャでは、順を追って実行される一連の[ノートブック][notebooks]に基づく [Azure Databricks][databricks] の中にその全体が含まれるデータ フローが定義されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-125">The architecture defines a data flow that is entirely contained within [Azure Databricks][databricks] based on a set of sequentially executed [notebooks][notebooks].</span></span> <span data-ttu-id="06a1a-126">これは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-126">It consists of the following components:</span></span>

<span data-ttu-id="06a1a-127">**[データ ファイル][github]**。</span><span class="sxs-lookup"><span data-stu-id="06a1a-127">**[Data files][github]**.</span></span> <span data-ttu-id="06a1a-128">参照実装では、5 つの静的なデータ ファイルに含まれているシミュレートされたデータ セットが使用されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-128">The reference implementation uses a simulated data set contained in five static data files.</span></span>

<span data-ttu-id="06a1a-129">**[取り込み][notebooks]**。</span><span class="sxs-lookup"><span data-stu-id="06a1a-129">**[Ingestion][notebooks]**.</span></span> <span data-ttu-id="06a1a-130">データ取り込みノートブックでは、入力データ ファイルを Databricks データ セットのコレクションにダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="06a1a-130">The data ingestion notebook downloads the input data files into a collection of Databricks data sets.</span></span> <span data-ttu-id="06a1a-131">現実のシナリオでは、IoT デバイスからのデータが、Databricks にアクセス可能なストレージ (Azure SQL Server や Azure Blob Storage など) にストリーミングされます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-131">In a real-world scenario, data from IoT devices would stream onto Databricks-accessible storage such as Azure SQL Server or Azure Blob storage.</span></span> <span data-ttu-id="06a1a-132">Databricks では、複数の[データソース][data-sources]がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-132">Databricks supports multiple [data sources][data-sources].</span></span>

<span data-ttu-id="06a1a-133">**トレーニング パイプライン**。</span><span class="sxs-lookup"><span data-stu-id="06a1a-133">**Training pipeline**.</span></span> <span data-ttu-id="06a1a-134">このノートブックでは、取り込まれたデータから分析データ セットを作成する特徴エンジニアリング ノートブックが実行されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-134">This notebook executes the feature engineering notebook to create an analysis data set from the ingested data.</span></span> <span data-ttu-id="06a1a-135">その後、[Apache Spark MLlib][mllib] スケーラブル機械学習ライブラリを使用して機械学習モデルをトレーニングするモデル構築ノートブックが実行されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-135">It then executes a model building notebook that trains the machine learning model using the [Apache Spark MLlib][mllib] scalable machine learning library.</span></span>

<span data-ttu-id="06a1a-136">**スコアリング パイプライン**。</span><span class="sxs-lookup"><span data-stu-id="06a1a-136">**Scoring pipeline**.</span></span> <span data-ttu-id="06a1a-137">このノートブックでは、取り込まれたデータからスコアリング データ セットを作成する特徴エンジニアリング ノートブックが実行され、スコアリング ノートブックが実行されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-137">This notebook executes the feature engineering notebook to create scoring data set from the ingested data and executes the scoring notebook.</span></span> <span data-ttu-id="06a1a-138">スコアリング ノートブックでは、トレーニング済みの [Spark MLlib][mllib-spark] モデルを使用して、スコアリング データ セットの観測結果から予測が生成されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-138">The scoring notebook uses the trained [Spark MLlib][mllib-spark] model to generate predictions for the observations in the scoring data set.</span></span> <span data-ttu-id="06a1a-139">予測は、結果ストアに格納され、Databricks データ ストアの新しいデータ セットになります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-139">The predictions are stored in the results store, a new data set on the Databricks data store.</span></span>

<span data-ttu-id="06a1a-140">**スケジューラ**。</span><span class="sxs-lookup"><span data-stu-id="06a1a-140">**Scheduler**.</span></span> <span data-ttu-id="06a1a-141">スケジュールされた Databricks [ジョブ][job]によって、Spark モデルによるバッチ スコアリングが処理されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-141">A scheduled Databricks [job][job] handles batch scoring with the Spark model.</span></span> <span data-ttu-id="06a1a-142">このジョブでは、スコアリング パイプライン ノートブックが実行され、スコアリング データ セットを構築するための詳細と結果データ セットの格納場所を指定する変数の引数が、ノートブックのパラメーターを通して渡されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-142">The job executes the scoring pipeline notebook, passing variable arguments through notebook parameters to specify the details for constructing the scoring data set and where to store the results data set.</span></span>

<span data-ttu-id="06a1a-143">このシナリオは、パイプライン フローとして構築されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-143">The scenario is constructed as a pipeline flow.</span></span> <span data-ttu-id="06a1a-144">各ノートブックは、操作 (取り込み、特徴エンジニアリング、モデルの構築、およびモデルのスコアリング) の各操作をバッチ設定で実行するように最適化されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-144">Each notebook is optimized to perform in a batch setting for each of the operations: ingestion, feature engineering, model building, and model scorings.</span></span> <span data-ttu-id="06a1a-145">これを実現するため、特徴エンジニアリング ノートブックは、トレーニング、較正、テスト、またはスコアリング操作のすべてにとって一般的なデータ セットを生成するように設計されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-145">To accomplish this, the feature engineering notebook is designed to generate a general data set for any of the training, calibration, testing, or scoring operations.</span></span> <span data-ttu-id="06a1a-146">このシナリオでは、これらの操作に対して一時分割戦略を使用するため、ノートブックのパラメーターを使用して日付範囲フィルターが設定されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-146">In this scenario, we use a temporal split strategy for these operations, so the notebook parameters are used to set date-range filtering.</span></span>

<span data-ttu-id="06a1a-147">このシナリオでは、バッチ パイプラインを作成しているため、パイプライン ノートブックの出力を調査するための一連の調査ノートブックがオプションとして用意されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-147">Because the scenario creates a batch pipeline, we provide a set of optional examination notebooks to explore the output of the pipeline notebooks.</span></span> <span data-ttu-id="06a1a-148">これらは、GitHub リポジトリで見つけることができます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-148">You can find these in the GitHub repository:</span></span>

- `1a_raw-data_exploring`
- `2a_feature_exploration`
- `2b_model_testing`
- `3b_model_scoring_evaluation`

## <a name="recommendations"></a><span data-ttu-id="06a1a-149">Recommendations</span><span class="sxs-lookup"><span data-stu-id="06a1a-149">Recommendations</span></span>

<span data-ttu-id="06a1a-150">Databricks は、お客様がご自身でトレーニングしたモデルを読み込んでデプロイし、新しいデータを使用して予測を行えるようにセットアップされています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-150">Databricks is set up so you can load and deploy your trained models to make predictions with new data.</span></span> <span data-ttu-id="06a1a-151">このシナリオで Databricks を使用したのは、次のような利点が追加されるためです。</span><span class="sxs-lookup"><span data-stu-id="06a1a-151">We used Databricks for this scenario because it provides these additional advantages:</span></span>

- <span data-ttu-id="06a1a-152">Azure Active Directory 資格情報を使用したシングル サインオンのサポート。</span><span class="sxs-lookup"><span data-stu-id="06a1a-152">Single sign-on support using Azure Active Directory credentials.</span></span>
- <span data-ttu-id="06a1a-153">運用パイプライン用のジョブを実行するジョブ スケジューラ。</span><span class="sxs-lookup"><span data-stu-id="06a1a-153">Job scheduler to execute jobs for production pipelines.</span></span>
- <span data-ttu-id="06a1a-154">コラボレーション、ダッシュボード、REST API による完全対話型ノートブック。</span><span class="sxs-lookup"><span data-stu-id="06a1a-154">Fully interactive notebook with collaboration, dashboards, REST APIs.</span></span>
- <span data-ttu-id="06a1a-155">任意のサイズにスケーリングできる制限のないクラスター。</span><span class="sxs-lookup"><span data-stu-id="06a1a-155">Unlimited clusters that can scale to any size.</span></span>
- <span data-ttu-id="06a1a-156">高度なセキュリティ、ロールベースのアクセス制御、および監査ログ。</span><span class="sxs-lookup"><span data-stu-id="06a1a-156">Advanced security, role-based access controls, and audit logs.</span></span>

<span data-ttu-id="06a1a-157">Azure Databricks サービスを操作するには、Web ブラウザーまたは[コマンド ライン インターフェイス][cli] (CLI) で Databricks [ワークスペース][workspace] インターフェイスを使用します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-157">To interact with the Azure Databricks service, use the Databricks [Workspace][workspace] interface in a web browser or the [command-line interface][cli] (CLI).</span></span> <span data-ttu-id="06a1a-158">Databricks CLI には、Python 2.7.9 から 3.6 をサポートするプラットフォームからアクセスします。</span><span class="sxs-lookup"><span data-stu-id="06a1a-158">Access the Databricks CLI from any platform that supports Python 2.7.9 to 3.6.</span></span>

<span data-ttu-id="06a1a-159">この参照実装では、[ノートブック][notebooks]を使用して、タスクを順を追って実行します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-159">The reference implementation uses [notebooks][notebooks] to execute tasks in sequence.</span></span> <span data-ttu-id="06a1a-160">各ノートブックでは、入力データと同じデータ ストアに、中間データ成果物 (トレーニング、テスト、スコアリング、または結果データ セット) が格納されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-160">Each notebook stores intermediate data artifacts (training, test, scoring, or results data sets) to the same data store as the input data.</span></span> <span data-ttu-id="06a1a-161">目標は、お客様固有のユース ケースで、必要に応じて簡単に使用できるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="06a1a-161">The goal is to make it easy for you to use it as needed in your particular use case.</span></span> <span data-ttu-id="06a1a-162">実際には、お使いのデータ ソースをノートブックの Azure Databricks インスタンスに接続して、ストレージとの読み取りと書き込みを直接的に実行します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-162">In practice, you would connect your data source to your Azure Databricks instance for the notebooks to read and write directly back into your storage.</span></span>

<span data-ttu-id="06a1a-163">必要に応じて、Databricks ユーザー インターフェイス、データ ストア、または Databricks [CLI][cli] を通してジョブの実行を監視できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-163">You can monitor job execution through the Databricks user interface, the data store, or the Databricks [CLI][cli] as necessary.</span></span> <span data-ttu-id="06a1a-164">Databricks が提供する[イベント ログ][log]やその他の[メトリック][metrics]を使用して、クラスターを監視します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-164">Monitor the cluster using the [event log][log] and other [metrics][metrics] that Databricks provides.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="06a1a-165">パフォーマンスに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="06a1a-165">Performance considerations</span></span>

<span data-ttu-id="06a1a-166">Azure Databricks クラスターは、既定で自動スケールが可能になっているため、実行時に Databricks によってお客様のジョブの特性を構成するワーカーが動的に再割り当てされます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-166">An Azure Databricks cluster enables autoscaling by default so that during runtime, Databricks dynamically reallocates workers to account for the characteristics of your job.</span></span> <span data-ttu-id="06a1a-167">パイプラインの特定の部分で、その計算負荷が他の部分よりも高くなる場合があります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-167">Certain parts of your pipeline may be more computationally demanding than others.</span></span> <span data-ttu-id="06a1a-168">Databricks では、ジョブのこれらのフェーズ中にワーカーが追加されます (不要になったときには削除されます)。</span><span class="sxs-lookup"><span data-stu-id="06a1a-168">Databricks adds additional workers during these phases of your job (and removes them when they’re no longer needed).</span></span> <span data-ttu-id="06a1a-169">ワークロードと一致するクラスターを自身でプロビジョニングする必要がないため、自動スケールによって高い[クラスター使用率][cluster]を簡単に達成できるようになります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-169">Autoscaling makes it easier to achieve high [cluster utilization][cluster], because you don’t need to provision the cluster to match a workload.</span></span>

<span data-ttu-id="06a1a-170">さらに、[Azure Data Factory][adf] と Azure Databricks を使用して、さらに複雑なスケジュールのパイプラインを開発できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-170">Additionally, more complex scheduled pipelines can be developed by using [Azure Data Factory][adf] with Azure Databricks.</span></span>

## <a name="storage-considerations"></a><span data-ttu-id="06a1a-171">ストレージに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="06a1a-171">Storage considerations</span></span>

<span data-ttu-id="06a1a-172">この参照実装では、単純化するために、データは Databricks ストレージ内に直接格納されます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-172">In this reference implementation, the data is stored directly within Databricks storage for simplicity.</span></span> <span data-ttu-id="06a1a-173">ただし、運用環境の設定では、[Azure Blob Storage][blob] などのクラウド データ ストレージにデータを格納できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-173">In a production setting, however, the data can be stored on cloud data storage such as [Azure Blob Storage][blob].</span></span> <span data-ttu-id="06a1a-174">[Databricks][databricks-connect] では、Azure Data Lake Store、Azure SQL Data Warehouse、Azure Cosmos DB、Apache Kafka、および Hadoop もサポートしています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-174">[Databricks][databricks-connect] also supports Azure Data Lake Store, Azure SQL Data Warehouse, Azure Cosmos DB, Apache Kafka, and Hadoop.</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="06a1a-175">コストに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="06a1a-175">Cost considerations</span></span>

<span data-ttu-id="06a1a-176">Azure Databricks は、優れた Spark サービスであり、関連するコストがあります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-176">Azure Databricks is a premium Spark offering with an associated cost.</span></span> <span data-ttu-id="06a1a-177">さらに、Databricks の[価格レベル][pricing]には、Standard と Premium があります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-177">In addition, there are standard and premium Databricks [pricing tiers][pricing].</span></span>

<span data-ttu-id="06a1a-178">このシナリオでは、Standard 価格レベルで十分です。</span><span class="sxs-lookup"><span data-stu-id="06a1a-178">For this scenario, the standard pricing tier is sufficient.</span></span> <span data-ttu-id="06a1a-179">ただし、特定のアプリケーションで、大規模なワークロードまたは Databricks の対話型ダッシュボードを処理するために、自動的にスケーリングされるクラスターが必要な場合は、Premium レベルの使用によってコストが増加する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="06a1a-179">However, if your specific application requires automatically scaling clusters to handle larger workloads or interactive Databricks dashboards, the premium level could increase costs further.</span></span>

<span data-ttu-id="06a1a-180">このソリューションのノートブックは、Databricks 固有のパッケージを削除するという最小限の編集によって、任意の Spark ベースのプラットフォームで実行できます。</span><span class="sxs-lookup"><span data-stu-id="06a1a-180">The solution notebooks can run on any Spark-based platform with minimal edits to remove the Databricks-specific packages.</span></span> <span data-ttu-id="06a1a-181">次のさまざまな Azure プラットフォーム向けの類似するソリューションを参照してください。</span><span class="sxs-lookup"><span data-stu-id="06a1a-181">See the following similar solutions for various Azure platforms:</span></span>

- <span data-ttu-id="06a1a-182">[Azure Machine Learning Studio 上の Python][python-aml]</span><span class="sxs-lookup"><span data-stu-id="06a1a-182">[Python on Azure Machine Learning Studio][python-aml]</span></span>
- <span data-ttu-id="06a1a-183">[SQL Server R Services][sql-r]</span><span class="sxs-lookup"><span data-stu-id="06a1a-183">[SQL Server R services][sql-r]</span></span>
- <span data-ttu-id="06a1a-184">[Azure Data Science Virtual Machine 上の PySpark][py-dvsm]</span><span class="sxs-lookup"><span data-stu-id="06a1a-184">[PySpark on an Azure Data Science Virtual Machine][py-dvsm]</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="06a1a-185">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="06a1a-185">Deploy the solution</span></span>

<span data-ttu-id="06a1a-186">この参照アーキテクチャをデプロイするには、 [GitHub][github] リポジトリで説明されている手順に従って、Azure Databricks で Spark モデルをスコアリングするためのスケーラブルなソリューションを構築します。</span><span class="sxs-lookup"><span data-stu-id="06a1a-186">To deploy this reference architecture, follow the steps described in the [GitHub][github] repository to build a scalable solution for scoring Spark models in batch on Azure Databricks.</span></span>

## <a name="related-architectures"></a><span data-ttu-id="06a1a-187">関連するアーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="06a1a-187">Related architectures</span></span>

<span data-ttu-id="06a1a-188">オフラインの事前計算済みのスコアを使用する[リアルタイム レコメンデーション システム][recommendation]を構築するために Spark を使用する参照アーキテクチャも用意されています。</span><span class="sxs-lookup"><span data-stu-id="06a1a-188">We have also built a reference architecture that uses Spark for building [real-time recommendation systems][recommendation] with offline, pre-computed scores.</span></span> <span data-ttu-id="06a1a-189">これらのレコメンデーション システムは、スコアがバッチ処理される一般的なシナリオです。</span><span class="sxs-lookup"><span data-stu-id="06a1a-189">These recommendation systems are common scenarios where scores are batch-processed.</span></span>

[adf]: https://azure.microsoft.com/blog/operationalize-azure-databricks-notebooks-using-data-factory/
[ai-guide]: /azure/machine-learning/team-data-science-process/cortana-analytics-playbook-predictive-maintenance
[blob]: https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[cli]: https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html
[cluster]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[databricks]: /azure/azure-databricks/
[databricks-connect]: /azure/azure-databricks/databricks-connect-to-data-sources
[data-sources]: https://docs.databricks.com/spark/latest/data-sources/index.html
[github]: https://github.com/Azure/BatchSparkScoringPredictiveMaintenance
[job]: https://docs.databricks.com/user-guide/jobs.html
[log]: https://docs.databricks.com/user-guide/clusters/event-log.html
[metrics]: https://docs.databricks.com/user-guide/clusters/metrics.html
[mllib]: https://docs.databricks.com/spark/latest/mllib/index.html
[mllib-spark]: https://docs.databricks.com/spark/latest/mllib/index.html#apache-spark-mllib
[notebooks]: https://docs.databricks.com/user-guide/notebooks/index.html
[pricing]: https://azure.microsoft.com/en-us/pricing/details/databricks/
[python-aml]: https://gallery.azure.ai/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1
[py-dvsm]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-using-PySpark
[recommendation]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[sql-r]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1
[workspace]: https://docs.databricks.com/user-guide/workspace.html
