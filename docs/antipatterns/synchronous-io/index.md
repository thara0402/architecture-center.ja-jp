---
title: "同期 I/O のアンチパターン"
description: "I/O が完了するまで呼び出し元スレッドをブロックすることにより、パフォーマンスが低下して、垂直拡張性に影響を及ぼすことがあります。"
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="8dbb7-103">同期 I/O のアンチパターン</span><span class="sxs-lookup"><span data-stu-id="8dbb7-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="8dbb7-104">I/O が完了するまで呼び出し元スレッドをブロックすることにより、パフォーマンスが低下して、垂直拡張性に影響を及ぼすことがあります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="8dbb7-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="8dbb7-105">Problem description</span></span>

<span data-ttu-id="8dbb7-106">同期 I/O 操作は、I/O が完了するまで呼び出し元のスレッドをブロックします。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="8dbb7-107">呼び出し元のスレッドは待機状態になり、その間は有益な処理を実行することができず処理リソースを浪費します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="8dbb7-108">I/O の一般的な例は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="8dbb7-109">データベースまたは任意の種類の永続的ストレージにデータを取得 (永続化) します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="8dbb7-110">Web サービスに要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="8dbb7-111">メッセージを投稿するか、キューからメッセージを取得します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="8dbb7-112">ローカル ファイルに対して書き込みまたは読み取りを行います。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="8dbb7-113">このアンチパターンは、通常、次の理由で発生します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="8dbb7-114">アンチパターンが、操作を実行するための最も簡単な方法であると感じられるため。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="8dbb7-115">アプリケーションが要求からの応答を必要とするため。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="8dbb7-116">アプリケーションが使用するライブラリが I/O に対して同期メソッドしか提供しないため。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="8dbb7-117">外部ライブラリが同期 I/O を内部で実行するため。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="8dbb7-118">1 回の同期 I/O 呼び出しによって呼び出しチェーン全体がブロックされます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="8dbb7-119">次のコードによって、ファイルが Azure BLOB ストレージにアップロードされます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="8dbb7-120">このコードで同期 I/O の待機をブロックする箇所が 2 つあります。`CreateIfNotExists` メソッドと `UploadFromStream` メソッドです。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="8dbb7-121">外部サービスからの応答を待機する例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="8dbb7-122">`GetUserProfile` メソッドは、`UserProfile` を返すリモート サービスを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="8dbb7-123">この 2 つの例両方の完全なコードについては、[こちら][sample-app]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="8dbb7-124">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="8dbb7-124">How to fix the problem</span></span>

<span data-ttu-id="8dbb7-125">同期 I/O 操作を非同期操作に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="8dbb7-126">これにより、現在のスレッドがブロックされずに有意義な作業の実行を続けて、コンピューティング リソースの使用率の向上につながります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="8dbb7-127">I/O の非同期実行は、クライアント アプリケーションからの要求の予期しない増加に対応する際に特に有効です。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="8dbb7-128">多くのライブラリでは、メソッドの同期バージョンと非同期バージョンの両方が提供されます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="8dbb7-129">可能な限り、非同期バージョンを使用してください。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="8dbb7-130">次に示すのは、ファイルを Azure BLOB ストレージにアップロードする前の例の非同期バージョンです。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="8dbb7-131">`await` 演算子は、非同期操作が実行されている間、制御を呼び出し元環境に返します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="8dbb7-132">非同期操作が完了してから、このステートメントの後のコードが継続して実行されます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="8dbb7-133">適切に設計されたサービスは、非同期操作も提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="8dbb7-134">ユーザー プロファイルを返す Web サービスの非同期バージョンを次に示します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="8dbb7-135">`GetUserProfileAsync` メソッドは、ユーザー プロファイル サービスの非同期バージョンの使用に依存しています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="8dbb7-136">操作の非同期バージョンが提供されないライブラリの場合は、選択した同期メソッドに対して非同期ラッパーを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="8dbb7-137">このアプローチに従う際には注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-137">Follow this approach with caution.</span></span> <span data-ttu-id="8dbb7-138">非同期ラッパーを呼び出したスレッドの応答性を向上させることができる一方で、実際には消費リソース量が増加します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="8dbb7-139">スレッドが追加作成されることがあり、このスレッドによって行われる作業の同期に関連するオーバーヘッドが生じます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="8dbb7-140">一部のトレードオフについては、このブログ記事「[Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]」(同期メソッドの非同期ラッパーを公開する必要があるか) で説明されています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="8dbb7-141">同期メソッドの非同期ラッパーの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="8dbb7-142">これで、呼び出し元コードは、ラッパーを待機できます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="8dbb7-143">考慮事項</span><span class="sxs-lookup"><span data-stu-id="8dbb7-143">Considerations</span></span>

- <span data-ttu-id="8dbb7-144">実行時間がきわめて短く、競合を引き起こす可能性が少ない I/O 操作は、同期操作とした方がパフォーマンスが向上する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="8dbb7-145">たとえば、SSD ドライブでの小容量ファイルの読み取りです。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="8dbb7-146">タスクの別のスレッドへのディスパッチや、タスク完了時のそのスレッドとの同期のオーバーヘッドが、非同期 I/O のメリットを上回る場合があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="8dbb7-147">ただし、このようなケースは比較的まれで、ほとんどの I/O 操作は非同期で行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="8dbb7-148">I/O パフォーマンスを向上させると、システムの他の部分のボトルネックを引き起こすことがあります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="8dbb7-149">たとえば、スレッドをブロック解除すると、共有リソースへの同時要求が増加して、リソースの枯渇すなわちリソースの調整につながることがあります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="8dbb7-150">この問題が生じた場合は、Web サーバーまたはパーティション データ ストアの数を拡張して、競合を減らす必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="8dbb7-151">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="8dbb7-151">How to detect the problem</span></span>

