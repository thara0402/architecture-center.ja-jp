---
title: Azure での計算流体力学 (CFD) シミュレーションの実行
description: Azure で計算流体力学 (CFD) シミュレーションを実行します。
author: mikewarr
ms.date: 09/20/2018
ms.custom: fasttrack
ms.openlocfilehash: 42921122d74d07bf890f55be61b04c7e9a4f4e87
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004650"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Azure での計算流体力学 (CFD) シミュレーションの実行

計算流体力学 (CFD) シミュレーションでは、膨大な計算時間と特殊なハードウェアが必要です。 クラスターの使用量が増えると、シミュレーションの時間と全体的なグリッドの使用が増え、予備の容量と長いキュー時間に関する問題が発生します。 物理ハードウェアの追加は高価であり、ビジネスで発生する使用量の山と谷に一致しない場合があります。 Azure を利用することで、これらの課題の多くを設備投資なしで克服できます。

Azure では、GPU および CPU 両方の仮想マシンで CFD ジョブを実行するために必要なハードウェアが提供されます。 RDMA (リモート ダイレクト メモリ アクセス) が有効な VM サイズには、低遅延 MPI (Message Passing Interface) 通信に対応する FDR InfiniBand ベースのネットワークがあります。 エンタープライズ規模のクラスター化されたファイル システムを提供する Avere vFXT と組み合わせると、Azure での読み取り操作に最大限のスループットを確保できます。

HPC クラスターの作成、管理、最適化を簡略化するには、Azure CycleCloud を使用して、クラスターをプロビジョニングし、ハイブリッド シナリオとクラウド シナリオ両方でのデータを調整できます。 保留中のジョブを監視することにより、CycleCloud は任意のワークロード スケジューラーに接続されたオンデマンド コンピューティングを自動的に起動し、支払いは使用した分だけで済みます。

## <a name="relevant-use-cases"></a>関連するユース ケース

CFD アプリケーションに関連するその他の業界:

* 航空
* 自動車
* 建築 HVAC
* 石油、ガス
* ライフ サイエンス

## <a name="architecture"></a>アーキテクチャ

![アーキテクチャ ダイアグラム][architecture]

この図は、Azure でオンデマンド ノードのジョブ監視を提供する一般的なハイブリッド設計の概要を示したものです。

1. Azure CycleCloud サーバーに接続してクラスターを構成します。
2. MPI 用の RDMA 対応マシンを使用して、クラスターのヘッド ノードを構成して作成します。
3. オンプレミスのヘッド ノードを追加して構成します。
4. 十分なリソースがない場合、Azure CycleCloud は Azure のコンピューティング リソースをスケールアップ (またはスケールダウン) します。 過剰な割り当てを防ぐため、あらかじめ設定された制限を定義できます。
5. 実行ノードに割り当てられたタスク。
6. オンプレミスの NFS サーバーから Azure にキャッシュされたデータ。
7. Azure キャッシュ用の Avere vFXT から読み取られたデータ。
8. Azure CycleCloud サーバーに中継されたジョブとタスクの情報。

### <a name="components"></a>コンポーネント

* [Azure CycleCloud][cyclecloud] は、Azure で HPC とビッグ コンピューティング クラスターを作成、管理、運用、最適化するためのツールです。
* [Azure 上の Avere vFXT][avere] は、クラウド向けに構築されたエンタープライズ規模のクラスター化されたファイル システムを提供するために使用されます。
* [Azure Virtual Machines (VM)][vms] は、コンピューティング インスタンスの静的なセットの作成に使用されます。
* [Virtual Machine Scale Sets (仮想マシン スケール セット)][vmss] は、Azure CycleCloud によってスケールアップまたはスケールダウンされる同じ VM のグループを提供します。
* [Azure Storage アカウント](/azure/storage/common/storage-introduction)は、同期とデータ保持に使用されます。
* [Virtual Network](/azure/virtual-network/virtual-networks-overview) では、Azure Virtual Machine (VM) などのさまざまな種類の Azure リソースが、他の Azure リソース、インターネット、およびオンプレミスのネットワークと安全に通信することができます。

### <a name="alternatives"></a>代替手段

お客様は、Azure CycleCloud を使用してグリッド全体を Azure 内に作成することもできます。 このセットアップでは、Azure CycleCloud サーバーは Azure サブスクリプション内で実行されます。

