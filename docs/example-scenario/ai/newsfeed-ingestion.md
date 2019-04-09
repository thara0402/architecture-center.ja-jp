---
title: Azure 上での大量のニュース フィードの取り込みと分析
description: Azure Cosmos DB や Azure Cognitive Services を含めた Azure サービスのみを使用して、RSS ニュース フィードからテキスト、画像、センチメントなどのデータを取り込んで分析するためのパイプラインを作成します。
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489268"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a><span data-ttu-id="32fb0-103">Azure 上での大量のニュース フィードの取り込みと分析</span><span class="sxs-lookup"><span data-stu-id="32fb0-103">Mass ingestion and analysis of news feeds on Azure</span></span>

<span data-ttu-id="32fb0-104">このサンプル シナリオでは、パブリック RSS ニュース フィードを使用したドキュメントの大量の取り込みとほぼリアルタイムの分析のパイプラインについて説明します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-104">This example scenario describes a pipeline for mass ingestion and near real-time analysis of documents using public RSS news feeds.</span></span>  <span data-ttu-id="32fb0-105">ここでは、Azure Cognitive Services を使用して、テキスト翻訳、顔認識、センチメント検出などの有益な洞察を提供します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-105">It uses Azure Cognitive Services to offer useful insights including text translation, facial recognition, and sentiment detection.</span></span>

<span data-ttu-id="32fb0-106">このシナリオに含まれているのは、[英語][english]、[ロシア語][russian]、および[ドイツ語][german]のニュース フィードですが、他の RSS フィードにも容易に拡張することができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-106">This scenario contains examples for [English][english], [Russian][russian], and [German][german] news feeds, but you can easily extend it to other RSS feeds.</span></span> <span data-ttu-id="32fb0-107">デプロイを容易にするために、データの収集、処理、および分析は、すべて Azure サービスに基づいて行われます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-107">For ease of deployment, the data collection, processing, and analysis are based entirely on Azure services.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="32fb0-108">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="32fb0-108">Relevant use cases</span></span>

<span data-ttu-id="32fb0-109">このシナリオは、RSS フィードの処理に基づいて作成されていますが、次のことを実行する必要があるすべてのドキュメント、Web サイト、または記事に関係があります。</span><span class="sxs-lookup"><span data-stu-id="32fb0-109">While this scenario is based on processing of RSS feeds, it's relevant to any document, website, or article where you would need to:</span></span>

* <span data-ttu-id="32fb0-110">任意のテキストを選択した言語に翻訳する。</span><span class="sxs-lookup"><span data-stu-id="32fb0-110">Translate any text to the language of choice.</span></span>
* <span data-ttu-id="32fb0-111">デジタル コンテンツでキー フレーズ、エンティティ、およびユーザーのセンチメントを見つける。</span><span class="sxs-lookup"><span data-stu-id="32fb0-111">Find key phrases, entities, and user sentiment in digital content.</span></span>
* <span data-ttu-id="32fb0-112">ディジタル記事に関連したイメージでオブジェクト、テキスト、およびランドマークを検出する。</span><span class="sxs-lookup"><span data-stu-id="32fb0-112">Detect objects, text, and landmarks in images associated with a digital article.</span></span>
* <span data-ttu-id="32fb0-113">デジタル コンテンツに関連したすべてのイメージで性別と年齢によって人々を検出する。</span><span class="sxs-lookup"><span data-stu-id="32fb0-113">Detect people by their gender and age in any image associated with digital content.</span></span>

## <a name="architecture"></a><span data-ttu-id="32fb0-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="32fb0-114">Architecture</span></span>

![][architecture]

<span data-ttu-id="32fb0-115">このソリューションのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="32fb0-115">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="32fb0-116">RSS ニュース フィードは、ドキュメントや記事からデータを取得するジェネレーターとして機能します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-116">An RSS news feed acts as the generator that obtains data from a document or article.</span></span> <span data-ttu-id="32fb0-117">たとえば、記事の場合、データには通常、ニュース項目のタイトルと元の本文の概要が含まれており、時にはイメージが含まれていることもあります。</span><span class="sxs-lookup"><span data-stu-id="32fb0-117">For example, with an article, data typically includes a title, a summary of the original body of the news item, and sometimes images.</span></span>

2. <span data-ttu-id="32fb0-118">ジェネレーターまたはインジェスト プロセスにより、記事および関連するイメージが Azure Cosmos DB の[コレクション][collection]に挿入されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-118">A generator or ingestion process inserts the article and any associated images into an Azure Cosmos DB [Collection][collection].</span></span>

