---
title: 不適切なインスタンス化のアンチパターン
titleSuffix: Performance antipatterns for cloud apps
description: 作成後に共有する予定のオブジェクトの新しいインスタンスを頻繁に作成することは避けてください。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a2e42e35ae1b56b61c8f9f9ecb21ee104cd3222e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346350"
---
# <a name="improper-instantiation-antipattern"></a>不適切なインスタンス化のアンチパターン

作成後に共有する予定のオブジェクトの新しいインスタンスを頻繁に作成すると、パフォーマンスが低下する可能性があります。

## <a name="problem-description"></a>問題の説明

多くのライブラリでは、外部リソースの抽象化が提供されます。 内部的には、これらのクラスは、通常、リソースへの独自の接続を管理して、クライアントがリソースへのアクセスに使用できるブローカーとして機能します。 Azure アプリケーションに関連するブローカー クラスのいくつかの例を次に示します。

- `System.Net.Http.HttpClient` HTTP を使用して Web サービスと通信します。
- `Microsoft.ServiceBus.Messaging.QueueClient` Service Bus キューとの間でメッセージを送受信します。
- `Microsoft.Azure.Documents.Client.DocumentClient` Cosmos DB インスタンスに接続します
- `StackExchange.Redis.ConnectionMultiplexer` Azure Redis Cache を含む Redis に接続します。

これらのクラスは、一度インスタンス化された後、アプリケーションの有効期間にわたって再利用されることが意図されています。 ただし、"これらのクラスは必要なときにのみ取得し、すぐに解放する必要がある" という考えはよくある誤解です  (ここに示されているのは .NET ライブラリですが、そのパターンは .NET に固有のものではありません)。次の ASP.NET の例では、リモート サービスと通信するために `HttpClient` のインスタンスを作成しています。 完全なサンプルは、[こちら][sample-app]でご覧いただけます。

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

Web アプリケーションでは、この手法はスケーラブルではありません。 それぞれのユーザー要求に対して新しい `HttpClient` オブジェクトが作成されます。 負荷が大きい場合、Web サーバーが使用可能な数のソケットを使い果たした結果、`SocketException` エラーが発生する可能性があります。

この問題は `HttpClient` クラスに限定されません。 同様の問題は、リソースをラップする他のクラスや作成するのが高価な他のクラスでも発生する可能性があります。 次の例では、`ExpensiveToCreateService` クラスのインスタンスを作成しています。 ここでの問題は必ずしもソケットの枯渇ではなく、各インスタンスの作成に要する時間です。 このクラスのインスタンスの作成と破棄を頻繁に繰り返すと、システムのスケーラビリティに悪影響が及ぶ可能性があります。

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>問題の解決方法

外部リソースをラップするクラスが共有可能かつスレッドセーフである場合は、クラスの共有シングルトン インスタンスまたは再利用可能なインスタンスのプールを作成します。

次の例では、静的な `HttpClient` インスタンスを使用しているため、すべての要求で接続を共有しています。

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>考慮事項

- このアンチパターンの重要な要素は、"*共有可能*" オブジェクトのインスタンスの作成と破棄を繰り返していることです。 クラスが共有可能でない (スレッドセーフでない) 場合、このアンチパターンは当てはまりません。

- 共有リソースの種類によって、シングルトンを使用すべきか、プールを作成すべきかが決まります。 `HttpClient` クラスは、プールされるのではなく共有されるように設計されています。 他のオブジェクトでは、プーリングをサポートして、システムによる複数のインスタンスへのワークロードの分散を可能にすることができます。

- 複数の要求で共有されるオブジェクトは、スレッドセーフである "*必要があります*"。 `HttpClient` クラスはこのように使用されるように設計されています。しかし、他のクラスは同時要求をサポートしない可能性があるため、参照可能なドキュメントを確認してください。

- 共有のオブジェクトへのプロパティの設定では、競合状態になる可能性があるため、注意が必要です。 たとえば、各要求の前に `HttpClient` クラスに `DefaultRequestHeaders` を設定すると、競合状態が発生する場合があります。 このようなプロパティは一度 (起動時などに) 設定し、別の設定を行う必要が生じたら、個別のインスタンスを作成します。

- リソースの種類によっては十分でないものもあり、このようなリソースは保持すべきではありません。 データベース接続はその一例です。 開かれたデータベース接続を必要でないのに保持すると、他の同時ユーザーがデータベースにアクセスできなくなる可能性があります。

- .NET Framework では、外部リソースへの接続を確立する多くのオブジェクトは、これらの接続を管理する他のクラスの静的ファクトリ メソッドを使用して作成されます。 これらのオブジェクトは、廃棄した後で再び作成するのではなく、保存して再利用することが意図されています。 たとえば、Azure Service Bus では、`QueueClient` オブジェクトは `MessagingFactory` オブジェクトを通じて作成されます。 内部的には、`MessagingFactory` が接続を管理します。 詳細については、「[Service Bus メッセージングを使用したパフォーマンス向上のためのベスト プラクティス][service-bus-messaging]」を参照してください。

## <a name="how-to-detect-the-problem"></a>問題の検出方法

この問題の現象としては、次の 1 つまたは複数の現象に加えて、スループットの低下またはエラー率の増加が挙げられます。

