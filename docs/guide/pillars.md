---
title: ソフトウェア品質の重要な要素
titleSuffix: Azure Application Architecture Guide
description: ソフトウェア品質の 5 つの重要な要素である拡張性、可用性、回復性、管理性、セキュリティについて説明します。
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 76870f58fc957f6d82f6dc176d1c538c795a7d20
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243063"
---
# <a name="pillars-of-software-quality"></a>ソフトウェア品質の重要な要素

成功するクラウド アプリケーションでは、ソフトウェア品質の 5 つの重要な要素である、拡張性、可用性、回復性、管理性、セキュリティに重点が置かれています。

| 重要な要素 | 説明 |
|--------|-------------|
| スケーラビリティ | 増加した負荷を処理するシステムの能力です。 |
| 可用性 | システムが動作し実際に使用可能である時間の割合です。 |
| 回復性 | 障害から回復して動作を続行するシステムの能力です。 |
| 管理 | 運用環境でシステムを継続的に動作させる運用プロセスです。 |
| セキュリティ | 脅威からアプリケーションとデータを保護することです。 |

## <a name="scalability"></a>スケーラビリティ

拡張性は、増加した負荷を処理する、システムの能力です。 アプリケーションをスケーリングする方法は、主に 2 つあります。 垂直スケーリング (スケール*アップ*) は、VM のサイズを大きくするなど、リソースの容量を増やすことです。 水平スケーリング (スケール*アウト*) とは、VM やデータベース レプリカなど、リソースの新しいインスタンスを追加することです。

垂直スケーリングに比べて、水平スケーリングには、次のような大きな利点があります。

- まさしく、クラウド規模です。 何百何千台ものノードで動作し、1 台のノードでは不可能なスケールを実現するように、アプリケーションを設計できます。
- 水平スケールは柔軟です。 負荷が増加した場合にさらにインスタンスを追加することや、負荷が低い期間にそれらを削除することができます。
- スケールアウトは、予定に合わせて、または負荷の変化に応じて、自動的にトリガーできます。
- スケールアウトは、スケール アップよりも低コストになる場合があります。 複数の小さな VM を実行するほうが、1 つの大きな VM を実行するよりもコストを抑えられます。
- 水平スケーリングでは、冗長性を追加して回復性を向上させることができます。 インスタンスが停止した場合も、アプリケーションは動作し続けます。

垂直スケーリングの利点は、アプリケーションに変更を加えずに行えるということです。 ただし、ある時点で上限に達し、それ以上は何もスケール アップできなくなります。 その時点で、さらにスケーリングする場合は、水平スケーリングにする必要があります。

水平スケールは、システムに対して設計する必要があります。 たとえば、VM をスケールアウトするには、それをロード バランサーより後ろに配置します。 ただし、プール内の各 VM が、どのようなクライアント要求でも処理できる必要があります。そのため、アプリケーションは、ステートレスであるか、状態を外部 (たとえば、分散キャッシュ内) に保存する必要があります。 管理された PaaS サービスには、多くの場合、水平スケーリングと自動スケーリングが組み込まれています。 これらのサービスのスケーリングを簡単に行えることは、PaaS サービスを使用する大きな利点です。

ただし、インスタンスの数を増やすだけでは、アプリケーションをスケーリングすることにはなりません。 別の場所のボトルネックが増大するだけになる可能性があります。 たとえば、より多くのクライアント要求を処理するように Web フロントエンドをスケーリングすると、データベース内のロック競合が引き起こされる可能性があります。 その場合は、データベースのスループットを向上させるために、オプティミスティック コンカレンシーやデータのパーティション分割など、その他の対策を検討する必要があります。

パフォーマンスと負荷のテストを常に実行して、このような潜在的なボトルネックを検出するようにします。 データベースなど、システムのステートフルな部分は、ボトルネックの最も一般的な原因であり、慎重に設計して水平にスケーリングする必要があります。 あるボトルネックを解決すると、別の場所の別のボトルネックが明らかになる場合があります。

[拡張性のチェックリスト][scalability-checklist]を使用して、拡張性の観点からご自分の設計を再確認してください。

### <a name="scalability-guidance"></a>拡張性のガイダンス

- [拡張性とパフォーマンスのための設計パターン][scalability-patterns]
- ベスト プラクティス:[自動スケーリング][autoscale]、[バックグラウンド ジョブ][background-jobs]、[キャッシュ][ caching]、[CDN][cdn]、[データのパーティション分割][data-partitioning]

