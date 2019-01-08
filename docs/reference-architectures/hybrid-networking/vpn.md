---
title: VPN を使用した Azure へのオンプレミス ネットワークの接続
titleSuffix: Azure Reference Architectures
description: VPN を使用して接続された Azure 仮想ネットワークとオンプレミス ネットワークにまたがる、セキュリティで保護されたサイト間ネットワーク アーキテクチャを実装します。
author: RohitSharma-pnp
ms.date: 10/22/2018
ms.openlocfilehash: 92a5a12675ca12075bec3c7f59f73a19287fe5d7
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644208"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>VPN ゲートウェイを使用した Azure へのオンプレミス ネットワークの接続

この参照アーキテクチャでは、サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure に拡張する方法を示します。 オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) 間のトラフィックは、IPSec VPN トンネルを介して行き来します。 [**このソリューションをデプロイします**](#deploy-the-solution)。

![オンプレミスと Azure インフラストラクチャにまたがるハイブリッド ネットワーク](./images/vpn.png)

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

- **オンプレミス ネットワーク**。 組織内で運用されているプライベートのローカル エリア ネットワーク。

- **VPN アプライアンス**。 オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービス。 VPN アプライアンスは、ハードウェア デバイスである場合や、ソフトウェア ソリューション (Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) など) である場合があります。 サポートされている VPN アプライアンスの一覧と、Azure VPN ゲートウェイに接続するためのその構成については、[サイト間 VPN Gateway 接続用の VPN デバイス][vpn-appliance]に関するページをご覧ください。

- **仮想ネットワーク (VNet)**。 Azure VPN ゲートウェイのクラウド アプリケーションとコンポーネントは同じ [VNet][azure-virtual-network] に存在します。

- **Azure VPN ゲートウェイ**。 [VPN ゲートウェイ][azure-vpn-gateway] サービスを使用すると、VNet を VPN アプライアンスを介してオンプレミス ネットワークに接続できます。 詳細については、「[オンプレミス ネットワークを Microsoft Azure 仮想ネットワークに接続する][connect-to-an-Azure-vnet]」を参照してください。 VPN ゲートウェイには、次の要素が含まれています。

  - **仮想ネットワーク ゲートウェイ**。 VNet の仮想 VPN アプライアンスを提供するリソース。 オンプレミス ネットワークから VNet へのトラフィックのルーティングを行います。
  - **ローカル ネットワーク ゲートウェイ**。 オンプレミス VPN アプライアンスの抽象化。 クラウド アプリケーションからオンプレミス ネットワークへのネットワーク トラフィックは、このゲートウェイを介してルーティングされます。
  - **接続**。 この接続には、接続の種類 (IPSec) を指定するプロパティと、オンプレミス VPN アプライアンスとの共有キー (トラフィックの暗号化用) を指定するプロパティがあります。
  - **ゲートウェイ サブネット**。 仮想ネットワーク ゲートウェイは独自のサブネットで保持され、以下の推奨事項セクションで説明されているさまざまな要件の対象となります。

- **クラウド アプリケーション**。 Azure でホストされるアプリケーション。 Azure ロード バランサーを介して接続された複数のサブネットを持つ、複数の層が含まれる場合があります。 アプリケーション インフラストラクチャの詳細については、「[Windows VM ワークロードの実行][windows-vm-ra]」と「[Linux VM ワークロードの実行][linux-vm-ra]」を参照してください。

- **内部ロード バランサー**:  VPN ゲートウェイからのネットワーク トラフィックは、内部ロード バランサーを介してクラウド アプリケーションにルーティングされます。 ロード バランサーは、アプリケーションのフロントエンドのサブネットに配置されます。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。

### <a name="vnet-and-gateway-subnet"></a>VNet とゲートウェイ サブネット

必要なすべてのリソースにとって十分な大きさのアドレス空間を持つ Azure VNet を作成します。 追加の VM が今後必要になると思われる場合は、拡大に対応できるだけの領域を VNet アドレス空間に確保します。 VNet のアドレス空間は、オンプレミス ネットワークと重複させることはできません。 たとえば、上の図は、VNet に対してアドレス空間 10.20.0.0/16 を使用しています。

*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。 このサブネットは、仮想ネットワーク ゲートウェイで必要です。 このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。 また、このサブネットは、アドレス空間の途中には配置しないでください。 ゲートウェイ サブネットのアドレス空間は、VNet アドレス空間の上端に設定することをお勧めします。 図の例では 10.20.255.224/27 を使用しています。  [CIDR] の簡単な計算手順を次に示します。

1. VNet のアドレス空間の変数ビットを、ゲートウェイ サブネットで使用されているビットまで 1 に設定し、残りのビットを 0 に設定します。
2. 結果のビットを 10 進数に変換し、プレフィックス長をゲートウェイ サブネットのサイズに設定して、その 10 進数をアドレス空間として表現します。

