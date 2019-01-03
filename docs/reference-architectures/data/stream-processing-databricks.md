---
title: Azure Databricks によるストリーム処理
titleSuffix: Azure Reference Architectures
description: Azure Databricks を使用して、Azure でエンド ツー エンドのストリーム処理パイプラインを作成します。
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: f7364334f889388ad432efadd46362a9fa82fe8b
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644123"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="73a62-103">Azure Databricks を使用してストリーム処理パイプラインを作成します</span><span class="sxs-lookup"><span data-stu-id="73a62-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="73a62-104">この参照アーキテクチャでは、エンド ツー エンドの[ストリーム処理](/azure/architecture/data-guide/big-data/real-time-processing)パイプラインを示します。</span><span class="sxs-lookup"><span data-stu-id="73a62-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="73a62-105">この種類のパイプラインには、取り込み、処理、格納、および分析とレポート作成の 4 つの段階があります。</span><span class="sxs-lookup"><span data-stu-id="73a62-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="73a62-106">この参照アーキテクチャでは、パイプラインは、2 つのソースからデータを取り込み、各ストリームの関連するレコードに対して結合を実行し、結果を強化させ、リアルタイムで平均を計算します。</span><span class="sxs-lookup"><span data-stu-id="73a62-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="73a62-107">結果が保存され、さらに詳しい分析が行われます。</span><span class="sxs-lookup"><span data-stu-id="73a62-107">The results are stored for further analysis.</span></span> <span data-ttu-id="73a62-108">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="73a62-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Azure Databricks によるストリーム処理の参照アーキテクチャ](./images/stream-processing-databricks.png)

<span data-ttu-id="73a62-110">**シナリオ**:タクシー会社が各乗車に関するデータを収集しています。</span><span class="sxs-lookup"><span data-stu-id="73a62-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="73a62-111">このシナリオでは、データを送信する 2 つのデバイスがあることを想定しています。</span><span class="sxs-lookup"><span data-stu-id="73a62-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="73a62-112">タクシーには、各乗車の情報 (走行時間、距離、乗車場所と降車場所) を送信するメーターがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="73a62-113">別のデバイスでは、乗客からの支払いを受け付け、料金に関するデータを送信します。</span><span class="sxs-lookup"><span data-stu-id="73a62-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="73a62-114">利用者数の傾向をつかむために、このタクシー会社では、各地で走行 1 マイルあたりの平均チップをリアルタイムで計算したいと考えています。</span><span class="sxs-lookup"><span data-stu-id="73a62-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="73a62-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="73a62-115">Architecture</span></span>

