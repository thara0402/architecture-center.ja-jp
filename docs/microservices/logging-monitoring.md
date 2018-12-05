---
title: マイクロサービスのログ記録と監視
description: マイクロサービスのログ記録と監視
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 9d385a141edb34b2b0f4badb7dfcaf53baac2666
ms.sourcegitcommit: 1b5411f07d74f0a0680b33c266227d24014ba4d1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/26/2018
ms.locfileid: "52305912"
---
# <a name="designing-microservices-logging-and-monitoring"></a>マイクロサービスの設計: ログ記録と監視

複雑なアプリケーションは、どこかの時点で何かがうまくいかなくなります。 マイクロサービス アプリケーションでは、数十から数百のサービス全体で何が起こっているのかを追跡する必要があります。 システムの全体像を把握するには、ログ記録と監視が非常に重要です。 

![](./images/monitoring.png)

マイクロサービス アーキテクチャでは、エラーやパフォーマンスのボトルネックの正確な原因を突き止めることが特に難しい場合があります。 1 つのユーザー操作が複数のサービスにまたがる場合があります。 サービスは、クラスター内のネットワーク I/O 制限に達することがあります。 サービスをまたがる一連の呼び出しによって、システムのバックプレッシャが発生し、結果として待機時間が長くなったり、連鎖するエラーが発生したりする可能性があります。 また、一般に、どのコンテナーがどのノードで実行されるかはわかりません。 同じノードに配置されたコンテナーが、限られた CPU またはメモリについて競合する可能性があります。 

何が起きているのかを把握するには、アプリケーションからテレメトリを収集する必要があります。  テレメトリは "*ログ*" と "*メトリック*" に分けられます。 [Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview) では、Azure プラットフォーム全体でログとメトリックの両方が収集されます。

**ログ**は、アプリケーションの実行中に発生するイベントに関するテキストベースの記録です。 アプリケーション ログ (トレース ステートメント) や Web サーバー ログなどが含まれます。 ログは、主に科学捜査と根本原因の解析に役立ちます。 

**メトリック**は、解析できる数値です。 メトリックを使用すると、リアルタイムで (またはほぼリアルタイムで) システムを観察したり、時間の経過と共にパフォーマンスの傾向を解析したりすることができます。 メトリックは、さらに次のように分類できます。

- **ノードレベル** メトリック。CPU、メモリ、ネットワーク、ディスク、ファイル システムの使用状況などです。 システム メトリックは、クラスター内の各ノードのリソース割り当てを把握し、異常値を解決するために役立ちます。

- **コンテナー** メトリック。 サービスがコンテナーで実行されている場合、VM レベルだけでなく、コンテナー レベルでメトリックを収集する必要があります。 Azure Monitor をセットアップすることで、Azure Kubernetes Service (AKS) のコンテナー ワークロードを監視できます。 詳細については、「[コンテナーに対する Azure Monitor の概要](/azure/monitoring/monitoring-container-insights-overview)」を参照してください。 その他のコンテナー オーケストレーターについては、[Log Analytics のコンテナー監視ソリューション](/azure/log-analytics/log-analytics-containers)を使用します。

- **アプリケーション** メトリック。 サービスの動作の把握に関連するメトリックが含まれます。 たとえば、キューに入ったインバウンド HTTP 要求の数、要求の待機時間、メッセージ キューの長さなどです。 アプリケーションでは、1 分あたりに処理されたビジネス トランザクションの数など、ドメインに固有のカスタム メトリックを作成することもできます。 [アプリケーション メトリック](/azure/application-insights/app-insights-overview)を有効にするには、Application Insights を使用します。 

- **依存サービス** メトリック。 サービスからは、マネージド PaaS サービスや SaaS サービスなど、外部サービスまたはエンドポイントが呼び出されることがあります。 サードパーティのサービスの場合、メトリックが提供される場合と提供されない場合があります。 提供されない場合、待機時間とエラー率の統計を追跡するには、独自のアプリケーション メトリックに頼る必要があります。

## <a name="considerations"></a>考慮事項

[監視と診断](../best-practices/monitoring.md)に関する記事では、アプリケーションの監視に関する全般的なベスト プラクティスについて説明しています。 ここでは、マイクロサービス アーキテクチャのコンテキストで考慮すべき特別な事項について説明します。

**構成と管理**。 ログ記録と監視にマネージド サービスを使用しますか。それともログと監視コンポーネントをクラスター内のコンテナーとして展開しますか。 このような選択肢の詳細については、後述の「[テクノロジの選択肢](#technology-options)」セクションを参照してください。

**取り込み率**。 システムがテレメトリ イベントを取り込ことができるスループットはどれくらいですか。 その率を超えるとどうなりますか。 たとえば、システムがクライアントを制限する方法があります。この場合は、テレメトリ データが失われます。また、データをダウンサンプリングする方法があります。 場合によっては、次のように収集するデータ量を減らすことで問題を軽減できます。

  - メトリックの集計。平均や標準偏差などの統計を計算し、その統計データを監視システムに送信します。  

  - データのダウンサンプリング。つまり、イベントの一部のみを処理します。

  - データの一括処理。監視サービスに対するネットワーク呼び出しの回数を減らします。

**コスト**。 テレメトリ データを取り込んで保存するコストは高くなる可能性があります。大量のデータの場合は特にそうです。 場合によっては、アプリケーションの実行コストを上回る可能性もあります。 その場合、前述のように、データの集計、ダウンサンプリング、または一括処理によってテレメトリの量を減らす必要が出てくることもあります。 
        
**データの忠実性**。 メトリックはどのくらい正確ですか。 大量のデータの場合、平均値で異常値を隠すことができます。 また、サンプリング レートが低すぎると、データの変動が滑らかになる可能性があります。 実際、要求の大部分の処理時間がはるかに長い場合、エンドツーエンドの待機時間がすべての要求でほぼ同じに見えるようになることがあります。 

**待機時間**。 リアルタイムの監視とアラートを可能にするには、テレメトリ データをすばやく利用できるようにする必要があります。 どのくらい前のデータが監視ダッシュボードに表示されると、"リアルタイム" と言えるでしょうか。 数秒前でしょうか。 それとも 1 分よりも前でしょうか。

**ストレージ。** ログについては、ログ イベントをクラスター内の一時ストレージに書き込み、ログ ファイルをより永続的なストレージに送るようにエージェントを構成するのが最も効率的です。  データは最終的には長期保存に移行し、遡及解析に利用できるようにする必要があります。 マイクロサービス アーキテクチャは大量のテレメトリ データを生成する可能性があるため、そのデータを保存するコストは重要な考慮事項です。 また、データのクエリ方法も考慮します。 

**ダッシュボードと視覚化。** クラスターと外部サービスの両方で、すべてのサービスにわたってシステムの全体像を把握していますか。 テレメトリ データとログを複数の場所に書き込んでいる場合は、ダッシュボードでそれらのすべてを表示して相互に関連付けることはできますか。 監視ダッシュボードには、少なくとも次の情報が表示されている必要があります。

- 容量と成長のための全体的なリソース割り当て。 コンテナーの数、ファイル システムのメトリック、ネットワーク、およびコアの割り当てが含まれます。
- コンテナー メトリックはサービス レベルで相関しています。
- コンテナーに関連付けられたシステム メトリック。
- サービス エラーと異常値。
    

## <a name="distributed-tracing"></a>分散トレース

前述したように、マイクロサービスの課題の 1 つは、サービス間のイベントの流れを把握することです。 1 つの操作またはトランザクションで複数のサービスへの呼び出しが実行される場合もあります。 一連の手順全体を再構成するには、各サービスがその操作の一意の識別子として機能する*相関 ID* を伝達する必要があります。 相関 ID で、サービス全体の[分散トレース](https://microservices.io/patterns/observability/distributed-tracing.html)が可能になります。

クライアント要求を受信する最初のサービスで、相関 ID を生成する必要があります。 そのサービスが別のサービスに HTTP 呼び出しを行う場合、要求ヘッダーに相関 ID を挿入します。 同様に、サービスが非同期メッセージを送信する場合も、相関 ID をメッセージに挿入します。 システム全体に流れるように、ダウンストリームのサービスは相関 ID を伝達し続けます。 さらに、アプリケーション メトリックまたはログ イベントを書き込むすべてのコードにも相関 ID を含めるようにします。

サービス呼び出しが相互に関連する場合、完全なトランザクションのエンドツーエンドの待機時間、1 秒あたりの成功したトランザクション数、失敗したトランザクションの割合などの操作メトリックを計算できます。 相関 ID をアプリケーション ログに含めることで、根本原因の解析を実行することができます。 操作が失敗した場合、同じ操作に含まれるすべてのサービス呼び出しのログ ステートメントを見つけることができます。 

分散トレースを実装する場合は、次の点を考慮します。

- 現時点では、相関 ID の標準 HTTP ヘッダーはありません。 チームでカスタム ヘッダー値を標準化する必要があります。 ログ/監視フレームワークまたはサービス メッシュの選択によって選択することができます。

- 非同期メッセージで、メッセージング インフラストラクチャがメッセージへのメタデータの追加をサポートしている場合は、相関 ID をメタデータとして含める必要があります。 サポートしていない場合は、メッセージ スキーマに含めます。

- 単一の非透過識別子ではなく、呼び出し元と呼び出し先の関係など、より豊富な情報を含む*相関コンテキスト*を送信する場合があります。 

- Azure Application Insights SDK では、HTTP ヘッダーに相関コンテキストを挿入する処理と、Application Insights ログに相関 ID を含める処理が自動的に行われます。 Application Insights に組み込まれている相関機能を使用する場合、使用するライブラリに応じて、相関ヘッダーを明示的に伝達する必要があるサービスもあります。 詳細については、「[Application Insights におけるテレメトリの相関付け](/azure/application-insights/application-insights-correlation)」を参照してください。
   
- サービス メッシュとして Istio または linkerd を使用している場合、HTTP メッセージがサービス メッシュ プロキシ経由でルーティングされるときに、これらのテクノロジで相関ヘッダーが自動的に生成されます。 サービスは、関連するヘッダーを転送する必要があります。 

    - Istio: [分散要求トレース](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
    
    - linkerd:[コンテキスト ヘッダー](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)
    
- ログをどのように集計するかを検討します。 ログに相関 ID を含める方法についてチーム間で標準化することもできます。 JSON などの構造化または半構造化形式を使用し、相関 ID を保持する共通フィールドを定義します。

## <a name="technology-options"></a>テクノロジの選択肢

**Application Insights** は、テレメトリ データを取り込んで保存する Azure のマネージド サービスです。データの解析と検索のためのツールが用意されています。 Application Insights を使用するには、アプリケーションにインストルメンテーション パッケージをインストールします。 このパッケージは、アプリケーションを監視し、テレメトリ データを Application Insights サービスに送信します。 また、テレメトリ データをホスト環境からプルすることもできます。 Application Insights には、相関関係と依存関係の追跡機能が組み込まれています。 そのため、システム メトリック、アプリケーション メトリック、および Azure サービス メトリックをすべて 1 か所で追跡できます。

データ レートが上限を超えた場合、Application Insights は制限されることに注意してください。詳細については、「[Application Insights の制限](/azure/azure-subscription-service-limits#application-insights-limits)」を参照してください。 1 回の操作で複数のテレメトリ イベントが生成されることがあります。そのため、アプリケーションで大量のトラフィックが発生すると、そのイベントが制限される可能性があります。 この問題を軽減するために、サンプリングを実行してテレメトリ トラフィックを減らすことができます。 その代わり、メトリックの精度は低くなります。 詳細については、「[Application Insights におけるサンプリング](/azure/application-insights/app-insights-sampling)」を参照してください。 事前にメトリックを集計してデータ量を減らすこともできます。つまり、平均や標準偏差などの統計値を計算し、生テレメトリの代わりにそれらの値を送信する方法です。 ブログ投稿「[Azure Monitoring and Analytics at Scale](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/)」(大規模な Azure の監視と解析) では、Application Insights を大規模に使用するアプローチについて説明しています。

また、データ量に基づいて課金されるため、Application Insights の価格モデルも把握する必要があります。 詳細については、「[Application Insights での価格とデータ ボリュームの管理](/azure/application-insights/app-insights-pricing)」を参照してください。 アプリケーションから大量のテレメトリが生成され、データのサンプリングまたは集計を実行しない場合は、Application Insights が適切な選択肢ではない可能性があります。 

Application Insights がお客様の要件を満たしていない場合は、一般的なオープンソース テクノロジを使用するアプローチをいくつか提案します。

システムとコンテナーのメトリックについては、クラスター内で実行されている **Prometheus** や **InfluxDB** などの時系列データベースにメトリックをエクスポートすることを検討してください。

- InfluxDB はプッシュベースのシステムです。 エージェントはメトリックをプッシュする必要があります。 [Heubester][heapster] を使用することもできます。Heapster は、kubelet からクラスター全体のメトリックを収集し、データを集計し、InfluxDB または他の時系列ストレージ ソリューションにプッシュするサービスです。 Azure Container Service はクラスター設定の一環として Heapster をデプロイしています。 もう 1 つの選択肢は [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) です。Telegraf はメトリクスの収集と報告のためのエージェントです。 

- Prometheus はプルベースのシステムです。 設定された場所から定期的にメトリックを収集します。 Prometheus は、cAdvisor または kube-state-metrics によって生成されたメトリクスを取得することができます。 [kube-state-metrics][kube-state-metrics] は、Kubernetes API サーバーからメトリックを収集し、Prometheus (または Prometheus クライアント エンドポイントと互換性のある取得ツール) で使用できるようにするサービスです。 Heapster は、Kubernetes が生成したメトリックを集計してシンクに転送します。一方、kube-state-metrics は、独自のメトリックを生成し、それらをエンドポイントから取得に利用できるようにします。 システム メトリックについては、[Node exporter](https://github.com/prometheus/node_exporter) を使用します。これは、システム メトリックの Prometheus 用エクスポート ツールです。 Prometheus は浮動小数点データをサポートしますが、文字列データはサポートしないため、システム メトリックには適していますが、ログには適していません。

- データを視覚化して監視するには、**Kibana** や **Grafana** などのダッシュボード ツールを使用します。 ダッシュボード サービスは、クラスターのコンテナー内で実行することもできます。

アプリケーション ログについては、**Fluentd** と **Elasticsearch** の使用を検討してください。 Fluentd は、オープン ソースのデータ コレクターです。Elasticsearch は、検索エンジンとして機能するように最適化されたドキュメント データベースです。 このアプローチを使用すると、各サービスは `stdout` と `stderr` にログを送信し、Kubernetes はこれらのストリームをローカル ファイル システムに書き込みます。 Fluentd はログを収集し、必要に応じて Kubernetes の追加のメタデータを使用してログを補足し、ログを Elasticsearch に送信します。 Elasticsearch のダッシュボードを作成するには、Kibana、Grafana などのツールを使用します。 Fluentd はクラスター内のデーモンセットとして実行されるので、1 つの Fluentd ポッドが各ノードに確実に割り当てられます。 kubelet ログとコンテナーログを収集するように Fluentd を設定することもできます。 ログが大量になると、特に複数のサービスが同じノード上で実行されている場合は、ログをローカル ファイル システムに書き込むことがパフォーマンスのボトルネックになる可能性があります。 ディスクの待機時間とファイル システムの使用率は、実稼働環境で監視してください。

ログ用に Elasticsearch で Fluentd を使用するメリットの 1 つは、サービスにライブラリの依存関係を追加する必要がないことです。 各サービスから `stdout` と `stderr` に書き込むだけで、Fluentd からログが Elasticsearch にエクスポートされます。 また、サービスを作成するチームが、ログ記録インフラストラクチャを構成する方法を理解する必要はありません。 1 つの課題は、Elasticsearch クラスターを実稼働環境向けに構成して、トラフィックを処理するように拡張することです。 

もう 1 つの選択肢は、ログを Operations Management Suite (OMS) Log Analytics に送信する方法です。 [Log Analytics][log-analytics] サービスは、ログ データを中央のリポジトリに収集します。また、アプリケーションが使用する他の Azure サービスからのデータを統合することもできます。 詳細については、「[Microsoft Operations Management Suite (OMS) を使用して Azure Container Service クラスターを監視する][k8s-to-oms]」を参照してください。

## <a name="example-logging-with-correlation-ids"></a>例: 相関 ID を使用したログ記録

この章で取り上げるいくつかの点を説明するために、パッケージ サービスがログ記録を実装する方法の拡張例を紹介します。 パッケージ サービスは TypeScript で記述され、Node.js 用の [Koa](https://koajs.com/) Web フレームワークを使用します。 いくつかの Node.js ログ記録ライブラリから選択できます。 ここでは [Winston](https://github.com/winstonjs/winston) を選びました。Winston は、私たちが行ったテストのパフォーマンス要件を満たした人気のあるログ記録ライブラリです。

この例では、実装の詳細をカプセル化するために、抽象 `ILogger` インターフェイスを定義しました。

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

Winston ライブラリをラップする `ILogger` 実装例を以下に示します。 相関 ID はコンストラクター パラメーターとして使用され、すべてのログメッセージに ID が挿入されます。 

```ts
class WinstonLogger implements ILogger {
    constructor(private correlationId: string) {}
    log(level: string, msg: string, payload?: any) {
        var meta : any = {};
        if (payload) { meta.payload = payload };
        if (this.correlationId) { meta.correlationId = this.correlationId }
        winston.log(level, msg, meta)
    }
  
    info(msg: string, payload?: any) {
        this.log('info', msg, payload);
    }
    debug(msg: string, payload?: any) {
        this.log('debug', msg, payload);
    }
    warn(msg: string, payload?: any) {
        this.log('warn', msg, payload);
    }
    error(msg: string, payload?: any) {
        this.log('error', msg, payload);
    }
}
```

パッケージ サービスは、HTTP 要求から相関 ID を抽出する必要があります。 たとえば、linkerd を使用している場合、相関 ID は `l5d-ctx-trace` ヘッダーにあります。 Koa の場合、HTTP 要求は、リクエスト処理パイプラインを経由して Context オブジェクトに格納されます。 Context から相関 ID を取得し、ロガーを初期化するミドルウェア関数を定義できます (Koa のミドルウェア関数は、単に要求ごとに実行される関数です)。

```ts
export type CorrelationIdFn = (ctx: Context) => string;

export function logger(level: string, getCorrelationId: CorrelationIdFn) {
    winston.configure({ 
        level: level,
        transports: [new (winston.transports.Console)()]
        });
    return async function(ctx: any, next: any) {
        ctx.state.logger = new WinstonLogger(getCorrelationId(ctx));
        await next();
    }
}
```

このミドルウェアは、呼び出し側が定義した関数 `getCorrelationId` を呼び出して相関 ID を取得します。 次に、ロガーのインスタンスを作成し、それを `ctx.state` 内に配置します。これは、Koa でパイプラインを介して情報を渡すために使用されるキーと値のディクショナリです。 

ロガー ミドルウェアは、起動時にパイプラインに追加されます。

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

すべての構成が完了したら、ログ記録ステートメントを簡単にコードに追加できます。 たとえば、パッケージを検索する方法を次に示します。 `ILogger.info` メソッドが 2 回呼び出されます。

```ts
async getById(ctx: any, next: any) {
  var logger : ILogger = ctx.state.logger;
  var packageId = ctx.params.packageId;
  logger.info('Entering getById, packageId = %s', packageId);

  await next();

  let pkg = await this.repository.findPackage(ctx.params.packageId)

  if (pkg == null) {
    logger.info(`getById: %s not found`, packageId);
    ctx.response.status= 404;
    return;
  }

  ctx.response.status = 200;
  ctx.response.body = this.mapPackageDbToApi(pkg);
}
```

ログ記録ステートメントには相関 ID を含める必要はありません。これは、ミドルウェア関数によって自動的に行われるためです。 その結果、ログ記録コードが洗練され、開発者が相関 ID を含めることを忘れる可能性が少なくなります。 また、すべてのログ記録ステートメントは抽象 `ILogger` インターフェイスを使用するため、後でロガーの実装を簡単に置き換えることができます。

> [!div class="nextstepaction"]
> [継続的インテグレーションと配信](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/
