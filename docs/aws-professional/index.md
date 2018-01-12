---
title: "AWS プロフェッショナルのための Azure"
description: "Microsoft Azure アカウント、プラットフォーム、およびサービスの基本を理解します。 AWS プラットフォームと Azure プラットフォームの主要な類似点と相違点についても説明します。 AWS の経験を Azure でご活用ください。"
keywords: "AWS エキスパート, Azure との比較, AWS との比較, Azure と AWS の違い, Azure と Aws"
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: ac96110e3fe69b4bb69714e18fd0f193208bc244
ms.sourcegitcommit: 744ad1381e01bbda6a1a7eff4b25e1a337385553
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2018
---
# <a name="azure-for-aws-professionals"></a>AWS プロフェッショナルのための Azure

この記事は、アマゾン ウェブ サービス (AWS) エキスパートが Microsoft Azure のアカウント、プラットフォーム、およびサービスの基本を理解するために役立ちます。 AWS プラットフォームと Azure プラットフォームの主要な類似点と相違点についても説明しています。

学習内容:

* アカウントとリソースが Azure でどのように構成されているか。
* 利用可能なソリューションが Azure でどのように構造化されているか。
* 主要な Azure サービスが AWS サービスとどのように異なっているか。

Azure と AWS のさまざまな機能は時間の経過と共に独立して構築されたため、各機能の実装と設計には重要な相違点があります。

## <a name="overview"></a>概要

AWS と同じように、Microsoft Azure も、コンピューティング、ストレージ、データベース、およびネットワーク サービスを中心に構築されています。 多くの場合、両方のプラットフォームが提供する製品とサービスは、基本的には同等です。 AWS と Azure の両方で、Windows または Linux ホストに基づく可用性が高いソリューションを構築できます。 したがって、Linux と OSS のテクノロジを使用する開発に慣れていれば、両方のプラットフォームでその仕事を行うことができます。

2 つのプラットフォームの機能は似ていますが、それらの機能を提供するリソースの構成は、しばしば異なっています。 ソリューションを構築するために必要なサービス間の一対一のリレーションシップは、常に明確であるとは限りません。 さらに、特定のサービスが片方のプラットフォームでのみ提供されている場合があります。 [Azure と AWS の対応するサービスの一覧](services.md)を参照してください。

## <a name="accounts-and-subscriptions"></a>アカウントとサブスクリプション

