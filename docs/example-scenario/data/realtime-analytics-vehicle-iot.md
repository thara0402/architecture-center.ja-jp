---
title: 自動車のリアルタイム IoT データのインジェストと処理
titleSuffix: Azure Example Scenarios
description: IoT を使用して、リアルタイムの車両データを取り込んで処理します。
author: msdpalam
ms.date: 09/12/2018
ms.custom: fasttrack
ms.openlocfilehash: edb0dc495db8742ae07826de5158db48f919e81f
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644072"
---
# <a name="ingestion-and-processing-of-real-time-automotive-iot-data"></a>自動車のリアルタイム IoT データのインジェストと処理

このシナリオの例では、IoT デバイス (一般にはセンサー) からのメッセージを Azure のビッグ データ分析プラットフォームに取り込んで処理するリアルタイム データ インジェストおよび処理パイプラインを構築します。 車両テレマティクスのインジェストおよび処理プラットフォームは、コネクテッド カー ソリューションを作成するための鍵です。 この特定のシナリオは、自動車のテレマティクスのインジェストおよび処理システムを目的としています。 しかし、その設計パターンは、スマート ビルディング、通信、製造、小売、医療など、複雑なシステムを管理および監視するためにセンサーを使用する多くの業界にも関係があります。

この例は、車両に搭載された IoT デバイスからのメッセージのリアルタイム データ インジェストおよび処理パイプラインを示しています。 数千および数百万のメッセージ (またはイベント) が、IoT デバイスおよびセンサーによって生成されます。 それらのメッセージをキャプチャして分析すれば、貴重な分析情報が得られ、適切な措置を講じることができます。 たとえば、テレマティクス デバイスが搭載されている車でデバイス (IoT) メッセージをリアルタイムでキャプチャできれば、車両の現在地を監視し、最適なルートを計画し、ドライバーに支援を提供し、自動車保険などのテレマティクス関連産業をサポートすることができます。

このデモンストレーションの例で取り上げるのは、テレマティクス デバイスからのメッセージを取り込んで処理するリアルタイムのシステムを作成しようとしている自動車製造会社です。 会社の目標は次のとおりです。

- 車両のセンサーやデバイスからリアルタイムでデータを取り込んで格納する。
- メッセージを分析して、さまざまな種類のセンサー (エンジン関連センサーや環境関連センサーなど) によって発信される、車両の位置などの情報を解釈する。
- 解析後のデータを格納して、下流の他の処理で実用的な分析情報が得られるようにする (たとえば事故のシナリオなら、保険会社は事故で何が起きたのかに関心を持つと考えられます)

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- 車両メンテナンスのリマインダーとアラート。
- 車両乗客用の位置情報に基づくサービス (つまり、SOS)。
- 自律走行 (自動運転) 車両。

## <a name="architecture"></a>アーキテクチャ

![スクリーンショット](media/architecture-realtime-analytics-vehicle-data1.png)

一般的なビッグ データ処理パイプラインの実装では、データは左から右に流れます。 このリアルタイムのビッグ データ処理パイプラインでは、データは次のようにソリューションを通過します。

1. IoT データ ソースから生成されたイベントは、Azure HDInsight Kafka を介して、メッセージのストリームとしてストリーム インジェスト レイヤーに送信されます。 HDInsight Kafka では、構成可能な時間にわたってデータのストリームをトピックに分けて格納します。
2. Kafka のコンシューマーである Azure Databricks は、Kafka のトピックからリアルタイムでメッセージをピックアップし、ビジネス ロジックに基づいてデータを処理し、ストレージ用のサービス レイヤーに送信できます。
3. Azure Cosmos DB、Azure SQL Data Warehouse、Azure SQL DB などの下流のストレージ サービスは、プレゼンテーションとアクション レイヤーのデータ ソースになります。
4. ビジネス アナリストは、Microsoft Power BI を使用して、データウェアハウスに格納されているデータを分析できます。 サービス レイヤー上に他のアプリケーションを構築することもできます。 たとえば、サービス レイヤー データに基づく API を公開し、サード パーティが使用できるようにすることができます。

### <a name="components"></a>コンポーネント

IoT デバイスで生成されたイベント (データまたはメッセージ) は、取り込まれ、処理され、格納された後、さらに分析、表示、処置が行われます。これらの処理は、次の Azure コンポーネントを使用して行われます。

