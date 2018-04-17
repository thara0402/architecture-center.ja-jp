---
title: Azure で n 層アプリケーションの Linux VM を実行する
description: Microsoft Azure で N 層アーキテクチャの Linux VM を実行する方法について説明します。
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 8d3e6e5124a0abb27a3c72e1ecbd52a1a1da2a33
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>N 層アプリケーションの Linux VM を実行する

この参照アーキテクチャでは、N 層アプリケーションの Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)  

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

N 層アーキテクチャを実装する方法は多数あります。 図は、典型的な 3 層 Web アプリケーションを示しています。 このアーキテクチャは「[スケーラビリティと可用性のために負荷分散された VM を実行する][multi-vm]」に基づいて作成されています。 Web 層とビジネス層では、負荷分散された VM が使用されます。

* **可用性セット。** 階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に少なくとも 2 つの VM をプロビジョニングします。  こうすると VM がより高度な VM の[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。 可用性セット内の単一の VM をデプロイできますが、単一の VM がすべての OS およびデータ ディスクに Azure Premium Storage を使用していない限り、その単一の VM は SLA 保証に対して適格ではありません。  
* **サブネット。** 階層ごとに個別のサブネットを作成します。 [CIDR] 表記を使用してアドレス範囲とサブネット マスクを指定します。 
* **ロード バランサー**。 [インターネットに接続するロード バランサー][load-balancer-external]を使用して着信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。
* **Azure DNS**。 [Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。
* **ジャンプボックス。** [要塞ホスト]とも呼ばれます。 管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG は Secure Shell (SSH) トラフィックを許可する必要があります。
* **監視。** [Nagios]、[Zabbix]、[Icinga] などの監視ソフトウェアを使用して、応答時間、VM の稼働時間、システムの全体的な正常性に関する洞察を得ることができます。 個別の管理サブネットに配置されている VM 上に監視ソフトウェアをインストールします。
* <strong>NSG。</strong> [ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。
* **Apache Cassandra データベース**。 レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。

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
5. ジャンプボックスのサブネットからの SSH トラフィックを許可するルールを追加します。 このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。
   
   > [!NOTE]
   > NSG の[既定のルール][nsg-rules]では、VNet 内からの着信トラフィックがすべて許可されます。 これらのルールは削除できませんが、より優先順位の高いルールを作成することで上書きできます。
   > 
   > 

### <a name="load-balancers"></a>ロード バランサー

外部ロード バランサーは、インターネット トラフィックを Web 層に分散します。 このロード バランサー用にパブリック IP アドレスを作成します。 [インターネットに接続するロード バランサーの作成][lb-external-create]に関する記事を参照してください。

内部ロード バランサーは、Web 層からのネットワーク トラフィックをビジネス層に分散します。 このロード バランサーにプライベート IP アドレスを付与するには、フロントエンド IP 構成を作成し、ビジネス層のサブネットに関連付けます。 [内部ロード バランサーの作成][lb-internal-create]に関する記事を参照してください。

### <a name="cassandra"></a>Cassandra

運用環境では [DataStax Enterprise][datastax] の使用をお勧めしますが、これらの推奨事項はすべての Cassandra エディションに適用されます。 Azure での DataStax の実行の詳細については、「[Azure 用の DataStax Enterprise Deployment ガイド][cassandra-in-azure]」を参照してください。 

Cassandra クラスター用の VM を可用性セット内に配置して、Cassandra レプリカが複数の障害ドメインおよびアップグレード ドメイン間に分散されることを保証します。 障害ドメインおよびアップグレード ドメインの詳細については、「[Virtual Machines の可用性管理][azure-availability-sets]」を参照してください。 

可用性セットごとに 3 個の障害ドメイン (最大数) と 18 個のアップグレード ドメインを構成します。 これは、障害ドメイン間で引き続き均等に分散できるアップグレード ドメインの最大数を提供します。   

ラック認識モードでノードを構成します。 `cassandra-rackdc.properties` ファイル内で障害ドメインをラックにマッピングします。

クラスターの前にロード バランサーは必要ありません。 クライアントはクラスターのノードに直接接続します。

### <a name="jumpbox"></a>Jumpbox

ジャンプボックスのパフォーマンス要件は最小限に抑えられるため、ジャンプボックスには Standard A1 などの小さな VM のサイズを選択します。 

ジャンプボックス用に[パブリック IP アドレス]を作成します。 ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。

アプリケーション ワークロードを実行する VM へのパブリック インターネットからの SSH アクセスを許可しないでください。 代わりに、これらの VM へのすべての SSH アクセスは、ジャンプボックスを経由する必要があります。 管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。 ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの SSH トラフィックを許可します。

ジャンプボックスをセキュリティで保護するには、NSG を作成してジャンプボックスのサブネットに適用します。 安全な一連のパブリック IP アドレスからのみ SSH 接続を許可する NSG ルールを追加します。 NSG は、サブネットまたはジャンプボックスの NIC のいずれかに関連付けることができます。 この場合は NIC に関連付けることをお勧めします。こうすることで、仮に同じサブネットに別の VM を追加した場合でも、SSH トラフィックはジャンプボックスのみに許可されます。

他のサブネットに対しても NSG を構成して、管理サブネットからの SSH トラフィックを許可します。

## <a name="availability-considerations"></a>可用性に関する考慮事項

各層または各 VM ロールを、個別の可用性セットに配置します。 

データベース層に複数の VM を備えても、自動的にデータベースの可用性が高くなるわけではありません。 リレーショナル データベースの場合は、通常、高可用性を実現するにはレプリケーションとフェールオーバーを使用する必要があります。  

[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、2 つのリージョンにアプリケーションをレプリケートし、Azure Traffic Manager を使用してフェールオーバーを行います。 詳細については、「[高可用性を得るために複数のリージョンで Linux VM を実行する][multi-dc]」を参照してください。  

## <a name="security-considerations"></a>セキュリティに関する考慮事項

ネットワーク仮想アプライアンス (NVA) を追加してパブリック インターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。 NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。 詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

ロード バランサーは、ネットワーク トラフィックを Web 層とビジネス層に分散します。 新しい VM インスタンスを追加することで水平方向にスケーリングします。 負荷に基づいて、Web 層とビジネス層を個別にスケーリングできることに注意してください。 クライアント アフィニティを維持するために発生する可能性がある複雑さを減らすには、Web 層の VM はステートレスである必要があります。 ビジネス ロジックをホストする VM もステートレスである必要があります。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

[Azure Automation][azure-administration]、[Microsoft Operations Management Suite][operations-management-suite]、[Chef][chef]、[Puppet][puppet] などの集中管理ツールを使用することで、システム全体の管理を簡略化します。 これらのツールでは、複数の VM から取り込まれた診断情報と正常性情報を統合して、システムの全体像を提供できます。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。 

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

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

N 層アプリケーションの参照アーキテクチャで Linux VM をデプロイするには、次の手順に従います。

1. 上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-linux` フォルダーに移動します。

2. このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。 参照アーキテクチャをデプロイする前に、これらを変更する必要があります。 `n-tier-linux.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。   ファイルを保存します。

3. 次に示すように、**azbb** コマンド ライン ツールを使用して参照アーキテクチャをデプロイします。

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "Microsoft Azure を使用した N 層アーキテクチャ"

