---
title: ネットワーク DMZ
description: ハイブリッド システムの一部として Azure で実行しているアプリケーションとコンポーネントを不正アクセスから保護するために使用できるさまざまなメソッドを説明して比較します。
layout: LandingPage
ms.openlocfilehash: 98df0a25767c7a7282e67381c6465fe3263ce1fa
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="network-dmz"></a>ネットワーク DMZ

これらの参照用アーキテクチャは、Azure 仮想ネットワークとオンプレミス ネットワークまたはインターネットの間の境界を保護するネットワーク DMZ を作成するための実証済みプラクティスを示します。

<section class="series">
    <ul class="panelContent">
    <!-- DMZ between Azure and on-premises -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-hybrid.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure とオンプレミスの間の DMZ</h3>
                        <p>オンプレミス ネットワークを Azure に拡張するセキュアなハイブリッド ネットワークを実装します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- DMZ between Azure and the Internet -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure とインターネットの間の DMZ</h3>
                        <p>Azure に対するインターネット トラフィックを受け入れるセキュリティで保護されたネットワークを実装します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>