<span data-ttu-id="8dbb7-152">ユーザーにとっては、定期的にアプリケーションが応答していないように見えたり、ハングアップしていると感じたりすることがあります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="8dbb7-153">アプリケーションは、タイムアウト例外で失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="8dbb7-154">このようなエラーでは、HTTP 500 (Internal Server) のエラーが返されることもあります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="8dbb7-155">サーバーでは、スレッドが使用可能になるまで、入っているクライアント要求がブロックされるため、要求キューの長さが超過して HTTP 503 (Service Unavailable) というエラーが発生します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="8dbb7-156">問題の識別に役立てるために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="8dbb7-157">実稼働システムを監視し、ブロックされたワーカー スレッドがスループットを抑制しているかどうかを判別します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="8dbb7-158">スレッド不足のために要求がブロックされている場合は、アプリケーションを確認して、操作の I/O が同期で実行しているかどうかを判別します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="8dbb7-159">同期 I/O を実行している各操作について制御したロード テストを行って、それらの操作がシステム パフォーマンスに影響しているかどうかを明らかにします。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="8dbb7-160">診断の例</span><span class="sxs-lookup"><span data-stu-id="8dbb7-160">Example diagnosis</span></span>

<span data-ttu-id="8dbb7-161">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="8dbb7-162">Web サーバーのパフォーマンスの監視</span><span class="sxs-lookup"><span data-stu-id="8dbb7-162">Monitor web server performance</span></span>

<span data-ttu-id="8dbb7-163">Azure の Web アプリケーションおよび Web ロールについて、IIS Web サーバーのパフォーマンスを監視することをお薦めします。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="8dbb7-164">具体的には、要求キューの長さに注意し、アクティビティ量が多い期間に、要求が使用可能なスレッドを待機した状態でブロックされているかどうかを証明します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="8dbb7-165">この情報は、Azure 診断を有効にして収集できます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="8dbb7-166">詳細については、次を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-166">For more information, see:</span></span>

- <span data-ttu-id="8dbb7-167">[Azure App Service のアプリの監視][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="8dbb7-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="8dbb7-168">[Azure アプリケーションでのパフォーマンス カウンターの作成と使用][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="8dbb7-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="8dbb7-169">アプリケーションをインストルメント化して、要求が受け入れられた後にどのように処理されるかを確認します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="8dbb7-170">要求のフローをトレースすると、要求が実行速度の遅い呼び出しを実行しており、現在のスレッドをブロックしているかどうかを特定するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="8dbb7-171">スレッドのプロファイリングによって、ブロックされている要求を明らかにすることもできます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="8dbb7-172">アプリケーションのロード テスト</span><span class="sxs-lookup"><span data-stu-id="8dbb7-172">Load test the application</span></span>

<span data-ttu-id="8dbb7-173">次のグラフは、前述の同期 `GetUserProfile` メソッドのパフォーマンスです。負荷は同時実行ユーザー数 4000 以内で変化しています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="8dbb7-174">このアプリケーションは、Azure Cloud Service Web ロールで実行している ASP.NET アプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![同期 I/O 操作を実行するサンプル アプリケーションのパフォーマンスのグラフ][sync-performance]

<span data-ttu-id="8dbb7-176">同期操作は、同期 I/O をシミュレートするため 2 秒間スリープ状態になるようにハードコーディングされます。したがって、最小応答時間は 2 秒をわずかに超えます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="8dbb7-177">負荷が 2500 同時実行ユーザー数に近づくと平均応答時間は一定になりますが、1 秒あたりの要求量は増加し続けます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="8dbb7-178">これら 2 つのメジャーのスケールが対数であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="8dbb7-179">1 秒あたりの要求数は、この時点とテストの終了時では倍増しています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="8dbb7-180">これだけでは、このテストから同期 I/O が問題であることは必ずしも明確にはなりません。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="8dbb7-181">さらに高い負荷では、アプリケーションは、Web サーバーが要求を適切なタイミングで処理できなくなる転換点に到達し、クライアント アプリケーションがタイムアウト例外を受け取ることになります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="8dbb7-182">入ってくる要求は IIS Web サーバーによってキューに入れられ、ASP.NET スレッド プールで実行しているスレッドに渡されます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="8dbb7-183">各操作は I/O を同期実行するため、操作が完了するまでスレッドはブロックされます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="8dbb7-184">ワークロードが増加するにつれ、最終的には、スレッド プール内の ASP.NET スレッドすべてが割り当てられてブロックされます。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="8dbb7-185">この時点で、これから入ってくる要求は、キュー内で使用可能なスレッドを待機する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="8dbb7-186">キューが長くなると、要求のタイムアウトが開始します。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="8dbb7-187">ソリューションの実装と結果の検証</span><span class="sxs-lookup"><span data-stu-id="8dbb7-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="8dbb7-188">次のグラフは、非同期バージョンのコードに対してロード テストを行った結果です。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![非同期 I/O 操作を実行するサンプル アプリケーションのパフォーマンスのグラフ][async-performance]

<span data-ttu-id="8dbb7-190">スループットははるかに高くなります。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-190">Throughput is far higher.</span></span> <span data-ttu-id="8dbb7-191">前のテストと同じ期間で、システムはほぼ 10 倍のスループット (1 秒あたりの要求数) を処理することに成功しています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="8dbb7-192">さらに、平均応答時間は相対的に一定であり、前のテストに比べて約 25 分の 1 を保っています。</span><span class="sxs-lookup"><span data-stu-id="8dbb7-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



