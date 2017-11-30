---
title: "スケーラビリティと可用性のために Azure で負荷分散された VM を実行する"
description: "スケーラビリティと可用性のために Azure で複数の Windows VM を実行する方法。"
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: d38cfb41255c547f1f1e87ef289c7a79033df778
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>スケーラビリティと可用性のために負荷分散された VM を実行する

この参照アーキテクチャでは、可用性とスケーラビリティを向上させるために、ロード バランサーの背後にあるスケール セット内でいくつかの Windows 仮想マシン (VM) を実行する、一連の実証済みのプラクティスが示されます。 このアーキテクチャは Web サーバーなどの任意のステートレス ワークロードで使用でき、n 層アプリケーションをデプロイするための構成要素となります。 [**このソリューションをデプロイします**。](#deploy-the-solution)

![[0]][0]

*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、「[Azure での Windows VM の実行][single vm]」に示されるアーキテクチャ上に構築されます。 そこにある推奨事項は、このアーキテクチャにも適用されます。

このアーキテクチャでは、ワークロードはいくつかの VM インスタンスに分散されます。 単一のパブリック IP アドレスが存在し、ロード バランサーを使用してインターネット トラフィックが VM に分散されます。 このアーキテクチャは、ステートレスな Web アプリケーションなどの、単一の層のアプリケーションに使用できます。

アーキテクチャには次のコンポーネントがあります。

* **リソース グループ。** ["*リソース グループ*"][resource-manager-overview] を使用してリソースをグループ化し、有効期間、所有者、その他の基準で管理できるようにします。
* **仮想ネットワーク (VNet) とサブネット。** Azure 上のすべての VM は、さらに複数のサブネットに分かれる VNet にデプロイされます。
* **Azure Load Balancer**。 [ロード バランサー]は着信インターネット要求を VM インスタンスに分散します。 
* **パブリック IP アドレス**。 パブリック IP アドレスは、ロード バランサーがインターネット トラフィックを受信するために必要です。
* **VM スケール セット**。 [VM スケール セット][vm-scaleset]は、ワークロードをホストするために使用される同一の VM のセットです。 スケール セットでは、VM の数を、手動でスケール インまたはスケール アウトしたり、定義済みの規則に基づいて設定したりできます。
* **可用性セット**。 [可用性セット][availability set]には VM が含まれ、VM がより高度な[サービス レベル アグリーメント (SLA)][vm-sla] に対応できるようになります。 より高度な SLA を適用するためには、可用性セットには少なくとも 2 つの VM が含まれる必要があります。 可用性セットはスケール セットの中で暗黙的です。 スケール セットの外で VM を作成する場合は、可用性セットを個別に作成する必要があります。
* **Managed Disks**。 Azure Managed Disks では、VM ディスクの仮想ハード ディスク (VHD) ファイルが管理されます。 
* **ストレージ**。 Azure ストレージ アカウントを作成して、VM の診断ログを保持します。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 これらの推奨事項は原案として使用してください。 

### <a name="availability-and-scalability-recommendations"></a>可用性とスケーラビリティの推奨事項

可用性とスケーラビリティのための 1 つのオプションは、[仮想マシン スケール セット][vmss]を使用することです。 VM スケール セットを使用して、同一の VM のセットをデプロイして管理できます。 スケール セットでは、パフォーマンス メトリックに基づく自動スケールがサポートされます。 VM の負荷が増えると、追加の VM が自動的にロード バランサーに追加されます。 VM をすばやくスケール アウトしたり、自動スケールしたりする必要がある場合は、スケール セットを検討してください。

既定では、スケール セットでは "オーバープロビジョニング" が使用されます。これは、スケール セットが要求するよりも多くの VM を最初にプロビジョニングし、次に余分な VM を削除するという意味です。 これにより、VM をプロビジョニングするときの全体的な成功率が向上します。 [管理ディスク](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)を使用しない場合、ストレージ アカウントごとの VM の数は、オーバープロビジョニングが有効なときは 20 以下に、無効なときは 40 以下にすることをお勧めします。

スケール セットにデプロイされる VM を構成するには、2 つの基本的な方法があります。

- 拡張機能を使用してプロビジョニングされた後に VM を構成します。 この方法では、新しい VM インスタンスは、拡張機能なしの VM よりも起動に時間がかかる場合があります。

- カスタム ディスク イメージと共に [Managed Disk](/azure/storage/storage-managed-disks-overview) をデプロイします。 このオプションの方が早くデプロイできる場合があります。 ただし、イメージを最新の状態に保つ必要があります。

詳しい考慮事項については、「[スケール セットの設計上の考慮事項][vmss-design]」を参照してください。

> [!TIP]
> 自動スケールのソリューションを使用する場合は、十分前もって実稼働レベルのワークロードでテストしてください。

スケール セットを使用しない場合は、少なくとも可用性セットの使用を検討してください。 [Azure VM の可用性 SLA][vm-sla] をサポートするために、可用性セットには少なくとも 2 つの VM を作成してください。 Azure Load Balancer でも、負荷分散された VM が同じ可用性セットに属する必要があります。

各 Azure サブスクリプションには、リージョンごとの VM の最大数などの、既定の制限があります。 サポート リクエストを提出することで、制限値を上げることができます。 詳細については、「[Azure サブスクリプションとサービスの制限、クォータ、制約][subscription-limits]」をご覧ください。

### <a name="network-recommendations"></a>ネットワークの推奨事項

VM を同じサブネット内に配置します。 VM を直接インターネットに公開せず、代わりに各 VM にプライベート IP アドレスを付与します。 クライアントは、ロード バランサーのパブリック IP アドレスを使用して接続します。

ロード バランサーの背後にある VM にログインする必要がある場合は、ログイン可能なパブリック IP アドレスを持つ要塞ホスト/ジャンプボックスとして 1 つの VM を追加することを検討してください。 ジャンプボックスからロード バランサーの背後にある VM にログインします。 または、ロード バランサーの受信 NAT 規則を同じ目的のために構成します。 ただし、n 層ワークロードか複数のワークロードをホストしている場合は、ジャンプボックスを設けるほうがソリューションとして優れています。

### <a name="load-balancer-recommendations"></a>ロード バランサーの推奨事項

可用性セット内のすべての VM をロード バランサーのバックエンド アドレス プールに追加します。

ロード バランサー規則を定義して、ネットワーク トラフィックを VM に転送します。 たとえば、HTTP トラフィックを有効にするには、フロントエンド構成からのポート 80 をバックエンド アドレス プールのポート 80 にマッピングする規則を作成します。 クライアントがポート 80 に HTTP 要求を送信するときに、ロード バランサーは、発信元 IP アドレスを含む[ハッシュ アルゴリズム][load balancer hashing]を使用して、バックエンド IP アドレスを選択します。 この方法で、クライアント要求がすべての VM に配布されます。

特定の VM にトラフィックをルーティングするには、NAT 規則を使用します。 たとえば、VM への RDP を有効にするには、VM ごとに個別の NAT 規則を作成します。 各規則では、RDP の既定のポートであるポート 3389 に個別のポート番号を割り当てる必要があります。 たとえば、"VM1" にはポート 50001 を、"VM2" にはポート 50002 を使用するなどです。 VM 上の NIC に NAT 規則を割り当てます。

### <a name="storage-account-recommendations"></a>ストレージ アカウントの推奨事項

ストレージ アカウントの 1 秒あたりの入力/出力処理 [(IOPS) 制限][vm-disk-limits]に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。

[管理ディスク](/azure/storage/storage-managed-disks-overview)を [Premium ストレージ][premium]と共に使用することをお勧めします。 管理ディスクでは、ストレージ アカウントは必要ありません。 ディスクのサイズと種類を指定するだけで、可用性の高い方法でデプロイされます。

診断ログ用のストレージ アカウントを 1 つ作成します。 このストレージ アカウントは、すべての VM で共有できます。 これには、標準ディスクを使ったアンマネージ ストレージ アカウントを指定できます。

## <a name="availability-considerations"></a>可用性に関する考慮事項

可用性セットは、計画済みおよび計画外の両方のメンテナンス イベントに対するアプリケーションの復元性を高めます。

* *計画済みメンテナンス*は、Microsoft が基本のプラットフォームを更新したときに行われ、VM の再起動を伴う場合もあります。 Azure は、可用性セット内の VM がすべて同時に再起動されないことを保証します。 他の VM が再起動されている間、少なくとも 1 つの VM は継続的に実行されます。
* *計画外のメンテナンス*は、ハードウェア障害が発生したときに行われます。 Azure では、可用性セット内の VM が複数のサーバー ラックにプロビジョニングされることを保証します。 これにより、ハードウェア障害、ネットワーク停止、電源の瞬停などの影響が小さくなります。

詳細については、「 [Virtual Machines の可用性管理][availability set]」を参照してください。 「[How Do I Configure an Availability Set to Scale VMs (VM をスケールする可用性セットの構成方法)][availability set ch9]」のビデオにも、可用性セットのわかりやすい概要が含まれています。

> [!WARNING]
> VM をプロビジョニングするときに、必ず可用性セットを構成してください。 現時点では、VM がプロビジョニングされた後に、Resource Manager VM を可用性セットに追加する方法はありません。

ロード バランサーは、[正常性プローブ]を使って VM インスタンスの可用性を監視します。 タイムアウト期間内にプローブがインスタンスに到達できなかった場合、ロード バランサーはその VM へのトラフィックの送信を停止します。 ただし、ロード バランサーは引き続きプローブを行い、VM が再び使用可能になると、ロード バランサーはその VM へのトラフィックの送信を再開します。

ロード バランサーの正常性プローブには、次のような推奨事項があります。

* プローブでは、HTTP または TCP のいずれかをテストできます。 VM で HTTP サーバーを実行している場合、HTTP プローブを作成します。 それ以外の場合は、TCP プローブを作成します。
* HTTP プローブでは、HTTP エンドポイントへのパスを指定します。 プローブでは、このパスからの HTTP 200 の応答をチェックします。 これには、ルート パス ("/")、またはアプリケーションの正常性をチェックするためのカスタム ロジックを実装した正常性監視エンドポイントを指定できます。 エンドポイントでは、匿名の HTTP 要求を許可する必要があります。
* プローブは、[既知の][health-probe-ip] IP アドレスである 168.63.129.16 から送信されます。 任意のファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) の規則で、この IP を送受信するトラフィックをブロックしないようにしてください。
* [正常性プローブ ログ][health probe log]を使用して、正常性プローブの状態を表示します。 各ロード バランサーに対して Azure Portal のログ記録を有効にします。 ログは Azure Blob Storage に書き込まれます。 ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない バック エンドの VM 数が表示されます。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

