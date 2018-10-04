---
title: 規制対象業界向けのセキュリティで保護された Windows Web アプリケーション
description: スケール セット、Application Gateway、ロード バランサーを使用する、セキュリティで保護された多層 Web アプリケーションを、Azure 上の Windows Server を使用して構築するための実証済みのシナリオ。
author: iainfoulds
ms.date: 07/11/2018
ms.openlocfilehash: 780b82791510b6ca06ef918b66d2547794dfcf87
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428756"
---
# <a name="secure-windows-web-application-for-regulated-industries"></a>規制対象業界向けのセキュリティで保護された Windows Web アプリケーション

このサンプル シナリオは、多層アプリケーションをセキュリティで保護する必要がある規制対象業界に適用されます。 このシナリオでは、フロントエンド ASP.NET アプリケーションから、保護されたバックエンド Microsoft SQL Server クラスターに安全に接続します。

アプリケーションのサンプル シナリオには、手術室アプリケーションの実行、患者の予約とレコードの保存、または処方箋の差し替えと注文が含まれます。 これまでは、これらのシナリオのために、従来のオンプレミス アプリケーションとサービスを組織が維持する必要がありました。 これらの Windows Server アプリケーションを、セキュリティで保護されたスケーラブルな方法で Azure にデプロイすることにより、組織は自身のデプロイを最新化し、オンプレミスの運用コストと管理オーバーヘッドを減らすことができます。

## <a name="related-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* セキュリティで保護されたクラウド環境におけるアプリケーション デプロイの最新化。
* 従来のオンプレミス アプリケーションとサービス管理の軽減。
* 新しいアプリケーション プラットフォームでの医療と患者体験の向上。

## <a name="architecture"></a>アーキテクチャ

![規制対象業界向けの多層 Windows Server アプリケーションに関与する Azure コンポーネントのアーキテクチャ概要][architecture]

このシナリオでは、ASP.NET および Microsoft SQL Server を使用する規制対象業界の多層アプリケーションに対応できます。 このシナリオのデータ フローは次のとおりです。

1. ユーザーが Azure Application Gateway 経由で、フロントエンドの規制対象業界向け ASP.NET アプリケーションにアクセスします。
2. Application Gateway は、Azure 仮想マシン スケール セット内でトラフィックを VM インスタンスに分散します。
3. ASP.NET アプリケーションは、Azure ロード バランサーを使用して、バックエンド層の Microsoft SQL Server クラスターに接続します。 これらのバックエンド SQL Server インスタンスは別個の Azure 仮想ネットワークにあり、トラフィック フローを制限するネットワーク セキュリティ グループの規則によってセキュリティで保護されています。
4. ロード バランサーは、SQL Server のトラフィックを、別の仮想マシン スケール セット内の VM インスタンスに分散します。
5. Azure Blob Storage は、バックエンド層の SQL Server クラスター用のクラウド監視として機能します。  VNet 内からの接続は、Azure Storage の VNet サービス エンドポイントで有効にされます。

### <a name="components"></a>コンポーネント

* [Azure Application Gateway][appgateway-docs] は、アプリケーション対応のレイヤー 7 Web トラフィック ロード バランサーで、特定のルーティング規則に基づいてトラフィックを分散できます。 App Gateway で、SSL オフロードを処理し、Web サーバーのパフォーマンスを向上させることもできます。
* [Azure Virtual Network][vnet-docs] を使用すると、VM などのリソースが、他の Azure リソース、インターネット、およびオンプレミスのネットワークと安全に通信することができます。 仮想ネットワークにより、分離性、セグメント化、トラフィックのフィルター処理とルーティングが提供され、場所間の接続が可能になります。 このシナリオでは適切な NSG で結合されている 2 つ仮想ネットワークを使用して、[非武装地帯][dmz] (DMZ) と、アプリケーション コンポーネントの分離性が提供されます。 仮想ネットワーク ピアリングによって、2 つのネットワークが接続されます。
* [Azure 仮想マシン スケール セット][scaleset-docs]では、負荷分散が行われる同一の VM のグループを作成して管理できます。 需要または定義されたスケジュールに応じて、VM インスタンスの数を自動的に増減させることができます。 このシナリオでは 2 つの別個の仮想マシン スケール セットが使用されます。1 つはフロント エンドの ASP.NET アプリケーション インスタンス用、もう 1 つはバックエンドの SQL Server クラスター VM インスタンス用です。 PowerShell の Desired State Configuration (DSC) または Azure カスタム スクリプト拡張機能を使用すると、必要なソフトウェアおよび構成設定で VM インスタンスをプロビジョニングできます。
* [Azure ネットワーク セキュリティ グループ][nsg-docs]には、ソースまたはターゲット IP アドレス、ポート、およびプロトコルを基に、受信/送信ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。 このシナリオの仮想ネットワークは、アプリケーション コンポーネント間のトラフィック フローを制限するネットワーク セキュリティ グループ規則によって保護されています。
* [Azure Load Balancer][loadbalancer-docs] は、規則と正常性プローブに従って受信トラフィックを分散します。 ロード バランサーは、低遅延と高スループットを実現できるだけでなく、あらゆる TCP アプリケーションと UDP アプリケーションの数百万ものフローにスケールアップできます。 このシナリオでは内部ロード バランサーを使用して、フロントエンド アプリケーション層からバックエンド SQL Server クラスターへのトラフィックを分散します。
* [Azure Blob Storage][cloudwitness-docs] は、SQL Server クラスター用のクラウド監視の場所として機能します。 この監視は、クォーラムの決定に追加の投票が必要な、クラスター操作と意思決定で使用されます。 クラウド監視を使用すると、従来のファイル共有監視として機能する追加の VM が不要になります。

### <a name="alternatives"></a>代替手段

* インフラストラクチャにはオペレーティング システム に依存しているものがないため、*nix、Windows は、他のさまざまなオペレーティング システムに置き換えることができます。

* バックエンド データ ストアの代わりに、[Linux 用 SQL Server][sql-linux] を使用できます。

* データ ストアの代わりに、[Cosmos DB][cosmos] を使用することもできます。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

このシナリオの VM インスタンスは、可用性ゾーンにまたがってデプロイされます。 それぞれのゾーンは、独立した電源、冷却手段、ネットワークを備えた 1 つまたは複数のデータセンターで構成されています。 最低で 3 つのゾーンを、有効なすべてのリージョンで使用できます。 このようにゾーン間で VM インスタンスを分散させることにより、アプリケーション層に高可用性が実現します。 詳細については、[Azure の可用性ゾーンの概要][azureaz-docs]に関するページをご覧ください。

データベース層は、AlwaysOn 可用性グループを使用するように構成できます。 この SQL Server 構成により、クラスター内の 1 つのプライマリ データベースが、最大 8 つのセカンダリ データベースと共に構成されます。 プライマリ データベースで問題が発生した場合、クラスターは、セカンダリ データベースのいずれかにフェールオーバーします。これにより、アプリケーションを引き続き使用できます。 詳細については、[SQL Server 用の Always On 可用性グループの概要][sqlalwayson-docs]に関するページをご覧ください。

可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

このシナリオでは、フロントエンド コンポーネントとバックエンド コンポーネントに対して、仮想マシン スケール セットを使用します。 スケール セットにより、顧客の要求に応じて、または定義されているスケジュールに基づいて、フロントエンド アプリケーション層を実行する VM インスタンスの数を自動的にスケーリングできます。 詳細については、[仮想マシン スケール セットでの自動スケールの概要][vmssautoscale-docs]に関するページをご覧ください。

スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。

### <a name="security"></a>セキュリティ

