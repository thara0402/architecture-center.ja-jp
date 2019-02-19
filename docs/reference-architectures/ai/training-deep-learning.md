---
title: Azure でのディープ ラーニング モデルの分散トレーニング
description: この参照アーキテクチャでは、Azure Batch AI を使用して、GPU 対応 VM のクラスター間でディープ ラーニング モデルの分散トレーニングを実施する方法を示します。
author: njray
ms.date: 01/14/19
ms.custom: azcat-ai
ms.openlocfilehash: 800defeb851f5a31dc730038c3699e1a3d54b923
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307783"
---
# <a name="distributed-training-of-deep-learning-models-on-azure"></a>Azure でのディープ ラーニング モデルの分散トレーニング

この参照アーキテクチャでは、GPU 対応 VM のクラスター間でディープ ラーニング モデルの分散トレーニングを実施する方法を示します。 シナリオは画像の分類ですが、ソリューションは、セグメント化やオブジェクト検出など、その他のディープ ラーニング シナリオに一般化することが可能です。

このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。

![分散型のディープ ラーニングのアーキテクチャ][0]

**シナリオ**: イメージの分類は、コンピューター ビジョンにおいて広く適用されている技法であり、多くの場合、畳み込みニューラル ネットワーク (CNN) をトレーニングすることで対応されます。 大規模なデータセットを備える特に大きなモデルでは、単一の GPU に対するトレーニング プロセスに数週間または数か月かかる場合があります。 状況によっては、モデルが非常に大きいために、GPU 上に妥当なバッチ サイズを適合させることができない場合もあります。 このような状況に分散トレーニングを使用すると、トレーニング時間を短縮できます。

今回の特定のシナリオでは、[Imagenet データセット][imagenet]および合成データに対して、[Horovod][horovod] を使用して [ResNet50 CNN モデル][resnet]がトレーニングされます。 リファレンス実装では、最も一般的な 3 つのディープ ラーニング フレームワークを使用して、このタスクを完了する方法を示します: TensorFlow、Keras、および PyTorch です。

分散型の方式でディープ ラーニング モデルをトレーニングするには、同期または非同期の更新に基づくデータ並列およびモデル並列の手法を含め、複数の方法があります。 現在、最も一般的なシナリオは、同期更新によるデータ並列です。 この手法は、実装が最も簡単であり、ほとんどのユース ケースに対応します。

同期更新によるデータ並列の分散トレーニングでは、モデルは *n* 個のハードウェア デバイスにわたってレプリケートされます。 トレーニング サンプルのミニバッチは、*n* 個のマイクロ バッチに分割されます。 各デバイスは、1 つのマイクロバッチに対して順方向および逆方向の受け渡しを実行します。 デバイスはプロセスを終了するきに、更新内容を他のデバイスと共有します。 これらの値は、ミニバッチ全体の更新後の重みを計算するために使用され、重みはモデル間で同期されます。 このシナリオについては、[GitHub][github] リポジトリの中で説明されています。

![データ並列の分散トレーニング][1]

このアーキテクチャは、モデル並列の非同期更新にも使用できます。 モデル並列の分散トレーニングでは、*n* 個のハードウェア デバイスにわたってモデルが分割され、各デバイスがモデルの一部を保持している状態になります。 最もシンプルな実装では、デバイスごとにネットワークの 1 つの層を保持することができ、順方向および逆方向の受け渡しの中で、デバイス間に情報が渡されます。 比較的規模の大きいニューラル ネットワークの場合、この方法によるトレーニングは可能ですが、デバイスは順方向または逆方向の受け渡しを互いに完了するまで常に待機するため、パフォーマンス コストがかかります。 一部の高度な技法では、合成勾配を使用して、この問題の部分的な軽減を試みています。

トレーニングの手順は次のとおりです。

1. クラスター上で実行されるスクリプトを作成し、お使いのモデルをトレーニングしてから、スクリプトをファイル ストレージに転送する。

