---
title: Active Directory Domain Services (AD DS) を Azure に拡張する
titleSuffix: Azure Reference Architectures
description: オンプレミスの Active Directory ドメインを Azure に拡張します。
author: telmosampaio
ms.date: 05/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 67f23ae3676d0fb95ef484fa6dcb7a8bb92e0fa2
ms.sourcegitcommit: 548374a0133f3caed3934fda6a380c76e6eaecea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2019
ms.locfileid: "58420007"
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a><span data-ttu-id="21fd8-103">Active Directory Domain Services (AD DS) を Azure に拡張する</span><span class="sxs-lookup"><span data-stu-id="21fd8-103">Extend Active Directory Domain Services (AD DS) to Azure</span></span>

<span data-ttu-id="21fd8-104">この参照アーキテクチャでは、Active Directory 環境を Azure に拡張し、Active Directory Domain Services (AD DS) を使用して分散認証サービスを提供する方法を示します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-104">This reference architecture shows how to extend your Active Directory environment to Azure to provide distributed authentication services using Active Directory Domain Services (AD DS).</span></span> <span data-ttu-id="21fd8-105">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="21fd8-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](./images/adds-extend-domain.png)

<span data-ttu-id="21fd8-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="21fd8-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="21fd8-108">AD DS は、ユーザー、コンピューター、アプリケーション、またはセキュリティ ドメインに含まれるその他の ID の認証に使用します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-108">AD DS is used to authenticate user, computer, application, or other identities that are included in a security domain.</span></span> <span data-ttu-id="21fd8-109">オンプレミスでホストできますが、アプリケーションがオンプレミスと Azure で部分的にホストされる場合は、Azure でこの機能をレプリケートする方が効率的です。</span><span class="sxs-lookup"><span data-stu-id="21fd8-109">It can be hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, it may be more efficient to replicate this functionality in Azure.</span></span> <span data-ttu-id="21fd8-110">これにより、クラウドからオンプレミスで実行されている AD DS に返される認証要求とローカルの承認要求の送信が原因の待機時間を削減できます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-110">This can reduce the latency caused by sending authentication and local authorization requests from the cloud back to AD DS running on-premises.</span></span>

<span data-ttu-id="21fd8-111">このアーキテクチャは、オンプレミス ネットワークと Azure 仮想ネットワークが VPN または ExpressRoute によって接続されている場合によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-111">This architecture is commonly used when the on-premises network and the Azure virtual network are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="21fd8-112">また、このアーキテクチャは、双方向レプリケーションをサポートします。つまり、オンプレミスまたはクラウドで変更を行うことができ、両方のソースの一貫性が確保されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-112">This architecture also supports bidirectional replication, meaning changes can be made either on-premises or in the cloud, and both sources will be kept consistent.</span></span> <span data-ttu-id="21fd8-113">このアーキテクチャの一般的な用途には、オンプレミスと Azure 間で機能が配布されるハイブリッド アプリケーション、および Active Directory を使用して認証を実行するアプリケーションとサービスがあります。</span><span class="sxs-lookup"><span data-stu-id="21fd8-113">Typical uses for this architecture include hybrid applications in which functionality is distributed between on-premises and Azure, and applications and services that perform authentication using Active Directory.</span></span>

<span data-ttu-id="21fd8-114">その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-114">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="21fd8-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="21fd8-115">Architecture</span></span>

<span data-ttu-id="21fd8-116">このアーキテクチャは、「[DMZ between Azure and the Internet][implementing-a-secure-hybrid-network-architecture-with-internet-access]」(Azure とインターネット間の DMZ) に示すアーキテクチャを拡張します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-116">This architecture extends the architecture shown in [DMZ between Azure and the Internet][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span> <span data-ttu-id="21fd8-117">アーキテクチャに含まれるコンポーネントを次に示します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-117">It has the following components.</span></span>

