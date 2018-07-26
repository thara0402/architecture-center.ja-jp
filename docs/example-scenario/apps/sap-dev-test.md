---
title: 開発/テスト ワークロード用の SAP
description: 開発/テスト環境の SAP シナリオ
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060963"
---
# <a name="sap-for-devtest-workloads"></a>開発/テスト ワークロード用の SAP

この例では、Azure で Windows または Linux 環境の SAP NetWeaver の開発/テスト実装を実行する方法に関するガイダンスを提供します。 使用するデータベースは AnyDB で、これはサポートされている任意の DBMS を表す SAP 用語 (SAP HANA 以外) です。 このアーキテクチャは、運用環境以外を対象に設計されているため、単一の仮想マシン (VM) のみでデプロイされ、サイズは組織のニーズに合わせて変更できます。

運用ユース ケースについては、以下で使用できる SAP リファレンス アーキテクチャを確認してください。

* [AnyDB 向けの SAP NetWeaver][sap-netweaver]
* [SAP S/4Hana][sap-hana]
* [SAP on Azure L インスタンス][sap-large]

## <a name="related-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* 重要度、生産性が低い SAP ワークロード (サンドボックス、開発、テスト、品質保証)
* 重要度が低い SAP Business One ワークロード

## <a name="architecture"></a>アーキテクチャ

![ダイアグラム](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

このシナリオでは、1 つの仮想マシンにおける 1 つの SAP システム データベースと SAP アプリケーション サーバーのプロビジョニングに対応できます。シナリオのデータ フローを次に示します。

1. プレゼンテーション層の顧客が、SAP GUI、またはオンプレミスの他のユーザー インターフェイス (Internet Explorer、Excel、または他の Web アプリケーション) を使用して、Azure ベースの SAP システムにアクセスします。
2. 確立された Express Route を使用して接続が提供されます。 Express Route は、Azure の Express Route ゲートウェイで終了します。 ネットワーク トラフィックが、Express Route ゲートウェイを介してゲートウェイ サブネット、アプリケーション層スポーク サブネットの順にルーティングされ ([ハブ スポーク][hub-spoke] パターンを参照)、さらにネットワーク セキュリティ ゲートウェイ経由で SAP アプリケーションの仮想マシンにルーティングされます。
3. ID 管理サーバーは、認証サービスを提供します。
4. ジャンプ ボックスは、ローカル管理機能を提供します。

### <a name="components"></a>コンポーネント

* [リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)は、Azure リソースの論理コンテナーです。
* [仮想ネットワーク](/azure/virtual-network/virtual-networks-overview)は、Azure 内のネットワーク通信の基盤です
* [仮想マシン](/azure/virtual-machines/windows/overview)である Azure Virtual Machines では、Windows または Linux サーバーを使用して、セキュリティで保護された高スケールな仮想化インフラストラクチャをオンデマンドで構築できます
* [Express Route](/azure/expressroute/expressroute-introduction) を利用すると、接続プロバイダーが提供するプライベート接続を介して、オンプレミスのネットワークを Microsoft クラウドに拡張できます。
* [ネットワーク セキュリティ グループ](/azure/virtual-network/security-overview)を使用すると、仮想ネットワーク内のリソースへのネットワーク トラフィックを制限できます。 ネットワーク セキュリティ グループには、ソースまたはターゲット IP アドレス、ポート、およびプロトコルを基に、受信/送信ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。 

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

 Microsoft は 1 つの VM インスタンスに対してサービス レベル アグリーメント (SLA) を提供します。 Virtual Machines の Microsoft Azure サービス レベル アグリーメントの詳細については、「[Virtual Machines の SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)」を参照してください

### <a name="scalability"></a>スケーラビリティ

スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べてください。すべてのサービスがコスト計算ツールで事前構成されています。  特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

取得するトラフィックの量に基づいて、次の 4 つのサンプル コスト プロファイルが用意されています。

|サイズ|SAP|VM の種類|Storage|Azure 料金計算ツール|
|----|----|-------|-------|---------------|
|Small|8000|D8s_v3|P20 x 2、P10 x 1|[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Medium|16000|D16s_v3|P20 x 3、P10 x 1|[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
Large|32000|E32s_v3|P20 x 3、P10 x 1|[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Extra Large|64000|M64s|P20 x 4、P10 x 1|[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

注: 価格はガイドであり、VM とストレージのコストのみを示しています (ネットワーク、バックアップ ストレージ、データの受信/送信料金は含まれません)。

* [Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): 小規模なシステムは、VM の種類 D8s_v3 (8 個の vCPU)、32 GB の RAM、200 GB の一時ストレージ、および 2 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。
* [Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): 中規模なシステムは、VM の種類は D16s_v3 (16 個の vCPU)、64 GB の RAM、400 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。
* [Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): 大規模なシステムは、VM の種類 E32s_v3 (32 個の vCPU)、256 GB の RAM、512 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。
* [Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): 非常に大規模なシステムは、VM の種類 M64s (64 個の vCPU)、1024 GB の RAM、2000 GB の一時ストレージ、および 4 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。

## <a name="deployment"></a>Deployment

上記のシナリオに似た基盤インフラストラクチャをデプロイするには、デプロイ ボタンを使用してください

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP はインストールされないため、インフラストラクチャを手動で構築した後、インストールする必要があります。

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke