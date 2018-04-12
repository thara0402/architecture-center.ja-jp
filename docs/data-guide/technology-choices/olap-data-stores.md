---
title: OLAP データ ストアの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Azure で使用する OLAP データ ストアの選択

オンライン分析処理 (OLAP) は、大規模なビジネス データベースを編成し、複雑な分析をサポートする技術です。 このトピックでは、Azure で使用する OLAP ソリューションのオプションを比較します。

> [!NOTE]
> OLAP データ ストアを使用する場合の詳細については、[オンライン分析処理](../scenarios/online-analytical-processing.md)に関するページを参照してください。

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>OLAP データ ストアを選択する場合のオプション

Azure では、以下のすべてのデータ ストアが OLAP のコア要件を満たしています。

- [列ストア インデックスを使用する SQL Server](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) は、ビジネス インテリジェンス アプリケーション向けに OLAP およびデータ マイニング機能を提供しています。 ローカル サーバーに SSAS をインストールするか、Azure の仮想マシン内でホストすることができます。 Azure Analysis Services は、SSAS と同じ主要機能を提供する完全管理のサービスです。 Azure Analysis Services では、組織のクラウドおよびオンプレミスにある[多様なデータ ソース](/azure/analysis-services/analysis-services-datasource)への接続がサポートされます。

クラスター化された列ストア インデックスは、SQL Server 2014 以降と Azure SQL Database で使用できます。OLAP ワークロードに最適です。 ただし、SQL Server 2016 (Azure SQL Database を含む) 以降は、更新可能な非クラスター化列ストア インデックスを使用して、ハイブリッド トランザクション/分析処理 (HTAP) を利用できます。 HTAP を使用すると、同じプラットフォーム上で OLTP と OLAP の処理を実行できます。そのため、データの複数のコピーを格納したり、OLTP システムと OLAP システムを別に用意したりする必要がなくなります。 詳細については、「[列ストアを使用したリアルタイム運用分析の概要](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)」を参照してください。

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- 独自のサーバーを管理するのではなく、マネージド サービスを使用しますか。

- Azure Active Directory (Azure AD) を使用するセキュリティで保護された認証は必要ですか。

- リアルタイム分析を実行したいですか。 "はい" の場合、リアルタイム分析をサポートするオプションに絞り込みます。 

    このコンテキストでの*リアルタイム分析*は、運用ワークロードと分析ワークロードの両方を実行するエンタープライズ リソース プランニング (ERP) アプリケーションなどの単一のデータ ソースに適用されます。 複数のソースのデータを統合する必要がある場合や、キューブなどの事前集計されたデータを使用する高度な分析パフォーマンスが必要な場合は、個別のデータ ウェアハウスが必要になる場合があります。

- ビジネス ユーザーに使いやすい分析にするセマンティック モデルを提供するなど、事前集計されたデータを使用する必要はありますか。 "はい" の場合、多次元キューブまたは表形式セマンティック モデルをサポートするオプションを選択します。 

    集計を提供すると、ユーザーがデータ集計を一貫した方法で計算するために役立ちます。 事前集計されたデータは、多数の行にわたる複数の列を処理する場合にもパフォーマンスが大幅に向上します。 データは、多次元キューブまたは表形式のセマンティック モデルで事前集計できます。

- OLTP データ ストア以外に、複数のソースのデータを統合する必要はありますか。 "はい" の場合、複数のデータ ソースを簡単に統合できるオプションを検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

| | Azure Analysis Services | SQL Server Analysis Services | 列ストア インデックスを使用する SQL Server | 列ストア インデックスを使用する Azure SQL Database |
| --- | --- | --- | --- | --- |
| マネージド サービスか | [はい] | いいえ  | いいえ  | [はい] |
| 多次元キューブをサポート | いいえ  | [はい] | いいえ  | いいえ  |
| 表形式のセマンティック モデルをサポート | [はい] | [はい] | いいえ  | いいえ  |
| 複数のデータ ソースを簡単に統合 | [はい] | [はい] | なし <sup>1</sup> | なし <sup>1</sup> |
| リアルタイムの分析をサポート | いいえ  | いいえ  | 可能  | [はい] |
| ソースからデータをコピーするプロセスが必要 | [はい] | [はい] | いいえ  | いいえ  |
| Azure AD の統合 | [はい] | いいえ  | いいえ <sup>2</sup> | [はい] |

[1] SQL Server と Azure SQL Database を使用して複数の外部データ ソースからクエリを実行して統合することはできませんが、[SSIS](/sql/integration-services/sql-server-integration-services) または [Azure Data Factory](/azure/data-factory/) を使用してこの処理を実行するパイプラインを構築することはできます。 Azure VM でホストされている SQL Server には、リンク サーバーや [PolyBase](/sql/relational-databases/polybase/polybase-guide) など、その他のオプションがあります。 詳細については、[パイプラインのオーケストレーション、制御フロー、およびデータの移動](../technology-choices/pipeline-orchestration-data-movement.md)に関するページを参照してください。

[2] Azure AD アカウントを使用している場合、Azure Virtual Machine 上で実行されている SQL Server への接続はサポートされていません。 代わりにドメインの Active Directory アカウントを使用してください。

### <a name="scalability-capabilities"></a>スケーラビリティ機能

| | Azure Analysis Services | SQL Server Analysis Services | 列ストア インデックスを使用する SQL Server | 列ストア インデックスを使用する Azure SQL Database |
| --- | --- | --- | --- | --- |
| 高可用性のための冗長リージョン サーバー  | [はい] | いいえ  | 可能  | [はい] |
| クエリのスケールアウトをサポート  | [はい] | いいえ  | [はい] | いいえ  |
| 動的スケーラビリティ (スケールアップ)  | [はい] | いいえ  | [はい] | いいえ  |

