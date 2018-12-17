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
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="16e45-103">Azure Databricks を使用してストリーム処理パイプラインを作成します</span><span class="sxs-lookup"><span data-stu-id="16e45-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="16e45-104">この参照アーキテクチャでは、エンド ツー エンドの[ストリーム処理](/azure/architecture/data-guide/big-data/real-time-processing)パイプラインを示します。</span><span class="sxs-lookup"><span data-stu-id="16e45-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="16e45-105">この種類のパイプラインには、取り込み、処理、格納、および分析とレポート作成の 4 つの段階があります。</span><span class="sxs-lookup"><span data-stu-id="16e45-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="16e45-106">この参照アーキテクチャでは、パイプラインは、2 つのソースからデータを取り込み、各ストリームの関連するレコードに対して結合を実行し、結果を強化させ、リアルタイムで平均を計算します。</span><span class="sxs-lookup"><span data-stu-id="16e45-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="16e45-107">結果が保存され、さらに詳しい分析が行われます。</span><span class="sxs-lookup"><span data-stu-id="16e45-107">The results are stored for further analysis.</span></span> <span data-ttu-id="16e45-108">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="16e45-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Azure Databricks によるストリーム処理の参照アーキテクチャ](./images/stream-processing-databricks.png)

<span data-ttu-id="16e45-110">**シナリオ**:タクシー会社が各乗車に関するデータを収集しています。</span><span class="sxs-lookup"><span data-stu-id="16e45-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="16e45-111">このシナリオでは、データを送信する 2 つのデバイスがあることを想定しています。</span><span class="sxs-lookup"><span data-stu-id="16e45-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="16e45-112">タクシーには、各乗車の情報 (走行時間、距離、乗車場所と降車場所) を送信するメーターがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="16e45-113">別のデバイスでは、乗客からの支払いを受け付け、料金に関するデータを送信します。</span><span class="sxs-lookup"><span data-stu-id="16e45-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="16e45-114">利用者数の傾向をつかむために、このタクシー会社では、各地で走行 1 マイルあたりの平均チップをリアルタイムで計算したいと考えています。</span><span class="sxs-lookup"><span data-stu-id="16e45-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="16e45-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="16e45-115">Architecture</span></span>

<span data-ttu-id="16e45-116">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="16e45-117">**データ ソース**。</span><span class="sxs-lookup"><span data-stu-id="16e45-117">**Data sources**.</span></span> <span data-ttu-id="16e45-118">このアーキテクチャには、リアルタイムでデータ ストリームを生成する 2 つのデータ ソースがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="16e45-119">1 つ目のストリームには乗車情報が含まれ、2 つ目のストリームには料金情報が含まれます。</span><span class="sxs-lookup"><span data-stu-id="16e45-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="16e45-120">参照アーキテクチャには、一連の静的ファイルから読み取り、データを Event Hubs にプッシュするシミュレートされたデータ ジェネレーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="16e45-121">実際のアプリケーションのデータ ソースは、タクシーに設置されたデバイスになります。</span><span class="sxs-lookup"><span data-stu-id="16e45-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="16e45-122">**Azure Event Hubs**。</span><span class="sxs-lookup"><span data-stu-id="16e45-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="16e45-123">[Event Hubs](/azure/event-hubs/) はイベント取り込みサービスです。</span><span class="sxs-lookup"><span data-stu-id="16e45-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="16e45-124">このアーキテクチャでは、2 つのイベント ハブ インスタンス (データ ソースごとに 1 つ) を使用します。</span><span class="sxs-lookup"><span data-stu-id="16e45-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="16e45-125">各データ ソースは、関連付けられたイベント ハブにデータ ストリームを送信します。</span><span class="sxs-lookup"><span data-stu-id="16e45-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="16e45-126">**Azure Databricks**。</span><span class="sxs-lookup"><span data-stu-id="16e45-126">**Azure Databricks**.</span></span> <span data-ttu-id="16e45-127">[Databricks](/azure/azure-databricks/) は、Microsoft Azure クラウド サービス プラットフォーム用に最適化された Apache Spark ベースの分析プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="16e45-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="16e45-128">Databricks を使用して、タクシーの乗車データと料金データを関連付け、さらに、その関連付けられたデータを、Databricks ファイル システムに格納されている地域データで強化します。</span><span class="sxs-lookup"><span data-stu-id="16e45-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="16e45-129">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="16e45-129">**Cosmos DB**.</span></span> <span data-ttu-id="16e45-130">Azure Databricks ジョブの出力は一連のレコードであり、Cassandra API を使用して [Cosmos DB](/azure/cosmos-db/) に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="16e45-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="16e45-131">Cassandra API が使用されるのは、時系列データ モデリングをサポートしているためです。</span><span class="sxs-lookup"><span data-stu-id="16e45-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="16e45-132">**Azure Log Analytics**。</span><span class="sxs-lookup"><span data-stu-id="16e45-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="16e45-133">[Azure Monitor](/azure/monitoring-and-diagnostics/) で収集されたアプリケーション ログ データは、[Log Analytics ワークスペース](/azure/log-analytics)に保存されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="16e45-134">Log Analytics クエリを使用してメトリックを分析および視覚化し、ログ メッセージを検査してアプリケーション内の問題を特定できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="16e45-135">データの取り込み</span><span class="sxs-lookup"><span data-stu-id="16e45-135">Data ingestion</span></span>

