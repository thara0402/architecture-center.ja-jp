---
title: データ分析とレポート テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: data-analytics
ms.openlocfilehash: 72b889e2fe0d862ab1ff280cea76c2880b0fadc4
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481918"
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Azure でのデータ分析とテクノロジの選択

ほとんどのビッグ データ ソリューションの目的は、分析とレポートによってデータに関する実用的な情報を提供することにあります。 これには、事前に構成されたレポートと視覚化や、対話型データ探索が含まれます。

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>データ分析テクノロジを選ぶときのオプション

<!-- markdownlint-disable MD026 -->

Azure での分析、視覚化、レポートには、ニーズに応じていくつかのオプションがあります。

- [Power BI](/power-bi/)
- [Jupyter Notebook](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin Notebook](https://zeppelin.apache.org/)
- [Microsoft Azure Notebooks](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/) はビジネス分析ツールのスイートです。 何百ものデータ ソースに接続でき、アド ホック分析に使用できます。 現在使用可能なデータ ソースについては、[こちらの一覧](/power-bi/desktop-data-sources)をご覧ください。 [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) は、追加のライセンスを必要とせずに独自のアプリケーション内で Power BI を統合する場合に使用します。

組織では、Power BI を使用してレポートを作成し、組織に公開できます。 すべてのユーザーは、ガバナンスと[セキュリティが組み込まれた](/power-bi/service-admin-power-bi-security)パーソナライズされたダッシュボードを作成できます。 Power BI は、[Azure Active Directory](/azure/active-directory/) (Azure AD) を使用して、Power BI サービスにログインするユーザーを認証し、ユーザーが認証を必要とするリソースへのアクセスを試みるたびに Power BI ログイン資格情報を使用します。

### <a name="jupyter-notebooks"></a>Jupyter Notebooks

[Jupyter Notebook](https://jupyter.readthedocs.io/en/latest/index.html) は、データ サイエンティストが Python、Scala、または R コードおよびマークダウン テキストを含む "*ノートブック*" ファイルを作成できるブラウザー ベースのシェルを提供し、コードと結果を 1 つのドキュメントで共有および文書化して効果的に共同作業できるようにします。

Spark や Hadoop など、さまざまな HDInsight クラスターのほとんどは、データと対話し、処理するジョブを送信するために、[Jupyter Notebook で事前構成](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels)されています。 使用する HDInsight クラスターの種類に応じて、コードの解釈と実行用に 1 つまたは複数のカーネルが提供されます。 たとえば、HDInsight 上の Spark クラスターは、Spark エンジンを使用して Python または Scala コードを実行するために選べる Spark 関連のカーネルを提供します。

Jupyter Notebook では、Power BI などの BI/レポート ツールでより高度な視覚化を構築する前にデータを分析、視覚化、処理するための優れた環境が提供されます。

### <a name="zeppelin-notebooks"></a>Zeppelin Notebook

[Zeppelin Notebook](https://zeppelin.apache.org/) は、ブラウザー ベース シェルのもう 1 つのオプションであり、機能は Jupyter に似ています。 一部の HDInsight クラスターは [Zeppelin Notebook で事前構成](/azure/hdinsight/spark/apache-spark-zeppelin-notebook)されています。 ただし、[HDInsight 対話型クエリ](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP) クラスターを使用する場合、[Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) は現在、対話型 Hive クエリの実行に使用できる唯一のノートブックです。 また、[ドメイン参加済み HDInsight クラスター](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)を使用する場合、Zeppelin Notebook は、ノートブックおよび基になる Hive テーブルへのアクセスを制御するために異なるユーザー ログインを割り当てることができる唯一の種類です。

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com/) は、オンライン Jupyter Notebook ベースのサービスであり、データ サイエンティストがクラウドベースのライブラリで Jupyter Notebook を作成、実行、共有できるようにします。 Azure Notebooks は Python 2、Python 3、F# および R の実行環境を提供し、ggplot、matplotlib、bokeh、seaborn など、データの視覚化用の複数のグラフ ライブラリを提供します。

クラスターの既定のストレージ アカウントに接続される HDInsight クラスターで実行される Jupyter Notebook とは異なり、Azure Notebooks ではデータは提供されません。 オンライン ソースからのデータのダウンロード、Azure Blob や Table Storage との対話、SQL データベースへの接続、Azure Data Factory のコピー ウィザードを使用したデータの読み込みなど、さまざまな方法で[データを読み込む](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb)必要があります。

主な利点:

- 無料のサービスで、Azure サブスクリプションは必要ありません。
- Jupyter やサポートする R または Python ディストリビューションをローカルにインストールする必要はありません。ブラウザーだけを使用します。
- 独自のオンライン ライブラリを管理し、任意のデバイスからアクセスできます。
- ノートブックをコラボレーターと共有できます。

考慮事項:

- オフラインのときは、ノートブックにアクセスできません。
- 無料のノートブック サービスの制限付き処理機能は、大規模または複雑なモデルのトレーニングに不十分な場合があります。

## <a name="key-selection-criteria"></a>主要な選択条件

選択を絞り込むには、以下の質問に答えることから開始します。

- 多数のデータ ソースに接続して、ドメイン全体に分散したデータのレポートを作成する一元的な場所を提供する必要がありますか。 その場合は、数百のデータ ソースに接続できるオプションを選びます。

- 動的視覚化を外部 Web サイトまたはアプリケーションに埋め込みますか。 その場合は、埋め込み機能を提供するオプションを選びます。

- オフライン中に視覚化とレポートをデザインしますか。 答えが「はい」の場合は、オフライン機能があるオプションを選びます。

- 大規模または複雑な AI モデルをトレーニングしたり非常に大規模なデータ セットを操作したりするために高い処理能力が必要ですか。 答えが「はい」の場合は、ビッグ データ クラスターに接続できるオプションを選びます。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

<!-- markdownlint-disable MD033 -->

| | Power BI | Jupyter Notebooks | Zeppelin Notebook | Microsoft Azure Notebooks |
| --- | --- | --- | --- | --- |
| 高度な処理のためのビッグ データ クラスターへの接続 | [はい] | はい | はい | いいえ  |
| 管理されたサービス | [はい] | はい <sup>1</sup> | はい <sup>1</sup> | [はい] |
| 数百のデータ ソースへの接続 | [はい] | いいえ  | いいえ  | いいえ  |
| オフライン機能 | はい <sup>2</sup> | いいえ  | いいえ  | いいえ  |
| 埋め込み機能 | [はい] | いいえ  | いいえ  | いいえ  |
| データの自動更新 | [はい] | いいえ  | いいえ  | いいえ  |
| 多数のオープン ソース パッケージへのアクセス | いいえ  | はい <sup>3</sup> | はい <sup>3</sup> | はい <sup>4</sup> |
| データ変換/クレンジング オプション | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/)、R | Python、R、Julia、Scala などの 40 言語 | Python、JDBC、R などの 20 を超えるインタープリター | Python、F#、R |
| 価格 | 無料の Power BI Desktop (作成) については、ホスティング オプションの[料金](https://powerbi.microsoft.com/pricing/)をご覧ください | 無料 | 無料 | 無料 |
| マルチユーザー コラボレーション | [はい](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | はい (共有または [JupyterHub](https://github.com/jupyterhub/jupyterhub) などのマルチユーザー サーバーを使用) | [はい] | はい (共有を使用) |

<!-- markdownlint-enable MD033 -->

[1] 管理される HDInsight クラスターの一部として使用する場合。

[2] Power BI Desktop を使用する場合。

[2] コミュニティから提供されたパッケージは [Maven リポジトリ](https://search.maven.org/)で検索できます。

[3] pip または conda を使用して Python パッケージをインストールできます。 R パッケージは CRAN または GitHub からインストールできます。 F# のパッケージは、[パケット依存関係マネージャー](https://fsprojects.github.io/Paket/)を使用して Nuget.org 経由でインストールできます。
