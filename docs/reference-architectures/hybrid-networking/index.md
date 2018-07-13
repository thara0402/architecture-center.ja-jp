---
title: オンプレミス ネットワークを Azure に接続するためのソリューションを選択する
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: 0cc07d3b7d45accf9f99ce32914b0ef065d62f32
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987480"
---
# <a name="connect-an-on-premises-network-to-azure"></a>オンプレミス ネットワークの Azure への接続

この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。 各オプションには、さらに詳細な参照アーキテクチャが用意されています。

## <a name="vpn-connection"></a>VPN 接続

[VPN ゲートウェイ](/azure/vpn-gateway/vpn-gateway-about-vpngateways)は、Azure 仮想ネットワークとオンプレミスの場所の間で暗号化されたトラフィックを送信する仮想ネットワーク ゲートウェイの一種です。 暗号化されたトラフィックは公共のインターネットを通過します。

このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。

**メリット**

- 構成が簡単です。

**課題**

- オンプレミスの VPN デバイスが必要です。
- Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。
- 現在、Azure VPN Gateway 経由の VPN 接続では、最大で 200 Mbps の帯域幅がサポートされています。 このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。

**参照アーキテクチャ**

- [VPN ゲートウェイを使用するハイブリッド ネットワーク](./vpn.md)

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute 接続

[ExpressRoute](/azure/expressroute/) 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。 プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。 

このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。 

**メリット**

- 接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。
- 要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。 ただし、このオプションがない接続プロバイダーもあります。
- 接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。
- 接続全体において、99.9% の可用性 SLA があります。

**課題**

- セットアップが複雑になることがあります。 ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。 このプロバイダーは、ネットワーク接続のプロビジョニングを行います。
- オンプレミスの、高帯域幅のルーターが必要です。

**参照アーキテクチャ**

- [ExpressRoute を使用したハイブリッド ネットワーク](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>VPN のフェールオーバーを伴う ExpressRoute

このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。

このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。 

**メリット**

- フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。

**課題**

- 構成が複雑です。 VPN 接続と ExpressRoute 回線の両方を設定する必要があります。
- 冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。

**参照アーキテクチャ**

- [ExpressRoute と VPN フェールオーバーを使用したハイブリッド ネットワーク](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a>ハブスポーク ネットワーク トポロジ

ハブスポーク ネットワーク トポロジは、ID やセキュリティなどのサービスを共有しながらワークロードを切り離す手法です。 ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。 スポークはハブとピア接続する Vnet です。 共有サービスはハブにデプロイされ、個々のワークロードはスポークとしてデプロイされます。


**参照アーキテクチャ**

- [ハブスポーク トポロジ](./hub-spoke.md)
- [共有サービスを含むハブスポーク](./shared-services.md)
