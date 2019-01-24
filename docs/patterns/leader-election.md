---
title: リーダー選定パターン
titleSuffix: Cloud Design Patterns
description: 1 つのインスタンスを、他のインスタンスの管理を担当するリーダーとして選定することで、分散アプリケーション内で連携するタスク インスタンスのコレクションによって実行されるアクションを調整します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1b74228fcceecb350d8df9b7ca2327b5de329a07
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486125"
---
# <a name="leader-election-pattern"></a><span data-ttu-id="b17dc-104">リーダー選定パターン</span><span class="sxs-lookup"><span data-stu-id="b17dc-104">Leader Election pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="b17dc-105">他のインスタンスを管理する役割を担うリーダーとして 1 つのインスタンスを選択することで、分散アプリケーション内で連携するインスタンスのコレクションによって実行されるアクションを調整します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-105">Coordinate the actions performed by a collection of collaborating instances in a distributed application by electing one instance as the leader that assumes responsibility for managing the others.</span></span> <span data-ttu-id="b17dc-106">これにより、インスタンスが互いに競合して、共有リソースとの競合を引き起こしたり、他のインスタンスが実行されている作業に誤って干渉したりしないようにできます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-106">This can help to ensure that instances don't conflict with each other, cause contention for shared resources, or inadvertently interfere with the work that other instances are performing.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b17dc-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="b17dc-107">Context and problem</span></span>

<span data-ttu-id="b17dc-108">一般的なクラウド アプリケーションには、協調した動きをする多くのタスクがあります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-108">A typical cloud application has many tasks acting in a coordinated manner.</span></span> <span data-ttu-id="b17dc-109">これらのタスクはすべて、同じコードを実行し、同じリソースへのアクセスを必要とするインスタンスの可能性があります。また、複雑な計算の個々の部分を実行するために、複数のタスクが並列で動作する場合もあります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-109">These tasks could all be instances running the same code and requiring access to the same resources, or they might be working together in parallel to perform the individual parts of a complex calculation.</span></span>

<span data-ttu-id="b17dc-110">タスク インスタンスは、大半の時間は個々に実行されますが、インスタンスが互いに競合して共有リソースとの競合を引き起こしたり、他のタスク インスタンスが実行されている作業に誤って干渉したりしないように、各インスタンスのアクションを調整する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-110">The task instances might run separately for much of the time, but it might also be necessary to coordinate the actions of each instance to ensure that they don’t conflict, cause contention for shared resources, or accidentally interfere with the work that other task instances are performing.</span></span>

<span data-ttu-id="b17dc-111">例: </span><span class="sxs-lookup"><span data-stu-id="b17dc-111">For example:</span></span>

- <span data-ttu-id="b17dc-112">水平方向のスケーリングを実装するクラウドベース システムでは、同一タスクの複数のインスタンスを、別のユーザーに対する各インスタンスと同時に実行できます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-112">In a cloud-based system that implements horizontal scaling, multiple instances of the same task could be running at the same time with each instance serving a different user.</span></span> <span data-ttu-id="b17dc-113">これらのインスタンスが共有リソースに書き込みを行う場合、各インスタンスが他のインスタンスによって行われた変更を上書きしないようにアクションを調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-113">If these instances write to a shared resource, it's necessary to coordinate their actions to prevent each instance from overwriting the changes made by the others.</span></span>
- <span data-ttu-id="b17dc-114">タスクが複雑な計算の個々 の要素を並列で実行している場合、すべてが完了した時点で結果が集計される必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-114">If the tasks are performing individual elements of a complex calculation in parallel, the results need to be aggregated when they all complete.</span></span>

<span data-ttu-id="b17dc-115">タスク インスタンスはすべてピアなので、コーディネーターやアグリゲーターとして機能できる自然なリーダーは存在しません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-115">The task instances are all peers, so there isn't a natural leader that can act as the coordinator or aggregator.</span></span>

