---
title: 機械学習テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 507d343ca7bc0a3f161602c50e6b96e921276374
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902375"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="9f5e4-102">Azure での機械学習テクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="9f5e4-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="9f5e4-103">データ サイエンスと機械学習は、通常はデータ サイエンティストによって実行されるワークロードです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="9f5e4-104">スペシャリスト ツールを必要とし、そのツールの多くはデータ サイエンティストが実行する必要のある対話型のデータ探索およびモデリング タスクの種類専用に設計されています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="9f5e4-105">機械学習ソリューションは反復的に構築され、2 つのフェーズがあります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>
* <span data-ttu-id="9f5e4-106">データの準備とモデリング。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-106">Data preparation and modeling.</span></span>
* <span data-ttu-id="9f5e4-107">予測サービスのデプロイと使用。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="9f5e4-108">データの準備およびモデリング用のツールとサービス</span><span class="sxs-lookup"><span data-stu-id="9f5e4-108">Tools and services for data preparation and modeling</span></span>
<span data-ttu-id="9f5e4-109">データ サイエンティストは通常、Python または R で記述されたカスタム コードを使用してデータを操作することを好みます。一般に、このコードは繰り返し実行され、データ サイエンティストはそのコードを使用してデータのクエリと探索を実行し、関係の特定に役立つ視覚化と統計を生成します。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="9f5e4-110">データ サイエンティストが使用できる R および Python 用の対話型環境は数多くあります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="9f5e4-111">特に人気があるのは **Jupyter Notebook** で、これが提供するブラウザー ベースのシェルを使用して、データ サイエンティストは R または Python のコードとマークダウン テキストを含む "*ノートブック*" ファイルを作成できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="9f5e4-112">これにより、コードと結果を 1 つのドキュメントで共有および文書化して効果的に共同作業を行うことができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="9f5e4-113">よく使用されるその他のツールは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-113">Other commonly used tools include:</span></span>
* <span data-ttu-id="9f5e4-114">**Spyder**: Anaconda Python ディストリビューションで提供される Python 用の対話型開発環境 (IDE) です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
* <span data-ttu-id="9f5e4-115">**R Studio**: R プログラミング言語用の IDE です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-115">**R Studio**: An IDE for the R programming language.</span></span>
* <span data-ttu-id="9f5e4-116">**Visual Studio Code**: Python に加えて一般に使用される機械学習および AI 開発用のフレームワークをサポートする軽量のクロスプラットフォーム コーディング環境です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="9f5e4-117">これらのツールに加えて、データ サイエンティストは、コードおよびモデル管理を簡略化する Azure サービスを利用できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="9f5e4-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="9f5e4-118">Azure Notebooks</span></span>
<span data-ttu-id="9f5e4-119">Azure Notebooks は、オンライン Jupyter Notebook サービスであり、データ サイエンティストがクラウドベースのライブラリで Jupyter Notebook を作成、実行、共有できるようにします。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="9f5e4-120">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-120">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-121">無料のサービスで、Azure サブスクリプションは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-121">Free service&mdash;no Azure subscription required.</span></span>
* <span data-ttu-id="9f5e4-122">Jupyter やサポートする R または Python ディストリビューションをローカルにインストールする必要はありません。ブラウザーだけを使用します。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
* <span data-ttu-id="9f5e4-123">独自のオンライン ライブラリを管理し、任意のデバイスからアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-123">Manage your own online libraries and access them from any device.</span></span>
* <span data-ttu-id="9f5e4-124">ノートブックをコラボレーターと共有できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="9f5e4-125">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-125">Considerations:</span></span>

* <span data-ttu-id="9f5e4-126">オフラインのときは、ノートブックにアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-126">You will be unable to access your notebooks when offline.</span></span>
* <span data-ttu-id="9f5e4-127">無料のノートブック サービスの制限付き処理機能は、大規模または複雑なモデルのトレーニングに不十分な場合があります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="9f5e4-128">データ サイエンス仮想マシン</span><span class="sxs-lookup"><span data-stu-id="9f5e4-128">Data science virtual machine</span></span>
<span data-ttu-id="9f5e4-129">データ サイエンス仮想マシンは、R、Python、Jupyter Notebook、Visual Studio Code、Microsoft Cognitive Toolkit のような機械学習モデリング用ライブラリなど、データ サイエンティストがよく使用するツールとフレームワークを含む Azure 仮想マシン イメージです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="9f5e4-130">これらのツールは、インストールが複雑で時間がかかる場合があり、バージョン管理の問題につながることの多い多くの相互依存関係を含みます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="9f5e4-131">事前インストール済みのイメージがあると、データ サイエンティストは環境の問題のトラブルシューティングに費やす時間を短縮でき、実行する必要のあるデータの探索およびモデリング タスクに集中することができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="9f5e4-132">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-132">Key benefits:</span></span>
* <span data-ttu-id="9f5e4-133">データ サイエンス ツールおよびフレームワークのインストール、管理、トラブルシューティングに要する時間を短縮できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
* <span data-ttu-id="9f5e4-134">よく使用されるツールとフレームワークの最新バージョンがすべて含まれています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
* <span data-ttu-id="9f5e4-135">仮想マシンのオプションには、集中的なデータ モデリングのための GPU 機能を備えたスケーラブルなイメージが含まれます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="9f5e4-136">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-136">Considerations:</span></span>
* <span data-ttu-id="9f5e4-137">オフラインのときは仮想マシンにアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-137">The virtual machine cannot be accessed when offline.</span></span>
* <span data-ttu-id="9f5e4-138">仮想マシンの実行には Azure の料金が発生するため、必要な場合にのみ実行するように注意する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="9f5e4-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="9f5e4-139">Azure Machine Learning</span></span>

<span data-ttu-id="9f5e4-140">Azure Machine Learning は、機械学習の実験とモデルを管理するためのクラウドベース サービスです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="9f5e4-141">データの準備状況とモデリング トレーニング スクリプトを追跡する実験サービスが含まれており、イテレーション間でモデルのパフォーマンスを比較できるようにすべての実行履歴が保持されます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="9f5e4-142">Azure Machine Learning Workbench という名前のクロスプラットフォーム クライアント ツールは、データ サイエンティストが Jupyter Notebook や Visual Studio Code などの選択したツールでスクリプトを作成できるようにしたまま、スクリプトの管理および履歴用の一元的なインターフェイスを提供します。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-142">A cross-platform client tool named Azure Machine Learning Workbench provides a central interface for script management and history, while still enabling data scientists to create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code.</span></span>

<span data-ttu-id="9f5e4-143">Azure Machine Learning Workbench から、対話型のデータ準備ツールを使用して一般的なデータ変換タスクを簡略化でき、スケーラブルな Docker コンテナーまたは Spark 内でモデル トレーニング スクリプトをローカルに実行するスクリプト実行環境を構成することができますす。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-143">From Azure Machine Learning Workbench, you can use the interactive data preparation tools to simplify common data transformation tasks, and you can configure the script execution environment to run model training scripts locally, in a scalable Docker container, or in Spark.</span></span>

<span data-ttu-id="9f5e4-144">モデルをデプロイする準備ができたら、Workbench 環境を使用してモデルをパッケージ化し、Docker コンテナー、Azure HDinsight 上の Spark、Microsoft Machine Learning Server、または SQL Server に Web サービスとしてデプロイします。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-144">When you are ready to deploy your model, use the Workbench environment to package the model and deploy it as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="9f5e4-145">Azure Machine Learning モデル管理サービスを使用すると、クラウド内、エッジ デバイス上、または企業全体にわたってモデルのデプロイを追跡および管理できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-145">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="9f5e4-146">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-146">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-147">スクリプトと実行履歴を一元管理して、モデルのバージョンを簡単に比較できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-147">Central management of scripts and run history, making it easy to compare model versions.</span></span>
* <span data-ttu-id="9f5e4-148">ビジュアル エディターを使用して対話的にデータ変換できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-148">Interactive data transformation through a visual editor.</span></span>
* <span data-ttu-id="9f5e4-149">クラウドまたはエッジ デバイスへのモデルのデプロイと管理が簡単です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-149">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="9f5e4-150">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-150">Considerations:</span></span>
* <span data-ttu-id="9f5e4-151">モデル管理モデルと Workbench ツール環境について、ある程度の知識が必要です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-151">Requires some familiarity with the model management model and Workbench tool environment.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="9f5e4-152">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="9f5e4-152">Azure Batch AI</span></span>

<span data-ttu-id="9f5e4-153">Azure Batch AI を使用すると、機械学習の実験を並列に実行でき、GPU を搭載した仮想マシンのクラスター全体にわたって大規模なモデル トレーニングを実行できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-153">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="9f5e4-154">Batch AI トレーニングを使用すると、Cognitive Toolkit、Caffe、Chainer、TensorFlow などのフレームワークを使用して、クラスター化された GPU にディープ ラーニング ジョブをスケールアウトできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-154">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span> 

<span data-ttu-id="9f5e4-155">Azure Machine Learning モデル管理を使用すると、Batch AI トレーニングのモデルをデプロイ、管理、監視できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-155">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span> 

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="9f5e4-156">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="9f5e4-156">Azure Machine Learning Studio</span></span>

<span data-ttu-id="9f5e4-157">Azure Machine Learning Studio は、データ実験を作成し、機械学習モデルをトレーニングして、Azure の Web サービスとして公開するためのクラウドベースのビジュアル開発環境です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-157">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="9f5e4-158">そのビジュアル ドラッグ アンド ドロップ インターフェイスは、カスタムの R および Python ロジック、機械学習モデリング タスク用の確立されたさまざまな統計的アルゴリズムと手法、Jupyter Notebook の組み込みサポートをサポートしながら、データ サイエンティストおよびパワー ユーザーが機械学習ソリューションをすばやく作成できるようにします。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-158">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="9f5e4-159">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-159">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-160">対話型のビジュアル インターフェイスにより、最小限のコードで機械学習モデリングを行うことができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-160">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
* <span data-ttu-id="9f5e4-161">データ探索用の組み込み Jupyter Notebook。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-161">Built-in Jupyter Notebooks for data exploration.</span></span>
* <span data-ttu-id="9f5e4-162">トレーニング済みモデルを Azure Web サービスとして直接デプロイできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-162">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="9f5e4-163">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-163">Considerations:</span></span>

* <span data-ttu-id="9f5e4-164">スケーラビリティが限定的です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-164">Limited scalability.</span></span> <span data-ttu-id="9f5e4-165">トレーニング データセットの最大サイズは 10 GB です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-165">The maximum size of a training dataset is 10 GB.</span></span>
* <span data-ttu-id="9f5e4-166">オンラインのみです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-166">Online only.</span></span> <span data-ttu-id="9f5e4-167">オフライン開発環境はありません。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-167">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="9f5e4-168">機械学習モデルをデプロイするためのツールとサービス</span><span class="sxs-lookup"><span data-stu-id="9f5e4-168">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="9f5e4-169">データ サイエンティストが機械学習モデルを作成した後、通常はそのモデルをデプロイし、アプリケーションまたは他のデータ フローで使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-169">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="9f5e4-170">機械学習モデルにはいくつかの潜在的なデプロイ ターゲットがあります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-170">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="9f5e4-171">Azure HDInsight での Spark</span><span class="sxs-lookup"><span data-stu-id="9f5e4-171">Spark on Azure HDInsight</span></span>

<span data-ttu-id="9f5e4-172">Apache Spark には、機械学習モデル用のフレームワークおよびライブラリである Spark MLlib が含まれています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-172">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="9f5e4-173">Microsoft Machine Learning library for Spark (MMLSpark) には、Spark の予測モデルのディープ ラーニング アルゴリズムのサポートも用意されています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-173">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="9f5e4-174">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-174">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-175">Spark は、大量の機械学習のプロセスに高いスケーラビリティを提供する分散型プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-175">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
* <span data-ttu-id="9f5e4-176">Azure Machine Learning Workbench から HDinsight の Spark にモデルを直接デプロイし、Azure Machine Learning モデル管理サービスを使用して管理することができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-176">You can deploy models directly to Spark in HDinsight from Azure Machine Learning Workbench, and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="9f5e4-177">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-177">Considerations:</span></span>

* <span data-ttu-id="9f5e4-178">Spark は、実行されている時間全体に対して料金が発生する HDinsght クラスターで実行されます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-178">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="9f5e4-179">機械学習サービスをときどき使用するだけの場合は、不要なコストが発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-179">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="azure-databricks"></a><span data-ttu-id="9f5e4-180">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="9f5e4-180">Azure Databricks</span></span>

