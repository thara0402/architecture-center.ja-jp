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
# <a name="training-of-python-scikit-learn-models-on-azure"></a><span data-ttu-id="044a1-103">Azure での Python scikit-learn モデルのトレーニング</span><span class="sxs-lookup"><span data-stu-id="044a1-103">Training of Python scikit-learn models on Azure</span></span>

<span data-ttu-id="044a1-104">この参照アーキテクチャでは、[scikit-learn][scikit] Python モデルのハイパーパラメーター (トレーニング パラメーター) の調整の推奨される方法が示されています。</span><span class="sxs-lookup"><span data-stu-id="044a1-104">This reference architecture shows recommended practices for tuning the hyperparameters (training parameters) of a [scikit-learn][scikit] Python model.</span></span> <span data-ttu-id="044a1-105">このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="044a1-105">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![アーキテクチャ ダイアグラム][0]

<span data-ttu-id="044a1-107">**シナリオ:** ここで取り上げる問題は、よく寄せられる質問 (FAQ) の一致です。</span><span class="sxs-lookup"><span data-stu-id="044a1-107">**Scenario:** The problem addressed here is Frequently Asked Question (FAQ) matching.</span></span> <span data-ttu-id="044a1-108">このシナリオでは、JavaScript としてタグ付けされた元の質問、重複する質問、およびその回答を含む Stack Overflow 質問データのサブセットが使用されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-108">This scenario uses a subset of Stack Overflow question data that includes original questions tagged as JavaScript, their duplicate questions, and their answers.</span></span> <span data-ttu-id="044a1-109">重複する質問が元の質問のいずれかと一致する可能性を予測するように scikit-learn パイプラインをトレーニングします。</span><span class="sxs-lookup"><span data-stu-id="044a1-109">It tunes a scikit-learn pipeline to predict the probability that a duplicate question matches one of the original questions.</span></span>

<span data-ttu-id="044a1-110">このシナリオでの処理には、次のステップが含まれます。</span><span class="sxs-lookup"><span data-stu-id="044a1-110">Processing in this scenario involves the following steps:</span></span>

1. <span data-ttu-id="044a1-111">トレーニング Python スクリプトが、[Azure Machine Learning service][aml] に送信されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-111">The training Python script is submitted to the [Azure Machine Learning service][aml].</span></span>

1. <span data-ttu-id="044a1-112">各ノード上に作成された Docker コンテナー内で、スクリプトが実行されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-112">The script runs in Docker containers that are created on each node.</span></span>

1. <span data-ttu-id="044a1-113">そのスクリプトで、[Azure Storage][storage] からトレーニング データとテスト データが取得されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-113">That script retrieves training and testing data from [Azure Storage][storage].</span></span>

1. <span data-ttu-id="044a1-114">スクリプトにより、トレーニング パラメーターの組み合わせを使用してトレーニング データからモデルが学習されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-114">The script learns a model from the training data using its combination of training parameters.</span></span>

1. <span data-ttu-id="044a1-115">モデルのパフォーマンスがテスト データで評価されて、Azure Storage に書き込まれます。</span><span class="sxs-lookup"><span data-stu-id="044a1-115">The model's performance is evaluated on the testing data, and is written to Azure Storage.</span></span>

1. <span data-ttu-id="044a1-116">最もパフォーマンスのよいモデルが、Azure Machine Learning service に登録されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-116">The best performing model is registered with the Azure Machine Learning service.</span></span>

<span data-ttu-id="044a1-117">GPU での[ディープ ラーニング モデル][training-deep-learning]のトレーニングに関する考慮事項も参照してください。</span><span class="sxs-lookup"><span data-stu-id="044a1-117">See also considerations for training [deep learning models][training-deep-learning] with GPUs.</span></span>

## <a name="architecture"></a><span data-ttu-id="044a1-118">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="044a1-118">Architecture</span></span>

<span data-ttu-id="044a1-119">このアーキテクチャは、ニーズに従ってリソースをスケーリングする複数の Azure クラウド サービスで構成されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-119">This architecture consists of several Azure cloud services that scale resources according to need.</span></span> <span data-ttu-id="044a1-120">サービスおよびこのソリューションでのその役割を、以下で説明します。</span><span class="sxs-lookup"><span data-stu-id="044a1-120">The services and their roles in this solution are described below.</span></span>