ワークロード スケジューラーの管理を必要としない最新のアプリケーション アプローチでは、[Azure Batch][batch] が役に立ちます。 Azure Batch では、大規模な並列コンピューティングやハイパフォーマンス コンピューティング (HPC) のアプリケーションをクラウドで効率的に実行することができます。 Azure Batch を使用すると、インフラストラクチャを手動で構成または管理することなく、アプリケーションを並列で実行したり大規模に実行したりするように Azure コンピューティング リソースを定義できます。 Azure Batch はコンピューティング集中型のタスクのスケジュールを設定し、要件に基づいてコンピューティング リソースを動的に追加または削除します。

### <a name="scalability-and-security"></a>スケーラビリティ、セキュリティ

Azure CycleCloud 上の実行ノードのスケーリングは、手動で、または自動スケールを使用して、実現できます。 詳しくは、[CycleCloud の自動スケール][cycle-scale]に関するページをご覧ください。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

## <a name="deploy-this-scenario"></a>このシナリオのデプロイ

Azure にデプロイする前に、いくつかの前提条件が必要です。 Resource Manager テンプレートをデプロイする前に、次の手順に従ってください。
1. アプリ ID、表示名、名前、パスワード、テナントを取得するための[サービス プリンシパル][cycle-svcprin]を作成します。
2. CycleCloud サーバーに安全にサインインするための [SSH キー ペア][cycle-ssh]を生成します。

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. [CycleCloud サーバーにログイン][cycle-login]して、新しいクラスターを構成および作成します。
4. [クラスターを作成][cycle-create]します。

Avere Cache は、アプリケーション ジョブ データの読み取りスループットを大幅に高めることができるオプションのソリューションです。 Avere vFXT for Azure は、オンプレミスまたは Azure Blob Storage に格納されているデータを利用しながら、これらのエンタープライズ HPC アプリケーションをクラウドで実行する問題を解決します。

オンプレミスのストレージとクラウド コンピューティングの両方を使用するハイブリッド インフラストラクチャを計画している場合、HPC アプリケーションは NAS デバイスに格納されているデータを使用して Azure に "介入" し、必要に応じて仮想 CPU を起動できます。 データ セットがクラウドに完全に移動されることはありません。 要求されたバイトは、処理の間に Avere クラスターを使用して一時的にキャッシュされます。

Avere vFXT のインストールのセットアップと構成については、[Avere のセットアップと構成ガイド][avere]に関するページをご覧ください。

## <a name="pricing"></a>価格

CycleCloud サーバーを使用して HPC の実装を実行するコストは、さまざまな要因によって異なります。 たとえば、CycleCloud は使用されたコンピューティング時間量によって課金され、Master サーバーと CycleCloud サーバーは通常常に割り当てられて実行しています。 Execute ノードを実行するコストは、稼働している時間と使用されたサイズによって決まります。 ストレージとネットワークに対する通常の Azure の料金も適用されます。

このシナリオでは CFD アプリケーションを Azure で実行する方法を示すので、マシンには RDMA 機能が必要であり、これは特定の VM サイズでのみ使用できます。 次に示すのは、1 か月間継続して 1 日 8 時間割り当てられた、データ送信が 1 TB のスケール セットにかかるコストの例です。 Azure CycleCloud サーバーと、Azure インストール用 Avere vFXT の価格も含まれます。

* リージョン:北ヨーロッパ
* Azure CycleCloud サーバー:1 x Standard D3 (4 x CPU、14 GB メモリ、Standard HDD 32 GB)
* Azure CycleCloud Master サーバー:1 x Standard D12 v (4 x CPU、28 GB メモリ、Standard HDD 32 GB)
* Azure CycleCloud ノード アレイ:10 x Standard H16r (16 x CPU、112 GB メモリ)
* Azure クラスター上の Avere vFXT:3 x D16s v3 (200 GB OS、Premium SSD 1-TB データ ディスク)
* データ送信:1 TB (テラバイト)

上に示したハードウェアについてはこちらの[価格見積もり][pricing]をご覧ください。

## <a name="next-steps"></a>次の手順

サンプルを展開した後、[Azure CycleCloud][cyclecloud] の詳細を学習してください。

## <a name="related-resources"></a>関連リソース

* [RDMA 対応のマシン インスタンス][rdma]
* [RDMA インスタンス VM のカスタマイズ][rdma-custom]

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
