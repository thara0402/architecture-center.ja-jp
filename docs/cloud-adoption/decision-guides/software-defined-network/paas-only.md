---
title: CAF:ソフトウェア定義ネットワーク - PaaS のみ
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ベースのネットワーク機能のための PaaS のみモデルについて説明します
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901726"
---
# <a name="software-defined-networks-paas-only"></a>ソフトウェア定義ネットワーク:PaaS のみ

サービスとしてのプラットフォーム (PaaS) リソースを実装すると、デプロイ プロセスによって、負荷分散、ポートのブロック、他の PaaS サービスへの接続など、そのネットワークに対する少数の制御を持つ仮の基礎ネットワークが自動的に作成されます。

Azure では、いくつかの PaaS リソースの種類を仮想ネットワークに[デプロイ](/azure/virtual-network/virtual-network-for-azure-services)または[接続](/azure/virtual-network/virtual-network-service-endpoints-overview)して、このようなリソースを既存の仮想ネットワーク インフラストラクチャと統合できます。 しかし、PaaS リソースがネイティブで提供するこのような既定のネットワーク機能のみに依存する PaaS のみのネットワーク アーキテクチャは、多くの場合、ワークロード要件を満たすのに十分です。

PaaS のみのネットワーク アーキテクチャを検討している場合は、必要な前提条件がお客様の要件と一致していることを確認してください。

## <a name="paas-only-assumptions"></a>PaaS のみの前提条件

PaaS のみのネットワーク アーキテクチャのデプロイは、以下を前提としています。

- デプロイされるアプリケーションはスタンドアロン アプリケーションであるか、他の PaaS リソースにのみ依存しています。
- お客様の IT 運用チームは、スタンドアロンの PaaS アプリケーションの管理、構成、デプロイをサポートするためにツール、トレーニング、プロセスを更新できます。
- PaaS アプリケーションは、IaaS リソースを含む、より広範なクラウド移行作業には含まれません。

このような前提条件は、PaaS のみのネットワークのデプロイに合わせた最低限の資格条件です。 この方法は単一アプリケーションのデプロイの要件と一致する可能性がありますが、クラウド導入チームは次のような長期的な問題を検討する必要があります。

- このデプロイは、PaaS 以外の他のリソースへのアクセスを必要とする範囲または規模に拡大する予定ですか。
- 現在のソリューション以外に他の PaaS のデプロイは計画されていますか。
- 組織は将来の他のクラウド移行の計画を立てていますか。

このような問題に対する答えは、チームが PaaS のみの選択肢を選ぶことを妨げるものではありませんが、最終的な決定を下す前に検討する必要があります。