<span data-ttu-id="16e45-136">データ ソースをシミュレートするために、この参照アーキテクチャでは [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) データセット<sup>[[1]](#note1)</sup> を使用します。</span><span class="sxs-lookup"><span data-stu-id="16e45-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="16e45-137">このデータセットには、ニューヨーク市の 4 年間 (2010 年から 2013 年) のタクシー乗車に関するデータが含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="16e45-138">乗車データと料金データの 2 種類のレコードがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="16e45-139">乗車データには、走行時間、乗車距離、乗車場所と降車場所が含まれます。</span><span class="sxs-lookup"><span data-stu-id="16e45-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="16e45-140">料金データには、料金、税、チップの金額が含まれます。</span><span class="sxs-lookup"><span data-stu-id="16e45-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="16e45-141">この 2 種類のレコードの共通フィールドには、営業許可番号、タクシー免許、ベンダー ID があります。</span><span class="sxs-lookup"><span data-stu-id="16e45-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="16e45-142">この 3 つのフィールドを組み合わせて、タクシーと運転手が一意に識別されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="16e45-143">データは CSV 形式で保存されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-143">The data is stored in CSV format.</span></span>

<span data-ttu-id="16e45-144">データ ジェネレーターは、レコードを読み取り、Azure Event Hubs に送信する .NET Core アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="16e45-144">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="16e45-145">ジェネレーターは、JSON 形式の乗車データと CSV 形式の料金データを送信します。</span><span class="sxs-lookup"><span data-stu-id="16e45-145">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="16e45-146">Event Hubs では、[パーティション](/azure/event-hubs/event-hubs-features#partitions)を使用してデータをセグメント化します。</span><span class="sxs-lookup"><span data-stu-id="16e45-146">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="16e45-147">複数のパーティションでは、コンシューマーは各パーティションを並列で読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="16e45-147">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="16e45-148">Event Hubs にデータを送信するときに、パーティション キーを明示的に指定できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-148">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="16e45-149">それ以外の場合は、ラウンド ロビン方式でパーティションにレコードが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="16e45-149">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="16e45-150">このシナリオでは、特定のタクシーの乗車データと料金データは、最終的に同じパーティション ID を共有します。</span><span class="sxs-lookup"><span data-stu-id="16e45-150">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="16e45-151">これにより、Databricks は 2 つのストリームを関連付けるときに、ある程度の並列処理を適用できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-151">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="16e45-152">乗車データのパーティション *n* 内のレコードは、料金データのパーティション *n* 内のレコードに対応します。</span><span class="sxs-lookup"><span data-stu-id="16e45-152">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Azure Databricks と Event Hubs によるストリーム処理のダイアグラム](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="16e45-154">データ ジェネレーターでは、両方のレコードの種類に対応した共通データ モデルに、`Medallion`、`HackLicense`、`VendorId` を連結した `PartitionKey` プロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-154">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="16e45-155">このプロパティを使用して、Event Hubs への送信時に明示的なパーティション キーが提供されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-155">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="16e45-156">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="16e45-156">Event Hubs</span></span>

<span data-ttu-id="16e45-157">Event Hubs のスループット容量は、[スループット ユニット](/azure/event-hubs/event-hubs-features#throughput-units)で測定されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-157">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="16e45-158">[自動インフレ](/azure/event-hubs/event-hubs-auto-inflate)を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="16e45-158">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="16e45-159">ストリーム処理</span><span class="sxs-lookup"><span data-stu-id="16e45-159">Stream processing</span></span>

<span data-ttu-id="16e45-160">Azure Databricks では、ジョブによってデータ処理が実行されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-160">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="16e45-161">ジョブはクラスターに割り当てられ、そこで実行されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-161">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="16e45-162">ジョブは、Java で記述されたカスタム コードか、Spark [Notebook](https://docs.databricks.com/user-guide/notebooks/index.html) です。</span><span class="sxs-lookup"><span data-stu-id="16e45-162">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="16e45-163">この参照アーキテクチャでは、ジョブは、Java と Scala の両方で記述されたクラスを含む Java アーカイブです。</span><span class="sxs-lookup"><span data-stu-id="16e45-163">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="16e45-164">Databricks ジョブの Java アーカイブを指定する場合、実行対象のクラスは Databricks クラスターによって指定されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-164">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="16e45-165">ここでは、**com.microsoft.pnp.TaxiCabReader** クラスの **main** メソッドに、データ処理ロジックが含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-165">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="16e45-166">2 つのイベント ハブ インスタンスからのストリームの読み取り</span><span class="sxs-lookup"><span data-stu-id="16e45-166">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="16e45-167">データ処理ロジックでは、2 つの Azure イベント ハブ インスタンスからの読み取りに、[Spark 構造化ストリーミング](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html)が使用されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-167">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="16e45-168">地域情報によるデータの強化</span><span class="sxs-lookup"><span data-stu-id="16e45-168">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="16e45-169">乗車データには、乗車場所と降車場所の緯度と経度の座標が含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-169">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="16e45-170">これらの座標は便利ですが、分析で使用するのは簡単ではありません。</span><span class="sxs-lookup"><span data-stu-id="16e45-170">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="16e45-171">したがって、このデータは、[シェープファイル](https://en.wikipedia.org/wiki/Shapefile)から読み取られる地域データで強化されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-171">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="16e45-172">シェープファイルはバイナリ形式で、簡単には解析されませんが、[GeoTools](http://geotools.org/) ライブラリには、シェープファイル形式を使用する地理空間データを対象としたツールが用意されています。</span><span class="sxs-lookup"><span data-stu-id="16e45-172">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="16e45-173">このライブラリは、乗車場所と降車場所の座標に基づいて地域名を判断するために、**com.microsoft.pnp.GeoFinder** クラスで使用されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-173">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="16e45-174">乗車データと料金データの結合</span><span class="sxs-lookup"><span data-stu-id="16e45-174">Joining the ride and fare data</span></span>

<span data-ttu-id="16e45-175">最初に、乗車データと料金データが変換されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-175">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="16e45-176">次に、乗車データが、料金データと結合されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-176">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="16e45-177">データの処理と Cosmos DB への挿入</span><span class="sxs-lookup"><span data-stu-id="16e45-177">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="16e45-178">指定された期間について、平均料金が地域ごとに計算されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-178">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="16e45-179">次に、これが Cosmos DB に挿入されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-179">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="16e45-180">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="16e45-180">Security considerations</span></span>

<span data-ttu-id="16e45-181">Azure Database ワークスペースへのアクセスは、[管理者コンソール](https://docs.databricks.com/administration-guide/admin-settings/index.html)を使用して制御されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-181">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="16e45-182">管理者コンソールには、ユーザーを追加する、ユーザーのアクセス許可を管理する、シングル サインオンを設定する、といった機能が含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-182">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="16e45-183">ワークスペース、クラスター、ジョブ、およびテーブルのアクセス制御も、管理者コンソールで設定できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-183">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="16e45-184">シークレットを管理する</span><span class="sxs-lookup"><span data-stu-id="16e45-184">Managing secrets</span></span>

<span data-ttu-id="16e45-185">Azure Databricks には[シークレット ストア](https://docs.azuredatabricks.net/user-guide/secrets/index.html)があり、これを使用して接続文字列、アクセス キー、ユーザー名、パスワードなどのシークレットを格納します。</span><span class="sxs-lookup"><span data-stu-id="16e45-185">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="16e45-186">Azure Databricks シークレット ストア内のシークレットは、**スコープ**ごとにパーティション分割されています。</span><span class="sxs-lookup"><span data-stu-id="16e45-186">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="16e45-187">シークレットは、スコープ レベルで追加されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-187">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="16e45-188">ネイティブの Azure Databricks スコープではなく、Azure Key Vault を実体とするスコープを使用できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-188">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="16e45-189">詳細については、「[Azure Key Vault-backed scopes (Azure Key Vault を実体とするスコープ)](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="16e45-189">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="16e45-190">コードでは、Azure Databricks [シークレット ユーティリティ](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities)を介してシークレットにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="16e45-190">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="16e45-191">監視に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="16e45-191">Monitoring considerations</span></span>

<span data-ttu-id="16e45-192">Azure Databricks は Apache Spark に基づいており、どちらも、ログ記録用の標準ライブラリとして [log4j](https://logging.apache.org/log4j/2.x/) を使用しています。</span><span class="sxs-lookup"><span data-stu-id="16e45-192">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="16e45-193">Apache Spark によって提供される既定のログ記録に加え、この参照アーキテクチャでは、ログとメトリックを [Azure Log Analytics](/azure/log-analytics/) に送信します。</span><span class="sxs-lookup"><span data-stu-id="16e45-193">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="16e45-194">**com.microsoft.pnp.TaxiCabReader** クラスでは、**log4j.properties** ファイルの値を使用して、Apache Spark のログを Azure Log Analytics に送信するように、Apache Spark ログ記録システムを構成します。</span><span class="sxs-lookup"><span data-stu-id="16e45-194">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="16e45-195">Apache Spark のロガー メッセージは文字列ですが、Azure Log Analytics では、ログ メッセージが JSON として書式設定されていなければなりません。</span><span class="sxs-lookup"><span data-stu-id="16e45-195">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="16e45-196">**com.microsoft.pnp.log4j.LogAnalyticsAppender** クラスは、これらのメッセージを JSON に変換します。</span><span class="sxs-lookup"><span data-stu-id="16e45-196">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="16e45-197">**com.microsoft.pnp.TaxiCabReader** クラスでは、乗車メッセージと料金メッセージが処理されるため、いずれかの形式が正しくない可能性があり、これが原因で無効になることがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-197">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="16e45-198">運用環境では、形式に誤りがあるこれらのメッセージを分析して、データ ソースの問題を特定することが重要です。これにより、その問題を迅速に修正して、データ損失を防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="16e45-198">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="16e45-199">**com.microsoft.pnp.TaxiCabReader** クラスにより、形式に誤りがある料金レコードと乗車レコードの数を追跡する Apache Spark アキュムレータが登録されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-199">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="16e45-200">Apache Spark は Dropwizard ライブラリを使用してメトリックを送信しますが、ネイティブ Dropwizard メトリック フィールドの中には、Azure Log Analytics と互換性がないものがあります。</span><span class="sxs-lookup"><span data-stu-id="16e45-200">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="16e45-201">このため、この参照アーキテクチャには、カスタム Dropwizard シンクおよびレポーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-201">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="16e45-202">これにより、メトリックは、Azure Log Analytics で予期される形式に書式設定されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-202">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="16e45-203">Apache Spark によってメトリックがレポートされると、形式に誤りがある乗車データと料金データに対するカスタム メトリックも送信されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-203">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="16e45-204">Azure Log Analytics ワークスペースに記録される最後のメトリックは、Spark Structured Streaming ジョブの累積的な進行状況です。</span><span class="sxs-lookup"><span data-stu-id="16e45-204">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="16e45-205">これは、**com.microsoft.pnp.StreamingMetricsListener** クラスで実装されるカスタム StreamingQuery リスナーを使用して、実行されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-205">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="16e45-206">このクラスは、ジョブの実行時に、Apache Spark セッションに登録されます。</span><span class="sxs-lookup"><span data-stu-id="16e45-206">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="16e45-207">StreamingMetricsListener 内のメソッドは、構造化ストリーミング イベントが発生すると必ず、Apache Spark ランタイムによって呼び出され、ログ メッセージとメトリックを Azure Log Analytics ワークスペースに送信します。</span><span class="sxs-lookup"><span data-stu-id="16e45-207">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="16e45-208">アプリケーションを監視するには、ワークスペースで次のクエリを使用とできます。</span><span class="sxs-lookup"><span data-stu-id="16e45-208">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="16e45-209">ストリーミング クエリの待機時間とスループット</span><span class="sxs-lookup"><span data-stu-id="16e45-209">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="16e45-210">ストリーム クエリの実行中に記録された例外</span><span class="sxs-lookup"><span data-stu-id="16e45-210">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="16e45-211">形式に誤りがある料金データと乗車データの蓄積</span><span class="sxs-lookup"><span data-stu-id="16e45-211">Accumulation of malformed fare and ride data</span></span>

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

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="16e45-212">回復性をトレースするジョブ実行</span><span class="sxs-lookup"><span data-stu-id="16e45-212">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a><span data-ttu-id="16e45-213">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="16e45-213">Deploy the solution</span></span>

<span data-ttu-id="16e45-214">この参照アーキテクチャのデプロイは、[GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics) で入手できます。</span><span class="sxs-lookup"><span data-stu-id="16e45-214">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="16e45-215">前提条件</span><span class="sxs-lookup"><span data-stu-id="16e45-215">Prerequisites</span></span>

1. <span data-ttu-id="16e45-216">「[Azure Databricks によるストリーム処理](https://github.com/mspnp/azure-databricks-streaming-analytics)」GitHub リポジトリを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="16e45-216">Clone, fork, or download the [stream processing with Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics) GitHub repository.</span></span>

2. <span data-ttu-id="16e45-217">[Docker](https://www.docker.com/) をインストールして、データ ジェネレーターを実行します。</span><span class="sxs-lookup"><span data-stu-id="16e45-217">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="16e45-218">[Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="16e45-218">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="16e45-219">[Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="16e45-219">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="16e45-220">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="16e45-220">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```shell
    az login
    ```
6. <span data-ttu-id="16e45-221">Java IDE と次のリソースをインストールします。</span><span class="sxs-lookup"><span data-stu-id="16e45-221">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="16e45-222">JDK 1.8</span><span class="sxs-lookup"><span data-stu-id="16e45-222">JDK 1.8</span></span>
    - <span data-ttu-id="16e45-223">Scala SDK 2.11</span><span class="sxs-lookup"><span data-stu-id="16e45-223">Scala SDK 2.11</span></span>
    - <span data-ttu-id="16e45-224">Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="16e45-224">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="16e45-225">ニューヨーク市のタクシー データ ファイルと地域データ ファイルをダウンロードする</span><span class="sxs-lookup"><span data-stu-id="16e45-225">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="16e45-226">ご自身のローカル ファイル システムで、複製された GitHub リポジトリのルートに `DataFile` という名前のディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="16e45-226">Create a directory named `DataFile` in the root of the cloned Github repository in your local file system.</span></span>

2. <span data-ttu-id="16e45-227">Web ブラウザーを開き、 https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935 に移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-227">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="16e45-228">このページの **[ダウンロード]** ボタンをクリックして、その年のすべてのタクシー データの ZIP ファイルをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="16e45-228">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="16e45-229">ZIP ファイルを `DataFile` ディレクトリに抽出します。</span><span class="sxs-lookup"><span data-stu-id="16e45-229">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="16e45-230">この ZIP ファイルには、他の ZIP ファイルが含まれています。</span><span class="sxs-lookup"><span data-stu-id="16e45-230">This zip file contains other zip files.</span></span> <span data-ttu-id="16e45-231">子 ZIP ファイルは抽出しないでください。</span><span class="sxs-lookup"><span data-stu-id="16e45-231">Don't extract the child zip files.</span></span>

    <span data-ttu-id="16e45-232">ディレクトリ構造は次のようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="16e45-232">The directory structure must look like the following:</span></span>

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. <span data-ttu-id="16e45-233">Web ブラウザーを開き、 https://www.zillow.com/howto/api/neighborhood-boundaries.htm に移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-233">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span>

6. <span data-ttu-id="16e45-234">**New York Neighborhood Boundaries** をクリックして、ファイルをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="16e45-234">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="16e45-235">お使いのブラウザーの**ダウンロード** ディレクトリにある **ZillowNeighborhoods-NY.zip** ファイルを、`DataFile` ディレクトリにコピーします。</span><span class="sxs-lookup"><span data-stu-id="16e45-235">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="16e45-236">Azure リソースをデプロイする</span><span class="sxs-lookup"><span data-stu-id="16e45-236">Deploy the Azure resources</span></span>

1. <span data-ttu-id="16e45-237">シェルまたは Windows コマンド プロンプトから次のコマンドを実行し、サインインプロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="16e45-237">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="16e45-238">GitHub リポジトリの `azure` という名前のフォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-238">Navigate to the folder named `azure` in the GitHub repository:</span></span>

    ```bash
    cd azure
    ```

3. <span data-ttu-id="16e45-239">次のコマンドを実行して、Azure リソースをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="16e45-239">Run the following commands to deploy the Azure resources:</span></span>

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

4. <span data-ttu-id="16e45-240">完了すると、デプロイの出力はコンソールに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="16e45-240">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="16e45-241">出力で次の JSON を探します。</span><span class="sxs-lookup"><span data-stu-id="16e45-241">Search the output for the following JSON:</span></span>

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

<span data-ttu-id="16e45-242">これらの値は、以降のセクションで Databricks シークレットに追加されるシークレットです。</span><span class="sxs-lookup"><span data-stu-id="16e45-242">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="16e45-243">これらのセクションで追加するまで、安全に保管してください。</span><span class="sxs-lookup"><span data-stu-id="16e45-243">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="16e45-244">Cassandra テーブルを Cosmos DB アカウントに追加する</span><span class="sxs-lookup"><span data-stu-id="16e45-244">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="16e45-245">Azure portal 上で、前の「**Azure リソースをデプロイする**」セクションで作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-245">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="16e45-246">**[Azure Cosmos DB アカウント]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-246">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="16e45-247">Cassandra API でテーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="16e45-247">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="16e45-248">**[概要]** ブレードで、**[テーブルの追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-248">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="16e45-249">**[テーブルの追加]** ブレードが開いたら、**[Keyspace name]\(キースペース名\)** テキスト ボックスに「`newyorktaxi`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-249">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span>

4. <span data-ttu-id="16e45-250">**[enter CQL command to create the table]\(テーブルを作成する CQL コマンドを入力\)** セクションで、`newyorktaxi` の横にあるテキスト ボックスに「`neighborhoodstats`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-250">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="16e45-251">下のテキスト ボックスに、次のように入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-251">In the text box below, enter the following:</span></span>
    ```shell
    (neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
    ```
6. <span data-ttu-id="16e45-252">**[スループット (1,000 - 1,000,000 RU/秒)]** テキスト ボックスに、値 `4000` を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-252">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="16e45-253">Click **OK**.</span><span class="sxs-lookup"><span data-stu-id="16e45-253">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="16e45-254">Databricks CLI を使用して Databricks シークレットを追加する</span><span class="sxs-lookup"><span data-stu-id="16e45-254">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="16e45-255">最初に、EventHub のシークレットを入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-255">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="16e45-256">前提条件の手順 2. でインストールした **Azure Databricks CLI** を使用して、Azure Databricks シークレット スコープを作成します。</span><span class="sxs-lookup"><span data-stu-id="16e45-256">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="16e45-257">タクシー乗車 EventHub のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="16e45-257">Add the secret for the taxi ride EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="16e45-258">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-258">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="16e45-259">「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-259">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-260">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="16e45-260">Save and exit vi.</span></span>

3. <span data-ttu-id="16e45-261">タクシー料金 EventHub のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="16e45-261">Add the secret for the taxi fare EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="16e45-262">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-262">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="16e45-263">「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-263">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-264">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="16e45-264">Save and exit vi.</span></span>

<span data-ttu-id="16e45-265">次に、Cosmos DB のシークレットを入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-265">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="16e45-266">Azure portal を開いて、「**Azure リソースをデプロイする**」セクションの手順 3. で指定したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-266">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="16e45-267">[Azure Cosmos DB アカウント] をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-267">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="16e45-268">**Azure Databricks CLI** を使用して、Cosmos DB ユーザー名のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="16e45-268">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="16e45-269">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-269">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="16e45-270">「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **username** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-270">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-271">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="16e45-271">Save and exit vi.</span></span>

3. <span data-ttu-id="16e45-272">次に、Cosmos DB パスワードのシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="16e45-272">Next, add the secret for the Cosmos DB password:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="16e45-273">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-273">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="16e45-274">「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **secret** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-274">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-275">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="16e45-275">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="16e45-276">[Azure Key Vault を実体とするシークレット スコープ](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)を使用している場合、スコープの名前は **azure-databricks-job** にする必要があります。また、シークレットの名前は、上記とまったく同じにしなければなりません。</span><span class="sxs-lookup"><span data-stu-id="16e45-276">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="16e45-277">Zillow 地域データ ファイルを Databricks ファイル システムに追加する</span><span class="sxs-lookup"><span data-stu-id="16e45-277">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="16e45-278">Databricks ファイル システムでディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="16e45-278">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="16e45-279">`DataFile` ディレクトリに移動し、次を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-279">Navigate to the `DataFile` directory and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="16e45-280">Azure Log Analytics ワークスペースの ID と主キーを構成ファイルに追加する</span><span class="sxs-lookup"><span data-stu-id="16e45-280">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="16e45-281">このセクションでは、Log Analytics ワークスペースの ID と主キーが必要です。</span><span class="sxs-lookup"><span data-stu-id="16e45-281">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="16e45-282">ワークスペース ID は、「*Azure リソースをデプロイする*」セクションの手順 4. の **logAnalytics** 出力セクションに示されている **workspaceId** 値です。</span><span class="sxs-lookup"><span data-stu-id="16e45-282">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-283">主キーは、その出力セクションの **secret** です。</span><span class="sxs-lookup"><span data-stu-id="16e45-283">The primary key is the **secret** from the output section.</span></span>

1. <span data-ttu-id="16e45-284">log4j ログ記録を構成するには、`\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties` を開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-284">To configure log4j logging, open `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`.</span></span> <span data-ttu-id="16e45-285">次の 2 つの値を編集します。</span><span class="sxs-lookup"><span data-stu-id="16e45-285">Edit the following two values:</span></span>
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="16e45-286">カスタム ログ記録を構成するには、`\azure\azure-databricks-monitoring\scripts\metrics.properties` を開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-286">To configure custom logging, open `\azure\azure-databricks-monitoring\scripts\metrics.properties`.</span></span> <span data-ttu-id="16e45-287">次の 2 つの値を編集します。</span><span class="sxs-lookup"><span data-stu-id="16e45-287">Edit the following two values:</span></span>
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="16e45-288">Databricks ジョブおよび Databricks 監視用の .jar ファイルをビルドする</span><span class="sxs-lookup"><span data-stu-id="16e45-288">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="16e45-289">ご自身の Java IDE を使用して、ルート ディレクトリにある、**pom.xml** という名前の Maven プロジェクト ファイルをインポートします。</span><span class="sxs-lookup"><span data-stu-id="16e45-289">Use your Java IDE to import the Maven project file named **pom.xml** located in the root directory.</span></span>

2. <span data-ttu-id="16e45-290">クリーン ビルドを実行します。</span><span class="sxs-lookup"><span data-stu-id="16e45-290">Perform a clean build.</span></span> <span data-ttu-id="16e45-291">このビルドの出力は、**azure-databricks-job-1.0-SNAPSHOT.jar** および **azure-databricks-monitoring-0.9.jar** という名前のファイルです。</span><span class="sxs-lookup"><span data-stu-id="16e45-291">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span>

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="16e45-292">Databricks ジョブのカスタム ログ記録を構成する</span><span class="sxs-lookup"><span data-stu-id="16e45-292">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="16e45-293">**Databricks CLI** で次のコマンドを入力して、**azure-databricks-monitoring-0.9.jar** ファイルを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="16e45-293">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="16e45-294">次のコマンドを入力して、`\azure\azure-databricks-monitoring\scripts\metrics.properties` のカスタム ログ記録プロパティを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="16e45-294">Copy the custom logging properties from `\azure\azure-databricks-monitoring\scripts\metrics.properties` to the Databricks file system by entering the following command:</span></span>
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="16e45-295">Databricks クラスターの名前をまだ決めていませんが、ここでそれを選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-295">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="16e45-296">お使いのクラスターの Databricks ファイル システム パスに、以下の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-296">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="16e45-297">次のコマンドを入力して、`\azure\azure-databricks-monitoring\scripts\spark.metrics` の初期化スクリプトを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="16e45-297">Copy the initialization script from `\azure\azure-databricks-monitoring\scripts\spark.metrics` to the Databricks file system by entering the following command:</span></span>
    ```shell
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="16e45-298">Databricks クラスターを作成する</span><span class="sxs-lookup"><span data-stu-id="16e45-298">Create a Databricks cluster</span></span>

1. <span data-ttu-id="16e45-299">Databricks ワークスペースで、[クラスター]、[クラスターの作成] の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-299">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="16e45-300">前の「**Databricks ジョブのカスタム ログ記録を構成する**」セクションの手順 3. で作成したクラスター名を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-300">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span>

2. <span data-ttu-id="16e45-301">**[標準]** クラスター モードを選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-301">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="16e45-302">**[Databricks runtime version]\(Databricks Runtime のバージョン\)** を **[4.3 (includes Apache Spark 2.3.1, Scala 2.11)]\(4.3 (Apache Spark 2.3.1、Scala 2.11 を含む)\)** に設定します</span><span class="sxs-lookup"><span data-stu-id="16e45-302">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="16e45-303">**[Python バージョン]** を **[2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="16e45-303">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="16e45-304">**[ドライバーの種類]** を **[Same as worker]\(worker と同じ\)** に設定します</span><span class="sxs-lookup"><span data-stu-id="16e45-304">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="16e45-305">**[ワーカー タイプ]** を **[Standard_DS3_v2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="16e45-305">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="16e45-306">**[Min Workers]\(最小 worker 数\)** を **[2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="16e45-306">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="16e45-307">**[Enable autoscaling]\(自動スケールを有効にする\)** の選択を解除します。</span><span class="sxs-lookup"><span data-stu-id="16e45-307">Deselect **Enable autoscaling**.</span></span>

9. <span data-ttu-id="16e45-308">**[Auto Termination]\(自動的に終了\)** ダイアログ ボックスで、**[Init Scripts]\(初期化スクリプト\)** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-308">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span>

10. <span data-ttu-id="16e45-309">「**dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**」と入力します。<cluster-name> には、手順 1. で作成したクラスター名を指定します。</span><span class="sxs-lookup"><span data-stu-id="16e45-309">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="16e45-310">**[追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-310">Click the **Add** button.</span></span>

12. <span data-ttu-id="16e45-311">**[クラスターの作成]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-311">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="16e45-312">Databricks ジョブを作成する</span><span class="sxs-lookup"><span data-stu-id="16e45-312">Create a Databricks job</span></span>

1. <span data-ttu-id="16e45-313">Databricks ワークスペースで、[ジョブ]、[ジョブの作成] の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-313">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="16e45-314">ジョブ名を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-314">Enter a job name.</span></span>

3. <span data-ttu-id="16e45-315">[set jar]\(jar の設定\) をクリックします。これにより、[Upload JAR to Run]\(実行する JAR のアップロード\) ダイアログ ボックスが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-315">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="16e45-316">「**Databricks ジョブ用の .jar ファイルをビルドする**」セクションで作成した **azure-databricks-job-1.0-SNAPSHOT.jar** ファイルを、**[Drop JAR here to upload]\(アップロードする JAR をここにドロップ\)** ボックスにドラッグします。</span><span class="sxs-lookup"><span data-stu-id="16e45-316">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="16e45-317">**[メイン クラス]** フィールドに「**com.microsoft.pnp.TaxiCabReader**」と入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-317">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="16e45-318">引数フィールドに、次を入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-318">In the arguments field, enter the following:</span></span>
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above>
    ```

7. <span data-ttu-id="16e45-319">次の手順に従って、依存するライブラリをインストールします。</span><span class="sxs-lookup"><span data-stu-id="16e45-319">Install the dependent libraries by following these steps:</span></span>

    1. <span data-ttu-id="16e45-320">Databricks ユーザー インターフェイスで、**[ホーム]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-320">In the Databricks user interface, click on the **home** button.</span></span>

    2. <span data-ttu-id="16e45-321">**[ユーザー]** ドロップダウンで、ご自身のユーザー アカウント名をクリックして、お使いのアカウント ワークスペースの設定を開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-321">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>

    3. <span data-ttu-id="16e45-322">ご自身のアカウント名の横にあるドロップダウン矢印をクリックし、**[作成]**、**[ライブラリ]** の順にクリックして、**[新しいライブラリ]** ダイアログを開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-322">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>

    4. <span data-ttu-id="16e45-323">**[ソース]** ドロップダウン コントロールで、**[Maven Coordinate]\(Maven 座標\)** を選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-323">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>

    5. <span data-ttu-id="16e45-324">**[Install Maven Artifacts]\(Maven Artifacts のインストール\)** 見出しで、**[Coordinate]\(座標\)** テキスト ボックスに「`com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-324">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span>

    6. <span data-ttu-id="16e45-325">**[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-325">Click on **Create Library** to open the **Artifacts** window.</span></span>

    7. <span data-ttu-id="16e45-326">**[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。</span><span class="sxs-lookup"><span data-stu-id="16e45-326">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

    8. <span data-ttu-id="16e45-327">`com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven 座標について、手順 1. から 7. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="16e45-327">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>

    9. <span data-ttu-id="16e45-328">`org.geotools:gt-shapefile:19.2` Maven 座標について、手順 1. から 6. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="16e45-328">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>

    10. <span data-ttu-id="16e45-329">**[詳細オプション]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-329">Click on **Advanced Options**.</span></span>

    11. <span data-ttu-id="16e45-330">**[リポジトリ]** テキスト ボックスに「`http://download.osgeo.org/webdav/geotools/`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="16e45-330">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span>

    12. <span data-ttu-id="16e45-331">**[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-331">Click **Create Library** to open the **Artifacts** window.</span></span> 

    13. <span data-ttu-id="16e45-332">**[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。</span><span class="sxs-lookup"><span data-stu-id="16e45-332">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="16e45-333">手順 7. で追加した依存ライブラリを、手順 6. の最後に作成したジョブに追加します。</span><span class="sxs-lookup"><span data-stu-id="16e45-333">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>

    1. <span data-ttu-id="16e45-334">Azure Databricks ワークスペースで、**[ジョブ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-334">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="16e45-335">「**Databricks ジョブを作成する**」セクションの手順 2. で作成されたジョブ名をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-335">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span>

    3. <span data-ttu-id="16e45-336">**[Dependent Libraries]\(依存ライブラリ\)** セクションの横で、**[追加]** をクリックして、**[Add Dependent Library]\(依存ライブラリの追加\)** ダイアログを開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-336">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span>

    4. <span data-ttu-id="16e45-337">**[Library From]\(ライブラリの選択元\)** で、**[ワークスペース]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-337">Under **Library From** select **Workspace**.</span></span>

    5. <span data-ttu-id="16e45-338">**[ユーザー]**、ご自身のユーザー名、`azure-eventhubs-spark_2.11:2.3.5` の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-338">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span>

    6. <span data-ttu-id="16e45-339">Click **OK**.</span><span class="sxs-lookup"><span data-stu-id="16e45-339">Click **OK**.</span></span>

    7. <span data-ttu-id="16e45-340">`spark-cassandra-connector_2.11:2.3.1` および `gt-shapefile:19.2` について、手順 1. から 6. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="16e45-340">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="16e45-341">**[クラスター]** の横で、**[編集]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-341">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="16e45-342">これにより、**[クラスターの構成]** ダイアログが開きます。</span><span class="sxs-lookup"><span data-stu-id="16e45-342">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="16e45-343">**[クラスターの種類]** ドロップダウンで、**[Existing Cluster]\(既存のクラスター\)** を選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-343">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="16e45-344">**[クラスターの選択]** ドロップダウンで、「**Databricks クラスターを作成する**」セクションで作成されたクラスターを選択します。</span><span class="sxs-lookup"><span data-stu-id="16e45-344">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="16e45-345">**[確認]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-345">Click **confirm**.</span></span>

10. <span data-ttu-id="16e45-346">**[今すぐ実行]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="16e45-346">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="16e45-347">データ ジェネレーターを実行する</span><span class="sxs-lookup"><span data-stu-id="16e45-347">Run the data generator</span></span>

1. <span data-ttu-id="16e45-348">GitHub リポジトリの `onprem` という名前のディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-348">Navigate to the directory named `onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="16e45-349">次のように、**main.env** ファイル内の値を更新します。</span><span class="sxs-lookup"><span data-stu-id="16e45-349">Update the values in the file **main.env** as follows:</span></span>

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="16e45-350">taxi-ride イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値です。</span><span class="sxs-lookup"><span data-stu-id="16e45-350">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="16e45-351">taxi-fare イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値です。</span><span class="sxs-lookup"><span data-stu-id="16e45-351">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="16e45-352">次のコマンドを実行して、Docker イメージをビルドします。</span><span class="sxs-lookup"><span data-stu-id="16e45-352">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="16e45-353">親ディレクトリに戻ります。</span><span class="sxs-lookup"><span data-stu-id="16e45-353">Navigate back to the parent directory.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="16e45-354">次のコマンドを実行して、Docker イメージを実行します。</span><span class="sxs-lookup"><span data-stu-id="16e45-354">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="16e45-355">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="16e45-355">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="16e45-356">Databricks ジョブが正しく実行されていることを確認するには、Azure portal を開き、Cosmos DB データベースに移動します。</span><span class="sxs-lookup"><span data-stu-id="16e45-356">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="16e45-357">**[データ エクスプローラー]** ブレードを開いて、**taxi records** テーブルでデータを確認します。</span><span class="sxs-lookup"><span data-stu-id="16e45-357">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="16e45-358">[1] <span id="note1">Donovan, Brian; Work, Dan (2016):New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="16e45-358">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="16e45-359">イリノイ大学アーバナシャンペーン校。</span><span class="sxs-lookup"><span data-stu-id="16e45-359">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="16e45-360">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="16e45-360">https://doi.org/10.13012/J8PN93H8</span></span>
