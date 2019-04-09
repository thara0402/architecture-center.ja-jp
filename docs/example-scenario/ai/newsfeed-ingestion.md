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
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a>Azure 上での大量のニュース フィードの取り込みと分析

このサンプル シナリオでは、パブリック RSS ニュース フィードを使用したドキュメントの大量の取り込みとほぼリアルタイムの分析のパイプラインについて説明します。  ここでは、Azure Cognitive Services を使用して、テキスト翻訳、顔認識、センチメント検出などの有益な洞察を提供します。

このシナリオに含まれているのは、[英語][english]、[ロシア語][russian]、および[ドイツ語][german]のニュース フィードですが、他の RSS フィードにも容易に拡張することができます。 デプロイを容易にするために、データの収集、処理、および分析は、すべて Azure サービスに基づいて行われます。

## <a name="relevant-use-cases"></a>関連するユース ケース

このシナリオは、RSS フィードの処理に基づいて作成されていますが、次のことを実行する必要があるすべてのドキュメント、Web サイト、または記事に関係があります。

* 任意のテキストを選択した言語に翻訳する。
* デジタル コンテンツでキー フレーズ、エンティティ、およびユーザーのセンチメントを見つける。
* ディジタル記事に関連したイメージでオブジェクト、テキスト、およびランドマークを検出する。
* デジタル コンテンツに関連したすべてのイメージで性別と年齢によって人々を検出する。

## <a name="architecture"></a>アーキテクチャ

![][architecture]

このソリューションのデータ フローは次のとおりです。

1. RSS ニュース フィードは、ドキュメントや記事からデータを取得するジェネレーターとして機能します。 たとえば、記事の場合、データには通常、ニュース項目のタイトルと元の本文の概要が含まれており、時にはイメージが含まれていることもあります。

2. ジェネレーターまたはインジェスト プロセスにより、記事および関連するイメージが Azure Cosmos DB の[コレクション][collection]に挿入されます。

3. 通知により、Azure Functions で Ingest 関数がトリガーされます。この関数は、記事のテキストを CosmosDB に格納し、記事のイメージ (ある場合) を Azure Blob Storage に格納します。  それから、記事が次のキューに渡されます。

4. キュー イベントによって Translate 関数がトリガーされます。 Azure Cognitive Services の [Translate Text API][translate-text] を使用して言語が検出され、必要に応じて翻訳が行われて、本文とタイトルからセンチメント、キー フレーズ、およびエンティティが収集されます。 それから、記事は次のキューに渡されます。

5. キューに入れられた記事から、Detect 関数がトリガーされます。 [Computer Vision][vision] サービスを使用して関連したイメージでオブジェクト、ランドマーク、および書き込まれたテキストが検出されて、記事が次のキューに渡されます。

6. キューに入れられた記事から、Face 関数がトリガーされます。 [Azure Face API][face] サービスを使用して関連したイメージで性別と年齢について顔の検出が行われて、記事が次のキューに渡されます。

7. すべての関数が完了すると、Notify 関数がトリガーされます。 これは、記事の処理されたレコードを読み込んで、必要な結果がないかスキャンします。 見つかるとコンテンツにフラグが設定され、選択したシステムに通知が送信されます。

この関数は、それぞれの処理手順で Azure Cosmos DB に結果を書き込みます。 最終的に、希望どおりにデータを使用することができます。 たとえば、それを使用して、ビジネス プロセスを強化したり、新しい顧客を見つけたり、顧客満足度の問題を識別したりすることができます。

### <a name="components"></a>コンポーネント

この例では、次の Azure コンポーネントの一覧が使用されます。

* [Azure Storage][storage] は、記事に関連付けられた生のイメージ ファイルとビデオ ファイルを保持するために使用されます。 Azure App Service によりセカンダリ ストレージ アカウントが作成され、Azure Function のコードとログをホストするために使用されます。

* [Azure Cosmos DB][cosmos-db] は、記事のテキスト、イメージ、およびビデオの追跡情報を保持します。 Cognitive Services の手順の結果もここに格納されます。

* [Azure Functions][functions] は、キュー メッセージに応答して着信コンテンツを変換するために使用される関数コードを実行します。 [Azure App Service][aas] は、関数コードをホストして、レコードを順次処理します。 このシナリオには、Ingest、Transform、Detect Object、Face、および Notify という 5 つの関数が含まれています。

* [Azure Service Bus][service-bus] は、関数によって使用される Azure Service Bus キューをホストします。

* [Azure Cognitive Services][acs] は、[Computer Vision][vision] サービス、[Face API][face]、および [Translate Text][translate-text] 機械翻訳サービスの実装に基づくパイプラインの AI を提供します。

* [Azure Application Insights][aai] は、問題を診断してアプリケーションの機能を理解するのに役立つ分析を提供します。

### <a name="alternatives"></a>代替手段

* キューの通知と Azure Functions に基づくパターンを使用する代わりには、このデータ フローに別のパターンを使用します。 たとえば、この例で行われたシリアル処理とは対照的に、[Azure Service Bus トピック][topics]を使用して、記事のさまざまな部分を並行して処理することができます。 詳細については、[キューとトピック][queues-topics]を比較してください。

* [Azure Logic App][logic-app] を使用して、関数コードを実装し、[Redlock][redlock] などのレコード レベルのロック (Azure Cosmos DB で[部分ドキュメントの更新][partial]がサポートされるまでは、並行処理で必要) を実装します。 詳細については、「[Functions と Logic Apps の比較][compare]」を参照してください。

* 既存の Azure サービスではなく、カスタマイズされた AI コンポーネントを使用して、このアーキテクチャを実装します。 たとえば、この例で収集される一般的な人々の数、性別、および年齢のデータとは対照的に、イメージで特定の人々を検出するカスタマイズしたモデルを使用してパイプラインを拡張します。 このアーキテクチャで、カスタマイズされた機械学習や AI のモデルを使用するには、Azure Functions から呼び出せるように RESTful エンドポイントとしてモデルを構築します。

* RSS フィードではなく別の入力メカニズムを使用します。 複数のジェネレーターまたはインジェスト プロセスを使用して Azure Cosmos DB と Azure Storage へのデータ フィードを行います。

## <a name="considerations"></a>考慮事項

簡単にするために、このサンプル シナリオでは、Azure Cognitive Services で使用可能な API およびサービスの一部のみを使用しています。 たとえば、イメージのテキストは、[Text Analytics API][text-analytics] を使用して分析できます。 このシナリオのターゲット言語は、英語と想定されていますが、[サポートされている言語][language]であれば、ユーザーが選択したどの言語にも入力を変更することができます。

### <a name="scalability"></a>スケーラビリティ

Azure Functions のスケーリングは、使用されている [ホスティング プラン][plan]によって異なります。 このソリューションでは、必要に応じてコンピューティング能力が自動的に関数に割り振られる[従量課金プラン][plan-c]が想定されています。 課金されるのは、関数が実行されているときのみになります。 [Azure App Service][plan-aas] プランを使用することも選択できます。このプランでは、複数のレベル間でスケーリングを行って、さまざまなリソース量を割り振ることができます。

Azure Cosmos DB の場合、鍵は、十分な数の[パーティション キー][keys]にワークロードをほぼ均等に分散することです。 コンテナーが格納できる合計データ量やコンテナーがサポートできる[スループット][throughput]の合計量には制限はありません。

### <a name="management-and-logging"></a>管理とログ記録

このソリューションは、[Application Insights][aai] を使用してパフォーマンスとログ記録の情報を収集します。 デプロイにより、このデプロイに必要なその他のサービスと同じリソース グループ内に Application Insights のインスタンスが作成されます。

ソリューションによって生成されたログを表示するには、以下の手順を実行します。

1. [Azure portal][portal] に移動し、デプロイのために作成されたリソース グループに移動します。

2. **Application Insights** インスタンスをクリックします。

3. **[Application Insights]** セクションから **[調査\\検索]** に移動して、データを検索します。

### <a name="security"></a>セキュリティ

Azure Cosmos DB は、Microsoft で提供されている C\# SDK を使用して、セキュリティで保護された接続と共有アクセス署名を使用します。 外部に面している攻撃対象は他にはありません。 Azure Cosmos DB のセキュリティの[ベスト プラクティス][db-practices]について確認します。

## <a name="pricing"></a>価格

このデプロイを使用可能にしておくための一日の概算コストは、システム内でデータの移動がない場合、約 \$20 です。

Azure Cosmos DB はパワフルですが、このデプロイで最も高い[コスト][db-cost]が発生します。 提供されている Azure Functions コードをリファクタリングすることにより、別のストレージ ソリューションを使用できます。

Azure Functions の価格は、実行される[プラン][function-plan]によって異なります。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

> [!NOTE]
> 既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][free]を作成してください。

このシナリオのすべてのコードは、[GitHub][github] リポジトリから入手できます。 このリポジトリには、このデモのパイプラインにデータをフィードするジェネレーター アプリケーションの構築に使用されるソース コードが含まれています。

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
