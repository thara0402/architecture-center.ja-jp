---
title: コンテナー ベースのワークロード用の CI/CD パイプライン
titleSuffix: Azure Example Scenarios
description: Jenkins、Azure Container Registry、Azure Kubernetes Service、Cosmos DB、Grafana を使用して Node.js Web アプリの DevOps パイプラインを構築します。
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: 7cf05b79b8984ff4bcad62df44058b0989bc124a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481968"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>コンテナー ベースのワークロード用の CI/CD パイプライン

このシナリオ例は、コンテナーと DevOps ワークフローを使用してアプリケーション開発を最新化することを求めている企業に適用されます。 このシナリオでは、Jenkins によって Node.js Web アプリが構築され、Azure Container Registry と Azure Kubernetes Service にデプロイされます。 グローバル分散データベース層には、Azure Cosmos DB が使用されます。 アプリケーションのパフォーマンスの監視とトラブルシューティングのために、Azure Monitor が Grafana インスタンスおよびダッシュボードと統合されています。

アプリケーションのシナリオ例には、自動化された開発環境の提供、新しいコードのコミットの検証、ステージング環境または運用環境への新しいデプロイのプッシュが含まれます。 従来、企業はアプリケーションや更新プログラムを手動でビルドしてコンパイルし、大規模なモノリシック コード ベースを維持する必要がありました。 継続的インテグレーション (CI) と継続的配置 (CD) を使用する最新のアプリケーション開発手法により、サービスの構築、テスト、デプロイを迅速化できます。 この最新の手法により、アプリケーションや更新プログラムを顧客により迅速にリリースし、変化するビジネス要求により俊敏に対応することができます。

Azure Kubernetes Service、Container Registry、Cosmos DB などの Azure サービスを使用することで、企業は最新のアプリケーション開発技術やツールを使って、高可用性の実装プロセスを簡素化できます。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- アプリケーション開発プラクティスを、コンテナー ベースのマイクロサービス手法に最新化する。
- アプリケーションの開発とデプロイのライフサイクルを加速化する。
- 検証のために、テスト環境または受け入れ環境へのデプロイを自動化する。

## <a name="architecture"></a>アーキテクチャ

![Jenkins、Azure Container Registry、Azure Kubernetes Service を使用した DevOps シナリオに関与する Azure コンポーネントのアーキテクチャの概要][architecture]

このシナリオでは、Node.js Web アプリケーションおよびデータベース バックエンドの DevOps パイプラインに対応できます。 このシナリオのデータ フローは次のとおりです。

1. 開発者が、Node.js Web アプリケーションのソース コードに変更を加えます。
2. コードの変更がソース管理リポジトリ (GitHub など) にコミットされます。
3. 継続的インテグレーション (CI) プロセスを開始するために、GitHub Webhook が Jenkins プロジェクトのビルドをトリガーします。
4. Jenkins ビルド ジョブが、Azure Kubernetes Service の動的ビルド エージェントを使用してコンテナーのビルド プロセスを実行します。
5. ソース管理のコードからコンテナー イメージが作成され、Azure Container Registry にプッシュされます。
6. 継続的配置 (CD) を通じて、Jenkins がこの最新のコンテナー イメージを Kubernetes クラスターに展開します。
7. Node.js Web アプリケーションは、Cosmos DB をバックエンドとして使用します。 Cosmos DB と Azure Kubernetes Service が、Azure Monitor にメトリックを報告します。
8. Grafana インスタンスが、Azure Monitor のデータに基づいて、アプリケーションのパフォーマンスのビジュアル ダッシュボードを提供します。

### <a name="components"></a>コンポーネント

- [Jenkins][jenkins]: オープンソースのオートメーション サーバーです。Azure サービスと統合することで、継続的インテグレーション (CI) と継続的配置 (CD) が可能になります。 このシナリオでは、Jenkins によって、ソース管理へのコミットに基づいて新しいコンテナー イメージの作成が調整され、それらのイメージが Azure Container Registry にプッシュされた後、Azure Kubernetes Service 内でアプリケーション インスタンスが更新されます。
- [Azure Linux Virtual Machines][docs-virtual-machines]: Jenkins インスタンスと Grafana インスタンスを実行するために使用される IaaS プラットフォームです。
- [Azure Container Registry][docs-acr]: Azure Kubernetes Service クラスターで使用されるコンテナー イメージを保存し、管理します。 イメージは安全に保存されています。Azure プラットフォームによって他のリージョンにレプリケートすることで、デプロイ時間を短縮できます。
- [Azure Kubernetes Service][docs-aks]: コンテナー オーケストレーションの専門知識がなくても、コンテナー化されたアプリケーションをデプロイし、管理できるマネージド Kubernetes プラットフォームです。 ホストされた Kubernetes サービスとして、Azure は正常性監視やメンテナンスなどの重要なタスクを自動的に処理します。
- [Azure Cosmos DB][docs-cosmos-db]: ニーズに合わせて、さまざまなデータベースや整合性モデルの中から選択できる、グローバル分散型のマルチモデル データベースです。 Cosmos DB を使用すると、データをグローバルにレプリケートできます。デプロイして構成するクラスター管理コンポーネントやレプリケーション コンポーネントはありません。
- [Azure Monitor][docs-azure-monitor]: パフォーマンスの追跡、セキュリティの維持、傾向の把握に役立ちます。 Monitor によって取得されたメトリックは、他のリソースやツール (Grafana など) で使用できます。
- [Grafana][grafana]: メトリックのクエリの実行、視覚化、アラートの作成を行い、メトリックを理解するためのオープンソース ソリューションです。 Azure Monitor のデータ ソース プラグインにより、Grafana では、Azure Kubernetes Service で実行され、Cosmos DB を使用するアプリケーションのパフォーマンスを監視するビジュアル ダッシュボードを作成できます。

### <a name="alternatives"></a>代替手段

- [Azure Pipelines][azure-pipelines] を使用して、アプリの継続的インテグレーション (CI)、テスト、および継続的配置 (CD) のパイプラインを実装できます。
- クラスターをより細かく制御する必要がある場合は、マネージド サービスを介してではなく、Azure VM 上で [Kubernetes][kubernetes] を直接実行できます。
- [Service Fabric][service-fabric] は、AKS の代わりに使用できる別の代替コンテナー オーケストレーターです。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

アプリケーションのパフォーマンスを監視し、問題を報告するために、このシナリオでは Azure Monitor と Grafana を組み合わせてビジュアル ダッシュボードを提供します。 これらのツールを使用すると、コードの更新が必要になる可能性のあるパフォーマンスの問題を監視し、トラブルシューティングを行うことができます。コードの更新は、すべて CI/CD パイプラインを使用して展開できます。

Azure Kubernetes Service クラスターに含まれるロード バランサーは、アプリケーションを実行する 1 つ以上のコンテナー (ポッド) にアプリケーション トラフィックを分散します。 コンテナー化されたアプリケーションを Kubernetes で実行するこのアプローチにより、可用性の高いインフラストラクチャが顧客に提供されます。

可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

Azure Kubernetes Service を使用すると、アプリケーションの要求に応じてクラスタ ノードの数をスケーリングできます。 アプリケーションの増加に伴って、サービスを実行する Kubernetes ノードの数をスケールアウトできます。

アプリケーション データは、グローバルにスケーリングできるグローバル分散型のマルチモデル データベースである Azure Cosmos DB に格納されます。 Cosmos DB は、従来のデータベース コンポーネントと同様に、インフラストラクチャをスケーリングする必要性を排除します。また、顧客の要求に応じて Cosmos DB をグローバルにレプリケートすることもできます。

スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

攻撃フットプリントを最小限に抑えるために、このシナリオでは Jenkins VM インスタンスが HTTP 経由で公開されることはありません。 Jenkins を操作する必要がある管理タスクについては、ローカル コンピューターから SSH トンネルを使用して、セキュリティで保護されたリモート接続を作成します。 Jenkins および Grafana VM インスタンスには、SSH 公開キー認証のみを使用できます。 パスワード ベースのログインは無効になります。 詳細については、「[Azure で Jenkins サーバーを実行する](../../reference-architectures/jenkins/index.md)」をご覧ください。

