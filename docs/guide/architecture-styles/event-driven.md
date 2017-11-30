---
title: "イベント ドリブン アーキテクチャのスタイル"
description: "Azure でのイベント ドリブン アーキテクチャと IoT アーキテクチャのメリット、課題、ベスト プラクティスを説明します"
author: MikeWasson
ms.openlocfilehash: ff7f936ceabefe7079a1ebbfa717ff4095bf133b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="event-driven-architecture-style"></a><span data-ttu-id="685e9-103">イベント ドリブン アーキテクチャのスタイル</span><span class="sxs-lookup"><span data-stu-id="685e9-103">Event-driven architecture style</span></span>

<span data-ttu-id="685e9-104">イベント ドリブン アーキテクチャは、イベントのストリームを生成する**イベント プロデューサー**と、イベントをリッスンする**イベント コンシューマー**で構成されます。</span><span class="sxs-lookup"><span data-stu-id="685e9-104">An event-driven architecture consists of **event producers** that generate a stream of events, and **event consumers** that listen for the events.</span></span> 

![](./images/event-driven.svg)

<span data-ttu-id="685e9-105">イベントはほぼリアルタイムで配信されるため、イベントが発生するとコンシューマーはすぐに応答できます。</span><span class="sxs-lookup"><span data-stu-id="685e9-105">Events are delivered in near real time, so consumers can respond immediately to events as they occur.</span></span> <span data-ttu-id="685e9-106">プロデューサーはコンシューマーから分離されており、&mdash;プロデューサーにはどのコンシューマーがリッスンしているのかがわかりません。</span><span class="sxs-lookup"><span data-stu-id="685e9-106">Producers are decoupled from consumers &mdash; a producer doesn't know which consumers are listening.</span></span> <span data-ttu-id="685e9-107">コンシューマーも互いに分離されており、すべてのコンシューマーがすべてのイベントを見ることができます。</span><span class="sxs-lookup"><span data-stu-id="685e9-107">Consumers are also decoupled from each other, and every consumer sees all of the events.</span></span> <span data-ttu-id="685e9-108">これは[競合コンシューマー][competing-consumers] パターンとは異なり、コンシューマーがキューからメッセージを取得し、メッセージは一度だけ処理されます (エラーは想定されません)。</span><span class="sxs-lookup"><span data-stu-id="685e9-108">This differs from a [Competing Consumers][competing-consumers] pattern, where consumers pull messages from a queue and a message is processed just once (assuming no errors).</span></span> <span data-ttu-id="685e9-109">IoT などの一部のシステムでは、イベントを大量に取り込む必要があります。</span><span class="sxs-lookup"><span data-stu-id="685e9-109">In some systems, such as IoT, events must be ingested at very high volumes.</span></span>

<span data-ttu-id="685e9-110">イベント ドリブン アーキテクチャでは、パブリッシュ/サブスクライブ モデルやイベント ストリーム モデルを使用できます。</span><span class="sxs-lookup"><span data-stu-id="685e9-110">An event driven architecture can use a pub/sub model or an event stream model.</span></span> 

- <span data-ttu-id="685e9-111">**パブリッシュ/サブスクライブ**: メッセージング インフラストラクチャはサブスクリプションを追跡し続けます。</span><span class="sxs-lookup"><span data-stu-id="685e9-111">**Pub/sub**: The messaging infrastructure keeps track of subscriptions.</span></span> <span data-ttu-id="685e9-112">イベントが発行されると、各サブスクライバーにイベントが送信されます。</span><span class="sxs-lookup"><span data-stu-id="685e9-112">When an event is published, it sends the event to each subscriber.</span></span> <span data-ttu-id="685e9-113">イベントの受信後は、イベントを再生することはできません。また、そのイベントは新しいサブスクライバーに表示されません。</span><span class="sxs-lookup"><span data-stu-id="685e9-113">After an event is received, it cannot be replayed, and new subscribers do not see the event.</span></span> 

