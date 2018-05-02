---
title: オンプレミスの AD ドメインと Azure Active Directory を統合する
description: Azure Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを実装する方法について説明します。
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: 9475d669b2cb8888a7ceabed7e36317fe63681fd
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/23/2018
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>オンプレミスの Active Directory ドメインと Azure Active Directory を統合する

Azure Active Directory (Azure AD) は、クラウド ベースでマルチテナントのディレクトリおよび ID サービスです。 この参照アーキテクチャでは、オンプレミスの Active Directory ドメインと Azure AD を統合してクラウド ベースの ID 認証を提供する場合のベスト プラクティスを示します。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

[![0]][0] 

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

> [!NOTE]
> わかりやすくするため、この図では Azure AD に直接関係する接続のみを示してあり、認証および ID フェデレーションの一部として発生する可能性があるプロトコル関連のトラフィックは示されていません。 たとえば、Web アプリケーションは Azure AD を介して要求を認証するため Web ブラウザーをリダイレクトする可能性があります。 認証が済むと、適切な ID 情報と共に要求を Web アプリケーションに戻すことができます。
> 

この参照アーキテクチャの一般的な用途は次のとおりです。

* 組織に属しているリモート ユーザーにアクセスを提供する、Azure にデプロイされた Web アプリケーション。
* パスワードのリセットやグループ管理の委任など、エンド ユーザー向けのセルフ サービス機能の実装。 これには Azure AD Premium Edition が必要であることにご注意ください。
* オンプレミスのネットワークとアプリケーションの Azure VNet が VPN トンネルまたは ExpressRoute 回線を使って接続されていないアーキテクチャ。

> [!NOTE]
> 現在、Azure AD はユーザー認証のみをサポートしています。 SQL Server などの一部のアプリケーションとサービスではコンピューターの認証が必要になることがあり、そのような場合にこのソリューションは適していません。
> 

その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。 

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャには次のコンポーネントがあります。

* **Azure AD テナント**。 組織によって作成された [Azure AD][azure-active-directory] のインスタンス。 オンプレミスの Active Directory からコピーされたオブジェクトを格納することによってクラウド アプリケーションのディレクトリ サービスとして動作し、ID サービスを提供します。
* **Web 層サブネット**。 このサブネットは、Web アプリケーションを実行する VM を保持します。 Azure AD は、このアプリケーションに対する ID ブローカーとして機能できます。
* **オンプレミスの AD DS サーバー**。 オンプレミスのディレクトリおよび ID サービスです。 AD DS ディレクトリは、Azure AD がオンプレミスのユーザーを認証できるように同期することができます。
* **Azure AD Connect 同期サーバー**。 [Azure AD Connect][azure-ad-connect] 同期サービスを実行するオンプレミスのコンピューターです。 このサービスは、オンプレミスの Active Directory に保持されている情報を Azure AD に同期します。 たとえば、オンプレミスのグループとユーザーをプロビジョニングまたはプロビジョニング解除すると、その変更が Azure AD に反映されます。 
  
  > [!NOTE]
  > セキュリティ上の理由により、Azure AD はユーザーのパスワードをハッシュとして格納します。 ユーザーがパスワードをリセットする必要がある場合、オンプレミスでリセットを実行して、新しいハッシュを Azure AD に送信する必要があります。 Azure AD Premium Edition には、このタスクを自動化してユーザーが自分のパスワードをリセットできるようにする機能が含まれます。
  > 

* **N 層アプリケーション用の VM**。 デプロイには、N 層アプリケーション用のインフラストラクチャが含まれます。 これらのリソースについて詳しくは、「[n 層アプリケーションの Windows VM を実行する][implementing-a-multi-tier-architecture-on-Azure]」をご覧ください。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、優先される特定の要件がない限り、従ってください。 

### <a name="azure-ad-connect-sync-service"></a>Azure AD Connect 同期サービス

Azure AD Connect 同期サービスは、クラウドに格納されている ID 情報とオンプレミスに保持されている情報の整合性を保証します。 このサービスは Azure AD Connect ソフトウェアを使ってインストールします。 

Azure AD Connect 同期を実装する前に、組織の同期要件を決定します。 たとえば、同期するもの、同期元のドメイン、同期の頻度などです。 詳しくは、「[ディレクトリ同期要件の決定][aad-sync-requirements]」をご覧ください。

Azure AD Connect 同期サービスは、オンプレミスでホストされている VM またはコンピューターで実行できます。 Active Directory ディレクトリ内の情報の変動の度合いによっては、Azure AD Connect 同期サービスの負荷が、Azure AD と初期同期された後はそれほど高くならない可能性があります。 VM でサービスを実行すると、必要に応じて簡単にサーバーを拡張できます。 監視に関する考慮事項のセクションで説明されているように VM でのアクティビティを監視し、拡張が必要かどうかを判断します。

フォレストに複数のオンプレミス ドメインがある場合は、フォレスト全体の情報を格納して 1 つの Azure AD テナントに同期することをお勧めします。 複数のドメインで発生する ID の情報をフィルター処理し、各 ID が Azure AD に 1 回だけ出現し、重複しないようにします。 重複していると、データを同期するときに不整合が生じる可能性があります。 詳しくは、後のトポロジに関するセクションをご覧ください。 

必要なデータだけが Azure AD に格納されるように、フィルター機能を使います。 たとえば、組織によっては、Azure AD の非アクティブなアカウントに関する情報を格納したくない場合があります。 フィルター処理は、グループ、ドメイン、組織単位 (OU)、または属性に基づいて行うことができます。 フィルターを組み合わせて、複雑なルールを生成できます。 たとえば、ドメインに保持されているオブジェクトのうち、選んだ属性が特定の値のものだけを同期できます。 詳しくは、「[Azure AD Connect Sync: フィルター処理の構成][aad-filtering]」をご覧ください。

AD Connect 同期サービスに高可用性を実装するには、セカンダリ ステージング サーバーを実行します。 詳しくは、トポロジの推奨事項に関するセクションをご覧ください。

### <a name="security-recommendations"></a>セキュリティに関する推奨事項

**ユーザー パスワードの管理**。 Azure AD Premium Edition ではパスワードの書き戻しがサポートされており、オンプレミスのユーザーは Azure Portal 内からセルフ サービスのパスワード リセットを実行できます。 この機能は、組織のパスワード セキュリティ ポリシーを確認した後でのみ有効にする必要があります。 たとえば、自分のパスワードを変更できるユーザーを制限したり、パスワード管理エクスペリエンスを調整したりすることができます。 詳しくは、「[組織ニーズに合わせたパスワード管理のカスタマイズ][aad-password-management]」をご覧ください。 

**外部からアクセスできるオンプレミスのアプリケーションを保護します。** Azure AD アプリケーション プロキシを使って、オンプレミスの Web アプリケーションに対する、外部ユーザーによる Azure AD を介したアクセスを制御します。 Azure ディレクトリに有効な資格情報を持つユーザーにだけ、アプリケーションを使うアクセス許可が与えられます。 詳しくは、「[Azure Portal でアプリケーション プロキシを有効にする][aad-application-proxy]」をご覧ください。

**Azure AD で不審なアクティビティの兆候を積極的に監視します。**    Azure AD Identity Protection が含まれる Azure AD Premium P2 Edition の使用を検討します。 Identity Protection はアダプティブ Machine Learning アルゴリズムとヒューリスティックを使用して、ID が侵害されていることを示している可能性のある異常とリスク イベントを検出します。 たとえば、不規則なサインイン アクティビティ、不明なソースまたは疑わしいアクティビティが発生している IP アドレスからのサインイン、感染しているおそれがあるデバイスからのサインインなど、異常なアクティビティの可能性を検出できます。 このデータを使って、Identity Protection はレポートとアラートを生成し、ユーザーがこれらのリスク イベントを調査して、適切なアクションを実行できるようにします。 詳しくは、「[Azure Active Directory Identity Protection][aad-identity-protection]」をご覧ください。
  
Azure Portal で Azure AD のレポート機能を使って、システムで発生しているセキュリティ関連のアクティビティを監視できます。 レポートの使い方について詳しくは、「[Azure Active Directory レポート ガイド][aad-reporting-guide]」をご覧ください。

### <a name="topology-recommendations"></a>トポロジに関する推奨事項

組織の要件に最も近いトポロジを実装するように Azure AD Connect を構成します。 Azure AD Connect は次のようなトポロジをサポートしています。

* **単一のフォレスト、単一の Azure AD ディレクトリ**。 このトポロジでは、Azure AD Connect が、オンプレミスの単一のフォレストに含まれる 1 つ以上のドメインから、単一の Azure AD テナントに、オブジェクトと ID 情報を同期します。 これは、Azure AD Connect の高速インストールによって実装される既定のトポロジです。
  
  > [!NOTE]
  > 後で説明するステージング モードでサーバーを実行している場合を除き、複数の Azure AD Connect 同期サーバーを使って、オンプレミスの同じフォレストに含まれる異なるドメインを、同じ Azure AD テナントに接続しないでください。
  > 
  > 

* **複数のフォレスト、単一の Azure AD ディレクトリ**。 このトポロジの Azure AD Connect は、複数のフォレストから、単一の Azure AD テナントに、オブジェクトと ID 情報を同期します。 組織にオンプレミスのフォレストが複数ある場合は、このトポロジを使います。 同じユーザーが複数のフォレストに存在する場合でも、個々のユーザーが Azure AD ディレクトリにおいて 1 回だけ表されるように、ID 情報を統合することができます。 すべてのフォレストは、同じ Azure AD Connect 同期サーバーを使います。 Azure AD Connect 同期サーバーは、ドメインの一部になっていなくてもかまいませんが、すべてのフォレストから到達可能である必要があります。
  
  > [!NOTE]
  > このトポロジでは、単一の Azure AD テナントに接続するために、オンプレミスのフォレストごとに異なる Azure AD Connect 同期サーバーを使わないでください。 このようにすると、同じユーザーが複数のフォレストに存在する場合、Azure AD に重複する ID 情報が作成されてしまいます。
  > 
  > 

* **複数のフォレスト、分離トポロジ**。 このトポロジは、異なるフォレストからの ID 情報を単一の Azure AD テナントにマージし、すべてのフォレストを個別のエンティティとして処理します。 このトポロジは、異なる複数の組織からフォレストを結合していて、各ユーザーの ID 情報が 1 つのフォレストにのみ保持されている場合に便利です。
  
  > [!NOTE]
  > 各フォレストのグローバル アドレス一覧 (GAL) が同期されている場合、あるフォレストのユーザーが別のフォレストに連絡先として存在する可能性があります。 このようなことは、組織が GALSync を Forefront Identity Manager 2010 または Microsoft Identity Manager 2016 で実装している場合に、発生する可能性があります。 このシナリオでは、ユーザーが "*メール*" 属性によって識別されるように指定できます。 また、*ObjectSID* および *msExchMasterAccountSID* 属性を使って ID を照合することもできます。 これは、アカウントが無効になったリソース フォレストが存在する場合に便利です。
  > 
  > 

* **ステージング サーバー**。 この構成では、Azure AD Connect 同期サーバーの第 2 のインスタンスを、第 1 のインスタンスと並行して実行します。 この構造は、次のようなシナリオをサポートします。
  
  * 高可用性:
  * Azure AD Connect 同期サーバーの新しい構成のテストとデプロイ。
  * 新しいサーバーの導入と前の構成の使用停止。 
    
    これらのシナリオでは、第 2 のインスタンスは "*ステージング モード*" で実行します。 サーバーは、インポートされたオブジェクトと同期データをデータベースに記録しますが、Azure AD にはデータを渡しません。 ステージング モードを無効にすると、サーバーは Azure AD へのデータの書き込みを開始し、必要に応じてオンプレミスのディレクトリへのパスワードの書き戻しを実行し始めます。 詳しくは、「[Azure AD Connect Sync: 操作タスクおよび考慮事項][aad-connect-sync-operational-tasks]」をご覧ください。

* **複数の Azure AD ディレクトリ**。 組織に対して 1 つの Azure AD ディレクトリを作成することをお勧めしますが、状況によっては、情報を異なる複数の Azure AD ディレクトリに分割することが必要になる場合があります。 そのような場合は、同期とパスワードの書き戻しの問題を防ぐため、オンプレミスのフォレストの各オブジェクトが、ただ 1 つの Azure AD ディレクトリだけに含まれるようにします。 このシナリオを実装するには、Azure AD ディレクトリごとに個別の Azure AD Connect 同期サーバーを構成し、フィルターを使って、各 Azure AD Connect 同期サーバーが、相互に排他的なオブジェクトのセットを処理するようにします。 

これらのトポロジについて詳しくは、「[Azure AD Connect のトポロジ][aad-topologies]」をご覧ください。

### <a name="user-authentication"></a>ユーザー認証

既定では、Azure AD Connect 同期サーバーはオンプレミスのドメインと Azure AD の間にパスワード ハッシュ同期を構成し、Azure AD サービスは、ユーザーがオンプレミスで使っているものと同じパスワードで認証を行うものと想定されています。 多くの組織にはこの方法が適していますが、組織の既存のポリシーとインフラストラクチャを考慮する必要があります。 例: 

* 組織のセキュリティ ポリシーで、クラウドへのパスワード ハッシュの同期が禁止されている可能性があります。 この場合は、お客様の組織が[パススルー認証](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication)を検討する必要があります。
* ユーザーが、企業ネットワーク上のドメイン参加マシンからクラウドのリソースに、シームレスなシングル サインオン (SSO) でアクセスできるようにすることが必要な場合があります。
* 組織によっては、Active Directory フェデレーション サービス (AD FS) またはサード パーティのフェデレーション プロバイダーが既にデプロイされている場合があります。 クラウドに保持されているパスワード情報ではなく、このインフラストラクチャを使って認証と SSO を実装するように、Azure AD を構成できます。

詳しくは、「[Azure AD Connect ユーザーのサインイン オプション][aad-user-sign-in]」をご覧ください。

### <a name="azure-ad-application-proxy"></a>Azure AD アプリケーション プロキシ 

Azure AD を使って、オンプレミスのアプリケーションにアクセスできるようにします。

Azure AD アプリケーション プロキシ コンポーネントによって管理されるアプリケーション プロキシ コネクタを使って、オンプレミスの Web アプリケーションを公開します。 アプリケーション プロキシ コネクタは Azure AD アプリケーション プロキシへの送信ネットワーク接続を開き、リモート ユーザーの要求は、この接続を介して Azure AD から Web アプリに戻るようにルーティングされます。 これにより、オンプレミスのファイアウォールで受信ポートを開く必要がなくなり、組織が攻撃にさらされている範囲が削減されます。

詳しくは、「[Azure AD アプリケーション プロキシを使用してアプリケーションを発行する][aad-application-proxy]」をご覧ください。

### <a name="object-synchronization"></a>オブジェクトの同期 

Azure AD Connect の既定の構成は、「[Azure AD Connect Sync: 既定の構成について][aad-connect-sync-default-rules]」で指定されている規則に基づいて、お使いのローカルな Active Directory ディレクトリからオブジェクトを同期します。 これらの規則を満たすオブジェクトは同期され、他のすべてのオブジェクトは無視されます。 規則の例を次に示します。

* ユーザー オブジェクトは、一意の *sourceAnchor* 属性を持ち、*accountEnabled* 属性が設定されている必要がある。
* ユーザー オブジェクトは、*sAMAccountName* 属性を持つ必要があり、テキスト *Azure AD_* または *MSOL_* で始まっていてはならない。

Azure AD Connect は、User、Contact、Group、ForeignSecurityPrincipal、Computer の各オブジェクトに、複数の規則を適用します。 既定の規則セットを変更する必要がある場合は、Azure AD Connect でインストールされる同期規則エディターを使います。 詳しくは、「[Azure AD Connect 同期: 既定の構成について][aad-connect-sync-default-rules]」をご覧ください。

また、独自のフィルターを定義し、同期されるオブジェクトをドメインまたは OU の値で制限することができます。 または、「[Azure AD Connect Sync: フィルター処理の構成][aad-filtering]」で説明されているように、さらに複雑なカスタム フィルター処理を実装することもできます。

### <a name="monitoring"></a>監視 

正常性の監視は、オンプレミスにインストールされる次のエージェントによって実行されます。