Azure サービスは、組織の規模とニーズに応じて、さまざまな価格オプションで購入できます。 詳細については、[価格の概要](https://azure.microsoft.com/pricing/)に関するページを参照してください。

[Azure サブスクリプション](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)は、リソースと割り当てられた所有者をグループ化したものです。所有者は課金とアクセス許可の管理を担当します。 特定の AWS アカウントの下に作成されたすべてのリソースはそのアカウントに関連付けられる AWS とは異なり、サブスクリプションは所有者アカウントとは無関係に存在し、必要に応じて新しい所有者に再割り当てすることができます。

![AWS アカウントと Azure サブスクリプションの構造と所有権の比較](./images/azure-aws-account-compare.png "AWS アカウントと Azure サブスクリプションの構造と所有権の比較")
<br/>*AWS アカウントと Azure サブスクリプションの構造と所有権の比較*
<br/><br/>

サブスクリプションには、3 種類の管理者アカウントが割り当てられます。

-   **アカウント管理者** - サブスクリプションの所有者であり、このアカウントに対して、サブスクリプションで使用されたリソースの課金が行われます。 アカウント管理者は、サブスクリプションの所有権を譲渡することでのみ変更できます。

-   **サービス管理者** - このアカウントには、サブスクリプション内にリソースを作成して管理する権限がありますが、課金は行われません。 既定では、アカウント管理者とサービス管理者は同じアカウントに割り当てられます。 アカウント管理者は、サブスクリプションの技術面と運用面を管理するサービス管理者アカウントに別のユーザーを割り当てることができます。 サービス管理者はサブスクリプションごとに 1 人だけ存在します。

-   **共同管理者** - 1 つのサブスクリプションに複数の共同管理者アカウントを割り当てることができます。 共同管理者はサービス管理者を変更できませんが、それ以外は、サブスクリプションのリソースとユーザーを完全に制御できます。

サブスクリプション レベルの下で、特定のリソースに対してユーザー ロールと個々のアクセス許可を割り当てることもできます。これは、AWS で IAM ユーザーとグループに対してアクセス許可を付与することに似ています。 Azure では、すべてのユーザー アカウントは、Microsoft アカウントまたは組織アカウント (Azure Active Directory を通して管理されるアカウント) に関連付けられます。

AWS アカウントと同じように、サブスクリプションには既定のサービスのクォータと制限があります。 これらの制限の完全な一覧については、「[Azure サブスクリプションとサービスの制限、クォータ、制約](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)」をご覧ください。
これらの制限は、[管理ポータルでのサポート リクエストの申し込み](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)によって、最大値まで増やすことができまです。

### <a name="see-also"></a>関連項目

-   [Azure 管理者ロールを追加または変更する方法](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Azure の請求書と毎日の使用状況データをダウンロードする方法](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>リソース管理

Azure では、"リソース" という用語を AWS と同じように使用しています。つまり、すべてのコンピューティング インスタンス、ストレージ オブジェクト、ネットワーク デバイス、プラットフォームで作成または構成できるその他のエンティティを意味します。

Azure リソースは、2 つのモデル ([Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) と従来の Azure [クラシック デプロイメント モデル](/azure/azure-resource-manager/resource-manager-deployment-model)) のどちらかでデプロイして管理されます。
すべての新しいリソースは、Resource Manager モデルを使用して作成されます。

### <a name="resource-groups"></a>リソース グループ

Azure と AWS の両方に、VM、ストレージ、仮想ネットワーク デバイスなどのリソースを整理する "リソース グループ" と呼ばれるエンティティがあります。 ただし、[Azure リソース グループ](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)と AWS リソース グループは同等ではありません。

AWS では、1 つのリソースを複数のリソース グループにタグ付けできますが、Azure のリソースは常に 1 つのリソース グループに関連付けられます。 あるリソース グループ内に作成されたリソースを別のグループに移動できますが、リソースは一度に 1 つのリソース グループ内にのみ存在できます。 リソース グループは、Azure Resource Manager によって使用される基本グループです。

リソースは、[タグ](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)を使用して整理することもできます。
タグは、サブスクリプションのリソースを、リソース グループのメンバーシップに関係なくグループ化できるようにするキーと値のペアです。

### <a name="management-interfaces"></a>管理インターフェイス

Azure では、さまざまな方法でリソースを管理できます。

-   [Web インターフェイス](https://azure.microsoft.com/documentation/articles/resource-group-portal/)。
    AWS Dashboard と同じように、Azure ポータルは、Azure リソース用の完全な Web ベースの管理インターフェイスを提供します。

-   [REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/)。
    Azure Resource Manager REST API は、Azure ポータルで使用できる機能の大半に、プログラムによってアクセスできるようにします。

-   [コマンド ライン](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/)。
    Azure CLI 2.0 ツールは、Azure リソースを作成して管理できるコマンド ライン インターフェイスを提供します。 Azure CLI は [Windows、Linux、および Mac OS X](https://aka.ms/azurecli2) で使用できます。

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/)。
    PowerShell 用の Azure モジュールによって、スクリプトを使用した自動管理タスクを実行できます。 PowerShell は [Windows、Linux、および Mac OS X](https://github.com/PowerShell/PowerShell) で使用できます。

-   [テンプレート](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)。
    Azure Resource Manager テンプレートは、AWS CloudFormation サービスのような JSON テンプレート ベースのリソース管理機能を提供します。

どのインターフェイスでも、Azure でリソースの作成、デプロイ、または変更を行うための中心はリソース グループです。 これは、CloudFormation のデプロイ時に AWS リソースをグループ化するために "スタック" が果たす役割に似ています。

これらのインターフェイスの構文と構造は AWS とは異なっていますが、同等の機能を備えています。 さらに、AWS で使用される多数のサード パーティ製の管理ツール ([Hashicorp の Terraform](https://www.terraform.io/docs/providers/azurerm/) や [Netflix Spinnaker](http://www.spinnaker.io/) など) を Azure でも使用できます。

### <a name="see-also"></a>関連項目

-   [Azure リソース グループのガイドライン](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>リージョンとゾーン (高可用性)

障害が及ぼす影響の範囲はさまざまです。 たとえば、ディスク障害などの一部のハードウェア障害が、1 台のホスト コンピューターに影響を及ぼすことがあります。 障害が発生したネットワーク スイッチが、サーバー ラック全体に影響する場合もあります。 データ センターの停電など、データ センター全体を中断させる障害はまれです。 ほとんど発生しませんが、リージョン全体が利用できなくなることもあります。

アプリケーションの回復性を実現するための 1 つの手段として、冗長性があります。 ただし、この冗長性は、アプリケーションの設計時に計画しなければなりません。 また、必要な冗長性のレベルはビジネス要件によって異なります。リージョン障害に備えるために、すべてのアプリケーションにリージョン間での冗長性が必要だとは限りません。 一般的に、冗長性と信頼性を高めると、コストと複雑さが増すというトレードオフがあります。  

AWS では、リージョンは、2 つ以上の可用性ゾーンに分割されます。 可用性ゾーンは、地理的地域内に物理的に分離されたデータ センターに対応します。 Azure には、**可用性セット**、**可用性ゾーン**、**ペアのリージョン**など、あらゆる障害レベルでアプリケーションの冗長性を確保するための機能が複数用意されています。 

![](../resiliency/images/redundancy.svg)

各オプションを以下の表に示します。

| &nbsp; | 可用性セット | 可用性ゾーン | ペアのリージョン |
|--------|------------------|-------------------|---------------|
| 障害の範囲 | ラック | データセンター | リージョン |
| 要求のルーティング | Load Balancer | クロスゾーン ロード バランサー | Traffic Manager |
| ネットワーク待ち時間 | 非常に低い | 低 | 中～高 |
| 仮想ネットワーク  | VNet | VNet | リージョン間 VNet ピアリング (プレビュー) |

### <a name="availability-sets"></a>可用性セット 

ディスクやネットワーク スイッチの障害など、ローカライズされたハードウェアの障害から保護するには、可用性セットに 2 つ以上の VM をデプロイします。 可用性セットは、電源とネットワーク スイッチを共有する 2 つ以上の "*障害ドメイン*" で構成されます。 可用性セットの VM は複数の障害ドメインに分散されるため、ある障害ドメインがハードウェア障害の影響を受けた場合は、ネットワーク トラフィックを他の障害ドメインの VM にルーティングできます。 可用性セットの詳細については、「[Azure での Windows 仮想マシンの可用性の管理](/azure/virtual-machines/windows/manage-availability)」を参照してください。

VM インスタンスが可用性セットに追加されると、それらには[更新ドメイン](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)も割り当てられます。 更新ドメインは、計画済みメンテナンス イベントが同時に設定される VM グループです。 複数の更新ドメインに VM を分散させることで、予定された更新と修正イベントが特定の時点でこれらの VM のサブセットのみに作用することが保証されます。

アプリケーション内のインスタンスのロール別に可用性セットを編成して、各ロールで 1 つのインスタンスが操作可能であることを保証する必要があります。 たとえば、3 層構造の Web アプリケーションでは、フロント エンド層、アプリケーション層、およびデータ層に個別の可用性セットを作成します。

![各アプリケーション ロール用の Azure 可用性セット](./images/three-tier-example.png "各アプリケーション ロール用の Azure 可用性セット")

### <a name="availability-zones-preview"></a>可用性ゾーン (プレビュー)

[可用性ゾーン](/azure/availability-zones/az-overview)とは、Azure リージョン内の物理的に独立したゾーンのことです。 可用性ゾーンはそれぞれ異なる電源、ネットワーク、および冷却装置を持ちます。 可用性ゾーンに VM をデプロイすると、データセンター全体の障害からアプリケーションを保護するのに役立ちます。 

### <a name="paired-regions"></a>ペアになっているリージョン

リージョンの障害からアプリケーションを保護するために、複数のリージョンにアプリケーションをデプロイし、[Azure Traffic Manager][traffic-manager] を使用してインターネット トラフィックを異なるリージョンに分散できます。 各 Azure リージョンは別のリージョンとペアになります。 これが[リージョン ペア][paired-regions]になります。 ブラジル南部を除き、リージョン ペアは、税および法の執行を目的としたデータ常駐要件を満たすために同じ地理的場所に配置されます。

可用性ゾーン (物理的に独立しているが、比較的近接する地理的地域に配置されている場合があるデータセンター) とは異なり、ペアになっているリージョンは、通常は少なくとも 300 マイル離れています。 その目的は、大規模災害がペアになっているリージョンの片方のみに影響を与えることを保証することです。 近接するペアにデータベースとステージ サービスのデータを同期するように設定して、プラットフォームの更新プログラムが、ペアになっているリージョンの片方にのみ同時にロールアウトされるように構成できます。

Azure の [geo 冗長ストレージ](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)は、適切なペアになっているリージョンに自動的にバックアップされます。 他のすべてのリソースでは、ペアになっているリージョンを使用した完全に冗長なソリューションの作成は、両方のリージョンにソリューションの完全なコピーを作成することを意味します。


### <a name="see-also"></a>関連項目

-   [Azure の仮想マシンのリージョンと可用性について](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Azure アプリケーションの高可用性](../resiliency/high-availability-azure-applications.md)

-   [Azure アプリケーションのディザスター リカバリー](../resiliency/disaster-recovery-azure-applications.md)

-   [Azure での Linux 仮想マシンに対する計画的なメンテナンス](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>サービス

プラットフォーム間のすべてのサービスの対応の一覧については、[AWS と Azure のサービスの完全比較マトリックス](https://aka.ms/azure4aws-services)を参照してください。

すべての Azure 製品とサービスがすべてのリージョンで使用できるわけではありません。 [リージョン別の製品](https://azure.microsoft.com/regions/services/)に関するページを参照してください。 Azure の各製品またはサービスのアップタイム保証とダウンタイム クレジット ポリシーについては、「[サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/)」ページで確認できます。

この後のセクションでは、一般的に使用される機能とサービスの AWS プラットフォームと Azure プラットフォームでの違いについて、簡単に説明します。

### <a name="compute-services"></a>Compute Services

#### <a name="ec2-instances-and-azure-virtual-machines"></a>EC2 インスタンスと Azure 仮想マシン

AWS インスタンスのタイプと Azure 仮想マシンのサイズは、似たような方法で分類できますが、RAM、CPU、およびストレージの機能に違いがあります。

-   [Amazon EC2 インスタンスのタイプ](https://aws.amazon.com/ec2/instance-types/)

-   [Azure の仮想マシンのサイズ (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Azure の仮想マシンのサイズ (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

AWS の秒単位の課金とは異なり、Azure のオンデマンド VM は分単位で課金されます。

Azure には EC2 スポット インスタンスと専用ホストに相当するものはありません。

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS と VM ディスク用の Azure Storage

Blob ストレージに存在する[データ ディスク](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)によって、Azure VM の持続性のあるデータ ストレージが提供されます。 これは、EC2 インスタンスが Elastic Block Store (EBS) にディスク ボリュームを格納することに似ています。 [Azure の一時ストレージ](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)も、EC2 Instance Storage と同じ待機時間の短い一時的な読み取り/書き込みストレージ (短期ストレージとも呼ばれます) を VM に提供します。

高パフォーマンスのディスク IO は、[Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage) を使用してサポートされます。
これは、AWS が提供する Provisioned IOPS ストレージ オプションに似ています。

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda、Azure Functions、Azure Web-Jobs、および Azure Logic Apps

[Azure Functions](https://azure.microsoft.com/services/functions/) は、サーバーレスなオンデマンド コードを提供するという点で、AWS Lambda に相当します。
ただし、Lambda の機能は、他の Azure サービスとも重複しています。

-   [WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - スケジュールされた、または継続的に実行されるバック グラウンド タスクを作成できます。

-   [Logic Apps](https://azure.microsoft.com/services/logic-apps/) - 通信、統合、およびビジネス ルール管理サービスを提供します。

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>自動スケール、Azure VM のスケーリング、および Azure App Service の自動スケール

Azure の自動スケールは、2 つのサービスによって処理されます。

-   [VM スケール セット](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - 同一の VM セットをデプロイして管理できます。 インスタンスの数は、パフォーマンスのニーズに基づいて自動スケールできます。

-   [App Service の自動スケール](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - Azure App Service ソリューションを自動スケールする機能を提供します。


#### <a name="container-service"></a>Container Service
[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) は、Docker Swarm、Kubernetes、または DC/OS を介して管理されている Docker コンテナーをサポートします。

#### <a name="other-compute-services"></a>その他のコンピューティング サービス 


Azure には、AWS に直接相当するものがないいくつかのコンピューティング サービスがあります。

-   [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 仮想マシンのスケーラブルなコレクション全体にわたってコンピューティング集中型の作業を管理できます。

-   [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 拡張性の高い[マイクロサービス](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) ソリューションを開発してホストするためのプラットフォームです。

#### <a name="see-also"></a>関連項目

-   [ポータルを使用して Azure に Linux VM を作成する](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Azure リファレンス アーキテクチャ: Azure での Linux VM の実行](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Get started with Node.js web apps in Azure App Service (Azure App Service で Node.js Web アプリの使用を開始する)](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Azure リファレンス アーキテクチャ: 基本的な Web アプリケーション](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [初めての Azure 関数の作成](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Storage

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS と Azure Storage

AWS プラットフォームでは、クラウド ストレージは主に 3 つのサービスに分類されます。

-   **Simple Storage Service (S3)** - 基本的なオブジェクト ストレージ。 インターネットでアクセス可能な API を介してデータを利用できるようにします。

-   **Elastic Block Storage (EBS)** - 1 台の VM によるアクセス向きのブロック レベルのストレージ。

-   **Elastic File System (EFS)** - 数千の EC2 インスタンスの共有ストレージとして使用するためのファイル ストレージ。

Azure Storage では、サブスクリプションに関連付けられた[ストレージ アカウント](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)を使用して、次のストレージ サービスを作成して管理できます。

-   [Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 任意の種類のテキスト データやバイナリ データを格納します (ドキュメント、メディア ファイル、アプリケーション インストーラーなど)。 BLOB ストレージをプライベート アクセス用に設定したり、インターネットに公開してコンテンツを共有したりできます。 BLOB ストレージの目的は、AWS の S3 と EBS の両方の目的と同じです。
-   [Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 構造型データセットを格納します。 Table Storage は、NoSQL キー属性データ ストアであるため、開発が迅速化され、大量のデータにすばやくアクセスできます。 AWS の SimpleDB サービスと DynamoDB サービスに似ています。

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - クラウド サービスのコンポーネント間のワークフロー処理と通信のためのメッセージング機能を提供します。

-   [File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 標準的なサーバー メッセージ ブロック (SMB) プロトコルを使用するレガシー アプリケーション用の共有ストレージを提供します。 File Storageは、AWS プラットフォームの EFS と同じように使用されます。
 
#### <a name="glacier-and-azure-storage"></a>Glacier と Azure Storage 

[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) は、AWS Glacier ストレージ サービスに相当します。 これは、少なくとも 180 日格納され、数時間の取得待ち時間が許容される、アクセス頻度が非常に少ないデータを対象としています。 

アクセス頻度が少ないが、アクセス時にすぐに使用できる必要があるデータについては、[Azure クール BLOB ストレージ層](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier)のストレージを使用すると、標準の BLOB ストレージよりもコストを抑えることができます。 このストレージ層は、AWS S3 (Infrequent Access ストレージ サービス) に相当します。

#### <a name="see-also"></a>関連項目

-   [Microsoft Azure Storage のパフォーマンスとスケーラビリティに対するチェック リスト](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Azure Storage セキュリティ ガイド](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [パターンとプラクティス: コンテンツ配信ネットワーク (CDN) のガイダンス](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>ネットワーク

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing、Azure Load Balancer、および Azure Application Gateway

Azure には、Elastic Load Balancing に相当する 2 つのサービスがあります。

-   [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - AWS Classic Load Balancer と同じ機能を提供し、複数の VM のトラフィックをネットワーク レベルで分散します。 フェールオーバー機能も提供します。

-   [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - AWS Application Load Balancer に相当するアプリケーション レベルのルール ベースのルーティングを提供します。

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53、Azure DNS、および Azure Traffic Manager

AWS の Route 53 では、DNS 名管理と DNS レベルのトラフィック ルーティング サービスと、フェールオーバー サービスの両方を提供します。 Azure では、これは 2 つのサービスで処理されます。

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/) は、ドメインと DNS の管理を提供します。

-   [Traffic Manager][traffic-manager] は、DNS レベルのトラフィック ルーティング、負荷分散、およびフェールオーバー機能を提供します。

#### <a name="direct-connect-and-azure-expressroute"></a>Direct Connect と Azure ExpressRoute

Azure では、[ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) サービスを介して、同様のサイト間専用接続を提供します。 ExpressRoute では、専用のプライベート ネットワーク接続を使用して、ローカル ネットワークを Azure のリソースに直接接続できます。 Azure では、従来の[サイト間 VPN 接続](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)も低コストで提供しています。

#### <a name="see-also"></a>関連項目

-   [Azure Portal を使用した仮想ネットワークの作成](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Azure Virtual Network の計画と設計](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Azure のネットワーク セキュリティに関するベスト プラクティス](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>データベース サービス

#### <a name="rds-and-azure-relational-database-services"></a>RDS と Azure リレーショナル データベース サービス

Azure には、AWS の Relational Database Service (RDS) に相当するリレーショナル データベース サービスが何種類かあります。

-   [SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

[SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/)、[Oracle](https://azure.microsoft.com/campaigns/oracle/)、[MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) などの他のデータベース エンジンは、Azure VM インスタンスを使用してデプロイできます。

AWS RDS のコストは、インスタンスが使用するハードウェア リソース (CPU、RAM、ストレージ、ネットワーク帯域幅など) の量によって決まります。 Azure データベース サービスでは、コストは、データベースのサイズ、同時接続数、およびスループット レベルによって決まります。

#### <a name="see-also"></a>関連項目

-   [Azure SQL Database のチュートリアル](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Azure ポータルを使用して Azure SQL Database の geo レプリケーションを構成する](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [Cosmos DB の概要: NoSQL JSON Database](https://azure.microsoft.com/documentation/articles/documentdb-introduction/)

-   [Node.js から Azure Table Storage を使用する方法](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>セキュリティと ID

#### <a name="directory-service-and-azure-active-directory"></a>ディレクトリ サービスと Azure Active Directory

Azure では、ディレクトリ サービスを次の製品に分割しています。

-   [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - クラウドベースのディレクトリおよび ID 管理サービスです。

-   [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - パートナーによって管理される ID から会社のアプリケーションにアクセスできるようにします。

-   [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - コンシューマー向けアプリケーションに対するシングル サインオンおよびユーザー管理をサポートするサービスです。

-   [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - ドメイン コントローラー サービスをホストし、Active Directory と互換性のあるドメイン参加とユーザー管理機能を提供できます。

#### <a name="web-application-firewall"></a>Web アプリケーション ファイアウォール

[Application Gateway Web アプリケーション ファイアウォール](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)に加え、[Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) などのサード パーティ製の [Web アプリケーション ファイアウォールも使用できます](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)。

#### <a name="see-also"></a>関連項目

-   [Microsoft Azure セキュリティの概要](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Azure の ID 管理とアクセス制御セキュリティのベスト プラクティス](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>アプリケーションとメッセージング サービス

#### <a name="simple-email-service"></a>Simple Email Service

AWS では、通知、トランザクション、またはマーケティングに関する電子メールを送信するための Simple Email Service (SES) を提供しています。 Azure では、[Sendgrid](https://sendgrid.com/partners/azure/) などのサード パーティ製のソリューションによって電子メール サービスを提供します。

#### <a name="simple-queueing-service"></a>Simple Queueing Service

AWS Simple Queueing Service (SQS) は、AWS プラットフォーム内のアプリケーション、サービス、およびデバイスを接続するためのメッセージング システムを提供します。 Azure には、同様の機能を提供する 2 つのサービスがあります。

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Azure プラットフォーム内のアプリケーション コンポーネント間で通信できるようにするクラウド メッセージング サービスです。

-   [Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - アプリケーション、サービス、およびデバイスを接続するためのより堅牢なメッセージング システムです。 Service Bus は、関連する [Service Bus Relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it) を使用して、リモートでホストされているアプリケーションとサービスにも接続できます。

#### <a name="device-farm"></a>Device Farm

AWS Device Farm は、デバイス間のテスト サービスを提供します。 Azure では、[Xamarin Test Cloud](https://www.xamarin.com/test-cloud) によって、モバイル デバイス用のデバイス間フロントエンド テストを提供しています。

フロントエンド テストだけでなく、[Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) によって、Linux 環境と Windows 環境向けのバックエンド テスト リソースも提供しています。

#### <a name="see-also"></a>関連項目

-   [Node.js から Queue ストレージを使用する方法](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [Service Bus キューの使用方法](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a>分析とビッグ データ

[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) は、大量のデータを取得、整理、分析、および視覚化することを目的とする Azure の製品とサービスのパッケージです。 Cortana スイートは、次のサービスで構成されています。

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Hadoop、Spark、Storm、または HBase を含む、管理された Apache ディストリビューションです。

-   [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - データ オーケストレーションおよびデータ パイプライン機能を提供します。

-   [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 大規模リレーショナル データ ストレージです。

-   [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - ビッグ データ分析ワークロード用に最適化された大規模ストレージです。

-   [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - データの予測分析を構築して適用するために使用されます。

-   [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - リアルタイムのデータ分析を行います。

-   [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - Data Lake Store と連携するように最適化された大規模分析サービスです。

-   [PowerBI](https://powerbi.microsoft.com/) - データの視覚化を促進するために使用されます。

#### <a name="see-also"></a>関連項目

-   [Cortana Intelligence ギャラリー](https://gallery.cortanaintelligence.com/)

-   [Microsoft ビッグ データ ソリューションを理解する](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Azure Data Lake と Azure の HDInsight ブログ](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>モノのインターネット

#### <a name="see-also"></a>関連項目

-   [Azure IoT Hub を使ってみる](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [IoT Hub と Event Hubs の比較](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>モバイル サービス

#### <a name="notifications"></a>通知

Notification Hubs は SMS または電子メール メッセージの送信をサポートしていないため、これらの配信の種類では、サード パーティが提供するサービスが必要です。

#### <a name="see-also"></a>関連項目

-   [Android アプリの作成](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Azure Mobile Apps での認証および承認](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Azure Notification Hubs によるプッシュ通知の送信](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>管理と監視

#### <a name="see-also"></a>関連項目
-   [監視と診断のガイダンス](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Azure Resource Manager テンプレートを作成するためのベスト プラクティス](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Azure Resource Manager のクイックスタート テンプレート](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>次の手順

-   [AWS と Azure のサービスの完全比較マトリックス](https://aka.ms/azure4aws-services)

-   [対話型 Azure プラットフォームの全体像](http://azureplatform.azurewebsites.net/)

-   [Azure を使ってみる](https://azure.microsoft.com/get-started/)

-   [Azure ソリューション アーキテクチャ](https://azure.microsoft.com/solutions/architecture/)

-   [Azure リファレンス アーキテクチャ](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [パターンとプラクティス: Azure のガイダンス](https://azure.microsoft.com/documentation/articles/guidance/)

-   [無料オンライン コース: AWS エキスパート向け Microsoft Azure](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/