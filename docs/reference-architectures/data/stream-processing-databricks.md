---
title: Azure Databricks によるストリーム処理
description: Azure Databricks を使用して、Azure でエンド ツー エンドのストリーム処理パイプラインを作成します
author: petertaylor9999
ms.date: 11/01/2018
ms.openlocfilehash: a7e9df57572c9b3a3b0e4f418f148449aa40b04c
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295735"
---
# <a name="stream-processing-with-azure-databricks"></a><span data-ttu-id="57c07-103">Azure Databricks によるストリーム処理</span><span class="sxs-lookup"><span data-stu-id="57c07-103">Stream processing with Azure Databricks</span></span>

<span data-ttu-id="57c07-104">この参照アーキテクチャでは、エンド ツー エンドの[ストリーム処理](/azure/architecture/data-guide/big-data/real-time-processing)パイプラインを示します。</span><span class="sxs-lookup"><span data-stu-id="57c07-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="57c07-105">この種類のパイプラインには、取り込み、処理、格納、および分析とレポート作成の 4 つの段階があります。</span><span class="sxs-lookup"><span data-stu-id="57c07-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="57c07-106">この参照アーキテクチャでは、パイプラインは、2 つのソースからデータを取り込み、各ストリームの関連するレコードに対して結合を実行し、結果を強化させ、リアルタイムで平均を計算します。</span><span class="sxs-lookup"><span data-stu-id="57c07-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="57c07-107">結果が保存され、さらに詳しい分析が行われます。</span><span class="sxs-lookup"><span data-stu-id="57c07-107">The results are stored for further analysis.</span></span> [<span data-ttu-id="57c07-108">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="57c07-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

<span data-ttu-id="57c07-109">**シナリオ**: タクシー会社が各乗車に関するデータを収集しています。</span><span class="sxs-lookup"><span data-stu-id="57c07-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="57c07-110">このシナリオでは、データを送信する 2 つのデバイスがあることを想定しています。</span><span class="sxs-lookup"><span data-stu-id="57c07-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="57c07-111">タクシーには、各乗車の情報 (走行時間、距離、乗車場所と降車場所) を送信するメーターがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="57c07-112">別のデバイスでは、乗客からの支払いを受け付け、料金に関するデータを送信します。</span><span class="sxs-lookup"><span data-stu-id="57c07-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="57c07-113">利用者数の傾向をつかむために、このタクシー会社では、各地で走行 1 マイルあたりの平均チップをリアルタイムで計算したいと考えています。</span><span class="sxs-lookup"><span data-stu-id="57c07-113">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="57c07-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="57c07-114">Architecture</span></span>

<span data-ttu-id="57c07-115">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="57c07-116">**データ ソース**。</span><span class="sxs-lookup"><span data-stu-id="57c07-116">**Data sources**.</span></span> <span data-ttu-id="57c07-117">このアーキテクチャには、リアルタイムでデータ ストリームを生成する 2 つのデータ ソースがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="57c07-118">1 つ目のストリームには乗車情報が含まれ、2 つ目のストリームには料金情報が含まれます。</span><span class="sxs-lookup"><span data-stu-id="57c07-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="57c07-119">参照アーキテクチャには、一連の静的ファイルから読み取り、データを Event Hubs にプッシュするシミュレートされたデータ ジェネレーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="57c07-120">実際のアプリケーションのデータ ソースは、タクシーに設置されたデバイスになります。</span><span class="sxs-lookup"><span data-stu-id="57c07-120">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="57c07-121">**Azure Event Hubs**。</span><span class="sxs-lookup"><span data-stu-id="57c07-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="57c07-122">[Event Hubs](/azure/event-hubs/) はイベント取り込みサービスです。</span><span class="sxs-lookup"><span data-stu-id="57c07-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="57c07-123">このアーキテクチャでは、2 つのイベント ハブ インスタンス (データ ソースごとに 1 つ) を使用します。</span><span class="sxs-lookup"><span data-stu-id="57c07-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="57c07-124">各データ ソースは、関連付けられたイベント ハブにデータ ストリームを送信します。</span><span class="sxs-lookup"><span data-stu-id="57c07-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="57c07-125">**Azure Databricks**。</span><span class="sxs-lookup"><span data-stu-id="57c07-125">**Azure Databricks**.</span></span> <span data-ttu-id="57c07-126">[Databricks](/azure/azure-databricks/) は、Microsoft Azure クラウド サービス プラットフォーム用に最適化された Apache Spark ベースの分析プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="57c07-126">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="57c07-127">Databricks を使用して、タクシーの乗車データと料金データを関連付け、さらに、その関連付けられたデータを、Databricks ファイル システムに格納されている地域データで強化します。</span><span class="sxs-lookup"><span data-stu-id="57c07-127">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="57c07-128">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="57c07-128">**Cosmos DB**.</span></span> <span data-ttu-id="57c07-129">Azure Databricks ジョブの出力は一連のレコードであり、Cassandra API を使用して [Cosmos DB](/azure/cosmos-db/) に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="57c07-129">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="57c07-130">Cassandra API が使用されるのは、時系列データ モデリングをサポートしているためです。</span><span class="sxs-lookup"><span data-stu-id="57c07-130">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="57c07-131">**Azure Log Analytics**。</span><span class="sxs-lookup"><span data-stu-id="57c07-131">**Azure Log Analytics**.</span></span> <span data-ttu-id="57c07-132">[Azure Monitor](/azure/monitoring-and-diagnostics/) で収集されたアプリケーション ログ データは、[Log Analytics ワークスペース](/azure/log-analytics)に保存されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-132">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="57c07-133">Log Analytics クエリを使用してメトリックを分析および視覚化し、ログ メッセージを検査してアプリケーション内の問題を特定できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-133">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="57c07-134">データの取り込み</span><span class="sxs-lookup"><span data-stu-id="57c07-134">Data ingestion</span></span>

<span data-ttu-id="57c07-135">データ ソースをシミュレートするために、この参照アーキテクチャでは [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) データセット<sup>[[1]](#note1)</sup> を使用します。</span><span class="sxs-lookup"><span data-stu-id="57c07-135">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="57c07-136">このデータセットには、ニューヨーク市の 4 年間 (2010 年から 2013 年) のタクシー乗車に関するデータが含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-136">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="57c07-137">乗車データと料金データの 2 種類のレコードがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-137">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="57c07-138">乗車データには、走行時間、乗車距離、乗車場所と降車場所が含まれます。</span><span class="sxs-lookup"><span data-stu-id="57c07-138">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="57c07-139">料金データには、料金、税、チップの金額が含まれます。</span><span class="sxs-lookup"><span data-stu-id="57c07-139">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="57c07-140">この 2 種類のレコードの共通フィールドには、営業許可番号、タクシー免許、ベンダー ID があります。</span><span class="sxs-lookup"><span data-stu-id="57c07-140">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="57c07-141">この 3 つのフィールドを組み合わせて、タクシーと運転手が一意に識別されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-141">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="57c07-142">データは CSV 形式で保存されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-142">The data is stored in CSV format.</span></span> 

<span data-ttu-id="57c07-143">データ ジェネレーターは、レコードを読み取り、Azure Event Hubs に送信する .NET Core アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="57c07-143">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="57c07-144">ジェネレーターは、JSON 形式の乗車データと CSV 形式の料金データを送信します。</span><span class="sxs-lookup"><span data-stu-id="57c07-144">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="57c07-145">Event Hubs では、[パーティション](/azure/event-hubs/event-hubs-features#partitions)を使用してデータをセグメント化します。</span><span class="sxs-lookup"><span data-stu-id="57c07-145">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="57c07-146">複数のパーティションでは、コンシューマーは各パーティションを並列で読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="57c07-146">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="57c07-147">Event Hubs にデータを送信するときに、パーティション キーを明示的に指定できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-147">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="57c07-148">それ以外の場合は、ラウンド ロビン方式でパーティションにレコードが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="57c07-148">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="57c07-149">このシナリオでは、特定のタクシーの乗車データと料金データは、最終的に同じパーティション ID を共有します。</span><span class="sxs-lookup"><span data-stu-id="57c07-149">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="57c07-150">これにより、Databricks は 2 つのストリームを関連付けるときに、ある程度の並列処理を適用できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-150">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="57c07-151">乗車データのパーティション *n* 内のレコードは、料金データのパーティション *n* 内のレコードに対応します。</span><span class="sxs-lookup"><span data-stu-id="57c07-151">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="57c07-152">データ ジェネレーターでは、両方のレコードの種類に対応した共通データ モデルに、`Medallion`、`HackLicense`、`VendorId` を連結した `PartitionKey` プロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-152">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="57c07-153">このプロパティを使用して、Event Hubs への送信時に明示的なパーティション キーが提供されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-153">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="57c07-154">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="57c07-154">Event Hubs</span></span>

<span data-ttu-id="57c07-155">Event Hubs のスループット容量は、[スループット ユニット](/azure/event-hubs/event-hubs-features#throughput-units)で測定されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-155">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="57c07-156">[自動インフレ](/azure/event-hubs/event-hubs-auto-inflate)を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="57c07-156">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

## <a name="stream-processing"></a><span data-ttu-id="57c07-157">ストリーム処理</span><span class="sxs-lookup"><span data-stu-id="57c07-157">Stream processing</span></span>

<span data-ttu-id="57c07-158">Azure Databricks では、ジョブによってデータ処理が実行されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-158">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="57c07-159">ジョブはクラスターに割り当てられ、そこで実行されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-159">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="57c07-160">ジョブは、Java で記述されたカスタム コードか、Spark [Notebook](https://docs.databricks.com/user-guide/notebooks/index.html) です。</span><span class="sxs-lookup"><span data-stu-id="57c07-160">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="57c07-161">この参照アーキテクチャでは、ジョブは、Java と Scala の両方で記述されたクラスを含む Java アーカイブです。</span><span class="sxs-lookup"><span data-stu-id="57c07-161">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="57c07-162">Databricks ジョブの Java アーカイブを指定する場合、実行対象のクラスは Databricks クラスターによって指定されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-162">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="57c07-163">ここでは、**com.microsoft.pnp.TaxiCabReader** クラスの **main** メソッドに、データ処理ロジックが含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-163">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span> 

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="57c07-164">2 つのイベント ハブ インスタンスからのストリームの読み取り</span><span class="sxs-lookup"><span data-stu-id="57c07-164">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="57c07-165">データ処理ロジックでは、2 つの Azure イベント ハブ インスタンスからの読み取りに、[Spark 構造化ストリーミング](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html)が使用されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-165">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="57c07-166">地域情報によるデータの強化</span><span class="sxs-lookup"><span data-stu-id="57c07-166">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="57c07-167">乗車データには、乗車場所と降車場所の緯度と経度の座標が含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-167">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="57c07-168">これらの座標は便利ですが、分析で使用するのは簡単ではありません。</span><span class="sxs-lookup"><span data-stu-id="57c07-168">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="57c07-169">したがって、このデータは、[シェープファイル](https://en.wikipedia.org/wiki/Shapefile)から読み取られる地域データで強化されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-169">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span> 

<span data-ttu-id="57c07-170">シェープファイルはバイナリ形式で、簡単には解析されませんが、[GeoTools](http://geotools.org/) ライブラリには、シェープファイル形式を使用する地理空間データを対象としたツールが用意されています。</span><span class="sxs-lookup"><span data-stu-id="57c07-170">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="57c07-171">このライブラリは、乗車場所と降車場所の座標に基づいて地域名を判断するために、**com.microsoft.pnp.GeoFinder** クラスで使用されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-171">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span> 

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="57c07-172">乗車データと料金データの結合</span><span class="sxs-lookup"><span data-stu-id="57c07-172">Joining the ride and fare data</span></span>

<span data-ttu-id="57c07-173">最初に、乗車データと料金データが変換されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-173">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="57c07-174">次に、乗車データが、料金データと結合されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-174">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="57c07-175">データの処理と Cosmos DB への挿入</span><span class="sxs-lookup"><span data-stu-id="57c07-175">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="57c07-176">指定された期間について、平均料金が地域ごとに計算されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-176">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="57c07-177">次に、これが Cosmos DB に挿入されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-177">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="57c07-178">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="57c07-178">Security considerations</span></span>

<span data-ttu-id="57c07-179">Azure Database ワークスペースへのアクセスは、[管理者コンソール](https://docs.databricks.com/administration-guide/admin-settings/index.html)を使用して制御されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-179">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="57c07-180">管理者コンソールには、ユーザーを追加する、ユーザーのアクセス許可を管理する、シングル サインオンを設定する、といった機能が含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-180">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="57c07-181">ワークスペース、クラスター、ジョブ、およびテーブルのアクセス制御も、管理者コンソールで設定できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-181">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="57c07-182">シークレットを管理する</span><span class="sxs-lookup"><span data-stu-id="57c07-182">Managing secrets</span></span>

<span data-ttu-id="57c07-183">Azure Databricks には[シークレット ストア](https://docs.azuredatabricks.net/user-guide/secrets/index.html)があり、これを使用して接続文字列、アクセス キー、ユーザー名、パスワードなどのシークレットを格納します。</span><span class="sxs-lookup"><span data-stu-id="57c07-183">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="57c07-184">Azure Databricks シークレット ストア内のシークレットは、**スコープ**ごとにパーティション分割されています。</span><span class="sxs-lookup"><span data-stu-id="57c07-184">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="57c07-185">シークレットは、スコープ レベルで追加されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-185">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="57c07-186">ネイティブの Azure Databricks スコープではなく、Azure Key Vault を実体とするスコープを使用できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-186">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="57c07-187">詳細については、「[Azure Key Vault-backed scopes (Azure Key Vault を実体とするスコープ)](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="57c07-187">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="57c07-188">コードでは、Azure Databricks [シークレット ユーティリティ](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities)を介してシークレットにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="57c07-188">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>


## <a name="monitoring-considerations"></a><span data-ttu-id="57c07-189">監視に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="57c07-189">Monitoring considerations</span></span>

<span data-ttu-id="57c07-190">Azure Databricks は Apache Spark に基づいており、どちらも、ログ記録用の標準ライブラリとして [log4j](https://logging.apache.org/log4j/2.x/) を使用しています。</span><span class="sxs-lookup"><span data-stu-id="57c07-190">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="57c07-191">Apache Spark によって提供される既定のログ記録に加え、この参照アーキテクチャでは、ログとメトリックを [Azure Log Analytics](/azure/log-analytics/) に送信します。</span><span class="sxs-lookup"><span data-stu-id="57c07-191">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="57c07-192">**com.microsoft.pnp.TaxiCabReader** クラスでは、**log4j.properties** ファイルの値を使用して、Apache Spark のログを Azure Log Analytics に送信するように、Apache Spark ログ記録システムを構成します。</span><span class="sxs-lookup"><span data-stu-id="57c07-192">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="57c07-193">Apache Spark のロガー メッセージは文字列ですが、Azure Log Analytics では、ログ メッセージが JSON として書式設定されていなければなりません。</span><span class="sxs-lookup"><span data-stu-id="57c07-193">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="57c07-194">**com.microsoft.pnp.log4j.LogAnalyticsAppender** クラスは、これらのメッセージを JSON に変換します。</span><span class="sxs-lookup"><span data-stu-id="57c07-194">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="57c07-195">**com.microsoft.pnp.TaxiCabReader** クラスでは、乗車メッセージと料金メッセージが処理されるため、いずれかの形式が正しくない可能性があり、これが原因で無効になることがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-195">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="57c07-196">運用環境では、形式に誤りがあるこれらのメッセージを分析して、データ ソースの問題を特定することが重要です。これにより、その問題を迅速に修正して、データ損失を防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="57c07-196">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="57c07-197">**com.microsoft.pnp.TaxiCabReader** クラスにより、形式に誤りがある料金レコードと乗車レコードの数を追跡する Apache Spark アキュムレータが登録されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-197">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="57c07-198">Apache Spark は Dropwizard ライブラリを使用してメトリックを送信しますが、ネイティブ Dropwizard メトリック フィールドの中には、Azure Log Analytics と互換性がないものがあります。</span><span class="sxs-lookup"><span data-stu-id="57c07-198">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="57c07-199">このため、この参照アーキテクチャには、カスタム Dropwizard シンクおよびレポーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-199">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="57c07-200">これにより、メトリックは、Azure Log Analytics で予期される形式に書式設定されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-200">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="57c07-201">Apache Spark によってメトリックがレポートされると、形式に誤りがある乗車データと料金データに対するカスタム メトリックも送信されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-201">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="57c07-202">Azure Log Analytics ワークスペースに記録される最後のメトリックは、Spark Structured Streaming ジョブの累積的な進行状況です。</span><span class="sxs-lookup"><span data-stu-id="57c07-202">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="57c07-203">これは、**com.microsoft.pnp.StreamingMetricsListener** クラスで実装されるカスタム StreamingQuery リスナーを使用して、実行されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-203">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="57c07-204">このクラスは、ジョブの実行時に、Apache Spark セッションに登録されます。</span><span class="sxs-lookup"><span data-stu-id="57c07-204">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="57c07-205">StreamingMetricsListener 内のメソッドは、構造化ストリーミング イベントが発生すると必ず、Apache Spark ランタイムによって呼び出され、ログ メッセージとメトリックを Azure Log Analytics ワークスペースに送信します。</span><span class="sxs-lookup"><span data-stu-id="57c07-205">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="57c07-206">アプリケーションを監視するには、ワークスペースで次のクエリを使用とできます。</span><span class="sxs-lookup"><span data-stu-id="57c07-206">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="57c07-207">ストリーミング クエリの待機時間とスループット</span><span class="sxs-lookup"><span data-stu-id="57c07-207">Latency and throughput for streaming queries</span></span> 

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="57c07-208">ストリーム クエリの実行中に記録された例外</span><span class="sxs-lookup"><span data-stu-id="57c07-208">Exceptions logged during stream query execution</span></span>

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="57c07-209">形式に誤りがある料金データと乗車データの蓄積</span><span class="sxs-lookup"><span data-stu-id="57c07-209">Accumulation of malformed fare and ride data</span></span>

```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedrides"

SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedfares" 
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="57c07-210">回復性をトレースするジョブ実行</span><span class="sxs-lookup"><span data-stu-id="57c07-210">Job execution to trace resiliency</span></span>
```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a><span data-ttu-id="57c07-211">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="57c07-211">Deploy the solution</span></span>

<span data-ttu-id="57c07-212">この参照アーキテクチャのデプロイは、[GitHub](https://github.com/mspnp/reference-architectures/tree/master/data) で入手できます。</span><span class="sxs-lookup"><span data-stu-id="57c07-212">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="57c07-213">前提条件</span><span class="sxs-lookup"><span data-stu-id="57c07-213">Prerequisites</span></span>

1. <span data-ttu-id="57c07-214">[参照アーキテクチャ](https://github.com/mspnp/reference-architectures) GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="57c07-214">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="57c07-215">[Docker](https://www.docker.com/) をインストールして、データ ジェネレーターを実行します。</span><span class="sxs-lookup"><span data-stu-id="57c07-215">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="57c07-216">[Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="57c07-216">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="57c07-217">[Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="57c07-217">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="57c07-218">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="57c07-218">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```
    az login
    ```
6. <span data-ttu-id="57c07-219">Java IDE と次のリソースをインストールします。</span><span class="sxs-lookup"><span data-stu-id="57c07-219">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="57c07-220">JDK 1.8</span><span class="sxs-lookup"><span data-stu-id="57c07-220">JDK 1.8</span></span>
    - <span data-ttu-id="57c07-221">Scala SDK 2.11</span><span class="sxs-lookup"><span data-stu-id="57c07-221">Scala SDK 2.11</span></span>
    - <span data-ttu-id="57c07-222">Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="57c07-222">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="57c07-223">ニューヨーク市のタクシー データ ファイルと地域データ ファイルをダウンロードする</span><span class="sxs-lookup"><span data-stu-id="57c07-223">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="57c07-224">ご自身のローカル ファイル システムで、`data/streaming_azuredatabricks` ディレクトリの下に `DataFile` という名前のディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="57c07-224">Create a directory named `DataFile` under the `data/streaming_azuredatabricks` directory in your local file system.</span></span>

2. <span data-ttu-id="57c07-225">Web ブラウザーを開き、 https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935 に移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-225">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="57c07-226">このページの **[ダウンロード]** ボタンをクリックして、その年のすべてのタクシー データの ZIP ファイルをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="57c07-226">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="57c07-227">ZIP ファイルを `DataFile` ディレクトリに抽出します。</span><span class="sxs-lookup"><span data-stu-id="57c07-227">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="57c07-228">この ZIP ファイルには、他の ZIP ファイルが含まれています。</span><span class="sxs-lookup"><span data-stu-id="57c07-228">This zip file contains other zip files.</span></span> <span data-ttu-id="57c07-229">子 ZIP ファイルは抽出しないでください。</span><span class="sxs-lookup"><span data-stu-id="57c07-229">Don't extract the child zip files.</span></span>

    <span data-ttu-id="57c07-230">ディレクトリ構造は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="57c07-230">The directory structure should look like the following:</span></span>

    ```
    /data
        /streaming_azuredatabricks
            /DataFile
                /FOIL2013
                    trip_data_1.zip
                    trip_data_2.zip
                    trip_data_3.zip
                    ...
    ```

5. <span data-ttu-id="57c07-231">Web ブラウザーを開き、 https://www.zillow.com/howto/api/neighborhood-boundaries.htm に移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-231">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span> 

6. <span data-ttu-id="57c07-232">**New York Neighborhood Boundaries** をクリックして、ファイルをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="57c07-232">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="57c07-233">お使いのブラウザーの**ダウンロード** ディレクトリにある **ZillowNeighborhoods-NY.zip** ファイルを、`DataFile` ディレクトリにコピーします。</span><span class="sxs-lookup"><span data-stu-id="57c07-233">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="57c07-234">Azure リソースをデプロイする</span><span class="sxs-lookup"><span data-stu-id="57c07-234">Deploy the Azure resources</span></span>

1. <span data-ttu-id="57c07-235">シェルまたは Windows コマンド プロンプトから次のコマンドを実行し、サインインプロンプトに従います。</span><span class="sxs-lookup"><span data-stu-id="57c07-235">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="57c07-236">GitHub リポジトリの `data/streaming_azuredatabricks` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-236">Navigate to the folder `data/streaming_azuredatabricks` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_azuredatabricks
    ```

3. <span data-ttu-id="57c07-237">次のコマンドを実行して、Azure リソースをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="57c07-237">Run the following commands to deploy the Azure resources:</span></span>

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
        --template-file ./azure/deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. <span data-ttu-id="57c07-238">完了すると、デプロイの出力はコンソールに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="57c07-238">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="57c07-239">出力で次の JSON を探します。</span><span class="sxs-lookup"><span data-stu-id="57c07-239">Search the output for the following JSON:</span></span>

```JSON
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
<span data-ttu-id="57c07-240">これらの値は、以降のセクションで Databricks シークレットに追加されるシークレットです。</span><span class="sxs-lookup"><span data-stu-id="57c07-240">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="57c07-241">これらのセクションで追加するまで、安全に保管してください。</span><span class="sxs-lookup"><span data-stu-id="57c07-241">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="57c07-242">Cassandra テーブルを Cosmos DB アカウントに追加する</span><span class="sxs-lookup"><span data-stu-id="57c07-242">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="57c07-243">Azure portal 上で、前の「**Azure リソースをデプロイする**」セクションで作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-243">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="57c07-244">**[Azure Cosmos DB アカウント]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-244">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="57c07-245">Cassandra API でテーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="57c07-245">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="57c07-246">**[概要]** ブレードで、**[テーブルの追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-246">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="57c07-247">**[テーブルの追加]** ブレードが開いたら、**[Keyspace name]\(キースペース名\)** テキスト ボックスに「`newyorktaxi`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-247">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span> 

4. <span data-ttu-id="57c07-248">**[enter CQL command to create the table]\(テーブルを作成する CQL コマンドを入力\)** セクションで、`newyorktaxi` の横にあるテキスト ボックスに「`neighborhoodstats`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-248">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="57c07-249">下のテキスト ボックスに、次を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-249">In the text box below, enter the following::</span></span>
```
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. <span data-ttu-id="57c07-250">**[スループット (1,000 - 1,000,000 RU/秒)]** テキスト ボックスに、値 `4000` を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-250">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="57c07-251">Click **OK**.</span><span class="sxs-lookup"><span data-stu-id="57c07-251">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="57c07-252">Databricks CLI を使用して Databricks シークレットを追加する</span><span class="sxs-lookup"><span data-stu-id="57c07-252">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="57c07-253">最初に、EventHub のシークレットを入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-253">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="57c07-254">前提条件の手順 2. でインストールした **Azure Databricks CLI** を使用して、Azure Databricks シークレット スコープを作成します。</span><span class="sxs-lookup"><span data-stu-id="57c07-254">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="57c07-255">タクシー乗車 EventHub のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="57c07-255">Add the secret for the taxi ride EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="57c07-256">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-256">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="57c07-257">「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-257">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-258">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="57c07-258">Save and exit vi.</span></span>

3. <span data-ttu-id="57c07-259">タクシー料金 EventHub のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="57c07-259">Add the secret for the taxi fare EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="57c07-260">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-260">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="57c07-261">「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-261">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-262">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="57c07-262">Save and exit vi.</span></span>

<span data-ttu-id="57c07-263">次に、Cosmos DB のシークレットを入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-263">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="57c07-264">Azure portal を開いて、「**Azure リソースをデプロイする**」セクションの手順 3. で指定したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-264">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="57c07-265">[Azure Cosmos DB アカウント] をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-265">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="57c07-266">**Azure Databricks CLI** を使用して、Cosmos DB ユーザー名のシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="57c07-266">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="57c07-267">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-267">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="57c07-268">「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **username** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-268">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-269">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="57c07-269">Save and exit vi.</span></span>

3. <span data-ttu-id="57c07-270">次に、Cosmos DB パスワードのシークレットを追加します。</span><span class="sxs-lookup"><span data-stu-id="57c07-270">Next, add the secret for the Cosmos DB password:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="57c07-271">実行されると、このコマンドによって vi エディターが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-271">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="57c07-272">「*Azure リソースをデプロイする*」セクションの手順 4. の **CosmosDb** 出力セクションに示されている **secret** 値を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-272">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-273">保存して vi を終了します。</span><span class="sxs-lookup"><span data-stu-id="57c07-273">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="57c07-274">[Azure Key Vault を実体とするシークレット スコープ](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)を使用している場合、スコープの名前は **azure-databricks-job** にする必要があります。また、シークレットの名前は、上記とまったく同じにしなければなりません。</span><span class="sxs-lookup"><span data-stu-id="57c07-274">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="57c07-275">Zillow 地域データ ファイルを Databricks ファイル システムに追加する</span><span class="sxs-lookup"><span data-stu-id="57c07-275">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="57c07-276">Databricks ファイル システムでディレクトリを作成します。</span><span class="sxs-lookup"><span data-stu-id="57c07-276">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="57c07-277">data/streaming_azuredatabricks/DataFile に移動し、次を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-277">Navigate to data/streaming_azuredatabricks/DataFile and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="57c07-278">Azure Log Analytics ワークスペースの ID と主キーを構成ファイルに追加する</span><span class="sxs-lookup"><span data-stu-id="57c07-278">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="57c07-279">このセクションでは、Log Analytics ワークスペースの ID と主キーが必要です。</span><span class="sxs-lookup"><span data-stu-id="57c07-279">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="57c07-280">ワークスペース ID は、「*Azure リソースをデプロイする*」セクションの手順 4. の **logAnalytics** 出力セクションに示されている **workspaceId** 値です。</span><span class="sxs-lookup"><span data-stu-id="57c07-280">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-281">主キーは、その出力セクションの **secret** です。</span><span class="sxs-lookup"><span data-stu-id="57c07-281">The primary key is the **secret** from the output section.</span></span> 

1. <span data-ttu-id="57c07-282">log4j ログ記録を構成するには、data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties を開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-282">To configure log4j logging, open data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties.</span></span> <span data-ttu-id="57c07-283">次の 2 つの値を編集します。</span><span class="sxs-lookup"><span data-stu-id="57c07-283">Edit the following two values:</span></span>
    ```
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="57c07-284">カスタム ログ記録を構成するには、data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties を開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-284">To configure custom logging, open data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties.</span></span> <span data-ttu-id="57c07-285">次の 2 つの値を編集します。</span><span class="sxs-lookup"><span data-stu-id="57c07-285">Edit the following two values:</span></span>
    ``` 
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="57c07-286">Databricks ジョブおよび Databricks 監視用の .jar ファイルをビルドする</span><span class="sxs-lookup"><span data-stu-id="57c07-286">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="57c07-287">お使いの Java IDE を使用して、**data/streaming_azuredatabricks** ディレクトリのルートにある、**pom.xml** という名前の Maven プロジェクト ファイルをインポートします。</span><span class="sxs-lookup"><span data-stu-id="57c07-287">Use your Java IDE to import the Maven project file named **pom.xml** located in the root of the **data/streaming_azuredatabricks** directory.</span></span> 

2. <span data-ttu-id="57c07-288">クリーン ビルドを実行します。</span><span class="sxs-lookup"><span data-stu-id="57c07-288">Perform a clean build.</span></span> <span data-ttu-id="57c07-289">このビルドの出力は、**azure-databricks-job-1.0-SNAPSHOT.jar** および **azure-databricks-monitoring-0.9.jar** という名前のファイルです。</span><span class="sxs-lookup"><span data-stu-id="57c07-289">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span> 

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="57c07-290">Databricks ジョブのカスタム ログ記録を構成する</span><span class="sxs-lookup"><span data-stu-id="57c07-290">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="57c07-291">**Databricks CLI** で次のコマンドを入力して、**azure-databricks-monitoring-0.9.jar** ファイルを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="57c07-291">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="57c07-292">次のコマンドを入力して、data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties のカスタム ログ記録プロパティを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="57c07-292">Copy the custom logging properties from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="57c07-293">Databricks クラスターの名前をまだ決めていませんが、ここでそれを選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-293">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="57c07-294">お使いのクラスターの Databricks ファイル システム パスに、以下の名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-294">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="57c07-295">次のコマンドを入力して、data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics の初期化スクリプトを Databricks ファイル システムにコピーします。</span><span class="sxs-lookup"><span data-stu-id="57c07-295">Copy the initialization script from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="57c07-296">Databricks クラスターを作成する</span><span class="sxs-lookup"><span data-stu-id="57c07-296">Create a Databricks cluster</span></span>

1. <span data-ttu-id="57c07-297">Databricks ワークスペースで、[クラスター]、[クラスターの作成] の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-297">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="57c07-298">前の「**Databricks ジョブのカスタム ログ記録を構成する**」セクションの手順 3. で作成したクラスター名を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-298">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span> 

2. <span data-ttu-id="57c07-299">**[標準]** クラスター モードを選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-299">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="57c07-300">**[Databricks runtime version]\(Databricks Runtime のバージョン\)** を **[4.3 (includes Apache Spark 2.3.1, Scala 2.11)]\(4.3 (Apache Spark 2.3.1、Scala 2.11 を含む)\)** に設定します</span><span class="sxs-lookup"><span data-stu-id="57c07-300">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="57c07-301">**[Python バージョン]** を **[2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="57c07-301">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="57c07-302">**[ドライバーの種類]** を **[Same as worker]\(worker と同じ\)** に設定します</span><span class="sxs-lookup"><span data-stu-id="57c07-302">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="57c07-303">**[ワーカー タイプ]** を **[Standard_DS3_v2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="57c07-303">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="57c07-304">**[Min Workers]\(最小 worker 数\)** を **[2]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="57c07-304">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="57c07-305">**[Enable autoscaling]\(自動スケールを有効にする\)** の選択を解除します。</span><span class="sxs-lookup"><span data-stu-id="57c07-305">Deselect **Enable autoscaling**.</span></span> 

9. <span data-ttu-id="57c07-306">**[Auto Termination]\(自動的に終了\)** ダイアログ ボックスで、**[Init Scripts]\(初期化スクリプト\)** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-306">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span> 

10. <span data-ttu-id="57c07-307">「**dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**」と入力します。<cluster-name> には、手順 1. で作成したクラスター名を指定します。</span><span class="sxs-lookup"><span data-stu-id="57c07-307">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="57c07-308">**[追加]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-308">Click the **Add** button.</span></span>

12. <span data-ttu-id="57c07-309">**[クラスターの作成]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-309">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="57c07-310">Databricks ジョブを作成する</span><span class="sxs-lookup"><span data-stu-id="57c07-310">Create a Databricks job</span></span>

1. <span data-ttu-id="57c07-311">Databricks ワークスペースで、[ジョブ]、[ジョブの作成] の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-311">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="57c07-312">ジョブ名を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-312">Enter a job name.</span></span>

3. <span data-ttu-id="57c07-313">[set jar]\(jar の設定\) をクリックします。これにより、[Upload JAR to Run]\(実行する JAR のアップロード\) ダイアログ ボックスが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-313">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="57c07-314">「**Databricks ジョブ用の .jar ファイルをビルドする**」セクションで作成した **azure-databricks-job-1.0-SNAPSHOT.jar** ファイルを、**[Drop JAR here to upload]\(アップロードする JAR をここにドロップ\)** ボックスにドラッグします。</span><span class="sxs-lookup"><span data-stu-id="57c07-314">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="57c07-315">**[メイン クラス]** フィールドに「**com.microsoft.pnp.TaxiCabReader**」と入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-315">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="57c07-316">引数フィールドに、次を入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-316">In the arguments field, enter the following:</span></span>
    ```
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. <span data-ttu-id="57c07-317">次の手順に従って、依存するライブラリをインストールします。</span><span class="sxs-lookup"><span data-stu-id="57c07-317">Install the dependent libraries by following these steps:</span></span>
    
    1. <span data-ttu-id="57c07-318">Databricks ユーザー インターフェイスで、**[ホーム]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-318">In the Databricks user interface, click on the **home** button.</span></span>
    
    2. <span data-ttu-id="57c07-319">**[ユーザー]** ドロップダウンで、ご自身のユーザー アカウント名をクリックして、お使いのアカウント ワークスペースの設定を開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-319">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>
    
    3. <span data-ttu-id="57c07-320">ご自身のアカウント名の横にあるドロップダウン矢印をクリックし、**[作成]**、**[ライブラリ]** の順にクリックして、**[新しいライブラリ]** ダイアログを開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-320">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>
    
    4. <span data-ttu-id="57c07-321">**[ソース]** ドロップダウン コントロールで、**[Maven Coordinate]\(Maven 座標\)** を選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-321">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>
    
    5. <span data-ttu-id="57c07-322">**[Install Maven Artifacts]\(Maven Artifacts のインストール\)** 見出しで、**[Coordinate]\(座標\)** テキスト ボックスに「`com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-322">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span> 
    
    6. <span data-ttu-id="57c07-323">**[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-323">Click on **Create Library** to open the **Artifacts** window.</span></span>
    
    7. <span data-ttu-id="57c07-324">**[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。</span><span class="sxs-lookup"><span data-stu-id="57c07-324">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>
    
    8. <span data-ttu-id="57c07-325">`com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven 座標について、手順 1. から 7. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="57c07-325">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>
    
    9. <span data-ttu-id="57c07-326">`org.geotools:gt-shapefile:19.2` Maven 座標について、手順 1. から 6. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="57c07-326">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>
    
    10. <span data-ttu-id="57c07-327">**[詳細オプション]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-327">Click on **Advanced Options**.</span></span>
    
    11. <span data-ttu-id="57c07-328">**[リポジトリ]** テキスト ボックスに「`http://download.osgeo.org/webdav/geotools/`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="57c07-328">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span> 
    
    12. <span data-ttu-id="57c07-329">**[ライブラリの作成]** をクリックして、**[アーティファクト]** ウィンドウを開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-329">Click **Create Library** to open the **Artifacts** window.</span></span> 
    
    13. <span data-ttu-id="57c07-330">**[Status on running clusters]\(実行中のクラスターの状態\)** で、**[Attach automatically to all clusters]\(すべてのクラスターに自動的に接続する\)** チェック ボックスをオンします。</span><span class="sxs-lookup"><span data-stu-id="57c07-330">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="57c07-331">手順 7. で追加した依存ライブラリを、手順 6. の最後に作成したジョブに追加します。</span><span class="sxs-lookup"><span data-stu-id="57c07-331">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>
    1. <span data-ttu-id="57c07-332">Azure Databricks ワークスペースで、**[ジョブ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-332">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="57c07-333">「**Databricks ジョブを作成する**」セクションの手順 2. で作成されたジョブ名をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-333">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span> 
    
    3. <span data-ttu-id="57c07-334">**[Dependent Libraries]\(依存ライブラリ\)** セクションの横で、**[追加]** をクリックして、**[Add Dependent Library]\(依存ライブラリの追加\)** ダイアログを開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-334">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span> 
    
    4. <span data-ttu-id="57c07-335">**[Library From]\(ライブラリの選択元\)** で、**[ワークスペース]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-335">Under **Library From** select **Workspace**.</span></span>
    
    5. <span data-ttu-id="57c07-336">**[ユーザー]**、ご自身のユーザー名、`azure-eventhubs-spark_2.11:2.3.5` の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-336">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span> 
    
    6. <span data-ttu-id="57c07-337">Click **OK**.</span><span class="sxs-lookup"><span data-stu-id="57c07-337">Click **OK**.</span></span>
    
    7. <span data-ttu-id="57c07-338">`spark-cassandra-connector_2.11:2.3.1` および `gt-shapefile:19.2` について、手順 1. から 6. を繰り返します。</span><span class="sxs-lookup"><span data-stu-id="57c07-338">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="57c07-339">**[クラスター]** の横で、**[編集]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-339">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="57c07-340">これにより、**[クラスターの構成]** ダイアログが開きます。</span><span class="sxs-lookup"><span data-stu-id="57c07-340">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="57c07-341">**[クラスターの種類]** ドロップダウンで、**[Existing Cluster]\(既存のクラスター\)** を選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-341">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="57c07-342">**[クラスターの選択]** ドロップダウンで、「**Databricks クラスターを作成する**」セクションで作成されたクラスターを選択します。</span><span class="sxs-lookup"><span data-stu-id="57c07-342">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="57c07-343">**[確認]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-343">Click **confirm**.</span></span>

10. <span data-ttu-id="57c07-344">**[今すぐ実行]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="57c07-344">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="57c07-345">データ ジェネレーターを実行する</span><span class="sxs-lookup"><span data-stu-id="57c07-345">Run the data generator</span></span>

1. <span data-ttu-id="57c07-346">GitHub リポジトリの `data/streaming_azuredatabricks/onprem` ディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-346">Navigate to the directory `data/streaming_azuredatabricks/onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="57c07-347">次のように、**main.env** ファイル内の値を更新します。</span><span class="sxs-lookup"><span data-stu-id="57c07-347">Update the values in the file **main.env** as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="57c07-348">taxi-ride イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-ride-eh** 値です。</span><span class="sxs-lookup"><span data-stu-id="57c07-348">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="57c07-349">taxi-fare イベント ハブの接続文字列は、「*Azure リソースをデプロイする*」セクションの手順 4. の **eventHubs** 出力セクションに示されている **taxi-fare-eh** 値です。</span><span class="sxs-lookup"><span data-stu-id="57c07-349">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="57c07-350">次のコマンドを実行して、Docker イメージをビルドします。</span><span class="sxs-lookup"><span data-stu-id="57c07-350">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="57c07-351">親ディレクトリ `data/stream_azuredatabricks` に戻ります。</span><span class="sxs-lookup"><span data-stu-id="57c07-351">Navigate back to the parent directory, `data/stream_azuredatabricks`.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="57c07-352">次のコマンドを実行して、Docker イメージを実行します。</span><span class="sxs-lookup"><span data-stu-id="57c07-352">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="57c07-353">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="57c07-353">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="57c07-354">Databricks ジョブが正しく実行されていることを確認するには、Azure portal を開き、Cosmos DB データベースに移動します。</span><span class="sxs-lookup"><span data-stu-id="57c07-354">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="57c07-355">**[データ エクスプローラー]** ブレードを開いて、**taxi records** テーブルでデータを確認します。</span><span class="sxs-lookup"><span data-stu-id="57c07-355">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="57c07-356">[1] <span id="note1">Brian Donovan、Dan Work (2016 年): New York City Taxi Trip Data (2010-2013)。</span><span class="sxs-lookup"><span data-stu-id="57c07-356">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="57c07-357">イリノイ大学アーバナシャンペーン校。</span><span class="sxs-lookup"><span data-stu-id="57c07-357">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="57c07-358">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="57c07-358">https://doi.org/10.13012/J8PN93H8</span></span>
