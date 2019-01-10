---
title: リアルタイム メッセージ取り込みテクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 961e377591f67aec995c8495fa9188c851e464fc
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111242"
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Azure でのリアルタイム メッセージ取り込みテクノロジの選択

リアルタイム処理は、リアルタイムでキャプチャされて最小限の待機時間で処理されるデータ ストリームを処理します。 多くのリアルタイム処理ソリューションには、メッセージのためのバッファーとして機能し、スケールアウト処理、信頼性の高い配信、その他のメッセージ キューのセマンティクスをサポートするメッセージ インジェスト ストアが必要です。

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>リアルタイム メッセージ取り込みのオプション

<!-- markdownlint-enable MD026 -->

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [HDInsight 上の Kafka](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Azure Event Hubs](/azure/event-hubs/) は高度にスケーラブルなデータ ストリーミング プラットフォームであり、毎秒数百万のイベントを受け取って処理できるイベント インジェスト サービスでもあります。 Event Hubs では、分散されたソフトウェアやデバイスから生成されるイベント、データ、またはテレメトリを処理および格納できます。 イベント ハブに送信されたデータは、任意のリアルタイム分析プロバイダーやバッチ処理/ストレージ アダプターを使用して、変換および保存できます。 Event Hubs は大規模で待機時間の短いパブリッシュ/サブスクライブ機能を提供するため、ビッグ データのシナリオに適しています。

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Azure IoT Hub](/azure/iot-hub/) は、何百万もの IoT デバイスとクラウドベースのバックエンド間で、セキュリティで保護された信頼性のある双方向通信を実現する、管理されたサービスです。

IoT Hub の機能は次のとおりです。

- device-to-cloud および cloud-to-device 通信における複数のオプション。 これらのオプションには、一方向メッセージング、ファイル転送、要求/応答メソッドなどがあります。
- 他の Azure サービスへのメッセージ ルーティング。
- デバイス メタデータと同期状態情報用にクエリ実行可能なストア。
- デバイスごとのセキュリティ キーまたは X.509 証明書を使用した、セキュリティ保護された通信とアクセス制御。
- デバイス接続イベントおよびデバイス ID 管理イベントの監視。

メッセージ取り込みの観点からは、IoT Hub は Event Hubs に似ています。 ただし、メッセージ取り込みだけでなく IoT デバイス接続の管理向けに設計されています。 詳しくは、「[Azure IoT Hub と Azure Event Hubs の比較](/azure/iot-hub/iot-hub-compare-event-hubs)」をご覧ください。

## <a name="kafka-on-hdinsight"></a>HDInsight 上の Kafka

[Apache Kafka](https://kafka.apache.org/) はオープン ソースの分散ストリーム プラットフォームで、リアルタイムのデータ パイプラインとストリーミング アプリケーションの構築に使用できます。 Kafka は、名前付きデータ ストリームへの公開および購読ができる、メッセージ キューと同様のメッセージ ブローカー機能も提供しています。 水平方向のスケーラビリティとフォールト トレランスを備え、きわめて高速です。 [HDInsight 上の Kafka](/azure/hdinsight/kafka/apache-kafka-get-started) では、管理された、拡張性の高い、高可用性のサービスとして Kafka が Azure で提供されます。

Kafka の一般的なユース ケースは次のとおりです。

- **メッセージング**。 Kafka はパブリッシュ/サブスクライブのメッセージ パターンをサポートするため、メッセージ ブローカーとしてよく使用されます。
- **アクティビティの追跡**。 Kafka ではレコードの受信順序のログ記録が提供されるため、Web サイトでのユーザー アクションなどのアクティビティの追跡と再現に使用することができます。
- **集計**。 ストリーム処理を使用して異なるストリームからの情報を集計し、情報をまとめて運用データに一元化することができます。
- **変換**。 ストリーム処理を使用して入力された複数のトピックからのデータを結合し、1 つまたは複数の出力トピックに変換することができます。

## <a name="key-selection-criteria"></a>主要な選択条件

選択を絞り込むには、以下の質問に答えることから開始します。

- IoT デバイスと Azure の間の双方向通信が必要ですか。 必要な場合は、IoT Hub を選びます。

- 個々のデバイスのアクセスを管理し、特定のデバイスへのアクセスを取り消せる必要がありますか。 答えが「はい」の場合は、IoT Hub を選びます。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

<!-- markdownlint-disable MD033 -->

| | IoT Hub | Event Hubs | HDInsight 上の Kafka |
| --- | --- | --- | --- |
| クラウドからデバイスへの通信 | [はい] | いいえ  | いいえ  |
| デバイスによって開始されたファイル アップロード | [はい] | いいえ  | いいえ  |
| デバイスの状態情報 | [デバイス ツイン](/azure/iot-hub/iot-hub-devguide-device-twins) | いいえ  | いいえ  |
| プロトコルのサポート | MQTT、AMQP、HTTPS <sup>1</sup> | AMQP、HTTPS | [Kafka プロトコル](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| セキュリティ | デバイス ID ごと。取り消し可能なアクセス制御。 | 共有アクセス ポリシー。パブリッシャー ポリシーを介した限られた取り消し。 | SASL を使用した認証。プラグ可能な承認。サポートされている外部認証サービスとの統合。 |

<!-- markdownlint-enable MD026 -->

[1] [Azure IoT プロトコル ゲートウェイ](/azure/iot-hub/iot-hub-protocol-gateway)をカスタム ゲートウェイとして使用して、IoT Hub に対してプロトコルを適用することもできます。

詳しくは、「[Azure IoT Hub と Azure Event Hubs の比較](/azure/iot-hub/iot-hub-compare-event-hubs)」をご覧ください。