たとえば、IP アドレス範囲が 10.20.0.0/16 の VNet については、上の手順 1. を適用すると、10.20.0b11111111.0b11100000 になります。  これを 10 進数に変換し、アドレス空間として表現すると、10.20.255.224/27 が生成されます。

> [!WARNING]
> VM をゲートウェイ サブネットにデプロイしないでください。 また、NSG をこのサブネットに割り当てないでください。割り当てると、ゲートウェイが機能しなくなります。
>

### <a name="virtual-network-gateway"></a>仮想ネットワーク ゲートウェイ

仮想ネットワーク ゲートウェイにパブリック IP アドレスを割り当てます。

ゲートウェイ サブネットで仮想ネットワーク ゲートウェイを作成し、新しく割り当てられたパブリック IP アドレスを割り当てます。 要件に最も合致し、VPN アプライアンスによって有効になっているゲートウェイの種類を使用します。

- アドレス プレフィックスなどのポリシー基準に基づいて、要求がどのようにルーティングされるかを厳密に制御する必要がある場合、[ポリシー ベースのゲートウェイ][policy-based-routing]を作成します。 ポリシー ベースのゲートウェイは、静的ルーティングを使用し、サイト間接続でのみ機能します。

- RRAS を使用してオンプレミス ネットワークに接続する場合、マルチサイトまたはリージョン間接続をサポートする場合、または VNet 間接続 (複数の VNet を横断するルートを含む) を実装する場合、[ルート ベースのゲートウェイ][route-based-routing]を作成します。 ルート ベースのゲートウェイは、ネットワーク間でトラフィックを転送する動的ルーティングを使用します。 これは代替ルートを試すことができるため、静的ルートよりもネットワーク パスの障害に耐性があります。 また、ルート ベースのゲートウェイでは、ネットワーク アドレスが変わったときに、ルートを手動で更新する必要がない可能性があるため、管理オーバーヘッドを削減できます。

サポートされている VPN アプライアンスの一覧については、[サイト間 VPN Gateway 接続の VPN デバイス][vpn-appliances]に関するページをご覧ください。

> [!NOTE]
> ゲートウェイを作成したら、ゲートウェイを削除して再作成しない限り、ゲートウェイの種類を変更することはできません。
>

ご自分のスループット要件に最も合致する Azure VPN ゲートウェイ SKU を選択します。 詳細については、「[ゲートウェイの SKU][azure-gateway-skus]」を参照してください。

> [!NOTE]
> Basic SKU は、Azure ExpressRoute と互換性がありません。 ゲートウェイの作成後、[SKU を変更][changing-SKUs]できます。
>

料金は、ゲートウェイがプロビジョニングされ、利用可能になっている時間に基づいて発生します。 「[VPN Gateway の価格][azure-gateway-charges]」を参照してください。

要求がアプリケーション VM に直接渡されるようにするのではなく、アプリケーションの受信トラフィックをゲートウェイから内部ロード バランサーに転送する、ゲートウェイ サブネットのルーティング ルールを作成します。

### <a name="on-premises-network-connection"></a>オンプレミス ネットワーク接続

ローカル ネットワーク ゲートウェイを作成します。 オンプレミスの VPN アプライアンスのパブリック IP アドレスと、オンプレミス ネットワークのアドレス空間を指定します。 オンプレミス VPN アプライアンスには、Azure VPN Gateway のローカル ネットワーク ゲートウェイがアクセスできるパブリック IP アドレスが必要であることに注意してください。 VPN デバイスはネットワーク アドレス変換 (NAT) デバイスの背後には配置できません。

仮想ネットワーク ゲートウェイおよびローカル ネットワーク ゲートウェイのサイト間接続を作成します。 サイト間 (IPSec) 接続の種類を選択し、共有キーを指定します。 Azure VPN ゲートウェイによるサイト間の暗号化は IPSec プロトコルに基づいており、認証に事前共有キーを使用します。 Azure VPN ゲートウェイを作成するときにキーを指定します。 同じキーを使用して、オンプレミスで実行する VPN アプライアンスを構成する必要があります。 他の認証メカニズムは現在サポートされていません。

Azure VNet のアドレスに向けた要求が VPN デバイスに転送されるように、オンプレミス ルーティング インフラストラクチャが構成されていることを確認してください。

オンプレミス ネットワークのクラウド アプリケーションに必要なポートを開きます。

接続をテストして、次の点を確認してください。

- トラフィックが Azure VPN ゲートウェイを介してクラウド アプリケーションに到達するように、オンプレミス VPN アプライアンスによって正しくルーティングしている。
- トラフィックがオンプレミス ネットワークに戻るように、VNet によって正しくルーティングされている。
- 両方向の禁止されたトラフィックが、正しくブロックされている。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

垂直スケーラビリティを制限するには、Basic または Standard の VPN Gateway SKU から High Performance VPN SKU に移行します。

大量の VPN トラフィックが想定される VNet については、さまざまなワークロードを小さな個別の VNet に分散し、それぞれに対して VPN ゲートウェイを構成することを検討してください。

