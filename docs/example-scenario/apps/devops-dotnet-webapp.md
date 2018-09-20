---
title: VSTS を使用した CI/CD パイプライン
description: .NET アプリをビルドして Azure Web Apps にリリースする例
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: aea757087f4a505a8c52658abe1841c5455977cc
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389275"
---
# <a name="cicd-pipeline-with-vsts"></a><span data-ttu-id="e3bf1-103">VSTS を使用した CI/CD パイプライン</span><span class="sxs-lookup"><span data-stu-id="e3bf1-103">CI/CD pipeline with VSTS</span></span>

<span data-ttu-id="e3bf1-104">DevOps とは、開発、品質保証、および IT 運用の統合です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-104">DevOps is the integration of development, quality assurance, and IT operations.</span></span> <span data-ttu-id="e3bf1-105">DevOps には、ソフトウェアを供給するための統合カルチャと一連の強力なプロセスの両方が必要です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-105">DevOps requires both unified culture and a strong set of processes for delivering software.</span></span>

<span data-ttu-id="e3bf1-106">このサンプル シナリオでは、開発チームが Visual Studio Team Services を使用して、.NET 2 層 Web アプリを Azure App Service にデプロイする方法を示します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-106">This example scenario demonstrates how development teams can use Visual Studio Team Services to deploy a .NET two-tier web application to Azure App Service.</span></span> <span data-ttu-id="e3bf1-107">Web アプリケーションは、ダウンストリームにある Azure PaaS (サービスとしてのプラットフォーム) サービスに依存します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-107">The Web Application depends on downstream Azure Platform as a Service (PaaS) services.</span></span> <span data-ttu-id="e3bf1-108">また、このドキュメントでは、Azure PaaS (サービスとしてのプラットフォーム) を使用してこのようなシナリオを設計するときに考慮すべきいくつかの点についても紹介します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-108">This document also points out some considerations that you should make when designing such a scenario using Azure Platform as a Service (PaaS).</span></span>

<span data-ttu-id="e3bf1-109">継続的インテグレーション (CI) と継続的デプロイ (CD) を使用したアプリケーション開発の最新アプローチを採用することで、堅牢なビルド、テスト、デプロイ、および監視サービスを通じて、ユーザーへの価値提供の迅速化に役立てることができます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-109">Adopting a modern approach to application development using Continuous Integration (CI) and Continuous Deployment (CD), helps you to accelerate the delivery of value to your users through a robust build, test, deployment, and monitoring service.</span></span> <span data-ttu-id="e3bf1-110">App Service などの Azure サービスに加え、Visual Studio Team Services などのプラットフォームを使用すると、組織は自身のシナリオを実現するためのインフラストラクチャの管理ではなく、シナリオ自体の開発に集中し続けることができます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-110">By using a platform such as Visual Studio Team Services in addition to Azure services such as App Service, organizations can ensure they remain focused on the development of their scenario, rather than the management of the infrastructure to enable it.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="e3bf1-111">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="e3bf1-111">Related use cases</span></span>

<span data-ttu-id="e3bf1-112">次のユース ケースについて、DevOps を検討してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-112">Consider DevOps for the following use cases:</span></span>

* <span data-ttu-id="e3bf1-113">アプリケーション開発とそのライフサイクルを加速化する</span><span class="sxs-lookup"><span data-stu-id="e3bf1-113">Speeding up application development and development life cycles</span></span>
* <span data-ttu-id="e3bf1-114">自動化されたビルドおよびリリース プロセスで品質と一貫性を確保する</span><span class="sxs-lookup"><span data-stu-id="e3bf1-114">Building quality and consistency into an automated build and release process</span></span>

## <a name="architecture"></a><span data-ttu-id="e3bf1-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="e3bf1-115">Architecture</span></span>

![Visual Studio Team Services および Azure App Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャ概要][architecture]

