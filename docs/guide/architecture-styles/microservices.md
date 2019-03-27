---
title: マイクロサービス アーキテクチャ スタイル
titleSuffix: Azure Application Architecture Guide
description: Azure のマイクロサービス アーキテクチャのメリット、課題、ベスト プラクティスについて説明します。
author: MikeWasson
ms.date: 11/13/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, microservices
ms.openlocfilehash: cc72f61003f4146fd65e501feebda0c0d1d27993
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245093"
---
# <a name="microservices-architecture-style"></a><span data-ttu-id="b10e5-103">マイクロサービス アーキテクチャ スタイル</span><span class="sxs-lookup"><span data-stu-id="b10e5-103">Microservices architecture style</span></span>

<span data-ttu-id="b10e5-104">マイクロサービス アーキテクチャは、小さな自律サービスのコレクションで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b10e5-104">A microservices architecture consists of a collection of small, autonomous services.</span></span> <span data-ttu-id="b10e5-105">各サービスは自己完結型であり、1 つのビジネス機能を実装している必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-105">Each service is self-contained and should implement a single business capability.</span></span>

![マイクロサービス アーキテクチャ スタイルの論理図](./images/microservices-logical.svg)

<span data-ttu-id="b10e5-107">いくつかの点で、マイクロサービスはサービス指向アーキテクチャ (SOA) の自然な進化形と言えますが、マイクロサービスと SOA にはいくつかの違いがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-107">In some ways, microservices are the natural evolution of service oriented architectures (SOA), but there are differences between microservices and SOA.</span></span> <span data-ttu-id="b10e5-108">次に、マイクロサービスの特徴を定義します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-108">Here are some defining characteristics of a microservice:</span></span>

- <span data-ttu-id="b10e5-109">マイクロサービス アーキテクチャでは、サービスは小さく、独立的で、疎結合しています。</span><span class="sxs-lookup"><span data-stu-id="b10e5-109">In a microservices architecture, services are small, independent, and loosely coupled.</span></span>

- <span data-ttu-id="b10e5-110">各サービスは個別のコードベースであり、小規模な開発チームで管理できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-110">Each service is a separate codebase, which can be managed by a small development team.</span></span>

- <span data-ttu-id="b10e5-111">サービスは個別にデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-111">Services can be deployed independently.</span></span> <span data-ttu-id="b10e5-112">チームは、アプリケーション全体を再構築したり再デプロイしたりすることなく、既存のサービスを更新できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-112">A team can update an existing service without rebuilding and redeploying the entire application.</span></span>

- <span data-ttu-id="b10e5-113">サービスはサービスのデータや外部の状態を保持する役割を担います。</span><span class="sxs-lookup"><span data-stu-id="b10e5-113">Services are responsible for persisting their own data or external state.</span></span> <span data-ttu-id="b10e5-114">これは、個別のデータ層でデータを保持する従来のモデルと異なる点です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-114">This differs from the traditional model, where a separate data layer handles data persistence.</span></span>

- <span data-ttu-id="b10e5-115">サービスは、明確に定義された API を使用して、互いに通信します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-115">Services communicate with each other by using well-defined APIs.</span></span> <span data-ttu-id="b10e5-116">各サービス内部の実装の詳細は、他のサービスに開示されません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-116">Internal implementation details of each service are hidden from other services.</span></span>

- <span data-ttu-id="b10e5-117">サービスは、同じテクノロジ スタック、ライブラリ、またはフレームワークを共有する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-117">Services don't need to share the same technology stack, libraries, or frameworks.</span></span>

<span data-ttu-id="b10e5-118">また、サービス自体については、いくつかの他のコンポーネントが次のような一般的なマイクロサービス アーキテクチャで使用されます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-118">Besides for the services themselves, some other components appear in a typical microservices architecture:</span></span>

<span data-ttu-id="b10e5-119">**管理**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-119">**Management**.</span></span> <span data-ttu-id="b10e5-120">管理コンポーネントは、ノードへのサービスの配置、障害の特定、ノード間のサービスの再調整などを行う役割を担います。</span><span class="sxs-lookup"><span data-stu-id="b10e5-120">The management component is responsible for placing services on nodes, identifying failures, rebalancing services across nodes, and so forth.</span></span>

<span data-ttu-id="b10e5-121">**サービスの探索**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-121">**Service Discovery**.</span></span> <span data-ttu-id="b10e5-122">サービスのリストとサービスが配置されているノードを保持します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-122">Maintains a list of services and which nodes they are located on.</span></span> <span data-ttu-id="b10e5-123">サービスがサービスのためにエンドポイントを探索できるようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-123">Enables service lookup to find the endpoint for a service.</span></span>

<span data-ttu-id="b10e5-124">**API ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-124">**API Gateway**.</span></span> <span data-ttu-id="b10e5-125">API ゲートウェイは、クライアントのエントリ ポイントです。</span><span class="sxs-lookup"><span data-stu-id="b10e5-125">The API gateway is the entry point for clients.</span></span> <span data-ttu-id="b10e5-126">クライアントは直接サービスを呼び出すことはしません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-126">Clients don't call services directly.</span></span> <span data-ttu-id="b10e5-127">代わりに API ゲートウェイを呼び出し、その呼び出しをバック エンドの適切なサービスに転送します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-127">Instead, they call the API gateway, which forwards the call to the appropriate services on the back end.</span></span> <span data-ttu-id="b10e5-128">API ゲートウェイは、複数のサービスからの応答を集計して、集計された応答を返すことがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-128">The API gateway might aggregate the responses from several services and return the aggregated response.</span></span>

<span data-ttu-id="b10e5-129">API ゲートウェイを使用することには次のようなメリットがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-129">The advantages of using an API gateway include:</span></span>

- <span data-ttu-id="b10e5-130">サービスからクライアントを切り離します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-130">It decouples clients from services.</span></span> <span data-ttu-id="b10e5-131">サービスのバージョン管理とリファクタリングを、すべてのクライアントを更新する必要なく行うことができます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-131">Services can be versioned or refactored without needing to update all of the clients.</span></span>

- <span data-ttu-id="b10e5-132">サービスは、AMQP などの Web フレンドリではないメッセージング プロトコルを使用できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-132">Services can use messaging protocols that are not web friendly, such as AMQP.</span></span>

- <span data-ttu-id="b10e5-133">API ゲートウェイは、認証、ログ記録、SSL 終了、負荷分散などの他の横断的な関数を実行できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-133">The API Gateway can perform other cross-cutting functions such as authentication, logging, SSL termination, and load balancing.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="b10e5-134">このアーキテクチャを使用する状況</span><span class="sxs-lookup"><span data-stu-id="b10e5-134">When to use this architecture</span></span>

<span data-ttu-id="b10e5-135">次の場合に、このアーキテクチャ スタイルを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-135">Consider this architecture style for:</span></span>

- <span data-ttu-id="b10e5-136">リリースの早さが要求される大規模アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="b10e5-136">Large applications that require a high release velocity.</span></span>

- <span data-ttu-id="b10e5-137">高い拡張性が必要とされる複雑なアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="b10e5-137">Complex applications that need to be highly scalable.</span></span>

- <span data-ttu-id="b10e5-138">ドメインが豊富な、またはサブドメインが多いアプリケーション。</span><span class="sxs-lookup"><span data-stu-id="b10e5-138">Applications with rich domains or many subdomains.</span></span>

- <span data-ttu-id="b10e5-139">小規模な開発チームから構成されている組織。</span><span class="sxs-lookup"><span data-stu-id="b10e5-139">An organization that consists of small development teams.</span></span>

## <a name="benefits"></a><span data-ttu-id="b10e5-140">メリット</span><span class="sxs-lookup"><span data-stu-id="b10e5-140">Benefits</span></span>

- <span data-ttu-id="b10e5-141">**独立したデプロイ**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-141">**Independent deployments**.</span></span> <span data-ttu-id="b10e5-142">アプリケーション全体を再デプロイしなくてもサービスを更新できたり、問題が生じた場合に更新プログラムをロールバックまたはロールフォワードできたりします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-142">You can update a service without redeploying the entire application, and roll back or roll forward an update if something goes wrong.</span></span> <span data-ttu-id="b10e5-143">バグ修正プログラムと機能のリリースがより管理しやすく、リスクが低くなります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-143">Bug fixes and feature releases are more manageable and less risky.</span></span>

- <span data-ttu-id="b10e5-144">**独立した開発**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-144">**Independent development**.</span></span> <span data-ttu-id="b10e5-145">1 つの開発チームでサービスのビルド、テスト、デプロイが可能です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-145">A single development team can build, test, and deploy a service.</span></span> <span data-ttu-id="b10e5-146">その結果、イノベーションを継続し、より早い周期でリリースが可能になります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-146">The result is continuous innovation and a faster release cadence.</span></span>

- <span data-ttu-id="b10e5-147">**集中的な小規模チーム**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-147">**Small, focused teams**.</span></span> <span data-ttu-id="b10e5-148">チームは 1 つのサービスに専念できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-148">Teams can focus on one service.</span></span> <span data-ttu-id="b10e5-149">各サービスの範囲が狭いことでコード ベースが理解しやすく、新しいチーム メンバーを増員しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-149">The smaller scope of each service makes the code base easier to understand, and it's easier for new team members to ramp up.</span></span>

- <span data-ttu-id="b10e5-150">**障害の分離**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-150">**Fault isolation**.</span></span> <span data-ttu-id="b10e5-151">サービスがダウンしても、アプリケーション全体を停止することはありません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-151">If a service goes down, it won't take out the entire application.</span></span> <span data-ttu-id="b10e5-152">ただし、無料で回復性を得られるという意味ではありません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-152">However, that doesn't mean you get resiliency for free.</span></span> <span data-ttu-id="b10e5-153">回復性のベスト プラクティスと設計パターンに従う必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-153">You still need to follow resiliency best practices and design patterns.</span></span> <span data-ttu-id="b10e5-154">「[Designing resilient applications for Azure (回復性に優れた Azure 用アプリケーションの設計)][resiliency-overview]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-154">See [Designing resilient applications for Azure][resiliency-overview].</span></span>

- <span data-ttu-id="b10e5-155">**混在テクノロジ スタック**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-155">**Mixed technology stacks**.</span></span> <span data-ttu-id="b10e5-156">チームは、自分たちのサービスに最適なテクノロジを選択できます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-156">Teams can pick the technology that best fits their service.</span></span>

- <span data-ttu-id="b10e5-157">**詳細なスケーリング**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-157">**Granular scaling**.</span></span> <span data-ttu-id="b10e5-158">サービスは個別にスケールできます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-158">Services can be scaled independently.</span></span> <span data-ttu-id="b10e5-159">同時に、VM あたりのサービスの密度が高いということは、VM リソースがフルに活用されるということです。</span><span class="sxs-lookup"><span data-stu-id="b10e5-159">At the same time, the higher density of services per VM means that VM resources are fully utilized.</span></span> <span data-ttu-id="b10e5-160">配置の制約を使用して、サービスを VM プロファイル (高 CPU、ハイ メモリなど) に合わせることができます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-160">Using placement constraints, a services can be matched to a VM profile (high CPU, high memory, and so on).</span></span>

## <a name="challenges"></a><span data-ttu-id="b10e5-161">課題</span><span class="sxs-lookup"><span data-stu-id="b10e5-161">Challenges</span></span>

- <span data-ttu-id="b10e5-162">**複雑さ**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-162">**Complexity**.</span></span> <span data-ttu-id="b10e5-163">マイクロサービス アプリケーションは、同等のモノリシック アプリケーションよりも多くの動的パーツで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b10e5-163">A microservices application has more moving parts than the equivalent monolithic application.</span></span> <span data-ttu-id="b10e5-164">各サービスはより単純ですが、システム全体としてはより複雑です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-164">Each service is simpler, but the entire system as a whole is more complex.</span></span>

- <span data-ttu-id="b10e5-165">**開発とテスト**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-165">**Development and test**.</span></span> <span data-ttu-id="b10e5-166">サービスの依存関係に対する開発では、別のアプローチが必要です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-166">Developing against service dependencies requires a different approach.</span></span> <span data-ttu-id="b10e5-167">既存のツールは、必ずしもサービスの依存関係を処理するようには設計されていません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-167">Existing tools are not necessarily designed to work with service dependencies.</span></span> <span data-ttu-id="b10e5-168">サービス間の境界にまたがってリファクタリングを行うことは困難です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-168">Refactoring across service boundaries can be difficult.</span></span> <span data-ttu-id="b10e5-169">また、アプリケーションの進化が早い場合は特に、サービスの依存関係をテストすることは困難です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-169">It is also challenging to test service dependencies, especially when the application is evolving quickly.</span></span>

- <span data-ttu-id="b10e5-170">**ガバナンスの欠如**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-170">**Lack of governance**.</span></span> <span data-ttu-id="b10e5-171">マイクロサービスを構築する際に分散アプローチをとることにはメリットがありますが、問題が発生しやすくもなります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-171">The decentralized approach to building microservices has advantages, but it can also lead to problems.</span></span> <span data-ttu-id="b10e5-172">多くの異なる言語やフレームワークを使用するために、アプリケーションの維持が難しくなることがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-172">You may end up with so many different languages and frameworks that the application becomes hard to maintain.</span></span> <span data-ttu-id="b10e5-173">プロジェクト全体に適用される標準を導入する方が役立ち、チームの柔軟性を過度に制限せずに済むということもあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-173">It may be useful to put some project-wide standards in place, without overly restricting teams' flexibility.</span></span> <span data-ttu-id="b10e5-174">これは特に、ログ記録などの横断的な機能に適用されます。</span><span class="sxs-lookup"><span data-stu-id="b10e5-174">This especially applies to cross-cutting functionality such as logging.</span></span>

- <span data-ttu-id="b10e5-175">**ネットワークの輻輳と待機時間**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-175">**Network congestion and latency**.</span></span> <span data-ttu-id="b10e5-176">多くの小さく細分化されたサービスを使用すると、よりインタラクティブな通信が可能になります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-176">The use of many small, granular services can result in more interservice communication.</span></span> <span data-ttu-id="b10e5-177">また、サービスの依存関係のチェーンが長すぎる (サービス A が B を呼び出し、その B が C を呼び出すなどの) 場合、追加される待機時間が問題になることがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-177">Also, if the chain of service dependencies gets too long (service A calls B, which calls C...), the additional latency can become a problem.</span></span> <span data-ttu-id="b10e5-178">API は慎重に設計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-178">You will need to design APIs carefully.</span></span> <span data-ttu-id="b10e5-179">API を過度に使用することを避け、シリアル化形式について検討し、非同期通信パターンを使用する場所を探してください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-179">Avoid overly chatty APIs, think about serialization formats, and look for places to use asynchronous communication patterns.</span></span>

- <span data-ttu-id="b10e5-180">**データ整合性**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-180">**Data integrity**.</span></span> <span data-ttu-id="b10e5-181">独自のデータを永続化する各マイクロサービスを使用します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-181">With each microservice responsible for its own data persistence.</span></span> <span data-ttu-id="b10e5-182">結果として、データの整合性をとることが難しくなることがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-182">As a result, data consistency can be a challenge.</span></span> <span data-ttu-id="b10e5-183">可能であれば、最終的な整合性を優先します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-183">Embrace eventual consistency where possible.</span></span>

- <span data-ttu-id="b10e5-184">**管理**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-184">**Management**.</span></span> <span data-ttu-id="b10e5-185">マイクロサービスを成功させるには、成熟した DevOps カルチャが必要です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-185">To be successful with microservices requires a mature DevOps culture.</span></span> <span data-ttu-id="b10e5-186">サービス間で相互に関連付けられたログを記録することは難しい場合があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-186">Correlated logging across services can be challenging.</span></span> <span data-ttu-id="b10e5-187">通常、ログ記録は単一のユーザー操作を要求するために複数のサービスの呼び出しを関連付ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-187">Typically, logging must correlate multiple service calls for a single user operation.</span></span>

- <span data-ttu-id="b10e5-188">**バージョン管理**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-188">**Versioning**.</span></span> <span data-ttu-id="b10e5-189">サービスの更新プログラムは、依存しているサービスを中断してはいけません。</span><span class="sxs-lookup"><span data-stu-id="b10e5-189">Updates to a service must not break services that depend on it.</span></span> <span data-ttu-id="b10e5-190">複数のサービスが特定の時点で更新される場合があるため、慎重に設計しないと下位互換性または上位互換性に問題が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-190">Multiple services could be updated at any given time, so without careful design, you might have problems with backward or forward compatibility.</span></span>

- <span data-ttu-id="b10e5-191">**スキルセット**。</span><span class="sxs-lookup"><span data-stu-id="b10e5-191">**Skillset**.</span></span> <span data-ttu-id="b10e5-192">マイクロサービスは高度な分散システムです。</span><span class="sxs-lookup"><span data-stu-id="b10e5-192">Microservices are highly distributed systems.</span></span> <span data-ttu-id="b10e5-193">成功のために必要なスキルと経験がチームにあるかどうかを慎重に評価してください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-193">Carefully evaluate whether the team has the skills and experience to be successful.</span></span>

## <a name="best-practices"></a><span data-ttu-id="b10e5-194">ベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="b10e5-194">Best practices</span></span>

- <span data-ttu-id="b10e5-195">ビジネス分野周辺のモデル サービス。</span><span class="sxs-lookup"><span data-stu-id="b10e5-195">Model services around the business domain.</span></span>

- <span data-ttu-id="b10e5-196">すべてを分散化します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-196">Decentralize everything.</span></span> <span data-ttu-id="b10e5-197">個々のチームがサービスの設計と構築を担います。</span><span class="sxs-lookup"><span data-stu-id="b10e5-197">Individual teams are responsible for designing and building services.</span></span> <span data-ttu-id="b10e5-198">コードまたはデータのスキーマを共有しないようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-198">Avoid sharing code or data schemas.</span></span>

- <span data-ttu-id="b10e5-199">データ ストレージは、そのデータを所有するサービス専用にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-199">Data storage should be private to the service that owns the data.</span></span> <span data-ttu-id="b10e5-200">各サービスとデータの種類に最適なストレージを使用します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-200">Use the best storage for each service and data type.</span></span>

- <span data-ttu-id="b10e5-201">サービスが、適切に設計された API を介して通信するようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-201">Services communicate through well-designed APIs.</span></span> <span data-ttu-id="b10e5-202">実装の詳細が漏えいしないようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-202">Avoid leaking implementation details.</span></span> <span data-ttu-id="b10e5-203">API は、サービス内部の実装ではなく、ドメインをモデル化する必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-203">APIs should model the domain, not the internal implementation of the service.</span></span>

- <span data-ttu-id="b10e5-204">サービス間の結合は行わないようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-204">Avoid coupling between services.</span></span> <span data-ttu-id="b10e5-205">結合の原因には、共有データベース スキーマや固定の通信プロトコルなどがあります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-205">Causes of coupling include shared database schemas and rigid communication protocols.</span></span>

- <span data-ttu-id="b10e5-206">認証や SSL 終了などの横断的な懸念事項をゲートウェイにオフロードします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-206">Offload cross-cutting concerns, such as authentication and SSL termination, to the gateway.</span></span>

- <span data-ttu-id="b10e5-207">ドメインのナレッジをゲートウェイの外部で保持します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-207">Keep domain knowledge out of the gateway.</span></span> <span data-ttu-id="b10e5-208">ゲートウェイは、ビジネス ルールやドメイン ロジックのナレッジを使用せずに、クライアントの要求を処理し、ルーティングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-208">The gateway should handle and route client requests without any knowledge of the business rules or domain logic.</span></span> <span data-ttu-id="b10e5-209">そうしなければ、ゲートウェイが依存関係となり、サービス間の結合が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-209">Otherwise, the gateway becomes a dependency and can cause coupling between services.</span></span>

- <span data-ttu-id="b10e5-210">サービスには、疎結合と機能の高い凝集度が必要です。</span><span class="sxs-lookup"><span data-stu-id="b10e5-210">Services should have loose coupling and high functional cohesion.</span></span> <span data-ttu-id="b10e5-211">まとめて変更される可能性がある機能は、まとめてパッケージ化してデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-211">Functions that are likely to change together should be packaged and deployed together.</span></span> <span data-ttu-id="b10e5-212">それらの機能が別々のサービスに格納された場合、1 つのサービスの変更には他のサービスの更新が必要であるため、結果として密接に結合されることになります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-212">If they reside in separate services, those services end up being tightly coupled, because a change in one service will require updating the other service.</span></span> <span data-ttu-id="b10e5-213">2 つのサービス間で過度に通信が行われると、密接な結合と低い凝集度の問題が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b10e5-213">Overly chatty communication between two services may be a symptom of tight coupling and low cohesion.</span></span>

- <span data-ttu-id="b10e5-214">障害を分離します。</span><span class="sxs-lookup"><span data-stu-id="b10e5-214">Isolate failures.</span></span> <span data-ttu-id="b10e5-215">回復性戦略を使用して、サービス内の障害が連鎖しないようにします。</span><span class="sxs-lookup"><span data-stu-id="b10e5-215">Use resiliency strategies to prevent failures within a service from cascading.</span></span> <span data-ttu-id="b10e5-216">「[Resiliency patterns (回復性パターン)][resiliency-patterns]」と「[Designing resilient applications (回復性に優れたアプリケーションの設計)][resiliency-overview]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-216">See [Resiliency patterns][resiliency-patterns] and [Designing resilient applications][resiliency-overview].</span></span>

## <a name="next-steps"></a><span data-ttu-id="b10e5-217">次の手順</span><span class="sxs-lookup"><span data-stu-id="b10e5-217">Next steps</span></span>

<span data-ttu-id="b10e5-218">Azure でのマイクロサービス アーキテクチャの構築に関する詳細なガイダンスいついては、「[Azure でのマイクロサービスの設計、構築、および操作](../../microservices/index.md)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b10e5-218">For detailed guidance about building a microservices architecture on Azure, see [Designing, building, and operating microservices on Azure](../../microservices/index.md).</span></span>

<!-- links -->

[resiliency-overview]: ../../resiliency/index.md
[resiliency-patterns]: ../../patterns/category/resiliency.md
