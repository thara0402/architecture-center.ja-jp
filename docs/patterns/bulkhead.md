---
title: バルクヘッド パターン
titleSuffix: Cloud Design Patterns
description: アプリケーションの要素をプールに分離し、1 つの要素が失敗しても、他の要素が引き続き機能できるようにします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4ad221ae5e9bd51c6d304ba33b884f71c5081b16
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898257"
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="f7319-104">バルクヘッド パターン</span><span class="sxs-lookup"><span data-stu-id="f7319-104">Bulkhead pattern</span></span>

<span data-ttu-id="f7319-105">アプリケーションの要素をプールに分離し、1 つの要素が失敗しても、他の要素が引き続き機能できるようにします。</span><span class="sxs-lookup"><span data-stu-id="f7319-105">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="f7319-106">このパターンは*バルクヘッド* (隔壁) という名前ですが、それは区分けされた船体部分に似ているためです。</span><span class="sxs-lookup"><span data-stu-id="f7319-106">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="f7319-107">船体が傷つけられた場合、水浸しになるのは破損した部分だけで、これによって船が沈むのを防ぎます。</span><span class="sxs-lookup"><span data-stu-id="f7319-107">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="f7319-108">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="f7319-108">Context and problem</span></span>

<span data-ttu-id="f7319-109">クラウド ベースのアプリケーションには複数のサービスが含まれ、各々のサービスは 1 つ以上のコンシューマーを持ちます。</span><span class="sxs-lookup"><span data-stu-id="f7319-109">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="f7319-110">サービスで過度の負荷やエラーがあると、そのサービスのすべてのコンシューマーに影響します。</span><span class="sxs-lookup"><span data-stu-id="f7319-110">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="f7319-111">さらに、コンシューマーは複数のサービスに同時に要求を送信し、要求ごとにリソースを使用しています。</span><span class="sxs-lookup"><span data-stu-id="f7319-111">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="f7319-112">コンシューマーが、正しく構成されていないまたは応答していないサービスに要求を送信すると、クライアントの要求で使用されるリソースが適切なタイミングで解放されない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="f7319-112">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="f7319-113">このサービスへの要求が続くと、これらのリソースが不足する場合があります。</span><span class="sxs-lookup"><span data-stu-id="f7319-113">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="f7319-114">たとえば、クライアントの接続プールを使い果たすことがあります。</span><span class="sxs-lookup"><span data-stu-id="f7319-114">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="f7319-115">その時点で、このコンシューマーによる他のサービスへの要求が影響を受けます。</span><span class="sxs-lookup"><span data-stu-id="f7319-115">At that point, requests by the consumer to other services are affected.</span></span> <span data-ttu-id="f7319-116">最終的にコンシューマーは、元々応答していないサービスだけでなく、他のサービスにも要求を送信できなくなります。</span><span class="sxs-lookup"><span data-stu-id="f7319-116">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="f7319-117">同様のリソース不足の問題が、複数のコンシューマーを伴うサービスに影響します。</span><span class="sxs-lookup"><span data-stu-id="f7319-117">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="f7319-118">1 つのクライアントから送信された大量の要求が、サービスで使用可能なリソースを使い果たす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="f7319-118">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="f7319-119">他のコンシューマーがサービスを利用できなくなり、エラーが積み重なるという結果になります。</span><span class="sxs-lookup"><span data-stu-id="f7319-119">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="f7319-120">解決策</span><span class="sxs-lookup"><span data-stu-id="f7319-120">Solution</span></span>

<span data-ttu-id="f7319-121">コンシューマーの負荷と可用性の要件に基づいて、サービス インスタンスを別々のグループに分割します。</span><span class="sxs-lookup"><span data-stu-id="f7319-121">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="f7319-122">この設計は障害を特定するのに役立ち、障害中でも一部のコンシューマーに対してサービス機能を維持することができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-122">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="f7319-123">コンシューマーはまた、リソースを分割し、1 つのサービスの呼び出しに使用されるリソースが別のサービスの呼び出しに使用されるリソースに影響しないようにできます。</span><span class="sxs-lookup"><span data-stu-id="f7319-123">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="f7319-124">たとえば、複数のサービスを呼び出すコンシューマーに、各々のサービス用の接続プールを割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-124">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="f7319-125">サービスが失敗し始めると、そのサービスに割り当てられている接続プールのみに影響し、コンシューマーは他のサービスを引き続き使用できます。</span><span class="sxs-lookup"><span data-stu-id="f7319-125">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="f7319-126">このパターンには次のような利点があります。</span><span class="sxs-lookup"><span data-stu-id="f7319-126">The benefits of this pattern include:</span></span>

- <span data-ttu-id="f7319-127">コンシューマーとサービスを、障害の連鎖から分離します。</span><span class="sxs-lookup"><span data-stu-id="f7319-127">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="f7319-128">コンシューマーまたはサービスに影響する問題は、自身のバルクヘッド内に分離でき、ソリューション全体に障害が発生するのを妨ぎます。</span><span class="sxs-lookup"><span data-stu-id="f7319-128">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="f7319-129">サービス エラーが発生した場合、一部の機能を保持できます。</span><span class="sxs-lookup"><span data-stu-id="f7319-129">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="f7319-130">アプリケーションのその他のサービスと機能は、引き続き機能します。</span><span class="sxs-lookup"><span data-stu-id="f7319-130">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="f7319-131">使用するアプリケーションに別の品質のサービスを提供するサービスを展開できます。</span><span class="sxs-lookup"><span data-stu-id="f7319-131">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="f7319-132">優先度の高いサービスを使用するように、優先度の高いコンシューマー プールを構成することができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-132">A high-priority consumer pool can be configured to use high-priority services.</span></span>

<span data-ttu-id="f7319-133">次の図は、個々 のサービスを呼び出す接続プールを中心に構造化したバルクヘッドを示しています。</span><span class="sxs-lookup"><span data-stu-id="f7319-133">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="f7319-134">サービス A で障害や他の問題が発生すると、接続プールが分離され、サービス A に割り当てられているスレッド プールを使用するワークロードだけが影響を受けます。</span><span class="sxs-lookup"><span data-stu-id="f7319-134">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="f7319-135">サービス B および C を使用するワークロードは影響を受けず、中断することなく機能し続けることができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-135">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![バルクヘッド パターンの最初の図](./_images/bulkhead-1.png)

<span data-ttu-id="f7319-137">次の図は、1 つのサービスを呼び出す複数のクライアントを示しています。</span><span class="sxs-lookup"><span data-stu-id="f7319-137">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="f7319-138">各クライアントには、別々のサービス インスタンスが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="f7319-138">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="f7319-139">クライアント 1 の要求が多すぎて、そのインスタンスを圧迫しています。</span><span class="sxs-lookup"><span data-stu-id="f7319-139">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="f7319-140">各サービス インスタンスは他から分離されているため、他のクライアントは呼び出しを続行することができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-140">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![バルクヘッド パターンの最初の図](./_images/bulkhead-2.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="f7319-142">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="f7319-142">Issues and considerations</span></span>

