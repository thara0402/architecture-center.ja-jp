---
title: Azure での Python scikit-learn モデルおよびディープ ラーニング モデルのリアルタイム スコアリング
description: この参照アーキテクチャでは、Azure で Python モデルを Web サービスとしてデプロイして、リアルタイムの予測を行う方法を示します。
author: njray
ms.date: 11/09/2018
ms.author: njray
ms.openlocfilehash: 860eb83f248c2a9f2a96065c6c4cdd02715e7ce1
ms.sourcegitcommit: cc234a522b7fc35af3bcacdc044c2e2b529e54ed
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/09/2018
ms.locfileid: "51347772"
---
# <a name="real-time-scoring-of-python-scikit-learn-and-deep-learning-models-on-azure"></a><span data-ttu-id="bc21e-103">Azure での Python scikit-learn モデルおよびディープ ラーニング モデルのリアルタイム スコアリング</span><span class="sxs-lookup"><span data-stu-id="bc21e-103">Real-time scoring of Python Scikit-Learn and Deep Learning Models on Azure</span></span>

<span data-ttu-id="bc21e-104">この参照アーキテクチャでは、Python モデルを Web サービスとしてデプロイして、リアルタイムの予測を行う方法を示します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-104">This reference architecture shows how to deploy Python models as web services to make real-time predictions.</span></span> <span data-ttu-id="bc21e-105">ここでは 2 つのシナリオについて説明します。1 つは通常の Python モデルのデプロイで、もう 1 つはディープ ラーニング モデルのデプロイという特定の要件です。</span><span class="sxs-lookup"><span data-stu-id="bc21e-105">Two scenarios are covered: deploying regular Python models, and the specific requirements of deploying deep learning models.</span></span> <span data-ttu-id="bc21e-106">ここで示すアーキテクチャは、両方のシナリオで使用されています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-106">Both scenarios use the architecture shown.</span></span>

<span data-ttu-id="bc21e-107">このアーキテクチャの 2 つの参照実装は GitHub から入手でき、1 つは[通常の Python モデル][github-python]、もう 1 つ[はディープ ラーニング モデル][github-dl]を対象としています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-107">Two reference implementations for this architecture are available on GitHub, one for [regular Python models][github-python] and one for [deep learning models][github-dl].</span></span>

![](./_images/python-model-architecture.png)

## <a name="scenarios"></a><span data-ttu-id="bc21e-108">シナリオ</span><span class="sxs-lookup"><span data-stu-id="bc21e-108">Scenarios</span></span>

<span data-ttu-id="bc21e-109">参照実装では、このアーキテクチャを使用して 2 つのシナリオを示します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-109">The reference implementations demonstrate two scenarios using this architecture.</span></span>

<span data-ttu-id="bc21e-110">**シナリオ 1: FAQ 照合**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-110">**Scenario 1: FAQ matching**.</span></span> <span data-ttu-id="bc21e-111">このシナリオは、よく寄せられる質問 (FAQ) 照合モデルを Web サービスとしてデプロイし、ユーザーの質問を予測する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-111">This scenario shows how to deploy a frequently asked questions (FAQ) matching model as a web service to provide predictions for user questions.</span></span> <span data-ttu-id="bc21e-112">このシナリオでは、アーキテクチャ ダイアグラムの "Input Data" によって、ユーザーの質問を含むテキスト文字列が参照され、FAQ の一覧と照合されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-112">For this scenario, "Input Data" in the architecture diagram refers to text strings containing user questions to match with a list of FAQs.</span></span> <span data-ttu-id="bc21e-113">このシナリオは、Python 用の [scikit-learn][scikit] 機械学習ライブラリを対象にしていますが、Python モデルを使用してリアルタイムの予測を行う任意のシナリオ向けに汎用化できます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-113">This scenario is designed for the [scikit-learn][scikit] machine learning library for Python, but can be generalized to any scenario that uses Python models to make real-time predictions.</span></span>

