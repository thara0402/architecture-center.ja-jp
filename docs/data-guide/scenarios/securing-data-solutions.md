---
title: "データ保護ソリューション"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2018
---
# <a name="securing-data-solutions"></a>データ保護ソリューション

多くの場合、クラウド内のデータにアクセスできるようにすると、(特にオンプレミスのデータ ストアのみでの作業から移行する場合は) そのデータへのアクセシビリティの向上とそれをセキュリティで保護する新しい方法について問題が生じることがあります。

## <a name="challenges"></a>課題

* 多くのログに格納されているセキュリティ イベントの監視と分析の一元化。
* アプリケーションとサービスをまたがる暗号化および承認管理の実装。
* オンプレミスかクラウド内かにかかわらず、一元化された ID 管理がすべてのソリューション コンポーネントで動作していることの確認。

## <a name="data-protection"></a>データ保護

情報保護の最初の手順は、保護対象の識別です。 存在する場所を問わず最も重要なデータ資産を識別、保護、監視するための適切に伝達された明確でシンプルなガイドラインを作成します。 組織の任務または収益性に過剰な影響を及ぼす資産に対して最も強力な保護を確立します。 これらは、高価値資産または HVA と呼ばれます。 HVA のライフ サイクルとセキュリティの依存関係の厳格な分析を実行し、適切なセキュリティ コントロールと条件を確立します。 同様に、機密資産を識別および分類し、セキュリティ コントロールを自動的に適用するテクノロジとプロセスを定義します。

保護する必要のあるデータを特定したら、"*保存*" データと "*転送中*" のデータを保護する方法を検討します。

* **保存データ**: 磁気ディスクか光ディスクか、オンプレミスかクラウド内かにかかわらず、物理メディアに静的に存在するデータ。
* **転送中のデータ**: ネットワーク経由、サービス バス経由 (オンプレミスとクラウド間)、または入出力処理中に、コンポーネント間、場所間、プログラム間で転送中のデータ。

保存データまたは転送中のデータの保護について詳しくは、「[Azure のデータ セキュリティと暗号化のベスト プラクティス](/azure/security/azure-security-data-encryption-best-practices)」をご覧ください。

## <a name="access-control"></a>Access Control

クラウド内のデータ保護の中心は、ID 管理とアクセス制御の組み合わせです。 さまざまな種類のクラウド サービスと、[ハイブリッド クラウド](../scenarios/hybrid-on-premises-and-cloud.md)の普及により、ID およびアクセス制御ではいくつかの重要なプラクティスに従う必要があります。

* ID 管理を一元化する。
* シングル サインオン (SSO) を有効にする。
* パスワード管理をデプロイする。
* 多要素認証 (MFA) をユーザーに適用する。
* ロールベースのアクセス制御 (RBAC) を使用する。
* ユーザーの場所、デバイスの種類、パッチ レベルなどに関連する追加のプロパティで従来のユーザー ID の概念を強化する条件付きアクセス ポリシーを構成する。
* リソース マネージャーを使用して、リソースが作成される場所を制御する。
* 疑わしいアクティビティを能動的に監視する

詳しくは、「[Azure の ID 管理とアクセス制御セキュリティのベスト プラクティス](/azure/security/azure-security-identity-management-best-practices)」をご覧ください。

## <a name="auditing"></a>監査

前に説明した ID およびアクセスの監視以外に、クラウドで使用するサービスとアプリケーションでは、監視するセキュリティ関連イベントを生成する必要があります。 これらのイベントを監視するための主な課題は、潜在的な問題を回避したり過去の問題のトラブルシューティングを行ったりするために、大量のログを処理することです。 クラウドベースのアプリケーションは多くの変動要素を含む傾向にあり、そのほとんどがある程度のログと利用統計情報を生成します。 一元的な監視および分析を使用して、大量の情報を管理および把握します。

詳しくは、「[Azure のログと監査](/azure/security/azure-log-audit)」をご覧ください。



## <a name="securing-data-solutions-in-azure"></a>Azure でのデータ保護ソリューション

### <a name="encryption"></a>暗号化

**仮想マシン**。 [Azure Disk Encryption](/azure/security/azure-security-disk-encryption) を使用して、Windows または Linux VM 上で接続されたディスクを暗号化します。 このソリューションは、ディスクの暗号化キーとシークレットを制御および管理できるように、[Azure Key Vault](/azure/key-vault/) と統合されています。 

**Azure Storage**。 [Azure Storage Service Encryption](/azure/storage/common/storage-service-encryption) を使用して、Azure Storage 内の保存データを自動的に暗号化します。 暗号化、解読、キーの管理は、ユーザーにはまったく意識されずに行われます。 Azure Key Vault でクライアント側の暗号化を使用して、データを転送中にセキュリティで保護することもできます。 詳しくは、「[Microsoft Azure Storage のクライアント側の暗号化と Azure Key Vault](/azure/storage/common/storage-client-side-encryption)」をご覧ください。

