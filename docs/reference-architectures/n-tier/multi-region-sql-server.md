---
title: 高可用性のためのマルチリージョン n 層アプリケーション
description: 高可用性と回復性を得るために Azure の複数のリージョンに VM をデプロイする方法。
author: MikeWasson
ms.date: 05/03/2018
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 48943094e7847e39b9fdc4c3f71e27f2e6e41293
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2018
---
# <a name="multi-region-n-tier-application-for-high-availability"></a>高可用性のためのマルチリージョン n 層アプリケーション

このリファレンス アーキテクチャは、可用性と堅牢な災害復旧インフラストラクチャを実現するために、複数の Azure リージョンで N 層アプリケーションを実行するための一連の実証済みのプラクティスを示しています。 

[![0]][0] 

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ 

このアーキテクチャは、「[SQL Server を使用した n 層アプリケーション](n-tier-sql-server.md)」に示されているアーキテクチャの上に構築されています。 

* **プライマリ リージョンとセカンダリ リージョン**。 2 つのリージョンを使用して高可用性を実現します。 1 つはプライマリ リージョンであり、 他方のリージョンはフェールオーバー用です。

* **Azure Traffic Manager**。 [Traffic Manager][traffic-manager] は、着信要求をいずれかのリージョンにルーティングします。 通常の運用中は、プライマリ リージョンに要求をルーティングします。 そのリージョンが使用できなくなった場合、Traffic Manager はセカンダリ リージョンへのフェールオーバーを実行します。 詳細については、「[Traffic Manager の構成](#traffic-manager-configuration)」を参照してください。

* **リソース グループ**。 プライマリ リージョン、セカンダリ リージョン、および Traffic Manager 用の個別の[リソース グループ][resource groups]を作成します。 これにより、各リージョンをリソースの 1 つのコレクションとして柔軟に管理できます。 たとえば、片方のリージョンの再デプロイを、他方のリージョンをダウンさせずに実行できます。 [リソース グループをリンク][resource-group-links]して、アプリケーション用のすべてのリソースを一覧表示するクエリを実行できるようにします。

* **VNets**。 リージョンごとに個別の VNet を作成します。 アドレス空間が重複していないことを確認してください。 

* **SQL Server Always On 可用性グループ**。 SQL Server を使用している場合は、[SQL Always On 可用性グループ][sql-always-on]を使用して高可用性を実現することをお勧めします。 両方のリージョンの SQL Server インスタンスを含む単一の可用性グループを作成します。 

    > [!NOTE]
    > [Azure SQL Database][azure-sql-db] の使用も検討してください。リレーショナル データベースがクラウドサービスとして提供されます。 SQL Database では、可用性グループの構成やフェールオーバーの管理は必要ありません。  
    > 

* **VPN ゲートウェイ**。 各 VNet 内に [VPN ゲートウェイ][vpn-gateway]を作成し、[VNet 間接続][vnet-to-vnet]を構成して、2 つの VNet 間のネットワーク トラフィックが可能になるようにします。 これは、SQL Always On 可用性グループで必要です。

## <a name="recommendations"></a>Recommendations

マルチリージョン アーキテクチャは、単一のリージョンにデプロイするよりも高い可用性を提供できます。 地域的な停止がプライマリ リージョンに影響する場合は、[Traffic Manager][traffic-manager] を使用して、セカンダリ リージョンにフェールオーバーできます。 このアーキテクチャは、アプリケーションの個々のサブシステムが失敗した場合にも役立ちます。

リージョン間で高可用性を実現する一般的な方法はいくつかあります。 

* アクティブ/パッシブ (ホット スタンバイ)。 トラフィックが片方のリージョンにルーティングされている間、他方のリージョンは、ホット スタンバイ状態で待機します。 ホット スタンバイとは、セカンダリ リージョン内の VM が割り当て済みであり、常に実行されていることを意味します。
* アクティブ/パッシブ (コールド スタンバイ)。 トラフィックが片方のリージョンにルーティングされている間、他方のリージョンは、コールド スタンバイ状態で待機します。 コールド スタンバイとは、セカンダリ リージョン内の VM がフェールオーバーが必要になるまで割り当てられないことを意味します。 この方法のほうが実行コストは低くなりますが、ほとんどの場合、障害発生時にオンラインになるまでの時間が長くなります。
* アクティブ/アクティブ。 両方のリージョンがアクティブであり、要求はそれらの間で負荷分散されます。 片方のリージョンが使用できなくなった場合は、ローテーションから外されます。 

この参照アーキテクチャでは、Traffic Manager を使用してフェールオーバーを行うアクティブ/パッシブ (ホット スタンバイ) に焦点を当てています。 ホット スタンバイ用の少数の VM をデプロイした後、必要に応じてスケール アウトできることに注意してください。

### <a name="regional-pairing"></a>リージョンのペアリング

各 Azure リージョンは、同じ地区内の別のリージョンとペアリングされます。 通常は、同じリージョン ペアからリージョンを選択します (たとえば、米国東部 2 と米国中部)。 これには、次のような利点があります。

* 広範囲にわたる停止が発生した場合は、すべてのペアで、少なくとも 1 つのリージョンの復旧が優先的に実行されます。
* Azure システムの計画的更新は、起こり得るダウンタイムを最小限に抑えるために、ペアになっているリージョンに対して順にロールアウトされます。
* ペアは、データの所在地要件を満たすために同じ地区内に所在します。 

ただし、両方のリージョンでアプリケーションに必要なすべての Azure サービスがサポートされていることを確認してください ([リージョン別サービス][services-by-region]に関する記事を参照してください)。 リージョン ペアの詳細については、「[ビジネス継続性とディザスター リカバリー (BCDR): Azure のペアになっているリージョン][regional-pairs]」を参照してください。

### <a name="traffic-manager-configuration"></a>Traffic Manager の構成

Traffic Manager を構成するときは、次の点を検討してください。

* **ルーティング**。 Traffic Manager は、さまざまな[ルーティング アルゴリズム][tm-routing]をサポートしています。 この記事で説明するシナリオでは、"*優先度による*" ルーティング (旧称 "*フェールオーバー*" ルーティング) を使用します。 この設定では、プライマリ リージョンが到達不能にならない限り、Traffic Manager はプライマリ リージョンにすべての要求を送信します。 到達不能になった時点で、セカンダリ リージョンに自動的にフェールオーバーします。 [フェールオーバーのルーティング方法の構成][tm-configure-failover]に関する記事を参照してください。
* **正常性プローブ**。 Traffic Manager は、HTTP (または HTTPS) [プローブ][tm-monitoring]を使用して、各リージョンの可用性を監視します。 プローブは、特定の URL パスの HTTP 200 応答をチェックします。 ベスト プラクティスとして、アプリケーションの全体的な正常性を報告するエンドポイントを作成し、そのエンドポイントを正常性プローブ用に使用します。 これを行わなかった場合、プローブは、アプリケーションの重要な部分で実際には障害が発生しているにもかかわらず、エンドポイントが正常であると報告する可能性があります。 詳細については、[正常性エンドポイント監視パターン][health-endpoint-monitoring-pattern]に関するページを参照してください。   

Traffic Manager がフェールオーバーを実行すると、クライアントがアプリケーションに到達できない時間が発生します。 この持続時間は、次の要因に影響されます。

* 正常性プローブが、プライマリ リージョンが到達不能になっていることを検出する必要があります。
* DNS サーバーが、IP アドレスのキャッシュされた DNS レコードを更新する必要があります。これは DNS 有効期限 (TTL) に依存します。 TTL の既定値は 300 秒 (5 分) ですが、この値は、Traffic Manager プロファイルを作成するときに構成できます。

詳細については、「[Traffic Manager の監視について][tm-monitoring]」を参照してください。

Traffic Manager でフェールオーバーを実行する場合は、自動フェールバックを実装するのではなく、手動でフェールバックを実行することをお勧めします。 これを行わなかった場合、リージョン間でアプリケーションが切り替わる状況が発生する可能性があります。 フェールバックする前に、すべてのアプリケーション サブシステムが正常であることを確認します。

Traffic Manager は、既定では自動的にフェールバックすることに注意してください。 これが起こらないようにするには、フェールオーバー イベントの後、手動でプライマリ リージョンの優先度を下げます。 たとえば、プライマリ リージョンの優先度は 1、セカンダリ リージョンの優先度は 2 であるとします。 フェールオーバーした後、プライマリ リージョンの優先度を 3 に設定して、自動フェールバックが起こらないにします。 元に戻す準備ができたら、優先度を 1 に更新します。

次の [Azure CLI][install-azure-cli] コマンドは、優先度を更新します。

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

別の方法は、フェールバックの準備ができるまで、エンドポイントを一時的に無効にすることです。

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```

フェールオーバーの原因によっては、リソースをリージョン内に再デプロイする必要があります。 フェールバックする前に、運用準備テストを実行します。 このテストでは、以下のような点を検証する必要があります。

* VM が正しく構成されている  (すべての必要なソフトウェアがインストールされていること、IIS が実行されていることなど)。
* アプリケーションのサブシステムが正常である。 
* 機能テスト  (たとえば、Web 層からデータベース層に到達可能である)。

### <a name="configure-sql-server-always-on-availability-groups"></a>SQL Server Always On 可用性グループを構成する

Windows Server 2016 より前に、SQL Server Always On 可用性グループではドメイン コントローラーを必要とし、可用性グループ内のすべてのノードが同じ Active Directory (AD) ドメイン内にある必要があります。 

可用性グループを構成するには:

* 各リージョンに少なくとも 2 つのドメイン コントローラーを配置します。
* 各ドメイン コントローラーに静的 IP アドレスを指定します。
* VNet 間接続を作成して、VNet 間の通信を可能にします。
* 各 VNet で、DNS サーバーの一覧に (両方のリージョンの) ドメイン コントローラーの IP アドレスを追加します。 次の CLI コマンドを使用できます。 詳細については、[仮想ネットワーク (VNet) で使用される DNS サーバーの管理][vnet-dns]に関する記事を参照してください。

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* 両方のリージョンの SQL Server インスタンスを含む [Windows Server フェールオーバー クラスタリング][wsfc] (WSFC) クラスターを作成します。 
* プライマリ リージョンとセカンダリ リージョンの SQL Server インスタンスを含む SQL Server Always On 可用性グループを作成します。 手順については、[Always On 可用性グループのリモート Azure データセンターへの拡張 (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/)に関する記事を参照してください。

  * プライマリ レプリカをプライマリ リージョンに配置します。
  * 1 つ以上のセカンダリ レプリカをプライマリ リージョンに配置します。 それらを自動フェールオーバーを伴う同期コミット モードを使用するように構成します。
  * 1 つ以上のセカンダリ レプリカをセカンダリ リージョンに配置します。 パフォーマンス上の理由で、それらは*非同期*コミットを使用するように構成します  (これを行わなかった場合、すべての T-SQL トランザクションがセカンダリ リージョンへのネットワーク上にラウンド トリップで待機する必要があります)。

    > [!NOTE]
    > 非同期コミット レプリカは、自動フェールオーバーをサポートしません。
    >
    >

## <a name="availability-considerations"></a>可用性に関する考慮事項

複雑な N 層アプリケーションでは、アプリケーション全体をセカンダリ リージョンにレプリケートする必要はない場合があります。 代わりに、ビジネス継続性をサポートするために必要な重要なサブシステムのみをレプリケートできます。

Traffic Manager は、システムの障害ポイントになる可能性があります。 Traffic Manager サービスが失敗すると、クライアントは、ダウンタイム中はアプリケーションにアクセスできなくなります。 「[Traffic Manager の SLA][tm-sla]」を確認して、Traffic Manager の使用だけで高可用性のビジネス要件が満たされるかどうかを確かめてください。 満たされない場合は、フェールバックとして別のトラフィック管理ソリューションを追加することを検討してください。 Azure Traffic Manager サービスで障害が発生した場合は、他のトラフィック管理サービスを参照するように、DNS の CNAME レコードを変更します  (この手順は手動で実行する必要があり、DNS の変更が反映されるまでアプリケーションを使用することはできません)。

SQL Server クラスターでは、2 つのフェールオーバー シナリオを考慮する必要があります。

- プライマリ リージョン内のすべての SQL Server データベースのレプリカが失敗する。 これは、たとえば地域的な停止中に発生することがあります。 この場合は、Traffic Manager がフロント エンドで自動的にフェールオーバーを実行する場合でも、可用性グループを手動でフェールオーバーする必要があります。 「[Perform a Forced Manual Failover of a SQL Server Availability Group](https://msdn.microsoft.com/library/ff877957.aspx)」(SQL Server 可用性グループの強制手動フェールオーバーを実行する) の手順に従います。この記事では、SQL Server 2016 で SQL Server Management Studio、Transact-SQL、または PowerShell を使用して強制フェールオーバーを実行する方法が説明されています。

   > [!WARNING]
   > 強制フェールオーバーには、データ損失のリスクがあります。 プライマリ リージョンがオンラインに戻ったら、データベースのスナップショットを取得し、[tablediff] を使用して差異を検出してください。
   >
   >
- Traffic Manager がセカンダリ リージョンへのフェールオーバーを実行するが、SQL Server データベースのプライマリ レプリカが引き続き使用可能である。 たとえば SQL Server VM に影響しない障害がフロント エンド層で発生することがあります。 この場合、インターネット トラフィックはセカンダリ リージョンにルーティングされますが、セカンダリ リージョンは引き続きプライマリ レプリカに接続できます。 ただし、SQL Server の接続がリージョンにまたがるため、待機時間が長くなります。 この状況では、次のように手動フェールオーバーを実行する必要があります。

   1. セカンダリ リージョンの SQL Server データベースのレプリカを一時的に*同期*コミットに切り替えます。 これにより、フェールオーバー中にデータの損失が発生しないことが保証されます。
   2. そのレプリカにフェールオーバーします。
   3. プライマリ リージョンにフェールバックするときに、設定を非同期コミットに戻します。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

デプロイを更新するときは、一度に 1 つのリージョンを更新することで、アプリケーションの不適切な構成やエラーによってグローバル エラーが発生する機会を減らします。

システムのエラーに対する回復性をテストします。 テストされる一般的な障害シナリオを次に示します。

* VM インスタンスのシャットダウン。
* CPU やメモリなどのリソースへの負荷。
* ネットワークの切断/遅延。
* プロセスのクラッシュ。
* 証明書の期限切れ。
* ハードウェア障害のシミュレート。
* ドメイン コントローラー上の DNS サービスのシャットダウン。

回復時間を測定し、ビジネス要件を満たしていることを確認します。 障害モードの組み合わせもテストします。



<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-sql-server.png "Azure の N 層アプリケーション用の可用性の高いネットワーク アーキテクチャ"
