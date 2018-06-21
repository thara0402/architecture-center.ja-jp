---
title: メッセージングのパターン
description: クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。 非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8bf37903df3a6eb23f1581e0405358a7aee61f79
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848582"
---
# <a name="messaging-patterns"></a><span data-ttu-id="c9b40-105">メッセージングのパターン</span><span class="sxs-lookup"><span data-stu-id="c9b40-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="c9b40-106">クラウド アプリケーションの分散特性には、スケーラビリティを最大化するために、コンポーネントとサービスが (できれば疎結合的に) 接続されているメッセージング インフラストラクチャが必要です。</span><span class="sxs-lookup"><span data-stu-id="c9b40-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="c9b40-107">非同期メッセージングは広く使用されており、多くの利点がもたらされますが、メッセージの順序、有害メッセージの管理、べき等など、課題も多数あります。</span><span class="sxs-lookup"><span data-stu-id="c9b40-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>


|                            <span data-ttu-id="c9b40-108">パターン</span><span class="sxs-lookup"><span data-stu-id="c9b40-108">Pattern</span></span>                             |                                                                        <span data-ttu-id="c9b40-109">まとめ</span><span class="sxs-lookup"><span data-stu-id="c9b40-109">Summary</span></span>                                                                         |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|        [<span data-ttu-id="c9b40-110">競合コンシューマー</span><span class="sxs-lookup"><span data-stu-id="c9b40-110">Competing Consumers</span></span>](../competing-consumers.md)        |                            <span data-ttu-id="c9b40-111">複数の同時実行コンシューマーが、同じメッセージング チャネルで受信したメッセージを処理できるようにします。</span><span class="sxs-lookup"><span data-stu-id="c9b40-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span>                            |
|          [<span data-ttu-id="c9b40-112">パイプとフィルター</span><span class="sxs-lookup"><span data-stu-id="c9b40-112">Pipes and Filters</span></span>](../pipes-and-filters.md)          |                       <span data-ttu-id="c9b40-113">複雑な処理を実行するタスクを、再利用できる一連の独立した要素に分解します。</span><span class="sxs-lookup"><span data-stu-id="c9b40-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span>                        |
|             [<span data-ttu-id="c9b40-114">優先順位キュー</span><span class="sxs-lookup"><span data-stu-id="c9b40-114">Priority Queue</span></span>](../priority-queue.md)             | <span data-ttu-id="c9b40-115">サービスに送信される要求に優先順位を設定し、優先順位の高い要求から順番に受信および処理されるようにします。</span><span class="sxs-lookup"><span data-stu-id="c9b40-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
|  [<span data-ttu-id="c9b40-116">キュー ベースの負荷平準化</span><span class="sxs-lookup"><span data-stu-id="c9b40-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  |              <span data-ttu-id="c9b40-117">タスクとそのタスクが呼び出すサービスとの間でバッファーとして機能するキューを使用して、断続的な大きい負荷を平準化します。</span><span class="sxs-lookup"><span data-stu-id="c9b40-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>               |
| [<span data-ttu-id="c9b40-118">Scheduler エージェント スーパーバイザー</span><span class="sxs-lookup"><span data-stu-id="c9b40-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) |                              <span data-ttu-id="c9b40-119">分散された一連のサービスやその他のリモート リソースにわたる一連のアクションを調整します。</span><span class="sxs-lookup"><span data-stu-id="c9b40-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span>                              |

