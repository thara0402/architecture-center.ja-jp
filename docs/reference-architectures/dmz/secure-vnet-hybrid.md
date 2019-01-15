---
title: セキュリティ保護されたハイブリッド ネットワーク アーキテクチャの実装
titleSuffix: Azure Reference Architectures
description: セキュリティ保護されたハイブリッド ネットワーク アーキテクチャを実装します。
author: telmosampaio
ms.date: 10/22/2018
ms.custom: seodec18
ms.openlocfilehash: 9a74401d3496807ce2dfc113476e001d19e657e5
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112296"
---
# <a name="implement-a-dmz-between-azure-and-your-on-premises-datacenter"></a>Azure とオンプレミス データセンター間に DMZ を実装する

次のリファレンス アーキテクチャは、オンプレミスのネットワークを Azure に拡張する、セキュリティ保護されたハイブリッド ネットワークを示しています。 このアーキテクチャには、*境界ネットワーク*とも呼ばれる DMZ が、オンプレミス ネットワークと Azure の仮想ネットワーク (VNet) の間に実装されています。 DMZ には、ファイアウォールやパケット検査などのセキュリティ機能を実装する、ネットワーク仮想アプライアンス (NVA) が含まれています。 VNet からの発信トラフィックはすべてオンプレミス ネットワーク経由でインターネットに強制的にトンネリングされるため、トラフィックの監査が可能です。 [**このソリューションをデプロイします**](#deploy-the-solution)。

![セキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](./images/dmz-private.png)

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

このアーキテクチャでは、[VPN ゲートウェイ][ra-vpn]または [ExpressRoute][ra-expressroute] 接続のいずれかを使用して、オンプレミスのデータセンターに接続する必要があります。 このアーキテクチャの一般的な用途は次のとおりです。

- ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。
- オンプレミスのデータセンターから Azure VNet に入ってくるトラフィックをきめ細かく制御する必要があるインフラストラクチャ。
- 送信トラフィックを監査する必要があるアプリケーション。 これはしばしば、多くの商用システムで規制上の要件となっていて、プライベートな情報の一般への漏えいを防ぐ助けになります。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

- **オンプレミス ネットワーク**。 組織内に実装されているプライベートなローカル エリア ネットワーク。
- **Azure の仮想ネットワーク (VNet)**。 VNet は、Azure で実行されているアプリケーションとその他のリソースをホストします。
- **ゲートウェイ**。 ゲートウェイは、オンプレミスのネットワーク内のルーターと VNet 間に接続を提供します。
- **ネットワーク仮想アプライアンス (NVA)**。 NVA は、ファイアウォールとしてのアクセスの許可や拒否、ワイド エリア ネットワーク (WAN) 操作の最適化 (ネットワーク圧縮を含む)、カスタム ルーティング、その他のネットワーク機能などのタスクを実行する VM を表す総称です。
- **Web 層、ビジネス層、およびデータ層のサブネット**。 クラウドで実行される、例のような 3 層アプリケーションを実装した VM やサービスをホストしているサブネット。 詳細については、「[Azure で N 層アーキテクチャの Windows VM を実行する][ra-n-tier]」をご覧ください。
- **ユーザー定義ルート(UDR)**。 [ユーザー定義ルート][udr-overview]は、Azure VNet 内の IP トラフィックのフローを定義します。

    > [!NOTE]
    > VPN 接続の要件に応じて、UDR を使用してオンプレミス ネットワーク経由でトラフィックを戻すよう指示する転送ルールを実装する代わりに、ボーダー ゲートウェイ プロトコル (BGP) のルートを構成できます。
    >

- **管理サブネット**。 このサブネットには、VNet で実行されているコンポーネントの管理および監視の機能を実装している VM が含まれます。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。

### <a name="access-control-recommendations"></a>アクセスの制御に関する推奨事項

アプリケーション内のリソースは、[ロール ベースのアクセス制御][rbac] (RBAC) を使用して管理します。 次の[カスタム ロール][rbac-custom-roles]の作成を検討してください。

- アプリケーションのインフラストラクチャの管理、アプリケーション コンポーネントのデプロイ、および VM の監視と再起動を行うためのアクセス許可を持つ DevOps ロール。

- ネットワーク リソースの管理と監視を行うための一元的 IT 管理者ロール。

- NVA などのセキュリティで保護されたネットワーク リソースを管理するためのセキュリティ IT 管理者ロール。

DevOps ロールと IT 管理者ロールは NVA リソースへのアクセス権を持っているべきではありません。 このアクセス権はセキュリティ IT 管理者ロールに限定してください。

### <a name="resource-group-recommendations"></a>リソース グループの推奨事項

VM、VNet、ロード バランサーなどの Azure リソースは、リソース グループにグループ化することで容易に管理できます。 RBAC ロールを各リソース グループに割り当てて、アクセスを制限します。

次のリソース グループを作成することをお勧めします。

- オンプレミス ネットワークに接続するための VNet (VM を除く)、NSG、およびゲートウェイのリソースを含むリソース グループ。 このリソース グループに一元的 IT 管理者ロールを割り当てます。
- NVA (ロード バランサーを含む) 用の VM、jumpbox とその他の管理 VM、およびすべてのトラフィックが NVA を経由するように強制するゲートウェイ サブネット用の UDR を含むリソース グループ。 このリソース グループにセキュリティ IT 管理者ロールを割り当てます。
- ロード バランサーと VM を含む、アプリケーション層ごとの個別のリソース グループ。 このリソース グループには各層のサブネットを含めてはいけないことに注意してください。 このリソース グループに DevOps ロールを割り当てます。

### <a name="virtual-network-gateway-recommendations"></a>仮想ネットワーク ゲートウェイに関する推奨事項

オンプレミスのトラフィックは、仮想ネットワーク ゲートウェイ経由で VNet に渡します。 [Azure VPN ゲートウェイ][guidance-vpn-gateway]または [Azure ExpressRoute ゲートウェイ][guidance-expressroute]をお勧めします。

### <a name="nva-recommendations"></a>NVA の推奨事項

NVA は、ネットワーク トラフィックの管理と監視のためのさまざまなサービスを提供します。 [Azure Marketplace][azure-marketplace-nva] では、使用できる複数のサード パーティー ベンダー製 NVA が提供されています。 これらのサード パーティー製 NVA に要件を満たしているものがない場合は、VM を使用してカスタム NVA を作成できます。

たとえば、この参照アーキテクチャのソリューションのデプロイでは、VM 上に次の機能を備えた NVA を実装します。

- トラフィックは、NVA ネットワーク インターフェイス (NIC) 上で [IP 転送][ip-forwarding]を使用してルーティングされます。
- トラフィックは、通過するのが適切な場合にのみ、NVA を通過することが許可されます。 参照アーキテクチャ内の各 NVA VM は、単純な Linux ルーターです。 受信トラフィックはネットワーク インターフェイス *eth0* に到着し、送信トラフィックは、ネットワーク インターフェイス *eth1* 経由で送信されたカスタム スクリプトによって定義されるルールと照合されます。
- NVA の構成は、管理サブネットからのみ可能です。
- 管理サブネットにルーティングされるトラフィックは NVA を通過しません。 そのようにしないと、NVA が停止した場合、それらを修正するための管理サブネットへのルートがなくなります。
- NVA 用の VM は、ロード バランサーの背後の[可用性セット][availability-set]内に配置されます。 ゲートウェイ サブネット内の UDR は、NVA 要求をロード バランサーに転送します。

第 7 層の NVA を含めて、アプリケーションの接続を NVA レベルで終了し、バックエンド層とのアフィニティを維持します。 これによって対称接続性が保証され、バックエンド層からの応答トラフィックが NVA を介して返されます。

その他には、複数の NVA を連続して接続し、各 NVA で特殊化したセキュリティ タスクを実行するオプションを検討する必要があります。 これにより、NVA ごとにそれぞれのセキュリティ機能を管理できます。 たとえば、ファイアウォールを実装する NVA を、ID サービスを実行する NVA と連続的に配置できます。 管理が容易なことのトレードオフとして、待機時間を増やす可能性がある余分のネットワーク ホップが追加になるため、これがアプリケーションのパフォーマンスに影響しないことを確認してください。

### <a name="nsg-recommendations"></a>NSG の推奨事項

VPN ゲートウェイでは、オンプレミス ネットワークへの接続のパブリック IP アドレスを公開します。 受信 NVA サブネットのために、オンプレミス ネットワークが発信元ではないすべてのトラフィックをブロックするルールを使用して、ネットワーク セキュリティ グループ (NSG) を作成することをお勧めします。

正しく構成されていない NVA や無効になっている NVA をバイパスする受信トラフィックからの 2 つ目の保護レベルを提供するため、各サブネット用に NSG を作成することをお勧めします。 たとえば、参照アーキテクチャの Web 層サブネットでは、オンプレミス ネットワーク (192.168.0.0/16) または VNet から受信したもの以外のすべての要求を無視するルールと、ポート 80 で作成されたのではないすべての要求を無視する別のルールを使用して、NSG を実装しています。

### <a name="internet-access-recommendations"></a>インターネットへのアクセスに関する推奨事項

サイト間 VPN トンネルを使用して、オンプレミス ネットワークを経由するすべての送信インターネット トラフィックを[強制的にトンネリング][azure-forced-tunneling]し、ネットワーク アドレス変換 (NAT) を使用してインターネットにルーティングします。 これにより、データ層に格納されている機密情報の偶発的漏えいが防がれて、すべての発信トラフィックの検査と監査が可能になります。

> [!NOTE]
> アプリケーション層からのインターネット トラフィックを完全にブロックしないでください。そのようにすると、VM 診断ログ記録や VM 拡張機能のダウンロードを始めとする機能など、パブリック IP アドレスに依存している Azure の PaaS サービスをアプリケーション層で使用できなくなるためです。 Azure 診断でも、コンポーネントが Azure Storage アカウントに対する読み取りと書き込みが可能な必要があります。

送信インターネット トラフィックが正しく強制的にトンネリングされることを確認してください。 オンプレミス サーバー上の[ルーティングとリモート アクセス サービス][routing-and-remote-access-service]と共に VPN 接続を使用している場合は、[WireShark][wireshark] や [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226) などのツールを使用してください。

### <a name="management-subnet-recommendations"></a>管理サブネットに関する推奨事項

管理サブネットには、管理と監視の機能を実行する jumpbox が含まれています。 Jumpbox に対するセキュリティで保護された管理タスクの実行は、すべて制限してください。

jumpbox 用のパブリック IP アドレスは作成しないでください。 代わりに、受信ゲートウェイ経由で jumpbox にアクセスするルートを 1 つ作成します。 管理サブネットが、許可されているルートからの要求にのみ応答するように NSG ルールを作成してください。

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

参照アーキテクチャでは、ロード バランサーを使用してオンプレミスのネットワーク トラフィックを NVA デバイスのプールにルーティングし、それらのデバイスがトラフィックをルーティングしています。 NVA は[可用性セット][availability-set]内に配置されています。 この設計により、一定期間にわたる NVA のスループットを監視し、負荷の増加に応じて NVA デバイスを追加できます。

Standard SKU の VPN ゲートウェイでは、最大 100 Mbps の持続スループットをサポートしています。 High Performance SKU では、最大で 200 Mbps が提供されます。 帯域幅が広い場合は、ExpressRoute ゲートウェイへのアップグレードをご検討ください。 ExpressRoute では、VPN 接続よりも待機時間が短い最大 10 Gbps の帯域幅が提供されます。

Azure ゲートウェイのスケーラビリティの詳細については、「[Azure とオンプレミスの VPN を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-vpn-gateway-scalability]」と「[Azure ExpressRoute を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-expressroute-scalability]」の、スケーラビリティの考慮事項についてのセクションをご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項

前述のように、参照アーキテクチャではロード バランサーの背後にある NVA デバイスのプールを使用しています。 ロード バランサーは、正常性プローブを使用して各 NVA を監視し、応答していない NVA をプールから削除します。

Azure ExpressRoute を使用して VNet およびオンプレミス ネットワーク間の接続を提供している場合は、ExpressRoute 接続が使用できなくなったときに[フェールオーバーが行われるように VPN ゲートウェイを構成][ra-vpn-failover]します。

VPN 接続と ExpressRoute 接続の可用性を維持することの具体的説明については、「[Azure とオンプレミスの VPN を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-vpn-gateway-availability]」と「[Azure ExpressRoute を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-expressroute-availability]」の、可用性に関する考慮事項をご覧ください。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

管理サブネットでは、jumpbox によるすべてのアプリケーションとリソースの監視を実行してください。 アプリケーションの要件に応じて、管理サブネット内で追加の監視リソースが必要になる場合があります。 その場合、jumpbox を介してこれらのリソースにアクセスする必要があります。

オンプレミスのネットワークから Azure へのゲートウェイ接続がダウンした場合でも、パブリック IP アドレスをデプロイしてジャンプボックスに追加し、インターネットからリモート接続することで、ジャンプボックスにアクセスできます。

参照アーキテクチャの各層のサブネットは、NSG ルールによって保護されています。 Windows VM でリモート デスクトップ プロトコル (RDP) アクセスのためにポート 3389 を開くルールや、Linux VM で Secure Shell (SSH) アクセスのためにポート 22 を開くルールを作成することが必要な場合があります。 その他の管理ツールや監視ツールのために、追加のポートを開くルールが必要な場合があります。

ExpressRoute を使用してオンプレミスのデータセンターと Azure の間の接続を提供する場合は、[Azure Connectivity Toolkit (AzureCT)][azurect] を使用して接続の問題の監視とトラブルシューティングを行ってください。

特に VPN 接続と ExpressRoute 接続の監視と管理に的を絞ったその他の説明については、「[Azure とオンプレミスの VPN を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-vpn-gateway-manageability]」および「[Azure ExpressRoute を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-expressroute-manageability]」という記事に記載されています。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

この参照アーキテクチャには、複数レベルのセキュリティが実装されています。

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>NVA を経由するすべてのオンプレミス ユーザー要求のルーティング

ゲートウェイ サブネット内の UDR は、オンプレミスから受信した要求を除くすべてのユーザー要求をブロックします。 UDR は許可された要求をプライベート DMZ 内の NVA に渡し、これらの要求は、NVA ルールによって許可されていればアプリケーションに渡されます。 UDR にその他のルートを追加できますが、それらのルートで誤って、NVA をバイパスしたり、管理サブネットに向けた管理トラフィックをブロックしたりしないようにしてください。

NVA の前のロード バランサーも、負荷分散規則で開いていないポート上のトラフィックを無視することで、セキュリティ デバイスとして機能します。 参照アーキテクチャ内のロード バランサーは、ポート 80 での HTTP 要求とポート 443 での HTTPS 要求のみをリッスンします。 ロード バランサーに追加した規則が他にあれば文書化し、トラフィックを監視してセキュリティ上の問題がないことを確認してください。

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>NSG を使用したアプリケーション層間でのトラフィックのブロック/通過

階層間のトラフィックは NSG を使用して制限します。 ビジネス層では Web 層で発生したのではないすべてのトラフィックをブロックし、データ層ではビジネス層で発生したのではないすべてのトラフィックをブロックします。 NSG ルールを拡張し、これらの層に対してより広範囲のアクセスを許可する必要がある場合は、その必要性をセキュリティ リスクと比較検討してください。 新しい受信経路が増えるたびに、偶発的または意図的に、データ漏えいやアプリケーションの破損が起きる可能性が高まります。

### <a name="devops-access"></a>DevOps アクセス

[RBAC][rbac] を使用して、DevOps が各階層で実行できる操作を制限します。 アクセス許可を付与する場合は、[最小限の特権の原則][security-principle-of-least-privilege]に従ってください。 構成の変更がすべて計画されたものであることを確認するため、すべての管理操作をログに記録し、定期的な監査を実行します。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

これらの推奨事項を実装する参照アーキテクチャのデプロイは、[GitHub][github-folder] で入手できます。

### <a name="prerequisites"></a>前提条件

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>リソースのデプロイ

1. 参照アーキテクチャ GitHub リポジトリの `/dmz/secure-vnet-hybrid` フォルダーに移動します。

2. 次のコマンドを実行します。

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. 次のコマンドを実行します。

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>オンプレミスと Azure ゲートウェイに接続する

この手順では、2 つのローカル ネットワーク ゲートウェイを接続します。

1. Azure portal で、作成したリソース グループに移動します。

2. `ra-vpn-vgw-pip` という名前のリソースを探し、**[概要]** ブレードに表示されている IP アドレスをコピーします。

3. `onprem-vpn-lgw` という名前のリソースを探します。

4. **[構成]** ブレードをクリックします。 **[IP アドレス]** に手順 2 の IP アドレスを貼り付けます。

    ![[IP アドレス] フィールドのスクリーンショット](./images/local-net-gw.png)

5. **[保存]** をクリックし、処理が完了するまで待ちます。 完了までに約 5 分かかります。

6. `onprem-vpn-gateway1-pip` という名前のリソースを探します。 **[概要]** ブレードに表示されている IP アドレスをコピーします。

7. `ra-vpn-lgw` という名前のリソースを探します。

8. **[構成]** ブレードをクリックします。 **[IP アドレス]** に手順 6 の IP アドレスを貼り付けます。

9. **[保存]** をクリックし、処理が完了するまで待ちます。

10. 接続を確認するには、各ゲートウェイの **[接続]** ブレードに移動します。 状態が **[接続済み]** であることを確認します。

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>ネットワーク トラフィックが Web 階層に到達していることを確認する

1. Azure portal で、作成したリソース グループに移動します。

2. プライベート DMZ の前面にあるロード バランサーである `int-dmz-lb` という名前のリソースを探します。 **[概要]** ブレードからプライベート IP アドレスをコピーします。

3. `jb-vm1` という名前の VM を見つけます。 **[接続]** をクリックし、リモート デスクトップを使用して VM に接続します。 ユーザー名とパスワードは、onprem.json ファイルに指定されています。

4. リモート デスクトップ セッションから Web ブラウザーを開き、手順 2 の IP アドレスに移動します。 既定の Apache2 サーバーのホーム ページが表示されます。

## <a name="next-steps"></a>次の手順

- [Azure とインターネットの間の DMZ](./secure-vnet-dmz.md) を実装する方法について説明します。
- [高可用性ハイブリッド ネットワーク アーキテクチャ][ra-vpn-failover]を実装する方法について説明します。
- Azure でのネットワーク セキュリティの管理の詳細については、「[Microsoft クラウド サービスとネットワーク セキュリティ][cloud-services-network-security]」をご覧ください。
- Azure のリソース保護の詳細については、「[Microsoft Azure セキュリティの概要][getting-started-with-azure-security]」をご覧ください。
- Azure ゲートウェイ接続の全体にわたるセキュリティ上の問題に対処することの詳細については、「[Azure とオンプレミスの VPN を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-vpn-gateway-security]」と「[Azure ExpressRoute を使ってハイブリッド ネットワーク アーキテクチャを実装する][guidance-expressroute-security]」をご覧ください。
- [Azure でのネットワーク仮想アプライアンスに関する問題のトラブルシューティング](/azure/virtual-network/virtual-network-troubleshoot-nva)

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: /azure/best-practices-network-security
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
