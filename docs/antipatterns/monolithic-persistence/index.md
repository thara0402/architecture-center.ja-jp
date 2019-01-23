---
title: モノリシックな永続化のアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: アプリケーションのすべてのデータを単一のデータ ストアに配置すると、パフォーマンスが低下する可能性があります。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e3d6fbb789e422af22ce54b0defd6bb73099f15a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487272"
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="97488-103">モノリシックな永続化のアンチパターン</span><span class="sxs-lookup"><span data-stu-id="97488-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="97488-104">アプリケーションのすべてのデータを単一のデータ ストアに配置すると、パフォーマンスが低下する可能性があります。その理由は、リソースが競合する可能性があること、あるいはデータ ストアが一部のデータに適していないことです。</span><span class="sxs-lookup"><span data-stu-id="97488-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="97488-105">問題の説明</span><span class="sxs-lookup"><span data-stu-id="97488-105">Problem description</span></span>

<span data-ttu-id="97488-106">歴史的に、アプリケーションでは、多くの場合、アプリケーションで格納することが必要になるデータの種類を問わずに単一のデータ ストアが使用されていました。</span><span class="sxs-lookup"><span data-stu-id="97488-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="97488-107">通常、その目的は、アプリケーションの設計を簡素化することや、開発チームの既存のスキル セットに合わせることにありました。</span><span class="sxs-lookup"><span data-stu-id="97488-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span>

<span data-ttu-id="97488-108">最新のクラウド ベースのシステムでは、多くの場合、機能やそれ以外の要件が余分にあり、ドキュメント、画像、キャッシュ データ、キューに格納されたメッセージ、アプリケーション ログ、テレメトリなど、さまざまな種類のデータを保存する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97488-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="97488-109">従来のアプローチに従ってこれらすべての情報を同じデータ ストアに格納すると、主に次の 2 つの理由から、パフォーマンスが低下する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="97488-110">関連性のない大量のデータを同じデータ ストアに格納および取得すると競合が発生し、応答時間が遅くなると共に、接続が失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="97488-111">どのデータ ストアを選択したとしても、そのデータ ストアは、各種データすべてにとって最適であるとは限らず、アプリケーションで実行する操作に最適化されていない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span>

<span data-ttu-id="97488-112">次の例は、新しいレコードをデータベースに追加し、その結果をログに記録する ASP.NET Web API コントローラーを示しています。</span><span class="sxs-lookup"><span data-stu-id="97488-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="97488-113">ログは、ビジネス データと同じデータベースに保持されます。</span><span class="sxs-lookup"><span data-stu-id="97488-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="97488-114">完全なサンプルは、[こちら][sample-app]でご覧いただけます。</span><span class="sxs-lookup"><span data-stu-id="97488-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="97488-115">ログ レコードの生成レートは、業務のパフォーマンスに影響するものと思われます。</span><span class="sxs-lookup"><span data-stu-id="97488-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="97488-116">また、アプリケーション プロセス モニターなどの別のコンポーネントが定期的にログ データを読み取って処理した結果、業務に影響が及ぶ可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="97488-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="97488-117">問題の解決方法</span><span class="sxs-lookup"><span data-stu-id="97488-117">How to fix the problem</span></span>

<span data-ttu-id="97488-118">用途に応じてデータを分けます。</span><span class="sxs-lookup"><span data-stu-id="97488-118">Separate data according to its use.</span></span> <span data-ttu-id="97488-119">データ セットごとに、そのデータ セットの使用方法に最も適合するデータ ストアを選択します。</span><span class="sxs-lookup"><span data-stu-id="97488-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="97488-120">前の例では、ビジネス データが保持されるデータベースとは別のストアにアプリケーションのログを保存する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97488-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span>

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="97488-121">考慮事項</span><span class="sxs-lookup"><span data-stu-id="97488-121">Considerations</span></span>

