---
title: "破損対策レイヤー パターン"
description: "最新アプリケーションとレガシ システムの間にファサード、すなわちアダプター レイヤーを実装します。"
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: e41f080abbef772596ee7f8b10ad72bb03a3b829
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/05/2018
---
# <a name="anti-corruption-layer-pattern"></a><span data-ttu-id="56d7d-103">破損対策レイヤー パターン</span><span class="sxs-lookup"><span data-stu-id="56d7d-103">Anti-Corruption Layer pattern</span></span>

<span data-ttu-id="56d7d-104">最新アプリケーションと、そのアプリケーションが依存するレガシ システムの間にファサード、すなわちアダプター レイヤーを実装します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-104">Implement a façade or adapter layer between a modern application and a legacy system that it depends on.</span></span> <span data-ttu-id="56d7d-105">このレイヤーにより、最新アプリケーションとレガシ システムの間の要求が変換されます。</span><span class="sxs-lookup"><span data-stu-id="56d7d-105">This layer translates requests between the modern application and the legacy system.</span></span> <span data-ttu-id="56d7d-106">このパターンを使用して、アプリケーションの設計が、レガシ システムへの依存関係で制限されないようにします。</span><span class="sxs-lookup"><span data-stu-id="56d7d-106">Use this pattern to ensure that an application's design is not limited by dependencies on legacy systems.</span></span> <span data-ttu-id="56d7d-107">このパターンは、Eric Evans が『*Domain-Driven Design*』で初めて説明しました。</span><span class="sxs-lookup"><span data-stu-id="56d7d-107">This pattern was first described by Eric Evans in *Domain-Driven Design*.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="56d7d-108">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="56d7d-108">Context and problem</span></span>

<span data-ttu-id="56d7d-109">アプリケーションのほとんどが、そのデータや機能の一部について、他のシステムに依存しています。</span><span class="sxs-lookup"><span data-stu-id="56d7d-109">Most applications rely on other systems for some data or functionality.</span></span> <span data-ttu-id="56d7d-110">たとえば、レガシ アプリケーションを最新のシステムに移行するとき、そのレガシ アプリケーションは、引き続き既存のレガシ リソースを必要とする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-110">For example, when a legacy application is migrated to a modern system, it may still need existing legacy resources.</span></span> <span data-ttu-id="56d7d-111">レガシ システムは、新しい機能で呼び出すことができなければなりません。</span><span class="sxs-lookup"><span data-stu-id="56d7d-111">New features must be able to call the legacy system.</span></span> <span data-ttu-id="56d7d-112">これは、特に、段階的な移行の場合に当てはまります。こうした移行では、大規模なアプリケーションのさまざまな機能が、時間をかけて最新システムに移動します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-112">This is especially true of gradual migrations, where different features of a larger application are moved to a modern system over time.</span></span>

<span data-ttu-id="56d7d-113">このようなレガシ システムには、多くの場合、データ スキーマが複雑、API が古いなど、品質に関する問題があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-113">Often these legacy systems suffer from quality issues such as convoluted data schemas or obsolete APIs.</span></span> <span data-ttu-id="56d7d-114">レガシ システムで使用されている機能とテクノロジは、最新のシステムとは大きく異なる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-114">The features and technologies used in legacy systems can vary widely from more modern systems.</span></span> <span data-ttu-id="56d7d-115">レガシ システムと相互運用するには、新しいアプリケーションには、最新アプリケーションに組み込まない古いインフラストラクチャ、プロトコル、データ モデル、API、またはその他の機能のサポートが必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-115">To interoperate with the legacy system, the new application may need to support outdated infrastructure, protocols, data models, APIs, or other features that you wouldn't otherwise put into a modern application.</span></span>

<span data-ttu-id="56d7d-116">新しいシステムとレガシ システムの間のアクセスを維持することによって、新しいシステムは、少なくともレガシ システムの API または他のセマンティクスの一部に強制的に従うことになります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-116">Maintaining access between new and legacy systems can force the new system to adhere to at least some of the legacy system's APIs or other semantics.</span></span> <span data-ttu-id="56d7d-117">こうしたレガシ機能の品質に問題がある場合、その機能をサポートすることで、正常に設計された最新のアプリケーションが "破損" する場合があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-117">When these legacy features have quality issues, supporting them "corrupts" what might otherwise be a cleanly designed modern application.</span></span> 

## <a name="solution"></a><span data-ttu-id="56d7d-118">解決策</span><span class="sxs-lookup"><span data-stu-id="56d7d-118">Solution</span></span>

<span data-ttu-id="56d7d-119">レガシ システムと最新システムの間に破損対策レイヤーを配置し、これらを切り離します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-119">Isolate the legacy and modern systems by placing an anti-corruption layer between them.</span></span> <span data-ttu-id="56d7d-120">このレイヤーは、2 つのシステム間の通信を変換するもので、レガシ システムはそのままで、最新のアプリケーションの設計と技術的アプローチも損なわれません。</span><span class="sxs-lookup"><span data-stu-id="56d7d-120">This layer translates communications between the two systems, allowing the legacy system to remain unchanged while the modern application can avoid compromising its design and technological approach.</span></span>

![](./_images/anti-corruption-layer.png) 

<span data-ttu-id="56d7d-121">最新のアプリケーションと破損対策レイヤーの間の通信では、常に、アプリケーションのデータ モデルとアーキテクチャが使用されます。</span><span class="sxs-lookup"><span data-stu-id="56d7d-121">Communication between the modern application and the anti-corruption layer always uses the application's data model and architecture.</span></span> <span data-ttu-id="56d7d-122">破損対策レイヤーからレガシ システムへの呼び出しは、そのシステムのデータ モデルまたはメソッドに準拠します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-122">Calls from the anti-corruption layer to the legacy system conform to that system's data model or methods.</span></span> <span data-ttu-id="56d7d-123">破損対策レイヤーには、2 つのシステム間の変換に必要なロジックがすべて含まれます。</span><span class="sxs-lookup"><span data-stu-id="56d7d-123">The anti-corruption layer contains all of the logic necessary to translate between the two systems.</span></span> <span data-ttu-id="56d7d-124">レイヤーは、アプリケーション内でコンポーネントとして実装できます。また、独立したサービスとして実装することもできます。</span><span class="sxs-lookup"><span data-stu-id="56d7d-124">The layer can be implemented as a component within the application or as an independent service.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="56d7d-125">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="56d7d-125">Issues and considerations</span></span>

- <span data-ttu-id="56d7d-126">破損対策レイヤーにより、2 つのシステム間で行われた呼び出しで、待ち時間が追加される場合があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-126">The anti-corruption layer may add latency to calls made between the two systems.</span></span>
- <span data-ttu-id="56d7d-127">破損対策レイヤーにより、管理および保守しなければならない追加サービスが増えます。</span><span class="sxs-lookup"><span data-stu-id="56d7d-127">The anti-corruption layer adds an additional service that must be managed and maintained.</span></span>
- <span data-ttu-id="56d7d-128">破損対策レイヤーをスケールする方法を検討してください。</span><span class="sxs-lookup"><span data-stu-id="56d7d-128">Consider how your anti-corruption layer will scale.</span></span>
- <span data-ttu-id="56d7d-129">複数の破損対策レイヤーが必要かどうかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="56d7d-129">Consider whether you need more than one anti-corruption layer.</span></span> <span data-ttu-id="56d7d-130">さまざまなテクノロジや言語を使用して、機能を複数のサービスに分解できます。破損対策レイヤーをパーティション分割する理由が他に存在する可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-130">You may want to decompose functionality into multiple services using different technologies or languages, or there may be other reasons to partition the anti-corruption layer.</span></span>
- <span data-ttu-id="56d7d-131">他のアプリケーションまたはサービスとの関係で、破損対策レイヤーがどのように管理されるかを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="56d7d-131">Consider how the anti-corruption layer will be managed in relation with your other applications or services.</span></span> <span data-ttu-id="56d7d-132">これは、監視、リリース、および構成プロセスにどのように統合されますか。</span><span class="sxs-lookup"><span data-stu-id="56d7d-132">How will it be integrated into your monitoring, release, and configuration processes?</span></span>
- <span data-ttu-id="56d7d-133">トランザクションおよびデータの整合性は保持され、監視できることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="56d7d-133">Make sure transaction and data consistency are maintained and can be monitored.</span></span>
- <span data-ttu-id="56d7d-134">破損対策レイヤーが処理する必要があるのが、レガシ システムと最新システムの間のすべての通信か、または機能のサブセットのみかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="56d7d-134">Consider whether the anti-corruption layer needs to handle all communication between legacy and modern systems, or just a subset of features.</span></span> 
- <span data-ttu-id="56d7d-135">破損対策レイヤーを永続的に使うか、いずれ、すべてのレガシ機能が移行された時点で使用をやめるのかを検討します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-135">Consider whether the anti-corruption layer is meant to be permanent, or eventually retired once all legacy functionality has been migrated.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="56d7d-136">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="56d7d-136">When to use this pattern</span></span>

<span data-ttu-id="56d7d-137">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="56d7d-137">Use this pattern when:</span></span>

- <span data-ttu-id="56d7d-138">複数の段階にわたって移行が発生するが、新しいシステムとレガシ システムの間の統合を保持する必要があるとき。</span><span class="sxs-lookup"><span data-stu-id="56d7d-138">A migration is planned to happen over multiple stages, but integration between new and legacy systems needs to be maintained.</span></span>
- <span data-ttu-id="56d7d-139">新しいシステムとレガシ システムでセマンティクスが異なるが、通信する必要があるとき。</span><span class="sxs-lookup"><span data-stu-id="56d7d-139">New and legacy system have different semantics, but still need to communicate.</span></span>

<span data-ttu-id="56d7d-140">新しいシステムとレガシ システムの間で大きなセマンティクスの違いがない場合、このパターンは適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="56d7d-140">This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.</span></span> 

## <a name="related-guidance"></a><span data-ttu-id="56d7d-141">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="56d7d-141">Related guidance</span></span>

- <span data-ttu-id="56d7d-142">[ストラングラー パターン][strangler]</span><span class="sxs-lookup"><span data-stu-id="56d7d-142">[Strangler pattern][strangler]</span></span>

[strangler]: ./strangler.md
