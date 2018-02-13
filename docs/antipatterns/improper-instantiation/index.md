---
title: "不適切なインスタンス化のアンチパターン"
description: "作成後に共有する予定のオブジェクトの新しいインスタンスを頻繁に作成することは避けてください。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8955f37e76c8b5e66c1ed7737d200d11ed321612
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/13/2018
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="4954d-103">不適切なインスタンス化のアンチパターン</span><span class="sxs-lookup"><span data-stu-id="4954d-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="4954d-104">作成後に共有する予定のオブジェクトの新しいインスタンスを頻繁に作成すると、パフォーマンスが低下する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="4954d-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="4954d-105">Problem description</span></span>

<span data-ttu-id="4954d-106">多くのライブラリでは、外部リソースの抽象化が提供されます。</span><span class="sxs-lookup"><span data-stu-id="4954d-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="4954d-107">内部的には、これらのクラスは、通常、リソースへの独自の接続を管理して、クライアントがリソースへのアクセスに使用できるブローカーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="4954d-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="4954d-108">Azure アプリケーションに関連するブローカー クラスのいくつかの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="4954d-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="4954d-109">`System.Net.Http.HttpClient`</span><span class="sxs-lookup"><span data-stu-id="4954d-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="4954d-110">HTTP を使用して Web サービスと通信します。</span><span class="sxs-lookup"><span data-stu-id="4954d-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="4954d-111">`Microsoft.ServiceBus.Messaging.QueueClient`</span><span class="sxs-lookup"><span data-stu-id="4954d-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="4954d-112">Service Bus キューとの間でメッセージを送受信します。</span><span class="sxs-lookup"><span data-stu-id="4954d-112">Posts and receives messages to a Service Bus queue.</span></span> 
- <span data-ttu-id="4954d-113">`Microsoft.Azure.Documents.Client.DocumentClient`</span><span class="sxs-lookup"><span data-stu-id="4954d-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="4954d-114">Cosmos DB インスタンスに接続します</span><span class="sxs-lookup"><span data-stu-id="4954d-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="4954d-115">`StackExchange.Redis.ConnectionMultiplexer`</span><span class="sxs-lookup"><span data-stu-id="4954d-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="4954d-116">Azure Redis Cache を含む Redis に接続します。</span><span class="sxs-lookup"><span data-stu-id="4954d-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="4954d-117">これらのクラスは、一度インスタンス化された後、アプリケーションの有効期間にわたって再利用されることが意図されています。</span><span class="sxs-lookup"><span data-stu-id="4954d-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="4954d-118">ただし、"これらのクラスは必要なときにのみ取得し、すぐに解放する必要がある" という考えはよくある誤解です </span><span class="sxs-lookup"><span data-stu-id="4954d-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="4954d-119">(ここに示されているのは .NET ライブラリですが、そのパターンは .NET に固有のものではありません)。</span><span class="sxs-lookup"><span data-stu-id="4954d-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.)</span></span>

<span data-ttu-id="4954d-120">次の ASP.NET の例では、リモート サービスと通信するために `HttpClient` のインスタンスを作成しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-120">The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="4954d-121">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="4954d-121">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="4954d-122">Web アプリケーションでは、この手法はスケーラブルではありません。</span><span class="sxs-lookup"><span data-stu-id="4954d-122">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="4954d-123">それぞれのユーザー要求に対して新しい `HttpClient` オブジェクトが作成されます。</span><span class="sxs-lookup"><span data-stu-id="4954d-123">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="4954d-124">負荷が大きい場合、Web サーバーが使用可能な数のソケットを使い果たした結果、`SocketException` エラーが発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-124">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="4954d-125">この問題は `HttpClient` クラスに限定されません。</span><span class="sxs-lookup"><span data-stu-id="4954d-125">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="4954d-126">同様の問題は、リソースをラップする他のクラスや作成するのが高価な他のクラスでも発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-126">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="4954d-127">次の例では、`ExpensiveToCreateService` クラスのインスタンスを作成しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-127">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="4954d-128">ここでの問題は必ずしもソケットの枯渇ではなく、各インスタンスの作成に要する時間です。</span><span class="sxs-lookup"><span data-stu-id="4954d-128">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="4954d-129">このクラスのインスタンスの作成と破棄を頻繁に繰り返すと、システムのスケーラビリティに悪影響が及ぶ可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-129">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="4954d-130">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="4954d-130">How to fix the problem</span></span>

