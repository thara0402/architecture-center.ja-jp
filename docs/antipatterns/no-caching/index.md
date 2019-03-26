---
title: キャッシュ不使用のアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: 同じデータを繰り返しフェッチすると、パフォーマンスとスケーラビリティが低下する可能性があります。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="no-caching-antipattern"></a><span data-ttu-id="8774d-103">キャッシュ不使用のアンチパターン</span><span class="sxs-lookup"><span data-stu-id="8774d-103">No Caching antipattern</span></span>

<span data-ttu-id="8774d-104">多数の同時要求を処理するクラウド アプリケーションでは、同じデータを繰り返しフェッチすることでパフォーマンスとスケーラビリティが低下する場合があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="8774d-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="8774d-105">Problem description</span></span>

<span data-ttu-id="8774d-106">データがキャッシュされていないことで、次のように、多数の好ましくない動作を引き起こすことがあります。</span><span class="sxs-lookup"><span data-stu-id="8774d-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="8774d-107">I/O オーバーヘッドや待ち時間の点でアクセス コストの大きいリソースから同じ情報を繰り返しフェッチする。</span><span class="sxs-lookup"><span data-stu-id="8774d-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="8774d-108">複数の要求で同じオブジェクトまたは同じデータ構造を繰り返し構築する。</span><span class="sxs-lookup"><span data-stu-id="8774d-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="8774d-109">サービス クォータのあるリモート サービスや、特定の限度を超えたときにクライアントをスロットルするリモート サービスを必要以上に呼び出す。</span><span class="sxs-lookup"><span data-stu-id="8774d-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="8774d-110">これらの問題は、応答時間の遅さ、データ ストアにおける競合の増加、スケーラビリティの低さとして跳ね返ってくることがあります。</span><span class="sxs-lookup"><span data-stu-id="8774d-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="8774d-111">以下の例では、Entity Framework を使用してデータベースに接続しています。</span><span class="sxs-lookup"><span data-stu-id="8774d-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="8774d-112">まったく同じデータをフェッチする要求が複数あっても、クライアントの要求ごとに毎回データベースが呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="8774d-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="8774d-113">要求が繰り返されることで、I/O オーバーヘッドとデータ アクセス料金の観点から見たコストはたちまち累積していくことでしょう。</span><span class="sxs-lookup"><span data-stu-id="8774d-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="8774d-114">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="8774d-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="8774d-115">このアンチパターンは、通常、次の理由で発生します。</span><span class="sxs-lookup"><span data-stu-id="8774d-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="8774d-116">キャッシュを使わない方が、実装が簡単で、負荷の低い条件下では動作上問題がない。</span><span class="sxs-lookup"><span data-stu-id="8774d-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="8774d-117">キャッシュを使うとコードが複雑化する。</span><span class="sxs-lookup"><span data-stu-id="8774d-117">Caching makes the code more complicated.</span></span>
- <span data-ttu-id="8774d-118">キャッシュを使うことの利点と欠点がよくわからない。</span><span class="sxs-lookup"><span data-stu-id="8774d-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="8774d-119">キャッシュされたデータの確度と鮮度を維持することで生じるオーバーヘッドが気になる。</span><span class="sxs-lookup"><span data-stu-id="8774d-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="8774d-120">ネットワーク待ち時間が問題にならないオンプレミス システムからアプリケーションを移行した。オンプレミス システムは高価で高性能なハードウェア上で運用されていたため、当初の設計ではキャッシュが考慮されていない。</span><span class="sxs-lookup"><span data-stu-id="8774d-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="8774d-121">特定のシナリオでキャッシュが有効な選択肢となりうることを開発者が理解していない。</span><span class="sxs-lookup"><span data-stu-id="8774d-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="8774d-122">たとえば Web API を実装する際に ETag が検討されない。</span><span class="sxs-lookup"><span data-stu-id="8774d-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="8774d-123">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="8774d-123">How to fix the problem</span></span>

