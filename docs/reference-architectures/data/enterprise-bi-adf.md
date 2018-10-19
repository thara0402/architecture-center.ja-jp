---
title: SQL Data Warehouse と Azure Data Factory を使用したエンタープライズ BI の自動化
description: Azure Data Factory を使用した Azure 上の ELT ワークフローの自動化
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: f004c02da93335e74b07b9720236832ad7f744db
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876904"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a><span data-ttu-id="0c25b-103">SQL Data Warehouse と Azure Data Factory を使用したエンタープライズ BI の自動化</span><span class="sxs-lookup"><span data-stu-id="0c25b-103">Automated enterprise BI with SQL Data Warehouse and Azure Data Factory</span></span>

<span data-ttu-id="0c25b-104">この参照アーキテクチャは、[ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (抽出 - 読み込み - 変換) パイプラインで段階的に読み込む方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-104">This reference architecture shows how to perform incremental loading in an [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline.</span></span> <span data-ttu-id="0c25b-105">Azure Data Factory を使用して ELT パイプラインを自動化します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-105">It uses Azure Data Factory to automate the ELT pipeline.</span></span> <span data-ttu-id="0c25b-106">パイプラインは、オンプレミスの SQL Server データベースから SQL Data Warehouse に最新の OLTP データを段階的に移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-106">The pipeline incrementally moves the latest OLTP data from an on-premises SQL Server database into SQL Data Warehouse.</span></span> <span data-ttu-id="0c25b-107">トランザクション データは、分析のために表形式モデルに変換されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-107">Transactional data is transformed into a tabular model for analysis.</span></span> [<span data-ttu-id="0c25b-108">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

<span data-ttu-id="0c25b-109">このアーキテクチャは、[SQL Data Warehouse を使用したエンタープライズ向け BI](./enterprise-bi-sqldw.md) に示されているアーキテクチャに基づいて構築されていますが、エンタープライズ データ ウェアハウス シナリオに重要な機能がいくつか追加されています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-109">This architecture builds on the one shown in [Enterprise BI with SQL Data Warehouse](./enterprise-bi-sqldw.md), but adds some features that are important for enterprise data warehousing scenarios.</span></span>

-   <span data-ttu-id="0c25b-110">Data Factory を使用したパイプラインの自動化</span><span class="sxs-lookup"><span data-stu-id="0c25b-110">Automation of the pipeline using Data Factory.</span></span>
-   <span data-ttu-id="0c25b-111">段階的な読み込み。</span><span class="sxs-lookup"><span data-stu-id="0c25b-111">Incremental loading.</span></span>
-   <span data-ttu-id="0c25b-112">複数のデータ ソースの統合。</span><span class="sxs-lookup"><span data-stu-id="0c25b-112">Integrating multiple data sources.</span></span>
-   <span data-ttu-id="0c25b-113">地理空間データや画像などのバイナリ データの読み込み。</span><span class="sxs-lookup"><span data-stu-id="0c25b-113">Loading binary data such as geospatial data and images.</span></span>

## <a name="architecture"></a><span data-ttu-id="0c25b-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="0c25b-114">Architecture</span></span>

<span data-ttu-id="0c25b-115">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-115">The architecture consists of the following components.</span></span>

### <a name="data-sources"></a><span data-ttu-id="0c25b-116">データ ソース</span><span class="sxs-lookup"><span data-stu-id="0c25b-116">Data sources</span></span>

<span data-ttu-id="0c25b-117">**オンプレミスの SQL Server**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-117">**On-premises SQL Server**.</span></span> <span data-ttu-id="0c25b-118">ソース データは、オンプレミスの SQL Server データベースにあります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-118">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="0c25b-119">オンプレミス環境をシミュレートするために、このアーキテクチャのデプロイ スクリプトでは、SQL Server がインストールされた Azure の仮想マシンがプロビジョニングされます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-119">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> <span data-ttu-id="0c25b-120">[Wide World Importers OLTP サンプル データベース][wwi]は、ソース データベースとして使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-120">The [Wide World Importers OLTP sample database][wwi] is used as the source database.</span></span>

<span data-ttu-id="0c25b-121">**外部データ**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-121">**External data**.</span></span> <span data-ttu-id="0c25b-122">データ ウェアハウスの一般的なシナリオとして、複数のデータ ソースの統合があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-122">A common scenario for data warehouses is to integrate multiple data sources.</span></span> <span data-ttu-id="0c25b-123">この参照アーキテクチャは、年別の市区町村の人口を含む外部データ セットを読み込み、それを OLTP データベースのデータと統合します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-123">This reference architecture loads an external data set that contains city populations by year, and integrates it with the data from the OLTP database.</span></span> <span data-ttu-id="0c25b-124">このデータは、"各地域の売上高の増加率は人口の増加率と一致するか、上回っているか" のような分析に使用できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-124">You can use this data for insights such as: "Does sales growth in each region match or exceed population growth?"</span></span>

### <a name="ingestion-and-data-storage"></a><span data-ttu-id="0c25b-125">インジェストとデータ ストレージ</span><span class="sxs-lookup"><span data-stu-id="0c25b-125">Ingestion and data storage</span></span>

<span data-ttu-id="0c25b-126">**Blob Storage**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-126">**Blob Storage**.</span></span> <span data-ttu-id="0c25b-127">Blob Storage は、SQL Data Warehouse に読み込む前のソース データのステージング領域として使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-127">Blob storage is used as a staging area for the source data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="0c25b-128">**Azure SQL Data Warehouse**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-128">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="0c25b-129">[SQL Data Warehouse](/azure/sql-data-warehouse/) は、大規模なデータの分析を目的として設計された分散システムです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-129">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="0c25b-130">超並列処理 (MPP) がサポートされているので、ハイパフォーマンス分析の実行に適しています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-130">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span> 

<span data-ttu-id="0c25b-131">**Azure Data Factory**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-131">**Azure Data Factory**.</span></span> <span data-ttu-id="0c25b-132">[Data Factory][adf] は、データ移動とデータ変換を調整し、自動化するマネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-132">[Data Factory][adf] is a managed service that orchestrates and automates data movement and data transformation.</span></span> <span data-ttu-id="0c25b-133">このアーキテクチャでは、ELT プロセスのさまざまな段階が調整されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-133">In this architecture, it coordinates the various stages of the ELT process.</span></span>

### <a name="analysis-and-reporting"></a><span data-ttu-id="0c25b-134">分析とレポート</span><span class="sxs-lookup"><span data-stu-id="0c25b-134">Analysis and reporting</span></span>

<span data-ttu-id="0c25b-135">**Azure Analysis Services**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-135">**Azure Analysis Services**.</span></span> <span data-ttu-id="0c25b-136">[Analysis Services](/azure/analysis-services/) は、データ モデリング機能を提供するフル マネージド サービスです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-136">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="0c25b-137">セマンティック モデルは Analysis Services に読み込まれます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-137">The semantic model is loaded into Analysis Services.</span></span>

<span data-ttu-id="0c25b-138">**Power BI**。</span><span class="sxs-lookup"><span data-stu-id="0c25b-138">**Power BI**.</span></span> <span data-ttu-id="0c25b-139">Power BI は、データを分析してビジネスの分析情報を得る一連のビジネス分析ツールです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-139">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="0c25b-140">このアーキテクチャでは、Analysis Services に格納されたセマンティック モデルに対してクエリが実行されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-140">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

### <a name="authentication"></a><span data-ttu-id="0c25b-141">Authentication</span><span class="sxs-lookup"><span data-stu-id="0c25b-141">Authentication</span></span>

<span data-ttu-id="0c25b-142">**Azure Active Directory** (Azure AD)。Power BI から Analysis Services サーバーに接続するユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-142">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

<span data-ttu-id="0c25b-143">Data Factory では、Azure AD を使用し、サービス プリンシパルまたはマネージド サービス ID (MSI) を使用して SQL Data Warehouse を認証することもできます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-143">Data Factory can use also use Azure AD to authenticate to SQL Data Warehouse, by using a service principal or Managed Service Identity (MSI).</span></span> <span data-ttu-id="0c25b-144">わかりやすくするために、この展開例では SQL Server 認証を使用しています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-144">For simplicity, the example deployment uses SQL Server authentication.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="0c25b-145">データ パイプライン</span><span class="sxs-lookup"><span data-stu-id="0c25b-145">Data pipeline</span></span>

<span data-ttu-id="0c25b-146">[Azure Data Factory][adf] では、パイプラインはタスク (この例では、データを SQL Data Warehouse に読み込んで変換するタスク) の調整に使用されるアクティビティの論理グループです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-146">In [Azure Data Factory][adf], a pipeline is a logical grouping of activities used to coordinate a task &mdash; in this case, loading and transforming data into SQL Data Warehouse.</span></span> 

<span data-ttu-id="0c25b-147">この参照アーキテクチャは、一連の子パイプラインを実行するマスター パイプラインを定義します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-147">This reference architecture defines a master pipeline that runs a sequence of child pipelines.</span></span> <span data-ttu-id="0c25b-148">各子パイプラインは、1 つまたは複数のデータ ウェアハウス テーブルにデータを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-148">Each child pipeline loads data into one or more data warehouse tables.</span></span>

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a><span data-ttu-id="0c25b-149">段階的な読み込み</span><span class="sxs-lookup"><span data-stu-id="0c25b-149">Incremental loading</span></span>

<span data-ttu-id="0c25b-150">自動化された ETL または ELT プロセスを実行する場合、以前の実行以降に変更されたデータのみを読み込むのが最も効率的です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-150">When you run an automated ETL or ELT process, it's most efficient to load only the data that changed since the previous run.</span></span> <span data-ttu-id="0c25b-151">すべてのデータを読み込む完全読み込みとは対照的に、これは*段階的な読み込み*と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-151">This is called an *incremental load*, as opposed to a full load that loads all of the data.</span></span> <span data-ttu-id="0c25b-152">段階的な読み込みを実行するには、変更されたデータを特定する方法が必要です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-152">To perform an incremental load, you need a way to identify which data has changed.</span></span> <span data-ttu-id="0c25b-153">最も一般的な方法は、*高基準値*を使用することです。つまり、ソース テーブルの特定の列の最新の値 (日時列または一意の整数列) を追跡することです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-153">The most common approach is to use a *high water mark* value, which means tracking the latest value of some column in the source table, either a datetime column or a unique integer column.</span></span> 

<span data-ttu-id="0c25b-154">SQL Server 2016 以降は[テンポラル テーブル](/sql/relational-databases/tables/temporal-tables)を使用できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-154">Starting with SQL Server 2016, you can use [temporal tables](/sql/relational-databases/tables/temporal-tables).</span></span> <span data-ttu-id="0c25b-155">テンポラル テーブルは、データの完全な変更履歴が保持されている、システムでバージョン管理されたテーブルです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-155">These are system-versioned tables that keep a full history of data changes.</span></span> <span data-ttu-id="0c25b-156">データベース エンジンは、各変更履歴を別々の履歴テーブルに自動的に記録します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-156">The database engine automatically records the history of every change in a separate history table.</span></span> <span data-ttu-id="0c25b-157">クエリに FOR SYSTEM_TIME 句を追加することで、履歴データのクエリを実行することができます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-157">You can query the historical data by adding a FOR SYSTEM_TIME clause to a query.</span></span> <span data-ttu-id="0c25b-158">内部的には、データベース エンジンは履歴テーブルのクエリを実行しますが、この処理はアプリケーションにとって透過的です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-158">Internally, the database engine queries the history table, but this is transparent to the application.</span></span> 

> [!NOTE]
> <span data-ttu-id="0c25b-159">以前のバージョンの SQL Server では、[変更データ キャプチャ](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC) を使用できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-159">For earlier versions of SQL Server, you can use [Change Data Capture](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span></span> <span data-ttu-id="0c25b-160">このアプローチでは、別の変更テーブルのクエリを実行する必要があり、変更がタイムスタンプではなくログ シーケンス番号で追跡されるため、テンポラル テーブルよりも不便です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-160">This approach is less convenient than temporal tables, because you have to query a separate change table, and changes are tracked by a log sequence number, rather than a timestamp.</span></span> 

<span data-ttu-id="0c25b-161">テンポラル テーブルは、時間と共に変化する可能性のあるディメンション データの場合に便利です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-161">Temporal tables are useful for dimension data, which can change over time.</span></span> <span data-ttu-id="0c25b-162">通常、ファクト テーブルは、販売などの不変トランザクションを表します。この場合、システムのバージョン履歴を保持することは意味がありません。</span><span class="sxs-lookup"><span data-stu-id="0c25b-162">Fact tables usually represent an immutable transaction such as a sale, in which case keeping the system version history doesn't make sense.</span></span> <span data-ttu-id="0c25b-163">その代わり、通常、トランザクションにはトランザクション日付を表す列があります。これをウォーターマーク値として使用できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-163">Instead, transactions usually have a column that represents the transaction date, which can be used as the watermark value.</span></span> <span data-ttu-id="0c25b-164">たとえば、Wide World Importers OLTP データベースでは、Sales.Invoices テーブルと Sales.InvoiceLines テーブルに既定が `sysdatetime()` の `LastEditedWhen` フィールドがあります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-164">For example, in the Wide World Importers OLTP database, the Sales.Invoices and Sales.InvoiceLines tables have a `LastEditedWhen` field that defaults to `sysdatetime()`.</span></span> 

<span data-ttu-id="0c25b-165">ELT パイプラインの一般的なフローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-165">Here is the general flow for the ELT pipeline:</span></span>

1. <span data-ttu-id="0c25b-166">ソース データベースの各テーブルについて、最後の ELT ジョブが実行されたときのカットオフ時間を追跡します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-166">For each table in the source database, track the cutoff time when the last ELT job ran.</span></span> <span data-ttu-id="0c25b-167">この情報をデータ ウェアハウスに格納します</span><span class="sxs-lookup"><span data-stu-id="0c25b-167">Store this information in the data warehouse.</span></span> <span data-ttu-id="0c25b-168">(初期設定では、すべての時間が '1-1-1900'に設定されます)。</span><span class="sxs-lookup"><span data-stu-id="0c25b-168">(On initial setup, all times are set to '1-1-1900'.)</span></span>

2. <span data-ttu-id="0c25b-169">データのエクスポート手順では、カットオフ時間がパラメーターとしてソース データベース内のストアド プロシージャのセットに渡されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-169">During the data export step, the cutoff time is passed as a parameter to a set of stored procedures in the source database.</span></span> <span data-ttu-id="0c25b-170">これらのストアド プロシージャは、カットオフ時間後に変更または作成されたレコードのクエリを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-170">These stored procedures query for any records that were changed or created after the cutoff time.</span></span> <span data-ttu-id="0c25b-171">Sales ファクト テーブルの場合は、`LastEditedWhen` 列が使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-171">For the Sales fact table, the `LastEditedWhen` column is used.</span></span> <span data-ttu-id="0c25b-172">ディメンション データの場合は、システムでバージョン管理されたテンポラル テーブルが使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-172">For the dimension data, system-versioned temporal tables are used.</span></span>

3. <span data-ttu-id="0c25b-173">データの移行が完了したら、カットオフ時間を格納するテーブルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-173">When the data migration is complete, update the table that stores the cutoff times.</span></span>

<span data-ttu-id="0c25b-174">ELT の実行ごとに*系列*を記録することも便利です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-174">It's also useful to record a *lineage* for each ELT run.</span></span> <span data-ttu-id="0c25b-175">特定のレコードについて、系列はそのレコードをそのデータを生成した ELT 実行と関連付けます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-175">For a given record, the lineage associates that record with the ELT run that produced the data.</span></span> <span data-ttu-id="0c25b-176">ETL 実行ごとに、読み込みの開始時刻と終了時刻が表示された新しい系列レコードがテーブルごとに作成されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-176">For each ETL run, a new lineage record is created for every table, showing the starting and ending load times.</span></span> <span data-ttu-id="0c25b-177">各レコードの系列キーは、ディメンション テーブルとファクト テーブルに格納されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-177">The lineage keys for each record are stored in the dimension and fact tables.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="0c25b-178">新しいデータのバッチがウェアハウスに読み込まれたら、Analysis Services 表形式モデルを更新します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-178">After a new batch of data is loaded into the warehouse, refresh the Analysis Services tabular model.</span></span> <span data-ttu-id="0c25b-179">「[REST API を使用した非同期更新](/azure/analysis-services/analysis-services-async-refresh)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-179">See [Asynchronous refresh with the REST API](/azure/analysis-services/analysis-services-async-refresh).</span></span>

## <a name="data-cleansing"></a><span data-ttu-id="0c25b-180">データ クレンジング</span><span class="sxs-lookup"><span data-stu-id="0c25b-180">Data cleansing</span></span>

<span data-ttu-id="0c25b-181">データ クレンジングは、ELT プロセスに含めるようにします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-181">Data cleansing should be part of the ELT process.</span></span> <span data-ttu-id="0c25b-182">この参照アーキテクチャでは、不適切なデータ ソースの 1 つが市区町村の人口テーブルです。このテーブルには、データを入手できなかったなどの理由で、人口が 0 の市区町村がいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-182">In this reference architecture, one source of bad data is the city population table, where some cities have zero population, perhaps because no data was available.</span></span> <span data-ttu-id="0c25b-183">処理中、ELT パイプラインは、市区町村の人口テーブルからそれらの市区町村を削除します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-183">During processing, the ELT pipeline removes those cities from the city population table.</span></span> <span data-ttu-id="0c25b-184">外部テーブルではなくステージング テーブルにタイしてデータ クレンジングを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-184">Perform data cleansing on staging tables, rather than external tables.</span></span>

<span data-ttu-id="0c25b-185">市区町村の人口テーブルから人口が 0 の市区町村を削除するストアド プロシージャを次に示します</span><span class="sxs-lookup"><span data-stu-id="0c25b-185">Here is the stored procedure that removes the cities with zero population from the City Population table.</span></span> <span data-ttu-id="0c25b-186">(ソース ファイルについては[こちら](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql)を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="0c25b-186">(You can find the source file [here](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span></span> 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a><span data-ttu-id="0c25b-187">外部データ ソース</span><span class="sxs-lookup"><span data-stu-id="0c25b-187">External data sources</span></span>

<span data-ttu-id="0c25b-188">データ ウェアハウスは、多くの場合、複数のソースのデータを統合します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-188">Data warehouses often consolidate data from multiple sources.</span></span> <span data-ttu-id="0c25b-189">この参照アーキテクチャは、人口統計データを含む外部データ ソースを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-189">This reference architecture loads an external data source that contains demographics data.</span></span> <span data-ttu-id="0c25b-190">このデータセットは、[WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) サンプルの一部として AzureBlob Storage で利用できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-190">This dataset is available in Azure blob storage as part of the [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) sample.</span></span>

<span data-ttu-id="0c25b-191">Azure Data Factory は、[BLOB ストレージ コネクタ](/azure/data-factory/connector-azure-blob-storage)を使用して、BLOB ストレージから直接コピーできます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-191">Azure Data Factory can copy directly from blob storage, using the [blob storage connector](/azure/data-factory/connector-azure-blob-storage).</span></span> <span data-ttu-id="0c25b-192">ただし、コネクタには接続文字列または共有アクセス署名が必要なため、パブリック読み取りアクセス権がある BLOB のコピーに使用することはできません。</span><span class="sxs-lookup"><span data-stu-id="0c25b-192">However, the connector requires a connection string or a shared access signature, so it can't be used to copy a blob with public read access.</span></span> <span data-ttu-id="0c25b-193">回避策として、PolyBase を使用して BLOB ストレージ上に外部テーブルを作成し、外部テーブルを SQL Data Warehouse にコピーすることができます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-193">As a workaround, you can use PolyBase to create an external table over Blob storage and then copy the external tables into SQL Data Warehouse.</span></span> 

## <a name="handling-large-binary-data"></a><span data-ttu-id="0c25b-194">大きなバイナリ データの処理</span><span class="sxs-lookup"><span data-stu-id="0c25b-194">Handling large binary data</span></span> 

<span data-ttu-id="0c25b-195">ソース データベースでは、Cities テーブルに [geography](/sql/t-sql/spatial-geography/spatial-types-geography) 空間データ型を保持する Location 列があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-195">In the source database, the Cities table has a Location column that holds a [geography](/sql/t-sql/spatial-geography/spatial-types-geography) spatial data type.</span></span> <span data-ttu-id="0c25b-196">SQL Data Warehouse は **geography** 型をネイティブでサポートしていないので、このフィールドは読み込み中に **varbinary** 型に変換されます</span><span class="sxs-lookup"><span data-stu-id="0c25b-196">SQL Data Warehouse doesn't support the **geography** type natively, so this field is converted to a **varbinary** type during loading.</span></span> <span data-ttu-id="0c25b-197">(「[サポートされていないデータ型の対処法](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types)」を参照してください)。</span><span class="sxs-lookup"><span data-stu-id="0c25b-197">(See [Workarounds for unsupported data types](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span></span>

<span data-ttu-id="0c25b-198">ただし、PolyBase でサポートされる最大列サイズは `varbinary(8000)` です。そのため、一部のデータが切り捨てられる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-198">However, PolyBase supports a maximum column size of `varbinary(8000)`, which means some data could be truncated.</span></span> <span data-ttu-id="0c25b-199">この問題の回避策は、次のようにエクスポート時にデータをチャンクに分割し、チャンクを再構成することです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-199">A workaround for this problem is to break the data up into chunks during export, and then reassemble the chunks, as follows:</span></span>

1. <span data-ttu-id="0c25b-200">Location 列の一時ステージング テーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-200">Create a temporary staging table for the Location column.</span></span>

2. <span data-ttu-id="0c25b-201">市区町村ごとに場所データを 8,000 バイトのチャンクに分割します。その結果、市区町村ごとに 1 &ndash; N 行になります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-201">For each city, split the location data into 8000-byte chunks, resulting in 1 &ndash; N rows for each city.</span></span>

3. <span data-ttu-id="0c25b-202">チャンクを再構成するには、T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) 演算子を使用して行を列に変換し、各市区町村の列の値を連結します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-202">To reassemble the chunks, use the T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operator to convert rows into columns and then concatenate the column values for each city.</span></span>

<span data-ttu-id="0c25b-203">課題は、地理データのサイズに応じて、各市区町村を異なる数の行に分割することです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-203">The challenge is that each city will be split into a different number of rows, depending on the size of geography data.</span></span> <span data-ttu-id="0c25b-204">PIVOT 演算子が機能するには、各市区町村を同じ行数にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-204">For the PIVOT operator to work, every city must have the same number of rows.</span></span> <span data-ttu-id="0c25b-205">この作業を行うために、T-SQL クエリ ([こちら][MergeLocation]を参照してください) で、ピボットの後にすべての市区町村が同じ列数になるように、空白値で行を埋める処理を実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-205">To make this work, the T-SQL query (which you can view [here][MergeLocation]) does some tricks to pad out the rows with blank values, so that every city has the same number of columns after the pivot.</span></span> <span data-ttu-id="0c25b-206">結果のクエリは、一度に 1 行ずつループするよりもはるかに高速になっていることがわかります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-206">The resulting query turns out to be much faster than looping through the rows one at a time.</span></span>

<span data-ttu-id="0c25b-207">同じアプローチが画像データに使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-207">The same approach is used for image data.</span></span>

## <a name="slowly-changing-dimensions"></a><span data-ttu-id="0c25b-208">緩やかに変化するディメンション</span><span class="sxs-lookup"><span data-stu-id="0c25b-208">Slowly changing dimensions</span></span>

<span data-ttu-id="0c25b-209">ディメンション データは比較的静的ですが、変化する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-209">Dimension data is relatively static, but it can change.</span></span> <span data-ttu-id="0c25b-210">たとえば、製品が別の製品カテゴリに再割り当てされることがあります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-210">For example, a product might get reassigned to a different product category.</span></span> <span data-ttu-id="0c25b-211">緩やかに変化するディメンションを処理するにはいくつかのアプローチがあります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-211">There are several approaches to handling slowly changing dimensions.</span></span> <span data-ttu-id="0c25b-212">[タイプ 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row) と呼ばれる一般的な手法では、ディメンションが変化するたびに新しいレコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-212">A common technique, called [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), is to add a new record whenever a dimension changes.</span></span> 

<span data-ttu-id="0c25b-213">タイプ 2 アプローチを実装するには、特定のレコードの有効な日付範囲を指定する列をディメンション テーブルに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-213">In order to implement the Type 2 approach, dimension tables need additional columns that specify the effective date range for a given record.</span></span> <span data-ttu-id="0c25b-214">また、ソース データベースの主キーが複製されるので、ディメンション テーブルには人工主キーが必要です。</span><span class="sxs-lookup"><span data-stu-id="0c25b-214">Also, primary keys from the source database will be duplicated, so the dimension table must have an artificial primary key.</span></span>

<span data-ttu-id="0c25b-215">次の図は、Dimension.City テーブルを示しています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-215">The following image shows the Dimension.City table.</span></span> <span data-ttu-id="0c25b-216">`WWI City ID` 列は、ソース データベースの主キーです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-216">The `WWI City ID` column is the primary key from the source database.</span></span> <span data-ttu-id="0c25b-217">`City Key` 列は、ETL パイプラインで生成された人工キーです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-217">The `City Key` column is an artificial key generated during the ETL pipeline.</span></span> <span data-ttu-id="0c25b-218">また、テーブルには `Valid From` 列と `Valid To` 列があり、各行が有効なときの範囲が定義されています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-218">Also notice that the table has `Valid From` and `Valid To` columns, which define the range when each row was valid.</span></span> <span data-ttu-id="0c25b-219">現在の値には `Valid To` = '9999-12-31' があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-219">Current values have a `Valid To` equal to '9999-12-31'.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="0c25b-220">このアプローチの利点は、過去のデータが保存されることです。このデータは分析に役立つ可能性があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-220">The advantage of this approach is that it preserves historical data, which can be valuable for analysis.</span></span> <span data-ttu-id="0c25b-221">ただし、これは同じエンティティに対して複数の行が存在することも意味します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-221">However, it also means there will be multiple rows for the same entity.</span></span> <span data-ttu-id="0c25b-222">たとえば、`WWI City ID` = 28561 に一致するレコードを次に示します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-222">For example, here are the records that match `WWI City ID` = 28561:</span></span>

![](./images/city-dimension-table-2.png)

<span data-ttu-id="0c25b-223">各 Sales ファクトについて、そのファクトを請求書の日付に対応する City ディメンション テーブルの単一の行に関連付ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-223">For each Sales fact, you want to associate that fact with a single row in City dimension table, corresponding to the invoice date.</span></span> <span data-ttu-id="0c25b-224">ETL プロセスの一環として、追加の列を作成します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-224">As part of the ETL process, create an additional column that</span></span> 

<span data-ttu-id="0c25b-225">次の T-SQL クエリは、各請求書を City ディメンション テーブルの正しい City Key に関連付けるテンポラル テーブルを作成します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-225">The following T-SQL query creates a temporary table that associates each invoice with the correct City Key from the City dimension table.</span></span>

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

<span data-ttu-id="0c25b-226">このテーブルは、Sales ファクト テーブルの列を設定するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-226">This table is used to populate a column in the Sales fact table:</span></span>

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

<span data-ttu-id="0c25b-227">この列では、Power BI クエリを使用して、特定の売上請求書に対して正しい City レコードを検索できます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-227">This column enables a Power BI query to find the correct City record for a given sales invoice.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="0c25b-228">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="0c25b-228">Security considerations</span></span>

<span data-ttu-id="0c25b-229">さらにセキュリティを強化するには、[仮想ネットワーク サービス エンドポイント](/azure/virtual-network/virtual-network-service-endpoints-overview)を使用して、仮想ネットワークに対してのみ Azure サービス リソースをセキュリティで保護することができます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-229">For additional security, you can use [Virtual Network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview) to secure Azure service resources to only your virtual network.</span></span> <span data-ttu-id="0c25b-230">これにより、これらのリソースに対するパブリック インターネット アクセスが完全に削除され、自分の仮想ネットワークからのトラフィックのみが許可されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-230">This fully removes public Internet access to those resources, allowing traffic only from your virtual network.</span></span>

<span data-ttu-id="0c25b-231">このアプローチでは、Azure 内に VNet を作成し、Azure サービス用のプライベート サービス エンドポイントを作成します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-231">With this approach, you create a VNet in Azure and then create private service endpoints for Azure services.</span></span> <span data-ttu-id="0c25b-232">これらのサービスは、その仮想ネットワークからのトラフィックに制限されるようになります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-232">Those services are then restricted to traffic from that virtual network.</span></span> <span data-ttu-id="0c25b-233">オンプレミス ネットワークからゲートウェイ経由でアクセスすることもできます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-233">You can also reach them from your on-premises network through a gateway.</span></span>

<span data-ttu-id="0c25b-234">次の制限事項に注意してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-234">Be aware of the following limitations:</span></span>

- <span data-ttu-id="0c25b-235">この参照アーキテクチャが作成された時点で、Azure Storage と Azure SQL Data Warehouse では VNet サービス エンドポイントがサポートされていますが、Azure Analysis Service ではサポートされていません。</span><span class="sxs-lookup"><span data-stu-id="0c25b-235">At the time this reference architecture was created, VNet service endpoints are supported for Azure Storage and Azure SQL Data Warehouse, but not for Azure Analysis Service.</span></span> <span data-ttu-id="0c25b-236">最新の状態については、[こちら](https://azure.microsoft.com/updates/?product=virtual-network)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-236">Check the latest status [here](https://azure.microsoft.com/updates/?product=virtual-network).</span></span> 

- <span data-ttu-id="0c25b-237">Azure Storage でサービス エンドポイントが有効な場合、PolyBase は Storage から SQL Data Warehouse にデータをコピーできません。</span><span class="sxs-lookup"><span data-stu-id="0c25b-237">If service endpoints are enabled for Azure Storage, PolyBase cannot copy data from Storage into SQL Data Warehouse.</span></span> <span data-ttu-id="0c25b-238">この問題には軽減策があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-238">There is a mitigation for this issue.</span></span> <span data-ttu-id="0c25b-239">詳細については、「[Azure Storage で VNet サービス エンドポイントを使用した場合の影響](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-239">For more information, see [Impact of using VNet Service Endpoints with Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span></span> 

- <span data-ttu-id="0c25b-240">オンプレミスから Azure Storage にデータを移動するには、オンプレミスまたは ExpressRoute からのパブリック IP アドレスをホワイトリストに登録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-240">To move data from on-premises into Azure Storage, you will need to whitelist public IP addresses from your on-premises or ExpressRoute.</span></span> <span data-ttu-id="0c25b-241">詳細については、「[Azure サービスへのアクセスを仮想ネットワークに限定する](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-241">For details, see [Securing Azure services to virtual networks](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span></span>

- <span data-ttu-id="0c25b-242">Analysis Services が SQL Data Warehouse からデータを読み取ることができるようにするには、SQL Data Warehouse サービス エンドポイントを含む仮想ネットワークに Windows VM を展開します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-242">To enable Analysis Services to read data from SQL Data Warehouse, deploy a Windows VM to the virtual network that contains the SQL Data Warehouse service endpoint.</span></span> <span data-ttu-id="0c25b-243">この VM に [Azure オンプレミス データゲートウェイ](/azure/analysis-services/analysis-services-gateway)をインストールします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-243">Install [Azure On-premises Data Gateway](/azure/analysis-services/analysis-services-gateway) on this VM.</span></span> <span data-ttu-id="0c25b-244">次に Azure Analysis サービスをデータ ゲートウェイに接続します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-244">Then connect your Azure Analysis service to the data gateway.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0c25b-245">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="0c25b-245">Deploy the solution</span></span>

<span data-ttu-id="0c25b-246">この参照アーキテクチャの展開については、[GitHub][ref-arch-repo-folder] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-246">A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder].</span></span> <span data-ttu-id="0c25b-247">以下がデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-247">It deploys the following:</span></span>

  * <span data-ttu-id="0c25b-248">オンプレミスのデータベース サーバーをシミュレートする Windows VM。</span><span class="sxs-lookup"><span data-stu-id="0c25b-248">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="0c25b-249">これには、SQL Server 2017 と関連ツール、および Power BI Desktop が含まれています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-249">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
  * <span data-ttu-id="0c25b-250">SQL Server データベースからエクスポートされたデータを保持する Blob Storage を提供する Azure ストレージ アカウント。</span><span class="sxs-lookup"><span data-stu-id="0c25b-250">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
  * <span data-ttu-id="0c25b-251">Azure SQL Data Warehouse インスタンス。</span><span class="sxs-lookup"><span data-stu-id="0c25b-251">An Azure SQL Data Warehouse instance.</span></span>
  * <span data-ttu-id="0c25b-252">Azure Analysis Services インスタンス。</span><span class="sxs-lookup"><span data-stu-id="0c25b-252">An Azure Analysis Services instance.</span></span>
  * <span data-ttu-id="0c25b-253">ELT ジョブ用の Azure Data Factory と Data Factory パイプライン。</span><span class="sxs-lookup"><span data-stu-id="0c25b-253">Azure Data Factory and the Data Factory pipeline for the ELT job.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0c25b-254">前提条件</span><span class="sxs-lookup"><span data-stu-id="0c25b-254">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a><span data-ttu-id="0c25b-255">variables</span><span class="sxs-lookup"><span data-stu-id="0c25b-255">Variables</span></span>

<span data-ttu-id="0c25b-256">以下の手順には、ユーザー定義変数が含まれています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-256">The steps that follow include some user-defined variables.</span></span> <span data-ttu-id="0c25b-257">これらは定義した値に置き換える必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-257">You will need to replace these with values that you define.</span></span>

- <span data-ttu-id="0c25b-258">`<data_factory_name>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-258">`<data_factory_name>`.</span></span> <span data-ttu-id="0c25b-259">データ ファクトリ名。</span><span class="sxs-lookup"><span data-stu-id="0c25b-259">Data Factory name.</span></span>
- <span data-ttu-id="0c25b-260">`<analysis_server_name>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-260">`<analysis_server_name>`.</span></span> <span data-ttu-id="0c25b-261">Analysis Services サーバー名。</span><span class="sxs-lookup"><span data-stu-id="0c25b-261">Analysis Services server name.</span></span>
- <span data-ttu-id="0c25b-262">`<active_directory_upn>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-262">`<active_directory_upn>`.</span></span> <span data-ttu-id="0c25b-263">Azure Active Directory ユーザー プリンシパル名 (UPN)。</span><span class="sxs-lookup"><span data-stu-id="0c25b-263">Your Azure Active Directory user principal name (UPN).</span></span> <span data-ttu-id="0c25b-264">たとえば、「 `user@contoso.com` 」のように入力します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-264">For example, `user@contoso.com`.</span></span>
- <span data-ttu-id="0c25b-265">`<data_warehouse_server_name>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-265">`<data_warehouse_server_name>`.</span></span> <span data-ttu-id="0c25b-266">SQL Data Warehouse サーバー名。</span><span class="sxs-lookup"><span data-stu-id="0c25b-266">SQL Data Warehouse server name.</span></span>
- <span data-ttu-id="0c25b-267">`<data_warehouse_password>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-267">`<data_warehouse_password>`.</span></span> <span data-ttu-id="0c25b-268">SQL Data Warehouse 管理者パスワード。</span><span class="sxs-lookup"><span data-stu-id="0c25b-268">SQL Data Warehouse administrator password.</span></span>
- <span data-ttu-id="0c25b-269">`<resource_group_name>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-269">`<resource_group_name>`.</span></span> <span data-ttu-id="0c25b-270">リソース グループの名前。</span><span class="sxs-lookup"><span data-stu-id="0c25b-270">The name of the resource group.</span></span>
- <span data-ttu-id="0c25b-271">`<region>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-271">`<region>`.</span></span> <span data-ttu-id="0c25b-272">リソースが配置される Azure リージョン。</span><span class="sxs-lookup"><span data-stu-id="0c25b-272">The Azure region where the resources will be deployed.</span></span>
- <span data-ttu-id="0c25b-273">`<storage_account_name>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-273">`<storage_account_name>`.</span></span> <span data-ttu-id="0c25b-274">ストレージ アカウント名。</span><span class="sxs-lookup"><span data-stu-id="0c25b-274">Storage account name.</span></span> <span data-ttu-id="0c25b-275">ストレージ アカウントの[名前付け規則](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)に従う必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-275">Must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span>
- <span data-ttu-id="0c25b-276">`<sql-db-password>`</span><span class="sxs-lookup"><span data-stu-id="0c25b-276">`<sql-db-password>`.</span></span> <span data-ttu-id="0c25b-277">SQL Server ログイン パスワード。</span><span class="sxs-lookup"><span data-stu-id="0c25b-277">SQL Server login password.</span></span>

### <a name="deploy-azure-data-factory"></a><span data-ttu-id="0c25b-278">Azure Data Factory をデプロイする</span><span class="sxs-lookup"><span data-stu-id="0c25b-278">Deploy Azure Data Factory</span></span>

1. <span data-ttu-id="0c25b-279">[GitHub リポジトリ][ref-arch-repo]の `data\enterprise_bi_sqldw_advanced\azure\templates` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-279">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="0c25b-280">次の Azure CLI コマンドを実行して、リソース グループを作成します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-280">Run the following Azure CLI command to create a resource group.</span></span>  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    <span data-ttu-id="0c25b-281">SQL Data Warehouse、Azure Analysis Services、および Data Factory v2 をサポートするリージョンを指定します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-281">Specify a region that supports SQL Data Warehouse, Azure Analysis Services, and Data Factory v2.</span></span> <span data-ttu-id="0c25b-282">「[リージョン別の Azure 製品](https://azure.microsoft.com/global-infrastructure/services/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0c25b-282">See [Azure Products by Region](https://azure.microsoft.com/global-infrastructure/services/)</span></span>

3. <span data-ttu-id="0c25b-283">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-283">Run the following command</span></span>

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

<span data-ttu-id="0c25b-284">次に、Azure portal を使用して、次のように Azure Data Factory [統合ランタイム](/azure/data-factory/concepts-integration-runtime)の認証キーを取得します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-284">Next, use the Azure Portal to get the authentication key for the Azure Data Factory [integration runtime](/azure/data-factory/concepts-integration-runtime), as follows:</span></span>

1. <span data-ttu-id="0c25b-285">[Azure portal](https://portal.azure.com/) で Data Factory インスタンスに移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-285">In the [Azure Portal](https://portal.azure.com/), navigate to the Data Factory instance.</span></span>

2. <span data-ttu-id="0c25b-286">Data Factory ブレードで **[Author & Monitor]\(作成と監視\)** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-286">In the Data Factory blade, click **Author & Monitor**.</span></span> <span data-ttu-id="0c25b-287">Azure Data Factory ポータルが別のブラウザー ウィンドウに開きます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-287">This opens the Azure Data Factory portal in another browser window.</span></span>

    ![](./images/adf-blade.png)

3. <span data-ttu-id="0c25b-288">Azure Data Factory ポータルで鉛筆アイコン ([作成]) を選択します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-288">In the Azure Data Factory portal, select the pencil icon ("Author").</span></span> 

4. <span data-ttu-id="0c25b-289">**[接続]** をクリックし、**[統合ランタイム]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-289">Click **Connections**, and then select **Integration Runtimes**.</span></span>

5. <span data-ttu-id="0c25b-290">**sourceIntegrationRuntime** の下にある鉛筆アイコン ([編集]) をクリックします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-290">Under **sourceIntegrationRuntime**, click the pencil icon ("Edit").</span></span>

    > [!NOTE]
    > <span data-ttu-id="0c25b-291">ポータルには状態が "使用不可" と表示されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-291">The portal will show the status as "unavailable".</span></span> <span data-ttu-id="0c25b-292">この状態は、オンプレミス サーバーを展開するまで続くと想定されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-292">This is expected until you deploy the on-premises server.</span></span>

6. <span data-ttu-id="0c25b-293">**Key1** を見つけて、認証キーの値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-293">Find **Key1** and copy the value of the authentication key.</span></span>

<span data-ttu-id="0c25b-294">この認証キーは、次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-294">You will need the authentication key for the next step.</span></span>

### <a name="deploy-the-simulated-on-premises-server"></a><span data-ttu-id="0c25b-295">シミュレートされたオンプレミスのサーバーをデプロイする</span><span class="sxs-lookup"><span data-stu-id="0c25b-295">Deploy the simulated on-premises server</span></span>

<span data-ttu-id="0c25b-296">この手順では、SQL Server 2017 と関連ツールを含む、シミュレートされたオンプレミスのサーバーとして VM を展開します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-296">This step deploys a VM as a simulated on-premises server, which includes SQL Server 2017 and related tools.</span></span> <span data-ttu-id="0c25b-297">また、[Wide World Importers OLTP データベース][wwi] を SQL Server に読み込みます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-297">It also loads the [Wide World Importers OLTP database][wwi] into SQL Server.</span></span>

1. <span data-ttu-id="0c25b-298">リポジトリの `data\enterprise_bi_sqldw_advanced\onprem\templates` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-298">Navigate to the `data\enterprise_bi_sqldw_advanced\onprem\templates` folder of the repository.</span></span>

2. <span data-ttu-id="0c25b-299">`onprem.parameters.json` ファイルで `adminPassword` を検索します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-299">In the `onprem.parameters.json` file, search for `adminPassword`.</span></span> <span data-ttu-id="0c25b-300">これは、SQL Server VM にログインするためのパスワードです。</span><span class="sxs-lookup"><span data-stu-id="0c25b-300">This is the password to log into the SQL Server VM.</span></span> <span data-ttu-id="0c25b-301">この値を別のパスワードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-301">Replace the value with another password.</span></span>

3. <span data-ttu-id="0c25b-302">同じファイルで、`SqlUserCredentials` を検索します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-302">In the same file, search for `SqlUserCredentials`.</span></span> <span data-ttu-id="0c25b-303">このプロパティには、SQL Server アカウントの資格情報が指定されています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-303">This property specifies the SQL Server account credentials.</span></span> <span data-ttu-id="0c25b-304">このパスワードを別の値に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-304">Replace the password with a different value.</span></span>

4. <span data-ttu-id="0c25b-305">同じファイルで、以下のように Integration Runtime 認証キーを `IntegrationRuntimeGatewayKey` パラメーターに貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-305">In the same file, paste the Integration Runtime authentication key into the `IntegrationRuntimeGatewayKey` parameter, as shown below:</span></span>

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. <span data-ttu-id="0c25b-306">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-306">Run the following command.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

<span data-ttu-id="0c25b-307">この手順は完了するまでに 20 から 30 分かかります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-307">This step may take 20 to 30 minutes to complete.</span></span> <span data-ttu-id="0c25b-308">これには、ツールをインストールしてデータベースを復元するための [DSC](/powershell/dsc/overview) スクリプトの実行が含まれます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-308">It includes running a [DSC](/powershell/dsc/overview) script to install the tools and restore the database.</span></span> 

### <a name="deploy-azure-resources"></a><span data-ttu-id="0c25b-309">Azure リソースを展開する</span><span class="sxs-lookup"><span data-stu-id="0c25b-309">Deploy Azure resources</span></span>

<span data-ttu-id="0c25b-310">この手順では、SQL Data Warehouse、Azure Analysis Services、および Data Factory をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="0c25b-310">This step provisions SQL Data Warehouse, Azure Analysis Services, and Data Factory.</span></span>

1. <span data-ttu-id="0c25b-311">[GitHub リポジトリ][ref-arch-repo]の `data\enterprise_bi_sqldw_advanced\azure\templates` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-311">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="0c25b-312">次の Azure CLI コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-312">Run the following Azure CLI command.</span></span> <span data-ttu-id="0c25b-313">山かっこ内に示されるパラメーター値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-313">Replace the parameter values shown in angle brackets.</span></span>

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - <span data-ttu-id="0c25b-314">`storageAccountName` パラメーターは、ストレージ アカウントの[名前付け規則](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)に従う必要があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-314">The `storageAccountName` parameter must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span> 
    - <span data-ttu-id="0c25b-315">`analysisServerAdmin` パラメーターには、Azure Active Directory ユーザー プリンシパル名 (UPN) を使用します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-315">For the `analysisServerAdmin` parameter, use your Azure Active Directory user principal name (UPN).</span></span>

3. <span data-ttu-id="0c25b-316">次の Azure CLI コマンドを実行してストレージ アカウントのアクセス キーを取得します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-316">Run the following Azure CLI command to get the access key for the storage account.</span></span> <span data-ttu-id="0c25b-317">このキーは、次の手順で使用します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-317">You will use this key in the next step.</span></span>

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. <span data-ttu-id="0c25b-318">次の Azure CLI コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-318">Run the following Azure CLI command.</span></span> <span data-ttu-id="0c25b-319">山かっこ内に示されるパラメーター値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-319">Replace the parameter values shown in angle brackets.</span></span> 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    <span data-ttu-id="0c25b-320">接続文字列には、置換する必要がある山かっこ内の部分文字列があります。</span><span class="sxs-lookup"><span data-stu-id="0c25b-320">The connection strings have substrings shown in angle brackets that must be replaced.</span></span> <span data-ttu-id="0c25b-321">`<storage_account_key>` の場合、前の手順で取得したキーを使用します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-321">For `<storage_account_key>`, use the key that you got in the previous step.</span></span> <span data-ttu-id="0c25b-322">`<sql-db-password>` の場合、以前に `onprem.parameters.json` ファイルで指定した SQL Server アカウントのパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-322">For `<sql-db-password>`, use the SQL Server account password that you specified in the `onprem.parameters.json` file previously.</span></span>

### <a name="run-the-data-warehouse-scripts"></a><span data-ttu-id="0c25b-323">データ ウェアハウスのスクリプトを実行する</span><span class="sxs-lookup"><span data-stu-id="0c25b-323">Run the data warehouse scripts</span></span>

1. <span data-ttu-id="0c25b-324">[Azure portal](https://portal.azure.com/) で、`sql-vm1` という名前のオンプレミス VM を探します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-324">In the [Azure Portal](https://portal.azure.com/), find the on-premises VM, which is named `sql-vm1`.</span></span> <span data-ttu-id="0c25b-325">VM のユーザー名とパスワードは、`onprem.parameters.json` ファイルに指定されています。</span><span class="sxs-lookup"><span data-stu-id="0c25b-325">The user name and password for the VM are specified in the `onprem.parameters.json` file.</span></span>

2. <span data-ttu-id="0c25b-326">**[接続]** をクリックし、リモート デスクトップを使用して VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-326">Click **Connect** and use Remote Desktop to connect to the VM.</span></span>

3. <span data-ttu-id="0c25b-327">リモート デスクトップ セッションからコマンド プロンプトを開き、VM 上の次のフォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-327">From your Remote Desktop session, open a command prompt and navigate to the following folder on the VM:</span></span>

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. <span data-ttu-id="0c25b-328">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-328">Run the following command:</span></span>

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    <span data-ttu-id="0c25b-329">`<data_warehouse_server_name>` および `<data_warehouse_password>` の場合、前述のデータ ウェアハウス サーバーの名前とパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-329">For `<data_warehouse_server_name>` and `<data_warehouse_password>`, use the data warehouse server name and password from earlier.</span></span>

<span data-ttu-id="0c25b-330">この手順を確認するには、SQL Server Management Studio (SSMS) を使用して SQL Data Warehouse データベースに接続します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-330">To verify this step, you can use SQL Server Management Studio (SSMS) to connect to the SQL Data Warehouse database.</span></span> <span data-ttu-id="0c25b-331">データベース テーブル スキーマが表示されます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-331">You should see the database table schemas.</span></span>

### <a name="run-the-data-factory-pipeline"></a><span data-ttu-id="0c25b-332">Data Factory パイプラインを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-332">Run the Data Factory pipeline</span></span>

1. <span data-ttu-id="0c25b-333">同じリモート デスクトップ セッションから PowerShell ウィンドウを開きます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-333">From the same Remote Desktop session, open a PowerShell window.</span></span>

2. <span data-ttu-id="0c25b-334">次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-334">Run the following PowerShell command.</span></span> <span data-ttu-id="0c25b-335">確認を求めるメッセージが表示されたら、 **[はい]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-335">Choose **Yes** when prompted.</span></span>

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. <span data-ttu-id="0c25b-336">次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-336">Run the following PowerShell command.</span></span> <span data-ttu-id="0c25b-337">メッセージが表示されたら、Azure の資格情報を入力します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-337">Enter your Azure credentials when prompted.</span></span>

    ```powershell
    Connect-AzureRmAccount 
    ```

4. <span data-ttu-id="0c25b-338">次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="0c25b-338">Run the following PowerShell commands.</span></span> <span data-ttu-id="0c25b-339">山かっこ内の値を置き換えます。</span><span class="sxs-lookup"><span data-stu-id="0c25b-339">Replace the values in angle brackets.</span></span>

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    <span data-ttu-id="0c25b-340">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span><span class="sxs-lookup"><span data-stu-id="0c25b-340">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span></span>
    ```

4. Repeat these steps to create the following measures:

    ```
    <span data-ttu-id="0c25b-341">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span><span class="sxs-lookup"><span data-stu-id="0c25b-341">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span></span>
    
    <span data-ttu-id="0c25b-342">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="0c25b-342">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="0c25b-343">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="0c25b-343">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="0c25b-344">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span><span class="sxs-lookup"><span data-stu-id="0c25b-344">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span></span>
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