## <a name="solution"></a><span data-ttu-id="b17dc-116">解決策</span><span class="sxs-lookup"><span data-stu-id="b17dc-116">Solution</span></span>

<span data-ttu-id="b17dc-117">リーダーとして機能するように、単一のタスク インスタンスが選定される必要があります。また、このインスタンスは、他の下位のタスク インスタンスのアクションを調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-117">A single task instance should be elected to act as the leader, and this instance should coordinate the actions of the other subordinate task instances.</span></span> <span data-ttu-id="b17dc-118">すべてのタスク インスタンスが同じコードを実行している場合、それぞれのインスタンスがリーダーとして機能できます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-118">If all of the task instances are running the same code, they are each capable of acting as the leader.</span></span> <span data-ttu-id="b17dc-119">そのため、2 つ以上のインスタンスが同時にリーダーのロールを担うことがないよう注意して、選定のプロセスを管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-119">Therefore, the election process must be managed carefully to prevent two or more instances taking over the leader role at the same time.</span></span>

<span data-ttu-id="b17dc-120">システムは、リーダーを選定するための堅牢なメカニズムを提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-120">The system must provide a robust mechanism for selecting the leader.</span></span> <span data-ttu-id="b17dc-121">選定のメソッドでは、ネットワークの停止やプロセスの失敗などの事態に対処する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-121">This method has to cope with events such as network outages or process failures.</span></span> <span data-ttu-id="b17dc-122">多くのソリューションでは、下位のタスク インスタンスは、任意のタイプのハートビート メソッド経由またはポーリングによって、リーダーを監視します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-122">In many solutions, the subordinate task instances monitor the leader through some type of heartbeat method, or by polling.</span></span> <span data-ttu-id="b17dc-123">指定したリーダーが予期せずに終了した場合や、ネットワーク障害によってリーダーを下位のタスク インスタンスで使用できない場合は、新しいリーダーを選定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-123">If the designated leader terminates unexpectedly, or a network failure makes the leader unavailable to the subordinate task instances, it's necessary for them to elect a new leader.</span></span>

<span data-ttu-id="b17dc-124">分散した環境にある一連のタスクからリーダーを選定するには、いくつかの戦略があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-124">There are several strategies for electing a leader among a set of tasks in a distributed environment, including:</span></span>