<span data-ttu-id="e3bf1-117">このシナリオでは、Visual Studio Team Services (VSTS) を使用した .NET Web アプリケーションの DevOps パイプラインに対応できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-117">This scenario covers a DevOps pipeline for a .NET web application using Visual Studio Team Services (VSTS).</span></span> <span data-ttu-id="e3bf1-118">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="e3bf1-119">アプリケーションのソース コードを変更します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-119">Change application source code.</span></span>
2. <span data-ttu-id="e3bf1-120">アプリケーション コードと Web Apps の web.config ファイルをコミットします。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-120">Commit application code and Web Apps web.config file.</span></span>
3. <span data-ttu-id="e3bf1-121">継続的インテグレーションによって、アプリケーションのビルドと単体テストがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-121">Continuous integration triggers application build and unit tests.</span></span>
4. <span data-ttu-id="e3bf1-122">継続的デプロイ トリガーにより、"*環境固有のパラメーター構成値を使用して*" アプリケーション成果物のデプロイが調整されます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-122">Continuous deployment trigger orchestrates deployment of application artifacts *with environment-specific parameterized configuration values*.</span></span>
5. <span data-ttu-id="e3bf1-123">Azure App Service にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-123">Deployment to Azure App Service.</span></span>
6. <span data-ttu-id="e3bf1-124">Azure Application Insights が、正常性、パフォーマンス、使用状況のデータを収集して分析します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-124">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="e3bf1-125">正常性、パフォーマンス、および使用状況の情報を確認します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-125">Review health, performance, and usage information.</span></span>

### <a name="components"></a><span data-ttu-id="e3bf1-126">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="e3bf1-126">Components</span></span>

* <span data-ttu-id="e3bf1-127">[リソース グループ][resource-groups]は Azure リソースの論理コンテナーであり、管理プレーンのアクセス制御の境界も提供します。リソース グループは、"デプロイの単位" を表すと考えてください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-127">[Resource Groups][resource-groups] are a logical container for Azure resources and also provide an access control boundary for the management plane - think of a Resource Group as representing a "unit of deployment".</span></span>
* <span data-ttu-id="e3bf1-128">[Visual Studio Team Services (VSTS)][vsts] は、計画やプロジェクト管理から、コード管理、ビルド、リリースまで、開発ライフサイクルをエンド ツー エンドで管理できるようにするサービスです。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-128">[Visual Studio Team Services (VSTS)][vsts] is a service that enables you to manage your development life cycle end-to-end; from planning and project management, to code management, through to build and release.</span></span>
* <span data-ttu-id="e3bf1-129">[Azure Web Apps][web-apps] は、Web アプリケーション、REST API、およびモバイル バックエンドをホストするための、サービスとしてのプラットフォーム (PaaS) サービスです。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-129">[Azure Web Apps][web-apps] is a Platform as a Service (PaaS) service for hosting web applications, REST APIs, and mobile backends.</span></span> <span data-ttu-id="e3bf1-130">この記事では .NET に焦点を当てていますが、他にも開発プラットフォームのオプションが複数サポートされています。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-130">While this article focuses on .NET, there are several additional development platform options supported.</span></span>
* <span data-ttu-id="e3bf1-131">[Application Insights][application-insights] は、複数のプラットフォームで使用できるファーストパーティの拡張可能な Web 開発者向けアプリケーション パフォーマンス管理 (APM) サービスです。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-131">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternative-devops-tooling-options"></a><span data-ttu-id="e3bf1-132">代替の DevOps ツール オプション</span><span class="sxs-lookup"><span data-stu-id="e3bf1-132">Alternative DevOps tooling options</span></span>

<span data-ttu-id="e3bf1-133">この記事では Visual Studio Team Services に焦点を当てていますが、オンプレミスでは [Team Foundation Server][team-foundation-server] を代わりに使用できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-133">Whilst this article focuses on Visual Studio Team Services, [Team Foundation Server][team-foundation-server] could be used as on premises substitute.</span></span> <span data-ttu-id="e3bf1-134">また、[Jenkins][jenkins-on-azure] を利用して、オープン ソース開発パイプライン向けに一連のテクノロジ コレクションを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-134">Alternatively, you may also find a collection of technologies being used together for an Open Source development pipeline leveraging [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="e3bf1-135">コードとしてのインフラストラクチャの観点からは、[Azure Resource Manager テンプレート][arm-templates]が Azure DevOps プロジェクトの一部として含まれていますが、[Terraform][terraform] や [Chef][chef] に投資済みの場合はここで検討することもできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-135">From an Infrastructure as Code perspective, [Azure Resource Manager Templates][arm-templates] are included as part of the Azure DevOps project, but you could consider [Terraform][terraform] or [Chef][chef] if you have investments here.</span></span> <span data-ttu-id="e3bf1-136">サービスとしてのインフラストラクチャ (IaaS) ベースのデプロイを使用するときに、構成管理が必要な場合は、[Azure Automation State Configuration][desired-state-configuration]、[Ansible][ansible]、または [Chef][chef] を検討できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-136">If you prefer an Infrastructure as a Service (IaaS) based deployment and require configuration management, then you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible] or [Chef][chef].</span></span>