<span data-ttu-id="044a1-121">[Microsoft Data Science Virtual Machine][dsvm] (DSVM) は、データ分析と機械学習に使用されるツールで構成されている VM イメージです。</span><span class="sxs-lookup"><span data-stu-id="044a1-121">[Microsoft Data Science Virtual Machine][dsvm] (DSVM) is a VM image configured with tools used for data analytics and machine learning.</span></span> <span data-ttu-id="044a1-122">Windows Server バージョンと Linux バージョンの両方を使用できます。</span><span class="sxs-lookup"><span data-stu-id="044a1-122">Both Windows Server and Linux versions are available.</span></span> <span data-ttu-id="044a1-123">このデプロイでは、Ubuntu 16.04 LTS 上の DSVM の Linux エディションを使用します。</span><span class="sxs-lookup"><span data-stu-id="044a1-123">This deployment uses the Linux editions of the DSVM on Ubuntu 16.04 LTS.</span></span>

<span data-ttu-id="044a1-124">[Azure Machine Learning service][aml] は、クラウド スケールで機械学習モデルをトレーニング、デプロイ、自動化、管理するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-124">[Azure Machine Learning service][aml] is used to train, deploy, automate, and manage machine learning models at cloud scale.</span></span> <span data-ttu-id="044a1-125">このアーキテクチャでは、以下で説明する Azure リソースの使用と割り当てを管理するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-125">It's used in this architecture to manage the allocation and use of the Azure resources described below.</span></span>

<span data-ttu-id="044a1-126">[Azure Machine Learning コンピューティング][aml-compute]は、Azure 内で大規模な機械学習モデルと AI モデルのトレーニングとテストを行うために使用されるリソースです。</span><span class="sxs-lookup"><span data-stu-id="044a1-126">[Azure Machine Learning Compute][aml-compute] is the resource used to train and test machine learning and AI models at scale in Azure.</span></span> <span data-ttu-id="044a1-127">このシナリオでの[コンピューティング先][target]は、自動[スケーリング][scaling] オプションに基づいてオンデマンドで割り当てられるノードのクラスターです。</span><span class="sxs-lookup"><span data-stu-id="044a1-127">The [compute target][target] in this scenario is a cluster of nodes that are allocated on demand based on an automatic [scaling][scaling] option.</span></span> <span data-ttu-id="044a1-128">各ノードは、特定の[ハイパーパラメーター][hyperparameter] セットに対するトレーニング ジョブが実行される VM です。</span><span class="sxs-lookup"><span data-stu-id="044a1-128">Each node is a VM that runs a training job for a particular [hyperparameter][hyperparameter] set.</span></span>

<span data-ttu-id="044a1-129">[Azure Container Registry][acr] には、あらゆる種類の Docker コンテナー デプロイのイメージが格納されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-129">[Azure Container Registry][acr] stores images for all types of Docker container deployments.</span></span> <span data-ttu-id="044a1-130">これらのコンテナーは、各ノード上に作成されて、トレーニング Python スクリプトの実行に使用されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-130">These containers are created on each node and used to run the training Python script.</span></span> <span data-ttu-id="044a1-131">Machine Learning コンピューティング クラスターで使用されるイメージは、ローカル実行とハイパーパラメーター調整ノートブックで Machine Learning service によって作成された後、Container Registry にプッシュされます。</span><span class="sxs-lookup"><span data-stu-id="044a1-131">The image used in the Machine Learning Compute cluster is created by the Machine Learning service in the local run and hyperparameter tuning notebooks, and then is pushed to Container Registry.</span></span>

<span data-ttu-id="044a1-132">[Azure BLOB][blob] ストレージは、トレーニング Python スクリプトによって使用されるトレーニングおよびテスト データ セットを Machine Learning service から受け取ります。</span><span class="sxs-lookup"><span data-stu-id="044a1-132">[Azure Blob][blob] storage receives the training and test data sets from the Machine Learning service that are used by the training Python script.</span></span> <span data-ttu-id="044a1-133">ストレージは、Machine Learning コンピューティング クラスターの各ノードに仮想ドライブとしてマウントされます。</span><span class="sxs-lookup"><span data-stu-id="044a1-133">Storage is mounted as a virtual drive onto each node of a Machine Learning Compute cluster.</span></span> 

