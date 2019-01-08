---
title: リアルタイムでの不正検出
titleSuffix: Azure Example Scenarios
description: Azure Event Hubs と Stream Analytics を使用して、リアルタイムで不正行為を検出します。
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: 9e4d8c5d24acc414ab38722d2df59102395250fb
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643407"
---
# <a name="real-time-fraud-detection-on-azure"></a><span data-ttu-id="2167f-103">Azure におけるリアルタイムでの不正検出</span><span class="sxs-lookup"><span data-stu-id="2167f-103">Real-time fraud detection on Azure</span></span>

<span data-ttu-id="2167f-104">このサンプル シナリオは、不正なトランザクションやその他の異常なアクティビティを検出するために、リアルタイムでデータを分析する必要がある組織に関連します。</span><span class="sxs-lookup"><span data-stu-id="2167f-104">This example scenario is relevant to organizations that need to analyze data in real time to detect fraudulent transactions or other anomalous activity.</span></span>

<span data-ttu-id="2167f-105">考えられる用途としては、不正なクレジット カード アクティビティや携帯電話の通話の特定などが挙げられます。</span><span class="sxs-lookup"><span data-stu-id="2167f-105">Potential applications include identifying fraudulent credit card activity or mobile phone calls.</span></span> <span data-ttu-id="2167f-106">従来のオンライン分析システムでは、異常なアクティビティを特定するためのデータの変換や分析に何時間もかかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="2167f-106">Traditional online analytical systems might take hours to transform and analyze the data to identify anomalous activity.</span></span>

<span data-ttu-id="2167f-107">Event Hubs、Stream Analytics などのフル マネージド Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減し、クラウド規模のデータ インジェストとリアルタイム分析で Microsoft の専門知識を活用することができます。</span><span class="sxs-lookup"><span data-stu-id="2167f-107">By using fully managed Azure services such as Event Hubs and Stream Analytics, companies can eliminate the need to manage individual servers, while reducing costs and leveraging Microsoft's expertise in cloud-scale data ingestion and real-time analytics.</span></span> <span data-ttu-id="2167f-108">このシナリオでは、特に不正行為の検出を扱っています。</span><span class="sxs-lookup"><span data-stu-id="2167f-108">This scenario specifically addresses the detection of fraudulent activity.</span></span> <span data-ttu-id="2167f-109">データ分析について他のニーズがある場合は、使用可能な [Azure Analytics サービス][product-category]の一覧を確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2167f-109">If you have other needs for data analytics, you should review the list of available [Azure Analytics services][product-category].</span></span>

<span data-ttu-id="2167f-110">このサンプルは、広範なデータ処理アーキテクチャと戦略の一部です。</span><span class="sxs-lookup"><span data-stu-id="2167f-110">This sample represents one part of a broader data processing architecture and strategy.</span></span> <span data-ttu-id="2167f-111">全体的なアーキテクチャのこの側面における別のオプションについては、この記事の後半で説明します。</span><span class="sxs-lookup"><span data-stu-id="2167f-111">Other options for this aspect of an overall architecture are discussed later in this article.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="2167f-112">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="2167f-112">Relevant use cases</span></span>

<span data-ttu-id="2167f-113">その他の関連するユース ケース:</span><span class="sxs-lookup"><span data-stu-id="2167f-113">Other relevant use cases include:</span></span>

- <span data-ttu-id="2167f-114">電気通信シナリオで不正な携帯電話呼び出しを検出する。</span><span class="sxs-lookup"><span data-stu-id="2167f-114">Detecting fraudulent mobile-phone calls in telecommunications scenarios.</span></span>
- <span data-ttu-id="2167f-115">金融機関向けに不正なクレジット カード トランザクションを特定する。</span><span class="sxs-lookup"><span data-stu-id="2167f-115">Identifying fraudulent credit card transactions for banking institutions.</span></span>
- <span data-ttu-id="2167f-116">小売または eコマースのシナリオで不正購入を特定する。</span><span class="sxs-lookup"><span data-stu-id="2167f-116">Identifying fraudulent purchases in retail or e-commerce scenarios.</span></span>

## <a name="architecture"></a><span data-ttu-id="2167f-117">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="2167f-117">Architecture</span></span>

![リアルタイム不正検出シナリオの Azure コンポーネント アーキテクチャの概要][architecture]

<span data-ttu-id="2167f-119">このシナリオでは、リアルタイム分析パイプラインのバックエンド コンポーネントに対応できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-119">This scenario covers the back-end components of a real-time analytics pipeline.</span></span> <span data-ttu-id="2167f-120">シナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="2167f-120">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="2167f-121">携帯電話呼び出しメタデータが、ソース システムから Azure Event Hubs インスタンスに送信されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-121">Mobile phone call metadata is sent from the source system to an Azure Event Hubs instance.</span></span>
2. <span data-ttu-id="2167f-122">Stream Analytics ジョブが開始され、イベント ハブ ソースを介してデータが受信されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-122">A Stream Analytics job is started, which receives data via the event hub source.</span></span>
3. <span data-ttu-id="2167f-123">Stream Analytics ジョブによって定義済みクエリが実行されます。これにより、入力ストリームが変換され、不正なトランザクションのアルゴリズムに基づいて分析されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-123">The Stream Analytics job runs a predefined query to transform the input stream and analyze it based on a fraudulent-transaction algorithm.</span></span> <span data-ttu-id="2167f-124">このクエリでは、タンブリング ウィンドウを使って、ストリームが個別のテンポラル ユニットにセグメント化されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-124">This query uses a tumbling window to segment the stream into distinct temporal units.</span></span>
4. <span data-ttu-id="2167f-125">Stream Analytics ジョブにより、検出された不正な呼び出しを表す変換済みストリームが、Azure Blob Storage の出力シンクに書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="2167f-125">The Stream Analytics job writes the transformed stream representing detected fraudulent calls to an output sink in Azure Blob storage.</span></span>

### <a name="components"></a><span data-ttu-id="2167f-126">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="2167f-126">Components</span></span>

- <span data-ttu-id="2167f-127">[Azure Event Hubs][docs-event-hubs] はリアルタイム ストリーミング プラットフォームであり、毎秒数百万のイベントを受け取って処理できるイベント インジェスト サービスです。</span><span class="sxs-lookup"><span data-stu-id="2167f-127">[Azure Event Hubs][docs-event-hubs] is a real-time streaming platform and event ingestion service, capable of receiving and processing millions of events per second.</span></span> <span data-ttu-id="2167f-128">Event Hubs では、分散されたソフトウェアやデバイスから生成されるイベント、データ、またはテレメトリを処理および格納できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-128">Event Hubs can process and store events, data, or telemetry produced by distributed software and devices.</span></span> <span data-ttu-id="2167f-129">このシナリオでは、Event Hubs が、すべての電話呼び出しメタデータを受け取り、不正行為に関する分析を実行します。</span><span class="sxs-lookup"><span data-stu-id="2167f-129">In this scenario, Event Hubs receives all phone call metadata to be analyzed for fraudulent activity.</span></span>
- <span data-ttu-id="2167f-130">[Azure Stream Analytics][docs-stream-analytics] は、デバイスおよび他のデータ ソースからの大量のデータ ストリームを分析できるイベント処理エンジンです。</span><span class="sxs-lookup"><span data-stu-id="2167f-130">[Azure Stream Analytics][docs-stream-analytics] is an event-processing engine that can analyze high volumes of data streaming from devices and other data sources.</span></span> <span data-ttu-id="2167f-131">また、データ ストリームから情報を抽出し、パターンやリレーションシップを特定することもできます。</span><span class="sxs-lookup"><span data-stu-id="2167f-131">It also supports extracting information from data streams to identify patterns and relationships.</span></span> <span data-ttu-id="2167f-132">これらのパターンでは、その他のダウンストリーム アクションをトリガーできます。</span><span class="sxs-lookup"><span data-stu-id="2167f-132">These patterns can trigger other downstream actions.</span></span> <span data-ttu-id="2167f-133">このシナリオでは、Stream Analytics によって、Event Hubs からの入力ストリームが変換され、不正な呼び出しが特定されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-133">In this scenario, Stream Analytics transforms the input stream from Event Hubs to identify fraudulent calls.</span></span>
- <span data-ttu-id="2167f-134">このシナリオでは、[BLOB ストレージ](/azure/storage/blobs/storage-blobs-introduction)を使って、Stream Analytics ジョブの結果が格納されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-134">[Blob storage](/azure/storage/blobs/storage-blobs-introduction) is used in this scenario to store the results of the Stream Analytics job.</span></span>

## <a name="considerations"></a><span data-ttu-id="2167f-135">考慮事項</span><span class="sxs-lookup"><span data-stu-id="2167f-135">Considerations</span></span>

### <a name="alternatives"></a><span data-ttu-id="2167f-136">代替手段</span><span class="sxs-lookup"><span data-stu-id="2167f-136">Alternatives</span></span>

<span data-ttu-id="2167f-137">リアルタイム メッセージ インジェスト、データ ストレージ、ストリーム処理、分析データのストレージ、および分析とレポート作成では、使用できるテクノロジが多数あります。</span><span class="sxs-lookup"><span data-stu-id="2167f-137">Many technology choices are available for real-time message ingestion, data storage, stream processing, storage of analytical data, and analytics and reporting.</span></span> <span data-ttu-id="2167f-138">これらのオプションやその機能、主要な選択条件の概要については、Azure データ アーキテクチャ ガイドの「[ビッグ データ アーキテクチャ: リアルタイム処理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-138">For an overview of these options, their capabilities, and key selection criteria, see [Big data architectures: Real-time processing](/azure/architecture/data-guide/technology-choices/real-time-ingestion) in the Azure Data Architecture Guide.</span></span>

<span data-ttu-id="2167f-139">また、より複雑な不正検出アルゴリズムを、Azure のさまざまな機械学習サービスで生成することもできます。</span><span class="sxs-lookup"><span data-stu-id="2167f-139">Additionally, more complex algorithms for fraud detection can be produced by various machine learning services in Azure.</span></span> <span data-ttu-id="2167f-140">これらのオプションの概要については、「[Azure データ アーキテクチャ ガイド](../../data-guide/index.md)」の[機械学習のテクノロジの選択](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-140">For an overview of these options, see [Technology choices for machine learning](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) in the [Azure Data Architecture Guide](../../data-guide/index.md).</span></span>

### <a name="availability"></a><span data-ttu-id="2167f-141">可用性</span><span class="sxs-lookup"><span data-stu-id="2167f-141">Availability</span></span>

<span data-ttu-id="2167f-142">Azure Monitor には、さまざまな Azure サービスにわたって監視するための統合ユーザー インターフェイスが用意されています。</span><span class="sxs-lookup"><span data-stu-id="2167f-142">Azure Monitor provides unified user interfaces for monitoring across various Azure services.</span></span> <span data-ttu-id="2167f-143">詳細については、[Microsoft Azure での監視](/azure/monitoring-and-diagnostics/monitoring-overview)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-143">For more information, see [Monitoring in Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview).</span></span> <span data-ttu-id="2167f-144">Event Hubs と Stream Analytics は両方とも、Azure Monitor に統合されています。</span><span class="sxs-lookup"><span data-stu-id="2167f-144">Event Hubs and Stream Analytics are both integrated with Azure Monitor.</span></span>

<span data-ttu-id="2167f-145">他の可用性に関する考慮事項については、Azure アーキテクチャ センターの「[可用性のチェックリスト][availability]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="2167f-145">For other availability considerations, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="2167f-146">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="2167f-146">Scalability</span></span>

<span data-ttu-id="2167f-147">このシナリオのコンポーネントは、ハイパースケール インジェストと超並列リアルタイム分析を実現するように設計されています。</span><span class="sxs-lookup"><span data-stu-id="2167f-147">The components of this scenario are designed for hyper-scale ingestion and massively parallel real-time analytics.</span></span> <span data-ttu-id="2167f-148">Azure Event Hubs は高度にスケーラブルで、毎秒数百万のイベントを、短い待機時間で受け取って処理できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-148">Azure Event Hubs is highly scalable, capable of receiving and processing millions of events per second with low latency.</span></span> <span data-ttu-id="2167f-149">Event Hubs では、スループット ユニット数が、使用量のニーズに合わせて[自動的にスケールアップ](/azure/event-hubs/event-hubs-auto-inflate)できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-149">Event Hubs can [automatically scale up](/azure/event-hubs/event-hubs-auto-inflate) the number of throughput units to meet usage needs.</span></span> <span data-ttu-id="2167f-150">Azure Stream Analytics では、多くのソースからの大量のストリーミング データを分析できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-150">Azure Stream Analytics is capable of analyzing high volumes of streaming data from many sources.</span></span> <span data-ttu-id="2167f-151">Stream Analytics をスケールアップするには、ご自身のストリーミング ジョブを実行するために割り当てられている[ストリーミング ユニット](/azure/stream-analytics/stream-analytics-streaming-unit-consumption)数を増やします。</span><span class="sxs-lookup"><span data-stu-id="2167f-151">You can scale up Stream Analytics by increasing the number of [streaming units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) allocated to execute your streaming job.</span></span>

<span data-ttu-id="2167f-152">スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="2167f-152">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="2167f-153">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="2167f-153">Security</span></span>

<span data-ttu-id="2167f-154">Azure Event Hubs では、Shared Access Signature (SAS) トークンとイベント パブリッシャーの組み合わせに基づく[認証とセキュリティ モデル][docs-event-hubs-security-model]によって、データが保護されます。</span><span class="sxs-lookup"><span data-stu-id="2167f-154">Azure Event Hubs secures data through an [authentication and security model][docs-event-hubs-security-model] based on a combination of Shared Access Signature (SAS) tokens and event publishers.</span></span> <span data-ttu-id="2167f-155">イベント パブリッシャーは Event Hub の仮想エンドポイントを定義します。</span><span class="sxs-lookup"><span data-stu-id="2167f-155">An event publisher defines a virtual endpoint for an event hub.</span></span> <span data-ttu-id="2167f-156">パブリッシャーは、Event Hub にメッセージを送信するためにのみ使用できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-156">The publisher can only be used to send messages to an event hub.</span></span> <span data-ttu-id="2167f-157">パブリッシャーからメッセージを受信することはできません。</span><span class="sxs-lookup"><span data-stu-id="2167f-157">It is not possible to receive messages from a publisher.</span></span>

<span data-ttu-id="2167f-158">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-158">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="2167f-159">回復性</span><span class="sxs-lookup"><span data-stu-id="2167f-159">Resiliency</span></span>

<span data-ttu-id="2167f-160">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="2167f-160">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="2167f-161">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="2167f-161">Deploy the scenario</span></span>

<span data-ttu-id="2167f-162">このシナリオをデプロイするには、こちらの[ステップ バイ ステップのチュートリアル][tutorial]に従います。このチュートリアルでは、シナリオの各コンポーネントを手動でデプロイする方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="2167f-162">To deploy this scenario, you can follow this [step-by-step tutorial][tutorial] demonstrating how to manually deploy each component of the scenario.</span></span> <span data-ttu-id="2167f-163">また、このチュートリアルは、サンプル電話呼び出しメタデータを生成し、そのデータをイベント ハブ インスタンスに送信するための .NET クライアント アプリケーションも提供します。</span><span class="sxs-lookup"><span data-stu-id="2167f-163">This tutorial also provides a .NET client application to generate sample phone call metadata and send that data to an event hub instance.</span></span>

## <a name="pricing"></a><span data-ttu-id="2167f-164">価格</span><span class="sxs-lookup"><span data-stu-id="2167f-164">Pricing</span></span>

<span data-ttu-id="2167f-165">このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="2167f-165">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="2167f-166">特定のユース ケースについて価格の変化を確認するには、予想されるデータ ボリュームに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="2167f-166">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected data volume.</span></span>

<span data-ttu-id="2167f-167">取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="2167f-167">We have provided three sample cost profiles based on amount of traffic you expect to get:</span></span>

- <span data-ttu-id="2167f-168">[Small][small-pricing]: 1 標準ストリーミング ユニットで 1 か月あたり 100万件のイベントを処理します。</span><span class="sxs-lookup"><span data-stu-id="2167f-168">[Small][small-pricing]: process one million events through one standard streaming unit per month.</span></span>
- <span data-ttu-id="2167f-169">[Medium][medium-pricing]: 5 標準ストリーミング ユニットで 1 か月あたり 1 億件のイベントを処理します。</span><span class="sxs-lookup"><span data-stu-id="2167f-169">[Medium][medium-pricing]: process 100M events through five standard streaming units per month.</span></span>
- <span data-ttu-id="2167f-170">[Large][large-pricing]: 20 標準ストリーミング ユニットで 1 か月あたり 9 億 9,900 万件のイベントを処理します。</span><span class="sxs-lookup"><span data-stu-id="2167f-170">[Large][large-pricing]: process 999 million events through 20 standard streaming units per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="2167f-171">関連リソース</span><span class="sxs-lookup"><span data-stu-id="2167f-171">Related resources</span></span>

<span data-ttu-id="2167f-172">不正検出のシナリオがさらに複雑な場合は、機械学習モデルを活用できます。</span><span class="sxs-lookup"><span data-stu-id="2167f-172">More complex fraud detection scenarios can benefit from a machine learning model.</span></span> <span data-ttu-id="2167f-173">Machine Learning Server を使用して構築されたシナリオについては、[Machine Learning Server を使用した不正検出][r-server-fraud-detection]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-173">For scenarios built using Machine Learning Server, see [Fraud detection using Machine Learning Server][r-server-fraud-detection].</span></span> <span data-ttu-id="2167f-174">Machine Learning Server を使用した他のソリューション テンプレートについては、[データ サイエンスのシナリオとソリューション テンプレート][docs-r-server-sample-solutions]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2167f-174">For other solution templates using Machine Learning Server, see [Data science scenarios and solution templates][docs-r-server-sample-solutions].</span></span> <span data-ttu-id="2167f-175">Azure Data Lake Analytics を使用したソリューションの例については、「[Using Azure Data Lake and R for Fraud Detection (不正検出に Azure Data Lake および R を使用する)][technet-fraud-detection]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="2167f-175">For an example solution using Azure Data Lake Analytics, see [Using Azure Data Lake and R for Fraud Detection][technet-fraud-detection].</span></span>

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture]: ./media/architecture-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/
