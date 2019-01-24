---
title: イベント ドリブン アーキテクチャのスタイル
titleSuffix: Azure Application Architecture Guide
description: Azure でのイベント ドリブン アーキテクチャと IoT アーキテクチャのメリット、課題、ベスト プラクティスを説明します。
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: b83a919e4ccd41d20b508b10604365a0b4ca90af
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485476"
---
# <a name="event-driven-architecture-style"></a><span data-ttu-id="d2863-103">イベント ドリブン アーキテクチャのスタイル</span><span class="sxs-lookup"><span data-stu-id="d2863-103">Event-driven architecture style</span></span>

<span data-ttu-id="d2863-104">イベント ドリブン アーキテクチャは、イベントのストリームを生成する**イベント プロデューサー**と、イベントをリッスンする**イベント コンシューマー**で構成されます。</span><span class="sxs-lookup"><span data-stu-id="d2863-104">An event-driven architecture consists of **event producers** that generate a stream of events, and **event consumers** that listen for the events.</span></span>

![イベント ドリブン アーキテクチャのスタイルの図](./images/event-driven.svg)

<span data-ttu-id="d2863-106">イベントはほぼリアルタイムで配信されるため、イベントが発生するとコンシューマーはすぐに応答できます。</span><span class="sxs-lookup"><span data-stu-id="d2863-106">Events are delivered in near real time, so consumers can respond immediately to events as they occur.</span></span> <span data-ttu-id="d2863-107">プロデューサーはコンシューマーから分離されており、&mdash;プロデューサーにはどのコンシューマーがリッスンしているのかがわかりません。</span><span class="sxs-lookup"><span data-stu-id="d2863-107">Producers are decoupled from consumers &mdash; a producer doesn't know which consumers are listening.</span></span> <span data-ttu-id="d2863-108">コンシューマーも互いに分離されており、すべてのコンシューマーがすべてのイベントを見ることができます。</span><span class="sxs-lookup"><span data-stu-id="d2863-108">Consumers are also decoupled from each other, and every consumer sees all of the events.</span></span> <span data-ttu-id="d2863-109">これは[競合コンシューマー][competing-consumers] パターンとは異なり、コンシューマーがキューからメッセージを取得し、メッセージは一度だけ処理されます (エラーは想定されません)。</span><span class="sxs-lookup"><span data-stu-id="d2863-109">This differs from a [Competing Consumers][competing-consumers] pattern, where consumers pull messages from a queue and a message is processed just once (assuming no errors).</span></span> <span data-ttu-id="d2863-110">IoT などの一部のシステムでは、イベントを大量に取り込む必要があります。</span><span class="sxs-lookup"><span data-stu-id="d2863-110">In some systems, such as IoT, events must be ingested at very high volumes.</span></span>

<span data-ttu-id="d2863-111">イベント ドリブン アーキテクチャでは、パブリッシュ/サブスクライブ モデルやイベント ストリーム モデルを使用できます。</span><span class="sxs-lookup"><span data-stu-id="d2863-111">An event driven architecture can use a pub/sub model or an event stream model.</span></span>

- <span data-ttu-id="d2863-112">**パブリッシュ/サブスクライブ**:メッセージング インフラストラクチャではサブスクリプションを追跡します。</span><span class="sxs-lookup"><span data-stu-id="d2863-112">**Pub/sub**: The messaging infrastructure keeps track of subscriptions.</span></span> <span data-ttu-id="d2863-113">イベントが発行されると、各サブスクライバーにイベントが送信されます。</span><span class="sxs-lookup"><span data-stu-id="d2863-113">When an event is published, it sends the event to each subscriber.</span></span> <span data-ttu-id="d2863-114">イベントの受信後は、イベントを再生することはできません。また、そのイベントは新しいサブスクライバーに表示されません。</span><span class="sxs-lookup"><span data-stu-id="d2863-114">After an event is received, it cannot be replayed, and new subscribers do not see the event.</span></span>

- <span data-ttu-id="d2863-115">**イベント ストリーミング**:イベントがログに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="d2863-115">**Event streaming**: Events are written to a log.</span></span> <span data-ttu-id="d2863-116">イベントは (パーティション内で) 厳密に順序付けされ、持続します。</span><span class="sxs-lookup"><span data-stu-id="d2863-116">Events are strictly ordered (within a partition) and durable.</span></span> <span data-ttu-id="d2863-117">クライアントはストリームにサブスクライブしませんが、その代わり、ストリームのどの部分からでも読み取ることができます。</span><span class="sxs-lookup"><span data-stu-id="d2863-117">Clients don't subscribe to the stream, instead a client can read from any part of the stream.</span></span> <span data-ttu-id="d2863-118">ストリーム内でクライアントの位置を進めるのは、クライアントの役割です。</span><span class="sxs-lookup"><span data-stu-id="d2863-118">The client is responsible for advancing its position in the stream.</span></span> <span data-ttu-id="d2863-119">つまりクライアントはいつでも参加でき、イベントを再生できます。</span><span class="sxs-lookup"><span data-stu-id="d2863-119">That means a client can join at any time, and can replay events.</span></span>

