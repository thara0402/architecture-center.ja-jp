---
title: eコマース フロントエンド
titleSuffix: Azure Example Scenarios
description: Azure で eコマース サイトをホストします。
author: masonch
ms.date: 07/13/2018
ms.custom: fasttrack
ms.openlocfilehash: f07c21b8eb9d812b9831abe8f2e4f6d131893df2
ms.sourcegitcommit: 7d9efe716e8c9e99f3fafa9d0213d48c23d9713d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2019
ms.locfileid: "54160810"
---
# <a name="an-e-commerce-front-end-on-azure"></a>Azure の eコマース フロントエンド

このサンプル シナリオでは、Azure の提供するサービスとしてのプラットフォーム (PaaS) ツールを使用した eコマース フロントエンドの実装について説明します。 eコマース Web サイトの多くが、季節性および時期的なトラフィックの変動に直面します。 PaaS ツールを利用すれば、予想通りでも、予想外であっても、商品やサービスの需要が急増したときに、顧客や取引の増加に自動的に対処できます。 また、このシナリオでは、使用した容量分だけ支払うため、経済的にもクラウドの恩恵を受けられます。

このドキュメントは、サンプル eコマース アプリケーションしての、*Relecloud Concerts* というオンライン コンサート発券プラットフォームをデプロイする際にまとめて使用されるさまざまな Azure PaaS コンポーネントと考慮事項について説明します。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- さまざまなタイミングでユーザーの急増に対処できるように弾力性のあるスケーリングを必要とするアプリケーションを構築する。
- 世界中のさまざまな Azure リージョンで高い可用性で運用されるよう設計されたアプリケーションを構築する。

## <a name="architecture"></a>アーキテクチャ

![eコマース アプリケーションのサンプル シナリオ アーキテクチャ][architecture]

このシナリオでは、eコマース サイトからのチケット購入に対応します。シナリオのデータ フローを次に示します。

1. Azure Traffic Manager により、ユーザーの要求が、Azure App Service でホストされている eコマース サイトにルーティングされます。
2. Azure CDN は、静的なイメージとコンテンツをユーザーに提供します。
3. ユーザーが Azure Active Directory B2C テナントを介してアプリケーションにサインインします。
4. ユーザーが Azure Search を使用してコンサートを検索します。
5. Web サイトによって、Azure SQL Database からコンサートの詳細がプルされます。
6. Web サイトから、Blob Storage にある購入済みチケットの画像を参照します。
7. データベース クエリの結果が、パフォーマンス向上のため、Azure Redis Cache にキャッシュされます。
8. ユーザーが送信したチケット注文とコンサート レビューがキューに配置されます。
9. Azure Functions によって、注文の支払いとコンサート レビューが処理されます。
10. Cognitive Services ではコンサート レビューが分析され、センチメント (肯定的または否定的) が判別されます。
11. Application Insights が、Web アプリケーションの正常性を監視するためのパフォーマンス メトリックを提供します。

### <a name="components"></a>コンポーネント

- [Azure CDN][docs-cdn] が、待機時間を減らすために、ユーザーに近い場所から、キャッシュされた静的なコンテンツを提供します。
- [Azure Traffic Manager][docs-traffic-manager] によって、さまざまな Azure リージョンにおけるサービス エンドポイントのユーザー トラフィックの分散が制御されます。
- [App Services - Web Apps][docs-webapps] により Web アプリケーションがホストされ、自動スケーリングと高可用性が可能になります。インフラストラクチャの管理は不要です。
- [Azure Active Directory B2C][docs-b2c] は ID 管理サービスです。このサービスを使用すると、アプリケーションでの顧客のサインアップ、サインイン、およびプロファイル管理の方法をカスタマイズし、制御できます。
- [Storage キュー][docs-storage-queues]には、アプリケーションからアクセスできるキュー メッセージが多数格納されています。
- [Functions][docs-functions] は、インフラストラクチャを管理せずに、アプリケーションをオンデマンドで実行できるようにするサーバーレス コンピューティング オプションです。
- [Cognitive Services - 感情分析][docs-sentiment-analysis]は機械学習 API を使用して、開発者が、アプリケーションに感情認識や映像検出、顔認識、音声認識、視覚認識、音声理解、言語理解など、インテリジェントな機能を簡単に追加できるようにします。
- [Azure Search][docs-search] はサービスとしての検索クラウド ソリューションで、多彩な検索機能を、Web、モバイル、およびエンタープライズ アプリケーションのプライベートな異種コンテンツに提供します。
- [Storage Blob][docs-storage-blobs] は、テキスト データ、バイナリ データなど、大量の非構造化データを格納するために最適化されています。
- [Redis Cache][docs-redis-cache] は、アクセス頻度が高いデータを、アプリケーションに近い場所にある高速ストレージに一時的にコピーすることで、バックエンドのデータストアに大きく依存するシステムのパフォーマンスとスケーラビリティを向上させます。
- [SQL Database][docs-sql-database] は、リレーショナル データ、JSON、空間、XML などの構造をサポートする、Microsoft Azure における汎用リレーショナル データベース管理サービスです。
- [Application Insights][docs-application-insights] は、パフォーマンスやユーザビリティを継続的に向上させるうえで役立つように設計されています。これを実現するために、アプリでのユーザーの操作内容を把握しやすいように、組み込みの分析ツールを使用してパフォーマンスの異常が自動検出されます。

### <a name="alternatives"></a>代替手段

他にもさまざまなテクノロジを、顧客向けの大規模な eコマース用アプリケーションの構築に使用できます。 これらは、アプリケーションのフロントエンドとデータ層の両方を対象としています。

Web 層と機能に関するその他のオプションは次のとおりです。

- [Service Fabric][docs-service-fabric] - 細かな制御でクラスター全体にデプロイして実行することでメリットを得られる分散コンポーネントの構築に重点を置いたプラットフォーム。 Service Fabric を使ってコンテナーをホストすることもできます。
- [Azure Kubernetes Service][docs-kubernetes-service] - マイクロサービス アーキテクチャの実装として使用できる、コンテナー ベースのソリューションを構築およびデプロイするためのプラットフォーム。 これにより、アプリケーションのさまざまなコンポーネントの俊敏性が実現し、オンデマンドで個別にスケーリングできます。
- [Azure Container Instances][docs-container-instances] - コンテナーを短いライフサイクルですばやくデプロイし、実行する手段。 このコンテナーは、メッセージの処理や計算の実行など、簡単な処理ジョブを実行するためにデプロイされ、完了するとすぐにプロビジョニングが解除されます。
- [Service Bus][service-bus] は、ストレージ キューの代わりに使用できます。

データ層の他のオプションを次に示します。

- [Cosmos DB](/azure/cosmos-db/introduction):Microsoft のグローバル分散型マルチモデル データベースです。 このサービスは、Mongo DB、Cassandra、Graph データ、シンプルなテーブル ストレージなど、他のデータ モデルを実行するためのプラットフォームを提供します。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

- クラウド アプリケーションを構築するときは、[可用性のための標準的な設計パターン][design-patterns-availability]の利用を検討してください。
- 適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]で可用性に関する考慮事項を確認します
- 可用性に関する追加の考慮事項については、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

- クラウド アプリケーションを構築するときは、[スケーラビリティのための標準的な設計パターン][design-patterns-scalability]に注意してください。
- 適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でスケーラビリティに関する考慮事項を確認します
- スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

- 必要に応じて、[セキュリティのための標準的な設計パターン][design-patterns-security]を利用することを検討してください。
- 適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]のセキュリティに関する考慮事項を確認します。
- [セキュリティで保護された開発ライフサイクル][secure-development] プロセスに従って、開発コストを削減しながら、開発者がより安全なソフトウェアを構築し、セキュリティ コンプライアンス要件に対応できるようにします。
- [Azure PCI DSS コンプライアンス][pci-dss-blueprint]を実現するためのブループリント アーキテクチャを確認してください。

### <a name="resiliency"></a>回復性

- アプリケーションの一部が使用できない場合は、[サーキット ブレーカー パターン][circuit-breaker]を利用して、グレースフル エラー処理を提供することを検討してください。
- [回復性のための標準的な設計パターン][design-patterns-resiliency]を確認し、必要に応じて、これらを実装することを検討します。
- Azure アーキテクチャ センターでは、多数の [App Service に関する推奨プラクティス][resiliency-app-service]を確認できます。
- データ層にはアクティブ [geo レプリケーション][sql-geo-replication]を、イメージおよびキューには [geo 冗長][storage-geo-redudancy]ストレージを使用することを検討します。
- [回復性][resiliency]の詳細については、Azure アーキテクチャ センターの関連記事を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

このシナリオをデプロイするには、こちらの[ステップ バイ ステップのチュートリアル][end-to-end-walkthrough]に従います。このチュートリアルでは、各コンポーネントを手動でデプロイする方法を示しています。 また、このチュートリアルでは、シンプルなチケット購入アプリケーションを実行する .NET サンプル アプリケーションも提供します。 さらに、ほとんどの Azure リソースのデプロイを自動化する Resource Manager テンプレートもあります。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べてください。すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。

- [Small][small-pricing]:この価格例は、最小の運用レベル インスタンスの構築に必要なコンポーネントを表します。 ここでは、1 か月あたり数千人の少人数のユーザーを想定しています。 アプリでは標準の Web アプリの単一インスタンスが使用されており、自動スケーリングを有効にするにはこれで十分です。 その他のコンポーネントはそれぞれ Basic レベルにスケーリングされ、最小限のコストで、SLA をサポートし、運用レベルのワークロードを処理できるだけの容量が確保されます。
- [Medium][medium-pricing]:この価格例は、中規模サイズのデプロイの指標となるコンポーネントを表します。 ここでは、1 か月間に約 100,000 人のユーザーがシステムを使用することを推定しています。 予想されるトラフィックは、中程度の Standard レベルによって単一アプリ サービス インスタンスで処理されます。 また、中レベルのコグニティブおよび検索サービスが計算ツールに追加されています。
- [Large][large-pricing]:この価格例は、高度のスケーリングを想定したアプリケーションを表します。このアプリケーションでは、1 か月あたり数百万人のユーザーが数テラバイトのデータをやり取りします。 このレベルの使用状況では、トラフィック マネージャーがアクセスする複数のリージョンにデプロイされた、高パフォーマンスの Premium レベルの Web アプリが必要です。 データは、ストレージ、データベース、および CDN で構成され、これらはテラバイト データ用に構成されています。

## <a name="related-resources"></a>関連リソース

- [複数リージョンの Web アプリケーション向けリファレンス アーキテクチャ][multi-region-web-app]
- [eShopOnContainers の参照用の例][microservices-ecommerce]

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[service-bus]: /azure/service-bus-messaging/
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
