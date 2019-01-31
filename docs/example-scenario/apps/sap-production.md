---
title: Oracle データベースを使用した SAP 運用環境ワークロードの実行
titleSuffix: Azure Example Scenarios
description: Oracle データベースを使用して Azure で SAP 運用環境デプロイを実行します。
author: DharmeshBhagat
ms.date: 09/12/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, SAP, Windows, Linux
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-sap-production.png
ms.openlocfilehash: 03714dbf08c23220fa95a3789adb40d7a5cfac92
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908241"
---
# <a name="running-sap-production-workloads-using-an-oracle-database-on-azure"></a>Azure での Oracle データベースを使用した SAP 運用環境ワークロードの実行

SAP システムは、ミッション クリティカルなビジネス アプリケーションの実行に使用されます。 停止すると主要なプロセスが中断され、経費が増加したり収入が失われたりする可能性があります。 こうした事態を回避するには、高可用性を備え、障害発生時の回復性がある SAP インフラストラクチャが必要です。

高可用性を備えた SAP 環境を構築するには、システム アーキテクチャとプロセスから単一障害点を排除する必要があります。 単一障害点は、サイトの障害やシステム コンポーネントのエラー、さらにはヒューマン エラーが原因で発生することがあります。

このシナリオの例では、高可用性 (HA) Oracle データベースと共に、Azure 上の Windows または Linux 仮想マシン (VM) 上で SAP をデプロイする方法について説明しています。 SAP のデプロイには、要件に応じてさまざまな規模の VM を使用できます。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- SAP で実行されるミッション クリティカルなワークロード。
- 非クリティカルな SAP ワークロード。
- 高可用性環境をシミュレートする SAP のテスト環境。

## <a name="architecture"></a>アーキテクチャ

![Azure の運用 SAP 環境のアーキテクチャの概要][architecture]

この例には、異なる仮想マシン上で動作する Oracle データベース、SAP セントラル サービス、複数の SAP アプリケーション サーバーの高可用性構成が含まれています。 Azure ネットワークでは、セキュリティを目的として[ハブ アンド スポーク トポロジ](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)が使用されています。 このソリューションのデータ フローは次のとおりです。

1. ユーザーは、SAP ユーザー インターフェイスや Web ブラウザー、Microsoft Excel などの他のクライアント ツールを使用して SAP システムにアクセスします。 ExpressRoute 接続は、組織のオンプレミスのネットワークから Azure で実行されるリソースへのアクセスを提供します。
2. ExpressRoute の Azure における末端は、ExpressRoute 仮想ネットワーク (VNet) ゲートウェイです。 ネットワーク トラフィックは、ハブ VNet で作成された ExpressRoute ゲートウェイ経由でゲートウェイ サブネットにルーティングされます。
3. ハブ VNet はスポーク VNet にピアリングされています。 アプリケーション層サブネットは、可用性セットで SAP を実行している仮想マシンをホストします。
4. ID 管理サーバーは、ソリューションに認証サービスを提供します。
5. システム管理者はジャンプ ボックスを使用して、Azure にデプロイされたリソースを安全に管理します。

### <a name="components"></a>コンポーネント

- このシナリオでは、Azure で仮想ハブ アンド スポーク トポロジを作成するために[仮想ネットワーク](/azure/virtual-network/virtual-networks-overview)が使用されています。