<span data-ttu-id="73a62-116">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="73a62-117">**データ ソース**。</span><span class="sxs-lookup"><span data-stu-id="73a62-117">**Data sources**.</span></span> <span data-ttu-id="73a62-118">このアーキテクチャには、リアルタイムでデータ ストリームを生成する 2 つのデータ ソースがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="73a62-119">1 つ目のストリームには乗車情報が含まれ、2 つ目のストリームには料金情報が含まれます。</span><span class="sxs-lookup"><span data-stu-id="73a62-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="73a62-120">参照アーキテクチャには、一連の静的ファイルから読み取り、データを Event Hubs にプッシュするシミュレートされたデータ ジェネレーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="73a62-121">実際のアプリケーションのデータ ソースは、タクシーに設置されたデバイスになります。</span><span class="sxs-lookup"><span data-stu-id="73a62-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="73a62-122">**Azure Event Hubs**。</span><span class="sxs-lookup"><span data-stu-id="73a62-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="73a62-123">[Event Hubs](/azure/event-hubs/) はイベント取り込みサービスです。</span><span class="sxs-lookup"><span data-stu-id="73a62-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="73a62-124">このアーキテクチャでは、2 つのイベント ハブ インスタンス (データ ソースごとに 1 つ) を使用します。</span><span class="sxs-lookup"><span data-stu-id="73a62-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="73a62-125">各データ ソースは、関連付けられたイベント ハブにデータ ストリームを送信します。</span><span class="sxs-lookup"><span data-stu-id="73a62-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="73a62-126">**Azure Databricks**。</span><span class="sxs-lookup"><span data-stu-id="73a62-126">**Azure Databricks**.</span></span> <span data-ttu-id="73a62-127">[Databricks](/azure/azure-databricks/) は、Microsoft Azure クラウド サービス プラットフォーム用に最適化された Apache Spark ベースの分析プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="73a62-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="73a62-128">Databricks を使用して、タクシーの乗車データと料金データを関連付け、さらに、その関連付けられたデータを、Databricks ファイル システムに格納されている地域データで強化します。</span><span class="sxs-lookup"><span data-stu-id="73a62-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="73a62-129">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="73a62-129">**Cosmos DB**.</span></span> <span data-ttu-id="73a62-130">Azure Databricks ジョブの出力は一連のレコードであり、Cassandra API を使用して [Cosmos DB](/azure/cosmos-db/) に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="73a62-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="73a62-131">Cassandra API が使用されるのは、時系列データ モデリングをサポートしているためです。</span><span class="sxs-lookup"><span data-stu-id="73a62-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="73a62-132">**Azure Log Analytics**。</span><span class="sxs-lookup"><span data-stu-id="73a62-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="73a62-133">[Azure Monitor](/azure/monitoring-and-diagnostics/) で収集されたアプリケーション ログ データは、[Log Analytics ワークスペース](/azure/log-analytics)に保存されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="73a62-134">Log Analytics クエリを使用してメトリックを分析および視覚化し、ログ メッセージを検査してアプリケーション内の問題を特定できます。</span><span class="sxs-lookup"><span data-stu-id="73a62-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="73a62-135">データの取り込み</span><span class="sxs-lookup"><span data-stu-id="73a62-135">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="73a62-136">データ ソースをシミュレートするために、この参照アーキテクチャでは [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) データセット<sup>[[1]](#note1)</sup> を使用します。</span><span class="sxs-lookup"><span data-stu-id="73a62-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="73a62-137">このデータセットには、ニューヨーク市の 4 年間 (2010 年から 2013 年) のタクシー乗車に関するデータが含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="73a62-138">乗車データと料金データの 2 種類のレコードがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="73a62-139">乗車データには、走行時間、乗車距離、乗車場所と降車場所が含まれます。</span><span class="sxs-lookup"><span data-stu-id="73a62-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="73a62-140">料金データには、料金、税、チップの金額が含まれます。</span><span class="sxs-lookup"><span data-stu-id="73a62-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="73a62-141">この 2 種類のレコードの共通フィールドには、営業許可番号、タクシー免許、ベンダー ID があります。</span><span class="sxs-lookup"><span data-stu-id="73a62-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="73a62-142">この 3 つのフィールドを組み合わせて、タクシーと運転手が一意に識別されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="73a62-143">データは CSV 形式で保存されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-143">The data is stored in CSV format.</span></span>

> <span data-ttu-id="73a62-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016):New York City Taxi Trip Data (2010-2013).</span><span class="sxs-lookup"><span data-stu-id="73a62-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="73a62-145">イリノイ大学アーバナシャンペーン校。</span><span class="sxs-lookup"><span data-stu-id="73a62-145">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="73a62-146">データ ジェネレーターは、レコードを読み取り、Azure Event Hubs に送信する .NET Core アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="73a62-146">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="73a62-147">ジェネレーターは、JSON 形式の乗車データと CSV 形式の料金データを送信します。</span><span class="sxs-lookup"><span data-stu-id="73a62-147">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="73a62-148">Event Hubs では、[パーティション](/azure/event-hubs/event-hubs-features#partitions)を使用してデータをセグメント化します。</span><span class="sxs-lookup"><span data-stu-id="73a62-148">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="73a62-149">複数のパーティションでは、コンシューマーは各パーティションを並列で読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="73a62-149">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="73a62-150">Event Hubs にデータを送信するときに、パーティション キーを明示的に指定できます。</span><span class="sxs-lookup"><span data-stu-id="73a62-150">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="73a62-151">それ以外の場合は、ラウンド ロビン方式でパーティションにレコードが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="73a62-151">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="73a62-152">このシナリオでは、特定のタクシーの乗車データと料金データは、最終的に同じパーティション ID を共有します。</span><span class="sxs-lookup"><span data-stu-id="73a62-152">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="73a62-153">これにより、Databricks は 2 つのストリームを関連付けるときに、ある程度の並列処理を適用できます。</span><span class="sxs-lookup"><span data-stu-id="73a62-153">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="73a62-154">乗車データのパーティション *n* 内のレコードは、料金データのパーティション *n* 内のレコードに対応します。</span><span class="sxs-lookup"><span data-stu-id="73a62-154">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Azure Databricks と Event Hubs によるストリーム処理のダイアグラム](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="73a62-156">データ ジェネレーターでは、両方のレコードの種類に対応した共通データ モデルに、`Medallion`、`HackLicense`、`VendorId` を連結した `PartitionKey` プロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-156">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="73a62-157">このプロパティを使用して、Event Hubs への送信時に明示的なパーティション キーが提供されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="73a62-158">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="73a62-158">Event Hubs</span></span>

<span data-ttu-id="73a62-159">Event Hubs のスループット容量は、[スループット ユニット](/azure/event-hubs/event-hubs-features#throughput-units)で測定されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-159">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="73a62-160">[自動インフレ](/azure/event-hubs/event-hubs-auto-inflate)を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="73a62-160">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="73a62-161">ストリーム処理</span><span class="sxs-lookup"><span data-stu-id="73a62-161">Stream processing</span></span>

<span data-ttu-id="73a62-162">Azure Databricks では、ジョブによってデータ処理が実行されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-162">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="73a62-163">ジョブはクラスターに割り当てられ、そこで実行されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-163">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="73a62-164">ジョブは、Java で記述されたカスタム コードか、Spark [Notebook](https://docs.databricks.com/user-guide/notebooks/index.html) です。</span><span class="sxs-lookup"><span data-stu-id="73a62-164">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="73a62-165">この参照アーキテクチャでは、ジョブは、Java と Scala の両方で記述されたクラスを含む Java アーカイブです。</span><span class="sxs-lookup"><span data-stu-id="73a62-165">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="73a62-166">Databricks ジョブの Java アーカイブを指定する場合、実行対象のクラスは Databricks クラスターによって指定されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-166">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="73a62-167">ここでは、**com.microsoft.pnp.TaxiCabReader** クラスの **main** メソッドに、データ処理ロジックが含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-167">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="73a62-168">2 つのイベント ハブ インスタンスからのストリームの読み取り</span><span class="sxs-lookup"><span data-stu-id="73a62-168">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="73a62-169">データ処理ロジックでは、2 つの Azure イベント ハブ インスタンスからの読み取りに、[Spark 構造化ストリーミング](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html)が使用されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-169">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="73a62-170">地域情報によるデータの強化</span><span class="sxs-lookup"><span data-stu-id="73a62-170">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="73a62-171">乗車データには、乗車場所と降車場所の緯度と経度の座標が含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-171">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="73a62-172">これらの座標は便利ですが、分析で使用するのは簡単ではありません。</span><span class="sxs-lookup"><span data-stu-id="73a62-172">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="73a62-173">したがって、このデータは、[シェープファイル](https://en.wikipedia.org/wiki/Shapefile)から読み取られる地域データで強化されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-173">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="73a62-174">シェープファイルはバイナリ形式で、簡単には解析されませんが、[GeoTools](http://geotools.org/) ライブラリには、シェープファイル形式を使用する地理空間データを対象としたツールが用意されています。</span><span class="sxs-lookup"><span data-stu-id="73a62-174">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="73a62-175">このライブラリは、乗車場所と降車場所の座標に基づいて地域名を判断するために、**com.microsoft.pnp.GeoFinder** クラスで使用されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-175">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="73a62-176">乗車データと料金データの結合</span><span class="sxs-lookup"><span data-stu-id="73a62-176">Joining the ride and fare data</span></span>

<span data-ttu-id="73a62-177">最初に、乗車データと料金データが変換されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-177">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="73a62-178">次に、乗車データが、料金データと結合されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-178">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="73a62-179">データの処理と Cosmos DB への挿入</span><span class="sxs-lookup"><span data-stu-id="73a62-179">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="73a62-180">指定された期間について、平均料金が地域ごとに計算されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-180">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="73a62-181">次に、これが Cosmos DB に挿入されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-181">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="73a62-182">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="73a62-182">Security considerations</span></span>

<span data-ttu-id="73a62-183">Azure Database ワークスペースへのアクセスは、[管理者コンソール](https://docs.databricks.com/administration-guide/admin-settings/index.html)を使用して制御されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-183">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="73a62-184">管理者コンソールには、ユーザーを追加する、ユーザーのアクセス許可を管理する、シングル サインオンを設定する、といった機能が含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-184">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="73a62-185">ワークスペース、クラスター、ジョブ、およびテーブルのアクセス制御も、管理者コンソールで設定できます。</span><span class="sxs-lookup"><span data-stu-id="73a62-185">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="73a62-186">シークレットを管理する</span><span class="sxs-lookup"><span data-stu-id="73a62-186">Managing secrets</span></span>

<span data-ttu-id="73a62-187">Azure Databricks には[シークレット ストア](https://docs.azuredatabricks.net/user-guide/secrets/index.html)があり、これを使用して接続文字列、アクセス キー、ユーザー名、パスワードなどのシークレットを格納します。</span><span class="sxs-lookup"><span data-stu-id="73a62-187">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="73a62-188">Azure Databricks シークレット ストア内のシークレットは、**スコープ**ごとにパーティション分割されています。</span><span class="sxs-lookup"><span data-stu-id="73a62-188">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="73a62-189">シークレットは、スコープ レベルで追加されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-189">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="73a62-190">ネイティブの Azure Databricks スコープではなく、Azure Key Vault を実体とするスコープを使用できます。</span><span class="sxs-lookup"><span data-stu-id="73a62-190">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="73a62-191">詳細については、「[Azure Key Vault-backed scopes (Azure Key Vault を実体とするスコープ)](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="73a62-191">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="73a62-192">コードでは、Azure Databricks [シークレット ユーティリティ](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities)を介してシークレットにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="73a62-192">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="73a62-193">監視に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="73a62-193">Monitoring considerations</span></span>

<span data-ttu-id="73a62-194">Azure Databricks は Apache Spark に基づいており、どちらも、ログ記録用の標準ライブラリとして [log4j](https://logging.apache.org/log4j/2.x/) を使用しています。</span><span class="sxs-lookup"><span data-stu-id="73a62-194">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="73a62-195">Apache Spark によって提供される既定のログ記録に加え、この参照アーキテクチャでは、ログとメトリックを [Azure Log Analytics](/azure/log-analytics/) に送信します。</span><span class="sxs-lookup"><span data-stu-id="73a62-195">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="73a62-196">**com.microsoft.pnp.TaxiCabReader** クラスでは、**log4j.properties** ファイルの値を使用して、Apache Spark のログを Azure Log Analytics に送信するように、Apache Spark ログ記録システムを構成します。</span><span class="sxs-lookup"><span data-stu-id="73a62-196">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="73a62-197">Apache Spark のロガー メッセージは文字列ですが、Azure Log Analytics では、ログ メッセージが JSON として書式設定されていなければなりません。</span><span class="sxs-lookup"><span data-stu-id="73a62-197">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="73a62-198">**com.microsoft.pnp.log4j.LogAnalyticsAppender** クラスは、これらのメッセージを JSON に変換します。</span><span class="sxs-lookup"><span data-stu-id="73a62-198">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="73a62-199">**com.microsoft.pnp.TaxiCabReader** クラスでは、乗車メッセージと料金メッセージが処理されるため、いずれかの形式が正しくない可能性があり、これが原因で無効になることがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-199">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="73a62-200">運用環境では、形式に誤りがあるこれらのメッセージを分析して、データ ソースの問題を特定することが重要です。これにより、その問題を迅速に修正して、データ損失を防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="73a62-200">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="73a62-201">**com.microsoft.pnp.TaxiCabReader** クラスにより、形式に誤りがある料金レコードと乗車レコードの数を追跡する Apache Spark アキュムレータが登録されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-201">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="73a62-202">Apache Spark は Dropwizard ライブラリを使用してメトリックを送信しますが、ネイティブ Dropwizard メトリック フィールドの中には、Azure Log Analytics と互換性がないものがあります。</span><span class="sxs-lookup"><span data-stu-id="73a62-202">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="73a62-203">このため、この参照アーキテクチャには、カスタム Dropwizard シンクおよびレポーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="73a62-203">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="73a62-204">これにより、メトリックは、Azure Log Analytics で予期される形式に書式設定されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-204">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="73a62-205">Apache Spark によってメトリックがレポートされると、形式に誤りがある乗車データと料金データに対するカスタム メトリックも送信されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-205">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="73a62-206">Azure Log Analytics ワークスペースに記録される最後のメトリックは、Spark Structured Streaming ジョブの累積的な進行状況です。</span><span class="sxs-lookup"><span data-stu-id="73a62-206">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="73a62-207">これは、**com.microsoft.pnp.StreamingMetricsListener** クラスで実装されるカスタム StreamingQuery リスナーを使用して、実行されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-207">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="73a62-208">このクラスは、ジョブの実行時に、Apache Spark セッションに登録されます。</span><span class="sxs-lookup"><span data-stu-id="73a62-208">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="73a62-209">StreamingMetricsListener 内のメソッドは、構造化ストリーミング イベントが発生すると必ず、Apache Spark ランタイムによって呼び出され、ログ メッセージとメトリックを Azure Log Analytics ワークスペースに送信します。</span><span class="sxs-lookup"><span data-stu-id="73a62-209">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="73a62-210">アプリケーションを監視するには、ワークスペースで次のクエリを使用とできます。</span><span class="sxs-lookup"><span data-stu-id="73a62-210">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="73a62-211">ストリーミング クエリの待機時間とスループット</span><span class="sxs-lookup"><span data-stu-id="73a62-211">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="73a62-212">ストリーム クエリの実行中に記録された例外</span><span class="sxs-lookup"><span data-stu-id="73a62-212">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="73a62-213">形式に誤りがある料金データと乗車データの蓄積</span><span class="sxs-lookup"><span data-stu-id="73a62-213">Accumulation of malformed fare and ride data</span></span>

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

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="73a62-214">回復性をトレースするジョブ実行</span><span class="sxs-lookup"><span data-stu-id="73a62-214">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a><span data-ttu-id="73a62-215">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="73a62-215">Deploy the solution</span></span>

<span data-ttu-id="73a62-216">リファレンス実装をデプロイおよび実行するには、[GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics) の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="73a62-216">To the deploy and run the reference implementation, follow the steps in the [GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>
