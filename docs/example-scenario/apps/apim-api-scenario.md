---
title: Web アプリの API ベースのアーキテクチャへの移行
titleSuffix: Azure Example Scenarios
description: Azure API Management を使用して、従来の Web アプリケーションを最新式にしています。
author: begim
ms.date: 09/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
ms.openlocfilehash: c7ddc8e8b5e41768745cf08aede27dc21e2b1a2c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485861"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Azure 上の API ベースのアーキテクチャへの、従来の Web アプリケーションの移行

旅行業界の e コマース企業は、従来のブラウザーベースのソフトウェア スタックを最新式にしています。 その既存のスタックはほとんどモノリシックですが、最近のプロジェクトには [SOAP ベースの HTTP サービス][soap]もいくつかあります。 開発された内部の知的財産の一部を収益化するために、追加の収益源を作り出すことが検討されています。

このプロジェクトの目標は、技術的な負債に対処し、継続的なメンテナンスを改善し、回帰バグの少ない機能開発を加速することです。 このプロジェクトでは、反復プロセスを使用してリスクを回避し、いくつかの手順を並行して実行します。

- 開発チームは、アプリケーション バック エンドを最新式にしています。これは、VM 上でホストされるリレーショナル データベースから構成されます。
- 社内開発チームは、新しい HTTP API を介して公開される新しいビジネス機能を作成します。
- 契約開発チームは、Azure でホストされる新しいブラウザーベースの UI を構築します。

新しいアプリケーション機能が段階的に提供されます。 これらの機能により、現在の e コマース ビジネスを強化する既存のブラウザーベースのクライアント/サーバー UI 機能 (オンプレミスでホストされている機能) が段階的に置き換えられます。

管理チームは、不要な最新化を望んでいません。 また、範囲とコストの管理を維持することを望んでいます。 この目的のために、既存の SOAP HTTP サービスを保持することに決めました。 また、既存の UI の変更を最小限に抑えるつもりです。 [Azure API 管理 (APIM)][apim] を利用すると、多くのプロジェクトの要件と制約に対処できます。

## <a name="architecture"></a>アーキテクチャ

![アーキテクチャ ダイアグラム][architecture]

新しい UI は、Azure 上のサービスとしてのプラットフォーム (PaaS) としてホストされ、既存と新規の HTTP API の両方に応じて変わります。 これらの API は、パフォーマンスの向上、統合の簡易化、将来的な拡張性を可能にする、より望ましい設計のインターフェイスと共にリリースされます。

### <a name="components-and-security"></a>コンポーネントとセキュリティ

1. 既存のオンプレミス Web アプリケーションは、引き続き既存のオンプレミス Web サービスを直接消費します。
2. 既存の Web アプリから既存の HTTP サービスへの呼び出しは変わりません。 このような呼び出しは、企業ネットワークの内部で行われます。
3. 受信呼び出しは、Azure から既存の内部サービスに対して行われます。
    - セキュリティ チームは、[セキュア トランスポート (HTTPS/SSL) を使用して][apim-ssl]、APIM インスタンスからのトラフィックを企業ファイアウォールを介して既存のオンプレミス サービスに渡すことができます。
    - 運用チームは、APIM インスタンスからサービスへの受信呼び出しのみを許可します。 この要件は、企業ネットワーク境界内の [APIM インスタンスの IP アドレスをホワイトリストに登録する][apim-whitelist-ip]ことで満たされます。
    - 新しいモジュールは、(外部から発信された接続の場合に**のみ**動作する) オンプレミスの HTTP サービス要求パイプラインに構成されます。これにより、[APIM が提供する証明書][apim-mutualcert-auth]が検証されます。
4. 新しい API:
    - APIM インスタンスを介してのみ公開され、API ファサードを提供します。 新しい API には直接アクセスしません。
    - [Azure PaaS Web API アプリ][azure-api-apps]として開発、公開されます。
    - [APIM VIP][apim-faq-vip] のみを受け入れるように、([Web アプリ設定][azure-appservice-ip-restrict]を介して) ホワイトリストに登録されます。
    - セキュア トランスポート/SSL を有効にして、Azure Web Apps でホストされます。
    - 承認が有効にされ、Azure Active Directory と OAuth2 を使用して、[Azure App Service に提供され][azure-appservice-auth]ています。
5. 新しいブラウザーベースの Web アプリケーションは、既存の HTTP API と新しい API の**両方**のために Azure API Management インスタンスに依存します。

APIM インスタンスは、レガシ HTTP サービスを新しい API コントラクトにマップするように構成されます。 これにより、新しい Web UI は、一連のレガシ サービス/API と新しい API との統合を認識していません。 今後、プロジェクト チームは段階的に機能を新しい API に移植し、元のサービスを廃止します。 このような変更は、APIM の構成内で処理され、フロントエンド UI は影響を受けないため、再開発作業を回避できます。

### <a name="alternatives"></a>代替手段

- 組織が、レガシ アプリケーションをホストしている VM を含め、インフラストラクチャを完全に Azure に移行する予定だった場合でも、APIM は、アドレス指定可能な HTTP エンドポイントのファサードとして機能することができるため、優れた選択肢になります。
- お客様が既存のエンドポイントを非公開なままにして公開しないと決めた場合、API Management インスタンスを [Azure Virtual Network (VNet)][azure-vnet] にリンクできます。
  - デプロイされた Azure Virtual Network にリンクされた [Azure リフトおよびシフト シナリオ][azure-vm-lift-shift]では、お客様はプライベート IP アドレスを介してバックエンド サービスに直接アドレス指定することができました。
  - オンプレミスのシナリオで、API Management インスタンスは、[Azure VPN ゲートウェイとサイト間 IPSec VPN 接続][azure-vpn]または [ExpressRoute][azure-er] を介して、非公開で内部サービスに到達し、これを[ハイブリッドの Azure およびオンプレミス シナリオ][azure-hybrid]にすることができます。
- 内部モードで API Management インスタンスを展開することで、API Management インスタンスを非公開のままにすることができます。 この展開を [Azure Application Gateway][azure-appgw] と共に使用することで、いくつかの API のパブリック アクセスを可能にしながら、その他を内部のままにすることができます。 詳細については、[内部モードの APIM を VNET に接続する方法][apim-vnet-internal]に関するページを参照してください。

> [!NOTE]
> API Management を VNET に接続するための一般的な情報については、[こちらを参照してください][apim-vnet]。

### <a name="availability-and-scalability"></a>可用性とスケーラビリティ

- Azure API Management は、価格レベルを選択し、ユニットを追加することで[スケールアウト][apim-scaleout]することができます。
- スケーリングは[自動スケーリング][apim-autoscale]で自動的に行われます。
- [複数リージョンにデプロイ][apim-multi-regions]すると、フェールオーバー オプションが有効になります。これは [Premium レベル][apim-pricing]で実行できます。
- 監視のために [Azure Monitor][azure-mon] を介してメトリックに接続することもできる [Azure Application Insights との統合][azure-apim-ai]を検討してみてください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

まず、[ポータルで Azure API Management インスタンスを作成します。][apim-create]

または、特定のユース ケースに合わせて既存の Azure Resource Manager [クイックスタート テンプレート][azure-quickstart-templates-apim]から選択することもできます。

## <a name="pricing"></a>価格

API Management は、Developer、Basic、Standard、Premium の 4 つのレベルで提供されています。 これらのレベルの違いの詳細については、[こちらの Azure API Management の価格ガイダンスを参照してください。][apim-pricing]

お客様はユニットの追加や削除を行うことにより、API Management をスケーリングできます。 各ユニットのキャパシティは、そのレベルによって異なります。

> [!NOTE]
> Developer レベルは、API Management 機能の評価に使用できます。 Developer レベルは運用に使用しないでください。

予想されるコストを表示し、デプロイのニーズに合わせてカスタマイズするには、[Azure 料金計算ツール][pricing-calculator]でスケール ユニットと App Service インスタンスの数を変更します。

## <a name="related-resources"></a>関連リソース

さまざまな Azure API Management の[ドキュメントとリファレンスの記事][apim]を確認してください。

<!-- links -->

[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
