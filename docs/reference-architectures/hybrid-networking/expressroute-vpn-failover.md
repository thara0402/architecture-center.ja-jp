---
title: 高可用性ハイブリッド ネットワーク アーキテクチャの実装
description: Azure 仮想ネットワークと、 VPN ゲートウェイ フェールオーバー付きの ExpressRoute で接続されたオンプレミス ネットワークをカバーする、セキュアなサイト間ネットワーク アーキテクチャの実装方法。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 31ed1dbf59c4fa2b7fa86b9ceb2fed7b36e75c8c
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428824"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する

このリファレンス アーキテクチャでは、サイト間仮想プライベート ネットワーク (VPN) をフェールオーバー接続として使用した ExpressRoute を使用して、オンプレミス ネットワークを Azure 仮想ネットワーク (VNet) に接続する方法を示します。 オンプレミス ネットワークと Azure VNet 間のトラフィックは、ExpressRoute 接続を介して行き来します。 ExpressRoute 回線の接続が失われた場合、トラフィックは IPSec VPN トンネルを経由してルーティングされます。 「[**Deploy this solution**](#deploy-the-solution)」(このソリューションのデプロイ)。

ExpressRoute 回線を使用できない場合、VPN ルートではプライベート ピアリング接続のみが処理されます。 パブリック ピアリングと Microsoft ピアリングの接続は、インターネットを使用して処理されます。 

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ 

アーキテクチャは、次のコンポーネントで構成されます。

* **オンプレミス ネットワーク**。 組織内で運用されているプライベートのローカル エリア ネットワーク。

* **VPN アプライアンス**。 オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービス。 VPN アプライアンスは、ハードウェア デバイスである場合もありますし、ソフトウェア ソリューション (Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) など) である場合もあります。 サポートされている VPN アプライアンスの一覧と、Azure への接続用に選択した VPN アプライアンスの構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」を参照してください。

* **ExpressRoute 回線**。 エッジ ルーターを介してオンプレミス ネットワークを Azure につなげる、レイヤー 2 またはレイヤー 3 の回線 (接続プロバイダーから提供されます)。 この回線では、接続プロバイダーによって管理されるハードウェア インフラストラクチャが使用されます。

* **ExpressRoute 仮想ネットワーク ゲートウェイ**。 ExpressRoute 仮想ネットワーク ゲートウェイは、オンプレミス ネットワークとの接続に使用される ExpressRoute 回線に、VNet が接続できるようにするためのものです。

* **VPN 仮想ネットワーク ゲートウェイ**。 VPN 仮想ネットワーク ゲートウェイは、 VNet がオンプレミス ネットワーク内の VPN アプライアンスに接続できるようにするためのものです。 VPN 仮想ネットワーク ゲートウェイは、オンプレミス ネットワークからの要求を、VPN アプライアンスを通じてのみ受け入れるように構成されます。 詳細については、「[オンプレミス ネットワークを Microsoft Azure 仮想ネットワークに接続する][connect-to-an-Azure-vnet]」を参照してください。

* **VPN 接続**。 この接続には、接続の種類 (IPSec) を指定するプロパティ と、オンプレミス VPN アプライアンスとの共有キー (トラフィックの暗号化用) を指定するプロパティがあります。

* **Azure 仮想ネットワーク (VNet)**。 各 VNet は 1 つの Azure リージョンに配置され、複数のアプリケーション層をホストできます。 アプリケーション層は、各 VNet 内でサブネットを使用してセグメント化できます。

* **ゲートウェイ サブネット**。 仮想ネットワーク ゲートウェイは、同じサブネット内に保持されます。

* **クラウド アプリケーション**。 Azure でホストされるアプリケーション。 Azure ロード バランサーを介して接続された複数のサブネットを持つ、複数の層が含まれる場合があります。 アプリケーション インフラストラクチャの詳細については、「[Windows VM ワークロードの実行][windows-vm-ra]」と「[Linux VM ワークロードの実行][linux-vm-ra]」を参照してください。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。

### <a name="vnet-and-gatewaysubnet"></a>VNet と GatewaySubnet

ExpressRoute 仮想ネットワーク ゲートウェイと VPN 仮想ネットワーク ゲートウェイは、同じ VNet 内に作成します。 つまり両者は、*GatewaySubnet* という同じサブネットを共有する必要があります。

*GatewaySubnet* という名前のサブネットが既に VNet に含まれている場合は、/27 以上のアドレス空間があることを確認してください。 既存のサブネットが小さすぎる場合は、次の PowerShell コマンドを使用して、そのサブネットを削除します。 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

**GatewaySubnet** という名前のサブネットが VNet に含まれていない場合は、次の Powershell コマンドを使用して新しいサブネットを作成します。

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN および ExpressRoute ゲートウェイ

Azure に接続するための [ExpressRoute 前提条件][expressroute-prereq]が満たされていることを確認してください。

Azure VNet 内に VPN 仮想ネットワーク ゲートウェイが既にある場合は、次の Powershell コマンドを使用して、そのゲートウェイを削除します。

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

「[Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute]」(Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャを実装する) の手順に従って、ExpressRoute 接続を確立します。

「[Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn]」(Azure とオンプレミス VPN を使用したハイブリッド ネットワーク アーキテクチャを実装する) の手順に従って、VPN 仮想ネットワーク ゲートウェイ接続を確立します。

仮想ネットワーク ゲートウェイ接続を確立したら、次の手順で環境をテストします。

1. オンプレミス ネットワークから Azure VNet に接続できることを確認します。
2. テストのために ExpressRoute 接続を停止するようプロバイダーに依頼します。
3. VPN 仮想ネットワーク ゲートウェイ接続を使用して、オンプレミス ネットワークから Azure VNet に接続できることを確認します。
4. ExpressRoute 接続を再確立するよう、プロバイダーに依頼します。

## <a name="considerations"></a>考慮事項

ExpressRoute の考慮事項については、「[Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute]」(Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャを実装する) のガイダンスを参照してください。

サイト間 VPN の考慮事項については、「[Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn]」(Azure とオンプレミス VPN を使用したハイブリッド ネットワーク アーキテクチャを実装する) のガイダンスを参照してください。

Azure のセキュリティに関する全般的な考慮事項については、「[Microsoft クラウド サービスとネットワーク セキュリティ][best-practices-security]」を参照してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

**前提条件。** 既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。

ソリューションをデプロイするには、次の手順を実行します。

1. 下記のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure ポータルでリンクが開くのを待った後、次の手順に従います。   
   * **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-er-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンをクリックします。
3. デプロイが完了するまで待ちます。
4. 下のボタンをクリックしてください。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Azure ポータルでリンクが開くのを待った後、次の手順に従います。
   * **[リソース グループ]** セクションで **[既存のものを使用]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-er-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンをクリックします。

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "ExpressRoute および VPN ゲートウェイを使用した高可用性ハイブリッド ネットワーク アーキテクチャ"
