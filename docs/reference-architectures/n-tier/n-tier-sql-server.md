---
title: SQL Server を使用した Windows N 層アプリケーション
titleSuffix: Azure Reference Architectures
description: 可用性、セキュリティ、スケーラビリティ、および管理容易性のために Azure で多層アーキテクチャを実装します。
author: MikeWasson
ms.date: 11/12/2018
ms.openlocfilehash: 38983dec83718f53fc1ffd79c1347582200f5db0
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120129"
---
# <a name="windows-n-tier-application-on-azure-with-sql-server"></a>SQL Server を使用した Azure の Windows N 層アプリケーション

この参照アーキテクチャでは、Windows 上の SQL Server をデータ層に使用して、N 層アプリケーション用に構成された VM と仮想ネットワークをデプロイする方法を示します。 [**このソリューションをデプロイします**](#deploy-the-solution)。

![Microsoft Azure を使用した N 層アーキテクチャ](./images/n-tier-sql-server.png)

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

アーキテクチャには次のコンポーネントがあります。

- **リソース グループ**。 [リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。

- **仮想ネットワーク (VNet) とサブネット**。 どの Azure VM も、サブネットにセグメント化できる VNet にデプロイされます。 階層ごとに個別のサブネットを作成します。

- **Application Gateway**。 [Azure Application Gateway](/azure/application-gateway/) はレイヤー 7 のロード バランサーです。 このアーキテクチャでは、これによって HTTP 要求が Web フロント エンドにルーティングされます。 Application Gateway には、一般的な脆弱性やその悪用からアプリケーションを保護する [Web アプリケーション ファイアウォール](/azure/application-gateway/waf-overview) (WAF) も用意されています。

- **NSG**。 [ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、VNet 内のネットワーク トラフィックを制限します。 たとえば、ここに示されている 3 層アーキテクチャでは、データベース層は Web フロントエンドからのトラフィックを受信せず、ビジネス層と管理サブネットからのトラフィックのみ受信します。

- **DDoS Protection**。 Azure プラットフォームには分散型サービス拒否 (DDoS) 攻撃に対する基本的な保護が用意されていますが、Microsoft では [DDoS Protection Standard][ddos] を使用することをお勧めします。このサービスでは、DDoS 攻撃を軽減する機能が強化されています。 「[セキュリティに関する考慮事項](#security-considerations)」を参照してください。

- **仮想マシン**。 VM の構成に関する推奨事項については、「[Run a Windows VM on Azure (Azure での Windows VM の実行)](./windows-vm.md)」および「[Run a Linux VM on Azure (Azure での Linux VM の実行)](./linux-vm.md)」をご覧ください。

- **可用性セット**。 階層ごとに[可用性セット][azure-availability-sets]を作成し、各階層に 2 つ以上の VM をプロビジョニングします。これにより、VM がさらに高度な[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。

- **ロード バランサー**。 [Azure Load Balancer][load-balancer] は、Web 層からビジネス層へ、ビジネス層から SQL Server へとネットワーク トラフィックを分散するために使用します。

- **パブリック IP アドレス**。 パブリック IP アドレスは、アプリケーションがインターネット トラフィックを受信するために必要です。

- **Jumpbox**。 [要塞ホスト]とも呼ばれます。 管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 ジャンプボックスの NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。

- **SQL Server Always On 可用性グループ**。 レプリケーションとフェールオーバーを有効にすることで、データ層で高い可用性を提供します。 これは、Windows Server フェールオーバー クラスター (WSFC) テクノロジを使用してフェールオーバーを行います。

- **Active Directory Domain Services (AD DS) サーバー。** フェールオーバー クラスターと、これに関連するクラスター化されたロールを表すコンピューター オブジェクトは、Active Directory Domain Services (AD DS) に作成されます。

- **クラウド監視**。 フェールオーバー クラスターでは、そのノードの半数以上が実行されている必要があり、"クォーラムに達している" と呼びます。 クラスターにノードが 2 つしかない場合は、ネットワークのパーティションにより、各ノードは自身がマスター ノードであると認識する可能性があります。 この場合、"*監視*" によって優先順位を決定し、クォーラムを確立する必要があります。 監視は、クォーラムを確立する際の優先順位決定者として動作可能な、共有ディスクなどのリソースです。 クラウド監視は、Azure Blob Storage を使用する一種の監視です。 クォーラムの概念の詳細については、「[Understanding cluster and pool quorum (クラスターとプール クォーラムについて)](/windows-server/storage/storage-spaces/understand-quorum)」を参照してください。 クラウド監視の詳細については、「[Deploy a cloud witness for a Failover Cluster (フェールオーバー クラスター用にクラウド監視をデプロイする)](/windows-server/failover-clustering/deploy-cloud-witness)」を参照してください。

- **Azure DNS**。 [Azure DNS][azure-dns] は DNS ドメインのホスティング サービスです。 これは Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項を開始点として使用してください。

### <a name="vnet--subnets"></a>VNet/サブネット

VNet を作成するときに、各サブネット内のリソースが要求する IP アドレスの数を決定します。 [CIDR] 表記を使用して、必要な IP アドレスにとって十分な規模のサブネット マスクと VNet アドレス範囲を指定します。 標準的な[プライベート IP アドレス ブロック][private-ip-space] (10.0.0.0/8、172.16.0.0/12、192.168.0.0/16) の範囲内にあるアドレス空間を使用します。

後で VNet とオンプレミスのネットワークとの間にゲートウェイを設定する必要がある場合は、オンプレミスのネットワークと重複しないアドレス範囲を選択します。 VNet を作成した後は、アドレス範囲を変更できません。

機能とセキュリティの要件を念頭に置いてサブネットを設計します。 同じ層または同じロール内のすべての VM は、同じサブネットに入れる必要があります。これがセキュリティ境界になります。 VNet とサブネットの設計に関する詳細については、「[Azure Virtual Network の計画と設計][plan-network]」を参照してください。

### <a name="load-balancers"></a>ロード バランサー

VM は直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。 クライアントは、Application Gateway に関連付けられているパブリック IP アドレスを使用して接続します。

ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。 たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマップします。 クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。 クライアント要求が、バックエンド アドレス プールのすべての VM に分散されます。

### <a name="network-security-groups"></a>ネットワーク セキュリティ グループ

NSG ルールを使用して階層間のトラフィックを制限します。 上記の 3 層アーキテクチャでは、Web 層はデータベース層と直接通信しません。 これを強制するには、データベース層が Web 層のサブネットからの着信トラフィックをブロックする必要があります。

1. VNet からのすべての受信トラフィックを拒否します。 (ルール内で `VIRTUAL_NETWORK` タグを使用します。)
2. ビジネス層のサブネットからの受信トラフィックを許可します。
3. データベース層のサブネットからの受信トラフィックを許可します。 このルールにより、データベースのレプリケーションやフェールオーバーに必要な、データベース VM 間の通信が可能になります。
4. ジャンプボックスのサブネットからの RDP トラフィック (ポート 3389) を許可します。 このルールによって、管理者がジャンプボックスからデータベース層に接続できるようにします。

最初のルールよりも高い優先順位を設定してルール 2 から 4 を作成し、最初のルールをオーバーライドします。

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

SQL クライアントが接続を試みると、ロード バランサーがプライマリ レプリカに接続要求をルーティングします。 別のレプリカへのフェールオーバーが発生した場合は、ロード バランサーは新しい要求を自動的に新しいプライマリ レプリカにルーティングします。 詳細については、[SQL Server Always On 可用性グループの ILB リスナーの構成][sql-alwayson-ilb]に関する記事を参照してください。

フェールオーバー中は、既存のクライアント接続は閉じられます。 フェールオーバーが完了すると、新しい接続は新しいプライマリ レプリカにルーティングされます。

アプリケーションが書き込みよりもはるかに多くの読み取りを行う場合は、読み取り専用クエリの一部をセカンダリ レプリカにオフロードできます。 「[Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing]」(リスナーを使用した読み取り専用セカンダリ レプリカへの接続 (読み取り専用ルーティング)) を参照してください。

可用性グループの[手動フェールオーバーの強制][sql-alwayson-force-failover]によってデプロイをテストします。

### <a name="jumpbox"></a>Jumpbox

アプリケーション ワークロードを実行する VM へのパブリック インターネットからの RDP アクセスを許可しないでください。 代わりに、これらの VM へのすべての RDP アクセスは、ジャンプボックスを経由する必要があります。 管理者はジャンプボックスにログインし、次にジャンプボックスから他の VM にログインします。 ジャンプボックスは、既知の安全な IP アドレスからのみ、インターネットからの RDP トラフィックを許可します。

ジャンプボックスのパフォーマンス要件は最小限に抑えられているので、小さな VM サイズを選択します。 ジャンプボックス用に[パブリック IP アドレス]を作成します。 ジャンプボックスを、他の VM と同じ VNet 内の、個別の管理サブネット内に配置します。

ジャンプボックスをセキュリティで保護するには、安全な一連のパブリック IP アドレスからのみ RDP 接続を許可する NSG ルールを追加します。 他のサブネットに対しても NSG を構成して、管理サブネットからの RDP トラフィックを許可します。

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

ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。 タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。 ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。

ロード バランサーの正常性プローブには、次のような推奨事項があります。

- プローブでは、HTTP または TCP のいずれかをテストできます。 VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。 それ以外の場合は、TCP プローブを作成します。
- HTTP プローブでは、HTTP エンドポイントへのパスを指定します。 プローブでは、このパスからの HTTP 200 の応答をチェックします。 このパスには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。 エンドポイントでは、匿名の HTTP 要求を許可する必要があります。
- このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。 任意のファイアウォール ポリシーまたは NSG ルールで、この IP アドレスとの間の送受信トラフィックをブロックしないでください。
- [正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。 各ロード バランサーに対して Azure Portal のログ記録を有効にします。 ログは Azure Blob Storage に書き込まれます。 ログには、プローブの応答に失敗したために、ネットワーク トラフィックを取得していない VM 数が表示されます。

[VM の Azure SLA][vm-sla] によって提供されるものよりも高い可用性が必要な場合は、フェールオーバーに Azure Traffic Manager を使用して、2 つのリージョンでのアプリケーションのレプリケーションを検討します。 詳細については、「[高可用性のためのマルチリージョン n 層アプリケーション][multi-dc]」を参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

仮想ネットワークは、Azure のトラフィックの分離境界です。 ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。 トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。 詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。

**DMZ**。 ネットワーク仮想アプライアンス (NVA) を追加してインターネットと Azure Virtual Network の間の DMZ を作成することを検討してください。 NVA とは、ネットワーク関連のタスク (ファイアウォール、パケット インスペクション、監査、カスタム ルーティングなど) を実行できる仮想アプライアンスの総称です。 詳細については、[Azure とインターネットの間の DMZ の実装][dmz]に関する記事を参照してください。

**暗号化**。 機密の保存データを暗号化し、[Azure Key Vault][azure-key-vault] を使用してデータベース暗号化キーを管理します。 Key Vault では、ハードウェア セキュリティ モジュール (HSM) に暗号化キーを格納することができます。 詳細については、[Azure VM 上の SQL Server 向け Azure Key Vault 統合の構成][sql-keyvault]に関するページを参照してください。 データベース接続文字列などのアプリケーション シークレットも Key Vault に格納することをお勧めします。

**DDoS 保護**。 Azure プラットフォームには、基本的な DDoS 保護が既定で用意されています。 この基本的な保護は、Azure インフラストラクチャ全体を保護することを目的としています。 基本的な DDoS 保護は自動的に有効になっていますが、Microsoft では [DDoS Protection Standard][ddos] を使用することをお勧めします。 Standard Protection では、アプリケーションのネットワーク トラフィック パターンに基づいて、アダプティブ チューニングが使用され、脅威が検出されます。 これにより、インフラストラクチャ全体の DDoS ポリシーで見落とされてしまう可能性のある DDoS 攻撃に対して、軽減策を適用することができます。 Standard Protection では、Azure Monitor を介して、アラート、テレメトリ、および分析機能も提供されます。 詳細については、[Azure DDoS Protection:ベスト プラクティスと参照アーキテクチャ][ddos-best-practices]に関するページを参照してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このリファレンス アーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。 デプロイ全体を完了するには最大 2 時間かかる場合があります。これには、AD DS、Windows Server フェールオーバー クラスター、および SQL Server 可用性グループを構成するスクリプトの実行時間が含まれます。

### <a name="prerequisites"></a>前提条件

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deployment-steps"></a>デプロイメントの手順

1. 次のコマンドを実行して、リソース グループを作成します。

    ```azurecli
    az group create --location <location> --name <resource-group-name>
    ```

2. 次のコマンドを実行して、クラウド監視のストレージ アカウントを作成します。

    ```azurecli
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. 参照アーキテクチャ GitHub リポジトリの `virtual-machines\n-tier-windows` フォルダーに移動します。

4. `n-tier-windows.json` ファイルを開きます。

5. "witnessStorageBlobEndPoint" のすべてのインスタンスを検索し、プレース ホルダー テキストを手順 2. のストレージ アカウントの名前で置き換えます。

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. 次のコマンドを実行して、ストレージ アカウントのアカウント キーを一覧表示します。

    ```azurecli
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    出力は次のようになります。 `key1` の値をコピーします。

    ```json
    [
    {
        "keyName": "key1",
        "permissions": "Full",
        "value": "..."
    },
    {
        "keyName": "key2",
        "permissions": "Full",
        "value": "..."
    }
    ]
    ```

7. `n-tier-windows.json` ファイルで、"witnessStorageAccountKey" のすべてのインスタンスを検索し、アカウント キーに貼り付けます。

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. `n-tier-windows.json` ファイルで、`[replace-with-password]` と `[replace-with-sql-password]` のすべてのインスタンスを検索し、そのインスタンスを強力なパスワードに置き換えます。 ファイルを保存します。

    > [!NOTE]
    > 管理者ユーザー名を変更する場合は、JSON ファイルの `extensions` ブロックも更新する必要があります。

9. 次のコマンドを実行して、アーキテクチャをデプロイします。

    ```azurecli
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

Azure の構成要素を使用してこのサンプルの参照アーキテクチャをデプロイする方法の詳細については、「[GitHub リポジトリ][git]」を参照してください。

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[要塞ホスト]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[nsg]: /azure/virtual-network/virtual-networks-nsg
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[パブリック IP アドレス]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
