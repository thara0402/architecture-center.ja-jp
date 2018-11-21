---
title: eコマースのインテリジェントな製品検索エンジン
description: eコマース アプリケーションに世界水準の検索エクスペリエンスを提供します。
author: jelledruyts
ms.date: 09/14/2018
ms.openlocfilehash: a57477c26665b4560671550f6fdd81c2d9505e71
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610670"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a><span data-ttu-id="53307-103">eコマースのインテリジェントな製品検索エンジン</span><span class="sxs-lookup"><span data-stu-id="53307-103">Intelligent product search engine for e-commerce</span></span>

<span data-ttu-id="53307-104">このシナリオ例では、専用の検索サービスを使用することにより、eコマースの顧客に対する検索結果の関連性が大幅に向上することを示します。</span><span class="sxs-lookup"><span data-stu-id="53307-104">This example scenario shows how using a dedicated search service can dramatically increase the relevance of search results for your e-commerce customers.</span></span>

<span data-ttu-id="53307-105">検索は顧客が製品を探して最終的に購入するときに利用する主要なメカニズムなので、検索結果が検索クエリの "_意図_" に関連していること、そしてほぼ即時の結果、言語分析、地理的な場所の一致、フィルター処理、ファセット、オートコンプリート、検索結果の強調表示などを提供することによって、検索エクスペリエンスが大手の検索エンジンに匹敵することが重要になります。</span><span class="sxs-lookup"><span data-stu-id="53307-105">Search is the primary mechanism through which customers find and ultimately purchase products, making it essential that search results are relevant to the _intent_ of the search query, and that the end-to-end search experience matches that of search giants by providing near-instant results, linguistic analysis, geo-location matching, filtering, faceting, autocomplete, hit highlighting, etc.</span></span>

<span data-ttu-id="53307-106">SQL Server や Azure SQL Database などのリレーショナル データベースに製品データが格納されている一般的な eコマース Web アプリケーションを想像してください。</span><span class="sxs-lookup"><span data-stu-id="53307-106">Imagine a typical e-commerce web application with product data stored in a relational database like SQL Server or Azure SQL Database.</span></span> <span data-ttu-id="53307-107">多くの場合、検索クエリは、`LIKE` クエリや[フル テキスト検索][docs-sql-fts]機能を使用して、データベース内で処理されます。</span><span class="sxs-lookup"><span data-stu-id="53307-107">Search queries are often handled inside the database using `LIKE` queries or [Full-Text Search][docs-sql-fts] features.</span></span> <span data-ttu-id="53307-108">代わりに [Azure Search][docs-search] を使用することで、運用データベースをクエリ処理から解放し、顧客に考えられる最良の検索エクスペリエンスを提供するこれらの実装が困難な機能を簡単に使い始めることができます。</span><span class="sxs-lookup"><span data-stu-id="53307-108">By using [Azure Search][docs-search] instead, you free up your operational database from the query processing and you can easily start taking advantage of those hard-to-implement features that provide your customers with the best possible search experience.</span></span> <span data-ttu-id="53307-109">また、Azure Search はサービスとしてのプラットフォーム (PaaS) コンポーネントであるため、インフラストラクチャの管理を心配したり、検索のエキスパートになる必要はありません。</span><span class="sxs-lookup"><span data-stu-id="53307-109">Also, because Azure Search is a platform as a service (PaaS) component, you don't have to worry about managing infrastructure or becoming a search expert.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="53307-110">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="53307-110">Relevant use cases</span></span>

<span data-ttu-id="53307-111">その他の関連するユース ケース:</span><span class="sxs-lookup"><span data-stu-id="53307-111">Other relevant use cases include:</span></span>

* <span data-ttu-id="53307-112">物理的にユーザーの近くにある不動産一覧または店舗の検索。</span><span class="sxs-lookup"><span data-stu-id="53307-112">Finding real estate listings or stores near the user's physical location.</span></span>
* <span data-ttu-id="53307-113">"_最近_" の情報を優先した、ニュース サイトの記事やスポーツ結果の検索。</span><span class="sxs-lookup"><span data-stu-id="53307-113">Searching for articles in a news site or looking for sports results, with a higher preference for more _recent_ information.</span></span>
* <span data-ttu-id="53307-114">政策立案者や公証人など、"_ドキュメント中心_" の組織向けの大規模なリポジトリの検索。</span><span class="sxs-lookup"><span data-stu-id="53307-114">Searching through large repositories for _document-centric_ organizations like policy makers and notaries.</span></span>

<span data-ttu-id="53307-115">最終的には、何らかの形式の検索機能を持つ "_すべての_" アプリケーションが、専用検索サービスからメリットを得ることができます。</span><span class="sxs-lookup"><span data-stu-id="53307-115">Ultimately, _any_ application that has some form of search functionality can benefit from a dedicated search service.</span></span>

## <a name="architecture"></a><span data-ttu-id="53307-116">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="53307-116">Architecture</span></span>

![eコマース向けのインテリジェントな製品検索エンジンに関連する Azure コンポーネントのアーキテクチャの概要][architecture]

<span data-ttu-id="53307-118">このシナリオでは、顧客が製品カタログを検索できる eコマース ソリューションについて説明します。</span><span class="sxs-lookup"><span data-stu-id="53307-118">This scenario covers an e-commerce solution where customers can search through a product catalog.</span></span>
1. <span data-ttu-id="53307-119">顧客は、任意のデバイスから **eコマース Web アプリケーション**に移動します。</span><span class="sxs-lookup"><span data-stu-id="53307-119">Customers navigate to the **e-commerce web application** from any device.</span></span>
2. <span data-ttu-id="53307-120">製品カタログは、トランザクション処理のために **Azure SQL Database** に保持されています。</span><span class="sxs-lookup"><span data-stu-id="53307-120">The product catalog is maintained in an **Azure SQL Database** for transactional processing.</span></span>
3. <span data-ttu-id="53307-121">Azure Search は、**検索インデクサー**を使用し、統合された変更追跡機能によって、検索インデックスを自動的に最新の状態に保ちます。</span><span class="sxs-lookup"><span data-stu-id="53307-121">Azure Search uses a **search indexer** to automatically keep its search index up-to-date through integrated change tracking.</span></span>
4. <span data-ttu-id="53307-122">顧客の検索クエリは **Azure Search** サービスにオフロードされ、そこでクエリが処理されて、最も関連性の高い結果が返されます。</span><span class="sxs-lookup"><span data-stu-id="53307-122">Customer's search queries are offloaded to the **Azure Search** service, which processes the query and returns the most relevant results.</span></span>
5. <span data-ttu-id="53307-123">Web ベースの検索エクスペリエンスの代替手段として、顧客は、ソーシャル メディア内で、またはデジタル アシスタントから直接**会話型ボット**を使用して、製品を検索し、検索クエリと結果を段階的に調整することもできます。</span><span class="sxs-lookup"><span data-stu-id="53307-123">As an alternative to a web-based search experience, customers can also use a **conversational bot** in social media or straight from digital assistants to search for products and incrementally refine their search query and results.</span></span>
6. <span data-ttu-id="53307-124">必要に応じて、**コグニティブ検索**機能を使用して人工知能を適用し、処理をさらにスマートにすることができます。</span><span class="sxs-lookup"><span data-stu-id="53307-124">Optionally, the **Cognitive Search** feature can be used to apply artificial intelligence for even smarter processing.</span></span>

### <a name="components"></a><span data-ttu-id="53307-125">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="53307-125">Components</span></span>

* <span data-ttu-id="53307-126">[App Services - Web Apps][docs-webapps] により Web アプリケーションがホストされ、自動スケーリングと高可用性が可能になります。インフラストラクチャの管理は不要です。</span><span class="sxs-lookup"><span data-stu-id="53307-126">[App Services - Web Apps][docs-webapps] hosts web applications allowing autoscale and high availability without having to manage infrastructure.</span></span>
* <span data-ttu-id="53307-127">[SQL Database][docs-sql-database] は、リレーショナル データ、JSON、空間、XML などの構造をサポートする、Microsoft Azure における汎用リレーショナル データベース管理サービスです。</span><span class="sxs-lookup"><span data-stu-id="53307-127">[SQL Database][docs-sql-database] is a general-purpose relational database-managed service in Microsoft Azure that supports structures such as relational data, JSON, spatial, and XML.</span></span>
* <span data-ttu-id="53307-128">[Azure Search][docs-search] はサービスとしての検索クラウド ソリューションで、多彩な検索機能を、Web、モバイル、およびエンタープライズ アプリケーションのプライベートな異種コンテンツに提供します。</span><span class="sxs-lookup"><span data-stu-id="53307-128">[Azure Search][docs-search] is a search-as-a-service cloud solution that provides a rich search experience over private, heterogenous content in web, mobile, and enterprise applications.</span></span>
* <span data-ttu-id="53307-129">[Bot Service][docs-botservice]: インテリジェント ボットの構築、テスト、デプロイ、管理を行うツールを提供します。</span><span class="sxs-lookup"><span data-stu-id="53307-129">[Bot Service][docs-botservice] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="53307-130">[Cognitive Services][docs-cognitive]: 自然なコミュニケーション手段を通じて、見る、聞く、話す、理解する、ユーザーのニーズを解釈することが可能なインテリジェントなアルゴリズムを使用できます。</span><span class="sxs-lookup"><span data-stu-id="53307-130">[Cognitive Services][docs-cognitive] lets you use intelligent algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.</span></span>

### <a name="alternatives"></a><span data-ttu-id="53307-131">代替手段</span><span class="sxs-lookup"><span data-stu-id="53307-131">Alternatives</span></span>

* <span data-ttu-id="53307-132">たとえば、SQL Server のフルテキスト検索で**データベース内検索**機能を使用できますが、その場合、トランザクション ストアでもクエリが処理され (必要な処理能力の増加)、データベース内の検索機能の方が限定的です。</span><span class="sxs-lookup"><span data-stu-id="53307-132">You could use **in-database search** capabilities, for example, through SQL Server full-text search, but then your transactional store also processes queries (increasing the need for processing power) and the search capabilities inside the database are more limited.</span></span>
* <span data-ttu-id="53307-133">オープン ソースの [Apache Lucene][apache-lucene] (その上に Azure Search が構築されます) を Azure Virtual Machines でホストできますが、そうすると、インフラストラクチャとしてのサービス (IaaS) の管理に戻り、Azure Search が Lucene 上で提供する多くの機能のメリットを得られません。</span><span class="sxs-lookup"><span data-stu-id="53307-133">You could host the open-source [Apache Lucene][apache-lucene] (on which Azure Search is built upon) on Azure Virtual Machines, but then you are back to managing Infrastructure-as-a-Service (IaaS) and don't benefit from the many features that Azure Search provides on top of Lucene.</span></span>
* <span data-ttu-id="53307-134">Azure Marketplace からの [Elastic Search][elastic-marketplace] の展開を検討することもできます。これは代替機能であり、サード パーティ ベンダーからの製品を検索できますが、この場合も IaaS ワークロードを実行することになります。</span><span class="sxs-lookup"><span data-stu-id="53307-134">You could also consider deploying [Elastic Search][elastic-marketplace] from the Azure Marketplace, which is an alternative and capable search product from a third-party vendor, but also in this case you are running an IaaS workload.</span></span>

<span data-ttu-id="53307-135">データ層の他のオプションを次に示します。</span><span class="sxs-lookup"><span data-stu-id="53307-135">Other options for the data tier include:</span></span>

* <span data-ttu-id="53307-136">[Cosmos DB](/azure/cosmos-db/introduction) は、Microsoft のグローバル分散型マルチモデル データベースです。</span><span class="sxs-lookup"><span data-stu-id="53307-136">[Cosmos DB](/azure/cosmos-db/introduction) - Microsoft's globally distributed, multi-model database.</span></span> <span data-ttu-id="53307-137">Costmos DB では、Mongo DB、Cassandra、Graph データ、シンプルなテーブル ストレージなど、他のデータ モデルを実行するためのプラットフォームが提供されます。</span><span class="sxs-lookup"><span data-stu-id="53307-137">Costmos DB provides a platform to run other data models such as Mongo DB, Cassandra, Graph data, or simple table storage.</span></span> <span data-ttu-id="53307-138">Azure Search では、Cosmos DB からのデータに直接インデックスを付けることもサポートされています。</span><span class="sxs-lookup"><span data-stu-id="53307-138">Azure Search also supports indexing the data from Cosmos DB directly.</span></span>

## <a name="considerations"></a><span data-ttu-id="53307-139">考慮事項</span><span class="sxs-lookup"><span data-stu-id="53307-139">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="53307-140">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="53307-140">Scalability</span></span>

