---
title: OLTP データ ストアの選択
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Azure での OLTP データ ストアの選択

オンライン トランザクション処理 (OLTP) は、トランザクション データとトランザクション処理の管理です。 このトピックでは、Azure での OLTP ソリューションのオプションを比較します。

> [!NOTE]
> OLTP データ ストアを使用する場合について詳しくは、[オンライン トランザクション処理](../scenarios/online-analytical-processing.md)に関するページをご覧ください。

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>OLTP データ ストアを選ぶときのオプション

Azure では、次のすべてのデータ ストアが OLTP とトランザクション データの管理のコア要件を満たします。

- [Azure SQL Database](/azure/sql-database/)
- [Azure の仮想マシン内の SQL Server](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>主要な選択条件

選択肢を絞り込むために、まず次の質問に答えてください。

- 独自のサーバーを管理するのではなく、管理されたサービスが必要ですか。

- ソリューションには、Microsoft SQL Server、MySQL、または PostgreSQL との互換性について特定の依存関係がありますか。 アプリケーションによってデータ ストアが制限される場合があり、データ ストアとの通信をサポートするドライバーや、使用されるデータベースに関する仮定に基づいて選択できます。

- 書き込みスループットの要件が特に高いですか。 答えが「はい」の場合は、インメモリ テーブルを提供するオプションを選びます。 

- ソリューションはマルチテナントですか。 その場合は、複数のデータベース インスタンスがデータベースごとに固定のリソースではなくリソースのエラスティック プールから取り込まれる容量プールをサポートするオプションを検討してください。 これにより、すべてのデータベース インスタンス間での容量の分配が向上し、ソリューションのコスト効果が向上する可能性があります。

- 複数のリージョンで待機時間の短いデータの読み取りが可能である必要がありますか。 答えが「はい」の場合は、読み取り可能なセカンダリ レプリカをサポートするオプションを選びます。

- データベースは地理的なリージョン全体で可用性が高い必要がありますか。 答えが「はい」の場合は、geography 型のレプリケーションをサポートするオプションを選びます。 また、プライマリ レプリカからセカンダリ レプリカへの自動フェールオーバーをサポートするオプションも検討します。

- データベースには特定のセキュリティ ニーズがありますか。 答えが「はい」の場合は、行レベル セキュリティ、データ マスキング、透過的なデータ暗号化などの機能を提供するオプションを確認します。

## <a name="capability-matrix"></a>機能のマトリックス

次の表は、機能の主な相違点をまとめたものです。

### <a name="general-capabilities"></a>一般的な機能 
| | Azure SQL Database | Azure の仮想マシン内の SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| マネージド サービス | [はい] | いいえ  | 可能  | [はい] |
| 実行するプラットフォーム | 該当なし | Windows、Linux、Docker | 該当なし | 該当なし |
| プログラミング <sup>1</sup> | T-SQL、.NET、R | T-SQL、.NET、R、Python | T-SQL、.NET、R、Python | SQL | SQL |

[1] 多くのプログラミング言語が OLTP データ ストアに接続して使用できるクライアント ドライバー サポートは含まれません。

### <a name="scalability-capabilities"></a>スケーラビリティ機能
| | Azure SQL Database | Azure の仮想マシン内の SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| データベース インスタンスの最大サイズ | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| 容量プールをサポート  | [はい] | [はい] | いいえ  | いいえ  |
| クラスターのスケールアウトをサポート  | いいえ  | [はい] | いいえ  | いいえ  |
| 動的スケーラビリティ (スケールアップ)  | [はい] | いいえ  | 可能  | [はい] |

### <a name="analytic-workload-capabilities"></a>分析ワークロード機能
| | Azure SQL Database | Azure の仮想マシン内の SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| テンポラル テーブル | [はい] | [はい] | いいえ  | いいえ  |
| インメモリ (メモリ最適化) テーブル | [はい] | [はい] | いいえ  | いいえ  |
| 列ストアをサポート | [はい] | [はい] | いいえ  | いいえ  |
| クエリの適応処理 | [はい] | [はい] | いいえ  | いいえ  |

### <a name="availability-capabilities"></a>可用性に関する機能
| | Azure SQL Database | Azure の仮想マシン内の SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 読み取り可能なセカンダリ | [はい] | [はい] | いいえ  | いいえ  | 
| geography 型のレプリケーション | [はい] | [はい] | いいえ  | いいえ  | 
| セカンダリへの自動フェールオーバー | [はい] | いいえ  | いいえ  | いいえ |
| ポイントインタイム リストア | [はい] | はい | はい | [はい] |

### <a name="security-capabilities"></a>セキュリティ機能
| | Azure SQL Database | Azure の仮想マシン内の SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 行レベルのセキュリティ | [はい] | はい | はい | [はい] |
| データ マスク | [はい] | [はい] | いいえ  | いいえ  |
| 透過的なデータ暗号化 | [はい] | はい | はい | [はい] |
| 特定の IP アドレスへのアクセスを制限 | [はい] | はい | はい | [はい] |
| VNET アクセスのみを許可するようにアクセスを制限 | [はい] | [はい] | いいえ  | いいえ  |
| Azure Active Directory 認証 | [はい] | [はい] | いいえ  | いいえ  |
| Active Directory 認証 | いいえ  | [はい] | いいえ  | いいえ  |
| 多要素認証 | [はい] | [はい] | いいえ  | いいえ  |
| [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) をサポート | [はい] | はい | [はい] | いいえ  | いいえ  |
| プライベート IP | いいえ  | 可能  | [はい] | いいえ  | いいえ  |

