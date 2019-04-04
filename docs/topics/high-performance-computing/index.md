---
title: Azure でのハイ パフォーマンス コンピューティング (HPC)
description: Azure 上で実行される HPC ワークロードを構築するためのガイド
author: adamboeglin
ms.date: 2/4/2019
ms.openlocfilehash: 5263dd3a06e5244bf804df4be6ec57d789574f76
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489200"
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a>Azure でのハイ パフォーマンス コンピューティング (HPC)

## <a name="introduction-to-hpc"></a>HPC の概要

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

"ビッグ コンピューティング" とも呼ばれるハイ パフォーマンス コンピューティング (HPC) では、CPU または GPU ベースのコンピューターを大量に使用して、複雑な数学的タスクを解決します。

多くの業界では HPC を使用して、最も困難な問題の一部を解決しています。  これらには、以下のようなワークロードがあります。

- Genomics
- 石油およびガスのシミュレーション
- Finance
- 半導体の設計
- Engineering
- 天気のモデリング

### <a name="how-is-hpc-different-on-the-cloud"></a>クラウドでの HPC の違い

オンプレミスの HPC システムとクラウドのそれとの主な違いの 1 つは、必要に応じてリソースを動的に追加および削除できることです。  動的スケーリングによって、コンピューター能力がボトルネットになることがなく、お客様はジョブの要件に応じてインフラストラクチャを適切にサイズ調整できます。

次の記事では、この動的スケーリング機能について詳しく説明します。

