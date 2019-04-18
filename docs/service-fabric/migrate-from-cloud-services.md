---
title: Azure Cloud Services アプリケーションを Azure Service Fabric に移行する
description: Azure Cloud Services アプリケーションを Azure Service Fabric に移行する方法。
author: MikeWasson
ms.date: 04/11/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: a1fc28737b194fe69e2ae094bd996d97363eb29c
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641112"
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a>Azure Cloud Services アプリケーションを Azure Service Fabric に移行する 

[![GitHub](../_images/github.png) サンプル コード][sample-code]

この記事では、Azure Cloud Services のアプリケーションを Azure Service Fabric に移行する方法について説明します。 これはアーキテクチャについての決定を中心とした、推奨されるプラクティスです。 

このプロジェクトでは、Surveys と呼ばれる Cloud Services アプリケーションで作業を開始し、それを Service Fabric に移植しました。 目標は、できるだけ少ない変更でアプリケーションを移行することでした。 後の記事では、マイクロサービス アーキテクチャを採用することで、アプリケーションを Service Fabric 用に最適化します。

この記事を読む前に、全体としての Service Fabric とマイクロサービスのアーキテクチャに関する基本を理解しておくと有益です。 次の記事を参照してください。

- [Azure Service Fabric の概要][sf-overview]
- [マイクロサービスの手法でアプリケーションを構築する理由は何ですか。][sf-why-microservices]

## <a name="about-the-surveys-application"></a>Surveys アプリケーションについて

2012 年に、パターンとプラクティスを担当するグループが、『[Developing Multi-tenant Applications for the Cloud (クラウド向けマルチテナント アプリケーションの開発)][tailspin-book]』というタイトルの書籍のために Surveys というアプリケーションを作成しました。 この書籍は、Surveys アプリケーションの設計と実装を行う Tailspin という架空の企業のことを描写しています。

Surveys は、顧客がアンケートを作成できるマルチテナント アプリケーションです。 顧客がアプリケーションにサインアップした後は、顧客の組織のメンバーが調査を作成して発行し、分析の結果を収集できます。 アプリケーションには、ユーザーが調査を実施できるパブリック Web サイトが含まれています。 元の Tailspin シナリオの詳細については、[ここ][tailspin-scenario]を参照してください。

Tailspin は現在、Azure で実行される Service Fabric を使用して Surveys アプリケーションをマイクロサービス アーキテクチャに移行しようと考えています。 このアプリケーションは既に Cloud Services アプリケーションとしてデプロイされているため、Tailspin は複数フェーズのアプローチを採用します。

1. アプリケーションへの変更を最小限にしながらクラウド サービスを Service Fabric に移植します。
2. マイクロサービス アーキテクチャに移行することで、アプリケーションを Service Fabric 用に最適化します。

この記事では最初のフェーズについて説明します。 後の記事では 2 番目のフェーズについて説明します。 実際のプロジェクトでは、おそらく両方のステージの時期が重なります。 Service Fabric への移植を行うときに、アプリケーションのマイクロサービスへの再設計も開始することになります。 その後、アーキテクチャをさらに調整する場合もあり、粒度の粗いサービスをより小さなサービスに分割する可能性があります。  

アプリケーション コードは [GitHub][sample-code] から入手できます。 このリポジトリには、Cloud Services アプリケーションと Service Fabric バージョンの両方が含まれています。 

> クラウド サービスは、書籍『*Developing Multi-tenant Applications for the Cloud (クラウド向けマルチテナント アプリケーションの開発)*』に記載された元のアプリケーションの更新されたバージョンです。

## <a name="why-microservices"></a>マイクロサービスを使用する理由

マイクロサービスの詳しい説明についてはこの記事の範囲外ですが、以下に Tailspin が、マイクロサービス アーキテクチャに移行することで得られると期待しているいくつかのメリットを示します。

