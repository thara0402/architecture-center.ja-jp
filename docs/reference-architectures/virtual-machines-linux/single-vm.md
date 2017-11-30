---
title: "Azure での Linux VM の実行"
description: "拡張性、回復性、管理容易性、セキュリティに注目しながら、Azure で Linux VM を実行する方法。"
author: telmosampaio
ms.date: 09/06/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f38c53db5df1681a1abc926df9f0b7f929ceb76b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-linux-vm-on-azure"></a>Azure での Linux VM の実行

この参照アーキテクチャでは、Azure で Linux 仮想マシン (VM) を実行するための一連の実証済みのプラクティスが示されます。 ネットワークおよびストレージ コンポーネントと共に VM をプロビジョニングするための推奨事項が含まれます。 このアーキテクチャは単一インスタンスを実行するために使用でき、n 層アプリケーションなどのさらに複雑なアーキテクチャの基礎となります。 [**このソリューションをデプロイします**。](#deploy-the-solution)

![[0]][0]

*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*

## <a name="architecture"></a>アーキテクチャ

Azure で VM をプロビジョニングする際は、VM 自体のみよりも、多くの変動的な部分があります。 コンピューティング、ネットワーク、およびストレージの要素を考慮する必要があります。

* **リソース グループ。** [*リソース グループ*][resource-manager-overview]は、関連リソースを保持するコンテナーです。 通常は、ユーザーが有効期限に基づいてソリューション内のリソースごとにリソース グループを作成し、リソースを管理します。 単一の VM ワークロードでは、すべてのリソースに対して単一のリソース グループを作成する場合があります。
* **VM**。 Azure は、CentOS、Debian、Red Hat Enterprise、Ubuntu、FreeBSD など、現在普及しているさまざまな Linux ディストリビューションに対応します。 詳細については、「[Azure と Linux][azure-linux]」を参照してください。 VM は、発行されたイメージの一覧、または Azure BLOB ストレージにアップロードした仮想ハード ディスク (VHD) ファイルからプロビジョニングできます。
* **OS ディスク。** OS ディスクは、[Azure Storage][azure-storage] に格納されている VHD です。 これは、ホスト コンピューターがダウンした場合でも VM が保持されることを意味します。 OS ディスクは `/dev/sda1`です。
* **一時ディスク。** VM は一時ディスクを使用して作成されます。 このディスクは、ホスト コンピューターの物理ドライブ上に格納されます。 Azure Storage には "*保存されない*" ため、再起動中や他の VM ライフサイクル イベント時に削除される可能性があります。 ページ ファイルやスワップ ファイルなどの一時的なデータにのみ、このディスクを使用してください。 一時ディスクは `/dev/sdb1` で、`/mnt/resource` または `/mnt` でマウントされます。
* **データ ディスク。** [データ ディスク][data-disk]は、アプリケーション データ用の永続的な VHD です。 データ ディスクは、OS ディスクと同様に、Azure Storage に格納されます。
* **仮想ネットワーク (VNet) とサブネット。** Azure 上のすべての VM は、さらに複数のサブネットに分かれる VNet にデプロイされます。
* **パブリック IP アドレス。** パブリック IP アドレスは、SSH 経由などで VM と通信するために必要です。
* **ネットワーク インターフェイス (NIC)**。 NIC を使用すると、VM は仮想ネットワークと通信できます。
* **ネットワーク セキュリティ グループ (NSG)**。 [NSG][nsg] は、サブネットへのネットワーク トラフィックを許可または拒否するために使用します。 NSG は、個々 の NIC またはサブネットに関連付けることができます。 NSG をサブネットに関連付けると、そのサブネット内のすべての VM に NSG ルールが適用されます。
* **[診断]。** 診断ログは、VM の管理とトラブルシューティングにとって非常に重要です。

## <a name="recommendations"></a>Recommendations

このアーキテクチャでは、Azure で Linux VM を実行するためのベースラインの推奨事項が示されます。 ただし、単一障害点ができてしまうため、ミッション クリティカルなワークロードに単一の VM を使用することは推奨されません。 可用性を高めるには、複数の VM を[可用性セット][availability-set]にデプロイします。 詳細については、[Azure での複数の VM の実行][multi-vm]に関するページをご覧ください。 

### <a name="vm-recommendations"></a>VM の推奨事項

Azure にはさまざまな仮想マシン サイズが用意されていますが、DS シリーズと GS シリーズをお勧めします。これらのマシン サイズでは [Premium Storage][premium-storage] がサポートされるためです。 ハイパフォーマンス コンピューティングなどの特殊なワークロードがない限り、これらのマシン サイズのいずれかを選択してください。 詳細については、[仮想マシンのサイズ][virtual-machine-sizes]に関するページをご覧ください。

既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。 次に、CPU、メモリ、およびディスクの 1 秒あたりの入力/出力操作 (IOPS) について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。 VM 用に複数の NIC が必要な場合は、NIC の最大数が [VM のサイズ][vm-size-tables]と関係することに注意してください。

VM および他のリソースをプロビジョニングする際は、リージョンを指定する必要があります。 一般的に、内部ユーザーや顧客に最も近いリージョンを選択します。 ただし、すべてのリージョンですべての VM サイズを利用できるとは限りません。 詳細については、[リージョン別のサービス][services-by-region]に関するページを参照してください。 特定のリージョンで利用できる VM サイズを一覧表示するには、次の Azure コマンド ライン インターフェイス (CLI) コマンドを実行します。

```
az vm list-sizes --location <location>
```

発行された VM イメージの選択については、「[Azure CLI を使用した Linux VM イメージの選択][select-vm-image]」を参照してください。

基本的な正常性メトリック、診断インフラストラクチャ ログ、[ブート診断][boot-diagnostics]などの監視と診断を有効にします。 VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。 詳細については、「[監視と診断の有効化][enable-monitoring]」を参照してください。  

### <a name="disk-and-storage-recommendations"></a>ディスクとストレージの推奨事項

最適なディスク I/O パフォーマンスを得るには、データがソリッド ステート ドライブ (SSD) に格納される [Premium Storage][premium-storage] をお勧めします。 コストは、プロビジョニングされたディスクのサイズに基づいて決まります。 また、IOPS とスループット (つまり、データ転送速度) もディスク サイズによって異なるため、ディスクをプロビジョニングする場合は、3 つの要素 (容量、IOPS、スループット) すべてを考慮してください。 

[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用することもお勧めします。 管理ディスクでは、ストレージ アカウントは必要ありません。 ディスクのサイズと種類を指定するだけで、可用性の高い方法でデプロイされます。

管理ディスクを使用しない場合は、ストレージ アカウントの IOPS 制限に達するのを回避するために、仮想ハード ディスク (VHD) を保持する Azure ストレージ アカウントを VM ごとに個別に作成します。

1 つ以上のデータ ディスクを追加します。 作成した VHD は、フォーマットされていません。 その VM にログインしてディスクをフォーマットしてください。 Linux のシェルでは、データ ディスクは `/dev/sdc`、`/dev/sdd` などのように表示されます。 `lsblk` を実行すると、ディスクなどのブロック デバイスの一覧を表示できます。 データ ディスクを使用するには、パーティションとファイル システムを作成し、ディスクをマウントします。 For example:

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

[管理ディスク](/azure/storage/storage-managed-disks-overview)を使用せず、データ ディスクの数が多い場合は、ストレージ アカウントの合計 I/O 制限に注意してください。 詳細については、「[仮想マシン ディスクの制限][vm-disk-limits]」を参照してください。

データ ディスクを追加すると、ディスクに論理ユニット番号 (LUN) の ID が割り当てられます。 LUN ID は必要に応じて指定できます。たとえば、ディスクを交換する際に同じ LUN ID を保持したい場合や、特定の LUN ID を検索するアプリケーションがある場合などに指定します。 ただし、ディスクごとに一意な LUN ID である必要があります。

Premium Storage アカウントで使用する VM のディスクは SSD なので、I/O スケジューラを変更して、SSD のパフォーマンスを最適化することができます。 一般的な推奨事項は、SSD に対して NOOP スケジューラを使用することですが、[iostat] などのツールを使用してワークロードのディスク I/O パフォーマンスを監視する必要があります。

最適なパフォーマンスを得るには、診断ログを保持するためのストレージ アカウントを別途作成します。 診断ログには、標準的なローカル冗長ストレージ (LRS) アカウントがあれば十分です。

### <a name="network-recommendations"></a>ネットワークの推奨事項

パブリック IP アドレスは、動的でも静的でもかまいません。 既定では、動的アドレスになっています。

* 変化しない固定 IP アドレスが必要な場合 (たとえば、DNS に A レコードを作成する必要がある場合や IP アドレスをホワイトリストに登録する必要がある場合) は、[静的 IP アドレス][static-ip]を予約します。
* IP アドレスの完全修飾ドメイン名 (FQDN) を作成することもできます。 これにより、その FQDN を参照する DNS で [CNAME レコード][cname-record]を登録できます。 詳細については、「[Azure Portal での完全修飾ドメイン名の作成][fqdn]」を参照してください。

すべての NSG に[既定の規則][nsg-default-rules] (すべての受信インターネット トラフィックをブロックする規則など) のセットが含まれています。 既定のルールを削除することはできませんが、他の規則でオーバーライドすることはできます。 インターネット トラフィックを有効にするには、特定のポート (HTTP のポート 80 など) への着信トラフィックを許可するルールを作成します。

SSH を有効にするには、TCP ポート 22 への着信トラフィックを許可する規則を NSG に追加します。

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

VM の規模は、[VM サイズを変更する][vm-resize]ことでスケールアップまたはスケールダウンできます。 水平方向にスケール アウトするには、ロード バランサーの背後に 2 つ以上の VM を配置します。 詳細については、「[Running multiple VMs on Azure for scalability and availability ][multi-vm]」 (スケーラビリティと可用性のために Azure で複数の VM を実行する) を参照してください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

可用性を高めるには、複数の VM を可用性セットにデプロイします。 こうすることにより、[サービス レベル アグリーメント][vm-sla] (SLA) も高くなります。

VM は、[計画的メンテナンス][planned-maintenance]または[計画外メンテナンス][manage-vm-availability]の影響を受ける可能性があります。 [VM の再起動ログ][reboot-logs]を参照すると、VM の再起動が計画的なメンテナンスによるものかどうかを確認できます。

VHD は [Azure Storage][azure-storage] に格納され、Azure Storage は持続性と可用性を確保するためにレプリケートされます。

通常の操作中に (ユーザー エラーなどによる) 偶発的なデータの損失から保護するために、[BLOB スナップショット][blob-snapshot]や他のツールを使用してポイントインタイム バックアップを実装することも必要です。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

**リソース グループ** 同じライフ サイクルを共有する密結合のリソースを同じ[リソース グループ][resource-manager-overview]に配置します。 リソース グループを使用すると、グループとしてリソースをデプロイおよび監視し、リソース グループ別に請求コストをまとめることができます。 セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。 リソースにはわかりやすい名前を付けます。 これにより、特定のリソースを見つけて、その役割を理解することが簡単になります。 「[Recommended Naming Conventions for Azure Resources][naming conventions]」(Azure リソースの推奨される名前付け規則) を参照してください。

**SSH**。 Linux VM を作成する前に、2048 ビット RSA 公開/秘密キー ペアを生成します。 VM を作成する場合は、公開キー ファイルを使用します。 詳細については、[Azure 上の Linux または Mac における SSH の使用方法][ssh-linux]に関する記事を参照してください。

**VM の停止。** Azure では、"停止" 状態と "割り当て解除済み" 状態が区別されます。 VM が割り当て解除されたときではなく、VM が停止状態のときに課金されます。 

Azure Portal の **[停止]** ボタンを使用すると、VM の割り当てが解除されます。 ログイン中に OS からシャットダウンした場合、VM は停止しますが割り当ては "*解除されない*" ため、引き続き課金されます。 

**VM の削除。** VM を削除しても VHD は削除されません。 つまり、データを失うことなく安全に VM を削除できます。 ただし、Storage に対して引き続き課金されます。 VHD を削除するには、[Blob Storage][blob-storage] からファイルを削除します。

誤って削除されないように、[リソース ロック][resource-lock]を使用してリソース グループ全体をロックするか、VM などの個々のリソースをロックします。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。 Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。 セキュリティ センターは、Azure サブスクリプションごとに構成されます。 「[Azure Security Center クイック スタート ガイド][security-center-get-started]」に従って、セキュリティ データの収集を有効にします。 データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。

**更新プログラムの管理。** 有効な場合、セキュリティ センターは、セキュリティと重要な更新プログラムが不足しているかどうかを確認します。 

**マルウェア対策。** 有効な場合、セキュリティ センターは、マルウェア対策ソフトウェアがインストールされているかどうかを確認します。 セキュリティ センターを使用して、Azure Portal 内からマルウェア対策ソフトウェアをインストールすることもできます。

**操作。** [ロールベースのアクセス制御][rbac] (RBAC) を使用して、デプロイする Azure リソースへのアクセスを制御します。 RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。 たとえば、閲覧者の役割では、Azure リソースを表示することはできますが、作成、削除、または管理することはできません。 一部の役割は、特定の Azure リソースの種類に固有です。 たとえば、仮想マシン作成協力者ロールでは、VM の再起動または割り当て解除、管理者パスワードのリセット、新しい VM の作成などができます。 このアーキテクチャで役立つ他の[組み込み RBAC ロール][rbac-roles]には、[DevTest Labs ユーザー][rbac-devtest]や[ネットワーク共同作成者][rbac-network]などがあります。 ユーザーを複数の役割に割り当てることができ、よりきめ細かいアクセス許可のカスタム ロールを作成することができます。

> [!NOTE]
> RBAC では、VM にログインしているユーザーが実行できる操作は制限されません。 これらのアクセス許可は、ゲスト OS のアカウントの種類によって決まります。   

プロビジョニング操作や他の VM イベントを確認するには、[監査ログ][audit-logs]を使用します。

**データの暗号化。** OS ディスクとデータ ディスクを暗号化する必要がある場合は、[Azure Disk Encryption][disk-encryption] を検討します。 

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのデプロイについては、[GitHub][github-folder] を参照してください。 次のものがデプロイされます。

  * VM をホストするために使用される、**web** という名前の 1 つのサブネットを含む仮想ネットワーク。
  * VM に対する SSH と HTTP トラフィックを許可する 2 つの受信ルールを持つ NSG。
  * Ubuntu 16.04.3 LTS の最新バージョンを実行する VM。
  * Apache HTTP Server を Ubuntu VM にデプロイし、2 つのデータ ディスクをフォーマットするカスタム スクリプト拡張機能の例。

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

1. 上記の前提条件でダウンロードしたリポジトリの `virtual-machines\single-vm\parameters\linux` フォルダーに移動します。

2. `single-vm-v2.json` ファイルを開き、次に示すような引用符の間にユーザー名と SSH キーを入力した後、ファイルを保存します。

  ```bash
  "adminUsername": "",
  "adminsshPublicKey": "",
  ```

3. 次に示すように、`azbb` を実行して VM のサンプルをデプロイします。

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

この参照アーキテクチャのサンプルのデプロイについて詳しくは、[GitHub リポジトリ][git]を参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure の構成要素][azbbv2]について確認します。
- Azure で[複数の VM][multi-vm] をデプロイします。

<!-- links -->
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[multi-vm]: ../virtual-machines-linux/multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[OSPatching]: https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[readme]: https://github.com/mspnp/reference-architectures/blob/master/virtual-machines/single-vm/README.md
[0]: ./images/single-vm-diagram.png "Azure における単一の Linux VM アーキテクチャ"

