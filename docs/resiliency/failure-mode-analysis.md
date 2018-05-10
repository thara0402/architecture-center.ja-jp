---
title: 障害モード分析
description: Azure に基づくクラウド ソリューションの障害モード分析を実行するためのガイドラインです。
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 95068bf8b1f5b559255e27819aaddb454d3427bc
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/07/2018
---
# <a name="failure-mode-analysis"></a>障害モード分析
[!INCLUDE [header](../_includes/header.md)]

障害モード分析 (FMA) は、システムで考えられる障害点を特定することによって、システムに回復力を持たせるためのプロセスです。 FMA をアーキテクチャおよび設計フェーズで行って、最初からシステムに障害復旧を組み込むことができるようにする必要があります。

FMA の一般的な実施プロセスを次に示します。

1. システム内のすべてのコンポーネントを明らかにします。 ID プロバイダーやサード パーティのサービスなど、外部の依存関係を含みます。   
2. コンポーネントごとに発生する可能性のある障害を特定します。 1 つのコンポーネントに複数の障害モードが存在する場合があります。 たとえば、影響と可能な軽減策が異なるため、読み取り障害と書き込み障害は別のものとして検討する必要があります。
3. 総合的なリスクに従って、各障害モードを評価します。 次の点を考慮します。  

   * 障害の可能性はどれくらいか。 比較的よくあることか。 非常にまれなことか。 正確な数値は必要はありません。目的は、優先順位をランク付けすることです。
   * 可用性、データの損失、金銭的コスト、および業務の中断の観点から、アプリケーションに対してどの程度の影響がありますか。
4. 各障害モードについて、アプリケーションがどのように対応および復旧するかを決定します。 コストとアプリケーションの複雑さのトレードオフを検討します。   

この記事では、FMA プロセスの出発点として、起こりうる障害モードの一覧とその軽減方法について説明します。 カタログは、テクノロジまたは Azure サービス、およびアプリケーション レベルの設計の一般的なカテゴリで分けられています。 カタログは完全ではありませんが、主要な Azure サービスの多くをカバーしています。

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>App Service アプリがシャットダウンする。
**検出**。 考えられる原因:

* 予期されたシャットダウン

  * オペレーターが、たとえば Azure Portal を使用してシャットダウンしました。
  * アプリはアイドル状態だったために、アンロードされました  (`Always On` の設定が無効の場合のみ)。
* 予期されないシャットダウン

  * アプリがクラッシュしました。
  * App Service の VM インスタンスが使用できなくなりました。

Application_End ログは、アプリ ドメインのシャットダウン (ソフト プロセス クラッシュ) をキャッチし、アプリケーション ドメインのシャットダウンをキャッチする唯一の方法です。

**復旧**

* シャットダウンが予期されていた場合は、アプリケーションのシャットダウン イベントを使用して正常にシャットダウンします。 たとえば、ASP.NET では、`Application_End` メソッドを使用します。
* アイドルの間にアプリケーションがアンロードされた場合は、次の要求で自動的に再起動されます。 ただし、"コールド スタート" コストが発生します。
* アイドル状態の間にアプリケーションがアンロードされないようにするには、Web アプリで `Always On` の設定を有効にします。 「[Azure App Service での Web アプリの構成][app-service-configure]」をご覧ください。
* オペレーターによるアプリのシャットダウンを防ぐには、`ReadOnly` レベルでリソース ロックを設定します。 詳細については、[Azure Resource Manager でのリソースのロック][rm-locks]に関するページを参照してください。
* アプリがクラッシュした場合、または App Service の VM が使用できなくなった場合は、App Service がアプリを自動的に再起動します。

**診断** アプリケーション ログと Web サーバー ログ。 「[Azure App Service の Web アプリの診断ログの有効化][app-service-logging]」を参照してください。

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>特定のユーザーが繰り返し不適切な要求を行ったり、システムを過負荷状態にする。
**検出**。 ユーザーを認証し、アプリケーション ログにユーザー ID を含めます。

**復旧**