- **アプリケーションのアップグレード**。 サービスは独立してデプロイできるため、アプリケーションのアップグレードには増分アプローチを採用できます。
- **回復性と障害の分離**。 サービスが失敗した場合、その他のサービスは実行され続けます。
- **スケーラビリティ**:  サービスは個別にスケールできます。
- **柔軟性**。 サービスは、テクノロジ スタックではなくビジネス シナリオを中心に設計されていて、サービスを新しいテクノロジ、フレームワーク、またはデータ ストアに移行しやすくなっています。
- **アジャイル開発**。 個々のサービスのコード量はモノリシック アプリケーションよりも少なく、コード ベースの理解、推論、およびテストが容易になっています。
- **集中的な小規模チーム**。 アプリケーションは、多くの小さなサービスに分割されているため、各サービスは集中的な小規模チームで構築できます。

## <a name="why-service-fabric"></a>Service Fabric を使用する理由
      
Service Fabric には、以下を含め、分散システムで必要とされるほとんどの機能が組み込まれているため、Service Fabric はマイクロサービス アーキテクチャとの使用に適しています。

- **クラスターの管理**。 Service Fabric は、ノードのフェールオーバー、正常性の監視、およびその他のクラスター管理機能に自動的に処理します。
- **水平スケーリング**。 Service Fabric クラスターにノードを追加すると、サービスは新しいノード全体にわたって分散されるため、アプリケーションは自動的にスケーリングされます。
- **サービスの探索**。 Service Fabric は、名前付きサービスのエンドポイントを解決できる探索サービスを提供します。
- **ステートレス サービスとステートフル サービス**。 ステートフル サービスは [Reliable Collection][sf-reliable-collections] を使用します。これはキャッシュやキューの代わりに使用でき、パーティション化できます。
- **アプリケーション ライフサイクル管理**。 サービスは、個別に、アプリケーションのダウンタイムなくアップグレードできます。
- マシンのクラスター全体にわたる**サービス オーケストレーション**。
- リソースの消費を最適化するために**密度が向上**。 1 つのノードで複数のサービスをホストできます。

Service Fabric は、Azure SQL Database、Cosmos DB、Azure Event Hubs などを含むさまざまな Microsoft サービスによって使用されており、分散クラウド アプリケーション構築用の実績あるプラットフォームとなっています。 

## <a name="comparing-cloud-services-with-service-fabric"></a>Cloud Services と Service Fabric の比較

次の表は、Cloud Services アプリケーションと Service Fabric アプリケーションの重要な違いの一部をまとめたものです。 詳細な説明については、「[アプリケーションの移行前に、Cloud Services と Service Fabric の違いについて学習する][sf-compare-cloud-services]」を参照してください。

|        | Cloud Services | Service Fabric |
|--------|---------------|----------------|
| アプリケーションの構成 | ロール| サービス |
| 密度 |VM ごとに 1 つのロール インスタンス | 1 つのノードで複数のサービス |
| 最小ノード数 | ロールあたり 2 | 運用デプロイメントの場合はクラスターあたり 5 |
| 状態管理 | ステートレス | ステートレスまたはステートフル* |
| Hosting | Azure | クラウドまたはオンプレミスの |
| Web ホスティング | IIS** | 自己ホスト |
| デプロイメント モデル | [クラシック デプロイ モデル][azure-deployment-models] | [Resource Manager][azure-deployment-models]  |
| 梱包 | クラウド サービス パッケージ ファイル (.cspkg) | アプリケーションおよびサービス パッケージ |
| アプリケーションの更新 | VIP のスワップまたはローリング アップデート | ローリング アップデート |
| 自動スケール | [組み込みのサービス][cloud-service-autoscale] | 自動スケール用の VM Scale Sets |
| デバッグ | ローカル エミュレーター | ローカル クラスター |

\* ステートフル サービスが [Reliable Collection][sf-reliable-collections] を使用してレプリカ間にわたる状態を保存し、すべての読み取りがクラスター内のノードに対してローカルであるようにしています。 書き込みは、信頼性のため、ノード間でレプリケートされます。 ステートレスなサービスは、データベースまたはその他の外部ストレージを使用して、外部の状態を持っている場合があります。

** worker ロールは、OWIN を使用して ASP.NET Web API を自己ホストすることもできます。

## <a name="the-surveys-application-on-cloud-services"></a>Cloud Services 上の Surveys アプリケーション

次の図は、Cloud Services で実行中の Surveys アプリケーションのアーキテクチャを示しています。 

![](./images/tailspin01.png)

アプリケーションは、2 つの Web ロールと 1 つのworker ロールで構成されています。

- **Tailspin.Web** Web ロールは、Tailspin の顧客が調査の作成と管理に使用する ASP.NET Web サイトをホストしています。 顧客は、アプリケーションへのサインアップとサブスクリプションの管理にもこの Web サイトを使用します。 最後に、Tailspin の管理者は、このロールを使用してテナントのリストを表示し、テナント データを管理できます。 

- **Tailspin.Web.Survey.Public** Web ロールは、Tailspin の顧客が公開する調査に人々が参加できる ASP.NET web サイトをホストしています。 

- **Tailspin.Workers.Survey** worker ロールは、バックグラウンド処理を行います。 Web ロールはキューに作業項目を入れ、worker ロールが項目を処理します。 2 つのバックグラウンド タスクが定義されています。Azure SQL Database に調査の回答をエクスポートするタスクと、調査の回答の統計を計算するタスクです。

Cloud Services に加えて、Surveys アプリケーションが他のいくつかの Azure サービスを使用します。

- 調査、調査の回答、テナント情報を格納する **Azure Storage**。

- 高速な読み取りアクセスのために Azure Storage に格納されているデータの一部をキャッシュする **Azure Redis Cache**。 

- 顧客と Tailspin 管理者を認証する **Azure Active Directory** (Azure AD)。

- 分析のために調査の回答を格納する **Azure SQL Database**。 

## <a name="moving-to-service-fabric"></a>Service Fabric への移行

前述のように、このフェーズの目的は、必要最低限の変更で Service Fabric に移行することでした。 そのために、元のアプリケーションの各クラウド サービス ロールに対応するステートレス サービスを作成しました。

![](./images/tailspin02.png)

意図的に、このアーキテクチャは元のアプリケーションによく似たものにしています。 ただし、図にはいくつかの重要な違いが隠れています。 この記事の残りの部分では、それらの相違点について説明します。 

## <a name="converting-the-cloud-service-roles-to-services"></a>クラウド サービス ロールのサービスへの変換

前述のように、各クラウド サービス ロールを Service Fabric サービスに移行しました。 クラウド サービス ロールはステートレスであるため、このフェーズでは Service Fabric 内にステートレス サービスを作成するのが合理的でした。 

移行については、「[Web と worker ロールを Service Fabric ステートレス サービスに変換する手順][sf-migration]」で概要が説明されているステップに従いました。 

### <a name="creating-the-web-front-end-services"></a>Web フロント エンド サービスの作成

Service Fabric 内では、サービスは Service Fabric ランタイムによって作成されるプロセス内で実行されます。 Web フロント エンドの場合、サービスは IIS 内で実行されていないことになります。 代わりに、サービスは Web サーバーをホストする必要があります。 プロセス内で実行されるコードは Web サーバー ホストとして機能するため、このアプローチは*自己ホスト*と呼ばれます。 

自己ホストの要件により、Service Fabric サービスは ASP.NET MVC や ASP.NET Web フォームを使用できないことになります。これらのフレームワークは IIS を必要とし、自己ホストをサポートしないためです。 自己ホストには次のオプションがあります。

