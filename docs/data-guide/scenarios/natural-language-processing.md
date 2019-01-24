---
title: 自然言語処理
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 281a2e9995d1d04aa9688e811e0d4ff8088fe30b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483005"
---
# <a name="natural-language-processing"></a><span data-ttu-id="3ff22-102">自然言語処理</span><span class="sxs-lookup"><span data-stu-id="3ff22-102">Natural language processing</span></span>

<span data-ttu-id="3ff22-103">自然言語処理 (NLP) は、感情分析、トピック検出、言語検出、キー フレーズ抽出、およびドキュメント分類などのタスクに使用されます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-103">Natural language processing (NLP) is used for tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span>

![自然言語処理パイプラインの図](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a><span data-ttu-id="3ff22-105">このソリューションを使用する状況</span><span class="sxs-lookup"><span data-stu-id="3ff22-105">When to use this solution</span></span>

<span data-ttu-id="3ff22-106">NLP は、"重要" や "迷惑" としてドキュメントをラベル付けするなど、ドキュメントの分類に使用できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-106">NLP can be use to classify documents, such as labeling documents as sensitive or spam.</span></span> <span data-ttu-id="3ff22-107">NLP の出力は、後続の処理や検索に使用できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-107">The output of NLP can be used for subsequent processing or search.</span></span> <span data-ttu-id="3ff22-108">NLP の別の用途として、ドキュメントに表示されたエンティティの識別によるテキストの集約があります。</span><span class="sxs-lookup"><span data-stu-id="3ff22-108">Another use for NLP is to summarize text by identifying the entities present in the document.</span></span> <span data-ttu-id="3ff22-109">また、これらのエンティティは、キーワードによるドキュメントのタグ付けに使用され、コンテンツに基づく検索や取得を可能にします。</span><span class="sxs-lookup"><span data-stu-id="3ff22-109">These entities can also be used to tag documents with keywords, which enables search and retrieval based on content.</span></span> <span data-ttu-id="3ff22-110">エンティティは、トピックに結び付けることができ、各ドキュメントに示された重要なトピックを説明する概要も使用できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-110">Entities might be combined into topics, with summaries that describe the important topics present in each document.</span></span> <span data-ttu-id="3ff22-111">検出されたトピックは、ナビゲーションのためのドキュメントの分類や、選択されたトピックに関連するドキュメントの列挙に使用できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-111">The detected topics may be used to categorize the documents for navigation, or to enumerate related documents given a selected topic.</span></span> <span data-ttu-id="3ff22-112">NLP のもう 1 つの用途として、ドキュメントのポジティブまたはネガティブな文調を評価する、感情に関するテキストのスコア化があります。</span><span class="sxs-lookup"><span data-stu-id="3ff22-112">Another use for NLP is to score text for sentiment, to assess the positive or negative tone of a document.</span></span> <span data-ttu-id="3ff22-113">これらの手法では、次のような自然言語処理から多くの技術を活用しています。</span><span class="sxs-lookup"><span data-stu-id="3ff22-113">These approaches use many techniques from natural language processing, such as:</span></span>

- <span data-ttu-id="3ff22-114">**トークナイザー**。</span><span class="sxs-lookup"><span data-stu-id="3ff22-114">**Tokenizer**.</span></span> <span data-ttu-id="3ff22-115">テキストを語句に分割する。</span><span class="sxs-lookup"><span data-stu-id="3ff22-115">Splitting the text into words or phrases.</span></span>
- <span data-ttu-id="3ff22-116">**語幹検索および見出し語認定**</span><span class="sxs-lookup"><span data-stu-id="3ff22-116">**Stemming and lemmatization**.</span></span> <span data-ttu-id="3ff22-117">さまざまな語形を同じ意味の一般的な単語にマッピングするために、単語を正規化する </span><span class="sxs-lookup"><span data-stu-id="3ff22-117">Normalizing words so that that different forms map to the canonical word with the same meaning.</span></span> <span data-ttu-id="3ff22-118">("running" と "ran" を "run" にマップするなど)。</span><span class="sxs-lookup"><span data-stu-id="3ff22-118">For example, "running" and "ran" map to "run."</span></span>
- <span data-ttu-id="3ff22-119">**エンティティ抽出**。</span><span class="sxs-lookup"><span data-stu-id="3ff22-119">**Entity extraction**.</span></span> <span data-ttu-id="3ff22-120">テキストの主題を識別する。</span><span class="sxs-lookup"><span data-stu-id="3ff22-120">Identifying subjects in the text.</span></span>
- <span data-ttu-id="3ff22-121">**品詞検出**。</span><span class="sxs-lookup"><span data-stu-id="3ff22-121">**Part of speech detection**.</span></span> <span data-ttu-id="3ff22-122">動詞、名詞、分詞、動詞句などにテキストを識別する。</span><span class="sxs-lookup"><span data-stu-id="3ff22-122">Identifying text as a verb, noun, participle, verb phrase, and so on.</span></span>
- <span data-ttu-id="3ff22-123">**文の境界検出**。</span><span class="sxs-lookup"><span data-stu-id="3ff22-123">**Sentence boundary detection**.</span></span> <span data-ttu-id="3ff22-124">段落のテキスト内で完全な文を検出します。</span><span class="sxs-lookup"><span data-stu-id="3ff22-124">Detecting complete sentences within paragraphs of text.</span></span>

<span data-ttu-id="3ff22-125">NLP を使用して自由形式のテキストから情報および洞察を抽出する場合、通常は Azure Storage や Azure Data Lake Store などオブジェクト ストレージに格納された未処理のドキュメントから開始します。</span><span class="sxs-lookup"><span data-stu-id="3ff22-125">When using NLP to extract information and insight from free-form text, the starting point is typically the raw documents stored in object storage such as Azure Storage or Azure Data Lake Store.</span></span>

## <a name="challenges"></a><span data-ttu-id="3ff22-126">課題</span><span class="sxs-lookup"><span data-stu-id="3ff22-126">Challenges</span></span>

- <span data-ttu-id="3ff22-127">自由形式のテキスト ドキュメントのコレクションの処理は通常、時間がかかるだけでなく、コンピューティングにおけるリソース負荷も高くなります。</span><span class="sxs-lookup"><span data-stu-id="3ff22-127">Processing a collection of free-form text documents is typically computationally resource intensive, as well as being time intensive.</span></span>
- <span data-ttu-id="3ff22-128">標準化されたドキュメント形式がなく、自由形式のテキスト処理を使用してドキュメントから特定のファクトを抽出して、常に正確な結果に到達することは、大変困難です。</span><span class="sxs-lookup"><span data-stu-id="3ff22-128">Without a standardized document format, it can be very difficult to achieve consistently accurate results using free-form text processing to extract specific facts from a document.</span></span> <span data-ttu-id="3ff22-129">たとえば、請求書のテキスト表現を考えてみてください。複数のベンダーの請求書から請求番号と請求日を正確に抽出する処理を構築することは、難しい可能性があります。</span><span class="sxs-lookup"><span data-stu-id="3ff22-129">For example, think of a text representation of an invoice&mdash;it can be difficult to build a process that correctly extracts the invoice number and invoice date for invoices across any number of vendors.</span></span>

## <a name="architecture"></a><span data-ttu-id="3ff22-130">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="3ff22-130">Architecture</span></span>

<span data-ttu-id="3ff22-131">NLP ソリューションでは、自由形式のテキスト処理は、段落のテキストを含むドキュメントに対して実行されます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-131">In an NLP solution, free-form text processing is performed against documents containing paragraphs of text.</span></span> <span data-ttu-id="3ff22-132">全体のアーキテクチャは、[バッチ処理](../big-data/batch-processing.md)または[リアルタイムでのストリーム処理](../big-data/real-time-processing.md)のアーキテクチャになる場合があります。</span><span class="sxs-lookup"><span data-stu-id="3ff22-132">The overall architecture can be a [batch processing](../big-data/batch-processing.md) or [real-time stream processing](../big-data/real-time-processing.md) architecture.</span></span>

<span data-ttu-id="3ff22-133">実際の処理は、目的とする成果物によって異なりますが、パイプラインの観点では、バッチやリアルタイム方式で NLP を適用できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-133">The actual processing varies based on the desired outcome, but in terms of the pipeline, NLP may be applied in a batch or real-time fashion.</span></span> <span data-ttu-id="3ff22-134">たとえば、感情分析は、テキスト ブロックに対して使用して、感情スコアを生成できます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-134">For example, sentiment analysis can be used against blocks of text to produce a sentiment score.</span></span> <span data-ttu-id="3ff22-135">この分析は、ストレージのデータに対してバッチ処理を実行したり、リアルタムでメッセージング サービス経由で送信されたより小さなデータチャンクを使用して、行うことができます。</span><span class="sxs-lookup"><span data-stu-id="3ff22-135">This can could be done by running a batch process against data in storage, or in real time using smaller chunks of data flowing through a messaging service.</span></span>

## <a name="technology-choices"></a><span data-ttu-id="3ff22-136">テクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="3ff22-136">Technology choices</span></span>

- [<span data-ttu-id="3ff22-137">自然言語処理</span><span class="sxs-lookup"><span data-stu-id="3ff22-137">Natural language processing</span></span>](../technology-choices/natural-language-processing.md)
