---
title: キャッシュ アサイド
description: オンデマンドでデータをデータ ストアからキャッシュに読み込みます。
keywords: 設計パターン
author: dragon119
ms.date: 11/01/2018
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 4c93ed02ff28e79cedc26f83364592baba96821d
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916382"
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="334c3-104">キャッシュ アサイド パターン</span><span class="sxs-lookup"><span data-stu-id="334c3-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="334c3-105">オンデマンドでデータをデータ ストアからキャッシュに読み込みます。</span><span class="sxs-lookup"><span data-stu-id="334c3-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="334c3-106">これにより、パフォーマンスが向上するだけでなく、キャッシュに保持されているデータと基になるデータ ストアのデータの間で整合性が維持されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="334c3-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="334c3-107">Context and problem</span></span>

<span data-ttu-id="334c3-108">アプリケーションは、キャッシュを使用して、データ ストアに保持されている情報への繰り返されるアクセスを向上させています。</span><span class="sxs-lookup"><span data-stu-id="334c3-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="334c3-109">ただし、キャッシュ データがデータ ストアのデータと常に完全に一致することを期待するのは現実的ではありません。</span><span class="sxs-lookup"><span data-stu-id="334c3-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="334c3-110">アプリケーションでは、キャッシュのデータをできるだけ最新の状態に維持できるようにするための戦略を実装する必要がありますが、キャッシュのデータが古くなったときに発生する状況を検出して対処することもできます。</span><span class="sxs-lookup"><span data-stu-id="334c3-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="334c3-111">解決策</span><span class="sxs-lookup"><span data-stu-id="334c3-111">Solution</span></span>

<span data-ttu-id="334c3-112">多くの市販のキャッシュ システムで、リード スルーおよびライト スルー/ライト ビハインド操作が提供されています。</span><span class="sxs-lookup"><span data-stu-id="334c3-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="334c3-113">これらのシステムでは、アプリケーションはキャッシュを参照してデータを取得します。</span><span class="sxs-lookup"><span data-stu-id="334c3-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="334c3-114">データがキャッシュにない場合は、データ ストアから取得され、キャッシュに追加されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="334c3-115">キャッシュに保持されているデータに対する変更は、データ ストアにも自動的にライトバックされます。</span><span class="sxs-lookup"><span data-stu-id="334c3-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="334c3-116">キャッシュがこの機能を備えていない場合、キャッシュを使用してデータを維持するアプリケーションがその役割を担います。</span><span class="sxs-lookup"><span data-stu-id="334c3-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="334c3-117">アプリケーションは、キャッシュ アサイド戦略を実装することで、リード スルー キャッシュの機能をエミュレートできます。</span><span class="sxs-lookup"><span data-stu-id="334c3-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="334c3-118">この戦略では、オンデマンドでデータをキャッシュに読み込みます。</span><span class="sxs-lookup"><span data-stu-id="334c3-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="334c3-119">次の図は、キャッシュ アサイド パターンを使用してデータをキャッシュに格納する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="334c3-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![キャッシュ アサイド パターンを使用したキャッシュへのデータの格納](./_images/cache-aside-diagram.png)


<span data-ttu-id="334c3-121">アプリケーションは、情報を更新したら、データ ストアに変更を加え、キャッシュ内の対応する項目を無効にすることによって、ライト スルー戦略に従うことができます。</span><span class="sxs-lookup"><span data-stu-id="334c3-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="334c3-122">その項目が次回必要になったときは、キャッシュ アサイド戦略を使用することで、データ ストアから最新のデータが取得され、キャッシュに再度追加されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="334c3-123">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="334c3-123">Issues and considerations</span></span>

