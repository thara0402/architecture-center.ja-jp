---
title: 高可用性を得るために複数の Azure リージョンで Linux VM を実行する
description: 高可用性と回復性を得るために Azure の複数のリージョンに VM をデプロイする方法。
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 07ccf44f28203e6d5001475b47adce01437e9600
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>高可用性を得るために複数のリージョンで Linux VM を実行する

このリファレンス アーキテクチャは、可用性と堅牢な災害復旧インフラストラクチャを実現するために、複数の Azure リージョンで N 層アプリケーションを実行するための一連の実証済みのプラクティスを示しています。 

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ 

このアーキテクチャは、「[Run Windows VMs for an N-tier application](n-tier.md)」(N 層アプリケーションでの Windows VM の実行) に示されているアーキテクチャの上に構築されています。 

* **プライマリ リージョンとセカンダリ リージョン**。 2 つのリージョンを使用して高可用性を実現します。 1 つはプライマリ リージョンであり、他方のリージョンはフェールオーバー用です。
* **Azure DNS**。 [Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。
* **Azure Traffic Manager**。 [Traffic Manager][traffic-manager] は、着信要求をいずれかのリージョンにルーティングします。 通常の運用中は、プライマリ リージョンに要求をルーティングします。 そのリージョンが使用できなくなった場合、Traffic Manager はセカンダリ リージョンへのフェールオーバーを実行します。 詳細については、「[Traffic Manager の構成](#traffic-manager-configuration)」を参照してください。
* **リソース グループ**。 プライマリ リージョン、セカンダリ リージョン、および Traffic Manager 用の個別の[リソース グループ][resource groups]を作成します。 これにより、各リージョンをリソースの 1 つのコレクションとして柔軟に管理できます。 たとえば、片方のリージョンの再デプロイを、他方のリージョンをダウンさせずに実行できます。 [リソース グループをリンク][resource-group-links]して、アプリケーション用のすべてのリソースを一覧表示するクエリを実行できるようにします。
* **VNets**。 リージョンごとに個別の VNet を作成します。 アドレス空間が重複していないことを確認してください。
* **Apache Cassandra**。 高可用性を得るために、Azure リージョンのデータセンターに Cassandra をデプロイします。 各リージョンのノードは、リージョン内の回復性を高めるために、障害ドメインとアップグレード ドメインを持つラック認識モードで構成されます。

## <a name="recommendations"></a>Recommendations

マルチリージョン アーキテクチャは、単一のリージョンにデプロイするよりも高い可用性を提供できます。 地域的な停止がプライマリ リージョンに影響する場合は、[Traffic Manager][traffic-manager] を使用して、セカンダリ リージョンにフェールオーバーできます。 このアーキテクチャは、アプリケーションの個々のサブシステムが失敗した場合にも役立ちます。

リージョン間で高可用性を実現する一般的な方法はいくつかあります。   

* アクティブ/パッシブ (ホット スタンバイ)。 トラフィックが片方のリージョンにルーティングされている間、他方のリージョンは、ホット スタンバイ状態で待機します。 ホット スタンバイとは、セカンダリ リージョン内の VM が割り当て済みであり、常に実行されていることを意味します。
* アクティブ/パッシブ (コールド スタンバイ)。 トラフィックが片方のリージョンにルーティングされている間、他方のリージョンは、コールド スタンバイ状態で待機します。 コールド スタンバイとは、セカンダリ リージョン内の VM がフェールオーバーが必要になるまで割り当てられないことを意味します。 この方法のほうが実行コストは低くなりますが、ほとんどの場合、障害発生時にオンラインになるまでの時間が長くなります。
* アクティブ/アクティブ。 両方のリージョンがアクティブであり、要求はそれらの間で負荷分散されます。 片方のリージョンが使用できなくなった場合は、ローテーションから外されます。 

このアーキテクチャでは、Traffic Manager を使用してフェールオーバーで するアクティブ/パッシブ (ホット スタンバイ) に焦点を当てています。 ホット スタンバイ用の少数の VM をデプロイした後、必要に応じてスケール アウトできることに注意してください。


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

### <a name="cassandra-deployment-across-multiple-regions"></a>複数のリージョンでの Cassandra のデプロイ

Cassandra データ センターとは、レプリケーションとワークロードを分離するためにクラスター内にまとめて構成されている関連性のあるデータ ノードのグループです。

実稼働環境では [DataStax Enterprise][datastax] の使用をお勧めします。 Azure での DataStax の実行の詳細については、「[DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure]」(Azure 用の DataStax Enterprise Deployment ガイド) を参照してください。 すべての Cassandra エディションに次の一般的な推奨事項が適用されます。 

* 各ノードにパブリック IP アドレスを割り当てます。 これにより、クラスターは、Azure バックボーン インフラストラクチャを使用してリージョン間で通信でき、高いスループットを低コストで提供します。
* 適切なファイアウォールとネットワーク セキュリティ グループ (NSG) 構成を使用してノードをセキュリティ保護して、クライアントと他のクラスター ノードも含めて、既知のホストに対するトラフィックのみを許可します。 Cassandra は、通信、OpsCenter、Spark などのために、異なるポートを使用します。 Cassandra のポートの使用方法については、「[Configuring firewall port access][cassandra-ports]」(ファイアウォールのポートのアクセスの構成) を参照してください。
* すべての[クライアントからノードへ][ssl-client-node]の通信と[ノードからノードへ][ssl-node-node]の通信で、SSL 暗号化を使用します。
* リージョン内では、[Cassandra に関する推奨事項](n-tier.md#cassandra)のガイドラインに従います。

## <a name="availability-considerations"></a>可用性に関する考慮事項

複雑な N 層アプリケーションでは、アプリケーション全体をセカンダリ リージョンにレプリケートする必要はない場合があります。 代わりに、ビジネス継続性をサポートするために必要な重要なサブシステムのみをレプリケートできます。

Traffic Manager は、システムの障害ポイントになる可能性があります。 Traffic Manager サービスが失敗すると、クライアントは、ダウンタイム中はアプリケーションにアクセスできなくなります。 「[Traffic Manager の SLA][tm-sla]」を確認して、Traffic Manager の使用だけで高可用性のビジネス要件が満たされるかどうかを確かめてください。 満たされない場合は、フェールバックとして別のトラフィック管理ソリューションを追加することを検討してください。 Azure Traffic Manager サービスで障害が発生した場合は、他のトラフィック管理サービスを参照するように、DNS の CNAME レコードを変更します  (この手順は手動で実行する必要があり、DNS の変更が反映されるまでアプリケーションを使用することはできません)。

Cassandra クラスターのために検討するフェールオーバー シナリオは、アプリケーションで使用される一貫性レベルと使用されるレプリカの数によって異なります。 Cassandra での一貫性レベルと使用については、「[Configuring data consistency][cassandra-consistency]」(データの一貫性の構成) と「[Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage]」(Cassandra: Quorum を使用してトークできるノードの数) を参照してください。 Cassandra でのデータの可用性は、アプリケーションで使用される一貫性レベルとレプリケーション メカニズムによって決まります。 Cassandra のレプリケーションについては、「[Data Replication in NoSQL Databases Explained][cassandra-replication]」(NoSQL Database でのデータレプリケーションの説明) を参照してください。

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
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Azure の N 層アプリケーション用の可用性の高いネットワーク アーキテクチャ"
