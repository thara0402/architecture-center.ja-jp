---
title: マイクロサービスの設計パターン
description: 堅牢なマイクロサービス アーキテクチャを実装する設計パターン。
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 8cdbf95c753770910fa4a94384c9809db9fda746
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243573"
---
# <a name="design-patterns-for-microservices"></a><span data-ttu-id="90a32-103">マイクロサービスの設計パターン</span><span class="sxs-lookup"><span data-stu-id="90a32-103">Design patterns for microservices</span></span>

<span data-ttu-id="90a32-104">マイクロサービスの目標は、個別にデプロイできる小さい自律的なサービスにアプリケーションを分解して、アプリケーションのリリース速度を上げることです。</span><span class="sxs-lookup"><span data-stu-id="90a32-104">The goal of microservices is to increase the velocity of application releases, by decomposing the application into small autonomous services that can be deployed independently.</span></span> <span data-ttu-id="90a32-105">マイクロサービス アーキテクチャには、いくつかの課題もあります。</span><span class="sxs-lookup"><span data-stu-id="90a32-105">A microservices architecture also brings some challenges.</span></span> <span data-ttu-id="90a32-106">ここに示す設計パターンは、それらの課題の緩和に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="90a32-106">The design patterns shown here can help mitigate these challenges.</span></span>

![マイクロサービスの設計パターン](../images/microservices-patterns.png)

<span data-ttu-id="90a32-108">[**アンバサダー**](../../patterns/ambassador.md)は、監視、ログ記録、ルーティング、セキュリティ (TLS など) といった一般的なクライアント接続のタスクを言語に関係ない方法でオフロードするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="90a32-108">[**Ambassador**](../../patterns/ambassador.md) can be used to offload common client connectivity tasks such as monitoring, logging, routing, and security (such as TLS) in a language agnostic way.</span></span> <span data-ttu-id="90a32-109">アンバサダー サービスは多くの場合、サイドカー (下記参照) としてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="90a32-109">Ambassador services are often deployed as a sidecar (see below).</span></span>

<span data-ttu-id="90a32-110">[**破損対策レイヤー** ](../../patterns/anti-corruption-layer.md)は、新しいアプリケーションの設計がレガシ システムへの依存によって確実に制限されないようにするため、新しいアプリケーションとレガシ アプリケーションの間にファサードを実装します。</span><span class="sxs-lookup"><span data-stu-id="90a32-110">[**Anti-corruption layer**](../../patterns/anti-corruption-layer.md) implements a façade between new and legacy applications, to ensure that the design of a new application is not limited by dependencies on legacy systems.</span></span>

<span data-ttu-id="90a32-111">[**フロントエンド用バックエンド**](../../patterns/backends-for-frontends.md)は、デスクトップやモバイルなど、クライアントのさまざまな種類に応じて独立したバックエンド サービスを作成します。</span><span class="sxs-lookup"><span data-stu-id="90a32-111">[**Backends for Frontends**](../../patterns/backends-for-frontends.md) creates separate backend services for different types of clients, such as desktop and mobile.</span></span> <span data-ttu-id="90a32-112">そうすれば、さまざまな種類のクライアントの競合する要件を単一のバックエンド サービスで処理する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="90a32-112">That way, a single backend service doesn’t need to handle the conflicting requirements of various client types.</span></span> <span data-ttu-id="90a32-113">このパターンを使用すると、クライアント固有の問題を切り離すことによって、各マイクロサービスのシンプルさを維持できます。</span><span class="sxs-lookup"><span data-stu-id="90a32-113">This pattern can help keep each microservice simple, by separating client-specific concerns.</span></span>

<span data-ttu-id="90a32-114">[**バルクヘッド**](../../patterns/bulkhead.md)は、接続プール、メモリ、CPU などの重要なリソースをワークロードまたはサービスごとに独立させます。</span><span class="sxs-lookup"><span data-stu-id="90a32-114">[**Bulkhead**](../../patterns/bulkhead.md) isolates critical resources, such as connection pool, memory, and CPU, for each workload or service.</span></span> <span data-ttu-id="90a32-115">バルクヘッドを使用すると、単一のワークロード (またはサービス) がすべてのリソースを消費して他のワークロードのリソースが枯渇することがなくなります。</span><span class="sxs-lookup"><span data-stu-id="90a32-115">By using bulkheads, a single workload (or service) can’t consume all of the resources, starving others.</span></span> <span data-ttu-id="90a32-116">このパターンでは、1 つのサービスによって発生した障害の連鎖を防ぐことによって、システムの回復性が向上します。</span><span class="sxs-lookup"><span data-stu-id="90a32-116">This pattern increases the resiliency of the system by preventing cascading failures caused by one service.</span></span>

<span data-ttu-id="90a32-117">[**ゲートウェイ集約**](../../patterns/gateway-aggregation.md)では、複数の個々のマイクロサービスへの要求を単一の要求に集約し、コンシューマーとサービスの間のトラフィックを削減します。</span><span class="sxs-lookup"><span data-stu-id="90a32-117">[**Gateway Aggregation**](../../patterns/gateway-aggregation.md) aggregates requests to multiple individual microservices into a single request, reducing chattiness between consumers and services.</span></span>

<span data-ttu-id="90a32-118">[**ゲートウェイ オフロード**](../../patterns/gateway-offloading.md)では、SSL 証明書の使用などの共有サービス機能を各マイクロサービスから API ゲートウェイにオフロードすることができます。</span><span class="sxs-lookup"><span data-stu-id="90a32-118">[**Gateway Offloading**](../../patterns/gateway-offloading.md) enables each microservice to offload shared service functionality, such as the use of SSL certificates, to an API gateway.</span></span>

<span data-ttu-id="90a32-119">[**ゲートウェイ ルーティング**](../../patterns/gateway-routing.md)では、単一のエンドポイントを使用して要求を複数のマイクロサービスにルーティングします。これにより、コンシューマーは多数の個別エンドポイントを管理する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="90a32-119">[**Gateway Routing**](../../patterns/gateway-routing.md) routes requests to multiple microservices using a single endpoint, so that consumers don't need to manage many separate endpoints.</span></span>

<span data-ttu-id="90a32-120">[**サイドカー**](../../patterns/sidecar.md)では、アプリケーションのヘルパー コンポーネントを別のコンテナーまたはプロセスとしてデプロイし、分離性とカプセル化を実現します。</span><span class="sxs-lookup"><span data-stu-id="90a32-120">[**Sidecar**](../../patterns/sidecar.md) deploys helper components of an application as a separate container or process to provide isolation and encapsulation.</span></span>

<span data-ttu-id="90a32-121">[**ストラングラー**](../../patterns/strangler.md)は、機能の特定の部分を新しいサービスに徐々に置き換えることで、アプリケーションの段階的なリファクタリングをサポートします。</span><span class="sxs-lookup"><span data-stu-id="90a32-121">[**Strangler**](../../patterns/strangler.md) supports incremental refactoring of an application, by gradually replacing specific pieces of functionality with new services.</span></span>

<span data-ttu-id="90a32-122">Azure アーキテクチャ センターでクラウド設計パターンの完全なカタログを見るには、[クラウド設計パターン](../../patterns/index.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="90a32-122">For the complete catalog of cloud design patterns on the Azure Architecture Center, see [Cloud Design Patterns](../../patterns/index.md).</span></span>
