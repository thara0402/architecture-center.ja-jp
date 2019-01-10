---
title: Azure サービスの回復性のチェックリスト
titleSuffix: Azure Design Review Framework
description: さまざまな Azure サービスの回復性のガイダンスを提供するチェックリストです。
author: petertaylor9999
ms.date: 11/26/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: e1fb780cf9f54a5078cc5d3c6b597b351f93e05e
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112670"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>特定の Azure サービスの回復性のチェックリスト

回復性とは、障害から回復して引き続き機能するシステムの能力であり、[ソフトウェア品質の重要な要素](../guide/pillars.md)の 1 つです。 各テクノロジには独自の障害モードがあり、アプリケーションの設計および実装の際に考慮する必要があります。 このチェックリストを使用して、個々の Azure サービスの回復性の考慮事項を確認します。 また、[一般的な回復性のチェックリスト](./resiliency.md)も確認します。

## <a name="app-service"></a>App Service

**Standard または Premium レベルを使用します。** これらの階層は、ステージング スロットと自動バックアップをサポートします。 詳細については、「[Azure App Service プランの詳細な概要](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)」を参照してください

**スケールアップまたはスケールダウンを回避します｡** 代わりに、通常の負荷の下でパフォーマンスの要件を満たす階層とインスタンスのサイズを選択してから、インスタンスを[スケールアウト](/azure/app-service-web/web-sites-scale/)してトラフィック量の変化を処理してください。 スケールアップとスケールダウンにより、アプリケーションの再起動がトリガーされることがあります。

**構成をアプリ設定として格納します。** アプリ設定を使用して、構成設定をアプリケーション設定として格納してください。 Resource Manager テンプレートに、または PowerShell 使用して設定を定義して、それを自動化されたデプロイや更新プロセスの一部として適用できるようにしてください。それによって信頼性が向上します。 詳細については、「[Azure App Service での Web アプリの構成](/azure/app-service-web/web-sites-configure/)」をご覧ください。

**運用とテスト用に別々の App Service プランを作成します。** テストには運用環境のデプロイのスロットを使用しないでください。  同じ App Service プラン内のすべてのアプリが同じ VM インスタンスを共有します。 運用デプロイとテスト デプロイを同じプランに入れると、運用デプロイに悪影響を与える可能性があります。 たとえば、負荷テストは実稼働の運用サイトを低下させる可能性があります。 テスト デプロイを別のプランに入れることで、運用バージョンから分離してください。

**Web アプリから Web API を分離します。** ソリューションに Web フロントエンドと Web API の両方がある場合は、それらを別々の App Service アプリに分解することを検討してください。 この設計により、ソリューションをワークロード別に分解しやすくなります。 Web アプリと API を別々の App Service プランで実行できるため、それらを個別にスケーリングできるようになります。 最初はこのレベルのスケーラビリティが不要な場合は、アプリを同じプランにデプロイし、後で必要に応じて別のプランに移行することができます。

**Azure SQL データベースをバックアップするのに App Service バックアップ機能を使用しないでください。** 代わりに、[SQL Database 自動バックアップ][sql-backup]を使用してください。 App Service のバックアップはデータベースを SQL .bacpac ファイルにエクスポートし、これには DTU のコストがかかります。

**ステージング スロットにデプロイします。** ステージング用のデプロイ スロットを作成します。 アプリケーションの更新プログラムをステージング スロットにデプロイし、運用環境にスワップする前にデプロイを確認してください。 これにより、運用環境で不正な更新が発生する可能性が減少します。 また、すべてのインスタンスが、運用環境にスワップする前に確実にウォーム アップされます。 多くのアプリケーションではウォームアップとコールドスタートに多くの時間がかかります。 詳細については、「[Azure App Service の Web アプリのステージング環境を設定する](/azure/app-service-web/web-sites-staged-publishing/)」をご覧ください。

**前回正常起動時 (LKG) のデプロイを保持するデプロイ スロットを作成します。** 運用環境に更新プログラムをデプロイするときは、以前の運用環境のデプロイを LKG スロットに移動してください。 これにより、不適切なデプロイをより簡単にロールバックできます。 後で問題を見つけた場合は、LKG バージョンにすばやく戻ることができます。 詳細については、[基本的な Web アプリケーション](../reference-architectures/app-service-web-app/basic-web-app.md)に関するページをご覧ください。

