---
title: ビジー状態のデータベースのアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: 処理をデータベース サーバーにオフロードすると、パフォーマンスおよびスケーラビリティの問題が発生する可能性があります。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="busy-database-antipattern"></a>ビジー状態のデータベースのアンチパターン

処理をデータベース サーバーにオフロードすると、データの格納および取得要求への応答よりも、コードの実行にかなりの時間が費やされる可能性があります。

## <a name="problem-description"></a>問題の説明

多くのデータベース システムでは、コードを実行できます。 例としては、ストアド プロシージャやトリガーが挙げられます。 多くの場合、データを処理のためにクライアント アプリケーションに転送するよりも、データの近くでこの処理を実行する方が効率的です。 ただし、これらの機能を過度に使用すると、次のような理由でパフォーマンスが低下する可能性があります。

- データベース サーバーが、新しいクライアント要求を受け入れてデータを取得するのよりも、処理に過剰な時間を費やす可能性があります。
- 通常、データベースは共有リソースであるため、使用率が高い期間にデータベースがボトルネックになる可能性があります。
- データ ストアが従量制の場合、実行時のコストが過剰になることがあります。 これは特に管理されたデータベース サービスに当てはまります。 たとえば、Azure SQL Database では、[データベース トランザクション ユニット][dtu] (DTU) に対して課金されます。
- データベースをスケールアップできる容量には限度があり、データベースを水平方向にスケーリングすることは簡単ではありません。 そのため、VM や App Service アプリなど、簡単にスケールアウトできる計算リソースに処理を移行する方が良い場合があります。

このアンチパターンは、通常、次の理由で発生します。

- データベースがリポジトリではなくサービスとして表示されている。 アプリケーションは、データベース サーバーを使用して、データの書式設定 (XML への変換など)、文字列データの操作、または複雑な計算を実行できます。
- 開発者が結果をユーザーに直接表示できるクエリを作成しようとしている。 たとえば、クエリを使用して、フィールドを組み合わせたり、ロケールに応じて日付、時刻、および通貨の形式を設定したりすることができます。
- 開発者が、計算をデータベースにプッシュすることによって、[余分なフェッチ][ExtraneousFetching]のアンチパターンを修正しようとしている。
- ストアド プロシージャの方が管理と更新が容易であると考えられるために、ビジネスロジックをカプセル化するためにストアド プロシージャが使用されている。

次の例では、指定された販売区域で最も重要な 20 件の注文を取得し、その結果の形式を XML として設定しています。
この例では、Transact-SQL 関数を使用してデータを解析し、結果を XML に変換しています。 完全なサンプルは、[こちら][sample-app]でご覧いただけます。

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

明らかに、これは複雑なクエリです。 後で説明するように、この例を実行すると、データベース サーバー上の大量の処理リソースが使用されます。

## <a name="how-to-fix-the-problem"></a>問題の解決方法

処理をデータベース サーバーから他のアプリケーション層に移動します。 理想的には、RDBMS での集計など、データベースが最適化される機能のみを使用して、データベースをデータ アクセス操作の実行に制限する必要があります。

たとえば、前の Transact-SQL コードは、処理するデータを単に取得するステートメントで置き換えることができます。

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

次に、このアプリケーションでは、.NET Framework の `System.Xml.Linq` API を使用して、結果の形式が XML として設定されます。

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> このコードはやや複雑です。 新しいアプリケーションではシリアル化ライブラリを使用するのを好む方もいると思われます。 ただし、ここでは開発チームが既存のアプリケーションをリファクタリングしていることを前提としているため、メソッドで元のコードとまったく同じ形式を返す必要があります。

## <a name="considerations"></a>考慮事項

- 多くのデータベース システムは、大規模なデータ セットに対する集計値の計算など、特定の種類のデータ処理を実行するために高度に最適化されています。 この種類の処理はデータベースから移動してはなりません。

- 処理を再配置することでデータベースからネットワーク経由で転送されるデータの量が増える場合は、処理を再配置しないようにしてください。 「[Extraneous Fetching antipattern (余分なフェッチのアンチパターン)][ExtraneousFetching]」を参照してください。

- 処理をアプリケーション層に移動するにあたっては、追加の操作を処理するためにその層をスケールアウトすることが必要になる場合があります。

## <a name="how-to-detect-the-problem"></a>問題の検出方法

ビジー状態のデータベースでは、データベースにアクセスする操作においてスループットと応答時間の急激な低下が見られます。

この問題の識別に役立てるために、次の手順を実行できます。

1. パフォーマンスの監視を使用して、運用システムにおいてデータベース アクティビティの実行に費やされる時間を識別します。

2. 対象の期間中にデータベースによって実行された操作を確認します。

3. 特定の操作でデータベース アクティビティが多すぎると思われる場合は、管理された環境でロード テストを実行します。 各テストは、疑わしい操作と可変のユーザー負荷を組み合わせて実行する必要があります。 ロード テストのテレメトリを調べて、データベースの使用状況を監視します。

4. 大量の処理が発生する一方でデータ トラフィックがほとんどないことがデータベース アクティビティからわかった場合は、ソース コードを見直して、どこかで効率よく処理を実行できる方法を検討します。

データベース アクティビティの量が少ない場合や応答時間が比較的速い場合は、ビジー状態のデータベースがパフォーマンス上の問題になる可能性は低くなります。

## <a name="example-diagnosis"></a>診断の例

以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。

### <a name="monitor-the-volume-of-database-activity"></a>データベース アクティビティの量を監視する

次のグラフは、最大 50 人の同時実行ユーザーのステップ負荷を使用してサンプル アプリケーションに対してロード テストを実行した結果を示しています。 要求の量はすぐに限界に達してそのレベルに留まりますが、平均応答時間は徐々に上昇しています。 これらの 2 つのメトリックには対数スケールが使用されていることに注意してください。

![データベースで処理を実行するためのロード テストの結果][ProcessingInDatabaseLoadTest]

次のグラフは、CPU 使用率と DTU をサービス クォータの割合として示しています。 DTU は、データベースで実行される処理量のメジャーを示します。 グラフから、CPU と DTU の両方の使用率がすぐに 100% に達したことがわかります。

![Azure SQL Database モニターに表示された、処理を実行中のデータベースのパフォーマンス][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a>データベースによって実行された操作を確認する

データベースによって実行されるタスクは処理ではなく本物のデータ アクセス操作であるため、データベースがビジー状態の間に実行される SQL ステートメントを理解することが重要です。 システムを監視して SQL トラフィックをキャプチャし、SQL 操作をアプリケーション要求と関連付けます。

データベース操作が大量の処理を含まない純粋なデータ アクセス操作である場合、問題の原因は[余分なフェッチ][ExtraneousFetching]にある可能性があります。

### <a name="implement-the-solution-and-verify-the-result"></a>ソリューションを実装して結果を検証する

次のグラフは、更新されたコードを使用したロード テストを示しています。 スループットについては、以前の 1 秒あたり 12 要求に対し、400 要求を上回る非常に高い値が得られています。 平均応答時間についても、以前の 4 秒以上から、0.1 秒をわずかに上回るだけの大幅に低い結果となっています。

![データベースで処理を実行するためのロード テストの結果][ProcessingInClientApplicationLoadTest]

CPU と DTU の使用率からは、スループットが高くなっているにもかかわらずシステムが限度に達するまでの時間が長くなっていることがわかります。

![Azure SQL Database モニターに表示された、クライアント アプリケーションで処理を実行中のデータベースのパフォーマンス][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a>関連リソース

- [余分なフェッチのアンチパターン][ExtraneousFetching]

[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
