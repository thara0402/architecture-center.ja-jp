---
title: データ ストレージ テクノロジの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b14611a2dc34bcb145cf420441795d4124e7baeb
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847211"
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Azure でのビッグ データ ストレージ テクノロジの選択

このトピックでは、[分析データ ストア](./analytical-data-stores.md)や[リアルタイムのストリーミング取り込み](./real-time-ingestion.md)とは対照的に、ビッグ データ ソリューション向けのデータ ストレージ オプション (具体的には、一括データ インジェストとバッチ処理用のデータ ストレージ) を比較します。

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Azure でデータ ストレージを選択するときのオプション

必要に応じて、Azure にデータの取り込むためのさまざまなオプションがあります。

**File Storage**

- [Azure Storage BLOB](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**NoSQL データベース**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HDInsight での HBase](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Azure Storage BLOB

Azure Storage は、高い可用性セキュリティ、耐久性、スケーラビリティ、および冗長性を備えた管理対象ストレージ サービスです。 メンテナンスや重大な問題には、Microsoft がお客様に代わって対処します。 Azure Storage は、Azure が提供する最大のユビキタス ストレージ ソリューションであり、多数のサービスとツールと連携させて使用できます。

データの格納に使用できるさまざまな Azure Storage サービスがあります。 多数のデータ ソースから BLOB を格納するための最も柔軟なオプションは、[BLOB ストレージ](/azure/storage/blobs/storage-blobs-introduction)です。 BLOB は、基本的にはファイルです。 それらは、画像、ドキュメント、HTML ファイル、仮想ハード ディスク (VHD) から、ログなどのビッグ データ、データベースのバックアップまで、ほぼすべてを格納できます。 BLOB は、フォルダーに似たコンテナーに格納されます。 コンテナーは、BLOB のセットをグループ化します。 ストレージ アカウントに含めることができるコンテナーの数には制限がなく、1 つのコンテナーに格納できる BLOB の数にも制限はありません。

Azure Storage は、柔軟性、高可用性、および低コストという理由で、ビッグ データと分析ソリューションに適した選択肢です。 さまざまなユース ケース用のホット、クール、およびアーカイブ ストレージ層を提供します。 詳細については、「[Azure Blob Storage: ホット、クール、アーカイブ ストレージ層](/azure/storage/blobs/storage-blob-storage-tiers)」を参照してください。

Azure Blob Storage は、Hadoop からアクセスできます (HDInsight から利用できます)。 HDInsight は、クラスターの既定のファイル システムとして Azure Storage 内の BLOB コンテナーを使用できます。 HDInsight のすべてのコンポーネントは、WASB ドライバーが提供する Hadoop 分散ファイル システム (HDFS) のインターフェイスを利用して、BLOB として格納された構造化データまたは非構造化データを直接操作できます。 Azure Blob Storage は、PolyBase 機能を使用して Azure SQL Data Warehouse 経由でアクセスできます。

Azure Storage を適切な選択肢にするその他の機能を次に示します。

- [複数の同時実行制御戦略](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [ディザスター リカバリーと高可用性のオプション](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [保存時の暗号化](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。
- [ロールベースのアクセス制御 (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security)。Azure Active Directory のユーザーとグループを使用してアクセスを制御します。

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/) は、ビッグ データの分析ワークロードに対応するエンタープライズ規模のハイパースケール リポジトリです。 Data Lake を使用すると、運用分析や調査分析を目的として任意のサイズ、種類、および取り込み速度のデータを 1 か所の[セキュリティで保護された](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity)場所にキャプチャできます。

Data Lake Store に格納できるアカウント サイズ、ファイル サイズ、またはデータ量に関する制限は設定されていません。 データは複数のコピーを作成して格納されるため、障害が発生しても保護されます。Data Lake でのデータの格納期間に制限はありません。 予期しない障害から保護するためのファイルの複数のコピーの作成に加え、Data Lake では、ファイルの一部を、多数の個別のストレージ サーバーに分散させます。 これにより、ファイルを並列に読み取ってデータ分析を実行する場合の読み取りスループットが向上します。

Data Lake Store には、Hadoop (HDInsight で使用可能) から、WebHDFS 互換の REST API を使用してアクセスできます。 個別のファイルまたは組み合わせたファイルのサイズが Azure Storage でサポートされるファイルのサイズを超える場合は、これを Azure Storage の代わりに使用することを検討できます。 ただし、Data Lake Store を HDInsight クラスターのプライマリ ストレージとして使用する場合に従う必要がある[パフォーマンス チューニング ガイドライン](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight)があり、[Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark)[Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive)[MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) および [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm) 用の特別なガイドラインがあります。 また、Data Lake Store の[リージョンでの可用性](https://azure.microsoft.com/regions/#services)を必ず確認してください。それは Azure Storage と同じ数のリージョンでは利用できず、HDInsight クラスターと同じリージョンに配置される必要があります。

Azure Data Lake Analytics と結合された Data Lake Store は、格納されたデータを分析できるように特別に設計されており、データ分析シナリオに合わせてパフォーマンスの調整が行われます。 Data Lake Store は、Azure SQL Data Warehouse の PolyBase 機能を使用してアクセスすることもできます。

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/) は、Microsoft のグローバル分散型マルチモデル データベースです。 Cosmos DB では、世界中のあらゆる場所で 99 パーセントのユーザーの待機時間が確実に 1 桁ミリ秒となります。また、明確でわかりやすい複数の整合性モデルでパフォーマンスを細かく調整することができ、マルチホーム機能により高可用性も保証されます。

Azure Cosmos DB はスキーマに依存しません。 それは、全データのインデックスを自動的に作成します。スキーマとインデックスの管理に対処する必要はありません。 またマルチモデルでもあり、ドキュメント、キー値、グラフ、列ファミリのデータ モデルにネイティブに対応しています。 

Azure Cosmos DB の機能:

- [geo レプリケーション](/azure/cosmos-db/distribute-data-globally)
- 世界規模での[スループットとストレージのエラスティック スケーリング](/azure/cosmos-db/partition-data)
- [明確に定義された 5 種類の整合性レベル](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HDInsight での HBase

[Apache HBase](http://hbase.apache.org/) は、オープン ソースの NoSQL データベースであり、Hadoop 上に構築され、Google BigTable をモデルにしています。 HBase は、大量の非構造化データと半構造化データに対するランダム アクセスと強力な一貫性を、列ファミリで整理されたスキーマなしのデータベースで実現します。

データはテーブルの行内に格納され、行内のデータは列ファミリによってグループ化されます。 HBase は、列や列内に格納されるデータの型を使用前に定義する必要がないという意味で、スキーマレスです。 オープン ソース コードは、直線的な拡張により何千ものノード上でペタバイト級のデータを扱うことができます。 また、Hadoop エコシステムの分散アプリケーションの利点であるデータの冗長性、バッチ処理などの機能を利用できます。

[HDInsight の実装](/azure/hdinsight/hbase/apache-hbase-overview)と HBase のスケールアウト アーキテクチャにより、テーブルの自動シャーディング、読み取りと書き込みの強力な一貫性、自動フェールオーバーなどが実現します。 また、メモリ内キャッシュを利用した読み取りと高スループットのストリーミングによる書き込みによって、パフォーマンスも拡張されています。 ほとんどの場合、[仮想ネットワークの内部に HBase クラスターを作成](/azure/hdinsight/hbase/apache-hbase-provision-vnet)して、他の HDInsight クラスターとアプリケーションがテーブルに直接アクセスできるようにします。

## <a name="key-selection-criteria"></a>主要な選択条件

選択を絞り込むには、以下の質問に答えることから開始します。

- あらゆる種類のテキストまたはバイナリ データ用の管理された高速のクラウド ベースのストレージが必要か。 必要な場合は、ファイル ストレージ オプションのいずれかを選択します。

- 並列分析ワークロードと高いスループット/IOPS 用に最適化されたファイル ストレージが必要か。 必要な場合は、分析ワークロードのパフォーマンスを調整するオプションを選択します。

- Schemaless データベースで非構造化または半構造化データを格納する必要があるか。 ある場合は、非リレーショナル オプションのいずれかを選択します。 インデックス作成とデータベース モデルのオプションを比較します。 格納する必要があるデータの種類によっては、プライマリ データベース モデルが最大の要素になることがあります。

- 在住しているリージョンでサービスを利用できるか。 各 Azure サービスの利用可能リージョンを確認します。 「[リージョン別の利用可能な製品](https://azure.microsoft.com/regions/services/)」を参照してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="file-storage-capabilities"></a>File Storage の機能

|  | Azure Data Lake Store | Azure Blob Storage コンテナー |
| --- | --- | --- |
| 目的 | ビッグ データ分析ワークロードに最適化されたストレージ |さまざまなストレージ シナリオに対応する汎用オブジェクト ストア |
| ユース ケース | バッチ、ストリーミング分析、および機械学習データ (ログ ファイル、IoT データ、クリック ストリーム、大規模なデータセットなど) | あらゆる種類のテキスト データまたはバイナリ データ (アプリケーション バックエンド、バックアップ データ、ストリーミング用メディア ストレージ、汎用データなど) |
| Structure | 階層型ファイル システム | フラットな名前空間を使用するオブジェクト ストア |
| 認証 | [Azure Active Directory ID](/azure/active-directory/active-directory-authentication-scenarios) | 共有シークレット ([アカウント アクセス キー](/azure/storage/common/storage-create-storage-account#manage-your-storage-account)と [Shared Access Signature キー)](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)、および[ロールベースのアクセス制御 (RBAC)](/azure/security/security-storage-overview) に基づく |
| 認証プロトコル | OAuth 2.0。 呼び出しには、Azure Active Directory によって発行された有効な JWT (JSON Web トークン) が含まれている必要があります。 | ハッシュベース メッセージ認証コード (HMAC)。 呼び出しには、HTTP 要求の一部に対する Base64 でエンコードされた SHA-256 ハッシュが含まれている必要があります。 |
| 承認 | POSIX アクセス制御リスト (ACL)。 Azure Active Directory ID に基づく ACL は、ファイルおよびフォルダー レベルで設定できます。 | アカウントレベルの承認には、[アカウント アクセス キー](/azure/storage/common/storage-create-storage-account#manage-your-storage-account)を使用します。 アカウント、コンテナー、または BLOB の承認には、[Shared Access Signature キー](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)を使用します。 |
| 監査 | 使用可能。  |使用可能 |
| 保存時の暗号化 | 透過的、サーバー側 | 透過、サーバー側。クライアント側の暗号化 |
| Developer SDK | .NET、Java、Python、Node.js | .Net、Java、Python、Node.js、C++、Ruby |
| 分析ワークロードのパフォーマンス | 並列分析ワークロードに最適化されたパフォーマンス。高いスループットと IOPS | 分析ワークロードに最適化されていません。 |
| サイズ制限 | アカウント サイズ、ファイル サイズ、ファイル数に制限はありません。 | 具体的な制限については、 [こちら](/azure/azure-subscription-service-limits#storage-limits) |
| geo 冗長 | ローカル冗長 (1 つの Azure リージョンにデータの複数のコピー) | ローカル冗長 (LRS)、geo 冗長 (GRS)、読み取りアクセス geo 冗長 (RA-GRS)。 詳細については、 [こちら](/azure/storage/common/storage-redundancy) をご覧ください。 |

### <a name="nosql-database-capabilities"></a>NoSQL データベースの機能

|                                    |                                           Azure Cosmos DB                                           |                                                             HDInsight での HBase                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       プライマリ データベース モデル       |                      ドキュメント ストア、グラフ、キー値ストア、ワイド カラム ストア                      |                                                             ワイド カラム ストア                                                              |
|         セカンダリ インデックス          |                                                 [はい]                                                 |                                                                     いいえ                                                                      |
|        SQL 言語のサポート        |                                                 [はい]                                                 |                                     はい ([Phoenix](http://phoenix.apache.org/) JDBC ドライバーを使用)                                      |
|            整合性             |                   強固、有界整合性制約、セッション、一貫性のあるプレフィックス、最終的                   |                                                                   Strong                                                                   |
| Azure Functions のネイティブ統合 |                        [はい](/azure/cosmos-db/serverless-computing-database)                        |                                                                     いいえ                                                                      |
|   自動的なグローバル分散    |                          [はい](/azure/cosmos-db/distribute-data-globally)                           | いいえ。最終的な整合性を指定して、リージョン間で[HBase クラスターのレプリケーションを構成可能](/azure/hdinsight/hbase/apache-hbase-replication) |
|           価格モデル            | 必要に応じて秒単位で課金され、弾力的にスケーラブルな要求ユニット (RU)。弾力的にスケーラブルなストレージ |                              HDInsight クラスターの分単位の料金 (ノードの水平スケーリング)、ストレージ                               |

