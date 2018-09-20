---
title: コンテナー ベースのワークロード用の CI/CD パイプライン
description: Jenkins、Azure Container Registry、Azure Kubernetes Service、Cosmos DB、Grafana を使用する、Node.js Web アプリの DevOps パイプラインを構築するための実証済みのシナリオ。
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: d659916e3af0caa2128db25faab441a2af8f3f6a
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389385"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a><span data-ttu-id="dd8e4-103">コンテナー ベースのワークロード用の CI/CD パイプライン</span><span class="sxs-lookup"><span data-stu-id="dd8e4-103">CI/CD pipeline for container-based workloads</span></span>

<span data-ttu-id="dd8e4-104">このシナリオ例は、コンテナーと DevOps ワークフローを使用してアプリケーション開発を最新化することを求めている企業に適用されます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-104">This example scenario is applicable to businesses that want to modernize application development by using containers and DevOps workflows.</span></span> <span data-ttu-id="dd8e4-105">このシナリオでは、Jenkins によって Node.js Web アプリが構築され、Azure Container Registry と Azure Kubernetes Service にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-105">In this scenario, a Node.js web app is built and deployed by Jenkins into an Azure Container Registry and Azure Kubernetes Service.</span></span> <span data-ttu-id="dd8e4-106">グローバル分散データベース層には、Azure Cosmos DB が使用されます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-106">For a globally distributed database tier, Azure Cosmos DB is used.</span></span> <span data-ttu-id="dd8e4-107">アプリケーションのパフォーマンスの監視とトラブルシューティングのために、Azure Monitor が Grafana インスタンスおよびダッシュボードと統合されています。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-107">To monitor and troubleshoot application performance, Azure Monitor integrates with a Grafana instance and dashboard.</span></span>

<span data-ttu-id="dd8e4-108">アプリケーションのシナリオ例には、自動化された開発環境の提供、新しいコードのコミットの検証、ステージング環境または運用環境への新しいデプロイのプッシュが含まれます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-108">Example application scenarios include providing an automated development environment, validating new code commits, and pushing new deployments into staging or production environments.</span></span> <span data-ttu-id="dd8e4-109">従来、企業はアプリケーションや更新プログラムを手動でビルドしてコンパイルし、大規模なモノリシック コード ベースを維持する必要がありました。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-109">Traditionally, businesses had to manually build and compile applications and updates, and maintain a large, monolithic code base.</span></span> <span data-ttu-id="dd8e4-110">継続的インテグレーション (CI) と継続的配置 (CD) を使用する最新のアプリケーション開発手法により、サービスの構築、テスト、デプロイを迅速化できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-110">With a modern approach to application development that uses continuous integration (CI) and continuous deployment (CD), you can more quickly build, test, and deploy services.</span></span> <span data-ttu-id="dd8e4-111">この最新の手法により、アプリケーションや更新プログラムを顧客により迅速にリリースし、変化するビジネス要求により俊敏に対応することができます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-111">This modern approach lets you release applications and updates to your customers faster, and respond to changing business demands in a more agile manner.</span></span>

<span data-ttu-id="dd8e4-112">Azure Kubernetes Service、Container Registry、Cosmos DB などの Azure サービスを使用することで、企業は最新のアプリケーション開発技術やツールを使って、高可用性の実装プロセスを簡素化できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-112">By using Azure services such as Azure Kubernetes Service, Container Registry, and Cosmos DB, companies can use the latest in application development techniques and tools to simplify the process of implementing high availability.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="dd8e4-113">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="dd8e4-113">Related use cases</span></span>

<span data-ttu-id="dd8e4-114">次のユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-114">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="dd8e4-115">アプリケーション開発プラクティスを、コンテナー ベースのマイクロサービス手法に最新化する。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-115">Modernizing application development practices to a microservice, container-based approach.</span></span>
* <span data-ttu-id="dd8e4-116">アプリケーションの開発とデプロイのライフサイクルを加速化する。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-116">Speeding up application development and deployment lifecycles.</span></span>
* <span data-ttu-id="dd8e4-117">検証のために、テスト環境または受け入れ環境へのデプロイを自動化する。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-117">Automating deployments to test or acceptance environments for validation.</span></span>

