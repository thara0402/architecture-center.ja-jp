---
title: 頻度の高い I/O のアンチパターン
description: 大量の I/O 要求によってパフォーマンスと応答性が損なわれる場合があります。
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
---
# <a name="chatty-io-antipattern"></a>頻度の高い I/O のアンチパターン

大量の I/O 要求の影響が累積して、パフォーマンスと応答性に著しい影響を与える場合があります。

## <a name="problem-description"></a>問題の説明

ネットワーク呼び出しをはじめとする I/O 操作は、コンピューティング タスクと比べて本質的に低速です。 I/O 要求は 1 回ごとに著しいオーバーヘッドを伴うのが普通で、多数の I/O 操作の影響が累積することでシステムの処理能力が低下するおそれがあります。 以降、頻度の高い I/O の代表的な原因をいくつか取り上げます。

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a>データベースに対する各レコードの読み取りと書き込みを個別の要求として実行する

以下の例では、製品のデータベースから読み取りを実行しています。 テーブルは、`Product`、`ProductSubcategory`、`ProductPriceListHistory` の 3 つが存在します。 このコードは、次の一連のクエリを実行して、サブカテゴリに属しているすべての製品とその価格情報を取得します。  

1. `ProductSubcategory` テーブルにサブカテゴリを照会します。
2. `Product` テーブルを照会して、そのサブカテゴリに属している製品をすべて検索します。
3. 製品ごとの価格データを `ProductPriceListHistory` テーブルに照会します。

このアプリケーションは、[Entity Framework][ef] を使用してデータベースを照会します。 完全なサンプルは、[こちら][code-sample]でご覧いただけます。 

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

この例では問題を明示的に示していますが、O/RM (オブジェクト関係マッピング) で暗黙的に子レコードが 1 件ずつフェッチされていると、問題に気付きにくいことがあります。 これは "N+1 問題" として知られています。 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a>1 つの論理的操作を一連の HTTP 要求として実装する

これは多くの場合、開発者がオブジェクト指向パラダイムに従おうと、リモート オブジェクトをメモリ内のローカル オブジェクトであるかのように扱うことで起こります。 この場合、過剰なネットワーク ラウンド トリップが発生します。 たとえば次の Web API は、`User` オブジェクトの各プロパティを別々の HTTP GET メソッドで公開しています。 

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

この手法に関して技術的に間違っているところは何もありませんが、おそらく大半のクライアントは、`User` ごとに複数のプロパティを取得しなければならなくなるでしょう。次のようにクライアント コードを記述しなければなりません。 

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

### <a name="reading-and-writing-to-a-file-on-disk"></a>ディスク上のファイルの読み取りと書き込みを実行する

ファイル I/O は必ずファイルを開いて、適切なポイントに移動したうえで、データの読み取りまたは書き込みを行います。 その操作が完了したら、オペレーティング システムのリソースを節約するために、そのファイルは閉じることになるでしょう。 ファイルに対して少量の情報の読み取りと書き込みを絶えず繰り返すアプリケーションでは、著しい I/O オーバーヘッドが生じます。 少量の書き込み要求がファイルの断片化を招き、その後の I/O 操作がさらに低速化する場合もあります。 

以下の例では、`FileStream` を使用して `Customer` オブジェクトをファイルに書き込んでいます。 `FileStream` を作成するとファイルが開き、破棄するとファイルが閉じます  (`FileStream` オブジェクトは `using` ステートメントによって自動的に破棄されます)。新しい顧客が追加されるたびにこのメソッドが繰り返し呼び出されると、I/O オーバーヘッドはすぐに累積していくことでしょう。

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

## <a name="how-to-fix-the-problem"></a>問題の解決方法

I/O 要求数を減らすには、データをより大きなまとまりにして要求の回数を減らします。

小さなクエリを何度も繰り返すのではなく、1 回のクエリでデータベースからデータをフェッチするようにしましょう。 製品情報を取得する変更後のコードは以下のとおりです。

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

Web API については、REST の設計原則に従います。 先ほどの例を修正した Web API を次に示します。 プロパティごとに別々の GET メソッドが存在するのではなく、`User` を返す GET メソッドが 1 つだけ存在します。 これによって 1 回の要求で返される応答本文は大きくなりますが、各クライアントが API を呼び出す回数は少なくなるでしょう。

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

ファイル I/O に関しては、データをメモリにバッファー処理しておき、それを 1 回の操作でファイルに書き込むことを検討してください。 この方法により、ファイルの開閉を繰り返すことで生じるオーバーヘッドを削減し、ディスク上のファイルの断片化を軽減できます。

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

## <a name="considerations"></a>考慮事項

- 最初の 2 つの例で実行される I/O 呼び出しの回数は "*減少*" しますが、1 回の呼び出しで取得される情報は "*増加*" します。 この 2 つの要因間のトレードオフを考慮する必要があります。 実際の使用パターンによって正解は異なります。 たとえば Web API の例で、クライアントが必要なのは多くの場合、ユーザー名だけであることがわかったとしましょう。 その場合は、それを独立した API 呼び出しとして公開することは理にかなっているかもしれません。 詳細については、[余分なフェッチ][extraneous-fetching]のアンチパターンを参照してください。

- データを読み取るとき、I/O 要求を大きくしすぎないでください。 アプリケーションで使う可能性の高い情報に限定して取得する必要があります。 

- ときにはオブジェクトの情報を 2 つのチャンク (塊) に分割することも有効な手段となります (要求の大半を占める "*アクセス頻度の高いデータ*" と、まれにしか使わない "*アクセス頻度の低いデータ*")。 アクセス頻度が最も高いデータは、オブジェクト全体のデータに比べてほんの一部であることも多く、その部分だけを取得するようにすれば、I/O オーバーヘッドを大幅に削減することができます。

- データを書き込むとき、リソースを必要以上に長い時間ロックすることを避け、時間のかかる操作の実行中に生じる競合のリスクを抑えるようにしてください。 書き込み操作が、複数のデータ ストアや複数のファイル、または複数のサービスにまたがる場合は、結果整合性のアプローチを採用します。 [データ整合性のガイダンス][data-consistency-guidance]に関するページを参照してください。

