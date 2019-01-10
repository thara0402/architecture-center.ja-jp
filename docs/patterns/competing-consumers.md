---
title: 競合コンシューマー パターン
titleSuffix: Cloud Design Patterns
description: 複数の同時実行コンシューマーが、同じメッセージング チャネルで受信したメッセージを処理できるようにします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 77459ff42422969acdc83e66535197547d555de1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112109"
---
# <a name="competing-consumers-pattern"></a><span data-ttu-id="4cae4-104">競合コンシューマー パターン</span><span class="sxs-lookup"><span data-stu-id="4cae4-104">Competing Consumers pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="4cae4-105">複数の同時実行コンシューマーが、同じメッセージング チャネルで受信したメッセージを処理できるようにします。</span><span class="sxs-lookup"><span data-stu-id="4cae4-105">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> <span data-ttu-id="4cae4-106">このパターンでは、システムは複数のメッセージを同時に処理して、スループットを最適化し、スケーラビリティと可用性を向上させ、ワークロードのバランスを取ることができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-106">This enables a system to process multiple messages concurrently to optimize throughput, to improve scalability and availability, and to balance the workload.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="4cae4-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="4cae4-107">Context and problem</span></span>

<span data-ttu-id="4cae4-108">クラウドで実行されるアプリケーションには、多数の要求を処理することが求められます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-108">An application running in the cloud is expected to handle a large number of requests.</span></span> <span data-ttu-id="4cae4-109">各要求を同期的に処理するのではなく、アプリケーションがメッセージング システムを介して、要求を非同期で処理する別のサービス (コンシューマー サービス) に要求を渡すのが一般的な手法です。</span><span class="sxs-lookup"><span data-stu-id="4cae4-109">Rather than process each request synchronously, a common technique is for the application to pass them through a messaging system to another service (a consumer service) that handles them asynchronously.</span></span> <span data-ttu-id="4cae4-110">この方法では、要求が処理されている間に、アプリケーションのビジネス ロジックがブロックされないようにすることができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-110">This strategy helps to ensure that the business logic in the application isn't blocked while the requests are being processed.</span></span>

<span data-ttu-id="4cae4-111">多くの理由から、要求の数が経時的に大きく変わる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-111">The number of requests can vary significantly over time for many reasons.</span></span> <span data-ttu-id="4cae4-112">ユーザー アクティビティや複数のテナントから集約された要求の急増により、予測不可能なワークロードが発生する場合があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-112">A sudden increase in user activity or aggregated requests coming from multiple tenants can cause an unpredictable workload.</span></span> <span data-ttu-id="4cae4-113">ピーク時には、システムは 1 秒あたり何百件もの要求を処理する必要がありますが、他の時間帯は要求数が非常に少ない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-113">At peak hours a system might need to process many hundreds of requests per second, while at other times the number could be very small.</span></span> <span data-ttu-id="4cae4-114">さらに、これらの要求を処理するために実行される作業の性質は非常に多様であると考えられます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-114">Additionally, the nature of the work performed to handle these requests might be highly variable.</span></span> <span data-ttu-id="4cae4-115">コンシューマー サービスのインスタンスを 1 つしか使用していない場合、そのインスタンスが要求で溢れてしまう可能性があります。また、アプリケーションから送信されるメッセージの流入によって、メッセージング システムが過負荷になることもあります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-115">Using a single instance of the consumer service can cause that instance to become flooded with requests, or the messaging system might be overloaded by an influx of messages coming from the application.</span></span> <span data-ttu-id="4cae4-116">この変動するワークロードを処理するために、システムはコンシューマー サービスの複数のインスタンスを実行できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-116">To handle this fluctuating workload, the system can run multiple instances of the consumer service.</span></span> <span data-ttu-id="4cae4-117">ただし、各メッセージが 1 つのコンシューマーにのみ配信されるように、これらのコンシューマーを調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-117">However, these consumers must be coordinated to ensure that each message is only delivered to a single consumer.</span></span> <span data-ttu-id="4cae4-118">また、コンシューマー間でワークロードを負荷分散して、インスタンスがボトルネックにならないようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-118">The workload also needs to be load balanced across consumers to prevent an instance from becoming a bottleneck.</span></span>

## <a name="solution"></a><span data-ttu-id="4cae4-119">解決策</span><span class="sxs-lookup"><span data-stu-id="4cae4-119">Solution</span></span>

<span data-ttu-id="4cae4-120">メッセージ キューを使用して、アプリケーションとコンシューマー サービスのインスタンス間の通信チャネルを実装します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-120">Use a message queue to implement the communication channel between the application and the instances of the consumer service.</span></span> <span data-ttu-id="4cae4-121">アプリケーションは要求をメッセージの形でキューに入れ、コンシューマー サービス インスタンスはキューからメッセージを受け取って処理します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-121">The application posts requests in the form of messages to the queue, and the consumer service instances receive messages from the queue and process them.</span></span> <span data-ttu-id="4cae4-122">この方法を使用すると、コンシューマー サービス インスタンスの同じプールで、アプリケーションのインスタンスからのメッセージを処理できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-122">This approach enables the same pool of consumer service instances to handle messages from any instance of the application.</span></span> <span data-ttu-id="4cae4-123">次の図は、メッセージ キューを使用した複数のサービス インスタンスへの処理の分散を示しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-123">The figure illustrates using a message queue to distribute work to instances of a service.</span></span>

![メッセージ キューを使用した複数のサービス インスタンスへの処理の分散](./_images/competing-consumers-diagram.png)

<span data-ttu-id="4cae4-125">このソリューションには次の利点があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-125">This solution has the following benefits:</span></span>

- <span data-ttu-id="4cae4-126">アプリケーション インスタンスから送信される要求量が大幅に変動する場合でも要求を処理できる負荷平準化システムがあります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-126">It provides a load-leveled system that can handle wide variations in the volume of requests sent by application instances.</span></span> <span data-ttu-id="4cae4-127">キューは、アプリケーション インスタンスとコンシューマー サービス インスタンス間のバッファーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-127">The queue acts as a buffer between the application instances and the consumer service instances.</span></span> <span data-ttu-id="4cae4-128">これは、「[Queue-based Load Leveling pattern](./queue-based-load-leveling.md)」(キューベースの負荷平準化パターン) で説明されているよう、アプリケーションとサービス インスタンスの両方の可用性と応答性に影響を最小限に抑えるために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-128">This can help to minimize the impact on availability and responsiveness for both the application and the service instances, as described by the [Queue-based Load Leveling pattern](./queue-based-load-leveling.md).</span></span> <span data-ttu-id="4cae4-129">一部の実行時間が長い処理が必要なメッセージの処理では、コンシューマー サービスの他のインスタンスによる他のメッセージの同時処理が回避されません。</span><span class="sxs-lookup"><span data-stu-id="4cae4-129">Handling a message that requires some long-running processing doesn't prevent other messages from being handled concurrently by other instances of the consumer service.</span></span>

- <span data-ttu-id="4cae4-130">そのため、信頼性が向上します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-130">It improves reliability.</span></span> <span data-ttu-id="4cae4-131">プロデューサーがこのパターンを使用するのではなく、コンシューマーと直接通信し、コンシューマーの監視は行わない場合、コンシューマーが失敗したときにメッセージが失われたり、処理に失敗したりする可能性が高くなります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-131">If a producer communicates directly with a consumer instead of using this pattern, but doesn't monitor the consumer, there's a high probability that messages could be lost or fail to be processed if the consumer fails.</span></span> <span data-ttu-id="4cae4-132">このパターンでは、メッセージは特定のサービス インスタンスに送信されません。</span><span class="sxs-lookup"><span data-stu-id="4cae4-132">In this pattern, messages aren't sent to a specific service instance.</span></span> <span data-ttu-id="4cae4-133">失敗したサービス インスタンスによってプロデューサーはブロックされず、機能している任意のサービス インスタンスがメッセージを処理できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-133">A failed service instance won't block a producer, and messages can be processed by any working service instance.</span></span>

