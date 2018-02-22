---
title: "バッチ処理テクノロジの選択"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bfb850ee8e9d8fd41927b4ca3b612e15b5ae6b11
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Azure で使用するバッチ処理テクノロジの選択

ビッグ データ ソリューションでは、長時間実行されるバッチ ジョブを使用して、データのフィルター処理、集計、または分析の準備を行うことがよくあります。 通常、このようなジョブには、スケーラブル ストレージ (HDFS、Azure Data Lake Store、Azure Storage など) からソース ファイルを読み取り、処理し、出力をスケーラブル ストレージの新しいファイルに書き込む作業が含まれます。 

このようなバッチ処理エンジンの重要な要件は、大量のデータを処理するために、計算をスケールアウトできることです。 ただし、リアルタイム処理とは異なり、バッチ処理の場合、分単位から時間単位の待機時間 (データ インジェストから結果の計算までの時間) が生じることが予想されます。

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>バッチ処理テクノロジを選択する場合のオプション

Azure では、以下のすべてのデータ ストアがバッチ処理のコア要件を満たしています。

- [Azure Data Lake Analytics](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Spark を使用する HDInsight](/azure/hdinsight/spark/apache-spark-overview)
- [Hive を使用する HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Hive LLAP を使用する HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- 独自のサーバーを管理するのではなく、マネージド サービスを使用しますか。

- バッチ処理ロジックの作成は宣言型と命令型のどちらですか。

- 大量にバッチ処理を実行しますか。 "はい" の場合は、クラスターを一時停止するか、バッチ ジョブごとの価格モデルを選択することを検討してください。

- 参照データを検索するなど、バッチ処理と共にリレーショナル データ ストアに対してクエリを実行する必要はありますか。 "はい" の場合は、外部リレーショナル ストアのクエリ処理が可能なオプションを検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。 

### <a name="general-capabilities"></a>一般的な機能

| | Azure Data Lake Analytics | Azure SQL Data Warehouse | Spark を使用する HDInsight | Hive を使用する HDInsight | Hive LLAP を使用する HDInsight |
| --- | --- | --- | --- | --- | --- |
| マネージド サービスか | [はい] | [はい] | はい <sup>1</sup> | はい <sup>1</sup> | はい <sup>1</sup> |
| 計算の一時停止をサポート | いいえ  | [はい] | いいえ  | いいえ  | いいえ  |
| リレーショナル データ ストア | [はい] | [はい] | いいえ  | いいえ  | いいえ  |
| プログラミング | U-SQL | T-SQL | Python、Scala、Java、R | HiveQL | HiveQL |
| プログラミング パラダイム | 宣言型と命令型の混合  | 宣言型 | 宣言型と命令型の混合 | 宣言型 | 宣言型 | 
| 価格モデル | バッチ ジョブごと | クラスター時間単位 | クラスター時間単位 | クラスター時間単位 | クラスター時間単位 |  

[1] 手動構成とスケーリングを使用。
 
### <a name="integration-capabilities"></a>統合機能
| | Azure Data Lake Analytics | SQL Data Warehouse | Spark を使用する HDInsight | Hive を使用する HDInsight | Hive LLAP を使用する HDInsight |
| --- | --- | --- | --- | --- | --- |
| Azure Data Lake Store からのアクセス | [はい] | [はい] | [はい] | [はい] | [はい] |
| Azure Storage からのクエリ | [はい] | [はい] | [はい] | [はい] | [はい] |
| 外部リレーショナル ストアからのクエリ | [はい] | いいえ  | [はい] | いいえ  | いいえ  |

### <a name="scalability-capabilities"></a>スケーラビリティ機能
| | Azure Data Lake Analytics | SQL Data Warehouse | Spark を使用する HDInsight | Hive を使用する HDInsight | Hive LLAP を使用する HDInsight |
| --- | --- | --- | --- | --- | --- |
| スケールアウトの細分性  | ジョブごと | クラスターごと | クラスターごと | クラスターごと | クラスターごと |
| 高速スケールアウト (1 分未満) | [はい] | [はい] | いいえ  | いいえ  | いいえ  |
| データのメモリ内キャッシュ | いいえ  | 可能  | [はい] | いいえ  | [はい] | 

### <a name="security-capabilities"></a>セキュリティ機能
| | Azure Data Lake Analytics | SQL Data Warehouse | Spark を使用する HDInsight | HDInsight 上の Apache Hive | HDInsight 上の Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| 認証  | Azure Active Directory (Azure AD) | SQL / Azure AD | いいえ  | ローカル / Azure AD <sup>1</sup> | ローカル / Azure AD <sup>1</sup> |
| 承認  | [はい] | [はい]| いいえ  | はい <sup>1</sup> | はい <sup>1</sup> |
| 監査  | [はい] | [はい] | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> |
| 保存データの暗号化 | [はい]| はい <sup>2</sup> | [はい] | [はい] | [はい] |
| 行レベルのセキュリティ | いいえ  | [はい] | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> |
| ファイアウォールをサポート | [はい] | [はい] | [はい] | はい <sup>3</sup> | はい <sup>3</sup> |
| 動的データ マスク | いいえ  | いいえ  | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> |

[1] [ドメイン参加済み HDInsight クラスター](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)を使用する必要があります。

[2] 保存データの暗号化と暗号化の解除には、Transparent Data Encryption (TDE) を使用する必要があります。

[3] [Azure Virtual Network 内で使用する](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)場合にサポートされます。
