---
title: Azure Functions を使用したサーバーレスなイベント処理
titleSuffix: Azure Reference Architectures
description: Azure Functions を使用したサーバーレスなイベントの取り込みと処理の参照アーキテクチャ。
author: MikeWasson
ms.date: 10/16/2018
ms.custom: seodec18
ms.openlocfilehash: 23edc69e52981cfd15717b491875b34ed025958a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111395"
---
# <a name="serverless-event-processing-using-azure-functions"></a><span data-ttu-id="28b01-103">Azure Functions を使用したサーバーレスなイベント処理</span><span class="sxs-lookup"><span data-stu-id="28b01-103">Serverless event processing using Azure Functions</span></span>

<span data-ttu-id="28b01-104">この参照アーキテクチャは、データ ストリームの取り込み、データ処理、およびバックエンド データベースへの結果の書き込みを行う[サーバーレス](https://azure.microsoft.com/solutions/serverless/)なイベント ドリブン アーキテクチャを示しています。</span><span class="sxs-lookup"><span data-stu-id="28b01-104">This reference architecture shows a [serverless](https://azure.microsoft.com/solutions/serverless/), event-driven architecture that ingests a stream of data, processes the data, and writes the results to a back-end database.</span></span> <span data-ttu-id="28b01-105">このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="28b01-105">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![Azure Functions を使用したサーバーレスなイベント処理の参照アーキテクチャ](./_images/serverless-event-processing.png)

## <a name="architecture"></a><span data-ttu-id="28b01-107">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="28b01-107">Architecture</span></span>

<span data-ttu-id="28b01-108">**Event Hubs** は、データ ストリームを取り込みます。</span><span class="sxs-lookup"><span data-stu-id="28b01-108">**Event Hubs** ingests the data stream.</span></span> <span data-ttu-id="28b01-109">[Event Hubs][eh] は、高スループットのデータ ストリーミング シナリオ用に設計されています。</span><span class="sxs-lookup"><span data-stu-id="28b01-109">[Event Hubs][eh] is designed for high-throughput data streaming scenarios.</span></span>

> [!NOTE]
> <span data-ttu-id="28b01-110">IoT シナリオの場合は、IoT Hub をお勧めします。</span><span class="sxs-lookup"><span data-stu-id="28b01-110">For IoT scenarios, we recommend IoT Hub.</span></span> <span data-ttu-id="28b01-111">IoT Hub には、Azure Event Hubs API と互換性がある組み込みのエンドポイントが含まれているため、このアーキテクチャではどちらのサービスも使用できます。その際、バックエンド処理に大きな変更は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="28b01-111">IoT Hub has a built-in endpoint that’s compatible with the Azure Event Hubs API, so you can use either service in this architecture with no major changes in the backend processing.</span></span> <span data-ttu-id="28b01-112">詳細については、「[IoT デバイスを Azure に接続する:IoT Hub と Event Hubs][iot]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-112">For more information, see [Connecting IoT Devices to Azure: IoT Hub and Event Hubs][iot].</span></span>

<span data-ttu-id="28b01-113">**Function App**。</span><span class="sxs-lookup"><span data-stu-id="28b01-113">**Function App**.</span></span> <span data-ttu-id="28b01-114">[Azure Functions][functions] はサーバーレス コンピューティングの 1 つのオプションです。</span><span class="sxs-lookup"><span data-stu-id="28b01-114">[Azure Functions][functions] is a serverless compute option.</span></span> <span data-ttu-id="28b01-115">ひとまとまりのコード ("関数") がトリガーによって呼び出されるイベント ドリブン モデルが使用されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-115">It uses an event-driven model, where a piece of code (a “function”) is invoked by a trigger.</span></span> <span data-ttu-id="28b01-116">このアーキテクチャでは、イベントが Event Hubs に到達したとき、そのイベントを処理し、結果をストレージに書き込む関数がトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="28b01-116">In this architecture, when events arrive at Event Hubs, they trigger a function that processes the events and writes the results to storage.</span></span>

<span data-ttu-id="28b01-117">Function App は、Event Hubs からの個々のレコードを処理するのに適しています。</span><span class="sxs-lookup"><span data-stu-id="28b01-117">Function Apps are suitable for processing individual records from Event Hubs.</span></span> <span data-ttu-id="28b01-118">ストリーム処理シナリオが複雑な場合は、Azure Databricks を使用した Apache Spark、または Azure Stream Analytics を検討してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-118">For more complex stream processing scenarios, consider Apache Spark using Azure Databricks, or Azure Stream Analytics.</span></span>

<span data-ttu-id="28b01-119">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="28b01-119">**Cosmos DB**.</span></span> <span data-ttu-id="28b01-120">[Cosmos DB][cosmosdb] は、マルチモデル データベース サービスです。</span><span class="sxs-lookup"><span data-stu-id="28b01-120">[Cosmos DB][cosmosdb] is a multi-model database service.</span></span> <span data-ttu-id="28b01-121">このシナリオでは、Cosmos DB [SQL API][cosmosdb-sql] を使用して、イベント処理関数に JSON レコードが格納されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-121">For this scenario, the event-processing function stores JSON records, using the Cosmos DB [SQL API][cosmosdb-sql].</span></span>

<span data-ttu-id="28b01-122">**Queue Storage**。</span><span class="sxs-lookup"><span data-stu-id="28b01-122">**Queue storage**.</span></span> <span data-ttu-id="28b01-123">[Queue Storage][queue] は、配信不能メッセージに使用されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-123">[Queue storage][queue] is used for dead letter messages.</span></span> <span data-ttu-id="28b01-124">イベントの処理中にエラーが発生した場合、イベント データは後で処理できるように、関数によって配信不能キューに格納されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-124">If an error occurs while processing an event, the function stores the event data in a dead letter queue for later processing.</span></span> <span data-ttu-id="28b01-125">詳細については、「[回復性に関する考慮事項](#resiliency-considerations)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-125">For more information, see [Resiliency Considerations](#resiliency-considerations).</span></span>

<span data-ttu-id="28b01-126">**Azure Monitor**。</span><span class="sxs-lookup"><span data-stu-id="28b01-126">**Azure Monitor**.</span></span> <span data-ttu-id="28b01-127">[Monitor][monitor] は、ソリューションにデプロイされた Azure サービスに関するパフォーマンス メトリックを収集します。</span><span class="sxs-lookup"><span data-stu-id="28b01-127">[Monitor][monitor] collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="28b01-128">ダッシュボードでこれらを視覚化することで、ソリューションの正常性を把握できます。</span><span class="sxs-lookup"><span data-stu-id="28b01-128">By visualizing these in a dashboard, you can get visibility into the health of the solution.</span></span>

<span data-ttu-id="28b01-129">**Azure Pipelines**。</span><span class="sxs-lookup"><span data-stu-id="28b01-129">**Azure Pipelines**.</span></span> <span data-ttu-id="28b01-130">[Pipelines][pipelines] は、アプリケーションをビルド、テスト、デプロイする継続的インテグレーション (CI) および継続的デリバリー (CD) サービスです。</span><span class="sxs-lookup"><span data-stu-id="28b01-130">[Pipelines][pipelines] is a continuous integration (CI) and continuous delivery (CD) service that builds, tests, and deploys the application.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="28b01-131">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="28b01-131">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="28b01-132">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="28b01-132">Event Hubs</span></span>

<span data-ttu-id="28b01-133">Event Hubs のスループット容量は、[スループット ユニット][eh-throughput]で測定されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-133">The throughput capacity of Event Hubs is measured in [throughput units][eh-throughput].</span></span> <span data-ttu-id="28b01-134">[自動インフレ][eh-autoscale]を有効にすると、イベント ハブを自動スケーリングできます。自動インフレでは、トラフィックに基づいて、スループット ユニットが構成済みの最大値まで自動的にスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="28b01-134">You can autoscale an event hub by enabling [auto-inflate][eh-autoscale], which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

<span data-ttu-id="28b01-135">関数アプリの [Event Hub トリガー][eh-trigger]は、イベント ハブ内のパーティション数に従ってスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="28b01-135">The [Event Hub trigger][eh-trigger] in the function app scales according to the number of partitions in the event hub.</span></span> <span data-ttu-id="28b01-136">各パーティションには、関数インスタンスが 1 つずつ割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="28b01-136">Each partition is assigned one function instance at a time.</span></span> <span data-ttu-id="28b01-137">スループットを最大化するには、イベントは 1 つずつではなくバッチで受信します。</span><span class="sxs-lookup"><span data-stu-id="28b01-137">To maximize throughput, receive the events in a batch, instead of one at a time.</span></span>

### <a name="cosmos-db"></a><span data-ttu-id="28b01-138">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="28b01-138">Cosmos DB</span></span>

<span data-ttu-id="28b01-139">Cosmos DB のスループット容量は、[要求ユニット][ru] (RU) で測定されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-139">Throughput capacity for Cosmos DB is measured in [Request Units][ru] (RU).</span></span> <span data-ttu-id="28b01-140">Cosmos DB コンテナーを 10,000 RU を超えてスケーリングするには、コンテナーの作成時に[パーティション キー][partition-key]を指定し、作成するすべてのドキュメントにパーティション キーを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="28b01-140">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key][partition-key] when you create the container, and include the partition key in every document that you create.</span></span>

<span data-ttu-id="28b01-141">適切なパーティション キーの特徴をいくつか次に示します。</span><span class="sxs-lookup"><span data-stu-id="28b01-141">Here are some characteristics of a good partition key:</span></span>

- <span data-ttu-id="28b01-142">キー値のスペースが大きい。</span><span class="sxs-lookup"><span data-stu-id="28b01-142">The key value space is large.</span></span>
- <span data-ttu-id="28b01-143">キー値ごとに読み取り/書き込みが均等に分散され、ホット キーが発生しない。</span><span class="sxs-lookup"><span data-stu-id="28b01-143">There will be an even distribution of reads/writes per key value, avoiding hot keys.</span></span>
- <span data-ttu-id="28b01-144">任意の単一キー値の最大格納データが、物理パーティションの最大サイズ (10 GB) を超えない。</span><span class="sxs-lookup"><span data-stu-id="28b01-144">The maximum data stored for any single key value will not exceed the maximum physical partition size (10 GB).</span></span>
- <span data-ttu-id="28b01-145">ドキュメントのパーティション キーが変更されない。</span><span class="sxs-lookup"><span data-stu-id="28b01-145">The partition key for a document won't change.</span></span> <span data-ttu-id="28b01-146">既存のドキュメントでパーティション キーを更新することはできません。</span><span class="sxs-lookup"><span data-stu-id="28b01-146">You can't update the partition key on an existing document.</span></span>

<span data-ttu-id="28b01-147">この参照アーキテクチャのシナリオでは、関数に格納されるドキュメントは、データを送信するデバイスごとに 1 つだけです。</span><span class="sxs-lookup"><span data-stu-id="28b01-147">In the scenario for this reference architecture, the function stores exactly one document per device that is sending data.</span></span> <span data-ttu-id="28b01-148">関数は、upsert 操作を使用して、最新のデバイスの状態でドキュメントを継続的に更新します。</span><span class="sxs-lookup"><span data-stu-id="28b01-148">The function continually updates the documents with latest device status, using an upsert operation.</span></span> <span data-ttu-id="28b01-149">このシナリオでは、書き込みがキーの間で均等に分散されるため、パーティション キーにはデバイス ID が適しています。また、キー値ごと 1 つのドキュメントが存在するため、各パーティションのサイズは厳密にバインドされています。</span><span class="sxs-lookup"><span data-stu-id="28b01-149">Device ID is a good partition key for this scenario, because writes will be evenly distributed across the keys, and the size of each partition will be strictly bounded, because there is a single document for each key value.</span></span> <span data-ttu-id="28b01-150">パーティション キーの詳細については、「[Azure Cosmos DB でのパーティション分割とスケーリング][cosmosdb-scale]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-150">For more information about partition keys, see [Partition and scale in Azure Cosmos DB][cosmosdb-scale].</span></span>

## <a name="resiliency-considerations"></a><span data-ttu-id="28b01-151">回復性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="28b01-151">Resiliency considerations</span></span>

<span data-ttu-id="28b01-152">Functions と共に Event Hubs トリガーを使用する場合、お使いの処理ループ内で例外がキャッチされます。</span><span class="sxs-lookup"><span data-stu-id="28b01-152">When using the Event Hubs trigger with Functions, catch exceptions within your processing loop.</span></span> <span data-ttu-id="28b01-153">ハンドルされない例外が発生すると、Functions ランタイムではメッセージの再試行は行われません。</span><span class="sxs-lookup"><span data-stu-id="28b01-153">If an unhandled exception occurs, the Functions runtime does not retry the messages.</span></span> <span data-ttu-id="28b01-154">処理できないメッセージは、配信不能キューに配置されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-154">If a message cannot be processed, put the message into a dead letter queue.</span></span> <span data-ttu-id="28b01-155">そのメッセージは、帯域外のプロセスを使用して調べられ、是正措置が決定されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-155">Use an out-of-band process to examine the messages and determine corrective action.</span></span>

<span data-ttu-id="28b01-156">次のコードは、どのようにインジェスト関数が例外をキャッチし、未処理のメッセージを配信不能キューに書き込むかを示しています。</span><span class="sxs-lookup"><span data-stu-id="28b01-156">The following code shows how the ingestion function catches exceptions and puts unprocessed messages onto a dead letter queue.</span></span>

```csharp
[FunctionName("RawTelemetryFunction")]
[StorageAccount("DeadLetterStorage")]
public static async Task RunAsync(
    [EventHubTrigger("%EventHubName%", Connection = "EventHubConnection", ConsumerGroup ="%EventHubConsumerGroup%")]EventData[] messages,
    [Queue("deadletterqueue")] IAsyncCollector<DeadLetterMessage> deadLetterMessages,
    ILogger logger)
{
    foreach (var message in messages)
    {
        DeviceState deviceState = null;

        try
        {
            deviceState = telemetryProcessor.Deserialize(message.Body.Array, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error deserializing message", message.SystemProperties.PartitionKey, message.SystemProperties.SequenceNumber);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message });
        }

        try
        {
            await stateChangeProcessor.UpdateState(deviceState, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error updating status document", deviceState);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message, DeviceState = deviceState });
        }
    }
}
```

<span data-ttu-id="28b01-157">関数が[キュー ストレージの出力バインド][queue-binding]を使用して、項目をキューに配置していることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-157">Notice that the function uses the [Queue storage output binding][queue-binding] to put items in the queue.</span></span>

<span data-ttu-id="28b01-158">また、上記のコードにより、例外が Application Insights にログ記録されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-158">The code shown above also logs exceptions to Application Insights.</span></span> <span data-ttu-id="28b01-159">パーティション キーとシーケンス番号を使用すると、配信不能メッセージを、ログの例外に関連付けることができます。</span><span class="sxs-lookup"><span data-stu-id="28b01-159">You can use the partition key and sequence number to correlate dead letter messages with the exceptions in the logs.</span></span>

<span data-ttu-id="28b01-160">配信不能キューのメッセージには、エラーのコンテキストを把握するのに十分な情報が記載されているはずです。</span><span class="sxs-lookup"><span data-stu-id="28b01-160">Messages in the dead letter queue should have enough information so that you can understand the context of error.</span></span> <span data-ttu-id="28b01-161">この例の `DeadLetterMessage` クラスには、例外メッセージ、元のイベント データ、および逆シリアル化されたイベント メッセージ (使用可能な場合) が含まれています。</span><span class="sxs-lookup"><span data-stu-id="28b01-161">In this example, the `DeadLetterMessage` class contains the exception message, the original event data, and the deserialized event message (if available).</span></span>

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

<span data-ttu-id="28b01-162">[Azure Monitor][monitor] を使用して、イベント ハブを監視します。</span><span class="sxs-lookup"><span data-stu-id="28b01-162">Use [Azure Monitor][monitor] to monitor the event hub.</span></span> <span data-ttu-id="28b01-163">入力があり、出力がない場合は、メッセージが処理されていないことを意味します。</span><span class="sxs-lookup"><span data-stu-id="28b01-163">If you see there is input but no output, it means that messages are not being processed.</span></span> <span data-ttu-id="28b01-164">その場合は、[Log Analytics][log-analytics] に移動し、例外またはその他のエラーを探します。</span><span class="sxs-lookup"><span data-stu-id="28b01-164">In that case, go into [Log Analytics][log-analytics] and look for exceptions or other errors.</span></span>

## <a name="disaster-recovery-considerations"></a><span data-ttu-id="28b01-165">ディザスター リカバリーの考慮事項</span><span class="sxs-lookup"><span data-stu-id="28b01-165">Disaster recovery considerations</span></span>

<span data-ttu-id="28b01-166">ここに示したデプロイは単一の Azure リージョンに存在します。</span><span class="sxs-lookup"><span data-stu-id="28b01-166">The deployment shown here resides in a single Azure region.</span></span> <span data-ttu-id="28b01-167">ディザスター リカバリーでより回復性に優れたアプローチを実現するには、さまざまなサービスで地理的分散機能を利用します。</span><span class="sxs-lookup"><span data-stu-id="28b01-167">For a more resilient approach to disaster-recovery, take advantage of geo-distribution features in the various services:</span></span>

- <span data-ttu-id="28b01-168">**Event Hubs**。</span><span class="sxs-lookup"><span data-stu-id="28b01-168">**Event Hubs**.</span></span> <span data-ttu-id="28b01-169">プライマリ (アクティブ) 名前空間とセカンダリ (パッシブ) 名前空間の 2 つの Event Hubs 名前空間を作成します。</span><span class="sxs-lookup"><span data-stu-id="28b01-169">Create two Event Hubs namespaces, a primary (active) namespace and a secondary (passive) namespace.</span></span> <span data-ttu-id="28b01-170">メッセージは、セカンダリ名前空間にフェールオーバーしない限り、アクティブな名前空間に自動的にルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="28b01-170">Messages are automatically routed to the active namespace unless you fail over to the secondary namespace.</span></span> <span data-ttu-id="28b01-171">詳細については、「[Azure Event Hubs geo ディザスター リカバリー][eh-dr]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="28b01-171">For more information, see [Azure Event Hubs Geo-disaster recovery][eh-dr].</span></span>

- <span data-ttu-id="28b01-172">**Function App**。</span><span class="sxs-lookup"><span data-stu-id="28b01-172">**Function App**.</span></span> <span data-ttu-id="28b01-173">セカンダリ Event Hubs 名前空間からの読み取りを待っている 2 つ目の関数アプリをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="28b01-173">Deploy a second function app that is waiting to read from the secondary Event Hubs namespace.</span></span> <span data-ttu-id="28b01-174">この関数は、配信不能キュー用にセカンダリ ストレージ アカウントに書き込みます。</span><span class="sxs-lookup"><span data-stu-id="28b01-174">This function writes to a secondary storage account for dead letter queue.</span></span>

- <span data-ttu-id="28b01-175">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="28b01-175">**Cosmos DB**.</span></span> <span data-ttu-id="28b01-176">Cosmos DB では[複数のマスター リージョン][cosmosdb-geo]がサポートされています。これにより、Cosmos DB アカウントに追加した任意のリージョンに書き込むことができます。</span><span class="sxs-lookup"><span data-stu-id="28b01-176">Cosmos DB supports [multiple master regions][cosmosdb-geo], which enables writes to any region that you add to your Cosmos DB account.</span></span> <span data-ttu-id="28b01-177">マルチマスターを有効にしなくても、プライマリ書き込みリージョンにはフェールオーバーできます。</span><span class="sxs-lookup"><span data-stu-id="28b01-177">If you don’t enable multi-master, you can still fail over the primary write region.</span></span> <span data-ttu-id="28b01-178">フェールオーバーは、Cosmos DB クライアント SDK と Azure Functions のバインディングによって自動的に処理されるため、アプリケーション構成設定を更新する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="28b01-178">The Cosmos DB client SDKs and the Azure Function bindings automatically handle the failover, so you don’t need to update any application configuration settings.</span></span>

- <span data-ttu-id="28b01-179">**Azure Storage**。</span><span class="sxs-lookup"><span data-stu-id="28b01-179">**Azure Storage**.</span></span> <span data-ttu-id="28b01-180">配信不能キューに対して [RA-GRS][ra-grs] ストレージを使用します。</span><span class="sxs-lookup"><span data-stu-id="28b01-180">Use [RA-GRS][ra-grs] storage for the dead letter queue.</span></span> <span data-ttu-id="28b01-181">これにより、別のリージョンに読み取り専用レプリカが作成されます。</span><span class="sxs-lookup"><span data-stu-id="28b01-181">This creates a read-only replica in another region.</span></span> <span data-ttu-id="28b01-182">プライマリ リージョンが利用不可になると、現在キューにある項目を読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="28b01-182">If the primary region becomes unavailable, you can read the items currently in the queue.</span></span> <span data-ttu-id="28b01-183">さらに、セカンダリ リージョンに別のストレージ アカウントをプロビジョニングします。これには、フェールオーバー後に関数が書き込むことができます。</span><span class="sxs-lookup"><span data-stu-id="28b01-183">In addition, provision another storage account in the secondary region that the function can write to after a fail-over.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="28b01-184">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="28b01-184">Deploy the solution</span></span>

<span data-ttu-id="28b01-185">この参照アーキテクチャをデプロイするには、[GitHub の Readme][readme] をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="28b01-185">To deploy this reference architecture, view the [GitHub readme][readme].</span></span>

<!-- links -->

[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[cosmosdb-sql]: /azure/cosmos-db/sql-api-introduction
[eh]: /azure/event-hubs/
[eh-autoscale]: /azure/event-hubs/event-hubs-auto-inflate
[eh-dr]: /azure/event-hubs/event-hubs-geo-dr
[eh-throughput]: /azure/event-hubs/event-hubs-features#throughput-units
[eh-trigger]: /azure/azure-functions/functions-bindings-event-hubs
[functions]: /azure/azure-functions/functions-overview
[iot]: /azure/iot-hub/iot-hub-compare-event-hubs
[log-analytics]: /azure/log-analytics/log-analytics-queries
[monitor]: /azure/azure-monitor/overview
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[queue]: /azure/storage/queues/storage-queues-introduction
[queue-binding]: /azure/azure-functions/functions-bindings-storage-queue#output
[ra-grs]: /azure/storage/common/storage-redundancy-grs
[ru]: /azure/cosmos-db/request-units

[github]: https://github.com/mspnp/serverless-reference-implementation
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md