## <a name="availability"></a>可用性

可用性とは、システムが動作し実際に使用可能である時間の割合です。 通常は、アップタイムの割合として測定されます。 アプリケーション エラー、インフラストラクチャの問題、システムの負荷はすべて、可用性を低下させる可能性があります。

クラウド アプリケーションには、期待される可用性、および可用性をどのように測定するかを明確に定義するサービス レベル目標 (SLO) が必要です。 可用性を定義するときには、クリティカル パスを確認します。 Web フロント エンドは、クライアント要求を処理できるかもしれませんが、Web フロント エンドがデータベースに接続できずにすべてのトランザクションが失敗した場合は、ユーザーがアプリケーションを利用できなくなります。

可用性は、よく "9s" (9 の数) の観点で説明されます。たとえば、"four 9s" (4 つの 9) はアップタイムが 99.99% であることを意味します。 次の表は、さまざまな可用性レベルで考えられる累積的なダウンタイムを示しています。

| アップタイムの % | 週当たりのダウンタイム | 月あたりのダウンタイム | 年あたりのダウンタイム |
|----------|-------------------|--------------------|-------------------|
| 99% | 1.68 時間 | 7.2 時間 | 3.65 日 |
| 99.9% | 10 分 | 43.2 分 | 8.76 時間 |
| 99.95% | 5 分 | 21.6 分 | 4.38 時間 |
| 99.99% | 1 分 | 4.32 分 | 52.56 分 |
| 99.999% | 6 秒 | 26 秒 | 5.26 分 |

アップタイム 99% は、週当たり約 2 時間のサービス停止に換算されます。 多くのアプリケーション、特にコンシューマー向けアプリケーションでは、許容できる SLO ではありません。 その一方で、"five 9s" (99.999%) は、*1 年*間のダウンタイムが 5 分以内であることを意味します。 問題の解決はもとより、停止状態を迅速に検出することだけでも十分に困難です。 非常に高い可用性 (99.99% 以上) を得るためには、障害からの復旧を手動操作に頼っているわけにはいきません。 アプリケーションは、自己診断機能と自己復旧機能を備えている必要があり、この点で回復性が非常に重要になります。

Azure では、サービス レベル アグリーメント (SLA) で、アップタイムと接続性について Microsoft のコミットメントが示されています。 特定のサービスの SLA が 99.95% の場合は、そのサービスを 99.95% の時間、使用可能であることが期待できるということです。

アプリケーションは、多くの場合、複数のサービスに依存しています。 一般に、いずれか 1 つのサービスにダウンタイムが発生する確率は、他のサービスには無関係です。 たとえば、ご自分のアプリケーションが 2 つのサービスに依存しており、それぞれの SLA が 99.9% だとします。 両方のサービスの複合的な SLA は、99.9% &times; 99.9% &asymp; 99.8% で、各サービス単体よりもやや少なくなります。

[可用性のチェックリスト][availability-checklist]を使用して、可用性の観点からご自分の設計を確認してください。

### <a name="availability-guidance"></a>可用性のガイダンス

- [可用性のための設計パターン][availability-patterns]
- ベスト プラクティス:[自動スケーリング][autoscale]、[バックグラウンド ジョブ][background-jobs]

## <a name="resiliency"></a>回復性

回復性とは、障害から回復して動作を続行する、システムの能力です。 回復性の目的は、障害の発生後にアプリケーションを十分に機能する状態に戻すことです。 回復性は、可用性に密接に関連しています。

従来のアプリケーション開発では、平均故障間隔 (MTBF) の短縮に重点が置かれていました。 システムに障害が発生しないようにすることに労力が費やされていました。 クラウド コンピューティングでは、次のいくつかの要因により、異なる考え方が必要になります。

- 分散システムは複雑であり、1 か所での障害がシステム全体に連鎖する可能性があります。
- クラウド環境のコストは、コモディティ化したハードウェアの使用によって低く保たれています。そのため、ハードウェア障害が時折発生することは予想しておく必要があります。
- アプリケーションは、多くの場合、外部のサービスに依存しています。外部のサービスは、一時的に使用できなくなることや、利用量の多いユーザーに対しては容量を制限することがあります。
- 今日のユーザーは、アプリケーションはオフラインになることなく毎日 24 時間使用できることを期待しています。

