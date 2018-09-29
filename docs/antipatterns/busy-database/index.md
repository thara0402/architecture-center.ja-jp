---
title: ビジー状態のデータベースのアンチパターン
description: 処理をデータベース サーバーにオフロードすると、パフォーマンスおよびスケーラビリティの問題が発生する可能性があります。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: a14a350aefc1801ae08cb4a8d0eb3d5b248c92bf
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428909"
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="dd1ee-103">ビジー状態のデータベースのアンチパターン</span><span class="sxs-lookup"><span data-stu-id="dd1ee-103">Busy Database antipattern</span></span>

<span data-ttu-id="dd1ee-104">処理をデータベース サーバーにオフロードすると、データの格納および取得要求への応答よりも、コードの実行にかなりの時間が費やされる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="dd1ee-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="dd1ee-105">Problem description</span></span>

<span data-ttu-id="dd1ee-106">多くのデータベース システムでは、コードを実行できます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-106">Many database systems can run code.</span></span> <span data-ttu-id="dd1ee-107">例としては、ストアド プロシージャやトリガーが挙げられます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="dd1ee-108">多くの場合、データを処理のためにクライアント アプリケーションに転送するよりも、データの近くでこの処理を実行する方が効率的です。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="dd1ee-109">ただし、これらの機能を過度に使用すると、次のような理由でパフォーマンスが低下する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="dd1ee-110">データベース サーバーが、新しいクライアント要求を受け入れてデータを取得するのよりも、処理に過剰な時間を費やす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="dd1ee-111">通常、データベースは共有リソースであるため、使用率が高い期間にデータベースがボトルネックになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="dd1ee-112">データ ストアが従量制の場合、実行時のコストが過剰になることがあります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="dd1ee-113">これは特に管理されたデータベース サービスに当てはまります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="dd1ee-114">たとえば、Azure SQL Database では、[データベース トランザクション ユニット][dtu] (DTU) に対して課金されます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="dd1ee-115">データベースをスケールアップできる容量には限度があり、データベースを水平方向にスケーリングすることは簡単ではありません。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="dd1ee-116">そのため、VM や App Service アプリなど、簡単にスケールアウトできる計算リソースに処理を移行する方が良い場合があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="dd1ee-117">このアンチパターンは、通常、次の理由で発生します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="dd1ee-118">データベースがリポジトリではなくサービスとして表示されている。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="dd1ee-119">アプリケーションは、データベース サーバーを使用して、データの書式設定 (XML への変換など)、文字列データの操作、または複雑な計算を実行できます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="dd1ee-120">開発者が結果をユーザーに直接表示できるクエリを作成しようとしている。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="dd1ee-121">たとえば、クエリを使用して、フィールドを組み合わせたり、ロケールに応じて日付、時刻、および通貨の形式を設定したりすることができます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="dd1ee-122">開発者が、計算をデータベースにプッシュすることによって、[余分なフェッチ][ExtraneousFetching]のアンチパターンを修正しようとしている。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="dd1ee-123">ストアド プロシージャの方が管理と更新が容易であると考えられるために、ビジネスロジックをカプセル化するためにストアド プロシージャが使用されている。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="dd1ee-124">次の例では、指定された販売区域で最も重要な 20 件の注文を取得し、その結果の形式を XML として設定しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="dd1ee-125">この例では、Transact-SQL 関数を使用してデータを解析し、結果を XML に変換しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="dd1ee-126">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-126">You can find the complete sample [here][sample-app].</span></span>

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

<span data-ttu-id="dd1ee-127">明らかに、これは複雑なクエリです。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-127">Clearly, this is complex query.</span></span> <span data-ttu-id="dd1ee-128">後で説明するように、この例を実行すると、データベース サーバー上の大量の処理リソースが使用されます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="dd1ee-129">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="dd1ee-129">How to fix the problem</span></span>

<span data-ttu-id="dd1ee-130">処理をデータベース サーバーから他のアプリケーション層に移動します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="dd1ee-131">理想的には、RDBMS での集計など、データベースが最適化される機能のみを使用して、データベースをデータ アクセス操作の実行に制限する必要があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="dd1ee-132">たとえば、前の Transact-SQL コードは、処理するデータを単に取得するステートメントで置き換えることができます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

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

<span data-ttu-id="dd1ee-133">次に、このアプリケーションでは、.NET Framework の `System.Xml.Linq` API を使用して、結果の形式が XML として設定されます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

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
> <span data-ttu-id="dd1ee-134">このコードはやや複雑です。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-134">This code is somewhat complex.</span></span> <span data-ttu-id="dd1ee-135">新しいアプリケーションではシリアル化ライブラリを使用するのを好む方もいると思われます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="dd1ee-136">ただし、ここでは開発チームが既存のアプリケーションをリファクタリングしていることを前提としているため、メソッドで元のコードとまったく同じ形式を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="dd1ee-137">考慮事項</span><span class="sxs-lookup"><span data-stu-id="dd1ee-137">Considerations</span></span>

- <span data-ttu-id="dd1ee-138">多くのデータベース システムは、大規模なデータ セットに対する集計値の計算など、特定の種類のデータ処理を実行するために高度に最適化されています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="dd1ee-139">この種類の処理はデータベースから移動してはなりません。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="dd1ee-140">処理を再配置することでデータベースからネットワーク経由で転送されるデータの量が増える場合は、処理を再配置しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="dd1ee-141">「[Extraneous Fetching antipattern (余分なフェッチのアンチパターン)][ExtraneousFetching]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="dd1ee-142">処理をアプリケーション層に移動するにあたっては、追加の操作を処理するためにその層をスケールアウトすることが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="dd1ee-143">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="dd1ee-143">How to detect the problem</span></span>

<span data-ttu-id="dd1ee-144">ビジー状態のデータベースでは、データベースにアクセスする操作においてスループットと応答時間の急激な低下が見られます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span> 

<span data-ttu-id="dd1ee-145">この問題の識別に役立てるために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-145">You can perform the following steps to help identify this problem:</span></span> 

1. <span data-ttu-id="dd1ee-146">パフォーマンスの監視を使用して、運用システムにおいてデータベース アクティビティの実行に費やされる時間を識別します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="dd1ee-147">対象の期間中にデータベースによって実行された操作を確認します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="dd1ee-148">特定の操作でデータベース アクティビティが多すぎると思われる場合は、管理された環境でロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="dd1ee-149">各テストは、疑わしい操作と可変のユーザー負荷を組み合わせて実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="dd1ee-150">ロード テストのテレメトリを調べて、データベースの使用状況を監視します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="dd1ee-151">大量の処理が発生する一方でデータ トラフィックがほとんどないことがデータベース アクティビティからわかった場合は、ソース コードを見直して、どこかで効率よく処理を実行できる方法を検討します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="dd1ee-152">データベース アクティビティの量が少ない場合や応答時間が比較的速い場合は、ビジー状態のデータベースがパフォーマンス上の問題になる可能性は低くなります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="dd1ee-153">診断の例</span><span class="sxs-lookup"><span data-stu-id="dd1ee-153">Example diagnosis</span></span>

<span data-ttu-id="dd1ee-154">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="dd1ee-155">データベース アクティビティの量を監視する</span><span class="sxs-lookup"><span data-stu-id="dd1ee-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="dd1ee-156">次のグラフは、最大 50 人の同時実行ユーザーのステップ負荷を使用してサンプル アプリケーションに対してロード テストを実行した結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="dd1ee-157">要求の量はすぐに限界に達してそのレベルに留まりますが、平均応答時間は徐々に上昇しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="dd1ee-158">これらの 2 つのメトリックには対数スケールが使用されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![データベースで処理を実行するためのロード テストの結果][ProcessingInDatabaseLoadTest]

<span data-ttu-id="dd1ee-160">次のグラフは、CPU 使用率と DTU をサービス クォータの割合として示しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="dd1ee-161">DTU は、データベースで実行される処理量のメジャーを示します。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="dd1ee-162">グラフから、CPU と DTU の両方の使用率がすぐに 100% に達したことがわかります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Azure SQL Database モニターに表示された、処理を実行中のデータベースのパフォーマンス][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="dd1ee-164">データベースによって実行された操作を確認する</span><span class="sxs-lookup"><span data-stu-id="dd1ee-164">Examine the work performed by the database</span></span>

<span data-ttu-id="dd1ee-165">データベースによって実行されるタスクは処理ではなく本物のデータ アクセス操作であるため、データベースがビジー状態の間に実行される SQL ステートメントを理解することが重要です。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="dd1ee-166">システムを監視して SQL トラフィックをキャプチャし、SQL 操作をアプリケーション要求と関連付けます。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="dd1ee-167">データベース操作が大量の処理を含まない純粋なデータ アクセス操作である場合、問題の原因は[余分なフェッチ][ExtraneousFetching]にある可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="dd1ee-168">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="dd1ee-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="dd1ee-169">次のグラフは、更新されたコードを使用したロード テストを示しています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="dd1ee-170">スループットについては、以前の 1 秒あたり 12 要求に対し、400 要求を上回る非常に高い値が得られています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="dd1ee-171">平均応答時間についても、以前の 4 秒以上から、0.1 秒をわずかに上回るだけの大幅に低い結果となっています。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![データベースで処理を実行するためのロード テストの結果][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="dd1ee-173">CPU と DTU の使用率からは、スループットが高くなっているにもかかわらずシステムが限度に達するまでの時間が長くなっていることがわかります。</span><span class="sxs-lookup"><span data-stu-id="dd1ee-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Azure SQL Database モニターに表示された、クライアント アプリケーションで処理を実行中のデータベースのパフォーマンス][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="dd1ee-175">関連リソース</span><span class="sxs-lookup"><span data-stu-id="dd1ee-175">Related resources</span></span> 

- <span data-ttu-id="dd1ee-176">[余分なフェッチのアンチパターン][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="dd1ee-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>


[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