VNet は、水平方向または垂直方向にパーティション分割できます。 水平方向にパーティション分割するには、各階層の VM インスタンスをいくつか、新しい VNet のサブネットに移動します。 これにより、各 VNet の構造と機能が同じになります。 垂直方向にパーティション分割する場合は、機能がさまざまな論理領域 (注文処理、請求、顧客アカウントの管理など) に分割されるように各階層を再設計します。 その後、各機能領域を独自の VNet に配置できます。

VNet でオンプレミスの Active Directory ドメイン コントローラーをレプリケートし、VNet に DNS を実装すると、オンプレミスからクラウドにフローするセキュリティ関連および管理トラフィックの削減に役立ちます。 詳細については、[Azure への Active Directory Domain Services (AD DS) の拡張][adds-extend-domain]に関するページをご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

オンプレミス ネットワークを、Azure VPN ゲートウェイで引き続き使用できるようにする必要がある場合は、オンプレミス VPN ゲートウェイのフェールオーバー クラスターを実装します。

組織に複数のオンプレミス サイトがある場合は、1 つ以上の Azure VNet への[マルチサイト接続][vpn-gateway-multi-site]を作成します。 このアプローチには、動的 (ルート ベース) のルーティングが必要です。したがって、オンプレミス VPN ゲートウェイがこの機能をサポートしていることを確認してください。

サービス レベル アグリーメントの詳細については、「[VPN Gateway のSLA][sla-for-vpn-gateway]」を参照してください。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

オンプレミスの VPN アプライアンスからの診断情報を監視します。 このプロセスは、VPN アプライアンスが提供する機能によって異なります。 たとえば、Windows Server 2012 でルーティングとリモート アクセス サービスを使用している場合は、[RRAS ログ記録][rras-logging]です。

[Azure VPN ゲートウェイ診断][gateway-diagnostic-logs]を使用して、接続の問題に関する情報を取得します。 こうしたログを使用すると、接続要求の要求元と要求先、使用されたプロトコル、接続の確立方法 (または、試行が失敗した理由) などの情報を追跡できます。

Azure Portal で使用できる監査ログを使用して、Azure VPN ゲートウェイの操作ログを監視します。 ローカル ネットワーク ゲートウェイ、Azure ネットワーク ゲートウェイ、および接続に対して個別のログを使用できます。 この情報は、ゲートウェイに対する変更の追跡に使用でき、前に機能していたゲートウェイの動作が何らかの理由で停止した場合に役立ちます。

![Azure portal の監査ログ](../_images/guidance-hybrid-network-vpn/audit-logs.png)

接続を監視し、接続エラー イベントを追跡します。 この情報は、[Nagios][nagios] などの監視パッケージを使用して取得し、レポートすることができます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

VPN ゲートウェイごとに別の共有キーを生成します。 強力な共有キーを使用すると、ブルート フォース攻撃に対抗するうえで役立ちます。

> [!NOTE]
> 現在、Azure VPN ゲートウェイのキーを事前共有するために、Azure Key Vault を使用することはできません。
>

オンプレミスの VPN アプライアンスによって、[Azure VPN ゲートウェイと互換性のある][vpn-appliance-ipsec]暗号化方式が使用されていることを確認してください。 ポリシー ベースのルーティングでは、Azure VPN ゲートウェイは、AES256、AES128、および 3DES 暗号化アルゴリズムをサポートしています。 ルート ベースのゲートウェイは、AES256 および 3DES をサポートしています。

オンプレミス VPN アプライアンスが境界ネットワーク (DMZ) にあり、その境界ネットワークとインターネットの間にファイアウォールが存在する場合は、サイト間 VPN 接続を許可するために、[追加のファイアウォール規則][additional-firewall-rules]を構成しなければならない可能性があります。

VNet のアプリケーションがインターネットにデータを送信する場合は、インターネットに向かうすべてのトラフィックがオンプレミス ネットワークを介してルーティングされるように、[強制トンネリングを実装][forced-tunneling]することを検討してください。 このアプローチを使用すると、アプリケーションによって行われた送信要求をオンプレミス インフラストラクチャから監査できます。

> [!NOTE]
> 強制トンネリングは、Azure サービス (Storage サービスなど) と Windows ライセンス マネージャーへの接続に影響を与える場合があります。
>

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

**前提条件**。 既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。

ソリューションをデプロイするには、次の手順を実行します。

<!-- markdownlint-disable MD033 -->

1. 下記のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure ポータルでリンクが開くのを待った後、次の手順に従います。
   - **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-vpn-rg`」と入力します。
   - **[場所]** ボックスの一覧でリージョンを選択します。
   - **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   - 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   - **[購入]** ボタンをクリックします。
3. デプロイが完了するまで待ちます。

<!-- markdownlint-enable MD033 -->

接続のトラブルシューティングについては、[ハイブリッド VPN 接続のトラブルシューティング](./troubleshoot-vpn.md)に関するページをご覧ください。

<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
