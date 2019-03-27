---
title: スケーラブルな Web アプリケーション
titleSuffix: Azure Reference Architectures
description: Microsoft Azure で実行される Web アプリケーションのスケーラビリティを向上させます。
author: MikeWasson
ms.date: 10/25/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: b015a130a74f3160c0e737b436d41e1b1ea7b5bf
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241373"
---
# <a name="improve-scalability-in-an-azure-web-application"></a><span data-ttu-id="47377-103">Azure Web アプリケーションのスケーラビリティの向上</span><span class="sxs-lookup"><span data-stu-id="47377-103">Improve scalability in an Azure web application</span></span>

<span data-ttu-id="47377-104">この参照アーキテクチャは、Azure App Service Web アプリケーションでスケーラビリティとパフォーマンスを改善するための実証済みの方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="47377-104">This reference architecture shows proven practices for improving scalability and performance in an Azure App Service web application.</span></span>

![スケーラビリティが改善された Azure の Web アプリケーション](./images/scalable-web-app.png)

<span data-ttu-id="47377-106">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="47377-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="47377-107">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="47377-107">Architecture</span></span>

<span data-ttu-id="47377-108">このアーキテクチャは、「[Basic web application][basic-web-app]」(基本的な Web アプリケーション) で説明されているアーキテクチャに基づいて構築されています。</span><span class="sxs-lookup"><span data-stu-id="47377-108">This architecture builds on the one shown in [Basic web application][basic-web-app].</span></span> <span data-ttu-id="47377-109">アーキテクチャに含まれるコンポーネントを次に示します。</span><span class="sxs-lookup"><span data-stu-id="47377-109">It includes the following components:</span></span>

