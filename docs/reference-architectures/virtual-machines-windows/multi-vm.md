---
title: "スケーラビリティと可用性のために Azure で負荷分散された VM を実行する"
description: "スケーラビリティと可用性のために Azure で複数の Windows VM を実行する方法。"
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: c9b1e52044d38348ecf1bd29cb24b3c20d1d6a45
ms.sourcegitcommit: 115db7ee008a0b1f2b0be50a26471050742ddb04
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/17/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>スケーラビリティと可用性のために負荷分散された VM を実行する

この参照アーキテクチャは、可用性とスケーラビリティを向上させるために、ロード バランサーの背後にあるスケール セット内で複数の Windows 仮想マシン (VM) を実行するための一連の実証済みの手法を示します。 このアーキテクチャは任意のステートレス ワークロード (Web サーバーなど) に使用でき、N 層アプリケーションをデプロイするための基礎となります。 [**以下のソリューションをデプロイします**。](#deploy-the-solution)

![[0]][0]

*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、[単一の VM 参照アーキテクチャ][single-vm]に基づいて構築されています。 それらの推奨事項は、このアーキテクチャにも適用されます。

このアーキテクチャでは、ワークロードが複数の VM インスタンスにわたって分散されます。 単一のパブリック IP アドレスが存在し、ロード バランサーを使用してインターネット トラフィックが VM に分散されます。 このアーキテクチャは、ステートレスな Web アプリケーションなどの、単一の層のアプリケーションに使用できます。

アーキテクチャには次のコンポーネントがあります。

* **リソース グループ。** [リソース グループ][resource-manager-overview]は、リソースをグループ化して、有効期間、所有者、またはその他の条件別に管理できるようにするために使用されます。
* **仮想ネットワーク (VNet) とサブネット。** どの Azure VM も、複数のサブネットにセグメント化できる VNet にデプロイされます。
* **Azure Load Balancer**。 [ロード バランサー][load-balancer]は、受信インターネット要求を各 VM インスタンスに分散します。 
* **パブリック IP アドレス**。 パブリック IP アドレスは、ロード バランサーがインターネット トラフィックを受信するために必要です。
* **VM スケール セット**。 [VM スケール セット][vm-scaleset]は、ワークロードをホストするために使用される同一の VM のセットです。 スケール セットにより、VM の数を手動でスケールインまたはスケールアウトしたり、定義済みの規則に基づいて自動的に設定したりできるようになります。
* **可用性セット**。 [可用性セット][availability-set]には VM が含まれ、VM がより高度な[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。 より高度な SLA を適用するためには、可用性セットには少なくとも 2 つの VM が含まれる必要があります。 可用性セットはスケール セットの中で暗黙的です。 スケール セットの外で VM を作成する場合は、可用性セットを個別に作成する必要があります。
* **Managed Disks**。 Azure Managed Disks では、VM ディスクの仮想ハード ディスク (VHD) ファイルが管理されます。 
* **ストレージ**。 Azure Storage アカウントを作成して、VM の診断ログを保持します。

## <a name="recommendations"></a>Recommendations

ユーザーの要件が、ここで説明されているアーキテクチャと完全には整合しない可能性があります。 これらの推奨事項は原案として使用してください。 

### <a name="availability-and-scalability-recommendations"></a>可用性とスケーラビリティの推奨事項

可用性とスケーラビリティのための 1 つのオプションは、[仮想マシン スケール セット][vmss]を使用することです。 VM スケール セットを使用して、同一の VM のセットをデプロイして管理できます。 スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。 VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。 VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。

既定では、スケール セットでは "オーバープロビジョニング" が使用されます。これは、スケール セットが要求するよりも多くの VM を最初にプロビジョニングし、次に余分な VM を削除するという意味です。 これにより、VM をプロビジョニングするときの全体的な成功率が向上します。 [管理ディスク](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)を使用していない場合は、ストレージ アカウントあたりの過剰プロビジョニングが有効な VM が 20 を超えず、また過剰プロビジョニングが無効な VM が 40 を超えないようにすることをお勧めします。

スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。

- 拡張機能を使用してプロビジョニングされた後に VM を構成します。 この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。

- カスタム ディスク イメージと共に [Managed Disk](/azure/storage/storage-managed-disks-overview) をデプロイします。 このオプションの方が早くデプロイできる場合があります。 ただし、イメージを最新の状態に保つ必要があります。

詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。

> [!TIP]
> 自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。

スケール セットを使用しない場合は、少なくとも可用性セットの使用を検討してください。 [Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セット内に少なくとも 2 つの VM を作成します。 Azure Load Balancer でも、負荷分散された VM が同じ可用性セットに属する必要があります。

各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。 サポート リクエストを提出することで、制限値を上げることができます。 詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。

### <a name="network-recommendations"></a>ネットワークの推奨事項

VM を同じサブネット内にデプロイします。 VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。 クライアントは、ロード バランサーのパブリック IP アドレスを使用して接続します。

ロード バランサーの背後にある VM にログインする必要がある場合は、ログインできるパブリック IP アドレスを使用して、単一の VM をジャンプボックス (要塞ホストとも呼ばれます) として追加することを検討してください。 ジャンプボックスからロード バランサーの背後にある VM にログインします。 あるいは、ロード バランサーの受信ネットワーク アドレス変換 (NAT) 規則を構成できます。 ただし、N 層ワークロードまたは複数のワークロードをホストしている場合は、ジャンプボックスを使用する方がより優れたソリューションです。

### <a name="load-balancer-recommendations"></a>ロード バランサーの推奨事項

可用性セット内のすべての VM をロード バランサーのバックエンド アドレス プールに追加します。

ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。 たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。 クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load-balancer-hashing]を使用して、バックエンド IP アドレスを選択します。 この方法で、クライアント要求がすべての VM に配布されます。

特定の VM にトラフィックをルーティングするには、NAT 規則を使用します。 たとえば、VM への RDP を有効にするには、VM ごとに個別の NAT 規則を作成します。 各規則では、RDP の既定のポートであるポート 3389 に個別のポート番号を割り当てる必要があります。 たとえば、"VM1" にはポート 50001 を、"VM2" にはポート 50002 を使用するなどです。 VM 上の NIC に NAT 規則を割り当てます。

### <a name="storage-account-recommendations"></a>ストレージ アカウントの推奨事項

[管理ディスク](/azure/storage/storage-managed-disks-overview)を [Premium ストレージ][premium]と共に使用することをお勧めします。 管理ディスクでは、ストレージ アカウントは必要ありません。 単にディスクのサイズと種類を指定するだけで、可用性の高いリソースとしてデプロイされます。

非管理対象ディスクを使用している場合は、ストレージ アカウントの 1 秒あたりの入出力操作 [(IOPS) の制限][vm-disk-limits]に達しないようにするために、仮想ハード ディスク (VHD) を保持するための個別の Azure ストレージ アカウントを VM ごとに作成します。

診断ログ用のストレージ アカウントを 1 つ作成します。 このストレージ アカウントは、すべての VM で共有できます。 これには、標準ディスクを使ったアンマネージ ストレージ アカウントを指定できます。

## <a name="availability-considerations"></a>可用性に関する考慮事項

可用性セットは、計画済みおよび計画外の両方のメンテナンス イベントに対するアプリケーションの復元性を高めます。

* *計画済みメンテナンス*は、Microsoft が基本のプラットフォームを更新したときに行われ、VM の再起動を伴う場合もあります。 Azure は、可用性セット内の VM がすべて同時に再起動されないことを保証します。 他の VM が再起動されている間、少なくとも 1 つの VM は継続的に実行されます。
* *計画外のメンテナンス*は、ハードウェア障害が発生したときに行われます。 Azure では、可用性セット内の VM が複数のサーバー ラックにプロビジョニングされることを保証します。 これにより、ハードウェア障害、ネットワーク停止、電源の瞬停などの影響が小さくなります。

詳細については、「 [Virtual Machines の可用性管理][availability-set]」を参照してください。 また、「[How Do I Configure an Availability Set to Scale VMs (VM をスケール調整するために可用性セットを構成する方法)][availability-set-ch9]」のビデオにも可用性セットの適切な概要が示されています。

> [!WARNING]
> VM をプロビジョニングするときに、必ず可用性セットを構成してください。 現時点では、VM がプロビジョニングされた後に、Resource Manager VM を可用性セットに追加する方法はありません。

ロード バランサーは、[正常性プローブ][health-probes]を使用して VM インスタンスの可用性を監視します。 タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。 ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。

ロード バランサーの正常性プローブには、次のような推奨事項があります。

* プローブでは、HTTP または TCP のいずれかをテストできます。 VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。 それ以外の場合は、TCP プローブを作成します。
* HTTP プローブでは、HTTP エンドポイントへのパスを指定します。 プローブでは、このパスからの HTTP 200 の応答をチェックします。 これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。 エンドポイントでは、匿名の HTTP 要求を許可する必要があります。
* このプローブは、[既知の IP アドレス][health-probe-ip]である 168.63.129.16 から送信されます。 この IP アドレスとの間のトラフィックを、どのファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) 規則でもブロックしていないことを確認してください。
* [正常性プローブ ログ][health-probe-log]を使用して、正常性プローブの状態を表示します。 各ロード バランサーに対して Azure Portal のログ記録を有効にします。 ログは Azure Blob Storage に書き込まれます。 ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

複数の VM の場合、信頼性を高め、反復可能にするために、プロセスを自動化することが重要です。 [Azure Automation][azure-automation] を使用して、デプロイ、OS パッチ、およびその他のタスクを自動化できます。 [Azure Automation][azure-automation] は、このために使用できる PowerShell に基づいたオートメーション サービスです。 オートメーション スクリプトの例は、[Runbook ギャラリー][runbook-gallery]から入手できます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

仮想ネットワークは、Azure のトラフィックの分離境界です。 ある VNet 内の VM が別の VNet 内の VM と直接通信することはできません。 トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。 詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。

インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。 しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのデプロイについては、[GitHub][github-folder] をご覧ください。 以下がデプロイされます。

  * VM を含む、**Web** という名前の 1 つのサブネットを持つ仮想ネットワーク。
  * Windows Server 2016 Datacenter Edition の最新バージョンを実行する VM を含んだ VM スケール セット。 自動スケールが有効になっています。
  * VM スケール セットの前に配置されているロード バランサー。
  * VM スケール セットへの HTTP トラフィックを許可する受信規則を備えた NSG。

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。 CLI のインストール手順については、「[Azure CLI 2.0 のインストール][azure-cli-2]」をご覧ください。

3. [Azure の構成要素][azbb] npm パッケージをインストールします。

4. コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、以下のコマンドの 1 つを使用して Azure アカウントにログインし、プロンプトに従います。

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>azbb を使用したソリューションのデプロイ

単一の VM ワークロードのサンプルをデプロイするには、次の手順に従います。

1. 上記の前提条件でダウンロードしたリポジトリの `virtual-machines\multi-vm\parameters\windows` フォルダーに移動します。

2. `multi-vm-v2.json` ファイルを開き、次に示すような引用符の間にユーザー名とパスワードを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. 次に示すように、`azbb` を実行して VM をデプロイします。

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "2 つの VM とロード バランサーを備えた可用性セットを構成する Azure 上のマルチ VM ソリューションのアーキテクチャ"
