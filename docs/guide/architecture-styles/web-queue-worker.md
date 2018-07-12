---
title: Web キュー ワーカーのアーキテクチャ スタイル
description: Azure の Web ワーカー キュー ワーカーのアーキテクチャのメリット、課題、ベスト プラクティスについて説明します。
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/10/2018
ms.locfileid: "37958791"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="abdbf-103">Web キュー ワーカーのアーキテクチャ スタイル</span><span class="sxs-lookup"><span data-stu-id="abdbf-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="abdbf-104">このアーキテクチャのコア コンポーネントは、クライアント要求を処理する **Web フロント エンド**と、リソースを消費するタスク、実行時間の長いワークフロー、またはバッチ ジョブを実行する**ワーカー**です。</span><span class="sxs-lookup"><span data-stu-id="abdbf-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="abdbf-105">Web フロント エンドは**メッセージ キュー**を通じてワーカーと通信します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="abdbf-106">このアーキテクチャで一般的に使用されているその他のコンポーネントは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="abdbf-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="abdbf-107">1 つまたは複数のデータベース。</span><span class="sxs-lookup"><span data-stu-id="abdbf-107">One or more databases.</span></span> 
- <span data-ttu-id="abdbf-108">データベースから高速でデータを読み取るために値を格納するキャッシュ。</span><span class="sxs-lookup"><span data-stu-id="abdbf-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="abdbf-109">静的コンテンツを提供するための CDN</span><span class="sxs-lookup"><span data-stu-id="abdbf-109">CDN to serve static content</span></span>
- <span data-ttu-id="abdbf-110">電子メールや SMS サービスなどのリモート サービス。</span><span class="sxs-lookup"><span data-stu-id="abdbf-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="abdbf-111">多くの場合、これらは、サード パーティによって提供されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="abdbf-112">認証 ID プロバイダー。</span><span class="sxs-lookup"><span data-stu-id="abdbf-112">Identity provider for authentication.</span></span>

<span data-ttu-id="abdbf-113">Web とワーカーはいずれもステートレスです。</span><span class="sxs-lookup"><span data-stu-id="abdbf-113">The web and worker are both stateless.</span></span> <span data-ttu-id="abdbf-114">セッション状態は、分散キャッシュに格納することができます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="abdbf-115">ワーカーによる実行時間の長い作業は、非同期的に実行されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="abdbf-116">ワーカーは、キューのメッセージによってトリガーされるか、またはバッチ処理のためにスケジュールに従って実行できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="abdbf-117">ワーカーは、オプションのコンポーネントです。</span><span class="sxs-lookup"><span data-stu-id="abdbf-117">The worker is an optional component.</span></span> <span data-ttu-id="abdbf-118">処理時間が長い操作がない場合は、ワーカーを省略できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="abdbf-119">フロント エンドは、Web API で構成されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="abdbf-119">The front end might consist of a web API.</span></span> <span data-ttu-id="abdbf-120">クライアント側では、Web API は、AJAX 呼び出しを実行する単一ページ アプリケーションまたはネイティブ クライアント アプリケーションが使用できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="abdbf-121">このアーキテクチャを使用する状況</span><span class="sxs-lookup"><span data-stu-id="abdbf-121">When to use this architecture</span></span>

<span data-ttu-id="abdbf-122">Web キュー ワーカーのアーキテクチャは、通常、マネージ コンピューティング サービス、Azure App Service または Azure Cloud Services のいずれかを使用して実装されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="abdbf-123">次の場合に、このアーキテクチャ スタイルを検討してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="abdbf-124">比較的単純なドメインのアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="abdbf-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="abdbf-125">実行時間の長いワークフローやバッチ操作があるアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="abdbf-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="abdbf-126">サービスとしてのインフラストラクチャ (IaaS) ではなく、マネージ サービスを使用する場合。</span><span class="sxs-lookup"><span data-stu-id="abdbf-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="abdbf-127">メリット</span><span class="sxs-lookup"><span data-stu-id="abdbf-127">Benefits</span></span>

- <span data-ttu-id="abdbf-128">比較的単純でわかりやすいアーキテクチャ。</span><span class="sxs-lookup"><span data-stu-id="abdbf-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="abdbf-129">展開と管理が容易。</span><span class="sxs-lookup"><span data-stu-id="abdbf-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="abdbf-130">懸念事項の明確な分離。</span><span class="sxs-lookup"><span data-stu-id="abdbf-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="abdbf-131">フロント エンドは、非同期メッセージングを使用してワーカーから切り離されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="abdbf-132">フロント エンド ロールとワーカーはそれぞれ個別に拡張できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="abdbf-133">課題</span><span class="sxs-lookup"><span data-stu-id="abdbf-133">Challenges</span></span>

- <span data-ttu-id="abdbf-134">慎重に設計しないと、フロント エンドおよびワーカーが、大型のモノリシック コンポーネントになり、管理や更新が困難になる。</span><span class="sxs-lookup"><span data-stu-id="abdbf-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="abdbf-135">フロント エンド ロールとワーカーがデータ スキーマまたはコード モジュールを共有している場合、依存関係が潜在することがある。</span><span class="sxs-lookup"><span data-stu-id="abdbf-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="abdbf-136">ベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="abdbf-136">Best practices</span></span>