- <span data-ttu-id="685e9-114">**イベント ストリーミング**: イベントがログに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="685e9-114">**Event streaming**: Events are written to a log.</span></span> <span data-ttu-id="685e9-115">イベントは (パーティション内で) 厳密に順序付けされ、持続します。</span><span class="sxs-lookup"><span data-stu-id="685e9-115">Events are strictly ordered (within a partition) and durable.</span></span> <span data-ttu-id="685e9-116">クライアントはストリームにサブスクライブしませんが、その代わり、ストリームのどの部分からでも読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="685e9-116">Clients don't subscribe to the stream, instead a client can read from any part of the stream.</span></span> <span data-ttu-id="685e9-117">ストリーム内でクライアントの位置を進めるのは、クライアントの役割です。</span><span class="sxs-lookup"><span data-stu-id="685e9-117">The client is responsible for advancing its position in the stream.</span></span> <span data-ttu-id="685e9-118">つまりクライアントはいつでも参加でき、イベントを再生できます。</span><span class="sxs-lookup"><span data-stu-id="685e9-118">That means a client can join at any time, and can replay events.</span></span>

<span data-ttu-id="685e9-119">コンシューマー側には、いくつかの一般的なバリエーションがあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-119">On the consumer side, there are some common variations:</span></span>

- <span data-ttu-id="685e9-120">**簡単なイベント処理**。</span><span class="sxs-lookup"><span data-stu-id="685e9-120">**Simple event processing**.</span></span> <span data-ttu-id="685e9-121">イベントがコンシューマーのアクションを即時トリガーします。</span><span class="sxs-lookup"><span data-stu-id="685e9-121">An event immediately triggers an action in the consumer.</span></span> <span data-ttu-id="685e9-122">たとえば、お客様は Service Bus トリガーを備えた Azure Functions を使用でき、それにより、Service Bus トピックにメッセージが発行されるたびに関数が実行されます。</span><span class="sxs-lookup"><span data-stu-id="685e9-122">For example, you could use Azure Functions with a Service Bus trigger, so that a function executes whenever a message is published to a Service Bus topic.</span></span>

- <span data-ttu-id="685e9-123">**複合イベント処理**。</span><span class="sxs-lookup"><span data-stu-id="685e9-123">**Complex event processing**.</span></span> <span data-ttu-id="685e9-124">コンシューマーは一連のイベントを処理し、Azure Stream Analytics または Apache Storm などのテクノロジを使用してイベント データのパターンを検索します。</span><span class="sxs-lookup"><span data-stu-id="685e9-124">A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm.</span></span> <span data-ttu-id="685e9-125">たとえば、組み込みデバイスからの測定値を時間枠で集計し、移動平均が特定のしきい値を超えた場合に通知を生成することができます。</span><span class="sxs-lookup"><span data-stu-id="685e9-125">For example, you could aggregate readings from an embedded device over a time window, and generate a notification if the moving average crosses a certain threshold.</span></span> 

- <span data-ttu-id="685e9-126">**イベント ストリーム処理**.</span><span class="sxs-lookup"><span data-stu-id="685e9-126">**Event stream processing**.</span></span> <span data-ttu-id="685e9-127">Azure IoT Hub や Apache Kafka などのデータ ストリーミング プラットフォームを、イベントを取り込んでストリーム プロセッサにフィードするパイプラインとして使用します。</span><span class="sxs-lookup"><span data-stu-id="685e9-127">Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors.</span></span> <span data-ttu-id="685e9-128">ストリーム プロセッサは、ストリームの処理や変換を行います。</span><span class="sxs-lookup"><span data-stu-id="685e9-128">The stream processors act to process or transform the stream.</span></span> <span data-ttu-id="685e9-129">アプリケーションの異なるサブシステムに対して、複数のストリーム プロセッサが存在する場合があります。</span><span class="sxs-lookup"><span data-stu-id="685e9-129">There may be multiple stream processors for different subsystems of the application.</span></span> <span data-ttu-id="685e9-130">この手法は IoT ワークロードに適しています。</span><span class="sxs-lookup"><span data-stu-id="685e9-130">This approach is a good fit for IoT workloads.</span></span>

<span data-ttu-id="685e9-131">イベントのソースはシステムの外部にある場合があります。たとえば、IoT ソリューションの物理デバイスなどです。</span><span class="sxs-lookup"><span data-stu-id="685e9-131">The source of the events may be external to the system, such as physical devices in an IoT solution.</span></span> <span data-ttu-id="685e9-132">その場合、システムはデータ ソースで必要なボリュームとスループットでデータを取り込むことができる必要があります。</span><span class="sxs-lookup"><span data-stu-id="685e9-132">In that case, the system must be able to ingest the data at the volume and throughput that is required by the data source.</span></span>

<span data-ttu-id="685e9-133">上記の図では、コンシューマーが種類ごとに 1 つのボックスで表示されています。</span><span class="sxs-lookup"><span data-stu-id="685e9-133">In the logical diagram above, each type of consumer is shown as a single box.</span></span> <span data-ttu-id="685e9-134">実際には、1 つのコンシューマーに複数のインスタンスがあるのが一般的ですが、それはコンシューマーがシステムの単一障害点にならないようにするためです。</span><span class="sxs-lookup"><span data-stu-id="685e9-134">In practice, it's common to have multiple instances of a consumer, to avoid having the consumer become a single point of failure in system.</span></span> <span data-ttu-id="685e9-135">イベントのボリュームと頻度を制御するのに、複数のインスタンスが必要になることもあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-135">Multiple instances might also be necessary to handle the volume and frequency of events.</span></span> <span data-ttu-id="685e9-136">また、1 つのコンシューマーが複数のスレッドのイベントを処理することもあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-136">Also, a single consumer might process events on multiple threads.</span></span> <span data-ttu-id="685e9-137">イベントを順番に処理しなければならない、または正確に 1 回のセマンティクスが必要な場合には、このために課題が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-137">This can create challenges if events must be processed in order, or require exactly-once semantics.</span></span> <span data-ttu-id="685e9-138">[「Minimize Coordination (調整を最小限に抑える)」][minimize-coordination]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="685e9-138">See [Minimize Coordination][minimize-coordination].</span></span> 

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="685e9-139">このアーキテクチャを使用する状況</span><span class="sxs-lookup"><span data-stu-id="685e9-139">When to use this architecture</span></span>

- <span data-ttu-id="685e9-140">複数のサブシステムで同じイベントを処理する必要がある。</span><span class="sxs-lookup"><span data-stu-id="685e9-140">Multiple subsystems must process the same events.</span></span> 
- <span data-ttu-id="685e9-141">最小のタイム ラグのリアルタイム処理。</span><span class="sxs-lookup"><span data-stu-id="685e9-141">Real-time processing with minimum time lag.</span></span>
- <span data-ttu-id="685e9-142">パターン マッチングや時間枠での集計などの複合イベント処理。</span><span class="sxs-lookup"><span data-stu-id="685e9-142">Complex event processing, such as pattern matching or aggregation over time windows.</span></span>
- <span data-ttu-id="685e9-143">IoT などの高ボリューム、高ベロシティのデータ。</span><span class="sxs-lookup"><span data-stu-id="685e9-143">High volume and high velocity of data, such as IoT.</span></span>

## <a name="benefits"></a><span data-ttu-id="685e9-144">メリット</span><span class="sxs-lookup"><span data-stu-id="685e9-144">Benefits</span></span>

- <span data-ttu-id="685e9-145">プロデューサーとコンシューマーが分離。</span><span class="sxs-lookup"><span data-stu-id="685e9-145">Producers and consumers are decoupled.</span></span>
- <span data-ttu-id="685e9-146">ポイント ツー ポイント統合でない。</span><span class="sxs-lookup"><span data-stu-id="685e9-146">No point-to point-integrations.</span></span> <span data-ttu-id="685e9-147">システムに新しいコンシューマーを追加するのが簡単。</span><span class="sxs-lookup"><span data-stu-id="685e9-147">It's easy to add new consumers to the system.</span></span>
- <span data-ttu-id="685e9-148">コンシューマーは、イベントが到着するとすぐに応答可能。</span><span class="sxs-lookup"><span data-stu-id="685e9-148">Consumers can respond to events immediately as they arrive.</span></span> 
- <span data-ttu-id="685e9-149">高い拡張性と分散。</span><span class="sxs-lookup"><span data-stu-id="685e9-149">Highly scalable and distributed.</span></span> 
- <span data-ttu-id="685e9-150">サブシステムにイベント ストリームの独立したビューがある。</span><span class="sxs-lookup"><span data-stu-id="685e9-150">Subsystems have independent views of the event stream.</span></span>