フロントエンド アプリケーション層へのすべての仮想ネットワーク トラフィックが、ネットワーク セキュリティ グループによって保護されます。 フロントエンド アプリケーション層 VM インスタンスのみがバックエンド データベース層にアクセスできるように、規則によってトラフィックのフローが制限されます。 データベース層からの送信インターネット トラフィックは許可されません。 攻撃フットプリントを減らすために、ダイレクト リモート管理ポートは開かれていません。 詳細については、[Azure ネットワーク セキュリティ グループ][nsg-docs]に関するページをご覧ください。

Payment Card Industry Data Security Standards (PCI DSS 3.2) の展開のガイダンスについては、[コンプライアンス インフラストラクチャ][pci-dss]に関するページをご覧ください。 セキュリティで保護されたシナリオの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」を参照してください。

### <a name="resiliency"></a>回復性

このシナリオでは、可用性ゾーンおよび仮想マシン スケール セットと組み合わて、Azure Application Gateway とロード バランサーが使用されます。 これら 2 つのネットワーク コンポーネントは、接続されている VM インスタンスにトラフィックを分散します。また、コンポーネントには正常性プローブが含まれ、正常な状態の VM にのみトラフィックが分散されることが保証されます。 2 つの Application Gateway インスタンスは、アクティブ/パッシブ構成で構成され、ゾーン冗長ロード バランサーが使用されます。 トラフィックを中断し、エンドユーザーへのアクセスに影響を及ぼす可能性のある問題に対する回復性が、この構成によってネットワーク リソースとアプリケーションで実現します。

回復性に優れたシナリオの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

**前提条件。**

* 既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。
* バックエンド スケール セットに SQL Server クラスターをデプロイするには、Azure Active Directory (AD) Domain Services 内のドメインが必要です。

Azure Resource Manager テンプレートを使用して、このシナリオのコア インフラストラクチャをデプロイするには、次の手順を実行します。

1. **[Deploy to Azure]\(Azure にデプロイ\)** を選択します。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure portal でテンプレートのデプロイが開くまで待ってから、次の手順を実行します。
   * リソース グループの **[新規作成]** を選択し、テキスト ボックスに名前 (例: *myWindowsscenario*) を入力します。
   * **[場所]** ドロップダウン ボックスでリージョンを選択します。
   * 仮想マシン スケール セット インスタンスのユーザー名と安全なパスワードを入力します。
   * 使用条件を確認し、**[上記の使用条件に同意する]** をオンにします。
   * **[購入]** をクリックします。

デプロイが完了するまでに 15 から 20 分かかることがあります。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。  特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

ご自身のアプリケーションを実行するスケール セット VM インスタンスの数に基づいて、3 つのサンプル コスト プロファイルが用意されています。

* [Small][small-pricing]: この価格例は、2 つのフロントエンドおよび 2 つのバックエンド VM インスタンスに対応します。
* [Medium][medium-pricing]: この価格例は、20 つのフロントエンドおよび 5 つのバックエンド VM インスタンスに対応します。
* [Large][large-pricing]: この価格例は、100 つのフロントエンドおよび 10 つのバックエンド VM インスタンスに対応します。

## <a name="related-resources"></a>関連リソース

このシナリオでは、Microsoft SQL Server クラスターを実行するバックエンド仮想マシン スケール セットを使用しました。 アプリケーション データ用に、安全でスケーラブルなデータベース層として Azure Cosmos DB を使用することもできます。 [Azure 仮想ネットワーク サービス エンドポイント][vnetendpoint-docs]を使用すると、重要な Azure サービス リソースへのアクセスを仮想ネットワークのみに限定することができます。 このシナリオでは、VNet エンドポイントを使用することで、フロントエンド アプリケーション層と Cosmos DB の間のトラフィックをセキュリティで保護できます。 Cosmos DB の詳細については、[Azure Cosmos DB の概要][azurecosmosdb-docs]に関するページをご覧ください。

[SQL Server を使用汎用 n 層アプリケーションの詳しいリファレンス アーキテクチャ][ntiersql-ra]を確認することもできます。

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/regulated-multitier-app/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[azureaz-docs]: /azure/availability-zones/az-overview
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/ 
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability 
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[cosmos]: /azure/cosmos-db/
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017

[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93