---
title: メッセージングのパターン
titleSuffix: Cloud Design Patterns
description: クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。 非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。
keywords: 設計パターン
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e2079d8ed33dd205c155348d455f6380fff660a3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249337"
---
# <a name="messaging-patterns"></a><span data-ttu-id="35dbb-105">メッセージングのパターン</span><span class="sxs-lookup"><span data-stu-id="35dbb-105">Messaging patterns</span></span>

<span data-ttu-id="35dbb-106">クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。</span><span class="sxs-lookup"><span data-stu-id="35dbb-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="35dbb-107">非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。</span><span class="sxs-lookup"><span data-stu-id="35dbb-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="35dbb-108">Pattern</span><span class="sxs-lookup"><span data-stu-id="35dbb-108">Pattern</span></span> | <span data-ttu-id="35dbb-109">まとめ</span><span class="sxs-lookup"><span data-stu-id="35dbb-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="35dbb-110">要求チェック</span><span class="sxs-lookup"><span data-stu-id="35dbb-110">Claim Check</span></span>](../claim-check.md) | <span data-ttu-id="35dbb-111">大きいメッセージを要求チェックとペイロードに分割して、メッセージ バスに過度な負荷がかかることを防ぎます。</span><span class="sxs-lookup"><span data-stu-id="35dbb-111">Split a large message into a claim check and a payload to avoid overwhelming a message bus.</span></span> |
| [<span data-ttu-id="35dbb-112">競合コンシューマー</span><span class="sxs-lookup"><span data-stu-id="35dbb-112">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="35dbb-113">複数の同時実行コンシューマーが、同じメッセージング チャネルで受信したメッセージを処理できるようにします。</span><span class="sxs-lookup"><span data-stu-id="35dbb-113">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="35dbb-114">パイプとフィルター</span><span class="sxs-lookup"><span data-stu-id="35dbb-114">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="35dbb-115">複雑な処理を実行するタスクを、再利用できる一連の独立した要素に分解します。</span><span class="sxs-lookup"><span data-stu-id="35dbb-115">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="35dbb-116">優先順位キュー</span><span class="sxs-lookup"><span data-stu-id="35dbb-116">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="35dbb-117">サービスに送信される要求に優先順位を設定し、優先順位の高い要求から順番に受信および処理されるようにします。</span><span class="sxs-lookup"><span data-stu-id="35dbb-117">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="35dbb-118">パブリッシャーとサブスクライバー</span><span class="sxs-lookup"><span data-stu-id="35dbb-118">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="35dbb-119">送信側と受信側を結合せずに、アプリケーションから複数の対象コンシューマーに対して非同期的にイベントを通知できるようにします。</span><span class="sxs-lookup"><span data-stu-id="35dbb-119">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="35dbb-120">キュー ベースの負荷平準化</span><span class="sxs-lookup"><span data-stu-id="35dbb-120">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="35dbb-121">タスクとそのタスクが呼び出すサービスとの間でバッファーとして機能するキューを使用して、断続的な大きい負荷を平準化します。</span><span class="sxs-lookup"><span data-stu-id="35dbb-121">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="35dbb-122">Scheduler エージェント スーパーバイザー</span><span class="sxs-lookup"><span data-stu-id="35dbb-122">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="35dbb-123">分散された一連のサービスやその他のリモート リソースにわたる一連のアクションを調整します。</span><span class="sxs-lookup"><span data-stu-id="35dbb-123">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
