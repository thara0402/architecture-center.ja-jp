---
title: "自然言語処理技術の選択"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Azure での自然言語処理技術の選択

自由形式テキストの処理は、通常は検索をサポートする目的でテキストの段落を含むドキュメントに対して実行されますが、センチメント分析、トピック検出、言語検出、キー フレーズ抽出、ドキュメントの分類などの他の自然言語処理 (NLP) タスクの実行にも使用されます。 この記事では、NLP タスクのサポートで機能するテクノロジの選択に重点を置いて説明します。

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>NLP サービスを選ぶときのオプション

Azure では、次のサービスに自然言語処理 (NLP) 機能があります。

- [Spark および Spark MLlib を使用する Azure HDInsight](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft Cognitive Services](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>主要な選択条件

選択を絞り込むには、以下の質問に答えることから開始します。

- 事前構築済みのモデルを使用しますか。 回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。

- テキスト データの大規模なコーパスに対してカスタム モデルをトレーニングする必要がありますか。 回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。

- トークン化、ステミング、レンマ化、単語の出現頻度/逆文書頻度 (TF/IDF) のような低レベルの NLP 機能が必要ですか。 回答が「はい」の場合は、Azure HDInsight を Spark MLlib および Spark NLP と共に使用することを検討します。

- エンティティと意図の識別、トピック検出、スペル チェック、センチメント分析などのシンプルで高レベルの NLP 機能が必要ですか。 回答が「はい」の場合は、Microsoft Cognitive Services によって提供される API の使用を検討します。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。  

### <a name="general-capabilities"></a>一般的な機能

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| サービスとして事前トレーニング済みモデルを提供 | いいえ  | [はい] |
| REST API | [はい] | [はい] |
| プログラミング | Python、Scala、Java | C#、Java、Node.js、Python、PHP、Ruby |
| 大規模なデータ セットとサイズの大きいドキュメントの処理のサポート | [はい] | いいえ  |

### <a name="low-level-natural-language-processing-capabilities"></a>低レベルの自然言語処理機能

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| トークナイザー | はい (Spark NLP) | はい (Linguistic Analysis API) |
| ステマー | はい (Spark NLP) | いいえ  |
| レンマタイザー | はい (Spark NLP) | いいえ  |
| 品詞のタグ付け | はい (Spark NLP) | はい (Linguistic Analysis API) |
| 用語の出現頻度/逆文書頻度 (TF/IDF) | はい (Spark MLlib) | いいえ  |
| 文字列の類似性 &mdash; 編集距離の計算 | はい (Spark MLlib) | いいえ  |
| N グラム計算 | はい (Spark MLlib) | いいえ  |
| ストップ ワードの削除 | はい (Spark MLlib) | いいえ  |

### <a name="high-level-natural-language-processing-capabilities"></a>高レベルの自然言語処理機能

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| エンティティ/意図の識別と抽出 | いいえ  | はい (Language Understanding Intelligent Service (LUIS) API) |    
| トピック検出 | はい (Spark NLP) | はい (Text Analytics API) |
| スペル チェック | はい (Spark NLP) | はい (Bing Spell Check API) |
| センチメント分析 | はい (Spark NLP) | はい (Text Analytics API) |
| 言語検出 | いいえ  | はい (Text Analytics API) |
| 英語以外の複数言語のサポート | いいえ  | はい (API によって異なる) |

## <a name="see-also"></a>関連項目

[自然言語処理](../scenarios/natural-language-processing.md)