3. <span data-ttu-id="32fb0-119">通知により、Azure Functions で Ingest 関数がトリガーされます。この関数は、記事のテキストを CosmosDB に格納し、記事のイメージ (ある場合) を Azure Blob Storage に格納します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-119">A notification triggers an ingest function in Azure Functions that stores the article text in CosmosDB and the article images (if any) in Azure Blob Storage.</span></span>  <span data-ttu-id="32fb0-120">それから、記事が次のキューに渡されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-120">The article is then passed to the next queue.</span></span>

4. <span data-ttu-id="32fb0-121">キュー イベントによって Translate 関数がトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-121">A translate function is triggered by the queue event.</span></span> <span data-ttu-id="32fb0-122">Azure Cognitive Services の [Translate Text API][translate-text] を使用して言語が検出され、必要に応じて翻訳が行われて、本文とタイトルからセンチメント、キー フレーズ、およびエンティティが収集されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-122">It uses the [Translate Text API][translate-text] of Azure Cognitive Services to detect the language, translate if necessary, and collect the sentiment, key phrases, and entities from the body and the title.</span></span> <span data-ttu-id="32fb0-123">それから、記事は次のキューに渡されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-123">Then it passes the article to the next queue.</span></span>

5. <span data-ttu-id="32fb0-124">キューに入れられた記事から、Detect 関数がトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-124">A detect function is triggered from the queued article.</span></span> <span data-ttu-id="32fb0-125">[Computer Vision][vision] サービスを使用して関連したイメージでオブジェクト、ランドマーク、および書き込まれたテキストが検出されて、記事が次のキューに渡されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-125">It uses the [Computer Vision][vision] service to detect objects, landmarks, and written words in the associated image, then passes the article to the next queue.</span></span>

6. <span data-ttu-id="32fb0-126">キューに入れられた記事から、Face 関数がトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-126">A face function is triggered is triggered from the queued article.</span></span> <span data-ttu-id="32fb0-127">[Azure Face API][face] サービスを使用して関連したイメージで性別と年齢について顔の検出が行われて、記事が次のキューに渡されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-127">It uses the [Azure Face API][face] service to detect faces for gender and age in the associated image, then passes the article to the next queue.</span></span>

7. <span data-ttu-id="32fb0-128">すべての関数が完了すると、Notify 関数がトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-128">When all functions are complete, the notify function is triggered.</span></span> <span data-ttu-id="32fb0-129">これは、記事の処理されたレコードを読み込んで、必要な結果がないかスキャンします。</span><span class="sxs-lookup"><span data-stu-id="32fb0-129">It loads the processed records for the article and scans them for any results you want.</span></span> <span data-ttu-id="32fb0-130">見つかるとコンテンツにフラグが設定され、選択したシステムに通知が送信されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-130">If found, the content is flagged and a notification is sent to the system of your choice.</span></span>

<span data-ttu-id="32fb0-131">この関数は、それぞれの処理手順で Azure Cosmos DB に結果を書き込みます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-131">At each processing step, the function writes the results to Azure Cosmos DB.</span></span> <span data-ttu-id="32fb0-132">最終的に、希望どおりにデータを使用することができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-132">Ultimately, the data can be used as desired.</span></span> <span data-ttu-id="32fb0-133">たとえば、それを使用して、ビジネス プロセスを強化したり、新しい顧客を見つけたり、顧客満足度の問題を識別したりすることができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-133">For example, you can use it to enhance business processes, locate new customers, or identify customer satisfaction issues.</span></span>

### <a name="components"></a><span data-ttu-id="32fb0-134">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="32fb0-134">Components</span></span>

<span data-ttu-id="32fb0-135">この例では、次の Azure コンポーネントの一覧が使用されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-135">The following list of Azure components is used in this example.</span></span>

* <span data-ttu-id="32fb0-136">[Azure Storage][storage] は、記事に関連付けられた生のイメージ ファイルとビデオ ファイルを保持するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-136">[Azure Storage][storage] is used to hold raw image and video files associated with an article.</span></span> <span data-ttu-id="32fb0-137">Azure App Service によりセカンダリ ストレージ アカウントが作成され、Azure Function のコードとログをホストするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-137">A secondary storage account is created with Azure App Service and is used to host the Azure Function code and logs.</span></span>

* <span data-ttu-id="32fb0-138">[Azure Cosmos DB][cosmos-db] は、記事のテキスト、イメージ、およびビデオの追跡情報を保持します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-138">[Azure Cosmos DB][cosmos-db] holds article text, image, and video tracking information.</span></span> <span data-ttu-id="32fb0-139">Cognitive Services の手順の結果もここに格納されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-139">The results of the Cognitive Services steps are also stored here.</span></span>

* <span data-ttu-id="32fb0-140">[Azure Functions][functions] は、キュー メッセージに応答して着信コンテンツを変換するために使用される関数コードを実行します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-140">[Azure Functions][functions] executes the function code used to respond to queue messages and transform the incoming content.</span></span> <span data-ttu-id="32fb0-141">[Azure App Service][aas] は、関数コードをホストして、レコードを順次処理します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-141">[Azure App Service][aas] hosts the function code and processes the records serially.</span></span> <span data-ttu-id="32fb0-142">このシナリオには、Ingest、Transform、Detect Object、Face、および Notify という 5 つの関数が含まれています。</span><span class="sxs-lookup"><span data-stu-id="32fb0-142">This scenario includes five functions: Ingest, Transform, Detect Object, Face, and Notify.</span></span>

* <span data-ttu-id="32fb0-143">[Azure Service Bus][service-bus] は、関数によって使用される Azure Service Bus キューをホストします。</span><span class="sxs-lookup"><span data-stu-id="32fb0-143">[Azure Service Bus][service-bus] hosts the Azure Service Bus queues used by the functions.</span></span>

* <span data-ttu-id="32fb0-144">[Azure Cognitive Services][acs] は、[Computer Vision][vision] サービス、[Face API][face]、および [Translate Text][translate-text] 機械翻訳サービスの実装に基づくパイプラインの AI を提供します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-144">[Azure Cognitive Services][acs] provides the AI for the pipeline based on implementations of the [Computer Vision][vision] service, [Face API][face], and [Translate Text][translate-text] machine translation service.</span></span>

* <span data-ttu-id="32fb0-145">[Azure Application Insights][aai] は、問題を診断してアプリケーションの機能を理解するのに役立つ分析を提供します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-145">[Azure Application Insights][aai] provides analytics to help you diagnose issues and to understand functionality of your application.</span></span>

### <a name="alternatives"></a><span data-ttu-id="32fb0-146">代替手段</span><span class="sxs-lookup"><span data-stu-id="32fb0-146">Alternatives</span></span>

* <span data-ttu-id="32fb0-147">キューの通知と Azure Functions に基づくパターンを使用する代わりには、このデータ フローに別のパターンを使用します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-147">Instead of using a pattern based on queue notification and Azure Functions, use another pattern for this data flow.</span></span> <span data-ttu-id="32fb0-148">たとえば、この例で行われたシリアル処理とは対照的に、[Azure Service Bus トピック][topics]を使用して、記事のさまざまな部分を並行して処理することができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-148">For example, [Azure Service Bus Topics][topics] can be used to processes the various parts of the article in parallel as opposed to the serial processing done in this example.</span></span> <span data-ttu-id="32fb0-149">詳細については、[キューとトピック][queues-topics]を比較してください。</span><span class="sxs-lookup"><span data-stu-id="32fb0-149">For more information, compare [queues and topics][queues-topics].</span></span>

* <span data-ttu-id="32fb0-150">[Azure Logic App][logic-app] を使用して、関数コードを実装し、[Redlock][redlock] などのレコード レベルのロック (Azure Cosmos DB で[部分ドキュメントの更新][partial]がサポートされるまでは、並行処理で必要) を実装します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-150">Use [Azure Logic App][logic-app] to implement the function code and implement record-level locking such as [Redlock][redlock] (needed for parallel processing until Azure Cosmos DB supports [partial document updates][partial]).</span></span> <span data-ttu-id="32fb0-151">詳細については、「[Functions と Logic Apps の比較][compare]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="32fb0-151">For more information, [compare Functions and Logic Apps][compare].</span></span>

