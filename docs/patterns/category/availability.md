---
title: "可用性のパターン"
description: "可用性は、システムが動作し実際に使用できる時間の割合を定義します。 システム エラー、インフラストラクチャの問題、悪意ある攻撃、およびシステムの負荷によって影響を受けます。 通常は、稼働時間の割合として測定されます。 クラウド アプリケーションは通常はサービス レベル アグリーメント (SLA) をユーザーに提供します。つまり、アプリケーションの可用性が最大限になるように設計および実装する必要があります。"
keywords: "設計パターン"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a><span data-ttu-id="e4574-107">可用性のパターン</span><span class="sxs-lookup"><span data-stu-id="e4574-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="e4574-108">可用性は、システムが動作し実際に使用できる時間の割合を定義します。</span><span class="sxs-lookup"><span data-stu-id="e4574-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="e4574-109">システム エラー、インフラストラクチャの問題、悪意ある攻撃、およびシステムの負荷によって影響を受けます。</span><span class="sxs-lookup"><span data-stu-id="e4574-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="e4574-110">通常は、稼働時間の割合として測定されます。</span><span class="sxs-lookup"><span data-stu-id="e4574-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="e4574-111">クラウド アプリケーションは通常はサービス レベル アグリーメント (SLA) をユーザーに提供します。つまり、アプリケーションの可用性が最大限になるように設計および実装する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e4574-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="e4574-112">パターン</span><span class="sxs-lookup"><span data-stu-id="e4574-112">Pattern</span></span> | <span data-ttu-id="e4574-113">概要</span><span class="sxs-lookup"><span data-stu-id="e4574-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="e4574-114">Health Endpoint Monitoring</span><span class="sxs-lookup"><span data-stu-id="e4574-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="e4574-115">公開されたエンドポイントを通じて、外部ツールが定期的にアクセスできる機能チェックをアプリケーションに実装します。</span><span class="sxs-lookup"><span data-stu-id="e4574-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="e4574-116">キュー ベースの負荷平準化</span><span class="sxs-lookup"><span data-stu-id="e4574-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="e4574-117">タスクとそのタスクが呼び出すサービスとの間でバッファーとして機能するキューを使用して、断続的な大きい負荷を平準化します。</span><span class="sxs-lookup"><span data-stu-id="e4574-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="e4574-118">調整</span><span class="sxs-lookup"><span data-stu-id="e4574-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="e4574-119">アプリケーションのインスタンス、個々のテナント、またはサービス全体によって使用されるリソースの使用量を制御します。</span><span class="sxs-lookup"><span data-stu-id="e4574-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |