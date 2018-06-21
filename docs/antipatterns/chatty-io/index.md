---
title: 頻度の高い I/O のアンチパターン
description: 大量の I/O 要求によってパフォーマンスと応答性が損なわれる場合があります。
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
ms.locfileid: "29477742"
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="38a21-103">頻度の高い I/O のアンチパターン</span><span class="sxs-lookup"><span data-stu-id="38a21-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="38a21-104">大量の I/O 要求の影響が累積して、パフォーマンスと応答性に著しい影響を与える場合があります。</span><span class="sxs-lookup"><span data-stu-id="38a21-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="38a21-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="38a21-105">Problem description</span></span>

<span data-ttu-id="38a21-106">ネットワーク呼び出しをはじめとする I/O 操作は、コンピューティング タスクと比べて本質的に低速です。</span><span class="sxs-lookup"><span data-stu-id="38a21-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="38a21-107">I/O 要求は 1 回ごとに著しいオーバーヘッドを伴うのが普通で、多数の I/O 操作の影響が累積することでシステムの処理能力が低下するおそれがあります。</span><span class="sxs-lookup"><span data-stu-id="38a21-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="38a21-108">以降、頻度の高い I/O の代表的な原因をいくつか取り上げます。</span><span class="sxs-lookup"><span data-stu-id="38a21-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="38a21-109">データベースに対する各レコードの読み取りと書き込みを個別の要求として実行する</span><span class="sxs-lookup"><span data-stu-id="38a21-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="38a21-110">以下の例では、製品のデータベースから読み取りを実行しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-110">The following example reads from a database of products.</span></span> <span data-ttu-id="38a21-111">テーブルは、`Product`、`ProductSubcategory`、`ProductPriceListHistory` の 3 つが存在します。</span><span class="sxs-lookup"><span data-stu-id="38a21-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="38a21-112">このコードは、次の一連のクエリを実行して、サブカテゴリに属しているすべての製品とその価格情報を取得します。</span><span class="sxs-lookup"><span data-stu-id="38a21-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="38a21-113">`ProductSubcategory` テーブルにサブカテゴリを照会します。</span><span class="sxs-lookup"><span data-stu-id="38a21-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="38a21-114">`Product` テーブルを照会して、そのサブカテゴリに属している製品をすべて検索します。</span><span class="sxs-lookup"><span data-stu-id="38a21-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="38a21-115">製品ごとの価格データを `ProductPriceListHistory` テーブルに照会します。</span><span class="sxs-lookup"><span data-stu-id="38a21-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="38a21-116">このアプリケーションは、[Entity Framework][ef] を使用してデータベースを照会します。</span><span class="sxs-lookup"><span data-stu-id="38a21-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="38a21-117">完全なサンプルは、[こちら][code-sample]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="38a21-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="38a21-118">この例では問題を明示的に示していますが、O/RM (オブジェクト関係マッピング) で暗黙的に子レコードが 1 件ずつフェッチされていると、問題に気付きにくいことがあります。</span><span class="sxs-lookup"><span data-stu-id="38a21-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="38a21-119">これは "N+1 問題" として知られています。</span><span class="sxs-lookup"><span data-stu-id="38a21-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="38a21-120">1 つの論理的操作を一連の HTTP 要求として実装する</span><span class="sxs-lookup"><span data-stu-id="38a21-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="38a21-121">これは多くの場合、開発者がオブジェクト指向パラダイムに従おうと、リモート オブジェクトをメモリ内のローカル オブジェクトであるかのように扱うことで起こります。</span><span class="sxs-lookup"><span data-stu-id="38a21-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="38a21-122">この場合、過剰なネットワーク ラウンド トリップが発生します。</span><span class="sxs-lookup"><span data-stu-id="38a21-122">This can result in too many network round trips.</span></span> <span data-ttu-id="38a21-123">たとえば次の Web API は、`User` オブジェクトの各プロパティを別々の HTTP GET メソッドで公開しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="38a21-124">この手法に関して技術的に間違っているところは何もありませんが、おそらく大半のクライアントは、`User` ごとに複数のプロパティを取得しなければならなくなるでしょう。次のようにクライアント コードを記述しなければなりません。</span><span class="sxs-lookup"><span data-stu-id="38a21-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="38a21-125">ディスク上のファイルの読み取りと書き込みを実行する</span><span class="sxs-lookup"><span data-stu-id="38a21-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="38a21-126">ファイル I/O は必ずファイルを開いて、適切なポイントに移動したうえで、データの読み取りまたは書き込みを行います。</span><span class="sxs-lookup"><span data-stu-id="38a21-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="38a21-127">その操作が完了したら、オペレーティング システムのリソースを節約するために、そのファイルは閉じることになるでしょう。</span><span class="sxs-lookup"><span data-stu-id="38a21-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="38a21-128">ファイルに対して少量の情報の読み取りと書き込みを絶えず繰り返すアプリケーションでは、著しい I/O オーバーヘッドが生じます。</span><span class="sxs-lookup"><span data-stu-id="38a21-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="38a21-129">少量の書き込み要求がファイルの断片化を招き、その後の I/O 操作がさらに低速化する場合もあります。</span><span class="sxs-lookup"><span data-stu-id="38a21-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="38a21-130">以下の例では、`FileStream` を使用して `Customer` オブジェクトをファイルに書き込んでいます。</span><span class="sxs-lookup"><span data-stu-id="38a21-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="38a21-131">`FileStream` を作成するとファイルが開き、破棄するとファイルが閉じます </span><span class="sxs-lookup"><span data-stu-id="38a21-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="38a21-132">(`FileStream` オブジェクトは `using` ステートメントによって自動的に破棄されます)。新しい顧客が追加されるたびにこのメソッドが繰り返し呼び出されると、I/O オーバーヘッドはすぐに累積していくことでしょう。</span><span class="sxs-lookup"><span data-stu-id="38a21-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="38a21-133">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="38a21-133">How to fix the problem</span></span>