* <span data-ttu-id="32fb0-152">既存の Azure サービスではなく、カスタマイズされた AI コンポーネントを使用して、このアーキテクチャを実装します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-152">Implement this architecture using customized AI components rather than existing Azure services.</span></span> <span data-ttu-id="32fb0-153">たとえば、この例で収集される一般的な人々の数、性別、および年齢のデータとは対照的に、イメージで特定の人々を検出するカスタマイズしたモデルを使用してパイプラインを拡張します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-153">For example, extend the pipeline using a customized model that detects certain people in an image as opposed to the generic people count, gender, and age data collected in this example.</span></span> <span data-ttu-id="32fb0-154">このアーキテクチャで、カスタマイズされた機械学習や AI のモデルを使用するには、Azure Functions から呼び出せるように RESTful エンドポイントとしてモデルを構築します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-154">To use customized machine learning or AI models with this architecture, build the models as RESTful endpoints so they can be called from Azure Functions.</span></span>

* <span data-ttu-id="32fb0-155">RSS フィードではなく別の入力メカニズムを使用します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-155">Use a different input mechanism instead of RSS feeds.</span></span> <span data-ttu-id="32fb0-156">複数のジェネレーターまたはインジェスト プロセスを使用して Azure Cosmos DB と Azure Storage へのデータ フィードを行います。</span><span class="sxs-lookup"><span data-stu-id="32fb0-156">Use multiple generators or ingestion processes to feed Azure Cosmos DB and Azure Storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="32fb0-157">考慮事項</span><span class="sxs-lookup"><span data-stu-id="32fb0-157">Considerations</span></span>

<span data-ttu-id="32fb0-158">簡単にするために、このサンプル シナリオでは、Azure Cognitive Services で使用可能な API およびサービスの一部のみを使用しています。</span><span class="sxs-lookup"><span data-stu-id="32fb0-158">For simplicity, this example scenario uses only a few of the available APIs and services from Azure Cognitive Services.</span></span> <span data-ttu-id="32fb0-159">たとえば、イメージのテキストは、[Text Analytics API][text-analytics] を使用して分析できます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-159">For example, text in images can be analyzed using the [Text Analytics API][text-analytics].</span></span> <span data-ttu-id="32fb0-160">このシナリオのターゲット言語は、英語と想定されていますが、[サポートされている言語][language]であれば、ユーザーが選択したどの言語にも入力を変更することができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-160">The target language in this scenario is assumed to be English, but you can change the input to any [supported language][language] of your choice.</span></span>

### <a name="scalability"></a><span data-ttu-id="32fb0-161">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="32fb0-161">Scalability</span></span>

<span data-ttu-id="32fb0-162">Azure Functions のスケーリングは、使用されている [ホスティング プラン][plan]によって異なります。</span><span class="sxs-lookup"><span data-stu-id="32fb0-162">Azure Functions scaling depends on the [hosting plan][plan] you use.</span></span> <span data-ttu-id="32fb0-163">このソリューションでは、必要に応じてコンピューティング能力が自動的に関数に割り振られる[従量課金プラン][plan-c]が想定されています。</span><span class="sxs-lookup"><span data-stu-id="32fb0-163">This solution assumes a [Consumption plan][plan-c], in which compute power is automatically allocated to the functions when required.</span></span> <span data-ttu-id="32fb0-164">課金されるのは、関数が実行されているときのみになります。</span><span class="sxs-lookup"><span data-stu-id="32fb0-164">You pay only when your functions are running.</span></span> <span data-ttu-id="32fb0-165">[Azure App Service][plan-aas] プランを使用することも選択できます。このプランでは、複数のレベル間でスケーリングを行って、さまざまなリソース量を割り振ることができます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-165">Another option is to use an [Azure App Service][plan-aas] plan, which allows you to scale between tiers to allocate a different amount of resources.</span></span>

<span data-ttu-id="32fb0-166">Azure Cosmos DB の場合、鍵は、十分な数の[パーティション キー][keys]にワークロードをほぼ均等に分散することです。</span><span class="sxs-lookup"><span data-stu-id="32fb0-166">With Azure Cosmos DB, the key is to distribute your workload roughly evenly among a sufficiently large number of [partition keys][keys].</span></span> <span data-ttu-id="32fb0-167">コンテナーが格納できる合計データ量やコンテナーがサポートできる[スループット][throughput]の合計量には制限はありません。</span><span class="sxs-lookup"><span data-stu-id="32fb0-167">There's no limit to the total amount of data that a container can store or to the total amount of [throughput][throughput] that a container can support.</span></span>

### <a name="management-and-logging"></a><span data-ttu-id="32fb0-168">管理とログ記録</span><span class="sxs-lookup"><span data-stu-id="32fb0-168">Management and logging</span></span>

