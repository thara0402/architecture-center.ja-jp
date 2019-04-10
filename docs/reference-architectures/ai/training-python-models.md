---
title: Azure での Python scikit-learn モデルのトレーニング
description: この参照アーキテクチャでは、scikit-learn Python モデルのハイパーパラメーター (トレーニング パラメーター) の調整の推奨される方法が示されています。
author: njray
ms.date: 03/07/19
ms.custom: azcat-ai
ms.openlocfilehash: 23689ff7423b681c6cf5aeb713920a98f658044f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242243"
---
# <a name="training-of-python-scikit-learn-models-on-azure"></a>Azure での Python scikit-learn モデルのトレーニング

この参照アーキテクチャでは、[scikit-learn][scikit] Python モデルのハイパーパラメーター (トレーニング パラメーター) の調整の推奨される方法が示されています。 このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。

![アーキテクチャ ダイアグラム][0]

**シナリオ:** ここで取り上げる問題は、よく寄せられる質問 (FAQ) の一致です。 このシナリオでは、JavaScript としてタグ付けされた元の質問、重複する質問、およびその回答を含む Stack Overflow 質問データのサブセットが使用されます。 重複する質問が元の質問のいずれかと一致する可能性を予測するように scikit-learn パイプラインをトレーニングします。

このシナリオでの処理には、次のステップが含まれます。

1. トレーニング Python スクリプトが、[Azure Machine Learning service][aml] に送信されます。

1. 各ノード上に作成された Docker コンテナー内で、スクリプトが実行されます。

1. そのスクリプトで、[Azure Storage][storage] からトレーニング データとテスト データが取得されます。

1. スクリプトにより、トレーニング パラメーターの組み合わせを使用してトレーニング データからモデルが学習されます。

1. モデルのパフォーマンスがテスト データで評価されて、Azure Storage に書き込まれます。

1. 最もパフォーマンスのよいモデルが、Azure Machine Learning service に登録されます。

GPU での[ディープ ラーニング モデル][training-deep-learning]のトレーニングに関する考慮事項も参照してください。

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、ニーズに従ってリソースをスケーリングする複数の Azure クラウド サービスで構成されます。 サービスおよびこのソリューションでのその役割を、以下で説明します。

[Microsoft Data Science Virtual Machine][dsvm] (DSVM) は、データ分析と機械学習に使用されるツールで構成されている VM イメージです。 Windows Server バージョンと Linux バージョンの両方を使用できます。 このデプロイでは、Ubuntu 16.04 LTS 上の DSVM の Linux エディションを使用します。

[Azure Machine Learning service][aml] は、クラウド スケールで機械学習モデルをトレーニング、デプロイ、自動化、管理するために使用されます。 このアーキテクチャでは、以下で説明する Azure リソースの使用と割り当てを管理するために使用されます。

[Azure Machine Learning コンピューティング][aml-compute]は、Azure 内で大規模な機械学習モデルと AI モデルのトレーニングとテストを行うために使用されるリソースです。 このシナリオでの[コンピューティング先][target]は、自動[スケーリング][scaling] オプションに基づいてオンデマンドで割り当てられるノードのクラスターです。 各ノードは、特定の[ハイパーパラメーター][hyperparameter] セットに対するトレーニング ジョブが実行される VM です。

[Azure Container Registry][acr] には、あらゆる種類の Docker コンテナー デプロイのイメージが格納されます。 これらのコンテナーは、各ノード上に作成されて、トレーニング Python スクリプトの実行に使用されます。 Machine Learning コンピューティング クラスターで使用されるイメージは、ローカル実行とハイパーパラメーター調整ノートブックで Machine Learning service によって作成された後、Container Registry にプッシュされます。

[Azure BLOB][blob] ストレージは、トレーニング Python スクリプトによって使用されるトレーニングおよびテスト データ セットを Machine Learning service から受け取ります。 ストレージは、Machine Learning コンピューティング クラスターの各ノードに仮想ドライブとしてマウントされます。 

## <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

[ハイパーパラメーター][hyperparameter]の各セットが、Machine Learning [コンピューティング先][target]の 1 つのノードで実行されます。 このアーキテクチャでは各ノードは Standard D4 v2 VM であり、4 つのコアを備えています。 このアーキテクチャでは、機械学習に対して [LightGBM][lightgbm] 分類子 (勾配ブースティング フレームワーク) が使用されます。 このソフトウェアは 4 つのコアすべてで同時に実行できるので、各実行が最大で 4 倍に高速化されます。 それにより、ハイパーパラメーター調整の実行全体にかかる時間は、それぞれがコアを 1 つしか備えていない Standard D1 v2 VM に基づく Machine Learning コンピューティング先で実行した場合より、最大で 4 分の 1 に短縮されます。