## <a name="architecture"></a><span data-ttu-id="dd8e4-118">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="dd8e4-118">Architecture</span></span>

![Jenkins、Azure Container Registry、Azure Kubernetes Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャの概要][architecture]

<span data-ttu-id="dd8e4-120">このシナリオでは、Node.js Web アプリケーションおよびデータベース バックエンドの DevOps パイプラインに対応できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-120">This scenario covers a DevOps pipeline for a Node.js web application and database backend.</span></span> <span data-ttu-id="dd8e4-121">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-121">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="dd8e4-122">開発者が、Node.js Web アプリケーションのソース コードに変更を加えます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-122">A developer makes changes to the Node.js web application source code.</span></span>
2. <span data-ttu-id="dd8e4-123">コードの変更がソース管理リポジトリ (GitHub など) にコミットされます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-123">The code change is committed to a source control repository, such as GitHub.</span></span>
3. <span data-ttu-id="dd8e4-124">継続的インテグレーション (CI) プロセスを開始するために、GitHub Webhook が Jenkins プロジェクトのビルドをトリガーします。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-124">To start the continuous integration (CI) process, a GitHub webhook triggers a Jenkins project build.</span></span>
4. <span data-ttu-id="dd8e4-125">Jenkins ビルド ジョブが、Azure Kubernetes Service の動的ビルド エージェントを使用してコンテナーのビルド プロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-125">The Jenkins build job uses a dynamic build agent in Azure Kubernetes Service to perform a container build process.</span></span>
5. <span data-ttu-id="dd8e4-126">ソース管理のコードからコンテナー イメージが作成され、Azure Container Registry にプッシュされます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-126">A container image is created from the code in source control, and is then pushed to an Azure Container Registry.</span></span>
6. <span data-ttu-id="dd8e4-127">継続的配置 (CD) を通じて、Jenkins がこの最新のコンテナー イメージを Kubernetes クラスターに展開します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-127">Through continuous deployment (CD), Jenkins deploys this updated container image to the Kubernetes cluster.</span></span>
7. <span data-ttu-id="dd8e4-128">Node.js Web アプリケーションは、Azure Cosmos DB をバックエンドとして使用します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-128">The Node.js web application uses Azure Cosmos DB as it's backend.</span></span> <span data-ttu-id="dd8e4-129">Cosmos DB と Azure Kubernetes Service が、Azure Monitor にメトリックを報告します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-129">Both Cosmos DB and Azure Kubernetes Service report metrics to Azure Monitor.</span></span>
8. <span data-ttu-id="dd8e4-130">Grafana インスタンスが、Azure Monitor のデータに基づいて、アプリケーションのパフォーマンスのビジュアル ダッシュボードを提供します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-130">A Grafana instance provides visual dashboards of the application performance based on the data from Azure Monitor.</span></span>

### <a name="components"></a><span data-ttu-id="dd8e4-131">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="dd8e4-131">Components</span></span>

* <span data-ttu-id="dd8e4-132">[Jenkins][jenkins]: オープンソースのオートメーション サーバーです。Azure サービスと統合することで、継続的インテグレーション (CI) と継続的配置 (CD) が可能になります。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-132">[Jenkins][jenkins] is an open-source automation server that can integrate with Azure services to enable continuous integration (CI) and continuous deployment (CD).</span></span> <span data-ttu-id="dd8e4-133">このシナリオでは、Jenkins によって、ソース管理へのコミットに基づいて新しいコンテナー イメージの作成が調整され、それらのイメージが Azure Container Registry にプッシュされた後、Azure Kubernetes Service 内でアプリケーション インスタンスが更新されます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-133">In this scenario, Jenkins orchestrates the creation of new container images based on commits to source control, pushes those images to Azure Container Registry, then updates application instances in Azure Kubernetes Service.</span></span>
* <span data-ttu-id="dd8e4-134">[Azure Linux Virtual Machines][azurevm-docs]: Jenkins インスタンスと Grafana インスタンスを実行するために使用される IaaS プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-134">[Azure Linux Virtual Machines][azurevm-docs] is the IaaS platform used to run the Jenkins and Grafana instances.</span></span>
* <span data-ttu-id="dd8e4-135">[Azure Container Registry][azureacr-docs]: Azure Kubernetes Service クラスターで使用されるコンテナー イメージを保存し、管理します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-135">[Azure Container Registry][azureacr-docs] stores and manages container images that are used by the Azure Kubernetes Service cluster.</span></span> <span data-ttu-id="dd8e4-136">イメージは安全に保存されています。Azure プラットフォームによって他のリージョンにレプリケートすることで、デプロイ時間を短縮できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-136">Images are securely stored, and can be replicated to other regions by the Azure platform to speed up deployment times.</span></span>
* <span data-ttu-id="dd8e4-137">[Azure Kubernetes Service][azureaks-docs]: コンテナー オーケストレーションの専門知識がなくても、コンテナー化されたアプリケーションをデプロイし、管理できるマネージド Kubernetes プラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-137">[Azure Kubernetes Service][azureaks-docs] is a managed Kubernetes platform that lets you deploy and manage containerized applications without container orchestration expertise.</span></span> <span data-ttu-id="dd8e4-138">ホストされた Kubernetes サービスとして、Azure は正常性監視やメンテナンスなどの重要なタスクを自動的に処理します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-138">As a hosted Kubernetes service, Azure handles critical tasks like health monitoring and maintenance for you.</span></span>
* <span data-ttu-id="dd8e4-139">[Azure Cosmos DB][azurecosmosdb-docs]: ニーズに合わせて、さまざまなデータベースや整合性モデルの中から選択できる、グローバル分散型のマルチモデル データベースです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-139">[Azure Cosmos DB][azurecosmosdb-docs] is a globally distributed, multi-model database that allows you to choose from various database and consistency models to suit your needs.</span></span> <span data-ttu-id="dd8e4-140">Cosmos DB を使用すると、データをグローバルにレプリケートできます。デプロイして構成するクラスター管理コンポーネントやレプリケーション コンポーネントはありません。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-140">With Cosmos DB, your data can be globally replicated, and there is no cluster management or replication components to deploy and configure.</span></span>
* <span data-ttu-id="dd8e4-141">[Azure Monitor][azuremonitor-docs]: パフォーマンスの追跡、セキュリティの維持、傾向の把握に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-141">[Azure Monitor][azuremonitor-docs] helps you track performance, maintain security, and identify trends.</span></span> <span data-ttu-id="dd8e4-142">Monitor によって取得されたメトリックは、他のリソースやツール (Grafana など) で使用できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-142">Metrics obtained by Monitor can be used by other resources and tools, such as Grafana.</span></span>
* <span data-ttu-id="dd8e4-143">[Grafana][grafana]: メトリックのクエリの実行、視覚化、アラートの作成を行い、メトリックを理解するためのオープンソース ソリューションです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-143">[Grafana][grafana] is an open-source solution to query, visualize, alert, and understand metrics.</span></span> <span data-ttu-id="dd8e4-144">Azure Monitor のデータ ソース プラグインにより、Grafana では、Azure Kubernetes Service で実行され、Cosmos DB を使用するアプリケーションのパフォーマンスを監視するビジュアル ダッシュボードを作成できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-144">A data source plugin for Azure Monitor allows Grafana to create visual dashboards to monitor the performance of your applications running in Azure Kubernetes Service and using Cosmos DB.</span></span>

### <a name="alternatives"></a><span data-ttu-id="dd8e4-145">代替手段</span><span class="sxs-lookup"><span data-stu-id="dd8e4-145">Alternatives</span></span>

* <span data-ttu-id="dd8e4-146">[Azure Pipelines][azure-pipelines] を使用して、アプリの継続的インテグレーション (CI)、テスト、および継続的配置 (CD) のパイプラインを実装できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-146">[Azure Pipelines][azure-pipelines] help you implement a continuous integration (CI), test, and deployment (CD) pipeline for any app.</span></span>
* <span data-ttu-id="dd8e4-147">クラスターをより細かく制御する必要がある場合は、マネージド サービスを介してではなく、Azure VM 上で [Kubernetes][kubernetes] を直接実行できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-147">[Kubernetes][kubernetes] can be run directly on Azure VMs instead of via a managed service if you would like more control over the cluster.</span></span>
* <span data-ttu-id="dd8e4-148">[Service Fabric][service-fabric] は、AKS の代わりに使用できる別の代替コンテナー オーケストレーターです。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-148">[Service Fabric][service-fabric] is another alternate container orchestrator that can replace AKS.</span></span>

