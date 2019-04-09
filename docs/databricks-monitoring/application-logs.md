---
title: Azure Databricks アプリケーション ログを Azure Monitor に送信する
description: カスタム ログとメトリックを Azure Databricks から Azure Monitor に送信する方法
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 49c631687fb3e3bbd807ffbbb49d9c5f6526bfb4
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503435"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a><span data-ttu-id="d8fc3-103">Azure Databricks アプリケーション ログを Azure Monitor に送信する</span><span class="sxs-lookup"><span data-stu-id="d8fc3-103">Send Azure Databricks application logs to Azure Monitor</span></span>

<span data-ttu-id="d8fc3-104">この記事では、アプリケーション ログとアプリケーション メトリックを Azure Databricks から [Log Analytics ワークスペース](/azure/azure-monitor/platform/manage-access)送信する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-104">This article shows how to send application logs and metrics from Azure Databricks to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="d8fc3-105">[Azure Databricks 監視ライブラリ](https://github.com/mspnp/spark-monitoring)が使用されます。これは GitHub から入手できます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d8fc3-106">前提条件</span><span class="sxs-lookup"><span data-stu-id="d8fc3-106">Prerequisites</span></span>

<span data-ttu-id="d8fc3-107">「[メトリックを Azure Monitor に送信するよう Azure Databricks を構成する][config-cluster]」の説明に従って、監視ライブラリを使用するように Azure Databricks クラスターを構成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-107">Configure your Azure Databricks cluster to use the monitoring library, as described in [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

> [!NOTE]
> <span data-ttu-id="d8fc3-108">監視ライブラリは、Apache Spark レベル イベントと Spark Structured Streaming メトリックをジョブから Azure Monitor にストリーミングします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-108">The monitoring library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="d8fc3-109">これらのイベントとメトリックのアプリケーション コードに変更を加える必要はありません。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-109">You don't need to make any changes to your application code for these events and metrics.</span></span>

## <a name="send-application-metrics-using-dropwizard"></a><span data-ttu-id="d8fc3-110">Dropwizard を使用してアプリケーション メトリックを送信する</span><span class="sxs-lookup"><span data-stu-id="d8fc3-110">Send application metrics using Dropwizard</span></span>

<span data-ttu-id="d8fc3-111">Spark は、Dropwizard メトリック ライブラリに基づいて構成可能なメトリック システムを使用します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-111">Spark uses a configurable metrics system based on the Dropwizard Metrics Library.</span></span> <span data-ttu-id="d8fc3-112">詳細については、Spark ドキュメントの「[Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-112">For more information, see [Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics) in the Spark documentation.</span></span>

<span data-ttu-id="d8fc3-113">アプリケーション メトリックを Azure Databricks のアプリケーション コードから Azure Monitor に送信するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-113">To send application metrics from Azure Databricks application code to Azure Monitor, follow these steps:</span></span>

1. <span data-ttu-id="d8fc3-114">「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、**spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR ファイルをビルドします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-114">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="d8fc3-115">アプリケーション コードに Dropwizard の[ゲージまたはカウンター](https://metrics.dropwizard.io/4.0.0/manual/core.html)を作成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-115">Create Dropwizard [gauges or counters](https://metrics.dropwizard.io/4.0.0/manual/core.html) in your application code.</span></span> <span data-ttu-id="d8fc3-116">監視ライブラリに定義されている `UserMetricsSystem` クラスを使用できます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-116">You can use the `UserMetricsSystem` class defined in the monitoring library.</span></span> <span data-ttu-id="d8fc3-117">次の例は、`counter1` という名前の VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-117">The following example creates a counter named `counter1`.</span></span>

    ```Scala
    import org.apache.spark.metrics.UserMetricsSystems
    import org.apache.spark.sql.SparkSession

    object StreamingQueryListenerSampleJob  {

      private final val METRICS_NAMESPACE = "samplejob"
      private final val COUNTER_NAME = "counter1"

      def main(args: Array[String]): Unit = {

        val spark = SparkSession
          .builder
          .getOrCreate

        val driverMetricsSystem = UserMetricsSystems
            .getMetricSystem(METRICS_NAMESPACE, builder => {
              builder.registerCounter(COUNTER_NAME)
            })

        driverMetricsSystem.counter(COUNTER_NAME).inc(5)
      }
    }
    ```

    <span data-ttu-id="d8fc3-118">監視のライブラリには、`UserMetricsSystem`クラスの使用方法を示す[サンプル アプリケーション][sample-app]が含まれています。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-118">The monitoring library includes a [sample application][sample-app] that demonstrates how to use the `UserMetricsSystem` class.</span></span>

## <a name="send-application-logs-using-log4j"></a><span data-ttu-id="d8fc3-119">Log4j を使用してアプリケーション ログを送信する</span><span class="sxs-lookup"><span data-stu-id="d8fc3-119">Send application logs using Log4j</span></span>

<span data-ttu-id="d8fc3-120">ライブラリ内の [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) を使用して Azure Databricks のアプリケーション ログを Azure Log Analytics に送信するには、以下の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-120">To send your Azure Databricks application logs to Azure Log Analytics using the [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) in the library, follow these steps:</span></span>

1. <span data-ttu-id="d8fc3-121">「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、**spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR ファイルをビルドします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-121">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="d8fc3-122">アプリケーション用の **log4j.properties** [構成ファイル](https://logging.apache.org/log4j/2.x/manual/configuration.html)を作成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-122">Create a **log4j.properties** [configuration file](https://logging.apache.org/log4j/2.x/manual/configuration.html) for your application.</span></span> <span data-ttu-id="d8fc3-123">次の構成プロパティを含めます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-123">Include the following configuration properties.</span></span> <span data-ttu-id="d8fc3-124">示されている場所のアプリケーション パッケージ名とログ レベルを置き換えます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-124">Substitute your application package name and log level where indicated:</span></span>

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    <span data-ttu-id="d8fc3-125">サンプル構成ファイルは[こちら][log4j.properties]にあります。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-125">You can find a sample configuration file [here][log4j.properties].</span></span>

1. <span data-ttu-id="d8fc3-126">アプリケーション コードに **spark-listeners-loganalytics** プロジェクトを含め、`com.microsoft.pnp.logging.Log4jconfiguration` をアプリケーション コードにインポートします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-126">In your application code, include the **spark-listeners-loganalytics** project, and import `com.microsoft.pnp.logging.Log4jconfiguration` to your application code.</span></span>

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. <span data-ttu-id="d8fc3-127">ステップ 3 で作成した **log4j.properties** ファイルを使用して Log4j を構成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-127">Configure Log4j using the **log4j.properties** file you created in step 3:</span></span>

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. <span data-ttu-id="d8fc3-128">必要に応じて、コード内の適切なレベルで Apache Spark ログ メッセージを追加します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-128">Add Apache Spark log messages at the appropriate level in your code as required.</span></span> <span data-ttu-id="d8fc3-129">たとえば、`logDebug` メソッドを使用してデバッグ ログ メッセージを送信します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-129">For example, use the `logDebug` method to send a debug log meesage.</span></span> <span data-ttu-id="d8fc3-130">詳細については、Spark ドキュメントの「[Logging][spark-logging]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-130">For more information, see [Logging][spark-logging] in the Spark documentation.</span></span>

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a><span data-ttu-id="d8fc3-131">サンプル アプリケーションの実行</span><span class="sxs-lookup"><span data-stu-id="d8fc3-131">Run the sample application</span></span>

<span data-ttu-id="d8fc3-132">監視ライブラリには、アプリケーション メトリックとアプリケーション ログの両方を Azure Monitor に送信する方法を示す[サンプル アプリケーション][sample-app]が含まれています。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-132">The monitoring library includes a [sample application][sample-app] that demonstrates how to send both application metrics and application logs to Azure Monitor.</span></span> <span data-ttu-id="d8fc3-133">サンプルを実行するには:</span><span class="sxs-lookup"><span data-stu-id="d8fc3-133">To run the sample:</span></span>

1. <span data-ttu-id="d8fc3-134">「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、監視ライブラリに.**spark-jobs** プロジェクトをビルドします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-134">Build the **spark-jobs** project in the monitoring library, as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="d8fc3-135">[こちら](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job)の説明に従って、Databricks ワークスペースに移動して新しいジョブを作成します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-135">Navigate to your Databricks workspace and create a new job, as described [here](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span></span>

1. <span data-ttu-id="d8fc3-136">[ジョブの詳細] ページで **[JAR の設定]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-136">In the job detail page, select **Set JAR**.</span></span>

1. <span data-ttu-id="d8fc3-137">`/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar` から JAR ファイルをアップロードします。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-137">Upload the JAR file from `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span></span>

1. <span data-ttu-id="d8fc3-138">**[Main クラス]** には `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob` を入力します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-138">For **Main class**, enter `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span></span>

1. <span data-ttu-id="d8fc3-139">既に監視ライブラリを使用するように構成されているクラスターを選択します。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-139">Select a cluster that is already configured to use the monitoring library.</span></span> <span data-ttu-id="d8fc3-140">「[メトリックを Azure Monitor に送信するよう Azure Databricks を構成する][config-cluster]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-140">See [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

<span data-ttu-id="d8fc3-141">ジョブが実行されると、Log Analytics ワークスペースでアプリケーション ログとアプリケーション メトリックを表示できます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-141">When the job runs, you can view the application logs and metrics in your Log Analytics workspace.</span></span>

<span data-ttu-id="d8fc3-142">アプリケーション ログは、SparkLoggingEvent_CL の下に表示されます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-142">Application logs appear under SparkLoggingEvent_CL:</span></span>

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

<span data-ttu-id="d8fc3-143">アプリケーション メトリックは、SparkMetric_CL の下に表示されます。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-143">Application metrics appear under SparkMetric_CL:</span></span>

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a><span data-ttu-id="d8fc3-144">次の手順</span><span class="sxs-lookup"><span data-stu-id="d8fc3-144">Next steps</span></span>

<span data-ttu-id="d8fc3-145">このコード ライブラリに付属するパフォーマンス監視ダッシュボードをデプロイして、実稼働 Azure Databricks ワークロードのパフォーマンス問題をトラブルシューティングする。</span><span class="sxs-lookup"><span data-stu-id="d8fc3-145">Deploy the performance monitoring dashboard that accompanies this code library to troubleshoot performance issues in your production Azure Databricks workloads.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d8fc3-146">ダッシュボードを使用して Azure Databricks のメトリックを視覚化する</span><span class="sxs-lookup"><span data-stu-id="d8fc3-146">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html