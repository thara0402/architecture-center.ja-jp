---
title: メッセージ キューとイベントを使用したエンタープライズ統合
titleSuffix: Azure Reference Architectures
description: Azure Logic Apps、Azure API Management、Azure Service Bus、および Azure Event Grid を使用してエンタープライズ統合パターンを実装するための推奨アーキテクチャ。
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: reference-architecture
ms.date: 12/03/2018
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: integration-services
ms.openlocfilehash: 4c9d2e201bcfc077990d746a1decd55ede2f220a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245513"
---
# <a name="enterprise-integration-on-azure-using-message-queues-and-events"></a>メッセージ キューとイベントを使用した Azure でのエンタープライズ統合

この参照アーキテクチャでは、スケーラビリティと信頼性を高めるためにメッセージ キューとイベントを使用してサービスを切り離し、エンタープライズ バックエンド システムを統合します。 バックエンド システムには、サービスとしてのソフトウェア (SaaS) システム、Azure サービス、およびご自身の企業内の既存の Web サービスが含まれる場合があります。

![キューとイベントを使用したエンタープライズ統合の参照アーキテクチャ](./_images/enterprise-integration-queues-events.png)

## <a name="architecture"></a>アーキテクチャ

ここで示すアーキテクチャは、[基本的なエンタープライズ統合][basic-enterprise-integration]に示されている、よりシンプルなアーキテクチャに基づいています。 そのアーキテクチャで使用されるのは、ワークフローを調整する [Logic Apps][logic-apps] と、API のカタログを作成する [API Management][apim] です。

このバージョンのアーキテクチャでは、システムの信頼性とスケーラビリティを高めるうえで役立つコンポーネントが 2 つ追加されています。

- **[Azure Service Bus][service-bus]**。 Service Bus は、セキュリティで保護された信頼できるメッセージ ブローカーです。

- **[Azure Event Grid][event-grid]**。 Event Grid は、イベント ルーティング サービスです。 このサービスでは、[発行/サブスクライブ](../../patterns/publisher-subscriber.md) (pub/sub) イベント モデルが使用されます。

メッセージ ブローカーを使用した非同期通信は、バックエンド サービスへの直接的な同期呼び出しに比べ、多くの利点があります。

- [キューベースの負荷平準化パターン](../../patterns/queue-based-load-leveling.md)を使用して、負荷平準化を実現し、ワークロードのバーストを処理します。
- 複数のステップまたは複数のアプリケーションを含む、実行時間の長いワークフローの進行状況を確実に追跡します。
- アプリケーションを切り離すうえで役立ちます。
- 既存のメッセージ ベースのシステムと統合します。
- バックエンド システムが利用できない場合に作業をキューに挿入できます。

Event Grid を使うと、システム内のさまざまなコンポーネントが、発生したイベントにすぐに対処できます。ポーリングやスケジュールされたタスクには依存しません。 メッセージ キューと同様、アプリケーションとサービスを切り離すうえでも役立ちます。 アプリケーションまたはサービスがイベントを発行でき、関与しているすべてのサブスクライバーに通知されます。 新しいサブスクライバーを追加するために、送信者を更新する必要はありません。

Event Grid へのイベント送信をサポートする Azure サービスは多数あります。 たとえば、新しいファイルが BLOB ストアに追加された場合に、ロジック アプリはイベントをリッスンできます。 このパターンにより、リアクティブ ワークフローが有効になり、ファイルのアップロードやキューでのメッセージ追加によって一連のプロセスが開始されます。 プロセスは並行して、または特定の順序で実行できます。

## <a name="recommendations"></a>Recommendations

このアーキテクチャには、[基本的なエンタープライズ統合][basic-enterprise-integration]に関するページに記載されている推奨事項が適用されます。 また、次の推奨事項も適用されます。

### <a name="service-bus"></a>Service Bus

Service Bus には、"*プル*" と "*プッシュ*" の 2 つの配信モードがあります。 プル モードでは、受信側は継続的に新しいメッセージをポーリングします。 ポーリングは、特に、それぞれ複数のメッセージを受信するキューが多数ある場合や、メッセージとメッセージの間にかなりの時間がある場合には、非効率になることがあります。 プッシュ モデルでは、新しいメッセージがあるとき、Service Bus が Event Grid を介してイベントを送信します。 受信側は、そのイベントをサブスクライブします。 イベントがトリガーされると、受信側は Service Bus から次のメッセージ バッチをプルします。

Service Bus メッセージを使用するためにロジック アプリを作成する場合は、Event Grid 統合でプッシュ モデルを使用することをお勧めします。 ロジック アプリで Service Bus をポーリングする必要がないため、ほとんどの場合、こちらの方がコスト効率に優れています。 詳細については、「[Azure Service Bus と Event Grid の統合の概要](/azure/service-bus-messaging/service-bus-to-event-grid-integration-concept)」を参照してください。 現在、Event Grid の通知には Service Bus の [Premium レベル](https://azure.microsoft.com/pricing/details/service-bus/)が必要です。

メッセージのグループへのアクセスには、[PeekLock](/azure/service-bus-messaging/service-bus-messaging-overview#queues) を使用してください。 PeekLock を使用すると、ロジック アプリは、メッセージを完了または中止する前に、各メッセージを検証する手順を実行できます。 この方法によって、メッセージの誤損失から保護します。

### <a name="event-grid"></a>Event Grid

Event Grid トリガーが発生した場合、これは、"*少なくとも 1 つ*" のイベントが発生したことを意味します。 たとえば、ロジック アプリが Service Bus メッセージに対する Event Grid トリガーを取得した場合、そのロジック アプリは、処理対象のメッセージを複数取得する可能性があると想定します。

Event Grid では、サーバーレス モデルを使用します。 課金は、操作 (イベントの実行) の回数に基づいて計算されます。 詳細については、「[Event Grid の価格](https://azure.microsoft.com/pricing/details/event-grid/)」を参照してください。 現在、Event Grid には、レベルに関する考慮事項はありません。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

Service Bus Premium レベルは、より高いスケーラビリティを実現するためにメッセージング ユニットの数をスケールアウトできます。 Premium レベルの構成では、1 つ、2 つ、または 4 つのメッセージング ユニットを持つことができます。 Service Bus のスケーリングの詳細については、「[Service Bus メッセージングを使用したパフォーマンス向上のためのベスト プラクティス](/azure/service-bus-messaging/service-bus-performance-improvements)」を参照してください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

各サービスの SLA を確認してください。

- [API Management の SLA][apim-sla]
- [Event Grid の SLA][event-grid-sla]
- [Logic Apps の SLA][logic-apps-sla]
- [Service Bus の SLA][sb-sla]

重大な障害が発生した場合にフェールオーバーを有効にするために、Service Bus Premium で geo ディザスター リカバリーを実装することを検討します。 詳細については、「[Azure Service Bus の geo ディザスター リカバリー](/azure/service-bus-messaging/service-bus-geo-dr)」を参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

Service Bus をセキュリティで保護するには、Shared Access Signature (SAS) を使用します。 [SAS 認証](/azure/service-bus-messaging/service-bus-sas)を使用して、特定の権限で、Service Bus リソースへのアクセス権をユーザーに付与できます。 詳細については、「[Service Bus の認証と承認](/azure/service-bus-messaging/service-bus-authentication-and-authorization)」を参照してください。

Service Bus キューを HTTP エンドポイントとして公開する必要がある場合は (たとえば、新しいメッセージを投稿する場合)、API Management を使用して、エンドポイントをフロントに配置してキューをセキュリティで保護します。 次に、必要に応じて、証明書または OAuth 認証を使用してこのエンドポイントをセキュリティ保護できます。 エンドポイントをセキュリティで保護する最も簡単な方法は、HTTP 要求/応答トリガーを仲介としてロジック アプリを使用することです。

Event Grid サービスは、検証コードを使ってイベント配信をセキュリティで保護します。 Logic Apps を利用してイベントを使用する場合、検証は自動的に行われます。 詳細については、「[Event Grid security and authentication](/azure/event-grid/security-authentication)」(Event Grid のセキュリティと認証) を参照してください。

[apim]: /azure/api-management
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[event-grid]: /azure/event-grid/
[event-grid-sla]: https://azure.microsoft.com/support/legal/sla/event-grid
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[sb-sla]: https://azure.microsoft.com/support/legal/sla/service-bus/
[service-bus]: /azure/service-bus-messaging/
[basic-enterprise-integration]: ./basic-enterprise-integration.md
