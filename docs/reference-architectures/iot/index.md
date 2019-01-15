---
title: Azure IoT 参照アーキテクチャ
description: PaaS (サービスとしてのプラットフォーム) コンポーネントを使用する Azure の IoT アプリケーションの推奨アーキテクチャ
titleSuffix: Azure Reference Architectures
author: MikeWasson
ms.date: 01/09/2019
ms.openlocfilehash: bde507527262a722219edba534275fb7ab499069
ms.sourcegitcommit: 7d9efe716e8c9e99f3fafa9d0213d48c23d9713d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2019
ms.locfileid: "54166022"
---
# <a name="azure-iot-reference-architecture"></a>Azure IoT 参照アーキテクチャ

この参照アーキテクチャは、PaaS (サービスとしてのプラットフォーム) コンポーネントを使用する Azure の IoT アプリケーションの推奨アーキテクチャを示しています。

![アーキテクチャの図](./_images/iot.png)

IoT アプリケーションは、**分析情報**を生成するデータを送信する**もの** (デバイス) として説明することができます。 これらの分析情報によりビジネスやプロセスを改善するための**アクション**が生成されます。 温度データを送信するエンジン (もの) はその一例です。 このデータは、エンジンが想定どおりに実行されているかどうかを評価するために使用されます (分析情報)。 分析情報は、エンジンのメンテナンス スケジュールを事前に優先順位付けするのに使用されます (アクション)。

この参照アーキテクチャでは、Azure PaaS (サービスとしてのプラットフォーム) コンポーネントを使用します。 Azure で IoT ソリューションをビルドするためのその他のオプションには、次のものがあります。

- [Azure IoT Central](/azure/iot-central/)。 IoT Central は、フル マネージド SaaS (サービスとしてのソフトウェア) ソリューションです。 技術的な選択肢を抽象化することで、自分のソリューションだけに注力できるようにします。 この簡素化には、PaaS ベースのソリューションと比べてカスタマイズしにくいというトレードオフが伴います。
- Azure VM にデプロイされた SMACK スタック (Spark、Mesos、Akka、Cassandra、Kafka) などの OSS コンポーネントの使用。 この方法では、多くの制御が可能ですが、より複雑になります。

テレメトリ データの処理方法には、大まかに言ってホット パスとコールド パスの 2 種類があります。 違いは、待機時間とデータ アクセスの要件にあります。

- **ホット パス**は、データが到着すると、ほぼリアルタイムでそのデータを分析します。 ホット パスでは、非常に短い待機時間でテレメトリを処理する必要があります。 ホット パスは通常、ストリーム処理エンジンを使用して実装されます。 出力は、アラートをトリガーしたり、分析ツールを使用してクエリ可能な構造化された形式に書き込むことができます。
- **コールド パス**は、より長い間隔 (毎時または毎日) でバッチ処理を実行します。 コールド パスは通常、大量のデータで動作しますが、結果は、ホット パスほどタイムリーである必要はありません。 コールド パスでは、未加工のテレメトリがキャプチャされ、バッチ処理にフィードされます。

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、次のコンポーネントで構成されます。 アプリケーションの中には、以下に一覧表示するすべてのコンポーネントを必要としないものもあります。

**IoT デバイス**。 デバイスはクラウドに安全に登録でき、クラウドに接続してデータを送受信できます。 一部のデバイスは、デバイス自体でまたはフィールド ゲートウェイで一部のデータ処理を実行する**エッジ デバイス**の場合があります。 エッジ処理には、[Azure IoT Edge](/azure/iot-edge/) をお勧めします。

**クラウド ゲートウェイ**。 クラウド ゲートウェイは、デバイスがクラウドに安全に接続してデータを送信するためのクラウド ハブを提供します。 デバイスの管理や、コマンドやデバイスの制御などの機能も提供します。 クラウド ゲートウェイには、[IoT Hub](/azure/iot-hub/) をお勧めします。 IoT Hub は、デバイスからイベントを取り込むホスト型クラウド サービスで、デバイスとバックエンド サービス間で分割されるメッセージとして機能します。 IoT Hub は、セキュリティで保護された接続、イベント インジェスト、双方向通信、およびデバイス管理を提供します。

**デバイス プロビジョニング**。 多数のデバイスの登録と接続には、[IoT Hub Device Provisioning Service](/azure/iot-dps/) (DPS) の使用をお勧めします。 DPS を使用すると、デバイスを特定の Azure IoT Hub エンドポイントに大規模に割り当てて登録することができます。