<span data-ttu-id="53307-141">Azure Search Service の[価格レベル][search-tier]では、使用できる機能は決まりませんが、取得できる最大ストレージおよびプロビジョニングできるパーティションとレプリカの数が定義されるので、主として[キャパシティ プランニング][search-capacity]に使用されます。</span><span class="sxs-lookup"><span data-stu-id="53307-141">The [pricing tier][search-tier] of the Azure Search service doesn't determine the available features but is used mainly for [capacity planning][search-capacity] as it defines the maximum storage you get and how many partitions and replicas you can provision.</span></span> <span data-ttu-id="53307-142">**パーティション**を使用するとより多くのドキュメントにインデックスを付けることができ、書き込みスループットが向上するのに対し、**レプリカ**では 1 秒あたりのクエリ数 (QPS) が増えて高可用性が提供されます。</span><span class="sxs-lookup"><span data-stu-id="53307-142">**Partitions** allow you to index more documents and get higher write throughputs, whereas **replicas** provide more Queries-Per-Second (QPS) and High Availability.</span></span>

<span data-ttu-id="53307-143">パーティションとレプリカの数を動的に変更することはできますが、価格レベルは変更できないので、対象のワークロードに適したレベルを慎重に検討する必要があります。</span><span class="sxs-lookup"><span data-stu-id="53307-143">You can dynamically change the number of partitions and replicas but it's not possible to change the pricing tier, so you should carefully consider the right tier for your target workload.</span></span> <span data-ttu-id="53307-144">どうしてもレベルを変更する必要がある場合は、新しいサービスを並行してプロビジョニングし、インデックスを読み込みなおす必要があります。この時点で、新しいサービスをアプリケーションから参照できます。</span><span class="sxs-lookup"><span data-stu-id="53307-144">If you need to change the tier anyway, you will need to provision a new service side by side and reload your indexes there, at which point you can point your applications at the new service.</span></span>

### <a name="availability"></a><span data-ttu-id="53307-145">可用性</span><span class="sxs-lookup"><span data-stu-id="53307-145">Availability</span></span>

<span data-ttu-id="53307-146">Azure Search では、"_読み取り_" (つまり、クエリ) については少なくとも 2 つのレプリカがある場合に、"_更新_" (つまり、検索インデックスの更新) については少なくとも 3 つのレプリカがある場合に、[99.9% の可用性の SLA][search-sla] が提供されます。</span><span class="sxs-lookup"><span data-stu-id="53307-146">Azure Search provides a [99.9% availability SLA][search-sla] for _reads_ (that is, querying) if you have at least two replicas, and for _updates_ (that is, updating the search indexes) if you have at least three replicas.</span></span> <span data-ttu-id="53307-147">したがって、顧客が確実に "_検索_" できるようにする場合は少なくとも 2 つのレプリカをプロビジョニングし、実際の "_インデックスに対する変更_" も高可用性操作と考える必要がある場合は少なくとも 3 つのレプリカをプロビジョニングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="53307-147">Therefore you should provision at least two replicas if you want your customers to be able to _search_ reliably, and 3 if actual _changes to the index_ should also be considered high availability operations.</span></span>

<span data-ttu-id="53307-148">インデックスに対する破壊的変更 (たとえば、データ型の変更や、フィールドの削除または名前変更) をダウンタイムなしで行う必要がある場合は、インデックスを再構築する必要があります。</span><span class="sxs-lookup"><span data-stu-id="53307-148">If there is a need to make breaking changes to the index without downtime (for example, changing data types, deleting or renaming fields), the index will need to be rebuilt.</span></span> <span data-ttu-id="53307-149">サービス レベルの変更と同様に、これは、新しいインデックスを作成し、それにデータを再設定して、新しいインデックスを参照するようアプリケーションを更新することを意味します。</span><span class="sxs-lookup"><span data-stu-id="53307-149">Similar to changing service tier, this means creating a new index, repopulating it with the data, and then updating your applications to point at the new index.</span></span>

### <a name="security"></a><span data-ttu-id="53307-150">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="53307-150">Security</span></span>

<span data-ttu-id="53307-151">Azure Search は多くの[セキュリティおよびデータ プライバシーの標準][search-security]に準拠しているので、ほとんどの業界で使用できます。</span><span class="sxs-lookup"><span data-stu-id="53307-151">Azure Search is compliant with many [security and data privacy standards][search-security], which makes it possible to be used in most industries.</span></span>