## <a name="challenges"></a><span data-ttu-id="685e9-151">課題</span><span class="sxs-lookup"><span data-stu-id="685e9-151">Challenges</span></span>

- <span data-ttu-id="685e9-152">保証された配信。</span><span class="sxs-lookup"><span data-stu-id="685e9-152">Guaranteed delivery.</span></span> <span data-ttu-id="685e9-153">一部のシステムでは (特に IoT シナリオで)、イベントが配信されたことを保証することが重要です。</span><span class="sxs-lookup"><span data-stu-id="685e9-153">In some systems, especially in IoT scenarios, it's crucial to guarantee that events are delivered.</span></span>
- <span data-ttu-id="685e9-154">イベントを順番に、または正確に 1 回処理。</span><span class="sxs-lookup"><span data-stu-id="685e9-154">Processing events in order or exactly once.</span></span> <span data-ttu-id="685e9-155">各コンシューマー タイプは通常、回復性とスケーラビリティのために複数のインスタンスで実行されます。</span><span class="sxs-lookup"><span data-stu-id="685e9-155">Each consumer type typically runs in multiple instances, for resiliency and scalability.</span></span> <span data-ttu-id="685e9-156">(コンシューマー タイプ内で) 順番にイベントを処理する必要がある場合や、処理ロジックがべき等でない場合は、このために課題が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-156">This can create a challenge if the events must be processed in order (within a consumer type), or if the processing logic is not idempotent.</span></span>

## <a name="iot-architecture"></a><span data-ttu-id="685e9-157">IoT アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="685e9-157">IoT architecture</span></span>

<span data-ttu-id="685e9-158">イベント ドリブン アーキテクチャは、IoT ソリューションにとって重要です。</span><span class="sxs-lookup"><span data-stu-id="685e9-158">Event-driven architectures are central to IoT solutions.</span></span> <span data-ttu-id="685e9-159">次の図は、IoT で考えられる論理アーキテクチャを示しています。</span><span class="sxs-lookup"><span data-stu-id="685e9-159">The following diagram shows a possible logical architecture for IoT.</span></span> <span data-ttu-id="685e9-160">この図では、アーキテクチャのイベント ストリーミング コンポーネントが強調されています。</span><span class="sxs-lookup"><span data-stu-id="685e9-160">The diagram emphasizes the event-streaming components of the architecture.</span></span>

![](./images/iot.png)

<span data-ttu-id="685e9-161">**クラウド ゲートウェイ**が、信頼性の高い低待機時間のメッセージング システムを使用して、クラウドの境界でデバイスのイベントを取り込みます。</span><span class="sxs-lookup"><span data-stu-id="685e9-161">The **cloud gateway** ingests device events at the cloud boundary, using a reliable, low latency messaging system.</span></span>

<span data-ttu-id="685e9-162">デバイスは、クラウド ゲートウェイに直接イベントを送信するか、**フィールド ゲートウェイ**を介して送信します。</span><span class="sxs-lookup"><span data-stu-id="685e9-162">Devices might send events directly to the cloud gateway, or through a **field gateway**.</span></span> <span data-ttu-id="685e9-163">フィールド ゲートウェイは特殊なデバイスまたはソフトウェアで、通常はデバイスと共に配置され、イベントを受信してクラウド ゲートウェイに転送します。</span><span class="sxs-lookup"><span data-stu-id="685e9-163">A field gateway is a specialized device or software, usually colocated with the devices, that receives events and forwards them to the cloud gateway.</span></span> <span data-ttu-id="685e9-164">フィールド ゲートウェイは、フィルター処理、集計、またはプロトコルの変換などの関数を実行する、ロウ デバイス イベントの前処理を行うこともあります。</span><span class="sxs-lookup"><span data-stu-id="685e9-164">The field gateway might also preprocess the raw device events, performing functions such as filtering, aggregation, or protocol transformation.</span></span>

<span data-ttu-id="685e9-165">取り込み後、イベントは 1 つ以上の**ストリーム プロセッサ**を経由します。それらのプロセッサは、データをルーティングしたり (ストレージへのルーティングなど)、分析やその他の処理を実行したりできます。</span><span class="sxs-lookup"><span data-stu-id="685e9-165">After ingestion, events go through one or more **stream processors** that can route the data (for example, to storage) or perform analytics and other processing.</span></span>

