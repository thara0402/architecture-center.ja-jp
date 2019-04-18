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
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-dotnet-webapp.svg
ms.openlocfilehash: f999b2ffdf234161f668887d5b2327ecf50f0e55
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740410"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>Azure DevOps を使用した CI/CD パイプラインの設計

このシナリオでは、継続的インテグレーション (CI) と継続的配置 (CD) パイプラインを構築するためのアーキテクチャと設計のガイダンスを示します。 この例の CI/CD パイプラインは、Azure App Service に 2 層の .NET Web アプリケーションをデプロイします。

最新の CI/CD プロセスへの移行には、アプリケーションのビルド、デプロイ、テスト、および監視の点で多くのメリットがあります。 Azure DevOps を App Service などの他のサービスと共に使用することで、組織では、自身のシナリオを実現するためのサポート インフラストラクチャの管理ではなく、アプリ自体の開発に集中することができます。

## <a name="relevant-use-cases"></a>関連するユース ケース

次のユースケースについて、Azure DevOps および CI/CD プロセスを検討してください。

- アプリケーション開発とそのライフサイクルを加速する
- 自動化されたビルドおよびリリース プロセスで品質と一貫性を確保する
- アプリケーションの安定性と稼働時間を増やす

## <a name="architecture"></a>アーキテクチャ

![Azure DevOps および Azure App Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャ ダイアグラム][architecture]

このシナリオのデータ フローは次のとおりです。

1. 開発者がアプリケーションのソース コードを変更します。
2. web.config ファイルを含むアプリケーション コードが、Azure Repos のソース コード リポジトリにコミットされます。
3. 継続的インテグレーションによって、Azure Test Plans を使用してアプリケーションのビルドと単体テストがトリガーされます。
4. Azure Pipelines 内の継続的配置により、"*環境固有の構成値を使用して*" アプリケーション成果物の自動化されたデプロイがトリガーされます。
5. 成果物が Azure App Service にデプロイされます。
6. Azure Application Insights が、正常性、パフォーマンス、使用状況のデータを収集して分析します。
7. 開発者が、正常性、パフォーマンス、および使用状況の情報を監視および管理します。
8. バックログ情報は、Azure Boards を使用した新機能とバグ修正の優先順位付けに使用されます。

### <a name="components"></a>コンポーネント

- [Azure DevOps][vsts] は、計画やプロジェクト管理から、コード管理、ビルド、リリースまで、開発ライフサイクルをエンド ツー エンドで管理するためのサービスです。

- [Azure Web Apps][web-apps] は、Web アプリケーション、REST API、およびモバイル バックエンドをホストするための PaaS サービスです。 この記事では .NET に焦点を当てていますが、他にも開発プラットフォームのオプションが複数サポートされています。

- [Application Insights][application-insights] は、複数のプラットフォームで使用できるファーストパーティの拡張可能な Web 開発者向けアプリケーション パフォーマンス管理 (APM) サービスです。

### <a name="alternatives"></a>代替手段

この記事では Azure DevOps に焦点を当てていますが、オンプレミスでは [Azure DevOps Server][azure-devops-server] (旧称 Team Foundation Server) を代わりに使用できます。 または、[Jenkins][jenkins-on-azure] を使用するオープンソース開発パイプライン用の一連のテクノロジを使用することもできます。

コードとしてのインフラストラクチャの観点からは、[Resource Manager テンプレート][arm-templates]が Azure DevOps プロジェクトの一部として使用されていますが、[Terraform][terraform] や [Chef][chef] などの他の管理テクノロジを検討することもできます。 サービスとしてのインフラストラクチャ (IaaS) ベースのデプロイを使用するときに、構成管理が必要な場合は、[Azure Automation State Configuration][desired-state-configuration]、[Ansible][ansible]、または [Chef][chef] を検討できます。

Azure Web Apps でのホスティングに代わる方法として以下を検討できます。

- [Azure Virtual Machines][compare-vm-hosting] は、細かな制御が必要なワークロード、または Web Apps では使用できない OS コンポーネントおよびサービス (Windows GAC、COM など) に依存するワークロードを処理します。

- [Service Fabric][service-fabric] は、ワークロード アーキテクチャが、クラスター全体で細かく制御しながらデプロイおよび実行できることでメリットを得られる分散コンポーネントであることを重視している場合に適したオプションです。 Service Fabric を使ってコンテナーをホストすることもできます。

- [Azure Functions][azure-functions] は、依存関係が最小限で済む細かな分散コンポーネントであることを重視したワークロード アーキテクチャに適した効果的なサーバーレス アプローチです。個々のコンポーネントは (継続的ではなく) オンデマンドで実行するためにのみ必要で、コンポーネントのオーケストレーションが不要な場合です。

移行の適切なパスを選択するときは、この [Azure コンピューティング サービスのデシジョン ツリー](/azure/architecture/guide/technology-choices/compute-decision-tree)が役に立つ場合があります。

## <a name="management-and-security-considerations"></a>管理とセキュリティに関する考慮事項

- VSTS マーケットプレースで入手できる[トークン化タスク][vsts-tokenization]のいずれかを利用することを検討してください。

- [Azure Key Vault][download-keyvault-secrets] タスクでは、Azure Key Vault からリリースにシークレットをダウンロードできます。 その後、リリース定義で変数としてこれらのシークレットを使用でき、ソース管理に格納しなくてもかまいません。

- お使いの環境の構成変更を促進するために、ご自身のリリース定義で[リリース変数][vsts-release-variables]を使用してください。 リリース変数は、リリース全体または特定の環境を対象にできます。 シークレット情報に変数を使用している場合は、必ず南京錠アイコンを選択します。

- リリース パイプラインでは[デプロイ ゲート][vsts-deployment-gates]を使用してください。 これにより、外部システム (インシデント管理システム、追加の特注システムなど) と共に監視データを利用して、リリースを昇格させる必要があるかどうかを判断できます。

- リリース パイプラインで手動介入が必要な場合は、[承認][vsts-approvals]機能を使用してください。

- リリース パイプラインで [Application Insights][application-insights] と追加の監視ツールをできるだけ早く使用することを検討します。 多くの組織は、運用環境でしか監視を開始しません。 他の環境を監視することにより、開発プロセスの早い段階でバグを発見し、運用環境で問題が発生するのを回避できます。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

### <a name="prerequisites"></a>前提条件

- 既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

- Azure DevOps 組織にサインアップする必要があります。 詳細については、[クイック スタートの組織の作成][vsts-account-create]に関するページを参照してください。

### <a name="walk-through"></a>チュートリアル

[Azure DevOps Projects](/azure/devops-project/azure-devops-project-github) では、App Service プラン、App Service と App Insights のリソースが自動的にデプロイされ、Azure Pipelines のパイプラインが自動的に構成されます。

Azure DevOps Projects によるパイプラインの構成が完了したら、関連するコード変更、作業項目、およびテスト結果を確認します。 コードには実行するテストが含まれないため、テスト結果は表示されません。

このパイプラインでは、リリース定義と継続的配置トリガーを作成して、アプリケーションを開発環境にデプロイします。 継続的配置プロセスの一環として、リリースが複数の環境にまたがっていることを確認できます。 リリースは両方のインフラストラクチャにまたがることができます (コードとしてのインフラストラクチャなどの手法を使用)。また、このリリースにより、必要なアプリケーション パッケージと、構成後のタスクもデプロイできます。

## <a name="pricing"></a>価格

Azure DevOps のコストは、アクセスを必要とする組織内のユーザーの数だけでなく、必要な同時実行ビルド/リリースの数やテスト ユーザーの数など、他の要因にも依存します。 詳しくは、「[Azure DevOps の料金][vsts-pricing-page]」をご覧ください。

この[料金計算ツール][vsts-pricing-calculator]は、Azure DevOps を 20 ユーザーで実行する場合の見積もりを提供します。

Azure DevOps はユーザー単位で 1 か月ごとに課金されます。 必要な同時実行パイプライン、追加のテスト ユーザー、またはユーザーの基本ライセンスによっては、追加料金が発生する場合があります。

## <a name="related-resources"></a>関連リソース

CI/CD および Azure DevOps の詳細については、次のリソースを参照してください。

- [DevOps とは][devops-whatis]
- [Microsoft での DevOps - Azure DevOps をどのように使用しているか][devops-microsoft]
- [ステップバイステップのチュートリアル:Azure DevOps を使用した DevOps][devops-with-vsts]
- [DevOps チェックリスト][devops-checklist]
- [Azure DevOps Projects を使用して .NET 用 CI/CD パイプラインを作成する][devops-project-create]

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
