---
title: "Azure とインターネットの間の DMZ の実装"
description: "インターネットにアクセスするセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する方法。"
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 372d5bb0fc0e3c272843e062210dec5c15b2b78a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-the-internet"></a>Azure とインターネットの間の DMZ

次のリファレンス アーキテクチャは、オンプレミスのネットワークを Azure に拡張してインターネット トラフィックも受け入れる、セキュリティ保護されたハイブリッド ネットワークを示しています。 

[![0]][0] 

*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*

このリファレンス アーキテクチャは、「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているアーキテクチャを拡張したものです。 オンプレミスのネットワークからのトラフィックを処理するプライベート DMZ に加え、インターネット トラフィックを処理するパブリック DMZ が追加されています。 

このアーキテクチャの一般的な用途は次のとおりです。

* ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。
* オンプレミスとインターネットからの着信トラフィックをルーティングする Azure インフラストラクチャ。

## <a name="architecture"></a>アーキテクチャ

アーキテクチャは、次のコンポーネントで構成されています。

* **パブリック IP アドレス (PIP)**。 パブリック エンドポイントの IP アドレス。 インターネットに接続されている外部ユーザーは、このアドレスを介してシステムにアクセスできます。
* **ネットワーク仮想アプライアンス (NVA)** このアーキテクチャには、インターネットから発信されるトラフィック用の独立した NVA プールが含まれています。
* **Azure ロード バランサー**。 インターネットから着信するすべての要求は、ロード バランサーを通過してパブリック DMZ 内の NVA に分散されます。
* **パブリック DMZ のインバウンド サブネット**。 このサブネットは、Azure ロード バランサーからの要求を受け入れます。 着信要求は、パブリック DMZ 内のいずれかの NVA に渡されます。
* **パブリック DMZ のアウトバウンド サブネット**。 NVA によって承認された要求は、このサブネットを通過して、Web 層の内部ロード バランサーに送られます。

## <a name="recommendations"></a>推奨事項

ほとんどのシナリオには、次の推奨事項が適用されます。 これらの推奨事項には、優先される特定の要件がない限り、従ってください。 

### <a name="nva-recommendations"></a>NVA の推奨事項

インターネットから発信されるトラフィックとオンプレミスから発信されるトラフィックには異なる NVA セットを使用します。 両方にただ 1 つの NVA セットを使用すると、2 つのネットワーク トラフィック セットの間にセキュリティ境界が提供されないため、セキュリティ リスクが発生します。 異なる NVA を使用すると、セキュリティ規則のチェックの複雑さが軽減され、規則と着信ネットワーク要求の対応が明確になります。 片方の NVA セットはインターネット トラフィックのみの規則を実装し、他方の NVA セットはオンプレミスのトラフィックのみの規則を実装します。

第 7 層の NVA を含めて、アプリケーションの接続を NVA レベルで終了し、バックエンド層との互換性を維持します。 これにより、バックエンド層からの応答トラフィックが NVA を介して返される対称接続性が保証されます。  

### <a name="public-load-balancer-recommendations"></a>パブリック ロード バランサーの推奨事項

スケーラビリティと可用性のために、パブリック DMZ の NVA を[可用性セット][availability-set]にデプロイし、[インターネットに接続するロード バランサー][load-balancer]を使用して、インターネットからの要求を可用性セット内の NVA に分散します。  

インターネット トラフィックに必要なポートのみで要求を受け入れるようにロード バランサーを構成します。 たとえば、インバウンドの HTTP 要求をポート 80 に制限し、インバウンドの HTTPS 要求をポート 443 に制限します。

## <a name="scalability-considerations"></a>拡張性に関する考慮事項

初期のアーキテクチャでは、パブリック DMZ 内に単一の NVA が必要である場合でも、最初からパブリック DMZ の前にロード バランサーを配置しておくことをお勧めします。 これにより、将来必要になった場合に、複数の NVA を簡単にスケーリングできます。

## <a name="availability-considerations"></a>可用性に関する考慮事項

インターネットに接続するロード バランサーは、パブリック DMZ の インバウンド サブネット内の各 NVA が[正常性プローブ][lb-probe]を実装することを要求します。 エンドポイントで応答しない正常性プローブは使用不可であるとみなされ、ロード バランサーは、同じ可用性セット内の他の NVA に要求を送信します。 すべての NVA が 応答しない場合、アプリケーションは失敗するため、正常な NVA インスタンスの数が定義されたしきい値を下回った場合に DevOps アラートを生成するように構成された監視機能を用意することが重要であることに注意してください。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項

パブリック DMZ 内の NVA に対するすべての監視と管理は、管理サブネット内のジャンプボックスによって実行する必要があります。 「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているように、アクセスを制限するために、オンプレミスのネットワークからゲートウェイ経由でジャンプボックスに至る単一のネットワーク ルートを定義します。

オンプレミスのネットワークから Azure へのゲートウェイ接続がダウンした場合でも、パブリック IP アドレスをデプロイしてジャンプボックスに追加し、インターネットからログインすることで、ジャンプボックスにアクセスできます。

## <a name="security-considerations"></a>セキュリティに関する考慮事項

このリファレンス アーキテクチャは、複数のレベルのセキュリティを実装します。

* インターネットに接続するロード バランサーは、アプリケーションで必要なポートでのみ、パブリック DMZ のインバウンド サブネット内の NVA に要求を送信します。
* パブリック DMZ のインバウンド サブネットとアウトバウンド サブネットの NSG 規則は、NSG 規則に適合しない要求をブロックすることで、NVA への侵入を防ぎます。
* NVA の NAT ルーティング構成は、ポート 80 とポート 443 で着信した要求を Web 層のロード バランサーに送信しますが、それ以外のすべてのポートの要求を無視します。

すべてのポートに着信したすべての要求をログに記録する必要があります。 このログを定期的に監査して、特に予期されたパラメーターの範囲外にある要求の有無を調べます。そのような要求は、侵入の試みを示唆している可能性があります。

## <a name="solution-deployment"></a>ソリューションのデプロイ

これらの推奨事項を実装するリファレンス アーキテクチャのデプロイを [GitHub][github-folder] で入手できます。 リファレンス アーキテクチャは、次の手順に従って、Windows または Linux Vm のいずれにもデプロイできます。

1. 下のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。
   * **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-public-dmz-network-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * **[OS の種類]** ボックスの一覧の **[Windows]** または **[linux]**. を選択します。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンを選択します。
3. デプロイが完了するまで待ちます。
4. 下のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。
   * **リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-public-dmz-wl-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンを選択します。
6. デプロイが完了するまで待ちます。
7. 下のボタンをクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。
   * **リソース グループ**の名前はパラメーター ファイルに既に定義されているため、**[既存のものを使用]** を選択し、テキスト ボックスに「`ra-public-dmz-network-rg`」と入力します。
   * **[場所]** ボックスの一覧でリージョンを選択します。
   * **[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。
   * 使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。
   * **[購入]** ボタンを選択します。
9. デプロイが完了するまで待ちます。
10. パラメーター ファイルには、すべての VM のハードコーディングされた管理者のユーザー名とパスワードが含まれているため、この両方をすぐに変更することを強くお勧めします。 デプロイの各 VM で、Azure ポータルで VM を選択し、**[サポート + トラブルシューティング]** ブレードで **[パスワードのリセット]** をクリックします。 **[モード]** ボックスの一覧の **[パスワードのリセット]** を選択し、新しい**ユーザー名**と**パスワード**を選択します。 **[更新]** ボタンをクリックして保存します。


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "セキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"