---
title: Azure Stream Analytics によるストリーム処理
description: Azure でエンド ツー エンドのストリーム処理パイプラインを作成します。
author: MikeWasson
ms.date: 08/09/2018
ms.openlocfilehash: 82887bdd45f811ac733ead18c1f256098e575253
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/10/2018
ms.locfileid: "40025510"
---
# <a name="stream-processing-with-azure-stream-analytics"></a>Azure Stream Analytics によるストリーム処理

この参照アーキテクチャでは、エンド ツー エンドのストリーム処理パイプラインを示します。 このパイプラインでは、2 つのソースからデータを取り込み、2 つのストリームのレコードを関連付けて、時間枠全体の移動平均を計算します。 結果が保存され、さらに詳しい分析が行われます。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

![](./images/stream-processing-asa/stream-processing-asa.png)

**シナリオ**: タクシー会社が各乗車に関するデータを収集しています。 このシナリオでは、データを送信する 2 つのデバイスがあることを想定しています。 タクシーには、各乗車の情報 (走行時間、距離、乗車場所と降車場所) を送信するメーターがあります。 別のデバイスでは、乗客からの支払いを受け付け、料金に関するデータを送信します。 タクシー会社では、傾向をつかむために、走行 1 マイルあたりの平均チップをリアルタイムで計算したいと考えています。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

**データ ソース**。 このアーキテクチャには、リアルタイムでデータ ストリームを生成する 2 つのデータ ソースがあります。 1 つ目のストリームには乗車情報が含まれ、2 つ目のストリームには料金情報が含まれます。 参照アーキテクチャには、一連の静的ファイルから読み取り、データを Event Hubs にプッシュするシミュレートされたデータ ジェネレーターが含まれています。 実際のアプリケーションでは、データ ソースはタクシーに設置されたデバイスになります。

**Azure Event Hubs**。 [Event Hubs](/azure/event-hubs/) はイベント取り込みサービスです。 このアーキテクチャでは、2 つのイベント ハブ インスタンス (データ ソースごとに 1 つ) を使用します。 各データ ソースは、関連付けられたイベント ハブにデータ ストリームを送信します。

**Azure Stream Analytics**。 [Stream Analytics](/azure/stream-analytics/) はイベント処理エンジンです。 Stream Analytics ジョブでは、2 つのイベント ハブからデータ ストリームを読み取り、ストリーム処理を実行します。

**Cosmos DB**。 Stream Analytics ジョブの出力は一連のレコードであり、JSON ドキュメントとして Cosmos DB ドキュメント データベースに書き込まれます。

**Microsoft Power BI**。 Power BI は、データを分析してビジネスの分析情報を得る一連のビジネス分析ツールです。 このアーキテクチャでは、Cosmos DB からデータを読み込みます。 これにより、ユーザーは収集された過去のデータの完全なセットを分析できます。 また、Stream Analytics から Power BI に結果を直接ストリーミングして、データのリアルタイム ビューを表示することもできます。 詳細については、「[Power BI のリアルタイム ストリーミング](/power-bi/service-real-time-streaming)」をご覧ください。

**Azure Monitor**。 [Azure Monitor](/azure/monitoring-and-diagnostics/) は、ソリューションにデプロイされた Azure サービスに関するパフォーマンス メトリックを収集します。 ダッシュボードでこれらを視覚化することで、ソリューションの正常性を把握できます。 

## <a name="data-ingestion"></a>データの取り込み

データ ソースをシミュレートするために、この参照アーキテクチャでは [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) データセット<sup>[[1]](#note1)</sup> を使用します。 このデータセットには、ニューヨーク市の 4 年間 (2010 から 2013 年) のタクシー乗車に関するデータが含まれています。 乗車データと料金データの 2 種類のレコードがあります。 乗車データには、走行時間、乗車距離、乗車場所と降車場所が含まれます。 料金データには、料金、税、チップの金額が含まれます。 この 2 種類のレコードの共通フィールドには、営業許可番号、タクシー免許、ベンダー ID があります。 この 3 つのフィールドを組み合わせて、タクシーと運転手が一意に識別されます。 データは CSV 形式で保存されます。 

データ ジェネレーターは、レコードを読み取り、Azure Event Hubs に送信する .NET Core アプリケーションです。 ジェネレーターは、JSON 形式の乗車データと CSV 形式の料金データを送信します。 

Event Hubs では、[パーティション](/azure/event-hubs/event-hubs-features#partitions)を使用してデータをセグメント化します。 複数のパーティションでは、コンシューマーは各パーティションを並列で読み取ることができます。 Event Hubs にデータを送信するときに、パーティション キーを明示的に指定できます。 それ以外の場合は、ラウンド ロビン方式でパーティションにレコードが割り当てられます。 

このシナリオでは、特定のタクシーの乗車データと料金データは、最終的に同じパーティション ID を共有します。 これにより、Stream Analytics は 2 つのストリームを関連付けるときに、ある程度の並列処理を適用できます。 乗車データのパーティション *n* 内のレコードは、料金データのパーティション *n* 内のレコードに対応します。

![](./images/stream-processing-asa/stream-processing-eh.png)

データ ジェネレーターでは、両方のレコードの種類の共通データ モデルに、`Medallion`、`HackLicense`、`VendorId` を連結した `PartitionKey` プロパティがあります。

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

## <a name="stream-processing"></a>ストリーム処理

ストリーム処理ジョブは、複数の異なるステップで SQL クエリを使用して定義されます。 最初の 2 つのステップでは、2 つの入力ストリームからレコードを選択するだけです。

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

次のステップでは、2 つの入力ストリームを結合して、各ストリームから一致するレコードを選択します。

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

このクエリでは、一致するレコードを一意に識別する一連のフィールド (Medallion、HackLicense、VendorId、PickupTime) でレコードを結合します。 `JOIN` ステートメントには、パーティション ID も含まれています。 前述のように、これは、このシナリオでは一致するレコードは常に同じパーティション ID を持つという事実を利用しています。

Stream Analytics では、結合は "*一時的*" なものであり、特定の期間内だけレコードが結合されます。 それ以外は、ジョブでは一致を無期限に待つ必要があります。 [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) 関数は、一致と見なされるために、一致する 2 つのレコードの許容される時間の間隔を指定します。 

ジョブの最後のステップでは、5 分のホッピング ウィンドウでグループ化された、1 マイルあたりの平均チップを計算します。

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

Stream Analytics には、[ウィンドウ関数](/azure/stream-analytics/stream-analytics-window-functions)がいくつか用意されています。 ホッピング ウィンドウは、一定期間単位で時間が進みます (この例では、1 ホップあたり 1 分)。 結果は、過去 5 分間の移動平均を計算したものです。

ここで示すアーキテクチャでは、Stream Analytics ジョブの結果だけが Cosmos DB に保存されます。 ビッグ データ シナリオでは、[Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) を使用して、生イベント データを Azure Blob Storage に保存することも検討してください。 生データを保持すると、データから新しい分析情報を引き出すために、後で過去のデータに対してバッチ クエリを実行できます。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

### <a name="event-hubs"></a>Event Hubs

Event Hubs のスループット容量は、[スループット ユニット](/azure/event-hubs/event-hubs-features#throughput-units)で測定されます。 [自動インフレ](/azure/event-hubs/event-hubs-auto-inflate)を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。 

### <a name="stream-analytics"></a>Stream Analytics

Stream Analytics の場合、ジョブに割り当てられたコンピューティング リソースがストリーミング ユニットで測定されます。 ジョブを並列化できる場合は、Stream Analytics ジョブが最も効果的にスケーリングされます。 この場合、Stream Analytics は複数のコンピューティング ノードにジョブを分散できます。

Event Hubs の入力では、`PARTITION BY` キーワードを使用して Stream Analytics ジョブをパーティション分割します。 データは、Event Hubs のパーティションに基づいてサブセットに分割されます。 

ウィンドウ関数と一時的な結合には追加の SU が必要です。 可能であれば、各パーティションが個別に処理されるように、`PARTITION BY` を使用します。 詳細については、「[ストリーミング ユニットの理解と調整](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates)」をご覧ください。

Stream Analytics ジョブ全体を並列化できない場合は、ジョブを複数のステップに分割し、1 つ以上の並列ステップから開始することを試みます。 これにより、最初のステップを並列実行できます。 たとえば、この参照アーキテクチャでは、ステップは次のようになります。

- ステップ 1 と 2 は、1 つのパーティション内のレコードを選択する単純な `SELECT` ステートメントです。 
- ステップ 3 では、2 つの入力ストリーム間でパーティション結合を実行します。 このステップでは、一致するレコードは同じパーティション キーを共有するので、各入力ストリームに同じパーティション ID が含まれていることが保証されるという事実を利用しています。
- ステップ 4 では、すべてのパーティションにわたる集計を実行します。 このステップは並列化できません。

ジョブの各ステップに割り当てられているパーティションの数を確認するには、Stream Analytics の[ジョブ ダイアグラム](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics)を使用します。 次の図は、この参照アーキテクチャのジョブ ダイアグラムを示しています。

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a>Cosmos DB

Cosmos DB のスループット容量は、[要求ユニット](/azure/cosmos-db/request-units) (RU) で測定されます。 Cosmos DB コンテナーを 10,000 RU を超えてスケーリングするには、コンテナーの作成時に[パーティション キー](/azure/cosmos-db/partition-data)を指定し、すべてのドキュメントにパーティション キーを含める必要があります。 

この参照アーキテクチャでは、新しいドキュメントは 1 分に 1 回だけ (ホッピング ウィンドウ間隔) 作成されるので、スループット要件は非常に低くなっています。 そのため、このシナリオではパーティション キーを割り当てる必要はありません。

## <a name="monitoring-considerations"></a>監視に関する考慮事項

ストリーム処理ソリューションでは、システムのパフォーマンスと正常性を監視することが重要です。 [Azure Monitor](/azure/monitoring-and-diagnostics/) は、アーキテクチャで使用されている Azure サービスのメトリックと診断ログを収集します。 Azure Monitor は Azure プラットフォームに組み込まれており、アプリケーションにコードを追加する必要はありません。

次の警告シグナルは、該当する Azure リソースをスケールアウトする必要があることを示しています。

- Event Hubs が要求を調整しているか、1 日のメッセージ クォータに近づいている。
- Stream Analytics ジョブが、割り当てられたストリーミング ユニット (SU) の 80% 以上を常に使用している。
- Cosmos DB が要求を調整し始めている。

参照アーキテクチャには、Azure portal にデプロイされるカスタム ダッシュボードが含まれています。 アーキテクチャをデプロイしたら、[Azure portal](https://portal.azure.com) を開き、ダッシュボードの一覧から `TaxiRidesDashboard` を選択することでダッシュボードを表示できます。 Azure portal でのカスタム ダッシュボードの作成とデプロイの詳細については、「[プログラムによる Azure ダッシュボードの作成](/azure/azure-portal/azure-portal-dashboards-create-programmatically)」をご覧ください。

次の画像は、Stream Analytics ジョブが約 1 時間実行された後のダッシュボードを示しています。

![](./images/stream-processing-asa/asa-dashboard.png)

左下のパネルは、Stream Analytics ジョブの SU 消費量が最初の 15 分間に上昇した後、横ばい状態になっていることを示しています。 これは、ジョブが定常状態に達するときの一般的なパターンです。 

右上のパネルに示すように、Event Hubs が要求を調整していることに注意してください。 Event Hubs クライアント SDK は、調整エラーが発生すると自動的に再試行するため、不定期に調整される要求は問題ありません。 ただし、調整エラーが常に発生する場合は、イベント ハブにより多くのスループット ユニットが必要であることを意味しています。 次のグラフは、必要に応じてスループット ユニットを自動的にスケールアウトする、Event Hubs の自動インフレ機能を使用したテストの実行を示しています。 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

自動インフレは、06:35 マークあたりで有効化されました。 Event Hubs が 3 スループット ユニットに自動的にスケールアップされたため、調整された要求数が激減していることがわかります。

興味深いのは、これには、Stream Analytics ジョブの SU 使用率が増加するという副作用があったことです。 調整によって、Event Hubs は Stream Analytics ジョブの取り込み率を故意に下げていました。 パフォーマンスの 1 つのボトルネックの解消によって、別のボトルネックが明らかになることは実際によくあります。 この例では、Stream Analytics ジョブに追加の SU を割り当てることで問題が解決されました。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照アーキテクチャのデプロイは、[GitHub](https://github.com/mspnp/reference-architectures/tree/master/data) で入手できます。 

### <a name="prerequisites"></a>前提条件

1. [参照アーキテクチャ](https://github.com/mspnp/reference-architectures) GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. [Docker](https://www.docker.com/) をインストールして、データ ジェネレーターを実行します。

3. [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。

    ```
    az login
    ```

### <a name="download-the-source-data-files"></a>ソース データ ファイルをダウンロードする

1. GitHub リポジトリの `data/streaming_asa` ディレクトリに、`DataFile` という名前のディレクトリを作成します。

2. Web ブラウザーを開き、https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935 に移動します。

3. このページの **[ダウンロード]** ボタンをクリックして、その年のすべてのタクシー データの ZIP ファイルをダウンロードします。

4. ZIP ファイルを `DataFile` ディレクトリに抽出します。

    > [!NOTE]
    > この ZIP ファイルには、他の ZIP ファイルが含まれています。 子 ZIP ファイルは抽出しないでください。

ディレクトリ構造は次のようになります。

```
/data
    /streaming_asa
        /DataFile
            /FOIL2013
                trip_data_1.zip
                trip_data_2.zip
                trip_data_3.zip
                ...
```

### <a name="deploy-the-azure-resources"></a>Azure リソースをデプロイする

1. シェルまたは Windows コマンド プロンプトから次のコマンドを実行し、サインインプロンプトに従います。

    ```bash
    az login
    ```

2. GitHub リポジトリの `data/streaming_asa` フォルダーに移動します。

    ```bash
    cd data/streaming_asa
    ```

2. 次のコマンドを実行して、Azure リソースをデプロイします。

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Location]'
    export cosmosDatabaseAccount='[Cosmos DB account name]'
    export cosmosDatabase='[Cosmod DB database name]'
    export cosmosDataBaseCollection='[Cosmos DB collection name]'
    export eventHubNamespace='[Event Hubs namespace name]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
      --template-file ./azure/deployresources.json --parameters \
      eventHubNamespace=$eventHubNamespace \
      outputCosmosDatabaseAccount=$cosmosDatabaseAccount \
      outputCosmosDatabase=$cosmosDatabase \
      outputCosmosDatabaseCollection=$cosmosDataBaseCollection

    # Create a database 
    az cosmosdb database create --name $cosmosDatabaseAccount \
        --db-name $cosmosDatabase --resource-group $resourceGroup

    # Create a collection
    az cosmosdb collection create --collection-name $cosmosDataBaseCollection \
        --name $cosmosDatabaseAccount --db-name $cosmosDatabase \
        --resource-group $resourceGroup
    ```

3. Azure portal で、作成済みのリソース グループに移動します。

4. Stream Analytics ジョブのブレードを開きます。

5. **[開始]** をクリックしてジョブを開始します。 出力開始時刻として **[Now]\(今すぐ\)** を選択します。 ジョブが開始されるまで待ちます。

### <a name="run-the-data-generator"></a>データ ジェネレーターを実行する

1. イベント ハブの接続文字列を取得します。 接続文字列は、Azure portal から取得することも、次の CLI コマンドを実行して取得することもできます。

    ```bash
    # RIDE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-ride \
        --name taxi-ride-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString

    # FARE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-fare \
        --name taxi-fare-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString
    ```

2. GitHub リポジトリの `data/streaming_asa/onprem` ディレクトリに移動します。

3. `main.env` ファイル内の値を次のように更新します。

    ```
    RIDE_EVENT_HUB=[Connection string for taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```

4. 次のコマンドを実行して、Docker イメージをビルドします。

    ```bash
    docker build --no-cache -t dataloader .
    ```

5. 親ディレクトリ `data/stream_asa` に戻ります。

    ```bash
    cd ..
    ```

6. 次のコマンドを実行して、Docker イメージを実行します。

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

プログラムを少なくとも 5 分間 (Stream Analyticsクエリで定義されているウィンドウ) 実行します。 Stream Analytics ジョブが正しく実行されていることを確認するには、Azure portal を開き、Cosmos DB データベースに移動します。 **データ エクスプローラー** ブレードを開き、ドキュメントを表示します。 

[1] <span id="note1">Brian Donovan、Dan Work (2016 年): New York City Taxi Trip Data (2010-2013)。 イリノイ大学アーバナシャンペーン校。 https://doi.org/10.13012/J8PN93H8
