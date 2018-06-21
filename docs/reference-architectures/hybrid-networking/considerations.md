---
title: オンプレミス ネットワークを Azure に接続するためのソリューションを選択する
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541051"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>オンプレミス ネットワークを Azure に接続するためのソリューションを選択する

この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。 各オプションの参照アーキテクチャおよびデプロイ可能なソリューションが提供されます。

## <a name="vpn-connection"></a>VPN 接続

ご利用の仮想プライベートネットワーク (VPN) を使用して、IPSec VPN トンネルを介してオンプレミス ネットワークを Azure VNet に接続します。

このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。

**メリット**

- 構成が簡単です。

**課題**

- オンプレミスの VPN デバイスが必要です。
- Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この SLA は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。
- 現在、Azure VPN Gateway 経由の VPN 接続では、最大で 200 Mbps の帯域幅がサポートされています。 このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。

**[詳細][vpn]**

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute 接続

ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。 プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。 

このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。 

**メリット**

- 接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。
- 要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。 ただし、このオプションがない接続プロバイダーもあります。
- 接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。
- 接続全体において、99.9% の可用性 SLA があります。

**課題**

- セットアップが複雑になることがあります。 ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。 このプロバイダーは、ネットワーク接続のプロビジョニングを行います。
- オンプレミスの、高帯域幅のルーターが必要です。

**[詳細][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>VPN のフェールオーバーを伴う ExpressRoute

このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。

このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。 

**メリット**

- フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。

**課題**

- 構成が複雑です。 VPN 接続と ExpressRoute 回線の両方を設定する必要があります。
- 冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。

**[詳細][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md