- <span data-ttu-id="97488-122">データの使用方法とアクセス方法によってデータを分けます。</span><span class="sxs-lookup"><span data-stu-id="97488-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="97488-123">たとえば、ログ情報とビジネス データを同じデータ ストアに格納しないようにします。</span><span class="sxs-lookup"><span data-stu-id="97488-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="97488-124">これらの種類のデータは、アクセスの要件とパターンが大きく異なります。</span><span class="sxs-lookup"><span data-stu-id="97488-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="97488-125">ログ レコードは、本質的にシーケンシャルです。これに対し、ビジネス データは、ランダム アクセスを必要とする可能性が高く、多くの場合リレーショナルです。</span><span class="sxs-lookup"><span data-stu-id="97488-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="97488-126">データの種類ごとのデータ アクセス パターンを検討します。</span><span class="sxs-lookup"><span data-stu-id="97488-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="97488-127">たとえば、書式設定されたレポートとドキュメントを格納するには [Cosmos DB][CosmosDB] などのドキュメント データベースを使用し、一時データをキャッシュするには [Azure Redis Cache][Azure-cache] を使用します。</span><span class="sxs-lookup"><span data-stu-id="97488-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="97488-128">このガイダンスに従ってもデータベースの限界に達する場合は、データベースのスケールアップが必要になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="97488-129">また、水平方向にスケーリングし、複数のデータベース サーバー間で負荷を分けることも検討してください。</span><span class="sxs-lookup"><span data-stu-id="97488-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="97488-130">ただし、パーティション分割を行うにはアプリケーションの再設計が必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="97488-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="97488-131">詳細については、「[Data partitioning (データのパーティション分割)][DataPartitioningGuidance]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="97488-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="97488-132">問題の検出方法</span><span class="sxs-lookup"><span data-stu-id="97488-132">How to detect the problem</span></span>

<span data-ttu-id="97488-133">データベース接続などのシステム リソースが不足すると、システムは大幅に低速になり、最終的には動作しなくなります。</span><span class="sxs-lookup"><span data-stu-id="97488-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="97488-134">原因の識別に役立てるために、次の手順を実行できます。</span><span class="sxs-lookup"><span data-stu-id="97488-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="97488-135">システムをインストルメント化して、主要なパフォーマンスの統計を記録します。</span><span class="sxs-lookup"><span data-stu-id="97488-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="97488-136">アプリケーションがデータを読み書きするポイントだけでなく、各操作のタイミング情報をキャプチャします。</span><span class="sxs-lookup"><span data-stu-id="97488-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
2. <span data-ttu-id="97488-137">可能であれば、運用環境でシステムの実行を数日間監視し、実際のシステムの使用状況を把握します。</span><span class="sxs-lookup"><span data-stu-id="97488-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="97488-138">これが不可能な場合は、典型的な一連の操作を実行する、現実的な数の仮想ユーザーを使用して、スクリプトによるロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="97488-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
3. <span data-ttu-id="97488-139">テレメトリ データを使用して、パフォーマンスが低下している期間を識別します。</span><span class="sxs-lookup"><span data-stu-id="97488-139">Use the telemetry data to identify periods of poor performance.</span></span>
4. <span data-ttu-id="97488-140">対象の期間中にアクセスされていたデータ ストアを識別します。</span><span class="sxs-lookup"><span data-stu-id="97488-140">Identify which data stores were accessed during those periods.</span></span>
5. <span data-ttu-id="97488-141">競合が発生している可能性のあるデータ ストレージ リソースを識別します。</span><span class="sxs-lookup"><span data-stu-id="97488-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="97488-142">診断の例</span><span class="sxs-lookup"><span data-stu-id="97488-142">Example diagnosis</span></span>

<span data-ttu-id="97488-143">以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。</span><span class="sxs-lookup"><span data-stu-id="97488-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="97488-144">システムをインストルメント化および監視する</span><span class="sxs-lookup"><span data-stu-id="97488-144">Instrument and monitor the system</span></span>

