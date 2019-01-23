---
title: オンプレミス ネットワークの Azure への接続
titleSuffix: Azure Reference Architectures
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 6172866b08197b0ca1cd3aabb3c14c01b4f06f9c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486858"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>オンプレミス ネットワークを Azure に接続するためのソリューションを選択する

この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。 各オプションには、さらに詳細な参照アーキテクチャが用意されています。

## <a name="vpn-connection"></a>VPN 接続

[VPN ゲートウェイ](/azure/vpn-gateway/vpn-gateway-about-vpngateways)は、Azure 仮想ネットワークとオンプレミスの場所の間で暗号化されたトラフィックを送信する仮想ネットワーク ゲートウェイの一種です。 暗号化されたトラフィックは公共のインターネットを通過します。

このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。

### <a name="benefits"></a>メリット

- 構成が簡単です。

### <a name="challenges"></a>課題

- オンプレミスの VPN デバイスが必要です。
- Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。
- 現在、Azure VPN Gateway 経由の VPN 接続では、最大で 1.25 Gbps の帯域幅がサポートされています。 このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。

### <a name="reference-architecture"></a>参照アーキテクチャ

- [VPN ゲートウェイを使用するハイブリッド ネットワーク](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute 接続

[ExpressRoute](/azure/expressroute/) 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。 プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。

このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。

### <a name="benefits"></a>メリット

- 接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。
- 要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。 ただし、このオプションがない接続プロバイダーもあります。
- 接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。
- 接続全体において、99.9% の可用性 SLA があります。

### <a name="challenges"></a>課題

- セットアップが複雑になることがあります。 ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。 このプロバイダーは、ネットワーク接続のプロビジョニングを行います。
- オンプレミスの、高帯域幅のルーターが必要です。

### <a name="reference-architecture"></a>参照アーキテクチャ

- [ExpressRoute を使用したハイブリッド ネットワーク](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>VPN のフェールオーバーを伴う ExpressRoute

このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。

このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。

### <a name="benefits"></a>メリット

- フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。

### <a name="challenges"></a>課題

- 構成が複雑です。 VPN 接続と ExpressRoute 回線の両方を設定する必要があります。
- 冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。

### <a name="reference-architecture"></a>参照アーキテクチャ

- [ExpressRoute と VPN フェールオーバーを使用したハイブリッド ネットワーク](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a>ハブスポーク ネットワーク トポロジ

ハブスポーク ネットワーク トポロジは、ID やセキュリティなどのサービスを共有しながらワークロードを切り離す手法です。 ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。 スポークはハブとピア接続する Vnet です。 共有サービスはハブにデプロイされ、個々のワークロードはスポークとしてデプロイされます。

### <a name="reference-architectures"></a>参照用アーキテクチャ

- [ハブスポーク トポロジ](./hub-spoke.md)
- [共有サービスを含むハブスポーク](./shared-services.md)
