---
title: "Active Directory Domain Services (AD DS) を Azure に拡張する"
description: "Active Directory の承認を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する方法。\nガイダンス,vpn gateway,expressroute,ロード バランサー,仮想ネットワーク,active directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: 216c59a0a5912d0fe90011e49ad20eb017ada6be
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2017
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a>Active Directory Domain Services (AD DS) を Azure に拡張する

この参照アーキテクチャでは、Active Directory 環境を Azure に拡張し、[Active Directory Domain Services (AD DS)][active-directory-domain-services] を使用して分散認証サービスを提供する方法を示します。  [**こちらのソリューションをデプロイしてください**。](#deploy-the-solution)

[![0]][0] 

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

AD DS は、ユーザー、コンピューター、アプリケーション、またはセキュリティ ドメインに含まれるその他の ID の認証に使用します。 オンプレミスでホストできますが、アプリケーションがオンプレミスと Azure で部分的にホストされる場合は、Azure でこの機能をレプリケートする方が効率的です。 これにより、クラウドからオンプレミスで実行されている AD DS に返される認証要求とローカルの承認要求の送信が原因の待機時間を削減できます。 

このアーキテクチャは、オンプレミス ネットワークと Azure 仮想ネットワークが VPN または ExpressRoute によって接続されている場合によく使用されます。 また、このアーキテクチャは、双方向レプリケーションをサポートします。つまり、オンプレミスまたはクラウドで変更を行うことができ、両方のソースの一貫性が確保されます。 このアーキテクチャの一般的な用途には、オンプレミスと Azure 間で機能が配布されるハイブリッド アプリケーション、および Active Directory を使用して認証を実行するアプリケーションとサービスがあります。

その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。 

## <a name="architecture"></a>アーキテクチャ 

このアーキテクチャは、「[DMZ between Azure and the Internet][implementing-a-secure-hybrid-network-architecture-with-internet-access]」(Azure とインターネット間の DMZ) に示すアーキテクチャを拡張します。 アーキテクチャに含まれるコンポーネントを次に示します。

* **オンプレミス ネットワーク**。 オンプレミス ネットワークには、オンプレミスにあるコンポーネントの認証と承認を実行できるローカルの Active Directory サーバーが含まれています。
* **Active Directory サーバー**。 クラウドで VM として実行されているディレクトリ サービス (AD DS) を実装するドメイン コントローラーです。 このようなサーバーは、Azure 仮想ネットワークで実行されるコンポーネントの認証を提供できます。
* **Active Directory サブネット**。 AD DS サーバーは、個別のサブネットでホストされます。 ネットワーク セキュリティ グループ (NSG) ルールによって AD DS サーバーが保護され、予期しないソースからのトラフィックに対するファイアウォールが提供されます。
* **Azure ゲートウェイと Active Directory 同期**。 Azure ゲートウェイによって、オンプレミス ネットワークと Azure VNet の間に接続が提供されます。 [VPN 接続][azure-vpn-gateway]または [Azure ExpressRoute][azure-expressroute] を使用できます。 クラウドおよびオンプレミスの Active Directory サーバー間のすべての同期要求はゲートウェイを経由します。 Azure に渡すオンプレミスのトラフィックのルーティングは、ユーザー定義ルート (UDR) によって処理されます。 Active Directory サーバーとの間のトラフィックは、このシナリオで使用するネットワーク仮想アプライアンス (NVA) を経由しません。

UDR と NVA の設定について詳しくは、「[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]」(セキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する) をご覧ください。 

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、優先される特定の要件がない限り、従ってください。 

### <a name="vm-recommendations"></a>VM の推奨事項

認証要求に必要なボリュームに基づいて、[VM サイズ][vm-windows-sizes]の要件を決定します。 AD DS をオンプレミスでホストしているコンピューターの仕様を始点として使用し、それに合わせて Azure VM サイズを指定します。 デプロイ後は、使用率を監視し、VM における実際の負荷に基づいてスケールアップまたはスケールダウンします。 AD DS ドメイン コントローラーのサイジングついて詳しくは、「[Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds]」(Active Directory Domain Services のキャパシティ プランニング) をご覧ください。

Active Directory 用の データベース、ログ、および SYSVOL を格納するための個別の仮想データ ディスクを作成します。 オペレーティング システムと同じディスクにこれらの項目を格納しないでください。 既定では、VM に接続されたデータ ディスクにはライト スルー キャッシュが使用されています。 ただし、このキャッシュ形式は AD DS の要件と矛盾する可能性があります。 そのため、データ ディスクの *[ホスト キャッシュ設定]* を *[なし]* に設定します。 詳しくは、「[Windows Server AD DS データベースと SYSVOL の配置][adds-data-disks]」をご覧ください。

ドメイン コントローラーとして、AD DS を実行する VM を少なくとも 2 つデプロイし、[可用性セット][availability-set]に追加します。

### <a name="networking-recommendations"></a>ネットワークの推奨事項

ドメイン ネーム サービス (DNS) の完全なサポートを実現するには、静的なプライベート IP アドレスを使用して各 AD DS サーバの VM ネットワーク インターフェイス (NIC) を構成します。 詳しくは、「[Azure Portal を使用して仮想マシンのプライベート IP アドレスを構成する][set-a-static-ip-address]」をご覧ください。

> [!NOTE]
> パブリック IP アドレスを使用して AD DS の VM NIC を構成しないでください。 詳しくは、「[セキュリティに関する考慮事項][security-considerations]」をご覧ください。
> 
> 

Active Directory サブネット NSG には、オンプレミスからの受信トラフィックを許可するルールが必要です。 AD DS で使用されるポートについて詳しくは、「[Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]」(Active Directory と Active Directory Domain Services のポート要件) をご覧ください。 また、UDR テーブルは、このアーキテクチャで使用される NVA を使用して AD DS トラフィックをルーティングしません。 

### <a name="active-directory-site"></a>Active Directory サイト

AD DS では、サイトは物理的な場所、ネットワーク、またはデバイスの集合を表します。 AD DS サイトは、互いに近くにあり、高速ネットワークによって接続される AD DS オブジェクトをグループ化することによって AD DS データベース レプリケーションを管理するために使用します。 AD DS には、サイト間で AD DS データベースをレプリケートするための最適な戦略を選択するロジックが含まれます。

アプリケーション用に定義されたサブネットを含む AD DS サイトを Azure で作成することをお勧めします。 続いて、オンプレミスの AD DS サイト間のサイト リンクを構成します。AD DS は最も効率的なデータベース レプリケーションを自動的に実行します。 このデータベース レプリケーションには、初期構成以外の構成が多少必要です。

### <a name="active-directory-operations-masters"></a>Active Directory 操作マスター

操作マスターの役割は、レプリケートされた AD DS データベースのインスタンス間の整合性チェックをサポートする AD DS ドメイン コントローラーに割り当てることができます。 操作マスターの役割は 5 つあります。スキーマ マスター、ドメイン名前付けマスター、相対識別子マスター、プライマリ ドメイン コントローラー マスター エミュレーター、インフラストラクチャ マスターです。 これらの役割について詳しくは、「[What are Operations Masters?][ad-ds-operations-masters]」(操作マスターとは) をご覧ください。

Azure にデプロイされたドメイン コントローラーには操作マスターの役割を割り当てないことを推奨します。

### <a name="monitoring"></a>監視

ドメイン コントローラー VM のリソースおよび AD DS を監視し、問題を迅速に修正するためのプランを作成します。 詳しくは、「[Monitoring Active Directory][monitoring_ad]」(Active Directory の監視) をご覧ください。 [Microsoft Systems Center][microsoft_systems_center] などのツールを監視サーバーにインストールして (アーキテクチャの図を参照)、これらのタスクを実行することもできます。  

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

AD DS は、拡張性を確保するために設計されています。 要求を AD DS ドメイン コントローラーに送信するようにロード バランサーやトラフィック コントローラーを構成する必要はありません。 拡張性に関する唯一の考慮事項は、AD DS を実行する、ネットワーク負荷の要件に対応したサイズの VM を構成し、VM 上の負荷を監視し、必要に応じてスケールアップまたはスケールダウンすることです。

## <a name="availability-considerations"></a>可用性に関する考慮事項

AD DS を実行する VM をデプロイして[可用性セット][availability-set]を作成します。 また、少なくとも 1 つのサーバー (要件に応じて、可能であればそれ以上の数のサーバー) に[スタンバイ操作マスター][standby-operations-masters]の役割を割り当てることを検討します。 スタンバイ操作マスターは、フェールオーバー時にプライマリ操作マスター サーバーの代わりに使用可能な操作マスターのアクティブ コピーです。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

AD DS の定期的なバックアップを実行します。 定期的なバックアップの代わりに、ドメイン コントローラーの VHD ファイルを単にコピーしないでください。これは、VHD 上の AD DS データベース ファイルをコピーすると、ファイルの状態に不整合が生じる可能性があり、データベースを再起動できなくなるためです。

Azure Portal を使用してドメイン コントローラー VM をシャットダウンしないでください。 代わりに、ゲスト オペレーティング システムをシャットダウンして再起動します。 Portal を使用してシャットダウンすると、VM の割り当てが解除され、Active Directory リポジトリの `VM-GenerationID` と `invocationID` の両方がリセットされます。 これにより、AD DS 相対識別子 (RID) プールが破棄され、SYSVOL が権限なしとしてマークされます。また、ドメイン コントローラーの再構成が必要になる場合があります。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

AD DS サーバーは認証サービスを提供するため、攻撃の最適なターゲットとなります。 サーバーを保護するには、ファイアウォールとして機能する NSG を使用する個別のサブネットに AD DS サーバーを配置して、直接インターネットに接続しないようにします。 認証、承認、サーバーの同期に必要なポートを除く、AD DS サーバー上のポートをすべて閉じてください。 詳しくは、「[Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]」(Active Directory と Active Directory Domain Services のポート要件) をご覧ください。

「[Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]」(インターネットへのアクセスが可能なセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する) に示すように、サブネットと NVA のペアを使用して、サーバーの周囲に追加のセキュリティ境界を実装することを検討してください。

BitLocker または Azure Disk Encryption を使用して、AD DS データベースをホストするディスクを暗号化します。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

この参照用アーキテクチャをデプロイするためのソリューションは、[GitHub][github] で入手できます。 このソリューションをデプロイする Powershell スクリプトを実行するには、最新バージョンの [Azure CLI][azure-powershell] が必要です。 参照アーキテクチャをデプロイするには、次の手順を実行します。

1. [GitHub][github] からローカル マシンにソリューション フォルダーをダウンロードまたは複製します。

2. Azure CLI を開き、ローカルのソリューション フォルダーに移動します。

3. 次のコマンドを実行します。
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
    `<subscription id>` は、Azure サブスクリプション ID に置き換えてください。
    `<location>` には、Azure リージョン (`eastus` や `westus` など) を指定します。
    `<mode>` パラメーターは、デプロイの細分性を制御します。このパラメーターの値は次のいずれかになります。
    * `Onpremise`: シミュレートされたオンプレミスの環境をデプロイします。
    * `Infrastructure`: Azure に VNet インフラストラクチャとジャンプ ボックスをデプロイします。
    * `CreateVpn`: Azure 仮想ネットワーク ゲートウェイをデプロイして、シミュレートされたオンプレミス ネットワークに接続します。
    * `AzureADDS`: Azure で AD DS サーバーとして機能する VM をデプロイし、Active Directory をそれらの VM にデプロイして、ドメインをデプロイします。
    * `Workload`: パブリック DMZ とプライベート DMZ およびワークロードの階層をデプロイします。
    * `All`: 上記のすべてをデプロイします。 **これは、既存のオンプレミス ネットワークがない状況で、テストまたは評価用に前述の完全な参照アーキテクチャをデプロイする場合に推奨されるオプションです。**

4. デプロイが完了するまで待ちます。 `All` のデプロイを指定した場合は、数時間かかります。

## <a name="next-steps"></a>次の手順

* Azure で [AD DS リソース フォレストを作成する][adds-resource-forest]ためのベスト プラクティスを学習します。
* Azure で [Active Directory フェデレーション サービス (AD FS) インフラストラクチャを作成する][adfs]ためのベスト プラクティスを学習します。

<!-- links -->
[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: https://azure.microsoft.com/documentation/articles/virtual-networks-static-private-ip-arm-pportal/
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"