### <a name="alternatives-to-web-app-hosting"></a><span data-ttu-id="e3bf1-137">Web アプリ ホスティングに代わる方法</span><span class="sxs-lookup"><span data-stu-id="e3bf1-137">Alternatives to Web App Hosting</span></span>

<span data-ttu-id="e3bf1-138">Azure Web Apps でのホスティングに代わる方法:</span><span class="sxs-lookup"><span data-stu-id="e3bf1-138">Alternatives to hosting in Azure Web Apps:</span></span>

* <span data-ttu-id="e3bf1-139">[VM][compare-vm-hosting] - 細かな制御が必要なワークロード、または Web Apps では使用できない OS コンポーネント/サービス (Windows GAC、COM など) に依存するワークロードの場合</span><span class="sxs-lookup"><span data-stu-id="e3bf1-139">[VM][compare-vm-hosting] - For workloads that require a high degree of control, or depend on OS components / services that are not possible with Web Apps (for example, the Windows GAC, or COM)</span></span>
* <span data-ttu-id="e3bf1-140">[コンテナーのホスティング][azure-containers] - OS の依存関係がある場合は、ホスティングの移植性またはホスティング密度も要件となります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-140">[Container Hosting][azure-containers] - Where there are OS dependencies and hosting portability, or hosting density, are also requirements.</span></span>
* <span data-ttu-id="e3bf1-141">[Service Fabric][service-fabric] - ワークロード アーキテクチャが、クラスター全体で細かく制御しながらデプロイおよび実行できることでメリットを得られる分散コンポーネントであることを重視している場合に適したオプションです。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-141">[Service Fabric][service-fabric] - A good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="e3bf1-142">Service Fabric を使ってコンテナーをホストすることもできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-142">Service Fabric can also be used to host containers.</span></span>
* <span data-ttu-id="e3bf1-143">[サーバーレス Azure Functions][azure-functions] - 依存関係が最小限で済む細かな分散コンポーネントであることを重視したワークロード アーキテクチャに適したオプションです。個々のコンポーネントは (継続的ではなく) オンデマンドで実行するためにのみ必要で、コンポーネントのオーケストレーションが不要な場合です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-143">[Serverless Azure functions][azure-functions] - A good option if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

### <a name="devops"></a><span data-ttu-id="e3bf1-144">DevOps</span><span class="sxs-lookup"><span data-stu-id="e3bf1-144">DevOps</span></span>

<span data-ttu-id="e3bf1-145">**[継続的インテグレーション (CI)][continuous-integration]** は、複数の個人開発者またはチームが共有コードベースに対して小さな変更を頻繁かつ継続的にコミットする状況において、安定したビルドを実証することを目標としています。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-145">**[Continuous Integration (CI)][continuous-integration]** should aim to demonstrate a stable build, with more than one individual developer or team continuously committing small, frequent changes to the shared codebase.</span></span>
<span data-ttu-id="e3bf1-146">継続的インテグレーション パイプラインの一環として、次を行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-146">As part of your Continuous Integration pipeline you should;</span></span>