<span data-ttu-id="9f5e4-181">[Azure Databricks](/azure/azure-databricks/) は、Apache Spark ベースの分析プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-181">[Azure Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform.</span></span> <span data-ttu-id="9f5e4-182">"サービスとしての Spark" と考えることができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-182">You can think of it as "Spark as a service."</span></span> <span data-ttu-id="9f5e4-183">Azure プラットフォームで Spark を使用する最も簡単な方法です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-183">It's the easiest way to use Spark on the Azure platform.</span></span> <span data-ttu-id="9f5e4-184">機械学習には、[MLFlow](https://www.mlflow.org/)、[Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html)、Apache Spark MLlib などを使用できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-184">For machine learning, you can use [MLFlow](https://www.mlflow.org/), [Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib, and others.</span></span> <span data-ttu-id="9f5e4-185">詳細については、[Azure Databricks の Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html) に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-185">For more information, see [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span></span> 

### <a name="web-service-in-a-container"></a><span data-ttu-id="9f5e4-186">コンテナー内の Web サービス</span><span class="sxs-lookup"><span data-stu-id="9f5e4-186">Web service in a container</span></span>

<span data-ttu-id="9f5e4-187">Docker コンテナーで機械学習モデルを Python Web サービスとしてデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-187">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="9f5e4-188">Azure またはエッジ デバイスにモデルをデプロイでき、操作するデータをローカルに使用できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-188">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="9f5e4-189">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-189">Key Benefits:</span></span>

* <span data-ttu-id="9f5e4-190">コンテナーは軽量で、一般にサービスをパッケージ化およびデプロイするためのコスト効果の高い方法です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-190">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
* <span data-ttu-id="9f5e4-191">エッジ デバイスにデプロイできるので、予測ロジックをデータの近くに移動することができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-191">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>
* <span data-ttu-id="9f5e4-192">Azure Machine Learning Workbench からコンテナーを直接デプロイできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-192">You can deploy to a container directly from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="9f5e4-193">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-193">Considerations:</span></span>

* <span data-ttu-id="9f5e4-194">このデプロイ モデルは Docker コンテナーに基づいているため、この方法で Web サービスをデプロイする前に、このテクノロジについて理解しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-194">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="9f5e4-195">Microsoft Machine Learning Server</span><span class="sxs-lookup"><span data-stu-id="9f5e4-195">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="9f5e4-196">Microsoft Learning Server (旧称 Microsoft R Server) は、R と Python のコード用のスケーラブルなプラットフォームで、機械学習シナリオ用に設計されています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-196">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="9f5e4-197">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-197">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-198">高いスケーラビリティ。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-198">High scalability.</span></span>
* <span data-ttu-id="9f5e4-199">Azure Machine Learning Workbench から直接デプロイできます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-199">Direct deployment from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="9f5e4-200">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-200">Considerations:</span></span>

* <span data-ttu-id="9f5e4-201">企業内で Machine Learning Server をデプロイし、管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-201">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="9f5e4-202">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="9f5e4-202">Microsoft SQL Server</span></span>

<span data-ttu-id="9f5e4-203">Microsoft SQL Server では R と Python がネイティブにサポートされるため、これらの言語で作成された機械学習モデルをデータベースに Transact-SQL 関数としてカプセル化できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-203">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="9f5e4-204">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-204">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-205">データベース関数に予測ロジックをカプセル化して、データ層ロジックに簡単に含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-205">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="9f5e4-206">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-206">Considerations:</span></span>

* <span data-ttu-id="9f5e4-207">アプリケーションのデータ層として SQL Server データベースが想定されます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-207">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="9f5e4-208">Azure Machine Learning Web サービス</span><span class="sxs-lookup"><span data-stu-id="9f5e4-208">Azure Machine Learning web service</span></span>

<span data-ttu-id="9f5e4-209">Azure Machine Learning Studio を使用して機械学習モデルを作成するときに、Web サービスとしてデプロイすることができます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-209">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="9f5e4-210">これは、HTTP で通信できる任意のクライアント アプリケーションから REST インターフェイスを通じて使用できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-210">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="9f5e4-211">主な利点:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-211">Key benefits:</span></span>

* <span data-ttu-id="9f5e4-212">開発とデプロイが簡単です。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-212">Ease of development and deployment.</span></span>
* <span data-ttu-id="9f5e4-213">基本的な監視メトリックを備えた Web サービス管理ポータル。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-213">Web service management portal with basic monitoring metrics.</span></span>
* <span data-ttu-id="9f5e4-214">Azure Data Lake Analytics、Azure Data Factory、Azure Stream Analytics からの Azure Machine Learning Web サービスの呼び出しのサポートが組み込まれています。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-214">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="9f5e4-215">考慮事項:</span><span class="sxs-lookup"><span data-stu-id="9f5e4-215">Considerations:</span></span>

* <span data-ttu-id="9f5e4-216">Azure Machine Learning Studio を使用して作成されたモデルに対してのみ使用できます。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-216">Only available for models built using Azure Machine Learning Studio.</span></span>
* <span data-ttu-id="9f5e4-217">Web ベースのアクセス専用で、トレーニング済みモデルをオンプレミスまたはオフラインで実行することはできません。</span><span class="sxs-lookup"><span data-stu-id="9f5e4-217">Web-based access only, trained models cannot run on-premises or offline.</span></span>