<span data-ttu-id="685e9-166">一般的な処理の種類を次に示します。</span><span class="sxs-lookup"><span data-stu-id="685e9-166">The following are some common types of processing.</span></span> <span data-ttu-id="685e9-167">(このリストは全てを網羅しているわけではありません。)</span><span class="sxs-lookup"><span data-stu-id="685e9-167">(This list is certainly not exhaustive.)</span></span>

- <span data-ttu-id="685e9-168">アーカイブまたはバッチ分析のための、コールド ストレージへのイベント データの書き込み。</span><span class="sxs-lookup"><span data-stu-id="685e9-168">Writing event data to cold storage, for archiving or batch analytics.</span></span>

- <span data-ttu-id="685e9-169">イベント ストリームを (ほぼ) リアルタイムで分析するホット パス分析で異常を検出し、ローリング時間枠でパターンを認識し、ストリームで特定の条件が発生した場合にアラートをトリガーする。</span><span class="sxs-lookup"><span data-stu-id="685e9-169">Hot path analytics, analyzing the event stream in (near) real time, to detect anomalies, recognize patterns over rolling time windows, or trigger alerts when a specific condition occurs in the stream.</span></span> 

- <span data-ttu-id="685e9-170">通知やアラームなど、デバイスからの特殊な非テレメトリ メッセージを処理。</span><span class="sxs-lookup"><span data-stu-id="685e9-170">Handling special types of non-telemetry messages from devices, such as notifications and alarms.</span></span> 

- <span data-ttu-id="685e9-171">機械学習。</span><span class="sxs-lookup"><span data-stu-id="685e9-171">Machine learning.</span></span>

<span data-ttu-id="685e9-172">網掛けのグレーのボックスに、IoT システムのコンポーネントが表示されています。これらのコンポーネントはイベント ストリーミングに直接関連はありませんが、ここでは完全を期すために盛り込んでいます。</span><span class="sxs-lookup"><span data-stu-id="685e9-172">The boxes that are shaded gray show components of an IoT system that are not directly related to event streaming, but are included here for completeness.</span></span>

- <span data-ttu-id="685e9-173">**デバイス レジストリ**はプロビジョニングされたデバイスのデータベースで、デバイス ID が含まれ、また、たいていは位置情報などのデバイスのメタデータも含まれています。</span><span class="sxs-lookup"><span data-stu-id="685e9-173">The **device registry** is a database of the provisioned devices, including the device IDs and usually device metadata, such as location.</span></span>

- <span data-ttu-id="685e9-174">**プロビジョニング API** は新しいデバイスのプロビジョニングや登録でよく使用される外部インターフェイスです。</span><span class="sxs-lookup"><span data-stu-id="685e9-174">The **provisioning API** is a common external interface for provisioning and registering new devices.</span></span>

- <span data-ttu-id="685e9-175">一部の IoT ソリューションでは、**コマンドやコントロール メッセージ**をデバイスに送信できます。</span><span class="sxs-lookup"><span data-stu-id="685e9-175">Some IoT solutions allow **command and control messages** to be sent to devices.</span></span>

> <span data-ttu-id="685e9-176">このセクションでは IoT の概要について示しましたが、考慮すべき詳細や課題は多数あります。</span><span class="sxs-lookup"><span data-stu-id="685e9-176">This section has presented a very high-level view of IoT, and there are many subtleties and challenges to consider.</span></span> <span data-ttu-id="685e9-177">さらに詳細な参照アーキテクチャやディスカッションについては、「[Microsoft Azure IoT Reference Architecture (Microsoft Azure IoT 参照アーキテクチャ)][iot-ref-arch]」 (PDF ダウンロード) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="685e9-177">For a more detailed reference architecture and discussion, see the [Microsoft Azure IoT Reference Architecture][iot-ref-arch] (PDF download).</span></span>

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[iot-ref-arch]: https://azure.microsoft.com/en-us/updates/microsoft-azure-iot-reference-architecture-available/
[minimize-coordination]: ../design-principles/minimize-coordination.md


