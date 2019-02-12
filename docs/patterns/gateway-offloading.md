---
title: ゲートウェイ オフロード パターン
titleSuffix: Cloud Design Patterns
description: 共有または専用のサービス機能の負荷をゲートウェイ プロキシにオフロードします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 8be05c30ac974b3e58fb0decc52ab623fc5478c8
ms.sourcegitcommit: eee3a35dd5a5a2f0dc117fa1c30f16d6db213ba2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/06/2019
ms.locfileid: "55782032"
---
# <a name="gateway-offloading-pattern"></a>ゲートウェイ オフロード パターン

共有または専用のサービス機能の負荷をゲートウェイ プロキシにオフロードします。 このパターンは、SSL 証明書の使用などの共有サービス機能を、アプリケーションの他の部分からゲートウェイに移動することで、アプリケーション開発をシンプルにします。

## <a name="context-and-problem"></a>コンテキストと問題

一部の機能は複数のサービスでよく使用されます。こうした機能には、構成、管理、およびメンテナンスが必要です。 すべてのアプリケーションのデプロイで分散される共有または専用のサービスにより、管理オーバーヘッドが増加し、デプロイ エラーの可能性が高くなります。 共有機能に対するすべての更新を、その機能を共有するすべてのサービスにデプロイする必要があります。

セキュリティの問題 (トークンの検証、暗号化、SSL 証明書の管理) とその他の複雑なタスクを適切に処理するには、チーム メンバーに高度な専門スキルが必要です。 たとえば、アプリケーションで必要な証明書は、すべてのアプリケーション インスタンスで構成およびデプロイする必要があります。 新しくデプロイするたびに、証明書の有効期限が切れないように、その証明書を管理する必要があります。 有効期限が切れそうな一般的な証明書は、すべてのアプリケーション デプロイで更新、テスト、および確認する必要があります。

認証、承認、ログ、監視、[調整](./throttling.md)など、他の一般的なサービスについては、デプロイ数が膨大になると、実装および管理するのが困難です。 オーバーヘッドとエラーの可能性を減らすために、この種類の機能は統合する方がよい場合があります。

## <a name="solution"></a>解決策

一部の機能、特に証明書の管理、認証、SSL 終了、監視、プロトコル変換、調整など、分野横断的な懸念事項を、API ゲートウェイにオフロードします。

次の図は、受信 SSL 接続を終了する API ゲートウェイを示しています。 これは、元の要求元に代わって、API ゲートウェイの HTTP サーバー アップ ストリームからデータを要求します。

 ![ゲートウェイ オフロード パターンの図](./_images/gateway-offload.png)

このパターンには次のような利点があります。

- Web サーバー証明書、安全な Web サイトの構成など、関連リソースを配布および管理する必要をなくして、サービス開発をシンプルにします。 構成がシンプルになると、管理が容易になり、スケーラビリティが実現し、サービスのアップグレードも簡単になります。

- 専用チームが、セキュリティなどの専門的知識を必要とする機能を実装できます。 これにより、こうした分野横断的で特殊な懸念事項を、関連するエキスパートに任せることができるため、コア チームはアプリケーション機能に集中できます。

- 要求と応答のログ記録および監視に対してある程度の一貫性を提供します。 サービスが正しくインストルメント化されていなくても、最小限の監視およびログ記録レベルが確保されるように、ゲートウェイを構成できます。

## <a name="issues-and-considerations"></a>問題と注意事項

- API ゲートウェイが高可用性を備え、障害に対する回復性があることを確認します。 API ゲートウェイの複数のインスタンスを実行し、単一障害点をなくします。
- アプリケーションおよびエンドポイントの容量とスケーリングの要件に対応できるようにゲートウェイが設計されていることを確認します。 ゲートウェイがアプリケーションのボトルネックになっていないこと、また、十分にスケーラブルであることを確認します。
- セキュリティ、データ転送など、アプリケーション全体で使用される機能のみをオフロードします。
- ビジネス ロジックは API ゲートウェイにオフロードしないでください。
- トランザクションを追跡する必要がある場合は、ログ記録のために関連付け ID を生成することを検討します。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の状況で使用します。

- アプリケーションのデプロイに、SSL 証明書、暗号化など、共通の懸念事項があるとき。
- アプリケーション デプロイで共通の機能に、さまざまなリソース要件 (メモリ リソース、ストレージ容量、ネットワーク接続など) があるとき。
- ネットワーク セキュリティ、調整、他のネットワーク境界の懸念事項にかかわる問題への対応を、専門チームに任せたいとき。

サービス間の結合が導入されている場合、このパターンは適切でない可能性があります。

## <a name="example"></a>例

次の構成は、Nginx を SSL オフロード アプライアンスとして使用して、受信 SSL 接続を終了し、接続を 3 台のアップストリーム HTTP サーバーのいずれかに分散します。

```console
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

Azure では、[Application Gateway で SSL 終了を設定する](/azure/application-gateway/tutorial-ssl-cli)ことによってこれを実現できます。

## <a name="related-guidance"></a>関連するガイダンス

- [フロントエンド用バックエンド パターン](./backends-for-frontends.md)
- [ゲートウェイ集約パターン](./gateway-aggregation.md)
- [Gateway Routing pattern](./gateway-routing.md) (ゲートウェイ ルーティング パターン)