## <a name="performance-considerations"></a><span data-ttu-id="044a1-134">パフォーマンスに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="044a1-134">Performance considerations</span></span>

<span data-ttu-id="044a1-135">[ハイパーパラメーター][hyperparameter]の各セットが、Machine Learning [コンピューティング先][target]の 1 つのノードで実行されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-135">Each set of [hyperparameters][hyperparameter] runs on one node of the Machine Learning [compute target][target].</span></span> <span data-ttu-id="044a1-136">このアーキテクチャでは各ノードは Standard D4 v2 VM であり、4 つのコアを備えています。</span><span class="sxs-lookup"><span data-stu-id="044a1-136">For this architecture, each node is a Standard D4 v2 VM, which has four cores.</span></span> <span data-ttu-id="044a1-137">このアーキテクチャでは、機械学習に対して [LightGBM][lightgbm] 分類子 (勾配ブースティング フレームワーク) が使用されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-137">This architecture uses a [LightGBM][lightgbm] classifier for machine learning, a gradient boosting framework.</span></span> <span data-ttu-id="044a1-138">このソフトウェアは 4 つのコアすべてで同時に実行できるので、各実行が最大で 4 倍に高速化されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-138">This software can run on all four cores at the same time, speeding up each run by a factor of up to four.</span></span> <span data-ttu-id="044a1-139">それにより、ハイパーパラメーター調整の実行全体にかかる時間は、それぞれがコアを 1 つしか備えていない Standard D1 v2 VM に基づく Machine Learning コンピューティング先で実行した場合より、最大で 4 分の 1 に短縮されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-139">That way, the whole hyperparameter tuning run takes up to one-quarter of the time it would take had it been run on a Machine Learning Compute target based on Standard D1 v2 VMs, which have only one core each.</span></span>

<span data-ttu-id="044a1-140">Machine Learning コンピューティング ノードの最大数は、合計実行時間に影響します。</span><span class="sxs-lookup"><span data-stu-id="044a1-140">The maximum number of Machine Learning Compute nodes affects the total run time.</span></span> <span data-ttu-id="044a1-141">ノードの推奨される最小数は 0 です。</span><span class="sxs-lookup"><span data-stu-id="044a1-141">The recommended minimum number of nodes is zero.</span></span> <span data-ttu-id="044a1-142">この設定では、ジョブの開始にかかる時間に、少なくとも 1 つのノードをクラスターに自動スケーリングするのに必要な何分かが含まれます。</span><span class="sxs-lookup"><span data-stu-id="044a1-142">With this setting, the time it takes for a job to start up includes some minutes for auto-scaling at least one node into the cluster.</span></span> <span data-ttu-id="044a1-143">ただし、ハイパーパラメーター調整の実行時間が短い場合は、ジョブのスケールアップによってオーバーヘッドが増えます。</span><span class="sxs-lookup"><span data-stu-id="044a1-143">If the hyperparameter tuning runs for a short time, however, scaling up the job adds to the overhead.</span></span> <span data-ttu-id="044a1-144">たとえば、ジョブを 5 分以下で実行できても、1 ノードへのスケールアップにさらに 5 分かかる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="044a1-144">For example, a job can run in under five minutes, but scaling up to one node might take another five minutes.</span></span> <span data-ttu-id="044a1-145">このような場合は、最小のノード数を 1 に設定すると時間を節約できますが、コストは増加します。</span><span class="sxs-lookup"><span data-stu-id="044a1-145">In this case, setting the minimum to one node saves time but adds to the cost.</span></span>

## <a name="monitoring-and-logging-considerations"></a><span data-ttu-id="044a1-146">監視とログ記録に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="044a1-146">Monitoring and logging considerations</span></span>

<span data-ttu-id="044a1-147">実行の進行状況の監視に使用するため、[HyperDrive][hyperparameter] 実行構成を送信して実行オブジェクトを返します。</span><span class="sxs-lookup"><span data-stu-id="044a1-147">Submit a [HyperDrive][hyperparameter] run configuration to return a Run object for use in monitoring the run's progress.</span></span>

### <a name="rundetails-jupyter-widget"></a><span data-ttu-id="044a1-148">RunDetails Jupyter ウィジェット</span><span class="sxs-lookup"><span data-stu-id="044a1-148">RunDetails Jupyter Widget</span></span>

<span data-ttu-id="044a1-149">キューに入っているとき、および子ジョブを実行しているときに、その進行状況を簡単に監視するには、RunDetails [Jupyter ウィジェット][jupyter]で実行オブジェクトを使用します。</span><span class="sxs-lookup"><span data-stu-id="044a1-149">Use the Run object with the RunDetails [Jupyter widget][jupyter] to conveniently monitor its progress at queuing and when running its children jobs.</span></span> <span data-ttu-id="044a1-150">ログに記録される主要な統計情報の値もリアルタイムで表示されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-150">It also shows the values of the primary statistic that they log in real time.</span></span>

### <a name="azure-portal"></a><span data-ttu-id="044a1-151">Azure ポータル</span><span class="sxs-lookup"><span data-stu-id="044a1-151">Azure portal</span></span>

<span data-ttu-id="044a1-152">実行オブジェクトを出力すると、次のように Azure portal での実行のページへのリンクが表示されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-152">Print a Run object to display a link to the run's page in Azure portal like this:</span></span>

![実行オブジェクト][1]

<span data-ttu-id="044a1-154">このページを使用して、実行とその子実行の状態を監視できます。</span><span class="sxs-lookup"><span data-stu-id="044a1-154">Use this page to monitor the status of the run and its children runs.</span></span> <span data-ttu-id="044a1-155">各子実行には、実行が済んだトレーニング スクリプトの出力を含むドライバー ログがあります。</span><span class="sxs-lookup"><span data-stu-id="044a1-155">Each child run has a driver log containing the output of the training script it has run.</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="044a1-156">コストに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="044a1-156">Cost considerations</span></span>

<span data-ttu-id="044a1-157">ハイパーパラメーター調整実行のコストは、Machine Learning コンピューティング VM のサイズ、優先順位の低いノードを使うかどうか、クラスターで許容される最大ノード数の選択に比例します。</span><span class="sxs-lookup"><span data-stu-id="044a1-157">The cost of a hyperparameter tuning run depends linearly on the choice of Machine Learning Compute VM size, whether low-priority nodes are used, and the maximum number of nodes allowed in the cluster.</span></span>

<span data-ttu-id="044a1-158">クラスターが使用されていないときの継続的なコストは、クラスターで必要なノードの最小数に依存します。</span><span class="sxs-lookup"><span data-stu-id="044a1-158">Ongoing costs when the cluster is not in use depend on the minimum number of nodes required by the cluster.</span></span> <span data-ttu-id="044a1-159">クラスターの自動スケーリングが有効なシステムでは、ノードはジョブの数に合わせて許容される最大数まで自動的に追加され、不要になると要求されている最小数まで削除されます。</span><span class="sxs-lookup"><span data-stu-id="044a1-159">With cluster autoscaling, the system automatically adds nodes up to the allowed maximum to match the number of jobs, and then removes nodes down to the requested minimum as they are no longer needed.</span></span> <span data-ttu-id="044a1-160">クラスターが 0 ノードまで自動的にスケールダウンできる場合は、使われていないときはコストがかかりません。</span><span class="sxs-lookup"><span data-stu-id="044a1-160">If the cluster can autoscale down to zero nodes, it does not cost anything when it is not in use.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="044a1-161">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="044a1-161">Security considerations</span></span>

### <a name="restrict-access-to-azure-blob-storage"></a><span data-ttu-id="044a1-162">Azure Blob Storage へのアクセスの制限</span><span class="sxs-lookup"><span data-stu-id="044a1-162">Restrict access to Azure Blob Storage</span></span>

