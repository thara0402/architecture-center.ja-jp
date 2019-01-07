---
title: クラウド アプリケーションのパフォーマンスのアンチパターン
titleSuffix: Azure Architecture Center
description: スケーラビリティの問題を引き起こす可能性がある一般的なプラクティス。
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 9ce1177bac2c93139faf6bc757f2866d6b7eac2d
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009731"
---
# <a name="performance-antipatterns-for-cloud-applications"></a><span data-ttu-id="362f0-103">クラウド アプリケーションのパフォーマンスのアンチパターン</span><span class="sxs-lookup"><span data-stu-id="362f0-103">Performance antipatterns for cloud applications</span></span>

<span data-ttu-id="362f0-104">"*パフォーマンスのアンチパターン*" は、アプリケーションの負荷が高いときに、スケーラビリティの問題を引き起こす可能性がある一般的なプラクティスです。</span><span class="sxs-lookup"><span data-stu-id="362f0-104">A *performance antipattern* is a common practice that is likely to cause scalability problems when an application is under pressure.</span></span>

<span data-ttu-id="362f0-105">一般的なシナリオは次のとおりです。アプリケーションはパフォーマンス テストで適切に動作しました。</span><span class="sxs-lookup"><span data-stu-id="362f0-105">Here is a common scenario: An application behaves well during performance testing.</span></span> <span data-ttu-id="362f0-106">これが実稼働用にリリースされ、実際のワークロードの処理を開始します。</span><span class="sxs-lookup"><span data-stu-id="362f0-106">It's released to production, and begins to handle real workloads.</span></span> <span data-ttu-id="362f0-107">ここで、ユーザー要求の拒否、失速、例外のスローなど、パフォーマンスが悪化し始めます。</span><span class="sxs-lookup"><span data-stu-id="362f0-107">At that point, it starts to perform poorly &mdash; rejecting user requests, stalling, or throwing exceptions.</span></span> <span data-ttu-id="362f0-108">開発チームは 2 つの質問に直面します。</span><span class="sxs-lookup"><span data-stu-id="362f0-108">The development team is then faced with two questions:</span></span>

- <span data-ttu-id="362f0-109">なぜこの動作がテスト中に現れなかったのか。</span><span class="sxs-lookup"><span data-stu-id="362f0-109">Why didn't this behavior show up during testing?</span></span>
- <span data-ttu-id="362f0-110">どうやって直すか。</span><span class="sxs-lookup"><span data-stu-id="362f0-110">How do we fix it?</span></span>

<span data-ttu-id="362f0-111">最初の質問の答えは単純です。</span><span class="sxs-lookup"><span data-stu-id="362f0-111">The answer to the first question is straightforward.</span></span> <span data-ttu-id="362f0-112">テスト環境で、実際のユーザー、その行動パターン、実行する可能性がある作業のボリュームをシミュレートするのは非常に困難です。</span><span class="sxs-lookup"><span data-stu-id="362f0-112">It's very difficult in a test environment to simulate real users, their behavior patterns, and the volumes of work they might perform.</span></span> <span data-ttu-id="362f0-113">システムが負荷の下でどのように動作するかを理解する確実な方法は、実稼働環境で観察することだけです。</span><span class="sxs-lookup"><span data-stu-id="362f0-113">The only completely sure way to understand how a system behaves under load is to observe it in production.</span></span> <span data-ttu-id="362f0-114">パフォーマンス テストを省略するようにお薦めしているのではないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="362f0-114">To be clear, we aren't suggesting that you should skip performance testing.</span></span> <span data-ttu-id="362f0-115">パフォーマンス テストはベースラインのパフォーマンス メトリックを得るために不可欠です。</span><span class="sxs-lookup"><span data-stu-id="362f0-115">Performance tests are crucial for getting baseline performance metrics.</span></span> <span data-ttu-id="362f0-116">しかし、パフォーマンスの問題が稼働中のシステムで発生した際に観察して解決できるように準備しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="362f0-116">But you must be prepared to observe and correct performance issues when they arise in the live system.</span></span>

<span data-ttu-id="362f0-117">この状況を直す方法という 2 つ目の質問の回答は単純ではありません。</span><span class="sxs-lookup"><span data-stu-id="362f0-117">The answer to the second question, how to fix the problem, is less straightforward.</span></span> <span data-ttu-id="362f0-118">いくつもの要因が関係している可能性があり、場合によっては特定の状況のみで問題が発生します。</span><span class="sxs-lookup"><span data-stu-id="362f0-118">Any number of factors might contribute, and sometimes the problem only manifests under certain circumstances.</span></span> <span data-ttu-id="362f0-119">インストルメント化とログは根本原因を突き止めるために重要ですが、何を調べるべきかを知ることも必要です。</span><span class="sxs-lookup"><span data-stu-id="362f0-119">Instrumentation and logging are key to finding the root cause, but you also have to know what to look for.</span></span>

<span data-ttu-id="362f0-120">マイクロソフトでは、Microsoft Azure の顧客エンゲージメントに基づいて、お客様が実稼働環境で目にする一般的なパフォーマンスの問題の一部を特定しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-120">Based on our engagements with Microsoft Azure customers, we've identified some of the most common performance issues that customers see in production.</span></span> <span data-ttu-id="362f0-121">各アンチパターンについて、一般的にそのアンチパターンが発生する理由、アンチパターンの症状、問題を解決する方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="362f0-121">For each antipattern, we describe why the antipattern typically occurs, symptoms of the antipattern, and techniques for resolving the problem.</span></span> <span data-ttu-id="362f0-122">また、アンチパターンと推奨ソリューションの両方を示すサンプル コードも提供しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-122">We also provide sample code that illustrates both the antipattern and a suggested solution.</span></span>