<span data-ttu-id="97488-145">次のグラフは、前述のサンプル アプリケーションのロード テストの結果を示しています。</span><span class="sxs-lookup"><span data-stu-id="97488-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="97488-146">テストでは、最大 1,000 人の同時実行ユーザーのステップ負荷を使用しました。</span><span class="sxs-lookup"><span data-stu-id="97488-146">The test used a step load of up to 1000 concurrent users.</span></span>

![SQL ベースのコントローラーのロード テストのパフォーマンス結果][MonolithicScenarioLoadTest]

<span data-ttu-id="97488-148">負荷が 700 ユーザーに増加するまでは、それに合わせてスループットも上がります。</span><span class="sxs-lookup"><span data-stu-id="97488-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="97488-149">しかし、その時点でスループットは横ばいになり、システムはその最大処理能力で動作しているように見えます。</span><span class="sxs-lookup"><span data-stu-id="97488-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="97488-150">ユーザーの負荷に応じて平均応答時間が徐々に上昇し、システムが需要に対応できないことがわかります。</span><span class="sxs-lookup"><span data-stu-id="97488-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="97488-151">パフォーマンスが低下している期間を識別する</span><span class="sxs-lookup"><span data-stu-id="97488-151">Identify periods of poor performance</span></span>

<span data-ttu-id="97488-152">運用システムを監視している場合、パターンに気付くかもしれません。</span><span class="sxs-lookup"><span data-stu-id="97488-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="97488-153">たとえば、毎日同じ時間に応答時間が大幅に低下しているとします。</span><span class="sxs-lookup"><span data-stu-id="97488-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="97488-154">このような状況は、通常のワークロードまたはスケジュールされたバッチ ジョブによって発生する可能性があるほか、単に特定の時間にシステムのユーザー数が増えるために発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="97488-155">こうしたイベントのテレメトリ データに注目する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97488-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="97488-156">応答時間の増加と、データベース アクティビティまたは共有リソースへの I/O の増加との間の相関関係を見つけてください。</span><span class="sxs-lookup"><span data-stu-id="97488-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="97488-157">相関関係がある場合、データベースがボトルネックになる可能性を意味します。</span><span class="sxs-lookup"><span data-stu-id="97488-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="97488-158">対象の期間中にアクセスされていたデータ ストアを識別する</span><span class="sxs-lookup"><span data-stu-id="97488-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="97488-159">次のグラフは、ロード テスト中のデータベース スループット単位 (DTU) の使用率を示しています </span><span class="sxs-lookup"><span data-stu-id="97488-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="97488-160">(DTU は使用可能な容量のメジャーであり、CPU 使用率、メモリ割り当て、I/O 率の組み合わせです)。DTU の使用率はすぐに 100% に達しています。</span><span class="sxs-lookup"><span data-stu-id="97488-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="97488-161">これは、前のグラフでスループットがピークに達したポイントとほぼ同じです。</span><span class="sxs-lookup"><span data-stu-id="97488-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="97488-162">データベースの使用率については、テストが終了するまで非常に高い値が維持されています。</span><span class="sxs-lookup"><span data-stu-id="97488-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="97488-163">最後の部分で若干の低下があります。これは、調整、データベース接続の競合、またはその他の要因が原因となっている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97488-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![Azure クラシック ポータルのデータベース モニターに表示された、データベースのリソース使用率][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="97488-165">データ ストアのテレメトリを調べる</span><span class="sxs-lookup"><span data-stu-id="97488-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="97488-166">アクティビティの低レベルの詳細をキャプチャするためにデータ ストアをインストルメント化します。</span><span class="sxs-lookup"><span data-stu-id="97488-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="97488-167">データ アクセス統計から、サンプル アプリケーションでは `PurchaseOrderHeader` テーブルと `MonoLog` テーブルの両方に対して大量の挿入操作が実行されていることがわかります。</span><span class="sxs-lookup"><span data-stu-id="97488-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span>

