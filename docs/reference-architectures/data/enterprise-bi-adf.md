---
title: 自動化されたエンタープライズ ビジネス インテリジェンス (BI)
titleSuffix: Azure Reference Architectures
description: Azure Data Factory と SQL Data Warehouse を使用して、Azure での抽出、読み込み、および変換 (ELT) ワークフローを自動化します。
author: MikeWasson
ms.date: 11/06/2018
ms.custom: seodec18
ms.openlocfilehash: 579ef0361ec44d0eb82b9076490eed5a6d88df35
ms.sourcegitcommit: cd3de23543f739a95a1daf38886561f67add9d64
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/10/2019
ms.locfileid: "54183594"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a><span data-ttu-id="df0fd-103">SQL Data Warehouse と Azure Data Factory を使用したエンタープライズ BI の自動化</span><span class="sxs-lookup"><span data-stu-id="df0fd-103">Automated enterprise BI with SQL Data Warehouse and Azure Data Factory</span></span>

<span data-ttu-id="df0fd-104">この参照アーキテクチャは、[抽出 - 読み込み - 変換 (ELT)](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) パイプラインで段階的に読み込む方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-104">This reference architecture shows how to perform incremental loading in an [extract, load, and transform (ELT)](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) pipeline.</span></span> <span data-ttu-id="df0fd-105">Azure Data Factory を使用して ELT パイプラインを自動化します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-105">It uses Azure Data Factory to automate the ELT pipeline.</span></span> <span data-ttu-id="df0fd-106">パイプラインは、オンプレミスの SQL Server データベースから SQL Data Warehouse に最新の OLTP データを段階的に移動します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-106">The pipeline incrementally moves the latest OLTP data from an on-premises SQL Server database into SQL Data Warehouse.</span></span> <span data-ttu-id="df0fd-107">トランザクション データは、分析のために表形式モデルに変換されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-107">Transactional data is transformed into a tabular model for analysis.</span></span>

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2Gnz2]

<span data-ttu-id="df0fd-108">このアーキテクチャのリファレンス実装は、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-108">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![SQL Data Warehouse と Azure Data Factory を使用して自動化されたエンタープライズ向け BI のアーキテクチャ ダイアグラム](./images/enterprise-bi-sqldw-adf.png)

<span data-ttu-id="df0fd-110">このアーキテクチャは、[SQL Data Warehouse を使用したエンタープライズ向け BI](./enterprise-bi-sqldw.md) に示されているアーキテクチャに基づいて構築されていますが、エンタープライズ データ ウェアハウス シナリオに重要な機能がいくつか追加されています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-110">This architecture builds on the one shown in [Enterprise BI with SQL Data Warehouse](./enterprise-bi-sqldw.md), but adds some features that are important for enterprise data warehousing scenarios.</span></span>

- <span data-ttu-id="df0fd-111">Data Factory を使用したパイプラインの自動化</span><span class="sxs-lookup"><span data-stu-id="df0fd-111">Automation of the pipeline using Data Factory.</span></span>
- <span data-ttu-id="df0fd-112">段階的な読み込み。</span><span class="sxs-lookup"><span data-stu-id="df0fd-112">Incremental loading.</span></span>
- <span data-ttu-id="df0fd-113">複数のデータ ソースの統合。</span><span class="sxs-lookup"><span data-stu-id="df0fd-113">Integrating multiple data sources.</span></span>
- <span data-ttu-id="df0fd-114">地理空間データや画像などのバイナリ データの読み込み。</span><span class="sxs-lookup"><span data-stu-id="df0fd-114">Loading binary data such as geospatial data and images.</span></span>

## <a name="architecture"></a><span data-ttu-id="df0fd-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="df0fd-115">Architecture</span></span>

<span data-ttu-id="df0fd-116">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-116">The architecture consists of the following components.</span></span>

### <a name="data-sources"></a><span data-ttu-id="df0fd-117">データ ソース</span><span class="sxs-lookup"><span data-stu-id="df0fd-117">Data sources</span></span>

<span data-ttu-id="df0fd-118">**オンプレミスの SQL Server**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-118">**On-premises SQL Server**.</span></span> <span data-ttu-id="df0fd-119">ソース データは、オンプレミスの SQL Server データベースにあります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-119">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="df0fd-120">オンプレミス環境をシミュレートするために、このアーキテクチャのデプロイ スクリプトでは、SQL Server がインストールされた Azure の仮想マシンがプロビジョニングされます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-120">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> <span data-ttu-id="df0fd-121">[Wide World Importers OLTP サンプル データベース][wwi]は、ソース データベースとして使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-121">The [Wide World Importers OLTP sample database][wwi] is used as the source database.</span></span>

<span data-ttu-id="df0fd-122">**外部データ**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-122">**External data**.</span></span> <span data-ttu-id="df0fd-123">データ ウェアハウスの一般的なシナリオとして、複数のデータ ソースの統合があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-123">A common scenario for data warehouses is to integrate multiple data sources.</span></span> <span data-ttu-id="df0fd-124">この参照アーキテクチャは、年別の市区町村の人口を含む外部データ セットを読み込み、それを OLTP データベースのデータと統合します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-124">This reference architecture loads an external data set that contains city populations by year, and integrates it with the data from the OLTP database.</span></span> <span data-ttu-id="df0fd-125">このデータは、"各地域の売上高の増加率は人口の増加率と一致するか、上回っているか" のような分析に使用できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-125">You can use this data for insights such as: "Does sales growth in each region match or exceed population growth?"</span></span>

### <a name="ingestion-and-data-storage"></a><span data-ttu-id="df0fd-126">インジェストとデータ ストレージ</span><span class="sxs-lookup"><span data-stu-id="df0fd-126">Ingestion and data storage</span></span>

<span data-ttu-id="df0fd-127">**Blob Storage**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-127">**Blob Storage**.</span></span> <span data-ttu-id="df0fd-128">Blob Storage は、SQL Data Warehouse に読み込む前のソース データのステージング領域として使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-128">Blob storage is used as a staging area for the source data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="df0fd-129">**Azure SQL Data Warehouse**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-129">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="df0fd-130">[SQL Data Warehouse](/azure/sql-data-warehouse/) は、大規模なデータの分析を目的として設計された分散システムです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-130">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="df0fd-131">超並列処理 (MPP) がサポートされているので、ハイパフォーマンス分析の実行に適しています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-131">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span>

<span data-ttu-id="df0fd-132">**Azure Data Factory**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-132">**Azure Data Factory**.</span></span> <span data-ttu-id="df0fd-133">[Data Factory][adf] は、データ移動とデータ変換を調整し、自動化するマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-133">[Data Factory][adf] is a managed service that orchestrates and automates data movement and data transformation.</span></span> <span data-ttu-id="df0fd-134">このアーキテクチャでは、ELT プロセスのさまざまな段階が調整されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-134">In this architecture, it coordinates the various stages of the ELT process.</span></span>

### <a name="analysis-and-reporting"></a><span data-ttu-id="df0fd-135">分析とレポート</span><span class="sxs-lookup"><span data-stu-id="df0fd-135">Analysis and reporting</span></span>

<span data-ttu-id="df0fd-136">**Azure Analysis Services**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-136">**Azure Analysis Services**.</span></span> <span data-ttu-id="df0fd-137">[Analysis Services](/azure/analysis-services/) は、データ モデリング機能を提供するフル マネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-137">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="df0fd-138">セマンティック モデルは Analysis Services に読み込まれます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-138">The semantic model is loaded into Analysis Services.</span></span>

<span data-ttu-id="df0fd-139">**Power BI**。</span><span class="sxs-lookup"><span data-stu-id="df0fd-139">**Power BI**.</span></span> <span data-ttu-id="df0fd-140">Power BI は、データを分析してビジネスの分析情報を得る一連のビジネス分析ツールです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-140">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="df0fd-141">このアーキテクチャでは、Analysis Services に格納されたセマンティック モデルに対してクエリが実行されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-141">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

### <a name="authentication"></a><span data-ttu-id="df0fd-142">Authentication</span><span class="sxs-lookup"><span data-stu-id="df0fd-142">Authentication</span></span>

<span data-ttu-id="df0fd-143">**Azure Active Directory** (Azure AD)。Power BI から Analysis Services サーバーに接続するユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-143">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

<span data-ttu-id="df0fd-144">Data Factory では、Azure AD を使用し、サービス プリンシパルまたはマネージド サービス ID (MSI) を使用して SQL Data Warehouse を認証することもできます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-144">Data Factory can use also use Azure AD to authenticate to SQL Data Warehouse, by using a service principal or Managed Service Identity (MSI).</span></span> <span data-ttu-id="df0fd-145">わかりやすくするために、この展開例では SQL Server 認証を使用しています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-145">For simplicity, the example deployment uses SQL Server authentication.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="df0fd-146">データ パイプライン</span><span class="sxs-lookup"><span data-stu-id="df0fd-146">Data pipeline</span></span>

