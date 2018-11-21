---
title: Azure DevOps を使用した CI/CD パイプライン
description: Azure DevOps を使用して .NET アプリを構築し、Azure Web Apps にリリースします。
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 97f16b2d3d9c15bc6f5db6fad4c9d8097243ad3d
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610789"
---
# <a name="cicd-pipeline-with-azure-devops"></a><span data-ttu-id="bdd29-103">Azure DevOps を使用した CI/CD パイプライン</span><span class="sxs-lookup"><span data-stu-id="bdd29-103">CI/CD pipeline with Azure DevOps</span></span>

<span data-ttu-id="bdd29-104">DevOps とは、開発、品質保証、および IT 運用の統合です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-104">DevOps is the integration of development, quality assurance, and IT operations.</span></span> <span data-ttu-id="bdd29-105">DevOps には、ソフトウェアを供給するための統合カルチャと一連の強力なプロセスの両方が必要です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-105">DevOps requires both unified culture and a strong set of processes for delivering software.</span></span>

<span data-ttu-id="bdd29-106">このシナリオ例では、開発チームが Azure DevOps を使用して、.NET 2 層 Web アプリを Azure App Service にデプロイする方法を示します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-106">This example scenario demonstrates how development teams can use Azure DevOps to deploy a .NET two-tier web application to Azure App Service.</span></span> <span data-ttu-id="bdd29-107">Web アプリケーションは、ダウンストリームにある Azure PaaS (サービスとしてのプラットフォーム) サービスに依存します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-107">The Web Application depends on downstream Azure platform as a service (PaaS) services.</span></span> <span data-ttu-id="bdd29-108">また、このドキュメントでは、Azure PaaS を使用してこのようなシナリオを設計するときに考慮すべきいくつかの点についても紹介します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-108">This document also points out some considerations that you should make when designing such a scenario using Azure PaaS.</span></span>

<span data-ttu-id="bdd29-109">継続的インテグレーションと継続的配置 (CI/CD) を使用したアプリケーション開発の最新アプローチを採用することで、堅牢なビルド、テスト、デプロイ、および監視サービスを通じて、ユーザーへの価値提供の迅速化に役立てることができます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-109">Adopting a modern approach to application development using Continuous Integration and Continuous Deployment (CI/CD), helps you to accelerate the delivery of value to your users through a robust build, test, deployment, and monitoring service.</span></span> <span data-ttu-id="bdd29-110">App Service などの Azure サービスと共に Azure DevOps などのプラットフォームを使用すると、組織は自身のシナリオを実現するためのサポート インフラストラクチャの管理ではなく、シナリオ自体の開発に集中することができます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-110">By using a platform such as Azure DevOps along with Azure services such as App Service, organizations can focus on the development of their scenario rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="bdd29-111">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="bdd29-111">Relevant use cases</span></span>

<span data-ttu-id="bdd29-112">次のユース ケースについて、DevOps を検討してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-112">Consider DevOps for the following use cases:</span></span>

* <span data-ttu-id="bdd29-113">アプリケーション開発とそのライフサイクルを加速する</span><span class="sxs-lookup"><span data-stu-id="bdd29-113">Accelerating application development and development life cycles</span></span>
* <span data-ttu-id="bdd29-114">自動化されたビルドおよびリリース プロセスで品質と一貫性を確保する</span><span class="sxs-lookup"><span data-stu-id="bdd29-114">Building quality and consistency into an automated build and release process</span></span>

## <a name="architecture"></a><span data-ttu-id="bdd29-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="bdd29-115">Architecture</span></span>

![Azure DevOps および Azure App Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャ概要][architecture]

<span data-ttu-id="bdd29-117">このシナリオでは、Azure DevOps を使用した .NET Web アプリケーションの CI/CD パイプラインに対応できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-117">This scenario covers a CI/CD pipeline for a .NET web application using Azure DevOps.</span></span> <span data-ttu-id="bdd29-118">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="bdd29-119">アプリケーションのソース コードを変更します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-119">Change application source code.</span></span>
2. <span data-ttu-id="bdd29-120">アプリケーション コードと Web Apps の web.config ファイルをコミットします。</span><span class="sxs-lookup"><span data-stu-id="bdd29-120">Commit application code and Web Apps web.config file.</span></span>
3. <span data-ttu-id="bdd29-121">継続的インテグレーションによって、アプリケーションのビルドと単体テストがトリガーされます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-121">Continuous integration triggers application build and unit tests.</span></span>
4. <span data-ttu-id="bdd29-122">継続的デプロイ トリガーにより、"*環境固有のパラメーター構成値を使用して*" アプリケーション成果物のデプロイが調整されます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-122">Continuous deployment trigger orchestrates deployment of application artifacts *with environment-specific parameterized configuration values*.</span></span>
5. <span data-ttu-id="bdd29-123">Azure App Service にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="bdd29-123">Deployment to Azure App Service.</span></span>
6. <span data-ttu-id="bdd29-124">Azure Application Insights が、正常性、パフォーマンス、使用状況のデータを収集して分析します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-124">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="bdd29-125">正常性、パフォーマンス、および使用状況の情報を確認します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-125">Review health, performance, and usage information.</span></span>

### <a name="components"></a><span data-ttu-id="bdd29-126">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="bdd29-126">Components</span></span>

* <span data-ttu-id="bdd29-127">[Azure DevOps][vsts] は、計画やプロジェクト管理から、コード管理、ビルド、リリースまで、開発ライフサイクルをエンド ツー エンドで管理するためのサービスです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-127">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>
* <span data-ttu-id="bdd29-128">[Azure Web Apps][web-apps] は、Web アプリケーション、REST API、およびモバイル バックエンドをホストするための PaaS サービスです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-128">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="bdd29-129">この記事では .NET に焦点を当てていますが、他にも開発プラットフォームのオプションが複数サポートされています。</span><span class="sxs-lookup"><span data-stu-id="bdd29-129">While this article focuses on .NET, there are several additional development platform options supported.</span></span>
* <span data-ttu-id="bdd29-130">[Application Insights][application-insights] は、複数のプラットフォームで使用できるファーストパーティの拡張可能な Web 開発者向けアプリケーション パフォーマンス管理 (APM) サービスです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-130">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternative-devops-tooling-options"></a><span data-ttu-id="bdd29-131">代替の DevOps ツール オプション</span><span class="sxs-lookup"><span data-stu-id="bdd29-131">Alternative DevOps tooling options</span></span>

<span data-ttu-id="bdd29-132">この記事では Azure DevOps に焦点を当てていますが、オンプレミスでは [Team Foundation Server][team-foundation-server] を代わりに使用できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-132">While this article focuses on Azure DevOps, [Team Foundation Server][team-foundation-server] could be used as on-premises substitute.</span></span> <span data-ttu-id="bdd29-133">または、[Jenkins][jenkins-on-azure] を使用するオープン ソース開発パイプライン用の一連のテクノロジを使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-133">Alternatively, you could also use a set of technologies for an open source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="bdd29-134">コードとしてのインフラストラクチャの観点からは、[Azure Resource Manager テンプレート][arm-templates]が Azure DevOps プロジェクトの一部として含まれていますが、[Terraform][terraform] や [Chef][chef] を検討することもできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-134">From an infrastructure-as-code perspective, [Azure Resource Manager Templates][arm-templates] are included as part of the Azure DevOps project, but you could consider [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="bdd29-135">サービスとしてのインフラストラクチャ (IaaS) ベースのデプロイを使用するときに、構成管理が必要な場合は、[Azure Automation State Configuration][desired-state-configuration]、[Ansible][ansible]、または [Chef][chef] を検討できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-135">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

### <a name="alternatives-to-azure-web-apps"></a><span data-ttu-id="bdd29-136">Azure Web Apps に代わる方法</span><span class="sxs-lookup"><span data-stu-id="bdd29-136">Alternatives to Azure Web Apps</span></span>

<span data-ttu-id="bdd29-137">Azure Web Apps でのホスティングに代わる方法として以下を検討できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-137">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

* <span data-ttu-id="bdd29-138">[Azure Virtual Machines][compare-vm-hosting] &mdash; 細かな制御が必要なワークロード、または Web Apps では使用できない OS コンポーネントおよびサービス (Windows GAC、COM など) に依存するワークロードの場合。</span><span class="sxs-lookup"><span data-stu-id="bdd29-138">[Azure Virtual Machines][compare-vm-hosting] &mdash; For workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>
* <span data-ttu-id="bdd29-139">[Service Fabric][service-fabric] &mdash; ワークロード アーキテクチャが、クラスター全体で細かく制御しながらデプロイおよび実行できることでメリットを得られる分散コンポーネントであることを重視している場合に適したオプションです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-139">[Service Fabric][service-fabric] &mdash; a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="bdd29-140">Service Fabric を使ってコンテナーをホストすることもできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-140">Service Fabric can also be used to host containers.</span></span>
* <span data-ttu-id="bdd29-141">[Azure Functions][azure-functions] - 依存関係が最小限で済む細かな分散コンポーネントであることを重視したワークロード アーキテクチャに適した効果的なサーバーレス アプローチです。個々のコンポーネントは (継続的ではなく) オンデマンドで実行するためにのみ必要で、コンポーネントのオーケストレーションが不要な場合です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-141">[Azure Functions][azure-functions] - an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="bdd29-142">移行の適切なパスを選択するときは、こちらの[デシジョン ツリー](/azure/architecture/guide/technology-choices/compute-decision-tree)が役に立つ場合があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-142">This [decision tree](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

### <a name="devops"></a><span data-ttu-id="bdd29-143">DevOps</span><span class="sxs-lookup"><span data-stu-id="bdd29-143">DevOps</span></span>

<span data-ttu-id="bdd29-144">**[継続的インテグレーション (CI)][continuous-integration]** では、複数の開発者が定期的に共有コードベースに対して小さい変更を頻繁にコミットする場合に、ビルドの安定性が維持されます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-144">**[Continuous Integration (CI)][continuous-integration]** maintains a stable build, with multiple developers regularly committing small, frequent changes to the shared codebase.</span></span> <span data-ttu-id="bdd29-145">継続的インテグレーション パイプラインの一環として、次を行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-145">As part of your continuous integration pipeline, you should:</span></span>
* <span data-ttu-id="bdd29-146">小さなコードの変更を頻繁にコミットします。</span><span class="sxs-lookup"><span data-stu-id="bdd29-146">Frequently commit smaller code changes.</span></span> <span data-ttu-id="bdd29-147">正常にマージするのがより難しい可能性のある、大きい変更またはより複雑な変更をバッチ処理するのは避けます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-147">Avoid batching up larger or more complex changes that may be more difficult to merge successfully.</span></span>
* <span data-ttu-id="bdd29-148">十分なコード カバレッジ (アンハッピー パスのテストを含む) でアプリケーション コンポーネントの単体テストを実施します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-148">Conduct unit testing of your application components with sufficient code coverage, including testing the unhappy paths.</span></span>
* <span data-ttu-id="bdd29-149">ビルドは、必ず共有のマスター (またはトランク) ブランチに対して実行します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-149">Ensure the build is run against the shared master (or trunk) branch.</span></span> <span data-ttu-id="bdd29-150">このブランチは安定性があり、"デプロイ準備完了" として維持されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-150">This branch should be stable and maintained as "deployment ready".</span></span> <span data-ttu-id="bdd29-151">不完全な変更または処理中の変更については、競合を回避するために、頻繁な "前方統合" マージによって個別のブランチに分離する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-151">Incomplete or work-in-progress changes should be isolated in a separate branch with frequent "forward integration" merges to avoid conflicts later.</span></span>

<span data-ttu-id="bdd29-152">**[継続的デリバリー (CD)][continuous-delivery]** では、安定したビルドだけでなく、安定したデプロイも実証されます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-152">**[Continuous Delivery (CD)][continuous-delivery]** demonstrates not just a stable build but a stable deployment.</span></span> <span data-ttu-id="bdd29-153">これにより CD の実現が少し困難になるため、環境固有の構成と、その値を正しく設定するメカニズムが必要になります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-153">This makes realizing CD a little more difficult, requiring environment-specific configuration and a mechanism for setting those values correctly.</span></span> <span data-ttu-id="bdd29-154">CD に関しては他に次のような考慮事項があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-154">Other CD considerations include the following:</span></span>
* <span data-ttu-id="bdd29-155">さまざまなコンポーネントが適切に構成され、エンド ツー エンドで動作することを検証するため、統合テストの十分なカバレッジが必要です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-155">Sufficient integration testing coverage is required to validate that the various components are configured and working correctly end-to-end.</span></span>
* <span data-ttu-id="bdd29-156">CD では、環境固有のデータの設定とリセット、およびデータベース スキーマ バージョンの管理が必要になる可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-156">CD may also require setting up and resetting environment-specific data and managing database schema versions.</span></span>
* <span data-ttu-id="bdd29-157">継続的デリバリーをさらにロード テストやユーザー受け入れテストの環境に拡張する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-157">Continuous delivery should also extend to load testing and user acceptance testing environments.</span></span>
* <span data-ttu-id="bdd29-158">継続的デリバリーでは、すべての環境で継続的監視のメリットを得られることが理想です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-158">Continuous delivery benefits from continuous monitoring, ideally across all environments.</span></span>
* <span data-ttu-id="bdd29-159">デプロイの一貫性と信頼性および環境全体に対する統合テストは、ホスティング インフラストラクチャの作成と構成をスクリプト化することで容易になります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-159">The consistency and reliability of deployments and integration testing across environments is made easier by scripting the creation and configuration of the hosting infrastructure.</span></span> <span data-ttu-id="bdd29-160">これは、クラウド ベースのワークロードではかなり簡単です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-160">This is considerably easier for cloud-based workloads.</span></span> <span data-ttu-id="bdd29-161">詳しくは、「[Infrastructure as Code][infra-as-code]」(コードとしてのインフラストラクチャ) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-161">For more information, see [Infrastructure as Code][infra-as-code].</span></span>
* <span data-ttu-id="bdd29-162">プロジェクトのライフサイクルのできるだけ早い段階で、継続的デリバリーを開始します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-162">Begin continuous delivery as early as possible in the project lifecycle.</span></span> <span data-ttu-id="bdd29-163">始めるのが遅いほど、組み込むのが難しくなります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-163">The later you begin, the more difficult it will be to incorporate.</span></span>
* <span data-ttu-id="bdd29-164">統合および単体テストの優先度は、アプリケーションの機能と同じにします。</span><span class="sxs-lookup"><span data-stu-id="bdd29-164">Integration and unit tests should be given the same priority as application features.</span></span>
* <span data-ttu-id="bdd29-165">環境に依存しないデプロイ パッケージを使用し、リリース プロセスを通じて環境固有の構成を管理します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-165">Use environment-agnostic deployment packages and manage environment-specific configuration via the release process.</span></span>
* <span data-ttu-id="bdd29-166">リリース プロセス中は、リリース管理ツールを使用するか、ハードウェア セキュリティ モジュール (HSM)、または [Azure Key Vault][azure-key-vault] を呼び出して、機密を要する構成を保護します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-166">Protect sensitive configuration using the release management tooling, or by calling out to a Hardware-security-module (HSM) or [Azure Key Vault][azure-key-vault] during the release process.</span></span> <span data-ttu-id="bdd29-167">ソース管理内には機密を要する構成を格納しないでください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-167">Do not store sensitive configuration within source control.</span></span>

<span data-ttu-id="bdd29-168">**継続的な学習**。</span><span class="sxs-lookup"><span data-stu-id="bdd29-168">**Continuous Learning**.</span></span> <span data-ttu-id="bdd29-169">CD 環境を最も効果的に監視するには、[Application Insights][application-insights] などのアプリケーション パフォーマンス監視ツール (APM) を使用します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-169">The most effective monitoring of a CD environment is provided by application performance monitoring (APM) tools such as [Application Insights][application-insights].</span></span> <span data-ttu-id="bdd29-170">アプリケーション ワークロードを十分に監視することは、バグや、負荷の下でのパフォーマンスを把握するうえで重要です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-170">Sufficient depth of monitoring for an application workload is critical to understand bugs or performance under load.</span></span> <span data-ttu-id="bdd29-171">[Application Insights を VSTS に統合して、CD パイプラインの継続的な監視を有効にできます][app-insights-cd-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="bdd29-171">Application Insights can be integrated into VSTS to enable [continuous monitoring of the CD pipeline][app-insights-cd-monitoring].</span></span> <span data-ttu-id="bdd29-172">これを使用すると、ユーザーの介入なしで、次のステージに自動的に進むことができます。また、アラートが検出された場合も自動的にロールバックできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-172">This could be used to enable automatic progression to the next stage, without human intervention, or rollback if an alert is detected.</span></span>

## <a name="considerations"></a><span data-ttu-id="bdd29-173">考慮事項</span><span class="sxs-lookup"><span data-stu-id="bdd29-173">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="bdd29-174">可用性</span><span class="sxs-lookup"><span data-stu-id="bdd29-174">Availability</span></span>

<span data-ttu-id="bdd29-175">クラウド アプリケーションを構築するときは、[可用性のための標準的な設計パターン][design-patterns-availability]の利用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-175">Consider leveraging the [typical design patterns for availability][design-patterns-availability] when building your cloud application.</span></span>

<span data-ttu-id="bdd29-176">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]で可用性に関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="bdd29-176">Review the availability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="bdd29-177">可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-177">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="bdd29-178">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="bdd29-178">Scalability</span></span>

<span data-ttu-id="bdd29-179">クラウド アプリケーションを構築するときは、[スケーラビリティのための標準的な設計パターン][design-patterns-scalability]に注意してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-179">When building a cloud application be aware of the [typical design patterns for scalability][design-patterns-scalability].</span></span>

<span data-ttu-id="bdd29-180">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でスケーラビリティに関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="bdd29-180">Review the scalability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="bdd29-181">スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-181">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="bdd29-182">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="bdd29-182">Security</span></span>

<span data-ttu-id="bdd29-183">必要に応じて、[セキュリティのための標準的な設計パターン][design-patterns-security]を利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-183">Consider leveraging the [typical design patterns for security][design-patterns-security] where appropriate.</span></span>

<span data-ttu-id="bdd29-184">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でセキュリティに関する考慮事項を確認します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-184">Review the security considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture].</span></span>

<span data-ttu-id="bdd29-185">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-185">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="bdd29-186">回復性</span><span class="sxs-lookup"><span data-stu-id="bdd29-186">Resiliency</span></span>

<span data-ttu-id="bdd29-187">必要に応じて、[回復性のための標準的な設計パターン][design-patterns-resiliency]を実装することを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-187">Consider implementing the [typical design patterns for resiliency][design-patterns-resiliency] where appropriate.</span></span>

<span data-ttu-id="bdd29-188">Azure アーキテクチャ センターでは、多数の [App Service に関する推奨プラクティス][resiliency-app-service]を確認できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-188">You can find a number of [recommended practices for App Service][resiliency-app-service] in the Azure Architecture Center.</span></span>

<span data-ttu-id="bdd29-189">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-189">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="bdd29-190">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="bdd29-190">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="bdd29-191">前提条件</span><span class="sxs-lookup"><span data-stu-id="bdd29-191">Prerequisites</span></span>

* <span data-ttu-id="bdd29-192">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="bdd29-192">You must have an existing Azure account.</span></span> <span data-ttu-id="bdd29-193">Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][azure-free-account]を作成してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-193">If you don't have an Azure subscription, create a [free account][azure-free-account] before you begin.</span></span>
* <span data-ttu-id="bdd29-194">Azure DevOps 組織にサインアップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-194">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="bdd29-195">詳しくは、「[Quickstart: Create your organization][vsts-account-create]」(クイック スタート: 組織を作成する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-195">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="bdd29-196">チュートリアル</span><span class="sxs-lookup"><span data-stu-id="bdd29-196">Walk-through</span></span>

<span data-ttu-id="bdd29-197">このシナリオでは、Azure DevOps プロジェクトを使用して、CI/CD パイプラインを作成します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-197">In this scenario, you'll use the Azure DevOps project to create your CI/CD pipeline.</span></span>

<span data-ttu-id="bdd29-198">Azure DevOps プロジェクトでは、App Service プラン、App Service と App Insights のリソースが自動的にデプロイされ、Azure DevOps プロジェクトが自動的に構成されます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-198">The Azure DevOps project will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="bdd29-199">Azure DevOps プロジェクトの開発とビルドが完了したら、関連するコード変更、作業項目、およびテスト結果を確認します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-199">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="bdd29-200">コードには実行するテストが含まれないため、テスト結果は表示されません。</span><span class="sxs-lookup"><span data-stu-id="bdd29-200">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="bdd29-201">リリース定義を確認します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-201">Review the release definitions.</span></span> <span data-ttu-id="bdd29-202">リリース パイプラインが設定され、アプリケーションが開発環境にリリースされていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-202">Notice that a release pipeline has been set up, releasing our application into the Dev environment.</span></span> <span data-ttu-id="bdd29-203">開発環境への自動リリースを含む**ドロップ** ビルド成果物から**継続的配置トリガー**が設定されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-203">Observe that there is a **continuous deployment trigger** set from the **Drop** build artifact, with automatic releases into the Dev environment.</span></span> <span data-ttu-id="bdd29-204">継続的配置プロセスの一環として、リリースが複数の環境にまたがっていることを確認できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-204">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="bdd29-205">リリースは両方のインフラストラクチャにまたがることができます (コードとしてのインフラストラクチャなどの手法を使用)。また、このリリースにより、必要なアプリケーション パッケージと、構成後のタスクもデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-205">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="additional-considerations"></a><span data-ttu-id="bdd29-206">追加の考慮事項</span><span class="sxs-lookup"><span data-stu-id="bdd29-206">Additional considerations</span></span>

* <span data-ttu-id="bdd29-207">VSTS マーケットプレースで入手できる[トークン化タスク][vsts-tokenization]のいずれかを利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-207">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>
* <span data-ttu-id="bdd29-208">[Azure Key Vault のデプロイ][download-keyvault-secrets] VSTS タスクを使用して、Azure Key Vault からリリースにシークレットをダウンロードすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-208">Consider using the [Deploy: Azure Key Vault][download-keyvault-secrets] VSTS task to download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="bdd29-209">その後、リリース定義で変数としてこれらのシークレットを使用できるので、ソース管理に格納しなくてもかまいません。</span><span class="sxs-lookup"><span data-stu-id="bdd29-209">You can then use those secrets as variables in your release definition, so you can avoid storing them in source control.</span></span>
* <span data-ttu-id="bdd29-210">お使いの環境の構成変更を促進するために、ご自身のリリース定義で[リリース変数][vsts-release-variables]を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-210">Consider using [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="bdd29-211">リリース変数は、リリース全体または特定の環境を対象にできます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-211">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="bdd29-212">シークレット情報に変数を使用している場合は、必ず南京錠アイコンを選択します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-212">When using variables for secret information, ensure that you select the padlock icon.</span></span>
* <span data-ttu-id="bdd29-213">リリース パイプラインで[デプロイ ゲート][vsts-deployment-gates]を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-213">Consider using [deployment gates][vsts-deployment-gates] in your release pipeline.</span></span> <span data-ttu-id="bdd29-214">これにより、外部システム (インシデント管理システム、追加の特注システムなど) と共に監視データを利用して、リリースを昇格させる必要があるかどうかを判断できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-214">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>
* <span data-ttu-id="bdd29-215">リリース パイプラインで手動介入が必要な場合は、[承認][vsts-approvals]機能を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-215">Where manual intervention in a release pipeline is required, consider using the [approvals][vsts-approvals] functionality.</span></span>
* <span data-ttu-id="bdd29-216">リリース パイプラインで [Application Insights][application-insights] と追加の監視ツールをできるだけ早く使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-216">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="bdd29-217">多くの組織は、運用環境でしか監視を開始しません。他の環境を監視することにより、開発プロセスの早い段階でバグを発見し、運用環境で問題が発生するのを回避できます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-217">Many organizations only begin monitoring in their production environment; by monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="pricing"></a><span data-ttu-id="bdd29-218">価格</span><span class="sxs-lookup"><span data-stu-id="bdd29-218">Pricing</span></span>

<span data-ttu-id="bdd29-219">Azure DevOps のコストは、アクセスを必要とする組織内のユーザーの数だけでなく、必要な同時実行ビルド/リリースの数やテスト ユーザーの数など、他の要因にも依存します。</span><span class="sxs-lookup"><span data-stu-id="bdd29-219">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="bdd29-220">詳しくは、「[Azure DevOps の料金][vsts-pricing-page]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="bdd29-220">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

* <span data-ttu-id="bdd29-221">[Azure DevOps][vsts-pricing-calculator] は、開発ライフサイクルの管理を可能にするサービスです。</span><span class="sxs-lookup"><span data-stu-id="bdd29-221">[Azure DevOps][vsts-pricing-calculator] is a service that enables you to manage your development life cycle.</span></span> <span data-ttu-id="bdd29-222">ユーザー単位で 1 か月ごとに課金されます。</span><span class="sxs-lookup"><span data-stu-id="bdd29-222">It is paid for on a per-user per-month basis.</span></span> <span data-ttu-id="bdd29-223">必要な同時実行パイプライン、追加のテスト ユーザー、またはユーザーの基本ライセンスによっては、追加料金が発生する場合があります。</span><span class="sxs-lookup"><span data-stu-id="bdd29-223">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="bdd29-224">関連リソース</span><span class="sxs-lookup"><span data-stu-id="bdd29-224">Related resources</span></span>

* <span data-ttu-id="bdd29-225">[DevOps とは][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="bdd29-225">[What is DevOps?][devops-whatis]</span></span>
* <span data-ttu-id="bdd29-226">[Microsoft での DevOps - Azure DevOps をどのように使用しているか][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="bdd29-226">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
* <span data-ttu-id="bdd29-227">[テップ バイ ステップ チュートリアル: Azure DevOps での DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="bdd29-227">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
* <span data-ttu-id="bdd29-228">[DevOps チェックリスト][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="bdd29-228">[Devops Checklist][devops-checklist]</span></span>
* <span data-ttu-id="bdd29-229">[Azure DevOps プロジェクトを使用して .NET 用 CI/CD パイプラインを作成する][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="bdd29-229">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
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
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/