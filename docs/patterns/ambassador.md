---
title: アンバサダー パターン
titleSuffix: Cloud Design Patterns
description: コンシューマー サービスまたはアプリケーションの代わりにネットワーク要求を送信するヘルパー サービスを作成します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: f03bfa0b45494ac1428aeee5cc6c413d5607ba79
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009782"
---
# <a name="ambassador-pattern"></a><span data-ttu-id="e458d-104">アンバサダー パターン</span><span class="sxs-lookup"><span data-stu-id="e458d-104">Ambassador pattern</span></span>

<span data-ttu-id="e458d-105">コンシューマー サービスまたはアプリケーションの代わりにネットワーク要求を送信するヘルパー サービスを作成します。</span><span class="sxs-lookup"><span data-stu-id="e458d-105">Create helper services that send network requests on behalf of a consumer service or application.</span></span> <span data-ttu-id="e458d-106">アンバサダー サービスは、クライアントに併置されているプロセス外のプロキシと考えることができます。</span><span class="sxs-lookup"><span data-stu-id="e458d-106">An ambassador service can be thought of as an out-of-process proxy that is co-located with the client.</span></span>

<span data-ttu-id="e458d-107">このパターンは、監視、ログ記録、ルーティング、セキュリティ (TLS など)、[回復性パターン][ resiliency-patterns]といった一般的なクライアント接続のタスクを言語に関係ない方法でオフロードするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="e458d-107">This pattern can be useful for offloading common client connectivity tasks such as monitoring, logging, routing, security (such as TLS), and [resiliency patterns][resiliency-patterns] in a language agnostic way.</span></span> <span data-ttu-id="e458d-108">これは、レガシ アプリケーションまたは変更が困難なアプリケーションで、ネットワーク機能を拡張するためによく使用されます。</span><span class="sxs-lookup"><span data-stu-id="e458d-108">It is often used with legacy applications, or other applications that are difficult to modify, in order to extend their networking capabilities.</span></span> <span data-ttu-id="e458d-109">専門のチームでこれらの機能を実装することもできます。</span><span class="sxs-lookup"><span data-stu-id="e458d-109">It can also enable a specialized team to implement those features.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="e458d-110">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="e458d-110">Context and problem</span></span>

<span data-ttu-id="e458d-111">回復力のあるクラウド ベースのアプリケーションには、[サーキット ブレーク](./circuit-breaker.md)、ルーティング、計測と監視、ネットワークに関連する構成を更新する機能などの機能が必要です。</span><span class="sxs-lookup"><span data-stu-id="e458d-111">Resilient cloud-based applications require features such as [circuit breaking](./circuit-breaker.md), routing, metering and monitoring, and the ability to make network-related configuration updates.</span></span> <span data-ttu-id="e458d-112">こうした機能を追加するためにレガシ アプリケーションまたは既存のコード ライブラリを更新するのは、コードがもはや管理されていないか、開発チームで変更するのが容易ではないために、困難あるいは不可能な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e458d-112">It may be difficult or impossible to update legacy applications or existing code libraries to add these features, because the code is no longer maintained or can't be easily modified by the development team.</span></span>

<span data-ttu-id="e458d-113">ネットワークの呼び出しも、接続、認証、および承認について大幅な構成が必要かもしれません。</span><span class="sxs-lookup"><span data-stu-id="e458d-113">Network calls may also require substantial configuration for connection, authentication, and authorization.</span></span> <span data-ttu-id="e458d-114">これらの呼び出しが、複数の言語とフレームワークで構築された複数のアプリケーションで使用されている場合、こうしたインスタンスの各々について呼び出しを構成しなければいけません。</span><span class="sxs-lookup"><span data-stu-id="e458d-114">If these calls are used across multiple applications, built using multiple languages and frameworks, the calls must be configured for each of these instances.</span></span> <span data-ttu-id="e458d-115">さらに、ネットワークとセキュリティの機能は、組織内の中核チームによって管理される必要があるかもしれません。</span><span class="sxs-lookup"><span data-stu-id="e458d-115">In addition, network and security functionality may need to be managed by a central team within your organization.</span></span> <span data-ttu-id="e458d-116">大規模なコード ベースでは、そうしたチームがなじみのないアプリケーション コードを更新するのは危険な場合があります。</span><span class="sxs-lookup"><span data-stu-id="e458d-116">With a large code base, it can be risky for that team to update application code they aren't familiar with.</span></span>

## <a name="solution"></a><span data-ttu-id="e458d-117">解決策</span><span class="sxs-lookup"><span data-stu-id="e458d-117">Solution</span></span>

<span data-ttu-id="e458d-118">クライアント フレームワークおよびライブラリを、アプリケーションと外部サービス間のプロキシとして機能する外部プロセスに配置します。</span><span class="sxs-lookup"><span data-stu-id="e458d-118">Put client frameworks and libraries into an external process that acts as a proxy between your application and external services.</span></span> <span data-ttu-id="e458d-119">アプリケーションと同じホスト環境にプロキシを展開して、ルーティング、回復性、セキュリティ機能の制御を許可し、ホストに関連するアクセス制限を回避します。</span><span class="sxs-lookup"><span data-stu-id="e458d-119">Deploy the proxy on the same host environment as your application to allow control over routing, resiliency, security features, and to avoid any host-related access restrictions.</span></span> <span data-ttu-id="e458d-120">アンバサダー パターンを使用して、インストルメンテーションを標準化および拡張することもできます。</span><span class="sxs-lookup"><span data-stu-id="e458d-120">You can also use the ambassador pattern to standardize and extend instrumentation.</span></span> <span data-ttu-id="e458d-121">プロキシは、待機時間やリソース使用率などのパフォーマンス メトリックを監視することができ、この監視はアプリケーションと同じホスト環境で行われます。</span><span class="sxs-lookup"><span data-stu-id="e458d-121">The proxy can monitor performance metrics such as latency or resource usage, and this monitoring happens in the same host environment as the application.</span></span>

![アンバサダー パターンの図](./_images/ambassador.png)

<span data-ttu-id="e458d-123">アンバサダーにオフロードされる機能は、アプリケーションとは別に管理できます。</span><span class="sxs-lookup"><span data-stu-id="e458d-123">Features that are offloaded to the ambassador can be managed independently of the application.</span></span> <span data-ttu-id="e458d-124">アンバサダーは、アプリケーションの従来の機能を妨げることなく、更新および変更することができます。</span><span class="sxs-lookup"><span data-stu-id="e458d-124">You can update and modify the ambassador without disturbing the application's legacy functionality.</span></span> <span data-ttu-id="e458d-125">さらに別の専門チームが、アンバサダーに移動されたセキュリティ、ネットワーク、認証の機能を実装および管理することも可能です。</span><span class="sxs-lookup"><span data-stu-id="e458d-125">It also allows for separate, specialized teams to implement and maintain security, networking, or authentication features that have been moved to the ambassador.</span></span>

<span data-ttu-id="e458d-126">アンバサダー サービスは、使用するアプリケーションまたはサービスのライフ サイクルを伴う[サイドカー](./sidecar.md)としてデプロイすることができます。</span><span class="sxs-lookup"><span data-stu-id="e458d-126">Ambassador services can be deployed as a [sidecar](./sidecar.md) to accompany the lifecycle of a consuming application or service.</span></span> <span data-ttu-id="e458d-127">あるいは、アンバサダーが共通のホストで複数の個別のプロセスによって共有されている場合は、デーモンまたは Windows サービスとしてデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="e458d-127">Alternatively, if an ambassador is shared by multiple separate processes on a common host, it can be deployed as a daemon or Windows service.</span></span> <span data-ttu-id="e458d-128">使用するサービスがコンテナー化されている場合は、アンバサダーは通信用に構成した適切なリンクとともに、同じホスト上の別のコンテナーとして作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e458d-128">If the consuming service is containerized, the ambassador should be created as a separate container on the same host, with the appropriate links configured for communication.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="e458d-129">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="e458d-129">Issues and considerations</span></span>

- <span data-ttu-id="e458d-130">このプロキシは、待機時間のオーバーヘッドをもたらします。</span><span class="sxs-lookup"><span data-stu-id="e458d-130">The proxy adds some latency overhead.</span></span> <span data-ttu-id="e458d-131">アプリケーションによって直接呼び出されるクライアント ライブラリのほうがより適切な方法かもしれませんので、検討してください。</span><span class="sxs-lookup"><span data-stu-id="e458d-131">Consider whether a client library, invoked directly by the application, is a better approach.</span></span>
- <span data-ttu-id="e458d-132">プロキシに汎用的な機能を含めることで考えられる影響を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="e458d-132">Consider the possible impact of including generalized features in the proxy.</span></span> <span data-ttu-id="e458d-133">たとえば、アンバサダーは再試行を処理する可能性がありますが、すべての操作がべき等でない限り、それは安全ではないかもしれません。</span><span class="sxs-lookup"><span data-stu-id="e458d-133">For example, the ambassador could handle retries, but that might not be safe unless all operations are idempotent.</span></span>
- <span data-ttu-id="e458d-134">クライアントがコンテキストをプロキシに渡すだけでなく、そのクライアントに返すこともできるようなメカニズムを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e458d-134">Consider a mechanism to allow the client to pass some context to the proxy, as well as back to the client.</span></span> <span data-ttu-id="e458d-135">たとえば、再試行を除外する、または再試行の最大回数を指定する HTTP 要求ヘッダーを含めます。</span><span class="sxs-lookup"><span data-stu-id="e458d-135">For example, include HTTP request headers to opt out of retry or specify the maximum number of times to retry.</span></span>
- <span data-ttu-id="e458d-136">プロキシをどのようにパッケージ化しデプロイするか検討してください。</span><span class="sxs-lookup"><span data-stu-id="e458d-136">Consider how you will package and deploy the proxy.</span></span>
- <span data-ttu-id="e458d-137">すべてのクライアントに 1 つの共有インスタンスを使用するか、各クライアントに 1 つのインスタンスを使用するか、検討してください。</span><span class="sxs-lookup"><span data-stu-id="e458d-137">Consider whether to use a single shared instance for all clients or an instance for each client.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="e458d-138">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="e458d-138">When to use this pattern</span></span>

<span data-ttu-id="e458d-139">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="e458d-139">Use this pattern when you:</span></span>

- <span data-ttu-id="e458d-140">複数の言語やフレームワークに、クライアント接続機能の共通セットを構築する必要がある。</span><span class="sxs-lookup"><span data-stu-id="e458d-140">Need to build a common set of client connectivity features for multiple languages or frameworks.</span></span>
- <span data-ttu-id="e458d-141">インフラストラクチャの開発者または他のさらに特殊化されたチームに、横断的なクライアント接続の懸案事項をまかせる必要がある。</span><span class="sxs-lookup"><span data-stu-id="e458d-141">Need to offload cross-cutting client connectivity concerns to infrastructure developers or other more specialized teams.</span></span>
- <span data-ttu-id="e458d-142">レガシ アプリケーションまたは変更が困難なアプリケーションで、クラウドまたはクラスターの接続要件をサポートする必要がある。</span><span class="sxs-lookup"><span data-stu-id="e458d-142">Need to support cloud or cluster connectivity requirements in a legacy application or an application that is difficult to modify.</span></span>

<span data-ttu-id="e458d-143">このパターンは次の状況では適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e458d-143">This pattern may not be suitable:</span></span>

- <span data-ttu-id="e458d-144">ネットワーク要求の待機時間が重要である場合。</span><span class="sxs-lookup"><span data-stu-id="e458d-144">When network request latency is critical.</span></span> <span data-ttu-id="e458d-145">プロキシによって、わずかだとしてもオーバーヘッドが発生し、場合によってはそれがアプリケーションに影響を与える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e458d-145">A proxy will introduce some overhead, although minimal, and in some cases this may affect the application.</span></span>
- <span data-ttu-id="e458d-146">クライアント接続の機能が、単一言語によって使用されている場合。</span><span class="sxs-lookup"><span data-stu-id="e458d-146">When client connectivity features are consumed by a single language.</span></span> <span data-ttu-id="e458d-147">その場合、パッケージとして開発チームに配布するクライアント ライブラリが、より適切な選択かもしれません。</span><span class="sxs-lookup"><span data-stu-id="e458d-147">In that case, a better option might be a client library that is distributed to the development teams as a package.</span></span>
- <span data-ttu-id="e458d-148">接続機能を一般化することができず、クライアント アプリケーションとの緊密な統合が必要な場合。</span><span class="sxs-lookup"><span data-stu-id="e458d-148">When connectivity features cannot be generalized and require deeper integration with the client application.</span></span>

## <a name="example"></a><span data-ttu-id="e458d-149">例</span><span class="sxs-lookup"><span data-stu-id="e458d-149">Example</span></span>

<span data-ttu-id="e458d-150">次の図は、アンバサダー プロキシを経由してリモート サービスに要求を送るアプリケーションを示しています。</span><span class="sxs-lookup"><span data-stu-id="e458d-150">The following diagram shows an application making a request to a remote service via an ambassador proxy.</span></span> <span data-ttu-id="e458d-151">アンバサダーは、ルーティング、サーキット ブレーキ、およびログ記録を提供します。</span><span class="sxs-lookup"><span data-stu-id="e458d-151">The ambassador provides routing, circuit breaking, and logging.</span></span> <span data-ttu-id="e458d-152">リモート サービスを呼び出してから、クライアント アプリケーションへ応答を返します。</span><span class="sxs-lookup"><span data-stu-id="e458d-152">It calls the remote service and then returns the response to the client application:</span></span>

![アンバサダー パターンの例](./_images/ambassador-example.png)

## <a name="related-guidance"></a><span data-ttu-id="e458d-154">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="e458d-154">Related guidance</span></span>

- [<span data-ttu-id="e458d-155">サイドカー パターン</span><span class="sxs-lookup"><span data-stu-id="e458d-155">Sidecar pattern</span></span>](./sidecar.md)

<!-- links -->

[resiliency-patterns]: ./category/resiliency.md
