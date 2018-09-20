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
# <a name="cicd-pipeline-with-vsts"></a>VSTS を使用した CI/CD パイプライン

DevOps とは、開発、品質保証、および IT 運用の統合です。 DevOps には、ソフトウェアを供給するための統合カルチャと一連の強力なプロセスの両方が必要です。

このサンプル シナリオでは、開発チームが Visual Studio Team Services を使用して、.NET 2 層 Web アプリを Azure App Service にデプロイする方法を示します。 Web アプリケーションは、ダウンストリームにある Azure PaaS (サービスとしてのプラットフォーム) サービスに依存します。 また、このドキュメントでは、Azure PaaS (サービスとしてのプラットフォーム) を使用してこのようなシナリオを設計するときに考慮すべきいくつかの点についても紹介します。

継続的インテグレーション (CI) と継続的デプロイ (CD) を使用したアプリケーション開発の最新アプローチを採用することで、堅牢なビルド、テスト、デプロイ、および監視サービスを通じて、ユーザーへの価値提供の迅速化に役立てることができます。 App Service などの Azure サービスに加え、Visual Studio Team Services などのプラットフォームを使用すると、組織は自身のシナリオを実現するためのインフラストラクチャの管理ではなく、シナリオ自体の開発に集中し続けることができます。

## <a name="related-use-cases"></a>関連するユース ケース

次のユース ケースについて、DevOps を検討してください。

* アプリケーション開発とそのライフサイクルを加速化する
* 自動化されたビルドおよびリリース プロセスで品質と一貫性を確保する

## <a name="architecture"></a>アーキテクチャ

![Visual Studio Team Services および Azure App Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャ概要][architecture]

このシナリオでは、Visual Studio Team Services (VSTS) を使用した .NET Web アプリケーションの DevOps パイプラインに対応できます。 このシナリオのデータ フローは次のとおりです。

1. アプリケーションのソース コードを変更します。
2. アプリケーション コードと Web Apps の web.config ファイルをコミットします。
3. 継続的インテグレーションによって、アプリケーションのビルドと単体テストがトリガーされます。
4. 継続的デプロイ トリガーにより、"*環境固有のパラメーター構成値を使用して*" アプリケーション成果物のデプロイが調整されます。
5. Azure App Service にデプロイします。
6. Azure Application Insights が、正常性、パフォーマンス、使用状況のデータを収集して分析します。
7. 正常性、パフォーマンス、および使用状況の情報を確認します。

### <a name="components"></a>コンポーネント

* [リソース グループ][resource-groups]は Azure リソースの論理コンテナーであり、管理プレーンのアクセス制御の境界も提供します。リソース グループは、"デプロイの単位" を表すと考えてください。
* [Visual Studio Team Services (VSTS)][vsts] は、計画やプロジェクト管理から、コード管理、ビルド、リリースまで、開発ライフサイクルをエンド ツー エンドで管理できるようにするサービスです。
* [Azure Web Apps][web-apps] は、Web アプリケーション、REST API、およびモバイル バックエンドをホストするための、サービスとしてのプラットフォーム (PaaS) サービスです。 この記事では .NET に焦点を当てていますが、他にも開発プラットフォームのオプションが複数サポートされています。
* [Application Insights][application-insights] は、複数のプラットフォームで使用できるファーストパーティの拡張可能な Web 開発者向けアプリケーション パフォーマンス管理 (APM) サービスです。

### <a name="alternative-devops-tooling-options"></a>代替の DevOps ツール オプション

この記事では Visual Studio Team Services に焦点を当てていますが、オンプレミスでは [Team Foundation Server][team-foundation-server] を代わりに使用できます。 また、[Jenkins][jenkins-on-azure] を利用して、オープン ソース開発パイプライン向けに一連のテクノロジ コレクションを使用することもできます。

コードとしてのインフラストラクチャの観点からは、[Azure Resource Manager テンプレート][arm-templates]が Azure DevOps プロジェクトの一部として含まれていますが、[Terraform][terraform] や [Chef][chef] に投資済みの場合はここで検討することもできます。 サービスとしてのインフラストラクチャ (IaaS) ベースのデプロイを使用するときに、構成管理が必要な場合は、[Azure Automation State Configuration][desired-state-configuration]、[Ansible][ansible]、または [Chef][chef] を検討できます。

### <a name="alternatives-to-web-app-hosting"></a>Web アプリ ホスティングに代わる方法

Azure Web Apps でのホスティングに代わる方法:

* [VM][compare-vm-hosting] - 細かな制御が必要なワークロード、または Web Apps では使用できない OS コンポーネント/サービス (Windows GAC、COM など) に依存するワークロードの場合
* [コンテナーのホスティング][azure-containers] - OS の依存関係がある場合は、ホスティングの移植性またはホスティング密度も要件となります。
* [Service Fabric][service-fabric] - ワークロード アーキテクチャが、クラスター全体で細かく制御しながらデプロイおよび実行できることでメリットを得られる分散コンポーネントであることを重視している場合に適したオプションです。 Service Fabric を使ってコンテナーをホストすることもできます。
* [サーバーレス Azure Functions][azure-functions] - 依存関係が最小限で済む細かな分散コンポーネントであることを重視したワークロード アーキテクチャに適したオプションです。個々のコンポーネントは (継続的ではなく) オンデマンドで実行するためにのみ必要で、コンポーネントのオーケストレーションが不要な場合です。

### <a name="devops"></a>DevOps

**[継続的インテグレーション (CI)][continuous-integration]** は、複数の個人開発者またはチームが共有コードベースに対して小さな変更を頻繁かつ継続的にコミットする状況において、安定したビルドを実証することを目標としています。
継続的インテグレーション パイプラインの一環として、次を行う必要があります。

* 少量のコードを頻繁にチェックインします (複雑で大きな変更を一括で行うと、適切にマージするのが難しくなるため避けてください)
* 十分なコード カバレッジ (アンハッピー パスを含む) でご自身のアプリケーションのコンポーネントを単体テストします
* ビルドは、必ず共有のマスター (またはトランク) ブランチに対して実行します。 このブランチは安定性があり、"デプロイ準備完了" として維持されている必要があります。 不完全な変更または処理中の変更については、競合を回避するために、頻繁な "前方統合" マージによって個別のブランチに分離する必要があります。

**[継続的デリバリー (CD)][continuous-delivery]** は、安定したビルドだけでなく、安定したデプロイを実証することを目標としています。 これにより CD の実現が少し困難になるため、環境固有の構成と、その値を正しく設定するメカニズムが必要になります。

また、さまざまなコンポーネントが適切に構成され、エンド ツー エンドで確実に動作するように、十分なカバレッジの統合テストも必要です。

このため、環境固有のデータの設定とリセット、およびデータベース スキーマ バージョンの管理が必要になる可能性もあります。

継続的デリバリーをさらにロード テストやユーザー受け入れテストの環境に拡張することもできます。

継続的デリバリーでは、すべての環境で継続的監視のメリットを得られることが理想です。
環境全体におけるデプロイおよび統合テストの一貫性と信頼性は、作成/構成またはホスティング インフラストラクチャのスクリプトを作成することで、さらに容易に実現します (クラウドベースのワークロードでは大幅に容易になるもの。Azure のコードとしてのインフラストラクチャを参照)。これは、"[コードとしてのインフラストラクチャ][infra-as-code]" とも呼ばれます。

* 継続的デリバリーは、プロジェクトのライフサイクルでできるだけ早く開始します。 これは後になるほど難しくなります。
* 統合および単体テストの優先度は、プロジェクトの機能と同じにします
* 環境に依存しないデプロイ パッケージを使用し、リリース プロセスを通じて環境固有の構成を管理します。
* リリース プロセス中は、リリース管理ツールを使用するか、ハードウェア セキュリティ モジュール (HSM)、または [Key Vault][azure-key-vault] を呼び出して、機密を要する構成を保護します。 ソース管理内には機密を要する構成を格納しないでください。

**継続的な学習** - CD 環境を最も効果的に監視するには、Microsoft の [Application Insights][application-insights] などのアプリケーション パフォーマンス監視ツール (略して APM) を使用します。 アプリケーション ワークロードを十分に監視することは、バグや、負荷の下でのパフォーマンスを把握するうえで重要です。 [App Insights を VSTS に統合して、CD パイプラインの継続的な監視を有効にできます][app-insights-cd-monitoring]。 これを使用すると、ユーザーの介入なしで、次のステージに自動的に進むことができます。また、アラートが検出された場合も自動的にロールバックできます。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

クラウド アプリケーションを構築するときは、[可用性のための標準的な設計パターン][design-patterns-availability]の利用を検討してください。

適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]で可用性に関する考慮事項を確認します

可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

クラウド アプリケーションを構築するときは、[スケーラビリティのための標準的な設計パターン][design-patterns-scalability]に注意してください。

適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でスケーラビリティに関する考慮事項を確認します

スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。

### <a name="security"></a>セキュリティ

必要に応じて、[セキュリティのための標準的な設計パターン][design-patterns-security]を利用することを検討してください。

適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でセキュリティに関する考慮事項を確認します。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

[回復性のための標準的な設計パターン][design-patterns-resiliency]を確認し、必要に応じて、これらを実装することを検討します。

Azure アーキテクチャ センターでは、多数の [App Service に関する推奨プラクティス][resiliency-app-service]を確認できます。

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

### <a name="prerequisites"></a>前提条件

* 既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に[無料アカウント][azure-free-account]を作成してください。
* 既存の Visual Studio Team Services (VSTS) アカウントが必要です。 詳細については、[Visual Studio Team Services (VSTS) アカウントの作成][vsts-account-create]に関するページをご覧ください。

### <a name="walk-through"></a>チュートリアル

このシナリオでは、Azure DevOps プロジェクトを使用して、ご自身の CI/CD パイプラインを作成します。

DevOps プロジェクトでは、App Service プラン、App Service と App Insights のリソースが自動的にデプロイされ、Visual Studio Team Services プロジェクトが自動的に構成されます。

DevOps プロジェクトの開発とビルドが完了したら、関連するコード変更、作業項目、およびテスト結果を確認します。 コードには実行するテストが含まれないため、テスト結果は表示されません。

リリース定義を確認します。 リリース パイプラインが設定され、アプリケーションが開発環境にリリースされていることに注意してください。 開発環境への自動リリースを含む**ドロップ** ビルド成果物から**継続的配置トリガー**が設定されていることを確認します。 継続的配置プロセスの一環として、リリースが複数の環境にまたがっていることを確認できます。 リリースは両方のインフラストラクチャにまたがることができます (コードとしてのインフラストラクチャなどの手法を使用)。また、このリリースにより、必要なアプリケーション パッケージと、構成後のタスクもデプロイされます。

**追加の考慮事項。**

* VSTS マーケットプレースで入手できる[トークン化タスク][vsts-tokenization]のいずれかを利用することを検討してください。
* [Azure Key Vault のデプロイ][download-keyvault-secrets] VSTS タスクを使用して、Azure Key Vault からご自身のリリースにシークレットをダウンロードすることを検討します。 その後、変数としてのこれらのシークレットを、リリース定義の一部に使用します。これはソース管理には格納しないでください。
* お使いの環境の構成変更を促進するために、ご自身のリリース定義で[リリース変数][vsts-release-variables]を使用することを検討します。 リリース変数は、リリース全体または特定の環境を対象にできます。 シークレット情報に変数を使用している場合は、必ず南京錠アイコンを選択します。
* リリース パイプラインで[デプロイ ゲート][vsts-deployment-gates]を使用することを検討します。 これにより、外部システム (インシデント管理システム、追加の特注システムなど) と共に監視データを利用して、リリースを昇格させる必要があるかどうかを判断できます。
* リリース パイプラインで手動介入が必要な場合は、[承認][vsts-approvals]機能を使用することを検討します。
* ご自身のリリース パイプラインで [Application Insights][application-insights] と追加の監視ツールをできるだけ早く使用することを検討します。 ほとんどの組織は、運用環境で監視を開始するだけで、プロセスの早い段階で潜在的なバグを特定し、運用環境のユーザーに影響を及ぶのを避けることができます。

## <a name="pricing"></a>価格

ご自身の Visual Studio Team Services のコストは、アクセスを必要とするご自身の組織のユーザー数のほか、必要な同時実行ビルド/リリース数、テスト ユーザー数などの要因によっても異なります。 詳細については、[VSTS の価格に関するページ][vsts-pricing-page]をご覧ください。

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] は、開発ライフサイクルを管理できるようにするサービスで、ユーザーあたりの料金が月単位で請求されます。 必要な同時実行パイプライン、追加のテスト ユーザー、またはユーザーの基本ライセンスによっては、追加料金が発生する場合があります。

## <a name="related-resources"></a>関連リソース

* [DevOps とは][devops-whatis]
* [Microsoft での DevOps - Visual Studio Team Services の使用方法][devops-microsoft]
* [ステップ バイ ステップのチュートリアル: Visual Studio Team Services を使用した DevOps][devops-with-vsts]
* [Azure DevOps プロジェクトを使用して .NET 用 CI/CD パイプラインを作成する][devops-project-create]

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