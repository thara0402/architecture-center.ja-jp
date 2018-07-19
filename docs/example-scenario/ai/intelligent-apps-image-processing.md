---
title: インテリジェントなアプリ - Azure でのイメージ処理
description: イメージ処理を Azure アプリケーションに組み込む実証済みのソリューション。
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: c5bfb9a929ddddda4336e1cbc8665a0b4d3bbe2c
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891333"
---
# <a name="insurance-claim-image-classification-on-azure"></a><span data-ttu-id="699cd-103">Azure での保険金請求イメージの分類</span><span class="sxs-lookup"><span data-stu-id="699cd-103">Insurance claim image classification on Azure</span></span>

<span data-ttu-id="699cd-104">このシナリオの例は、イメージを処理する必要があるビジネスに適用できます。</span><span class="sxs-lookup"><span data-stu-id="699cd-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="699cd-105">応用の可能性としては、ファッション Web サイトのイメージの分類、保険金請求のテキストおよびイメージの分析、ゲームのスクリーンショットからの利用統計情報の把握などが挙げられます。</span><span class="sxs-lookup"><span data-stu-id="699cd-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="699cd-106">従来、企業では、機械学習モデルで専門知識を開発したうえで、そのモデルをトレーニングし、最終的にカスタム プロセスによってイメージを実行し、イメージからデータを取得していました。</span><span class="sxs-lookup"><span data-stu-id="699cd-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="699cd-107">Computer Vision API、Azure Functions などの Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減できるほか、既に Microsoft で開発済みの、Cognitive Services でのイメージ処理に関する専門知識を活用することができます。</span><span class="sxs-lookup"><span data-stu-id="699cd-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="699cd-108">このシナリオでは、特にイメージ処理を扱っています。</span><span class="sxs-lookup"><span data-stu-id="699cd-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="699cd-109">別の AI ニーズがある場合は、一連の [Cognitive Services][cognitive-docs] について検討してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="699cd-110">考えられるユース ケース</span><span class="sxs-lookup"><span data-stu-id="699cd-110">Potential use cases</span></span>

<span data-ttu-id="699cd-111">次のユース ケースについて、このソリューションを検討してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-111">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="699cd-112">ファッション Web サイトのイメージを分類する。</span><span class="sxs-lookup"><span data-stu-id="699cd-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="699cd-113">保険金請求のイメージを分類する</span><span class="sxs-lookup"><span data-stu-id="699cd-113">Classify images for insurance claims</span></span>
* <span data-ttu-id="699cd-114">ゲームのスクリーンショットの利用統計情報を分類する。</span><span class="sxs-lookup"><span data-stu-id="699cd-114">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="699cd-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="699cd-115">Architecture</span></span>

![インテリジェントなアプリのアーキテクチャ - Computer Vision][architecture-computer-vision]

<span data-ttu-id="699cd-117">このソリューションは、Web またはモバイル アプリケーションのバックエンド コンポーネントを対象としています。</span><span class="sxs-lookup"><span data-stu-id="699cd-117">This solution covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="699cd-118">ソリューションのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="699cd-118">Data flows through the solution as follows:</span></span>

1. <span data-ttu-id="699cd-119">Azure Functions が、API レイヤーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="699cd-119">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="699cd-120">これらの API により、アプリケーションでイメージをアップロードし、Cosmos DB からデータを取得できます。</span><span class="sxs-lookup"><span data-stu-id="699cd-120">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="699cd-121">API 呼び出しでアップロードされたイメージが Blob Storage に格納されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-121">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="699cd-122">新しいファイルを Blob Storage に追加すると、EventGrid 通知がトリガーされ、Azure 関数に送信されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-122">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="699cd-123">Azure Functions により、新しくアップロードされたファイルへのリンクが、分析のために Computer Vision API に送信されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-123">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="699cd-124">Computer Vision API からデータが返されたら、Azure Functions によって、Cosmos DB のエントリに分析の結果とイメージ メタデータが保持されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-124">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="699cd-125">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="699cd-125">Components</span></span>

* <span data-ttu-id="699cd-126">[Computer Vision API][computer-vision-docs] は、Cognitive Services スイートに含まれ、各イメージに関する情報の取得に使用されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-126">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="699cd-127">[Azure Functions][functions-docs] は、Web アプリケーションにバックエンド API を、また、アップロードされたイメージにイベント処理を提供します。</span><span class="sxs-lookup"><span data-stu-id="699cd-127">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="699cd-128">[Event Grid][eventgrid-docs] は、新しいイメージが Blob Storage にアップロードされたときに、イベントをトリガーします。</span><span class="sxs-lookup"><span data-stu-id="699cd-128">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="699cd-129">その後、イメージは Azure 関数で処理されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-129">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="699cd-130">[Blob Storage][storage-docs] には、Web アプリケーションにアップロードされたすべてのイメージ ファイルと、Web アプリケーションによって使用される任意の静的ファイルが格納されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-130">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="699cd-131">[Cosmos DB][cosmos-docs] には、Computer Vision API からの処理結果を含め、アップロードされた各イメージに関するメタデータが格納されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-131">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="699cd-132">代替手段</span><span class="sxs-lookup"><span data-stu-id="699cd-132">Alternatives</span></span>

* <span data-ttu-id="699cd-133">[Custom Vision Service][custom-vision-docs]。</span><span class="sxs-lookup"><span data-stu-id="699cd-133">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="699cd-134">Computer Vision API は、一連の[分類ベースのカテゴリ][cv-categories]を返します。</span><span class="sxs-lookup"><span data-stu-id="699cd-134">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="699cd-135">Computer Vision API によって返されていない情報を処理する必要がある場合は、Custom Vision Service を検討してください。これにより、カスタム イメージ分類子を構築できます。</span><span class="sxs-lookup"><span data-stu-id="699cd-135">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="699cd-136">[Azure Search][azure-search-docs]。</span><span class="sxs-lookup"><span data-stu-id="699cd-136">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="699cd-137">特定の条件を満たすイメージを検索するために、ご自身のユース ケースでメタデータにクエリを実行する必要がある場合は、Azure Search を検討してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-137">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="699cd-138">現在、プレビュー中の[コグニティブ検索][cognitive-search]では、このワークフローがシームレスに統合されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-138">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="699cd-139">考慮事項</span><span class="sxs-lookup"><span data-stu-id="699cd-139">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="699cd-140">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="699cd-140">Scalability</span></span>

<span data-ttu-id="699cd-141">ほとんどの場合、このソリューションのすべてのコンポーネントが、自動スケーリングされるマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="699cd-141">For the most part all of the components of this solution are managed services that will automatically scale.</span></span> <span data-ttu-id="699cd-142">注目すべき例外がいくつかあります。Azure Functions のインスタンス数は最大 200 個に制限されています。</span><span class="sxs-lookup"><span data-stu-id="699cd-142">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="699cd-143">それを超えてスケーリングする場合は、複数のリージョンまたはアプリ プランを検討してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-143">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="699cd-144">Cosmos DB は、プロビジョニング済み要求ユニット (RU) の点からいうと、自動スケーリングされません。</span><span class="sxs-lookup"><span data-stu-id="699cd-144">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="699cd-145">ご自身の要件の推定に関するガイダンスについては、Microsoft ドキュメントの[要求ユニット][request-units]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="699cd-145">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="699cd-146">Cosmos DB でスケーリングのメリットを十分に活用するには、[パーティション キー][partition-key]についても確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="699cd-146">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="699cd-147">NoSQL データベースでは、可用性、スケーラビリティ、および分断性と引き換えに、一貫性 (CAP 定理という意味で) が犠牲になることがよくあります。</span><span class="sxs-lookup"><span data-stu-id="699cd-147">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="699cd-148">ただし、このシナリオで使用されているキーと値のデータ モデルの場合、ほとんどの操作が本質的にアトミックであるため、トランザクションの一貫性が必要になることはほとんどありません。</span><span class="sxs-lookup"><span data-stu-id="699cd-148">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="699cd-149">追加のガイダンスについては、アーキテクチャ センターの「[適切なデータ ストアの選択](../../guide/technology-choices/data-store-overview.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-149">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="699cd-150">スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="699cd-151">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="699cd-151">Security</span></span>

<span data-ttu-id="699cd-152">[マネージド サービス ID][msi] (MSI) は、ご自身のアカウントの内部にあり、ご自身の Azure Functions に割り当てられている他のリソースへのアクセスを提供するときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-152">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="699cd-153">これらの ID の必要なリソースへのアクセスのみを許可して、余分なものがお使いの関数 (およびご自身の顧客) に公開されないようにします。</span><span class="sxs-lookup"><span data-stu-id="699cd-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="699cd-154">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="699cd-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="699cd-155">回復性</span><span class="sxs-lookup"><span data-stu-id="699cd-155">Resiliency</span></span>

<span data-ttu-id="699cd-156">このソリューションのすべてのコンポーネントが管理されているため、すべてについて、リージョン レベルの回復性が自動的に確保されます。</span><span class="sxs-lookup"><span data-stu-id="699cd-156">All of the components in this solution are managed, so at a regional level they are all resilient automatically.</span></span> 

<span data-ttu-id="699cd-157">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="699cd-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="699cd-158">価格</span><span class="sxs-lookup"><span data-stu-id="699cd-158">Pricing</span></span>

<span data-ttu-id="699cd-159">このソリューションの実行コストを調べるために、すべてのサービスがコスト計算ツールに事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="699cd-159">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="699cd-160">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="699cd-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="699cd-161">トラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています (すべてのイメージのサイズが 100 KB であると想定しています)。</span><span class="sxs-lookup"><span data-stu-id="699cd-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="699cd-162">[Small][pricing]: 1 か月あたり &lt;5,000 件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="699cd-162">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="699cd-163">[Medium][medium-pricing]: 1 か月あたり 50 万件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="699cd-163">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="699cd-164">[Large][large-pricing]: 1 か月あたり 5,000 万件のイメージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="699cd-164">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="699cd-165">関連リソース</span><span class="sxs-lookup"><span data-stu-id="699cd-165">Related Resources</span></span>

<span data-ttu-id="699cd-166">このソリューションのガイド付きラーニング パスについては、「[Build a serverless web app in Azure (Azure でのサーバーレス Web アプリの構築)][serverless]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="699cd-166">For a guided learning path of this solution, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="699cd-167">これを運用環境に移行する前に、Azure Functions の[ベスト プラクティス][functions-best-practices]に関するページをご確認ください。</span><span class="sxs-lookup"><span data-stu-id="699cd-167">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data