**SQL Database** と **Azure SQL Data Warehouse**。 [Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) を使用して、アプリケーションに変更を加えずに、データベース、関連付けられているバックアップ、トランザクション ログ ファイルの暗号化と解読をリアルタイムで実行できます。 SQL Database で [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) を使用して、サーバーでの保存時、クライアント/サーバー間の移動中、およびデータの使用中に機密データを保護することができます。 Azure Key Vault を使用して、Always Encrypted 暗号化キーを格納することができます。 

### <a name="rights-management"></a>Rights management

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) は、暗号化、ID、承認のポリシーを使用してファイルと電子メールをセキュリティで保護するクラウドベースのサービスです。 電話、タブレット、PC などの複数のデバイスで機能します。 組織の境界外に出てもデータが保護され続けるため、組織内と組織外の両方で情報を保護できます。

### <a name="access-control"></a>アクセス制御

[ロールベースのアクセス制御](/azure/active-directory/role-based-access-control-what-is) (RBAC) を使用して、ユーザー ロールに基づいて Azure リソースへのアクセスを制限します。 オンプレミスの Active Directory を使用している場合は、[Azure AD と同期](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements)して、ユーザーにオンプレミスの ID に基づくクラウド ID を提供できます。

[Azure Active Directory の条件付きアクセス](/azure/active-directory/active-directory-conditional-access-azure-portal)を使用して、特定の条件に基づいて環境内のアプリケーションへのアクセスに対してコントロールを適用します。 たとえば、ポリシー ステートメントは "_請負業者が信頼されていないネットワークから会社のクラウド アプリにアクセスしようとしている場合は、アクセスをブロックします_" のような形式になります。 

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) では、ユーザーと、ユーザーが管理特権で実行するタスクの種類を管理、制御、監視できます。 これは、Azure AD、Azure、Office 365、または SaaS アプリで特権操作を実行できる組織内のユーザーを制限し、そのアクティビティを監視するための重要な手順です。

### <a name="network"></a>ネットワーク

転送中のデータを保護するには、さまざまな場所でデータを交換するときに必ず SSL/TLS 使用します。 場合によっては、オンプレミスとクラウド インフラストラクチャ間の通信チャネル全体を、仮想プライベート ネットワーク (VPN) または [ExpressRoute](/azure/expressroute/) を使用して隔離する必要があります。 詳しくは、「[Extending on-premises data solutions to the cloud](../scenarios/hybrid-on-premises-and-cloud.md)」(オンプレミスのデータ ソリューションをクラウドに拡張する) をご覧ください。

[ネットワーク セキュリティ グループ](/azure/virtual-network/virtual-networks-nsg) (NSG) を使用して、潜在的な攻撃ベクトルの数を減らします。 ネットワーク セキュリティ グループには、ソースまたはターゲット IP アドレス、ポート、およびプロトコルを基に、受信/送信ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。 

[仮想ネットワーク サービス エンドポイント](/azure/virtual-network/virtual-network-service-endpoints-overview)を使用して、Azure SQL または Azure Storage のリソースをセキュリティで保護して、仮想ネットワークからのトラフィックのみがこれらのリソースにアクセスできるようにします。

Azure Virtual Network (VNet) 内の VM は、[仮想ネットワーク ピアリング](/azure/virtual-network/virtual-network-peering-overview)を使用して他の VNet と安全に通信できます。 ピアリングされた仮想ネットワーク間のネットワーク トラフィックはプライベートである。 仮想ネットワーク間のトラフィックは、Microsoft のバックボーン ネットワーク上で保持されます。

詳しくは、「[Azure のネットワーク セキュリティ](/azure/security/azure-network-security)」をご覧ください

### <a name="monitoring"></a>監視

[Azure Security Center](/azure/security-center/security-center-intro) は、真の脅威を検出し、誤検知を減らすために、Azure のリソース、ネットワーク、接続されているパートナー ソリューション (ファイアウォール ソリューションなど) から、ログ データを自動的に収集、分析、統合します。 

[Log Analytics](/azure/log-analytics/log-analytics-overview) は、ログへの一元的なアクセスを提供し、そのデータの分析とカスタム アラートの作成を支援します。

[Azure SQL Database の脅威の検出](/azure/sql-database/sql-database-threat-detection)は、データベースにアクセスしたりデータベースを悪用したりしようとする、異常で有害な可能性がある不自然なアクティビティを検出します。 セキュリティ責任者や他の指定された管理者は、不審なデータベースのアクティビティが発生すると、直ちに通知を受け取ることができます。 それぞれの通知では、不審なアクティビティの詳細情報と、脅威に対して推奨されるさらなる調査方法や軽減策が記載されています。