<span data-ttu-id="53307-152">サービスへのアクセスをセキュリティ保護するため、Azure Search では 2 種類のキーが使用されています。**管理者キー**ではサービスに対して "_すべての_" タスクを実行できるのに対し、**クエリ キー**はクエリのような読み取り専用操作に対してのみ使用できます。</span><span class="sxs-lookup"><span data-stu-id="53307-152">For securing access to the service, Azure Search uses two types of keys: **admin keys**, which allow you to perform _any_ task against the service, and **query keys**, which can only be used for read-only operations like querying.</span></span> <span data-ttu-id="53307-153">通常、検索を実行するアプリケーションはインデックスを更新しないため、クエリ キーのみを構成し、管理者キーは構成しないようにする必要があります (特に、Web ブラウザーで実行されるスクリプトのように、エンド ユーザーのデバイスから検索が実行される場合)。</span><span class="sxs-lookup"><span data-stu-id="53307-153">Typically, the application that performs the search does not update the index, so it should only be configured with a query key and not an admin key (especially if the search is performed from an end-user device like script running in a web browser).</span></span>

### <a name="search-relevance"></a><span data-ttu-id="53307-154">検索の関連性</span><span class="sxs-lookup"><span data-stu-id="53307-154">Search Relevance</span></span>

<span data-ttu-id="53307-155">eコマース アプリケーションがどれくらい成功するかは、顧客に対する検索結果の関連性に大きく依存します。</span><span class="sxs-lookup"><span data-stu-id="53307-155">How successful your e-commerce application is depends largely on the relevance of the search results to your customers.</span></span> <span data-ttu-id="53307-156">ユーザーの研究に基づいて最適な結果を提供するように検索サービスを慎重に調整するか、または[検索トラフィック分析][search-analysis]などの組み込み機能を利用して顧客の検索パターンを理解することにより、データに基づいて決定を行うことができます。</span><span class="sxs-lookup"><span data-stu-id="53307-156">Carefully tuning your search service to provide optimal results based on user research, or relying on built-in features such as [search traffic analysis][search-analysis] to understand your customer's search patterns allows you to make decisions based on data.</span></span>

<span data-ttu-id="53307-157">検索サービスを調整する一般的な方法は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="53307-157">Typical ways to tune your search service include:</span></span>

* <span data-ttu-id="53307-158">[スコアリング プロファイル][search-scoring]を使用して、検索結果の関連性に影響を与えます。たとえば、クエリと一致したフィールド、データの新しさ、ユーザーに対する地理的な距離などを基にします。</span><span class="sxs-lookup"><span data-stu-id="53307-158">Using [scoring profiles][search-scoring] to influence the relevance of search results, for example, based on which field matched the query, how recent the data is, the geographical distance to the user, ...</span></span>
* <span data-ttu-id="53307-159">高度な自然言語処理 (NLP) スタックを使用してクエリをより適切に解釈する [Microsoft 提供の言語アナライザー][search-languages]を使用します。</span><span class="sxs-lookup"><span data-stu-id="53307-159">Using [Microsoft provided language analyzers][search-languages] that use an advanced Natural Language Processing (NLP) stack to better interpret queries</span></span>
* <span data-ttu-id="53307-160">[カスタム アナライザー][search-analyzers]を使用して、製品が正しく検出されるようにします。特に、製品の製造元やモデルのような言語に基づかない情報で検索する場合。</span><span class="sxs-lookup"><span data-stu-id="53307-160">Using [custom analyzers][search-analyzers] to ensure your products are found correctly, especially if you want to search on non-language based information like a product's make and model.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="53307-161">このシナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="53307-161">Deploy this scenario</span></span>

<span data-ttu-id="53307-162">このシナリオのさらに完全な eコマース バージョンを展開するには、簡単なチケット購入アプリケーションを実行する .NET サンプル アプリケーションを提供するこちらの[ステップ バイ ステップ チュートリアル][end-to-end-walkthrough]に従ってください。</span><span class="sxs-lookup"><span data-stu-id="53307-162">To deploy a more complete e-commerce version of this scenario, you can follow this [step-by-step tutorial][end-to-end-walkthrough] that provides a .NET sample application that runs a simple ticket purchasing application.</span></span> <span data-ttu-id="53307-163">これには Azure Search も含まれ、説明した機能の多くが使用されています。</span><span class="sxs-lookup"><span data-stu-id="53307-163">It also includes Azure Search and uses many of the features discussed.</span></span> <span data-ttu-id="53307-164">さらに、ほとんどの Azure リソースのデプロイを自動化する Resource Manager テンプレートもあります。</span><span class="sxs-lookup"><span data-stu-id="53307-164">Additionally, there is a Resource Manager template to automate the deployment of most of the Azure resources.</span></span>

## <a name="pricing"></a><span data-ttu-id="53307-165">価格</span><span class="sxs-lookup"><span data-stu-id="53307-165">Pricing</span></span>

