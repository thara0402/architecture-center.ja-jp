---
title: ゲートウェイ ルーティング パターン
titleSuffix: Cloud Design Patterns
description: 単一のエンドポイントを使用して複数のサービスに要求をルーティングします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 4db98038f582e0315a743a55d46013d2eda187e3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010479"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="7de28-104">ゲートウェイ ルーティング パターン</span><span class="sxs-lookup"><span data-stu-id="7de28-104">Gateway Routing pattern</span></span>

<span data-ttu-id="7de28-105">単一のエンドポイントを使用して複数のサービスに要求をルーティングします。</span><span class="sxs-lookup"><span data-stu-id="7de28-105">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="7de28-106">このパターンは、単一のエンドポイントで複数のサービスを公開し、要求に基づいて適切なサービスにルーティングする場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="7de28-106">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="7de28-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="7de28-107">Context and problem</span></span>

<span data-ttu-id="7de28-108">クライアントが複数のサービスを使用する必要がある場合、サービスごとに個別のエンドポイントを設定し、クライアントが各エンドポイントを管理するのは難しい場合があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-108">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="7de28-109">たとえば、電子商取引アプリケーションが、検索、レビュー、カート、チェックアウト、注文履歴などのサービスを提供しているとします。</span><span class="sxs-lookup"><span data-stu-id="7de28-109">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="7de28-110">クライアントが対話する必要のある API がサービスごとに異なり、クライアントはサービスに接続するために、各エンドポイントを把握する必要があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-110">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="7de28-111">API が変更された場合、クライアントも更新する必要があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-111">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="7de28-112">サービスを複数の個別のサービスにリファクタリングする場合は、サービスとクライアントの両方でコードを変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-112">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="7de28-113">解決策</span><span class="sxs-lookup"><span data-stu-id="7de28-113">Solution</span></span>

<span data-ttu-id="7de28-114">一連のアプリケーション、サービス、またはデプロイの前にゲートウェイを配置します。</span><span class="sxs-lookup"><span data-stu-id="7de28-114">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="7de28-115">アプリケーションのレイヤー 7 ルーティングを使用して、要求を適切なインスタンスにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="7de28-115">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="7de28-116">このパターンでは、クライアント アプリケーションは、単一のエンドポイントを把握し、そのエンドポイントと通信するだけで済みます。</span><span class="sxs-lookup"><span data-stu-id="7de28-116">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="7de28-117">サービスが統合または分解されても、クライアントを必ずしも更新する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="7de28-117">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="7de28-118">引き続きゲートウェイに要求することができ、ルーティングだけが変更されます。</span><span class="sxs-lookup"><span data-stu-id="7de28-118">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="7de28-119">また、ゲートウェイにより、クライアントからバックエンド サービスを抽象化できるので、ゲートウェイの背後でバックエンド サービスの変更を可能にしながら、クライアント呼び出しをシンプルに保つことができます。</span><span class="sxs-lookup"><span data-stu-id="7de28-119">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="7de28-120">クライアント呼び出しは、クライアントの予想される動作を処理する必要がある 1 つまたは複数のサービスにルーティングできるので、クライアントを変更せずに、ゲートウェイの背後でサービスを追加、分割、再構成できます。</span><span class="sxs-lookup"><span data-stu-id="7de28-120">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![ゲートウェイ ルーティング パターンの図](./_images/gateway-routing.png)

<span data-ttu-id="7de28-122">このパターンは、更新プログラムをユーザーにロールアウトする方法を管理できるため、デプロイでも役立ちます。</span><span class="sxs-lookup"><span data-stu-id="7de28-122">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="7de28-123">サービスの新しいバージョンをデプロイするときは、既存のバージョンと並行してデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="7de28-123">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="7de28-124">ルーティングにより、クライアントに提示するサービスのバージョンを制御できるため、更新プログラムのロールアウトが増分、並列、完全のいずれであるかを問わず、さまざまなリリース戦略を使用する柔軟がもたらされます。</span><span class="sxs-lookup"><span data-stu-id="7de28-124">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="7de28-125">新しいサービスのデプロイ後に問題が見つかった場合、ゲートウェイで構成変更を行うことで、クライアントに影響を及ぼすことなく、簡単に元に戻すことができます。</span><span class="sxs-lookup"><span data-stu-id="7de28-125">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="7de28-126">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="7de28-126">Issues and considerations</span></span>

- <span data-ttu-id="7de28-127">ゲートウェイ サービスによって、単一障害点が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-127">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="7de28-128">可用性の要件が満たされるように、適切に設計されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="7de28-128">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="7de28-129">実装時の回復性とフォールト トレランス機能を検討してください。</span><span class="sxs-lookup"><span data-stu-id="7de28-129">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="7de28-130">ゲートウェイ サービスによって、ボトルネックが生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="7de28-130">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="7de28-131">ゲートウェイが負荷を処理できる十分なパフォーマンスを備えており、成長予測に合わせて簡単に拡張できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="7de28-131">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="7de28-132">サービスのカスケードのエラーが発生しないように、ゲートウェイに対してロード テストを実行します。</span><span class="sxs-lookup"><span data-stu-id="7de28-132">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="7de28-133">ゲートウェイ ルーティングはレベル 7 です。</span><span class="sxs-lookup"><span data-stu-id="7de28-133">Gateway routing is level 7.</span></span> <span data-ttu-id="7de28-134">IP、ポート、ヘッダー、または URL に基づくことができます。</span><span class="sxs-lookup"><span data-stu-id="7de28-134">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="7de28-135">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="7de28-135">When to use this pattern</span></span>

<span data-ttu-id="7de28-136">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="7de28-136">Use this pattern when:</span></span>

- <span data-ttu-id="7de28-137">クライアントが、ゲートウェイの背後でアクセスできる複数のサービスを使用する必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="7de28-137">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="7de28-138">単一のエンドポイントを使用することで、クライアント アプリケーションを簡素化する場合。</span><span class="sxs-lookup"><span data-stu-id="7de28-138">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="7de28-139">外部でアドレス指定可能なエンドポイントから内部仮想エンドポイントに要求をルーティングする必要がある場合 (VM のポートをクラスターの仮想 IP アドレスに公開する場合など)。</span><span class="sxs-lookup"><span data-stu-id="7de28-139">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="7de28-140">このパターンは、1 つか 2 つのサービスしか使用しない単純なアプリケーションには適していないことがあります。</span><span class="sxs-lookup"><span data-stu-id="7de28-140">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="7de28-141">例</span><span class="sxs-lookup"><span data-stu-id="7de28-141">Example</span></span>

<span data-ttu-id="7de28-142">ルーターとして Nginx を使用して、さまざまな仮想ディレクトリに存在するアプリケーションの要求を、それぞれバックエンドの異なるコンピューターにルーティングするサーバーの構成ファイルの簡単な例を次に示します。</span><span class="sxs-lookup"><span data-stu-id="7de28-142">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```console
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

## <a name="related-guidance"></a><span data-ttu-id="7de28-143">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="7de28-143">Related guidance</span></span>

- [<span data-ttu-id="7de28-144">フロントエンド用バックエンド パターン</span><span class="sxs-lookup"><span data-stu-id="7de28-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- <span data-ttu-id="7de28-145">[Gateway Aggregation pattern](./gateway-aggregation.md) (ゲートウェイ集約パターン)</span><span class="sxs-lookup"><span data-stu-id="7de28-145">[Gateway Aggregation pattern](./gateway-aggregation.md)</span></span>
- [<span data-ttu-id="7de28-146">ゲートウェイ オフロード パターン</span><span class="sxs-lookup"><span data-stu-id="7de28-146">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
