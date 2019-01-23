---
title: ビジー状態のフロントエンドのアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: 多数のバックグラウンド スレッドで非同期処理が実行されることによって、フォアグラウンドで実行される他のタスクのリソースが逼迫する場合があります。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 61470b630f735c1d49ad9b4bfbec853b308630cf
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481526"
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="26c55-103">ビジー状態のフロントエンドのアンチパターン</span><span class="sxs-lookup"><span data-stu-id="26c55-103">Busy Front End antipattern</span></span>

<span data-ttu-id="26c55-104">多数のバックグラウンド スレッドで非同期処理が実行されることによって、フォアグラウンドで実行される他の同時実行タスクのリソースが逼迫し、容認できないレベルにまで応答時間が悪化する場合があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="26c55-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="26c55-105">Problem description</span></span>

<span data-ttu-id="26c55-106">リソースを集中的に使用するタスクは、ユーザーの要求に対する応答時間の増加を招き、長い待ち時間が発生する原因となる場合があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="26c55-107">応答時間を改善する 1 つの方法としては、リソースを集中的に使用するタスクの負荷を別のスレッドに移すことが考えられます。</span><span class="sxs-lookup"><span data-stu-id="26c55-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="26c55-108">この方法により、バックグラウンドで処理が実行されている間も、アプリケーションの応答性を確保することができます。</span><span class="sxs-lookup"><span data-stu-id="26c55-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="26c55-109">しかしバックグラウンド スレッドで実行されるタスクは依然としてリソースを消費します。</span><span class="sxs-lookup"><span data-stu-id="26c55-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="26c55-110">それらが多すぎると、要求を処理しているスレッドがリソース不足に陥る可能性があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="26c55-111">"*リソース*" は意味の広い言葉であり、CPU 使用率、メモリ占有率、ネットワーク I/O、ディスク I/O などさまざまな事柄が含まれます。</span><span class="sxs-lookup"><span data-stu-id="26c55-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="26c55-112">この問題が起こる典型的な状況は、プレゼンテーション レイヤーとの間で共有される単一の層にすべてのビジネス ロジックを組み込んだモノリシックなコードとしてアプリケーションが開発されているときです。</span><span class="sxs-lookup"><span data-stu-id="26c55-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="26c55-113">以下この問題を ASP.NET を使用した例で説明します。</span><span class="sxs-lookup"><span data-stu-id="26c55-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="26c55-114">完全なサンプルは、[こちら][code-sample]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="26c55-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="26c55-115">`WorkInFrontEnd` コントローラーの `Post` メソッドは、HTTP POST 操作を実装しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="26c55-116">CPU 負荷の高い長時間実行されるタスクをこの操作でシミュレートします。</span><span class="sxs-lookup"><span data-stu-id="26c55-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="26c55-117">POST 操作がすぐに完了できるよう、この処理は独立したスレッドで実行されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="26c55-118">`UserProfile` コントローラーの `Get` メソッドは、HTTP GET 操作を実装しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="26c55-119">このメソッドによって CPU にかかる負荷はさほど高くありません。</span><span class="sxs-lookup"><span data-stu-id="26c55-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="26c55-120">一番気になる点は、`Post` メソッドのリソース要件です。</span><span class="sxs-lookup"><span data-stu-id="26c55-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="26c55-121">この処理はバックグラウンド スレッドに置かれていますが、それなりに CPU リソースが消費されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="26c55-122">これらのリソースは、別のユーザーによって同時実行されている他の操作との間で共有されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="26c55-123">その要求が、ある程度の人数のユーザーから同時に送信された場合、全体のパフォーマンスに影響が及び、すべての操作の処理速度が低下してしまいます。</span><span class="sxs-lookup"><span data-stu-id="26c55-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="26c55-124">たとえばユーザーが、`Get` メソッドで著しい待ち時間を強いられることも考えられます。</span><span class="sxs-lookup"><span data-stu-id="26c55-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="26c55-125">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="26c55-125">How to fix the problem</span></span>

<span data-ttu-id="26c55-126">リソースが著しく消費されるプロセスを、独立したバックエンドに移動します。</span><span class="sxs-lookup"><span data-stu-id="26c55-126">Move processes that consume significant resources to a separate back end.</span></span>

<span data-ttu-id="26c55-127">この方法により、リソースを集中的に使用するフロントエンドのタスクはメッセージ キューに置かれます。</span><span class="sxs-lookup"><span data-stu-id="26c55-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="26c55-128">そのタスクがバックエンドによって取得され、非同期で処理されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="26c55-129">このキューには、要求をバックエンドにバッファー処理することによって負荷を平準化する働きもあります。</span><span class="sxs-lookup"><span data-stu-id="26c55-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="26c55-130">キューの長さが限度を超えた場合は、バックエンドをスケールアウトする自動スケールを構成することができます。</span><span class="sxs-lookup"><span data-stu-id="26c55-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="26c55-131">前述のコードを修正したバージョンを次に示します。</span><span class="sxs-lookup"><span data-stu-id="26c55-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="26c55-132">このバージョンでは、`Post` メソッドによってメッセージが Service Bus キューに追加されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span>

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="26c55-133">バックエンドは、Service Bus キューからメッセージを取得して処理を実行します。</span><span class="sxs-lookup"><span data-stu-id="26c55-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="26c55-134">考慮事項</span><span class="sxs-lookup"><span data-stu-id="26c55-134">Considerations</span></span>

- <span data-ttu-id="26c55-135">この方法によってアプリケーションの複雑さが多少増します。</span><span class="sxs-lookup"><span data-stu-id="26c55-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="26c55-136">万一障害が発生しても要求が失われないよう、キューとデキューを安全に処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="26c55-137">メッセージ キューのためのサービスが別途アプリケーションに必要となります。</span><span class="sxs-lookup"><span data-stu-id="26c55-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="26c55-138">想定されるワークロードを処理し、必要なスループット目標を満たすために、処理環境にはスケーラビリティを十分に確保してください。</span><span class="sxs-lookup"><span data-stu-id="26c55-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="26c55-139">この方法によって全体的な応答性は向上すると考えられますが、バックエンドに移されたタスクは、以前よりも完了までに時間がかかるようになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="26c55-140">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="26c55-140">How to detect the problem</span></span>

<span data-ttu-id="26c55-141">ビジー状態のフロントエンドは、その症状の 1 つとして、リソースを集中的に使用するタスクが実行されていると待ち時間が長くなるということがあります。</span><span class="sxs-lookup"><span data-stu-id="26c55-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="26c55-142">エンド ユーザーからは、応答時間の増加やサービスのタイムアウトに起因した障害が報告されることになります。また、こうした障害が発生した場合、HTTP 500 (内部サーバー) エラーまたは HTTP 503 (サービスを利用できません) エラーが返されます。</span><span class="sxs-lookup"><span data-stu-id="26c55-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="26c55-143">Web サーバーのイベント ログをよく調べてください。エラーの原因や状況についてさらに詳しい情報が記録されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="26c55-144">この問題の識別に役立てるために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="26c55-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="26c55-145">運用システムのプロセス監視を実行して、応答速度が低下したポイントを特定します。</span><span class="sxs-lookup"><span data-stu-id="26c55-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="26c55-146">これらのポイントで収集されたテレメトリ データを調べ、実行されている操作と使用されているリソースの組み合わせを確認します。</span><span class="sxs-lookup"><span data-stu-id="26c55-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span>
3. <span data-ttu-id="26c55-147">これらのタイミングで実行されていた操作のボリュームや組み合わせと、応答速度の悪化との間に相関関係がないか調べます。</span><span class="sxs-lookup"><span data-stu-id="26c55-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="26c55-148">疑わしい操作についてそれぞれロード テストを実行し、どの操作がリソースを消費して他の操作のリソースを逼迫させているかを特定します。</span><span class="sxs-lookup"><span data-stu-id="26c55-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span>
5. <span data-ttu-id="26c55-149">それらの操作をソース コードで確認し、リソースが過剰に消費されている理由を特定します。</span><span class="sxs-lookup"><span data-stu-id="26c55-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="26c55-150">診断の例</span><span class="sxs-lookup"><span data-stu-id="26c55-150">Example diagnosis</span></span>