Machine Learning コンピューティング ノードの最大数は、合計実行時間に影響します。 ノードの推奨される最小数は 0 です。 この設定では、ジョブの開始にかかる時間に、少なくとも 1 つのノードをクラスターに自動スケーリングするのに必要な何分かが含まれます。 ただし、ハイパーパラメーター調整の実行時間が短い場合は、ジョブのスケールアップによってオーバーヘッドが増えます。 たとえば、ジョブを 5 分以下で実行できても、1 ノードへのスケールアップにさらに 5 分かかる可能性があります。 このような場合は、最小のノード数を 1 に設定すると時間を節約できますが、コストは増加します。

## <a name="monitoring-and-logging-considerations"></a>監視とログ記録に関する考慮事項

実行の進行状況の監視に使用するため、[HyperDrive][hyperparameter] 実行構成を送信して実行オブジェクトを返します。

### <a name="rundetails-jupyter-widget"></a>RunDetails Jupyter ウィジェット

キューに入っているとき、および子ジョブを実行しているときに、その進行状況を簡単に監視するには、RunDetails [Jupyter ウィジェット][jupyter]で実行オブジェクトを使用します。 ログに記録される主要な統計情報の値もリアルタイムで表示されます。

### <a name="azure-portal"></a>Azure ポータル

実行オブジェクトを出力すると、次のように Azure portal での実行のページへのリンクが表示されます。

![実行オブジェクト][1]

このページを使用して、実行とその子実行の状態を監視できます。 各子実行には、実行が済んだトレーニング スクリプトの出力を含むドライバー ログがあります。

## <a name="cost-considerations"></a>コストに関する考慮事項

ハイパーパラメーター調整実行のコストは、Machine Learning コンピューティング VM のサイズ、優先順位の低いノードを使うかどうか、クラスターで許容される最大ノード数の選択に比例します。

クラスターが使用されていないときの継続的なコストは、クラスターで必要なノードの最小数に依存します。 クラスターの自動スケーリングが有効なシステムでは、ノードはジョブの数に合わせて許容される最大数まで自動的に追加され、不要になると要求されている最小数まで削除されます。 クラスターが 0 ノードまで自動的にスケールダウンできる場合は、使われていないときはコストがかかりません。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

### <a name="restrict-access-to-azure-blob-storage"></a>Azure Blob Storage へのアクセスの制限

このアーキテクチャでは、[ストレージ アカウント キー][storage-security]を使用して Blob ストレージにアクセスします。 さらに制御と保護を強化するには、共有アクセス署名 (SAS) を代わりに使用することを検討します。 これは、ストレージ内のオブジェクトへの制限付きアクセスを付与し、アカウント キーをハード コーディングしたり、それをプレーンテキストで保存したりする必要はありません。 また、SAS を使用すると、ストレージ アカウントに適切なガバナンスがあること、およびアクセスがそれを必要とするユーザーだけに付与されることを保証することができます。

ストレージ キーはワークロードのすべての入出力データに対するフル アクセス権を与えるため、データの機密性がさらに高いシナリオでは、すべてのストレージ キーが保護されるようにします。

### <a name="encrypt-data-at-rest-and-in-motion"></a>保存時および稼働時のデータの暗号化

機密データを使用するシナリオでは、保存データつまりストレージ内にあるデータを暗号化します。 データがある場所から次の場所に移動されるたびに、SSL を使用してデータ転送をセキュリティ保護します。 詳細については、「[Azure Storage セキュリティ ガイド][storage-security]」をご覧ください。

### <a name="secure-data-in-a-virtual-network"></a>仮想ネットワーク内のデータのセキュリティ保護

運用環境にデプロイする場合、指定した仮想ネットワークのサブネットにクラスターをデプロイすることを検討します。 これにより、クラスター内のコンピューティング ノードは、他の仮想マシンやオンプレミスのネットワークと安全に通信できます。 また、BLOB ストレージを使用する[サービス エンドポイント][endpoints]を使って、仮想ネットワークからのアクセスを許可したり、Azure Machine Learning service を備えた仮想ネットワークの内部で単一ノード NFS を使用したりすることも可能です。

## <a name="deployment"></a>Deployment

このアーキテクチャの参照実装をデプロイするには、[GitHub][github] リポジトリの手順に従ってください。

[0]: ./_images//training-python-models.png
[1]: ./_images/run-object.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml]: /azure/machine-learning/service/overview-what-is-azure-ml
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[blob]: /azure/storage/blobs/storage-blobs-introduction
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[github]: https://github.com/Microsoft/MLHyperparameterTuning
[hyperparameter]: /azure/machine-learning/service/how-to-tune-hyperparameters
[jupyter]: http://jupyter.org/widgets
[lightgbm]: https://github.com/Microsoft/LightGBM
[scaling]: /azure/virtual-machine-scale-sets/overview
[scikit]: https://pypi.org/project/scikit-learn/
[storage]: /azure/storage/common/storage-introduction
[storage-security]: /azure/storage/common/storage-security-guide
[target]: /azure/machine-learning/service/how-to-auto-train-remote
[training-deep-learning]: /azure/architecture/reference-architectures/ai/training-deep-learning