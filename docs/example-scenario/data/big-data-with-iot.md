---
title: 建設業界での IoT とデータ分析
titleSuffix: Azure Example Scenarios
description: IoT デバイスとデータ分析を使用して、建設プロジェクトを包括的に管理および運用します。
author: alexbuckgit
ms.date: 08/29/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: IoT, data-analytics
ms.openlocfilehash: dba67dfa7eb480a892229a9bc57d5c5f7ee21017
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488156"
---
# <a name="iot-and-data-analytics-in-the-construction-industry"></a>建設業界での IoT とデータ分析

このシナリオ例は、多数の IoT デバイスからのデータを包括的なデータ分析アーキテクチャに統合して意思決定を向上および自動化するソリューションを構築している組織に関連します。 潜在的なアプリケーションには、建設、鉱業、製造、または多数の IoT ベースのデータ入力からの大量のデータが関係するその他の業界のソリューションが含まれます。

このシナリオの建設機械メーカーは、IoT と GPS の技術を使用してテレメトリ データを送信する車両、機器、ドローンを製造しています。 この会社は、動作状態と機器の正常性の監視を改善するため、データ アーキテクチャの最新化を考えています。 オンプレミスのインフラストラクチャを使用している会社の従来のソリューションを置き換えるのでは、時間と労力の両方がかかり、予想されるデータ量を処理するのに十分なスケールにできません。

会社は、クラウド ベースの "スマート建設" ソリューションの構築を望んでいます。 構築現場の広範囲のデータ セットを収集し、現場のさまざまな要素の操作とメンテナンスを自動化するものでなければなりません。 会社の目標は次のとおりです。

- 建設現場のすべての機器とデータを統合して分析することで、機器のダウンタイムを最小限にし、盗難を減らす。
- リモートで自動的に建設機器を制御して、労働力不足の影響を軽減し、最終的に作業員を減らして、経験の浅い作業員でもうまくいくようにする。
- サポート インフラストラクチャの運用コストと必要な労働力を最小限に抑え、生産性と安全性を高める。
- テレメトリ データの増加をサポートするため、インフラストラクチャを簡単に拡大できるようにする。
- システムの可用性を損なうことなく、国内でリソースをプロビジョニングすることにより、関連するすべての法的要件に準拠する。
- オープン ソースのソフトウェアを使用して、作業員の現在のスキルへの投資を最大化する。

IoT Hub や HDInsight などのマネージド Azure サービスを使用すると、お客様は、包括的なソリューションを迅速に構築して展開しながら、運用コストを抑えることができます。 データ分析のニーズが他にもある場合は、[Azure で使用可能なフル マネージドのデータ分析サービス][product-category]の一覧をご覧ください。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- 建設、鉱業、または機器製造のシナリオ
- 保管と分析のためのデバイス データの大規模なコレクション
- 大規模なデータセットの取り込みと分析

## <a name="architecture"></a>アーキテクチャ

![建設業界での IoT とデータ分析のためのアーキテクチャ][architecture]

このソリューションのデータ フローは次のとおりです。

1. 建設機器はセンサー データを収集し、Azure 仮想マシンのクラスターでホストされている負荷分散された Web サービスに、建設結果データを定期的に送信します。
2. カスタム Web サービスは、建設結果データを取り込み、やはり Azure 仮想マシン上で実行されている Apache Cassandra クラスターに格納します。
3. さまざまな建設機器の IoT センサーによって別のデータセットが収集され、IoT Hub に送信されます。
4. 収集された生データは、IoT Hub から Azure Blob Storage に直接送信されて、表示と分析にすぐに利用できます。
5. IoT Hub 経由で収集されたデータは、Azure Stream Analytics ジョブによってほぼリアルタイムで処理され、Azure SQL Database に格納されます。
6. アナリストとエンド ユーザーがセンサー データと画像を表示および分析するには、スマート建設クラウド Web アプリケーションを使用できます。
7. バッチ ジョブは、Web アプリケーションのユーザーによってオンデマンドで開始されます。 バッチ ジョブは、HDInsight 上の Apache Spark で実行されて、Cassandra クラスターに格納された新しいデータを分析します。

### <a name="components"></a>コンポーネント

