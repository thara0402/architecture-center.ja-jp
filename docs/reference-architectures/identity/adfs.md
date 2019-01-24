---
title: オンプレミスの AD FS を Azure に拡張する
titleSuffix: Azure Reference Architectures
description: Active Directory フェデレーション サービスの承認を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure 上に実装します。
author: telmosampaio
ms.date: 12/18.2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 22a2a2042c85e70d0d5a523c9ecf72395a9e774c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488275"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="adc1f-103">Active Directory フェデレーション サービス (AD FS) を Azure に拡張する</span><span class="sxs-lookup"><span data-stu-id="adc1f-103">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="adc1f-104">この参照アーキテクチャは、オンプレミス ネットワークを Azure に拡張する安全なハイブリッド ネットワークを実装し、[Active Directory フェデレーション サービス (AD FS)][active-directory-federation-services] を使用して、Azure で実行されているコンポーネントのフェデレーション認証と承認を実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-104">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> <span data-ttu-id="adc1f-105">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](./images/adfs.png)

<span data-ttu-id="adc1f-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="adc1f-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="adc1f-108">AD FS はオンプレミスでホストできますが、アプリケーションがハイブリッドであって、その一部が Azure で実装されている場合は、クラウドに AD FS をレプリケートする方が効率的です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-108">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span>

<span data-ttu-id="adc1f-109">この図は、以下のシナリオを示しています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-109">The diagram shows the following scenarios:</span></span>

- <span data-ttu-id="adc1f-110">自社の Azure VNet 内でホストされている Web アプリケーションに、取引先組織のアプリケーション コードがアクセスします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-110">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="adc1f-111">自社の Azure VNet 内でホストされている Web アプリケーションに、資格情報が Active Directory Domain Services (DS) 内に格納されている外部登録ユーザーがアクセスします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-111">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="adc1f-112">承認済みのデバイスを使用して自社の VNet に接続しているユーザーが、自社の Azure VNet 内でホストされている Web アプリケーションを実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-112">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="adc1f-113">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="adc1f-113">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="adc1f-114">ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="adc1f-114">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="adc1f-115">フェデレーション承認を使用して Web アプリケーションを取引先組織に公開するソリューション。</span><span class="sxs-lookup"><span data-stu-id="adc1f-115">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
- <span data-ttu-id="adc1f-116">組織のファイアウォールの外部で実行されている Web ブラウザーからのアクセスをサポートするシステム。</span><span class="sxs-lookup"><span data-stu-id="adc1f-116">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="adc1f-117">リモート コンピューター、ノートブック、その他のモバイル デバイスなどの承認された外部デバイスから接続することで、ユーザーが Web アプリケーションにアクセスできるようにするシステム。</span><span class="sxs-lookup"><span data-stu-id="adc1f-117">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span>

<span data-ttu-id="adc1f-118">この参照アーキテクチャでは、"*パッシブ フェデレーション*" に焦点を当てています。この場合、フェデレーション サーバーがユーザーの認証方法とタイミングを決定します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-118">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="adc1f-119">ユーザーは、アプリケーションの起動時にサインイン情報を指定します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-119">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="adc1f-120">このメカニズムは、Web ブラウザーで最もよく使用され、ユーザーが認証するサイトにブラウザーをリダイレクトするプロトコルが含まれています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-120">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="adc1f-121">AD FS は、"*アクティブ フェデレーション*" もサポートしています。この場合、アプリケーションはユーザーによる操作なしで資格情報を提供する責任を負いますが、そのシナリオはこのアーキテクチャの範囲外です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-121">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="adc1f-122">その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-122">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="adc1f-123">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="adc1f-123">Architecture</span></span>

<span data-ttu-id="adc1f-124">このアーキテクチャは、[Azure への AD DS の拡張][extending-ad-to-azure]に関するページで説明されている実装を拡張します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-124">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="adc1f-125">これには、次のコンポーネントが含まれます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-125">It contains the followign components.</span></span>

- <span data-ttu-id="adc1f-126">**AD DS サブネット**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-126">**AD DS subnet**.</span></span> <span data-ttu-id="adc1f-127">AD DS サーバーは、ネットワーク セキュリティ グループ (NSG) 規則がファイアウォールとして機能している独自のサブネットに含まれています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-127">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

- <span data-ttu-id="adc1f-128">**AD DS サーバー**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-128">**AD DS servers**.</span></span> <span data-ttu-id="adc1f-129">Azure で VM として実行されているドメイン コントローラー。</span><span class="sxs-lookup"><span data-stu-id="adc1f-129">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="adc1f-130">これらのサーバーは、ドメイン内のローカル ID の認証を提供します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-130">These servers provide authentication of local identities within the domain.</span></span>

- <span data-ttu-id="adc1f-131">**AD FS サブネット**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-131">**AD FS subnet**.</span></span> <span data-ttu-id="adc1f-132">AD FS サーバーは、NSG 規則がファイアウォールとして機能している独自のサブネット内に存在します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-132">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

- <span data-ttu-id="adc1f-133">**AD FS サーバー**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-133">**AD FS servers**.</span></span> <span data-ttu-id="adc1f-134">AD FS サーバーは、フェデレーション承認と認証を提供します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-134">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="adc1f-135">このアーキテクチャでは、以下のタスクを実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-135">In this architecture, they perform the following tasks:</span></span>

  - <span data-ttu-id="adc1f-136">パートナー ユーザーの代わりにパートナー フェデレーション サーバーが作成した要求を含むセキュリティ トークンを受け取ります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-136">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="adc1f-137">AD FS は、トークンが有効であることを確認してから、要求の承認のために、Azure で実行されている Web アプリケーションに要求を渡します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-137">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span>

    <span data-ttu-id="adc1f-138">Azure で実行されるアプリケーションが "*証明書利用者*" になります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-138">The application running in Azure is the *relying party*.</span></span> <span data-ttu-id="adc1f-139">パートナー フェデレーション サーバーは、Web アプリケーションで認識される要求を発行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-139">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="adc1f-140">パートナー フェデレーション サーバーは、取引先組織内の認証済みアカウントに代わってアクセス要求を送信するため、"*アカウント パートナー*" と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-140">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="adc1f-141">AD FS サーバーは、リソース (Web アプリケーション) へのアクセスを提供するため、"*リソース パートナー*" と呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-141">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  - <span data-ttu-id="adc1f-142">Web アプリケーションへのアクセスを必要とする Web ブラウザーまたはデバイスを実行している外部ユーザーから受信した要求の認証と承認を、AD DS と [Active Directory Device Registration Service][ADDRS] を使用して行います。</span><span class="sxs-lookup"><span data-stu-id="adc1f-142">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>

  <span data-ttu-id="adc1f-143">AD FS サーバーは、Azure ロード バランサーを通じてアクセスされるファームとして構成されています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-143">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="adc1f-144">この実装により、可用性とスケーラビリティが向上します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-144">This implementation improves availability and scalability.</span></span> <span data-ttu-id="adc1f-145">AD FS サーバーは、インターネットに直接公開されません。</span><span class="sxs-lookup"><span data-stu-id="adc1f-145">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="adc1f-146">すべてのインターネット トラフィックは、AD FS Web アプリケーション プロキシ サーバーと DMZ (境界ネットワークとも呼ばれます) を介してフィルター処理されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-146">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="adc1f-147">AD FS のしくみの詳細については、[Active Directory フェデレーション サービスの概要][active-directory-federation-services-overview]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-147">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="adc1f-148">また、[Azure での AD FS デプロイ][adfs-intro]に関する記事には、実装の詳細な手順が記載されています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-148">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

- <span data-ttu-id="adc1f-149">**AD FS プロキシ サブネット**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-149">**AD FS proxy subnet**.</span></span> <span data-ttu-id="adc1f-150">AD FS プロキシ サーバーは、独自のサブネット内に含めることができ、NSG 規則によって保護が提供されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-150">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="adc1f-151">このサブネット内のサーバーは、Azure 仮想ネットワークとインターネットの間にファイアウォールを提供する一連のネットワーク仮想アプライアンスを通じて、インターネットに公開されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-151">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

- <span data-ttu-id="adc1f-152">**AD FS Web アプリケーション プロキシ (WAP) サーバー**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-152">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="adc1f-153">これらの VM は、取引先組織と外部デバイスからの受信要求に対する AD FS サーバーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-153">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="adc1f-154">WAP サーバーはフィルターとして機能し、インターネットからの直接アクセスから AD FS サーバーを保護します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-154">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="adc1f-155">AD FS サーバーと同様に、負荷分散されているファームに WAP サーバーをデプロイすると、一連のスタンドアロン サーバーをデプロイする場合よりも可用性とスケーラビリティが向上します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-155">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>

  > [!NOTE]
  > <span data-ttu-id="adc1f-156">WAP サーバーのインストールの詳細については、「[Web アプリケーション プロキシ サーバーをインストールし、構成する][install_and_configure_the_web_application_proxy_server]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-156">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  >

- <span data-ttu-id="adc1f-157">**取引先組織**。</span><span class="sxs-lookup"><span data-stu-id="adc1f-157">**Partner organization**.</span></span> <span data-ttu-id="adc1f-158">Azure で実行されている Web アプリケーションへのアクセスを要求する、Web アプリケーションを実行している取引先組織。</span><span class="sxs-lookup"><span data-stu-id="adc1f-158">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="adc1f-159">取引先組織のフェデレーション サーバーは、ローカルで要求を認証し、要求が含まれているセキュリティ トークンを Azure で実行されている AD FS に送信します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-159">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="adc1f-160">Azure の AD FS はセキュリティ トークンを検証し、有効であれば、Azure で実行されている Web アプリケーションに要求を渡して承認することができます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-160">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>

  > [!NOTE]
  > <span data-ttu-id="adc1f-161">信頼できるパートナーに AD FS への直接アクセスを提供するように、Azure ゲートウェイを使用して VPN トンネルを構成することもできます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-161">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="adc1f-162">これらのパートナーから受信した要求は、WAP サーバーをパススルーしません。</span><span class="sxs-lookup"><span data-stu-id="adc1f-162">Requests received from these partners do not pass through the WAP servers.</span></span>
  >

## <a name="recommendations"></a><span data-ttu-id="adc1f-163">Recommendations</span><span class="sxs-lookup"><span data-stu-id="adc1f-163">Recommendations</span></span>

<span data-ttu-id="adc1f-164">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-164">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="adc1f-165">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-165">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="adc1f-166">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="adc1f-166">Networking recommendations</span></span>

<span data-ttu-id="adc1f-167">AD FS サーバーおよび WAP サーバーをホストしている各 VM のネットワーク インターフェイスを、静的プライベート IP アドレスを使用して構成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-167">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="adc1f-168">AD FS VM にパブリック IP アドレスを指定しないでください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-168">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="adc1f-169">詳細については、「[セキュリティに関する考慮事項](#security-considerations)」セクションを参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-169">For more information, see the [Security considerations](#security-considerations) section.</span></span>

<span data-ttu-id="adc1f-170">各 AD FS VM および WAP VM のネットワーク インターフェイスが Active Directory DS VM を参照するように、優先およびセカンダリ ドメイン ネーム サービス (DNS) サーバーの IP アドレスを設定します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-170">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="adc1f-171">Active Directory DS VM では、DNS が実行されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-171">The Active Directory DS VMs should be running DNS.</span></span> <span data-ttu-id="adc1f-172">この手順は、各 VM がドメインに参加できるようにするために必要です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-172">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="adc1f-173">AD FS のインストール</span><span class="sxs-lookup"><span data-stu-id="adc1f-173">AD FS installation</span></span>

<span data-ttu-id="adc1f-174">[フェデレーション サーバー ファームのデプロイ][Deploying_a_federation_server_farm]に関する記事には、AD FS のインストールと構成の詳細な手順が記載されています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-174">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="adc1f-175">ファーム内の最初の AD FS サーバーを構成する前に、次のタスクを実行してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-175">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="adc1f-176">サーバー認証を行うために一般的に信頼されている証明書を取得します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-176">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="adc1f-177">"*サブジェクト名*" には、クライアントがフェデレーション サービスへのアクセスに使用する名前を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-177">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="adc1f-178">これには、*adfs.contoso.com* など、ロード バランサーに登録されている DNS 名を指定することもできます (セキュリティ上の理由から、\**.contoso.com* などのワイルドカード名の使用は避けてください)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-178">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="adc1f-179">すべての AD FS サーバー VM では同じ証明書を使用します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-179">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="adc1f-180">信頼された証明機関から証明書を購入できますが、組織で Active Directory 証明書サービスを使用している場合は、独自の証明書を作成できます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-180">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span>

    <span data-ttu-id="adc1f-181">"*サブジェクトの別名*" は、外部デバイスからのアクセスを可能にするために、デバイス登録サービス (DRS) によって使用されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-181">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="adc1f-182">これは、*enterpriseregistration.contoso.com* の形式にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-182">This should be of the form *enterpriseregistration.contoso.com*.</span></span>

    <span data-ttu-id="adc1f-183">詳細については、[AD FS 用の Secure Sockets Layer (SSL) 証明書の取得と構成][adfs_certificates]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-183">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="adc1f-184">ドメイン コントローラーで、キー配布サービスの新しいルート キーを生成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-184">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="adc1f-185">有効時間を現在の時刻から 10 時間差し引いた時間に設定します (この構成により、ドメイン全体でのキーの配布と同期で発生する可能性のある遅延が軽減されます)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-185">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="adc1f-186">この手順は、AD FS サービスの実行に使用されるグループ サービス アカウントの作成をサポートするために必要です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-186">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="adc1f-187">次の PowerShell コマンドは、その実行方法の例を示しています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-187">The following PowerShell command shows an example of how to do this:</span></span>

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="adc1f-188">各 AD FS サーバー VM をドメインに追加します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-188">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="adc1f-189">AD FS をインストールするには、ドメインでプライマリ ドメイン コントローラー (PDC) エミュレーターの Flexible Single Master Operation (FSMO) ロールを実行しているドメイン コントローラーが実行されていて、AD FS VM からアクセス可能である必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-189">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="adc1f-190"><<RBC:この繰り返しを少なくする方法はありますか?>></span><span class="sxs-lookup"><span data-stu-id="adc1f-190"><<RBC: Is there a way to make this less repetitive?>></span></span>
>

### <a name="ad-fs-trust"></a><span data-ttu-id="adc1f-191">AD FS の信頼</span><span class="sxs-lookup"><span data-stu-id="adc1f-191">AD FS trust</span></span>

<span data-ttu-id="adc1f-192">AD FS インストールと、すべての取引先組織のフェデレーション サーバーとの間で、フェデレーションの信頼を確立します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-192">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="adc1f-193">必要な任意の要求のフィルターおよびマッピングを構成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-193">Configure any claims filtering and mapping required.</span></span>

- <span data-ttu-id="adc1f-194">各取引先組織の DevOps スタッフは、AD FS サーバーを介してアクセス可能な Web アプリケーションの証明書利用者信頼を追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-194">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
- <span data-ttu-id="adc1f-195">所属する組織の DevOps スタッフは、取引先組織から提供される要求を AD FS サーバーが信頼できるように、要求とプロバイダー間の信頼を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-195">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
- <span data-ttu-id="adc1f-196">所属する組織の DevOps スタッフは、組織の Web アプリケーションに要求を渡すように AD FS を構成する必要もあります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-196">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>

<span data-ttu-id="adc1f-197">詳細については、「[Establishing Federation Trust (フェデレーションの信頼の確立)][establishing-federation-trust]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-197">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="adc1f-198">組織の Web アプリケーションを発行し、それを外部パートナーが使用できるようにするには、WAP サーバーを介した事前認証を使用します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-198">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="adc1f-199">詳細については、「[AD FS 事前認証を使用してアプリケーションを公開する][publish_applications_using_AD_FS_preauthentication]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-199">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="adc1f-200">AD FS は、トークンの変換と拡張をサポートしています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-200">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="adc1f-201">Azure Active Directory では、この機能を提供していません。</span><span class="sxs-lookup"><span data-stu-id="adc1f-201">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="adc1f-202">AD FS を使用した場合、信頼関係を設定する際に、次のことが可能です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-202">With AD FS, when you set up the trust relationships, you can:</span></span>

- <span data-ttu-id="adc1f-203">承認規則の要求変換を構成する。</span><span class="sxs-lookup"><span data-stu-id="adc1f-203">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="adc1f-204">たとえば、Microsoft 以外の取引先組織で使用されている表現のグループ セキュリティを、所属組織内で Active Directory DS が承認できるものにマップできます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-204">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that that Active Directory DS can authorize in your organization.</span></span>
- <span data-ttu-id="adc1f-205">要求の形式を変換する。</span><span class="sxs-lookup"><span data-stu-id="adc1f-205">Transform claims from one format to another.</span></span> <span data-ttu-id="adc1f-206">たとえば、アプリケーションで SAML 1.1 要求のみがサポートされている場合は、SAML 2.0 から SAML 1.1 にマップすることができます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-206">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="adc1f-207">AD FS の監視</span><span class="sxs-lookup"><span data-stu-id="adc1f-207">AD FS monitoring</span></span>

<span data-ttu-id="adc1f-208">[Active Directory フェデレーション サービス 2012 R2 用 Microsoft System Center 管理パック][oms-adfs-pack]を使用すると、フェデレーション サーバーの AD FS デプロイをプロアクティブにもリアクティブにも監視することができます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-208">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="adc1f-209">この管理パックで監視するものは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="adc1f-209">This management pack monitors:</span></span>

- <span data-ttu-id="adc1f-210">AD FS サービスがイベント ログに記録するイベント。</span><span class="sxs-lookup"><span data-stu-id="adc1f-210">Events that the AD FS service records in its event logs.</span></span>
- <span data-ttu-id="adc1f-211">AD FS パフォーマンス カウンターで収集されるパフォーマンス データ。</span><span class="sxs-lookup"><span data-stu-id="adc1f-211">The performance data that the AD FS performance counters collect.</span></span>
- <span data-ttu-id="adc1f-212">AD FS システムおよび Web アプリケーション (証明書利用者) の全体的な正常性。重大な問題や警告のアラートも提供されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-212">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="adc1f-213">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="adc1f-213">Scalability considerations</span></span>

<span data-ttu-id="adc1f-214">「[AD FS の配置を計画する][plan-your-adfs-deployment]」の記事から要約された以下の考慮事項は、AD FS ファームのサイズ設定の出発点となります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-214">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

- <span data-ttu-id="adc1f-215">ユーザーが 1,000 人未満の場合は、専用サーバーを作成しないでください。ただし、その代わりに、クラウド内の各 Active Directory DS サーバーに AD FS をインストールします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-215">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="adc1f-216">可用性を維持するために、少なくとも 2 台の Active Directory DS サーバーがあることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-216">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="adc1f-217">1 つの WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-217">Create a single WAP server.</span></span>
- <span data-ttu-id="adc1f-218">ユーザーが 1,000 ～ 15,000 人の場合は、2 台の専用 AD FS サーバーと 2 台の専用 WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-218">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
- <span data-ttu-id="adc1f-219">ユーザーが 15,000 ～ 60,000 人の場合は、3 ～ 5 台の専用 AD FS サーバーと、少なくとも 2 台の専用 WAP サーバーを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-219">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="adc1f-220">これらの考慮事項では、Azure でデュアル クアッドコア VM (Standard D4_v2 以上) サイズを使用していることを想定しています。</span><span class="sxs-lookup"><span data-stu-id="adc1f-220">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="adc1f-221">AD FS 構成データの格納に Windows Internal Database を使用している場合、ファーム内の AD FS サーバーは 8 台に制限されます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-221">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="adc1f-222">将来的にもっと必要になることが予想される場合は、SQL Server を使用してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-222">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="adc1f-223">詳細については、「[AD FS 構成データベースの役割][adfs-configuration-database]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-223">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="adc1f-224">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="adc1f-224">Availability considerations</span></span>

<span data-ttu-id="adc1f-225">サービスの可用性を向上させるには、少なくとも 2 台のサーバーを含む AD FS ファームを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-225">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="adc1f-226">ファーム内の AD FS VM ごとに異なるストレージ アカウントを使用します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-226">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="adc1f-227">この手法により、1 つのストレージ アカウントのエラーによってファーム全体にアクセスできなくなることを回避します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-227">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

<span data-ttu-id="adc1f-228">AD FS VM と WAP VM に個別の Azure 可用性セットを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-228">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="adc1f-229">各セットに少なくとも 2 つの VM があることを確認します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-229">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="adc1f-230">各可用性セットには、少なくとも 2 つの更新ドメインと 2 つの障害ドメインが必要です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-230">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="adc1f-231">AD FS VM および WAP VM のロード バランサーを次のように構成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-231">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

- <span data-ttu-id="adc1f-232">Azure ロード バランサーを使用して WAP VM への外部アクセスを提供し、内部ロード バランサーを使用してファーム内の AD FS サーバー間で負荷を分散します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-232">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
- <span data-ttu-id="adc1f-233">ポート 443 (HTTPS) で生じるトラフィックのみを AD FS/WAP サーバーに渡します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-233">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
- <span data-ttu-id="adc1f-234">ロード バランサーに静的 IP アドレスを指定します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-234">Give the load balancer a static IP address.</span></span>
- <span data-ttu-id="adc1f-235">`/adfs/probe` に対して HTTP を使用して、正常性プローブを作成します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-235">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="adc1f-236">詳細については、「[Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2 (ハードウェア ロード バランサー正常性チェックと Web アプリケーション プロキシ/AD FS 2012 R2)](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-236">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>

  > [!NOTE]
  > <span data-ttu-id="adc1f-237">AD FS サーバーは Server Name Indication (SNI) プロトコルを使用しているため、ロード バランサーから HTTPS エンドポイントを使用してプローブしようとすると失敗します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-237">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  >

- <span data-ttu-id="adc1f-238">AD FS ロード バランサーのドメインに DNS *A* レコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-238">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="adc1f-239">ロード バランサーの IP アドレスを指定し、ドメイン内での名前を指定します (adfs.contoso.com など)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-239">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="adc1f-240">これは、クライアントと WAP サーバーが AD FS サーバー ファームにアクセスする際に使用する名前です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-240">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

<span data-ttu-id="adc1f-241">AD FS の構成情報を保持するには、SQL Server または Windows Internal Database を使用できます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-241">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="adc1f-242">Windows Internal Database は、基本的な冗長性を提供します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-242">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="adc1f-243">変更は AD FS クラスター内の 1 つの AD FS データベースだけに直接書き込まれる一方で、他のサーバーはプル レプリケーションを使用してそれらのデータベースを最新の状態に保ちます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-243">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="adc1f-244">SQL Server を使用すると、フェールオーバー クラスタリングまたはミラーリングを使用してデータベースの完全な冗長性と高可用性を実現できます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-244">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="adc1f-245">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="adc1f-245">Manageability considerations</span></span>

<span data-ttu-id="adc1f-246">DevOps スタッフは、以下のタスクの実行を準備する必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-246">DevOps staff should be prepared to perform the following tasks:</span></span>

- <span data-ttu-id="adc1f-247">フェデレーション サーバーの管理 (AD FS ファームの管理、フェデレーション サーバーでの信頼ポリシーの管理、フェデレーション サービスで使用される証明書の管理など)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-247">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
- <span data-ttu-id="adc1f-248">WAP サーバーの管理 (WAP ファームや証明書の管理など)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-248">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
- <span data-ttu-id="adc1f-249">Web アプリケーションの管理 (証明書利用者、認証方法、要求のマッピングの構成など)。</span><span class="sxs-lookup"><span data-stu-id="adc1f-249">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
- <span data-ttu-id="adc1f-250">AD FS コンポーネントのバックアップ。</span><span class="sxs-lookup"><span data-stu-id="adc1f-250">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="adc1f-251">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="adc1f-251">Security considerations</span></span>

<span data-ttu-id="adc1f-252">AD FS では HTTPS が使用されるため、Web 層の VM が含まれているサブネットの NSG 規則で HTTPS 要求が許可されていることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-252">AD FS uses HTTPS, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="adc1f-253">これらの要求は、オンプレミス ネットワークのほか、Web 層、ビジネス層、データ層、プライベート DMZ、パブリック DMZ を含むサブネットや AD FS サーバーを含むサブネットから発信される場合があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-253">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="adc1f-254">AD FS サーバーがインターネットに直接公開されないようにします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-254">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="adc1f-255">AD FS サーバーは、セキュリティ トークンを付与するための完全な権限を持つ、ドメイン参加コンピューターです。</span><span class="sxs-lookup"><span data-stu-id="adc1f-255">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="adc1f-256">サーバーが侵害されると、悪意のあるユーザーは、すべての Web アプリケーションのほか、AD FS によって保護されているすべてのフェデレーション サーバーに対するフル アクセス トークンを発行できます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-256">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="adc1f-257">信頼できるパートナー サイトから接続されていない外部ユーザーからの要求をシステムで処理する必要がある場合は、WAP サーバーを使用してそれらの要求を処理します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-257">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="adc1f-258">詳細については、「[フェデレーション サーバー プロキシを配置する場所][where-to-place-an-fs-proxy]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-258">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="adc1f-259">AD FS サーバーと WAP サーバーをそれぞれ独自のファイアウォールを持つ別々のサブネットに配置します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-259">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="adc1f-260">NSG 規則を使用してファイアウォール規則を定義できます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-260">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="adc1f-261">すべてのファイアウォールは、ポート 443 (HTTPS) のトラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-261">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="adc1f-262">AD FS サーバーおよび WAP サーバーへの直接的なサインイン アクセスを制限します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-262">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="adc1f-263">DevOps スタッフだけが接続できるようにしてください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-263">Only DevOps staff should be able to connect.</span></span> <span data-ttu-id="adc1f-264">WAP サーバーをドメインに参加させないでください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-264">Do not join the WAP servers to the domain.</span></span>

<span data-ttu-id="adc1f-265">監査のために、仮想ネットワークの境界を越えるトラフィックに関する詳細情報を記録する一連のネットワーク仮想アプライアンスの使用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-265">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="adc1f-266">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="adc1f-266">Deploy the solution</span></span>

<span data-ttu-id="adc1f-267">このアーキテクチャのデプロイについては、[GitHub][github] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-267">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="adc1f-268">デプロイ全体を完了するには最大 2 時間かかる場合があることに注意してください。これには、VPN ゲートウェイの作成、Active Directory と AD FS を構成するスクリプトの実行などの処理が含まれます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-268">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure Active Directory and AD FS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="adc1f-269">前提条件</span><span class="sxs-lookup"><span data-stu-id="adc1f-269">Prerequisites</span></span>

1. <span data-ttu-id="adc1f-270">[GitHub リポジトリ](https://github.com/mspnp/identity-reference-architectures) の zip ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-270">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

1. <span data-ttu-id="adc1f-271">[Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-271">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

1. <span data-ttu-id="adc1f-272">[Azure の構成要素](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-272">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. <span data-ttu-id="adc1f-273">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-273">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="adc1f-274">シミュレートされたオンプレミスのデータセンターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="adc1f-274">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="adc1f-275">GitHub リポジトリの `adfs` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-275">Navigate to the `adfs` folder of the GitHub repository.</span></span>

1. <span data-ttu-id="adc1f-276">`onprem.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-276">Open the `onprem.json` file.</span></span> <span data-ttu-id="adc1f-277">`adminPassword``Password`、および `SafeModeAdminPassword` のインスタンスを検索し、これらのパスワードを更新します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-277">Search for instances of `adminPassword`, `Password`, and `SafeModeAdminPassword` and update the passwords.</span></span>

1. <span data-ttu-id="adc1f-278">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-278">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a><span data-ttu-id="adc1f-279">Azure インフラストラクチャをデプロイする</span><span class="sxs-lookup"><span data-stu-id="adc1f-279">Deploy the Azure infrastructure</span></span>

1. <span data-ttu-id="adc1f-280">`azure.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-280">Open the `azure.json` file.</span></span>  <span data-ttu-id="adc1f-281">`adminPassword` と `Password` のインスタンスを検索し、パスワードの値を追加します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-281">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

1. <span data-ttu-id="adc1f-282">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-282">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a><span data-ttu-id="adc1f-283">AD FS ファームを設定する</span><span class="sxs-lookup"><span data-stu-id="adc1f-283">Set up the AD FS farm</span></span>

1. <span data-ttu-id="adc1f-284">`adfs-farm-first.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-284">Open the `adfs-farm-first.json` file.</span></span>  <span data-ttu-id="adc1f-285">`AdminPassword` を検索し、既定のパスワードを置き換えます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-285">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="adc1f-286">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-286">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. <span data-ttu-id="adc1f-287">`adfs-farm-rest.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-287">Open the `adfs-farm-rest.json` file.</span></span>  <span data-ttu-id="adc1f-288">`AdminPassword` を検索し、既定のパスワードを置き換えます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-288">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="adc1f-289">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-289">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a><span data-ttu-id="adc1f-290">AD FS を構成する (パート 1)</span><span class="sxs-lookup"><span data-stu-id="adc1f-290">Configure AD FS (part 1)</span></span>

1. <span data-ttu-id="adc1f-291">`ra-adfs-jb-vm1` という名前の VM (ジャンプボックス VM) へのリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-291">Open a remote desktop session to the VM named `ra-adfs-jb-vm1`, which is the jumpbox VM.</span></span> <span data-ttu-id="adc1f-292">ユーザー名は `testuser` です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-292">The user name is `testuser`.</span></span>

1. <span data-ttu-id="adc1f-293">ジャンプボックスから、`ra-adfs-proxy-vm1` という名前の VM へのリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-293">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm1`.</span></span> <span data-ttu-id="adc1f-294">プライベート IP アドレスは 10.0.6.4 です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-294">The private IP address is 10.0.6.4.</span></span>

1. <span data-ttu-id="adc1f-295">このリモート デスクトップ セッションから、[PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-) を実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-295">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="adc1f-296">PowerShell で、次のディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-296">In PowerShell, navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="adc1f-297">次のコードをスクリプト ペインに貼り付けて実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-297">Paste the following code into a script pane and run it:</span></span>

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    <span data-ttu-id="adc1f-298">`Get-Credential` プロンプトで、デプロイ パラメーター ファイル内に指定したパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-298">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="adc1f-299">次のコマンドを実行して、[DSC](/powershell/dsc/overview/overview) 構成の進行状況を監視します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-299">Run the following command to monitor the progress of the [DSC](/powershell/dsc/overview/overview) configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="adc1f-300">一貫性に到達するまでに数分かかる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-300">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="adc1f-301">この時間の間に、コマンドからエラーが表示される場合があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-301">During this time, you may see errors from the command.</span></span> <span data-ttu-id="adc1f-302">構成が成功すると、出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-302">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a><span data-ttu-id="adc1f-303">AD FS を構成する (パート 2)</span><span class="sxs-lookup"><span data-stu-id="adc1f-303">Configure AD FS (part 2)</span></span>

1. <span data-ttu-id="adc1f-304">ジャンプボックスから、`ra-adfs-proxy-vm2` という名前の VM へのリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-304">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm2`.</span></span> <span data-ttu-id="adc1f-305">プライベート IP アドレスは 10.0.6.5 です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-305">The private IP address is 10.0.6.5.</span></span>

1. <span data-ttu-id="adc1f-306">このリモート デスクトップ セッションから、[PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-) を実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-306">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="adc1f-307">次のディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-307">Navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="adc1f-308">次をスクリプト ペインに貼り付け、スクリプトを実行します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-308">Past the following in a script pane and run the script:</span></span>

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    <span data-ttu-id="adc1f-309">`Get-Credential` プロンプトで、デプロイ パラメーター ファイル内に指定したパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-309">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="adc1f-310">次のコマンドを実行して、DSC 構成の進行状況を監視します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-310">Run the following command to monitor the progress of the DSC configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="adc1f-311">整合性が取れるまでに数分かかる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-311">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="adc1f-312">この時間の間に、コマンドからエラーが表示される場合があります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-312">During this time, you may see errors from the command.</span></span> <span data-ttu-id="adc1f-313">構成が成功すると、出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-313">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    <span data-ttu-id="adc1f-314">この DSC は失敗することがあります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-314">Sometimes this DSC fails.</span></span> <span data-ttu-id="adc1f-315">状態チェックで `Status=Failure` と `Type=Consistency` が表示されている場合は、手順 4 を再実行してください。</span><span class="sxs-lookup"><span data-stu-id="adc1f-315">If the status check shows `Status=Failure` and `Type=Consistency`, try re-running step 4.</span></span>

### <a name="sign-into-ad-fs"></a><span data-ttu-id="adc1f-316">AD FS にサインインする</span><span class="sxs-lookup"><span data-stu-id="adc1f-316">Sign into AD FS</span></span>

1. <span data-ttu-id="adc1f-317">ジャンプボックスから、`ra-adfs-adfs-vm1` という名前の VM へのリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="adc1f-317">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-adfs-vm1`.</span></span> <span data-ttu-id="adc1f-318">プライベート IP アドレスは 10.0.5.4 です。</span><span class="sxs-lookup"><span data-stu-id="adc1f-318">The private IP address is 10.0.5.4.</span></span>

1. <span data-ttu-id="adc1f-319">「[Enable the Idp-Intiated Sign on page (IdP によって開始されるサインオン ページを有効にする)](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page)」の手順に従って、サインオン ページを有効にします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-319">Follow the steps in [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) to enable the sign-on page.</span></span>

1. <span data-ttu-id="adc1f-320">ジャンプボックスから、`https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` を参照します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-320">From the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span></span> <span data-ttu-id="adc1f-321">このテストでは無視できる証明書の警告を受け取ることがあります。</span><span class="sxs-lookup"><span data-stu-id="adc1f-321">You may receive a certificate warning that you can ignore for this test.</span></span>

1. <span data-ttu-id="adc1f-322">Contoso Corporation サインイン ページが表示されることを確認します。</span><span class="sxs-lookup"><span data-stu-id="adc1f-322">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="adc1f-323">**contoso\testuser** としてサインインします。</span><span class="sxs-lookup"><span data-stu-id="adc1f-323">Sign in as **contoso\testuser**.</span></span>

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
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
