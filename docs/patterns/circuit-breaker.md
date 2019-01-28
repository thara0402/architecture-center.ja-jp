---
title: サーキット ブレーカー パターン
titleSuffix: Cloud Design Patterns
description: リモート サービスまたはリソースとの接続時の修正に要する時間が一定しないエラーを処理します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7cc84b3c14ea277aa82643f3141f0693ec702a49
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485606"
---
# <a name="circuit-breaker-pattern"></a><span data-ttu-id="77a60-104">サーキット ブレーカー パターン</span><span class="sxs-lookup"><span data-stu-id="77a60-104">Circuit Breaker pattern</span></span>

<span data-ttu-id="77a60-105">リモート サービスまたはリソースとの接続時に、復旧に要する時間が一定しないエラーを処理します。</span><span class="sxs-lookup"><span data-stu-id="77a60-105">Handle faults that might take a variable amount of time to recover from, when connecting to a remote service or resource.</span></span> <span data-ttu-id="77a60-106">これにより、アプリケーションの安定性と回復性を向上させることができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-106">This can improve the stability and resiliency of an application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="77a60-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="77a60-107">Context and problem</span></span>

<span data-ttu-id="77a60-108">分散環境では、リモートのリソースやサービスへの呼び出しは、低速なネットワーク接続、タイムアウト、または、リソースが過剰にコミットされたり一時的に使用できなくなったりするといった一時的なエラーのために失敗する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-108">In a distributed environment, calls to remote resources and services can fail due to transient faults, such as slow network connections, timeouts, or the resources being overcommitted or temporarily unavailable.</span></span> <span data-ttu-id="77a60-109">このようなエラーは、通常は短時間で自動的に修正され、堅牢なクラウド アプリケーションでは、[再試行パターン](./retry.md)などの方法を使用して、これらのエラーを処理する準備が整っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-109">These faults typically correct themselves after a short period of time, and a robust cloud application should be prepared to handle them by using a strategy such as the [Retry pattern](./retry.md).</span></span>

<span data-ttu-id="77a60-110">ただし、エラーが予期しないイベントによるものであることや、修正にかなり時間がかかる可能性があることも考えられます。</span><span class="sxs-lookup"><span data-stu-id="77a60-110">However, there can also be situations where faults are due to unanticipated events, and that might take much longer to fix.</span></span> <span data-ttu-id="77a60-111">このようなエラーの重大度は、部分的な接続の損失からサービスの完全な障害に至るまで、さまざまな可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-111">These faults can range in severity from a partial loss of connectivity to the complete failure of a service.</span></span> <span data-ttu-id="77a60-112">このような状況では、成功する可能性の低い操作を、アプリケーションが連続して無意味に再試行することがあります。そうではなくアプリケーションは、操作が失敗したことをすぐに受け入れて、そのエラーを適切に処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-112">In these situations it might be pointless for an application to continually retry an operation that is unlikely to succeed, and instead the application should quickly accept that the operation has failed and handle this failure accordingly.</span></span>

<span data-ttu-id="77a60-113">さらに、サービスが非常にビジーな状態の場合は、システムの一部分のエラーが障害の連鎖につながる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-113">Additionally, if a service is very busy, failure in one part of the system might lead to cascading failures.</span></span> <span data-ttu-id="77a60-114">たとえば、サービスを呼び出す操作は、タイムアウトを実装し、サービスがこの期間内に応答できない場合はエラー メッセージを返信するよう構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-114">For example, an operation that invokes a service could be configured to implement a timeout, and reply with a failure message if the service fails to respond within this period.</span></span> <span data-ttu-id="77a60-115">ただし、この方法では、タイムアウト期間が過ぎるまで、同じ操作に対する多数の同時要求がブロックされる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-115">However, this strategy could cause many concurrent requests to the same operation to be blocked until the timeout period expires.</span></span> <span data-ttu-id="77a60-116">これらのブロックされた要求が、重要なシステム リソース (メモリ、しきい値、データベース接続など) をとどめてしまう可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-116">These blocked requests might hold critical system resources such as memory, threads, database connections, and so on.</span></span> <span data-ttu-id="77a60-117">その結果、これらのリソースが使い尽くされて、同じリソースを使用する必要がある、関連性のない可能性のあるシステムの他の部分の障害を引き起こす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-117">Consequently, these resources could become exhausted, causing failure of other possibly unrelated parts of the system that need to use the same resources.</span></span> <span data-ttu-id="77a60-118">このような場合は、操作がすぐに失敗し、成功する可能性がある場合はサービスの呼び出しを試みるだけのほうが望ましいでしょう。</span><span class="sxs-lookup"><span data-stu-id="77a60-118">In these situations, it would be preferable for the operation to fail immediately, and only attempt to invoke the service if it's likely to succeed.</span></span> <span data-ttu-id="77a60-119">短いタイムアウトを設定すると、この問題を解決しやすくなることもあります。ただし、最終的にサービスへの要求が成功するとしても、ほとんど常に操作が失敗するような短いタイムアウトにすべきではないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="77a60-119">Note that setting a shorter timeout might help to resolve this problem, but the timeout shouldn't be so short that the operation fails most of the time, even if the request to the service would eventually succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="77a60-120">解決策</span><span class="sxs-lookup"><span data-stu-id="77a60-120">Solution</span></span>

<span data-ttu-id="77a60-121">サーキット ブレーカー パターンは、Michael Nygard 氏がその著書である「[Release It!](https://pragprog.com/book/mnee/release-it)」によって広めたもので、アプリケーションが失敗する可能性のある操作を繰り返し試行するのを防ぐことができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-121">The Circuit Breaker pattern, popularized by Michael Nygard in his book, [Release It!](https://pragprog.com/book/mnee/release-it), can prevent an application from repeatedly trying to execute an operation that's likely to fail.</span></span> <span data-ttu-id="77a60-122">障害が長期間にわたると判断している間に、エラーが修正されたり CPU サイクルが浪費されたりするのを待つことなく、続行することができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-122">Allowing it to continue without waiting for the fault to be fixed or wasting CPU cycles while it determines that the fault is long lasting.</span></span> <span data-ttu-id="77a60-123">サーキット ブレーカー パターンでは、エラーが解決されたかどうかをアプリケーションで検出することもできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-123">The Circuit Breaker pattern also enables an application to detect whether the fault has been resolved.</span></span> <span data-ttu-id="77a60-124">問題が解決済みのように見える場合、アプリケーションは操作の呼び出しを試みることができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-124">If the problem appears to have been fixed, the application can try to invoke the operation.</span></span>

> <span data-ttu-id="77a60-125">サーキット ブレーカー パターンの目的は、再試行パターンとは異なります。</span><span class="sxs-lookup"><span data-stu-id="77a60-125">The purpose of the Circuit Breaker pattern is different than the Retry pattern.</span></span> <span data-ttu-id="77a60-126">再試行パターンでは、アプリケーションが成功を見込んで操作を再試行することができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-126">The Retry pattern enables an application to retry an operation in the expectation that it'll succeed.</span></span> <span data-ttu-id="77a60-127">サーキット ブレーカー パターンは、失敗する可能性がある操作をアプリケーションが実行しないようにします。</span><span class="sxs-lookup"><span data-stu-id="77a60-127">The Circuit Breaker pattern prevents an application from performing an operation that is likely to fail.</span></span> <span data-ttu-id="77a60-128">アプリケーションは、サーキット ブレーカーによって操作を呼び出す再試行パターンを使用することで、これら 2 つのパターンを組み合わせることができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-128">An application can combine these two patterns by using the Retry pattern to invoke an operation through a circuit breaker.</span></span> <span data-ttu-id="77a60-129">ただし、再試行ロジックは、サーキット ブレーカーによって返されるすべての例外から大きな影響を受け、エラーが一時的なものではないことが示されると、再試行回数を破棄します。</span><span class="sxs-lookup"><span data-stu-id="77a60-129">However, the retry logic should be sensitive to any exceptions returned by the circuit breaker and abandon retry attempts if the circuit breaker indicates that a fault is not transient.</span></span>

<span data-ttu-id="77a60-130">サーキット ブレーカーは、失敗する可能性のある操作のプロキシとして機能します。</span><span class="sxs-lookup"><span data-stu-id="77a60-130">A circuit breaker acts as a proxy for operations that might fail.</span></span> <span data-ttu-id="77a60-131">プロキシは、最近発生した障害の数を監視し、その情報を使用して、操作を続行できるか、すぐに例外を返すだけにするかを決定します。</span><span class="sxs-lookup"><span data-stu-id="77a60-131">The proxy should monitor the number of recent failures that have occurred, and use this information to decide whether to allow the operation to proceed, or simply return an exception immediately.</span></span>

<span data-ttu-id="77a60-132">プロキシは、電路遮断器の機能を模倣する、次の状態のステート マシンとして実装できます。</span><span class="sxs-lookup"><span data-stu-id="77a60-132">The proxy can be implemented as a state machine with the following states that mimic the functionality of an electrical circuit breaker:</span></span>

- <span data-ttu-id="77a60-133">**クローズド**:アプリケーションからの要求は、操作にルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="77a60-133">**Closed**: The request from the application is routed to the operation.</span></span> <span data-ttu-id="77a60-134">プロキシは最近のエラーの数のカウントを保持し、操作への呼び出しが成功しなかった場合、プロキシはこのカウントを増分します。</span><span class="sxs-lookup"><span data-stu-id="77a60-134">The proxy maintains a count of the number of recent failures, and if the call to the operation is unsuccessful the proxy increments this count.</span></span> <span data-ttu-id="77a60-135">最近のエラーの数が指定された期間内に指定されたしきい値を超えると、プロキシは**オープン**状態になります。</span><span class="sxs-lookup"><span data-stu-id="77a60-135">If the number of recent failures exceeds a specified threshold within a given time period, the proxy is placed into the **Open** state.</span></span> <span data-ttu-id="77a60-136">プロキシはこの時点でタイムアウト タイマーを開始し、このタイマーの期限が切れると、プロキシは**ハーフオープン**状態になります。</span><span class="sxs-lookup"><span data-stu-id="77a60-136">At this point the proxy starts a timeout timer, and when this timer expires the proxy is placed into the **Half-Open** state.</span></span>

    > <span data-ttu-id="77a60-137">タイムアウト タイマーの目的は、エラーの原因となった問題を解決するための時間をシステムに与えて、その後でアプリケーションが操作の実行を再度試みられるようにすることです。</span><span class="sxs-lookup"><span data-stu-id="77a60-137">The purpose of the timeout timer is to give the system time to fix the problem that caused the failure before allowing the application to try to perform the operation again.</span></span>

- <span data-ttu-id="77a60-138">**未解決**: アプリケーションからの要求はすぐに失敗し、アプリケーションに例外が返されます。</span><span class="sxs-lookup"><span data-stu-id="77a60-138">**Open**: The request from the application fails immediately and an exception is returned to the application.</span></span>

- <span data-ttu-id="77a60-139">**ハーフオープン**:アプリケーションからの限られた数の要求が、操作のパス スルーと呼び出しを許可されます。</span><span class="sxs-lookup"><span data-stu-id="77a60-139">**Half-Open**: A limited number of requests from the application are allowed to pass through and invoke the operation.</span></span> <span data-ttu-id="77a60-140">これらの要求が成功した場合、以前にエラーの原因となった障害は既に修正されている見なされ、サーキット ブレーカーは**クローズド**状態に切り替わります (エラー カウンターはリセットされます)。</span><span class="sxs-lookup"><span data-stu-id="77a60-140">If these requests are successful, it's assumed that the fault that was previously causing the failure has been fixed and the circuit breaker switches to the **Closed** state (the failure counter is reset).</span></span> <span data-ttu-id="77a60-141">どの要求が失敗しても、サーキット ブレーカーはエラーがまだ存在していると見なすため、**オープン**状態に戻り、障害から復旧するためのさらに長い時間をシステムに与えるために、タイムアウト タイマーを再起動します。</span><span class="sxs-lookup"><span data-stu-id="77a60-141">If any request fails, the circuit breaker assumes that the fault is still present so it reverts back to the **Open** state and restarts the timeout timer to give the system a further period of time to recover from the failure.</span></span>

    > <span data-ttu-id="77a60-142">**ハーフオープン**状態は、復旧中のサービスに突然大量の要求が送信されないようにするために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="77a60-142">The **Half-Open** state is useful to prevent a recovering service from suddenly being flooded with requests.</span></span> <span data-ttu-id="77a60-143">サービスの復旧中は、復旧が完了するまでは限られた量の要求に対応できますが、復旧の進行中は大量の作業によってサービスがタイムアウトになったり、再び失敗したりする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-143">As a service recovers, it might be able to support a limited volume of requests until the recovery is complete, but while recovery is in progress a flood of work can cause the service to time out or fail again.</span></span>

![サーキット ブレーカーの状態](./_images/circuit-breaker-diagram.png)

<span data-ttu-id="77a60-145">この図では、**クローズド**状態で使用されるエラー カウンターは時間ベースです。</span><span class="sxs-lookup"><span data-stu-id="77a60-145">In the figure, the failure counter used by the **Closed** state is time based.</span></span> <span data-ttu-id="77a60-146">これは定期的な間隔で自動的にリセットされます。</span><span class="sxs-lookup"><span data-stu-id="77a60-146">It's automatically reset at periodic intervals.</span></span> <span data-ttu-id="77a60-147">これにより、一時的なエラーが発生した場合に、サーキット ブレーカーが**オープン**状態に入らないようにすることができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-147">This helps to prevent the circuit breaker from entering the **Open** state if it experiences occasional failures.</span></span> <span data-ttu-id="77a60-148">サーキット ブレーカーを**オープン**状態にトリップするエラーのしきい値に達するのは、指定された期間の間に指定された数のエラーが発生したときにのみです。</span><span class="sxs-lookup"><span data-stu-id="77a60-148">The failure threshold that trips the circuit breaker into the **Open** state is only reached when a specified number of failures have occurred during a specified interval.</span></span> <span data-ttu-id="77a60-149">**ハーフオープン**状態で使用されるカウンターは、操作の呼び出しに成功した試行の数を記録します。</span><span class="sxs-lookup"><span data-stu-id="77a60-149">The counter used by the **Half-Open** state records the number of successful attempts to invoke the operation.</span></span> <span data-ttu-id="77a60-150">サーキット ブレーカーは、指定された数の連続した操作の呼び出しが成功した後、**クローズド**状態に戻ります。</span><span class="sxs-lookup"><span data-stu-id="77a60-150">The circuit breaker reverts to the **Closed** state after a specified number of consecutive operation invocations have been successful.</span></span> <span data-ttu-id="77a60-151">呼び出しが失敗すると、サーキット ブレーカーは**オープン**状態に入り、次に**ハーフオープン**状態に入ったときに成功カウンターがリセットされます。</span><span class="sxs-lookup"><span data-stu-id="77a60-151">If any invocation fails, the circuit breaker enters the **Open** state immediately and the success counter will be reset the next time it enters the **Half-Open** state.</span></span>

> <span data-ttu-id="77a60-152">システムの復旧は、障害が発生したコンポーネントを復元または再起動するか、ネットワーク接続を修復することで、外部的に処理される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-152">How the system recovers is handled externally, possibly by restoring or restarting a failed component or repairing a network connection.</span></span>

<span data-ttu-id="77a60-153">サーキット ブレーカー パターンにより、エラーからのシステムの復旧中の安定性が保たれ、パフォーマンスに対する影響が最小限に抑えられます。</span><span class="sxs-lookup"><span data-stu-id="77a60-153">The Circuit Breaker pattern provides stability while the system recovers from a failure and minimizes the impact on performance.</span></span> <span data-ttu-id="77a60-154">これにより、失敗する可能性がある操作の要求を、操作がタイムアウトになるまで待機したり返さないようにしたりするのではなく、すぐに拒否することによって、システムの応答時間を維持することができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-154">It can help to maintain the response time of the system by quickly rejecting a request for an operation that's likely to fail, rather than waiting for the operation to time out, or never return.</span></span> <span data-ttu-id="77a60-155">状態が変化するたびにサーキット ブレーカーでイベントが発生する場合は、この情報を使用して、サーキット ブレーカーによって保護されているシステムの一部の正常性を監視したり、サーキット ブレーカーが**オープン**状態にトリップするときに管理者に警告したりできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-155">If the circuit breaker raises an event each time it changes state, this information can be used to monitor the health of the part of the system protected by the circuit breaker, or to alert an administrator when a circuit breaker trips to the **Open** state.</span></span>

<span data-ttu-id="77a60-156">このパターンはカスタマイズ可能で、可能性のあるエラーの種類に応じて適用できます。</span><span class="sxs-lookup"><span data-stu-id="77a60-156">The pattern is customizable and can be adapted according to the type of the possible failure.</span></span> <span data-ttu-id="77a60-157">たとえば、サーキット ブレーカーにタイムアウト タイマーの増加を適用できます。</span><span class="sxs-lookup"><span data-stu-id="77a60-157">For example, you can apply an increasing timeout timer to a circuit breaker.</span></span> <span data-ttu-id="77a60-158">サーキット ブレーカーを最初の数秒間**オープン**状態にし、その後でエラーが解決されていない場合は、タイムアウトを数分に延ばすといったこともできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-158">You could place the circuit breaker in the **Open** state for a few seconds initially, and then if the failure hasn't been resolved increase the timeout to a few minutes, and so on.</span></span> <span data-ttu-id="77a60-159">場合によっては、エラーを返し例外を発生させる**オープン**状態ではなく、アプリケーションにとって意味がある既定値を返すほうが有益な場合もあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-159">In some cases, rather than the **Open** state returning failure and raising an exception, it could be useful to return a default value that is meaningful to the application.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="77a60-160">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="77a60-160">Issues and considerations</span></span>

<span data-ttu-id="77a60-161">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="77a60-161">You should consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="77a60-162">**例外処理**。</span><span class="sxs-lookup"><span data-stu-id="77a60-162">**Exception Handling**.</span></span> <span data-ttu-id="77a60-163">サーキット ブレーカーによって操作を呼び出すアプリケーションは、操作が使用不可になった場合に発生する例外を処理するよう準備する必要があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-163">An application invoking an operation through a circuit breaker must be prepared to handle the exceptions raised if the operation is unavailable.</span></span> <span data-ttu-id="77a60-164">例外の処理方法はアプリケーション固有になります。</span><span class="sxs-lookup"><span data-stu-id="77a60-164">The way exceptions are handled will be application specific.</span></span> <span data-ttu-id="77a60-165">たとえば、アプリケーションは、その機能を一時的に低下させたり、同じタスクの実行や同じデータの取得を試みる代替の操作を呼び出したり、ユーザーに例外を報告して後でもう一度やり直すよう求めたりすることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-165">For example, an application could temporarily degrade its functionality, invoke an alternative operation to try to perform the same task or obtain the same data, or report the exception to the user and ask them to try again later.</span></span>

<span data-ttu-id="77a60-166">**例外の種類**。</span><span class="sxs-lookup"><span data-stu-id="77a60-166">**Types of Exceptions**.</span></span> <span data-ttu-id="77a60-167">要求は多くの理由で失敗する可能性があり、エラーによっては他のエラーよりもより重大な種類のエラーを示すものもあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-167">A request might fail for many reasons, some of which might indicate a more severe type of failure than others.</span></span> <span data-ttu-id="77a60-168">たとえば、リモート サービスがクラッシュし、回復するのに数分かかるために要求が失敗することも、一時的に過負荷になっているサービスによるタイムアウトのために要求が失敗することもあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-168">For example, a request might fail because a remote service has crashed and will take several minutes to recover, or because of a timeout due to the service being temporarily overloaded.</span></span> <span data-ttu-id="77a60-169">サーキット ブレーカーは、発生する例外の種類を確認し、その例外の性質に応じて戦略を調整できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-169">A circuit breaker might be able to examine the types of exceptions that occur and adjust its strategy depending on the nature of these exceptions.</span></span> <span data-ttu-id="77a60-170">たとえば、完全に使用不可になっているサービスによるエラーの数と比べると、サーキット ブレーカーを**オープン**状態にトリップするにはより多くのタイムアウト例外が必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-170">For example, it might require a larger number of timeout exceptions to trip the circuit breaker to the **Open** state compared to the number of failures due to the service being completely unavailable.</span></span>

<span data-ttu-id="77a60-171">**ログの記録**。</span><span class="sxs-lookup"><span data-stu-id="77a60-171">**Logging**.</span></span> <span data-ttu-id="77a60-172">サーキット ブレーカーは、管理者が操作の正常性を監視できるよう、すべての失敗した要求 (および成功した可能性のある要求) のログを記録します。</span><span class="sxs-lookup"><span data-stu-id="77a60-172">A circuit breaker should log all failed requests (and possibly successful requests) to enable an administrator to monitor the health of the operation.</span></span>

<span data-ttu-id="77a60-173">**回復性**。</span><span class="sxs-lookup"><span data-stu-id="77a60-173">**Recoverability**.</span></span> <span data-ttu-id="77a60-174">サーキット ブレーカーは、保護する操作の、可能性の高い復旧パターンに一致するよう構成してください。</span><span class="sxs-lookup"><span data-stu-id="77a60-174">You should configure the circuit breaker to match the likely recovery pattern of the operation it's protecting.</span></span> <span data-ttu-id="77a60-175">たとえば、サーキット ブレーカーが長期間**オープン**状態のままの場合は、エラーの原因が解決されているとしても、例外が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-175">For example, if the circuit breaker remains in the **Open** state for a long period, it could raise exceptions even if the reason for the failure has been resolved.</span></span> <span data-ttu-id="77a60-176">同様に、**オープン**状態から**ハーフオープン**状態に切り替えるのが早すぎると、サーキット ブレーカーが変動し、アプリケーションの応答時間を短くすることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-176">Similarly, a circuit breaker could fluctuate and reduce the response times of applications if it switches from the **Open** state to the **Half-Open** state too quickly.</span></span>

<span data-ttu-id="77a60-177">**失敗した操作のテスト**。</span><span class="sxs-lookup"><span data-stu-id="77a60-177">**Testing Failed Operations**.</span></span> <span data-ttu-id="77a60-178">**オープン**状態では、タイマーを使用して**ハーフオープン**状態に切り替えるタイミングを決定する代わりに、サーキット ブレーカーでリモート サービスまたはリソースに定期的に ping を実行して、もう一度使用可能になるかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="77a60-178">In the **Open** state, rather than using a timer to determine when to switch to the **Half-Open** state, a circuit breaker can instead periodically ping the remote service or resource to determine whether it's become available again.</span></span> <span data-ttu-id="77a60-179">この ping は、以前に失敗した操作の呼び出しを試みる形で行われることも、「[正常性エンドポイント監視パターン](./health-endpoint-monitoring.md)」に記載されている、サービスの正常性のテスト専用のリモート サービスによって提供される、特殊な操作を使用することもあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-179">This ping could take the form of an attempt to invoke an operation that had previously failed, or it could use a special operation provided by the remote service specifically for testing the health of the service, as described by the [Health Endpoint Monitoring pattern](./health-endpoint-monitoring.md).</span></span>

<span data-ttu-id="77a60-180">**手動オーバーライド**。</span><span class="sxs-lookup"><span data-stu-id="77a60-180">**Manual Override**.</span></span> <span data-ttu-id="77a60-181">障害が発生する操作の復旧時間に極端な幅があるシステムでは、手動リセットのオプションを用意して、管理者がサーキット ブレーカーをクローズして失敗カウンターをリセットできるようにすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="77a60-181">In a system where the recovery time for a failing operation is extremely variable, it's beneficial to provide a manual reset option that enables an administrator to close a circuit breaker (and reset the failure counter).</span></span> <span data-ttu-id="77a60-182">同様に、サーキット ブレーカーで保護されている操作が一時的に使用できなくなる場合は、管理者が強制的にサーキット ブレーカーを**オープン**状態にし、タイムアウト タイマーを再起動することもできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-182">Similarly, an administrator could force a circuit breaker into the **Open** state (and restart the timeout timer) if the operation protected by the circuit breaker is temporarily unavailable.</span></span>

<span data-ttu-id="77a60-183">**コンカレンシー**。</span><span class="sxs-lookup"><span data-stu-id="77a60-183">**Concurrency**.</span></span> <span data-ttu-id="77a60-184">アプリケーションの多数の同時実行インスタンスが、同じサーキット ブレーカーにアクセスすることもできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-184">The same circuit breaker could be accessed by a large number of concurrent instances of an application.</span></span> <span data-ttu-id="77a60-185">この実装は、同時要求をブロックしたり、操作へのそれぞれの呼び出しに過剰なオーバーヘッドを加えたりしません。</span><span class="sxs-lookup"><span data-stu-id="77a60-185">The implementation shouldn't block concurrent requests or add excessive overhead to each call to an operation.</span></span>

<span data-ttu-id="77a60-186">**リソースの区別**。</span><span class="sxs-lookup"><span data-stu-id="77a60-186">**Resource Differentiation**.</span></span> <span data-ttu-id="77a60-187">複数の基になる独立したプロバイダーがある可能性がある場合、1 つの種類のリソースに単一のサーキット ブレーカーを使用するときは注意してください。</span><span class="sxs-lookup"><span data-stu-id="77a60-187">Be careful when using a single circuit breaker for one type of resource if there might be multiple underlying independent providers.</span></span> <span data-ttu-id="77a60-188">たとえば、複数のシャードが含まれているデータ ストアでは、あるシャードで一時的な問題が発生しているときに、別のシャードには完全にアクセス可能であることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-188">For example, in a data store that contains multiple shards, one shard might be fully accessible while another is experiencing a temporary issue.</span></span> <span data-ttu-id="77a60-189">このようなシナリオのエラー応答が統合されると、アプリケーションは、エラーの可能性が高いときでも一部のシャードへのアクセスを試みる一方、他のシャードへのアクセスは、成功する可能性があってもブロックすることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-189">If the error responses in these scenarios are merged, an application might try to access some shards even when failure is highly likely, while access to other shards might be blocked even though it's likely to succeed.</span></span>

<span data-ttu-id="77a60-190">**高速なサーキット ブレーク**。</span><span class="sxs-lookup"><span data-stu-id="77a60-190">**Accelerated Circuit Breaking**.</span></span> <span data-ttu-id="77a60-191">エラー応答には、サーキット ブレーカーをすぐにトリップさせ、最短時間トリップしたままのするのに十分な情報が含まれていることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-191">Sometimes a failure response can contain enough information for the circuit breaker to trip immediately and stay tripped for a minimum amount of time.</span></span> <span data-ttu-id="77a60-192">たとえば、過負荷になっている共有リソースからのエラー応答で、即時の再試行が推奨されず、代わりに、アプリケーションが数分後にもう一度試みることを示されることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-192">For example, the error response from a shared resource that's overloaded could indicate that an immediate retry isn't recommended and that the application should instead try again in a few minutes.</span></span>

> [!NOTE]
> <span data-ttu-id="77a60-193">サービスは、クライアントが調整中の場合は HTTP 429 (要求が多すぎます) を返し、サービスが現在使用可能でない場合は HTTP 503 (サービス利用不可) を返すことがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-193">A service can return HTTP 429 (Too Many Requests) if it is throttling the client, or HTTP 503 (Service Unavailable) if the service is not currently available.</span></span> <span data-ttu-id="77a60-194">応答には、予想される遅延時間などの追加情報を含めることができます。</span><span class="sxs-lookup"><span data-stu-id="77a60-194">The response can include additional information, such as the anticipated duration of the delay.</span></span>

<span data-ttu-id="77a60-195">**失敗した要求の再現**。</span><span class="sxs-lookup"><span data-stu-id="77a60-195">**Replaying Failed Requests**.</span></span> <span data-ttu-id="77a60-196">**オープン**状態では、単にすぐに失敗させるのではなく、サーキット ブレーカーでジャーナルへの各要求の詳細を記録して、リモート リソースやサービスが使用可能になったときにこれらの要求が再生されるように準備することもできます。</span><span class="sxs-lookup"><span data-stu-id="77a60-196">In the **Open** state, rather than simply failing quickly, a circuit breaker could also record the details of each request to a journal and arrange for these requests to be replayed when the remote resource or service becomes available.</span></span>

<span data-ttu-id="77a60-197">**外部サービスでの不適切なタイムアウト**。</span><span class="sxs-lookup"><span data-stu-id="77a60-197">**Inappropriate Timeouts on External Services**.</span></span> <span data-ttu-id="77a60-198">サーキット ブレーカーは、タイムアウト時間が長く構成されている外部のサービスで失敗する操作からは、アプリケーションを完全に保護できない場合があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-198">A circuit breaker might not be able to fully protect applications from operations that fail in external services that are configured with a lengthy timeout period.</span></span> <span data-ttu-id="77a60-199">タイムアウト時間が長すぎると、操作が失敗したことをサーキット ブレーカーが示す前に、サーキット ブレーカーを実行するスレッドが長期間ブロックされることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-199">If the timeout is too long, a thread running a circuit breaker might be blocked for an extended period before the circuit breaker indicates that the operation has failed.</span></span> <span data-ttu-id="77a60-200">この時点では、他の多くのアプリケーション インスタンスもサーキット ブレーカーによってサービスを呼び出そうとして、すべてが失敗するまで多数のスレッドを占有することがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-200">In this time, many other application instances might also try to invoke the service through the circuit breaker and tie up a significant number of threads before they all fail.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="77a60-201">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="77a60-201">When to use this pattern</span></span>

<span data-ttu-id="77a60-202">このパターンは次の目的で使用します。</span><span class="sxs-lookup"><span data-stu-id="77a60-202">Use this pattern:</span></span>

- <span data-ttu-id="77a60-203">この操作が失敗する可能性が高い場合に、アプリケーションがリモート サービスを呼び出そうとしたり、共有リソースにアクセスしようとしたりしないようにするため。</span><span class="sxs-lookup"><span data-stu-id="77a60-203">To prevent an application from trying to invoke a remote service or access a shared resource if this operation is highly likely to fail.</span></span>

<span data-ttu-id="77a60-204">このパターンは次の場合は推奨されません。</span><span class="sxs-lookup"><span data-stu-id="77a60-204">This pattern isn't recommended:</span></span>

- <span data-ttu-id="77a60-205">メモリ内データ構造など、アプリケーションのローカルのプライベート リソースへのアクセスを処理する場合。</span><span class="sxs-lookup"><span data-stu-id="77a60-205">For handling access to local private resources in an application, such as in-memory data structure.</span></span> <span data-ttu-id="77a60-206">この環境では、サーキット ブレーカーを使用すると、システムにオーバーヘッドが加わることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-206">In this environment, using a circuit breaker would add overhead to your system.</span></span>
- <span data-ttu-id="77a60-207">例外をアプリケーションのビジネス ロジックで処理する代わりとして。</span><span class="sxs-lookup"><span data-stu-id="77a60-207">As a substitute for handling exceptions in the business logic of your applications.</span></span>

## <a name="example"></a><span data-ttu-id="77a60-208">例</span><span class="sxs-lookup"><span data-stu-id="77a60-208">Example</span></span>

<span data-ttu-id="77a60-209">Web アプリケーションでは、一部のページに外部サービスから取得されたデータが設定されます。</span><span class="sxs-lookup"><span data-stu-id="77a60-209">In a web application, several of the pages are populated with data retrieved from an external service.</span></span> <span data-ttu-id="77a60-210">システムが実装しているキャッシュが最小限の場合、これらのページへのほとんどのヒットで、サービスへのラウンド トリップが発生します。</span><span class="sxs-lookup"><span data-stu-id="77a60-210">If the system implements minimal caching, most hits to these pages will cause a round trip to the service.</span></span> <span data-ttu-id="77a60-211">Web アプリケーションからサービスへの接続は、タイムアウト期間 (通常 60 秒) で構成され、サービスが時間内に応答しない場合は、各 web ページ内のロジックがサービスを使用不可と見なし、例外をスローします。</span><span class="sxs-lookup"><span data-stu-id="77a60-211">Connections from the web application to the service could be configured with a timeout period (typically 60 seconds), and if the service doesn't respond in this time the logic in each web page will assume that the service is unavailable and throw an exception.</span></span>

<span data-ttu-id="77a60-212">ただし、サービスが失敗し、システムが非常にビジー状態な場合は、例外が発生するまで最大 60 秒間、ユーザーを強制的に待機させます。</span><span class="sxs-lookup"><span data-stu-id="77a60-212">However, if the service fails and the system is very busy, users could be forced to wait for up to 60 seconds before an exception occurs.</span></span> <span data-ttu-id="77a60-213">最終的にはメモリ、接続、スレッドなどのリソースが使い果たされ、他のユーザーは、サービスからデータを取得するページにアクセスしていない場合でもシステムに接続できなくなります。</span><span class="sxs-lookup"><span data-stu-id="77a60-213">Eventually resources such as memory, connections, and threads could be exhausted, preventing other users from connecting to the system, even if they aren't accessing pages that retrieve data from the service.</span></span>

<span data-ttu-id="77a60-214">Web サーバーをさらに追加して負荷分散を実装することでシステムをスケーリングすると、リソースが使い果たされるのは遅れますが、ユーザーの要求は応答しないままになり、Web サーバーは最終的にリソース不足になるため、問題は解決しません。</span><span class="sxs-lookup"><span data-stu-id="77a60-214">Scaling the system by adding further web servers and implementing load balancing might delay when resources become exhausted, but it won't resolve the issue because user requests will still be unresponsive and all web servers could still eventually run out of resources.</span></span>

<span data-ttu-id="77a60-215">サービスに接続し、サーキット ブレーカーでデータを取得するロジックをラップすれば、この問題が解決して、サービスのエラーをより効率よく処理できる場合があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-215">Wrapping the logic that connects to the service and retrieves the data in a circuit breaker could help to solve this problem and handle the service failure more elegantly.</span></span> <span data-ttu-id="77a60-216">それでもユーザーの要求は失敗しますが、より速く失敗し、リソースはブロックされません。</span><span class="sxs-lookup"><span data-stu-id="77a60-216">User requests will still fail, but they'll fail more quickly and the resources won't be blocked.</span></span>

<span data-ttu-id="77a60-217">`CircuitBreaker` クラスは、次のコードに示されている `ICircuitBreakerStateStore` インターフェイスを実装するオブジェクトに、サーキット ブレーカーに関する状態情報を保持します。</span><span class="sxs-lookup"><span data-stu-id="77a60-217">The `CircuitBreaker` class maintains state information about a circuit breaker in an object that implements the `ICircuitBreakerStateStore` interface shown in the following code.</span></span>

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

<span data-ttu-id="77a60-218">`State` プロパティは、サーキット ブレーカーの現在の状態を示し、`CircuitBreakerStateEnum` 列挙型の定義に従って**オープン**、**ハーフオープン**、または**クローズド**のいずれかになります。</span><span class="sxs-lookup"><span data-stu-id="77a60-218">The `State` property indicates the current state of the circuit breaker, and will be either **Open**, **HalfOpen**, or **Closed** as defined by the `CircuitBreakerStateEnum` enumeration.</span></span> <span data-ttu-id="77a60-219">`IsClosed` プロパティは、サーキット ブレーカーがクローズドの場合は true ですが、オープンまたはハーフオープンの場合は false になります。</span><span class="sxs-lookup"><span data-stu-id="77a60-219">The `IsClosed` property should be true if the circuit breaker is closed, but false if it's open or half open.</span></span> <span data-ttu-id="77a60-220">`Trip` メソッドは、サーキット ブレーカーの状態をオープン状態に切り替えて、例外が発生した日時と共に、状態の変化の原因となった例外を記録します。</span><span class="sxs-lookup"><span data-stu-id="77a60-220">The `Trip` method switches the state of the circuit breaker to the open state and records the exception that caused the change in state, together with the date and time that the exception occurred.</span></span> <span data-ttu-id="77a60-221">`LastException` および `LastStateChangedDateUtc` プロパティはこの情報を返します。</span><span class="sxs-lookup"><span data-stu-id="77a60-221">The `LastException` and the `LastStateChangedDateUtc` properties return this information.</span></span> <span data-ttu-id="77a60-222">`Reset` メソッドはサーキット ブレーカーをクローズし、`HalfOpen` メソッドはサーキット ブレーカーをハーフオープンに設定します。</span><span class="sxs-lookup"><span data-stu-id="77a60-222">The `Reset` method closes the circuit breaker, and the `HalfOpen` method sets the circuit breaker to half open.</span></span>

<span data-ttu-id="77a60-223">この例の `InMemoryCircuitBreakerStateStore` クラスには、`ICircuitBreakerStateStore` インターフェイスの実装が含まれています。</span><span class="sxs-lookup"><span data-stu-id="77a60-223">The `InMemoryCircuitBreakerStateStore` class in the example contains an implementation of the `ICircuitBreakerStateStore` interface.</span></span> <span data-ttu-id="77a60-224">`CircuitBreaker` クラスは、このクラスのインスタンスを作成して、サーキット ブレーカーの状態を保持します。</span><span class="sxs-lookup"><span data-stu-id="77a60-224">The `CircuitBreaker` class creates an instance of this class to hold the state of the circuit breaker.</span></span>

<span data-ttu-id="77a60-225">`CircuitBreaker` クラスの `ExecuteAction` メソッドは、`Action` デリゲートとして指定された操作をラップします。</span><span class="sxs-lookup"><span data-stu-id="77a60-225">The `ExecuteAction` method in the `CircuitBreaker` class wraps an operation, specified as an `Action` delegate.</span></span> <span data-ttu-id="77a60-226">サーキット ブレーカーがクローズドの場合は、`ExecuteAction` が `Action` デリゲートを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="77a60-226">If the circuit breaker is closed, `ExecuteAction` invokes the `Action` delegate.</span></span> <span data-ttu-id="77a60-227">操作が失敗すると、例外ハンドラーが `TrackException` を呼び出し、それによってサーキット ブレーカーの状態がオープンに設定されます。</span><span class="sxs-lookup"><span data-stu-id="77a60-227">If the operation fails, an exception handler calls `TrackException`, which sets the circuit breaker state to open.</span></span> <span data-ttu-id="77a60-228">次のコード例はこのフローを示しています。</span><span class="sxs-lookup"><span data-stu-id="77a60-228">The following code example highlights this flow.</span></span>

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

<span data-ttu-id="77a60-229">次の例では、サーキット ブレーカーがクローズドではない場合に実行されるコード (前の例で省略されているもの) を示します。</span><span class="sxs-lookup"><span data-stu-id="77a60-229">The following example shows the code (omitted from the previous example) that is executed if the circuit breaker isn't closed.</span></span> <span data-ttu-id="77a60-230">これは、サーキット ブレーカーが `CircuitBreaker` クラスのローカルの `OpenToHalfOpenWaitTime` フィールドで指定されている時間より長い期間の間オープンだったかどうかをチェックします。</span><span class="sxs-lookup"><span data-stu-id="77a60-230">It first checks if the circuit breaker has been open for a period longer than the time specified by the local `OpenToHalfOpenWaitTime` field in the `CircuitBreaker` class.</span></span> <span data-ttu-id="77a60-231">このような場合、`ExecuteAction` メソッドは、サーキット ブレーカーをハーフオープンに設定してから、`Action` デリゲートによって指定されている操作を実行しようとします。</span><span class="sxs-lookup"><span data-stu-id="77a60-231">If this is the case, the `ExecuteAction` method sets the circuit breaker to half open, then tries to perform the operation specified by the `Action` delegate.</span></span>

<span data-ttu-id="77a60-232">操作が成功すると、サーキット ブレーカーはクローズド状態にリセットされます。</span><span class="sxs-lookup"><span data-stu-id="77a60-232">If the operation is successful, the circuit breaker is reset to the closed state.</span></span> <span data-ttu-id="77a60-233">操作が失敗すると、オープン状態に再度トリップされて例外が発生した時刻が更新されるため、サーキット ブレーカーは、さらに長い時間待機してから操作の実行を再度試みます。</span><span class="sxs-lookup"><span data-stu-id="77a60-233">If the operation fails, it is tripped back to the open state and the time the exception occurred is updated so that the circuit breaker will wait for a further period before trying to perform the operation again.</span></span>

<span data-ttu-id="77a60-234">サーキット ブレーカーが `OpenToHalfOpenWaitTime` 値未満の短い時間しかオープンでなかった場合、`ExecuteAction` メソッドは単に `CircuitBreakerOpenException` 例外をスローし、サーキット ブレーカーがオープン状態に遷移した原因となったエラーを返します。</span><span class="sxs-lookup"><span data-stu-id="77a60-234">If the circuit breaker has only been open for a short time, less than the `OpenToHalfOpenWaitTime` value, the `ExecuteAction` method simply throws a `CircuitBreakerOpenException` exception and returns the error that caused the circuit breaker to transition to the open state.</span></span>

<span data-ttu-id="77a60-235">さらに、ロックを使用して、サーキット ブレーカーがハーフオープンの間に操作への同時呼び出しの実行を試みないようにします。</span><span class="sxs-lookup"><span data-stu-id="77a60-235">Additionally, it uses a lock to prevent the circuit breaker from trying to perform concurrent calls to the operation while it's half open.</span></span> <span data-ttu-id="77a60-236">同時に操作を呼び出す試みは、サーキット ブレーカーがオープンであるかのように処理され、後述のように例外で失敗します。</span><span class="sxs-lookup"><span data-stu-id="77a60-236">A concurrent attempt to invoke the operation will be handled as if the circuit breaker was open, and it'll fail with an exception as described later.</span></span>

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
        }
        catch (Exception ex)
        {
          // If there's still an exception, trip the breaker again immediately.
          this.stateStore.Trip(ex);

          // Throw the exception so that the caller knows which exception occurred.
          throw;
        }
        finally
        {
          if (lockTaken)
          {
            Monitor.Exit(halfOpenSyncObject);
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

<span data-ttu-id="77a60-237">`CircuitBreaker` オブジェクトを使用して操作を保護するために、アプリケーションは `CircuitBreaker` クラスのインスタンスを作成し、実行する操作をパラメーターとして指定して `ExecuteAction` メソッドを呼び出します。</span><span class="sxs-lookup"><span data-stu-id="77a60-237">To use a `CircuitBreaker` object to protect an operation, an application creates an instance of the `CircuitBreaker` class and invokes the `ExecuteAction` method, specifying the operation to be performed as the parameter.</span></span> <span data-ttu-id="77a60-238">サーキット ブレーカーがオープンであるために操作が失敗する場合は、`CircuitBreakerOpenException` 例外をキャッチするようにアプリケーションを準備する必要があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-238">The application should be prepared to catch the `CircuitBreakerOpenException` exception if the operation fails because the circuit breaker is open.</span></span> <span data-ttu-id="77a60-239">次に例を示します。</span><span class="sxs-lookup"><span data-stu-id="77a60-239">The following code shows an example:</span></span>

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="77a60-240">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="77a60-240">Related patterns and guidance</span></span>

<span data-ttu-id="77a60-241">このパターンを実装する場合は、次のパターンも役に立つことがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-241">The following patterns might also be useful when implementing this pattern:</span></span>

- <span data-ttu-id="77a60-242">[再試行パターン](./retry.md)。</span><span class="sxs-lookup"><span data-stu-id="77a60-242">[Retry pattern](./retry.md).</span></span> <span data-ttu-id="77a60-243">失敗した操作を透過的に再試行することで、サービスまたはネットワーク リソースに接続しようとする際に、予測される一時的な障害をアプリケーションがどのように処理できるかを説明しています。</span><span class="sxs-lookup"><span data-stu-id="77a60-243">Describes how an application can handle anticipated temporary failures when it tries to connect to a service or network resource by transparently retrying an operation that has previously failed.</span></span>

- <span data-ttu-id="77a60-244">[正常性エンドポイントの監視パターン](./health-endpoint-monitoring.md)。</span><span class="sxs-lookup"><span data-stu-id="77a60-244">[Health Endpoint Monitoring pattern](./health-endpoint-monitoring.md).</span></span> <span data-ttu-id="77a60-245">サーキット ブレーカーは、サービスによって公開されるエンドポイントに要求を送信することで、サービスの正常性をテストできることがあります。</span><span class="sxs-lookup"><span data-stu-id="77a60-245">A circuit breaker might be able to test the health of a service by sending a request to an endpoint exposed by the service.</span></span> <span data-ttu-id="77a60-246">サービスは、その状態を示す情報を返す必要があります。</span><span class="sxs-lookup"><span data-stu-id="77a60-246">The service should return information indicating its status.</span></span>
