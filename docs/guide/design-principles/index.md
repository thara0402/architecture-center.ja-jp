---
title: Azure アプリケーションの設計原則
description: Azure アプリケーションの設計原則
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206806"
---
# <a name="design-principles-for-azure-applications"></a>Azure アプリケーションの設計原則

次の設計原則に従って、アプリケーションのスケーラビリティを上げて、回復力や管理しやすさを強化します。 

**[自動修復機能を設計します](self-healing.md)**。 分散システムでは障害が発生します。 障害の発生に備えてアプリケーションの自動修復機能を設計します。

**[すべての要素を冗長にします](redundancy.md)**。 単一障害点をなくすようにアプリケーションに冗長性を組み込みます。
 
**[調整を最小限に抑えます](minimize-coordination.md)**。 アプリケーション サービス間の調整を最小限に抑えてスケーラビリティを実現します。
 
**[スケール アウトするように設計します](scale-out.md)**。需要に応じて新規インスタンスを追加または削除して水平方向に拡張できるようにアプリケーションを設計します。

**[制限に対処するようにパーティション化します](partition.md)**。 パーティション分割を使用して、データベース、ネットワーク、コンピューティングの制限に対処します。

**[操作に合わせて設計します](design-for-operations.md)**。 運用チームが必要なツールを得られるようにアプリケーションを設計します。

**[管理対象サービスを使用します](managed-services.md)**。 可能であればサービスとしてのインフラストラクチャ (IaaS) ではなくサービスとしてのプラットフォーム (PaaS) を使用します。

**[ジョブに最適なデータ ストアを使用します](use-the-best-data-store.md)**。 データに最適なストレージ技術とその使用方法を選択します。 
 
**[展開を見込んで設計します](design-for-evolution.md)**。 成功するすべてのアプリケーションは時間の経過と共に変化します。 展開を見込んだ設計は継続的な技術革新のキーです。

**[ビジネスのニーズに合わせて構築します](build-for-business.md)**。 設計の決定はすべてビジネス要件によって正当化される必要があります。

