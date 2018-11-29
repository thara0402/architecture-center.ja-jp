---
title: Azure に Active Directory フェデレーション サービス (AD FS) を実装する
description: >-
  Active Directory フェデレーション サービスの承認を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する方法。

  ガイダンス,vpn gateway,expressroute,ロード バランサー,仮想ネットワーク,active directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: adf989ec40f837ce95000e75b7bc642db446f396
ms.sourcegitcommit: 1287d635289b1c49e94f839b537b4944df85111d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/27/2018
ms.locfileid: "52332342"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Active Directory フェデレーション サービス (AD FS) を Azure に拡張する

この参照アーキテクチャは、オンプレミス ネットワークを Azure に拡張する安全なハイブリッド ネットワークを実装し、[Active Directory フェデレーション サービス (AD FS)][active-directory-federation-services] を使用して、Azure で実行されているコンポーネントのフェデレーション認証と承認を実行します。 [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

[![0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

AD FS はオンプレミスでホストできますが、アプリケーションがハイブリッドであって、その一部が Azure で実装されている場合は、クラウドに AD FS をレプリケートする方が効率的です。 

この図は、以下のシナリオを示しています。

* 自社の Azure VNet 内でホストされている Web アプリケーションに、取引先組織のアプリケーション コードがアクセスします。
* 自社の Azure VNet 内でホストされている Web アプリケーションに、資格情報が Active Directory Domain Services (DS) 内に格納されている外部登録ユーザーがアクセスします。
* 承認済みのデバイスを使用して自社の VNet に接続しているユーザーが、自社の Azure VNet 内でホストされている Web アプリケーションを実行します。

このアーキテクチャの一般的な用途は次のとおりです。

* ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。
* フェデレーション承認を使用して Web アプリケーションを取引先組織に公開するソリューション。
* 組織のファイアウォールの外部で実行されている Web ブラウザーからのアクセスをサポートするシステム。
* リモート コンピューター、ノートブック、その他のモバイル デバイスなどの承認された外部デバイスから接続することで、ユーザーが Web アプリケーションにアクセスできるようにするシステム。 

この参照アーキテクチャでは、"*パッシブ フェデレーション*" に焦点を当てています。この場合、フェデレーション サーバーがユーザーの認証方法とタイミングを決定します。 ユーザーは、アプリケーションの起動時にサインイン情報を指定します。 このメカニズムは、Web ブラウザーで最もよく使用され、ユーザーが認証するサイトにブラウザーをリダイレクトするプロトコルが含まれています。 AD FS は、"*アクティブ フェデレーション*" もサポートしています。この場合、アプリケーションはユーザーによる操作なしで資格情報を提供する責任を負いますが、そのシナリオはこのアーキテクチャの範囲外です。

その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。 

## <a name="architecture"></a>アーキテクチャ

このアーキテクチャは、[Azure への AD DS の拡張][extending-ad-to-azure]に関するページで説明されている実装を拡張します。 これには、次のコンポーネントが含まれます。

* **AD DS サブネット**。 AD DS サーバーは、ネットワーク セキュリティ グループ (NSG) 規則がファイアウォールとして機能している独自のサブネットに含まれています。

* **AD DS サーバー**。 Azure で VM として実行されているドメイン コントローラー。 これらのサーバーは、ドメイン内のローカル ID の認証を提供します。

* **AD FS サブネット**。 AD FS サーバーは、NSG 規則がファイアウォールとして機能している独自のサブネット内に存在します。

* **AD FS サーバー**。 AD FS サーバーは、フェデレーション承認と認証を提供します。 このアーキテクチャでは、以下のタスクを実行します。
  
  * パートナー ユーザーの代わりにパートナー フェデレーション サーバーが作成した要求を含むセキュリティ トークンを受け取ります。 AD FS は、トークンが有効であることを確認してから、要求の承認のために、Azure で実行されている Web アプリケーションに要求を渡します。 
  
    Azure で実行されている Web アプリケーションは、"*証明書利用者*" です。 パートナー フェデレーション サーバーは、Web アプリケーションで認識される要求を発行する必要があります。 パートナー フェデレーション サーバーは、取引先組織内の認証済みアカウントに代わってアクセス要求を送信するため、"*アカウント パートナー*" と呼ばれます。 AD FS サーバーは、リソース (Web アプリケーション) へのアクセスを提供するため、"*リソース パートナー*" と呼ばれます。

  * Web アプリケーションへのアクセスを必要とする Web ブラウザーまたはデバイスを実行している外部ユーザーから受信した要求の認証と承認を、AD DS と [Active Directory Device Registration Service][ADDRS] を使用して行います。
    
  AD FS サーバーは、Azure ロード バランサーを通じてアクセスされるファームとして構成されています。 この実装により、可用性とスケーラビリティが向上します。 AD FS サーバーは、インターネットに直接公開されません。 すべてのインターネット トラフィックは、AD FS Web アプリケーション プロキシ サーバーと DMZ (境界ネットワークとも呼ばれます) を介してフィルター処理されます。

  AD FS のしくみの詳細については、[Active Directory フェデレーション サービスの概要][active-directory-federation-services-overview]に関するページを参照してください。 また、[Azure での AD FS デプロイ][adfs-intro]に関する記事には、実装の詳細な手順が記載されています。

* **AD FS プロキシ サブネット**。 AD FS プロキシ サーバーは、独自のサブネット内に含めることができ、NSG 規則によって保護が提供されます。 このサブネット内のサーバーは、Azure 仮想ネットワークとインターネットの間にファイアウォールを提供する一連のネットワーク仮想アプライアンスを通じて、インターネットに公開されます。

* **AD FS Web アプリケーション プロキシ (WAP) サーバー**。 これらの VM は、取引先組織と外部デバイスからの受信要求に対する AD FS サーバーとして機能します。 WAP サーバーはフィルターとして機能し、インターネットからの直接アクセスから AD FS サーバーを保護します。 AD FS サーバーと同様に、負荷分散されているファームに WAP サーバーをデプロイすると、一連のスタンドアロン サーバーをデプロイする場合よりも可用性とスケーラビリティが向上します。
  
  > [!NOTE]
  > WAP サーバーのインストールの詳細については、「[Web アプリケーション プロキシ サーバーをインストールし、構成する][install_and_configure_the_web_application_proxy_server]」を参照してください。
  > 
  > 

* **取引先組織**。 Azure で実行されている Web アプリケーションへのアクセスを要求する、Web アプリケーションを実行している取引先組織。 取引先組織のフェデレーション サーバーは、ローカルで要求を認証し、要求が含まれているセキュリティ トークンを Azure で実行されている AD FS に送信します。 Azure の AD FS はセキュリティ トークンを検証し、有効であれば、Azure で実行されている Web アプリケーションに要求を渡して承認することができます。
  
  > [!NOTE]
  > 信頼できるパートナーに AD FS への直接アクセスを提供するように、Azure ゲートウェイを使用して VPN トンネルを構成することもできます。 これらのパートナーから受信した要求は、WAP サーバーをパススルーしません。
  > 
  > 

アーキテクチャの AD FS に関連していない部分の詳細については、以下を参照してください。
- [Azure に安全なハイブリッド ネットワーク アーキテクチャを実装する][implementing-a-secure-hybrid-network-architecture]
- [インターネットへのアクセスを備えた安全なハイブリッド ネットワーク アーキテクチャを Azure に実装する][implementing-a-secure-hybrid-network-architecture-with-internet-access]
- [Active Directory ID を使用する安全なハイブリッド ネットワーク アーキテクチャを Azure に実装する][extending-ad-to-azure]


## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。 

### <a name="vm-recommendations"></a>VM の推奨事項

予想されるトラフィック量を処理するための十分なリソースを持つ VM を作成します。 まずは、オンプレミスで AD FS をホストしている既存のマシンのサイズを使用します。 リソースの使用率を監視します。 VM が大きすぎる場合は、VM のサイズを変更してスケールダウンすることができます。

[Azure での Windows VM の実行][vm-recommendations]に関するページに記載されている推奨事項に従ってください。

### <a name="networking-recommendations"></a>ネットワークの推奨事項

AD FS サーバーおよび WAP サーバーをホストしている各 VM のネットワーク インターフェイスを、静的プライベート IP アドレスを使用して構成します。

AD FS VM にパブリック IP アドレスを指定しないでください。 詳細については、「セキュリティに関する考慮事項」セクションを参照してください。

各 AD FS VM および WAP VM のネットワーク インターフェイスが Active Directory DS VM を参照するように、優先およびセカンダリ ドメイン ネーム サービス (DNS) サーバーの IP アドレスを設定します。 Active Directory DS VMS は、DNS を実行している必要があります。 この手順は、各 VM がドメインに参加できるようにするために必要です。

### <a name="ad-fs-availability"></a>AD FS の可用性 

サービスの可用性を向上させるには、少なくとも 2 台のサーバーを含む AD FS ファームを作成します。 ファーム内の AD FS VM ごとに異なるストレージ アカウントを使用します。 この手法により、1 つのストレージ アカウントのエラーによってファーム全体にアクセスできなくなることを回避します。

> [!IMPORTANT]
> [マネージド ディスク](/azure/storage/storage-managed-disks-overview)を使用することをお勧めします。 マネージド ディスクでは、ストレージ アカウントは必要ありません。 ディスクのサイズと種類を指定するだけで、可用性の高い方法でデプロイされます。 Microsoft の[参照用アーキテクチャ](/azure/architecture/reference-architectures/)では、現在、マネージド ディスクがデプロイされていませんが、[テンプレートの構成要素](https://github.com/mspnp/template-building-blocks/wiki)は、バージョン 2 でマネージド ディスクをデプロイするように更新される予定です。

AD FS VM と WAP VM に個別の Azure 可用性セットを作成します。 各セットに少なくとも 2 つの VM があることを確認します。 各可用性セットには、少なくとも 2 つの更新ドメインと 2 つの障害ドメインが必要です。

AD FS VM および WAP VM のロード バランサーを次のように構成します。

* Azure ロード バランサーを使用して WAP VM への外部アクセスを提供し、内部ロード バランサーを使用してファーム内の AD FS サーバー間で負荷を分散します。
* ポート 443 (HTTPS) で生じるトラフィックのみを AD FS/WAP サーバーに渡します。
* ロード バランサーに静的 IP アドレスを指定します。
* `/adfs/probe` に対して HTTP を使用して、正常性プローブを作成します。 詳細については、「[Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2 (ハードウェア ロード バランサー正常性チェックと Web アプリケーション プロキシ/AD FS 2012 R2)](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/)」を参照してください。
  
  > [!NOTE]
  > AD FS サーバーは Server Name Indication (SNI) プロトコルを使用しているため、ロード バランサーから HTTPS エンドポイントを使用してプローブしようとすると失敗します。
  > 
  > 
* AD FS ロード バランサーのドメインに DNS *A* レコードを追加します。 ロード バランサーの IP アドレスを指定し、ドメイン内での名前を指定します (adfs.contoso.com など)。 これは、クライアントと WAP サーバーが AD FS サーバー ファームにアクセスする際に使用する名前です。

### <a name="ad-fs-security"></a>AD FS のセキュリティ 

AD FS サーバーがインターネットに直接公開されないようにします。 AD FS サーバーは、セキュリティ トークンを付与するための完全な権限を持つ、ドメイン参加コンピューターです。 サーバーが侵害されると、悪意のあるユーザーは、すべての Web アプリケーションのほか、AD FS によって保護されているすべてのフェデレーション サーバーに対するフル アクセス トークンを発行できます。 信頼できるパートナー サイトから接続されていない外部ユーザーからの要求をシステムで処理する必要がある場合は、WAP サーバーを使用してそれらの要求を処理します。 詳細については、「[フェデレーション サーバー プロキシを配置する場所][where-to-place-an-fs-proxy]」を参照してください。

AD FS サーバーと WAP サーバーをそれぞれ独自のファイアウォールを持つ別々のサブネットに配置します。 NSG 規則を使用してファイアウォール規則を定義できます。 より包括的な保護が必要な場合は、[インターネット アクセスを備えた安全なハイブリッド ネットワーク アーキテクチャの Azure への実装][implementing-a-secure-hybrid-network-architecture-with-internet-access]に関するドキュメントで説明されているように、サブネットとネットワーク仮想アプライアンス (NVA) のペアを使用して、サーバーの周囲に追加のセキュリティ境界を実装できます。 すべてのファイアウォールは、ポート 443 (HTTPS) のトラフィックを許可する必要があります。

AD FS サーバーおよび WAP サーバーへの直接的なサインイン アクセスを制限します。 DevOps スタッフだけが接続できるようにしてください。

WAP サーバーをドメインに参加させないでください。

### <a name="ad-fs-installation"></a>AD FS のインストール 

[フェデレーション サーバー ファームのデプロイ][Deploying_a_federation_server_farm]に関する記事には、AD FS のインストールと構成の詳細な手順が記載されています。 ファーム内の最初の AD FS サーバーを構成する前に、次のタスクを実行してください。

1. サーバー認証を行うために一般的に信頼されている証明書を取得します。 "*サブジェクト名*" には、クライアントがフェデレーション サービスへのアクセスに使用する名前を含める必要があります。 これには、*adfs.contoso.com* など、ロード バランサーに登録されている DNS 名を指定することもできます (セキュリティ上の理由から、**.contoso.com* などのワイルドカード名の使用は避けてください)。 すべての AD FS サーバー VM では同じ証明書を使用します。 信頼された証明機関から証明書を購入できますが、組織で Active Directory 証明書サービスを使用している場合は、独自の証明書を作成できます。 
   
    "*サブジェクトの別名*" は、外部デバイスからのアクセスを可能にするために、デバイス登録サービス (DRS) によって使用されます。 これは、*enterpriseregistration.contoso.com* の形式にする必要があります。
   
    詳細については、[AD FS 用の Secure Sockets Layer (SSL) 証明書の取得と構成][adfs_certificates]に関するページを参照してください。

2. ドメイン コントローラーで、キー配布サービスの新しいルート キーを生成します。 有効時間を現在の時刻から 10 時間差し引いた時間に設定します (この構成により、ドメイン全体でのキーの配布と同期で発生する可能性のある遅延が軽減されます)。 この手順は、AD FS サービスの実行に使用されるグループ サービス アカウントの作成をサポートするために必要です。 次の PowerShell コマンドは、その実行方法の例を示しています。
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. 各 AD FS サーバー VM をドメインに追加します。

> [!NOTE]
> AD FS をインストールするには、ドメインでプライマリ ドメイン コントローラー (PDC) エミュレーターの Flexible Single Master Operation (FSMO) ロールを実行しているドメイン コントローラーが実行されていて、AD FS VM からアクセス可能である必要があります。 <<RBC: この繰り返しを少なくする方法はありますか?>>
> 
> 

### <a name="ad-fs-trust"></a>AD FS の信頼 

AD FS インストールと、すべての取引先組織のフェデレーション サーバーとの間で、フェデレーションの信頼を確立します。 必要な任意の要求のフィルターおよびマッピングを構成します。 

* 各取引先組織の DevOps スタッフは、AD FS サーバーを介してアクセス可能な Web アプリケーションの証明書利用者信頼を追加する必要があります。
* 所属する組織の DevOps スタッフは、取引先組織から提供される要求を AD FS サーバーが信頼できるように、要求とプロバイダー間の信頼を構成する必要があります。
* 所属する組織の DevOps スタッフは、組織の Web アプリケーションに要求を渡すように AD FS を構成する必要もあります。
  
詳細については、「[Establishing Federation Trust (フェデレーションの信頼の確立)][establishing-federation-trust]」を参照してください。

組織の Web アプリケーションを発行し、それを外部パートナーが使用できるようにするには、WAP サーバーを介した事前認証を使用します。 詳細については、「[AD FS 事前認証を使用してアプリケーションを公開する][publish_applications_using_AD_FS_preauthentication]」を参照してください。

AD FS は、トークンの変換と拡張をサポートしています。 Azure Active Directory では、この機能を提供していません。 AD FS を使用した場合、信頼関係を設定する際に、次のことが可能です。

* 承認規則の要求変換を構成する。 たとえば、Microsoft 以外の取引先組織で使用されている表現のグループ セキュリティを、所属組織内で Active Directory DS が承認できるものにマップできます。
* 要求の形式を変換する。 たとえば、アプリケーションで SAML 1.1 要求のみがサポートされている場合は、SAML 2.0 から SAML 1.1 にマップすることができます。

### <a name="ad-fs-monitoring"></a>AD FS の監視 

[Active Directory フェデレーション サービス 2012 R2 用 Microsoft System Center 管理パック][oms-adfs-pack]を使用すると、フェデレーション サーバーの AD FS デプロイをプロアクティブにもリアクティブにも監視することができます。 この管理パックで監視するものは次のとおりです。

* AD FS サービスがイベント ログに記録するイベント。
* AD FS パフォーマンス カウンターで収集されるパフォーマンス データ。 
* AD FS システムおよび Web アプリケーション (証明書利用者) の全体的な正常性。重大な問題や警告のアラートも提供されます。 

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

「[AD FS の配置を計画する][plan-your-adfs-deployment]」の記事から要約された以下の考慮事項は、AD FS ファームのサイズ設定の出発点となります。

* ユーザーが 1,000 人未満の場合は、専用サーバーを作成しないでください。ただし、その代わりに、クラウド内の各 Active Directory DS サーバーに AD FS をインストールします。 可用性を維持するために、少なくとも 2 台の Active Directory DS サーバーがあることを確認してください。 1 つの WAP サーバーを作成します。
* ユーザーが 1,000 ～ 15,000 人の場合は、2 台の専用 AD FS サーバーと 2 台の専用 WAP サーバーを作成します。
* ユーザーが 15,000 ～ 60,000 人の場合は、3 ～ 5 台の専用 AD FS サーバーと、少なくとも 2 台の専用 WAP サーバーを作成します。

これらの考慮事項では、Azure でデュアル クアッドコア VM (Standard D4_v2 以上) サイズを使用していることを想定しています。

AD FS 構成データの格納に Windows Internal Database を使用している場合、ファーム内の AD FS サーバーは 8 台に制限されます。 将来的にもっと必要になることが予想される場合は、SQL Server を使用してください。 詳細については、「[AD FS 構成データベースの役割][adfs-configuration-database]」を参照してください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

AD FS の構成情報を保持するには、SQL Server または Windows Internal Database を使用できます。 Windows Internal Database は、基本的な冗長性を提供します。 変更は AD FS クラスター内の 1 つの AD FS データベースだけに直接書き込まれる一方で、他のサーバーはプル レプリケーションを使用してそれらのデータベースを最新の状態に保ちます。 SQL Server を使用すると、フェールオーバー クラスタリングまたはミラーリングを使用してデータベースの完全な冗長性と高可用性を実現できます。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

DevOps スタッフは、以下のタスクの実行を準備する必要があります。

* フェデレーション サーバーの管理 (AD FS ファームの管理、フェデレーション サーバーでの信頼ポリシーの管理、フェデレーション サービスで使用される証明書の管理など)。
* WAP サーバーの管理 (WAP ファームや証明書の管理など)。
* Web アプリケーションの管理 (証明書利用者、認証方法、要求のマッピングの構成など)。
* AD FS コンポーネントのバックアップ。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

AD FS は HTTPS プロトコルを使用するため、Web 層 VM が含まれているサブネットの NSG 規則で HTTPS 要求が許可されていることを確認してください。 これらの要求は、オンプレミス ネットワークのほか、Web 層、ビジネス層、データ層、プライベート DMZ、パブリック DMZ を含むサブネットや AD FS サーバーを含むサブネットから発信される場合があります。

監査のために、仮想ネットワークの境界を越えるトラフィックに関する詳細情報を記録する一連のネットワーク仮想アプライアンスの使用を検討してください。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照用アーキテクチャをデプロイするためのソリューションは、[GitHub][github] で入手できます。 このソリューションをデプロイする Powershell スクリプトを実行するには、最新バージョンの [Azure CLI][azure-cli] が必要です。 参照アーキテクチャをデプロイするには、次の手順を実行します。

1. [GitHub][github] からローカル マシンにソリューション フォルダーをダウンロードまたは複製します。

2. Azure CLI を開き、ローカルのソリューション フォルダーに移動します。

3. 次のコマンドを実行します。
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    `<subscription id>` は、Azure サブスクリプション ID に置き換えてください。
   
    `<location>` には、Azure リージョン (`eastus` や `westus` など) を指定します。
   
    `<mode>` パラメーターは、デプロイの細分性を制御します。このパラメーターの値は次のいずれかになります。
   
   * `Onpremise`: シミュレートされたオンプレミスの環境をデプロイします。 既存のオンプレミス ネットワークがない場合、または既存のオンプレミス ネットワークの構成を変更せずにこの参照用アーキテクチャをテストしたい場合は、このデプロイを使用するとテストおよび実験することができます。
   * `Infrastructure`: VNet インフラストラクチャとジャンプ ボックスをデプロイします。
   * `CreateVpn`: Azure 仮想ネットワーク ゲートウェイをデプロイし、それをシミュレートされたオンプレミス ネットワークに接続します。
   * `AzureADDS`: Active Directory DS サーバーとして機能する VM をデプロイし、Active Directory をそれらの VM にデプロイして、Azure にドメインを作成します。
   * `AdfsVm`: AD FS VM をデプロイし、それらを Azure のドメインに参加させます。
   * `PublicDMZ`: Azure にパブリック DMZ をデプロイします。
   * `ProxyVm`: AD FS プロキシ VM をデプロイし、それらを Azure のドメインに参加させます。
   * `Prepare`: 上記のすべてをデプロイします。 **これは、まったく新しいデプロイを構築していて、既存のオンプレミス インフラストラクチャがない場合に推奨されるオプションです。** 
   * `Workload`: オプションで、Web 層、ビジネス層、データ層の VM とサポートするネットワークをデプロイします。 `Prepare` デプロイ モードには含まれていません。
   * `PrivateDMZ`: オプションで、上でデプロイした `Workload` VM の前に、Azure のプライベート DMZ をデプロイします。 `Prepare` デプロイ モードには含まれていません。

4. デプロイが完了するまで待ちます。 `Prepare` オプションを使用した場合、デプロイは完了するまでに数時間かかり、`Preparation is completed. Please install certificate to all AD FS and proxy VMs.` というメッセージで終了します。

5. ジャンプ ボックス (*ra-adfs-security-rg* グループの *ra-adfs-mgmt-vm1*) を再起動すると、その DNS 設定を有効にすることができます。

6. [AD FS の SSL 証明書を入手][adfs_certificates]し、この証明書を AD FS VM にインストールします。 ジャンプ ボックスを使用してそれらに接続できることに注意してください。 IP アドレスは、<em>10.0.5.4</em> と <em>10.0.5.5</em> です。 既定のユーザー名は <em>contoso\testuser</em>、パスワードは <em>AweSome@PW</em> です。
   
   > [!NOTE]
   > Deploy-ReferenceArchitecture.ps1 スクリプトのこの部分のコメントは、`makecert` コマンドを使用して自己署名テスト証明書と機関を作成するための詳細な手順を示しています。 ただし、この手順は**テスト**として実行するだけにし、makecert によって生成された証明書を運用環境では使用しないでください。
   > 
   > 

7. 次の PowerShell コマンドを実行して、AD FS サーバー ファームをデプロイします。
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. ジャンプ ボックスで `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` に移動し、AD FS のインストールをテストします (証明書の警告が表示されることがありますが、このテストでは無視できます)。 Contoso Corporation サインイン ページが表示されることを確認します。 パスワード <em>AweS0me@PW</em> を使用して <em>contoso\testuser</em> としてサインインします。

9. SSL 証明書を AD FS プロキシ VM にインストールします。 IP アドレスは *10.0.6.4* と *10.0.6.5* です。

10. 次の PowerShell コマンドを実行し、最初の AD FS プロキシ サーバーをデプロイします。
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. スクリプトによって表示される指示に従って、最初のプロキシ サーバーのインストールをテストします。

12. 次の PowerShell コマンドを実行して、2 番目のプロキシ サーバーをデプロイします。
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. スクリプトによって表示される指示に従って、完全なプロキシ構成をテストします。

## <a name="next-steps"></a>次の手順

* [Azure Active Directory][aad] について学習します。
* [Azure Active Directory B2C][aadb2c] について学習します。

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"