## <a name="considerations"></a><span data-ttu-id="dd8e4-149">考慮事項</span><span class="sxs-lookup"><span data-stu-id="dd8e4-149">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="dd8e4-150">可用性</span><span class="sxs-lookup"><span data-stu-id="dd8e4-150">Availability</span></span>

<span data-ttu-id="dd8e4-151">アプリケーションのパフォーマンスを監視し、問題を報告するために、このシナリオでは Azure Monitor と Grafana を組み合わせてビジュアル ダッシュボードを提供します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-151">To monitor your application performance and report on issues, this scenario combines Azure Monitor with Grafana for visual dashboards.</span></span> <span data-ttu-id="dd8e4-152">これらのツールを使用すると、コードの更新が必要になる可能性のあるパフォーマンスの問題を監視し、トラブルシューティングを行うことができます。コードの更新は、すべて CI/CD パイプラインを使用して展開できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-152">These tools let you monitor and troubleshoot performance issues that may require code updates, which can all then be deployed with the CI/CD pipeline.</span></span>

<span data-ttu-id="dd8e4-153">Azure Kubernetes Service クラスターに含まれるロード バランサーは、アプリケーションを実行する 1 つ以上のコンテナー (ポッド) にアプリケーション トラフィックを分散します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-153">As part of the Azure Kubernetes Service cluster, a load balancer distributes application traffic to one or more containers (pods) that run your application.</span></span> <span data-ttu-id="dd8e4-154">コンテナー化されたアプリケーションを Kubernetes で実行するこのアプローチにより、可用性の高いインフラストラクチャが顧客に提供されます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-154">This approach to running containerized applications in Kubernetes provides a highly available infrastructure for your customers.</span></span>

<span data-ttu-id="dd8e4-155">可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-155">For other availability topics, see the [availability checklist][availability] available in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="dd8e4-156">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="dd8e4-156">Scalability</span></span>

<span data-ttu-id="dd8e4-157">Azure Kubernetes Service を使用すると、アプリケーションの要求に応じてクラスタ ノードの数をスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-157">Azure Kubernetes Service lets you scale the number of cluster nodes to meet the demands of your applications.</span></span> <span data-ttu-id="dd8e4-158">アプリケーションの増加に伴って、サービスを実行する Kubernetes ノードの数をスケールアウトできます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-158">As your application increases, you can scale out the number of Kubernetes nodes that run your service.</span></span>

<span data-ttu-id="dd8e4-159">アプリケーション データは、グローバルにスケーリングできるグローバル分散型のマルチモデル データベースである Azure Cosmos DB に格納されます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-159">Application data is stored in Azure Cosmos DB, a globally distributed, multi-model database that can scale globally.</span></span> <span data-ttu-id="dd8e4-160">Cosmos DB は、従来のデータベース コンポーネントと同様に、インフラストラクチャをスケーリングする必要性を排除します。また、顧客の要求に応じて Cosmos DB をグローバルにレプリケートすることもできます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-160">Cosmos DB abstracts the need to scale your  infrastructure as with traditional database components, and you can choose to replicate your Cosmos DB globally to meet the demands of your customers.</span></span>

<span data-ttu-id="dd8e4-161">スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-161">For other scalability topics, see the [scalability checklist][scalability] available in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="dd8e4-162">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="dd8e4-162">Security</span></span>

<span data-ttu-id="dd8e4-163">攻撃フットプリントを最小限に抑えるために、このシナリオでは Jenkins VM インスタンスが HTTP 経由で公開されることはありません。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-163">To minimize the attack footprint, this scenario does not expose the Jenkins VM instance over HTTP.</span></span> <span data-ttu-id="dd8e4-164">Jenkins を操作する必要がある管理タスクについては、ローカル コンピューターから SSH トンネルを使用して、セキュリティで保護されたリモート接続を作成します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-164">For any management tasks that require you to interact with Jenkins, you create a secure remote connection using an SSH tunnel from your local machine.</span></span> <span data-ttu-id="dd8e4-165">Jenkins および Grafana VM インスタンスには、SSH 公開キー認証のみを使用できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-165">Only SSH public key authentication is allowed for the Jenkins and Grafana VM instances.</span></span> <span data-ttu-id="dd8e4-166">パスワード ベースのログインは無効になります。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-166">Password-based logins are disabled.</span></span> <span data-ttu-id="dd8e4-167">詳細については、「[Azure で Jenkins サーバーを実行する](../../reference-architectures/jenkins/index.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-167">For more information, see [Run a Jenkins server on Azure](../../reference-architectures/jenkins/index.md).</span></span>

<span data-ttu-id="dd8e4-168">資格情報とアクセス許可を分離するために、このシナリオでは専用の Azure Active Directory (AD) サービス プリンシパルを使用します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-168">For separation of credentials and permissions, this scenario uses a dedicated Azure Active Directory (AD) service principal.</span></span> <span data-ttu-id="dd8e4-169">このサービス プリンシパルの資格情報は、セキュリティで保護された資格情報オブジェクトとして Jenkins に保存されているため、スクリプトやビルド パイプライン内で直接公開されたり表示されたりすることはありません。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-169">The credentials for this service principal are stored as a secure credential object in Jenkins so that they are not directly exposed and visible within scripts or the build pipeline.</span></span>

<span data-ttu-id="dd8e4-170">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-170">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="dd8e4-171">回復性</span><span class="sxs-lookup"><span data-stu-id="dd8e4-171">Resiliency</span></span>

<span data-ttu-id="dd8e4-172">このシナリオでは、アプリケーションに Azure Kubernetes Service を使用します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-172">This scenario uses Azure Kubernetes Service for your application.</span></span> <span data-ttu-id="dd8e4-173">Kubernetes には、コンテナー (ポッド) を監視し、問題が発生した場合に再起動する回復性コンポーネントが組み込まれています。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-173">Built into Kubernetes are resiliency components that monitor and restart the containers (pods) if there is an issue.</span></span> <span data-ttu-id="dd8e4-174">実行中の複数の Kubernetes ノードと組み合わせることで、アプリケーションは使用できなくなっているポッドやノードを許容できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-174">Combined with running multiple Kubernetes nodes, your application can tolerate a pod or node being unavailable.</span></span>

<span data-ttu-id="dd8e4-175">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-175">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="dd8e4-176">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="dd8e4-176">Deploy the scenario</span></span>

<span data-ttu-id="dd8e4-177">**前提条件。**</span><span class="sxs-lookup"><span data-stu-id="dd8e4-177">**Prerequisites.**</span></span>

* <span data-ttu-id="dd8e4-178">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-178">You must have an existing Azure account.</span></span> <span data-ttu-id="dd8e4-179">Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-179">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>
* <span data-ttu-id="dd8e4-180">SSH 公開キー ペアが必要です。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-180">You need an SSH public key pair.</span></span> <span data-ttu-id="dd8e4-181">公開キー ペアの作成手順については、[Linux VM 用の SSH キー ペアの作成と使用][sshkeydocs]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-181">For steps on how to create a public key pair, see [Create and use an SSH key pair for Linux VMs][sshkeydocs].</span></span>
* <span data-ttu-id="dd8e4-182">サービスとリソースの認証用に Azure Active Directory (AD) サービス プリンシパルが必要です。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-182">You need an Azure Active Directory (AD) service principal for the authentication of service and resources.</span></span> <span data-ttu-id="dd8e4-183">必要に応じて、[az ad sp create-for-rbac][createsp] を使用してサービス プリンシパルを作成できます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-183">If needed, you can create a service principal with [az ad sp create-for-rbac][createsp]</span></span>

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    <span data-ttu-id="dd8e4-184">このコマンドの出力の *appId* と *password* をメモしておきます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-184">Make a note of the *appId* and *password* in the output from this command.</span></span> <span data-ttu-id="dd8e4-185">これらの値は、シナリオをデプロイするときにテンプレートに入力します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-185">You provide these values to the template when you deploy the scenario.</span></span>

<span data-ttu-id="dd8e4-186">Azure Resource Manager テンプレートを使用してこのシナリオをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-186">To deploy this scenario with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="dd8e4-187">**[Deploy to Azure]\(Azure にデプロイ\)** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-187">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="dd8e4-188">Azure portal でテンプレートのデプロイが開くまで待ってから、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-188">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="dd8e4-189">リソース グループを**新規作成**し、テキスト ボックスに名前 (例: *myAKSDevOpsScenario*) を指定します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-189">Choose to **Create new** resource group, then provide a name such as *myAKSDevOpsScenario* in the text box.</span></span>
   * <span data-ttu-id="dd8e4-190">**[場所]** ドロップダウン ボックスでリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-190">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="dd8e4-191">`az ad sp create-for-rbac` コマンドで出力されたサービス プリンシパルのアプリ ID とパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-191">Enter your service principal app ID and password from the `az ad sp create-for-rbac` command.</span></span>
   * <span data-ttu-id="dd8e4-192">Jenkins インスタンスと Grafana コンソールのユーザー名とセキュリティで保護されたパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-192">Provide a username and secure password for the Jenkins instance and Grafana console.</span></span>
   * <span data-ttu-id="dd8e4-193">Linux VM へのログインをセキュリティで保護する SSH キーを提供します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-193">Provide an SSH key to secure logins to the Linux VMs.</span></span>
   * <span data-ttu-id="dd8e4-194">使用条件を確認し、**[上記の使用条件に同意する]** をオンにします。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-194">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="dd8e4-195">**[購入]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-195">Select the **Purchase** button.</span></span>

<span data-ttu-id="dd8e4-196">デプロイが完了するまでに 15 から 20 分かかることがあります。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-196">It can take 15-20 minutes for the deployment to complete.</span></span>

## <a name="pricing"></a><span data-ttu-id="dd8e4-197">価格</span><span class="sxs-lookup"><span data-stu-id="dd8e4-197">Pricing</span></span>

<span data-ttu-id="dd8e4-198">このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-198">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="dd8e4-199">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-199">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="dd8e4-200">保存するコンテナー イメージの数とアプリケーションを実行する Kubernetes ノードの数に基づいて、3 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-200">We have provided three sample cost profiles based on the number of container images to store and Kubernetes nodes to run your applications.</span></span>

* <span data-ttu-id="dd8e4-201">[Small][small-pricing]: この価格例は、1 か月あたり 1,000 個のコンテナー ビルドに対応します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-201">[Small][small-pricing]: this pricing example correlates to 1000 container builds per month.</span></span>
* <span data-ttu-id="dd8e4-202">[Medium][medium-pricing]: この価格例は、1 か月あたり 100,000 個のコンテナー ビルドに対応します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-202">[Medium][medium-pricing]: this pricing example correlates to 100,000 container builds per month.</span></span>
* <span data-ttu-id="dd8e4-203">[Large][large-pricing]: この価格例は、1 か月あたり 1,000,000 個のコンテナー ビルドに対応します。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-203">[Large][large-pricing]: this pricing example correlates to 1,000,000 container builds per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="dd8e4-204">関連リソース</span><span class="sxs-lookup"><span data-stu-id="dd8e4-204">Related Resources</span></span>

<span data-ttu-id="dd8e4-205">このシナリオでは、Azure Container Registry と Azure Kubernetes Service を使用して、コンテナー ベースのアプリケーションを格納し、実行しました。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-205">This scenario used Azure Container Registry and Azure Kubernetes Service to store and run your container-based applications.</span></span> <span data-ttu-id="dd8e4-206">オーケストレーション コンポーネントをプロビジョニングしなくても、Azure Container Instances を使用して、コンテナー ベースのアプリケーションを実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-206">Azure Container Instances can also be used to run container-based applications, without having to provision any orchestration components.</span></span> <span data-ttu-id="dd8e4-207">詳細については、[Azure Container Instances の概要][azureaci-docs]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="dd8e4-207">For more information, see [Azure Container Instances overview][azureaci-docs].</span></span>

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[azure-pipelines]: /azure/devops/pipelines
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64