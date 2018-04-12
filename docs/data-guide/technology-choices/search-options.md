---
title: 検索データ ストアの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ead07e307e96696faa5ddf48505eee378027523c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>Azure での検索データ ストアの選択

この記事では、Azure での検索データ ストア向けのテクノロジーの選択肢を比較します。 検索データ ストアは、自由形式のテキストで検索を実行するための特殊なインデックスを作成して格納するために使用されます。 インデックスが設定されたテキストは、BLOB ストレージなどの独立したデータ ストアに配置できます。 アプリケーションは、検索データ ストアにクエリを送信します。その結果は一致するドキュメントの一覧です。 このシナリオの詳細については、[自由形式のテキストでの検索の処理](../scenarios/search.md)に関する記事を参照してください。 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>検索データ ストアを選択するときのオプション
Azure では、次のデータ ストアのすべてが、検索インデックスを提供することによって自由形式のテキスト データに対する検索のコア要件を満たしています。
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [Solr を使用する HDInsight](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [フルテキスト検索を行う Azure SQL Database](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>主要な選択条件

検索シナリオでは、次の質問に答えることによって、ニーズに適した検索データ ストアを選択することから始めます。

- 独自のサーバーを管理するのではなく、管理されたサービスが必要ですか。

- 設計時に、インデックス スキーマを指定できますか。 指定できない場合は、更新可能スキーマをサポートするオプションを選択してください。

- インデックスが必要なのはフルテキスト検索のみですか。数値データの高速集計とその他の分析も必要ですか。 フルテキスト検索以上の機能が必要な場合は、追加分析をサポートするオプションを検討してください。

- ログの収集、集計、およびインデックス付きのデータのビジュアル化がサポートされるログ分析用の検索インデックスが必要ですか。 該当する場合は、ログ分析スタックの一部である Elasticsearch を検討してください。

- PDF、Word、PowerPoint、Excel などの一般的なドキュメント形式のデータにインデックスを作成する必要がありますか。 「はい」の場合は、ドキュメント インデクサーを提供するオプションを選択してください。

- データベースに特殊なセキュリティ ニーズがありますか。 「はい」の場合は、次のセキュリティ機能を検討してください。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能

| | Azure Search | Elasticsearch | Solr を使用する HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| マネージド サービスか | [はい] | いいえ  | 可能  | [はい] |  
| REST API | [はい] | はい | [はい] | いいえ  |
| プログラミング | .NET | Java | Java | T-SQL | 
| 一般的なファイルの種類 (PDF、DOCX、TXT、およびなど) 向けのドキュメント インデクサー | [はい] | いいえ  | [はい] | いいえ  |

### <a name="manageability-capabilities"></a>管理容易性機能

| | Azure Search | Elasticsearch | Solr を使用する HDInsight | SQL Database | 
| --- | --- | --- | --- | --- |
| 更新可能なスキーマ | いいえ  | 可能  | はい | [はい] |
| スケール アウトのサポート  | [はい] | はい | [はい] | いいえ  |

### <a name="analytic-workload-capabilities"></a>分析ワークロード機能

| | Azure Search | Elasticsearch | Solr を使用する HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| フルテキスト検索を上回る分析のサポート | いいえ  | 可能  | はい | [はい] |
| ログ分析スタックの一部 | いいえ  | はい (ELK) |  いいえ  | いいえ  |
| セマンティック検索のサポート | はい (類似のドキュメントのみ検索) | [はい] | はい | [はい] | 

### <a name="security-capabilities"></a>セキュリティ機能

| | Azure Search | Elasticsearch | Solr を使用する HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| 行レベルのセキュリティ | 一部 (グループ ID でフィルター処理するアプリケーション クエリが必要) | 一部 (グループ ID でフィルター処理するアプリケーション クエリが必要) | [はい] | [はい] | 
| 透過的なデータ暗号化 | いいえ  | いいえ  | いいえ  | [はい] |  
| 特定の IP アドレスへのアクセスを制限 | いいえ  | 可能  | はい | [はい] |   
| 仮想ネットワーク アクセスのみを許可するようにアクセスを制限 | いいえ  | 可能  | はい | [はい] |  
| Active Directory 認証 (統合認証) | いいえ  | いいえ  | いいえ  | [はい] | 

## <a name="see-also"></a>関連項目

[検索用の自由形式テキストの処理](../scenarios/search.md)