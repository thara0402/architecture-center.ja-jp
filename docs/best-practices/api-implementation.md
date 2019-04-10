---
title: API 実装ガイダンス
titleSuffix: Best practices for cloud applications
description: API の実装方法に関するガイダンス。
author: dragon119
ms.date: 07/13/2016
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7a484aa9e4fde8fd5056608ca5dd98aefbc077b7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244233"
---
# <a name="api-implementation"></a>API 実装

入念に設計された REST ベースの Web API では、リソース、関係、およびクライアント アプリケーションにアクセスできるナビゲーション スキームを定義します。 Web API を実装し、デプロイするときは、データの論理構造ではなく、Web API をホストする環境の物理的な要件と、Web API を構成する方法を検討する必要があります。 このガイダンスでは、Web API を実装し、Web API を公開してクライアント アプリケーションで使用できるようにするためのベスト プラクティスに重点を置いて説明します。 Web API の設計の詳細については、[API 設計](/azure/architecture/best-practices/api-design)をご覧ください。

## <a name="processing-requests"></a>要求の処理

要求を処理するコードを実装する場合は、以下の点を考慮してください。

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a>GET、PUT、DELETE、HEAD、PATCH の各操作は、べき等にする

これらの要求を実装するコードが副次的な影響を及ぼさないようにしてください。 同じリソースに対して繰り返される同じ要求は、同じ状態になる必要があります。 たとえば、応答メッセージの HTTP ステータス コードが異なる場合でも、同じ URI に複数の DELETE 要求を送信したときの結果は同じである必要があります。 最初の DELETE 要求はステータス コード 204 (No Content) を返す場合があり、その次の DELETE 要求はステータス コード 404 (Not Found) を返す可能性があります。

> [!NOTE]
> べき等の概要とデータ管理操作との関連性については、Jonathan Oliver のブログ記事「 [Idempotency Patterns (べき等のパターン)](https://blog.jonathanoliver.com/idempotency-patterns/) 」をご覧ください。

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a>新しいリソースを作成する POST 操作が、関連のない副次的な影響を与えない

POST 要求が新しいリソースを作成することを目的としている場合、要求の影響をその新しいリソース (および関連する何らかの結合が存在する場合は直接関連するリソース) に限定する必要があります。たとえば、電子商取引システムにおいて、顧客の新しい注文を作成する POST 要求で、在庫レベルの修正と課金情報の生成も行うことがありますが、その注文に直接関連しない情報を変更したり、システムの全体的な状態に他の副次的な影響を及ぼしたりしないようにする必要があります。

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a>煩雑な POST、PUT、DELETE 操作を実装しない

リソース コレクションに対する POST、PUT、DELETE の各要求をサポートします。 POST 要求には複数の新しいリソースの詳細を含めることができ、それらをすべて同じコレクションに追加できます。PUT 要求では、コレクション内のリソース全体を置き換えることができ、DELETE 要求ではコレクション全体を削除できます。

ASP.NET Web API 2 に含まれる OData サポートにより、要求をバッチ処理できます。 クライアント アプリケーションは、複数の Web API 要求をパッケージ化して、それらを 1 つの HTTP 要求でサーバーに送信し、各要求への応答が含まれた 1 つの HTTP 応答を受信できます。 詳細については、「[Introducing Batch Support in Web API and Web API OData (Web API と Web API OData でのバッチ処理のサポートの概要)](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/)」をご覧ください。

### <a name="follow-the-http-specification-when-sending-a-response"></a>応答を送信する場合は、HTTP の仕様に準拠します。

Web API は、正しい HTTP 状態コード (クライアントが結果を処理する方法を判断できるようにするため)、適切な HTTP ヘッダー (クライアントが結果の特性を理解できるようにするため)、適切に書式設定された本文 (クライアントが結果を解析できるようにするため) を含むメッセージを返す必要があります。

たとえば、POST 操作は状態コード 201 (Created) を返す必要があり、応答メッセージの Location ヘッダーに新しく作成されたリソースの URI が含まれている必要があります。

### <a name="support-content-negotiation"></a>コンテンツ ネゴシエーションをサポートする

応答メッセージの本文には、さまざまな形式のデータを含めることができます。 たとえば、HTTP GET 要求では、データを JSON 形式または XML 形式で返すことができます。 クライアントは、要求の送信時に、クライアントが処理できるデータ形式を指定した Accept ヘッダーを含めることができます。 これらの形式は、メディアの種類として指定されます。 たとえば、画像を取得する GET 要求を発行するクライアントは、`image/jpeg, image/gif, image/png` のように、クライアントが処理できるメディアの種類を列挙した Accept ヘッダーを指定できます。 Web API は、結果を返すときに、これらのメディアの種類のいずれかを使用してデータを書式設定し、応答の Content-Type ヘッダーにその形式を指定する必要があります。

クライアントが Accept ヘッダーを指定しない場合は、応答の本文に実用的な既定の形式を使用します。 たとえば、ASP.NET Web API フレームワークでは、テキスト ベースのデータに既定で JSON を使用します。

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a>HATEOAS スタイルのナビゲーションとリソースの検出をサポートするリンクを提供する

HATEOAS の手法に従うことによって、クライアントは、はじめの開始ポイントからナビゲートし、リソースを特定できます。 これは、URI を含むリンクを使用することで実現されます。クライアントがリソースを取得する HTTP GET 要求を発行したときは、クライアント アプリケーションが直接関連するリソースをすばやく見つけることができるようにするための URI が応答に含まれている必要があります。 たとえば、電子商取引ソリューションをサポートする Web API では、顧客が多数の注文を行っている場合があります。 クライアント アプリケーションが顧客の詳細を取得するときには、クライアント アプリケーションがこれらの注文を取得する HTTP GET 要求を送信できるようにするためのリンクが応答に含まれている必要があります。 さらに、HATEOAS スタイルのリンクでは、リンクされた各リソースがサポートするその他の操作 (POST、PUT、DELETE など) と、各要求を実行するための対応する URI を示す必要があります。 この方法の詳細については、[API 設計][api-design]をご覧ください。

現在、HATEOAS の実装を管理する標準はありませんが、次の例は考えられる 1 つの方法を示しています。 この例では、顧客の詳細を検索する HTTP GET 要求は、その顧客の注文を参照する HATEOAS リンクを含む応答を返します。

```HTTP
GET https://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"https://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"https://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"https://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"https://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"https://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

この例では、顧客データは、次のコード スニペットに示す `Customer` クラスで表されます。 HATEOAS リンクは、 `Links` コレクション プロパティに保持されます。

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

HTTP GET 操作では、ストレージから顧客データを取得して `Customer` オブジェクトを作成し、`Links` コレクションを設定します。 結果は、JSON 応答メッセージとして書式設定されます。 各リンクは次のフィールドで構成されます。

- 返されるオブジェクトとリンクで示されているオブジェクトの関係。 この例の `self` は、リンクがそのオブジェクト自体の参照であることを示しています (多くのオブジェクト指向言語での `this` ポインターと同様)。また、`orders` は関連する注文情報が含まれたコレクションの名前です。
- リンクで示されているオブジェクトの URI 形式のハイパーリンク (`Href`)。
- この URI に送信できる HTTP 要求の種類 (`Action`)。
- 要求の種類に応じて、HTTP 要求で提供するか、または応答で返すことができるデータの形式 (`Types`)。

HTTP 応答の例に示す HATEOAS リンクは、クライアント アプリケーションが次の操作を実行できることを示しています。

- 顧客の詳細を (再度) 取得するための URI `https://adventure-works.com/customers/2` への HTTP GET 要求。 データは、XML または JSON として返すことができます。
- 顧客の詳細を変更するための URI `https://adventure-works.com/customers/2` への HTTP PUT 要求。 新しいデータは、要求メッセージで x-www-form-urlencoded 形式で提供する必要があります。
- 顧客を削除するための URI `https://adventure-works.com/customers/2` への HTTP DELETE 要求。 この要求は、追加情報を求めることも、応答メッセージの本文でデータを返すこともありません。
- 顧客のすべての注文を検索するために URI `https://adventure-works.com/customers/2/orders` に HTTP GET 要求を送信します。 データは、XML または JSON として返すことができます。
- この顧客の新規の注文を作成するために URI `https://adventure-works.com/customers/2/orders` に HTTP PUT 要求を送信します。 データは、要求メッセージで x-www-form-urlencoded 形式で提供する必要があります。

## <a name="handling-exceptions"></a>例外の処理

操作によりキャッチされない例外がスローされた場合は、次の点を検討してください。

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a>例外をキャプチャし、クライアントに意味のある応答を返す

HTTP 操作を実装するコードでは、キャッチされない例外をフレームワークに伝達するのではなく、包括的な例外処理を提供する必要があります。 例外によって操作を正常に完了することができない場合、その例外を応答メッセージで返すことができますが、例外の原因となったエラーのわかりやすい説明が含まれている必要があります。 また、あらゆる状況で状態コード 500 を単に返すのではなく、例外に適切な HTTP 状態コードが含まれている必要があります。 たとえば、ユーザー要求によって制約に違反するデータベースの更新 (未処理の注文がある顧客の削除など) が行われる場合は、状態コード 409 (Conflict) と、競合の原因を示すメッセージ本文を返します。 他の条件によって要求を達成できない状態になった場合は、状態コード 400 (Bad Request) を返すことができます。 HTTP 状態コードの全一覧については、W3C の Web サイトの「 [Status Code Definitions (状態コードの定義)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) 」をご覧ください。

次のコードは、さまざまな条件をトラップし、適切な応答を返す例を示しています。

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> API に侵入しようとする攻撃者に悪用される危険性のある情報を含めないようにしてください。
  
多くの Web サーバーは、Web API に到達する前にエラー条件自体をトラップします。 たとえば、Web サイトの認証を構成した場合、ユーザーが適切な認証情報を提供できないと、Web サーバーは状態コード 401 (Unauthorized) で応答します。 クライアントが認証されたら、クライアントが要求されたリソースにアクセスできることを確認するために、コードで独自のチェックを実行できます。 この認証に失敗した場合は、状態コード 403 (Forbidden) を返す必要があります。

### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a>一貫した方法で例外を処理し、エラーに関する情報をログに記録する

一貫した方法で例外を処理するには、Web API 全体にわたるグローバルなエラー処理方法を実装することを検討します。 また、各例外の全詳細をキャプチャするエラー ログも組み込む必要があります。Web 経由でクライアントにアクセスできないようにしているのであれば、このエラー ログに詳細情報を含めることができます。

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a>クライアント側のエラーとサーバー側のエラーを区別する

HTTP プロトコルでは、クライアント アプリケーションが原因で発生したエラー (HTTP 4xx 状態コード) と、サーバーでの障害が原因で発生したエラー (HTTP 5xx 状態コード) を区別します。 すべてのエラー応答メッセージでこの規約を遵守していることを確認します。

## <a name="optimizing-client-side-data-access"></a>クライアント側のデータ アクセスの最適化

Web サーバーとクライアント アプリケーションが関与する環境など、分散環境で懸念の主な原因の 1 つとなっているのがネットワークです。 特に、クライアント アプリケーションが要求の送信やデータの受信を頻繁に行う場合、これがかなりのボトルネックになる可能性があります。 そのため、ネットワーク上を流れるトラフィックの量を最小限に抑えることを目標にします。 データを取得して保持するコードを実装するときには、以下の点を考慮してください。

### <a name="support-client-side-caching"></a>クライアント側のキャッシュをサポートする

HTTP 1.1 プロトコルでは、Cache-Control ヘッダーを使用することで、クライアントと要求のルーティングで経由する中間サーバーでのキャッシュをサポートしています。 クライアント アプリケーションが Web API に HTTP GET 要求を送信したときは、クライアントまたは要求のルーティングで経由した中間サーバーで応答の本文のデータを安全にキャッシュできるかどうかと、データを期限切れにして古いデータと見なすまでの時間を示す Cache-Control ヘッダーを応答に含めることができます。 次の例は、HTTP GET 要求と、Cache-Control ヘッダーが含まれた対応する応答を示しています。

```HTTP
GET https://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

この例では、Cache-Control ヘッダーは、返されるデータを 600 秒後に期限切れにすることと、返されるデータは 1 つのクライアント専用であり、他のクライアントが使用する共有キャッシュには保存できないこと (*private*) を指定しています。 データを共有キャッシュに保存できる場合は、Cache-Control ヘッダーで *private* ではなく *public* を指定できます。また、データをクライアントでキャッシュ**できない**場合は、*no-store* を指定できます。 次のコード例は、応答メッセージの Cache-Control ヘッダーを作成する方法を示しています。

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

このコードでは、`OkResultWithCaching` という名前のカスタム `IHttpActionResult` クラスを使用しています。 このクラスを使用して、コントローラーはキャッシュ ヘッダーの内容を設定できます。

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> HTTP プロトコルでは、Cache-Control ヘッダーの *no-cache* ディレクティブも定義されています。 少し紛らわしいのですが、このディレクティブは "キャッシュしない" ことを意味するのではなく、"キャッシュされた情報を返す前に、サーバーでその情報を再検証する" ことを意味します。データは、引き続きキャッシュに保存できますが、使用するたびにチェックされ、最新であることが確認されます。

キャッシュ管理は、クライアント アプリケーションまたは中間サーバーが行いますが、キャッシュ管理を適切に実装すると、最近既に取得したデータを取得する必要性が排除されるので、帯域幅を節約し、パフォーマンスを向上させることができます。

Cache-Control ヘッダーの *max-age* 値は 1 つの目安にすぎず、指定した期間に対応するデータが変更されないことを保証するわけではありません。 Web API では、データの予想される揮発性に応じて、max-age を適切な値に設定する必要があります。 この期間が経過したら、クライアントはキャッシュからオブジェクトを破棄する必要があります。

> [!NOTE]
> 最新の Web ブラウザーは、記述に従って適切な Cache-Control ヘッダーを要求に追加し、結果のヘッダーを調べることで、クライアント側のキャッシュをサポートします。 ただし、一部の古いブラウザーは、クエリ文字列を含む URL から返された値をキャッシュしません。 通常、これは、ここで説明したプロトコルに基づく独自のキャッシュ管理方法を実装しているカスタム クライアント アプリケーションの問題ではありません。
>
> 一部の古いプロキシも同じ動作を示し、クエリ文字列を含む URL に基づく要求をキャッシュしないことがあります。 これは、このようなプロキシ経由で Web サーバーに接続するカスタム クライアント アプリケーションの問題である場合があります。

### <a name="provide-etags-to-optimize-query-processing"></a>ETag を提供してクエリ処理を最適化する

クライアント アプリケーションがオブジェクトを取得するときに、応答メッセージに *ETag* (エンティティ タグ) を含めることもできます。 ETag は、リソースのバージョンを示す不透明な文字列です。リソースが変更されるたびに、Etag も変更されます。 この ETag は、クライアント アプリケーションでデータの一部としてキャッシュする必要があります。 次のコード例は、HTTP GET 要求への応答の一部として ETag を追加する方法を示しています。 このコードでは、オブジェクトの `GetHashCode` メソッドを使用して、オブジェクトを識別する数値を生成します (必要に応じてこのメソッドをオーバーライドし、MD5 などのアルゴリズムを使用して独自のハッシュを生成できます)。

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

Web API によってポストされる応答メッセージは次のようになります。

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> セキュリティ上の理由から、機密データや認証された (HTTPS) 接続経由で返されるデータのキャッシュは許可しないでください。

クライアント アプリケーションは、次の GET 要求を発行して同じリソースをいつでも取得できます。リソースが変更されている場合 (別の ETag が含まれている場合) は、キャッシュされたバージョンを破棄し、新しいバージョンをキャッシュに追加する必要があります。 リソースのサイズが大きく、クライアントに送信する際に大量の帯域幅を必要とする場合、同じデータを取得する要求を繰り返すと、非効率的になる可能性があります。 これを解決するために、HTTP プロトコルでは、Web API でサポートする必要がある、GET 要求を最適化するための次のプロセスが定義されています。

- クライアントは、If-None-Match HTTP ヘッダーで参照されるリソースの現在キャッシュされているバージョンの ETag を含む GET 要求を作成します。

    ```HTTP
    GET https://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```

- Web API の GET 操作で、要求されたデータ (前の例では order 2) の現在の ETag を取得し、If-None-Match ヘッダーの値と比較します。

- 要求されたデータの現在の ETag が要求で提供された ETag と一致した場合、リソースは変更されていないので、Web API は空のメッセージ本文と状態コード 304 (Not Modified) を含む HTTP 応答を返す必要があります。

- 要求されたデータの現在の ETag が要求で提供された ETag と一致しない場合は、データが変更されているので、Web API は新しいデータを含めたメッセージ本文と状態コード 200 (OK) を含む HTTP 応答を返す必要があります。

- 要求されたデータが既に存在しない場合、Web API は、状態コード 404 (Not Found) を含む HTTP 応答を返す必要があります。

- クライアントは、状態コードを使用してキャッシュを管理します。 データが変更されていない場合 (状態コード 304)、オブジェクトはキャッシュされた状態を維持することができ、クライアント アプリケーションはオブジェクトのこのバージョンを引き続き使用します。 データが変更されている場合 (状態コード 200) は、キャッシュされたオブジェクトを破棄し、新しいオブジェクトを挿入します。 データが使用できなくなっている場合 (状態コード 404) は、オブジェクトをキャッシュから削除します。

> [!NOTE]
> 応答ヘッダーに Cache-Control ヘッダーの no-store が含まれている場合は、HTTP 状態コードに関係なく、オブジェクトをキャッシュから常に削除する必要があります。

次のコードは、If-None-Match ヘッダーをサポートするように拡張された `FindOrderByID` メソッドを示しています。 If-None-Match ヘッダーを省略すると、指定した注文が常に取得されることに注意してください。

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

次の例では、`EmptyResultWithCaching` という名前の追加のカスタム `IHttpActionResult` クラスが組み込まれています。 このクラスは、応答の本文を含まない、 `HttpResponseMessage` オブジェクトのラッパーとして機能します。

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> この例では、基になるデータ ソースから取得したデータをハッシュすることによって、データの ETag が生成されます。 他の方法で ETag を計算できる場合は、このプロセスをさらに最適化できます。データが変更されている場合、データ ソースから取得する必要があるのはそのデータだけになります。 この方法は、データのサイズが大きい場合や、データ ソースにアクセスするとかなりの待機時間が発生する可能性がある場合 (データ ソースがリモート データベースの場合など) に特に役立ちます。

### <a name="use-etags-to-support-optimistic-concurrency"></a>Etag を使用してオプティミスティック コンカレンシーをサポートする

以前にキャッシュされたデータに対する更新を可能にするために、HTTP プロトコルでは、オプティミスティック コンカレンシーをサポートしています。 リソースの取得またはキャッシュ後に、クライアント アプリケーションがリソースを変更または削除する PUT 要求または DELETE 要求を送信する場合、ETag を参照する If-Match ヘッダーを含める必要があります。 次のように、Web API はこの情報を使用して、リソースが取得された後に別のユーザーによって既に変更されているかどうかを確認し、クライアント アプリケーションに適切な応答を送信できます。

- クライアントは、リソースの新しい詳細と、If-Match HTTP ヘッダーで参照されるリソースの現在キャッシュされているバージョンの ETag を含む PUT 要求を作成します。 次の例は、注文を更新する PUT 要求を示しています。

    ```HTTP
    PUT https://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```

- Web API の PUT 操作で、要求されたデータ (前の例では order 1) の現在の ETag を取得し、If-Match ヘッダーの値と比較します。

- 要求されたデータの現在の ETag が要求で提供された ETag と一致した場合、リソースは変更されていないので、Web API は更新を実行します。更新が正常に完了したら、HTTP 状態コード 204 (No Content) を含むメッセージを返す必要があります。 応答には、Cache-Control ヘッダーとリソースの更新バージョンの ETag ヘッダーを含めることができます。 応答には、新しく更新されたリソースの URI を参照する Location ヘッダーを必ず含める必要があります。

- 要求されたデータの現在の ETag が要求で提供された ETag と一致しない場合は、データを取得した後に別のユーザーによってデータが変更されているので、Web API は空のメッセージ本文と状態コード 412 (Precondition Failed) を含む HTTP 応答を返す必要があります。

- 更新対象のリソースが既に存在しない場合、Web API は状態コード 404 (Not Found) を含む HTTP 応答を返す必要があります。

- クライアントは、状態コードと応答ヘッダーを使用してキャッシュを管理します。 データが更新された場合 (状態コード 204)、オブジェクトはキャッシュされた状態を維持できますが (Cache-Control ヘッダーで no-store が指定されていない場合)、ETag を更新する必要があります。 データが別のユーザーによって変更されている場合 (状態コード 412)、またはデータが見つからない場合 (状態コード 404) は、キャッシュされたオブジェクトを破棄する必要があります。

次のコード例は、Orders コントローラーの PUT 操作の実装を示しています。

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> If-Match ヘッダーの使用は完全なオプションです。このヘッダーを省略すると、Web API は指定した注文を常に更新しようとするので、別のユーザーが行った更新が無条件に上書きされる可能性があります。 更新データの損失に起因する問題を回避するには、常に If-Match ヘッダーを指定します。

## <a name="handling-large-requests-and-responses"></a>サイズの大きい要求と応答の処理

クライアント アプリケーションは、サイズが数メガバイト (以上) になる可能性のあるデータを送受信する要求を発行することが必要な場合があります。 この量のデータが送信されるまで待機すると、クライアント アプリケーションが応答しなくなる可能性があります。 大量のデータを含む要求を処理する必要がある場合は、以下の点を考慮してください。

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a>ラージ オブジェクトを伴う要求と応答を最適化する

グラフィック イメージや他の種類のバイナリ データなど、リソースがラージ オブジェクトであったり、大規模なフィールドが含まれていたりする場合があります。 Web API は、このようなリソースのアップロードとダウンロードを最適化できるように、ストリーミングをサポートする必要があります。

HTTP プロトコルは、ラージ データ オブジェクトをクライアントにストリーミングするためのチャンク転送エンコード メカニズムを提供します。 クライアントがラージ オブジェクトの HTTP GET 要求を送信したときに、Web API は HTTP 接続を介して段階的な "*チャンク*" で応答を送信できます。 応答のデータの長さが最初はわからない場合があるので (データが生成される場合があります)、Web API をホストするサーバーは、Content-Length ヘッダーではなく Transfer-Encoding: Chunked ヘッダーを指定した各チャンクを含む応答メッセージを送信する必要があります。 クライアント アプリケーションは、各チャンクを順に受信して完全な応答を作成できます。 サーバーが、サイズがゼロの最後のチャンクを送信すると、データ転送が完了します。

1 つの要求により、大量のリソースを消費する大規模なオブジェクトが生成される可能性があります。 Web API は、ストリーミング プロセスの途中で要求のデータ量が許容範囲を超えたことを確認すると、操作を中止し、状態コード 413 (Request Entity Too Large) を含む応答メッセージを返します。

HTTP 圧縮を使用すると、ネットワーク経由で送信されるラージ オブジェクトのサイズを最小限に抑えることができます。 この方法では、ネットワーク トラフィックの量と、関連するネットワーク待ち時間を削減できますが、クライアントと Web API をホストするサーバーで追加の処理が必要となります。 たとえば、圧縮データを受信することを求めるクライアント アプリケーションは、Accept-Encoding: gzip 要求ヘッダーを含めることができます (他のデータ圧縮アルゴリズムを指定することもできます)。 サーバーが圧縮をサポートしている場合、メッセージ本文に gzip 形式で保持されたコンテンツと、Content-Encoding: gzip 応答ヘッダーを使用して応答する必要があります。

エンコードされた圧縮をストリーミングと組み合わせることができます。ストリーミングの前に、まずデータを圧縮し、メッセージのヘッダーに gzip コンテンツ エンコードとチャンク転送エンコードを指定します。 また、一部の Web サーバー (Internet Information Server など) は、Web API でデータを圧縮するかどうかに関係なく、HTTP 応答を自動的に圧縮するように構成できます。

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a>非同期操作をサポートしていないクライアントでは部分的な応答を実装する

非同期ストリーミングの代わりに、クライアント アプリケーションは、ラージ オブジェクトのデータをチャンク単位で明示的に要求できます。これは部分的な応答と呼ばれます。 クライアント アプリケーションは、HTTP HEAD 要求を送信して、オブジェクトに関する情報を取得します。 Web API が部分的な応答をサポートしている場合は、Accept-Ranges ヘッダーとオブジェクトの合計サイズを示す Content-Length ヘッダーを含み、メッセージの本文を空にした応答メッセージで HEAD 要求に応答する必要があります。 クライアント アプリケーションは、この情報を使用して、受信するバイト範囲を指定した一連の GET 要求を作成できます。 Web API は、HTTP 状態コード 206 (Partial Content)、応答メッセージの本文に含まれるデータの実際の量を指定した Content-Length ヘッダー、このデータが表すオブジェクトの部分 (4000 ～ 8000 バイトなど) を示す Content-Range ヘッダーを含む応答メッセージを返す必要があります。

HTTP HEAD 要求と部分的な応答の詳細については、[API 設計][api-design]をご覧ください。

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a>クライアント アプリケーションで不要な 100-Continue 状態メッセージを送信しない

大量のデータをサーバーに送信しようとしているクライアント アプリケーションは、サーバーが実際にその要求を受け入れる用意があるかどうかを最初に確認できます。 クライアント アプリケーションは、データを送信する前に、Expect: 100-Continue ヘッダー、データのサイズを示す Content-Length ヘッダー、空のメッセージ本文を含む HTTP 要求を送信できます。 サーバーが要求を処理する用意がある場合、HTTP 状態コード 100 (Continue) を指定したメッセージで応答する必要があります。 クライアント アプリケーションは要求を続行し、メッセージ本文にデータが含まれた完全な要求を送信できます。

IIS を使用してサービスをホストしている場合は、Web アプリケーションに要求を渡す前に、HTTP.sys ドライバーによって Expect: 100-Continue ヘッダーが自動的に検出され、処理されます。 つまり、アプリケーション コードでこれらのヘッダーを使用することはほとんどありません。不適切であったり、大きすぎると思われるメッセージは、IIS によって既にフィルター処理されていることを前提にすることができます。

.NET Framework を使用してクライアント アプリケーションを構築している場合は、すべての POST メッセージと PUT メッセージで、Expect: 100-Continue ヘッダーを既定で含むメッセージが最初に送信されます。 サーバー側と同様に、このプロセスは .NET Framework によって透過的に処理されます。 ただし、このプロセスでは、要求のサイズが小さい場合でも、各 POST 要求と PUT 要求でサーバーへのラウンド トリップが 2 回発生することになります。 アプリケーションが大量のデータを含む要求を送信しない場合は、クライアント アプリケーションで `ServicePointManager` クラスを使用して `ServicePoint` オブジェクトを作成することで、この機能を無効にすることができます。 `ServicePoint` オブジェクトは、サーバー上のリソースを特定する URI のスキームとホスト フラグメントに基づいて、クライアントが行うサーバーへの接続を処理します。 その後、`ServicePoint` オブジェクトの `Expect100Continue` プロパティを false に設定できます。 `ServicePoint` オブジェクトのスキームおよびホスト フラグメントと一致する URI を使用してクライアントで行われる後続の POST 要求と PUT 要求は、すべて Expect: 100-Continue ヘッダーなしで送信されます。 次のコードは、スキームが `http` で、ホストが `www.contoso.com` である URI に送信されるすべての要求を構成する `ServicePoint` オブジェクトを構成する方法を示しています。

```csharp
Uri uri = new Uri("https://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

`ServicePointManager` クラスの静的 `Expect100Continue` プロパティを設定して、今後作成されるすべての [ServicePoint](/dotnet/api/system.net.servicepoint) オブジェクトのこのプロパティの既定値を指定することもできます。

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a>多数のオブジェクトを返す可能性のある要求の改ページ調整をサポートする

コレクションに多数のリソースが含まれている場合、対応する URI に GET 要求を発行すると、Web API をホストするサーバーでパフォーマンスに影響する大量の処理が発生し、待機時間の増加をもたらす大量のネットワーク トラフィックが生成される可能性があります。

このような状況に対応するには、Web API でクエリ文字列をサポートする必要があります。クエリ文字列により、クライアント アプリケーションは、要求の調整や管理しやすい個別のブロック (またはページ) でのデータの取得が可能になります。 次のコードは、`Orders` コントローラーの `GetAllOrders` メソッドを示しています。 このメソッドは注文の詳細を取得します。 このメソッドに制約がない場合、大量のデータが返される可能性があります。 `limit` パラメーターと `offset` パラメーターは、データの量を減らして小さなサブセットにすることを目的としています。この例では、既定で最初 10 個の注文に制限されています。

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

クライアント アプリケーションは、URI `https://www.adventure-works.com/api/orders?limit=30&offset=50` を使用して、オフセット 50 で始まる 30 個の注文を取得する要求を発行できます。

> [!TIP]
> URI が 2000 文字を超える長さになるクエリ文字列をクライアント アプリケーションが指定できないようにします。 多くの Web クライアントとサーバーは、この長さの URI を処理できません。

## <a name="maintaining-responsiveness-scalability-and-availability"></a>応答性、スケーラビリティ、可用性の維持

世界のあらゆる場所で実行される多数のクライアント アプリケーションが、同じ Web API を使用する場合があります。 負荷が大きい状況で応答性を維持し、大きく変化するワークロードをサポートするスケーラビリティを備え、業務に不可欠な操作を実行するクライアントの可用性を保証するように Web API が実装されていることが重要です。 これらの要件を満たす方法を決めるときには、以下の点を考慮してください。

### <a name="provide-asynchronous-support-for-long-running-requests"></a>実行時間の長い要求の非同期サポートを提供する

処理に時間がかかると考えられる要求は、その要求を送信したクライアントをブロックせずに実行する必要があります。 Web API では、要求を検証する初期チェックを実行しし、処理を実行する別のタスクを開始した後、HTTP コード 202 (Accepted) を含む応答メッセージを返すことができます。 タスクは、Web API 処理の一部として非同期的に実行できます。またはバック グラウンド タスクとしてオフロードできます。

Web API では、処理の結果をクライアント アプリケーションに返すためのメカニズムも提供する必要があります。 これを実現するには、クライアント アプリケーションが、処理が完了したかどうかを定期的に照会し、結果を取得するためのポーリング メカニズムを提供するか、Web API が操作の完了時に通知を送信できるようにします。

次の方法を使用して仮想リソースとして機能する *polling* URI を提供することで、単純なポーリング メカニズムを実装できます。

1. クライアント アプリケーションが Web API に最初の要求を送信します。
2. Web API は、テーブル ストレージまたは Microsoft Azure Cache に保持されるテーブルに要求に関する情報を保存し、このエントリの一意のキーを、場合によっては GUID の形式で生成します。
3. Web API は別のタスクとして処理を開始します。 Web API は、タスクの状態を *Running* としてテーブルに記録します。
4. Web API は、HTTP 状態コード 202 (Accepted) と、テーブル エントリの GUID をメッセージの本文として含む応答メッセージを返します。
5. タスクが完了したら、Web API は結果をテーブルに保存し、タスクの状態を *Complete* に設定します。 タスクが失敗した場合、Web API は失敗に関する情報を保存し、状態を *Failed* に設定することもできます。
6. タスクの実行中、クライアントは独自の処理を引き続き実行できます。 クライアントは、要求を URI */polling/{guid}* に定期的に送信できます。*{guid}* は、Web API から 202 応答メッセージで返される GUID です。
7. Web API は、*/polling/{guid}* URI でテーブル内の対応するタスクの状態を照会し、HTTP 状態コード 200 (OK) と、この状態 (*Running*、*Complete*、または *Failed*) を含む応答メッセージを返します。 タスクが完了または失敗した場合、処理の結果や失敗の原因に関する入手可能な情報を応答メッセージに含めることもできます。

通知を実装するためのオプションは次のとおりです。

- Azure Notification Hubs を使用して、非同期応答をクライアント アプリケーションにプッシュする。 詳細については、「[Azure Notification Hubs によるユーザーへの通知](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)」を参照してください。
- Comet モデルを使用して、クライアントと Web API をホストするサーバー間の永続的なネットワーク接続を保持し、この接続を使用してサーバーからのメッセージをクライアントにプッシュする。 ソリューションの例については、MSDN マガジンの記事「 [Microsoft .NET Framework でシンプルな Comet アプリケーションをビルドする](https://msdn.microsoft.com/magazine/jj891053.aspx) 」をご覧ください。
- SignalR を使用して、永続的なネットワーク接続経由で Web サーバーからクライアントにリアルタイムでデータをプッシュする。 SignalR は、NuGet パッケージとして ASP.NET Web アプリケーションで使用できます。 詳細については、 [ASP.NET SignalR](https://www.asp.net/signalr) の Web サイトをご覧ください。

### <a name="ensure-that-each-request-is-stateless"></a>各要求がステートレスであることを確認する

各要求はアトミックと見なす必要があります。 クライアント アプリケーションによって行われるある要求と、同じクライアントによって送信される後続の要求との間に依存関係が存在しないようにする必要があります。 この方法はスケーラビリティを支援します。つまり、多数のサーバーで Web サービスのインスタンスをデプロイできます。 これらのどのインスタンスでもクライアント要求を送信できるので、結果が常に同じである必要があります。 同様の理由から可用性も向上します。Web サーバーで障害が発生した場合、(Azure Traffic Manager を使用して) 要求を別のインスタンスにルーティングすることができ、クライアント アプリケーションに悪影響を与えずにサーバーが再起動されます。

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a>クライアントを追跡し、調整を実装して DOS 攻撃の危険性を低減する

特定のクライアントが一定の期間内に多数の要求を行うと、サービスが独占され、他のクライアントのパフォーマンスに影響を及ぼす可能性があります。 この問題を軽減するには、Web API ですべての受信要求の IP アドレスを追跡するか、各認証済みアクセスをログに記録して、クライアント アプリケーションからの呼び出しを監視します。 この情報を使用して、リソースへのアクセスを制限できます。 クライアントが定義済みの上限を超えた場合、Web API は状態コード 503 (Service Unavailable) を含む応答メッセージを返し、クライアントが拒否されずに次の要求を送信できる時間を指定した Retry-After ヘッダーを含めることができます。 この方法により、システムを停止させる、一連のクライアントからのサービス拒否 (DOS) 攻撃の可能性を低減できます。

### <a name="manage-persistent-http-connections-carefully"></a>永続的な HTTP 接続を慎重に管理する

HTTP プロトコルは、永続的な HTTP 接続をサポートしています。 HTTP 1.0 仕様で Connection:Keep-Alive ヘッダーが追加されました。このヘッダーにより、クライアント アプリケーションは、新しい接続を開くのではなく、同じ接続を使用して後続の要求を送信できることをサーバーに示すことができます。 ホストで定義されている期間内にクライアントが接続を再利用していない場合は、接続が自動的に閉じられます。 これは、Azure サービスで使用される HTTP 1.1 での既定の動作であるため、Keep-Alive ヘッダーをメッセージに含める必要はありません。

接続を開いたままにしておくと、待機時間とネットワークの輻輳が低減されるので、応答性を向上させることができますが、不要な接続を必要以上に開いたままにしておくと、他の同時実行クライアントによる接続が制限され、スケーラビリティに悪影響を及ぼす可能性があります。 また、クライアント アプリケーションがモバイル デバイスで実行されている場合は、バッテリの寿命にも影響する可能性があります。アプリケーションがサーバーに要求することがあまりない場合、開いている接続を維持すると、バッテリ消費が早くなる可能性があります。 HTTP 1.1 で接続が永続化されないようにするには、クライアントが Connection:Close ヘッダーをメッセージに含め、既定の動作をオーバーライドします。 同様に、サーバーが非常に多くのクライアントに対応している場合は、応答メッセージに Connection:Close ヘッダーを含めることで、接続を閉じ、サーバー リソースを節約します。

> [!NOTE]
> 永続的な HTTP 接続は、通信チャネルを何度も確立することに関連して発生するネットワークのオーバーヘッドを削減するための完全なオプション機能です。 Web API とクライアント アプリケーションは、どちらも永続的な HTTP 接続が使用可能であることに依存しないようにする必要があります。 永続的な HTTP 接続を使用して、Comet スタイルの通知システムを実装しないでください。代わりに、TCP レイヤーで Socket (使用可能な場合は WebSocket) を利用します。 最後に、クライアント アプリケーションがプロキシ経由でサーバーと通信する場合、Keep-Alive ヘッダーの使用が制限されます。クライアントおよびプロキシとの接続だけが永続化されます。

## <a name="publishing-and-managing-a-web-api"></a>Web API の公開と管理

Web API をクライアント アプリケーションで使用できるようにするには、Web API をホスト環境にデプロイする必要があります。 通常、この環境は Web サーバーですが、他の種類のホスト プロセスの場合もあります。 Web API を公開するときには、以下の点を考慮してください。

- すべての要求を認証および承認する必要があり、適切なレベルのアクセス制御を適用する必要があります。
- 商用の Web API は、応答時間に関するさまざまな品質保証の対象となる場合があります。 時間の経過に伴って負荷が大きく変わる可能性がある場合は、ホスト環境のスケーラビリティを確保することが重要です。
- 収益化のために要求を測定することが必要な場合があります。
- Web API へのトラフィック フローを調整し、クォータを使い果たしたクライアントのために調整を実装することが必要な場合があります。
- 規制の要件により、すべての要求と応答のログと監査が義務付けられる場合があります。
- 可用性を確保するために、Web API をホストするサーバーの正常性を監視し、必要に応じてサーバーを再起動することが必要な場合があります。

これらの問題を、Web API の実装に関する技術的な問題から切り離すことができると便利です。 そのため、別のプロセスとして実行され、要求を Web API にルーティングする [ファサード](https://en.wikipedia.org/wiki/Facade_pattern)を作成することを検討してください。 ファサードで管理操作を提供し、検証済みの要求を Web API に転送できます。 また、ファサードを使用すると、次のような機能面での多くの利点がもたらされます。

- 複数の Web API の統合ポイントとして機能する。
- さまざまなテクノロジを使用して構築されたクライアントに対応するために、メッセージと通信プロトコルを変換する。
- Web API をホストするサーバーの負荷を軽減するために、要求と応答をキャッシュする。

## <a name="testing-a-web-api"></a>Web API のテスト

ソフトウェアの他の部分と同様に、Web API を徹底的にテストする必要があります。 機能を検証する単体テストを作成することを検討してください。

Web API の性質により、Web API が正しく動作することを確認するための独自の追加要件が発生します。 次の側面には特に注意が必要です。

- すべてのルートをテストして、正しい操作が呼び出されていることを確認します。 HTTP 状態コード 405 (Method Not Allowed) は、ルートとそのルートにディスパッチできる HTTP メソッド (GET、POST、PUT、DELETE) の不一致を示している場合があるため、この状態コードが予期せず返されたときは特に注意してください。

    HTTP 要求をサポートしていないルートに要求を送信します。たとえば、POST 要求を特定のリソースに送信します (POST 要求の送信先はリソース コレクションに限られます)。 このような場合、状態コード 405 (Not Allowed) が唯一の有効な応答である "*必要があります*"。

- すべてのルートが適切に保護され、適切な認証および承認チェックの対象となっていることを確認します。

  > [!NOTE]
  > ユーザー認証など、セキュリティの一部の側面は、Web API ではなくホスト環境の責任範囲と考えられますが、デプロイメント プロセスの一環として、セキュリティ テストも含める必要があります。
  >
  >

- 各操作で実行される例外処理をテストし、意味のある適切な HTTP 応答がクライアント アプリケーションに返されていることを確認します。

- 要求メッセージと応答メッセージが適切な形式であることを確認します。 たとえば、HTTP POST 要求に x-www-form-urlencoded 形式で新しいリソースのデータが含まれている場合は、対応する操作でデータが正しく解析されていること、リソースが作成されていること、正しい Location ヘッダーなど、新しいリソースの詳細を含む応答が返されていることを確認します。

- 応答メッセージのすべてのリンクと URI を確認します。 たとえば、HTTP POST メッセージでは、新しく作成されたリソースの URI を返す必要があります。 すべての HATEOAS リンクが有効である必要があります。

- 各操作が入力のさまざまな組み合わせに対して正しい状態コードを返していることを確認します。 例: 

  - クエリが成功した場合は、状態コード 200 (OK) を返します。
  - リソースが見つからない場合は、HTTP 状態コード 404 (Not Found) を返します。
  - クライアントがリソースを正常に削除する要求を送信した場合、状態コードは 204 (No Content) になります。
  - クライアントが新しいリソースを作成する要求を送信した場合、状態コードは 201 (Created) になります。

5xx の範囲の予期しない応答ステータ スコードに注意します。 これらのメッセージは、有効な要求を処理できなかったことを示すために、通常はホスト サーバーによって報告されます。

- クライアント アプリケーションが指定できる要求ヘッダーのさまざま組み合わせをテストし、Web API が求められている情報を応答メッセージで返していることを確認します。

- クエリ文字列をテストします。 操作が省略可能なパラメーター (改ページ調整要求など) を受け取ることができる場合は、パラメーターのさまざまな組み合わせと順序をテストします。

- 非同期操作が正常に完了していることを確認します。 Web API がラージ バイナリ オブジェクト (ビデオやオーディオなど) を返す要求のストリーミングをサポートしている場合は、データのストリーミング中にクライアント要求がブロックされていないことを確認します。 Web API が実行時間の長いデータ変更操作のポーリングを実装している場合は、操作の進行に伴って状態が正しく報告されていることを確認します。

パフォーマンス テストを作成して実行し、Web API が強制下で十分に機能していることを確認します。 Visual Studio Ultimate を使用して、Web パフォーマンスおよびロード テスト プロジェクトを作成できます。 詳細については、[リリース前のアプリケーションでのパフォーマンス テストの実行](https://msdn.microsoft.com/library/dn250793.aspx)に関するページをご覧ください。

## <a name="using-azure-api-management"></a>Azure API Management の使用

Azure で、Web API の公開と管理に [Azue API Management](/azure/api-management//services/api-management/) を使用することを検討します。 この機能を使用して、1 つ以上の Web API のファサードとして機能するサービスを生成できます。 このサービスは、Microsoft Azure 管理ポータルを使用して作成および構成できるスケーラブルな Web サービスです。 次のように、このサービスを使用して Web API を公開および管理できます。

1. Web サイト、Azure クラウド サービス、または Azure 仮想マシンに Web API をデプロイします。

2. API Management サービスを Web API に接続します。 管理 API の URL に送信された要求は、Web API の URI にマップされます。 1 つの API Management サービスで、複数の Web API に要求をルーティングできます。 これにより、複数の Web API を 1 つの管理サービスに集約できます。 同様に、さまざまなアプリケーションが使用できる機能を制限したり、パーティションに分割したりする必要がある場合は、複数の API Management サービスから 1 つの Web API を参照できます。

     > [!NOTE]
     > HTTP GET 要求に対する応答の一部として生成される HATEOAS リンクの URI は、Web API をホストする Web サーバーではなく、API Management サービスの URL を参照する必要があります。

3. Web API ごとに、その Web API が公開する HTTP 操作と、操作が入力として受け取ることができる省略可能なパラメーターを指定します。 同じデータに対して繰り返される要求を最適化するために、Web API から受信する応答を API Management サービスでキャッシュする必要があるかどうかを構成することもできます。 各操作で生成できる HTTP 応答の詳細を記録します。 この情報を使用して開発者向けのドキュメントが生成されるので、情報が正確かつ完全なものであることが重要です。

    操作は、Microsoft Azure 管理ポータルに用意されているウィザードを使用して手動で定義することも、定義が含まれたファイルから WADL 形式または Swagger 形式でインポートすることもできます。

4. API Management サービスと Web API をホストする Web サーバー間の通信のセキュリティ設定を構成します。 現在、API Management サービスは、証明書を使用した基本認証と相互認証、および OAuth 2.0 ユーザー認証をサポートしています。

5. 成果物を作成します。 成果物とはパブリケーションの単位です。前に管理サービスに接続した Web API を成果物に追加します。 成果物を発行すると、それらの Web API を開発者が使用できるようになります。

    > [!NOTE]
    > 成果物を発行する前に、その成果物にアクセスできるユーザー グループを定義し、それらのグループにユーザーを追加することもできます。 これにより、Web API を使用できる開発者とアプリケーションを制御できます。 Web API が承認を必要とする場合、Web API にアクセスするには、開発者が成果物管理者に要求を送信しておく必要があります。 管理者は、開発者に対してアクセスを許可または拒否できます。 状況が変わった場合は、既存の開発者をブロックすることもできます。

6. 各 Web API のポリシーを構成します。 ポリシーでは、ドメイン間呼び出しを許可するかどうか、クライアントの認証方法、XML データ形式と JSON データ形式間で透過的な変換を行うかどうか、特定の IP 範囲からの呼び出しを制限するかどうか、使用量のクォータ、呼び出しレートを制限するかどうかなどの側面を制御します。 ポリシーは、成果物全体にグローバルに適用することも、成果物の 1 つの Web API に適用することもできます。また、Web API の個々の操作に適用することもできます。

詳細については、[API Management のドキュメント](/azure/api-management/)をご覧ください。

> [!TIP]
> Azure には、フェールオーバーと負荷分散を実装し、さまざまな地理的な場所でホストされている Web サイトの複数のインスタンスで待機時間を短縮できる Azure Traffic Manager が用意されています。 Azure Traffic Manager は、API Management サービスと組み合わせて使用できます。API Management サービスでは、Azure Traffic Manager を使用して Web サイトのインスタンスに要求をルーティングできます。 詳細については、「[Traffic Manager のルーティング方法](/azure/traffic-manager/traffic-manager-routing-methods/)」を参照してください。
>
> この構造では、Web サイトにカスタム DNS 名を使用している場合に、各 Web サイトの適切な CNAME レコードを、Azure Traffic Manager の Web サイトの DNS 名を指すように構成する必要があります。

## <a name="supporting-client-side-developers"></a>クライアント側の開発者のサポート

一般に、クライアント アプリケーションを構築する開発者は、Web API へのアクセス方法に関する情報と、パラメーター、データ型、戻り値の型、Web サービスとクライアント アプリケーション間のさまざまな要求と応答を示すリターン コードに関するドキュメントを必要としています。

### <a name="document-the-rest-operations-for-a-web-api"></a>Web API の REST 操作に関するドキュメント

Azure API Management サービスには、Web API で公開される REST 操作について説明する開発者向けポータルが用意されています。 成果物を発行すると、このポータルに表示されます。 開発者は、このポータルを使用してアクセスへのサインアップを行うことができます。管理者は、この要求を承認または拒否できます。 開発者が承認されると、開発されたクライアント アプリケーションからの呼び出しの認証に使用するサブスクリプション キーが割り当てられます。 Web API 呼び出しごとにこのキーを提供する必要があります。そうしないと、呼び出しは拒否されます。

このポータルでは、以下も提供されます。

- 成果物、公開されている操作のリスト、必須パラメーター、および返される可能性のあるさまざまな応答に関するドキュメント。 この情報は、「Microsoft Azure API Management サービスを使用した Web API の公開と管理」の手順 3. に記載されている詳細から生成されます。
- JavaScript、C#、Java、Ruby、Python、PHP など、複数の言語から操作を呼び出す方法を示すコード スニペット。
- 開発者が HTTP 要求を送信して成果物の各操作をテストし、結果を表示できる開発者向けコンソール。
- 開発者が課題や見つかった問題を報告できるページ。

Microsoft Azure 管理ポータルでは、開発者向けポータルをカスタマイズして、組織のブランドに合わせてスタイル設定やレイアウトを変更できます。

### <a name="implement-a-client-sdk"></a>クライアント SDK の実装

REST 要求を呼び出して Web API にアクセスするクライアント アプリケーションを構築するには、各要求の作成と適切な書式設定、Web サービスをホストするサーバーへの要求の送信、応答の解析による要求の成功/失敗の確認と返されたデータの抽出など、大量のコードを作成する必要があります。 クライアント アプリケーションをこれらの問題から切り離すには、REST インターフェイスをラップし、これらの低レベルの詳細をより機能的な一連のメソッド内で抽象化する SDK を提供します。 クライアント アプリケーションでこれらのメソッドを使用して、呼び出しを REST 要求に透過的に変換し、応答をメソッドの戻り値に変換します。 これは、Azure SDK をはじめとする多くのサービスで実装されている一般的な手法です。

クライアント側の SDK を作成する場合、SDK を一貫した方法で実装し、入念にテストする必要があるため、かなりの作業量になります。 ただし、このプロセスの大部分は機械的に行うことができるので、多数のベンダーがこれらのタスクの多くを自動化できるツールを提供しています。

## <a name="monitoring-a-web-api"></a>Web API の監視

Web API を公開し、デプロイした方法に応じて、Web API を直接監視することも、API Management サービスを使用して通過するトラフィックを分析し、利用状況と正常性に関する情報を収集することもできます。

### <a name="monitoring-a-web-api-directly"></a>Web API の直接監視

Visual Studio 2013 と、ASP.NET Web API テンプレートを (Azure クラウド サービスで Web API プロジェクトまたは Web ロールとして) 使用して Web API を実装した場合は、ASP.NET Application Insights を使用して、可用性、パフォーマンス、利用状況のデータを収集できます。 Application Insights は、Web API がクラウドにデプロイされている場合に、要求と応答に関する情報を透過的に追跡して記録するパッケージです。このパッケージをインストールして構成したら、パッケージを使用するために Web API でコードを修正する必要はありません。 Web API を Azure Web サイトにデプロイすると、すべてのトラフィックが検査され、次の統計情報が収集されます。

- サーバーの応答時間。
- サーバー要求の数と各要求の詳細。
- 平均応答時間の観点で最も時間がかかった要求。
- 失敗した要求の詳細。
- さまざまなブラウザーやユーザー エージェントによって開始されたセッションの数。
- 最も頻繁に表示されたページ (Web API ではなく、主に Web アプリケーションで役立ちます)。
- Web API にアクセスする各種ユーザー ロール。

このデータは、Microsoft Azure 管理ポータルからリアルタイムで表示できます。 Web API の正常性を監視する WebTest を作成することもできます。 WebTest では、Web API の指定した URI に定期的な要求を送信し、応答をキャプチャします。 正常な応答の定義 (HTTP 状態コード 200 など) を指定し、要求でこの応答が返されない場合に、管理者に送信するアラートを準備できます。 必要に応じて、管理者は Web API をホストするサーバーで障害が発生した場合にサーバーを再起動できます。

詳細については、[Application Insights - ASP.NET の概要](/azure/application-insights/app-insights-asp-net/)を参照してください。

### <a name="monitoring-a-web-api-through-the-api-management-service"></a>API Management サービスを使用した Web API の監視

API Management サービスを使用して Web API を公開した場合、Microsoft Azure 管理ポータルの API Management ページにあるダッシュボードを使用して、サービスの全体的なパフォーマンスを表示できます。 [分析] ページでは、製品の使用状況の詳細にドリルダウンできます。 このページには次のタブがあります。

- **[使用状況]**:  このタブには、API 呼び出し数とこれらの呼び出しを経時的に処理する際に使用された帯域幅に関する情報が表示されます。 利用状況の詳細は、成果物、API、操作でフィルターできます。
- **[正常性]**:  このタブでは、API 要求の結果 (返された HTTP ステータ スコード)、キャッシュ ポリシーの有効性、API の応答時間、サービスの応答時間を表示できます。 このタブでも、正常性データを成果物、API、操作でフィルターできます。
- **アクティビティ**。 このタブでは、成功した呼び出し数、失敗した呼び出し数、ブロックされた呼び出し数、平均応答時間、成果物、Web API、操作ごとの応答時間がテキストでまとめられています。 このページには、各開発者による呼び出し数も示されます。
- **[概要]**:  このタブには、最も多くの API 呼び出しを行った開発者、これらの呼び出しを受信した成果物、Web API、および操作など、パフォーマンス データの概要が表示されます。

この情報を使用して、特定の Web API または操作がボトルネックの原因となっているかどうかを確認し、必要に応じてホスト環境を拡張したり、サーバーをさらに追加したりできます。 また、1 つ以上のアプリケーションがリソースを過度に使用していないかどうかを確認し、クォータを設定して呼び出しレートを制限する適切なポリシーを適用することもできます。

> [!NOTE]
> 発行された成果物の詳細を変更できます。変更はすぐに適用されます。 たとえば、Web API を含む成果物を再発行しなくても、その Web API に対して操作を追加または削除できます。

## <a name="more-information"></a>詳細情報

- ASP.NET を使用した OData Web API の実装の例と詳細については、[ASP.NET Web API OData](https://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) に関するページをご覧ください。
- OData を使用して Web API でバッチ操作を実装する方法については、[Web API と Web API OData でのバッチ処理のサポートの概要](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/)に関するページをご覧ください。
- べき等の概要とデータ管理操作との関連性については、Jonathan Oliver のブログ記事「[Idempotency Patterns (べき等のパターン)](https://blog.jonathanoliver.com/idempotency-patterns/)」をご覧ください。
- HTTP 状態コードの全一覧と説明については、W3C の Web サイトの「[Status Code Definitions (状態コードの定義)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)」をご覧ください。
- Web ジョブを使用してバックグラウンド操作を実行する方法と例については、Microsoft Web サイト「[Web ジョブでバックグラウンド タスクを実行する](/azure/app-service-web/web-sites-create-web-jobs/)」をご覧ください。
- Azure Notification Hubs を使用して非同期応答をクライアント アプリケーションにプッシュする方法については、「[Azure Notification Hubs によるユーザーへの通知](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)」をご覧ください。
- Web API への制御された安全なアクセスを提供する成果物を発行する方法については、「[API Management](https://azure.microsoft.com/services/api-management/)」をご覧ください。
- API Management REST API を使用して、カスタム管理アプリケーションを構築する方法については、「[Azure API Management REST API リファレンス](https://msdn.microsoft.com/library/azure/dn776326.aspx)」をご覧ください。
- Azure Traffic Manager を使用して、Web API をホストする Web サイトの複数のインスタンスに要求の負荷を分散する方法の概要については、「[Traffic Manager のルーティング方法](/azure/traffic-manager/traffic-manager-routing-methods/)」をご覧ください。
- ASP.NET Web API プロジェクトでの Application Insights のインストールと構成の詳細については、 [ASP.NET での Application Insights の使用](/azure/application-insights/app-insights-asp-net/)に関するページをご覧ください。

<!-- links -->

[api-design]: ./api-design.md
