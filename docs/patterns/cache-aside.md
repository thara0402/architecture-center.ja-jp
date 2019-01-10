---
title: キャッシュ アサイド パターン
titleSuffix: Cloud Design Patterns
description: オンデマンドでデータをデータ ストアからキャッシュに読み込みます。
keywords: 設計パターン
author: dragon119
ms.date: 11/01/2018
ms.custom: seodec18
ms.openlocfilehash: 96dee3ca766414a3a17ea161f13c9fcd15001b4d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114166"
---
# <a name="cache-aside-pattern"></a>キャッシュ アサイド パターン

[!INCLUDE [header](../_includes/header.md)]

オンデマンドでデータをデータ ストアからキャッシュに読み込みます。 これにより、パフォーマンスが向上するだけでなく、キャッシュに保持されているデータと基になるデータ ストアのデータの間で整合性が維持されます。

## <a name="context-and-problem"></a>コンテキストと問題

アプリケーションは、キャッシュを使用して、データ ストアに保持されている情報への繰り返されるアクセスを向上させています。 ただし、キャッシュ データがデータ ストアのデータと常に完全に一致することを期待するのは現実的ではありません。 アプリケーションでは、キャッシュのデータをできるだけ最新の状態に維持できるようにするための戦略を実装する必要がありますが、キャッシュのデータが古くなったときに発生する状況を検出して対処することもできます。

## <a name="solution"></a>解決策

多くの市販のキャッシュ システムで、リード スルーおよびライト スルー/ライト ビハインド操作が提供されています。 これらのシステムでは、アプリケーションはキャッシュを参照してデータを取得します。 データがキャッシュにない場合は、データ ストアから取得され、キャッシュに追加されます。 キャッシュに保持されているデータに対する変更は、データ ストアにも自動的にライトバックされます。

キャッシュがこの機能を備えていない場合、キャッシュを使用してデータを維持するアプリケーションがその役割を担います。

アプリケーションは、キャッシュ アサイド戦略を実装することで、リード スルー キャッシュの機能をエミュレートできます。 この戦略では、オンデマンドでデータをキャッシュに読み込みます。 次の図は、キャッシュ アサイド パターンを使用してデータをキャッシュに格納する方法を示しています。

![キャッシュ アサイド パターンを使用したキャッシュへのデータの格納](./_images/cache-aside-diagram.png)

アプリケーションは、情報を更新したら、データ ストアに変更を加え、キャッシュ内の対応する項目を無効にすることによって、ライト スルー戦略に従うことができます。

その項目が次回必要になったときは、キャッシュ アサイド戦略を使用することで、データ ストアから最新のデータが取得され、キャッシュに再度追加されます。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

**キャッシュ データの有効期間**:  多くのキャッシュで、指定された期間アクセスされなかったデータを、無効にしてキャッシュから削除する有効期限ポリシーが実装されています。 キャッシュ アサイドを効果的に使用するには、有効期限ポリシーが、データを使用するアプリケーションのアクセス パターンに適合していることを確認します。 有効期限が短すぎると、アプリケーションが常にデータ ストアからデータを取得してキャッシュに追加する可能性があるので、有効期限をあまり短くしないでください。 同様に、キャッシュ データが古くなる可能性が高いので、有効期限をあまり長くしないでください。 キャッシュは、比較的静的なデータまたは頻繁に読み取られるデータに最も効果的であることに注意してください。

**データの削除**:  ほとんどのキャッシュは、データのソースであるデータ ストアに比べてサイズが限られており、必要に応じてデータが削除されます。 ほとんどのキャッシュでは、削除する項目を選択するために最低使用頻度ポリシーを採用していますが、これはカスタマイズ可能です。 キャッシュのコスト効率を確保するために、キャッシュのグローバルな有効期限プロパティと他のプロパティ、および各キャッシュ項目の有効期限プロパティを構成します。 キャッシュ内のすべての項目にグローバルな削除ポリシーを適用することが必ずしも適切であるとは限りません。 たとえば、データ ストアからの取得に非常にコストがかかるキャッシュ項目がある場合、アクセス頻度は高くても低コストの項目と引き換えに、その項目をキャッシュに保持する方が有益な場合があります。

**キャッシュの準備**:  多くのソリューションでは、アプリケーションが起動処理の一環として必要とする可能性の高いデータをキャッシュに事前に配置します。 キャッシュ アサイド パターンは、このようなデータの一部が期限切れになったり、削除されたりした場合にも役立ちます。

**整合性**:  キャッシュ アサイド パターンを実装しても、データ ストアとキャッシュの間での整合性が保証されるわけではありません。 データ ストア内の項目は、外部プロセスによって常に変更される可能性があります。この変更は、その項目が次回読み込まれるまでキャッシュに反映されないことがあります。 データ ストア間でデータをレプリケートするシステムでは、同期が頻繁に発生すると、この問題が深刻化する可能性があります。

**ローカル (メモリ内) キャッシュ**:  キャッシュがアプリケーション インスタンスに対してローカルであり、メモリ内に格納されている場合があります。 アプリケーションが同じデータに繰り返しアクセスする場合は、この環境でキャッシュ アサイドが役立ちます。 ただし、ローカル キャッシュはプライベートであるため、さまざまなアプリケーション インスタンスがそれぞれ同じキャッシュ データのコピーを持つことができます。 このデータはキャッシュ間での整合性がすぐに失われる可能性があるため、プライベート キャッシュに保持されているデータを期限切れにし、より頻繁に更新する必要があります。 これらのシナリオでは、共有または分散キャッシュ メカニズムの使用方法を調べることを検討してください。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の状況で使用します。

- キャッシュが、ネイティブのリード スルーおよびライト スルー操作を提供していない場合。
- リソースの需要は予測できません。 このパターンを使用すると、アプリケーションはオンデマンドでデータを読み込むことができます。 アプリケーションに必要なデータは事前に想定されません。

このパターンが適していないと考えられる状況は次のとおりです。

- キャッシュされたデータ セットが静的な場合。 データが使用可能なキャッシュ領域に収まる場合は、起動時にデータをキャッシュに配置し、データが期限切れになるのを防ぐポリシーを適用します。
- Web ファームでホストされている Web アプリケーションでセッション状態情報をキャッシュする場合。 この環境では、クライアント/サーバー アフィニティに基づく依存関係の導入を避ける必要があります。

## <a name="example"></a>例

Microsoft Azure では、Azure Redis Cache を使用して、アプリケーションの複数のインスタンスで共有できる分散キャッシュを作成できます。

次のコード例では、.NET 向けに作成された Redis クライアント ライブラリである、[StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) クライアントを使用しています。 Azure Redis Cache インスタンスに接続するには、静的 `ConnectionMultiplexer.Connect` メソッドを呼び出し、接続文字列を渡します。 このメソッドは、接続を表す `ConnectionMultiplexer` を返します。 アプリケーション内の `ConnectionMultiplexer` インスタンスを共有する方法の 1 つに、次の例のように、接続されたインスタンスを返す静的プロパティを設定する方法があります。 この方法では、接続された 1 つのインスタンスだけがスレッドセーフな方法で初期化されます。

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

次のコード例の `GetMyEntityAsync` メソッドは、キャッシュ アサイド パターンの実装を示しています。 このメソッドは、リード スルー手法を使用してキャッシュからオブジェクトを取得します。

オブジェクトは、整数 ID をキーとして使用して識別されます。 `GetMyEntityAsync` メソッドは、このキーを持つ項目をキャッシュから取得することを試みます。 一致する項目が見つかった場合は、それが返されます。 キャッシュに一致するものがない場合、`GetMyEntityAsync` メソッドはデータ ストアからオブジェクトを取得し、キャッシュに追加してから返します。 データ ストアから実際にデータを読み取るコードはデータ ストアによって異なるため、ここでは示していません。 キャッシュ項目は、他の場所で更新された場合に古くならないように有効期限が構成されています。

```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();

  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json)
                ? default(MyEntity)
                : JsonConvert.DeserializeObject<MyEntity>(json);

  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it is data store dependent.
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

> 各例では、Redis Cache を使用してストアにアクセスし、キャッシュから情報を取得します。 詳細については、[Microsoft Azure Redis Cache の使用方法](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache)に関する記事、および「[Redis Cache で Web アプリを作成する方法](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)」をご覧ください。

次に示す `UpdateEntityAsync` メソッドは、アプリケーションによって値が変更されたときにキャッシュ内のオブジェクトを無効にする方法を示しています。 このコードでは、元のデータ ストアを更新した後、キャッシュ項目をキャッシュから削除します。

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false);

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> ステップの順序が重要です。 項目をキャッシュから削除する "*前に*" データ ストアを更新します。 キャッシュ項目を先に削除すると、データ ストアが更新されるまでの短い時間に、クライアントがその項目を取得する可能性があります。 これにより、キャッシュ ミスが発生し (キャッシュから項目が削除されたため)、以前のバージョンの項目がデータ ストアから取得され、キャッシュに再度追加されることになります。 その結果、キャッシュ データが古くなります。

## <a name="related-guidance"></a>関連するガイダンス

このパターンを実装するときは、次の情報を参考にしてください。

- [キャッシュ ガイダンス](https://docs.microsoft.com/azure/architecture/best-practices/caching)。 クラウド ソリューションでデータをキャッシュする方法と、キャッシュを実装する際に考慮すべき問題に関する追加情報を提供します。

- [Data consistency primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)。 通常、クラウド アプリケーションは、複数のデータ ストアに分散したデータを使用します。 この環境でのデータ整合性の管理と維持は、システム (特に、発生する可能性のあるコンカレンシーと可用性の問題) の重要な側面です。 この入門書では、分散データの整合性に関する問題について説明し、アプリケーションがデータの可用性を維持するために最終的な整合性を実装する方法の概要を説明します。
