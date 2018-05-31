---
title: Azure における Linux Virtual Machines の SAP S/4HANA
description: 高可用性を備えた Azure の Linux環境で SAP S/4HANA を実行するための実証済みプラクティス。
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: d24ef6f9e4eae460d0d0dcfff35568c812d09951
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/21/2018
ms.locfileid: "34423096"
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>Azure における Linux Virtual Machines の SAP S/4HANA

この参照用アーキテクチャは、Azure でディザスター リカバリーをサポートする高可用性環境で S/4HANA を実行するための一連の実証済みプラクティスを示しています。 このアーキテクチャは特定の仮想マシン (VM) サイズでデプロイされ、お客様の組織のニーズに合わせて変更できます。 


![](./images/sap-s4hana.png)

## <a name="architecture"></a>アーキテクチャ
 
> [!NOTE] 
> この参照用アーキテクチャに従って SAP 製品をデプロイするには、SAP 製品と他の Microsoft 以外のテクノロジの適切なライセンスが必要です。

この参照アーキテクチャでは、エンタープライズ レベル、運用レベルのシステムについて説明します。 この構成は、お客様のビジネス ニーズに合わせて単一の仮想マシンに縮小できます。 ただし、以下のコンポーネントが必要です。

**Virtual network**。 [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) サービスによって、Azure リソースが安全に相互接続されます。 このアーキテクチャでは、仮想ネットワークは、[ハブスポーク トポロジ](../hybrid-networking/hub-spoke.md)のハブにデプロイされたゲートウェイ経由でオンプレミス環境に接続されます。 スポークは、SAP アプリケーションに使用される仮想ネットワークです。

**サブネット**。 仮想ネットワークは、階層 (ゲートウェイ層、アプリケーション層、データベース層、共有サービス層) ごとに個別の[サブネット](/azure/virtual-network/virtual-network-manage-subnet)に分割されます。 

**仮想マシン**。 このアーキテクチャでは、アプリケーション層とデータベース層に Linux が実行されている仮想マシンが使用され、次のようにグループ化されます。

- **アプリケーション層**。 Fiori Front-end Server プール、SAP Web Dispatcher プール、アプリケーション サーバー プール、および SAP セントラル サービス クラスターが含まれます。 Azure Linux 仮想マシンでセントラル サービスの高可用性を実現するには、高可用性ネットワーク ファイル システム (NFS) サービスが必要です。
- **NFS クラスター**。 このアーキテクチャでは、Linux クラスターで実行されている [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) サーバーを使って、SAP システム間で共有されるデータが格納されます。 この一元化されたクラスターは、複数の SAP システムで共有できます。 NFS サービスの高可用性は、選択されている Linux ディストリビューションに適した High Availability Extension によって実現します。
- **SAP HANA**。 データベース層では、高可用性を実現するために、クラスターで複数の Linux 仮想マシンが使用されます。 HANA システム レプリケーション (HSR) を使用して、プライマリ HANA システムとセカンダリHANA システムの間でコンテンツがレプリケートされます。 システム障害の検出と自動フェールオーバーの促進には Linux クラスタリングが使用されます。 また、ストレージ ベースまたはクラウド ベースのフェンス メカニズムが採用され、障害が発生したシステムを確実に分離またはシャットダウンし、クラスターのスプリット ブレイン状態を回避できます。
- **Jumpbox**。 要塞ホストとも呼ばれます。 これは、他の仮想マシンに接続するために管理者が使用するネットワークの安全な仮想マシンです。 Windows または Linux を実行できます。 HANA Cockpit または HANA Studio 管理ツールを使用するときは、Web を閲覧しやすいように Windows Jumpbox を使用します。

**ロード バランサー**。 組み込みの SAP ロード バランサーと [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) の両方が、HA を実現するために使用されます。 Azure Load Balancer インスタンスは、アプリケーション層のサブネットで仮想マシンにトラフィックを分散させるときに使用されます。

**可用性セット**。 すべてのプールおよびクラスター (Web Dispatcher、SAP アプリケーション サーバー、セントラル サービス、NFS、および HANA) の仮想マシンが個別の[可用性セット](/azure/virtual-machines/windows/tutorial-availability-sets)にグループ化され、ロールあたり少なくとも 2 つの仮想マシンがプロビジョニングされます。 これにより、仮想マシンが、より高度な[サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA) に対応できるようになります。 

**NIC**。 [ネットワーク インターフェイス カード](/azure/virtual-network/virtual-network-network-interface) (NIC) により、仮想ネットワーク上の仮想マシンのすべての通信が有効になります。

**ネットワーク セキュリティ グループ**。 仮想ネットワークで受信トラフィック、送信トラフィック、およびサブネット間トラフィックを制限するために、[ネットワーク セキュリティ グループ](/azure/virtual-network/virtual-networks-nsg) (NSG) が使用されます。

**ゲートウェイ**。 ゲートウェイにより、オンプレミス ネットワークが Azure 仮想ネットワークに拡張されます。 [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) は、パブリック インターネットを経由しないプライベート接続を作成するための推奨 Azure サービスですが、[サイト間](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)接続を使用することもできます。 

**Azure Storage**。 仮想マシンの仮想ハード ディスク (VHD) の永続的ストレージを提供するには、[Azure Storage](/azure/storage/) が必要です。 

## <a name="recommendations"></a>Recommendations

このアーキテクチャでは、小規模な運用レベルのエンタープライズ デプロイについて説明します。 対象となるデプロイは、ビジネス要件によって異なります。 これらの推奨事項は原案として使用してください。

### <a name="virtual-machines"></a>仮想マシン

アプリケーション サーバーのプールおよびクラスターで、それぞれの要件に基づいて仮想マシンの数を調整します。 [Azure Virtual Machines の計画と実装ガイド](/azure/virtual-machines/workloads/sap/planning-guide)に関するページには、仮想マシンでの SAP NetWeaver の実行について詳しく説明されていますが、この説明は SAP S/4HANA にも適用されます。

Azure 仮想マシンの種類とスループットのメトリック (SAPS) の SAP サポートの詳細については、[SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533) を参照してください。 

### <a name="sap-web-dispatcher-pool"></a>SAP Web Dispatcher プール

Web Dispatcher コンポーネントは、SAP アプリケーション サーバー間の SAP トラフィックのロード バランサーとして使用されます。 Web Dispatcher コンポーネントの高可用性は、バランサーのバックエンド プールで使用可能な Web Dispatcher 間での HTTP (S) トラフィック分散のために、Azure Load Balancer を使って、ラウンドロビン構成で並列 Web Dispatcher セットアップを実装することで実現します。 

### <a name="fiori-front-end-server"></a>Fiori Front-end Server

Fiori Front-end Server では [NetWeaver Gateway](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true) が使用されます。 小規模なデプロイについては、Fiori サーバーに読み込むことができます。 大規模なデプロイでは、NetWeaver Gateway 用の別個のサーバーを、Fiori Front-end Server プールの外側にデプロイできます。

### <a name="application-servers-pool"></a>アプリケーション サーバー プール

ABAP アプリケーション サーバーのログオン グループの管理には、SMLG トランザクションが使用されます。 この場合、セントラル サービスのメッセージ サーバー内の負荷分散機能を使って、SAPGUI および RFC トラフィックの SAP アプリケーション サーバーのプールにワークロードが分散されます。 アプリケーション サーバーは、クラスター仮想ネットワーク名を介して高可用性セントラル サービスに接続されます。 これにより、ローカル フェールオーバー後に、セントラル サービス接続用のアプリケーション サーバー プロファイルを変更する必要がなくなります。 

### <a name="sap-central-services-cluster"></a>SAP セントラル サービス クラスター

高可用性が不要な場合は、セントラル サービスを単一の仮想マシンにデプロイできます。 ただし、単一の仮想マシンは、SAP 環境の単一障害点 (SPOF) になる可能性があります。 高可用性セントラル サービス デプロイについては、高可用性 NFS クラスターと高可用性セントラル サービス クラスターが使用されます。

### <a name="nfs-cluster"></a>NFS クラスター

NFS クラスターのノード間のレプリケーションには DRBD (Distributed Replicated Block Device) が使用されます。

### <a name="availability-sets"></a>可用性セット

可用性セットにより、サーバーがさまざまな物理インフラストラクチャおよび更新グループに分散され、サービスの可用性が向上します。 同じロールを実行する仮想マシンを可用性セットに配置すると、Azure インフラストラクチャのメンテナンスに伴うダウンタイムを防ぎ、[サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/virtual-machines) に準拠するのに役立ちます。 可用性セットごとに複数の仮想マシンを配置することをお勧めします。

セット内の仮想マシンはすべて、同じロールを実行します。 同じ可用性セットに異なるロールのサーバーを混在させないでください。 たとえば、アプリケーション サーバーを含む可用性セットには、ASCS ノードを配置しないでください。

### <a name="nics"></a>NIC

従来のオンプレミスの SAP 環境では、管理トラフィックとビジネス トラフィックを切り離すために、マシンごとに複数のネットワーク インターフェイス カード (NIC) が実装されます。 Azure では、仮想ネットワークは、すべてのトラフィックを同じネットワーク ファブリック経由で送信するソフトウェア定義ネットワークです。 したがって、複数の NIC を使用する必要はありません。 ただし、お客様の組織がトラフィックを分離する必要がある場合は、VM ごとに複数の NIC をデプロイし、各 NIC をそれぞれ異なるサブネットに接続することで、NSG を使ってさまざまなアクセス制御ポリシーを強制できます。

### <a name="subnets-and-nsgs"></a>サブネットと NSG

このアーキテクチャでは、仮想ネットワーク アドレス空間がサブネットに分割されます。 各サブネットを、サブネットのアクセス ポリシーを定義する NSG に関連付けることができます。 アプリケーション サーバーは、切り離されたサブネットに配置してください。これにより管理対象が個別のサーバーではなく、サブネット セキュリティ ポリシーになるため、サーバーのセキュリティが確保しやすくなります。

NSG がサブネットに関連付けられている場合、その NSG はサブネット内のすべてのサーバーに適用されます。 NSG を使用してサブネット内のサーバーをきめ細かく制御する方法の詳細については、[ネットワーク セキュリティ グループによるネットワーク トラフィックのフィルター処理](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/)に関するページをご覧ください。

また、「[VPN ゲートウェイの計画と設計](/azure/vpn-gateway/vpn-gateway-plan-design)」も参照してください。

### <a name="load-balancers"></a>ロード バランサー

[SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) では、SAP アプリケーション サーバー プールへの HTTP(S) トラフィック (Fiori スタイルのアプリケーションを含む) の負荷分散が処理されます。 

DIAG またはリモート ファンクション コール (RFC) を使用して SAP サーバーに接続している SAP GUI クライアントからのトラフィックについては、セントラル サービスのメッセージ サーバーでは、SAP アプリケーション サーバーの[ログオン グループ](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing)を使用して負荷が分散されます。したがって、追加のロード バランサーは必要ありません。 

### <a name="azure-storage"></a>Azure Storage

データベース サーバーの仮想マシンに対して Azure Premium Storage を使用することをお勧めします。 Premium Storage により、一貫した読み取り/書き込みの待機時間を実現できます。 単一インスタンス仮想マシンのオペレーティング システム ディスクおよびデータ ディスクに対する Premium Storage の使用に関する詳細については、「[Virtual Machines の SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/)」を参照してください。 

すべての運用 SAP システムで、Premium [Azure Managed Disks](/azure/storage/storage-managed-disks-overview) を使用することをお勧めします。 ディスクの VHD ファイルの管理には、信頼性を高めるために、Managed Disks が使用されます。 また、これにより可用性セット内の仮想マシンのディスクが必ず分離されるため、単一障害点を回避できます。

セントラル サービス仮想マシンなどの SAP アプリケーション サーバーでは、アプリケーション実行はメモリ内で行われ、ログ記録だけにディスクが使用されるため、Azure Standard Storage を使用してコストを削減できます。 ただし、現時点では、Standard Storage はアンマネージド ストレージに対してのみ認定されています。 アプリケーション サーバーではデータがホストされないため、小さいサイズの P4 および P6 Premium Storage ディスクが、コストを最小限に抑えるうえで役に立つこともあります。

バックアップ データ ストアでは、Azure の[クール アクセス層ストレージやアーカイブ アクセス層ストレージ](/azure/storage/storage-blob-storage-tiers)を使用することをお勧めします。 これらのストレージ層により、コスト効果の高い方法で、有効期間が長くアクセスの少ないデータを格納できます。