- [IoT Hub](/azure/iot-hub/about-iot-hub) は、クラウド プラットフォームと建設機器および他の現場要素の間で行われる、デバイスごとの ID によるセキュリティ保護された双方向通信に対する中央のメッセージ ハブとして機能します。 IoT Hub は、データ分析パイプラインへのインジェストのために、各デバイスのデータを迅速に収集できます。
- [Azure Stream Analytics](/azure/stream-analytics/stream-analytics-introduction) は、デバイスおよび他のデータ ソースからの大量のデータ ストリームを分析できるイベント処理エンジンです。 また、データ ストリームから情報を抽出し、パターンやリレーションシップを特定することもできます。 このシナリオの Stream Analytics は、IoT デバイスからのデータを取り込んで分析し、結果を Azure SQL Database に格納します。
- [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) には、IoT デバイスやメーターからの分析されたデータの結果が格納されており、アナリストやユーザーは Azure ベースの Web アプリケーションを使用してそれを表示できます。
- [Blob ストレージ](/azure/storage/blobs/storage-blobs-introduction)には、IoT ハブ デバイスから収集された画像データが格納されています。 画像データは、Web アプリケーションを使用して表示できます。
- [Traffic Manager](/azure/traffic-manager/traffic-manager-overview) によって、さまざまな Azure リージョンにおけるサービス エンドポイントのユーザー トラフィックの分散が制御されます。
- [Load Balancer](/azure/load-balancer/load-balancer-overview) では、建設機器のデバイスからのデータ送信が VM ベースの Web サービス間に分散されて、高可用性が提供されます。
- [Azure Virtual Machines](/azure/virtual-machines) では、建設結果データを受信して Apache Cassandra データベースに取り込む Web サービスがホストされています。
- [Apache Cassandra](https://cassandra.apache.org) は分散型 NoSQL データベースであり、後で Apache Spark を使用して処理できるよう建設データを格納するために使用されます。
- [Web Apps](/azure/app-service/app-service-web-overview) では、エンド ユーザーの Web アプリケーションがホストされており、それを使用してソース データと画像のクエリと表示を行うことができます。 ユーザーは、アプリケーションを使用して Apache Spark でバッチ ジョブを開始することもできます。
- [HDInsight 上の Apache Spark](/azure/hdinsight/spark/apache-spark-overview) は、ビッグ データ分析アプリケーションのパフォーマンスを向上させるメモリ内処理をサポートします。 このシナリオの Spark は、Apache Cassandra に格納されているデータに対する複雑なアルゴリズムの実行に使用されます。

### <a name="alternatives"></a>代替手段

- [Cosmos DB](/azure/cosmos-db/introduction) は、代わりの NoSQL データベース テクノロジです。 Cosmos DB では、[複数の明確に定義された整合性レベル](/azure/cosmos-db/consistency-levels)で[世界規模のマルチマスターのサポート](/azure/cosmos-db/multi-region-writers)が提供され、さまざまな顧客要件を満たすことができます。 また、[Cassandra API](/azure/cosmos-db/cassandra-introduction) もサポートされています。
- [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) は、Azure に最適化された Apache Spark ベースの分析プラットフォームです。 Azure と統合されており、ワン クリックのセットアップ、合理化されたワークフロー、対話型のコラボレーション ワークスペースを提供します。
- [Data Lake Storage](/azure/storage/data-lake-storage) は BLOB ストレージの代替機能です。 このシナリオの対象リージョンでは、Data Lake Storage を使用できませんでした。
- [Web Apps](/azure/app-service) は、建設結果データを取り込むための Web サービスのホストにも使用できました。
- リアルタイム メッセージ インジェスト、データ ストレージ、ストリーム処理、分析データのストレージ、および分析とレポート作成では、使用できるテクノロジ オプションが多数あります。 これらのオプションやその機能、主要な選択条件の概要については、「[ビッグ データ アーキテクチャ: リアルタイム処理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)」([Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)内) をご覧ください。

## <a name="considerations"></a>考慮事項

Azure リージョンを広範に使用できることは、このシナリオの重要な要素です。 1 つの国に複数のリージョンがあると、ディザスター リカバリーを実現しながら、契約上の義務と法律実施要件にも準拠できます。 Azure のリージョン間の高速通信も、このシナリオでは重要な要因です。

Azure によるオープン ソース テクノロジのサポートにより、お客様は既存の作業員のスキルを活用できました。 また、お客様は、オンプレミスのソリューションと比較して少ないコストと運用ワークロードで、新しいテクノロジの導入を加速できます。

## <a name="pricing"></a>価格

以下の考慮事項によって、このソリューションのコストのかなりの部分が決まります。

- Azure の仮想マシンのコストは、インスタンスが追加プロビジョニングされのに比例して増加します。 割り当てを解除された仮想マシンについては、ストレージ コストのみが発生し、コンピューティング コストはかかりません。 割り当てを解除されたマシンは、需要が増えたら再割り当てできます。
- [IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) のコストは、プロビジョニングされた IoT ユニットの数と、選択されたサービス レベルによって決まります。サービス レベルにより、ユニットごとに許可される 1 日のメッセージ数が決まります。
- [Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics) の価格は、このサービスに入力されるデータを処理するために必要なストリーミング ユニットの数に基づいて決まります。

## <a name="related-resources"></a>関連リソース

同様のアーキテクチャの実装については、[コマツの事例][customer-story]をご覧ください。

ビッグ データ アーキテクチャのガイダンスについては、「[Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)」をご覧ください。

<!-- links -->

[product-category]: https://azure.microsoft.com/product-categories/analytics/
[customer-site]: https://home.komatsu/en/
[customer-story]: https://customers.microsoft.com/story/komatsu-manufacturing-azure-iot-hub-japan
[architecture]: ./media/architecture-big-data-with-iot.png
