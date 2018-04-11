---
title: ビジー状態のフロントエンドのアンチパターン
description: 多数のバックグラウンド スレッドで非同期処理が実行されることによって、フォアグラウンドで実行される他のタスクのリソースが逼迫する場合があります。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a>ビジー状態のフロントエンドのアンチパターン

多数のバックグラウンド スレッドで非同期処理が実行されることによって、フォアグラウンドで実行される他の同時実行タスクのリソースが逼迫し、容認できないレベルにまで応答時間が悪化する場合があります。

## <a name="problem-description"></a>問題の説明

リソースを集中的に使用するタスクは、ユーザーの要求に対する応答時間の増加を招き、長い待ち時間が発生する原因となる場合があります。 応答時間を改善する 1 つの方法としては、リソースを集中的に使用するタスクの負荷を別のスレッドに移すことが考えられます。 この方法により、バックグラウンドで処理が実行されている間も、アプリケーションの応答性を確保することができます。 しかしバックグラウンド スレッドで実行されるタスクは依然としてリソースを消費します。 それらが多すぎると、要求を処理しているスレッドがリソース不足に陥る可能性があります。

> [!NOTE]
> "*リソース*" は意味の広い言葉であり、CPU 使用率、メモリ占有率、ネットワーク I/O、ディスク I/O などさまざまな事柄が含まれます。

この問題が起こる典型的な状況は、プレゼンテーション レイヤーとの間で共有される単一の層にすべてのビジネス ロジックを組み込んだモノリシックなコードとしてアプリケーションが開発されているときです。

以下この問題を ASP.NET を使用した例で説明します。 完全なサンプルは、[こちら][code-sample]でご覧いただけます。

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- `WorkInFrontEnd` コントローラーの `Post` メソッドは、HTTP POST 操作を実装しています。 CPU 負荷の高い長時間実行されるタスクをこの操作でシミュレートします。 POST 操作がすぐに完了できるよう、この処理は独立したスレッドで実行されます。

- `UserProfile` コントローラーの `Get` メソッドは、HTTP GET 操作を実装しています。 このメソッドによって CPU にかかる負荷はさほど高くありません。

一番気になる点は、`Post` メソッドのリソース要件です。 この処理はバックグラウンド スレッドに置かれていますが、それなりに CPU リソースが消費されます。 これらのリソースは、別のユーザーによって同時実行されている他の操作との間で共有されます。 その要求が、ある程度の人数のユーザーから同時に送信された場合、全体のパフォーマンスに影響が及び、すべての操作の処理速度が低下してしまいます。 たとえばユーザーが、`Get` メソッドで著しい待ち時間を強いられることも考えられます。

## <a name="how-to-fix-the-problem"></a>問題の解決方法

リソースが著しく消費されるプロセスを、独立したバックエンドに移動します。 

この方法により、リソースを集中的に使用するフロントエンドのタスクはメッセージ キューに置かれます。 そのタスクがバックエンドによって取得され、非同期で処理されます。 このキューには、要求をバックエンドにバッファー処理することによって負荷を平準化する働きもあります。 キューの長さが限度を超えた場合は、バックエンドをスケールアウトする自動スケールを構成することができます。

前述のコードを修正したバージョンを次に示します。 このバージョンでは、`Post` メソッドによってメッセージが Service Bus キューに追加されます。 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

バックエンドは、Service Bus キューからメッセージを取得して処理を実行します。

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a>考慮事項

- この方法によってアプリケーションの複雑さが多少増します。 万一障害が発生しても要求が失われないよう、キューとデキューを安全に処理する必要があります。
- メッセージ キューのためのサービスが別途アプリケーションに必要となります。
- 想定されるワークロードを処理し、必要なスループット目標を満たすために、処理環境にはスケーラビリティを十分に確保してください。
- この方法によって全体的な応答性は向上すると考えられますが、バックエンドに移されたタスクは、以前よりも完了までに時間がかかるようになる可能性があります。 

## <a name="how-to-detect-the-problem"></a>問題の検出方法

ビジー状態のフロントエンドは、その症状の 1 つとして、リソースを集中的に使用するタスクが実行されていると待ち時間が長くなるということがあります。 エンド ユーザーからは、応答時間の増加やサービスのタイムアウトに起因した障害が報告されることになります。また、こうした障害が発生した場合、HTTP 500 (内部サーバー) エラーまたは HTTP 503 (サービスを利用できません) エラーが返されます。 Web サーバーのイベント ログをよく調べてください。エラーの原因や状況についてさらに詳しい情報が記録されている可能性があります。

この問題の識別に役立てるために、次の手順を実行できます。

1. 運用システムのプロセス監視を実行して、応答速度が低下したポイントを特定します。
2. これらのポイントで収集されたテレメトリ データを調べ、実行されている操作と使用されているリソースの組み合わせを確認します。 
3. これらのタイミングで実行されていた操作のボリュームや組み合わせと、応答速度の悪化との間に相関関係がないか調べます。
4. 疑わしい操作についてそれぞれロード テストを実行し、どの操作がリソースを消費して他の操作のリソースを逼迫させているかを特定します。 
5. それらの操作をソース コードで確認し、リソースが過剰に消費されている理由を特定します。

## <a name="example-diagnosis"></a>診断の例 

以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用していきます。

### <a name="identify-points-of-slowdown"></a>速度が低下したポイントを特定する

各メソッドをインストルメント化して、それぞれの要求に費やされている時間とリソースを追跡します。 アプリケーションは運用環境で監視してください。 そうすることで、個々の要求が互いにどのように競合しているのかについて全体的な視点から把握することができます。 負荷の大きい時間帯は、リソース消費の激しい、実行に時間のかかる要求が互いに影響を及ぼし合うことが考えられます。システムを監視し、パフォーマンスの低下に注目することによって、その動作を観察することができます。

次の画像は、監視ダッシュボードを示しています  (テストには [AppDynamics] を使用しました)。最初、システムには軽い負荷がかかっているだけです。 その後、ユーザーは `UserProfile` GET メソッドを要求し始めます。 パフォーマンスは比較的良好ですが、他のユーザーが `WorkInFrontEnd` POST メソッドの要求を発行した途端に状況が一変します。 その時点で、応答時間が急激に悪化します (1 つ目の矢印)。 応答時間が改善されるのは、`WorkInFrontEnd` コントローラーに対する要求のボリュームが減少した後です (2 つ目の矢印)。

![WorkInFrontEnd コントローラーが使用されているときのすべての要求の応答時間の影響を示す AppDynamics の [Business Transactions]\(ビジネス トランザクション\) ウィンドウ][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>テレメトリ データを調べて相関関係を見つける

次の画像は、同じ期間におけるリソース使用率を監視するために収集されたメトリックを示しています。 最初、システムにアクセスしているユーザーはほとんどいません。 接続ユーザーが増えると CPU 使用率が一気に上昇します (100%)。 また、CPU 使用率の上昇に伴い、最初はネットワーク I/O 率が上昇していることに注目してください。 しかし実は、いったん CPU 使用率がピークに達すると、ネットワーク I/O が下降しているのです。 なぜならば、CPU がその処理能力に達した後は、システムで処理できる要求が比較的少数に限られるからです。 ユーザーが接続を解除すると、CPU の負荷が次第に減少します。

![CPU 使用率とネットワーク使用率を示す AppDynamics のメトリック][AppDynamics-Metrics-Front-End-Requests]

この時点で、さらに詳しい調査の対象として、見かけ上まず候補に挙げられるのは `WorkInFrontEnd` コントローラーの `Post` メソッドでしょう。 この仮説を検証するためには、管理された環境で、さらに調査を進める必要があります。

### <a name="perform-load-testing"></a>ロード テストを実行する 

次のステップは、管理された環境でテストを実行することです。 たとえば、それぞれの要求について、それを含めた場合と省略した場合を比較する一連のロード テストを実行し、順次その影響を確認していきます。

以下のグラフは、前のテストで使用したクラウド サービスとまったく同じデプロイ環境に対するロード テストの結果を示しています。 このテストでは、500 ユーザーという一定の負荷で `UserProfile` コントローラーの `Get` 操作を実行すると共に、`WorkInFrontEnd` コントローラーの `Post` 操作を実行するユーザー数 (負荷) を段階的に増やしています。 

![WorkInFrontEnd コントローラーの初回ロード テストの結果][Initial-Load-Test-Results-Front-End]

最初の段階の負荷は 0 で、アクティブなユーザーが実行しているのは `UserProfile` 要求だけです。 このシステムは毎秒約 500 件の要求に応答することができます。 60 秒後、負荷として、新たに 100 ユーザーが `WorkInFrontEnd` コントローラーに POST 要求を送信し始めます。 すると、ほとんど間髪を入れずに、`UserProfile` コントローラーに送信されるワークロードは、1 秒あたり約 150 の要求まで減少します。 これはロード テスト実行プログラムの動作に起因するものです。 応答を待って次の要求が送信されるので、応答が返されるまでに時間がかかるほど、要求の速度が低下します。

`WorkInFrontEnd` コントローラーに POST 要求を送信するユーザーが増えるにつれて、`UserProfile` コントローラーの応答速度は低下し続けていきます。 しかし、`WorkInFrontEnd` コントローラーによって処理される要求のボリュームは比較的一定の状態を保っていることに注目してください。 両方の要求の全体的な速度が安定に向かいつつも、下限に近い状態にあることから、システムが飽和状態になっていることがはっきりと見て取れます。


### <a name="review-the-source-code"></a>ソース コードをレビューする

最後のステップとして、ソース コードに注目してみましょう。 開発チームは `Post` メソッドにかなりの時間がかかる可能性があることを認識していました。最初の実装段階から、スレッドを独立させていたのは、そのためです。 それによって差し迫った問題は解決されました。長時間実行されるタスクの完了を、`Post` メソッドがブロック状態で待つことはないからです。

しかしこのメソッドによって実行される処理は、依然として CPU やメモリなどのリソースを消費します。 このプロセスが非同期に実行できる場合、実際にはパフォーマンスが低下する可能性があります。ユーザーによって際限なく、大量の操作が同時にトリガーされることも考えられるからです。 サーバーが実行できるスレッドの数には制限があります。 その制限を超えて新しいスレッドをアプリケーションが起動しようとしたとき、高い確率で例外が発生します。

> [!NOTE]
> だからといって非同期操作を避けなければならない、という意味ではありません。 このような場合は、ネットワーク呼び出しで非同期の await を実行することが推奨されます  ([同期 I/O][sync-io]のアンチパターンに関するページを参照)。ここでの問題は、CPU 負荷の高い処理が別のスレッドで生成されることです。 

### <a name="implement-the-solution-and-verify-the-result"></a>ソリューションを実装して結果を検証する

次の画像は、解決策の導入後にパフォーマンスを監視した結果です。 負荷は前回と同様ですが、`UserProfile` コントローラーの応答時間が大幅に短縮されています。 同じ時間における要求のボリュームは、2,759 から 23,565 に増えています。 

![WorkInBackground コントローラーが使用されているときのすべての要求の応答時間の影響を示す AppDynamics の [Business Transactions]\(ビジネス トランザクション\) ウィンドウ][AppDynamics-Transactions-Background-Requests]

`WorkInBackground` コントローラーによって処理される要求のボリュームも大幅に増えていることに注目してください。 しかしこの場合、両者を直接比較することはできません。このコントローラーで実行される処理は、最初のコードと大きく異なるからです。 新たに書き換えたコードでは、時間がかかる計算を実行せず、要求をキューに追加しているにすぎません。 重要な点は、このメソッドがシステム全体の足を引っ張ることがなくなったということです。

CPU 使用率とネットワーク使用率にも、パフォーマンスの改善が見られます。 CPU 使用率が 100% に達することはなくなりました。また、処理されたネットワーク要求のボリュームも、先ほどより大幅に増え、ワークロードが低下するまで減少しませんでした。

![WorkInBackground コントローラーの CPU 使用率とネットワーク使用率を示す AppDynamics のメトリック][AppDynamics-Metrics-Background-Requests]

次のグラフは、ロード テストの結果を示しています。 前回のテストと比べ、処理された要求の全体的なボリュームが大幅に改善しています。

![BackgroundImageProcessing コントローラーのロード テストの結果][Load-Test-Results-Background]

## <a name="related-guidance"></a>関連するガイダンス

- [自動スケールのベスト プラクティス][autoscaling]
- [バックグラウンド ジョブのベスト プラクティス][background-jobs]
- [キュー ベースの負荷平準化パターン][load-leveling]
- [Web/キュー/ワーカー アーキテクチャ スタイル][web-queue-worker]

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