* <span data-ttu-id="e3bf1-147">少量のコードを頻繁にチェックインします (複雑で大きな変更を一括で行うと、適切にマージするのが難しくなるため避けてください)</span><span class="sxs-lookup"><span data-stu-id="e3bf1-147">Check in small amounts of code frequently (avoid batching up larger or more complex changes as these can be harder to merge successfully)</span></span>
* <span data-ttu-id="e3bf1-148">十分なコード カバレッジ (アンハッピー パスを含む) でご自身のアプリケーションのコンポーネントを単体テストします</span><span class="sxs-lookup"><span data-stu-id="e3bf1-148">Unit Test the components of your application with sufficient code coverage (including the unhappy paths)</span></span>
* <span data-ttu-id="e3bf1-149">ビルドは、必ず共有のマスター (またはトランク) ブランチに対して実行します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-149">Ensuring the build is run against the shared, master (or trunk) branch.</span></span> <span data-ttu-id="e3bf1-150">このブランチは安定性があり、"デプロイ準備完了" として維持されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-150">This branch should be stable and maintained as "deployment ready".</span></span> <span data-ttu-id="e3bf1-151">不完全な変更または処理中の変更については、競合を回避するために、頻繁な "前方統合" マージによって個別のブランチに分離する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-151">Incomplete or work-in-progress changes should be isolated in a separate branch with frequent 'forward integration' merges to avoid conflicts later.</span></span>

<span data-ttu-id="e3bf1-152">**[継続的デリバリー (CD)][continuous-delivery]** は、安定したビルドだけでなく、安定したデプロイを実証することを目標としています。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-152">**[Continuous Delivery (CD)][continuous-delivery]** should aim to demonstrate not only a stable build but a stable deployment.</span></span> <span data-ttu-id="e3bf1-153">これにより CD の実現が少し困難になるため、環境固有の構成と、その値を正しく設定するメカニズムが必要になります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-153">This makes realizing CD a little more difficult, environment-specific configuration is required and a mechanism for setting those values correctly.</span></span>

<span data-ttu-id="e3bf1-154">また、さまざまなコンポーネントが適切に構成され、エンド ツー エンドで確実に動作するように、十分なカバレッジの統合テストも必要です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-154">In addition, sufficient coverage of Integration Testing is required to ensure that the various components are configured and working correctly end-to-end.</span></span>

<span data-ttu-id="e3bf1-155">このため、環境固有のデータの設定とリセット、およびデータベース スキーマ バージョンの管理が必要になる可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-155">This may also require setting up and resetting environment-specific data and managing database schema versions.</span></span>

<span data-ttu-id="e3bf1-156">継続的デリバリーをさらにロード テストやユーザー受け入れテストの環境に拡張することもできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-156">Continuous Delivery may also extend to Load Testing and User Acceptance Testing Environments.</span></span>

<span data-ttu-id="e3bf1-157">継続的デリバリーでは、すべての環境で継続的監視のメリットを得られることが理想です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-157">Continuous Delivery benefits from continuous Monitoring, ideally across all environments.</span></span>
<span data-ttu-id="e3bf1-158">環境全体におけるデプロイおよび統合テストの一貫性と信頼性は、作成/構成またはホスティング インフラストラクチャのスクリプトを作成することで、さらに容易に実現します (クラウドベースのワークロードでは大幅に容易になるもの。Azure のコードとしてのインフラストラクチャを参照)。これは、"[コードとしてのインフラストラクチャ][infra-as-code]" とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-158">The consistency and reliability of deployments and integration testing across environments is made easier by scripting the creation and configuration or the hosting infrastructure (something that is considerably easier for Cloud-based workloads, see Azure infrastructure as code) - this is also known as ["infrastructure-as-code"][infra-as-code].</span></span>

* <span data-ttu-id="e3bf1-159">継続的デリバリーは、プロジェクトのライフサイクルでできるだけ早く開始します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-159">Start Continuous Delivery as early as possible in the project life-cycle.</span></span> <span data-ttu-id="e3bf1-160">これは後になるほど難しくなります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-160">The later you leave it, the more difficult it will be.</span></span>
* <span data-ttu-id="e3bf1-161">統合および単体テストの優先度は、プロジェクトの機能と同じにします</span><span class="sxs-lookup"><span data-stu-id="e3bf1-161">Integration & unit tests should be given the same priority as the project features</span></span>
* <span data-ttu-id="e3bf1-162">環境に依存しないデプロイ パッケージを使用し、リリース プロセスを通じて環境固有の構成を管理します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-162">Use environment agnostic deployment packages and manage environment-specific configuration through the release process.</span></span>
* <span data-ttu-id="e3bf1-163">リリース プロセス中は、リリース管理ツールを使用するか、ハードウェア セキュリティ モジュール (HSM)、または [Key Vault][azure-key-vault] を呼び出して、機密を要する構成を保護します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-163">Protect sensitive configuration within the release management tooling or by calling out to a Hardware-security-module (HSM), or [Key Vault][azure-key-vault], during the release process.</span></span> <span data-ttu-id="e3bf1-164">ソース管理内には機密を要する構成を格納しないでください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-164">Do not store sensitive configuration within source control.</span></span>

<span data-ttu-id="e3bf1-165">**継続的な学習** - CD 環境を最も効果的に監視するには、Microsoft の [Application Insights][application-insights] などのアプリケーション パフォーマンス監視ツール (略して APM) を使用します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-165">**Continuous Learning** - The most effective monitoring of a CD environment is provided by Application-Performance-Monitoring tools (APM for short), for example Microsoft's [Application Insights][application-insights].</span></span> <span data-ttu-id="e3bf1-166">アプリケーション ワークロードを十分に監視することは、バグや、負荷の下でのパフォーマンスを把握するうえで重要です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-166">Sufficient depth of monitoring for an application workload is critical to understand bugs, performance under load.</span></span> <span data-ttu-id="e3bf1-167">[App Insights を VSTS に統合して、CD パイプラインの継続的な監視を有効にできます][app-insights-cd-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-167">[App Insights can be integrated into VSTS to enable continuous monitoring of the CD pipeline][app-insights-cd-monitoring].</span></span> <span data-ttu-id="e3bf1-168">これを使用すると、ユーザーの介入なしで、次のステージに自動的に進むことができます。また、アラートが検出された場合も自動的にロールバックできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-168">This could be used to enable automatic progression to the next stage, without human intervention, or rollback if an alert is detected.</span></span>

## <a name="considerations"></a><span data-ttu-id="e3bf1-169">考慮事項</span><span class="sxs-lookup"><span data-stu-id="e3bf1-169">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="e3bf1-170">可用性</span><span class="sxs-lookup"><span data-stu-id="e3bf1-170">Availability</span></span>

<span data-ttu-id="e3bf1-171">クラウド アプリケーションを構築するときは、[可用性のための標準的な設計パターン][design-patterns-availability]の利用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-171">Consider leveraging the [typical design patterns for availability][design-patterns-availability] when building your cloud application.</span></span>

<span data-ttu-id="e3bf1-172">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]で可用性に関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="e3bf1-172">Review the availability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="e3bf1-173">可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-173">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="e3bf1-174">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="e3bf1-174">Scalability</span></span>

<span data-ttu-id="e3bf1-175">クラウド アプリケーションを構築するときは、[スケーラビリティのための標準的な設計パターン][design-patterns-scalability]に注意してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-175">When building a cloud application be aware of the [typical design patterns for scalability][design-patterns-scalability].</span></span>

<span data-ttu-id="e3bf1-176">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でスケーラビリティに関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="e3bf1-176">Review the scalability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="e3bf1-177">スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-177">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="e3bf1-178">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="e3bf1-178">Security</span></span>

<span data-ttu-id="e3bf1-179">必要に応じて、[セキュリティのための標準的な設計パターン][design-patterns-security]を利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-179">Consider leveraging the [typical design patterns for security][design-patterns-security] where appropriate.</span></span>

<span data-ttu-id="e3bf1-180">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でセキュリティに関する考慮事項を確認します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-180">Review the security considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture].</span></span>

<span data-ttu-id="e3bf1-181">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-181">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="e3bf1-182">回復性</span><span class="sxs-lookup"><span data-stu-id="e3bf1-182">Resiliency</span></span>

<span data-ttu-id="e3bf1-183">[回復性のための標準的な設計パターン][design-patterns-resiliency]を確認し、必要に応じて、これらを実装することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-183">Review the [typical design patterns for resiliency][design-patterns-resiliency] and consider implementing these where appropriate.</span></span>

<span data-ttu-id="e3bf1-184">Azure アーキテクチャ センターでは、多数の [App Service に関する推奨プラクティス][resiliency-app-service]を確認できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-184">You can find a number of [recommended practices for App Service][resiliency-app-service] in the Azure Architecture Center.</span></span>

<span data-ttu-id="e3bf1-185">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-185">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="e3bf1-186">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="e3bf1-186">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e3bf1-187">前提条件</span><span class="sxs-lookup"><span data-stu-id="e3bf1-187">Prerequisites</span></span>

* <span data-ttu-id="e3bf1-188">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-188">You must have an existing Azure account.</span></span> <span data-ttu-id="e3bf1-189">Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][azure-free-account]を作成してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-189">If you don't have an Azure subscription, create a [free account][azure-free-account] before you begin.</span></span>
* <span data-ttu-id="e3bf1-190">既存の Visual Studio Team Services (VSTS) アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-190">You must have an existing Visual Studio Team Services (VSTS) account.</span></span> <span data-ttu-id="e3bf1-191">詳細については、[Visual Studio Team Services (VSTS) アカウントの作成][vsts-account-create]に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-191">Find out more details about [creating a Visual Studio Team Services (VSTS) account][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="e3bf1-192">チュートリアル</span><span class="sxs-lookup"><span data-stu-id="e3bf1-192">Walk through</span></span>

<span data-ttu-id="e3bf1-193">このシナリオでは、Azure DevOps プロジェクトを使用して、ご自身の CI/CD パイプラインを作成します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-193">In this scenario, you'll use the Azure DevOps Project to create your CI/CD pipeline.</span></span>

<span data-ttu-id="e3bf1-194">DevOps プロジェクトでは、App Service プラン、App Service と App Insights のリソースが自動的にデプロイされ、Visual Studio Team Services プロジェクトが自動的に構成されます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-194">The DevOps project will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Visual Studio Team Services Project for you.</span></span>

<span data-ttu-id="e3bf1-195">DevOps プロジェクトの開発とビルドが完了したら、関連するコード変更、作業項目、およびテスト結果を確認します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-195">Once you've deployed the DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="e3bf1-196">コードには実行するテストが含まれないため、テスト結果は表示されません。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-196">You will notice no test results are displayed, as the code does not contain any tests to run.</span></span>

<span data-ttu-id="e3bf1-197">リリース定義を確認します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-197">Review the Release definitions.</span></span> <span data-ttu-id="e3bf1-198">リリース パイプラインが設定され、アプリケーションが開発環境にリリースされていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-198">Notice that a release pipeline has been setup, releasing our application into Dev.</span></span> <span data-ttu-id="e3bf1-199">開発環境への自動リリースを含む**ドロップ** ビルド成果物から**継続的配置トリガー**が設定されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-199">Observe that there is a **continuous deployment trigger** set from the **Drop** build artifact, with automatic releases into the Dev environments.</span></span> <span data-ttu-id="e3bf1-200">継続的配置プロセスの一環として、リリースが複数の環境にまたがっていることを確認できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-200">As part of a Continuous Deployment process, you may see releases span across multiple environments.</span></span> <span data-ttu-id="e3bf1-201">リリースは両方のインフラストラクチャにまたがることができます (コードとしてのインフラストラクチャなどの手法を使用)。また、このリリースにより、必要なアプリケーション パッケージと、構成後のタスクもデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-201">A release can span both infrastructure (using techniques such as Infrastructure as Code), and also deploy the application packages required as well as any post-configuration tasks.</span></span>

<span data-ttu-id="e3bf1-202">**追加の考慮事項。**</span><span class="sxs-lookup"><span data-stu-id="e3bf1-202">**Additional Considerations.**</span></span>

* <span data-ttu-id="e3bf1-203">VSTS マーケットプレースで入手できる[トークン化タスク][vsts-tokenization]のいずれかを利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-203">Consider leveraging one of the [tokenization tasks][vsts-tokenization] that are available in the VSTS marketplace.</span></span>
* <span data-ttu-id="e3bf1-204">[Azure Key Vault のデプロイ][download-keyvault-secrets] VSTS タスクを使用して、Azure Key Vault からご自身のリリースにシークレットをダウンロードすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-204">Consider using the [Deploy: Azure Key Vault][download-keyvault-secrets] VSTS task to download secrets from an Azure KeyVault into your release.</span></span> <span data-ttu-id="e3bf1-205">その後、変数としてのこれらのシークレットを、リリース定義の一部に使用します。これはソース管理には格納しないでください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-205">You can then use those secrets as variables as part of your release definition, and should not be storing them in source control.</span></span>
* <span data-ttu-id="e3bf1-206">お使いの環境の構成変更を促進するために、ご自身のリリース定義で[リリース変数][vsts-release-variables]を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-206">Consider using [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="e3bf1-207">リリース変数は、リリース全体または特定の環境を対象にできます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-207">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="e3bf1-208">シークレット情報に変数を使用している場合は、必ず南京錠アイコンを選択します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-208">If using variables for secret information, ensure that you select the padlock icon.</span></span>
* <span data-ttu-id="e3bf1-209">リリース パイプラインで[デプロイ ゲート][vsts-deployment-gates]を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-209">Consider using [deployment gates][vsts-deployment-gates] in your release pipeline.</span></span> <span data-ttu-id="e3bf1-210">これにより、外部システム (インシデント管理システム、追加の特注システムなど) と共に監視データを利用して、リリースを昇格させる必要があるかどうかを判断できます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-210">This allows you to leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>
* <span data-ttu-id="e3bf1-211">リリース パイプラインで手動介入が必要な場合は、[承認][vsts-approvals]機能を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-211">Where manual intervention in a release pipeline is required, consider using the [approvals][vsts-approvals] functionality.</span></span>
* <span data-ttu-id="e3bf1-212">ご自身のリリース パイプラインで [Application Insights][application-insights] と追加の監視ツールをできるだけ早く使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-212">Consider using [Application Insights][application-insights] and additional monitoring tooling as early as possible in your release pipeline.</span></span> <span data-ttu-id="e3bf1-213">ほとんどの組織は、運用環境で監視を開始するだけで、プロセスの早い段階で潜在的なバグを特定し、運用環境のユーザーに影響を及ぶのを避けることができます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-213">Most organizations only begin monitoring in their production environment, though you could identify potential bugs earlier in the process and prevent impact to your users in production.</span></span>

## <a name="pricing"></a><span data-ttu-id="e3bf1-214">価格</span><span class="sxs-lookup"><span data-stu-id="e3bf1-214">Pricing</span></span>

<span data-ttu-id="e3bf1-215">ご自身の Visual Studio Team Services のコストは、アクセスを必要とするご自身の組織のユーザー数のほか、必要な同時実行ビルド/リリース数、テスト ユーザー数などの要因によっても異なります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-215">Your Visual Studio Team Services costing will depend upon the number of users in your organization that require access, in addition to factors such as the number of concurrent build/releases required, and number of test users.</span></span> <span data-ttu-id="e3bf1-216">詳細については、[VSTS の価格に関するページ][vsts-pricing-page]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-216">These are detailed further on the [VSTS pricing page][vsts-pricing-page].</span></span>

* <span data-ttu-id="e3bf1-217">[Visual Studio Team Services (VSTS)][vsts-pricing-calculator] は、開発ライフサイクルを管理できるようにするサービスで、ユーザーあたりの料金が月単位で請求されます。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-217">[Visual Studio Team Services (VSTS)][vsts-pricing-calculator] is a service that enables you to manage your development life cycle and is paid for on a per user, per month basis.</span></span> <span data-ttu-id="e3bf1-218">必要な同時実行パイプライン、追加のテスト ユーザー、またはユーザーの基本ライセンスによっては、追加料金が発生する場合があります。</span><span class="sxs-lookup"><span data-stu-id="e3bf1-218">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users, or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="e3bf1-219">関連リソース</span><span class="sxs-lookup"><span data-stu-id="e3bf1-219">Related Resources</span></span>

* <span data-ttu-id="e3bf1-220">[DevOps とは][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="e3bf1-220">[What is DevOps?][devops-whatis]</span></span>
* <span data-ttu-id="e3bf1-221">[Microsoft での DevOps - Visual Studio Team Services の使用方法][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="e3bf1-221">[DevOps at Microsoft - How we work with Visual Studio Team Services][devops-microsoft]</span></span>
* <span data-ttu-id="e3bf1-222">[ステップ バイ ステップのチュートリアル: Visual Studio Team Services を使用した DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="e3bf1-222">[Step-by-step Tutorials: DevOps with Visual Studio Team Services][devops-with-vsts]</span></span>
* <span data-ttu-id="e3bf1-223">[Azure DevOps プロジェクトを使用して .NET 用 CI/CD パイプラインを作成する][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="e3bf1-223">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/