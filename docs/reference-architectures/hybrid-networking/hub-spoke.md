---
title: "Azure にハブスポーク ネットワーク トポロジを実装する"
description: "Azure にハブスポーク ネットワーク トポロジを実装する方法。"
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
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

アーキテクチャは、次のコンポーネントで構成されています。

* **オンプレミス ネットワーク**。 組織内で実行されているプライベートなローカル エリア ネットワークです。

* **VPN デバイス**。 オンプレミス ネットワークへの外部接続を提供するデバイスまたはサービスです。 VPN デバイスには、ハードウェア デバイス、または Windows Server 2012 のルーティングとリモート アクセス サービス (RRAS) などのソフトウェア ソリューションがあります。 サポート対象の VPN アプライアンスの一覧と選択した VPN アプライアンスを Azure に接続するための構成については、「[サイト間 VPN ゲートウェイ接続用の VPN デバイスと IPsec/IKE パラメーターについて][vpn-appliance]」をご覧ください。

* **VPN 仮想ネットワーク ゲートウェイまたは ExpressRoute ゲートウェイ**。 仮想ネットワーク ゲートウェイでは、オンプレミス ネットワークとの接続に使用する VPN デバイス (ExpressRoute 回線) に VNet を接続できます。 詳しくは、「[Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet]」(Microsoft Azure 仮想ネットワークにオンプレミス ネットワークを接続する) をご覧ください。

> [!NOTE]
> この参照アーキテクチャのデプロイ スクリプトでは、VPN ゲートウェイを使用して接続し、Azure の VNet を使用してオンプレミス ネットワークをシミュレートします。

* **ハブ VNet**。 ハブスポーク トポロジのハブとして使用する Azure VNet。 ハブは、オンプレミス ネットワークへの主要な接続ポイントであり、スポーク VNet でホストされるさまざまなワークロードによって消費できるサービスをホストする場所です。

* **ゲートウェイ サブネット**。 複数の仮想ネットワーク ゲートウェイが同じサブネットに保持されます。

* **共有サービス サブネット**。 DNS や AD DS など、すべてのスポーク間で共有できるサービスをホストするために使用されるハブ VNet 内のサブネットです。

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

このアーキテクチャのデプロイについては、[GitHub][ref-arch-repo] をご覧ください。 このデプロイでは、各 VNet の Ubuntu VM を使用して接続をテストします。 実際のサービスは、**ハブ VNet** の **shared-services** サブネットでホストされません。

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. Azure CLI を使用する方がよい場合は、Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。 CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。

3. PowerShell を使用する方がよい場合は、Azure 用の最新の PowerShell モジュールがコンピューターにインストールされていることを確認してください。 最新の Azure PowerShell モジュールをインストールするには、「[Install PowerShell for Azure][azure-powershell]」(Azure 用の PowerShell をインストールする) の手順に従ってください。

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>シミュレートされたオンプレミスのデータセンターをデプロイする

シミュレートされたオンプレミスのデータセンターを Azure VNet としてデプロイするには、次の手順を実行します。

1. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\onprem` フォルダーに移動します。

2. `onprem.vm.parameters.json` ファイルを開き、次に示す 11 行目と 12 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > 別のリソース グループ名 (`ra-onprem-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。

4. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、Ubuntu を実行する仮想マシン、および VPN ゲートウェイを作成します。 VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。

### <a name="azure-hub-vnet"></a>Azure のハブ VNet

ハブ VNet をデプロイして、上記の手順で作成したシミュレートされたオンプレミスの VNet に接続するには、次の手順を実行します。

1. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\hub` フォルダーに移動します。

2. `hub.vm.parameters.json` ファイルを開き、次に示す 11 行目と 12 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. `hub.gateway.parameters.json` ファイルを開き、次に示す 23 行目の引用符の間に共有キーを入力した後、ファイルを保存します。 この値をメモしておいてください。後からデプロイで使用します。

  ```bash
  "sharedKey": "",
  ```

4. 次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > 別のリソース グループ名 (`ra-hub-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。

5. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、Ubuntu を実行する仮想マシン、VPN ゲートウェイ、および前のセクションで作成したゲートウェイへの接続を作成します。 VPN ゲートウェイの作成の完了には 40 分以上かかる場合があります。

### <a name="connection-from-on-premises-to-the-hub"></a>オンプレミスからハブへの接続

シミュレートされたオンプレミスのデータセンターからハブ VNet に接続するには、次の手順を実行します。

1. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\onprem` フォルダーに移動します。

2. `onprem.connection.parameters.json` ファイルを開き、次に示す 9 行目の引用符の間に共有キーを入力した後、ファイルを保存します。 この共有キーの値は、以前にデプロイしたオンプレミスのゲートウェイで使用されるものと同じにする必要があります。

  ```bash
  "sharedKey": "",
  ```

3. 次の bash または PowerShell コマンドを実行して、シミュレートされたオンプレミスの環境を Azure の VNet としてデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > 別のリソース グループ名 (`ra-onprem-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。

4. デプロイが完了するのを待機します。 このデプロイでは、オンプレミスのデータセンターのシミュレートに使用する VNet とハブ VNet 間の接続を作成します。

### <a name="azure-spoke-vnets"></a>Azure のスポーク VNet

スポーク VNet をデプロイして、上記の手順で作成したハブ VNet に接続するには、次の手順を実行します。

1. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\spokes` フォルダーに切り替えます。

2. `spoke1.web.parameters.json` ファイルを開き、次に示す 53 行目と 54 行目の引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. `spoke2.web.parameters.json` ファイルについて前の手順を繰り返します。

4. 次の bash または PowerShell コマンドを実行して、最初のスポークをデプロイしてハブに接続します。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > 別のリソース グループ名 (`ra-spoke1-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。

5. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、Ubuntu と Apache を実行する 3 つの仮想マシンを含むロード バランサー、VPN ゲートウェイ、および前のセクションで作成したハブ VNet への VNet ピアリング接続を作成します。 このデプロイには 20 分以上かかる場合があります。

6. 次の bash または PowerShell コマンドを実行して、最初のスポークをデプロイしてハブに接続します。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > 別のリソース グループ名 (`ra-spoke2-rg` 以外) を使用する場合は、その名前を使用するすべてのパラメーター ファイルを検索し、独自のリソース グループ名を使用するように編集してください。

5. デプロイが完了するのを待機します。 このデプロイでは、仮想ネットワーク、Ubuntu と Apache を実行する 3 つの仮想マシンを含むロード バランサー、VPN ゲートウェイ、および前のセクションで作成したハブ VNet への VNet ピアリング接続を作成します。 このデプロイには 20 分以上かかる場合があります。

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Azure のスポーク VNet へのハブ VNet のピアリング

ハブ VNet の VNet ピアリング接続をデプロイするには、次の手順を実行します。

1. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\hub` フォルダーに切り替えます。

2. 次の bash または PowerShell コマンドを実行して、最初のスポークにピアリング接続をデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. 次の bash または PowerShell コマンドを実行して、2 番目のスポークにピアリング接続をデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>接続をテストする

オンプレミスのデータセンターに接続されているハブスポーク トポロジが機能していることを確認するには、次の手順を実行します。

1. [Azure Portal][ポータル] からサブスクリプションに接続し、`ra-onprem-rg` リソース グループの `ra-onprem-vm1` 仮想マシンに移動します。

2. `Overview` ブレードで、VM の`Public IP address`をメモします。

3. SSH クライアントを使用して、前の手順でメモした IP アドレスに接続します。この場合、デプロイ時に指定したユーザー名とパスワードを使用します。

4. 接続先の VM のコマンド プロンプトから次のコマンドを実行して、オンプレミスの VNet から Spoke1 VNet への接続をテストします。

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a>スポーク間に接続を追加する

スポークを相互に接続できるようにする場合は、他のスポーク宛てのトラフィックをハブ VNet のゲートウェイに転送する UDR を各スポークにデプロイする必要があります。 次の手順を実行して、現在はスポークから別のスポークに接続できないことを確認します。続いて、UDR をデプロイし、接続をもう一度テストします。

1. ジャンプボックス VM に接続されていない場合は、前のセクションの手順 1 ～ 4 を繰り返します。

2. スポーク 1 でいずれかの Web サーバーに接続します。

  ```bash
  ssh 10.1.1.37
  ```

3. スポーク 1 とスポーク 2 の間の接続をテストします。 この接続は失敗します。

  ```bash
  ping 10.1.2.37
  ```

4. コンピューターのコマンド プロンプトに戻ります。

5. 上記の前提条件でダウンロードしたリポジトリの `hybrid-networking\hub-spoke\spokes` フォルダーに切り替えます。

6. 次の bash または PowerShell コマンドを実行して、最初のスポークに UDR をデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. 次の bash または PowerShell コマンドを実行して、2 番目のスポークに UDR をデプロイします。 値をそれぞれサブスクリプション、リソース グループ名、および Azure リージョンで置き換えてください。

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. SSH ターミナルに戻ります。

9. スポーク 1 とスポーク 2 の間の接続をテストします。 この接続は成功します。

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Hub-spoke topology in Azure" (Azure のハブスポーク トポロジ)
[1]: ./images/hub-spoke-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing" (Azure のハブスポーク トポロジと推移的なルーティング)
[2]: ./images/hub-spoke-no-gateway-routing.svg "Hub-spoke topology in Azure with transitive routing using an NVA" (Azure のハブスポーク トポロジと NVA を使用した推移的なルーティング)
[3]: ./images/hub-spokehub-spoke.svg "Hub-spoke-hub-spoke topology in Azure" (Azure のハブスポークハブスポーク トポロジ)
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
