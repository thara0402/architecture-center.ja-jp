---
title: Azure で Jenkins サーバーを実行する
description: このリファレンス アーキテクチャでは、シングル サインオン (SSO) で保護されたスケーラブルなエンタープライズ レベルの Jenkins サーバーを Azure にデプロイして運用する方法を示します。
author: njray
ms.date: 01/21/18
ms.openlocfilehash: 5f9c54e71a8750e88de1ae633ccc1316f8375d3a
ms.sourcegitcommit: 0de300b6570e9990e5c25efc060946cb9d079954
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/03/2018
ms.locfileid: "32323926"
---
# <a name="run-a-jenkins-server-on-azure"></a>Azure で Jenkins サーバーを実行する

このリファレンス アーキテクチャでは、シングル サインオン (SSO) で保護されたスケーラブルなエンタープライズ レベルの Jenkins サーバーを Azure にデプロイして運用する方法を示します。 また、このアーキテクチャでは、Azure Monitor を使用して Jenkins サーバーの状態を監視します。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

![Azure で実行される Jenkins サーバー][0]

*このアーキテクチャ ダイアグラムを含む [Visio ファイル](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx)をダウンロードします。*

このアーキテクチャは、Azure サービスを使用したディザスター リカバリーをサポートしていますが、複数マスターやダウンタイムのない高可用性 (HA) を必要とする高度なスケールアウト シナリオには対応していません。 Azure での CI/CD パイプラインの構築の手順を示すチュートリアルなど、さまざまな Azure コンポーネントに関する全般的な情報については、「[Azure 上の Jenkins][jenkins-on-azure]」をご覧ください。

このドキュメントでは、Azure Storage を使用したビルド成果物の保持、SSO に必要なセキュリティ項目、統合可能なその他のサービス、パイプラインのスケーラビリティなど、Jenkins をサポートするために必要となる中心的な Azure 操作について説明します。 このアーキテクチャは、既存のソース管理リポジトリと連携するように設計されています。 たとえば、一般的なシナリオとして、GitHub のコミットに基づいて Jenkins ジョブを開始します。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されています。

- **リソース グループ。** [リソース グループ][rg]を使用して Azure 資産をグループ化し、有効期間、所有者などの条件に基づいて管理できるようにします。 リソース グループを使用して、Azure 資産をグループとしてデプロイおよび監視したり、リソース グループ別に請求コストを追跡したりできます。 セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。

- **Jenkins サーバー**。 [Jenkins][azure-market] をオートメーション サーバーとして実行し、Jenkins マスターとして機能させるために、仮想マシンをデプロイします。 このリファレンス アーキテクチャでは、Azure の Linux (Ubuntu 16.04 LTS) 仮想マシンにインストールされた、[Azure 上の Jenkins 用ソリューション テンプレート][solution]を使用します。 他の Jenkins 製品は、Azure Marketplace で入手できます。

  > [!NOTE]
  > Nginx を VM にインストールして、Jenkins のリバース プロキシとして動作させます。 Jenkins サーバーの SSL を有効にするように Nginx を構成できます。
  > 
  > 

- **仮想ネットワーク**。 [仮想ネットワーク][vnet]は Azure リソースを相互に接続し、論理的な分離を実現します。 このアーキテクチャでは、Jenkins サーバーは仮想ネットワークで実行されます。

- **サブネット**。 パフォーマンスに影響を与えずに、ネットワーク トラフィックを簡単に管理および分離できるように、Jenkins サーバーは[サブネット][subnet]に分離されています。

- <strong>NSG</strong>。 [ネットワーク セキュリティ グループ][nsg] (NSG) を使用して、インターネットから仮想ネットワークのサブネットへのネットワーク トラフィックを制限します。

- **Managed Disks**。 [管理ディスク][managed-disk]は、アプリケーション ストレージに使用される永続的な仮想ハード ディスク (VHD) です。また、Jenkins サーバーの状態を保持し、ディザスター リカバリーを提供します。 データ ディスクは、Azure Storage に格納されます。 高パフォーマンスを実現するために、[Premium Storage][premium] が推奨されます。

- **Azure Blob Storage**。 [Windows Azure Storage プラグイン][configure-storage]では、Azure Blob Storage を使用して、他の Jenkins ビルドと共有する作成済みのビルド成果物を保存します。

- <strong>Azure Active Directory (Azure AD)</strong>。 [Azure AD][azure-ad] はユーザー認証をサポートしているので、SSO を設定できます。 Azure AD [サービス プリンシパル][service-principal]は、[ロールベースのアクセス制御][rbac] (RBAC) を使用して、ワークフローの各ロールの承認に関するポリシーとアクセス許可を定義します。 各サービス プリンシパルは、Jenkins ジョブに関連付けられています。

- **Azure Key Vault**。 シークレットが必要なときに、Azure リソースのプロビジョニングに使用するシークレットと暗号化キーを管理するために、このアーキテクチャでは [Key Vault][key-vault] を使用します。 パイプラインのアプリケーションに関連するシークレットの保存については、Jenkins の [Azure Credentials][configure-credential] プラグインも参照してください。

- **Azure Monitoring サービス**。 このサービスは、Jenkins をホストする Azure 仮想マシンを[監視][monitor]します。 このデプロイでは、仮想マシンの状態と CPU 使用率を監視し、アラートを送信します。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、優先される特定の要件がない限り、従ってください。

### <a name="azure-ad"></a>Azure AD

Azure サブスクリプションの [Azure AD][azure-ad] テナントを使用して、Jenkins ユーザーの SSO を有効にし、Jenkins ジョブが Azure リソースにアクセスできるようにするための[サービス プリンシパル][service-principal]を設定します。

SSO の認証と承認は、Jenkins サーバーにインストールされた Azure AD プラグインによって実装されます。 SSO では、Jenkins サーバーにログオンするときに、Azure AD の組織の資格情報を使用して認証できます。 Azure AD プラグインを構成するときに、Jenkins サーバーへのユーザーの承認済みアクセスのレベルを指定できます。

Jenkins ジョブが Azure リソースにアクセスできるようにするには、Azure AD 管理者がサービス プリンシパルを作成します。 これらにより、Azure リソースへの[認証、承認されたアクセス][ad-sp]がアプリケーション (ここでは Jenkins ジョブ) に付与されます。

[RBAC][rbac] により、割り当てられたロールを通じてユーザーまたはサービス プリンシパルの Azure リソースへのアクセスがさらに明確に定義され、制御されます。 組み込みロールとカスタム ロールの両方がサポートされています。 また、ロールは、パイプラインをセキュリティで保護したり、ユーザーやエージェントの責任が適切に割り当てられ、承認されていることを保証したりするうえでも役立ちます。 さらに、Azure 資産へのアクセスを制限するように RBAC を設定することもできます。 たとえば、ユーザーが特定のリソース グループ内の資産だけを使用するように制限できます。

### <a name="storage"></a>Storage

Azure Marketplace からインストールされた Jenkins の [Windows Azure Storage プラグイン][storage-plugin]を使用して、他のビルドやテストと共有できるビルド成果物を保存します。 このプラグインを Jenkins ジョブで使用するには、Azure ストレージ アカウントを構成しておく必要があります。

### <a name="jenkins-azure-plugins"></a>Jenkins の Azure プラグイン

Azure 上の Jenkins 用ソリューション テンプレートでは、複数の Azure プラグインをインストールします。 Azure DevOps チームがこのソリューション テンプレートと次のプラグインを作成し、管理しています。各プラグインは、Azure Marketplace の他の Jenkins 製品や、オンプレミスでの Jenkins マスターのセットアップで動作します。

-   [Azure AD プラグイン][configure-azure-ad]: Jenkins サーバーが Azure AD に基づいてユーザーの SSO をサポートできるようにします。

-   [Azure VM Agents プラグイン][configure-agent]: Azure Resource Manager テンプレートを使用して、Azure 仮想マシンに Jenkins エージェントを作成します。

-   [Azure Credentials プラグイン][configure-credential]: Azure サービス プリンシパルの資格情報を Jenkins に保存できるようにします。

-   [Windows Azure Storage プラグイン][configure-storage]: ビルド成果物を [Azure Blob Storage][blob] にアップロードしたり、ビルド依存関係をダウンロードしたりします。

Azure リソースで動作する利用可能なすべての Azure プラグインの一覧を確認することをお勧めします。 最新の一覧を確認するには、[Jenkins プラグイン インデックス][index]のページにアクセスし、Azure を検索してください。 たとえば、次のプラグインをデプロイに使用できます。

-   [Azure Container Agents][container-agents]: Jenkins でコンテナーをエージェントとして実行できます。

-   [Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s): リソース構成を Kubernetes クラスターにデプロイします。

-   [Azure Container Service][acs]: Kubernetes、DC/OS と Marathon、または Docker Swarm を使用する Azure Container Service に構成をデプロイします。

-   [Azure Functions][functions]: プロジェクトを Azure Function にデプロイします。

-   [Azure App Service][app-service]: Azure App Service にデプロイします。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

非常に大規模なワークロードをサポートするために、Jenkins をスケールできます。 エラスティック ビルドの場合、Jenkins マスター サーバーでビルドを実行しないでください。 代わりに、必要に応じて弾力的にスケールインおよびスケールアウトできる Jenkins エージェントにビルド タスクをオフロードします。 エージェントをスケールする場合、次の 2 つのオプションを検討してください。

- [Azure VM Agents][vm-agent] プラグインを使用して、Azure VM で実行する Jenkins エージェントを作成します。 このプラグインを使用すると、エージェントの柔軟なスケールアウトが可能になり、異なる種類の仮想マシンを使用できます。 Azure Marketplace から別の基本イメージを選択することも、カスタム イメージを使用することもできます。 Jenkins エージェントをスケールする方法の詳細については、Jenkins ドキュメントの「[Architecting for Scale (スケールのための設計)][scale]」をご覧ください。

- [Azure Container Agents][container-agents] プラグインを使用して、[Kubernetes を使用する Azure Container Service](/azure/container-service/kubernetes/) または [Azure Container Instances](/azure/container-instances/) でコンテナーをエージェントとして実行します。

一般に、仮想マシンはコンテナーよりもスケールにコストがかかります。 ただし、スケーリングのためにコンテナーを使用するには、コンテナーでビルド プロセスを実行する必要があります。

また、Azure Storage を使用して、パイプラインの次の段階で他のビルド エージェントが使用する可能性のあるビルド成果物を共有します。

### <a name="scaling-the-jenkins-server"></a>Jenkins サーバーのスケーリング 

Jenkins サーバー VM をスケールアップまたはスケールダウンするには、VM サイズを変更します。 [Azure 上の Jenkins 用ソリューション テンプレート][azure-market]では、既定で DS2 v2 サイズ (2 つの CPU、7 GB) が指定されます。 このサイズは、小規模から中規模のチーム ワークロードに対応します。 サーバーを構築するときに別のオプションを選択して、VM サイズを変更します。 

適切なサーバー サイズの選択は、予想されるワークロードのサイズによって異なります。 Jenkins コミュニティは、要件に最も適した構成を特定する際に役立つ[選択ガイド][selection-guide]を保持しています。 Azure には、あらゆる要件に対応するために、[Linux VM のさまざまなサイズ][sizes-linux]が用意されています。 Jenkins マスターのスケーリングの詳細については、Jenkins コミュニティの[ベスト プラクティス][best-practices]を参照してください。


## <a name="availability-considerations"></a>可用性に関する考慮事項

Jenkins サーバーのコンテキストにおける可用性とは、ワークフローに関連付けられているすべての状態情報 (テスト結果など)、作成したライブラリ、または他の成果物を復旧できることを意味します。 Jenkins サーバーがダウンしたときにワークフローを復旧できるように、重要なワークフローの状態または成果物を保持しておく必要があります。 可用性の要件を評価するには、次の 2 つの一般的なメトリックを検討します。

-   目標復旧時間 (RTO) は、Jenkins なしで継続できる時間を示します。

-   目標復旧時点 (RPO) は、サービスの中断が Jenkins に影響を及ぼす場合に、失われても差し支えないデータの量を示します。

実際には、RTO と RPO は冗長性とバックアップを意味します。 可用性は、Azure の一部であるハードウェアの復旧の問題ではなく、Jenkins サーバーの状態を確実に維持することを表します。 Microsoft は 1 つの VM インスタンスに対して[サービス レベル アグリーメント][sla] (SLA) を提供します。 この SLA がアップタイムの要件を満たしていない場合は、ディザスター リカバリーを計画するか、[マルチマスター Jenkins サーバー][multi-master] デプロイ (このドキュメントでは取り上げていません) の使用を検討してください。

デプロイの手順 7 のディザスター リカバリー [スクリプト][disaster]を使用して、Jenkins サーバーの状態を保存する管理ディスクを使用する Azure ストレージ アカウントを作成することを検討します。 Jenkins がダウンした場合、この別のストレージ アカウントに保存されている状態に復元できます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

基本的な Jenkins サーバーは基本的な状態では安全でないため、次の方法を使用してサーバーのセキュリティを強化します。

-   Jenkins サーバーにログインするための安全な方法を設定します。 このアーキテクチャは HTTP とパブリック IP を使用しますが、HTTP は、既定では安全ではありません。 セキュリティで保護されたログオンに使用する [Nginx サーバーで HTTPS][nginx] を設定することを検討してください。

    > [!NOTE]
    > サーバーに SSL を追加するときは、Jenkins サブネットの NSG ルールを作成してポート 443 を開きます。 詳細については、「[Azure Portal を使用して仮想マシンへのポートを開く方法][port443]」をご覧ください。
    > 

-   Jenkins の構成がクロスサイト リクエスト フォージェリを防いでいることを確認します ([Manage Jenkins]\(Jenkins の管理\) \> [Configure Global Security]\(グローバル セキュリティの構成\))。 これは Microsoft Jenkins Server の既定値です。

-   [Matrix Authorization Strategy プラグイン][matrix]を使用して、Jenkins ダッシュボードへの読み取り専用アクセスを構成します。

-   [Azure Credentials][configure-credential] プラグインをインストールし、Key Vault を使用して、Azure 資産、パイプライン内のエージェント、およびサード パーティ コンポーネントのシークレットを処理します。

-   RBAC を使用して、ジョブの実行に必要なサービス プリンシパルのアクセス権を最小限に抑えます。 これは、承認されていないジョブによる被害の範囲を制限するうえで役立ちます。

多くの場合、Jenkins ジョブは、承認が必要な Azure サービス (Azure Container Service など) にアクセスするためにシークレットを必要とします。 [Azure Credentials プラグイン][configure-credential]と共に [Key Vault][key-vault] を使用して、これらのシークレットを安全に管理します。 Key Vault を使用して、サービス プリンシパルの資格情報、パスワード、トークン、その他のシークレットを保存します。

Azure リソースのセキュリティの状態を一元的に表示して把握するには、[Azure Security Center][security-center] を使用します。 Security Center は、潜在的なセキュリティ上の問題を監視し、デプロイのセキュリティの正常性を包括的に示します。 セキュリティ センターは、Azure サブスクリプションごとに構成されます。 「[Azure Security Center クイック スタート ガイド][quick-start]」の説明に従って、セキュリティ データの収集を有効にします。 データ収集を有効にすると、Security Center は、そのサブスクリプションに作成されているすべての仮想マシンを自動的にスキャンします。

Jenkins サーバーには独自のユーザー管理システムがあります。Jenkins コミュニティでは、[Azure 上の Jenkins インスタンスのセキュリティ保護][secure-jenkins]に関するベスト プラクティスを提供しています。 Azure 上の Jenkins 用ソリューション テンプレートは、これらのベストプラクティスを実装しています。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

リソース グループを使用して、デプロイされている Azure リソースを整理します。 運用環境と開発/テスト環境を別々のリソース グループに配置することで、各環境のリソースを監視し、リソース グループ別に請求コストをまとめることができます。 セットとしてリソースを削除することもできます。これはテスト デプロイの場合に便利です。

Azure には、インフラストラクチャ全体の[監視と診断][monitoring-diag]のための機能がいくつか用意されています。 CPU 使用率を監視するために、このアーキテクチャでは Azure Monitor をデプロイします。 たとえば、Azure Monitor を使用して CPU 使用率を監視し、CPU 使用率が 80% を超えた場合に通知を送信できます  (CPU 使用率が高い場合、Jenkins サーバー VM をスケールアップした方がよいことを示しています)。また、VM に障害が発生した場合や VM が使用できなくなった場合に、指定されたユーザーに通知することもできます。

## <a name="communities"></a>コミュニティ

コミュニティは質問に答え、デプロイを正常に完了できるよう支援します。 以下、具体例に沿って説明します。

-   [Jenkins Community Blog](https://jenkins.io/node/)
-   [Azure フォーラム](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

Jenkins コミュニティが提供するベスト プラクティスについては、「[Jenkins Best Practices (Jenkins のベスト プラクティス)][jenkins-best]」をご覧ください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

このアーキテクチャをデプロイするには、下記の手順に従って、[Azure 上の Jenkins 用ソリューション テンプレート][azure-market]をインストールし、下記の手順の監視とディザスター リカバリーを設定するスクリプトをインストールします。

### <a name="prerequisites"></a>前提条件

- このリファレンス アーキテクチャには、Azure サブスクリプションが必要です。 
- Azure サービス プリンシパルを作成するには、デプロイ済みの Jenkins サーバーに関連付けられている Azure AD テナントの管理者権限が必要です。
- 下記の手順は、Jenkins 管理者が、少なくとも共同作成者権限を持つ Azure ユーザーでもあることを前提としています。

### <a name="step-1-deploy-the-jenkins-server"></a>手順 1: Jenkins サーバーをデプロイする

1.  Web ブラウザーで [Jenkins の Azure Marketplace イメージ][azure-market]を開き、ページの左側の **[今すぐ入手する]** を選択します。

2.  料金の詳細を確認して **[続行]** を選択し、**[作成]** を選択して、Azure Portal で Jenkins サーバーを構成します。

詳細については、「[Azure Portal から Azure Linux VM に Jenkins サーバーを作成する][create-jenkins]」をご覧ください。 このリファレンス アーキテクチャでは、管理者ログオンでサーバーを起動して実行するだけで十分です。 その後、他のさまざまなサービスを使用するためにサーバーをプロビジョニングできます。

### <a name="step-2-set-up-sso"></a>手順 2: SSO を設定する

この手順は Jenkins 管理者が実行します。管理者には、サブスクリプションの Azure AD ディレクトリのユーザー アカウントも必要であり、共同作成者ロールが割り当てられている必要があります。

Jenkins サーバーで Jenkins Update Center の [Azure AD プラグイン][configure-azure-ad]を使用し、指示に従って SSO を設定します。

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>手順 3: Azure VM Agents プラグインを使用して Jenkins サーバーをプロビジョニングする

この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Azure VM Agents プラグインを設定します。

[こちらの手順に従って、プラグインを構成します][configure-agent]。 プラグインのサービス プリンシパルの設定に関するチュートリアルについては、「[Scale your Jenkins deployments to meet demand with Azure VM agents (要求を満たすために Azure VM Agents を使用して Jenkins デプロイをスケールする)][scale-agent]」をご覧ください。

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>手順 4: Azure Storage を使用して Jenkins サーバーをプロビジョニングする

この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Windows Azure Storage プラグインを設定します。

[こちらの手順に従って、プラグインを構成します][configure-storage]。

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>手順 5: Azure Credentials プラグインを使用して Jenkins サーバーをプロビジョニングする

この手順は Jenkins 管理者が実行します。管理者は、既にインストールされている Azure Credentials プラグインを設定します。

[こちらの手順に従って、プラグインを構成します][configure-credential]。

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>手順 6: Azure Monitor サービスで監視するために Jenkins サーバーをプロビジョニングする

Jenkins サーバーの監視を設定するには、「[Azure Monitor での Azure サービス メトリック アラートの作成][create-metric]」の手順に従います。

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>手順 7: ディザスター リカバリー用の管理ディスクを使用して Jenkins サーバーをプロビジョニングする

Microsoft Jenkins 製品グループは、Jenkins の状態を保存するために使用する管理ディスクを作成するディザスター リカバリー スクリプトを作成しました。 サーバーがダウンした場合、最新の状態に復元できます。

[GitHub][disaster] からディザスター リカバリー スクリプトをダウンロードして実行します。

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
