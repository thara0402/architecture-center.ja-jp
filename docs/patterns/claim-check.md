---
title: 要求チェック パターン
titleSuffix: Cloud Design Patterns
description: 大きいメッセージを要求チェックとペイロードに分割して、メッセージ バスに過度な負荷がかかることを防ぎます。
keywords: 設計パターン
author: yorek
ms.date: 03/05/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 88457983df4ea3fd4ed8d0c447de6302422339f8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248867"
---
# <a name="claim-check-pattern"></a><span data-ttu-id="d9bfc-104">要求チェック パターン</span><span class="sxs-lookup"><span data-stu-id="d9bfc-104">Claim-Check Pattern</span></span>

<span data-ttu-id="d9bfc-105">大きいメッセージを要求チェックとペイロードに分割します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-105">Split a large message into a claim check and a payload.</span></span> <span data-ttu-id="d9bfc-106">要求チェックはメッセージング プラットフォームに送信し、ペイロードは外部サービスに格納します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-106">Send the claim check to the messaging platform and store the payload to an external service.</span></span> <span data-ttu-id="d9bfc-107">このパターンを使用して大きいメッセージを処理すると、メッセージ バスやクライアントで大きな負荷や速度の低下が発生するのを防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-107">This pattern allows large messages to be processed, while protecting the message bus and the client from being overwhelmed or slowed down.</span></span> <span data-ttu-id="d9bfc-108">また、通常はストレージの方がメッセージング プラットフォームで使用されるリソース ユニットより安いので、このパターンはコストの削減にも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-108">This pattern also helps to reduce costs, as storage is usually cheaper than resource units used by the messaging platform.</span></span>

<span data-ttu-id="d9bfc-109">このパターンは参照ベースのメッセージングとも呼ばれ、もともとは Gregor Hohpe と Bobby Woolf による本『*Enterprise Integration Patterns*』(エンタープライズ統合パターン) で[説明][enterprise-integration-patterns]されたものです。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-109">This pattern is also known as Reference-Based Messaging, and was originally [described][enterprise-integration-patterns] in the book *Enterprise Integration Patterns*, by Gregor Hohpe and Bobby Woolf.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="d9bfc-110">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="d9bfc-110">Context and problem</span></span>

<span data-ttu-id="d9bfc-111">メッセージング ベースのアーキテクチャは、いつかは大きいメッセージを送信、受信、操作できるようになる必要があります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-111">A messaging-based architecture at some point must be able to send, receive, and manipulate large messages.</span></span> <span data-ttu-id="d9bfc-112">このようなメッセージには、画像 (MRI スキャンなど)、サウンド ファイル (コール センターの呼び出しなど)、テキスト ドキュメント、任意のサイズのあらゆる種類のバイナリ データなど、何でも含まれる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-112">Such messages may contain anything, including images (for example, MRI scans), sound files (for example, call-center calls), text documents, or any kind of binary data of arbitrary size.</span></span>

<span data-ttu-id="d9bfc-113">このような大きいメッセージをメッセージ バスに直接送信することは、より多くのリソースと帯域幅を消費する必要があるため、推奨されません。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-113">Sending such large messages to the message bus directly is not recommended, because they require more resources and bandwidth to be consumed.</span></span> <span data-ttu-id="d9bfc-114">また、通常、メッセージング プラットフォームは多数の小さいメッセージを処理するように微調整されているため、大きいメッセージではソリューション全体が遅くなることがあります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-114">Large messages can also slow down the entire solution, because messaging platforms are usually fine-tuned to handle huge quantities of small messages.</span></span> <span data-ttu-id="d9bfc-115">また、ほとんどのメッセージング プラットフォームにはメッセージ サイズに制限があるので、大きいメッセージではこれらの制限を回避する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-115">Also, most messaging platforms have limits on message size, so you may need to work around these limits for large messages.</span></span>

## <a name="solution"></a><span data-ttu-id="d9bfc-116">解決策</span><span class="sxs-lookup"><span data-stu-id="d9bfc-116">Solution</span></span>