- [仮想マシン](/azure/virtual-machines/windows/overview)は、ソリューションの各層の計算リソースを提供します。 仮想マシンの各クラスターは、[可用性セット](/azure/virtual-machines/windows/regions-and-availability#availability-sets)として構成されています。

- [ExpressRoute](/azure/expressroute/expressroute-introduction) を利用すると、接続プロバイダーが確立するプライベート接続を介して、オンプレミスのネットワークが Microsoft クラウドに拡張されます。

- [ネットワーク セキュリティ グループ (NSG)](/azure/virtual-network/security-overview) は、仮想ネットワーク内のリソースへのネットワーク アクセスを制限します。 NSG には、ソースまたはターゲット IP アドレス、ポート、プロトコルを基に、ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。

- [リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)は、Azure リソースの論理コンテナーとして機能します。

### <a name="alternatives"></a>代替手段

SAP では、Azure の環境でのさまざまなオペレーティング システム、データベース管理システム、VM の種類の組み合わせを柔軟に選択できます。 詳細については、[SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533) の「SAP Applications on Azure: Supported Products and Azure VM Types (Azure 上の SAP アプリケーション:サポート対象の製品と Azure VM の種類)」を参照してください。

## <a name="considerations"></a>考慮事項

- Azure で高可用性 SAP 環境を構築するための推奨プラクティスが定義されています。 詳細については、「[SAP NetWeaver のための高可用性のアーキテクチャとシナリオ](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios)」を参照してください。 [Azure VM での SAP アプリケーションの高可用性](/azure/virtual-machines/workloads/sap/high-availability-guide)に関する記事もご覧ください。

- Oracle データベースにも Azure 用の推奨プラクティスがあります。 詳細については、「[Azure での Oracle データベースの設計と実装](/azure/virtual-machines/workloads/oracle/oracle-design)」を参照してください。

- Oracle Data Guard は、ミッション クリティカルな Oracle データベースの単一障害点を排除するために使用されます。 詳細については、「[Azure Linux 仮想マシンで Oracle Data Guard を実装する](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard)」を参照してください。

- Microsoft Azure は、Oracle データベースと共に SAP 製品をデプロイするために使用できるインフラストラクチャ サービスを提供します。 詳細については、「[SAP ワークロードのための Oracle DBMS のデプロイ](/azure/virtual-machines/workloads/sap/dbms_guide_oracle)」を参照してください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるよう、すべてのサービスが料金計算ツールで事前構成されています。以下に例を示します。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

受信するトラフィックの量に基づいて、次の 4 つのサンプル コスト プロファイルが用意されています。

|サイズ|SAP|DB VM の種類|DB ストレージ|(A)SCS VM|(A)SCS ストレージ|アプリの VM の種類|アプリ ストレージ|Azure 料金計算ツール|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Small|30000|DS13_v2|4xP20、1xP20|DS11_v2|1x P10|DS13_v2|1x P10|[Small](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Medium|70000|DS14_v2|6xP20、1xP20|DS11_v2|1x P10|4x DS13_v2|1x P10|[Medium](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
Large|180000|E32s_v3|5xP30、1xP20|DS11_v2|1x P10|6x DS14_v2|1x P10|[Large](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Extra Large|250000|M64s|6xP30、1xP30|DS11_v2|1x P10|10x DS14_v2|1x P10|[Extra Large](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

> [!NOTE]
> この価格は目安であり、VM とストレージの料金を示しているに過ぎません。 ネットワーク、バックアップ ストレージ、データ イングレス/エグレスの料金は含まれていません。

- [Small](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c):小規模なシステムは、データベース サーバー用の VM の種類 DS13_v2 (8 個の vCPU)、56 GB の RAM、112 GB の一時ストレージ、追加の 5 個の 512 GB の Premium Storage ディスクで構成されています。 VM の種類 DS11_v2 (2 個の vCPU)、14 GB の RAM、28 GB の一時ストレージを使用する SAP セントラル インスタンス サーバーも含まれます。 SAP アプリケーション サーバー用の 1 個の VM の種類 DS13_v2 (8 個の vCPU)、56 GB の RAM、400 GB の一時ストレージ、追加の 1 個の 128 GB の Premium Storage ディスクも含まれます。

- [Medium](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a):中規模のシステムは、データベース サーバー用の VM の種類 DS14_v2 (16 個の vCPU)、112 GB の RAM、800 GB の一時ストレージ、追加の 7 個の 512 GB の Premium Storage ディスクで構成されています。 VM の種類 DS11_v2 (2 個の vCPU)、14 GB の RAM、28 GB の一時ストレージを使用する SAP セントラル インスタンス サーバーも含まれます。 SAP アプリケーション サーバー用の 4 個の VM の種類 DS13_v2 (8 個の vCPU)、56 GB の RAM、400 GB の一時ストレージ、追加の 1 個の 128 GB の Premium Storage ディスクも含まれます。

- [Large](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42):大規模なシステムは、データベース サーバー用の VM の種類 E32s_v3 (32 個の vCPU)、256 GB の RAM、800 GB の一時ストレージ、追加の 3 個の 512 GB と 1 個の 128 GB Premium Storage ディスクで構成されています。 VM の種類 DS11_v2 (2 個の vCPU)、14 GB の RAM、28 GB の一時ストレージを使用する SAP セントラル インスタンス サーバーも含まれます。 SAP アプリケーション サーバー用の 6 個の VM の種類 DS14_v2 (16 個の vCPU)、112 GB の RAM、224 GB の一時ストレージ、追加の 6 個の 128 GB の Premium Storage ディスクも含まれます。

- [Extra Large](https://azure.com/e/58c636922cf94faf9650f583ff35e97b):超大規模なシステムは、データベース サーバー用の VM の種類 M64s (64 個の vCPU)、1,024 GB の RAM、2,000 GB の一時ストレージ、追加の 7 個の 1024 GB の Premium Storage ディスクで構成されています。 VM の種類 DS11_v2 (2 個の vCPU)、14 GB の RAM、28 GB の一時ストレージを使用する SAP セントラル インスタンス サーバーも含まれます。 SAP アプリケーション サーバー用の 10 個の VM の種類 DS14_v2 (16 個の vCPU)、112 GB の RAM、224 GB の一時ストレージ、追加の 10 個の 128 GB の Premium Storage ディスクも含まれます。

## <a name="deployment"></a>Deployment

次のリンクを使用して、このシナリオの基盤となるインフラストラクチャをデプロイします。

<!-- markdownlint-disable MD033 -->

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> このデプロイでは、SAP と Oracle はインストールされません。 それらのコンポーネントは、個別にデプロイする必要があります。

## <a name="related-resources"></a>関連リソース

Azure での SAP 運用ワークロードの実行の詳細については、以下の参照アーキテクチャをご確認ください。

- [AnyDB 向けの SAP NetWeaver](/azure/architecture/reference-architectures/sap/sap-netweaver)
- [SAP S/4HANa](/azure/architecture/reference-architectures/sap/sap-s4hana)
- [SAP HANA L インスタンス](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-sap-production.png