<span data-ttu-id="8774d-124">最も一般的なキャッシュ方式は "*オンデマンド*" と "*キャッシュ アサイド*" です。</span><span class="sxs-lookup"><span data-stu-id="8774d-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="8774d-125">読み取り時、アプリケーションはキャッシュからデータの読み取りを試みます。</span><span class="sxs-lookup"><span data-stu-id="8774d-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="8774d-126">データがキャッシュにない場合、アプリケーションはデータ ソースからデータを取得してキャッシュに追加します。</span><span class="sxs-lookup"><span data-stu-id="8774d-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="8774d-127">書き込み時、アプリケーションは変更を直接データ ソースに書き込み、古い値をキャッシュから削除します。</span><span class="sxs-lookup"><span data-stu-id="8774d-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="8774d-128">次回必要になったときに、それが取得されてキャッシュに追加されます。</span><span class="sxs-lookup"><span data-stu-id="8774d-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="8774d-129">この方法は、頻繁に変更されるデータに適しています。</span><span class="sxs-lookup"><span data-stu-id="8774d-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="8774d-130">以下に示したのは、[キャッシュ アサイド][cache-aside-pattern] パターンを使用するように先ほどの例を書き換えたものです。</span><span class="sxs-lookup"><span data-stu-id="8774d-130">Here is the previous example updated to use the [Cache-Aside][cache-aside-pattern] pattern.</span></span>

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="8774d-131">`GetAsync` メソッドに注目してください。先ほどはデータベースを直接呼び出していましたが、`CacheService` クラスを呼び出しています。</span><span class="sxs-lookup"><span data-stu-id="8774d-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="8774d-132">`CacheService` クラスはまず、Azure Redis Cache からキャッシュ項目の取得を試みます。</span><span class="sxs-lookup"><span data-stu-id="8774d-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="8774d-133">目的の値が Redis Cache に見つからなかった場合、`CacheService` は、呼び出し元から渡されたラムダ関数を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="8774d-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="8774d-134">このラムダ関数の役割は、データベースからデータをフェッチすることです。</span><span class="sxs-lookup"><span data-stu-id="8774d-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="8774d-135">この実装によって、特定のキャッシュ ソリューションからリポジトリが分離され、`CacheService` がデータベースから分離されています。</span><span class="sxs-lookup"><span data-stu-id="8774d-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span>

## <a name="considerations"></a><span data-ttu-id="8774d-136">考慮事項</span><span class="sxs-lookup"><span data-stu-id="8774d-136">Considerations</span></span>

- <span data-ttu-id="8774d-137">一時的な障害などでキャッシュが利用できない場合、クライアントにはエラーを返さないでください。</span><span class="sxs-lookup"><span data-stu-id="8774d-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="8774d-138">代わりに、元のデータ ソースからデータをフェッチするようにします。</span><span class="sxs-lookup"><span data-stu-id="8774d-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="8774d-139">ただしキャッシュを回復している間に、元のデータ ストアが要求に対応しきれず、タイムアウトが発生したり接続に失敗したりする可能性があります </span><span class="sxs-lookup"><span data-stu-id="8774d-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="8774d-140">(結局はそれが、キャッシュを使用するそもそもの動機の 1 つです)。[サーキット ブレーカー パターン][circuit-breaker]などの手法を用いて、データ ソースへの過剰な負荷を回避してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="8774d-141">非静的データをキャッシュするアプリケーションは、結果整合性をサポートするように設計されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="8774d-142">Web API に関しては、要求メッセージと応答メッセージに Cache-Control ヘッダーを追加し、ETag を使ってオブジェクトのバージョンを識別することによって、クライアント側キャッシュをサポートできます。</span><span class="sxs-lookup"><span data-stu-id="8774d-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="8774d-143">詳細については、「[API implementation (API の実装)][api-implementation]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="8774d-144">エンティティ全体をキャッシュする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="8774d-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="8774d-145">エンティティの大半が静的データで、頻繁に変更される部分がごく一部に限られるのであれば、静的な要素をキャッシュして、動的な要素はデータ ソースから取得するようにしてください。</span><span class="sxs-lookup"><span data-stu-id="8774d-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="8774d-146">この方法には、データ ソースに対して実行される I/O の量を減らす効果があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="8774d-147">場合によっては、存続期間が短い揮発性データはキャッシュすると効果的です。</span><span class="sxs-lookup"><span data-stu-id="8774d-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="8774d-148">たとえば最新のステータス情報を絶えず送信するデバイスがあるとします。</span><span class="sxs-lookup"><span data-stu-id="8774d-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="8774d-149">受信した情報をキャッシュし、永続ストアには一切書き込まないことは理にかなっていると考えられます。</span><span class="sxs-lookup"><span data-stu-id="8774d-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>

- <span data-ttu-id="8774d-150">データの鮮度を保つために、多くのキャッシュ ソリューションは有効期限を構成できるようになっていて、その場合、一定時間が経過したデータはキャッシュから自動的に削除されます。</span><span class="sxs-lookup"><span data-stu-id="8774d-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="8774d-151">その有効期限は、実際のシナリオに応じて自分で調整することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="8774d-152">変更頻度の低い静的なデータは、すぐに鮮度が落ちる揮発性データよりも長くキャッシュに置くことができます。</span><span class="sxs-lookup"><span data-stu-id="8774d-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="8774d-153">キャッシュ ソリューションに有効期限が標準装備されていない場合は、キャッシュが際限なく肥大化するのを防ぐために、ときどきキャッシュを掃除するバックグラウンド処理を独自に実装する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span>

- <span data-ttu-id="8774d-154">外部データ ソースからデータをキャッシュするだけでなく、キャッシュを使用して、複雑な計算の結果を保存することもできます。</span><span class="sxs-lookup"><span data-stu-id="8774d-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="8774d-155">ただしその前に、アプリケーションをインストルメント化して、アプリケーションに本当に CPU 制約が適用されているかどうかを確かめてください。</span><span class="sxs-lookup"><span data-stu-id="8774d-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="8774d-156">場合によっては、アプリケーションが起動した時点で最初からキャッシュにデータを投入しておくと効果的です。</span><span class="sxs-lookup"><span data-stu-id="8774d-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="8774d-157">使用される可能性が最も高いデータをキャッシュに格納してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="8774d-158">キャッシュ ヒットとキャッシュ ミスを検出するインストルメンテーションは必ず追加してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="8774d-159">その情報をもとに、キャッシュするデータやキャッシュにデータを保持する時間 (有効期限) など、キャッシュ ポリシーを調整することができます。</span><span class="sxs-lookup"><span data-stu-id="8774d-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="8774d-160">キャッシュがないことがボトルネックになっている場合に、キャッシュを追加したことで要求のボリュームが増えすぎて、Web フロントエンドがオーバーフローする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="8774d-161">クライアントに HTTP 503 (サービスを利用できません) エラーが返されることがあります。</span><span class="sxs-lookup"><span data-stu-id="8774d-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="8774d-162">それらはフロントエンドのスケールアウトの必要性を示す徴候となります。</span><span class="sxs-lookup"><span data-stu-id="8774d-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="8774d-163">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="8774d-163">How to detect the problem</span></span>

<span data-ttu-id="8774d-164">キャッシュの不使用がパフォーマンスの問題につながるどうかを特定しやすくするために、次の手順を実行してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="8774d-165">アプリケーションの設計を確認します。</span><span class="sxs-lookup"><span data-stu-id="8774d-165">Review the application design.</span></span> <span data-ttu-id="8774d-166">アプリケーションで使用するすべてのデータ ストアの一覧を作成してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="8774d-167">それぞれについて、アプリケーションにキャッシュが使用されているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="8774d-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="8774d-168">可能であれば、データの変更頻度を特定します。</span><span class="sxs-lookup"><span data-stu-id="8774d-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="8774d-169">まずキャッシュの候補となるのは、少しずつ変化するデータや、頻繁に読み取られる静的な参照データです。</span><span class="sxs-lookup"><span data-stu-id="8774d-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span>