**ストリーム処理**。 ストリーム処理は、データ レコードの大規模なストリームを分析し、これらのストリームの規則を評価します。 ストリーム処理には、[Azure Stream Analytics](/azure/stream-analytics/) をお勧めします。 Stream Analytics では、time windowing 関数、ストリーム集計、外部データ ソース結合を使用して、大規模で複雑な分析を実行できます。 もう 1 つのオプションは、[Azure Databricks](/azure/azure-databricks/) の Apache Spark です。

機械学習では、履歴テレメトリ データに対して予測アルゴリズムを実行できるようにすることで、予測メンテナンスなどのシナリオを可能にします。 機械学習には、[Azure Machine Learning Service](/azure/machine-learning/service/) をお勧めします。

**ウォーム パス ストレージ**には、レポートと視覚化のためにデバイスからすぐに使用できる必要があるデータが保持されます。 ウォーム パス ストレージには、[Cosmos DB](/azure/cosmos-db/introduction) をお勧めします。 Cosmos DB は、グローバル分散型のマルチモデル データベースです。

**コールド パス ストレージ**には、長期間保持され、バッチ処理に使用されるデータが保持されます。 コールド パス ストレージには、[Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) をお勧めします。 データは、低コストで無期限に Blob Storage にアーカイブすることができ、バッチ処理のために簡単にアクセスできます。

**データ変換**は、テレメトリ ストリームを操作または集計します。 例には、バイナリ データの JSON への変換などのプロトコル変換や、データ ポイントの結合が含まれます。 IoT Hub に到達する前にデータを変換する必要がある場合は、[プロトコル ゲートウェイ](/azure/iot-hub/iot-hub-protocol-gateway) (示されていません) の使用をお勧めします。 そうでない場合、データは IoT Hub に到達した後で変換できます。 その場合は、[Azure Functions](/azure/azure-functions/) の使用をお勧めします。 Functions には、IoT Hub、Cosmos DB、および Blob Storage とのビルトイン統合があります。

**ビジネス プロセス統合**は、デバイス データからの分析情報に基づいてアクションを実行します。 これには、情報メッセージの格納、アラームの発生、電子メールまたは SMS メッセージの送信、CRM との統合などが含まれる場合があります。 ビジネス プロセス統合には、[Azure Logic Apps](/azure/logic-apps/logic-apps-overview) の使用をお勧めします。

**ユーザー管理**は、デバイスでファームウェアのアップグレードなどのアクションを実行できるユーザーまたはグループを制限します。 アプリケーションでユーザーが利用できる機能も定義します。 ユーザーの認証と承認には、[Azure Active Directory](/azure/active-directory/) の使用をお勧めします。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

IoT アプリケーションは、個別にスケールできる個別のサービスとしてビルドする必要があります。 次のスケーラビリティ ポイントを検討してください。

**IoT Hub**。 IoT Hub については、次のスケール係数を検討してください。

- IoT Hub に送信されるメッセージの[日単位の最大クォータ](/azure/iot-hub/iot-hub-devguide-quotas-throttling)。
- IoT Hub インスタンスで接続されたデバイスのクォータ。
- インジェストのスループット &mdash; IoT Hub がメッセージを取り込むことができる速度。
- 処理スループット &mdash; 受信メッセージの処理速度。

各 IoT Hub は、特定のレベルのユニット数でプロビジョニングされます。 レベルとユニット数により、デバイスがハブに送信できるメッセージの 1 日あたりの最大クォータが決定されます。 詳細については、IoT Hub のクォータと調整に関するページを参照してください。 既存の操作を中断することなく、ハブをスケール アップすることができます。

**Stream Analytics**。 Stream Analytics ジョブは、クエリへの入力から出力まで、Stream Analytics パイプラインのすべてのポイントで並列化されると最適にスケールされます。 完全な並列ジョブにより、Stream Analytics は複数のコンピューティング ノード間で作業を分割することができます。 それ以外の場合、Stream Analytics はストリーム データを 1 つの場所に結合する必要があります。 詳細については、「[Azure Stream Analytics でのクエリの並列処理の活用](/azure/stream-analytics/stream-analytics-parallelization)」を参照してください。

デバイス ID に基づいて、IoT Hub によりデバイス メッセージが自動的にパーティション分割されます。 特定のデバイスからのメッセージはすべて、常に同じパーティションに送信されますが、1 つのパーティションが複数のデバイスからのメッセージを受け取ります。 そのため、並列処理の単位はパーティション ID です。

**Functions**。 Event Hubs エンドポイントから読み取る場合、イベント ハブ パーティションごとに関数のインスタンスの最大値があります。 最大処理レートは、1 つの関数インスタンスが 1 つのパーティションからイベントを処理できる速度によって決まります。 この関数は、メッセージをバッチで処理する必要があります。

**Cosmos DB**。 Cosmos DB コレクションをスケール アウトするには、パーティション キーを使用してコレクションを作成し、記述する各ドキュメントにそのパーティション キーを含めます。 詳細については、[パーティション キーを選択する際のベスト プラクティス](/azure/cosmos-db/partition-data#best-practices-when-choosing-a-partition-key)に関するページを参照してください。

- デバイスごとに 1 つのドキュメントを格納して更新する場合、デバイス ID が適切なパーティション キーとなります。 書き込みはキー間で均等に分散されます。 キー値ごとに 1 つのドキュメントが存在するため、各パーティションのサイズは厳密にバインドされます。
- すべてのデバイス メッセージに対して個別のドキュメントを保存する場合、パーティション キーとしてデバイス ID を使用すると、パーティションあたり 10 GB の制限をすぐに超えてしまいます。 その場合は、メッセージ ID の方が適切なパーティション キーとなります。 通常、インデックス作成とクエリのため、ドキュメントにデバイス ID を含めることもできます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

### <a name="trustworthy-and-secure-communication"></a>信頼できるセキュリティで保護された通信

デバイスと送受信するすべての情報は、信頼できる必要があります。 デバイスで次の暗号化機能をサポートできないのであれば、そのデバイスをローカル ネットワークに制限し、すべてのネットワーク間通信をフィールド ゲートウェイを経由して行う必要があります。

- 証明可能な安全性を備え、公的に分析され、広く実装されている対称キー暗号化アルゴリズムを使用したデータの暗号化。
- 証明可能な安全性を備え、公的に分析され、広く実装されていいる対称キー署名アルゴリズムを使用したデジタル署名。
- TCP またはその他のストリーム ベースの通信パス向けの TLS 1.2、またはデータグラム ベースの通信パス向けの DTLS 1.2 のいずれかのサポート。 X.509 証明書の処理のサポートはオプションで、より計算効率が高く転送効率の高い TLS の事前共有キー モードで置き換えることができます。このモードは、AES および SHA-2 のアルゴリズムをサポートするために実装できます。
- 更新可能なキー ストアとデバイスごとのキー。 各デバイスには、システムに対して認証を行う一意のキー マテリアルまたはトークンが必要です。 デバイスは、(セキュリティで保護されたキー ストアを使用するなどして) キーを安全に保存する必要があります。 デバイスは、キーまたはトークンを定期的、またはシステムのセキュリティ侵害などの緊急の状況では事後的に更新できる必要があります。
- デバイスのファームウェアとアプリケーション ソフトウェアは、更新プログラムが検出されたセキュリティの脆弱性を修復できるようする必要があります。

ただし、多くのデバイスでは、制約が多すぎてこれらの要件をサポートできません。 その場合は、フィールド ゲートウェイを使用する必要があります。 デバイスは、ローカル エリア ネットワーク経由でフィールド ゲートウェイに安全に接続し、ゲートウェイはクラウドへのセキュリティで保護された通信を有効にします。

### <a name="physical-tamper-proofing"></a>物理的な改ざん防止

セキュリティの整合性とシステム全体の信頼性を確保するため、デバイスの設計に物理操作の試行を防御する機能を組み込むことを強くお勧めします。

例: 

- セキュリティで保護されたストレージを備え、トラステッド プラットフォーム モジュール (TPM) の統合などの暗号化キー マテリアルを使用する、マイクロコントローラー/マイクロプロセッサまたは補助ハードウェアを選択します。
- TPM に固定されている、セキュリティで保護されたブート ローダーとセキュリティで保護されたソフトウェアの読み込み。
- センサーを使用して、侵入の試みや、アラートを生成したりデバイスの "デジタル自己破壊" を引き起こす可能性があるデバイス環境の操作を検出します。

セキュリティに関するその他の考慮事項については、「[モノのインターネット (IoT) のセキュリティ アーキテクチャ](/azure/iot-fundamentals/iot-security-architecture)」を参照してください。

### <a name="monitoring-and-logging"></a>監視およびログ記録

ログ記録と監視システムは、ソリューションが機能しているかどうかを判断するため、および問題のトラブルシューティングのために使用されます。 監視とログ記録システムは、次の運用上の質問に答えるときに役立ちます。

- デバイスまたはシステムはエラー状態か。
- デバイスまたはシステムは正しく構成されているか。
- デバイスまたはシステムは正確なデータを生成しているか。
- システムは企業の顧客とエンド カスタマーの両方の期待を満たしているか。

ログ記録と監視ツールは通常、次の 4 つの構成要素で構成されます。

- システムを監視し、基本的なトラブルシューティングを行うためのシステム パフォーマンスとタイムライン視覚化ツール。
- ログ データをバッファーするバッファー処理されたデータ インジェスト。
- ログ データを格納する永続化ストア。
- 詳細なトラブルシューティングで使用するためにログ データを表示する検索とクエリ機能。

監視システムは、正常性、セキュリティ、および安定性に関する分析情報と、IoT ソリューションのパフォーマンスを提供します。 これらのシステムは、より詳細なビューも提供できます。コンポーネントの構成変更を記録し、潜在的なセキュリティの脆弱性を明らかにし、インシデント管理プロセスを強化し、システムの所有者が問題をトラブルシューティングするのに役立つ、抽出したログ データを提供します。 包括的な監視ソリューションには、特定のサブシステムの情報や複数のサブシステムの集計に対してクエリを実行する機能が含まれます。

監視システムの開発は、正常な動作、規制に対するコンプライアンス、監査要件を定義することから始める必要があります。 収集されるメトリックには、次のものが含まれます。

- 構成の変更を報告している物理デバイス、エッジ デバイス、およびインフラストラクチャ コンポーネント。
- 構成の変更を報告しているアプリケーション、セキュリティ監査ログ、要求レート、応答時間、エラー率、およびマネージド言語のガベージ コレクションの統計情報。
- データベース、永続化ストア、クエリと書き込みのパフォーマンスを報告しているキャッシュ、スキーマの変更、セキュリティ監査ログ、ロックまたはデッドロック、インデックスのパフォーマンス、CPU、メモリ、ディスク使用量。
- 正常性メトリックと依存システムの正常性とパフォーマンスに影響を与える構成の変更を報告しているマネージド サービス (IaaS、PaaS、SaaS、FaaS)。

監視メトリックを視覚化することで、システムが不安定であることをオペレーターに警告し、インシデント対応を促します。

### <a name="tracing-telemetry"></a>テレメトリのトレース

テレメトリをトレースすることで、オペレーターはシステム全体で作成時からテレメトリを追跡することができます。 トレースは、デバッグとトラブルシューティングにとって重要です。 Azure IoT Hub および [IoT Hub Device SDK](/azure/iot-hub/iot-hub-devguide-sdks) を使用する IoT ソリューションの場合、データグラムのトレースは、クラウドからデバイスへのメッセージとして始めることができ、テレメトリ ストリームに含めることができます。

### <a name="logging"></a>ログの記録

ログ記録システムは、ソリューションが実行したアクションまたはアクティビティの内容、発生したエラーを把握するうえで不可欠で、こうしたエラーを修正するのにも役立ちます。 ログは分析して、エラー状態の理解と修正、パフォーマンス特性の強化、および適用される規則や法令の順守に役立てることができます。

プレーン テキスト ログは、開発の初期コストへの影響はあまりありませんが、マシンでの解析/読み取りがより困難になります。 収集された情報はマシンで解析可能で人間が判読できるため、構造化ログを使用することをお勧めします。 構造化ログにより、状況コンテキストとメタデータがログ情報に追加されます。 構造化ログでは、プロパティは、検索およびクエリ機能を強化するために、キー/値のペアとしてまたは固定スキーマで書式設定された重要な要素です。

## <a name="next-steps"></a>次の手順

- 推奨されるアーキテクチャと実装の選択の詳細については、「[Microsoft Azure IoT Reference Architecture](http://aka.ms/iotrefarchitecture)」 (Microsoft Azure IoT 参照アーキテクチャ) (PDF) を参照してください。

- 各種 Azure IoT サービスの詳細なドキュメントについては、「[Azure IoT の基礎](/azure/iot-fundamentals/)」を参照してください。