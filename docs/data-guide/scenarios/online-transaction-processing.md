---
title: "オンライン トランザクション処理 (OLTP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a><span data-ttu-id="b9fd3-102">オンライン トランザクション処理 (OLTP)</span><span class="sxs-lookup"><span data-stu-id="b9fd3-102">Online transaction processing (OLTP)</span></span>

<span data-ttu-id="b9fd3-103">コンピューター システムを使用した[トランザクション データ](../concepts/transactional-data.md)の管理は、オンライン トランザクション処理 (OLTP) と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-103">The management of [transactional data](../concepts/transactional-data.md) using computer systems is referred to as Online Transaction Processing (OLTP).</span></span> <span data-ttu-id="b9fd3-104">OLTP システムは、組織の日常的な業務でビジネス インタラクションが発生したときに記録し、そのデータに対してクエリを実行して推論する処理をサポートします。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-104">OLTP systems record business interactions as they occur in the day-to-day operation of the organization, and support querying of this data to make inferences.</span></span>

![Azure の OLTP](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a><span data-ttu-id="b9fd3-106">このソリューションを使用する状況</span><span class="sxs-lookup"><span data-stu-id="b9fd3-106">When to use this solution</span></span>

<span data-ttu-id="b9fd3-107">ビジネス トランザクションを効率的に処理して保存し、一貫した方法でクライアント アプリケーションですぐに利用できるようにする必要がある場合は、OLTP を選択します。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-107">Choose OLTP when you need to efficiently process and store business transactions and immediately make them available to client applications in a consistent way.</span></span> <span data-ttu-id="b9fd3-108">処理が明らかに遅れた場合に日常業務に悪影響が及ぶ場合は、このアーキテクチャを使用します。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-108">Use this architecture when any tangible delay in processing would have a negative impact on the day-to-day operations of the business.</span></span>

<span data-ttu-id="b9fd3-109">OLTP システムは、トランザクションを効率的に処理および格納し、トランザクション データに対してクエリを実行できるように設計されています。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-109">OLTP systems are designed to efficiently process and store transactions, as well as query transactional data.</span></span> <span data-ttu-id="b9fd3-110">OLTP システムで個々のトランザクションを効率的に処理して格納するという目標は、データの正規化 (つまり、データを冗長性があまりない小さなチャンクに分割する処理) によって部分的に達成されます。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-110">The goal of efficiently processing and storing individual transactions by an OLTP system is partly accomplished by data normalization &mdash; that is, breaking the data up into smaller chunks that are less redundant.</span></span> <span data-ttu-id="b9fd3-111">OLTP システムを使用して大量のトランザクションを独立して処理できるようになり、冗長データが存在する場合は、データ整合性を維持するために必要な余計な処理が回避されるため、効率的でもあります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-111">This supports efficiency because it enables the OLTP system to process large numbers of transactions independently, and avoids extra processing needed to maintain data integrity in the presence of redundant data.</span></span>

## <a name="challenges"></a><span data-ttu-id="b9fd3-112">課題</span><span class="sxs-lookup"><span data-stu-id="b9fd3-112">Challenges</span></span>
<span data-ttu-id="b9fd3-113">OLTP システムを実装して使用すると、いくつかの課題が生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-113">Implementing and using an OLTP system can create a few challenges:</span></span>

- <span data-ttu-id="b9fd3-114">OLTP システムは、入念に計画された SQL Server ベースのソリューションなどの例外はありますが、大量のデータに対する集計の処理には必ずしも適していません。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-114">OLTP systems are not always good for handling aggregates over large amounts of data, although there are exceptions, such as a well-planned SQL Server-based solution.</span></span> <span data-ttu-id="b9fd3-115">データに対する分析は、数百万単位の個々のトランザクションを集計する計算に依存しており、OLTP システムにとっては非常にリソースを消費する処理です。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-115">Analytics against the data, that rely on aggregate calculations over millions of individual transactions, are very resource intensive for an OLTP system.</span></span> <span data-ttu-id="b9fd3-116">実行に時間がかかり、データベース内の他のトランザクションをブロックして速度の低下が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-116">They can be slow to execute and can cause a slow-down by blocking other transactions in the database.</span></span>
- <span data-ttu-id="b9fd3-117">高度に正規化されたデータの分析とレポート作成を行う場合、クエリは複雑になる傾向があります。なぜなら、ほとんどのクエリでは、結合を使用してデータを正規化する必要があるためです。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-117">When conducting analytics and reporting on data that is highly normalized, the queries tend to be complex, because most queries need to de-normalize the data by using joins.</span></span> <span data-ttu-id="b9fd3-118">また、OLTP システムのデータベース オブジェクトの名前付け規則は簡潔な傾向があります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-118">Also, naming conventions for database objects in OLTP systems tend to be terse and succinct.</span></span> <span data-ttu-id="b9fd3-119">正規化の増加と簡潔な名前付け規則の組み合わせにより、ビジネス ユーザーが DBA やデータの開発者から支援を受けずに OLTP システムでクエリを実行することは困難になっています。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-119">The increased normalization coupled with terse naming conventions makes OLTP systems difficult for business users to query, without the help of a DBA or data developer.</span></span>
- <span data-ttu-id="b9fd3-120">トランザクションの履歴を無期限に格納し、1 つのテーブルに格納されるデータ量が多くなりすぎると、格納されるトランザクション数に応じてクエリのパフォーマンスが低下する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-120">Storing the history of transactions indefinitely and storing too much data in any one table can lead to slow query performance, depending on the number of transactions stored.</span></span> <span data-ttu-id="b9fd3-121">一般的な解決策は、OLTP システムで関連する時間枠 (現在の会計年度など) を維持し、履歴データをデータ マートや[データ ウェアハウス](../technology-choices/data-warehouses.md)などの他のシステムにオフロードすることです。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-121">The common solution is to maintain a relevant window of time (such as the current fiscal year) in the OLTP system and offload historical data to other systems, such as a data mart or [data warehouse](../technology-choices/data-warehouses.md).</span></span>

## <a name="oltp-in-azure"></a><span data-ttu-id="b9fd3-122">Azure の OLTP</span><span class="sxs-lookup"><span data-stu-id="b9fd3-122">OLTP in Azure</span></span>

<span data-ttu-id="b9fd3-123">[App Service Web Apps](/azure/app-service/app-service-web-overview) でホストされている Web サイト、App Service で実行されている REST API、モバイル アプリケーションやデスクトップ アプリケーションなどのアプリケーションは、通常、REST API の中継ぎを介して OLTP システムと通信します。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-123">Applications such as websites hosted in [App Service Web Apps](/azure/app-service/app-service-web-overview), REST APIs running in App Service, or mobile or desktop applications communicate with the OLTP system, typically via a REST API intermediary.</span></span>

<span data-ttu-id="b9fd3-124">実際には、ほとんどのワークロードは純粋には OLTP ではありません。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-124">In practice, most workloads are not purely OLTP.</span></span> <span data-ttu-id="b9fd3-125">[分析コンポーネント](../scenarios/online-analytical-processing.md)に近い傾向もあります。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-125">There tends to be an [analytical component](../scenarios/online-analytical-processing.md) as well.</span></span> <span data-ttu-id="b9fd3-126">さらに、運用システムに対してレポートを実行するなど、リアルタイム レポートの需要が高まっています。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-126">In addition, there is an increasing demand for real-time reporting, such as running reports against the operational system.</span></span> <span data-ttu-id="b9fd3-127">これは、HTAP (ハイブリッド取引と分析処理) とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-127">This is also referred to as HTAP (Hybrid Transactional and Analytical Processing).</span></span> <span data-ttu-id="b9fd3-128">詳細については、[オンライン分析処理 (OLAP) データ ストア](../technology-choices/olap-data-stores.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-128">For more information, see [Online Analytical Processing (OLAP) data stores](../technology-choices/olap-data-stores.md).</span></span>

## <a name="technology-choices"></a><span data-ttu-id="b9fd3-129">テクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="b9fd3-129">Technology choices</span></span>

<span data-ttu-id="b9fd3-130">データ ストレージ:</span><span class="sxs-lookup"><span data-stu-id="b9fd3-130">Data storage:</span></span>

- [<span data-ttu-id="b9fd3-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="b9fd3-131">Azure SQL Database</span></span>](/azure/sql-database/)
- [<span data-ttu-id="b9fd3-132">Azure VM の SQL Server</span><span class="sxs-lookup"><span data-stu-id="b9fd3-132">SQL Server in an Azure VM</span></span>](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [<span data-ttu-id="b9fd3-133">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="b9fd3-133">Azure Database for MySQL</span></span>](/azure/mysql/)
- [<span data-ttu-id="b9fd3-134">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="b9fd3-134">Azure Database for PostgreSQL</span></span>](/azure/postgresql/)

<span data-ttu-id="b9fd3-135">詳細については、[OLTP データ ストアの選択](../technology-choices/oltp-data-stores.md)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="b9fd3-135">For more information, see [Choosing an OLTP data store](../technology-choices/oltp-data-stores.md)</span></span>

<span data-ttu-id="b9fd3-136">データ ソース:</span><span class="sxs-lookup"><span data-stu-id="b9fd3-136">Data sources:</span></span>

- [<span data-ttu-id="b9fd3-137">App Service</span><span class="sxs-lookup"><span data-stu-id="b9fd3-137">App service</span></span>](/azure/app-service/)
- [<span data-ttu-id="b9fd3-138">Mobile Apps</span><span class="sxs-lookup"><span data-stu-id="b9fd3-138">Mobile Apps</span></span>](/azure/app-service-mobile/)

