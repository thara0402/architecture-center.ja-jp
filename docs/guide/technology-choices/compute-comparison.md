---
title: Azure コンピューティング サービスを選択するための条件
titleSuffix: Azure Application Architecture Guide
description: 複数の軸間で Azure コンピューティング サービスを比較します。
author: MikeWasson
ms.date: 08/08/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2b6b9b941bf7a3c0136b71ecb65bfe4b4a59e07b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484858"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Azure コンピューティング サービスを選択するための条件

"*コンピューティング*" という用語は、アプリケーションが実行するコンピューティング リソースのホスティング モデルを指します。 以下の表では、複数の軸間で Azure コンピューティング サービスを比較しています。 アプリケーションのコンピューティング オプションを選択するときに、これらの表をご覧ください。

## <a name="hosting-model"></a>ホスティング モデル

<!-- markdownlint-disable MD033 -->

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| アプリケーションの構成 | 非依存 | アプリケーション、コンテナー | サービス、ゲスト実行可能ファイル、コンテナー | Functions | Containers | Containers | スケジュールされたジョブ  |
| 密度 | 非依存 | App Service プランによる、インスタンスごとの複数のアプリ | VM ごとに複数のサービス | サーバーレス <a href="#note1"><sup>1</sup></a> | ノードごとに複数のコンテナー |専用インスタンスなし | VM ごとに複数のアプリ |
| 最小ノード数 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | サーバーレス <a href="#note1"><sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | 専用ノードなし | 1 <a href="#note4"><sup>4</sup></a> |
| 状態管理 | ステートレスまたはステートフル | ステートレス | ステートレスまたはステートフル | ステートレス | ステートレスまたはステートフル | ステートレス | ステートレス |
| Web ホスティング | 非依存 | 組み込み | 非依存 | 適用不可 | 非依存 | 非依存 | いいえ  |
| 専用 VNet にデプロイできるかどうか | サポートされています | サポートされています <a href="#note5"><sup>5</sup></a> | サポートされています | サポートされています <a href="#note5"><sup>5</sup></a> | [サポートされています](/azure/aks/networking-overview) | サポートされていません | サポートされています |
| ハイブリッド接続 | サポートされています | サポートされています <a href="#note6"><sup>6</sup></a>  | サポートされています | サポートされています <a href="#node7"><sup>7</sup></a> | サポートされています | サポートされていません | サポートされています |

メモ

1. <span id="note1">従量課金プランを使用している場合。App Service プランを使用している場合、関数は、App Service プランに割り当てられた VM 上で実行されます。[Azure Functions の適切なサービス プランを選択する][function-plans]に関する記事をご覧ください。</span>
2. <span id="note2">2 つ以上のインスタンスによる、より高い SLA。</span>
3. <span id="note3">運用環境の場合に推奨されます。</span>
4. <span id="note4">ジョブの完了後、0 にスケールダウンできます。</span>
5. <span id="note5">App Service Environment (ASE) が必要です。</span>
6. <span id="note6">[Azure App Service のハイブリッド接続][app-service-hybrid]を使用します。</span>
7. <span id="note7">App Service プランが必要です。</span>

## <a name="devops"></a>DevOps

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| ローカル デバッグ | 非依存 | IIS Express、その他 <a href="#note1b"><sup>1</sup></a> | ローカル ノードのクラスター | Visual Studio または Azure Functions CLI | Minikube、その他 | ローカルのコンテナー ランタイム | サポートされていません |
| プログラミング モデル | 非依存 | Web および API アプリケーション、バックグラウンド タスク用の Web ジョブ | ゲスト実行可能ファイル、サービス モデル、アクター モデル、コンテナー | トリガーを持つ関数 | 非依存 | 非依存 | コマンド ライン アプリケーション |
| アプリケーションの更新 | 組み込みのサポートなし | デプロイ スロット | ローリング アップグレード (サービスあたり) | デプロイ スロット | ローリング アップデート | 適用不可 |

メモ

1. <span id="note1b">オプションには、ASP.NET または node.js (iisnode) の IIS Express、PHP Web サーバー、Azure Toolkit for IntelliJ、Azure Toolkit for Eclipse などがあります。App Service では、デプロイ済みの Web アプリのリモート デバッグもサポートしています。</span>
2. <span id="note2b">[Resource Manager のプロバイダー、リージョン、API のバージョン、およびスキーマ][resource-manager-supported-services]に関する記事をご覧ください。</span>

## <a name="scalability"></a>スケーラビリティ

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 自動スケール | VM スケール セット | 組み込みのサービス | VM スケール セット | 組み込みのサービス | サポートされていません | サポートされていません | 該当なし |
| Load Balancer | Azure Load Balancer | 統合 | Azure Load Balancer | 統合 | 統合 |  組み込みのサポートなし | Azure Load Balancer |
| スケールの制限<a href="#note1c"><sup>1</sup></a> | プラットフォーム イメージ:VMSS あたり 1000 個のノード、カスタム イメージ:VMSS あたり 100 個のノード | 20 個のインスタンス、App Service Environment で 100 | VMSS あたり 100 個のノード | Function App あたり 200 個のインスタンス | クラスターあたり 100 ノード (既定の制限) |サブスクリプションあたり 20 コンテナー グループ (既定の制限)。 | 20 個のコアの制限 (既定の制限)。 |

メモ

1. <span id="note1c">「[Azure サブスクリプションとサービスの制限、クォータ、制約](/azure/azure-subscription-service-limits)</span>」をご覧ください。

## <a name="availability"></a>可用性

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [VIrtual Machines の SLA][sla-vm] | [App Service の SLA][sla-app-service] | [Service Fabric の SLA][sla-sf] | [Functions の SLA][sla-functions] | [AKS の SLA][sla-acs] | [Container Instances の SLA](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Azure Batch の SLA][sla-batch] |
| 複数リージョンのフェールオーバー | Traffic Manager | Traffic Manager | Traffic manager、複数リージョンのクラスター | サポートされていません | Traffic Manager | サポートされていません | サポートされていません |

## <a name="other"></a>その他

| 条件 | Virtual Machines | App Service | Service Fabric | Azure Functions | Azure Kubernetes Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | VM で構成済み | サポートされています | サポートされています  | サポートされています | [イングレス コントローラー](/azure/aks/ingress) | [サイドカー](../../patterns/sidecar.md) コンテナーを使用 | サポートされています |
| コスト | [Windows][cost-windows-vm]、[Linux][cost-linux-vm] | [App Service の価格][cost-app-service] | [Service Fabric の価格][cost-service-fabric] | [Azure Functions の価格][cost-functions] | [AKS の価格][cost-acs] | [Container Instances の価格](https://azure.microsoft.com/pricing/details/container-instances/) | [Azure Batch の価格][cost-batch]
| 適切なアーキテクチャ スタイル | [n 層][n-tier]、[ビッグ コンピューティング][big-compute] (HPC) | [Web キューワーカー][w-q-w]、[N 層][n-tier] | [マイクロサービス][microservices]、[イベント駆動型アーキテクチャ][event-driven] | [マイクロサービス][microservices]、[イベント駆動型アーキテクチャ][event-driven] | [マイクロサービス][microservices]、[イベント駆動型アーキテクチャ][event-driven] | [マイクロサービス][microservices]、タスクの自動化、バッチ ジョブ  | [ビッグ コンピューティング][big-compute] (HPC) |

<!-- markdownlint-enable MD033 -->

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
