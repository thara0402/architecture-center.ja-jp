---
title: 自動化されたエンタープライズ ビジネス インテリジェンス (BI)
titleSuffix: Azure Reference Architectures
description: Azure Data Factory と SQL Data Warehouse を使用して、Azure での抽出、読み込み、および変換 (ELT) ワークフローを自動化します。
author: MikeWasson
ms.date: 11/06/2018
ms.custom: seodec18
ms.openlocfilehash: d87583802496f8be85e44c896ae7d6a26306cffc
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120341"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>SQL Data Warehouse と Azure Data Factory を使用したエンタープライズ BI の自動化

この参照アーキテクチャは、[抽出 - 読み込み - 変換 (ELT)](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) パイプラインで段階的に読み込む方法を示しています。 Azure Data Factory を使用して ELT パイプラインを自動化します。 パイプラインは、オンプレミスの SQL Server データベースから SQL Data Warehouse に最新の OLTP データを段階的に移動します。 トランザクション データは、分析のために表形式モデルに変換されます。

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2Gnz2]

このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。

![SQL Data Warehouse と Azure Data Factory を使用して自動化されたエンタープライズ向け BI のアーキテクチャ ダイアグラム](./images/enterprise-bi-sqldw-adf.png)

このアーキテクチャは、[SQL Data Warehouse を使用したエンタープライズ向け BI](./enterprise-bi-sqldw.md) に示されているアーキテクチャに基づいて構築されていますが、エンタープライズ データ ウェアハウス シナリオに重要な機能がいくつか追加されています。

- Data Factory を使用したパイプラインの自動化
- 段階的な読み込み。
- 複数のデータ ソースの統合。
- 地理空間データや画像などのバイナリ データの読み込み。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

### <a name="data-sources"></a>データ ソース

**オンプレミスの SQL Server**。 ソース データは、オンプレミスの SQL Server データベースにあります。 オンプレミス環境をシミュレートするために、このアーキテクチャのデプロイ スクリプトでは、SQL Server がインストールされた Azure の仮想マシンがプロビジョニングされます。 [Wide World Importers OLTP サンプル データベース][wwi]は、ソース データベースとして使用されます。

**外部データ**。 データ ウェアハウスの一般的なシナリオとして、複数のデータ ソースの統合があります。 この参照アーキテクチャは、年別の市区町村の人口を含む外部データ セットを読み込み、それを OLTP データベースのデータと統合します。 このデータは、"各地域の売上高の増加率は人口の増加率と一致するか、上回っているか" のような分析に使用できます。

### <a name="ingestion-and-data-storage"></a>インジェストとデータ ストレージ

**Blob Storage**。 Blob Storage は、SQL Data Warehouse に読み込む前のソース データのステージング領域として使用されます。

**Azure SQL Data Warehouse**。 [SQL Data Warehouse](/azure/sql-data-warehouse/) は、大規模なデータの分析を目的として設計された分散システムです。 超並列処理 (MPP) がサポートされているので、ハイパフォーマンス分析の実行に適しています。

**Azure Data Factory**。 [Data Factory][adf] は、データ移動とデータ変換を調整し、自動化するマネージド サービスです。 このアーキテクチャでは、ELT プロセスのさまざまな段階が調整されます。

### <a name="analysis-and-reporting"></a>分析とレポート

**Azure Analysis Services**。 [Analysis Services](/azure/analysis-services/) は、データ モデリング機能を提供するフル マネージド サービスです。 セマンティック モデルは Analysis Services に読み込まれます。

**Power BI**。 Power BI は、データを分析してビジネスの分析情報を得る一連のビジネス分析ツールです。 このアーキテクチャでは、Analysis Services に格納されたセマンティック モデルに対してクエリが実行されます。

### <a name="authentication"></a>Authentication

**Azure Active Directory** (Azure AD)。Power BI から Analysis Services サーバーに接続するユーザーを認証します。

Data Factory では、Azure AD を使用し、サービス プリンシパルまたはマネージド サービス ID (MSI) を使用して SQL Data Warehouse を認証することもできます。 わかりやすくするために、この展開例では SQL Server 認証を使用しています。

## <a name="data-pipeline"></a>データ パイプライン

[Azure Data Factory][adf] では、パイプラインはタスク (この例では、データを SQL Data Warehouse に読み込んで変換するタスク) の調整に使用されるアクティビティの論理グループです。

この参照アーキテクチャは、一連の子パイプラインを実行するマスター パイプラインを定義します。 各子パイプラインは、1 つまたは複数のデータ ウェアハウス テーブルにデータを読み込みます。

![Azure Data Factory のパイプラインのスクリーンショット](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>段階的な読み込み

自動化された ETL または ELT プロセスを実行する場合、以前の実行以降に変更されたデータのみを読み込むのが最も効率的です。 すべてのデータを読み込む完全読み込みとは対照的に、これは*段階的な読み込み*と呼ばれます。 段階的な読み込みを実行するには、変更されたデータを特定する方法が必要です。 最も一般的な方法は、*高基準値*を使用することです。つまり、ソース テーブルの特定の列の最新の値 (日時列または一意の整数列) を追跡することです。

SQL Server 2016 以降は[テンポラル テーブル](/sql/relational-databases/tables/temporal-tables)を使用できます。 テンポラル テーブルは、データの完全な変更履歴が保持されている、システムでバージョン管理されたテーブルです。 データベース エンジンは、各変更履歴を別々の履歴テーブルに自動的に記録します。 クエリに FOR SYSTEM_TIME 句を追加することで、履歴データのクエリを実行することができます。 内部的には、データベース エンジンは履歴テーブルのクエリを実行しますが、この処理はアプリケーションにとって透過的です。

> [!NOTE]
> 以前のバージョンの SQL Server では、[変更データ キャプチャ](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC) を使用できます。 このアプローチでは、別の変更テーブルのクエリを実行する必要があり、変更がタイムスタンプではなくログ シーケンス番号で追跡されるため、テンポラル テーブルよりも不便です。
>

テンポラル テーブルは、時間と共に変化する可能性のあるディメンション データの場合に便利です。 通常、ファクト テーブルは、販売などの不変トランザクションを表します。この場合、システムのバージョン履歴を保持することは意味がありません。 その代わり、通常、トランザクションにはトランザクション日付を表す列があります。これをウォーターマーク値として使用できます。 たとえば、Wide World Importers OLTP データベースでは、Sales.Invoices テーブルと Sales.InvoiceLines テーブルに既定が `sysdatetime()` の `LastEditedWhen` フィールドがあります。

ELT パイプラインの一般的なフローは次のとおりです。

1. ソース データベースの各テーブルについて、最後の ELT ジョブが実行されたときのカットオフ時間を追跡します。 この情報をデータ ウェアハウスに格納します (初期設定では、すべての時間が '1-1-1900'に設定されます)。

2. データのエクスポート手順では、カットオフ時間がパラメーターとしてソース データベース内のストアド プロシージャのセットに渡されます。 これらのストアド プロシージャは、カットオフ時間後に変更または作成されたレコードのクエリを実行します。 Sales ファクト テーブルの場合は、`LastEditedWhen` 列が使用されます。 ディメンション データの場合は、システムでバージョン管理されたテンポラル テーブルが使用されます。

3. データの移行が完了したら、カットオフ時間を格納するテーブルを更新します。

ELT の実行ごとに*系列*を記録することも便利です。 特定のレコードについて、系列はそのレコードをそのデータを生成した ELT 実行と関連付けます。 ETL 実行ごとに、読み込みの開始時刻と終了時刻が表示された新しい系列レコードがテーブルごとに作成されます。 各レコードの系列キーは、ディメンション テーブルとファクト テーブルに格納されます。

![City ディメンション テーブルのスクリーンショット](./images/city-dimension-table.png)

新しいデータのバッチがウェアハウスに読み込まれたら、Analysis Services 表形式モデルを更新します。 「[REST API を使用した非同期更新](/azure/analysis-services/analysis-services-async-refresh)」を参照してください。

## <a name="data-cleansing"></a>データ クレンジング

データ クレンジングは、ELT プロセスに含めるようにします。 この参照アーキテクチャでは、不適切なデータ ソースの 1 つが市区町村の人口テーブルです。このテーブルには、データを入手できなかったなどの理由で、人口が 0 の市区町村がいくつかあります。 処理中、ELT パイプラインは、市区町村の人口テーブルからそれらの市区町村を削除します。 外部テーブルではなくステージング テーブルにタイしてデータ クレンジングを実行します。

市区町村の人口テーブルから人口が 0 の市区町村を削除するストアド プロシージャを次に示します (ソース ファイルについては[こちら](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql)を参照してください)。

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>外部データ ソース

データ ウェアハウスは、多くの場合、複数のソースのデータを統合します。 この参照アーキテクチャは、人口統計データを含む外部データ ソースを読み込みます。 このデータセットは、[WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) サンプルの一部として AzureBlob Storage で利用できます。

Azure Data Factory は、[BLOB ストレージ コネクタ](/azure/data-factory/connector-azure-blob-storage)を使用して、BLOB ストレージから直接コピーできます。 ただし、コネクタには接続文字列または共有アクセス署名が必要なため、パブリック読み取りアクセス権がある BLOB のコピーに使用することはできません。 回避策として、PolyBase を使用して BLOB ストレージ上に外部テーブルを作成し、外部テーブルを SQL Data Warehouse にコピーすることができます。

## <a name="handling-large-binary-data"></a>大きなバイナリ データの処理

ソース データベースでは、Cities テーブルに [geography](/sql/t-sql/spatial-geography/spatial-types-geography) 空間データ型を保持する Location 列があります。 SQL Data Warehouse は **geography** 型をネイティブでサポートしていないので、このフィールドは読み込み中に **varbinary** 型に変換されます (「[サポートされていないデータ型の対処法](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types)」を参照してください)。

ただし、PolyBase でサポートされる最大列サイズは `varbinary(8000)` です。そのため、一部のデータが切り捨てられる可能性があります。 この問題の回避策は、次のようにエクスポート時にデータをチャンクに分割し、チャンクを再構成することです。

1. Location 列の一時ステージング テーブルを作成します。

2. 市区町村ごとに場所データを 8,000 バイトのチャンクに分割します。その結果、市区町村ごとに 1 &ndash; N 行になります。

3. チャンクを再構成するには、T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) 演算子を使用して行を列に変換し、各市区町村の列の値を連結します。

課題は、地理データのサイズに応じて、各市区町村を異なる数の行に分割することです。 PIVOT 演算子が機能するには、各市区町村を同じ行数にする必要があります。 この作業を行うために、T-SQL クエリ ([こちら][MergeLocation]を参照してください) で、ピボットの後にすべての市区町村が同じ列数になるように、空白値で行を埋める処理を実行します。 結果のクエリは、一度に 1 行ずつループするよりもはるかに高速になっていることがわかります。

同じアプローチが画像データに使用されます。

## <a name="slowly-changing-dimensions"></a>緩やかに変化するディメンション

ディメンション データは比較的静的ですが、変化する可能性があります。 たとえば、製品が別の製品カテゴリに再割り当てされることがあります。 緩やかに変化するディメンションを処理するにはいくつかのアプローチがあります。 [タイプ 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row) と呼ばれる一般的な手法では、ディメンションが変化するたびに新しいレコードを追加します。

タイプ 2 アプローチを実装するには、特定のレコードの有効な日付範囲を指定する列をディメンション テーブルに追加する必要があります。 また、ソース データベースの主キーが複製されるので、ディメンション テーブルには人工主キーが必要です。

次の図は、Dimension.City テーブルを示しています。 `WWI City ID` 列は、ソース データベースの主キーです。 `City Key` 列は、ETL パイプラインで生成された人工キーです。 また、テーブルには `Valid From` 列と `Valid To` 列があり、各行が有効なときの範囲が定義されています。 現在の値には `Valid To` = '9999-12-31' があります。

![City ディメンション テーブルのスクリーンショット](./images/city-dimension-table.png)

このアプローチの利点は、過去のデータが保存されることです。このデータは分析に役立つ可能性があります。 ただし、これは同じエンティティに対して複数の行が存在することも意味します。 たとえば、`WWI City ID` = 28561 に一致するレコードを次に示します。

![City ディメンション テーブルの 2 つ目のスクリーンショット](./images/city-dimension-table-2.png)

各 Sales ファクトについて、そのファクトを請求書の日付に対応する City ディメンション テーブルの単一の行に関連付ける必要があります。 ETL プロセスの一環として、追加の列を作成します。 

次の T-SQL クエリは、各請求書を City ディメンション テーブルの正しい City Key に関連付けるテンポラル テーブルを作成します。

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

このテーブルは、Sales ファクト テーブルの列を設定するために使用されます。

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

この列では、Power BI クエリを使用して、特定の売上請求書に対して正しい City レコードを検索できます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

さらにセキュリティを強化するには、[仮想ネットワーク サービス エンドポイント](/azure/virtual-network/virtual-network-service-endpoints-overview)を使用して、仮想ネットワークに対してのみ Azure サービス リソースをセキュリティで保護することができます。 これにより、これらのリソースに対するパブリック インターネット アクセスが完全に削除され、自分の仮想ネットワークからのトラフィックのみが許可されます。

このアプローチでは、Azure 内に VNet を作成し、Azure サービス用のプライベート サービス エンドポイントを作成します。 これらのサービスは、その仮想ネットワークからのトラフィックに制限されるようになります。 オンプレミス ネットワークからゲートウェイ経由でアクセスすることもできます。

次の制限事項に注意してください。

- この参照アーキテクチャが作成された時点で、Azure Storage と Azure SQL Data Warehouse では VNet サービス エンドポイントがサポートされていますが、Azure Analysis Service ではサポートされていません。 最新の状態については、[こちら](https://azure.microsoft.com/updates/?product=virtual-network)を参照してください。

- Azure Storage でサービス エンドポイントが有効な場合、PolyBase は Storage から SQL Data Warehouse にデータをコピーできません。 この問題には軽減策があります。 詳細については、「[Azure Storage で VNet サービス エンドポイントを使用した場合の影響](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage)」を参照してください。

- オンプレミスから Azure Storage にデータを移動するには、オンプレミスまたは ExpressRoute からのパブリック IP アドレスをホワイトリストに登録する必要があります。 詳細については、「[Azure サービスへのアクセスを仮想ネットワークに限定する](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks)」を参照してください。

- Analysis Services が SQL Data Warehouse からデータを読み取ることができるようにするには、SQL Data Warehouse サービス エンドポイントを含む仮想ネットワークに Windows VM を展開します。 この VM に [Azure オンプレミス データゲートウェイ](/azure/analysis-services/analysis-services-gateway)をインストールします。 次に Azure Analysis サービスをデータ ゲートウェイに接続します。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

リファレンス実装をデプロイおよび実行するには、[GitHub readme][github] の手順に従ってください。 以下がデプロイされます。

- オンプレミスのデータベース サーバーをシミュレートする Windows VM。 これには、SQL Server 2017 と関連ツール、および Power BI Desktop が含まれています。
- SQL Server データベースからエクスポートされたデータを保持する Blob Storage を提供する Azure ストレージ アカウント。
- Azure SQL Data Warehouse インスタンス。
- Azure Analysis Services インスタンス。
- ELT ジョブ用の Azure Data Factory と Data Factory パイプライン。

[adf]: /azure/data-factory
[github]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
