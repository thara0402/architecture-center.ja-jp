---
title: データ転送テクノロジの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 53dcf8a69ad8ae100dbdbb230a9280efd419342a
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252755"
---
# <a name="transferring-data-to-and-from-azure"></a>Azure との間のデータ転送

Azure との間でデータを転送するオプションはいくつかあり、必要に応じて選択できます。

## <a name="physical-transfer"></a>物理的転送

物理ハードウェアを使用して Azure にデータを転送する方法は、次の場合に適しています。

- ネットワークが低速、または信頼性が低い。
- ネットワーク帯域幅を追加するためにコストがかかる。
- 機密データを扱う場合、セキュリティまたは組織のポリシーで発信接続は許可されていない。 

主な懸案事項がデータの転送にかかる時間の場合、ネットワーク転送が実際に物理的転送よりも遅いかどうかを確認するテストを実行することができます。

物理的にデータを Azure に転送するには、主に 2 つのオプションがあります。
- **Azure Import/Export**。 [Azure Import/Export サービス](/azure/storage/common/storage-import-export-service)を使用すると、内部 SATA HDD または SDD を Azure データセンターに送付することで、大量のデータを Azure Blob Storage または Azure Files に安全に転送できます。 また、このサービスを使用して、Azure Storage のデータをハード ディスク ドライブに転送し、それらのドライブをオンプレミスの読み込みのために返送することもできます。

- **Azure Data Box**。 [Azure Data Box](https://azure.microsoft.com/services/storage/databox/) は、Microsoft が提供するアプライアンスです。Azure Import/Export サービスと同様に機能します。 Microsoft は、独自のセキュリティで保護された改ざん防止機能を持つ転送アプライアンスを提供しています。また、出荷の物流全体を管理しているので、ポータルで確認できます。 Azure Data Box サービスの利点の 1 つは使いやすさです。 複数のハード ドライブを購入して準備し、それぞれにファイルを転送する必要はありません。 Azure Data Box は、業界をリードする多数の Azure パートナーの支援を受けているので、パートナー製品からクラウドへのオフライン転送をシームレスに利用できます。 

## <a name="command-line-tools-and-apis"></a>コマンド ライン ツールと API

スクリプト化とプログラムによるデータ転送を行う場合は、これらのオプションを検討してください。

- **Azure CLI**。 [Azure CLI](/azure/hdinsight/hdinsight-upload-data#commandline) は、Azure サービスを管理し、Azure Storage にデータをアップロードすることができるクロスプラットフォーム ツールです。 

- **AzCopy**。 [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) または [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) のコマンドラインから AzCopy を使用すると、最適なパフォーマンスで Azure BLOB、File、および Table ストレージとの間で簡単にデータをコピーできます。 AzCopy は同時実行と並列処理をサポートし、中断された場合にコピー操作を再開することができます。 他のほとんどのオプションよりも高速です。 プログラムによるアクセスの場合、[Microsoft Azure Storage Data Movement Library](/azure/storage/common/storage-use-data-movement-library) は、AzCopy を強化するコア フレームワークです。 .NET Core ライブラリとして提供されています。 

- **PowerShell**。 [`Start-AzureStorageBlobCopy` PowerShell コマンドレット](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0)は、PowerShell に慣れている Windows 管理者向けのオプションです。  

- **AdlCopy**。 [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) を使用すると、Azure Storage Blob のデータを Data Lake Store にコピーできます。 2 つの Azure Data Lake Store アカウント間でデータをコピーすることもできます。 ただし、Data Lake Store から Storage Blob にデータをコピーするために使用することはできません。

- **Distcp**。 Data Lake Store にアクセスできる HDInsight クラスターがある場合、[Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp) などの Hadoop エコシステム ツールを使用し、HDInsight クラスター記憶域 (WASB) と Data Lake Store アカウントの間でデータをコピーできます。

- **Sqoop**。 [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) は Apache プロジェクトであり、Hadoop エコシステムの一部です。 すべての HDInsight クラスターにプリインストールされています。 HDInsight クラスターと、SQL、Oracle、MySQL などのリレーショナル データベース間でデータを転送できます。 Sqoop は、インポートとエクスポートを含む関連ツールのコレクションです。 Sqoop は、Azure Storage Blob または Data Lake Store に接続されているストレージを使用する HDInsight クラスターで動作します。

- **PolyBase**。 [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) は、T-SQL 言語を使用してデータベースの外部にあるデータにアクセスするテクノロジです。 SQL Server 2016 では、Hadoop の外部データに対してクエリを実行し、Azure Blob Storage からデータをインポート/エクスポートすることができます。 Azure SQL Data Warehouse では、Azure Blob Storage と Azure Data Lake Store からデータをインポート/エクスポートできます。 SQL Data Warehouse にデータをインポートする場合、現時点で PolyBase が最速の方法です。

- **Hadoop コマンド ライン**。 HDInsight クラスター ヘッド ノード上にデータがある場合、`hadoop -copyFromLocal` コマンドを使用して、Azure Storage Blob や Azure Data Lake Store などのクラスターに接続されたストレージにそのデータをコピーできます。 Hadoop コマンドを使用するには、まずヘッド ノードに接続する必要があります。 接続後は、ファイルをストレージにアップロードできます。

## <a name="graphical-interface"></a>グラフィカル インターフェイス

少数のファイルまたはデータ オブジェクトのみを転送し、プロセスを自動化する必要がない場合は、次のオプションを検討してください。

- **Azure Storage Explorer**。 [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) は、Azure ストレージ アカウントの内容を管理するためのクロスプラットフォーム ツールです。 BLOB、ファイル、キュー、テーブル、Azure Cosmos DB のエンティティをアップロード、ダウンロード、および管理できます。 BLOB ストレージと共に使用して BLOB とフォルダーを管理できるだけでなく、ローカル ファイル システムと BLOB ストレージ間、またはストレージ アカウント間で BLOB をアップロードおよびダウンロードすることができます。

- **Azure Portal** BLOB ストレージと Data Lake Store のいずれにも、ファイルを探索して新しいファイルを 1 つずつアップロードできる Web ベースのインターフェイスが用意されています。 ツールをインストールしたくない場合、またはファイルをすばやく探索するためや少数のファイルをアップロードするためだけにコマンドを発行したくない場合に適しています。

## <a name="data-pipeline"></a>データ パイプライン

**Azure Data Factory**。 [Azure Data Factory](/azure/data-factory/) は、複数の Azure サービス、オンプレミス、またはその 2 つの組み合わせの間でファイルを定期的に転送する場合に最適なマネージド サービスです。 Azure Data Factory を使用すると、さまざまなデータ ストアからデータを取り込むデータ駆動型ワークフロー (パイプラインと呼ばれます) を作成し、スケジュールを設定できます。 そのデータは、Azure HDInsight Hadoop、Spark、Azure Data Lake Analytics、Azure Machine Learning などのコンピューティング サービスを使って処理し、変換することができます。 データの移動とデータ変換を[調整](../technology-choices/pipeline-orchestration-data-movement.md)し、自動化するためのデータ駆動型ワークフローを作成します。

## <a name="key-selection-criteria"></a>主要な選択条件

データ転送のシナリオについて、次の質問に答えてニーズに適したシステムを選択してください。

- 非常に大量のデータを転送する必要はありますか。インターネット接続上でその処理を行う場合、処理時間が長すぎる、信頼性が低い、コストが高すぎるという問題はありますか。 "はい" の場合、物理的転送を検討してください。

- データ転送タスクをスクリプト化して再利用できるようにしたいですか。 "はい" の場合、コマンド ラインのオプションまたは Azure Data Factory のいずれかを選択します。

- 非常に大量のデータをネットワーク接続経由​​で転送する必要はありますか。 必要な場合、ビッグ データに合わせて最適化されたオプションを選択します。

- リレーショナル データベースとの間でデータを転送する必要はありますか。 "はい" の場合、1 つ以上のリレーショナル データベースをサポートするオプションを選択します。 この中には Hadoop クラスターが必要なオプションもあることに注意してください。

- 自動データ パイプラインまたはワークフロー オーケストレーションは必要ですか。 "はい" の場合、Azure Data Factory を検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="physical-transfer"></a>物理的転送

| | Azure Import/Export サービス | Azure Data Box |
| --- | --- | --- |
| フォーム ファクター | 内部 SATA HDD または SDD | セキュリティで保護された改ざん防止機能を持つ単一のハードウェア アプライアンス |
| Microsoft が出荷の物流を管理 | いいえ  | [はい] |
| パートナー製品との統合 | いいえ  | [はい] |
| カスタム アプライアンス | いいえ  | [はい] |

### <a name="command-line-tools"></a>コマンド ライン ツール

**Hadoop/HDInsight**

| | Distcp | Sqoop | Hadoop CLI |
| --- | --- | --- | --- |
| ビッグ データに合わせて最適化 | [はい] | はい |  [はい] |
| リレーショナル データベースへのコピー |  いいえ  | はい | いいえ  |
| リレーショナル データベースからのコピー |  いいえ  | はい | いいえ  |
| BLOB ストレージへのコピー |  [はい] | はい | [はい] |
| BLOB ストレージからのコピー | [はい] |  [はい] | いいえ  |
| Data Lake Store へのコピー | [はい] | はい | [はい] |
| Data Lake Store からのコピー | [はい] | [はい] | いいえ  |

**その他**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 互換性のあるプラットフォーム | Linux、OS X、Windows | Linux、Windows | Windows | Linux、OS X、Windows | SQL Server、Azure SQL Data Warehouse | 
| ビッグ データに合わせて最適化 | いいえ  | いいえ  | いいえ  | はい <sup>1</sup> | はい <sup>2</sup> |
| リレーショナル データベースへのコピー | いいえ  | いいえ  | いいえ  | いいえ  | [はい] | 
| リレーショナル データベースからのコピー | いいえ  | いいえ  | いいえ  | いいえ  | [はい] | 
| BLOB ストレージへのコピー | [はい] | はい | [はい] | いいえ  | [はい] | 
| BLOB ストレージからのコピー | [はい] | はい | はい | はい | [はい] |
| Data Lake Store へのコピー | いいえ  | いいえ  | 可能  | はい |  [はい] | 
| Data Lake Store からのコピー | いいえ  | いいえ  | 可能  | はい | [はい] | 


[1] AdlCopy は、Data Lake Analytics アカウントで使用するときのビッグ データの転送に合わせて最適化されています。

[2] [PolyBase のパフォーマンスを向上する](/sql/relational-databases/polybase/polybase-guide#performance)には、計算を Hadoop にプッシュし、[PolyBase スケールアウト グループ](/sql/relational-databases/polybase/polybase-scale-out-groups)を使用して、SQL Server インスタンスと Hadoop ノード間の並列データ転送を可能にします。

### <a name="graphical-interface-and-azure-data-factory"></a>グラフィカル インターフェイスと Azure Data Factory

| | Azure ストレージ エクスプローラー | Azure Portal * | Azure Data Factory |
| --- | --- | --- | --- |
| ビッグ データに合わせて最適化 | いいえ  | いいえ  | [はい] | 
| リレーショナル データベースへのコピー | いいえ  | いいえ  | [はい] |
| リレーショナル データベースへのコピー | いいえ  | いいえ  | [はい] |
| BLOB ストレージへのコピー | [はい] | いいえ  | [はい] |
| BLOB ストレージからのコピー | [はい] | いいえ  | [はい] |
| Data Lake Store へのコピー | いいえ  | いいえ  | [はい] |
| Data Lake Store からのコピー | いいえ  | いいえ  | [はい] |
| BLOB ストレージへのアップロード | [はい] | はい | [はい] |
| Data Lake Store へのアップロード | [はい] | はい | [はい] |
| データ転送の調整 | いいえ  | いいえ  | [はい] |
| カスタム データ変換 | いいえ  | いいえ  | [はい] |
| 価格モデル | 無料 | 無料 | 使用した分を支払う |

\* この場合の Azure Portal は、BLOB ストレージと Data Lake Store に Web ベースの探索ツールを使用することを意味します。

