---
title: Azure コンピューティング サービスを選択するための条件
description: 複数の軸間で Azure コンピューティング サービスを比較します
author: MikeWasson
layout: LandingPage
ms.date: 04/21/2018
ms.openlocfilehash: ff90ec41c56ae0ecb81bc82128f02fd06d02cb32
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/23/2018
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Azure コンピューティング サービスを選択するための条件

"*コンピューティング*" という用語は、アプリケーションが実行するコンピューティング リソースのホスティング モデルを指します。 以下の表では、複数の軸間で Azure コンピューティング サービスを比較しています。 アプリケーションのコンピューティング オプションを選択するときに、これらの表をご覧ください。

## <a name="hosting-model"></a>ホスティング モデル

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| アプリケーションの構成 | 非依存 | [アプリケーション] | サービス、ゲスト実行可能ファイル、コンテナー | Functions | Containers | Containers | スケジュールされたジョブ  |
| 密度 | 非依存 | アプリの計画を介した、インスタンスごとの複数のアプリ | VM ごとに複数のサービス | 専用インスタンスなし <a href="#note1"><sup>1</sup></a> | VM ごとに複数のコンテナー |専用インスタンスなし | VM ごとに複数のアプリ |
| 最小ノード数 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | 専用ノードなし <a href="#note1"><sup>1</sup></a> | 3 | 専用ノードなし | 1 <a href="#note4"><sup>4</sup></a> |
| 状態管理 | ステートレスまたはステートフル | ステートレス | ステートレスまたはステートフル | ステートレス | ステートレスまたはステートフル | ステートレス | ステートレス |
| Web ホスティング | 非依存 | 組み込み | 非依存 | 適用不可 | 非依存 | 非依存 | いいえ  |
| OS | Windows、Linux | Windows、Linux  | Windows、Linux | 適用不可 | Windows (プレビュー)、Linux | Windows、Linux | Windows、Linux |
| 専用 VNet にデプロイできるかどうか | サポートされています | サポートされています <a href="#note5"><sup>5</sup></a> | サポートされています | サポートされていません | サポートされています | サポートされていません | サポートされています |
| ハイブリッド接続 | サポートされています | サポートされています <a href="#note1"><sup>6</sup></a>  | サポートされています | サポートされていません | サポートされています | サポートされていません | サポートされています |

メモ

1. <span id="note1">従量課金プランを使用している場合。 App Service プランを使用している場合、関数は、App Service プランに割り当てられた VM 上で実行されます。 [Azure Functions の適切なサービス プランを選択する][function-plans]に関する記事をご覧ください。</a>
2. <span id="note2">2 つ以上のインスタンスによる、より高い SLA。</a>
3. <span id="note3">運用環境の場合。</a>
4. <span id="note4">ジョブの完了後、0 にスケールダウンできます。</a>
5. <span id="note5">App Service Environment (ASE) が必要です。</a>
6. <span id="note7">ASE または BizTalk ハイブリッド接続が必要です。</a>

## <a name="devops"></a>DevOps

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| ローカル デバッグ | 非依存 | IIS Express、その他 <a href="#note1b"><sup>1</sup></a> | ローカル ノードのクラスター | Azure Functions CLI | ローカルのコンテナー ランタイム | ローカルのコンテナー ランタイム | サポートされていません |
| プログラミング モデル | 非依存 | Web アプリケーション、バック グラウンド タスク用の Web ジョブ | ゲスト実行可能ファイル、サービス モデル、アクター モデル、コンテナー | トリガーを持つ関数 | 非依存 | 非依存 | コマンド ライン アプリケーション |
| アプリケーションの更新 | 組み込みのサポートなし | デプロイ スロット | ローリング アップグレード (サービスあたり) | 組み込みのサポートなし | オーケストレーターに依存。 ほとんどがローリング アップデートをサポート | コンテナー イメージの更新 | 適用不可 |

メモ

1. <span id="note1b">オプションには、ASP.NET または node.js (iisnode) の IIS Express、PHP Web サーバー、Azure Toolkit for IntelliJ、Azure Toolkit for Eclipse などがあります。 App Service では、デプロイ済みの Web アプリのリモート デバッグもサポートしています。</a>
2. <span id="note2b">[リソース マネージャーのプロバイダー、リージョン、API のバージョン、およびスキーマ][resource-manager-supported-services]に関する記事をご覧ください。 


## <a name="scalability"></a>スケーラビリティ

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 自動スケール | VM スケール セット | 組み込みのサービス | VM スケール セット | 組み込みのサービス | サポートされていません | サポートされていません | 該当なし |
| Load Balancer | Azure Load Balancer | 統合 | Azure Load Balancer | 統合 | Azure Load Balancer |  組み込みのサポートなし | Azure Load Balancer |
| スケールの制限 | プラットフォーム イメージ: VMSS あたり 1000 個のノード、カスタム イメージ: VMSS あたり 100 個のノード | 20 個のインスタンス、App Service Environment で 50 | VMSS あたり 100 個のノード | 無制限 <a href="#note1c"><sup>1</sup></a> | 100 |サブスクリプションあたり 20 コンテナー グループ <a href="#note2c"><sup>2</sup></a> | 既定で 20 個のコアの上限。 増やすには、カスタマー サービスに問い合わせてください。 |

メモ

1. <span id="note1c">従量課金プランを使用している場合。 App Service プランを使用している場合は、App Service のスケール制限が適用されます。 [Azure Functions の適切なサービス プランを選択する][function-plans]に関する記事をご覧ください。</a>
2. <span id="note2c">「[Azure Container Instances のクォータとリージョンの可用性](/azure/container-instances/container-instances-quotas)」を参照してください。</a>


## <a name="availability"></a>可用性

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [VIrtual Machines の SLA][sla-vm] | [App Service の SLA][sla-app-service] | [Service Fabric の SLA][sla-sf] | [Functions の SLA][sla-functions] | [Azure Container Service の SLA][sla-acs] | [Container Instances の SLA](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Azure Batch の SLA][sla-batch] |
| 複数リージョンのフェールオーバー | Traffic Manager | Traffic Manager | Traffic manager、複数リージョンのクラスター | サポートされていません  | Traffic Manager | サポートされていません | サポートされていません |

## <a name="other"></a>その他

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | VM で構成済み | サポートされています | サポートされています  | サポートされています | VM で構成済み | サポートされていません | サポートされています |
| コスト | [Windows][cost-windows-vm]、[Linux][cost-linux-vm] | [App Service の価格][cost-app-service] | [Service Fabric の価格][cost-service-fabric] | [Azure Functions の価格][cost-functions] | [Azure Container Service の価格][cost-acs] | [Container Instances の価格](https://azure.microsoft.com/pricing/details/container-instances/) | [Azure Batch の価格][cost-batch]
| 適切なアーキテクチャ スタイル | n 層、ビッグ コンピューティング (HPC) | Web キューワーカー | マイクロサービス、イベント駆動型アーキテクチャ (EDA) | マイクロサービス、EDA | マイクロサービス、EDA | マイクロサービス、タスクの自動化、バッチ ジョブ  | ビッグ コンピューティング |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services