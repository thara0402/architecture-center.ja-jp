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

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a>メトリックを Azure Monitor に送信するよう Azure Databricks を構成する

この記事では、[Log Analytics ワークスペース](/azure/azure-monitor/platform/manage-access)にメトリックを送信するように Azure Databricks クラスターを構成する方法について説明します。 [Azure Databricks 監視ライブラリ](https://github.com/mspnp/spark-monitoring)が使用されます。これは GitHub から入手できます。 前提条件として Java、Scala、および Maven について理解していることが推奨されます。

## <a name="about-the-azure-databricks-monitoring-library"></a>Azure Databricks 監視ライブラリについて

Azure Databricks 監視ライブラリの [GitHub リポジトリ](https://github.com/mspnp/spark-monitoring)には、次のディレクトリ構造があります。

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

**spark-jobs** ディレクトリは、Spark アプリケーションのメトリック カウンタを実装する方法を示すサンプル Spark アプリケーションです。

**spark-listeners** ディレクトリには、Azure Databricks がサービス レベルの Apache Spark イベントを Azure Log Analytics ワークスペースに送信できるようにする機能が含まれています。 Azure Databricks は、Apache Spark に基づいたサービスです。これには、データセット、データフレーム、および SQL を使用してデータをバッチ処理するための構造化された API が含まれています。 Apache Spark 2.0 では、Spark のバッチ処理 API で構築されたデータ ストリーム処理 API である [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) のサポートが追加されました。

**spark-listeners-loganalytics** ディレクトリには、Spark リスナーのシンク、DropWizard のシンク、および Azure Log Analytics ワークスペースのクライアントが含まれています。 また、このディレクトリには、Apache Spark アプリケーション ログの log4j アペンダーも含まれています。

**spark-listeners-loganalytics** ディレクトリと **spark-listeners** ディレクトリには、Databricks クラスターにデプロイされる 2 つの JAR ファイルを構築するためのコードが含まれています。 **spark-listeners** ディレクトリには、Azure Databricks ファイル システムのステージング ディレクトリから実行ノードに JAR ファイルをコピーするためのクラスター ノード初期化スクリプトを含む **scripts** ディレクトリがあります。

**pom.xml** ファイルは、プロジェクト全体のメインの Maven ビルド ファイルです。

## <a name="prerequisites"></a>前提条件

開始するには、次の Azure リソースをデプロイします。

- Azure Databricks ワークスペース。 「[クイック スタート:Azure portal を使用して Azure Databricks 上で Spark ジョブを実行する](/azure/azure-databricks/quickstart-create-databricks-workspace-portal)。
- Log Analytics ワークスペース。 「[Azure portal で Log Analytics ワークスペースを作成する](/azure/azure-monitor/learn/quick-create-workspace)」を参照してください。

次に、[Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli) をインストールします。 CLI を使用するには、Azure Databricks の個人用アクセス トークンが必要です。 手順については、「[トークンを生成する](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management)」を参照してください。

Log Analytics ワークスペースの ID とキーを見つけます。 手順については、「[ワークスペース ID とキーを取得する](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key)」を参照してください。 これらの値は、Databricks クラスターを構成するときに必要になります。

監視ライブラリをビルドするには、次のリソースを含む Java IDE が必要になります。

- [Java Development Kit (JDK) バージョン 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Scala 言語 SDK 2.11](https://www.scala-lang.org/download/)
- [Apache Maven 3.5.4](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a>Azure Databricks 監視ライブラリをビルドする

Azure Databricks 監視ライブラリをビルドするには、次の手順を実行します。

1. [GitHub リポジトリ](https://github.com/mspnp/spark-monitoring)を複製、フォーク、またはダウンロードします。

1. **/src** フォルダーにある Maven プロジェクト オブジェクト モデル ファイル _pom.xml_ をプロジェクトにインポートします。 これにより、次の 3 つのプロジェクトがインポートされます。

    - spark-jobs
    - spark-listeners
    - spark-listeners-loganalytics

1. Java IDE で Maven の**パッケージ** ビルド フェーズを実行して、次の各プロジェクトの JAR ファイルをビルドします。

    |Project| JAR ファイル|
    |-------|---------|
    |spark-jobs|spark-jobs-1.0-SNAPSHOT.jar|
    |spark-listeners|spark-listeners-1.0-SNAPSHOT.jar|
    |spark-listeners-loganalytics|spark-listeners-loganalytics-1.0-SNAPSHOT.jar|

1. Azure Databricks CLI を使用して、**dbfs:/databricks/monitoring-staging** という名前のディレクトリを作成します。  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. Azure Databricks CLI を使用して **/src/spark-listeners/scripts/listeners.sh** を前に作成したディレクトリにコピーします。

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. Azure Databricks CLI を使用して **/src/spark-listeners/scripts/metrics.properties** を前に作成したディレクトリにコピーします。

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. Azure Databricks CLI を使用して **spark-listeners-1.0-SNAPSHOT.jar** と **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** を前に作成したディレクトリにコピーします。

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a>Azure Databricks クラスターを作成および構成する

Azure Databricks クラスターを作成および構成するには、次の手順に従います。

1. Azure portal で Azure Databricks ワークスペースに移動します。
1. ホーム ページで、**[クラスターの作成]** をクリックします。
1. クラスターの名前を **[クラスター名]** テキスト ボックスに入力します。
1. **[Databricks Runtime のバージョン]** ドロップダウンで、**4.3** 以上を選択します。
1. **[詳細オプション]** で、**[Spark]** タブをクリックします。**[Spark の構成]** テキスト ボックスに次の名前と値の組を入力します。

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. 引き続き **[Spark]** タブで、**[環境変数]** テキスト ボックスに次の値を入力します。

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Databricks UI のスクリーン ショット](./_images/create-cluster1.png)

1. 引き続き **[詳細オプション]** セクションで、**[初期スクリプト]** タブをクリックします。
1. **[宛先]** ドロップダウンで、**[DBFS]** を選択します。
1. **[初期スクリプトのパス]** で、`dbfs:/databricks/monitoring-staging/listeners.sh` と入力します。
1. **[追加]** をクリックします。

    ![Databricks UI のスクリーン ショット](./_images/create-cluster2.png)

1. **[クラスターの作成]** をクリックしてクラスターを作成します。

## <a name="view-metrics"></a>メトリックを表示する

これらの手順を完了すると、Databricks クラスターは、クラスター自体に関するいくつかのメトリック データを Azure Monitor にストリーミングします。 このログ データは、Azure Log Analytics ワークスペースの "アクティブ |カスタム ログ | SparkMetric_CL" スキーマにあります。 ログに記録されるメトリックの種類の詳細については、Dropwizard ドキュメントの「[Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html)」を参照してください。 ゲージ、カウンター、またはヒストグラムなどのメトリックの種類は、[種類] フィールドに書き込まれます。

さらに、ライブラリは、Apache Spark レベル イベントと Spark Structured Streaming メトリックをジョブから Azure Monitor にストリーミングします。 これらのイベントとメトリックのアプリケーション コードに変更を加える必要はありません。 Spark Structured Streaming のログ データは、"アクティブ | カスタム ログ | SparkListenerEvent_CL" スキーマにあります。

![Log Analytics ワークスペースのスクリーンショット](./_images/workspace.png)

ログの表示方法の詳細については、「[Azure Monitor におけるログ データの表示と分析](/azure/azure-monitor/log-query/portals)」を参照してください。

## <a name="next-steps"></a>次の手順

この記事では、Azure Monitor にメトリックを送信するようにクラスターを構成する方法について説明しました。 次の記事では、Azure Databricks の監視ライブラリを使用してアプリケーションのメトリックとログを送信する方法を示します。

> [!div class="nextstepaction"]
> [アプリケーション ログを Azure Monitor に送信する](./application-logs.md)