- <span data-ttu-id="47377-110">**リソース グループ**。</span><span class="sxs-lookup"><span data-stu-id="47377-110">**Resource group**.</span></span> <span data-ttu-id="47377-111">[リソース グループ][resource-group]は、Azure リソースの論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="47377-111">A [resource group][resource-group] is a logical container for Azure resources.</span></span>
- <span data-ttu-id="47377-112">**[Web アプリ][app-service-web-app]**。</span><span class="sxs-lookup"><span data-stu-id="47377-112">**[Web app][app-service-web-app]**.</span></span> <span data-ttu-id="47377-113">最新の一般的なアプリケーションには、Web サイトと、1 つ以上の RESTful Web API の両方が含まれていることがあります。</span><span class="sxs-lookup"><span data-stu-id="47377-113">A typical modern application might include both a website and one or more RESTful web APIs.</span></span> <span data-ttu-id="47377-114">Web API は、ブラウザー クライアント (AJAX 経由)、ネイティブ クライアント アプリケーション、およびサーバー側アプリケーションから使用できます。</span><span class="sxs-lookup"><span data-stu-id="47377-114">A web API might be consumed by browser clients through AJAX, by native client applications, or by server-side applications.</span></span> <span data-ttu-id="47377-115">Web API の考慮事項については、[API 設計のガイダンス][api-guidance]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-115">For considerations on designing web APIs, see [API design guidance][api-guidance].</span></span>
- <span data-ttu-id="47377-116">**Function App**。</span><span class="sxs-lookup"><span data-stu-id="47377-116">**Function App**.</span></span> <span data-ttu-id="47377-117">[Function App][functions] を使用してバックグラウンド タスクを実行します。</span><span class="sxs-lookup"><span data-stu-id="47377-117">Use [Function Apps][functions] to run background tasks.</span></span> <span data-ttu-id="47377-118">関数は、タイマー イベントや、キューに配置されているメッセージなどのトリガーによって呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="47377-118">Functions are invoked by a trigger, such as a timer event or a message being placed on queue.</span></span> <span data-ttu-id="47377-119">実行時間の長いステートフル タスクについては、[Durable Functions][durable-functions] を使用します。</span><span class="sxs-lookup"><span data-stu-id="47377-119">For long-running stateful tasks, use [Durable Functions][durable-functions].</span></span>
- <span data-ttu-id="47377-120">**キュー**。</span><span class="sxs-lookup"><span data-stu-id="47377-120">**Queue**.</span></span> <span data-ttu-id="47377-121">ここで示すアーキテクチャの場合、アプリケーションはメッセージを [Azure Queue Storage][queue-storage] キューに格納することで、バックグラウンド タスクをキューに格納します。</span><span class="sxs-lookup"><span data-stu-id="47377-121">In the architecture shown here, the application queues background tasks by putting a message onto an [Azure Queue storage][queue-storage] queue.</span></span> <span data-ttu-id="47377-122">このメッセージによって、関数アプリがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="47377-122">The message triggers a function app.</span></span> <span data-ttu-id="47377-123">または、Service Bus キューを使用できます。</span><span class="sxs-lookup"><span data-stu-id="47377-123">Alternatively, you can use Service Bus queues.</span></span> <span data-ttu-id="47377-124">比較については、[Storage キューと Service Bus キューの比較][queues-compared]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-124">For a comparison, see [Azure Queues and Service Bus queues - compared and contrasted][queues-compared].</span></span>
- <span data-ttu-id="47377-125">**キャッシュ**。</span><span class="sxs-lookup"><span data-stu-id="47377-125">**Cache**.</span></span> <span data-ttu-id="47377-126">[Azure Redis Cache][azure-redis] に半静的データを格納します。</span><span class="sxs-lookup"><span data-stu-id="47377-126">Store semi-static data in [Azure Redis Cache][azure-redis].</span></span>
- <span data-ttu-id="47377-127">**CDN**。</span><span class="sxs-lookup"><span data-stu-id="47377-127">**CDN**.</span></span> <span data-ttu-id="47377-128">[Azure Content Delivery Network][azure-cdn] (CDN) を使用して、コンテンツの待機時間を短縮して高速に配信できるように、パブリックに使用できるコンテンツをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="47377-128">Use [Azure Content Delivery Network][azure-cdn] (CDN) to cache publicly available content for lower latency and faster delivery of content.</span></span>
- <span data-ttu-id="47377-129">**データ ストレージ**。</span><span class="sxs-lookup"><span data-stu-id="47377-129">**Data storage**.</span></span> <span data-ttu-id="47377-130">リレーショナル データには [Azure SQL Database][sql-db] を使用します。</span><span class="sxs-lookup"><span data-stu-id="47377-130">Use [Azure SQL Database][sql-db] for relational data.</span></span> <span data-ttu-id="47377-131">リレーショナル データ以外については、[Cosmos DB][cosmosdb] を検討してください。</span><span class="sxs-lookup"><span data-stu-id="47377-131">For non-relational data, consider [Cosmos DB][cosmosdb].</span></span>
- <span data-ttu-id="47377-132">**Azure Search**。</span><span class="sxs-lookup"><span data-stu-id="47377-132">**Azure Search**.</span></span> <span data-ttu-id="47377-133">[Azure Search][azure-search] を使用して、検索候補、あいまい検索、言語固有の検索などの検索機能を追加します。</span><span class="sxs-lookup"><span data-stu-id="47377-133">Use [Azure Search][azure-search] to add search functionality such as search suggestions, fuzzy search, and language-specific search.</span></span> <span data-ttu-id="47377-134">通常、Azure Search は別のデータ ストアと組み合わせて使用されます。プライマリ データ ストアに厳密な一貫性が必要な場合は特にそうです。</span><span class="sxs-lookup"><span data-stu-id="47377-134">Azure Search is typically used in conjunction with another data store, especially if the primary data store requires strict consistency.</span></span> <span data-ttu-id="47377-135">この方法では、信頼できるデータを他のデータ ストアに格納し、検索インデックスは Azure Search に格納します。</span><span class="sxs-lookup"><span data-stu-id="47377-135">In this approach, store authoritative data in the other data store and the search index in Azure Search.</span></span> <span data-ttu-id="47377-136">また、Azure Search は、複数のデータ ストアから単一の検索インデックスに統合する場合にも使用できます。</span><span class="sxs-lookup"><span data-stu-id="47377-136">Azure Search can also be used to consolidate a single search index from multiple data stores.</span></span>
- <span data-ttu-id="47377-137">**Azure DNS**。</span><span class="sxs-lookup"><span data-stu-id="47377-137">**Azure DNS**.</span></span> <span data-ttu-id="47377-138">[Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。</span><span class="sxs-lookup"><span data-stu-id="47377-138">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="47377-139">Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。</span><span class="sxs-lookup"><span data-stu-id="47377-139">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>
- <span data-ttu-id="47377-140">**Application Gateway**。</span><span class="sxs-lookup"><span data-stu-id="47377-140">**Application gateway**.</span></span> <span data-ttu-id="47377-141">[Application Gateway](/azure/application-gateway/) はレイヤー 7 のロード バランサーです。</span><span class="sxs-lookup"><span data-stu-id="47377-141">[Application Gateway](/azure/application-gateway/) is a layer 7 load balancer.</span></span> <span data-ttu-id="47377-142">このアーキテクチャでは、これによって HTTP 要求が Web フロント エンドにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="47377-142">In this architecture, it routes HTTP requests to the web front end.</span></span> <span data-ttu-id="47377-143">Application Gateway には、一般的な脆弱性やその悪用からアプリケーションを保護する [Web アプリケーション ファイアウォール](/azure/application-gateway/waf-overview) (WAF) も用意されています。</span><span class="sxs-lookup"><span data-stu-id="47377-143">Application Gateway also provides a [web application firewall](/azure/application-gateway/waf-overview) (WAF) that protects the application from common exploits and vulnerabilities.</span></span>

## <a name="recommendations"></a><span data-ttu-id="47377-144">Recommendations</span><span class="sxs-lookup"><span data-stu-id="47377-144">Recommendations</span></span>

<span data-ttu-id="47377-145">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="47377-145">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="47377-146">このセクションに記載されている推奨事項は出発点として利用してください。</span><span class="sxs-lookup"><span data-stu-id="47377-146">Use the recommendations in this section as a starting point.</span></span>

### <a name="app-service-apps"></a><span data-ttu-id="47377-147">App Service アプリ</span><span class="sxs-lookup"><span data-stu-id="47377-147">App Service apps</span></span>

<span data-ttu-id="47377-148">Web アプリケーションと Web API は、別の App Service アプリとして作成することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="47377-148">We recommend creating the web application and the web API as separate App Service apps.</span></span> <span data-ttu-id="47377-149">この設計にすると、別の App Service プランで実行して個別にスケーリングできるようになります。</span><span class="sxs-lookup"><span data-stu-id="47377-149">This design lets you run them in separate App Service plans so they can be scaled independently.</span></span> <span data-ttu-id="47377-150">初期段階でこのレベルのスケーラビリティが不要な場合は、アプリを同じプランにデプロイし、後で必要に応じて別のプランに移行することができます。</span><span class="sxs-lookup"><span data-stu-id="47377-150">If you don't need that level of scalability initially, you can deploy the apps into the same plan and move them into separate plans later if necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="47377-151">Basic、Standard、および Premium プランの場合、アプリごとではなく、そのプランの VM インスタンスに対して課金されます。</span><span class="sxs-lookup"><span data-stu-id="47377-151">For the Basic, Standard, and Premium plans, you are billed for the VM instances in the plan, not per app.</span></span> <span data-ttu-id="47377-152">「[App Service の価格][app-service-pricing]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-152">See [App Service Pricing][app-service-pricing]</span></span>
>

### <a name="cache"></a><span data-ttu-id="47377-153">キャッシュ</span><span class="sxs-lookup"><span data-stu-id="47377-153">Cache</span></span>

<span data-ttu-id="47377-154">パフォーマンスとスケーラビリティを改善するには、[Azure Redis Cache][azure-redis] を使用して一部のデータをキャッシュします。</span><span class="sxs-lookup"><span data-stu-id="47377-154">You can improve performance and scalability by using [Azure Redis Cache][azure-redis] to cache some data.</span></span> <span data-ttu-id="47377-155">以下の場合に Redis Cache の使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="47377-155">Consider using Redis Cache for:</span></span>

- <span data-ttu-id="47377-156">半静的なトランザクション データ。</span><span class="sxs-lookup"><span data-stu-id="47377-156">Semi-static transaction data.</span></span>
- <span data-ttu-id="47377-157">セッションの状態。</span><span class="sxs-lookup"><span data-stu-id="47377-157">Session state.</span></span>
- <span data-ttu-id="47377-158">HTML 出力。</span><span class="sxs-lookup"><span data-stu-id="47377-158">HTML output.</span></span> <span data-ttu-id="47377-159">複雑な HTML 出力を表示するアプリケーションの場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="47377-159">This can be useful in applications that render complex HTML output.</span></span>

<span data-ttu-id="47377-160">キャッシュ戦略の設計の詳細については、[キャッシュのガイダンス][caching-guidance]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-160">For more detailed guidance on designing a caching strategy, see [Caching guidance][caching-guidance].</span></span>

### <a name="cdn"></a><span data-ttu-id="47377-161">CDN</span><span class="sxs-lookup"><span data-stu-id="47377-161">CDN</span></span>

<span data-ttu-id="47377-162">静的コンテンツのキャッシュには、[Azure CDN][azure-cdn] を使用します。</span><span class="sxs-lookup"><span data-stu-id="47377-162">Use [Azure CDN][azure-cdn] to cache static content.</span></span> <span data-ttu-id="47377-163">CDN の主な利点は、ユーザーに地理的に近いエッジ サーバーにコンテンツがキャッシュされるため、ユーザーの待機時間が短くなることです。</span><span class="sxs-lookup"><span data-stu-id="47377-163">The main benefit of a CDN is to reduce latency for users, because content is cached at an edge server that is geographically close to the user.</span></span> <span data-ttu-id="47377-164">また、トラフィックがアプリケーションで処理されないため、CDN でアプリケーションの負荷を減らすこともできます。</span><span class="sxs-lookup"><span data-stu-id="47377-164">CDN can also reduce load on the application, because that traffic is not being handled by the application.</span></span>

<span data-ttu-id="47377-165">ほぼ静的なページで構成されるアプリの場合は、[CDN を使用してアプリ全体をキャッシュする][cdn-app-service]ことを検討してください。</span><span class="sxs-lookup"><span data-stu-id="47377-165">If your app consists mostly of static pages, consider using [CDN to cache the entire app][cdn-app-service].</span></span> <span data-ttu-id="47377-166">それ以外の場合は、画像、CSS、HTML ファイルなどの静的コンテンツを [Azure Storage に格納し、CDN を使用してそれらのファイルをキャッシュ][cdn-storage-account]します。</span><span class="sxs-lookup"><span data-stu-id="47377-166">Otherwise, put static content such as images, CSS, and HTML files, into [Azure Storage and use CDN to cache those files][cdn-storage-account].</span></span>

> [!NOTE]
> <span data-ttu-id="47377-167">Azure CDN では、認証が必要なコンテンツを提供できません。</span><span class="sxs-lookup"><span data-stu-id="47377-167">Azure CDN cannot serve content that requires authentication.</span></span>
>

<span data-ttu-id="47377-168">詳細については、[Content Delivery Network (CDN) のガイダンス][cdn-guidance]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-168">For more detailed guidance, see [Content Delivery Network (CDN) guidance][cdn-guidance].</span></span>

### <a name="storage"></a><span data-ttu-id="47377-169">Storage</span><span class="sxs-lookup"><span data-stu-id="47377-169">Storage</span></span>

<span data-ttu-id="47377-170">多くの場合、最新のアプリケーションは大量のデータを処理しています。</span><span class="sxs-lookup"><span data-stu-id="47377-170">Modern applications often process large amounts of data.</span></span> <span data-ttu-id="47377-171">クラウドに合わせてスケーリングするには、適切なストレージの種類を選択することが重要です。</span><span class="sxs-lookup"><span data-stu-id="47377-171">In order to scale for the cloud, it's important to choose the right storage type.</span></span> <span data-ttu-id="47377-172">以下に、基本的な推奨事項をいくつか示します。</span><span class="sxs-lookup"><span data-stu-id="47377-172">Here are some baseline recommendations.</span></span>

| <span data-ttu-id="47377-173">格納するもの</span><span class="sxs-lookup"><span data-stu-id="47377-173">What you want to store</span></span> | <span data-ttu-id="47377-174">例</span><span class="sxs-lookup"><span data-stu-id="47377-174">Example</span></span> | <span data-ttu-id="47377-175">推奨されるストレージ</span><span class="sxs-lookup"><span data-stu-id="47377-175">Recommended storage</span></span> |
| --- | --- | --- |
| <span data-ttu-id="47377-176">ファイル</span><span class="sxs-lookup"><span data-stu-id="47377-176">Files</span></span> |<span data-ttu-id="47377-177">画像、ドキュメント、PDF</span><span class="sxs-lookup"><span data-stu-id="47377-177">Images, documents, PDFs</span></span> |<span data-ttu-id="47377-178">Azure Blob Storage</span><span class="sxs-lookup"><span data-stu-id="47377-178">Azure Blob Storage</span></span> |
| <span data-ttu-id="47377-179">キー/値のペア</span><span class="sxs-lookup"><span data-stu-id="47377-179">Key/Value pairs</span></span> |<span data-ttu-id="47377-180">ユーザー ID で検索されるユーザー プロファイル データ</span><span class="sxs-lookup"><span data-stu-id="47377-180">User profile data looked up by user ID</span></span> |<span data-ttu-id="47377-181">Azure Table Storage</span><span class="sxs-lookup"><span data-stu-id="47377-181">Azure Table storage</span></span> |
| <span data-ttu-id="47377-182">以降の処理をトリガーするための短いメッセージ</span><span class="sxs-lookup"><span data-stu-id="47377-182">Short messages intended to trigger further processing</span></span> |<span data-ttu-id="47377-183">注文要求</span><span class="sxs-lookup"><span data-stu-id="47377-183">Order requests</span></span> |<span data-ttu-id="47377-184">Azure Queue Storage、Service Bus キュー、または Service Bus トピック</span><span class="sxs-lookup"><span data-stu-id="47377-184">Azure Queue storage, Service Bus queue, or Service Bus topic</span></span> |
| <span data-ttu-id="47377-185">基本的なクエリ実行を必要とする柔軟なスキーマの非リレーショナル データ</span><span class="sxs-lookup"><span data-stu-id="47377-185">Non-relational data with a flexible schema requiring basic querying</span></span> |<span data-ttu-id="47377-186">製品カタログ</span><span class="sxs-lookup"><span data-stu-id="47377-186">Product catalog</span></span> |<span data-ttu-id="47377-187">Azure Cosmos DB、MongoDB、Apache CouchDB などのドキュメント データベース</span><span class="sxs-lookup"><span data-stu-id="47377-187">Document database, such as Azure Cosmos DB, MongoDB, or Apache CouchDB</span></span> |
| <span data-ttu-id="47377-188">より高度なクエリのサポート、厳格なスキーマ、強力な一貫性を必要とするリレーショナル データ</span><span class="sxs-lookup"><span data-stu-id="47377-188">Relational data requiring richer query support, strict schema, and/or strong consistency</span></span> |<span data-ttu-id="47377-189">製品のインベントリ</span><span class="sxs-lookup"><span data-stu-id="47377-189">Product inventory</span></span> |<span data-ttu-id="47377-190">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="47377-190">Azure SQL Database</span></span> |

<span data-ttu-id="47377-191">「[Choose the right data store (適切なデータ ストアの選択)][datastore]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="47377-191">See [Choose the right data store][datastore].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="47377-192">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="47377-192">Scalability considerations</span></span>

<span data-ttu-id="47377-193">Azure App Service の主な利点は、負荷に応じてアプリケーションをスケーリングできることです。</span><span class="sxs-lookup"><span data-stu-id="47377-193">A major benefit of Azure App Service is the ability to scale your application based on load.</span></span> <span data-ttu-id="47377-194">アプリケーションのスケーリングを計画する場合の考慮事項を次に示します。</span><span class="sxs-lookup"><span data-stu-id="47377-194">Here are some considerations to keep in mind when planning to scale your application.</span></span>

### <a name="app-service-app"></a><span data-ttu-id="47377-195">App Service アプリ</span><span class="sxs-lookup"><span data-stu-id="47377-195">App Service app</span></span>

<span data-ttu-id="47377-196">ソリューションに複数の App Service アプリが含まれる場合は、別の App Service プランにデプロイすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="47377-196">If your solution includes several App Service apps, consider deploying them to separate App Service plans.</span></span> <span data-ttu-id="47377-197">この方法にすると、別インスタンス上で実行されるので、個別にスケーリングすることができます。</span><span class="sxs-lookup"><span data-stu-id="47377-197">This approach enables you to scale them independently because they run on separate instances.</span></span>

<span data-ttu-id="47377-198">同様に、HTTP 要求を処理するのと同じインスタンス上でバックグラウンド タスクが実行されないように、関数アプリを専用のプランに配置することを検討します。</span><span class="sxs-lookup"><span data-stu-id="47377-198">Similarly, consider putting a function app into its own plan so that background tasks don't run on the same instances that handle HTTP requests.</span></span> <span data-ttu-id="47377-199">バックグラウンド タスクが断続的に実行される場合は、[従量課金プラン][functions-consumption-plan]の使用を検討します。このプランでは、時間単位ではなく、実行数に基づいて課金されます。</span><span class="sxs-lookup"><span data-stu-id="47377-199">If background tasks run intermittently, consider using a [consumption plan][functions-consumption-plan], which is billed based on the number of executions, rather than hourly.</span></span>

### <a name="sql-database"></a><span data-ttu-id="47377-200">SQL Database</span><span class="sxs-lookup"><span data-stu-id="47377-200">SQL Database</span></span>

<span data-ttu-id="47377-201">データベースを*シャード化*することで、SQL Database のスケーラビリティを向上します。</span><span class="sxs-lookup"><span data-stu-id="47377-201">Increase scalability of a SQL database by *sharding* the database.</span></span> <span data-ttu-id="47377-202">シャード化とは、データベースを水平方向にパーティション分割することを指します。</span><span class="sxs-lookup"><span data-stu-id="47377-202">Sharding refers to partitioning the database horizontally.</span></span> <span data-ttu-id="47377-203">シャード化すると、[Elastic Database ツール][sql-elastic]を使用してデータベースを水平方向にスケール アウトできます。</span><span class="sxs-lookup"><span data-stu-id="47377-203">Sharding allows you to scale out the database horizontally using [Elastic Database tools][sql-elastic].</span></span> <span data-ttu-id="47377-204">シャード化には、次のような利点があります。</span><span class="sxs-lookup"><span data-stu-id="47377-204">Potential benefits of sharding include:</span></span>

- <span data-ttu-id="47377-205">トランザクション スループットの改善。</span><span class="sxs-lookup"><span data-stu-id="47377-205">Better transaction throughput.</span></span>
- <span data-ttu-id="47377-206">データのサブセットに対するクエリの実行が高速になります。</span><span class="sxs-lookup"><span data-stu-id="47377-206">Queries can run faster over a subset of the data.</span></span>

### <a name="azure-search"></a><span data-ttu-id="47377-207">Azure Search</span><span class="sxs-lookup"><span data-stu-id="47377-207">Azure Search</span></span>

<span data-ttu-id="47377-208">Azure Search は、プライマリ データ ストアから複雑なデータ検索を実行するオーバーヘッドを取り除き、負荷を処理できるようにスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="47377-208">Azure Search removes the overhead of performing complex data searches from the primary data store, and it can scale to handle load.</span></span> <span data-ttu-id="47377-209">「[Azure Search でクエリとインデックス作成のワークロードに応じてリソース レベルをスケールする][azure-search-scaling]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-209">See [Scale resource levels for query and indexing workloads in Azure Search][azure-search-scaling].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="47377-210">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="47377-210">Security considerations</span></span>

<span data-ttu-id="47377-211">このセクションでは、この記事で説明している Azure サービスに固有のセキュリティの考慮事項について説明します。</span><span class="sxs-lookup"><span data-stu-id="47377-211">This section lists security considerations that are specific to the Azure services described in this article.</span></span> <span data-ttu-id="47377-212">これはセキュリティ上のベスト プラクティスを網羅した一覧ではありません。</span><span class="sxs-lookup"><span data-stu-id="47377-212">It's not a complete list of security best practices.</span></span> <span data-ttu-id="47377-213">その他のセキュリティの考慮事項については、[Azure App Service でのアプリのセキュリティ保護][app-service-security]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="47377-213">For some additional security considerations, see [Secure an app in Azure App Service][app-service-security].</span></span>

### <a name="cross-origin-resource-sharing-cors"></a><span data-ttu-id="47377-214">クロスオリジン リソース共有 (CORS)</span><span class="sxs-lookup"><span data-stu-id="47377-214">Cross-Origin Resource Sharing (CORS)</span></span>

<span data-ttu-id="47377-215">Web サイトと Web API を別のアプリとして作成する場合、CORS を有効にしない限り、Web サイトで API に対するクライアント側 AJAX の呼び出しを行うことはできません。</span><span class="sxs-lookup"><span data-stu-id="47377-215">If you create a website and web API as separate apps, the website cannot make client-side AJAX calls to the API unless you enable CORS.</span></span>

> [!NOTE]
> <span data-ttu-id="47377-216">ブラウザーのセキュリティ機能により、Web ページでは AJAX 要求を別のドメインに送信することはできません。</span><span class="sxs-lookup"><span data-stu-id="47377-216">Browser security prevents a web page from making AJAX requests to another domain.</span></span> <span data-ttu-id="47377-217">この制限は、同一オリジン ポリシーと呼ばれ、悪意のあるサイトが、別のサイトから機密データを読み取れないようにします。</span><span class="sxs-lookup"><span data-stu-id="47377-217">This restriction is called the same-origin policy, and prevents a malicious site from reading sentitive data from another site.</span></span> <span data-ttu-id="47377-218">CORS は W3C 標準であり、サーバーによる同一オリジン ポリシーの緩和を許可し、一部のクロスオリジン要求を許可して他の要求を拒否することができます。</span><span class="sxs-lookup"><span data-stu-id="47377-218">CORS is a W3C standard that allows a server to relax the same-origin policy and allow some cross-origin requests while rejecting others.</span></span>
>

<span data-ttu-id="47377-219">App Services は CORS のサポートが組み込まれているため、アプリケーション コードを作成する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="47377-219">App Services has built-in support for CORS, without needing to write any application code.</span></span> <span data-ttu-id="47377-220">[CORS を使用して JavaScript から API アプリを使用する][cors]方法に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="47377-220">See [Consume an API app from JavaScript using CORS][cors].</span></span> <span data-ttu-id="47377-221">API の許可されているオリジンの一覧に Web サイトを追加します。</span><span class="sxs-lookup"><span data-stu-id="47377-221">Add the website to the list of allowed origins for the API.</span></span>

### <a name="sql-database-encryption"></a><span data-ttu-id="47377-222">SQL Database の暗号化</span><span class="sxs-lookup"><span data-stu-id="47377-222">SQL Database encryption</span></span>

<span data-ttu-id="47377-223">データベースに保存されているデータを暗号化する必要がある場合は、[Transparent Data Encryption][sql-encryption] を使用します。</span><span class="sxs-lookup"><span data-stu-id="47377-223">Use [Transparent Data Encryption][sql-encryption] if you need to encrypt data at rest in the database.</span></span> <span data-ttu-id="47377-224">この機能で、(バックアップとトランザクション ログ ファイルを含む) データベース全体のリアルタイムの暗号化と復号が実行されるので、アプリケーションを変更する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="47377-224">This feature performs real-time encryption and decryption of an entire database (including backups and transaction log files) and requires no changes to the application.</span></span> <span data-ttu-id="47377-225">暗号化で、ある程度の待機時間が増えるので、セキュリティで保護する必要があるデータを独自のデータベースに分離し、そのデータベースについてのみ暗号化を有効にすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="47377-225">Encryption does add some latency, so it's a good practice to separate the data that must be secure into its own database and enable encryption only for that database.</span></span>

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: /azure/search
[azure-search-scaling]: /azure/search/search-capacity-planning
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[datastore]: ../..//guide/technology-choices/data-store-overview.md
[durable-functions]: /azure/azure-functions/durable-functions-overview
[functions]: /azure/azure-functions/functions-overview
[functions-consumption-plan]: /azure/azure-functions/functions-scale#consumption-plan
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: /azure/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
