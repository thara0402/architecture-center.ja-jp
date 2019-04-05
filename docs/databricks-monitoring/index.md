---
title: Azure Monitor を使用した Azure Databricks の監視
description: Azure Log Analytics で Azure Databricks の監視を有効にする Scala ライブラリ
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 4544d3abc3264ec459a80ac1a61a912e6d30d6b2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887694"
---
# <a name="monitoring-azure-databricks"></a><span data-ttu-id="d2eac-103">Azure Databricks の監視</span><span class="sxs-lookup"><span data-stu-id="d2eac-103">Monitoring Azure Databricks</span></span>

<span data-ttu-id="d2eac-104">[Azure Databricks](/azure/azure-databricks/) は、ビッグ データ分析ソリューションと人工知能 (AI) ソリューションの高速な開発とデプロイを容易にする、[Apache Spark](https://spark.apache.org/) ベースの高速かつ強力な分析サービスです。</span><span class="sxs-lookup"><span data-stu-id="d2eac-104">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="d2eac-105">多くのユーザーが、自分の Azure Databricks ソリューションでノートブックのシンプルさを活用しています。</span><span class="sxs-lookup"><span data-stu-id="d2eac-105">Many users take advantage of the simplicity of notebooks in their Azure Databricks solutions.</span></span> <span data-ttu-id="d2eac-106">より堅牢なコンピューティング オプションを必要とするユーザーのために、Azure Databricks では、カスタム アプリケーション コードの分散実行がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="d2eac-106">For users that require more robust computing options, Azure Databricks supports the distributed execution of custom application code.</span></span>

<span data-ttu-id="d2eac-107">監視はどの運用レベル ソリューションでも重要な要素です。そして Azure Databricks は、カスタム アプリケーション メトリック、ストリーミング クエリ イベント、アプリケーション ログ メッセージを監視するための堅牢な機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="d2eac-107">Monitoring is a critical part of any production-level solution, and Azure Databricks offers robust functionality for monitoring custom application metrics, streaming query events, and application log messages.</span></span> <span data-ttu-id="d2eac-108">Azure Databricks では、この監視データをさまざまなログ記録サービスに送信できます。</span><span class="sxs-lookup"><span data-stu-id="d2eac-108">Azure Databricks can send this monitoring data to different logging services.</span></span>

<span data-ttu-id="d2eac-109">以下の記事では、Azure Databricks から [Azure Monitor](/azure/azure-monitor/overview) (Azure の監視データ プラットフォーム) に監視データを送信する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="d2eac-109">The following articles show how to send monitoring data from Azure Databricks to [Azure Monitor](/azure/azure-monitor/overview), the monitoring data platform for Azure.</span></span> <span data-ttu-id="d2eac-110">これらの記事は、順番どおりに完了する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d2eac-110">You should follow these articles in order.</span></span>

1. [<span data-ttu-id="d2eac-111">メトリックを Azure Monitor に送信するよう Azure Databricks を構成する</span><span class="sxs-lookup"><span data-stu-id="d2eac-111">Configure Azure Databricks to send metrics to Azure Monitor</span></span>](./configure-cluster.md)
1. [<span data-ttu-id="d2eac-112">Azure Databricks アプリケーション ログを Azure Monitor に送信する</span><span class="sxs-lookup"><span data-stu-id="d2eac-112">Send Azure Databricks application logs to Azure Monitor</span></span>](./application-logs.md)
1. [<span data-ttu-id="d2eac-113">ダッシュボードを使用して Azure Databricks のメトリックを視覚化する</span><span class="sxs-lookup"><span data-stu-id="d2eac-113">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<span data-ttu-id="d2eac-114">これらの記事に付属しているコード ライブラリを使用すると、Spark のメトリック、イベント、ログ情報を Azure Monitor に送信する Azure Databricks の基本的な監視機能が拡張されます。</span><span class="sxs-lookup"><span data-stu-id="d2eac-114">The code library that accompanies these articles extends the core monitoring functionality of Azure Databricks to send Spark metrics, events, and logging information to Azure Monitor.</span></span>

<span data-ttu-id="d2eac-115">これらの記事と付属のコード ライブラリの対象ユーザーは、Apache Spark と Azure Databricks のソリューション開発者です。</span><span class="sxs-lookup"><span data-stu-id="d2eac-115">The audience for these articles and the accompanying code library are Apache Spark and Azure Databricks solution developers.</span></span> <span data-ttu-id="d2eac-116">コードは、Java アーカイブ (JAR) ファイルに組み込まれ、Azure Databricks クラスターにデプロイされる必要があります。</span><span class="sxs-lookup"><span data-stu-id="d2eac-116">The code must be built into Java Archive (JAR) files and then deployed to an Azure Databricks cluster.</span></span> <span data-ttu-id="d2eac-117">コードでは [Scala](https://www.scala-lang.org/) と Java が組み合わされていて、出力 JAR ファイルをビルドするための、対応する一連の [Maven](https://maven.apache.org) プロジェクト オブジェクト モデル (POM) ファイルがあります。</span><span class="sxs-lookup"><span data-stu-id="d2eac-117">The code is a combination of [Scala](https://www.scala-lang.org/) and Java, with a corresponding set of [Maven](https://maven.apache.org) project object model (POM) files to build the output JAR files.</span></span> <span data-ttu-id="d2eac-118">Java、Scala、および Maven についての理解が前提条件として推奨されます。</span><span class="sxs-lookup"><span data-stu-id="d2eac-118">Understanding of Java, Scala, and Maven are recommended as prerequisistes.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d2eac-119">次の手順</span><span class="sxs-lookup"><span data-stu-id="d2eac-119">Next steps</span></span>

<span data-ttu-id="d2eac-120">まずはコード ライブラリを構築して、それを Azure Databricks クラスターにデプロイしましょう。</span><span class="sxs-lookup"><span data-stu-id="d2eac-120">Start by building the code library and deploying it to your Azure Databricks cluster.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d2eac-121">メトリックを Azure Monitor に送信するよう Azure Databricks を構成する</span><span class="sxs-lookup"><span data-stu-id="d2eac-121">Configure Azure Databricks to send metrics to Azure Monitor</span></span>](./configure-cluster.md)