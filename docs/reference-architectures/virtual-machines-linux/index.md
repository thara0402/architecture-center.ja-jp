---
title: Linux VM ワークロード
description: エンタープライズ規模のアプリケーションを Azure でホストする VM をデプロイするための一般的なアーキテクチャについて説明します。
layout: LandingPage
ms.openlocfilehash: ef06fb93355f4676f44954930bcfeb2c2d012d43
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="linux-vm-workloads"></a>Linux VM ワークロード

これらの参照用アーキテクチャは、Azure で Linux VM を実行するための実証済みプラクティスを示します。

<section class="series">
    <ul class="panelContent">
    <!-- Single VM -->
<li style="display: flex; flex-direction: column;">
    <a href="./single-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/single-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>単一の VM</h3>
                        <p>Azure ですべての Linux 仮想マシンを実行するためのベースラインの推奨事項です。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Load balanced VMs -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>負荷分散された VM</h3>
                        <p>スケーラビリティと可用性のためにロード バランサーの背後に配置された複数の VM。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>n 層アプリケーション</h3>
                        <p>Apache Cassandra を使用する n 層アプリケーションのために構成された VM。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Multi-region application -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-application.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>複数地域のアプリケーション</h3>
                        <p>高可用性のために 2 つの地域にデプロイされた n 層アプリケーション。</p>
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