1. Blob Storage にデータを書き込む。

1. Batch AI ファイル サーバーを作成して、Blob Storage からこのサーバーにデータをダウンロードする。

1. 各ディープ ラーニング フレームワーク用の Docker コンテナーを作成し、コンテナー レジストリ (Docker Hub) に転送する。

1. Batch AI ファイル サーバーもマウントする Batch AI プールを作成する。

1. ジョブを送信する。 それぞれが、適切な Docker イメージとスクリプトをプルします。

1. ジョブが完了した後に、すべての結果を Files ストレージに書き込む。

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、次のコンポーネントで構成されます。

**[Azure Batch AI][batch-ai]** は、ニーズに応じてリソースをスケールアップおよびスケールダウンすることで、このアーキテクチャにおける中心的な役割を担います。 Batch AI は、VM のクラスターのプロビジョニングおよび管理、ジョブのスケジュール設定、結果の収集、リソースのスケーリング、エラーの処理、および適切なストレージの作成に利用できるサービスです。 ディープ ラーニング ワークロードでの GPU 対応 VM をサポートしています。 Batch AI には、Python SDK およびコマンド ライン インターフェイス (CLI) が利用可能です。

> [!NOTE]
> Azure Batch AI サービスは 2019 年 3 月に終了する予定であり、このサービスの大規模トレーニングとスコアリングの機能は現在、[Azure Machine Learning Service][amls] において利用可能になっています。 この参照アーキテクチャは近日中に Machine Learning を使用するように改定されます。そして、[Azure Machine Learning コンピューティング][aml-compute]という、機械学習モデルのトレーニング、デプロイ、およびスコアリングのためのマネージド コンピューティング先が提供されます。

**[Blob ストレージ][azure-blob]** は、データをステージングするために使用されます。 このデータはトレーニング中に、Batch AI ファイル サーバーにダウンロードされます。

**[Azure Files][files]** は、スクリプト、ログ、およびトレーニングからの最終結果を格納するために使用されます。 ファイル ストレージは、ログとスクリプトの格納に適していますが、パフォーマンスは Blob ストレージほどには優れていないので、データ量の多いタスクには使用できません。

**[Batch AI ファイル サーバー][batch-ai-files]** は、トレーニング データを格納するために、このアーキテクチャ内で使用される単一ノードの NFS 共有です。 Batch AI では NFS 共有を作成して、クラスター上にマウントします。 Batch AI ファイル サーバーは、必要なスループットでクラスターにデータを提供するためのお勧めの方法です。

**[Docker Hub][docker]** は、Batch AI がトレーニングの実行に利用する Docker イメージを格納するために使用されます。 Docker Hub は、使いやすく、Docker ユーザーに対する既定のイメージ リポジトリであるため、このアーキテクチャに選択されました。 [Azure Container Registry][acr] を使用することもできます。

## <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

Azure では、ディープ ラーニング モデルのトレーニングに適した 4 つの [GPU 対応 VM の種類][gpu]を提供しています。 種類は次に示すとおりで、価格および速度が低いものから高いものまであります。

| **Azure VM シリーズ** | **NVIDIA GPU** |
|---------------------|----------------|
| NC                  | K80            |
| ND                  | P40            |
| NCv2                | P100           |
| NCv3                | V100           |

Microsoft では、お使いのトレーニングをスケールアウトするよりも、まずはスケールアップすることを推奨しました。たとえば、K80 のクラスターを試す前に、単一の V100 を試してください。

次のグラフは、Batch AI 上で TensorFlow および Horovod を使用して実施された[ベンチマーク テスト][benchmark]に基づいて、異なる GPU の種類におけるパフォーマンスの差異を示しています。 グラフでは、異なる GPU の種類と MPI バージョンにおいて、各種のモデルにわたる 32 GPU クラスターのスループットを示しています。 モデルは、TensorFlow 1.9 で実装されました。

![GPU クラスター上での TensorFlow モデルに対するスループットの結果][2]

前記の表に示した各 VM シリーズには、InfiniBand を利用した構成が組み込まれています。 分散トレーニングを実行する場合、ノード間でのより高速な通信のために、InfiniBand 構成を使用してください。 また、InfiniBand は、InfiniBand を活用できるフレームワークにおいて、トレーニングのスケーリング効率を向上させます。 詳細については、Infiniband の[ベンチマークの比較][benchmark]に関する記事をご覧ください。

Batch AI では [blobfuse][blobfuse] アダプターを使用して Blob Storage をマウントできますが、必要なスループットを処理するにはパフォーマンスが不十分なので、分散トレーニングに Blob Storage をこのように使用することはお勧めしません。 代わりに、アーキテクチャの図に示すように、Batch AI ファイル サーバーにデータを移動してください。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

ネットワークのオーバーヘッドが原因で、分散トレーニングのスケーリング効率は常に 100 パーセント未満になります &mdash; デバイス間でモデル全体の同期がボトルネックになります。 そのため、分散トレーニングは、単一の GPU 上で適切なバッチ サイズを使用してトレーニングできない大規模なモデルや、シンプルな並列手法でモデルを分散することでは対処できない問題に、最も適しています。

分散トレーニングは、ハイパーパラメーター探索の実行には推奨されていません。 スケーリング効率がパフォーマンスに影響を及ぼし、複数のモデル構成を別個にトレーニングするよりも、分散手法の効率を低下させます。

スケーリング効率を向上させる 1 つの方法は、バッチ サイズを引き上げることです。 ただし、他のパラメーターを調整せずにバッチ サイズを引き上げると、モデルの最終的なパフォーマンスを低下させる可能性があるため、慎重に行う必要があります。

## <a name="storage-considerations"></a>ストレージに関する考慮事項

ディープ ラーニング モデルをトレーニングするときに、見落とされがちな側面は、データが格納される場所です。 ストレージが遅すぎて GPU の需要に対応できない場合、トレーニング パフォーマンスが低下する場合があります。

Batch AI では、多数のストレージ ソリューションをサポートしています。 Batch AI ファイル サーバーは使いやすさとパフォーマンスの最適なバランスを提供しているため、このアーキテクチャでは Batch AI ファイル サーバーを使用しています。 最適なパフォーマンスのためには、データをローカルに読み込みます。 しかし、すべてのノードにおいて Blob ストレージからデータをダウンロードする必要があり、ImageNet データセットの場合、このダウンロードに数時間かかる可能性があるため、これは煩雑になることがあります。 [Azure Premium Blob Storage][blob] (限定パブリック プレビュー) は、検討すべきもう 1 つのオプションです。

分散トレーニング用のデータ ストアとして Blob および File ストレージをマウントしないでください。 非常に低速であり、トレーニング パフォーマンスの妨げになります。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

### <a name="restrict-access-to-azure-blob-storage"></a>Azure Blob Storage へのアクセスの制限

このアーキテクチャでは、[ストレージ アカウント キー][security-guide]を使用して Blob ストレージにアクセスします。 さらに制御と保護を強化するには、共有アクセス署名 (SAS) を代わりに使用することを検討します。 これは、ストレージ内のオブジェクトへの制限付きアクセスを付与し、アカウント キーをハード コーディングしたり、それをプレーンテキストで保存したりする必要はありません。 また、SAS を使用すると、ストレージ アカウントに適切なガバナンスがあること、およびアクセスがそれを必要とするユーザーだけに付与されることを保証することができます。

ストレージ キーはワークロードのすべての入出力データに対するフル アクセス権を与えるため、データの機密性がさらに高いシナリオでは、すべてのストレージ キーが保護されるようにします。

### <a name="encrypt-data-at-rest-and-in-motion"></a>保存時および稼働時のデータの暗号化

機密データを使用するシナリオでは、保存データ &mdash; つまりストレージ内にあるデータを暗号化します。 データがある場所から次の場所に移動されるたびに、SSL を使用してデータ転送をセキュリティ保護します。 詳細については、「[Azure Storage セキュリティ ガイド][security-guide]」をご覧ください。

### <a name="secure-data-in-a-virtual-network"></a>仮想ネットワーク内のデータのセキュリティ保護

運用環境にデプロイする場合、指定した仮想ネットワークのサブネットに Batch AI クラスターをデプロイすることを検討します。 これにより、クラスター内のコンピューティング ノードは、他の仮想マシンやオンプレミスのネットワークと安全に通信できます。 また、Blob ストレージを利用した[サービス エンドポイント][endpoints]を使って、仮想ネットワークからのアクセスを許可したり、Batch AI を備えた仮想ネットワークの内部で単一ノード NFS を使用したりすることも可能です。

## <a name="monitoring-considerations"></a>監視に関する考慮事項

ジョブの実行中は、進行状況を監視し、想定どおりに動作していることを確認することが重要です。 ただし、アクティブなノードのクラスター全体を監視するのは困難な場合があります。

Batch AI ファイル サーバーは、Azure portal 経由または [Azure CLI][cli] および Python SDK 経由で管理できます。 クラスターの全体的な状態を把握するには、Azure portal 内で **[Batch AI]** に移動して、クラスター ノードの状態を調べます。 ノードが非アクティブになった場合、またはジョブが失敗した場合は、エラー ログが Blob ストレージに保存され、Azure portal の **[ジョブ]** の下からもアクセスできます。

ログを [Azure Application Insights][ai] に接続するか、または Batch AI クラスターとそのジョブの状態をポーリングする別個のプロセスを実行することで、監視を強化します。

Batch AI では、関連付けられている BLOB ストレージ アカウントに、すべての stdout/stderr を自動的に記録します。 [Azure Storage Explorer][storage-explorer] などのストレージ ナビゲーション ツールを使用すると、ログ ファイルをナビゲートするときのエクスペリエンスが簡単になります。

また、各ジョブのログをストリーミングすることも可能です。 このオプションの詳細については、[GitHub][github] 上の開発手順をご覧ください。

## <a name="deployment"></a>Deployment

このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。 そこに示された手順に従って、GPU 対応 VM のクラスター間でディープ ラーニング モデルの分散トレーニングを実施してください。

## <a name="next-steps"></a>次の手順

このアーキテクチャからの出力は、Blob ストレージに保存されるトレーニング済みモデルです。 リアルタイム スコアリングまたはバッチ スコアリングのどちらかに、このモデルを運用化できます。 詳細については、次の参照アーキテクチャをご覧ください。

- [Azure での Python scikit-learn モデルおよびディープ ラーニング モデルのリアルタイム スコアリング][real-time-scoring]
- [ディープ ラーニング モデル用の Azure でのバッチ スコアリング][batch-scoring]

[0]: ./_images/distributed_dl_architecture.png
[1]: ./_images/distributed_dl_flow.png
[2]: ./_images/distributed_dl_tests.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azure-blob]: /azure/storage/blobs/storage-blobs-introduction
[batch-ai]: /azure/batch-ai/overview
[batch-ai-files]: /azure/batch-ai/resource-concepts#file-server
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[benchmark]: https://github.com/msalvaris/BatchAIHorovodBenchmark
[blob]: https://azure.microsoft.com/en-gb/blog/introducing-azure-premium-blob-storage-limited-public-preview/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[cli]: https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md
[docker]: https://hub.docker.com/
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[files]: /azure/storage/files/storage-files-introduction
[github]: https://github.com/Azure/DistributedDeepLearning/
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[horovod]: https://github.com/uber/horovod
[imagenet]: http://www.image-net.org/
[real-time-scoring]: /azure/architecture/reference-architectures/ai/realtime-scoring-python
[resnet]: https://arxiv.org/abs/1512.03385
[security-guide]: /azure/storage/common/storage-security-guide
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows
[tutorial]: https://github.com/Azure/DistributedDeepLearning