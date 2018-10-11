---
title: ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続
description: Azure ExpressRoute を使用して接続された Azure 仮想ネットワークとオンプレミス ネットワークにまたがる、セキュリティで保護されたサイト間ネットワーク アーキテクチャの実装方法。
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 754542b37988e0cd2694ae84eb6b03d68c147243
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819178"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>ExpressRoute を使用した Azure へのオンプレミス ネットワークの接続

この参照アーキテクチャでは、[Azure ExpressRoute][expressroute-introduction] を使用して、オンプレミス ネットワークを Azure の仮想ネットワークに接続する方法を示します。 ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。 プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。 [**以下のソリューションをデプロイします**。](#deploy-the-solution)

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されます。

* **オンプレミスの企業ネットワーク**。 組織内で運用されているプライベートのローカル エリア ネットワーク。

* **ExpressRoute 回線**。 エッジ ルーターを介してオンプレミス ネットワークを Azure につなげる、レイヤー 2 またはレイヤー 3 の回線 (接続プロバイダーから提供されます)。 この回線では、接続プロバイダーによって管理されるハードウェア インフラストラクチャが使用されます。

* **ローカルのエッジ ルーター**。 オンプレミス ネットワークを、プロバイダーによって管理されている回線に接続するルーター。 接続のプロビジョニング方法によっては、ルーターが使用するパブリック IP アドレスを指定しなければならない可能性があります。
* **Microsoft エッジ ルーター**。 アクティブ/アクティブの高可用性構成の 2 つのルーター。 このルーターにより、接続プロバイダーが、その回線をデータセンターに直接接続できます。 接続のプロビジョニング方法によっては、ルーターが使用するパブリック IP アドレスを指定しなければならない可能性があります。

* **Azure 仮想ネットワーク (VNet)**。 各 VNet は 1 つの Azure リージョンに配置され、複数のアプリケーション層をホストできます。 アプリケーション層は、各 VNet 内でサブネットを使用してセグメント化できます。

* **Azure パブリック サービス**。 ハイブリッド アプリケーションで使用できる Azure サービス。 このサービスはインターネット経由でも使用できますが、ExpressRoute 回線を使用してアクセスすると、トラフィックがインターネットを経由しないため、待ち時間が短縮され、パフォーマンスの予測可能性が向上します。 [パブリック ピアリング][expressroute-peering]を使用して接続が実行され、アドレスは、組織が所有するか、接続プロバイダーによって提供されます。

* **Office 365 サービス**。 公開されている、Microsoft 提供の Office 365 アプリケーションとサービス。 [Microsoft ピアリング][expressroute-peering]を使用して接続が実行され、アドレスは、組織が所有するか、接続プロバイダーによって提供されます。 また、Microsoft ピアリングを介して、Microsoft CRM Online に直接接続することもできます。

* **接続プロバイダー** (非表示)。 レイヤー 2 またはレイヤー 3 の接続を使用して、ご利用のデータセンターと Azure データセンターの間に接続を提供する企業。

## <a name="recommendations"></a>Recommendations

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。

### <a name="connectivity-providers"></a>接続プロバイダー

自分の場所に適した ExpressRoute 接続プロバイダーを選択します。 自分の場所で使用できる接続プロバイダーの一覧を取得するには、次の Azure PowerShell コマンドを使用します。

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute 接続プロバイダーは、ご利用のデータセンターを次の方法で Microsoft に接続します。

* **クラウド エクスチェンジにコロケーションされている**。 クラウド エクスチェンジがある施設に併置されている場合、併置プロバイダーのイーサネット エクスチェンジ経由で Azure に仮想交差接続を要請できます。 併置プロバイダーは、共有施設のインフラストラクチャと Azure の間に、レイヤー 2 交差接続と管理レイヤー 3 交差接続のいずれかを提供します。
* **ポイント ツー ポイントのイーサネット接続**。 オンプレミス データセンター/オフィスと Azure をポイント ツー ポイントのイーサネット リンクで接続できます。 ポイント ツー ポイントのイーサネットのプロバイダーは、サイトと Azure の間にレイヤー 2 接続と管理レイヤー 3 接続のいずれかを提供できます。
* **任意の環境間 (IPVPN) ネットワーク**。 ワイド エリア ネットワーク (WAN) を Azure と統合できます。 インターネット プロトコル仮想プライベート ネットワーク (IPVPN) プロバイダー (通常、マルチプロトコル ラベル スイッチング VPN) は、ブランチ オフィスとデータセンターの間に任意の環境間の接続を提供します。 Azure をご使用の WAN に相互接続し、ブランチ オフィスのように見せることができます。 通常、WAN プロバイダーは管理レイヤー 3 接続を提供します。

接続プロバイダーの詳細については、[ExpressRoute の概要][expressroute-introduction]に関するページをご覧ください。

### <a name="expressroute-circuit"></a>ExpressRoute 回線

Azure に接続するための [ExpressRoute 前提条件][expressroute-prereqs]を組織が満たしていることを確認してください。

`GatewaySubnet` という名前のサブネットを Azure VNet にまだ追加していない場合は追加し、Azure VPN ゲートウェイ サービスを使用して、ExpressRoute 仮想ネットワーク ゲートウェイを作成します。 このプロセスの詳細については、「[回線のプロビジョニングと回線の状態の ExpressRoute ワークフロー][ExpressRoute-provisioning]」を参照してください。

次のように、ExpressRoute 回線を作成します。

1. 次の PowerShell コマンドを実行します。
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. 新しい回線の `ServiceKey` をサービス プロバイダーに送信します。

3. プロバイダーによって回線がプロビジョニングされるのを待ちます。 回線のプロビジョニング状態を確認するには、次の PowerShell コマンドを実行します。
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    回線の準備ができたら、出力の `Service Provider` セクションの `Provisioning state` フィールドが `NotProvisioned` から `Provisioned` に変わります。

    > [!NOTE]
    > レイヤー 3 接続を使用している場合、ルーティングはプロバイダーによって構成および管理されます。 プロバイダーが適切なルートを実装できるように、必要な情報を提供します。
    > 
    > 

4. レイヤー 2 接続を使用している場合:

    1. 実装するピアリングの種類ごとに、有効なパブリック IP アドレスで構成される 2 つの /30 サブネットを予約します。 これらの /30 サブネットは、回線で使用されるルーターに IP アドレスを提供するために使用されます。 プライベート、パブリック、および Microsoft ピアリングを実装する場合は、有効なパブリック IP アドレスを持つ 6 つの /30 サブネットが必要になります。     

    2. ExpressRoute 回線用のルーティングを構成します。 構成するピアリングの種類 (プライベート、パブリック、および Microsoft) ごとに、次の PowerShell コマンドを実行します。 詳細については、[ExpressRoute 回線のルーティングの作成と変更][configure-expressroute-routing]に関するページをご覧ください。
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. パブリックおよび Microsoft ピアリングのネットワーク アドレス変換 (NAT) で使用するために、有効なパブリック IP アドレスのプールをもう 1 つ予約します。 ピアリングごとに異なるプールを指定することをお勧めします。 接続プロバイダーに対してそのプールを指定します。これにより、プロバイダーはこれらの範囲の境界ゲートウェイ プロトコル (BGP) アドバタイズを構成できます。

5. 次の PowerShell コマンドを実行して、プライベート VNet を ExpressRoute 回線にリンクします。 詳細については、[ExpressRoute 回線への仮想ネットワークのリンク][link-vnet-to-expressroute]に関するページをご覧ください。

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

すべての VNet と ExpressRoute 回線が、同じ地政学的リージョンに配置されている場合は、さまざまなリージョンにある複数の VNet を同じ ExpressRoute 回線に接続できます。

### <a name="troubleshooting"></a>トラブルシューティング 

オンプレミスまたはプライベート VNet 内で構成を変更していないにもかかわらず、以前に機能していた ExpressRoute 回線が接続できなくなった場合、接続プロバイダーに連絡し、協力して問題解決にあたらなければならない可能性があります。 次の PowerShell コマンドを実行して、ExpressRoute 回線がプロビジョニングされていることを確認します。

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

以下のように、このコマンドの出力には、`ProvisioningState`、`CircuitProvisioningState`、`ServiceProviderProvisioningState` など、回線のプロパティがいくつか示されます。

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

新しい回線を作成しようとした後に、`ProvisioningState` が `Succeeded` に設定されていない場合は、次のコマンドを使用して回線を削除し、もう一度作成してみてください。

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

プロバイダーによって既に回線がプロビジョニング済みであるのに、`ProvisioningState` が `Failed` に設定されているか、`CircuitProvisioningState` が `Enabled` でない場合は、対処方法についてプロバイダーにお問い合わせください。

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

ExpressRoute 回線が提供するのは、ネットワーク間の高帯域幅のパスです。 一般的に、高帯域幅であるほど、コストが高くなります。 

ExpressRoute には、従量制課金プランと無制限データ プランの 2 つの[価格プラン][expressroute-pricing]が用意されています。 料金は、回線の帯域幅によって異なります。 使用できる帯域幅は、プロバイダーによって異なる可能性があります。 `Get-AzureRmExpressRouteServiceProvider` コマンドレットを使用すると、リージョンで使用できるプロバイダーと、そのプロバイダーが提供する帯域幅を確認できます。
 
1 つの ExpressRoute 回線が、特定の数のピアリングと VNet リンクをサポートします。 詳細については、[ExpressRoute の制限](/azure/azure-subscription-service-limits)に関するページをご覧ください。

追加料金を支払うことで、ExpressRoute Premium アドオンでいくつかの追加機能を利用できます。

* パブリックおよびプライベート ピアリングのルート制限の増加。 
* ExpressRoute 回線あたりの VNet リンク数の増加。 
* サービスのグローバル接続。

詳細については、「[ExpressRoute の価格][expressroute-pricing]」を参照してください。 

ExpressRoute 回線は、購入した帯域幅制限の 2 倍までの一時的ネットワーク バーストを無料で許容できるように設計されています。 これは、冗長なリンクを使用することで実現します。 ただし、すべての接続プロバイダーが、この機能をサポートしているわけではありません。 この機能を使用する前に、接続プロバイダーによってこの機能が有効になっていることを確認してください。

プロバイダーによっては帯域幅を変更できる場合もありますが、必ずニーズを上回る初期帯域幅を選択し、拡大に対応できるだけの余地を確保してください。 今後、帯域幅を増やす必要が出てきたとき、選択できるオプションは 2 つあります。

- 帯域幅を増やします。 このオプションはできるだけ避けてください。一部のプロバイダーでは、帯域幅を動的に拡大できません。 それでも帯域幅を拡大する必要がある場合は、プロバイダーに連絡して、Powershell コマンドによる ExpressRoute 帯域幅プロパティの変更をサポートしていることを確認します。 対応している場合は、次のコマンドを実行します。

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    接続状態を維持したまま帯域幅を増やすことができます。 帯域幅をダウン グレードすると、接続が中断されます。これは、回線を削除し、新しい構成で再作成する必要があるためです。

- 価格プランを変更するか、Premium にアップグレードします。 これを行うには、次のコマンドを実行します。 `Sku.Tier` プロパティには `Standard` または `Premium` を指定できます。`Sku.Name` プロパティには `MeteredData` または `UnlimitedData` を指定できます。

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > `Sku.Name` プロパティが `Sku.Tier` および `Sku.Family` と一致することを確認します。 ファミリとレベルを変更しても、名前を変更しないと、接続が無効になります。
    > 
    > 

    SKU は中断なしでアップグレードできますが、無制限の価格プランを従量制課金に切り替えることはできません。 SKU をダウングレードする場合、帯域幅の消費は、標準 SKU の既定の制限内に維持する必要があります。

## <a name="availability-considerations"></a>可用性に関する考慮事項

ExpressRoute では、高可用性を実装するための、ホット スタンバイ ルーティング プロトコル (HSRP) や仮想ルーター冗長性プロトコル (VRRP) などのルーター冗長性プロトコルがサポートされていません。 代わりに、ピアリングごとに BGP セッションの冗長ペアが使用されます。 ネットワークへの高可用性接続を容易にするために、Azure では、アクティブ/アクティブ構成の 2 つのルーター (Microsoft Edge の一部) に 2 つの冗長ポートがプロビジョニングされます。

既定では、BGP セッションでは、60 秒のアイドル タイムアウト値を使用します。 セッションが 3 回 (合計 180 秒) タイムアウトすると、ルーターは利用不可とマークされ、すべてのトラフィックが残りのルーターにリダイレクトされます。 この 180 秒のタイムアウトは、クリティカルなアプリケーションにとっては長すぎる可能性があります。 その場合は、オンプレミスのルーターで、BGP のタイムアウト設定をより小さな値に変更できます。

Azure 接続の高可用性は、ご使用のプロバイダーの種類と、構成する ExpressRoute 回線および仮想ネットワーク ゲートウェイ接続の数に応じて、さまざまな方法で構成できます。 可用性オプションを次にまとめます。

* レイヤー 2 接続を使用している場合は、アクティブ/アクティブ構成でオンプレミス ネットワークに冗長ルーターを配置します。 プライマリ回線を一方のルーターに接続し、セカンダリ回線をもう一方に接続します。 これにより、接続の両端で高可用性接続が実現します。 これが必要なのは、ExpressRoute サービス レベル アグリーメント (SLA) を必要とする場合です。 詳細については、[Azure ExpressRoute の SLA][sla-for-expressroute] に関するページをご覧ください。

    次の図は、冗長オンプレミス ルーターがプライマリ回線とセカンダリ回線に接続されている構成を示しています。 各回線によってパブリック ピアリングとプライベート ピアリングのトラフィックが処理されます (前のセクションで説明したように、各ピアリングに /30 アドレス空間のペアが指定されています)。

    ![[1]][1]

* レイヤー 3 接続を使用している場合は、自動的に可用性を処理する冗長 BGP セッションが提供されていることを確認します。

* VNet を、さまざまなサービス プロバイダーによって提供される、複数の ExpressRoute 回線に接続します。 この戦略により、追加の高可用性機能と、ディザスター リカバリー機能が提供されます。

* ExpressRoute のフェールオーバー パスとしてサイト間 VPN を構成します。 このオプションの詳細については、「[VPN フェールオーバー付きの ExpressRoute を使用してオンプレミス ネットワークを Azure に接続する][highly-available-network-architecture]」を参照してください。
 このオプションは、プライベート ピアリングにのみ適用されます。 Azure と Office 365 サービスについては、インターネットが唯一のフェールオーバー パスです。 

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

[Azure Connectivity Toolkit (AzureCT)][azurect] を使用すると、オンプレミスのデータセンターと Azure の間の接続を監視できます。 

## <a name="security-considerations"></a>セキュリティに関する考慮事項

Azure 接続のセキュリティ オプションは、セキュリティ上の懸念事項とコンプライアンス ニーズに応じて、さまざまな方法で構成できます。 

ExpressRoute はレイヤー 3 で動作します。 アプリケーション レイヤーの脅威を防止するには、トラフィックを正当なリソースに制限するネットワーク セキュリティ アプライアンスを使用します。 さらに、パブリック ピアリングを使用した ExpressRoute 接続は、オンプレミスからのみ開始できます。 これにより、不正なサービスが、インターネットからオンプレミス データにアクセスして侵害するのを防ぐことができます。

セキュリティを最大限に高めるには、オンプレミス ネットワークとプロバイダー エッジ ルーターの間に、ネットワーク セキュリティ アプライアンスを追加します。 これは、VNet から未承認トラフィックが流れ込んでくるのを制限するうえで役立ちます。

![[2]][2]

監査やコンプライアンスの目的で、VNet で実行されているコンポーネントからインターネットへの直接アクセスを禁止し、[強制トンネリング][forced-tuneling]を実装しなければならない場合があります。 このような場合は、インターネット トラフィックを、オンプレミスで実行されているプロキシにリダイレクトして、監査できるようにする必要があります。 プロキシは、承認されていないトラフィックが流出するのをブロックし、悪意のある可能性がある受信トラフィックをフィルター処理するように構成できます。

![[3]][3]

セキュリティを最大限に高めるためには、VM のパブリック IP アドレスは有効にしないでください。また、NSG を使用して、こうした VM にはパブリックにアクセスできないようにします。 VM は、内部 IP アドレスでのみ使用できるようにします。 こうしたアドレスは ExpressRoute ネットワークを介してアクセス可能にできます。これにより、オンプレミスの DevOps スタッフが、構成やメンテナンスを行うことができます。

VM の管理エンドポイントを外部ネットワークに公開する必要がある場合は、NSG またはアクセス制御リストを使用して、こうしたポートの可視性を、IP アドレスまたはネットワークのホワイトリストに制限します。

> [!NOTE]
> 既定では、Azure Portal でデプロイされた Azure VM には、ログイン アクセスを提供するパブリック IP アドレスが含まれます。  
> 
> 


## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

**前提条件。** 既存のオンプレミス インフラストラクチャが、適切なネットワーク アプライアンスで既に構成されている必要があります。

ソリューションをデプロイするには、次の手順を実行します。

1. 下記のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure ポータルでリンクが開くのを待った後、次の手順に従います。
   * **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-hybrid-er-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンをクリックします。
3. デプロイが完了するまで待ちます。
4. 下記のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Azure ポータルでリンクが開くのを待った後、次の手順に従います。
   * **[リソース グループ]** セクションで **[既存のものを使用]** を選択し、テキスト ボックスに「`ra-hybrid-er-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンをクリックします。
6. デプロイが完了するまで待ちます。


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Azure ExpressRoute を使用したハイブリッド ネットワーク アーキテクチャ"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "冗長ルーターと、ExpressRoute のプライマリ回路とセカンダリ回路の使用"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "オンプレミス ネットワークへのセキュリティ デバイスの追加"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "強制トンネリングを使用したインターネットへのトラフィックの監査"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "ExpressRoute 回線の ServiceKey の検索"  