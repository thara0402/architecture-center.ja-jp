---
title: 要求チェック パターン
titleSuffix: Cloud Design Patterns
description: 大きいメッセージを要求チェックとペイロードに分割して、メッセージ バスに過度な負荷がかかることを防ぎます。
keywords: 設計パターン
author: yorek
ms.date: 03/05/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 88457983df4ea3fd4ed8d0c447de6302422339f8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58248867"
---
# <a name="claim-check-pattern"></a>要求チェック パターン

大きいメッセージを要求チェックとペイロードに分割します。 要求チェックはメッセージング プラットフォームに送信し、ペイロードは外部サービスに格納します。 このパターンを使用して大きいメッセージを処理すると、メッセージ バスやクライアントで大きな負荷や速度の低下が発生するのを防ぐことができます。 また、通常はストレージの方がメッセージング プラットフォームで使用されるリソース ユニットより安いので、このパターンはコストの削減にも役立ちます。

このパターンは参照ベースのメッセージングとも呼ばれ、もともとは Gregor Hohpe と Bobby Woolf による本『*Enterprise Integration Patterns*』(エンタープライズ統合パターン) で[説明][enterprise-integration-patterns]されたものです。

## <a name="context-and-problem"></a>コンテキストと問題

メッセージング ベースのアーキテクチャは、いつかは大きいメッセージを送信、受信、操作できるようになる必要があります。 このようなメッセージには、画像 (MRI スキャンなど)、サウンド ファイル (コール センターの呼び出しなど)、テキスト ドキュメント、任意のサイズのあらゆる種類のバイナリ データなど、何でも含まれる可能性があります。

このような大きいメッセージをメッセージ バスに直接送信することは、より多くのリソースと帯域幅を消費する必要があるため、推奨されません。 また、通常、メッセージング プラットフォームは多数の小さいメッセージを処理するように微調整されているため、大きいメッセージではソリューション全体が遅くなることがあります。 また、ほとんどのメッセージング プラットフォームにはメッセージ サイズに制限があるので、大きいメッセージではこれらの制限を回避する必要もあります。

## <a name="solution"></a>解決策

メッセージのペイロード全体を、データベースなどの外部サービスに格納します。 格納されているペイロードへの参照を取得し、メッセージ バスにはその参照だけを送信します。 参照は、荷物を受け取るための預かり証のように、パターンの名前として機能します。 その特定のメッセージの処理に関心のあるクライアントは、必要に応じて、入手した参照を使用してペイロードを取得できます。

![要求チェック パターンの図](./_images/claim-check.jpg)

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

- メッセージをアーカイブする必要がない場合は、使用後にメッセージ データを削除することを検討します。 BLOB ストレージは比較的安価ですが、期間が長くなれば、特に大量のデータがある場合、ある程度のコストがかかります。 メッセージの削除は、メッセージを受信して処理するアプリケーションによって同期的に、または独立した専用プロセスによって非同期的に、実行できます。 非同期的な方法では、受信側アプリケーションのスループットとメッセージ処理パフォーマンスに影響を与えずに、古いデータが削除されます。

- メッセージの格納と取得では、若干のオーバーヘッドと待機時間が余分に発生します。 メッセージのサイズがメッセージ バスのデータ制限を超えた場合にのみこのパターンを使用するロジックを、送信側アプリケーションに実装することができます。 小さいメッセージの場合はパターンはスキップされます。 このアプローチでは、条件付きの要求チェック パターンになります。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

選択されているメッセージ バス テクノロジでサポートされているメッセージ制限にメッセージが適合できない場合は常に、このパターンを使用する必要があります。 たとえば、現在、Event Hubs には 256 KB (Basic レベル) の制限がありますが、Event Grid では 64 KB のメッセージしかサポートされていません。

読み取りを許可されたサービスだけがペイロードにアクセスできるようにする必要がある場合も、パターンを使用できます。 ペイロードを外部リソースにオフロードすることで、より厳密な認証と承認の規則を実施し、ペイロードに機密データが格納されている場合にセキュリティを適用することができます。

## <a name="examples"></a>例

Azure では、さまざまな方法とテクノロジを使用してこのパターンを実装することができますが、2 つの主なカテゴリがあります。 どちらの場合も、受信側が、要求チェックを読み取り、それを使用してペイロードを取得する責任を持ちます。

- **要求チェックの自動生成**。 この方法では、[Azure Event Grid](/azure/event-grid/) を使用して、自動的に要求チェックを生成し、メッセージ バスにプッシュします。

- **要求チェックの手動生成**。 この方法では、ペイロードを管理する責任は送信側にあります。 送信側は、適切なサービスを使用してペイロードを格納し、要求チェックを取得または生成して、メッセージ バスに要求チェックを送信します。

Event Grid はイベント ルーティング サービスであり、最大 24 時間の構成可能な時間内にイベントの配信が試みられます。 その後、イベントは破棄されるか配信不能にされます。 イベント ペイロードをアーカイブしたり、イベント ストリームを再生したりする必要がある場合は、Event Grid サブスクリプションを Event Hubs または Queue storage に追加できます。そこでは、メッセージを長期間保持でき、メッセージのアーカイブがサポートされています。 Event Grid メッセージの配信と再試行および配信不能の構成の微調整については、「[配信不能と再試行に関する方針](/azure/event-grid/manage-event-delivery)」をご覧ください。

### <a name="automatic-claim-check-generation-with-blob-storage-and-event-grid"></a>Blob Storage と Event Grid による要求チェックの自動生成

この方法では、送信側が指定された Azure Blob Storage コンテナーにメッセージ ペイロードをドロップします。 Event Grid により、自動的にタグ/参照が生成されて、Azure Storage キューなどのサポートされているメッセージ バスに送信されます。 受信側は、キューをポーリングし、メッセージを取得した後、格納されている参照データを使用して Blob Storage から直接ペイロードをダウンロードできます。

同じ Event Grid メッセージを、[Azure Functions](/azure/azure-functions/) で直接使用できます。メッセージ バスを経由する必要はありません。 この方法では、Event Grid と Functions 両方のサーバーレスの性質を最大限に利用します。

この方法のコードの例は、[こちら][example-1]で見つかります。

### <a name="event-grid-with-event-hubs"></a>Event Grid と Event Hubs

前の例と同様に、ペイロードが Azure BLOB コンテナーに書き込まれると、Event Grid によって自動的にメッセージが生成されます。 ただし、この例では、メッセージ バスは Event Hubs を使って実装されます。 クライアントは、それ自体を登録して、イベント ハブに書き込まれたメッセージのストリームを受信できます。 イベント ハブは、メッセージをアーカイブし、Apache Spark、Apache Drill、または利用可能な任意の Avro ライブラリなどのツールを使用してクエリ可能な Avro ファイルとして使用できるようにするように、構成することもできます。

この方法のコードの例は、[こちら][example-2]で見つかります。

### <a name="claim-check-generation-with-service-bus"></a>Service Bus での要求チェックの生成

このソリューションでは特定の Service Bus プラグイン [ServiceBus.AttachmentPlugin](https://www.nuget.org/packages/ServiceBus.AttachmentPlugin/) を利用することにより、要求チェック ワークフローの実装が容易になります。 プラグインでは、メッセージの本文が、メッセージ送信時に Azure Blob Storage に格納される添付ファイルに変換されます。

```csharp
using ServiceBus.AttachmentPlugin;
...

// Getting connection information
var serviceBusConnectionString = Environment.GetEnvironmentVariable("SERVICE_BUS_CONNECTION_STRING");
var queueName = Environment.GetEnvironmentVariable("QUEUE_NAME");
var storageConnectionString = Environment.GetEnvironmentVariable("STORAGE_CONNECTION_STRING");

// Creating config for sending message
var config = new AzureStorageAttachmentConfiguration(storageConnectionString);

// Creating and registering the sender using Service Bus Connection String and Queue Name
var sender = new MessageSender(serviceBusConnectionString, queueName);
sender.RegisterAzureStorageAttachmentPlugin(config);

// Create payload
var payload = new { data = "random data string for testing" };
var serialized = JsonConvert.SerializeObject(payload);
var payloadAsBytes = Encoding.UTF8.GetBytes(serialized);
var message = new Message(payloadAsBytes);

// Send the message
await sender.SendAsync(message);
```

Service Bus のメッセージは、クライアントがサブスクライブできる通知キューとして機能します。 コンシューマーがメッセージを受信すると、プラグインによって Blob Storage から直接メッセージ データを読み取ることができるようになります。 その後、さらにメッセージを処理する方法をお客様が選択できます。 この方法の利点は、要求チェックのワークフローが送信側と受信側から抽象化されることです。

この方法のコードの例は、[こちら][example-3]で見つかります。

### <a name="manual-claim-check-generation-with-kafka"></a>Kafka での要求チェックの手動生成

この例では、Kafka クライアントによって、ペイロードが Azure Blob Storage に書き込まれます。 その後、[Kakfa 対応の Event Hubs](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs) を使用して通知メッセージが送信されます。 コンシューマーはメッセージを受信し、Blob Storage からペイロードにアクセスできます。 この例では、異なるメッセージング プロトコルを使用して、Azure に要求チェック パターンを実装する方法を示します。 たとえば、既存の Kafka クライアントをサポートする必要があるような場合です。

この方法のコードの例は、[こちら][example-4]で見つかります。

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

- 上で説明した例は、[GitHub][sample-code] で入手できます。
- エンタープライズ統合パターンのサイトには、このパターンの[説明][enterprise-integration-patterns]があります。
- 別の例については、「[Dealing with large Service Bus messages using claim check pattern (要求チェック パターンを使用する大きな Service Bus メッセージの処理)](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern)」(ブログの投稿) をご覧ください。
- 大きいメッセージを処理するための別のパターンとしては、[分割][ splitter]と[集約][aggregator]があります。

<!-- links -->
[aggregator]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Aggregator.html
[enterprise-integration-patterns]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html
[example-1]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-1
[example-2]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-2
[example-3]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-3
[example-4]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-4
[sample-code]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check
[splitter]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html
