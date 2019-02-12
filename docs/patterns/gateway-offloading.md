---
title: ゲートウェイ オフロード パターン
titleSuffix: Cloud Design Patterns
description: 共有または専用のサービス機能の負荷をゲートウェイ プロキシにオフロードします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 8be05c30ac974b3e58fb0decc52ab623fc5478c8
ms.sourcegitcommit: eee3a35dd5a5a2f0dc117fa1c30f16d6db213ba2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/06/2019
ms.locfileid: "55782032"
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="3ebcb-104">ゲートウェイ オフロード パターン</span><span class="sxs-lookup"><span data-stu-id="3ebcb-104">Gateway Offloading pattern</span></span>

<span data-ttu-id="3ebcb-105">共有または専用のサービス機能の負荷をゲートウェイ プロキシにオフロードします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-105">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="3ebcb-106">このパターンは、SSL 証明書の使用などの共有サービス機能を、アプリケーションの他の部分からゲートウェイに移動することで、アプリケーション開発をシンプルにします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-106">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="3ebcb-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="3ebcb-107">Context and problem</span></span>

<span data-ttu-id="3ebcb-108">一部の機能は複数のサービスでよく使用されます。こうした機能には、構成、管理、およびメンテナンスが必要です。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-108">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="3ebcb-109">すべてのアプリケーションのデプロイで分散される共有または専用のサービスにより、管理オーバーヘッドが増加し、デプロイ エラーの可能性が高くなります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-109">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="3ebcb-110">共有機能に対するすべての更新を、その機能を共有するすべてのサービスにデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-110">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="3ebcb-111">セキュリティの問題 (トークンの検証、暗号化、SSL 証明書の管理) とその他の複雑なタスクを適切に処理するには、チーム メンバーに高度な専門スキルが必要です。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-111">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="3ebcb-112">たとえば、アプリケーションで必要な証明書は、すべてのアプリケーション インスタンスで構成およびデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-112">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="3ebcb-113">新しくデプロイするたびに、証明書の有効期限が切れないように、その証明書を管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-113">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="3ebcb-114">有効期限が切れそうな一般的な証明書は、すべてのアプリケーション デプロイで更新、テスト、および確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-114">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="3ebcb-115">認証、承認、ログ、監視、[調整](./throttling.md)など、他の一般的なサービスについては、デプロイ数が膨大になると、実装および管理するのが困難です。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-115">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="3ebcb-116">オーバーヘッドとエラーの可能性を減らすために、この種類の機能は統合する方がよい場合があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-116">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="3ebcb-117">解決策</span><span class="sxs-lookup"><span data-stu-id="3ebcb-117">Solution</span></span>

<span data-ttu-id="3ebcb-118">一部の機能、特に証明書の管理、認証、SSL 終了、監視、プロトコル変換、調整など、分野横断的な懸念事項を、API ゲートウェイにオフロードします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-118">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span>

<span data-ttu-id="3ebcb-119">次の図は、受信 SSL 接続を終了する API ゲートウェイを示しています。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-119">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="3ebcb-120">これは、元の要求元に代わって、API ゲートウェイの HTTP サーバー アップ ストリームからデータを要求します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-120">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![ゲートウェイ オフロード パターンの図](./_images/gateway-offload.png)

<span data-ttu-id="3ebcb-122">このパターンには次のような利点があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-122">Benefits of this pattern include:</span></span>

- <span data-ttu-id="3ebcb-123">Web サーバー証明書、安全な Web サイトの構成など、関連リソースを配布および管理する必要をなくして、サービス開発をシンプルにします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-123">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="3ebcb-124">構成がシンプルになると、管理が容易になり、スケーラビリティが実現し、サービスのアップグレードも簡単になります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-124">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="3ebcb-125">専用チームが、セキュリティなどの専門的知識を必要とする機能を実装できます。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-125">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="3ebcb-126">これにより、こうした分野横断的で特殊な懸念事項を、関連するエキスパートに任せることができるため、コア チームはアプリケーション機能に集中できます。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-126">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="3ebcb-127">要求と応答のログ記録および監視に対してある程度の一貫性を提供します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-127">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="3ebcb-128">サービスが正しくインストルメント化されていなくても、最小限の監視およびログ記録レベルが確保されるように、ゲートウェイを構成できます。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-128">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="3ebcb-129">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="3ebcb-129">Issues and considerations</span></span>

- <span data-ttu-id="3ebcb-130">API ゲートウェイが高可用性を備え、障害に対する回復性があることを確認します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-130">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="3ebcb-131">API ゲートウェイの複数のインスタンスを実行し、単一障害点をなくします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-131">Avoid single points of failure by running multiple instances of your API gateway.</span></span>
- <span data-ttu-id="3ebcb-132">アプリケーションおよびエンドポイントの容量とスケーリングの要件に対応できるようにゲートウェイが設計されていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-132">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="3ebcb-133">ゲートウェイがアプリケーションのボトルネックになっていないこと、また、十分にスケーラブルであることを確認します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-133">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="3ebcb-134">セキュリティ、データ転送など、アプリケーション全体で使用される機能のみをオフロードします。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-134">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="3ebcb-135">ビジネス ロジックは API ゲートウェイにオフロードしないでください。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-135">Business logic should never be offloaded to the API gateway.</span></span>
- <span data-ttu-id="3ebcb-136">トランザクションを追跡する必要がある場合は、ログ記録のために関連付け ID を生成することを検討します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-136">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="3ebcb-137">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="3ebcb-137">When to use this pattern</span></span>

<span data-ttu-id="3ebcb-138">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-138">Use this pattern when:</span></span>

- <span data-ttu-id="3ebcb-139">アプリケーションのデプロイに、SSL 証明書、暗号化など、共通の懸念事項があるとき。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-139">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="3ebcb-140">アプリケーション デプロイで共通の機能に、さまざまなリソース要件 (メモリ リソース、ストレージ容量、ネットワーク接続など) があるとき。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-140">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="3ebcb-141">ネットワーク セキュリティ、調整、他のネットワーク境界の懸念事項にかかわる問題への対応を、専門チームに任せたいとき。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-141">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="3ebcb-142">サービス間の結合が導入されている場合、このパターンは適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-142">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="3ebcb-143">例</span><span class="sxs-lookup"><span data-stu-id="3ebcb-143">Example</span></span>

<span data-ttu-id="3ebcb-144">次の構成は、Nginx を SSL オフロード アプライアンスとして使用して、受信 SSL 接続を終了し、接続を 3 台のアップストリーム HTTP サーバーのいずれかに分散します。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-144">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```console
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

<span data-ttu-id="3ebcb-145">Azure では、[Application Gateway で SSL 終了を設定する](/azure/application-gateway/tutorial-ssl-cli)ことによってこれを実現できます。</span><span class="sxs-lookup"><span data-stu-id="3ebcb-145">On Azure, this can be achieved by [setting up SSL termination on Application Gateway](/azure/application-gateway/tutorial-ssl-cli).</span></span>

## <a name="related-guidance"></a><span data-ttu-id="3ebcb-146">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="3ebcb-146">Related guidance</span></span>

- [<span data-ttu-id="3ebcb-147">フロントエンド用バックエンド パターン</span><span class="sxs-lookup"><span data-stu-id="3ebcb-147">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="3ebcb-148">ゲートウェイ集約パターン</span><span class="sxs-lookup"><span data-stu-id="3ebcb-148">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- <span data-ttu-id="3ebcb-149">[Gateway Routing pattern](./gateway-routing.md) (ゲートウェイ ルーティング パターン)</span><span class="sxs-lookup"><span data-stu-id="3ebcb-149">[Gateway Routing pattern](./gateway-routing.md)</span></span>
