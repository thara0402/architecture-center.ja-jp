---
title: "高可用性 SharePoint Server 2016 ファームの Azure での実行"
description: "高可用性 SharePoint Server 2016 ファームを Azure で設定するための実証済みプラクティス。"
author: njray
ms.date: 08/01/2017
ms.openlocfilehash: 0c0e9a7b2ae12a2d12919548f91304e6cbd2d8a6
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2017
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a>高可用性 SharePoint Server 2016 ファームの Azure での実行

この参照用アーキテクチャは、高可用性 SharePoint Server 2016 ファームを Azure で設定し、MinRole トポロジおよび SQL Server Always On 可用性グループを使用するための一連の実証済みプラクティスを示します。 SharePoint ファームは、インターネットに接続するエンドポイントまたはプレゼンスがない、セキュアな仮想ネットワークにデプロイされます。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、「[Run Windows VMs for an N-tier application][windows-n-tier]」(n 層アプリケーションでの Windows VM の実行) で説明したアーキテクチャ上に構築されています。 Azure 仮想ネットワーク (VNet) 内に高可用性を備えた SharePoint Server 2016 ファームをデプロイします。 このアーキテクチャは、テスト環境または実稼働環境、Office 365 を含む SharePoint ハイブリッド インフラストラクチャに適しており、ディザスター リカバリー シナリオの基礎としても適しています。

アーキテクチャは、次のコンポーネントで構成されています。

- **リソース グループ**。 [リソース グループ][resource-group]は、Azure の関連リソースを保持するコンテナーです。 1 つのリソース グループが SharePoint サーバーで使用され、もう 1 つのリソース グループが VM とは独立しているインフラストラクチャ コンポーネント (仮想ネットワークやロード バランサーなど) で使用されます。

- **仮想ネットワーク (VNet)**。 VM は一意のイントラネット アドレス空間を使用して VNet にデプロイされます。 VNet はさらにサブネットに分割されます。 

- **仮想マシン (VM)**。 VM は VNet にデプロイされ、プライベート静的 IP アドレスがすべての VM に割り当てられます。 静的 IP アドレスは、IP アドレスのキャッシュや再起動後のアドレスの変更に伴う問題を回避するため、SQL Server と SharePoint Server 2016 を実行する VM に推奨されます。

- **可用性セット**。 各 SharePoint ロールの VM を別の[可用性セット][availability-set]に配置し、ロールごとに少なくとも 2 つの仮想マシン (VM) をプロビジョニングします。 こうすると VM が高度なサービス レベル アグリーメント (SLA) に対応できるようになります。 

- **内部ロード バランサー**:  [ロード バランサー][load-balancer]は、SharePoint の要求トラフィックをオンプレミス ネットワークから SharePoint ファームのフロントエンド Web サーバーに均等配置します。 

- **ネットワーク セキュリティ グループ (NSG)**。 仮想マシンを含むサブネットごとに、[ネットワーク セキュリティ グループ][nsg]が作成されます。 NSG を使用して、サブネットを分離するために VNet 内のネットワーク トラフィックを制限します。 

- **ゲートウェイ**。 ゲートウェイによって、オンプレミス ネットワークと Azure 仮想ネットワークの間に接続が提供されます。 この接続は ExpressRoute またはサイト間 VPN を使用できます。 詳しくは、「[オンプレミス ネットワークの Azure への接続][hybrid-ra]」をご覧ください。

- **Windows Server Active Directory (AD) ドメイン コントローラー**。 SharePoint Server 2016 では Azure Active Directory Domain Services の使用はサポートされないため、Windows Server AD ドメイン コントローラーをデプロイする必要があります。 これらのドメイン コントローラーは Azure VNet で実行し、オンプレミスの Windows Server AD フォレストと信頼関係を保ちます。 SharePoint ファーム リソースに対するクライアント Web 要求は、ゲートウェイ接続経由でオンプレミス ネットワークに認証のトラフィックを送信する代わりに、VNet 内で認証されます。 DNS では、イントラネット A または CNAME レコードが作成されるため、イントラネット ユーザーは SharePoint ファームの名前を内部ロード バランサーのプライベート IP アドレスに解決できます。

- **SQL Server Always On 可用性グループ**。 SQL Server データベースの高可用性を実現するには、[SQL Server Always On 可用性グループ][sql-always-on]を推奨します。 2 つの仮想マシンが SQL Server で使用されます。 1 つにはプライマリ データベース レプリカが含まれ、もう 1 つにはセカンダリ レプリカが含まれます。 

- **マジョリティ ノード VM**。 この VM を使用すると、フェールオーバー クラスターがクォーラムを確立できます。 詳しくは、「[フェールオーバー クラスターのクォーラム構成について][sql-quorum]」をご覧ください。

- **SharePoint サーバー**。 SharePoint サーバーは、Web フロントエンド、キャッシュ、アプリケーション、およびロールの検索を実行します。 

- **Jumpbox**。 [要塞ホスト][bastion-host]とも呼ばれます。 これは、管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。 jumpbox の NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。 NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項を開始点として使用してください。

### <a name="resource-group-recommendations"></a>リソース グループの推奨事項

サーバー ロールごとにリソース グループを分け、グローバル リソースであるインフラストラクチャ コンポーネントにも別のリソース グループを用意することを推奨します。 このアーキテクチャでは、SharePoint リソースが 1 つのグループを形成し、SQL Server と他のユーティリティ アセットがもう 1 つのグループを形成します。

### <a name="virtual-network-and-subnet-recommendations"></a>仮想ネットワークとサブネットの推奨事項

SharePoint ロールごとに 1 つのサブネット、ゲートウェイに 1 つのサブネット、および jumpbox に 1 つのサブネットを使用します。 

ゲートウェイ サブネット名は、*GatewaySubnet* にする必要があります。 ゲートウェイ サブネットのアドレス空間には、仮想ネットワーク アドレス空間の最後の部分を割り当てます。 詳しくは、「[Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra]」(VPN ゲートウェイを使用したオンプレミス ネットワークの Azure への接続) をご覧ください。

### <a name="vm-recommendations"></a>VM の推奨事項

Standard DSv2 仮想マシン サイズに基づくと、このアーキテクチャでは最小で 38 コアが必要です。

- Standard_DS3_v2 上に 8 つの SharePoint サーバー (それぞれ 4 コア) = 32 コア
- Standard_DS1_v2 上に 2 つの Active Directory ドメイン コントローラー (それぞれ 1 コア) = 2 コア
- Standard_DS1_v2 上に 2 つの SQL Server VM = 2 コア
- Standard_DS1_v2 上に 1 つのマジョリティ ノード = 1 コア
- Standard_DS1_v2 上に 1 つの管理サーバー = 1 コア

コアの合計数は、選択する VM のサイズによって異なります。 詳しくは、以下の「[SharePoint Server の推奨事項](#sharepoint-server-recommendations)」をご覧ください。

デプロイについて Azure サブスクリプションに十分な VM コア クォータがあることを確認してください。不足している場合はデプロイで障害が発生します。 「[Azure サブスクリプションとサービスの制限、クォータ、制約][quotas]」をご覧ください。 
 
### <a name="nsg-recommendations"></a>NSG の推奨事項

VM を含むサブネットごとに 1 つの NSG を用意して、サブネットを分離できるようにすることを推奨します。 サブネットの分離を構成する場合は、各サブネットについて許可または拒否するインバウンド トラフィックまたはアウトバウンド トラフィックを定義する NSG ルールを追加します。 詳しくは、「[ネットワーク セキュリティ グループによるネットワーク トラフィックのフィルタリング][virtual-networks-nsg]」をご覧ください。 

ゲートウェイには NSG を割り当てないでください。割り当てると、ゲートウェイが機能を停止します。 

### <a name="storage-recommendations"></a>記憶域の推奨事項

ファームの VM の記憶域構成は、オンプレミス デプロイで使用される適切なベスト プラクティスと一致する必要があります。 SharePoint サーバーではログ用に別のディスクが必要です。 検索インデックス ロールをホストする SharePoint サーバーでは、検索インデックスを格納するために追加のディスク領域が必要です。 SQL Server の場合、標準プラクティスではデータとログを分離します。 データベース バックアップ ストレージにディスクをさらに追加し、[tempdb][tempdb] 用に別のディスクを使用します。

最高の信頼性を得るには、[Azure Managed Disks][managed-disks] の使用を推奨します。 管理対象ディスクでは可用性セット内の VM のディスクが必ず分離されるため、単一障害点を回避できます。 

> [!NOTE]
> 現在、この参照用アーキテクチャの Resource Manager テンプレートでは管理対象ディスクは使用されていません。 管理対象ディスクを使用するようにテンプレートを更新する予定です。

SharePoint および SQL Server のすべての VM で Premium 管理対象ディスクを使用してください。 マジョリティ ノード サーバー、ドメイン コントローラー、および管理サーバでは、Standard 管理対象ディスクを使用できます。 

### <a name="sharepoint-server-recommendations"></a>SharePoint Server の推奨事項

SharePoint ファームを構成する前に、サービスごとに 1 つの Windows Server Active Directory サービス アカウントがあることを確認します。 このアーキテクチャでは、ロールごとに特権を分離するために、少なくとも次に示すドメインレベル アカウントが必要です。

- SQL Server サービス アカウント
- セットアップ ユーザー アカウント
- サーバー ファーム アカウント
- Search Service アカウント
- コンテンツ アクセス アカウント
- Web アプリ プール アカウント
- サービス アプリ プール アカウント
- キャッシュ スーパー ユーザー アカウント
- キャッシュ スーパー リーダー アカウント

Search インデクサーを除くすべてのロールについて、[Standard_DS3_v2][vm-sizes-general] VM サイズの使用を推奨します。 Search インデクサーは少なくとも [Standard_DS13_v2][vm-sizes-memory] のサイズにする必要があります。 

> [!NOTE]
> この参照用アーキテクチャの Resource Manager テンプレートでは、デプロイをテストするという目的に合わせて Search インデクサーについて小さな DS3 のサイズを使用しています。 実稼働デプロイでは DS13 以上のサイズを使用してください。 

実稼働ワークロードについて詳しくは、「[SharePoint Server 2016 のハードウェア要件およびソフトウェア要件][sharepoint-reqs]」を参照してください。 

最小 200 MB/s のディスク スループットのサポート要件を満たすには、必ず検索アーキテクチャを計画してください。 「[SharePoint Server 2013 でエンタープライズ検索アーキテクチャを計画する][sharepoint-search]」をご覧ください。 また、「[Best practices for crawling in SharePoint Server 2016][sharepoint-crawling](SharePoint Server 2016 のクロールのベスト プラクティス)」のガイドラインに従います。

さらに、検索コンポーネント データを、パフォーマンスに優れている別の記憶域ボリュームまたはパーティションに格納します。 負荷を減らしてスループットを向上するには、オブジェクト キャッシュ ユーザー アカウントを構成します。これはこのアーキテクチャで必要です。 Windows Server オペレーティング システム ファイル、SharePoint Server 2016 プログラム ファイル、および診断ログは、通常のパフォーマンスの 3 つの記憶域ボリュームまたはパーティションに分割します。 

これらの推奨事項について詳しくは、「[SharePoint Server 2016 での初期展開の管理およびサービス アカウント][sharepoint-accounts]」をご覧ください。

### <a name="hybrid-workloads"></a>ハイブリッド ワークロード

この参照用アーキテクチャでは、[SharePoint ハイブリッド環境][sharepoint-hybrid]&mdash;として使用できる SharePoint Server 2016 ファームがデプロイされます。つまり、SharePoint Server 2016 を Office 365 SharePoint Online に拡張しています。 Office Online Server がある場合は、「[Office Web Apps and Office Online Server supportability in Azure][office-web-apps](Azure での Office Web Apps および Office Online Server のサポート)」をご覧ください。

このデプロイにおける既定のサービス アプリケーションは、ハイブリッド ワークロードに対応するように設計されています。 SharePoint インフラストラクチャを変更せずに、SharePoint Server 2016 および Office 365 のすべてのハイブリッド ワークロードをこのファームにデプロイできます。ただし、1 つの例外として、Cloud Hybrid Search Service Application は、既存の検索トポロジをホストするサーバーにデプロイしないでください。 したがって、このハイブリッド シナリオに対応するためには、1 つ以上の検索ロール用の VM をファームに追加する必要があります。

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性グループ

SharePoint Server 2016 は Azure SQL Database を使用できないため、このアーキテクチャでは SQL Server 仮想マシンが使用されます。 SQL Server の高可用性をサポートするために、Always On 可用性グループの使用を推奨します。これによって、一緒にフェールオーバーする一連のデータベースが指定され、それらのデータベースの可用性が向上して復旧可能になります。 この参照用アーキテクチャではデータベースはデプロイ時に作成されますが、手動で Always On 可用性グループを有効にして、SharePoint データベースを可用性グループに追加する必要があります。 詳しくは、[可用性グループの作成と SharePoint データベースの追加][create-availability-group]に関する記事をご覧ください。

また、リスナー IP アドレスのクラスターへの追加も推奨します。これは、SQL Server 仮想マシンの内部ロード バランサーのプライベート IP アドレスです。

Azure で実行する SQL Server の推奨 VM サイズやその他のパフォーマンスの推奨事項について詳しくは、「[Performance best practices for SQL Server in Azure Virtual Machines][sql-performance]」(Azure 仮想マシンでの SQL Server のパフォーマンス ベスト プラクティス) をご覧ください。 「[SharePoint Server 2016 ファーム内の SQL Server のベスト プラクティス][sql-sharepoint-best-practices]」の推奨事項にも従ってください。

マジョリティ ノード サーバーはレプリケーション パートナーとは別のコンピューターに配置することを推奨します。 このサーバーによって、高度セーフティ モード セッションでセカンダリ パートナー サーバーが、自動フェールオーバーを開始するかどうかを認識できるようになります。 2 つのパートナーとは異なり、マジョリティ ノード サーバーはデータベースでは使用されず、自動フェールオーバーをサポートします。 

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

既存のサーバーを拡張するには、VM サイズを変更するだけですみます。 

SharePoint Server 2016 の [MinRoles][minroles] 機能により、サーバーのロールに基づいてサーバーをスケールアウトできます。ロールからサーバーを削除することもできます。 サーバーをロールに追加するときは、単独ロールのいずれか、または組み合わされたロールのいずれかを指定できます。 ただし、サーバーを検索ロールに追加する場合は、PowerShell を使用して検索トポロジも再構成する必要があります。 MinRoles を使用してロールを変換することもできます。 詳しくは、「[SharePoint Server 2016 での MinRole サーバー ファームの管理][sharepoint-minrole]」をご覧ください。

SharePoint Server 2016 では自動スケーリングのための仮想マシン スケール セットの使用はサポートされないことに注意してください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

この参照用アーキテクチャでは、Azure リージョン内の高可用性がサポートされます。ロールごとに少なくとも 2 つの VM が可用性セット内にデプロイされているためです。

リージョン障害に対して保護するには、異なる Azure リージョンに別のディザスター リカバリー ファームを作成します。 目標復旧時間 (RTO) と目標復旧時点 (RPO) によって設定の要件が決まります。 詳しくは、[SharePoint Server 2016 用のディザスター リカバリー戦略の選択][sharepoint-dr]に関する記事をご覧ください。 セカンダリ リージョンは、プライマリ リージョンと "*ペアになっているリージョン*" であることが必要です。 広範囲にわたって障害が発生した場合は、すべてのペアで一方のリージョンの復旧が優先されます。 詳しくは、「[ビジネス継続性とディザスター リカバリー (BCDR): Azure のペアになっているリージョン][paired-regions]」をご覧ください。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

サーバー、サーバー ファーム、およびサイトを運用して管理するには、SharePoint の運用の推奨プラクティスに従います。 詳しくは、[SharePoint Server 2016 の運用][sharepoint-ops]に関する記事をご覧ください。

SharePoint 環境で SQL Server を管理する際に検討するタスクは、データベース アプリケーションで一般的に検討されるタスクとは異なる場合があります。 ベスト プラクティスは、すべての SQL データベースを週 1 回完全にバックアップし、さらに毎晩増分バックアップすることです。 トランザクション ログは 15 分間隔でバックアップします。 別のプラクティスでは、SQL Server メンテナンス タスクをデータベースに実装し、組み込みの SharePoint メンテナンス タスクは無効にします。 詳しくは、「[ストレージおよび SQL Server の容量計画と構成 ][sql-server-capacity-planning]」をご覧ください。 

## <a name="security-considerations"></a>セキュリティに関する考慮事項

SharePoint Server 2016 の実行に使用されるドメインレベル サービス アカウントでは、ドメイン参加や認証のプロセスで Windows Server AD ドメイン コントローラーが必要です。 Azure Active Directory Domain Services はこの目的には使用できません。 イントラネットに既に配置されている Windows Server AD の ID インフラストラクチャを拡張するために、このアーキテクチャは、既存のオンプレミス Windows Server AD フォレストの 2 つの Windows Server AD レプリカ ドメイン コントローラーを使用します。

また、セキュリティ強化を計画するのは常に賢明です。 他の推奨事項は次のとおりです。

- NSG にルールを追加してサブネットとロールを分離します。
- VM にパブリック IP アドレスを割り当てないでください。
- 侵入の検出とペイロードの分析のためには、内部 Azure ロード バランサーではなくフロントエンド Web サーバーの前での、ネットワーク仮想アプライアンスの使用を検討します。
- オプションとして、サーバー間のクリア テキスト トラフィックの暗号化のために IPsec ポリシーを使用します。 サブネットの分離も実行する場合は、IPsec トラフィックを許可するようにネットワーク セキュリティ グループ ルールを更新します。
- VM 用のマルウェア対策エージェントをインストールします。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照用アーキテクチャのデプロイ スクリプトについては、[GitHub][github] をご覧ください。 

このアーキテクチャは段階的にデプロイすることも一度にデプロイすることもできます。 最初は、各デプロイの内容を確認できるように、段階的なデプロイをお勧めします。 次のいずれかの "*モード*" パラメーターを使用して段階を指定します。

| Mode           | 実行内容                                                                                                            |
|----------------|-------------------------------------------------------------------------------------------------------------------------|
| onprem         | (省略可能) テストまたは評価のために、シミュレーション オンプレミス ネットワーク環境をデプロイします。 この手順では実際のオンプレミス ネットワークには接続しません。 |
| infrastructure | SharePoint 2016 ネットワーク インフラストラクチャと jumpbox を Azure にデプロイします。                                                |
| createvpn      | SharePoint とオンプレミス ネットワークの両方に仮想ネットワーク ゲートウェイをデプロイし、接続します。 この手順は、`onprem` 手順を実行した場合のみ実行します。                |
| ワークロード       | SharePoint サーバーを SharePoint ネットワークにデプロイします。                                                               |
| security       | ネットワーク セキュリティ グループを SharePoint ネットワークにデプロイします。                                                           |
| すべて            | 上記のすべてをデプロイします。                            


シミュレーション オンプレミス ネットワーク環境に対してアーキテクチャを段階的にデプロイするには、次の手順を順番に実行します。

1. onprem
2. インフラストラクチャ
3. createvpn
4. ワークロード
5. security

シミュレーション オンプレミス ネットワーク環境なしで、アーキテクチャを段階的にデプロイするには、次の手順を順番に実行します。

1. インフラストラクチャ
2. ワークロード
3. security

すべてを 1 回の手順でデプロイするには `all` を使用します。 プロセス全体に数時間かかる場合があることに注意してください。

### <a name="prerequisites"></a>前提条件

* 最新バージョンの [Azure PowerShell][azure-ps] をインストールします。

* この参照用アーキテクチャをデプロイする前に、サブスクリプションに十分なクォータ (38 個以上のコア) があることを確認します。 不足している場合は、Azure ポータルを使用して、クォータを増やすためのサポート要求を発行します。

* このデプロイのコストを見積もるには、[料金計算ツール][azure-pricing]をご覧ください。

### <a name="deploy-the-reference-architecture"></a>参照用アーキテクチャのデプロイ

1.  [GitHub リポジトリ][github]をローカル コンピューターにダウンロードするか複製します。

2.  PowerShell ウィンドウを開き、`/sharepoint/sharepoint-2016` フォルダーに移動します。

3.  次の PowerShell コマンドを実行します。 \<subscription id\> として Azure サブスクリプション ID を指定します。 \<location\> には、Azure リージョン (`eastus` や `westus` など) を指定します。 \<mode\> には、`onprem``infrastructure`、`createvpn`、`workload``security`、または `all` を指定します。

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```   
4. プロンプトが表示されたら、Azure アカウントにログオンします。 選択したモードによって異なりますが、デプロイ スクリプトは完了するまでに数時間かかることがあります。

5. デプロイが完了したら、スクリプトを実行して SQL Server Always On 可用性グループを構成します。 詳しくは、[readme][readme] をご覧ください。

> [!WARNING]
> パラメーター ファイルには、ハードコーディングされたパスワード (`AweS0me@PW`) がさまざまな場所に含まれています。 デプロイする前にこれらの値を変更します。


## <a name="validate-the-deployment"></a>デプロイの検証

この参照用アーキテクチャをデプロイすると、次のリソース グループが使用したサブスクリプションの下に一覧表示されます。

| リソース グループ        | 目的                                                                                         |
|-----------------------|-------------------------------------------------------------------------------------------------|
| ra-onprem-sp2016-rg   | オンプレミス ネットワークの Active Directory とのシミュレーションおよび SharePoint 2016 ネットワークとのフェデレーション |
| ra-sp2016-network-rg  | SharePoint デプロイをサポートするインフラストラクチャ                                                 |
| ra-sp2016-workload-rg | SharePoint およびサポート リソース                                                             |

### <a name="validate-access-to-the-sharepoint-site-from-the-on-premises-network"></a>オンプレミス ネットワークから SharePoint サイトへのアクセスの検証

1. [Azure ポータル][azure-portal]の **[リソース グループ]** の下で `ra-onprem-sp2016-rg` リソース グループを選択します。

2. リソースの一覧で、`ra-adds-user-vm1` という名前の VM リソースを選択します。 

3. 「[仮想マシンへの接続][connect-to-vm]」の説明に従って、VM に接続します。 ユーザー名は `\onpremuser` です。

5.  VM へのリモート接続が確立されたら、VM をブラウザーで開いて `http://portal.contoso.local` に移動します。

6.  **[Windows セキュリティ]** ボックスで、ユーザー名として `contoso.local\testuser` を使用して SharePoint ポータルにログオンします。

このログオンは、オンプレミス ネットワークで使用される Fabrikam.com ドメインから SharePoint ポータルで使用される contoso.local ドメインにトンネリングします。 SharePoint サイトが開くと、ルート デモ サイトが表示されます。

### <a name="validate-jumpbox-access-to-vms-and-check-configuration-settings"></a>jumpbox の VM へのアクセスの検証と構成設定の確認

1.  [Azure ポータル][azure-portal]の **[リソース グループ]** の下で `ra-sp2016-network-rg` リソース グループを選択します。

2.  リソースの一覧で、`ra-sp2016-jb-vm1` という名前の VM リソースを選択します。これが jumpbox です。

3. 「[仮想マシンへの接続][connect-to-vm]」の説明に従って、VM に接続します。 ユーザー名は `testuser` です。

4.  jumpbox にログオンした後で、jumpbox から RDP セッションを開きます。 VNet 内の他の VM に接続します。 ユーザー名は `testuser` です。 リモート コンピューターのセキュリティ証明書に関する警告は無視してかまいません。

5.  VM へのリモート接続が開いたら、構成を確認し、サーバー マネージャーなど管理ツールを使用して変更を加えます。

次の表には、デプロイされる VM を示します。 

| リソース名      | 目的                                   | リソース グループ        | VM 名                       |
|--------------------|-------------------------------------------|-----------------------|-------------------------------|
| Ra-sp2016-ad-vm1   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad1.contoso.local             |
| Ra-sp2016-ad-vm2   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad2.contoso.local             |
| Ra-sp2016-fsw-vm1  | SharePoint                                | Ra-sp2016-network-rg  | Fsw1.contoso.local            |
| Ra-sp2016-jb-vm1   | Jumpbox                                   | Ra-sp2016-network-rg  | Jb (パブリック IP を使用してログオン) |
| Ra-sp2016-sql-vm1  | SQL Always On - フェールオーバー                  | Ra-sp2016-network-rg  | Sq1.contoso.local             |
| Ra-sp2016-sql-vm2  | SQL Always On - プライマリ                   | Ra-sp2016-network-rg  | Sq2.contoso.local             |
| Ra-sp2016-app-vm1  | SharePoint 2016 アプリケーション MinRole       | Ra-sp2016-workload-rg | App1.contoso.local            |
| Ra-sp2016-app-vm2  | SharePoint 2016 アプリケーション MinRole       | Ra-sp2016-workload-rg | App2.contoso.local            |
| Ra-sp2016-dch-vm1  | SharePoint 2016 分散キャッシュ MinRole | Ra-sp2016-workload-rg | Dch1.contoso.local            |
| Ra-sp2016-dch-vm2  | SharePoint 2016 分散キャッシュ MinRole | Ra-sp2016-workload-rg | Dch2.contoso.local            |
| Ra-sp2016-srch-vm1 | SharePoint 2016 検索 MinRole            | Ra-sp2016-workload-rg | Srch1.contoso.local           |
| Ra-sp2016-srch-vm2 | SharePoint 2016 検索 MinRole            | Ra-sp2016-workload-rg | Srch2.contoso.local           |
| Ra-sp2016-wfe-vm1  | SharePoint 2016 Web フロント エンド MinRole     | Ra-sp2016-workload-rg | Wfe1.contoso.local            |
| Ra-sp2016-wfe-vm2  | SharePoint 2016 Web フロント エンド MinRole     | Ra-sp2016-workload-rg | Wfe2.contoso.local            |


**_この参照用アーキテクチャの共同作成者_** &mdash; Joe Davies、Bob Fox、Neil Hodgkinson、Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/en-us/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.azureedge.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