## <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

SAP アプリケーション サーバーは、データベース サーバーと常に通信しています。 HANA データベースの仮想マシンについては、[書き込みアクセラレータ](/azure/virtual-machines/linux/how-to-enable-write-accelerator)を有効にして、ログ書き込みの待機時間を短縮することを検討してください。 サーバー間の通信を最適化するには、[高速ネットワーク](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/)を使用します。 これらのアクセラレータは、特定の VM シリーズでしか使用できないことに注意してください。

高い IOPS とディスク帯域幅のスループットを達成するために、ストレージ ボリュームの[パフォーマンス最適化](/azure/virtual-machines/linux/premium-storage-performance)の共通プラクティスが、Azure Storage レイアウトに適用されます。 たとえば、ストライピングされたディスク ボリュームを作成するために複数のディスクを結合すると、IO パフォーマンスが向上します。 変更が頻繁に行われないストレージ コンテンツの読み取りキャッシュを有効にすると、データ取得の速度が向上します。 パフォーマンス要件の詳細については、「[SAP Note 1943937 - Hardware Configuration Check Tool (SAP Note 1943937 - ハードウェア構成チェックツール)](https://launchpad.support.sap.com/#/notes/1943937)」を参照してください (アクセスするには、SAP Service Marketplace アカウントが必要です)。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

SAP アプリケーション レイヤーでは、Azure は、スケールアップおよびスケールアウトのための幅広い仮想マシン サイズを提供しています。詳細な一覧については、[SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533) の「SAP Applications on Azure: Supported Products and Azure VM types (Azure 上の SAP アプリケーション: サポートされる製品と Azure VM の種類)」を参照してください (アクセスするには、SAP Service Marketplace アカウントが必要です)。 認定済み仮想マシンの種類が引き続き増えていくことで、ユーザーは同じクラウド デプロイでスケールアップまたはスケールダウンできます。 

データベース層では、このアーキテクチャによって VM で HANA が実行されます。 ご自身のワークロードが最大 VM サイズを超えた場合は、Microsoft では、[Azure Large Instances](/azure/virtual-machines/workloads/sap/hana-overview-architecture) for SAP HANA も提供しています。 これらの物理サーバーは Microsoft Azure 認定データセンターに併置され、このドキュメントの作成時点では、単一インスタンスに対して最大 20 TB のメモリ容量を提供します。 複数ノードの構成では、最大 60 TB の合計メモリ容量を備えることも可能です。

## <a name="availability-considerations"></a>可用性に関する考慮事項

リソースの冗長性は、高可用性インフラストラクチャ ソリューションの一般的なテーマです。 SLA がそれほど厳しくない企業については、単一インスタンスの Azure VM によってアップタイム SLA が提供されます。 詳細については、[Azure サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/)に関するページをご覧ください。

このように SAP アプリケーションを分散してインストールした場合、高可用性を実現するために基本のインストールがレプリケートされます。 高可用性の設計は、アーキテクチャのレイヤーごとに異なります。 

### <a name="application-tier"></a>アプリケーション層

- Web Dispatcher。 高可用性は冗長 Web Dispatcher インスタンスによって実現します。 SAP ドキュメントの「[SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm)」を参照してください。
- Fiori サーバー。 高可用性は、サーバーのプール内の負荷分散トラフィックによって実現します。
- セントラル サービス。 Azure Linux 仮想マシンのセントラル サービスの高可用性は、選択されている Linux ディストリビューションに適した High Availability Extension によって実現します。また、高可用性 NFS クラスターによって DRBD ストレージがホストされます。
- アプリケーション サーバー。 高可用性は、アプリケーション サーバーのプール内の負荷分散トラフィックによって実現されます。

### <a name="database-tier"></a>データベース層

この参照アーキテクチャは、2 つの Azure 仮想マシンで構成される高可用性 SAP HANA データベース システムを示しています。 データベース層のネイティブのシステム レプリケーション機能では、レプリケートされたノード間でのフェールオーバーを手動または自動で行うことができます。

- 手動フェールオーバーを行うには、複数の HANA インスタンスをデプロイし、HANA システム レプリケーション (HSR) を使用します。
- 自動フェールオーバーを行うには、お使いの Linux ディストリビューションに適した HSR と Linux High Availability Extension (HAE) の両方を使用します。 Linux HAE はクラスター サービスを HANA リソースに提供することで、障害イベントを検出し、正常なノードへの誤ったサービスのフェールオーバーを調整します。 

「[Microsoft Azure で実行されている SAP の認定と構成](/azure/virtual-machines/workloads/sap/sap-certifications)」を参照してください。

### <a name="disaster-recovery-considerations"></a>ディザスター リカバリーの考慮事項
各層では、さまざまな戦略を利用して、ディザスター リカバリー (DR) の保護を提供しています。

- **アプリケーション サーバー**。 SAP アプリケーション サーバーには、ビジネス データが含まれていません。 Azure での単純な DR 戦略は、セカンダリ リージョンで SAP アプリケーション サーバーを作成し、そのサーバーをシャットダウンすることです。 プライマリ アプリケーション サーバーで任意の構成変更やカーネル更新を行った場合、同じ変更がセカンダリ リージョンの仮想マシンに適用されなければなりません。 たとえば、SAP カーネルの実行可能ファイルを DR 仮想マシンにコピーします。 アプリケーション サーバーをセカンダリ リージョンに自動的にレプリケートするためのソリューションとしては、[Azure Site Recovery](/azure/site-recovery/site-recovery-overview) をお勧めします。 このドキュメントの作成時点では、ASR では、Azure VM における高速ネットワーク構成設定のレプリケーションがまだサポートされていません。

- **セントラル サービス**。 SAP アプリケーション スタックのこのコンポーネントにも、ビジネス データが保持されません。 セカンダリ リージョンで VM を作成すると、セントラル サービス ロールを実行できます。 プライマリ セントラル サービス ノードから同期されるコンテンツは、/sapmnt 共有コンテンツだけです。 また、構成の変更やカーネルの更新がプライマリ セントラル サービス サーバーで発生した場合、その変更や更新は、セントラル サービスが実行されているセカンダリ リージョンの VM でもう一度行われなければなりません。 2 つのサーバーを同期するには、Azure Site Recovery を使用してクラスター ノードをレプリケートするか、単純に、定期的にコピーするようにスケジュール設定されたコピー ジョブを使用して、/sapmnt を DR 側にコピーします。 作成、コピー、およびテスト フェールオーバー プロセスの詳細については、「[SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution (SAP NetWeaver: Hyper-V および Microsoft Azure ベースのディザスター リカバリー ソリューション)](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx)」をダウンロードして、「4.3. SAP SPOF layer (ASCS) (4.3 SAP SPOF レイヤー (ASCS))」を参照してください。 このドキュメントは、Windows で実行されている NetWeaver に適用されますが、Linux 用の同等の構成を作成することもできます。 セントラル サービスについては、[Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview) を使用して、クラスター ノードとストレージをレプリケートします。 Linux については、High Availability Extension を使用して、3 つのノード geo クラスターを作成します。 

- **SAP データベース層**。 HANA でサポートされているレプリケーションに対して HSR を使用します。 2 ノードのローカル高可用性セットアップのほかに、HSR では多層レプリケーションがサポートされます。このレプリケーションで、切り離された Azure リージョンの 3 つ目のノードは、クラスターの一部としてではなく、外部エンティティとして動作し、そのレプリケーション ターゲットとして、クラスター化された HSR ペアのセカンダリ レプリカに登録します。 これにより、レプリケーション デイジー チェーンが形成されます。 DR ノードへのフェールオーバーは、手動プロセスです。

Azure Site Recovery を使用して、ご自身の元のサイトが完全にレプリケートされた運用サイトを自動的に構築するには、カスタマイズされた[デプロイ スクリプト](/azure/site-recovery/site-recovery-runbook-automation)を実行する必要があります。 Site Recovery によって、最初に仮想マシンが可用性セットにデプロイされ、その後、ロード バランサーなどのリソースを追加するスクリプトが実行されます。 

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

SAP HANA は、基本となる Azure インフラストラクチャを使用したバックアップ機能を備えています。 Azure 仮想マシンで実行されている SAP HANA データベースをバックアップするために、SAP HANA スナップショットと Azure ストレージ スナップショットの両方が、バックアップ ファイルの一貫性を確保するために使用されます。 詳細については、「[Azure Virtual Machines 上の SAP HANA のバックアップ ガイド](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)」および「[Azure Backup サービスについての質問](/azure/backup/backup-azure-backup-faq)」を参照してください。 Azure ストレージ スナップショットをサポートするのは、HANA の単一コンテナー デプロイだけです。

### <a name="identity-management"></a>ID 管理

すべてのレベルで一元化された ID 管理システムを使用して、リソースへのアクセスを制御します。

- [ロールベースのアクセス制御](/azure/active-directory/role-based-access-control-what-is) (RBAC) を使用して、Azure リソースへのアクセスを提供します。 
- LDAP、Azure Active Directory、Kerberos、または他のシステム介して Azure VM へのアクセスを許可します。 
- SAP が提供するサービスを介してアプリ自体でアクセスをサポートするか、[OAuth 2.0 と Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code) を使用します。 

### <a name="monitoring"></a>監視

Azure には、インフラストラクチャ全体の[監視と診断](/azure/architecture/best-practices/monitoring)を行うための機能が複数用意されています。 また、Azure 仮想マシン (Linux または Windows) の高度な監視は、Azure Operations Management Suite (OMS) で処理されます。 

SAP インフラストラクチャのリソースとサービス パフォーマンスの SAP ベースの監視には、[Azure SAP Enhanced Monitoring](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca) 拡張機能が使用されます。 この拡張機能では、Azure 監視統計情報をオペレーティング システムの監視と DBA Cockpit 機能のための SAP アプリケーションにフィードします。 SAP の拡張された監視機能は、Azure で SAP を実行するうえで必須の前提条件です。 詳細については、[SAP Note 2191498](https://launchpad.support.sap.com/#/notes/2191498) の「SAP on Linux with Azure: Enhanced Monitoring (Azure を使用した Linux 上の SAP: 拡張された監視機能)」を参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

SAP は、SAP アプリケーション内でのロールベース アクセスと承認を制御するために、独自のユーザー管理エンジン (UME) を備えています。 詳細については、「[SAP HANA Security - An Overview (SAP HANA のセキュリティ - 概要)](https://archive.sap.com/documents/docs/DOC-62943)」を参照してください (アクセスするには、SAP Service Marketplace アカウントが必要です)。

追加のネットワーク セキュリティについては、[ネットワーク DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid) を実装することを検討します。これにより、ネットワーク仮想アプライアンスを使用して、Web Dispatcher および Fiori Front-End Server プールのサブネットの外側にファイアウォールが作成されます。

インフラストラクチャ セキュリティでは、データは転送時および保存時に暗号化されます。 [Azure Virtual Machines 上の SAP NetWeaver の計画と実装に関するガイド](/azure/virtual-machines/workloads/sap/planning-guide)の「セキュリティに関する考慮事項」セクションでは、冒頭にネットワーク セキュリティへの対応について記述されています。このセクションは S/4HANA に適用されます。 また、このガイドでは、アプリケーション通信を許可するために、ファイアウォールで開く必要があるネットワーク ポートも指定しています。 

Linux IaaS 仮想マシン ディスクを暗号化するには、[Azure Disk Encryption](/azure/security/azure-security-disk-encryption) を使用できます。 この場合、Linux の DM-Crypt 機能によって、オペレーティング システムおよびデータ ディスクのボリュームが暗号化されます。 また、このソリューションは Azure Key Vault と共に動作するため、お使いのキー コンテナー サブスクリプションのディスク暗号化キーとシークレットの制御および管理にも役立ちます。 仮想マシンのディスク上のデータは暗号化され、お使いの Azure ストレージに保存されます。

保存されている SAP HANA データの暗号化には、SAP HANA のネイティブ暗号化テクノロジを使用することをお勧めします。 

> [!NOTE]
> 同じサーバー上の Azure Disk Encryption には、保存されている HANA データの暗号化を使用しないでください。 HANA については、HANA データの暗号化のみを使用します。

## <a name="communities"></a>コミュニティ

コミュニティは質問に答え、デプロイを正常に完了できるよう支援します。 以下、具体例に沿って説明します。

- [Microsoft プラットフォームでの SAP アプリケーションの実行 (ブログ)](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Azure コミュニティ サポート](https://azure.microsoft.com/support/community/)
- [SAP Community](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)
