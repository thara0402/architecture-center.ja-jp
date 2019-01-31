---
title: HA/DR 用に構築された多階層 Web アプリケーション
titleSuffix: Azure Example Scenarios
description: Azure 上で Azure 仮想マシン、可用性セット、可用性ゾーン、Azure Site Recovery、Azure Traffic Manager を使用して高可用性とディザスター リカバリー用にビルドされた多層 Web アプリケーションを作成します。
author: sujayt
ms.date: 11/16/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: product-team
social_image_url: /azure/architecture/example-scenario/infrastructure/media/arhitecture-disaster-recovery-multi-tier-app.png
ms.openlocfilehash: c60a2a07db578c447eb0682270c105b79e80e12b
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908513"
---
# <a name="multitier-web-application-built-for-high-availability-and-disaster-recovery-on-azure"></a>Azure における高可用性とディザスター リカバリー用にビルドされた多層 Web アプリケーション

このシナリオの例は、高可用性とディザスター リカバリーを実現できるようにビルドされた、回復性がある多層アプリケーションをデプロイする必要があるあらゆる業界に適用されます。 このシナリオでは、アプリケーションは 3 つのレイヤーで構成されます。

- Web 層:ユーザー インターフェイスを含む最上位レイヤー。 このレイヤーでは、ユーザーの操作が解析され、処理するためにアクションが次のレイヤーに渡されます。
- ビジネス層:ユーザーの操作が処理され、次のステップに関して論理的な意思決定が行われます。 このレイヤーは、Web 層とデータ層に接続されます。
- データ層:アプリケーション データが格納されます。 通常は、データベース、オブジェクト ストレージ、またはファイル ストレージが使用されます。

一般的なアプリケーション シナリオには、Windows または Linux で実行されているあらゆるミッション クリティカルなアプリケーションが含まれます。 SAP や SharePoint など、市販のアプリケーションのほか、カスタム基幹業務アプリケーションも該当します。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- SAP や SharePoint などの、回復性の高いアプリケーションのデプロイ
- 基幹業務アプリケーション用の事業継続とディザスター リカバリー計画の策定
- ディザスター リカバリーの構成、およびコンプライアンスを目的とした関連する訓練の実施

## <a name="architecture"></a>アーキテクチャ

このシナリオでは、ASP.NET と Microsoft SQL Server を使用する多層アプリケーションを示します。 [可用性ゾーンがサポートされる Azure リージョン](/azure/availability-zones/az-overview#regions-that-support-availability-zones)で、ご自身の仮想マシン (VM) を、可用性ゾーンにまたがる 1 つのソース リージョンにデプロイし、その VM をディザスター リカバリーに使用されるターゲット リージョンにレプリケートできます。 可用性ゾーンがサポートされていない Azure リージョンでは、ご自身の VM を可用性セット内にデプロイし、その VM をターゲット リージョンにレプリケートできます。

![高い回復性がある多層 Web アプリケーションのアーキテクチャの概要][architecture]

- ゾーンがサポートされているリージョン内で、2 つの可用性ゾーンにまたがる各階層に VM を分散します。 他のリージョンでは、1 つの可用性セット内の各階層に VM をデプロイします。
- データベース層は、AlwaysOn 可用性グループを使用するように構成できます。 この SQL Server 構成により、クラスター内の 1 つのプライマリ データベースが、最大 8 つのセカンダリ データベースと共に構成されます。 プライマリ データベースで問題が発生した場合、クラスターは、セカンダリ データベースのいずれかにフェールオーバーします。これにより、アプリケーションを引き続きご利用いただけます。 詳細については、[SQL Server 用の Always On 可用性グループの概要][docs-sql-always-on]に関するページをご覧ください。
- ディザスター リカバリーのシナリオでは、ディザスター リカバリーに使用されるターゲット リージョンへの SQL Always On 非同期ネイティブ レプリケーションを構成できます。 データ変更率が、Azure Site Recovery でサポートされている制限内の場合は、ターゲット リージョンへの Azure Site Recovery レプリケーションを構成することもできます。
- ユーザーは、Traffic Manager エンドポイントを使用して、フロントエンド ASP.NET Web 層にアクセスします。
- Traffic Manager により、トラフィックが、プライマリ ソース リージョンのプライマリ パブリック IP エンドポイントにリダイレクトされます。
- パブリック IP により、Web 層の VM インスタンスのいずれかへの呼び出しが、パブリック ロード バランサーを介してリダイレクトされます。 Web 層の VM インスタンスがすべて、1 つのサブネットに含まれています。
- Web 層の VM から、各呼び出しが、処理のために内部ロード バランサーを介して、ビジネス層の VM インスタンスのいずれかにルーティングされます。 ビジネス層の VM がすべて、別のサブネットに含まれています。
- 操作はビジネス層で処理され、ASP.NET アプリケーションは、Azure の内部ロード バランサーを介して、バックエンド層の Microsoft SQL Server クラスターに接続されます。 これらのバックエンド SQL Server インスタンスは、別のサブネットに含まれています。
- Traffic Manager のセカンダリ エンドポイントは、ディザスター リカバリーに使用されるターゲット リージョンでパブリック IP として構成されます。
- プライマリ リージョンで中断が発生した場合は、Azure Site Recovery のフェールオーバーを呼び出します。これにより、アプリケーションは、ターゲット リージョンでアクティブになります。
- Traffic Manager のエンドポイントにより、クライアント トラフィックは、ターゲット リージョンのパブリック IP に自動的にリダイレクトされます。

### <a name="components"></a>コンポーネント

- [可用性セット][docs-availability-sets]を使うことで、Azure にデプロイする VM は、クラスター内で切り離された複数のハードウェア ノードに確実に分散されます。 Azure 内でハードウェアまたはソフトウェアの障害が発生した場合、影響を受けるのは VM のサブセットのみで、ソリューション全体は引き続き利用および運用可能です。
- [可用性ゾーン][docs-availability-zones]により、ご自身のアプリケーションとデータが、データセンターの障害から保護されます。 可用性ゾーンは、Azure リージョン内の切り離された物理的な場所です。 それぞれのゾーンは、独立した電源、冷却手段、ネットワークを備えた 1 つまたは複数のデータセンターで構成されています。
- [Azure Site Recovery (ASR)][docs-azure-site-recovery] を使用すると、事業継続とディザスター リカバリーのニーズに応じて、VM を別の Azure リージョンにレプリケートできます。 コンプライアンスのニーズを満たしていることを確認するために、定期的なディザスター リカバリー訓練を実施できます。 ソース リージョンで障害が発生した場合、アプリケーションを回復できるように、選択したリージョンに指定した設定で VM がレプリケートされます。
- [Azure Traffic Manager][docs-traffic-manager] は、世界中の Azure リージョン間でサービスへのトラフィックを最適に分散しながら、高可用性と応答性を実現する DNS ベースのトラフィック ロード バランサーです。
- [Azure Load Balancer][docs-load-balancer] により、定義された規則と正常性プローブに従って受信トラフィックが分散されます。 ロード バランサーは、低遅延と高スループットを実現し、あらゆる TCP アプリケーションと UDP アプリケーションの数百万ものフローにスケールアップします。 このシナリオでは、パブリック ロード バランサーを使って、着信クライアント トラフィックが Web 層に分散されます。 このシナリオでは、内部ロード バランサーを使って、ビジネス層からバックエンド SQL Server クラスターにトラフィックが分散されます。

### <a name="alternatives"></a>代替手段

- このインフラストラクチャにはオペレーティング システムに依存している要素がないため、Windows は、他のオペレーティング システムで置き換えることができます。
- バックエンド データ ストアの代わりに、[Linux 用 SQL Server][docs-sql-server-linux] を使用できます。
- データベースは、使用可能な任意の標準データベース アプリケーションで置き換えることができます。

## <a name="other-considerations"></a>その他の考慮事項

### <a name="scalability"></a>スケーラビリティ

ご自身のスケーリング要件に基づいて、各階層で VM を追加または削除できます。 このシナリオではロード バランサーが使用されているため、アプリケーションのアップタイムに影響を与えずに、VM を階層に追加できます。

スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。

### <a name="security"></a>セキュリティ

フロントエンド アプリケーション層へのすべての仮想ネットワーク トラフィックが、ネットワーク セキュリティ グループによって保護されます。 フロントエンド アプリケーション層 VM インスタンスのみがバックエンド データベース層にアクセスできるように、規則によってトラフィックのフローが制限されます。 ビジネス層またはデータベース層からのアウトバウンド インターネット トラフィックは許可されません。 攻撃フットプリントを減らすために、ダイレクト リモート管理ポートは開かれていません。 詳細については、[Azure ネットワーク セキュリティ グループ][docs-nsg]に関するページをご覧ください。

セキュリティで保護されたシナリオの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」を参照してください。

## <a name="pricing"></a>価格

Azure Site Recovery を使用して Azure VM のディザスター リカバリーを構成すると、次の料金が継続的に発生します。

- VM あたりの Azure Site Recovery ライセンス コスト。
- データ変更をソース VM ディスクから別の Azure リージョンにレプリケートするためのネットワーク エグレス コスト。 Azure Site Recovery では組み込みの圧縮機能によって、データ転送要件が約 50% 削減されます。
- 復旧サイトでのストレージ コスト。 これは、通常、ソース リージョンのストレージ コストに、復旧ポイントをスナップショットとして維持するための追加ストレージのコストを足したものです。

6 台の仮想マシンを使用して 3 層アプリケーションのディザスター リカバリーを構成するための[サンプル コスト計算ツール][calculator]をご用意いたしました。 すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースの場合、価格がどのように異なるかを確認するには、該当する変数を変更して、コストを見積もります。

<!-- links -->
[architecture]: ./media/arhitecture-disaster-recovery-multi-tier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-availability-zones]: /azure/availability-zones/az-overview
[docs-load-balancer]: /azure/load-balancer/load-balancer-overview
[docs-nsg]: /azure/virtual-network/security-overview
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-sql-always-on]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-vnet]: /azure/virtual-network/virtual-networks-overview
[docs-sql-server-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[docs-traffic-manager]: /azure/traffic-manager/
[docs-azure-site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[docs-availability-sets]: /azure/virtual-machines/windows/manage-availability/
[calculator]: https://azure.com/e/6835332265044d6d931d68c917979e6d/