* [Azure API Management][api-management] を使用して、ユーザーからの要求を調整します。 「[Azure API Management を使用した高度な要求スロットル][api-management-throttling]」を参照してください。
* ユーザーをブロックします。

**診断** すべての認証要求をログに記録します。

### <a name="a-bad-update-was-deployed"></a>不正な更新プログラムがデプロイされた。
**検出**。 Azure Portal を使用して (「[Azure Web アプリのパフォーマンスの監視][app-insights-web-apps]」を参照してください)、または[正常性エンドポイントの監視パターン][health-endpoint-monitoring-pattern]を実装して、アプリケーションの正常性を監視します。

**復旧**。 複数の[デプロイ スロット][app-service-slots]を使用して、最後の正常な展開にロールバックします。 詳細については、[基本的な Web アプリケーション][ra-web-apps-basic]に関するページを参照してください。

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>OpenID Connect (OIDC) 認証が失敗する。
**検出**。 次のような障害モードの可能性があります。

1. Azure AD が使用できないか、またはネットワークの問題のために到達できません。 認証エンドポイントへのリダイレクトが失敗し、OIDC ミドルウェアが例外をスローします。
2. Azure AD テナントが存在しません。 認証エンドポイントへのリダイレクトから HTTP エラー コードが返り、OIDC ミドルウェアが例外をスローします。
3. ユーザーを認証できません。 検出戦略は不要です。Azure AD がログインの失敗を処理します。

**復旧**

1. ミドルウェアからのハンドルされない例外をキャッチします。
2. `AuthenticationFailed` イベントを処理します。
3. エラー ページにユーザーをリダイレクトします。
4. ユーザーが再試行します。

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Azure Search へのデータの書き込みが失敗する。
**検出**。 `Microsoft.Rest.Azure.CloudException` エラーをキャッチします。

**復旧**

[Search .NET SDK][search-sdk] は、一時的な障害の後で自動的に再試行します。 クライアント SDK によってスローされた例外は、一時的ではないエラーとして処理する必要があります。

既定の再試行ポリシーは、指数バックオフを使用します。 異なる再試行ポリシーを使用するには、`SearchIndexClient` クラスまたは `SearchServiceClient` クラスで `SetRetryPolicy` を呼び出します。 詳細については、[自動再試行][auto-rest-client-retry]に関するページを参照してください。

**診断** [検索トラフィックの分析][search-analytics]を使用します。

### <a name="reading-data-from-azure-search-fails"></a>Azure Search からのデータの読み取りが失敗する。
**検出**。 `Microsoft.Rest.Azure.CloudException` エラーをキャッチします。

**復旧**

[Search .NET SDK][search-sdk] は、一時的な障害の後で自動的に再試行します。 クライアント SDK によってスローされた例外は、一時的ではないエラーとして処理する必要があります。

既定の再試行ポリシーは、指数バックオフを使用します。 異なる再試行ポリシーを使用するには、`SearchIndexClient` クラスまたは `SearchServiceClient` クラスで `SetRetryPolicy` を呼び出します。 詳細については、[自動再試行][auto-rest-client-retry]に関するページを参照してください。

**診断** [検索トラフィックの分析][search-analytics]を使用します。

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>ノードに対する読み取りまたは書き込みが失敗する。
**検出**。 例外をキャッチします。 .NET クライアントの場合は、通常、これは `System.Web.HttpException` です。 他のクライアントでは、例外の種類が異なる場合があります。  詳細については、「[Cassandra error handling done right](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right)」(Cassandra のエラー処理) を参照してください。

**復旧**

