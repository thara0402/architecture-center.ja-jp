---
title: 自然言語処理技術の選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bac0318a587a944c104360eb31223cc8755c1860
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251772"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="480e1-102">Azure での自然言語処理技術の選択</span><span class="sxs-lookup"><span data-stu-id="480e1-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="480e1-103">自由形式テキストの処理は、通常は検索をサポートする目的でテキストの段落を含むドキュメントに対して実行されますが、センチメント分析、トピック検出、言語検出、キー フレーズ抽出、ドキュメントの分類などの他の自然言語処理 (NLP) タスクの実行にも使用されます。</span><span class="sxs-lookup"><span data-stu-id="480e1-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="480e1-104">この記事では、NLP タスクのサポートで機能するテクノロジの選択に重点を置いて説明します。</span><span class="sxs-lookup"><span data-stu-id="480e1-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="480e1-105">NLP サービスを選ぶときのオプション</span><span class="sxs-lookup"><span data-stu-id="480e1-105">What are your options when choosing an NLP service?</span></span>

<span data-ttu-id="480e1-106">Azure では、次のサービスに自然言語処理 (NLP) 機能があります。</span><span class="sxs-lookup"><span data-stu-id="480e1-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="480e1-107">Spark および Spark MLlib を使用する Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="480e1-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="480e1-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="480e1-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- [<span data-ttu-id="480e1-109">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="480e1-109">Microsoft Cognitive Services</span></span>](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a><span data-ttu-id="480e1-110">主要な選択条件</span><span class="sxs-lookup"><span data-stu-id="480e1-110">Key selection criteria</span></span>

<span data-ttu-id="480e1-111">選択を絞り込むには、以下の質問に答えることから開始します。</span><span class="sxs-lookup"><span data-stu-id="480e1-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="480e1-112">事前構築済みのモデルを使用しますか。</span><span class="sxs-lookup"><span data-stu-id="480e1-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="480e1-113">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="480e1-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="480e1-114">テキスト データの大規模なコーパスに対してカスタム モデルをトレーニングする必要がありますか。</span><span class="sxs-lookup"><span data-stu-id="480e1-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="480e1-115">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="480e1-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="480e1-116">トークン化、ステミング、レンマ化、単語の出現頻度/逆文書頻度 (TF/IDF) のような低レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="480e1-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="480e1-117">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="480e1-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="480e1-118">エンティティと意図の識別、トピック検出、スペル チェック、センチメント分析などのシンプルで高レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="480e1-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="480e1-119">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="480e1-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="480e1-120">機能のマトリックス</span><span class="sxs-lookup"><span data-stu-id="480e1-120">Capability matrix</span></span>

<span data-ttu-id="480e1-121">次の表は、機能の主な相違点をまとめたものです。</span><span class="sxs-lookup"><span data-stu-id="480e1-121">The following tables summarize the key differences in capabilities.</span></span>  

### <a name="general-capabilities"></a><span data-ttu-id="480e1-122">一般的な機能</span><span class="sxs-lookup"><span data-stu-id="480e1-122">General capabilities</span></span>

| | <span data-ttu-id="480e1-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="480e1-123">Azure HDInsight</span></span> | <span data-ttu-id="480e1-124">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="480e1-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="480e1-125">サービスとして事前トレーニング済みモデルを提供</span><span class="sxs-lookup"><span data-stu-id="480e1-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="480e1-126">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-126">No</span></span> | <span data-ttu-id="480e1-127">[はい]</span><span class="sxs-lookup"><span data-stu-id="480e1-127">Yes</span></span> |
| <span data-ttu-id="480e1-128">REST API</span><span class="sxs-lookup"><span data-stu-id="480e1-128">REST API</span></span> | <span data-ttu-id="480e1-129">[はい]</span><span class="sxs-lookup"><span data-stu-id="480e1-129">Yes</span></span> | <span data-ttu-id="480e1-130">[はい]</span><span class="sxs-lookup"><span data-stu-id="480e1-130">Yes</span></span> |
| <span data-ttu-id="480e1-131">プログラミング</span><span class="sxs-lookup"><span data-stu-id="480e1-131">Programmability</span></span> | <span data-ttu-id="480e1-132">Python、Scala、Java</span><span class="sxs-lookup"><span data-stu-id="480e1-132">Python, Scala, Java</span></span> | <span data-ttu-id="480e1-133">C#、Java、Node.js、Python、PHP、Ruby</span><span class="sxs-lookup"><span data-stu-id="480e1-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="480e1-134">大規模なデータ セットとサイズの大きいドキュメントの処理のサポート</span><span class="sxs-lookup"><span data-stu-id="480e1-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="480e1-135">[はい]</span><span class="sxs-lookup"><span data-stu-id="480e1-135">Yes</span></span> | <span data-ttu-id="480e1-136">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="480e1-137">低レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="480e1-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="480e1-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="480e1-138">Azure HDInsight</span></span> | <span data-ttu-id="480e1-139">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="480e1-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- | 
| <span data-ttu-id="480e1-140">トークナイザー</span><span class="sxs-lookup"><span data-stu-id="480e1-140">Tokenizer</span></span> | <span data-ttu-id="480e1-141">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-142">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="480e1-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="480e1-143">ステマー</span><span class="sxs-lookup"><span data-stu-id="480e1-143">Stemmer</span></span> | <span data-ttu-id="480e1-144">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-145">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-145">No</span></span> |
| <span data-ttu-id="480e1-146">レンマタイザー</span><span class="sxs-lookup"><span data-stu-id="480e1-146">Lemmatizer</span></span> | <span data-ttu-id="480e1-147">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-148">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-148">No</span></span> |
| <span data-ttu-id="480e1-149">品詞のタグ付け</span><span class="sxs-lookup"><span data-stu-id="480e1-149">Part of speech tagging</span></span> | <span data-ttu-id="480e1-150">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-151">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="480e1-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="480e1-152">用語の出現頻度/逆文書頻度 (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="480e1-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="480e1-153">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="480e1-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="480e1-154">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-154">No</span></span> |
| <span data-ttu-id="480e1-155">文字列の類似性 &mdash; 編集距離の計算</span><span class="sxs-lookup"><span data-stu-id="480e1-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="480e1-156">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="480e1-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="480e1-157">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-157">No</span></span> |
| <span data-ttu-id="480e1-158">N グラム計算</span><span class="sxs-lookup"><span data-stu-id="480e1-158">N-gram calculation</span></span> | <span data-ttu-id="480e1-159">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="480e1-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="480e1-160">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-160">No</span></span> |
| <span data-ttu-id="480e1-161">ストップ ワードの削除</span><span class="sxs-lookup"><span data-stu-id="480e1-161">Stop word removal</span></span> | <span data-ttu-id="480e1-162">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="480e1-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="480e1-163">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="480e1-164">高レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="480e1-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="480e1-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="480e1-165">Azure HDInsight</span></span> | <span data-ttu-id="480e1-166">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="480e1-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- | 
| <span data-ttu-id="480e1-167">エンティティ/意図の識別と抽出</span><span class="sxs-lookup"><span data-stu-id="480e1-167">Entity/intent identification & extraction</span></span> | <span data-ttu-id="480e1-168">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-168">No</span></span> | <span data-ttu-id="480e1-169">はい (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="480e1-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |    
| <span data-ttu-id="480e1-170">トピック検出</span><span class="sxs-lookup"><span data-stu-id="480e1-170">Topic detection</span></span> | <span data-ttu-id="480e1-171">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-172">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="480e1-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="480e1-173">スペル チェック</span><span class="sxs-lookup"><span data-stu-id="480e1-173">Spell checking</span></span> | <span data-ttu-id="480e1-174">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-175">はい (Bing Spell Check API)</span><span class="sxs-lookup"><span data-stu-id="480e1-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="480e1-176">センチメント分析</span><span class="sxs-lookup"><span data-stu-id="480e1-176">Sentiment analysis</span></span> | <span data-ttu-id="480e1-177">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="480e1-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="480e1-178">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="480e1-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="480e1-179">言語検出</span><span class="sxs-lookup"><span data-stu-id="480e1-179">Language detection</span></span> | <span data-ttu-id="480e1-180">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-180">No</span></span> | <span data-ttu-id="480e1-181">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="480e1-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="480e1-182">英語以外の複数言語のサポート</span><span class="sxs-lookup"><span data-stu-id="480e1-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="480e1-183">いいえ </span><span class="sxs-lookup"><span data-stu-id="480e1-183">No</span></span> | <span data-ttu-id="480e1-184">はい (API によって異なる)</span><span class="sxs-lookup"><span data-stu-id="480e1-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="480e1-185">関連項目</span><span class="sxs-lookup"><span data-stu-id="480e1-185">See also</span></span>

[<span data-ttu-id="480e1-186">自然言語処理</span><span class="sxs-lookup"><span data-stu-id="480e1-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