<span data-ttu-id="d2863-120">コンシューマー側には、いくつかの一般的なバリエーションがあります。</span><span class="sxs-lookup"><span data-stu-id="d2863-120">On the consumer side, there are some common variations:</span></span>

- <span data-ttu-id="d2863-121">**簡単なイベント処理**。</span><span class="sxs-lookup"><span data-stu-id="d2863-121">**Simple event processing**.</span></span> <span data-ttu-id="d2863-122">イベントがコンシューマーのアクションを即時トリガーします。</span><span class="sxs-lookup"><span data-stu-id="d2863-122">An event immediately triggers an action in the consumer.</span></span> <span data-ttu-id="d2863-123">たとえば、お客様は Service Bus トリガーを備えた Azure Functions を使用でき、それにより、Service Bus トピックにメッセージが発行されるたびに関数が実行されます。</span><span class="sxs-lookup"><span data-stu-id="d2863-123">For example, you could use Azure Functions with a Service Bus trigger, so that a function executes whenever a message is published to a Service Bus topic.</span></span>

- <span data-ttu-id="d2863-124">**複合イベント処理**。</span><span class="sxs-lookup"><span data-stu-id="d2863-124">**Complex event processing**.</span></span> <span data-ttu-id="d2863-125">コンシューマーは一連のイベントを処理し、Azure Stream Analytics または Apache Storm などのテクノロジを使用してイベント データのパターンを検索します。</span><span class="sxs-lookup"><span data-stu-id="d2863-125">A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm.</span></span> <span data-ttu-id="d2863-126">たとえば、組み込みデバイスからの測定値を時間枠で集計し、移動平均が特定のしきい値を超えた場合に通知を生成することができます。</span><span class="sxs-lookup"><span data-stu-id="d2863-126">For example, you could aggregate readings from an embedded device over a time window, and generate a notification if the moving average crosses a certain threshold.</span></span>

- <span data-ttu-id="d2863-127">**イベント ストリーム処理**.</span><span class="sxs-lookup"><span data-stu-id="d2863-127">**Event stream processing**.</span></span> <span data-ttu-id="d2863-128">Azure IoT Hub や Apache Kafka などのデータ ストリーミング プラットフォームを、イベントを取り込んでストリーム プロセッサにフィードするパイプラインとして使用します。</span><span class="sxs-lookup"><span data-stu-id="d2863-128">Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors.</span></span> <span data-ttu-id="d2863-129">ストリーム プロセッサは、ストリームの処理や変換を行います。</span><span class="sxs-lookup"><span data-stu-id="d2863-129">The stream processors act to process or transform the stream.</span></span> <span data-ttu-id="d2863-130">アプリケーションの異なるサブシステムに対して、複数のストリーム プロセッサが存在する場合があります。</span><span class="sxs-lookup"><span data-stu-id="d2863-130">There may be multiple stream processors for different subsystems of the application.</span></span> <span data-ttu-id="d2863-131">この手法は IoT ワークロードに適しています。</span><span class="sxs-lookup"><span data-stu-id="d2863-131">This approach is a good fit for IoT workloads.</span></span>

<span data-ttu-id="d2863-132">イベントのソースはシステムの外部にある場合があります。たとえば、IoT ソリューションの物理デバイスなどです。</span><span class="sxs-lookup"><span data-stu-id="d2863-132">The source of the events may be external to the system, such as physical devices in an IoT solution.</span></span> <span data-ttu-id="d2863-133">その場合、システムはデータ ソースで必要なボリュームとスループットでデータを取り込むことができる必要があります。</span><span class="sxs-lookup"><span data-stu-id="d2863-133">In that case, the system must be able to ingest the data at the volume and throughput that is required by the data source.</span></span>

