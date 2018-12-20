---
title: サーバーレス Web アプリケーション
titleSuffix: Azure Reference Architectures
description: サーバーレス Web アプリケーションと Web API の推奨アーキテクチャ。
author: MikeWasson
ms.date: 10/16/2018
ms.custom: seodec18
ms.openlocfilehash: ee735ac4f23cc2a819e2322bd9c4fb3b5adf5f3b
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120312"
---
# <a name="serverless-web-application-on-azure"></a><span data-ttu-id="039a8-103">Azure 上のサーバーレス Web アプリケーション</span><span class="sxs-lookup"><span data-stu-id="039a8-103">Serverless web application on Azure</span></span>

<span data-ttu-id="039a8-104">この参照アーキテクチャは、[サーバーレス](https://azure.microsoft.com/solutions/serverless/) Web アプリケーションを示しています。</span><span class="sxs-lookup"><span data-stu-id="039a8-104">This reference architecture shows a [serverless](https://azure.microsoft.com/solutions/serverless/) web application.</span></span> <span data-ttu-id="039a8-105">このアプリケーションでは Azure Blob Storage から静的コンテンツを提供し、Azure Functions を使用して API が実装されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-105">The application serves static content from Azure Blob Storage, and implements an API using Azure Functions.</span></span> <span data-ttu-id="039a8-106">API は Cosmos DB からデータを読み取り、結果を Web アプリに返します。</span><span class="sxs-lookup"><span data-stu-id="039a8-106">The API reads data from Cosmos DB and returns the results to the web app.</span></span> <span data-ttu-id="039a8-107">このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-107">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![サーバーレス Web アプリケーションの参照アーキテクチャ](./_images/serverless-web-app.png)

<span data-ttu-id="039a8-109">サーバーレスという言葉は 2 つの異なる意味を持ちますが、この 2 つの意味は関連しています。</span><span class="sxs-lookup"><span data-stu-id="039a8-109">The term serverless has two distinct but related meanings:</span></span>

- <span data-ttu-id="039a8-110">**サービスとしてのバックエンド** (BaaS)。</span><span class="sxs-lookup"><span data-stu-id="039a8-110">**Backend as a service** (BaaS).</span></span> <span data-ttu-id="039a8-111">データベースやストレージなどのバックエンド クラウド サービスには、クライアント アプリケーションがこれらのサービスに直接接続できるようにする API が用意されています。</span><span class="sxs-lookup"><span data-stu-id="039a8-111">Backend cloud services, such as databases and storage, provide APIs that enable client applications to connect directly to these services.</span></span>
- <span data-ttu-id="039a8-112">**サービスとしての関数** (FaaS)。</span><span class="sxs-lookup"><span data-stu-id="039a8-112">**Functions as a service** (FaaS).</span></span> <span data-ttu-id="039a8-113">このモデルでは、"関数" はクラウドにデプロイされるひとまとまりのコードで、コードを実行するサーバーが完全に抽象化されたホスティング環境内で実行されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-113">In this model, a "function" is a piece of code that is deployed to the cloud and runs inside a hosting environment that completely abstracts the servers that run the code.</span></span>

<span data-ttu-id="039a8-114">両方の定義に共通しているのは、開発者と DevOps スタッフが、サーバーをデプロイ、構成、または管理する必要がないというアイデアです。</span><span class="sxs-lookup"><span data-stu-id="039a8-114">Both definitions have in common the idea that developers and DevOps personnel don't need to deploy, configure, or manage servers.</span></span> <span data-ttu-id="039a8-115">Azure Blob Storage からの Web コンテンツ提供は BaaS の一例ですが、この参照アーキテクチャでは、Azure Functions を使用した FaaS に重点を置いています。</span><span class="sxs-lookup"><span data-stu-id="039a8-115">This reference architecture focuses on FaaS using Azure Functions, although serving web content from Azure Blob Storage is an example of BaaS.</span></span> <span data-ttu-id="039a8-116">FaaS の重要な特性をいくつか次に示します。</span><span class="sxs-lookup"><span data-stu-id="039a8-116">Some important characteristics of FaaS are:</span></span>

1. <span data-ttu-id="039a8-117">コンピューティング リソースが、プラットフォームによって必要に応じて動的に割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="039a8-117">Compute resources are allocated dynamically as needed by the platform.</span></span>
1. <span data-ttu-id="039a8-118">使用量ベースの価格:ご自身のコードの実行に使用されたコンピューティング リソースに対してのみ課金されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-118">Consumption-based pricing: You are charged only for the compute resources used to execute your code.</span></span>
1. <span data-ttu-id="039a8-119">コンピューティング リソースがトラフィックに基づいてオンデマンドでスケーリングされます。開発者が構成する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="039a8-119">The compute resources scale on demand based on traffic, without the developer needing to do any configuration.</span></span>

<span data-ttu-id="039a8-120">HTTP 要求やメッセージがキューに到着するなど、外部トリガーが発生すると、関数が実行されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-120">Functions are executed when an external trigger occurs, such as an HTTP request or a message arriving on a queue.</span></span> <span data-ttu-id="039a8-121">このため、サーバーレス アーキテクチャにとって、[イベント ドリブン アーキテクチャのスタイル][event-driven]が自然になります。</span><span class="sxs-lookup"><span data-stu-id="039a8-121">This makes an [event-driven architecture style][event-driven] natural for serverless architectures.</span></span> <span data-ttu-id="039a8-122">アーキテクチャでのコンポーネント間の動作を調整するには、メッセージ ブローカーまたはパブリッシュ/サブスクライブ パターンの使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-122">To coordinate work between components in the architecture, consider using message brokers or pub/sub patterns.</span></span> <span data-ttu-id="039a8-123">Azure でのメッセージング テクノロジの選択に関するヘルプについては、「[メッセージを配信する Azure サービスの選択][azure-messaging]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-123">For help choosing between messaging technologies in Azure, see [Choose between Azure services that deliver messages][azure-messaging].</span></span>

## <a name="architecture"></a><span data-ttu-id="039a8-124">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="039a8-124">Architecture</span></span>

<span data-ttu-id="039a8-125">アーキテクチャは、次のコンポーネントで構成されています。</span><span class="sxs-lookup"><span data-stu-id="039a8-125">The architecture consists of the following components:</span></span>

<span data-ttu-id="039a8-126">**Blob Storage**。</span><span class="sxs-lookup"><span data-stu-id="039a8-126">**Blob Storage**.</span></span> <span data-ttu-id="039a8-127">HTML、CSS、JavaScript ファイルなどの静的 Web コンテンツは Azure Blob Storage に格納され、[静的 Web サイトのホスティング][static-hosting]を使用してクライアントに提供されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-127">Static web content, such as HTML, CSS, and JavaScript files, are stored in Azure Blob Storage and served to clients by using [static website hosting][static-hosting].</span></span> <span data-ttu-id="039a8-128">動的な対話はすべて、バックエンド API を呼び出す JavaScript コードを介して行われます。</span><span class="sxs-lookup"><span data-stu-id="039a8-128">All dynamic interaction happens through JavaScript code making calls to the backend APIs.</span></span> <span data-ttu-id="039a8-129">Web ページをレンダリングするサーバー側のコードはありません。</span><span class="sxs-lookup"><span data-stu-id="039a8-129">There is no server-side code to render the web page.</span></span> <span data-ttu-id="039a8-130">静的な Web サイトのホスティングでは、インデックス ドキュメントとカスタム 404 エラー ページがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-130">Static website hosting supports index documents and custom 404 error pages.</span></span>

> [!NOTE]
> <span data-ttu-id="039a8-131">静的 Web サイトのホスティングは現在[プレビュー][static-hosting-preview]の段階です。</span><span class="sxs-lookup"><span data-stu-id="039a8-131">Static website hosting is currently in [preview][static-hosting-preview].</span></span>

<span data-ttu-id="039a8-132">**CDN**。</span><span class="sxs-lookup"><span data-stu-id="039a8-132">**CDN**.</span></span> <span data-ttu-id="039a8-133">[Azure Content Delivery Network][cdn] (CDN) を使用して、待機時間を短縮してコンテンツを高速に配信できるようにコンテンツをキャッシュし、HTTPS エンドポイントを提供します。</span><span class="sxs-lookup"><span data-stu-id="039a8-133">Use [Azure Content Delivery Network][cdn] (CDN) to cache content for lower latency and faster delivery of content, as well as providing an HTTPS endpoint.</span></span>

<span data-ttu-id="039a8-134">**Function App**。</span><span class="sxs-lookup"><span data-stu-id="039a8-134">**Function Apps**.</span></span> <span data-ttu-id="039a8-135">[Azure Functions][functions] はサーバーレス コンピューティングの 1 つのオプションです。</span><span class="sxs-lookup"><span data-stu-id="039a8-135">[Azure Functions][functions] is a serverless compute option.</span></span> <span data-ttu-id="039a8-136">ひとまとまりのコード ("関数") がトリガーによって呼び出されるイベント ドリブン モデルが使用されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-136">It uses an event-driven model, where a piece of code (a "function") is invoked by a trigger.</span></span> <span data-ttu-id="039a8-137">このアーキテクチャでは、関数は、クライアントが HTTP 要求を行うと呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-137">In this architecture, the function is invoked when a client makes an HTTP request.</span></span> <span data-ttu-id="039a8-138">要求は、以下で説明するように、常に API ゲートウェイ経由でルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-138">The request is always routed through an API gateway, described below.</span></span>

<span data-ttu-id="039a8-139">**API Management**。</span><span class="sxs-lookup"><span data-stu-id="039a8-139">**API Management**.</span></span> <span data-ttu-id="039a8-140">[API Management][apim] は、HTTP 関数の前に配置される API ゲートウェイを提供します。</span><span class="sxs-lookup"><span data-stu-id="039a8-140">[API Management][apim] provides a API gateway that sits in front of the HTTP function.</span></span> <span data-ttu-id="039a8-141">API Management を使用すると、クライアント アプリケーションで使用される API を発行および管理できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-141">You can use API Management to publish and manage APIs used by client applications.</span></span> <span data-ttu-id="039a8-142">ゲートウェイは、バックエンド API からフロントエンド アプリケーションを分離するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="039a8-142">Using a gateway helps to decouple the front-end application from the back-end APIs.</span></span> <span data-ttu-id="039a8-143">たとえば、API Management では、URL の再書き込み、バックエンドに到達する前の要求の変換、要求または応答ヘッダーの設定などを行うことができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-143">For example, API Management can rewrite URLs, transform requests before they reach the backend, set request or response headers, and so forth.</span></span>

<span data-ttu-id="039a8-144">API Management を使って、複数のサービスにまたがる、次のような機能を実装することもできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-144">API Management can also be used to implement cross-cutting concerns such as:</span></span>

- <span data-ttu-id="039a8-145">使用量クォータとレート制限の強制</span><span class="sxs-lookup"><span data-stu-id="039a8-145">Enforcing usage quotas and rate limits</span></span>
- <span data-ttu-id="039a8-146">認証用の OAuth トークンの検証</span><span class="sxs-lookup"><span data-stu-id="039a8-146">Validating OAuth tokens for authentication</span></span>
- <span data-ttu-id="039a8-147">クロス オリジン要求 (CORS) の有効化</span><span class="sxs-lookup"><span data-stu-id="039a8-147">Enabling cross-origin requests (CORS)</span></span>
- <span data-ttu-id="039a8-148">応答のキャッシュ</span><span class="sxs-lookup"><span data-stu-id="039a8-148">Caching responses</span></span>
- <span data-ttu-id="039a8-149">要求の監視とログ記録</span><span class="sxs-lookup"><span data-stu-id="039a8-149">Monitoring and logging requests</span></span>

<span data-ttu-id="039a8-150">API Management によって提供される一部の機能が不要の場合は、別の選択肢として[関数プロキシ][functions-proxy]を使用できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-150">If you don't need all of the functionality provided by API Management, another option is to use [Functions Proxies][functions-proxy].</span></span> <span data-ttu-id="039a8-151">Azure Functions のこの機能を使用すると、バックエンド関数へのルートを作成することで、複数の関数アプリに対して 1 つの API サーフェスを定義できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-151">This feature of Azure Functions lets you define a single API surface for multiple function apps, by creating routes to back-end functions.</span></span> <span data-ttu-id="039a8-152">関数プロキシによって、HTTP 要求および応答で制限付き変換を実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-152">Function proxies can also perform limited transformations on the HTTP request and response.</span></span> <span data-ttu-id="039a8-153">ただし、API Management ほど豊富なポリシー ベース機能は用意されていません。</span><span class="sxs-lookup"><span data-stu-id="039a8-153">However, they don't provide the same rich policy-based capabilities of API Management.</span></span>

<span data-ttu-id="039a8-154">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="039a8-154">**Cosmos DB**.</span></span> <span data-ttu-id="039a8-155">[Cosmos DB][cosmosdb] は、マルチモデル データベース サービスです。</span><span class="sxs-lookup"><span data-stu-id="039a8-155">[Cosmos DB][cosmosdb] is a multi-model database  service.</span></span> <span data-ttu-id="039a8-156">このシナリオでは、関数アプリケーションは、クライアントからの HTTP GET 要求に応答して Cosmos DB からドキュメントをフェッチします。</span><span class="sxs-lookup"><span data-stu-id="039a8-156">For this scenario, the function application fetches documents from Cosmos DB in response to HTTP GET requests from the client.</span></span>

<span data-ttu-id="039a8-157">**Azure Active Directory** (Azure AD)。</span><span class="sxs-lookup"><span data-stu-id="039a8-157">**Azure Active Directory** (Azure AD).</span></span> <span data-ttu-id="039a8-158">ユーザーは、自身の Azure AD 資格情報を使用して Web アプリケーションにサインインします。</span><span class="sxs-lookup"><span data-stu-id="039a8-158">Users sign into the web application by using their Azure AD credentials.</span></span> <span data-ttu-id="039a8-159">Azure AD から API のアクセス トークンが返されます。Web アプリケーションはこれを使って API 要求を認証します (「[認証](#authentication)」を参照)。</span><span class="sxs-lookup"><span data-stu-id="039a8-159">Azure AD returns an access token for the API, which the web application uses to authenticate API requests (see [Authentication](#authentication)).</span></span>

<span data-ttu-id="039a8-160">**Azure Monitor**。</span><span class="sxs-lookup"><span data-stu-id="039a8-160">**Azure Monitor**.</span></span> <span data-ttu-id="039a8-161">[Monitor][monitor] は、ソリューションにデプロイされた Azure サービスに関するパフォーマンス メトリックを収集します。</span><span class="sxs-lookup"><span data-stu-id="039a8-161">[Monitor][monitor] collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="039a8-162">ダッシュボードでこれらを視覚化することで、ソリューションの正常性を把握できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-162">By visualizing these in a dashboard, you can get visibility into the health of the solution.</span></span> <span data-ttu-id="039a8-163">アプリケーション ログも収集されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-163">It also collected application logs.</span></span>

<span data-ttu-id="039a8-164">**Azure Pipelines**。</span><span class="sxs-lookup"><span data-stu-id="039a8-164">**Azure Pipelines**.</span></span> <span data-ttu-id="039a8-165">[Pipelines][pipelines] は、アプリケーションをビルド、テスト、デプロイする継続的インテグレーション (CI) および継続的デリバリー (CD) サービスです。</span><span class="sxs-lookup"><span data-stu-id="039a8-165">[Pipelines][pipelines] is a continuous integration (CI) and continuous delivery (CD) service that builds, tests, and deploys the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="039a8-166">Recommendations</span><span class="sxs-lookup"><span data-stu-id="039a8-166">Recommendations</span></span>

### <a name="function-app-plans"></a><span data-ttu-id="039a8-167">Function App プラン</span><span class="sxs-lookup"><span data-stu-id="039a8-167">Function App plans</span></span>

<span data-ttu-id="039a8-168">Azure Functions では 2 つのホスティング モデルがサポートされています。</span><span class="sxs-lookup"><span data-stu-id="039a8-168">Azure Functions supports two hosting models.</span></span> <span data-ttu-id="039a8-169">**従量課金プラン**では、コードの実行時にコンピューティング能力が自動的に割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="039a8-169">With the **consumption plan**, compute power is automatically allocated when your code is running.</span></span>  <span data-ttu-id="039a8-170">**App Service** プランでは、一連の VM がお使いのコードに対して割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="039a8-170">With the **App Service** plan, a set of VMs are allocated for your code.</span></span> <span data-ttu-id="039a8-171">App Service プランで定義されるのは VM の数とサイズです。</span><span class="sxs-lookup"><span data-stu-id="039a8-171">The App Service plan defines the number of VMs and the VM size.</span></span>

<span data-ttu-id="039a8-172">上記の定義に従うと、App Service プランは厳密には "*サーバーレス*" ではありません。</span><span class="sxs-lookup"><span data-stu-id="039a8-172">Note that the App Service plan is not strictly *serverless*, according to the definition given above.</span></span> <span data-ttu-id="039a8-173">プログラミング モデルは同じですが、従量課金プランと App Service プランの両方で、同じ関数コードを実行できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-173">The programming model is the same, however &mdash; the same function code can run in both a consumption plan and an App Service plan.</span></span>

<span data-ttu-id="039a8-174">使用するプランの種類を選択するときに検討すべき要素を、次に示します。</span><span class="sxs-lookup"><span data-stu-id="039a8-174">Here are some factors to consider when choosing which type of plan to use:</span></span>

- <span data-ttu-id="039a8-175">**コールド スタート**。</span><span class="sxs-lookup"><span data-stu-id="039a8-175">**Cold start**.</span></span> <span data-ttu-id="039a8-176">従量課金プランでは、最近呼び出されていない関数の場合、次回の実行時に待ち時間が少し長くなります。</span><span class="sxs-lookup"><span data-stu-id="039a8-176">With the consumption plan, a function that hasn't been invoked recently will incur some additional latency the next time it runs.</span></span> <span data-ttu-id="039a8-177">この追加の待ち時間は、ランタイム環境の割り当てと準備が原因で発生します。</span><span class="sxs-lookup"><span data-stu-id="039a8-177">This additional latency is due to allocating and preparing the runtime environment.</span></span> <span data-ttu-id="039a8-178">通常は数秒程度ですが、読み込む必要がある依存関係の数など、いくつかの要因によって異なってきます。</span><span class="sxs-lookup"><span data-stu-id="039a8-178">It is usually on the order of seconds but depends on several factors, including the number of dependencies that need to be loaded.</span></span> <span data-ttu-id="039a8-179">詳細については、「[Understanding Serverless Cold Start (サーバーレスのコールド スタートについて)][functions-cold-start]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-179">For more information, see [Understanding Serverless Cold Start][functions-cold-start].</span></span> <span data-ttu-id="039a8-180">通常、コールド スタートが問題になるのは、非同期のメッセージ ドリブン ワークロード (キューまたはイベント ハブ トリガー) よりも対話型ワークロード (HTTP トリガー) です。対話型ワークロードでは、ユーザーが待ち時間が長くなったことに直接気付くためです。</span><span class="sxs-lookup"><span data-stu-id="039a8-180">Cold start is usually more of a concern for interactive workloads (HTTP triggers) than asynchronous message-driven workloads (queue or event hubs triggers), because the additional latency is directly observed by users.</span></span>
- <span data-ttu-id="039a8-181">**タイムアウト時間**。</span><span class="sxs-lookup"><span data-stu-id="039a8-181">**Timeout period**.</span></span>  <span data-ttu-id="039a8-182">従量課金プランでは、[構成可能な][functions-timeout]期間 (最大 10 分) が経過すると関数の実行はタイムアウトします</span><span class="sxs-lookup"><span data-stu-id="039a8-182">In the consumption plan, a function execution times out after a [configurable][functions-timeout] period of time (to a maximum of 10 minutes)</span></span>
- <span data-ttu-id="039a8-183">**仮想ネットワークの分離**。</span><span class="sxs-lookup"><span data-stu-id="039a8-183">**Virtual network isolation**.</span></span> <span data-ttu-id="039a8-184">App Service プランを使用すると、分離された専用ホスティング環境である [App Service 環境][ase]内で関数を実行できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-184">Using an App Service plan allows functions to run inside of an [App Service Environment][ase], which is a dedicated and isolated hosting environment.</span></span>
- <span data-ttu-id="039a8-185">**価格モデル**。</span><span class="sxs-lookup"><span data-stu-id="039a8-185">**Pricing model**.</span></span> <span data-ttu-id="039a8-186">従量課金プランでは、実行数とリソース消費量 (メモリ &times; 実行時間) によって課金されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-186">The consumption plan is billed by the number of executions and resource consumption (memory &times; execution time).</span></span> <span data-ttu-id="039a8-187">App Service プランでは、VM インスタンス SKU に基づいて 1 時間ごとに課金されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-187">The App Service plan is billed hourly based on VM instance SKU.</span></span> <span data-ttu-id="039a8-188">従量課金プランは、使用したコンピューティング リソースにしか支払いが発生しないため、多くの場合 App Service プランよりもコストがかかりません。</span><span class="sxs-lookup"><span data-stu-id="039a8-188">Often, the consumption plan can be cheaper than an App Service plan, because you pay only for the compute resources that you use.</span></span> <span data-ttu-id="039a8-189">これは、トラフィックに山と谷がある場合に特に当てはまります。</span><span class="sxs-lookup"><span data-stu-id="039a8-189">This is especially true if your traffic experiences peaks and troughs.</span></span> <span data-ttu-id="039a8-190">ただし、アプリケーションのスループットが一定して高い場合は、App Service プランの方が従量課金プランよりも低コストになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-190">However, if an application experiences constant high-volume throughput, an App Service plan may cost less than the consumption plan.</span></span>
- <span data-ttu-id="039a8-191">**スケーリング**。</span><span class="sxs-lookup"><span data-stu-id="039a8-191">**Scaling**.</span></span> <span data-ttu-id="039a8-192">従量課金モデルの大きな利点は、着信トラフィックに基づいて、必要に応じて動的にスケーリングされることです。</span><span class="sxs-lookup"><span data-stu-id="039a8-192">A big advantage of the consumption model is that it scales dynamically as needed, based on the incoming traffic.</span></span> <span data-ttu-id="039a8-193">このスケーリングは迅速に行われますが、やはり準備期間はあります。</span><span class="sxs-lookup"><span data-stu-id="039a8-193">While this scaling occurs quickly, there is still a ramp-up period.</span></span> <span data-ttu-id="039a8-194">一部のワークロードでは、準備時間なしでトラフィックのバーストを処理できるように、VM の意図的オーバープロビジョニングが必要なことがあります。</span><span class="sxs-lookup"><span data-stu-id="039a8-194">For some workloads, you might want to deliberately overprovision the VMs, so that you can handle bursts of traffic with zero ramp-up time.</span></span> <span data-ttu-id="039a8-195">その場合は、App Service プランを検討してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-195">In that case, consider an App Service plan.</span></span>

### <a name="function-app-boundaries"></a><span data-ttu-id="039a8-196">Function App の境界</span><span class="sxs-lookup"><span data-stu-id="039a8-196">Function App boundaries</span></span>

<span data-ttu-id="039a8-197">"*関数アプリ*" によって、1 つ以上の "*関数*" の実行がホストされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-197">A *function app* hosts the execution of one or more *functions*.</span></span> <span data-ttu-id="039a8-198">関数アプリを使用すると、複数の関数を 1 つの論理ユニットとしてまとめることができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-198">You can use a function app to group several functions together as a logical unit.</span></span> <span data-ttu-id="039a8-199">関数アプリ内の関数は、同じアプリケーション設定、ホスティング プラン、およびデプロイ ライフサイクルを共有しています。</span><span class="sxs-lookup"><span data-stu-id="039a8-199">Within a function app, the functions share the same application settings, hosting plan, and deployment lifecycle.</span></span> <span data-ttu-id="039a8-200">関数アプリはそれぞれ独自のホスト名を持ちます。</span><span class="sxs-lookup"><span data-stu-id="039a8-200">Each function app has its own hostname.</span></span>

<span data-ttu-id="039a8-201">関数アプリを使用して、同じライフサイクルと設定を共有する関数をグループ化します。</span><span class="sxs-lookup"><span data-stu-id="039a8-201">Use function apps to group functions that share the same lifecycle and settings.</span></span> <span data-ttu-id="039a8-202">同じライフサイクルを共有していない関数は、別の関数アプリでホストする必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-202">Functions that don't share the same lifecycle should be hosted in different function apps.</span></span>

<span data-ttu-id="039a8-203">マイクロサービス アプローチを使用することを検討します。このアプローチでは、各関数アプリが 1 つのマイクロサービスを表しており、関連する複数の関数で構成されている可能性があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-203">Consider taking a microservices approach, where each function app represents one microservice, possibly consisting of several related functions.</span></span> <span data-ttu-id="039a8-204">マイクロサービス アーキテクチャのサービスには、疎結合と、機能の高い凝集度が必要です。</span><span class="sxs-lookup"><span data-stu-id="039a8-204">In a microservices architecture, services should have loose coupling and high functional cohesion.</span></span> <span data-ttu-id="039a8-205">"*疎*" 結合とは、他のサービスを同時に更新しなくても、あるサービスを変更できることを意味します。</span><span class="sxs-lookup"><span data-stu-id="039a8-205">*Loosely* coupled means you can change one service without requiring other services to be updated at the same time.</span></span> <span data-ttu-id="039a8-206">"*高凝集*" とは、明確に定義された 1 つの目的をサービスが持つことを意味します。</span><span class="sxs-lookup"><span data-stu-id="039a8-206">*Cohesive* means a service has a single, well-defined purpose.</span></span> <span data-ttu-id="039a8-207">これらのアイデアの詳細については、「[マイクロサービスの設計:ドメイン分析][microservices-domain-analysis]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-207">For more discussion of these ideas, see [Designing microservices: Domain analysis][microservices-domain-analysis].</span></span>

### <a name="function-bindings"></a><span data-ttu-id="039a8-208">関数のバインディング</span><span class="sxs-lookup"><span data-stu-id="039a8-208">Function bindings</span></span>

<span data-ttu-id="039a8-209">可能な場合は、関数の[バインディング][functions-bindings]を使用します。</span><span class="sxs-lookup"><span data-stu-id="039a8-209">Use Functions [bindings][functions-bindings] when possible.</span></span> <span data-ttu-id="039a8-210">バインディングにより、お使いのコードをデータに接続し、他の Azure サービスと統合するために宣言型の方法が提供されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-210">Bindings provide a declarative way to connect your code to data and integrate with other Azure services.</span></span> <span data-ttu-id="039a8-211">入力バインディングでは、入力パラメーターが外部データ ソースから設定されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-211">An input binding populates an input parameter from an external data source.</span></span> <span data-ttu-id="039a8-212">出力バインディングでは、キューやデータベースなどのデータ シンクに関数の戻り値が送信されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-212">An output binding sends the function's return value to a data sink, such as a queue or database.</span></span>

<span data-ttu-id="039a8-213">たとえば、リファレンス実装の `GetStatus` 関数では、Cosmos DB の[入力バインディング][cosmosdb-input-binding]が使用されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-213">For example, the `GetStatus` function in the reference implementation uses the Cosmos DB [input binding][cosmosdb-input-binding].</span></span> <span data-ttu-id="039a8-214">このバインディングは、HTTP 要求のクエリ文字列から取得されるクエリ パラメーターを使用して、Cosmos DB のドキュメントを検索するように構成されています。</span><span class="sxs-lookup"><span data-stu-id="039a8-214">This binding is configured to look up a document in Cosmos DB, using query parameters that are taken from the query string in the HTTP request.</span></span> <span data-ttu-id="039a8-215">ドキュメントが見つかった場合、そのドキュメントは関数にパラメーターとして渡されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-215">If the document is found, it is passed to the function as a parameter.</span></span>

```csharp
[FunctionName("GetStatusFunction")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "%COSMOSDB_DATABASE_NAME%",
        collectionName: "%COSMOSDB_DATABASE_COL%",
        ConnectionStringSetting = "COSMOSDB_CONNECTION_STRING",
        Id = "{Query.deviceId}",
        PartitionKey = "{Query.deviceId}")] dynamic deviceStatus,
    ILogger log)
{
    ...
}
```

<span data-ttu-id="039a8-216">バインディングを使用すると、サービスと直接通信するコードを記述する必要がありません。これにより関数コードが簡単になるだけでなく、データ ソースまたはシンクの詳細が抽象化されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-216">By using bindings, you don't need to write code that talks directly to the service, which makes the function code simpler and also abstracts the details of the data source or sink.</span></span> <span data-ttu-id="039a8-217">ただし、場合によっては、バインディングが提供するよりも複雑なロジックが必要になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-217">In some cases, however, you may need more complex logic than the binding provides.</span></span> <span data-ttu-id="039a8-218">その場合は、Azure のクライアント SDK を直接使用します。</span><span class="sxs-lookup"><span data-stu-id="039a8-218">In that case, use the Azure client SDKs directly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="039a8-219">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="039a8-219">Scalability considerations</span></span>

<span data-ttu-id="039a8-220">**Functions**。</span><span class="sxs-lookup"><span data-stu-id="039a8-220">**Functions**.</span></span> <span data-ttu-id="039a8-221">従量課金プランの場合、HTTP トリガーはトラフィックに基づいてスケーリングされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-221">For the consumption plan, the HTTP trigger scales based on the traffic.</span></span> <span data-ttu-id="039a8-222">コンカレント関数インスタンスの数には制限がありますが、それぞれのインスタンスが一度に複数の要求を処理できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-222">There is a limit to the number of concurrent function instances, but each instance can process more than one request at a time.</span></span> <span data-ttu-id="039a8-223">App Service プランでは、HTTP トリガーは VM インスタンスの数に応じてスケーリングされます。この数は固定値にすることも、一連の自動スケーリング ルールに基づいて自動スケーリングすることもできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-223">For an App Service plan, the HTTP trigger scales according to the number of VM instances, which can be a fixed value or can autoscale based on a set of autoscaling rules.</span></span> <span data-ttu-id="039a8-224">詳細については、「[Azure Functions のスケールとホスティング][functions-scale]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-224">For information, see [Azure Functions scale and hosting][functions-scale].</span></span>

<span data-ttu-id="039a8-225">**Cosmos DB**。</span><span class="sxs-lookup"><span data-stu-id="039a8-225">**Cosmos DB**.</span></span> <span data-ttu-id="039a8-226">Cosmos DB のスループット容量は、[要求ユニット][ru] (RU) で測定されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-226">Throughput capacity for Cosmos DB is measured in [Request Units][ru] (RU).</span></span> <span data-ttu-id="039a8-227">1 RU のスループットは、1 KB のドキュメントを取得するのに必要なスループットに対応します。</span><span class="sxs-lookup"><span data-stu-id="039a8-227">A 1-RU throughput corresponds to the throughput need to GET a 1KB document.</span></span> <span data-ttu-id="039a8-228">Cosmos DB コンテナーを 10,000 RU を超えてスケーリングするには、コンテナーの作成時に[パーティション キー][partition-key]を指定し、作成するすべてのドキュメントにパーティション キーを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-228">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key][partition-key] when you create the container and include the partition key in every document that you create.</span></span> <span data-ttu-id="039a8-229">パーティション キーの詳細については、「[Azure Cosmos DB でのパーティション分割とスケーリング][cosmosdb-scale]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-229">For more information about partition keys, see [Partition and scale in Azure Cosmos DB][cosmosdb-scale].</span></span>

<span data-ttu-id="039a8-230">**API Management**。</span><span class="sxs-lookup"><span data-stu-id="039a8-230">**API Management**.</span></span> <span data-ttu-id="039a8-231">API Management はスケールアウトが可能で、ルールベースの自動スケーリングがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-231">API Management can scale out and supports rule-based autoscaling.</span></span> <span data-ttu-id="039a8-232">このスケーリング プロセスには、少なくとも 20 分かかることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-232">Note that the scaling process takes at least 20 minutes.</span></span> <span data-ttu-id="039a8-233">トラフィックのバーストが発生する場合は、最大バースト トラフィックの予測に基づいてプロビジョニングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-233">If your traffic is bursty, you should provision for the maximum burst traffic that you expect.</span></span> <span data-ttu-id="039a8-234">ただし、自動スケーリングを使えば、時間単位または日単位でトラフィックの変動が処理されるため便利です。</span><span class="sxs-lookup"><span data-stu-id="039a8-234">However, autoscaling is useful for handling hourly or daily variations in traffic.</span></span> <span data-ttu-id="039a8-235">詳細については、「[Azure API Management インスタンスを自動的にスケーリングする][apim-scale]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-235">For more information, see [Automatically scale an Azure API Management instance][apim-scale].</span></span>

## <a name="disaster-recovery-considerations"></a><span data-ttu-id="039a8-236">ディザスター リカバリーの考慮事項</span><span class="sxs-lookup"><span data-stu-id="039a8-236">Disaster recovery considerations</span></span>

<span data-ttu-id="039a8-237">ここに示したデプロイは単一の Azure リージョンに存在します。</span><span class="sxs-lookup"><span data-stu-id="039a8-237">The deployment shown here resides in a single Azure region.</span></span> <span data-ttu-id="039a8-238">ディザスター リカバリーでより回復性に優れたアプローチを実現するには、さまざまなサービスで地理的分散機能を利用します。</span><span class="sxs-lookup"><span data-stu-id="039a8-238">For a more resilient approach to disaster-recovery, take advantage of the geo-distribution features in the various services:</span></span>

- <span data-ttu-id="039a8-239">API Management では複数リージョンのデプロイがサポートされており、API Management サービスの単一インスタンスを任意の数の Azure リージョンに分散できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-239">API Management supports multi-region deployment, which can be used to distribute a single API Management instance across any number of Azure regions.</span></span> <span data-ttu-id="039a8-240">詳細については、「[複数の Azure リージョンに Azure API Management サービス インスタンスをデプロイする方法][api-geo]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-240">For more information, see [How to deploy an Azure API Management service instance to multiple Azure regions][api-geo].</span></span>

- <span data-ttu-id="039a8-241">[Traffic Manager][tm] を使用して、HTTP 要求をプライマリ リージョンにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="039a8-241">Use [Traffic Manager][tm] to route HTTP requests to the primary region.</span></span> <span data-ttu-id="039a8-242">そのリージョンで実行されている Function App が使用できなくなった場合、Traffic Manager によってセカンダリ リージョンにフェールオーバーできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-242">If the Function App running in that region becomes unavailable, Traffic Manager can fail over to a secondary region.</span></span>

- <span data-ttu-id="039a8-243">Cosmos DB では[複数のマスター リージョン][cosmosdb-geo]がサポートされています。これにより、Cosmos DB アカウントに追加した任意のリージョンに書き込むことができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-243">Cosmos DB supports [multiple master regions][cosmosdb-geo], which enables writes to any region that you add to your Cosmos DB account.</span></span> <span data-ttu-id="039a8-244">マルチマスターを有効にしなくても、プライマリ書き込みリージョンにはフェールオーバーできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-244">If you don't enable multi-master, you can still fail over the primary write region.</span></span> <span data-ttu-id="039a8-245">フェールオーバーは、Cosmos DB クライアント SDK と Azure Functions のバインディングによって自動的に処理されるため、アプリケーション構成設定を更新する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="039a8-245">The Cosmos DB client SDKs and the Azure Function bindings automatically handle the failover, so you don't need to update any application configuration settings.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="039a8-246">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="039a8-246">Security considerations</span></span>

### <a name="authentication"></a><span data-ttu-id="039a8-247">Authentication</span><span class="sxs-lookup"><span data-stu-id="039a8-247">Authentication</span></span>

<span data-ttu-id="039a8-248">リファレンス実装の `GetStatus` API では、Azure AD を使用して要求が認証されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-248">The `GetStatus` API in the reference implementation uses Azure AD to authenticate requests.</span></span> <span data-ttu-id="039a8-249">Azure AD がサポートするのは Open ID Connect プロトコルで、この認証プロトコルは、OAuth 2 プロトコルを基盤として構築されています。</span><span class="sxs-lookup"><span data-stu-id="039a8-249">Azure AD supports the Open ID Connect protocol, which is an authentication protocol built on top of the OAuth 2 protocol.</span></span>

<span data-ttu-id="039a8-250">このアーキテクチャでは、クライアント アプリケーションは、ブラウザーで実行されるシングル ページ アプリケーション (SPA) です。</span><span class="sxs-lookup"><span data-stu-id="039a8-250">In this architecture, the client application is a single-page application (SPA) that runs in the browser.</span></span> <span data-ttu-id="039a8-251">この種類のクライアント アプリケーションでは、クライアントをシークレットにできません。また、認証コードも非表示になりません。このため暗黙的な許可フローが適しています </span><span class="sxs-lookup"><span data-stu-id="039a8-251">This type of client application cannot keep a client secret or an authorization code hidden, so the implicit grant flow is appropriate.</span></span> <span data-ttu-id="039a8-252">([使用するべき OAuth 2.0 フロー][oauth-flow]に関するページをご覧ください)。</span><span class="sxs-lookup"><span data-stu-id="039a8-252">(See [Which OAuth 2.0 flow should I use?][oauth-flow]).</span></span> <span data-ttu-id="039a8-253">全体的なフローを次に示します。</span><span class="sxs-lookup"><span data-stu-id="039a8-253">Here's the overall flow:</span></span>

1. <span data-ttu-id="039a8-254">ユーザーが Web アプリケーションで "サインイン" リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="039a8-254">The user clicks the "Sign in" link in the web application.</span></span>
1. <span data-ttu-id="039a8-255">ブラウザーが Azure AD のサインイン ページにリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-255">The browser is redirected the Azure AD sign in page.</span></span>
1. <span data-ttu-id="039a8-256">ユーザーがサインインします。</span><span class="sxs-lookup"><span data-stu-id="039a8-256">The user signs in.</span></span>
1. <span data-ttu-id="039a8-257">Azure AD がクライアント アプリケーションにもう一度リダイレクトされます。その URL フラグメントにはアクセス トークンが含まれています。</span><span class="sxs-lookup"><span data-stu-id="039a8-257">Azure AD redirects back to the client application, including an access token in the URL fragment.</span></span>
1. <span data-ttu-id="039a8-258">Web アプリケーションが API を呼び出すとき、認証ヘッダーにはアクセス トークンが含まれています。</span><span class="sxs-lookup"><span data-stu-id="039a8-258">When the web application calls the API, it includes the access token in the Authentication header.</span></span> <span data-ttu-id="039a8-259">アプリケーション ID は、アクセス トークンで audience ("aud") クレームとして送信されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-259">The application ID is sent as the audience ('aud') claim in the access token.</span></span>
1. <span data-ttu-id="039a8-260">バックエンド API によってアクセス トークンが検証されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-260">The backend API validates the access token.</span></span>

<span data-ttu-id="039a8-261">認証を構成するには:</span><span class="sxs-lookup"><span data-stu-id="039a8-261">To configure authentication:</span></span>

- <span data-ttu-id="039a8-262">アプリケーションをお使いの Azure AD テナントに登録します。</span><span class="sxs-lookup"><span data-stu-id="039a8-262">Register an application in your Azure AD tenant.</span></span> <span data-ttu-id="039a8-263">これによりアプリケーション ID が生成されます。クライアントはこれをログイン URL に含めます。</span><span class="sxs-lookup"><span data-stu-id="039a8-263">This generates an application ID, which the client includes with the login URL.</span></span>

- <span data-ttu-id="039a8-264">Function App 内の Azure AD 認証を有効にします。</span><span class="sxs-lookup"><span data-stu-id="039a8-264">Enable Azure AD authentication inside the Function App.</span></span> <span data-ttu-id="039a8-265">詳細については、「[Azure App Service での認証および承認][app-service-auth]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-265">For more information, see [Authentication and authorization in Azure App Service][app-service-auth].</span></span>

- <span data-ttu-id="039a8-266">API Management に [validate-jwt ポリシー][apim-validate-jwt]を追加し、アクセス トークンを検証することで要求を事前承認できるようにします。</span><span class="sxs-lookup"><span data-stu-id="039a8-266">Add the [validate-jwt policy][apim-validate-jwt] to API Management to pre-authorize the request by validating the access token.</span></span>

<span data-ttu-id="039a8-267">詳細については、[GitHub の Readme][readme] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-267">For more details, see the [GitHub readme][readme].</span></span>

<span data-ttu-id="039a8-268">クライアント アプリケーションとバックエンド API については、Azure AD で個別にアプリ登録を作成することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="039a8-268">It's recommended to create separate app registrations in Azure AD for the client application and the backend API.</span></span> <span data-ttu-id="039a8-269">API を呼び出すアクセス許可をクライアント アプリケーションに付与します。</span><span class="sxs-lookup"><span data-stu-id="039a8-269">Grant the client application permission to call the API.</span></span> <span data-ttu-id="039a8-270">このアプローチにより、複数の API とクライアントを定義して、それぞれのアクセス許可を柔軟に制御できるようになります。</span><span class="sxs-lookup"><span data-stu-id="039a8-270">This approach gives you the flexibility to define multiple APIs and clients and control the permissions for each.</span></span>

<span data-ttu-id="039a8-271">API では、[スコープ][scopes]を使用して、アプリケーションがユーザーに要求するアクセス許可を細かく制御できるようにします。</span><span class="sxs-lookup"><span data-stu-id="039a8-271">Within an API, use [scopes][scopes] to give applications fine-grained control over what permissions they request from a user.</span></span> <span data-ttu-id="039a8-272">たとえば、API に `Read` スコープと `Write` スコープを指定し、特定のクライアント アプリが、ユーザーに `Read` アクセス許可のみを承認するよう求めることができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-272">For example, an API might have `Read` and `Write` scopes, and a particular client app might ask the user to authorize `Read` permissions only.</span></span>

### <a name="authorization"></a><span data-ttu-id="039a8-273">Authorization</span><span class="sxs-lookup"><span data-stu-id="039a8-273">Authorization</span></span>

<span data-ttu-id="039a8-274">多くのアプリケーションでは、バックエンド API で、ユーザーが特定のアクションを実行するアクセス許可があるかどうかを確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-274">In many applications, the backend API must check whether a user has permission to perform a given action.</span></span> <span data-ttu-id="039a8-275">[クレームベースの承認][claims]を使用することをお勧めします。この方法では、ユーザーに関する情報は ID プロバイダー (この例では Azure AD) によって伝達され、承認の判断に使用されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-275">It's recommended to use [claims-based authorization][claims], where information about the user is conveyed by the identity provider (in this case, Azure AD) and used to make authorization decisions.</span></span>

<span data-ttu-id="039a8-276">Azure AD がクライアントに返す ID トークン内で提供されるクレームもあります。</span><span class="sxs-lookup"><span data-stu-id="039a8-276">Some claims are provided inside the ID token that Azure AD returns to the client.</span></span> <span data-ttu-id="039a8-277">これらのクレームを関数アプリ内から取得するには、要求の X-MS-CLIENT-PRINCIPAL ヘッダーを調べます。</span><span class="sxs-lookup"><span data-stu-id="039a8-277">You can get these claims from within the function app by examining the X-MS-CLIENT-PRINCIPAL header in the request.</span></span> <span data-ttu-id="039a8-278">他のクレームについては、[Microsoft Graph][graph] を使用して、Azure AD にクエリを実行します (サインイン時にユーザーの同意が必要です)。</span><span class="sxs-lookup"><span data-stu-id="039a8-278">For other claims, use [Microsoft Graph][graph] to query Azure AD (requires user consent during sign-in).</span></span>

<span data-ttu-id="039a8-279">たとえば、Azure AD でアプリケーションを登録するときに、アプリケーションの登録マニフェストで一連のアプリケーション ロールを定義できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-279">For example, when you register an application in Azure AD, you can define a set of application roles in the application's registration manifest.</span></span> <span data-ttu-id="039a8-280">ユーザーがアプリケーションにサインインするとき、Azure AD には、そのユーザーに付与されたロール (グループ メンバーシップを通じて付与されているロールを含む) ごとに "roles" クレームが含まれています。</span><span class="sxs-lookup"><span data-stu-id="039a8-280">When a user signs into the application, Azure AD includes a "roles" claim for each role that the user has been granted (including roles that are inherited through group membership).</span></span>

<span data-ttu-id="039a8-281">リファレンス実装では、認証されたユーザーが `GetStatus` アプリケーション ロールのメンバーであるかどうかが関数によって確認されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-281">In the reference implementation, the function checks whether the authenticated user is a member of the `GetStatus` application role.</span></span> <span data-ttu-id="039a8-282">メンバーでない場合、関数は HTTP 未承認 (401) 応答を返します。</span><span class="sxs-lookup"><span data-stu-id="039a8-282">If not, the function returns an HTTP Unauthorized (401) response.</span></span>

```csharp
[FunctionName("GetStatusFunction")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(
        databaseName: "%COSMOSDB_DATABASE_NAME%",
        collectionName: "%COSMOSDB_DATABASE_COL%",
        ConnectionStringSetting = "COSMOSDB_CONNECTION_STRING",
        Id = "{Query.deviceId}",
        PartitionKey = "{Query.deviceId}")] dynamic deviceStatus,
    ILogger log)
{
    log.LogInformation("Processing GetStatus request.");

    return req.HandleIfAuthorizedForRoles(new[] { GetDeviceStatusRoleName },
        async () =>
        {
            string deviceId = req.Query["deviceId"];
            if (deviceId == null)
            {
                return new BadRequestObjectResult("Missing DeviceId");
            }

            return await Task.FromResult<IActionResult>(deviceStatus != null
                    ? (ActionResult)new OkObjectResult(deviceStatus)
                    : new NotFoundResult());
        },
        log);
}
```

<span data-ttu-id="039a8-283">このコード例では、`HandleIfAuthorizedForRoles` はロール クレームをチェックし、クレームが見つからない場合は HTTP 401 を返す拡張メソッドです。</span><span class="sxs-lookup"><span data-stu-id="039a8-283">In this code example, `HandleIfAuthorizedForRoles` is an extension method that checks for the role claim and returns HTTP 401 if the claim isn't found.</span></span> <span data-ttu-id="039a8-284">ソース コードについては、[こちら][HttpRequestAuthorizationExtensions]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-284">You can find the source code [here][HttpRequestAuthorizationExtensions].</span></span> <span data-ttu-id="039a8-285">`HandleIfAuthorizedForRoles` で `ILogger` パラメーターが使用されていることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-285">Notice that `HandleIfAuthorizedForRoles` takes an `ILogger` parameter.</span></span> <span data-ttu-id="039a8-286">監査証跡を確保し、必要に応じて問題を診断できるように、未承認の要求をログに記録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-286">You should log unauthorized requests so that you have an audit trail and can diagnose issues if needed.</span></span> <span data-ttu-id="039a8-287">同時に、HTTP 401 応答の詳細情報が漏えいしないようにします。</span><span class="sxs-lookup"><span data-stu-id="039a8-287">At the same time, avoid leaking any detailed information inside the HTTP 401 response.</span></span>

### <a name="cors"></a><span data-ttu-id="039a8-288">CORS</span><span class="sxs-lookup"><span data-stu-id="039a8-288">CORS</span></span>

<span data-ttu-id="039a8-289">この参照アーキテクチャでは、Web アプリケーションと API はオリジンを共有しません。</span><span class="sxs-lookup"><span data-stu-id="039a8-289">In this reference architecture, the web application and the API do not share the same origin.</span></span> <span data-ttu-id="039a8-290">つまり、アプリケーションが API を呼び出す場合、それはクロス オリジン要求になります。</span><span class="sxs-lookup"><span data-stu-id="039a8-290">That means when the application calls the API, it is a cross-origin request.</span></span> <span data-ttu-id="039a8-291">ブラウザーのセキュリティ機能により、Web ページでは AJAX 要求を別のドメインに送信することはできません。</span><span class="sxs-lookup"><span data-stu-id="039a8-291">Browser security prevents a web page from making AJAX requests to another domain.</span></span> <span data-ttu-id="039a8-292">この制限は、"*同一オリジン ポリシー*" と呼ばれ、悪意のあるサイトが、別のサイトから機密データを読み取れないようにします。</span><span class="sxs-lookup"><span data-stu-id="039a8-292">This restriction is called the *same-origin policy* and prevents a malicious site from reading sensitive data from another site.</span></span> <span data-ttu-id="039a8-293">クロス オリジン要求を有効にするには、クロス オリジン リソース共有 (CORS) [ポリシー][cors-policy]を API Management ゲートウェイに追加します。</span><span class="sxs-lookup"><span data-stu-id="039a8-293">To enable a cross-origin request, add a Cross-Origin Resource Sharing (CORS) [policy][cors-policy] to the API Management gateway:</span></span>

```xml
<cors allow-credentials="true">
    <allowed-origins>
        <origin>[Website URL]</origin>
    </allowed-origins>
    <allowed-methods>
        <method>GET</method>
    </allowed-methods>
    <allowed-headers>
        <header>*</header>
    </allowed-headers>
</cors>
```

<span data-ttu-id="039a8-294">この例では、**allow-credentials** 属性が **true** になっています。</span><span class="sxs-lookup"><span data-stu-id="039a8-294">In this example, the **allow-credentials** attribute is **true**.</span></span> <span data-ttu-id="039a8-295">これにより、ブラウザーが要求と共に資格情報 (Cookie など) を送信することが承認されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-295">This authorizes the browser to send credentials (including cookies) with the request.</span></span> <span data-ttu-id="039a8-296">それ以外の場合、既定では、ブラウザーからクロス オリジン要求と共に資格情報が送信されません。</span><span class="sxs-lookup"><span data-stu-id="039a8-296">Otherwise, by default the browser does not send credentials with a cross-origin request.</span></span>

> [!NOTE]
> <span data-ttu-id="039a8-297">**allow-credentials** を **true** に設定すると、Web サイトが、ユーザーの資格情報を、そのユーザーが知らないところでユーザーの代わりに API に送信できるようになるため、注意が必要です。</span><span class="sxs-lookup"><span data-stu-id="039a8-297">Be very careful about setting **allow-credentials** to **true**, because it means a website can send the user's credentials to your API on the user's behalf, without the user being aware.</span></span> <span data-ttu-id="039a8-298">許可されたオリジンを信頼する必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-298">You must trust the allowed origin.</span></span>

### <a name="enforce-https"></a><span data-ttu-id="039a8-299">HTTPS の適用</span><span class="sxs-lookup"><span data-stu-id="039a8-299">Enforce HTTPS</span></span>

<span data-ttu-id="039a8-300">セキュリティを最大限に高めるには、要求パイプライン全体で HTTPS を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-300">For maximum security, require HTTPS throughout the request pipeline:</span></span>

- <span data-ttu-id="039a8-301">**CDN**。</span><span class="sxs-lookup"><span data-stu-id="039a8-301">**CDN**.</span></span> <span data-ttu-id="039a8-302">既定では、Azure CDN は、`*.azureedge.net` サブドメインで HTTPS をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="039a8-302">Azure CDN supports HTTPS on the `*.azureedge.net` subdomain by default.</span></span> <span data-ttu-id="039a8-303">CDN でカスタム ドメイン名に対して HTTPS を有効にするには、「[チュートリアル:Azure CDN カスタム ドメインで HTTPS を構成する][cdn-https]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-303">To enable HTTPS in the CDN for custom domain names, see [Tutorial: Configure HTTPS on an Azure CDN custom domain][cdn-https].</span></span>

- <span data-ttu-id="039a8-304">**静的 Web サイトのホスティング**。</span><span class="sxs-lookup"><span data-stu-id="039a8-304">**Static website hosting**.</span></span> <span data-ttu-id="039a8-305">ストレージ アカウントで [[安全な転送が必須]][storage-https] オプションを有効にします。</span><span class="sxs-lookup"><span data-stu-id="039a8-305">Enable the "[Secure transfer required][storage-https]" option on the Storage account.</span></span> <span data-ttu-id="039a8-306">このオプションを有効にすると、ストレージ アカウントでは、セキュリティで保護された HTTPS 接続からの要求のみが許可されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-306">When this option is enabled, the storage account only allows requests from secure HTTPS connections.</span></span>

- <span data-ttu-id="039a8-307">**API Management**。</span><span class="sxs-lookup"><span data-stu-id="039a8-307">**API Management**.</span></span> <span data-ttu-id="039a8-308">HTTPS プロトコルのみを使用するように API を構成します。</span><span class="sxs-lookup"><span data-stu-id="039a8-308">Configure the APIs to use HTTPS protocol only.</span></span> <span data-ttu-id="039a8-309">これは、Azure portal または Resource Manager テンプレートを使用して構成できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-309">You can configure this in the Azure portal or through a Resource Manager template:</span></span>

    ```json
    {
        "apiVersion": "2018-01-01",
        "type": "apis",
        "name": "dronedeliveryapi",
        "dependsOn": [
            "[concat('Microsoft.ApiManagement/service/', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "displayName": "Drone Delivery API",
            "description": "Drone Delivery API",
            "path": "api",
            "protocols": [ "HTTPS" ]
        },
        ...
    }
    ```

- <span data-ttu-id="039a8-310">**Azure Functions**。</span><span class="sxs-lookup"><span data-stu-id="039a8-310">**Azure Functions**.</span></span> <span data-ttu-id="039a8-311">[[HTTPS のみ]][functions-https] の設定を有効にします。</span><span class="sxs-lookup"><span data-stu-id="039a8-311">Enable the "[HTTPS Only][functions-https]" setting.</span></span>

### <a name="lock-down-the-function-app"></a><span data-ttu-id="039a8-312">関数アプリのロックダウン</span><span class="sxs-lookup"><span data-stu-id="039a8-312">Lock down the function app</span></span>

<span data-ttu-id="039a8-313">関数へのすべての呼び出しが、API ゲートウェイを経由する必要があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-313">All calls to the function should go through the API gateway.</span></span> <span data-ttu-id="039a8-314">これは次のように実現できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-314">You can achieve this as follows:</span></span>

- <span data-ttu-id="039a8-315">関数キーを必要とするように関数アプリを構成します。</span><span class="sxs-lookup"><span data-stu-id="039a8-315">Configure the function app to require a function key.</span></span> <span data-ttu-id="039a8-316">API Management ゲートウェイが関数アプリを呼び出すと、そのゲートウェイに関数キーが追加されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-316">The API Management gateway will include the function key when it calls the function app.</span></span> <span data-ttu-id="039a8-317">これにより、クライアントはゲートウェイをバイパスして直接関数を呼び出すことができなくなります。</span><span class="sxs-lookup"><span data-stu-id="039a8-317">This prevents clients from calling the function directly, bypassing the gateway.</span></span>

- <span data-ttu-id="039a8-318">API Management ゲートウェイには[静的 IP アドレス][apim-ip]があります。</span><span class="sxs-lookup"><span data-stu-id="039a8-318">The API Management gateway has a [static IP address][apim-ip].</span></span> <span data-ttu-id="039a8-319">その静的 IP アドレスからの呼び出しのみを許可するように Azure 関数を制限します。</span><span class="sxs-lookup"><span data-stu-id="039a8-319">Restrict the Azure Function to allow only calls from that static IP address.</span></span> <span data-ttu-id="039a8-320">詳細については、「[Azure App Service 静的 IP 制限][app-service-ip-restrictions]」を参照してください </span><span class="sxs-lookup"><span data-stu-id="039a8-320">For more information, see [Azure App Service Static IP Restrictions][app-service-ip-restrictions].</span></span> <span data-ttu-id="039a8-321">(この機能は、Standard レベルのサービスでのみ使用できます)。</span><span class="sxs-lookup"><span data-stu-id="039a8-321">(This feature is available for Standard tier services only.)</span></span>

### <a name="protect-application-secrets"></a><span data-ttu-id="039a8-322">アプリケーション シークレットの保護</span><span class="sxs-lookup"><span data-stu-id="039a8-322">Protect application secrets</span></span>

<span data-ttu-id="039a8-323">データベースの資格情報などのアプリケーション シークレット情報を、コードや構成ファイルに保存しないでください。</span><span class="sxs-lookup"><span data-stu-id="039a8-323">Don't store application secrets, such as database credentials, in your code or configuration files.</span></span> <span data-ttu-id="039a8-324">代わりに、暗号化された状態で Azure に保存されているアプリ設定を使用します。</span><span class="sxs-lookup"><span data-stu-id="039a8-324">Instead, use App settings, which are stored encrypted in Azure.</span></span> <span data-ttu-id="039a8-325">詳細については、「[Azure App Service と Azure Functions のセキュリティ][app-service-security]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-325">For more information, see [Security in Azure App Service and Azure Functions][app-service-security].</span></span>

<span data-ttu-id="039a8-326">また、アプリケーション シークレット情報を Key Vault に格納することもできます。</span><span class="sxs-lookup"><span data-stu-id="039a8-326">Alternatively, you can store application secrets in Key Vault.</span></span> <span data-ttu-id="039a8-327">これにより、シークレットのストレージを一元化し、その配布を制御して、シークレットへのアクセス方法とそのタイミングを監視することができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-327">This allows you to centralize the storage of secrets, control their distribution, and monitor how and when secrets are being accessed.</span></span> <span data-ttu-id="039a8-328">詳細については、[キー コンテナーからシークレットを読み取るように Azure Web アプリケーションを構成する][key-vault-web-app]方法に関するチュートリアルをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="039a8-328">For more information, see [Configure an Azure web application to read a secret from Key Vault][key-vault-web-app].</span></span> <span data-ttu-id="039a8-329">ただし、その構成設定は、関数のトリガーとバインディングにより、アプリ設定から読み込まれることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-329">However, note that Functions triggers and bindings load their configuration settings from app settings.</span></span> <span data-ttu-id="039a8-330">Key Vault のシークレットを使用するようにトリガーとバインディングを構成する方法は、組み込まれていません。</span><span class="sxs-lookup"><span data-stu-id="039a8-330">There is no built-in way to configure the triggers and bindings to use Key Vault secrets.</span></span>

## <a name="devops-considerations"></a><span data-ttu-id="039a8-331">DevOps の考慮事項</span><span class="sxs-lookup"><span data-stu-id="039a8-331">DevOps considerations</span></span>

### <a name="deployment"></a><span data-ttu-id="039a8-332">Deployment</span><span class="sxs-lookup"><span data-stu-id="039a8-332">Deployment</span></span>

<span data-ttu-id="039a8-333">関数アプリをデプロイするには、[パッケージ ファイル][functions-run-from-package]を使用することをお勧めします ("パッケージから実行")。</span><span class="sxs-lookup"><span data-stu-id="039a8-333">To deploy the function app, we recommend using [package files][functions-run-from-package] ("Run from package").</span></span> <span data-ttu-id="039a8-334">このアプローチを使用して、ZIP ファイルを Blob Storage コンテナーにアップロードすると、ZIP ファイルは、Functions ランタイムによって読み取り専用ファイル システムとしてマウントされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-334">Using this approach, you upload a zip file to a Blob Storage container and the Functions runtime mounts the zip file as a read-only file system.</span></span> <span data-ttu-id="039a8-335">これはアトミック操作で、デプロイの失敗によりアプリケーションが不整合な状態のままになる可能性が少なくなります。</span><span class="sxs-lookup"><span data-stu-id="039a8-335">This is an atomic operation, which reduces the chance that a failed deployment will leave the application in an inconsistent state.</span></span> <span data-ttu-id="039a8-336">また、すべてのファイルが一度にスワップされるため、特に Node.js アプリについては、コールド スタート時間を改善することができます。</span><span class="sxs-lookup"><span data-stu-id="039a8-336">It can also improve cold start times, especially for Node.js apps, because all of the files are swapped at once.</span></span>

### <a name="api-versioning"></a><span data-ttu-id="039a8-337">API のバージョン管理</span><span class="sxs-lookup"><span data-stu-id="039a8-337">API versioning</span></span>

<span data-ttu-id="039a8-338">API は、サービスとクライアントの間のコントラクトです。</span><span class="sxs-lookup"><span data-stu-id="039a8-338">An API is a contract between a service and clients.</span></span> <span data-ttu-id="039a8-339">このアーキテクチャでは、API コントラクトは、API Management レイヤーで定義されます。</span><span class="sxs-lookup"><span data-stu-id="039a8-339">In this architecture, the API contract is defined at the API Management layer.</span></span> <span data-ttu-id="039a8-340">API Management では、次に示すように、2 つの異なる (ただし補完的な) [バージョン管理の概念][apim-versioning]がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="039a8-340">API Management supports two distinct but complementary [versioning concepts][apim-versioning]:</span></span>

- <span data-ttu-id="039a8-341">"*バージョン*" により、API のコンシューマーはニーズに基づいて API のバージョン (v1、v2 など) を選択できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-341">*Versions* allow API consumers to choose an API version based on their needs, such as v1 versus v2.</span></span>

- <span data-ttu-id="039a8-342">"*リビジョン*" により、API 管理者が API の非破壊的変更を行い、これらの変更をデプロイできるようになります。その際、API コンシューマーにこれらの変更を通知する変更ログもデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="039a8-342">*Revisions* allow API administrators to make non-breaking changes in an API and deploy those changes, along with a change log to inform API consumers about the changes.</span></span>

<span data-ttu-id="039a8-343">API で破壊的変更を行う場合は、API Management で新しいバージョンを発行します。</span><span class="sxs-lookup"><span data-stu-id="039a8-343">If you make a breaking change in an API, publish a new version in API Management.</span></span> <span data-ttu-id="039a8-344">新しいバージョンは、別の関数アプリで、元のバージョンと共にデプロイしてください。</span><span class="sxs-lookup"><span data-stu-id="039a8-344">Deploy the new version side-by-side with the original version, in a separate Function App.</span></span> <span data-ttu-id="039a8-345">これにより、クライアント アプリケーションを中断することなく、既存のクライアントを新しい API に移行できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-345">This lets you migrate existing clients to the new API without breaking client applications.</span></span> <span data-ttu-id="039a8-346">最終的には、以前のバージョンを廃止できます。</span><span class="sxs-lookup"><span data-stu-id="039a8-346">Eventually, you can deprecate the previous version.</span></span> <span data-ttu-id="039a8-347">API Management では、URL のパス、HTTP ヘッダー、クエリ文字列など、複数の[バージョン管理スキーム][apim-versioning-schemes]がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="039a8-347">API Management supports several [versioning schemes][apim-versioning-schemes]: URL path, HTTP header, or query string.</span></span> <span data-ttu-id="039a8-348">一般的な API のバージョン管理の詳細については、「[RESTful Web API のバージョン管理][api-versioning]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="039a8-348">For more information about API versioning in general, see [Versioning a RESTful web API][api-versioning].</span></span>

<span data-ttu-id="039a8-349">API の破壊的変更とはならない更新プログラムの場合、新しいバージョンは、同じ関数アプリのステージング スロットにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="039a8-349">For updates that are not breaking API changes, deploy the new version to a staging slot in the same Function App.</span></span> <span data-ttu-id="039a8-350">デプロイが成功したことを確認したら、ステージング バージョンを運用バージョンと入れ替えます。</span><span class="sxs-lookup"><span data-stu-id="039a8-350">Verify the deployment succeeded and then swap the staged version with the production version.</span></span> <span data-ttu-id="039a8-351">API Management でリビジョンを発行します。</span><span class="sxs-lookup"><span data-stu-id="039a8-351">Publish a revision in API Management.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="039a8-352">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="039a8-352">Deploy the solution</span></span>

<span data-ttu-id="039a8-353">この参照アーキテクチャをデプロイするには、[GitHub の Readme][readme] をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="039a8-353">To deploy this reference architecture, view the [GitHub readme][readme].</span></span>

<!-- links -->

[api-versioning]: ../../best-practices/api-design.md#versioning-a-restful-web-api
[apim]: /azure/api-management/api-management-key-concepts
[apim-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[api-geo]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-scale]: /azure/api-management/api-management-howto-autoscale
[apim-validate-jwt]: /azure/api-management/api-management-access-restriction-policies#ValidateJWT
[apim-versioning]: /azure/api-management/api-management-get-started-publish-versions
[apim-versioning-schemes]: /azure/api-management/api-management-get-started-publish-versions#choose-a-versioning-scheme
[app-service-auth]: /azure/app-service/app-service-authentication-overview
[app-service-ip-restrictions]: /azure/app-service/app-service-ip-restrictions
[app-service-security]: /azure/app-service/app-service-security
[ase]: /azure/app-service/environment/intro
[azure-messaging]: /azure/event-grid/compare-messaging-services
[claims]: https://en.wikipedia.org/wiki/Claims-based_identity
[cdn]: https://azure.microsoft.com/services/cdn/
[cdn-https]: /azure/cdn/cdn-custom-ssl
[cors-policy]: /azure/api-management/api-management-cross-domain-policies
[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-input-binding]: /azure/azure-functions/functions-bindings-cosmosdb-v2#input
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[event-driven]: ../../guide/architecture-styles/event-driven.md
[functions]: /azure/azure-functions/functions-overview
[functions-bindings]: /azure/azure-functions/functions-triggers-bindings
[functions-cold-start]: https://blogs.msdn.microsoft.com/appserviceteam/2018/02/07/understanding-serverless-cold-start/
[functions-https]: /azure/app-service/app-service-web-tutorial-custom-ssl#enforce-https
[functions-proxy]: /azure/azure-functions/functions-proxies
[functions-run-from-package]: /azure/azure-functions/run-functions-from-deployment-package
[functions-scale]: /azure/azure-functions/functions-scale
[functions-timeout]: /azure/azure-functions/functions-scale#consumption-plan
[functions-zip-deploy]: /azure/azure-functions/deployment-zip-push
[graph]: https://developer.microsoft.com/graph/docs/concepts/overview
[key-vault-web-app]: /azure/key-vault/tutorial-web-application-keyvault
[microservices-domain-analysis]: ../../microservices/domain-analysis.md
[monitor]: /azure/azure-monitor/overview
[oauth-flow]: https://auth0.com/docs/api-auth/which-oauth-flow-to-use
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[ru]: /azure/cosmos-db/request-units
[scopes]: /azure/active-directory/develop/v2-permissions-and-consent
[static-hosting]: /azure/storage/blobs/storage-blob-static-website
[static-hosting-preview]: https://azure.microsoft.com/blog/azure-storage-static-web-hosting-public-preview/
[storage-https]: /azure/storage/common/storage-require-secure-transfer
[tm]: /azure/traffic-manager/traffic-manager-overview

[github]: https://github.com/mspnp/serverless-reference-implementation
[HttpRequestAuthorizationExtensions]: https://github.com/mspnp/serverless-reference-implementation/blob/master/src/DroneStatus/dotnet/DroneStatusFunctionApp/HttpRequestAuthorizationExtensions.cs
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md
