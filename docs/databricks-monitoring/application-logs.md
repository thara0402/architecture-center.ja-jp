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
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a>Azure Databricks アプリケーション ログを Azure Monitor に送信する

この記事では、アプリケーション ログとアプリケーション メトリックを Azure Databricks から [Log Analytics ワークスペース](/azure/azure-monitor/platform/manage-access)送信する方法について説明します。 [Azure Databricks 監視ライブラリ](https://github.com/mspnp/spark-monitoring)が使用されます。これは GitHub から入手できます。

## <a name="prerequisites"></a>前提条件

「[メトリックを Azure Monitor に送信するよう Azure Databricks を構成する][config-cluster]」の説明に従って、監視ライブラリを使用するように Azure Databricks クラスターを構成します。

> [!NOTE]
> 監視ライブラリは、Apache Spark レベル イベントと Spark Structured Streaming メトリックをジョブから Azure Monitor にストリーミングします。 これらのイベントとメトリックのアプリケーション コードに変更を加える必要はありません。

## <a name="send-application-metrics-using-dropwizard"></a>Dropwizard を使用してアプリケーション メトリックを送信する

Spark は、Dropwizard メトリック ライブラリに基づいて構成可能なメトリック システムを使用します。 詳細については、Spark ドキュメントの「[Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics)」を参照してください。

アプリケーション メトリックを Azure Databricks のアプリケーション コードから Azure Monitor に送信するには、次の手順を実行します。

1. 「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、**spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR ファイルをビルドします。

1. アプリケーション コードに Dropwizard の[ゲージまたはカウンター](https://metrics.dropwizard.io/4.0.0/manual/core.html)を作成します。 監視ライブラリに定義されている `UserMetricsSystem` クラスを使用できます。 次の例は、`counter1` という名前の VM を作成します。

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

    監視のライブラリには、`UserMetricsSystem`クラスの使用方法を示す[サンプル アプリケーション][sample-app]が含まれています。

## <a name="send-application-logs-using-log4j"></a>Log4j を使用してアプリケーション ログを送信する

ライブラリ内の [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) を使用して Azure Databricks のアプリケーション ログを Azure Log Analytics に送信するには、以下の手順を実行します。

1. 「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、**spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR ファイルをビルドします。

1. アプリケーション用の **log4j.properties** [構成ファイル](https://logging.apache.org/log4j/2.x/manual/configuration.html)を作成します。 次の構成プロパティを含めます。 示されている場所のアプリケーション パッケージ名とログ レベルを置き換えます。

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    サンプル構成ファイルは[こちら][log4j.properties]にあります。

1. アプリケーション コードに **spark-listeners-loganalytics** プロジェクトを含め、`com.microsoft.pnp.logging.Log4jconfiguration` をアプリケーション コードにインポートします。

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. ステップ 3 で作成した **log4j.properties** ファイルを使用して Log4j を構成します。

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. 必要に応じて、コード内の適切なレベルで Apache Spark ログ メッセージを追加します。 たとえば、`logDebug` メソッドを使用してデバッグ ログ メッセージを送信します。 詳細については、Spark ドキュメントの「[Logging][spark-logging]」を参照してください。

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a>サンプル アプリケーションの実行

監視ライブラリには、アプリケーション メトリックとアプリケーション ログの両方を Azure Monitor に送信する方法を示す[サンプル アプリケーション][sample-app]が含まれています。 サンプルを実行するには:

1. 「[Azure Databricks 監視ライブラリをビルドする][build-lib]」の説明に従って、監視ライブラリに.**spark-jobs** プロジェクトをビルドします。

1. [こちら](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job)の説明に従って、Databricks ワークスペースに移動して新しいジョブを作成します。

1. [ジョブの詳細] ページで **[JAR の設定]** を選択します。

1. `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar` から JAR ファイルをアップロードします。

1. **[Main クラス]** には `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob` を入力します。

1. 既に監視ライブラリを使用するように構成されているクラスターを選択します。 「[メトリックを Azure Monitor に送信するよう Azure Databricks を構成する][config-cluster]」を参照してください。

ジョブが実行されると、Log Analytics ワークスペースでアプリケーション ログとアプリケーション メトリックを表示できます。

アプリケーション ログは、SparkLoggingEvent_CL の下に表示されます。

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

アプリケーション メトリックは、SparkMetric_CL の下に表示されます。

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a>次の手順

このコード ライブラリに付属するパフォーマンス監視ダッシュボードをデプロイして、実稼働 Azure Databricks ワークロードのパフォーマンス問題をトラブルシューティングする。

> [!div class="nextstepaction"]
> [ダッシュボードを使用して Azure Databricks のメトリックを視覚化する](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html