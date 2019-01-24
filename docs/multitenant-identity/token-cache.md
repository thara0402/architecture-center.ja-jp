---
title: マルチテナント アプリケーションでアクセス トークンをキャッシュする
description: バックエンド Web API を呼び出すために使用されるアクセス トークンのキャッシュ。
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: ccecee8856d3c053d320ebd2bb26469389ce9a10
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481696"
---
# <a name="cache-access-tokens"></a>アクセス トークンのキャッシュ

[![GitHub](../_images/github.png) サンプル コード][sample application]

トークン エンドポイントに対する HTTP 要求が必要なので、OAuth アクセス トークンの取得は比較的コストが高い処理です。 そのため、可能な限りトークンをキャッシュすることをお勧めします。 [Azure AD Authentication Library][ADAL] (ADAL) が Azure AD から取得したトークンを自動的にキャッシュします。トークンには更新トークンも含まれます。

ADAL には、既定のトークン キャッシュ機能が実装されています。 ただし、このトークン キャッシュは、ネイティブ クライアント アプリ用なので、Web アプリには **適していません** 。

* また、静的インスタンスなので、スレッド セーフではありません。
* すべてのユーザーのトークンは同じディレクトリに保存されるので、多数のユーザーに合わせて拡張されません。
* フォーム内の Web サーバー全体で共有することはできません。

代わりに、ADAL `TokenCache` クラスから派生したカスタムのトークン キャッシュを実装する必要がありますが、サーバー環境に適しており、異なるユーザーのトークン間を適度なレベルで分離できます。

`TokenCache` クラスは、発行者、リソース、クライアント ID、およびユーザーでインデックス付けされたトークンのディクショナリを格納します。 カスタム トークンのキャッシュは、Redis キャッシュなどのバッキング ストアに、このディクショナリを書き込む必要があります。

Tailspin Surveys アプリケーションでは、 `DistributedTokenCache` クラスがトークン キャッシュを実装します。 この実装には、ASP.NET Core の [IDistributedCache][distributed-cache] 抽象化を使用します。 このように、 `IDistributedCache` 実装はバッキング ストアとして使用できます。

* Surveys アプリの既定では、Redis Cache を使用します。
* 単一のインスタンスの Web サーバーの場合は、ASP.NET Core の[メモリ内キャッシュ][in-memory-cache]を使用できます。 (アプリ開発中にローカルで実行するのにも適しています。)

`DistributedTokenCache` は、キー/値ペアとしてキャッシュ データをバッキング ストアに保存します。 キーはユーザー ID + クライアント ID なので、ユーザー/クライアントの一意の組み合わせごとに異なるキャッシュ データがバッキング ストアに保存されます。

![トークンのキャッシュ](./images/token-cache.png)

バッキング ストアは、ユーザーごとにパーティション分割されています。 各 HTTP 要求に対して、そのユーザーのトークンはバッキング ストアから読み取られ、`TokenCache` ディクショナリに読み込まれます。 Redis をバッキング ストアとして使用する場合は、サーバー ファーム内のすべてのサーバー インスタンスが同じキャッシュに読み取りと書き込みを行い、このアプローチは多数のユーザーに適用されます。

## <a name="encrypting-cached-tokens"></a>キャッシュされたトークンを暗号化する

トークンによりユーザーのリソースへのアクセス権が付与されるため、トークンは機密性の高いデータです。 (さらに、ユーザー パスワードと異なり、トークンのハッシュだけを格納することはできません。)そのため、侵害される前にトークンを保護することが非常に重要です。 Redis でサポートされるキャッシュはパスワードで保護されますが、他のユーザーがパスワードを取得した場合、そのユーザーはすべてのキャッシュされたアクセス トークンを取得できます。 このような理由から、`DistributedTokenCache` は、バッキング ストアへのすべての書き込みを暗号化します。 暗号化は、ASP.NET Core の[データ保護][data-protection] API を使用して行われます。

> [!NOTE]
> Azure Web サイトにデプロイする場合、暗号化キーはネットワーク ストレージにバックアップされ、すべてのマシン間で同期されます (詳細は、「[キーの管理と有効期間][key-management]」をご覧ください)。 既定では、キーは Azure Web サイト内で実行されるときには暗号化されませんが、[X.509 証明書を使用して暗号化を有効にする][x509-cert-encryption]ことができます。

## <a name="distributedtokencache-implementation"></a>DistributedTokenCache の実装

`DistributedTokenCache` クラスは、ADAL [TokenCache][tokencache-class] クラスから派生しています。

コンストラクターの `DistributedTokenCache` クラスによって、現在のユーザーのキーが作成され、バッキング ストアからキャッシュが読み込まれます。

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

キーは、ユーザー ID とクライアント ID を連結して作成されます。 この両方の ID は、ユーザーの `ClaimsPrincipal`で見つかった要求から取得されます。

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

キャッシュ データを読み込むには、バッキング ストアからシリアル化された BLOB を読み取り、 `TokenCache.Deserialize` を呼び出して BLOB をキャッシュ データに変換します。

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

ADAL がキャッシュにアクセスするたびに、 `AfterAccess` イベントが発生します。 キャッシュ データが変更されると、 `HasStateChanged` プロパティは true になります。 その場合、バッキング ストアを更新して変更を反映し、 `HasStateChanged` を false に設定します。

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

TokenCache からは、他にも 2 つのイベントが送信されます。

* `BeforeWrite` ADAL がキャッシュに書き込む直前の呼び出し。 これを使用して、コンカレンシー戦略を実装できます。
* `BeforeAccess` ADAL がキャッシュから読み取る直前の呼び出し。 ここで、最新バージョンを取得するキャッシュを再読み込みできます。

この例では、これら 2 つのイベントを処理していません。

* コンカレンシーの場合、最後の書き込みが有効になりますが、問題ありません。 ユーザー + クライアントごとにトークンは独立して保存されるので、同一ユーザーが同時にログイン セッションを実行した場合にのみ、競合が発生します。
* 読み取りについては、要求ごとにキャッシュを読み込みます。 要求の存続期間は短期です。 存続期間内にキャッシュが変更された場合、次の要求は新しい値を選択します。

[**次へ**][client-assertion]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