<span data-ttu-id="bc21e-114">このシナリオでは、JavaScript としてタグ付けされた元の質問、重複する質問、およびその回答を含む Stack Overflow 質問データのサブセットが使用されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-114">This scenario uses a subset of Stack Overflow question data that includes original questions tagged as JavaScript, their duplicate questions, and their answers.</span></span> <span data-ttu-id="bc21e-115">元の質問それぞれについて、重複する質問と一致する可能性を予測するように scikit-learn パイプラインがトレーニングされます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-115">It trains a scikit-learn pipeline to predict the match probability of a duplicate question with each of the original questions.</span></span> <span data-ttu-id="bc21e-116">これらの予測は、REST API エンドポイントを使用してリアルタイムで行われます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-116">These predictions are made in real time using a REST API  endpoint.</span></span>

<span data-ttu-id="bc21e-117">このアーキテクチャのアプリケーション フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="bc21e-117">The application flow for this architecture is as follows:</span></span>

1.  <span data-ttu-id="bc21e-118">クライアントは、HTTP POST 要求を、エンコード済み質問データと一緒に送信します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-118">The client sends an HTTP POST request with the encoded question data.</span></span>

2.  <span data-ttu-id="bc21e-119">Flask アプリは、その要求から質問を抽出します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-119">The Flask app extracts the question from the request.</span></span>

3.  <span data-ttu-id="bc21e-120">質問は scikit-learn パイプライン モデルに送信され、特徴付けとスコアリングが行われます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-120">The question is sent to the scikit-learn pipeline model for featurization and scoring.</span></span>

4.  <span data-ttu-id="bc21e-121">一致する FAQ の質問とそのスコアは JSON オブジェクトにパイプされ、クライアントに返されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-121">The matching FAQ questions with their scores are piped into a JSON object and returned to the client.</span></span>

<span data-ttu-id="bc21e-122">結果を使用するサンプル アプリのスクリーンショットを次に示します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-122">Here is a screenshot of the example app that consumes the results:</span></span>

![](./_images/python-faq-matches.png)

<span data-ttu-id="bc21e-123">**シナリオ 2: 画像の分類。**</span><span class="sxs-lookup"><span data-stu-id="bc21e-123">**Scenario 2: Image classification.**</span></span> <span data-ttu-id="bc21e-124">このシナリオは、畳み込みニューラル ネットワーク (CNN) モデルを Web サービスとしてデプロイし、画像に関する予測を提供する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-124">This scenario shows how to deploy a Convolutional Neural Network (CNN) model as a web service to provide predictions on images.</span></span> <span data-ttu-id="bc21e-125">このシナリオでは、アーキテクチャ ダイアグラムの "Input Data" によって、画像ファイルが参照されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-125">For this scenario, "Input Data" in the architecture diagram refers to image files.</span></span> <span data-ttu-id="bc21e-126">画像の分類やオブジェクト検出などのタスク向けのコンピューター ビジョンでは、CNN がきわめて効果的です。</span><span class="sxs-lookup"><span data-stu-id="bc21e-126">CNNs are very effective in computer vision for tasks such as image classification and object detection.</span></span> <span data-ttu-id="bc21e-127">このシナリオは、TensorFlow、Keras (TensorFlow バックエンドを含む)、および PyTorch フレームワークを対象にしています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-127">This scenario is designed for the frameworks TensorFlow, Keras (with the TensorFlow back end), and PyTorch.</span></span> <span data-ttu-id="bc21e-128">ただし、ディープ ラーニング モデルを使用してリアルタイムの予測を行う任意のシナリオ向けに汎用化できます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-128">However, it can be generalized to any scenario that uses deep learning models to make real-time predictions.</span></span>

<span data-ttu-id="bc21e-129">このシナリオでは、ImageNet-1K (1,000 クラス) データセットで事前にトレーニングされた ResNet-152 モデルを使用して、画像がどのカテゴリに属すかを予測します (以下の図参照)。</span><span class="sxs-lookup"><span data-stu-id="bc21e-129">This scenario uses a pre-trained ResNet-152 model trained on ImageNet-1K (1,000 classes) dataset to predict which category (see figure below) an image belongs to.</span></span> <span data-ttu-id="bc21e-130">これらの予測は、REST API エンドポイントを使用してリアルタイムで行われます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-130">These predictions are made in real time using a REST API endpoint.</span></span>

![](./_images/python-example-predictions.png)

<span data-ttu-id="bc21e-131">ディープ ラーニング モデルのアプリケーション フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="bc21e-131">The application flow for the deep learning model is as follows:</span></span>

1.  <span data-ttu-id="bc21e-132">クライアントは、HTTP POST 要求を、エンコード済み画像データと一緒に送信します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-132">The client sends an HTTP POST request with the encoded image data.</span></span>

2.  <span data-ttu-id="bc21e-133">Flask アプリは、その要求から画像を抽出します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-133">The Flask app extracts the image from the request.</span></span>

3.  <span data-ttu-id="bc21e-134">画像は前処理されてモデルに送信され、スコアリングが行われます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-134">The image is preprocessed and sent to the model for scoring.</span></span>

4.  <span data-ttu-id="bc21e-135">スコアリングの結果は JSON オブジェクトにパイプされ、クライアントに返されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-135">The scoring result is piped into a JSON object and returned to the client.</span></span>

## <a name="architecture"></a><span data-ttu-id="bc21e-136">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="bc21e-136">Architecture</span></span>

<span data-ttu-id="bc21e-137">このアーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-137">This architecture consists of the following components.</span></span>

<span data-ttu-id="bc21e-138">**[仮想マシン][vm]** (VM)。</span><span class="sxs-lookup"><span data-stu-id="bc21e-138">**[Virtual machine][vm]** (VM).</span></span> <span data-ttu-id="bc21e-139">VM は、HTTP 要求を送信できる &mdash;ローカルまたはクラウドの&mdash;デバイスの例として示されています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-139">The VM is shown as an example of a device &mdash; local or in the cloud &mdash; that can send an HTTP request.</span></span>

<span data-ttu-id="bc21e-140">**[Azure Kubernetes Service][aks]** (AKS) は、アプリケーションを Kubernetes クラスターにデプロイするときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-140">**[Azure Kubernetes Service][aks]** (AKS) is used to deploy the application on a Kubernetes cluster.</span></span> <span data-ttu-id="bc21e-141">AKS により、Kubernetes のデプロイと操作が簡略化されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-141">AKS simplifies the deployment and operations of Kubernetes.</span></span> <span data-ttu-id="bc21e-142">クラスターを構成する場合、通常の Python モデルについては CPU のみの VM を使用し、ディープ ラーニング モデルについては GPU 対応 VM を使用します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-142">The cluster can be configured using CPU-only VMs for regular Python models or GPU-enabled VMs for deep learning models.</span></span>

<span data-ttu-id="bc21e-143">**[ロード バランサー][lb]**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-143">**[Load balancer][lb]**.</span></span> <span data-ttu-id="bc21e-144">サービスを外部に公開するときに、AKS によってプロビジョニングされたロード バランサーが使用されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-144">A load balancer, provisioned by AKS, is used to expose the service externally.</span></span> <span data-ttu-id="bc21e-145">ロード バランサーからのトラフィックは、バックエンド ポッドに送信されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-145">Traffic from the load balancer is directed to the back-end pods.</span></span>

<span data-ttu-id="bc21e-146">**[Docker Hub][docker]** は、Kubernetes クラスターにデプロイされている Docker イメージを格納するときに使用されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-146">**[Docker Hub][docker]** is used to store the Docker image that is deployed on Kubernetes cluster.</span></span> <span data-ttu-id="bc21e-147">Docker Hub は、使いやすく、Docker ユーザーに対する既定のイメージ リポジトリであるため、このアーキテクチャに選択されました。</span><span class="sxs-lookup"><span data-stu-id="bc21e-147">Docker Hub was chosen for this architecture because it's easy to use and is the default image repository for Docker users.</span></span> <span data-ttu-id="bc21e-148">[Azure Container Registry][acr] を、このアーキテクチャに使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-148">[Azure Container Registry][acr] can also be used for this architecture.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="bc21e-149">パフォーマンスに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="bc21e-149">Performance considerations</span></span>

<span data-ttu-id="bc21e-150">リアルタイム スコアリング アーキテクチャでは、スループット パフォーマンスが主な考慮事項になります。</span><span class="sxs-lookup"><span data-stu-id="bc21e-150">For real-time scoring architectures, throughput performance becomes a dominant consideration.</span></span> <span data-ttu-id="bc21e-151">通常の Python モデルの場合は、CPU で十分にワークロードを処理できると一般に考えられています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-151">For regular Python models, it's generally accepted that CPUs are sufficient to handle the workload.</span></span> 

<span data-ttu-id="bc21e-152">しかし、ディープ ラーニングのワークロードで速度がボトルネックになっているときは、一般的に CPU よりも GPU の方が優れた[パフォーマンス][gpus-vs-cpus]を発揮します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-152">However for deep learning workloads, when speed is a bottleneck, GPUs generally provide better [performance][gpus-vs-cpus] compared to CPUs.</span></span> <span data-ttu-id="bc21e-153">CPU を使用して GPU に相当するパフォーマンスを実現するには、通常、多数の CPU を備えたクラスターが必要です。</span><span class="sxs-lookup"><span data-stu-id="bc21e-153">To match GPU performance using CPUs, a cluster with large number of CPUs is usually needed.</span></span>

<span data-ttu-id="bc21e-154">どちらのシナリオでもこのアーキテクチャには CPU を使用できますが、ディープ ラーニング モデルの場合、GPU が提供するスループット値は、同じコストの CPU クラスターに比べてかなり高くなります。</span><span class="sxs-lookup"><span data-stu-id="bc21e-154">You can use CPUs for this architecture in either scenario, but for deep learning models, GPUs provide significantly higher throughput values compared to a CPU cluster of similar cost.</span></span> <span data-ttu-id="bc21e-155">AKS では GPU の使用がサポートされています。これは、このアーキテクチャに AKS を使用するメリットの 1 つです。</span><span class="sxs-lookup"><span data-stu-id="bc21e-155">AKS supports the use of GPUs, which is one advantage of using AKS for this architecture.</span></span> <span data-ttu-id="bc21e-156">また、ディープ ラーニング デプロイで使用されるモデルには、通常、多数のパラメーターが含まれます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-156">Also, deep learning deployments typically use models with a high number of parameters.</span></span> <span data-ttu-id="bc21e-157">GPU を使用することで、CPU のみのデプロイで問題になる、モデルと Web サービスの間のリソース競合を防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-157">Using GPUs prevents contention for resources between the model and the web service, which is an issue in CPU-only deployments.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="bc21e-158">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="bc21e-158">Scalability considerations</span></span>

<span data-ttu-id="bc21e-159">CPU のみの VM で AKS クラスターがプロビジョニングされている通常の Python モデルの場合、[ポッドの数をスケールアウト][manually-scale-pods]するときに注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="bc21e-159">For regular Python models, where AKS cluster is provisioned with CPU-only VMs, take care when [scaling out the number of pods][manually-scale-pods].</span></span> <span data-ttu-id="bc21e-160">目標は、クラスターを完全に利用することです。</span><span class="sxs-lookup"><span data-stu-id="bc21e-160">The goal is to fully utilize the cluster.</span></span> <span data-ttu-id="bc21e-161">スケーリングは、CPU 要求と、ポッドに対して定義されている制限によって左右されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-161">Scaling depends on the CPU requests and limits defined for the pods.</span></span> <span data-ttu-id="bc21e-162">また、Kubernetes ではポッドの[自動スケーリング][autoscale-pods]もサポートされ、CPU 使用率などの選ばれたメトリックに応じて、デプロイのポッド数が調整されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-162">Kubernetes also supports [autoscaling][autoscale-pods] of the pods to adjust the number of pods in a deployment depending on CPU utilization or other select metrics.</span></span> <span data-ttu-id="bc21e-163">[クラスター オートスケーラー][autoscaler] (プレビュー) は、保留中のポッドに基づいてエージェント ノードをスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-163">The [cluster autoscaler][autoscaler] (in preview) can scale agent nodes based on pending pods.</span></span>

<span data-ttu-id="bc21e-164">GPU 対応 VM を使用しているディープ ラーニング シナリオでは、ポッドのリソース制限は、1 つのポッドに 1 つの GPU を割り当てる、といったものです。</span><span class="sxs-lookup"><span data-stu-id="bc21e-164">For deep learning scenarios, using GPU-enabled VMs, resource limits on pods are such that one GPU is assigned to one pod.</span></span> <span data-ttu-id="bc21e-165">使用する VM の種類に応じて[クラスターのノードをスケーリング][scale-cluster]して、サービスの需要に対応する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bc21e-165">Depending on the type of VM used, you must [scale the nodes of the cluster][scale-cluster] to meet the demand for the service.</span></span> <span data-ttu-id="bc21e-166">これを簡単に行うには、Azure CLI と kubectl を使用します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-166">You can do this easily using the Azure CLI and kubectl.</span></span>

## <a name="monitoring-and-logging-considerations"></a><span data-ttu-id="bc21e-167">監視とログ記録に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="bc21e-167">Monitoring and logging considerations</span></span>

### <a name="aks-monitoring"></a><span data-ttu-id="bc21e-168">AKS の監視</span><span class="sxs-lookup"><span data-stu-id="bc21e-168">AKS monitoring</span></span>

<span data-ttu-id="bc21e-169">AKS のパフォーマンスを可視化するには、[コンテナーに対する Azure Monitor][monitor-containers] 機能を使用します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-169">For visibility into AKS performance, use the [Azure Monitor for containers][monitor-containers] feature.</span></span> <span data-ttu-id="bc21e-170">この機能により、Kubernetes で使用可能なコントローラー、ノード、およびコンテナーから、メモリやプロセッサのメトリックが Metrics API 経由で収集されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-170">It collects memory and processor metrics from controllers, nodes, and containers that are available in Kubernetes through the Metrics API.</span></span>

<span data-ttu-id="bc21e-171">アプリケーションのデプロイ中、AKS クラスターを監視して、想定どおりに動作していること、すべてのノードが運用可能で、すべてのポッドが実行中であることを確認します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-171">While deploying your application, monitor the AKS cluster to make sure it's working as expected, all the nodes are operational, and all pods are running.</span></span> <span data-ttu-id="bc21e-172">[kubectl][kubectl] コマンドライン ツールを使用してポッドの状態を取得できますが、Kubernetes にも、クラスターの状態の基本的な監視と管理のための Web ダッシュボードがあります。</span><span class="sxs-lookup"><span data-stu-id="bc21e-172">Although you can use the [kubectl][kubectl] command-line tool to retrieve pod status, Kubernetes also includes a web dashboard for basic monitoring of the cluster status and management.</span></span>

![](./_images/python-kubernetes-dashboard.png)

<span data-ttu-id="bc21e-173">クラスターとノードの全体的な状態を確認するには、Kubernetes ダッシュボードの **[ノード]** セクションに移動します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-173">To see the overall state of the cluster and nodes, go to the **Nodes** section of the Kubernetes dashboard.</span></span> <span data-ttu-id="bc21e-174">ノードが非アクティブか失敗した場合は、そのページからエラー ログを表示できます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-174">If a node is inactive or has failed, you can display the error logs from that page.</span></span> <span data-ttu-id="bc21e-175">同様に、ポッドの数やデプロイの状態に関する情報を確認するには、**[ポッド]** セクションおよび **[デプロイ]** セクションに移動します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-175">Similarly, go to the **Pods** and **Deployments** sections for information about the number of pods and status of your deployment.</span></span>

### <a name="aks-logs"></a><span data-ttu-id="bc21e-176">AKS のログ</span><span class="sxs-lookup"><span data-stu-id="bc21e-176">AKS logs</span></span> 

<span data-ttu-id="bc21e-177">AKS では、すべての stdout と stderr が、クラスター内のポッドのログに自動的に記録されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-177">AKS automatically logs all stdout/stderr to the logs of the pods in the cluster.</span></span> <span data-ttu-id="bc21e-178">kubectl を使用すると、これらのログのほか、ノード レベルのイベントおよびログも確認できます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-178">Use kubectl to see these and also node-level events and logs.</span></span> <span data-ttu-id="bc21e-179">詳細については、デプロイの手順を確認してください。</span><span class="sxs-lookup"><span data-stu-id="bc21e-179">For details, see the deployment steps.</span></span>

<span data-ttu-id="bc21e-180">[コンテナー用の Azure Monitor][monitor-containers] を使用して、コンテナー化されたバージョンの Linux 用 Log Analytics エージェントを介してメトリックとログを収集します。こうしたメトリックやログは、Log Analytics ワークスペースに保存されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-180">Use [Azure Monitor for containers][monitor-containers] to collect metrics and logs through a containerized version of the Log Analytics agent for Linux, which is stored in your Log Analytics workspace.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="bc21e-181">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="bc21e-181">Security considerations</span></span>

<span data-ttu-id="bc21e-182">[Azure Security Center][security-center] を使用すると、Azure リソースのセキュリティの状態を一元的に表示して把握できます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-182">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="bc21e-183">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。ただし、AKS エージェント ノードは監視されません。</span><span class="sxs-lookup"><span data-stu-id="bc21e-183">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment, although it doesn't monitor AKS agent nodes.</span></span> <span data-ttu-id="bc21e-184">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-184">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="bc21e-185">[Azure サブスクリプションでの Security Center Standard の利用開始][get-started]に関するページの説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="bc21e-185">Enable security data collection as described in [Onboard your Azure subscription to Security Center Standard][get-started].</span></span> <span data-ttu-id="bc21e-186">データ収集を有効にすると、セキュリティ センターは、そのサブスクリプションに作成されているすべての VM を自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="bc21e-186">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="bc21e-187">**操作**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-187">**Operations**.</span></span> <span data-ttu-id="bc21e-188">Azure Active Directory (Azure AD) 認証トークンを使用して AKS クラスターにサインインするには、[ユーザー認証][aad-auth]に Azure AD を使用するように AKS を構成します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-188">To sign in to an AKS cluster using your Azure Active Directory (Azure AD) authentication token, configure AKS to use Azure AD for [user authentication][aad-auth].</span></span> <span data-ttu-id="bc21e-189">また、クラスター管理者が、ユーザーの ID またはディレクトリ グループ メンバーシップに基づいて、Kubernetes のロールベースのアクセス制御 (RBAC) を構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-189">Cluster administrators can also configure Kubernetes role-based access control (RBAC) based on a user's identity or directory group membership.</span></span>

<span data-ttu-id="bc21e-190">[RBAC][rbac] を使用して、デプロイする Azure リソースへのアクセスを制御してください。</span><span class="sxs-lookup"><span data-stu-id="bc21e-190">Use [RBAC][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="bc21e-191">RBAC を使用すると、DevOps チームのメンバーに承認の役割を割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-191">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="bc21e-192">ユーザーを複数の役割に割り当てることができ、よりきめ細かい[アクセス許可]のカスタム ロールを作成することができます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-192">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained [permissions].</span></span>

<span data-ttu-id="bc21e-193">**HTTPS**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-193">**HTTPS**.</span></span> <span data-ttu-id="bc21e-194">セキュリティのベスト プラクティスとして、アプリケーションでは HTTPS を強制して、HTTP 要求をリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="bc21e-194">As a security best practice, the application should enforce HTTPS and redirect HTTP requests.</span></span> <span data-ttu-id="bc21e-195">SSL を終了して HTTP 要求をリダイレクトするリバース プロキシをデプロイするには、[イングレス コントローラー][ingress-controller]を使用します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-195">Use an [ingress controller][ingress-controller] to deploy a reverse proxy that terminates SSL and redirects HTTP requests.</span></span> <span data-ttu-id="bc21e-196">詳細については、「[Azure Kubernetes Service (AKS) で HTTPS イングレス コントローラーを作成する][https-ingress]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="bc21e-196">For more information, see [Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)][https-ingress].</span></span>

<span data-ttu-id="bc21e-197">**[認証]**: </span><span class="sxs-lookup"><span data-stu-id="bc21e-197">**Authentication**.</span></span> <span data-ttu-id="bc21e-198">このソリューションでは、エンドポイントへのアクセスは制限されません。</span><span class="sxs-lookup"><span data-stu-id="bc21e-198">This solution doesn't restrict access to the endpoints.</span></span> <span data-ttu-id="bc21e-199">エンタープライズ設定でアーキテクチャをデプロイするには、API キーによってエンドポイントをセキュリティで保護し、何らかの形式のユーザー認証をクライアント アプリケーションに追加します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-199">To deploy the architecture in an enterprise setting, secure the endpoints through API keys and add some form of user authentication to the client application.</span></span>

<span data-ttu-id="bc21e-200">**コンテナー レジストリ**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-200">**Container registry**.</span></span> <span data-ttu-id="bc21e-201">このソリューションでは、Docker イメージの格納にパブリック レジストリが使用されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-201">This solution uses a public registry to store the Docker image.</span></span> <span data-ttu-id="bc21e-202">アプリケーションが依存するコード、およびモデルは、このイメージ内に含まれています。</span><span class="sxs-lookup"><span data-stu-id="bc21e-202">The code that the application depends on, and the model, are contained within this image.</span></span> <span data-ttu-id="bc21e-203">エンタープライズ アプリケーションでは、悪意のあるコードが実行されるのを防ぎ、コンテナー内の情報が侵害されないようにするために、プライベート レジストリを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bc21e-203">Enterprise applications should use a private registry to help guard against running malicious code and to help keep the information inside the container from being compromised.</span></span>

<span data-ttu-id="bc21e-204">**DDoS 保護**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-204">**DDoS protection**.</span></span> <span data-ttu-id="bc21e-205">[DDoS Protection Standard][ddos] を有効にすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-205">Consider enabling [DDoS Protection Standard][ddos].</span></span> <span data-ttu-id="bc21e-206">Azure プラットフォームの一部として基本な DDoS 保護が有効になりますが、DDoS Protection Standard により、特に Azure 仮想ネットワークのリソース向けにチューニングされた軽減機能が提供されます。</span><span class="sxs-lookup"><span data-stu-id="bc21e-206">Although basic DDoS protection is enabled as part of the Azure platform, DDoS Protection Standard provides mitigation capabilities that are tuned specifically to Azure virtual network resources.</span></span>

<span data-ttu-id="bc21e-207">**ログの記録**。</span><span class="sxs-lookup"><span data-stu-id="bc21e-207">**Logging**.</span></span> <span data-ttu-id="bc21e-208">ログ データを格納する前にベスト プラクティスを使用します。たとえば、セキュリティ不正アクセスに使用できるユーザー パスワードなどの情報を除去します。</span><span class="sxs-lookup"><span data-stu-id="bc21e-208">Use best practices before storing log data, such as scrubbing user passwords and other information that could be used to commit security fraud.</span></span>

## <a name="deployment"></a><span data-ttu-id="bc21e-209">Deployment</span><span class="sxs-lookup"><span data-stu-id="bc21e-209">Deployment</span></span>

<span data-ttu-id="bc21e-210">この参照アーキテクチャを展開するには、GitHub リポジトリで説明されている手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="bc21e-210">To deploy this reference architecture, follow the steps described in the GitHub repos:</span></span> 

  - <span data-ttu-id="bc21e-211">[通常の Python のモデル][github-python]</span><span class="sxs-lookup"><span data-stu-id="bc21e-211">[Regular Python models][github-python]</span></span>
  - <span data-ttu-id="bc21e-212">[ディープ ラーニング モデル][github-dl]</span><span class="sxs-lookup"><span data-stu-id="bc21e-212">[Deep learning models][github-dl]</span></span>

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

