---
title: フロント エンド用バックエンドのパターン
description: 特定のフロント エンド アプリケーションやインターフェイスによって使用される個別のバックエンド サービスを作成します。
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: a0dbc9ab58aa218f6faf40b70dad1bdc22d71458
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428790"
---
# <a name="backends-for-frontends-pattern"></a><span data-ttu-id="c656a-103">フロント エンド用バックエンドのパターン</span><span class="sxs-lookup"><span data-stu-id="c656a-103">Backends for Frontends pattern</span></span>

<span data-ttu-id="c656a-104">特定のフロント エンド アプリケーションやインターフェイスによって使用される個別のバックエンド サービスを作成します。</span><span class="sxs-lookup"><span data-stu-id="c656a-104">Create separate backend services to be consumed by specific frontend applications or interfaces.</span></span> <span data-ttu-id="c656a-105">このパターンは、複数のインターフェイスのために 1 つのバックエンドをカスタマイズすることが非効率な場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="c656a-105">This pattern is useful when you want to avoid customizing a single backend for multiple interfaces.</span></span> <span data-ttu-id="c656a-106">このパターンは、Sam Newman が初めて説明しました。</span><span class="sxs-lookup"><span data-stu-id="c656a-106">This pattern was first described by Sam Newman.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c656a-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="c656a-107">Context and problem</span></span>

<span data-ttu-id="c656a-108">アプリケーションは、当初デスクトップの Web UI 用に導入される場合があります。</span><span class="sxs-lookup"><span data-stu-id="c656a-108">An application may initially be targeted at a desktop web UI.</span></span> <span data-ttu-id="c656a-109">通常、バックエンド サービスは、その UI に必要な機能を提供するために、並行して開発されます。</span><span class="sxs-lookup"><span data-stu-id="c656a-109">Typically, a backend service is developed in parallel that provides the features needed for that UI.</span></span> <span data-ttu-id="c656a-110">アプリケーションのユーザー ベースが増えてくると、同じバックエンドとやりとりする、モバイル アプリケーションが開発されます。</span><span class="sxs-lookup"><span data-stu-id="c656a-110">As the application's user base grows, a mobile application is developed that must interact with the same backend.</span></span> <span data-ttu-id="c656a-111">その結果、バックエンド サービスは、デスクトップとモバイルの両方のインターフェイスの要件に対応する、汎用的なバックエンドになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-111">The backend service becomes a general-purpose backend, serving the requirements of both the desktop and mobile interfaces.</span></span>

<span data-ttu-id="c656a-112">しかし、モバイル デバイスの機能は、デスクトップ ブラウザーとは大きく異なります (画面サイズ、パフォーマンス、ディスプレイ制限など)。</span><span class="sxs-lookup"><span data-stu-id="c656a-112">But the capabilities of a mobile device differ significantly from a desktop browser, in terms of screen size, performance, and display limitations.</span></span> <span data-ttu-id="c656a-113">結果として、モバイル アプリケーションのバックエンド要件も、デスクトップ Web UI のものとは異なります。</span><span class="sxs-lookup"><span data-stu-id="c656a-113">As a result, the requirements for a mobile application backend differ from the desktop web UI.</span></span> 

<span data-ttu-id="c656a-114">これらの違いにより、バックエンドの要件に競合が発生するようになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-114">These differences result in competing requirements for the backend.</span></span> <span data-ttu-id="c656a-115">デスクトップ Web UI とモバイル アプリケーションの両方に対応するには、バックエンドを定期的に、かつ大幅に変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="c656a-115">The backend requires regular and significant changes to serve both the desktop web UI and the mobile application.</span></span> <span data-ttu-id="c656a-116">多くの場合、各フロント エンドでは個別のインターフェイス チームが作業するため、バックエンドは開発プロセスのボトルネックになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-116">Often, separate interface teams work on each frontend, causing the backend to become a bottleneck in the development process.</span></span> <span data-ttu-id="c656a-117">更新要件が競合している場合、両方のフロント エンドに対するサービス提供を維持しようとすると、1 つのリソースをデプロイするのにも、多くの労力が費やされることになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-117">Conflicting update requirements, and the need to keep the service working for both frontends, can result in spending a lot of effort on a single deployable resource.</span></span>

![](./_images/backend-for-frontend.png) 

