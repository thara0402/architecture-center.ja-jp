---
title: Retry
description: 予測される一時的な障害をアプリケーションが処理できるようにします。アプリケーションがサービスまたはネットワーク リソースに接続しようとする際に、失敗した操作を透過的に再試行します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 73fdcbcc2bd75593a4c8e33dc2259c90593e14db
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/23/2018
---
# <a name="retry-pattern"></a><span data-ttu-id="3997a-104">再試行パターン</span><span class="sxs-lookup"><span data-stu-id="3997a-104">Retry pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="3997a-105">一時的な障害をアプリケーションが処理できるようにします。アプリケーションがサービスまたはネットワーク リソースに接続しようとしたときに失敗した操作を透過的に再試行します。</span><span class="sxs-lookup"><span data-stu-id="3997a-105">Enable an application to handle transient failures when it tries to connect to a service or network resource, by transparently retrying a failed operation.</span></span> <span data-ttu-id="3997a-106">これにより、アプリケーションの安定性を向上させることができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-106">This can improve the stability of the application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="3997a-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="3997a-107">Context and problem</span></span>

<span data-ttu-id="3997a-108">クラウドで実行されている要素と通信するアプリケーションは、この環境で発生する一時的な障害に敏感である必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-108">An application that communicates with elements running in the cloud has to be sensitive to the transient faults that can occur in this environment.</span></span> <span data-ttu-id="3997a-109">障害とは、たとえば、コンポーネントやサービスとのネットワーク接続が一瞬失われたり、サービスを一時的に利用できなくなったり、サービスがビジー状態となってタイムアウトしたりすることが該当します。</span><span class="sxs-lookup"><span data-stu-id="3997a-109">Faults include the momentary loss of network connectivity to components and services, the temporary unavailability of a service, or timeouts that occur when a service is busy.</span></span>

<span data-ttu-id="3997a-110">通常、これらの障害は自動修正され、障害のトリガーとなったアクションを適切な待ち時間の経過後に再試行すると、高い確率で正常に実行されます。</span><span class="sxs-lookup"><span data-stu-id="3997a-110">These faults are typically self-correcting, and if the action that triggered a fault is repeated after a suitable delay it's likely to be successful.</span></span> <span data-ttu-id="3997a-111">たとえば、多数の同時要求を処理しているデータベース サービスは、ワークロードが緩和されるまで一時的に新たな要求を拒否する調整ストラテジを実装することができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-111">For example, a database service that's processing a large number of concurrent requests can implement a throttling strategy that temporarily rejects any further requests until its workload has eased.</span></span> <span data-ttu-id="3997a-112">データベースにアクセスしようとしているアプリケーションが接続に失敗した場合は、時間をおいてから接続を再試行すれば成功する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-112">An application trying to access the database might fail to connect, but if it tries again after a delay it might succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="3997a-113">解決策</span><span class="sxs-lookup"><span data-stu-id="3997a-113">Solution</span></span>

<span data-ttu-id="3997a-114">クラウドでは、一時的な障害は珍しいことではないため、それらを適切かつ透過的に処理するようにアプリケーションを設計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-114">In the cloud, transient faults aren't uncommon and an application should be designed to handle them elegantly and transparently.</span></span> <span data-ttu-id="3997a-115">これにより、アプリケーションによって実行されているビジネス タスクに与える可能性がある障害の影響を最小限に抑えることができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-115">This minimizes the effects faults can have on the business tasks the application is performing.</span></span>

<span data-ttu-id="3997a-116">アプリケーションがリモート サービスに要求を送信しようとしたときに障害が検出された場合は、次の方法を使用して障害を処理できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-116">If an application detects a failure when it tries to send a request to a remote service, it can handle the failure using the following strategies:</span></span>

- <span data-ttu-id="3997a-117">**キャンセルする**。</span><span class="sxs-lookup"><span data-stu-id="3997a-117">**Cancel**.</span></span> <span data-ttu-id="3997a-118">障害が一時的でないか、操作を繰り返しても成功する可能性が低い場合、アプリケーションは、操作をキャンセルして例外を報告する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-118">If the fault indicates that the failure isn't transient or is unlikely to be successful if repeated, the application should cancel the operation and report an exception.</span></span> <span data-ttu-id="3997a-119">たとえば、無効な資格情報を提供することで発生した認証の失敗は、何回試しても成功する見込みはありません。</span><span class="sxs-lookup"><span data-stu-id="3997a-119">For example, an authentication failure caused by providing invalid credentials is not likely to succeed no matter how many times it's attempted.</span></span>

- <span data-ttu-id="3997a-120">**再試行する**。</span><span class="sxs-lookup"><span data-stu-id="3997a-120">**Retry**.</span></span> <span data-ttu-id="3997a-121">報告された特定の障害が異常であるか、めったに発生しないものである場合は、ネットワーク パケットが転送中に破損しているといった普通でない状況が原因である可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-121">If the specific fault reported is unusual or rare, it might have been caused by unusual circumstances such as a network packet becoming corrupted while it was being transmitted.</span></span> <span data-ttu-id="3997a-122">この場合、アプリケーションは、失敗した要求をすぐに再試行できます。これは、同じ障害が繰り返される可能性は低く、要求はおそらく成功するためです。</span><span class="sxs-lookup"><span data-stu-id="3997a-122">In this case, the application could retry the failing request again immediately because the same failure is unlikely to be repeated and the request will probably be successful.</span></span>

- <span data-ttu-id="3997a-123">**時間をおいて再試行する**。</span><span class="sxs-lookup"><span data-stu-id="3997a-123">**Retry after delay.**</span></span> <span data-ttu-id="3997a-124">障害の原因が、一般的な接続エラーまたはビジー エラーのいずれかである場合、ネットワークまたはサービスは、接続の問題が修正されるか作業のバックログがクリアされるための時間を必要とします。</span><span class="sxs-lookup"><span data-stu-id="3997a-124">If the fault is caused by one of the more commonplace connectivity or busy failures, the network or service might need a short period while the connectivity issues are corrected or the backlog of work is cleared.</span></span> <span data-ttu-id="3997a-125">アプリケーションは、要求を再試行する前に、適切な時間、待機する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-125">The application should wait for a suitable time before retrying the request.</span></span>

<span data-ttu-id="3997a-126">もっと一般的な一時的な障害では、アプリケーションの複数のインスタンスからの要求をできるだけ均等に分散するように再試行の間隔を選択する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-126">For the more common transient failures, the period between retries should be chosen to spread requests from multiple instances of the application as evenly as possible.</span></span> <span data-ttu-id="3997a-127">これにより、ビジー状態にあるサービスが過負荷になる可能性が軽減されます。</span><span class="sxs-lookup"><span data-stu-id="3997a-127">This reduces the chance of a busy service continuing to be overloaded.</span></span> <span data-ttu-id="3997a-128">アプリケーションの多数のインスタンスが再試行要求によってサービスに負荷をかけ続けると、サービスが回復するまでの時間が長くなります。</span><span class="sxs-lookup"><span data-stu-id="3997a-128">If many instances of an application are continually overwhelming a service with retry requests, it'll take the service longer to recover.</span></span>

<span data-ttu-id="3997a-129">要求が失敗する場合、アプリケーションは待機してから再試行できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-129">If the request still fails, the application can wait and make another attempt.</span></span> <span data-ttu-id="3997a-130">必要であれば、要求の最大最高回数に達するまで、再試行の待ち時間を長くしてこのプロセスを繰り返すことができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-130">If necessary, this process can be repeated with increasing delays between retry attempts, until some maximum number of requests have been attempted.</span></span> <span data-ttu-id="3997a-131">待ち時間は、障害の種類とその時間中に修正される確率に応じて、増分的または指数関数的に長くすることができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-131">The delay can be increased incrementally or exponentially, depending on the type of failure and the probability that it'll be corrected during this time.</span></span>

<span data-ttu-id="3997a-132">次の図は、このパターンを使用したホスト サービス内の操作の呼び出しを示しています。</span><span class="sxs-lookup"><span data-stu-id="3997a-132">The following diagram illustrates invoking an operation in a hosted service using this pattern.</span></span> <span data-ttu-id="3997a-133">あらかじめ定義された試行回数で要求が成功しない場合、アプリケーションは、障害を例外として適切に処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-133">If the request is unsuccessful after a predefined number of attempts, the application should treat the fault as an exception and handle it accordingly.</span></span>

![図 1 - 再試行パターンを使用したホスト サービス内の操作の呼び出し](./_images/retry-pattern.png)

<span data-ttu-id="3997a-135">アプリケーションは、リモート サービスにアクセスするすべての試みを、前述の方法のいずれかに一致する再試行ポリシーを実装するコード内にラップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-135">The application should wrap all attempts to access a remote service in code that implements a retry policy matching one of the strategies listed above.</span></span> <span data-ttu-id="3997a-136">異なるサービスに送信される要求は、異なるポリシーの対象にすることができます。</span><span class="sxs-lookup"><span data-stu-id="3997a-136">Requests sent to different services can be subject to different policies.</span></span> <span data-ttu-id="3997a-137">一部のベンダーは、再試行ポリシーを実装するライブラリを提供しています。アプリケーションは、再試行の最大回数、再試行の間隔、およびその他のパラメーターを指定できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-137">Some vendors provide libraries that implement retry policies, where the application can specify the maximum number of retries, the time between retry attempts, and other parameters.</span></span>

<span data-ttu-id="3997a-138">アプリケーションは、障害と失敗した操作の詳細をログに記録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-138">An application should log the details of faults and failing operations.</span></span> <span data-ttu-id="3997a-139">この情報はオペレーターにとって有益です。</span><span class="sxs-lookup"><span data-stu-id="3997a-139">This information is useful to operators.</span></span> <span data-ttu-id="3997a-140">サービスが頻繁に使用できなくなるかビジー状態になる場合、その原因の多くは、サービスのリソースが使い果たされていることです。</span><span class="sxs-lookup"><span data-stu-id="3997a-140">If a service is frequently unavailable or busy, it's often because the service has exhausted its resources.</span></span> <span data-ttu-id="3997a-141">サービスをスケール アウトすることで、このような障害の頻度を軽減できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-141">You can reduce the frequency of these faults by scaling out the service.</span></span> <span data-ttu-id="3997a-142">たとえば、データベース サービスが継続的に過負荷になる場合は、データベースをパーティション分割し、複数のサーバーに負荷を分散すると効果的である可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-142">For example, if a database service is continually overloaded, it might be beneficial to partition the database and spread the load across multiple servers.</span></span>

> <span data-ttu-id="3997a-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/) は、データベースの操作を再試行するための機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="3997a-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/) provides facilities for retrying database operations.</span></span> <span data-ttu-id="3997a-144">ほとんどの Azure サービスとクライアント SDK にも、再試行メカニズムが組み込まれています。</span><span class="sxs-lookup"><span data-stu-id="3997a-144">Also, most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="3997a-145">詳細については、「[特定のサービスの再試行ガイダンス](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3997a-145">For more information, see [Retry guidance for specific services](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific).</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="3997a-146">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="3997a-146">Issues and considerations</span></span>

<span data-ttu-id="3997a-147">このパターンの実装方法を決めるときは、以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="3997a-147">You should consider the following points when deciding how to implement this pattern.</span></span>

<span data-ttu-id="3997a-148">再試行ポリシーは、アプリケーションのビジネス要件と障害の性質と一致するように調整する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-148">The retry policy should be tuned to match the business requirements of the application and the nature of the failure.</span></span> <span data-ttu-id="3997a-149">一部の重要でない操作では、再試行を複数回実行してアプリケーションのスループットに影響を与えるよりも、フェイル ファストすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3997a-149">For some noncritical operations, it's better to fail fast rather than retry several times and impact the throughput of the application.</span></span> <span data-ttu-id="3997a-150">たとえば、リモート サービスにアクセスする対話型の Web アプリケーションでは、数回の再試行を短い待ち時間で実行した後、適切なメッセージ (たとえば、「後でもう一度やり直してください」) をユーザーに表示することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3997a-150">For example, in an interactive web application accessing a remote service, it's better to fail after a smaller number of retries with only a short delay between retry attempts, and display a suitable message to the user (for example, “please try again later”).</span></span> <span data-ttu-id="3997a-151">バッチ アプリケーションの場合は、再試行の待ち時間を指数関数的に長くしながら何度も再試行するほうが適切である可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-151">For a batch application, it might be more appropriate to increase the number of retry attempts with an exponentially increasing delay between attempts.</span></span>

<span data-ttu-id="3997a-152">最短の待ち時間で何回も再試行を行う攻撃的な再試行ポリシーは、フル稼働しているかその状態に近いビジー状態のサービスをさらに低下させる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-152">An aggressive retry policy with minimal delay between attempts, and a large number of retries, could further degrade a busy service that's running close to or at capacity.</span></span> <span data-ttu-id="3997a-153">この再試行ポリシーは、障害が発生した操作を継続的に再試行しようとした場合、アプリケーションの応答性にも影響を与える可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-153">This retry policy could also affect the responsiveness of the application if it's continually trying to perform a failing operation.</span></span>

<span data-ttu-id="3997a-154">何回も再試行した後で要求がまだ失敗する場合、アプリケーションは、同じリソースにさらに要求を送信することを停止し、ただちに障害を報告することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3997a-154">If a request still fails after a significant number of retries, it's better for the application to prevent further requests going to the same resource and simply report a failure immediately.</span></span> <span data-ttu-id="3997a-155">一定時間が経過した後で、アプリケーションは、試しに 1 つまたは複数の要求を送信して、それらが正常に実行されるかどうかを確認できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-155">When the period expires, the application can tentatively allow one or more requests through to see whether they're successful.</span></span> <span data-ttu-id="3997a-156">この方法の詳細については、「[サーキット ブレーカー パターン](circuit-breaker.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3997a-156">For more details of this strategy, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

<span data-ttu-id="3997a-157">操作がべき等であるかどうかを検討します。</span><span class="sxs-lookup"><span data-stu-id="3997a-157">Consider whether the operation is idempotent.</span></span> <span data-ttu-id="3997a-158">その場合、再試行は本質的に安全です。</span><span class="sxs-lookup"><span data-stu-id="3997a-158">If so, it's inherently safe to retry.</span></span> <span data-ttu-id="3997a-159">それ以外の場合、再試行によって操作を複数回実行すると、意図しない副作用が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-159">Otherwise, retries could cause the operation to be executed more than once, with unintended side effects.</span></span> <span data-ttu-id="3997a-160">たとえば、サービスは要求を受信して要求を正常に処理したが、応答の送信に失敗したとします。</span><span class="sxs-lookup"><span data-stu-id="3997a-160">For example, a service might receive the request, process the request successfully, but fail to send a response.</span></span> <span data-ttu-id="3997a-161">その時点で、再試行ロジックは、最初の要求が受信されなかったと仮定して、要求を再送信する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-161">At that point, the retry logic might re-send the request, assuming that the first request wasn't received.</span></span>

<span data-ttu-id="3997a-162">サービスへの要求は、さまざまな理由で失敗し、障害の性質によってさまざまな例外が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-162">A request to a service can fail for a variety of reasons raising different exceptions depending on the nature of the failure.</span></span> <span data-ttu-id="3997a-163">一部の例外はすぐに解決できる障害を示し、一部の例外は継続時間が長い障害を示します。</span><span class="sxs-lookup"><span data-stu-id="3997a-163">Some exceptions indicate a failure that can be resolved quickly, while others indicate that the failure is longer lasting.</span></span> <span data-ttu-id="3997a-164">再試行ポリシーでは、再試行間隔を例外の種類に基づいて調整すると便利です。</span><span class="sxs-lookup"><span data-stu-id="3997a-164">It's useful for the retry policy to adjust the time between retry attempts based on the type of the exception.</span></span>

<span data-ttu-id="3997a-165">トランザクションの一部である操作の再試行がトランザクションの全体的な一貫性に与える影響を検討します。</span><span class="sxs-lookup"><span data-stu-id="3997a-165">Consider how retrying an operation that's part of a transaction will affect the overall transaction consistency.</span></span> <span data-ttu-id="3997a-166">成功の確率を最大化し、トランザクションのすべてのステップを元に戻す必要性を軽減するように、トランザクション操作の再試行ポリシーを微調整します。</span><span class="sxs-lookup"><span data-stu-id="3997a-166">Fine tune the retry policy for transactional operations to maximize the chance of success and reduce the need to undo all the transaction steps.</span></span>

<span data-ttu-id="3997a-167">すべての再試行コードが、さまざまなエラー条件に対して完全にテストされていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="3997a-167">Ensure that all retry code is fully tested against a variety of failure conditions.</span></span> <span data-ttu-id="3997a-168">アプリケーションのパフォーマンスや信頼性に重大な影響を与えないこと、サービスやリソースに過剰な負荷が発生しないこと、競合状態やボトルネックが生成されないことを確認します。</span><span class="sxs-lookup"><span data-stu-id="3997a-168">Check that it doesn't severely impact the performance or reliability of the application, cause excessive load on services and resources, or generate race conditions or bottlenecks.</span></span>

<span data-ttu-id="3997a-169">再試行ロジックは、失敗した操作の完全なコンテキストを理解できる場所のみに実装します。</span><span class="sxs-lookup"><span data-stu-id="3997a-169">Implement retry logic only where the full context of a failing operation is understood.</span></span> <span data-ttu-id="3997a-170">たとえば、再試行ポリシーを含むタスクが、同じように再試行ポリシーを含む別のタスクを呼び出すと、再試行の追加によって、処理の待ち時間がさらに長くなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-170">For example, if a task that contains a retry policy invokes another task that also contains a retry policy, this extra layer of retries can add long delays to the processing.</span></span> <span data-ttu-id="3997a-171">下位レベルのタスクはフェイル ファストするように構成し、呼び出し元のタスクに失敗の理由を報告するほうが適している場合があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-171">It might be better to configure the lower-level task to fail fast and report the reason for the failure back to the task that invoked it.</span></span> <span data-ttu-id="3997a-172">上位レベルのタスクは、独自のポリシーに基づいて障害を処理できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-172">This higher-level task can then handle the failure based on its own policy.</span></span>

<span data-ttu-id="3997a-173">重要なのは、再試行の原因となるすべての接続障害をログに記録して、基になるアプリケーション、サービス、またはリソースの問題を識別できるようすることです。</span><span class="sxs-lookup"><span data-stu-id="3997a-173">It's important to log all connectivity failures that cause a retry so that underlying problems with the application, services, or resources can be identified.</span></span>

<span data-ttu-id="3997a-174">サービスまたはリソースで最も発生する可能性がある障害を調べて、それらの障害が長く続くか末期的になる可能性があるかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="3997a-174">Investigate the faults that are most likely to occur for a service or a resource to discover if they're likely to be long lasting or terminal.</span></span> <span data-ttu-id="3997a-175">該当する場合は、障害を例外として処理することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3997a-175">If they are, it's better to handle the fault as an exception.</span></span> <span data-ttu-id="3997a-176">アプリケーションは、例外を報告するかログに記録した後、別のサービスを呼び出す (使用可能なものがある場合) か、機能を低下させることで、続行することを試行できます。</span><span class="sxs-lookup"><span data-stu-id="3997a-176">The application can report or log the exception, and then try to continue either by invoking an alternative service (if one is available), or by offering degraded functionality.</span></span> <span data-ttu-id="3997a-177">長く続く障害を検出して処理する方法の詳細については、「[サーキット ブレーカー パターン](circuit-breaker.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="3997a-177">For more information on how to detect and handle long-lasting faults, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="3997a-178">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="3997a-178">When to use this pattern</span></span>

<span data-ttu-id="3997a-179">このパターンは、アプリケーションがリモート サービスと対話するかリモート リソースにアクセスするときに、一時的な障害が発生する可能性がある場合に使用します。</span><span class="sxs-lookup"><span data-stu-id="3997a-179">Use this pattern when an application could experience transient faults as it interacts with a remote service or accesses a remote resource.</span></span> <span data-ttu-id="3997a-180">このような障害は長く続かないことが予想されるため、失敗した要求を繰り返し試行すれば成功する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-180">These faults are expected to be short lived, and repeating a request that has previously failed could succeed on a subsequent attempt.</span></span>

<span data-ttu-id="3997a-181">以下の場合は、このパターンの使用は適していません。</span><span class="sxs-lookup"><span data-stu-id="3997a-181">This pattern might not be useful:</span></span>

- <span data-ttu-id="3997a-182">障害が長く続きそうな場合。これは、アプリケーションの応答性に影響する可能性があるためです。</span><span class="sxs-lookup"><span data-stu-id="3997a-182">When a fault is likely to be long lasting, because this can affect the responsiveness of an application.</span></span> <span data-ttu-id="3997a-183">失敗する可能性が高い要求を繰り返し試行すると、アプリケーションが時間とリソースを無駄に費やすことになる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-183">The application might be wasting time and resources trying to repeat a request that's likely to fail.</span></span>
- <span data-ttu-id="3997a-184">一時的な障害が原因ではない障害 (アプリケーションのビジネス ロジックのエラーが原因で発生する内部例外など) を処理するため。</span><span class="sxs-lookup"><span data-stu-id="3997a-184">For handling failures that aren't due to transient faults, such as internal exceptions caused by errors in the business logic of an application.</span></span>
- <span data-ttu-id="3997a-185">システムのスケーラビリティの問題に対応するための代替方法として。</span><span class="sxs-lookup"><span data-stu-id="3997a-185">As an alternative to addressing scalability issues in a system.</span></span> <span data-ttu-id="3997a-186">アプリケーションがビジー状態を頻繁に経験することは、多くの場合、アクセス対象のサービスまたはリソースをスケール アップする必要があることを示すサインです。</span><span class="sxs-lookup"><span data-stu-id="3997a-186">If an application experiences frequent busy faults, it's often a sign that the service or resource being accessed should be scaled up.</span></span>

## <a name="example"></a><span data-ttu-id="3997a-187">例</span><span class="sxs-lookup"><span data-stu-id="3997a-187">Example</span></span>

<span data-ttu-id="3997a-188">次の C# の例は、再試行パターンの実装を示しています。</span><span class="sxs-lookup"><span data-stu-id="3997a-188">This example in C# illustrates an implementation of the Retry pattern.</span></span> <span data-ttu-id="3997a-189">次に示す `OperationWithBasicRetryAsync` メソッドは、`TransientOperationAsync` メソッドを通して外部サービスを非同期に呼び出します。</span><span class="sxs-lookup"><span data-stu-id="3997a-189">The `OperationWithBasicRetryAsync` method, shown below, invokes an external service asynchronously through the `TransientOperationAsync` method.</span></span> <span data-ttu-id="3997a-190">`TransientOperationAsync` メソッドはサービス固有であるため、このサンプル コードでは省略されています。</span><span class="sxs-lookup"><span data-stu-id="3997a-190">The details of the `TransientOperationAsync` method will be specific to the service and are omitted from the sample code.</span></span>

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry, 
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

<span data-ttu-id="3997a-191">このメソッドを呼び出すステートメントは、for ループにラップされた try/catch ブロックに含まれています。</span><span class="sxs-lookup"><span data-stu-id="3997a-191">The statement that invokes this method is contained in a try/catch block wrapped in a for loop.</span></span> <span data-ttu-id="3997a-192">for ループは、`TransientOperationAsync` メソッドの呼び出しが例外のスローなしで成功した場合に実行されます。</span><span class="sxs-lookup"><span data-stu-id="3997a-192">The for loop exits if the call to the `TransientOperationAsync` method succeeds without throwing an exception.</span></span> <span data-ttu-id="3997a-193">`TransientOperationAsync` メソッドが失敗した場合は、catch ブロックが失敗の理由を調べます。</span><span class="sxs-lookup"><span data-stu-id="3997a-193">If the `TransientOperationAsync` method fails, the catch block examines the reason for the failure.</span></span> <span data-ttu-id="3997a-194">一時的なエラーであると思われる場合、このコードは、操作を再試行する前に短時間待機します。</span><span class="sxs-lookup"><span data-stu-id="3997a-194">If it's believed to be a transient error the code waits for a short delay before retrying the operation.</span></span>

<span data-ttu-id="3997a-195">for ループは操作の試行回数も追跡し、コードが 3 回失敗した場合は、例外が長く続くとみなします。</span><span class="sxs-lookup"><span data-stu-id="3997a-195">The for loop also tracks the number of times that the operation has been attempted, and if the code fails three times the exception is assumed to be more long lasting.</span></span> <span data-ttu-id="3997a-196">例外が一時的ではなく、長く続く場合は、catch ハンドラーが例外をスローします。</span><span class="sxs-lookup"><span data-stu-id="3997a-196">If the exception isn't transient or it's long lasting, the catch handler throws an exception.</span></span> <span data-ttu-id="3997a-197">この例外によって for ループが終了し、例外は、`OperationWithBasicRetryAsync` メソッドを呼び出すコードによってキャッチされます。</span><span class="sxs-lookup"><span data-stu-id="3997a-197">This exception exits the for loop and should be caught by the code that invokes the `OperationWithBasicRetryAsync` method.</span></span>

<span data-ttu-id="3997a-198">次に示す `IsTransient` メソッドは、コードが実行されている環境に関連する例外のセットをチェックします。</span><span class="sxs-lookup"><span data-stu-id="3997a-198">The `IsTransient` method, shown below, checks for a specific set of exceptions that are relevant to the environment the code is run in.</span></span> <span data-ttu-id="3997a-199">一時的な例外の定義は、アクセスされるリソースと操作が実行される環境によって異なります。</span><span class="sxs-lookup"><span data-stu-id="3997a-199">The definition of a transient exception will vary according to the resources being accessed and the environment the operation is being performed in.</span></span>

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="3997a-200">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="3997a-200">Related patterns and guidance</span></span>

- <span data-ttu-id="3997a-201">[サーキット ブレーカー パターン](circuit-breaker.md)。</span><span class="sxs-lookup"><span data-stu-id="3997a-201">[Circuit Breaker pattern](circuit-breaker.md).</span></span> <span data-ttu-id="3997a-202">再試行パターンは、一時的な障害を処理するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="3997a-202">The Retry pattern is useful for handling transient faults.</span></span> <span data-ttu-id="3997a-203">障害が長く続くことが予想される場合は、サーキット ブレーカー パターンを実装するほうが適切である可能があります。</span><span class="sxs-lookup"><span data-stu-id="3997a-203">If a failure is expected to be more long lasting, it might be more appropriate to implement the Circuit Breaker pattern.</span></span> <span data-ttu-id="3997a-204">再試行パターンとサーキット ブレーカー パターンを組み合わせて、障害を処理するための包括的なアプローチを提供することもできます。</span><span class="sxs-lookup"><span data-stu-id="3997a-204">The Retry pattern can also be used in conjunction with a circuit breaker to provide a comprehensive approach to handling faults.</span></span>
- [<span data-ttu-id="3997a-205">特定のサービスの再試行ガイダンス</span><span class="sxs-lookup"><span data-stu-id="3997a-205">Retry guidance for specific services</span></span>](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)
- [<span data-ttu-id="3997a-206">接続の回復性</span><span class="sxs-lookup"><span data-stu-id="3997a-206">Connection Resiliency</span></span>](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency)