これらの要因すべてが、障害が時折発生することを予期し、障害から回復するようにクラウド アプリケーションを設計する必要があることを示しています。 Azure では、多くの回復性機能があらかじめプラットフォームに組み込まれています。 例: 

- Azure Storage、SQL Database、Cosmos DB はすべて、1 つのリージョン内とリージョン全体の両方で、組み込みのデータ レプリケーションを提供します。
- Azure Managed Disks は、ハードウェア障害の影響を抑えるために、さまざまなストレージ スケール ユニットに自動的に配置されます。
- 可用性セット内の VM は、複数の障害ドメイン間で分散されます。 障害ドメインは、共通の電源とネットワーク スイッチを使用する、VM のグループです。 VM を複数の障害ドメイン間で分散すると、物理的なハードウェア障害、ネットワークの停止、または停電の影響を抑えることができます。

ただし、それでもまだ、ご自分のアプリケーションに回復性を組み込む必要はあります。 回復性戦略は、アーキテクチャのすべてのレベルで適用できます。 一時的なネットワーク障害の後にリモート呼び出しを再試行するなど、一時的な対処としての性質が強い緩和策もあれば、 セカンダリ リージョンへのアプリケーション全体のフェールオーバーなど、より大局的な緩和策もあります。 一時的な対処による緩和策で状況を大きく改善できます。 1 つのリージョンの全体に障害が発生することはほとんどありませんが、ネットワークの輻輳などの一時的な問題は珍しくありません。そのため、まず一時的な問題をターゲットにします。 障害発生時に障害を検出し、根本原因を特定するために、監視と診断を適切に行うことも重要です。

回復性のあるアプリケーションを設計するときには、ご自分の可用性要件を理解する必要があります。 どの程度のダウンタイムを許容できるのでしょうか。 これは、ある程度はコストによって決まります。 起こり得るダウンタイムによって貴社のビジネスが被るコストはいくらになるでしょうか。 アプリケーションの可用性を高めることにいくら投資すべきでしょうか。

[回復性のチェックリスト][resiliency-checklist]を使用して、回復性の観点からご自分の設計を確認してください。

### <a name="resiliency-guidance"></a>回復性のガイダンス

- [回復性に優れた Azure 用アプリケーションの設計][resiliency]
- [回復性のための設計パターン][resiliency-patterns]
- ベスト プラクティス:[一時的な障害の処理][transient-fault-handling]、[特定のサービスの再試行ガイダンス][retry-service-specific]

## <a name="management-and-devops"></a>管理性と DevOps

この要素は、運用環境においてアプリケーションの稼働を維持する運用プロセスを対象とするものです。

デプロイは、信頼性が高く、予測可能である必要があります。 人的ミスの可能性を下げるために、自動化されている必要があります。 迅速かつ定期的なプロセスであるべきで、新機能やバグ修正のリリースを遅らせることはありません。 同じくらい重要なのは、　更新プログラムに問題がある場合に、ご自分で迅速にロールバックまたはロールフォワードできる必要があるということです。

監視と診断は非常に重要です。 クラウド アプリケーションは、リモートのデータセンターで実行されます。ご自分でそのインフラストラクチャを、または場合によってはオペレーティング システムを、完全に制御することはできません。 大規模なアプリケーションでは、VM にログインして問題をトラブルシューティングすることや、複数のログ ファイルを調べることは現実的ではありません。 PaaS サービスでは、ログインする専用の VM すら存在しない場合があります。 システムに関する分析情報が監視と診断によって提供されるため、いつどこで障害が発生したかがわかります。 すべてのシステムは監視できる必要があります。 すべてのシステムにわたりイベント同士を関連付けることができる、共通の、一貫性のあるログ スキーマを使用します。

監視と診断のプロセスには、次のいくつかの異なるフェーズがあります。

- インストルメンテーション。 アプリケーション ログ、Web サーバー ログ、Azure プラットフォームに組み込まれている診断などのソースから生データを生成します。
- 収集と保存。 データを 1 か所に統合します。
- 分析と診断。 問題をトラブルシューティングし、全体の正常性を確認します。
- 視覚化とアラート。 テレメトリ データを使用して、傾向をつかみ、運用チームに警告します。

[DevOps チェックリスト][devops-checklist]を使用して、管理性と DevOps の観点からご自分の設計を確認してください。

### <a name="management-and-devops-guidance"></a>管理性と DevOps のガイダンス

- [管理と監視のための設計パターン][management-patterns]
- ベスト プラクティス:[監視と診断][monitoring]

## <a name="security"></a>セキュリティ

設計と実装から、デプロイと運用まで、アプリケーションのライフ サイクル全体を通してセキュリティを考える必要があります。 Azure プラットフォームでは、ネットワークへの侵入や DDoS 攻撃など、さまざまな脅威に対する保護を提供します。 ただし、それでもまだ、ご自分のアプリケーションと DevOps プロセスにセキュリティを組み込む必要はあります。

ここでは、いくつかの一般的なセキュリティ領域について考えてみましょう。

### <a name="identity-management"></a>ID 管理

ユーザーの認証と承認に Azure Active Directory (Azure AD) を使用することについて考えてみます。 Azure AD は、フル マネージドの ID 管理およびアクセス管理のサービスです。 これを使用して Azure 上にだけ存在するドメインを作成することや、ご自分のオンプレミスの Active Directory ID と統合することができます。 Azure AD は、Office365、Dynamics CRM Online、および多くのサードパーティ製 SaaS アプリケーションとも統合できます。 コンシューマー向けアプリケーションの場合、Azure Active Directory B2C では、ユーザーは自身の既存のソーシャル アカウント (Facebook、Google、LinkedIn など) で認証することや、Azure AD によって管理される、新しいユーザー アカウントを作成することができます。

オンプレミスの Active Directory 環境を Azure ネットワークと統合する場合は、ご自分の要件に応じて、いくつかの手法を使用できます。 詳細については、[ID 管理][identity-ref-arch]の参照アーキテクチャをご覧ください。

### <a name="protecting-your-infrastructure"></a>インフラストラクチャの保護

デプロイする Azure リソースへのアクセスを制御します。 すべての Azure サブスクリプションには、Azure AD テナントとの[信頼関係][ad-subscriptions]があります。
[ロールベースのアクセス制御][rbac] (RBAC) を使用して、ご自分の組織内のユーザーに Azure リソースに対する適切なアクセス許可を付与します。 特定のスコープでユーザーまたはグループに RBAC のロールを割り当てて、アクセス権を付与します。 スコープには、サブスクリプション、リソース グループ、または 1 つのリソースを指定できます。 インフラストラクチャに対するすべての変更を[監査][resource-manager-auditing]します。

### <a name="application-security"></a>アプリケーションのセキュリティ

一般に、アプリケーション開発でのセキュリティのベスト プラクティスは、クラウドにも当てはまります。 これらには、あらゆる場所での SSL の使用、CSRF 攻撃と XSS 攻撃からの保護、SQL インジェクション攻撃の防止などがあります。

クラウド アプリケーションでは、多くの場合、アクセス キーを持つ管理されたサービスが使用されます。 これらがソース管理に含まれることはありません。 Azure Key Vault にアプリケーション シークレットを保存することについて考えてみます。

### <a name="data-sovereignty-and-encryption"></a>データの支配権と暗号化

Azure の高可用性を使用するときは、ご自分のデータが適切な地理的ゾーンに残っていることを確認してください。 Azure の geo レプリケートされている ストレージでは、同じ地理的リージョン内の[ペアのリージョン][paired-region]の概念が使用されます。

暗号化キーおよびシークレットを保護するには、Key Vault を使用します。 Key Vault を使用すると、ハードウェア セキュリティ モジュール (HSM) によって保護されているキーを使用して、キーとシークレットを暗号化できます。 多くの Azure Storage サービスおよび DB サービスでは、[Azure Storage][storage-encryption]、[Azure SQL Database][sql-db-encryption]、[Azure SQL Data Warehouse][data-warehouse-encryption]、[Cosmos DB][cosmosdb-encryption] など、保存データの暗号化がサポートされています。

### <a name="security-resources"></a>セキュリティ リソース

- [Azure Security Center][security-center] は、お客様のすべての Azure サブスクリプションにわたり、統合されたセキュリティ監視とポリシー管理を提供します。
- [Azure のセキュリティのドキュメント][security-documentation]
- [Microsoft セキュリティ センター][trust-center]

<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md

<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md

<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
