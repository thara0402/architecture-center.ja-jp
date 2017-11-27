---
title: "Azure の参照用アーキテクチャ"
description: "Azure での一般的なワークロードに対応する、参照用アーキテクチャ、計画、および規範的実装ガイダンス。"
layout: LandingPage
ms.openlocfilehash: eaff531c28faeb0aac774acf70d4334fb4e1f319
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="azure-reference-architectures"></a>Azure の参照用アーキテクチャ

これらの参照用アーキテクチャは、関連するアーキテクチャをまとめてシナリオごとに配置されています。 各アーキテクチャの説明には、推奨プラクティスが、スケーラビリティ、可用性、管理性、およびセキュリティに関する考慮事項と共に含まれます。 また、ほとんどにはデプロイ可能なソリューションも含まれています。

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Windows VM ワークロード</h3>
                            <p>このシリーズでは、1 つの Windows VM を実行するためのベスト プラクティスから始まり、負荷分散された複数の VM、最後には複数地域の n 層アプリケーションへと続きます。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Linux VM ワークロード</h3>
                            <p>このシリーズでは、1 つの Linux VM を実行するためのベスト プラクティスから始まり、負荷分散された複数の VM、最後には複数地域の n 層アプリケーションへと続きます。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Hybrid network -->
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ハイブリッド ネットワーク</h3>
                            <p>このシリーズでは、オンプレミス ネットワークと Azure の間にネットワーク接続を作成するためのオプションを説明します。</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
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
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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

<ul class="panelContent cardsI">
<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
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

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
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


</section>

