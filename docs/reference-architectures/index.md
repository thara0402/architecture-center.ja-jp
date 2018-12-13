---
title: Azure の参照アーキテクチャ
description: Azure での一般的なワークロードに対応する、参照アーキテクチャ、計画、および規範的実装ガイダンス。
layout: LandingPage
ms.topic: landing-page
ms.date: 08/30/2018
ms.openlocfilehash: efac05ff95431f9551c6fb76939dbb91cfa01d1d
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307019"
---
# <a name="azure-reference-architectures"></a>Azure の参照アーキテクチャ

これらの参照アーキテクチャは、関連するアーキテクチャをまとめてシナリオごとに配置されています。 各アーキテクチャの説明には、推奨プラクティスが、スケーラビリティ、可用性、管理性、およびセキュリティに関する考慮事項と共に含まれます。 また、ほとんどにはデプロイ可能なソリューションも含まれています。

ジャンプ先:[AI](#ai-and-machine-learning) | [ビッグ データ](#big-data-solutions) | [サーバーレス](#serverless-applications) | [仮想ネットワーク](#virtual-networks) | [VM ワークロード](#vm-workloads) | [SAP](#sap) | [Web アプリ](#web-applications) | [Active Directory](#extend-on-premises-active-directory-to-azure)

## <a name="ai-and-machine-learning"></a>AI と機械学習

<!-- markdownlint-disable MD033 -->
<ul  class="panelContent cardsF">
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ディープ ラーニング モデル用のバッチ スコアリング</h3>
                        <p>ビデオにニューラル スタイルの転送を適用するバッチ ジョブの実行を自動化します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Python モデルのリアルタイム スコアリング</h3>
                        <p>通常の Python モデルまたはディープ ラーニング モデルを使用して、リアルタイムの予測を行う Web サービスとして Python モデルをデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring for R models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-r.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/logo-r.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>R 機械学習モデルを使用したリアルタイム スコアリング</h3>
                        <p>Azure Kubernetes Service (AKS) で実行される Microsoft Machine Learning Server を使用して、R のリアルタイム予測サービスを実装します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="big-data-solutions"></a>ビッグ データ ソリューション

<ul  class="panelContent cardsF">
<!-- SQL Data Warehouse -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-sqldw.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Data Warehouse を使用したエンタープライズ向け BI</h3>
                        <p>ELT (extract-load-transform) パイプラインでオンプレミス データベースから SQL Data Warehouse にデータを移動します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-adf.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Data Factory を使用したエンタープライズ BI の自動化</h3>
                        <p>ELT パイプラインを自動化し、オンプレミス データベースから増分を読み込みます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Databricks によるストリーム処理</h3>
                        <p>2 つのストリームからのレコードを結合するストリーム処理パイプラインによって、結果を充実させ、ローリング平均を計算します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream Analytics -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-stream-analytics.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-analysis-service.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Stream Analytics によるストリーム処理</h3>
                        <p>2 つのデータ ストリームからのレコードを関連付けてローリング平均を計算する、エンド ツー エンドのストリーム処理パイプライン。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="serverless-applications"></a>サーバーレス アプリケーション

<ul class="panelContent cardsF">
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>サーバーレス Web アプリケーション</h3>
                        <p>Blob Storage から静的コンテンツを提供し、Azure Functions を使用して API を実装するサーバーレス Web アプリケーション。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/event-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Functions を使用したイベント処理</h3>
                        <p>データのストリームを取り込み、Functions を使用してデータ処理を行うイベント ドリブン アーキテクチャ。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="virtual-networks"></a>仮想ネットワーク

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>仮想プライベート ネットワーク (VPN) を使用したハイブリッド ネットワーク</h3>
                        <p>オンプレミス ネットワークを Azure Virtual Network に接続します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute を使用したハイブリッド ネットワーク</h3>
                        <p>プライベートの専用接続を使用し、オンプレミス ネットワークを Azure まで拡張します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute + VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute と VPN フェールオーバーを使用したハイブリッド ネットワーク</h3>
                        <p>ExpressRoute を使用し、フェールオーバー接続として VPN を使用して可用性を高めます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub spoke -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ハブスポーク ネットワーク トポロジ</h3>
                        <p>オンプレミス ネットワークへの接続の中心点を作成し、同時にワークロードを切り離します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>共有サービスを含むハブスポーク トポロジ</h3>
                        <p>Active Directory などの共有サービスを含めることで、ハブスポーク トポロジを拡張します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure とオンプレミスの間の DMZ</h3>
                        <p>ネットワーク仮想アプライアンスを使用し、セキュリティで保護されたハイブリッド ネットワークを作成します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Internet DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure とインターネットの間の DMZ</h3>
                        <p>ネットワーク仮想アプライアンスを使用し、インターネット トラフィックを受信する、セキュリティで保護されたネットワークを作成します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- HA NVA -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/nva-ha.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>高可用性ネットワーク仮想アプライアンス</h3>
                        <p>高可用性のネットワーク仮想アプライアンス (NVA) セットを Azure にデプロイします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="vm-workloads"></a>VM ワークロード

<ul  class="panelContent cardsF">
<!-- n-tier windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server を使用した n 層アプリケーション</h3>
                        <p>SQL Server on Windows を使用し、n 層アプリケーション用に仮想マシンを構成します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Multi-region n-tier windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>マルチリージョン n 層アプリケーション</h3>
                        <p>SQL Server AlwaysOn 可用性グループを使用し、2 つのリージョンに n 層アプリケーションを置くことで可用性を高めます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/linux-penguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cassandra を使用した n 層アプリケーション</h3>
                        <p>Apache Cassandra on Linux を使用し、n 層アプリケーション用に仮想マシンを構成します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Jenkins -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/jenkins.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Jenkins ビルド サーバー</h3>
                        <p>拡張性が高い、エンタープライズレベルの Jenkins サーバーを Azure に置きます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SharePoint -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SharePoint Server 2016 ファーム</h3>
                        <p>SQL Server AlwaysOn 可用性グループを使用し、Azure の SharePoint Server 2016 ファームの可用性を高くします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="sap"></a>SAP

<ul  class="panelContent cardsF">
<!-- SAP -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-netweaver.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver</h3>
                        <p>ディザスター リカバリー対応で可用性の高い環境に SAP NetWeaver on Windows を置きます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-s4hana.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP S/4HANA</h3>
                        <p>ディザスター リカバリー対応で可用性の高い環境に SAP S/4HANA on Linux Windows を置きます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/hana-large-instances.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP HANA on Azure Large Instances</h3>
                        <p>HANA L インスタンスは、Azure リージョン内の物理サーバーに配置されます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="extend-on-premises-active-directory-to-azure"></a>オンプレミスの Active Directory を Azure に拡張する

<ul class="panelContent cardsF">
<!-- Azure AD -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-active-directory.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Active Directory との統合</h3>
                        <p>オンプレミス AD ドメインと Azure Active Directory を統合します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>オンプレミス Active Directory ドメインを Azure まで拡張する</h3>
                        <p>Active Directory Domain Services (AD DS) を Azure にデプロイし、オンプレミス ドメインを拡張します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS Forest -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure での AD DS フォレストの作成</h3>
                        <p>オンプレミス AD フォレスト内によって信頼される別個の AD ドメインを Azure で作成します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD FS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Active Directory フェデレーション サービス (AD FS) を Azure に拡張する</h3>
                        <p>Azure で実行しているコンポーネントのフェデレーション認証/承認に AD FS を使用します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="web-applications"></a>Web アプリケーション

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/basic-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>基本的な Web アプリケーション</h3>
                        <p>Azure App Service と Azure SQL Database を使用する Web アプリケーション。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>拡張性の高い Web アプリケーション</h3>
                        <p>Web アプリケーションの拡張性を向上させる証明済みの手法。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/multi-region.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>可用性の高い Web アプリケーション</h3>
                        <p>App Service Web アプリを複数のリージョンで実行し、高可用性を実現します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