- [Apache Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) は、インジェスト レイヤーです。 データは Kafka のプロデューサー API を使用して Kafka トピックに書き込まれます。
- [Azure Databricks](/services/databricks) は、変換レイヤーと分析レイヤーに位置しています。 Databricks ノートブックは、Kafka トピックからデータを読み込む Kafka のコンシューマー API を実装します。
- [Azure Cosmos DB](/services/cosmos-db)、[Azure SQL Database](/azure/sql-database/sql-database-technical-overview)、Azure SQL Data Warehouse は、サービス ストレージ レイヤーにあります。Azure Databricks はデータ コネクタを介してこのレイヤーにデータを書き込むことができます。
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) は、大規模なデータセットを格納および分析するための分散システムです。 超並列処理 (MPP) が使用されているので、ハイパフォーマンス分析の実行に適しています。
- [Power BI](https://docs.microsoft.com/power-bi) は、データを分析し、洞察を共有する一連のビジネス分析ツールです。 Power BI は、Analysis Services に格納されているセマンティック モデルに対してクエリを実行することも、SQL Data Warehouse に対して直接クエリを実行することもできます。
- [Azure Active Directory (Azure AD)](/azure/active-directory) は、[Azure Databricks](https://azure.microsoft.com/services/databricks) に接続するときにユーザーを認証します。 Azure SQL Data Warehouse のデータに基づくモデルに従って [Analysis Services](/azure/analysis-services) にキューブを構築する場合は、AAD を使用して Power BI を介して Analysis Services サーバーに接続できます。 Data Factory も Azure AD を使用して、サービス プリンシパルまたはマネージド サービス ID (MSI) 経由で SQL Data Warehouse に対する認証を行えます。
- [Azure App Services](/azure/app-service/app-service-web-overview)、特に [API App](/services/app-service/api) を使用して、サービス レイヤーの格納データに基づいてサード パーティにデータを公開することができます。

## <a name="alternatives"></a>代替手段

![スクリーンショット](media/architecture-realtime-analytics-vehicle-data2.png)

他の Azure コンポーネントを使用すると、より汎用性の高いビッグ データのパイプラインを実装できます。

- ストリーム インジェスト レイヤーでは、[HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) の代わりに [IoT Hub](https://azure.microsoft.com/services/iot-hub) または[イベント ハブ](https://azure.microsoft.com/services/event-hubs)を使用してデータを取り込むことができます。
- 変換レイヤーと分析レイヤーでは、[HDInsight Storm](/azure/hdinsight/storm/apache-storm-overview)、[HDInsight Spark](/azure/hdinsight/spark/apache-spark-overview)、[Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics) を使用することもできます。
- [Analysis Services](/azure/analysis-services) は、データのセマンティック モデルを提供します。 これはデータの分析時のシステム パフォーマンスの向上にもつながります。 Azure DW データに基づいてモデルを構築することができます。

## <a name="considerations"></a>考慮事項

このアーキテクチャに含まれているテクノロジは、イベントを処理するために必要な規模、サービスの SLA、コスト管理、コンポーネントの管理容易性に基づいて選択されました。

- 管理された [HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) には、Azure Managed Disks と統合された 99.9% の SLA が付属しています。
- [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) は、クラウドのパフォーマンスとコスト効率が完全に最適化されています。 Databricks Runtime は、いくつかの重要な機能を Apache Spark ワークロードに加えて、Azure での実行時に 10 から 100 倍のパフォーマンス向上やコスト削減が可能です。これには以下が含まれます。
- Azure Databricks は、[Azure SQL Data Warehouse](/azure/sql-data-warehouse)、[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db)、[Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage)、[Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs) などの Azure のデータベースおよびストアと深いレベルで統合します
  - Spark クラスターの自動スケーリングと自動終了により、自動的にコストが最小限に抑えられます。
  - キャッシュ、インデックス作成、高度なクエリ最適化などのパフォーマンスの最適化により、クラウド環境やオンプレミスの環境での従来の Apache Spark のデプロイと比べてパフォーマンスを 10 倍から 100 倍向上させることができます。
  - Azure Active Directory との統合により、Azure Databricks を使って Azure ベースの完全なソリューションを実行することができます。
  - Azure Databricks のロール ベースのアクセスでは、ノートブック、クラスター、ジョブ、データに対してきめ細かいユーザー アクセス許可を設定できます。
  - エンタープライズ グレードの SLA が付属します。
- Azure Cosmos DB は、Microsoft のグローバル分散型マルチモデル データベースです。 Azure Cosmos DB は、グローバル分散と水平方向への拡張性を中心として一から構築されました。 透過的なスケーリングとあらゆるユーザーの所在地へのデータ レプリケーションにより、使い始めてすぐに任意の数の Azure リージョン全体でグローバル分散を実現できます。 スループットとストレージを世界規模で弾力的にスケーリングでき、お支払いは必要なスループットとストレージの分のみとなります。
- SQL Data Warehouse の超並列処理アーキテクチャにより、スケーラビリティとハイパフォーマンスを実現します。
- Azure SQL Data Warehouse には、保証された SLA と高可用性を実現するための推奨プラクティスがあります。
- 分析アクティビティが低下した場合、会社は Azure SQL Data Warehouse をオンデマンドでスケーリングし、計算を減らしたり、さらには計算を一時停止したりすることにより、コストを削減できます。
- Azure SQL Data Warehouse のセキュリティ モデルは、Azure AD または SQL Server の認証と暗号化による接続セキュリティ、認証、認可を提供します。

## <a name="pricing"></a>価格

Azure 料金計算ツールを使用して、[Azure Databricks の料金](https://azure.microsoft.com/pricing/details/databricks)、[Azure HDInsight の料金](https://azure.microsoft.com/pricing/details/hdinsight)、[データウェアハウス シナリオの料金サンプル](https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276)をご確認ください。 値を調整して、要件の変更に伴うコストの変化をご確認ください。

- [Azure HDInsight](/azure/hdinsight) はフルマネージド クラウド サービスで、膨大な量のデータを簡単かつ迅速に、コスト効率よく処理できます
- [Azure Databricks](https://azure.microsoft.com/services/databricks) では、データ分析のワークフローに合わせた 2 つの別々のワークロードを複数の [VM インスタンス](https://azure.microsoft.com/pricing/details/databricks/#instances)で使用できます。Data Engineering ワークロードを使用すると、データ エンジニアは、ジョブの作成と実行が簡単に行えます。また、Data Analytics ワークロードを使用すると、データ サイエンティストは、データや分析情報のインタラクティブな調査、視覚化、操作、共有が簡単に行えます。
- [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) では、世界中のあらゆる場所で待ち時間の 99 パーセンタイルが確実に 10 ミリ秒未満となります。また、[明確でわかりやすい複数の整合性モデル](/azure/cosmos-db/consistency-levels)でパフォーマンスを細かく調整することができ、マルチホーム機能により高可用性も保証されます。これらはすべて、業界トップ レベルの包括的な[サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/cosmos-db) (SLA) の対象となります。
- [Azure SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) を使用すると、計算レベルとストレージ レベルを個別にスケーリングすることができます。 計算リソースは 1 時間単位で課金されるため、オンデマンドでそのリソースをスケーリングまたは一時停止できます。 ストレージ リソースはテラバイト単位で課金されるため、データを取り込んだ分だけコストが増加します。
- [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) は、Developer レベル、Basic レベル、Standard レベルでご利用いただけます。 インスタンスは、クエリ処理単位 (QPU) と使用可能なメモリに基づいて価格設定されます。 コストを抑えるには、実行するクエリの数、処理するデータの量、実行頻度をできるだけ少なくします。
- [Power BI](https://powerbi.microsoft.com/pricing) には、要件に応じたさまざまな製品オプションがあります。 [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) では、Power BI の機能をアプリケーションに埋め込むための Azure ベースのオプションが提供されます。 Power BI Embedded インスタンスは上記の料金サンプルに含まれています。

## <a name="next-steps"></a>次の手順

- ビッグ データ パイプライン フローを含む[リアルタイム分析](https://azure.microsoft.com/solutions/architecture/real-time-analytics)参照アーキテクチャをご確認ください。
- 各種 Azure コンポーネントでビッグ データ パイプラインを構築する方法については、[ビッグ データの高度な分析](https://azure.microsoft.com/solutions/architecture/advanced-analytics-on-big-data)参照アーキテクチャをご確認ください。
- 各種 Azure コンポーネントでのデータ ストリームのリアルタイム処理方法の概略については、[リアルタイム処理](/azure/architecture/data-guide/big-data/real-time-processing)という Azure のドキュメントをお読みください。
- データ パイプライン、データウェアハウス、オンライン分析処理 (OLAP)、ビッグ データに関する包括的なアーキテクチャ ガイダンスについては、[Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)をご確認ください。