- <span data-ttu-id="4cae4-134">コンシューマー間、またはプロデューサー インスタンスとコンシューマー インスタンス間に複雑な調整は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="4cae4-134">It doesn't require complex coordination between the consumers, or between the producer and the consumer instances.</span></span> <span data-ttu-id="4cae4-135">メッセージ キューによって、各メッセージは少なくとも 1 回は配信されます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-135">The message queue ensures that each message is delivered at least once.</span></span>

- <span data-ttu-id="4cae4-136">スケーラブルです。</span><span class="sxs-lookup"><span data-stu-id="4cae4-136">It's scalable.</span></span> <span data-ttu-id="4cae4-137">メッセージ量の変動に応じて、システムはコンシューマー サービスのインスタンス数を動的に増減することができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-137">The system can dynamically increase or decrease the number of instances of the consumer service as the volume of messages fluctuates.</span></span>

- <span data-ttu-id="4cae4-138">メッセージ キューでトランザクションの読み取り操作を提供する場合、回復性を改善できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-138">It can improve resiliency if the message queue provides transactional read operations.</span></span> <span data-ttu-id="4cae4-139">コンシューマー サービス インスタンスがトランザクション操作の一環としてメッセージの読み取りと処理を行い、コンシューマー サービス インスタンスが失敗した場合、このパターンによってメッセージはキューに返され、コンシューマー サービスの別インスタンスで受け取り、処理することができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-139">If a consumer service instance reads and processes the message as part of a transactional operation, and the consumer service instance fails, this pattern can ensure that the message will be returned to the queue to be picked up and handled by another instance of the consumer service.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="4cae4-140">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="4cae4-140">Issues and considerations</span></span>

