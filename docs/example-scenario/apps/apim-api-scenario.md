---
title: Azure 上の API ベースのアーキテクチャへの、従来の Web アプリケーションの移行
description: Azure API Management を使用して、従来の Web アプリケーションを最新式にしています。
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: 1aa7ea6dc895146e13677dd9867fb2530f0a8f04
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876792"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a><span data-ttu-id="0a841-103">Azure 上の API ベースのアーキテクチャへの、従来の Web アプリケーションの移行</span><span class="sxs-lookup"><span data-stu-id="0a841-103">Migrating a legacy web application to an API-based architecture on Azure</span></span>

<span data-ttu-id="0a841-104">旅行業界の e コマース企業は、従来のブラウザーベースのソフトウェア スタックを最新式にしています。</span><span class="sxs-lookup"><span data-stu-id="0a841-104">An e-commerce company in the travel industry is modernizing their legacy browser-based software stack.</span></span> <span data-ttu-id="0a841-105">その既存のスタックはほとんどモノリシックですが、最近のプロジェクトには [SOAP ベースの HTTP サービス][soap]もいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="0a841-105">While their existing stack is mostly monolithic, some [SOAP-based HTTP services][soap] exist from a recent project.</span></span> <span data-ttu-id="0a841-106">開発された内部の知的財産の一部を収益化するために、追加の収益源を作り出すことが検討されています。</span><span class="sxs-lookup"><span data-stu-id="0a841-106">They are considering the creation of additional revenue streams to monetize some of the internal intellectual property that's been developed.</span></span>

<span data-ttu-id="0a841-107">このプロジェクトの目標は、技術的な負債に対処し、継続的なメンテナンスを改善し、回帰バグの少ない機能開発を加速することです。</span><span class="sxs-lookup"><span data-stu-id="0a841-107">Goals for the project include addressing technical debt, improving ongoing maintenance, and accelerating feature development with fewer regression bugs.</span></span> <span data-ttu-id="0a841-108">このプロジェクトでは、反復プロセスを使用してリスクを回避し、いくつかの手順を並行して実行します。</span><span class="sxs-lookup"><span data-stu-id="0a841-108">The project will use an iterative process to avoid risk, with some steps performed in parallel:</span></span>

* <span data-ttu-id="0a841-109">開発チームは、アプリケーション バック エンドを最新式にしています。これは、VM 上でホストされるリレーショナル データベースから構成されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-109">The development team will modernize the application back end, which is composed of relational databases hosted on VMs.</span></span>
* <span data-ttu-id="0a841-110">社内開発チームは、新しいビジネス機能を作成し、新しい HTTP API を介して公開します。</span><span class="sxs-lookup"><span data-stu-id="0a841-110">The in-house development team will write new business functionality, which will be exposed over new HTTP APIs.</span></span>
* <span data-ttu-id="0a841-111">契約開発チームは、Azure でホストされる新しいブラウザーベースの UI を構築します。</span><span class="sxs-lookup"><span data-stu-id="0a841-111">A contract development team will build a new browser-based UI, which will be hosted in Azure.</span></span>

<span data-ttu-id="0a841-112">新しいアプリケーション機能が段階的に提供されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-112">New application features will be delivered in stages.</span></span> <span data-ttu-id="0a841-113">現在の e コマース ビジネスを強化する既存のブラウザーベースのクライアント/サーバー UI 機能 (オンプレミスでホストされている機能) が*段階的に置き換え*られます。</span><span class="sxs-lookup"><span data-stu-id="0a841-113">They will *gradually replace* the existing browser-based client-server UI functionality (hosted on-premises) that powers their e-commerce business today.</span></span>

<span data-ttu-id="0a841-114">管理チームは、不要な最新化を望んでいません。</span><span class="sxs-lookup"><span data-stu-id="0a841-114">The management team does not want to modernize unnecessarily.</span></span> <span data-ttu-id="0a841-115">また、範囲とコストの管理を維持することを望んでいます。</span><span class="sxs-lookup"><span data-stu-id="0a841-115">They also want to maintain control of scope and costs.</span></span> <span data-ttu-id="0a841-116">この目的のために、既存の SOAP HTTP サービスを保持することに決めました。</span><span class="sxs-lookup"><span data-stu-id="0a841-116">To do this, they have decided to preserve their existing SOAP HTTP services.</span></span> <span data-ttu-id="0a841-117">また、既存の UI の変更を最小限に抑えるつもりです。</span><span class="sxs-lookup"><span data-stu-id="0a841-117">They also intend to minimize changes to the existing UI.</span></span> <span data-ttu-id="0a841-118">[Azure API 管理 (APIM)][apim] を利用すると、多くのプロジェクトの要件と制約に対処できます。</span><span class="sxs-lookup"><span data-stu-id="0a841-118">[Azure API Management (APIM)][apim] can be utilized to address many of the project's requirements and constraints.</span></span>

## <a name="architecture"></a><span data-ttu-id="0a841-119">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="0a841-119">Architecture</span></span>

![アーキテクチャ ダイアグラム][architecture]

<span data-ttu-id="0a841-121">新しい UI は、Azure 上のサービスとしてのプラットフォーム (PaaS) としてホストされ、既存と新規の HTTP API の両方に応じて変わります。</span><span class="sxs-lookup"><span data-stu-id="0a841-121">The new UI will be hosted as a platform as a service (PaaS) application on Azure, and will depend on both existing and new HTTP APIs.</span></span> <span data-ttu-id="0a841-122">これらの API は、パフォーマンスの向上、統合の簡易化、将来的な拡張性を可能にする、より望ましい設計のインターフェイスと共にリリースされます。</span><span class="sxs-lookup"><span data-stu-id="0a841-122">These APIs will ship with a better-designed set of interfaces enabling better performance, easier integration, and future extensibility.</span></span>

### <a name="components-and-security"></a><span data-ttu-id="0a841-123">コンポーネントとセキュリティ</span><span class="sxs-lookup"><span data-stu-id="0a841-123">Components and Security</span></span>

1. <span data-ttu-id="0a841-124">既存のオンプレミス Web アプリケーションは、引き続き既存のオンプレミス Web サービスを直接消費します。</span><span class="sxs-lookup"><span data-stu-id="0a841-124">The existing on-premises web application will continue to directly consume the existing on-premises web services.</span></span>
2. <span data-ttu-id="0a841-125">既存の Web アプリから既存の HTTP サービスへの呼び出しは変わりません。</span><span class="sxs-lookup"><span data-stu-id="0a841-125">Calls from the existing web app to the existing HTTP services will remain unchanged.</span></span> <span data-ttu-id="0a841-126">このような呼び出しは、企業ネットワークの内部で行われます。</span><span class="sxs-lookup"><span data-stu-id="0a841-126">These calls are internal to the corporate network.</span></span>
3. <span data-ttu-id="0a841-127">受信呼び出しは、Azure から既存の内部サービスに対して行われます。</span><span class="sxs-lookup"><span data-stu-id="0a841-127">Inbound calls are made from Azure to the existing internal services:</span></span>
    * <span data-ttu-id="0a841-128">セキュリティ チームは、[セキュア トランスポート (HTTPS/SSL) を使用して][apim-ssl]、APIM インスタンスからのトラフィックを企業ファイアウォールを介して既存のオンプレミス サービスに渡すことができます。</span><span class="sxs-lookup"><span data-stu-id="0a841-128">The security team allows traffic from the APIM instance to pass through the corporate firewall to the existing on-premises services [using secure transport (HTTPS/SSL)][apim-ssl].</span></span>
    * <span data-ttu-id="0a841-129">運用チームは、APIM インスタンスからサービスへの受信呼び出しのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="0a841-129">The operations team will allow inbound calls to the services only from the APIM instance.</span></span> <span data-ttu-id="0a841-130">この要件は、企業ネットワーク境界内の [APIM インスタンスの IP アドレスをホワイトリストに登録する][apim-whitelist-ip]ことで満たされます。</span><span class="sxs-lookup"><span data-stu-id="0a841-130">This requirement is met by [white-listing the IP address of the APIM instance][apim-whitelist-ip] within the corporate network perimeter.</span></span>
    * <span data-ttu-id="0a841-131">新しいモジュールは、(外部から発信された接続の場合に**のみ**動作する) オンプレミスの HTTP サービス要求パイプラインに構成されます。これにより、[APIM が提供する証明書][apim-mutualcert-auth]が検証されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-131">A new module is configured into the on-premises HTTP services request pipeline (to act upon **only** those connections originating externally), which will validate [a certificate which APIM will provide][apim-mutualcert-auth].</span></span>
1. <span data-ttu-id="0a841-132">新しい API:</span><span class="sxs-lookup"><span data-stu-id="0a841-132">The new API:</span></span>
    * <span data-ttu-id="0a841-133">APIM インスタンスを介してのみ公開され、API ファサードを提供します。</span><span class="sxs-lookup"><span data-stu-id="0a841-133">Is surfaced only through the APIM instance, which will provide the API facade.</span></span> <span data-ttu-id="0a841-134">新しい API には直接アクセスしません。</span><span class="sxs-lookup"><span data-stu-id="0a841-134">The new API won't be accessed directly.</span></span>
    * <span data-ttu-id="0a841-135">[Azure PaaS Web API アプリ][azure-api-apps]として開発、公開されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-135">Is developed and published as an [Azure PaaS Web API App][azure-api-apps].</span></span>
    * <span data-ttu-id="0a841-136">[APIM VIP][apim-faq-vip] のみを受け入れるように、([Web アプリ設定][azure-appservice-ip-restrict]を介して) ホワイトリストに登録されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-136">Is white-listed (via [Web App settings][azure-appservice-ip-restrict]) to accept only the [APIM VIP][apim-faq-vip].</span></span>
    * <span data-ttu-id="0a841-137">セキュア トランスポート/SSL を有効にして、Azure Web Apps でホストされます。</span><span class="sxs-lookup"><span data-stu-id="0a841-137">Is hosted in Azure Web Apps with Secure Transport/SSL turned on.</span></span>
    * <span data-ttu-id="0a841-138">承認が有効にされ、Azure Active Directory と OAuth2 を使用して、[Azure App Service に提供され][azure-appservice-auth]ています。</span><span class="sxs-lookup"><span data-stu-id="0a841-138">Has authorization enabled, [provided by the Azure App Service][azure-appservice-auth] using Azure Active Directory and OAuth2.</span></span>
2. <span data-ttu-id="0a841-139">新しいブラウザーベースの Web アプリケーションは、既存の HTTP API と新しい API の**両方**のために Azure API Management インスタンスに依存します。</span><span class="sxs-lookup"><span data-stu-id="0a841-139">The new browser-based web application will depend on the Azure API Management instance for **both** the existing HTTP API and the new API.</span></span>

<span data-ttu-id="0a841-140">APIM インスタンスは、レガシ HTTP サービスを新しい API コントラクトにマップするように構成されます。</span><span class="sxs-lookup"><span data-stu-id="0a841-140">The APIM instance will be configured to map the legacy HTTP services to a new API contract.</span></span> <span data-ttu-id="0a841-141">これにより、新しい Web UI は、一連のレガシ サービス/API と新しい API との統合を認識していません。</span><span class="sxs-lookup"><span data-stu-id="0a841-141">By doing this, the new Web UI is unaware of the integration with a set of legacy services/APIs and new APIs.</span></span> <span data-ttu-id="0a841-142">今後、プロジェクト チームは段階的に機能を新しい API に移植し、元のサービスを廃止します。</span><span class="sxs-lookup"><span data-stu-id="0a841-142">In the future, the project team will gradually port functionality to the new APIs and retire the original services.</span></span> <span data-ttu-id="0a841-143">このような変更は、APIM の構成内で処理され、フロントエンド UI は影響を受けないため、再開発作業を回避できます。</span><span class="sxs-lookup"><span data-stu-id="0a841-143">These changes will be handled within APIM configuration, leaving the front-end UI unaffected and avoiding redevelopment work.</span></span>

### <a name="alternatives"></a><span data-ttu-id="0a841-144">代替手段</span><span class="sxs-lookup"><span data-stu-id="0a841-144">Alternatives</span></span>

* <span data-ttu-id="0a841-145">組織が、レガシ アプリケーションをホストしている VM を含め、インフラストラクチャを完全に Azure に移行する予定だった場合でも、APIM は、アドレス指定可能な HTTP エンドポイントのファサードとして機能することができるため、優れた選択肢になります。</span><span class="sxs-lookup"><span data-stu-id="0a841-145">If the organization was planning to move their infrastructure entirely to Azure, including the VMs hosting the legacy applications, then APIM would still be a great option since it can act as a facade for any addressable HTTP endpoint.</span></span>
* <span data-ttu-id="0a841-146">お客様が既存のエンドポイントを非公開なままにして公開しないと決めた場合、API Management インスタンスを [Azure Virtual Network (VNet)][azure-vnet] にリンクできます。</span><span class="sxs-lookup"><span data-stu-id="0a841-146">If the customer had decided to keep the existing endpoints private and not expose them publicly, their API Management instance could be linked to an [Azure Virtual Network (VNet)][azure-vnet]:</span></span>
  * <span data-ttu-id="0a841-147">デプロイされた Azure Virtual Network にリンクされた [Azure リフトおよびシフト シナリオ][azure-vm-lift-shift]では、お客様はプライベート IP アドレスを介してバックエンド サービスに直接アドレス指定することができました。</span><span class="sxs-lookup"><span data-stu-id="0a841-147">In an [Azure lift and shift scenario][azure-vm-lift-shift] linked to their deployed Azure Virtual Network, the customer could directly address the back-end service through private IP addresses.</span></span>
  * <span data-ttu-id="0a841-148">オンプレミスのシナリオで、API Management インスタンスは、[Azure VPN ゲートウェイとサイト間 IPSec VPN 接続][azure-vpn]または [ExpressRoute][azure-er] を介して、非公開で内部サービスに到達し、これを[ハイブリッドの Azure およびオンプレミス シナリオ][azure-hybrid]にすることができます。</span><span class="sxs-lookup"><span data-stu-id="0a841-148">In the on-premises scenario, the API Management instance could reach back to the internal service privately via an [Azure VPN gateway and site-to-site IPSec VPN connection][azure-vpn] or [ExpressRoute][azure-er] making this a [hybrid Azure and on-premises scenario][azure-hybrid].</span></span>
* <span data-ttu-id="0a841-149">内部モードで API Management インスタンスを展開することで、API Management インスタンスを非公開のままにすることができます。</span><span class="sxs-lookup"><span data-stu-id="0a841-149">The API Management instance can be kept private by deploying the API Management instance in Internal mode.</span></span> <span data-ttu-id="0a841-150">この展開を [Azure Application Gateway][azure-appgw] と共に使用することで、いくつかの API のパブリック アクセスを可能にしながら、その他を内部のままにすることができます。</span><span class="sxs-lookup"><span data-stu-id="0a841-150">The deployment could then be used with an [Azure Application Gateway][azure-appgw] to enable public access for some APIs while others remain internal.</span></span> <span data-ttu-id="0a841-151">詳細については、[内部モードの APIM を VNET に接続する方法][apim-vnet-internal]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="0a841-151">For more information, see [Connecting APIM in internal mode to a VNET][apim-vnet-internal].</span></span>

> [!NOTE]
> <span data-ttu-id="0a841-152">API Management を VNET に接続するための一般的な情報については、[こちらを参照してください][apim-vnet]。</span><span class="sxs-lookup"><span data-stu-id="0a841-152">For general information on connecting API Management to a VNET, [see here][apim-vnet].</span></span>

### <a name="availability-and-scalability"></a><span data-ttu-id="0a841-153">可用性とスケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="0a841-153">Availability and scalability</span></span>

* <span data-ttu-id="0a841-154">Azure API Management は、価格レベルを選択し、ユニットを追加することで[スケールアウト][apim-scaleout]することができます。</span><span class="sxs-lookup"><span data-stu-id="0a841-154">Azure API Management can be [scaled out][apim-scaleout] by choosing a pricing tier and then adding units.</span></span>
* <span data-ttu-id="0a841-155">スケーリングは[自動スケーリング][apim-autoscale]で自動的に行われます。</span><span class="sxs-lookup"><span data-stu-id="0a841-155">Scaling also happen [automatically with auto scaling][apim-autoscale].</span></span>
* <span data-ttu-id="0a841-156">[複数リージョンにデプロイ][apim-multi-regions]すると、フェールオーバー オプションが有効になります。これは [Premium レベル][apim-pricing]で実行できます。</span><span class="sxs-lookup"><span data-stu-id="0a841-156">[Deploying across multiple regions][apim-multi-regions] will enable fail over options and can be done in the [Premium tier][apim-pricing].</span></span>
* <span data-ttu-id="0a841-157">監視のために [Azure Monitor][azure-mon] を介してメトリックに接続することもできる [Azure Application Insights との統合][azure-apim-ai]を検討してみてください。</span><span class="sxs-lookup"><span data-stu-id="0a841-157">Consider [Integrating with Azure Application Insights][azure-apim-ai], which also surfaces metrics through [Azure Monitor][azure-mon] for monitoring.</span></span>

## <a name="deployment"></a><span data-ttu-id="0a841-158">Deployment</span><span class="sxs-lookup"><span data-stu-id="0a841-158">Deployment</span></span>

<span data-ttu-id="0a841-159">まず、[ポータルで Azure API Management インスタンスを作成します。][apim-create]</span><span class="sxs-lookup"><span data-stu-id="0a841-159">To get started, [create an Azure API Management instance in the portal.][apim-create]</span></span>

<span data-ttu-id="0a841-160">または、特定のユース ケースに合わせて既存の Azure Resource Manager [クイックスタート テンプレート][azure-quickstart-templates-apim]から選択することもできます。</span><span class="sxs-lookup"><span data-stu-id="0a841-160">Alternatively, you can choose from an existing Azure Resource Manager [quickstart template][azure-quickstart-templates-apim] that aligns to your specific use case.</span></span>

## <a name="pricing"></a><span data-ttu-id="0a841-161">価格</span><span class="sxs-lookup"><span data-stu-id="0a841-161">Pricing</span></span>

<span data-ttu-id="0a841-162">API Management は、Developer、Basic、Standard、Premium の 4 つのレベルで提供されています。</span><span class="sxs-lookup"><span data-stu-id="0a841-162">API Management is offered in four tiers: developer, basic, standard, and premium.</span></span> <span data-ttu-id="0a841-163">これらのレベルの違いの詳細については、[こちらの Azure API Management の価格ガイダンスを参照してください。][apim-pricing]</span><span class="sxs-lookup"><span data-stu-id="0a841-163">You can find detailed guidance on the difference in these tiers at the [Azure API Management pricing guidance here.][apim-pricing]</span></span>

<span data-ttu-id="0a841-164">お客様はユニットの追加や削除を行うことにより、API Management をスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="0a841-164">Customers can scale API Management by adding and removing units.</span></span> <span data-ttu-id="0a841-165">各ユニットのキャパシティは、そのレベルによって異なります。</span><span class="sxs-lookup"><span data-stu-id="0a841-165">Each unit has capacity that depends on its tier.</span></span>

> [!NOTE]
> <span data-ttu-id="0a841-166">Developer レベルは、API Management 機能の評価に使用できます。</span><span class="sxs-lookup"><span data-stu-id="0a841-166">The Developer tier can be used for evaluation of the API Management features.</span></span> <span data-ttu-id="0a841-167">Developer レベルは運用に使用しないでください。</span><span class="sxs-lookup"><span data-stu-id="0a841-167">The Developer tier should not be used for production.</span></span>

<span data-ttu-id="0a841-168">予想されるコストを表示し、デプロイのニーズに合わせてカスタマイズするには、[Azue 料金計算ツール][pricing-calculator]でスケール ユニットと App Service インスタンスの数を変更します。</span><span class="sxs-lookup"><span data-stu-id="0a841-168">To view projected costs and customize to your deployment needs, you can modify the number of scale units and App Service instances in the [Azue Pricing Calculator][pricing-calculator].</span></span>

## <a name="related-resources"></a><span data-ttu-id="0a841-169">関連リソース</span><span class="sxs-lookup"><span data-stu-id="0a841-169">Related resources</span></span>

<span data-ttu-id="0a841-170">さまざまな Azure API Management の[ドキュメントとリファレンスの記事][apim]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="0a841-170">Check out the extensive Azure API Management [documentation and reference articles.][apim]</span></span>

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