2. <span data-ttu-id="8774d-170">アプリケーションをインストルメント化したうえでライブ システムを監視し、アプリケーションがデータを取得したり情報を計算したりする頻度を突き止めます。</span><span class="sxs-lookup"><span data-stu-id="8774d-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="8774d-171">テスト環境でアプリケーションをプロファイリングし、データ アクセス操作やその他の頻繁に実行される計算に関連付けられたオーバーヘッドについて、低レベルのメトリックを収集します。</span><span class="sxs-lookup"><span data-stu-id="8774d-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="8774d-172">テスト環境でロード テストを実行し、通常のワークロード下と大きな負荷がかかっている状況下でシステムが応答するようすを確認します。</span><span class="sxs-lookup"><span data-stu-id="8774d-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="8774d-173">ロード テストでは、運用環境で観察されるデータ アクセスのパターンを現実的なワークロードでシミュレートする必要があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span>

5. <span data-ttu-id="8774d-174">基になるデータ ストアに関してデータ アクセスの統計を調査し、同じデータ要求が繰り返される頻度を確認します。</span><span class="sxs-lookup"><span data-stu-id="8774d-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="8774d-175">診断の例</span><span class="sxs-lookup"><span data-stu-id="8774d-175">Example diagnosis</span></span>

<span data-ttu-id="8774d-176">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。</span><span class="sxs-lookup"><span data-stu-id="8774d-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="8774d-177">アプリケーションをインストルメント化してライブ システムを監視する</span><span class="sxs-lookup"><span data-stu-id="8774d-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="8774d-178">アプリケーションをインストルメント化して監視し、その運用環境でユーザーが実行する具体的な要求についての情報を入手します。</span><span class="sxs-lookup"><span data-stu-id="8774d-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span>

<span data-ttu-id="8774d-179">次の画像は、ロード テスト時に [New Relic][NewRelic] によって収集された監視データを示したものです。</span><span class="sxs-lookup"><span data-stu-id="8774d-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="8774d-180">この例で実行された HTTP GET 操作は `Person/GetAsync` だけです。</span><span class="sxs-lookup"><span data-stu-id="8774d-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="8774d-181">しかし実際の運用環境では、個々の要求が実行される相対的な頻度を把握することで、キャッシュすべきリソースについての洞察が得られます。</span><span class="sxs-lookup"><span data-stu-id="8774d-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![サーバーに対して CachingDemo アプリケーションが実行する要求を New Relic で監視][NewRelic-server-requests]

<span data-ttu-id="8774d-183">さらに詳しく分析する必要がある場合は、プロファイラーを使用して低レベルのパフォーマンス データを (運用環境のシステムではなく) テスト環境で収集することができます。</span><span class="sxs-lookup"><span data-stu-id="8774d-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="8774d-184">I/O 要求レート、メモリ使用量、CPU 使用率などのメトリックに注目してください。</span><span class="sxs-lookup"><span data-stu-id="8774d-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="8774d-185">データ ストアやサービスに対する大量の要求、または同じ計算を実行する処理の繰り返しが、これらのメトリックによって明らかになる場合があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="8774d-186">アプリケーションのロード テストを実行する</span><span class="sxs-lookup"><span data-stu-id="8774d-186">Load test the application</span></span>

<span data-ttu-id="8774d-187">次のグラフは、サンプル アプリケーションのロード テストの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="8774d-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="8774d-188">このロード テストでは、標準的な操作の流れを実行するユーザーの負荷を最大 800 人まで段階的に増やしていくシミュレーションを実行します。</span><span class="sxs-lookup"><span data-stu-id="8774d-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span>

