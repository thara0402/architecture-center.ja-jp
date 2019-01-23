---
title: 余分なフェッチのアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: 業務に使用するデータを必要以上に取得することが、不要な I/O オーバーヘッドを招き、応答性を低下させる場合があります。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c1172531b332854a6d4940c072b61cb3f6bcd7ba
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488190"
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="c9930-103">余分なフェッチのアンチパターン</span><span class="sxs-lookup"><span data-stu-id="c9930-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="c9930-104">業務に使用するデータを必要以上に取得することが、不要な I/O オーバーヘッドを招き、応答性を低下させる場合があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="c9930-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="c9930-105">Problem description</span></span>

<span data-ttu-id="c9930-106">アプリケーションが I/O 要求数を最小限に抑えようと、必要になる "*可能性のある*" データをすべて取得した場合に、このアンチパターンに陥ることがあります。</span><span class="sxs-lookup"><span data-stu-id="c9930-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="c9930-107">これはえてして、[頻度の高い I/O][chatty-io] のアンチパターンを過度に意識したことによって生じます。</span><span class="sxs-lookup"><span data-stu-id="c9930-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="c9930-108">たとえば、データベースから製品ごとの詳しい情報をアプリケーションでフェッチすることがあります。</span><span class="sxs-lookup"><span data-stu-id="c9930-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="c9930-109">しかしユーザーが必要としているのは一部の情報だけで (顧客には関係のない情報が含まれている場合があります)、一度に "*すべて*" の製品を見る必要はないかもしれません。</span><span class="sxs-lookup"><span data-stu-id="c9930-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="c9930-110">ユーザーがカタログ全体を閲覧することになる場合でも、ページ単位で結果を表示すれば実用上は問題ないでしょう (20 件ずつ表示するなど)。</span><span class="sxs-lookup"><span data-stu-id="c9930-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="c9930-111">この問題の原因としてもう 1 つ考えられることは、実践している手法が、プログラミングまたはデザインの観点から好ましくない、ということです。</span><span class="sxs-lookup"><span data-stu-id="c9930-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="c9930-112">たとえば、次のコードは、Entity Framework を使用して製品ごとの詳しい情報をすべてフェッチします。</span><span class="sxs-lookup"><span data-stu-id="c9930-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="c9930-113">その後、その結果をフィルター処理して一部のフィールドだけを返し、残りは破棄されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="c9930-114">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="c9930-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="c9930-115">次の例では、データベース側でも実行できる集計を、アプリケーション側でデータを取得して実行しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="c9930-116">アプリケーションは、販売されたすべての注文のレコードを全件取得し、それらの合計を計算することによって、売上の合計を計算します。</span><span class="sxs-lookup"><span data-stu-id="c9930-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="c9930-117">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="c9930-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="c9930-118">次に、少し気付きにくい問題の例を紹介します。Entity Framework の LINQ to Entities の振る舞いに起因するものです。</span><span class="sxs-lookup"><span data-stu-id="c9930-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span>

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="c9930-119">このアプリケーションは、`SellStartDate` の日付から 1 週間以上経過している製品を探そうと試みます。</span><span class="sxs-lookup"><span data-stu-id="c9930-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="c9930-120">ほとんどの場合、`where` 句は、LINQ to Entities によって、データベース側で実行される SQL ステートメントに変換されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="c9930-121">しかしこのケースでは、LINQ to Entities が `AddDays` メソッドを SQL にマッピングできません。</span><span class="sxs-lookup"><span data-stu-id="c9930-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="c9930-122">`Product` テーブルからすべての行が返され、その結果がメモリ内でフィルター処理されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span>

<span data-ttu-id="c9930-123">問題は、`AsEnumerable` を呼び出す部分に潜んでいます。</span><span class="sxs-lookup"><span data-stu-id="c9930-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="c9930-124">このメソッドは、その結果を `IEnumerable` インターフェイスに変換します。</span><span class="sxs-lookup"><span data-stu-id="c9930-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="c9930-125">`IEnumerable` はフィルター処理をサポートしていますが、フィルター処理はデータベース側ではなく "*クライアント*" 側で実行されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="c9930-126">LINQ to Entities でフィルター処理をデータ ソース側に引き渡すには既定の `IQueryable` を使用します。</span><span class="sxs-lookup"><span data-stu-id="c9930-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="c9930-127">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="c9930-127">How to fix the problem</span></span>

<span data-ttu-id="c9930-128">すぐに古くなったり破棄されたりするようなデータを大量にフェッチすることは避け、実行する処理に必要なデータだけをフェッチするようにしましょう。</span><span class="sxs-lookup"><span data-stu-id="c9930-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span>

<span data-ttu-id="c9930-129">テーブルからすべての列を取得してからフィルター処理するのではなく、必要な列だけをデータベースから選択します。</span><span class="sxs-lookup"><span data-stu-id="c9930-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="c9930-130">同様に、集計はデータベース内で実行し、アプリケーションのメモリ内で実行することは避けます。</span><span class="sxs-lookup"><span data-stu-id="c9930-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="c9930-131">Entity Framework の使用時、LINQ クエリは `IEnumerable` ではなく、`IQueryable`インターフェイスを使って解決されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="c9930-132">データ ソースにマッピング可能な関数だけを使用するように自分でクエリを調整することも、場合によっては必要となります。</span><span class="sxs-lookup"><span data-stu-id="c9930-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="c9930-133">前述の例は、クエリから `AddDays` メソッドを削除し、フィルター処理がデータベース側で実行されるようにすることでリファクタリングできます。</span><span class="sxs-lookup"><span data-stu-id="c9930-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out.
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="c9930-134">考慮事項</span><span class="sxs-lookup"><span data-stu-id="c9930-134">Considerations</span></span>

- <span data-ttu-id="c9930-135">一部のケースでは、データを行方向にパーティション分割することでパフォーマンスを高めることができます。</span><span class="sxs-lookup"><span data-stu-id="c9930-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="c9930-136">データが持つさまざまな属性に複数の異なる操作からアクセスする場合、行方向のパーティション分割によって競合を減らすことができます。</span><span class="sxs-lookup"><span data-stu-id="c9930-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="c9930-137">多くの場合ほとんどの操作は、データのごく一部分に対して実行すれば済みます。このように負荷を分散させることでパフォーマンスを向上できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="c9930-138">[データのパーティション分割][data-partitioning]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="c9930-139">あえてクエリに特定の検索条件を設けず大量のデータが返される可能性を考慮しなければならない場合は、改ページ位置の自動修正を実装し、一度にフェッチするエンティティの件数を限定するようにします。</span><span class="sxs-lookup"><span data-stu-id="c9930-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="c9930-140">たとえばユーザーがブラウザーで製品カタログを閲覧する場合、1 ページずつ結果を表示することができます。</span><span class="sxs-lookup"><span data-stu-id="c9930-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="c9930-141">データ ストアに組み込まれている機能を可能な限り活用します。</span><span class="sxs-lookup"><span data-stu-id="c9930-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="c9930-142">たとえば SQL データベースには通常、集計関数が備わっています。</span><span class="sxs-lookup"><span data-stu-id="c9930-142">For example, SQL databases typically provide aggregate functions.</span></span>

- <span data-ttu-id="c9930-143">ご使用のデータ ストアが特定の関数 (集計など) をサポートしない場合、計算結果をどこか別の場所に保存しておき、レコードが追加されたり変更されたりしたときに値を更新することを検討してください。そうすれば値が必要になるたびにアプリケーション側で再計算する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="c9930-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="c9930-144">要求によって取得されるフィールドの数が多いと感じる場合は、ソース コードをよく調べて、本当にそれらのフィールドが全部必要であるかどうかを確かめます。</span><span class="sxs-lookup"><span data-stu-id="c9930-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="c9930-145">`SELECT *` クエリの設計に改善の余地がある可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span>

- <span data-ttu-id="c9930-146">同様に、要求の結果として大量のエンティティが取得される場合、アプリケーション側でデータが正しくフィルター処理されていない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="c9930-147">このようなエンティティが本当に全部必要であるかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="c9930-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="c9930-148">可能であれば、SQL の `WHERE` 句を使用するなど、データベース側のフィルター処理を使用します。</span><span class="sxs-lookup"><span data-stu-id="c9930-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span>

