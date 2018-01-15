---
title: "N 層アーキテクチャの Windows VM を実行する"
description: "可用性、セキュリティ、スケーラビリティ、および管理容易性のセキュリティに特に注意して Azure で多層アーキテクチャを実装する方法について説明します。"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 0654239a5bbd966a2aa776415b7f15ae723ffd63
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>n 層アプリケーションの Windows VM を実行する

この参照アーキテクチャでは、N 層アプリケーションの Windows 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution) 

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ 

N 層アーキテクチャを実装する方法は多数あります。 図は、典型的な 3 層 Web アプリケーションを示しています。 このアーキテクチャは「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」に基づいて作成されています。 Web 層とビジネス層では、負荷分散された VM が使用されます。

* **可用性セット。** 階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。 こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。 可用性セット内の単一の VM をデプロイできますが、単一の VM がすべての OS およびデータ ディスクに Azure Premium Storage を使用していない限り、その単一の VM は SLA 保証に対して適格ではありません。  
* **サブネット。** 階層ごとに個別のサブネットを作成します。 [CIDR] 表記を使用してアドレス範囲とサブネット マスクを指定します。 
* **ロード バランサー**。 [インターネットに接続するロード バランサー][load-balancer-external]を使用して着信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。
* **ジャンプボックス。** [要塞ホスト]とも呼ばれます。 管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。
* **監視。** [Nagios]、[Zabbix]、[Icinga] などの監視ソフトウェアを使用して、応答時間、VM の稼働時間、システムの全体的な正常性に関する洞察を得ることができます。 個別の管理サブネットに配置されている VM 上に監視ソフトウェアをインストールします。
* **NSG。** [ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。
* **SQL Server Always On 可用性グループ。** レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。
* **Active Directory Domain Services (AD DS) サーバー。** Windows Server 2016 に先立って、SQL Server Always On 可用性グループがドメインに参加する必要があります。 これは、可用性グループが Windows Server フェールオーバー クラスター (WSFC) テクノロジに依存するためです。 Windows Server 2016 では Active Directory なしでフェールオーバー クラスターを作成する機能が導入されました。この場合は AD DS サーバーはこのアーキテクチャには不要です。 詳細については、「[What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new]」(Windows Server 2016 でのフェールオーバー クラスタリングの新機能) を参照してください。
* **Azure DNS**。 [Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項は原案として使用してください。 

### <a name="vnet--subnets"></a>VNet/サブネット

VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。 [CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。 標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。

後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。 VNet を作成した後は、アドレス範囲を変更できません。

機能とセキュリティの要件を念頭に置いてサブネットを設計します。 同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。 VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。

各サブネットに対して、サブネットのアドレス空間を CIDR 表記で指定します。 たとえば、"10.0.0.0/24" は IP アドレス 256 個の範囲を作成します。 VM ではこの内の 251 個を使用できます。5 個は予約されています。 アドレス範囲がサブネット間で重複しないことを確認してください。 [Virtual Network に関する FAQ][vnet faq] を参照してください。

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

NSG ルールを使用して階層間のトラフィックを制限します。 たとえば、上の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。 これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。  

1. NSG を作成し、データベース層のサブネットに関連付けます。
2. VNet からのすべての着信トラフィックを拒否するルールを追加します。 (ルール内で `VIRTUAL_NETWORK` タグを使用します。) 
3. ビジネス層のサブネットからの着信トラフィックを許可するルールを、より高い優先順位で追加します。 このルールが前のルールを上書きし、ビジネス層がデータベース層と対話できるようになります。
4. データベース層のサブネット自体からの着信トラフィックを許可するルールを追加します。 このルールによって、データベースのレプリケーションやフェールオーバーに必要な、データベース層内の VM どうしの対話が可能になります。
5. ジャンプボックスのサブネットからの RDP トラフィックを許可するルールを追加します。 このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。
   
   > [!NOTE]
   > NSG の既定のルールでは、VNet 内からの着信トラフィックがすべて許可されます。 これらのルールは削除できませんが、より優先順位の高いルールを作成することで上書きできます。
   > 
   > 

### <a name="load-balancers"></a>ロード バランサー

外部ロード バランサーは、インターネット トラフィックを Web 層に分散します。 このロード バランサー用にパブリック IP アドレスを作成します。 [インターネットに接続するロード バランサーの作成][lb-external-create]に関する記事を参照してください。

内部ロード バランサーは、Web 層からのネットワーク トラフィックをビジネス層に分散します。 このロード バランサーにプライベート IP アドレスを付与するには、フロントエンド IP 構成を作成し、ビジネス層のサブネットに関連付けます。 [内部ロード バランサーの作成][lb-internal-create]に関する記事を参照してください。

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性グループ

SQL Server の高可用性のために [Always On 可用性グループ][sql-alwayson]の使用をお勧めします。 Windows Server 2016 に先立って、Always On 可用性グループはドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ AD ドメイン内にある必要があります。

他の層は[可用性グループ リスナー][sql-alwayson-listeners]を使用してデータベースに接続します。 リスナーを使用することで、SQL クライアントは SQL Server の物理インスタンスの名前を知らなくても接続できます。 データベースにアクセスする VM はドメインに参加している必要があります。 クライアント (ここでは、別の層) は、DNS を使用してリスナーの仮想ネットワーク名を IP アドレスに解決します。

SQL Server Always On 可用性グループを構成する手順は、次のとおりです。

1. Windows Server フェールオーバー クラスタリング (WSFC) クラスター、SQL Server Always On 可用性グループ、プライマリ レプリカを作成します。 詳細については、「[AlwaysOn 可用性グループの概要][sql-alwayson-getting-started]」を参照してください。 
2. 静的プライベート IP アドレスを持つ内部ロード バランサーを作成します。
3. 可用性グループ リスナーを作成し、リスナーの DNS 名を内部ロード バランサーの IP アドレスにマッピングします。 
4. SQL Server リスニング ポート (既定ではTCP ポート 1433) に対するロード バランサーのルールを作成します。 ロード バランサーのルールでは *Floating IP* (Direct Server Return とも呼ばれます) を有効にする必要があります。 これにより VM が直接クライアントに応答でき、プライマリ レプリカへの直接接続が可能になります。
  
  > [!NOTE]
  > Floating IP が有効になっている場合は、フロントエンド ポート番号を、ロード バランサーのルール内のバックエンド ポート番号と同じにする必要があります。
  > 
  > 

SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。 別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーはその後の要求を自動的に新しいプライマリ レプリカにルーティングします。 詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成][sql-alwayson-ilb]に関する記事を参照してください。

フェールオーバー中は、既存のクライアント接続は閉じられます。 フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。

アプリケーションが書き込みよりもはるかに多くの読み取りを行う場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。 「[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]」(リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)) を参照してください。

可用性グループの[手動フェールオーバーの強制][sql-alwayson-force-failover]によってデプロイをテストします。

### <a name="jumpbox"></a>Jumpbox

ジャンプボックスのパフォーマンス要件は最小限に抑えられるため、ジャンプボックスには Standard A1 などの小さな VM のサイズを選択します。 

ジャンプボックス用に[パブリック IP アドレス]を作成します。 ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。

アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。 代わりに、これらの VM へのすべての RDP アクセスは、ジャンプボックスを経由する必要があります。 管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。 ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。

ジャンプボックスをセキュリティで保護するには、NSG を作成してジャンプボックスのサブネットに適用します。 安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。 NSG は、サブネットまたはジャンプボックス NIC のいずれかに関連付けることができます。 この場合は NIC に関連付けることをお勧めします。こうすることで、仮に同じサブネットに別の VM を追加した場合でも、RDP トラフィックはジャンプボックスのみに許可されます。

他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。

## <a name="availability-considerations"></a>可用性に関する考慮事項

データベース層に複数の VM を備えても、自動的にデータベースの可用性が高くなるわけではありません。 リレーショナル データベースの場合は、通常、高可用性を実現するにはレプリケーションとフェールオーバーを使用する必要があります。 SQL Server の場合は、[Always On 可用性グループ][sql-alwayson]の使用をお勧めします。 

[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、2 つのリージョンにアプリケーションをレプリケートし、Azure Traffic Manager を使用してフェールオーバーを行います。 詳細については、「[高可用性のために複数のリージョンで Windows VM を実行する][multi-dc]」を参照してください。   

## <a name="security-considerations"></a>セキュリティに関する考慮事項

機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。 Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。 詳細については、「[Azure VM 上の SQL Server 向け Azure Key Vault 統合の構成][sql-keyvault]」を参照してください。アプリケーション シークレット (データベースの接続文字列など) も Key Vault に格納することをお勧めします。

ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。 NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。 詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

ロード バランサーは、ネットワーク トラフィックを Web 層とビジネス層に分散します。 新しい VM インスタンスを追加することで水平方向にスケーリングします。 負荷に基づいて、Web 層とビジネス層を個別にスケーリングできることに注意してください。 クライアント アフィニティを維持するために発生する可能性がある複雑さを減らすには、Web 層の VM はステートレスである必要があります。 ビジネス ロジックをホストする VM もステートレスである必要があります。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

[Azure Automation][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef]、[Puppet][puppet] などの集中管理ツールを使用することで、システム全体の管理を簡略化します。 これらのツールでは、複数の VM から取り込まれた診断情報と正常性情報を統合して、システムの全体像を提供できます。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。 

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。 CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。

3. [Azure の構成要素][azbb] npm パッケージをインストールします。

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>azbb を使用したソリューションのデプロイ

N 層アプリケーションの参照アーキテクチャで Windows VM をデプロイするには、次の手順に従います。

1. 上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-windows` フォルダーに移動します。

2. このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。 参照アーキテクチャをデプロイする前に、これらを変更する必要があります。 `n-tier-windows.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。
  
  > [!NOTE]
  > このデプロイ中に実行される複数のスクリプトが **VirtualMachineExtension** オブジェクトと、一部の **VirtualMachine** オブジェクトの **extensions** 設定の両方に存在します。 これらのスクリプトのいくつかには、今変更した管理者ユーザー名とパスワードが必要です。 これらのスクリプトをレビューして、正しい資格情報を指定したことを確認することをお勧めします。 正しい資格情報を指定していない場合は、デプロイが失敗する可能性があります。
  > 
  > 

ファイルを保存します。

3. 次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Microsoft Azure を使用した N 層アーキテクチャ"