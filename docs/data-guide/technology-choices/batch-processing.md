---
title: バッチ処理テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 53f8b233b0e0c1ff83a72a04b2707caa528d6f6b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248517"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Azure で使用するバッチ処理テクノロジの選択

ビッグ データ ソリューションでは、長時間実行されるバッチ ジョブを使用して、データのフィルター処理、集計、または分析の準備を行うことがよくあります。 通常、このようなジョブには、スケーラブル ストレージ (HDFS、Azure Data Lake Store、Azure Storage など) からソース ファイルを読み取り、処理し、出力をスケーラブル ストレージの新しいファイルに書き込む作業が含まれます。

このようなバッチ処理エンジンの重要な要件は、大量のデータを処理するために、計算をスケールアウトできることです。 ただし、リアルタイム処理とは異なり、バッチ処理の場合、分単位から時間単位の待機時間 (データ インジェストから結果の計算までの時間) が生じることが予想されます。

## <a name="technology-choices-for-batch-processing"></a>バッチ処理向けのテクノロジの選択

### <a name="azure-sql-data-warehouse"></a>Azure SQL Data Warehouse

[SQL Data Warehouse](/azure/sql-data-warehouse/) は、大規模なデータの分析を目的として設計された分散システムです。 超並列処理 (MPP) がサポートされているので、ハイパフォーマンス分析の実行に適しています。 大量のデータ (1 TB 超) があり、並列処理のメリットが得られる分析ワークロードを実行する場合に、SQL Data Warehouse を検討します。

### <a name="azure-data-lake-analytics"></a>Azure Data Lake Analytics

[Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview) は、オンデマンド分析ジョブ サービスです。 Azure Data Lake Store に格納されている非常に大きなデータ セットの分散処理に最適化されています。

- 言語:[U-SQL](/azure/data-lake-analytics/data-lake-analytics-u-sql-get-started) (Python、R、および C# の拡張機能を含む)。
- Azure Data Lake Store、Azure Storage Blob、Azure SQL Database、および SQL Data Warehouse と統合されています。
- 価格モデルはジョブごとです。

### <a name="hdinsight"></a>HDInsight

HDInsight はマネージド Hadoop サービスです。 これを使用して Azure で Hadoop クラスターをデプロイおよび管理します。 バッチ処理では、[Spark](/azure/hdinsight/spark/apache-spark-overview)、[Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)、[Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)、[MapReduce](/azure/hdinsight/hadoop/hdinsight-use-mapreduce) を使用できます。

- 言語:R、Python、Java、Scala、SQL
- Active Directory による Kerberos 認証、Apache Ranger ベースのアクセス制御
- Hadoop クラスターのフル コントロールが可能

### <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/) は、Apache Spark ベースの分析プラットフォームです。 "サービスとしての Spark" と考えることができます。 Azure プラットフォームで Spark を使用する最も簡単な方法です。

- 言語:R、Python、Java、Scala、Spark SQL
- 迅速なクラスターの開始、自動終了、自動スケーリング
- Spark クラスターの自動管理
- Azure Blob Storage、Azure Data Lake Storage (ADLS)、Azure SQL Data Warehouse (SQL DW) などのサービスを統合済み。 [データ ソース](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html)に関するセクションを参照してください。
- Azure Active Directory によるユーザー認証
- Web ベースの[ノートブック](https://docs.azuredatabricks.net/user-guide/notebooks/index.html)によるコラボレーションとデータ探索
- [GPU 対応クラスター](https://docs.azuredatabricks.net/user-guide/clusters/gpu.html)をサポート

### <a name="azure-distributed-data-engineering-toolkit"></a>Azure Distributed Data Engineering Toolkit

[Distributed Data Engineering Toolkit](https://github.com/azure/aztk) (AZTK) は、Azure の Docker クラスターでオンデマンドの Spark をプロビジョニングするためのツールです。

AZTK は Azure サービスではありません。 CLI および Python SDK インターフェイスを使用するクライアント側のツールであり、Azure Batch 上に構築されます。 このオプションを使用すると、Spark クラスターをデプロイするときにインフラストラクチャを最大限制御できます。

- 独自の Docker イメージを持ち込みます。
- 優先順位の低い VM を使用すると 80% の割引が適用されます。
- 優先順位の低い VM と専用 VM の両方を使用する混合モード クラスター
- Azure Blob Storage と Azure Data Lake 接続をサポート済み

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- 独自のサーバーを管理するのではなく、マネージド サービスを使用しますか。

- バッチ処理ロジックの作成は宣言型と命令型のどちらですか。

- 大量にバッチ処理を実行しますか。 "はい" の場合は、クラスターを自動終了するオプションか、価格モデルがバッチ ジョブごとであるオプションを検討してください。

- 参照データを検索するなど、バッチ処理と共にリレーショナル データ ストアに対してクエリを実行する必要はありますか。 "はい" の場合は、外部リレーショナル ストアのクエリ処理が可能なオプションを検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

<!-- markdownlint-disable MD033 -->

| | Azure Data Lake Analytics | Azure SQL Data Warehouse | HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- |
| マネージド サービスか | はい | はい | はい <sup>1</sup> | はい |
| リレーショナル データ ストア | はい | はい | いいえ  | いいえ  |
| 価格モデル | バッチ ジョブごと | クラスター時間単位 | クラスター時間単位 | Databricks の単位<sup>2</sup> + クラスター時間 |

[1] 手動構成とスケーリングを使用。

[2] Databricks の単位 (DBU) は、1 時間あたりの処理能力の単位です。

### <a name="capabilities"></a>機能

| | Azure Data Lake Analytics | SQL Data Warehouse | Spark を使用する HDInsight | Hive を使用する HDInsight | Hive LLAP を使用する HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- | --- |
| 自動スケール | いいえ  | いいえ  | いいえ  | いいえ  | いいえ  | はい |
| スケールアウトの細分性  | ジョブごと | クラスターごと | クラスターごと | クラスターごと | クラスターごと | クラスターごと |
| データのメモリ内キャッシュ | いいえ  | 可能  | はい | いいえ  | 可能  | はい |
| 外部リレーショナル ストアからのクエリ | はい | いいえ  | はい | いいえ  | いいえ  | はい |
| Authentication  | Azure AD | SQL / Azure AD | いいえ  | Azure AD<sup>1</sup> | Azure AD<sup>1</sup> | Azure AD |
| 監査  | はい | はい | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> | はい |
| 行レベルのセキュリティ | いいえ  | いいえ  | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> | いいえ  |
| ファイアウォールをサポート | はい | はい | はい | はい <sup>2</sup> | はい <sup>2</sup> | いいえ  |
| 動的データ マスク | いいえ  | いいえ  | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> | いいえ  |

<!-- markdownlint-enable MD033 -->

[1] [ドメイン参加済み HDInsight クラスター](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)を使用する必要があります。

[2] [Azure Virtual Network 内で使用する](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)場合にサポートされます。
