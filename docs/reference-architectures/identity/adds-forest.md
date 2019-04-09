---
title: Azure での AD DS リソース フォレストの作成
titleSuffix: Azure Reference Architectures
description: >-
  Azure で信頼された Active Directory ドメインを作成する方法。

  ガイダンス,vpn gateway,expressroute,ロード バランサー,仮想ネットワーク,active directory
author: telmosampaio
ms.date: 05/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 5fe966f657782b41ec1926d0fd4bb83eb7a3c0fb
ms.sourcegitcommit: 548374a0133f3caed3934fda6a380c76e6eaecea
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/25/2019
ms.locfileid: "58420041"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="e4f03-104">Azure での Active Directory Domain Services (AD DS) リソース フォレストの作成</span><span class="sxs-lookup"><span data-stu-id="e4f03-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="e4f03-105">この参照アーキテクチャは、オンプレミスの AD フォレストでドメインから信頼される別の Active Directory ドメインを Azure に作成する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="e4f03-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> <span data-ttu-id="e4f03-106">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="e4f03-106">[**Deploy this solution**](#deploy-the-solution).</span></span>

![別の Active Directory ドメインを使用するセキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](./images/adds-forest.png)

<span data-ttu-id="e4f03-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="e4f03-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="e4f03-109">Active Directory Domain Services (AD DS) は、階層構造に ID 情報を格納します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="e4f03-110">階層構造の最上位ノードはフォレストと呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="e4f03-111">1 つのフォレストには複数のドメインが含まれ、ドメインには他の種類のオブジェクトが含まれています。</span><span class="sxs-lookup"><span data-stu-id="e4f03-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="e4f03-112">この参照アーキテクチャは、オンプレミス ドメインとの間に一方向の送信の信頼関係がある AD DS を Azure に作成します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="e4f03-113">Azure のフォレストには、オンプレミスに存在しないドメインが含まれています。</span><span class="sxs-lookup"><span data-stu-id="e4f03-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="e4f03-114">信頼関係があるため、オンプレミス ドメインに対して行われたログオンは、別の Azure ドメイン内のリソースにアクセスする場合にも信頼できます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span>

<span data-ttu-id="e4f03-115">このアーキテクチャの一般的な使用例として、クラウドに保持するオブジェクトと ID のセキュリティの分離を維持する場合や、個々のドメインをオンプレミスからクラウドに移行する場合があります。</span><span class="sxs-lookup"><span data-stu-id="e4f03-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="e4f03-116">その他の考慮事項については、「[オンプレミスの Active Directory を Azure と統合するためのソリューションの選択][considerations]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="e4f03-117">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="e4f03-117">Architecture</span></span>

<span data-ttu-id="e4f03-118">このアーキテクチャには次のコンポーネントがあります。</span><span class="sxs-lookup"><span data-stu-id="e4f03-118">The architecture has the following components.</span></span>

- <span data-ttu-id="e4f03-119">**オンプレミス ネットワーク**。</span><span class="sxs-lookup"><span data-stu-id="e4f03-119">**On-premises network**.</span></span> <span data-ttu-id="e4f03-120">オンプレミス ネットワークには、独自の Active Directory フォレストとドメインが含まれています。</span><span class="sxs-lookup"><span data-stu-id="e4f03-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
- <span data-ttu-id="e4f03-121">**Active Directory サーバー**。</span><span class="sxs-lookup"><span data-stu-id="e4f03-121">**Active Directory servers**.</span></span> <span data-ttu-id="e4f03-122">クラウドで VM として実行されているドメイン サービスを実装するドメイン コントローラーです。</span><span class="sxs-lookup"><span data-stu-id="e4f03-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="e4f03-123">これらのサーバーは、1 つ以上のドメインを含むフォレストをオンプレミスとは別にホストします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
- <span data-ttu-id="e4f03-124">**一方向の信頼**。</span><span class="sxs-lookup"><span data-stu-id="e4f03-124">**One-way trust relationship**.</span></span> <span data-ttu-id="e4f03-125">次の図は、Azure のドメインからオンプレミス ドメインへの一方向の信頼の例を示します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="e4f03-126">この関係があると、オンプレミス ユーザーは Azure のドメイン内のリソースにアクセスできますが、逆方向にはアクセスできません。</span><span class="sxs-lookup"><span data-stu-id="e4f03-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="e4f03-127">クラウド ユーザーがオンプレミス リソースへのアクセスも必要な場合は、両方向の信頼を作成することができます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
- <span data-ttu-id="e4f03-128">**Active Directory サブネット**。</span><span class="sxs-lookup"><span data-stu-id="e4f03-128">**Active Directory subnet**.</span></span> <span data-ttu-id="e4f03-129">AD DS サーバーは、個別のサブネットでホストされます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="e4f03-130">ネットワーク セキュリティ グループ (NSG) ルールによって AD DS サーバーが保護され、予期しないソースからのトラフィックに対するファイアウォールが提供されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
- <span data-ttu-id="e4f03-131">**Azure ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="e4f03-131">**Azure gateway**.</span></span> <span data-ttu-id="e4f03-132">Azure ゲートウェイによって、オンプレミス ネットワークと Azure VNet の間に接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="e4f03-133">[VPN 接続][azure-vpn-gateway]または [Azure ExpressRoute][azure-expressroute] を使用できます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="e4f03-134">詳細については、「[Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture]」(Azure における安全なハイブリッド ネットワーク アーキテクチャの実装) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="e4f03-135">Recommendations</span><span class="sxs-lookup"><span data-stu-id="e4f03-135">Recommendations</span></span>

<span data-ttu-id="e4f03-136">Azure での Active Directory の実装に関する具体的な推奨事項については、「[Active Directory Domain Services (AD DS) を Azure に拡張する][adds-extend-domain]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-136">For specific recommendations on implementing Active Directory in Azure, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

### <a name="trust"></a><span data-ttu-id="e4f03-137">[Trust (信頼)]</span><span class="sxs-lookup"><span data-stu-id="e4f03-137">Trust</span></span>

<span data-ttu-id="e4f03-138">オンプレミス ドメインは、クラウドのドメインとは別のフォレストに含まれています。</span><span class="sxs-lookup"><span data-stu-id="e4f03-138">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="e4f03-139">クラウドでオンプレミス ユーザーの認証を有効にするには、Azure のドメインがオンプレミス フォレストのログオン ドメインを信頼する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e4f03-139">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="e4f03-140">同様に、クラウドが外部ユーザーに対してログオン ドメインを提供している場合は、必要に応じてオンプレミス フォレストがクラウド ドメインを信頼します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-140">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="e4f03-141">フォレスト レベルで信頼を確立するには、[フォレストの信頼を作成][creating-forest-trusts]するか、[外部信頼を作成][creating-external-trusts]してドメイン レベルで信頼を確立します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-141">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="e4f03-142">フォレスト レベルの信頼で、2 つのフォレスト内のすべてのドメイン間にリレーションシップが作成されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-142">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="e4f03-143">外部ドメイン レベルの信頼では、2 つの指定したドメイン間にのみリレーションシップが作成されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-143">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="e4f03-144">外部ドメイン レベルの信頼は、異なるフォレストにあるドメイン間にのみ作成することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-144">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="e4f03-145">一方向または双方向の信頼を作成することができます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-145">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

- <span data-ttu-id="e4f03-146">一方向の信頼では、一方のドメインまたはフォレスト (*着信*ドメインまたはフォレストと呼ばれます) のユーザーが、もう一方 (*送信*ドメインまたはフォレスト) に保持されているリソースにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-146">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
- <span data-ttu-id="e4f03-147">双方向の信頼では、いずれのドメインまたはフォレストのユーザーも、もう一方に保持されているリソースにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-147">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="e4f03-148">次の表は、いくつの単純なシナリオについて信頼の構成をまとめた一覧です。</span><span class="sxs-lookup"><span data-stu-id="e4f03-148">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="e4f03-149">シナリオ</span><span class="sxs-lookup"><span data-stu-id="e4f03-149">Scenario</span></span> | <span data-ttu-id="e4f03-150">オンプレミスの信頼</span><span class="sxs-lookup"><span data-stu-id="e4f03-150">On-premises trust</span></span> | <span data-ttu-id="e4f03-151">クラウドの信頼</span><span class="sxs-lookup"><span data-stu-id="e4f03-151">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e4f03-152">オンプレミス ユーザーは、クラウド内のリソースにアクセスする必要がありますが、反対方向のアクセスは不要です。</span><span class="sxs-lookup"><span data-stu-id="e4f03-152">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="e4f03-153">一方向、着信</span><span class="sxs-lookup"><span data-stu-id="e4f03-153">One-way, incoming</span></span> |<span data-ttu-id="e4f03-154">一方向、送信</span><span class="sxs-lookup"><span data-stu-id="e4f03-154">One-way, outgoing</span></span> |
| <span data-ttu-id="e4f03-155">クラウドのユーザーは、オンプレミスにあるリソースにアクセスする必要がありますが、反対方向のアクセスは不要です。</span><span class="sxs-lookup"><span data-stu-id="e4f03-155">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="e4f03-156">一方向、送信</span><span class="sxs-lookup"><span data-stu-id="e4f03-156">One-way, outgoing</span></span> |<span data-ttu-id="e4f03-157">一方向、着信</span><span class="sxs-lookup"><span data-stu-id="e4f03-157">One-way, incoming</span></span> |
| <span data-ttu-id="e4f03-158">クラウドとオンプレミスの両方のユーザーが、クラウドとオンプレミスに保持されているリソースにアクセスする必要があります。</span><span class="sxs-lookup"><span data-stu-id="e4f03-158">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="e4f03-159">双方向、着信と送信</span><span class="sxs-lookup"><span data-stu-id="e4f03-159">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="e4f03-160">双方向、着信と送信</span><span class="sxs-lookup"><span data-stu-id="e4f03-160">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="e4f03-161">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e4f03-161">Scalability considerations</span></span>

<span data-ttu-id="e4f03-162">同じドメインに属するドメイン コントローラーの場合、Active Directory は自動的にスケーリングします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-162">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="e4f03-163">要求は、ドメイン内のすべてのコントローラーに分散されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-163">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="e4f03-164">別のドメイン コントローラーを追加することもできます。追加したドメイン コントローラーはドメインと自動的に同期されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-164">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="e4f03-165">トラフィックをドメイン内のコントローラーに送信するために別のロード バランサーは構成しないでください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-165">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="e4f03-166">すべてのドメイン コントローラーに、ドメイン データベースを処理できる十分なメモリとストレージ リソースを確保します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-166">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="e4f03-167">すべてのドメイン コントローラーの VM を同じサイズにします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-167">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="e4f03-168">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e4f03-168">Availability considerations</span></span>

<span data-ttu-id="e4f03-169">各ドメインに少なくとも 2 つのドメイン コントローラーをプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-169">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="e4f03-170">こうすることで、サーバー間の自動レプリケーションが可能になります。</span><span class="sxs-lookup"><span data-stu-id="e4f03-170">This enables automatic replication between servers.</span></span> <span data-ttu-id="e4f03-171">各ドメインを処理する Active Directory サーバーとして動作する VM に可用性セットを作成します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-171">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="e4f03-172">この可用性セットに少なくとも 2 つのサーバーを配置します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-172">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="e4f03-173">また、Flexible Single Master Operations (FSMO) ロールとして動作するサーバーへの接続が失敗した場合に備えて、各ドメインの 1 つ以上のサーバーを[スタンバイ操作マスター][standby-operations-masters]に指定することを検討します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-173">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="e4f03-174">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e4f03-174">Manageability considerations</span></span>

<span data-ttu-id="e4f03-175">管理と監視の考慮事項については、[Active Directory の Azure への拡張][adds-extend-domain]に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-175">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span>

<span data-ttu-id="e4f03-176">その他の情報については、「[Monitoring Active Directory][monitoring_ad]」(Active Directory の監視) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-176">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="e4f03-177">[Microsoft Systems Center][microsoft_systems_center] などのツールを監視サーバーにインストールして、これらのタスクを実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-177">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="e4f03-178">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e4f03-178">Security considerations</span></span>

<span data-ttu-id="e4f03-179">フォレスト レベルの信頼は推移的です。</span><span class="sxs-lookup"><span data-stu-id="e4f03-179">Forest level trusts are transitive.</span></span> <span data-ttu-id="e4f03-180">オンプレミスのフォレストとクラウドのフォレスト間にフォレスト レベルの信頼を確立する場合、この信頼は、いずれかのフォレストで作成された他の新しいドメインにも拡張されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-180">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="e4f03-181">セキュリティ上の目的で分離のために複数のドメインを使用する場合は、ドメイン レベルでのみ信頼を作成することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-181">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="e4f03-182">ドメイン レベルの信頼は非推移的です。</span><span class="sxs-lookup"><span data-stu-id="e4f03-182">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="e4f03-183">Active Directory 固有のセキュリティの考慮事項については、[Active Directory の Azure への拡張][adds-extend-domain]に関するページのセキュリティに関する考慮事項を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-183">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="e4f03-184">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="e4f03-184">Deploy the solution</span></span>

<span data-ttu-id="e4f03-185">このアーキテクチャのデプロイについては、[GitHub][github] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="e4f03-185">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="e4f03-186">デプロイ全体を完了するには最大 2 時間かかる場合があることに注意してください。これには、VPN ゲートウェイの作成、AD DS を構成するスクリプトの実行などの処理が含まれます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-186">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure AD DS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e4f03-187">前提条件</span><span class="sxs-lookup"><span data-stu-id="e4f03-187">Prerequisites</span></span>

1. <span data-ttu-id="e4f03-188">[GitHub リポジトリ](https://github.com/mspnp/identity-reference-architectures) の zip ファイルを複製、フォーク、またはダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-188">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

2. <span data-ttu-id="e4f03-189">[Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest) をインストールします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-189">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

3. <span data-ttu-id="e4f03-190">[Azure の構成要素](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm パッケージをインストールします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-190">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="e4f03-191">コマンド プロンプト、bash プロンプト、または PowerShell プロンプトから、次のように Azure アカウントにサインインします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-191">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="e4f03-192">シミュレートされたオンプレミスのデータセンターをデプロイする</span><span class="sxs-lookup"><span data-stu-id="e4f03-192">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="e4f03-193">GitHub リポジトリの `identity/adds-forest` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-193">Navigate to the `identity/adds-forest` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="e4f03-194">`onprem.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-194">Open the `onprem.json` file.</span></span> <span data-ttu-id="e4f03-195">`adminPassword` と `Password` のインスタンスを検索し、パスワードの値を追加します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-195">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

3. <span data-ttu-id="e4f03-196">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-196">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a><span data-ttu-id="e4f03-197">Azure VNet をデプロイする</span><span class="sxs-lookup"><span data-stu-id="e4f03-197">Deploy the Azure VNet</span></span>

1. <span data-ttu-id="e4f03-198">`azure.json` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-198">Open the `azure.json` file.</span></span> <span data-ttu-id="e4f03-199">`adminPassword` と `Password` のインスタンスを検索し、パスワードの値を追加します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-199">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

2. <span data-ttu-id="e4f03-200">同じファイルで `sharedKey` のインスタンスを検索し、VPN 接続の共有キーを入力します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-200">In the same file, search for instances of `sharedKey` and enter shared keys for the VPN connection.</span></span>

    ```json
    "sharedKey": "",
    ```

3. <span data-ttu-id="e4f03-201">次のコマンドを実行し、デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-201">Run the following command and wait for the deployment to finish.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

   <span data-ttu-id="e4f03-202">オンプレミスの VNet と同じリソース グループにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-202">Deploy to the same resource group as the on-premises VNet.</span></span>

### <a name="test-the-ad-trust-relation"></a><span data-ttu-id="e4f03-203">AD の信頼関係をテストする</span><span class="sxs-lookup"><span data-stu-id="e4f03-203">Test the AD trust relation</span></span>

1. <span data-ttu-id="e4f03-204">Azure Portal を使用して、作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-204">Use the Azure portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="e4f03-205">Azure Portal を使用して、`ra-adt-mgmt-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-205">Use the Azure portal to find the VM named `ra-adt-mgmt-vm1`.</span></span>

3. <span data-ttu-id="e4f03-206">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-206">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="e4f03-207">ユーザー名は `contoso\testuser` で、パスワードは、`onprem.json` パラメーター ファイルで指定したものを使用します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-207">The username is `contoso\testuser`, and the password is the one that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="e4f03-208">リモート デスクトップ セッション内から、192.168.0.4 への別のリモート デスクトップ セッションを開きます。これは、`ra-adtrust-onpremise-ad-vm1` という名前の VM の IP アドレスです。</span><span class="sxs-lookup"><span data-stu-id="e4f03-208">From inside your remote desktop session, open another remote desktop session to 192.168.0.4, which is the IP address of the VM named `ra-adtrust-onpremise-ad-vm1`.</span></span> <span data-ttu-id="e4f03-209">ユーザー名は `contoso\testuser` で、パスワードは、`azure.json` パラメーター ファイルで指定したものを使用します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-209">The username is `contoso\testuser`, and the password is the one that you specified in the `azure.json` parameter file.</span></span>

5. <span data-ttu-id="e4f03-210">`ra-adtrust-onpremise-ad-vm1` のリモート デスクトップ セッション内から、**サーバー マネージャー**に移動し、**[ツール]** > **[Active Directory ドメインと信頼関係]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="e4f03-210">From inside the remote desktop session for `ra-adtrust-onpremise-ad-vm1`, go to **Server Manager** and click **Tools** > **Active Directory Domains and Trusts**.</span></span>

6. <span data-ttu-id="e4f03-211">左側のウィンドウで contoso.com を右クリックし、**[プロパティ]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-211">In the left pane, right-click on the contoso.com and select **Properties**.</span></span>

7. <span data-ttu-id="e4f03-212">**[信頼]** タブをクリックします。入力方向の信頼として treyresearch.net が表示されます。</span><span class="sxs-lookup"><span data-stu-id="e4f03-212">Click the **Trusts** tab. You should see treyresearch.net listed as an incoming trust.</span></span>

![Active Directory のフォレスト信頼ダイアログのスクリーンショット](./images/ad-forest-trust.png)

## <a name="next-steps"></a><span data-ttu-id="e4f03-214">次の手順</span><span class="sxs-lookup"><span data-stu-id="e4f03-214">Next steps</span></span>

- <span data-ttu-id="e4f03-215">[オンプレミスの AD DS ドメインを Azure に拡張する][adds-extend-domain]場合のベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-215">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
- <span data-ttu-id="e4f03-216">Azure で [AD DS インフラストラクチャを作成する][adfs]場合のベスト プラクティスを学習します。</span><span class="sxs-lookup"><span data-stu-id="e4f03-216">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://microsoft.com/cloud-platform/system-center
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