<span data-ttu-id="26c55-151">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。</span><span class="sxs-lookup"><span data-stu-id="26c55-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="26c55-152">速度が低下したポイントを特定する</span><span class="sxs-lookup"><span data-stu-id="26c55-152">Identify points of slowdown</span></span>

<span data-ttu-id="26c55-153">各メソッドをインストルメント化して、それぞれの要求に費やされている時間とリソースを追跡します。</span><span class="sxs-lookup"><span data-stu-id="26c55-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="26c55-154">アプリケーションは運用環境で監視してください。</span><span class="sxs-lookup"><span data-stu-id="26c55-154">Then monitor the application in production.</span></span> <span data-ttu-id="26c55-155">そうすることで、個々の要求が互いにどのように競合しているのかについて全体的な視点から把握することができます。</span><span class="sxs-lookup"><span data-stu-id="26c55-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="26c55-156">負荷の大きい時間帯は、リソース消費の激しい、実行に時間のかかる要求が互いに影響を及ぼし合うことが考えられます。システムを監視し、パフォーマンスの低下に注目することによって、その動作を観察することができます。</span><span class="sxs-lookup"><span data-stu-id="26c55-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="26c55-157">次の画像は、監視ダッシュボードを示しています </span><span class="sxs-lookup"><span data-stu-id="26c55-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="26c55-158">(テストには [AppDynamics] を使用しました)。最初、システムには軽い負荷がかかっているだけです。</span><span class="sxs-lookup"><span data-stu-id="26c55-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="26c55-159">その後、ユーザーは `UserProfile` GET メソッドを要求し始めます。</span><span class="sxs-lookup"><span data-stu-id="26c55-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="26c55-160">パフォーマンスは比較的良好ですが、他のユーザーが `WorkInFrontEnd` POST メソッドの要求を発行した途端に状況が一変します。</span><span class="sxs-lookup"><span data-stu-id="26c55-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="26c55-161">その時点で、応答時間が急激に悪化します (1 つ目の矢印)。</span><span class="sxs-lookup"><span data-stu-id="26c55-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="26c55-162">応答時間が改善されるのは、`WorkInFrontEnd` コントローラーに対する要求のボリュームが減少した後です (2 つ目の矢印)。</span><span class="sxs-lookup"><span data-stu-id="26c55-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![WorkInFrontEnd コントローラーが使用されているときのすべての要求の応答時間の影響を示す AppDynamics の [Business Transactions]\(ビジネス トランザクション\) ウィンドウ][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="26c55-164">テレメトリ データを調べて相関関係を見つける</span><span class="sxs-lookup"><span data-stu-id="26c55-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="26c55-165">次の画像は、同じ期間におけるリソース使用率を監視するために収集されたメトリックを示しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="26c55-166">最初、システムにアクセスしているユーザーはほとんどいません。</span><span class="sxs-lookup"><span data-stu-id="26c55-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="26c55-167">接続ユーザーが増えると CPU 使用率が一気に上昇します (100%)。</span><span class="sxs-lookup"><span data-stu-id="26c55-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="26c55-168">また、CPU 使用率の上昇に伴い、最初はネットワーク I/O 率が上昇していることに注目してください。</span><span class="sxs-lookup"><span data-stu-id="26c55-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="26c55-169">しかし実は、いったん CPU 使用率がピークに達すると、ネットワーク I/O が下降しているのです。</span><span class="sxs-lookup"><span data-stu-id="26c55-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="26c55-170">なぜならば、CPU がその処理能力に達した後は、システムで処理できる要求が比較的少数に限られるからです。</span><span class="sxs-lookup"><span data-stu-id="26c55-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="26c55-171">ユーザーが接続を解除すると、CPU の負荷が次第に減少します。</span><span class="sxs-lookup"><span data-stu-id="26c55-171">As users disconnect, the CPU load tails off.</span></span>

