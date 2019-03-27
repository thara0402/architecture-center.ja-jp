---
title: eコマースのインテリジェントな製品検索エンジン
titleSuffix: Azure Example Scenarios
description: eコマース アプリケーションに世界水準の検索エクスペリエンスを提供します。
author: jelledruyts
ms.date: 09/14/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-ecommerce-search.png
ms.openlocfilehash: 12829c0b765831efdd08c8116a8fd847bd4e6abd
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245813"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a>eコマースのインテリジェントな製品検索エンジン

このシナリオ例では、専用の検索サービスを使用することにより、eコマースの顧客に対する検索結果の関連性が大幅に向上することを示します。

検索は顧客が製品を探して最終的に購入するときに利用する主要なメカニズムなので、検索結果が検索クエリの "_意図_" に関連していること、そしてほぼ即時の結果、言語分析、地理的な場所の一致、フィルター処理、ファセット、オートコンプリート、検索結果の強調表示などを提供することによって、検索エクスペリエンスが大手の検索エンジンに匹敵することが重要になります。

SQL Server や Azure SQL Database などのリレーショナル データベースに製品データが格納されている一般的な eコマース Web アプリケーションを想像してください。 多くの場合、検索クエリは、`LIKE` クエリや[フル テキスト検索][docs-sql-fts]機能を使用して、データベース内で処理されます。 代わりに [Azure Search][docs-search] を使用することで、運用データベースをクエリ処理から解放し、顧客に考えられる最良の検索エクスペリエンスを提供するこれらの実装が困難な機能を簡単に使い始めることができます。 また、Azure Search はサービスとしてのプラットフォーム (PaaS) コンポーネントであるため、インフラストラクチャの管理を心配したり、検索のエキスパートになる必要はありません。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- 物理的にユーザーの近くにある不動産一覧または店舗の検索。
- "_最近_" の情報を優先した、ニュース サイトの記事やスポーツ結果の検索。
- 政策立案者や公証人など、"_ドキュメント中心_" の組織向けの大規模なリポジトリの検索。

最終的には、何らかの形式の検索機能を持つ "_すべての_" アプリケーションが、専用検索サービスからメリットを得ることができます。

## <a name="architecture"></a>アーキテクチャ

![eコマース向けのインテリジェントな製品検索エンジンに関連する Azure コンポーネントのアーキテクチャの概要][architecture]

このシナリオでは、顧客が製品カタログを検索できる eコマース ソリューションについて説明します。

1. 顧客は、任意のデバイスから **eコマース Web アプリケーション**に移動します。
2. 製品カタログは、トランザクション処理のために **Azure SQL Database** に保持されています。
3. Azure Search は、**検索インデクサー**を使用し、統合された変更追跡機能によって、検索インデックスを自動的に最新の状態に保ちます。
4. 顧客の検索クエリは **Azure Search** サービスにオフロードされ、そこでクエリが処理されて、最も関連性の高い結果が返されます。
5. Web ベースの検索エクスペリエンスの代替手段として、顧客は、ソーシャル メディア内で、またはデジタル アシスタントから直接**会話型ボット**を使用して、製品を検索し、検索クエリと結果を段階的に調整することもできます。
6. 必要に応じて、**コグニティブ検索**機能を使用して人工知能を適用し、処理をさらにスマートにすることができます。

### <a name="components"></a>コンポーネント

- [App Services - Web Apps][docs-webapps] により Web アプリケーションがホストされ、自動スケーリングと高可用性が可能になります。インフラストラクチャの管理は不要です。
- [SQL Database][docs-sql-database] は、リレーショナル データ、JSON、空間、XML などの構造をサポートする、Microsoft Azure における汎用リレーショナル データベース管理サービスです。
- [Azure Search][docs-search] はサービスとしての検索クラウド ソリューションで、多彩な検索機能を、Web、モバイル、およびエンタープライズ アプリケーションのプライベートな異種コンテンツに提供します。
- [Bot Service][docs-botservice]: インテリジェント ボットの構築、テスト、デプロイ、管理を行うツールを提供します。
- [Cognitive Services][docs-cognitive]: 自然なコミュニケーション手段を通じて、見る、聞く、話す、理解する、ユーザーのニーズを解釈することが可能なインテリジェントなアルゴリズムを使用できます。

### <a name="alternatives"></a>代替手段

- たとえば、SQL Server のフルテキスト検索で**データベース内検索**機能を使用できますが、その場合、トランザクション ストアでもクエリが処理され (必要な処理能力の増加)、データベース内の検索機能の方が限定的です。
- オープン ソースの [Apache Lucene][apache-lucene] (その上に Azure Search が構築されます) を Azure Virtual Machines でホストできますが、そうすると、インフラストラクチャとしてのサービス (IaaS) の管理に戻り、Azure Search が Lucene 上で提供する多くの機能のメリットを得られません。
- Azure Marketplace からの [Elastic Search][elastic-marketplace] の展開を検討することもできます。これは代替機能であり、サード パーティ ベンダーからの製品を検索できますが、この場合も IaaS ワークロードを実行することになります。

データ層の他のオプションを次に示します。

- [Cosmos DB](/azure/cosmos-db/introduction) は、Microsoft のグローバル分散型マルチモデル データベースです。 Costmos DB では、Mongo DB、Cassandra、Graph データ、シンプルなテーブル ストレージなど、他のデータ モデルを実行するためのプラットフォームが提供されます。 Azure Search では、Cosmos DB からのデータに直接インデックスを付けることもサポートされています。