- [ビッグ コンピューティング アーキテクチャ スタイル](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [自動スケーリングのベスト プラクティス](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a>実装チェックリスト

独自の HPC ソリューションを Azure に実装しようとしている場合は、以下のトピックをご確認ください。

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - 要件に基づいて適切な[アーキテクチャ](#infrastructure)を選択する
> - ワークロードに適した[コンピューティング](#compute) オプションを把握する
> - ニーズを満たす適切な[ストレージ](#storage) ソリューションを特定する
> - すべてのリソースを[管理](#management)することになる方法を決定する
> - クラウドに対して[アプリケーション](#hpc-applications)を最適化する
> - インフラストラクチャを[セキュリティで保護](#security)する

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a>インフラストラクチャ

HPC システムの構築には、多数のインフラストラクチャ コンポーネントが必要です。  どのような方法で HPC ワークロードを管理することにしても、基礎となるコンポーネントを提供するのは、コンピューティング、ストレージ、およびネットワークです。

### <a name="example-hpc-architectures"></a>HPC アーキテクチャの例

HPC アーキテクチャを設計して Azure に実装するには、さまざまな方法があります。  HPC アプリケーションは、数千のコンピューティング コアにスケーリングしたり、オンプレミスのクラスターに拡張したり、100% クラウド ネイティブのソリューションとして実行したりできます。

次のシナリオでは、HPC ソリューションを構築する一般的な方法をいくつか説明します。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Azure でのコンピューター支援エンジニアリング サービス</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Azure で、コンピューター支援エンジニアリング (CAE) に、サービスとしてのソフトウェア (SaaS) プラットフォームを提供します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Azure での計算流体力学 (CFD) シミュレーション</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Azure で計算流体力学 (CFD) シミュレーションを実行します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Azure での 3D ビデオのレンダリング</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Azure Batch サービスを使用して、Azure でネイティブ HPC ワークロードを実行します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a>Compute

Azure では、CPU の負荷が高いワークロードと GPU の負荷が高いワークロードの両方に対して最適化された幅広いサイズが提供されています。

#### <a name="cpu-based-virtual-machines"></a>CPU ベースの仮想マシン

- [Linux VM](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Windows VM](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) の VM
  
#### <a name="gpu-enabled-virtual-machines"></a>GPU 対応仮想マシン

N シリーズ VM は、人工知能 (AI) の学習や視覚化などによりコンピューティングやグラフィック使用量が多いアプリケーションのために設計された NVIDIA GPU を採用しています。

- [Linux VM](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Windows VM](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a>Storage

バッチ ワークロードや HPC ワークロードが大規模の場合には、従来のクラウド ファイル システムの容量を上回るデータ ストレージが必要になったり、データ アクセスが発生したりします。  Azure 上の HPC アプリケーションの速度ニーズと容量ニーズ、両方に対応できるソリューションが多数あります

- [Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) (データ ストレージに対する処理速度とアクセス性を高め、エッジでハイパフォーマンス コンピューティングを実現)
- [BeeGFS](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [ストレージ最適化仮想マシン](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Blob、Table、Queue Storage](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Azure SMB ファイル ストレージ](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Intel Cloud Edition Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

Azure での Lustre、GlusterFS、BeeGFS の詳しい比較情報については、[Azure での並列ファイル システムに関する電子ブック](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)を参照してください。

### <a name="networking"></a>ネットワーク

H16r、H16mr、A8、A9 の VM は、高スループットのバックエンド RDMA ネットワークに接続できます。 このネットワークでは、Microsoft MPI または Intel MPI の下で実行される緊密に結合した並列アプリケーションのパフォーマンスを高めることができます。

- [RDMA 対応のインスタンス](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [Virtual Network](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [ExpressRoute](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a>管理

### <a name="do-it-yourself"></a>自作

Azure で一から HPC システムを構築すると、高度な柔軟性が得られるものの、多くの場合、メンテナンスの手間が非常に大きくなります。  

1. Azure 仮想マシンまたは[仮想マシンのスケール セット](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)に独自のクラスター環境を設定します。
2. Azure Resource Manager テンプレートを使って、業界をリードする[ワークロード マネージャー](#workload-managers)、インフラストラクチャ、および[アプリケーション](#hpc-applications)をデプロイします。
3. HPC および GPU の [VM サイズ](#compute)を選択します。これには、MPI ワークロードまたは GPU ワークロードのための特別なハードウェアとネットワーク接続が含まれます。
4. I/O が集中するワークロードのための[高性能ストレージ](#storage)を追加します。

### <a name="hybrid-and-cloud-bursting"></a>ハイブリッドとクラウド バースティング

Azure に接続したい既存のオンプレミス HPC システムがある場合、作業の開始に役立つリソースが多数あります。

最初に、ドキュメントの[オンプレミス ネットワークを Azure に接続するオプション](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)に関する記事を確認してください。  そこで、以下の接続オプションに関する情報が必要になるでしょう。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">VPN ゲートウェイを使用した Azure へのオンプレミス ネットワークの接続</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>この参照アーキテクチャでは、サイト間の仮想プライベート ネットワーク (VPN) を使用して、オンプレミス ネットワークを Azure に拡張する方法を示します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。 プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Azure 仮想ネットワークと、 VPN ゲートウェイ フェールオーバー付きの ExpressRoute で接続されたオンプレミス ネットワークをカバーする、可用性が高くセキュアなサイト間ネットワーク アーキテクチャを実装します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

ネットワーク接続が安全に確立されたら、既存の[ワークロード マネージャー](#workload-managers)のバースティング機能と共に、オンデマンドでクラウド コンピューティング リソースの使用を開始できます。

### <a name="marketplace-solutions"></a>Marketplace のソリューション

[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) では、多数のワークロード マネージャーが提供されています。

- [RogueWave CentOS-based HPC](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [SUSE Linux Enterprise Server for HPC](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [TIBCO Grid Server Engine](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [Windows および Linux 用 Azure データ サイエンス VM ](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [D3View](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [UberCloud](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a>Azure Batch

[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) は、大規模な並列コンピューティングやハイ パフォーマンス コンピューティング (HPC) のアプリケーションをクラウドで効率的に実行するためのプラットフォーム サービスです。 大量のコンピューティングを要する作業を仮想マシンの管理されたプールで実行するようにスケジュールを設定し、ジョブのニーズに合わせてコンピューティング リソースを自動的にスケールできます。

SaaS のプロバイダーやデベロッパーは、Batch の SDK とツールを使って、HPC アプリケーションやコンテナー ワークロードを Azure に統合し、Azure にデータをステージングして、ジョブ実行パイプラインを作成できます。

### <a name="azure-cyclecloud"></a>Azure CycleCloud

[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) は、Azure で任意のスケジューラ (Slurm、Grid Engine、HPC Pack、HTCondor、LSF、PBS Pro、Symphony など) を使用して HPC ワークロードを管理する最も簡単な方法を提供します

CycleCloud では以下を行えます。

- スケジューラ、コンピューティング VM、ストレージ、ネットワーク、キャッシュなど、すべてのクラスターとその他のリソースをデプロイする
- ジョブ、データ、クラウドのワークフローを調整する
- ジョブを実行できるユーザー、その実行場所、その実行にかかるコストを管理者が完全に制御できるようにする
- コスト管理、Active Directory の統合、監視、レポートなど、高度なポリシーおよび管理機能を使用して、クラスターをカスタマイズおよび最適化する
- 変更せずに現在のジョブ スケジューラとアプリケーションを使用する
- さまざまな HPC ワークロードや業界に対して、組み込みの自動スケーリングと実績が証明されている参照アーキテクチャを活用する

### <a name="workload-managers"></a>ワークロード マネージャー

Azure のインフラストラクチャで実行できるクラスターおよびワークロード マネージャーの例を次に示します。 Azure VM にスタンドアロンのクラスターを作成するか、オンプレミス クラスターから Azure VM にバーストします。

- [Alces フライト コンピューティング](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [TIBCO DataSynapse GridServer](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [Bright Cluster Manager](http://www.brightcomputing.com/technology-partners/microsoft)
- [IBM Spectrum Symphony および Symphony LSF](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [PBS Pro](http://pbspro.org)
- [Altair](http://www.altair.com/)
- [Rescale](https://www.rescale.com/azure/)
- [Microsoft HPC Pack](https://technet.microsoft.com/library/mt744885.aspx)
  - [Windows 用の HPC Pack](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [Linux 用の HPC Pack](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a>Containers

コンテナーは、一部の HPC ワークロードを管理するためにも使用できます。  Azure Kubernetes Service (AKS) などのサービスを使用すると、マネージド Kubernetes クラスターを Azure 内に簡単にデプロイできます。

- [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [コンテナー レジストリ](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a>コスト管理

Azure での HPC コストの管理は、いくつかの異なる方法を通じて行えます。  [Azure の購入オプション](https://azure.microsoft.com/pricing/purchase-options/)を確認して、ご自分の組織にとって最適な方法を見つけてください。

[低優先度 VM](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) を使うと、非常に低コストで未使用の容量を利用できます。

## <a name="security"></a>セキュリティ

Azure におけるセキュリティのベスト プラクティスの概要については、[Azure のセキュリティに関するドキュメント](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)を参照してください。  

[クラウド バースティング](#hybrid-and-cloud-bursting)に関するセクションにあるネットワーク構成のほかに、ハブ/スポーク構成を実装して、コンピューティング リソースを分離することができます。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Azure にハブスポーク ネットワーク トポロジを実装する</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。 スポークは、ハブに対して配置される VNet であり、ワークロードを分離するために使用されます。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>この参照アーキテクチャは、ハブスポーク参照アーキテクチャに基づいて作成されており、すべてのスポークで利用できる共有サービスがハブに含まれます。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a>HPC アプリケーション

Azure ではカスタム HPC アプリケーションや商用 HPC アプリケーションを実行できます。 このセクションのさまざまな例で、VM やコンピューティング コアを追加して効率的にスケールできることが確認されています。 すぐにデプロイ可能なソリューションについては、[Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) を参照してください。

> [!NOTE]
> クラウドで実行するためのライセンスまたはその他の制限事項については、商用アプリケーションのベンダーに確認してください。 すべてのベンダーが従量課金制ライセンスを提供しているとは限りません。 ソリューション用にクラウド内にライセンス サーバーを用意したり、オンプレミスのライセンス サーバーに接続することが必要になる場合があります。

### <a name="engineering-applications"></a>エンジニアリング アプリケーション

- [Altair RADIOSS](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [ANSYS CFD](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [MATLAB Distributed Computing Server](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [StarCCM+](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [OpenFOAM](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a>グラフィックとレンダリング

- Azure Batch での [Autodesk Maya、3ds Max、Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="ai-and-deep-learning"></a>AI とディープ ラーニング

- [Microsoft Cognitive Toolkit](/cognitive-toolkit/cntk-on-azure)
- [ディープ ラーニング VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [ディープ ラーニング用の Batch Shipyard レシピ](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a>MPI プロバイダー

- [Microsoft MPI](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a>リモート視覚化

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Citrix を使用した Linux 仮想デスクトップ</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Azure で Citrix を使用して Linux デスクトップ向けの VDI 環境を構築します。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a>パフォーマンス ベンチマーク

- [コンピューティング ベンチマーク](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a>顧客事例

多くのお客様が、Azure を HPC ワークロードに使用して大きな成功を収めています。  以下では、それらのお客様のケース スタディをいくつかご紹介します。

- [ANEO](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [AXA Global P&C](https://customers.microsoft.com/story/axa-global-p-and-c)
- [Axioma](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [d3View](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [EFS](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [Hymans Robertson](https://customers.microsoft.com/story/hymans-robertson)
- [MetLife](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [Microsoft Research](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [Milliman](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [Mitsubishi UFJ Securities International](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [NeuroInitiative](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [Schlumberger](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [Towers Watson](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a>その他の重要な情報

- 大規模なワークロードの実行を試行する前には、[vCPU クォータ](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)を引き上げておくようにしてください。

## <a name="next-steps"></a>次の手順

最新のお知らせについては、以下を参照してください。

- [Microsoft HPC と Batch のチーム ブログ](http://blogs.technet.com/b/windowshpc/)
- [Azure のブログ](https://azure.microsoft.com/blog/tag/hpc/)にアクセスしてください。

### <a name="microsoft-batch-examples"></a>Microsoft Batch の例

以下のチュートリアルでは、Microsoft Batch でのアプリケーションの実行について詳しく説明します

- [Batch を使った初めての開発](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Azure Batch のコード サンプルを使用する](https://github.com/Azure/azure-batch-samples)
- [Batch で優先順位の低い VM を使用する](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Batch Shipyard を使ってコンテナー化した HPC ワークロードを実行する](https://github.com/Azure/batch-shipyard)
- [Batch で並列 R ワークロードを実行する](https://github.com/Azure/doAzureParallel)
- [Batch でオンデマンド Spark ジョブを実行する](https://github.com/Azure/aztk)
- [Batch プールでコンピューティング集中型 VM を使用する](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)