- <span data-ttu-id="f7319-143">アプリケーションのビジネス要件と技術的要件を中心にパーティションを定義します。</span><span class="sxs-lookup"><span data-stu-id="f7319-143">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="f7319-144">サービスまたはコンシューマーをバルクヘッドに分割するときは、コスト、パフォーマンス、管理容易性の観点からのオーバーヘッドと、テクノロジによる分離レベルを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="f7319-144">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="f7319-145">より高度なエラー処理を提供するために、バルクヘッドを再試行、サーキット ブレーカー、調整パターンと組み合わせることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="f7319-145">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="f7319-146">コンシューマーをバルクヘッドに分割する場合は、プロセス、スレッド プール、およびセマフォの使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="f7319-146">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="f7319-147">[Netflix Hystrix][hystrix] および [Polly][polly] のようなプロジェクトは、コンシューマー バルクヘッドを作成するためのフレームワークを提供しています。</span><span class="sxs-lookup"><span data-stu-id="f7319-147">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="f7319-148">サービスをバルクヘッドに分割する場合は、別々 の仮想マシン、コンテナー、またはプロセスに展開することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="f7319-148">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="f7319-149">コンテナーは、かなり少ないオーバーヘッドで、バランスのとれたリソース分離を提供します。</span><span class="sxs-lookup"><span data-stu-id="f7319-149">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="f7319-150">非同期メッセージを使用して通信するサービスは、異なるセットのキューを介して分離することができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-150">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="f7319-151">各キューは、そのキュー上のメッセージを処理するインスタンスの専用セット、または、デキューして処理をディスパッチするアルゴリズムを使用するインスタンスの単一グループを持つことができます。</span><span class="sxs-lookup"><span data-stu-id="f7319-151">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="f7319-152">バルクヘッドの細分性のレベルを決定します。</span><span class="sxs-lookup"><span data-stu-id="f7319-152">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="f7319-153">たとえば、テナントをパーティションに分散する場合は、各テナントを別のパーティションに配置するか、1 つのパーティションに複数のテナントを置きます。</span><span class="sxs-lookup"><span data-stu-id="f7319-153">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, or put several tenants into one partition.</span></span>
- <span data-ttu-id="f7319-154">各パーティションのパフォーマンスと SLA を監視します。</span><span class="sxs-lookup"><span data-stu-id="f7319-154">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="f7319-155">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="f7319-155">When to use this pattern</span></span>

<span data-ttu-id="f7319-156">このパターンは次の目的で使用します。</span><span class="sxs-lookup"><span data-stu-id="f7319-156">Use this pattern to:</span></span>

- <span data-ttu-id="f7319-157">バックエンド サービスのセットを使用するのに使われているリソースを分離する。特に、サービスのいずれかが応答していない場合でも、アプリケーションがなんらかのレベルの機能を提供できる場合。</span><span class="sxs-lookup"><span data-stu-id="f7319-157">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="f7319-158">標準的なコンシューマーから重要なコンシューマーを分離する。</span><span class="sxs-lookup"><span data-stu-id="f7319-158">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="f7319-159">エラーの連鎖からアプリケーションを保護する。</span><span class="sxs-lookup"><span data-stu-id="f7319-159">Protect the application from cascading failures.</span></span>

<span data-ttu-id="f7319-160">このパターンは、次の状況では適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="f7319-160">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="f7319-161">プロジェクト内で、効率性の低いリソース使用は受け入れられない。</span><span class="sxs-lookup"><span data-stu-id="f7319-161">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="f7319-162">複雑さを追加する必要はない。</span><span class="sxs-lookup"><span data-stu-id="f7319-162">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="f7319-163">例</span><span class="sxs-lookup"><span data-stu-id="f7319-163">Example</span></span>

<span data-ttu-id="f7319-164">次の Kubernetes 構成ファイルでは、独自の CPU とメモリ リソースと制限で 1 つのサービスを実行する、分離されたコンテナーを作成します。</span><span class="sxs-lookup"><span data-stu-id="f7319-164">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a><span data-ttu-id="f7319-165">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="f7319-165">Related guidance</span></span>

- [<span data-ttu-id="f7319-166">回復性に優れた Azure 用アプリケーションの設計</span><span class="sxs-lookup"><span data-stu-id="f7319-166">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="f7319-167">サーキット ブレーカー パターン</span><span class="sxs-lookup"><span data-stu-id="f7319-167">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="f7319-168">再試行パターン</span><span class="sxs-lookup"><span data-stu-id="f7319-168">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="f7319-169">調整パターン</span><span class="sxs-lookup"><span data-stu-id="f7319-169">Throttling pattern</span></span>](./throttling.md)

<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly
