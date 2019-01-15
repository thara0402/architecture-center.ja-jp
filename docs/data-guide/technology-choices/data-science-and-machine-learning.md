---
title: 機械学習テクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: e4b8ecb64afbacc8a4e24e9c6274455db0451d62
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112127"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Azure での機械学習テクノロジの選択

データ サイエンスと機械学習は、通常はデータ サイエンティストによって実行されるワークロードです。 スペシャリスト ツールを必要とし、そのツールの多くはデータ サイエンティストが実行する必要のある対話型のデータ探索およびモデリング タスクの種類専用に設計されています。

機械学習ソリューションは反復的に構築され、2 つのフェーズがあります。

- データの準備とモデリング。
- 予測サービスのデプロイと使用。

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>データの準備およびモデリング用のツールとサービス

データ サイエンティストは通常、Python または R で記述されたカスタム コードを使用してデータを操作することを好みます。一般に、このコードは繰り返し実行され、データ サイエンティストはそのコードを使用してデータのクエリと探索を実行し、関係の特定に役立つ視覚化と統計を生成します。 データ サイエンティストが使用できる R および Python 用の対話型環境は数多くあります。 特に人気があるのは **Jupyter Notebook** で、これが提供するブラウザー ベースのシェルを使用して、データ サイエンティストは R または Python のコードとマークダウン テキストを含む "*ノートブック*" ファイルを作成できます。 これにより、コードと結果を 1 つのドキュメントで共有および文書化して効果的に共同作業を行うことができます。

よく使用されるその他のツールは次のとおりです。

- **Spyder**:Anaconda Python ディストリビューションで提供される Python 用の対話型開発環境 (IDE)。
- **R Studio**:R プログラミング言語用の IDE。
- **Visual Studio Code**:Python に加え、一般に使用される機械学習および AI 開発用のフレームワークをサポートする軽量のクロスプラットフォーム コーディング環境。

これらのツールに加えて、データ サイエンティストは、コードおよびモデル管理を簡略化する Azure サービスを利用できます。

### <a name="azure-notebooks"></a>Azure Notebooks

Azure Notebooks は、オンライン Jupyter Notebook サービスであり、データ サイエンティストがクラウドベースのライブラリで Jupyter Notebook を作成、実行、共有できるようにします。

主な利点:

- 無料のサービスで、Azure サブスクリプションは必要ありません。
- Jupyter やサポートする R または Python ディストリビューションをローカルにインストールする必要はありません。ブラウザーだけを使用します。
- 独自のオンライン ライブラリを管理し、任意のデバイスからアクセスできます。
- ノートブックをコラボレーターと共有できます。

考慮事項:

- オフラインのときは、ノートブックにアクセスできません。
- 無料のノートブック サービスの制限付き処理機能は、大規模または複雑なモデルのトレーニングに不十分な場合があります。

### <a name="data-science-virtual-machine"></a>データ サイエンス仮想マシン

データ サイエンス仮想マシンは、R、Python、Jupyter Notebook、Visual Studio Code、Microsoft Cognitive Toolkit のような機械学習モデリング用ライブラリなど、データ サイエンティストがよく使用するツールとフレームワークを含む Azure 仮想マシン イメージです。 これらのツールは、インストールが複雑で時間がかかる場合があり、バージョン管理の問題につながることの多い多くの相互依存関係を含みます。 事前インストール済みのイメージがあると、データ サイエンティストは環境の問題のトラブルシューティングに費やす時間を短縮でき、実行する必要のあるデータの探索およびモデリング タスクに集中することができます。

主な利点:

- データ サイエンス ツールおよびフレームワークのインストール、管理、トラブルシューティングに要する時間を短縮できます。
- よく使用されるツールとフレームワークの最新バージョンがすべて含まれています。
- 仮想マシンのオプションには、集中的なデータ モデリングのための GPU 機能を備えたスケーラブルなイメージが含まれます。

考慮事項:

- オフラインのときは仮想マシンにアクセスできません。
- 仮想マシンの実行には Azure の料金が発生するため、必要な場合にのみ実行するように注意する必要があります。

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning は、機械学習の実験とモデルを管理するためのクラウドベース サービスです。 データの準備状況とモデリング トレーニング スクリプトを追跡する実験サービスが含まれており、イテレーション間でモデルのパフォーマンスを比較できるようにすべての実行履歴が保持されます。 データ サイエンティストは、Jupyter Notebook や Visual Studio Code などの任意のツールでスクリプトを作成してから、Azure でさまざまな[コンピューティング リソース](/azure/machine-learning/service/how-to-set-up-training-targets)にデプロイできます。

モデルは Web サービスとして、Docker コンテナー、Azure HDinsight の Spark、Microsoft Machine Learning Server、または SQL Server にデプロイできます。 Azure Machine Learning モデル管理サービスを使用すると、クラウド内、エッジ デバイス上、または企業全体にわたってモデルのデプロイを追跡および管理できます。

主な利点:

- スクリプトと実行履歴を一元管理して、モデルのバージョンを簡単に比較できます。
- ビジュアル エディターを使用して対話的にデータ変換できます。
- クラウドまたはエッジ デバイスへのモデルのデプロイと管理が簡単です。

考慮事項:

- モデル管理モデルについて、ある程度の知識が必要です。

### <a name="azure-batch-ai"></a>Azure Batch AI

Azure Batch AI を使用すると、機械学習の実験を並列に実行でき、GPU を搭載した仮想マシンのクラスター全体にわたって大規模なモデル トレーニングを実行できます。 Batch AI トレーニングを使用すると、Cognitive Toolkit、Caffe、Chainer、TensorFlow などのフレームワークを使用して、クラスター化された GPU にディープ ラーニング ジョブをスケールアウトできます。

Azure Machine Learning モデル管理を使用すると、Batch AI トレーニングのモデルをデプロイ、管理、監視できます。

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Azure Machine Learning Studio は、データ実験を作成し、機械学習モデルをトレーニングして、Azure の Web サービスとして公開するためのクラウドベースのビジュアル開発環境です。 そのビジュアル ドラッグ アンド ドロップ インターフェイスは、カスタムの R および Python ロジック、機械学習モデリング タスク用の確立されたさまざまな統計的アルゴリズムと手法、Jupyter Notebook の組み込みサポートをサポートしながら、データ サイエンティストおよびパワー ユーザーが機械学習ソリューションをすばやく作成できるようにします。

主な利点:

- 対話型のビジュアル インターフェイスにより、最小限のコードで機械学習モデリングを行うことができます。
- データ探索用の組み込み Jupyter Notebook。
- トレーニング済みモデルを Azure Web サービスとして直接デプロイできます。

考慮事項:

- スケーラビリティが限定的です。 トレーニング データセットの最大サイズは 10 GB です。
- オンラインのみです。 オフライン開発環境はありません。

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>機械学習モデルをデプロイするためのツールとサービス

データ サイエンティストが機械学習モデルを作成した後、通常はそのモデルをデプロイし、アプリケーションまたは他のデータ フローで使用する必要があります。 機械学習モデルにはいくつかの潜在的なデプロイ ターゲットがあります。

### <a name="spark-on-azure-hdinsight"></a>Azure HDInsight での Spark

Apache Spark には、機械学習モデル用のフレームワークおよびライブラリである Spark MLlib が含まれています。 Microsoft Machine Learning library for Spark (MMLSpark) には、Spark の予測モデルのディープ ラーニング アルゴリズムのサポートも用意されています。

主な利点:

- Spark は、大量の機械学習のプロセスに高いスケーラビリティを提供する分散型プラットフォームです。
- HDinsight の Spark にモデルを直接デプロイし、Azure Machine Learning モデル管理サービスを使用して管理することができます。

考慮事項:

- Spark は、実行されている時間全体に対して料金が発生する HDinsght クラスターで実行されます。 機械学習サービスをときどき使用するだけの場合は、不要なコストが発生することがあります。

### <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/) は、Apache Spark ベースの分析プラットフォームです。 "サービスとしての Spark" と考えることができます。 Azure プラットフォームで Spark を使用する最も簡単な方法です。 機械学習には、[MLFlow](https://www.mlflow.org/)、[Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html)、Apache Spark MLlib などを使用できます。 詳しくは、[Azure Databricks の機械学習](https://docs.azuredatabricks.net/spark/latest/mllib/index.html)に関するページを参照してください。

### <a name="web-service-in-a-container"></a>コンテナー内の Web サービス

Docker コンテナーで機械学習モデルを Python Web サービスとしてデプロイできます。 Azure またはエッジ デバイスにモデルをデプロイでき、操作するデータをローカルに使用できます。

主な利点:

- コンテナーは軽量で、一般にサービスをパッケージ化およびデプロイするためのコスト効果の高い方法です。
- エッジ デバイスにデプロイできるので、予測ロジックをデータの近くに移動することができます。

考慮事項:

- このデプロイ モデルは Docker コンテナーに基づいているため、この方法で Web サービスをデプロイする前に、このテクノロジについて理解しておく必要があります。

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

Microsoft Learning Server (旧称 Microsoft R Server) は、R と Python のコード用のスケーラブルなプラットフォームで、機械学習シナリオ用に設計されています。

主な利点:

- 高いスケーラビリティ。

考慮事項:

- 企業内で Machine Learning Server をデプロイし、管理する必要があります。

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server では R と Python がネイティブにサポートされるため、これらの言語で作成された機械学習モデルをデータベースに Transact-SQL 関数としてカプセル化できます。

主な利点:

- データベース関数に予測ロジックをカプセル化して、データ層ロジックに簡単に含めることができます。

考慮事項:

- アプリケーションのデータ層として SQL Server データベースが想定されます。

### <a name="azure-machine-learning-web-service"></a>Azure Machine Learning Web サービス

Azure Machine Learning Studio を使用して機械学習モデルを作成するときに、Web サービスとしてデプロイすることができます。 これは、HTTP で通信できる任意のクライアント アプリケーションから REST インターフェイスを通じて使用できます。

主な利点:

- 開発とデプロイが簡単です。
- 基本的な監視メトリックを備えた Web サービス管理ポータル。
- Azure Data Lake Analytics、Azure Data Factory、Azure Stream Analytics からの Azure Machine Learning Web サービスの呼び出しのサポートが組み込まれています。

考慮事項:

- Azure Machine Learning Studio を使用して作成されたモデルに対してのみ使用できます。
- Web ベースのアクセス専用で、トレーニング済みモデルをオンプレミスまたはオフラインで実行することはできません。
