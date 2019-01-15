---
title: 検索用の自由形式テキストの処理
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 6b90d70129f6b5ba978ce0f79dc5a04d57126b5d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113027"
---
# <a name="processing-free-form-text-for-search"></a><span data-ttu-id="b7b02-102">検索用の自由形式テキストの処理</span><span class="sxs-lookup"><span data-stu-id="b7b02-102">Processing free-form text for search</span></span>

<span data-ttu-id="b7b02-103">検索をサポートするため、テキストの段落を含むドキュメントに対して自由形式のテキスト処理を実行されます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-103">To support search, free-form text processing can be performed against documents containing paragraphs of text.</span></span>

<span data-ttu-id="b7b02-104">テキスト検索は、ドキュメントのコレクションに対して事前に計算済みの特殊なインデックスを構築することによって機能します。</span><span class="sxs-lookup"><span data-stu-id="b7b02-104">Text search works by constructing a specialized index that is precomputed against a collection of documents.</span></span> <span data-ttu-id="b7b02-105">クライアント アプリケーションは、検索用語を含むクエリを送信します。</span><span class="sxs-lookup"><span data-stu-id="b7b02-105">A client application submits a query that contains the search terms.</span></span> <span data-ttu-id="b7b02-106">クエリでは、ドキュメントの一覧で構成される結果セットが返ります。一覧は、各ドキュメントが検索条件と一致する度合いに従って並べられます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-106">The query returns a result set, consisting of a list of documents sorted by how well each document matches the search criteria.</span></span> <span data-ttu-id="b7b02-107">結果セットには、ドキュメントが条件と一致するコンテキストも含まれる可能性があり、それによってアプリケーションは、ドキュメント内の一致する語句を強調表示できます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-107">The result set may also include the context in which the document matches the criteria, which enables the application to highlight the matching phrase in the document.</span></span>

![検索パイプラインの図](./images/search-pipeline.png)

<span data-ttu-id="b7b02-109">自由形式テキストの処理では、大量のテキスト データから有用で実用的なデータを生成できます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-109">Free-form text processing can produce useful, actionable data from large amounts of noisy text data.</span></span> <span data-ttu-id="b7b02-110">結果は、構造化されていないドキュメントに対して、適切に定義され、クエリ可能な構造を与えることができます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-110">The results can give unstructured documents a well-defined and queryable structure.</span></span>

## <a name="challenges"></a><span data-ttu-id="b7b02-111">課題</span><span class="sxs-lookup"><span data-stu-id="b7b02-111">Challenges</span></span>

- <span data-ttu-id="b7b02-112">自由形式テキストのドキュメントのコレクションを処理する操作は、通常は、時間がかかるだけでなく、コンピューティングにおけるリソース負荷も高くなります。</span><span class="sxs-lookup"><span data-stu-id="b7b02-112">Processing a collection of free-form text documents is typically computationally intensive, as well as time intensive.</span></span>
- <span data-ttu-id="b7b02-113">自由形式テキストを効率的に検索するために、検索インデックスでは、構造が似ている用語に基づくあいまい検索をサポートする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b7b02-113">In order to search free-form text effectively, the search index should support fuzzy search based on terms that have a similar construction.</span></span> <span data-ttu-id="b7b02-114">たとえば、見出し語解析と語幹抽出による検索インデックスを構築して、"run" のクエリを実行したときに、"ran" と "running" を含むドキュメントも検索されるようにします。</span><span class="sxs-lookup"><span data-stu-id="b7b02-114">For example, search indexes are built with lemmatization and linguistic stemming, so that queries for "run" will match documents that contain "ran" and "running."</span></span>

## <a name="architecture"></a><span data-ttu-id="b7b02-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="b7b02-115">Architecture</span></span>

<span data-ttu-id="b7b02-116">ほとんどのシナリオでは、ソース テキスト ドキュメントは Azure Storage または Azure Data Lake Store などのオブジェクト ストレージに読み込まれます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-116">In most scenarios, the source text documents are loaded into object storage such as Azure Storage or Azure Data Lake Store.</span></span> <span data-ttu-id="b7b02-117">例外は、SQL Server または Azure SQL データベース内でのフルテキスト検索の使用です。</span><span class="sxs-lookup"><span data-stu-id="b7b02-117">An exception is using full text search within SQL Server or Azure SQL Database.</span></span> <span data-ttu-id="b7b02-118">この場合、ドキュメントのデータは、データベースによって管理されるテーブルに読み込まれます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-118">In this case, the document data is loaded into tables managed by the database.</span></span> <span data-ttu-id="b7b02-119">格納されると、ドキュメントがバッチ処理されてインデックスが作成されます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-119">Once stored, the documents are processed in a batch to create the index.</span></span>

## <a name="technology-choices"></a><span data-ttu-id="b7b02-120">テクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="b7b02-120">Technology choices</span></span>

<span data-ttu-id="b7b02-121">検索インデックスを作成するためのオプションには、Azure Search、Elasticsearch、および Solr を使用する HDInsight が含まれます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-121">Options for creating a search index include Azure Search, Elasticsearch, and HDInsight with Solr.</span></span> <span data-ttu-id="b7b02-122">これらの各テクノロジは、ドキュメントのコレクションから検索インデックスを設定できます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-122">Each of these technologies can populate a search index from a collection of documents.</span></span> <span data-ttu-id="b7b02-123">Azure Search には、プレーン テキストから Excel 形式と PDF 形式に至るまでのドキュメントのインデックスを自動的に設定できるインデクサーが用意されています。</span><span class="sxs-lookup"><span data-stu-id="b7b02-123">Azure Search provides indexers that can automatically populate the index for documents ranging from plain text to Excel and PDF formats.</span></span> <span data-ttu-id="b7b02-124">HDInsight では、Apache Solr で、プレーン テキスト、Word、PDF を含むさまざまな種類のバイナリ ファイルにインデックスを設定できます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-124">On HDInsight, Apache Solr can index binary files of many types, including plain text, Word, and PDF.</span></span> <span data-ttu-id="b7b02-125">インデックスが構築されたら、クライアントは、REST API を使用して検索インターフェイスにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-125">Once the index is constructed, clients can access the search interface by means of a REST API.</span></span>

<span data-ttu-id="b7b02-126">テキスト データが SQL Server または Azure SQL Database に格納されている場合は、データベースに組み込まれているフルテキスト検索を使用できます。</span><span class="sxs-lookup"><span data-stu-id="b7b02-126">If your text data is stored in SQL Server or Azure SQL Database, you can use the full-text search that is built into the database.</span></span> <span data-ttu-id="b7b02-127">データベースは、同じデータベース内に格納されたテキスト、バイナリ、または XML データからインデックスを設定します。</span><span class="sxs-lookup"><span data-stu-id="b7b02-127">The database populates the index from text, binary, or XML data stored within the same database.</span></span> <span data-ttu-id="b7b02-128">クライアントは、T-SQL クエリを使用して検索します。</span><span class="sxs-lookup"><span data-stu-id="b7b02-128">Clients search by using T-SQL queries.</span></span>

<span data-ttu-id="b7b02-129">詳細については、[検索データ ストア](../technology-choices/search-options.md)に関する記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b7b02-129">For more information, see [Search data stores](../technology-choices/search-options.md).</span></span>
