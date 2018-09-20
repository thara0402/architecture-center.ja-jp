---
title: Azure での 3D ビデオのレンダリング
description: Azure Batch サービスを使用した、Azure でのネイティブ HPC ワークロードの実行
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 723d437671c52dc9f717bef9641663d0e7a8fbc4
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389351"
---
# <a name="3d-video-rendering-on-azure"></a>Azure での 3D ビデオのレンダリング

3D レンダリングは時間がかかるプロセスで、完了までにかなりの CPU 時間が必要です。  1 台のコンピューターで静的なアセットからビデオ ファイルを生成している場合、作成しているビデオの長さと複雑さよっては、そのプロセスに数時間から、数日かかる場合もあります。  多くの企業は、これらのタスクを実行するために高価なハイエンド デスクトップ コンピューターを購入するか、ジョブを送信できる大規模なレンダリング ファームに投資しています。  しかし、Azure Batch を使用すれば、その能力を必要なときだけ使用して、不要なときはシャットダウンできます。資本投資は必要ありません。

Windows Server と Linux コンピューティングのどちらのノードを選択しても、Batch によって一貫したジョブの管理やスケジュール設定を実行できます。 Batch を使用すると、AutoDesk Maya、Blender など、既存の Windows または Linux アプリケーションを使用して、Azure で大規模なレンダリング ジョブを実行することができます。

## <a name="related-use-cases"></a>関連するユース ケース

次のようなユース ケースについて、このシナリオを検討してください。

* 3D モデリング
* Visual FX (VFX) レンダリング
* ビデオ コード変換
* 画像処理、色補正、およびサイズ変更

## <a name="architecture"></a>アーキテクチャ

![Azure Batch を使用したクラウド ネイティブ HPC ソリューションに関与するコンポーネントのアーキテクチャの概要][architecture]

このサンプル シナリオでは、Azure Batch を使用するときのワークフローに対応します。データ フローは次のとおりです。

1. 入力ファイルとそのファイルを処理するアプリケーションを Azure Storage アカウントにアップロードする
2. Batch アカウント内のコンピューティング ノードの Batch プール、プールでワークロードを実行するジョブ、およびジョブ内のタスクを作成する。
3. 入力ファイルとアプリケーションを Batch にダウンロードする
4. タスクの実行を監視する
5. タスク出力をアップロードする
6. 出力ファイルをダウンロードする

このプロセスを簡略化するために、[Maya および 3ds Max 用の Batch プラグイン][batch-plugins]を使用することもできます

### <a name="components"></a>コンポーネント

Azure Batch は、次の Azure テクノロジに基づいています。

* [リソース グループ][resource-groups]は、Azure リソースの論理コンテナーです。
* [仮想ネットワーク][vnet]は、ヘッド ノードとコンピューティング リソースの両方に使用されます
* [ストレージ][storage] アカウントは、同期とデータ保有に使用されます
* [仮想マシン スケール セット][vmss]は、CycleCloud によってコンピューティング リソース用に使用されます

## <a name="considerations"></a>考慮事項

### <a name="machine-sizes-available-for-azure-batch"></a>Azure Batch で使用できるマシンのサイズ

レンダリング カスタマーのほとんどが高 CPU パワーのリソースを選択しますが、仮想マシンスケール セットを使用するワークロードでは VM を異なる方法で選択でき、そのワークロードはさまざまな要因に依存します。

* バインドされたメモリでアプリケーションが実行されているか。
* アプリケーションが GPU を使用する必要があるか。 
* ジョブの種類が驚異的並列か、または密に結合されたジョブに対する InfiniBand 接続が必要か。
* コンピューティング ノードでストレージへの高速の I/O が必要か

Azure では、上記のアプリケーション要件のそれぞれに対処できる広範な VM サイズが必要で、その一部は HPC に固有ですが、最小のサイズを使用して、効果的なグリッド実装を提供することもできます。

* [HPC VM サイズ][compute-hpc]: CPU バインド型レンダリングの性質があるため、Microsoft では、通常、Azure H シリーズの VM を提案しています。  この種類の VM は、特にハイエンド コンピューティング ニーズ用に構築され、8 および 16 コア vCPU サイズが使用可能で、DDR4 メモリ、SSD 一時ストレージ、および Haswell E5 Intel テクノロジを備えています。
* [GPU VM サイズ][compute-gpu]: GPU 最適化済み VM サイズは、1 つまたは複数の NVIDIA GPU を備えた、特殊な用途に特化した仮想マシンです。 これらのサイズは、コンピューティング処理やグラフィック処理の負荷が高い視覚化ワークロードを意図して設計されています。
* NC、NCv2、NCv3、ND の各サイズは、CUDA および OpenCL ベースのアプリケーションやシミュレーション、AI、ディープ ラーニングなどの、コンピューティング集中型およびネットワーク集中型のアプリケーション、アルゴリズム用に最適化されています。 NV サイズは、リモートの視覚化、ストリーミング、ゲーム、エンコーディング、および OpenGL や DirectX などのフレームワークを使用する VDI シナリオ用に最適化されています。
* [メモリ最適化 VM サイズ][compute-memory]: さらに多くのメモリが必要な場合は、より高いメモリ対 CPU 比率を提供するメモリ最適化 VM サイズを使用します。
* [汎用 VM サイズ][compute-general]: バランスのとれた CPU 対メモリ比率を提供する汎用 VM サイズを使用することもできます。

### <a name="alternatives"></a>代替手段

Azure のレンダリング環境をより細かく制御する必要がある場合、またはハイブリッド実装が必要な場合は、CycleCloud コンピューティングがクラウドでの IaaS グリッドの調整に役立ちます。 基盤となる Azure テクノロジとして Azure Batch と同じものを使用すると、IaaS グリッドの作成および維持プロセスを効率化できます。 設計の原則の詳細については、次のリンクを使用してください。

Azure で使用できるすべての HPC ソリューションの完全な概要については、「[Azure VM を使用した HPC、Batch、および Big Compute ソリューション][hpc-alt-solutions]」を参照してください

### <a name="availability"></a>可用性

さまざまなサービス、ツール、および API を利用して、Azure Batch のコンポーネントを監視できます。 監視の詳細については、「[Batch ソリューションの監視][batch-monitor]」を参照してください。

### <a name="scalability"></a>スケーラビリティ

Azure Batch アカウント内のプールをスケーリングするには、手動で介入するか、Azure Batch メトリックに基づく数式を使用して自動的に行います。 スケーラビリティの詳細については、[Batch プール内のノードをスケーリングするための自動スケーリングの数式の作成][batch-scaling]に関するページを参照してください。

### <a name="security"></a>セキュリティ

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

現在 Azure Batch にはフェールオーバー機能がありません。計画外の停止が発生した場合は、次の手順を使用して、可用性を確保することをお勧めします。

* 代替ストレージ アカウントを使用して代替の Azure の場所で Azure Batch アカウントを作成します
* 同じノード プールを、同じ名前で、ノードの割り当てなしで作成します
* アプリケーションが作成され、代替ストレージ アカウントに更新されていることを確認します
* 入力ファイルをアップロードし、ジョブを代替 Azure Batch アカウントに送信します

## <a name="deploy-this-scenario"></a>このシナリオのデプロイ

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>Azure Batch アカウントとプールを手動で作成する

このサンプル シナリオでは、ご自身の顧客向けに開発できる Azure Batch ラボを SaaS ソリューションの例として紹介しながら、Azure Batch のしくみを学習するうえで役立ちます。

[Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートを使用してサンプル シナリオをデプロイする

テンプレートによってデプロイされるもの:

* 新しい Azure Batch アカウント
* ストレージ アカウント
* バッチ アカウントに関連付けられているノード プール
* ノード プールは、A2 v2 VM Canonical Ubuntu イメージを使用するように構成されます
* ノード プールには最初は VM がなく、VM を追加して手動でスケーリングする必要があります

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

詳しくは、[Resource Manager テンプレート][azure-arm-templates]に関するページをご覧ください。

## <a name="pricing"></a>価格

Azure Batch の使用コストは、プールに使用されている VM のサイズと VM が割り当てられ実行されている期間によって異なります。Azure Batch アカウントの作成に関連するコストはありません。 ストレージとデータの送信を追加コストとして考慮する必要があります。

あるジョブを 8 時間実行した場合に発生するコストの例を、サーバー数ごとに次に示します。

* 100 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-high]

  100 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信

* 50 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-med]

  50 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信

* 10 台の高パフォーマンス CPU VM: [コストの見積もり][hpc-est-low]
  
  10 x H16m (16 コア、225 GB RAM、Premium Storage 512 GB)、2 TB Blob Storage、1 TB 送信

### <a name="low-priority-vm-pricing"></a>低優先度 VM の価格

Azure Batch では、ノード プールでの低優先度 VM* の使用もサポートされます。これにより、大幅にコストを削減できる可能性があります。 標準 VM と低優先度 VM の価格の比較と、低優先度 VM の詳細については、「[Batch の価格][batch-pricing]」を参照してください。

\*低優先度 VM での実行に適しているのは、特定のアプリケーションとワークロードだけであることに注意してください。

## <a name="related-resources"></a>関連リソース

[Azure Batch の概要][batch-overview]

[Azure Batch のドキュメント][batch-doc]

[Azure Batch でのコンテナーの使用][batch-containers]

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job