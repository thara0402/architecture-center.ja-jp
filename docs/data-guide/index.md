---
title: "Azure データ アーキテクチャ ガイド"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Azure データ アーキテクチャ ガイド

このガイドでは、Microsoft Azure でデータ中心のソリューションを設計するための体系的なアプローチについて説明します。 このアプローチは、顧客エンゲージメントから派生した実証済みのプラクティスに基づいています。

## <a name="introduction"></a>はじめに

クラウドによって、データの処理方法や格納方法など、アプリケーションの設計方法は変化しています。 _多言語永続化_ソリューションは、ソリューションのすべてのデータを処理する単一の汎用データベースではなく、特定の機能を提供するために個々に最適化された、複数の専用データ ストアを使用します。 その結果、ソリューション内のデータの観点が変わります。 単一のデータ レイヤーの読み取りと書き込むを行う複数レイヤーのビジネス ロジックはなくなりました。 代わりに、*データ パイプライン*を中心にして、データがソリューションを経由してどのように流れるか、どこで処理されるか、どこに格納されるか、パイプライン内の次のコンポーネントによってどのように使用されるかを記述するソリューションが設計されています。 

## <a name="how-this-guide-is-structured"></a>本書の構成

このガイドは、*リレーショナル* データと*非リレーショナル* データの区別という基本的な点を中心に構成されています。 

![](./images/guide-steps.svg)

通常、リレーショナル データは従来の RDBMS またはデータ ウェアハウスに格納されています。 参照整合性を維持するための一連の制約を持つ事前定義スキーマ ("書き込み時のスキーマ") があります。 ほとんどのリレーショナル データベースは、クエリに構造化照会言語 (SQL) を使用しています。 リレーショナル データベースを使用するソリューションには、オンライン トランザクション処理 (OLTP) とオンライン分析処理 (OLAP) があります。

非リレーショナル データは、従来の RDBMS システムで使用されている[リレーショナル モデル](https://en.wikipedia.org/wiki/Relational_model)を使用しないデータです。 これには、キー/値データ、JSON データ、グラフ データ、時系列データなどのデータ型が含まれます。 *NoSQL* という用語は、さまざまな種類の非リレーショナル データを格納するように設計されたデータベースを指します。 ただし、多くの非リレーショナル データ ストアは SQL 互換のクエリをサポートしているため、この用語は完全に正確とは言えません。 非リレーショナル データと NoSQL データベースは、*ビッグ データ* ソリューションのディスカッションでよく出てくる用語です。 ビッグ データ アーキテクチャは、従来のデータベース システムには多すぎる、または複雑すぎるデータのインジェスト、処理、分析を扱うために設計されています。 

データ アーキテクチャ ガイドには、これら 2 つの主なカテゴリを説明する以下のセクションがあります。

- **概念。** この種類のデータを操作する場合に理解する必要がある主な概念を紹介する概要の記事。
- **シナリオ。** 関連する Azure サービスの説明とシナリオに適したアーキテクチャを含む、代表的な一連のデータ シナリオ。
- **テクノロジの選択**。 オープン ソースのオプションを含む、Azure で利用できるさまざまなデータ テクノロジの詳細な比較。 各カテゴリ内で、シナリオに適したテクノロジの選択に役立つ主な選択基準と機能のマトリックスについて説明します。

このガイドの目的は、データ サイエンスやデータベース理論を教えることではありません。このようなテーマについては関連する書籍を参照してください。 このガイドの目標は、シナリオに適したデータ アーキテクチャまたはデータ パイプラインを選択し、要件に最適な Azure サービスとテクノロジを選択できるようにすることです。 既に念頭に置いているアーキテクチャがある場合は、そのままテクノロジのオプションに進んでください。

## <a name="traditional-rdbms"></a>従来の RDBMS

### <a name="concepts"></a>概念

- [リレーショナル データ](./concepts/relational-data.md) 
- [トランザクション データ](./concepts/transactional-data.md) 
- [セマンティック モデリング](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>シナリオ

- [オンライン分析処理 (OLAP)](./scenarios/online-analytical-processing.md)
- [オンライン トランザクション処理 (OLTP)](./scenarios/online-transaction-processing.md) 
- [データ ウェアハウスとデータ マート](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>ビッグ データと NoSQL

### <a name="concepts"></a>概念

- [非リレーショナル データ ストア](./concepts/non-relational-data.md)
- [CSV ファイルと JSON ファイルの操作](./concepts/csv-and-json.md)
- [ビッグ データ アーキテクチャ](./concepts/big-data.md)
- [高度な分析](./concepts/advanced-analytics.md) 
- [大規模な機械学習](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>シナリオ

- [バッチ処理](./scenarios/batch-processing.md)
- [リアルタイム処理](./scenarios/real-time-processing.md)
- [自由形式のテキスト検索](./scenarios/search.md)
- [対話型データ探索](./scenarios/interactive-data-exploration.md)
- [自然言語処理](./scenarios/natural-language-processing.md)
- [時系列ソリューション](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>横断的関心事

- [データ転送](./scenarios/data-transfer.md) 
- [オンプレミス データ ソリューションのクラウドへの拡張](./scenarios/hybrid-on-premises-and-cloud.md) 
- [データ保護ソリューション](./scenarios/securing-data-solutions.md) 
