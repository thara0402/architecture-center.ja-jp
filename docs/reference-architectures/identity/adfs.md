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
ms.openlocfilehash: f48f8e41b6b90931e7393808cf41ae01d7187416
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429266"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="abbef-104">Active Directory フェデレーション サービス (AD FS) を Azure に拡張する</span><span class="sxs-lookup"><span data-stu-id="abbef-104">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="abbef-105">この参照アーキテクチャは、オンプレミス ネットワークを Azure に拡張する安全なハイブリッド ネットワークを実装し、[Active Directory フェデレーション サービス (AD FS)][active-directory-federation-services] を使用して、Azure で実行されているコンポーネントのフェデレーション認証と承認を実行します。</span><span class="sxs-lookup"><span data-stu-id="abbef-105">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> [<span data-ttu-id="abbef-106">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="abbef-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="abbef-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="abbef-107">[![0]][0]</span></span>

<span data-ttu-id="abbef-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="abbef-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="abbef-109">AD FS はオンプレミスでホストできますが、アプリケーションがハイブリッドであって、その一部が Azure で実装されている場合は、クラウドに AD FS をレプリケートする方が効率的です。</span><span class="sxs-lookup"><span data-stu-id="abbef-109">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span> 

<span data-ttu-id="abbef-110">この図は、以下のシナリオを示しています。</span><span class="sxs-lookup"><span data-stu-id="abbef-110">The diagram shows the following scenarios:</span></span>

* <span data-ttu-id="abbef-111">自社の Azure VNet 内でホストされている Web アプリケーションに、取引先組織のアプリケーション コードがアクセスします。</span><span class="sxs-lookup"><span data-stu-id="abbef-111">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
* <span data-ttu-id="abbef-112">自社の Azure VNet 内でホストされている Web アプリケーションに、資格情報が Active Directory Domain Services (DS) 内に格納されている外部登録ユーザーがアクセスします。</span><span class="sxs-lookup"><span data-stu-id="abbef-112">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
* <span data-ttu-id="abbef-113">承認済みのデバイスを使用して自社の VNet に接続しているユーザーが、自社の Azure VNet 内でホストされている Web アプリケーションを実行します。</span><span class="sxs-lookup"><span data-stu-id="abbef-113">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="abbef-114">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="abbef-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="abbef-115">ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="abbef-115">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="abbef-116">フェデレーション承認を使用して Web アプリケーションを取引先組織に公開するソリューション。</span><span class="sxs-lookup"><span data-stu-id="abbef-116">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
* <span data-ttu-id="abbef-117">組織のファイアウォールの外部で実行されている Web ブラウザーからのアクセスをサポートするシステム。</span><span class="sxs-lookup"><span data-stu-id="abbef-117">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="abbef-118">リモート コンピューター、ノートブック、その他のモバイル デバイスなどの承認された外部デバイスから接続することで、ユーザーが Web アプリケーションにアクセスできるようにするシステム。</span><span class="sxs-lookup"><span data-stu-id="abbef-118">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span> 

<span data-ttu-id="abbef-119">この参照アーキテクチャでは、"*パッシブ フェデレーション*" に焦点を当てています。この場合、フェデレーション サーバーがユーザーの認証方法とタイミングを決定します。</span><span class="sxs-lookup"><span data-stu-id="abbef-119">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="abbef-120">ユーザーは、アプリケーションの起動時にサインイン情報を指定します。</span><span class="sxs-lookup"><span data-stu-id="abbef-120">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="abbef-121">このメカニズムは、Web ブラウザーで最もよく使用され、ユーザーが認証するサイトにブラウザーをリダイレクトするプロトコルが含まれています。</span><span class="sxs-lookup"><span data-stu-id="abbef-121">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="abbef-122">AD FS は、"*アクティブ フェデレーション*" もサポートしています。この場合、アプリケーションはユーザーによる操作なしで資格情報を提供する責任を負いますが、そのシナリオはこのアーキテクチャの範囲外です。</span><span class="sxs-lookup"><span data-stu-id="abbef-122">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="abbef-123">その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="abbef-123">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="abbef-124">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="abbef-124">Architecture</span></span>

<span data-ttu-id="abbef-125">このアーキテクチャは、[Azure への AD DS の拡張][extending-ad-to-azure]に関するページで説明されている実装を拡張します。</span><span class="sxs-lookup"><span data-stu-id="abbef-125">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="abbef-126">これには、次のコンポーネントが含まれます。</span><span class="sxs-lookup"><span data-stu-id="abbef-126">It contains the followign components.</span></span>

* <span data-ttu-id="abbef-127">**AD DS サブネット**。</span><span class="sxs-lookup"><span data-stu-id="abbef-127">**AD DS subnet**.</span></span> <span data-ttu-id="abbef-128">AD DS サーバーは、ネットワーク セキュリティ グループ (NSG) 規則がファイアウォールとして機能している独自のサブネットに含まれています。</span><span class="sxs-lookup"><span data-stu-id="abbef-128">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

* <span data-ttu-id="abbef-129">**AD DS サーバー**。</span><span class="sxs-lookup"><span data-stu-id="abbef-129">**AD DS servers**.</span></span> <span data-ttu-id="abbef-130">Azure で VM として実行されているドメイン コントローラー。</span><span class="sxs-lookup"><span data-stu-id="abbef-130">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="abbef-131">これらのサーバーは、ドメイン内のローカル ID の認証を提供します。</span><span class="sxs-lookup"><span data-stu-id="abbef-131">These servers provide authentication of local identities within the domain.</span></span>

* <span data-ttu-id="abbef-132">**AD FS サブネット**。</span><span class="sxs-lookup"><span data-stu-id="abbef-132">**AD FS subnet**.</span></span> <span data-ttu-id="abbef-133">AD FS サーバーは、NSG 規則がファイアウォールとして機能している独自のサブネット内に存在します。</span><span class="sxs-lookup"><span data-stu-id="abbef-133">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