<span data-ttu-id="38a21-134">I/O 要求数を減らすには、データをより大きなまとまりにして要求の回数を減らします。</span><span class="sxs-lookup"><span data-stu-id="38a21-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="38a21-135">小さなクエリを何度も繰り返すのではなく、1 回のクエリでデータベースからデータをフェッチするようにしましょう。</span><span class="sxs-lookup"><span data-stu-id="38a21-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="38a21-136">製品情報を取得する変更後のコードは以下のとおりです。</span><span class="sxs-lookup"><span data-stu-id="38a21-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="38a21-137">Web API については、REST の設計原則に従います。</span><span class="sxs-lookup"><span data-stu-id="38a21-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="38a21-138">先ほどの例を修正した Web API を次に示します。</span><span class="sxs-lookup"><span data-stu-id="38a21-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="38a21-139">プロパティごとに別々の GET メソッドが存在するのではなく、`User` を返す GET メソッドが 1 つだけ存在します。</span><span class="sxs-lookup"><span data-stu-id="38a21-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="38a21-140">これによって 1 回の要求で返される応答本文は大きくなりますが、各クライアントが API を呼び出す回数は少なくなるでしょう。</span><span class="sxs-lookup"><span data-stu-id="38a21-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="38a21-141">ファイル I/O に関しては、データをメモリにバッファー処理しておき、それを 1 回の操作でファイルに書き込むことを検討してください。</span><span class="sxs-lookup"><span data-stu-id="38a21-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="38a21-142">この方法により、ファイルの開閉を繰り返すことで生じるオーバーヘッドを削減し、ディスク上のファイルの断片化を軽減できます。</span><span class="sxs-lookup"><span data-stu-id="38a21-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="38a21-143">考慮事項</span><span class="sxs-lookup"><span data-stu-id="38a21-143">Considerations</span></span>

- <span data-ttu-id="38a21-144">最初の 2 つの例で実行される I/O 呼び出しの回数は "*減少*" しますが、1 回の呼び出しで取得される情報は "*増加*" します。</span><span class="sxs-lookup"><span data-stu-id="38a21-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="38a21-145">この 2 つの要因間のトレードオフを考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="38a21-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="38a21-146">実際の使用パターンによって正解は異なります。</span><span class="sxs-lookup"><span data-stu-id="38a21-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="38a21-147">たとえば Web API の例で、クライアントが必要なのは多くの場合、ユーザー名だけであることがわかったとしましょう。</span><span class="sxs-lookup"><span data-stu-id="38a21-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="38a21-148">その場合は、それを独立した API 呼び出しとして公開することは理にかなっているかもしれません。</span><span class="sxs-lookup"><span data-stu-id="38a21-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="38a21-149">詳細については、[余分なフェッチ][extraneous-fetching]のアンチパターンを参照してください。</span><span class="sxs-lookup"><span data-stu-id="38a21-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="38a21-150">データを読み取るとき、I/O 要求を大きくしすぎないでください。</span><span class="sxs-lookup"><span data-stu-id="38a21-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="38a21-151">アプリケーションで使う可能性の高い情報に限定して取得する必要があります。</span><span class="sxs-lookup"><span data-stu-id="38a21-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="38a21-152">ときにはオブジェクトの情報を 2 つのチャンク (塊) に分割することも有効な手段となります (要求の大半を占める "*アクセス頻度の高いデータ*" と、まれにしか使わない "*アクセス頻度の低いデータ*")。</span><span class="sxs-lookup"><span data-stu-id="38a21-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="38a21-153">アクセス頻度が最も高いデータは、オブジェクト全体のデータに比べてほんの一部であることも多く、その部分だけを取得するようにすれば、I/O オーバーヘッドを大幅に削減することができます。</span><span class="sxs-lookup"><span data-stu-id="38a21-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="38a21-154">データを書き込むとき、リソースを必要以上に長い時間ロックすることを避け、時間のかかる操作の実行中に生じる競合のリスクを抑えるようにしてください。</span><span class="sxs-lookup"><span data-stu-id="38a21-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="38a21-155">書き込み操作が、複数のデータ ストアや複数のファイル、または複数のサービスにまたがる場合は、結果整合性のアプローチを採用します。</span><span class="sxs-lookup"><span data-stu-id="38a21-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="38a21-156">[データ整合性のガイダンス][data-consistency-guidance]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="38a21-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="38a21-157">データをメモリにバッファー処理した後で書き込む場合、そのデータはプロセスのクラッシュに対して脆弱になります。</span><span class="sxs-lookup"><span data-stu-id="38a21-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="38a21-158">通常のデータ転送に連続性がある場合やさほど混雑しない場合は、持続性のある外部キュー ([Event Hubs](http://azure.microsoft.com/services/event-hubs/) など) にデータをバッファー処理した方が安全です。</span><span class="sxs-lookup"><span data-stu-id="38a21-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="38a21-159">サービスまたはデータベースから取得したデータをキャッシュすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="38a21-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="38a21-160">同じデータの要求を繰り返す必要がないので、I/O の量を減らすことにつながります。</span><span class="sxs-lookup"><span data-stu-id="38a21-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="38a21-161">詳細については、[キャッシュのベスト プラクティス][caching-guidance]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="38a21-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="38a21-162">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="38a21-162">How to detect the problem</span></span>

<span data-ttu-id="38a21-163">頻度の高い I/O は、待ち時間の長さやスループットの低さといった症状となって現れます。</span><span class="sxs-lookup"><span data-stu-id="38a21-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="38a21-164">I/O リソースの競合が増えることから、応答時間の増加やサービスのタイムアウトに起因した障害がエンド ユーザーから報告されることになります。</span><span class="sxs-lookup"><span data-stu-id="38a21-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="38a21-165">問題の原因を特定しやすくするために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="38a21-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="38a21-166">運用システムのプロセス監視を実行して、応答時間が長い操作を特定します。</span><span class="sxs-lookup"><span data-stu-id="38a21-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="38a21-167">前の手順で特定した各操作のロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="38a21-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="38a21-168">ロード テスト中、各操作によって行われたデータ アクセス要求に関するテレメトリ データを収集します。</span><span class="sxs-lookup"><span data-stu-id="38a21-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="38a21-169">データ ストアに送信された要求ごとに詳細な統計情報を収集します。</span><span class="sxs-lookup"><span data-stu-id="38a21-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="38a21-170">テスト環境でアプリケーションをプロファイリングし、I/O ボトルネックがあるとすれば、どこで生じる可能性があるかを特定します。</span><span class="sxs-lookup"><span data-stu-id="38a21-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="38a21-171">次のような症状がないか調べます。</span><span class="sxs-lookup"><span data-stu-id="38a21-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="38a21-172">大量の小さな I/O 要求が同じファイルに対して実行されている。</span><span class="sxs-lookup"><span data-stu-id="38a21-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="38a21-173">アプリケーションのインスタンスで大量の小さなネットワーク要求が同じサービスに対して実行されている。</span><span class="sxs-lookup"><span data-stu-id="38a21-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="38a21-174">アプリケーションのインスタンスで大量の小さな要求が同じデータ ストアに対して実行されている。</span><span class="sxs-lookup"><span data-stu-id="38a21-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="38a21-175">アプリケーションやサービスが I/O バウンドになっている。</span><span class="sxs-lookup"><span data-stu-id="38a21-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="38a21-176">診断の例</span><span class="sxs-lookup"><span data-stu-id="38a21-176">Example diagnosis</span></span>

<span data-ttu-id="38a21-177">以降のセクションでは、上記の手順を、データベースに対してクエリを実行する前述の例に当てはめていきます。</span><span class="sxs-lookup"><span data-stu-id="38a21-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="38a21-178">アプリケーションのロード テストを実行する</span><span class="sxs-lookup"><span data-stu-id="38a21-178">Load test the application</span></span>

<span data-ttu-id="38a21-179">このグラフはロード テストの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="38a21-180">応答時間の中央値は、要求あたりの時間 (10 秒単位) で測定されています。</span><span class="sxs-lookup"><span data-stu-id="38a21-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="38a21-181">このグラフを見ると、待ち時間が非常に長いことが確認できます。</span><span class="sxs-lookup"><span data-stu-id="38a21-181">The graph shows very high latency.</span></span> <span data-ttu-id="38a21-182">負荷が 1,000 ユーザーになると、クエリの結果が表示されるまでに、ユーザーは 1 分近くの待ち時間を強いられます。</span><span class="sxs-lookup"><span data-stu-id="38a21-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![頻度の高い I/O サンプル アプリケーションのキー インジケーター (ロード テストの結果)][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="38a21-184">このアプリケーションは、Azure SQL Database を使って Azure App Service Web アプリとしてデプロイされました。</span><span class="sxs-lookup"><span data-stu-id="38a21-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="38a21-185">ロード テストには、最大 1,000 人の同時実行ユーザーのステップ ワークロードをシミュレートして使用しました。</span><span class="sxs-lookup"><span data-stu-id="38a21-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="38a21-186">接続の競合が結果に影響する可能性を低く抑えるために、データベースは、最大 1,000 件の同時接続をサポートする接続プールを使って構成されています。</span><span class="sxs-lookup"><span data-stu-id="38a21-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="38a21-187">アプリケーションを監視する</span><span class="sxs-lookup"><span data-stu-id="38a21-187">Monitor the application</span></span>

<span data-ttu-id="38a21-188">頻度の高い I/O の特定に寄与する可能性のある主要なメトリックは、アプリケーション パフォーマンス監視 (APM) パッケージを使って収集し、分析することができます。</span><span class="sxs-lookup"><span data-stu-id="38a21-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="38a21-189">どのメトリックが重要かは、I/O ワークロードによって異なります。</span><span class="sxs-lookup"><span data-stu-id="38a21-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="38a21-190">この例で注目する I/O 要求はデータベース クエリです。</span><span class="sxs-lookup"><span data-stu-id="38a21-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="38a21-191">次の図は、[New Relic APM][new-relic] を使って生成された結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="38a21-192">データベースの平均応答時間は、最大ワークロード時に約 5.6 秒/要求でピークとなっています。</span><span class="sxs-lookup"><span data-stu-id="38a21-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="38a21-193">このシステムが耐えることのできる 1 分あたりの要求数は、テスト期間を通じて平均 410 RPM でした。</span><span class="sxs-lookup"><span data-stu-id="38a21-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![AdventureWorks2012 データベースにヒットするトラフィックの概要][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="38a21-195">詳細なデータ アクセス情報を収集する</span><span class="sxs-lookup"><span data-stu-id="38a21-195">Gather detailed data access information</span></span>

<span data-ttu-id="38a21-196">監視データをさらに詳しく分析すると、このアプリケーションで実行される SQL SELECT ステートメントは 3 種類あることがわかります。</span><span class="sxs-lookup"><span data-stu-id="38a21-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="38a21-197">それぞれ `ProductListPriceHistory`、`Product`、`ProductSubcategory` の各テーブルからデータをフェッチするために Entity Framework によって生成される要求に対応します。</span><span class="sxs-lookup"><span data-stu-id="38a21-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="38a21-198">さらに、最も頻繁に実行されている SELECT ステートメントは、`ProductListPriceHistory` テーブルからデータを取得するクエリで、他に比べて桁違いに多くなっています。</span><span class="sxs-lookup"><span data-stu-id="38a21-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![テスト下のサンプル アプリケーションによって実行されたクエリ][queries]

<span data-ttu-id="38a21-200">前出の `GetProductsInSubCategoryAsync` メソッドでは、45 回の SELECT クエリが実行されたことがわかります。</span><span class="sxs-lookup"><span data-stu-id="38a21-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="38a21-201">アプリケーションは、クエリごとに新しい SQL 接続を開きます。</span><span class="sxs-lookup"><span data-stu-id="38a21-201">Each query causes the application to open a new SQL connection.</span></span>

![テスト下のサンプル アプリケーションに使用されているクエリの統計情報][queries2]

> [!NOTE]
> <span data-ttu-id="38a21-203">この画像は、ロード テストにおいて、`GetProductsInSubCategoryAsync` 操作に最も時間のかかったインスタンスのトレース情報を示したものです。</span><span class="sxs-lookup"><span data-stu-id="38a21-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="38a21-204">運用環境では、最も時間のかかるインスタンスのトレースを調査して、問題を示すパターンが存在するかどうかを確かめるようお勧めします。</span><span class="sxs-lookup"><span data-stu-id="38a21-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="38a21-205">平均値にのみ注目すると、負荷条件下で著しく悪化する問題を見過ごすおそれがあります。</span><span class="sxs-lookup"><span data-stu-id="38a21-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="38a21-206">次の画像は、実際に発行された SQL ステートメントを示しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="38a21-207">価格情報をフェッチするクエリは、製品サブカテゴリ内の製品ごとに実行されています。</span><span class="sxs-lookup"><span data-stu-id="38a21-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="38a21-208">JOIN を使用すれば、データベースの呼び出し回数を大幅に減らせる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="38a21-208">Using a join would considerably reduce the number of database calls.</span></span>

![テスト下のサンプル アプリケーションに使用されているクエリの詳細][queries3]

<span data-ttu-id="38a21-210">Entity Framework などの O/RM を使用している場合、SQL クエリをトレースすることによって、プログラミングによる SQL ステートメントの呼び出しが O/RM によってどのように変換されるかについての洞察が得られ、また、データ アクセスが最適化される可能性のある領域を把握できます。</span><span class="sxs-lookup"><span data-stu-id="38a21-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="38a21-211">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="38a21-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="38a21-212">Entity Framework の呼び出しに変更を加えたところ、次のような結果が得られました。</span><span class="sxs-lookup"><span data-stu-id="38a21-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![I/O 頻度を下げて塊で送信するようにサンプル アプリケーションの API を見直した後のキー インジケーター (ロード テストの結果)][key-indicators-chunky-io]

<span data-ttu-id="38a21-214">このロード テストは、同じデプロイ環境で同じ負荷プロファイルを使用して実行されました。</span><span class="sxs-lookup"><span data-stu-id="38a21-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="38a21-215">このグラフを見ると、今回は待ち時間が大幅に短縮されていることが確認できます。</span><span class="sxs-lookup"><span data-stu-id="38a21-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="38a21-216">前回 1 分近くかかっていた、ユーザー数が 1,000 人のときの平均要求時間は 5 ～ 6 秒にまで短縮されています。</span><span class="sxs-lookup"><span data-stu-id="38a21-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="38a21-217">システムが耐える 1 分あたりの要求数も、前回のテストでは 410 RPM であったのに対し、今回は平均 3,970 RPM となりました。</span><span class="sxs-lookup"><span data-stu-id="38a21-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![チャンクを重視した API のトランザクションの概要][databasetraffic2]

<span data-ttu-id="38a21-219">SQL ステートメントのトレース結果は、すべてのデータが 1 回の SELECT ステートメントでフェッチされていることを示しています。</span><span class="sxs-lookup"><span data-stu-id="38a21-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="38a21-220">先ほどよりもかなり複雑になりましたが、このクエリが実行される回数は、各操作につき 1 回だけです。</span><span class="sxs-lookup"><span data-stu-id="38a21-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="38a21-221">また、複雑な JOIN はコストが大きくなる場合がありますが、リレーショナル データベース システムは、このタイプのクエリに最適化されています。</span><span class="sxs-lookup"><span data-stu-id="38a21-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![チャンクを重視した API のクエリの詳細][queries4]

## <a name="related-resources"></a><span data-ttu-id="38a21-223">関連リソース</span><span class="sxs-lookup"><span data-stu-id="38a21-223">Related resources</span></span>

- <span data-ttu-id="38a21-224">[API 設計のベスト プラクティス][api-design]</span><span class="sxs-lookup"><span data-stu-id="38a21-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="38a21-225">[キャッシングのベスト プラクティス][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="38a21-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="38a21-226">[データ整合性入門][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="38a21-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="38a21-227">[余分なフェッチのアンチパターン][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="38a21-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="38a21-228">[キャッシュ不使用のアンチパターン][no-cache]</span><span class="sxs-lookup"><span data-stu-id="38a21-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