## <a name="considerations"></a>考慮事項

### <a name="scalability"></a>スケーラビリティ

Azure Search Service の[価格レベル][search-tier]では、使用できる機能は決まりませんが、取得できる最大ストレージおよびプロビジョニングできるパーティションとレプリカの数が定義されるので、主として[キャパシティ プランニング][search-capacity]に使用されます。 **パーティション**を使用するとより多くのドキュメントにインデックスを付けることができ、書き込みスループットが向上するのに対し、**レプリカ**では 1 秒あたりのクエリ数 (QPS) が増えて高可用性が提供されます。

パーティションとレプリカの数を動的に変更することはできますが、価格レベルは変更できないので、対象のワークロードに適したレベルを慎重に検討する必要があります。 どうしてもレベルを変更する必要がある場合は、新しいサービスを並行してプロビジョニングし、インデックスを読み込みなおす必要があります。この時点で、新しいサービスをアプリケーションから参照できます。

### <a name="availability"></a>可用性

Azure Search では、"_読み取り_" (つまり、クエリ) については少なくとも 2 つのレプリカがある場合に、"_更新_" (つまり、検索インデックスの更新) については少なくとも 3 つのレプリカがある場合に、[99.9% の可用性の SLA][search-sla] が提供されます。 したがって、顧客が確実に "_検索_" できるようにする場合は少なくとも 2 つのレプリカをプロビジョニングし、実際の "_インデックスに対する変更_" も高可用性操作と考える必要がある場合は少なくとも 3 つのレプリカをプロビジョニングする必要があります。

インデックスに対する破壊的変更 (たとえば、データ型の変更や、フィールドの削除または名前変更) をダウンタイムなしで行う必要がある場合は、インデックスを再構築する必要があります。 サービス レベルの変更と同様に、これは、新しいインデックスを作成し、それにデータを再設定して、新しいインデックスを参照するようアプリケーションを更新することを意味します。

### <a name="security"></a>セキュリティ

Azure Search は多くの[セキュリティおよびデータ プライバシーの標準][search-security]に準拠しているので、ほとんどの業界で使用できます。

サービスへのアクセスをセキュリティ保護するため、Azure Search では 2 種類のキーが使用されています。**管理者キー**ではサービスに対して "_すべての_" タスクを実行できるのに対し、**クエリ キー**はクエリのような読み取り専用操作に対してのみ使用できます。 通常、検索を実行するアプリケーションはインデックスを更新しないため、クエリ キーのみを構成し、管理者キーは構成しないようにする必要があります (特に、Web ブラウザーで実行されるスクリプトのように、エンド ユーザーのデバイスから検索が実行される場合)。

### <a name="search-relevance"></a>検索の関連性

eコマース アプリケーションがどれくらい成功するかは、顧客に対する検索結果の関連性に大きく依存します。 ユーザーの研究に基づいて最適な結果を提供するように検索サービスを慎重に調整するか、または[検索トラフィック分析][search-analysis]などの組み込み機能を利用して顧客の検索パターンを理解することにより、データに基づいて決定を行うことができます。

検索サービスを調整する一般的な方法は次のとおりです。

- [スコアリング プロファイル][search-scoring]を使用して、検索結果の関連性に影響を与えます。たとえば、クエリと一致したフィールド、データの新しさ、ユーザーに対する地理的な距離などを基にします。
- 高度な自然言語処理 (NLP) スタックを使用してクエリをより適切に解釈する [Microsoft 提供の言語アナライザー][search-languages]を使用します。
- [カスタム アナライザー][search-analyzers]を使用して、製品が正しく検出されるようにします。特に、製品の製造元やモデルのような言語に基づかない情報で検索する場合。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

このシナリオのさらに完全な eコマース バージョンを展開するには、簡単なチケット購入アプリケーションを実行する .NET サンプル アプリケーションを提供するこちらの[ステップ バイ ステップ チュートリアル][end-to-end-walkthrough]に従ってください。 これには Azure Search も含まれ、説明した機能の多くが使用されています。 さらに、ほとんどの Azure リソースのデプロイを自動化する Resource Manager テンプレートもあります。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、これまでに説明したすべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想される使用量に合わせて該当する変数を変更します。

取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。

- [小][small-pricing]:このプロファイルでは、Web サイトをホストするための 1 つの `Standard S1` Web アプリ、Azure Bot Service の Free レベル、1 つの `Basic` Azure Search Service、`Standard S2` SQL Database を使用しています。
- [中][medium-pricing]:ここでは、Web アプリを `Standard S3` レベルの 2 つのインスタンスにスケールアップし、Search Service を `Standard S1` レベルにアップグレードし、`Standard S6` SQL Database を使用しています。
- [大][large-pricing]:最大のプロファイルでは、`Premium P2V2` Web アプリの 4 つのインスタンスを使用し、Azure Bot Service を `Standard S1` レベル (Premium チャネルで 1.000.000 メッセージ) にアップグレードし、2 ユニットの `Standard S3` Azure Search Service と `Premium P6` SQL Database を使用しています。

## <a name="related-resources"></a>関連リソース

Azure Search について詳しくは、[ドキュメント センター][docs-search]を参照するか、[サンプル][search-samples]を確認するか、本格的に動作する[デモ サイト][search-demo]をご覧ください。

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