![サンプル アプリケーションのデータ アクセス統計][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="97488-169">リソースの競合を識別する</span><span class="sxs-lookup"><span data-stu-id="97488-169">Identify resource contention</span></span>

<span data-ttu-id="97488-170">この時点で、どのポイントでアプリケーションが競合するリソースにアクセスするかに注目して、ソース コードを確認することができます。</span><span class="sxs-lookup"><span data-stu-id="97488-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="97488-171">次のような状況を確認します。</span><span class="sxs-lookup"><span data-stu-id="97488-171">Look for situations such as:</span></span>

- <span data-ttu-id="97488-172">論理的に区別されるデータが同じストアに書き込まれている。</span><span class="sxs-lookup"><span data-stu-id="97488-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="97488-173">ログ、レポート、およびキューに入れられたメッセージなどのデータをビジネス情報と同じデータベースに保持しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="97488-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="97488-174">データ ストアの選択とデータの種類 (リレーショナル データベース内の大規模な BLOB または XML ドキュメントなど) のミスマッチ。</span><span class="sxs-lookup"><span data-stu-id="97488-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="97488-175">同じストアを共有しながら使用パターンが著しく異なるデータ (たとえば、書き込みが多く読み取りが少ないデータと書き込みが少なく読み取りが多いデータが一緒に格納されているケース)。</span><span class="sxs-lookup"><span data-stu-id="97488-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="97488-176">ソリューションを実装して結果を検証する</span><span class="sxs-lookup"><span data-stu-id="97488-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="97488-177">アプリケーションは、ログを別のデータ ストアに書き込むように変更されました。</span><span class="sxs-lookup"><span data-stu-id="97488-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="97488-178">ロード テストの結果を次に示します。</span><span class="sxs-lookup"><span data-stu-id="97488-178">Here are the load test results:</span></span>

![ポリグロット コントローラーを使用したロード テストのパフォーマンス結果][PolyglotScenarioLoadTest]

<span data-ttu-id="97488-180">スループットのパターンは以前のグラフと似ていますが、パフォーマンスのピーク ポイントは、1 秒あたり約 500 要求高くなっています。</span><span class="sxs-lookup"><span data-stu-id="97488-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="97488-181">平均応答時間は少し低くなっています。</span><span class="sxs-lookup"><span data-stu-id="97488-181">The average response time is marginally lower.</span></span> <span data-ttu-id="97488-182">ただし、これらの統計は完全なストーリーを表していません。</span><span class="sxs-lookup"><span data-stu-id="97488-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="97488-183">ビジネス データベースのテレメトリでは、DTU の使用率は 100% ではなく約 75% でピークに達しています。</span><span class="sxs-lookup"><span data-stu-id="97488-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![Azure クラシック ポータルのデータベース モニターに表示された、ポリグロット シナリオでのデータベースのリソース使用率][PolyglotDatabaseUtilization]

<span data-ttu-id="97488-185">同様に、ログ データベースの最大 DTU 使用率は約 70% に留まります。</span><span class="sxs-lookup"><span data-stu-id="97488-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="97488-186">データベースは、システムのパフォーマンスを制限する要因ではなくなりました。</span><span class="sxs-lookup"><span data-stu-id="97488-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![Azure クラシック ポータルのデータベース モニターに表示された、ポリグロット シナリオでのログ データベースのリソース使用率][LogDatabaseUtilization]

## <a name="related-resources"></a><span data-ttu-id="97488-188">関連リソース</span><span class="sxs-lookup"><span data-stu-id="97488-188">Related resources</span></span>

- <span data-ttu-id="97488-189">[適切なデータ ストアの選択][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="97488-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="97488-190">[データ ストアの選択条件][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="97488-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="97488-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence (拡張性の高いソリューション用のデータ アクセス: SQL、NoSQL、および Polyglot の永続化機能の使用)][Data-Access-Guide]</span><span class="sxs-lookup"><span data-stu-id="97488-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="97488-192">[データのパーティション分割][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="97488-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: https://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