<span data-ttu-id="334c3-124">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="334c3-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="334c3-125">**キャッシュ データの有効期間**: </span><span class="sxs-lookup"><span data-stu-id="334c3-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="334c3-126">多くのキャッシュで、指定された期間アクセスされなかったデータを、無効にしてキャッシュから削除する有効期限ポリシーが実装されています。</span><span class="sxs-lookup"><span data-stu-id="334c3-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="334c3-127">キャッシュ アサイドを効果的に使用するには、有効期限ポリシーが、データを使用するアプリケーションのアクセス パターンに適合していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="334c3-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="334c3-128">有効期限が短すぎると、アプリケーションが常にデータ ストアからデータを取得してキャッシュに追加する可能性があるので、有効期限をあまり短くしないでください。</span><span class="sxs-lookup"><span data-stu-id="334c3-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="334c3-129">同様に、キャッシュ データが古くなる可能性が高いので、有効期限をあまり長くしないでください。</span><span class="sxs-lookup"><span data-stu-id="334c3-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="334c3-130">キャッシュは、比較的静的なデータまたは頻繁に読み取られるデータに最も効果的であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="334c3-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="334c3-131">**データの削除**: </span><span class="sxs-lookup"><span data-stu-id="334c3-131">**Evicting data**.</span></span> <span data-ttu-id="334c3-132">ほとんどのキャッシュは、データのソースであるデータ ストアに比べてサイズが限られており、必要に応じてデータが削除されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="334c3-133">ほとんどのキャッシュでは、削除する項目を選択するために最低使用頻度ポリシーを採用していますが、これはカスタマイズ可能です。</span><span class="sxs-lookup"><span data-stu-id="334c3-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="334c3-134">キャッシュのコスト効率を確保するために、キャッシュのグローバルな有効期限プロパティと他のプロパティ、および各キャッシュ項目の有効期限プロパティを構成します。</span><span class="sxs-lookup"><span data-stu-id="334c3-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="334c3-135">キャッシュ内のすべての項目にグローバルな削除ポリシーを適用することが必ずしも適切であるとは限りません。</span><span class="sxs-lookup"><span data-stu-id="334c3-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="334c3-136">たとえば、データ ストアからの取得に非常にコストがかかるキャッシュ項目がある場合、アクセス頻度は高くても低コストの項目と引き換えに、その項目をキャッシュに保持する方が有益な場合があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="334c3-137">**キャッシュの準備**: </span><span class="sxs-lookup"><span data-stu-id="334c3-137">**Priming the cache**.</span></span> <span data-ttu-id="334c3-138">多くのソリューションでは、アプリケーションが起動処理の一環として必要とする可能性の高いデータをキャッシュに事前に配置します。</span><span class="sxs-lookup"><span data-stu-id="334c3-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="334c3-139">キャッシュ アサイド パターンは、このようなデータの一部が期限切れになったり、削除されたりした場合にも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="334c3-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="334c3-140">**整合性**: </span><span class="sxs-lookup"><span data-stu-id="334c3-140">**Consistency**.</span></span> <span data-ttu-id="334c3-141">キャッシュ アサイド パターンを実装しても、データ ストアとキャッシュの間での整合性が保証されるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="334c3-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="334c3-142">データ ストア内の項目は、外部プロセスによって常に変更される可能性があります。この変更は、その項目が次回読み込まれるまでキャッシュに反映されないことがあります。</span><span class="sxs-lookup"><span data-stu-id="334c3-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="334c3-143">データ ストア間でデータをレプリケートするシステムでは、同期が頻繁に発生すると、この問題が深刻化する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="334c3-144">**ローカル (メモリ内) キャッシュ**: </span><span class="sxs-lookup"><span data-stu-id="334c3-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="334c3-145">キャッシュがアプリケーション インスタンスに対してローカルであり、メモリ内に格納されている場合があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="334c3-146">アプリケーションが同じデータに繰り返しアクセスする場合は、この環境でキャッシュ アサイドが役立ちます。</span><span class="sxs-lookup"><span data-stu-id="334c3-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="334c3-147">ただし、ローカル キャッシュはプライベートであるため、さまざまなアプリケーション インスタンスがそれぞれ同じキャッシュ データのコピーを持つことができます。</span><span class="sxs-lookup"><span data-stu-id="334c3-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="334c3-148">このデータはキャッシュ間での整合性がすぐに失われる可能性があるため、プライベート キャッシュに保持されているデータを期限切れにし、より頻繁に更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="334c3-149">これらのシナリオでは、共有または分散キャッシュ メカニズムの使用方法を調べることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="334c3-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="334c3-150">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="334c3-150">When to use this pattern</span></span>

<span data-ttu-id="334c3-151">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="334c3-151">Use this pattern when:</span></span>

- <span data-ttu-id="334c3-152">キャッシュが、ネイティブのリード スルーおよびライト スルー操作を提供していない場合。</span><span class="sxs-lookup"><span data-stu-id="334c3-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="334c3-153">リソースの需要は予測できません。</span><span class="sxs-lookup"><span data-stu-id="334c3-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="334c3-154">このパターンを使用すると、アプリケーションはオンデマンドでデータを読み込むことができます。</span><span class="sxs-lookup"><span data-stu-id="334c3-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="334c3-155">アプリケーションに必要なデータは事前に想定されません。</span><span class="sxs-lookup"><span data-stu-id="334c3-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="334c3-156">このパターンが適していないと考えられる状況は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="334c3-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="334c3-157">キャッシュされたデータ セットが静的な場合。</span><span class="sxs-lookup"><span data-stu-id="334c3-157">When the cached data set is static.</span></span> <span data-ttu-id="334c3-158">データが使用可能なキャッシュ領域に収まる場合は、起動時にデータをキャッシュに配置し、データが期限切れになるのを防ぐポリシーを適用します。</span><span class="sxs-lookup"><span data-stu-id="334c3-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="334c3-159">Web ファームでホストされている Web アプリケーションでセッション状態情報をキャッシュする場合。</span><span class="sxs-lookup"><span data-stu-id="334c3-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="334c3-160">この環境では、クライアント/サーバー アフィニティに基づく依存関係の導入を避ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="334c3-161">例</span><span class="sxs-lookup"><span data-stu-id="334c3-161">Example</span></span>

<span data-ttu-id="334c3-162">Microsoft Azure では、Azure Redis Cache を使用して、アプリケーションの複数のインスタンスで共有できる分散キャッシュを作成できます。</span><span class="sxs-lookup"><span data-stu-id="334c3-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="334c3-163">次のコード例では、.NET 向けに作成された Redis クライアント ライブラリである、[StackExchange.Redis] クライアントを使用しています。</span><span class="sxs-lookup"><span data-stu-id="334c3-163">This following code examples use the [StackExchange.Redis] client, which is a Redis client library written for .NET.</span></span> <span data-ttu-id="334c3-164">Azure Redis Cache インスタンスに接続するには、静的 `ConnectionMultiplexer.Connect` メソッドを呼び出し、接続文字列を渡します。</span><span class="sxs-lookup"><span data-stu-id="334c3-164">To connect to an Azure Redis Cache instance, call the static `ConnectionMultiplexer.Connect` method and pass in the connection string.</span></span> <span data-ttu-id="334c3-165">このメソッドは、接続を表す `ConnectionMultiplexer` を返します。</span><span class="sxs-lookup"><span data-stu-id="334c3-165">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="334c3-166">アプリケーション内の `ConnectionMultiplexer` インスタンスを共有する方法の 1 つに、次の例のように、接続されたインスタンスを返す静的プロパティを設定する方法があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-166">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="334c3-167">この方法では、接続された 1 つのインスタンスだけがスレッドセーフな方法で初期化されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-167">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="334c3-168">次のコード例の `GetMyEntityAsync` メソッドは、キャッシュ アサイド パターンの実装を示しています。</span><span class="sxs-lookup"><span data-stu-id="334c3-168">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern.</span></span> <span data-ttu-id="334c3-169">このメソッドは、リード スルー手法を使用してキャッシュからオブジェクトを取得します。</span><span class="sxs-lookup"><span data-stu-id="334c3-169">This method retrieves an object from the cache using the read-through approach.</span></span>

<span data-ttu-id="334c3-170">オブジェクトは、整数 ID をキーとして使用して識別されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-170">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="334c3-171">`GetMyEntityAsync` メソッドは、このキーを持つ項目をキャッシュから取得することを試みます。</span><span class="sxs-lookup"><span data-stu-id="334c3-171">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="334c3-172">一致する項目が見つかった場合は、それが返されます。</span><span class="sxs-lookup"><span data-stu-id="334c3-172">If a matching item is found, it's returned.</span></span> <span data-ttu-id="334c3-173">キャッシュに一致するものがない場合、`GetMyEntityAsync` メソッドはデータ ストアからオブジェクトを取得し、キャッシュに追加してから返します。</span><span class="sxs-lookup"><span data-stu-id="334c3-173">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="334c3-174">データ ストアから実際にデータを読み取るコードはデータ ストアによって異なるため、ここでは示していません。</span><span class="sxs-lookup"><span data-stu-id="334c3-174">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="334c3-175">キャッシュ項目は、他の場所で更新された場合に古くならないように有効期限が構成されています。</span><span class="sxs-lookup"><span data-stu-id="334c3-175">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  <span data-ttu-id="334c3-176">各例では、Redis Cache を使用してストアにアクセスし、キャッシュから情報を取得します。</span><span class="sxs-lookup"><span data-stu-id="334c3-176">The examples use Redis Cache to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="334c3-177">詳細については、[Microsoft Azure Redis Cache の使用方法](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache)に関する記事、および「[Redis Cache で Web アプリを作成する方法](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="334c3-177">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="334c3-178">次に示す `UpdateEntityAsync` メソッドは、アプリケーションによって値が変更されたときにキャッシュ内のオブジェクトを無効にする方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="334c3-178">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="334c3-179">このコードでは、元のデータ ストアを更新した後、キャッシュ項目をキャッシュから削除します。</span><span class="sxs-lookup"><span data-stu-id="334c3-179">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="334c3-180">ステップの順序が重要です。</span><span class="sxs-lookup"><span data-stu-id="334c3-180">The order of the steps is important.</span></span> <span data-ttu-id="334c3-181">項目をキャッシュから削除する "*前に*" データ ストアを更新します。</span><span class="sxs-lookup"><span data-stu-id="334c3-181">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="334c3-182">キャッシュ項目を先に削除すると、データ ストアが更新されるまでの短い時間に、クライアントがその項目を取得する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="334c3-182">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="334c3-183">これにより、キャッシュ ミスが発生し (キャッシュから項目が削除されたため)、以前のバージョンの項目がデータ ストアから取得され、キャッシュに再度追加されることになります。</span><span class="sxs-lookup"><span data-stu-id="334c3-183">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="334c3-184">その結果、キャッシュ データが古くなります。</span><span class="sxs-lookup"><span data-stu-id="334c3-184">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="334c3-185">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="334c3-185">Related guidance</span></span> 

<span data-ttu-id="334c3-186">このパターンを実装するときは、次の情報を参考にしてください。</span><span class="sxs-lookup"><span data-stu-id="334c3-186">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="334c3-187">[キャッシュ ガイダンス](https://docs.microsoft.com/azure/architecture/best-practices/caching)。</span><span class="sxs-lookup"><span data-stu-id="334c3-187">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="334c3-188">クラウド ソリューションでデータをキャッシュする方法と、キャッシュを実装する際に考慮すべき問題に関する追加情報を提供します。</span><span class="sxs-lookup"><span data-stu-id="334c3-188">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="334c3-189">[Data consistency primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)。</span><span class="sxs-lookup"><span data-stu-id="334c3-189">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="334c3-190">通常、クラウド アプリケーションは、複数のデータ ストアに分散したデータを使用します。</span><span class="sxs-lookup"><span data-stu-id="334c3-190">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="334c3-191">この環境でのデータ整合性の管理と維持は、システム (特に、発生する可能性のあるコンカレンシーと可用性の問題) の重要な側面です。</span><span class="sxs-lookup"><span data-stu-id="334c3-191">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="334c3-192">この入門書では、分散データの整合性に関する問題について説明し、アプリケーションがデータの可用性を維持するために最終的な整合性を実装する方法の概要を説明します。</span><span class="sxs-lookup"><span data-stu-id="334c3-192">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>


[StackExchange.Redis]: https://github.com/StackExchange/StackExchange.Redis
