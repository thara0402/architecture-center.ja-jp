---
title: Azure Kubernetes Service のマイクロサービス リファレンス実装
description: このリファレンス実装では、マイクロサービス アーキテクチャのベスト プラクティスについて説明します
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 15e9aa16c0e2cfccecbfb84d217c275cc99a66fd
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58344412"
---
# <a name="designing-a-microservices-architecture"></a><span data-ttu-id="574c7-103">マイクロサービス アーキテクチャの設計</span><span class="sxs-lookup"><span data-stu-id="574c7-103">Designing a microservices architecture</span></span>

<span data-ttu-id="574c7-104">現在、マイクロサービスは、回復性優れ、単独でのデプロイが可能で、迅速に展開できるスケーラブルなクラウド アプリケーションを構築するための一般的なアーキテクチャ スタイルになっています。</span><span class="sxs-lookup"><span data-stu-id="574c7-104">Microservices have become a popular architectural style for building cloud applications that are resilient, highly scalable, independently deployable, and able to evolve quickly.</span></span> <span data-ttu-id="574c7-105">しかし、このマイクロサービスを、単なる業界用語に留めないためには、アプリケーションを設計および構築するのためのさまざまなアプローチが必要です。</span><span class="sxs-lookup"><span data-stu-id="574c7-105">To be more than just a buzzword, however, microservices require a different approach to designing and building applications.</span></span>

<span data-ttu-id="574c7-106">この一連の記事では、Azure でマイクロサービス アーキテクチャを構築して実行する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="574c7-106">In this set of articles, we explore how to build and run a microservices architecture on Azure.</span></span> <span data-ttu-id="574c7-107">取り上げるトピックは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="574c7-107">Topics include:</span></span>

- [<span data-ttu-id="574c7-108">サービス間の通信</span><span class="sxs-lookup"><span data-stu-id="574c7-108">Interservice communication</span></span>](./interservice-communication.md)
- [<span data-ttu-id="574c7-109">API 設計</span><span class="sxs-lookup"><span data-stu-id="574c7-109">API design</span></span>](./api-design.md)
- [<span data-ttu-id="574c7-110">API ゲートウェイ</span><span class="sxs-lookup"><span data-stu-id="574c7-110">API gateways</span></span>](./gateway.md)
- [<span data-ttu-id="574c7-111">データに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="574c7-111">Data considerations</span></span>](./data-considerations.md)
- [<span data-ttu-id="574c7-112">設計パターン</span><span class="sxs-lookup"><span data-stu-id="574c7-112">Design patterns</span></span>](./patterns.md)

## <a name="prerequisites"></a><span data-ttu-id="574c7-113">前提条件</span><span class="sxs-lookup"><span data-stu-id="574c7-113">Prerequisites</span></span>

<span data-ttu-id="574c7-114">これらの記事を読む前に、まず以下をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="574c7-114">Before reading these articles, you might start with the following:</span></span>

- <span data-ttu-id="574c7-115">[マイクロサービス アーキテクチャの概要](../introduction.md)。</span><span class="sxs-lookup"><span data-stu-id="574c7-115">[Introduction to microservices architectures](../introduction.md).</span></span> <span data-ttu-id="574c7-116">マイクロサービスのメリットと課題のほか、このスタイルのアーキテクチャを使用する場面について確認します。</span><span class="sxs-lookup"><span data-stu-id="574c7-116">Understand the benefits and challenges of microservices, and when to use this style of architecture.</span></span>
- <span data-ttu-id="574c7-117">[ドメイン分析を使用したマイクロサービスのモデル化](../model/domain-analysis.md)。</span><span class="sxs-lookup"><span data-stu-id="574c7-117">[Using domain analysis to model microservices](../model/domain-analysis.md).</span></span> <span data-ttu-id="574c7-118">マイクロサービスをモデル化するためのドメイン駆動のアプローチについて学習します。</span><span class="sxs-lookup"><span data-stu-id="574c7-118">Learn a domain-driven approach to modeling microservices.</span></span>

## <a name="reference-implementation"></a><span data-ttu-id="574c7-119">リファレンス実装</span><span class="sxs-lookup"><span data-stu-id="574c7-119">Reference implementation</span></span>

<span data-ttu-id="574c7-120">マイクロサービス アーキテクチャのベスト プラクティスを説明するために、ドローン配送アプリケーションというリファレンス実装を作成しました。</span><span class="sxs-lookup"><span data-stu-id="574c7-120">To illustrate best practices for a microservices architecture, we created a reference implementation that we call the Drone Delivery application.</span></span> <span data-ttu-id="574c7-121">この実装は、Azure Kubernetes Service (AKS) を使用して Kubernetes 上で動作します。</span><span class="sxs-lookup"><span data-stu-id="574c7-121">This implementation runs on Kubernetes using Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="574c7-122">リファレンス実装は [GitHub][drone-ri] にあります。</span><span class="sxs-lookup"><span data-stu-id="574c7-122">You can find the reference implementation on [GitHub][drone-ri].</span></span>

![ドローン配送アプリケーションのアーキテクチャ](../images/drone-delivery.png)

## <a name="scenario"></a><span data-ttu-id="574c7-124">シナリオ</span><span class="sxs-lookup"><span data-stu-id="574c7-124">Scenario</span></span>

<span data-ttu-id="574c7-125">Fabrikam, Inc. は、ドローン配送サービスを開始しようとしています。</span><span class="sxs-lookup"><span data-stu-id="574c7-125">Fabrikam, Inc. is starting a drone delivery service.</span></span> <span data-ttu-id="574c7-126">同社は、ドローン機団を管理しています。</span><span class="sxs-lookup"><span data-stu-id="574c7-126">The company manages a fleet of drone aircraft.</span></span> <span data-ttu-id="574c7-127">企業がサービスに登録すると、ユーザーは、ドローンで商品を集荷して配送するように依頼できます。</span><span class="sxs-lookup"><span data-stu-id="574c7-127">Businesses register with the service, and users can request a drone to pick up goods for delivery.</span></span> <span data-ttu-id="574c7-128">顧客が集荷のスケジュールを設定すると、バックエンド システムによってドローンが割り当てられ、推定配送時刻がユーザーに通知されます。</span><span class="sxs-lookup"><span data-stu-id="574c7-128">When a customer schedules a pickup, a backend system assigns a drone and notifies the user with an estimated delivery time.</span></span> <span data-ttu-id="574c7-129">配送中、ETA は常時更新され、顧客はドローンの場所を追跡できます。</span><span class="sxs-lookup"><span data-stu-id="574c7-129">While the delivery is in progress, the customer can track the location of the drone, with a continuously updated ETA.</span></span>

<span data-ttu-id="574c7-130">このシナリオには、かなり複雑なドメインが含まれます。</span><span class="sxs-lookup"><span data-stu-id="574c7-130">This scenario involves a fairly complicated domain.</span></span> <span data-ttu-id="574c7-131">ビジネスに関する懸念事項には、ドローンのスケジュール設定、荷物の追跡、ユーザー アカウントの管理、履歴データの保存と分析などもあります。</span><span class="sxs-lookup"><span data-stu-id="574c7-131">Some of the business concerns include scheduling drones, tracking packages, managing user accounts, and storing and analyzing historical data.</span></span> <span data-ttu-id="574c7-132">さらに、Fabrikam が求めているのは、迅速に市場に出し、迅速に反復して、新機能を追加することです。</span><span class="sxs-lookup"><span data-stu-id="574c7-132">Moreover, Fabrikam wants to get to market quickly and then iterate quickly, adding new functionality and capabilities.</span></span> <span data-ttu-id="574c7-133">アプリケーションは、高いサービス レベル目標 (SLO) に基づいてクラウド スケールで動作する必要があります。</span><span class="sxs-lookup"><span data-stu-id="574c7-133">The application needs to operate at cloud scale, with a high service level objective (SLO).</span></span> <span data-ttu-id="574c7-134">また、Fabrikam は、システムの部分ごとに、データ ストレージとクエリの要件がまったく異なることを予期しています。</span><span class="sxs-lookup"><span data-stu-id="574c7-134">Fabrikam also expects that different parts of the system will have very different requirements for data storage and querying.</span></span> <span data-ttu-id="574c7-135">Fabrikam はこれらをすべて考慮した結果、ドローン配送アプリケーションにマイクロサービス アーキテクチャを選択します。</span><span class="sxs-lookup"><span data-stu-id="574c7-135">All of these considerations lead Fabrikam to choose a microservices architecture for the Drone Delivery application.</span></span>

> [!NOTE]
> <span data-ttu-id="574c7-136">マイクロサービス アーキテクチャと他のアーキテクチャ スタイルのどちらを選択するかについては、「[Azure アプリケーション アーキテクチャ ガイド](../../guide/index.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="574c7-136">For help in choosing between a microservices architecture and other architectural styles, see the [Azure Application Architecture Guide](../../guide/index.md).</span></span>

<span data-ttu-id="574c7-137">このリファレンス実装では、[Azure Kubernetes Service (AKS)](/azure/aks/) で Kubernetes を使用しています。</span><span class="sxs-lookup"><span data-stu-id="574c7-137">Our reference implementation uses Kubernetes with [Azure Kubernetes Service](/azure/aks/) (AKS).</span></span> <span data-ttu-id="574c7-138">ただし、高いレベルでのアーキテクチャの決定と課題の多くは、[Azure Service Fabric](/azure/service-fabric/) を含む、すべてのコンテナー オーケストレーターに当てはまります。</span><span class="sxs-lookup"><span data-stu-id="574c7-138">However, many of the high-level architectural decisions and challenges will apply to any container orchestrator, including [Azure Service Fabric](/azure/service-fabric/).</span></span>

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation

## <a name="next-steps"></a><span data-ttu-id="574c7-139">次の手順</span><span class="sxs-lookup"><span data-stu-id="574c7-139">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="574c7-140">コンピューティング オプションの選択</span><span class="sxs-lookup"><span data-stu-id="574c7-140">Choose a compute option</span></span>](./compute-options.md)