<span data-ttu-id="d2863-134">上記の図では、コンシューマーが種類ごとに 1 つのボックスで表示されています。</span><span class="sxs-lookup"><span data-stu-id="d2863-134">In the logical diagram above, each type of consumer is shown as a single box.</span></span> <span data-ttu-id="d2863-135">実際には、1 つのコンシューマーに複数のインスタンスがあるのが一般的ですが、それはコンシューマーがシステムの単一障害点にならないようにするためです。</span><span class="sxs-lookup"><span data-stu-id="d2863-135">In practice, it's common to have multiple instances of a consumer, to avoid having the consumer become a single point of failure in system.</span></span> <span data-ttu-id="d2863-136">イベントのボリュームと頻度を制御するのに、複数のインスタンスが必要になることもあります。</span><span class="sxs-lookup"><span data-stu-id="d2863-136">Multiple instances might also be necessary to handle the volume and frequency of events.</span></span> <span data-ttu-id="d2863-137">また、1 つのコンシューマーが複数のスレッドのイベントを処理することもあります。</span><span class="sxs-lookup"><span data-stu-id="d2863-137">Also, a single consumer might process events on multiple threads.</span></span> <span data-ttu-id="d2863-138">イベントを順番に処理しなければならない、または正確に 1 回のセマンティクスが必要な場合には、このために課題が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="d2863-138">This can create challenges if events must be processed in order, or require exactly-once semantics.</span></span> <span data-ttu-id="d2863-139">[「Minimize Coordination (調整を最小限に抑える)」][minimize-coordination]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d2863-139">See [Minimize Coordination][minimize-coordination].</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="d2863-140">このアーキテクチャを使用する状況</span><span class="sxs-lookup"><span data-stu-id="d2863-140">When to use this architecture</span></span>

- <span data-ttu-id="d2863-141">複数のサブシステムで同じイベントを処理する必要がある。</span><span class="sxs-lookup"><span data-stu-id="d2863-141">Multiple subsystems must process the same events.</span></span>
- <span data-ttu-id="d2863-142">最小のタイム ラグのリアルタイム処理。</span><span class="sxs-lookup"><span data-stu-id="d2863-142">Real-time processing with minimum time lag.</span></span>
- <span data-ttu-id="d2863-143">パターン マッチングや時間枠での集計などの複合イベント処理。</span><span class="sxs-lookup"><span data-stu-id="d2863-143">Complex event processing, such as pattern matching or aggregation over time windows.</span></span>
- <span data-ttu-id="d2863-144">IoT などの高ボリューム、高ベロシティのデータ。</span><span class="sxs-lookup"><span data-stu-id="d2863-144">High volume and high velocity of data, such as IoT.</span></span>

## <a name="benefits"></a><span data-ttu-id="d2863-145">メリット</span><span class="sxs-lookup"><span data-stu-id="d2863-145">Benefits</span></span>

- <span data-ttu-id="d2863-146">プロデューサーとコンシューマーが分離。</span><span class="sxs-lookup"><span data-stu-id="d2863-146">Producers and consumers are decoupled.</span></span>
- <span data-ttu-id="d2863-147">ポイント ツー ポイント統合でない。</span><span class="sxs-lookup"><span data-stu-id="d2863-147">No point-to point-integrations.</span></span> <span data-ttu-id="d2863-148">システムに新しいコンシューマーを追加するのが簡単。</span><span class="sxs-lookup"><span data-stu-id="d2863-148">It's easy to add new consumers to the system.</span></span>
- <span data-ttu-id="d2863-149">コンシューマーは、イベントが到着するとすぐに応答可能。</span><span class="sxs-lookup"><span data-stu-id="d2863-149">Consumers can respond to events immediately as they arrive.</span></span>
- <span data-ttu-id="d2863-150">高い拡張性と分散。</span><span class="sxs-lookup"><span data-stu-id="d2863-150">Highly scalable and distributed.</span></span>
- <span data-ttu-id="d2863-151">サブシステムにイベント ストリームの独立したビューがある。</span><span class="sxs-lookup"><span data-stu-id="d2863-151">Subsystems have independent views of the event stream.</span></span>

## <a name="challenges"></a><span data-ttu-id="d2863-152">課題</span><span class="sxs-lookup"><span data-stu-id="d2863-152">Challenges</span></span>

- <span data-ttu-id="d2863-153">保証された配信。</span><span class="sxs-lookup"><span data-stu-id="d2863-153">Guaranteed delivery.</span></span> <span data-ttu-id="d2863-154">一部のシステムでは (特に IoT シナリオで)、イベントが配信されたことを保証することが重要です。</span><span class="sxs-lookup"><span data-stu-id="d2863-154">In some systems, especially in IoT scenarios, it's crucial to guarantee that events are delivered.</span></span>
- <span data-ttu-id="d2863-155">イベントを順番に、または正確に 1 回処理。</span><span class="sxs-lookup"><span data-stu-id="d2863-155">Processing events in order or exactly once.</span></span> <span data-ttu-id="d2863-156">各コンシューマー タイプは通常、回復性とスケーラビリティのために複数のインスタンスで実行されます。</span><span class="sxs-lookup"><span data-stu-id="d2863-156">Each consumer type typically runs in multiple instances, for resiliency and scalability.</span></span> <span data-ttu-id="d2863-157">(コンシューマー タイプ内で) 順番にイベントを処理する必要がある場合や、処理ロジックがべき等でない場合は、このために課題が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="d2863-157">This can create a challenge if the events must be processed in order (within a consumer type), or if the processing logic is not idempotent.</span></span>

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[minimize-coordination]: ../design-principles/minimize-coordination.md
