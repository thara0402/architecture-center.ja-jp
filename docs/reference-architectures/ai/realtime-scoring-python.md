---
title: Python モデルのリアルタイム スコアリング
titleSuffix: Azure Reference Architectures
description: この参照アーキテクチャでは、Azure で Python モデルを Web サービスとしてデプロイして、リアルタイムの予測を行う方法を示します。
author: njray
ms.date: 11/09/2018
ms.custom: azcat-ai
ms.openlocfilehash: e2312d1d1d2444f9915f4e6aa067c1487e096d3e
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120358"
---
# <a name="real-time-scoring-of-python-scikit-learn-and-deep-learning-models-on-azure"></a><span data-ttu-id="996ee-103">Azure での Python scikit-learn モデルおよびディープ ラーニング モデルのリアルタイム スコアリング</span><span class="sxs-lookup"><span data-stu-id="996ee-103">Real-time scoring of Python Scikit-Learn and deep learning models on Azure</span></span>

<span data-ttu-id="996ee-104">この参照アーキテクチャでは、Python モデルを Web サービスとしてデプロイして、リアルタイムの予測を行う方法を示します。</span><span class="sxs-lookup"><span data-stu-id="996ee-104">This reference architecture shows how to deploy Python models as web services to make real-time predictions.</span></span> <span data-ttu-id="996ee-105">ここでは 2 つのシナリオについて説明します。1 つは通常の Python モデルのデプロイで、もう 1 つはディープ ラーニング モデルのデプロイという特定の要件です。</span><span class="sxs-lookup"><span data-stu-id="996ee-105">Two scenarios are covered: deploying regular Python models, and the specific requirements of deploying deep learning models.</span></span> <span data-ttu-id="996ee-106">ここで示すアーキテクチャは、両方のシナリオで使用されています。</span><span class="sxs-lookup"><span data-stu-id="996ee-106">Both scenarios use the architecture shown.</span></span>

<span data-ttu-id="996ee-107">このアーキテクチャの 2 つの参照実装は GitHub から入手でき、1 つは[通常の Python モデル][github-python]、もう 1 つ[はディープ ラーニング モデル][github-dl]を対象としています。</span><span class="sxs-lookup"><span data-stu-id="996ee-107">Two reference implementations for this architecture are available on GitHub, one for [regular Python models][github-python] and one for [deep learning models][github-dl].</span></span>

![Azure での Python モデルのリアルタイム スコアリングのアーキテクチャ ダイアグラム](./_images/python-model-architecture.png)

## <a name="scenarios"></a><span data-ttu-id="996ee-109">シナリオ</span><span class="sxs-lookup"><span data-stu-id="996ee-109">Scenarios</span></span>

<span data-ttu-id="996ee-110">参照実装では、このアーキテクチャを使用して 2 つのシナリオを示します。</span><span class="sxs-lookup"><span data-stu-id="996ee-110">The reference implementations demonstrate two scenarios using this architecture.</span></span>

<span data-ttu-id="996ee-111">**シナリオ 1:FAQ 照合**。</span><span class="sxs-lookup"><span data-stu-id="996ee-111">**Scenario 1: FAQ matching**.</span></span> <span data-ttu-id="996ee-112">このシナリオは、よく寄せられる質問 (FAQ) 照合モデルを Web サービスとしてデプロイし、ユーザーの質問を予測する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="996ee-112">This scenario shows how to deploy a frequently asked questions (FAQ) matching model as a web service to provide predictions for user questions.</span></span> <span data-ttu-id="996ee-113">このシナリオでは、アーキテクチャ ダイアグラムの "Input Data" によって、ユーザーの質問を含むテキスト文字列が参照され、FAQ の一覧と照合されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-113">For this scenario, "Input Data" in the architecture diagram refers to text strings containing user questions to match with a list of FAQs.</span></span> <span data-ttu-id="996ee-114">このシナリオは、Python 用の [scikit-learn][scikit] 機械学習ライブラリを対象にしていますが、Python モデルを使用してリアルタイムの予測を行う任意のシナリオ向けに汎用化できます。</span><span class="sxs-lookup"><span data-stu-id="996ee-114">This scenario is designed for the [scikit-learn][scikit] machine learning library for Python, but can be generalized to any scenario that uses Python models to make real-time predictions.</span></span>

<span data-ttu-id="996ee-115">このシナリオでは、JavaScript としてタグ付けされた元の質問、重複する質問、およびその回答を含む Stack Overflow 質問データのサブセットが使用されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-115">This scenario uses a subset of Stack Overflow question data that includes original questions tagged as JavaScript, their duplicate questions, and their answers.</span></span> <span data-ttu-id="996ee-116">元の質問それぞれについて、重複する質問と一致する可能性を予測するように scikit-learn パイプラインがトレーニングされます。</span><span class="sxs-lookup"><span data-stu-id="996ee-116">It trains a scikit-learn pipeline to predict the match probability of a duplicate question with each of the original questions.</span></span> <span data-ttu-id="996ee-117">これらの予測は、REST API エンドポイントを使用してリアルタイムで行われます。</span><span class="sxs-lookup"><span data-stu-id="996ee-117">These predictions are made in real time using a REST API  endpoint.</span></span>

<span data-ttu-id="996ee-118">このアーキテクチャのアプリケーション フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="996ee-118">The application flow for this architecture is as follows:</span></span>

1. <span data-ttu-id="996ee-119">クライアントは、HTTP POST 要求を、エンコード済み質問データと一緒に送信します。</span><span class="sxs-lookup"><span data-stu-id="996ee-119">The client sends an HTTP POST request with the encoded question data.</span></span>

2. <span data-ttu-id="996ee-120">Flask アプリは、その要求から質問を抽出します。</span><span class="sxs-lookup"><span data-stu-id="996ee-120">The Flask app extracts the question from the request.</span></span>

3. <span data-ttu-id="996ee-121">質問は scikit-learn パイプライン モデルに送信され、特徴付けとスコアリングが行われます。</span><span class="sxs-lookup"><span data-stu-id="996ee-121">The question is sent to the scikit-learn pipeline model for featurization and scoring.</span></span>

4. <span data-ttu-id="996ee-122">一致する FAQ の質問とそのスコアは JSON オブジェクトにパイプされ、クライアントに返されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-122">The matching FAQ questions with their scores are piped into a JSON object and returned to the client.</span></span>

<span data-ttu-id="996ee-123">結果を使用するサンプル アプリのスクリーンショットを次に示します。</span><span class="sxs-lookup"><span data-stu-id="996ee-123">Here is a screenshot of the example app that consumes the results:</span></span>

![サンプル アプリのスクリーンショット](./_images/python-faq-matches.png)

<span data-ttu-id="996ee-125">**シナリオ 2:画像の分類**。</span><span class="sxs-lookup"><span data-stu-id="996ee-125">**Scenario 2: Image classification**.</span></span> <span data-ttu-id="996ee-126">このシナリオは、畳み込みニューラル ネットワーク (CNN) モデルを Web サービスとしてデプロイし、画像に関する予測を提供する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="996ee-126">This scenario shows how to deploy a Convolutional Neural Network (CNN) model as a web service to provide predictions on images.</span></span> <span data-ttu-id="996ee-127">このシナリオでは、アーキテクチャ ダイアグラムの "Input Data" によって、画像ファイルが参照されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-127">For this scenario, "Input Data" in the architecture diagram refers to image files.</span></span> <span data-ttu-id="996ee-128">画像の分類やオブジェクト検出などのタスク向けのコンピューター ビジョンでは、CNN がきわめて効果的です。</span><span class="sxs-lookup"><span data-stu-id="996ee-128">CNNs are very effective in computer vision for tasks such as image classification and object detection.</span></span> <span data-ttu-id="996ee-129">このシナリオは、TensorFlow、Keras (TensorFlow バックエンドを含む)、および PyTorch フレームワークを対象にしています。</span><span class="sxs-lookup"><span data-stu-id="996ee-129">This scenario is designed for the frameworks TensorFlow, Keras (with the TensorFlow back end), and PyTorch.</span></span> <span data-ttu-id="996ee-130">ただし、ディープ ラーニング モデルを使用してリアルタイムの予測を行う任意のシナリオ向けに汎用化できます。</span><span class="sxs-lookup"><span data-stu-id="996ee-130">However, it can be generalized to any scenario that uses deep learning models to make real-time predictions.</span></span>

