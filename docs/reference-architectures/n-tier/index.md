---
title: n 層アプリケーションの参照アーキテクチャ
description: エンタープライズ規模のアプリケーションを Azure でホストする VM をデプロイするための一般的なアーキテクチャについて説明します。
layout: LandingPage
ms.openlocfilehash: 288acc36e7c310e70240caa3ed9f2095bbb8bc58
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/10/2018
ms.locfileid: "34007627"
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="n-tier-application-reference-architectures"></a>n 層アプリケーションの参照アーキテクチャ

<section class="series">
    <ul class="panelContent">

<!-- N-tier Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-sql-server.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server を使用した n 層アプリケーション</h3>
                        <p>SQL Server on Windows を使用して、n 層アプリケーション用に構成された VM と仮想ネットワークをデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Multi-region Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server を使用したマルチリージョン n 層アプリケーション</h3>
                        <p>SQL Server AlwaysOn 可用性グループを使用して、高可用性を実現するために n 層アプリケーションを 2 つのリージョンにデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-cassandra.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cassandra を使用した n 層アプリケーション</h3>
                        <p>Apache Cassandra を使用して、n 層アプリケーション用に構成された Linux VM と仮想ネットワークをデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <a href="./windows-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Windows VM</h3>
                        <p>Azure で Windows VM を実行するためのベースラインの推奨事項。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<li style="display: flex; flex-direction: column;">
    <a href="./linux-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/LinuxPenguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linux VM</h3>
                        <p>Azure で Linux VM を実行するためのベースラインの推奨事項。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

</ul>