資格情報とアクセス許可を分離するために、このシナリオでは専用の Azure Active Directory (AD) サービス プリンシパルを使用します。 このサービス プリンシパルの資格情報は、セキュリティで保護された資格情報オブジェクトとして Jenkins に保存されているため、スクリプトやビルド パイプライン内で直接公開されたり表示されたりすることはありません。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

このシナリオでは、アプリケーションに Azure Kubernetes Service を使用します。 Kubernetes には、コンテナー (ポッド) を監視し、問題が発生した場合に再起動する回復性コンポーネントが組み込まれています。 実行中の複数の Kubernetes ノードと組み合わせることで、アプリケーションは使用できなくなっているポッドやノードを許容できます。

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

### <a name="prerequisites"></a>前提条件

- 既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

- SSH 公開キー ペアが必要です。 公開キー ペアの作成手順については、[Linux VM 用の SSH キー ペアの作成と使用][sshkeydocs]に関する記事をご覧ください。

- サービスとリソースの認証用に Azure Active Directory (AD) サービス プリンシパルが必要です。 必要に応じて、[az ad sp create-for-rbac][createsp] を使用してサービス プリンシパルを作成できます。

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    このコマンドの出力の *appId* と *password* をメモしておきます。 これらの値は、シナリオをデプロイするときにテンプレートに入力します。

### <a name="walk-through"></a>チュートリアル

Azure Resource Manager テンプレートを使用してこのシナリオをデプロイするには、次の手順を実行します。

<!-- markdownlint-disable MD033 -->

1. **[Deploy to Azure]\(Azure にデプロイ\)** をクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure portal でテンプレートのデプロイが開くまで待ってから、次の手順を実行します。
   - リソース グループを**新規作成**し、テキスト ボックスに名前 (例: *myAKSDevOpsScenario*) を指定します。
   - **[場所]** ドロップダウン ボックスでリージョンを選択します。
   - `az ad sp create-for-rbac` コマンドで出力されたサービス プリンシパルのアプリ ID とパスワードを入力します。
   - Jenkins インスタンスと Grafana コンソールのユーザー名とセキュリティで保護されたパスワードを入力します。
   - Linux VM へのログインをセキュリティで保護する SSH キーを提供します。
   - 使用条件を確認し、**[上記の使用条件に同意する]** をオンにします。
   - **[購入]** をクリックします。

<!-- markdownlint-enable MD033 -->

デプロイが完了するまでに 15 から 20 分かかることがあります。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

保存するコンテナー イメージの数とアプリケーションを実行する Kubernetes ノードの数に基づいて、3 つのサンプル コスト プロファイルが用意されています。

- [Small][small-pricing]: この価格例は、1 か月あたり 1,000 個のコンテナー ビルドに対応します。
- [Medium][medium-pricing]: この価格例は、1 か月あたり 100,000 個のコンテナー ビルドに対応します。
- [Large][large-pricing]: この価格例は、1 か月あたり 1,000,000 個のコンテナー ビルドに対応します。

## <a name="related-resources"></a>関連リソース

このシナリオでは、Azure Container Registry と Azure Kubernetes Service を使用して、コンテナー ベースのアプリケーションを格納し、実行しました。 オーケストレーション コンポーネントをプロビジョニングしなくても、Azure Container Instances を使用して、コンテナー ベースのアプリケーションを実行することもできます。 詳細については、[Azure Container Instances の概要][docs-aci]に関する記事をご覧ください。

<!-- links -->
[architecture]: ./media/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[docs-aci]: /azure/container-instances/container-instances-overview
[docs-acr]: /azure/container-registry/container-registry-intro
[docs-aks]: /azure/aks/intro-kubernetes
[docs-azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[docs-cosmos-db]: /azure/cosmos-db/introduction
[docs-virtual-machines]: /azure/virtual-machines/linux/overview
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