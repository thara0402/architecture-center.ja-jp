---
title: 再試行サービス固有のガイダンス
description: 再試行メカニズムを設定するためのサービス固有のガイダンスです。
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: c80a4aa232cca1283d84368a36dd7341cab8a314
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/29/2018
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="95469-103">特定のサービスの再試行ガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-103">Retry guidance for specific services</span></span>

<span data-ttu-id="95469-104">ほとんどの Azure サービスとクライアント SDK には、再試行メカニズムが組み込まれています。</span><span class="sxs-lookup"><span data-stu-id="95469-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="95469-105">しかし、再試行メカニズムは、サービスごとにさまざまな特性や要件があるため一定ではなく、それぞれの再試行メカニズムは特定のサービスに合わせて調整されます。</span><span class="sxs-lookup"><span data-stu-id="95469-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="95469-106">このガイドでは、主要な Azure サービスの再試行メカニズム機能の概要と、そのサービスに合わせて再試行メカニズムを使用、適合、または拡張するために役立つ情報を記載しています。</span><span class="sxs-lookup"><span data-stu-id="95469-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="95469-107">一時的なエラーの処理、およびサービスとリソースに対する接続と操作の再試行に関する一般的なガイダンスについては、「 [Retry general guidance (再試行の一般ガイダンス)](./transient-faults.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="95469-108">次の表は、このガイダンスで説明されている Azure サービスの再試行機能をまとめています。</span><span class="sxs-lookup"><span data-stu-id="95469-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="95469-109">**サービス**</span><span class="sxs-lookup"><span data-stu-id="95469-109">**Service**</span></span> | <span data-ttu-id="95469-110">**再試行機能**</span><span class="sxs-lookup"><span data-stu-id="95469-110">**Retry capabilities**</span></span> | <span data-ttu-id="95469-111">**ポリシーの構成**</span><span class="sxs-lookup"><span data-stu-id="95469-111">**Policy configuration**</span></span> | <span data-ttu-id="95469-112">**スコープ**</span><span class="sxs-lookup"><span data-stu-id="95469-112">**Scope**</span></span> | <span data-ttu-id="95469-113">**テレメトリ機能**</span><span class="sxs-lookup"><span data-stu-id="95469-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="95469-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="95469-115">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-115">Native in client</span></span> |<span data-ttu-id="95469-116">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-116">Programmatic</span></span> |<span data-ttu-id="95469-117">クライアントと個々の操作</span><span class="sxs-lookup"><span data-stu-id="95469-117">Client and individual operations</span></span> |<span data-ttu-id="95469-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="95469-118">TraceSource</span></span> |
| <span data-ttu-id="95469-119">**[Entity Framework を使用した SQL Database](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="95469-120">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-120">Native in client</span></span> |<span data-ttu-id="95469-121">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-121">Programmatic</span></span> |<span data-ttu-id="95469-122">AppDomain ごとにグローバル</span><span class="sxs-lookup"><span data-stu-id="95469-122">Global per AppDomain</span></span> |<span data-ttu-id="95469-123">なし</span><span class="sxs-lookup"><span data-stu-id="95469-123">None</span></span> |
| <span data-ttu-id="95469-124">**[Entity Framework Core を使用した SQL Database](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="95469-125">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-125">Native in client</span></span> |<span data-ttu-id="95469-126">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-126">Programmatic</span></span> |<span data-ttu-id="95469-127">AppDomain ごとにグローバル</span><span class="sxs-lookup"><span data-stu-id="95469-127">Global per AppDomain</span></span> |<span data-ttu-id="95469-128">なし</span><span class="sxs-lookup"><span data-stu-id="95469-128">None</span></span> |
| <span data-ttu-id="95469-129">**[ADO.NET を使用した SQL Database](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="95469-130">Polly</span><span class="sxs-lookup"><span data-stu-id="95469-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="95469-131">宣言型およびプログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-131">Declarative and programmatic</span></span> |<span data-ttu-id="95469-132">1 つのステートメントまたはコードのブロック</span><span class="sxs-lookup"><span data-stu-id="95469-132">Single statements or blocks of code</span></span> |<span data-ttu-id="95469-133">カスタム</span><span class="sxs-lookup"><span data-stu-id="95469-133">Custom</span></span> |
| <span data-ttu-id="95469-134">**[Service Bus](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="95469-135">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-135">Native in client</span></span> |<span data-ttu-id="95469-136">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-136">Programmatic</span></span> |<span data-ttu-id="95469-137">名前空間マネージャー、メッセージング ファクトリ、およびクライアント</span><span class="sxs-lookup"><span data-stu-id="95469-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="95469-138">ETW</span><span class="sxs-lookup"><span data-stu-id="95469-138">ETW</span></span> |
| <span data-ttu-id="95469-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="95469-140">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-140">Native in client</span></span> |<span data-ttu-id="95469-141">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-141">Programmatic</span></span> |<span data-ttu-id="95469-142">クライアント</span><span class="sxs-lookup"><span data-stu-id="95469-142">Client</span></span> |<span data-ttu-id="95469-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="95469-143">TextWriter</span></span> |
| <span data-ttu-id="95469-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span></span> |<span data-ttu-id="95469-145">サービスでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-145">Native in service</span></span> |<span data-ttu-id="95469-146">構成不可</span><span class="sxs-lookup"><span data-stu-id="95469-146">Non-configurable</span></span> |<span data-ttu-id="95469-147">グローバル</span><span class="sxs-lookup"><span data-stu-id="95469-147">Global</span></span> |<span data-ttu-id="95469-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="95469-148">TraceSource</span></span> |
| <span data-ttu-id="95469-149">**[Azure Search](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="95469-150">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-150">Native in client</span></span> |<span data-ttu-id="95469-151">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-151">Programmatic</span></span> |<span data-ttu-id="95469-152">クライアント</span><span class="sxs-lookup"><span data-stu-id="95469-152">Client</span></span> |<span data-ttu-id="95469-153">ETW またはカスタム</span><span class="sxs-lookup"><span data-stu-id="95469-153">ETW or Custom</span></span> |
| <span data-ttu-id="95469-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="95469-155">ADAL ライブラリのネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-155">Native in ADAL library</span></span> |<span data-ttu-id="95469-156">ADAL ライブラリに埋め込み済み</span><span class="sxs-lookup"><span data-stu-id="95469-156">Embeded into ADAL library</span></span> |<span data-ttu-id="95469-157">内部</span><span class="sxs-lookup"><span data-stu-id="95469-157">Internal</span></span> |<span data-ttu-id="95469-158">なし</span><span class="sxs-lookup"><span data-stu-id="95469-158">None</span></span> |
| <span data-ttu-id="95469-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="95469-160">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-160">Native in client</span></span> |<span data-ttu-id="95469-161">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-161">Programmatic</span></span> |<span data-ttu-id="95469-162">クライアント</span><span class="sxs-lookup"><span data-stu-id="95469-162">Client</span></span> |<span data-ttu-id="95469-163">なし</span><span class="sxs-lookup"><span data-stu-id="95469-163">None</span></span> | 
| <span data-ttu-id="95469-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="95469-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="95469-165">クライアントでネイティブ</span><span class="sxs-lookup"><span data-stu-id="95469-165">Native in client</span></span> |<span data-ttu-id="95469-166">プログラムによる</span><span class="sxs-lookup"><span data-stu-id="95469-166">Programmatic</span></span> |<span data-ttu-id="95469-167">クライアント</span><span class="sxs-lookup"><span data-stu-id="95469-167">Client</span></span> |<span data-ttu-id="95469-168">なし</span><span class="sxs-lookup"><span data-stu-id="95469-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="95469-169">Azure の組み込み再試行メカニズムのほとんどには、再試行ポリシーに組み込まれている機能以外に、さまざまな種類のエラーや例外に対して異なる再試行ポリシーを適用する方法は現在のところありません。</span><span class="sxs-lookup"><span data-stu-id="95469-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="95469-170">したがって、この記事の執筆時点で選択できる最良のガイダンスは、最適な平均的パフォーマンスと可用性を提供するポリシーを構成することです。</span><span class="sxs-lookup"><span data-stu-id="95469-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="95469-171">ポリシーを微調整する 1 つの方法は、ログ ファイルを分析して、発生する一時的エラーの種類を判別することです。</span><span class="sxs-lookup"><span data-stu-id="95469-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="95469-172">たとえば、エラーの多くがネットワーク接続問題に関連している場合、最初の再試行を長時間待つのではなく、すぐに再試行することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="95469-173">Microsoft Azure Storage の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="95469-174">Microsoft Azure Storage サービスには、テーブル、Blob Storage、ファイル、およびストレージ キューが含まれています。</span><span class="sxs-lookup"><span data-stu-id="95469-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-175">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-175">Retry mechanism</span></span>
<span data-ttu-id="95469-176">再試行は、個々の REST 操作レベルで実行され、クライアント API 実装の不可欠な部分です。</span><span class="sxs-lookup"><span data-stu-id="95469-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="95469-177">クライアント ストレージ SDK は、 [IExtendedRetryPolicy インターフェイス](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx)を実装するクラスを使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="95469-178">このインターフェイスにはさまざまな実装があります。</span><span class="sxs-lookup"><span data-stu-id="95469-178">There are different implementations of the interface.</span></span> <span data-ttu-id="95469-179">ストレージ クライアントは、ポリシーを、テーブル、BLOB、およびキューにアクセスするために特別に設計されたものの中から選択できます。</span><span class="sxs-lookup"><span data-stu-id="95469-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="95469-180">各実装は、基本的に、再試行間隔や他の詳細を定義する、それぞれに異なる再試行戦略を使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="95469-181">組み込みクラスは、Linear (一定遅延) と、ランダム化された再試行間隔が指定される Exponential をサポートします。</span><span class="sxs-lookup"><span data-stu-id="95469-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="95469-182">別のプロセスがより高いレベルで再試行を処理している場合に使用する、再試行なしポリシーもあります。</span><span class="sxs-lookup"><span data-stu-id="95469-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="95469-183">ただし、組み込みクラスによって提供されていない特定の要件がある場合は、独自の再試行クラスを実装できます。</span><span class="sxs-lookup"><span data-stu-id="95469-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="95469-184">代替再試行では、読み取りアクセス geo 冗長ストレージ (RA-GRS) を使用しており、要求の結果が再試行可能エラーになる場合は、プライマリとセカンダリのストレージ サービス場所での切り替えが行われます。</span><span class="sxs-lookup"><span data-stu-id="95469-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="95469-185">詳細については、「 [Azure Storage 冗長オプション](http://msdn.microsoft.com/library/azure/dn727290.aspx) 」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="95469-186">ポリシーの構成</span><span class="sxs-lookup"><span data-stu-id="95469-186">Policy configuration</span></span>
<span data-ttu-id="95469-187">再試行ポリシーは、プログラムにより構成されます。</span><span class="sxs-lookup"><span data-stu-id="95469-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="95469-188">一般的なプロシージャでは、**TableRequestOptions**、**BlobRequestOptions**、**FileRequestOptions**、または **QueueRequestOptions** の各インスタンスが作成されて設定されます。</span><span class="sxs-lookup"><span data-stu-id="95469-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="95469-189">要求オプション インスタンスは、クライアントに対して設定することができ、そのクライアントからのすべての操作は、指定された要求オプションを使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="95469-190">クライアント要求オプションは、要求オプション クラスの設定済みインスタンスを操作メソッドのパラメーターとして渡すことによって、オーバーライドできます。</span><span class="sxs-lookup"><span data-stu-id="95469-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="95469-191">**OperationContext** インスタンスは、再試行が行われたときと操作が完了したときに実行するコードの指定に使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="95469-192">このコードは、ログとテレメトリで使用する、操作に関する情報を収集できます。</span><span class="sxs-lookup"><span data-stu-id="95469-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

<span data-ttu-id="95469-193">さらに、エラーに対して再試行が適切であるかどうかを示すために、拡張再試行ポリシーは、再試行回数、前回の要求の結果、次回の再試行が行われる場所 (プライマリまたはセカンダリ) を示す、 **RetryContext** オブジェクトを返します (詳細については、次の表を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="95469-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="95469-194">**RetryContext** オブジェクトのプロパティは、再試行を行うかどうか、およびいつ行うかを決定するために使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="95469-195">詳細については、 [IExtendedRetryPolicy.Evaluate メソッド](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="95469-196">次の表は、組み込み再試行ポリシーの既定の設定を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-196">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="95469-197">**要求のオプション**</span><span class="sxs-lookup"><span data-stu-id="95469-197">**Request options**</span></span>

| <span data-ttu-id="95469-198">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-198">**Setting**</span></span> | <span data-ttu-id="95469-199">**既定値**</span><span class="sxs-lookup"><span data-stu-id="95469-199">**Default value**</span></span> | <span data-ttu-id="95469-200">**意味**</span><span class="sxs-lookup"><span data-stu-id="95469-200">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="95469-201">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="95469-201">MaximumExecutionTime</span></span> | <span data-ttu-id="95469-202">120 秒</span><span class="sxs-lookup"><span data-stu-id="95469-202">120 seconds</span></span> | <span data-ttu-id="95469-203">要求の最大実行時間 (考えられるすべての再試行が含まれます)。</span><span class="sxs-lookup"><span data-stu-id="95469-203">Maximum execution time for the request, including all potential retry attempts.</span></span> |
| <span data-ttu-id="95469-204">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="95469-204">ServerTimeout</span></span> | <span data-ttu-id="95469-205">なし</span><span class="sxs-lookup"><span data-stu-id="95469-205">None</span></span> | <span data-ttu-id="95469-206">要求のサーバー タイムアウト間隔 (値は秒単位に丸められます)。</span><span class="sxs-lookup"><span data-stu-id="95469-206">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="95469-207">指定しない場合、サーバーに対するすべての要求に既定値が使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-207">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="95469-208">通常、この設定を省略してサーバーの既定値が使用されるようにすることが最善のオプションになります。</span><span class="sxs-lookup"><span data-stu-id="95469-208">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="95469-209">LocationMode</span><span class="sxs-lookup"><span data-stu-id="95469-209">LocationMode</span></span> | <span data-ttu-id="95469-210">なし</span><span class="sxs-lookup"><span data-stu-id="95469-210">None</span></span> | <span data-ttu-id="95469-211">ストレージ アカウントが読み取りアクセス geo 冗長ストレージ (RA-GRS) のレプリケーション オプションを指定して作成されている場合、場所モードを使用して、要求を受け取る場所を示すことができます。</span><span class="sxs-lookup"><span data-stu-id="95469-211">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="95469-212">たとえば、**PrimaryThenSecondary** を指定した場合、要求は必ず最初にプライマリの場所に送信されます。</span><span class="sxs-lookup"><span data-stu-id="95469-212">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="95469-213">失敗した場合、要求はセカンダリの場所に送信されます。</span><span class="sxs-lookup"><span data-stu-id="95469-213">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="95469-214">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="95469-214">RetryPolicy</span></span> | <span data-ttu-id="95469-215">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="95469-215">ExponentialPolicy</span></span> | <span data-ttu-id="95469-216">各オプションの詳細については、以下をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="95469-216">See below for details of each option.</span></span> |

<span data-ttu-id="95469-217">**Exponential ポリシー**</span><span class="sxs-lookup"><span data-stu-id="95469-217">**Exponential policy**</span></span> 

| <span data-ttu-id="95469-218">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-218">**Setting**</span></span> | <span data-ttu-id="95469-219">**既定値**</span><span class="sxs-lookup"><span data-stu-id="95469-219">**Default value**</span></span> | <span data-ttu-id="95469-220">**意味**</span><span class="sxs-lookup"><span data-stu-id="95469-220">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="95469-221">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="95469-221">maxAttempt</span></span> | <span data-ttu-id="95469-222">3</span><span class="sxs-lookup"><span data-stu-id="95469-222">3</span></span> | <span data-ttu-id="95469-223">再試行回数。</span><span class="sxs-lookup"><span data-stu-id="95469-223">Number of retry attempts.</span></span> |
| <span data-ttu-id="95469-224">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-224">deltaBackoff</span></span> | <span data-ttu-id="95469-225">4 秒</span><span class="sxs-lookup"><span data-stu-id="95469-225">4 seconds</span></span> | <span data-ttu-id="95469-226">再試行のバックオフ間隔。</span><span class="sxs-lookup"><span data-stu-id="95469-226">Back-off interval between retries.</span></span> <span data-ttu-id="95469-227">この期間のランダムな倍数が、後続の再試行に使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-227">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="95469-228">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-228">MinBackoff</span></span> | <span data-ttu-id="95469-229">3 秒</span><span class="sxs-lookup"><span data-stu-id="95469-229">3 seconds</span></span> | <span data-ttu-id="95469-230">deltaBackoff から計算されたすべての再試行間隔に追加されます。</span><span class="sxs-lookup"><span data-stu-id="95469-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="95469-231">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-231">This value cannot be changed.</span></span>
| <span data-ttu-id="95469-232">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-232">MaxBackoff</span></span> | <span data-ttu-id="95469-233">120 秒</span><span class="sxs-lookup"><span data-stu-id="95469-233">120 seconds</span></span> | <span data-ttu-id="95469-234">MaxBackoff は、計算された再試行間隔が MaxBackoff より大きい場合に使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-234">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="95469-235">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-235">This value cannot be changed.</span></span> |

<span data-ttu-id="95469-236">**Linear ポリシー**</span><span class="sxs-lookup"><span data-stu-id="95469-236">**Linear policy**</span></span>

| <span data-ttu-id="95469-237">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-237">**Setting**</span></span> | <span data-ttu-id="95469-238">**既定値**</span><span class="sxs-lookup"><span data-stu-id="95469-238">**Default value**</span></span> | <span data-ttu-id="95469-239">**意味**</span><span class="sxs-lookup"><span data-stu-id="95469-239">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="95469-240">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="95469-240">maxAttempt</span></span> | <span data-ttu-id="95469-241">3</span><span class="sxs-lookup"><span data-stu-id="95469-241">3</span></span> | <span data-ttu-id="95469-242">再試行回数。</span><span class="sxs-lookup"><span data-stu-id="95469-242">Number of retry attempts.</span></span> |
| <span data-ttu-id="95469-243">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-243">deltaBackoff</span></span> | <span data-ttu-id="95469-244">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-244">30 seconds</span></span> | <span data-ttu-id="95469-245">再試行のバックオフ間隔。</span><span class="sxs-lookup"><span data-stu-id="95469-245">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-246">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-246">Retry usage guidance</span></span>
<span data-ttu-id="95469-247">ストレージ クライアント API を使用して Microsoft Azure Storage サービスにアクセスする場合は、次のガイドラインを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-247">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="95469-248">Microsoft.WindowsAzure.Storage.RetryPolicies 名前空間からの組み込み再試行ポリシーを使用します (これらのポリシーが要件に適している場合)。</span><span class="sxs-lookup"><span data-stu-id="95469-248">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="95469-249">ほとんどの場合、これらのポリシーで十分対応できます。</span><span class="sxs-lookup"><span data-stu-id="95469-249">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="95469-250">バッチ操作、バックグラウンド タスク、または非対話型のシナリオでは、**ExponentialRetry** ポリシーを使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-250">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="95469-251">これらのシナリオでは、一般に、サービスを復旧するためにより多くの時間を確保できます。結果として、操作は最終的に成功する可能性が高くなります。</span><span class="sxs-lookup"><span data-stu-id="95469-251">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="95469-252">合計実行時間を制限するには、**RequestOptions** パラメーターの **MaximumExecutionTime** プロパティを指定することを検討してください。ただし、タイムアウト値を選択する場合は、操作の種類とサイズを考慮に入れてください。</span><span class="sxs-lookup"><span data-stu-id="95469-252">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="95469-253">カスタム再試行を実装する必要がある場合は、ストレージ クライアント クラスを囲むラッパーは作成しないでください。</span><span class="sxs-lookup"><span data-stu-id="95469-253">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="95469-254">代わりに **IExtendedRetryPolicy** インターフェイスから、既存のポリシーを拡張する機能を使用してください。</span><span class="sxs-lookup"><span data-stu-id="95469-254">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="95469-255">読み取りアクセス geo 冗長ストレージ (RA-GRS) を使用している場合、 **LocationMode** を使用して、プライマリへのアクセスが失敗した場合に、再試行がストアのセカンダリ読み取り専用コピーにアクセスするように指定できます。</span><span class="sxs-lookup"><span data-stu-id="95469-255">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="95469-256">ただし、このオプションを使用するときは、プライマリ ストアからのレプリケーションがまだ完了していない場合に、古くなった可能性があるデータを用いてアプリケーションが正常に動作できることを確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-256">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="95469-257">再試行操作について次の設定から始めることを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-257">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="95469-258">これらは汎用の設定であり、操作を監視して、独自のシナリオに合うように値を微調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-258">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="95469-259">**コンテキスト**</span><span class="sxs-lookup"><span data-stu-id="95469-259">**Context**</span></span> | <span data-ttu-id="95469-260">**サンプルのターゲット E2E<br />最大待機時間**</span><span class="sxs-lookup"><span data-stu-id="95469-260">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="95469-261">**再試行ポリシー**</span><span class="sxs-lookup"><span data-stu-id="95469-261">**Retry policy**</span></span> | <span data-ttu-id="95469-262">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-262">**Settings**</span></span> | <span data-ttu-id="95469-263">**値**</span><span class="sxs-lookup"><span data-stu-id="95469-263">**Values**</span></span> | <span data-ttu-id="95469-264">**動作のしくみ**</span><span class="sxs-lookup"><span data-stu-id="95469-264">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="95469-265">対話型、UI、</span><span class="sxs-lookup"><span data-stu-id="95469-265">Interactive, UI,</span></span><br /><span data-ttu-id="95469-266">またはフォアグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-266">or foreground</span></span> |<span data-ttu-id="95469-267">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-267">2 seconds</span></span> |<span data-ttu-id="95469-268">Linear</span><span class="sxs-lookup"><span data-stu-id="95469-268">Linear</span></span> |<span data-ttu-id="95469-269">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="95469-269">maxAttempt</span></span><br /><span data-ttu-id="95469-270">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-270">deltaBackoff</span></span> |<span data-ttu-id="95469-271">3</span><span class="sxs-lookup"><span data-stu-id="95469-271">3</span></span><br /><span data-ttu-id="95469-272">500 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-272">500 ms</span></span> |<span data-ttu-id="95469-273">試行 1 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-273">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="95469-274">試行 2 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-274">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="95469-275">試行 3 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-275">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="95469-276">バックグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-276">Background</span></span><br /><span data-ttu-id="95469-277">またはバッチ</span><span class="sxs-lookup"><span data-stu-id="95469-277">or batch</span></span> |<span data-ttu-id="95469-278">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-278">30 seconds</span></span> |<span data-ttu-id="95469-279">Exponential</span><span class="sxs-lookup"><span data-stu-id="95469-279">Exponential</span></span> |<span data-ttu-id="95469-280">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="95469-280">maxAttempt</span></span><br /><span data-ttu-id="95469-281">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-281">deltaBackoff</span></span> |<span data-ttu-id="95469-282">5</span><span class="sxs-lookup"><span data-stu-id="95469-282">5</span></span><br /><span data-ttu-id="95469-283">4 秒</span><span class="sxs-lookup"><span data-stu-id="95469-283">4 seconds</span></span> |<span data-ttu-id="95469-284">試行 1 - 最大 3 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-284">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="95469-285">試行 2 - 最大 7 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-285">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="95469-286">試行 3 - 最大 15 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-286">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="95469-287">テレメトリ</span><span class="sxs-lookup"><span data-stu-id="95469-287">Telemetry</span></span>
<span data-ttu-id="95469-288">再試行回数は **TraceSource**に記録されます。</span><span class="sxs-lookup"><span data-stu-id="95469-288">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="95469-289">イベントをキャプチャし、それらを適切な宛先ログに書き込むには、 **TraceListener** を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-289">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="95469-290">データをログ ファイルに書き込むには **TextWriterTraceListener** または **XmlWriterTraceListener** を、Windows イベント ログに書き込むには **EventLogTraceListener** を、トレース データを ETW サブシステムに書き込むには **EventProviderTraceListener** をそれぞれ使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-290">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="95469-291">バッファーの自動フラッシュと、ログに記録するイベントの詳細度 (たとえば、エラー、警告、情報、および冗長など) を構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="95469-291">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="95469-292">詳細については、「[.NET ストレージ クライアント ライブラリによるクライアント側のログ](http://msdn.microsoft.com/library/azure/dn782839.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-292">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="95469-293">操作は **OperationContext** インスタンスを受け取る可能性があります。これはカスタム テレメトリ ロジックをアタッチするために使用できる **Retrying** イベントを公開します。</span><span class="sxs-lookup"><span data-stu-id="95469-293">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="95469-294">詳細については、「[OperationContext.Retrying Event (OperationContext.Retrying イベント)](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-294">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="95469-295">例</span><span class="sxs-lookup"><span data-stu-id="95469-295">Examples</span></span>
<span data-ttu-id="95469-296">次のコード例は、異なる再試行設定で 2 つの **TableRequestOptions** インスタンスを作成する方法を示しています。1 つは対話型の要求向け、1 つはバックグラウンドの要求向けです。</span><span class="sxs-lookup"><span data-stu-id="95469-296">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="95469-297">次にこの例では、これら 2 つの再試行ポリシーをクライアントに対して設定して、それらのポリシーがすべての要求に適用されるようにします。さらに、対話型戦略を特定の要求に対して設定して、クライアントに適用された既定の設定をオーバーライドできるようにします。</span><span class="sxs-lookup"><span data-stu-id="95469-297">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="95469-298">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-298">More information</span></span>
* [<span data-ttu-id="95469-299">Azure Storage Client Library Retry Policy Recommendations (Microsoft Azure Storage クライアント ライブラリの再試行ポリシーに対する推奨事項)</span><span class="sxs-lookup"><span data-stu-id="95469-299">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="95469-300">Storage Client Library 2.0 – Implementing Retry Policies (Storage Client Library 2.0 – 再試行ポリシーの実装)</span><span class="sxs-lookup"><span data-stu-id="95469-300">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="95469-301">Entity Framework 6 を使用して SQL Database にアクセスする場合の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-301">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="95469-302">SQL Database は、多様なサイズで利用できるホステッド SQL データベースです。これは Standard (共有) サービスと Premium (非共有) サービスのどちらとしてでも使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-302">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="95469-303">Entity Framework は、.NET 開発者がドメイン固有オブジェクトを使用してリレーショナル データを操作できるようにする、オブジェクトリレーショナル マッパーです。</span><span class="sxs-lookup"><span data-stu-id="95469-303">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="95469-304">これにより、開発者が通常は記述する必要のあるデータアクセス コードの大部分が不要になります。</span><span class="sxs-lookup"><span data-stu-id="95469-304">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-305">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-305">Retry mechanism</span></span>
<span data-ttu-id="95469-306">再試行サポートは、SQL Database データベースに、Entity Framework 6.0 以降で [接続回復/再試行ロジック](http://msdn.microsoft.com/data/dn456835.aspx)というメカニズムを用いてアクセスするときに提供されます。</span><span class="sxs-lookup"><span data-stu-id="95469-306">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="95469-307">再試行メカニズムの主な機能は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="95469-307">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="95469-308">主要な抽象化は、 **IDbExecutionStrategy** インターフェイスです。</span><span class="sxs-lookup"><span data-stu-id="95469-308">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="95469-309">このインターフェイスは、次の事柄を実行します。</span><span class="sxs-lookup"><span data-stu-id="95469-309">This interface:</span></span>
  * <span data-ttu-id="95469-310">同期および非同期の **Execute**\* メソッドを定義します。</span><span class="sxs-lookup"><span data-stu-id="95469-310">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="95469-311">既定の戦略として、直接使用できるクラス、またはデータベース コンテキストに基づいて構成できるクラスを定義します。このクラスは、プロバイダー名にマップされるかまたはプロバイダー名とサーバー名にマップされます。</span><span class="sxs-lookup"><span data-stu-id="95469-311">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="95469-312">コンテキストに基づいて構成された場合、再試行は個々のデータベース操作のレベルで行われます。特定の 1 コンテキスト操作に対して複数の再試行が行われる場合もあります。</span><span class="sxs-lookup"><span data-stu-id="95469-312">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="95469-313">失敗した接続をいつ、どのように再試行するかを定義します。</span><span class="sxs-lookup"><span data-stu-id="95469-313">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="95469-314">これには、 **IDbExecutionStrategy** インターフェイスの次のいくつかの組み込み実装が含まれます。</span><span class="sxs-lookup"><span data-stu-id="95469-314">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="95469-315">既定値 - 再試行なし。</span><span class="sxs-lookup"><span data-stu-id="95469-315">Default - no retrying.</span></span>
  * <span data-ttu-id="95469-316">SQL Database の既定値 (自動) - 再試行なし。ただし例外を検査し、SQL Database 戦略を使用するという提案でそれらをラップします。</span><span class="sxs-lookup"><span data-stu-id="95469-316">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="95469-317">SQL Database の既定値 - Exponential (基本クラスからの継承) に、SQL Database の検出ロジックを加えたもの。</span><span class="sxs-lookup"><span data-stu-id="95469-317">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="95469-318">ランダム化を含む指数バックオフ戦略を実装します。</span><span class="sxs-lookup"><span data-stu-id="95469-318">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="95469-319">組み込み再試行クラスは、ステートフルですが、スレッド セーフではありません。</span><span class="sxs-lookup"><span data-stu-id="95469-319">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="95469-320">ただし、このクラスは現在の操作が完了した後に再利用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-320">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="95469-321">指定した再試行回数を超えた場合、結果は新しい例外でラップされます。</span><span class="sxs-lookup"><span data-stu-id="95469-321">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="95469-322">これは現在の例外をバブルアップしません。</span><span class="sxs-lookup"><span data-stu-id="95469-322">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="95469-323">ポリシーの構成</span><span class="sxs-lookup"><span data-stu-id="95469-323">Policy configuration</span></span>
<span data-ttu-id="95469-324">再試行サポートは、Entity Framework 6.0 以降を使用して SQL Database にアクセスする場合に提供されます。</span><span class="sxs-lookup"><span data-stu-id="95469-324">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="95469-325">再試行ポリシーは、プログラムにより構成されます。</span><span class="sxs-lookup"><span data-stu-id="95469-325">Retry policies are configured programmatically.</span></span> <span data-ttu-id="95469-326">構成を操作ごとに変更することはできません。</span><span class="sxs-lookup"><span data-stu-id="95469-326">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="95469-327">コンテキストに基づいて既定値として戦略を構成する場合は、新しい戦略をオンデマンドで作成する機能を指定します。</span><span class="sxs-lookup"><span data-stu-id="95469-327">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="95469-328">次のコードは、 **DbConfiguration** 基本クラスを拡張する、再試行構成クラスを作成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-328">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="95469-329">アプリケーションを起動したときに **DbConfiguration** インスタンスの **SetConfiguration** メソッドを使用して、すべての操作に対する既定の再試行戦略として、これを指定することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-329">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="95469-330">既定では、EF は構成クラスを自動的に検出して使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-330">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="95469-331">コンテキスト クラスに **DbConfigurationType** 属性で注釈を付けることにより、コンテキストに対して再試行構成クラスを指定できます。</span><span class="sxs-lookup"><span data-stu-id="95469-331">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="95469-332">ただし、構成クラスが 1 つしかない場合、EF はコンテキストに注釈を付けずにそれを使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-332">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="95469-333">特定の操作に対して異なる再試行戦略を使用する必要があるか、または特定の操作に対する再試行を無効にする必要がある場合、 **CallContext**にフラグを設定することで、戦略を中断またはスワップできる構成クラスを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-333">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="95469-334">構成クラスでは、このフラグを使用して、戦略を切り替えたり、指定した戦略を無効にして既定の戦略を使用したりできます。</span><span class="sxs-lookup"><span data-stu-id="95469-334">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="95469-335">詳細については、「Limitations with Retrying Execution Strategies (EF6 onwards) (再試行実行戦略の制限 (EF6 以降))」のページにある「 [Suspend Execution Strategy (実行戦略の中断)](http://msdn.microsoft.com/dn307226#transactions_workarounds) 」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-335">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="95469-336">個々の操作に特定の再試行戦略を使用する別の手法として、必要な戦略クラスのインスタンスを作成し、パラメーターにより目的の設定を指定することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-336">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="95469-337">次に、その **ExecuteAsync** メソッドを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="95469-337">You then invoke its **ExecuteAsync** method.</span></span>

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

<span data-ttu-id="95469-338">**DbConfiguration** クラスを使用する最も簡単な方法は、それを **DbContext** クラスと同じアセンブリ内に配置することです。</span><span class="sxs-lookup"><span data-stu-id="95469-338">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="95469-339">ただし、異なるシナリオ (対話型やバックグラウンドでの再試行戦略が異なるなど) で同じコンテキストが必要な場合には、これは適しません。</span><span class="sxs-lookup"><span data-stu-id="95469-339">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="95469-340">異なるコンテキストを別の Appdomain で実行する場合は、構成ファイル内で構成クラスを指定するために組み込みサポートを使用するか、またはコードを使用して構成クラスを明示的に設定することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-340">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="95469-341">異なる複数のコンテキストを同じ AppDomain 内で実行する必要がある場合には、カスタム ソリューションが必要です。</span><span class="sxs-lookup"><span data-stu-id="95469-341">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="95469-342">詳細については、「 [Code-Based Configuration (EF6 onwards) (コード ベースの構成 (EF6 以降))](http://msdn.microsoft.com/data/jj680699.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-342">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="95469-343">次の表は、EF6 を使用している場合の、組み込み再試行ポリシーの既定の設定を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-343">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="95469-344">Setting</span><span class="sxs-lookup"><span data-stu-id="95469-344">Setting</span></span> | <span data-ttu-id="95469-345">既定値</span><span class="sxs-lookup"><span data-stu-id="95469-345">Default value</span></span> | <span data-ttu-id="95469-346">意味</span><span class="sxs-lookup"><span data-stu-id="95469-346">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="95469-347">ポリシー</span><span class="sxs-lookup"><span data-stu-id="95469-347">Policy</span></span> | <span data-ttu-id="95469-348">指数</span><span class="sxs-lookup"><span data-stu-id="95469-348">Exponential</span></span> | <span data-ttu-id="95469-349">指数バックオフ。</span><span class="sxs-lookup"><span data-stu-id="95469-349">Exponential back-off.</span></span> |
| <span data-ttu-id="95469-350">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="95469-350">MaxRetryCount</span></span> | <span data-ttu-id="95469-351">5</span><span class="sxs-lookup"><span data-stu-id="95469-351">5</span></span> | <span data-ttu-id="95469-352">最大再試行回数。</span><span class="sxs-lookup"><span data-stu-id="95469-352">The maximum number of retries.</span></span> |
| <span data-ttu-id="95469-353">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="95469-353">MaxDelay</span></span> | <span data-ttu-id="95469-354">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-354">30 seconds</span></span> | <span data-ttu-id="95469-355">再試行間の最大遅延。</span><span class="sxs-lookup"><span data-stu-id="95469-355">The maximum delay between retries.</span></span> <span data-ttu-id="95469-356">この値は一連の遅延の計算方法には影響しません。</span><span class="sxs-lookup"><span data-stu-id="95469-356">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="95469-357">上限値のみが定義されます。</span><span class="sxs-lookup"><span data-stu-id="95469-357">It only defines an upper bound.</span></span> |
| <span data-ttu-id="95469-358">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="95469-358">DefaultCoefficient</span></span> | <span data-ttu-id="95469-359">1 秒</span><span class="sxs-lookup"><span data-stu-id="95469-359">1 second</span></span> | <span data-ttu-id="95469-360">指数バックオフ計算の係数。</span><span class="sxs-lookup"><span data-stu-id="95469-360">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="95469-361">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-361">This value cannot be changed.</span></span> |
| <span data-ttu-id="95469-362">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="95469-362">DefaultRandomFactor</span></span> | <span data-ttu-id="95469-363">1.1</span><span class="sxs-lookup"><span data-stu-id="95469-363">1.1</span></span> | <span data-ttu-id="95469-364">各エントリのランダム遅延を追加するために使用する乗数。</span><span class="sxs-lookup"><span data-stu-id="95469-364">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="95469-365">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-365">This value cannot be changed.</span></span> |
| <span data-ttu-id="95469-366">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="95469-366">DefaultExponentialBase</span></span> | <span data-ttu-id="95469-367">2</span><span class="sxs-lookup"><span data-stu-id="95469-367">2</span></span> | <span data-ttu-id="95469-368">次の遅延を計算するために使用する乗数。</span><span class="sxs-lookup"><span data-stu-id="95469-368">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="95469-369">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-369">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-370">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-370">Retry usage guidance</span></span>
<span data-ttu-id="95469-371">EF6 を使用して SQL Database にアクセスする場合は、次のガイドラインを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-371">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="95469-372">適切なサービス オプション (Shared または Premium) を選択します。</span><span class="sxs-lookup"><span data-stu-id="95469-372">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="95469-373">共有インスタンスは、共有サーバーの他のテナントによる使用状況により、通常よりも長い接続遅延や調整の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-373">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="95469-374">予測可能なパフォーマンスと信頼性の高い低待機時間での操作が必要な場合は、Premium オプションを選択することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-374">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="95469-375">固定間隔戦略を Azure SQL Database で使用することは推奨されていません。</span><span class="sxs-lookup"><span data-stu-id="95469-375">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="95469-376">代わりに、指数バックオフ戦略を使用します。サービスがオーバーロードする可能性があり、遅延が長くなれば回復するための時間をより多くとることができるからです。</span><span class="sxs-lookup"><span data-stu-id="95469-376">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="95469-377">接続を定義するときは、接続タイムアウトとコマンド タイムアウトに適切な値を選択します。</span><span class="sxs-lookup"><span data-stu-id="95469-377">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="95469-378">タイムアウトは、ビジネス ロジック設計と十分なテストの両方に基づいた値にします。</span><span class="sxs-lookup"><span data-stu-id="95469-378">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="95469-379">この値は、時間の経過によるデータの量やビジネス プロセスの変化に応じて、変更することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="95469-379">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="95469-380">タイムアウトが短すぎると、データベースがビジー状態の場合に、接続が途中でエラーになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-380">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="95469-381">タイムアウトが長すぎると、接続エラーを検出するまで長く待ちすぎて、再試行ロジックが正常に機能しなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-381">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="95469-382">コンテキストの保存時に実行されるコマンドの数は簡単には特定できないとしても、タイムアウトの値は、エンドツーエンド待機時間の構成要素です。</span><span class="sxs-lookup"><span data-stu-id="95469-382">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="95469-383">既定のタイムアウトは、**DbContext** インスタンスの **CommandTimeout** プロパティを設定することで変更できます。</span><span class="sxs-lookup"><span data-stu-id="95469-383">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="95469-384">Entity Framework は、構成ファイルで定義されている再試行構成をサポートします。</span><span class="sxs-lookup"><span data-stu-id="95469-384">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="95469-385">ただし、Azure 上で最大の柔軟性を実現するために、アプリケーション内でプログラムを使用して構成を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-385">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="95469-386">再試行ポリシーの特定のパラメーター (再試行数や再試行間隔など) は、サービス構成ファイルに格納して、実行時に適切なポリシーを作成するために使用することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-386">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="95469-387">これにより、アプリケーションを再起動する必要なく設定を変更できます。</span><span class="sxs-lookup"><span data-stu-id="95469-387">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="95469-388">再試行操作を次の設定から始めることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-388">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="95469-389">再試行間の遅延を指定することはできません (固定されており、指数のシーケンスとして生成されます)。</span><span class="sxs-lookup"><span data-stu-id="95469-389">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="95469-390">カスタム再試行戦略を作成しない限り、ここに示すとおり、指定できるのは最大値のみです。</span><span class="sxs-lookup"><span data-stu-id="95469-390">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="95469-391">これらは汎用の設定であり、操作を監視して、独自のシナリオに合うように値を微調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-391">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="95469-392">**コンテキスト**</span><span class="sxs-lookup"><span data-stu-id="95469-392">**Context**</span></span> | <span data-ttu-id="95469-393">**サンプルのターゲット E2E<br />最大待機時間**</span><span class="sxs-lookup"><span data-stu-id="95469-393">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="95469-394">**再試行ポリシー**</span><span class="sxs-lookup"><span data-stu-id="95469-394">**Retry policy**</span></span> | <span data-ttu-id="95469-395">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-395">**Settings**</span></span> | <span data-ttu-id="95469-396">**値**</span><span class="sxs-lookup"><span data-stu-id="95469-396">**Values**</span></span> | <span data-ttu-id="95469-397">**動作のしくみ**</span><span class="sxs-lookup"><span data-stu-id="95469-397">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="95469-398">対話型、UI、</span><span class="sxs-lookup"><span data-stu-id="95469-398">Interactive, UI,</span></span><br /><span data-ttu-id="95469-399">またはフォアグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-399">or foreground</span></span> |<span data-ttu-id="95469-400">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-400">2 seconds</span></span> |<span data-ttu-id="95469-401">指数</span><span class="sxs-lookup"><span data-stu-id="95469-401">Exponential</span></span> |<span data-ttu-id="95469-402">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="95469-402">MaxRetryCount</span></span><br /><span data-ttu-id="95469-403">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="95469-403">MaxDelay</span></span> |<span data-ttu-id="95469-404">3</span><span class="sxs-lookup"><span data-stu-id="95469-404">3</span></span><br /><span data-ttu-id="95469-405">750 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-405">750 ms</span></span> |<span data-ttu-id="95469-406">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-406">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-407">試行 2 - 750 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-407">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="95469-408">試行 3 - 750 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-408">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="95469-409">バックグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-409">Background</span></span><br /> <span data-ttu-id="95469-410">またはバッチ</span><span class="sxs-lookup"><span data-stu-id="95469-410">or batch</span></span> |<span data-ttu-id="95469-411">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-411">30 seconds</span></span> |<span data-ttu-id="95469-412">指数</span><span class="sxs-lookup"><span data-stu-id="95469-412">Exponential</span></span> |<span data-ttu-id="95469-413">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="95469-413">MaxRetryCount</span></span><br /><span data-ttu-id="95469-414">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="95469-414">MaxDelay</span></span> |<span data-ttu-id="95469-415">5</span><span class="sxs-lookup"><span data-stu-id="95469-415">5</span></span><br /><span data-ttu-id="95469-416">12 秒</span><span class="sxs-lookup"><span data-stu-id="95469-416">12 seconds</span></span> |<span data-ttu-id="95469-417">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-417">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-418">試行 2 - 最大 1 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-418">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="95469-419">試行 3 - 最大 3 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-419">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="95469-420">試行 4 - 最大 7 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-420">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="95469-421">試行 5 - 12 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-421">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="95469-422">エンドツーエンド待機時間のターゲットには、サービスへの接続用の既定のタイムアウトが想定されます。</span><span class="sxs-lookup"><span data-stu-id="95469-422">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="95469-423">接続タイムアウトにより長い時間を指定する場合、エンドツーエンド待機時間は、すべての再試行についてこの追加時間分だけ延長されます。</span><span class="sxs-lookup"><span data-stu-id="95469-423">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="95469-424">例</span><span class="sxs-lookup"><span data-stu-id="95469-424">Examples</span></span>
<span data-ttu-id="95469-425">次のコード例は、Entity Framework を使用する単純なデータ アクセス ソリューションを定義します。</span><span class="sxs-lookup"><span data-stu-id="95469-425">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="95469-426">これは、**DbConfiguration** を拡張する **BlogConfiguration** というクラスのインスタンスを定義することで、特定の再試行戦略を設定します。</span><span class="sxs-lookup"><span data-stu-id="95469-426">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="95469-427">Entity Framework の再試行メカニズムを使用する他の例については、「 [Connection Resiliency / Retry Logic (接続の回復/再試行ロジック)](http://msdn.microsoft.com/data/dn456835.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-427">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="95469-428">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-428">More information</span></span>
* [<span data-ttu-id="95469-429">Microsoft Azure SQL Database Performance and Elasticity Guide (Microsoft Azure SQL Database のパフォーマンスと弾力性に関するガイド)</span><span class="sxs-lookup"><span data-stu-id="95469-429">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="95469-430">Entity Framework Core を使用する SQL Database の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-430">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="95469-431">[Entity Framework Core](/ef/core/) は、.NET Core 開発者がドメイン固有オブジェクトを使用してリレーショナル データを操作できるようにする、オブジェクト リレーショナル マッパーです。</span><span class="sxs-lookup"><span data-stu-id="95469-431">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="95469-432">これにより、開発者が通常は記述する必要のあるデータアクセス コードの大部分が不要になります。</span><span class="sxs-lookup"><span data-stu-id="95469-432">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="95469-433">このバージョンの Entity Framework は一から作成されているため、EF6.x の機能の一部は自動的に継承されません。</span><span class="sxs-lookup"><span data-stu-id="95469-433">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-434">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-434">Retry mechanism</span></span>
<span data-ttu-id="95469-435">再試行サポートは、SQL Database データベースに[接続の弾力性](/ef/core/miscellaneous/connection-resiliency)というメカニズム経由で Entity Framework Core を使用してアクセスするときに提供されます。</span><span class="sxs-lookup"><span data-stu-id="95469-435">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="95469-436">接続の弾力性は、EF Core 1.1.0 で導入されました。</span><span class="sxs-lookup"><span data-stu-id="95469-436">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="95469-437">プライマリの抽象型は、`IExecutionStrategy` インターフェイスです。</span><span class="sxs-lookup"><span data-stu-id="95469-437">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="95469-438">SQL Azure を含む SQL Server の実行戦略では、再試行できる例外タイプが認識されており、最大再試行回数、再試行間の遅延などについて実用的な既定値が設定されています。</span><span class="sxs-lookup"><span data-stu-id="95469-438">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="95469-439">例</span><span class="sxs-lookup"><span data-stu-id="95469-439">Examples</span></span>

<span data-ttu-id="95469-440">次のコードは、データベースとのセッションを表す DbContext オブジェクトを構成するときの自動再試行を実行します。</span><span class="sxs-lookup"><span data-stu-id="95469-440">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="95469-441">次のコードは、実行戦略に従った、自動再試行を使用したトランザクションの実行方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-441">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="95469-442">トランザクションは、デリゲート内で定義されています。</span><span class="sxs-lookup"><span data-stu-id="95469-442">The transaction is defined in a delegate.</span></span> <span data-ttu-id="95469-443">一時的な障害が発生した場合、実行戦略は、デリゲートをもう一度呼び出します。</span><span class="sxs-lookup"><span data-stu-id="95469-443">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a><span data-ttu-id="95469-444">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-444">More information</span></span>
* [<span data-ttu-id="95469-445">接続の回復性</span><span class="sxs-lookup"><span data-stu-id="95469-445">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="95469-446">データ ポイント - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="95469-446">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="95469-447">ADO.NET の再試行ガイドラインを使用する SQL Database</span><span class="sxs-lookup"><span data-stu-id="95469-447">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="95469-448">SQL Database は、多様なサイズで利用できるホステッド SQL データベースです。これは Standard (共有) サービスと Premium (非共有) サービスのどちらとしてでも使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-448">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-449">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-449">Retry mechanism</span></span>
<span data-ttu-id="95469-450">ADO.NET を使用してアクセスする際には、SQL Database には再試行の組み込みサポートはありません。</span><span class="sxs-lookup"><span data-stu-id="95469-450">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="95469-451">ただし、要求からのリターン コードを使用して、要求が失敗した理由を判別することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-451">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="95469-452">SQL Database の調整の詳細については、「[Azure SQL データベースのリソース制限](/azure/sql-database/sql-database-resource-limits)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-452">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="95469-453">関連するエラー コードの一覧については、「[SQL Database クライアント アプリケーションの SQL エラー コード](/azure/sql-database/sql-database-develop-error-messages)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-453">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="95469-454">SQL Database の再試行は、Polly ライブラリを使用して実装できます。</span><span class="sxs-lookup"><span data-stu-id="95469-454">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="95469-455">[Polly での一時的な障害処理](#transient-fault-handling-with-polly)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-455">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-456">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-456">Retry usage guidance</span></span>
<span data-ttu-id="95469-457">ADO.NET を使用して SQL Database にアクセスする場合は、次のガイドラインを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-457">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="95469-458">適切なサービス オプション (Shared または Premium) を選択します。</span><span class="sxs-lookup"><span data-stu-id="95469-458">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="95469-459">共有インスタンスは、共有サーバーの他のテナントによる使用状況により、通常よりも長い接続遅延や調整の影響を受ける可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-459">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="95469-460">予測可能性の高いパフォーマンスと信頼性の高い低待機時間での操作が必要な場合は、Premium オプションを選択することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-460">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="95469-461">データの不整合の原因となる非べき等操作を避けるために、再試行は必ず適切なレベルまたはスコープで実行します。</span><span class="sxs-lookup"><span data-stu-id="95469-461">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="95469-462">理想的には、すべての操作はべき等にして、不整合を発生させずに繰り返し実行できるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-462">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="95469-463">これが当てはまらない場合、再試行は、操作が失敗した場合に、関連するすべての変更を元に戻すことができるレベルまたはスコープで (たとえば 1 トランザクションのスコープ内で) 実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-463">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="95469-464">詳細については、「 [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling (クラウド サービスの基本データ アクセス層 – 一時的エラー処理)](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-464">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="95469-465">固定間隔戦略は、非常に短い間隔でごくわずかな回数の再試行が実行されるのみという対話型のシナリオを除き、Azure SQL Database での使用は推奨されていません。</span><span class="sxs-lookup"><span data-stu-id="95469-465">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="95469-466">代わりに、ほとんどのシナリオで、指数バックオフ戦略を使用することを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="95469-466">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="95469-467">接続を定義するときは、接続タイムアウトとコマンド タイムアウトに適切な値を選択します。</span><span class="sxs-lookup"><span data-stu-id="95469-467">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="95469-468">タイムアウトが短すぎると、データベースがビジー状態の場合に、接続が途中でエラーになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-468">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="95469-469">タイムアウトが長すぎると、接続エラーを検出するまで長く待ちすぎて、再試行ロジックが正常に機能しなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-469">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="95469-470">タイムアウトの値は、エンドツーエンド待機時間の構成要素です。これはすべての再試行向けの再試行ポリシーに指定される再試行遅延に、事実上追加されます。</span><span class="sxs-lookup"><span data-stu-id="95469-470">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="95469-471">指数バックオフ再試行ロジックを使用している場合でも、一定回数の再試行が実行された後は接続を閉じ、新しい接続で操作を再試行します。</span><span class="sxs-lookup"><span data-stu-id="95469-471">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="95469-472">同じ接続で同じ操作を複数回再試行することは、接続問題を生じさせる要因となる場合があります。</span><span class="sxs-lookup"><span data-stu-id="95469-472">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="95469-473">この技法の例については、「 [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling (クラウド サービスの基本データ アクセス層 – 一時的エラー処理)](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-473">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="95469-474">接続プールが使用中であれば (既定値)、接続を閉じてから再び開いた後であっても、同じ接続がプールから選択される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-474">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="95469-475">これが該当する場合、解決するための技法は、**SqlConnection** クラスの **ClearPool** メソッドを呼び出して、接続を再利用不可とマークすることです。</span><span class="sxs-lookup"><span data-stu-id="95469-475">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="95469-476">ただし、これは数回の接続試行が失敗し、問題がある接続に関連した SQL タイムアウト (エラー コード -2) などの、特定クラスの一時的エラーを検出した場合にのみ実行してください。</span><span class="sxs-lookup"><span data-stu-id="95469-476">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="95469-477">データ アクセス コードが **TransactionScope** インスタンスとして開始されたトランザクションを使用している場合、再試行ロジックは接続を再度開き、新しいトランザクション スコープを開始する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-477">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="95469-478">この理由から、再試行可能コード ブロックは、トランザクションのスコープ全体をカバーしている必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-478">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="95469-479">再試行操作について次の設定から始めることを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-479">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="95469-480">これらは汎用の設定であり、操作を監視して、独自のシナリオに合うように値を微調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-480">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="95469-481">**コンテキスト**</span><span class="sxs-lookup"><span data-stu-id="95469-481">**Context**</span></span> | <span data-ttu-id="95469-482">**サンプルのターゲット E2E<br />最大待機時間**</span><span class="sxs-lookup"><span data-stu-id="95469-482">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="95469-483">**再試行戦略**</span><span class="sxs-lookup"><span data-stu-id="95469-483">**Retry strategy**</span></span> | <span data-ttu-id="95469-484">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-484">**Settings**</span></span> | <span data-ttu-id="95469-485">**値**</span><span class="sxs-lookup"><span data-stu-id="95469-485">**Values**</span></span> | <span data-ttu-id="95469-486">**動作のしくみ**</span><span class="sxs-lookup"><span data-stu-id="95469-486">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="95469-487">対話型、UI、</span><span class="sxs-lookup"><span data-stu-id="95469-487">Interactive, UI,</span></span><br /><span data-ttu-id="95469-488">またはフォアグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-488">or foreground</span></span> |<span data-ttu-id="95469-489">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-489">2 sec</span></span> |<span data-ttu-id="95469-490">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="95469-490">FixedInterval</span></span> |<span data-ttu-id="95469-491">再試行回数</span><span class="sxs-lookup"><span data-stu-id="95469-491">Retry count</span></span><br /><span data-ttu-id="95469-492">再試行間隔</span><span class="sxs-lookup"><span data-stu-id="95469-492">Retry interval</span></span><br /><span data-ttu-id="95469-493">最初の高速再試行</span><span class="sxs-lookup"><span data-stu-id="95469-493">First fast retry</span></span> |<span data-ttu-id="95469-494">3</span><span class="sxs-lookup"><span data-stu-id="95469-494">3</span></span><br /><span data-ttu-id="95469-495">500 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-495">500 ms</span></span><br /><span data-ttu-id="95469-496">true</span><span class="sxs-lookup"><span data-stu-id="95469-496">true</span></span> |<span data-ttu-id="95469-497">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-497">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-498">試行 2 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-498">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="95469-499">試行 3 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-499">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="95469-500">バックグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-500">Background</span></span><br /><span data-ttu-id="95469-501">またはバッチ</span><span class="sxs-lookup"><span data-stu-id="95469-501">or batch</span></span> |<span data-ttu-id="95469-502">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-502">30 sec</span></span> |<span data-ttu-id="95469-503">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-503">ExponentialBackoff</span></span> |<span data-ttu-id="95469-504">再試行回数</span><span class="sxs-lookup"><span data-stu-id="95469-504">Retry count</span></span><br /><span data-ttu-id="95469-505">最小バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-505">Min back-off</span></span><br /><span data-ttu-id="95469-506">最大バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-506">Max back-off</span></span><br /><span data-ttu-id="95469-507">差分バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-507">Delta back-off</span></span><br /><span data-ttu-id="95469-508">最初の高速再試行</span><span class="sxs-lookup"><span data-stu-id="95469-508">First fast retry</span></span> |<span data-ttu-id="95469-509">5</span><span class="sxs-lookup"><span data-stu-id="95469-509">5</span></span><br /><span data-ttu-id="95469-510">0 秒</span><span class="sxs-lookup"><span data-stu-id="95469-510">0 sec</span></span><br /><span data-ttu-id="95469-511">60 秒</span><span class="sxs-lookup"><span data-stu-id="95469-511">60 sec</span></span><br /><span data-ttu-id="95469-512">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-512">2 sec</span></span><br /><span data-ttu-id="95469-513">false</span><span class="sxs-lookup"><span data-stu-id="95469-513">false</span></span> |<span data-ttu-id="95469-514">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-514">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-515">試行 2 - 最大 2 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-515">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="95469-516">試行 3 - 最大 6 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-516">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="95469-517">試行 4 - 最大 14 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-517">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="95469-518">試行 5 - 最大 30 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-518">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="95469-519">エンドツーエンド待機時間のターゲットには、サービスへの接続用の既定のタイムアウトが想定されます。</span><span class="sxs-lookup"><span data-stu-id="95469-519">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="95469-520">接続タイムアウトにより長い時間を指定する場合、エンドツーエンド待機時間は、すべての再試行についてこの追加時間分だけ延長されます。</span><span class="sxs-lookup"><span data-stu-id="95469-520">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="95469-521">例</span><span class="sxs-lookup"><span data-stu-id="95469-521">Examples</span></span>
<span data-ttu-id="95469-522">このセクションでは、Polly を使用して、`Policy` クラスに構成されている再試行ポリシーを使用して Azure SQL Database にアクセスする方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="95469-522">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="95469-523">次のコードは、指数バックオフを使用して `ExecuteAsync` を呼び出す、`SqlCommand` クラスの拡張メソッドを示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-523">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="95469-524">この非同期の拡張メソッドは、次のように使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-524">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="95469-525">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-525">More information</span></span>
* [<span data-ttu-id="95469-526">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling (クラウド サービスの基本データアクセス層 – 一時的エラー処理)</span><span class="sxs-lookup"><span data-stu-id="95469-526">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="95469-527">SQL Database を最大限活用するための一般的なガイダンスについては、[Azure SQL Database パフォーマンス/弾力性に関するガイド](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-527">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="95469-528">Service Bus の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-528">Service Bus retry guidelines</span></span>
<span data-ttu-id="95469-529">Service Bus は、クラウド メッセージング プラットフォームであり、クラウドまたはオンプレミスでホストされているアプリケーションのコンポーネントに対して、向上したスケーラビリティと回復性を備えた疎結合のメッセージ交換を提供します。</span><span class="sxs-lookup"><span data-stu-id="95469-529">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-530">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-530">Retry mechanism</span></span>
<span data-ttu-id="95469-531">Service Bus は、 [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) 基本クラスの実装を使用して、再試行を実装します。</span><span class="sxs-lookup"><span data-stu-id="95469-531">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="95469-532">すべての Service Bus クライアントは、**RetryPolicy** 基本クラスのいずれかの実装に設定することができる、**RetryPolicy** プロパティを公開します。</span><span class="sxs-lookup"><span data-stu-id="95469-532">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="95469-533">組み込み実装は、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="95469-533">The built-in implementations are:</span></span>

* <span data-ttu-id="95469-534">[RetryExponential クラス](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx)。</span><span class="sxs-lookup"><span data-stu-id="95469-534">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="95469-535">これは、バックオフ間隔と再試行数を制御するプロパティ、および操作が完了するまでの合計時間を制限するために使用される **TerminationTimeBuffer** プロパティを公開します。</span><span class="sxs-lookup"><span data-stu-id="95469-535">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="95469-536">[NoRetry クラス](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx)。</span><span class="sxs-lookup"><span data-stu-id="95469-536">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="95469-537">これは、再試行がバッチまたは複数ステップの操作の一部として別のプロセスにより管理されている場合などの、Service Bus API レベルでの再試行が不要な場合に使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-537">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="95469-538">Service Bus アクションは、さまざまな例外を返す可能性があります。それらの例外は、[メッセージングの例外に関する付録](http://msdn.microsoft.com/library/hh418082.aspx)にリストされています。</span><span class="sxs-lookup"><span data-stu-id="95469-538">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="95469-539">このリストには、操作の再試行が適切であると例外が示す場合の関連情報が記載されています。</span><span class="sxs-lookup"><span data-stu-id="95469-539">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="95469-540">たとえば、[ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) は、クライアントが一定時間待機して、それから操作を再試行する必要があることを示します。</span><span class="sxs-lookup"><span data-stu-id="95469-540">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="95469-541">また、**ServerBusyException** が発生すると、Service Bus は別のモードに切り替わります。この場合には計算された再試行遅延にさらに 10 秒の遅延が追加されます。</span><span class="sxs-lookup"><span data-stu-id="95469-541">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="95469-542">このモードは、短時間が経過した後にリセットされます。</span><span class="sxs-lookup"><span data-stu-id="95469-542">This mode is reset after a short period.</span></span>

<span data-ttu-id="95469-543">Service Bus から返される例外は、クライアントが操作を再試行するかどうかを示す **IsTransient** プロパティを公開します。</span><span class="sxs-lookup"><span data-stu-id="95469-543">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="95469-544">組み込みの **RetryExponential** ポリシーは、すべての Service Bus 例外の基本クラスである **MessagingException** クラス内の、**IsTransient** プロパティに依存しています。</span><span class="sxs-lookup"><span data-stu-id="95469-544">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="95469-545">**RetryPolicy** 基本クラスのカスタム実装を作成する場合、例外タイプと **IsTransient** プロパティの組み合わせを使用して、再試行アクションに対するより細かい制御を提供することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-545">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="95469-546">たとえば、 **QuotaExceededException** を検出した場合に、キューへのメッセージの送信を再試行する前に、キューを排出するアクションを実行することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-546">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="95469-547">ポリシーの構成</span><span class="sxs-lookup"><span data-stu-id="95469-547">Policy configuration</span></span>
<span data-ttu-id="95469-548">再試行ポリシーは、プログラムによって設定されます。このポリシーは、**NamespaceManager** および **MessagingFactory** の既定のポリシーとして設定することも、各メッセージング クライアントに対して個別に設定することもできます。</span><span class="sxs-lookup"><span data-stu-id="95469-548">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="95469-549">メッセージング セッションの既定の再試行ポリシーとして設定するには、**NamespaceManager** の **RetryPolicy** を設定します。</span><span class="sxs-lookup"><span data-stu-id="95469-549">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="95469-550">メッセージング ファクトリから作成されたすべてのクライアントに対して既定の再試行ポリシーを設定するには、**MessagingFactory** の **RetryPolicy** を設定します。</span><span class="sxs-lookup"><span data-stu-id="95469-550">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="95469-551">メッセージング クライアントに対して再試行ポリシーを設定するか、または既定のポリシーをオーバーライドするには、その **RetryPolicy** プロパティを、必要なポリシー クラスのインスタンスを使用して設定します。</span><span class="sxs-lookup"><span data-stu-id="95469-551">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="95469-552">再試行ポリシーは、個々の操作レベルで設定することはできません。</span><span class="sxs-lookup"><span data-stu-id="95469-552">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="95469-553">これはメッセージング クライアントのすべての操作に適用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-553">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="95469-554">次の表は、組み込み再試行ポリシーの既定の設定を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-554">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="95469-555">Setting</span><span class="sxs-lookup"><span data-stu-id="95469-555">Setting</span></span> | <span data-ttu-id="95469-556">既定値</span><span class="sxs-lookup"><span data-stu-id="95469-556">Default value</span></span> | <span data-ttu-id="95469-557">意味</span><span class="sxs-lookup"><span data-stu-id="95469-557">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="95469-558">ポリシー</span><span class="sxs-lookup"><span data-stu-id="95469-558">Policy</span></span> | <span data-ttu-id="95469-559">指数</span><span class="sxs-lookup"><span data-stu-id="95469-559">Exponential</span></span> | <span data-ttu-id="95469-560">指数バックオフ。</span><span class="sxs-lookup"><span data-stu-id="95469-560">Exponential back-off.</span></span> |
| <span data-ttu-id="95469-561">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-561">MinimalBackoff</span></span> | <span data-ttu-id="95469-562">0</span><span class="sxs-lookup"><span data-stu-id="95469-562">0</span></span> | <span data-ttu-id="95469-563">最小バックオフ間隔。</span><span class="sxs-lookup"><span data-stu-id="95469-563">Minimum back-off interval.</span></span> <span data-ttu-id="95469-564">これはdeltaBackoff から計算された再試行間隔に追加されます。</span><span class="sxs-lookup"><span data-stu-id="95469-564">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="95469-565">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-565">MaximumBackoff</span></span> | <span data-ttu-id="95469-566">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-566">30 seconds</span></span> | <span data-ttu-id="95469-567">最大バックオフ間隔。</span><span class="sxs-lookup"><span data-stu-id="95469-567">Maximum back-off interval.</span></span> <span data-ttu-id="95469-568">MaximumBackoff は、計算された再試行間隔が MaxBackoff より大きい場合に使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-568">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="95469-569">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-569">DeltaBackoff</span></span> | <span data-ttu-id="95469-570">3 秒</span><span class="sxs-lookup"><span data-stu-id="95469-570">3 seconds</span></span> | <span data-ttu-id="95469-571">再試行のバックオフ間隔。</span><span class="sxs-lookup"><span data-stu-id="95469-571">Back-off interval between retries.</span></span> <span data-ttu-id="95469-572">この期間の倍数は、後続の再試行に使用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-572">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="95469-573">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="95469-573">TimeBuffer</span></span> | <span data-ttu-id="95469-574">5 秒</span><span class="sxs-lookup"><span data-stu-id="95469-574">5 seconds</span></span> | <span data-ttu-id="95469-575">再試行に関連付けられる終了時間バッファー。</span><span class="sxs-lookup"><span data-stu-id="95469-575">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="95469-576">残り時間が TimeBuffer 未満になると再試行は中止されます。</span><span class="sxs-lookup"><span data-stu-id="95469-576">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="95469-577">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="95469-577">MaxRetryCount</span></span> | <span data-ttu-id="95469-578">10</span><span class="sxs-lookup"><span data-stu-id="95469-578">10</span></span> | <span data-ttu-id="95469-579">最大再試行回数。</span><span class="sxs-lookup"><span data-stu-id="95469-579">The maximum number of retries.</span></span> |
| <span data-ttu-id="95469-580">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="95469-580">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="95469-581">10 秒</span><span class="sxs-lookup"><span data-stu-id="95469-581">10 seconds</span></span> | <span data-ttu-id="95469-582">検出された最後の例外が **ServerBusyException** の場合、この値は、計算された再試行間隔に追加されます。</span><span class="sxs-lookup"><span data-stu-id="95469-582">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="95469-583">この値は変更できません。</span><span class="sxs-lookup"><span data-stu-id="95469-583">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-584">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-584">Retry usage guidance</span></span>
<span data-ttu-id="95469-585">Service Bus を使用する場合は、次のガイドラインについて検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-585">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="95469-586">組み込みの **RetryExponential** 実装を使用する際には、ポリシーがサーバー ビジー例外に反応し、適切な再試行モードに自動的に切り替えるので、フォールバック操作を実装しないでください。</span><span class="sxs-lookup"><span data-stu-id="95469-586">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="95469-587">Service Bus は、組み合わせ名前空間という機能をサポートします。これは、プライマリ名前空間内のキューでエラーが発生した場合の、別の名前空間内にあるバックアップ キューへの自動フェールオーバーを実装します。</span><span class="sxs-lookup"><span data-stu-id="95469-587">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="95469-588">セカンダリ キューからのメッセージは、プライマリ キューが回復したらそこに送り戻すことができます。</span><span class="sxs-lookup"><span data-stu-id="95469-588">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="95469-589">この機能は、一時的なエラーに対処するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="95469-589">This feature helps to address transient failures.</span></span> <span data-ttu-id="95469-590">詳細については、「 [非同期メッセージング パターンと高可用性](http://msdn.microsoft.com/library/azure/dn292562.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-590">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="95469-591">再試行操作について次の設定から始めることを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-591">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="95469-592">これらは汎用の設定であり、操作を監視して、独自のシナリオに合うように値を微調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-592">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="95469-593">Context</span><span class="sxs-lookup"><span data-stu-id="95469-593">Context</span></span> | <span data-ttu-id="95469-594">最大待機時間の例</span><span class="sxs-lookup"><span data-stu-id="95469-594">Example maximum latency</span></span> | <span data-ttu-id="95469-595">再試行ポリシー</span><span class="sxs-lookup"><span data-stu-id="95469-595">Retry policy</span></span> | <span data-ttu-id="95469-596">設定</span><span class="sxs-lookup"><span data-stu-id="95469-596">Settings</span></span> | <span data-ttu-id="95469-597">動作のしくみ</span><span class="sxs-lookup"><span data-stu-id="95469-597">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="95469-598">対話型、UI、またはフォアグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-598">Interactive, UI, or foreground</span></span> | <span data-ttu-id="95469-599">2 秒\*</span><span class="sxs-lookup"><span data-stu-id="95469-599">2 seconds\*</span></span>  | <span data-ttu-id="95469-600">指数</span><span class="sxs-lookup"><span data-stu-id="95469-600">Exponential</span></span> | <span data-ttu-id="95469-601">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="95469-601">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="95469-602">MaximumBackoff = 30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-602">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="95469-603">DeltaBackoff = 300 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-603">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="95469-604">TimeBuffer = 300 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-604">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="95469-605">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="95469-605">MaxRetryCount = 2</span></span> | <span data-ttu-id="95469-606">試行 1: 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-606">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="95469-607">試行 2: 最大 300 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-607">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="95469-608">試行 3: 最大 900 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-608">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="95469-609">バックグラウンドまたはバッチ</span><span class="sxs-lookup"><span data-stu-id="95469-609">Background or batch</span></span> | <span data-ttu-id="95469-610">30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-610">30 seconds</span></span> | <span data-ttu-id="95469-611">指数</span><span class="sxs-lookup"><span data-stu-id="95469-611">Exponential</span></span> | <span data-ttu-id="95469-612">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="95469-612">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="95469-613">MaximumBackoff = 30 秒</span><span class="sxs-lookup"><span data-stu-id="95469-613">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="95469-614">DeltaBackoff = 1.75 秒</span><span class="sxs-lookup"><span data-stu-id="95469-614">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="95469-615">TimeBuffer = 5 秒</span><span class="sxs-lookup"><span data-stu-id="95469-615">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="95469-616">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="95469-616">MaxRetryCount = 3</span></span> | <span data-ttu-id="95469-617">試行 1: 最大 1 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-617">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="95469-618">試行 2: 最大 3 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-618">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="95469-619">試行 3: 最大 6 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-619">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="95469-620">試行 4: 最大 13 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-620">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="95469-621">\* サーバー ビジー応答を受信した場合に追加される遅延は含まれません。</span><span class="sxs-lookup"><span data-stu-id="95469-621">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="95469-622">テレメトリ</span><span class="sxs-lookup"><span data-stu-id="95469-622">Telemetry</span></span>
<span data-ttu-id="95469-623">Service Bus は、再試行を ETW イベントとして **EventSource**を使ってログに記録します。</span><span class="sxs-lookup"><span data-stu-id="95469-623">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="95469-624">イベントをキャプチャしてパフォーマンス ビューアーに表示したり、イベントを適切な宛先ログに書き込んだりするには、 **EventListener** をイベント ソースにアタッチする必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-624">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="95469-625">これを行うには、 [セマンティック ログ記録アプリケーション ブロック](http://msdn.microsoft.com/library/dn775006.aspx) を使用することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-625">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="95469-626">再試行イベントは、次の形式です。</span><span class="sxs-lookup"><span data-stu-id="95469-626">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="95469-627">例</span><span class="sxs-lookup"><span data-stu-id="95469-627">Examples</span></span>
<span data-ttu-id="95469-628">次のコード例は、以下に対する再試行ポリシーの設定方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-628">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="95469-629">名前空間マネージャー。</span><span class="sxs-lookup"><span data-stu-id="95469-629">A namespace manager.</span></span> <span data-ttu-id="95469-630">ポリシーは、そのマネージャー上のすべての操作に適用されます。個々の操作向けにオーバーライドすることはできません。</span><span class="sxs-lookup"><span data-stu-id="95469-630">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="95469-631">メッセージング ファクトリ。</span><span class="sxs-lookup"><span data-stu-id="95469-631">A messaging factory.</span></span> <span data-ttu-id="95469-632">ポリシーは、このファクトリから作成されたすべてのクライアントに適用されます。個々のクライアントの作成時にオーバーライドすることはできません。</span><span class="sxs-lookup"><span data-stu-id="95469-632">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="95469-633">個々のメッセージング クライアント</span><span class="sxs-lookup"><span data-stu-id="95469-633">An individual messaging client.</span></span> <span data-ttu-id="95469-634">クライアントの作成後に、そのクライアントに再試行ポリシーを設定することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-634">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="95469-635">ポリシーは、そのクライアント上のすべての操作に適用されます。</span><span class="sxs-lookup"><span data-stu-id="95469-635">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="95469-636">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-636">More information</span></span>
* [<span data-ttu-id="95469-637">非同期メッセージング パターンと高可用性</span><span class="sxs-lookup"><span data-stu-id="95469-637">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="95469-638">Azure Redis Cache の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-638">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="95469-639">Azure Redis Cache は、一般的なオープン ソース Redis Cache に基づく、高速データ アクセスと低待機時間のキャッシュ サービスです。</span><span class="sxs-lookup"><span data-stu-id="95469-639">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="95469-640">これは安全で、Microsoft により管理されており、Azure の任意のアプリケーションからアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="95469-640">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="95469-641">このセクションのガイダンスは、StackExchange.Redis クライアントを使用したキャッシュへのアクセスに基づいています。</span><span class="sxs-lookup"><span data-stu-id="95469-641">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="95469-642">他の適切なクライアントのリストについては、[Redis の Web サイト](http://redis.io/clients)を参照してください。それらのクライアントは異なる再試行メカニズムを備えている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="95469-642">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="95469-643">StackExchange.Redis クライアントは、1 つの接続で多重化を使用することに注意してください。</span><span class="sxs-lookup"><span data-stu-id="95469-643">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="95469-644">推奨される使用法は、アプリケーションの起動時にこのクライアントのインスタンスを作成し、そのインスタンスをキャッシュに対するすべての操作に使用するというものです。</span><span class="sxs-lookup"><span data-stu-id="95469-644">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="95469-645">このため、キャッシュへの接続が行われるのは 1 回限りであるので、このセクションのすべてのガイダンスは、その初期接続の再試行ポリシーに関連するものとなっており、キャッシュにアクセスする各操作に対するものではありません。</span><span class="sxs-lookup"><span data-stu-id="95469-645">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-646">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-646">Retry mechanism</span></span>
<span data-ttu-id="95469-647">StackExchange.Redis クライアントは、以下を含む一連のオプションで構成される接続マネージャー クラスを使用します。</span><span class="sxs-lookup"><span data-stu-id="95469-647">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="95469-648">**ConnectRetry**。</span><span class="sxs-lookup"><span data-stu-id="95469-648">**ConnectRetry**.</span></span> <span data-ttu-id="95469-649">キャッシュに対し失敗した接続を再試行する回数。</span><span class="sxs-lookup"><span data-stu-id="95469-649">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="95469-650">**ReconnectRetryPolicy**。</span><span class="sxs-lookup"><span data-stu-id="95469-650">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="95469-651">使用する再試行戦略。</span><span class="sxs-lookup"><span data-stu-id="95469-651">The retry strategy to use.</span></span>
- <span data-ttu-id="95469-652">**ConnectTimeout**。</span><span class="sxs-lookup"><span data-stu-id="95469-652">**ConnectTimeout**.</span></span> <span data-ttu-id="95469-653">最大待機時間 (ミリ秒)。</span><span class="sxs-lookup"><span data-stu-id="95469-653">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="95469-654">ポリシーの構成</span><span class="sxs-lookup"><span data-stu-id="95469-654">Policy configuration</span></span>
<span data-ttu-id="95469-655">再試行ポリシーは、キャッシュに接続する前に、クライアントにオプションを設定することで、プログラムによって構成されます。</span><span class="sxs-lookup"><span data-stu-id="95469-655">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="95469-656">これは、**ConfigurationOptions** クラスのインスタンスを作成し、そのプロパティを設定し、それを **Connect** メソッドに渡すことによって実行できます。</span><span class="sxs-lookup"><span data-stu-id="95469-656">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="95469-657">組み込みクラスは、ランダム化された再試行間隔の線形 (一定) 遅延および指数バックオフをサポートします。</span><span class="sxs-lookup"><span data-stu-id="95469-657">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="95469-658">**IReconnectRetryPolicy** インターフェイスを実装することによって、カスタムの再試行ポリシーを作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="95469-658">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="95469-659">次の例では、指数バックオフを使用して、再試行戦略を構成します。</span><span class="sxs-lookup"><span data-stu-id="95469-659">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="95469-660">または、文字列としてオプションを指定して、それを **Connect** メソッドに渡すこともできます。</span><span class="sxs-lookup"><span data-stu-id="95469-660">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="95469-661">なお、**ReconnectRetryPolicy** プロパティはコードを使用してのみ設定でき、この方法では設定できません。</span><span class="sxs-lookup"><span data-stu-id="95469-661">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="95469-662">キャッシュに接続する際に、オプションを直接指定することもできます。</span><span class="sxs-lookup"><span data-stu-id="95469-662">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="95469-663">詳細については、StackExchange.Redis のマニュアルの [Stack Exchange Redis の構成](https://stackexchange.github.io/StackExchange.Redis/Configuration)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-663">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="95469-664">次の表は、組み込み再試行ポリシーの既定の設定を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-664">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="95469-665">**コンテキスト**</span><span class="sxs-lookup"><span data-stu-id="95469-665">**Context**</span></span> | <span data-ttu-id="95469-666">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-666">**Setting**</span></span> | <span data-ttu-id="95469-667">**既定値**</span><span class="sxs-lookup"><span data-stu-id="95469-667">**Default value**</span></span><br /><span data-ttu-id="95469-668">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="95469-668">(v 1.2.2)</span></span> | <span data-ttu-id="95469-669">**意味**</span><span class="sxs-lookup"><span data-stu-id="95469-669">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="95469-670">構成オプション</span><span class="sxs-lookup"><span data-stu-id="95469-670">ConfigurationOptions</span></span> |<span data-ttu-id="95469-671">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="95469-671">ConnectRetry</span></span><br /><br /><span data-ttu-id="95469-672">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="95469-672">ConnectTimeout</span></span><br /><br /><span data-ttu-id="95469-673">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="95469-673">SyncTimeout</span></span><br /><br /><span data-ttu-id="95469-674">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="95469-674">ReconnectRetryPolicy</span></span> |<span data-ttu-id="95469-675">3</span><span class="sxs-lookup"><span data-stu-id="95469-675">3</span></span><br /><br /><span data-ttu-id="95469-676">最大 5,000 ミリ秒に SyncTimeout を加算</span><span class="sxs-lookup"><span data-stu-id="95469-676">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="95469-677">1,000</span><span class="sxs-lookup"><span data-stu-id="95469-677">1000</span></span><br /><br /><span data-ttu-id="95469-678">LinearRetry 5000 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-678">LinearRetry 5000 ms</span></span> |<span data-ttu-id="95469-679">初期接続操作中に接続試行を繰り返す回数。</span><span class="sxs-lookup"><span data-stu-id="95469-679">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="95469-680">接続操作のタイムアウト (ミリ秒)。</span><span class="sxs-lookup"><span data-stu-id="95469-680">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="95469-681">再試行間の遅延ではありません。</span><span class="sxs-lookup"><span data-stu-id="95469-681">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="95469-682">同期操作が許容される時間 (ミリ秒)。</span><span class="sxs-lookup"><span data-stu-id="95469-682">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="95469-683">5000 ミリ秒ごとに再試行してください。</span><span class="sxs-lookup"><span data-stu-id="95469-683">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="95469-684">同期操作では、`SyncTimeout` によりエンド ツー エンドの待機時間を追加できますが、設定値が低すぎると、過剰にタイムアウトが発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="95469-684">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="95469-685">「[Azure Redis Cache のトラブルシューティング方法][redis-cache-troubleshoot]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-685">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="95469-686">一般に、同期操作ではなく、非同期操作を使用してください。</span><span class="sxs-lookup"><span data-stu-id="95469-686">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="95469-687">詳細については、「[Pipelines and Multiplexers (パイプラインとマルチプレクサー)](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-687">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-688">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-688">Retry usage guidance</span></span>
<span data-ttu-id="95469-689">Azure Redis Cache を使用する場合は、次のガイドラインについて検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-689">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="95469-690">StackExchange Redis クライアントは、その独自の再試行を管理します。ただし、アプリケーションの初回の起動時にキャッシュへの接続を確立するときのみです。</span><span class="sxs-lookup"><span data-stu-id="95469-690">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="95469-691">接続タイムアウト、再試行回数、接続の再試行間の間隔を設定できますが、キャッシュに対する操作には再試行ポリシーは適用されません。</span><span class="sxs-lookup"><span data-stu-id="95469-691">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="95469-692">多数の再試行を使用するのではなく、元のデータ ソースにアクセスすることによってフォールバックすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-692">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="95469-693">テレメトリ</span><span class="sxs-lookup"><span data-stu-id="95469-693">Telemetry</span></span>
<span data-ttu-id="95469-694">接続 (他の操作ではない) に関する情報は、 **TextWriter**を使用して収集できます。</span><span class="sxs-lookup"><span data-stu-id="95469-694">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="95469-695">これが生成する出力の例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="95469-695">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="95469-696">例</span><span class="sxs-lookup"><span data-stu-id="95469-696">Examples</span></span>
<span data-ttu-id="95469-697">次のコード例では、StackExchange.Redis クライアントを初期化する際に、再試行と再試行の間の一定 (線形) の待ち時間を構成します。</span><span class="sxs-lookup"><span data-stu-id="95469-697">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="95469-698">この例は、**ConfigurationOptions** のインスタンスを使用して構成を設定する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-698">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="95469-699">次の例では、オプションを文字列として指定することで、構成を設定しています。</span><span class="sxs-lookup"><span data-stu-id="95469-699">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="95469-700">接続タイムアウトとは、キャッシュへの接続での最大待ち期間であり、再試行間の待ち時間ではありません。</span><span class="sxs-lookup"><span data-stu-id="95469-700">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="95469-701">なお、**ReconnectRetryPolicy** プロパティは、コードを使用してのみ設定できます。</span><span class="sxs-lookup"><span data-stu-id="95469-701">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="95469-702">詳細な例については、プロジェクト Web サイトの「[Configuration (構成)](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-702">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="95469-703">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-703">More information</span></span>
* [<span data-ttu-id="95469-704">Redis の Web サイト</span><span class="sxs-lookup"><span data-stu-id="95469-704">Redis website</span></span>](http://redis.io/)

## <a name="cosmos-db-retry-guidelines"></a><span data-ttu-id="95469-705">Cosmos DB の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-705">Cosmos DB retry guidelines</span></span>

<span data-ttu-id="95469-706">Cosmos DB は、スキーマのない JSON データをサポートする完全に管理されたマルチモデル データベースです。</span><span class="sxs-lookup"><span data-stu-id="95469-706">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="95469-707">これは構成可能で信頼性の高いパフォーマンスと、ネイティブの JavaScript トランザクション処理を提供し、クラウド用にエラスティックなスケーラビリティを備えて作成されています。</span><span class="sxs-lookup"><span data-stu-id="95469-707">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-708">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-708">Retry mechanism</span></span>
<span data-ttu-id="95469-709">`DocumentClient` クラスによって、失敗した試行が自動的にもう一度実行されます。</span><span class="sxs-lookup"><span data-stu-id="95469-709">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="95469-710">再試行回数と最大待機時間を設定するには、[ConnectionPolicy.RetryOptions] を構成します。</span><span class="sxs-lookup"><span data-stu-id="95469-710">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="95469-711">クライアントがスローする例外は、再試行ポリシーを超えているか、一時的なエラーでないかのいずれかです。</span><span class="sxs-lookup"><span data-stu-id="95469-711">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="95469-712">クライアントが Cosmos DB によってスロットルされると、HTTP 429 エラーが返されます。</span><span class="sxs-lookup"><span data-stu-id="95469-712">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="95469-713">`DocumentClientException` で状態コードを確認します。</span><span class="sxs-lookup"><span data-stu-id="95469-713">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="95469-714">ポリシーの構成</span><span class="sxs-lookup"><span data-stu-id="95469-714">Policy configuration</span></span>
<span data-ttu-id="95469-715">次の表は、`RetryOptions` クラスの既定の設定を示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-715">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="95469-716">Setting</span><span class="sxs-lookup"><span data-stu-id="95469-716">Setting</span></span> | <span data-ttu-id="95469-717">既定値</span><span class="sxs-lookup"><span data-stu-id="95469-717">Default value</span></span> | <span data-ttu-id="95469-718">[説明]</span><span class="sxs-lookup"><span data-stu-id="95469-718">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="95469-719">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="95469-719">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="95469-720">9</span><span class="sxs-lookup"><span data-stu-id="95469-720">9</span></span> |<span data-ttu-id="95469-721">Cosmos DB によってクライアントに対してレート制限が適用されたために要求が失敗した場合の最大再試行回数。</span><span class="sxs-lookup"><span data-stu-id="95469-721">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="95469-722">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="95469-722">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="95469-723">30</span><span class="sxs-lookup"><span data-stu-id="95469-723">30</span></span> |<span data-ttu-id="95469-724">再試行の最大待機時間 (秒)。</span><span class="sxs-lookup"><span data-stu-id="95469-724">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="95469-725">例</span><span class="sxs-lookup"><span data-stu-id="95469-725">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="95469-726">テレメトリ</span><span class="sxs-lookup"><span data-stu-id="95469-726">Telemetry</span></span>
<span data-ttu-id="95469-727">再試行回数は、.NET **TraceSource**により、構造化されていないトレース メッセージとしてログに記録されます。</span><span class="sxs-lookup"><span data-stu-id="95469-727">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="95469-728">イベントをキャプチャし、それらを適切な宛先ログに書き込むには、**TraceListener** を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-728">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="95469-729">たとえば、次のコードを App.config ファイルに追加すると、同じ場所のテキスト ファイルに、実行可能ファイルとしてトレースが生成されます。</span><span class="sxs-lookup"><span data-stu-id="95469-729">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="95469-730">Azure Search の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-730">Azure Search retry guidelines</span></span>
<span data-ttu-id="95469-731">Azure Search は、Web サイトまたはアプリケーションへの強力で高度な検索機能の追加、検索結果のすばやく簡単な調整、および微調整された豊富な順位付けモデルの構築を行うために使用できます。</span><span class="sxs-lookup"><span data-stu-id="95469-731">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-732">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-732">Retry mechanism</span></span>
<span data-ttu-id="95469-733">Azure Search SDK の再試行動作は、[SearchServiceClient] クラスと [SearchIndexClient] クラスの `SetRetryPolicy` メソッドで制御されます。</span><span class="sxs-lookup"><span data-stu-id="95469-733">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="95469-734">既定のポリシーでは、Azure Search から 5xx または 408 (要求タイムアウト) の応答が返された場合に、指数関数的バックオフによる再試行が行われます。</span><span class="sxs-lookup"><span data-stu-id="95469-734">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="95469-735">テレメトリ</span><span class="sxs-lookup"><span data-stu-id="95469-735">Telemetry</span></span>
<span data-ttu-id="95469-736">ETW を使用してトレースするか、カスタム トレース プロバイダーを登録してトレースします。</span><span class="sxs-lookup"><span data-stu-id="95469-736">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="95469-737">詳細については、[AutoRest のドキュメント][autorest]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="95469-737">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="95469-738">Azure Active Directory の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-738">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="95469-739">Azure Active Directory (Azure AD) は、コア ディレクトリ サービス、拡張 ID 制御、セキュリティ、およびアプリケーション アクセス管理を結合した、包括的な ID 管理とアクセス管理のクラウド ソリューションです。</span><span class="sxs-lookup"><span data-stu-id="95469-739">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="95469-740">Azure AD は、一元化されたポリシーとルールに基づいてアプリケーションへのアクセス制御を実現するための、ID 管理プラットフォームを開発者に提供します。</span><span class="sxs-lookup"><span data-stu-id="95469-740">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-741">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-741">Retry mechanism</span></span>
<span data-ttu-id="95469-742">Active Directory Authentication Library (ADAL) には、Azure Active Directory のための組み込み再試行メカニズムがあります。</span><span class="sxs-lookup"><span data-stu-id="95469-742">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="95469-743">予想外のロックアウトを避けるためには、サードパーティのライブラリとアプリケーション コードで失敗した接続の再試行を**実行せず**、ADAL で再試行を処理することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="95469-743">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="95469-744">再試行使用のガイダンス</span><span class="sxs-lookup"><span data-stu-id="95469-744">Retry usage guidance</span></span>
<span data-ttu-id="95469-745">Azure Active Directory を使用する場合は、次のガイドラインについて検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-745">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="95469-746">可能な場合は、ADAL ライブラリと組み込みの再試行サポートを使用してください。</span><span class="sxs-lookup"><span data-stu-id="95469-746">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="95469-747">Azure Active Directory 用の REST API を使用している場合は、結果コードが 429 (Too Many Requests) または 5xx の範囲内のエラーの場合に操作を再試行してください。</span><span class="sxs-lookup"><span data-stu-id="95469-747">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="95469-748">その他のエラーの場合は、再試行しないでください。</span><span class="sxs-lookup"><span data-stu-id="95469-748">Do not retry for any other errors.</span></span>
* <span data-ttu-id="95469-749">Azure Active Directory を使用するバッチのシナリオでは、指数バックオフ ポリシーの使用が推奨されています。</span><span class="sxs-lookup"><span data-stu-id="95469-749">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="95469-750">再試行操作を次の設定から始めることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="95469-750">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="95469-751">これらは汎用の設定であり、操作を監視して、独自のシナリオに合うように値を微調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="95469-751">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="95469-752">**コンテキスト**</span><span class="sxs-lookup"><span data-stu-id="95469-752">**Context**</span></span> | <span data-ttu-id="95469-753">**サンプルのターゲット E2E<br />最大待機時間**</span><span class="sxs-lookup"><span data-stu-id="95469-753">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="95469-754">**再試行戦略**</span><span class="sxs-lookup"><span data-stu-id="95469-754">**Retry strategy**</span></span> | <span data-ttu-id="95469-755">**設定**</span><span class="sxs-lookup"><span data-stu-id="95469-755">**Settings**</span></span> | <span data-ttu-id="95469-756">**値**</span><span class="sxs-lookup"><span data-stu-id="95469-756">**Values**</span></span> | <span data-ttu-id="95469-757">**動作のしくみ**</span><span class="sxs-lookup"><span data-stu-id="95469-757">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="95469-758">対話型、UI、</span><span class="sxs-lookup"><span data-stu-id="95469-758">Interactive, UI,</span></span><br /><span data-ttu-id="95469-759">またはフォアグラウンド</span><span class="sxs-lookup"><span data-stu-id="95469-759">or foreground</span></span> |<span data-ttu-id="95469-760">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-760">2 sec</span></span> |<span data-ttu-id="95469-761">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="95469-761">FixedInterval</span></span> |<span data-ttu-id="95469-762">再試行回数</span><span class="sxs-lookup"><span data-stu-id="95469-762">Retry count</span></span><br /><span data-ttu-id="95469-763">再試行間隔</span><span class="sxs-lookup"><span data-stu-id="95469-763">Retry interval</span></span><br /><span data-ttu-id="95469-764">最初の高速再試行</span><span class="sxs-lookup"><span data-stu-id="95469-764">First fast retry</span></span> |<span data-ttu-id="95469-765">3</span><span class="sxs-lookup"><span data-stu-id="95469-765">3</span></span><br /><span data-ttu-id="95469-766">500 ミリ秒</span><span class="sxs-lookup"><span data-stu-id="95469-766">500 ms</span></span><br /><span data-ttu-id="95469-767">true</span><span class="sxs-lookup"><span data-stu-id="95469-767">true</span></span> |<span data-ttu-id="95469-768">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-768">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-769">試行 2 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-769">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="95469-770">試行 3 - 500 ミリ秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-770">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="95469-771">バック グラウンドまたは</span><span class="sxs-lookup"><span data-stu-id="95469-771">Background or</span></span><br /><span data-ttu-id="95469-772">バッチ</span><span class="sxs-lookup"><span data-stu-id="95469-772">batch</span></span> |<span data-ttu-id="95469-773">60 秒</span><span class="sxs-lookup"><span data-stu-id="95469-773">60 sec</span></span> |<span data-ttu-id="95469-774">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="95469-774">ExponentialBackoff</span></span> |<span data-ttu-id="95469-775">再試行回数</span><span class="sxs-lookup"><span data-stu-id="95469-775">Retry count</span></span><br /><span data-ttu-id="95469-776">最小バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-776">Min back-off</span></span><br /><span data-ttu-id="95469-777">最大バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-777">Max back-off</span></span><br /><span data-ttu-id="95469-778">差分バックオフ</span><span class="sxs-lookup"><span data-stu-id="95469-778">Delta back-off</span></span><br /><span data-ttu-id="95469-779">最初の高速再試行</span><span class="sxs-lookup"><span data-stu-id="95469-779">First fast retry</span></span> |<span data-ttu-id="95469-780">5</span><span class="sxs-lookup"><span data-stu-id="95469-780">5</span></span><br /><span data-ttu-id="95469-781">0 秒</span><span class="sxs-lookup"><span data-stu-id="95469-781">0 sec</span></span><br /><span data-ttu-id="95469-782">60 秒</span><span class="sxs-lookup"><span data-stu-id="95469-782">60 sec</span></span><br /><span data-ttu-id="95469-783">2 秒</span><span class="sxs-lookup"><span data-stu-id="95469-783">2 sec</span></span><br /><span data-ttu-id="95469-784">false</span><span class="sxs-lookup"><span data-stu-id="95469-784">false</span></span> |<span data-ttu-id="95469-785">試行 1 - 0 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-785">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="95469-786">試行 2 - 最大 2 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-786">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="95469-787">試行 3 - 最大 6 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-787">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="95469-788">試行 4 - 最大 14 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-788">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="95469-789">試行 5 - 最大 30 秒の遅延</span><span class="sxs-lookup"><span data-stu-id="95469-789">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="95469-790">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-790">More information</span></span>
* <span data-ttu-id="95469-791">[Azure Active Directory 認証ライブラリ][adal]</span><span class="sxs-lookup"><span data-stu-id="95469-791">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="95469-792">Service Fabric の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-792">Service Fabric retry guidelines</span></span>

<span data-ttu-id="95469-793">Service Fabric クラスターで信頼性の高いサービスを配信することにより、この記事に記載されているほぼすべての潜在的な一時障害を防止できます。</span><span class="sxs-lookup"><span data-stu-id="95469-793">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="95469-794">ただ、それでも一部の一時障害は発生します。</span><span class="sxs-lookup"><span data-stu-id="95469-794">Some transient faults are still possible, however.</span></span> <span data-ttu-id="95469-795">たとえば、名前付けサービスでルーティングを変更中に要求を受信すると例外がスローされます。</span><span class="sxs-lookup"><span data-stu-id="95469-795">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="95469-796">同じ要求を 100 ミリ秒後に受信すると、要求は成功する可能性が高くなります。</span><span class="sxs-lookup"><span data-stu-id="95469-796">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="95469-797">内部的には、Service Fabric が、この種の一時的な障害を管理します。</span><span class="sxs-lookup"><span data-stu-id="95469-797">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="95469-798">サービスのセットアップ時に、クラス `OperationRetrySettings` を使用して一部の設定を構成することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-798">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="95469-799">次のコードは例を示します。</span><span class="sxs-lookup"><span data-stu-id="95469-799">The following code shows an example.</span></span> <span data-ttu-id="95469-800">ほとんどの場合、既定の設定で対応できるため、このコードは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="95469-800">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        OperationTimeout = TimeSpan.FromSeconds(30)
    };

    var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

    var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

    var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

    var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
        new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
        new ServicePartitionKey(0));
```

## <a name="more-information"></a><span data-ttu-id="95469-801">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-801">More information</span></span>

* [<span data-ttu-id="95469-802">リモート例外処理</span><span class="sxs-lookup"><span data-stu-id="95469-802">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="95469-803">Azure Event Hubs の再試行ガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-803">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="95469-804">Azure Event Hubs は、数百万のイベントを収集、変換、保存する、ハイパースケールのテレメトリ インジェスト サービスです。</span><span class="sxs-lookup"><span data-stu-id="95469-804">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="95469-805">再試行メカニズム</span><span class="sxs-lookup"><span data-stu-id="95469-805">Retry mechanism</span></span>
<span data-ttu-id="95469-806">Azure Event Hubs Client Library での再試行動作は、`EventHubClient` クラスの `RetryPolicy` プロパティによって制御されます。</span><span class="sxs-lookup"><span data-stu-id="95469-806">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="95469-807">既定のポリシーでは、Azure Hub が一時 `EventHubsException` または `OperationCanceledException` を返したときに、指数バックオフで再試行します。</span><span class="sxs-lookup"><span data-stu-id="95469-807">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="95469-808">例</span><span class="sxs-lookup"><span data-stu-id="95469-808">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="95469-809">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-809">More information</span></span>
[<span data-ttu-id="95469-810">Azure Event Hubs の .NET Standard クライアント ライブラリ</span><span class="sxs-lookup"><span data-stu-id="95469-810"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="95469-811">一般的な REST および再試行のガイドライン</span><span class="sxs-lookup"><span data-stu-id="95469-811">General REST and retry guidelines</span></span>
<span data-ttu-id="95469-812">Azure またはサード パーティ提供のサービスにアクセスする場合は、次の事柄を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="95469-812">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="95469-813">再試行の管理に体系的な方法を使用し (再利用可能コードとするなど)、すべてのクライアントとすべてのソリューションの間で一貫性のある方式を適用できるようにします。</span><span class="sxs-lookup"><span data-stu-id="95469-813">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="95469-814">対象となるサービスまたはクライアントに組み込み再試行メカニズムがない場合の再試行を管理するために、一時的エラー処理アプリケーション ブロックなどの再試行フレームワークを使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="95469-814">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="95469-815">これは、一貫性のある再試行動作を実装するのに役立ち、ターゲットのサービスに適切な既定の再試行戦略を提供することができます。</span><span class="sxs-lookup"><span data-stu-id="95469-815">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="95469-816">ただし、非標準動作を持つサービスと一時的エラーを示す例外に依存しないサービス用に、カスタム再試行コードを作成することが必要になる場合があります。また、再試行動作を管理するために **Retry-Response** 応答を使用する場合も、カスタム再試行コードの作成が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="95469-816">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="95469-817">一時的なエラー検出ロジックは、REST 呼び出しの実行に使用する、実際のクライアント API によって異なります。</span><span class="sxs-lookup"><span data-stu-id="95469-817">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="95469-818">比較的新しい **HttpClient** クラスなどの一部のクライアントは、不成功 HTTP 状態コードで完了した要求には例外をスローしません。</span><span class="sxs-lookup"><span data-stu-id="95469-818">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="95469-819">これによりパフォーマンスは向上しますが、一時的エラー処理アプリケーション ブロックは使用できなくなります。</span><span class="sxs-lookup"><span data-stu-id="95469-819">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="95469-820">この場合、REST API への呼び出しを、非成功 HTTP 状態コードに対して例外を生成するコードでラップすることができます。こうすると、ブロック (Topaz) で処理できるようになります。</span><span class="sxs-lookup"><span data-stu-id="95469-820">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="95469-821">別の方法として、再試行を実施する別のメカニズムを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="95469-821">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="95469-822">サービスから返される HTTP 状態コードは、エラーが一時的なものであるかどうかを知るのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="95469-822">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="95469-823">状態コードにアクセスするか、または同等の例外の種類を判別するには、クライアントまたは再試行フレームワークによって生成される例外を調べることが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="95469-823">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="95469-824">一般に、次の HTTP コードは、再試行が適切であることを示します。</span><span class="sxs-lookup"><span data-stu-id="95469-824">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="95469-825">408 要求タイムアウト</span><span class="sxs-lookup"><span data-stu-id="95469-825">408 Request Timeout</span></span>
  * <span data-ttu-id="95469-826">429 Too Many Requests</span><span class="sxs-lookup"><span data-stu-id="95469-826">429 Too Many Requests</span></span>
  * <span data-ttu-id="95469-827">500 内部サーバー エラー</span><span class="sxs-lookup"><span data-stu-id="95469-827">500 Internal Server Error</span></span>
  * <span data-ttu-id="95469-828">502 無効なゲートウェイ</span><span class="sxs-lookup"><span data-stu-id="95469-828">502 Bad Gateway</span></span>
  * <span data-ttu-id="95469-829">503 サービス利用不可</span><span class="sxs-lookup"><span data-stu-id="95469-829">503 Service Unavailable</span></span>
  * <span data-ttu-id="95469-830">504 ゲートウェイ タイムアウト</span><span class="sxs-lookup"><span data-stu-id="95469-830">504 Gateway Timeout</span></span>
* <span data-ttu-id="95469-831">再試行ロジックを例外に基づくものにしている場合、次のものは一般に、接続が確立できなかった一時的なエラーを示しています。</span><span class="sxs-lookup"><span data-stu-id="95469-831">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="95469-832">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="95469-832">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="95469-833">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="95469-833">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="95469-834">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="95469-834">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="95469-835">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="95469-835">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="95469-836">サービス使用不可状態は、サービスが **Retry-After** 応答ヘッダーまたは別のカスタム ヘッダーで、再試行前の適切な遅延を示している場合があります。</span><span class="sxs-lookup"><span data-stu-id="95469-836">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="95469-837">サービスは、カスタム ヘッダーとして、または応答の内容に埋め込んで、追加情報を送信することもあります。</span><span class="sxs-lookup"><span data-stu-id="95469-837">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="95469-838">一時的エラー処理アプリケーション ブロックは、標準またはカスタムの "retry-after" ヘッダーを使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="95469-838">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="95469-839">408 要求タイムアウトを除き、クライアント エラー (4xx の範囲内のエラー) を表す状態コードに対しては再試行しないでください。</span><span class="sxs-lookup"><span data-stu-id="95469-839">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="95469-840">再試行戦略および再試行メカニズムは、多様なネットワーク状態やさまざまなシステム負荷などの幅広い条件下で十分にテストします。</span><span class="sxs-lookup"><span data-stu-id="95469-840">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="95469-841">再試行戦略</span><span class="sxs-lookup"><span data-stu-id="95469-841">Retry strategies</span></span>
<span data-ttu-id="95469-842">次に示すのは、標準的な種類の再試行戦略間隔です。</span><span class="sxs-lookup"><span data-stu-id="95469-842">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="95469-843">**Exponential**: 指定回数の再試行を実行し、ランダムな指数バックオフ アプローチを使用して再試行間の間隔を決定する再試行ポリシーです。</span><span class="sxs-lookup"><span data-stu-id="95469-843">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="95469-844">例: </span><span class="sxs-lookup"><span data-stu-id="95469-844">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="95469-845">**Incremental**: 指定回数の再試行を実行し、再試行ごとに時間間隔を長くする再試行戦略です。</span><span class="sxs-lookup"><span data-stu-id="95469-845">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="95469-846">例: </span><span class="sxs-lookup"><span data-stu-id="95469-846">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="95469-847">**LinearRetry**: 指定回数の再試行を実行し、再試行間には指定の固定時間間隔を使用する再試行ポリシーです。</span><span class="sxs-lookup"><span data-stu-id="95469-847">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="95469-848">例: </span><span class="sxs-lookup"><span data-stu-id="95469-848">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="95469-849">詳細情報</span><span class="sxs-lookup"><span data-stu-id="95469-849">More information</span></span>
* [<span data-ttu-id="95469-850">Circuit Breaker Pattern (サーキット ブレーカーのパターン)</span><span class="sxs-lookup"><span data-stu-id="95469-850">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="95469-851">Polly での一時的な障害処理</span><span class="sxs-lookup"><span data-stu-id="95469-851">Transient fault handling with Polly</span></span>
<span data-ttu-id="95469-852">[Polly](http://www.thepollyproject.org) は、再試行と[サーキット ブレーカー][circuit-breaker]戦略をプログラムで処理するライブラリです。</span><span class="sxs-lookup"><span data-stu-id="95469-852">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="95469-853">Polly プロジェクトは、[.NET Foundation][dotnet-foundation] のメンバーです。</span><span class="sxs-lookup"><span data-stu-id="95469-853">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="95469-854">クライアントがネイティブで再試行をサポートしないサービスでは、Polly は有効な代替手段で、正しく実装することが難しい可能性もある、カスタム再試行コードを書く必要がなくなります。</span><span class="sxs-lookup"><span data-stu-id="95469-854">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="95469-855">Polly でも、再試行をログに記録できるよう、エラーを追跡する方法が提供されています。</span><span class="sxs-lookup"><span data-stu-id="95469-855">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