複数の VM の場合、信頼性を高め、反復可能にするために、プロセスを自動化することが重要です。 [Azure Automation][azure-automation] を使用して、デプロイ、OS パッチ、およびその他のタスクを自動化できます。 [Azure Automation][azure-automation] は、このために使用できる、Windows Powershell に基づいたオートメーション サービスです。 オートメーションのスクリプト例は、TechNet の [Runbook ギャラリー]から利用可能です。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

仮想ネットワークは、Azure のトラフィックの分離境界です。 VNet 内の VM は、別の VNet 内の VM とは直接通信できません。 トラフィックを制限する[ネットワーク セキュリティ グループ][nsg] (NSG) を作成していないかぎり、同じ VNet 内の VM 同士は通信可能です。 詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][network-security]」をご覧ください。

インターネット トラフィックを受信する場合、ロード バランサーの規則でバック エンドに到達できるトラフィックを定義しています。 しかし、ロード バランサーの規則では IP の安全な一覧をサポートしていないため、特定のパブリック IP アドレスを安全な一覧に追加したい場合は、NSG をサブネットに追加してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのデプロイについては、[GitHub][github-folder] をご覧ください。 次のものがデプロイされます。

  * VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。
  * Windows Server 2016 Datacenter Edition の最新バージョンを実行する VM を含んだ VM スケール セット。 自動スケールが有効になっています。
  * VM スケール セットの前に配置されているロード バランサー。
  * VM スケール セットへの HTTP トラフィックを許可する受信規則を備えた NSG。

### <a name="prerequisites"></a>前提条件

参照アーキテクチャをご自身のサブスクリプションにデプロイする前に、次の手順を実行する必要があります。

1. [AzureCAT 参照アーキテクチャ][ref-arch-repo] GitHub リポジトリに ZIP ファイルを複製、フォーク、またはダウンロードします。

2. Azure CLI 2.0 がコンピューターにインストールされていることを確認してください。 CLI をインストールするには、「[Azure CLI 2.0 のインストール][azure-cli-2]」の手順に従ってください。

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

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[正常性プローブ]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[ロード バランサー]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbook ギャラリー]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "2 つの VM とロード バランサーを備えた可用性セットを構成する Azure 上のマルチ VM ソリューションのアーキテクチャ"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Windows)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm