---
title: Azure での保険金請求イメージの分類
description: ご使用の Azure アプリケーションに画像処理を組み込みます。
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 9640f8b5454891ed00f669bada9f7c9c69b89734
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610534"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="fa002-103">Azure での保険金請求イメージの分類</span><span class="sxs-lookup"><span data-stu-id="fa002-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="fa002-104">このシナリオは、イメージを処理する必要があるビジネスに関連があります。</span><span class="sxs-lookup"><span data-stu-id="fa002-104">This scenario is relevant for businesses that need to process images.</span></span>

<span data-ttu-id="fa002-105">応用の可能性としては、ファッション Web サイトのイメージの分類、保険金請求のテキストおよびイメージの分析、ゲームのスクリーンショットからの利用統計情報の把握などが挙げられます。</span><span class="sxs-lookup"><span data-stu-id="fa002-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="fa002-106">従来、企業では、機械学習モデルで専門知識を開発したうえで、そのモデルをトレーニングし、最終的にカスタム プロセスによってイメージを実行し、イメージからデータを取得していました。</span><span class="sxs-lookup"><span data-stu-id="fa002-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="fa002-107">Computer Vision API、Azure Functions などの Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減できるほか、既に Microsoft で開発済みの、Cognitive Services でのイメージ処理に関する専門知識を活用することができます。</span><span class="sxs-lookup"><span data-stu-id="fa002-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive Services.</span></span> <span data-ttu-id="fa002-108">この例では、特にイメージ処理ユース ケースを扱っています。</span><span class="sxs-lookup"><span data-stu-id="fa002-108">This example scenario specifically addresses an image-processing use case.</span></span> <span data-ttu-id="fa002-109">別の AI ニーズがある場合は、一連の [Cognitive Services](/azure/#pivot=products&panel=ai) について検討してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-109">If you have different AI needs, consider the full suite of [Cognitive Services](/azure/#pivot=products&panel=ai).</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="fa002-110">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="fa002-110">Relevant use cases</span></span>

<span data-ttu-id="fa002-111">その他の関連するユース ケース:</span><span class="sxs-lookup"><span data-stu-id="fa002-111">Other relevant use cases include:</span></span>

* <span data-ttu-id="fa002-112">ファッション Web サイトの画像の分類。</span><span class="sxs-lookup"><span data-stu-id="fa002-112">Classifying images on a fashion website.</span></span>
* <span data-ttu-id="fa002-113">ゲームのスクリーンショットの利用統計情報の分類。</span><span class="sxs-lookup"><span data-stu-id="fa002-113">Classifying telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="fa002-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="fa002-114">Architecture</span></span>

![イメージ分類のアーキテクチャ][architecture]

<span data-ttu-id="fa002-116">このシナリオでは、Web またはモバイル アプリケーションのバックエンド コンポーネントに対応できます。</span><span class="sxs-lookup"><span data-stu-id="fa002-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="fa002-117">シナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="fa002-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="fa002-118">API レイヤーは、Azure Functions を使用して構築されています。</span><span class="sxs-lookup"><span data-stu-id="fa002-118">The API layer is built using Azure Functions.</span></span> <span data-ttu-id="fa002-119">これらの API により、アプリケーションでイメージをアップロードし、Cosmos DB からデータを取得できます。</span><span class="sxs-lookup"><span data-stu-id="fa002-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>
2. <span data-ttu-id="fa002-120">API 呼び出しでアップロードされたイメージが Blob Storage に格納されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>
3. <span data-ttu-id="fa002-121">新しいファイルを Blob Storage に追加すると、EventGrid 通知がトリガーされ、Azure 関数に送信されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-121">Adding new files to Blob storage triggers an Event Grid notification to be sent to an Azure Function.</span></span>
4. <span data-ttu-id="fa002-122">Azure Functions により、新しくアップロードされたファイルへのリンクが、分析のために Computer Vision API に送信されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>
5. <span data-ttu-id="fa002-123">Computer Vision API からデータが返されたら、Azure Functions によって、Cosmos DB のエントリに分析の結果とイメージ メタデータが保持されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis along with the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="fa002-124">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="fa002-124">Components</span></span>

* <span data-ttu-id="fa002-125">[Computer Vision API](/azure/cognitive-services/computer-vision/home) は、Cognitive Services スイートに含まれ、各イメージに関する情報の取得に使用されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-125">[Computer Vision API](/azure/cognitive-services/computer-vision/home) is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>
* <span data-ttu-id="fa002-126">[Azure Functions](/azure/azure-functions/functions-overview) は、Web アプリケーションにバックエンド API を、また、アップロードされたイメージにイベント処理を提供します。</span><span class="sxs-lookup"><span data-stu-id="fa002-126">[Azure Functions](/azure/azure-functions/functions-overview) provides the back-end API for the web application, as well as the event processing for uploaded images.</span></span>
* <span data-ttu-id="fa002-127">[Event Grid](/azure/event-grid/overview) は、新しいイメージが Blob Storage にアップロードされたときに、イベントをトリガーします。</span><span class="sxs-lookup"><span data-stu-id="fa002-127">[Event Grid](/azure/event-grid/overview) triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="fa002-128">その後、イメージは Azure 関数で処理されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-128">The image is then processed with Azure functions.</span></span>
* <span data-ttu-id="fa002-129">[Blob Storage](/azure/storage/blobs/storage-blobs-introduction) には、Web アプリケーションにアップロードされたすべてのイメージ ファイルと、Web アプリケーションによって使用される任意の静的ファイルが格納されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-129">[Blob storage](/azure/storage/blobs/storage-blobs-introduction) stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>
* <span data-ttu-id="fa002-130">[Cosmos DB](/azure/cosmos-db/introduction) には、Computer Vision API からの処理結果を含め、アップロードされた各イメージに関するメタデータが格納されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-130">[Cosmos DB](/azure/cosmos-db/introduction) stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="fa002-131">代替手段</span><span class="sxs-lookup"><span data-stu-id="fa002-131">Alternatives</span></span>

* <span data-ttu-id="fa002-132">[Custom Vision Service](/azure/cognitive-services/custom-vision-service/home)。</span><span class="sxs-lookup"><span data-stu-id="fa002-132">[Custom Vision Service](/azure/cognitive-services/custom-vision-service/home).</span></span> <span data-ttu-id="fa002-133">Computer Vision API は、一連の[分類ベースのカテゴリ][cv-categories]を返します。</span><span class="sxs-lookup"><span data-stu-id="fa002-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="fa002-134">Computer Vision API によって返されていない情報を処理する必要がある場合は、Custom Vision Service を検討してください。これにより、カスタム イメージ分類子を構築できます。</span><span class="sxs-lookup"><span data-stu-id="fa002-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>
* <span data-ttu-id="fa002-135">[Azure Search](/azure/search/search-what-is-azure-search)。</span><span class="sxs-lookup"><span data-stu-id="fa002-135">[Azure Search](/azure/search/search-what-is-azure-search).</span></span> <span data-ttu-id="fa002-136">特定の条件を満たすイメージを検索するために、ご自身のユース ケースでメタデータにクエリを実行する必要がある場合は、Azure Search を検討してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="fa002-137">現在、プレビュー中の[コグニティブ検索](/azure/search/cognitive-search-concept-intro)では、このワークフローがシームレスに統合されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-137">Currently in preview, [Cognitive search](/azure/search/cognitive-search-concept-intro) seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="fa002-138">考慮事項</span><span class="sxs-lookup"><span data-stu-id="fa002-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="fa002-139">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="fa002-139">Scalability</span></span>

<span data-ttu-id="fa002-140">このシナリオの例で使用されるコンポーネントの大半が、自動スケーリングされるマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="fa002-140">The majority of the components used in this example scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="fa002-141">注目すべき例外がいくつかあります。Azure Functions のインスタンス数は最大 200 個に制限されています。</span><span class="sxs-lookup"><span data-stu-id="fa002-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="fa002-142">この限界を超えてスケーリングする場合は、複数のリージョンまたはアプリ プランを検討してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-142">If you need to scale beyond this limit, consider multiple regions or app plans.</span></span>

<span data-ttu-id="fa002-143">Cosmos DB は、プロビジョニング済み要求ユニット (RU) の点からいうと、自動スケーリングされません。</span><span class="sxs-lookup"><span data-stu-id="fa002-143">Cosmos DB doesn’t autoscale in terms of provisioned request units (RUs).</span></span> <span data-ttu-id="fa002-144">ご自身の要件の推定に関するガイダンスについては、Microsoft ドキュメントの[要求ユニット](/azure/cosmos-db/request-units)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="fa002-144">For guidance on estimating your requirements see [request units](/azure/cosmos-db/request-units) in our documentation.</span></span> <span data-ttu-id="fa002-145">Cosmos DB でスケーリングのメリットを十分に活用するには、Cosmos DB での[パーティション キー](/azure/cosmos-db/partition-data)のしくみを理解してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-145">To fully take advantage of the scaling in Cosmos DB, understand how [partition keys](/azure/cosmos-db/partition-data) work in CosmosDB.</span></span>

<span data-ttu-id="fa002-146">NoSQL データベースでは、可用性、スケーラビリティ、および分断性と引き換えに、一貫性 (CAP 定理という意味で) が犠牲になることがよくあります。</span><span class="sxs-lookup"><span data-stu-id="fa002-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability, and partitioning.</span></span> <span data-ttu-id="fa002-147">ただし、このシナリオの例では、キーと値のデータ モデルが使用され、ほとんどの操作が本質的にアトミックであるため、トランザクションの一貫性が必要になることはほとんどありません。</span><span class="sxs-lookup"><span data-stu-id="fa002-147">In this example scenario, a key-value data model is used and transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="fa002-148">追加のガイダンスについては、Azure アーキテクチャ センターの「[適切なデータ ストアの選択](../../guide/technology-choices/data-store-overview.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the Azure Architecture Center.</span></span> <span data-ttu-id="fa002-149">実装で高い一貫性が必要な場合は、Cosmos DB で[整合性レベルを選択する](/azure/cosmos-db/consistency-levels)ことができます。</span><span class="sxs-lookup"><span data-stu-id="fa002-149">If your implementation requires high consistency, you can [choose your consistency level](/azure/cosmos-db/consistency-levels) in CosmosDB.</span></span>

<span data-ttu-id="fa002-150">スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="fa002-151">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="fa002-151">Security</span></span>

<span data-ttu-id="fa002-152">[Azure リソース用のマネージド ID][msi] は、自分のアカウントの内部にある他のリソースへのアクセスを提供するために使用された後、Azure Functions に割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="fa002-152">[Managed identities for Azure resources][msi] are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="fa002-153">これらの ID の必要なリソースへのアクセスのみを許可して、余分なものがお使いの関数 (およびご自身の顧客) に公開されないようにします。</span><span class="sxs-lookup"><span data-stu-id="fa002-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>

<span data-ttu-id="fa002-154">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="fa002-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="fa002-155">回復性</span><span class="sxs-lookup"><span data-stu-id="fa002-155">Resiliency</span></span>

<span data-ttu-id="fa002-156">このシナリオのすべてのコンポーネントが管理されているため、すべてについて、リージョン レベルの回復性が自動的に確保されます。</span><span class="sxs-lookup"><span data-stu-id="fa002-156">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="fa002-157">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="fa002-158">価格</span><span class="sxs-lookup"><span data-stu-id="fa002-158">Pricing</span></span>

<span data-ttu-id="fa002-159">このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="fa002-159">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="fa002-160">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="fa002-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="fa002-161">トラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています (すべてのイメージのサイズが 100 KB であると想定しています)。</span><span class="sxs-lookup"><span data-stu-id="fa002-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100 kb in size):</span></span>

* <span data-ttu-id="fa002-162">[Small][small-pricing]: この価格例は、1 か月あたり &lt;5,000 件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="fa002-162">[Small][small-pricing]: this pricing example correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="fa002-163">[Medium][medium-pricing]: この価格例は、1 か月あたり 500,000 件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="fa002-163">[Medium][medium-pricing]: this pricing example correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="fa002-164">[Large][large-pricing]: この価格例は、1 か月あたり 50,000,000 件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="fa002-164">[Large][large-pricing]: this pricing example correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="fa002-165">関連リソース</span><span class="sxs-lookup"><span data-stu-id="fa002-165">Related resources</span></span>

<span data-ttu-id="fa002-166">ガイド付きラーニング パスについては、「[Azure でサーバーレス Web アプリを作成する][serverless]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fa002-166">For a guided learning path, see [Build a serverless web app in Azure][serverless].</span></span>

<span data-ttu-id="fa002-167">このシナリオの例を運用環境にデプロイする前に、[Azure Functions のパフォーマンスと信頼性を最適化する][functions-best-practices]ための推奨プラクティスをご確認ください。</span><span class="sxs-lookup"><span data-stu-id="fa002-167">Before deploying this example scenario in a production environment, review recommended practices for [optimizing the performance and reliability of Azure Functions][functions-best-practices].</span></span>

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
