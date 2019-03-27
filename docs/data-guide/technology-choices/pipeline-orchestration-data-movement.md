---
title: データ パイプライン オーケストレーション テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 76a101b76497ae2b2aacff973175bb0fe4703d9e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245873"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Azure でのデータ パイプライン オーケストレーション テクノロジの選択

ほとんどのビッグ データ ソリューションは、ワークフローにカプセル化された繰り返されるデータ処理操作で構成されます。 パイプライン オーケストレーターは、これらのワークフローを自動化するのに役立つツールです。 オーケストレーターは、ジョブのスケジュール設定、ワークフローの実行、およびタスク間の依存関係を調整できます。

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>データ パイプライン オーケストレーションのオプションとは

Azure では、次のサービスとツールがパイプライン オーケストレーション、制御フロー、およびデータ移動のコア要件を満たしています。

- [Azure Data Factory](/azure/data-factory/)
- [HDInsight での Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

これらのサービスとツールは、単独で使用することも、一緒に使用してハイブリッド ソリューションを作成することもできます。 たとえば、Azure Data Factory V2 の Integration Runtime (IR) は、管理対象の Azure コンピューティング環境で SSIS パッケージをネイティブに実行できます。 これらのサービスの機能には重複がありますが、大きな違いはほとんどありません。

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- データを移動して変換するためにビッグ データの機能が必要ですか。 通常、これは、数ギガバイトから数テラバイトのデータがあることを意味します。 「はい」の場合は、ビッグ データに最適なものにオプションを絞りです。

- 大規模に操作できる管理対象サービスが必要ですか。 「はい」の場合は、ローカルな処理能力によって制限されないクラウド ベースのサービスのいずれかを選択します。

- データ ソースの一部がオンプレミスに配置されていますか。 「はい」の場合は、クラウドとオンプレミスのデータ ソースまたは変換先の両方で機能できるオプションを探します。

- ソース データは、HDFS ファイル システムの BLOB ストレージに格納されていますか。 該当する場合は、Hive クエリをサポートするオプションを選択します。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

| | Azure Data Factory | SQL Server Integration Services (SSIS) | HDInsight での Oozie
| --- | --- | --- | --- |
| 管理者常駐型 | はい | いいえ  | はい |
| クラウド ベース | はい | いいえ (ローカル) | はい |
| 前提条件 | Azure サブスクリプション | SQL Server  | Azure サブスクリプション、HDInsight クラスター |
| 管理ツール | Azure Portal、PowerShell、CLI、.NET SDK | SSMS、PowerShell | Bash シェル、Oozie REST API、Oozie web UI |
| 価格 | 使用した分を支払う | ライセンス/機能の料金を支払う | HDInsight クラスターでの実行に対する追加料金なし |

### <a name="pipeline-capabilities"></a>パイプラインの機能

| | Azure Data Factory | SQL Server Integration Services (SSIS) | HDInsight での Oozie
| --- | --- | --- | --- |
| データをコピーする | はい | はい | はい |
| カスタム変換 | はい | はい | はい (MapReduce、Pig、および Hive ジョブ) |
| Azure Machine Learning のスコア付け | はい | はい (スクリプト使用) | いいえ  |
| HDInsight On-Demand | はい | いいえ  | いいえ  |
| Azure Batch | はい | いいえ  | いいえ  |
| Pig、Hive、MapReduce | はい | いいえ  | はい |
| Spark | はい | いいえ  | いいえ  |
| SSIS パッケージの実行 | はい | はい | いいえ  |
| 制御フロー | はい | はい | はい |
| オンプレミスのデータへのアクセス | はい | はい | いいえ  |

### <a name="scalability-capabilities"></a>スケーラビリティ機能

| | Azure Data Factory | SQL Server Integration Services (SSIS) | HDInsight での Oozie
| --- | --- | --- | --- |
| スケールアップ | はい | いいえ  | いいえ  |
| スケールアウト | はい | いいえ  | はい (クラスターへの worker ノードの追加) |
| ビッグ データに合わせて最適化 | はい | いいえ  | はい |