- <span data-ttu-id="abdbf-137">クライアントに適切に設計された API を公開する。</span><span class="sxs-lookup"><span data-stu-id="abdbf-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="abdbf-138">「[API 設計のベスト プラクティス][api-design]」を参照してください</span><span class="sxs-lookup"><span data-stu-id="abdbf-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="abdbf-139">負荷の変化に対応するために自動スケールする。</span><span class="sxs-lookup"><span data-stu-id="abdbf-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="abdbf-140">「[自動スケールのベスト プラクティス][autoscaling]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="abdbf-141">半静的なデータをキャッシュする。</span><span class="sxs-lookup"><span data-stu-id="abdbf-141">Cache semi-static data.</span></span> <span data-ttu-id="abdbf-142">「[キャッシングのベスト プラクティス][caching]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="abdbf-143">CDN を使用して、静的コンテンツをホストする。</span><span class="sxs-lookup"><span data-stu-id="abdbf-143">Use a CDN to host static content.</span></span> <span data-ttu-id="abdbf-144">「[CDN のベスト プラクティス][cdn]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="abdbf-145">適切な場合は、多言語永続化を使用する。</span><span class="sxs-lookup"><span data-stu-id="abdbf-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="abdbf-146">「[ジョブに最適なデータ ストアの使用][polyglot]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="abdbf-147">拡張性を向上し、競合を予防し、パフォーマンスを最適化するためにデータを分割する。</span><span class="sxs-lookup"><span data-stu-id="abdbf-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="abdbf-148">「[データのパーティション分割のベスト プラクティス][data-partition]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="abdbf-149">Azure App Service の Web キュー ワーカー</span><span class="sxs-lookup"><span data-stu-id="abdbf-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="abdbf-150">このセクションでは、Azure App Service を使用する、推奨 Web キュー ワーカー アーキテクチャについて説明します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="abdbf-151">フロント エンドは、Azure App Service Web アプリとして実装され、ワーカーは、WebJob として実装されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="abdbf-152">Web アプリと WebJob の両方が VM インスタンスを提供する App Service プランに関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="abdbf-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="abdbf-153">メッセージ キューには、Azure Service Bus または Azure Storage キューのいずれかを使用できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="abdbf-154">(図は、Azure Storage キューを示しています)。</span><span class="sxs-lookup"><span data-stu-id="abdbf-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="abdbf-155">Azure Redis Cache は、セッション状態と短い待ち時間でアクセスが必要なその他のデータを格納します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="abdbf-156">Azure CDN を使用して、画像、CSS、HTML などの静的なコンテンツをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="abdbf-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="abdbf-157">ストレージは、アプリケーションのニーズに最適なストレージ テクノロジを選択します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="abdbf-158">複数のストレージ テクノロジ (多言語持続性) を使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="abdbf-159">この概念を示すために、この図には、Azure SQL Database と Azure Cosmos DB を示しています。</span><span class="sxs-lookup"><span data-stu-id="abdbf-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="abdbf-160">詳細については、[App Service Web アプリケーションの参照アーキテクチャ][scalable-web-app]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="abdbf-161">追加の考慮事項</span><span class="sxs-lookup"><span data-stu-id="abdbf-161">Additional considerations</span></span>

- <span data-ttu-id="abdbf-162">ストレージまでキューやワーカーを経由しないトランザクションもあります。</span><span class="sxs-lookup"><span data-stu-id="abdbf-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="abdbf-163">Web フロント エンドは、単純な読み取り/書き込み操作を直接実行できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="abdbf-164">ワーカーは、リソースを消費するタスクまたは実行時間の長いワークフローの設計されています。</span><span class="sxs-lookup"><span data-stu-id="abdbf-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="abdbf-165">ワーカーがまったく必要ない場合もあります。</span><span class="sxs-lookup"><span data-stu-id="abdbf-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="abdbf-166">App Service の組み込み自動スケール機能を使用して、VM インスタンスの数をスケール アウト。</span><span class="sxs-lookup"><span data-stu-id="abdbf-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="abdbf-167">アプリケーションで負荷が、予測可能なパターン通りの場合は、スケジュールに基づく自動スケールを使用します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="abdbf-168">負荷が予測可能でない場合は、メトリックに基づいた自動スケール ルールを使用します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="abdbf-169">Web アプリと WebJob を個別の App Service プランに配置することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="abdbf-170">それにより、別の VM インスタンスでホストされるため、個別に拡張できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="abdbf-171">運用環境とテスト環境では、異なる App Service プランを使用してください。</span><span class="sxs-lookup"><span data-stu-id="abdbf-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="abdbf-172">運用環境とテスト環境で同じプランを使用すると、テストが、運用環境の VM で実行されます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="abdbf-173">デプロイ スロットを使用して展開を管理します。</span><span class="sxs-lookup"><span data-stu-id="abdbf-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="abdbf-174">これにより、ステージング スロットに更新されたバージョンを展開してた後、新しいバージョンに交換できます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="abdbf-175">また、更新バージョンで問題があった場合は、前のバージョンに戻ることもできます。</span><span class="sxs-lookup"><span data-stu-id="abdbf-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md