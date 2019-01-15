---
title: 破損対策レイヤー パターン
titleSuffix: Cloud Design Patterns
description: 最新アプリケーションとレガシ システムの間にファサード、すなわちアダプター レイヤーを実装します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d1023140deea4a2714c762945935d0838136e508
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011278"
---
# <a name="anti-corruption-layer-pattern"></a><span data-ttu-id="6fe20-104">破損対策レイヤー パターン</span><span class="sxs-lookup"><span data-stu-id="6fe20-104">Anti-Corruption Layer pattern</span></span>

<span data-ttu-id="6fe20-105">同じセマンティクスを共有しない異なるサブシステム間にファサード、つまりアダプター レイヤーを実装します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-105">Implement a façade or adapter layer between different subsystems that don't share the same semantics.</span></span> <span data-ttu-id="6fe20-106">このレイヤーでは、一方のサブシステムがもう一方のサブシステムに行った要求が変換されます。</span><span class="sxs-lookup"><span data-stu-id="6fe20-106">This layer translates requests that one subsystem makes to the other subsystem.</span></span> <span data-ttu-id="6fe20-107">このパターンを使用することで、アプリケーションの設計が、外部サブシステムへの依存関係によって制限されないようにします。</span><span class="sxs-lookup"><span data-stu-id="6fe20-107">Use this pattern to ensure that an application's design is not limited by dependencies on outside subsystems.</span></span> <span data-ttu-id="6fe20-108">このパターンは、Eric Evans が『*Domain-Driven Design*』で初めて説明しました。</span><span class="sxs-lookup"><span data-stu-id="6fe20-108">This pattern was first described by Eric Evans in *Domain-Driven Design*.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6fe20-109">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="6fe20-109">Context and problem</span></span>

<span data-ttu-id="6fe20-110">アプリケーションのほとんどが、そのデータや機能の一部について、他のシステムに依存しています。</span><span class="sxs-lookup"><span data-stu-id="6fe20-110">Most applications rely on other systems for some data or functionality.</span></span> <span data-ttu-id="6fe20-111">たとえば、レガシ アプリケーションを最新のシステムに移行するとき、そのレガシ アプリケーションは、引き続き既存のレガシ リソースを必要とする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-111">For example, when a legacy application is migrated to a modern system, it may still need existing legacy resources.</span></span> <span data-ttu-id="6fe20-112">レガシ システムは、新しい機能で呼び出すことができなければなりません。</span><span class="sxs-lookup"><span data-stu-id="6fe20-112">New features must be able to call the legacy system.</span></span> <span data-ttu-id="6fe20-113">これは、特に、段階的な移行の場合に当てはまります。こうした移行では、大規模なアプリケーションのさまざまな機能が、時間をかけて最新システムに移動します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-113">This is especially true of gradual migrations, where different features of a larger application are moved to a modern system over time.</span></span>

<span data-ttu-id="6fe20-114">このようなレガシ システムには、多くの場合、データ スキーマが複雑、API が古いなど、品質に関する問題があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-114">Often these legacy systems suffer from quality issues such as convoluted data schemas or obsolete APIs.</span></span> <span data-ttu-id="6fe20-115">レガシ システムで使用されている機能とテクノロジは、最新のシステムとは大きく異なる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-115">The features and technologies used in legacy systems can vary widely from more modern systems.</span></span> <span data-ttu-id="6fe20-116">レガシ システムと相互運用するには、新しいアプリケーションには、最新アプリケーションに組み込まない古いインフラストラクチャ、プロトコル、データ モデル、API、またはその他の機能のサポートが必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-116">To interoperate with the legacy system, the new application may need to support outdated infrastructure, protocols, data models, APIs, or other features that you wouldn't otherwise put into a modern application.</span></span>

<span data-ttu-id="6fe20-117">新しいシステムとレガシ システムの間のアクセスを維持することによって、新しいシステムは、少なくともレガシ システムの API または他のセマンティクスの一部に強制的に従うことになります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-117">Maintaining access between new and legacy systems can force the new system to adhere to at least some of the legacy system's APIs or other semantics.</span></span> <span data-ttu-id="6fe20-118">こうしたレガシ機能の品質に問題がある場合、その機能をサポートすることで、正常に設計された最新のアプリケーションが "破損" する場合があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-118">When these legacy features have quality issues, supporting them "corrupts" what might otherwise be a cleanly designed modern application.</span></span>

<span data-ttu-id="6fe20-119">レガシ システムだけでなく、開発チームが制御できない外部システムでも似た問題が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-119">Similar issues can arise with any external system that your development team doesn't control, not just legacy systems.</span></span>

## <a name="solution"></a><span data-ttu-id="6fe20-120">解決策</span><span class="sxs-lookup"><span data-stu-id="6fe20-120">Solution</span></span>

<span data-ttu-id="6fe20-121">異なるサブシステムの間に破損対策レイヤーを配置し、これらを切り離します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-121">Isolate the different subsystems by placing an anti-corruption layer between them.</span></span> <span data-ttu-id="6fe20-122">このレイヤーは、2 つのシステム間の通信を変換するもので、一方のシステムはそのままで、もう一方のシステムの設計と技術的アプローチも損なわれません。</span><span class="sxs-lookup"><span data-stu-id="6fe20-122">This layer translates communications between the two systems, allowing one system to remain unchanged while the other can avoid compromising its design and technological approach.</span></span>

![破損対策レイヤー パターンの図](./_images/anti-corruption-layer.png)

<span data-ttu-id="6fe20-124">上記の図は、アプリケーションと 2 つのサブシステムを示しています。</span><span class="sxs-lookup"><span data-stu-id="6fe20-124">The diagram above shows an application with two subsystems.</span></span> <span data-ttu-id="6fe20-125">サブシステム A は、破損対策レイヤーを通じてサブシステム B を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-125">Subsystem A calls to subsystem B through an anti-corruption layer.</span></span> <span data-ttu-id="6fe20-126">サブシステム A と破損対策レイヤーの間の通信では、常に、サブシステム A のデータ モデルとアーキテクチャが使用されます。破損対策レイヤーからサブシステム B への呼び出しは、そのサブシステムのデータ モデルまたはメソッドに準拠します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-126">Communication between subsystem A and the anti-corruption layer always uses the data model and architecture of subsystem A. Calls from the anti-corruption layer to subsystem B conform to that subsystem's data model or methods.</span></span> <span data-ttu-id="6fe20-127">破損対策レイヤーには、2 つのシステム間の変換に必要なロジックがすべて含まれます。</span><span class="sxs-lookup"><span data-stu-id="6fe20-127">The anti-corruption layer contains all of the logic necessary to translate between the two systems.</span></span> <span data-ttu-id="6fe20-128">レイヤーは、アプリケーション内でコンポーネントとして実装できます。また、独立したサービスとして実装することもできます。</span><span class="sxs-lookup"><span data-stu-id="6fe20-128">The layer can be implemented as a component within the application or as an independent service.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="6fe20-129">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="6fe20-129">Issues and considerations</span></span>

- <span data-ttu-id="6fe20-130">破損対策レイヤーにより、2 つのシステム間で行われた呼び出しで、待ち時間が追加される場合があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-130">The anti-corruption layer may add latency to calls made between the two systems.</span></span>
- <span data-ttu-id="6fe20-131">破損対策レイヤーにより、管理および保守しなければならない追加サービスが増えます。</span><span class="sxs-lookup"><span data-stu-id="6fe20-131">The anti-corruption layer adds an additional service that must be managed and maintained.</span></span>
- <span data-ttu-id="6fe20-132">破損対策レイヤーをスケールする方法を検討してください。</span><span class="sxs-lookup"><span data-stu-id="6fe20-132">Consider how your anti-corruption layer will scale.</span></span>
- <span data-ttu-id="6fe20-133">複数の破損対策レイヤーが必要かどうかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="6fe20-133">Consider whether you need more than one anti-corruption layer.</span></span> <span data-ttu-id="6fe20-134">さまざまなテクノロジや言語を使用して、機能を複数のサービスに分解できます。破損対策レイヤーをパーティション分割する理由が他に存在する可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-134">You may want to decompose functionality into multiple services using different technologies or languages, or there may be other reasons to partition the anti-corruption layer.</span></span>
- <span data-ttu-id="6fe20-135">他のアプリケーションまたはサービスとの関係で、破損対策レイヤーがどのように管理されるかを考慮してください。</span><span class="sxs-lookup"><span data-stu-id="6fe20-135">Consider how the anti-corruption layer will be managed in relation with your other applications or services.</span></span> <span data-ttu-id="6fe20-136">これは、監視、リリース、および構成プロセスにどのように統合されますか。</span><span class="sxs-lookup"><span data-stu-id="6fe20-136">How will it be integrated into your monitoring, release, and configuration processes?</span></span>
- <span data-ttu-id="6fe20-137">トランザクションおよびデータの整合性は保持され、監視できることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="6fe20-137">Make sure transaction and data consistency are maintained and can be monitored.</span></span>
- <span data-ttu-id="6fe20-138">破損対策レイヤーが処理する必要があるのが、異なるサブシステム間のすべての通信か、または機能のサブセットのみかを検討してください。</span><span class="sxs-lookup"><span data-stu-id="6fe20-138">Consider whether the anti-corruption layer needs to handle all communication between different subsystems, or just a subset of features.</span></span>
- <span data-ttu-id="6fe20-139">破損対策レイヤーがアプリケーション移行戦略の一部である場合は、そのレイヤーを永続的に使うか、すべてのレガシ機能が移行された時点で使用をやめるのかを検討します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-139">If the anti-corruption layer is part of an application migration strategy, consider whether it will be permanent, or will be retired after all legacy functionality has been migrated.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6fe20-140">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="6fe20-140">When to use this pattern</span></span>

<span data-ttu-id="6fe20-141">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="6fe20-141">Use this pattern when:</span></span>

- <span data-ttu-id="6fe20-142">複数の段階にわたって移行が発生するが、新しいシステムとレガシ システムの間の統合を保持する必要があるとき。</span><span class="sxs-lookup"><span data-stu-id="6fe20-142">A migration is planned to happen over multiple stages, but integration between new and legacy systems needs to be maintained.</span></span>
- <span data-ttu-id="6fe20-143">2 つ以上のサブシステムでセマンティクスが異なるが、通信する必要があるとき。</span><span class="sxs-lookup"><span data-stu-id="6fe20-143">Two or more subsystems have different semantics, but still need to communicate.</span></span>

<span data-ttu-id="6fe20-144">新しいシステムとレガシ システムの間で大きなセマンティクスの違いがない場合、このパターンは適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="6fe20-144">This pattern may not be suitable if there are no significant semantic differences between new and legacy systems.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="6fe20-145">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="6fe20-145">Related guidance</span></span>

- [<span data-ttu-id="6fe20-146">ストラングラー パターン</span><span class="sxs-lookup"><span data-stu-id="6fe20-146">Strangler pattern</span></span>](./strangler.md)