<span data-ttu-id="32fb0-169">このソリューションは、[Application Insights][aai] を使用してパフォーマンスとログ記録の情報を収集します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-169">This solution uses [Application Insights][aai] to collect performance and logging information.</span></span> <span data-ttu-id="32fb0-170">デプロイにより、このデプロイに必要なその他のサービスと同じリソース グループ内に Application Insights のインスタンスが作成されます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-170">An instance of Application Insights is created with the deployment in the same resource group as the other services needed for this deployment.</span></span>

<span data-ttu-id="32fb0-171">ソリューションによって生成されたログを表示するには、以下の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-171">To view the logs generated by the solution:</span></span>

1. <span data-ttu-id="32fb0-172">[Azure portal][portal] に移動し、デプロイのために作成されたリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-172">Go to [Azure portal][portal] and navigate to the resource group created for the deployment.</span></span>

2. <span data-ttu-id="32fb0-173">**Application Insights** インスタンスをクリックします。</span><span class="sxs-lookup"><span data-stu-id="32fb0-173">Click the **Application Insights** instance.</span></span>

3. <span data-ttu-id="32fb0-174">**[Application Insights]** セクションから **[調査\\検索]** に移動して、データを検索します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-174">From the **Application Insights** section, navigate to **Investigate\\Search** and search the data.</span></span>

### <a name="security"></a><span data-ttu-id="32fb0-175">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="32fb0-175">Security</span></span>

<span data-ttu-id="32fb0-176">Azure Cosmos DB は、Microsoft で提供されている C\# SDK を使用して、セキュリティで保護された接続と共有アクセス署名を使用します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-176">Azure Cosmos DB uses a secured connection and shared access signature through the C\# SDK provided by Microsoft.</span></span> <span data-ttu-id="32fb0-177">外部に面している攻撃対象は他にはありません。</span><span class="sxs-lookup"><span data-stu-id="32fb0-177">There are no other externally facing surface areas.</span></span> <span data-ttu-id="32fb0-178">Azure Cosmos DB のセキュリティの[ベスト プラクティス][db-practices]について確認します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-178">Learn more about security [best practices][db-practices] for Azure Cosmos DB.</span></span>

## <a name="pricing"></a><span data-ttu-id="32fb0-179">価格</span><span class="sxs-lookup"><span data-stu-id="32fb0-179">Pricing</span></span>

<span data-ttu-id="32fb0-180">このデプロイを使用可能にしておくための一日の概算コストは、システム内でデータの移動がない場合、約 \$20 です。</span><span class="sxs-lookup"><span data-stu-id="32fb0-180">The estimated daily cost to keep this deployment available is approximately \$20 U.S. with no data moving through the system.</span></span>

<span data-ttu-id="32fb0-181">Azure Cosmos DB はパワフルですが、このデプロイで最も高い[コスト][db-cost]が発生します。</span><span class="sxs-lookup"><span data-stu-id="32fb0-181">Azure Cosmos DB is powerful but incurs the greatest [cost][db-cost] in this deployment.</span></span> <span data-ttu-id="32fb0-182">提供されている Azure Functions コードをリファクタリングすることにより、別のストレージ ソリューションを使用できます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-182">You can use another storage solution by refactoring the Azure Functions code provided.</span></span>

<span data-ttu-id="32fb0-183">Azure Functions の価格は、実行される[プラン][function-plan]によって異なります。</span><span class="sxs-lookup"><span data-stu-id="32fb0-183">Pricing for Azure Functions varies depending on the [plan][function-plan] it runs in.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="32fb0-184">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="32fb0-184">Deploy the scenario</span></span>

> [!NOTE]
> <span data-ttu-id="32fb0-185">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="32fb0-185">You must have an existing Azure account.</span></span> <span data-ttu-id="32fb0-186">Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][free]を作成してください。</span><span class="sxs-lookup"><span data-stu-id="32fb0-186">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="32fb0-187">このシナリオのすべてのコードは、[GitHub][github] リポジトリから入手できます。</span><span class="sxs-lookup"><span data-stu-id="32fb0-187">All the code for this scenario is available in the [GitHub][github] repository.</span></span> <span data-ttu-id="32fb0-188">このリポジトリには、このデモのパイプラインにデータをフィードするジェネレーター アプリケーションの構築に使用されるソース コードが含まれています。</span><span class="sxs-lookup"><span data-stu-id="32fb0-188">This repository contains the source code used to build the generator application that feeds the pipeline for this demo.</span></span>

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
