---
title: Azure でのスケーラブルな注文処理
description: Azure Cosmos DB を使用して高度にスケーラブルな注文処理パイプラインを構築します。
author: alexbuckgit
ms.date: 07/10/2018
ms.openlocfilehash: fe642ffde733914389c36c5be50f35d242a22edf
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818515"
---
# <a name="scalable-order-processing-on-azure"></a>Azure でのスケーラブルな注文処理

このサンプル シナリオは、オンライン注文処理のために拡張性と回復性の高いアーキテクチャを必要とする組織に関連します。 考えられる用途としては、eコマースと小売販売管理、受注処理、および在庫予約と追跡が挙げられます。 

このシナリオのイベント ソーシング アプローチでは、マイクロサービスを介して実装される関数型プログラミング モデルが使用されています。 各マイクロサービスがストリーム プロセッサとして扱われ、すべてのビジネス ロジックがマイクロサービスを使って実装されます。 このアプローチにより、高可用性と回復性、geo レプリケーション、および高速なパフォーマンスが実現します。

Cosmos DB、HDInsight などのマネージド Azure サービスを使用して、クラウド規模のグローバル分散データ ストレージおよび取得における Microsoft の専門知識を活用すると、コスト削減に役立ちます。 このシナリオでは、特に eコマースと小売のシナリオを扱っています。データ サービスについて他のニーズがある場合は、使用可能な [Azure のフル マネージドのインテリジェント データベース サービス][product-category]の一覧を確認する必要があります。

## <a name="relevant-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* eコマースまたは小売販売管理のバックエンド システム。
* 在庫管理システム。
* 受注システム。
* 注文処理パイプラインに関連する他の統合シナリオ。

## <a name="architecture"></a>アーキテクチャ

![スケーラブルな注文処理パイプラインのアーキテクチャの例][architecture]

このアーキテクチャは、注文処理パイプラインの主要コンポーネントの詳細を示しています。 このシナリオのデータ フローは次のとおりです。

1. イベント メッセージが、顧客向けアプリケーションを介して (HTTP 経由で同期的に) システムに届きます。また、さまざまなバックエンド システムに (Apache Kafka 経由で非同期的に) 届きます。 これらのメッセージは、コマンド処理パイプラインに渡されます。
2. 各イベント メッセージが取り込まれ、コマンド プロセッサ マイクロサービスによって定義された一連のコマンドのいずれかにマップされます。 コマンド プロセッサは、イベント ストリームのスナップショット データベースからのコマンド実行に関連する現在の状態を取得します。 その後、コマンドが実行され、そのコマンドの出力が新しいイベントとして生成されます。
3. コマンドの出力として生成された各イベントが、Cosmos DB を使用して、イベント ストリームのデータベースにコミットされます。
4. イベント ストリームのデータベースにコミットされたデータベース挿入または更新ごとに、Cosmos DB Change Feed によってイベントが発生します。 そのシステムに関連するすべてのイベント トピックに、ダウンストリーム システムがサブスクライブできます。
5. Cosmos DB Change Feed からのイベントもすべて、スナップショット イベント ストリームのマイクロサービスに送信されます。これにより、発生したイベントによるすべての状態の変更が計算されます。 その後、新しい状態が、Cosmos DB に格納されているイベント ストリームのスナップショット データベースにコミットされます。 スナップショット データベースは、現在の状態のすべてのデータ要素に対して、待機時間の短いグローバル分散データ ソースを提供します。 イベント ストリームのデータベースは、アーキテクチャを介して渡されるすべてのイベント メッセージの完全なレコードを提供することで、堅牢なテスト、トラブルシューティング、およびディザスター リカバリーのシナリオを有効にします。

### <a name="components"></a>コンポーネント

* [Cosmos DB](/azure/cosmos-db/introduction) は Microsoft のグローバル分散マルチモデル データベースです。これにより、ソリューションでは、スループットとストレージを、任意の数の地理的リージョンで柔軟かつ個別にスケーリングできるようになります。 このサービスは包括的なサービス レベル アグリーメント (SLA) により、スループット、待機時間、可用性、一貫性が保証されています。 このシナリオでは、イベント ストリーム ストレージおよびスナップショット ストレージに Cosmos DB を使用し、[Cosmos DB の Change Feed][docs-cosmos-db-change-feed] 機能を利用して、データの一貫性と障害からの復旧を実現します。
* [HDInsight 上の Apache Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) は、リアルタイムのストリーミング データ パイプラインとアプリケーションを構築するための、オープン ソースの分散ストリーム プラットフォームである Apache Kafka のマネージド サービス実装です。 Kafka は、名前付きデータ ストリームへの公開とサブスクライブのために、メッセージ キューと同様のメッセージ ブローカー機能も提供しています。 このシナリオでは、Kafka を使用して、注文処理パイプラインの受信およびダウンストリーム イベントを処理します。 

## <a name="considerations"></a>考慮事項

リアルタイム メッセージ インジェスト、データ ストレージ、ストリーム処理、分析データのストレージ、および分析とレポート作成では、使用できるテクノロジ オプションが多数あります。 これらのオプションやその機能、主要な選択条件の概要については、「[Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)」の[ビッグ データ アーキテクチャのリアルタイム処理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)に関するページをご覧ください。

現在、マイクロサービスは、回復性優れ、単独でのデプロイが可能で、迅速に展開できるスケーラブルなクラウド アプリケーションを構築するための一般的なアーキテクチャ スタイルになっています。 マイクロサービスには、アプリケーションを設計およびビルドするのためのさまざまなアプローチが必要です。 Docker、Kubernetes、Azure Service Fabric、Nomad などのツールを使用すると、マイクロサービス ベースのアーキテクチャを開発できます。 マイクロサービス ベースのアーキテクチャの構築とガイダンスについては、Azure アーキテクチャ センターの [Azure でのマイクロサービスの設計](/azure/architecture/microservices)に関するページをご覧ください。

### <a name="availability"></a>可用性

このシナリオのイベント ソーシング アプローチを使用すると、システム コンポーネントの疎結合が可能になり、それぞれ個別にデプロイできます。 Cosmos DB により[高可用性][docs-cosmos-db-regional-failover]が実現します。また、Cosmos DB は、組織が一貫性、可用性、パフォーマンスと関連するトレードオフを、それぞれに[対応する保証][docs-cosmos-db-guarantees]と共に、管理する際に役立ちます。 HDInsight 上の Apache Kafka も、[高可用性][docs-kafka-high-availability]を確保できるよう設計されています。

Azure Monitor には、さまざまな Azure サービスにわたって監視するための統合ユーザー インターフェイスが用意されています。 詳細については、[Microsoft Azure での監視](/azure/monitoring-and-diagnostics/monitoring-overview)に関するページをご覧ください。 Event Hubs と Stream Analytics は両方とも、Azure Monitor に統合されています。 

その他の可用性の考慮事項については、[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

HDInsight 上の Kafka を使用すると、Kafka クラスターの[ストレージとスケーラビリティを構成](/azure/hdinsight/kafka/apache-kafka-scalability)できます。 Cosmos DB により、予測したとおりのパフォーマンスを迅速に確保し、アプリケーションの成長に合わせて[シームレスにスケーリング](/azure/cosmos-db/partition-data)できます。
このシナリオのイベント ソーシング マイクロサービス ベースのアーキテクチャでも、システムのスケーリングとその機能の拡張が行いやすくなります。

スケーラビリティに関する考慮事項については、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

[Cosmos DB セキュリティ モデル](/azure/cosmos-db/secure-access-to-data)では、ユーザーを認証し、そのデータとリソースへのアクセスを提供しています。 詳細については、[Cosmos DB のデータ セキュリティ](/azure/cosmos-db/database-security)に関するページをご覧ください。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

このサンプル シナリオのイベント ソーシング アーキテクチャと関連テクノロジにより、障害発生時に高い回復性が実現します。 回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のシナリオについて価格の変化を確認するには、予想されるデータ ボリュームに合わせて該当する変数を変更します。 このシナリオで価格の例に含まれるのは、Cosmos DB Change Feed から生成されたイベントを処理するための Cosmos DB と Kafka クラスターのみです。 送信元システムと他のダウンストリーム システムのイベント プロセッサおよびマイクロサービスは含まれず、そのコストは、これらのサービスと、そのサービスを実装するために選択されたテクノロジの数量とスケールによって大きく異なります。

Azure Cosmos DB の通貨は要求ユニット (RU) です。 要求ユニットが使用されるので、読み取り/書き込み容量を予約したり、CPU、メモリ、IOPS をプロビジョニングしたりする必要はありません。 Azure Cosmos DB は、単純な読み取りと書き込みから複雑なグラフのクエリまで、さまざまな操作を行うための API を幅広くサポートしています。 すべての要求が同等とは限らないので、要求には、それを処理するために必要な計算量に基づいて、正規化された要求ユニットの量が割り当てられます。 ご自身のソリューションに必要な要求ユニット数は、データ要素のサイズと、秒あたりのデータベース読み取りおよび書き込み操作数に酔って異なります。 詳細については、「[Azure Cosmos DB の要求ユニット](/azure/cosmos-db/request-units)」を参照してください。 これらの推定価格は、2 つの Azure リージョンで実行されている Cosmos DB に基づきます。

想定するアクティビティの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。

* [Small][small-pricing]: この価格例は、Cosmos DB と小規模 (D3 v2) Kafka クラスターの 1 TB のデータ ストアで予約された 5 RU に対応します。
* [Medium][medium-pricing]: この価格例は、Cosmos DB と中規模 (D4 v2) Kafka クラスターの 10 TB のデータ ストアで予約された 50 RU に対応します。
* [Large][large-pricing]: この価格例は、Cosmos DB と大規模 (D5 v2) Kafka クラスターの 30 TB のデータ ストアで予約された 500 RU に対応します。

## <a name="related-resources"></a>関連リソース

このサンプル シナリオは、[jet.com 社](https://jet.com)が、そのエンド ツー エンド注文処理パイプライン用に構築した、このアーキテクチャの大規模バージョンに基づいています。 詳細については、[Jet.com 社のテクニカル カスタマー プロファイル][source-document]と、[Build 2018 での Jet.com 社のプレゼンテーション][source-presentation]を参照してください。

その他の関連リソースには次のものがあります。
* _[データ量の多いアプリケーションの設計](https://dataintensive.net)_ Martin Kleppmann (O'Reilly Media、2017)。
* _[機能的なドメイン モデリング: ドメイン主導の設計と F# でソフトウェアの複雑さに取り組む](https://pragprog.com/book/swdddf/domain-modeling-made-functional)_ Scott Wlaschin (Pragmatic Programmers LLC、2018)。
* その他の[Cosmos DB のユース ケース][docs-cosmos-db-use-cases]
* 「[Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)」の「[リアルタイム処理](/azure/architecture/data-guide/big-data/real-time-processing)」

<!-- links -->
[architecture]: ./media/architecture-ecommerce-order-processing.png
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
