---
title: 機械学習テクノロジの選択
description: 機械学習モデルを構築、デプロイ、および管理するための選択肢を比較します。 ご自身のソリューションのためにどの Microsoft 製品を選択するかを決定します。
author: MikeWasson
ms.date: 03/06/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 1020e938a04c6a82e6cc831e6620ec17c9a10cc7
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503232"
---
# <a name="what-are-the-machine-learning-products-at-microsoft"></a>Microsoft の機械学習製品とは

機械学習は、コンピューターで既存のデータを使って、将来の動き、結果、傾向を予測できるデータ サイエンスの手法の 1 つです。 機械学習を使用することで、明示的にプログラムすることなく、コンピューターが学習します。

機械学習ソリューションは対話形式で構築され、次のような個別のフェーズがあります。

- データの準備
- モデルの実験とトレーニング
- トレーニングされたモデルのデプロイ
- デプロイされたモデルの管理

Microsoft では、機械学習モデルを準備、構築、デプロイ、および管理するためのさまざまな製品オプションを提供しています。 これらの製品を比較し、機械学習ソリューションを最も効率的に開発するために必要なものを選択します。

## <a name="cloud-based-options"></a>クラウドベースのオプション

Azure クラウドでの機械学習には、次の選択肢があります。

| クラウド&nbsp;の選択肢 | 説明 | この製品でできること |
|-|-|-|
| [Azure Machine Learning サービス](#azure-machine-learning-service) | 機械学習用のマネージド クラウド サービス  | Python と CLI を使用して Azure でモデルをトレーニング、デプロイ、および管理する |
| [Azure Machine Learning Studio](#azure-machine-learning-studio) | 機械学習用のドラッグ アンド ドロップ式のビジュアル インターフェース | 事前に構成されたアルゴリズムを使用してモデルを構築、実験、およびデプロイする |

事前構築済みの AI と機械学習のモデルを使用する場合、[Azure Cognitive Services](#azure-cognitive-services) を使用すると、インテリジェントな機能を簡単にアプリケーションに追加することができます。

## <a name="on-premises-options"></a>オンプレミスの選択肢

オンプレミスでの機械学習には、次の選択肢があります。 クラウド上の仮想マシンでオンプレミスのサーバーを実行することもできます。

| オンプレミス&nbsp;の選択肢 | 説明 | この製品でできること |
|-|-|-|
| [SQL Server Machine Learning サービス](#sql-server-machine-learning-services) | SQL に組み込まれた分析エンジン | SQL Server の内部にモデルを構築およびデプロイする |
| [Microsoft Machine Learning Server](#microsoft-machine-learning-server) | 予測分析のためのスタンドアロンのエンタープライズ サーバー | 前処理されたデータでモデルを構築およびデプロイする |

## <a name="development-platforms-and-tools"></a>開発プラットフォームおよびツール

次の開発プラットフォームとツールを機械学習に利用できます。

| プラットフォーム/ツール | 説明 | この製品でできること |
|-|-|-|
| [Azure Data Science Virtual Machine](#azure-data-science-virtual-machine) | データ サイエンス ツールがプレインストールされた仮想マシン | 事前に構成された環境で機械学習ソリューションを開発する |
| [Azure Databricks](#azure-databricks) | Spark ベースの分析プラットフォーム | モデルとデータ ワークフローを構築およびデプロイする |
| [ML.NET](#mlnet) | オープン ソースでクロスプラットフォームの機械学習 SDK | .NET アプリケーション用の機械学習ソリューションを開発する |
| [Windows ML](#windows-ml) | Windows 10 機械学習プラットフォーム | Windows 10 デバイス上のトレーニング済みのモデルを評価する |

## <a name="azure-machine-learning-service"></a>Azure Machine Learning サービス

[Azure Machine Learning service](/azure/machine-learning/service/overview-what-is-azure-ml.md) は、機械学習モデルを大規模にトレーニング、デプロイ、および管理するために使用されるフル マネージド クラウド サービスです。 オープンソース テクノロジを完全にサポートしているため、TensorFlow、PyTorch、scikit-learn などの数万のオープンソース Python パッケージを使用できます。 データの探索と変換、モデルのトレーニングとデプロイを容易にするために、[Azure Notebooks](https://notebooks.azure.com/)、[Jupyter Notebook](http://jupyter.org)、[Azure Machine Learning for Visual Studio Code](https://aka.ms/vscodetoolsforai) などの豊富なツールも利用できます。 Azure Machine Learning サービスには、モデルの生成とチューニングを簡単に、効率よく、かつ正確に自動化する機能が含まれています。

Azure Machine Learning service は、Python と CLI を使用して機械学習モデルをクラウド規模でトレーニング、デプロイ、および管理する場合に使用します。

[無料版または有料版の Azure Machine Learning service](http://aka.ms/AMLFree) をお試しください。

|||
|-|-|
|**Type**                   |クラウドベースの機械学習ソリューション|
|**サポートされている言語**    |Python|
|**機械学習のフェーズ**|データの準備<br>モデル トレーニング<br>Deployment<br>管理|
|**主な利点**           |スクリプトと実行履歴を一元管理して、モデルのバージョンを簡単に比較できます。<br/><br/>クラウドまたはエッジ デバイスへのモデルのデプロイと管理が簡単です。|
|**考慮事項**         |モデル管理モデルについて、ある程度の知識が必要です。|

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

[Azure Machine Learning Studio](/azure/machine-learning/studio/) は、事前に構築された機械学習アルゴリズムを使用して、モデルを簡単かつ迅速に構築、テスト、およびデプロイするために使用できる対話型のビジュアル ワークスペースを提供します。 Machine Learning Studio でモデルを Web サービスとして公開すれば、カスタム アプリや BI ツール (Excel など) からそのモデルを簡単に利用することができます。
プログラミングは必要ありません。対話型のキャンバス上のデータセットと分析モジュールを接続することによって機械学習モデルを構成した後、それを数回のクリックでデプロイします。

Machine Learning Studio は、コードを必要とすることなくモデルを開発およびデプロイする場合に使用します。

有料と無料のオプションで提供されている [Azure Machine Learning Studio](https://studio.azureml.net/?selectAccess=true&o=2&target=_blank) をお試しください。

|||
|-|-|
|**Type**                   |クラウドベースでドラッグアンドドロップ式の機械学習ソリューション|
|**サポートされている言語**    |Python、R|
|**機械学習のフェーズ**|データの準備<br>モデル トレーニング<br>Deployment<br>管理|
|**主な利点**           |対話型のビジュアル インターフェイスにより、最小限のコードで機械学習モデリングを行うことができます。<br/><br/>データ探索用の組み込み Jupyter Notebook。<br/><br/>トレーニング済みモデルを Azure Web サービスとして直接デプロイできます。|
|**考慮事項**         |スケーラビリティが限定的です。 トレーニング データセットの最大サイズは 10 GB です。<br/><br/>オンラインのみです。 オフライン開発環境はありません。|

## <a name="azure-cognitive-services"></a>Azure Cognitive Services

[Azure Cognitive Services](/azure/cognitive-services/welcome) は、自然なコミュニケーション方法を使用するアプリを構築できるようにする一連の API です。 これらの API を使用すると、数行のコードを書くだけで、見る、聞く、話す、理解する、ユーザーのニーズを解釈することができるアプリを作成できます。 次のようなインテリジェントな機能をアプリに簡単に追加できます。

- 感情とセンチメントの検出
- 視覚と音声認識
- 言語の理解 (LUIS)
- 知識と検索

Cognitive Services は、デバイスやプラットフォームにまたがるアプリを開発する場合に使用します。 API は、継続的に品質改善され、設定も簡単です。

|||
|-|-|
|**Type**                   |インテリジェントなアプリケーションを構築するための API|
|**サポートされている言語**    |サービスに依存する多くのオプション|
|**機械学習のフェーズ**|Deployment|
|**主な利点**           |事前トレーニング済みのモデルを使用してアプリケーションに機械学習機能を組み込む。<br/><br/>視覚や音声を使用した自然なコミュニケーション手段に対応したさまざまなモデル。|
|**考慮事項**         |モデルは事前トレーニング済みで、カスタマイズできない。|

## <a name="sql-server-machine-learning-services"></a>SQL Server Machine Learning サービス

[SQL Server Microsoft Machine Learning サービス](https://docs.microsoft.com/sql/advanced-analytics/r/r-services)は、SQL Server データベース内のリレーショナル データに対する R と Python での統計分析、データ視覚化、および予測分析を追加します。 Microsoft の R および Python ライブラリには、SQL Server で大規模に、並行して実行できる高度なモデリングおよび機械学習アルゴリズムが含まれています。

SQL Server Machine Learning サービスは、SQL Server 内のリレーショナル データに対する組み込みの AI および予測分析が必要な場合に使用します。

|||
|-|-|
|**Type**                   |オンプレミスでのリレーショナル データの予測分析|
|**サポートされている言語**    |Python、R|
|**機械学習のフェーズ**|データの準備<br>モデル トレーニング<br>Deployment|
|**主な利点**           |データベース関数に予測ロジックをカプセル化して、データ層ロジックに簡単に含めることができます。|
|**考慮事項**         |アプリケーションのデータ層として SQL Server データベースが想定されます。|

## <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

[Microsoft Machine Learning Server](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server) は、R プロセスと Python プロセスの並列および分散ワークロードをホストして管理するためのエンタープライズ サーバーです。 Microsoft Machine Learning Server は Linux、Windows、Hadoop、および Apache Spark 上で動作するほか、[HDInsight](https://azure.microsoft.com/services/hdinsight/r-server/) でも使用できます。 これにより、[RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)、[revoscalepy](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package)、および [MicrosoftML パッケージ](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package)を使用して構築されたソリューション用の実行エンジンが提供されると共に、オープンソースの R と Python が高パフォーマンスの分析、統計分析、機械学習、および非常に大規模なデータセットのサポートで拡張されます。 この機能は、サーバーと共にインストールされる独自のパッケージによって提供されます。 開発用には、[R Tools for Visual Studio](https://www.visualstudio.com/vs/rtvs/) や [Python Tools for Visual Studio](https://www.visualstudio.com/vs/python/) などの IDE を使用することができます。

Microsoft Machine Learning Server は、R と Python で構築されたモデルをサーバー上で構築して運用化するか、あるいは R と Python のトレーニングを Hadoop または Spark クラスター上で大規模に配布する必要がある場合に使用します。

|||
|-|-|
|**Type**                   |予測分析のためのオンプレミスのエンタープライズ サーバー|
|**サポートされている言語**    |Python、R|
|**機械学習のフェーズ**|モデル トレーニング<br>Deployment|
|**主な利点**           |高いスケーラビリティ。|
|**考慮事項**         |企業内で Machine Learning Server をデプロイし、管理する必要があります。|

## <a name="azure-data-science-virtual-machine"></a>Azure Data Science Virtual Machine

[Azure Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) は、特にデータ サイエンスを実行するために構築された Microsoft Azure クラウド上のカスタマイズされた仮想マシン環境です。 多くのよく使われるデータ サイエンス ツールや他のツールが事前にインストールおよび構成されており、高度な分析のためのインテリジェントなアプリケーションの構築をすぐに始めることができます。

Data Science Virtual Machine は、Azure Machine Learning サービスのターゲットとしてサポートされています。
Windows と Linux Ubuntu の両方のバージョンで使用できます (Linux CentOS では Azure Machine Learning サービスはサポートされていません)。
特定のバージョン情報および含まれている内容の一覧については、[Azure Data Science Virtual Machine の概要](/azure/machine-learning/data-science-virtual-machine/overview.md)に関するページを参照してください。

Data Science VM は、単一のノードでジョブを実行またはホストする必要がある場合や、 1 台のマシンで処理をリモートでスケールアップする必要がある場合に使用します。

|||
|-|-|
|**Type**                   |データ サイエンス用のカスタマイズされた仮想マシン環境|
|**主な利点**           |データ サイエンス ツールおよびフレームワークのインストール、管理、トラブルシューティングに要する時間を短縮できます。<br/><br/>よく使用されるツールとフレームワークの最新バージョンがすべて含まれています。<br/><br/>仮想マシンのオプションには、集中的なデータ モデリングのための GPU 機能を備えたスケーラブルなイメージが含まれます。|
|**考慮事項**         |オフラインのときは仮想マシンにアクセスできません。<br/><br/>仮想マシンの実行には Azure の料金が発生するため、必要な場合にのみ実行するように注意する必要があります。|

## <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) は、Microsoft Azure クラウド サービス プラットフォーム用に最適化された Apache Spark ベースの分析プラットフォームです。 Databricks は、ワンクリックでのセットアップ、効率化されたワークフロー、およびデータ サイエンティスト、データ エンジニア、ビジネス アナリストの間のコラボレーションを可能にする対話型ワークスペースを実現するために、Azure に統合されています。
データのクエリ実行、視覚化、およびモデル化を行うには、Web ベースのノートブックで Python、R、Scala、および SQL コードを使用します。

Databricks は、Apache Spark 上で機械学習ソリューションの構築に関して共同作業する場合に使用します。

|||
|-|-|
|**Type**                   |Apache Spark ベースの分析プラットフォーム|
|**サポートされている言語**    |Python、R、Scala、SQL|
|**機械学習のフェーズ**|データ クエリ<br>モデル トレーニング|

## <a name="mlnet"></a>ML.NET

[ML.NET](https://docs.microsoft.com/dotnet/machine-learning/) は、カスタムの機械学習ソリューションを構築し、それを .NET アプリケーションに統合できるようにする、無料のオープンソースおよびクロスプラットフォームの機械学習フレームワークです。

ML.NET は、機械学習ソリューションを .NET アプリケーションに統合する場合に使用します。

|||
|-|-|
|**Type**                   |カスタムの機械学習アプリケーションを開発するためのオープン ソース フレームワーク|
|**サポートされている言語**    |.NET|

## <a name="windows-ml"></a>Windows ML

[Windows ML](https://docs.microsoft.com/windows/uwp/machine-learning/) インターフェース エンジンを使用すると、アプリケーションでトレーニング済みの機械学習モデルを使用して、トレーニング済みのモデルを Windows 10 デバイス上でローカルで評価できます。

Windows ML は、Windows アプリケーション内でトレーニング済みの機械学習モデルを使用する場合に使用します。

|||
|-|-|
|**Type**                   |Windows デバイスでのトレーニング済みモデルの推論エンジン|
|**サポートされている言語**    |C#/C++、JavaScript|

## <a name="next-steps"></a>次の手順

- Microsoft から入手できるすべての人工知能 (AI) 開発製品の詳細については、「[Microsoft AI プラットフォーム](https://www.microsoft.com/ai)」を参照してください。
- AI ソリューションを開発する方法のトレーニングについては、「[Microsoft AI スクール](https://aischool.microsoft.com/learning-paths)」を参照してください。
