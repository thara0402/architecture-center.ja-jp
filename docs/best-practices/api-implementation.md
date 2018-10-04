---
title: API 実装ガイダンス
description: API の実装方法に関するガイダンス。
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: fff377d347ce93e9fb83fff1f5a44fe1c7b4dbea
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429402"
---
# <a name="api-implementation"></a><span data-ttu-id="e86f5-103">API 実装</span><span class="sxs-lookup"><span data-stu-id="e86f5-103">API implementation</span></span>

<span data-ttu-id="e86f5-104">入念に設計された REST ベースの Web API では、リソース、関係、およびクライアント アプリケーションにアクセスできるナビゲーション スキームを定義します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-104">A carefully-designed RESTful web API defines the resources, relationships, and navigation schemes that are accessible to client applications.</span></span> <span data-ttu-id="e86f5-105">Web API を実装し、デプロイするときは、データの論理構造ではなく、Web API をホストする環境の物理的な要件と、Web API を構成する方法を検討する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-105">When you implement and deploy a web API, you should consider the physical requirements of the environment hosting the web API and the way in which the web API is constructed rather than the logical structure of the data.</span></span> <span data-ttu-id="e86f5-106">このガイダンスでは、Web API を実装し、Web API を公開してクライアント アプリケーションで使用できるようにするためのベスト プラクティスに重点を置いて説明します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-106">This guidance focusses on best practices for implementing a web API and publishing it to make it available to client applications.</span></span> <span data-ttu-id="e86f5-107">Web API の設計の詳細については、[API 設計](/azure/architecture/best-practices/api-design)をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-107">For detailed information about web API design, see [API Design Guidance](/azure/architecture/best-practices/api-design).</span></span>

## <a name="processing-requests"></a><span data-ttu-id="e86f5-108">要求の処理</span><span class="sxs-lookup"><span data-stu-id="e86f5-108">Processing requests</span></span>

<span data-ttu-id="e86f5-109">要求を処理するコードを実装する場合は、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-109">Consider the following points when you implement the code to handle requests.</span></span>

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a><span data-ttu-id="e86f5-110">GET、PUT、DELETE、HEAD、PATCH の各操作は、べき等にする</span><span class="sxs-lookup"><span data-stu-id="e86f5-110">GET, PUT, DELETE, HEAD, and PATCH actions should be idempotent</span></span>

<span data-ttu-id="e86f5-111">これらの要求を実装するコードが副次的な影響を及ぼさないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-111">The code that implements these requests should not impose any side-effects.</span></span> <span data-ttu-id="e86f5-112">同じリソースに対して繰り返される同じ要求は、同じ状態になる必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-112">The same request repeated over the same resource should result in the same state.</span></span> <span data-ttu-id="e86f5-113">たとえば、応答メッセージの HTTP ステータス コードが異なる場合でも、同じ URI に複数の DELETE 要求を送信したときの結果は同じである必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-113">For example, sending multiple DELETE requests to the same URI should have the same effect, although the HTTP status code in the response messages may be different.</span></span> <span data-ttu-id="e86f5-114">最初の DELETE 要求はステータス コード 204 (No Content) を返す場合があり、その次の DELETE 要求はステータス コード 404 (Not Found) を返す可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-114">The first DELETE request might return status code 204 (No Content), while a subsequent DELETE request might return status code 404 (Not Found).</span></span>

> [!NOTE]
> <span data-ttu-id="e86f5-115">べき等の概要とデータ管理操作との関連性については、Jonathan Oliver のブログ記事「 [Idempotency Patterns (べき等のパターン)](https://blog.jonathanoliver.com/idempotency-patterns/) 」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-115">The article [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a><span data-ttu-id="e86f5-116">新しいリソースを作成する POST 操作が、関連のない副次的な影響を与えない</span><span class="sxs-lookup"><span data-stu-id="e86f5-116">POST actions that create new resources should not have unrelated side-effects</span></span>

<span data-ttu-id="e86f5-117">POST 要求が新しいリソースを作成することを目的としている場合、要求の影響をその新しいリソース (および関連する何らかの結合が存在する場合は直接関連するリソース) に限定する必要があります。たとえば、電子商取引システムにおいて、顧客の新しい注文を作成する POST 要求で、在庫レベルの修正と課金情報の生成も行うことがありますが、その注文に直接関連しない情報を変更したり、システムの全体的な状態に他の副次的な影響を及ぼしたりしないようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-117">If a POST request is intended to create a new resource, the effects of the request should be limited to the new resource (and possibly any directly related resources if there is some sort of linkage involved) For example, in an ecommerce system, a POST request that creates a new order for a customer might also amend inventory levels and generate billing information, but it should not modify information not directly related to the order or have any other side-effects on the overall state of the system.</span></span>

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a><span data-ttu-id="e86f5-118">煩雑な POST、PUT、DELETE 操作を実装しない</span><span class="sxs-lookup"><span data-stu-id="e86f5-118">Avoid implementing chatty POST, PUT, and DELETE operations</span></span>

<span data-ttu-id="e86f5-119">リソース コレクションに対する POST、PUT、DELETE の各要求をサポートします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-119">Support POST, PUT and DELETE requests over resource collections.</span></span> <span data-ttu-id="e86f5-120">POST 要求には複数の新しいリソースの詳細を含めることができ、それらをすべて同じコレクションに追加できます。PUT 要求では、コレクション内のリソース全体を置き換えることができ、DELETE 要求ではコレクション全体を削除できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-120">A POST request can contain the details for multiple new resources and add them all to the same collection, a PUT request can replace the entire set of resources in a collection, and a DELETE request can remove an entire collection.</span></span>

<span data-ttu-id="e86f5-121">ASP.NET Web API 2 に含まれる OData サポートにより、要求をバッチ処理できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-121">The OData support included in ASP.NET Web API 2 provides the ability to batch requests.</span></span> <span data-ttu-id="e86f5-122">クライアント アプリケーションは、複数の Web API 要求をパッケージ化して、それらを 1 つの HTTP 要求でサーバーに送信し、各要求への応答が含まれた 1 つの HTTP 応答を受信できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-122">A client application can package up several web API requests and send them to the server in a single HTTP request, and receive a single HTTP response that contains the replies to each request.</span></span> <span data-ttu-id="e86f5-123">詳細については、「[Introducing Batch Support in Web API and Web API OData (Web API と Web API OData でのバッチ処理のサポートの概要)](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-123">For more information, [Introducing Batch Support in Web API and Web API OData](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/).</span></span>

### <a name="follow-the-http-specification-when-sending-a-response"></a><span data-ttu-id="e86f5-124">応答を送信する場合は、HTTP の仕様に準拠します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-124">Follow the HTTP specification when sending a response</span></span> 

<span data-ttu-id="e86f5-125">Web API は、正しい HTTP 状態コード (クライアントが結果を処理する方法を判断できるようにするため)、適切な HTTP ヘッダー (クライアントが結果の特性を理解できるようにするため)、適切に書式設定された本文 (クライアントが結果を解析できるようにするため) を含むメッセージを返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-125">A web API must return messages that contain the correct HTTP status code to enable the client to determine how to handle the result, the appropriate HTTP headers so that the client understands the nature of the result, and a suitably formatted body to enable the client to parse the result.</span></span> 

<span data-ttu-id="e86f5-126">たとえば、POST 操作は状態コード 201 (Created) を返す必要があり、応答メッセージの Location ヘッダーに新しく作成されたリソースの URI が含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-126">For example, a POST operation should return status code 201 (Created) and the response message should include the URI of the newly created resource in the Location header of the response message.</span></span>

### <a name="support-content-negotiation"></a><span data-ttu-id="e86f5-127">コンテンツ ネゴシエーションをサポートする</span><span class="sxs-lookup"><span data-stu-id="e86f5-127">Support content negotiation</span></span>

<span data-ttu-id="e86f5-128">応答メッセージの本文には、さまざまな形式のデータを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-128">The body of a response message may contain data in a variety of formats.</span></span> <span data-ttu-id="e86f5-129">たとえば、HTTP GET 要求では、データを JSON 形式または XML 形式で返すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-129">For example, an HTTP GET request could return data in JSON, or XML format.</span></span> <span data-ttu-id="e86f5-130">クライアントは、要求の送信時に、クライアントが処理できるデータ形式を指定した Accept ヘッダーを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-130">When the client submits a request, it can include an Accept header that specifies the data formats that it can handle.</span></span> <span data-ttu-id="e86f5-131">これらの形式は、メディアの種類として指定されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-131">These formats are specified as media types.</span></span> <span data-ttu-id="e86f5-132">たとえば、画像を取得する GET 要求を発行するクライアントは、"image/jpeg, image/gif, image/png" のように、クライアントが処理できるメディアの種類を列挙した Accept ヘッダーを指定できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-132">For example, a client that issues a GET request that retrieves an image can specify an Accept header that lists the media types that the client can handle, such as "image/jpeg, image/gif, image/png".</span></span>  <span data-ttu-id="e86f5-133">Web API は、結果を返すときに、これらのメディアの種類のいずれかを使用してデータを書式設定し、応答の Content-Type ヘッダーにその形式を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-133">When the web API returns the result, it should format the data by using one of these media types and specify the format in the Content-Type header of the response.</span></span>

<span data-ttu-id="e86f5-134">クライアントが Accept ヘッダーを指定しない場合は、応答の本文に実用的な既定の形式を使用します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-134">If the client does not specify an Accept header, then use a sensible default format for the response body.</span></span> <span data-ttu-id="e86f5-135">たとえば、ASP.NET Web API フレームワークでは、テキスト ベースのデータに既定で JSON を使用します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-135">As an example, the ASP.NET Web API framework defaults to JSON for text-based data.</span></span>

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a><span data-ttu-id="e86f5-136">HATEOAS スタイルのナビゲーションとリソースの検出をサポートするリンクを提供する</span><span class="sxs-lookup"><span data-stu-id="e86f5-136">Provide links to support HATEOAS-style navigation and discovery of resources</span></span>

<span data-ttu-id="e86f5-137">HATEOAS の手法に従うことによって、クライアントは、はじめの開始ポイントからナビゲートし、リソースを特定できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-137">The HATEOAS approach enables a client to navigate and discover resources from an initial starting point.</span></span> <span data-ttu-id="e86f5-138">これは、URI を含むリンクを使用することで実現されます。クライアントがリソースを取得する HTTP GET 要求を発行したときは、クライアント アプリケーションが直接関連するリソースをすばやく見つけることができるようにするための URI が応答に含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-138">This is achieved by using links containing URIs; when a client issues an HTTP GET request to obtain a resource, the response should contain URIs that enable a client application to quickly locate any directly related resources.</span></span> <span data-ttu-id="e86f5-139">たとえば、電子商取引ソリューションをサポートする Web API では、顧客が多数の注文を行っている場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-139">For example, in a web API that supports an e-commerce solution, a customer may have placed many orders.</span></span> <span data-ttu-id="e86f5-140">クライアント アプリケーションが顧客の詳細を取得するときには、クライアント アプリケーションがこれらの注文を取得する HTTP GET 要求を送信できるようにするためのリンクが応答に含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-140">When a client application retrieves the details for a customer, the response should include links that enable the client application to send HTTP GET requests that can retrieve these orders.</span></span> <span data-ttu-id="e86f5-141">さらに、HATEOAS スタイルのリンクでは、リンクされた各リソースがサポートするその他の操作 (POST、PUT、DELETE など) と、各要求を実行するための対応する URI を示す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-141">Additionally, HATEOAS-style links should describe the other operations (POST, PUT, DELETE, and so on) that each linked resource supports together with the corresponding URI to perform each request.</span></span> <span data-ttu-id="e86f5-142">この方法の詳細については、[API 設計][api-design]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-142">This approach is described in more detail in [API Design][api-design].</span></span>

<span data-ttu-id="e86f5-143">現在、HATEOAS の実装を管理する標準はありませんが、次の例は考えられる 1 つの方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-143">Currently there are no standards that govern the implementation of HATEOAS, but the following example illustrates one possible approach.</span></span> <span data-ttu-id="e86f5-144">この例では、顧客の詳細を検索する HTTP GET 要求は、その顧客の注文を参照する HATEOAS リンクを含む応答を返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-144">In this example, an HTTP GET request that finds the details for a customer returns a response that include HATEOAS links that reference the orders for that customer:</span></span>

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

<span data-ttu-id="e86f5-145">この例では、顧客データは、次のコード スニペットに示す `Customer` クラスで表されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-145">In this example, the customer data is represented by the `Customer` class shown in the following code snippet.</span></span> <span data-ttu-id="e86f5-146">HATEOAS リンクは、 `Links` コレクション プロパティに保持されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-146">The HATEOAS links are held in the `Links` collection property:</span></span>

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

<span data-ttu-id="e86f5-147">HTTP GET 操作では、ストレージから顧客データを取得して `Customer` オブジェクトを作成し、`Links` コレクションを設定します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-147">The HTTP GET operation retrieves the customer data from storage and constructs a `Customer` object, and then populates the `Links` collection.</span></span> <span data-ttu-id="e86f5-148">結果は、JSON 応答メッセージとして書式設定されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-148">The result is formatted as a JSON response message.</span></span> <span data-ttu-id="e86f5-149">各リンクは次のフィールドで構成されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-149">Each link comprises the following fields:</span></span>

* <span data-ttu-id="e86f5-150">返されるオブジェクトとリンクで示されているオブジェクトの関係。</span><span class="sxs-lookup"><span data-stu-id="e86f5-150">The relationship between the object being returned and the object described by the link.</span></span> <span data-ttu-id="e86f5-151">この例の "self" は、リンクがそのオブジェクト自体の参照であることを示しています (多くのオブジェクト指向言語での `this` ポインターと同様)。また、"orders" は関連する注文情報が含まれたコレクションの名前です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-151">In this case "self" indicates that the link is a reference back to the object itself (similar to a `this` pointer in many object-oriented languages), and "orders" is the name of a collection containing the related order information.</span></span>
* <span data-ttu-id="e86f5-152">リンクで示されているオブジェクトの URI 形式のハイパーリンク (`Href`)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-152">The hyperlink (`Href`) for the object being described by the link in the form of a URI.</span></span>
* <span data-ttu-id="e86f5-153">この URI に送信できる HTTP 要求の種類 (`Action`)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-153">The type of HTTP request (`Action`) that can be sent to this URI.</span></span>
* <span data-ttu-id="e86f5-154">要求の種類に応じて、HTTP 要求で提供するか、または応答で返すことができるデータの形式 (`Types`)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-154">The format of any data (`Types`) that should be provided in the HTTP request or that can be returned in the response, depending on the type of the request.</span></span>

<span data-ttu-id="e86f5-155">HTTP 応答の例に示す HATEOAS リンクは、クライアント アプリケーションが次の操作を実行できることを示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-155">The HATEOAS links shown in the example HTTP response indicate that a client application can perform the following operations:</span></span>

* <span data-ttu-id="e86f5-156">顧客の詳細を (再度) 取得するための URI `https://adventure-works.com/customers/2` への HTTP GET 要求。</span><span class="sxs-lookup"><span data-stu-id="e86f5-156">An HTTP GET request to the URI `https://adventure-works.com/customers/2` to fetch the details of the customer (again).</span></span> <span data-ttu-id="e86f5-157">データは、XML または JSON として返すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-157">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="e86f5-158">顧客の詳細を変更するための URI `https://adventure-works.com/customers/2` への HTTP PUT 要求。</span><span class="sxs-lookup"><span data-stu-id="e86f5-158">An HTTP PUT request to the URI `https://adventure-works.com/customers/2` to modify the details of the customer.</span></span> <span data-ttu-id="e86f5-159">新しいデータは、要求メッセージで x-www-form-urlencoded 形式で提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-159">The new data must be provided in the request message in x-www-form-urlencoded format.</span></span>
* <span data-ttu-id="e86f5-160">顧客を削除するための URI `https://adventure-works.com/customers/2` への HTTP DELETE 要求。</span><span class="sxs-lookup"><span data-stu-id="e86f5-160">An HTTP DELETE request to the URI `https://adventure-works.com/customers/2` to delete the customer.</span></span> <span data-ttu-id="e86f5-161">この要求は、追加情報を求めることも、応答メッセージの本文でデータを返すこともありません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-161">The request does not expect any additional information or return data in the response message body.</span></span>
* <span data-ttu-id="e86f5-162">顧客のすべての注文を検索するために URI `https://adventure-works.com/customers/2/orders` に HTTP GET 要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-162">An HTTP GET request to the URI `https://adventure-works.com/customers/2/orders` to find all the orders for the customer.</span></span> <span data-ttu-id="e86f5-163">データは、XML または JSON として返すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-163">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="e86f5-164">この顧客の新規の注文を作成するために URI `https://adventure-works.com/customers/2/orders` に HTTP PUT 要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-164">An HTTP PUT request to the URI `https://adventure-works.com/customers/2/orders` to create a new order for this customer.</span></span> <span data-ttu-id="e86f5-165">データは、要求メッセージで x-www-form-urlencoded 形式で提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-165">The data must be provided in the request message in x-www-form-urlencoded format.</span></span>

## <a name="handling-exceptions"></a><span data-ttu-id="e86f5-166">例外の処理</span><span class="sxs-lookup"><span data-stu-id="e86f5-166">Handling exceptions</span></span>

<span data-ttu-id="e86f5-167">操作によりキャッチされない例外がスローされた場合は、次の点を検討してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-167">Consider the following points if an operation throws an uncaught exception.</span></span>

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a><span data-ttu-id="e86f5-168">例外をキャプチャし、クライアントに意味のある応答を返す</span><span class="sxs-lookup"><span data-stu-id="e86f5-168">Capture exceptions and return a meaningful response to clients</span></span>

<span data-ttu-id="e86f5-169">HTTP 操作を実装するコードでは、キャッチされない例外をフレームワークに伝達するのではなく、包括的な例外処理を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-169">The code that implements an HTTP operation should provide comprehensive exception handling rather than letting uncaught exceptions propagate to the framework.</span></span> <span data-ttu-id="e86f5-170">例外によって操作を正常に完了することができない場合、その例外を応答メッセージで返すことができますが、例外の原因となったエラーのわかりやすい説明が含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-170">If an exception makes it impossible to complete the operation successfully, the exception can be passed back in the response message, but it should include a meaningful description of the error that caused the exception.</span></span> <span data-ttu-id="e86f5-171">また、あらゆる状況で状態コード 500 を単に返すのではなく、例外に適切な HTTP 状態コードが含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-171">The exception should also include the appropriate HTTP status code rather than simply returning status code 500 for every situation.</span></span> <span data-ttu-id="e86f5-172">たとえば、ユーザー要求によって制約に違反するデータベースの更新 (未処理の注文がある顧客の削除など) が行われる場合は、状態コード 409 (Conflict) と、競合の原因を示すメッセージ本文を返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-172">For example, if a user request causes a database update that violates a constraint (such as attempting to delete a customer that has outstanding orders), you should return status code 409 (Conflict) and a message body indicating the reason for the conflict.</span></span> <span data-ttu-id="e86f5-173">他の条件によって要求を達成できない状態になった場合は、状態コード 400 (Bad Request) を返すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-173">If some other condition renders the request unachievable, you can return status code 400 (Bad Request).</span></span> <span data-ttu-id="e86f5-174">HTTP 状態コードの全一覧については、W3C の Web サイトの「 [Status Code Definitions (状態コードの定義)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) 」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-174">You can find a full list of HTTP status codes on the [Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) page on the W3C website.</span></span>

<span data-ttu-id="e86f5-175">次のコードは、さまざまな条件をトラップし、適切な応答を返す例を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-175">The code example traps different conditions and returns an appropriate response.</span></span>

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
> <span data-ttu-id="e86f5-176">API に侵入しようとする攻撃者に悪用される危険性のある情報を含めないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-176">Do not include information that could be useful to an attacker attempting to penetrate your API.</span></span>
  
<span data-ttu-id="e86f5-177">多くの Web サーバーは、Web API に到達する前にエラー条件自体をトラップします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-177">Many web servers trap error conditions themselves before they reach the web API.</span></span> <span data-ttu-id="e86f5-178">たとえば、Web サイトの認証を構成した場合、ユーザーが適切な認証情報を提供できないと、Web サーバーは状態コード 401 (Unauthorized) で応答します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-178">For example, if you configure authentication for a web site and the user fails to provide the correct authentication information, the web server should respond with status code 401 (Unauthorized).</span></span> <span data-ttu-id="e86f5-179">クライアントが認証されたら、クライアントが要求されたリソースにアクセスできることを確認するために、コードで独自のチェックを実行できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-179">Once a client has been authenticated, your code can perform its own checks to verify that the client should be able access the requested resource.</span></span> <span data-ttu-id="e86f5-180">この認証に失敗した場合は、状態コード 403 (Forbidden) を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-180">If this authorization fails, you should return status code 403 (Forbidden).</span></span>
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a><span data-ttu-id="e86f5-181">一貫した方法で例外を処理し、エラーに関する情報をログに記録する</span><span class="sxs-lookup"><span data-stu-id="e86f5-181">Handle exceptions consistently and log information about errors</span></span>

<span data-ttu-id="e86f5-182">一貫した方法で例外を処理するには、Web API 全体にわたるグローバルなエラー処理方法を実装することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-182">To handle exceptions in a consistent manner, consider implementing a global error handling strategy across the entire web API.</span></span> <span data-ttu-id="e86f5-183">また、各例外の全詳細をキャプチャするエラー ログも組み込む必要があります。Web 経由でクライアントにアクセスできないようにしているのであれば、このエラー ログに詳細情報を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-183">You should also incorporate error logging which captures the full details of each exception; this error log can contain detailed information as long as it is not made accessible over the web to clients.</span></span> 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a><span data-ttu-id="e86f5-184">クライアント側のエラーとサーバー側のエラーを区別する</span><span class="sxs-lookup"><span data-stu-id="e86f5-184">Distinguish between client-side errors and server-side errors</span></span>

<span data-ttu-id="e86f5-185">HTTP プロトコルでは、クライアント アプリケーションが原因で発生したエラー (HTTP 4xx 状態コード) と、サーバーでの障害が原因で発生したエラー (HTTP 5xx 状態コード) を区別します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-185">The HTTP protocol distinguishes between errors that occur due to the client application (the HTTP 4xx status codes), and errors that are caused by a mishap on the server (the HTTP 5xx status codes).</span></span> <span data-ttu-id="e86f5-186">すべてのエラー応答メッセージでこの規約を遵守していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-186">Make sure that you respect this convention in any error response messages.</span></span>

## <a name="optimizing-client-side-data-access"></a><span data-ttu-id="e86f5-187">クライアント側のデータ アクセスの最適化</span><span class="sxs-lookup"><span data-stu-id="e86f5-187">Optimizing client-side data access</span></span>
<span data-ttu-id="e86f5-188">Web サーバーとクライアント アプリケーションが関与する環境など、分散環境で懸念の主な原因の 1 つとなっているのがネットワークです。</span><span class="sxs-lookup"><span data-stu-id="e86f5-188">In a distributed environment such as that involving a web server and client applications, one of the primary sources of concern is the network.</span></span> <span data-ttu-id="e86f5-189">特に、クライアント アプリケーションが要求の送信やデータの受信を頻繁に行う場合、これがかなりのボトルネックになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-189">This can act as a considerable bottleneck, especially if a client application is frequently sending requests or receiving data.</span></span> <span data-ttu-id="e86f5-190">そのため、ネットワーク上を流れるトラフィックの量を最小限に抑えることを目標にします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-190">Therefore you should aim to minimize the amount of traffic that flows across the network.</span></span> <span data-ttu-id="e86f5-191">データを取得して保持するコードを実装するときには、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-191">Consider the following points when you implement the code to retrieve and maintain data:</span></span>

### <a name="support-client-side-caching"></a><span data-ttu-id="e86f5-192">クライアント側のキャッシュをサポートする</span><span class="sxs-lookup"><span data-stu-id="e86f5-192">Support client-side caching</span></span>

<span data-ttu-id="e86f5-193">HTTP 1.1 プロトコルでは、Cache-Control ヘッダーを使用することで、クライアントと要求のルーティングで経由する中間サーバーでのキャッシュをサポートしています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-193">The HTTP 1.1 protocol supports caching in clients and intermediate servers through which a request is routed by the use of the Cache-Control header.</span></span> <span data-ttu-id="e86f5-194">クライアント アプリケーションが Web API に HTTP GET 要求を送信したときは、クライアントまたは要求のルーティングで経由した中間サーバーで応答の本文のデータを安全にキャッシュできるかどうかと、データを期限切れにして古いデータと見なすまでの時間を示す Cache-Control ヘッダーを応答に含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-194">When a client application sends an HTTP GET request to the web API, the response can include a Cache-Control header that indicates whether the data in the body of the response can be safely cached by the client or an intermediate server through which the request has been routed, and for how long before it should expire and be considered out-of-date.</span></span> <span data-ttu-id="e86f5-195">次の例は、HTTP GET 要求と、Cache-Control ヘッダーが含まれた対応する応答を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-195">The following example shows an HTTP GET request and the corresponding response that includes a Cache-Control header:</span></span>

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

<span data-ttu-id="e86f5-196">この例では、Cache-Control ヘッダーは、返されるデータを 600 秒後に期限切れにすることと、返されるデータは 1 つのクライアント専用であり、他のクライアントが使用する共有キャッシュには保存できないこと (*private*) を指定しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-196">In this example, the Cache-Control header specifies that the data returned should be expired after 600 seconds, and is only suitable for a single client and must not be stored in a shared cache used by other clients (it is *private*).</span></span> <span data-ttu-id="e86f5-197">データを共有キャッシュに保存できる場合は、Cache-Control ヘッダーで *private* ではなく *public* を指定できます。また、データをクライアントでキャッシュ**できない**場合は、*no-store* を指定できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-197">The Cache-Control header could specify *public* rather than *private* in which case the data can be stored in a shared cache, or it could specify *no-store* in which case the data must **not** be cached by the client.</span></span> <span data-ttu-id="e86f5-198">次のコード例は、応答メッセージの Cache-Control ヘッダーを作成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-198">The following code example shows how to construct a Cache-Control header in a response message:</span></span>

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

<span data-ttu-id="e86f5-199">このコードでは、`OkResultWithCaching` という名前のカスタム `IHttpActionResult` クラスを使用しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-199">This code makes use of a custom `IHttpActionResult` class named `OkResultWithCaching`.</span></span> <span data-ttu-id="e86f5-200">このクラスを使用して、コントローラーはキャッシュ ヘッダーの内容を設定できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-200">This class enables the controller to set the cache header contents:</span></span>

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
> <span data-ttu-id="e86f5-201">HTTP プロトコルでは、Cache-Control ヘッダーの *no-cache* ディレクティブも定義されています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-201">The HTTP protocol also defines the *no-cache* directive for the Cache-Control header.</span></span> <span data-ttu-id="e86f5-202">少し紛らわしいのですが、このディレクティブは "キャッシュしない" ことを意味するのではなく、"キャッシュされた情報を返す前に、サーバーでその情報を再検証する" ことを意味します。データは、引き続きキャッシュに保存できますが、使用するたびにチェックされ、最新であることが確認されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-202">Rather confusingly, this directive does not mean "do not cache" but rather "revalidate the cached information with the server before returning it"; the data can still be cached, but it is checked each time it is used to ensure that it is still current.</span></span>
>
>

<span data-ttu-id="e86f5-203">キャッシュ管理は、クライアント アプリケーションまたは中間サーバーが行いますが、キャッシュ管理を適切に実装すると、最近既に取得したデータを取得する必要性が排除されるので、帯域幅を節約し、パフォーマンスを向上させることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-203">Cache management is the responsibility of the client application or intermediate server, but if properly implemented it can save bandwidth and improve performance by removing the need to fetch data that has already been recently retrieved.</span></span>

<span data-ttu-id="e86f5-204">Cache-Control ヘッダーの *max-age* 値は 1 つの目安にすぎず、指定した期間に対応するデータが変更されないことを保証するわけではありません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-204">The *max-age* value in the Cache-Control header is only a guide and not a guarantee that the corresponding data won't change during the specified time.</span></span> <span data-ttu-id="e86f5-205">Web API では、データの予想される揮発性に応じて、max-age を適切な値に設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-205">The web API should set the max-age to a suitable value depending on the expected volatility of the data.</span></span> <span data-ttu-id="e86f5-206">この期間が経過したら、クライアントはキャッシュからオブジェクトを破棄する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-206">When this period expires, the client should discard the object from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="e86f5-207">最新の Web ブラウザーは、記述に従って適切な Cache-Control ヘッダーを要求に追加し、結果のヘッダーを調べることで、クライアント側のキャッシュをサポートします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-207">Most modern web browsers support client-side caching by adding the appropriate cache-control headers to requests and examining the headers of the results, as described.</span></span> <span data-ttu-id="e86f5-208">ただし、一部の古いブラウザーは、クエリ文字列を含む URL から返された値をキャッシュしません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-208">However, some older browsers will not cache the values returned from a URL that includes a query string.</span></span> <span data-ttu-id="e86f5-209">通常、これは、ここで説明したプロトコルに基づく独自のキャッシュ管理方法を実装しているカスタム クライアント アプリケーションの問題ではありません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-209">This is not usually an issue for custom client applications which implement their own cache management strategy based on the protocol discussed here.</span></span>
>
> <span data-ttu-id="e86f5-210">一部の古いプロキシも同じ動作を示し、クエリ文字列を含む URL に基づく要求をキャッシュしないことがあります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-210">Some older proxies exhibit the same behavior and might not cache requests based on URLs with query strings.</span></span> <span data-ttu-id="e86f5-211">これは、このようなプロキシ経由で Web サーバーに接続するカスタム クライアント アプリケーションの問題である場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-211">This could be an issue for custom client applications that connect to a web server through such a proxy.</span></span>
>

### <a name="provide-etags-to-optimize-query-processing"></a><span data-ttu-id="e86f5-212">ETag を提供してクエリ処理を最適化する</span><span class="sxs-lookup"><span data-stu-id="e86f5-212">Provide ETags to optimize query processing</span></span>

<span data-ttu-id="e86f5-213">クライアント アプリケーションがオブジェクトを取得するときに、応答メッセージに *ETag* (エンティティ タグ) を含めることもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-213">When a client application retrieves an object, the response message can also include an *ETag* (Entity Tag).</span></span> <span data-ttu-id="e86f5-214">ETag は、リソースのバージョンを示す不透明な文字列です。リソースが変更されるたびに、Etag も変更されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-214">An ETag is an opaque string that indicates the version of a resource; each time a resource changes the Etag is also modified.</span></span> <span data-ttu-id="e86f5-215">この ETag は、クライアント アプリケーションでデータの一部としてキャッシュする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-215">This ETag should be cached as part of the data by the client application.</span></span> <span data-ttu-id="e86f5-216">次のコード例は、HTTP GET 要求への応答の一部として ETag を追加する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-216">The following code example shows how to add an ETag as part of the response to an HTTP GET request.</span></span> <span data-ttu-id="e86f5-217">このコードでは、オブジェクトの `GetHashCode` メソッドを使用して、オブジェクトを識別する数値を生成します (必要に応じてこのメソッドをオーバーライドし、MD5 などのアルゴリズムを使用して独自のハッシュを生成できます)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-217">This code uses the `GetHashCode` method of an object to generate a numeric value that identifies the object (you can override this method if necessary and generate your own hash using an algorithm such as MD5) :</span></span>

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

<span data-ttu-id="e86f5-218">Web API によってポストされる応答メッセージは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-218">The response message posted by the web API looks like this:</span></span>

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
> <span data-ttu-id="e86f5-219">セキュリティ上の理由から、機密データや認証された (HTTPS) 接続経由で返されるデータのキャッシュは許可しないでください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-219">For security reasons, do not allow sensitive data or data returned over an authenticated (HTTPS) connection to be cached.</span></span>
>
>

<span data-ttu-id="e86f5-220">クライアント アプリケーションは、次の GET 要求を発行して同じリソースをいつでも取得できます。リソースが変更されている場合 (別の ETag が含まれている場合) は、キャッシュされたバージョンを破棄し、新しいバージョンをキャッシュに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-220">A client application can issue a subsequent GET request to retrieve the same resource at any time, and if the resource has changed (it has a different ETag) the cached version should be discarded and the new version added to the cache.</span></span> <span data-ttu-id="e86f5-221">リソースのサイズが大きく、クライアントに送信する際に大量の帯域幅を必要とする場合、同じデータを取得する要求を繰り返すと、非効率的になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-221">If a resource is large and requires a significant amount of bandwidth to transmit back to the client, repeated requests to fetch the same data can become inefficient.</span></span> <span data-ttu-id="e86f5-222">これを解決するために、HTTP プロトコルでは、Web API でサポートする必要がある、GET 要求を最適化するための次のプロセスが定義されています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-222">To combat this, the HTTP protocol defines the following process for optimizing GET requests that you should support in a web API:</span></span>

* <span data-ttu-id="e86f5-223">クライアントは、If-None-Match HTTP ヘッダーで参照されるリソースの現在キャッシュされているバージョンの ETag を含む GET 要求を作成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-223">The client constructs a GET request containing the ETag for the currently cached version of the resource referenced in an If-None-Match HTTP header:</span></span>

    ```HTTP
    GET https://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* <span data-ttu-id="e86f5-224">Web API の GET 操作で、要求されたデータ (前の例では order 2) の現在の ETag を取得し、If-None-Match ヘッダーの値と比較します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-224">The GET operation in the web API obtains the current ETag for the requested data (order 2 in the above example), and compares it to the value in the If-None-Match header.</span></span>
* <span data-ttu-id="e86f5-225">要求されたデータの現在の ETag が要求で提供された ETag と一致した場合、リソースは変更されていないので、Web API は空のメッセージ本文と状態コード 304 (Not Modified) を含む HTTP 応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-225">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should return an HTTP response with an empty message body and a status code of 304 (Not Modified).</span></span>
* <span data-ttu-id="e86f5-226">要求されたデータの現在の ETag が要求で提供された ETag と一致しない場合は、データが変更されているので、Web API は新しいデータを含めたメッセージ本文と状態コード 200 (OK) を含む HTTP 応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-226">If the current ETag for the requested data does not match the ETag provided by the request, then the data has changed and the web API should return an HTTP response with the new data in the message body and a status code of 200 (OK).</span></span>
* <span data-ttu-id="e86f5-227">要求されたデータが既に存在しない場合、Web API は、状態コード 404 (Not Found) を含む HTTP 応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-227">If the requested data no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="e86f5-228">クライアントは、状態コードを使用してキャッシュを管理します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-228">The client uses the status code to maintain the cache.</span></span> <span data-ttu-id="e86f5-229">データが変更されていない場合 (状態コード 304)、オブジェクトはキャッシュされた状態を維持することができ、クライアント アプリケーションはオブジェクトのこのバージョンを引き続き使用します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-229">If the data has not changed (status code 304) then the object can remain cached and the client application should continue to use this version of the object.</span></span> <span data-ttu-id="e86f5-230">データが変更されている場合 (状態コード 200) は、キャッシュされたオブジェクトを破棄し、新しいオブジェクトを挿入します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-230">If the data has changed (status code 200) then the cached object should be discarded and the new one inserted.</span></span> <span data-ttu-id="e86f5-231">データが使用できなくなっている場合 (状態コード 404) は、オブジェクトをキャッシュから削除します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-231">If the data is no longer available (status code 404) then the object should be removed from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="e86f5-232">応答ヘッダーに Cache-Control ヘッダーの no-store が含まれている場合は、HTTP 状態コードに関係なく、オブジェクトをキャッシュから常に削除する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-232">If the response header contains the Cache-Control header no-store then the object should always be removed from the cache regardless of the HTTP status code.</span></span>
>

<span data-ttu-id="e86f5-233">次のコードは、If-None-Match ヘッダーをサポートするように拡張された `FindOrderByID` メソッドを示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-233">The code below shows the `FindOrderByID` method extended to support the If-None-Match header.</span></span> <span data-ttu-id="e86f5-234">If-None-Match ヘッダーを省略すると、指定した注文が常に取得されることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-234">Notice that if the If-None-Match header is omitted, the specified order is always retrieved:</span></span>

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

<span data-ttu-id="e86f5-235">次の例では、`EmptyResultWithCaching` という名前の追加のカスタム `IHttpActionResult` クラスが組み込まれています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-235">This example incorporates an additional custom `IHttpActionResult` class named `EmptyResultWithCaching`.</span></span> <span data-ttu-id="e86f5-236">このクラスは、応答の本文を含まない、 `HttpResponseMessage` オブジェクトのラッパーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-236">This class simply acts as a wrapper around an `HttpResponseMessage` object that does not contain a response body:</span></span>

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
> <span data-ttu-id="e86f5-237">この例では、基になるデータ ソースから取得したデータをハッシュすることによって、データの ETag が生成されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-237">In this example, the ETag for the data is generated by hashing the data retrieved from the underlying data source.</span></span> <span data-ttu-id="e86f5-238">他の方法で ETag を計算できる場合は、このプロセスをさらに最適化できます。データが変更されている場合、データ ソースから取得する必要があるのはそのデータだけになります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-238">If the ETag can be computed in some other way, then the process can be optimized further and the data only needs to be fetched from the data source if it has changed.</span></span>  <span data-ttu-id="e86f5-239">この方法は、データのサイズが大きい場合や、データ ソースにアクセスするとかなりの待機時間が発生する可能性がある場合 (データ ソースがリモート データベースの場合など) に特に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-239">This approach is especially useful if the data is large or accessing the data source can result in significant latency (for example, if the data source is a remote database).</span></span>
>

### <a name="use-etags-to-support-optimistic-concurrency"></a><span data-ttu-id="e86f5-240">Etag を使用してオプティミスティック同時実行制御をサポートする</span><span class="sxs-lookup"><span data-stu-id="e86f5-240">Use ETags to Support Optimistic Concurrency</span></span>

<span data-ttu-id="e86f5-241">以前にキャッシュされたデータに対する更新を可能にするために、HTTP プロトコルでは、オプティミスティック同時実行制御をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-241">To enable updates over previously cached data, the HTTP protocol supports an optimistic concurrency strategy.</span></span> <span data-ttu-id="e86f5-242">リソースの取得またはキャッシュ後に、クライアント アプリケーションがリソースを変更または削除する PUT 要求または DELETE 要求を送信する場合、ETag を参照する If-Match ヘッダーを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-242">If, after fetching and caching a resource, the client application subsequently sends a PUT or DELETE request to change or remove the resource, it should include in If-Match header that references the ETag.</span></span> <span data-ttu-id="e86f5-243">次のように、Web API はこの情報を使用して、リソースが取得された後に別のユーザーによって既に変更されているかどうかを確認し、クライアント アプリケーションに適切な応答を送信できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-243">The web API can then use this information to determine whether the resource has already been changed by another user since it was retrieved and send an appropriate response back to the client application as follows:</span></span>

* <span data-ttu-id="e86f5-244">クライアントは、リソースの新しい詳細と、If-Match HTTP ヘッダーで参照されるリソースの現在キャッシュされているバージョンの ETag を含む PUT 要求を作成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-244">The client constructs a PUT request containing the new details for the resource and the ETag for the currently cached version of the resource referenced in an If-Match HTTP header.</span></span> <span data-ttu-id="e86f5-245">次の例は、注文を更新する PUT 要求を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-245">The following example shows a PUT request that updates an order:</span></span>

    ```HTTP
    PUT https://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* <span data-ttu-id="e86f5-246">Web API の PUT 操作で、要求されたデータ (前の例では order 1) の現在の ETag を取得し、If-Match ヘッダーの値と比較します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-246">The PUT operation in the web API obtains the current ETag for the requested data (order 1 in the above example), and compares it to the value in the If-Match header.</span></span>
* <span data-ttu-id="e86f5-247">要求されたデータの現在の ETag が要求で提供された ETag と一致した場合、リソースは変更されていないので、Web API は更新を実行します。更新が正常に完了したら、HTTP 状態コード 204 (No Content) を含むメッセージを返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-247">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should perform the update, returning a message with HTTP status code 204 (No Content) if it is successful.</span></span> <span data-ttu-id="e86f5-248">応答には、Cache-Control ヘッダーとリソースの更新バージョンの ETag ヘッダーを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-248">The response can include Cache-Control and ETag headers for the updated version of the resource.</span></span> <span data-ttu-id="e86f5-249">応答には、新しく更新されたリソースの URI を参照する Location ヘッダーを必ず含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-249">The response should always include the Location header that references the URI of the newly updated resource.</span></span>
* <span data-ttu-id="e86f5-250">要求されたデータの現在の ETag が要求で提供された ETag と一致しない場合は、データを取得した後に別のユーザーによってデータが変更されているので、Web API は空のメッセージ本文と状態コード 412 (Precondition Failed) を含む HTTP 応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-250">If the current ETag for the requested data does not match the ETag provided by the request, then the data has been changed by another user since it was fetched and the web API should return an HTTP response with an empty message body and a status code of 412 (Precondition Failed).</span></span>
* <span data-ttu-id="e86f5-251">更新対象のリソースが既に存在しない場合、Web API は状態コード 404 (Not Found) を含む HTTP 応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-251">If the resource to be updated no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="e86f5-252">クライアントは、状態コードと応答ヘッダーを使用してキャッシュを管理します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-252">The client uses the status code and response headers to maintain the cache.</span></span> <span data-ttu-id="e86f5-253">データが更新された場合 (状態コード 204)、オブジェクトはキャッシュされた状態を維持できますが (Cache-Control ヘッダーで no-store が指定されていない場合)、ETag を更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-253">If the data has been updated (status code 204) then the object can remain cached (as long as the Cache-Control header does not specify no-store) but the ETag should be updated.</span></span> <span data-ttu-id="e86f5-254">データが別のユーザーによって変更されている場合 (状態コード 412)、またはデータが見つからない場合 (状態コード 404) は、キャッシュされたオブジェクトを破棄する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-254">If the data was changed by another user changed (status code 412) or not found (status code 404) then the cached object should be discarded.</span></span>

<span data-ttu-id="e86f5-255">次のコード例は、Orders コントローラーの PUT 操作の実装を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-255">The next code example shows an implementation of the PUT operation for the Orders controller:</span></span>

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
> <span data-ttu-id="e86f5-256">If-Match ヘッダーの使用は完全なオプションです。このヘッダーを省略すると、Web API は指定した注文を常に更新しようとするので、別のユーザーが行った更新が無条件に上書きされる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-256">Use of the If-Match header is entirely optional, and if it is omitted the web API will always attempt to update the specified order, possibly blindly overwriting an update made by another user.</span></span> <span data-ttu-id="e86f5-257">更新データの損失に起因する問題を回避するには、常に If-Match ヘッダーを指定します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-257">To avoid problems due to lost updates, always provide an If-Match header.</span></span>
>
>

## <a name="handling-large-requests-and-responses"></a><span data-ttu-id="e86f5-258">サイズの大きい要求と応答の処理</span><span class="sxs-lookup"><span data-stu-id="e86f5-258">Handling large requests and responses</span></span>
<span data-ttu-id="e86f5-259">クライアント アプリケーションは、サイズが数メガバイト (以上) になる可能性のあるデータを送受信する要求を発行することが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-259">There may be occasions when a client application needs to issue requests that send or receive data that may be several megabytes (or bigger) in size.</span></span> <span data-ttu-id="e86f5-260">この量のデータが送信されるまで待機すると、クライアント アプリケーションが応答しなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-260">Waiting while this amount of data is transmitted could cause the client application to become unresponsive.</span></span> <span data-ttu-id="e86f5-261">大量のデータを含む要求を処理する必要がある場合は、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-261">Consider the following points when you need to handle requests that include significant amounts of data:</span></span>

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a><span data-ttu-id="e86f5-262">ラージ オブジェクトを伴う要求と応答を最適化する</span><span class="sxs-lookup"><span data-stu-id="e86f5-262">Optimize requests and responses that involve large objects</span></span>

<span data-ttu-id="e86f5-263">グラフィック イメージや他の種類のバイナリ データなど、リソースがラージ オブジェクトであったり、大規模なフィールドが含まれていたりする場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-263">Some resources may be large objects or include large fields, such as graphics images or other types of binary data.</span></span> <span data-ttu-id="e86f5-264">Web API は、このようなリソースのアップロードとダウンロードを最適化できるように、ストリーミングをサポートする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-264">A web API should support streaming to enable optimized uploading and downloading of these resources.</span></span>

<span data-ttu-id="e86f5-265">HTTP プロトコルは、ラージ データ オブジェクトをクライアントにストリーミングするためのチャンク転送エンコード メカニズムを提供します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-265">The HTTP protocol provides the chunked transfer encoding mechanism to stream large data objects back to a client.</span></span> <span data-ttu-id="e86f5-266">クライアントがラージ オブジェクトの HTTP GET 要求を送信したときに、Web API は HTTP 接続を介して段階的な "*チャンク*" で応答を送信できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-266">When the client sends an HTTP GET request for a large object, the web API can send the reply back in piecemeal *chunks* over an HTTP connection.</span></span> <span data-ttu-id="e86f5-267">応答のデータの長さが最初はわからない場合があるので (データが生成される場合があります)、Web API をホストするサーバーは、Content-Length ヘッダーではなく Transfer-Encoding: Chunked ヘッダーを指定した各チャンクを含む応答メッセージを送信する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-267">The length of the data in the reply may not be known initially (it might be generated), so the server hosting the web API should send a response message with each chunk that specifies the Transfer-Encoding: Chunked header rather than a Content-Length header.</span></span> <span data-ttu-id="e86f5-268">クライアント アプリケーションは、各チャンクを順に受信して完全な応答を作成できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-268">The client application can receive each chunk in turn to build up the complete response.</span></span> <span data-ttu-id="e86f5-269">サーバーが、サイズがゼロの最後のチャンクを送信すると、データ転送が完了します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-269">The data transfer completes when the server sends back a final chunk with zero size.</span></span> 

<span data-ttu-id="e86f5-270">1 つの要求により、大量のリソースを消費する大規模なオブジェクトが生成される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-270">A single request could conceivably result in a massive object that consumes considerable resources.</span></span> <span data-ttu-id="e86f5-271">Web API は、ストリーミング プロセスの途中で要求のデータ量が許容範囲を超えたことを確認すると、操作を中止し、状態コード 413 (Request Entity Too Large) を含む応答メッセージを返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-271">If, during the streaming process, the web API determines that the amount of data in a request has exceeded some acceptable bounds, it can abort the operation and return a response message with status code 413 (Request Entity Too Large).</span></span>

<span data-ttu-id="e86f5-272">HTTP 圧縮を使用すると、ネットワーク経由で送信されるラージ オブジェクトのサイズを最小限に抑えることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-272">You can minimize the size of large objects transmitted over the network by using HTTP compression.</span></span> <span data-ttu-id="e86f5-273">この方法では、ネットワーク トラフィックの量と、関連するネットワーク待ち時間を削減できますが、クライアントと Web API をホストするサーバーで追加の処理が必要となります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-273">This approach helps to reduce the amount of network traffic and the associated network latency, but at the cost of requiring additional processing at the client and the server hosting the web API.</span></span> <span data-ttu-id="e86f5-274">たとえば、圧縮データを受信することを求めるクライアント アプリケーションは、Accept-Encoding: gzip 要求ヘッダーを含めることができます (他のデータ圧縮アルゴリズムを指定することもできます)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-274">For example, a client application that expects to receive compressed data can include an Accept-Encoding: gzip request header (other data compression algorithms can also be specified).</span></span> <span data-ttu-id="e86f5-275">サーバーが圧縮をサポートしている場合、メッセージ本文に gzip 形式で保持されたコンテンツと、Content-Encoding: gzip 応答ヘッダーを使用して応答する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-275">If the server supports compression it should respond with the content held in gzip format in the message body and the Content-Encoding: gzip response header.</span></span>

<span data-ttu-id="e86f5-276">エンコードされた圧縮をストリーミングと組み合わせることができます。ストリーミングの前に、まずデータを圧縮し、メッセージのヘッダーに gzip コンテンツ エンコードとチャンク転送エンコードを指定します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-276">You can combine encoded compression with streaming; compress the data first before streaming it, and specify the gzip content encoding and chunked transfer encoding in the message headers.</span></span> <span data-ttu-id="e86f5-277">また、一部の Web サーバー (Internet Information Server など) は、Web API でデータを圧縮するかどうかに関係なく、HTTP 応答を自動的に圧縮するように構成できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-277">Also note that some web servers (such as Internet Information Server) can be configured to automatically compress HTTP responses regardless of whether the web API compresses the data or not.</span></span>

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a><span data-ttu-id="e86f5-278">非同期操作をサポートしていないクライアントでは部分的な応答を実装する</span><span class="sxs-lookup"><span data-stu-id="e86f5-278">Implement partial responses for clients that do not support asynchronous operations</span></span>

<span data-ttu-id="e86f5-279">非同期ストリーミングの代わりに、クライアント アプリケーションは、ラージ オブジェクトのデータをチャンク単位で明示的に要求できます。これは部分的な応答と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-279">As an alternative to asynchronous streaming, a client application can explicitly request data for large objects in chunks, known as partial responses.</span></span> <span data-ttu-id="e86f5-280">クライアント アプリケーションは、HTTP HEAD 要求を送信して、オブジェクトに関する情報を取得します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-280">The client application sends an HTTP HEAD request to obtain information about the object.</span></span> <span data-ttu-id="e86f5-281">Web API が部分的な応答をサポートしている場合は、Accept-Ranges ヘッダーとオブジェクトの合計サイズを示す Content-Length ヘッダーを含み、メッセージの本文を空にした応答メッセージで HEAD 要求に応答する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-281">If the web API supports partial responses if should respond to the HEAD request with a response message that contains an Accept-Ranges header and a Content-Length header that indicates the total size of the object, but the body of the message should be empty.</span></span> <span data-ttu-id="e86f5-282">クライアント アプリケーションは、この情報を使用して、受信するバイト範囲を指定した一連の GET 要求を作成できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-282">The client application can use this information to construct a series of GET requests that specify a range of bytes to receive.</span></span> <span data-ttu-id="e86f5-283">Web API は、HTTP 状態コード 206 (Partial Content)、応答メッセージの本文に含まれるデータの実際の量を指定した Content-Length ヘッダー、このデータが表すオブジェクトの部分 (4000 ～ 8000 バイトなど) を示す Content-Range ヘッダーを含む応答メッセージを返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-283">The web API should return a response message with HTTP status 206 (Partial Content), a Content-Length header that specifies the actual amount of data included in the body of the response message, and a Content-Range header that indicates which part (such as bytes 4000 to 8000) of the object this data represents.</span></span>

<span data-ttu-id="e86f5-284">HTTP HEAD 要求と部分的な応答の詳細については、[API 設計][api-design]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-284">HTTP HEAD requests and partial responses are described in more detail in [API Design][api-design].</span></span>

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a><span data-ttu-id="e86f5-285">クライアント アプリケーションで不要な 100-Continue 状態メッセージを送信しない</span><span class="sxs-lookup"><span data-stu-id="e86f5-285">Avoid sending unnecessary 100-Continue status messages in client applications</span></span>

<span data-ttu-id="e86f5-286">大量のデータをサーバーに送信しようとしているクライアント アプリケーションは、サーバーが実際にその要求を受け入れる用意があるかどうかを最初に確認できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-286">A client application that is about to send a large amount of data to a server may determine first whether the server is actually willing to accept the request.</span></span> <span data-ttu-id="e86f5-287">クライアント アプリケーションは、データを送信する前に、Expect: 100-Continue ヘッダー、データのサイズを示す Content-Length ヘッダー、空のメッセージ本文を含む HTTP 要求を送信できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-287">Prior to sending the data, the client application can submit an HTTP request with an Expect: 100-Continue header, a Content-Length header that indicates the size of the data, but an empty message body.</span></span> <span data-ttu-id="e86f5-288">サーバーが要求を処理する用意がある場合、HTTP 状態コード 100 (Continue) を指定したメッセージで応答する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-288">If the server is willing to handle the request, it should respond with a message that specifies the HTTP status 100 (Continue).</span></span> <span data-ttu-id="e86f5-289">クライアント アプリケーションは要求を続行し、メッセージ本文にデータが含まれた完全な要求を送信できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-289">The client application can then proceed and send the complete request including the data in the message body.</span></span>

<span data-ttu-id="e86f5-290">IIS を使用してサービスをホストしている場合は、Web アプリケーションに要求を渡す前に、HTTP.sys ドライバーによって Expect: 100-Continue ヘッダーが自動的に検出され、処理されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-290">If you are hosting a service by using IIS, the HTTP.sys driver automatically detects and handles Expect: 100-Continue headers before passing requests to your web application.</span></span> <span data-ttu-id="e86f5-291">つまり、アプリケーション コードでこれらのヘッダーを使用することはほとんどありません。不適切であったり、大きすぎると思われるメッセージは、IIS によって既にフィルター処理されていることを前提にすることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-291">This means that you are unlikely to see these headers in your application code, and you can assume that IIS has already filtered any messages that it deems to be unfit or too large.</span></span>

<span data-ttu-id="e86f5-292">.NET Framework を使用してクライアント アプリケーションを構築している場合は、すべての POST メッセージと PUT メッセージで、Expect: 100-Continue ヘッダーを既定で含むメッセージが最初に送信されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-292">If you are building client applications by using the .NET Framework, then all POST and PUT messages will first send messages with Expect: 100-Continue headers by default.</span></span> <span data-ttu-id="e86f5-293">サーバー側と同様に、このプロセスは .NET Framework によって透過的に処理されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-293">As with the server-side, the process is handled transparently by the .NET Framework.</span></span> <span data-ttu-id="e86f5-294">ただし、このプロセスでは、要求のサイズが小さい場合でも、各 POST 要求と PUT 要求でサーバーへのラウンド トリップが 2 回発生することになります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-294">However, this process results in each POST and PUT request causing two round-trips to the server, even for small requests.</span></span> <span data-ttu-id="e86f5-295">アプリケーションが大量のデータを含む要求を送信しない場合は、クライアント アプリケーションで `ServicePointManager` クラスを使用して `ServicePoint` オブジェクトを作成することで、この機能を無効にすることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-295">If your application is not sending requests with large amounts of data, you can disable this feature by using the `ServicePointManager` class to create `ServicePoint` objects in the client application.</span></span> <span data-ttu-id="e86f5-296">`ServicePoint` オブジェクトは、サーバー上のリソースを特定する URI のスキームとホスト フラグメントに基づいて、クライアントが行うサーバーへの接続を処理します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-296">A `ServicePoint` object handles the connections that the client makes to a server based on the scheme and host fragments of URIs that identify resources on the server.</span></span> <span data-ttu-id="e86f5-297">その後、`ServicePoint` オブジェクトの `Expect100Continue` プロパティを false に設定できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-297">You can then set the `Expect100Continue` property of the `ServicePoint` object to false.</span></span> <span data-ttu-id="e86f5-298">`ServicePoint` オブジェクトのスキームおよびホスト フラグメントと一致する URI を使用してクライアントで行われる後続の POST 要求と PUT 要求は、すべて Expect: 100-Continue ヘッダーなしで送信されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-298">All subsequent POST and PUT requests made by the client through a URI that matches the scheme and host fragments of the `ServicePoint` object will be sent without Expect: 100-Continue headers.</span></span> <span data-ttu-id="e86f5-299">次のコードは、スキームが `http` で、ホストが `www.contoso.com` である URI に送信されるすべての要求を構成する `ServicePoint` オブジェクトを構成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-299">The following code shows how to configure a `ServicePoint` object that configures all requests sent to URIs with a scheme of `http` and a host of `www.contoso.com`.</span></span>

```csharp
Uri uri = new Uri("https://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

<span data-ttu-id="e86f5-300">`ServicePointManager` クラスの静的 `Expect100Continue` プロパティを設定して、今後作成されるすべての `ServicePoint` オブジェクトのこのプロパティの既定値を指定することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-300">You can also set the static `Expect100Continue` property of the `ServicePointManager` class to specify the default value of this property for all subsequently created `ServicePoint` objects.</span></span> <span data-ttu-id="e86f5-301">詳細については、[ServicePoint クラス](https://msdn.microsoft.com/library/system.net.servicepoint.aspx)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-301">For more information, see [ServicePoint Class](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).</span></span>

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a><span data-ttu-id="e86f5-302">多数のオブジェクトを返す可能性のある要求の改ページ調整をサポートする</span><span class="sxs-lookup"><span data-stu-id="e86f5-302">Support pagination for requests that may return large numbers of objects</span></span>

<span data-ttu-id="e86f5-303">コレクションに多数のリソースが含まれている場合、対応する URI に GET 要求を発行すると、Web API をホストするサーバーでパフォーマンスに影響する大量の処理が発生し、待機時間の増加をもたらす大量のネットワーク トラフィックが生成される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-303">If a collection contains a large number of resources, issuing a GET request to the corresponding URI could result in significant processing on the server hosting the web API affecting performance, and generate a significant amount of network traffic resulting in increased latency.</span></span>

<span data-ttu-id="e86f5-304">このような状況に対応するには、Web API でクエリ文字列をサポートする必要があります。クエリ文字列により、クライアント アプリケーションは、要求の調整や管理しやすい個別のブロック (またはページ) でのデータの取得が可能になります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-304">To handle these cases, the web API should support query strings that enable the client application to refine requests or fetch data in more manageable, discrete blocks (or pages).</span></span> <span data-ttu-id="e86f5-305">次のコードは、`Orders` コントローラーの `GetAllOrders` メソッドを示しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-305">The code below shows the `GetAllOrders` method in the `Orders` controller.</span></span> <span data-ttu-id="e86f5-306">このメソッドは注文の詳細を取得します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-306">This method retrieves the details of orders.</span></span> <span data-ttu-id="e86f5-307">このメソッドに制約がない場合、大量のデータが返される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-307">If this method was unconstrained, it could conceivably return a large amount of data.</span></span> <span data-ttu-id="e86f5-308">`limit` パラメーターと `offset` パラメーターは、データの量を減らして小さなサブセットにすることを目的としています。この例では、既定で最初 10 個の注文に制限されています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-308">The `limit` and `offset` parameters are intended to reduce the volume of data to a smaller subset, in this case only the first 10 orders by default:</span></span>

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

<span data-ttu-id="e86f5-309">クライアント アプリケーションは、URI `https://www.adventure-works.com/api/orders?limit=30&offset=50` を使用して、オフセット 50 で始まる 30 個の注文を取得する要求を発行できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-309">A client application can issue a request to retrieve 30 orders starting at offset 50 by using the URI `https://www.adventure-works.com/api/orders?limit=30&offset=50`.</span></span>

> [!TIP]
> <span data-ttu-id="e86f5-310">URI が 2000 文字を超える長さになるクエリ文字列をクライアント アプリケーションが指定できないようにします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-310">Avoid enabling client applications to specify query strings that result in a URI that is more than 2000 characters long.</span></span> <span data-ttu-id="e86f5-311">多くの Web クライアントとサーバーは、この長さの URI を処理できません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-311">Many web clients and servers cannot handle URIs that are this long.</span></span>
>
>

## <a name="maintaining-responsiveness-scalability-and-availability"></a><span data-ttu-id="e86f5-312">応答性、スケーラビリティ、可用性の維持</span><span class="sxs-lookup"><span data-stu-id="e86f5-312">Maintaining responsiveness, scalability, and availability</span></span>
<span data-ttu-id="e86f5-313">世界のあらゆる場所で実行される多数のクライアント アプリケーションが同じ Web API を利用する場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-313">The same web API might be utilized by many client applications running anywhere in the world.</span></span> <span data-ttu-id="e86f5-314">負荷が大きい状況で応答性を維持し、大きく変化するワークロードをサポートするスケーラビリティを備え、業務に不可欠な操作を実行するクライアントの可用性を保証するように Web API が実装されていることが重要です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-314">It is important to ensure that the web API is implemented to maintain responsiveness under a heavy load, to be scalable to support a highly varying workload, and to guarantee availability for clients that perform business-critical operations.</span></span> <span data-ttu-id="e86f5-315">これらの要件を満たす方法を決めるときには、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-315">Consider the following points when determining how to meet these requirements:</span></span>

### <a name="provide-asynchronous-support-for-long-running-requests"></a><span data-ttu-id="e86f5-316">実行時間の長い要求の非同期サポートを提供する</span><span class="sxs-lookup"><span data-stu-id="e86f5-316">Provide asynchronous support for long-running requests</span></span>

<span data-ttu-id="e86f5-317">処理に時間がかかると考えられる要求は、その要求を送信したクライアントをブロックせずに実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-317">A request that might take a long time to process should be performed without blocking the client that submitted the request.</span></span> <span data-ttu-id="e86f5-318">Web API では、要求を検証する初期チェックを実行しし、処理を実行する別のタスクを開始した後、HTTP コード 202 (Accepted) を含む応答メッセージを返すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-318">The web API can perform some initial checking to validate the request, initiate a separate task to perform the work, and then return a response message with HTTP code 202 (Accepted).</span></span> <span data-ttu-id="e86f5-319">タスクは、Web API 処理の一部として非同期的に実行できます。またはバック グラウンド タスクとしてオフロードできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-319">The task could run asynchronously as part of the web API processing, or it could be offloaded to a background task.</span></span>

<span data-ttu-id="e86f5-320">Web API では、処理の結果をクライアント アプリケーションに返すためのメカニズムも提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-320">The web API should also provide a mechanism to return the results of the processing to the client application.</span></span> <span data-ttu-id="e86f5-321">これを実現するには、クライアント アプリケーションが、処理が完了したかどうかを定期的に照会し、結果を取得するためのポーリング メカニズムを提供するか、Web API が操作の完了時に通知を送信できるようにします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-321">You can achieve this by providing a polling mechanism for client applications to periodically query whether the processing has finished and obtain the result, or enabling the web API to send a notification when the operation has completed.</span></span>

<span data-ttu-id="e86f5-322">次の方法を使用して仮想リソースとして機能する *polling* URI を提供することで、単純なポーリング メカニズムを実装できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-322">You can implement a simple polling mechanism by providing a *polling* URI that acts as a virtual resource using the following approach:</span></span>

1. <span data-ttu-id="e86f5-323">クライアント アプリケーションが Web API に最初の要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-323">The client application sends the initial request to the web API.</span></span>
2. <span data-ttu-id="e86f5-324">Web API は、テーブル ストレージまたは Microsoft Azure Cache に保持されるテーブルに要求に関する情報を保存し、このエントリの一意のキーを、場合によっては GUID の形式で生成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-324">The web API stores information about the request in a table held in table storage or Microsoft Azure Cache, and generates a unique key for this entry, possibly in the form of a GUID.</span></span>
3. <span data-ttu-id="e86f5-325">Web API は別のタスクとして処理を開始します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-325">The web API initiates the processing as a separate task.</span></span> <span data-ttu-id="e86f5-326">Web API は、タスクの状態を *Running* としてテーブルに記録します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-326">The web API records the state of the task in the table as *Running*.</span></span>
4. <span data-ttu-id="e86f5-327">Web API は、HTTP 状態コード 202 (Accepted) と、テーブル エントリの GUID をメッセージの本文として含む応答メッセージを返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-327">The web API returns a response message with HTTP status code 202 (Accepted), and the GUID of the table entry in the body of the message.</span></span>
5. <span data-ttu-id="e86f5-328">タスクが完了したら、Web API は結果をテーブルに保存し、タスクの状態を *Complete* に設定します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-328">When the task has completed, the web API stores the results in the table, and sets the state of the task to *Complete*.</span></span> <span data-ttu-id="e86f5-329">タスクが失敗した場合、Web API は失敗に関する情報を保存し、状態を *Failed* に設定することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-329">Note that if the task fails, the web API could also store information about the failure and set the status to *Failed*.</span></span>
6. <span data-ttu-id="e86f5-330">タスクの実行中、クライアントは独自の処理を引き続き実行できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-330">While the task is running, the client can continue performing its own processing.</span></span> <span data-ttu-id="e86f5-331">クライアントは、要求を URI */polling/{guid}* に定期的に送信できます。*{guid}* は、Web API から 202 応答メッセージで返される GUID です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-331">It can periodically send a request to the URI */polling/{guid}* where *{guid}* is the GUID returned in the 202 response message by the web API.</span></span>
7. <span data-ttu-id="e86f5-332">Web API は、*/polling/{guid}* URI でテーブル内の対応するタスクの状態を照会し、HTTP 状態コード 200 (OK) と、この状態 (*Running*、*Complete*、または *Failed*) を含む応答メッセージを返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-332">The web API at the */polling/{guid}* URI queries the state of the corresponding task in the table and returns a response message with HTTP status code 200 (OK) containing this state (*Running*, *Complete*, or *Failed*).</span></span> <span data-ttu-id="e86f5-333">タスクが完了または失敗した場合、処理の結果や失敗の原因に関する入手可能な情報を応答メッセージに含めることもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-333">If the task has completed or failed, the response message can also include the results of the processing or any information available about the reason for the failure.</span></span>

<span data-ttu-id="e86f5-334">通知を実装するためのオプションは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="e86f5-334">Options for implementing notifications include:</span></span>

- <span data-ttu-id="e86f5-335">Azure Notification Hubs を使用して、非同期応答をクライアント アプリケーションにプッシュする。</span><span class="sxs-lookup"><span data-stu-id="e86f5-335">Using an Azure Notification Hub to push asynchronous responses to client applications.</span></span> <span data-ttu-id="e86f5-336">詳細については、「[Azure Notification Hubs によるユーザーへの通知](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-336">For more information, see [Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).</span></span>
- <span data-ttu-id="e86f5-337">Comet モデルを使用して、クライアントと Web API をホストするサーバー間の永続的なネットワーク接続を保持し、この接続を使用してサーバーからのメッセージをクライアントにプッシュする。</span><span class="sxs-lookup"><span data-stu-id="e86f5-337">Using the Comet model to retain a persistent network connection between the client and the server hosting the web API, and using this connection to push messages from the server back to the client.</span></span> <span data-ttu-id="e86f5-338">ソリューションの例については、MSDN マガジンの記事「 [Microsoft .NET Framework でシンプルな Comet アプリケーションをビルドする](https://msdn.microsoft.com/magazine/jj891053.aspx) 」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-338">The MSDN magazine article [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) describes an example solution.</span></span>
- <span data-ttu-id="e86f5-339">SignalR を使用して、永続的なネットワーク接続経由で Web サーバーからクライアントにリアルタイムでデータをプッシュする。</span><span class="sxs-lookup"><span data-stu-id="e86f5-339">Using SignalR to push data in real-time from the web server to the client over a persistent network connection.</span></span> <span data-ttu-id="e86f5-340">SignalR は、NuGet パッケージとして ASP.NET Web アプリケーションで使用できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-340">SignalR is available for ASP.NET web applications as a NuGet package.</span></span> <span data-ttu-id="e86f5-341">詳細については、 [ASP.NET SignalR](https://www.asp.net/signalr) の Web サイトをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-341">You can find more information on the [ASP.NET SignalR](https://www.asp.net/signalr) website.</span></span>

### <a name="ensure-that-each-request-is-stateless"></a><span data-ttu-id="e86f5-342">各要求がステートレスであることを確認する</span><span class="sxs-lookup"><span data-stu-id="e86f5-342">Ensure that each request is stateless</span></span>

<span data-ttu-id="e86f5-343">各要求はアトミックと見なす必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-343">Each request should be considered atomic.</span></span> <span data-ttu-id="e86f5-344">クライアント アプリケーションによって行われるある要求と、同じクライアントによって送信される後続の要求との間に依存関係が存在しないようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-344">There should be no dependencies between one request made by a client application and any subsequent requests submitted by the same client.</span></span> <span data-ttu-id="e86f5-345">この方法はスケーラビリティを支援します。つまり、多数のサーバーで Web サービスのインスタンスをデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-345">This approach assists in scalability; instances of the web service can be deployed on a number of servers.</span></span> <span data-ttu-id="e86f5-346">これらのどのインスタンスでもクライアント要求を送信できるので、結果が常に同じである必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-346">Client requests can be directed at any of these instances and the results should always be the same.</span></span> <span data-ttu-id="e86f5-347">同様の理由から可用性も向上します。Web サーバーで障害が発生した場合、(Azure Traffic Manager を使用して) 要求を別のインスタンスにルーティングすることができ、クライアント アプリケーションに悪影響を与えずにサーバーが再起動されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-347">It also improves availability for a similar reason; if a web server fails requests can be routed to another instance (by using Azure Traffic Manager) while the server is restarted with no ill effects on client applications.</span></span>

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a><span data-ttu-id="e86f5-348">クライアントを追跡し、調整を実装して DOS 攻撃の危険性を低減する</span><span class="sxs-lookup"><span data-stu-id="e86f5-348">Track clients and implement throttling to reduce the chances of DOS attacks</span></span>

<span data-ttu-id="e86f5-349">特定のクライアントが一定の期間内に多数の要求を行うと、サービスが独占され、他のクライアントのパフォーマンスに影響を及ぼす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-349">If a specific client makes a large number of requests within a given period of time it might monopolize the service and affect the performance of other clients.</span></span> <span data-ttu-id="e86f5-350">この問題を軽減するには、Web API ですべての受信要求の IP アドレスを追跡するか、各認証済みアクセスをログに記録して、クライアント アプリケーションからの呼び出しを監視します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-350">To mitigate this issue, a web API can monitor calls from client applications either by tracking the IP address of all incoming requests or by logging each authenticated access.</span></span> <span data-ttu-id="e86f5-351">この情報を使用して、リソースへのアクセスを制限できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-351">You can use this information to limit resource access.</span></span> <span data-ttu-id="e86f5-352">クライアントが定義済みの上限を超えた場合、Web API は状態コード 503 (Service Unavailable) を含む応答メッセージを返し、クライアントが拒否されずに次の要求を送信できる時間を指定した Retry-After ヘッダーを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-352">If a client exceeds a defined limit, the web API can return a response message with status 503 (Service Unavailable) and include a Retry-After header that specifies when the client can send the next request without it being declined.</span></span> <span data-ttu-id="e86f5-353">この方法により、システムを停止させる、一連のクライアントからのサービス拒否 (DOS) 攻撃の可能性を低減できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-353">This strategy can help to reduce the chances of a Denial Of Service (DOS) attack from a set of clients stalling the system.</span></span>

### <a name="manage-persistent-http-connections-carefully"></a><span data-ttu-id="e86f5-354">永続的な HTTP 接続を慎重に管理する</span><span class="sxs-lookup"><span data-stu-id="e86f5-354">Manage persistent HTTP connections carefully</span></span>

<span data-ttu-id="e86f5-355">HTTP プロトコルは、永続的な HTTP 接続をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-355">The HTTP protocol supports persistent HTTP connections where they are available.</span></span> <span data-ttu-id="e86f5-356">HTTP 1.0 仕様で Connection:Keep-Alive ヘッダーが追加されました。このヘッダーにより、クライアント アプリケーションは、新しい接続を開くのではなく、同じ接続を使用して後続の要求を送信できることをサーバーに示すことができます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-356">The HTTP 1.0 specificiation added the Connection:Keep-Alive header that enables a client application to indicate to the server that it can use the same connection to send subsequent requests rather than opening new ones.</span></span> <span data-ttu-id="e86f5-357">ホストで定義されている期間内にクライアントが接続を再利用していない場合は、接続が自動的に閉じられます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-357">The connection closes automatically if the client does not reuse the connection within a period defined by the host.</span></span> <span data-ttu-id="e86f5-358">これは、Azure サービスで使用される HTTP 1.1 での既定の動作であるため、Keep-Alive ヘッダーをメッセージに含める必要はありません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-358">This behavior is the default in HTTP 1.1 as used by Azure services, so there is no need to include Keep-Alive headers in messages.</span></span>

<span data-ttu-id="e86f5-359">接続を開いたままにしておくと、待機時間とネットワークの輻輳が低減されるので、応答性を向上させることができますが、不要な接続を必要以上に開いたままにしておくと、他の同時実行クライアントによる接続が制限され、スケーラビリティに悪影響を及ぼす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-359">Keeping a connection open can help to improve responsiveness by reducing latency and network congestion, but it can be detrimental to scalability by keeping unnecessary connections open for longer than required, limiting the ability of other concurrent clients to connect.</span></span> <span data-ttu-id="e86f5-360">また、クライアント アプリケーションがモバイル デバイスで実行されている場合は、バッテリの寿命にも影響する可能性があります。アプリケーションがサーバーに要求することがあまりない場合、開いている接続を維持すると、バッテリ消費が早くなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-360">It can also affect battery life if the client application is running on a mobile device; if the application only makes occasional requests to the server, maintaining an open connection can cause the battery to drain more quickly.</span></span> <span data-ttu-id="e86f5-361">HTTP 1.1 で接続が永続化されないようにするには、クライアントが Connection:Close ヘッダーをメッセージに含め、既定の動作をオーバーライドします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-361">To ensure that a connection is not made persistent with HTTP 1.1, the client can include a Connection:Close header with messages to override the default behavior.</span></span> <span data-ttu-id="e86f5-362">同様に、サーバーが非常に多くのクライアントに対応している場合は、応答メッセージに Connection:Close ヘッダーを含めることで、接続を閉じ、サーバー リソースを節約します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-362">Similarly, if a server is handling a very large number of clients it can include a Connection:Close header in response messages which should close the connection and save server resources.</span></span>

> [!NOTE]
> <span data-ttu-id="e86f5-363">永続的な HTTP 接続は、通信チャネルを何度も確立することに関連して発生するネットワークのオーバーヘッドを削減するための完全なオプション機能です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-363">Persistent HTTP connections are a purely optional feature to reduce the network overhead associated with repeatedly establishing a communications channel.</span></span> <span data-ttu-id="e86f5-364">Web API とクライアント アプリケーションは、どちらも永続的な HTTP 接続が使用可能であることに依存しないようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-364">Neither the web API nor the client application should depend on a persistent HTTP connection being available.</span></span> <span data-ttu-id="e86f5-365">永続的な HTTP 接続を使用して、Comet スタイルの通知システムを実装しないでください。代わりに、TCP レイヤーで Socket (使用可能な場合は WebSocket) を利用します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-365">Do not use persistent HTTP connections to implement Comet-style notification systems; instead you should utilize sockets (or websockets if available) at the TCP layer.</span></span> <span data-ttu-id="e86f5-366">最後に、クライアント アプリケーションがプロキシ経由でサーバーと通信する場合、Keep-Alive ヘッダーの使用が制限されます。クライアントおよびプロキシとの接続だけが永続化されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-366">Finally, note Keep-Alive headers are of limited use if a client application communicates with a server via a proxy; only the connection with the client and the proxy will be persistent.</span></span>
>
>

## <a name="publishing-and-managing-a-web-api"></a><span data-ttu-id="e86f5-367">Web API の公開と管理</span><span class="sxs-lookup"><span data-stu-id="e86f5-367">Publishing and managing a web API</span></span>
<span data-ttu-id="e86f5-368">Web API をクライアント アプリケーションで使用できるようにするには、Web API をホスト環境にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-368">To make a web API available for client applications, the web API must be deployed to a host environment.</span></span> <span data-ttu-id="e86f5-369">通常、この環境は Web サーバーですが、他の種類のホスト プロセスの場合もあります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-369">This environment is typically a web server, although it may be some other type of host process.</span></span> <span data-ttu-id="e86f5-370">Web API を公開するときには、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-370">You should consider the following points when publishing a web API:</span></span>

* <span data-ttu-id="e86f5-371">すべての要求を認証および承認する必要があり、適切なレベルのアクセス制御を適用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-371">All requests must be authenticated and authorized, and the appropriate level of access control must be enforced.</span></span>
* <span data-ttu-id="e86f5-372">商用の Web API は、応答時間に関するさまざまな品質保証の対象となる場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-372">A commercial web API might be subject to various quality guarantees concerning response times.</span></span> <span data-ttu-id="e86f5-373">時間の経過に伴って負荷が大きく変わる可能性がある場合は、ホスト環境のスケーラビリティを確保することが重要です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-373">It is important to ensure that host environment is scalable if the load can vary significantly over time.</span></span>
* <span data-ttu-id="e86f5-374">収益化のために要求を測定することが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-374">It may be necessary to meter requests for monetization purposes.</span></span>
* <span data-ttu-id="e86f5-375">Web API へのトラフィック フローを調整し、クォータを使い果たしたクライアントのために調整を実装することが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-375">It might be necessary to regulate the flow of traffic to the web API, and implement throttling for specific clients that have exhausted their quotas.</span></span>
* <span data-ttu-id="e86f5-376">規制の要件により、すべての要求と応答のログと監査が義務付けられる場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-376">Regulatory requirements might mandate logging and auditing of all requests and responses.</span></span>
* <span data-ttu-id="e86f5-377">可用性を確保するために、Web API をホストするサーバーの正常性を監視し、必要に応じてサーバーを再起動することが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-377">To ensure availability, it may be necessary to monitor the health of the server hosting the web API and restart it if necessary.</span></span>

<span data-ttu-id="e86f5-378">これらの問題を、Web API の実装に関する技術的な問題から切り離すことができると便利です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-378">It is useful to be able to decouple these issues from the technical issues concerning the implementation of the web API.</span></span> <span data-ttu-id="e86f5-379">そのため、別のプロセスとして実行され、要求を Web API にルーティングする [ファサード](https://en.wikipedia.org/wiki/Facade_pattern)を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-379">For this reason, consider creating a [façade](https://en.wikipedia.org/wiki/Facade_pattern), running as a separate process and that routes requests to the web API.</span></span> <span data-ttu-id="e86f5-380">ファサードで管理操作を提供し、検証済みの要求を Web API に転送できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-380">The façade can provide the management operations and forward validated requests to the web API.</span></span> <span data-ttu-id="e86f5-381">また、ファサードを使用すると、次のような機能面での多くの利点がもたらされます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-381">Using a façade can also bring many functional advantages, including:</span></span>

* <span data-ttu-id="e86f5-382">複数の Web API の統合ポイントとして機能する。</span><span class="sxs-lookup"><span data-stu-id="e86f5-382">Acting as an integration point for multiple web APIs.</span></span>
* <span data-ttu-id="e86f5-383">さまざまなテクノロジを使用して構築されたクライアントに対応するために、メッセージと通信プロトコルを変換する。</span><span class="sxs-lookup"><span data-stu-id="e86f5-383">Transforming messages and translating communications protocols for clients built by using varying technologies.</span></span>
* <span data-ttu-id="e86f5-384">Web API をホストするサーバーの負荷を軽減するために、要求と応答をキャッシュする。</span><span class="sxs-lookup"><span data-stu-id="e86f5-384">Caching requests and responses to reduce load on the server hosting the web API.</span></span>

## <a name="testing-a-web-api"></a><span data-ttu-id="e86f5-385">Web API のテスト</span><span class="sxs-lookup"><span data-stu-id="e86f5-385">Testing a web API</span></span>
<span data-ttu-id="e86f5-386">ソフトウェアの他の部分と同様に、Web API を徹底的にテストする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-386">A web API should be tested as thoroughly as any other piece of software.</span></span> <span data-ttu-id="e86f5-387">追加の要件がある Web API の本質的な機能が正しく機能することを検証する単体テストを作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-387">You should consider creating unit tests to validate the functionality of The nature of a web API brings its own additional requirements to verify that it operates correctly.</span></span> <span data-ttu-id="e86f5-388">次の側面には特に注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-388">You should pay particular attention to the following aspects:</span></span>

* <span data-ttu-id="e86f5-389">すべてのルートをテストして、正しい操作が呼び出されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-389">Test all routes to verify that they invoke the correct operations.</span></span> <span data-ttu-id="e86f5-390">HTTP 状態コード 405 (Method Not Allowed) は、ルートとそのルートにディスパッチできる HTTP メソッド (GET、POST、PUT、DELETE) の不一致を示している場合があるため、この状態コードが予期せず返されたときは特に注意してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-390">Be especially aware of HTTP status code 405 (Method Not Allowed) being returned unexpectedly as this can indicate a mismatch between a route and the HTTP methods (GET, POST, PUT, DELETE) that can be dispatched to that route.</span></span>

    <span data-ttu-id="e86f5-391">HTTP 要求をサポートしていないルートに要求を送信します。たとえば、POST 要求を特定のリソースに送信します (POST 要求の送信先はリソース コレクションに限られます)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-391">Send HTTP requests to routes that do not support them, such as submitting a POST request to a specific resource (POST requests should only be sent to resource collections).</span></span> <span data-ttu-id="e86f5-392">このような場合、状態コード 405 (Not Allowed) が唯一の有効な応答である "*必要があります*"。</span><span class="sxs-lookup"><span data-stu-id="e86f5-392">In these cases, the only valid response *should* be status code 405 (Not Allowed).</span></span>
* <span data-ttu-id="e86f5-393">すべてのルートが適切に保護され、適切な認証および承認チェックの対象となっていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-393">Verify that all routes are protected properly and are subject to the appropriate authentication and authorization checks.</span></span>

  > [!NOTE]
  > <span data-ttu-id="e86f5-394">ユーザー認証など、セキュリティの一部の側面は、Web API ではなくホスト環境の責任範囲と考えられますが、デプロイメント プロセスの一環として、セキュリティ テストも含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-394">Some aspects of security such as user authentication are most likely to be the responsibility of the host environment rather than the web API, but it is still necessary to include security tests as part of the deployment process.</span></span>
  >
  >
* <span data-ttu-id="e86f5-395">各操作で実行される例外処理をテストし、意味のある適切な HTTP 応答がクライアント アプリケーションに返されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-395">Test the exception handling performed by each operation and verify that an appropriate and meaningful HTTP response is passed back to the client application.</span></span>
* <span data-ttu-id="e86f5-396">要求メッセージと応答メッセージが適切な形式であることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-396">Verify that request and response messages are well-formed.</span></span> <span data-ttu-id="e86f5-397">たとえば、HTTP POST 要求に x-www-form-urlencoded 形式で新しいリソースのデータが含まれている場合は、対応する操作でデータが正しく解析されていること、リソースが作成されていること、正しい Location ヘッダーなど、新しいリソースの詳細を含む応答が返されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-397">For example, if an HTTP POST request contains the data for a new resource in x-www-form-urlencoded format, confirm that the corresponding operation correctly parses the data, creates the resources, and returns a response containing the details of the new resource, including the correct Location header.</span></span>
* <span data-ttu-id="e86f5-398">応答メッセージのすべてのリンクと URI を確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-398">Verify all links and URIs in response messages.</span></span> <span data-ttu-id="e86f5-399">たとえば、HTTP POST メッセージでは、新しく作成されたリソースの URI を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-399">For example, an HTTP POST message should return the URI of the newly-created resource.</span></span> <span data-ttu-id="e86f5-400">すべての HATEOAS リンクが有効である必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-400">All HATEOAS links should be valid.</span></span>

* <span data-ttu-id="e86f5-401">各操作が入力のさまざまな組み合わせに対して正しい状態コードを返していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-401">Ensure that each operation returns the correct status codes for different combinations of input.</span></span> <span data-ttu-id="e86f5-402">例: </span><span class="sxs-lookup"><span data-stu-id="e86f5-402">For example:</span></span>

  * <span data-ttu-id="e86f5-403">クエリが成功した場合は、状態コード 200 (OK) を返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-403">If a query is successful, it should return status code 200 (OK)</span></span>
  * <span data-ttu-id="e86f5-404">リソースが見つからない場合は、HTTP 状態コード 404 (Not Found) を返します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-404">If a resource is not found, the operation should return HTTP status code 404 (Not Found).</span></span>
  * <span data-ttu-id="e86f5-405">クライアントがリソースを正常に削除する要求を送信した場合、状態コードは 204 (No Content) になります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-405">If the client sends a request that successfully deletes a resource, the status code should be 204 (No Content).</span></span>
  * <span data-ttu-id="e86f5-406">クライアントが新しいリソースを作成する要求を送信した場合、状態コードは 201 (Created) になります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-406">If the client sends a request that creates a new resource, the status code should be 201 (Created)</span></span>

<span data-ttu-id="e86f5-407">5xx の範囲の予期しない応答ステータ スコードに注意します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-407">Watch out for unexpected response status codes in the 5xx range.</span></span> <span data-ttu-id="e86f5-408">これらのメッセージは、有効な要求を処理できなかったことを示すために、通常はホスト サーバーによって報告されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-408">These messages are usually reported by the host server to indicate that it was unable to fulfill a valid request.</span></span>

* <span data-ttu-id="e86f5-409">クライアント アプリケーションが指定できる要求ヘッダーのさまざま組み合わせをテストし、Web API が求められている情報を応答メッセージで返していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-409">Test the different request header combinations that a client application can specify and ensure that the web API returns the expected information in response messages.</span></span>
* <span data-ttu-id="e86f5-410">クエリ文字列をテストします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-410">Test query strings.</span></span> <span data-ttu-id="e86f5-411">操作が省略可能なパラメーター (改ページ調整要求など) を受け取ることができる場合は、パラメーターのさまざまな組み合わせと順序をテストします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-411">If an operation can take optional parameters (such as pagination requests), test the different combinations and order of parameters.</span></span>
* <span data-ttu-id="e86f5-412">非同期操作が正常に完了していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-412">Verify that asynchronous operations complete successfully.</span></span> <span data-ttu-id="e86f5-413">Web API がラージ バイナリ オブジェクト (ビデオやオーディオなど) を返す要求のストリーミングをサポートしている場合は、データのストリーミング中にクライアント要求がブロックされていないことを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-413">If the web API supports streaming for requests that return large binary objects (such as video or audio), ensure that client requests are not blocked while the data is streamed.</span></span> <span data-ttu-id="e86f5-414">Web API が実行時間の長いデータ変更操作のポーリングを実装している場合は、操作の進行に伴って状態が正しく報告されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-414">If the web API implements polling for long-running data modification operations, verify that that the operations report their status correctly as they proceed.</span></span>

<span data-ttu-id="e86f5-415">パフォーマンス テストを作成して実行し、Web API が強制下で十分に機能していることを確認します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-415">You should also create and run performance tests to check that the web API operates satisfactorily under duress.</span></span> <span data-ttu-id="e86f5-416">Visual Studio Ultimate を使用して、Web パフォーマンスおよびロード テスト プロジェクトを作成できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-416">You can build a web performance and load test project by using Visual Studio Ultimate.</span></span> <span data-ttu-id="e86f5-417">詳細については、[リリース前のアプリケーションでのパフォーマンス テストの実行](https://msdn.microsoft.com/library/dn250793.aspx)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-417">For more information, see [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx).</span></span>

## <a name="using-azure-api-management"></a><span data-ttu-id="e86f5-418">Azure API Management の使用</span><span class="sxs-lookup"><span data-stu-id="e86f5-418">Using Azure API Management</span></span> 

<span data-ttu-id="e86f5-419">Azure で、Web API の公開と管理に [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/) を使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-419">On Azure, consider using [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/) to publish and manage a web API.</span></span> <span data-ttu-id="e86f5-420">この機能を使用して、1 つ以上の Web API のファサードとして機能するサービスを生成できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-420">Using this facility, you can generate a service that acts as a façade for one or more web APIs.</span></span> <span data-ttu-id="e86f5-421">このサービスは、Microsoft Azure 管理ポータルを使用して作成および構成できるスケーラブルな Web サービスです。</span><span class="sxs-lookup"><span data-stu-id="e86f5-421">The service is itself a scalable web service that you can create and configure by using the Azure Management portal.</span></span> <span data-ttu-id="e86f5-422">次のように、このサービスを使用して Web API を公開および管理できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-422">You can use this service to publish and manage a web API as follows:</span></span>

1. <span data-ttu-id="e86f5-423">Web サイト、Azure クラウド サービス、または Azure 仮想マシンに Web API をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-423">Deploy the web API to a website, Azure cloud service, or Azure virtual machine.</span></span>
2. <span data-ttu-id="e86f5-424">API Management サービスを Web API に接続します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-424">Connect the API management service to the web API.</span></span> <span data-ttu-id="e86f5-425">管理 API の URL に送信された要求は、Web API の URI にマップされます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-425">Requests sent to the URL of the management API are mapped to URIs in the web API.</span></span> <span data-ttu-id="e86f5-426">1 つの API Management サービスで、複数の Web API に要求をルーティングできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-426">The same API management service can route requests to more than one web API.</span></span> <span data-ttu-id="e86f5-427">これにより、複数の Web API を 1 つの管理サービスに集約できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-427">This enables you to aggregate multiple web APIs into a single management service.</span></span> <span data-ttu-id="e86f5-428">同様に、さまざまなアプリケーションが使用できる機能を制限したり、パーティションに分割したりする必要がある場合は、複数の API Management サービスから 1 つの Web API を参照できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-428">Similarly, the same web API can be referenced from more than one API management service if you need to restrict or partition the functionality available to different applications.</span></span>

   > [!NOTE]
   > <span data-ttu-id="e86f5-429">HTTP GET 要求に対する応答の一部として生成される HATEOAS リンクの URI は、Web API をホストする Web サーバーではなく、API Management サービスの URL を参照する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-429">The URIs in HATEOAS links generated as part of the response for HTTP GET requests should reference the URL of the API management service and not the web server hosting the web API.</span></span>
   >
   >
3. <span data-ttu-id="e86f5-430">Web API ごとに、その Web API が公開する HTTP 操作と、操作が入力として受け取ることができる省略可能なパラメーターを指定します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-430">For each web API, specify the HTTP operations that the web API exposes together with any optional parameters that an operation can take as input.</span></span> <span data-ttu-id="e86f5-431">同じデータに対して繰り返される要求を最適化するために、Web API から受信する応答を API Management サービスでキャッシュする必要があるかどうかを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-431">You can also configure whether the API management service should cache the response received from the web API to optimize repeated requests for the same data.</span></span> <span data-ttu-id="e86f5-432">各操作で生成できる HTTP 応答の詳細を記録します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-432">Record the details of the HTTP responses that each operation can generate.</span></span> <span data-ttu-id="e86f5-433">この情報を使用して開発者向けのドキュメントが生成されるので、情報が正確かつ完全なものであることが重要です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-433">This information is used to generate documentation for developers, so it is important that it is accurate and complete.</span></span>

   <span data-ttu-id="e86f5-434">操作は、Microsoft Azure 管理ポータルに用意されているウィザードを使用して手動で定義することも、定義が含まれたファイルから WADL 形式または Swagger 形式でインポートすることもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-434">You can either define operations manually using the wizards provided by the Azure Management portal, or you can import them from a file containing the definitions in WADL or Swagger format.</span></span>
4. <span data-ttu-id="e86f5-435">API Management サービスと Web API をホストする Web サーバー間の通信のセキュリティ設定を構成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-435">Configure the security settings for communications between the API management service and the web server hosting the web API.</span></span> <span data-ttu-id="e86f5-436">現在、API Management サービスは、証明書を使用した基本認証と相互認証、および OAuth 2.0 ユーザー認証をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-436">The API management service currently supports Basic authentication and mutual authentication using certificates, and OAuth 2.0 user authorization.</span></span>
5. <span data-ttu-id="e86f5-437">成果物を作成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-437">Create a product.</span></span> <span data-ttu-id="e86f5-438">成果物とはパブリケーションの単位です。前に管理サービスに接続した Web API を成果物に追加します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-438">A product is the unit of publication; you add the web APIs that you previously connected to the management service to the product.</span></span> <span data-ttu-id="e86f5-439">成果物を発行すると、それらの Web API を開発者が使用できるようになります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-439">When the product is published, the web APIs become available to developers.</span></span>

   > [!NOTE]
   > <span data-ttu-id="e86f5-440">成果物を発行する前に、その成果物にアクセスできるユーザー グループを定義し、それらのグループにユーザーを追加することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-440">Prior to publishing a product, you can also define user-groups that can access the product and add users to these groups.</span></span> <span data-ttu-id="e86f5-441">これにより、Web API を使用できる開発者とアプリケーションを制御できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-441">This gives you control over the developers and applications that can use the web API.</span></span> <span data-ttu-id="e86f5-442">Web API が承認を必要とする場合、Web API にアクセスするには、開発者が成果物管理者に要求を送信しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-442">If a web API is subject to approval, prior to being able to access it a developer must send a request to the product administrator.</span></span> <span data-ttu-id="e86f5-443">管理者は、開発者に対してアクセスを許可または拒否できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-443">The administrator can grant or deny access to the developer.</span></span> <span data-ttu-id="e86f5-444">状況が変わった場合は、既存の開発者をブロックすることもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-444">Existing developers can also be blocked if circumstances change.</span></span>
   >
   >
6. <span data-ttu-id="e86f5-445">各 Web API のポリシーを構成します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-445">Configure policies for each web API.</span></span> <span data-ttu-id="e86f5-446">ポリシーでは、ドメイン間呼び出しを許可するかどうか、クライアントの認証方法、XML データ形式と JSON データ形式間で透過的な変換を行うかどうか、特定の IP 範囲からの呼び出しを制限するかどうか、使用量のクォータ、呼び出しレートを制限するかどうかなどの側面を制御します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-446">Policies govern aspects such as whether cross-domain calls should be allowed, how to authenticate clients, whether to convert between XML and JSON data formats transparently, whether to restrict calls from a given IP range, usage quotas, and whether to limit the call rate.</span></span> <span data-ttu-id="e86f5-447">ポリシーは、成果物全体にグローバルに適用することも、成果物の 1 つの Web API に適用することもできます。また、Web API の個々の操作に適用することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-447">Policies can be applied globally across the entire product, for a single web API in a product, or for individual operations in a web API.</span></span>

<span data-ttu-id="e86f5-448">詳細については、[API Management のドキュメント](/azure/api-management/)をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-448">For more information, see the [API Management Documentation](/azure/api-management/).</span></span> 

> [!TIP]
> <span data-ttu-id="e86f5-449">Azure には、フェールオーバーと負荷分散を実装し、さまざまな地理的な場所でホストされている Web サイトの複数のインスタンスで待機時間を短縮できる Azure Traffic Manager が用意されています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-449">Azure provides the Azure Traffic Manager which enables you to implement failover and load-balancing, and reduce latency across multiple instances of a web site hosted in different geographic locations.</span></span> <span data-ttu-id="e86f5-450">Azure Traffic Manager は、API Management サービスと組み合わせて使用できます。API Management サービスでは、Azure Traffic Manager を使用して Web サイトのインスタンスに要求をルーティングできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-450">You can use Azure Traffic Manager in conjunction with the API Management Service; the API Management Service can route requests to instances of a web site through Azure Traffic Manager.</span></span>  <span data-ttu-id="e86f5-451">詳細については、「[Traffic Manager のルーティング方法](/azure/traffic-manager/traffic-manager-routing-methods/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-451">For more information, see [Traffic Manager routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/).</span></span>
>
> <span data-ttu-id="e86f5-452">この構造では、Web サイトにカスタム DNS 名を使用している場合に、各 Web サイトの適切な CNAME レコードを、Azure Traffic Manager の Web サイトの DNS 名を指すように構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-452">In this structure, if you are using custom DNS names for your web sites, you should configure the appropriate CNAME record for each web site to point to the DNS name of the Azure Traffic Manager web site.</span></span>
>

## <a name="supporting-client-side-developers"></a><span data-ttu-id="e86f5-453">クライアント側の開発者のサポート</span><span class="sxs-lookup"><span data-stu-id="e86f5-453">Supporting client-side developers</span></span>
<span data-ttu-id="e86f5-454">一般に、クライアント アプリケーションを構築する開発者は、Web API へのアクセス方法に関する情報と、パラメーター、データ型、戻り値の型、Web サービスとクライアント アプリケーション間のさまざまな要求と応答を示すリターン コードに関するドキュメントを必要としています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-454">Developers constructing client applications typically require information on how to access the web API, and documentation concerning the parameters, data types, return types, and return codes that describe the different requests and responses between the web service and the client application.</span></span>

### <a name="document-the-rest-operations-for-a-web-api"></a><span data-ttu-id="e86f5-455">Web API の REST 操作に関するドキュメント</span><span class="sxs-lookup"><span data-stu-id="e86f5-455">Document the REST operations for a web API</span></span>
<span data-ttu-id="e86f5-456">Azure API Management サービスには、Web API で公開される REST 操作について説明する開発者向けポータルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-456">The Azure API Management Service includes a developer portal that describes the REST operations exposed by a web API.</span></span> <span data-ttu-id="e86f5-457">成果物を発行すると、このポータルに表示されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-457">When a product has been published it appears on this portal.</span></span> <span data-ttu-id="e86f5-458">開発者は、このポータルを使用してアクセスへのサインアップを行うことができます。管理者は、この要求を承認または拒否できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-458">Developers can use this portal to sign up for access; the administrator can then approve or deny the request.</span></span> <span data-ttu-id="e86f5-459">開発者が承認されると、開発されたクライアント アプリケーションからの呼び出しの認証に使用するサブスクリプション キーが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-459">If the developer is approved, they are assigned a subscription key that is used to authenticate calls from the client applications that they develop.</span></span> <span data-ttu-id="e86f5-460">Web API 呼び出しごとにこのキーを提供する必要があります。そうしないと、呼び出しは拒否されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-460">This key must be provided with each web API call otherwise it will be rejected.</span></span>

<span data-ttu-id="e86f5-461">このポータルでは、以下も提供されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-461">This portal also provides:</span></span>

* <span data-ttu-id="e86f5-462">成果物、公開されている操作のリスト、必須パラメーター、および返される可能性のあるさまざまな応答に関するドキュメント。</span><span class="sxs-lookup"><span data-stu-id="e86f5-462">Documentation for the product, listing the operations that it exposes, the parameters required, and the different responses that can be returned.</span></span> <span data-ttu-id="e86f5-463">この情報は、「Microsoft Azure API Management サービスを使用した Web API の公開と管理」の手順 3. に記載されている詳細から生成されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-463">Note that this information is generated from the details provided in step 3 in the list in the Publishing a web API by using the Microsoft Azure API Management Service section.</span></span>
* <span data-ttu-id="e86f5-464">JavaScript、C#、Java、Ruby、Python、PHP など、複数の言語から操作を呼び出す方法を示すコード スニペット。</span><span class="sxs-lookup"><span data-stu-id="e86f5-464">Code snippets that show how to invoke operations from several languages, including JavaScript, C#, Java, Ruby, Python, and PHP.</span></span>
* <span data-ttu-id="e86f5-465">開発者が HTTP 要求を送信して成果物の各操作をテストし、結果を表示できる開発者向けコンソール。</span><span class="sxs-lookup"><span data-stu-id="e86f5-465">A developers' console that enables a developer to send an HTTP request to test each operation in the product and view the results.</span></span>
* <span data-ttu-id="e86f5-466">開発者が課題や見つかった問題を報告できるページ。</span><span class="sxs-lookup"><span data-stu-id="e86f5-466">A page where the developer can report any issues or problems found.</span></span>

<span data-ttu-id="e86f5-467">Microsoft Azure 管理ポータルでは、開発者向けポータルをカスタマイズして、組織のブランドに合わせてスタイル設定やレイアウトを変更できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-467">The Azure Management portal enables you to customize the developer portal to change the styling and layout to match the branding of your organization.</span></span>

### <a name="implement-a-client-sdk"></a><span data-ttu-id="e86f5-468">クライアント SDK の実装</span><span class="sxs-lookup"><span data-stu-id="e86f5-468">Implement a client SDK</span></span>
<span data-ttu-id="e86f5-469">REST 要求を呼び出して Web API にアクセスするクライアント アプリケーションを構築するには、各要求の作成と適切な書式設定、Web サービスをホストするサーバーへの要求の送信、応答の解析による要求の成功/失敗の確認と返されたデータの抽出など、大量のコードを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-469">Building a client application that invokes REST requests to access a web API requires writing a significant amount of code to construct each request and format it appropriately, send the request to the server hosting the web service, and parse the response to work out whether the request succeeded or failed and extract any data returned.</span></span> <span data-ttu-id="e86f5-470">クライアント アプリケーションをこれらの問題から切り離すには、REST インターフェイスをラップし、これらの低レベルの詳細をより機能的な一連のメソッド内で抽象化する SDK を提供します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-470">To insulate the client application from these concerns, you can provide an SDK that wraps the REST interface and abstracts these low-level details inside a more functional set of methods.</span></span> <span data-ttu-id="e86f5-471">クライアント アプリケーションでこれらのメソッドを使用して、呼び出しを REST 要求に透過的に変換し、応答をメソッドの戻り値に変換します。</span><span class="sxs-lookup"><span data-stu-id="e86f5-471">A client application uses these methods, which transparently convert calls into REST requests and then convert the responses back into method return values.</span></span> <span data-ttu-id="e86f5-472">これは、Azure SDK をはじめとする多くのサービスで実装されている一般的な手法です。</span><span class="sxs-lookup"><span data-stu-id="e86f5-472">This is a common technique that is implemented by many services, including the Azure SDK.</span></span>

<span data-ttu-id="e86f5-473">クライアント側の SDK を作成する場合、SDK を一貫した方法で実装し、入念にテストする必要があるため、かなりの作業量になります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-473">Creating a client-side SDK is a considerable undertaking as it has to be implemented consistently and tested carefully.</span></span> <span data-ttu-id="e86f5-474">ただし、このプロセスの大部分は機械的に行うことができるので、多数のベンダーがこれらのタスクの多くを自動化できるツールを提供しています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-474">However, much of this process can be made mechanical, and many vendors supply tools that can automate many of these tasks.</span></span>

## <a name="monitoring-a-web-api"></a><span data-ttu-id="e86f5-475">Web API の監視</span><span class="sxs-lookup"><span data-stu-id="e86f5-475">Monitoring a web API</span></span>
<span data-ttu-id="e86f5-476">Web API を公開し、デプロイした方法に応じて、Web API を直接監視することも、API Management サービスを使用して通過するトラフィックを分析し、利用状況と正常性に関する情報を収集することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-476">Depending on how you have published and deployed your web API you can monitor the web API directly, or you can gather usage and health information by analyzing the traffic that passes through the API Management service.</span></span>

### <a name="monitoring-a-web-api-directly"></a><span data-ttu-id="e86f5-477">Web API の直接監視</span><span class="sxs-lookup"><span data-stu-id="e86f5-477">Monitoring a web API directly</span></span>
<span data-ttu-id="e86f5-478">Visual Studio 2013 と、ASP.NET Web API テンプレートを (Azure クラウド サービスで Web API プロジェクトまたは Web ロールとして) 使用して Web API を実装した場合は、ASP.NET Application Insights を使用して、可用性、パフォーマンス、利用状況のデータを収集できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-478">If you have implemented your web API by using the ASP.NET Web API template (either as a Web API project or as a Web role in an Azure cloud service) and Visual Studio 2013, you can gather availability, performance, and usage data by using ASP.NET Application Insights.</span></span> <span data-ttu-id="e86f5-479">Application Insights は、Web API がクラウドにデプロイされている場合に、要求と応答に関する情報を透過的に追跡して記録するパッケージです。このパッケージをインストールして構成したら、パッケージを使用するために Web API でコードを修正する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="e86f5-479">Application Insights is a package that transparently tracks and records information about requests and responses when the web API is deployed to the cloud; once the package is installed and configured, you don't need to amend any code in your web API to use it.</span></span> <span data-ttu-id="e86f5-480">Web API を Azure Web サイトにデプロイすると、すべてのトラフィックが検査され、次の統計情報が収集されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-480">When you deploy the web API to an Azure web site, all traffic is examined and the following statistics are gathered:</span></span>

* <span data-ttu-id="e86f5-481">サーバーの応答時間。</span><span class="sxs-lookup"><span data-stu-id="e86f5-481">Server response time.</span></span>
* <span data-ttu-id="e86f5-482">サーバー要求の数と各要求の詳細。</span><span class="sxs-lookup"><span data-stu-id="e86f5-482">Number of server requests and the details of each request.</span></span>
* <span data-ttu-id="e86f5-483">平均応答時間の観点で最も時間がかかった要求。</span><span class="sxs-lookup"><span data-stu-id="e86f5-483">The top slowest requests in terms of average response time.</span></span>
* <span data-ttu-id="e86f5-484">失敗した要求の詳細。</span><span class="sxs-lookup"><span data-stu-id="e86f5-484">The details of any failed requests.</span></span>
* <span data-ttu-id="e86f5-485">さまざまなブラウザーやユーザー エージェントによって開始されたセッションの数。</span><span class="sxs-lookup"><span data-stu-id="e86f5-485">The number of sessions initiated by different browsers and user agents.</span></span>
* <span data-ttu-id="e86f5-486">最も頻繁に表示されたページ (Web API ではなく、主に Web アプリケーションで役立ちます)。</span><span class="sxs-lookup"><span data-stu-id="e86f5-486">The most frequently viewed pages (primarily useful for web applications rather than web APIs).</span></span>
* <span data-ttu-id="e86f5-487">Web API にアクセスする各種ユーザー ロール。</span><span class="sxs-lookup"><span data-stu-id="e86f5-487">The different user roles accessing the web API.</span></span>

<span data-ttu-id="e86f5-488">このデータは、Microsoft Azure 管理ポータルからリアルタイムで表示できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-488">You can view this data in real time from the Azure Management portal.</span></span> <span data-ttu-id="e86f5-489">Web API の正常性を監視する WebTest を作成することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-489">You can also create webtests that monitor the health of the web API.</span></span> <span data-ttu-id="e86f5-490">WebTest では、Web API の指定した URI に定期的な要求を送信し、応答をキャプチャします。</span><span class="sxs-lookup"><span data-stu-id="e86f5-490">A webtest sends a periodic request to a specified URI in the web API and captures the response.</span></span> <span data-ttu-id="e86f5-491">正常な応答の定義 (HTTP 状態コード 200 など) を指定し、要求でこの応答が返されない場合に、管理者に送信するアラートを準備できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-491">You can specify the definition of a successful response (such as HTTP status code 200), and if the request does not return this response you can arrange for an alert to be sent to an administrator.</span></span> <span data-ttu-id="e86f5-492">必要に応じて、管理者は Web API をホストするサーバーで障害が発生した場合にサーバーを再起動できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-492">If necessary, the administrator can restart the server hosting the web API if it has failed.</span></span>

<span data-ttu-id="e86f5-493">詳細については、[Application Insights - ASP.NET の概要](/azure/application-insights/app-insights-asp-net/)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-493">For more information, see [Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/).</span></span>

### <a name="monitoring-a-web-api-through-the-api-management-service"></a><span data-ttu-id="e86f5-494">API Management サービスを使用した Web API の監視</span><span class="sxs-lookup"><span data-stu-id="e86f5-494">Monitoring a web API through the API Management Service</span></span>
<span data-ttu-id="e86f5-495">API Management サービスを使用して Web API を公開した場合、Microsoft Azure 管理ポータルの API Management ページにあるダッシュボードを使用して、サービスの全体的なパフォーマンスを表示できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-495">If you have published your web API by using the API Management service, the API Management page on the Azure Management portal contains a dashboard that enables you to view the overall performance of the service.</span></span> <span data-ttu-id="e86f5-496">[分析] ページでは、製品の使用状況の詳細にドリルダウンできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-496">The Analytics page enables you to drill down into the details of how the product is being used.</span></span> <span data-ttu-id="e86f5-497">このページには次のタブがあります。</span><span class="sxs-lookup"><span data-stu-id="e86f5-497">This page contains the following tabs:</span></span>

* <span data-ttu-id="e86f5-498">**[使用状況]**: </span><span class="sxs-lookup"><span data-stu-id="e86f5-498">**Usage**.</span></span> <span data-ttu-id="e86f5-499">このタブには、API 呼び出し数とこれらの呼び出しを経時的に処理する際に使用された帯域幅に関する情報が表示されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-499">This tab provides information about the number of API calls made and the bandwidth used to handle these calls over time.</span></span> <span data-ttu-id="e86f5-500">利用状況の詳細は、成果物、API、操作でフィルターできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-500">You can filter usage details by product, API, and operation.</span></span>
* <span data-ttu-id="e86f5-501">**[正常性]**: </span><span class="sxs-lookup"><span data-stu-id="e86f5-501">**Health**.</span></span> <span data-ttu-id="e86f5-502">このタブでは、API 要求の結果 (返された HTTP ステータ スコード)、キャッシュ ポリシーの有効性、API の応答時間、サービスの応答時間を表示できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-502">This tab enables you to view the outcome of API requests (the HTTP status codes returned), the effectiveness of the caching policy, the API response time, and the service response time.</span></span> <span data-ttu-id="e86f5-503">このタブでも、正常性データを成果物、API、操作でフィルターできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-503">Again, you can filter health data by product, API, and operation.</span></span>
* <span data-ttu-id="e86f5-504">**アクティビティ**。</span><span class="sxs-lookup"><span data-stu-id="e86f5-504">**Activity**.</span></span> <span data-ttu-id="e86f5-505">このタブでは、成功した呼び出し数、失敗した呼び出し数、ブロックされた呼び出し数、平均応答時間、成果物、Web API、操作ごとの応答時間がテキストでまとめられています。</span><span class="sxs-lookup"><span data-stu-id="e86f5-505">This tab provides a text summary of the numbers of successful calls, failed calls, blocked calls, average response time, and response times for each product, web API, and operation.</span></span> <span data-ttu-id="e86f5-506">このページには、各開発者による呼び出し数も示されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-506">This page also lists the number of calls made by each developer.</span></span>
* <span data-ttu-id="e86f5-507">**[概要]**: </span><span class="sxs-lookup"><span data-stu-id="e86f5-507">**At a glance**.</span></span> <span data-ttu-id="e86f5-508">このタブには、最も多くの API 呼び出しを行った開発者、これらの呼び出しを受信した成果物、Web API、および操作など、パフォーマンス データの概要が表示されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-508">This tab displays a summary of the performance data, including the developers responsible for making the most API calls, and the products, web APIs, and operations that received these calls.</span></span>

<span data-ttu-id="e86f5-509">この情報を使用して、特定の Web API または操作がボトルネックの原因となっているかどうかを確認し、必要に応じてホスト環境を拡張したり、サーバーをさらに追加したりできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-509">You can use this information to determine whether a particular web API or operation is causing a bottleneck, and if necessary scale the host environment and add more servers.</span></span> <span data-ttu-id="e86f5-510">また、1 つ以上のアプリケーションがリソースを過度に使用していないかどうかを確認し、クォータを設定して呼び出しレートを制限する適切なポリシーを適用することもできます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-510">You can also ascertain whether one or more applications are using a disproportionate volume of resources and apply the appropriate policies to set quotas and limit call rates.</span></span>

> [!NOTE]
> <span data-ttu-id="e86f5-511">発行された成果物の詳細を変更できます。変更はすぐに適用されます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-511">You can change the details for a published product, and the changes are applied immediately.</span></span> <span data-ttu-id="e86f5-512">たとえば、Web API を含む成果物を再発行しなくても、その Web API に対して操作を追加または削除できます。</span><span class="sxs-lookup"><span data-stu-id="e86f5-512">For example, you can add or remove an operation from a web API without requiring that you republish the product that contains the web API.</span></span>
>
>

## <a name="more-information"></a><span data-ttu-id="e86f5-513">詳細情報</span><span class="sxs-lookup"><span data-stu-id="e86f5-513">More information</span></span>
* <span data-ttu-id="e86f5-514">ASP.NET を使用した OData Web API の実装の例と詳細については、[ASP.NET Web API OData](https://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-514">[ASP.NET Web API OData](https://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) contains examples and further information on implementing an OData web API by using ASP.NET.</span></span>
* <span data-ttu-id="e86f5-515">OData を使用して Web API でバッチ操作を実装する方法については、「[Introducing Batch Support in Web API and Web API OData (Web API と Web API OData でのバッチ処理のサポートの概要)](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-515">[Introducing Batch Support in Web API and Web API OData](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/) describes how to implement batch operations in a web API by using OData.</span></span>
* <span data-ttu-id="e86f5-516">べき等の概要とデータ管理操作との関連性については、Jonathan Oliver のブログ記事「[Idempotency Patterns (べき等のパターン)](https://blog.jonathanoliver.com/idempotency-patterns/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-516">[Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
* <span data-ttu-id="e86f5-517">HTTP 状態コードの全一覧と説明については、W3C の Web サイトの「[Status Code Definitions (状態コードの定義)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-517">[Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) on the W3C website contains a full list of HTTP status codes and their descriptions.</span></span>
* <span data-ttu-id="e86f5-518">Web ジョブを使用してバックグラウンド操作を実行する方法と例については、Microsoft Web サイト「[Web ジョブでバックグラウンド タスクを実行する](/azure/app-service-web/web-sites-create-web-jobs/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-518">[Run background tasks with WebJobs](/azure/app-service-web/web-sites-create-web-jobs/) provides information and examples on using WebJobs to perform background operations.</span></span>
* <span data-ttu-id="e86f5-519">Azure Notification Hubs を使用して非同期応答をクライアント アプリケーションにプッシュする方法については、「[Azure Notification Hubs によるユーザーへの通知](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-519">[Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) shows how to use an Azure Notification Hub to push asynchronous responses to client applications.</span></span>
* <span data-ttu-id="e86f5-520">Web API への制御された安全なアクセスを提供する成果物を発行する方法については、「[API Management](https://azure.microsoft.com/services/api-management/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-520">[API Management](https://azure.microsoft.com/services/api-management/) describes how to publish a product that provides controlled and secure access to a web API.</span></span>
* <span data-ttu-id="e86f5-521">API Management REST API を使用して、カスタム管理アプリケーションを構築する方法については、「[Azure API Management REST API リファレンス](https://msdn.microsoft.com/library/azure/dn776326.aspx)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-521">[Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx) describes how to use the API Management REST API to build custom management applications.</span></span>
* <span data-ttu-id="e86f5-522">Azure Traffic Manager を使用して、Web API をホストする Web サイトの複数のインスタンスに要求の負荷を分散する方法の概要については、[Traffic Manager のルーティング方法](/azure/traffic-manager/traffic-manager-routing-methods/)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-522">[Traffic Manager Routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/) summarizes how Azure Traffic Manager can be used to load-balance requests across multiple instances of a website hosting a web API.</span></span>
* <span data-ttu-id="e86f5-523">ASP.NET Web API プロジェクトでの Application Insights のインストールと構成の詳細については、 [ASP.NET での Application Insights の使用](/azure/application-insights/app-insights-asp-net/)に関するページをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e86f5-523">[Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/) provides detailed information on installing and configuring Application Insights in an ASP.NET Web API project.</span></span>


<!-- links -->

[api-design]: ./api-design.md
