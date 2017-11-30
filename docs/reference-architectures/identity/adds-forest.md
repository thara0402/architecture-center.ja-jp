---
title: "Azure での AD DS リソース フォレストの作成"
description: "Azure で信頼された Active Directory ドメインを作成する方法。\nガイダンス,vpn gateway,expressroute,ロード バランサー,仮想ネットワーク,active directory"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: bb7e57af2afacf1faa7679c854bf49217918eba8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="8f45e-104">Azure での Active Directory Domain Services (AD DS) リソース フォレストの作成</span><span class="sxs-lookup"><span data-stu-id="8f45e-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="8f45e-105">この参照アーキテクチャは、オンプレミスの AD フォレストでドメインから信頼される別の Active Directory ドメインを Azure に作成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="8f45e-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> [<span data-ttu-id="8f45e-106">**以下のソリューションをデプロイします**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="8f45e-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="8f45e-107">[![0]][0]</span></span> 

<span data-ttu-id="8f45e-108">*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*</span><span class="sxs-lookup"><span data-stu-id="8f45e-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="8f45e-109">Active Directory Domain Services (AD DS) は、階層構造に ID 情報を格納します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="8f45e-110">階層構造の最上位ノードはフォレストと呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="8f45e-111">1 つのフォレストには複数のドメインが含まれ、ドメインには他の種類のオブジェクトが含まれています。</span><span class="sxs-lookup"><span data-stu-id="8f45e-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="8f45e-112">この参照アーキテクチャは、オンプレミス ドメインとの間に一方向の送信の信頼関係がある AD DS を Azure に作成します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="8f45e-113">Azure のフォレストには、オンプレミスに存在しないドメインが含まれています。</span><span class="sxs-lookup"><span data-stu-id="8f45e-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="8f45e-114">信頼関係があるため、オンプレミス ドメインに対して行われたログオンは、別の Azure ドメイン内のリソースにアクセスする場合にも信頼できます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span> 

<span data-ttu-id="8f45e-115">このアーキテクチャの一般的な使用例として、クラウドに保持するオブジェクトと ID のセキュリティの分離を維持する場合や、個々のドメインをオンプレミスからクラウドに移行する場合があります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span> 

<span data-ttu-id="8f45e-116">その他の考慮事項については、「[Choose a solution for integrating on-premises Active Directory with Azure][considerations]」(オンプレミスの Active Directory と Azure の統合のソリューションを選択する) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="8f45e-117">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="8f45e-117">Architecture</span></span>

<span data-ttu-id="8f45e-118">アーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-118">The architecture has the following components.</span></span>

* <span data-ttu-id="8f45e-119">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-119">**On-premises network**.</span></span> <span data-ttu-id="8f45e-120">オンプレミス ネットワークには、独自の Active Directory フォレストとドメインが含まれています。</span><span class="sxs-lookup"><span data-stu-id="8f45e-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
* <span data-ttu-id="8f45e-121">**Active Directory サーバー**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-121">**Active Directory servers**.</span></span> <span data-ttu-id="8f45e-122">クラウドで VM として実行されているドメイン サービスを実装するドメイン コントローラーです。</span><span class="sxs-lookup"><span data-stu-id="8f45e-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="8f45e-123">これらのサーバーは、1 つ以上のドメインを含むフォレストをオンプレミスとは別にホストします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
* <span data-ttu-id="8f45e-124">**一方向の信頼**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-124">**One-way trust relationship**.</span></span> <span data-ttu-id="8f45e-125">次の図は、Azure のドメインからオンプレミス ドメインへの一方向の信頼の例を示します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="8f45e-126">この関係があると、オンプレミス ユーザーは Azure のドメイン内のリソースにアクセスできますが、逆方向にはアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="8f45e-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="8f45e-127">クラウド ユーザーがオンプレミス リソースへのアクセスも必要な場合は、両方向の信頼を作成することができます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
* <span data-ttu-id="8f45e-128">**Active Directory サブネット**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-128">**Active Directory subnet**.</span></span> <span data-ttu-id="8f45e-129">AD DS サーバーは、個別のサブネットでホストされます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="8f45e-130">ネットワーク セキュリティ グループ (NSG) ルールによって AD DS サーバーが保護され、予期しないソースからのトラフィックに対するファイアウォールが提供されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="8f45e-131">**Azure ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="8f45e-131">**Azure gateway**.</span></span> <span data-ttu-id="8f45e-132">Azure ゲートウェイによって、オンプレミス ネットワークと Azure VNet の間に接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="8f45e-133">[VPN 接続][azure-vpn-gateway]または [Azure ExpressRoute][azure-expressroute] を使用できます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="8f45e-134">詳細については、「[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]」(Azure における安全なハイブリッド ネットワーク アーキテクチャの実装) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="8f45e-135">Recommendations</span><span class="sxs-lookup"><span data-stu-id="8f45e-135">Recommendations</span></span>

<span data-ttu-id="8f45e-136">Azure に Active Directory を実装する場合の具体的な推奨事項については、以下の記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-136">For specific recommendations on implementing Active Directory in Azure, see the following articles:</span></span>

- <span data-ttu-id="8f45e-137">[Active Directory Domain Services (AD DS) を Azure に拡張する][adds-extend-domain]。</span><span class="sxs-lookup"><span data-stu-id="8f45e-137">[Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span> 
- <span data-ttu-id="8f45e-138">[Azure Virtual Machines での Windows Server Active Directory の展開ガイドライン][ad-azure-guidelines]</span><span class="sxs-lookup"><span data-stu-id="8f45e-138">[Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][ad-azure-guidelines].</span></span>

### <a name="trust"></a><span data-ttu-id="8f45e-139">信頼</span><span class="sxs-lookup"><span data-stu-id="8f45e-139">Trust</span></span>

<span data-ttu-id="8f45e-140">オンプレミス ドメインは、クラウドのドメインとは別のフォレストに含まれています。</span><span class="sxs-lookup"><span data-stu-id="8f45e-140">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="8f45e-141">クラウドでオンプレミス ユーザーの認証を有効にするには、Azure のドメインがオンプレミス フォレストのログオン ドメインを信頼する必要があります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-141">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="8f45e-142">同様に、クラウドが外部ユーザーに対してログオン ドメインを提供している場合は、必要に応じてオンプレミス フォレストがクラウド ドメインを信頼します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-142">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="8f45e-143">フォレスト レベルで信頼を確立するには、[フォレストの信頼を作成][creating-forest-trusts]するか、[外部信頼を作成][creating-external-trusts]してドメイン レベルで信頼を確立します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-143">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="8f45e-144">フォレスト レベルの信頼で、2 つのフォレスト内のすべてのドメイン間にリレーションシップが作成されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-144">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="8f45e-145">外部ドメイン レベルの信頼では、2 つの指定したドメイン間にのみリレーションシップが作成されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-145">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="8f45e-146">外部ドメイン レベルの信頼は、異なるフォレストにあるドメイン間にのみ作成することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-146">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="8f45e-147">一方向または双方向の信頼を作成することができます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-147">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

* <span data-ttu-id="8f45e-148">一方向の信頼では、一方のドメインまたはフォレスト (*着信*ドメインまたはフォレストと呼ばれます) のユーザーが、もう一方 (*送信*ドメインまたはフォレスト) に保持されているリソースにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-148">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
* <span data-ttu-id="8f45e-149">双方向の信頼では、いずれのドメインまたはフォレストのユーザーも、もう一方に保持されているリソースにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-149">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="8f45e-150">次の表は、いくつの単純なシナリオについて信頼の構成をまとめた一覧です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-150">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="8f45e-151">シナリオ</span><span class="sxs-lookup"><span data-stu-id="8f45e-151">Scenario</span></span> | <span data-ttu-id="8f45e-152">オンプレミスの信頼</span><span class="sxs-lookup"><span data-stu-id="8f45e-152">On-premises trust</span></span> | <span data-ttu-id="8f45e-153">クラウドの信頼</span><span class="sxs-lookup"><span data-stu-id="8f45e-153">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="8f45e-154">オンプレミス ユーザーは、クラウド内のリソースにアクセスする必要がありますが、反対方向のアクセスは不要です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-154">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="8f45e-155">一方向、着信</span><span class="sxs-lookup"><span data-stu-id="8f45e-155">One-way, incoming</span></span> |<span data-ttu-id="8f45e-156">一方向、送信</span><span class="sxs-lookup"><span data-stu-id="8f45e-156">One-way, outgoing</span></span> |
| <span data-ttu-id="8f45e-157">クラウドのユーザーは、オンプレミスにあるリソースにアクセスする必要がありますが、反対方向のアクセスは不要です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-157">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="8f45e-158">一方向、送信</span><span class="sxs-lookup"><span data-stu-id="8f45e-158">One-way, outgoing</span></span> |<span data-ttu-id="8f45e-159">一方向、着信</span><span class="sxs-lookup"><span data-stu-id="8f45e-159">One-way, incoming</span></span> |
| <span data-ttu-id="8f45e-160">クラウドとオンプレミスの両方のユーザーが、クラウドとオンプレミスに保持されているリソースにアクセスする必要があります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-160">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="8f45e-161">双方向、着信と送信</span><span class="sxs-lookup"><span data-stu-id="8f45e-161">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="8f45e-162">双方向、着信と送信</span><span class="sxs-lookup"><span data-stu-id="8f45e-162">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="8f45e-163">拡張性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="8f45e-163">Scalability considerations</span></span>

<span data-ttu-id="8f45e-164">同じドメインに属するドメイン コントローラーの場合、Active Directory は自動的にスケーリングします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-164">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="8f45e-165">要求は、ドメイン内のすべてのコントローラーに分散されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-165">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="8f45e-166">別のドメイン コントローラーを追加することもできます。追加したドメイン コントローラーはドメインと自動的に同期されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-166">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="8f45e-167">トラフィックをドメイン内のコントローラーに送信するために別のロード バランサーは構成しないでください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-167">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="8f45e-168">すべてのドメイン コントローラーに、ドメイン データベースを処理できる十分なメモリとストレージ リソースを確保します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-168">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="8f45e-169">すべてのドメイン コントローラーの VM を同じサイズにします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-169">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="8f45e-170">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="8f45e-170">Availability considerations</span></span>

<span data-ttu-id="8f45e-171">各ドメインに少なくとも 2 つのドメイン コントローラーをプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-171">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="8f45e-172">こうすることで、サーバー間の自動レプリケーションが可能になります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-172">This enables automatic replication between servers.</span></span> <span data-ttu-id="8f45e-173">各ドメインを処理する Active Directory サーバーとして動作する VM に可用性セットを作成します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-173">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="8f45e-174">この可用性セットに少なくとも 2 つのサーバーを配置します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-174">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="8f45e-175">また、Flexible Single Master Operations (FSMO) ロールとして動作するサーバーへの接続が失敗した場合に備えて、各ドメインの 1 つ以上のサーバーを[スタンバイ操作マスター][standby-operations-masters]に指定することを検討します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-175">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="8f45e-176">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="8f45e-176">Manageability considerations</span></span>

<span data-ttu-id="8f45e-177">管理と監視の考慮事項については、[Active Directory の Azure への拡張][adds-extend-domain]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-177">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span> 
 
<span data-ttu-id="8f45e-178">その他の情報については、「[Monitoring Active Directory][monitoring_ad]」(Active Directory の監視) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-178">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="8f45e-179">[Microsoft Systems Center][microsoft_systems_center] などのツールを監視サーバーにインストールして、これらのタスクを実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-179">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="8f45e-180">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="8f45e-180">Security considerations</span></span>

<span data-ttu-id="8f45e-181">フォレスト レベルの信頼は推移的です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-181">Forest level trusts are transitive.</span></span> <span data-ttu-id="8f45e-182">オンプレミスのフォレストとクラウドのフォレスト間にフォレスト レベルの信頼を確立する場合、この信頼は、いずれかのフォレストで作成された他の新しいドメインにも拡張されます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-182">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="8f45e-183">セキュリティ上の目的で分離のために複数のドメインを使用する場合は、ドメイン レベルでのみ信頼を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-183">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="8f45e-184">ドメイン レベルの信頼は非推移的です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-184">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="8f45e-185">Active Directory 固有のセキュリティの考慮事項については、[Active Directory の Azure への拡張][adds-extend-domain]に関するページのセキュリティに関する考慮事項を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-185">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8f45e-186">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="8f45e-186">Deploy the solution</span></span>

<span data-ttu-id="8f45e-187">この参照アーキテクチャをデプロイするソリューションについては、[Github][github] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-187">A solution is available on [Github][github] to deploy this reference architecture.</span></span> <span data-ttu-id="8f45e-188">このソリューションをデプロイする Powershell スクリプトを実行するには、最新バージョンの Azure CLI が必要です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-188">You will need the latest version of the Azure CLI to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="8f45e-189">参照アーキテクチャをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-189">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="8f45e-190">[Github][github] からローカル コンピューターにソリューション フォルダーをダウンロードまたは複製します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-190">Download or clone the solution folder from [Github][github] to your local machine.</span></span>

2. <span data-ttu-id="8f45e-191">Azure CLI を開き、ローカルのソリューション フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-191">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="8f45e-192">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-192">Run the following command:</span></span>
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    <span data-ttu-id="8f45e-193">`<subscription id>` は、Azure サブスクリプション ID に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="8f45e-193">Replace `<subscription id>` with your Azure subscription ID.</span></span>
   
    <span data-ttu-id="8f45e-194">`<location>` には、Azure リージョン (`eastus` や `westus` など) を指定します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-194">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
   
    <span data-ttu-id="8f45e-195">`<mode>` パラメーターは、デプロイの細分性を制御します。このパラメーターの値は次のいずれかになります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-195">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
   
   * <span data-ttu-id="8f45e-196">`Onpremise`: シミュレートされたオンプレミスの環境をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-196">`Onpremise`: deploys the simulated on-premises environment.</span></span>
   * <span data-ttu-id="8f45e-197">`Infrastructure`: Azure に VNet インフラストラクチャとジャンプ ボックスをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-197">`Infrastructure`: deploys the VNet infrastructure and jump box in Azure.</span></span>
   * <span data-ttu-id="8f45e-198">`CreateVpn`: Azure 仮想ネットワーク ゲートウェイをデプロイして、シミュレートされたオンプレミス ネットワークに接続します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-198">`CreateVpn`: deploys the Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   * <span data-ttu-id="8f45e-199">`AzureADDS`: Azure で Active Directory DS サーバーとして機能する VM をデプロイし、Active Directory をそれらの VM にデプロイして、ドメインをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-199">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and deploys the domain in Azure.</span></span>
   * <span data-ttu-id="8f45e-200">`WebTier`: Web 層の VM とロード バランサーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-200">`WebTier`: deploys the web tier VMs and load balancer.</span></span>
   * <span data-ttu-id="8f45e-201">`Prepare`: 上記のすべてをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-201">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="8f45e-202">**これは、既存のオンプレミス ネットワークがない状況で、テストまたは評価用に前述の完全な参照アーキテクチャをデプロイする場合に推奨されるオプションです。**</span><span class="sxs-lookup"><span data-stu-id="8f45e-202">**This is the recommended option if If you do not have an existing on-premises network but you want to deploy the complete reference architecture described above for testing or evaluation.**</span></span> 
   * <span data-ttu-id="8f45e-203">`Workload`: ビジネスおよびデータ層の VM とロード バランサーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-203">`Workload`: deploys the business and data tier VMs and load balancers.</span></span> <span data-ttu-id="8f45e-204">これらの VM は `Prepare` のデプロイには含まれません。</span><span class="sxs-lookup"><span data-stu-id="8f45e-204">Note that these VMs are not included in the `Prepare` deployment.</span></span>

4. <span data-ttu-id="8f45e-205">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-205">Wait for the deployment to complete.</span></span> <span data-ttu-id="8f45e-206">`Prepare` のデプロイを指定した場合は、数時間かかります。</span><span class="sxs-lookup"><span data-stu-id="8f45e-206">If you are deploying the `Prepare` deployment, it will take several hours.</span></span>
     
5. <span data-ttu-id="8f45e-207">シミュレートしたオンプレミスの構成を使用している場合は、着信の信頼関係を構成します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-207">If you are using the simulated on-premises configuration, configure the incoming trust relationship:</span></span>
   
   1. <span data-ttu-id="8f45e-208">ジャンプ ボックス (*ra-adtrust-security-rg* リソース グループの *ra-adtrust-mgmt-vm1*) に接続します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-208">Connect to the jump box (*ra-adtrust-mgmt-vm1* in the *ra-adtrust-security-rg* resource group).</span></span> <span data-ttu-id="8f45e-209">パスワード *AweS0me@PW* を使用して *testuser* としてログインします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-209">Log in as *testuser* with password *AweS0me@PW*.</span></span>
   2. <span data-ttu-id="8f45e-210">ジャンプ ボックスで、*contoso.com* ドメイン (オンプレミス ドメイン) の最初の VM で RDP セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-210">On the jump box open an RDP session on the first VM in the *contoso.com* domain (the on-premises domain).</span></span> <span data-ttu-id="8f45e-211">この VM の IP アドレスは 192.168.0.4 です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-211">This VM has the IP address 192.168.0.4.</span></span> <span data-ttu-id="8f45e-212">ユーザー名は *contoso\testuser*、パスワードは *AweS0me@PW* です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-212">The username is *contoso\testuser* with password *AweS0me@PW*.</span></span>
   3. <span data-ttu-id="8f45e-213">[incoming-trust.ps1][incoming-trust] スクリプトをダウンロードし、実行して *treyresearch.com* ドメインからの着信信頼を作成します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-213">Download the [incoming-trust.ps1][incoming-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span>

6. <span data-ttu-id="8f45e-214">独自のオンプレミス インフラストラクチャを使用している場合:</span><span class="sxs-lookup"><span data-stu-id="8f45e-214">If you are using your own on-premises infrastructure:</span></span>
   
   1. <span data-ttu-id="8f45e-215">[incoming-trust.ps1][incoming-trust] スクリプトをダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="8f45e-215">Download the [incoming-trust.ps1][incoming-trust] script.</span></span>
   2. <span data-ttu-id="8f45e-216">スクリプトを編集し、`$TrustedDomainName` 変数の値を独自のドメイン名で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="8f45e-216">Edit the script and replace the value of the `$TrustedDomainName` variable with the name of your own domain.</span></span>
   3. <span data-ttu-id="8f45e-217">スクリプトを実行します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-217">Run the script.</span></span>

7. <span data-ttu-id="8f45e-218">ジャンプ ボックスから、*treyresearch.com* ドメイン (クラウドのドメイン) の最初の VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-218">From the jump-box, connect to the first VM in the *treyresearch.com* domain (the domain in the cloud).</span></span> <span data-ttu-id="8f45e-219">この VM の IP アドレスは 10.0.4.4 です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-219">This VM has the IP address 10.0.4.4.</span></span> <span data-ttu-id="8f45e-220">ユーザー名は *treyresearch\testuser*、パスワードは *AweS0me@PW* です。</span><span class="sxs-lookup"><span data-stu-id="8f45e-220">The username is *treyresearch\testuser* with password *AweS0me@PW*.</span></span>

8. <span data-ttu-id="8f45e-221">[outgoing-trust.ps1][outgoing-trust] スクリプトをダウンロードし、実行して *treyresearch.com* ドメインからの着信信頼を作成します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-221">Download the [outgoing-trust.ps1][outgoing-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span> <span data-ttu-id="8f45e-222">独自のオンプレミス コンピューターを使用している場合は、まずスクリプトを編集します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-222">If you are using your own on-premises machines, then edit the script first.</span></span> <span data-ttu-id="8f45e-223">`$TrustedDomainName` 変数を使用するオンプレミス ドメイン名に設定し、`$TrustedDomainDnsIpAddresses` 変数にこのドメインの Active Directory DS サーバーの IP アドレスを指定します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-223">Set the `$TrustedDomainName` variable to the name of your on-premises domain, and specify the IP addresses of the Active Directory DS servers for this domain in the `$TrustedDomainDnsIpAddresses` variable.</span></span>

9. <span data-ttu-id="8f45e-224">前の手順が完了するまで数分待ってから、オンプレミスの VM に接続し、「[信頼を検証する][verify-a-trust]」の記事で説明されている手順を実行して *contoso.com* ドメインと *treyresearch.com* ドメイン間の信頼関係が正しく構成されているかどうかを判断します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-224">Wait a few minutes for the previous steps to complete, then connect to an on-premises VM and perform the steps outlined in the article [Verify a Trust][verify-a-trust] to determine whether the trust relationship between the *contoso.com* and *treyresearch.com* domains is correctly configured.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8f45e-225">次のステップ</span><span class="sxs-lookup"><span data-stu-id="8f45e-225">Next steps</span></span>

* <span data-ttu-id="8f45e-226">[オンプレミスの AD DS ドメインを Azure に拡張する][adds-extend-domain]場合のベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-226">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
* <span data-ttu-id="8f45e-227">Azure で [AD DS インフラストラクチャを作成する][adfs]場合のベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="8f45e-227">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "別の Active Directory ドメインを使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"