- <span data-ttu-id="c9930-149">処理の負荷をデータベース側に移すことが常に最善の方法とは限りません。</span><span class="sxs-lookup"><span data-stu-id="c9930-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="c9930-150">この手法を用いるのは、データベースがそのように設計されている、または最適化されている場合に限定してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="c9930-151">ほとんどのデータベース システムは、特定の関数を想定して高度に最適化されていますが、汎用のアプリケーション エンジンの役割を果たすような設計にはなっていません。</span><span class="sxs-lookup"><span data-stu-id="c9930-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="c9930-152">詳細については、「[ビジー状態のデータベースのアンチパターン][BusyDatabase]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="c9930-153">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="c9930-153">How to detect the problem</span></span>

<span data-ttu-id="c9930-154">余分なフェッチは、待ち時間の長さやスループットの低さなどの症状となって現れます。</span><span class="sxs-lookup"><span data-stu-id="c9930-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="c9930-155">データ ストアからデータを取得する場合、競合の増加が関係している可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="c9930-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="c9930-156">エンド ユーザーからは、応答時間の増加やサービスのタイムアウトに起因した障害が報告されることになります。こうした障害が発生した場合、HTTP 500 (内部サーバー) エラーまたは HTTP 503 (サービスを利用できません) エラーが返されます。</span><span class="sxs-lookup"><span data-stu-id="c9930-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="c9930-157">Web サーバーのイベント ログをよく調べてください。エラーの原因や状況についてさらに詳しい情報が記録されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="c9930-158">このアンチパターンの症状と収集されるテレメトリは、[モノリシックな永続化のアンチパターン][MonolithicPersistence]と酷似している場合があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span>

<span data-ttu-id="c9930-159">原因を特定しやすくするために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="c9930-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="c9930-160">ロード テストやプロセス監視など、インストルメンテーション データの収集手法を用いて、低速なワークロードまたはトランザクションを特定します。</span><span class="sxs-lookup"><span data-stu-id="c9930-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="c9930-161">システムが示す動作パターンを観察します。</span><span class="sxs-lookup"><span data-stu-id="c9930-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="c9930-162">毎秒トランザクション数やユーザー数の観点で目立った制限はありますか。</span><span class="sxs-lookup"><span data-stu-id="c9930-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="c9930-163">低速なワークロードの事例と動作パターンとの相関関係を明らかにします。</span><span class="sxs-lookup"><span data-stu-id="c9930-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="c9930-164">使用中のデータ ストアを特定します。</span><span class="sxs-lookup"><span data-stu-id="c9930-164">Identify the data stores being used.</span></span> <span data-ttu-id="c9930-165">データ ソースごとに、より深いレベルのテレメトリを実行し、処理の動作を観察します。</span><span class="sxs-lookup"><span data-stu-id="c9930-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
5. <span data-ttu-id="c9930-166">そうしたデータ ソースを参照するクエリのうち、実行速度が遅いものがあれば特定します。</span><span class="sxs-lookup"><span data-stu-id="c9930-166">Identify any slow-running queries that reference these data sources.</span></span>
6. <span data-ttu-id="c9930-167">実行速度が遅いクエリについてリソースごとに分析し、データがどのように使用され、消費されているかを突き止めます。</span><span class="sxs-lookup"><span data-stu-id="c9930-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="c9930-168">次のような症状がないか調べます。</span><span class="sxs-lookup"><span data-stu-id="c9930-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="c9930-169">同じリソース (データ ストア) に対して大量の I/O 要求を頻繁に実行している。</span><span class="sxs-lookup"><span data-stu-id="c9930-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="c9930-170">共有リソース (データ ストア) で競合が生じている。</span><span class="sxs-lookup"><span data-stu-id="c9930-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="c9930-171">特定の操作がネットワーク経由で大量のデータを頻繁に受信している。</span><span class="sxs-lookup"><span data-stu-id="c9930-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="c9930-172">アプリケーションやサービスが I/O の完了に費やす待ち時間が著しく長い。</span><span class="sxs-lookup"><span data-stu-id="c9930-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="c9930-173">診断の例</span><span class="sxs-lookup"><span data-stu-id="c9930-173">Example diagnosis</span></span>

<span data-ttu-id="c9930-174">以降のセクションでは、上記の手順を前述の例に適用しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="c9930-175">低速のワークロードを特定する</span><span class="sxs-lookup"><span data-stu-id="c9930-175">Identify slow workloads</span></span>

<span data-ttu-id="c9930-176">このグラフは、先ほど紹介した `GetAllFieldsAsync` メソッドを最大 400 人のユーザーが同時に実行するようすをシミュレートしたロード テストからのパフォーマンス測定結果を示したものです。</span><span class="sxs-lookup"><span data-stu-id="c9930-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="c9930-177">負荷が増えるにつれ少しずつスループットが低下しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="c9930-178">ワークロードが増えるにつれて平均応答時間が上昇します。</span><span class="sxs-lookup"><span data-stu-id="c9930-178">Average response time goes up as the workload increases.</span></span>

![GetAllFieldsAsync メソッドのロード テストの結果][Load-Test-Results-Client-Side1]

<span data-ttu-id="c9930-180">`AggregateOnClientAsync` 操作のロード テストも同様のパターンを示しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="c9930-181">当然、要求のボリュームは不変です。</span><span class="sxs-lookup"><span data-stu-id="c9930-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="c9930-182">平均応答時間は、前のグラフと比べるとなだらかではありますが、ワークロードと共に上昇します。</span><span class="sxs-lookup"><span data-stu-id="c9930-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![AggregateOnClientAsync メソッドのロード テストの結果][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="c9930-184">低速なワークロードと動作パターンとの相関関係を明らかにする</span><span class="sxs-lookup"><span data-stu-id="c9930-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="c9930-185">定期的に使用率が高くなる時間帯とパフォーマンス低下との相関関係から問題の領域が明らかになる場合があります。</span><span class="sxs-lookup"><span data-stu-id="c9930-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="c9930-186">実行速度低下が疑われる機能のパフォーマンス プロファイルを綿密に調査し、先ほど実行したロード テストと合致しているかどうかを確認してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="c9930-187">同じ機能を対象に、ユーザー数を段階的に増やしていきながらロード テストを実施し、パフォーマンスが著しく低下したり、完全にエラーになったりするポイントを見極めます。</span><span class="sxs-lookup"><span data-stu-id="c9930-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="c9930-188">そのポイントが、皆さんの想定した実際の使用状況の範囲内で生じるのであれば、その機能がどのように実装されているかを詳しく調査してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="c9930-189">システムに負荷のかかったタイミングで実行されることがない場合や急を要さない場合、他の重要な操作のパフォーマンスに悪影響を及ぼさない場合には、低速な処理が必ずしも問題となるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="c9930-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="c9930-190">たとえば毎月の運用統計データの生成は、時間のかかる処理かもしれませんが、通常はバッチ処理として、低優先度のジョブとして実行されていることでしょう。</span><span class="sxs-lookup"><span data-stu-id="c9930-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="c9930-191">一方、顧客が製品カタログを問い合わせるために行うクエリは、重要度のきわめて高い業務です。</span><span class="sxs-lookup"><span data-stu-id="c9930-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="c9930-192">このような重要な操作から生成されるテレメトリに注目して、使用率が上昇する時間帯におけるパフォーマンスの変化を調べましょう。</span><span class="sxs-lookup"><span data-stu-id="c9930-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="c9930-193">低速なワークロードのデータ ソースを特定する</span><span class="sxs-lookup"><span data-stu-id="c9930-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="c9930-194">サービスのパフォーマンス低下の原因がデータの取得方法にあることが疑われる場合、アプリケーションとそのリポジトリとの対話を調査します。</span><span class="sxs-lookup"><span data-stu-id="c9930-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="c9930-195">ライブ システムを監視して、パフォーマンスの低い時間帯にどのソースがアクセスの対象になっているかを把握してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span>

<span data-ttu-id="c9930-196">データ ソースごとに、次の情報を収集するようにシステムをインストルメント化します。</span><span class="sxs-lookup"><span data-stu-id="c9930-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="c9930-197">それぞれのデータ ストアのアクセス頻度。</span><span class="sxs-lookup"><span data-stu-id="c9930-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="c9930-198">データ ストアが送受するデータ量。</span><span class="sxs-lookup"><span data-stu-id="c9930-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="c9930-199">該当する操作の (特に要求の待ち時間が発生する) タイミング。</span><span class="sxs-lookup"><span data-stu-id="c9930-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="c9930-200">通常負荷の下で各データ ストアにアクセスしているときに発生するエラーの特性と割合。</span><span class="sxs-lookup"><span data-stu-id="c9930-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="c9930-201">この情報を、アプリケーションからクライアントに返されるデータの量と比較してみましょう。</span><span class="sxs-lookup"><span data-stu-id="c9930-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="c9930-202">データ ストアから返されるデータの量とクライアントに返されるデータの量の比率を追跡します。</span><span class="sxs-lookup"><span data-stu-id="c9930-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="c9930-203">その差が大きい場合、不要なデータをアプリケーションでフェッチしていないかどうかを調査します。</span><span class="sxs-lookup"><span data-stu-id="c9930-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="c9930-204">このデータは、ライブ システムを観察し、ユーザーの要求ごとのライフサイクルをトレースすることによって収集することができます。または、一連の人工的ワークロードをモデル化し、それをテスト システムに対して実行してもかまいません。</span><span class="sxs-lookup"><span data-stu-id="c9930-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="c9930-205">次のグラフは、`GetAllFieldsAsync` メソッドのロード テスト中に [New Relic APM][new-relic] を使用して収集されたテレメトリを示したものです。</span><span class="sxs-lookup"><span data-stu-id="c9930-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="c9930-206">データベースから受信したデータの量と、それに対応する HTTP 応答の量の違いに注目してください。</span><span class="sxs-lookup"><span data-stu-id="c9930-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![`GetAllFieldsAsync` メソッドのテレメトリ][TelemetryAllFields]

<span data-ttu-id="c9930-208">それぞれの要求でデータベースから返されたのは 80,503 バイトですが、そのうちクライアントへの応答に含まれていたのはたった 19,855 バイトでした。データベースの応答の約 25% です。</span><span class="sxs-lookup"><span data-stu-id="c9930-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="c9930-209">クライアントに返されるデータのサイズは、その形式によって変わることがあります。</span><span class="sxs-lookup"><span data-stu-id="c9930-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="c9930-210">このロード テストに関して言えば、クライアントが要求したのは JSON データです。</span><span class="sxs-lookup"><span data-stu-id="c9930-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="c9930-211">ここでは紹介していませんが、別途 XML を使用したテストでは応答サイズが 35,655 バイトと、データベースの応答の 44% のサイズになりました。</span><span class="sxs-lookup"><span data-stu-id="c9930-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="c9930-212">`AggregateOnClientAsync` メソッドのロード テストは、さらに極端な結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="c9930-213">このケースの各テストで実行したクエリがデータベースから取得したデータは 280 KB を超えましたが、JSON 形式の応答はわずか 14 バイトでした。</span><span class="sxs-lookup"><span data-stu-id="c9930-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="c9930-214">これほど大きな差があるのは、このメソッドが、集計結果を大量のデータから計算しているためです。</span><span class="sxs-lookup"><span data-stu-id="c9930-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![`AggregateOnClientAsync` メソッドのテレメトリ][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="c9930-216">実行速度が遅いクエリを特定して分析する</span><span class="sxs-lookup"><span data-stu-id="c9930-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="c9930-217">リソース消費が最も激しく実行に著しい時間がかかるデータベース クエリを探します。</span><span class="sxs-lookup"><span data-stu-id="c9930-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="c9930-218">さまざまなデータベース操作の開始時刻と完了時刻は、インストルメンテーションを追加することで見極めることができます。</span><span class="sxs-lookup"><span data-stu-id="c9930-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="c9930-219">多くのデータ ストアは、クエリがどのように実行され、どのように最適化されるかについての詳細な情報を入手できるようになっています。</span><span class="sxs-lookup"><span data-stu-id="c9930-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="c9930-220">たとえば Azure SQL Database 管理ポータルの [クエリ パフォーマンス] ウィンドウでは、クエリを選択し、実行時の詳しいパフォーマンス情報を確認することができます。</span><span class="sxs-lookup"><span data-stu-id="c9930-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="c9930-221">以下に示したのは、`GetAllFieldsAsync` 操作によって生成されるクエリです。</span><span class="sxs-lookup"><span data-stu-id="c9930-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Microsoft Azure SQL Database 管理ポータルの [クエリの詳細] ウィンドウ][QueryDetails]

## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="c9930-223">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="c9930-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="c9930-224">データベース側で SELECT ステートメントを使用するように `GetRequiredFieldsAsync` メソッドを変更した後、ロード テストでは、次の結果が示されました。</span><span class="sxs-lookup"><span data-stu-id="c9930-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![GetRequiredFieldsAsync メソッドのロード テストの結果][Load-Test-Results-Database-Side1]

<span data-ttu-id="c9930-226">このロード テストで使用したデプロイは前回と同じです。また、ワークロードをシミュレートするために使用した同時ユーザー数も、前回と同じ 400 ユーザーです。</span><span class="sxs-lookup"><span data-stu-id="c9930-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="c9930-227">このグラフを見ると、待ち時間が大幅に短縮されていることが確認できます。</span><span class="sxs-lookup"><span data-stu-id="c9930-227">The graph shows much lower latency.</span></span> <span data-ttu-id="c9930-228">応答時間は負荷と共に上昇しますが、前回の 4 秒に比べて、最大でも 1.3 秒程度に留まっています。</span><span class="sxs-lookup"><span data-stu-id="c9930-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="c9930-229">スループットも向上し、毎秒要求数が前回の 100 に対し、350 にまで増加しています。</span><span class="sxs-lookup"><span data-stu-id="c9930-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="c9930-230">データベースから取得されたデータの量は、HTTP 応答メッセージのサイズとほぼ一致するようになりました。</span><span class="sxs-lookup"><span data-stu-id="c9930-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![`GetRequiredFieldsAsync` メソッドのテレメトリ][TelemetryRequiredFields]

<span data-ttu-id="c9930-232">`AggregateOnDatabaseAsync` メソッドのロード テストから生成された結果は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="c9930-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![AggregateOnDatabaseAsync メソッドのロード テストの結果][Load-Test-Results-Database-Side2]

<span data-ttu-id="c9930-234">平均応答時間は最小限に抑えられています。</span><span class="sxs-lookup"><span data-stu-id="c9930-234">The average response time is now minimal.</span></span> <span data-ttu-id="c9930-235">パフォーマンスが桁違いに向上しています。その主な要因は、データベースからの I/O が大幅に減ったことです。</span><span class="sxs-lookup"><span data-stu-id="c9930-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span>

<span data-ttu-id="c9930-236">以下は、`AggregateOnDatabaseAsync` メソッドに関して対応するテレメトリを示したものです。</span><span class="sxs-lookup"><span data-stu-id="c9930-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="c9930-237">データベースから取得されるデータの量は、トランザクションあたり 280 KB 超から 53 バイトへと大幅に削減されました。</span><span class="sxs-lookup"><span data-stu-id="c9930-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="c9930-238">その結果、1 分あたりの持続可能な最大要求数は、約 2,000 RPM から 25,000 RPM 超にまで増加しました。</span><span class="sxs-lookup"><span data-stu-id="c9930-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![`AggregateOnDatabaseAsync` メソッドのテレメトリ][TelemetryAggregateInDatabaseAsync]

## <a name="related-resources"></a><span data-ttu-id="c9930-240">関連リソース</span><span class="sxs-lookup"><span data-stu-id="c9930-240">Related resources</span></span>

- <span data-ttu-id="c9930-241">[ビジー状態のデータベースのアンチパターン][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="c9930-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="c9930-242">[頻度の高い I/O のアンチパターン][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="c9930-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="c9930-243">[データのパーティション分割のベスト プラクティス][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="c9930-243">[Data partitioning best practices][data-partitioning]</span></span>

[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
