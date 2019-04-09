---
title: Azure Databricks での Spark モデルのバッチ スコアリング
description: Azure Databricks を使用して、スケジュールに従って Apache Spark 分類モデルをバッチ スコアリングするスケーラブルなソリューションを構築します。
author: njray
ms.date: 02/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: cba8d272ddbdbf2c2da94f68b288e9fb79be7de2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887813"
---
# <a name="batch-scoring-of-spark-machine-learning-models-on-azure-databricks"></a>Azure Databricks での Spark 機械学習モデルのバッチ スコアリング

この参照アーキテクチャでは、Azure 向けに最適化された Apache Spark ベースの分析プラットフォームである Azure Databricks を使用して、スケジュールに従って Apache Spark 分類モデルをバッチ スコアリングするスケーラブルなソリューションを構築する方法を示します。 このソリューションは、他のシナリオのために一般化できるテンプレートとして使用できます。

このアーキテクチャのリファレンス実装は、 [GitHub][github] で入手できます。

![Azure Databricks での Spark モデルのバッチ スコアリング](./_images/batch-scoring-spark.png)

**シナリオ**: 資産が多い業界の企業が、予期しない機械の不具合に関連するコストとダウンタイムを最小化することを望んでいます。 機械から収集される IoT データを使用して、予知メンテナンス モデルを作成できます。 企業は、このモデルを使用して、部品を積極的に保守し、不具合が発生する前にそれらを修繕できます。 機械部品を最大限使用できるようにすることで、コストを制御し、ダウンタイムを短縮できます。

予知メンテナンス モデルでは、機械からデータが収集され、部品の不具合例の履歴が保持されます。 その後、このモデルを使用して、部品の現在の状態を監視し、特定の部品で近い将来不具合が発生するかどうかを予測します。 一般的なユース ケースとモデリング アプローチについては、「[予測メンテナンス ソリューションのための Azure AI ガイド][ai-guide]」をご覧ください。

この参照アーキテクチャは、機械部品の新しいデータの存在によってトリガーされるワークロード用に設計されています。 処理には次の手順が含まれます。

1. 外部データ ストアから Azure Databricks データ ストアにデータを取り込みます。

2. データをトレーニング データセットに変換した後、Spark MLlib モデルを構築して、機械学習モデルをトレーニングします。 MLlib は、最も一般的な機械学習アルゴリズムと、Spark のデータ スケーラビリティ機能を活用するように最適化されたユーティリティで構成されています。

3. データをスコアリング データセットに変換することで、部品の不具合を予測 (分類) するトレーニング済みモデルを適用します。 Spark MLLib モデルを使用してデータをスコア付けします。

4. 処理後の使用のために、Databricks データ ストアに結果を格納します。

これらのタスクのそれぞれを実行するノートブックが  [GitHub][github] に提供されています。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャでは、順を追って実行される一連の[ノートブック][notebooks]に基づく [Azure Databricks][databricks] の中にその全体が含まれるデータ フローが定義されます。 これは、次のコンポーネントで構成されます。

**[データ ファイル][github]**。 参照実装では、5 つの静的なデータ ファイルに含まれているシミュレートされたデータ セットが使用されます。

**[取り込み][notebooks]**。 データ取り込みノートブックでは、入力データ ファイルを Databricks データ セットのコレクションにダウンロードします。 現実のシナリオでは、IoT デバイスからのデータが、Databricks にアクセス可能なストレージ (Azure SQL Server や Azure Blob Storage など) にストリーミングされます。 Databricks では、複数の[データソース][data-sources]がサポートされています。

**トレーニング パイプライン**。 このノートブックでは、取り込まれたデータから分析データ セットを作成する特徴エンジニアリング ノートブックが実行されます。 その後、[Apache Spark MLlib][mllib] スケーラブル機械学習ライブラリを使用して機械学習モデルをトレーニングするモデル構築ノートブックが実行されます。

**スコアリング パイプライン**。 このノートブックでは、取り込まれたデータからスコアリング データ セットを作成する特徴エンジニアリング ノートブックが実行され、スコアリング ノートブックが実行されます。 スコアリング ノートブックでは、トレーニング済みの [Spark MLlib][mllib-spark] モデルを使用して、スコアリング データ セットの観測結果から予測が生成されます。 予測は、結果ストアに格納され、Databricks データ ストアの新しいデータ セットになります。

**スケジューラ**。 スケジュールされた Databricks [ジョブ][job]によって、Spark モデルによるバッチ スコアリングが処理されます。 このジョブでは、スコアリング パイプライン ノートブックが実行され、スコアリング データ セットを構築するための詳細と結果データ セットの格納場所を指定する変数の引数が、ノートブックのパラメーターを通して渡されます。

このシナリオは、パイプライン フローとして構築されます。 各ノートブックは、操作 (取り込み、特徴エンジニアリング、モデルの構築、およびモデルのスコアリング) の各操作をバッチ設定で実行するように最適化されています。 これを実現するため、特徴エンジニアリング ノートブックは、トレーニング、較正、テスト、またはスコアリング操作のすべてにとって一般的なデータ セットを生成するように設計されています。 このシナリオでは、これらの操作に対して一時分割戦略を使用するため、ノートブックのパラメーターを使用して日付範囲フィルターが設定されます。

このシナリオでは、バッチ パイプラインを作成しているため、パイプライン ノートブックの出力を調査するための一連の調査ノートブックがオプションとして用意されています。 これらは、GitHub リポジトリで見つけることができます。

- `1a_raw-data_exploring`
- `2a_feature_exploration`
- `2b_model_testing`
- `3b_model_scoring_evaluation`

## <a name="recommendations"></a>Recommendations

Databricks は、お客様がご自身でトレーニングしたモデルを読み込んでデプロイし、新しいデータを使用して予測を行えるようにセットアップされています。 このシナリオで Databricks を使用したのは、次のような利点が追加されるためです。

- Azure Active Directory 資格情報を使用したシングル サインオンのサポート。
- 運用パイプライン用のジョブを実行するジョブ スケジューラ。
- コラボレーション、ダッシュボード、REST API による完全対話型ノートブック。
- 任意のサイズにスケーリングできる制限のないクラスター。
- 高度なセキュリティ、ロールベースのアクセス制御、および監査ログ。

Azure Databricks サービスを操作するには、Web ブラウザーまたは[コマンド ライン インターフェイス][cli] (CLI) で Databricks [ワークスペース][workspace] インターフェイスを使用します。 Databricks CLI には、Python 2.7.9 から 3.6 をサポートするプラットフォームからアクセスします。

この参照実装では、[ノートブック][notebooks]を使用して、タスクを順を追って実行します。 各ノートブックでは、入力データと同じデータ ストアに、中間データ成果物 (トレーニング、テスト、スコアリング、または結果データ セット) が格納されます。 目標は、お客様固有のユース ケースで、必要に応じて簡単に使用できるようにすることです。 実際には、お使いのデータ ソースをノートブックの Azure Databricks インスタンスに接続して、ストレージとの読み取りと書き込みを直接的に実行します。

必要に応じて、Databricks ユーザー インターフェイス、データ ストア、または Databricks [CLI][cli] を通してジョブの実行を監視できます。 Databricks が提供する[イベント ログ][log]やその他の[メトリック][metrics]を使用して、クラスターを監視します。

## <a name="performance-considerations"></a>パフォーマンスに関する考慮事項

Azure Databricks クラスターは、既定で自動スケールが可能になっているため、実行時に Databricks によってお客様のジョブの特性を構成するワーカーが動的に再割り当てされます。 パイプラインの特定の部分で、その計算負荷が他の部分よりも高くなる場合があります。 Databricks では、ジョブのこれらのフェーズ中にワーカーが追加されます (不要になったときには削除されます)。 ワークロードと一致するクラスターを自身でプロビジョニングする必要がないため、自動スケールによって高い[クラスター使用率][cluster]を簡単に達成できるようになります。

さらに、[Azure Data Factory][adf] と Azure Databricks を使用して、さらに複雑なスケジュールのパイプラインを開発できます。

## <a name="storage-considerations"></a>ストレージに関する考慮事項

この参照実装では、単純化するために、データは Databricks ストレージ内に直接格納されます。 ただし、運用環境の設定では、[Azure Blob Storage][blob] などのクラウド データ ストレージにデータを格納できます。 [Databricks][databricks-connect] では、Azure Data Lake Store、Azure SQL Data Warehouse、Azure Cosmos DB、Apache Kafka、および Hadoop もサポートしています。

## <a name="cost-considerations"></a>コストに関する考慮事項

Azure Databricks は、優れた Spark サービスであり、関連するコストがあります。 さらに、Databricks の[価格レベル][pricing]には、Standard と Premium があります。

このシナリオでは、Standard 価格レベルで十分です。 ただし、特定のアプリケーションで、大規模なワークロードまたは Databricks の対話型ダッシュボードを処理するために、自動的にスケーリングされるクラスターが必要な場合は、Premium レベルの使用によってコストが増加する可能性があります。

このソリューションのノートブックは、Databricks 固有のパッケージを削除するという最小限の編集によって、任意の Spark ベースのプラットフォームで実行できます。 次のさまざまな Azure プラットフォーム向けの類似するソリューションを参照してください。

- [Azure Machine Learning Studio 上の Python][python-aml]
- [SQL Server R Services][sql-r]
- [Azure Data Science Virtual Machine 上の PySpark][py-dvsm]

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照アーキテクチャをデプロイするには、 [GitHub][github] リポジトリで説明されている手順に従って、Azure Databricks で Spark モデルをスコアリングするためのスケーラブルなソリューションを構築します。

## <a name="related-architectures"></a>関連するアーキテクチャ

オフラインの事前計算済みのスコアを使用する[リアルタイム レコメンデーション システム][recommendation]を構築するために Spark を使用する参照アーキテクチャも用意されています。 これらのレコメンデーション システムは、スコアがバッチ処理される一般的なシナリオです。

[adf]: https://azure.microsoft.com/blog/operationalize-azure-databricks-notebooks-using-data-factory/
[ai-guide]: /azure/machine-learning/team-data-science-process/cortana-analytics-playbook-predictive-maintenance
[blob]: https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[cli]: https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html
[cluster]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[databricks]: /azure/azure-databricks/
[databricks-connect]: /azure/azure-databricks/databricks-connect-to-data-sources
[data-sources]: https://docs.databricks.com/spark/latest/data-sources/index.html
[github]: https://github.com/Azure/BatchSparkScoringPredictiveMaintenance
[job]: https://docs.databricks.com/user-guide/jobs.html
[log]: https://docs.databricks.com/user-guide/clusters/event-log.html
[metrics]: https://docs.databricks.com/user-guide/clusters/metrics.html
[mllib]: https://docs.databricks.com/spark/latest/mllib/index.html
[mllib-spark]: https://docs.databricks.com/spark/latest/mllib/index.html#apache-spark-mllib
[notebooks]: https://docs.databricks.com/user-guide/notebooks/index.html
[pricing]: https://azure.microsoft.com/en-us/pricing/details/databricks/
[python-aml]: https://gallery.azure.ai/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1
[py-dvsm]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-using-PySpark
[recommendation]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[sql-r]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1
[workspace]: https://docs.databricks.com/user-guide/workspace.html
