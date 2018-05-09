---
title: Azure の参照アーキテクチャ
description: Azure での一般的なワークロードに対応する、参照アーキテクチャ、計画、および規範的実装ガイダンス。
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 4c0006404ed0c0f603e9599cd9c04e6d02fdbfeb
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Azure の参照アーキテクチャ

これらの参照アーキテクチャは、関連するアーキテクチャをまとめてシナリオごとに配置されています。 各アーキテクチャの説明には、推奨プラクティスが、スケーラビリティ、可用性、管理性、およびセキュリティに関する考慮事項と共に含まれます。 また、ほとんどにはデプロイ可能なソリューションも含まれています。

<section class="series">
    <ul class="panelContent">

<!-- N-tier -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/n-tier-sql-server.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>n 層アプリケーション</h3>
                        <p>Windows または Linux 用の n 層アプリケーションを Azure にデプロイします。</p>
                        <p>SQL Server と Apache Cassandra で構成が説明されています。 高可用性を実現するために、アクティブ/パッシブ構成を 2 つのリージョンにデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Hybrid network -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ハイブリッド ネットワーク</h3>
                        <p>このシリーズでは、オンプレミス ネットワークと Azure の間にネットワーク接続を作成するためのオプションを説明します。</p>
                        <p>構成には、プライベートの専用接続を実現するサイト間 VPN または Azure ExpressRoute が含まれます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Network DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ネットワーク DMZ</h3>
                        <p>このシリーズでは、ネットワーク DMZ を作成して、Azure 仮想ネットワークとオンプレミス ネットワークまたはインターネットの境界を保護する方法を説明します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Identity management -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./identity/images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ID 管理</h3>
                        <p>このシリーズでは、オンプレミスの Active Directory (AD) 環境と Azure ネットワークを統合するためのオプションを示します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- App Service web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>App Service Web アプリケーション</h3>
                        <p>このシリーズでは、Azure App Service を使用する Web アプリケーションのベスト プラクティスを説明します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
    <!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./jenkins/images/logo.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Jenkins ビルド サーバー</h3>
                        <p>スケーラブルなエンタープライズ レベルの Jenkins サーバーを Azure にデプロイして運用します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sharepoint/images/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SharePoint Server 2016 ファーム</h3>
                        <p>SQL Server Always On 可用性グループを使用して、高可用性 SharePoint Server 2016 ファームを Azure にデプロイして実行します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SAP NetWeaver and SAP HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver および SAP HANA</h3>
                        <p>SAP NetWeaver および SAP HANA を Azure の高可用性環境にデプロイして実行します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>