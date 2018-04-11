---
title: 自然言語処理技術の選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="2ffbc-102">Azure での自然言語処理技術の選択</span><span class="sxs-lookup"><span data-stu-id="2ffbc-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="2ffbc-103">自由形式テキストの処理は、通常は検索をサポートする目的でテキストの段落を含むドキュメントに対して実行されますが、センチメント分析、トピック検出、言語検出、キー フレーズ抽出、ドキュメントの分類などの他の自然言語処理 (NLP) タスクの実行にも使用されます。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="2ffbc-104">この記事では、NLP タスクのサポートで機能するテクノロジの選択に重点を置いて説明します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="2ffbc-105">NLP サービスを選ぶときのオプション</span><span class="sxs-lookup"><span data-stu-id="2ffbc-105">What are your options when choosing an NLP service?</span></span>

<span data-ttu-id="2ffbc-106">Azure では、次のサービスに自然言語処理 (NLP) 機能があります。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="2ffbc-107">Spark および Spark MLlib を使用する Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="2ffbc-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="2ffbc-108">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="2ffbc-108">Microsoft Cognitive Services</span></span>](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a><span data-ttu-id="2ffbc-109">主要な選択条件</span><span class="sxs-lookup"><span data-stu-id="2ffbc-109">Key selection criteria</span></span>

<span data-ttu-id="2ffbc-110">選択を絞り込むには、以下の質問に答えることから開始します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-110">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="2ffbc-111">事前構築済みのモデルを使用しますか。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-111">Do you want to use prebuilt models?</span></span> <span data-ttu-id="2ffbc-112">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-112">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="2ffbc-113">テキスト データの大規模なコーパスに対してカスタム モデルをトレーニングする必要がありますか。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-113">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="2ffbc-114">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-114">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="2ffbc-115">トークン化、ステミング、レンマ化、単語の出現頻度/逆文書頻度 (TF/IDF) のような低レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-115">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="2ffbc-116">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-116">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="2ffbc-117">エンティティと意図の識別、トピック検出、スペル チェック、センチメント分析などのシンプルで高レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-117">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="2ffbc-118">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-118">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="2ffbc-119">機能のマトリックス</span><span class="sxs-lookup"><span data-stu-id="2ffbc-119">Capability matrix</span></span>

<span data-ttu-id="2ffbc-120">次の表は、機能の主な相違点をまとめたものです。</span><span class="sxs-lookup"><span data-stu-id="2ffbc-120">The following tables summarize the key differences in capabilities.</span></span>  

### <a name="general-capabilities"></a><span data-ttu-id="2ffbc-121">一般的な機能</span><span class="sxs-lookup"><span data-stu-id="2ffbc-121">General capabilities</span></span>

| | <span data-ttu-id="2ffbc-122">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="2ffbc-122">Azure HDInsight</span></span> | <span data-ttu-id="2ffbc-123">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="2ffbc-123">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="2ffbc-124">サービスとして事前トレーニング済みモデルを提供</span><span class="sxs-lookup"><span data-stu-id="2ffbc-124">Provides pretrained models as a service</span></span> | <span data-ttu-id="2ffbc-125">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-125">No</span></span> | <span data-ttu-id="2ffbc-126">[はい]</span><span class="sxs-lookup"><span data-stu-id="2ffbc-126">Yes</span></span> |
| <span data-ttu-id="2ffbc-127">REST API</span><span class="sxs-lookup"><span data-stu-id="2ffbc-127">REST API</span></span> | <span data-ttu-id="2ffbc-128">[はい]</span><span class="sxs-lookup"><span data-stu-id="2ffbc-128">Yes</span></span> | <span data-ttu-id="2ffbc-129">[はい]</span><span class="sxs-lookup"><span data-stu-id="2ffbc-129">Yes</span></span> |
| <span data-ttu-id="2ffbc-130">プログラミング</span><span class="sxs-lookup"><span data-stu-id="2ffbc-130">Programmability</span></span> | <span data-ttu-id="2ffbc-131">Python、Scala、Java</span><span class="sxs-lookup"><span data-stu-id="2ffbc-131">Python, Scala, Java</span></span> | <span data-ttu-id="2ffbc-132">C#、Java、Node.js、Python、PHP、Ruby</span><span class="sxs-lookup"><span data-stu-id="2ffbc-132">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="2ffbc-133">大規模なデータ セットとサイズの大きいドキュメントの処理のサポート</span><span class="sxs-lookup"><span data-stu-id="2ffbc-133">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="2ffbc-134">[はい]</span><span class="sxs-lookup"><span data-stu-id="2ffbc-134">Yes</span></span> | <span data-ttu-id="2ffbc-135">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-135">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="2ffbc-136">低レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="2ffbc-136">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="2ffbc-137">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="2ffbc-137">Azure HDInsight</span></span> | <span data-ttu-id="2ffbc-138">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="2ffbc-138">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- | 
| <span data-ttu-id="2ffbc-139">トークナイザー</span><span class="sxs-lookup"><span data-stu-id="2ffbc-139">Tokenizer</span></span> | <span data-ttu-id="2ffbc-140">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-140">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-141">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-141">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="2ffbc-142">ステマー</span><span class="sxs-lookup"><span data-stu-id="2ffbc-142">Stemmer</span></span> | <span data-ttu-id="2ffbc-143">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-143">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-144">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-144">No</span></span> |
| <span data-ttu-id="2ffbc-145">レンマタイザー</span><span class="sxs-lookup"><span data-stu-id="2ffbc-145">Lemmatizer</span></span> | <span data-ttu-id="2ffbc-146">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-146">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-147">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-147">No</span></span> |
| <span data-ttu-id="2ffbc-148">品詞のタグ付け</span><span class="sxs-lookup"><span data-stu-id="2ffbc-148">Part of speech tagging</span></span> | <span data-ttu-id="2ffbc-149">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-149">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-150">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-150">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="2ffbc-151">用語の出現頻度/逆文書頻度 (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-151">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="2ffbc-152">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-152">Yes (Spark MLlib)</span></span> | <span data-ttu-id="2ffbc-153">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-153">No</span></span> |
| <span data-ttu-id="2ffbc-154">文字列の類似性 &mdash; 編集距離の計算</span><span class="sxs-lookup"><span data-stu-id="2ffbc-154">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="2ffbc-155">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-155">Yes (Spark MLlib)</span></span> | <span data-ttu-id="2ffbc-156">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-156">No</span></span> |
| <span data-ttu-id="2ffbc-157">N グラム計算</span><span class="sxs-lookup"><span data-stu-id="2ffbc-157">N-gram calculation</span></span> | <span data-ttu-id="2ffbc-158">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-158">Yes (Spark MLlib)</span></span> | <span data-ttu-id="2ffbc-159">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-159">No</span></span> |
| <span data-ttu-id="2ffbc-160">ストップ ワードの削除</span><span class="sxs-lookup"><span data-stu-id="2ffbc-160">Stop word removal</span></span> | <span data-ttu-id="2ffbc-161">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-161">Yes (Spark MLlib)</span></span> | <span data-ttu-id="2ffbc-162">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-162">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="2ffbc-163">高レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="2ffbc-163">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="2ffbc-164">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="2ffbc-164">Azure HDInsight</span></span> | <span data-ttu-id="2ffbc-165">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="2ffbc-165">Microsoft Cognitive Services</span></span> |
| --- | --- | --- | 
| <span data-ttu-id="2ffbc-166">エンティティ/意図の識別と抽出</span><span class="sxs-lookup"><span data-stu-id="2ffbc-166">Entity/intent identification & extraction</span></span> | <span data-ttu-id="2ffbc-167">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-167">No</span></span> | <span data-ttu-id="2ffbc-168">はい (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-168">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |    
| <span data-ttu-id="2ffbc-169">トピック検出</span><span class="sxs-lookup"><span data-stu-id="2ffbc-169">Topic detection</span></span> | <span data-ttu-id="2ffbc-170">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-170">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-171">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-171">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="2ffbc-172">スペル チェック</span><span class="sxs-lookup"><span data-stu-id="2ffbc-172">Spell checking</span></span> | <span data-ttu-id="2ffbc-173">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-173">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-174">はい (Bing Spell Check API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-174">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="2ffbc-175">センチメント分析</span><span class="sxs-lookup"><span data-stu-id="2ffbc-175">Sentiment analysis</span></span> | <span data-ttu-id="2ffbc-176">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-176">Yes (Spark NLP)</span></span> | <span data-ttu-id="2ffbc-177">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-177">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="2ffbc-178">言語検出</span><span class="sxs-lookup"><span data-stu-id="2ffbc-178">Language detection</span></span> | <span data-ttu-id="2ffbc-179">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-179">No</span></span> | <span data-ttu-id="2ffbc-180">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-180">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="2ffbc-181">英語以外の複数言語のサポート</span><span class="sxs-lookup"><span data-stu-id="2ffbc-181">Supports multiple languages besides English</span></span> | <span data-ttu-id="2ffbc-182">いいえ </span><span class="sxs-lookup"><span data-stu-id="2ffbc-182">No</span></span> | <span data-ttu-id="2ffbc-183">はい (API によって異なる)</span><span class="sxs-lookup"><span data-stu-id="2ffbc-183">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="2ffbc-184">関連項目</span><span class="sxs-lookup"><span data-stu-id="2ffbc-184">See also</span></span>

[<span data-ttu-id="2ffbc-185">自然言語処理</span><span class="sxs-lookup"><span data-stu-id="2ffbc-185">Natural language processing</span></span>](../scenarios/natural-language-processing.md)