- <span data-ttu-id="21fd8-118">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="21fd8-118">**On-premises network**.</span></span> <span data-ttu-id="21fd8-119">オンプレミス ネットワークには、オンプレミスにあるコンポーネントの認証と承認を実行できるローカルの Active Directory サーバーが含まれています。</span><span class="sxs-lookup"><span data-stu-id="21fd8-119">The on-premises network includes local Active Directory servers that can perform authentication and authorization for components located on-premises.</span></span>
- <span data-ttu-id="21fd8-120">**Active Directory サーバー**。</span><span class="sxs-lookup"><span data-stu-id="21fd8-120">**Active Directory servers**.</span></span> <span data-ttu-id="21fd8-121">クラウドで VM として実行されているディレクトリ サービス (AD DS) を実装するドメイン コントローラーです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-121">These are domain controllers implementing directory services (AD DS) running as VMs in the cloud.</span></span> <span data-ttu-id="21fd8-122">このようなサーバーは、Azure 仮想ネットワークで実行されるコンポーネントの認証を提供できます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-122">These servers can provide authentication of components running in your Azure virtual network.</span></span>
- <span data-ttu-id="21fd8-123">**Active Directory サブネット**。</span><span class="sxs-lookup"><span data-stu-id="21fd8-123">**Active Directory subnet**.</span></span> <span data-ttu-id="21fd8-124">AD DS サーバーは、個別のサブネットでホストされます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-124">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="21fd8-125">ネットワーク セキュリティ グループ (NSG) ルールによって AD DS サーバーが保護され、予期しないソースからのトラフィックに対するファイアウォールが提供されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-125">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
- <span data-ttu-id="21fd8-126">**Azure ゲートウェイと Active Directory 同期**。</span><span class="sxs-lookup"><span data-stu-id="21fd8-126">**Azure Gateway and Active Directory synchronization**.</span></span> <span data-ttu-id="21fd8-127">Azure ゲートウェイによって、オンプレミス ネットワークと Azure VNet の間に接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-127">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="21fd8-128">[VPN 接続][azure-vpn-gateway]または [Azure ExpressRoute][azure-expressroute] を使用できます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-128">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="21fd8-129">クラウドおよびオンプレミスの Active Directory サーバー間のすべての同期要求はゲートウェイを経由します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-129">All synchronization requests between the Active Directory servers in the cloud and on-premises pass through the gateway.</span></span> <span data-ttu-id="21fd8-130">Azure に渡すオンプレミスのトラフィックのルーティングは、ユーザー定義ルート (UDR) によって処理されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-130">User-defined routes (UDRs) handle routing for on-premises traffic that passes to Azure.</span></span> <span data-ttu-id="21fd8-131">Active Directory サーバーとの間のトラフィックは、このシナリオで使用するネットワーク仮想アプライアンス (NVA) を経由しません。</span><span class="sxs-lookup"><span data-stu-id="21fd8-131">Traffic to and from the Active Directory servers does not pass through the network virtual appliances (NVAs) used in this scenario.</span></span>

<span data-ttu-id="21fd8-132">UDR と NVA の設定について詳しくは、「[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]」(セキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-132">For more information about configuring UDRs and the NVAs, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="21fd8-133">Recommendations</span><span class="sxs-lookup"><span data-stu-id="21fd8-133">Recommendations</span></span>

<span data-ttu-id="21fd8-134">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-134">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="21fd8-135">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-135">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vm-recommendations"></a><span data-ttu-id="21fd8-136">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-136">VM recommendations</span></span>

<span data-ttu-id="21fd8-137">認証要求に必要なボリュームに基づいて、[VM サイズ][vm-windows-sizes]の要件を決定します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-137">Determine your [VM size][vm-windows-sizes] requirements based on the expected volume of authentication requests.</span></span> <span data-ttu-id="21fd8-138">AD DS をオンプレミスでホストしているコンピューターの仕様を始点として使用し、それに合わせて Azure VM サイズを指定します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-138">Use the specifications of the machines hosting AD DS on premises as a starting point, and match them with the Azure VM sizes.</span></span> <span data-ttu-id="21fd8-139">デプロイ後は、使用率を監視し、VM における実際の負荷に基づいてスケールアップまたはスケールダウンします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-139">Once deployed, monitor utilization and scale up or down based on the actual load on the VMs.</span></span> <span data-ttu-id="21fd8-140">AD DS ドメイン コントローラーのサイジングついて詳しくは、「[Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds]」(Active Directory Domain Services のキャパシティ プランニング) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-140">For more information about sizing AD DS domain controllers, see [Capacity Planning for Active Directory Domain Services][capacity-planning-for-adds].</span></span>

<span data-ttu-id="21fd8-141">Active Directory 用の データベース、ログ、および SYSVOL を格納するための個別の仮想データ ディスクを作成します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-141">Create a separate virtual data disk for storing the database, logs, and SYSVOL for Active Directory.</span></span> <span data-ttu-id="21fd8-142">オペレーティング システムと同じディスクにこれらの項目を格納しないでください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-142">Do not store these items on the same disk as the operating system.</span></span> <span data-ttu-id="21fd8-143">既定では、VM に接続されたデータ ディスクにはライト スルー キャッシュが使用されています。</span><span class="sxs-lookup"><span data-stu-id="21fd8-143">Note that by default, data disks that are attached to a VM use write-through caching.</span></span> <span data-ttu-id="21fd8-144">ただし、このキャッシュ形式は AD DS の要件と矛盾する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="21fd8-144">However, this form of caching can conflict with the requirements of AD DS.</span></span> <span data-ttu-id="21fd8-145">そのため、データ ディスクの *[ホスト キャッシュ設定]* を *[なし]* に設定します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-145">For this reason, set the *Host Cache Preference* setting on the data disk to *None*.</span></span>

<span data-ttu-id="21fd8-146">ドメイン コントローラーとして、AD DS を実行する VM を少なくとも 2 つデプロイし、[可用性セット][availability-set]に追加します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-146">Deploy at least two VMs running AD DS as domain controllers and add them to an [availability set][availability-set].</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="21fd8-147">ネットワークの推奨事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-147">Networking recommendations</span></span>

<span data-ttu-id="21fd8-148">ドメイン ネーム サービス (DNS) の完全なサポートを実現するには、静的なプライベート IP アドレスを使用して各 AD DS サーバの VM ネットワーク インターフェイス (NIC) を構成します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-148">Configure the VM network interface (NIC) for each AD DS server with a static private IP address for full domain name service (DNS) support.</span></span> <span data-ttu-id="21fd8-149">詳しくは、「[Azure Portal を使用して仮想マシンのプライベート IP アドレスを構成する][set-a-static-ip-address]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-149">For more information, see [How to set a static private IP address in the Azure portal][set-a-static-ip-address].</span></span>

> [!NOTE]
> <span data-ttu-id="21fd8-150">パブリック IP アドレスを使用して AD DS の VM NIC を構成しないでください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-150">Do not configure the VM NIC for any AD DS with a public IP address.</span></span> <span data-ttu-id="21fd8-151">詳しくは、「[セキュリティに関する考慮事項][security-considerations]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-151">See [Security considerations][security-considerations] for more details.</span></span>
>

<span data-ttu-id="21fd8-152">Active Directory サブネット NSG には、オンプレミスからの受信トラフィックを許可するルールが必要です。</span><span class="sxs-lookup"><span data-stu-id="21fd8-152">The Active Directory subnet NSG requires rules to permit incoming traffic from on-premises.</span></span> <span data-ttu-id="21fd8-153">AD DS で使用されるポートについて詳しくは、「[Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]」(Active Directory と Active Directory Domain Services のポート要件) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-153">For detailed information on the ports used by AD DS, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span> <span data-ttu-id="21fd8-154">また、UDR テーブルは、このアーキテクチャで使用される NVA を使用して AD DS トラフィックをルーティングしません。</span><span class="sxs-lookup"><span data-stu-id="21fd8-154">Also, ensure the UDR tables do not route AD DS traffic through the NVAs used in this architecture.</span></span>

### <a name="active-directory-site"></a><span data-ttu-id="21fd8-155">Active Directory サイト</span><span class="sxs-lookup"><span data-stu-id="21fd8-155">Active Directory site</span></span>

<span data-ttu-id="21fd8-156">AD DS では、サイトは物理的な場所、ネットワーク、またはデバイスの集合を表します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-156">In AD DS, a site represents a physical location, network, or collection of devices.</span></span> <span data-ttu-id="21fd8-157">AD DS サイトは、互いに近くにあり、高速ネットワークによって接続される AD DS オブジェクトをグループ化することによって AD DS データベース レプリケーションを管理するために使用します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-157">AD DS sites are used to manage AD DS database replication by grouping together AD DS objects that are located close to one another and are connected by a high speed network.</span></span> <span data-ttu-id="21fd8-158">AD DS には、サイト間で AD DS データベースをレプリケートするための最適な戦略を選択するロジックが含まれます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-158">AD DS includes logic to select the best strategy for replacating the AD DS database between sites.</span></span>

<span data-ttu-id="21fd8-159">アプリケーション用に定義されたサブネットを含む AD DS サイトを Azure で作成することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-159">We recommend that you create an AD DS site including the subnets defined for your application in Azure.</span></span> <span data-ttu-id="21fd8-160">続いて、オンプレミスの AD DS サイト間のサイト リンクを構成します。AD DS は最も効率的なデータベース レプリケーションを自動的に実行します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-160">Then, configure a site link between your on-premises AD DS sites, and AD DS will automatically perform the most efficient database replication possible.</span></span> <span data-ttu-id="21fd8-161">このデータベース レプリケーションには、初期構成以外の構成が多少必要です。</span><span class="sxs-lookup"><span data-stu-id="21fd8-161">Note that this database replication requires little beyond the initial configuration.</span></span>

### <a name="active-directory-operations-masters"></a><span data-ttu-id="21fd8-162">Active Directory 操作マスター</span><span class="sxs-lookup"><span data-stu-id="21fd8-162">Active Directory operations masters</span></span>

<span data-ttu-id="21fd8-163">操作マスターの役割は、レプリケートされた AD DS データベースのインスタンス間の整合性チェックをサポートする AD DS ドメイン コントローラーに割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-163">The operations masters role can be assigned to AD DS domain controllers to support consistency checking between instances of replicated AD DS databases.</span></span> <span data-ttu-id="21fd8-164">操作マスターの役割は 5 つあります。スキーマ マスター、ドメイン名前付けマスター、相対識別子マスター、プライマリ ドメイン コントローラー マスター エミュレーター、インフラストラクチャ マスターです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-164">There are five operations master roles: schema master, domain naming master, relative identifier master, primary domain controller master emulator, and infrastructure master.</span></span> <span data-ttu-id="21fd8-165">これらの役割について詳しくは、「[What are Operations Masters?][ad-ds-operations-masters]」(操作マスターとは) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-165">For more information about these roles, see [What are Operations Masters?][ad-ds-operations-masters].</span></span>

<span data-ttu-id="21fd8-166">Azure にデプロイされたドメイン コントローラーには操作マスターの役割を割り当てないことを推奨します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-166">We recommend you do not assign operations masters roles to the domain controllers deployed in Azure.</span></span>

### <a name="monitoring"></a><span data-ttu-id="21fd8-167">監視</span><span class="sxs-lookup"><span data-stu-id="21fd8-167">Monitoring</span></span>

<span data-ttu-id="21fd8-168">ドメイン コントローラー VM のリソースおよび AD DS を監視し、問題を迅速に修正するためのプランを作成します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-168">Monitor the resources of the domain controller VMs as well as the AD DS Services and create a plan to quickly correct any problems.</span></span> <span data-ttu-id="21fd8-169">詳しくは、「[Monitoring Active Directory][monitoring_ad]」(Active Directory の監視) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-169">For more information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="21fd8-170">[Microsoft Systems Center][microsoft_systems_center] などのツールを監視サーバーにインストールして (アーキテクチャの図を参照)、これらのタスクを実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-170">You can also install tools such as [Microsoft Systems Center][microsoft_systems_center] on the monitoring server (see the architecture diagram) to help perform these tasks.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="21fd8-171">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-171">Scalability considerations</span></span>

<span data-ttu-id="21fd8-172">AD DS は、拡張性を確保するために設計されています。</span><span class="sxs-lookup"><span data-stu-id="21fd8-172">AD DS is designed for scalability.</span></span> <span data-ttu-id="21fd8-173">要求を AD DS ドメイン コントローラーに送信するようにロード バランサーやトラフィック コントローラーを構成する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="21fd8-173">You don't need to configure a load balancer or traffic controller to direct requests to AD DS domain controllers.</span></span> <span data-ttu-id="21fd8-174">拡張性に関する唯一の考慮事項は、AD DS を実行する、ネットワーク負荷の要件に対応したサイズの VM を構成し、VM 上の負荷を監視し、必要に応じてスケールアップまたはスケールダウンすることです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-174">The only scalability consideration is to configure the VMs running AD DS with the correct size for your network load requirements, monitor the load on the VMs, and scale up or down as necessary.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="21fd8-175">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-175">Availability considerations</span></span>

<span data-ttu-id="21fd8-176">AD DS を実行する VM をデプロイして[可用性セット][availability-set]を作成します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-176">Deploy the VMs running AD DS into an [availability set][availability-set].</span></span> <span data-ttu-id="21fd8-177">また、少なくとも 1 つのサーバー (要件に応じて、可能であればそれ以上の数のサーバー) に[スタンバイ操作マスター][standby-operations-masters]の役割を割り当てることを検討します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-177">Also, consider assigning the role of [standby operations master][standby-operations-masters] to at least one server, and possibly more depending on your requirements.</span></span> <span data-ttu-id="21fd8-178">スタンバイ操作マスターは、フェールオーバー時にプライマリ操作マスター サーバーの代わりに使用可能な操作マスターのアクティブ コピーです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-178">A standby operations master is an active copy of the operations master that can be used in place of the primary operations masters server during fail over.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="21fd8-179">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-179">Manageability considerations</span></span>

<span data-ttu-id="21fd8-180">AD DS の定期的なバックアップを実行します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-180">Perform regular AD DS backups.</span></span> <span data-ttu-id="21fd8-181">定期的なバックアップの代わりに、ドメイン コントローラーの VHD ファイルを単にコピーしないでください。これは、VHD 上の AD DS データベース ファイルをコピーすると、ファイルの状態に不整合が生じる可能性があり、データベースを再起動できなくなるためです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-181">Don't simply copy the VHD files of domain controllers instead of performing regular backups, because the AD DS database file on the VHD may not be in a consistent state when it's copied, making it impossible to restart the database.</span></span>

<span data-ttu-id="21fd8-182">Azure Portal を使用してドメイン コントローラー VM をシャットダウンしないでください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-182">Do not shut down a domain controller VM using Azure portal.</span></span> <span data-ttu-id="21fd8-183">代わりに、ゲスト オペレーティング システムをシャットダウンして再起動します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-183">Instead, shut down and restart from the guest operating system.</span></span> <span data-ttu-id="21fd8-184">Portal を使用してシャットダウンすると、VM の割り当てが解除され、Active Directory リポジトリの `VM-GenerationID` と `invocationID` の両方がリセットされます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-184">Shutting down through the portal causes the VM to be deallocated, which resets both the `VM-GenerationID` and the `invocationID` of the Active Directory repository.</span></span> <span data-ttu-id="21fd8-185">これにより、AD DS 相対識別子 (RID) プールが破棄され、SYSVOL が権限なしとしてマークされます。また、ドメイン コントローラーの再構成が必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="21fd8-185">This discards the AD DS relative identifier (RID) pool and marks SYSVOL as nonauthoritative, and may require reconfiguration of the domain controller.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="21fd8-186">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="21fd8-186">Security considerations</span></span>

<span data-ttu-id="21fd8-187">AD DS サーバーは認証サービスを提供するため、攻撃の最適なターゲットとなります。</span><span class="sxs-lookup"><span data-stu-id="21fd8-187">AD DS servers provide authentication services and are an attractive target for attacks.</span></span> <span data-ttu-id="21fd8-188">サーバーを保護するには、ファイアウォールとして機能する NSG を使用する個別のサブネットに AD DS サーバーを配置して、直接インターネットに接続しないようにします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-188">To secure them, prevent direct Internet connectivity by placing the AD DS servers in a separate subnet with an NSG acting as a firewall.</span></span> <span data-ttu-id="21fd8-189">認証、承認、サーバーの同期に必要なポートを除く、AD DS サーバー上のポートをすべて閉じてください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-189">Close all ports on the AD DS servers except those necessary for authentication, authorization, and server synchronization.</span></span> <span data-ttu-id="21fd8-190">詳しくは、「[Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports]」(Active Directory と Active Directory Domain Services のポート要件) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-190">For more information, see [Active Directory and Active Directory Domain Services Port Requirements][ad-ds-ports].</span></span>

<span data-ttu-id="21fd8-191">「[Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access]」(インターネットへのアクセスが可能なセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する) に示すように、サブネットと NVA のペアを使用して、サーバーの周囲に追加のセキュリティ境界を実装することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-191">Consider implementing an additional security perimeter around servers with a pair of subnets and NVAs, as described in [Implementing a secure hybrid network architecture with Internet access in Azure][implementing-a-secure-hybrid-network-architecture-with-internet-access].</span></span>

<span data-ttu-id="21fd8-192">BitLocker または Azure Disk Encryption を使用して、AD DS データベースをホストするディスクを暗号化します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-192">Use either BitLocker or Azure disk encryption to encrypt the disk hosting the AD DS database.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="21fd8-193">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="21fd8-193">Deploy the solution</span></span>

<span data-ttu-id="21fd8-194">このアーキテクチャのデプロイについては、[GitHub][github] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="21fd8-194">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="21fd8-195">デプロイ全体を完了するには最大 2 時間かかる場合があることに注意してください。これには、VPN ゲートウェイの作成、AD DS を構成するスクリプトの実行などの処理が含まれます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-195">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure AD DS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="21fd8-196">前提条件</span><span class="sxs-lookup"><span data-stu-id="21fd8-196">Prerequisites</span></span>

1. <span data-ttu-id="21fd8-197">[GitHub リポジトリ](https://github.com/mspnp/identity-reference-architectures) の zip ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-197">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

2. <span data-ttu-id="21fd8-198">[Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-198">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

3. <span data-ttu-id="21fd8-199">[Azure の構成要素](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-199">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="21fd8-200">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-200">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="21fd8-201">シミュレートされたオンプレミスのデータセンターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="21fd8-201">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="21fd8-202">GitHub リポジトリの `identity/adds-extend-domain` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-202">Navigate to the `identity/adds-extend-domain` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="21fd8-203">`onprem.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-203">Open the `onprem.json` file.</span></span> <span data-ttu-id="21fd8-204">`adminPassword` と `Password` のインスタンスを検索し、パスワードの値を追加します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-204">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

3. <span data-ttu-id="21fd8-205">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-205">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a><span data-ttu-id="21fd8-206">Azure VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="21fd8-206">Deploy the Azure VNet</span></span>

1. <span data-ttu-id="21fd8-207">`azure.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-207">Open the `azure.json` file.</span></span>  <span data-ttu-id="21fd8-208">`adminPassword` と `Password` のインスタンスを検索し、パスワードの値を追加します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-208">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

2. <span data-ttu-id="21fd8-209">同じファイルで `sharedKey` のインスタンスを検索し、VPN 接続の共有キーを入力します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-209">In the same file, search for instances of `sharedKey` and enter shared keys for the VPN connection.</span></span>

    ```json
    "sharedKey": "",
    ```

3. <span data-ttu-id="21fd8-210">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-210">Run the following command and wait for the deployment to finish.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

   <span data-ttu-id="21fd8-211">オンプレミスの VNet と同じリソース グループにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-211">Deploy to the same resource group as the on-premises VNet.</span></span>

### <a name="test-connectivity-with-the-azure-vnet"></a><span data-ttu-id="21fd8-212">Azure VNet との接続をテストする</span><span class="sxs-lookup"><span data-stu-id="21fd8-212">Test connectivity with the Azure VNet</span></span>

<span data-ttu-id="21fd8-213">デプロイが完了したら、シミュレートされたオンプレミスの環境から Azure VNet への接続をテストできます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-213">After deployment completes, you can test conectivity from the simulated on-premises environment to the Azure VNet.</span></span>

1. <span data-ttu-id="21fd8-214">Azure Portal を使用して、作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-214">Use the Azure portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="21fd8-215">`ra-onpremise-mgmt-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-215">Find the VM named `ra-onpremise-mgmt-vm1`.</span></span>

3. <span data-ttu-id="21fd8-216">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-216">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="21fd8-217">ユーザー名は `contoso\testuser` で、パスワードは、`onprem.json` パラメーター ファイルで指定したものを使用します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-217">The username is `contoso\testuser`, and the password is the one that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="21fd8-218">リモート デスクトップ セッション内から、10.0.4.4 への別のリモート デスクトップ セッションを開きます。これは、`adds-vm1` という名前の VM の IP アドレスです。</span><span class="sxs-lookup"><span data-stu-id="21fd8-218">From inside your remote desktop session, open another remote desktop session to 10.0.4.4, which is the IP address of the VM named `adds-vm1`.</span></span> <span data-ttu-id="21fd8-219">ユーザー名は `contoso\testuser` で、パスワードは、`azure.json` パラメーター ファイルで指定したものを使用します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-219">The username is `contoso\testuser`, and the password is the one that you specified in the `azure.json` parameter file.</span></span>

5. <span data-ttu-id="21fd8-220">`adds-vm1` のリモート デスクトップ セッション内から、**サーバー マネージャー**に移動し、**[Add other servers to manage]\(管理する他のサーバーを追加する)** に移動します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-220">From inside the remote desktop session for `adds-vm1`, go to **Server Manager** and click **Add other servers to manage**.</span></span>

6. <span data-ttu-id="21fd8-221">**[Active Directory]** タブで、**[今すぐ検索]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="21fd8-221">In the **Active Directory** tab, click **Find now**.</span></span> <span data-ttu-id="21fd8-222">AD、AD DS、および Web VM の一覧が表示されます。</span><span class="sxs-lookup"><span data-stu-id="21fd8-222">You should see a list of the AD, AD DS, and Web VMs.</span></span>

   ![[サーバーの追加] ダイアログのスクリーンショット](./images/add-servers-dialog.png)

## <a name="next-steps"></a><span data-ttu-id="21fd8-224">次の手順</span><span class="sxs-lookup"><span data-stu-id="21fd8-224">Next steps</span></span>

- <span data-ttu-id="21fd8-225">Azure で [AD DS リソース フォレストを作成する][adds-resource-forest]ためのベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-225">Learn the best practices for [creating an AD DS resource forest][adds-resource-forest] in Azure.</span></span>
- <span data-ttu-id="21fd8-226">Azure で [Active Directory フェデレーション サービス (AD FS) インフラストラクチャを作成する][adfs]ためのベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="21fd8-226">Learn the best practices for [creating an Active Directory Federation Services (AD FS) infrastructure][adfs] in Azure.</span></span>

<!-- links -->

[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[adds-data-disks]: https://msdn.microsoft.com/library/mt674703.aspx
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[capacity-planning-for-adds]: https://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/download/details.aspx?id=50013
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: /azure/virtual-network/virtual-networks-static-private-ip-arm-pportal
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "Active Directory を使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"