* Azure AD Connect は、同期操作に関する情報をキャプチャするエージェントをインストールします。 正常性とパフォーマンスを監視するには、Azure Portal の [Azure AD Connect Health] ブレードを使います。 詳しくは、「[Azure AD Connect Health for Sync の使用][aad-health]」をご覧ください。
* AD DS ドメインとディレクトリの正常性を Azure から監視するには、オンプレミスのドメイン内のマシンに Azure AD Connect Health for AD DS エージェントをインストールします。 正常性の監視には、Azure Portal の [Azure Active Directory Connect Health] ブレードを使います。 詳しくは、「[AD DS での Azure AD Connect Health の使用][aad-health-adds]」をご覧ください。 
* オンプレミスで実行しているサービスの正常性を監視するには Azure AD Connect Health for AD FS エージェントをインストールし、AD FS を監視するには Azure Portal の [Azure Active Directory Connect Health] ブレードを使います。 詳しくは、「[AD FS での Azure AD Connect Health の使用][aad-health-adfs]」をご覧ください。

AD Connect Health エージェントのインストールとその要件について詳しくは、「[Azure AD Connect Health エージェントのインストール][aad-agent-installation]」をご覧ください。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

Azure AD サービスはレプリカに基づく拡張性をサポートし、書き込み操作を処理する 1 つのプライマリ レプリカと、複数の読み取り専用セカンダリ レプリカを使います。 Azure AD は、セカンダリ レプリカに対して試みられた書き込みをプライマリ レプリカに透過的にリダイレクトして、最終的な整合性を提供します。 プライマリ レプリカに加えられたすべての変更は、セカンダリ レプリカに伝達されます。 Azure AD に対するほとんどの操作は書き込みではなく読み取りなので、このアーキテクチャは優れた拡張性を備えています。 詳しくは、「[Azure AD: Under the hood of our geo-redundant, highly available, distributed cloud directory][aad-scalability]」(Azure AD: 地理冗長で高可用性の分散クラウド ディレクトリの内部) をご覧ください。

Azure AD Connect 同期サーバーの場合、ローカル ディレクトリから同期すると予想されるオブジェクトの数を決定します。 オブジェクトが 100,000 未満の場合は、Azure AD Connect に付属する既定の SQL Server Express LocalDB ソフトウェアを使うことができます。 オブジェクトの数がそれより多い場合は、運用バージョンの SQL Server をインストールし、Azure AD Connect のカスタム インストールを実行して、SQL Server の既存のインスタンスを使う必要があることを指定します。

## <a name="availability-considerations"></a>可用性に関する考慮事項

Azure AD サービスは、地理的に分散されており、自動フェールオーバーを利用して、世界中に点在する複数のデータ センターで実行されます。 データ センターが利用できなくなった場合、Azure AD は少なくとも 2 つの他のリージョンにあるデータ センターでディレクトリ データのインスタンスにアクセスできることを保証します。

> [!NOTE]
> Azure AD Basic および Premium サービスのサービス レベル アグリーメント (SLA) では、99.9% 以上の可用性が保証されます。 Azure AD の Free レベルには SLA はありません。 詳しくは、「[Azure Active Directory の SLA][sla-aad]」をご覧ください。
> 
> 

トポロジの推奨事項のセクションで説明したように、可用性を高めるには、ステージング モードで Azure AD Connect 同期サーバーの第 2 のインスタンスをプロビジョニングすることを検討してください。 

Azure AD Connect に付属する SQL Server Express LocalDB のインスタンスを使っていない場合は、SQL のクラスタリングを使って高可用性を実現することを検討します。 ミラーリングや Always On などのソリューションは、Azure AD Connect ではサポートされていません。

Azure AD Connect 同期サーバーの高可用性の実現に関するその他の考慮事項と、障害発生後の復旧方法については、「[Azure AD Connect Sync: 操作タスクおよび考慮事項 - ディザスター リカバリー][aad-sync-disaster-recovery]」をご覧ください。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

Azure AD の管理には次の 2 つの側面があります。

* クラウドでの Azure AD の管理。
* Azure AD Connect 同期サーバーの管理。

Azure AD では、クラウド内のドメインとディレクトリの管理に関して、次のオプションが用意されています。 

* **Azure Active Directory PowerShell モジュール**。 ユーザー管理、ドメイン管理、シングル サインオンの構成など、Azure AD でよく行われる管理タスクのスクリプトを作成する必要がある場合は、この[モジュール][aad-powershell]を使います。
* **Azure Portal の Azure AD 管理ブレード**。 このブレードでは、ディレクトリの対話形式の管理ビューが提供されており、Azure AD のほとんどの側面を制御および構成することができます。 

Azure AD Connect では、オンプレミスのコンピューターから Azure AD Connect 同期サービスを維持するための、次のツールがインストールされます。
  
* **Microsoft Azure Active Directory Connect コンソール**。 このツールを使うと、Azure AD Sync サーバーの構成の変更、同期方法のカスタマイズ、ステージング モードの有効化または無効化、ユーザーのサインイン モードの切り替えを行うことができます。 オンプレミスのインフラストラクチャを使って Active Directory FS サインインを有効にできることにご注意ください。
* **Synchronization Service Manager**。 同期プロセスを管理し、プロセスのどこかが失敗したかどうかを検出するには、このツールの *[Operations]\(操作\)* タブを使います。 このツールを使うと、同期を手動でトリガーできます。 *[Connectors]\(コネクタ\)* タブでは、同期エンジンがアタッチされているドメインに対する接続を制御することができます。
* **同期規則エディター**。 このツールは、オンプレミスのディレクトリと Azure AD の間でオブジェクトがコピーされるときのオブジェクトの変換方法をカスタマイズするために使います。 このツールを使うと、同期の追加属性とオブジェクトを指定した後、フィルターを実行して、同期の必要なオブジェクトと不要なオブジェクトを決定できます。 詳しくは、「[Azure AD Connect Sync: 既定の構成について][aad-connect-sync-default-rules]」の同期規則エディターに関するセクションをご覧ください。

Azure AD Connect の管理に関する詳細とヒントについては、「[Azure AD Connect Sync: 既定の構成を変更するためのベスト プラクティス][aad-sync-best-practices]」をご覧ください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

予期しないソースからの認証要求を拒否するには、条件付きアクセス制御を使います。

- 信頼されたネットワークではなくインターネット経由などのように、ユーザーが信頼できない場所から接続しようとしている場合は、[Azure Multi-factor Authentication (MFA)][azure-multifactor-authentication] をトリガーします。

- ユーザーのデバイス プラットフォームの種類 (iOS、Android、Windows Mobile、Windows) を使って、アプリケーションや機能へのアクセス ポリシーを決定します。

- ユーザーのデバイスの有効/無効状態を記録し、この情報をアクセス ポリシーのチェックに組み込みます。 たとえば、ユーザーが電話をなくしたり盗まれたりした場合は、その電話を無効と記録し、アクセスに使われないようにする必要があります。

- グループ メンバーシップに基づいて、リソースへのユーザーのアクセスを制御します。 グループの管理を簡素化するには、[Azure AD 動的メンバーシップ ルール][aad-dynamic-membership-rules]を使います。 この機能の簡単な概要については、「[Introduction to Dynamic Memberships for Groups][aad-dynamic-memberships]」(グループの動的メンバーシップの概要) をご覧ください。

- 疑わしいサインイン アクティビティや他のイベントに基づいて高度な保護を提供するには、Azure AD Identity Protection の条件付きアクセス リスク ポリシーを使います。

詳しくは、「[Azure Active Directory の条件付きアクセス][aad-conditional-access]」をご覧ください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

以上の推奨事項と考慮事項を実装する参照アーキテクチャのデプロイは、GitHub で入手できます。 この参照アーキテクチャでは、シミュレートされたオンプレミスのネットワークが Azure にデプロイされるので、それを使ってテストや実験を行うことができます。 参照アーキテクチャは、次の手順に従って、Windows または Linux VM にデプロイできます。 

1. 下のボタンをクリックしてください。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。 
   * **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-aad-onpremise-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * **[OS の種類]** ボックスの一覧で、**[Windows]** または **[Linux]** を選びます。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンをクリックします。
3. デプロイが完了するまで待ちます。
4. パラメーター ファイルには、ハードコーディングされた管理者のユーザー名とパスワードが含まれているため、すべての VM 上でこの両方をすぐに変更することを強くお勧めします。 Azure Portal で各 VM をクリックし、**[サポート + トラブルシューティング]** ブレードで **[パスワードのリセット]** をクリックします。 **[モード]** ボックスの一覧の **[パスワードのリセット]** を選択し、新しい**ユーザー名**と**パスワード**を選択します。 **[更新]** をクリックして、新しいユーザー名とパスワードを保持します。

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Azure Active Directory を使用するクラウド ID アーキテクチャ"