- <span data-ttu-id="b17dc-125">最低ランクのインスタンス ID またはプロセス ID のタスク インスタンスを選択する。</span><span class="sxs-lookup"><span data-stu-id="b17dc-125">Selecting the task instance with the lowest-ranked instance or process ID.</span></span>
- <span data-ttu-id="b17dc-126">共有されている分散ミューテックスを獲得するために競わせる。</span><span class="sxs-lookup"><span data-stu-id="b17dc-126">Racing to acquire a shared, distributed mutex.</span></span> <span data-ttu-id="b17dc-127">ミューテックスを獲得した最初のタスク インスタンスがリーダーになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-127">The first task instance that acquires the mutex is the leader.</span></span> <span data-ttu-id="b17dc-128">ただし、システムは、リーダーが終了したりシステムの他の部分から切り離されたりした場合に、別のタスク インスタンスがリーダーになれるようにミューテックスが解放されることを保証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-128">However, the system must ensure that, if the leader terminates or becomes disconnected from the rest of the system, the mutex is released to allow another task instance to become the leader.</span></span>
- <span data-ttu-id="b17dc-129">[Bully Algorithm (ブリー アルゴリズム) ](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) や [Ring Algorithm (リング アルゴリズム) ](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)など、一般的なリーダー選定のアルゴリズムの 1 つを実装する。</span><span class="sxs-lookup"><span data-stu-id="b17dc-129">Implementing one of the common leader election algorithms such as the [Bully Algorithm](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) or the [Ring Algorithm](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span> <span data-ttu-id="b17dc-130">これらのアルゴリズムでは、選定の各候補が一意の ID を保持し、他の候補と確実に通信できることを前提としています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-130">These algorithms assume that each candidate in the election has a unique ID, and that it can communicate with the other candidates reliably.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b17dc-131">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="b17dc-131">Issues and considerations</span></span>

<span data-ttu-id="b17dc-132">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="b17dc-132">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="b17dc-133">リーダー選定のプロセスでは、一時的および永続的な障害に対して復元性を備える必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-133">The process of electing a leader should be resilient to transient and persistent failures.</span></span>
- <span data-ttu-id="b17dc-134">リーダーに障害が発生した場合や、使用不可能になった場合 (通信障害に起因するなど) に、検出できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-134">It must be possible to detect when the leader has failed or has become otherwise unavailable (such as due to a communications failure).</span></span> <span data-ttu-id="b17dc-135">検出の際に必要とされる迅速さは、システムによって異なります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-135">How quickly detection is needed is system dependent.</span></span> <span data-ttu-id="b17dc-136">一部のシステムは、一時的な障害が収束するまでなどの短い時間は、リーダー不在で機能できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-136">Some systems might be able to function for a short time without a leader, during which a transient fault might be fixed.</span></span> <span data-ttu-id="b17dc-137">それ以外の場合は、リーダーの障害をただちに検出して、新しい選定をトリガーする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-137">In other cases, it might be necessary to detect leader failure immediately and trigger a new election.</span></span>
- <span data-ttu-id="b17dc-138">水平方向の自動スケーリングを実装するシステムでは、システムが規模を縮小し、一部のコンピューティング リソースをシャットダウンした場合に、リーダーが終了する場合があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-138">In a system that implements horizontal autoscaling, the leader could be terminated if the system scales back and shuts down some of the computing resources.</span></span>
- <span data-ttu-id="b17dc-139">共有されている分散ミューテックスを使用すると、ミューテックスを提供している外部サービスへの依存が可能になります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-139">Using a shared, distributed mutex introduces a dependency on the external service that provides the mutex.</span></span> <span data-ttu-id="b17dc-140">サービスは、単一障害点を構成しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-140">The service constitutes a single point of failure.</span></span> <span data-ttu-id="b17dc-141">この障害点が何らかの理由で利用できなくなった場合、システムはリーダーを選定できなくなります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-141">If it becomes unavailable for any reason, the system won't be able to elect a leader.</span></span>
- <span data-ttu-id="b17dc-142">簡単な方法は、1 つの専用プロセスをリーダーとして使用することです。</span><span class="sxs-lookup"><span data-stu-id="b17dc-142">Using a single dedicated process as the leader is a straightforward approach.</span></span> <span data-ttu-id="b17dc-143">ただし、プロセスが失敗した場合、再起動時に大幅な遅延が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-143">However, if the process fails there could be a significant delay while it's restarted.</span></span> <span data-ttu-id="b17dc-144">他のプロセスがリーダーによる操作の調整を待機している場合、結果として生じる待機時間が、それらのプロセスのパフォーマンスと応答時間に影響する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-144">The resulting latency can affect the performance and response times of other processes if they're waiting for the leader to coordinate an operation.</span></span>
- <span data-ttu-id="b17dc-145">いずれかのリーダー選定アルゴリズムを手動で実装すると、コードの調整と最適化を最も柔軟に行うことができます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-145">Implementing one of the leader election algorithms manually provides the greatest flexibility for tuning and optimizing the code.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="b17dc-146">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="b17dc-146">When to use this pattern</span></span>

<span data-ttu-id="b17dc-147">クラウド ホスト ソリューションなどの分散アプリケーション内のタスクに慎重な調整が必要であり、自然なリーダーが存在しない場合は、このパターンを使用します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-147">Use this pattern when the tasks in a distributed application, such as a cloud-hosted solution, need careful coordination and there's no natural leader.</span></span>

> <span data-ttu-id="b17dc-148">リーダーがシステムのボトルネックにならないようにします。</span><span class="sxs-lookup"><span data-stu-id="b17dc-148">Avoid making the leader a bottleneck in the system.</span></span> <span data-ttu-id="b17dc-149">リーダーの目的は、下位のタスクの作業を調整することであり、必ずしもこの作業自体に参加する必要はありません。ただし、タスクがリーダーとして選定されていない場合は、作業に参加できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-149">The purpose of the leader is to coordinate the work of the subordinate tasks, and it doesn't necessarily have to participate in this work itself&mdash;although it should be able to do so if the task isn't elected as the leader.</span></span>

<span data-ttu-id="b17dc-150">以下の場合は、このパターンの使用は適していません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-150">This pattern might not be useful if:</span></span>

- <span data-ttu-id="b17dc-151">自然なリーダーまたはリーダーとして常に機能する専用のプロセスがある。</span><span class="sxs-lookup"><span data-stu-id="b17dc-151">There's a natural leader or dedicated process that can always act as the leader.</span></span> <span data-ttu-id="b17dc-152">たとえば、タスク インスタンスを調整するシングルトン プロセスを実装できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-152">For example, it might be possible to implement a singleton process that coordinates the task instances.</span></span> <span data-ttu-id="b17dc-153">このプロセスが失敗したり正常でなくなった場合、システムはプロセスをシャットダウンして、再起動できます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-153">If this process fails or becomes unhealthy, the system can shut it down and restart it.</span></span>
- <span data-ttu-id="b17dc-154">タスク間の調整が、より簡易的な方法を使用して実現できる。</span><span class="sxs-lookup"><span data-stu-id="b17dc-154">The coordination between tasks can be achieved using a more lightweight method.</span></span> <span data-ttu-id="b17dc-155">たとえば、複数のタスク インスタンスが共有リソースへのアクセス調整を必要としている場合、アクセスを制御するために楽観的ロックまたは排他的ロックを使用すると、より優れたソリューションになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-155">For example, if several task instances simply need coordinated access to a shared resource, a better solution is to use optimistic or pessimistic locking to control access.</span></span>
- <span data-ttu-id="b17dc-156">サード パーティのソリューションがよりふさわしい。</span><span class="sxs-lookup"><span data-stu-id="b17dc-156">A third-party solution is more appropriate.</span></span> <span data-ttu-id="b17dc-157">たとえば、(Apache Hadoop に基づく) Microsoft Azure HDInsight サービスは、Apache Zookeeper が提供するサービスを使用してマップを調整し、データを収集して集計するタスクを減らします。</span><span class="sxs-lookup"><span data-stu-id="b17dc-157">For example, the Microsoft Azure HDInsight service (based on Apache Hadoop) uses the services provided by Apache Zookeeper to coordinate the map and reduce tasks that collect and summarize data.</span></span>

## <a name="example"></a><span data-ttu-id="b17dc-158">例</span><span class="sxs-lookup"><span data-stu-id="b17dc-158">Example</span></span>

<span data-ttu-id="b17dc-159">LeaderElection ソリューションの DistributedMutex プロジェクト (このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) で利用可能です) は、共有されている分散ミューテックスを実装するメカニズムを提供するために、Azure Storage Blob でリースを使用する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-159">The DistributedMutex project in the LeaderElection solution (a sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) shows how to use a lease on an Azure Storage blob to provide a mechanism for implementing a shared, distributed mutex.</span></span> <span data-ttu-id="b17dc-160">Azure クラウド サービスのロール インスタンスのグループ間でリーダーを選定するために、このミューテックスを使用できます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-160">This mutex can be used to elect a leader among a group of role instances in an Azure cloud service.</span></span> <span data-ttu-id="b17dc-161">リースを取得する最初のロール インスタンスがリーダーに選定され、リースを解放するか、リースを更新できない時点までリーダーのままになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-161">The first role instance to acquire the lease is elected the leader, and remains the leader until it releases the lease or isn't able to renew the lease.</span></span> <span data-ttu-id="b17dc-162">他のロール インスタンスは、リーダーが使用できなくなった場合に備えて、Blob リースの監視を続行できます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-162">Other role instances can continue to monitor the blob lease in case the leader is no longer available.</span></span>

