---
title: Cognitive Services テクノロジの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 055769188fbd6742b94094ee18766293812849fa
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Microsoft Cognitive Services テクノロジの選択

Microsoft Cognitive Services は、人工知能 (AI) アプリケーションおよびデータ フローで使用できるクラウドベースの API です。 アプリケーションで使用する準備ができているトレーニング済みモデルが用意されているので、ユーザー側でデータの準備やモデルのトレーニングを行う必要はありません。 Cognitive Services は、Microsoft の AI and Research チームによって開発され、最新のディープ ラーニング アルゴリズムを活用しています。 このサービスは、HTTP REST インターフェイスを使用して使用されます。 さらに、多くの一般的なアプリケーション開発フレームワークで SDK を使用できます。

Cognitive Services の内容:

* テキスト分析
* Computer Vision
* ビデオ分析
* 音声認識と生成
* 自然言語の理解
* インテリジェント検索

主な利点:

* 最小限の開発作業で最先端の AI サービスを利用。
* HTTP REST インターフェイス経由でアプリケーションに簡単に統合。
* Azure Data Lake Analytics で Cognitive Services を使用するための組み込みのサポート。

考慮事項:

* Web 経由でのみ利用可能です。 通常、インターネット接続が必要です。 ただし、Custom Vision Service は例外です。このサービスでは、デバイスおよび IoT エッジで、トレーニングされたモデルを予測のためにエクスポートできます。
* さまざまなカスタマイズがサポートされていますが、利用できるサービスがすべての予測分析要件に適合しない可能性があります。

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Cognitive Services の中から選ぶ場合のオプション
Azure には数十種類の Cognitive Services があります。 最新の一覧は、サポートする機能領域別に分類されたディレクトリを確認してください。
- [Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Speech](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Knowledge](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Search](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [言語](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- どのような種類のデータを扱っていますか。 使用している入力データの種類に基づいてオプションを絞り込んでください。 たとえば、入力がテキストの場合は、入力種類がテキストのサービスから選択します。 

- モデルをトレーニングするためのデータは持っていますか。 "はい" の場合、精度とパフォーマンスを向上するために、用意したデータを使用して基になるモデルをトレーニングすることができるカスタム サービスを検討してください。 

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。 

### <a name="uses-prebuilt-models"></a>事前構築済みのモデルを使用

|                                                   |             入力の種類              |                                                                                主な長所                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                Text Analytics API                 |                テキスト                 |                                                       センチメントとトピックを評価して、ユーザーが求めるものを理解できます。                                                        |
|                Entity Linking API                 |                テキスト                 |                                               固有表現認識とあいまいさ排除でアプリのデータ リンクを強化します。                                               |
| Language Understanding Intelligent Service (LUIS) |                テキスト                 |                                                          ユーザーが入力したコマンドをアプリが理解できるようにします。                                                          |
|                 QnA Maker Service                 |                テキスト                 |                                             FAQ 形式の情報から会話形式のナビゲーションしやすい回答を抽出します。                                              |
|              Linguistic Analysis API              |                テキスト                 |                                                            複雑な言語の概念を単純化し、テキストを解析します。                                                             |
|           Knowledge Exploration Service           |                テキスト                 |                                          自然言語入力による構造化データの対話型検索を可能にします。                                          |
|              Web Language Model API               |                テキスト                 |                                                         Web 規模のデータで学習した予測言語モデルを使用します。                                                         |
|              Academic Knowledge API               |                テキスト                 |                                        Bing によって設定されている Microsoft Academic Graph の豊富な学術コンテンツを利用します。                                         |
|               Bing Autosuggest API                |                テキスト                 |                                                        アプリにインテリジェントな自動提案機能を追加します。                                                        |
|               Bing Spell Check API                |                テキスト                 |                                                             アプリでのスペル ミスを検出して修正します。                                                             |
|                Translator Text API                |                テキスト                 |                                                                           機械翻訳。                                                                            |
|                Recommendations API                |                テキスト                 |                                                             顧客が欲しい品物を予測して推奨します。                                                              |
|              Bing Entity Search API               |       テキスト (Web 検索クエリ)       |                                                           Web からエンティティ情報を特定して拡張します。                                                           |
|               Bing Image Search API               |       テキスト (Web 検索クエリ)       |                                                                            画像を検索します。                                                                             |
|               Bing News Search API                |       テキスト (Web 検索クエリ)       |                                                                             ニュースを検索します。                                                                              |
|               Bing Video Search API               |       テキスト (Web 検索クエリ)       |                                                                            ビデオを検索します。                                                                             |
|                Bing Web Search API                |       テキスト (Web 検索クエリ)       |                                                        何十億もの Web ドキュメントから、より優れた検索結果を入手します。                                                        |
|                  Bing Speech API                  |           テキストまたは音声            |                                                                  音声からテキストへの変換とテキストから音声への変換。                                                                   |
|              Speaker Recognition API              |               音声                |                                                       音声を使用して個々の話者を識別および認証します。                                                        |
|               Translator Speech API               |               音声                |                                                                   リアルタイムの音声翻訳を実行します。                                                                   |
|                Computer Vision API                |    画像 (またはビデオのフレーム)    | 画像から実用的な情報を抽出し、自動的に写真の説明を作成し、タグを導き出し、有名人を認識し、テキストを抽出し、正確なサムネイルを作成します。 |
|                 Content Moderator                 |        テキスト、画像、またはビデオ        |                                                               画像、テキスト、ビデオの自動モデレート。                                                                |
|                    Emotion API                    | 画像 (人物を含む写真) |                                                              人物のさまざまな感情を特定します。                                                               |
|                     Face API                      | 画像 (人物を含む写真) |                                                       写真に含まれる顔を検出、識別、分析、グループ化、タグ付けします。                                                       |
|                   Video Indexer                   |                ビデオ                |                        感情、トランスクリプト音声、音声翻訳、顔や感情の認識、キーワードの抽出などのビデオの詳細情報。                         |

### <a name="trained-with-custom-data-you-provide"></a>用意したカスタム データによるトレーニング

| | 入力の種類 | 主な長所 |
| --- | --- | --- |
| Custom Vision Service | 画像 (またはビデオのフレーム) | 独自のコンピューター ビジョン モデルをカスタマイズします。 |
| Custom Speech Service | 音声 | 話し方、背景ノイズ、ボキャブラリといった音声認識の障壁をなくします。 | 
| Custom Decision Service | Web コンテンツ (RSS フィードなど) | 機械学習を使用して、ホーム ページに適したコンテンツを自動的に選択します |
| Bing Custom Search API | テキスト (Web 検索クエリ) | 商用グレードの検索ツール。 |

