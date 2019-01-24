---
title: 操作に合わせた設計
titleSuffix: Azure Application Architecture Guide
description: 運用チームが必要なツールを得られるように、アプリケーションを設計します。
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 75eaa7f8e322c66a83d2b43d180780a2fdcd745b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480486"
---
# <a name="design-for-operations"></a><span data-ttu-id="fe6c1-103">操作に合わせた設計</span><span class="sxs-lookup"><span data-stu-id="fe6c1-103">Design for operations</span></span>

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a><span data-ttu-id="fe6c1-104">運用チームが必要なツールを得られるようにアプリケーションを設計します</span><span class="sxs-lookup"><span data-stu-id="fe6c1-104">Design an application so that the operations team has the tools they need</span></span>

<span data-ttu-id="fe6c1-105">クラウドによって、運用チームの役割は大きく変わりました。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-105">The cloud has dramatically changed the role of the operations team.</span></span> <span data-ttu-id="fe6c1-106">アプリケーションをホストするハードウェアやインフラストラクチャの管理は、彼らの担当業務ではなくなったのです。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-106">They are no longer responsible for managing the hardware and infrastructure that hosts the application.</span></span>  <span data-ttu-id="fe6c1-107">とは言え、クラウド アプリケーションを効果的に実行するうえで、運用チームは今でも重要な役割を担っています。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-107">That said, operations is still a critical part of running a successful cloud application.</span></span> <span data-ttu-id="fe6c1-108">運用チームの重要な役割としては、次のことが挙げられます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-108">Some of the important functions of the operations team include:</span></span>

- <span data-ttu-id="fe6c1-109">Deployment</span><span class="sxs-lookup"><span data-stu-id="fe6c1-109">Deployment</span></span>
- <span data-ttu-id="fe6c1-110">監視</span><span class="sxs-lookup"><span data-stu-id="fe6c1-110">Monitoring</span></span>
- <span data-ttu-id="fe6c1-111">エスカレーション</span><span class="sxs-lookup"><span data-stu-id="fe6c1-111">Escalation</span></span>
- <span data-ttu-id="fe6c1-112">インシデント対応</span><span class="sxs-lookup"><span data-stu-id="fe6c1-112">Incident response</span></span>
- <span data-ttu-id="fe6c1-113">セキュリティ監査</span><span class="sxs-lookup"><span data-stu-id="fe6c1-113">Security auditing</span></span>

<span data-ttu-id="fe6c1-114">ログ記録とトレースを安定的に機能させることは、クラウド アプリケーションにおいて非常に重要です。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-114">Robust logging and tracing are particularly important in cloud applications.</span></span> <span data-ttu-id="fe6c1-115">業務に必要なデータやインサイトをユーザーに提供できるようにするには、運用チームと連携してアプリケーションを設計、計画する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-115">Involve the operations team in design and planning, to ensure the application gives them the data and insight thay need to be successful.</span></span>  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a><span data-ttu-id="fe6c1-116">Recommendations</span><span class="sxs-lookup"><span data-stu-id="fe6c1-116">Recommendations</span></span>

<span data-ttu-id="fe6c1-117">**すべての要素を観測可能にする**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-117">**Make all things observable**.</span></span> <span data-ttu-id="fe6c1-118">ソリューションが展開されて実行された後、ログとトレースはシステムに関する主要なインサイトになります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-118">Once a solution is deployed and running, logs and traces are your primary insight into the system.</span></span> <span data-ttu-id="fe6c1-119">*トレース*は、システム内のパスを記録するものです。これは、ボトルネック、パフォーマンス問題、および障害点を特定するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-119">*Tracing* records a path through the system, and is useful to pinpoint bottlenecks, performance issues, and failure points.</span></span> <span data-ttu-id="fe6c1-120">*ログ記録*は、アプリケーション状態の変更、エラー、例外など、個々 のイベントをキャプチャするものです。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-120">*Logging* captures individual events such as application state changes, errors, and exceptions.</span></span> <span data-ttu-id="fe6c1-121">運用環境では必ずログを記録しましょう。この記録がないと、必要なインサイトを必要なタイミングで取得できなくなります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-121">Log in production, or else you lose insight at the very times when you need it the most.</span></span>

<span data-ttu-id="fe6c1-122">**監視機能をインストルメント化する**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-122">**Instrument for monitoring**.</span></span> <span data-ttu-id="fe6c1-123">監視機能は、アプリケーションがどの程度正常に (または異常に) 実行されているかを、可用性、パフォーマンス、およびシステム正常性の観点から観測し、そのインサイトを提供するものです。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-123">Monitoring gives insight into how well (or poorly) an application is performing, in terms of availability, performance, and system health.</span></span> <span data-ttu-id="fe6c1-124">たとえば、監視データでは、SLA が達成されているかどうかなどを確認できます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-124">For example, monitoring tells you whether you are meeting your SLA.</span></span> <span data-ttu-id="fe6c1-125">監視はシステムの通常運用時に実行されます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-125">Monitoring happens during the normal operation of the system.</span></span> <span data-ttu-id="fe6c1-126">運用スタッフが問題に迅速に対処できるよう、監視はできるだけリアルタイムに実行するようにしましょう。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-126">It should be as close to real-time as possible, so that the operations staff can react to issues quickly.</span></span> <span data-ttu-id="fe6c1-127">監視機能を上手く使用すれば、重大なエラーの発生前に問題を回避することができます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-127">Ideally, monitoring can help avert problems before they lead to a critical failure.</span></span> <span data-ttu-id="fe6c1-128">詳細については、「[監視と診断][monitoring]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-128">For more information, see [Monitoring and diagnostics][monitoring].</span></span>

<span data-ttu-id="fe6c1-129">**根本原因分析をインストルメント化する**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-129">**Instrument for root cause analysis**.</span></span> <span data-ttu-id="fe6c1-130">根本原因分析は、障害の根本原因を特定するプロセスです。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-130">Root cause analysis is the process of finding the underlying cause of failures.</span></span> <span data-ttu-id="fe6c1-131">これは、既にエラーが発生した後で実行されます。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-131">It occurs after a failure has already happened.</span></span>

<span data-ttu-id="fe6c1-132">**分散トランザクションを使用する**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-132">**Use distributed tracing**.</span></span> <span data-ttu-id="fe6c1-133">コンカレンシー、非同期性、およびクラウド スケールを考慮して設計された分散トレース システムを使用しましょう。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-133">Use a distributed tracing system that is designed for concurrency, asynchrony, and cloud scale.</span></span> <span data-ttu-id="fe6c1-134">トレースには、サービス境界間のフローを示す相関 ID を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-134">Traces should include a correlation ID that flows across service boundaries.</span></span> <span data-ttu-id="fe6c1-135">1 つの操作で複数のアプリケーション サービスへの呼び出しが実行される場合もありますが、</span><span class="sxs-lookup"><span data-stu-id="fe6c1-135">A single operation may involve calls to multiple application services.</span></span> <span data-ttu-id="fe6c1-136">相関 ID あれば、操作が失敗した場合でも、エラーの原因を特定しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-136">If an operation fails, the correlation ID helps to pinpoint the cause of the failure.</span></span>

<span data-ttu-id="fe6c1-137">**ログとメトリックを標準化する**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-137">**Standardize logs and metrics**.</span></span> <span data-ttu-id="fe6c1-138">運用チームでは、ソリューション内のさまざまなサービスからログを集計する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-138">The operations team will need to aggregate logs from across the various services in your solution.</span></span> <span data-ttu-id="fe6c1-139">すべてのサービスで独自のログ ファイル形式が使用されていると、そこからの有用な情報を取得することが困難 (または不可能) になります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-139">If every service uses its own logging format, it becomes difficult or impossible to get useful information from them.</span></span> <span data-ttu-id="fe6c1-140">相関 ID、イベント名、送信者の IP アドレスなどのフィールドを含んだ、共通のスキーマを定義するようにしましょう。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-140">Define a common schema that includes fields such as correlation ID, event name, IP address of the sender, and so forth.</span></span> <span data-ttu-id="fe6c1-141">個々 のサービスで、基本スキーマを継承したカスタム スキーマを派生させたり、追加フィールドを格納したりすることは可能です。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-141">Individual services can derive custom schemas that inherit the base schema, and contain additional fields.</span></span>

<span data-ttu-id="fe6c1-142">**管理タスクを自動化する** (プロビジョニング、デプロイ、監視など)。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-142">**Automate management tasks**, including provisioning, deployment, and monitoring.</span></span> <span data-ttu-id="fe6c1-143">タスクを自動化することで、タスクが反復可能になり、ヒューマン エラーも発生しにくくなります。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-143">Automating a task makes it repeatable and less prone to human errors.</span></span>

<span data-ttu-id="fe6c1-144">**構成をコードとして扱う**。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-144">**Treat configuration as code**.</span></span> <span data-ttu-id="fe6c1-145">構成ファイルをバージョン管理システム内に含めて、変更の追跡やバージョン管理を可能にするとともに、必要に応じてロールバックも実行できるようにしましょう。</span><span class="sxs-lookup"><span data-stu-id="fe6c1-145">Check configuration files into a version control system, so that you can track and version your changes, and roll back if needed.</span></span>

<!-- links -->

[monitoring]: ../../best-practices/monitoring.md