<span data-ttu-id="4954d-131">外部リソースをラップするクラスが共有可能かつスレッドセーフである場合は、クラスの共有シングルトン インスタンスまたは再利用可能なインスタンスのプールを作成します。</span><span class="sxs-lookup"><span data-stu-id="4954d-131">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="4954d-132">次の例では、静的な `HttpClient` インスタンスを使用しているため、すべての要求で接続を共有しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-132">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient HttpClient;

    static SingleHttpClientInstanceController()
    {
        HttpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await HttpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="4954d-133">考慮事項</span><span class="sxs-lookup"><span data-stu-id="4954d-133">Considerations</span></span>

- <span data-ttu-id="4954d-134">このアンチパターンの重要な要素は、"*共有可能*" オブジェクトのインスタンスの作成と破棄を繰り返していることです。</span><span class="sxs-lookup"><span data-stu-id="4954d-134">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="4954d-135">クラスが共有可能でない (スレッドセーフでない) 場合、このアンチパターンは当てはまりません。</span><span class="sxs-lookup"><span data-stu-id="4954d-135">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="4954d-136">共有リソースの種類によって、シングルトンを使用すべきか、プールを作成すべきかが決まります。</span><span class="sxs-lookup"><span data-stu-id="4954d-136">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="4954d-137">`HttpClient` クラスは、プールされるのではなく共有されるように設計されています。</span><span class="sxs-lookup"><span data-stu-id="4954d-137">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="4954d-138">他のオブジェクトでは、プーリングをサポートして、システムによる複数のインスタンスへのワークロードの分散を可能にすることができます。</span><span class="sxs-lookup"><span data-stu-id="4954d-138">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="4954d-139">複数の要求で共有されるオブジェクトは、スレッドセーフである "*必要があります*"。</span><span class="sxs-lookup"><span data-stu-id="4954d-139">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="4954d-140">`HttpClient` クラスはこのように使用されるように設計されています。しかし、他のクラスは同時要求をサポートしない可能性があるため、参照可能なドキュメントを確認してください。</span><span class="sxs-lookup"><span data-stu-id="4954d-140">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="4954d-141">リソースの種類によっては十分でないものもあり、このようなリソースは保持すべきではありません。</span><span class="sxs-lookup"><span data-stu-id="4954d-141">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="4954d-142">データベース接続はその一例です。</span><span class="sxs-lookup"><span data-stu-id="4954d-142">Database connections are an example.</span></span> <span data-ttu-id="4954d-143">開かれたデータベース接続を必要でないのに保持すると、他の同時ユーザーがデータベースにアクセスできなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-143">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="4954d-144">.NET Framework では、外部リソースへの接続を確立する多くのオブジェクトは、これらの接続を管理する他のクラスの静的ファクトリ メソッドを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="4954d-144">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="4954d-145">これらのファクトリ オブジェクトは、廃棄した後で再び作成するのではなく、保存して再利用することが意図されています。</span><span class="sxs-lookup"><span data-stu-id="4954d-145">These factories  objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="4954d-146">たとえば、Azure Service Bus では、`QueueClient` オブジェクトは `MessagingFactory` オブジェクトを通じて作成されます。</span><span class="sxs-lookup"><span data-stu-id="4954d-146">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="4954d-147">内部的には、`MessagingFactory` が接続を管理します。</span><span class="sxs-lookup"><span data-stu-id="4954d-147">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="4954d-148">詳細については、「[Service Bus メッセージングを使用したパフォーマンス向上のためのベスト プラクティス][service-bus-messaging]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="4954d-148">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="4954d-149">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="4954d-149">How to detect the problem</span></span>

<span data-ttu-id="4954d-150">この問題の現象としては、次の 1 つまたは複数の現象に加えて、スループットの低下またはエラー率の増加が挙げられます。</span><span class="sxs-lookup"><span data-stu-id="4954d-150">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span> 

- <span data-ttu-id="4954d-151">ソケット、データベース接続、ファイル ハンドルなどのリソースの枯渇を示す例外の増加。</span><span class="sxs-lookup"><span data-stu-id="4954d-151">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span> 
- <span data-ttu-id="4954d-152">メモリ使用量とガベージ コレクションの増加。</span><span class="sxs-lookup"><span data-stu-id="4954d-152">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="4954d-153">ネットワーク、ディスク、またはデータベース アクティビティの増加。</span><span class="sxs-lookup"><span data-stu-id="4954d-153">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="4954d-154">この問題の識別に役立てるために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="4954d-154">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="4954d-155">運用システムのプロセス監視を実行して、応答速度が低下したポイントや、リソース不足が原因でシステムにエラーが発生したポイントを識別します。</span><span class="sxs-lookup"><span data-stu-id="4954d-155">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="4954d-156">これらのポイントでキャプチャされたテレメトリ データを調べて、リソースを大量に消費するオブジェクトを作成および破棄している可能性のある操作を判定します。</span><span class="sxs-lookup"><span data-stu-id="4954d-156">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="4954d-157">運用システムではなく、管理されたテスト環境で、疑わしい操作のそれぞれに対してロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="4954d-157">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="4954d-158">ソース コードを調べて、ブローカー オブジェクトの管理方法を確認します。</span><span class="sxs-lookup"><span data-stu-id="4954d-158">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="4954d-159">スタック トレースを調べて、システムに負荷がかかっているときに実行が低速になる操作や例外を生成する操作を確認します。</span><span class="sxs-lookup"><span data-stu-id="4954d-159">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="4954d-160">この情報は、これらの操作がどのようにリソースを利用しているかを識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4954d-160">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="4954d-161">例外は、共用リソースの枯渇が原因でエラーが発生しているかどうかを判断するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4954d-161">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span> 

## <a name="example-diagnosis"></a><span data-ttu-id="4954d-162">診断の例</span><span class="sxs-lookup"><span data-stu-id="4954d-162">Example diagnosis</span></span>

<span data-ttu-id="4954d-163">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。</span><span class="sxs-lookup"><span data-stu-id="4954d-163">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="4954d-164">速度低下またはエラーのポイントを識別する</span><span class="sxs-lookup"><span data-stu-id="4954d-164">Identify points of slow down or failure</span></span>

<span data-ttu-id="4954d-165">次の画像は、[New Relic APM][new-relic] を使用して生成された結果を示しています。応答時間が低速な操作が示されています。</span><span class="sxs-lookup"><span data-stu-id="4954d-165">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="4954d-166">この場合、`NewHttpClientInstancePerRequest` コントローラーの `GetProductAsync` メソッドをさらに調査する価値があります。</span><span class="sxs-lookup"><span data-stu-id="4954d-166">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="4954d-167">これらの操作が実行されているときにエラー率も増加していることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="4954d-167">Notice that the error rate also increases when these operations are running.</span></span> 

![New Relic モニター ダッシュボードに表示された、要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーション][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="4954d-169">テレメトリ データを調べて相関関係を見つける</span><span class="sxs-lookup"><span data-stu-id="4954d-169">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="4954d-170">次の画像は、前の画像に対応する同じ期間にスレッド プロファイリングを使用してキャプチャされたデータを示しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-170">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="4954d-171">システムは、ソケット接続を開くためにかなりの時間を費やしているだけでなく、ソケット接続を閉じたりソケット例外を処理したりするためにそれ以上の時間を費やしています。</span><span class="sxs-lookup"><span data-stu-id="4954d-171">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![New Relic スレッド プロファイラーに表示された、要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーション][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="4954d-173">ロード テストを実行する</span><span class="sxs-lookup"><span data-stu-id="4954d-173">Performing load testing</span></span>

<span data-ttu-id="4954d-174">ロード テストを使用して、ユーザーが実行する典型的な操作をシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="4954d-174">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="4954d-175">これは、変化する負荷に対し、システムのどの部分でリソースが枯渇するかを識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4954d-175">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="4954d-176">これらのテストは、運用システムではなく管理された環境で実行します。</span><span class="sxs-lookup"><span data-stu-id="4954d-176">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="4954d-177">次のグラフは、ユーザー負荷が 100 同時ユーザーに増えたときに `NewHttpClientInstancePerRequest` コントローラーによって処理される要求のスループットを示しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-177">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーションのスループット][throughput-new-HTTPClient-instance]

<span data-ttu-id="4954d-179">最初は、ワークロードが増加するにつれて、1 秒あたりに処理される要求の量が増えます。</span><span class="sxs-lookup"><span data-stu-id="4954d-179">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="4954d-180">しかし、ユーザー数が約 30 人になると、成功する要求の量が限度に達し、システムが例外を生成し始めます。</span><span class="sxs-lookup"><span data-stu-id="4954d-180">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="4954d-181">それ以後、ユーザーの負荷に応じて例外の量が徐々に増えます。</span><span class="sxs-lookup"><span data-stu-id="4954d-181">From then on, the volume of exceptions gradually increases with the user load.</span></span> 

<span data-ttu-id="4954d-182">ロード テストでは、これらのエラーが HTTP 500 (内部サーバー) エラーとして報告されています。</span><span class="sxs-lookup"><span data-stu-id="4954d-182">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="4954d-183">テレメトリを調べると、これらのエラーの原因は、`HttpClient` オブジェクトが作成され続けた結果、システムのソケット リソースが不足したことにあることがわかります。</span><span class="sxs-lookup"><span data-stu-id="4954d-183">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="4954d-184">次のグラフは、カスタム `ExpensiveToCreateService` オブジェクトを作成するコントローラーに対する同様のテストを示しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-184">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![要求ごとに ExpensiveToCreateService の新しいインスタンスが作成されるサンプル アプリケーションのスループット][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="4954d-186">今回、コントローラーで例外は生成されません。しかし、平均応答時間が 20 倍になっている一方でスループットはまだ横ばい状態になったままです </span><span class="sxs-lookup"><span data-stu-id="4954d-186">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="4954d-187">(グラフでは、応答時間とスループットに対数スケールが使用されています)。テレメトリから、`ExpensiveToCreateService` の新しいインスタンスを作成することが問題の主な原因であることがわかります。</span><span class="sxs-lookup"><span data-stu-id="4954d-187">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="4954d-188">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="4954d-188">Implement the solution and verify the result</span></span>

<span data-ttu-id="4954d-189">1 つの `HttpClient` インスタンスを共有するように `GetProductAsync` メソッドを切り替えたところ、2 番目のロード テストでパフォーマンスの向上が見られました。</span><span class="sxs-lookup"><span data-stu-id="4954d-189">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="4954d-190">エラーは報告されておらず、システムは 1 秒あたり最大 500 要求の高い負荷を処理することができました。</span><span class="sxs-lookup"><span data-stu-id="4954d-190">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="4954d-191">平均応答時間は、以前のテストと比較して半分に短縮されました。</span><span class="sxs-lookup"><span data-stu-id="4954d-191">The average response time was cut in half, compared with the previous test.</span></span>

![要求ごとに HttpClient オブジェクトの同じインスタンスが再利用されるサンプル アプリケーションのスループット][throughput-single-HTTPClient-instance]

<span data-ttu-id="4954d-193">比較のために、スタック トレース テレメトリを示す次の画像をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="4954d-193">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="4954d-194">今回、システムのほとんどの時間は、ソケットを開く操作と閉じる操作ではなく、実際の作業に費やされています。</span><span class="sxs-lookup"><span data-stu-id="4954d-194">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![New Relic スレッド プロファイラーに表示された、すべての要求に対して HttpClient オブジェクトの単一のインスタンスが作成されるサンプル アプリケーション][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="4954d-196">次のグラフは、`ExpensiveToCreateService` オブジェクトの共有インスタンスを使用した場合の同様のロード テストを示しています。</span><span class="sxs-lookup"><span data-stu-id="4954d-196">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="4954d-197">この場合も、処理された要求の量はユーザーの負荷に応じて増加していますが、平均応答時間は低いままです。</span><span class="sxs-lookup"><span data-stu-id="4954d-197">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span> 

![要求ごとに HttpClient オブジェクトの同じインスタンスが再利用されるサンプル アプリケーションのスループット][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