<span data-ttu-id="c656a-118">開発アクティビティではバックエンド サービスに重点が置かれるため、バックエンドの管理と保守に個別のチームが配備されることがあります。</span><span class="sxs-lookup"><span data-stu-id="c656a-118">As the development activity focuses on the backend service, a separate team may be created to manage and maintain the backend.</span></span> <span data-ttu-id="c656a-119">このことは最終的に、インターフェイスとバックエンドの開発チーム間の分断につながり、各 UI チームのさまざまな要件を調整する必要が生じて、バックエンド チームに負担がかかることになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-119">Ultimately, this results in a disconnect between the interface and backend development teams, placing a burden on the backend team to balance the competing requirements of the different UI teams.</span></span> <span data-ttu-id="c656a-120">あるインターフェイス チームでバックエンドを変更する必要が生じても、それらの変更を他のインターフェイス チームで検証してからでないと、変更をバックエンドに統合することはできません。</span><span class="sxs-lookup"><span data-stu-id="c656a-120">When one interface team requires changes to the backend, those changes must be validated with other interface teams before they can be integrated into the backend.</span></span> 

## <a name="solution"></a><span data-ttu-id="c656a-121">解決策</span><span class="sxs-lookup"><span data-stu-id="c656a-121">Solution</span></span>

<span data-ttu-id="c656a-122">ユーザー インターフェイスごとに 1 つのバックエンドを作成します。</span><span class="sxs-lookup"><span data-stu-id="c656a-122">Create one backend per user interface.</span></span> <span data-ttu-id="c656a-123">そうすれば、フロントエンド環境のニーズに応じて各バックエンドの動作やパフォーマンスを調整しても、他のフロントエンドの動作に影響する心配はなくなります。</span><span class="sxs-lookup"><span data-stu-id="c656a-123">Fine tune the behavior and performance of each backend to best match the needs of the frontend environment, without worrying about affecting other frontend experiences.</span></span>

![](./_images/backend-for-frontend-example.png) 

<span data-ttu-id="c656a-124">各バックエンドが 1 つのインターフェイスに対応するので、バックエンドをそのインターフェイス用に最適化できます。</span><span class="sxs-lookup"><span data-stu-id="c656a-124">Because each backend is specific to one interface, it can be optimized for that interface.</span></span> <span data-ttu-id="c656a-125">その結果、すべてのインターフェイスの要件を満たそうとする汎用バックエンドよりも、サイズが小さくなり、複雑さが軽減され、多くの場合は処理速度も速くなります。</span><span class="sxs-lookup"><span data-stu-id="c656a-125">As a result, it will be smaller, less complex, and likely faster than a generic backend that tries to satisfy the requirements for all interfaces.</span></span> <span data-ttu-id="c656a-126">各インターフェイス チームは、専用のバックエンドを自律的に制御できるようになり、中央のバックエンド開発チームに依存しなくて済むようになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-126">Each interface team has autonomy to control their own backend and doesn't rely on a centralized backend development team.</span></span> <span data-ttu-id="c656a-127">その結果インターフェイス チームは、言語の選択、リリース間隔、ワークロードの優先度、およびバックエンドでの機能統合を、柔軟に決められるようになります。</span><span class="sxs-lookup"><span data-stu-id="c656a-127">This gives the interface team flexibility in language selection, release cadence, prioritization of workload, and feature integration in their backend.</span></span>

<span data-ttu-id="c656a-128">詳細については、「[Pattern: Backends For Frontends (パターン: フロントエンド用バックエンド)](https://samnewman.io/patterns/architectural/bff/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="c656a-128">For more information, see [Pattern: Backends For Frontends](https://samnewman.io/patterns/architectural/bff/).</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c656a-129">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="c656a-129">Issues and considerations</span></span>

- <span data-ttu-id="c656a-130">デプロイするバックエンドの数を検討してください。</span><span class="sxs-lookup"><span data-stu-id="c656a-130">Consider how many backends to deploy.</span></span>
- <span data-ttu-id="c656a-131">複数の異なるインターフェイス (モバイル クライアントなど) が同じ要求をする場合は、インターフェイスごとにバックエンドを実装する必要があるか、それとも 1 つのバックエンドで十分かを検討してください。</span><span class="sxs-lookup"><span data-stu-id="c656a-131">If different interfaces (such as mobile clients) will make the same requests, consider whether it is necessary to implement a backend for each interface, or if a single backend will suffice.</span></span>
- <span data-ttu-id="c656a-132">このパターンを実装すると、サービス間でコードが重複する可能性が高くなります。</span><span class="sxs-lookup"><span data-stu-id="c656a-132">Code duplication across services is highly likely when implementing this pattern.</span></span>
- <span data-ttu-id="c656a-133">フロントエンドに特化したバックエンド サービスでは、クライアント固有のロジックと動作だけを実装するようにしてください。</span><span class="sxs-lookup"><span data-stu-id="c656a-133">Frontend-focused backend services should only contain client-specific logic and behavior.</span></span> <span data-ttu-id="c656a-134">汎用的なビジネス ロジックやその他のグローバル機能は、アプリケーション内の別の場所で管理するようにしてください。</span><span class="sxs-lookup"><span data-stu-id="c656a-134">General business logic and other global features should be managed elsewhere in your application.</span></span>
- <span data-ttu-id="c656a-135">このパターンが、開発チームの担当業務にどのように影響するかを考えてください。</span><span class="sxs-lookup"><span data-stu-id="c656a-135">Think about how this pattern might be reflected in the responsibilities of a development team.</span></span>
- <span data-ttu-id="c656a-136">このパターンを実装するのにかかる時間を検討してください。</span><span class="sxs-lookup"><span data-stu-id="c656a-136">Consider how long it will take to implement this pattern.</span></span> <span data-ttu-id="c656a-137">新しいバックエンドの構築によって技術的負債が発生し、既存の汎用バックエンドもサポートし続けなければならなくなりますか。</span><span class="sxs-lookup"><span data-stu-id="c656a-137">Will the effort of building the new backends incur technical debt, while you continue to support the existing generic backend?</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c656a-138">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="c656a-138">When to use this pattern</span></span>

<span data-ttu-id="c656a-139">このパターンは次の状況で使用します。</span><span class="sxs-lookup"><span data-stu-id="c656a-139">Use this pattern when:</span></span>

- <span data-ttu-id="c656a-140">共有または汎用目的のバックエンド サービスを保守するために、多大な開発オーバーヘッドが生じている。</span><span class="sxs-lookup"><span data-stu-id="c656a-140">A shared or general purpose backend service must be maintained with significant development overhead.</span></span>
- <span data-ttu-id="c656a-141">特定のクライアント インターフェイスの要件に合わせて、バックエンドを最適化する必要がある。</span><span class="sxs-lookup"><span data-stu-id="c656a-141">You want to optimize the backend for the requirements of specific client interfaces.</span></span>
- <span data-ttu-id="c656a-142">複数のインターフェイスに対応するために、汎用バックエンドのカスタマイズが行われている。</span><span class="sxs-lookup"><span data-stu-id="c656a-142">Customizations are made to a general-purpose backend to accommodate multiple interfaces.</span></span>
- <span data-ttu-id="c656a-143">他のユーザー インターフェイスのバックエンドには、別の言語を使用したほうが望ましい。</span><span class="sxs-lookup"><span data-stu-id="c656a-143">An alternative language is better suited for the backend of a different user interface.</span></span>

<span data-ttu-id="c656a-144">このパターンは、次の場合には適切でない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c656a-144">This pattern may not be suitable:</span></span>

- <span data-ttu-id="c656a-145">各インターフェイスが、バックエンドに対して同一または類似の要求をする。</span><span class="sxs-lookup"><span data-stu-id="c656a-145">When interfaces make the same or similar requests to the backend.</span></span>
- <span data-ttu-id="c656a-146">バックエンドとのやりとりに使用されされるインターフェイスが 1 つしかない。</span><span class="sxs-lookup"><span data-stu-id="c656a-146">When only one interface is used to interact with the backend.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="c656a-147">関連するガイダンス</span><span class="sxs-lookup"><span data-stu-id="c656a-147">Related guidance</span></span>

- <span data-ttu-id="c656a-148">[Gateway Aggregation pattern](./gateway-aggregation.md) (ゲートウェイ集約パターン)</span><span class="sxs-lookup"><span data-stu-id="c656a-148">[Gateway Aggregation pattern](./gateway-aggregation.md)</span></span>
- <span data-ttu-id="c656a-149">[Gateway Offloading pattern](./gateway-offloading.md) (ゲートウェイ オフロード パターン)</span><span class="sxs-lookup"><span data-stu-id="c656a-149">[Gateway Offloading pattern](./gateway-offloading.md)</span></span>
- <span data-ttu-id="c656a-150">[Gateway Routing pattern](./gateway-routing.md) (ゲートウェイ ルーティング パターン)</span><span class="sxs-lookup"><span data-stu-id="c656a-150">[Gateway Routing pattern](./gateway-routing.md)</span></span>


