---
title: 分析データ ストアの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 3cf7dc533cc6ae3e6d7e2326852b585da8613e18
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428875"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Azure で使用する分析データ ストアの選択

[ビッグ データ](../big-data/index.md) アーキテクチャでは、分析ツールを使用してクエリを実行できる構造化された形式の処理済みデータを提供する分析データ ストアが必要になることがよくあります。 ホットパス データとコールドパス データの両方のクエリ処理をサポートする分析データ ストアは、まとめてサービス レイヤーまたはデータ サービス ストレージと呼ばれます。

サービス レイヤーは、ホットパスとコールドパス両方の処理済みデータを扱います。 [ラムダ アーキテクチャ](../big-data/index.md#lambda-architecture)の場合、サービス レイヤーは、増分処理されたデータを格納する_スピード サービス_ レイヤーと、バッチ処理された出力を含む_バッチ サービス_ レイヤーに分割されます。 サービス レイヤーは、短い待機時間のランダム読み取りを強力にサポートする必要があります。 このストアにデータのバッチ読み込みを行うと、望ましくない遅延が生じるので、速度レイヤー用のデータ ストレージはランダム書き込みもサポートする必要があります。 一方、バッチ レイヤーのデータ ストレージは、ランダム書き込みをサポートする必要はありませんが、代わりにバッチ書き込みをサポートする必要があります。

すべてのデータ ストレージ タスクに最適な 1 つのデータ管理方法はありません。 各データ管理ソリューションは、異なるタスクに合わせて最適化されています。 ほとんどの実際のクラウド アプリケーションとビッグ データ プロセスには、さまざまなデータ ストレージ要件があり、多くの場合、複数のデータ ストレージ ソリューションを組み合わせて使用します。

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>分析データ ストアを選択する場合のオプション

Azure にはデータ サービス ストレージのオプションがいくつかあり、必要に応じて選択できます。

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL Database](/azure/sql-database/)
- [Azure VM の SQL Server](/sql/sql-server/sql-server-technical-documentation)
- [HDInsight 上の HBase/Phoenix](/azure/hdinsight/hbase/apache-hbase-overview)
- [HDInsight 上の Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

これらのオプションは、タスクの種類に合わせて最適化されたさまざまなデータベース モデルを提供しています。

- [キー/値](https://msdn.microsoft.com/library/dn313285.aspx#sec7)データベースは、各キー値に対して 1 つのシリアル化されたオブジェクトを保持します。 指定されたキー値に対して 1 つの項目を取得し、項目の他のプロパティに基づいてクエリを実行する必要がない、大量のデータを格納する場合に適しています。
- [ドキュメント](https://msdn.microsoft.com/library/dn313285.aspx#sec8) データベースは、値が*ドキュメント*であるキー/値データベースです。 この文脈での "ドキュメント" とは、名前付きフィールドと値のコレクションです。 通常、データベースには XML、YAML、JSON、BSON などの形式でデータが格納されますが、プレーンテキストを使用することもできます。 ドキュメント データベースでは、非キー フィールドに対してクエリを実行できます。また、クエリをより効率的にするためにセカンダリ インデックスを定義できます。 そのため、ドキュメント データベースは、ドキュメント キーの値よりも複雑な基準に基づいてデータを取得する必要があるアプリケーションに適しています。 たとえば、製品 ID、顧客 ID、顧客名などのフィールドに対してクエリを実行することができます。
- [列ファミリ](https://msdn.microsoft.com/library/dn313285.aspx#sec9) データベースは、データ ストレージを、列ファミリと呼ばれる関連する列のコレクションに構成するキー/値データ ストアです。 たとえば、国勢調査データベースには、個人の名前 (姓、名、ミドルネーム) 用の列のグループ、個人の住所のグループ、個人のプロフィール情報 (生年月日、性別のデータ) のグループが含まれる可能性があります。 このデータベースは、各列ファミリを個別のパーティションに格納し、さらに 1 人のすべてのデータと同じキーへの関連付けを維持することができます。 アプリケーションは、エンティティのすべてのデータを読み取らずに、単一の列ファミリを読み取ることができます。
- [グラフ](https://msdn.microsoft.com/library/dn313285.aspx#sec10) データベースは、情報をオブジェクトとリレーションシップのコレクションとして格納します。 グラフ データベースは、オブジェクトのネットワークとオブジェクト間のリレーションシップにまたがるクエリを効率的に実行することができます。 たとえば、人事データベースではオブジェクトは従業員の可能性があります。また、"佐藤さんのために直接的または間接的に働いているすべての従業員を検索する" などのクエリを簡単にすることもできます。

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- データのホット パスに対応できるサービス ストレージは必要ですか。 "はい" の場合、スピード サービス レイヤーに合わせて最適化されたオプションに絞り込みます。

- クエリが複数のプロセスやノードに自動的に分散される、超並列処理 (MPP) のサポートは必要ですか。 "はい" の場合、クエリのスケールアウトをサポートするオプションを選択します。

- リレーショナル データ ストアを使用したいですか。 "はい" の場合、リレーショナル データベース モデルを使用するオプションに絞り込みます。 ただし、一部の非リレーショナル ストアはクエリの SQL 構文をサポートしており、PolyBase などのツールを使用して非リレーショナル データ ストアに対してクエリを実行することができます。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

| | SQL Database | SQL Data Warehouse | HDInsight 上の HBase/Phoenix | HDInsight 上の Hive LLAP | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| マネージド サービスか | [はい] | [はい] | はい <sup>1</sup> | はい <sup>1</sup> | [はい] | [はい] |
| プライマリ データベース モデル | リレーショナル (列ストア インデックスを使用する場合の列の形式) | 単票形式ストレージのリレーショナル テーブル | ワイド カラム ストア | Hive/In-Memory | 表形式/MOLAP セマンティック モデル | ドキュメント ストア、グラフ、キー値ストア、ワイド カラム ストア |
| SQL 言語のサポート | [はい] | [はい] | はい ([Phoenix](https://phoenix.apache.org/) JDBC ドライバーを使用) | [はい] | いいえ  | [はい] |
| スピード サービス レイヤーに合わせて最適化 | はい <sup>2</sup> | いいえ  | 可能  | [はい] | いいえ  | [はい] |

[1] 手動構成とスケーリングを使用。

[2] メモリ最適化テーブルとハッシュ インデックスまたは非クラスター化インデックスを使用。
 
### <a name="scalability-capabilities"></a>スケーラビリティ機能

|                                                  | SQL Database | SQL Data Warehouse | HDInsight 上の HBase/Phoenix | HDInsight 上の Hive LLAP | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| 高可用性のための冗長リージョン サーバー |     [はい]      |        はい         |            [はい]             |           いいえ            |           いいえ             |    [はい]    |
|             クエリのスケールアウトをサポート             |      いいえ       |        可能          |            はい             |          はい           |           はい           |    [はい]    |
|          動的スケーラビリティ (スケールアップ)          |     [はい]      |        [はい]         |             いいえ              |           いいえ            |           可能            |    [はい]    |
|        データのメモリ内キャッシュをサポート        |     [はい]      |        [はい]         |             いいえ              |          可能            |           [はい]           |    いいえ      |

### <a name="security-capabilities"></a>セキュリティ機能

| | SQL Database | SQL Data Warehouse | HDInsight 上の HBase/Phoenix | HDInsight 上の Hive LLAP | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Authentication  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD | ローカル / Azure AD <sup>1</sup> | ローカル / Azure AD <sup>1</sup> | Azure AD | アクセスの制御 (IAM) によるデータベース ユーザー/Azure AD |
| 保存データの暗号化 | はい <sup>2</sup> | はい <sup>2</sup> | はい <sup>1</sup> | はい <sup>1</sup> | [はい] | [はい] |
| 行レベルのセキュリティ | [はい] | いいえ  | はい <sup>1</sup> | はい <sup>1</sup> | はい (モデル内のオブジェクトレベルのセキュリティを使用) | いいえ  |
| ファイアウォールをサポート | [はい] | [はい] | はい <sup>3</sup> | はい <sup>3</sup> | [はい] | [はい] |
| 動的データ マスク | [はい] | いいえ  | はい <sup>1</sup> | はい * | いいえ  | いいえ  |

[1] [ドメイン参加済み HDInsight クラスター](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)を使用する必要があります。

[2] 保存データの暗号化と暗号化の解除には、Transparent Data Encryption (TDE) を使用する必要があります。

[3] Azure Virtual Network 内で使用する場合。 「[Azure Virtual Network を使用した Azure HDInsight の拡張](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)」を参照してください。
