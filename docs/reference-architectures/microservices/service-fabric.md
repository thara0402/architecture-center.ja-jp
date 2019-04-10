---
title: Azure Service Fabric でのマイクロサービス アーキテクチャ
description: Azure Service Fabric にマイクロサービス アーキテクチャをデプロイします
author: PageWriter-MSFT
ms.date: 3/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 228e3242ce9f1ca45bc15c5b34f770bfc092dd43
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244343"
---
# <a name="microservices-architecture-on-azure-service-fabric"></a>Azure Service Fabric でのマイクロサービス アーキテクチャ

この参照アーキテクチャでは、Azure Service Fabric にデプロイされるマイクロサービス アーキテクチャを示します。 それには、ほとんどのデプロイの出発点にすることができる基本的なクラスター構成が示されています。

![Service Fabric の参照アーキテクチャ](./_images/ra-sf-arch.png)

> [!NOTE]
> この記事では、Service Fabric の [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction) プログラミング モデルに注目します。 Service Fabric を使用した[コンテナー](/azure/service-fabric/service-fabric-containers-overview)のデプロイと管理については、この記事では説明されていません。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。 他の用語については、「[Service Fabric の用語の概要](/azure/service-fabric/service-fabric-technical-overview)」をご覧ください。

**Service Fabric クラスター**。 マイクロサービスをデプロイして管理する、ネットワークで接続された一連の仮想マシン (VM)。

**仮想マシン スケール セット**。 仮想マシン スケール セットを使用すると、負荷分散と自動スケーリングが行われる同一の VM のグループを作成して管理できます。 障害ドメインとアップグレード ドメインも提供されます。

**ノード**。 ノードは、Service Fabric クラスターに属する VM です。

**ノード タイプ**。 ノード タイプは、ノードのコレクションをデプロイする仮想マシン スケール セットを表します。 Service Fabric クラスターには、少なくとも 1 つのノード タイプがあります。 クラスターに複数のノード タイプがある場合は、その 1 つを[プライマリ ノード タイプ](/azure/service-fabric/service-fabric-cluster-capacity#primary-node-type)として宣言する必要があります。 クラスターのプライマリ ノード タイプでは、[Service Fabric システム サービス](/azure/service-fabric/service-fabric-technical-overview#system-services)が実行されます。 これらのサービスでは、Service Fabric のプラットフォーム機能が提供されます。 プライマリ ノード タイプは、クラスターに対する[シード ノード](/azure/service-fabric/service-fabric-disaster-recovery#random-failures-leading-to-cluster-failures)としても機能します。シード ノードは、基になるクラスターの可用性を維持するノードです。 独自のサービスを実行するには、[追加のノード タイプ](/azure/service-fabric/service-fabric-cluster-capacity#non-primary-node-type)を構成します。

**サービス**。 サービスでは、他のサービスから独立して開始および実行できる、スタンドアロンの機能が実行されます。 サービスのインスタンスは、クラスター内のノードにデプロイされます。 Service Fabric のサービスには、2 つの種類あります。

- **ステートレス サービス**。 ステートレス サービスでは、サービス内の状態は保持されません。 状態の永続性が必要な場合は、Azure Cosmos DB などの外部ストアに状態を書き込み、取得します。
- **ステートフル サービス**。 [サービスの状態](/azure/service-fabric/service-fabric-concepts-state)は、サービス自体の内部に保持されます。 ほとんどのステートフル サービスでは、Service Fabric の[リライアブル コレクション](/azure/service-fabric/service-fabric-reliable-services-reliable-collections)によってこれが実装されます。

**Service Fabric Explorer**。 [Service Fabric Explorer][sfx] は、Service Fabric クラスターを検査および管理するためのオープン ソース ツールです。

**Azure Pipelines**。 [Pipelines](/azure/devops/pipelines/?view=azure-devops) は [Azure DevOps Services](/azure/devops/index?view=azure-devops) の一部であり、自動化されたビルド、テスト、およびデプロイを実行します。 Jenkins などのサードパーティ CI/CD ソリューションも使用できます。

**Azure Monitor**。 [Azure Monitor](/azure/azure-monitor/) では、ソリューション内の Azure サービスのプラットフォーム メトリックやアプリケーション テレメトリなど、メトリックとログが収集されて格納されます。 このデータは、アプリケーションを監視したり、アラートやダッシュボードを設定したり、障害の根本原因分析を実行したりするために使用します。 Azure Monitor は、コントローラー、ノード、およびコンテナーからのメトリックや、コンテナー ログおよびマスター ノード ログを収集するために Service Fabric と統合します。

**Azure Key Vault**。 接続文字列など、マイクロサービスで使用されるすべてのアプリケーション シークレットを格納するには、[Key Vault](/azure/key-vault/) を使用します。

**Azure API Management**。 このアーキテクチャの [API Management](/azure/api-management/api-management-key-concepts) は、クライアントから要求を受け取って独自のサービスにルーティングする API ゲートウェイとして機能します。

## <a name="design-considerations"></a>設計上の考慮事項

この参照アーキテクチャでは、[マイクロサービス アーキテクチャ](../../guide/architecture-styles/microservices.md)に重点が置かれています。 マイクロサービスは、独立してバージョン管理されるコードの小さな単位です。 サービス検出メカニズムによって検出可能であり、API 経由で他のサービスと通信できます。 各サービスは自己完結型であり、1 つのビジネス機能を実装している必要があります。 アプリケーション ドメインをマイクロサービスに分解する方法について詳しくは、「[Using domain analysis to model microservices (ドメイン分析を使用してマイクロサービスをモデル化する)](../../microservices/model/domain-analysis.md)」をご覧ください。

Service Fabric では、マイクロサービスを効率的に構築、デプロイ、およびアップグレードするためのインフラストラクチャが提供されています。 また、自動スケーリング、状態の管理、正常性の監視、および障害発生時のサービスの再起動に対するオプションも提供されています。

Service Fabric は、アプリケーションがマイクロサービスのコレクションであるアプリケーション モデルに従います。 アプリケーションは[アプリケーション マニフェスト](/azure/service-fabric/service-fabric-application-and-service-manifests) ファイルに記述されており、このファイルでは、そのアプリケーションに含まれるさまざまな種類のサービスと、独立したサービス パッケージへのポインターが定義されています。 アプリケーション パッケージには、通常、サービスによって使用される特定の設定に対するオーバーライドとして機能するパラメーターも含まれています。 サービス パッケージごとにマニフェスト ファイルがあり、そのサービスのバイナリ、構成ファイル、読み取り専用データなど、そのサービスを実行するために必要な物理ファイルとフォルダーが記述されています。 サービスとアプリケーションは、個別にバージョン管理され、アップグレード可能です。

アプリケーション マニフェストでは、必要に応じて、アプリケーションのインスタンスが作成されるときに自動的にプロビジョニングされるサービスを記述できます。 これらは既定のサービスと呼ばれます。 この場合、アプリケーション マニフェストでは、サービスの名前、インスタンスの数、セキュリティ/分離ポリシー、配置の制約など、これらのサービスに必要な作成方法も記述されます。

> [!NOTE]
> 独自のサービスの有効期間を制御する場合は、既定のサービスを使用しないでください。 既定のサービスは、アプリケーションが作成されると作成され、アプリケーションが実行されている限り実行されます。

Service Fabric に関して詳しくは、「[Service Fabric に興味をお持ちでしょうか](/azure/service-fabric/service-fabric-content-roadmap)」をご覧ください。

### <a name="choose-an-application-to-service-packaging-model"></a>アプリケーションからサービスへのパッケージ化モデルを選択する

マイクロサービスの基本思想は、各サービスを独立してデプロイできる、というものです。 Service Fabric では、すべてのサービスを 1 つのアプリケーション パッケージにグループ化した場合、1 つのサービスのアップグレードが失敗すると、アプリケーション全体のアップグレードがロールバックされ、他のサービスもアップグレードされなくなります。

そのため、マイクロサービス アーキテクチャでは、複数のアプリケーション パッケージを使用することをお勧めします。 1 つのアプリケーションの種類には、1 つまたは密接に関連する複数のサービスの種類を格納します。 実行期間が同じで、同時に更新する、ライフサイクルを同じにする、または依存関係や構成などのリソースを共有する必要があるサービスのセットがある場合は、それらのサービスの種類を同じアプリケーションの種類に格納します。

### <a name="service-fabric-programming-models"></a>Service Fabric のプログラミング モデル

Service Fabric アプリケーションにマイクロサービスを追加するときは、高可用性で高信頼性にする必要がある状態またはデータがあるかどうかを判断します。 そうである場合、データを外部に格納できますか、それともデータはサービスの一部として含まれますか。 外部ストレージにデータを格納する必要がない場合、またはデータを格納したくない場合は、ステートレス サービスを選択します。 サービスの一部として状態またはデータを維持する場合 (たとえば、そのデータがコードに近いメモリに存在する必要がある場合)、または外部ストアへの依存関係を許容できない場合は、ステートフル サービスの選択を検討します。

Service Fabric で実行する既存のコードがある場合は、ゲスト実行可能ファイルとして実行できます。ゲスト実行可能ファイルは、サービスとして実行される任意の実行可能ファイルです。 または、デプロイに必要なすべての依存関係が含まれるコンテナーに、実行可能ファイルをパッケージ化することができます。 Service Fabric では、コンテナーとゲスト実行可能ファイルの両方が、ステートレス サービスとしてモデル化されます。 モデルの選択に関するガイダンスについては、「[Service Fabric プログラミング モデルの概要](/azure/service-fabric/service-fabric-choose-framework)」をご覧ください。

ゲスト実行可能ファイルの場合は、それが実行される環境を自分で管理する必要があります。 たとえば、ゲスト実行可能ファイルで Python が必要であるとします。 実行可能ファイルが自己完結型でない場合、必要なバージョンの Python が環境にプレインストールされていることを、自分で確認する必要があります。 Service Fabric では、環境の管理は行われません。 Azure では、カスタム仮想マシン イメージや拡張機能など、環境を設定する複数のメカニズムが提供されています。

詳細については、次を参照してください。

- [アプリケーションのパッケージ化](/azure/service-fabric/service-fabric-package-apps)
- [既存の実行可能ファイルのパッケージ化と Service Fabric へのデプロイ](/azure/service-fabric/service-fabric-deploy-existing-app)

### <a name="api-gateway"></a>API ゲートウェイ

[API ゲートウェイ](../..//microservices/design/gateway.md) (イングレス) は、外部クライアントとマイクロサービスの間に配置されます。 これは、要求をクライアントからマイクロサービスにルーティングするリバース プロキシとして機能します。 さらに、認証、SSL 終了、レート制限などのさまざまな横断的タスクを実行できます。

ほとんどのシナリオでは Azure API Management が推奨されますが、[Træfik](https://docs.traefik.io/) は人気のあるオープン ソースの代替手段です。 どちらのテクノロジのオプションも、Service Fabric と統合されます。

- API Management では、パブリック IP アドレスが公開され、サービスにトラフィックがルーティングされします。 Service Fabric クラスターと同じ仮想ネットワーク内の専用サブネットで実行されます。  プライベート IP アドレスを使用してロード バランサー経由で公開されるノード タイプのサービスにアクセスできます。 このオプションは、API Management の Premium レベルと Developer レベルでのみ使用できます。 運用ワークロードの場合は、Premium レベルを使用します。 価格の情報については、「[API Management の価格](https://azure.microsoft.com/pricing/details/api-management/)」をご覧ください。 詳しくは、「[Azure Service Fabric と API Management の概要](/azure/service-fabric/service-fabric-api-management-overview)」をご覧ください。
- Træfik では、ルーティング、トレース、ログ、メトリックなどの機能がサポートされています。 Træfik は、Service Fabric クラスターでステートレス サービスとして実行されます。 サービスのバージョン管理は、ルーティングによってサポートできます。 サービス イングレス用に、およびクラスター内のリバース プロキシとして、Træfik を設定する方法については、「[Azure Service Fabric Provider (Azure Service Fabric プロバイダー)](https://docs.traefik.io/configuration/backends/servicefabric/)」をご覧ください。 Service Fabric での Træfik の使用について詳しくは、「[Intelligent routing on Service Fabric with Træfik (Træfik による Service Fabric でのインテリジェントなルーティング)](https://blogs.msdn.microsoft.com/azureservicefabric/2018/04/05/intelligent-routing-on-service-fabric-with-traefik/)」 (ブログ投稿) をご覧ください。

API Managementment のその他のオプションとしては、[Azure Application Gateway](/azure/application-gateway/) と [Azure Front Door](/azure/frontdoor/) があります。 これらのサービスを API Management と組み合わせて使用し、ルーティング、SSL 終了、ファイアウォールなどのタスクを実行できます。

### <a name="interservice-communication"></a>サービス間の通信

サービス間の通信を容易にするには、通信プロトコルとして HTTP を使用することを検討します。 ほとんどのシナリオのベースラインとして、サービスの検出に[リバース プロキシ サービス](/azure/service-fabric/service-fabric-reverseproxy)を使うことをお勧めします。

- 通信プロトコル。 マイクロサービス アーキテクチャでは、サービス同士が実行時に最小限の結合で通信する必要があります。 言語に依存しないで通信を行う手段としては HTTP が業界標準であり、さまざまな言語で利用できる広範なツールと HTTP サーバーのすべてが、Service Fabric でサポートされています。 したがって、Service Fabric に組み込まれているサービス リモート処理の代わりに HTTP を使用することが、ほとんどのワークロードで推奨されます。
- サービス検出。 クラスター内の他のサービスと通信するには、クライアント サービスで対象サービスの現在の場所を解決する必要があります。 Service fabric では、サービスはノード間を移動できるため、サービス エンドポイントは動的に変化します。 古いエンドポイントへの接続を防ぐには、Service Fabric の Naming Service を使用して、更新されたエンドポイント情報を取得できます。 ただし、Service Fabric では、ネーム サービスを抽象化する組み込みの[リバース プロキシ サービス](/azure/service-fabric/service-fabric-reverseproxy)も提供されています。 このオプションの方が簡単に使用でき、コードがシンプルになります。

サービス間通信用のオプションとしては、他に次のものがあります。

- 高度なルーティング用の [Træfik](https://docs.traefik.io/)。
- DNS を使用する必要があるサービスとの互換性シナリオ用の [DNS](/azure/dns/)。
- [ServicePartitionClient&lt;TCommunicationClient&gt;](/dotnet/api/microsoft.servicefabric.services.communication.client.servicepartitionclient-1?view=azure-dotnet) クラス。 サービス エンドポイントをキャッシュするクラスであり、仲介者やカスタム プロトコルを必要とせずにサービス間で直接呼び出しが行われるので、パフォーマンスが向上します。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

Service Fabric では、次のクラスター エンティティのスケーリングがサポートされています。

- 各ノード タイプのノード数のスケーリング。
- サービスのスケーリング。

このセクションでは、自動スケーリングに注目します。 それが適切な状況では、手動スケーリングを選択できます。 たとえば、インスタンスの数を設定するために手動による介入が必要な状況です。  

### <a name="initial-cluster-configuration-for-scalability"></a>スケーラビリティのための初期クラスター構成

Service Fabric クラスターを作成するときに、セキュリティとスケーラビリティのニーズに基づいて、ノード タイプをプロビジョニングします。 各ノード タイプは 1 つの仮想マシン スケール セットにマップされ、独立してスケーリングできます。

- スケーラビリティまたはリソースの要件が異なるサービスのグループごとに、ノード タイプを作成します。 最初に、Service Fabric システム サービス用のノード タイプをプロビジョニングします (これが [プライマリ ノード タイプ](/azure/service-fabric/service-fabric-cluster-capacity#primary-node-type)になります)。 その後、パブリック サービスまたはフロントエンド サービスを実行するための別のノード タイプを作成し、バックエンドのプライベート サービスまたは分離サービス用の他のノード タイプを必要に応じて作成します。 サービスが目的のノード タイプにのみデプロイされるように、[配置の制約](/azure/service-fabric/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies)を指定します。
- 各ノード タイプの持続性層を指定します。 持続性層は、仮想マシン スケール セットの更新とメンテナンス操作に対して Service Fabric が影響を与える機能を表します。 運用ワークロードでは、シルバー以上の持続性層を選択します。 各層については、「[クラスターの耐久性の特徴](/azure/service-fabric/service-fabric-cluster-capacity#the-durability-characteristics-of-the-cluster)」をご覧ください。
- ブロンズ持続性層を使用する場合は、特定の操作で手動手順が必要になります。  ブロンズ持続性層のノード タイプでは、スケールインの間に追加の手順が必要です。 スケーリング操作について詳しくは、[こちらのガイド](/azure/service-fabric/service-fabric-cluster-scale-up-down)をご覧ください。

### <a name="scaling-nodes"></a>ノードのスケーリング

Service Fabric では、スケールインとスケールアウトに対する自動スケーリングがサポートされています。ノード タイプごとに個別に自動スケーリングを構成できます。

各ノード タイプは、最大 100 個のノードを持つことができます。 少数のノード セットで開始し、負荷に応じてノードを追加します。 1 つのノード タイプで必要なノードが 100 個を超える場合は、ノード タイプをさらに追加する必要があります。 詳しくは、「[Service Fabric クラスターの容量計画に関する考慮事項](/azure/service-fabric/service-fabric-cluster-capacity)」をご覧ください。 仮想マシン スケール セットは瞬時にはスケーリングしないので、自動スケーリングの規則を設定するときは、そのことを考慮する必要があります。

自動スケールインをサポートするには、シルバーまたはゴールドの持続性層を持つようにノード タイプを構成します。 これにより、Service Fabric でサービスの再配置が完了するまでスケールインがデプロイされ、仮想マシン スケール セットから Service Fabric に VM が単に一時的にダウンしたのではなく削除されたことが通知されます。

ノード/クラスター レベルでのスケーリングについて詳しくは、「[Azure Service Fabric クラスターのスケーリング](/azure/service-fabric/service-fabric-cluster-scaling)」をご覧ください。

### <a name="scaling-services"></a>サービスのスケーリング

ステートレス サービスとステートフル サービスでは、適用されるスケーリングのアプローチが異なります。

#### <a name="autoscaling-for-stateless-services"></a>ステートレス サービスの場合の自動スケーリング

- パーティションの平均負荷トリガーを使用します。 このトリガーでは、スケーリング ポリシーで指定されている負荷のしきい値に基づいて、サービスがスケールインまたはスケールアウトされるタイミングが決定されます。 トリガーをチェックする頻度も設定できます。 「[インスタンス ベースのスケーリングで使用するパーティションの平均負荷トリガー](/azure/service-fabric/service-fabric-cluster-resource-manager-autoscaling#average-partition-load-trigger-with-instance-based-scaling)」をご覧ください。 これにより、使用可能なノードの数までスケールアップできます。
- サービス マニフェストで **InstanceCount** を -1 に設定すると、すべてのノードでサービスのインスタンスを実行するよう Service Fabric に指示されます。 この方法では、クラスターのスケーリングに合わせてサービスを動的にスケーリングできます。 クラスターのノードの数が変化すると、Service Fabric はそれと一致するようにサービス インスタンスを自動的に作成および削除します。

> [!NOTE]
> 場合によっては、サービスを手動でスケーリングしたいことがあります。  たとえば、Event Hubs から読み取るサービスがある場合、パーティションへの同時アクセスを回避するため、専用のインスタンスで各イベント ハブ パーティションからの読み取りを行うことがあります。  

#### <a name="scaling-for-stateful-services"></a>ステートフル サービスのスケーリング

ステートフル サービスでのスケーリングは、パーティションの数、各パーティションのサイズ、および特定のマシンで実行されるパーティション/レプリカの数によって制御されます。

- パーティション分割されたサービスを作成する場合は、小さいサイズのパーティションを多数作成することをお勧めします。 ノードが追加される場合、Service Fabric は既定で新しいマシンにワークロードを分配します。 たとえば、5 個のノードと 10 個のパーティションがある場合、既定では、Service Fabric は 2 個のプライマリ レプリカを各ノードに配置します。 ノードをスケールアウトすると、より多くのリソースに作業が均等に分配されるため、パフォーマンスを向上させることができます。 この戦略を利用するシナリオについては、「[Service Fabric での拡大縮小](/azure/service-fabric/service-fabric-concepts-scalability)」をご覧ください。
- パーティションの追加または削除は、あまり適切にサポートされていません。 スケーリングによく使用されるもう 1 つのオプションは、サービスまたはアプリケーション インスタンス全体を動的に作成または削除することです。 そのパターンの例については、「[新しい名前付きサービスの作成または削除による拡大縮小](/azure/service-fabric/service-fabric-concepts-scalability#scaling-by-creating-or-removing-new-named-services)」で説明されています。

詳細については、次を参照してください。

- [自動スケール ルールを使用した、または手動による Service Fabric クラスターのスケールインとスケールアウト](/azure/service-fabric/service-fabric-cluster-scale-up-down)
- [プログラムによる Service Fabric クラスターのスケール](/azure/service-fabric/service-fabric-cluster-programmatic-scaling)
- [仮想マシン スケール セットを追加して Service Fabric クラスターをスケールアウトする](/azure/service-fabric/virtual-machine-scale-set-scale-node-type-scale-out)

### <a name="using-metrics-to-balance-load"></a>メトリックを使用して負荷を分散させる

パーティションの設計方法によっては、あるノードのレプリカに他よりも多くのトラフィックが送られることがあります。 このような状況を避けるには、すべてのパーティションに分散されるようにサービスの状態をパーティション分割します。 範囲パーティション構成と適切なハッシュ アルゴリズムを使用します。 「[パーティション分割の使用](/azure/service-fabric/service-fabric-concepts-partitioning#get-started-with-partitioning)」をご覧ください。

Service Fabric では、メトリックを使用して、クラスター内にサービスを配置してバランスを取る方法が認識されます。 サービスを作成するときに、そのサービスに関連付けられる各メトリックの既定の負荷を指定することができます。 その後、Service Fabric では、サービスを配置するとき、またはクラスター内のノードのバランスを取るために (たとえば、アップグレード中に) サービスを移動する必要があるとき常に、その負荷が考慮されます。

サービスに対して最初に指定された既定の負荷は、サービスの有効期間を通して変更されません。 特定のサービスについて変化するメトリックをキャプチャするため、サービスを監視して負荷を動的にレポートすることお勧めします。 これにより、Service Fabric では特定の時点に報告される負荷に基づいて割り当てを調整できます。 カスタム メトリックを報告するには、[IServicePartition.ReportLoad](/dotnet/api/system.fabric.iservicepartition.reportload?view=azure-dotnet) メソッドを使用します。 詳しくは、「[動的な負荷](/azure/service-fabric/service-fabric-cluster-resource-manager-metrics#dynamic-load)」をご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

プライマリ ノード タイプ以外のノード タイプに、サービスを配置します。 Service Fabric のシステム サービスは、プライマリ ノード タイプに常にデプロイされます。 独自のサービスをプライマリ ノード タイプにデプロイすると、リソースがシステム サービスと競合し、システム サービスを干渉する可能性があります。 ノード タイプでステートフル サービスをホストすることが予想される場合は、ノード インスタンスの数を 5 個以上にして、シルバーまたはゴールドの持続性層を選択します。

サービスのリソースを制限することを検討します。 「[リソース ガバナンスのメカニズム](/azure/service-fabric/service-fabric-resource-governance#resource-governance-mechanism)」をご覧ください。

- リソース管理サービスとリソース非管理サービスを、同じノード タイプに混在させないでください。 非管理サービスによって消費されるリソースが多すぎて、リソース管理サービスに影響する可能性があります。 これらの種類のサービスが同じノードのセットで実行されないようにするには、[配置の制約](/azure/service-fabric/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies)を指定します。 「[リソース ガバナンスの指定](/azure/service-fabric/service-fabric-resource-governance#specify-resource-governance)」をご覧ください。 (これは、[バルクヘッド パターン](../../patterns/bulkhead.md)の例です)。
- サービス インスタンス用に予約する CPU コア数とメモリを指定します。 リソース ガバナンス ポリシーの使用と制限事項については、「[リソース ガバナンス](/azure/service-fabric/service-fabric-resource-governance)」をご覧ください。

単一障害点 (SPOF) を回避するため、すべてのサービスのターゲット インスタンスまたはレプリカの数を 1 より大きくします。 サービス インスタンスまたはレプリカの数として使用できる最大数は、サービスが制約されているノードの数と同じです。

すべてのステートフル サービスに、少なくとも 2 つのアクティブなセカンダリ レプリカがあるようにします。 運用ワークロードでは 5 個のレプリカをお勧めします。

詳しくは、「[Service Fabric サービスの可用性](/azure/service-fabric/service-fabric-availability-services)」をご覧ください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

Service Fabric でアプリケーションを保護するための重要な点を次に示します。

### <a name="virtual-network"></a>仮想ネットワーク

各仮想マシン スケール セットにサブネット境界を定義し、通信のフローを制御することを検討します。 各ノード タイプには、Service Fabric クラスターの仮想ネットワーク内のサブネットに、専用の仮想マシン スケール セットがあります。 ネットワーク セキュリティ グループ (NSG) をサブネットに追加して、ネットワーク トラフィックを許可または拒否できます。 たとえば、フロントエンドとバックエンドのノード タイプでは、バックエンドのサブネットに NSG を追加し、フロントエンドのサブネットだけで受信トラフィックを受け取ることができます。

クラスターから外部の Azure サービスを呼び出すとき、Azure サービスでサポートされている場合は、[仮想ネットワーク サービス エンドポイント](/azure/virtual-network/virtual-network-service-endpoints-overview)を使用します。 サービス エンドポイントを使用すると、サービスはクラスターの仮想ネットワークのみに固定されます。 たとえば、Cosmos DB を使用してデータを格納する場合は、サービス エンドポイントで Cosmos DB アカウントを構成し、特定のサブネットからのアクセスのみを許可します。 「[仮想ネットワークから Azure Cosmos DB リソースへのアクセス](/azure/cosmos-db/vnet-service-endpoint)」をご覧ください。

### <a name="endpoints-and-interservice-communication"></a>エンドポイントとサービス間の通信

セキュリティ保護されていない Service Fabric クラスターを作成しないでください。 クラスターでパブリック インターネットに管理エンドポイントを公開している場合、匿名ユーザーがそのクラスターに接続できるようになります。 セキュリティで保護されていないクラスターは、運用ワークロードでサポートされません。 参照:[Service Fabric クラスターのセキュリティに関するシナリオ](/azure/service-fabric/service-fabric-cluster-security)。

サービス間の通信をセキュリティ保護するには:

- ASP.NET Core サービスまたは Java Web サービスでは、HTTPS エンドポイントを有効にすることを考えます。
- リバース プロキシとサービスの間に、セキュリティ保護された接続を確立します。  詳しくは、[セキュリティで保護されたサービスへの接続](/azure/service-fabric/service-fabric-reverseproxy-configure-secure-communication)に関する記事をご覧ください。

[API ゲートウェイ](../../microservices/design/gateway.md)を使用している場合は、ゲートウェイに[認証をオフロードする](../../patterns/gateway-offloading.md)ことができます。 メッセージがゲートウェイから送られて来たのかどうかを認証する追加のセキュリティが設定されていない限り、個々のサービスに直接 (API ゲートウェイなしで) アクセスできないようにします。

Service Fabric のリバース プロキシをパブリックに公開しないでください。 そのようにすると、HTTP エンドポイントを公開するすべてのサービスが、クラスターの外部からアドレス指定可能になり、セキュリティの脆弱性が発生して、その他の情報が不必要にクラスターの外部に公開される可能性があります。 サービスをパブリックにアクセス可能にする場合は、API ゲートウェイを使用します。 いくつかのオプションが、「[API ゲートウェイ](#api-gateway)」セクションで説明されています。

リモート デスクトップは診断とトラブルシューティングに便利ですが、開いたままにしないようにしてください。そうしないと、セキュリティ ホールの原因になります。

### <a name="secrets-and-certificates"></a>シークレットと証明書

Service Fabric サービスから Key Vault のシークレットにアクセスするには、サービスがホストされている仮想マシン スケール セットで[マネージド ID](/azure/active-directory/managed-identities-azure-resources/services-support-msi#azure-virtual-machine-scale-sets) を有効にします。 サンプル コード:マネージド サービス ID を使用して App Service から Key Vault を使用します。

クライアント証明書を使用して Service Fabric Explorer にアクセスしないでください。 代わりに、Azure Active Directory (Azure AD) を使用します。 「[Azure AD 認証をサポートしている Azure サービス](/azure/active-directory/managed-identities-azure-resources/services-support-msi%23azure-services-that-support-azure-ad-authentication)」もご覧ください。

運用環境では自己署名証明書を使用しないでください。

Service Fabric のセキュリティ保護について詳しくは、以下をご覧ください。

- [Azure Service Fabric のセキュリティの概要](/azure/security/azure-service-fabric-security-overview)
- [Azure Service Fabric のセキュリティに関するベスト プラクティス](/azure/security/azure-service-fabric-security-best-practices)
- [Azure Service Fabric セキュリティのチェックリスト](/azure/security/azure-service-fabric-security-checklist)

## <a name="monitoring-considerations"></a>監視に関する考慮事項

監視オプションを調べる前に、[Service Fabric での一般的なシナリオの診断](/azure/service-fabric/service-fabric-diagnostics-common-scenarios)に関するこちらの記事を読むことをお勧めします。 データの監視については以下のセットで考えることができます。

- [アプリケーションのメトリックとログ](#application-metrics-and-logs)
- [Service Fabric の正常性とイベント データ](#service-fabric-health-and-event-data)
- [インフラストラクチャのメトリックとログ](#infrastructure-metrics-and-logs)
- [依存サービスのメトリックとログ](#dependent-service-metrics)

そのデータを分析するための主なオプションは次の 2 つです。

- Application Insights
- Log Analytics

Azure Monitor を使用して、監視用のダッシュボードを設定し、オペレーターにアラートを送信することができます。 Dynatrace など、Service Fabric と統合できるサード パーティ製の監視ツールもあります。 詳しくは、「[Azure Service Fabric 監視パートナー](/azure/service-fabric/service-fabric-diagnostics-partners)」をご覧ください。

### <a name="application-metrics-and-logs"></a>アプリケーションのメトリックとログ

アプリケーションのテレメトリでは、サービスの正常性を監視して問題を特定するのに役立つ、サービスに関するデータが提供されます。 サービスでトレースとイベントを追加するには:

- ASP.NET Core でサービスを開発している場合は、[Microsoft.Extensions.Logging](/azure/service-fabric/service-fabric-how-to-diagnostics-log#microsoftextensionslogging)。 他のフレームワークの場合は、Serilog などの適当なログ記録ライブラリを使用します。
- SDK の [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient?view=azure-dotnet) クラスを使用して独自のインストルメンテーションを追加し、Application Insights でデータを表示することができます。 「[カスタム インストルメンテーションをアプリケーションに追加する](/azure/service-fabric/service-fabric-tutorial-monitoring-aspnet#add-custom-instrumentation-to-your-application)」をご覧ください。
- [EventSource](/azure/service-fabric/service-fabric-diagnostics-event-generation-app#eventsource) を使用して、ETW イベントをログに記録します。 このオプションは、Visual Studio の Service Fabric ソリューションにおいて既定で使用できます。

トレースとイベント ログを表示するには、構造化ログのシンクの 1 つとして [Application Insights](/azure/service-fabric/service-fabric-diagnostics-event-analysis-appinsights) を使用し、サービスでトレースとイベントを視覚化できるようにします。 Application Insights には、多くの組み込みテレメトリ (要求、トレース、イベント、例外、メトリック、依存関係) が用意されています。 Application Insights に対するサービスのインストルメント化については、次の記事をご覧ください。

- [チュートリアル:Application Insights を使用して Service Fabric 上の ASP.NET Core アプリケーションを監視および診断する](/azure/service-fabric/service-fabric-tutorial-monitoring-aspnet)。
- [Application Insights for ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
- [Application Insights .NET SDK](/azure/application-insights/app-insights-api-custom-events-metrics)
- [Application Insights SDK for Service Fabric](https://github.com/Microsoft/ApplicationInsights-ServiceFabric)

ASP.NET Core サービスでは、アプリケーション ログ記録に [ILogger インターフェイス](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2)が使用されます。 これらのアプリケーション ログを Azure Monitor で使用できるようにするには、`ILogger` イベントを Application Insights に送信します。 詳しくは、「[ILogger in an ASP.NET Core application (ASP.NET Core アプリケーションでの ILogger)](https://github.com/Microsoft/ApplicationInsights-dotnet-logging/blob/develop/src/ILogger/Readme.md#aspnet-core-application)」をご覧ください。 Application Insights では、相関関係プロパティを ILogger イベントに追加できます。これは、分散トレースの視覚化に役立ちます。

詳細については、次を参照してください。

- [アプリケーションのログ記録](/azure/service-fabric/service-fabric-diagnostics-event-generation-app)
- [Service Fabric アプリケーションにログ記録を追加する](/azure/azure-monitor/app/correlation)

### <a name="service-fabric-health-and-event-data"></a>Service Fabric の正常性とイベント データ

Service Fabric のテレメトリには、Service Fabric クラスターとそのエンティティ (ノード、アプリケーション、サービス、パーティション、レプリカ) の操作とパフォーマンスに関する正常性メトリックとイベントが含まれます。

- [EventStore](/azure/service-fabric/service-fabric-diagnostics-eventstore)。 クラスターとそのエンティティに関連するイベントを収集するステートフルなシステム サービスです。 Service Fabric では EventStore を使用して [Service Fabric イベント](/azure/service-fabric/service-fabric-diagnostics-event-generation-operational)が書き込まれてクラスターに関する情報が提供され、状態の更新、トラブルシューティング、監視に使用できます。 また、特定の時点でのさまざまなエンティティからのイベントが関連付けられ、クラスター内の問題が識別されます。 サービスでは、REST API を介してこれらのイベントが公開されます。 EventStore API のクエリを実行する方法については、「[クラスター イベントに対して EventStore API のクエリを実行する](/azure/service-fabric/service-fabric-diagnostics-eventstore-query)」をご覧ください。 WAD 拡張機能でクラスターを構成することにより、Log Analytics で EventStore からのイベントを表示できます。
- [HealthStore](/azure/service-fabric/service-fabric-health-introduction)。 クラスターの現在の正常性のスナップショットが提供されます。 階層内のエンティティによって報告されたすべての正常性データを集約するステートフル サービスです。 データは、[Service Fabric Explorer][sfx] で視覚化されます。 HealthStore では、アプリケーションのアップグレードも監視されます。 PowerShell、.NET アプリケーション、または REST API で、正常性クエリを使用できます。 「[Service Fabric の正常性モニタリングの概要](/azure/service-fabric/service-fabric-health-introduction)」をご覧ください。
- 内部カスタム ウォッチドッグ サービスの実装を検討します。 それらのサービスでは、実行中のサービスの障害の状態など、カスタム正常性データを定期的に報告できます。 詳しくは、[カスタム正常性レポート](/azure/service-fabric/service-fabric-report-health)に関する記事をご覧ください。 Service Fabric Explorer を使用して正常性レポートを読み取ることができます。  

### <a name="infrastructure-metrics-and-logs"></a>インフラストラクチャのメトリックとログ

インフラストラクチャのメトリックを使用すると、クラスター内のリソース割り当てを理解できます。 この情報を収集するための主なオプションを次に示します。

- Windows Azure Diagnostics (WAD)。 Windows のノード レベルでログとメトリックを収集します。 ノード タイプにマップされる任意の仮想マシン スケール セットで IaaSDiagnostics VM 拡張機能を構成することによって WAD を使用して、Windows イベント ログ、パフォーマンス カウンター、ETW/マニフェスト システムと運用イベント、カスタム ログなどの診断イベントを収集できます。
- Log Analytics エージェント。 MicrosoftMonitoringAgent VM 拡張機能を構成し、Windows イベント ログ、パフォーマンス カウンター、カスタム ログを Log Analytics に送信します。

パフォーマンス カウンターなど、上記のメカニズムを通じて収集されるメトリックの種類にはいくつか重複するものがあります。 重複する部分がある場合は、Log Analytics エージェントを使うことをお勧めします。 Log Analytics エージェントの場合は Azure ストレージがないため、待ち時間が短くなります。 また、IaaSDiagnostics のパフォーマンス カウンターでは、Log Analytics に簡単にフィードできません。

![Service Fabric でのインフラストラクチャの監視](./_images/ra-sf-monitoring.png)

VM 拡張機能の使用については、「[Azure 仮想マシンの拡張機能と機能](/azure/virtual-machines/extensions/overview)」をご覧ください。

データを表示するには、WAD によって収集されたデータを表示するように Log Analytics を構成します。 ストレージ アカウントからイベントを読み取るように Log Analytics を構成する方法については、「[クラスターに Log Analytics を設定する](/azure/service-fabric/service-fabric-diagnostics-oms-setup)」をご覧ください。

Service Fabric クラスター、ワークロード、ネットワーク トラフィック、保留中の更新プログラムなどに関連するパフォーマンス ログと テレメトリ データを表示することもできます。 「[Log Analytics でのパフォーマンスの監視](/azure/service-fabric/service-fabric-diagnostics-oms-agent)」をご覧ください。

[Log Analytics の Service Map ソリューション](/azure/azure-monitor/insights/service-map)では、クラスターのトポロジ (つまり、各ノードで実行されているプロセス) に関する情報が提供されます。 ストレージ アカウント内のデータを [Application Insights](/azure/monitoring-and-diagnostics/azure-diagnostics-configure-application-insights) に送信します。 Application Insights へのデータの取得には若干の遅延がある可能性があります。 データをリアルタイムで見たい場合は、シンクとチャネルを使用して [Event Hubs](/azure/monitoring-and-diagnostics/azure-diagnostics-streaming-event-hubs) を構成することを考えます。 詳しくは、「[Windows Azure 診断を使用したイベントの集計と収集](/azure/service-fabric/service-fabric-diagnostics-event-aggregation-wad)」をご覧ください。

### <a name="dependent-service-metrics"></a>依存サービスのメトリック

- [Application Insights のアプリケーション マップ](/azure/azure-monitor/app/app-map)では、インストールされている Application Insights SDK によりサービス間で行われた HTTP 依存関係呼び出しを使用して、アプリケーションのトポロジが提供されます。
- [Log Analytics の Service Map ソリューション](/azure/azure-monitor/insights/service-map)では、外部サービスとの間の受信および送信トラフィックについての情報が提供されます。 さらに、Service Map は、更新やセキュリティなどの他のソリューションと統合します。
- カスタム ウォッチドッグを使用すると、外部サービスでのエラー状態をレポートできます。 たとえば、サービスは、外部サービスまたはデータ ストレージ (Azure Cosmos DB) にアクセスできない場合に、エラー正常性レポートを報告します。  

### <a name="distributed-tracing"></a>分散トレース

マイクロサービス アーキテクチャでは、タスクを完了するために複数のサービスが参加することがよくあります。 それらの各サービスからのテレメトリは、分散トレースのコンテキスト フィールド (操作 ID、要求 ID など) を使用して関連付けられます。 Application Insights で[アプリケーション マップ](/azure/azure-monitor/app/app-map)を使用することにより、分散型論理操作のビューを構築し、アプリケーションのサービス グラフ全体を視覚化できます。 また、Application Insight でトランザクション診断を使用して、サーバー側テレメトリを関連付けることもできます。 くわしくは、「[統合されたコンポーネント間のトランザクションの診断](/azure/application-insights/app-insights-transaction-diagnostics)」をご覧ください。

[Application Insights のアプリケーション マップ](/azure/azure-monitor/app/app-map)では、インストールされている Application Insights SDK によりサービス間で行われた HTTP 依存関係呼び出しを使用して、アプリケーションのトポロジが提供されます。 キューを使用して非同期にディスパッチされるタスクを関連付けることも重要です。 キュー メッセージで相関関係テレメトリを送信する方法について詳しくは、「[キューのインストルメンテーション](/azure/azure-monitor/app/custom-operations-tracking#queue-instrumentation)」をご覧ください。

詳細については、次を参照してください。

- [複数のリソース間でのクエリの実行](/azure/azure-monitor/log-query/cross-workspace-query#performing-a-query-across-multiple-resources)
- [Application Insights におけるテレメトリの相関付け](/azure/azure-monitor/app/correlation)

#### <a name="alerts-and-dashboards"></a>アラートとダッシュボード

Application Insights と Log Analytics では[広範なクエリ言語](/azure/log-analytics/query-language/get-started-queries) (Kusto クエリ言語) がサポートされており、ログ データを取得して分析できます。 クエリを使用してデータ セットを作成し、診断ダッシュボードでそれを視覚化します。

Azure Monitor のアラートを使用して、特定のリソースで特定の状態が発生したときにシステム管理者に通知します。 通知には、メール、Azure 関数、Webhook の呼び出しなどを使用できます。 詳しくは、[Azure Monitor でのアラート](/azure/azure-monitor/platform/alerts-overview)に関する記事をご覧ください。

[ログ検索アラート ルール](/azure/azure-monitor/platform/alerts-unified-log)では、Kusto クエリを定義し、一定の間隔で Log Analytics ワークスペースに対して実行できます。 クエリの結果が特定の条件に一致する場合は、アラートが作成されます。

## <a name="next-steps"></a>次の手順

- [ドメイン分析を使用したマイクロサービスのモデル化](../../microservices/model/domain-analysis.md)
- [マイクロサービス アーキテクチャの設計](../../microservices/design/index.md)

[sfx]: /azure/service-fabric/service-fabric-visualizing-your-cluster
