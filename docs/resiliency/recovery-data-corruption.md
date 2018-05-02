---
title: データの破損または偶発的な削除から復旧する
description: データの破損または偶発的なデータの削除から復旧する方法、回復力と高可用性を備えたフォールト トレラント アプリケーションを設計する方法、障害復旧を計画する方法に関する記事
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: b0716de39fe69d607b9a63e51356d28bbcdbfeae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>データの破損または偶発的な削除から復旧する 

堅牢なビジネス継続性計画の一環として、データが破損した場合や誤って削除された場合に備えた計画を作成します。 以下に、アプリケーション エラーによってデータが破損したか、オペレーターのエラーによってデータが誤って削除された後の復旧に関する情報を示します。

## <a name="virtual-machines"></a>Virtual Machines

Azure Virtual Machines (VM) をアプリケーション エラーや誤削除から保護するには、[Azure Backup](/azure/backup/) を使用します。 Azure Backup を使用すると、複数の VM ディスク間で一貫性のあるバックアップを作成できます。 さらに、バックアップ コンテナーをリージョン間でレプリケートし、リージョン損失からの復旧に備えることができます。

## <a name="storage"></a>Storage

Azure Storage は、自動レプリカによるデータ回復性を備えています。 ただし、アプリケーション コードやユーザーが、誤ってまたは悪意を持ってデータを破損させるのを防ぐことはできません。 アプリケーション エラーやユーザー エラーが発生した場合にデータの忠実性を維持するには、監査ログと共にセカンダリ ストレージの場所にデータをコピーするなどの高度な手法が必要です。 

- **ブロック BLOB**:  各ブロック BLOB の特定時点のスナップショットを作成します。 詳細については、「 [BLOB のスナップショットの作成](/rest/api/storageservices/creating-a-snapshot-of-a-blob)」を参照してください。 各スナップショットについて、BLOB 内で前回のスナップショットの状態との差分を保存するために必要なストレージのみに課金されます。 スナップショットには、差分を取るための基になる BLOB が存在している必要があるため、別の BLOB または別のストレージ アカウントにコピーすることをお勧めします。 これにより、バックアップ データを誤って削除してしまうことから確実に保護できます。 [AzCopy](/azure/storage/common/storage-use-azcopy) または [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) を使用して、別のストレージ アカウントに BLOB をコピーできます。

- **ファイル**:  [共有のスナップショット](/azure/storage/files/storage-snapshots-files)を使用するか、AzCopy または PowerShell を使用して別のストレージ アカウントにファイルをコピーします。

- **テーブル**:  AzCopy を使用して、他のリージョンの別のストレージ アカウントにテーブル データをエクスポートします。

## <a name="database"></a>データベース

### <a name="azure-sql-database"></a>Azure SQL Database 

SQL Database は、データ損失からビジネスを守るために、データベースの完全バックアップ (毎週)、データベースの差分バックアップ (1 時間ごと)、およびトランザクション ログのバックアップ (5 ～ 10 分ごと) を組み合わせて自動的に実行します。 ポイントインタイム リストアを使用して、データベースを以前の状態に復元します。 詳細については、次を参照してください。

- [データベースの自動バックアップを使用した Azure SQL Database の復旧](/azure/sql-database/sql-database-recovery-using-backups)

- [Azure SQL Database によるビジネス継続性の概要](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>VM 上の SQL Server

VM 上で実行される SQL Server の場合、従来のバックアップとログ配布の 2 つのオプションがあります。 従来のバックアップを使用すると、特定の時点の状態に復元できますが、回復プロセスに時間がかかります。 従来のバックアップを復元するには、最初に完全バックアップから開始し、その後にそれ以降に作成されたバックアップを適用する必要があります。 2 番目のオプションでは、ログ バックアップの復元を (たとえば 2 時間) 遅延するログ配布セッションを構成します。 これにより、プライマリで発生したエラーから回復するための時間が確保されます。

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB では、一定の間隔でバックアップを自動的に作成します。 バックアップは別のストレージ サービスに個別に保存されます。これらのバックアップは、リージョンの障害に対する回復性を確保するためにグローバルにレプリケートされます。 データベースまたはコレクションを誤って削除した場合は、サポート チケットを申請するか、Azure サポートに連絡して、最新の自動バックアップからデータを復元できます。 詳細については、「[Azure Cosmos DB での自動オンライン バックアップと復元](/azure/cosmos-db/online-backup-and-restore)」をご覧ください。

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL、Azure Database for PostreSQL

Azure Database for MySQL または Azure Database for PostreSQL を使用すると、5 分ごとにサービスのバックアップが自動的に作成されます。 自動バックアップ機能を使用して、過去の特定の時点までサーバーとそのサーバーのすべてのデータベースを新しいサーバーに復元できます。 詳細については、次を参照してください。

- [Azure Portal を使用して Azure Database for MySQL サーバーのバックアップと復元を行う方法](/azure/mysql/howto-restore-server-portal)

- [Azure Portal を使用した Azure Database for PostgreSQL サーバーのバックアップと復元方法](/azure/postgresql/howto-restore-server-portal)