- ソケット、データベース接続、ファイル ハンドルなどのリソースの枯渇を示す例外の増加。
- メモリ使用量とガベージ コレクションの増加。
- ネットワーク、ディスク、またはデータベース アクティビティの増加。

この問題の識別に役立てるために、次の手順を実行できます。

1. 運用システムのプロセス監視を実行して、応答速度が低下したポイントや、リソース不足が原因でシステムにエラーが発生したポイントを識別します。
2. これらのポイントでキャプチャされたテレメトリ データを調べて、リソースを大量に消費するオブジェクトを作成および破棄している可能性のある操作を判定します。
3. 運用システムではなく、管理されたテスト環境で、疑わしい操作のそれぞれに対してロード テストを実行します。
4. ソース コードを調べて、ブローカー オブジェクトの管理方法を確認します。

スタック トレースを調べて、システムに負荷がかかっているときに実行が低速になる操作や例外を生成する操作を確認します。 この情報は、これらの操作がどのようにリソースを利用しているかを識別するのに役立ちます。 例外は、共用リソースの枯渇が原因でエラーが発生しているかどうかを判断するのに役立ちます。

## <a name="example-diagnosis"></a>診断の例

以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。

### <a name="identify-points-of-slow-down-or-failure"></a>速度低下またはエラーのポイントを識別する

次の画像は、[New Relic APM][new-relic] を使用して生成された結果を示しています。応答時間が低速な操作が示されています。 この場合、`NewHttpClientInstancePerRequest` コントローラーの `GetProductAsync` メソッドをさらに調査する価値があります。 これらの操作が実行されているときにエラー率も増加していることに注意してください。

![New Relic モニター ダッシュボードに表示された、要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーション][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>テレメトリ データを調べて相関関係を見つける

次の画像は、前の画像に対応する同じ期間にスレッド プロファイリングを使用してキャプチャされたデータを示しています。 システムは、ソケット接続を開くためにかなりの時間を費やしているだけでなく、ソケット接続を閉じたりソケット例外を処理したりするためにそれ以上の時間を費やしています。

![New Relic スレッド プロファイラーに表示された、要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーション][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>ロード テストを実行する

ロード テストを使用して、ユーザーが実行する典型的な操作をシミュレートします。 これは、変化する負荷に対し、システムのどの部分でリソースが枯渇するかを識別するのに役立ちます。 これらのテストは、運用システムではなく管理された環境で実行します。 次のグラフは、ユーザー負荷が 100 同時ユーザーに増えたときに `NewHttpClientInstancePerRequest` コントローラーによって処理される要求のスループットを示しています。

![要求ごとに HttpClient オブジェクトの新しいインスタンスが作成されるサンプル アプリケーションのスループット][throughput-new-HTTPClient-instance]

最初は、ワークロードが増加するにつれて、1 秒あたりに処理される要求の量が増えます。 しかし、ユーザー数が約 30 人になると、成功する要求の量が限度に達し、システムが例外を生成し始めます。 それ以後、ユーザーの負荷に応じて例外の量が徐々に増えます。

ロード テストでは、これらのエラーが HTTP 500 (内部サーバー) エラーとして報告されています。 テレメトリを調べると、これらのエラーの原因は、`HttpClient` オブジェクトが作成され続けた結果、システムのソケット リソースが不足したことにあることがわかります。

次のグラフは、カスタム `ExpensiveToCreateService` オブジェクトを作成するコントローラーに対する同様のテストを示しています。

![要求ごとに ExpensiveToCreateService の新しいインスタンスが作成されるサンプル アプリケーションのスループット][throughput-new-ExpensiveToCreateService-instance]

今回、コントローラーで例外は生成されません。しかし、平均応答時間が 20 倍になっている一方でスループットはまだ横ばい状態になったままです  (グラフでは、応答時間とスループットに対数スケールが使用されています)。テレメトリから、`ExpensiveToCreateService` の新しいインスタンスを作成することが問題の主な原因であることがわかります。

### <a name="implement-the-solution-and-verify-the-result"></a>ソリューションを実装して結果を検証する

1 つの `HttpClient` インスタンスを共有するように `GetProductAsync` メソッドを切り替えたところ、2 番目のロード テストでパフォーマンスの向上が見られました。 エラーは報告されておらず、システムは 1 秒あたり最大 500 要求の高い負荷を処理することができました。 平均応答時間は、以前のテストと比較して半分に短縮されました。

![要求ごとに HttpClient オブジェクトの同じインスタンスが再利用されるサンプル アプリケーションのスループット][throughput-single-HTTPClient-instance]

比較のために、スタック トレース テレメトリを示す次の画像をご覧ください。 今回、システムのほとんどの時間は、ソケットを開く操作と閉じる操作ではなく、実際の作業に費やされています。

![New Relic スレッド プロファイラーに表示された、すべての要求に対して HttpClient オブジェクトの単一のインスタンスが作成されるサンプル アプリケーション][thread-profiler-single-HTTPClient-instance]

次のグラフは、`ExpensiveToCreateService` オブジェクトの共有インスタンスを使用した場合の同様のロード テストを示しています。 この場合も、処理された要求の量はユーザーの負荷に応じて増加していますが、平均応答時間は低いままです。

![要求ごとに HttpClient オブジェクトの同じインスタンスが再利用されるサンプル アプリケーションのスループット][throughput-single-ExpensiveToCreateService-instance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
