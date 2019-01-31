---
title: 既存のオンプレミス SSIS と Azure Data Factory を使用したハイブリッド ETL
titleSuffix: Azure Example Scenarios
description: 既存のオンプレミス SQL Server Integration Services (SSIS) デプロイと Azure Data Factory を使用したハイブリッド ETL。
author: alhieng
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: tsp-team
social_image_url: /azure/architecture/example-scenario/data/media/architecture-diagram-hybrid-etl-with-adf.png
ms.openlocfilehash: e8d80bb55d51bfbc982936d2b5dc98a232e061b5
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908530"
---
# <a name="hybrid-etl-with-existing-on-premises-ssis-and-azure-data-factory"></a>既存のオンプレミス SSIS と Azure Data Factory を使用したハイブリッド ETL

SQL Server データベースをクラウドに移行すると、組織はコストを大幅に削減し、パフォーマンス、柔軟性、スケーラビリティを高めることができます。 ただし、SQL Server Integration Services (SSIS) で構築された既存の抽出、変換、読み込み (ETL) プロセスを作り変えることが、移行の障害になる可能性があります。 場合によっては、データ読み込みプロセスには、複雑なロジックや、Azure Data Factory v2 でまだサポートされていない特定のデータ ツール コンポーネントが必要です。 一般的に使用される SSIS 機能には、あいまい参照変換とあいまいグループ化変換、変更データ キャプチャ (CDC)、緩やかに変化するディメンション (SCD)、および Data Quality Services (DQS) が含まれます。

既存の SQL データベースのリフトアンドシフト移行を支援するには、ハイブリッド ETL アプローチがオプションとして最も適している可能性があります。 ハイブリッドのアプローチでは、プライマリ オーケストレーション エンジンとして Data Factory が使用されますが、データのクリーニングとオンプレミス リソースの操作には既存の SSIS パッケージが引き続き活用されます。 このアプローチでは、既存のコードと SSIS パッケージを使用しながら、Data Factory SQL Server 統合ランタイム (IR) を使うことで、クラウドへの既存のデータベースのリフトアンドシフトが可能になります。

このサンプル シナリオの対象は、既存の SSIS パッケージを新しいクラウド データ ワークフローに組み込みながら、データベースをクラウドに移動し、プライマリ クラウドベース ETL エンジンとして Data Factory を使用することを検討している組織です。 特定のデータ タスク向けの SSIS ETL パッケージの開発に、多額の投資を行ってきた組織は多数存在します。 これらのパッケージの書き換えには、かなり手間がかかることがあります。 また、既存のコード パッケージの多くがローカル リソースに依存しており、これがクラウドへの移行を妨げています。

Data Factory によって、お客様はオンプレミスの ETL 開発に対するさらなる投資を抑え、既存の ETL パッケージを活用できます。 この例では、Azure Data Factory v2 を使用して、既存の SSIS パッケージを新しいクラウド データ ワークフローの一部として利用するために考えられるユース ケースについて説明します。

## <a name="potential-use-cases"></a>考えられるユース ケース

従来、SSIS は、多数の SQL Server データ プロフェッショナルに選ばれてきた、データの変換と読み込みを行うための ETL ツールです。 特定の SSIS 機能やサード パーティ製の組み込みコンポーネントが、開発の取り組みを加速するために使用されることもあります。 これらのパッケージについては、置き換えたり開発し直したりできないことがあるため、お客様がクラウドにデータベースの移行できない原因になっています。 お客様が求めているのは、既存のデータベースをクラウドに移行する際の影響を小さく抑え、既存の SSIS パッケージを利用できるアプローチです。

考えられるオンプレミスのユース ケースを次に示します。

- 分析用にネットワーク ルーター ログをデータベースに読み込む。
- 分析レポート作成のために人事雇用データを準備する。
- 売上予測のために製品および売上データをデータ ウェアハウスに読み込む。
- 財務会計のデータ ストアまたはデータ ウェアハウスの読み込みや運用を自動化する。

## <a name="architecture"></a>アーキテクチャ

![Azure Data Factory を使用したハイブリッド ETL プロセスのアーキテクチャの概要][architecture-diagram]

1. データ ソースである Blob Storage から Data Factory にデータが移動します。
2. Data Factory パイプラインによってストアド プロシージャが呼び出され、統合ランタイムを介してオンプレミスでホストされている SSIS ジョブが実行されます。
3. データ クレンジング ジョブが実行され、ダウンストリームでの使用のためにデータが準備されます。
4. データ クレンジング タスクが正常に完了すると、コピー タスクが実行され、クリーン データが Azure に読み込まれます。
5. その後、クリーン データは、SQL Data Warehouse のテーブルに読み込まれます。

### <a name="components"></a>コンポーネント

- [Blob Storage][docs-blob-storage]: ファイルの格納に使用されます。Data Factory がデータを取得するソースとして機能します。
- [SQL Server Integration Services][docs-ssis]: タスク固有のワークロードの実行に使用されるオンプレミス ETL パッケージが含まれます。
- [Azure Data Factory][docs-data-factory]: 複数のソースからデータを取得、結合、調整して、データ ウェアハウスに読み込むクラウド オーケストレーション エンジンです。
- [SQL Data Warehouse][docs-sql-data-warehouse]: データをクラウド内で一元化して、標準の ANSI SQL クエリを使用して簡単にアクセスできるようにします。

### <a name="alternatives"></a>代替手段

Data Factory では、仮想マシンで実行されている SSIS インスタンス、Databricks ノートブック、Python スクリプトなど、他のテクノロジを使用して実装されているデータ クレンジング プロシージャが呼び出される場合があります。 [Azure SSIS 統合ランタイムの有料 (ライセンスあり) カスタム コンポーネントのインストール](/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components)が、ハイブリッド アプローチの代替手段として利用できる可能性があります。

## <a name="considerations"></a>考慮事項

統合ランタイム (IR) では、セルフホステッド IR と Azure でホストされる IR の 2 つのモデルがサポートされています。 まず、これら 2 つのオプションのどちらを使用するかを決める必要があります。 セルフホスティングはコスト効率に優れていますが、メンテナンスと管理のオーバーヘッドが増えます。 詳細については、[セルフホステッド IR](/azure/data-factory/concepts-integration-runtime#self-hosted-integration-runtime)に関する記事をご覧ください。 使用する IR を判断する際にサポートが必要な場合は、「[使用する IR の判別](/azure/data-factory/concepts-integration-runtime#determining-which-ir-to-use)」を参照してください。

Azure ホスト アプローチの場合、データの処理にどれくらい電力が必要かを判断する必要があります。 Azure ホスト構成を選択すると、構成手順の一環として VM のサイズを選択できます。 VM サイズの選択の詳細については、[VM のパフォーマンスに関する考慮事項](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations)の記事をご覧ください。

既存の SSIS パッケージに、データ ソースやファイルなどのオンプレミスの依存関係が既に含まれていて、それらに Azure からアクセスできない場合は、決めるのは簡単です。 このシナリオでは、セルフホステッド IR しか選択できません。 このアプローチでは、クラウドをオーケストレーション エンジンとして活用するうえで最高の柔軟性が提供され、既存のパッケージを再生成する必要がありません。

最終的な目標は、処理されたデータをクラウドに移動して、さらなる改善を実現する、あるいは、クラウドに格納されている他のデータと結合することです。 設計プロセスの一環として、Data Factory のパイプラインで使用されているアクティビティの数を追跡してください。 詳細については、「[Azure Data Factory のパイプラインとアクティビティ](/azure/data-factory/concepts-pipelines-activities)」を参照してください。

## <a name="pricing"></a>価格

Data Factory は、クラウドでのデータ移動を調整するための、コスト効率に優れた方法です。 コストは、複数の要因に基づいています。

- パイプラインの実行の数
- パイプラインで使用されるエンティティおよびアクティビティの数
- 監視操作の数
- 統合実行 (Azure でホストされる IR またはセルフホステッド IR) の数

Data Factory では、使用量ベースの課金が採用されています。 このため、コストが発生するのは、パイプラインの実行中と監視中のみです。 基本パイプラインの実行コストはわずか 50 セント、監視コストはわずか 25 セントです。 [Azure の料金計算ツール](https://azure.microsoft.com/pricing/calculator/)を使用すると、ご自身のワークロードに基づいて、さらに正確な見積もりを作成できます。

ハイブリッド ETL ワークロードを実行するときは、SSIS パッケージのホストに使用される仮想マシンのコストを考慮する必要があります。 このコストは、D1v2 (1 コア、3.5 GB RAM、50 GB ディスク) から E64V3 (64 コア、432 GB RAM、1600 GB ディスク) までの VM サイズに基づいています。 適切な VM サイズの選択について詳しいガイダンスが必要な場合は、[VM のパフォーマンスに関する考慮事項](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations)の記事をご覧ください。

## <a name="next-steps"></a>次の手順

- [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) の詳細を確認します。
- [ステップ バイ ステップのチュートリアル](/azure/data-factory/#step-by-step-tutorials)に従って、Azure Data Factory を使ってみます。
- [Azure Data Factory で Azure-SSIS Integration Runtime をプロビジョニング](/azure/data-factory/tutorial-deploy-ssis-packages-azure)します。

<!-- links -->
[architecture-diagram]: ./media/architecture-diagram-hybrid-etl-with-adf.png
[small-pricing]: https://azure.com/e/
[medium-pricing]: https://azure.com/e/
[large-pricing]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-blob-storage]: /azure/storage/blobs/
[docs-data-factory]: /azure/data-factory/introduction
[docs-resource-groups]: /azure/azure-resource-manager/resource-group-overview
[docs-ssis]: /sql/integration-services/sql-server-integration-services
[docs-sql-data-warehouse]: /azure/sql-data-warehouse/sql-data-warehouse-overview-what-is