<span data-ttu-id="53307-166">このシナリオの実行コストを調べることができるように、これまでに説明したすべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="53307-166">To explore the cost of running this scenario, all the services mentioned above are pre-configured in the cost calculator.</span></span> <span data-ttu-id="53307-167">特定のユース ケースについて価格の変化を確認するには、予想される使用量に合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="53307-167">To see how the pricing would change for your particular use case change the appropriate variables to match your expected usage.</span></span>

<span data-ttu-id="53307-168">取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="53307-168">We have provided three sample cost profiles based on amount of traffic you expect to get:</span></span>

* <span data-ttu-id="53307-169">[小][small-pricing]: このプロファイルでは、Web サイトをホストするための 1 つの `Standard S1` Web アプリ、Azure Bot Service の Free レベル、1 つの `Basic` Azure Search Service、`Standard S2` SQL Database を使用しています。</span><span class="sxs-lookup"><span data-stu-id="53307-169">[Small][small-pricing]: In this profile, we're using a single `Standard S1` Web App to host the website, the free tier of the Azure Bot service, a single `Basic` Azure Search service, and a `Standard S2` SQL Database.</span></span>
* <span data-ttu-id="53307-170">[中][medium-pricing]: ここでは、Web アプリを `Standard S3` レベルの 2 つのインスタンスにスケールアップし、Search Service を `Standard S1` レベルにアップグレードし、`Standard S6` SQL Database を使用しています。</span><span class="sxs-lookup"><span data-stu-id="53307-170">[Medium][medium-pricing]: Here we are scaling up the Web App to two instances of the `Standard S3` tier, upgrading the Search Service to a `Standard S1` tier, and using a `Standard S6` SQL Database.</span></span>
* <span data-ttu-id="53307-171">[大][large-pricing]: 最大のプロファイルでは、`Premium P2V2` Web アプリの 4 つのインスタンスを使用し、Azure Bot Service を `Standard S1` レベル (Premium チャネルで 1.000.000 メッセージ) にアップグレードし、2 ユニットの `Standard S3` Azure Search Service と `Premium P6` SQL Database を使用しています。</span><span class="sxs-lookup"><span data-stu-id="53307-171">[Large][large-pricing]: In the largest profile, we use four instances of a `Premium P2V2` Web App, upgrade the Azure Bot service to the `Standard S1` tier (with 1.000.000 messages in Premium channels), use 2 units of the `Standard S3` Azure Search service, and a `Premium P6` SQL Database.</span></span>

## <a name="related-resources"></a><span data-ttu-id="53307-172">関連リソース</span><span class="sxs-lookup"><span data-stu-id="53307-172">Related resources</span></span>

<span data-ttu-id="53307-173">Azure Search について詳しくは、[ドキュメント センター][docs-search]を参照するか、[サンプル][search-samples]を確認するか、本格的に動作する[デモ サイト][search-demo]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="53307-173">To learn more about Azure Search, visit the [documentation center][docs-search], check out the [samples][search-samples], or see a full fledged [demo site][search-demo] in action.</span></span>

<!-- links -->
[architecture]: ./media/architecture-ecommerce-search.png
[docs-sql-fts]: /sql/relational-databases/search/query-with-full-text-search
[docs-search]: /azure/search/search-what-is-azure-search
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[docs-botservice]: /azure/bot-service/
[docs-cognitive]: /azure/cognitive-services/
[apache-lucene]: https://lucene.apache.org/
[elastic-marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[search-sla]: https://go.microsoft.com/fwlink/?LinkId=716855
[search-tier]: /azure/search/search-sku-tier
[search-capacity]: /azure/search/search-capacity-planning
[search-security]: /azure/search/search-security-overview
[search-analysis]: /azure/search/search-traffic-analytics
[search-languages]: /rest/api/searchservice/language-support
[search-analyzers]: /rest/api/searchservice/custom-analyzers-in-azure-search
[search-scoring]: /rest/api/searchservice/add-scoring-profiles-to-a-search-index
[search-samples]: https://azure.microsoft.com/resources/samples/?service=search&sort=0
[search-demo]: https://azjobsdemo.azurewebsites.net/
[small-pricing]: https://azure.com/e/db2672a55b6b4d768ef0060a8d9759bd
[medium-pricing]: https://azure.com/e/a5ad0706c9e74add811e83ef83766a1c
[large-pricing]: https://azure.com/e/57f95a898daa487795bd305599973ee6
