---
title: "API 設計ガイダンス"
description: "適切に設計された API の作成方法についてのガイダンス。"
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a><span data-ttu-id="b6d88-103">API 設計</span><span class="sxs-lookup"><span data-stu-id="b6d88-103">API design</span></span>
[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="b6d88-104">現代の多くの Web ベース ソリューションでは、Web サーバーによりホストされる Web サービスを使用することで、リモート クライアント アプリケーションの機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-104">Many modern web-based solutions make the use of web services, hosted by web servers, to provide functionality for remote client applications.</span></span> <span data-ttu-id="b6d88-105">Web サービスにより公開される操作によって Web API が構成されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-105">The operations that a web service exposes constitute a web API.</span></span> <span data-ttu-id="b6d88-106">適切に設計された Web API には、次をサポートする目的があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-106">A well-designed web API should aim to support:</span></span>

* <span data-ttu-id="b6d88-107">**プラットフォームの独立**。</span><span class="sxs-lookup"><span data-stu-id="b6d88-107">**Platform independence**.</span></span> <span data-ttu-id="b6d88-108">クライアント アプリケーションは、API により公開されるデータや操作の物理的な実装方法を要求することなく、Web サービスが提供する API を利用できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-108">Client applications should be able to utilize the API that the web service provides without requiring how the data or operations that API exposes are physically implemented.</span></span> <span data-ttu-id="b6d88-109">そのためには、API が共通基準を順守し、クライアント アプリケーションおよび Web サービスが、使用するデータ形式や、クライアント アプリケーションと Web サービスの間で交換されるデータ構造に同意できるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-109">This requires that the API abides by common standards that enable a client application and web service to agree on which data formats to use, and the structure of the data that is exchanged between client applications and the web service.</span></span>
* <span data-ttu-id="b6d88-110">**サービスの進化**。</span><span class="sxs-lookup"><span data-stu-id="b6d88-110">**Service evolution**.</span></span> <span data-ttu-id="b6d88-111">Web サービスはクライアント アプリケーションから独立して進化し、機能を追加 (または削除) できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-111">The web service should be able to evolve and add (or remove) functionality independently from client applications.</span></span> <span data-ttu-id="b6d88-112">Web サービスにより提供される機能は変化するため、既存のクライアント アプリケーションは変化せずに操作を継続できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-112">Existing client applications should be able to continue to operate unmodified as the features provided by the web service change.</span></span> <span data-ttu-id="b6d88-113">すべての機能も検出可能で、クライアント アプリケーションで十分に利用できるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-113">All functionality should also be discoverable, so that client applications can fully utilize it.</span></span>

<span data-ttu-id="b6d88-114">このガイダンスでは、Web API を設計するときに考慮する問題について説明します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-114">The purpose of this guidance is to describe the issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-representational-state-transfer-rest"></a><span data-ttu-id="b6d88-115">Representational State Transfer (REST) の概要</span><span class="sxs-lookup"><span data-stu-id="b6d88-115">Introduction to Representational State Transfer (REST)</span></span>
<span data-ttu-id="b6d88-116">ロイ・フィールディングは 2000 年に著した論文で、Web サービスにより公開される操作を構築する、代わりとなるアーキテクチャ アプローチを提唱しました。それが REST です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-116">In his dissertation in 2000, Roy Fielding proposed an alternative architectural approach to structuring the operations exposed by web services; REST.</span></span> <span data-ttu-id="b6d88-117">REST はハイパーメディアに基づき分散システムを構築するアーキテクチャ スタイルです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-117">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="b6d88-118">REST モデルの主な利点は、オープン スタンダードに基づいており、それにアクセスするモデルやクライアント アプリケーションの実装を、特定の実装に結びつけないという点です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-118">A primary advantage of the REST model is that it is based on open standards and does not bind the implementation of the model or the client applications that access it to any specific implementation.</span></span> <span data-ttu-id="b6d88-119">たとえば、REST Web サービスは Microsoft ASP.NET Web API を使用して実装でき、クライアント アプリケーションは任意の言語および HTTP 要求の生成と HTTP 応答の解析を行うことができるツールセットを使用して開発できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-119">For example, a REST web service could be implemented by using the Microsoft ASP.NET Web API, and client applications could be developed by using any language and toolset that can generate HTTP requests and parse HTTP responses.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-120">実際、REST は基盤となるプロトコルに一切依存せず、必ずしも HTTP と結び付けられていません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-120">REST is actually independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="b6d88-121">とはいえ、REST に基づくシステムを実装する際には、要求を送受信するアプリケーション プロトコルとして HTTP を利用するのが最も一般的です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-121">However, most common implementations of systems that are based on REST utilize HTTP as the application protocol for sending and receiving requests.</span></span> <span data-ttu-id="b6d88-122">本文書では、HTTP を使用して操作するように設計されているシステムに REST の原則をマッピングすることに重点を置いて説明します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-122">This document focuses on mapping REST principles to systems designed to operate using HTTP.</span></span>
>
>

<span data-ttu-id="b6d88-123">REST モデルでは、ネットワーク経由のオブジェクトおよびサービスを表すために、ナビゲーション スキームを使用します ("*リソース*" と呼ばれる)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-123">The REST model uses a navigational scheme to represent objects and services over a network (referred to as *resources*).</span></span> <span data-ttu-id="b6d88-124">通常、REST を実装する多くのシステムでは、これらのリソースにアクセスするための要求を送信するのに HTTP プロトコルを使用します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-124">Many systems that implement REST typically use the HTTP protocol to transmit requests to access these resources.</span></span> <span data-ttu-id="b6d88-125">これらのシステムでクライアント アプリケーションは、リソースを識別する URI、およびそのリソースで実行する操作を示す HTTP メソッド (最も一般的なのは GET、POST、PUT、DELETE) の形式で要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-125">In these systems, a client application submits a request in the form of a URI that identifies a resource, and an HTTP method (the most common being GET, POST, PUT, or DELETE) that indicates the operation to be performed on that resource.</span></span>  <span data-ttu-id="b6d88-126">HTTP 要求の本文には、操作を実行するのに必要なデータが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-126">The body of the HTTP request contains the data required to perform the operation.</span></span> <span data-ttu-id="b6d88-127">REST はステートレスの要求モデルを定義することを理解することは重要です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-127">The important point to understand is that REST defines a stateless request model.</span></span> <span data-ttu-id="b6d88-128">HTTP 要求は独立している必要があり、任意の順序で起こる場合があります。そのため、要求間の遷移状態の情報を保持しようとすることはできません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-128">HTTP requests should be independent and may occur in any order, so attempting to retain transient state information between requests is not feasible.</span></span>  <span data-ttu-id="b6d88-129">情報の格納場所はリソース自体のみであり、それぞれの要求はアトミックな操作である必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-129">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="b6d88-130">実用上、REST モデルは Finite State Machine を実装し、要求によって、リソースは適切に定義された非遷移状態から別の状態に切り替えられます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-130">Effectively, a REST model implements a finite state machine where a request transitions a resource from one well-defined non-transient state to another.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-131">REST モデルの個々の要求はステートレスであるため、これらの原則に従って構築されたシステムには非常に高い拡張性を持たせることができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-131">The stateless nature of individual requests in the REST model enables a system constructed by following these principles to be highly scalable.</span></span> <span data-ttu-id="b6d88-132">一連の要求を行うクライアント アプリケーションと、それらの要求を処理する特定の Web サーバーの関係を保持する必要は一切ありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-132">There is no need to retain any affinity between a client application making a series of requests and the specific web servers handling those requests.</span></span>
>
>

<span data-ttu-id="b6d88-133">効率的な REST モデルを実装するうえで他にも重要な点は、モデルによりアクセスが提供されるさまざまなリソース間のリレーションシップを理解することです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-133">Another crucial point in implementing an effective REST model is to understand the relationships between the various resources to which the model provides access.</span></span> <span data-ttu-id="b6d88-134">通常、これらのリソースはコレクションおよびリレーションシップとして構成されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-134">These resources are typically organized as collections and relationships.</span></span> <span data-ttu-id="b6d88-135">たとえば、電子商取引システムの迅速な分析により、クライアント アプリケーションが関心を持つ可能性の高い、注文と顧客という 2 つのコレクションがあることが示されると想定してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-135">For example, suppose that a quick analysis of an ecommerce system shows that there are two collections in which client applications are likely to be interested: orders and customers.</span></span> <span data-ttu-id="b6d88-136">それぞれの注文と顧客には、特定するための独自の一意なキーがある必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-136">Each order and customer should have its own unique key for identification purposes.</span></span> <span data-ttu-id="b6d88-137">注文のコレクションにアクセスする URI は、*/orders* のように単純に記述でき、同様にすべての顧客を取得する URI は */customers* のように単純に記述できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-137">The URI to access the collection of orders could be something as simple as */orders*, and similarly the URI for retrieving all customers could be */customers*.</span></span> <span data-ttu-id="b6d88-138">*/orders* URI に HTTP GET 要求を発行すると、HTTP 応答としてエンコードされたコレクションのすべての注文を表すリストが返されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-138">Issuing an HTTP GET request to the */orders* URI should return a list representing all orders in the collection encoded as an HTTP response:</span></span>

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

<span data-ttu-id="b6d88-139">以下に示す応答は、注文を JSON リスト構造でエンコードします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-139">The response shown below encodes the orders as a JSON list structure:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
<span data-ttu-id="b6d88-140">個々の注文を取得するには、注文の識別子を */orders/2* などの *orders* リソースから指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-140">To fetch an individual order requires specifying the identifier for the order from the *orders* resource, such as */orders/2*:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> <span data-ttu-id="b6d88-141">わかりやすくするため、これらの例では JSON テキスト データとして返される応答の情報が表示されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-141">For simplicity, these examples show the information in responses being returned as JSON text data.</span></span> <span data-ttu-id="b6d88-142">ただし、バイナリ情報や暗号化情報など HTTP によりサポートされている他の種類のデータをリソースに一切含めるべきでない理由はありません。HTTP 応答の content-type では種類を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-142">However, there is no reason why resources should not contain any other type of data supported by HTTP, such as binary or encrypted information; the content-type in the HTTP response should specify the type.</span></span> <span data-ttu-id="b6d88-143">また、REST モデルは XML や JSON など、異なる形式の同じデータを返すことができる場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-143">Also, a REST model may be able to return the same data in different formats, such as XML or JSON.</span></span> <span data-ttu-id="b6d88-144">この場合、Web サービスは要求を行うクライアントとのコンテント ネゴシエーションを実行できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-144">In this case, the web service should be able to perform content negotiation with the client making the request.</span></span> <span data-ttu-id="b6d88-145">要求にはクライアントが受け取る優先形式を指定する *Accept* ヘッダーを含めることができ、Web サービスは可能な限りこの形式を使用しようとする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-145">The request can include an *Accept* header which specifies the preferred format that the client would like to receive and the web service should attempt to honor this format if at all possible.</span></span>
>
>

<span data-ttu-id="b6d88-146">REST 要求からの応答では標準 HTTP 状態 コードが使用されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-146">Notice that the response from a REST request makes use of the standard HTTP status codes.</span></span> <span data-ttu-id="b6d88-147">たとえば、有効なデータを返す要求には HTTP 応答コード 200 (OK) を含める必要があり、指定したリソースの確認および削除に失敗した要求は HTTP 状態コード 404 (Not Found) を含む応答を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-147">For example, a request that returns valid data should include the HTTP response code 200 (OK), while a request that fails to find or delete a specified resource should return a response that includes the HTTP status code 404 (Not Found).</span></span>

## <a name="design-and-structure-of-a-restful-web-api"></a><span data-ttu-id="b6d88-148">RESTful Web API の設計と構造</span><span class="sxs-lookup"><span data-stu-id="b6d88-148">Design and structure of a RESTful web API</span></span>
<span data-ttu-id="b6d88-149">正常な Web API を設計するポイントは、単純さと一貫性です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-149">The keys to designing a successful web API are simplicity and consistency.</span></span> <span data-ttu-id="b6d88-150">Web API がこれらの 2 つの要素を備えていると、API を消費する必要があるクライアント アプリケーションを簡単に構築できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-150">A Web API that exhibits these two factors makes it easier to build client applications that need to consume the API.</span></span>

<span data-ttu-id="b6d88-151">RESTful Web API は一連の接続されたリソースを公開し、アプリケーションがこれらのリソースを操作してリソース間を簡単にナビゲートできるようにするコア操作を提供することに重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-151">A RESTful web API is focused on exposing a set of connected resources, and providing the core operations that enable an application to manipulate these resources and easily navigate between them.</span></span> <span data-ttu-id="b6d88-152">このため、一般的な RESTful Web API を構成する URI は、公開するデータ指向でなければならず、このデータを操作するために HTTP により提供される機能を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-152">For this reason, the URIs that constitute a typical RESTful web API should be oriented towards the data that it exposes, and use the facilities provided by HTTP to operate on this data.</span></span> <span data-ttu-id="b6d88-153">オブジェクトおよびクラスの動作に促される傾向がある オブジェクト指向 API の一連のクラスを設計する場合、このアプローチには一般的に採用されている考え方とは異なる考え方が必要です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-153">This approach requires a different mindset from that typically employed when designing a set of classes in an object-oriented API which tends to be more motivated by the behavior of objects and classes.</span></span> <span data-ttu-id="b6d88-154">さらに、RESTful Web API はステートレスで、特定のシーケンスで呼び出される操作に依存しない必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-154">Additionally, a RESTful web API should be stateless and not depend on operations being invoked in a particular sequence.</span></span> <span data-ttu-id="b6d88-155">次のセクションでは、RESTful Wb API を設計するときに考慮する必要があるポイントをまとめます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-155">The following sections summarize the points you should consider when designing a RESTful web API.</span></span>

### <a name="organizing-the-web-api-around-resources"></a><span data-ttu-id="b6d88-156">リソースに関係する Web API の構成</span><span class="sxs-lookup"><span data-stu-id="b6d88-156">Organizing the web API around resources</span></span>
> [!TIP]
> <span data-ttu-id="b6d88-157">REST Web サービスにより公開される URI は名詞 (Web API プロバイダーがアクセスするデータ) に基づくものであり、動詞 (データを使用してアプリケーションが行うことができること) に基づくものにはできません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-157">The URIs exposed by a REST web service should be based on nouns (the data to which the web API provides access) and not verbs (what an application can do with the data).</span></span>
>
>

<span data-ttu-id="b6d88-158">Web API が公開するビジネス エンティティに注目して説明します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-158">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="b6d88-159">たとえば、前に説明した電子商取引システムをサポートするために設計された Web API で、主エンティティは顧客と注文です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-159">For example, in a web API designed to support the ecommerce system described earlier, the primary entities are customers and orders.</span></span> <span data-ttu-id="b6d88-160">発注行為などのプロセスは、注文情報を取り込み、それを顧客の注文リストに追加する HTTP POST 操作を指定することで行うことができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-160">Processes such as the act of placing an order can be achieved by providing an HTTP POST operation that takes the order information and adds it to the list of orders for the customer.</span></span> <span data-ttu-id="b6d88-161">内部ではこの POST 操作により、在庫レベルの確認や顧客への請求などのタスクを実行できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-161">Internally, this POST operation can perform tasks such as checking stock levels, and billing the customer.</span></span> <span data-ttu-id="b6d88-162">HTTP 応答は、発注が正常に行われたかどうかを示すことができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-162">The HTTP response can indicate whether the order was placed successfully or not.</span></span> <span data-ttu-id="b6d88-163">リソースは単一の物理データ項目に基づく必要はないことにも注意してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-163">Also note that a resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="b6d88-164">たとえば注文リソースは、リレーショナル データベースの複数のテーブルに分散する多くの行から集計した情報を使用して内部的に実装されるものの、クライアントには単一のエンティティとして表示される場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-164">As an example, an order resource might be implemented internally by using information aggregated from many rows spread across several tables in a relational database but presented to the client as a single entity.</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-165">設計する REST インターフェイスが、公開するデータの内部構造をミラーリングしたり、それに依存しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-165">Avoid designing a REST interface that mirrors or depends on the internal structure of the data that it exposes.</span></span> <span data-ttu-id="b6d88-166">REST は、リレーショナル データベースの個別のテーブルに対して単純な CRUD (作成、取得、更新、削除) 操作を実行するだけのものではありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-166">REST is about more than implementing simple CRUD (Create, Retrieve, Update, Delete) operations over separate tables in a relational database.</span></span> <span data-ttu-id="b6d88-167">REST の目的は、ビジネス エンティティおよびアプリケーションがそれらのエンティティに実行できる操作を、それらのエンティティの物理実装にマッピングすることですが、これらの物理的詳細をクライアントに公開すべきではありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-167">The purpose of REST is to map business entities and the operations that an application can perform on these entities to the physical implementation of these entities, but a client should not be exposed to these physical details.</span></span>
>
>

<span data-ttu-id="b6d88-168">個々のビジネス エンティティはまれに分離して存在していますが (ただし、いくつかのシングルトン オブジェクトが存在する場合があります)、むしろ共にコレクションにグループ化される傾向にあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-168">Individual business entities rarely exist in isolation (although some singleton objects may exist), but instead tend to be grouped together into collections.</span></span> <span data-ttu-id="b6d88-169">REST 用語では、各エンティティおよび各コレクションはリソースです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-169">In REST terms, each entity and each collection are resources.</span></span> <span data-ttu-id="b6d88-170">RESTful web API で、各コレクションには Web サービス内に独自の URIがあり、コレクションの URI に対して HTTP GET 要求を実行することで、そのコレクションの項目の一覧を取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-170">In a RESTful web API, each collection has its own URI within the web service, and performing an HTTP GET request over a URI for a collection retrieves a list of items in that collection.</span></span> <span data-ttu-id="b6d88-171">個々の項目にも独自の URI があり、アプリケーションはその URI を使用して別の HTTP GET 要求を送信することで、その項目の詳細を取得できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-171">Each individual item also has its own URI, and an application can submit another HTTP GET request using that URI to retrieve the details of that item.</span></span> <span data-ttu-id="b6d88-172">コレクションと項目の URI は階層的に整理してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-172">You should organize the URIs for collections and items in a hierarchical manner.</span></span> <span data-ttu-id="b6d88-173">電子商取引システムでは、URI */customers* は顧客のコレクションを表し、*/customers/5* はこのコレクションから ID 5 の単一顧客に関する詳細を取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-173">In the ecommerce system, the URI */customers* denotes the customer’s collection, and */customers/5* retrieves the details for the single customer with the ID 5 from this collection.</span></span> <span data-ttu-id="b6d88-174">この方法は、Web API を直感的に保つのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-174">This approach helps to keep the web API intuitive.</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-175">URI で一貫性のある命名規則を使用します。一般的に、コレクションを参照する URI で複数形名詞を使用するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-175">Adopt a consistent naming convention in URIs; in general it helps to use plural nouns for URIs that reference collections.</span></span>
>
>

<span data-ttu-id="b6d88-176">異なる種類のリソース間のリレーションシップ、およびこの関連を公開する方法を検討する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-176">You also need to consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="b6d88-177">たとえば、顧客の発注が 0 またはそれ以上である場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-177">For example, customers may place zero or more orders.</span></span> <span data-ttu-id="b6d88-178">このリレーションシップを表す自然な方法は、*/customers/5/orders* などの URI より顧客 5 のすべての注文を検索することです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-178">A natural way to represent this relationship would be through a URI such as */customers/5/orders* to find all the orders for customer 5.</span></span> <span data-ttu-id="b6d88-179">注文 99 の顧客を検索するには、*/orders/99/customer* などの URI より、注文から特定の顧客への逆の関連を表すことも検討できます。とはいえ、このモデルをあまりにも拡張すると、実装が煩雑になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-179">You might also consider representing the association from an order back to a specific customer through a URI such as */orders/99/customer* to find the customer for order 99, but extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="b6d88-180">より優れたソリューションは、注文のクエリを実行するときに返される HTTP 応答メッセージの本文に、顧客など関連するリソースに誘導できるリンクを記載することです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-180">A better solution is to provide navigable links to associated resources, such as the customer, in the body of the HTTP response message returned when the order is queried.</span></span> <span data-ttu-id="b6d88-181">このメカニズムの詳細については、本ガイダンスの「関連リソースへのナビゲーションを可能にする HATEOAS アプローチの使用」のセクションで後に説明します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-181">This mechanism is described in more detail in the section Using the HATEOAS Approach to Enable Navigation To Related Resources later in this guidance.</span></span>

<span data-ttu-id="b6d88-182">より複雑なシステムでは、さらに多くの種類のエンティティが存在する場合があり、クライアント アプリケーションがさまざまなレベルのリレーションシップをナビゲートするのを可能にする URI を提供するように促すことができます。たとえば、*/customers/1/orders/99/products* は顧客 1 が発注した注文 99 の製品リストを取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-182">In more complex systems there may be many more types of entity, and it can be tempting to provide URIs that enable a client application to navigate through several levels of relationships, such as */customers/1/orders/99/products* to obtain the list of products in order 99 placed by customer 1.</span></span> <span data-ttu-id="b6d88-183">ただし、将来的にリソース間のリレーションシップが変わる場合、このレベルの複雑さを維持するのは難しく、柔軟性がありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-183">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="b6d88-184">むしろ、URI を比較的シンプルに維持するようにします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-184">Rather, you should seek to keep URIs relatively simple.</span></span> <span data-ttu-id="b6d88-185">一度アプリケーションもリソースへの参照が備わると、この参照を使用してそのリソースに関連する項目を検索できることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-185">Bear in mind that once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="b6d88-186">上記のクエリは URI */customers/1/orders* と置き換えると顧客 1 のすべての注文を検索でき、URI */orders/99/products* のクエリを実行するとこの注文の製品を検索できます (注文 99 は顧客 1 により発注されたと仮定)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-186">The preceding query can be replaced with the URI */customers/1/orders* to find all the orders for customer 1, and then query the URI */orders/99/products* to find the products in this order (assuming order 99 was placed by customer 1).</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-187">"*コレクション/項目/コレクション*" よりも複雑なリソース URI を要求しないでください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-187">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>
>
>

<span data-ttu-id="b6d88-188">考慮すべき別の点は、すべての Web 要求は Web サーバーに負荷を与えるため、要求回数が多くなるほど負荷も大きくなることです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-188">Another point to consider is that all web requests impose a load on the web server, and the greater the number of requests the bigger the load.</span></span> <span data-ttu-id="b6d88-189">定義するリソースが、大量の小さなリソースを公開する「おしゃべりな」Web API にならないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-189">You should attempt to define your resources to avoid “chatty” web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="b6d88-190">そのような API では、クライアント アプリケーションが必要なすべてのデータを検索するために、複数の要求を送信する必要がある場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-190">Such an API may require a client application to submit multiple requests to find all the data that it requires.</span></span> <span data-ttu-id="b6d88-191">データを非正規化し、1 回の要求を発行して取得できる大きなリソースに関連情報を一緒に組み合わせることが有益な場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-191">It may be beneficial to denormalize data and combine related information together into bigger resources that can be retrieved by issuing a single request.</span></span> <span data-ttu-id="b6d88-192">ただし、このアプローチでは、クライアントが頻繁には必要としないデータ取得のオーバーヘッドに対してバランスを取る必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-192">However, you need to balance this approach against the overhead of fetching data that might not be frequently required by the client.</span></span> <span data-ttu-id="b6d88-193">ラージ オブジェクトを取得すると、追加データをあまり使用しない場合は要求の待機時間が延長し、追加の帯域幅コストが発生する可能性があり、メリットはほとんどありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-193">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs for little advantage if the additional data is not often used.</span></span>

<span data-ttu-id="b6d88-194">Web API と、構造、種類、または基となるデータ ソースの間の依存性を導入しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-194">Avoid introducing dependencies between the web API to the structure, type, or location of the underlying data sources.</span></span> <span data-ttu-id="b6d88-195">たとえば、データがリレーショナル データベースにある場合、Web API はそれぞれのテーブルをリソースのコレクションとして公開する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-195">For example, if your data is located in a relational database, the web API does not need to expose each table as a collection of resources.</span></span> <span data-ttu-id="b6d88-196">Web API をデータベースの抽象化と捉え、必要に応じてデータベースと Web API 間のマッピング レイヤーを導入します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-196">Think of the web API as an abstraction of the database, and if necessary introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="b6d88-197">この方法により、データベースの設計や実装が変更されても (正規化されたテーブルのコレクションを含むリレーショナル データベースから、ドキュメント データベースなどの非正規化された NoSQL ストレージ システムへの移行など)、クライアント アプリケーションはこれらの変更による影響を受けません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-197">In this way, if the design or implementation of the database changes (for example, you move from a relational database containing a collection of normalized tables to a denormalized NoSQL storage system such as a document database) client applications are insulated from these changes.</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-198">Web API を支えるデータのソースがデータ ストアである必要はありません。別のサービスや基幹業務アプリケーション、または組織内でオンプレミスで実行している従来のアプリケーションの場合さえあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-198">The source of the data that underpins a web API does not have to be a data store; it could be another service or line-of-business application or even a legacy application running on-premises within an organization.</span></span>
>
>

<span data-ttu-id="b6d88-199">最後に、Web API により実装されるすべての操作を特定のリソースにマッピングできない場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-199">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="b6d88-200">1 つの機能を呼び出し、結果を HTTP 応答メッセージとして返す HTTP GET 応答により、そのような "*非リソース*" シナリオを処理できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-200">You can handle such *non-resource* scenarios through HTTP GET requests that invoke a piece of functionality and return the results as an HTTP response message.</span></span> <span data-ttu-id="b6d88-201">加算や減算などの単純な電卓スタイルの操作を実装する Web API は、これらの操作を疑似リソースとして公開し、クエリ文字列を使用して必要なパラメーターを指定する URI を提供できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-201">A web API that implements simple calculator-style operations such as add and subtract could provide URIs that expose these operations as pseudo resources and utilize the query string to specify the parameters required.</span></span> <span data-ttu-id="b6d88-202">たとえば、URI */add?operand1=99&operand2=1* に対する GET 要求により、値 100 を含む本文の応答メッセージを返し、URI */subtract?operand1=50&operand2=20* に対する GET 要求により、値 30 を含む本文の応答メッセージを返すことができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-202">For example a GET request to the URI */add?operand1=99&operand2=1* could return a response message with the body containing the value 100, and GET request to the URI */subtract?operand1=50&operand2=20* could return a response message with the body containing the value 30.</span></span> <span data-ttu-id="b6d88-203">ただし、これらの形式の URI は常に慎重に使用してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-203">However, only use these forms of URIs sparingly.</span></span>

### <a name="defining-operations-in-terms-of-http-methods"></a><span data-ttu-id="b6d88-204">HTTP メソッドに関する操作の定義</span><span class="sxs-lookup"><span data-stu-id="b6d88-204">Defining operations in terms of HTTP methods</span></span>
<span data-ttu-id="b6d88-205">HTTP プロトコルにより、要求にセマンティックな意味を割り当てる多くのメソッドが定義されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-205">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="b6d88-206">ほとんどの RESTful Web API により使用される一般的な HTTP メソッドは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-206">The common HTTP methods used by most RESTful web APIs are:</span></span>

* <span data-ttu-id="b6d88-207">**GET**、指定した URI のリソースのコピーを取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-207">**GET**, to retrieve a copy of the resource at the specified URI.</span></span> <span data-ttu-id="b6d88-208">応答メッセージの本文には、要求されたリソースの詳細が含まれています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-208">The body of the response message contains the details of the requested resource.</span></span>
* <span data-ttu-id="b6d88-209">**POST**、指定された URI で新しいリソースを作成します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-209">**POST**, to create a new resource at the specified URI.</span></span> <span data-ttu-id="b6d88-210">応答メッセージの本文には、新しいリソースの詳細が記載されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-210">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="b6d88-211">POST は実際にはリソースを作成しない操作をトリガーするのにも使用できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-211">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
* <span data-ttu-id="b6d88-212">**PUT**、指定した URI のリソースを置き換える、または更新します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-212">**PUT**, to replace or update the resource at the specified URI.</span></span> <span data-ttu-id="b6d88-213">応答メッセージの本文では、変更するリソースおよび適用する値が指定されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-213">The body of the request message specifies the resource to be modified and the values to be applied.</span></span>
* <span data-ttu-id="b6d88-214">**DELETE**、指定した URI のリソースを削除します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-214">**DELETE**, to remove the resource at the specified URI.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-215">HTTP プロトコルではそれほど一般的に使用されない他のメソッドも定義されます。たとえば、リソースへの選択的更新を要求するのに使用される PATCH、リソースの説明を要求するのに使用される HEAD、サーバーによりサポートされる通信オプションについての情報をクライアント情報が取得できるようにする OPTIONS、テストや診断目的で使用可能な情報をクライアントが要求できるようにする TRACE などです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-215">The HTTP protocol also defines other less commonly-used methods, such as PATCH which is used to request selective updates to a resource, HEAD which is used to request a description of a resource, OPTIONS which enables a client information to obtain information about the communication options supported by the server, and TRACE which allows a client to request information that it can use for testing and diagnostics purposes.</span></span>
>
>

<span data-ttu-id="b6d88-216">特定の要求に対する影響は、適用されるリソースがコレクションまたは個別 の項目であるかによって異なります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-216">The effect of a specific request should depend on whether the resource to which it is applied is a collection or an individual item.</span></span> <span data-ttu-id="b6d88-217">以下の表では、電子商取引の例を使用しながら、ほとんどの RESTful の実装で使用される一般的な規則をまとめています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-217">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="b6d88-218">これらの要求がすべて実装されない場合があります。具体的なシナリオにより異なります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-218">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="b6d88-219">**リソース**</span><span class="sxs-lookup"><span data-stu-id="b6d88-219">**Resource**</span></span> | <span data-ttu-id="b6d88-220">**POST**</span><span class="sxs-lookup"><span data-stu-id="b6d88-220">**POST**</span></span> | <span data-ttu-id="b6d88-221">**GET**</span><span class="sxs-lookup"><span data-stu-id="b6d88-221">**GET**</span></span> | <span data-ttu-id="b6d88-222">**PUT**</span><span class="sxs-lookup"><span data-stu-id="b6d88-222">**PUT**</span></span> | <span data-ttu-id="b6d88-223">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="b6d88-223">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="b6d88-224">/customers</span><span class="sxs-lookup"><span data-stu-id="b6d88-224">/customers</span></span> |<span data-ttu-id="b6d88-225">新しい顧客を作成</span><span class="sxs-lookup"><span data-stu-id="b6d88-225">Create a new customer</span></span> |<span data-ttu-id="b6d88-226">すべての顧客を取得</span><span class="sxs-lookup"><span data-stu-id="b6d88-226">Retrieve all customers</span></span> |<span data-ttu-id="b6d88-227">顧客を一括更新 ("*実装されている場合*")</span><span class="sxs-lookup"><span data-stu-id="b6d88-227">Bulk update of customers (*if implemented*)</span></span> |<span data-ttu-id="b6d88-228">すべての顧客を削除</span><span class="sxs-lookup"><span data-stu-id="b6d88-228">Remove all customers</span></span> |
| <span data-ttu-id="b6d88-229">/customers/1</span><span class="sxs-lookup"><span data-stu-id="b6d88-229">/customers/1</span></span> |<span data-ttu-id="b6d88-230">エラー</span><span class="sxs-lookup"><span data-stu-id="b6d88-230">Error</span></span> |<span data-ttu-id="b6d88-231">顧客 1 の詳細を取得</span><span class="sxs-lookup"><span data-stu-id="b6d88-231">Retrieve the details for customer 1</span></span> |<span data-ttu-id="b6d88-232">存在する場合に顧客 1 の詳細を更新。その他の場合はエラーを返す</span><span class="sxs-lookup"><span data-stu-id="b6d88-232">Update the details of customer 1 if it exists, otherwise return an error</span></span> |<span data-ttu-id="b6d88-233">顧客 1 を削除</span><span class="sxs-lookup"><span data-stu-id="b6d88-233">Remove customer 1</span></span> |
| <span data-ttu-id="b6d88-234">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="b6d88-234">/customers/1/orders</span></span> |<span data-ttu-id="b6d88-235">顧客 1 の新しい注文を作成</span><span class="sxs-lookup"><span data-stu-id="b6d88-235">Create a new order for customer 1</span></span> |<span data-ttu-id="b6d88-236">顧客 1 のすべての注文を取得</span><span class="sxs-lookup"><span data-stu-id="b6d88-236">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="b6d88-237">顧客 1 の注文を一括更新 ("*実装されている場合*")</span><span class="sxs-lookup"><span data-stu-id="b6d88-237">Bulk update of orders for customer 1 (*if implemented*)</span></span> |<span data-ttu-id="b6d88-238">顧客 1 のすべての注文を削除 ("*実装されている場合*")</span><span class="sxs-lookup"><span data-stu-id="b6d88-238">Remove all orders for customer 1(*if implemented*)</span></span> |

<span data-ttu-id="b6d88-239">GET および DELETE 要求の目的は比較的単純ですが、POST および PUT 要求の目的および影響については混乱するかもしれません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-239">The purpose of GET and DELETE requests are relatively straightforward, but there is scope for confusion concerning the purpose and effects of POST and PUT requests.</span></span>

<span data-ttu-id="b6d88-240">POST 要求は、要求の本文に含まれるデータで新しいリソースを作成します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-240">A POST request should create a new resource with data provided in the body of the request.</span></span> <span data-ttu-id="b6d88-241">REST モデルでは、コレクションであるリソースに POST 要求を適用することがよくあります。新しいリソースがコレクションに追加されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-241">In the REST model, you frequently apply POST requests to resources that are collections; the new resource is added to the collection.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-242">一部の機能をトリガーする (また、必ずしもデータを返さない) POST 要求を定義することもでき、これらの種類の要求はコレクションに適用できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-242">You can also define POST requests that trigger some functionality (and that don't necessarily return data), and these types of request can be applied to collections.</span></span> <span data-ttu-id="b6d88-243">たとえば POST 要求を使用することで、給与支払い処理サービスにタイムシートを渡し、計算した税金が応答として返されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-243">For example you could use a POST request to pass a timesheet to a payroll processing service and get the calculated taxes back as a response.</span></span>
>
>

<span data-ttu-id="b6d88-244">PUT 要求は既存のリソースを変更します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-244">A PUT request is intended to modify an existing resource.</span></span> <span data-ttu-id="b6d88-245">指定されたリソースが存在しない場合、PUT 要求はエラーを返すことができます (場合によっては、実際にリソースを作成することがあります)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-245">If the specified resource does not exist, the PUT request could return an error (in some cases, it might actually create the resource).</span></span> <span data-ttu-id="b6d88-246">PUT 要求は、個々の項目 (特定の顧客や注文など) であるリソースに適用されることが最も多いとはいえ、実装の頻度は低いもののコレクションに適用できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-246">PUT requests are most frequently applied to resources that are individual items (such as a specific customer or order), although they can be applied to collections, although this is less-commonly implemented.</span></span> <span data-ttu-id="b6d88-247">PUT 要求はべき等ですが、POST 要求はそうでないことに注意してください。あるアプリケーションが同じ PUT 要求を複数回送信すると結果は常に同じになるはずですが (同じリソースが同じ値で変更される)、あるアプリケーションが同じ POST 要求を繰り返しても、複数のリソースが作成されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-247">Note that PUT requests are idempotent whereas POST requests are not; if an application submits the same PUT request multiple times the results should always be the same (the same resource will be modified with the same values), but if an application repeats the same POST request the result will be the creation of multiple resources.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-248">厳密に言えば、HTTP PUT 要求は既存のリソースを、要求の本文で指定されているリソースと置き換えます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-248">Strictly speaking, an HTTP PUT request replaces an existing resource with the resource specified in the body of the request.</span></span> <span data-ttu-id="b6d88-249">リソースのプロパティの選択を変更するものの、他のプロパティは変更しない場合は、HTTP PATCH 要求を使用して実装します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-249">If the intention is to modify a selection of properties in a resource but leave other properties unchanged, then this should be implemented by using an HTTP PATCH request.</span></span> <span data-ttu-id="b6d88-250">とはいえ、多くの RESTful の実装ではこの規則が緩和され、どちらの状況でも PUT が使用されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-250">However, many RESTful implementations relax this rule and use PUT for both situations.</span></span>
>
>

### <a name="processing-http-requests"></a><span data-ttu-id="b6d88-251">HTTP 要求の処理</span><span class="sxs-lookup"><span data-stu-id="b6d88-251">Processing HTTP requests</span></span>
<span data-ttu-id="b6d88-252">多くの HTTP 要求でクライアント アプリケーションにより含められるデータや、Web サーバーからの対応する応答メッセージは、さまざまな形式 (またはメディアの種類) で表示できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-252">The data included by a client application in many HTTP requests, and the corresponding response messages from the web server, could be presented in a variety of formats (or media types).</span></span> <span data-ttu-id="b6d88-253">たとえば、顧客または注文の詳細を指定するデータは、XML、JSON、または他のエンコードおよび圧縮された形式として提供できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-253">For example, the data that specifies the details for a customer or order could be provided as XML, JSON, or some other encoded and compressed format.</span></span> <span data-ttu-id="b6d88-254">要求を送信するクライアント アプリケーションの要求に従い、RESTful Web API は異なるメディアの種類をサポートします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-254">A RESTful web API should support different media types as requested by the client application that submits a request.</span></span>

<span data-ttu-id="b6d88-255">クライアント アプリケーションがメッセージの本文でデータを返す要求を送信する場合、要求の Accept ヘッダーで処理可能なメディアの種類を指定できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-255">When a client application sends a request that returns data in the body of a message, it can specify the media types it can handle in the Accept header of the request.</span></span> <span data-ttu-id="b6d88-256">次のコードは注文 2 の詳細を取得し、結果が JSON として返されることを要求する HTTP GET 要求を示しています (クライアントは引き続き、応答に含まれるデータのメディアの種類を確認し、返されるデータの形式を確認する必要があります)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-256">The following code illustrates an HTTP GET request that retrieves the details of order 2 and requests the result to be returned as JSON (the client should still examine the media type of the data in the response to verify the format of the data returned):</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="b6d88-257">Web サーバーがこのメディアの種類をサポートしている場合、メッセージの本文にデータの形式を指定する Content-Type ヘッダーを含む応答により、応答できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-257">If the web server supports this media type, it can reply with a response that includes Content-Type header that specifies the format of the data in the body of the message:</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-258">相互運用性を最大にするため、Accept および Content-Type ヘッダーで参照されるメディアの種類は、一部のカスタム メディアの種類ではなく MIME の種類として認識される必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-258">For maximum interoperability, the media types referenced in the Accept and Content-Type headers should be recognized MIME types rather than some custom media type.</span></span>
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

<span data-ttu-id="b6d88-259">Web サーバーが要求されたメディアの種類をサポートしていない場合、異なる形式でデータを送信できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-259">If the web server does not support the requested media type, it can send the data in a different format.</span></span> <span data-ttu-id="b6d88-260">すべての場合で、Content-Type ヘッダーでメディアの種類を指定する必要があります (*application/json* など)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-260">IN all cases it must specify the media type (such as *application/json*) in the Content-Type header.</span></span> <span data-ttu-id="b6d88-261">応答メッセージを解析し、メッセージ本文内の結果を適切に解釈することはクライアント アプリケーションの役割です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-261">It is the responsibility of the client application to parse the response message and interpret the results in the message body appropriately.</span></span>

<span data-ttu-id="b6d88-262">この例では、Web サーバーが要求されたデータを正常に取得し、応答ヘッダーで状態コード 200 を渡すことで成功を示しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-262">Note that in this example, the web server successfully retrieves the requested data and indicates success by passing back a status code of 200 in the response header.</span></span> <span data-ttu-id="b6d88-263">一致するデータが見つからない場合は、代わりに状態コード 404 (見つかりません) が返され、応答メッセージの本文には追加情報を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-263">If no matching data is found, it should instead return a status code of 404 (not found) and the body of the response message can contain additional information.</span></span> <span data-ttu-id="b6d88-264">この情報の形式は、次の例で示されるように Content-Type ヘッダーにより指定されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-264">The format of this information is specified by the Content-Type header, as shown in the following example:</span></span>

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="b6d88-265">注文 222 が存在しないため、応答メッセージは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-265">Order 222 does not exist, so the response message looks like this:</span></span>

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

<span data-ttu-id="b6d88-266">アプリケーションがリソースを更新するために HTTP PUT 要求を送信すると、リソースの URI が指定され、要求メッセージの本文で変更されるデータが提供されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-266">When an application sends an HTTP PUT request to update a resource, it specifies the URI of the resource and provides the data to be modified in the body of the request message.</span></span> <span data-ttu-id="b6d88-267">Content-Type ヘッダーを使用して、このデータの形式を指定する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-267">It should also specify the format of this data by using the Content-Type header.</span></span> <span data-ttu-id="b6d88-268">テキスト ベースの情報で使用される一般的な形式は *application/x-www-form-urlencoded* で、& の文字で区切られた一連の名前/値のペアで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-268">A common format used for text-based information is *application/x-www-form-urlencoded*, which comprises a set of name/value pairs separated by the & character.</span></span> <span data-ttu-id="b6d88-269">次の例は、注文 1 の情報を変更する HTTP PUT 要求を示しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-269">The next example shows an HTTP PUT request that modifies the information in order 1:</span></span>

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

<span data-ttu-id="b6d88-270">正常に変更が行われる場合、プロセスが正常に処理されたものの、応答本文にはそれ以上の情報が含まれないことを示す HTTP 204 状態コードで応答するのが理想的です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-270">If the modification is successful, it should ideally respond with an HTTP 204 status code, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="b6d88-271">応答の Location ヘッダーには、新しく更新されたリソースの URI が含まれています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-271">The Location header in the response contains the URI of the newly updated resource:</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> <span data-ttu-id="b6d88-272">HTTP PUT 要求メッセージ内のデータに日付および時刻の情報が含まれる場合、お使いの Web サービスが、ISO 8601 標準に従って書式設定された日付および時刻を受け入れるようにしてください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-272">If the data in an HTTP PUT request message includes date and time information, make sure that your web service accepts dates and times formatted following the ISO 8601 standard.</span></span>
>
>

<span data-ttu-id="b6d88-273">更新するリソースが存在しない場合、Web サーバーは前に説明したように、Not Found 応答により応答できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-273">If the resource to be updated does not exist, the web server can respond with a Not Found response as described earlier.</span></span> <span data-ttu-id="b6d88-274">または、サーバーが実際にはオブジェクト自体を作成する場合、状態コード HTTP 200 (OK) または HTTP 201 (Created) が返され、応答本文には新しいリソースのデータが含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-274">Alternatively, if the server actually creates the object itself it could return the status codes HTTP 200 (OK) or HTTP 201 (Created) and the response body could contain the data for the new resource.</span></span> <span data-ttu-id="b6d88-275">要求の Content-Type ヘッダーが Web サーバーで処理できないデータ形式を指定する場合は、HTTP 状態コード 415 (Unsupported Media Type) で応答します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-275">If the Content-Type header of the request specifies a data format that the web server cannot handle, it should respond with HTTP status code 415 (Unsupported Media Type).</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-276">コレクションの複数のリソースの更新をバッチ処理できる一括 HTTP PUT 操作の実装を検討してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-276">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="b6d88-277">PUT 要求はコレクションの URI を指定し、要求本文は変更するリソースの詳細を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-277">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="b6d88-278">この方法によりおしゃべりを減らし、パフォーマンスを向上させることができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-278">This approach can help to reduce chattiness and improve performance.</span></span>
>
>

<span data-ttu-id="b6d88-279">新しいリソースを作成する HTTP POST 要求の形式は PUT 要求と類似しています。メッセージ本文に、追加する新しいリソースの詳細が含まれています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-279">The format of an HTTP POST requests that create new resources are similar to those of PUT requests; the message body contains the details of the new resource to be added.</span></span> <span data-ttu-id="b6d88-280">ただし、URI は通常、リソースの追加先となるコレクションを指定します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-280">However, the URI typically specifies the collection to which the resource should be added.</span></span> <span data-ttu-id="b6d88-281">以下の例では新しい注文を作成し、注文コレクションにそれを追加しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-281">The following example creates a new order and adds it to the orders collection:</span></span>

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

<span data-ttu-id="b6d88-282">要求が成功する場合、Web サーバーは HTTP 状態コード 201 (Created) を含むメッセージ コードで応答します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-282">If the request is successful, the web server should respond with a message code with HTTP status code 201 (Created).</span></span> <span data-ttu-id="b6d88-283">Location ヘッダーには新しく作成されたリソースの URI が含まれ、応答の本文には新しいリソースのコピーが含まれます。Content-Type ヘッダーは次のデータの形式を指定します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-283">The Location header should contain the URI of the newly created resource, and the body of the response should contain a copy of the new resource; the Content-Type header specifies the format of this data:</span></span>

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> <span data-ttu-id="b6d88-284">PUT または POST 要求によって提供されたデータが無効な場合、Web サーバーは HTTP 状態コード 400 (Bad Request) を含むメッセージで応答します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-284">If the data provided by a PUT or POST request is invalid, the web server should respond with a message with HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="b6d88-285">このメッセージの本文には、要求の問題および想定される形式に関する追加情報を含めるか、詳細が記載された URL のリンクを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-285">The body of this message can contain additional information about the problem with the request and the formats expected, or it can contain a link to a URL that provides more details.</span></span>
>
>

<span data-ttu-id="b6d88-286">リソースを削除するには、HTTP DELETE 要求によって、削除するリソースの URI を単に提供します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-286">To remove a resource, an HTTP DELETE request simply provides the URI of the resource to be deleted.</span></span> <span data-ttu-id="b6d88-287">次の例では注文 99 を削除しようとしています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-287">The following example attempts to remove order 99:</span></span>

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

<span data-ttu-id="b6d88-288">削除操作が成功すると、Web サーバーは HTTP 状態コード 204 で応答します。このコードは、プロセスが正常に処理されたものの、応答本文にはそれ以上の情報が含まれないことを示します (これは正常な PUT 操作によって返されるのと同じ応答ですが、リソースがもう存在しないため Location ヘッダーがありません)。削除が非同期的に実行される場合、DELETE 要求で HTTP 状態コード 200 (OK) または 202 (Accepted) を返すこともできます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-288">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information (this is the same response returned by a successful PUT operation, but without a Location header as the resource no longer exists.) It is also possible for a DELETE request to return HTTP status code 200 (OK) or 202 (Accepted) if the deletion is performed asynchronously.</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

<span data-ttu-id="b6d88-289">リソースが見つからない場合、Web サーバーは代わりに 404 (Not Found) メッセージを返します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-289">If the resource is not found, the web server should return a 404 (Not Found) message instead.</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-290">コレクションに含まれるすべてのリソースを削除する必要がある場合は、強制的にアプリケーションでコレクションから各リソースを順番に削除するのではなく、コレクションの URI を指定した HTTP DELETE 要求を有効にします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-290">If all the resources in a collection need to be deleted, enable an HTTP DELETE request to be specified for the URI of the collection rather than forcing an application to remove each resource in turn from the collection.</span></span>
>
>

### <a name="filtering-and-paginating-data"></a><span data-ttu-id="b6d88-291">データのフィルター処理と改ページ</span><span class="sxs-lookup"><span data-stu-id="b6d88-291">Filtering and paginating data</span></span>
<span data-ttu-id="b6d88-292">URI をシンプルで直感的に保つように努める必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-292">You should endeavor to keep the URIs simple and intuitive.</span></span> <span data-ttu-id="b6d88-293">この点で単一の URI を使用してリソースのコレクションを公開することは役に立ちますが、情報のサブセットしか必要ではないときに、アプリケーションが大量のデータを取得する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-293">Exposing a collection of resources through a single URI assists in this respect, but it can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="b6d88-294">膨大な量のトラフィックを生成することは Web サーバーのパフォーマンスおよび拡張性に影響を与えるだけでなく、データを要求しているクライアント アプリケーションの応答性にも悪影響を与えます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-294">Generating a large volume of traffic impacts not only the performance and scalability of the web server but also adversely affect the responsiveness of client applications requesting the data.</span></span>

<span data-ttu-id="b6d88-295">たとえば、注文に、注文に対して支払った料金が含まれている場合、特定の値に対してコストがかかるすべての注文を取得する必要があるクライアント アプリケーションは、*/orders* URI からすべての注文を取得し、これらの注文をローカルでフィルター処理する必要がある場合があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-295">For example, if orders contain the price paid for the order, a client application that needs to retrieve all orders that have a cost over a specific value might need to retrieve all orders from the */orders* URI and then filter these orders locally.</span></span> <span data-ttu-id="b6d88-296">明らかにこのプロセスの効率はあまり良くありません。Web API をホストするサーバー上のネットワーク帯域幅および処理能力を無駄にします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-296">Clearly this process is highly inefficient; it wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="b6d88-297">1 つの解決策は */orders/ordervalue_greater_than_n* などの URI スキームを使用することで、*n* は注文価格です。とはいえ、この方法は限られた料金を除きどの料金でも現実的ではありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-297">One solution may be to provide a URI scheme such as */orders/ordervalue_greater_than_n* where *n* is the order price, but for all but a limited number of prices such an approach is impractical.</span></span> <span data-ttu-id="b6d88-298">さらに、他の条件に基づいて注文のクエリを実行する必要がある場合、直感的でない可能性がある名前が記載された長い URI リストが最終的に提供されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-298">Additionally, if you need to query orders based on other criteria, you can end up being faced with providing with a long list of URIs with possibly non-intuitive names.</span></span>

<span data-ttu-id="b6d88-299">データをフィルター処理するより良い方法は、*/orders?ordervaluethreshold=n* など、Web API に渡されるクエリ文字列でフィルター条件を指定することです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-299">A better strategy to filtering data is to provide the filter criteria in the query string that is passed to the web API, such as */orders?ordervaluethreshold=n*.</span></span> <span data-ttu-id="b6d88-300">この例では、Web API の対応する操作はクエリ文字列の `ordervaluethreshold` パラメーターの解析および処理を行い、HTTP 応答でフィルター処理された結果を返します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-300">In this example, the corresponding operation in the web API is responsible for parsing and handling the `ordervaluethreshold` parameter in the query string and returning the filtered results in the HTTP response.</span></span>

<span data-ttu-id="b6d88-301">コレクション リソースに対する一部の単純な HTTP GET 要求では、多数の項目が返される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-301">Some simple HTTP GET requests over collection resources could potentially return a large number of items.</span></span> <span data-ttu-id="b6d88-302">これが起こる可能性を解決するため、任意の単一な要求により返されるデータの量を制限するように Web API を設計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-302">To combat the possibility of this occurring you should design the web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="b6d88-303">これを実現するには、取得する項目の最大数 (それ自体が、サービス拒否の攻撃を防ぐのに役立つ上限に依存) と、コレクションへの開始オフセットをユーザーが指定できるようにするクエリ文字列をサポートします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-303">You can achieve this by supporting query strings that enable the user to specify the maximum number of items to be retrieved (which could itself be subject to an upperbound limit to help prevent Denial of Service attacks), and a starting offset into the collection.</span></span> <span data-ttu-id="b6d88-304">たとえば、URI */orders?limit=25&offset=50* のクエリ文字列は、注文コレクションで見つかる 50 番目の注文から始まる 25 件の注文を取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-304">For example, the query string in the URI */orders?limit=25&offset=50* should retrieve 25 orders starting with the 50th order found in the orders collection.</span></span> <span data-ttu-id="b6d88-305">データのフィルター処理と同様、Web API で GET 要求を実装する操作は、クエリ文字列の `limit` および `offset` パラメーターの解析および処理を担います。</span><span class="sxs-lookup"><span data-stu-id="b6d88-305">As with filtering data, the operation that implements the GET request in the web API is responsible for parsing and handling the `limit` and `offset` parameters in the query string.</span></span> <span data-ttu-id="b6d88-306">クライアント アプリケーションを支援するには、改ページ調整されたデータを返す GET 要求に、コレクションで利用できるリソースの合計を示す何らかの形式のメタデータも含まれるようにします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-306">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span> <span data-ttu-id="b6d88-307">また、他のインテリジェントなページング方法を検討することもできます。詳細については、「[API Design Notes: Smart Paging (API の設計に関する注記: スマート ページング)](http://bizcoder.com/api-design-notes-smart-paging)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-307">You might also consider other intelligent paging strategies; for more information, see [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)</span></span>

<span data-ttu-id="b6d88-308">データを取得時に並べ替える場合にも同様の戦略に従うことができます。*/orders?sort=ProductID* など、フィールド名を値として使用する並べ替えパラメーターを指定できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-308">You can follow a similar strategy for sorting data as it is fetched; you could provide a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="b6d88-309">とはいえ、この方法はキャッシュに有害な影響を与える可能性があることに注意してください (クエリ文字列パラメーターは、キャッシュされたデータのキーとして多くのキャッシュ実装により使用されるリソース識別子の一部を形成します)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-309">However, note that this approach can have a deleterious effect on caching (query string parameters form part of the resource identifier used by many cache implementations as the key to cached data).</span></span>

<span data-ttu-id="b6d88-310">単一のリソース項目に大量のデータが含まれる場合、返されるフィールドを制限 (計画) するように、この方法を拡張できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-310">You can extend this approach to limit (project) the fields returned if a single resource item contains a large amount of data.</span></span> <span data-ttu-id="b6d88-311">たとえば、*/orders?fields=ProductID,Quantity* など、コンマ区切りのフィールド一覧を受け取るクエリ文字列パラメーターを使用することができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-311">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span>

> [!TIP]
> <span data-ttu-id="b6d88-312">クエリ文字列内の省略可能なすべてのパラメーターに、意味のある既定値を設定します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-312">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="b6d88-313">たとえば、改ページ調整を実装する場合は `limit` パラメーターを 10、`offset` パラメーターを 0 に設定し、順序付けを実装する場合はリソースのキーに並べ替えパラメーターを設定し、プロジェクションをサポートする場合は`fields` パラメーターをリソースのすべてのフィールドに設定します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-313">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>
>
>

### <a name="handling-large-binary-resources"></a><span data-ttu-id="b6d88-314">大きなバイナリ  リソースの処理</span><span class="sxs-lookup"><span data-stu-id="b6d88-314">Handling large binary resources</span></span>
<span data-ttu-id="b6d88-315">単一のリソースに、ファイルやイメージなど、大きなバイナリ フィールドが含まれている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-315">A single resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="b6d88-316">信頼性が低く断続的な接続であることが原因となっている伝送の問題を克服し、応答時間を向上させるには、そのようなリソースをクライアント アプリケーションがチャンクで取得できるようにする操作を提供することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-316">To overcome the transmission problems caused by unreliable and intermittent connections and to improve response times, consider providing operations that enable such resources to be retrieved in chunks by the client application.</span></span> <span data-ttu-id="b6d88-317">そのためには、Web API が大きなリソースの GET 要求で Accept-Ranges ヘッダーをサポートし、理想的にはこれらのリソースの HTTP HEAD 要求を実装するようにします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-317">To do this, the web API should support the Accept-Ranges header for GET requests for large resources, and ideally implement HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="b6d88-318">Accept-Ranges ヘッダーは、GET 操作が一部の結果をサポートし、クライアント アプリケーションがバイト数の範囲として指定されたリソースのサブセットを返す GET 要求を送信できることを示します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-318">The Accept-Ranges header indicates that the GET operation supports partial results, and that a client application can submit GET requests that return a subset of a resource specified as a range of bytes.</span></span> <span data-ttu-id="b6d88-319">HEAD 要求は GET 要求に似ていますが、リソースおよび空のメッセージ本文を記述するヘッダーのみを返す点が異なります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-319">A HEAD request is similar to a GET request except that it only returns a header that describes the resource and an empty message body.</span></span> <span data-ttu-id="b6d88-320">クライアント アプリケーションは HEAD 要求を発行し、部分的 GET 要求を使用して、リソースを取得するかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-320">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="b6d88-321">次の例は、製品イメージに関する情報を取得する HEAD 要求を示しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-321">The following example shows a HEAD request that obtains information about a product image:</span></span>

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

<span data-ttu-id="b6d88-322">応答メッセージには、リソースのサイズ  (4,580 バイト)  が含まれるヘッダー、および対応する GET 操作が部分的な結果をサポートする Accept-Ranges ヘッダーが含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-322">The response message contains a header that includes the size of the resource (4580 bytes), and the Accept-Ranges header that the corresponding GET operation supports partial results:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

<span data-ttu-id="b6d88-323">クライアント アプリケーションはこの情報を使用して、より小さいチャンクのイメージを取得する一連の GET 要求を作成できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-323">The client application can use this information to construct a series of GET operations to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="b6d88-324">最初の要求では、Range ヘッダーを使用して最初の 2,500 バイトを取得します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-324">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

<span data-ttu-id="b6d88-325">応答メッセージは、HTTP 状態コード 206 を返すことで、これが部分的な応答であることを示します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-325">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="b6d88-326">Content-Length ヘッダーはメッセージ本文で返される実際のバイト数を指定し (リソースのサイズではない)、Content-Range ヘッダーはこれがどの部分のリソースであるかを示します (4,580 のうち 0 ～ 2499 バイト):</span><span class="sxs-lookup"><span data-stu-id="b6d88-326">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

<span data-ttu-id="b6d88-327">クライアント アプリケーションのその後の要求により、適切な Range ヘッダーを使用することで残りのリソースを取得できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-327">A subsequent request from the client application can retrieve the remainder of the resource by using an appropriate Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

<span data-ttu-id="b6d88-328">対応する結果メッセージは、次のように表示されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-328">The corresponding result message should look like this:</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a><span data-ttu-id="b6d88-329">関連リソースへのナビゲーションを可能にする HATEOAS アプローチの使用</span><span class="sxs-lookup"><span data-stu-id="b6d88-329">Using the HATEOAS approach to enable navigation to related resources</span></span>
<span data-ttu-id="b6d88-330">REST の背後にある主な動機の 1 つは、URI スキームの事前知識を必要とせずに、リソースのセット全体を移動できることです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-330">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="b6d88-331">各 HTTP GET 要求は応答に含まれるハイパーリンクより、要求したオブジェクトに直接関連するリソースを検索するのに必要な情報を返し、これらの各リソースで使用可能な操作を記述する情報も提供されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-331">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="b6d88-332">この原則は HATEOAS、つまり アプリケーション状態のエンジンとしてのハイパーテキストとして知られます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-332">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="b6d88-333">システムは効率的に Finite State Machine であり、各要求への応答には、ある状態を別の状態に移すのに必要な情報が含まれています。他の情報は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-333">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-334">現在、HATEOAS の原則をモデル化する方法を定義する標準や仕様はありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-334">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="b6d88-335">このセクションで示されている例は、可能性のある一つのソリューションを示しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-335">The examples shown in this section illustrate one possible solution.</span></span>
>
>

<span data-ttu-id="b6d88-336">例として、顧客と注文のリレーションシップを処理するには、特定の注文の応答で返されるデータに、注文を行った顧客を識別するハイパーリンク形式の URI、およびその顧客に対して実行できる操作 を含めます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-336">As an example, to handle the relationship between customers and orders, the data returned in the response for a specific order should contain URIs in the form of a hyperlink identifying the customer that placed the order, and the operations that can be performed on that customer.</span></span>

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

<span data-ttu-id="b6d88-337">応答メッセージの本文には、リレーションシップの性質を指定する `links` 配列 (コード例で強調表示されている) (*Customer*)、顧客の URI (*http://adventure-works.com/customers/3*)、この顧客の詳細を取得する方法 (*GET*)、この情報を取得するのに Web サーバーがサポートする MIME の種類 (*text/xml* と *application/json*) が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-337">The body of the response message contains a `links` array (highlighted in the code example) that specifies the nature of the relationship (*Customer*), the URI of the customer (*http://adventure-works.com/customers/3*), how to retrieve the details of this customer (*GET*), and the MIME types that the web server supports for retrieving this information (*text/xml* and *application/json*).</span></span> <span data-ttu-id="b6d88-338">クライアント アプリケーションが顧客の詳細を取得するのに必要な情報はこれですべてです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-338">This is all the information that a client application needs to be able to fetch the details of the customer.</span></span> <span data-ttu-id="b6d88-339">さらに Links アレイには、PUT (Web サーバーがクライアントに提供を期待する形式とともに顧客を変更する) や DELETE など、実行可能な他の操作のリンクも含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-339">Additionally, the Links array also includes links for the other operations that can be performed, such as PUT (to modify the customer, together with the format that the web server expects the client to provide), and DELETE.</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

<span data-ttu-id="b6d88-340">完全を期すために、Links アレイには取得したリソースに関する自己参照型の情報も含めるようにします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-340">For completeness, the Links array should also include self-referencing information pertaining to the resource that has been retrieved.</span></span> <span data-ttu-id="b6d88-341">これらのリンクは、以前の例から省略されていますが、次のコードでは強調表示されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-341">These links have been omitted from the previous example, but are highlighted in the following code.</span></span> <span data-ttu-id="b6d88-342">これらのリンクで、これが操作により返されるリソースの参照であることを示すためにリレーションシップ *self* が使用されることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-342">Notice that in these links, the relationship *self* has been used to indicate that this is a reference to the resource being returned by the operation:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

<span data-ttu-id="b6d88-343">この方法を効率的にするために、クライアント アプリケーションでこの追加情報を取得および解析する準備を行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-343">For this approach to be effective, client applications must be prepared to retrieve and parse this additional information.</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="b6d88-344">RESTful Web API のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="b6d88-344">Versioning a RESTful web API</span></span>
<span data-ttu-id="b6d88-345">最も単純な状況を除き、すべての状況では Web API が静的であることはほとんどありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-345">It is highly unlikely that in all but the simplest of situations that a web API will remain static.</span></span> <span data-ttu-id="b6d88-346">ビジネスの要求は変化するため、新しいリソースのコレクションが加わり、リソース間のリレーションシップが変化し、リソース内のデータ構造が修正される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-346">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="b6d88-347">新しいまたは異なる要件を処理するために Web API を更新することは比較的簡単なプロセスですが、そのような変化が Web API を消費するクライアント アプリケーションに対して与える影響を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-347">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="b6d88-348">問題なのは、Web API を設計および実装している開発者はその API を完全に制御できるものの、リモートで操作しているサードパーティの組織により構築されたクライアント アプリケーションを、その開発者が同程度には制御できないことです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-348">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="b6d88-349">まず必要なのは、新しいクライアント アプリケーションが新しい機能やリソースを活用できるようにしながら、既存のアプリケーションに変更なく機能を続行させることです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-349">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="b6d88-350">バージョン管理により Web API は公開する機能およびリソースを示すことができ、クライアント アプリケーションは特定のバージョンの機能またはリソースへの要求を送信できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-350">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="b6d88-351">次のセクションではいくつかの方法について説明しますが、それぞれに独自の利点とトレードオフがあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-351">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="b6d88-352">バージョン管理なし</span><span class="sxs-lookup"><span data-stu-id="b6d88-352">No versioning</span></span>
<span data-ttu-id="b6d88-353">これは最も単純な方法で、一部の内部 API で許容されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-353">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="b6d88-354">大きな変更は新しいリソースまたは新しいリンクとして示されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-354">Big changes could be represented as new resources or new links.</span></span>  <span data-ttu-id="b6d88-355">既存のリソースにコンテンツを追加しても、このコンテンツの表示を想定していないクライアント アプリケーションはそれを無視するだけであるため、重大な変更はありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-355">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="b6d88-356">たとえば、URI *http://adventure-works.com/customers/3* への要求により、次のクライアント アプリケーションにより期待される `id`、`name`、および `address` フィールドを含む単一の顧客に関する詳細が返されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-356">For example, a request to the URI *http://adventure-works.com/customers/3* should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="b6d88-357">簡潔でわかりやすくするために、このセクションで示す応答例には HATEOAS リンクは含まれません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-357">For the purposes of simplicity and clarity, the example responses shown in this section do not include HATEOAS links.</span></span>
>
>

<span data-ttu-id="b6d88-358">`DateCreated` フィールドが顧客リソースのスキーマに追加される場合、応答は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-358">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="b6d88-359">既存のクライアント アプリケーションが認識されていないフィールドを無視できる場合は、正常に機能を続行する場合がありますが、新しいクライアント アプリケーションではこの新しいフィールドを処理するように設計できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-359">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="b6d88-360">とはいえ、リソースのスキーマにさらに重大な変更があるか (フィールドの削除や名前の変更など)、リソース間のリレーションシップが変更される場合、これらは既存のクライアント アプリケーションが正常に機能できなくなる重大な変更となる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-360">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="b6d88-361">このような状況では、次の方法のいずれかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-361">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="b6d88-362">URI のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="b6d88-362">URI versioning</span></span>
<span data-ttu-id="b6d88-363">Web API を変更するか、リソースのスキーマを変更するたびに、バージョン番号を各リソースの URI に追加します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-363">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="b6d88-364">既存の URI はこれまでと同様に動作を続け、元のスキーマに準拠するリソースを返します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-364">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="b6d88-365">前の例を拡張し、`address` フィールドをアドレスの各構成部分 (`streetAddress`、`city`、`state`、`zipCode` など) を含むサブフィールドに再構築する場合、このバージョンのリソースは http://adventure-works.com/v2/customers/3 などのバージョン番号を含む URI より公開できます: </span><span class="sxs-lookup"><span data-stu-id="b6d88-365">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as http://adventure-works.com/v2/customers/3:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="b6d88-366">このバージョン管理メカニズムは非常に単純ですが、適切なエンドポイントに要求をルーティングするサーバーにより変わります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-366">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="b6d88-367">とはいえ、Web API は何度も繰り返すことで成熟し、サーバーは多くの異なるバージョンをサポートする必要があるため、扱いにくくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-367">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="b6d88-368">また、純正主義者の観点からすると、すべての場合でクライアント アプリケーションは同じデータを取得しているため (顧客 3)、URI はバージョンによってそれほど異なるべきではありません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-368">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="b6d88-369">すべてのリンクで URI にバージョン番号が含まれる必要があるため、このスキームによっても HATEOAS の実装が複雑になります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-369">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="b6d88-370">クエリ文字列のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="b6d88-370">Query string versioning</span></span>
<span data-ttu-id="b6d88-371">複数の URI を提供するのではなく、*http://adventure-works.com/customers/3?version=2* など、HTTP 要求に追加されたクエリ文字列内のパラメーターを使用してリソースのバージョンを指定できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-371">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as *http://adventure-works.com/customers/3?version=2*.</span></span> <span data-ttu-id="b6d88-372">バージョン パラメーターがより古いクライアント アプリケーションで省略される場合、1 などの有効な値を既定にします。</span><span class="sxs-lookup"><span data-stu-id="b6d88-372">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="b6d88-373">この方法には同じリソースからは常に同じ URI が取得されるというセマンティックな利点がありますが、クエリ文字列を解析し、適切な HTTP 応答を返送する要求を処理するコードにより異なります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-373">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="b6d88-374">この方法は、URI のバージョン管理メカニズムとして HATEOAS を実装する同様の複雑さによっても影響されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-374">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-375">一部の古い Web ブラウザーおよび Web プロキシは、URL にクエリ文字列を含む要求の応答をキャッシュしません。</span><span class="sxs-lookup"><span data-stu-id="b6d88-375">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URL.</span></span> <span data-ttu-id="b6d88-376">これは、Web API を使用し、そのような Web ブラウザー内から実行する Web アプリケーションのパフォーマンスに悪影響を与える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-376">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>
>
>

### <a name="header-versioning"></a><span data-ttu-id="b6d88-377">ヘッダーのバージョン管理</span><span class="sxs-lookup"><span data-stu-id="b6d88-377">Header versioning</span></span>
<span data-ttu-id="b6d88-378">バージョン番号をクエリ文字列パラメーターとして追加するのではなく、リソースのバージョンを示すカスタム ヘッダーを実装できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-378">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="b6d88-379">この方法ではクライアント アプリケーションにより任意の要求に適切なヘッダーを追加する必要がありますが、バージョン ヘッダーが省略されている場合、クライアントの要求を処理するコードは既定値 (バージョン 1) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-379">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="b6d88-380">次の例では、*Custom-Header* という名前のカスタム ヘッダーを使用します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-380">The following examples utilize a custom header named *Custom-Header*.</span></span> <span data-ttu-id="b6d88-381">このヘッダーの値は、Web API のバージョンを示します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-381">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="b6d88-382">バージョン 1: </span><span class="sxs-lookup"><span data-stu-id="b6d88-382">Version 1:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="b6d88-383">バージョン 2: </span><span class="sxs-lookup"><span data-stu-id="b6d88-383">Version 2:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="b6d88-384">前の 2 つの方法と同様、HATEOAS の実装では任意のリンクに適切なカスタム ヘッダーを追加することが必要です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-384">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="b6d88-385">メディアの種類のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="b6d88-385">Media type versioning</span></span>
<span data-ttu-id="b6d88-386">クライアント アプリケーションが Web サーバーに HTTP GET 要求を送信する場合、このガイダンスで既に説明したように、Accept ヘッダーを使用して処理できるコンテンツの書式を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-386">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="b6d88-387">多くの場合、*Accept* ヘッダーの目的は、応答の本文が XML、JSON、またはクライアントが解析可能な他の一般的な形式であるかどうかをクライアント アプリケーションが指定できるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="b6d88-387">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="b6d88-388">とはいえ、想定しているリソースのバージョンをクライアント アプリケーションが示すことができるようにする情報を含むカスタム メディアの種類を定義できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-388">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="b6d88-389">次の例は、*application/vnd.adventure-works.v1+json* の値を含む *Accept* ヘッダーを指定する要求を示します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-389">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="b6d88-390">*vnd.adventure-works.v1* 要素は Web サーバーに対し、バージョン 1 のリソースを返すように指示しますが、*json* 要素は応答本文の形式が JSON であるように指定します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-390">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

<span data-ttu-id="b6d88-391">要求を処理しているコードは、*Accept* ヘッダーを処理し、可能な限りそれを使用します (クライアント アプリケーションは複数の形式を *Accept* ヘッダーで指定する場合があり、その場合 Web サーバーは最も適切な形式を応答本文に選択できます)。</span><span class="sxs-lookup"><span data-stu-id="b6d88-391">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="b6d88-392">Web サーバーは次のように Content-Type ヘッダーを使用することで、応答本文のデータの形式を確認します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-392">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="b6d88-393">Accept ヘッダーにより既知のメディアの種類が指定されない場合、Web サーバーは HTTP 406 (Not Acceptable) 応答メッセージを生成するか、既定のメディアの種類でメッセージを返すことができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-393">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="b6d88-394">この方法が最も純粋と思われるバージョン管理メカニズムで、当然ながら HATEOAS で役立ち、リソース リンクの関連データの MIME の種類を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-394">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="b6d88-395">バージョン管理の方法を選択すると、パフォーマンスに対する影響、特に Web サーバーでのキャッシュについて検討する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="b6d88-395">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="b6d88-396">同じ URI/クエリ文字列の組み合わせは同じデータを毎回参照するため、URI バージョン管理および Query String バージョン管理スキームはキャッシュに適しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-396">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="b6d88-397">通常、Header バージョン管理および Media Type バージョン管理メカニズムでは、カスタム ヘッダーまたは Accept ヘッダーの値を確認するのに追加のロジックが必要です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-397">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="b6d88-398">大規模な環境では、多くのクライアントで異なるバージョンの Web API が使用されているため、サーバー側のキャッシュの重複データ量が著しく増加します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-398">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="b6d88-399">この問題が深刻になるのは、クライアント アプリケーションがキャッシュを実装するプロキシを経由して Web サーバーと通信する場合です。また、キャッシュに要求されたデータのコピーが現在保持されていない場合は Web サーバーに要求の転送のみ行います。</span><span class="sxs-lookup"><span data-stu-id="b6d88-399">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>
>
>

## <a name="open-api-initiative"></a><span data-ttu-id="b6d88-400">Open API イニシアチブ</span><span class="sxs-lookup"><span data-stu-id="b6d88-400">Open API Initiative</span></span>
<span data-ttu-id="b6d88-401">[Open API イニシアチブ](https://www.openapis.org/)は、ベンダー間の REST API の記述を標準化するために業界コンソーシアムによって作成されました。</span><span class="sxs-lookup"><span data-stu-id="b6d88-401">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="b6d88-402">このイニシアチブの一環として、Swagger 2.0 仕様という名前が OpenAPI 仕様 (OAS) に変更され、Open API イニシアチブの下に配置されました。</span><span class="sxs-lookup"><span data-stu-id="b6d88-402">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="b6d88-403">Web API に OpenAPI を採用することもできます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-403">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="b6d88-404">考慮すべき点:</span><span class="sxs-lookup"><span data-stu-id="b6d88-404">Some points to consider:</span></span>

- <span data-ttu-id="b6d88-405">OpenAPI 仕様には、REST API の設計方法に関する一連の厳密なガイドラインが付属しています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-405">The OpenAPI Specification comes with with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="b6d88-406">相互運用性の利点はありますが、API を設計するときは、仕様に準拠するようにさらに注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="b6d88-406">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>
- <span data-ttu-id="b6d88-407">OpenAPI では、実装優先のアプローチではなく、コントラクト優先のアプローチが推進されます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-407">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="b6d88-408">コントラクト優先とは、API コントラクト (インターフェイス) を最初に設計してから、コントラクトを実装するコードを記述することを意味します。</span><span class="sxs-lookup"><span data-stu-id="b6d88-408">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span> 
- <span data-ttu-id="b6d88-409">Swagger などのツールにより、API コントラクトからクライアント ライブラリまたはドキュメントを生成できます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-409">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="b6d88-410">例については、[Swagger を使用する ASP.NET Web API のヘルプ ページ](/aspnet/core/tutorials/web-api-help-pages-using-swagger)に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b6d88-410">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="b6d88-411">詳細情報</span><span class="sxs-lookup"><span data-stu-id="b6d88-411">More information</span></span>
* <span data-ttu-id="b6d88-412">[Microsoft REST API ガイドライン](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)には、パブリック REST API の設計に関する詳細な推奨事項が示されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-412">The [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contain detailed recommendations for designing public REST APIs.</span></span>
* <span data-ttu-id="b6d88-413">[RESTful Cookbook](http://restcookbook.com/) では、RESTful API の構築に関する概要が説明されています。</span><span class="sxs-lookup"><span data-stu-id="b6d88-413">The [RESTful Cookbook](http://restcookbook.com/) contains an introduction to building RESTful APIs.</span></span>
* <span data-ttu-id="b6d88-414">[Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) には、Web API を設計および実装するときに検討する、役立つ項目の一覧が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-414">The [Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) contains a useful list of items to consider when designing and implementing a web API.</span></span>
* <span data-ttu-id="b6d88-415">[Open API イニシアチブ](https://www.openapis.org/) サイトには、Open API に関するすべての関連ドキュメントと実装の詳細が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b6d88-415">The [Open API Initiative](https://www.openapis.org/) site, contains all related documentation and implementation details on Open API.</span></span>
