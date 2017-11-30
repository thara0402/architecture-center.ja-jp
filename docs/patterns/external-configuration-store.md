---
title: "外部構成ストア"
description: "アプリケーション展開パッケージの構成情報を一元管理される場所に移動します。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="external-configuration-store-pattern"></a>外部構成ストア パターン

[!INCLUDE [header](../_includes/header.md)]

アプリケーション展開パッケージの構成情報を一元管理される場所に移動します。 こうすることで、構成データの管理と制御が簡単になり、構成データをアプリケーションとアプリケーション インスタンス全体で共有できるようになります。

## <a name="context-and-problem"></a>コンテキストと問題

多くのアプリケーション ランタイム環境には、アプリケーションと共にデプロイされるファイルに保持される構成情報が含まれています。 場合によっては、これらのファイルを編集して、デプロイ後にアプリケーションの動作を変更することができます。 ただし、構成を変更するには、アプリケーションを再デプロイする必要があります。また、その結果、許容できないダウンタイムや他の管理上のオーバーヘッドが発生することもよくあります。

ローカル構成ファイルでも構成は単一のアプリケーションに制限されていますが、複数のアプリケーションで構成設定を共有することが適している場合もあります。 たとえば、データベース接続文字列、UI テーマ情報、関連するアプリケーション セットに使用されるキューとストレージの URL があります。

アプリケーションの複数の実行インスタンス全体でローカル構成の変更を管理することが困難になります。クラウドホスト型のシナリオの場合は特に困難です。 インスタンスは異なる構成設定を使用することになりますが、更新プログラムはデプロイされます。

さらに、アプリケーションとコンポーネントの更新プログラムでは、構成スキーマの変更が必要になることがあります。 多くの構成システムは、異なるバージョンの構成情報をサポートしていません。

## <a name="solution"></a>解決策

外部ストレージに構成情報を格納し、構成設定の読み取りと更新をすばやく効率的に行うために使用できるインターフェイスを用意します。 外部ストアの種類は、アプリケーションのホスティングおよびランタイム環境によって変わります。 クラウドホスト型シナリオでは、一般的にクラウドベースのストレージ サービスですが、ホスト型データベースや他のシステムの場合もあります。

構成情報のために選択するバッキング ストアには、一貫した使いやすいアクセスを提供するインターフェイスがあるものをお勧めします。 正しく入力され、構造化された形式で情報を公開します。 実装では、必要に応じてユーザーのアクセスを承認して、構成データを保護するようにします。また、複数バージョンの構成 (それぞれの複数リリース バージョンを含め、開発、ステージング、運用など) のストレージを許容できるように柔軟にします。

> 多くの組み込み構成システムは、アプリケーションの起動時にデータを読み取り、データをメモリ内にキャッシュして高速なアクセスを提供し、アプリケーション参照に対する影響を最小限に抑えます。 使用するバッキング ストアの種類とそのストアの待機時間によっては、外部構成ストア内にキャッシュ メカニズムを実装することが役立つ場合があります。 詳細については、「 [Caching Guidance (キャッシュのガイダンス)](https://msdn.microsoft.com/library/dn589802.aspx)」を参照してください。 この図は、省略可能なローカル キャッシュがある外部構成ストア パターンの概要を示しています。

![省略可能なローカル キャッシュがある外部構成ストア パターンの概要。](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

許容できるパフォーマンス、高可用性、堅牢性を提供し、アプリケーション保守と管理プロセスの一部としてバックアップすることができるバッキング ストアを選択します。 クラウドホスト型アプリケーションの場合、これらの要件を満たすには、通常、クラウド ストレージ メカニズムの使用がお勧めです。

保持できる情報の種類に柔軟性を持たせるようにバッキング ストアのスキーマを設計します。 型指定されたデータ、設定のコレクション、複数バージョンの設定、使用するアプリケーションに必要なその他の機能など、すべての構成要件に備えるようにします。 要件の変更に応じて追加の設定をサポートできるように、拡張しやすいスキーマにします。

バッキング ストアの物理機能、構成情報を格納する方法と関連付ける方法、およびパフォーマンスに対する影響を検討します。 たとえば、構成情報を含む XML ドキュメントの保存には、個々の設定を読み取るために、ドキュメントを解析できる構成インターフェイスまたはアプリケーションが必要です。 設定の更新は複雑になりますが、設定のキャッシュによって、読み取りパフォーマンスの低下と相殺することができます。

構成インターフェイスで、構成設定の範囲と継承の制御を許可する方法を検討します。 たとえば、構成設定の範囲を組織、アプリケーション、およびコンピューター レベルに設定する要件が考えられます。 必要に応じて、異なる範囲に対するアクセスの制御の委任をサポートし、個々のアプリケーションが設定を上書きすることを禁止または許可します。

構成インターフェイスで、型指定された値、コレクション、キー/値のペア、プロパティ バッグなど、必要な形式で構成データを公開できるようにします。

設定にエラーが含まれる場合、または設定がバッキング ストアに存在しない場合の、構成ストア インターフェイスの動作方法を検討します。 既定の設定とログ エラーを返すことが適切な場合もあります。 また、構成設定のキーや名前の大文字と小文字の区別、バイナリ データの保存と処理、null 値または空の値の処理方法などの側面も検討します。

構成データを保護して、適切なユーザーとアプリケーションにのみアクセスを許可する方法を検討します。 これは構成ストア インターフェイスの機能が考えられますが、適切なアクセス許可なしでバッキング ストアのデータに直接アクセスできないようにすることも必要です。 構成データの読み取りと書き込みに必要なアクセス許可は、厳密に分離します。 また、構成設定の一部またはすべてを暗号化する必要があるかどうか、それを構成ストア インターフェイスにどのように実装するかについても検討します。

一元的に保存されている構成で、実行時にアプリケーションの動作を変更する構成は非常に重要です。アプリケーション コードのデプロイと同じメカニズムを使用して、デプロイ、更新、および管理することをお勧めします。 たとえば、複数のアプリケーションに影響する可能性がある変更は、その構成を使用するすべてのアプリケーションにその変更が適切できることを確認するために、完全なテストと段階的なデプロイ手法を使用して実施する必要があります。 管理者が 1 つのアプリケーションを更新するように設定を編集する場合、同じ設定を使用する他のアプリケーションに悪影響が出る可能性があります。

アプリケーションが構成情報をキャッシュしている場合、構成が変更されたときにアプリケーションは通知を受ける必要があります。 その情報が定期的に自動更新され、すべての変更が取得 (および実行) されるように、キャッシュされている構成データに対して有効期限ポリシーを実装することができます。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の場合に役立ちます。

- 複数のアプリケーションとアプリケーション インスタンス間で構成設定を共有する場合、または標準の構成を複数のアプリケーションとアプリケーション インスタンス全体に適用する必要がある場合。

- 画像や複雑なデータ型の格納など、必要なすべての構成設定をサポートしない標準の構成システム。

- 一元的に保存されている設定の一部またはすべてをアプリケーションで上書きできるようにする場合など、アプリケーションの一部の設定の補完的なストアとして。

- 複数のアプリケーションの管理を簡易化する方法として。また、必要に応じて、構成ストアに対する一部またはすべての種類のアクセスをログに記録して、構成設定の使用を監視するため。

## <a name="example"></a>例

Microsoft Azure でホストされるアプリケーションの場合、構成情報を外部に保存する一般的な選択肢は、Azure Storage を使用することです。 Azure Storage は回復力があり、パフォーマンスが高く、自動フェールオーバーで 3 回レプリケートされるので、高可用性を実現できます。 Azure Table Storage には、値に柔軟なスキーマを使用できる機能を持つキー/値のストアが用意されています。 Azure Blob ストレージには、個別の名前の BLOB に任意の種類のデータを保持できる階層型でコンテナーベースのストアが用意されています。

次の例は、構成情報の格納と公開に使用する構成ストアを BLOB ストレージに実装する方法を示しています。 次のコードのように、構成情報を保持できるように `BlobSettingsStore` クラスで BLOB ストレージを抽象化し、`ISettingsStore` インターフェイスを実装します。

> このコードは、[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) から入手できる _ExternalConfigurationStore_ ソリューションの _ExternalConfigurationStore.Cloud_ プロジェクトで提供されています。

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

このインターフェイスでは、構成ストアに保持される構成設定の取得と更新に使用されるメソッドを定義しています。また、いずれかの構成設定が最近変更されたかどうかを検出するために使用できるバージョン番号が含まれています。 `BlobSettingsStore` クラスは、バージョン管理を実装するために BLOB の `ETag` プロパティを使用しています。 `ETag` プロパティは、BLOB が書き込まれるたびに自動的に更新されます。

> 設計により、この単純なソリューションは、すべての構成設定を型指定された値ではなく文字列値として公開します。

`ExternalConfigurationManager` クラスは、`BlobSettingsStore` オブジェクトのラッパーを提供します。 アプリケーションはこのクラスを使用して、構成情報を格納および取得できます。 このクラスは、Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) ライブラリを使用し、`IObservable` インターフェイスを実装して、構成に変更が加えられた場合はその変更を公開します。 `SetAppSetting` メソッドを呼び出して設定が変更された場合は、`Changed` イベントが発生し、このイベントのサブスクライバーすべてに通知されます。

すべての設定は、すばやくアクセスできるように、`ExternalConfigurationManager` クラス内の `Dictionary` オブジェクトにもキャッシュされます。 構成設定の取得に使用された `GetSetting` メソッドで、キャッシュからデータを読み取ります。 キャッシュに設定が見つからなかった場合は、代わりに `BlobSettingsStore` オブジェクトから取得されます。

`GetSettings` メソッドは `CheckForConfigurationChanges` メソッドを呼び出して、BLOB ストレージの構成情報が変更されたかどうかを検出します。 この処理のために、バージョン番号が確認され、`ExternalConfigurationManager` オブジェクトに保持されている最新バージョン番号が比較されます。 1 つ以上の変更が発生した場合、`Changed` イベントが発生し、`Dictionary` オブジェクトにキャッシュされている構成設定は更新されます。 これは、[キャッシュアサイド パターン](cache-aside.md)のアプリケーションです。

次のコード サンプルは、`Changed` イベント、`GetSettings` メソッド、および `CheckForConfigurationChanges` メソッドの実装方法を示しています。

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> `ExternalConfigurationManager` クラスにも、`Environment` というプロパティがあります。 このプロパティは、ステージングや運用など、さまざまな環境で実行されるアプリケーションの多様な構成をサポートします。

`ExternalConfigurationManager` オブジェクトは `BlobSettingsStore` オブジェクトに対して、すべての変更を照会するクエリを定期的に実行することもできます。 次のコードでは、`StartMonitor` メソッドは、すべての変更を検出できる間隔で `CheckForConfigurationChanges` を呼び出し、前述のように `Changed` イベントを発生させます。

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

`ExternalConfigurationManager` クラスは、以下のように `ExternalConfiguration` クラスによるシングルトン インスタンスとしてインスタンス化されます。

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

次のコードは、_ExternalConfigurationStore.Cloud_ プロジェクトの `WorkerRole` クラスから引用されたものです。 アプリケーションが設定を読み取るために `ExternalConfiguration` クラスを使用する方法を示しています。

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

次のコードも `WorkerRole` クラスから引用されたものです。アプリケーションが構成イベントにサブスクライブする方法を示しています。

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

- このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) から入手できます。
