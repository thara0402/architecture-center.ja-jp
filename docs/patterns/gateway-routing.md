---
title: ゲートウェイ ルーティング パターン
description: 単一のエンドポイントを使用して複数のサービスに要求をルーティングします。
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 53239b23cfd98fad1edc38ca37c2274d5a9d7a0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="df239-103">ゲートウェイ ルーティング パターン</span><span class="sxs-lookup"><span data-stu-id="df239-103">Gateway Routing pattern</span></span>

<span data-ttu-id="df239-104">単一のエンドポイントを使用して複数のサービスに要求をルーティングします。</span><span class="sxs-lookup"><span data-stu-id="df239-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="df239-105">このパターンは、単一のエンドポイントで複数のサービスを公開し、要求に基づいて適切なサービスにルーティングする場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="df239-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="df239-106">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="df239-106">Context and problem</span></span>

<span data-ttu-id="df239-107">クライアントが複数のサービスを使用する必要がある場合、サービスごとに個別のエンドポイントを設定し、クライアントが各エンドポイントを管理するのは難しい場合があります。</span><span class="sxs-lookup"><span data-stu-id="df239-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="df239-108">たとえば、電子商取引アプリケーションが、検索、レビュー、カート、チェックアウト、注文履歴などのサービスを提供しているとします。</span><span class="sxs-lookup"><span data-stu-id="df239-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="df239-109">クライアントが対話する必要のある API がサービスごとに異なり、クライアントはサービスに接続するために、各エンドポイントを把握する必要があります。</span><span class="sxs-lookup"><span data-stu-id="df239-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="df239-110">API が変更された場合、クライアントも更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="df239-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="df239-111">サービスを複数の個別のサービスにリファクタリングする場合は、サービスとクライアントの両方でコードを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="df239-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="df239-112">解決策</span><span class="sxs-lookup"><span data-stu-id="df239-112">Solution</span></span>

<span data-ttu-id="df239-113">一連のアプリケーション、サービス、またはデプロイの前にゲートウェイを配置します。</span><span class="sxs-lookup"><span data-stu-id="df239-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="df239-114">アプリケーションのレイヤー 7 ルーティングを使用して、要求を適切なインスタンスにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="df239-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="df239-115">このパターンでは、クライアント アプリケーションは、単一のエンドポイントを把握し、そのエンドポイントと通信するだけで済みます。</span><span class="sxs-lookup"><span data-stu-id="df239-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="df239-116">サービスが統合または分解されても、クライアントを必ずしも更新する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="df239-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="df239-117">引き続きゲートウェイに要求することができ、ルーティングだけが変更されます。</span><span class="sxs-lookup"><span data-stu-id="df239-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="df239-118">また、ゲートウェイにより、クライアントからバックエンド サービスを抽象化できるので、ゲートウェイの背後でバックエンド サービスの変更を可能にしながら、クライアント呼び出しをシンプルに保つことができます。</span><span class="sxs-lookup"><span data-stu-id="df239-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="df239-119">クライアント呼び出しは、クライアントの予想される動作を処理する必要がある 1 つまたは複数のサービスにルーティングできるので、クライアントを変更せずに、ゲートウェイの背後でサービスを追加、分割、再構成できます。</span><span class="sxs-lookup"><span data-stu-id="df239-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="df239-120">このパターンは、更新プログラムをユーザーにロールアウトする方法を管理できるため、デプロイでも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="df239-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="df239-121">サービスの新しいバージョンをデプロイするときは、既存のバージョンと並行してデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="df239-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="df239-122">ルーティングにより、クライアントに提示するサービスのバージョンを制御できるため、更新プログラムのロールアウトが増分、並列、完全のいずれであるかを問わず、さまざまなリリース戦略を使用する柔軟がもたらされます。</span><span class="sxs-lookup"><span data-stu-id="df239-122">Routing let you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="df239-123">新しいサービスのデプロイ後に問題が見つかった場合、ゲートウェイで構成変更を行うことで、クライアントに影響を及ぼすことなく、簡単に元に戻すことができます。</span><span class="sxs-lookup"><span data-stu-id="df239-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="df239-124">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="df239-124">Issues and considerations</span></span>

- <span data-ttu-id="df239-125">ゲートウェイ サービスによって、単一障害点が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="df239-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="df239-126">可用性の要件が満たされるように、適切に設計されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="df239-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="df239-127">実装時の回復性とフォールト トレランス機能を検討してください。</span><span class="sxs-lookup"><span data-stu-id="df239-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="df239-128">ゲートウェイ サービスによって、ボトルネックが生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="df239-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="df239-129">ゲートウェイが負荷を処理できる十分なパフォーマンスを備えており、成長予測に合わせて簡単に拡張できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="df239-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="df239-130">サービスのカスケードのエラーが発生しないように、ゲートウェイに対してロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="df239-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="df239-131">ゲートウェイ ルーティングはレベル 7 です。</span><span class="sxs-lookup"><span data-stu-id="df239-131">Gateway routing is level 7.</span></span> <span data-ttu-id="df239-132">IP、ポート、ヘッダー、または URL に基づくことができます。</span><span class="sxs-lookup"><span data-stu-id="df239-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="df239-133">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="df239-133">When to use this pattern</span></span>

<span data-ttu-id="df239-134">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="df239-134">Use this pattern when:</span></span>

- <span data-ttu-id="df239-135">クライアントが、ゲートウェイの背後でアクセスできる複数のサービスを使用する必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="df239-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="df239-136">単一のエンドポイントを使用することで、クライアント アプリケーションを簡素化する場合。</span><span class="sxs-lookup"><span data-stu-id="df239-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="df239-137">外部でアドレス指定可能なエンドポイントから内部仮想エンドポイントに要求をルーティングする必要がある場合 (VM のポートをクラスターの仮想 IP アドレスに公開する場合など)。</span><span class="sxs-lookup"><span data-stu-id="df239-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="df239-138">このパターンは、1 つか 2 つのサービスしか使用しない単純なアプリケーションには適していないことがあります。</span><span class="sxs-lookup"><span data-stu-id="df239-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="df239-139">例</span><span class="sxs-lookup"><span data-stu-id="df239-139">Example</span></span>

<span data-ttu-id="df239-140">ルーターとして Nginx を使用して、さまざまな仮想ディレクトリに存在するアプリケーションの要求を、それぞれバックエンドの異なるコンピューターにルーティングするサーバーの構成ファイルの簡単な例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="df239-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="df239-141">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="df239-141">Related guidance</span></span>

- [<span data-ttu-id="df239-142">フロントエンド用バックエンド パターン</span><span class="sxs-lookup"><span data-stu-id="df239-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="df239-143">ゲートウェイ集約パターン</span><span class="sxs-lookup"><span data-stu-id="df239-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="df239-144">ゲートウェイ オフロード パターン</span><span class="sxs-lookup"><span data-stu-id="df239-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



