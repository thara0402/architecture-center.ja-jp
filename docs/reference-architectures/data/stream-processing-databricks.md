---
title: Azure Databricks によるストリーム処理
titleSuffix: Azure Reference Architectures
description: Azure Databricks を使用して、Azure でエンド ツー エンドのストリーム処理パイプラインを作成します。
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: 822a3c448dcc2bdd4ae77ef2a2b7a9ffad633440
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120324"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Azure Databricks を使用してストリーム処理パイプラインを作成します

この参照アーキテクチャでは、エンド ツー エンドの[ストリーム処理](/azure/architecture/data-guide/big-data/real-time-processing)パイプラインを示します。 この種類のパイプラインには、取り込み、処理、格納、および分析とレポート作成の 4 つの段階があります。 この参照アーキテクチャでは、パイプラインは、2 つのソースからデータを取り込み、各ストリームの関連するレコードに対して結合を実行し、結果を強化させ、リアルタイムで平均を計算します。 結果が保存され、さらに詳しい分析が行われます。 [**このソリューションをデプロイします**](#deploy-the-solution)。

![Azure Databricks によるストリーム処理の参照アーキテクチャ](./images/stream-processing-databricks.png)

**シナリオ**:タクシー会社が各乗車に関するデータを収集しています。 このシナリオでは、データを送信する 2 つのデバイスがあることを想定しています。 タクシーには、各乗車の情報 (走行時間、距離、乗車場所と降車場所) を送信するメーターがあります。 別のデバイスでは、乗客からの支払いを受け付け、料金に関するデータを送信します。 利用者数の傾向をつかむために、このタクシー会社では、各地で走行 1 マイルあたりの平均チップをリアルタイムで計算したいと考えています。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

**データ ソース**。 このアーキテクチャには、リアルタイムでデータ ストリームを生成する 2 つのデータ ソースがあります。 1 つ目のストリームには乗車情報が含まれ、2 つ目のストリームには料金情報が含まれます。 参照アーキテクチャには、一連の静的ファイルから読み取り、データを Event Hubs にプッシュするシミュレートされたデータ ジェネレーターが含まれています。 実際のアプリケーションのデータ ソースは、タクシーに設置されたデバイスになります。

**Azure Event Hubs**。 [Event Hubs](/azure/event-hubs/) はイベント取り込みサービスです。 このアーキテクチャでは、2 つのイベント ハブ インスタンス (データ ソースごとに 1 つ) を使用します。 各データ ソースは、関連付けられたイベント ハブにデータ ストリームを送信します。

**Azure Databricks**。 [Databricks](/azure/azure-databricks/) は、Microsoft Azure クラウド サービス プラットフォーム用に最適化された Apache Spark ベースの分析プラットフォームです。 Databricks を使用して、タクシーの乗車データと料金データを関連付け、さらに、その関連付けられたデータを、Databricks ファイル システムに格納されている地域データで強化します。

**Cosmos DB**。 Azure Databricks ジョブの出力は一連のレコードであり、Cassandra API を使用して [Cosmos DB](/azure/cosmos-db/) に書き込まれます。 Cassandra API が使用されるのは、時系列データ モデリングをサポートしているためです。

**Azure Log Analytics**。 [Azure Monitor](/azure/monitoring-and-diagnostics/) で収集されたアプリケーション ログ データは、[Log Analytics ワークスペース](/azure/log-analytics)に保存されます。 Log Analytics クエリを使用してメトリックを分析および視覚化し、ログ メッセージを検査してアプリケーション内の問題を特定できます。

## <a name="data-ingestion"></a>データの取り込み

データ ソースをシミュレートするために、この参照アーキテクチャでは [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) データセット<sup>[[1]](#note1)</sup> を使用します。 このデータセットには、ニューヨーク市の 4 年間 (2010 年から 2013 年) のタクシー乗車に関するデータが含まれています。 乗車データと料金データの 2 種類のレコードがあります。 乗車データには、走行時間、乗車距離、乗車場所と降車場所が含まれます。 料金データには、料金、税、チップの金額が含まれます。 この 2 種類のレコードの共通フィールドには、営業許可番号、タクシー免許、ベンダー ID があります。 この 3 つのフィールドを組み合わせて、タクシーと運転手が一意に識別されます。 データは CSV 形式で保存されます。

データ ジェネレーターは、レコードを読み取り、Azure Event Hubs に送信する .NET Core アプリケーションです。 ジェネレーターは、JSON 形式の乗車データと CSV 形式の料金データを送信します。

Event Hubs では、[パーティション](/azure/event-hubs/event-hubs-features#partitions)を使用してデータをセグメント化します。 複数のパーティションでは、コンシューマーは各パーティションを並列で読み取ることができます。 Event Hubs にデータを送信するときに、パーティション キーを明示的に指定できます。 それ以外の場合は、ラウンド ロビン方式でパーティションにレコードが割り当てられます。

このシナリオでは、特定のタクシーの乗車データと料金データは、最終的に同じパーティション ID を共有します。 これにより、Databricks は 2 つのストリームを関連付けるときに、ある程度の並列処理を適用できます。 乗車データのパーティション *n* 内のレコードは、料金データのパーティション *n* 内のレコードに対応します。

![Azure Databricks と Event Hubs によるストリーム処理のダイアグラム](./images/stream-processing-databricks-eh.png)

データ ジェネレーターでは、両方のレコードの種類に対応した共通データ モデルに、`Medallion`、`HackLicense`、`VendorId` を連結した `PartitionKey` プロパティがあります。

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

このプロパティを使用して、Event Hubs への送信時に明示的なパーティション キーが提供されます。

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a>Event Hubs

Event Hubs のスループット容量は、[スループット ユニット](/azure/event-hubs/event-hubs-features#throughput-units)で測定されます。 [自動インフレ](/azure/event-hubs/event-hubs-auto-inflate)を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。

## <a name="stream-processing"></a>ストリーム処理

Azure Databricks では、ジョブによってデータ処理が実行されます。 ジョブはクラスターに割り当てられ、そこで実行されます。 ジョブは、Java で記述されたカスタム コードか、Spark [Notebook](https://docs.databricks.com/user-guide/notebooks/index.html) です。

この参照アーキテクチャでは、ジョブは、Java と Scala の両方で記述されたクラスを含む Java アーカイブです。 Databricks ジョブの Java アーカイブを指定する場合、実行対象のクラスは Databricks クラスターによって指定されます。 ここでは、**com.microsoft.pnp.TaxiCabReader** クラスの **main** メソッドに、データ処理ロジックが含まれています。

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>2 つのイベント ハブ インスタンスからのストリームの読み取り

データ処理ロジックでは、2 つの Azure イベント ハブ インスタンスからの読み取りに、[Spark 構造化ストリーミング](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html)が使用されます。

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a>地域情報によるデータの強化

乗車データには、乗車場所と降車場所の緯度と経度の座標が含まれています。 これらの座標は便利ですが、分析で使用するのは簡単ではありません。 したがって、このデータは、[シェープファイル](https://en.wikipedia.org/wiki/Shapefile)から読み取られる地域データで強化されます。

シェープファイルはバイナリ形式で、簡単には解析されませんが、[GeoTools](http://geotools.org/) ライブラリには、シェープファイル形式を使用する地理空間データを対象としたツールが用意されています。 このライブラリは、乗車場所と降車場所の座標に基づいて地域名を判断するために、**com.microsoft.pnp.GeoFinder** クラスで使用されます。

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>乗車データと料金データの結合

最初に、乗車データと料金データが変換されます。

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

次に、乗車データが、料金データと結合されます。

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>データの処理と Cosmos DB への挿入

指定された期間について、平均料金が地域ごとに計算されます。

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

次に、これが Cosmos DB に挿入されます。

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>セキュリティに関する考慮事項

Azure Database ワークスペースへのアクセスは、[管理者コンソール](https://docs.databricks.com/administration-guide/admin-settings/index.html)を使用して制御されます。 管理者コンソールには、ユーザーを追加する、ユーザーのアクセス許可を管理する、シングル サインオンを設定する、といった機能が含まれています。 ワークスペース、クラスター、ジョブ、およびテーブルのアクセス制御も、管理者コンソールで設定できます。

### <a name="managing-secrets"></a>シークレットを管理する

Azure Databricks には[シークレット ストア](https://docs.azuredatabricks.net/user-guide/secrets/index.html)があり、これを使用して接続文字列、アクセス キー、ユーザー名、パスワードなどのシークレットを格納します。 Azure Databricks シークレット ストア内のシークレットは、**スコープ**ごとにパーティション分割されています。

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

シークレットは、スコープ レベルで追加されます。

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> ネイティブの Azure Databricks スコープではなく、Azure Key Vault を実体とするスコープを使用できます。 詳細については、「[Azure Key Vault-backed scopes (Azure Key Vault を実体とするスコープ)](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)」を参照してください。

コードでは、Azure Databricks [シークレット ユーティリティ](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities)を介してシークレットにアクセスします。

## <a name="monitoring-considerations"></a>監視に関する考慮事項

Azure Databricks は Apache Spark に基づいており、どちらも、ログ記録用の標準ライブラリとして [log4j](https://logging.apache.org/log4j/2.x/) を使用しています。 Apache Spark によって提供される既定のログ記録に加え、この参照アーキテクチャでは、ログとメトリックを [Azure Log Analytics](/azure/log-analytics/) に送信します。

**com.microsoft.pnp.TaxiCabReader** クラスでは、**log4j.properties** ファイルの値を使用して、Apache Spark のログを Azure Log Analytics に送信するように、Apache Spark ログ記録システムを構成します。 Apache Spark のロガー メッセージは文字列ですが、Azure Log Analytics では、ログ メッセージが JSON として書式設定されていなければなりません。 **com.microsoft.pnp.log4j.LogAnalyticsAppender** クラスは、これらのメッセージを JSON に変換します。

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

**com.microsoft.pnp.TaxiCabReader** クラスでは、乗車メッセージと料金メッセージが処理されるため、いずれかの形式が正しくない可能性があり、これが原因で無効になることがあります。 運用環境では、形式に誤りがあるこれらのメッセージを分析して、データ ソースの問題を特定することが重要です。これにより、その問題を迅速に修正して、データ損失を防ぐことができます。 **com.microsoft.pnp.TaxiCabReader** クラスにより、形式に誤りがある料金レコードと乗車レコードの数を追跡する Apache Spark アキュムレータが登録されます。

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

Apache Spark は Dropwizard ライブラリを使用してメトリックを送信しますが、ネイティブ Dropwizard メトリック フィールドの中には、Azure Log Analytics と互換性がないものがあります。 このため、この参照アーキテクチャには、カスタム Dropwizard シンクおよびレポーターが含まれています。 これにより、メトリックは、Azure Log Analytics で予期される形式に書式設定されます。 Apache Spark によってメトリックがレポートされると、形式に誤りがある乗車データと料金データに対するカスタム メトリックも送信されます。

Azure Log Analytics ワークスペースに記録される最後のメトリックは、Spark Structured Streaming ジョブの累積的な進行状況です。 これは、**com.microsoft.pnp.StreamingMetricsListener** クラスで実装されるカスタム StreamingQuery リスナーを使用して、実行されます。 このクラスは、ジョブの実行時に、Apache Spark セッションに登録されます。

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

StreamingMetricsListener 内のメソッドは、構造化ストリーミング イベントが発生すると必ず、Apache Spark ランタイムによって呼び出され、ログ メッセージとメトリックを Azure Log Analytics ワークスペースに送信します。 アプリケーションを監視するには、ワークスペースで次のクエリを使用とできます。

### <a name="latency-and-throughput-for-streaming-queries"></a>ストリーミング クエリの待機時間とスループット

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>ストリーム クエリの実行中に記録された例外

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>形式に誤りがある料金データと乗車データの蓄積

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedrides"

SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedfares"
```

### <a name="job-execution-to-trace-resiliency"></a>回復性をトレースするジョブ実行

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照アーキテクチャのデプロイは、[GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics) で入手できます。

### <a name="prerequisites"></a>前提条件

1. 「[Azure Databricks によるストリーム処理](https://github.com/mspnp/azure-databricks-streaming-analytics)」GitHub リポジトリを複製、フォーク、またはダウンロードします。

2. [Docker](https://www.docker.com/) をインストールして、データ ジェネレーターを実行します。

3. [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。

4. [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html) をインストールします。

5. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。
    ```shell
    az login
    ```
6. Java IDE と次のリソースをインストールします。
    - JDK 1.8
    - Scala SDK 2.11
    - Maven 3.5.4

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>ニューヨーク市のタクシー データ ファイルと地域データ ファイルをダウンロードする

1. ご自身のローカル ファイル システムで、複製された GitHub リポジトリのルートに `DataFile` という名前のディレクトリを作成します。

2. Web ブラウザーを開き、 https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935 に移動します。

3. このページの **[ダウンロード]** ボタンをクリックして、その年のすべてのタクシー データの ZIP ファイルをダウンロードします。

4. ZIP ファイルを `DataFile` ディレクトリに抽出します。

    > [!NOTE]
    > この ZIP ファイルには、他の ZIP ファイルが含まれています。 子 ZIP ファイルは抽出しないでください。

    ディレクトリ構造は次のようにする必要があります。

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. Web ブラウザーを開き、 https://www.zillow.com/howto/api/neighborhood-boundaries.htm に移動します。

6. **New York Neighborhood Boundaries** をクリックして、ファイルをダウンロードします。

7. お使いのブラウザーの**ダウンロード** ディレクトリにある **ZillowNeighborhoods-NY.zip** ファイルを、`DataFile` ディレクトリにコピーします。

### <a name="deploy-the-azure-resources"></a>Azure リソースをデプロイする

1. シェルまたは Windows コマンド プロンプトから次のコマンドを実行し、サインインプロンプトに従います。

    ```bash
    az login
    ```

2. GitHub リポジトリの `azure` という名前のフォルダーに移動します。

    ```bash
    cd azure
    ```

3. 次のコマンドを実行して、Azure リソースをデプロイします。

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. 完了すると、デプロイの出力はコンソールに書き込まれます。 出力で次の JSON を探します。

```json
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```

これらの値は、以降のセクションで Databricks シークレットに追加されるシークレットです。 これらのセクションで追加するまで、安全に保管してください。

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>Cassandra テーブルを Cosmos DB アカウントに追加する

1. Azure portal 上で、前の「**Azure リソースをデプロイする**」セクションで作成したリソース グループに移動します。 **[Azure Cosmos DB アカウント]** をクリックします。 Cassandra API でテーブルを作成します。

2. **[概要]** ブレードで、**[テーブルの追加]** をクリックします。

3. **[テーブルの追加]** ブレードが開いたら、**[Keyspace name]\(キースペース名\)** テキスト ボックスに「`newyorktaxi`」と入力します。

4. **[enter CQL command to create the table]\(テーブルを作成する CQL コマンドを入力\)** セクションで、`newyorktaxi` の横にあるテキスト ボックスに「`neighborhoodstats`」と入力します。

5. 下のテキスト ボックスに、次のように入力します。
    ```shell
    (neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
    ```
6. **[スループット (1,000 - 1,000,000 RU/秒)]** テキスト ボックスに、値 `4000` を入力します。

7. Click **OK**.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>Databricks CLI を使用して Databricks シークレットを追加する

最初に、EventHub のシークレットを入力します。

1. 前提条件の手順 2. でインストールした **Azure Databricks CLI** を使用して、Azure Databricks シークレット スコープを作成します。
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. タクシー乗車 EventHub のシークレットを追加します。
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    実行されると、このコマンドによって vi エディターが開きます。 「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値を入力します。 保存して vi を終了します。

3. タクシー料金 EventHub のシークレットを追加します。
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    実行されると、このコマンドによって vi エディターが開きます。 「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値を入力します。 保存して vi を終了します。

次に、Cosmos DB のシークレットを入力します。

1. Azure portal を開いて、「**Azure リソースをデプロイする**」セクションの手順 3. で指定したリソース グループに移動します。 [Azure Cosmos DB アカウント] をクリックします。

2. **Azure Databricks CLI** を使用して、Cosmos DB ユーザー名のシークレットを追加します。
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
実行されると、このコマンドによって vi エディターが開きます。 「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **username** 値を入力します。 保存して vi を終了します。

3. 次に、Cosmos DB パスワードのシークレットを追加します。
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

実行されると、このコマンドによって vi エディターが開きます。 「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **secret** 値を入力します。 保存して vi を終了します。

> [!NOTE]
> [Azure Key Vault を実体とするシークレット スコープ](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)を使用している場合、スコープの名前は **azure-databricks-job** にする必要があります。また、シークレットの名前は、上記とまったく同じにしなければなりません。

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Zillow 地域データ ファイルを Databricks ファイル システムに追加する

1. Databricks ファイル システムでディレクトリを作成します。
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. `DataFile` ディレクトリに移動し、次を入力します。
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Azure Log Analytics ワークスペースの ID と主キーを構成ファイルに追加する

このセクションでは、Log Analytics ワークスペースの ID と主キーが必要です。 ワークスペース ID は、「*Azure リソースをデプロイする*」セクションの手順 4. の **logAnalytics** 出力セクションに示されている **workspaceId** 値です。 主キーは、その出力セクションの **secret** です。

1. log4j ログ記録を構成するには、`\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties` を開きます。 次の 2 つの値を編集します。
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. カスタム ログ記録を構成するには、`\azure\azure-databricks-monitoring\scripts\metrics.properties` を開きます。 次の 2 つの値を編集します。
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Databricks ジョブおよび Databricks 監視用の .jar ファイルをビルドする

1. ご自身の Java IDE を使用して、ルート ディレクトリにある、**pom.xml** という名前の Maven プロジェクト ファイルをインポートします。

2. クリーン ビルドを実行します。 このビルドの出力は、**azure-databricks-job-1.0-SNAPSHOT.jar** および **azure-databricks-monitoring-0.9.jar** という名前のファイルです。

### <a name="configure-custom-logging-for-the-databricks-job"></a>Databricks ジョブのカスタム ログ記録を構成する

1. **Databricks CLI** で次のコマンドを入力して、**azure-databricks-monitoring-0.9.jar** ファイルを Databricks ファイル システムにコピーします。
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. 次のコマンドを入力して、`\azure\azure-databricks-monitoring\scripts\metrics.properties` のカスタム ログ記録プロパティを Databricks ファイル システムにコピーします。
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. Databricks クラスターの名前をまだ決めていませんが、ここでそれを選択します。 お使いのクラスターの Databricks ファイル システム パスに、以下の名前を入力します。 次のコマンドを入力して、`\azure\azure-databricks-monitoring\scripts\spark.metrics` の初期化スクリプトを Databricks ファイル システムにコピーします。
    ```shell
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Databricks クラスターを作成する

1. Databricks ワークスペースで、[クラスター]、[クラスターの作成] の順にクリックします。 前の「**Databricks ジョブのカスタム ログ記録を構成する**」セクションの手順 3. で作成したクラスター名を入力します。

2. **[標準]** クラスター モードを選択します。

3. **[Databricks runtime version]\(Databricks Runtime のバージョン\)** を **[4.3 (includes Apache Spark 2.3.1, Scala 2.11)]\(4.3 (Apache Spark 2.3.1、Scala 2.11 を含む)\)** に設定します

4. **[Python バージョン]** を **[2]** に設定します。

5. **[ドライバーの種類]** を **[Same as worker]\(worker と同じ\)** に設定します

6. **[ワーカー タイプ]** を **[Standard_DS3_v2]** に設定します。

7. **[Min Workers]\(最小 worker 数\)** を **[2]** に設定します。

8. **[Enable autoscaling]\(自動スケールを有効にする\)** の選択を解除します。

9. **[Auto Termination]\(自動的に終了\)** ダイアログ ボックスで、**[Init Scripts]\(初期化スクリプト\)** をクリックします。

10. 「**dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**」と入力します。<cluster-name> には、手順 1. で作成したクラスター名を指定します。

11. **[追加]** をクリックします。

12. **[クラスターの作成]** をクリックします。

### <a name="create-a-databricks-job"></a>Databricks ジョブを作成する

1. Databricks ワークスペースで、[ジョブ]、[ジョブの作成] の順にクリックします。

2. ジョブ名を入力します。

3. [set jar]\(jar の設定\) をクリックします。これにより、[Upload JAR to Run]\(実行する JAR のアップロード\) ダイアログ ボックスが開きます。

4. 「**Databricks ジョブ用の .jar ファイルをビルドする**」セクションで作成した **azure-databricks-job-1.0-SNAPSHOT.jar** ファイルを、**[Drop JAR here to upload]\(アップロードする JAR をここにドロップ\)** ボックスにドラッグします。

5. **[メイン クラス]** フィールドに「**com.microsoft.pnp.TaxiCabReader**」と入力します。

6. 引数フィールドに、次を入力します。
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above>
    ```

7. 次の手順に従って、依存するライブラリをインストールします。

    1. Databricks ユーザー インターフェイスで、**[ホーム]** をクリックします。

    2. **[ユーザー]** ドロップダウンで、ご自身のユーザー アカウント名をクリックして、お使いのアカウント ワークスペースの設定を開きます。

    3. ご自身のアカウント名の横にあるドロップダウン矢印をクリックし、**[作成]**、**[ライブラリ]** の順にクリックして、**[新しいライブラリ]** ダイアログを開きます。

    4. **[ソース]** ドロップダウン コントロールで、**[Maven Coordinate]\(Maven 座標\)** を選択します。

    5. **[Install Maven Artifacts]\(Maven Artifacts のインストール\)** 見出しで、**[Coordinate]\(座標\)** テキスト ボックスに「`com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5`」と入力します。

    6. **[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。

    7. **[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。

    8. `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven 座標について、手順 1. から 7. を繰り返します。

    9. `org.geotools:gt-shapefile:19.2` Maven 座標について、手順 1. から 6. を繰り返します。

    10. **[詳細オプション]** をクリックします。

    11. **[リポジトリ]** テキスト ボックスに「`http://download.osgeo.org/webdav/geotools/`」と入力します。

    12. **[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。 

    13. **[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。

8. 手順 7. で追加した依存ライブラリを、手順 6. の最後に作成したジョブに追加します。

    1. Azure Databricks ワークスペースで、**[ジョブ]** をクリックします。

    2. 「**Databricks ジョブを作成する**」セクションの手順 2. で作成されたジョブ名をクリックします。

    3. **[Dependent Libraries]\(依存ライブラリ\)** セクションの横で、**[追加]** をクリックして、**[Add Dependent Library]\(依存ライブラリの追加\)** ダイアログを開きます。

    4. **[Library From]\(ライブラリの選択元\)** で、**[ワークスペース]** を選択します。

    5. **[ユーザー]**、ご自身のユーザー名、`azure-eventhubs-spark_2.11:2.3.5` の順にクリックします。

    6. Click **OK**.

    7. `spark-cassandra-connector_2.11:2.3.1` および `gt-shapefile:19.2` について、手順 1. から 6. を繰り返します。

9. **[クラスター]** の横で、**[編集]** をクリックします。 これにより、**[クラスターの構成]** ダイアログが開きます。 **[クラスターの種類]** ドロップダウンで、**[Existing Cluster]\(既存のクラスター\)** を選択します。 **[クラスターの選択]** ドロップダウンで、「**Databricks クラスターを作成する**」セクションで作成されたクラスターを選択します。 **[確認]** をクリックします。

10. **[今すぐ実行]** をクリックします。

### <a name="run-the-data-generator"></a>データ ジェネレーターを実行する

1. GitHub リポジトリの `onprem` という名前のディレクトリに移動します。

2. 次のように、**main.env** ファイル内の値を更新します。

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    taxi-ride イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値です。 taxi-fare イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値です。

3. 次のコマンドを実行して、Docker イメージをビルドします。

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. 親ディレクトリに戻ります。

    ```bash
    cd ..
    ```

5. 次のコマンドを実行して、Docker イメージを実行します。

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

出力は次のようになります。

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Databricks ジョブが正しく実行されていることを確認するには、Azure portal を開き、Cosmos DB データベースに移動します。 **[データ エクスプローラー]** ブレードを開いて、**taxi records** テーブルでデータを確認します。 

[1] <span id="note1">Donovan, Brian; Work, Dan (2016):New York City Taxi Trip Data (2010-2013). イリノイ大学アーバナシャンペーン校。 https://doi.org/10.13012/J8PN93H8
