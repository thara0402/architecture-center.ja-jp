---
title: Azure Monitor を使用した Azure Databricks の監視
description: Azure Log Analytics で Azure Databricks の監視を有効にする Scala ライブラリ
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 6d3d17b2e49919aea7bb08f59e19032c1c8bdafd
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503273"
---
# <a name="monitoring-azure-databricks-with-azure-monitor"></a>Azure Monitor を使用した Azure Databricks の監視

[Azure Databricks](/azure/azure-databricks/) は、ビッグ データ分析ソリューションと人工知能 (AI) ソリューションの高速な開発とデプロイを容易にする、[Apache Spark](https://spark.apache.org/) ベースの高速かつ強力な分析サービスです。 多くのユーザーが、自分の Azure Databricks ソリューションでノートブックのシンプルさを活用しています。 より堅牢なコンピューティング オプションを必要とするユーザーのために、Azure Databricks では、カスタム アプリケーション コードの分散実行がサポートされています。

監視はどの運用レベル ソリューションでも重要な要素です。そして Azure Databricks は、カスタム アプリケーション メトリック、ストリーミング クエリ イベント、アプリケーション ログ メッセージを監視するための堅牢な機能を提供します。 Azure Databricks では、この監視データをさまざまなログ記録サービスに送信できます。

以下の記事では、Azure Databricks から [Azure Monitor](/azure/azure-monitor/overview) (Azure の監視データ プラットフォーム) に監視データを送信する方法について説明します。 これらの記事は、順番どおりに完了する必要があります。

1. [メトリックを Azure Monitor に送信するよう Azure Databricks を構成する](./configure-cluster.md)
1. [Azure Databricks アプリケーション ログを Azure Monitor に送信する](./application-logs.md)
1. [ダッシュボードを使用して Azure Databricks のメトリックを視覚化する](./dashboards.md)

これらの記事に付属しているコード ライブラリを使用すると、Spark のメトリック、イベント、ログ情報を Azure Monitor に送信する Azure Databricks の基本的な監視機能が拡張されます。

これらの記事と付属のコード ライブラリの対象ユーザーは、Apache Spark と Azure Databricks のソリューション開発者です。 コードは、Java アーカイブ (JAR) ファイルに組み込まれ、Azure Databricks クラスターにデプロイされる必要があります。 コードでは [Scala](https://www.scala-lang.org/) と Java が組み合わされていて、出力 JAR ファイルをビルドするための、対応する一連の [Maven](https://maven.apache.org) プロジェクト オブジェクト モデル (POM) ファイルがあります。 Java、Scala、および Maven についての理解が前提条件として推奨されます。

## <a name="next-steps"></a>次の手順

まずはコード ライブラリを構築して、それを Azure Databricks クラスターにデプロイしましょう。

> [!div class="nextstepaction"]
> [メトリックを Azure Monitor に送信するよう Azure Databricks を構成する](./configure-cluster.md)