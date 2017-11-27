---
title: "オンプレミス ネットワークの Azure への接続"
description: "オンプレミス ネットワークと Azure の間のセキュアかつ堅牢なネットワーク接続のための推奨アーキテクチャ。"
layout: LandingPage
ms.openlocfilehash: 0707d17295e338af0176bd0cea806615ef05f9ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure"></a>オンプレミス ネットワークの Azure への接続

これらの参照用アーキテクチャは、オンプレミス ネットワークと Azure の間に堅牢なネットワーク接続を作成するための実証済みプラクティスを示します。 [どれを選択すればよいでしょうか。](./considerations.md)

<ul class="panelContent">
    <li>
        <a href="./vpn.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/vpn.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN</h3>
                            <p>サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure にまで延長します。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute</h3>
                            <p>内部設置型ネットワークを Azure ExpressRoute を使用して Azure にまで延長します。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute-vpn-failover.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN フェールオーバーを含む ExpressRoute</h3>
                            <p>Azure ExpressRoute を使用し、VPN をフェールオーバー接続として、オンプレミス ネットワークを Azure にまで延長します。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./hub-spoke.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/hub-spoke.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ハブスポーク トポロジ</h3>
                            <p>ハブは、オンプレミス ネットワークの接続の中心です。 スポークは、ハブに対して配置される VNet であり、ワークロードを分離するために使用されます。 </p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
</ul>