- [ASP.NET Core][aspnet-core] ([Kestrel][kestrel] Web サーバーを使用して自己ホストされる)。 
- [ASP.NET Web API][aspnet-webapi] ([OWIN][owin] して自己ホストされる)。
- サードパーティ フレームワーク ([Nancy](http://nancyfx.org/) など)。

元の Surveys アプリケーションは ASP.NET MVC を使用しています。 ASP.NET MVC は Service Fabric で自己ホストすることはできないため、以下の移行オプションを検討しました。

- Web ロールを、自己ホストが可能な ASP.NET Core に移植する。
- Web サイトを、ASP.NET Web API を使用して実装された Web API を呼び出す単一ページ アプリケーション (SPA) に変換する。 このためには、Web フロント エンドの完全な再設計が必要でした。
- ASP.NET MVC の既存のコードを保持し、Windows Server コンテナー内の IIS を Service Fabric にデプロイする。 このアプローチでは、コード変更の必要がほとんどないかまったくありません。 

ASP.NET Core に移植するという最初のオプションを使用すると、ASP.NET Core の最新機能を利用できます。 この変換を行うために、[ASP.NET MVC から ASP.NET Core MVC への移行][aspnet-migration]に関するページに記載されている手順に従いました。 

> [!NOTE]
> Kestrel と共に ASP.NET Core を使用する場合は、セキュリティ上の理由から、Kestrel の前にリバース プロキシを配置し、インターネットからのトラフィックを処理する必要があります。 詳細については、「[Kestrel web server implementation in ASP.NET Core (ASP.NET Core での Kestrel Web サーバーの実装)][kestrel]」を参照してください。 「[アプリケーションのデプロイ](#deploying-the-application)」のセクションでは、推奨される Azure のデプロイについて説明します。

### <a name="http-listeners"></a>HTTP リスナー

Cloud Services では、[サービス定義ファイル][cloud-service-endpoints]で HTTP エンドポイントを宣言することで、Web ロールまたは worker ロールが HTTP エンドポイントを公開します。 Web ロールにはエンドポイントが少なくとも 1 つ必要です。

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

同様に、Service Fabric エンドポイントはサービス マニフェストで宣言されます。 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

クラウド サービス ロールとは異なり、Service Fabric のサービスは同じノード内に併置できます。 そのため、すべてのサービスは別個のポートでリッスンする必要があります。 この記事の後方で、ポート 80 またはポート 443 上のクライアント要求が、サービスの正しいポートにルーティングされるようにする方法を説明します。

サービスは、各エンドポイント用のリスナーを明示的に作成する必要があります。 Service Fabric は通信スタックを区別しないことがその理由です。 詳細については、「[ASP.NET Core を使用したアプリケーション用の Web サービス フロントエンドの構築][sf-aspnet-core]」を参照してください。

## <a name="packaging-and-configuration"></a>パッケージと構成

 1 つのクラウド サービスには、次の構成とパッケージ ファイルが含まれます。

| ファイル | 説明 |
|------|-------------|
| サービス定義 (.csdef) | クラウド サービスを構成するために Azure によって使用される設定。 ロール、エンドポイント、スタートアップ タスク、および構成設定の名前を定義します。 |
| サービス構成 (.cscfg) | ロール インスタンスの数、エンドポイントのポート番号、構成設定の値を含むデプロイごとの設定。 
| サービス パッケージ (.cspkg) | アプリケーション コード、構成、およびサービス定義ファイルが含まれます。  |

アプリケーション全体に対して 1 つの .csdef ファイルがあります。 ローカル、テスト、運用など、異なる環境のために複数の .cscfg ファイルを持つことができます。 サービスの実行中、.cscfg は更新可能ですが、.csdef は更新できません。 詳細については、「[クラウド サービス モデルとそのパッケージ化について][cloud-service-config]」を参照してください。

Service Fabric では、サービス*定義*とサービス*設定*の間に同様の区別がありますが、構造の粒度はより高くなっています。 Service Fabric の構成モデルを理解するには、Service Fabric アプリケーションがどのようにパッケージ化されるかを理解すると役立ちます。 構造は次のようになっています。

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

デプロイするのはアプリケーション パッケージです。 1 つまたは複数のサービス パッケージが含まれています。 1 つのサービス パッケージには、コード、構成、およびデータのパッケージが含まれます。 コード パッケージにはサービスのバイナリが含まれていて、構成パッケージには構成設定が含まれます。 このモデルでは、アプリケーション全体を再デプロイしなくても個々のサービスをアップグレードできます。 また、コードの再デプロイやサービスの再起動を行わず、構成設定のみを更新できます。

Service Fabric アプリケーションには以下の構成ファイルが含まれています。

| ファイル | Location | 説明 |
|------|----------|-------------|
| ApplicationManifest.xml | アプリケーション パッケージ | アプリケーションを構成するサービスを定義します。 |
| ServiceManifest.xml | サービス パッケージ| 1 つまたは複数のサービスについて記述します。 |
| Settings.xml | 構成パッケージ | サービス パッケージで定義されているサービスの構成設定が含まれます。 |

詳細については、「[Service Fabric のアプリケーションのモデル化][sf-application-model]」を参照してください。

複数の環境の異なる構成設定をサポートするには、「[複数の環境のアプリケーション パラメーターを管理する][sf-multiple-environments]」で説明されている次のアプローチを使用します。

1. サービスの Setting.xml ファイルで設定を定義します。
2. アプリケーション マニフェストで、設定のオーバーライドを定義します。
3. アプリケーション パラメーター ファイルに環境固有の設定を記入します。

## <a name="deploying-the-application"></a>アプリケーションのデプロイ

Azure Cloud Services は管理されたサービスであるのに対して、Service Fabric はランタイムです。 Azure とオンプレミスを含めて、多くの環境で Service Fabric クラスターを作成できます。 この記事では、Azure へのデプロイに焦点を合わせます。 

次の図は、推奨されるデプロイを示しています。

![](./images/tailspin-cluster.png)

Service Fabric クラスターは [VM スケール セット][vm-scale-sets]にデプロイされます。 スケール セットは、まったく同じ VM のセットをデプロイおよび管理するために使用できる Azure コンピューティング リソースです。 

前述のように、Kestrel Web サーバーには、セキュリティ上の理由でリバース プロキシが必要です。 この図は、さまざまな第 7 層負荷分散機能を提供する Azure サービスである [Azure Application Gateway][application-gateway] を示しています。 これは、クライアント接続を終了して、バックエンドのエンドポイントへの要求を転送し、リバース プロキシ サービスとして機能します。 nginx など、異なるリバース プロキシ ソリューションを使用できます。  

### <a name="layer-7-routing"></a>第 7 層のルーティング

[元の Surveys アプリケーション](https://msdn.microsoft.com/library/hh534477.aspx#sec21)では、1 つの Web ロールがポート 80 でリッスンし、別の Web ロールがポート 443 でリッスンしていました。 

| 公開サイト | Survey 管理サイト |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

その他に、第 7 層のルーティングを使用するオプションがあります。 このアプローチでは、異なる URL パスはバック エンドの異なるポート番号にルーティングされます。 たとえば、公開サイトには `/public/` で始まる URL パスを使用できます。 

第 7 層のルーティングには以下のオプションがあります。

- Application Gateway を使用します。 

- nginx などのネットワーク仮想アプライアンス (NVA) を使用します。

- カスタム ゲートウェイをステートレス サービスとして記述します。

公開の HTTP エンドポイントを持つサービスが 2 つ以上あっても、1 つのドメイン名を持つ 1 つのサイトとして表示する場合は、このアプローチを検討してください。

> お勧め*しない*アプローチの 1 つは、外部クライアントが Service Fabric の[リバース プロキシ][sf-reverse-proxy]経由で要求を送信するのを許可することです。 これは可能ですが、リバース プロキシはサービス間の通信を目的としています。 これを外部のクライアントに開くことで、クラスター内で実行されている、HTTP エンドポイントを持つサービスを*すべて*公開することになります。

### <a name="node-types-and-placement-constraints"></a>ノードの種類と配置の制約

前に示したデプロイでは、すべてのサービスがすべてのノードで実行されます。 ただし、サービスをグループ化して、特定のサービスがクラスター内の特定のノードでのみ実行されるようにもできます。 このアプローチは、次のような理由で使用します。

- タイプが異なる VM タイプで一部のサービスを実行する。 たとえば、一部のサービスがコンピューティング集中型であったり、GPU を必要としたりする場合があります。 Service Fabric クラスター内には、複数タイプの VM を混在できます。
- セキュリティ上の理由で、バックエンド サービスからフロントエンド サービスを分離する。 すべてのフロントエンド サービスは 1 セットのノードで実行され、バックエンド サービスは同じクラスター内の異なるノードで実行されます。
- スケール要件が異なる。 一部のサービスは、他のサービスよりも多くのノードで実行する必要がある場合があります。 たとえば、フロントエンド ノードとバックエンド ノードを定義する場合、各セットを個別にスケールできます。

次の図は、フロントエンド サービスとバックエンド サービスを分けたクラスターを示しています。

![](././images/node-placement.png)

このアプローチの実装方法:

1. クラスターを作成するときに、ノード タイプを 2 つ以上定義します。 
2. サービスごとに[配置の制約][sf-placement-constraints]を使用して、サービスをノード タイプに割り当てます。

Azure にデプロイするときに、各ノード タイプは別個の VM スケール セットにデプロイされます。 Service Fabric クラスターは、すべてのノード タイプにまたがります。 詳細については、「[Azure Service Fabric ノードの種類と仮想マシン スケール セット][sf-node-types]」を参照してください。

> クラスターに複数のノード タイプがある場合、1 つのノード タイプが*プライマリ* ノード タイプとして指定されます。 Cluster Management Service などの Service Fabric ランタイム サービスは、プライマリ ノード タイプで実行されます。 運用環境では、プライマリ ノード タイプには少なくとも 5 つのノードをプロビジョニングしてください。 その他のノード タイプには、少なくとも 2 つのノードが必要です。

## <a name="configuring-and-managing-the-cluster"></a>クラスターの構成と管理

クラスターをセキュリティで保護し、承認されていないユーザーがクラスターに接続しないようにする必要があります。 クライアントの認証には Azure AD を、ノード間のセキュリティのためには X.509 証明書を使用することをお勧めします。 詳細については、「[Service Fabric クラスターのセキュリティに関するシナリオ][sf-security]」を参照してください。

公開の HTTPS エンドポイントを構成するには、「[サービス マニフェストにリソースを指定する][sf-manifest-resources]」を参照してください。

クラスターに VM を追加することで、アプリケーションをスケールアウトできます。 VM スケール セットは、パフォーマンス カウンターに基づく自動スケール ルールを使用した自動スケールをサポートしています。 詳細については、「[自動スケール ルールを使用した Service Fabric クラスターのスケールインとスケールアウト][sf-auto-scale]」を参照してください。

クラスターの実行中には、1 つの場所ですべてのノードからログを収集する必要があります。 詳細については、[Azure Diagnostics でログを収集する方法][sf-logs]に関するページを参照してください。   

## <a name="conclusion"></a>まとめ

Surveys アプリケーションの Service Fabric への移植は、かなりわかりやすいものでした。 要約すると、以下のことを行いました。

- ロールをステートレス サービスに変換しました。
- Web フロントエンドを ASP.NET Core に変換しました。
- パッケージと構成のファイルを Service Fabric モデルに変更しました。

さらに、デプロイを Cloud Services から VM スケール セットで実行される Service Fabric クラスターに変更しました。

## <a name="next-steps"></a>次の手順

Survey アプリケーションは適切に移植されました。Tailspin が次に求めているのは、独立したサービスのデプロイ、バージョン管理など、Service Fabric 機能を利用することです。 「[Refactor an Azure Service Fabric Application migrated from Azure Cloud Services (Azure Cloud Services から移行した Azure Service Fabric アプリケーションをリファクタリングする)][refactor-surveys]」では、Tailspin が、このような Service Fabric 機能を利用するために、こうしたサービスをどのように詳細なアーキテクチャに分解したかを説明しています

<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: /aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[refactor-surveys]: refactor-migrated-app.md
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
