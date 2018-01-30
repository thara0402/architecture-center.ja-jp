---
title: "Azure で Jenkins サーバーを実行する"
description: "このリファレンス アーキテクチャでは、シングル サインオン (SSO) で保護されたスケーラブルなエンタープライズ レベルの Jenkins サーバーを Azure にデプロイして運用する方法を示します。"
author: njray
ms.date: 01/21/18
ms.openlocfilehash: d06b16c212951c629612d69b13fa2b32b1030475
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/26/2018
---
# <a name="run-a-jenkins-server-on-azure"></a><span data-ttu-id="d4f7b-103">Azure で Jenkins サーバーを実行する</span><span class="sxs-lookup"><span data-stu-id="d4f7b-103">Run a Jenkins server on Azure</span></span>

<span data-ttu-id="d4f7b-104">このリファレンス アーキテクチャでは、シングル サインオン (SSO) で保護されたスケーラブルなエンタープライズ レベルの Jenkins サーバーを Azure にデプロイして運用する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-104">This reference architecture shows how to deploy and operate a scalable, enterprise-grade Jenkins server on Azure secured with single sign-on (SSO).</span></span> <span data-ttu-id="d4f7b-105">また、このアーキテクチャでは、Azure Monitor を使用して Jenkins サーバーの状態を監視します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-105">The architecture also uses Azure Monitor to monitor the state of the Jenkins server.</span></span> [<span data-ttu-id="d4f7b-106">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

![Azure で実行される Jenkins サーバー][0]

<span data-ttu-id="d4f7b-108">*このアーキテクチャ ダイアグラムを含む [Visio ファイル](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx)をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="d4f7b-108">*Download a [Visio file](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx) that contains this architecture diagram.*</span></span>

<span data-ttu-id="d4f7b-109">このアーキテクチャは、Azure サービスを使用したディザスター リカバリーをサポートしていますが、複数マスターやダウンタイムのない高可用性 (HA) を必要とする高度なスケールアウト シナリオには対応していません。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-109">This architecture supports disaster recovery with Azure services but does not cover more advanced scale-out scenarios involving multiple masters or high availability (HA) with no downtime.</span></span> <span data-ttu-id="d4f7b-110">Azure での CI/CD パイプラインの構築の手順を示すチュートリアルなど、さまざまな Azure コンポーネントに関する全般的な情報については、「[Azure 上の Jenkins][jenkins-on-azure]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-110">For general insights about the various Azure components, including a step-by-step tutorial about building out a CI/CD pipeline on Azure, see [Jenkins on  Azure][jenkins-on-azure].</span></span>

<span data-ttu-id="d4f7b-111">このドキュメントでは、Azure Storage を使用したビルド成果物の保持、SSO に必要なセキュリティ項目、統合可能なその他のサービス、パイプラインのスケーラビリティなど、Jenkins をサポートするために必要となる中心的な Azure 操作について説明します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-111">The focus of this document is on the core Azure operations needed to support Jenkins, including the use of Azure Storage to maintain build artifacts, the security items needed for SSO, other services that can be integrated, and scalability for the pipeline.</span></span> <span data-ttu-id="d4f7b-112">このアーキテクチャは、既存のソース管理リポジトリと連携するように設計されています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-112">The architecture is designed to work with an existing source control repository.</span></span> <span data-ttu-id="d4f7b-113">たとえば、一般的なシナリオとして、GitHub のコミットに基づいて Jenkins ジョブを開始します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-113">For example, a common scenario is to start Jenkins jobs based on GitHub commits.</span></span>

## <a name="architecture"></a><span data-ttu-id="d4f7b-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="d4f7b-114">Architecture</span></span>

<span data-ttu-id="d4f7b-115">アーキテクチャは、次のコンポーネントで構成されています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-115">The architecture consists of the following components:</span></span>

-   <span data-ttu-id="d4f7b-116">**リソース グループ。**</span><span class="sxs-lookup"><span data-stu-id="d4f7b-116">**Resource group.**</span></span> <span data-ttu-id="d4f7b-117">[リソース グループ][rg]を使用して Azure 資産をグループ化し、有効期間、所有者などの条件に基づいて管理できるようにします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-117">A [resource group][rg] is used to group Azure assets so they can be managed by lifetime, owner, and other criteria.</span></span> <span data-ttu-id="d4f7b-118">リソース グループを使用して、Azure 資産をグループとしてデプロイおよび監視したり、リソース グループ別に請求コストを追跡したりできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-118">Use resource groups to deploy and monitor Azure assets as a group and track billing costs by resource group.</span></span> <span data-ttu-id="d4f7b-119">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-119">You can also delete resources as a set, which is very useful for test deployments.</span></span>

-   <span data-ttu-id="d4f7b-120">**Jenkins サーバー**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-120">**Jenkins server**.</span></span> <span data-ttu-id="d4f7b-121">[Jenkins][azure-market] をオートメーション サーバーとして実行し、Jenkins マスターとして機能させるために、仮想マシンをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-121">A virtual machine is deployed to run [Jenkins][azure-market] as an automation server and serve as Jenkins Master.</span></span> <span data-ttu-id="d4f7b-122">このリファレンス アーキテクチャでは、Azure の Linux (Ubuntu 14.04 LTS) 仮想マシンにインストールされた、[Azure 上の Jenkins 用ソリューション テンプレート][solution]を使用します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-122">This reference architecture uses the [solution template for Jenkins on Azure][solution], installed on a Linux (Ubuntu 14.04 LTS) virtual machine on Azure.</span></span> <span data-ttu-id="d4f7b-123">他の Jenkins 製品は、Azure Marketplace で入手できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-123">Other Jenkins offerings are available in the Azure Marketplace.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d4f7b-124">Nginx を VM にインストールして、Jenkins のリバース プロキシとして動作させます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-124">Nginx is installed on the VM to act as a reverse proxy to Jenkins.</span></span> <span data-ttu-id="d4f7b-125">Jenkins サーバーの SSL を有効にするように Nginx を構成できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-125">You can configure Nginx to enable SSL for the Jenkins server.</span></span>
    > 
    > 

-   <span data-ttu-id="d4f7b-126">**仮想ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-126">**Virtual network**.</span></span> <span data-ttu-id="d4f7b-127">[仮想ネットワーク][vnet]は Azure リソースを相互に接続し、論理的な分離を実現します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-127">A [virtual network][vnet] connects Azure resources to each other and provides logical isolation.</span></span> <span data-ttu-id="d4f7b-128">このアーキテクチャでは、Jenkins サーバーは仮想ネットワークで実行されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-128">In this architecture, the Jenkins server runs in a virtual network.</span></span>

-   <span data-ttu-id="d4f7b-129">**サブネット**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-129">**Subnets**.</span></span> <span data-ttu-id="d4f7b-130">パフォーマンスに影響を与えずに、ネットワーク トラフィックを簡単に管理および分離できるように、Jenkins サーバーは[サブネット][subnet]に分離されています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-130">The Jenkins server is isolated in a [subnet][subnet] to make it easier to manage and segregate network traffic without impacting performance.</span></span>

-   <span data-ttu-id="d4f7b-131">**NSG**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-131">**NSGs**.</span></span> <span data-ttu-id="d4f7b-132">[ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、インターネットから仮想ネットワークのサブネットへのネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-132">Use [network security groups][nsg] (NSGs) to restrict network traffic from the Internet to the subnet of a virtual network.</span></span>

-   <span data-ttu-id="d4f7b-133">**Managed Disks**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-133">**Managed disks**.</span></span> <span data-ttu-id="d4f7b-134">[管理ディスク][managed-disk]は、アプリケーション ストレージに使用される永続的な仮想ハード ディスク (VHD) です。また、Jenkins サーバーの状態を保持し、ディザスター リカバリーを提供します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-134">A [managed disk][managed-disk] is a persistent virtual hard disk (VHD) used for application storage and also to maintain the state of the Jenkins server and provide disaster recovery.</span></span> <span data-ttu-id="d4f7b-135">データ ディスクは、Azure Storage に格納されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-135">Data disks are stored in Azure Storage.</span></span> <span data-ttu-id="d4f7b-136">高パフォーマンスを実現するために、[Premium Storage][premium] が推奨されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-136">For high performance, [premium storage][premium] is recommended.</span></span>

-   <span data-ttu-id="d4f7b-137">**Azure Blob Storage**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-137">**Azure Blob Storage**.</span></span> <span data-ttu-id="d4f7b-138">[Windows Azure Storage プラグイン][configure-storage]では、Azure Blob Storage を使用して、他の Jenkins ビルドと共有する作成済みのビルド成果物を保存します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-138">The [Windows Azure Storage plugin][configure-storage] uses Azure Blob  Storage to store the build artifacts that are created and shared with other Jenkins builds.</span></span>

-   <span data-ttu-id="d4f7b-139">**Azure Active Directory (Azure AD)**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-139">**Azure Active Directory (Azure AD)**.</span></span> <span data-ttu-id="d4f7b-140">[Azure AD][azure-ad] はユーザー認証をサポートしているので、SSO を設定できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-140">[Azure AD][azure-ad] supports user authentication, allowing you to set up SSO.</span></span> <span data-ttu-id="d4f7b-141">Azure AD [サービス プリンシパル][service-principal]は、[ロールベースのアクセス制御][rbac] (RBAC) を使用して、ワークフローの各ロールの承認に関するポリシーとアクセス許可を定義します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-141">Azure AD [service principals][service-principal] define the policy and permissions for each role authorization in the workflow via [role-based access control][rbac] (RBAC).</span></span> <span data-ttu-id="d4f7b-142">各サービス プリンシパルは、Jenkins ジョブに関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-142">Each service principal is associated with a Jenkins job.</span></span>

-   <span data-ttu-id="d4f7b-143">**Azure Key Vault**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-143">**Azure Key Vault.**</span></span> <span data-ttu-id="d4f7b-144">シークレットが必要なときに、Azure リソースのプロビジョニングに使用するシークレットと暗号化キーを管理するために、このアーキテクチャでは [Key Vault][key-vault] を使用します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-144">To manage secrets and cryptographic keys used to provision Azure resources when secrets are required, this architecture uses [Key Vault][key-vault].</span></span> <span data-ttu-id="d4f7b-145">パイプラインのアプリケーションに関連するシークレットの保存については、Jenkins の [Azure Credentials][configure-credential] プラグインも参照してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-145">For added help storing secrets associated with the application in the pipeline, see also the [Azure Credentials][configure-credential] plugin for Jenkins.</span></span>

-   <span data-ttu-id="d4f7b-146">**Azure Monitoring サービス**。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-146">**Azure monitoring services**.</span></span> <span data-ttu-id="d4f7b-147">このサービスは、Jenkins をホストする Azure 仮想マシンを[監視][monitor]します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-147">This service [monitors][monitor] the Azure virtual machine hosting Jenkins.</span></span> <span data-ttu-id="d4f7b-148">このデプロイでは、仮想マシンの状態と CPU 使用率を監視し、アラートを送信します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-148">This deployment monitors the virtual machine status and CPU utilization and sends alerts.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d4f7b-149">Recommendations</span><span class="sxs-lookup"><span data-stu-id="d4f7b-149">Recommendations</span></span>

<span data-ttu-id="d4f7b-150">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="d4f7b-151">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="azure-ad"></a><span data-ttu-id="d4f7b-152">Azure AD</span><span class="sxs-lookup"><span data-stu-id="d4f7b-152">Azure AD</span></span>

<span data-ttu-id="d4f7b-153">Azure サブスクリプションの [Azure AD][azure-ad] テナントを使用して、Jenkins ユーザーの SSO を有効にし、Jenkins ジョブが Azure リソースにアクセスできるようにするための[サービス プリンシパル][service-principal]を設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-153">The [Azure AD][azure-ad] tenant for your Azure subscription is used to enable SSO for Jenkins users and set up [service principals][service-principal] that enable Jenkins jobs to access Azure resources.</span></span>

<span data-ttu-id="d4f7b-154">SSO の認証と承認は、Jenkins サーバーにインストールされた Azure AD プラグインによって実装されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-154">SSO authentication and authorization are implemented by the Azure AD plugin  installed on the Jenkins server.</span></span> <span data-ttu-id="d4f7b-155">SSO では、Jenkins サーバーにログオンするときに、Azure AD の組織の資格情報を使用して認証できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-155">SSO allows you to authenticate using your organization credentials from Azure AD when logging on to the Jenkins server.</span></span> <span data-ttu-id="d4f7b-156">Azure AD プラグインを構成するときに、Jenkin サーバーへのユーザーの承認済みアクセスのレベルを指定できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-156">When configuring the Azure AD plugin, you can specify the level of a user’s authorized access to the Jenkin server.</span></span>

<span data-ttu-id="d4f7b-157">Jenkins ジョブが Azure リソースにアクセスできるようにするには、Azure AD 管理者がサービス プリンシパルを作成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-157">To provide Jenkins jobs with access to Azure resources, an Azure AD administrator creates service principals.</span></span> <span data-ttu-id="d4f7b-158">これらにより、Azure リソースへの[認証、承認されたアクセス][ad-sp]がアプリケーション (ここでは Jenkins ジョブ) に付与されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-158">These grant applications—in this case, the Jenkins jobs—[authenticated, authorized access][ad-sp] to Azure resources.</span></span>

<span data-ttu-id="d4f7b-159">[RBAC][rbac] により、割り当てられたロールを通じてユーザーまたはサービス プリンシパルの Azure リソースへのアクセスがさらに明確に定義され、制御されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-159">[RBAC][rbac] further defines and controls access to Azure resources for users or service principals through their assigned role.</span></span> <span data-ttu-id="d4f7b-160">組み込みロールとカスタム ロールの両方がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-160">Both built-in and custom roles are supported.</span></span> <span data-ttu-id="d4f7b-161">また、ロールは、パイプラインをセキュリティで保護したり、ユーザーやエージェントの責任が適切に割り当てられ、承認されていることを保証したりするうえでも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-161">Roles also help secure the pipeline and ensure that a user’s or agent’s responsibilities are assigned and authorized correctly.</span></span> <span data-ttu-id="d4f7b-162">さらに、Azure 資産へのアクセスを制限するように RBAC を設定することもできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-162">In addition, RBAC can be set up to limit access to Azure assets.</span></span> <span data-ttu-id="d4f7b-163">たとえば、ユーザーが特定のリソース グループ内の資産だけを使用するように制限できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-163">For example, a user can be limited to working with only the assets in a particular resource group.</span></span>

### <a name="storage"></a><span data-ttu-id="d4f7b-164">Storage</span><span class="sxs-lookup"><span data-stu-id="d4f7b-164">Storage</span></span>

<span data-ttu-id="d4f7b-165">Azure Marketplace からインストールされた Jenkins の [Windows Azure Storage プラグイン][storage-plugin]を使用して、他のビルドやテストと共有できるビルド成果物を保存します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-165">Use the Jenkins [Windows Azure Storage plugin][storage-plugin], which is installed from the Azure Marketplace, to store build artifacts that can be shared with other builds and tests.</span></span> <span data-ttu-id="d4f7b-166">このプラグインを Jenkins ジョブで使用するには、Azure ストレージ アカウントを構成しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-166">An Azure Storage account must be configured before this plugin can be used by the Jenkins jobs.</span></span>

### <a name="jenkins-azure-plugins"></a><span data-ttu-id="d4f7b-167">Jenkins の Azure プラグイン</span><span class="sxs-lookup"><span data-stu-id="d4f7b-167">Jenkins Azure plugins</span></span>

<span data-ttu-id="d4f7b-168">Azure 上の Jenkins 用ソリューション テンプレートでは、複数の Azure プラグインをインストールします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-168">The solution template for Jenkins on Azure installs several Azure plugins.</span></span> <span data-ttu-id="d4f7b-169">Azure DevOps チームがこのソリューション テンプレートと次のプラグインを作成し、管理しています。各プラグインは、Azure Marketplace の他の Jenkins 製品や、オンプレミスでの Jenkins マスターのセットアップで動作します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-169">The Azure DevOps Team builds and maintains the solution template and the following plugins, which work with other Jenkins offerings in Azure Marketplace as well as any Jenkins master set up on premises:</span></span>

-   <span data-ttu-id="d4f7b-170">[Azure AD プラグイン][configure-azure-ad]: Jenkins サーバーが Azure AD に基づいてユーザーの SSO をサポートできるようにします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-170">[Azure AD plugin][configure-azure-ad] allows the Jenkins server to support SSO for users based on Azure AD.</span></span>

-   <span data-ttu-id="d4f7b-171">[Azure VM Agents プラグイン][configure-agent]: Azure Resource Manager (ARM) テンプレートを使用して、Azure 仮想マシンに Jenkins エージェントを作成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-171">[Azure VM Agents][configure-agent] plugin uses an Azure Resource Manager (ARM) template to create Jenkins agents in Azure virtual machines.</span></span>

-   <span data-ttu-id="d4f7b-172">[Azure Credentials プラグイン][configure-credential]: Azure サービス プリンシパルの資格情報を Jenkins に保存できるようにします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-172">[Azure Credentials][configure-credential] plugin allows you to store Azure service principal credentials in Jenkins.</span></span>

-   <span data-ttu-id="d4f7b-173">[Windows Azure Storage プラグイン][configure-storage]: ビルド成果物を [Azure Blob Storage][blob] にアップロードしたり、ビルド依存関係をダウンロードしたりします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-173">[Windows Azure Storage plugin][configure-storage] uploads build artifacts to, or downloads build dependencies from, [Azure Blob storage][blob].</span></span>

<span data-ttu-id="d4f7b-174">Azure リソースで動作する利用可能なすべての Azure プラグインの一覧を確認することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-174">We also recommend reviewing the growing list of all available Azure plugins that work with Azure resources.</span></span> <span data-ttu-id="d4f7b-175">最新の一覧を確認するには、[Jenkins プラグイン インデックス][index]のページにアクセスし、Azure を検索してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-175">To see all the latest list, visit [Jenkins Plugin Index][index] and search for Azure.</span></span> <span data-ttu-id="d4f7b-176">たとえば、次のプラグインをデプロイに使用できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-176">For example, the following plugins are available for deployment:</span></span>

-   <span data-ttu-id="d4f7b-177">[Azure Container Agents][container-agents]: Jenkins でコンテナーをエージェントとして実行できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-177">[Azure Container Agents][container-agents] helps you to run a container as an agent in Jenkins.</span></span>

-   <span data-ttu-id="d4f7b-178">[Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s): リソース構成を Kubernetes クラスターにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-178">[Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) deploys resource configurations to a Kubernetes cluster.</span></span>

-   <span data-ttu-id="d4f7b-179">[Azure Container Service][acs]: Kubernetes、DC/OS と Marathon、または Docker Swarm を使用する Azure Container Service に構成をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-179">[Azure Container Service][acs] deploys configurations to Azure Container Service with Kubernetes, DC/OS with Marathon, or Docker Swarm.</span></span>

-   <span data-ttu-id="d4f7b-180">[Azure Functions][functions]: プロジェクトを Azure Function にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-180">[Azure Functions][functions] deploys your project to Azure Function.</span></span>

-   <span data-ttu-id="d4f7b-181">[Azure App Service][app-service]: Azure App Service にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-181">[Azure App Service][app-service] deploys to Azure App Service.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d4f7b-182">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d4f7b-182">Scalability considerations</span></span>

<span data-ttu-id="d4f7b-183">非常に大規模なワークロードをサポートするために、Jenkins をスケールできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-183">Jenkins can scale to support very large workloads.</span></span> <span data-ttu-id="d4f7b-184">エラスティック ビルドの場合、Jenkins マスター サーバーでビルドを実行しないでください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-184">For elastic builds, do not run builds on the Jenkins master server.</span></span> <span data-ttu-id="d4f7b-185">代わりに、必要に応じて弾力的にスケールインおよびスケールアウトできる Jenkins エージェントにビルド タスクをオフロードします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-185">Instead, offload build tasks to Jenkins agents, which can be elastically scaled in and out as need.</span></span> <span data-ttu-id="d4f7b-186">エージェントをスケールする場合、次の 2 つのオプションを検討してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-186">Consider two options for scaling agents:</span></span>

- <span data-ttu-id="d4f7b-187">[Azure VM Agents][vm-agent] プラグインを使用して、Azure VM で実行する Jenkins エージェントを作成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-187">Use the [Azure VM Agents][vm-agent] plugin to create Jenkins agents that run in Azure VMs.</span></span> <span data-ttu-id="d4f7b-188">このプラグインを使用すると、エージェントの柔軟なスケールアウトが可能になり、異なる種類の仮想マシンを使用できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-188">This plugin enables elastic scale-out for agents and can use distinct types of virtual machines.</span></span> <span data-ttu-id="d4f7b-189">Azure Marketplace から別の基本イメージを選択することも、カスタム イメージを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-189">You can select a different base image from Azure Marketplace or use a custom image.</span></span> <span data-ttu-id="d4f7b-190">Jenkins エージェントをスケールする方法の詳細については、Jenkins ドキュメントの「[Architecting for Scale (スケールのための設計)][scale]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-190">For details about how the Jenkins agents scale, see [Architecting for Scale][scale] in the Jenkins documentation.</span></span>

- <span data-ttu-id="d4f7b-191">[Azure Container Agents][container-agents] プラグインを使用して、[Kubernetes を使用する Azure Container Service](/azure/container-service/kubernetes/) または [Azure Container Instances](/azure/container-instances/) でコンテナーをエージェントとして実行します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-191">Use the [Azure Container Agents][container-agents] plugin to run a container as an agent in either [Azure Container Service with Kubernetes](/azure/container-service/kubernetes/), or [Azure Container Instances](/azure/container-instances/).</span></span>

<span data-ttu-id="d4f7b-192">一般に、仮想マシンはコンテナーよりもスケールにコストがかかります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-192">Virtual machines generally cost more to scale than containers.</span></span> <span data-ttu-id="d4f7b-193">ただし、スケーリングのためにコンテナーを使用するには、コンテナーでビルド プロセスを実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-193">To use containers for scaling, however, your build process must run with containers.</span></span>

<span data-ttu-id="d4f7b-194">また、Azure Storage を使用して、パイプラインの次の段階で他のビルド エージェントが使用する可能性のあるビルド成果物を共有します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-194">Also, use Azure Storage to share build artifacts that may be used in the next stage of the pipeline by other build agents.</span></span>

### <a name="scaling-the-jenkins-server"></a><span data-ttu-id="d4f7b-195">Jenkins サーバーのスケーリング</span><span class="sxs-lookup"><span data-stu-id="d4f7b-195">Scaling the Jenkins server</span></span> 

<span data-ttu-id="d4f7b-196">Jenkins サーバー VM をスケールアップまたはスケールダウンするには、VM サイズを変更します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-196">You can scale the Jenkins server VM up or down by changing the VM size.</span></span> <span data-ttu-id="d4f7b-197">[Azure 上の Jenkins 用ソリューション テンプレート][azure-market]では、既定で DS2 v2 サイズ (2 つの CPU、7 GB) が指定されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-197">The [solution template for Jenkins on Azure][azure-market] specifies the DS2 v2 size (with two CPUs, 7 GB) by default.</span></span> <span data-ttu-id="d4f7b-198">このサイズは、小規模から中規模のチーム ワークロードに対応します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-198">This size handles a small to medium team workload.</span></span> <span data-ttu-id="d4f7b-199">サーバーを構築するときに別のオプションを選択して、VM サイズを変更します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-199">Change the VM size by choosing a different option when building out the server.</span></span> 

<span data-ttu-id="d4f7b-200">適切なサーバー サイズの選択は、予想されるワークロードのサイズによって異なります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-200">Selecting the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="d4f7b-201">Jenkins コミュニティは、要件に最も適した構成を特定する際に役立つ[選択ガイド][selection-guide]を保持しています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-201">The Jenkins community maintains a [selection guide][selection-guide] to help identify the configuration that best meets your requirements.</span></span> <span data-ttu-id="d4f7b-202">Azure には、あらゆる要件に対応するために、[Linux VM のさまざまなサイズ][sizes-linux]が用意されています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-202">Azure offers many [sizes for Linux VMs][sizes-linux] to meet any requirements.</span></span> <span data-ttu-id="d4f7b-203">Jenkins マスターのスケーリングの詳細については、Jenkins コミュニティの[ベスト プラクティス][best-practices]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-203">For more information about scaling the Jenkins master, refer to the Jenkins community of [best practices][best-practices], which also includes details about scaling Jenkins master.</span></span>


## <a name="availability-considerations"></a><span data-ttu-id="d4f7b-204">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d4f7b-204">Availability considerations</span></span>

<span data-ttu-id="d4f7b-205">ワークフローの可用性の要件と、Jenkins サーバーがダウンした場合に Jenkins の状態を回復する方法を評価します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-205">Assess the availability requirements for your workflow and how to recover the Jenkin state should the Jenkin server goes down.</span></span> <span data-ttu-id="d4f7b-206">可用性の要件を評価するには、次の 2 つの一般的なメトリックを検討します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-206">To assess your availability requirements, consider two common metrics:</span></span>

-   <span data-ttu-id="d4f7b-207">目標復旧時間 (RTO) は、Jenkins なしで継続できる時間を示します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-207">Recovery Time Objective (RTO) specifies how long you can go without Jenkins.</span></span>

-   <span data-ttu-id="d4f7b-208">目標復旧時点 (RPO) は、サービスの中断が Jenkins に影響を及ぼす場合に、失われても差し支えないデータの量を示します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-208">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects Jenkins.</span></span>

<span data-ttu-id="d4f7b-209">実際には、RTO と RPO は冗長性とバックアップを意味します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-209">In practice, RTO and RPO imply redundancy and backup.</span></span> <span data-ttu-id="d4f7b-210">可用性は、Azure の一部であるハードウェアの復旧の問題ではなく、Jenkins サーバーの状態を確実に維持することを表します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-210">Availability is not a question of hardware recovery—that is part of Azure—but rather ensuring you maintain the state of your Jenkins server.</span></span> <span data-ttu-id="d4f7b-211">このリファレンス アーキテクチャでは、1 つの仮想マシンについて 99.9% のアップタイムを保証する、[Azure サービス レベル アグリーメント][sla] (SLA) を使用します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-211">This reference architecture uses the [Azure service level agreement][sla] (SLA), which guarantees 99.9-percent uptime for a single virtual machine.</span></span> <span data-ttu-id="d4f7b-212">この SLA がアップタイムの要件を満たしていない場合は、ディザスター リカバリーを計画するか、[マルチマスター Jenkins サーバー][multi-master] デプロイ (このドキュメントでは取り上げていません) の使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-212">If this SLA doesn't meet your uptime requirements, make sure you have a plan for disaster recovery, or consider using a [multi-master Jenkins server][multi-master] deployment (not covered in this document).</span></span>

<span data-ttu-id="d4f7b-213">デプロイの手順 7 のディザスター リカバリー [スクリプト][disaster]を使用して、Jenkins サーバーの状態を保存する管理ディスクを使用する Azure ストレージ アカウントを作成することを検討します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-213">Consider using the disaster recovery [scripts][disaster] in step 7 of the deployment to create an Azure Storage account with managed disks to store the Jenkins server state.</span></span> <span data-ttu-id="d4f7b-214">Jenkins がダウンした場合、この別のストレージ アカウントに保存されている状態に復元できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-214">If Jenkins goes down, it can be restored to the state stored in this separate storage account.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="d4f7b-215">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d4f7b-215">Security considerations</span></span>

<span data-ttu-id="d4f7b-216">基本的な Jenkins サーバーは基本的な状態では安全でないため、次の方法を使用してサーバーのセキュリティを強化します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-216">Use the following approaches to help lock down security on a basic Jenkins server, since in its basic state, it is not secure.</span></span>

-   <span data-ttu-id="d4f7b-217">Jenkins サーバーへのログオンをセキュリティで保護する方法を設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-217">Set up a way to secure logon to the Jenkin server.</span></span> <span data-ttu-id="d4f7b-218">HTTP は安全ではありませんが、既定では、このアーキテクチャは HTTP とパブリック IP を使用します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-218">HTTP is not secure By default, this architecture uses HTTP and has a public IP.</span></span> <span data-ttu-id="d4f7b-219">セキュリティで保護されたログオンに使用する [Nginx サーバーで HTTPS][nginx] を設定することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-219">Consider setting up [HTTPS on the Nginx server][nginx] being used for a secure logon.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d4f7b-220">サーバーに SSL を追加するときは、Jenkins サブネットの NSG ルールを作成してポート 443 を開きます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-220">When adding SSL to your server, create an NSG rule for the Jenkins subnet to open port 443.</span></span> <span data-ttu-id="d4f7b-221">詳細については、「[Azure Portal を使用して仮想マシンへのポートを開く方法][port443]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-221">For more information, see [How to open ports to a virtual machine with the Azure portal][port443].</span></span>
    > 

-   <span data-ttu-id="d4f7b-222">Jenkins の構成がクロスサイト リクエスト フォージェリを防いでいることを確認します ([Manage Jenkins]\(Jenkins の管理\) \> [Configure Global Security]\(グローバル セキュリティの構成\))。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-222">Ensure that the Jenkins configuration prevents cross site request forgery (Manage Jenkins \> Configure Global Security).</span></span> <span data-ttu-id="d4f7b-223">これは Microsoft Jenkins Server の既定値です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-223">This is the default for Microsoft Jenkins Server.</span></span>

-   <span data-ttu-id="d4f7b-224">[Matrix Authorization Strategy プラグイン][matrix]を使用して、Jenkins ダッシュボードへの読み取り専用アクセスを構成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-224">Configure read-only access to the Jenkins dashboard by using the [Matrix Authorization Strategy Plugin][matrix].</span></span>

-   <span data-ttu-id="d4f7b-225">[Azure Credentials][configure-credential] プラグインをインストールし、Key Vault を使用して、Azure 資産、パイプライン内のエージェント、およびサード パーティ コンポーネントのシークレットを処理します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-225">Install the [Azure Credentials][configure-credential] plugin to use Key Vault to handle secrets for the Azure assets, the agents in the pipeline, and third-party components.</span></span>

-   <span data-ttu-id="d4f7b-226">ユーザー、サービス、パイプライン エージェントがジョブを実行するために必要なリソースを定義するセキュリティ プロファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-226">Create a security profile that defines the resources required by users, services, and pipeline agents to do their jobs—but no more.</span></span> <span data-ttu-id="d4f7b-227">この手順は、セキュリティ設定を検討するときに重要になります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-227">This step becomes critical when considering your security settings.</span></span>

<span data-ttu-id="d4f7b-228">多くの場合、Jenkins ジョブは、承認が必要な Azure サービス (Azure Container Service など) にアクセスするためにシークレットを必要とします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-228">Jenkins jobs often require secrets to access Azure services that require authorization, such as Azure Container Service.</span></span> <span data-ttu-id="d4f7b-229">[Azure Credentials プラグイン][configure-credential]と共に [Key Vault][key-vault] を使用して、これらのシークレットを安全に管理します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-229">Use [Key Vault][key-vault] along with the [Azure Credential plugin][configure-credential] to manage these secrets securely.</span></span> <span data-ttu-id="d4f7b-230">Key Vault を使用して、サービス プリンシパルの資格情報、パスワード、トークン、その他のシークレットを保存します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-230">Use Key Vault to store service principal credentials, passwords, tokens, and other secrets.</span></span>

<span data-ttu-id="d4f7b-231">Azure リソースのセキュリティの状態を一元的に表示して把握するには、[Azure Security Center][security-center] を使用します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-231">To get a central view of the security state of your Azure resources, use [Azure Security Center][security-center].</span></span> <span data-ttu-id="d4f7b-232">Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-232">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="d4f7b-233">セキュリティ センターは、Azure サブスクリプションごとに構成されます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-233">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="d4f7b-234">「[Azure Security Center クイック スタート ガイド][quick-start]」の説明に従って、セキュリティ データの収集を有効にします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-234">Enable security data collection as described in the [Azure Security Center quick start guide][quick-start].</span></span> <span data-ttu-id="d4f7b-235">データ収集を有効にすると、Security Center は、そのサブスクリプションに作成されているすべての仮想マシンを自動的にスキャンします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-235">When data collection is enabled, Security Center automatically scans any virtual machines created under that subscription.</span></span>

<span data-ttu-id="d4f7b-236">Jenkins サーバーには独自のユーザー管理システムがあります。Jenkins コミュニティでは、[Azure 上の Jenkins インスタンスのセキュリティ保護][secure-jenkins]に関するベスト プラクティスを提供しています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-236">The Jenkins server has its own user management system, and the Jenkins community provides best practices for [securing a Jenkins instance on Azure][secure-jenkins].</span></span> <span data-ttu-id="d4f7b-237">Azure 上の Jenkins 用ソリューション テンプレートは、これらのベストプラクティスを実装しています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-237">The solution template for Jenkins on Azure implements these best practices.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="d4f7b-238">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="d4f7b-238">Manageability considerations</span></span>

<span data-ttu-id="d4f7b-239">リソース グループを使用して、デプロイされている Azure リソースを整理します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-239">Use resource groups to organize the Azure resources that are deployed.</span></span> <span data-ttu-id="d4f7b-240">運用環境と開発/テスト環境を別々のリソース グループに配置することで、各環境のリソースを監視し、リソース グループ別に請求コストをまとめることができます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-240">Deploy production environments and development/test environments in separate resource groups, so that you can monitor each environment’s resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="d4f7b-241">セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-241">You can also delete resources as a set, which is very useful for test deployments.</span></span>

<span data-ttu-id="d4f7b-242">Azure には、インフラストラクチャ全体の[監視と診断][monitoring-diag]のための機能がいくつか用意されています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-242">Azure provides several features for [monitoring and diagnostics][monitoring-diag] of the overall infrastructure.</span></span> <span data-ttu-id="d4f7b-243">CPU 使用率を監視するために、このアーキテクチャでは Azure Monitor をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-243">To monitor CPU usage, this architecture deploys Azure Monitor.</span></span> <span data-ttu-id="d4f7b-244">たとえば、Azure Monitor を使用して CPU 使用率を監視し、CPU 使用率が 80% を超えた場合に通知を送信できます </span><span class="sxs-lookup"><span data-stu-id="d4f7b-244">For example, you can use Azure Monitor to monitor CPU utilization, and send a notification if CPU usage exceeds 80 percent.</span></span> <span data-ttu-id="d4f7b-245">(CPU 使用率が高い場合、Jenkins サーバー VM をスケールアップした方がよいことを示しています)。また、VM に障害が発生した場合や VM が使用できなくなった場合に、指定されたユーザーに通知することもできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-245">(High CPU usage indicates that you might want to scale up the Jenkins server VM.) You can also notify a designated user if the VM fails or becomes unavailable.</span></span>

## <a name="communities"></a><span data-ttu-id="d4f7b-246">コミュニティ</span><span class="sxs-lookup"><span data-stu-id="d4f7b-246">Communities</span></span>

<span data-ttu-id="d4f7b-247">コミュニティは質問に答え、デプロイを正常に完了できるよう支援します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-247">Communities can answer questions and help you set up a successful deployment.</span></span> <span data-ttu-id="d4f7b-248">以下、具体例に沿って説明します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-248">Consider the following:</span></span>

-   [<span data-ttu-id="d4f7b-249">Jenkins Community Blog</span><span class="sxs-lookup"><span data-stu-id="d4f7b-249">Jenkins Community Blog</span></span>](https://jenkins.io/node/)
-   [<span data-ttu-id="d4f7b-250">Azure フォーラム</span><span class="sxs-lookup"><span data-stu-id="d4f7b-250">Azure Forum</span></span>](https://azure.microsoft.com/support/forums/)
-   [<span data-ttu-id="d4f7b-251">Stack Overflow Jenkins</span><span class="sxs-lookup"><span data-stu-id="d4f7b-251">Stack Overflow Jenkins</span></span>](https://stackoverflow.com/tags/jenkins/info)

<span data-ttu-id="d4f7b-252">Jenkins コミュニティが提供するベスト プラクティスについては、「[Jenkins Best Practices (Jenkins のベスト プラクティス)][jenkins-best]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-252">For more best practices from the Jenkins community, visit [Jenkins best practices][jenkins-best].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d4f7b-253">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="d4f7b-253">Deploy the solution</span></span>

<span data-ttu-id="d4f7b-254">このアーキテクチャをデプロイするには、下記の手順に従って、[Azure 上の Jenkins 用ソリューション テンプレート][azure-market]をインストールし、下記の手順の監視とディザスター リカバリーを設定するスクリプトをインストールします。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-254">To deploy this architecture, follow the steps below to install the [solution template for Jenkins on Azure][azure-market], then install the scripts that set up monitoring and disaster recovery in the steps below.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d4f7b-255">前提条件</span><span class="sxs-lookup"><span data-stu-id="d4f7b-255">Prerequisites</span></span>

- <span data-ttu-id="d4f7b-256">このリファレンス アーキテクチャには、Azure サブスクリプションが必要です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-256">This reference architecture requires an Azure subscription.</span></span> 
- <span data-ttu-id="d4f7b-257">Azure サービス プリンシパルを作成するには、デプロイ済みの Jenkins サーバーに関連付けられている Azure AD テナントの管理者権限が必要です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-257">To create an Azure service principal, you must have admin rights to the Azure AD tenant that is associated with the deployed Jenkins server.</span></span>
- <span data-ttu-id="d4f7b-258">下記の手順は、Jenkins 管理者が、少なくとも共同作成者権限を持つ Azure ユーザーでもあることを前提としています。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-258">These instructions assume that the Jenkins administrator is also an Azure user with at least Contributor privileges.</span></span>

### <a name="step-1-deploy-the-jenkins-server"></a><span data-ttu-id="d4f7b-259">手順 1: Jenkins サーバーをデプロイする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-259">Step 1: Deploy the Jenkins server</span></span>

1.  <span data-ttu-id="d4f7b-260">Web ブラウザーで [Jenkins の Azure Marketplace イメージ][azure-market]を開き、ページの左側の **[今すぐ入手する]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-260">Open the [Azure marketplace image for Jenkins][azure-market] in your web browser and select **GET IT NOW** from the left side of the page.</span></span>

2.  <span data-ttu-id="d4f7b-261">料金の詳細を確認して **[続行]** を選択し、**[作成]** を選択して、Azure Portal で Jenkins サーバーを構成します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-261">Review the pricing details and select **Continue**, then select **Create** to configure the Jenkins server in the Azure portal.</span></span>

<span data-ttu-id="d4f7b-262">詳細については、「[Azure Portal から Azure Linux VM に Jenkins サーバーを作成する][create-jenkins]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-262">For detailed instructions, see [Create a Jenkins server on an Azure Linux VM from the Azure portal][create-jenkins].</span></span> <span data-ttu-id="d4f7b-263">このリファレンス アーキテクチャでは、管理者ログオンでサーバーを起動して実行するだけで十分です。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-263">For this reference architecture, it is sufficient to get the server up and running with the admin logon.</span></span> <span data-ttu-id="d4f7b-264">その後、他のさまざまなサービスを使用するためにサーバーをプロビジョニングできます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-264">Then you can provision it to use various other services.</span></span>

### <a name="step-2-set-up-sso"></a><span data-ttu-id="d4f7b-265">手順 2: SSO を設定する</span><span class="sxs-lookup"><span data-stu-id="d4f7b-265">Step 2: Set up SSO</span></span>

<span data-ttu-id="d4f7b-266">この手順は Jenkins 管理者が実行します。管理者には、サブスクリプションの Azure AD ディレクトリのユーザー アカウントも必要であり、共同作成者ロールが割り当てられている必要があります。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-266">The step is run by the Jenkins administrator, who must also have a user account in the subscription’s Azure AD directory and must be assigned the Contributor role.</span></span>

<span data-ttu-id="d4f7b-267">Jenkins サーバーで Jenkins Update Center の [Azure AD プラグイン][configure-azure-ad]を使用し、指示に従って SSO を設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-267">Use the [Azure AD Plugin][configure-azure-ad] from the Jenkins Update Center in the Jenkin server and follow the instructions to set up SSO.</span></span>

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a><span data-ttu-id="d4f7b-268">手順 3: Azure VM Agents プラグインを使用して Jenkins サーバーをプロビジョニングする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-268">Step 3: Provision Jenkins server with Azure VM Agent plugin</span></span>

<span data-ttu-id="d4f7b-269">この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Azure VM Agents プラグインを設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-269">The step is run by the Jenkins administrator to set up the Azure VM Agent plugin, which is already installed.</span></span>

<span data-ttu-id="d4f7b-270">[こちらの手順に従って、プラグインを構成します][configure-agent]。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-270">[Follow these steps to configure the plugin][configure-agent].</span></span> <span data-ttu-id="d4f7b-271">プラグインのサービス プリンシパルの設定に関するチュートリアルについては、「[Scale your Jenkins deployments to meet demand with Azure VM agents (要求を満たすために Azure VM Agents を使用して Jenkins デプロイをスケールする)][scale-agent]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-271">For a tutorial about setting up service principals for the plugin, see [Scale your  Jenkins deployments to meet demand with Azure VM agents][scale-agent].</span></span>

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a><span data-ttu-id="d4f7b-272">手順 4: Azure Storage を使用して Jenkins サーバーをプロビジョニングする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-272">Step 4: Provision Jenkins server with Azure Storage</span></span>

<span data-ttu-id="d4f7b-273">この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Windows Azure Storage プラグインを設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-273">The step is run by the Jenkins administrator, who sets up the Windows Azure Storage Plugin, which is already installed.</span></span>

<span data-ttu-id="d4f7b-274">[こちらの手順に従って、プラグインを構成します][configure-storage]。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-274">[Follow these steps to configure the plugin][configure-storage].</span></span>

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a><span data-ttu-id="d4f7b-275">手順 5: Azure Credentials プラグインを使用して Jenkins サーバーをプロビジョニングする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-275">Step 5: Provision Jenkins server with Azure Credential plugin</span></span>

<span data-ttu-id="d4f7b-276">この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Azure Credentials プラグインを設定します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-276">The step is run by the Jenkins administrator to set up the Azure Credential plugin, which is already installed.</span></span>

<span data-ttu-id="d4f7b-277">[こちらの手順に従って、プラグインを構成します][configure-credential]。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-277">[Follow these steps to configure the plugin][configure-credential].</span></span>

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a><span data-ttu-id="d4f7b-278">手順 6: Azure Monitor サービスで監視するために Jenkins サーバーをプロビジョニングする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-278">Step 6: Provision Jenkins server for monitoring by the Azure Monitor Service</span></span>

<span data-ttu-id="d4f7b-279">Jenkins サーバーの監視を設定するには、「[Azure Monitor での Azure サービス メトリック アラートの作成][create-metric]」の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-279">To set up monitoring for your Jenkins server, follow the instructions in [Create metric alerts in Azure Monitor for Azure services][create-metric].</span></span>

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a><span data-ttu-id="d4f7b-280">手順 7: ディザスター リカバリー用の管理ディスクを使用して Jenkins サーバーをプロビジョニングする</span><span class="sxs-lookup"><span data-stu-id="d4f7b-280">Step 7: Provision Jenkins server with Managed Disks for disaster recovery</span></span>

<span data-ttu-id="d4f7b-281">Microsoft Jenkins 製品グループは、Jenkins の状態を保存するために使用する管理ディスクを作成するディザスター リカバリー スクリプトを作成しました。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-281">The Microsoft Jenkins product group has created disaster recovery scripts that build a managed disk used to save the Jenkins state.</span></span> <span data-ttu-id="d4f7b-282">サーバーがダウンした場合、最新の状態に復元できます。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-282">If the server goes down, it can be restored to its latest state.</span></span>

<span data-ttu-id="d4f7b-283">[GitHub][disaster] からディザスター リカバリー スクリプトをダウンロードして実行します。</span><span class="sxs-lookup"><span data-stu-id="d4f7b-283">Download and run the disaster recovery scripts from [GitHub][disaster].</span></span>

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 