![CPU 使用率とネットワーク使用率を示す AppDynamics のメトリック][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="26c55-173">この時点で、さらに詳しい調査の対象として、見かけ上まず候補に挙げられるのは `WorkInFrontEnd` コントローラーの `Post` メソッドでしょう。</span><span class="sxs-lookup"><span data-stu-id="26c55-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="26c55-174">この仮説を検証するためには、管理された環境で、さらに調査を進める必要があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="26c55-175">ロード テストを実行する</span><span class="sxs-lookup"><span data-stu-id="26c55-175">Perform load testing</span></span>

<span data-ttu-id="26c55-176">次のステップは、管理された環境でテストを実行することです。</span><span class="sxs-lookup"><span data-stu-id="26c55-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="26c55-177">たとえば、それぞれの要求について、それを含めた場合と省略した場合を比較する一連のロード テストを実行し、順次その影響を確認していきます。</span><span class="sxs-lookup"><span data-stu-id="26c55-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="26c55-178">以下のグラフは、前のテストで使用したクラウド サービスとまったく同じデプロイ環境に対するロード テストの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="26c55-179">このテストでは、500 ユーザーという一定の負荷で `UserProfile` コントローラーの `Get` 操作を実行すると共に、`WorkInFrontEnd` コントローラーの `Post` 操作を実行するユーザー数 (負荷) を段階的に増やしています。</span><span class="sxs-lookup"><span data-stu-id="26c55-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span>

![WorkInFrontEnd コントローラーの初回ロード テストの結果][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="26c55-181">最初の段階の負荷は 0 で、アクティブなユーザーが実行しているのは `UserProfile` 要求だけです。</span><span class="sxs-lookup"><span data-stu-id="26c55-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="26c55-182">このシステムは毎秒約 500 件の要求に応答することができます。</span><span class="sxs-lookup"><span data-stu-id="26c55-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="26c55-183">60 秒後、負荷として、新たに 100 ユーザーが `WorkInFrontEnd` コントローラーに POST 要求を送信し始めます。</span><span class="sxs-lookup"><span data-stu-id="26c55-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="26c55-184">すると、ほとんど間髪を入れずに、`UserProfile` コントローラーに送信されるワークロードは、1 秒あたり約 150 の要求まで減少します。</span><span class="sxs-lookup"><span data-stu-id="26c55-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="26c55-185">これはロード テスト実行プログラムの動作に起因するものです。</span><span class="sxs-lookup"><span data-stu-id="26c55-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="26c55-186">応答を待って次の要求が送信されるので、応答が返されるまでに時間がかかるほど、要求の速度が低下します。</span><span class="sxs-lookup"><span data-stu-id="26c55-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="26c55-187">`WorkInFrontEnd` コントローラーに POST 要求を送信するユーザーが増えるにつれて、`UserProfile` コントローラーの応答速度は低下し続けていきます。</span><span class="sxs-lookup"><span data-stu-id="26c55-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="26c55-188">しかし、`WorkInFrontEnd` コントローラーによって処理される要求のボリュームは比較的一定の状態を保っていることに注目してください。</span><span class="sxs-lookup"><span data-stu-id="26c55-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="26c55-189">両方の要求の全体的な速度が安定に向かいつつも、下限に近い状態にあることから、システムが飽和状態になっていることがはっきりと見て取れます。</span><span class="sxs-lookup"><span data-stu-id="26c55-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>

### <a name="review-the-source-code"></a><span data-ttu-id="26c55-190">ソース コードをレビューする</span><span class="sxs-lookup"><span data-stu-id="26c55-190">Review the source code</span></span>

<span data-ttu-id="26c55-191">最後のステップとして、ソース コードに注目してみましょう。</span><span class="sxs-lookup"><span data-stu-id="26c55-191">The final step is to look at the source code.</span></span> <span data-ttu-id="26c55-192">開発チームは `Post` メソッドにかなりの時間がかかる可能性があることを認識していました。最初の実装段階から、スレッドを独立させていたのは、そのためです。</span><span class="sxs-lookup"><span data-stu-id="26c55-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="26c55-193">それによって差し迫った問題は解決されました。長時間実行されるタスクの完了を、`Post` メソッドがブロック状態で待つことはないからです。</span><span class="sxs-lookup"><span data-stu-id="26c55-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="26c55-194">しかしこのメソッドによって実行される処理は、依然として CPU やメモリなどのリソースを消費します。</span><span class="sxs-lookup"><span data-stu-id="26c55-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="26c55-195">このプロセスが非同期に実行できる場合、実際にはパフォーマンスが低下する可能性があります。ユーザーによって際限なく、大量の操作が同時にトリガーされることも考えられるからです。</span><span class="sxs-lookup"><span data-stu-id="26c55-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="26c55-196">サーバーが実行できるスレッドの数には制限があります。</span><span class="sxs-lookup"><span data-stu-id="26c55-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="26c55-197">その制限を超えて新しいスレッドをアプリケーションが起動しようとしたとき、高い確率で例外が発生します。</span><span class="sxs-lookup"><span data-stu-id="26c55-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="26c55-198">だからといって非同期操作を避けなければならない、という意味ではありません。</span><span class="sxs-lookup"><span data-stu-id="26c55-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="26c55-199">このような場合は、ネットワーク呼び出しで非同期の await を実行することが推奨されます </span><span class="sxs-lookup"><span data-stu-id="26c55-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="26c55-200">([同期 I/O][sync-io]のアンチパターンに関するページを参照)。ここでの問題は、CPU 負荷の高い処理が別のスレッドで生成されることです。</span><span class="sxs-lookup"><span data-stu-id="26c55-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="26c55-201">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="26c55-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="26c55-202">次の画像は、解決策の導入後にパフォーマンスを監視した結果です。</span><span class="sxs-lookup"><span data-stu-id="26c55-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="26c55-203">負荷は前回と同様ですが、`UserProfile` コントローラーの応答時間が大幅に短縮されています。</span><span class="sxs-lookup"><span data-stu-id="26c55-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="26c55-204">同じ時間における要求のボリュームは、2,759 から 23,565 に増えています。</span><span class="sxs-lookup"><span data-stu-id="26c55-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span>

![WorkInBackground コントローラーが使用されているときのすべての要求の応答時間の影響を示す AppDynamics の [Business Transactions]\(ビジネス トランザクション\) ウィンドウ][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="26c55-206">`WorkInBackground` コントローラーによって処理される要求のボリュームも大幅に増えていることに注目してください。</span><span class="sxs-lookup"><span data-stu-id="26c55-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="26c55-207">しかしこの場合、両者を直接比較することはできません。このコントローラーで実行される処理は、最初のコードと大きく異なるからです。</span><span class="sxs-lookup"><span data-stu-id="26c55-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="26c55-208">新たに書き換えたコードでは、時間がかかる計算を実行せず、要求をキューに追加しているにすぎません。</span><span class="sxs-lookup"><span data-stu-id="26c55-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="26c55-209">重要な点は、このメソッドがシステム全体の足を引っ張ることがなくなったということです。</span><span class="sxs-lookup"><span data-stu-id="26c55-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="26c55-210">CPU 使用率とネットワーク使用率にも、パフォーマンスの改善が見られます。</span><span class="sxs-lookup"><span data-stu-id="26c55-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="26c55-211">CPU 使用率が 100% に達することはなくなりました。また、処理されたネットワーク要求のボリュームも、先ほどより大幅に増え、ワークロードが低下するまで減少しませんでした。</span><span class="sxs-lookup"><span data-stu-id="26c55-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![WorkInBackground コントローラーの CPU 使用率とネットワーク使用率を示す AppDynamics のメトリック][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="26c55-213">次のグラフは、ロード テストの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="26c55-214">前回のテストと比べ、処理された要求の全体的なボリュームが大幅に改善しています。</span><span class="sxs-lookup"><span data-stu-id="26c55-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![BackgroundImageProcessing コントローラーのロード テストの結果][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="26c55-216">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="26c55-216">Related guidance</span></span>

- <span data-ttu-id="26c55-217">[自動スケールのベスト プラクティス][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="26c55-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="26c55-218">[バックグラウンド ジョブのベスト プラクティス][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="26c55-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="26c55-219">[キュー ベースの負荷平準化パターン][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="26c55-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="26c55-220">[Web/キュー/ワーカー アーキテクチャ スタイル][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="26c55-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: https://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg
