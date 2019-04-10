---
title: Apache Cassandra を使用する N 層アプリケーション
titleSuffix: Azure Reference Architectures
description: Microsoft Azure で Apache Cassandra を使用して N 層アーキテクチャの Linux 仮想マシンを実行します。
author: MikeWasson
ms.date: 11/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: df7c638f5ebeedcd4e04574afb655b1ac5ea65c8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249267"
---
# <a name="linux-n-tier-application-in-azure-with-apache-cassandra"></a>Apache Cassandra を使用する Azure の Linux N 層アプリケーション

この参照アーキテクチャでは、Linux 上の Apache Cassandra をデータ層に使用して、[N 層](../../guide/architecture-styles/n-tier.md)アプリケーション用に構成された仮想マシン (VM) と仮想ネットワークをデプロイする方法を示します。 [**このソリューションをデプロイします**](#deploy-the-solution)。

![Microsoft Azure を使用した N 層アーキテクチャ](./images/n-tier-cassandra.png)

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

アーキテクチャには次のコンポーネントがあります。

- **リソース グループ**。 [リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。

- **仮想ネットワーク (VNet) とサブネット**。 どの Azure VM も、サブネットにセグメント化できる VNet にデプロイされます。 階層ごとに個別のサブネットを作成します。

- **NSG**。 [ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は、ビジネス層と管理サブネットからトラフィックを受信します。Web フロントエンドからは受信しません。

- **DDoS Protection**。 Azure プラットフォームには分散型サービス拒否 (DDoS) 攻撃に対する基本的な保護が用意されていますが、Microsoft では [DDoS Protection Standard][ddos] を使用することをお勧めします。このサービスでは、DDoS 攻撃を軽減する機能が強化されています。 「[セキュリティに関する考慮事項](#security-considerations)」を参照してください。

- **仮想マシン**。 VM の構成に関する推奨事項については、「[Run a Windows VM on Azure (Azure での Windows VM の実行)](./windows-vm.md)」および「[Run a Linux VM on Azure (Azure での Linux VM の実行)](./linux-vm.md)」をご覧ください。

- **可用性セット**。 階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に 2 つ以上の VM をプロビジョニングします。これにより、VM がさらに高度な[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。

- **Azure ロード バランサー**。 [ロード バランサー][load-balancer]は、受信インターネット要求を各 VM インスタンスに分散します。 [パブリック ロード バランサー][load-balancer-external]を使用して受信インターネット トラフィックを Web 層に分散し、[内部ロード バランサー][load-balancer-internal]を使用して Web 層からのネットワーク トラフィックをビジネス層に分散します。

- **パブリック IP アドレス**。 パブリック IP アドレスは、パブリック ロード バランサーがインターネット トラフィックを受信するために必要です。

- **Jumpbox**。 [要塞ホスト]とも呼ばれます。 管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 jumpbox の NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG では SSH トラフィックを許可する必要があります。

- **Apache Cassandra データベース**。 レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。

- **Azure DNS**。 [Azure DNS][azure-dns] は DNS ドメインのホスティング サービスです。 これは Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項を開始点として使用してください。

### <a name="vnet--subnets"></a>VNet/サブネット

VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。 [CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。 標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。

後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。 VNet を作成した後は、アドレス範囲を変更できません。

機能とセキュリティの要件を念頭に置いてサブネットを設計します。 同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。 VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。

### <a name="load-balancers"></a>ロード バランサー

VM は直接インターネットに公開しないでください。 代わりに、各 VM にプライベート IP アドレスを付与します。 クライアントは、パブリック ロード バランサーの IP アドレスを使用して接続します。

ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。 たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。 クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。 クライアント要求がすべての VM に分散されます。

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

NSG ルールを使用して階層間のトラフィックを制限します。 たとえば、上記の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。 これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。

1. VNet からのすべての受信トラフィックを拒否します。 (ルール内で `VIRTUAL_NETWORK` タグを使用します。)
2. ビジネス層のサブネットからの受信トラフィックを許可します。
3. データベース層のサブネットからの受信トラフィックを許可します。 このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。
4. ジャンプボックスのサブネットからの SSH トラフィック (ポート 22) を許可します。 このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。

最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。

### <a name="cassandra"></a>Cassandra

運用環境では [DataStax Enterprise][datastax] の使用をお勧めしますが、これらの推奨事項はすべての Cassandra エディションに適用されます。 Azure での DataStax の実行の詳細については、「[Azure 用の DataStax Enterprise Deployment ガイド][cassandra-in-azure]」を参照してください。

Cassandra クラスター用の VM を可用性セット内に配置して、Cassandra レプリカが複数の障害ドメインおよびアップグレード ドメイン間に分散されることを保証します。 障害ドメインおよびアップグレード ドメインの詳細については、「[Virtual Machines の可用性管理][azure-availability-sets]」を参照してください。

可用性セットごとに 3 個の障害ドメイン (最大数) と 18 個のアップグレード ドメインを構成します。 これは、障害ドメイン間で引き続き均等に分散できるアップグレード ドメインの最大数を提供します。

ラック認識モードでノードを構成します。 `cassandra-rackdc.properties` ファイル内で障害ドメインをラックにマッピングします。

クラスターの前にロード バランサーは必要ありません。 クライアントはクラスターのノードに直接接続します。

高可用性を確保するために、複数の Azure リージョンに Cassandra をデプロイします。 各リージョンのノードが、リージョン内の回復性を高めるために、障害ドメインとアップグレード ドメインを持つラック認識モードで構成されます。

### <a name="jumpbox"></a>Jumpbox

アプリケーション ワークロードを実行する VM へのパブリック インターネットからの SSH アクセスを許可しないでください。 代わりに、これらの VM へのすべての SSH アクセスは、ジャンプボックスを経由する必要があります。 管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。 ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの SSH トラフィックを許可します。

ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。 ジャンプボックス用に[パブリック IP アドレス]を作成します。 ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。

ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ SSH 接続を許可する NSG ルールを追加します。 管理サブネットからの SSH トラフィックを許可するように、他のサブネットの NSG を構成します。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

Web 層とビジネス層については、個別の VM を可用性セットにデプロイするのではなく、[仮想マシン スケール セット][vmss]を使用することを検討してください。 スケール セットにより、同一の VM のセットを簡単にデプロイおよび管理し、パフォーマンス メトリックに基づいて VM を自動スケーリングできるようになります。 VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。 VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。

スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。

- デプロイ後に、拡張機能を使用して VM を構成します。 この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。

- カスタム ディスク イメージと共に[マネージド ディスク](/azure/storage/storage-managed-disks-overview)をデプロイします。 このオプションの方が早くデプロイできる場合があります。 ただし、イメージを最新の状態に保つ必要があります。

詳細については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。

> [!TIP]
> 自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。

各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。 サポート リクエストを提出することで、制限値を上げることができます。 詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

仮想マシン スケール セットを使用しない場合は、同じ層の VM を可用性セットに配置します。 [Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。 詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。 スケール セットでは "*配置グループ*" が自動的に使用されます。これは、暗黙的な可用性セットとして機能します。

ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。 タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。 ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。

ロード バランサーの正常性プローブには、次のような推奨事項があります。

- プローブでは、HTTP または TCP のいずれかをテストできます。 VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。 それ以外の場合は、TCP プローブを作成します。
- HTTP プローブでは、HTTP エンドポイントへのパスを指定します。 プローブでは、このパスからの HTTP 200 の応答をチェックします。 これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。 エンドポイントでは、匿名の HTTP 要求を許可する必要があります。
- このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。 任意のファイアウォール ポリシーまたは NSG ルールで、この IP アドレスを送受信するトラフィックをブロックしていないことを確認します。
- [正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。 各ロード バランサーに対して Azure Portal のログ記録を有効にします。 ログは Azure Blob Storage に書き込まれます。 ログには、プローブの応答に失敗したために、ネットワーク トラフィックを取得していない VM 数が表示されます。

Cassandra クラスターの場合、フェールオーバー シナリオは、アプリケーションで使用される一貫性レベルと、レプリカの数によって異なります。 Cassandra での一貫性レベルと使用については、[データの一貫性の構成][cassandra-consistency]および [Cassandra:Quorum を使用してトークできるノードの数][cassandra-consistency-usage]に関するページを参照してください。 Cassandra でのデータの可用性は、アプリケーションで使用される一貫性レベルとレプリケーション メカニズムによって決まります。 Cassandra のレプリケーションについては、「[Data Replication in NoSQL Databases Explained][cassandra-replication]」(NoSQL Database でのデータレプリケーションの説明) を参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

仮想ネットワークは、Azure のトラフィックの分離境界です。 ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。 トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。 詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。

インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。 しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。

**DMZ**。 ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。 NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。 詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。

**暗号化**。 機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。 Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。 データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。

**DDoS 保護**。 Azure プラットフォームには、基本的な DDoS 保護が既定で用意されています。 この基本的な保護は、Azure インフラストラクチャ全体を保護することを目的としています。 基本的な DDoS 保護は自動的に有効になっていますが、Microsoft では [DDoS Protection Standard][ddos] を使用することをお勧めします。 Standard Protection では、アプリケーションのネットワーク トラフィック パターンに基づいて、アダプティブ チューニングが使用され、脅威が検出されます。 これにより、インフラストラクチャ全体の DDoS ポリシーで見落とされてしまう可能性のある DDoS 攻撃に対して、軽減策を適用することができます。 Standard Protection では、Azure Monitor を介して、アラート、テレメトリ、および分析機能も提供されます。 詳細については、「[Azure DDoS Protection:ベスト プラクティスと参照アーキテクチャ][ddos-best-practices]に関するページを参照してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。

### <a name="prerequisites"></a>前提条件

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>azbb を使用したソリューションのデプロイ

N 層アプリケーションの参照アーキテクチャで Linux VM をデプロイするには、次の手順に従います。

1. 上の前提条件の手順 1 で複製したリポジトリの `virtual-machines\n-tier-linux` フォルダーに移動します。

2. このパラメーター ファイルは、デプロイ内の各 VM の既定の管理者ユーザー名とパスワードを指定します。 参照アーキテクチャをデプロイする前に、これらを変更してください。 `n-tier-linux.json` ファイルを開き、各 **adminUsername** および **adminPassword** フィールドを新しい設定に置き換えます。   ファイルを保存します。

3. 次に示すように、**azbb** ツールを使用して参照アーキテクチャをデプロイします。

   ```azurecli
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。

## <a name="next-steps"></a>次の手順

- [Microsoft Learn モジュール:N 層アーキテクチャ スタイルのツアー](/learn/modules/n-tier-architecture/)

<!-- links -->

[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[datastax]: https://www.datastax.com/products/datastax-enterprise
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
