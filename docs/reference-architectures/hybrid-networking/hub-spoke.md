---
title: Azure にハブスポーク ネットワーク トポロジを実装する
description: Azure にハブスポーク ネットワーク トポロジを実装する方法。
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: f04af90f328a0434d44ca7ea90309f3209a3b69d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Azure にハブスポーク ネットワーク トポロジを実装する

この参照アーキテクチャでは、Azure にハブスポーク トポロジを実装する方法を示します。 *ハブ*は、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。 *スポーク*は、ハブとピアリングする VNet であり、ワークロードを分離するために使用できます。 トラフィックは、ExpressRoute または VPN ゲートウェイ接続を経由してオンプレミスのデータセンターとハブの間を流れます。  [**このソリューションをデプロイします**](#deploy-the-solution)。

![[0]][0]

*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*


このトポロジの利点は次のとおりです。

* **コストの削減**: 複数のワークロードを共有するサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めます。
* **サブスクリプション数の上限の解消**: 中央のハブに別のサブスクリプションから VNet をピアリングします。
* **問題の分離**: 中央の IT (SecOps、InfraOps) とワークロード (DevOps) の間で実施します。

このアーキテクチャの一般的な用途は次のとおりです。

* DNS、IDS、NTP、AD DS などの共有サービスを必要とするさまざまな環境 (開発、テスト、運用など) でデプロイされるワークロード。 共有サービスはハブ VNet に配置され、分離性を維持するために各環境はスポークにデプロイされます。
* 相互接続が不要であり、共有サービスへのアクセスは必要なワークロード。
* セキュリティ要素 (DMZ としてのハブのファイアウォールなど) の一元管理、および各スポークにおけるワークロードの分別管理が必要なエンタープライズ。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

* **オンプレミス ネットワーク**。 組織内で実行されているプライベートなローカル エリア ネットワークです。

* **VPN デバイス**。 オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。 VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。 サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。

* **VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。 仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。 詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。

> [!NOTE]
> この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。

* **ハブ VNet**。 ハブスポーク トポロジのハブとして使用する Azure VNet。 ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。

* **ゲートウェイ サブネット**。 仮想ネットワーク ゲートウェイは、同じサブネット内に保持されます。

* **スポーク VNet**。 ハブスポーク トポロジでスポークとして使用される 1 つ以上の Azure VNet です。 スポークを使用すると、独自の VNet にワークロードを分離して、その他のスポークから個別に管理できます。 各ワークロードには複数の階層が含まれる場合があります。これらの階層には、Azure ロード バランサーを使用して接続されている複数のサブネットがあります。 アプリケーション インフラストラクチャについて詳しくは、「[Running Windows VM workloads][windows-vm-ra]」(Windows VM ワークロードを実行する) および「[Running Linux VM workloads][linux-vm-ra]」(Linux VM ワークロードを実行する) をご覧ください。

* **VNet ピアリング**。 [ピアリング接続][vnet-peering]を使用して、同じ Azure リージョン内の 2 つの VNet を接続できます。 ピアリング接続は、VNet 間の待機時間の短い非推移的な接続です。 ピアリングが完了すると、VNet は、ルーターがなくても Azure のバックボーンを使用してトラフィックを交換します。 ハブスポーク ネットワーク トポロジでは、VNet ピアリングを使用して、ハブを各スポークに接続します。

> [!NOTE]
> この記事で説明するのは [Resource Manager](/azure/azure-resource-manager/resource-group-overview) のデプロイのみですが、クラシック VNet を同じサブスクリプションの Resource Manager VNet に接続することもできます。 これにより、クラシック デプロイ をスポークでホストして、ハブで共有するサービスのメリットを引き続き得ることができます。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、優先される特定の要件がない限り、従ってください。

### <a name="resource-groups"></a>リソース グループ

ハブ VNet と各スポーク VNet を異なるリソース グループに実装できます。同じ Azure リージョン内の同じ Azure Active Directory (Azure AD) テナントに属している限り、サブスクリプションが異なっていてもかまいません。 これにより、各ワークロードを分散管理し、サービスの共有はハブ VNet で維持できます。

### <a name="vnet-and-gatewaysubnet"></a>VNet と GatewaySubnet

*GatewaySubnet* という名前のサブネットを作成します。アドレス範囲は /27 です。 このサブネットは、仮想ネットワーク ゲートウェイで必要です。 このサブネットに 32 個のアドレスを割り当てると、将来的にゲートウェイ サイズの制限に達するのを回避できます。

ゲートウェイの設定について詳しくは、接続の種類に応じて、次の参照アーキテクチャをご覧ください。

- [ExpressRoute を使用するハイブリッド ネットワーク][guidance-expressroute]
- [VPN ゲートウェイを使用するハイブリッド ネットワーク][guidance-vpn]

高可用性を確保するために、ExpressRoute に加えてフェールオーバー用の VPN を使用できます。 「[Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha]」(ExpressRoute と VPN フェールオーバーを使用してオンプレミス ネットワークを Azure に接続する) をご覧ください。

オンプレミス ネットワークに接続する必要がない場合は、ゲートウェイを設定せずにハブスポーク トポロジを使用することもできます。 

### <a name="vnet-peering"></a>VNET ピアリング

VNet ピアリングは、2 つの VNet 間の非推移的な関係です。 相互接続のためにスポークが必要な場合は、これらのスポーク間に個別のピアリング接続を追加することを検討してください。

ただし、相互接続に必要なスポークが複数ある場合は、[VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]のために、有効なピアリング接続をすぐに使い果たします。 このシナリオでは、ユーザー定義ルート (UDR) を使用して、スポーク宛てのトラフィックをハブ VNet のルーターとして機能する NVA に強制的に送信することを検討してください。 これにより、スポークを相互に接続できます。

リモート ネットワークと通信するハブ VNet ゲートウェイを使用するようにスポークを構成することもできます。 ゲートウェイ トラフィックをスポークからハブに流して、リモート ネットワークに接続するには、次の手順を実行する必要があります。

  - **ゲートウェイ トランジットを許可する**ようにハブで VNet ピアリング接続を構成します。
  - **リモート ゲートウェイを使用する**ように各スポークで VNet ピアリング接続を構成します。
  - **転送されたトラフィックを許可する**ようにすべての VNet ピアリング接続を構成します。

## <a name="considerations"></a>考慮事項

### <a name="spoke-connectivity"></a>スポーク接続

スポーク間の接続が必要な場合は、ハブでのルーティング用の NVA の実装、およびスポークでの UDR を使用したハブへのトラフィックの転送を検討してください。

![[2]][2]

このシナリオでは、**転送されたトラフィックを許可する**ようにピアリング接続を構成する必要があります。

### <a name="overcoming-vnet-peering-limits"></a>VNet ピアリングの制限の解消

Azure の [VNet ごとの VNet ピアリング数の制限][vnet-peering-limit]を考慮してください。 制限を超える数のスポークを許可する場合は、ハブスポークハブスポーク トポロジの作成を検討してください。このトポロジでは、第 1 レベルのスポークもハブとして機能します。 この手法を次の図に示します。

![[3]][3]

また、多数のスポークに合わせてハブを拡張するために、ハブで共有するサービスも検討してください。 たとえば、ハブがファイアウォール サービスを提供する場合は、複数のスポークを追加するときにファイアウォール ソリューションの帯域幅の制限を検討します。 このような一部の共有サービスを第 2 レベルのハブに移動することをお勧めします。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] を参照してください。 ここでは、各 VNet の VM を使用して接続をテストします。 実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。

