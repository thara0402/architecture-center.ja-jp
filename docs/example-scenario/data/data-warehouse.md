---
title: 販売およびマーケティング向けのデータ ウェアハウスと分析
description: 複数のソースのデータを統合し、データ分析を最適化します。
author: alexbuckgit
ms.date: 09/15/2018
ms.openlocfilehash: f9ca9785b65f18098a91aedc1f3157f49456a6e1
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818651"
---
# <a name="data-warehousing-and-analytics-for-sales-and-marketing"></a>販売およびマーケティング向けのデータ ウェアハウスと分析

このシナリオ例では、複数のソースからの大量のデータを Azure の統合分析プラットフォームに統合するデータ パイプラインを示します。 この特定のシナリオは販売とマーケティングのソリューションに基づいていますが、この設計パターンは、eコマース、小売り、医療など、大規模なデータセットの高度な分析を必要とする多くの業界に関係があります。

この例では、インセンティブ プログラムを作成する販売およびマーケティング会社を示します。 これらのプログラムは、顧客、仕入先、営業担当者、および従業員に報奨を提供します。 データはこれらのプログラムの基礎であり、会社は Azure を使用してデータ解析により得られる分析情報を向上させることを望んでいます。

適切なデータを使用して適切なタイミングで意思決定が行われるように、最新のデータ分析アプローチが必要です。 会社の目標は次のとおりです。
* 異なる種類のデータ ソースをクラウド規模のプラットフォームに組み合わせる。
* データに一貫性を持たせて簡単に比較できるようにするため、ソース データを共通の分類と構造に変換する。
* オンプレミスのインフラストラクチャの展開と保守に高いコストをかけることなく、数千のインセンティブ プログラムをサポートできる高度に並列化されたアプローチを使用してデータを読み込む。
* ユーザーがデータの分析に集中できるように、データの収集と変換に必要な時間を大幅に短縮する。

## <a name="relevant-use-cases"></a>関連するユース ケース

このアプローチは、次のことを実現するためにも使用できます。

* データ ウェアハウスをデータの信頼できる単一のソースとして確立する。
* リレーショナル データ ソースを他の非構造化データセットと統合する。
* セマンティック モデリングと強力な視覚化ツールを使用してデータ分析を簡単にする。

## <a name="architecture"></a>アーキテクチャ

![Azure でのデータ ウェアハウスと分析のシナリオのためのアーキテクチャ][architecture]

このソリューションのデータ フローは次のとおりです。

1. データ ソースごとに、すべての更新が Azure Blob Storage 内のステージング領域に定期的にエクスポートされます。
2. Data Factory によって、BLOB ストレージから SQL Data Warehouse 内のステージング テーブルに、少しずつデータが読み込まれます。 このプロセスの間にデータのクレンジングと変換が行われます。 Polybase は、大規模なデータセットに対して処理を並列化できます。
3. 新しいデータのバッチがウェアハウスに読み込まれた後、以前に作成されていた Analysis Services の表形式モデルが更新されます。 このセマンティック モデルにより、ビジネス データと関係の分析が簡略化します。
4. ビジネス アナリストは、Microsoft Power BI を使用し、Analysis Services のセマンティック モデルにより、ウェアハウスのデータを分析します。

### <a name="components"></a>コンポーネント

会社のデータ ソースは多種多様なプラットフォーム上にあります。
* オンプレミスの SQL Server
* オンプレミスの Oracle
* Azure SQL Database
* Azure テーブル ストレージ
* Cosmos DB

複数の Azure コンポーネントを使用して、これらの異なるデータ ソースからデータが読み込まれます。
* [BLOB ストレージ](/azure/storage/blobs/storage-blobs-introduction) は、SQL Data Warehouse に読み込まれる前のソース データのステージングに使用されます。
* [Data Factory](/azure/data-factory) は、SQL Data Warehouse 内の共通構造へのステージング データの変換を調整します。 Data Factory では、スループットを最大化するため、[SQL Data Warehouse にデータを読み込むときに Polybase を使用](/azure/data-factory/connector-azure-sql-data-warehouse#use-polybase-to-load-data-into-azure-sql-data-warehouse)します。 
* [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) は、大規模なデータセットを格納および分析するための分散システムです。 超並列処理 (MPP) が使用されているので、ハイパフォーマンス分析の実行に適しています。 SQL Data Warehouse では、[PolyBase](/sql/relational-databases/polybase/polybase-guide) を使用して、BLOB ストレージからのデータの読み込みを高速化できます。
* [Analysis Services](/azure/analysis-services) は、データのセマンティック モデルを提供します。 このコンポーネントの使用により、データの分析時のシステム パフォーマンスの向上にもつながります。 
* [Power BI](/power-bi) は、データを分析し、洞察を共有する一連のビジネス分析ツールです。 Power BI は、Analysis Services に格納されているセマンティック モデルに対してクエリを実行することも、SQL Data Warehouse に直接照会することもできます。
* [Azure Active Directory (Azure AD)](/azure/active-directory) では、Power BI から Analysis Services サーバーに接続するユーザーの認証が行われます。 Data Factory では、Azure AD を使用して、サービス プリンシパルまたは [Azure リソース用のマネージド ID](/azure/active-directory/managed-identities-azure-resources/overview) により、SQL Data Warehouse に対する認証を行うこともできます。

### <a name="alternatives"></a>代替手段

* パイプラインの例には、複数の異なる種類のデータ ソースが含まれます。 このアーキテクチャでは、さまざまなリレーショナルおよび非リレーショナル データ ソースを処理できます。
* Data Factory では、データ パイプラインに対するワークフローが調整されます。 データの読み込みを 1 回だけ、またはオンデマンドで行いたい場合は、SQL Server の一括コピー (bcp) や AzCopy などのツールを使用して、データを BLOB ストレージにコピーできます。 その後、PolyBase を使用して SQL Data Warehouse に直接データをロードできます。
* 非常に大きいデータセットがある場合は、[Data Lake Storage](/azure/storage/data-lake-storage/introduction) の使用を検討します。この機能では、分析データ用に無制限のストレージが提供されます。
* ビッグ データの処理には、オンプレミスの [SQL Server 並列データ ウェアハウス](/sql/analytics-platform-system) アプライアンスも使用できます。 ただし、多くの場合、運用コストは、SQL Data Warehouse のようなクラウド ベースのマネージド ソリューションの方がはるかに少なくて済みます。 
* SQL Data Warehouse は、OLTP ワークロードや 250 GB 未満のデータ セットには適していません。 このような場合は、Azure SQL Database または SQL Server を使用する必要があります。
* 他の代替手段の比較については、以下をご覧ください。
    * [Azure でのデータ パイプライン オーケストレーション テクノロジの選択](/azure/architecture/data-guide/technology-choices/pipeline-orchestration-data-movement)
    * [Azure で使用するバッチ処理テクノロジの選択](/azure/architecture/data-guide/technology-choices/batch-processing)
    * [Azure で使用する分析データ ストアの選択](/azure/architecture/data-guide/technology-choices/analytical-data-stores)
    * [Azure でのデータ分析とテクノロジの選択](/azure/architecture/data-guide/technology-choices/analysis-visualizations-reporting)

## <a name="considerations"></a>考慮事項

このアーキテクチャのテクノロジは、スケーラビリティおよび可用性とコスト管理の両立という会社の要件を満たすために選択されました。

* SQL Data Warehouse の[超並列処理アーキテクチャ](/azure/sql-data-warehouse/massively-parallel-processing-mpp-architecture)により、スケーラビリティとハイパフォーマンスを実現します。
* SQL Data Warehouse には、[保証された SLA](https://azure.microsoft.com/support/legal/sla/sql-data-warehouse) と[高可用性を実現するための推奨プラクティス](/azure/sql-data-warehouse/sql-data-warehouse-best-practices)があります。
* 分析アクティビティが低下した場合、会社は [SQL Data Warehouse をオンデマンドでスケーリング](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)し、計算を減らしたり、さらには計算を一時停止することにより、コストを削減できます。
* クエリのワークロードが高いときは、Azure Analysis Services を[スケールアウト](/azure/analysis-services/analysis-services-scale-out)して応答時間を短縮できます。 クエリ プールから処理を分離して、処理操作によってクライアントのクエリが遅くならないようにすることもできます。 
* Azure Analysis Services には、[保証された SLA](https://azure.microsoft.com/support/legal/sla/analysis-services) と[高可用性を実現するための推奨プラクティス](/azure/analysis-services/analysis-services-bcdr)もあります。
* [SQL Data Warehouse のセキュリティ モデル](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)は、Azure AD または SQL Server の認証と暗号化による接続セキュリティ、[認証、承認](/azure/sql-data-warehouse/sql-data-warehouse-authentication)を提供します。 [Azure Analysis Services](/azure/analysis-services/analysis-services-manage-users) では、ID 管理とユーザー認証に Azure AD を使用します。 

## <a name="pricing"></a>価格

Azure 料金計算ツールを使用して、[データ ウェアハウジングのシナリオの価格サンプル][calculator]を確認してください。 値を調整して、要件によるコストへの影響を確認できます。

* [SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) を使用すると、計算レベルとストレージ レベルを個別にスケーリングすることができます。 計算リソースは 1 時間単位で課金されるため、オンデマンドでそのリソースをスケーリングまたは一時停止できます。 ストレージ リソースはテラバイト単位で課金されるため、データを取り込んだ分だけコストが増加します。
* [Data Factory](https://azure.microsoft.com/pricing/details/data-factory) のコストは、ワークロード内で実行された読み取り/書き込み操作、監視操作、オーケストレーション アクティビティの数に基づきます。 Data Factory のコストは、データ ストリームおよび各データ ストリームでのデータ量が追加されると増加します。
* [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) は、Developer レベル、Basic レベル、および Standard レベルで利用できます。 インスタンスは、クエリ処理単位 (QPU) と使用可能なメモリに基づいて価格設定されます。 コストを抑えるには、実行するクエリの数、処理するデータの量、実行頻度をできるだけ少なくします。
* [Power BI](https://powerbi.microsoft.com/pricing) には、要件に応じたさまざまな製品オプションがあります。 [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) では、Power BI の機能をアプリケーションに埋め込むための Azure ベースのオプションが提供されます。 Power BI Embedded インスタンスは上記の料金サンプルに含まれています。

## <a name="next-steps"></a>次の手順

* [自動化されたエンタープライズ BI に関する Azure 参照アーキテクチャ](/azure/architecture/reference-architectures/data/enterprise-bi-adf)を確認してください。この参照アーキテクチャには、このアーキテクチャのインスタンスを Azure に展開する手順が含まれます。
* [Maritz Motivation Solutions の顧客事例][source-document]をご覧ください。 顧客データの管理に関する同様のアプローチについて説明されています。
* データ パイプライン、データ ウェアハウジング、オンライン分析処理 (OLAP)、ビッグ データに関する包括的なアーキテクチャ ガイダンスについては、[Azure データ アーキテクチャ ガイド](/azure/architecture/data-guide)をご覧ください。

<!-- links -->
[source-document]: https://customers.microsoft.com/story/maritz
[calculator]: https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276
[architecture]: ./media/architecture-data-warehouse.png