<span data-ttu-id="4cae4-141">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-141">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="4cae4-142">**メッセージの順序付け**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-142">**Message ordering**.</span></span> <span data-ttu-id="4cae4-143">コンシューマー サービス インスタンスでメッセージを受け取る順序は保証されていません。また、メッセージが作成された順序を反映しているとは限りません。</span><span class="sxs-lookup"><span data-stu-id="4cae4-143">The order in which consumer service instances receive messages isn't guaranteed, and doesn't necessarily reflect the order in which the messages were created.</span></span> <span data-ttu-id="4cae4-144">メッセージが処理される順序への依存を排除できるので、メッセージ処理がべき等になるようにシステムを設計します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-144">Design the system to ensure that message processing is idempotent because this will help to eliminate any dependency on the order in which messages are handled.</span></span> <span data-ttu-id="4cae4-145">詳細については、Jonathan Oliver のブログ「[Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/)」(べき等パターン) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-145">For more information, see [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathon Oliver’s blog.</span></span>

    > <span data-ttu-id="4cae4-146">Microsoft Azure Service Bus Queues は、メッセージ セッションを使用して、保証された先入れ先出しの順序を実装することができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-146">Microsoft Azure Service Bus Queues can implement guaranteed first-in-first-out ordering of messages by using message sessions.</span></span> <span data-ttu-id="4cae4-147">詳細については、「[セッションを使用するメッセージング パターン](https://msdn.microsoft.com/magazine/jj863132.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-147">For more information, see [Messaging Patterns Using Sessions](https://msdn.microsoft.com/magazine/jj863132.aspx).</span></span>

- <span data-ttu-id="4cae4-148">**回復性に対応するサービスの設計**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-148">**Designing services for resiliency**.</span></span> <span data-ttu-id="4cae4-149">失敗したサービス インスタンスを検出して再起動するようにシステムを設計する場合、必要に応じて、サービス インスタンスがべき等操作として実行する処理を実装し、単一のメッセージが複数回取得および処理される影響を最小限に抑えます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-149">If the system is designed to detect and restart failed service instances, it might be necessary to implement the processing performed by the service instances as idempotent operations to minimize the effects of a single message being retrieved and processed more than once.</span></span>

- <span data-ttu-id="4cae4-150">**有害メッセージの検出**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-150">**Detecting poison messages**.</span></span> <span data-ttu-id="4cae4-151">不適切な形式のメッセージ、または使用できないリソースにアクセスする必要があるタスクによって、サービス インスタンスが失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-151">A malformed message, or a task that requires access to resources that aren't available, can cause a service instance to fail.</span></span> <span data-ttu-id="4cae4-152">システムでこのようなメッセージがキューに返されないように防ぎ、代わりにこれらのメッセージをキャプチャしてどこか別の場所に格納して、必要に応じて分析できるようにすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="4cae4-152">The system should prevent such messages being returned to the queue, and instead capture and store the details of these messages elsewhere so that they can be analyzed if necessary.</span></span>

- <span data-ttu-id="4cae4-153">**結果の処理**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-153">**Handling results**.</span></span> <span data-ttu-id="4cae4-154">メッセージを処理するサービス インスタンスは、メッセージを生成するアプリケーション ロジックから完全に切り離されているので、直接通信できない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-154">The service instance handling a message is fully decoupled from the application logic that generates the message, and they might not be able to communicate directly.</span></span> <span data-ttu-id="4cae4-155">サービス インスタンスから、アプリケーション ロジックに戻す必要がある結果が生成される場合、この情報は、両方からアクセスできる場所に保存する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-155">If the service instance generates results that must be passed back to the application logic, this information must be stored in a location that's accessible to both.</span></span> <span data-ttu-id="4cae4-156">アプリケーション ロジックで不完全なデータが取得されないようにするために、システムで処理が完了したときを示す必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-156">In order to prevent the application logic from retrieving incomplete data the system must indicate when processing is complete.</span></span>

     > <span data-ttu-id="4cae4-157">Azure を使用している場合、ワーカー プロセスで専用のメッセージ返信キューを使用することで、アプリケーション ロジックに結果を戻すことができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-157">If you're using Azure, a worker process can pass results back to the application logic by using a dedicated message reply queue.</span></span> <span data-ttu-id="4cae4-158">アプリケーション ロジックで、このような結果を元のメッセージと関連付けられる必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-158">The application logic must be able to correlate these results with the original message.</span></span> <span data-ttu-id="4cae4-159">このシナリオの詳細については、「[Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx)」(非同期メッセージングの基本) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-159">This scenario is described in more detail in the [Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx).</span></span>

- <span data-ttu-id="4cae4-160">**メッセージング システムのスケーリング**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-160">**Scaling the messaging system**.</span></span> <span data-ttu-id="4cae4-161">大規模なソリューションの場合、1 つのメッセージ キューが大量のメッセージで一杯になり、システムのボトルネックになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-161">In a large-scale solution, a single message queue could be overwhelmed by the number of messages and become a bottleneck in the system.</span></span> <span data-ttu-id="4cae4-162">このような場合は、メッセージング システムをパーティション分割し、特定のプロデューサーからのメッセージを特定のキューに送信するか、負荷分散を使用して、複数のメッセージ キュー全体にメッセージを分散させることを検討します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-162">In this situation, consider partitioning the messaging system to send messages from specific producers to a particular queue, or use load balancing to distribute messages across multiple message queues.</span></span>

- <span data-ttu-id="4cae4-163">**メッセージング システムの信頼性を確保**。</span><span class="sxs-lookup"><span data-stu-id="4cae4-163">**Ensuring reliability of the messaging system**.</span></span> <span data-ttu-id="4cae4-164">信頼できるメッセージング システムは、アプリケーションがメッセージをキューに格納した後に、メッセージが失われないことを保証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-164">A reliable messaging system is needed to guarantee that after the application enqueues a message it won't be lost.</span></span> <span data-ttu-id="4cae4-165">これは、すべてのメッセージを少なくとも 1 回配信するために重要です。</span><span class="sxs-lookup"><span data-stu-id="4cae4-165">This is essential for ensuring that all messages are delivered at least once.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="4cae4-166">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="4cae4-166">When to use this pattern</span></span>

<span data-ttu-id="4cae4-167">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-167">Use this pattern when:</span></span>

- <span data-ttu-id="4cae4-168">アプリケーションのワークロードは、非同期に実行できる複数のタスクに分割されます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-168">The workload for an application is divided into tasks that can run asynchronously.</span></span>
- <span data-ttu-id="4cae4-169">タスクは独立しており、並列して実行できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-169">Tasks are independent and can run in parallel.</span></span>
- <span data-ttu-id="4cae4-170">作業量の変動が大きい場合、スケーラブルなソリューションが必要です。</span><span class="sxs-lookup"><span data-stu-id="4cae4-170">The volume of work is highly variable, requiring a scalable solution.</span></span>
- <span data-ttu-id="4cae4-171">ソリューションは高可用性を提供する必要があります。また、タスクの処理が失敗した場合に回復できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-171">The solution must provide high availability, and must be resilient if the processing for a task fails.</span></span>

<span data-ttu-id="4cae4-172">このパターンが適さない状況</span><span class="sxs-lookup"><span data-stu-id="4cae4-172">This pattern might not be useful when:</span></span>

- <span data-ttu-id="4cae4-173">アプリケーションのワークロードを個別のタスクに分離することが容易ではない場合、またはタスク間の依存度が高い場合。</span><span class="sxs-lookup"><span data-stu-id="4cae4-173">It's not easy to separate the application workload into discrete tasks, or there's a high degree of dependence between tasks.</span></span>
- <span data-ttu-id="4cae4-174">タスクを同期して実行する必要があり、アプリケーション ロジックで 1 つのタスクが完了するまで待ってから続行する必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="4cae4-174">Tasks must be performed synchronously, and the application logic must wait for a task to complete before continuing.</span></span>
- <span data-ttu-id="4cae4-175">特定の順序でタスクを実行する必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="4cae4-175">Tasks must be performed in a specific sequence.</span></span>

> <span data-ttu-id="4cae4-176">一部のメッセージング システムは、プロデューサーがメッセージをグループ化し、そのすべてを同じコンシューマーが処理するように確保するセッションをサポートしています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-176">Some messaging systems support sessions that enable a producer to group messages together and ensure that they're all handled by the same consumer.</span></span> <span data-ttu-id="4cae4-177">このメカニズムを優先度が付けられたメッセージ (優先度付けがサポートされている場合) に使用して、プロデューサーから単一のコンシューマーに対して順番にメッセージを配信するメッセージの順序付けのフォームを実装することができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-177">This mechanism can be used with prioritized messages (if they are supported) to implement a form of message ordering that delivers messages in sequence from a producer to a single consumer.</span></span>

## <a name="example"></a><span data-ttu-id="4cae4-178">例</span><span class="sxs-lookup"><span data-stu-id="4cae4-178">Example</span></span>

<span data-ttu-id="4cae4-179">Azure には、このパターンを実装するメカニズムとして機能するストレージ キューと Service Bus キューが用意されています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-179">Azure provides storage queues and Service Bus queues that can act as a mechanism for implementing this pattern.</span></span> <span data-ttu-id="4cae4-180">アプリケーション ロジックで、メッセージをキューに投稿できます。また、1 つ以上のロールでタスクとして実装されているコンシューマーで、このキューからメッセージを取得し、処理することができます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-180">The application logic can post messages to a queue, and consumers implemented as tasks in one or more roles can retrieve messages from this queue and process them.</span></span> <span data-ttu-id="4cae4-181">Service Bus キューでは回復性のために、コンシューマーがキューからメッセージを取得するときに `PeekLock` モードを使用できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-181">For resiliency, a Service Bus queue enables a consumer to use `PeekLock` mode when it retrieves a message from the queue.</span></span> <span data-ttu-id="4cae4-182">このモードでは、メッセージは実際に削除されず、他のコンシューマーには単に非表示にされます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-182">This mode doesn't actually remove the message, but simply hides it from other consumers.</span></span> <span data-ttu-id="4cae4-183">元のコンシューマーは、処理の完了時にメッセージを削除できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-183">The original consumer can delete the message when it's finished processing it.</span></span> <span data-ttu-id="4cae4-184">コンシューマーが失敗した場合はピーク ロックがタイムアウトし、メッセージは再び可視状態になり、別のコンシューマーが取得できるようになります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-184">If the consumer fails, the peek lock will time out and the message will become visible again, allowing another consumer to retrieve it.</span></span>

<span data-ttu-id="4cae4-185">Azure Service Bus キューの使用の詳細については、「[Service Bus のキュー、トピック、サブスクリプション](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-185">For detailed information on using Azure Service Bus queues, see [Service Bus queues, topics, and subscriptions](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).</span></span>

<span data-ttu-id="4cae4-186">Azure Storage キューの使用の詳細については、「[.NET を使用して Azure Queue Storage を使用する](/azure/storage/queues/storage-dotnet-how-to-use-queues)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-186">For information on using Azure storage queues, see [Get started with Azure Queue storage using .NET](/azure/storage/queues/storage-dotnet-how-to-use-queues).</span></span>

<span data-ttu-id="4cae4-187">[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) で入手できる CompetingConsumers ソリューションの `QueueManager` クラスから引用した次のコードは、Web または worker ロールで `Start` イベント ハンドラーの `QueueClient` インスタンスを使用してキューを作成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-187">The following code from the `QueueManager` class in CompetingConsumers solution available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) shows how you can create a queue by using a `QueueClient` instance in the `Start` event handler in a web or worker role.</span></span>

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

<span data-ttu-id="4cae4-188">次のコード スニペットは、アプリケーションでメッセージのバッチを作成し、キューに送信する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-188">The next code snippet shows how an application can create and send a batch of messages to the queue.</span></span>

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

<span data-ttu-id="4cae4-189">次のコードは、コンシューマー サービス インスタンスが、イベント主導の手法に従ってキューからメッセージを受け取る方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-189">The following code shows how a consumer service instance can receive messages from the queue by following an event-driven approach.</span></span> <span data-ttu-id="4cae4-190">`ReceiveMessages` メソッドに対する `processMessageTask` パラメーターは、メッセージを受け取ったときに実行するコードを参照するデリゲートです。</span><span class="sxs-lookup"><span data-stu-id="4cae4-190">The `processMessageTask` parameter to the `ReceiveMessages` method is a delegate that references the code to run when a message is received.</span></span> <span data-ttu-id="4cae4-191">このコードは非同期に実行されます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-191">This code is run asynchronously.</span></span>

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

<span data-ttu-id="4cae4-192">Azure などで使用できる自動スケール機能を使用して、キューの長さの変動に応じてロール インスタンスを開始および停止できます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-192">Note that autoscaling features, such as those available in Azure, can be used to start and stop role instances as the queue length fluctuates.</span></span> <span data-ttu-id="4cae4-193">詳細については、[自動スケールのガイダンス](https://msdn.microsoft.com/library/dn589774.aspx)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-193">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="4cae4-194">また、ロール インスタンスとワーカー プロセス間に 1 対 1 の対応が維持されているとは限りません。1 つのロール インスタンスが複数のワーカー プロセスを実装している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-194">Also, it's not necessary to maintain a one-to-one correspondence between role instances and worker processes&mdash;a single role instance can implement multiple worker processes.</span></span> <span data-ttu-id="4cae4-195">詳細については、「[Compute Resource Consolidation pattern](./compute-resource-consolidation.md)」(コンピューティング リソース統合パターン) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4cae4-195">For more information, see [Compute Resource Consolidation pattern](./compute-resource-consolidation.md).</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="4cae4-196">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="4cae4-196">Related patterns and guidance</span></span>

<span data-ttu-id="4cae4-197">このパターンを実装する場合は、次のパターンとガイダンスが関連している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-197">The following patterns and guidance might be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="4cae4-198">[非同期メッセージングの基本](https://msdn.microsoft.com/library/dn589781.aspx)。</span><span class="sxs-lookup"><span data-stu-id="4cae4-198">[Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx).</span></span> <span data-ttu-id="4cae4-199">メッセージ キューは、非同期通信メカニズムです。</span><span class="sxs-lookup"><span data-stu-id="4cae4-199">Message queues are an asynchronous communications mechanism.</span></span> <span data-ttu-id="4cae4-200">コンシューマー サービスがアプリケーションに返信を送信する必要がある場合、状況に応じて何らかの形式の応答メッセージングを実装します。</span><span class="sxs-lookup"><span data-stu-id="4cae4-200">If a consumer service needs to send a reply to an application, it might be necessary to implement some form of response messaging.</span></span> <span data-ttu-id="4cae4-201">「Asynchronous Messaging Primer」(非同期メッセージングの基本) では、メッセージ キューを使用して要求/返信メッセージングを実装する方法が説明されています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-201">The Asynchronous Messaging Primer provides information on how to implement request/reply messaging using message queues.</span></span>

- <span data-ttu-id="4cae4-202">[自動スケール ガイダンス](https://msdn.microsoft.com/library/dn589774.aspx)。</span><span class="sxs-lookup"><span data-stu-id="4cae4-202">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="4cae4-203">キュー アプリケーションの投稿メッセージの長さは変動するので、コンシューマー サービスのインスタンスを開始および停止できることがあります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-203">It might be possible to start and stop instances of a consumer service since the length of the queue applications post messages on varies.</span></span> <span data-ttu-id="4cae4-204">自動スケールは、ピーク時処理中のスループットの維持に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-204">Autoscaling can help to maintain throughput during times of peak processing.</span></span>

- <span data-ttu-id="4cae4-205">[Compute Resource Consolidation パターン](./compute-resource-consolidation.md)。</span><span class="sxs-lookup"><span data-stu-id="4cae4-205">[Compute Resource Consolidation pattern](./compute-resource-consolidation.md).</span></span> <span data-ttu-id="4cae4-206">複数のインスタンスのコンシューマー サービスを 1 つのプロセスに統合して、コストと管理のオーバーヘッドを軽減できることがあります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-206">It might be possible to consolidate multiple instances of a consumer service into a single process to reduce costs and management overhead.</span></span> <span data-ttu-id="4cae4-207">「Compute Resource Consolidation」(コンピューティング リソース統合パターン) では、この手法に従う場合の利点とトレードオフについて説明しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-207">The Compute Resource Consolidation pattern describes the benefits and tradeoffs of following this approach.</span></span>

- <span data-ttu-id="4cae4-208">[キュー ベースの負荷平準化パターン](./queue-based-load-leveling.md)。</span><span class="sxs-lookup"><span data-stu-id="4cae4-208">[Queue-based Load Leveling pattern](./queue-based-load-leveling.md).</span></span> <span data-ttu-id="4cae4-209">メッセージ キューを導入すると、システムに回復性が加わり、アプリケーション インスタンスからの変動が大きい要求量をサービス インスタンスで処理できるようになります。</span><span class="sxs-lookup"><span data-stu-id="4cae4-209">Introducing a message queue can add resiliency to the system, enabling service instances to handle widely varying volumes of requests from application instances.</span></span> <span data-ttu-id="4cae4-210">メッセージ キューはバッファーとして機能し、負荷が平準化されます。</span><span class="sxs-lookup"><span data-stu-id="4cae4-210">The message queue acts as a buffer, which levels the load.</span></span> <span data-ttu-id="4cae4-211">「Queue-based Load Leveling pattern」(キューベースの負荷平準化パターン) では、このスキーマについて詳しく説明しています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-211">The Queue-based Load Leveling pattern describes this scenario in more detail.</span></span>

- <span data-ttu-id="4cae4-212">このパターンには、[サンプル アプリケーション](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers)が関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="4cae4-212">This pattern has a [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) associated with it.</span></span>