デプロイによってサブスクリプション内に作成されるリソース グループは、次のとおりです。

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

テンプレート パラメーター ファイルは、これらの名前を参照します。したがって、名前を変更する場合は、それに合わせてパラメーター ファイルも更新します。

### <a name="prerequisites"></a>前提条件

1. [参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. [Azure CLI 2.0][azure-cli-2] をインストールします。

3. [Azure の構成要素][azbb] npm パッケージをインストールします。

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドを使用して Azure アカウントにログインします。

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>シミュレートされたオンプレミスのデータセンターをデプロイする

シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順に従います。

1. 参照アーキテクチャ リポジトリの `hybrid-networking/hub-spoke` フォルダーに移動します。

2. `onprem.json` ファイルを開きます。 `adminUsername` および `adminPassword` の値を置き換えます。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。

4. 次のコマンドを実行します。

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、仮想マシン、および VPN ゲートウェイを作成します。 VPN ゲートウェイの作成には約 40 分かかります。

### <a name="deploy-the-hub-vnet"></a>ハブ VNet をデプロイする

ハブ VNet をデプロイするには、次の手順を実行します。

1. `hub-vnet.json` ファイルを開きます。 `adminUsername` および `adminPassword` の値を置き換えます。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。

3. `sharedKey` については、VPN 接続の共有キーを入力します。 

    ```bash
    "sharedKey": "",
    ```

4. 次のコマンドを実行します。

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、仮想マシン、VPN ゲートウェイ、およびゲートウェイへの接続が作成されます。  VPN ゲートウェイの作成には約 40 分かかります。

### <a name="test-connectivity-with-the-hub"></a>ハブとの接続をテストする

シミュレートされたオンプレミスの環境からハブ VNet への接続をテストします。

**Windows デプロイ**

1. Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。

2. `Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。 `onprem.json` パラメーター ファイルで指定したパスワードを使用します。

3. VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、ハブ VNet の ジャンプボックス VM に接続できることを確認します。

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
出力は次のようになります。

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> 既定で、Windows Server VM では Azure の ICMP 応答が許可されていません。 接続のテストに `ping` を使用する場合は、VM ごとに Windows の高度なファイアウォールで ICMP トラフィックを有効にする必要があります。

**Linux デプロイ**

1. Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。

2. `Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。 

3. Linux プロンプトから `ssh` を実行して、シミュレートされたオンプレミスの環境に接続します。 `onprem.json` パラメーター ファイルで指定したパスワードを使用します。

4. `ping` コマンドを使用して、ハブ VNet のジャンプボックス VM への接続をテストします。

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>スポーク VNet をデプロイする

スポーク VNet をデプロイするには、次の手順を実行します。

1. `spoke1.json` ファイルを開きます。 `adminUsername` および `adminPassword` の値を置き換えます。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (省略可能) Linux デプロイの場合は、`osType` を `Linux` に設定します。

3. 次のコマンドを実行します。

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. `spoke2.json` ファイルに対して手順 1. ～ 2. を繰り返します。

5. 次のコマンドを実行します。

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. 次のコマンドを実行します。

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>接続をテストする

シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストします。

**Windows デプロイ**

1. Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。

2. `Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。 `onprem.json` パラメーター ファイルで指定したパスワードを使用します。

3. VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、ハブ VNet の ジャンプボックス VM に接続できることを確認します。

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux デプロイ**

Linux VM を使用して、シミュレートされたオンプレミスの環境からスポーク VNet への接続をテストするには、次の手順を実行します。

1. Azure Portal を使用して、`onprem-jb-rg` リソース グループで `jb-vm1` という名前の VM を見つけます。

2. `Connect` をクリックし、ポータルに表示されている `ssh` コマンドをコピーします。 

3. Linux プロンプトから `ssh` を実行して、シミュレートされたオンプレミスの環境に接続します。 `onprem.json` パラメーター ファイルで指定したパスワードを使用します。

5. `ping` コマンドを使用して、各スポーク内のジャンプボックス VM への接続をテストします。

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>スポーク間に接続を追加する

この手順は省略可能です。 スポークに相互接続を許可する場合は、ハブの VNet 内のルーターとしてネットワーク仮想アプライアンス (NVA) を使用し、別のスポークへの接続を試みるときにスポークからルーターへのトラフィックを強制する必要があります。 1 つの VM としての基本的なサンプル NVA と、ユーザー定義のルート (UDR) をデプロイして、2 つのスポーク VNet の接続を許可するには、次の手順を実行します。

1. `hub-nva.json` ファイルを開きます。 `adminUsername` および `adminPassword` の値を置き換えます。

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. 次のコマンドを実行します。

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-spoke topology in Azure" (Azure のハブスポーク トポロジ)
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing" (Azure のハブスポーク トポロジと推移的なルーティング)
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing using an NVA" (Azure のハブスポーク トポロジと NVA を使用した推移的なルーティング)
[3]: ./images/hub-spokehub-spoke.svg "Hub-spoke-hub-spoke topology in Azure" (Azure のハブスポークハブスポーク トポロジ)
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
