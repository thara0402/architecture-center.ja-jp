---
title: R 機械学習モデルを使用したリアルタイム スコアリング
description: Azure Kubernetes Service (AKS) で実行される Machine Learning Server を使用して、R のリアルタイム予測サービスを実装します。
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 5f3cc62c81c9ef9e5c3c27b1d66badd3e481c228
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887847"
---
# <a name="real-time-scoring-of-r-machine-learning-models-on-azure"></a>Azure で R 機械学習モデルを使用したリアルタイム スコアリング

このリファレンス アーキテクチャでは、Azure Kubernetes Service (AKS) で実行される Microsoft Machine Learning Server を使用して、R のリアルタイム (同期) 予測サービスを実装する方法を示しています。 このアーキテクチャは、汎用性があり、リアルタイム サービスとしてデプロイするつもりの、R で構築されるすべての予測モデルに適するように設計されています。 **[このソリューションをデプロイする][github]**。

## <a name="architecture"></a>アーキテクチャ

![Azure で R 機械学習モデルを使用したリアルタイム スコアリング][0]

このリファレンス アーキテクチャでは、コンテナー ベースのアプローチを採用しています。 新しいデータのスコア付けに必要なさまざまな成果物はもちろん、R を含む Docker イメージが構築されます。 これには、モデル オブジェクト自体とスコアリング スクリプトが含まれます。 このイメージは、Azure でホストされている Docker レジストリにプッシュされた後、同様に Azure 内にある Kubernetes クラスターにデプロイされます。

このワークフローのアーキテクチャには、以下のコンポーネントが含まれています。

- **[Azure Container Registry][acr]** は、このワークフローのイメージを格納するために使用されます。 Container Registry で作成されたレジストリは、標準の [Docker Registry V2 API][docker] とクライアントを使用して管理できます。

- **[Azure Kubernetes Service][aks]** は、デプロイとサービスをホストするために使用されます。 AKS で作成されたクラスターは、標準の [Kubernetes API][k-api] とクライアント (kubectl) を使用して管理できます。

- **[Microsoft Machine Learning Server][mmls]** は、サービスの REST API を定義するために使用され、[モデルの運用化][operationalization]を含んでいます。 このサービス指向の Web サーバー プロセスが要求をリッスンし、要求はその後、結果を生成する実際の R コードを実行する他のバックグラウンド プロセスに渡されます。 これらのプロセスはすべて、この構成ではコンテナーにラップされている 1 つのノードで実行されます。 開発環境やテスト環境の外でこのサービスを使用することの詳細については、Microsoft の担当者に問い合わせてください。

## <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

機械学習のワークロードは、トレーニング時と新しいデータのスコアリング時のどちらでも、多くのコンピューティング処理を要する傾向があります。 一般的に、コアあたり 1 つより多いスコアリング プロセスを実行しないようにしてください。 Machine Learning Server では、各コンテナーで実行される R プロセスの数を定義できます。 既定では 5 つのプロセスです。 変数の数が少ない線形回帰や、小さなデシジョン ツリーなどの比較的単純なモデルを作成するときには、プロセスの数を増やすことができます。 クラスター ノードでの CPU 負荷を監視して、コンテナーの数に関する適切な上限を決定します。

GPU 対応クラスターは、一部の種類のワークロード、特にディープ ラーニング モデルを高速化できます。 すべてのワークロードが GPU を活用できるわけではなく、それが可能なのはマトリックス代数を大量に使用するワークロードのみです。 たとえば、ランダム フォレスト モデルやブースティング モデルを含むツリー ベースのモデルは、一般に、GPU から利点を引き出せません。

ランダム フォレストなど、いくつかの種類のモデルは、CPU で大規模な並列化が可能です。 このような場合は、複数のコア間でワークロードを分散させることにで、1 つの要求のスコアリングを高速化します。 ただしそのようにすると、固定のクラスター サイズを指定された複数のスコアリング要求を処理するキャパシティが減少します。

一般に、オープン ソースの R モデルでは、すべてのデータがメモリ内に格納されるため、同時に実行する予定のプロセスに対応するために十分なメモリがノードにあることを確認します。 モデルに合わせて Machine Learning Server を使おうとしている場合は、メモリにすべてを読み込むのではなく、ディスク上のデータを処理できるライブラリを使用します。 これは、メモリの要件を大幅に引き下げる助けになります。 Machine Learning Server とオープン ソースの R のどちらを使用するかに関わらず、ノードを監視して、スコアリング プロセスでメモリが不足していないことを確認します。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

### <a name="network-encryption"></a>ネットワークの暗号化

このリファレンス アーキテクチャでは、クラスターとの通信では HTTPS が有効になっていて、[Let's Encrypt][encrypt] のステージング証明書が使用されています。 運用目的の場合は、適切な署名機関から入手した独自の証明書に置き換えてください。

### <a name="authentication-and-authorization"></a>認証と権限承認

Machine Learning Server の[モデルの運用化][operationalization]を使用するには、スコアリング要求が認証される必要があります。 このデプロイでは、ユーザー名とパスワードが使用されます。 企業での設定では、[Azure Active Directory][AAD] を使用して認証を有効にするか、[Azure API Management][API] を使用して別のフロント エンドを作成します。

モデルの運用化をコンテナー上の Machine Learning Server で正しく機能させるには、JSON Web トークン (JWT) の証明書をインストールする必要があります。 このデプロイでは、Microsoft が提供する証明書を使用しています。 運用環境の設定では、独自のものを提供してください。

Container Registry と AKS 間のトラフィックについては、[ロール ベースのアクセス制御][rbac] (RBAC) を有効にして、必要なものだけにアクセス特権を制限します。

### <a name="separate-storage"></a>別個のストレージ

このリファレンス アーキテクチャでは、1 つのイメージにアプリケーション (R) とデータ (モデル オブジェクトとスコアリング スクリプト) がまとめられています。 場合によっては、これらを分けるメリットがあります。 モデルのデータとコードを Azure の BLOB やファイル [ストレージ][storage]に配置し、コンテナーの初期化時にそれらを取得します。 この場合は、ストレージ アカウントが、認証されたアクセスのみを許可し、HTTPS を要求するように設定されていることを確認します。

## <a name="monitoring-and-logging-considerations"></a>監視とログ記録に関する考慮事項

[Kubernetes ダッシュ ボード][dashboard]使用して、AKS クラスターの全体的な状態を監視します。 詳細については、Azure portal でクラスターの概要ブレードを参照してください。 [GitHub][github] のリソースにも、R からダッシュ ボードを表示する方法が示されています。

ダッシュ ボードには、クラスターの全体的な正常性のビューが表示されますが、個々のコンテナーの状態を追跡することも重要です。 これを行うには、Azure portal でクラスターの概要ブレードから [Azure Monitor Insights][monitor] を有効にします。または、[コンテナー用の Azure Monitor][monitor-containers] (プレビュー段階) についてのページを参照してください。

## <a name="cost-considerations"></a>コストに関する考慮事項

Machine Learning Server は、コア単位でライセンスされます。クラスター内で Machine Learning Server を実行するすべてのコアがこれに加算されます。 Machine Learning Server または Microsoft SQL Server を使用する企業のお客様の場合は、料金の詳細について、マイクロソフトの担当者にお問い合わせください。

Machine Learning Server のオープン ソース代替品として、[Plumber][plumber] があります、これは、コードを REST API に変換する R パッケージです。 Plumber に備わる機能は Machine Learning Server よりも少なくなっています。 たとえば、要求の認証を提供する機能は、既定では一切含まれません。 Plumber を使用する場合は、[Azure API Management][API] を有効にして認証の細部を処理することをお勧めします。

ライセンス以外のコストに関する主な考慮事項は、Kubernetes クラスターのコンピューティング リソースです。 クラスターは、ピーク時に予想される要求量を処理するために十分なサイズである必要がありますが、この手法では、その他のときにリソースがアイドル状態に残されます。 アイドル状態のリソースの影響を限定するには、kubectl ツールを使用して、クラスターに対して[水平オートスケーラー][autoscaler]を有効にします。 または、AKS の[クラスター オートスケーラー][cluster-autoscaler]を使用します。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。 そこで説明されている手順に従って、単純な予測モデルをサービスとしてデプロイします。

<!-- links -->
[AAD]: /azure/active-directory/fundamentals/active-directory-whatis
[API]: /azure/api-management/api-management-key-concepts
[ACR]: /azure/container-registry/container-registry-intro
[AKS]: /azure/aks/intro-kubernetes
[autoscaler]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[cluster-autoscaler]: /azure/aks/autoscaler
[monitor]: /azure/monitoring/monitoring-container-insights-overview
[dashboard]: /azure/aks/kubernetes-dashboard
[docker]: https://docs.docker.com/registry/spec/api/
[encrypt]: https://letsencrypt.org/
[gitHub]: https://github.com/Azure/RealtimeRDeployment
[K-API]: https://kubernetes.io/docs/reference/
[MMLS]: /machine-learning-server/what-is-machine-learning-server
[monitor-containers]: /azure/azure-monitor/insights/container-insights-overview
[operationalization]: /machine-learning-server/what-is-operationalization
[plumber]: https://www.rplumber.io
[RBAC]: /azure/role-based-access-control/overview
[storage]: /azure/storage/common/storage-introduction
[0]: ./_images/realtime-scoring-r.png
