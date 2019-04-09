---
title: Azure Kubernetes Service のマイクロサービス リファレンス実装
description: このリファレンス実装では、マイクロサービス アーキテクチャのベスト プラクティスについて説明します
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 17e275e5b5f45233f7467192402cb28fce35c57b
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068907"
---
# <a name="designing-a-microservices-architecture"></a>マイクロサービス アーキテクチャの設計

現在、マイクロサービスは、回復性優れ、単独でのデプロイが可能で、迅速に展開できるスケーラブルなクラウド アプリケーションを構築するための一般的なアーキテクチャ スタイルになっています。 しかし、このマイクロサービスを、単なる業界用語に留めないためには、アプリケーションを設計および構築するのためのさまざまなアプローチが必要です。

この一連の記事では、Azure でマイクロサービス アーキテクチャを構築して実行する方法について説明します。 取り上げるトピックは次のとおりです。

- [サービス間の通信](./interservice-communication.md)
- [API 設計](./api-design.md)
- [API ゲートウェイ](./gateway.md)
- [データに関する考慮事項](./data-considerations.md)
- [設計パターン](./patterns.md)

## <a name="prerequisites"></a>前提条件

これらの記事を読む前に、まず以下をご覧ください。

- [マイクロサービス アーキテクチャの概要](../introduction.md)。 マイクロサービスのメリットと課題のほか、このスタイルのアーキテクチャを使用する場面について確認します。
- [ドメイン分析を使用したマイクロサービスのモデル化](../model/domain-analysis.md)。 マイクロサービスをモデル化するためのドメイン駆動のアプローチについて学習します。

## <a name="reference-implementation"></a>リファレンス実装

マイクロサービス アーキテクチャのベスト プラクティスを説明するために、ドローン配送アプリケーションというリファレンス実装を作成しました。 この実装は、Azure Kubernetes Service (AKS) を使用して Kubernetes 上で動作します。 リファレンス実装は [GitHub][drone-ri] にあります。

![ドローン配送アプリケーションのアーキテクチャ](../images/drone-delivery.png)

## <a name="scenario"></a>シナリオ

Fabrikam, Inc. は、ドローン配送サービスを開始しようとしています。 同社は、ドローン機団を管理しています。 企業がサービスに登録すると、ユーザーは、ドローンで商品を集荷して配送するように依頼できます。 顧客が集荷のスケジュールを設定すると、バックエンド システムによってドローンが割り当てられ、推定配送時刻がユーザーに通知されます。 配送中、ETA は常時更新され、顧客はドローンの場所を追跡できます。

このシナリオには、かなり複雑なドメインが含まれます。 ビジネスに関する懸念事項には、ドローンのスケジュール設定、荷物の追跡、ユーザー アカウントの管理、履歴データの保存と分析などもあります。 さらに、Fabrikam が求めているのは、迅速に市場に出し、迅速に反復して、新機能を追加することです。 アプリケーションは、高いサービス レベル目標 (SLO) に基づいてクラウド スケールで動作する必要があります。 また、Fabrikam は、システムの部分ごとに、データ ストレージとクエリの要件がまったく異なることを予期しています。 Fabrikam はこれらをすべて考慮した結果、ドローン配送アプリケーションにマイクロサービス アーキテクチャを選択します。

> [!NOTE]
> マイクロサービス アーキテクチャと他のアーキテクチャ スタイルのどちらを選択するかについては、「[Azure アプリケーション アーキテクチャ ガイド](../../guide/index.md)」を参照してください。

このリファレンス実装では、[Azure Kubernetes Service (AKS)](/azure/aks/) で Kubernetes を使用しています。 ただし、高いレベルでのアーキテクチャの決定と課題の多くは、[Azure Service Fabric](/azure/service-fabric/) を含む、すべてのコンテナー オーケストレーターに当てはまります。

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [コンピューティング オプションの選択](./compute-options.md)