<span data-ttu-id="df0fd-147">[Azure Data Factory][adf] では、パイプラインはタスク (この例では、データを SQL Data Warehouse に読み込んで変換するタスク) の調整に使用されるアクティビティの論理グループです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-147">In [Azure Data Factory][adf], a pipeline is a logical grouping of activities used to coordinate a task &mdash; in this case, loading and transforming data into SQL Data Warehouse.</span></span>

<span data-ttu-id="df0fd-148">この参照アーキテクチャは、一連の子パイプラインを実行するマスター パイプラインを定義します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-148">This reference architecture defines a master pipeline that runs a sequence of child pipelines.</span></span> <span data-ttu-id="df0fd-149">各子パイプラインは、1 つまたは複数のデータ ウェアハウス テーブルにデータを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-149">Each child pipeline loads data into one or more data warehouse tables.</span></span>

![Azure Data Factory のパイプラインのスクリーンショット](./images/adf-pipeline.png)

## <a name="incremental-loading"></a><span data-ttu-id="df0fd-151">段階的な読み込み</span><span class="sxs-lookup"><span data-stu-id="df0fd-151">Incremental loading</span></span>

<span data-ttu-id="df0fd-152">自動化された ETL または ELT プロセスを実行する場合、以前の実行以降に変更されたデータのみを読み込むのが最も効率的です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-152">When you run an automated ETL or ELT process, it's most efficient to load only the data that changed since the previous run.</span></span> <span data-ttu-id="df0fd-153">すべてのデータを読み込む完全読み込みとは対照的に、これは*段階的な読み込み*と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-153">This is called an *incremental load*, as opposed to a full load that loads all of the data.</span></span> <span data-ttu-id="df0fd-154">段階的な読み込みを実行するには、変更されたデータを特定する方法が必要です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-154">To perform an incremental load, you need a way to identify which data has changed.</span></span> <span data-ttu-id="df0fd-155">最も一般的な方法は、*高基準値*を使用することです。つまり、ソース テーブルの特定の列の最新の値 (日時列または一意の整数列) を追跡することです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-155">The most common approach is to use a *high water mark* value, which means tracking the latest value of some column in the source table, either a datetime column or a unique integer column.</span></span>

<span data-ttu-id="df0fd-156">SQL Server 2016 以降は[テンポラル テーブル](/sql/relational-databases/tables/temporal-tables)を使用できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-156">Starting with SQL Server 2016, you can use [temporal tables](/sql/relational-databases/tables/temporal-tables).</span></span> <span data-ttu-id="df0fd-157">テンポラル テーブルは、データの完全な変更履歴が保持されている、システムでバージョン管理されたテーブルです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-157">These are system-versioned tables that keep a full history of data changes.</span></span> <span data-ttu-id="df0fd-158">データベース エンジンは、各変更履歴を別々の履歴テーブルに自動的に記録します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-158">The database engine automatically records the history of every change in a separate history table.</span></span> <span data-ttu-id="df0fd-159">クエリに FOR SYSTEM_TIME 句を追加することで、履歴データのクエリを実行することができます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-159">You can query the historical data by adding a FOR SYSTEM_TIME clause to a query.</span></span> <span data-ttu-id="df0fd-160">内部的には、データベース エンジンは履歴テーブルのクエリを実行しますが、この処理はアプリケーションにとって透過的です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-160">Internally, the database engine queries the history table, but this is transparent to the application.</span></span>

> [!NOTE]
> <span data-ttu-id="df0fd-161">以前のバージョンの SQL Server では、[変更データ キャプチャ](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-161">For earlier versions of SQL Server, you can use [Change Data Capture](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span></span> <span data-ttu-id="df0fd-162">このアプローチでは、別の変更テーブルのクエリを実行する必要があり、変更がタイムスタンプではなくログ シーケンス番号で追跡されるため、テンポラル テーブルよりも不便です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-162">This approach is less convenient than temporal tables, because you have to query a separate change table, and changes are tracked by a log sequence number, rather than a timestamp.</span></span>
>

<span data-ttu-id="df0fd-163">テンポラル テーブルは、時間と共に変化する可能性のあるディメンション データの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-163">Temporal tables are useful for dimension data, which can change over time.</span></span> <span data-ttu-id="df0fd-164">通常、ファクト テーブルは、販売などの不変トランザクションを表します。この場合、システムのバージョン履歴を保持することは意味がありません。</span><span class="sxs-lookup"><span data-stu-id="df0fd-164">Fact tables usually represent an immutable transaction such as a sale, in which case keeping the system version history doesn't make sense.</span></span> <span data-ttu-id="df0fd-165">その代わり、通常、トランザクションにはトランザクション日付を表す列があります。これをウォーターマーク値として使用できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-165">Instead, transactions usually have a column that represents the transaction date, which can be used as the watermark value.</span></span> <span data-ttu-id="df0fd-166">たとえば、Wide World Importers OLTP データベースでは、Sales.Invoices テーブルと Sales.InvoiceLines テーブルに既定が `sysdatetime()` の `LastEditedWhen` フィールドがあります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-166">For example, in the Wide World Importers OLTP database, the Sales.Invoices and Sales.InvoiceLines tables have a `LastEditedWhen` field that defaults to `sysdatetime()`.</span></span>

<span data-ttu-id="df0fd-167">ELT パイプラインの一般的なフローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-167">Here is the general flow for the ELT pipeline:</span></span>

1. <span data-ttu-id="df0fd-168">ソース データベースの各テーブルについて、最後の ELT ジョブが実行されたときのカットオフ時間を追跡します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-168">For each table in the source database, track the cutoff time when the last ELT job ran.</span></span> <span data-ttu-id="df0fd-169">この情報をデータ ウェアハウスに格納します</span><span class="sxs-lookup"><span data-stu-id="df0fd-169">Store this information in the data warehouse.</span></span> <span data-ttu-id="df0fd-170">(初期設定では、すべての時間が '1-1-1900'に設定されます)。</span><span class="sxs-lookup"><span data-stu-id="df0fd-170">(On initial setup, all times are set to '1-1-1900'.)</span></span>

2. <span data-ttu-id="df0fd-171">データのエクスポート手順では、カットオフ時間がパラメーターとしてソース データベース内のストアド プロシージャのセットに渡されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-171">During the data export step, the cutoff time is passed as a parameter to a set of stored procedures in the source database.</span></span> <span data-ttu-id="df0fd-172">これらのストアド プロシージャは、カットオフ時間後に変更または作成されたレコードのクエリを実行します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-172">These stored procedures query for any records that were changed or created after the cutoff time.</span></span> <span data-ttu-id="df0fd-173">Sales ファクト テーブルの場合は、`LastEditedWhen` 列が使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-173">For the Sales fact table, the `LastEditedWhen` column is used.</span></span> <span data-ttu-id="df0fd-174">ディメンション データの場合は、システムでバージョン管理されたテンポラル テーブルが使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-174">For the dimension data, system-versioned temporal tables are used.</span></span>

3. <span data-ttu-id="df0fd-175">データの移行が完了したら、カットオフ時間を格納するテーブルを更新します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-175">When the data migration is complete, update the table that stores the cutoff times.</span></span>

<span data-ttu-id="df0fd-176">ELT の実行ごとに*系列*を記録することも便利です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-176">It's also useful to record a *lineage* for each ELT run.</span></span> <span data-ttu-id="df0fd-177">特定のレコードについて、系列はそのレコードをそのデータを生成した ELT 実行と関連付けます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-177">For a given record, the lineage associates that record with the ELT run that produced the data.</span></span> <span data-ttu-id="df0fd-178">ETL 実行ごとに、読み込みの開始時刻と終了時刻が表示された新しい系列レコードがテーブルごとに作成されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-178">For each ETL run, a new lineage record is created for every table, showing the starting and ending load times.</span></span> <span data-ttu-id="df0fd-179">各レコードの系列キーは、ディメンション テーブルとファクト テーブルに格納されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-179">The lineage keys for each record are stored in the dimension and fact tables.</span></span>

![City ディメンション テーブルのスクリーンショット](./images/city-dimension-table.png)

<span data-ttu-id="df0fd-181">新しいデータのバッチがウェアハウスに読み込まれたら、Analysis Services 表形式モデルを更新します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-181">After a new batch of data is loaded into the warehouse, refresh the Analysis Services tabular model.</span></span> <span data-ttu-id="df0fd-182">「[REST API を使用した非同期更新](/azure/analysis-services/analysis-services-async-refresh)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-182">See [Asynchronous refresh with the REST API](/azure/analysis-services/analysis-services-async-refresh).</span></span>

## <a name="data-cleansing"></a><span data-ttu-id="df0fd-183">データ クレンジング</span><span class="sxs-lookup"><span data-stu-id="df0fd-183">Data cleansing</span></span>

<span data-ttu-id="df0fd-184">データ クレンジングは、ELT プロセスに含めるようにします。</span><span class="sxs-lookup"><span data-stu-id="df0fd-184">Data cleansing should be part of the ELT process.</span></span> <span data-ttu-id="df0fd-185">この参照アーキテクチャでは、不適切なデータ ソースの 1 つが市区町村の人口テーブルです。このテーブルには、データを入手できなかったなどの理由で、人口が 0 の市区町村がいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-185">In this reference architecture, one source of bad data is the city population table, where some cities have zero population, perhaps because no data was available.</span></span> <span data-ttu-id="df0fd-186">処理中、ELT パイプラインは、市区町村の人口テーブルからそれらの市区町村を削除します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-186">During processing, the ELT pipeline removes those cities from the city population table.</span></span> <span data-ttu-id="df0fd-187">外部テーブルではなくステージング テーブルにタイしてデータ クレンジングを実行します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-187">Perform data cleansing on staging tables, rather than external tables.</span></span>

<span data-ttu-id="df0fd-188">市区町村の人口テーブルから人口が 0 の市区町村を削除するストアド プロシージャを次に示します</span><span class="sxs-lookup"><span data-stu-id="df0fd-188">Here is the stored procedure that removes the cities with zero population from the City Population table.</span></span> <span data-ttu-id="df0fd-189">(ソース ファイルについては[こちら](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql)を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="df0fd-189">(You can find the source file [here](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span></span>

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a><span data-ttu-id="df0fd-190">外部データ ソース</span><span class="sxs-lookup"><span data-stu-id="df0fd-190">External data sources</span></span>

<span data-ttu-id="df0fd-191">データ ウェアハウスは、多くの場合、複数のソースのデータを統合します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-191">Data warehouses often consolidate data from multiple sources.</span></span> <span data-ttu-id="df0fd-192">この参照アーキテクチャは、人口統計データを含む外部データ ソースを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-192">This reference architecture loads an external data source that contains demographics data.</span></span> <span data-ttu-id="df0fd-193">このデータセットは、[WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) サンプルの一部として AzureBlob Storage で利用できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-193">This dataset is available in Azure blob storage as part of the [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) sample.</span></span>

<span data-ttu-id="df0fd-194">Azure Data Factory は、[BLOB ストレージ コネクタ](/azure/data-factory/connector-azure-blob-storage)を使用して、BLOB ストレージから直接コピーできます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-194">Azure Data Factory can copy directly from blob storage, using the [blob storage connector](/azure/data-factory/connector-azure-blob-storage).</span></span> <span data-ttu-id="df0fd-195">ただし、コネクタには接続文字列または共有アクセス署名が必要なため、パブリック読み取りアクセス権がある BLOB のコピーに使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="df0fd-195">However, the connector requires a connection string or a shared access signature, so it can't be used to copy a blob with public read access.</span></span> <span data-ttu-id="df0fd-196">回避策として、PolyBase を使用して BLOB ストレージ上に外部テーブルを作成し、外部テーブルを SQL Data Warehouse にコピーすることができます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-196">As a workaround, you can use PolyBase to create an external table over Blob storage and then copy the external tables into SQL Data Warehouse.</span></span>

## <a name="handling-large-binary-data"></a><span data-ttu-id="df0fd-197">大きなバイナリ データの処理</span><span class="sxs-lookup"><span data-stu-id="df0fd-197">Handling large binary data</span></span>

<span data-ttu-id="df0fd-198">ソース データベースでは、Cities テーブルに [geography](/sql/t-sql/spatial-geography/spatial-types-geography) 空間データ型を保持する Location 列があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-198">In the source database, the Cities table has a Location column that holds a [geography](/sql/t-sql/spatial-geography/spatial-types-geography) spatial data type.</span></span> <span data-ttu-id="df0fd-199">SQL Data Warehouse は **geography** 型をネイティブでサポートしていないので、このフィールドは読み込み中に **varbinary** 型に変換されます</span><span class="sxs-lookup"><span data-stu-id="df0fd-199">SQL Data Warehouse doesn't support the **geography** type natively, so this field is converted to a **varbinary** type during loading.</span></span> <span data-ttu-id="df0fd-200">(「[サポートされていないデータ型の対処法](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types)」を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="df0fd-200">(See [Workarounds for unsupported data types](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span></span>

<span data-ttu-id="df0fd-201">ただし、PolyBase でサポートされる最大列サイズは `varbinary(8000)` です。そのため、一部のデータが切り捨てられる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-201">However, PolyBase supports a maximum column size of `varbinary(8000)`, which means some data could be truncated.</span></span> <span data-ttu-id="df0fd-202">この問題の回避策は、次のようにエクスポート時にデータをチャンクに分割し、チャンクを再構成することです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-202">A workaround for this problem is to break the data up into chunks during export, and then reassemble the chunks, as follows:</span></span>

1. <span data-ttu-id="df0fd-203">Location 列の一時ステージング テーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-203">Create a temporary staging table for the Location column.</span></span>

2. <span data-ttu-id="df0fd-204">市区町村ごとに場所データを 8,000 バイトのチャンクに分割します。その結果、市区町村ごとに 1 &ndash; N 行になります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-204">For each city, split the location data into 8000-byte chunks, resulting in 1 &ndash; N rows for each city.</span></span>

3. <span data-ttu-id="df0fd-205">チャンクを再構成するには、T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) 演算子を使用して行を列に変換し、各市区町村の列の値を連結します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-205">To reassemble the chunks, use the T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operator to convert rows into columns and then concatenate the column values for each city.</span></span>

<span data-ttu-id="df0fd-206">課題は、地理データのサイズに応じて、各市区町村を異なる数の行に分割することです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-206">The challenge is that each city will be split into a different number of rows, depending on the size of geography data.</span></span> <span data-ttu-id="df0fd-207">PIVOT 演算子が機能するには、各市区町村を同じ行数にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-207">For the PIVOT operator to work, every city must have the same number of rows.</span></span> <span data-ttu-id="df0fd-208">この作業を行うために、T-SQL クエリ ([こちら][MergeLocation]を参照してください) で、ピボットの後にすべての市区町村が同じ列数になるように、空白値で行を埋める処理を実行します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-208">To make this work, the T-SQL query (which you can view [here][MergeLocation]) does some tricks to pad out the rows with blank values, so that every city has the same number of columns after the pivot.</span></span> <span data-ttu-id="df0fd-209">結果のクエリは、一度に 1 行ずつループするよりもはるかに高速になっていることがわかります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-209">The resulting query turns out to be much faster than looping through the rows one at a time.</span></span>

<span data-ttu-id="df0fd-210">同じアプローチが画像データに使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-210">The same approach is used for image data.</span></span>

## <a name="slowly-changing-dimensions"></a><span data-ttu-id="df0fd-211">緩やかに変化するディメンション</span><span class="sxs-lookup"><span data-stu-id="df0fd-211">Slowly changing dimensions</span></span>

<span data-ttu-id="df0fd-212">ディメンション データは比較的静的ですが、変化する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-212">Dimension data is relatively static, but it can change.</span></span> <span data-ttu-id="df0fd-213">たとえば、製品が別の製品カテゴリに再割り当てされることがあります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-213">For example, a product might get reassigned to a different product category.</span></span> <span data-ttu-id="df0fd-214">緩やかに変化するディメンションを処理するにはいくつかのアプローチがあります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-214">There are several approaches to handling slowly changing dimensions.</span></span> <span data-ttu-id="df0fd-215">[タイプ 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row) と呼ばれる一般的な手法では、ディメンションが変化するたびに新しいレコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-215">A common technique, called [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), is to add a new record whenever a dimension changes.</span></span>

<span data-ttu-id="df0fd-216">タイプ 2 アプローチを実装するには、特定のレコードの有効な日付範囲を指定する列をディメンション テーブルに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-216">In order to implement the Type 2 approach, dimension tables need additional columns that specify the effective date range for a given record.</span></span> <span data-ttu-id="df0fd-217">また、ソース データベースの主キーが複製されるので、ディメンション テーブルには人工主キーが必要です。</span><span class="sxs-lookup"><span data-stu-id="df0fd-217">Also, primary keys from the source database will be duplicated, so the dimension table must have an artificial primary key.</span></span>

<span data-ttu-id="df0fd-218">次の図は、Dimension.City テーブルを示しています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-218">The following image shows the Dimension.City table.</span></span> <span data-ttu-id="df0fd-219">`WWI City ID` 列は、ソース データベースの主キーです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-219">The `WWI City ID` column is the primary key from the source database.</span></span> <span data-ttu-id="df0fd-220">`City Key` 列は、ETL パイプラインで生成された人工キーです。</span><span class="sxs-lookup"><span data-stu-id="df0fd-220">The `City Key` column is an artificial key generated during the ETL pipeline.</span></span> <span data-ttu-id="df0fd-221">また、テーブルには `Valid From` 列と `Valid To` 列があり、各行が有効なときの範囲が定義されています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-221">Also notice that the table has `Valid From` and `Valid To` columns, which define the range when each row was valid.</span></span> <span data-ttu-id="df0fd-222">現在の値には `Valid To` = '9999-12-31' があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-222">Current values have a `Valid To` equal to '9999-12-31'.</span></span>

![City ディメンション テーブルのスクリーンショット](./images/city-dimension-table.png)

<span data-ttu-id="df0fd-224">このアプローチの利点は、過去のデータが保存されることです。このデータは分析に役立つ可能性があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-224">The advantage of this approach is that it preserves historical data, which can be valuable for analysis.</span></span> <span data-ttu-id="df0fd-225">ただし、これは同じエンティティに対して複数の行が存在することも意味します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-225">However, it also means there will be multiple rows for the same entity.</span></span> <span data-ttu-id="df0fd-226">たとえば、`WWI City ID` = 28561 に一致するレコードを次に示します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-226">For example, here are the records that match `WWI City ID` = 28561:</span></span>

![City ディメンション テーブルの 2 つ目のスクリーンショット](./images/city-dimension-table-2.png)

<span data-ttu-id="df0fd-228">各 Sales ファクトについて、そのファクトを請求書の日付に対応する City ディメンション テーブルの単一の行に関連付ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-228">For each Sales fact, you want to associate that fact with a single row in City dimension table, corresponding to the invoice date.</span></span> <span data-ttu-id="df0fd-229">ETL プロセスの一環として、追加の列を作成します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-229">As part of the ETL process, create an additional column that</span></span> 

<span data-ttu-id="df0fd-230">次の T-SQL クエリは、各請求書を City ディメンション テーブルの正しい City Key に関連付けるテンポラル テーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-230">The following T-SQL query creates a temporary table that associates each invoice with the correct City Key from the City dimension table.</span></span>

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

<span data-ttu-id="df0fd-231">このテーブルは、Sales ファクト テーブルの列を設定するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-231">This table is used to populate a column in the Sales fact table:</span></span>

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

<span data-ttu-id="df0fd-232">この列では、Power BI クエリを使用して、特定の売上請求書に対して正しい City レコードを検索できます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-232">This column enables a Power BI query to find the correct City record for a given sales invoice.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="df0fd-233">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="df0fd-233">Security considerations</span></span>

<span data-ttu-id="df0fd-234">さらにセキュリティを強化するには、[仮想ネットワーク サービス エンドポイント](/azure/virtual-network/virtual-network-service-endpoints-overview)を使用して、仮想ネットワークに対してのみ Azure サービス リソースをセキュリティで保護することができます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-234">For additional security, you can use [Virtual Network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview) to secure Azure service resources to only your virtual network.</span></span> <span data-ttu-id="df0fd-235">これにより、これらのリソースに対するパブリック インターネット アクセスが完全に削除され、自分の仮想ネットワークからのトラフィックのみが許可されます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-235">This fully removes public Internet access to those resources, allowing traffic only from your virtual network.</span></span>

<span data-ttu-id="df0fd-236">このアプローチでは、Azure 内に VNet を作成し、Azure サービス用のプライベート サービス エンドポイントを作成します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-236">With this approach, you create a VNet in Azure and then create private service endpoints for Azure services.</span></span> <span data-ttu-id="df0fd-237">これらのサービスは、その仮想ネットワークからのトラフィックに制限されるようになります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-237">Those services are then restricted to traffic from that virtual network.</span></span> <span data-ttu-id="df0fd-238">オンプレミス ネットワークからゲートウェイ経由でアクセスすることもできます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-238">You can also reach them from your on-premises network through a gateway.</span></span>

<span data-ttu-id="df0fd-239">次の制限事項に注意してください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-239">Be aware of the following limitations:</span></span>

- <span data-ttu-id="df0fd-240">この参照アーキテクチャが作成された時点で、Azure Storage と Azure SQL Data Warehouse では VNet サービス エンドポイントがサポートされていますが、Azure Analysis Service ではサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="df0fd-240">At the time this reference architecture was created, VNet service endpoints are supported for Azure Storage and Azure SQL Data Warehouse, but not for Azure Analysis Service.</span></span> <span data-ttu-id="df0fd-241">最新の状態については、[こちら](https://azure.microsoft.com/updates/?product=virtual-network)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-241">Check the latest status [here](https://azure.microsoft.com/updates/?product=virtual-network).</span></span>

- <span data-ttu-id="df0fd-242">Azure Storage でサービス エンドポイントが有効な場合、PolyBase は Storage から SQL Data Warehouse にデータをコピーできません。</span><span class="sxs-lookup"><span data-stu-id="df0fd-242">If service endpoints are enabled for Azure Storage, PolyBase cannot copy data from Storage into SQL Data Warehouse.</span></span> <span data-ttu-id="df0fd-243">この問題には軽減策があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-243">There is a mitigation for this issue.</span></span> <span data-ttu-id="df0fd-244">詳細については、「[Azure Storage で VNet サービス エンドポイントを使用した場合の影響](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-244">For more information, see [Impact of using VNet Service Endpoints with Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span></span>

- <span data-ttu-id="df0fd-245">オンプレミスから Azure Storage にデータを移動するには、オンプレミスまたは ExpressRoute からのパブリック IP アドレスをホワイトリストに登録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="df0fd-245">To move data from on-premises into Azure Storage, you will need to whitelist public IP addresses from your on-premises or ExpressRoute.</span></span> <span data-ttu-id="df0fd-246">詳細については、「[Azure サービスへのアクセスを仮想ネットワークに限定する](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-246">For details, see [Securing Azure services to virtual networks](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span></span>

- <span data-ttu-id="df0fd-247">Analysis Services が SQL Data Warehouse からデータを読み取ることができるようにするには、SQL Data Warehouse サービス エンドポイントを含む仮想ネットワークに Windows VM を展開します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-247">To enable Analysis Services to read data from SQL Data Warehouse, deploy a Windows VM to the virtual network that contains the SQL Data Warehouse service endpoint.</span></span> <span data-ttu-id="df0fd-248">この VM に [Azure オンプレミス データゲートウェイ](/azure/analysis-services/analysis-services-gateway)をインストールします。</span><span class="sxs-lookup"><span data-stu-id="df0fd-248">Install [Azure On-premises Data Gateway](/azure/analysis-services/analysis-services-gateway) on this VM.</span></span> <span data-ttu-id="df0fd-249">次に Azure Analysis サービスをデータ ゲートウェイに接続します。</span><span class="sxs-lookup"><span data-stu-id="df0fd-249">Then connect your Azure Analysis service to the data gateway.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="df0fd-250">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="df0fd-250">Deploy the solution</span></span>

<span data-ttu-id="df0fd-251">リファレンス実装をデプロイおよび実行するには、[GitHub readme][github] の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="df0fd-251">To the deploy and run the reference implementation, follow the steps in the [GitHub readme][github].</span></span> <span data-ttu-id="df0fd-252">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-252">It deploys the following:</span></span>

- <span data-ttu-id="df0fd-253">オンプレミスのデータベース サーバーをシミュレートする Windows VM。</span><span class="sxs-lookup"><span data-stu-id="df0fd-253">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="df0fd-254">これには、SQL Server 2017 と関連ツール、および Power BI Desktop が含まれています。</span><span class="sxs-lookup"><span data-stu-id="df0fd-254">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
- <span data-ttu-id="df0fd-255">SQL Server データベースからエクスポートされたデータを保持する Blob Storage を提供する Azure ストレージ アカウント。</span><span class="sxs-lookup"><span data-stu-id="df0fd-255">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
- <span data-ttu-id="df0fd-256">Azure SQL Data Warehouse インスタンス。</span><span class="sxs-lookup"><span data-stu-id="df0fd-256">An Azure SQL Data Warehouse instance.</span></span>
- <span data-ttu-id="df0fd-257">Azure Analysis Services インスタンス。</span><span class="sxs-lookup"><span data-stu-id="df0fd-257">An Azure Analysis Services instance.</span></span>
- <span data-ttu-id="df0fd-258">ELT ジョブ用の Azure Data Factory と Data Factory パイプライン。</span><span class="sxs-lookup"><span data-stu-id="df0fd-258">Azure Data Factory and the Data Factory pipeline for the ELT job.</span></span>

## <a name="related-resources"></a><span data-ttu-id="df0fd-259">関連リソース</span><span class="sxs-lookup"><span data-stu-id="df0fd-259">Related resources</span></span>

<span data-ttu-id="df0fd-260">同じテクノロジの一部を使用する具体的なソリューションを示す次の [Azure のサンプル シナリオ](/azure/architecture/example-scenario)をレビューできます。</span><span class="sxs-lookup"><span data-stu-id="df0fd-260">You may want to review the following [Azure example scenarios](/azure/architecture/example-scenario) that demonstrate specific solutions using some of the same technologies:</span></span>

- [<span data-ttu-id="df0fd-261">販売およびマーケティング向けのデータ ウェアハウスと分析</span><span class="sxs-lookup"><span data-stu-id="df0fd-261">Data warehousing and analytics for sales and marketing</span></span>](/azure/architecture/example-scenario/data/data-warehouse)
- [<span data-ttu-id="df0fd-262">既存のオンプレミス SSIS と Azure Data Factory を使用したハイブリッド ETL</span><span class="sxs-lookup"><span data-stu-id="df0fd-262">Hybrid ETL with existing on-premises SSIS and Azure Data Factory</span></span>](/azure/architecture/example-scenario/data/hybrid-etl-with-adf)

<!-- links -->

[adf]: /azure/data-factory
[github]: https://github.com/mspnp/azure-data-factory-sqldw-elt-pipeline
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