![パフォーマンス ロード テストの結果 (キャッシュ不使用時)][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="8774d-190">1 秒あたりの成功したテストの数が頭打ちになると、結果的にそれ以降の要求は処理速度が低下します。</span><span class="sxs-lookup"><span data-stu-id="8774d-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="8774d-191">平均テスト時間は、ワークロードと共に着実に増えていきます。</span><span class="sxs-lookup"><span data-stu-id="8774d-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="8774d-192">ユーザー負荷がピークに達した後、応答時間は横ばいになります。</span><span class="sxs-lookup"><span data-stu-id="8774d-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="8774d-193">データ アクセスの統計を調査する</span><span class="sxs-lookup"><span data-stu-id="8774d-193">Examine data access statistics</span></span>

<span data-ttu-id="8774d-194">データ アクセスの統計をはじめ、データ ストアから提供される各種の情報から、どのクエリが極端な頻度で繰り返されているかなど、有益なデータが得られる場合があります。</span><span class="sxs-lookup"><span data-stu-id="8774d-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="8774d-195">たとえば Microsoft SQL Server では、最近実行されたクエリの統計情報が `sys.dm_exec_query_stats` 管理ビューに表示されます。</span><span class="sxs-lookup"><span data-stu-id="8774d-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="8774d-196">各クエリのテキストは、`sys.dm_exec-query_plan` ビューで確認できます。</span><span class="sxs-lookup"><span data-stu-id="8774d-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="8774d-197">SQL Server Management Studio などのツールから次の SQL クエリを実行することで、クエリの実行頻度を確認できます。</span><span class="sxs-lookup"><span data-stu-id="8774d-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="8774d-198">実行結果の `UseCount` 列にそれぞれのクエリの実行頻度が示されます。</span><span class="sxs-lookup"><span data-stu-id="8774d-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="8774d-199">次の画像を見ると、3 つ目クエリが 250,000 回以上実行され、他のクエリに比べて著しく多いことがわかります。</span><span class="sxs-lookup"><span data-stu-id="8774d-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![SQL Server Management Server で動的管理ビューにクエリを実行した結果][Dynamic-Management-Views]

<span data-ttu-id="8774d-201">多数のデータベース要求の原因となっている SQL クエリを次に示します。</span><span class="sxs-lookup"><span data-stu-id="8774d-201">Here is the SQL query that is causing so many database requests:</span></span>

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="8774d-202">このクエリは、前述の `GetByIdAsync` メソッドで Entity Framework によって生成されたものです。</span><span class="sxs-lookup"><span data-stu-id="8774d-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="8774d-203">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="8774d-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="8774d-204">キャッシュを実装したうえで再度ロード テストを実行し、キャッシュを使わずに行った前回のロード テストの結果と比較します。</span><span class="sxs-lookup"><span data-stu-id="8774d-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="8774d-205">サンプル アプリケーションにキャッシュを追加した後のロード テストの結果は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="8774d-205">Here are the load test results after adding a cache to the sample application.</span></span>

![パフォーマンス ロード テストの結果 (キャッシュ使用時)][Performance-Load-Test-Results-Cached]

<span data-ttu-id="8774d-207">成功したテストの数はやはり頭打ちになりますが、そのときのユーザー負荷は、さきほどよりも高くなっています。</span><span class="sxs-lookup"><span data-stu-id="8774d-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="8774d-208">この負荷時の要求レートは、前回よりも大幅に高くなっています。</span><span class="sxs-lookup"><span data-stu-id="8774d-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="8774d-209">平均テスト時間は負荷と共に上昇しますが、最大応答時間は 0.05 ms と、前回の 1ms と比べて &mdash;20&times; 倍の改善が見られます。</span><span class="sxs-lookup"><span data-stu-id="8774d-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span>

## <a name="related-resources"></a><span data-ttu-id="8774d-210">関連リソース</span><span class="sxs-lookup"><span data-stu-id="8774d-210">Related resources</span></span>

- <span data-ttu-id="8774d-211">[API 実装のベスト プラクティス][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="8774d-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="8774d-212">[キャッシュ アサイド パターン][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="8774d-212">[Cache-Aside pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="8774d-213">[キャッシングのベスト プラクティス][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="8774d-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="8774d-214">[サーキット ブレーカー パターン][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="8774d-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: https://newrelic.com/partner/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg
