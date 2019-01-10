---
title: ストリーム処理テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 51129bd09974151ef66a1d660ee3cfa8f79eeb0c
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111514"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Azure でのストリーム処理テクノロジの選択

この記事では、Azure でのリアルタイム ストリーミング処理を行うためのテクノロジの選択肢を比較します。

リアルタイム ストリーム処理では、キューまたはファイル ベースのストレージのメッセージを使用し、メッセージを処理し、結果を別のメッセージ キュー、ファイル ストア、またはデータベースに転送します。 処理には、クエリの実行、フィルター処理、およびメッセージの集計が含まれることがあります。 ストリーム処理エンジンは、終わりのないデータのストリームを使用して、最小限の待機時間で結果を生成できる必要があります。 詳しくは、「[Real time processing](../big-data/real-time-processing.md)」(リアルタイム処理) をご覧ください。

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>リアルタイム処理用のテクノロジを選択する場合の選択肢

<!-- markdownlint-enable MD026 -->

Azure では、以下のすべてのデータ ストアがリアルタイム処理のコア要件を満たしています。

- [Azure Stream Analytics](/azure/stream-analytics/)
- [Spark Streaming を使用する HDInsight](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Azure Databricks における Apache Spark](/azure/azure-databricks/)
- [Storm を使用する HDInsight](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure App Service WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>主要な選択条件

リアルタイム処理のシナリオでは、次の質問に答えることによって、ニーズに適した適切なサービスを選択することから始めます。

- 宣言型または命令型のアプローチでストリーム処理ロジックを作成したいですか。

- テンポラル処理またはウィンドウに対する組み込みのサポートが必要ですか。

- データは、Avro、JSON、または CSV 以外の形式で到着しますか。 「はい」の場合は、カスタム コードを使用して任意の形式をサポートするオプションを検討してください。

- 1 GB/秒を超えるように処理を拡張する必要がありますか。 「はい」の場合は、クラスター サイズに対応するオプションを検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

| | Azure Stream Analytics | Spark Streaming を使用する HDInsight | Azure Databricks における Apache Spark | Storm を使用する HDInsight | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| プログラミング | ストリーム分析クエリ言語、JavaScript | Scala、Python、Java | Scala、Python、Java、R | Java、C# | C#、F#、Node.js | C#、Node.js、PHP、Java、Python |
| プログラミング パラダイム | 宣言型 | 宣言型と命令型の混合 | 宣言型と命令型の混合 | 命令型 | 命令型 | 命令型 |
| 価格モデル | [ストリーミング ユニット](https://azure.microsoft.com/pricing/details/stream-analytics/) | クラスター時間単位 | [Databricks の単位](https://azure.microsoft.com/pricing/details/databricks/) | クラスター時間単位 | 関数の実行とリソースの消費量あたり | App Service プランの時間単位 |  

### <a name="integration-capabilities"></a>統合機能

| | Azure Stream Analytics | Spark Streaming を使用する HDInsight | Azure Databricks における Apache Spark | Storm を使用する HDInsight | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| 入力 | Azure Event Hubs、Azure IoT Hub、Azure Blob ストレージ  | Event Hubs、IoT Hub、Kafka、HDFS、Storage Blobs、Azure Data Lake Store  | Event Hubs、IoT Hub、Kafka、HDFS、Storage Blobs、Azure Data Lake Store  | Event Hubs、IoT Hub、Storage Blobs、Azure Data Lake Store  | [サポートされるバインディング](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus、Storage Queues、Storage Blobs、Event Hubs、WebHooks、Cosmos DB、Files |
| シンク |  Azure Data Lake Store、Azure SQL Database、Storage Blobs、Event Hubs、Power BI、Table Storage、Service Bus Queues、Service Bus Topics、Cosmos DB、Azure Functions  | HDFS、Kafka、Storage Blobs、Azure Data Lake Store、Cosmos DB | HDFS、Kafka、Storage Blobs、Azure Data Lake Store、Cosmos DB | Event Hubs、Service Bus、Kafka | [サポートされるバインディング](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus、Storage Queues、Storage Blobs、Event Hubs、WebHooks、Cosmos DB、Files |

### <a name="processing-capabilities"></a>処理機能

| | Azure Stream Analytics | Spark Streaming を使用する HDInsight | Azure Databricks における Apache Spark | Storm を使用する HDInsight | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- |
| 組み込みのテンポラル/ウィンドウの サポート | [はい] | はい | はい | はい | いいえ  | いいえ  |
| 入力データ形式 | Avro、JSON または CSV、UTF-8 エンコード | カスタム コードを使用する任意の形式 | カスタム コードを使用する任意の形式 | カスタム コードを使用する任意の形式 | カスタム コードを使用する任意の形式 | カスタム コードを使用する任意の形式 |
| スケーラビリティ | [クエリ パーティション](/azure/stream-analytics/stream-analytics-parallelization) | クラスターのサイズによる制限 | Databricks クラスターのスケール構成による制限 | クラスターのサイズによる制限 | 並列処理される最大 200 の関数アプリ インスタンス | アプリ サービス プランの容量による制限 |
| 遅延着信と順不同のイベントの処理 | [はい] | はい | はい | はい | いいえ  | いいえ  |

関連項目:

- [リアルタイム メッセージ取り込みテクノロジの選択](./real-time-ingestion.md)
- [リアルタイム処理](../big-data/real-time-processing.md)
