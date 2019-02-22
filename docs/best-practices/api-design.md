---
title: API 設計ガイダンス
titleSuffix: Best practices for cloud applications
description: 適切に設計された Web API を作成する方法に関するガイダンス。
author: dragon119
ms.date: 01/12/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: b15b97de2042a0e213192dd586ffdcc4c51b1f11
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897985"
---
# <a name="api-design"></a><span data-ttu-id="9398c-103">API 設計</span><span class="sxs-lookup"><span data-stu-id="9398c-103">API design</span></span>

<span data-ttu-id="9398c-104">ほとんどの最新の Web アプリケーションでは、クライアントがアプリケーションと対話する際に使用できる API を公開しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-104">Most modern web applications expose APIs that clients can use to interact with the application.</span></span> <span data-ttu-id="9398c-105">適切に設計された Web API には、次をサポートする目的があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-105">A well-designed web API should aim to support:</span></span>

- <span data-ttu-id="9398c-106">**プラットフォームの独立**。</span><span class="sxs-lookup"><span data-stu-id="9398c-106">**Platform independence**.</span></span> <span data-ttu-id="9398c-107">API の内部的な実装方法に関係なく、すべてのクライアントが API を呼び出すことができる必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-107">Any client should be able to call the API, regardless of how the API is implemented internally.</span></span> <span data-ttu-id="9398c-108">そのためには、標準プロトコルを使用し、クライアントと Web サービスが交換するデータの形式に同意できるメカニズムを備えている必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-108">This requires using standard protocols, and having a mechanism whereby the client and the web service can agree on the format of the data to exchange.</span></span>

- <span data-ttu-id="9398c-109">**サービスの進化**。</span><span class="sxs-lookup"><span data-stu-id="9398c-109">**Service evolution**.</span></span> <span data-ttu-id="9398c-110">Web API はクライアント アプリケーションから独立して進化し、機能を追加できる必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-110">The web API should be able to evolve and add functionality independently from client applications.</span></span> <span data-ttu-id="9398c-111">API の進化に伴い、既存のクライアント アプリケーションが変更なしに引き続き機能する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-111">As the API evolves, existing client applications should continue to function without modification.</span></span> <span data-ttu-id="9398c-112">クライアント アプリケーションが機能を十分に使用できるように、すべての機能が検出可能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-112">All functionality should be discoverable so that client applications can fully use it.</span></span>

<span data-ttu-id="9398c-113">このガイダンスでは、Web API の設計時に考慮すべき問題について説明します。</span><span class="sxs-lookup"><span data-stu-id="9398c-113">This guidance describes issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-rest"></a><span data-ttu-id="9398c-114">REST の概要</span><span class="sxs-lookup"><span data-stu-id="9398c-114">Introduction to REST</span></span>

<span data-ttu-id="9398c-115">2000 年に、ロイ・フィールディングが、Web サービスを設計するためのアーキテクチャ アプローチとして Representational State Transfer (REST) を提唱しました。</span><span class="sxs-lookup"><span data-stu-id="9398c-115">In 2000, Roy Fielding proposed Representational State Transfer (REST) as an architectural approach to designing web services.</span></span> <span data-ttu-id="9398c-116">REST はハイパーメディアに基づき分散システムを構築するアーキテクチャ スタイルです。</span><span class="sxs-lookup"><span data-stu-id="9398c-116">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="9398c-117">REST は基になるプロトコルに依存せず、HTTP に必ずしも関連付けられているわけではありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-117">REST is independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="9398c-118">ただし、最も一般的な REST 実装では、アプリケーション プロトコルとして HTTP を使用します。このガイドでは、HTTP の REST API の設計に重点を置きます。</span><span class="sxs-lookup"><span data-stu-id="9398c-118">However, most common REST implementations use HTTP as the application protocol, and this guide focuses on designing REST APIs for HTTP.</span></span>

<span data-ttu-id="9398c-119">REST over HTTP の主な利点は、オープン スタンダードを使用し、API やクライアント アプリケーションの実装を特定の実装にバインドしないことです。</span><span class="sxs-lookup"><span data-stu-id="9398c-119">A primary advantage of REST over HTTP is that it uses open standards, and does not bind the implementation of the API or the client applications to any specific implementation.</span></span> <span data-ttu-id="9398c-120">たとえば、REST Web サービスが ASP.NET で記述されていても、クライアント アプリケーションは任意の言語やツールセットを使用して、HTTP 要求を生成し、HTTP 応答を解析できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-120">For example, a REST web service could be written in ASP.NET, and client applications can use any language or toolset that can generate HTTP requests and parse HTTP responses.</span></span>

<span data-ttu-id="9398c-121">HTTP を使用した RESTful API の主な設計原則を次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-121">Here are some of the main design principles of RESTful APIs using HTTP:</span></span>

- <span data-ttu-id="9398c-122">REST API は "*リソース*" を中心に設計します。リソースは、クライアントがアクセスできるあらゆる種類のオブジェクト、データ、またはサービスです。</span><span class="sxs-lookup"><span data-stu-id="9398c-122">REST APIs are designed around *resources*, which are any kind of object, data, or service that can be accessed by the client.</span></span>

- <span data-ttu-id="9398c-123">リソースには "*識別子*" があります。識別子は、そのリソースを一意に識別する URI です。</span><span class="sxs-lookup"><span data-stu-id="9398c-123">A resource has an *identifier*, which is a URI that uniquely identifies that resource.</span></span> <span data-ttu-id="9398c-124">たとえば、特定の顧客注文の URI は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="9398c-124">For example, the URI for a particular customer order might be:</span></span>

    ```HTTP
    https://adventure-works.com/orders/1
    ```

- <span data-ttu-id="9398c-125">クライアントは、リソースの "*表現*" を交換することでサービスと対話します。</span><span class="sxs-lookup"><span data-stu-id="9398c-125">Clients interact with a service by exchanging *representations* of resources.</span></span> <span data-ttu-id="9398c-126">多くの Web API では、交換形式として JSON を使用します。</span><span class="sxs-lookup"><span data-stu-id="9398c-126">Many web APIs use JSON as the exchange format.</span></span> <span data-ttu-id="9398c-127">たとえば、上記の URI に対する GET 要求では、次の応答本文が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-127">For example, a GET request to the URI listed above might return this response body:</span></span>

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- <span data-ttu-id="9398c-128">REST API では、クライアントとサービスの実装の分離に役立つ統一インターフェイスを使用します。</span><span class="sxs-lookup"><span data-stu-id="9398c-128">REST APIs use a uniform interface, which helps to decouple the client and service implementations.</span></span> <span data-ttu-id="9398c-129">HTTP を基盤とする REST API の場合、統一インターフェイスで標準の HTTP 動詞を使用して、リソースに対する操作を実行します。</span><span class="sxs-lookup"><span data-stu-id="9398c-129">For REST APIs built on HTTP, the uniform interface includes using standard HTTP verbs to perform operations on resources.</span></span> <span data-ttu-id="9398c-130">最も一般的な操作は、GET、POST、PUT、PATCH、DELETE です。</span><span class="sxs-lookup"><span data-stu-id="9398c-130">The most common operations are GET, POST, PUT, PATCH, and DELETE.</span></span>

- <span data-ttu-id="9398c-131">REST API では、ステートレスな要求モデルを使用します。</span><span class="sxs-lookup"><span data-stu-id="9398c-131">REST APIs use a stateless request model.</span></span> <span data-ttu-id="9398c-132">HTTP 要求は独立しており、任意の順序で発生する可能性があるため、要求間の遷移状態の情報を保持することはできません。</span><span class="sxs-lookup"><span data-stu-id="9398c-132">HTTP requests should be independent and may occur in any order, so keeping transient state information between requests is not feasible.</span></span> <span data-ttu-id="9398c-133">情報の格納場所はリソース自体のみであり、それぞれの要求はアトミックな操作である必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-133">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="9398c-134">この制約により、非常にスケーラブルな Web サービスが実現します。クライアントと特定のサーバー間のアフィニティを保持する必要がないためです。</span><span class="sxs-lookup"><span data-stu-id="9398c-134">This constraint enables web services to be highly scalable, because there is no need to retain any affinity between clients and specific servers.</span></span> <span data-ttu-id="9398c-135">どのサーバーも、任意のクライアントからの要求を処理できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-135">Any server can handle any request from any client.</span></span> <span data-ttu-id="9398c-136">ただし、他の要因によってスケーラビリティが制限される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-136">That said, other factors can limit scalability.</span></span> <span data-ttu-id="9398c-137">たとえば、多くの Web サービスは、スケールアウトが難しい可能性のあるバックエンド データ ストアに書き込みを行います (データ ストアのスケールアウト戦略については、「[データのパーティション分割](./data-partitioning.md)」をご覧ください)。</span><span class="sxs-lookup"><span data-stu-id="9398c-137">For example, many web services write to a backend data store, which may be hard to scale out. (The article [Data Partitioning](./data-partitioning.md) describes strategies to scale out a data store.)</span></span>

- <span data-ttu-id="9398c-138">REST API は、表現に含まれているハイパーメディア リンクによって動作します。</span><span class="sxs-lookup"><span data-stu-id="9398c-138">REST APIs are driven by hypermedia links that are contained in the representation.</span></span> <span data-ttu-id="9398c-139">たとえば、次の例は注文の JSON 表現を示しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-139">For example, the following shows a JSON representation of an order.</span></span> <span data-ttu-id="9398c-140">これには、注文に関連付けられた顧客を取得または更新するためのリンクが含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-140">It contains links to get or update the customer associated with the order.</span></span>

    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"PUT" }
        ]
    }
    ```

<span data-ttu-id="9398c-141">2008 年に、レオナルド・リチャードソンが Web API の次の[成熟度モデル](https://martinfowler.com/articles/richardsonMaturityModel.html)を提唱しました。</span><span class="sxs-lookup"><span data-stu-id="9398c-141">In 2008, Leonard Richardson proposed the following [maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html) for web APIs:</span></span>

- <span data-ttu-id="9398c-142">レベル 0:1 つの URI を定義し、すべての操作がこの URI に対する POST 要求です。</span><span class="sxs-lookup"><span data-stu-id="9398c-142">Level 0: Define one URI, and all operations are POST requests to this URI.</span></span>
- <span data-ttu-id="9398c-143">レベル 1:リソースごとに個別の URI を作成します。</span><span class="sxs-lookup"><span data-stu-id="9398c-143">Level 1: Create separate URIs for individual resources.</span></span>
- <span data-ttu-id="9398c-144">レベル 2:HTTP メソッドを使用して、リソースに対する操作を定義します。</span><span class="sxs-lookup"><span data-stu-id="9398c-144">Level 2: Use HTTP methods to define operations on resources.</span></span>
- <span data-ttu-id="9398c-145">レベル 3:ハイパーメディアを使用します (後述の HATEOAS)。</span><span class="sxs-lookup"><span data-stu-id="9398c-145">Level 3: Use hypermedia (HATEOAS, described below).</span></span>

<span data-ttu-id="9398c-146">フィールディングの定義に従うと、レベル 3 が真の RESTful API になります。</span><span class="sxs-lookup"><span data-stu-id="9398c-146">Level 3 corresponds to a truly RESTful API according to Fielding's definition.</span></span> <span data-ttu-id="9398c-147">実際には、公開されている多くの Web API がほぼレベル 2 に位置します。</span><span class="sxs-lookup"><span data-stu-id="9398c-147">In practice, many published web APIs fall somewhere around level 2.</span></span>

## <a name="organize-the-api-around-resources"></a><span data-ttu-id="9398c-148">リソースを中心とした API の整理</span><span class="sxs-lookup"><span data-stu-id="9398c-148">Organize the API around resources</span></span>

<span data-ttu-id="9398c-149">Web API が公開するビジネス エンティティに注目して説明します。</span><span class="sxs-lookup"><span data-stu-id="9398c-149">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="9398c-150">たとえば、電子商取引システムでは、主エンティティは顧客と注文です。</span><span class="sxs-lookup"><span data-stu-id="9398c-150">For example, in an e-commerce system, the primary entities might be customers and orders.</span></span> <span data-ttu-id="9398c-151">注文の作成は、注文情報が含まれた HTTP POST 要求を送信することで実現できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-151">Creating an order can be achieved by sending an HTTP POST request that contains the order information.</span></span> <span data-ttu-id="9398c-152">HTTP 応答では、注文が正常に行われたかどうかを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-152">The HTTP response indicates whether the order was placed successfully or not.</span></span> <span data-ttu-id="9398c-153">可能であれば、リソース URI は動詞 (リソースに対する操作) ではなく、名詞 (リソース) に基づくようにします。</span><span class="sxs-lookup"><span data-stu-id="9398c-153">When possible, resource URIs should be based on nouns (the resource) and not verbs (the operations on the resource).</span></span>

```HTTP
https://adventure-works.com/orders // Good

https://adventure-works.com/create-order // Avoid
```

<span data-ttu-id="9398c-154">リソースは、単一の物理データ項目に基づく必要はありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-154">A resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="9398c-155">たとえば、注文リソースをリレーショナル データベースの複数のテーブルとして内部的に実装しながら、クライアントには単一のエンティティとして表示することができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-155">For example, an order resource might be implemented internally as several tables in a relational database, but presented to the client as a single entity.</span></span> <span data-ttu-id="9398c-156">データベースの内部構造を単にミラー化する API は作成しないようにします。</span><span class="sxs-lookup"><span data-stu-id="9398c-156">Avoid creating APIs that simply mirror the internal structure of a database.</span></span> <span data-ttu-id="9398c-157">REST の目的は、エンティティと、アプリケーションがそれらのエンティティに対して実行できる操作をモデル化することです。</span><span class="sxs-lookup"><span data-stu-id="9398c-157">The purpose of REST is to model entities and the operations that an application can perform on those entities.</span></span> <span data-ttu-id="9398c-158">クライアントを内部実装に公開しないでください。</span><span class="sxs-lookup"><span data-stu-id="9398c-158">A client should not be exposed to the internal implementation.</span></span>

<span data-ttu-id="9398c-159">多くの場合、エンティティはコレクション (orders、customers) にグループ化されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-159">Entities are often grouped together into collections (orders, customers).</span></span> <span data-ttu-id="9398c-160">コレクションはコレクションの項目とは別のリソースであり、独自の URI が必要です。</span><span class="sxs-lookup"><span data-stu-id="9398c-160">A collection is a separate resource from the item within the collection, and should have its own URI.</span></span> <span data-ttu-id="9398c-161">たとえば、次の URI は注文のコレクションを表しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-161">For example, the following URI might represent the collection of orders:</span></span>

```HTTP
https://adventure-works.com/orders
```

<span data-ttu-id="9398c-162">コレクション URI に HTTP GET 要求を送信することで、コレクションの項目の一覧を取得します。</span><span class="sxs-lookup"><span data-stu-id="9398c-162">Sending an HTTP GET request to the collection URI retrieves a list of items in the collection.</span></span> <span data-ttu-id="9398c-163">コレクションの各項目にも、それぞれ一意の URI があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-163">Each item in the collection also has its own unique URI.</span></span> <span data-ttu-id="9398c-164">項目の URI に対する HTTP GET 要求では、その項目の詳細が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-164">An HTTP GET request to the item's URI returns the details of that item.</span></span>

<span data-ttu-id="9398c-165">URI には一貫性のある名前付け規則を使用します。</span><span class="sxs-lookup"><span data-stu-id="9398c-165">Adopt a consistent naming convention in URIs.</span></span> <span data-ttu-id="9398c-166">一般に、コレクションを参照する URI には複数形の名詞を使用すると便利です。</span><span class="sxs-lookup"><span data-stu-id="9398c-166">In general, it helps to use plural nouns for URIs that reference collections.</span></span> <span data-ttu-id="9398c-167">コレクションと項目の URI は、階層構造に整理することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="9398c-167">It's a good practice to organize URIs for collections and items into a hierarchy.</span></span> <span data-ttu-id="9398c-168">たとえば、`/customers` は customers コレクションのパスであり、`/customers/5` は ID が 5 の顧客のパスです。</span><span class="sxs-lookup"><span data-stu-id="9398c-168">For example, `/customers` is the path to the customers collection, and `/customers/5` is the path to the customer with ID equal to 5.</span></span> <span data-ttu-id="9398c-169">この方法は、Web API を直感的に保つのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="9398c-169">This approach helps to keep the web API intuitive.</span></span> <span data-ttu-id="9398c-170">また、多くの Web API フレームワークでは、パラメーター化された URI パスに基づいて要求をルーティングできるので、パス `/customers/{id}` のルートを定義できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-170">Also, many web API frameworks can route requests based on parameterized URI paths, so you could define a route for the path `/customers/{id}`.</span></span>

<span data-ttu-id="9398c-171">さまざまな種類のリソース間の関係と、これらの関連付けを公開する方法も検討します。</span><span class="sxs-lookup"><span data-stu-id="9398c-171">Also consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="9398c-172">たとえば、`/customers/5/orders` は顧客 5 のすべての注文を表します。</span><span class="sxs-lookup"><span data-stu-id="9398c-172">For example, the `/customers/5/orders` might represent all of the orders for customer 5.</span></span> <span data-ttu-id="9398c-173">逆方向にし、`/orders/99/customer` などの URI を使用して、注文から顧客への関連付けを表すこともできます。</span><span class="sxs-lookup"><span data-stu-id="9398c-173">You could also go in the other direction, and represent the association from an order back to a customer with a URI such as `/orders/99/customer`.</span></span> <span data-ttu-id="9398c-174">ただし、このモデルを拡張しすぎると、実装が煩雑になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-174">However, extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="9398c-175">より優れたソリューションは、HTTP 応答メッセージの本文に、関連するリソースに誘導できるリンクを含めることです。</span><span class="sxs-lookup"><span data-stu-id="9398c-175">A better solution is to provide navigable links to associated resources in the body of the HTTP response message.</span></span> <span data-ttu-id="9398c-176">このメカニズムについては、[関連リソースへのナビゲーションを可能にする HATEOAS アプローチの使用](#using-the-hateoas-approach-to-enable-navigation-to-related-resources)に関するセクションで詳しく説明します。</span><span class="sxs-lookup"><span data-stu-id="9398c-176">This mechanism is described in more detail in the section [Using the HATEOAS Approach to Enable Navigation To Related Resources later](#using-the-hateoas-approach-to-enable-navigation-to-related-resources).</span></span>

<span data-ttu-id="9398c-177">複雑なシステムでは、クライアント が複数のレベルの関係をナビゲートできるようにする URI (例: `/customers/1/orders/99/products`) を提供したくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-177">In more complex systems, it can be tempting to provide URIs that enable a client to navigate through several levels of relationships, such as `/customers/1/orders/99/products`.</span></span> <span data-ttu-id="9398c-178">ただし、将来的にリソース間のリレーションシップが変わる場合、このレベルの複雑さを維持するのは難しく、柔軟性がありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-178">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="9398c-179">代わりに、URI を比較的単純に保つように努めます。</span><span class="sxs-lookup"><span data-stu-id="9398c-179">Instead, try to keep URIs relatively simple.</span></span> <span data-ttu-id="9398c-180">アプリケーションがリソースへの参照を取得したら、この参照を使用してそのリソースに関連する項目を検索できるようにします。</span><span class="sxs-lookup"><span data-stu-id="9398c-180">Once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="9398c-181">前述のクエリを URI `/customers/1/orders` に置き換えると、顧客 1 のすべての注文を検索でき、次に `/orders/99/products` に置き換えると、この注文の製品を検索できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-181">The preceding query can be replaced with the URI `/customers/1/orders` to find all the orders for customer 1, and then `/orders/99/products` to find the products in this order.</span></span>

> [!TIP]
> <span data-ttu-id="9398c-182">"*コレクション/項目/コレクション*" よりも複雑なリソース URI を要求しないでください。</span><span class="sxs-lookup"><span data-stu-id="9398c-182">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>

<span data-ttu-id="9398c-183">Web 要求はいずれも Web サーバーに負荷をかけるという点も考慮します。</span><span class="sxs-lookup"><span data-stu-id="9398c-183">Another factor is that all web requests impose a load on the web server.</span></span> <span data-ttu-id="9398c-184">要求が増えるほど、負荷も大きくなります。</span><span class="sxs-lookup"><span data-stu-id="9398c-184">The more requests, the bigger the load.</span></span> <span data-ttu-id="9398c-185">そのため、多数の小さなリソースを公開する "おしゃべりな" Web API にならないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="9398c-185">Therefore, try to avoid "chatty" web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="9398c-186">このような API では、クライアント アプリケーションが必要なすべてのデータを検索するために、複数の要求を送信することが必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-186">Such an API may require a client application to send multiple requests to find all of the data that it requires.</span></span> <span data-ttu-id="9398c-187">代わりに、データを非正規化し、1 つの要求で取得できるより大きなリソースに関連情報をまとめることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-187">Instead, you might want to denormalize the data and combine related information into bigger resources that can be retrieved with a single request.</span></span> <span data-ttu-id="9398c-188">ただし、このアプローチでは、クライアントが必要としないデータをフェッチするオーバーヘッドとバランスを取る必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-188">However, you need to balance this approach against the overhead of fetching data that the client doesn't need.</span></span> <span data-ttu-id="9398c-189">ラージ オブジェクトを取得すると、要求の待機時間が増加し、追加の帯域幅コストが発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-189">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs.</span></span> <span data-ttu-id="9398c-190">これらのパフォーマンスのアンチパターンの詳細については、[頻度の高い I/O](../antipatterns/chatty-io/index.md) に関する記事、および[余分なフェッチ](../antipatterns/extraneous-fetching/index.md)に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="9398c-190">For more information about these performance antipatterns, see [Chatty I/O](../antipatterns/chatty-io/index.md) and [Extraneous Fetching](../antipatterns/extraneous-fetching/index.md).</span></span>

<span data-ttu-id="9398c-191">Web API と基になるデータ ソース間に依存関係を導入しないようにします。</span><span class="sxs-lookup"><span data-stu-id="9398c-191">Avoid introducing dependencies between the web API and the underlying data sources.</span></span> <span data-ttu-id="9398c-192">たとえば、データがリレーショナル データベースに格納されている場合、Web API は各テーブルをリソースのコレクションとして公開する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-192">For example, if your data is stored in a relational database, the web API doesn't need to expose each table as a collection of resources.</span></span> <span data-ttu-id="9398c-193">実際には、これは不適切な設計と考えられます。</span><span class="sxs-lookup"><span data-stu-id="9398c-193">In fact, that's probably a poor design.</span></span> <span data-ttu-id="9398c-194">代わりに、Web API をデータベースの抽象化と捉えます。</span><span class="sxs-lookup"><span data-stu-id="9398c-194">Instead, think of the web API as an abstraction of the database.</span></span> <span data-ttu-id="9398c-195">必要に応じて、データベースと Web API 間のマッピング レイヤーを導入します。</span><span class="sxs-lookup"><span data-stu-id="9398c-195">If necessary, introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="9398c-196">これにより、クライアント アプリケーションは基になるデータベース スキームの変更から分離されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-196">That way, client applications are isolated from changes to the underlying database scheme.</span></span>

<span data-ttu-id="9398c-197">最後に、Web API により実装されるすべての操作を特定のリソースにマッピングできない場合があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-197">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="9398c-198">機能を呼び出し、HTTP 応答メッセージとして結果を返す HTTP 要求によって、そのような "*非リソース*" シナリオに対応できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-198">You can handle such *non-resource* scenarios through HTTP requests that invoke a function and return the results as an HTTP response message.</span></span> <span data-ttu-id="9398c-199">たとえば、加算や減算などの単純な電卓操作を実装する Web API は、これらの操作を疑似リソースとして公開し、クエリ文字列を使用して必要なパラメーターを指定する URI を提供できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-199">For example, a web API that implements simple calculator operations such as add and subtract could provide URIs that expose these operations as pseudo resources and use the query string to specify the parameters required.</span></span> <span data-ttu-id="9398c-200">たとえば、URI */add?operand1=99&operand2=1* に対する GET 要求では、本文に値 100 が含まれた応答メッセージが返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-200">For example a GET request to the URI */add?operand1=99&operand2=1* would return a response message with the body containing the value 100.</span></span> <span data-ttu-id="9398c-201">ただし、これらの形式の URI は常に慎重に使用してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-201">However, only use these forms of URIs sparingly.</span></span>

## <a name="define-operations-in-terms-of-http-methods"></a><span data-ttu-id="9398c-202">HTTP メソッドに関する操作の定義</span><span class="sxs-lookup"><span data-stu-id="9398c-202">Define operations in terms of HTTP methods</span></span>

<span data-ttu-id="9398c-203">HTTP プロトコルにより、要求にセマンティックな意味を割り当てる多くのメソッドが定義されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-203">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="9398c-204">ほとんどの RESTful Web API により使用される一般的な HTTP メソッドは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="9398c-204">The common HTTP methods used by most RESTful web APIs are:</span></span>

- <span data-ttu-id="9398c-205">**GET**: 指定した URI のリソースの表現を取得します。</span><span class="sxs-lookup"><span data-stu-id="9398c-205">**GET** retrieves a representation of the resource at the specified URI.</span></span> <span data-ttu-id="9398c-206">応答メッセージの本文には、要求されたリソースの詳細が含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-206">The body of the response message contains the details of the requested resource.</span></span>
- <span data-ttu-id="9398c-207">**POST**: 指定した URI の新しいリソースを作成します。</span><span class="sxs-lookup"><span data-stu-id="9398c-207">**POST** creates a new resource at the specified URI.</span></span> <span data-ttu-id="9398c-208">応答メッセージの本文には、新しいリソースの詳細が記載されています。</span><span class="sxs-lookup"><span data-stu-id="9398c-208">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="9398c-209">POST は実際にはリソースを作成しない操作をトリガーするのにも使用できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-209">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
- <span data-ttu-id="9398c-210">**PUT**: 指定した URI のリソースを作成または置換します。</span><span class="sxs-lookup"><span data-stu-id="9398c-210">**PUT** either creates or replaces the resource at the specified URI.</span></span> <span data-ttu-id="9398c-211">要求メッセージの本文で、作成または更新するリソースを指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-211">The body of the request message specifies the resource to be created or updated.</span></span>
- <span data-ttu-id="9398c-212">**PATCH**: リソースの部分的な更新を実行します。</span><span class="sxs-lookup"><span data-stu-id="9398c-212">**PATCH** performs a partial update of a resource.</span></span> <span data-ttu-id="9398c-213">要求本文でリソースに適用する一連の変更を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-213">The request body specifies the set of changes to apply to the resource.</span></span>
- <span data-ttu-id="9398c-214">**DELETE**: 指定した URI のリソースを削除します。</span><span class="sxs-lookup"><span data-stu-id="9398c-214">**DELETE** removes the resource at the specified URI.</span></span>

<span data-ttu-id="9398c-215">特定の要求の影響は、リソースがコレクションと個々の項目のどちらであるかによって異なります。</span><span class="sxs-lookup"><span data-stu-id="9398c-215">The effect of a specific request should depend on whether the resource is a collection or an individual item.</span></span> <span data-ttu-id="9398c-216">以下の表では、電子商取引の例を使用しながら、ほとんどの RESTful の実装で使用される一般的な規則をまとめています。</span><span class="sxs-lookup"><span data-stu-id="9398c-216">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="9398c-217">これらの要求がすべて実装されない場合があります。具体的なシナリオにより異なります。</span><span class="sxs-lookup"><span data-stu-id="9398c-217">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="9398c-218">**リソース**</span><span class="sxs-lookup"><span data-stu-id="9398c-218">**Resource**</span></span> | <span data-ttu-id="9398c-219">**POST**</span><span class="sxs-lookup"><span data-stu-id="9398c-219">**POST**</span></span> | <span data-ttu-id="9398c-220">**GET**</span><span class="sxs-lookup"><span data-stu-id="9398c-220">**GET**</span></span> | <span data-ttu-id="9398c-221">**PUT**</span><span class="sxs-lookup"><span data-stu-id="9398c-221">**PUT**</span></span> | <span data-ttu-id="9398c-222">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="9398c-222">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="9398c-223">/customers</span><span class="sxs-lookup"><span data-stu-id="9398c-223">/customers</span></span> |<span data-ttu-id="9398c-224">新しい顧客を作成</span><span class="sxs-lookup"><span data-stu-id="9398c-224">Create a new customer</span></span> |<span data-ttu-id="9398c-225">すべての顧客を取得</span><span class="sxs-lookup"><span data-stu-id="9398c-225">Retrieve all customers</span></span> |<span data-ttu-id="9398c-226">顧客を一括更新</span><span class="sxs-lookup"><span data-stu-id="9398c-226">Bulk update of customers</span></span> |<span data-ttu-id="9398c-227">すべての顧客を削除</span><span class="sxs-lookup"><span data-stu-id="9398c-227">Remove all customers</span></span> |
| <span data-ttu-id="9398c-228">/customers/1</span><span class="sxs-lookup"><span data-stu-id="9398c-228">/customers/1</span></span> |<span data-ttu-id="9398c-229">Error</span><span class="sxs-lookup"><span data-stu-id="9398c-229">Error</span></span> |<span data-ttu-id="9398c-230">顧客 1 の詳細を取得</span><span class="sxs-lookup"><span data-stu-id="9398c-230">Retrieve the details for customer 1</span></span> |<span data-ttu-id="9398c-231">顧客 1 の詳細を更新 (顧客 1 が存在する場合)</span><span class="sxs-lookup"><span data-stu-id="9398c-231">Update the details of customer 1 if it exists</span></span> |<span data-ttu-id="9398c-232">顧客 1 を削除</span><span class="sxs-lookup"><span data-stu-id="9398c-232">Remove customer 1</span></span> |
| <span data-ttu-id="9398c-233">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="9398c-233">/customers/1/orders</span></span> |<span data-ttu-id="9398c-234">顧客 1 の新しい注文を作成</span><span class="sxs-lookup"><span data-stu-id="9398c-234">Create a new order for customer 1</span></span> |<span data-ttu-id="9398c-235">顧客 1 のすべての注文を取得</span><span class="sxs-lookup"><span data-stu-id="9398c-235">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="9398c-236">顧客 1 の注文を一括更新</span><span class="sxs-lookup"><span data-stu-id="9398c-236">Bulk update of orders for customer 1</span></span> |<span data-ttu-id="9398c-237">顧客 1 のすべての注文を削除</span><span class="sxs-lookup"><span data-stu-id="9398c-237">Remove all orders for customer 1</span></span> |

<span data-ttu-id="9398c-238">POST、PUT、PATCH の違いがわかりにくい可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-238">The differences between POST, PUT, and PATCH can be confusing.</span></span>

- <span data-ttu-id="9398c-239">POST 要求ではリソースを作成します。</span><span class="sxs-lookup"><span data-stu-id="9398c-239">A POST request creates a resource.</span></span> <span data-ttu-id="9398c-240">サーバーは新しいリソースの URI を割り当て、その URI をクライアントに返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-240">The server assigns a URI for the new resource, and returns that URI to the client.</span></span> <span data-ttu-id="9398c-241">REST モデルでは、POST 要求をコレクションに適用することがよくあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-241">In the REST model, you frequently apply POST requests to collections.</span></span> <span data-ttu-id="9398c-242">新しいリソースがコレクションに追加されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-242">The new resource is added to the collection.</span></span> <span data-ttu-id="9398c-243">POST 要求を使用して、新しいリソースを作成せずに、処理するデータを既存のリソースに送信することもできます。</span><span class="sxs-lookup"><span data-stu-id="9398c-243">A POST request can also be used to submit data for processing to an existing resource, without any new resource being created.</span></span>

- <span data-ttu-id="9398c-244">PUT 要求では、リソースの作成*または*既存のリソースの更新を行います。</span><span class="sxs-lookup"><span data-stu-id="9398c-244">A PUT request creates a resource *or* updates an existing resource.</span></span> <span data-ttu-id="9398c-245">クライアントがリソースの URI を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-245">The client specifies the URI for the resource.</span></span> <span data-ttu-id="9398c-246">要求本文には、リソースの完全な表現が含まれます。</span><span class="sxs-lookup"><span data-stu-id="9398c-246">The request body contains a complete representation of the resource.</span></span> <span data-ttu-id="9398c-247">この URI を使用するリソースが既に存在する場合は、そのリソースが置き換えられます。</span><span class="sxs-lookup"><span data-stu-id="9398c-247">If a resource with this URI already exists, it is replaced.</span></span> <span data-ttu-id="9398c-248">それ以外の場合は、新しいリソースが作成されます (サーバーが作成をサポートしている場合)。</span><span class="sxs-lookup"><span data-stu-id="9398c-248">Otherwise a new resource is created, if the server supports doing so.</span></span> <span data-ttu-id="9398c-249">PUT 要求は、コレクションではなく、個々の項目 (特定の顧客など) であるリソースに最も頻繁に適用されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-249">PUT requests are most frequently applied to resources that are individual items, such as a specific customer, rather than collections.</span></span> <span data-ttu-id="9398c-250">サーバーが PUT を介した更新をサポートしていても、作成はサポートしていないことがあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-250">A server might support updates but not creation via PUT.</span></span> <span data-ttu-id="9398c-251">PUT を介した作成をサポートするかどうかは、リソースが存在する前に、クライアントが意味のある方法でリソースに URI を割り当てることができるかどうかに左右されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-251">Whether to support creation via PUT depends on whether the client can meaningfully assign a URI to a resource before it exists.</span></span> <span data-ttu-id="9398c-252">できない場合は、POST を使用してリソースを作成し、PUT または PATCH を使用して更新します。</span><span class="sxs-lookup"><span data-stu-id="9398c-252">If not, then use POST to create resources and PUT or PATCH to update.</span></span>

- <span data-ttu-id="9398c-253">PATCH 要求では、既存のリソースに対して "*部分的な更新*" を実行します。</span><span class="sxs-lookup"><span data-stu-id="9398c-253">A PATCH request performs a *partial update* to an existing resource.</span></span> <span data-ttu-id="9398c-254">クライアントがリソースの URI を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-254">The client specifies the URI for the resource.</span></span> <span data-ttu-id="9398c-255">要求本文で、リソースに適用する一連の "*変更*" を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-255">The request body specifies a set of *changes* to apply to the resource.</span></span> <span data-ttu-id="9398c-256">クライアントは、リソースの表現全体ではなく、変更のみを送信するため、PUT を使用するよりも効率的です。</span><span class="sxs-lookup"><span data-stu-id="9398c-256">This can be more efficient than using PUT, because the client only sends the changes, not the entire representation of the resource.</span></span> <span data-ttu-id="9398c-257">技術的には、PATCH で ("null" リソースに対する一連の更新を指定することによって) 新しいリソースを作成することもできます (サーバーがこれをサポートしている場合)。</span><span class="sxs-lookup"><span data-stu-id="9398c-257">Technically PATCH can also create a new resource (by specifying a set of updates to a "null" resource), if the server supports this.</span></span>

<span data-ttu-id="9398c-258">PUT 要求はべき等である必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-258">PUT requests must be idempotent.</span></span> <span data-ttu-id="9398c-259">クライアントが同じ PUT 要求を複数回送信した場合、結果は常に同じになります (同じリソースが同じ値で変更されます)。</span><span class="sxs-lookup"><span data-stu-id="9398c-259">If a client submits the same PUT request multiple times, the results should always be the same (the same resource will be modified with the same values).</span></span> <span data-ttu-id="9398c-260">POST 要求と PATCH 要求はべき等である保証はありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-260">POST and PATCH requests are not guaranteed to be idempotent.</span></span>

## <a name="conform-to-http-semantics"></a><span data-ttu-id="9398c-261">HTTP セマンティクスへの準拠</span><span class="sxs-lookup"><span data-stu-id="9398c-261">Conform to HTTP semantics</span></span>

<span data-ttu-id="9398c-262">このセクションでは、HTTP 仕様に準拠した API の設計に関する一般的な考慮事項について説明します。</span><span class="sxs-lookup"><span data-stu-id="9398c-262">This section describes some typical considerations for designing an API that conforms to the HTTP specification.</span></span> <span data-ttu-id="9398c-263">ただし、考えられるすべての詳細やシナリオを取り上げているわけではありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-263">However, it doesn't cover every possible detail or scenario.</span></span> <span data-ttu-id="9398c-264">不確かな場合は、HTTP 仕様を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-264">When in doubt, consult the HTTP specifications.</span></span>

### <a name="media-types"></a><span data-ttu-id="9398c-265">メディアの種類</span><span class="sxs-lookup"><span data-stu-id="9398c-265">Media types</span></span>

<span data-ttu-id="9398c-266">前述のように、クライアントとサーバーはリソースの表現を交換します。</span><span class="sxs-lookup"><span data-stu-id="9398c-266">As mentioned earlier, clients and servers exchange representations of resources.</span></span> <span data-ttu-id="9398c-267">たとえば、POST 要求では、要求本文に作成するリソースの表現が含まれます。</span><span class="sxs-lookup"><span data-stu-id="9398c-267">For example, in a POST request, the request body contains a representation of the resource to create.</span></span> <span data-ttu-id="9398c-268">GET 要求では、応答本文にフェッチされたリソースの表現が含まれます。</span><span class="sxs-lookup"><span data-stu-id="9398c-268">In a GET request, the response body contains a representation of the fetched resource.</span></span>

<span data-ttu-id="9398c-269">HTTP プロトコルでは、"*メディアの種類*" (MIME の種類とも呼ばれます) を使用して形式が指定されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-269">In the HTTP protocol, formats are specified through the use of *media types*, also called MIME types.</span></span> <span data-ttu-id="9398c-270">非バイナリ データの場合、ほとんどの Web API が JSON (メディアの種類 = application/json) をサポートしており、場合によっては XML (メディアの種類 = application/xml) もサポートしています。</span><span class="sxs-lookup"><span data-stu-id="9398c-270">For non-binary data, most web APIs support JSON (media type = application/json) and possibly XML (media type = application/xml).</span></span>

<span data-ttu-id="9398c-271">要求または応答の Content-Type ヘッダーでは、表現の形式を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-271">The Content-Type header in a request or response specifies the format of the representation.</span></span> <span data-ttu-id="9398c-272">JSON データを含む POST 要求の例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-272">Here is an example of a POST request that includes JSON data:</span></span>

```HTTP
POST https://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

<span data-ttu-id="9398c-273">サーバーがこのメディアの種類をサポートしていない場合、HTTP 状態コード 415 (Unsupported Media Type) が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-273">If the server doesn't support the media type, it should return HTTP status code 415 (Unsupported Media Type).</span></span>

<span data-ttu-id="9398c-274">クライアント要求には、クライアントが応答メッセージでサーバーから受け入れるメディアの種類の一覧を含む Accept ヘッダーを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-274">A client request can include an Accept header that contains a list of media types the client will accept from the server in the response message.</span></span> <span data-ttu-id="9398c-275">例: </span><span class="sxs-lookup"><span data-stu-id="9398c-275">For example:</span></span>

```HTTP
GET https://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

<span data-ttu-id="9398c-276">サーバーが記載されているメディアの種類のいずれかに対応できない場合、HTTP 状態コード406 (Not Acceptable) が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-276">If the server cannot match any of the media type(s) listed, it should return HTTP status code 406 (Not Acceptable).</span></span>

### <a name="get-methods"></a><span data-ttu-id="9398c-277">GET メソッド</span><span class="sxs-lookup"><span data-stu-id="9398c-277">GET methods</span></span>

<span data-ttu-id="9398c-278">GET メソッドが成功すると、通常は HTTP 状態コード200 (OK) が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-278">A successful GET method typically returns HTTP status code 200 (OK).</span></span> <span data-ttu-id="9398c-279">リソースが見つからない場合、メソッドは 404 (Not Found) を返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-279">If the resource cannot be found, the method should return 404 (Not Found).</span></span>

### <a name="post-methods"></a><span data-ttu-id="9398c-280">POST メソッド</span><span class="sxs-lookup"><span data-stu-id="9398c-280">POST methods</span></span>

<span data-ttu-id="9398c-281">POST メソッドは、新しいリソースを作成すると、HTTP 状態コード 201 (Created) を返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-281">If a POST method creates a new resource, it returns HTTP status code 201 (Created).</span></span> <span data-ttu-id="9398c-282">新しいリソースの URI は、応答の Location ヘッダーに含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-282">The URI of the new resource is included in the Location header of the response.</span></span> <span data-ttu-id="9398c-283">応答本文には、リソースの表現が含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-283">The response body contains a representation of the resource.</span></span>

<span data-ttu-id="9398c-284">メソッドが新しいリソースの作成以外の処理を実行した場合は、HTTP 状態コード 200 を返し、応答本文に操作の結果を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-284">If the method does some processing but does not create a new resource, the method can return HTTP status code 200 and include the result of the operation in the response body.</span></span> <span data-ttu-id="9398c-285">また、返す結果がない場合、メソッドは応答本文なしで HTTP 状態コード 204 (No Content) を返すことができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-285">Alternatively, if there is no result to return, the method can return HTTP status code 204 (No Content) with no response body.</span></span>

<span data-ttu-id="9398c-286">クライアントが要求に無効なデータを挿入した場合、サーバーは HTTP 状態コード 400 (Bad Request) を返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-286">If the client puts invalid data into the request, the server should return HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="9398c-287">応答本文には、エラーに関する追加情報または詳細を提供する URI へのリンクを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-287">The response body can contain additional information about the error or a link to a URI that provides more details.</span></span>

### <a name="put-methods"></a><span data-ttu-id="9398c-288">PUT メソッド</span><span class="sxs-lookup"><span data-stu-id="9398c-288">PUT methods</span></span>

<span data-ttu-id="9398c-289">POST メソッドと同様に、PUT メソッドは新しいリソースを作成すると、HTTP 状態コード 201 (Created) を返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-289">If a PUT method creates a new resource, it returns HTTP status code 201 (Created), as with a POST method.</span></span> <span data-ttu-id="9398c-290">既存のリソースを更新した場合は、200 (OK) または 204 (No Content) を返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-290">If the method updates an existing resource, it returns either 200 (OK) or 204 (No Content).</span></span> <span data-ttu-id="9398c-291">既存のリソースを更新できない場合もあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-291">In some cases, it might not be possible to update an existing resource.</span></span> <span data-ttu-id="9398c-292">その場合は、HTTP 状態コード 409 (Conflict) を返すことを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-292">In that case, consider returning HTTP status code 409 (Conflict).</span></span>

<span data-ttu-id="9398c-293">コレクションの複数のリソースの更新をバッチ処理できる一括 HTTP PUT 操作の実装を検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-293">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="9398c-294">PUT 要求はコレクションの URI を指定し、要求本文は変更するリソースの詳細を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-294">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="9398c-295">この方法によりおしゃべりを減らし、パフォーマンスを向上させることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-295">This approach can help to reduce chattiness and improve performance.</span></span>

### <a name="patch-methods"></a><span data-ttu-id="9398c-296">PATCH メソッド</span><span class="sxs-lookup"><span data-stu-id="9398c-296">PATCH methods</span></span>

<span data-ttu-id="9398c-297">PATCH 要求では、クライアントが*パッチ ドキュメント*の形で既存のリソースに対する一連の更新を送信します。</span><span class="sxs-lookup"><span data-stu-id="9398c-297">With a PATCH request, the client sends a set of updates to an existing resource, in the form of a *patch document*.</span></span> <span data-ttu-id="9398c-298">サーバーはパッチ ドキュメントを処理して更新を実行します。</span><span class="sxs-lookup"><span data-stu-id="9398c-298">The server processes the patch document to perform the update.</span></span> <span data-ttu-id="9398c-299">パッチ ドキュメントは、リソース全体を記述するのではなく、適用する一連の変更のみを記述します。</span><span class="sxs-lookup"><span data-stu-id="9398c-299">The patch document doesn't describe the whole resource, only a set of changes to apply.</span></span> <span data-ttu-id="9398c-300">PATCH メソッドの仕様 ([RFC 5789](https://tools.ietf.org/html/rfc5789)) では、パッチ ドキュメントの特定の形式は定義されていません。</span><span class="sxs-lookup"><span data-stu-id="9398c-300">The specification for the PATCH method ([RFC 5789](https://tools.ietf.org/html/rfc5789)) doesn't define a particular format for patch documents.</span></span> <span data-ttu-id="9398c-301">形式は、要求内のメディアの種類から推論する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-301">The format must be inferred from the media type in the request.</span></span>

<span data-ttu-id="9398c-302">JSON が Web API の最も一般的なデータ形式と考えられます。</span><span class="sxs-lookup"><span data-stu-id="9398c-302">JSON is probably the most common data format for web APIs.</span></span> <span data-ttu-id="9398c-303">主な JSON ベースのパッチ形式として、*JSON パッチ*と *JSON マージ パッチ*の 2 つがあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-303">There are two main JSON-based patch formats, called *JSON patch* and *JSON merge patch*.</span></span>

<span data-ttu-id="9398c-304">JSON マージ パッチの方が若干シンプルです。</span><span class="sxs-lookup"><span data-stu-id="9398c-304">JSON merge patch is somewhat simpler.</span></span> <span data-ttu-id="9398c-305">パッチ ドキュメントは元の JSON リソースと同じ構造ですが、変更または追加する必要のあるフィールドのサブセットだけが含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-305">The patch document has the same structure as the original JSON resource, but includes just the subset of fields that should be changed or added.</span></span> <span data-ttu-id="9398c-306">また、パッチ ドキュメントのフィールド値に `null` を指定することによって、そのフィールドを削除できます </span><span class="sxs-lookup"><span data-stu-id="9398c-306">In addition, a field can be deleted by specifying `null` for the field value in the patch document.</span></span> <span data-ttu-id="9398c-307">(つまり、元のリソースに明示的な null 値を含めることができる場合、マージ パッチは適していません)。</span><span class="sxs-lookup"><span data-stu-id="9398c-307">(That means merge patch is not suitable if the original resource can have explicit null values.)</span></span>

<span data-ttu-id="9398c-308">たとえば、元のリソースが次の JSON 表現を持つとします。</span><span class="sxs-lookup"><span data-stu-id="9398c-308">For example, suppose the original resource has the following JSON representation:</span></span>

```json
{
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

<span data-ttu-id="9398c-309">このリソースの使用可能な JSON マージ パッチを次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-309">Here is a possible JSON merge patch for this resource:</span></span>

```json
{
    "price":12,
    "color":null,
    "size":"small"
}
```

<span data-ttu-id="9398c-310">これは、`price` を更新し、`color` を削除し、`size` を追加するようサーバーに指示しています。&mdash; `name` と `category` は変更されません。</span><span class="sxs-lookup"><span data-stu-id="9398c-310">This tells the server to update `price`, delete `color`, and add `size` &mdash; `name` and `category` are not modified.</span></span> <span data-ttu-id="9398c-311">JSON マージ パッチの詳細については、[RFC 7396](https://tools.ietf.org/html/rfc7396) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-311">For the exact details of JSON merge patch, see [RFC 7396](https://tools.ietf.org/html/rfc7396).</span></span> <span data-ttu-id="9398c-312">JSON マージ パッチのメディアの種類は `application/merge-patch+json` です。</span><span class="sxs-lookup"><span data-stu-id="9398c-312">The media type for JSON merge patch is `application/merge-patch+json`.</span></span>

<span data-ttu-id="9398c-313">マージ パッチでは、パッチ ドキュメントの `null` に特別な意味があるため、元のリソースに明示的な null 値を含めることができる場合、マージ パッチは適していません。</span><span class="sxs-lookup"><span data-stu-id="9398c-313">Merge patch is not suitable if the original resource can contain explicit null values, due to the special meaning of `null` in the patch document.</span></span> <span data-ttu-id="9398c-314">また、パッチ ドキュメントでは、サーバーが更新を適用する順序は指定されていません。</span><span class="sxs-lookup"><span data-stu-id="9398c-314">Also, the patch document doesn't specify the order that the server should apply the updates.</span></span> <span data-ttu-id="9398c-315">データとドメインによっては、これが重要な場合とそうでない場合があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-315">That may or may not matter, depending on the data and the domain.</span></span> <span data-ttu-id="9398c-316">[RFC 6902](https://tools.ietf.org/html/rfc6902) で定義されている JSON パッチの方が柔軟性に優れています。</span><span class="sxs-lookup"><span data-stu-id="9398c-316">JSON patch, defined in [RFC 6902](https://tools.ietf.org/html/rfc6902), is more flexible.</span></span> <span data-ttu-id="9398c-317">適用する一連の操作として変更を指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-317">It specifies the changes as a sequence of operations to apply.</span></span> <span data-ttu-id="9398c-318">操作には、追加、削除、置換、コピー、テスト (値の検証) があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-318">Operations include add, remove, replace, copy, and test (to validate values).</span></span> <span data-ttu-id="9398c-319">JSON パッチのメディアの種類は `application/json-patch+json` です。</span><span class="sxs-lookup"><span data-stu-id="9398c-319">The media type for JSON patch is `application/json-patch+json`.</span></span>

<span data-ttu-id="9398c-320">PATCH 要求を処理するときに発生する可能性のある一般的なエラー状態と、該当する HTTP 状態コードを次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-320">Here are some typical error conditions that might be encountered when processing a PATCH request, along with the appropriate HTTP status code.</span></span>

| <span data-ttu-id="9398c-321">エラー状態</span><span class="sxs-lookup"><span data-stu-id="9398c-321">Error condition</span></span> | <span data-ttu-id="9398c-322">HTTP 状態コード</span><span class="sxs-lookup"><span data-stu-id="9398c-322">HTTP status code</span></span> |
|-----------|------------|
| <span data-ttu-id="9398c-323">パッチ ドキュメントの形式がサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="9398c-323">The patch document format isn't supported.</span></span> | <span data-ttu-id="9398c-324">415 (Unsupported Media Type)</span><span class="sxs-lookup"><span data-stu-id="9398c-324">415 (Unsupported Media Type)</span></span> |
| <span data-ttu-id="9398c-325">無効な形式のパッチ ドキュメントです。</span><span class="sxs-lookup"><span data-stu-id="9398c-325">Malformed patch document.</span></span> | <span data-ttu-id="9398c-326">400 (Bad Request)</span><span class="sxs-lookup"><span data-stu-id="9398c-326">400 (Bad Request)</span></span> |
| <span data-ttu-id="9398c-327">パッチ ドキュメントは有効ですが、現在の状態のリソースに変更を適用することはできません。</span><span class="sxs-lookup"><span data-stu-id="9398c-327">The patch document is valid, but the changes can't be applied to the resource in its current state.</span></span> | <span data-ttu-id="9398c-328">409 (Conflict)</span><span class="sxs-lookup"><span data-stu-id="9398c-328">409 (Conflict)</span></span>

### <a name="delete-methods"></a><span data-ttu-id="9398c-329">DELETE メソッド</span><span class="sxs-lookup"><span data-stu-id="9398c-329">DELETE methods</span></span>

<span data-ttu-id="9398c-330">削除操作が成功すると、Web サーバーは HTTP 状態コード 204 で応答します。これは、プロセスは正常に処理されたが、応答本文にそれ以上の情報は含まれていないことを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-330">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="9398c-331">リソースが存在しない場合、Web サーバーは HTTP 404 (Not Found) を返すことができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-331">If the resource doesn't exist, the web server can return HTTP 404 (Not Found).</span></span>

### <a name="asynchronous-operations"></a><span data-ttu-id="9398c-332">非同期操作</span><span class="sxs-lookup"><span data-stu-id="9398c-332">Asynchronous operations</span></span>

<span data-ttu-id="9398c-333">POST、PUT、PATCH、または DELETE 操作では、完了までに時間のかかる処理が必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-333">Sometimes a POST, PUT, PATCH, or DELETE operation might require processing that takes awhile to complete.</span></span> <span data-ttu-id="9398c-334">完了を待ってからクライアントに応答を送信すると、許容できない待機時間が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-334">If you wait for completion before sending a response to the client, it may cause unacceptable latency.</span></span> <span data-ttu-id="9398c-335">その場合は、操作を非同期にすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-335">If so, consider making the operation asynchronous.</span></span> <span data-ttu-id="9398c-336">HTTP 状態コード 202 (Accepted) を返して、要求が処理のために受理されたが、まだ完了していないことを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-336">Return HTTP status code 202 (Accepted) to indicate the request was accepted for processing but is not completed.</span></span>

<span data-ttu-id="9398c-337">クライアントが状態エンドポイントをポーリングして状態を監視できるように、非同期要求の状態を返すエンドポイントを公開する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-337">You should expose an endpoint that returns the status of an asynchronous request, so the client can monitor the status by polling the status endpoint.</span></span> <span data-ttu-id="9398c-338">202 応答の Location ヘッダーに、状態エンドポイントの URI を含めます。</span><span class="sxs-lookup"><span data-stu-id="9398c-338">Include the URI of the status endpoint in the Location header of the 202 response.</span></span> <span data-ttu-id="9398c-339">例: </span><span class="sxs-lookup"><span data-stu-id="9398c-339">For example:</span></span>

```HTTP
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

<span data-ttu-id="9398c-340">クライアントがこのエンドポイントに GET 要求を送信した場合、応答に要求の現在の状態を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-340">If the client sends a GET request to this endpoint, the response should contain the current status of the request.</span></span> <span data-ttu-id="9398c-341">必要に応じて、完了までの推定時間や操作を取り消すためのリンクを含めることもできます。</span><span class="sxs-lookup"><span data-stu-id="9398c-341">Optionally, it could also include an estimated time to completion or a link to cancel the operation.</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

<span data-ttu-id="9398c-342">非同期操作で新しいリソースを作成する場合、状態エンドポイントは、操作の完了後に状態コード 303 (See Other) を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-342">If the asynchronous operation creates a new resource, the status endpoint should return status code 303 (See Other) after the operation completes.</span></span> <span data-ttu-id="9398c-343">303 応答には、新しいリソースの URI を示す Location ヘッダーを含めます。</span><span class="sxs-lookup"><span data-stu-id="9398c-343">In the 303 response, include a Location header that gives the URI of the new resource:</span></span>

```HTTP
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

<span data-ttu-id="9398c-344">詳細については、「[Asynchronous operations in REST (REST での非同期操作)](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="9398c-344">For more information, see [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/).</span></span>

## <a name="filter-and-paginate-data"></a><span data-ttu-id="9398c-345">データのフィルター処理とページング処理</span><span class="sxs-lookup"><span data-stu-id="9398c-345">Filter and paginate data</span></span>

<span data-ttu-id="9398c-346">単一の URI を使用してリソースのコレクションを公開すると、情報のサブセットだけが必要な場合に、アプリケーションが大量のデータをフェッチする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-346">Exposing a collection of resources through a single URI can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="9398c-347">たとえば、クライアント アプリケーションが、コストが特定の値を超えるすべての注文を検索する必要があるとします。</span><span class="sxs-lookup"><span data-stu-id="9398c-347">For example, suppose a client application needs to find all orders with a cost over a specific value.</span></span> <span data-ttu-id="9398c-348">*/orders* URI からすべての注文を取得し、クライアント側でこれらの注文をフィルター処理します。</span><span class="sxs-lookup"><span data-stu-id="9398c-348">It might retrieve all orders from the */orders* URI and then filter these orders on the client side.</span></span> <span data-ttu-id="9398c-349">このプロセスが非常に非効率的であることは明らかです。</span><span class="sxs-lookup"><span data-stu-id="9398c-349">Clearly this process is highly inefficient.</span></span> <span data-ttu-id="9398c-350">Web API をホストするサーバーのネットワーク帯域幅と処理能力が無駄になります。</span><span class="sxs-lookup"><span data-stu-id="9398c-350">It wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="9398c-351">代わりに、API は URI のクエリ文字列でフィルター (例: */orders?minCost=n*) を渡すことを許可することができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-351">Instead, the API can allow passing a filter in the query string of the URI, such as */orders?minCost=n*.</span></span> <span data-ttu-id="9398c-352">Web API は、クエリ文字列の `minCost` パラメーターを解析して処理し、サーバー側でフィルター処理された結果を返す役割を担います。</span><span class="sxs-lookup"><span data-stu-id="9398c-352">The web API is then responsible for parsing and handling the `minCost` parameter in the query string and returning the filtered results on the server side.</span></span>

<span data-ttu-id="9398c-353">コレクション リソースに対する GET 要求では、多数の項目が返される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-353">GET requests over collection resources can potentially return a large number of items.</span></span> <span data-ttu-id="9398c-354">1 つの要求で返されるデータの量を制限するように Web API を設計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-354">You should design a web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="9398c-355">取得する項目の最大数とコレクションの開始オフセットを指定するクエリ文字列のサポートを検討します。</span><span class="sxs-lookup"><span data-stu-id="9398c-355">Consider supporting query strings that specify the maximum number of items to retrieve and a starting offset into the collection.</span></span> <span data-ttu-id="9398c-356">例: </span><span class="sxs-lookup"><span data-stu-id="9398c-356">For example:</span></span>

```HTTP
/orders?limit=25&offset=50
```

<span data-ttu-id="9398c-357">サービス拒否攻撃を防ぐために、返される項目の数に上限を課すことも検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-357">Also consider imposing an upper limit on the number of items returned, to help prevent Denial of Service attacks.</span></span> <span data-ttu-id="9398c-358">クライアント アプリケーションを支援するには、改ページ調整されたデータを返す GET 要求に、コレクションで利用できるリソースの合計を示す何らかの形式のメタデータも含まれるようにします。</span><span class="sxs-lookup"><span data-stu-id="9398c-358">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span>

<span data-ttu-id="9398c-359">同様の方法を使用して、フェッチされたデータを並べ替えることができます。*/orders?sort=ProductID* のように、フィールド名を値として取得する並べ替えパラメーターを指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-359">You can use a similar strategy to sort data as it is fetched, by providing a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="9398c-360">ただし、クエリ文字列パラメーターは、キャッシュ データのキーとして多くのキャッシュ実装で使用されるリソース識別子の一部となるため、この方法はキャッシュに悪影響を与える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-360">However, this approach can have a negative effect on caching, because query string parameters form part of the resource identifier used by many cache implementations as the key to cached data.</span></span>

<span data-ttu-id="9398c-361">各項目に大量のデータが含まれている場合、返されるフィールドを項目ごとに制限するように、この方法を拡張できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-361">You can extend this approach to limit the fields returned for each item, if each item contains a large amount of data.</span></span> <span data-ttu-id="9398c-362">たとえば、*/orders?fields=ProductID,Quantity* など、コンマ区切りのフィールド一覧を受け取るクエリ文字列パラメーターを使用することができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-362">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span>

<span data-ttu-id="9398c-363">クエリ文字列内の省略可能なすべてのパラメーターに、意味のある既定値を設定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-363">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="9398c-364">たとえば、改ページ調整を実装する場合は `limit` パラメーターを 10、`offset` パラメーターを 0 に設定し、順序付けを実装する場合はリソースのキーに並べ替えパラメーターを設定し、プロジェクションをサポートする場合は`fields` パラメーターをリソースのすべてのフィールドに設定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-364">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>

## <a name="support-partial-responses-for-large-binary-resources"></a><span data-ttu-id="9398c-365">大きなバイナリ リソースの部分的な応答のサポート</span><span class="sxs-lookup"><span data-stu-id="9398c-365">Support partial responses for large binary resources</span></span>

<span data-ttu-id="9398c-366">リソースに大きなバイナリ フィールド (ファイルや画像など) が含まれている場合があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-366">A resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="9398c-367">信頼性の低い断続的な接続に起因する問題を克服し、応答時間を向上させるには、このようなリソースをチャンクで取得できるようにすることを検討します。</span><span class="sxs-lookup"><span data-stu-id="9398c-367">To overcome problems caused by unreliable and intermittent connections and to improve response times, consider enabling such resources to be retrieved in chunks.</span></span> <span data-ttu-id="9398c-368">そのためには、Web API が大きなリソースの GET 要求で Accept-Ranges ヘッダーをサポートする必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-368">To do this, the web API should support the Accept-Ranges header for GET requests for large resources.</span></span> <span data-ttu-id="9398c-369">このヘッダーは、GET 操作が部分的な要求をサポートすることを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-369">This header indicates that the GET operation supports partial requests.</span></span> <span data-ttu-id="9398c-370">クライアント アプリケーションは、バイト範囲として指定されたリソースのサブセットを返す GET 要求を送信できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-370">The client application can submit GET requests that return a subset of a resource, specified as a range of bytes.</span></span>

<span data-ttu-id="9398c-371">これらのリソースの HTTP HEAD 要求を実装することも検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-371">Also, consider implementing HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="9398c-372">HEAD 要求は GET 要求に似ていますが、空のメッセージ本文と共に、リソースを示す HTTP ヘッダーだけを返す点が異なります。</span><span class="sxs-lookup"><span data-stu-id="9398c-372">A HEAD request is similar to a GET request, except that it only returns the HTTP headers that describe the resource, with an empty message body.</span></span> <span data-ttu-id="9398c-373">クライアント アプリケーションは HEAD 要求を発行し、部分的 GET 要求を使用して、リソースを取得するかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="9398c-373">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="9398c-374">例: </span><span class="sxs-lookup"><span data-stu-id="9398c-374">For example:</span></span>

```HTTP
HEAD https://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

<span data-ttu-id="9398c-375">応答メッセージの例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-375">Here is an example response message:</span></span>

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

<span data-ttu-id="9398c-376">Content-Length ヘッダーはリソースの合計サイズを示し、Accept-Ranges ヘッダーは対応する GET 操作が部分的な結果をサポートすることを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-376">The Content-Length header gives the total size of the resource, and the Accept-Ranges header indicates that the corresponding GET operation supports partial results.</span></span> <span data-ttu-id="9398c-377">クライアント アプリケーションは、この情報を使用して画像を小さなチャンクで取得できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-377">The client application can use this information to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="9398c-378">最初の要求では、Range ヘッダーを使用して最初の 2,500 バイトを取得します。</span><span class="sxs-lookup"><span data-stu-id="9398c-378">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET https://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

<span data-ttu-id="9398c-379">応答メッセージは、HTTP 状態コード 206 を返すことで、これが部分的な応答であることを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-379">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="9398c-380">Content-Length ヘッダーはメッセージ本文で返される実際のバイト数を指定し (リソースのサイズではない)、Content-Range ヘッダーはこれがどの部分のリソースであるかを示します (4,580 のうち 0 ～ 2499 バイト):</span><span class="sxs-lookup"><span data-stu-id="9398c-380">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

<span data-ttu-id="9398c-381">クライアント アプリケーションのその後の要求で、リソースの残りの部分を取得できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-381">A subsequent request from the client application can retrieve the remainder of the resource.</span></span>

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a><span data-ttu-id="9398c-382">HATEOAS を使用した関連リソースへのナビゲーションの実現</span><span class="sxs-lookup"><span data-stu-id="9398c-382">Use HATEOAS to enable navigation to related resources</span></span>

<span data-ttu-id="9398c-383">REST の背後にある主な動機の 1 つは、URI スキームの事前知識を必要とせずに、リソースのセット全体を移動できることです。</span><span class="sxs-lookup"><span data-stu-id="9398c-383">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="9398c-384">各 HTTP GET 要求は応答に含まれるハイパーリンクより、要求したオブジェクトに直接関連するリソースを検索するのに必要な情報を返し、これらの各リソースで使用可能な操作を記述する情報も提供されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-384">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="9398c-385">この原則は HATEOAS、つまり アプリケーション状態のエンジンとしてのハイパーテキストとして知られます。</span><span class="sxs-lookup"><span data-stu-id="9398c-385">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="9398c-386">システムは効率的に Finite State Machine であり、各要求への応答には、ある状態を別の状態に移すのに必要な情報が含まれています。他の情報は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-386">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="9398c-387">現在、HATEOAS の原則をモデル化する方法を定義する標準や仕様はありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-387">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="9398c-388">このセクションで示されている例は、可能性のある一つのソリューションを示しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-388">The examples shown in this section illustrate one possible solution.</span></span>

<span data-ttu-id="9398c-389">たとえば、注文と顧客の関係を処理するために、注文の表現にその注文の顧客に対して使用できる操作を示すリンクを含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-389">For example, to handle the relationship between an order and a customer, the representation of an order could include links that identify the available operations for the customer of the order.</span></span> <span data-ttu-id="9398c-390">使用可能な表現を次に示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-390">Here is a possible representation:</span></span>

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"DELETE",
      "types":[]
    }]
}
```

<span data-ttu-id="9398c-391">この例では、`links` 配列に一連のリンクが含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-391">In this example, the `links` array has a set of links.</span></span> <span data-ttu-id="9398c-392">各リンクは、関連エンティティに対する操作を表しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-392">Each link represents an operation on a related entity.</span></span> <span data-ttu-id="9398c-393">各リンクのデータには、関係 ("customer")、URI (`https://adventure-works.com/customers/3`)、HTTP メソッド、サポートされる MIME の種類が含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-393">The data for each link includes the relationship ("customer"), the URI (`https://adventure-works.com/customers/3`), the HTTP method, and the supported MIME types.</span></span> <span data-ttu-id="9398c-394">クライアント アプリケーションが操作を呼び出すことができるようにするために必要な情報はこれですべてです。</span><span class="sxs-lookup"><span data-stu-id="9398c-394">This is all the information that a client application needs to be able to invoke the operation.</span></span>

<span data-ttu-id="9398c-395">`links` 配列には、取得したリソース自体に関する自己参照型の情報も含まれています。</span><span class="sxs-lookup"><span data-stu-id="9398c-395">The `links` array also includes self-referencing information about the resource itself that has been retrieved.</span></span> <span data-ttu-id="9398c-396">これらの情報の関係は *self* です。</span><span class="sxs-lookup"><span data-stu-id="9398c-396">These have the relationship *self*.</span></span>

<span data-ttu-id="9398c-397">返されるリンクのセットは、リソースの状態によって変わる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-397">The set of links that are returned may change, depending on the state of the resource.</span></span> <span data-ttu-id="9398c-398">"アプリケーション状態のエンジン" であるハイパーリンクとはこのことです。</span><span class="sxs-lookup"><span data-stu-id="9398c-398">This is what is meant by hypertext being the "engine of application state."</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="9398c-399">RESTful Web API のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="9398c-399">Versioning a RESTful web API</span></span>

<span data-ttu-id="9398c-400">Web API が更新されないことはありえません。</span><span class="sxs-lookup"><span data-stu-id="9398c-400">It is highly unlikely that a web API will remain static.</span></span> <span data-ttu-id="9398c-401">ビジネスの要求は変化するため、新しいリソースのコレクションが加わり、リソース間のリレーションシップが変化し、リソース内のデータ構造が修正される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-401">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="9398c-402">新しいまたは異なる要件を処理するために Web API を更新することは比較的簡単なプロセスですが、そのような変化が Web API を消費するクライアント アプリケーションに対して与える影響を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-402">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="9398c-403">問題なのは、Web API を設計および実装している開発者はその API を完全に制御できるものの、リモートで操作しているサードパーティの組織により構築されたクライアント アプリケーションを、その開発者が同程度には制御できないことです。</span><span class="sxs-lookup"><span data-stu-id="9398c-403">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="9398c-404">まず必要なのは、新しいクライアント アプリケーションが新しい機能やリソースを活用できるようにしながら、既存のアプリケーションに変更なく機能を続行させることです。</span><span class="sxs-lookup"><span data-stu-id="9398c-404">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="9398c-405">バージョン管理により Web API は公開する機能およびリソースを示すことができ、クライアント アプリケーションは特定のバージョンの機能またはリソースへの要求を送信できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-405">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="9398c-406">次のセクションではいくつかの方法について説明しますが、それぞれに独自の利点とトレードオフがあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-406">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="9398c-407">バージョン管理なし</span><span class="sxs-lookup"><span data-stu-id="9398c-407">No versioning</span></span>

<span data-ttu-id="9398c-408">これは最も単純な方法で、一部の内部 API で許容されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-408">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="9398c-409">大きな変更は新しいリソースまたは新しいリンクとして示されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-409">Big changes could be represented as new resources or new links.</span></span> <span data-ttu-id="9398c-410">既存のリソースにコンテンツを追加しても、このコンテンツの表示を想定していないクライアント アプリケーションはそれを無視するだけであるため、重大な変更はありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-410">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="9398c-411">たとえば、URI `https://adventure-works.com/customers/3` への要求により、次のクライアント アプリケーションにより期待される `id`、`name`、および `address` フィールドを含む単一の顧客に関する詳細が返されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-411">For example, a request to the URI `https://adventure-works.com/customers/3` should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="9398c-412">わかりやすくするために、このセクションで示す応答例には HATEOAS リンクは含まれていません。</span><span class="sxs-lookup"><span data-stu-id="9398c-412">For simplicity, the example responses shown in this section do not include HATEOAS links.</span></span>

<span data-ttu-id="9398c-413">`DateCreated` フィールドが顧客リソースのスキーマに追加される場合、応答は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="9398c-413">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="9398c-414">既存のクライアント アプリケーションが認識されていないフィールドを無視できる場合は、正常に機能を続行する場合がありますが、新しいクライアント アプリケーションではこの新しいフィールドを処理するように設計できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-414">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="9398c-415">とはいえ、リソースのスキーマにさらに重大な変更があるか (フィールドの削除や名前の変更など)、リソース間のリレーションシップが変更される場合、これらは既存のクライアント アプリケーションが正常に機能できなくなる重大な変更となる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-415">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="9398c-416">このような状況では、次の方法のいずれかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9398c-416">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="9398c-417">URI のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="9398c-417">URI versioning</span></span>

<span data-ttu-id="9398c-418">Web API を変更するか、リソースのスキーマを変更するたびに、バージョン番号を各リソースの URI に追加します。</span><span class="sxs-lookup"><span data-stu-id="9398c-418">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="9398c-419">既存の URI はこれまでと同様に動作を続け、元のスキーマに準拠するリソースを返します。</span><span class="sxs-lookup"><span data-stu-id="9398c-419">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="9398c-420">前の例を拡張し、`address` フィールドをアドレスの各構成部分 (`streetAddress`、`city`、`state`、`zipCode` など) を含むサブフィールドに再構築する場合、このバージョンのリソースはバージョン番号を含む `https://adventure-works.com/v2/customers/3` などの URI より公開できます</span><span class="sxs-lookup"><span data-stu-id="9398c-420">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as `https://adventure-works.com/v2/customers/3`:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="9398c-421">このバージョン管理メカニズムは非常に単純ですが、適切なエンドポイントに要求をルーティングするサーバーにより変わります。</span><span class="sxs-lookup"><span data-stu-id="9398c-421">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="9398c-422">とはいえ、Web API は何度も繰り返すことで成熟し、サーバーは多くの異なるバージョンをサポートする必要があるため、扱いにくくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-422">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="9398c-423">また、純正主義者の観点からすると、すべての場合でクライアント アプリケーションは同じデータを取得しているため (顧客 3)、URI はバージョンによってそれほど異なるべきではありません。</span><span class="sxs-lookup"><span data-stu-id="9398c-423">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="9398c-424">すべてのリンクで URI にバージョン番号が含まれる必要があるため、このスキームによっても HATEOAS の実装が複雑になります。</span><span class="sxs-lookup"><span data-stu-id="9398c-424">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="9398c-425">クエリ文字列のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="9398c-425">Query string versioning</span></span>

<span data-ttu-id="9398c-426">複数の URI を提供するのではなく、`https://adventure-works.com/customers/3?version=2` など、HTTP 要求に追加されたクエリ文字列内のパラメーターを使用してリソースのバージョンを指定できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-426">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as `https://adventure-works.com/customers/3?version=2`.</span></span> <span data-ttu-id="9398c-427">バージョン パラメーターがより古いクライアント アプリケーションで省略される場合、1 などの有効な値を既定にします。</span><span class="sxs-lookup"><span data-stu-id="9398c-427">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="9398c-428">この方法には同じリソースからは常に同じ URI が取得されるというセマンティックな利点がありますが、クエリ文字列を解析し、適切な HTTP 応答を返送する要求を処理するコードにより異なります。</span><span class="sxs-lookup"><span data-stu-id="9398c-428">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="9398c-429">この方法は、URI のバージョン管理メカニズムとして HATEOAS を実装する同様の複雑さによっても影響されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-429">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="9398c-430">一部の古い Web ブラウザーや Web プロキシは、URI にクエリ文字列を含む要求の応答をキャッシュしません。</span><span class="sxs-lookup"><span data-stu-id="9398c-430">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URI.</span></span> <span data-ttu-id="9398c-431">これは、Web API を使用し、そのような Web ブラウザー内から実行する Web アプリケーションのパフォーマンスに悪影響を与える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-431">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>

### <a name="header-versioning"></a><span data-ttu-id="9398c-432">ヘッダーのバージョン管理</span><span class="sxs-lookup"><span data-stu-id="9398c-432">Header versioning</span></span>

<span data-ttu-id="9398c-433">バージョン番号をクエリ文字列パラメーターとして追加するのではなく、リソースのバージョンを示すカスタム ヘッダーを実装できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-433">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="9398c-434">この方法ではクライアント アプリケーションにより任意の要求に適切なヘッダーを追加する必要がありますが、バージョン ヘッダーが省略されている場合、クライアントの要求を処理するコードは既定値 (バージョン 1) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-434">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="9398c-435">次の例では、*Custom-Header* という名前のカスタム ヘッダーを使用します。</span><span class="sxs-lookup"><span data-stu-id="9398c-435">The following examples use a custom header named *Custom-Header*.</span></span> <span data-ttu-id="9398c-436">このヘッダーの値は、Web API のバージョンを示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-436">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="9398c-437">バージョン 1: </span><span class="sxs-lookup"><span data-stu-id="9398c-437">Version 1:</span></span>

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="9398c-438">バージョン 2: </span><span class="sxs-lookup"><span data-stu-id="9398c-438">Version 2:</span></span>

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="9398c-439">前の 2 つの方法と同様、HATEOAS の実装では任意のリンクに適切なカスタム ヘッダーを追加することが必要です。</span><span class="sxs-lookup"><span data-stu-id="9398c-439">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="9398c-440">メディアの種類のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="9398c-440">Media type versioning</span></span>

<span data-ttu-id="9398c-441">クライアント アプリケーションが Web サーバーに HTTP GET 要求を送信する場合、このガイダンスで既に説明したように、Accept ヘッダーを使用して処理できるコンテンツの書式を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="9398c-441">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="9398c-442">多くの場合、*Accept* ヘッダーの目的は、応答の本文が XML、JSON、またはクライアントが解析可能な他の一般的な形式であるかどうかをクライアント アプリケーションが指定できるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="9398c-442">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="9398c-443">とはいえ、想定しているリソースのバージョンをクライアント アプリケーションが示すことができるようにする情報を含むカスタム メディアの種類を定義できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-443">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="9398c-444">次の例は、*application/vnd.adventure-works.v1+json* の値を含む *Accept* ヘッダーを指定する要求を示します。</span><span class="sxs-lookup"><span data-stu-id="9398c-444">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="9398c-445">*vnd.adventure-works.v1* 要素は Web サーバーに対し、バージョン 1 のリソースを返すように指示しますが、*json* 要素は応答本文の形式が JSON であるように指定します。</span><span class="sxs-lookup"><span data-stu-id="9398c-445">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

<span data-ttu-id="9398c-446">要求を処理しているコードは、*Accept* ヘッダーを処理し、可能な限りそれを使用します (クライアント アプリケーションは複数の形式を *Accept* ヘッダーで指定する場合があり、その場合 Web サーバーは最も適切な形式を応答本文に選択できます)。</span><span class="sxs-lookup"><span data-stu-id="9398c-446">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="9398c-447">Web サーバーは次のように Content-Type ヘッダーを使用することで、応答本文のデータの形式を確認します。</span><span class="sxs-lookup"><span data-stu-id="9398c-447">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="9398c-448">Accept ヘッダーにより既知のメディアの種類が指定されない場合、Web サーバーは HTTP 406 (Not Acceptable) 応答メッセージを生成するか、既定のメディアの種類でメッセージを返すことができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-448">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="9398c-449">この方法が最も純粋と思われるバージョン管理メカニズムで、当然ながら HATEOAS で役立ち、リソース リンクの関連データの MIME の種類を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="9398c-449">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="9398c-450">バージョン管理の方法を選択すると、パフォーマンスに対する影響、特に Web サーバーでのキャッシュについて検討する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="9398c-450">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="9398c-451">同じ URI/クエリ文字列の組み合わせは同じデータを毎回参照するため、URI バージョン管理および Query String バージョン管理スキームはキャッシュに適しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-451">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="9398c-452">通常、Header バージョン管理および Media Type バージョン管理メカニズムでは、カスタム ヘッダーまたは Accept ヘッダーの値を確認するのに追加のロジックが必要です。</span><span class="sxs-lookup"><span data-stu-id="9398c-452">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="9398c-453">大規模な環境では、多くのクライアントで異なるバージョンの Web API が使用されているため、サーバー側のキャッシュの重複データ量が著しく増加します。</span><span class="sxs-lookup"><span data-stu-id="9398c-453">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="9398c-454">この問題が深刻になるのは、クライアント アプリケーションがキャッシュを実装するプロキシを経由して Web サーバーと通信する場合です。また、キャッシュに要求されたデータのコピーが現在保持されていない場合は Web サーバーに要求の転送のみ行います。</span><span class="sxs-lookup"><span data-stu-id="9398c-454">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>

## <a name="open-api-initiative"></a><span data-ttu-id="9398c-455">Open API イニシアチブ</span><span class="sxs-lookup"><span data-stu-id="9398c-455">Open API Initiative</span></span>

<span data-ttu-id="9398c-456">[Open API イニシアチブ](https://www.openapis.org/)は、ベンダー間の REST API の記述を標準化するために業界コンソーシアムによって作成されました。</span><span class="sxs-lookup"><span data-stu-id="9398c-456">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="9398c-457">このイニシアチブの一環として、Swagger 2.0 仕様という名前が OpenAPI 仕様 (OAS) に変更され、Open API イニシアチブの下に配置されました。</span><span class="sxs-lookup"><span data-stu-id="9398c-457">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="9398c-458">Web API に OpenAPI を採用することもできます。</span><span class="sxs-lookup"><span data-stu-id="9398c-458">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="9398c-459">考慮すべき点:</span><span class="sxs-lookup"><span data-stu-id="9398c-459">Some points to consider:</span></span>

- <span data-ttu-id="9398c-460">OpenAPI 仕様には、REST API の設計方法に関する一連の厳密なガイドラインが付属しています。</span><span class="sxs-lookup"><span data-stu-id="9398c-460">The OpenAPI Specification comes with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="9398c-461">相互運用性の利点はありますが、API を設計するときは、仕様に準拠するようにさらに注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="9398c-461">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>

- <span data-ttu-id="9398c-462">OpenAPI では、実装優先のアプローチではなく、コントラクト優先のアプローチが推進されます。</span><span class="sxs-lookup"><span data-stu-id="9398c-462">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="9398c-463">コントラクト優先とは、API コントラクト (インターフェイス) を最初に設計してから、コントラクトを実装するコードを記述することを意味します。</span><span class="sxs-lookup"><span data-stu-id="9398c-463">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span>

- <span data-ttu-id="9398c-464">Swagger などのツールにより、API コントラクトからクライアント ライブラリまたはドキュメントを生成できます。</span><span class="sxs-lookup"><span data-stu-id="9398c-464">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="9398c-465">例については、[Swagger を使用する ASP.NET Web API のヘルプ ページ](/aspnet/core/tutorials/web-api-help-pages-using-swagger)に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="9398c-465">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="9398c-466">詳細情報</span><span class="sxs-lookup"><span data-stu-id="9398c-466">More information</span></span>

- <span data-ttu-id="9398c-467">[Microsoft REST API のガイドライン](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md): </span><span class="sxs-lookup"><span data-stu-id="9398c-467">[Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md).</span></span> <span data-ttu-id="9398c-468">パブリック REST API の設計に関する詳細な推奨事項。</span><span class="sxs-lookup"><span data-stu-id="9398c-468">Detailed recommendations for designing public REST APIs.</span></span>

- <span data-ttu-id="9398c-469">[Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/): </span><span class="sxs-lookup"><span data-stu-id="9398c-469">[Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/).</span></span> <span data-ttu-id="9398c-470">Web API を設計および実装するときに検討すべき項目の便利な一覧。</span><span class="sxs-lookup"><span data-stu-id="9398c-470">A useful list of items to consider when designing and implementing a web API.</span></span>

- <span data-ttu-id="9398c-471">[Open API イニシアチブ](https://www.openapis.org/): </span><span class="sxs-lookup"><span data-stu-id="9398c-471">[Open API Initiative](https://www.openapis.org/).</span></span> <span data-ttu-id="9398c-472">Open API に関するドキュメントと実装の詳細。</span><span class="sxs-lookup"><span data-stu-id="9398c-472">Documentation and implementation details on Open API.</span></span>
