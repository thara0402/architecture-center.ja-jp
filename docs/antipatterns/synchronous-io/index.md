---
title: 同期 I/O のアンチパターン
description: I/O が完了するまで呼び出し元スレッドをブロックすることにより、パフォーマンスが低下して、垂直拡張性に影響を及ぼすことがあります。
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538555"
---
# <a name="synchronous-io-antipattern"></a>同期 I/O のアンチパターン

I/O が完了するまで呼び出し元スレッドをブロックすることにより、パフォーマンスが低下して、垂直拡張性に影響を及ぼすことがあります。

## <a name="problem-description"></a>問題の説明

同期 I/O 操作は、I/O が完了するまで呼び出し元のスレッドをブロックします。 呼び出し元のスレッドは待機状態になり、その間は有益な処理を実行することができず処理リソースを浪費します。

I/O の一般的な例は次のとおりです。

- データベースまたは任意の種類の永続的ストレージにデータを取得 (永続化) します。
- Web サービスに要求を送信します。
- メッセージを投稿するか、キューからメッセージを取得します。
- ローカル ファイルに対して書き込みまたは読み取りを行います。

このアンチパターンは、通常、次の理由で発生します。

- アンチパターンが、操作を実行するための最も簡単な方法であると感じられるため。 
- アプリケーションが要求からの応答を必要とするため。
- アプリケーションが使用するライブラリが I/O に対して同期メソッドしか提供しないため。 
- 外部ライブラリが同期 I/O を内部で実行するため。 1 回の同期 I/O 呼び出しによって呼び出しチェーン全体がブロックされます。

次のコードによって、ファイルが Azure BLOB ストレージにアップロードされます。 このコードで同期 I/O の待機をブロックする箇所が 2 つあります。`CreateIfNotExists` メソッドと `UploadFromStream` メソッドです。

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

外部サービスからの応答を待機する例を次に示します。 `GetUserProfile` メソッドは、`UserProfile` を返すリモート サービスを呼び出します。

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

この 2 つの例両方の完全なコードについては、[こちら][sample-app]を参照してください。

## <a name="how-to-fix-the-problem"></a>問題の解決方法

同期 I/O 操作を非同期操作に置き換えます。 これにより、現在のスレッドがブロックされずに有意義な作業の実行を続けて、コンピューティング リソースの使用率の向上につながります。 I/O の非同期実行は、クライアント アプリケーションからの要求の予期しない増加に対応する際に特に有効です。 

多くのライブラリでは、メソッドの同期バージョンと非同期バージョンの両方が提供されます。 可能な限り、非同期バージョンを使用してください。 次に示すのは、ファイルを Azure BLOB ストレージにアップロードする前の例の非同期バージョンです。

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

`await` 演算子は、非同期操作が実行されている間、制御を呼び出し元環境に返します。 非同期操作が完了してから、このステートメントの後のコードが継続して実行されます。

適切に設計されたサービスは、非同期操作も提供する必要があります。 ユーザー プロファイルを返す Web サービスの非同期バージョンを次に示します。 `GetUserProfileAsync` メソッドは、ユーザー プロファイル サービスの非同期バージョンの使用に依存しています。

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

操作の非同期バージョンが提供されないライブラリの場合は、選択した同期メソッドに対して非同期ラッパーを作成することができます。 このアプローチに従う際には注意が必要です。 非同期ラッパーを呼び出したスレッドの応答性を向上させることができる一方で、実際には消費リソース量が増加します。 スレッドが追加作成されることがあり、このスレッドによって行われる作業の同期に関連するオーバーヘッドが生じます。 一部のトレードオフについては、このブログ記事「[Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]」(同期メソッドの非同期ラッパーを公開する必要があるか) で説明されています。

同期メソッドの非同期ラッパーの例を次に示します。

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

これで、呼び出し元コードは、ラッパーを待機できます。

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>考慮事項

- 実行時間がきわめて短く、競合を引き起こす可能性が少ない I/O 操作は、同期操作とした方がパフォーマンスが向上する可能性があります。 たとえば、SSD ドライブでの小容量ファイルの読み取りです。 タスクの別のスレッドへのディスパッチや、タスク完了時のそのスレッドとの同期のオーバーヘッドが、非同期 I/O のメリットを上回る場合があります。 ただし、このようなケースは比較的まれで、ほとんどの I/O 操作は非同期で行う必要があります。

- I/O パフォーマンスを向上させると、システムの他の部分のボトルネックを引き起こすことがあります。 たとえば、スレッドをブロック解除すると、共有リソースへの同時要求が増加して、リソースの枯渇すなわちリソースの調整につながることがあります。 この問題が生じた場合は、Web サーバーまたはパーティション データ ストアの数を拡張して、競合を減らす必要があります。

## <a name="how-to-detect-the-problem"></a>問題の検出方法

ユーザーにとっては、定期的にアプリケーションが応答していないように見えたり、ハングアップしていると感じたりすることがあります。 アプリケーションは、タイムアウト例外で失敗する可能性があります。 このようなエラーでは、HTTP 500 (Internal Server) のエラーが返されることもあります。 サーバーでは、スレッドが使用可能になるまで、入っているクライアント要求がブロックされるため、要求キューの長さが超過して HTTP 503 (Service Unavailable) というエラーが発生します。

問題の識別に役立てるために、次の手順を実行できます。

1. 実稼働システムを監視し、ブロックされたワーカー スレッドがスループットを抑制しているかどうかを判別します。

2. スレッド不足のために要求がブロックされている場合は、アプリケーションを確認して、操作の I/O が同期で実行しているかどうかを判別します。

3. 同期 I/O を実行している各操作について制御したロード テストを行って、それらの操作がシステム パフォーマンスに影響しているかどうかを明らかにします。

## <a name="example-diagnosis"></a>診断の例

以降のセクションでは、これらの手順を前述のサンプル アプリケーションに適用します。

### <a name="monitor-web-server-performance"></a>Web サーバーのパフォーマンスの監視

Azure の Web アプリケーションおよび Web ロールについて、IIS Web サーバーのパフォーマンスを監視することをお薦めします。 具体的には、要求キューの長さに注意し、アクティビティ量が多い期間に、要求が使用可能なスレッドを待機した状態でブロックされているかどうかを証明します。 この情報は、Azure 診断を有効にして収集できます。 詳細については、次を参照してください。

- [Azure App Service のアプリの監視][web-sites-monitor]
- [Azure アプリケーションでのパフォーマンス カウンターの作成と使用][performance-counters]

アプリケーションをインストルメント化して、要求が受け入れられた後にどのように処理されるかを確認します。 要求のフローをトレースすると、要求が実行速度の遅い呼び出しを実行しており、現在のスレッドをブロックしているかどうかを特定するのに役立ちます。 スレッドのプロファイリングによって、ブロックされている要求を明らかにすることもできます。

### <a name="load-test-the-application"></a>アプリケーションのロード テスト

次のグラフは、前述の同期 `GetUserProfile` メソッドのパフォーマンスです。負荷は同時実行ユーザー数 4000 以内で変化しています。 このアプリケーションは、Azure Cloud Service Web ロールで実行している ASP.NET アプリケーションです。

![同期 I/O 操作を実行するサンプル アプリケーションのパフォーマンスのグラフ][sync-performance]

同期操作は、同期 I/O をシミュレートするため 2 秒間スリープ状態になるようにハードコーディングされます。したがって、最小応答時間は 2 秒をわずかに超えます。 負荷が 2500 同時実行ユーザー数に近づくと平均応答時間は一定になりますが、1 秒あたりの要求量は増加し続けます。 これら 2 つのメジャーのスケールが対数であることに注意してください。 1 秒あたりの要求数は、この時点とテストの終了時では倍増しています。

これだけでは、このテストから同期 I/O が問題であることは必ずしも明確にはなりません。 さらに高い負荷では、アプリケーションは、Web サーバーが要求を適切なタイミングで処理できなくなる転換点に到達し、クライアント アプリケーションがタイムアウト例外を受け取ることになります。

入ってくる要求は IIS Web サーバーによってキューに入れられ、ASP.NET スレッド プールで実行しているスレッドに渡されます。 各操作は I/O を同期実行するため、操作が完了するまでスレッドはブロックされます。 ワークロードが増加するにつれ、最終的には、スレッド プール内の ASP.NET スレッドすべてが割り当てられてブロックされます。 この時点で、これから入ってくる要求は、キュー内で使用可能なスレッドを待機する必要があります。 キューが長くなると、要求のタイムアウトが開始します。

### <a name="implement-the-solution-and-verify-the-result"></a>ソリューションの実装と結果の検証

次のグラフは、非同期バージョンのコードに対してロード テストを行った結果です。

![非同期 I/O 操作を実行するサンプル アプリケーションのパフォーマンスのグラフ][async-performance]

スループットははるかに高くなります。 前のテストと同じ期間で、システムはほぼ 10 倍のスループット (1 秒あたりの要求数) を処理することに成功しています。 さらに、平均応答時間は相対的に一定であり、前のテストに比べて約 25 分の 1 を保っています。


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



