---
title: "管理と監視のパターン"
description: "クラウド アプリケーションは、リモートのデータセンターで実行されます。ご自分でそのインフラストラクチャを、または場合によってはオペレーティング システムを、完全に制御することはできません。 このため、オンプレミス デプロイよりも管理と監視は難しくなります。 アプリケーションは、管理者やオペレーターがシステムの管理と監視に使用できるランタイム情報を公開していたり、アプリケーションの停止や再デプロイを必要とせずに、ビジネス要件やカスタマイズの変更をサポートしていたりする必要があります。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6281f7e5c62593cc60727c994d5dba2c52976b75
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="management-and-monitoring-patterns"></a>管理と監視のパターン

クラウド アプリケーションは、リモートのデータセンターで実行されます。ご自分でそのインフラストラクチャを、または場合によってはオペレーティング システムを、完全に制御することはできません。 このため、オンプレミス デプロイよりも管理と監視は難しくなります。 アプリケーションは、管理者やオペレーターがシステムの管理と監視に使用できるランタイム情報を公開していたり、アプリケーションの停止や再デプロイを必要とせずに、ビジネス要件やカスタマイズの変更をサポートしていたりする必要があります。

| パターン | 概要 |
| ------- | ------- |
| [Ambassador](../ambassador.md) | コンシューマー サービスまたはアプリケーションの代わりにネットワーク要求を送信するヘルパー サービスを作成します。 |
| [Anti-Corruption Layer](../anti-corruption-layer.md) | 最新アプリケーションと従来システムの間にファサード、すなわちアダプター レイヤーを実装します。 |
| [External Configuration Store](../external-configuration-store.md) | アプリケーション展開パッケージから、一元管理される場所に構成情報を移動します。 |
| [Gateway Aggregation](../gateway-aggregation.md) | ゲートウェイを使用して、複数の個々の要求を 1 つの要求に集約します。 |
| [Gateway Offloading](../gateway-offloading.md) | 共有または専用のサービス機能の負荷をゲートウェイ プロキシに渡します。 |
| [Gateway Routing](../gateway-routing.md) | 単一のエンドポイントを使用して複数のサービスに要求をルーティングします。 |
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | 公開されたエンドポイントを通じて、外部ツールが定期的にアクセスできる機能チェックをアプリケーションに実装します。 |
| [Sidecar](../sidecar.md) | アプリケーションのコンポーネントを別のプロセスまたはコンテナーにデプロイして、分離性とカプセル化を実現します。 |
| [Strangler](../strangler.md) | 機能の特定の部分を新しいアプリケーションやサービスに徐々に置き換えることで、レガシ システムを段階的に移行します。 |