アプリケーションのログ記録や Web サーバーのログ記録を含む、**診断のログ記録を有効にしてください**。 監視と診断にはログ記録が重要です。 「[Azure App Service での Web アプリの診断ログの有効化](/azure/app-service-web/web-sites-enable-diagnostic-log/)」をご覧ください。

**Blob Storage にログを記録します。** これによって、データの収集と分析が簡単になります。

**ログ用の別のストレージ アカウントを作成します。** ログとアプリケーション データに同じストレージ アカウントを使用しないでください。 これは、ログ記録によってアプリケーションのパフォーマンスが低下するのを防ぐのに役立ちます。

**パフォーマンスを監視します。** [New Relic](https://newrelic.com/) や[Application Insights](/azure/application-insights/app-insights-overview/) などのパフォーマンス監視サービスを使用して、負荷がかかった状態のアプリケーションのパフォーマンスと動作を監視してください。  パフォーマンスの監視により、アプリケーションをリアルタイムで理解できます。 これにより、問題を診断し、障害の根本原因分析を行うことができます。

## <a name="application-gateway"></a>Application Gateway

**少なくとも 2 つのインスタンスをプロビジョニングします。** 少なくとも 2 つのインスタンスのある Application Gateway をデプロイしてください。 単一のインスタンスは、単一障害点です。 冗長性とスケーラビリティのために、2 つ以上のインスタンスを使用してください。 [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway) に適合するためには、2 つ以上のメディアまたは大きめのインスタンスをプロビジョニングする必要があります。

## <a name="cosmos-db"></a>Cosmos DB

**リージョン間でデータベースをレプリケートします。** Cosmos DB では、Cosmos DB データベース アカウントに Azure リージョンをいくつでも関連付けることができます。 Cosmos DB データベースには、1 つの書き込みリージョンと複数の読み取りリージョンを含めることができます。 書き込みリージョンに障害がある場合は、別のレプリカから読み取ることができます。 クライアント SDK はこれを自動的に処理します。 書き込みリージョンを別のリージョンにフェールオーバーすることもできます。 詳細については、「[Azure Cosmos DB を使用してデータをグローバルに分散させる方法](/azure/cosmos-db/distribute-data-globally)」をご覧ください。

## <a name="event-hubs"></a>Event Hubs

**チェックポイントを使用します**。  イベント コンシューマーは、事前定義された間隔で永続的ストレージに現在位置を書き込む必要があります。 このように、コンシューマーで障害が発生した場合 (たとえば、コンシューマーがクラッシュした場合や、ホストに障害が発生した場合)、新しいインスタンスは最後に記録された位置からストリームの読み取りを再開できます。 詳細については、[イベント コンシューマー](/azure/event-hubs/event-hubs-features#event-consumers)に関するページをご覧ください。

**重複メッセージを処理します**。 イベント コンシューマーに障害が発生すると、メッセージの処理は最後に記録されたチェックポイントから再開されます。 最後のチェックポイントの後で既に処理されたメッセージはすべて、再度処理されます。 したがって、メッセージ処理ロジックがべき等であるか、アプリケーションでメッセージの重複を除去できる必要があります。

**例外を処理します**。 イベント コンシューマーは、通常、ループ内のメッセージをバッチで処理します。 1 つのメッセージによって例外が発生した場合は、メッセージのバッチ全体を失わないように、その処理ループ内の例外を処理する必要があります。

**配信不能キューを使用します**。 メッセージを処理した結果、一時的でないエラーが発生した場合は、状態を追跡できるように、メッセージを配信不能キューに配置します。 シナリオによっては、メッセージを後で再試行したり、補正トランザクションを適用したり、他の何らかのアクションを実行したりする可能性があります。 Event Hubs には、組み込みの配信不能キュー機能がないことに注意してください。 Azure Queue Storage または Service Bus を使用して配信不能キューを実装したり、Azure Functions またはその他のイベント処理メカニズムを使用したりすることができます。

**セカンダリ Event Hubs 名前空間にフェールオーバーして、ディザスター リカバリーを実装します。** 詳細については、「[Azure Event Hubs geo ディザスター リカバリー](/azure/event-hubs/event-hubs-geo-dr)」を参照してください。

## <a name="redis-cache"></a>Redis Cache

**geo レプリケーションを構成します。** geo レプリケーションは、Premium レベルの Azure Redis Cache の 2 つのインスタンスをリンクするメカニズムを用意しています。 プライマリ キャッシュに書き込まれたデータは、読み取り専用のセカンダリ キャッシュにレプリケートされます。 詳細については、「[Azure Redis Cache の geo レプリケーションの構成方法](/azure/redis-cache/cache-how-to-geo-replication)」をご覧ください。

**データ永続化を構成します。** Redis の永続化を使用すると、Redis に格納されたデータを保持できます。 また、スナップショットを取得したりデータをバックアップしたりして、ハードウェア障害のときに読み込むことができます。 詳細については、「[Premium Azure Redis Cache のデータ永続化の構成方法](/azure/redis-cache/cache-how-to-premium-persistence)」をご覧ください。

永続ストアとしてではなく、一時的なデータ キャッシュとして Redis Cache を使用する場合、これらの推奨事項は適用されません。

## <a name="search"></a>Search

**複数のレプリカをプロビジョニングします。** 読み取りの高可用性のためには少なくとも 2 つのレプリカ、読み取り/書き込みの高可用性のためには少なくとも 3 つのレプリカを使用してください。

**複数リージョンのデプロイ用のインデクサーを構成します。** 複数リージョンのデプロイがある場合は、インデックス作成の継続性のためのオプションを検討してください。

- データ ソースが geo レプリケーションされている場合は、一般に、各リージョンの Azure Search サービスのそれぞれのインデクサーをそのローカル データ ソース レプリカにポイントする必要があります。 ただし、このアプローチは、Azure SQL データベースに格納されている大規模なデータセットは推奨されません。 その理由は、Azure Search がセカンダリ SQL Database レプリカからは増分インデックス作成を実行できず、プライマリ レプリカからしか実行できないためです。 代わりに、すべてのインデクサーをプライマリ レプリカにポイントしてください。 フェールオーバー後は、新しいプライマリ レプリカで Azure Search インデクサーをポイントします。

- データ ソースが geo レプリケートされていない場合は、同じデータ ソースにある複数のインデクサーをポイントして、複数のリージョン内の Azure Search サービスがデータ ソースからのインデックス作成を継続的に単独で行うようにしてください。 詳細については、「[Azure Search のパフォーマンスと最適化に関する考慮事項][search-optimization]」をご覧ください。

## <a name="service-bus"></a>Service Bus

**運用ワークロードには Premium レベルを使用します**。 [Service Bus の Premium メッセージング](/azure/service-bus-messaging/service-bus-premium-messaging)では、専用の予約済み処理リソースと、予測可能なパフォーマンスとスループットをサポートするためのメモリ容量が提供されます。 また、Premium メッセージング レベルでは、Premium のお客様だけがいち早く利用できる新機能にもアクセスできます。 予想されるワークロードに基づいて、メッセージング ユニットの数を決定できます。

**重複メッセージを処理します**。 メッセージ送信後すぐにパブリッシャーで障害が発生した場合、またはネットワークやシステムで問題が発生した場合は、メッセージが配信されたことをエラーのため記録できないことがあり、同じメッセージがシステムに 2 回送信される可能性があります。 Service Bus では、重複の検出を有効にすることで、この問題を処理できます。 詳しくは、「[重複検出](/azure/service-bus-messaging/duplicate-detection)」をご覧ください。

**例外を処理します**。 メッセージング API では、ユーザー エラー、構成エラー、その他のエラーが発生すると、例外が生成されます。 クライアント コード (送信側と受信側) では、コード内でこれらの例外を処理する必要があります。 バッチ処理では、例外処理を使用してメッセージのバッチ全体が失われるのを防ぐことができるので、特に重要です。 詳しくは、「[Service Bus メッセージングの例外](/azure/service-bus-messaging/service-bus-messaging-exceptions)」をご覧ください。

**再試行ポリシー**。 Service Bus を使用すると、アプリケーションに最適な再試行ポリシーを選択できます。 既定のポリシーでは、最大 9 回の再試行が許され、待機時間は 30 秒ですが、これをさらに調整できます。 詳しくは、[Service Bus の再試行ポリシー](/azure/architecture/best-practices/retry-service-specific#service-bus)に関するページをご覧ください。

**配信不能キューを使用します**。 複数回再試行しても、メッセージを処理できないか、またはどの受信者にも配信できない場合、メッセージは配信不能キューに移動されます。 配信不能キューからメッセージを読み取り、検査して、問題を修復するプロセスを実装します。 シナリオに応じて、そのままのメッセージで再試行したり、メッセージを変更して再試行したり、メッセージを破棄したりします。 詳しくは、「[Service Bus の配信不能キューの概要](/azure/service-bus-messaging/service-bus-dead-letter-queues)」をご覧ください。

**geo ディザスター リカバリーを使用します**。 geo ディザスター リカバリーを使用すると、Azure リージョン全体またはデータセンターが災害により使用できなくなった場合でも、別のリージョンまたはデータセンターでデータの処理が続けられることが保証されます。 詳細については、「[Azure Service Bus の geo ディザスター リカバリー](/azure/service-bus-messaging/service-bus-geo-dr)」を参照してください。

## <a name="storage"></a>Storage

**アプリケーション データには、読み取りアクセス geo 冗長ストレージ (RA-GRS)** を使用します。 RA-GRS ストレージは、セカンダリ リージョンにデータをレプリケートし、セカンダリ リージョンからの読み取り専用のアクセスを提供します。 プライマリ リージョンでストレージが停止した場合、アプリケーションはセカンダリ リージョンからデータを読み取ることができます。 詳細については、「[Azure Storage のレプリケーション](/azure/storage/storage-redundancy/)」をご覧ください。

**VM ディスクには、Managed Disks を使用します。** [Managed Disks][managed-disks] では、ディスクが単一障害点にならないように相互に十分に分離されるため、可用性セットの VM の信頼性が向上します。 また、Managed Disks はストレージ アカウントで作成した VHD の IOPS 制限の対象ではありません。 詳細については、「[Azure での Windows 仮想マシンの可用性の管理][vm-manage-availability]」をご覧ください。

**Queue Storage では、別のリージョンにバックアップ キューを作成します。** Queue Storage の場合、項目のキューやデキューができないため、読み取り専用レプリカの使用は制限されています。 代わりに、別のリージョンのストレージ アカウントにバックアップ キューを作成します。 ストレージの障害がある場合、アプリケーションは、プライマリ リージョンが再び使用可能になるまで、バックアップ キューを使用できます。 したがって、アプリケーションは、引き続き新しい要求を処理できます。

## <a name="sql-database"></a>SQL Database

**Standard または Premium レベルを使用します。** これらの階層は、より長いポイントインタイム リストア期間 (35 日間) を提供します。 詳細については、「[SQL Database のオプションとパフォーマンス](/azure/sql-database/sql-database-service-tiers/)」をご覧ください。

**SQL Database の監査を有効にします。** 監査は、悪意のある攻撃や人的ミスの診断に使用できます。 詳細については、「[SQL Database 監査の使用](/azure/sql-database/sql-database-auditing-get-started/)」をご覧ください。

**アクティブ Geo レプリケーションを使用します。** アクティブ Geo レプリケーションを使用して、さまざまなリージョンに読み取り可能なセカンダリ レプリカを作成します。  プライマリ データベースに障害が発生した場合、またはオフラインにする必要がある場合は、セカンダリ データベースへの手動フェールオーバーを実行します。  セカンダリ データベースは、フェールオーバーするまでは読み取り専用のままです。  詳細については、「[Azure SQL Database のアクティブ geo レプリケーション](/azure/sql-database/sql-database-geo-replication-overview/)」をご覧ください。

**シャーディングを使用します。** シャーディングを使用して、データベースを水平方向にパーティション分割することを検討してください。 シャーディングによって障害を分離できます。 詳細については、「[Azure SQL Database によるスケール アウト](/azure/sql-database/sql-database-elastic-scale-introduction/)」をご覧ください。

**人的ミスから復旧するには、ポイントインタイム リストアを使用します。**  ポイントインタイム リストアは、データベースを以前の特定の時点に戻します。 詳細については、「[データベースの自動バックアップを使用した Azure SQL Database の復旧][sql-restore]」をご覧ください。

**サービスの停止から復旧するには、geo リストアを使用します。** geo リストアは geo 冗長バックアップからデータベースを復元します。  詳細については、「[データベースの自動バックアップを使用した Azure SQL Database の復旧][sql-restore]」をご覧ください。

## <a name="sql-data-warehouse"></a>SQL Data Warehouse

**geo バックアップを無効にしないでください。** 既定では、SQL Data Warehouse により、ディザスター リカバリー用にデータの完全バックアップが 24 時間ごとに作成されます。 この機能を無効にすることはお勧めしません。 詳細については、「[geo バックアップ](/azure/sql-data-warehouse/backup-and-restore#geo-backups)」を参照してください。

## <a name="sql-server-running-in-a-vm"></a>VM で実行されている SQL Server

**データベースをレプリケートします。** データベースをレプリケートするには、SQL Server Always On 可用性グループを使用します。 1 つの SQL Server インスタンスが失敗した場合は、高可用性を提供してください。 詳細については、「[n 層アプリケーションの Windows VM を実行する](../reference-architectures/virtual-machines-windows/n-tier.md)」をご覧ください。

**データベースをバックアップします。** 既に [Azure Backup](/azure/backup/) を使用して VM をバックアップしている場合は、[DPM を使用した SQL Server ワークロード用 Azure Backup](/azure/backup/backup-azure-backup-sql/) を使用することを検討してください。 このアプローチでは、組織用の 1 つのバックアップ管理者の役割と、VM および SQL Server 用の 1 つにまとめられた復旧手順があります。 それ以外の場合は、[Microsoft Azure への SQL Server マネージド バックアップ](https://msdn.microsoft.com/library/dn449496.aspx)を使用してください。

## <a name="traffic-manager"></a>Traffic Manager

**手動フェールバックを実行します。** Traffic Manager のフェールオーバーの後は、自動的にフェールバックするのではなく、手動フェールバックを実行してください。 フェールバックする前に、すべてのアプリケーション サブシステムが正常であることを確認してください。  これを行わないと、データセンター間でアプリケーションが切り替わる状況が発生する可能性があります。 詳細については、「[高可用性を得るために複数のリージョンで VM を実行する](../reference-architectures/virtual-machines-windows/multi-region-application.md)」をご覧ください。

**正常性プローブのエンドポイントを作成します。** アプリケーションの全体的な正常性についてレポートするカスタム エンドポイントを作成してください。 これにより、クリティカル パスが失敗した場合に、フロント エンドだけでなく Traffic Manager をフェールオーバーできます。 エンドポイントは、重要な依存関係が正常でない場合、または到達できない場合に、HTTP エラー コードを返す必要があります。 ただし、重要でないサービスについてはレポートしないでください。 そうしないと、必要のないときに正常性プローブがフェールオーバーをトリガーして、誤検知が生じる可能性があります。 詳細については、「[Traffic Manager エンドポイントの監視とフェールオーバー](/azure/traffic-manager/traffic-manager-monitoring/)」をご覧ください。

## <a name="virtual-machines"></a>Virtual Machines

**単一の VM で運用環境のワークロードを実行しないでください。** 単一の VM のデプロイには計画または計画外メンテナンスに対する回復力がありません。 代わりに、複数の VM を可用性セットまたは [VM スケール セット](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)に配置し、前面にロード バランサーを配置してください。

**VM をプロビジョニングするときに、可用性セットを指定します。** 現時点では、VM がプロビジョニングされた後に、VM を可用性セットに追加する方法はありません。 既存の可用性セットに新しい VM を追加するときは、必ず VM 用の NIC を作成し、NIC をロード バランサーのバックエンド アドレス プールに追加してください。 そうしないと、ロード バランサーがその VM にネットワーク トラフィックをルーティングしなくなります。

**各アプリケーション層を別々の可用性セットに配置します。** N 層のアプリケーションでは、同じ可用性セットの別の層からは VM を配置しないでください。 可用性セット内の VM は、障害ドメイン (FD) と更新ドメイン (UD) にまたがって配置されています。 ただし、FD と UD の冗長性の利点を活用するには、可用性セットのすべての VM が同じクライアント要求を処理できる必要があります。

**Azure Site Recovery を使用して VM をレプリケートします。** [Site Recovery][site-recovery] を使用して Azure VM をレプリケートすると、すべての VM ディスクが、ターゲット リージョンに継続的かつ非同期的にレプリケートされます。 復旧ポイントは数分ごとに作成されます。 これにより目標復旧時点 (RPO) が数分単位で設定されます。 ディザスター リカバリー訓練を、運用環境のアプリケーションまたは実行中のレプリケーションに影響を与えることなく、必要な回数だけ実施できます。 詳細については、「[Azure へのディザスター リカバリー訓練を実行する][site-recovery-test]」を参照してください。

**パフォーマンスの要件に基づいて適切な VM サイズを選択します。** 既存のワークロードを Azure に移動する場合は、オンプレミスのサーバーに最も適合性が高い VM サイズから開始します。 次に、CPU、メモリ、およびディスクの IOPS について、実際のワークロードのパフォーマンスを測定し、必要に応じてサイズを調整します。 これにより、アプリケーションがクラウド環境で期待どおりに動作することを確認できます。 また、複数の NIC が必要な場合は、各サイズの NIC の制限に注意してください。

**VHD 用の Managed Disks を使用します。** [Managed Disks][managed-disks] では、ディスクが単一障害点にならないように相互に十分に分離されるため、可用性セットの VM の信頼性が向上します。 また、Managed Disks はストレージ アカウントで作成した VHD の IOPS 制限の対象ではありません。 詳細については、「[Azure での Windows 仮想マシンの可用性の管理][vm-manage-availability]」をご覧ください。

**OS ディスクではなく、データ ディスクにアプリケーションをインストールします。** そうしないと、ディスク サイズの上限に達する可能性があります。

**Azure Backup を使用して VM をバックアップします。** バックアップにより偶発的なデータ損失から保護されます。 詳細については、「[Recovery Services コンテナーを使用した Azure VM の保護](/azure/backup/backup-azure-vms-first-look-arm/)」をご覧ください。

基本的な正常性メトリック、インフラストラクチャ ログ、および[ブート診断][boot-diagnostics]などの**診断ログ**を有効にします。 VM が起動不可能な状態になった場合は、起動エラーを診断するのにブート診断が役立ちます。 詳細については、「[Azure 診断ログの概要][diagnostics-logs]」をご覧ください。

**AzureLogCollector 拡張機能を使用します。** (Windows VM のみ)この拡張機能は、オペレーターがリモートで VM にログを記録しなくても、Azure プラットフォームのログを集計し、それを Azure ストレージにアップロードします。 詳細については、「[AzureLogCollector 拡張機能](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)」をご覧ください。

## <a name="virtual-network"></a>Virtual Network

**パブリック IP アドレスをホワイトリストに登録したりブロックしたりするには、サブネットに NSG を追加します。** 悪意のあるユーザーからのアクセスをブロックしたり、アプリケーションにアクセスする権限を持つユーザーからのアクセスのみを許可したりしてください。

**カスタムの正常性プローブを作成します。** ロード バランサー正常性プローブでは、HTTP または TCP のいずれかをテストできます。 VM が HTTP サーバーを実行している場合、HTTP プローブは TCP プローブよりも優れた正常性状態のインジケーターです。 HTTP プローブでは、すべての重要な依存関係を含む、アプリケーションの全体的な正常性をレポートするカスタム エンドポイントを使用してください。 詳細については、「[Azure Load Balancer の概要](/azure/load-balancer/load-balancer-overview/)」を参照してください。

**正常性プローブをブロックしないでください。** ロード バランサー正常性プローブは、既知の IP アドレスである 168.63.129.16 から送信されます。 任意のファイアウォール ポリシーまたはネットワーク セキュリティ グループ (NSG) の規則で、この IP を送受信するトラフィックをブロックしないでください。 正常性プローブをブロックすると、ロード バランサーによってローテーションから VM が削除されることがあります。

**ロード バランサーのログ記録を有効にします。** ログには、プローブの応答に失敗したことが原因で、ネットワーク トラフィックを受信していない、バックエンドの VM 数が表示されます。 詳細については、「[Azure Load Balancer のログ分析](/azure/load-balancer/load-balancer-monitor-log/)」をご覧ください。

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[site-recovery]: /azure/site-recovery/
[site-recovery-test]: /azure/site-recovery/site-recovery-test-failover-to-azure
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
