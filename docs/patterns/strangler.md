---
title: ストラングラー パターン
titleSuffix: Cloud Design Patterns
description: 機能の特定の部分を新しいアプリケーションやサービスに徐々に置き換えることで、レガシ システムを段階的に移行します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f139d368c98256c0190753930983a47df81a5134
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480571"
---
# <a name="strangler-pattern"></a><span data-ttu-id="1e497-104">ストラングラー パターン</span><span class="sxs-lookup"><span data-stu-id="1e497-104">Strangler pattern</span></span>

<span data-ttu-id="1e497-105">機能の特定の部分を新しいアプリケーションやサービスに徐々に置き換えることで、レガシ システムを段階的に移行します。</span><span class="sxs-lookup"><span data-stu-id="1e497-105">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="1e497-106">レガシ システムからの機能が置き換えられていくと、新しいシステムは最終的に古いシステムの機能すべてを置き換え、古いシステムを抑圧して使用停止できるようにします。</span><span class="sxs-lookup"><span data-stu-id="1e497-106">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1e497-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="1e497-107">Context and problem</span></span>

<span data-ttu-id="1e497-108">システムが古くなるにつれ、このシステムが構築された開発ツール、ホスティング テクノロジ、システム アーキテクチャも徐々に使われなくなっていきます。</span><span class="sxs-lookup"><span data-stu-id="1e497-108">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="1e497-109">新機能が追加されると、これらのアプリケーションも大幅に複雑化し、メンテナンスや新機能の追加が難しくなっていきます。</span><span class="sxs-lookup"><span data-stu-id="1e497-109">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="1e497-110">複雑なシステムを完全に置き換えるには、膨大な作業が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="1e497-110">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="1e497-111">多くの場合、まだ移行されていない機能を古いシステムで処理し続けながら、新しいシステムに段階的に移行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1e497-111">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="1e497-112">ただし、2 つの異なるバージョンのアプリケーションを実行している場合、クライアントが特定の機能の場所を把握している必要があることになります。</span><span class="sxs-lookup"><span data-stu-id="1e497-112">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="1e497-113">機能またはサービスが移行されるたびに、クライアントを更新して、新しい場所が示されるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="1e497-113">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="1e497-114">解決策</span><span class="sxs-lookup"><span data-stu-id="1e497-114">Solution</span></span>

<span data-ttu-id="1e497-115">段階的に、機能の特定の部分を新しいアプリケーションやサービスに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="1e497-115">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="1e497-116">バックエンド レガシ システムに送信される要求をインターセプトするファサードを作成します。</span><span class="sxs-lookup"><span data-stu-id="1e497-116">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="1e497-117">ファサードは、これらの要求をレガシ アプリケーションまたは新しいサービスにルーティングします。</span><span class="sxs-lookup"><span data-stu-id="1e497-117">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="1e497-118">既存の機能は段階的に新しいシステムに移行でき、コンシューマーは、移行が行われていることに気付くことなく、同じインターフェイスを引き続き使用できます。</span><span class="sxs-lookup"><span data-stu-id="1e497-118">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![ストラングラー パターンの図](./_images/strangler.png)

<span data-ttu-id="1e497-120">このパターンは、移行によるリスクを最小化し、長期にわたって開発を分散させるのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1e497-120">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="1e497-121">ファサードを使用すると、ユーザーを正しいアプリケーションに安全にルーティングし、レガシ アプリケーションが引き続き機能するようにしながら、任意のペースで新しいシステムに機能を追加できます。</span><span class="sxs-lookup"><span data-stu-id="1e497-121">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="1e497-122">時間の経過とともに、機能が新しいシステムに移行されると、レガシ システムは最終的に "抑圧" されて、必要なくなります。</span><span class="sxs-lookup"><span data-stu-id="1e497-122">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="1e497-123">このプロセスが完了すると、レガシ システムを安全に廃止できます。</span><span class="sxs-lookup"><span data-stu-id="1e497-123">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="1e497-124">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="1e497-124">Issues and considerations</span></span>

- <span data-ttu-id="1e497-125">新旧両方のシステムで使用される可能性のあるサービスとデータ ストアを処理する方法を検討してください。</span><span class="sxs-lookup"><span data-stu-id="1e497-125">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="1e497-126">両方が並列でこれらのリソースにアクセスできるようにしてください。</span><span class="sxs-lookup"><span data-stu-id="1e497-126">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="1e497-127">新しいアプリケーションおよびサービスを、これらのインターセプトと将来のストラングラー移行による置き換えが簡単になるように構成します。</span><span class="sxs-lookup"><span data-stu-id="1e497-127">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="1e497-128">ある時点で、移行が完了すると、ストラングラー ファサードは消失するか、レガシ クライアントのアダプターに進化します。</span><span class="sxs-lookup"><span data-stu-id="1e497-128">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="1e497-129">ファサードが移行に対して古くならないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="1e497-129">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="1e497-130">ファサードが単一障害点やパフォーマンスのボトルネックとならないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="1e497-130">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="1e497-131">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="1e497-131">When to use this pattern</span></span>

<span data-ttu-id="1e497-132">バックエンド アプリケーションを新しいアーキテクチャに段階的に移行する場合に、このパターンを使用します。</span><span class="sxs-lookup"><span data-stu-id="1e497-132">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="1e497-133">このパターンは、次の場合には適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="1e497-133">This pattern may not be suitable:</span></span>

- <span data-ttu-id="1e497-134">バックエンド システムへの要求がインターセプトされない場合。</span><span class="sxs-lookup"><span data-stu-id="1e497-134">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="1e497-135">システム全体の置き換えの複雑さが少ない、小規模なシステムの場合。</span><span class="sxs-lookup"><span data-stu-id="1e497-135">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="1e497-136">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="1e497-136">Related guidance</span></span>

- <span data-ttu-id="1e497-137">[StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html) に関する Martin Fowler のブログ記事</span><span class="sxs-lookup"><span data-stu-id="1e497-137">Martin Fowler's blog post on [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)</span></span>