<span data-ttu-id="d9bfc-117">メッセージのペイロード全体を、データベースなどの外部サービスに格納します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-117">Store the entire message payload into an external service, such as a database.</span></span> <span data-ttu-id="d9bfc-118">格納されているペイロードへの参照を取得し、メッセージ バスにはその参照だけを送信します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-118">Get the reference to the stored payload, and send just that reference to the message bus.</span></span> <span data-ttu-id="d9bfc-119">参照は、荷物を受け取るための預かり証のように、パターンの名前として機能します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-119">The reference acts like a claim check used to retrieve a piece of luggage, hence the name of the pattern.</span></span> <span data-ttu-id="d9bfc-120">その特定のメッセージの処理に関心のあるクライアントは、必要に応じて、入手した参照を使用してペイロードを取得できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-120">Clients interested in processing that specific message can use the obtained reference to retrieve the payload, if needed.</span></span>

![要求チェック パターンの図](./_images/claim-check.jpg)

## <a name="issues-and-considerations"></a><span data-ttu-id="d9bfc-122">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="d9bfc-122">Issues and considerations</span></span>

<span data-ttu-id="d9bfc-123">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-123">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="d9bfc-124">メッセージをアーカイブする必要がない場合は、使用後にメッセージ データを削除することを検討します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-124">Consider deleting the message data after consuming it, if you don't need to archive the messages.</span></span> <span data-ttu-id="d9bfc-125">BLOB ストレージは比較的安価ですが、期間が長くなれば、特に大量のデータがある場合、ある程度のコストがかかります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-125">Although blob storage is relatively cheap, it costs some money in the long run, especially if there is a lot of data.</span></span> <span data-ttu-id="d9bfc-126">メッセージの削除は、メッセージを受信して処理するアプリケーションによって同期的に、または独立した専用プロセスによって非同期的に、実行できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-126">Deleting the message can be done synchronously by the application that receives and processes the message, or asynchronously by a separate dedicated process.</span></span> <span data-ttu-id="d9bfc-127">非同期的な方法では、受信側アプリケーションのスループットとメッセージ処理パフォーマンスに影響を与えずに、古いデータが削除されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-127">The asynchronous approach removes old data with no impact on the throughput and message processing performance of the receiving application.</span></span>

- <span data-ttu-id="d9bfc-128">メッセージの格納と取得では、若干のオーバーヘッドと待機時間が余分に発生します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-128">Storing and retrieving the message causes some additional overhead and latency.</span></span> <span data-ttu-id="d9bfc-129">メッセージのサイズがメッセージ バスのデータ制限を超えた場合にのみこのパターンを使用するロジックを、送信側アプリケーションに実装することができます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-129">You may want to implement logic in the sending application to use this pattern only when the message size exceeds the data limit of the message bus.</span></span> <span data-ttu-id="d9bfc-130">小さいメッセージの場合はパターンはスキップされます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-130">The pattern would be skipped for smaller messages.</span></span> <span data-ttu-id="d9bfc-131">このアプローチでは、条件付きの要求チェック パターンになります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-131">This approach would result in a conditional claim-check pattern.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="d9bfc-132">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="d9bfc-132">When to use this pattern</span></span>

<span data-ttu-id="d9bfc-133">選択されているメッセージ バス テクノロジでサポートされているメッセージ制限にメッセージが適合できない場合は常に、このパターンを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-133">This pattern should be used whenever a message cannot fit the supported message limit of the chosen message bus technology.</span></span> <span data-ttu-id="d9bfc-134">たとえば、現在、Event Hubs には 256 KB (Basic レベル) の制限がありますが、Event Grid では 64 KB のメッセージしかサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-134">For example, Event Hubs currently has a limit of 256 KB (Basic Tier), while Event Grid supports only 64-KB messages.</span></span>

<span data-ttu-id="d9bfc-135">読み取りを許可されたサービスだけがペイロードにアクセスできるようにする必要がある場合も、パターンを使用できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-135">The pattern can also be used if the payload should be accessed only by services that are authorized to see it.</span></span> <span data-ttu-id="d9bfc-136">ペイロードを外部リソースにオフロードすることで、より厳密な認証と承認の規則を実施し、ペイロードに機密データが格納されている場合にセキュリティを適用することができます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-136">By offloading the payload to an external resource, stricter authentication and authorization rules can be put in place, to ensure that security is enforced when sensitive data is stored in the payload.</span></span>

## <a name="examples"></a><span data-ttu-id="d9bfc-137">例</span><span class="sxs-lookup"><span data-stu-id="d9bfc-137">Examples</span></span>

<span data-ttu-id="d9bfc-138">Azure では、さまざまな方法とテクノロジを使用してこのパターンを実装することができますが、2 つの主なカテゴリがあります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-138">On Azure, this pattern can be implemented in several ways and with different technologies, but there are two main categories.</span></span> <span data-ttu-id="d9bfc-139">どちらの場合も、受信側が、要求チェックを読み取り、それを使用してペイロードを取得する責任を持ちます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-139">In both cases, the receiver has the responsibility to read the claim check and use it to retrieve the payload.</span></span>

- <span data-ttu-id="d9bfc-140">**要求チェックの自動生成**。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-140">**Automatic claim-check generation**.</span></span> <span data-ttu-id="d9bfc-141">この方法では、[Azure Event Grid](/azure/event-grid/) を使用して、自動的に要求チェックを生成し、メッセージ バスにプッシュします。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-141">This approach uses [Azure Event Grid](/azure/event-grid/) to automatically generate the claim check and push it into the message bus.</span></span>

- <span data-ttu-id="d9bfc-142">**要求チェックの手動生成**。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-142">**Manual claim-check generation**.</span></span> <span data-ttu-id="d9bfc-143">この方法では、ペイロードを管理する責任は送信側にあります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-143">In this approach, the sender is responsible for managing the payload.</span></span> <span data-ttu-id="d9bfc-144">送信側は、適切なサービスを使用してペイロードを格納し、要求チェックを取得または生成して、メッセージ バスに要求チェックを送信します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-144">The sender stores the payload using the appropriate service, gets or generates the claim check, and sends the claim check to the message bus.</span></span>

<span data-ttu-id="d9bfc-145">Event Grid はイベント ルーティング サービスであり、最大 24 時間の構成可能な時間内にイベントの配信が試みられます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-145">Event Grid is an event routing service and tries to deliver events within a configurable amount of time up to 24 hours.</span></span> <span data-ttu-id="d9bfc-146">その後、イベントは破棄されるか配信不能にされます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-146">After that, events are either discarded or dead lettered.</span></span> <span data-ttu-id="d9bfc-147">イベント ペイロードをアーカイブしたり、イベント ストリームを再生したりする必要がある場合は、Event Grid サブスクリプションを Event Hubs または Queue storage に追加できます。そこでは、メッセージを長期間保持でき、メッセージのアーカイブがサポートされています。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-147">If you need to archive the event payloads or replay the event stream, you can add an Event Grid subscription to Event Hubs or Queue Storage, where messages can be retained for longer periods and archiving messages is supported.</span></span> <span data-ttu-id="d9bfc-148">Event Grid メッセージの配信と再試行および配信不能の構成の微調整については、「[配信不能と再試行に関する方針](/azure/event-grid/manage-event-delivery)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-148">For information about fine tuning Event Grid message delivery and retry, and dead letter configuration, see  [Dead letter and retry policies](/azure/event-grid/manage-event-delivery).</span></span>

### <a name="automatic-claim-check-generation-with-blob-storage-and-event-grid"></a><span data-ttu-id="d9bfc-149">Blob Storage と Event Grid による要求チェックの自動生成</span><span class="sxs-lookup"><span data-stu-id="d9bfc-149">Automatic claim-check generation with Blob Storage and Event Grid</span></span>

<span data-ttu-id="d9bfc-150">この方法では、送信側が指定された Azure Blob Storage コンテナーにメッセージ ペイロードをドロップします。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-150">In this approach, the sender drops the message payload into a designated Azure Blob Storage container.</span></span> <span data-ttu-id="d9bfc-151">Event Grid により、自動的にタグ/参照が生成されて、Azure Storage キューなどのサポートされているメッセージ バスに送信されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-151">Event Grid automatically generates a tag/reference and sends it to a supported message bus, such as Azure Storage Queues.</span></span> <span data-ttu-id="d9bfc-152">受信側は、キューをポーリングし、メッセージを取得した後、格納されている参照データを使用して Blob Storage から直接ペイロードをダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-152">The receiver can poll the queue, get the message, and then use the stored reference data to download the payload directly from Blob Storage.</span></span>

