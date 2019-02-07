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
# <a name="movie-recommendations-on-azure"></a><span data-ttu-id="e315d-103">Azure での映画のレコメンデーション</span><span class="sxs-lookup"><span data-stu-id="e315d-103">Movie recommendations on Azure</span></span>

<span data-ttu-id="e315d-104">この例のシナリオでは、企業が機械学習を使用して、顧客に対する製品のレコメンデーションを自動化する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="e315d-104">This example scenario shows how a business can use machine learning to automate product recommendations for their customers.</span></span> <span data-ttu-id="e315d-105">Azure データ サイエンス仮想マシン (DSVM) は、映画に与えられた評価に基づいて、ユーザーに映画をレコメンドする Azure 上のモデルをトレーニングするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-105">An Azure Data Science Virtual Machine (DSVM) is used to train a model on Azure that recommends movies to users based on ratings that have been given to movies.</span></span>

<span data-ttu-id="e315d-106">レコメンデーションは、小売業からニュース、メディアまで、さまざまな業界で役立つ可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e315d-106">Recommendations can be useful in various industries from retail to news to media.</span></span> <span data-ttu-id="e315d-107">考えられるアプリケーションとしては、仮想ストア内で製品のレコメンデーションを提供する、ニュースや投稿のレコメンデーションを提供する、音楽のレコメンデーションを提供するなどです。</span><span class="sxs-lookup"><span data-stu-id="e315d-107">Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations.</span></span> <span data-ttu-id="e315d-108">従来は、企業がアシスタントを採用して教育し、顧客に対して個人によるレコメンデーションを行う必要がありました。</span><span class="sxs-lookup"><span data-stu-id="e315d-108">Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers.</span></span> <span data-ttu-id="e315d-109">今日では、顧客の好みを理解するために Azure を利用してモデルをトレーニングすることで、カスタマイズされたレコメンデーションをまとめて提供することが可能です。</span><span class="sxs-lookup"><span data-stu-id="e315d-109">Today, we can  provide customized recommendations at scale by utilizing Azure to train models to understand customer preferences.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="e315d-110">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="e315d-110">Relevant use cases</span></span>

<span data-ttu-id="e315d-111">次のユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e315d-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="e315d-112">Web サイト上での映画のレコメンデーション。</span><span class="sxs-lookup"><span data-stu-id="e315d-112">Movie recommendations on a website.</span></span>
* <span data-ttu-id="e315d-113">モバイル アプリでのコンシューマー製品のレコメンデーション。</span><span class="sxs-lookup"><span data-stu-id="e315d-113">Consumer product recommendations in a mobile app.</span></span>
* <span data-ttu-id="e315d-114">ストリーミング メディア上でのニュースのレコメンデーション。</span><span class="sxs-lookup"><span data-stu-id="e315d-114">News recommendations on streaming media.</span></span>

## <a name="architecture"></a><span data-ttu-id="e315d-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="e315d-115">Architecture</span></span>

![映画のレコメンデーションをトレーニングするための機械学習モデルのアーキテクチャ][architecture]

<span data-ttu-id="e315d-117">このシナリオでは、映画評価のデータセット上で Spark の[交互最小二乗法][als] (ALS) アルゴリズムを使用した機械学習モデルのトレーニングと評価について取り上げます。</span><span class="sxs-lookup"><span data-stu-id="e315d-117">This scenario covers the training and evaluating of the machine learning model using the Spark [alternating least squares][als] (ALS) algorithm on a dataset of movie ratings.</span></span> <span data-ttu-id="e315d-118">このシナリオの手順を次に示します。</span><span class="sxs-lookup"><span data-stu-id="e315d-118">The steps for this scenario are as following:</span></span>

1. <span data-ttu-id="e315d-119">フロントエンド Web サイトまたはアプリ サービスによって、ユーザー、項目、および数値評価タプルのテーブルに表されたユーザーの映画に関する操作の履歴データを収集します。</span><span class="sxs-lookup"><span data-stu-id="e315d-119">The front-end website or app service collects historical data of user-movie interactions, which are represented in a table of user, item, and numerical rating tuples.</span></span>

2. <span data-ttu-id="e315d-120">収集された履歴データが、Blob ストレージに格納されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-120">The collected historical data is stored in a blob storage.</span></span>

