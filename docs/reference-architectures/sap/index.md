---
title: Azure での SAP NetWeaver および SAP HANA のデプロイ
description: Azure の高可用性環境で SAP HANA を実行するための実証済みのプラクティス
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 33171164c59a520a87ef3209c5bb1b208377221c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2018
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Azure での SAP NetWeaver および SAP HANA のデプロイ

この参照用アーキテクチャは、Azure の高可用性環境で SAP HANA を実行するための実証済みのプラクティスのセットを示しています。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

![0][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

> [!NOTE]
> この参照用アーキテクチャをデプロイするには、SAP 製品と他の Microsoft 以外のテクノロジの適切なライセンスが必要です。 Microsoft と SAP のパートナーシップに関する情報については、「[SAP HANA on Azure][sap-hana-on-azure]」をご覧ください。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

- **仮想ネットワーク (VNet)**。 VNet は、Azure の論理的に分離されたネットワークの表現です。 この参照用アーキテクチャ内のすべての VM は、同じ VNet にデプロイされます。 VNet はさらにサブネットに分割されます。 アプリケーション (SAP NetWeaver)、データベース (SAP HANA)、管理 (Jumpbox)、および Active Directory など、階層ごとに別個のサブネットを作成します。

- **仮想マシン (VM)**。 このアーキテクチャの VM は、複数の個別の階層にグループ化されます。

    - **SAP NetWeaver**。 SAP ASCS、SAP Web Dispatcher、および SAP アプリケーション サーバーを含みます。 
    
    - **SAP HANA**。 この参照用アーキテクチャは、データベース層の SAP HANA を使用して、単一の [GS5] [ vm-sizes-mem]インスタンスで実行されます。 SAP HANA は、GS5 または [Azure ラージ インスタンスの SAP HANA][azure-large-instances] 上にある運用 OLAP ワークロードに対して認定されています。 この参照用アーキテクチャは、G シリーズおよび M シリーズの Azure 仮想マシン向けです。 Azure ラージ インスタンスの SAP HANA に関する情報については、「[SAP HANA on Azure (L インスタンス) の概要とアーキテクチャ][azure-large-instances]」をご覧ください。
   
    - **Jumpbox**。 要塞ホストとも呼ばれます。 これは、他の VM に接続するために管理者が使用するネットワークの安全な VM です。 
     
    - **Windows Server Active Directory (AD) ドメイン コントローラー**。 ドメイン コント ローラーは、Windows Server フェールオーバー クラスター (下記参照) を構成するために使用されます。
 
- **可用性セット**。 SAP Web Dispatcher、SAP アプリケーション サーバー、および SAP ACSC ロールの VM を別個の可用性セットに配置し、各ロールには少なくとも 2 つの VM をプロビジョニングします。 こうすると VM が高度なサービス レベル アグリーメント (SLA) に対応できるようになります。
    
- **NIC**。 SAP NetWeaver および SAP HANA を実行する VM は、2 つのネットワーク インターフェイス (NIC) を必要とします。 各 NIC は、異なる種類のトラフィックを分離するために、別のサブネットに割り当てられます。 詳細については、後述する「[推奨事項](#recommendations)」をご覧ください。

- **Windows Server フェールオーバー クラスター**。 SAP ACSC を実行している VM は、高可用性のためにフェールオーバー クラスターとして構成されます。 フェールオーバー クラスターをサポートするために、SIOS DataKeeper クラスター エディションは、クラスター ノードが所有する独立したディスクをレプリケートすることによって、クラスターの共有ボリューム (CSV) 機能を実行します。 詳細については、「[Running SAP applications on the Microsoft platform (Microsoft platform での SAP アプリケーションの実行)][running-sap]」をご覧ください。
    
- **ロード バランサー**。 2 つ [Azure Load Balancer] [ azure-lb]インスタンスが使用されます。 図の左側にある 1 つ目のバランサーは、SAP Web Dispatcher の VM にトラフィックを分散させます。 この構成は、「[High Availability of the SAP Web Dispatcher (SAP Web Dispatcher の高可用性)][sap-dispatcher-ha]」で説明されている平行 Web Dispatcher を実装しています。 右側にある 2 つ目のバランサーは、アクティブかつ正常なノードへの受信接続を転送することで、Windows Server フェールオーバー クラスターでのフェールオーバーを可能にします。

- **VPN Gateway**。 VPN Gateway では、オンプレミス ネットワークを Azure VNet に拡張します。 ExpressRoute を使用することも可能です。その場合は、パブリック インターネットを通過しない専用のプライベート接続が使用されます。 サンプルのソリューションでは、ゲートウェイをデプロイしていません。 詳細については、「[オンプレミス ネットワークの Azure への接続][hybrid-networking]」をご覧ください。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項は原案として使用してください。

### <a name="load-balancers"></a>ロード バランサー

[SAP Web Dispatcher] [ sap-dispatcher]は、デュアル スタック サーバー (ABAP と Java) への HTTP(S) トラフィックの負荷分散を処理します。 SAP では長年、シングルスタック アプリケーション サーバーを提唱してきたため、デュアルスタック デプロイメント モデルで実行されるアプリケーションは現在、ほとんどありません。 アーキテクチャの図に示された Azure Load Balancer は、SAP Dispatcher の高可用性クラスターを実装します。

アプリケーション サーバーへのトラフィックの負荷分散は、SAP 内で処理されます。 DIAG およびリモート ファンクション コール (RFC) で SAP サーバーに接続する SAPGUI クライアントからのトラフィックの場合、SCS メッセージ サーバーは SAP App Server の[ログオン グループ][logon-groups]を作成して、負荷を分散します。 

SMLG は、SAP セントラル サービスのログオンの負荷分散機能の管理に使用される SAP ABAP トランザクションです。 ログオン グループのバックエンド プールには、複数の ABAP アプリケーション サーバーがあります。 ASCS クラスター サービスにアクセスするクライアントは、フロントエンド IP アドレスを使って、Azure Load Balancer に接続します。 ASCS クラスター仮想ネットワークの名前には、IP アドレスも含まれています。 必要に応じて、クラスターをリモートで管理できるように、このアドレスは Azure Load Balancer の追加の IP アドレスに関連付けできます。  

### <a name="nics"></a>NIC

SAP ランドスケープ管理機能では、さまざまな NIC 上のサーバー トラフィックの分離が必要です。 たとえば、ビジネス データは、管理トラフィックおよびバックアップ トラフィックとは分離する必要があります。 複数の NIC を別のサブネットに割り当てると、このデータ分離が可能です。 詳細については、「[Building High Availability for SAP NetWeaver and SAP HANA (SAP NetWeaver および SAP HANA の高可用性の構築)][sap-ha]」(PDF) の「Network (ネットワーク)」をご覧ください。

管理 NIC を管理サブネットに割り当て、データ通信 NIC を別のサブネットに割り当てます。 構成の詳細については、「[複数の NIC を持つ Windows 仮想マシンの作成と管理][multiple-vm-nics]」をご覧ください。

### <a name="azure-storage"></a>Azure Storage

すべてデータベース サーバーの VM では、読み取り/書き込みの待機時間に一貫性を得るために Azure Premium Storage を使用することをお勧めします。 (A)SCS 仮想マシンなどの SAP アプリケーション サーバーでは、アプリケーション実行はメモリ内で行われ、ログ記録だけにディスクを使用するため、Azure Standard Storage を使用できます。

最高の信頼性を得るには、[Azure Managed Disks][managed-disks] の使用を推奨します。 管理対象ディスクでは可用性セット内の VM のディスクが必ず分離されるため、単一障害点を回避できます。

> [!NOTE]
> 現在、この参照用アーキテクチャの Resource Manager テンプレートでは管理対象ディスクは使用されていません。 管理対象ディスクを使用するようにテンプレートを更新する予定です。

高い IOPS とディスク帯域幅のスループットを達成するために、ストレージ ボリュームのパフォーマンス最適化の共通プラクティスが、Azure Storage レイアウトに適用されます。 たとえば、より大きなディスク ボリュームを作成するために複数のディスクをまとめてストライピングすると、IO パフォーマンスが向上します。 変更が頻繁に行われないストレージ コンテンツの読み取りキャッシュを有効にすると、データ取得の速度が向上します。 パフォーマンス要件の詳細については、「 [SAP note 1943937 - Hardware Configuration Check Tool (SAP Note 1943937 - ハードウェアの構成チェック ツール)][sap-1943937]」をご覧ください。

バックアップ データ ストアでは、Azure BLOB ストレージの[クール ストレージ層][ cool-blob-storage]を使用することをお勧めします。 クール ストレージ層は、アクセスの頻度が低く長期保管されるデータを保存するには、コスト効率が高い方法です。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

SAP アプリケーション レイヤーでは、Azure はスケールアップのための幅広い仮想マシン サイズを提供しています。 詳細な一覧については、「[SAP note 1928533 - SAP Applications on Azure: Supported Products and Azure VM types (SAP Note 1928533 - Azure 上の SAP アプリケーション: サポートされる製品と Azure VM の種類)」][sap-1928533]」をご覧ください。 スケールアウトするには、より多くの VM を可用性セットに追加します。

OLTP および OLAP SAP の両方のアプリケーションを搭載した Azure 仮想マシン上の SAP HANA では、SAP 認定を受けた仮想マシンのサイズが、単一の VM インスタンスを含む GS5 になっています。 より大きなワークロードに対応するために、Microsoft では、Microsoft Azure 認定データセンターに共同配置されている物理サーバー上に SAP HANA 向けの [Azure ラージ インスタンス][azure-large-instances]を提供しています。現時点では、単一のインスタンスに対して最大 4 TB のメモリ容量を提供しています。 複数ノードの構成では、最大 32 TB の合計メモリ容量を備えることも可能です。

## <a name="availability-considerations"></a>可用性に関する考慮事項

このように、一元化されたデータベース上に SAP アプリケーションを分散してインストールした場合、高可用性を実現するために基本のインストールがレプリケートされます。 アーキテクチャの各レイヤーでは、次のように高可用性の設計が異なります。

- **Web Dispatcher**。 高可用性は、SAP アプリケーション トラフィックによる冗長な SAP Dispatcher インスタンスによって達成されます。 SAP ドキュメントの「[SAP Web Dispatcher (SAP Web Dispatcher)][swd]」をご覧ください。

- **ASCS**。 Azure Windows 仮想マシン上では ASCS の高可用性を実現するために、Windows サーバー フェールオーバー クラスタリングが SIOS DataKeeper と共に使用され、クラスターの共有ボリュームを実装しています。 実装の詳細については、[Azure での SAP ASCS のクラスタリング][clustering]に関するページをご覧ください。

- **アプリケーション サーバー**。 高可用性は、アプリケーション サーバーのプール内の負荷分散トラフィックによって実現されます。

- **データベース層**。 この参照用アーキテクチャでは、単一の SAP HANA データベース インスタンスをデプロイしています。 高可用性を実現するためには、複数のインスタンスをデプロイし、HANA System Replication (HSR) を使用して手動フェールオーバーを実装します。 自動フェールオーバーを有効にするには、特定の Linux ディストリビューションの HA 拡張機能が必要です。

### <a name="disaster-recovery-considerations"></a>ディザスター リカバリーの考慮事項

各層では、さまざまな戦略を利用して、ディザスター リカバリー (DR) の保護を提供しています。

- **アプリケーション サーバー**。 SAP アプリケーション サーバーには、ビジネス データが含まれていません。 Azure での単純な DR 戦略は、別のリージョンで SAP アプリケーション サーバーを作成することです。 プライマリ アプリケーション サーバー上で任意の構成変更やカーネル更新を行った場合、同じ変更が DR リージョンの VM にコピーされる必要があります。 たとえば、カーネルの実行可能ファイルが DR VM にコピーされます。

- **SAP セントラル サービス**。 SAP アプリケーション スタックのこのコンポーネントも、ビジネス データを保持しません。 DR リージョンで VM を構築して、SCS ロールを実行できます。 同期するプライマリ SCS ノードからのコンテンツは、**/sapmnt** 共有コンテンツだけです。 また、構成の変更やカーネルの更新がプライマリ SCS サーバー上で発生した場合、これらの変更または更新が DR SCS でも繰り返される必要があります。 2 台のサーバーを同期するには、単に定期スケジュールされたコピー ジョブを使用して、**/sapmnt** を DR 側にコピーします。 作成、コピー、およびテスト フェールオーバー プロセスの詳細については、「[SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution (SAP NetWeaver: Hyper-V および Microsoft Azure ベースのディザスター リカバリー ソリューション)][sap-netweaver-dr]」をダウンロードして、「4.3. SAP SPOF layer (ASCS) (4.3 SAP SPOF レイヤー (ASCS))」を参照してください。

- **データベース層**。 HSR やストレージ レプリケーションなど、HANA でサポートされているレプリケーション ソリューションを使用します。 

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

SAP HANA は、基本となる Azure インフラストラクチャを使用したバックアップ機能を備えています。 Azure 仮想マシンで実行されている SAP HANA データベースをバックアップするために、SAP HANA スナップショットと Azure ストレージ スナップショットの両方が、バックアップ ファイルの一貫性を確保するために使用されます。 詳細については、「[Azure Virtual Machines 上の SAP HANA のバックアップ ガイド][ hana-backup]」および「[Azure Backup サービスについての質問][backup-faq]」をご覧ください。

Azure では、インフラストラクチャ全体の[監視と診断][ monitoring]のために、複数の機能を提供しています。 また、Azure 仮想マシン (Linux または Windows) の高度な監視は、Azure Operations Management Suite (OMS) で処理されます。

SAP インフラストラクチャのリソースとサービス パフォーマンスの SAP ベースでの監視を提供するには、Azure SAP Enhanced Monitoring 拡張機能を使用します。 この拡張機能では、Azure 監視統計情報をオペレーティング システムの監視と DBA Cockpit 機能のための SAP アプリケーションにフィードします。 

## <a name="security-considerations"></a>セキュリティに関する考慮事項

SAP は、SAP アプリケーション内でのロールベース アクセスと承認を制御するために、独自のユーザー管理エンジン (UME) を備えています。 詳細については、「[SAP HANA Security - An Overview (SAP HANA セキュリティ - 概要)][sap-security]」をご覧ください  (アクセスするには、AP Service Marketplace アカウントが必要です)。

インフラストラクチャ セキュリティでは、データは転送時および保存時に保護されます。 [Azure Virtual Machines (VM) 上の SAP NetWeaver の計画と実装][netweaver-on-azure]に関するガイドの「セキュリティに関する考慮事項」 セクションでは、冒頭にネットワーク セキュリティへの対応について記述されています。 また、このガイドでは、アプリケーション通信を許可するために、ファイアウォールで開く必要があるネットワーク ポートも指定しています。 

Windows および Linux IaaS 仮想マシン ディスクを暗号化するには、[Azure Disk Encryption][disk-encryption] を使用できます。 Azure Disk Encryption では、Windows の BitLocker 機能と Linux の DM-Crypt 機能を使用して、オペレーティング システムおよびデータ ディスクのボリュームの暗号化を提供しています。 ソリューションは Azure Key Vault と共に動作し、お使いの Key Vault サブスクリプションのディスク暗号化キーとシークレットを制御および管理できます。 仮想マシンのディスク上のデータは暗号化され、お使いの Azure ストレージに保存されます。

保存されている SAP HANA データの暗号化には、SAP HANA のネイティブ暗号化テクノロジを使用することをお勧めします。

> [!NOTE]
> 同じサーバー上の Azure ディスクの暗号化には、保存されている HANA データの暗号化を使用しないでください。

VNet にあるさまざまなサブネット間のトラフィックを制限するために、[ネットワーク セキュリティ グループ][nsg] (NSG) を使用することを検討してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法 

この参照用アーキテクチャのデプロイ スクリプトについては、[GitHub][github] をご覧ください。


### <a name="prerequisites"></a>前提条件

- インストールを完了するには、SAP ソフトウェア ダウンロード センターへのアクセスが必要です。
 
- 最新バージョンの [Azure PowerShell][azure-ps] をインストールします。 

- このデプロイには、51 コアが必要です。 デプロイする前に、お使いのサブスクリプションに VM コアのクォータが十分にあることを確認します。 十分にない場合は、Azure ポータルを使用して、クォータを増やすためのサポート要求を発行します。
 
- このデプロイでは、GS シリーズの VM を使用します。 リージョンごとの GS シリーズの可用性については、[このリンク先][region-availability]を確認してください。

- このデプロイのコストを見積もるには、「[料金計算ツール][azure-pricing]」をご覧ください。 
 
この参照用アーキテクチャでは、次の VM をデプロイしています。

| リソース名 | VM サイズ | 目的  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP セントラル サービス |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | SAP NetWeaver アプリケーション |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | SAP HANA データベース インスタンス |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

単一の SAP HANA インスタンスがデプロイされます。 アプリケーション VM の場合、デプロイするインスタンス数はテンプレート パラメーターに指定します。

### <a name="deploy-sap-infrastructure"></a>SAP インフラストラクチャのデプロイ

このアーキテクチャは段階的にデプロイすることも一度にデプロイすることもできます。 最初は、各デプロイの手順を確認できるように、段階的なデプロイをお勧めします。 次のいずれかの "*モード*" パラメーターを使用して段階を指定します。

| Mode           | 実行内容                                                                                                            |
|----------------|-----------------------------------------------------|
| インフラストラクチャ | Azure にネットワーク インフラストラクチャをデプロイします。        |
| ワークロード       | SAP サーバーをネットワークにデプロイします。             |
| すべて            | 上記のすべてをデプロイします。              |

ソリューションをデプロイするには、次の手順を実行します。

1. [GitHub リポジトリ][github]をローカル コンピューターにダウンロードするか複製します。

2. PowerShell ウィンドウを開き、`/sap/sap-hana/` フォルダーに移動します。

3. 次の PowerShell コマンドレットを実行します。 `<subscription id>` には、Azure サブスクリプション ID を使用します。 `<location>` には、Azure リージョン (`eastus` や `westus` など) を指定します。 `<mode>` には、上記のモードのいずれかを指定します。

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  プロンプトが表示されたら、Azure アカウントにログオンします。 

選択したモードによって異なりますが、デプロイ スクリプトは完了するまでに数時間かかることがあります。

> [!WARNING]
> パラメーター ファイルには、ハードコーディングされたパスワード (`AweS0me@PW`) がさまざまな場所に含まれています。 デプロイする前にこれらの値を変更します。
 
### <a name="configure-sap-applications-and-database"></a>SAP アプリケーションおよびデータベースの構成

SAP インフラストラクチャのデプロイ後、SAP アプリケーションおよび HANA データベースを仮想マシン上に次のようにインストールして構成します。

> [!NOTE]
> SAP のインストール手順を確認するには、「[SAP installation guides (SAP インストール ガイド)][sap-guide]」をダウンロードするために、SAP サポート ポータルのユーザー名とパスワードが必要です。

1. Jumpbox (`ra-sap-jumpbox-vm1`) へログインします。 他の VM にログインするために、Jumpbox を使用します。 

2.  `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` という名前の各 VM に対して、VM にログインして、[Web Dispatcher インストール][sap-dispatcher-install]の wiki に記載されている手順を使用して、SAP Web Dispatcher インスタンスをインストールして構成します。

3.  `ra-sap-data-vm1` という名前の VM にログインします。 「[SAP HANA Server Installation and Update Guide (SAP HANA サーバーのインストールおよび更新ガイド)][hana-guide]」を使用して、SAP Hana データベース インスタンスをインストールして構成します。

4. `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` という名前の各 VM に対して、VM にログインして、「[SAP installation guides (SAP インストール ガイド)][sap-guide]」を使用して SAP セントラル サービス (SCS) をインストールして構成します。

5.  `ra-sapApps-vm1` ... `ra-sapApps-vmN` という名前の各 VM に対して、VM にログインして、「[SAP installation guides (SAP インストール ガイド)][sap-guide]」を使用して SAP NetWeaver アプリケーションをインストールして構成します。

**_この参照用アーキテクチャの作成者_** &mdash; Rick Rainey、Ross Sponholtz、Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.blob.core.windows.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "Microsoft Azure を使用した SAP HANA アーキテクチャ"