<span data-ttu-id="362f0-123">説明を読むとアンチパターンのいくつかはあまりにも明白なために回避できそうですが、想像するよりも頻繁に発生しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-123">Some of these antipatterns may seem obvious when you read the descriptions, but they occur more often than you might think.</span></span> <span data-ttu-id="362f0-124">アプリケーションは、オンプレミスで機能していたがクラウドでは拡張しない設計を継承することがあります。</span><span class="sxs-lookup"><span data-stu-id="362f0-124">Sometimes an application inherits a design that worked on-premises, but doesn't scale in the cloud.</span></span> <span data-ttu-id="362f0-125">あるいは、アプリケーションの開発当初は設計がきわめて明確だったが、新しい機能が追加されるにつれて、このようなアンチパターンが知らない間に入り込むことがあります。</span><span class="sxs-lookup"><span data-stu-id="362f0-125">Or an application might start with a very clean design, but as new features are added, one or more of these antipatterns creeps in.</span></span> <span data-ttu-id="362f0-126">いずれにせよ、このガイドではアンチパターンを見つけて直す方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="362f0-126">Regardless, this guide will help you to identify and fix these antipatterns.</span></span>

<span data-ttu-id="362f0-127">特定できたアンチパターンの一覧を次に示します。</span><span class="sxs-lookup"><span data-stu-id="362f0-127">Here is the list of the antipatterns that we've identified:</span></span>

| <span data-ttu-id="362f0-128">アンチパターン</span><span class="sxs-lookup"><span data-stu-id="362f0-128">Antipattern</span></span> | <span data-ttu-id="362f0-129">説明</span><span class="sxs-lookup"><span data-stu-id="362f0-129">Description</span></span> |
|-------------|-------------|
| <span data-ttu-id="362f0-130">[ビジー状態のデータベース][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="362f0-130">[Busy Database][BusyDatabase]</span></span> | <span data-ttu-id="362f0-131">データ ストアに大量の処理をオフロードします。</span><span class="sxs-lookup"><span data-stu-id="362f0-131">Offloading too much processing to a data store.</span></span> |
| <span data-ttu-id="362f0-132">[ビジー状態のフロント エンド][BusyFrontEnd]</span><span class="sxs-lookup"><span data-stu-id="362f0-132">[Busy Front End][BusyFrontEnd]</span></span> | <span data-ttu-id="362f0-133">リソースを消費するタスクをバックグラウンド スレッドに移動します。</span><span class="sxs-lookup"><span data-stu-id="362f0-133">Moving resource-intensive tasks onto background threads.</span></span> |
| <span data-ttu-id="362f0-134">[頻度の高い I/O][ChattyIO]</span><span class="sxs-lookup"><span data-stu-id="362f0-134">[Chatty I/O][ChattyIO]</span></span> | <span data-ttu-id="362f0-135">多数の小さなネットワーク要求を継続的に送信しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-135">Continually sending many small network requests.</span></span> |
| <span data-ttu-id="362f0-136">[余分なフェッチ][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="362f0-136">[Extraneous Fetching][ExtraneousFetching]</span></span> | <span data-ttu-id="362f0-137">必要以上のデータを取得して、不要な I/O を発生させます。</span><span class="sxs-lookup"><span data-stu-id="362f0-137">Retrieving more data than is needed, resulting in unnecessary I/O.</span></span> |
| <span data-ttu-id="362f0-138">[不適切なインスタンス化][ImproperInstantiation]</span><span class="sxs-lookup"><span data-stu-id="362f0-138">[Improper Instantiation][ImproperInstantiation]</span></span> | <span data-ttu-id="362f0-139">共有して再利用されるように設計されているオブジェクトの作成と破棄を繰り返しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-139">Repeatedly creating and destroying objects that are designed to be shared and reused.</span></span> |
| <span data-ttu-id="362f0-140">[モノリシック永続化][MonolithicPersistence]</span><span class="sxs-lookup"><span data-stu-id="362f0-140">[Monolithic Persistence][MonolithicPersistence]</span></span> | <span data-ttu-id="362f0-141">使用パターンが大きく異なるデータで同じデータ ストアを使用しています。</span><span class="sxs-lookup"><span data-stu-id="362f0-141">Using the same data store for data with very different usage patterns.</span></span> |
| <span data-ttu-id="362f0-142">[キャッシュなし][NoCaching]</span><span class="sxs-lookup"><span data-stu-id="362f0-142">[No Caching][NoCaching]</span></span> | <span data-ttu-id="362f0-143">データのキャッシュに失敗します。</span><span class="sxs-lookup"><span data-stu-id="362f0-143">Failing to cache data.</span></span> |
| <span data-ttu-id="362f0-144">[同期 I/O][SynchronousIO]</span><span class="sxs-lookup"><span data-stu-id="362f0-144">[Synchronous I/O][SynchronousIO]</span></span> | <span data-ttu-id="362f0-145">I/O が完了するまで、呼び出し元スレッドをブロックします。</span><span class="sxs-lookup"><span data-stu-id="362f0-145">Blocking the calling thread while I/O completes.</span></span> |

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md
