---
title: Azure DevOps を使用した CI/CD パイプラインの設計
titleSuffix: Azure Example Scenarios
description: Azure DevOps を使用して .NET アプリを構築し、Azure Web Apps にリリースします。
author: christianreddington
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, seodec18
ms.openlocfilehash: 8934200aca8e4055596dd6dc27ede2f0a4d03f23
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483362"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="8497f-103">Azure DevOps を使用した CI/CD パイプラインの設計</span><span class="sxs-lookup"><span data-stu-id="8497f-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="8497f-104">このシナリオでは、継続的インテグレーション (CI) と継続的配置 (CD) パイプラインを構築するためのアーキテクチャと設計のガイダンスを示します。</span><span class="sxs-lookup"><span data-stu-id="8497f-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span> <span data-ttu-id="8497f-105">この例の CI/CD パイプラインは、Azure App Service に 2 層の .NET Web アプリケーションをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8497f-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="8497f-106">最新の CI/CD プロセスへの移行には、アプリケーションのビルド、デプロイ、テスト、および監視の点で多くのメリットがあります。</span><span class="sxs-lookup"><span data-stu-id="8497f-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="8497f-107">Azure DevOps を App Service などの他のサービスと共に使用することで、組織では、自身のシナリオを実現するためのサポート インフラストラクチャの管理ではなく、アプリ自体の開発に集中することができます。</span><span class="sxs-lookup"><span data-stu-id="8497f-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="8497f-108">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="8497f-108">Relevant use cases</span></span>

<span data-ttu-id="8497f-109">次のユースケースについて、Azure DevOps および CI/CD プロセスを検討してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="8497f-110">アプリケーション開発とそのライフサイクルを加速する</span><span class="sxs-lookup"><span data-stu-id="8497f-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="8497f-111">自動化されたビルドおよびリリース プロセスで品質と一貫性を確保する</span><span class="sxs-lookup"><span data-stu-id="8497f-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="8497f-112">アプリケーションの安定性と稼働時間を増やす</span><span class="sxs-lookup"><span data-stu-id="8497f-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="8497f-113">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="8497f-113">Architecture</span></span>

![Azure DevOps および Azure App Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャ ダイアグラム][architecture]

<span data-ttu-id="8497f-115">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="8497f-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="8497f-116">開発者がアプリケーションのソース コードを変更します。</span><span class="sxs-lookup"><span data-stu-id="8497f-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="8497f-117">web.config ファイルを含むアプリケーション コードが、Azure Repos のソース コード リポジトリにコミットされます。</span><span class="sxs-lookup"><span data-stu-id="8497f-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="8497f-118">継続的インテグレーションによって、Azure Test Plans を使用してアプリケーションのビルドと単体テストがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="8497f-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="8497f-119">Azure Pipelines 内の継続的配置により、"*環境固有の構成値を使用して*" アプリケーション成果物の自動化されたデプロイがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="8497f-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="8497f-120">成果物が Azure App Service にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="8497f-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="8497f-121">Azure Application Insights が、正常性、パフォーマンス、使用状況のデータを収集して分析します。</span><span class="sxs-lookup"><span data-stu-id="8497f-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="8497f-122">開発者が、正常性、パフォーマンス、および使用状況の情報を監視および管理します。</span><span class="sxs-lookup"><span data-stu-id="8497f-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="8497f-123">バックログ情報は、Azure Boards を使用した新機能とバグ修正の優先順位付けに使用されます。</span><span class="sxs-lookup"><span data-stu-id="8497f-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="8497f-124">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="8497f-124">Components</span></span>

- <span data-ttu-id="8497f-125">[Azure DevOps][vsts] は、計画やプロジェクト管理から、コード管理、ビルド、リリースまで、開発ライフサイクルをエンド ツー エンドで管理するためのサービスです。</span><span class="sxs-lookup"><span data-stu-id="8497f-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="8497f-126">[Azure Web Apps][web-apps] は、Web アプリケーション、REST API、およびモバイル バックエンドをホストするための PaaS サービスです。</span><span class="sxs-lookup"><span data-stu-id="8497f-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="8497f-127">この記事では .NET に焦点を当てていますが、他にも開発プラットフォームのオプションが複数サポートされています。</span><span class="sxs-lookup"><span data-stu-id="8497f-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="8497f-128">[Application Insights][application-insights] は、複数のプラットフォームで使用できるファーストパーティの拡張可能な Web 開発者向けアプリケーション パフォーマンス管理 (APM) サービスです。</span><span class="sxs-lookup"><span data-stu-id="8497f-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="8497f-129">代替手段</span><span class="sxs-lookup"><span data-stu-id="8497f-129">Alternatives</span></span>

<span data-ttu-id="8497f-130">この記事では Azure DevOps に焦点を当てていますが、オンプレミスでは [Azure DevOps Server][azure-devops-server] (旧称 Team Foundation Server) を代わりに使用できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="8497f-131">または、[Jenkins][jenkins-on-azure] を使用するオープンソース開発パイプライン用の一連のテクノロジを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="8497f-132">コードとしてのインフラストラクチャの観点からは、[Resource Manager テンプレート][arm-templates]が Azure DevOps プロジェクトの一部として使用されていますが、[Terraform][terraform] や [Chef][chef] などの他の管理テクノロジを検討することもできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="8497f-133">サービスとしてのインフラストラクチャ (IaaS) ベースのデプロイを使用するときに、構成管理が必要な場合は、[Azure Automation State Configuration][desired-state-configuration]、[Ansible][ansible]、または [Chef][chef] を検討できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="8497f-134">Azure Web Apps でのホスティングに代わる方法として以下を検討できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="8497f-135">[Azure Virtual Machines][compare-vm-hosting] は、細かな制御が必要なワークロード、または Web Apps では使用できない OS コンポーネントおよびサービス (Windows GAC、COM など) に依存するワークロードを処理します。</span><span class="sxs-lookup"><span data-stu-id="8497f-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="8497f-136">[Service Fabric][service-fabric] は、ワークロード アーキテクチャが、クラスター全体で細かく制御しながらデプロイおよび実行できることでメリットを得られる分散コンポーネントであることを重視している場合に適したオプションです。</span><span class="sxs-lookup"><span data-stu-id="8497f-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="8497f-137">Service Fabric を使ってコンテナーをホストすることもできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="8497f-138">[Azure Functions][azure-functions] は、依存関係が最小限で済む細かな分散コンポーネントであることを重視したワークロード アーキテクチャに適した効果的なサーバーレス アプローチです。個々のコンポーネントは (継続的ではなく) オンデマンドで実行するためにのみ必要で、コンポーネントのオーケストレーションが不要な場合です。</span><span class="sxs-lookup"><span data-stu-id="8497f-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="8497f-139">移行の適切なパスを選択するときは、この [Azure コンピューティング サービスのデシジョン ツリー](/azure/architecture/guide/technology-choices/compute-decision-tree)が役に立つ場合があります。</span><span class="sxs-lookup"><span data-stu-id="8497f-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="8497f-140">管理とセキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="8497f-140">Management and Security Considerations</span></span>

- <span data-ttu-id="8497f-141">VSTS マーケットプレースで入手できる[トークン化タスク][vsts-tokenization]のいずれかを利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="8497f-142">[Azure Key Vault][download-keyvault-secrets] タスクでは、Azure Key Vault からリリースにシークレットをダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="8497f-143">その後、リリース定義で変数としてこれらのシークレットを使用でき、ソース管理に格納しなくてもかまいません。</span><span class="sxs-lookup"><span data-stu-id="8497f-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="8497f-144">お使いの環境の構成変更を促進するために、ご自身のリリース定義で[リリース変数][vsts-release-variables]を使用してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="8497f-145">リリース変数は、リリース全体または特定の環境を対象にできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="8497f-146">シークレット情報に変数を使用している場合は、必ず南京錠アイコンを選択します。</span><span class="sxs-lookup"><span data-stu-id="8497f-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="8497f-147">リリース パイプラインでは[デプロイ ゲート][vsts-deployment-gates]を使用してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="8497f-148">これにより、外部システム (インシデント管理システム、追加の特注システムなど) と共に監視データを利用して、リリースを昇格させる必要があるかどうかを判断できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="8497f-149">リリース パイプラインで手動介入が必要な場合は、[承認][vsts-approvals]機能を使用してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="8497f-150">リリース パイプラインで [Application Insights][application-insights] と追加の監視ツールをできるだけ早く使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="8497f-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="8497f-151">多くの組織は、運用環境でしか監視を開始しません。</span><span class="sxs-lookup"><span data-stu-id="8497f-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="8497f-152">他の環境を監視することにより、開発プロセスの早い段階でバグを発見し、運用環境で問題が発生するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="8497f-153">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="8497f-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="8497f-154">前提条件</span><span class="sxs-lookup"><span data-stu-id="8497f-154">Prerequisites</span></span>

