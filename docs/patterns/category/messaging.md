---
title: "メッセージングのパターン"
description: "クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。 非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a><span data-ttu-id="6af5d-105">メッセージングのパターン</span><span class="sxs-lookup"><span data-stu-id="6af5d-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="6af5d-106">クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。</span><span class="sxs-lookup"><span data-stu-id="6af5d-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="6af5d-107">非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。</span><span class="sxs-lookup"><span data-stu-id="6af5d-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="6af5d-108">パターン</span><span class="sxs-lookup"><span data-stu-id="6af5d-108">Pattern</span></span> | <span data-ttu-id="6af5d-109">概要</span><span class="sxs-lookup"><span data-stu-id="6af5d-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="6af5d-110">競合コンシューマー</span><span class="sxs-lookup"><span data-stu-id="6af5d-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="6af5d-111">複数の同時実行コンシューマーが、同じメッセージング チャネルで受信したメッセージを処理できるようにします。</span><span class="sxs-lookup"><span data-stu-id="6af5d-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="6af5d-112">パイプとフィルター</span><span class="sxs-lookup"><span data-stu-id="6af5d-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="6af5d-113">複雑な処理を実行するタスクを、再利用できる一連の独立した要素に分解します。</span><span class="sxs-lookup"><span data-stu-id="6af5d-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="6af5d-114">優先順位キュー</span><span class="sxs-lookup"><span data-stu-id="6af5d-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="6af5d-115">サービスに送信される要求に優先順位を設定し、優先順位の高い要求から順番に受信および処理されるようにします。</span><span class="sxs-lookup"><span data-stu-id="6af5d-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="6af5d-116">キュー ベースの負荷平準化</span><span class="sxs-lookup"><span data-stu-id="6af5d-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="6af5d-117">タスクとそのタスクが呼び出すサービスとの間でバッファーとして機能するキューを使用して、断続的な大きい負荷を平準化します。</span><span class="sxs-lookup"><span data-stu-id="6af5d-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="6af5d-118">Scheduler Agent Supervisor</span><span class="sxs-lookup"><span data-stu-id="6af5d-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="6af5d-119">分散されている一連のサービスやその他のリモート リソースに対して、一連のアクションを調整します。</span><span class="sxs-lookup"><span data-stu-id="6af5d-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |