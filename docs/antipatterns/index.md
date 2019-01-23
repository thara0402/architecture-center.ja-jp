---
title: クラウド アプリケーションのパフォーマンスのアンチパターン
titleSuffix: Azure Architecture Center
description: スケーラビリティの問題を引き起こす可能性がある一般的なプラクティス。
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 212930368942728fc0be0c9b2af1a90293906b39
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483056"
---
# <a name="performance-antipatterns-for-cloud-applications"></a><span data-ttu-id="a5bba-103">クラウド アプリケーションのパフォーマンスのアンチパターン</span><span class="sxs-lookup"><span data-stu-id="a5bba-103">Performance antipatterns for cloud applications</span></span>

<span data-ttu-id="a5bba-104">"*パフォーマンスのアンチパターン*" は、アプリケーションの負荷が高いときに、スケーラビリティの問題を引き起こす可能性がある一般的なプラクティスです。</span><span class="sxs-lookup"><span data-stu-id="a5bba-104">A *performance antipattern* is a common practice that is likely to cause scalability problems when an application is under pressure.</span></span>

<span data-ttu-id="a5bba-105">一般的なシナリオは次のとおりです。アプリケーションはパフォーマンス テストで適切に動作しました。</span><span class="sxs-lookup"><span data-stu-id="a5bba-105">Here is a common scenario: An application behaves well during performance testing.</span></span> <span data-ttu-id="a5bba-106">これが実稼働用にリリースされ、実際のワークロードの処理を開始します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-106">It's released to production, and begins to handle real workloads.</span></span> <span data-ttu-id="a5bba-107">ここで、ユーザー要求の拒否、失速、例外のスローなど、パフォーマンスが悪化し始めます。</span><span class="sxs-lookup"><span data-stu-id="a5bba-107">At that point, it starts to perform poorly &mdash; rejecting user requests, stalling, or throwing exceptions.</span></span> <span data-ttu-id="a5bba-108">開発チームは 2 つの質問に直面します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-108">The development team is then faced with two questions:</span></span>

- <span data-ttu-id="a5bba-109">なぜこの動作がテスト中に現れなかったのか。</span><span class="sxs-lookup"><span data-stu-id="a5bba-109">Why didn't this behavior show up during testing?</span></span>
- <span data-ttu-id="a5bba-110">どうやって直すか。</span><span class="sxs-lookup"><span data-stu-id="a5bba-110">How do we fix it?</span></span>

<span data-ttu-id="a5bba-111">最初の質問の答えは単純です。</span><span class="sxs-lookup"><span data-stu-id="a5bba-111">The answer to the first question is straightforward.</span></span> <span data-ttu-id="a5bba-112">テスト環境で、実際のユーザー、その行動パターン、実行する可能性がある作業のボリュームをシミュレートするのは非常に困難です。</span><span class="sxs-lookup"><span data-stu-id="a5bba-112">It's very difficult in a test environment to simulate real users, their behavior patterns, and the volumes of work they might perform.</span></span> <span data-ttu-id="a5bba-113">システムが負荷の下でどのように動作するかを理解する確実な方法は、実稼働環境で観察することだけです。</span><span class="sxs-lookup"><span data-stu-id="a5bba-113">The only completely sure way to understand how a system behaves under load is to observe it in production.</span></span> <span data-ttu-id="a5bba-114">パフォーマンス テストを省略するようにお薦めしているのではないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="a5bba-114">To be clear, we aren't suggesting that you should skip performance testing.</span></span> <span data-ttu-id="a5bba-115">パフォーマンス テストはベースラインのパフォーマンス メトリックを得るために不可欠です。</span><span class="sxs-lookup"><span data-stu-id="a5bba-115">Performance tests are crucial for getting baseline performance metrics.</span></span> <span data-ttu-id="a5bba-116">しかし、パフォーマンスの問題が稼働中のシステムで発生した際に観察して解決できるように準備しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5bba-116">But you must be prepared to observe and correct performance issues when they arise in the live system.</span></span>

<span data-ttu-id="a5bba-117">この状況を直す方法という 2 つ目の質問の回答は単純ではありません。</span><span class="sxs-lookup"><span data-stu-id="a5bba-117">The answer to the second question, how to fix the problem, is less straightforward.</span></span> <span data-ttu-id="a5bba-118">いくつもの要因が関係している可能性があり、場合によっては特定の状況のみで問題が発生します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-118">Any number of factors might contribute, and sometimes the problem only manifests under certain circumstances.</span></span> <span data-ttu-id="a5bba-119">インストルメント化とログは根本原因を突き止めるために重要ですが、何を調べるべきかを知ることも必要です。</span><span class="sxs-lookup"><span data-stu-id="a5bba-119">Instrumentation and logging are key to finding the root cause, but you also have to know what to look for.</span></span>

<span data-ttu-id="a5bba-120">マイクロソフトでは、Microsoft Azure の顧客エンゲージメントに基づいて、お客様が実稼働環境で目にする一般的なパフォーマンスの問題の一部を特定しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-120">Based on our engagements with Microsoft Azure customers, we've identified some of the most common performance issues that customers see in production.</span></span> <span data-ttu-id="a5bba-121">各アンチパターンについて、一般的にそのアンチパターンが発生する理由、アンチパターンの症状、問題を解決する方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-121">For each antipattern, we describe why the antipattern typically occurs, symptoms of the antipattern, and techniques for resolving the problem.</span></span> <span data-ttu-id="a5bba-122">また、アンチパターンと推奨ソリューションの両方を示すサンプル コードも提供しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-122">We also provide sample code that illustrates both the antipattern and a suggested solution.</span></span>

<span data-ttu-id="a5bba-123">説明を読むとアンチパターンのいくつかはあまりにも明白なために回避できそうですが、想像するよりも頻繁に発生しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-123">Some of these antipatterns may seem obvious when you read the descriptions, but they occur more often than you might think.</span></span> <span data-ttu-id="a5bba-124">アプリケーションは、オンプレミスで機能していたがクラウドでは拡張しない設計を継承することがあります。</span><span class="sxs-lookup"><span data-stu-id="a5bba-124">Sometimes an application inherits a design that worked on-premises, but doesn't scale in the cloud.</span></span> <span data-ttu-id="a5bba-125">あるいは、アプリケーションの開発当初は設計がきわめて明確だったが、新しい機能が追加されるにつれて、このようなアンチパターンが知らない間に入り込むことがあります。</span><span class="sxs-lookup"><span data-stu-id="a5bba-125">Or an application might start with a very clean design, but as new features are added, one or more of these antipatterns creeps in.</span></span> <span data-ttu-id="a5bba-126">いずれにせよ、このガイドではアンチパターンを見つけて直す方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-126">Regardless, this guide will help you to identify and fix these antipatterns.</span></span>

<span data-ttu-id="a5bba-127">特定できたアンチパターンの一覧を次に示します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-127">Here is the list of the antipatterns that we've identified:</span></span>

| <span data-ttu-id="a5bba-128">アンチパターン</span><span class="sxs-lookup"><span data-stu-id="a5bba-128">Antipattern</span></span> | <span data-ttu-id="a5bba-129">説明</span><span class="sxs-lookup"><span data-stu-id="a5bba-129">Description</span></span> |
|-------------|-------------|
| <span data-ttu-id="a5bba-130">[ビジー状態のデータベース][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="a5bba-130">[Busy Database][BusyDatabase]</span></span> | <span data-ttu-id="a5bba-131">データ ストアに大量の処理をオフロードします。</span><span class="sxs-lookup"><span data-stu-id="a5bba-131">Offloading too much processing to a data store.</span></span> |
| <span data-ttu-id="a5bba-132">[ビジー状態のフロント エンド][BusyFrontEnd]</span><span class="sxs-lookup"><span data-stu-id="a5bba-132">[Busy Front End][BusyFrontEnd]</span></span> | <span data-ttu-id="a5bba-133">リソースを消費するタスクをバックグラウンド スレッドに移動します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-133">Moving resource-intensive tasks onto background threads.</span></span> |
| <span data-ttu-id="a5bba-134">[頻度の高い I/O][ChattyIO]</span><span class="sxs-lookup"><span data-stu-id="a5bba-134">[Chatty I/O][ChattyIO]</span></span> | <span data-ttu-id="a5bba-135">多数の小さなネットワーク要求を継続的に送信しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-135">Continually sending many small network requests.</span></span> |
| <span data-ttu-id="a5bba-136">[余分なフェッチ][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="a5bba-136">[Extraneous Fetching][ExtraneousFetching]</span></span> | <span data-ttu-id="a5bba-137">必要以上のデータを取得して、不要な I/O を発生させます。</span><span class="sxs-lookup"><span data-stu-id="a5bba-137">Retrieving more data than is needed, resulting in unnecessary I/O.</span></span> |
| <span data-ttu-id="a5bba-138">[不適切なインスタンス化][ImproperInstantiation]</span><span class="sxs-lookup"><span data-stu-id="a5bba-138">[Improper Instantiation][ImproperInstantiation]</span></span> | <span data-ttu-id="a5bba-139">共有して再利用されるように設計されているオブジェクトの作成と破棄を繰り返しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-139">Repeatedly creating and destroying objects that are designed to be shared and reused.</span></span> |
| <span data-ttu-id="a5bba-140">[モノリシック永続化][MonolithicPersistence]</span><span class="sxs-lookup"><span data-stu-id="a5bba-140">[Monolithic Persistence][MonolithicPersistence]</span></span> | <span data-ttu-id="a5bba-141">使用パターンが大きく異なるデータで同じデータ ストアを使用しています。</span><span class="sxs-lookup"><span data-stu-id="a5bba-141">Using the same data store for data with very different usage patterns.</span></span> |
| <span data-ttu-id="a5bba-142">[キャッシュなし][NoCaching]</span><span class="sxs-lookup"><span data-stu-id="a5bba-142">[No Caching][NoCaching]</span></span> | <span data-ttu-id="a5bba-143">データのキャッシュに失敗します。</span><span class="sxs-lookup"><span data-stu-id="a5bba-143">Failing to cache data.</span></span> |
| <span data-ttu-id="a5bba-144">[同期 I/O][SynchronousIO]</span><span class="sxs-lookup"><span data-stu-id="a5bba-144">[Synchronous I/O][SynchronousIO]</span></span> | <span data-ttu-id="a5bba-145">I/O が完了するまで、呼び出し元スレッドをブロックします。</span><span class="sxs-lookup"><span data-stu-id="a5bba-145">Blocking the calling thread while I/O completes.</span></span> |

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md