<span data-ttu-id="996ee-131">このシナリオでは、ImageNet-1K (1,000 クラス) データセットで事前にトレーニングされた ResNet-152 モデルを使用して、画像がどのカテゴリに属すかを予測します (以下の図参照)。</span><span class="sxs-lookup"><span data-stu-id="996ee-131">This scenario uses a pre-trained ResNet-152 model trained on ImageNet-1K (1,000 classes) dataset to predict which category (see figure below) an image belongs to.</span></span> <span data-ttu-id="996ee-132">これらの予測は、REST API エンドポイントを使用してリアルタイムで行われます。</span><span class="sxs-lookup"><span data-stu-id="996ee-132">These predictions are made in real time using a REST API endpoint.</span></span>

![予測の例](./_images/python-example-predictions.png)

<span data-ttu-id="996ee-134">ディープ ラーニング モデルのアプリケーション フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="996ee-134">The application flow for the deep learning model is as follows:</span></span>

1. <span data-ttu-id="996ee-135">クライアントは、HTTP POST 要求を、エンコード済み画像データと一緒に送信します。</span><span class="sxs-lookup"><span data-stu-id="996ee-135">The client sends an HTTP POST request with the encoded image data.</span></span>

2. <span data-ttu-id="996ee-136">Flask アプリは、その要求から画像を抽出します。</span><span class="sxs-lookup"><span data-stu-id="996ee-136">The Flask app extracts the image from the request.</span></span>

3. <span data-ttu-id="996ee-137">画像は前処理されてモデルに送信され、スコアリングが行われます。</span><span class="sxs-lookup"><span data-stu-id="996ee-137">The image is preprocessed and sent to the model for scoring.</span></span>

4. <span data-ttu-id="996ee-138">スコアリングの結果は JSON オブジェクトにパイプされ、クライアントに返されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-138">The scoring result is piped into a JSON object and returned to the client.</span></span>

## <a name="architecture"></a><span data-ttu-id="996ee-139">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="996ee-139">Architecture</span></span>

<span data-ttu-id="996ee-140">このアーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-140">This architecture consists of the following components.</span></span>

<span data-ttu-id="996ee-141">**[仮想マシン][vm]** (VM)。</span><span class="sxs-lookup"><span data-stu-id="996ee-141">**[Virtual machine][vm]** (VM).</span></span> <span data-ttu-id="996ee-142">VM は、HTTP 要求を送信できる &mdash;ローカルまたはクラウドの&mdash;デバイスの例として示されています。</span><span class="sxs-lookup"><span data-stu-id="996ee-142">The VM is shown as an example of a device &mdash; local or in the cloud &mdash; that can send an HTTP request.</span></span>

<span data-ttu-id="996ee-143">**[Azure Kubernetes Service][aks]** (AKS) は、アプリケーションを Kubernetes クラスターにデプロイするときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-143">**[Azure Kubernetes Service][aks]** (AKS) is used to deploy the application on a Kubernetes cluster.</span></span> <span data-ttu-id="996ee-144">AKS により、Kubernetes のデプロイと操作が簡略化されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-144">AKS simplifies the deployment and operations of Kubernetes.</span></span> <span data-ttu-id="996ee-145">クラスターを構成する場合、通常の Python モデルについては CPU のみの VM を使用し、ディープ ラーニング モデルについては GPU 対応 VM を使用します。</span><span class="sxs-lookup"><span data-stu-id="996ee-145">The cluster can be configured using CPU-only VMs for regular Python models or GPU-enabled VMs for deep learning models.</span></span>

<span data-ttu-id="996ee-146">**[ロード バランサー][lb]**。</span><span class="sxs-lookup"><span data-stu-id="996ee-146">**[Load balancer][lb]**.</span></span> <span data-ttu-id="996ee-147">サービスを外部に公開するときに、AKS によってプロビジョニングされたロード バランサーが使用されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-147">A load balancer, provisioned by AKS, is used to expose the service externally.</span></span> <span data-ttu-id="996ee-148">ロード バランサーからのトラフィックは、バックエンド ポッドに送信されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-148">Traffic from the load balancer is directed to the back-end pods.</span></span>

<span data-ttu-id="996ee-149">**[Docker Hub][docker]** は、Kubernetes クラスターにデプロイされている Docker イメージを格納するときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-149">**[Docker Hub][docker]** is used to store the Docker image that is deployed on Kubernetes cluster.</span></span> <span data-ttu-id="996ee-150">Docker Hub は、使いやすく、Docker ユーザーに対する既定のイメージ リポジトリであるため、このアーキテクチャに選択されました。</span><span class="sxs-lookup"><span data-stu-id="996ee-150">Docker Hub was chosen for this architecture because it's easy to use and is the default image repository for Docker users.</span></span> <span data-ttu-id="996ee-151">[Azure Container Registry][acr] を、このアーキテクチャに使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="996ee-151">[Azure Container Registry][acr] can also be used for this architecture.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="996ee-152">パフォーマンスに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="996ee-152">Performance considerations</span></span>

<span data-ttu-id="996ee-153">リアルタイム スコアリング アーキテクチャでは、スループット パフォーマンスが主な考慮事項になります。</span><span class="sxs-lookup"><span data-stu-id="996ee-153">For real-time scoring architectures, throughput performance becomes a dominant consideration.</span></span> <span data-ttu-id="996ee-154">通常の Python モデルの場合は、CPU で十分にワークロードを処理できると一般に考えられています。</span><span class="sxs-lookup"><span data-stu-id="996ee-154">For regular Python models, it's generally accepted that CPUs are sufficient to handle the workload.</span></span>

<span data-ttu-id="996ee-155">しかし、ディープ ラーニングのワークロードで速度がボトルネックになっているときは、一般的に CPU よりも GPU の方が優れた[パフォーマンス][gpus-vs-cpus]を発揮します。</span><span class="sxs-lookup"><span data-stu-id="996ee-155">However for deep learning workloads, when speed is a bottleneck, GPUs generally provide better [performance][gpus-vs-cpus] compared to CPUs.</span></span> <span data-ttu-id="996ee-156">CPU を使用して GPU に相当するパフォーマンスを実現するには、通常、多数の CPU を備えたクラスターが必要です。</span><span class="sxs-lookup"><span data-stu-id="996ee-156">To match GPU performance using CPUs, a cluster with large number of CPUs is usually needed.</span></span>

<span data-ttu-id="996ee-157">どちらのシナリオでもこのアーキテクチャには CPU を使用できますが、ディープ ラーニング モデルの場合、GPU が提供するスループット値は、同じコストの CPU クラスターに比べてかなり高くなります。</span><span class="sxs-lookup"><span data-stu-id="996ee-157">You can use CPUs for this architecture in either scenario, but for deep learning models, GPUs provide significantly higher throughput values compared to a CPU cluster of similar cost.</span></span> <span data-ttu-id="996ee-158">AKS では GPU の使用がサポートされています。これは、このアーキテクチャに AKS を使用するメリットの 1 つです。</span><span class="sxs-lookup"><span data-stu-id="996ee-158">AKS supports the use of GPUs, which is one advantage of using AKS for this architecture.</span></span> <span data-ttu-id="996ee-159">また、ディープ ラーニング デプロイで使用されるモデルには、通常、多数のパラメーターが含まれます。</span><span class="sxs-lookup"><span data-stu-id="996ee-159">Also, deep learning deployments typically use models with a high number of parameters.</span></span> <span data-ttu-id="996ee-160">GPU を使用することで、CPU のみのデプロイで問題になる、モデルと Web サービスの間のリソース競合を防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="996ee-160">Using GPUs prevents contention for resources between the model and the web service, which is an issue in CPU-only deployments.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="996ee-161">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="996ee-161">Scalability considerations</span></span>

<span data-ttu-id="996ee-162">CPU のみの VM で AKS クラスターがプロビジョニングされている通常の Python モデルの場合、[ポッドの数をスケールアウト][manually-scale-pods]するときに注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="996ee-162">For regular Python models, where AKS cluster is provisioned with CPU-only VMs, take care when [scaling out the number of pods][manually-scale-pods].</span></span> <span data-ttu-id="996ee-163">目標は、クラスターを完全に利用することです。</span><span class="sxs-lookup"><span data-stu-id="996ee-163">The goal is to fully utilize the cluster.</span></span> <span data-ttu-id="996ee-164">スケーリングは、CPU 要求と、ポッドに対して定義されている制限によって左右されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-164">Scaling depends on the CPU requests and limits defined for the pods.</span></span> <span data-ttu-id="996ee-165">また、Kubernetes ではポッドの[自動スケーリング][autoscale-pods]もサポートされ、CPU 使用率などの選ばれたメトリックに応じて、デプロイのポッド数が調整されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-165">Kubernetes also supports [autoscaling][autoscale-pods] of the pods to adjust the number of pods in a deployment depending on CPU utilization or other select metrics.</span></span> <span data-ttu-id="996ee-166">[クラスター オートスケーラー][autoscaler] (プレビュー) は、保留中のポッドに基づいてエージェント ノードをスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="996ee-166">The [cluster autoscaler][autoscaler] (in preview) can scale agent nodes based on pending pods.</span></span>

<span data-ttu-id="996ee-167">GPU 対応 VM を使用しているディープ ラーニング シナリオでは、ポッドのリソース制限は、1 つのポッドに 1 つの GPU を割り当てる、といったものです。</span><span class="sxs-lookup"><span data-stu-id="996ee-167">For deep learning scenarios, using GPU-enabled VMs, resource limits on pods are such that one GPU is assigned to one pod.</span></span> <span data-ttu-id="996ee-168">使用する VM の種類に応じて[クラスターのノードをスケーリング][scale-cluster]して、サービスの需要に対応する必要があります。</span><span class="sxs-lookup"><span data-stu-id="996ee-168">Depending on the type of VM used, you must [scale the nodes of the cluster][scale-cluster] to meet the demand for the service.</span></span> <span data-ttu-id="996ee-169">これを簡単に行うには、Azure CLI と kubectl を使用します。</span><span class="sxs-lookup"><span data-stu-id="996ee-169">You can do this easily using the Azure CLI and kubectl.</span></span>

## <a name="monitoring-and-logging-considerations"></a><span data-ttu-id="996ee-170">監視とログ記録に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="996ee-170">Monitoring and logging considerations</span></span>

### <a name="aks-monitoring"></a><span data-ttu-id="996ee-171">AKS の監視</span><span class="sxs-lookup"><span data-stu-id="996ee-171">AKS monitoring</span></span>

<span data-ttu-id="996ee-172">AKS のパフォーマンスを可視化するには、[コンテナーに対する Azure Monitor][monitor-containers] 機能を使用します。</span><span class="sxs-lookup"><span data-stu-id="996ee-172">For visibility into AKS performance, use the [Azure Monitor for containers][monitor-containers] feature.</span></span> <span data-ttu-id="996ee-173">この機能により、Kubernetes で使用可能なコントローラー、ノード、およびコンテナーから、メモリやプロセッサのメトリックが Metrics API 経由で収集されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-173">It collects memory and processor metrics from controllers, nodes, and containers that are available in Kubernetes through the Metrics API.</span></span>

<span data-ttu-id="996ee-174">アプリケーションのデプロイ中、AKS クラスターを監視して、想定どおりに動作していること、すべてのノードが運用可能で、すべてのポッドが実行中であることを確認します。</span><span class="sxs-lookup"><span data-stu-id="996ee-174">While deploying your application, monitor the AKS cluster to make sure it's working as expected, all the nodes are operational, and all pods are running.</span></span> <span data-ttu-id="996ee-175">[kubectl][kubectl] コマンドライン ツールを使用してポッドの状態を取得できますが、Kubernetes にも、クラスターの状態の基本的な監視と管理のための Web ダッシュボードがあります。</span><span class="sxs-lookup"><span data-stu-id="996ee-175">Although you can use the [kubectl][kubectl] command-line tool to retrieve pod status, Kubernetes also includes a web dashboard for basic monitoring of the cluster status and management.</span></span>

![Kubernetes ダッシュボードのスクリーンショット](./_images/python-kubernetes-dashboard.png)

<span data-ttu-id="996ee-177">クラスターとノードの全体的な状態を確認するには、Kubernetes ダッシュボードの **[ノード]** セクションに移動します。</span><span class="sxs-lookup"><span data-stu-id="996ee-177">To see the overall state of the cluster and nodes, go to the **Nodes** section of the Kubernetes dashboard.</span></span> <span data-ttu-id="996ee-178">ノードが非アクティブか失敗した場合は、そのページからエラー ログを表示できます。</span><span class="sxs-lookup"><span data-stu-id="996ee-178">If a node is inactive or has failed, you can display the error logs from that page.</span></span> <span data-ttu-id="996ee-179">同様に、ポッドの数やデプロイの状態に関する情報を確認するには、**[ポッド]** セクションおよび **[デプロイ]** セクションに移動します。</span><span class="sxs-lookup"><span data-stu-id="996ee-179">Similarly, go to the **Pods** and **Deployments** sections for information about the number of pods and status of your deployment.</span></span>

### <a name="aks-logs"></a><span data-ttu-id="996ee-180">AKS のログ</span><span class="sxs-lookup"><span data-stu-id="996ee-180">AKS logs</span></span>

<span data-ttu-id="996ee-181">AKS では、すべての stdout と stderr が、クラスター内のポッドのログに自動的に記録されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-181">AKS automatically logs all stdout/stderr to the logs of the pods in the cluster.</span></span> <span data-ttu-id="996ee-182">kubectl を使用すると、これらのログのほか、ノード レベルのイベントおよびログも確認できます。</span><span class="sxs-lookup"><span data-stu-id="996ee-182">Use kubectl to see these and also node-level events and logs.</span></span> <span data-ttu-id="996ee-183">詳細については、デプロイの手順を確認してください。</span><span class="sxs-lookup"><span data-stu-id="996ee-183">For details, see the deployment steps.</span></span>

<span data-ttu-id="996ee-184">[コンテナー用の Azure Monitor][monitor-containers] を使用して、コンテナー化されたバージョンの Linux 用 Log Analytics エージェントを介してメトリックとログを収集します。こうしたメトリックやログは、Log Analytics ワークスペースに保存されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-184">Use [Azure Monitor for containers][monitor-containers] to collect metrics and logs through a containerized version of the Log Analytics agent for Linux, which is stored in your Log Analytics workspace.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="996ee-185">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="996ee-185">Security considerations</span></span>

<span data-ttu-id="996ee-186">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="996ee-186">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="996ee-187">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。ただし、AKS エージェント ノードは監視されません。</span><span class="sxs-lookup"><span data-stu-id="996ee-187">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment, although it doesn't monitor AKS agent nodes.</span></span> <span data-ttu-id="996ee-188">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-188">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="996ee-189">[Azure サブスクリプションでの Security Center Standard の利用開始][get-started]に関するページの説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="996ee-189">Enable security data collection as described in [Onboard your Azure subscription to Security Center Standard][get-started].</span></span> <span data-ttu-id="996ee-190">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="996ee-190">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="996ee-191">**操作**。</span><span class="sxs-lookup"><span data-stu-id="996ee-191">**Operations**.</span></span> <span data-ttu-id="996ee-192">Azure Active Directory (Azure AD) 認証トークンを使用して AKS クラスターにサインインするには、[ユーザー認証][aad-auth]に Azure AD を使用するように AKS を構成します。</span><span class="sxs-lookup"><span data-stu-id="996ee-192">To sign in to an AKS cluster using your Azure Active Directory (Azure AD) authentication token, configure AKS to use Azure AD for [user authentication][aad-auth].</span></span> <span data-ttu-id="996ee-193">また、クラスター管理者が、ユーザーの ID またはディレクトリ グループ メンバーシップに基づいて、Kubernetes のロールベースのアクセス制御 (RBAC) を構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="996ee-193">Cluster administrators can also configure Kubernetes role-based access control (RBAC) based on a user's identity or directory group membership.</span></span>

<span data-ttu-id="996ee-194">[RBAC][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御してください。</span><span class="sxs-lookup"><span data-stu-id="996ee-194">Use [RBAC][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="996ee-195">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="996ee-195">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="996ee-196">ユーザーを複数の役割に割り当てることができ、よりきめ細かい[アクセス許可]のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="996ee-196">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained [permissions].</span></span>

<span data-ttu-id="996ee-197">**HTTPS**。</span><span class="sxs-lookup"><span data-stu-id="996ee-197">**HTTPS**.</span></span> <span data-ttu-id="996ee-198">セキュリティのベスト プラクティスとして、アプリケーションでは HTTPS を強制して、HTTP 要求をリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="996ee-198">As a security best practice, the application should enforce HTTPS and redirect HTTP requests.</span></span> <span data-ttu-id="996ee-199">SSL を終了して HTTP 要求をリダイレクトするリバース プロキシをデプロイするには、[イングレス コントローラー][ingress-controller]を使用します。</span><span class="sxs-lookup"><span data-stu-id="996ee-199">Use an [ingress controller][ingress-controller] to deploy a reverse proxy that terminates SSL and redirects HTTP requests.</span></span> <span data-ttu-id="996ee-200">詳細については、「[Azure Kubernetes Service (AKS) で HTTPS イングレス コントローラーを作成する][https-ingress]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="996ee-200">For more information, see [Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)][https-ingress].</span></span>

<span data-ttu-id="996ee-201">**[認証]**: </span><span class="sxs-lookup"><span data-stu-id="996ee-201">**Authentication**.</span></span> <span data-ttu-id="996ee-202">このソリューションでは、エンドポイントへのアクセスは制限されません。</span><span class="sxs-lookup"><span data-stu-id="996ee-202">This solution doesn't restrict access to the endpoints.</span></span> <span data-ttu-id="996ee-203">エンタープライズ設定でアーキテクチャをデプロイするには、API キーによってエンドポイントをセキュリティで保護し、何らかの形式のユーザー認証をクライアント アプリケーションに追加します。</span><span class="sxs-lookup"><span data-stu-id="996ee-203">To deploy the architecture in an enterprise setting, secure the endpoints through API keys and add some form of user authentication to the client application.</span></span>

<span data-ttu-id="996ee-204">**コンテナー レジストリ**。</span><span class="sxs-lookup"><span data-stu-id="996ee-204">**Container registry**.</span></span> <span data-ttu-id="996ee-205">このソリューションでは、Docker イメージの格納にパブリック レジストリが使用されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-205">This solution uses a public registry to store the Docker image.</span></span> <span data-ttu-id="996ee-206">アプリケーションが依存するコード、およびモデルは、このイメージ内に含まれています。</span><span class="sxs-lookup"><span data-stu-id="996ee-206">The code that the application depends on, and the model, are contained within this image.</span></span> <span data-ttu-id="996ee-207">エンタープライズ アプリケーションでは、悪意のあるコードが実行されるのを防ぎ、コンテナー内の情報が侵害されないようにするために、プライベート レジストリを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="996ee-207">Enterprise applications should use a private registry to help guard against running malicious code and to help keep the information inside the container from being compromised.</span></span>

<span data-ttu-id="996ee-208">**DDoS 保護**。</span><span class="sxs-lookup"><span data-stu-id="996ee-208">**DDoS protection**.</span></span> <span data-ttu-id="996ee-209">[DDoS Protection Standard][ddos] を有効にすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="996ee-209">Consider enabling [DDoS Protection Standard][ddos].</span></span> <span data-ttu-id="996ee-210">Azure プラットフォームの一部として基本な DDoS 保護が有効になりますが、DDoS Protection Standard により、特に Azure 仮想ネットワークのリソース向けにチューニングされた軽減機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="996ee-210">Although basic DDoS protection is enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure virtual network resources.</span></span>

<span data-ttu-id="996ee-211">**ログの記録**。</span><span class="sxs-lookup"><span data-stu-id="996ee-211">**Logging**.</span></span> <span data-ttu-id="996ee-212">ログ データを格納する前にベスト プラクティスを使用します。たとえば、セキュリティ不正アクセスに使用できるユーザー パスワードなどの情報を除去します。</span><span class="sxs-lookup"><span data-stu-id="996ee-212">Use best practices before storing log data, such as scrubbing user passwords and other information that could be used to commit security fraud.</span></span>

## <a name="deployment"></a><span data-ttu-id="996ee-213">Deployment</span><span class="sxs-lookup"><span data-stu-id="996ee-213">Deployment</span></span>

<span data-ttu-id="996ee-214">この参照アーキテクチャを展開するには、GitHub リポジトリで説明されている手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="996ee-214">To deploy this reference architecture, follow the steps described in the GitHub repos:</span></span>

- <span data-ttu-id="996ee-215">[通常の Python のモデル][github-python]</span><span class="sxs-lookup"><span data-stu-id="996ee-215">[Regular Python models][github-python]</span></span>
- <span data-ttu-id="996ee-216">[ディープ ラーニング モデル][github-dl]</span><span class="sxs-lookup"><span data-stu-id="996ee-216">[Deep learning models][github-dl]</span></span>

<!-- links -->

[aad-auth]: /azure/aks/aad-integration
[acr]: /azure/container-registry/
[something]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
[aks]: /azure/aks/intro-kubernetes
[autoscaler]: /azure/aks/autoscaler
[autoscale-pods]: /azure/aks/tutorial-kubernetes-scale#autoscale-pods
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[ddos]: /azure/virtual-network/ddos-protection-overview
[docker]: https://hub.docker.com/
[get-started]: /azure/security-center/security-center-get-started
[github-python]: https://github.com/Azure/MLAKSDeployment
[github-dl]: https://github.com/Microsoft/AKSDeploymentTutorial
[gpus-vs-cpus]: https://azure.microsoft.com/en-us/blog/gpus-vs-cpus-for-deployment-of-deep-learning-models/
[https-ingress]: /azure/aks/ingress-tls
[ingress-controller]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[lb]: /azure/load-balancer/load-balancer-overview
[manually-scale-pods]: /azure/aks/tutorial-kubernetes-scale#manually-scale-pods
[monitor-containers]: /azure/monitoring/monitoring-container-insights-overview
[アクセス許可]: /azure/aks/concepts-identity
[permissions]: /azure/aks/concepts-identity
[rbac]: /azure/active-directory/role-based-access-control-what-is
[scale-cluster]: /azure/aks/scale-cluster
[scikit]: https://pypi.org/project/scikit-learn/
[security-center]: /azure/security-center/security-center-intro
[vm]: /azure/virtual-machines/
