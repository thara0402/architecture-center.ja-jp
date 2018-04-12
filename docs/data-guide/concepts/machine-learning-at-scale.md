---
title: 大規模な Machine Learning
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: a92060008f90f43f71869bd1ad251af150b4a9db
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/31/2018
---
# <a name="machine-learning-at-scale"></a><span data-ttu-id="ad73e-102">大規模な Machine Learning</span><span class="sxs-lookup"><span data-stu-id="ad73e-102">Machine learning at scale</span></span>

<span data-ttu-id="ad73e-103">Machine Learning (ML) は、数学的アルゴリズムに基づく予測モデルのトレーニングに使用される手法です。</span><span class="sxs-lookup"><span data-stu-id="ad73e-103">Machine learning (ML) is a technique used to train predictive models based on mathematical algorithms.</span></span> <span data-ttu-id="ad73e-104">Machine Learning では、不明な値を予測するデータ フィールド間のリレーションシップを分析します。</span><span class="sxs-lookup"><span data-stu-id="ad73e-104">Machine learning analyzes the relationships between data fields to predict unknown values.</span></span>

<span data-ttu-id="ad73e-105">機械学習モデルの作成とデプロイは、次のような反復的プロセスです。</span><span class="sxs-lookup"><span data-stu-id="ad73e-105">Creating and deploying a machine learning model is an iterative process:</span></span>

* <span data-ttu-id="ad73e-106">データ サイエンティストが元データを調査し、*機能*と予測される*ラベル*間のリレーションシップを決定する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-106">Data scientists explore the source data to determine relationships between *features* and predicted *labels*.</span></span>
* <span data-ttu-id="ad73e-107">データ サイエンティストが、適切なアルゴリズムに基づくモデルをトレーニングして検証し、予測に最適なモデルを見つける。</span><span class="sxs-lookup"><span data-stu-id="ad73e-107">The data scientists train and validate models based on appropriate algorithms to find the optimal model for prediction.</span></span>
* <span data-ttu-id="ad73e-108">最適なモデルが、Web サービスやその他のカプセル化された機能として、運用環境にデプロイされる。</span><span class="sxs-lookup"><span data-stu-id="ad73e-108">The optimal model is deployed into production, as a web service or some other encapsulated function.</span></span>
* <span data-ttu-id="ad73e-109">新しいデータが収集されると、モデルが定期的に再トレーニングされて有効性を改善する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-109">As new data is collected, the model is periodically retrained to improve is effectiveness.</span></span>

<span data-ttu-id="ad73e-110">大規模な Machine Learning では、2 つの異なるスケーラビリティ上の注意点に対応します。</span><span class="sxs-lookup"><span data-stu-id="ad73e-110">Machine learning at scale addresses two different scalability concerns.</span></span> <span data-ttu-id="ad73e-111">1 つ目の注意点は、クラスターのスケールアウト機能をトレーニングする必要がある、大規模データ セットに対するモデルをトレーニングすることです。</span><span class="sxs-lookup"><span data-stu-id="ad73e-111">The first is training a model against large data sets that require the scale-out capabilities of a cluster to train.</span></span> <span data-ttu-id="ad73e-112">2 つ目の注意点は、モデルを使用するアプリケーションの要件に合うように、スケーリング可能な方法で学習したモデルを操作化することです。</span><span class="sxs-lookup"><span data-stu-id="ad73e-112">The second centers is operationalizating the learned model in a way that can scale to meet the demands of the applications that consume it.</span></span> <span data-ttu-id="ad73e-113">これは通常、スケールアウト可能な Web サービスとして予測機能をデプロイすることで、対応できます。</span><span class="sxs-lookup"><span data-stu-id="ad73e-113">Typically this is accomplished by deploying the predictive capabilities as a web service that can then be scaled out.</span></span>

<span data-ttu-id="ad73e-114">多くの場合、データが多いほどより優れたモデルが得られるため、大規模な Machine Learning では、強力な予測機能を生み出せるという利点があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-114">Machine learning at scale has the benefit that it can produce powerful, predictive capabilities because better models typically result from more data.</span></span> <span data-ttu-id="ad73e-115">モデルがトレーニングされると、ステートレスで高パフォーマンスのスケールアウト Web サービスとしてデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="ad73e-115">Once a model is trained, it can be deployed as a stateless, highly-performant, scale-out web service.</span></span> 

## <a name="model-preparation-and-training"></a><span data-ttu-id="ad73e-116">モデルの準備およびトレーニング</span><span class="sxs-lookup"><span data-stu-id="ad73e-116">Model preparation and training</span></span>

<span data-ttu-id="ad73e-117">モデルの準備とトレーニングの段階では、データ サイエンティストは Python や R などの言語を使用して対話的にデータを探索し、以下の作業を行います。</span><span class="sxs-lookup"><span data-stu-id="ad73e-117">During the model preparation and training phase, data scientists explore the data interactively using languages like Python and R to:</span></span>

* <span data-ttu-id="ad73e-118">大量データ ストアからサンプルを抽出する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-118">Extract samples from high volume data stores.</span></span>
* <span data-ttu-id="ad73e-119">外れ値、重複値、および欠損値を見つけて処理し、データを整理する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-119">Find and treat outliers, duplicates, and missing values to clean the data.</span></span>
* <span data-ttu-id="ad73e-120">統計分析および視覚化を通して、データの相関関係およびリレーションシップを決定する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-120">Determine correlations and relationships in the data through statistical analysis and visualization.</span></span>
* <span data-ttu-id="ad73e-121">統計的リレーションシップの予測性を改善する、新しい集計機能を生成する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-121">Generate new calculated features that improve the predictiveness of statistical relationships.</span></span>
* <span data-ttu-id="ad73e-122">予測アルゴリズムに基づいて ML モデルをトレーニングする。</span><span class="sxs-lookup"><span data-stu-id="ad73e-122">Train ML models based on predictive algorithms.</span></span>
* <span data-ttu-id="ad73e-123">トレーニング時に留保したデータを使用して、トレーニング済みモデルを検証する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-123">Validate trained models using data that was withheld during training.</span></span>

<span data-ttu-id="ad73e-124">この対話型の分析およびモデリングの段階をサポートするには、データ プラットフォームで、データ サイエンティストがさまざまなツールを使用してデータを探索できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-124">To support this interactive analysis and modeling phase, the data platform must enable data scientists to explore data using a variety of tools.</span></span> <span data-ttu-id="ad73e-125">さらに、複雑な機械学習モデルのトレーニングでは、大量データを集中処理する必要が生じる場合があるので、モデルのトレーニングをスケールアウトするための十分なリソースが不可欠です。</span><span class="sxs-lookup"><span data-stu-id="ad73e-125">Additionally, the training of a complex machine learning model can require a lot of intensive processing of high volumes of data, so sufficient resources for scaling out the model training is essential.</span></span>

## <a name="model-deployment-and-consumption"></a><span data-ttu-id="ad73e-126">モデルのデプロイと使用</span><span class="sxs-lookup"><span data-stu-id="ad73e-126">Model deployment and consumption</span></span>

<span data-ttu-id="ad73e-127">モデルをデプロイする準備ができたら、Web サービスとしてカプセル化して、クラウド内に、エッジ デバイスに、または企業の ML 実行環境内にデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="ad73e-127">When a model is ready to be deployed, it can be encapsulated as a web service and deployed in the cloud, to an edge device, or within an enterprise ML execution environment.</span></span> <span data-ttu-id="ad73e-128">このデプロイ プロセスは、操作化と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="ad73e-128">This deployment process is referred to as operationalization.</span></span>

## <a name="challenges"></a><span data-ttu-id="ad73e-129">課題</span><span class="sxs-lookup"><span data-stu-id="ad73e-129">Challenges</span></span>

<span data-ttu-id="ad73e-130">大規模な Machine Learning では、以下のような課題が発生します。</span><span class="sxs-lookup"><span data-stu-id="ad73e-130">Machine learning at scale produces a few challenges:</span></span>

- <span data-ttu-id="ad73e-131">通常、モデルのトレーニング、特に深層学習モデルのトレーニングには、大量のデータが必要になります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-131">You typically need a lot of data to train a model, especially for deep learning models.</span></span>
- <span data-ttu-id="ad73e-132">モデルのトレーニングを開始する前に、これらの大規模データ セットを準備する必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-132">You need to prepare these big data sets before you can even begin training your model.</span></span>
- <span data-ttu-id="ad73e-133">モデルのトレーニングの段階では、ビッグ データ ストアにアクセスする必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-133">The model training phase must access the big data stores.</span></span> <span data-ttu-id="ad73e-134">一般的には、Spark など、データ準備に使用したときと同じビッグ データ クラスターを使用して、モデル トレーニングを実行します。</span><span class="sxs-lookup"><span data-stu-id="ad73e-134">It's common to perform the model training using the same big data cluster, such as Spark, that is used for data preparation.</span></span> 
- <span data-ttu-id="ad73e-135">深層学習などのシナリオでは、CPU でのスケールアウトを可能にするクラスターが必要になるだけでなく、GPU が有効化されたノードでクラスターが構成されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-135">For scenarios such as deep learning, not only will you need a cluster that can provide you scale out on CPUs, but your cluster will need to consist of GPU-enabled nodes.</span></span>

## <a name="machine-learning-at-scale-in-azure"></a><span data-ttu-id="ad73e-136">Azure での大規模 Machine Learning</span><span class="sxs-lookup"><span data-stu-id="ad73e-136">Machine learning at scale in Azure</span></span>

<span data-ttu-id="ad73e-137">どの ML サービスを使ってトレーニングおよび操作化を行うかを決定する前に、そもそもモデルのトレーニングが必要か、また、事前構築済みのモデルが要件に合うかどうかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="ad73e-137">Before deciding which ML services to use in training and operationalization, consider whether you need to train a model at all, or if a prebuilt model can meet your requirements.</span></span> <span data-ttu-id="ad73e-138">多くの場合、事前構築済みモデルの使用とは、単に Web サービスの呼び出しまたは ML ライブラリの使用による既存モデルの読み込みのことです。</span><span class="sxs-lookup"><span data-stu-id="ad73e-138">In many cases, using a prebuilt model is just a matter of calling a web service or using an ML library to load an existing model.</span></span> <span data-ttu-id="ad73e-139">次に、いくつかの選択肢を示します。</span><span class="sxs-lookup"><span data-stu-id="ad73e-139">Some options include:</span></span> 

- <span data-ttu-id="ad73e-140">Microsoft Cognitive Services が提供する Web サービスを使用する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-140">Use the web services provided by Microsoft Cognitive Services.</span></span>
- <span data-ttu-id="ad73e-141">Cognitive Toolkit が提供する事前トレーニング済みのニューラル ネットワーク モデルを使用する。</span><span class="sxs-lookup"><span data-stu-id="ad73e-141">Use the pretrained neural network models provided by Cognitive Toolkit.</span></span>
- <span data-ttu-id="ad73e-142">iOS アプリ対応の Core ML が提供するシリアル化モデルを埋め込む。</span><span class="sxs-lookup"><span data-stu-id="ad73e-142">Embed the serialized models provided by Core ML for an iOS apps.</span></span> 

<span data-ttu-id="ad73e-143">事前構築済みのモデルがお使いのデータやシナリオに適合しない場合、Azure での選択肢には、Azure Machine Learning、Spark MLlib および MMLSpark 搭載の HDInsight、Cognitive Toolkit、および SQL Machine Learning Services が含まれます。</span><span class="sxs-lookup"><span data-stu-id="ad73e-143">If a prebuilt model does not fit your data or your scenario, options in Azure include Azure Machine Learning, HDInsight with Spark MLlib and MMLSpark, Cognitive Toolkit, and SQL Machine Learning Services.</span></span> <span data-ttu-id="ad73e-144">カスタム モデルを使用することに決めた場合、モデルのトレーニングと操作化を含むパイプラインを設計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="ad73e-144">If you decide to use a custom model, you must design a pipeline that includes model training and operationalization.</span></span> 

![Azure でのモデル オプション](./images/machine-learning-model-training-and-deployment.png)

<span data-ttu-id="ad73e-146">Azure の ML で選択可能な技術の一覧については、以下のトピックをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="ad73e-146">For a list of technology choices for ML in Azure, see the following topics:</span></span>

- [<span data-ttu-id="ad73e-147">認識サービス技術の選択</span><span class="sxs-lookup"><span data-stu-id="ad73e-147">Choosing a cognitive services technology</span></span>](../technology-choices/cognitive-services.md)
- [<span data-ttu-id="ad73e-148">機械学習技術の選択</span><span class="sxs-lookup"><span data-stu-id="ad73e-148">Choosing a machine learning technology</span></span>](../technology-choices/data-science-and-machine-learning.md)
- [<span data-ttu-id="ad73e-149">自然言語処理技術の選択</span><span class="sxs-lookup"><span data-stu-id="ad73e-149">Choosing a natural language processing technology</span></span>](../technology-choices/natural-language-processing.md)