* 各 [Cassandra クライアント](https://wiki.apache.org/cassandra/ClientOptions)には、専用の再試行ポリシーと機能があります。 詳細については、「[Cassandra error handling done right][cassandra-error-handling]」(Cassandra のエラー処理) を参照してください。
* データ ノードがフォールト ドメイン間に分散される、ラック対応のデプロイを使用します。
* ローカル クォーラム整合性で複数のリージョンにデプロイします。 一時的ではない障害が発生した場合は、別のリージョンにフェールオーバーします。

**診断** アプリケーション ログ

## <a name="cloud-service"></a>クラウド サービス
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Web ロールまたは worker ロールが予期せずシャットダウンされる。
**検出**。 [RoleEnvironment.Stopping][RoleEnvironment.Stopping] イベントが発生します。

<strong>復旧</strong>。 [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] メソッドをオーバーライドして、正常にクリーンアップします。 詳細については、「[The Right Way to Handle Azure OnStop Events][onstop-events]」(Azure OnStop イベントを処理する正しい方法) (ブログ) を参照してください。

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>データの読み取りが失敗する。
**検出**。 `System.Net.Http.HttpRequestException` または `Microsoft.Azure.Documents.DocumentClientException` をキャッチします。

**復旧**

* SDK によって、失敗した試行が自動的にもう一度実行されます。 再試行回数と最大待機時間を設定するには、`ConnectionPolicy.RetryOptions` を構成します。 クライアントがスローする例外は、再試行ポリシーを超えているか、一時的なエラーでないかのいずれかです。
* クライアントが Cosmos DB によってスロットルされると、HTTP 429 エラーが返されます。 `DocumentClientException` で状態コードを確認します。 エラー 429 が繰り返し発生する場合は、コレクションのスループットの値を増やすことを検討してください。
    * MongoDB API を使用している場合は、調整のときに、サービスはエラー コード 16500 を返します。
* 複数のリージョンの間で Cosmos DB データベースをレプリケートします。 すべてのレプリカは読み取り可能です。 クライアント SDK を使用して、`PreferredLocations` パラメーターを指定します。 これは、Azure リージョンの順序付きリストです。 すべての読み取りは、リストで最初に利用可能なリージョンに送信されます。 要求が失敗すると、クライアントはリストのリージョンを順番に試します。 詳細については、「[SQL API を使用して Azure Cosmos DB グローバル分散をセットアップする方法][cosmosdb-multi-region]」を参照してください。

**診断** クライアント側ですべてのエラーをログに記録します。

### <a name="writing-data-fails"></a>データの書き込みが失敗する。
**検出**。 `System.Net.Http.HttpRequestException` または `Microsoft.Azure.Documents.DocumentClientException` をキャッチします。

**復旧**

* SDK によって、失敗した試行が自動的にもう一度実行されます。 再試行回数と最大待機時間を設定するには、`ConnectionPolicy.RetryOptions` を構成します。 クライアントがスローする例外は、再試行ポリシーを超えているか、一時的なエラーでないかのいずれかです。
* クライアントが Cosmos DB によってスロットルされると、HTTP 429 エラーが返されます。 `DocumentClientException` で状態コードを確認します。 エラー 429 が繰り返し発生する場合は、コレクションのスループットの値を増やすことを検討してください。
* 複数のリージョンの間で Cosmos DB データベースをレプリケートします。 プライマリ リージョンが失敗した場合、別のリージョンが書き込みにレベル上げされます。 手動でフェールオーバーをトリガーすることもできます。 SDK が自動的に検出とルーティングを行うので、アプリケーション コードはフェールオーバー後も引き続き動作します。 フェールオーバー期間中は (通常は分単位)、SDK が新しい書き込みリージョンを探しているので、書き込み操作の待機時間が長くなります。
  詳細については、「[SQL API を使用して Azure Cosmos DB グローバル分散をセットアップする方法][cosmosdb-multi-region]」を参照してください。
* フォールバックとしては、ドキュメントをバックアップ キューに保存し、後でキューを処理します。

**診断** クライアント側ですべてのエラーをログに記録します。

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Elasticsearch からのデータの読み取りが失敗する。
**検出**。 使用されている特定の [Elasticsearch クライアント][elasticsearch-client]に対する適切な例外をキャッチします。

**復旧**

* 再試行メカニズムを使用します。 各クライアントには、独自の再試行ポリシーがあります。
* 高可用性のためには、複数の Elasticsearch ノードをデプロイして、レプリケーションを使用します。

詳細については、「[Running Elasticsearch on Azure][elasticsearch-azure]」(Azure で Elasticsearch を実行する) を参照してください。

**診断** Elasticsearch 用の監視ツールを使用するか、クライアント側でペイロードを含むすべてのエラーをログに記録することができます。 「[Running Elasticsearch on Azure][elasticsearch-azure]」(Azure で Elasticsearch を実行する) の「Monitoring」(監視) セクションを参照してください。

### <a name="writing-data-to-elasticsearch-fails"></a>Elasticsearch へのデータの書き込みが失敗する。
**検出**。 使用されている特定の [Elasticsearch クライアント][elasticsearch-client]に対する適切な例外をキャッチします。  

**復旧**

* 再試行メカニズムを使用します。 各クライアントには、独自の再試行ポリシーがあります。
* アプリケーションが低下した整合性レベルを許容できる場合は、`write_consistency` の設定を `quorum` にした書き込みを検討します。

詳細については、「[Running Elasticsearch on Azure][elasticsearch-azure]」(Azure で Elasticsearch を実行する) を参照してください。

**診断** Elasticsearch 用の監視ツールを使用するか、クライアント側でペイロードを含むすべてのエラーをログに記録することができます。 「[Running Elasticsearch on Azure][elasticsearch-azure]」(Azure で Elasticsearch を実行する) の「Monitoring」(監視) セクションを参照してください。

## <a name="queue-storage"></a>Queue Storage
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Azure Queue Storage へのメッセージの書き込みが常に失敗する。
**検出**。 *N* 回再試行を試みた後も、書き込み操作がまだ失敗します。

**復旧**

* ローカル キャッシュにデータを格納しておき、後でサービスが利用可能になったら、ストレージに書き込みを転送します。
* セカンダリ キューを作成し、プライマリ キューが使用できない場合は、そのキューに書き込みます。

**診断** [ストレージ メトリック][storage-metrics]を使用します。

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>特定のキューからのメッセージをアプリケーションが処理できない。
**検出**。 アプリケーション固有です。 たとえば、メッセージに無効なデータが含まれる場合や、ビジネス ロジックが何らかの理由で失敗する場合があります。

**復旧**

別のキューにメッセージを移動します。 そのキューで別のプロセスを実行してメッセージを調べます。

Azure Service Bus メッセージング キューの使用を検討します。このキューは、この目的に対して[配信不能キュー][sb-dead-letter-queue]の機能を提供します。

> [!NOTE]
> WebJobs でストレージ キューを使用している場合、WebJobs SDK は組み込みの有害メッセージ処理を提供します。 「[Web ジョブ SDK を使用して Azure Queue Storage を操作する方法][sb-poison-message]」を参照してください。

**診断** アプリケーション ログを使用します。

## <a name="redis-cache"></a>Redis Cache
### <a name="reading-from-the-cache-fails"></a>キャッシュからの読み取りが失敗する。
**検出**。 `StackExchange.Redis.RedisConnectionException` をキャッチします。

**復旧**

1. 一時的な障害の場合は再試行します。 Azure Redis Cache は、「[Azure Redis Cache の再試行ガイドライン][redis-retry]」に従って組み込みの再試行をサポートします。
2. 一時的でない障害はキャッシュ ミスとして処理し、元のデータ ソースにフォールバックします。

**診断** [Redis Cache の診断][redis-monitor]を使用します。

### <a name="writing-to-the-cache-fails"></a>キャッシュへの書き込みが失敗する。
**検出**。 `StackExchange.Redis.RedisConnectionException` をキャッチします。

**復旧**

1. 一時的な障害の場合は再試行します。 Azure Redis Cache は、「[Azure Redis Cache の再試行ガイドライン][redis-retry]」に従って組み込みの再試行をサポートします。
2. エラーが一時的でない場合は、それを無視し、他のトランザクションが後でキャッシュに書き込めるようにします。

**診断** [Redis Cache の診断][redis-monitor]を使用します。

## <a name="sql-database"></a>SQL Database
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>プライマリ リージョンのデータベースに接続できない。
**検出**。 接続が失敗します。

**復旧**

前提条件: データベースがアクティブ geo レプリケーション用に構成されている必要があります。 「[Azure SQL Database のアクティブ geo レプリケーション][sql-db-replication]」を参照してください。

* クエリの場合は、セカンダリ レプリカから読み取ります。
* 挿入と更新の場合は、手動でセカンダリ レプリカにフェールオーバーします。 [Azure SQL Database の計画的または非計画的なフェールオーバーの開始][sql-db-failover]に関するページを参照してください。

レプリカは異なる接続文字列を使うので、アプリケーションで接続文字列を更新する必要があります。

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>クライアントが接続プール内の接続を使い切る。
**検出**。 `System.InvalidOperationException` エラーをキャッチします。

**復旧**

* 操作をやり直してください。
* 軽減計画としては、ユース ケースごとに接続プールを分離して、1 つのユース ケースがすべての接続を独占できないようにします。
* 最大接続プール数を増やします。

**診断** アプリケーション ログ。

### <a name="database-connection-limit-is-reached"></a>データベース接続の制限に達する。
**検出**。 Azure SQL Database では、同時実行 worker、ログイン、およびセッションの数が制限されています。 制限は、サービス レベルに依存します。 詳細については、「[Azure SQL Database のリソース制限][sql-db-limits]」を参照してください。

これらのエラーを検出するには、`System.Data.SqlClient.SqlException` をキャッチし、`SqlException.Number` の値で SQL エラー コードを調べます。 関連するエラー コードの一覧については、「[SQL Database クライアント アプリケーションの SQL エラー コード: データベース接続エラーとその他の問題][sql-db-errors]」を参照してください。

**復旧**。 これらのエラーは一時的なものと見なされるので、再試行すると問題が解決する可能性があります。 常にこれらのエラーが発生する場合は、データベースの拡張を検討してください。

**診断** - [sys.event_log][sys.event_log] をクエリすると、正常なデータベース接続、接続エラー、デッドロックが返ります。

* 失敗した接続に対する[警告ルール][azure-alerts]を作成します。
* [SQL Database 監査][sql-db-audit]を有効にして、失敗したログインを調べます。

## <a name="service-bus-messaging"></a>Service Bus メッセージング
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Service Bus キューからのメッセージの読み取りが失敗する。
**検出**。 クライアント SDK からの例外をキャッチします。 Service Bus 例外の基底クラスは [MessagingException][sb-messagingexception-class] です。 エラーが一時的なものである場合は、`IsTransient` プロパティが true です。

詳細については、「[Service Bus メッセージングの例外][sb-messaging-exceptions]」を参照してください。

**復旧**

1. 一時的な障害の場合は再試行します。 「[Service Bus の再試行ガイドライン][sb-retry]」を参照してください。
2. どの受信者にも配信できないメッセージは、"*配信不能キュー*" に入れられます。 このキューを使用して、受信できなかったメッセージを確認します。 配信不能キューは自動的にクリーンアップされません。 明示的に取得するまで、メッセージは残っています。 「[Service Bus の配信不能キューの概要][sb-dead-letter-queue]」を参照してください。

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Service Bus キューへのメッセージの書き込みが失敗する。
**検出**。 クライアント SDK からの例外をキャッチします。 Service Bus 例外の基底クラスは [MessagingException][sb-messagingexception-class] です。 エラーが一時的なものである場合は、`IsTransient` プロパティが true です。

詳細については、「[Service Bus メッセージングの例外][sb-messaging-exceptions]」を参照してください。

**復旧**

1. Service Bus クライアントは、一時的なエラーの後で自動的に再試行します。 既定では、指数バックオフを使用します。 最大再試行回数または最大タイムアウト期間に達した後、クライアントは例外をスローします。 詳細については、「[Service Bus の再試行ガイドライン][sb-retry]」を参照してください。
2. キューのクォータを超えた場合、クライアントは [QuotaExceededException][QuotaExceededException] をスローします。 例外メッセージを見ると、さらに詳細がわかります。 再試行の前にキューからメッセージをいくつかドレインし、サーキット ブレーカー パターンを使用してクォータを超えている間は再試行を続けないようにすることを検討します。 また、[BrokeredMessage.TimeToLive] プロパティの設定が高すぎないことを確認します。
3. 1 つのリージョン内では、[パーティション分割されたキューまたはトピック][sb-partition]を使用して回復性を向上させることができます。 パーティション分割されていないキューまたはトピックは 1 つのメッセージング ストアに割り当てられます。 このメッセージング ストアを使用できない場合、そのキューまたはトピックに対するすべての操作が失敗します。 パーティション分割されたキューまたはトピックは、複数のメッセージング ストア間で分割されます。
4. 回復性を高めるには、異なるリージョンに 2 つの Service Bus 名前空間を作成し、メッセージを複製します。 アクティブ レプリケーションまたはパッシブ レプリケーションを使用することができます。

   * アクティブ レプリケーション: クライアントは、両方のキューにすべてのメッセージを送信します。 受信側は、両方のキューでリッスンします。 メッセージに一意識別子でタグを付け、クライアントが重複したメッセージを破棄できるようにします。
   * パッシブ レプリケーション: クライアントは、メッセージを 1 つのキューに送信します。 エラーが発生した場合、クライアントは他のキューにフォールバックします。 受信側は、両方のキューでリッスンします。 この方法では、送信される重複したメッセージの数が減ります。 ただし、それでも受信側は重複したメッセージを処理する必要があります。

     詳細については、[GeoReplication サンプル][sb-georeplication-sample]に関するページ、および「[Service Bus の障害および災害に対するアプリケーションの保護のベスト プラクティス](/azure/service-bus-messaging/service-bus-outages-disasters/)」を参照してください。

### <a name="duplicate-message"></a>重複メッセージ。
**検出**。 メッセージの `MessageId` プロパティと `DeliveryCount` プロパティを調べます。

**復旧**

* 可能であれば、メッセージ処理操作をべき等になるように設計します。 できない場合は、既に処理されたメッセージのメッセージ ID を保存しておき、メッセージを処理する前に ID を確認します。
* `RequiresDuplicateDetection` を true に設定してキューを作成することで、重複の検出を有効にします。 このように設定すると、Service Bus は、前のメッセージと同じ `MessageId` で送信されたメッセージを自動的に削除します。  以下の点に注意してください。

  * この設定は、重複するメッセージがキューに格納されるのを防ぎます。 受信側が同じメッセージを複数回処理することを防ぐものではありません。
  * 重複データの検出には時間枠があります。 この期間を超えて重複が送信された場合は検出されません。

**診断** 重複するメッセージを記録します。

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>特定のキューからのメッセージをアプリケーションが処理できない。
**検出**。 アプリケーション固有です。 たとえば、メッセージに無効なデータが含まれる場合や、ビジネス ロジックが何らかの理由で失敗する場合があります。

**復旧**

2 つの障害モードを考慮する必要があります。

* 受信側が障害を検出します。 この場合、配信不能キューにメッセージを移動します。 後で、別のプロセスを実行して配信不能キューのメッセージを調べます。
* 受信側が、メッセージの処理の途中で失敗します (ハンドルされない例外など)。 このケースを処理するには、`PeekLock` モードを使用します。 このモードでは、ロックの期限が切れると、他の受信者がメッセージを処理できるようになります。 メッセージが最大配信数または Time-To-Live を超えた場合、メッセージは配信不能キューに自動的に移動されます。

詳細については、「[Service Bus の配信不能キューの概要][sb-dead-letter-queue]」を参照してください。

**診断** アプリケーションは、配信不能キューにメッセージを移動したときは常に、アプリケーション ログにイベントを書き込みます。

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>サービスに対する要求が失敗する。
**検出**。 サービスがエラーを返します。

**復旧**

* プロキシをもう一度検索し (`ServiceProxy` または `ActorProxy`)、サービス/アクター メソッドを再度呼び出します。
* **ステートフル サービス**。 トランザクションの信頼できるコレクションに操作をラップします。 エラーがある場合、トランザクションはロールバックされます。 要求は、キューからプルされた場合は再度処理されます。
* **ステートレス サービス**。 サービスが外部ストアにデータを保存する場合は、すべての操作をべき等にする必要があります。

**診断** アプリケーション ログ

### <a name="service-fabric-node-is-shut-down"></a>Service Fabric ノードがシャットダウンされる。
**検出**。 キャンセル トークンが、サービスの `RunAsync` メソッドに渡されます。 Service Fabric は、ノードをシャットダウンする前に、タスクをキャンセルします。

**復旧**。 キャンセル トークンを使用して、シャットダウンを検出します。 Service Fabric がキャンセルを要求しているときは、すべての作業を終了して、可能な限り早く  `RunAsync` を終了します。

**診断** アプリケーション ログ

## <a name="storage"></a>Storage
### <a name="writing-data-to-azure-storage-fails"></a>Azure Storage へのデータの書き込みが失敗する
**検出**。 クライアントは、書き込み時にエラーを受け取ります。

**復旧**

1. 操作を再試行し、一時的な障害から復旧します。 クライアント SDK の[再試行ポリシー][Storage.RetryPolicies]がこれを自動的に処理します。
2. ストレージに対する過剰な負荷を回避するため、サーキット ブレーカー パターンを実装します。
3. N 回再試行が失敗した場合は、正常なフォールバックを実行します。 例: 

   * ローカル キャッシュにデータを格納しておき、後でサービスが利用可能になったら、ストレージに書き込みを転送します。
   * 書き込みアクションがトランザクション スコープであった場合は、トランザクションを補正します。

**診断** [ストレージ メトリック][storage-metrics]を使用します。

### <a name="reading-data-from-azure-storage-fails"></a>Azure Storage からのデータの読み取りが失敗する。
**検出**。 クライアントは、読み取り時にエラーを受け取ります。

**復旧**

1. 操作を再試行し、一時的な障害から復旧します。 クライアント SDK の[再試行ポリシー][Storage.RetryPolicies]がこれを自動的に処理します。
2. RA-GRS ストレージでは、プライマリ エンドポイントからの読み取りが失敗した場合、セカンダリ エンドポイントから読み取りを再試行してください。 クライアント SDK はこれを自動的に処理できます。 「[Azure Storage のレプリケーション][storage-replication]」を参照してください。
3. 再試行が *N* 回失敗する場合は、フォールバック アクションを実行して正常にデグレードします。 たとえば、製品の画像をストレージから取得できない場合は、汎用的なプレースホルダー画像を表示します。

**診断** [ストレージ メトリック][storage-metrics]を使用します。

## <a name="virtual-machine"></a>仮想マシン
### <a name="connection-to-a-backend-vm-fails"></a>バックエンド VM への接続が失敗する。
**検出**。 ネットワーク接続エラー。

**復旧**

* ロード バランサーの背後の可用性セットに、少なくとも 2 つのバックエンド VM をデプロイします。
* 接続エラーが一時的な場合は、TCP が正常にメッセージの送信を再試行することがあります。
* アプリケーションに再試行ポリシーを実装します。
* 永続的なエラーまたは一時的でないエラーの場合は、[サーキット ブレーカー][circuit-breaker] パターンを実装します。
* 呼び出し元の VM がネットワーク送信制限を超えた場合は、送信キューがいっぱいになります。 送信キューが常にいっぱいになっている場合は、スケールアウトを検討してください。

**診断** サービスの境界でイベントのログを記録します。

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>VM インスタンスが使用不可または異常になる。
**検出**。 VM インスタンスが正常かどうかを通知する Load Balancer [正常性プローブ][lb-probe]を構成します。 プローブは、重要な機能が正しく応答しているかどうかを確認する必要があります。

**復旧**。 アプリケーション層ごとに、複数の VM インスタンスを同じ可用性セットに格納し、VM の前にロード バランサーを配置します。 正常性プローブが失敗した場合、Load Balancer は異常なインスタンスへの新しい接続の送信を停止します。

**診断** - Load Balancer の [Log Analytics][lb-monitor] を使用します。

* すべての正常性監視エンドポイントを監視するように、監視システムを構成します。

### <a name="operator-accidentally-shuts-down-a-vm"></a>オペレーターが誤って VM をシャットダウンした。
**検出**。 該当なし

**復旧**。 リソース ロックを `ReadOnly` レベルに設定します。 詳細については、[Azure Resource Manager でのリソースのロック][rm-locks]に関するページを参照してください。

**診断** [Azure Activity Logs][azure-activity-logs] を使用します。

## <a name="webjobs"></a>Web ジョブ
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>SCM ホストがアイドル状態のとき、継続的なジョブが実行を停止する。
**検出**。 WebJob 関数にキャンセル トークンを渡します。 詳細については、[グレースフル シャットダウン][web-jobs-shutdown]に関するページを参照してください。

**復旧**。 Web アプリで `Always On` の設定を有効にします。 詳細については、[WebJobs でのバックグラウンド タスクの実行][web-jobs]に関するページを参照してください。

## <a name="application-design"></a>アプリケーションの設計
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>アプリケーションが受信要求のスパイクを処理できない。
**検出**。 アプリケーションによって異なります。 一般的な現象:

* Web サイトが HTTP 5xx エラー コードを返し始めます。
* データベースやストレージなどの依存サービスが、要求の調整を始めます。 サービスに応じて、HTTP 429 (要求数が多すぎる) などの HTTP エラーを探します。
* HTTP キューが長くなりすぎます。

**復旧**

* 増加した負荷を処理するためにスケールアウトします。
* エラーを軽減し、障害が連鎖してアプリケーション全体が停止するのを防ぎます。 次のような軽減戦略があります。

  * [調整パターン][throttling-pattern]を実装して、バックエンド システムの過負荷を回避します。
  * [キュー ベースの負荷の平準化][queue-based-load-leveling]を使用して、要求をバッファリングし、適切な速度で処理します。
  * 特定のクライアントを優先します。 たとえば、アプリケーションに無料レベルと有料レベルがある場合、無料レベルの顧客を調整し、有料レベルの顧客は調整しないようにします。 「[Priority Queue パターン][priority-queue-pattern]」を参照してください。

**診断** [App Service 診断ログ][app-service-logging]を使用します。 [Azure Log Analytics][azure-log-analytics]、[Application Insights][app-insights]、[New Relic][new-relic] などのサービスを使用して、診断ログを理解します。

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>ワークフローまたは分散トランザクションの操作の 1 つが失敗する。
**検出**。 *N* 回再試行した後で、まだ失敗します。

**復旧**

* 軽減計画としては、[スケジューラー エージェント スーパーバイザー][scheduler-agent-supervisor] パターンを実装して、ワークフロー全体を管理します。
* タイムアウト時に再試行しないでください。 このエラーの場合、成功する確率は高くありません。
* 後で再試行するため、キューに作業を格納します。

**診断** 補正アクションなど、(成功と失敗を含む) すべての操作をログに記録します。 相関 ID を使用して、同じトランザクション内のすべての操作を追跡できるようにします。

### <a name="a-call-to-a-remote-service-fails"></a>リモート サービスの呼び出しが失敗する。
**検出**。 HTTP エラー コード。

**復旧**

1. 一時的な障害の場合は再試行します。
2. *N* 回試みても呼び出しが失敗する場合は、フォールバック アクションを実行します。 (アプリケーション固有。)
3. [サーキット ブレーカー パターン][circuit-breaker]を実装し、障害の連鎖を回避します。

**診断** リモート呼び出しのすべてのエラーをログに記録します。

## <a name="next-steps"></a>次の手順
FMA プロセスについては、「[Resilience by design for cloud services][resilience-by-design-pdf]」(クラウド サービス向けの設計による回復力) を参照してください (PDF のダウンロード)。

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