> <span data-ttu-id="b17dc-163">Blob リースは、Blob 経由での排他的な書き込みロックです。</span><span class="sxs-lookup"><span data-stu-id="b17dc-163">A blob lease is an exclusive write lock over a blob.</span></span> <span data-ttu-id="b17dc-164">単一の Blob は、任意の時点で 1 つのリースだけの対象になることができます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-164">A single blob can be the subject of only one lease at any point in time.</span></span> <span data-ttu-id="b17dc-165">ロール インスタンスは指定された Blob 経由でリースを要求できます。また、その他のロール インスタンスが同じ Blob 経由でリースを保持していない場合は、確実にそのリースになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-165">A role instance can request a lease over a specified blob, and it'll be granted the lease if no other role instance holds a lease over the same blob.</span></span> <span data-ttu-id="b17dc-166">それ以外の場合、要求は例外をスローします。</span><span class="sxs-lookup"><span data-stu-id="b17dc-166">Otherwise the request will throw an exception.</span></span>
>
> <span data-ttu-id="b17dc-167">リースを無期限に保持するロール インスタンスの障害を回避するために、リースの有効期限を指定します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-167">To avoid a faulted role instance retaining the lease indefinitely, specify a lifetime for the lease.</span></span> <span data-ttu-id="b17dc-168">この有効期限が切れると、リースは使用可能になります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-168">When this expires, the lease becomes available.</span></span> <span data-ttu-id="b17dc-169">ただし、ロール インスタンスがリースを保持している間は、リースの更新を要求でき、要求した後の期間は、確実にそのリースが許可されます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-169">However, while a role instance holds the lease it can request that the lease is renewed, and it'll be granted the lease for a further period of time.</span></span> <span data-ttu-id="b17dc-170">ロール インスタンスは、リースを保持する必要がある場合、このプロセスを継続的に繰り返すことができます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-170">The role instance can continually repeat this process if it wants to retain the lease.</span></span>
> <span data-ttu-id="b17dc-171">Blob をリースする方法の詳細については、「[Lease Blob (REST API) (Blob のリース (REST API))](https://msdn.microsoft.com/library/azure/ee691972.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b17dc-171">For more information on how to lease a blob, see [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx).</span></span>

<span data-ttu-id="b17dc-172">以下の C# の例にある `BlobDistributedMutex` クラスには、ロール インスタンスを有効にして指定した Blob 経由でリースの取得を試行する `RunTaskWhenMutexAquired` メソッドが含まれます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-172">The `BlobDistributedMutex` class in the C# example below contains the `RunTaskWhenMutexAquired` method that enables a role instance to attempt to acquire a lease over a specified blob.</span></span> <span data-ttu-id="b17dc-173">Blob (名前、コンテナー、およびストレージ アカウント) の詳細は、`BlobDistributedMutex` オブジェクトが作成されたときに (このオブジェクトは、サンプル コードに含まれている単純な構造体です)、`BlobSettings` オブジェクトのコンス トラクターに渡されます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-173">The details of the blob (the name, container, and storage account) are passed to the constructor in a `BlobSettings` object when the `BlobDistributedMutex` object is created (this object is a simple struct that is included in the sample code).</span></span> <span data-ttu-id="b17dc-174">また、Blob 経由で正常にリースを取得してリーダーに選定された場合、ロール インスタンスが実行されるコードを参照する `Task` をそのコンストラクターで受け入れます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-174">The constructor also accepts a `Task` that references the code that the role instance should run if it successfully acquires the lease over the blob and is elected the leader.</span></span> <span data-ttu-id="b17dc-175">リースを取得する低レベルの詳細情報を処理するコードが、`BlobLeaseManager` という名前の個々のヘルパー クラスで実装されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b17dc-175">Note that the code that handles the low-level details of acquiring the lease is implemented in a separate helper class named `BlobLeaseManager`.</span></span>

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

<span data-ttu-id="b17dc-176">上記のコード サンプル内の `RunTaskWhenMutexAquired` メソッドは、実際にリースを取得する下記のサンプル コードに示された `RunTaskWhenBlobLeaseAcquired` メソッドを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-176">The `RunTaskWhenMutexAquired` method in the code sample above invokes the `RunTaskWhenBlobLeaseAcquired` method shown in the following code sample to actually acquire the lease.</span></span> <span data-ttu-id="b17dc-177">`RunTaskWhenBlobLeaseAcquired` メソッドが非同期で実行されます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-177">The `RunTaskWhenBlobLeaseAcquired` method runs asynchronously.</span></span> <span data-ttu-id="b17dc-178">リースが正常に取得されると、ロール インスタンスがリーダーを選定したことになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-178">If the lease is successfully acquired, the role instance has been elected the leader.</span></span> <span data-ttu-id="b17dc-179">`taskToRunWhenLeaseAcquired` デリゲートの目的は、他のロール インスタンスを調整する作業を実行することです。</span><span class="sxs-lookup"><span data-stu-id="b17dc-179">The purpose of the `taskToRunWhenLeaseAcquired` delegate is to perform the work that coordinates the other role instances.</span></span> <span data-ttu-id="b17dc-180">リースを取得しなかった場合は、リーダーとして別のロール インスタンスが選択されており、現在のロール インスタンスは下位のままになります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-180">If the lease isn't acquired, another role instance has been elected as the leader and the current role instance remains a subordinate.</span></span> <span data-ttu-id="b17dc-181">`TryAcquireLeaseOrWait` メソッドは、リースを取得する `BlobLeaseManager` オブジェクトを使用しているヘルパー メソッドであることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b17dc-181">Note that the `TryAcquireLeaseOrWait` method is a helper method that uses the `BlobLeaseManager` object to acquire the lease.</span></span>

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

<span data-ttu-id="b17dc-182">リーダーによって開始されたタスクは、非同期で実行されます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-182">The task started by the leader also runs asynchronously.</span></span> <span data-ttu-id="b17dc-183">このタスクの実行中、次のサンプル コードに示された `RunTaskWhenBlobLeaseAquired` メソッドは、定期的にリースの更新を試行しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-183">While this task is running, the `RunTaskWhenBlobLeaseAquired` method shown in the following code sample periodically attempts to renew the lease.</span></span> <span data-ttu-id="b17dc-184">これにより、ロール インスタンスが確実にリーダーのままでいられます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-184">This helps to ensure that the role instance remains the leader.</span></span> <span data-ttu-id="b17dc-185">サンプル ソリューションでは、他のロール インスタンスがリーダーに選定されないように、更新要求同士の間に生じる遅延は、リースの期間として指定された時間よりも短くなります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-185">In the sample solution, the delay between renewal requests is less than the time specified for the duration of the lease in order to prevent another role instance from being elected the leader.</span></span> <span data-ttu-id="b17dc-186">何らかの理由で更新に失敗した場合、タスクはキャンセルされます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-186">If the renewal fails for any reason, the task is canceled.</span></span>

<span data-ttu-id="b17dc-187">リースの更新に失敗した場合、またはタスクがキャンセルされた場合 (たとえば、ロール インスタンスがシャット ダウンされた結果として)、リースは解放されます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-187">If the lease fails to be renewed or the task is canceled (possibly as a result of the role instance shutting down), the lease is released.</span></span> <span data-ttu-id="b17dc-188">この時点で、このロール インスタンスまたは別のロール インスタンスが、リーダーとして選定される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-188">At this point, this or another role instance might be elected as the leader.</span></span> <span data-ttu-id="b17dc-189">以下のコードの抜粋は、プロセスのこの部分を示しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-189">The code extract below shows this part of the process.</span></span>

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

<span data-ttu-id="b17dc-190">`KeepRenewingLease` メソッドは、リースを更新するために `BlobLeaseManager` オブジェクトを使用するもう 1 つのヘルパー メソッドです。</span><span class="sxs-lookup"><span data-stu-id="b17dc-190">The `KeepRenewingLease` method is another helper method that uses the `BlobLeaseManager` object to renew the lease.</span></span> <span data-ttu-id="b17dc-191">`CancelAllWhenAnyCompletes` メソッドは、最初の 2 つのパラメーターに指定されたタスクをキャンセルします。</span><span class="sxs-lookup"><span data-stu-id="b17dc-191">The `CancelAllWhenAnyCompletes` method cancels the tasks specified as the first two parameters.</span></span> <span data-ttu-id="b17dc-192">次の図では、リーダーを選定して操作を調整するタスクを実行するために、`BlobDistributedMutex` クラスを使用しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-192">The following diagram illustrates using the `BlobDistributedMutex` class to elect a leader and run a task that coordinates operations.</span></span>

![図 1 は、BlobDistributedMutex クラスの関数を示しています](./_images/leader-election-diagram.png)

<span data-ttu-id="b17dc-194">次のコード例は、worker ロールで `BlobDistributedMutex` クラスを使用する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-194">The following code example shows how to use the `BlobDistributedMutex` class in a worker role.</span></span> <span data-ttu-id="b17dc-195">このコードでは、開発ストレージのリースのコンテナーにある `MyLeaderCoordinatorTask` という名前の Blob 経由でリースを取得し、ロール インスタンスがリーダーに選定された場合に、`MyLeaderCoordinatorTask` メソッドに定義されたコードが実行されるように指定しています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-195">This code acquires a lease over a blob named `MyLeaderCoordinatorTask` in the lease's container in development storage, and specifies that the code defined in the `MyLeaderCoordinatorTask` method should run if the role instance is elected the leader.</span></span>

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

<span data-ttu-id="b17dc-196">サンプル ソリューションでは、次の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="b17dc-196">Note the following points about the sample solution:</span></span>

- <span data-ttu-id="b17dc-197">Blob は潜在的な単一障害点になります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-197">The blob is a potential single point of failure.</span></span> <span data-ttu-id="b17dc-198">Blob サービスが利用できない、またはアクセスできない場合、リーダーはリースを更新できず、他のロール インスタンスもリースを取得できません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-198">If the blob service becomes unavailable, or is inaccessible, the leader won't be able to renew the lease and no other role instance will be able to acquire the lease.</span></span> <span data-ttu-id="b17dc-199">この場合、どのロール インスタンスもリーダーとして機能できません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-199">In this case, no role instance will be able to act as the leader.</span></span> <span data-ttu-id="b17dc-200">しかし、Blob サービスは復元性を備えた設計になっているため、Blob サービスの致命的な障害は極めて発生しにくいと考えられています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-200">However, the blob service is designed to be resilient, so complete failure of the blob service is considered to be extremely unlikely.</span></span>
- <span data-ttu-id="b17dc-201">リーダーが実行しているタスクが停止した場合、リーダーは引き続きリースの更新を行って、他のロール インスタンスによるリースの取得を防止し、リースのロールを引き継いでタスクを調整します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-201">If the task being performed by the leader stalls, the leader might continue to renew the lease, preventing any other role instance from acquiring the lease and taking over the leader role in order to coordinate tasks.</span></span> <span data-ttu-id="b17dc-202">実際の運用では、リーダーの正常性を頻繁にチェックする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-202">In the real world, the health of the leader should be checked at frequent intervals.</span></span>
- <span data-ttu-id="b17dc-203">選定プロセスは非決定性です。</span><span class="sxs-lookup"><span data-stu-id="b17dc-203">The election process is nondeterministic.</span></span> <span data-ttu-id="b17dc-204">どのロール インスタンスが Blob リースを取得してリーダーになるか予測することはできません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-204">You can't make any assumptions about which role instance will acquire the blob lease and become the leader.</span></span>
- <span data-ttu-id="b17dc-205">Blob リースのターゲットとして使用される Blob は、他の目的には使用できません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-205">The blob used as the target of the blob lease shouldn't be used for any other purpose.</span></span> <span data-ttu-id="b17dc-206">ロール インスタンスがこの Blob へのデータの格納を試行した場合、そのロール インスタンスがリーダーとなり Blob リースを保持していないかぎり、このデータにはアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="b17dc-206">If a role instance attempts to store data in this blob, this data won't be accessible unless the role instance is the leader and holds the blob lease.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="b17dc-207">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="b17dc-207">Related patterns and guidance</span></span>

<span data-ttu-id="b17dc-208">このパターンを実装する場合は、次のガイダンスも関連している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b17dc-208">The following guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="b17dc-209">このパターンには、ダウンロード可能な[サンプル アプリケーション](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)が用意されています。</span><span class="sxs-lookup"><span data-stu-id="b17dc-209">This pattern has a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election).</span></span>
- <span data-ttu-id="b17dc-210">[自動スケール ガイダンス](https://msdn.microsoft.com/library/dn589774.aspx)。</span><span class="sxs-lookup"><span data-stu-id="b17dc-210">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="b17dc-211">アプリケーションの負荷は変化するため、タスク ホストのインスタンスを開始および停止することが可能です。</span><span class="sxs-lookup"><span data-stu-id="b17dc-211">It's possible to start and stop instances of the task hosts as the load on the application varies.</span></span> <span data-ttu-id="b17dc-212">自動スケーリングは、ピーク時の処理中のスループットとパフォーマンスの維持に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="b17dc-212">Autoscaling can help to maintain throughput and performance during times of peak processing.</span></span>
- <span data-ttu-id="b17dc-213">[計算分割ガイダンス](https://msdn.microsoft.com/library/dn589773.aspx)。</span><span class="sxs-lookup"><span data-stu-id="b17dc-213">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="b17dc-214">このガイダンスでは、サービスのスケーラビリティ、パフォーマンス、可用性、およびセキュリティを維持しながら実行中のコストを最小限に抑える 1 つの方法として、クラウド サービスのホストにタスクを割り当てる方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="b17dc-214">This guidance describes how to allocate tasks to hosts in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>
- <span data-ttu-id="b17dc-215">[タスク ベースの非同期パターン](https://msdn.microsoft.com/library/hh873175.aspx)。</span><span class="sxs-lookup"><span data-stu-id="b17dc-215">The [Task-based Asynchronous Pattern](https://msdn.microsoft.com/library/hh873175.aspx).</span></span>
- <span data-ttu-id="b17dc-216">[Bully Algorithm (ブリー アルゴリズム) ](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)を示したサンプル。</span><span class="sxs-lookup"><span data-stu-id="b17dc-216">An example illustrating the [Bully Algorithm](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).</span></span>
- <span data-ttu-id="b17dc-217">[Ring Algorithm (リング アルゴリズム) ](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)を示したサンプル。</span><span class="sxs-lookup"><span data-stu-id="b17dc-217">An example illustrating the [Ring Algorithm](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span>
- <span data-ttu-id="b17dc-218">Apache ZooKeeper の [Apache Curator](https://curator.apache.org/) クライアント ライブラリ。</span><span class="sxs-lookup"><span data-stu-id="b17dc-218">[Apache Curator](https://curator.apache.org/) a client library for Apache ZooKeeper.</span></span>
- <span data-ttu-id="b17dc-219">MSDN の記事「[Lease Blob (REST API) (Blob のリース (REST API))](https://msdn.microsoft.com/library/azure/ee691972.aspx)」。</span><span class="sxs-lookup"><span data-stu-id="b17dc-219">The article [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) on MSDN.</span></span>
