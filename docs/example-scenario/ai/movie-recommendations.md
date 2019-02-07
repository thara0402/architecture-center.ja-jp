---
title: Azure での映画のレコメンデーション
description: 機械学習を利用して、映画、製品、およびその他のレコメンデーションを自動化します。Azure 上でモデルをトレーニングするために 機械学習と Azure データ サイエンス仮想マシン (DSVM) を使用します。
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: 9387ab7989695df29df53d7aa4a437010cdd9fdf
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482996"
---
# <a name="movie-recommendations-on-azure"></a>Azure での映画のレコメンデーション

この例のシナリオでは、企業が機械学習を使用して、顧客に対する製品のレコメンデーションを自動化する方法を示します。 Azure データ サイエンス仮想マシン (DSVM) は、映画に与えられた評価に基づいて、ユーザーに映画をレコメンドする Azure 上のモデルをトレーニングするために使用されます。

レコメンデーションは、小売業からニュース、メディアまで、さまざまな業界で役立つ可能性があります。 考えられるアプリケーションとしては、仮想ストア内で製品のレコメンデーションを提供する、ニュースや投稿のレコメンデーションを提供する、音楽のレコメンデーションを提供するなどです。 従来は、企業がアシスタントを採用して教育し、顧客に対して個人によるレコメンデーションを行う必要がありました。 今日では、顧客の好みを理解するために Azure を利用してモデルをトレーニングすることで、カスタマイズされたレコメンデーションをまとめて提供することが可能です。

## <a name="relevant-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* Web サイト上での映画のレコメンデーション。
* モバイル アプリでのコンシューマー製品のレコメンデーション。
* ストリーミング メディア上でのニュースのレコメンデーション。

## <a name="architecture"></a>アーキテクチャ

![映画のレコメンデーションをトレーニングするための機械学習モデルのアーキテクチャ][architecture]

このシナリオでは、映画評価のデータセット上で Spark の[交互最小二乗法][als] (ALS) アルゴリズムを使用した機械学習モデルのトレーニングと評価について取り上げます。 このシナリオの手順を次に示します。

1. フロントエンド Web サイトまたはアプリ サービスによって、ユーザー、項目、および数値評価タプルのテーブルに表されたユーザーの映画に関する操作の履歴データを収集します。

2. 収集された履歴データが、Blob ストレージに格納されます。

3. Spark ALS レコメンダー モデルを試行したり製品化したりするために、DSVM がよく使用されます。 ALS モデルは、適切なデータ分割戦略を適用してデータセット全体から生成されたトレーニング データセットを使って、トレーニングされます。 たとえば、データセットは、ビジネス要件に応じて、無作為、時系列、または階層化による複数のセットに分割することができます。 他の機械学習タスクと同様に、レコメンダーは評価メトリック (たとえば、precision\@*k*、recall\@*k*、[MAP][map]、[nDCG\@k][ndcg]) を使用して検証されます。

4. Azure Machine Learning serivce は、ハイパーパラメーターのスイープおよびモデルの管理など、実験を調整するために使用されます。

5. トレーニング済みのモデルは、Azure Cosmos DB 上に保存され、その後は、指定されたユーザーに対して上位 *k* 作品の映画のレコメンデーションを行うために適用できます。

6. 次に、モデルは Azure Container Instances または Azure Kubernetes Service を使用して、Web またはアプリ サービス上にデプロイされます。

レコメンダー サービスの構築とスケーリングの詳細なガイドについては、「[Azure 上でリアルタイム レコメンデーション API を構築する][ref-arch]」を参照してください。

### <a name="components"></a>コンポーネント

* [データ サイエンス仮想マシン][dsvm] (DSVM) は、機械学習およびデータ サイエンスに対応したディープ ラーニング フレームワークとツールを備えた Azure 仮想マシンです。 DSVM は、ALS の実行に使用できるスタンドアロンの Spark 環境を備えています。

* [Azure Blob ストレージ][blob]では、映画のレコメンデーション用のデータセットを格納します。

* [Azure Machine Learning service][mls] は機械学習モデルの構築、管理、およびデプロイを高速化するために使用されます。

* [Azure Cosmos DB][cosmosdb] は、グローバル分散型のマルチモデルのデータベース ストレージに対応しています。

* [Azure Container Instances][aci] は、必要に応じて [Azure Kubernetes Service][aks] を使用して、Web サービスまたはアプリ サービスにトレーニング済みのモデルをデプロイするために使用されます。

### <a name="alternatives"></a>代替手段

[Azure Databricks][databricks] は、モデルのトレーニングと評価が行われるマネージド Spark クラスターです。 マネージド Spark の環境を数分で設定できます。また、[自動スケーリング][autoscale]によってスケールアップ/ダウンを行うことで、手動によるクラスターのスケーリングに伴うリソースとコストを低減させることができます。 リソースを節約するもう 1 つのオプションは、非アクティブ [クラスター][clusters]を構成して自動的に終了することです。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

機械学習の組み込みアプリは、トレーニング用のリソースとサービス用のリソースという 2 つのリソース コンポーネントに分けられます。 トレーニングに必要とされるリソースは、実際の本番環境の要求によって直接操作されることはないので、一般的に高可用性は必要ありません。 サービスに必要とされるリソースは、顧客の要求を処理するために高可用性を備えている必要があります。

トレーニングについては、DSVM が世界中の[複数のリージョン][regions]で利用可能であり、仮想マシン用の[サービス レベル アグリーメント][sla] (SLA) を満たしています。 サービスについては、Azure Kubernetes Service が[高可用性][ha]インフラストラクチャを提供しています。 また、エージェント ノードも、仮想マシン用の [SLA][sla-aks] に準拠しています。

### <a name="scalability"></a>スケーラビリティ

保持しているデータのサイズが大きい場合は、ご利用の DSVM をスケーリングしてトレーニング時間を短縮することが可能です。 [VM サイズ][vm-size]を変更することで、VM をスケールアップまたはスケールダウンできます。 トレーニングにかかる時間を減らすために、メモリ内のご自身のデータセットに見合う十分な大きさのメモリ サイズと多めの vCPU 数を選択します。

### <a name="security"></a>セキュリティ

このシナリオでは、Azure Active Directory を使用して、ご自身のコード、モデル、および (メモリ内) データを保管している [DSVM へのアクセス][dsvm-id]に対してユーザーを認証することが可能です。 データは、DSVM 上に読み込まれるより前に Azure Storage 内に格納されます。その際、データは [Storage Service Encryption][storage-security] を使用して自動的に暗号化されます。 アクセス許可は、Azure Active Directory 認証またはロールベースのアクセス制御を利用して管理できます。

## <a name="deploy-this-scenario"></a>このシナリオのデプロイ

**前提条件**:既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][free]を作成してください。

このシナリオのすべてのコードは、[Microsoft の「Recommenders (レコメンダー)」リポジトリ][github]から入手できます。

次の手順に従って、[ALS クイックスタート ノートブック][notebook]を実行します。

1. Azure portal から [DSVM を作成][dsvm-ubuntu]します。

2. Notebooks フォルダー内にリポジトリを複製します。

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. [SETUP.md][setup] ファイルに示された手順に従って、conda 依存関係をインストールします。

4. ブラウザーで、ご自身の jupyterlab VM に移動してから、`notebooks/00_quick_start/als_pyspark_movielens.ipynb` に移動します。

5. ノートブックを実行します。

## <a name="related-resources"></a>関連リソース

レコメンダー サービスの構築とスケーリングの詳細なガイドについては、「[Azure 上でリアルタイム レコメンデーション API を構築する][ref-arch]」を参照してください。 レコメンデーション システムのチュートリアルと例については、[Microsoft の「Recommenders (レコメンダー)」リポジトリ][github]を参照してください。

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