<span data-ttu-id="d9bfc-153">同じ Event Grid メッセージを、[Azure Functions](/azure/azure-functions/) で直接使用できます。メッセージ バスを経由する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-153">The same Event Grid message can be directly consumed by [Azure Functions](/azure/azure-functions/), without needing to go through a message bus.</span></span> <span data-ttu-id="d9bfc-154">この方法では、Event Grid と Functions 両方のサーバーレスの性質を最大限に利用します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-154">This approach takes full advantage of the serverless nature of both Event Grid and Functions.</span></span>

<span data-ttu-id="d9bfc-155">この方法のコードの例は、[こちら][example-1]で見つかります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-155">You can find example code for this approach [here][example-1].</span></span>

### <a name="event-grid-with-event-hubs"></a><span data-ttu-id="d9bfc-156">Event Grid と Event Hubs</span><span class="sxs-lookup"><span data-stu-id="d9bfc-156">Event Grid with Event Hubs</span></span>

<span data-ttu-id="d9bfc-157">前の例と同様に、ペイロードが Azure BLOB コンテナーに書き込まれると、Event Grid によって自動的にメッセージが生成されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-157">Similar to the previous example, Event Grid automatically generates a message when a payload is written to an Azure Blob container.</span></span> <span data-ttu-id="d9bfc-158">ただし、この例では、メッセージ バスは Event Hubs を使って実装されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-158">But in this example,  the message bus is implemented using Event Hubs.</span></span> <span data-ttu-id="d9bfc-159">クライアントは、それ自体を登録して、イベント ハブに書き込まれたメッセージのストリームを受信できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-159">A client can register itself to receive the stream of messages as they are written to the event hub.</span></span> <span data-ttu-id="d9bfc-160">イベント ハブは、メッセージをアーカイブし、Apache Spark、Apache Drill、または利用可能な任意の Avro ライブラリなどのツールを使用してクエリ可能な Avro ファイルとして使用できるようにするように、構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-160">The event hub can also be configured to archive messages, making them available as an Avro file that can be queried using tools like Apache Spark, Apache Drill, or any of the available Avro libraries.</span></span>

<span data-ttu-id="d9bfc-161">この方法のコードの例は、[こちら][example-2]で見つかります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-161">You can find example code for this approach [here][example-2].</span></span>

### <a name="claim-check-generation-with-service-bus"></a><span data-ttu-id="d9bfc-162">Service Bus での要求チェックの生成</span><span class="sxs-lookup"><span data-stu-id="d9bfc-162">Claim check generation with Service Bus</span></span>

<span data-ttu-id="d9bfc-163">このソリューションでは特定の Service Bus プラグイン [ServiceBus.AttachmentPlugin](https://www.nuget.org/packages/ServiceBus.AttachmentPlugin/) を利用することにより、要求チェック ワークフローの実装が容易になります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-163">This solution takes advantage of a specific Service Bus plugin, [ServiceBus.AttachmentPlugin](https://www.nuget.org/packages/ServiceBus.AttachmentPlugin/), which makes the claim-check workflow easy to implement.</span></span> <span data-ttu-id="d9bfc-164">プラグインでは、メッセージの本文が、メッセージ送信時に Azure Blob Storage に格納される添付ファイルに変換されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-164">The plugin converts any message body into an attachment that gets stored in Azure Blob Storage when the message is sent.</span></span>

```csharp
using ServiceBus.AttachmentPlugin;
...

// Getting connection information
var serviceBusConnectionString = Environment.GetEnvironmentVariable("SERVICE_BUS_CONNECTION_STRING");
var queueName = Environment.GetEnvironmentVariable("QUEUE_NAME");
var storageConnectionString = Environment.GetEnvironmentVariable("STORAGE_CONNECTION_STRING");

// Creating config for sending message
var config = new AzureStorageAttachmentConfiguration(storageConnectionString);

// Creating and registering the sender using Service Bus Connection String and Queue Name
var sender = new MessageSender(serviceBusConnectionString, queueName);
sender.RegisterAzureStorageAttachmentPlugin(config);

// Create payload
var payload = new { data = "random data string for testing" };
var serialized = JsonConvert.SerializeObject(payload);
var payloadAsBytes = Encoding.UTF8.GetBytes(serialized);
var message = new Message(payloadAsBytes);

// Send the message
await sender.SendAsync(message);
```

<span data-ttu-id="d9bfc-165">Service Bus のメッセージは、クライアントがサブスクライブできる通知キューとして機能します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-165">The Service Bus message acts as a notification queue, which a client can subscribe to.</span></span> <span data-ttu-id="d9bfc-166">コンシューマーがメッセージを受信すると、プラグインによって Blob Storage から直接メッセージ データを読み取ることができるようになります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-166">When the consumer receives the message, the plugin makes it possible to directly read the message data from Blob Storage.</span></span> <span data-ttu-id="d9bfc-167">その後、さらにメッセージを処理する方法をお客様が選択できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-167">You can then choose how to process the message further.</span></span> <span data-ttu-id="d9bfc-168">この方法の利点は、要求チェックのワークフローが送信側と受信側から抽象化されることです。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-168">An advantage of this approach is that it abstracts the claim-check workflow from the sender and receiver.</span></span>

<span data-ttu-id="d9bfc-169">この方法のコードの例は、[こちら][example-3]で見つかります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-169">You can find example code for this approach [here][example-3].</span></span>

### <a name="manual-claim-check-generation-with-kafka"></a><span data-ttu-id="d9bfc-170">Kafka での要求チェックの手動生成</span><span class="sxs-lookup"><span data-stu-id="d9bfc-170">Manual claim-check generation with Kafka</span></span>

<span data-ttu-id="d9bfc-171">この例では、Kafka クライアントによって、ペイロードが Azure Blob Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-171">In this example, a Kafka client writes the payload to Azure Blob Storage.</span></span> <span data-ttu-id="d9bfc-172">その後、[Kakfa 対応の Event Hubs](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs) を使用して通知メッセージが送信されます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-172">Then it sends a notification message using [Kakfa-enabled Event Hubs](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs).</span></span> <span data-ttu-id="d9bfc-173">コンシューマーはメッセージを受信し、Blob Storage からペイロードにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-173">The consumer receives the message and can access the payload from Blob Storage.</span></span> <span data-ttu-id="d9bfc-174">この例では、異なるメッセージング プロトコルを使用して、Azure に要求チェック パターンを実装する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-174">This example shows how a different messaging protocol can be used to implement the claim-check pattern in Azure.</span></span> <span data-ttu-id="d9bfc-175">たとえば、既存の Kafka クライアントをサポートする必要があるような場合です。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-175">For example, you might need to support existing Kafka clients.</span></span>

<span data-ttu-id="d9bfc-176">この方法のコードの例は、[こちら][example-4]で見つかります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-176">You can find example code for this approach [here][example-4].</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="d9bfc-177">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="d9bfc-177">Related patterns and guidance</span></span>

- <span data-ttu-id="d9bfc-178">上で説明した例は、[GitHub][sample-code] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-178">The examples described above are available on [GitHub][sample-code].</span></span>
- <span data-ttu-id="d9bfc-179">エンタープライズ統合パターンのサイトには、このパターンの[説明][enterprise-integration-patterns]があります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-179">The Enterprise Integration Patterns site has a [description][enterprise-integration-patterns] of this pattern.</span></span>
- <span data-ttu-id="d9bfc-180">別の例については、「[Dealing with large Service Bus messages using claim check pattern (要求チェック パターンを使用する大きな Service Bus メッセージの処理)](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern)」(ブログの投稿) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-180">For another example, see [Dealing with large Service Bus messages using claim check pattern](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern) (blog post).</span></span>
- <span data-ttu-id="d9bfc-181">大きいメッセージを処理するための別のパターンとしては、[分割][ splitter]と[集約][aggregator]があります。</span><span class="sxs-lookup"><span data-stu-id="d9bfc-181">An alternative pattern for handling large messages is [Split][splitter] and [Aggregate][aggregator].</span></span>

<!-- links -->
[aggregator]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Aggregator.html
[enterprise-integration-patterns]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html
[example-1]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-1
[example-2]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-2
[example-3]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-3
[example-4]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-4
[sample-code]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check
[splitter]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html