- <span data-ttu-id="8497f-155">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="8497f-155">You must have an existing Azure account.</span></span> <span data-ttu-id="8497f-156">Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-156">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="8497f-157">Azure DevOps 組織にサインアップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="8497f-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="8497f-158">詳細については、[クイック スタートの組織の作成][vsts-account-create]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="8497f-159">チュートリアル</span><span class="sxs-lookup"><span data-stu-id="8497f-159">Walk-through</span></span>

<span data-ttu-id="8497f-160">[Azure DevOps](/azure/devops-project/azure-devops-project-github) プロジェクトでは、App Service プラン、App Service と App Insights のリソースが自動的にデプロイされ、Azure DevOps プロジェクトが自動的に構成されます。</span><span class="sxs-lookup"><span data-stu-id="8497f-160">The [Azure DevOps project](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="8497f-161">Azure DevOps プロジェクトの開発とビルドが完了したら、関連するコード変更、作業項目、およびテスト結果を確認します。</span><span class="sxs-lookup"><span data-stu-id="8497f-161">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="8497f-162">コードには実行するテストが含まれないため、テスト結果は表示されません。</span><span class="sxs-lookup"><span data-stu-id="8497f-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="8497f-163">このプロジェクトでは、リリース パイプラインと継続的配置トリガーを作成して、アプリケーションを開発環境にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8497f-163">The project creates a release pipeline and continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="8497f-164">継続的配置プロセスの一環として、リリースが複数の環境にまたがっていることを確認できます。</span><span class="sxs-lookup"><span data-stu-id="8497f-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="8497f-165">リリースは両方のインフラストラクチャにまたがることができます (コードとしてのインフラストラクチャなどの手法を使用)。また、このリリースにより、必要なアプリケーション パッケージと、構成後のタスクもデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="8497f-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="8497f-166">価格</span><span class="sxs-lookup"><span data-stu-id="8497f-166">Pricing</span></span>

<span data-ttu-id="8497f-167">Azure DevOps のコストは、アクセスを必要とする組織内のユーザーの数だけでなく、必要な同時実行ビルド/リリースの数やテスト ユーザーの数など、他の要因にも依存します。</span><span class="sxs-lookup"><span data-stu-id="8497f-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="8497f-168">詳しくは、「[Azure DevOps の料金][vsts-pricing-page]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="8497f-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="8497f-169">この[料金計算ツール][vsts-pricing-calculator]は、Azure DevOps を 20 ユーザーで実行する場合の見積もりを提供します。</span><span class="sxs-lookup"><span data-stu-id="8497f-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="8497f-170">Azure DevOps はユーザー単位で 1 か月ごとに課金されます。</span><span class="sxs-lookup"><span data-stu-id="8497f-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="8497f-171">必要な同時実行パイプライン、追加のテスト ユーザー、またはユーザーの基本ライセンスによっては、追加料金が発生する場合があります。</span><span class="sxs-lookup"><span data-stu-id="8497f-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="8497f-172">関連リソース</span><span class="sxs-lookup"><span data-stu-id="8497f-172">Related resources</span></span>

<span data-ttu-id="8497f-173">CI/CD および Azure DevOps の詳細については、次のリソースを参照してください。</span><span class="sxs-lookup"><span data-stu-id="8497f-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="8497f-174">[DevOps とは][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="8497f-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="8497f-175">[Microsoft での DevOps - Azure DevOps をどのように使用しているか][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="8497f-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="8497f-176">[ステップバイステップのチュートリアル:Azure DevOps を使用した DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="8497f-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="8497f-177">[DevOps チェックリスト][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="8497f-177">[Devops Checklist][devops-checklist]</span></span>
- <span data-ttu-id="8497f-178">[Azure DevOps プロジェクトを使用して .NET 用 CI/CD パイプラインを作成する][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="8497f-178">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