<span data-ttu-id="044a1-163">このアーキテクチャでは、[ストレージ アカウント キー][storage-security]を使用して Blob ストレージにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="044a1-163">This architecture uses [storage account keys][storage-security] to access the Blob storage.</span></span> <span data-ttu-id="044a1-164">さらに制御と保護を強化するには、共有アクセス署名 (SAS) を代わりに使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="044a1-164">For further control and protection, consider using a shared access signature (SAS) instead.</span></span> <span data-ttu-id="044a1-165">これは、ストレージ内のオブジェクトへの制限付きアクセスを付与し、アカウント キーをハード コーディングしたり、それをプレーンテキストで保存したりする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="044a1-165">This grants limited access to objects in storage, without needing to hard-code the account keys or save them in plaintext.</span></span> <span data-ttu-id="044a1-166">また、SAS を使用すると、ストレージ アカウントに適切なガバナンスがあること、およびアクセスがそれを必要とするユーザーだけに付与されることを保証することができます。</span><span class="sxs-lookup"><span data-stu-id="044a1-166">Using a SAS also helps to ensure that the storage account has proper governance, and that access is granted only to the people intended to have it.</span></span>

<span data-ttu-id="044a1-167">ストレージ キーはワークロードのすべての入出力データに対するフル アクセス権を与えるため、データの機密性がさらに高いシナリオでは、すべてのストレージ キーが保護されるようにします。</span><span class="sxs-lookup"><span data-stu-id="044a1-167">For scenarios with more sensitive data, make sure that all of your storage keys are protected, because these keys grant full access to all input and output data from the workload.</span></span>

### <a name="encrypt-data-at-rest-and-in-motion"></a><span data-ttu-id="044a1-168">保存時および稼働時のデータの暗号化</span><span class="sxs-lookup"><span data-stu-id="044a1-168">Encrypt data at rest and in motion</span></span>

<span data-ttu-id="044a1-169">機密データを使用するシナリオでは、保存データつまりストレージ内にあるデータを暗号化します。</span><span class="sxs-lookup"><span data-stu-id="044a1-169">In scenarios that use sensitive data, encrypt the data at rest—that is, the data in storage.</span></span> <span data-ttu-id="044a1-170">データがある場所から次の場所に移動されるたびに、SSL を使用してデータ転送をセキュリティ保護します。</span><span class="sxs-lookup"><span data-stu-id="044a1-170">Each time data moves from one location to the next, use SSL to secure the data transfer.</span></span> <span data-ttu-id="044a1-171">詳細については、「[Azure Storage セキュリティ ガイド][storage-security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="044a1-171">For more information, see the [Azure Storage security guide][storage-security].</span></span>

### <a name="secure-data-in-a-virtual-network"></a><span data-ttu-id="044a1-172">仮想ネットワーク内のデータのセキュリティ保護</span><span class="sxs-lookup"><span data-stu-id="044a1-172">Secure data in a virtual network</span></span>

<span data-ttu-id="044a1-173">運用環境にデプロイする場合、指定した仮想ネットワークのサブネットにクラスターをデプロイすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="044a1-173">For production deployments, consider deploying the cluster into a subnet of a virtual network that you specify.</span></span> <span data-ttu-id="044a1-174">これにより、クラスター内のコンピューティング ノードは、他の仮想マシンやオンプレミスのネットワークと安全に通信できます。</span><span class="sxs-lookup"><span data-stu-id="044a1-174">This allows the compute nodes in the cluster to communicate securely with other virtual machines or with an on-premises network.</span></span> <span data-ttu-id="044a1-175">また、BLOB ストレージを使用する[サービス エンドポイント][endpoints]を使って、仮想ネットワークからのアクセスを許可したり、Azure Machine Learning service を備えた仮想ネットワークの内部で単一ノード NFS を使用したりすることも可能です。</span><span class="sxs-lookup"><span data-stu-id="044a1-175">You can also use [service endpoints][endpoints] with blob storage to grant access from a virtual network or use a single-node NFS inside the virtual network with Azure Machine Learning service.</span></span>

## <a name="deployment"></a><span data-ttu-id="044a1-176">Deployment</span><span class="sxs-lookup"><span data-stu-id="044a1-176">Deployment</span></span>

<span data-ttu-id="044a1-177">このアーキテクチャの参照実装をデプロイするには、[GitHub][github] リポジトリの手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="044a1-177">To deploy the reference implementation for this architecture, follow the steps in the [GitHub][github] repo.</span></span>

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