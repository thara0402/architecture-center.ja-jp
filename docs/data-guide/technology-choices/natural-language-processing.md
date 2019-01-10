---
title: 自然言語処理技術の選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 699e01bc9905d02fc8ec1113039087189f6e8caf
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114115"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="a9433-102">Azure での自然言語処理技術の選択</span><span class="sxs-lookup"><span data-stu-id="a9433-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="a9433-103">自由形式テキストの処理は、通常は検索をサポートする目的でテキストの段落を含むドキュメントに対して実行されますが、センチメント分析、トピック検出、言語検出、キー フレーズ抽出、ドキュメントの分類などの他の自然言語処理 (NLP) タスクの実行にも使用されます。</span><span class="sxs-lookup"><span data-stu-id="a9433-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="a9433-104">この記事では、NLP タスクのサポートで機能するテクノロジの選択に重点を置いて説明します。</span><span class="sxs-lookup"><span data-stu-id="a9433-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="a9433-105">NLP サービスを選ぶときのオプション</span><span class="sxs-lookup"><span data-stu-id="a9433-105">What are your options when choosing an NLP service?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="a9433-106">Azure では、次のサービスに自然言語処理 (NLP) 機能があります。</span><span class="sxs-lookup"><span data-stu-id="a9433-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="a9433-107">Spark および Spark MLlib を使用する Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="a9433-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="a9433-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="a9433-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- [<span data-ttu-id="a9433-109">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="a9433-109">Microsoft Cognitive Services</span></span>](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a><span data-ttu-id="a9433-110">主要な選択条件</span><span class="sxs-lookup"><span data-stu-id="a9433-110">Key selection criteria</span></span>

<span data-ttu-id="a9433-111">選択を絞り込むには、以下の質問に答えることから開始します。</span><span class="sxs-lookup"><span data-stu-id="a9433-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="a9433-112">事前構築済みのモデルを使用しますか。</span><span class="sxs-lookup"><span data-stu-id="a9433-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="a9433-113">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="a9433-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="a9433-114">テキスト データの大規模なコーパスに対してカスタム モデルをトレーニングする必要がありますか。</span><span class="sxs-lookup"><span data-stu-id="a9433-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="a9433-115">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="a9433-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="a9433-116">トークン化、ステミング、レンマ化、単語の出現頻度/逆文書頻度 (TF/IDF) のような低レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="a9433-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="a9433-117">回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="a9433-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="a9433-118">エンティティと意図の識別、トピック検出、スペル チェック、センチメント分析などのシンプルで高レベルの NLP 機能が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="a9433-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="a9433-119">回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="a9433-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="a9433-120">機能のマトリックス</span><span class="sxs-lookup"><span data-stu-id="a9433-120">Capability matrix</span></span>

<span data-ttu-id="a9433-121">次の表は、機能の主な相違点をまとめたものです。</span><span class="sxs-lookup"><span data-stu-id="a9433-121">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="a9433-122">一般的な機能</span><span class="sxs-lookup"><span data-stu-id="a9433-122">General capabilities</span></span>

| | <span data-ttu-id="a9433-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="a9433-123">Azure HDInsight</span></span> | <span data-ttu-id="a9433-124">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="a9433-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="a9433-125">サービスとして事前トレーニング済みモデルを提供</span><span class="sxs-lookup"><span data-stu-id="a9433-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="a9433-126">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-126">No</span></span> | <span data-ttu-id="a9433-127">[はい]</span><span class="sxs-lookup"><span data-stu-id="a9433-127">Yes</span></span> |
| <span data-ttu-id="a9433-128">REST API</span><span class="sxs-lookup"><span data-stu-id="a9433-128">REST API</span></span> | <span data-ttu-id="a9433-129">[はい]</span><span class="sxs-lookup"><span data-stu-id="a9433-129">Yes</span></span> | <span data-ttu-id="a9433-130">[はい]</span><span class="sxs-lookup"><span data-stu-id="a9433-130">Yes</span></span> |
| <span data-ttu-id="a9433-131">プログラミング</span><span class="sxs-lookup"><span data-stu-id="a9433-131">Programmability</span></span> | <span data-ttu-id="a9433-132">Python、Scala、Java</span><span class="sxs-lookup"><span data-stu-id="a9433-132">Python, Scala, Java</span></span> | <span data-ttu-id="a9433-133">C#、Java、Node.js、Python、PHP、Ruby</span><span class="sxs-lookup"><span data-stu-id="a9433-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="a9433-134">大規模なデータ セットとサイズの大きいドキュメントの処理のサポート</span><span class="sxs-lookup"><span data-stu-id="a9433-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="a9433-135">[はい]</span><span class="sxs-lookup"><span data-stu-id="a9433-135">Yes</span></span> | <span data-ttu-id="a9433-136">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="a9433-137">低レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="a9433-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="a9433-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="a9433-138">Azure HDInsight</span></span> | <span data-ttu-id="a9433-139">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="a9433-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- |
| <span data-ttu-id="a9433-140">トークナイザー</span><span class="sxs-lookup"><span data-stu-id="a9433-140">Tokenizer</span></span> | <span data-ttu-id="a9433-141">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-142">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="a9433-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="a9433-143">ステマー</span><span class="sxs-lookup"><span data-stu-id="a9433-143">Stemmer</span></span> | <span data-ttu-id="a9433-144">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-145">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-145">No</span></span> |
| <span data-ttu-id="a9433-146">レンマタイザー</span><span class="sxs-lookup"><span data-stu-id="a9433-146">Lemmatizer</span></span> | <span data-ttu-id="a9433-147">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-148">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-148">No</span></span> |
| <span data-ttu-id="a9433-149">品詞のタグ付け</span><span class="sxs-lookup"><span data-stu-id="a9433-149">Part of speech tagging</span></span> | <span data-ttu-id="a9433-150">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-151">はい (Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="a9433-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="a9433-152">用語の出現頻度/逆文書頻度 (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="a9433-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="a9433-153">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="a9433-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="a9433-154">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-154">No</span></span> |
| <span data-ttu-id="a9433-155">文字列の類似性 &mdash; 編集距離の計算</span><span class="sxs-lookup"><span data-stu-id="a9433-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="a9433-156">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="a9433-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="a9433-157">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-157">No</span></span> |
| <span data-ttu-id="a9433-158">N グラム計算</span><span class="sxs-lookup"><span data-stu-id="a9433-158">N-gram calculation</span></span> | <span data-ttu-id="a9433-159">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="a9433-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="a9433-160">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-160">No</span></span> |
| <span data-ttu-id="a9433-161">ストップ ワードの削除</span><span class="sxs-lookup"><span data-stu-id="a9433-161">Stop word removal</span></span> | <span data-ttu-id="a9433-162">はい (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="a9433-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="a9433-163">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="a9433-164">高レベルの自然言語処理機能</span><span class="sxs-lookup"><span data-stu-id="a9433-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="a9433-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="a9433-165">Azure HDInsight</span></span> | <span data-ttu-id="a9433-166">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="a9433-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="a9433-167">エンティティ/意図の識別と抽出</span><span class="sxs-lookup"><span data-stu-id="a9433-167">Entity/intent identification & extraction</span></span> | <span data-ttu-id="a9433-168">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-168">No</span></span> | <span data-ttu-id="a9433-169">はい (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="a9433-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |
| <span data-ttu-id="a9433-170">トピック検出</span><span class="sxs-lookup"><span data-stu-id="a9433-170">Topic detection</span></span> | <span data-ttu-id="a9433-171">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-172">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="a9433-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="a9433-173">スペル チェック</span><span class="sxs-lookup"><span data-stu-id="a9433-173">Spell checking</span></span> | <span data-ttu-id="a9433-174">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-175">はい (Bing Spell Check API)</span><span class="sxs-lookup"><span data-stu-id="a9433-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="a9433-176">センチメント分析</span><span class="sxs-lookup"><span data-stu-id="a9433-176">Sentiment analysis</span></span> | <span data-ttu-id="a9433-177">はい (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="a9433-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="a9433-178">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="a9433-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="a9433-179">言語検出</span><span class="sxs-lookup"><span data-stu-id="a9433-179">Language detection</span></span> | <span data-ttu-id="a9433-180">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-180">No</span></span> | <span data-ttu-id="a9433-181">はい (Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="a9433-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="a9433-182">英語以外の複数言語のサポート</span><span class="sxs-lookup"><span data-stu-id="a9433-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="a9433-183">いいえ </span><span class="sxs-lookup"><span data-stu-id="a9433-183">No</span></span> | <span data-ttu-id="a9433-184">はい (API によって異なる)</span><span class="sxs-lookup"><span data-stu-id="a9433-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="a9433-185">関連項目</span><span class="sxs-lookup"><span data-stu-id="a9433-185">See also</span></span>

[<span data-ttu-id="a9433-186">自然言語処理</span><span class="sxs-lookup"><span data-stu-id="a9433-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