3. <span data-ttu-id="e315d-121">Spark ALS レコメンダー モデルを試行したり製品化したりするために、DSVM がよく使用されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-121">A DSVM is often used to experiment with or productize a Spark ALS recommender model.</span></span> <span data-ttu-id="e315d-122">ALS モデルは、適切なデータ分割戦略を適用してデータセット全体から生成されたトレーニング データセットを使って、トレーニングされます。</span><span class="sxs-lookup"><span data-stu-id="e315d-122">The ALS model is trained using a training dataset, which is produced from the overall dataset by applying the appropriate data splitting strategy.</span></span> <span data-ttu-id="e315d-123">たとえば、データセットは、ビジネス要件に応じて、無作為、時系列、または階層化による複数のセットに分割することができます。</span><span class="sxs-lookup"><span data-stu-id="e315d-123">For example, the dataset can be split into sets randomly, chronologically, or stratified, depending on the business requirement.</span></span> <span data-ttu-id="e315d-124">他の機械学習タスクと同様に、レコメンダーは評価メトリック (たとえば、precision\@*k*、recall\@*k*、[MAP][map]、[nDCG\@k][ndcg]) を使用して検証されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-124">Similar to other machine learning tasks, a recommender is validated by using evaluation metrics (for example, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span></span>

4. <span data-ttu-id="e315d-125">Azure Machine Learning serivce は、ハイパーパラメーターのスイープおよびモデルの管理など、実験を調整するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-125">Azure Machine Learning service is used for coordinating the experimentation, such as hyperparameter sweeping and model management.</span></span>

5. <span data-ttu-id="e315d-126">トレーニング済みのモデルは、Azure Cosmos DB 上に保存され、その後は、指定されたユーザーに対して上位 *k* 作品の映画のレコメンデーションを行うために適用できます。</span><span class="sxs-lookup"><span data-stu-id="e315d-126">A trained model is preserved on Azure Cosmos DB, which can then be applied for recommending the top *k* movies for a given user.</span></span>

6. <span data-ttu-id="e315d-127">次に、モデルは Azure Container Instances または Azure Kubernetes Service を使用して、Web またはアプリ サービス上にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="e315d-127">The model is then deployed onto a web or app service by using Azure Container Instances or Azure Kubernetes Service.</span></span>

<span data-ttu-id="e315d-128">レコメンダー サービスの構築とスケーリングの詳細なガイドについては、「[Azure 上でリアルタイム レコメンデーション API を構築する][ref-arch]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e315d-128">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span>

### <a name="components"></a><span data-ttu-id="e315d-129">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="e315d-129">Components</span></span>

* <span data-ttu-id="e315d-130">[データ サイエンス仮想マシン][dsvm] (DSVM) は、機械学習およびデータ サイエンスに対応したディープ ラーニング フレームワークとツールを備えた Azure 仮想マシンです。</span><span class="sxs-lookup"><span data-stu-id="e315d-130">[Data Science Virtual Machine][dsvm] (DSVM) is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="e315d-131">DSVM は、ALS の実行に使用できるスタンドアロンの Spark 環境を備えています。</span><span class="sxs-lookup"><span data-stu-id="e315d-131">The DSVM has a standalone Spark environment that can be used to run ALS.</span></span>

* <span data-ttu-id="e315d-132">[Azure Blob ストレージ][blob]では、映画のレコメンデーション用のデータセットを格納します。</span><span class="sxs-lookup"><span data-stu-id="e315d-132">[Azure Blob storage][blob] stores the dataset for movie recommendations.</span></span>

* <span data-ttu-id="e315d-133">[Azure Machine Learning service][mls] は機械学習モデルの構築、管理、およびデプロイを高速化するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-133">[Azure Machine Learning service][mls] is used to accelerate the building, managing, and deploying of machine learning models.</span></span>

* <span data-ttu-id="e315d-134">[Azure Cosmos DB][cosmosdb] は、グローバル分散型のマルチモデルのデータベース ストレージに対応しています。</span><span class="sxs-lookup"><span data-stu-id="e315d-134">[Azure Cosmos DB][cosmosdb] enables globally distributed and multi-model database storage.</span></span>

* <span data-ttu-id="e315d-135">[Azure Container Instances][aci] は、必要に応じて [Azure Kubernetes Service][aks] を使用して、Web サービスまたはアプリ サービスにトレーニング済みのモデルをデプロイするために使用されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-135">[Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].</span></span>

### <a name="alternatives"></a><span data-ttu-id="e315d-136">代替手段</span><span class="sxs-lookup"><span data-stu-id="e315d-136">Alternatives</span></span>

<span data-ttu-id="e315d-137">[Azure Databricks][databricks] は、モデルのトレーニングと評価が行われるマネージド Spark クラスターです。</span><span class="sxs-lookup"><span data-stu-id="e315d-137">[Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed.</span></span> <span data-ttu-id="e315d-138">マネージド Spark の環境を数分で設定できます。また、[自動スケーリング][autoscale]によってスケールアップ/ダウンを行うことで、手動によるクラスターのスケーリングに伴うリソースとコストを低減させることができます。</span><span class="sxs-lookup"><span data-stu-id="e315d-138">You can set up a managed Spark environment in minutes, and [autoscale][autoscale] up and down to help reduce the resources and costs associated with scaling clusters manually.</span></span> <span data-ttu-id="e315d-139">リソースを節約するもう 1 つのオプションは、非アクティブ [クラスター][clusters]を構成して自動的に終了することです。</span><span class="sxs-lookup"><span data-stu-id="e315d-139">Another resource-saving option is to configure inactive [clusters][clusters] to terminate automatically.</span></span>

## <a name="considerations"></a><span data-ttu-id="e315d-140">考慮事項</span><span class="sxs-lookup"><span data-stu-id="e315d-140">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="e315d-141">可用性</span><span class="sxs-lookup"><span data-stu-id="e315d-141">Availability</span></span>

<span data-ttu-id="e315d-142">機械学習の組み込みアプリは、トレーニング用のリソースとサービス用のリソースという 2 つのリソース コンポーネントに分けられます。</span><span class="sxs-lookup"><span data-stu-id="e315d-142">Machine-learning-built apps are split into two resource components: resources for training, and resources for serving.</span></span> <span data-ttu-id="e315d-143">トレーニングに必要とされるリソースは、実際の本番環境の要求によって直接操作されることはないので、一般的に高可用性は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="e315d-143">Resources required for training generally do not need  high availability, as live production requests do not directly hit these resources.</span></span> <span data-ttu-id="e315d-144">サービスに必要とされるリソースは、顧客の要求を処理するために高可用性を備えている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e315d-144">Resources required for serving need to have high availability to serve customer requests.</span></span>

<span data-ttu-id="e315d-145">トレーニングについては、DSVM が世界中の[複数のリージョン][regions]で利用可能であり、仮想マシン用の[サービス レベル アグリーメント][sla] (SLA) を満たしています。</span><span class="sxs-lookup"><span data-stu-id="e315d-145">For training, the DSVM is available in [multiple regions][regions] around the globe and meets the [service level agreement][sla] (SLA) for virtual machines.</span></span> <span data-ttu-id="e315d-146">サービスについては、Azure Kubernetes Service が[高可用性][ha]インフラストラクチャを提供しています。</span><span class="sxs-lookup"><span data-stu-id="e315d-146">For serving, Azure Kubernetes Service provides a [highly available][ha] infrastructure.</span></span> <span data-ttu-id="e315d-147">また、エージェント ノードも、仮想マシン用の [SLA][sla-aks] に準拠しています。</span><span class="sxs-lookup"><span data-stu-id="e315d-147">Agent nodes also follow the [SLA][sla-aks] for virtual machines.</span></span>

### <a name="scalability"></a><span data-ttu-id="e315d-148">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="e315d-148">Scalability</span></span>

<span data-ttu-id="e315d-149">保持しているデータのサイズが大きい場合は、ご利用の DSVM をスケーリングしてトレーニング時間を短縮することが可能です。</span><span class="sxs-lookup"><span data-stu-id="e315d-149">If you have a large data size, you can scale your DSVM to shorten training time.</span></span> <span data-ttu-id="e315d-150">[VM サイズ][vm-size]を変更することで、VM をスケールアップまたはスケールダウンできます。</span><span class="sxs-lookup"><span data-stu-id="e315d-150">You can scale a VM up or down by changing the [VM size][vm-size].</span></span> <span data-ttu-id="e315d-151">トレーニングにかかる時間を減らすために、メモリ内のご自身のデータセットに見合う十分な大きさのメモリ サイズと多めの vCPU 数を選択します。</span><span class="sxs-lookup"><span data-stu-id="e315d-151">Choose a memory size large enough to fit your dataset in-memory and a higher vCPU count in order to decrease the amount of time that training takes.</span></span>

### <a name="security"></a><span data-ttu-id="e315d-152">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="e315d-152">Security</span></span>

<span data-ttu-id="e315d-153">このシナリオでは、Azure Active Directory を使用して、ご自身のコード、モデル、および (メモリ内) データを保管している [DSVM へのアクセス][dsvm-id]に対してユーザーを認証することが可能です。</span><span class="sxs-lookup"><span data-stu-id="e315d-153">This scenario can use Azure Active Directory to authenticate users for [access to the DSVM][dsvm-id], which contains your code, models, and (in-memory) data.</span></span> <span data-ttu-id="e315d-154">データは、DSVM 上に読み込まれるより前に Azure Storage 内に格納されます。その際、データは [Storage Service Encryption][storage-security] を使用して自動的に暗号化されます。</span><span class="sxs-lookup"><span data-stu-id="e315d-154">Data is stored in Azure Storage prior to being loaded on a DSVM, where it is automatically encrypted using [Storage Service Encryption][storage-security].</span></span> <span data-ttu-id="e315d-155">アクセス許可は、Azure Active Directory 認証またはロールベースのアクセス制御を利用して管理できます。</span><span class="sxs-lookup"><span data-stu-id="e315d-155">Permissions can be managed via Azure Active Directory authentication or role-based access control.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="e315d-156">このシナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="e315d-156">Deploy this scenario</span></span>

<span data-ttu-id="e315d-157">**前提条件**:既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="e315d-157">**Prerequisites**: You must have an existing Azure account.</span></span> <span data-ttu-id="e315d-158">Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][free]を作成してください。</span><span class="sxs-lookup"><span data-stu-id="e315d-158">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="e315d-159">このシナリオのすべてのコードは、[Microsoft の「Recommenders (レコメンダー)」リポジトリ][github]から入手できます。</span><span class="sxs-lookup"><span data-stu-id="e315d-159">All the code for this scenario is available in the [Microsoft Recommenders repository][github].</span></span>

<span data-ttu-id="e315d-160">次の手順に従って、[ALS クイックスタート ノートブック][notebook]を実行します。</span><span class="sxs-lookup"><span data-stu-id="e315d-160">Follow these steps to run the [ALS quickstart notebook][notebook]:</span></span>

1. <span data-ttu-id="e315d-161">Azure portal から [DSVM を作成][dsvm-ubuntu]します。</span><span class="sxs-lookup"><span data-stu-id="e315d-161">[Create a DSVM][dsvm-ubuntu] from the Azure portal.</span></span>

2. <span data-ttu-id="e315d-162">Notebooks フォルダー内にリポジトリを複製します。</span><span class="sxs-lookup"><span data-stu-id="e315d-162">Clone the repo in the Notebooks folder:</span></span>

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. <span data-ttu-id="e315d-163">[SETUP.md][setup] ファイルに示された手順に従って、conda 依存関係をインストールします。</span><span class="sxs-lookup"><span data-stu-id="e315d-163">Install the conda dependencies following the steps described in the [SETUP.md][setup] file.</span></span>

4. <span data-ttu-id="e315d-164">ブラウザーで、ご自身の jupyterlab VM に移動してから、`notebooks/00_quick_start/als_pyspark_movielens.ipynb` に移動します。</span><span class="sxs-lookup"><span data-stu-id="e315d-164">In a browser, go to your jupyterlab VM and navigate to `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span></span>

5. <span data-ttu-id="e315d-165">ノートブックを実行します。</span><span class="sxs-lookup"><span data-stu-id="e315d-165">Execute the notebook.</span></span>

## <a name="related-resources"></a><span data-ttu-id="e315d-166">関連リソース</span><span class="sxs-lookup"><span data-stu-id="e315d-166">Related resources</span></span>

<span data-ttu-id="e315d-167">レコメンダー サービスの構築とスケーリングの詳細なガイドについては、「[Azure 上でリアルタイム レコメンデーション API を構築する][ref-arch]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e315d-167">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span> <span data-ttu-id="e315d-168">レコメンデーション システムのチュートリアルと例については、[Microsoft の「Recommenders (レコメンダー)」リポジトリ][github]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e315d-168">For tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].</span></span>

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