- データをメモリにバッファー処理した後で書き込む場合、そのデータはプロセスのクラッシュに対して脆弱になります。 通常のデータ転送に連続性がある場合やさほど混雑しない場合は、持続性のある外部キュー ([Event Hubs](http://azure.microsoft.com/services/event-hubs/) など) にデータをバッファー処理した方が安全です。

- サービスまたはデータベースから取得したデータをキャッシュすることを検討してください。 同じデータの要求を繰り返す必要がないので、I/O の量を減らすことにつながります。 詳細については、[キャッシュのベスト プラクティス][caching-guidance]に関するページを参照してください。

## <a name="how-to-detect-the-problem"></a>問題の検出方法

頻度の高い I/O は、待ち時間の長さやスループットの低さといった症状となって現れます。 I/O リソースの競合が増えることから、応答時間の増加やサービスのタイムアウトに起因した障害がエンド ユーザーから報告されることになります。

問題の原因を特定しやすくするために、次の手順を実行できます。

1. 運用システムのプロセス監視を実行して、応答時間が長い操作を特定します。
2. 前の手順で特定した各操作のロード テストを実行します。
3. ロード テスト中、各操作によって行われたデータ アクセス要求に関するテレメトリ データを収集します。
4. データ ストアに送信された要求ごとに詳細な統計情報を収集します。
5. テスト環境でアプリケーションをプロファイリングし、I/O ボトルネックがあるとすれば、どこで生じる可能性があるかを特定します。 

次のような症状がないか調べます。

- 大量の小さな I/O 要求が同じファイルに対して実行されている。
- アプリケーションのインスタンスで大量の小さなネットワーク要求が同じサービスに対して実行されている。
- アプリケーションのインスタンスで大量の小さな要求が同じデータ ストアに対して実行されている。
- アプリケーションやサービスが I/O バウンドになっている。

## <a name="example-diagnosis"></a>診断の例

以降のセクションでは、上記の手順を、データベースに対してクエリを実行する前述の例に当てはめていきます。

### <a name="load-test-the-application"></a>アプリケーションのロード テストを実行する

このグラフはロード テストの結果を示しています。 応答時間の中央値は、要求あたりの時間 (10 秒単位) で測定されています。 このグラフを見ると、待ち時間が非常に長いことが確認できます。 負荷が 1,000 ユーザーになると、クエリの結果が表示されるまでに、ユーザーは 1 分近くの待ち時間を強いられます。 

![頻度の高い I/O サンプル アプリケーションのキー インジケーター (ロード テストの結果)][key-indicators-chatty-io]

> [!NOTE]
> このアプリケーションは、Azure SQL Database を使って Azure App Service Web アプリとしてデプロイされました。 ロード テストには、最大 1,000 人の同時実行ユーザーのステップ ワークロードをシミュレートして使用しました。 接続の競合が結果に影響する可能性を低く抑えるために、データベースは、最大 1,000 件の同時接続をサポートする接続プールを使って構成されています。 

### <a name="monitor-the-application"></a>アプリケーションを監視する

頻度の高い I/O の特定に寄与する可能性のある主要なメトリックは、アプリケーション パフォーマンス監視 (APM) パッケージを使って収集し、分析することができます。 どのメトリックが重要かは、I/O ワークロードによって異なります。 この例で注目する I/O 要求はデータベース クエリです。 

次の図は、[New Relic APM][new-relic] を使って生成された結果を示しています。 データベースの平均応答時間は、最大ワークロード時に約 5.6 秒/要求でピークとなっています。 このシステムが耐えることのできる 1 分あたりの要求数は、テスト期間を通じて平均 410 RPM でした。

![AdventureWorks2012 データベースにヒットするトラフィックの概要][databasetraffic]

### <a name="gather-detailed-data-access-information"></a>詳細なデータ アクセス情報を収集する

監視データをさらに詳しく分析すると、このアプリケーションで実行される SQL SELECT ステートメントは 3 種類あることがわかります。 それぞれ `ProductListPriceHistory`、`Product`、`ProductSubcategory` の各テーブルからデータをフェッチするために Entity Framework によって生成される要求に対応します。
さらに、最も頻繁に実行されている SELECT ステートメントは、`ProductListPriceHistory` テーブルからデータを取得するクエリで、他に比べて桁違いに多くなっています。

![テスト下のサンプル アプリケーションによって実行されたクエリ][queries]

前出の `GetProductsInSubCategoryAsync` メソッドでは、45 回の SELECT クエリが実行されたことがわかります。 アプリケーションは、クエリごとに新しい SQL 接続を開きます。

![テスト下のサンプル アプリケーションに使用されているクエリの統計情報][queries2]

> [!NOTE]
> この画像は、ロード テストにおいて、`GetProductsInSubCategoryAsync` 操作に最も時間のかかったインスタンスのトレース情報を示したものです。 運用環境では、最も時間のかかるインスタンスのトレースを調査して、問題を示すパターンが存在するかどうかを確かめるようお勧めします。 平均値にのみ注目すると、負荷条件下で著しく悪化する問題を見過ごすおそれがあります。

次の画像は、実際に発行された SQL ステートメントを示しています。 価格情報をフェッチするクエリは、製品サブカテゴリ内の製品ごとに実行されています。 JOIN を使用すれば、データベースの呼び出し回数を大幅に減らせる可能性があります。

![テスト下のサンプル アプリケーションに使用されているクエリの詳細][queries3]

Entity Framework などの O/RM を使用している場合、SQL クエリをトレースすることによって、プログラミングによる SQL ステートメントの呼び出しが O/RM によってどのように変換されるかについての洞察が得られ、また、データ アクセスが最適化される可能性のある領域を把握できます。 

### <a name="implement-the-solution-and-verify-the-result"></a>ソリューションを実装して結果を検証する

Entity Framework の呼び出しに変更を加えたところ、次のような結果が得られました。

![I/O 頻度を下げて塊で送信するようにサンプル アプリケーションの API を見直した後のキー インジケーター (ロード テストの結果)][key-indicators-chunky-io]

このロード テストは、同じデプロイ環境で同じ負荷プロファイルを使用して実行されました。 このグラフを見ると、今回は待ち時間が大幅に短縮されていることが確認できます。 前回 1 分近くかかっていた、ユーザー数が 1,000 人のときの平均要求時間は 5 ～ 6 秒にまで短縮されています。

システムが耐える 1 分あたりの要求数も、前回のテストでは 410 RPM であったのに対し、今回は平均 3,970 RPM となりました。

![チャンクを重視した API のトランザクションの概要][databasetraffic2]

SQL ステートメントのトレース結果は、すべてのデータが 1 回の SELECT ステートメントでフェッチされていることを示しています。 先ほどよりもかなり複雑になりましたが、このクエリが実行される回数は、各操作につき 1 回だけです。 また、複雑な JOIN はコストが大きくなる場合がありますが、リレーショナル データベース システムは、このタイプのクエリに最適化されています。  

![チャンクを重視した API のクエリの詳細][queries4]

## <a name="related-resources"></a>関連リソース

- [API 設計のベスト プラクティス][api-design]
- [キャッシングのベスト プラクティス][caching-guidance]
- [データ整合性入門][data-consistency-guidance]
- [余分なフェッチのアンチパターン][extraneous-fetching]
- [キャッシュ不使用のアンチパターン][no-cache]

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