* <span data-ttu-id="abbef-134">**AD FS サーバー**。</span><span class="sxs-lookup"><span data-stu-id="abbef-134">**AD FS servers**.</span></span> <span data-ttu-id="abbef-135">AD FS サーバーは、フェデレーション承認と認証を提供します。</span><span class="sxs-lookup"><span data-stu-id="abbef-135">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="abbef-136">このアーキテクチャでは、以下のタスクを実行します。</span><span class="sxs-lookup"><span data-stu-id="abbef-136">In this architecture, they perform the following tasks:</span></span>
  
  * <span data-ttu-id="abbef-137">パートナー ユーザーの代わりにパートナー フェデレーション サーバーが作成した要求を含むセキュリティ トークンを受け取ります。</span><span class="sxs-lookup"><span data-stu-id="abbef-137">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="abbef-138">AD FS は、トークンが有効であることを確認してから、要求の承認のために、Azure で実行されている Web アプリケーションに要求を渡します。</span><span class="sxs-lookup"><span data-stu-id="abbef-138">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span> 
  
    <span data-ttu-id="abbef-139">Azure で実行されている Web アプリケーションは、"*証明書利用者*" です。</span><span class="sxs-lookup"><span data-stu-id="abbef-139">The web application running in Azure is the *relying party*.</span></span> <span data-ttu-id="abbef-140">パートナー フェデレーション サーバーは、Web アプリケーションで認識される要求を発行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-140">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="abbef-141">パートナー フェデレーション サーバーは、取引先組織内の認証済みアカウントに代わってアクセス要求を送信するため、"*アカウント パートナー*" と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="abbef-141">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="abbef-142">AD FS サーバーは、リソース (Web アプリケーション) へのアクセスを提供するため、"*リソース パートナー*" と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="abbef-142">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  * <span data-ttu-id="abbef-143">Web アプリケーションへのアクセスを必要とする Web ブラウザーまたはデバイスを実行している外部ユーザーから受信した要求の認証と承認を、AD DS と [Active Directory Device Registration Service][ADDRS] を使用して行います。</span><span class="sxs-lookup"><span data-stu-id="abbef-143">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>
    
  <span data-ttu-id="abbef-144">AD FS サーバーは、Azure ロード バランサーを通じてアクセスされるファームとして構成されています。</span><span class="sxs-lookup"><span data-stu-id="abbef-144">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="abbef-145">この実装により、可用性とスケーラビリティが向上します。</span><span class="sxs-lookup"><span data-stu-id="abbef-145">This implementation improves availability and scalability.</span></span> <span data-ttu-id="abbef-146">AD FS サーバーは、インターネットに直接公開されません。</span><span class="sxs-lookup"><span data-stu-id="abbef-146">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="abbef-147">すべてのインターネット トラフィックは、AD FS Web アプリケーション プロキシ サーバーと DMZ (境界ネットワークとも呼ばれます) を介してフィルター処理されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-147">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="abbef-148">AD FS のしくみの詳細については、[Active Directory フェデレーション サービスの概要][active-directory-federation-services-overview]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-148">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="abbef-149">また、[Azure での AD FS デプロイ][adfs-intro]に関する記事には、実装の詳細な手順が記載されています。</span><span class="sxs-lookup"><span data-stu-id="abbef-149">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

* <span data-ttu-id="abbef-150">**AD FS プロキシ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="abbef-150">**AD FS proxy subnet**.</span></span> <span data-ttu-id="abbef-151">AD FS プロキシ サーバーは、独自のサブネット内に含めることができ、NSG 規則によって保護が提供されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-151">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="abbef-152">このサブネット内のサーバーは、Azure 仮想ネットワークとインターネットの間にファイアウォールを提供する一連のネットワーク仮想アプライアンスを通じて、インターネットに公開されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-152">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

* <span data-ttu-id="abbef-153">**AD FS Web アプリケーション プロキシ (WAP) サーバー**。</span><span class="sxs-lookup"><span data-stu-id="abbef-153">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="abbef-154">これらの VM は、取引先組織と外部デバイスからの受信要求に対する AD FS サーバーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="abbef-154">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="abbef-155">WAP サーバーはフィルターとして機能し、インターネットからの直接アクセスから AD FS サーバーを保護します。</span><span class="sxs-lookup"><span data-stu-id="abbef-155">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="abbef-156">AD FS サーバーと同様に、負荷分散されているファームに WAP サーバーをデプロイすると、一連のスタンドアロン サーバーをデプロイする場合よりも可用性とスケーラビリティが向上します。</span><span class="sxs-lookup"><span data-stu-id="abbef-156">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="abbef-157">WAP サーバーのインストールの詳細については、「[Web アプリケーション プロキシ サーバーをインストールし、構成する][install_and_configure_the_web_application_proxy_server]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-157">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  > 
  > 

* <span data-ttu-id="abbef-158">**取引先組織**。</span><span class="sxs-lookup"><span data-stu-id="abbef-158">**Partner organization**.</span></span> <span data-ttu-id="abbef-159">Azure で実行されている Web アプリケーションへのアクセスを要求する、Web アプリケーションを実行している取引先組織。</span><span class="sxs-lookup"><span data-stu-id="abbef-159">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="abbef-160">取引先組織のフェデレーション サーバーは、ローカルで要求を認証し、要求が含まれているセキュリティ トークンを Azure で実行されている AD FS に送信します。</span><span class="sxs-lookup"><span data-stu-id="abbef-160">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="abbef-161">Azure の AD FS はセキュリティ トークンを検証し、有効であれば、Azure で実行されている Web アプリケーションに要求を渡して承認することができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-161">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="abbef-162">信頼できるパートナーに AD FS への直接アクセスを提供するように、Azure ゲートウェイを使用して VPN トンネルを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="abbef-162">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="abbef-163">これらのパートナーから受信した要求は、WAP サーバーをパススルーしません。</span><span class="sxs-lookup"><span data-stu-id="abbef-163">Requests received from these partners do not pass through the WAP servers.</span></span>
  > 
  > 

<span data-ttu-id="abbef-164">アーキテクチャの AD FS に関連していない部分の詳細については、以下を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-164">For more information about the parts of the architecture that are not related to AD FS, see the following:</span></span>
- <span data-ttu-id="abbef-165">[Azure に安全なハイブリッド ネットワーク アーキテクチャを実装する][implementing-a-secure-hybrid-network-architecture]</span><span class="sxs-lookup"><span data-stu-id="abbef-165">[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]</span></span>
- <span data-ttu-id="abbef-166">[インターネットへのアクセスを備えた安全なハイブリッド ネットワーク アーキテクチャを Azure に実装する][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span><span class="sxs-lookup"><span data-stu-id="abbef-166">[Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]</span></span>
- <span data-ttu-id="abbef-167">[Active Directory ID を使用する安全なハイブリッド ネットワーク アーキテクチャを Azure に実装する][extending-ad-to-azure]</span><span class="sxs-lookup"><span data-stu-id="abbef-167">[Implementing a secure hybrid network architecture with Active Directory identities in Azure][extending-ad-to-azure].</span></span>


## <a name="recommendations"></a><span data-ttu-id="abbef-168">Recommendations</span><span class="sxs-lookup"><span data-stu-id="abbef-168">Recommendations</span></span>

<span data-ttu-id="abbef-169">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-169">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="abbef-170">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="abbef-170">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="abbef-171">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="abbef-171">VM recommendations</span></span>

<span data-ttu-id="abbef-172">予想されるトラフィック量を処理するための十分なリソースを持つ VM を作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-172">Create VMs with sufficient resources to handle the expected volume of traffic.</span></span> <span data-ttu-id="abbef-173">まずは、オンプレミスで AD FS をホストしている既存のマシンのサイズを使用します。</span><span class="sxs-lookup"><span data-stu-id="abbef-173">Use the size of the existing machines hosting AD FS on premises as a starting point.</span></span> <span data-ttu-id="abbef-174">リソースの使用率を監視します。</span><span class="sxs-lookup"><span data-stu-id="abbef-174">Monitor the resource utilization.</span></span> <span data-ttu-id="abbef-175">VM が大きすぎる場合は、VM のサイズを変更してスケールダウンすることができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-175">You can resize the VMs and scale down if they are too large.</span></span>

<span data-ttu-id="abbef-176">[Azure での Windows VM の実行][vm-recommendations]に関するページに記載されている推奨事項に従ってください。</span><span class="sxs-lookup"><span data-stu-id="abbef-176">Follow the recommendations listed in [Running a Windows VM on Azure][vm-recommendations].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="abbef-177">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="abbef-177">Networking recommendations</span></span>

<span data-ttu-id="abbef-178">AD FS サーバーおよび WAP サーバーをホストしている各 VM のネットワーク インターフェイスを、静的プライベート IP アドレスを使用して構成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-178">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="abbef-179">AD FS VM にパブリック IP アドレスを指定しないでください。</span><span class="sxs-lookup"><span data-stu-id="abbef-179">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="abbef-180">詳細については、「セキュリティに関する考慮事項」セクションを参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-180">For more information, see the Security considerations section.</span></span>

<span data-ttu-id="abbef-181">各 AD FS VM および WAP VM のネットワーク インターフェイスが Active Directory DS VM を参照するように、優先およびセカンダリ ドメイン ネーム サービス (DNS) サーバーの IP アドレスを設定します。</span><span class="sxs-lookup"><span data-stu-id="abbef-181">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="abbef-182">Active Directory DS VMS は、DNS を実行している必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-182">The Active Directory DS VMS should be running DNS.</span></span> <span data-ttu-id="abbef-183">この手順は、各 VM がドメインに参加できるようにするために必要です。</span><span class="sxs-lookup"><span data-stu-id="abbef-183">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-availability"></a><span data-ttu-id="abbef-184">AD FS の可用性</span><span class="sxs-lookup"><span data-stu-id="abbef-184">AD FS availability</span></span> 

<span data-ttu-id="abbef-185">サービスの可用性を向上させるには、少なくとも 2 台のサーバーを含む AD FS ファームを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-185">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="abbef-186">ファーム内の AD FS VM ごとに異なるストレージ アカウントを使用します。</span><span class="sxs-lookup"><span data-stu-id="abbef-186">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="abbef-187">この手法により、1 つのストレージ アカウントのエラーによってファーム全体にアクセスできなくなることを回避します。</span><span class="sxs-lookup"><span data-stu-id="abbef-187">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="abbef-188">[マネージド ディスク](/azure/storage/storage-managed-disks-overview)を使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="abbef-188">We recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="abbef-189">マネージド ディスクでは、ストレージ アカウントは必要ありません。</span><span class="sxs-lookup"><span data-stu-id="abbef-189">Managed disks do not require a storage account.</span></span> <span data-ttu-id="abbef-190">ディスクのサイズと種類を指定するだけで、可用性の高い方法でデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="abbef-190">You simply specify the size and type of disk and it is deployed in a highly available way.</span></span> <span data-ttu-id="abbef-191">Microsoft の[参照用アーキテクチャ](/azure/architecture/reference-architectures/)では、現在、マネージド ディスクがデプロイされていませんが、[テンプレートの構成要素](https://github.com/mspnp/template-building-blocks/wiki)は、バージョン 2 でマネージド ディスクをデプロイするように更新される予定です。</span><span class="sxs-lookup"><span data-stu-id="abbef-191">Our [reference architectures](/azure/architecture/reference-architectures/) do not currently deploy managed disks but the [template building blocks](https://github.com/mspnp/template-building-blocks/wiki) will be updated to deploy managed disks in version 2.</span></span>

<span data-ttu-id="abbef-192">AD FS VM と WAP VM に個別の Azure 可用性セットを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-192">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="abbef-193">各セットに少なくとも 2 つの VM があることを確認します。</span><span class="sxs-lookup"><span data-stu-id="abbef-193">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="abbef-194">各可用性セットには、少なくとも 2 つの更新ドメインと 2 つの障害ドメインが必要です。</span><span class="sxs-lookup"><span data-stu-id="abbef-194">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="abbef-195">AD FS VM および WAP VM のロード バランサーを次のように構成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-195">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

* <span data-ttu-id="abbef-196">Azure ロード バランサーを使用して WAP VM への外部アクセスを提供し、内部ロード バランサーを使用してファーム内の AD FS サーバー間で負荷を分散します。</span><span class="sxs-lookup"><span data-stu-id="abbef-196">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
* <span data-ttu-id="abbef-197">ポート 443 (HTTPS) で生じるトラフィックのみを AD FS/WAP サーバーに渡します。</span><span class="sxs-lookup"><span data-stu-id="abbef-197">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
* <span data-ttu-id="abbef-198">ロード バランサーに静的 IP アドレスを指定します。</span><span class="sxs-lookup"><span data-stu-id="abbef-198">Give the load balancer a static IP address.</span></span>
* <span data-ttu-id="abbef-199">`/adfs/probe` に対して HTTP を使用して、正常性プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-199">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="abbef-200">詳細については、「[Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2 (ハードウェア ロード バランサー正常性チェックと Web アプリケーション プロキシ/AD FS 2012 R2)](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-200">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>
  
  > [!NOTE]
  > <span data-ttu-id="abbef-201">AD FS サーバーは Server Name Indication (SNI) プロトコルを使用しているため、ロード バランサーから HTTPS エンドポイントを使用してプローブしようとすると失敗します。</span><span class="sxs-lookup"><span data-stu-id="abbef-201">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  > 
  > 
* <span data-ttu-id="abbef-202">AD FS ロード バランサーのドメインに DNS *A* レコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="abbef-202">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="abbef-203">ロード バランサーの IP アドレスを指定し、ドメイン内での名前を指定します (adfs.contoso.com など)。</span><span class="sxs-lookup"><span data-stu-id="abbef-203">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="abbef-204">これは、クライアントと WAP サーバーが AD FS サーバー ファームにアクセスする際に使用する名前です。</span><span class="sxs-lookup"><span data-stu-id="abbef-204">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

### <a name="ad-fs-security"></a><span data-ttu-id="abbef-205">AD FS のセキュリティ</span><span class="sxs-lookup"><span data-stu-id="abbef-205">AD FS security</span></span> 

<span data-ttu-id="abbef-206">AD FS サーバーがインターネットに直接公開されないようにします。</span><span class="sxs-lookup"><span data-stu-id="abbef-206">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="abbef-207">AD FS サーバーは、セキュリティ トークンを付与するための完全な権限を持つ、ドメイン参加コンピューターです。</span><span class="sxs-lookup"><span data-stu-id="abbef-207">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="abbef-208">サーバーが侵害されると、悪意のあるユーザーは、すべての Web アプリケーションのほか、AD FS によって保護されているすべてのフェデレーション サーバーに対するフル アクセス トークンを発行できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-208">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="abbef-209">信頼できるパートナー サイトから接続されていない外部ユーザーからの要求をシステムで処理する必要がある場合は、WAP サーバーを使用してそれらの要求を処理します。</span><span class="sxs-lookup"><span data-stu-id="abbef-209">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="abbef-210">詳細については、「[フェデレーション サーバー プロキシを配置する場所][where-to-place-an-fs-proxy]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-210">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="abbef-211">AD FS サーバーと WAP サーバーをそれぞれ独自のファイアウォールを持つ別々のサブネットに配置します。</span><span class="sxs-lookup"><span data-stu-id="abbef-211">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="abbef-212">NSG 規則を使用してファイアウォール規則を定義できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-212">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="abbef-213">より包括的な保護が必要な場合は、[インターネット アクセスを備えた安全なハイブリッド ネットワーク アーキテクチャの Azure への実装][implementing-a-secure-hybrid-network-architecture-with-internet-access]に関するドキュメントで説明されているように、サブネットとネットワーク仮想アプライアンス (NVA) のペアを使用して、サーバーの周囲に追加のセキュリティ境界を実装できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-213">If you require more comprehensive protection you can implement an additional security perimeter around servers by using a pair of subnets and network virtual appliances (NVAs), as described in the document [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="abbef-214">すべてのファイアウォールは、ポート 443 (HTTPS) のトラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-214">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="abbef-215">AD FS サーバーおよび WAP サーバーへの直接的なサインイン アクセスを制限します。</span><span class="sxs-lookup"><span data-stu-id="abbef-215">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="abbef-216">DevOps スタッフだけが接続できるようにしてください。</span><span class="sxs-lookup"><span data-stu-id="abbef-216">Only DevOps staff should be able to connect.</span></span>

<span data-ttu-id="abbef-217">WAP サーバーをドメインに参加させないでください。</span><span class="sxs-lookup"><span data-stu-id="abbef-217">Do not join the WAP servers to the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="abbef-218">AD FS のインストール</span><span class="sxs-lookup"><span data-stu-id="abbef-218">AD FS installation</span></span> 

<span data-ttu-id="abbef-219">[フェデレーション サーバー ファームのデプロイ][Deploying_a_federation_server_farm]に関する記事には、AD FS のインストールと構成の詳細な手順が記載されています。</span><span class="sxs-lookup"><span data-stu-id="abbef-219">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="abbef-220">ファーム内の最初の AD FS サーバーを構成する前に、次のタスクを実行してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-220">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="abbef-221">サーバー認証を行うために一般的に信頼されている証明書を取得します。</span><span class="sxs-lookup"><span data-stu-id="abbef-221">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="abbef-222">"*サブジェクト名*" には、クライアントがフェデレーション サービスへのアクセスに使用する名前を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-222">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="abbef-223">これには、*adfs.contoso.com* など、ロード バランサーに登録されている DNS 名を指定することもできます (セキュリティ上の理由から、\**.contoso.com* などのワイルドカード名の使用は避けてください)。</span><span class="sxs-lookup"><span data-stu-id="abbef-223">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="abbef-224">すべての AD FS サーバー VM では同じ証明書を使用します。</span><span class="sxs-lookup"><span data-stu-id="abbef-224">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="abbef-225">信頼された証明機関から証明書を購入できますが、組織で Active Directory 証明書サービスを使用している場合は、独自の証明書を作成できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-225">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span> 
   
    <span data-ttu-id="abbef-226">"*サブジェクトの別名*" は、外部デバイスからのアクセスを可能にするために、デバイス登録サービス (DRS) によって使用されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-226">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="abbef-227">これは、*enterpriseregistration.contoso.com* の形式にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-227">This should be of the form *enterpriseregistration.contoso.com*.</span></span>
   
    <span data-ttu-id="abbef-228">詳細については、[AD FS 用の Secure Sockets Layer (SSL) 証明書の取得と構成][adfs_certificates]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-228">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="abbef-229">ドメイン コントローラーで、キー配布サービスの新しいルート キーを生成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-229">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="abbef-230">有効時間を現在の時刻から 10 時間差し引いた時間に設定します (この構成により、ドメイン全体でのキーの配布と同期で発生する可能性のある遅延が軽減されます)。</span><span class="sxs-lookup"><span data-stu-id="abbef-230">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="abbef-231">この手順は、AD FS サービスの実行に使用されるグループ サービス アカウントの作成をサポートするために必要です。</span><span class="sxs-lookup"><span data-stu-id="abbef-231">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="abbef-232">次の PowerShell コマンドは、その実行方法の例を示しています。</span><span class="sxs-lookup"><span data-stu-id="abbef-232">The following PowerShell command shows an example of how to do this:</span></span>
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="abbef-233">各 AD FS サーバー VM をドメインに追加します。</span><span class="sxs-lookup"><span data-stu-id="abbef-233">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="abbef-234">AD FS をインストールするには、ドメインでプライマリ ドメイン コントローラー (PDC) エミュレーターの Flexible Single Master Operation (FSMO) ロールを実行しているドメイン コントローラーが実行されていて、AD FS VM からアクセス可能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-234">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="abbef-235"><<RBC: この繰り返しを少なくする方法はありますか?>></span><span class="sxs-lookup"><span data-stu-id="abbef-235"><<RBC: Is there a way to make this less repetitive?>></span></span>
> 
> 

### <a name="ad-fs-trust"></a><span data-ttu-id="abbef-236">AD FS の信頼</span><span class="sxs-lookup"><span data-stu-id="abbef-236">AD FS trust</span></span> 

<span data-ttu-id="abbef-237">AD FS インストールと、すべての取引先組織のフェデレーション サーバーとの間で、フェデレーションの信頼を確立します。</span><span class="sxs-lookup"><span data-stu-id="abbef-237">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="abbef-238">必要な任意の要求のフィルターおよびマッピングを構成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-238">Configure any claims filtering and mapping required.</span></span> 

* <span data-ttu-id="abbef-239">各取引先組織の DevOps スタッフは、AD FS サーバーを介してアクセス可能な Web アプリケーションの証明書利用者信頼を追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-239">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
* <span data-ttu-id="abbef-240">所属する組織の DevOps スタッフは、取引先組織から提供される要求を AD FS サーバーが信頼できるように、要求とプロバイダー間の信頼を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-240">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
* <span data-ttu-id="abbef-241">所属する組織の DevOps スタッフは、組織の Web アプリケーションに要求を渡すように AD FS を構成する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="abbef-241">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>
  
<span data-ttu-id="abbef-242">詳細については、「[Establishing Federation Trust (フェデレーションの信頼の確立)][establishing-federation-trust]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-242">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="abbef-243">組織の Web アプリケーションを発行し、それを外部パートナーが使用できるようにするには、WAP サーバーを介した事前認証を使用します。</span><span class="sxs-lookup"><span data-stu-id="abbef-243">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="abbef-244">詳細については、「[AD FS 事前認証を使用してアプリケーションを公開する][publish_applications_using_AD_FS_preauthentication]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-244">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="abbef-245">AD FS は、トークンの変換と拡張をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="abbef-245">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="abbef-246">Azure Active Directory では、この機能を提供していません。</span><span class="sxs-lookup"><span data-stu-id="abbef-246">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="abbef-247">AD FS を使用した場合、信頼関係を設定する際に、次のことが可能です。</span><span class="sxs-lookup"><span data-stu-id="abbef-247">With AD FS, when you set up the trust relationships, you can:</span></span>

* <span data-ttu-id="abbef-248">承認規則の要求変換を構成する。</span><span class="sxs-lookup"><span data-stu-id="abbef-248">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="abbef-249">たとえば、Microsoft 以外の取引先組織で使用されている表現のグループ セキュリティを、所属組織内で Active Directory DS が承認できるものにマップできます。</span><span class="sxs-lookup"><span data-stu-id="abbef-249">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that that Active Directory DS can authorize in your organization.</span></span>
* <span data-ttu-id="abbef-250">要求の形式を変換する。</span><span class="sxs-lookup"><span data-stu-id="abbef-250">Transform claims from one format to another.</span></span> <span data-ttu-id="abbef-251">たとえば、アプリケーションで SAML 1.1 要求のみがサポートされている場合は、SAML 2.0 から SAML 1.1 にマップすることができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-251">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="abbef-252">AD FS の監視</span><span class="sxs-lookup"><span data-stu-id="abbef-252">AD FS monitoring</span></span> 

<span data-ttu-id="abbef-253">[Active Directory フェデレーション サービス 2012 R2 用 Microsoft System Center 管理パック][oms-adfs-pack]を使用すると、フェデレーション サーバーの AD FS デプロイをプロアクティブにもリアクティブにも監視することができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-253">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="abbef-254">この管理パックで監視するものは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="abbef-254">This management pack monitors:</span></span>

* <span data-ttu-id="abbef-255">AD FS サービスがイベント ログに記録するイベント。</span><span class="sxs-lookup"><span data-stu-id="abbef-255">Events that the AD FS service records in its event logs.</span></span>
* <span data-ttu-id="abbef-256">AD FS パフォーマンス カウンターで収集されるパフォーマンス データ。</span><span class="sxs-lookup"><span data-stu-id="abbef-256">The performance data that the AD FS performance counters collect.</span></span> 
* <span data-ttu-id="abbef-257">AD FS システムおよび Web アプリケーション (証明書利用者) の全体的な正常性。重大な問題や警告のアラートも提供されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-257">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span> 

## <a name="scalability-considerations"></a><span data-ttu-id="abbef-258">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="abbef-258">Scalability considerations</span></span>

<span data-ttu-id="abbef-259">「[AD FS の配置を計画する][plan-your-adfs-deployment]」の記事から要約された以下の考慮事項は、AD FS ファームのサイズ設定の出発点となります。</span><span class="sxs-lookup"><span data-stu-id="abbef-259">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

* <span data-ttu-id="abbef-260">ユーザーが 1,000 人未満の場合は、専用サーバーを作成しないでください。ただし、その代わりに、クラウド内の各 Active Directory DS サーバーに AD FS をインストールします。</span><span class="sxs-lookup"><span data-stu-id="abbef-260">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="abbef-261">可用性を維持するために、少なくとも 2 台の Active Directory DS サーバーがあることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-261">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="abbef-262">1 つの WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-262">Create a single WAP server.</span></span>
* <span data-ttu-id="abbef-263">ユーザーが 1,000 ～ 15,000 人の場合は、2 台の専用 AD FS サーバーと 2 台の専用 WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-263">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
* <span data-ttu-id="abbef-264">ユーザーが 15,000 ～ 60,000 人の場合は、3 ～ 5 台の専用 AD FS サーバーと、少なくとも 2 台の専用 WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-264">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="abbef-265">これらの考慮事項では、Azure でデュアル クアッドコア VM (Standard D4_v2 以上) サイズを使用していることを想定しています。</span><span class="sxs-lookup"><span data-stu-id="abbef-265">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="abbef-266">AD FS 構成データの格納に Windows Internal Database を使用している場合、ファーム内の AD FS サーバーは 8 台に制限されます。</span><span class="sxs-lookup"><span data-stu-id="abbef-266">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="abbef-267">将来的にもっと必要になることが予想される場合は、SQL Server を使用してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-267">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="abbef-268">詳細については、「[AD FS 構成データベースの役割][adfs-configuration-database]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-268">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="abbef-269">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="abbef-269">Availability considerations</span></span>

<span data-ttu-id="abbef-270">AD FS の構成情報を保持するには、SQL Server または Windows Internal Database を使用できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-270">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="abbef-271">Windows Internal Database は、基本的な冗長性を提供します。</span><span class="sxs-lookup"><span data-stu-id="abbef-271">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="abbef-272">変更は AD FS クラスター内の 1 つの AD FS データベースだけに直接書き込まれる一方で、他のサーバーはプル レプリケーションを使用してそれらのデータベースを最新の状態に保ちます。</span><span class="sxs-lookup"><span data-stu-id="abbef-272">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="abbef-273">SQL Server を使用すると、フェールオーバー クラスタリングまたはミラーリングを使用してデータベースの完全な冗長性と高可用性を実現できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-273">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="abbef-274">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="abbef-274">Manageability considerations</span></span>

<span data-ttu-id="abbef-275">DevOps スタッフは、以下のタスクの実行を準備する必要があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-275">DevOps staff should be prepared to perform the following tasks:</span></span>

* <span data-ttu-id="abbef-276">フェデレーション サーバーの管理 (AD FS ファームの管理、フェデレーション サーバーでの信頼ポリシーの管理、フェデレーション サービスで使用される証明書の管理など)。</span><span class="sxs-lookup"><span data-stu-id="abbef-276">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
* <span data-ttu-id="abbef-277">WAP サーバーの管理 (WAP ファームや証明書の管理など)。</span><span class="sxs-lookup"><span data-stu-id="abbef-277">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
* <span data-ttu-id="abbef-278">Web アプリケーションの管理 (証明書利用者、認証方法、要求のマッピングの構成など)。</span><span class="sxs-lookup"><span data-stu-id="abbef-278">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
* <span data-ttu-id="abbef-279">AD FS コンポーネントのバックアップ。</span><span class="sxs-lookup"><span data-stu-id="abbef-279">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="abbef-280">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="abbef-280">Security considerations</span></span>

<span data-ttu-id="abbef-281">AD FS は HTTPS プロトコルを使用するため、Web 層 VM が含まれているサブネットの NSG 規則で HTTPS 要求が許可されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-281">AD FS utilizes the HTTPS protocol, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="abbef-282">これらの要求は、オンプレミス ネットワークのほか、Web 層、ビジネス層、データ層、プライベート DMZ、パブリック DMZ を含むサブネットや AD FS サーバーを含むサブネットから発信される場合があります。</span><span class="sxs-lookup"><span data-stu-id="abbef-282">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="abbef-283">監査のために、仮想ネットワークの境界を越えるトラフィックに関する詳細情報を記録する一連のネットワーク仮想アプライアンスの使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-283">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="abbef-284">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="abbef-284">Deploy the solution</span></span>

<span data-ttu-id="abbef-285">この参照用アーキテクチャをデプロイするためのソリューションは、[GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="abbef-285">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="abbef-286">このソリューションをデプロイする Powershell スクリプトを実行するには、最新バージョンの [Azure CLI][azure-cli] が必要です。</span><span class="sxs-lookup"><span data-stu-id="abbef-286">You will need the latest version of the [Azure CLI][azure-cli] to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="abbef-287">参照アーキテクチャをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="abbef-287">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="abbef-288">[GitHub][github] からローカル マシンにソリューション フォルダーをダウンロードまたは複製します。</span><span class="sxs-lookup"><span data-stu-id="abbef-288">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="abbef-289">Azure CLI を開き、ローカルのソリューション フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="abbef-289">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="abbef-290">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="abbef-290">Run the following command:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    <span data-ttu-id="abbef-291">`<subscription id>` は、Azure サブスクリプション ID に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="abbef-291">Replace `<subscription id>` with your Azure subscription ID.</span></span>
   
    <span data-ttu-id="abbef-292">`<location>` には、Azure リージョン (`eastus` や `westus` など) を指定します。</span><span class="sxs-lookup"><span data-stu-id="abbef-292">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
   
    <span data-ttu-id="abbef-293">`<mode>` パラメーターは、デプロイの細分性を制御します。このパラメーターの値は次のいずれかになります。</span><span class="sxs-lookup"><span data-stu-id="abbef-293">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
   
   * <span data-ttu-id="abbef-294">`Onpremise`: シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-294">`Onpremise`: Deploys a simulated on-premises environment.</span></span> <span data-ttu-id="abbef-295">既存のオンプレミス ネットワークがない場合、または既存のオンプレミス ネットワークの構成を変更せずにこの参照用アーキテクチャをテストしたい場合は、このデプロイを使用するとテストおよび実験することができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-295">You can use this deployment to test and experiment if you do not have an existing on-premises network, or if you want to test this reference architecture without changing the configuration of your existing on-premises network.</span></span>
   * <span data-ttu-id="abbef-296">`Infrastructure`: VNet インフラストラクチャとジャンプ ボックスをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-296">`Infrastructure`: deploys the VNet infrastructure and jump box.</span></span>
   * <span data-ttu-id="abbef-297">`CreateVpn`: Azure 仮想ネットワーク ゲートウェイをデプロイし、それをシミュレートされたオンプレミス ネットワークに接続します。</span><span class="sxs-lookup"><span data-stu-id="abbef-297">`CreateVpn`: deploys an Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   * <span data-ttu-id="abbef-298">`AzureADDS`: Active Directory DS サーバーとして機能する VM をデプロイし、Active Directory をそれらの VM にデプロイして、Azure にドメインを作成します。</span><span class="sxs-lookup"><span data-stu-id="abbef-298">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and creates the domain in Azure.</span></span>
   * <span data-ttu-id="abbef-299">`AdfsVm`: AD FS VM をデプロイし、それらを Azure のドメインに参加させます。</span><span class="sxs-lookup"><span data-stu-id="abbef-299">`AdfsVm`: deploys the AD FS VMs and joins them to the domain in Azure.</span></span>
   * <span data-ttu-id="abbef-300">`PublicDMZ`: Azure にパブリック DMZ をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-300">`PublicDMZ`: deploys the public DMZ in Azure.</span></span>
   * <span data-ttu-id="abbef-301">`ProxyVm`: AD FS プロキシ VM をデプロイし、それらを Azure のドメインに参加させます。</span><span class="sxs-lookup"><span data-stu-id="abbef-301">`ProxyVm`: deploys the AD FS proxy VMs and joins them to the domain in Azure.</span></span>
   * <span data-ttu-id="abbef-302">`Prepare`: 上記のすべてをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-302">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="abbef-303">**これは、まったく新しいデプロイを構築していて、既存のオンプレミス インフラストラクチャがない場合に推奨されるオプションです。**</span><span class="sxs-lookup"><span data-stu-id="abbef-303">**This is the recommended option if you are building an entirely new deployment and you don't have an existing on-premises infrastructure.**</span></span> 
   * <span data-ttu-id="abbef-304">`Workload`: オプションで、Web 層、ビジネス層、データ層の VM とサポートするネットワークをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-304">`Workload`: optionally deploys web, business, and data tier VMs and supporting network.</span></span> <span data-ttu-id="abbef-305">`Prepare` デプロイ モードには含まれていません。</span><span class="sxs-lookup"><span data-stu-id="abbef-305">Not included in the `Prepare` deployment mode.</span></span>
   * <span data-ttu-id="abbef-306">`PrivateDMZ`: オプションで、上でデプロイした `Workload` VM の前に、Azure のプライベート DMZ をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-306">`PrivateDMZ`: optionally deploys the private DMZ in Azure in front of the `Workload` VMs deployed above.</span></span> <span data-ttu-id="abbef-307">`Prepare` デプロイ モードには含まれていません。</span><span class="sxs-lookup"><span data-stu-id="abbef-307">Not included in the `Prepare` deployment mode.</span></span>

4. <span data-ttu-id="abbef-308">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="abbef-308">Wait for the deployment to complete.</span></span> <span data-ttu-id="abbef-309">`Prepare` オプションを使用した場合、デプロイは完了するまでに数時間かかり、`Preparation is completed. Please install certificate to all AD FS and proxy VMs.` というメッセージで終了します。</span><span class="sxs-lookup"><span data-stu-id="abbef-309">If you used the `Prepare` option, the deployment takes several hours to complete, and finishes with the message `Preparation is completed. Please install certificate to all AD FS and proxy VMs.`</span></span>

5. <span data-ttu-id="abbef-310">ジャンプ ボックス (*ra-adfs-security-rg* グループの *ra-adfs-mgmt-vm1*) を再起動すると、その DNS 設定を有効にすることができます。</span><span class="sxs-lookup"><span data-stu-id="abbef-310">Restart the jump box (*ra-adfs-mgmt-vm1* in the *ra-adfs-security-rg* group) to allow its DNS settings to take effect.</span></span>

6. <span data-ttu-id="abbef-311">[AD FS の SSL 証明書を入手][adfs_certificates]し、この証明書を AD FS VM にインストールします。</span><span class="sxs-lookup"><span data-stu-id="abbef-311">[Obtain an SSL Certificate for AD FS][adfs_certificates] and install this certificate on the AD FS VMs.</span></span> <span data-ttu-id="abbef-312">ジャンプ ボックスを使用してそれらに接続できることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="abbef-312">Note that you can connect to them through the jump box.</span></span> <span data-ttu-id="abbef-313">IP アドレスは、<em>10.0.5.4</em> と <em>10.0.5.5</em> です。</span><span class="sxs-lookup"><span data-stu-id="abbef-313">The IP addresses are <em>10.0.5.4</em> and <em>10.0.5.5</em>.</span></span> <span data-ttu-id="abbef-314">既定のユーザー名は <em>contoso\testuser</em>、パスワードは <em>AweSome@PW</em> です。</span><span class="sxs-lookup"><span data-stu-id="abbef-314">The default username is <em>contoso\testuser</em> with password <em>AweSome@PW</em>.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="abbef-315">Deploy-ReferenceArchitecture.ps1 スクリプトのこの部分のコメントは、`makecert` コマンドを使用して自己署名テスト証明書と機関を作成するための詳細な手順を示しています。</span><span class="sxs-lookup"><span data-stu-id="abbef-315">The comments in the Deploy-ReferenceArchitecture.ps1 script at this point provides detailed instructions for creating a self-signed test certificate and authority using the `makecert` command.</span></span> <span data-ttu-id="abbef-316">ただし、この手順は**テスト**として実行するだけにし、makecert によって生成された証明書を運用環境では使用しないでください。</span><span class="sxs-lookup"><span data-stu-id="abbef-316">However, perform these steps as a **test** only and do not use the certificates generated by makecert in a production environment.</span></span>
   > 
   > 

7. <span data-ttu-id="abbef-317">次の PowerShell コマンドを実行して、AD FS サーバー ファームをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-317">Run the following PowerShell command to deploy the AD FS server farm:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. <span data-ttu-id="abbef-318">ジャンプ ボックスで `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` に移動し、AD FS のインストールをテストします (証明書の警告が表示されることがありますが、このテストでは無視できます)。</span><span class="sxs-lookup"><span data-stu-id="abbef-318">On the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` to test the AD FS installation (you may receive a certificate warning that you can ignore for this test).</span></span> <span data-ttu-id="abbef-319">Contoso Corporation サインイン ページが表示されることを確認します。</span><span class="sxs-lookup"><span data-stu-id="abbef-319">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="abbef-320">パスワード <em>AweS0me@PW</em> を使用して <em>contoso\testuser</em> としてサインインします。</span><span class="sxs-lookup"><span data-stu-id="abbef-320">Sign in as <em>contoso\testuser</em> with password <em>AweS0me@PW</em>.</span></span>

9. <span data-ttu-id="abbef-321">SSL 証明書を AD FS プロキシ VM にインストールします。</span><span class="sxs-lookup"><span data-stu-id="abbef-321">Install the SSL certificate on the AD FS proxy VMs.</span></span> <span data-ttu-id="abbef-322">IP アドレスは *10.0.6.4* と *10.0.6.5* です。</span><span class="sxs-lookup"><span data-stu-id="abbef-322">The IP addresses are *10.0.6.4* and *10.0.6.5*.</span></span>

10. <span data-ttu-id="abbef-323">次の PowerShell コマンドを実行し、最初の AD FS プロキシ サーバーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-323">Run the following PowerShell command to deploy the first AD FS proxy server:</span></span>
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. <span data-ttu-id="abbef-324">スクリプトによって表示される指示に従って、最初のプロキシ サーバーのインストールをテストします。</span><span class="sxs-lookup"><span data-stu-id="abbef-324">Follow the instructions displayed by the script to test the installation of the first proxy server.</span></span>

12. <span data-ttu-id="abbef-325">次の PowerShell コマンドを実行して、2 番目のプロキシ サーバーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="abbef-325">Run the following PowerShell command to deploy the second proxy server:</span></span>
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. <span data-ttu-id="abbef-326">スクリプトによって表示される指示に従って、完全なプロキシ構成をテストします。</span><span class="sxs-lookup"><span data-stu-id="abbef-326">Follow the instructions displayed by the script to test the complete proxy configuration.</span></span>

## <a name="next-steps"></a><span data-ttu-id="abbef-327">次の手順</span><span class="sxs-lookup"><span data-stu-id="abbef-327">Next steps</span></span>

* <span data-ttu-id="abbef-328">[Azure Active Directory][aad] について学習します。</span><span class="sxs-lookup"><span data-stu-id="abbef-328">Learn about [Azure Active Directory][aad].</span></span>
* <span data-ttu-id="abbef-329">[Azure Active Directory B2C][aadb2c] について学習します。</span><span class="sxs-lookup"><span data-stu-id="abbef-329">Learn about [Azure Active Directory B2C][aadb2c].</span></span>

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
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"
