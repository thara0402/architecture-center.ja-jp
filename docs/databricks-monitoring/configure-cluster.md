---
title: メトリックを Azure Monitor に送信するよう Azure Databricks を構成する
description: Azure Log Analytics でメトリックの監視とデータのログ記録を有効にするための Scala ライブラリ
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: af6b6433f87964ac60c179ecf498e54129344126
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503438"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a><span data-ttu-id="551b6-103">メトリックを Azure Monitor に送信するよう Azure Databricks を構成する</span><span class="sxs-lookup"><span data-stu-id="551b6-103">Configure Azure Databricks to send metrics to Azure Monitor</span></span>

<span data-ttu-id="551b6-104">この記事では、[Log Analytics ワークスペース](/azure/azure-monitor/platform/manage-access)にメトリックを送信するように Azure Databricks クラスターを構成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="551b6-104">This article shows how to configure an Azure Databricks cluster to send metrics to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="551b6-105">[Azure Databricks 監視ライブラリ](https://github.com/mspnp/spark-monitoring)が使用されます。これは GitHub から入手できます。</span><span class="sxs-lookup"><span data-stu-id="551b6-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span> <span data-ttu-id="551b6-106">前提条件として Java、Scala、および Maven について理解していることが推奨されます。</span><span class="sxs-lookup"><span data-stu-id="551b6-106">Understanding of Java, Scala, and Maven are recommended as prerequisistes.</span></span>

## <a name="about-the-azure-databricks-monitoring-library"></a><span data-ttu-id="551b6-107">Azure Databricks 監視ライブラリについて</span><span class="sxs-lookup"><span data-stu-id="551b6-107">About the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="551b6-108">Azure Databricks 監視ライブラリの [GitHub リポジトリ](https://github.com/mspnp/spark-monitoring)には、次のディレクトリ構造があります。</span><span class="sxs-lookup"><span data-stu-id="551b6-108">The [GitHub repo](https://github.com/mspnp/spark-monitoring) for the Azure Databricks Monitoring Library has following directory structure:</span></span>

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

<span data-ttu-id="551b6-109">**spark-jobs** ディレクトリは、Spark アプリケーションのメトリック カウンタを実装する方法を示すサンプル Spark アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="551b6-109">The **spark-jobs** directory is a sample Spark application demonstrating how to implement a Spark application metric counter.</span></span>

<span data-ttu-id="551b6-110">**spark-listeners** ディレクトリには、Azure Databricks がサービス レベルの Apache Spark イベントを Azure Log Analytics ワークスペースに送信できるようにする機能が含まれています。</span><span class="sxs-lookup"><span data-stu-id="551b6-110">The **spark-listeners** directory includes functionality that enables Azure Databricks to send Apache Spark events at the service level to an Azure Log Analytics workspace.</span></span> <span data-ttu-id="551b6-111">Azure Databricks は、Apache Spark に基づいたサービスです。これには、データセット、データフレーム、および SQL を使用してデータをバッチ処理するための構造化された API が含まれています。</span><span class="sxs-lookup"><span data-stu-id="551b6-111">Azure Databricks is a service based on Apache Spark, which includes a set of structured APIs for batch processing data using Datasets, DataFrames, and SQL.</span></span> <span data-ttu-id="551b6-112">Apache Spark 2.0 では、Spark のバッチ処理 API で構築されたデータ ストリーム処理 API である [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) のサポートが追加されました。</span><span class="sxs-lookup"><span data-stu-id="551b6-112">With Apache Spark 2.0, support was added for [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), a data stream processing API built on Spark's batch processing APIs.</span></span>

<span data-ttu-id="551b6-113">**spark-listeners-loganalytics** ディレクトリには、Spark リスナーのシンク、DropWizard のシンク、および Azure Log Analytics ワークスペースのクライアントが含まれています。</span><span class="sxs-lookup"><span data-stu-id="551b6-113">The **spark-listeners-loganalytics** directory includes a sink for Spark listeners, a sink for DropWizard, and a client for an Azure Log Analytics Workspace.</span></span> <span data-ttu-id="551b6-114">また、このディレクトリには、Apache Spark アプリケーション ログの log4j アペンダーも含まれています。</span><span class="sxs-lookup"><span data-stu-id="551b6-114">This directory also includes a log4j Appender for your Apache Spark application logs.</span></span>

<span data-ttu-id="551b6-115">**spark-listeners-loganalytics** ディレクトリと **spark-listeners** ディレクトリには、Databricks クラスターにデプロイされる 2 つの JAR ファイルを構築するためのコードが含まれています。</span><span class="sxs-lookup"><span data-stu-id="551b6-115">The **spark-listeners-loganalytics** and **spark-listeners** directories contain the code for building the two JAR files that are deployed to the Databricks cluster.</span></span> <span data-ttu-id="551b6-116">**spark-listeners** ディレクトリには、Azure Databricks ファイル システムのステージング ディレクトリから実行ノードに JAR ファイルをコピーするためのクラスター ノード初期化スクリプトを含む **scripts** ディレクトリがあります。</span><span class="sxs-lookup"><span data-stu-id="551b6-116">The **spark-listeners** directory includes a **scripts** directory that contains a cluster node initialization script to copy the JAR files from a staging directory in the Azure Databricks file system to execution nodes.</span></span>

<span data-ttu-id="551b6-117">**pom.xml** ファイルは、プロジェクト全体のメインの Maven ビルド ファイルです。</span><span class="sxs-lookup"><span data-stu-id="551b6-117">The **pom.xml** file is the main Maven build file for the entire project.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="551b6-118">前提条件</span><span class="sxs-lookup"><span data-stu-id="551b6-118">Prerequisites</span></span>

<span data-ttu-id="551b6-119">開始するには、次の Azure リソースをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="551b6-119">To get started, deploy the following Azure resources:</span></span>

- <span data-ttu-id="551b6-120">Azure Databricks ワークスペース。</span><span class="sxs-lookup"><span data-stu-id="551b6-120">An Azure Databricks workspace.</span></span> <span data-ttu-id="551b6-121">「[クイック スタート:Azure portal を使用して Azure Databricks 上で Spark ジョブを実行する](/azure/azure-databricks/quickstart-create-databricks-workspace-portal)。</span><span class="sxs-lookup"><span data-stu-id="551b6-121">See [Quickstart: Run a Spark job on Azure Databricks using the Azure portal](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span></span>
- <span data-ttu-id="551b6-122">Log Analytics ワークスペース。</span><span class="sxs-lookup"><span data-stu-id="551b6-122">A Log Analytics workspace.</span></span> <span data-ttu-id="551b6-123">「[Azure portal で Log Analytics ワークスペースを作成する](/azure/azure-monitor/learn/quick-create-workspace)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="551b6-123">See [Create a Log Analytics workspace in the Azure portal](/azure/azure-monitor/learn/quick-create-workspace).</span></span>

<span data-ttu-id="551b6-124">次に、[Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="551b6-124">Next, install the [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span></span> <span data-ttu-id="551b6-125">CLI を使用するには、Azure Databricks の個人用アクセス トークンが必要です。</span><span class="sxs-lookup"><span data-stu-id="551b6-125">An Azure Databricks personal access token is required to use the CLI.</span></span> <span data-ttu-id="551b6-126">手順については、「[トークンを生成する](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="551b6-126">For instructions, see [Generate a token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span></span>

<span data-ttu-id="551b6-127">Log Analytics ワークスペースの ID とキーを見つけます。</span><span class="sxs-lookup"><span data-stu-id="551b6-127">Find your Log Analytics workspace ID and key.</span></span> <span data-ttu-id="551b6-128">手順については、「[ワークスペース ID とキーを取得する](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="551b6-128">For instructions, see [Obtain workspace ID and key](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span></span> <span data-ttu-id="551b6-129">これらの値は、Databricks クラスターを構成するときに必要になります。</span><span class="sxs-lookup"><span data-stu-id="551b6-129">You will need these values when you configure the Databricks cluster.</span></span>

<span data-ttu-id="551b6-130">監視ライブラリをビルドするには、次のリソースを含む Java IDE が必要になります。</span><span class="sxs-lookup"><span data-stu-id="551b6-130">To build the monitoring library, you will need a Java IDE with the following resources:</span></span>

- [<span data-ttu-id="551b6-131">Java Development Kit (JDK) バージョン 1.8</span><span class="sxs-lookup"><span data-stu-id="551b6-131">Java Development Kit (JDK) version 1.8</span></span>](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [<span data-ttu-id="551b6-132">Scala 言語 SDK 2.11</span><span class="sxs-lookup"><span data-stu-id="551b6-132">Scala language SDK 2.11</span></span>](https://www.scala-lang.org/download/)
- [<span data-ttu-id="551b6-133">Apache Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="551b6-133">Apache Maven 3.5.4</span></span>](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a><span data-ttu-id="551b6-134">Azure Databricks 監視ライブラリをビルドする</span><span class="sxs-lookup"><span data-stu-id="551b6-134">Build the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="551b6-135">Azure Databricks 監視ライブラリをビルドするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="551b6-135">To build the Azure Databricks Monitoring Library, perform the following steps:</span></span>

1. <span data-ttu-id="551b6-136">[GitHub リポジトリ](https://github.com/mspnp/spark-monitoring)を複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="551b6-136">Clone, fork, or download the [GitHub repository](https://github.com/mspnp/spark-monitoring).</span></span>

1. <span data-ttu-id="551b6-137">**/src** フォルダーにある Maven プロジェクト オブジェクト モデル ファイル _pom.xml_ をプロジェクトにインポートします。</span><span class="sxs-lookup"><span data-stu-id="551b6-137">Import the Maven project object model file, _pom.xml_, located in the **/src** folder into your project.</span></span> <span data-ttu-id="551b6-138">これにより、次の 3 つのプロジェクトがインポートされます。</span><span class="sxs-lookup"><span data-stu-id="551b6-138">This will import three projects:</span></span>

    - <span data-ttu-id="551b6-139">spark-jobs</span><span class="sxs-lookup"><span data-stu-id="551b6-139">spark-jobs</span></span>
    - <span data-ttu-id="551b6-140">spark-listeners</span><span class="sxs-lookup"><span data-stu-id="551b6-140">spark-listeners</span></span>
    - <span data-ttu-id="551b6-141">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="551b6-141">spark-listeners-loganalytics</span></span>

1. <span data-ttu-id="551b6-142">Java IDE で Maven の**パッケージ** ビルド フェーズを実行して、次の各プロジェクトの JAR ファイルをビルドします。</span><span class="sxs-lookup"><span data-stu-id="551b6-142">Execute the Maven **package** build phase in your Java IDE to build the JAR files for each of these projects:</span></span>

    |<span data-ttu-id="551b6-143">Project</span><span class="sxs-lookup"><span data-stu-id="551b6-143">Project</span></span>| <span data-ttu-id="551b6-144">JAR ファイル</span><span class="sxs-lookup"><span data-stu-id="551b6-144">JAR file</span></span>|
    |-------|---------|
    |<span data-ttu-id="551b6-145">spark-jobs</span><span class="sxs-lookup"><span data-stu-id="551b6-145">spark-jobs</span></span>|<span data-ttu-id="551b6-146">spark-jobs-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="551b6-146">spark-jobs-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="551b6-147">spark-listeners</span><span class="sxs-lookup"><span data-stu-id="551b6-147">spark-listeners</span></span>|<span data-ttu-id="551b6-148">spark-listeners-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="551b6-148">spark-listeners-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="551b6-149">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="551b6-149">spark-listeners-loganalytics</span></span>|<span data-ttu-id="551b6-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="551b6-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span></span>|

1. <span data-ttu-id="551b6-151">Azure Databricks CLI を使用して、**dbfs:/databricks/monitoring-staging** という名前のディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="551b6-151">Use the Azure Databricks CLI to create a directory named **dbfs:/databricks/monitoring-staging**:</span></span>  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. <span data-ttu-id="551b6-152">Azure Databricks CLI を使用して **/src/spark-listeners/scripts/listeners.sh** を前に作成したディレクトリにコピーします。</span><span class="sxs-lookup"><span data-stu-id="551b6-152">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/listeners.sh** to the directory previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. <span data-ttu-id="551b6-153">Azure Databricks CLI を使用して **/src/spark-listeners/scripts/metrics.properties** を前に作成したディレクトリにコピーします。</span><span class="sxs-lookup"><span data-stu-id="551b6-153">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/metrics.properties** to the directory created previously:</span></span>

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. <span data-ttu-id="551b6-154">Azure Databricks CLI を使用して **spark-listeners-1.0-SNAPSHOT.jar** と **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** を前に作成したディレクトリにコピーします。</span><span class="sxs-lookup"><span data-stu-id="551b6-154">Use the Azure Databricks CLI to copy **spark-listeners-1.0-SNAPSHOT.jar** and **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** to the directory created previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a><span data-ttu-id="551b6-155">Azure Databricks クラスターを作成および構成する</span><span class="sxs-lookup"><span data-stu-id="551b6-155">Create and configure an Azure Databricks cluster</span></span>

<span data-ttu-id="551b6-156">Azure Databricks クラスターを作成および構成するには、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="551b6-156">To create and configure an Azure Databricks cluster, follow these steps:</span></span>

1. <span data-ttu-id="551b6-157">Azure portal で Azure Databricks ワークスペースに移動します。</span><span class="sxs-lookup"><span data-stu-id="551b6-157">Navigate to your Azure Databricks workspace in the Azure portal.</span></span>
1. <span data-ttu-id="551b6-158">ホーム ページで、**[クラスターの作成]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="551b6-158">On the home page, click **New Cluster**.</span></span>
1. <span data-ttu-id="551b6-159">クラスターの名前を **[クラスター名]** テキスト ボックスに入力します。</span><span class="sxs-lookup"><span data-stu-id="551b6-159">Enter a name for your cluster in the **Cluster Name** text box.</span></span>
1. <span data-ttu-id="551b6-160">**[Databricks Runtime のバージョン]** ドロップダウンで、**4.3** 以上を選択します。</span><span class="sxs-lookup"><span data-stu-id="551b6-160">In the **Databricks Runtime Version** dropdown, select **4.3** or greater.</span></span>
1. <span data-ttu-id="551b6-161">**[詳細オプション]** で、**[Spark]** タブをクリックします。**[Spark の構成]** テキスト ボックスに次の名前と値の組を入力します。</span><span class="sxs-lookup"><span data-stu-id="551b6-161">Under **Advanced Options**, click on the **Spark** tab. Enter the following name-value pairs in the **Spark Config** text box:</span></span>

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. <span data-ttu-id="551b6-162">引き続き **[Spark]** タブで、**[環境変数]** テキスト ボックスに次の値を入力します。</span><span class="sxs-lookup"><span data-stu-id="551b6-162">While still under the **Spark** tab, enter the following values in the **Environment Variables** text box:</span></span>

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Databricks UI のスクリーン ショット](./_images/create-cluster1.png)

1. <span data-ttu-id="551b6-164">引き続き **[詳細オプション]** セクションで、**[初期スクリプト]** タブをクリックします。</span><span class="sxs-lookup"><span data-stu-id="551b6-164">While still under the **Advanced Options** section, click on the **Init Scripts** tab.</span></span>
1. <span data-ttu-id="551b6-165">**[宛先]** ドロップダウンで、**[DBFS]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="551b6-165">In the **Destination** dropdown, select **DBFS**.</span></span>
1. <span data-ttu-id="551b6-166">**[初期スクリプトのパス]** で、`dbfs:/databricks/monitoring-staging/listeners.sh` と入力します。</span><span class="sxs-lookup"><span data-stu-id="551b6-166">Under **Init Script Path**, enter `dbfs:/databricks/monitoring-staging/listeners.sh`.</span></span>
1. <span data-ttu-id="551b6-167">**[追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="551b6-167">Click **Add**.</span></span>

    ![Databricks UI のスクリーン ショット](./_images/create-cluster2.png)

1. <span data-ttu-id="551b6-169">**[クラスターの作成]** をクリックしてクラスターを作成します。</span><span class="sxs-lookup"><span data-stu-id="551b6-169">Click **Create Cluster** to create the cluster.</span></span>

## <a name="view-metrics"></a><span data-ttu-id="551b6-170">メトリックを表示する</span><span class="sxs-lookup"><span data-stu-id="551b6-170">View metrics</span></span>

<span data-ttu-id="551b6-171">これらの手順を完了すると、Databricks クラスターは、クラスター自体に関するいくつかのメトリック データを Azure Monitor にストリーミングします。</span><span class="sxs-lookup"><span data-stu-id="551b6-171">After you complete these steps, your Databricks cluster streams some metric data about the cluster itself to Azure Monitor.</span></span> <span data-ttu-id="551b6-172">このログ データは、Azure Log Analytics ワークスペースの "アクティブ |カスタム ログ | SparkMetric_CL" スキーマにあります。</span><span class="sxs-lookup"><span data-stu-id="551b6-172">This log data is available in your Azure Log Analytics workspace under the "Active | Custom Logs | SparkMetric_CL" schema.</span></span> <span data-ttu-id="551b6-173">ログに記録されるメトリックの種類の詳細については、Dropwizard ドキュメントの「[Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="551b6-173">For more information about the types of metrics that are logged, see [Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) in the Dropwizard documentation.</span></span> <span data-ttu-id="551b6-174">ゲージ、カウンター、またはヒストグラムなどのメトリックの種類は、[種類] フィールドに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="551b6-174">The metric type, such as gauge, counter, or histogram, is written to the Type field.</span></span>

<span data-ttu-id="551b6-175">さらに、ライブラリは、Apache Spark レベル イベントと Spark Structured Streaming メトリックをジョブから Azure Monitor にストリーミングします。</span><span class="sxs-lookup"><span data-stu-id="551b6-175">In addition, the library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="551b6-176">これらのイベントとメトリックのアプリケーション コードに変更を加える必要はありません。</span><span class="sxs-lookup"><span data-stu-id="551b6-176">You don't need to make any changes to your application code for these events and metrics.</span></span> <span data-ttu-id="551b6-177">Spark Structured Streaming のログ データは、"アクティブ | カスタム ログ | SparkListenerEvent_CL" スキーマにあります。</span><span class="sxs-lookup"><span data-stu-id="551b6-177">Spark Structured Streaming log data is available under the "Active | Custom Logs | SparkListenerEvent_CL" schema.</span></span>

![Log Analytics ワークスペースのスクリーンショット](./_images/workspace.png)

<span data-ttu-id="551b6-179">ログの表示方法の詳細については、「[Azure Monitor におけるログ データの表示と分析](/azure/azure-monitor/log-query/portals)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="551b6-179">For more information about viewing logs, see [Viewing and analyzing log data in Azure Monitor](/azure/azure-monitor/log-query/portals).</span></span>

## <a name="next-steps"></a><span data-ttu-id="551b6-180">次の手順</span><span class="sxs-lookup"><span data-stu-id="551b6-180">Next steps</span></span>

<span data-ttu-id="551b6-181">この記事では、Azure Monitor にメトリックを送信するようにクラスターを構成する方法について説明しました。</span><span class="sxs-lookup"><span data-stu-id="551b6-181">This article described how to configure your cluster to send metrics to Azure Monitor.</span></span> <span data-ttu-id="551b6-182">次の記事では、Azure Databricks の監視ライブラリを使用してアプリケーションのメトリックとログを送信する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="551b6-182">The next article shows how to use the Azure Databricks Monitoring Library to send application metrics and logs.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="551b6-183">アプリケーション ログを Azure Monitor に送信する</span><span class="sxs-lookup"><span data-stu-id="551b6-183">Send application logs to Azure Monitor</span></span>](./application-logs.md)
