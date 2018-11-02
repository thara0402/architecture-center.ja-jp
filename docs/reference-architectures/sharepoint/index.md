---
title: 高可用性 SharePoint Server 2016 ファームの Azure での実行
description: 高可用性 SharePoint Server 2016 ファームを Azure で設定するための実証済みプラクティス。
author: njray
ms.date: 07/26/2018
ms.openlocfilehash: 1e44c2817a02cda919bfa94e0b8f07b73b35531f
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916500"
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a><span data-ttu-id="51f77-103">高可用性 SharePoint Server 2016 ファームの Azure での実行</span><span class="sxs-lookup"><span data-stu-id="51f77-103">Run a high availability SharePoint Server 2016 farm in Azure</span></span>

<span data-ttu-id="51f77-104">この参照用アーキテクチャは、高可用性 SharePoint Server 2016 ファームを Azure で設定し、MinRole トポロジおよび SQL Server Always On 可用性グループを使用するための一連の実証済みプラクティスを示します。</span><span class="sxs-lookup"><span data-stu-id="51f77-104">This reference architecture shows a set of proven practices for setting up a high availability SharePoint Server 2016 farm on Azure, using MinRole topology and SQL Server Always On availability groups.</span></span> <span data-ttu-id="51f77-105">SharePoint ファームは、インターネットに接続するエンドポイントまたはプレゼンスがない、セキュアな仮想ネットワークにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="51f77-105">The SharePoint farm is deployed in a secured virtual network with no Internet-facing endpoint or presence.</span></span> [<span data-ttu-id="51f77-106">**こちらのソリューションをデプロイしてください**。</span><span class="sxs-lookup"><span data-stu-id="51f77-106">**Deploy this solution**.</span></span>](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

<span data-ttu-id="51f77-107">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="51f77-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="51f77-108">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="51f77-108">Architecture</span></span>

<span data-ttu-id="51f77-109">このアーキテクチャは、「[Run Windows VMs for an N-tier application][windows-n-tier]」(n 層アプリケーションでの Windows VM の実行) で説明したアーキテクチャ上に構築されています。</span><span class="sxs-lookup"><span data-stu-id="51f77-109">This architecture builds on the one shown in [Run Windows VMs for an N-tier application][windows-n-tier].</span></span> <span data-ttu-id="51f77-110">Azure 仮想ネットワーク (VNet) 内に高可用性を備えた SharePoint Server 2016 ファームをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="51f77-110">It deploys a SharePoint Server 2016 farm with high availability inside an Azure virtual network (VNet).</span></span> <span data-ttu-id="51f77-111">このアーキテクチャは、テスト環境または実稼働環境、Office 365 を含む SharePoint ハイブリッド インフラストラクチャに適しており、ディザスター リカバリー シナリオの基礎としても適しています。</span><span class="sxs-lookup"><span data-stu-id="51f77-111">This architecture is suitable for a test or production environment, a SharePoint hybrid infrastructure with Office 365, or as the basis for a disaster recovery scenario.</span></span>

<span data-ttu-id="51f77-112">アーキテクチャは、次のコンポーネントで構成されています。</span><span class="sxs-lookup"><span data-stu-id="51f77-112">The architecture consists of the following components:</span></span>

- <span data-ttu-id="51f77-113">**リソース グループ**。</span><span class="sxs-lookup"><span data-stu-id="51f77-113">**Resource groups**.</span></span> <span data-ttu-id="51f77-114">[リソース グループ][resource-group]は、Azure の関連リソースを保持するコンテナーです。</span><span class="sxs-lookup"><span data-stu-id="51f77-114">A [resource group][resource-group] is a container that holds related Azure resources.</span></span> <span data-ttu-id="51f77-115">1 つのリソース グループが SharePoint サーバーで使用され、もう 1 つのリソース グループが VM とは独立しているインフラストラクチャ コンポーネント (仮想ネットワークやロード バランサーなど) で使用されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-115">One resource group is used for the SharePoint servers, and another resource group is used for infrastructure components that are independent of VMs, such as the virtual network and load balancers.</span></span>

- <span data-ttu-id="51f77-116">**仮想ネットワーク (VNet)**。</span><span class="sxs-lookup"><span data-stu-id="51f77-116">**Virtual network (VNet)**.</span></span> <span data-ttu-id="51f77-117">VM は一意のイントラネット アドレス空間を使用して VNet にデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="51f77-117">The VMs are deployed in a VNet with a unique intranet address space.</span></span> <span data-ttu-id="51f77-118">VNet はさらにサブネットに分割されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-118">The VNet is further subdivided into subnets.</span></span> 

- <span data-ttu-id="51f77-119">**仮想マシン (VM)**。</span><span class="sxs-lookup"><span data-stu-id="51f77-119">**Virtual machines (VMs)**.</span></span> <span data-ttu-id="51f77-120">VM は VNet にデプロイされ、プライベート静的 IP アドレスがすべての VM に割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="51f77-120">The VMs are deployed into the VNet, and private static IP addresses are assigned to all of the VMs.</span></span> <span data-ttu-id="51f77-121">静的 IP アドレスは、IP アドレスのキャッシュや再起動後のアドレスの変更に伴う問題を回避するため、SQL Server と SharePoint Server 2016 を実行する VM に推奨されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-121">Static IP addresses are recommended for the VMs running SQL Server and SharePoint Server 2016, to avoid issues with IP address caching and changes of addresses after a restart.</span></span>

- <span data-ttu-id="51f77-122">**可用性セット**。</span><span class="sxs-lookup"><span data-stu-id="51f77-122">**Availability sets**.</span></span> <span data-ttu-id="51f77-123">各 SharePoint ロールの VM を別の[可用性セット][availability-set]に配置し、ロールごとに少なくとも 2 つの仮想マシン (VM) をプロビジョニングします。</span><span class="sxs-lookup"><span data-stu-id="51f77-123">Place the VMs for each SharePoint role into separate [availability sets][availability-set], and provision at least two virtual machines (VMs) for each role.</span></span> <span data-ttu-id="51f77-124">こうすると VM が高度なサービス レベル アグリーメント (SLA) に対応できるようになります。</span><span class="sxs-lookup"><span data-stu-id="51f77-124">This makes the VMs eligible for a higher service level agreement (SLA).</span></span> 

- <span data-ttu-id="51f77-125">**内部ロード バランサー**: </span><span class="sxs-lookup"><span data-stu-id="51f77-125">**Internal load balancer**.</span></span> <span data-ttu-id="51f77-126">[ロード バランサー][load-balancer]は、SharePoint の要求トラフィックをオンプレミス ネットワークから SharePoint ファームのフロントエンド Web サーバーに均等配置します。</span><span class="sxs-lookup"><span data-stu-id="51f77-126">The [load balancer][load-balancer] distributes SharePoint request traffic from the on-premises network to the front-end web servers of the SharePoint farm.</span></span> 

- <span data-ttu-id="51f77-127">**ネットワーク セキュリティ グループ (NSG)**。</span><span class="sxs-lookup"><span data-stu-id="51f77-127">**Network security groups (NSGs)**.</span></span> <span data-ttu-id="51f77-128">仮想マシンを含むサブネットごとに、[ネットワーク セキュリティ グループ][nsg]が作成されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-128">For each subnet that contains virtual machines, a [network security group][nsg] is created.</span></span> <span data-ttu-id="51f77-129">NSG を使用して、サブネットを分離するために VNet 内のネットワーク トラフィックを制限します。</span><span class="sxs-lookup"><span data-stu-id="51f77-129">Use NSGs to restrict network traffic within the VNet, in order to isolate subnets.</span></span> 

- <span data-ttu-id="51f77-130">**ゲートウェイ**。</span><span class="sxs-lookup"><span data-stu-id="51f77-130">**Gateway**.</span></span> <span data-ttu-id="51f77-131">ゲートウェイによって、オンプレミス ネットワークと Azure 仮想ネットワークの間に接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-131">The gateway provides a connection between your on-premises network and the Azure virtual network.</span></span> <span data-ttu-id="51f77-132">この接続は ExpressRoute またはサイト間 VPN を使用できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-132">Your connection can use ExpressRoute or site-to-site VPN.</span></span> <span data-ttu-id="51f77-133">詳しくは、「[オンプレミス ネットワークの Azure への接続][hybrid-ra]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-133">For more information, see [Connect an on-premises network to Azure][hybrid-ra].</span></span>

- <span data-ttu-id="51f77-134">**Windows Server Active Directory (AD) ドメイン コントローラー**。</span><span class="sxs-lookup"><span data-stu-id="51f77-134">**Windows Server Active Directory (AD) domain controllers**.</span></span> <span data-ttu-id="51f77-135">この参照アーキテクチャは、Windows Server AD ドメイン コントローラーをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="51f77-135">This reference architecture deploys Windows Server AD domain controllers.</span></span> <span data-ttu-id="51f77-136">これらのドメイン コントローラーは Azure VNet で実行し、オンプレミスの Windows Server AD フォレストと信頼関係を保ちます。</span><span class="sxs-lookup"><span data-stu-id="51f77-136">These domain controllers run in the Azure VNet and have a trust relationship with the on-premises Windows Server AD forest.</span></span> <span data-ttu-id="51f77-137">SharePoint ファーム リソースに対するクライアント Web 要求は、ゲートウェイ接続経由でオンプレミス ネットワークに認証のトラフィックを送信する代わりに、VNet 内で認証されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-137">Client web requests for SharePoint farm resources are authenticated in the VNet rather than sending that authentication traffic across the gateway connection to the on-premises network.</span></span> <span data-ttu-id="51f77-138">DNS では、イントラネット A または CNAME レコードが作成されるため、イントラネット ユーザーは SharePoint ファームの名前を内部ロード バランサーのプライベート IP アドレスに解決できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-138">In DNS, intranet A or CNAME records are created so that intranet users can resolve the name of the SharePoint farm to the private IP address of the internal load balancer.</span></span>

  <span data-ttu-id="51f77-139">SharePoint Server 2016 でも、[Azure Active Directory Domain Services](/azure/active-directory-domain-services/) を利用できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-139">SharePoint Server 2016 also supports using [Azure Active Directory Domain Services](/azure/active-directory-domain-services/).</span></span> <span data-ttu-id="51f77-140">Azure AD Domain Services はマネージド ドメイン サービスを提供するので、Azure にドメイン コントローラーをデプロイし、管理する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="51f77-140">Azure AD Domain Services provides managed domain services, so that you don't need to deploy and manage domain controllers in Azure.</span></span>

- <span data-ttu-id="51f77-141">**SQL Server Always On 可用性グループ**。</span><span class="sxs-lookup"><span data-stu-id="51f77-141">**SQL Server Always On Availability Group**.</span></span> <span data-ttu-id="51f77-142">SQL Server データベースの高可用性を実現するには、[SQL Server Always On 可用性グループ][sql-always-on]を推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-142">For high availability of the SQL Server database, we recommend [SQL Server Always On Availability Groups][sql-always-on].</span></span> <span data-ttu-id="51f77-143">2 つの仮想マシンが SQL Server で使用されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-143">Two virtual machines are used for SQL Server.</span></span> <span data-ttu-id="51f77-144">1 つにはプライマリ データベース レプリカが含まれ、もう 1 つにはセカンダリ レプリカが含まれます。</span><span class="sxs-lookup"><span data-stu-id="51f77-144">One contains the primary database replica and the other contains the secondary replica.</span></span> 

- <span data-ttu-id="51f77-145">**マジョリティ ノード VM**。</span><span class="sxs-lookup"><span data-stu-id="51f77-145">**Majority node VM**.</span></span> <span data-ttu-id="51f77-146">この VM を使用すると、フェールオーバー クラスターがクォーラムを確立できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-146">This VM allows the failover cluster to establish quorum.</span></span> <span data-ttu-id="51f77-147">詳しくは、「[フェールオーバー クラスターのクォーラム構成について][sql-quorum]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-147">For more information, see [Understanding Quorum Configurations in a Failover Cluster][sql-quorum].</span></span>

- <span data-ttu-id="51f77-148">**SharePoint サーバー**。</span><span class="sxs-lookup"><span data-stu-id="51f77-148">**SharePoint servers**.</span></span> <span data-ttu-id="51f77-149">SharePoint サーバーは、Web フロントエンド、キャッシュ、アプリケーション、およびロールの検索を実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-149">The SharePoint servers perform the web front-end, caching, application, and search roles.</span></span> 

- <span data-ttu-id="51f77-150">**Jumpbox**。</span><span class="sxs-lookup"><span data-stu-id="51f77-150">**Jumpbox**.</span></span> <span data-ttu-id="51f77-151">[要塞ホスト][bastion-host]とも呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="51f77-151">Also called a [bastion host][bastion-host].</span></span> <span data-ttu-id="51f77-152">これは、管理者が他の VM に接続するために使用するネットワーク上のセキュアな VM です。</span><span class="sxs-lookup"><span data-stu-id="51f77-152">This is a secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="51f77-153">jumpbox の NSG は、セーフ リストにあるパブリック IP アドレスからのリモート トラフィックのみを許可します。</span><span class="sxs-lookup"><span data-stu-id="51f77-153">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="51f77-154">NSG は、リモート デスクトップ (RDP) トラフィックを許可する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-154">The NSG should permit remote desktop (RDP) traffic.</span></span>

## <a name="recommendations"></a><span data-ttu-id="51f77-155">Recommendations</span><span class="sxs-lookup"><span data-stu-id="51f77-155">Recommendations</span></span>

<span data-ttu-id="51f77-156">実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-156">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="51f77-157">これらの推奨事項を開始点として使用してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-157">Use these recommendations as a starting point.</span></span>

### <a name="resource-group-recommendations"></a><span data-ttu-id="51f77-158">リソース グループの推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-158">Resource group recommendations</span></span>

<span data-ttu-id="51f77-159">サーバー ロールごとにリソース グループを分け、グローバル リソースであるインフラストラクチャ コンポーネントにも別のリソース グループを用意することを推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-159">We recommend separating resource groups according to the server role, and having a separate resource group for infrastructure components that are global resources.</span></span> <span data-ttu-id="51f77-160">このアーキテクチャでは、SharePoint リソースが 1 つのグループを形成し、SQL Server と他のユーティリティ アセットがもう 1 つのグループを形成します。</span><span class="sxs-lookup"><span data-stu-id="51f77-160">In this architecture, the SharePoint resources form one group, while the SQL Server and other utility assets form another.</span></span>

### <a name="virtual-network-and-subnet-recommendations"></a><span data-ttu-id="51f77-161">仮想ネットワークとサブネットの推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-161">Virtual network and subnet recommendations</span></span>

<span data-ttu-id="51f77-162">SharePoint ロールごとに 1 つのサブネット、ゲートウェイに 1 つのサブネット、および jumpbox に 1 つのサブネットを使用します。</span><span class="sxs-lookup"><span data-stu-id="51f77-162">Use one subnet for each SharePoint role, plus a subnet for the gateway and one for the jumpbox.</span></span> 

<span data-ttu-id="51f77-163">ゲートウェイ サブネット名は、*GatewaySubnet* にする必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-163">The gateway subnet must be named *GatewaySubnet*.</span></span> <span data-ttu-id="51f77-164">ゲートウェイ サブネットのアドレス空間には、仮想ネットワーク アドレス空間の最後の部分を割り当てます。</span><span class="sxs-lookup"><span data-stu-id="51f77-164">Assign the gateway subnet address space from the last part of the virtual network address space.</span></span> <span data-ttu-id="51f77-165">詳しくは、「[Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra]」(VPN ゲートウェイを使用したオンプレミス ネットワークの Azure への接続) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-165">For more information, see [Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra].</span></span>

### <a name="vm-recommendations"></a><span data-ttu-id="51f77-166">VM の推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-166">VM recommendations</span></span>

<span data-ttu-id="51f77-167">このアーキテクチャでは最小で 44 コアが必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-167">This architecture requires a minimum of 44 cores:</span></span>

- <span data-ttu-id="51f77-168">Standard_DS3_v2 上に 8 つの SharePoint サーバー (それぞれ 4 コア) = 32 コア</span><span class="sxs-lookup"><span data-stu-id="51f77-168">8 SharePoint servers on Standard_DS3_v2 (4 cores each) = 32 cores</span></span>
- <span data-ttu-id="51f77-169">Standard_DS1_v2 上に 2 つの Active Directory ドメイン コントローラー (それぞれ 1 コア) = 2 コア</span><span class="sxs-lookup"><span data-stu-id="51f77-169">2 Active Directory domain controllers on Standard_DS1_v2 (1 core each) = 2 cores</span></span>
- <span data-ttu-id="51f77-170">Standard_DS3_v2 上に 2 つの SQL Server VM = 8 コア</span><span class="sxs-lookup"><span data-stu-id="51f77-170">2 SQL Server VMs on Standard_DS3_v2 = 8 cores</span></span>
- <span data-ttu-id="51f77-171">Standard_DS1_v2 上に 1 つのマジョリティ ノード = 1 コア</span><span class="sxs-lookup"><span data-stu-id="51f77-171">1 majority node on Standard_DS1_v2 = 1 core</span></span>
- <span data-ttu-id="51f77-172">Standard_DS1_v2 上に 1 つの管理サーバー = 1 コア</span><span class="sxs-lookup"><span data-stu-id="51f77-172">1 management server on Standard_DS1_v2 = 1 core</span></span>

<span data-ttu-id="51f77-173">デプロイについて Azure サブスクリプションに十分な VM コア クォータがあることを確認してください。不足している場合はデプロイで障害が発生します。</span><span class="sxs-lookup"><span data-stu-id="51f77-173">Make sure your Azure subscription has enough VM core quota for the deployment, or the deployment will fail.</span></span> <span data-ttu-id="51f77-174">「[Azure サブスクリプションとサービスの制限、クォータ、制約][quotas]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-174">See [Azure subscription and service limits, quotas, and constraints][quotas].</span></span> 

<span data-ttu-id="51f77-175">Search インデクサーを除くすべての SharePoint ロールについて、[Standard_DS3_v2][vm-sizes-general] VM サイズの使用を推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-175">For all SharePoint roles except the Search Indexer, we recommended using the [Standard_DS3_v2][vm-sizes-general] VM size.</span></span> <span data-ttu-id="51f77-176">Search インデクサーは少なくとも [Standard_DS13_v2][vm-sizes-memory] のサイズにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-176">The Search Indexer should be at least the [Standard_DS13_v2][vm-sizes-memory] size.</span></span> <span data-ttu-id="51f77-177">テストの場合、この参照アーキテクチャのパラメーター ファイルでは、Search インデクサー ロールに対して小さな DS3_v2 のサイズを指定しています。</span><span class="sxs-lookup"><span data-stu-id="51f77-177">For testing, the parameter files for this reference architecture specify the smaller DS3_v2 size for the Search Indexer role.</span></span> <span data-ttu-id="51f77-178">運用環境のデプロイでは、パラメーター ファイルを更新して DS13 以上のサイズを使用してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-178">For a production deployment, update the parameter files to use the DS13 size or larger.</span></span> <span data-ttu-id="51f77-179">詳細については、「[SharePoint Server 2016 のハードウェア要件およびソフトウェア要件][sharepoint-reqs]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-179">For more information, see [Hardware and software requirements for SharePoint Server 2016][sharepoint-reqs].</span></span> 

<span data-ttu-id="51f77-180">SQL Server VM については、少なくとも 4 つのコアと 8 GB の RAM を推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-180">For the SQL Server VMs, we recommend a minimum of 4 cores and 8 GB RAM.</span></span> <span data-ttu-id="51f77-181">この参照アーキテクチャのパラメーター ファイルでは、DS3_v2 のサイズを指定しています。</span><span class="sxs-lookup"><span data-stu-id="51f77-181">The parameter files for this reference architecture specify the DS3_v2 size.</span></span> <span data-ttu-id="51f77-182">運用環境のデプロイでは、より大きな VM サイズを指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-182">For a production deployment, you might need to specify a larger VM size.</span></span> <span data-ttu-id="51f77-183">詳細については、「[ストレージおよび SQL Server の容量計画と構成 (SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-183">For more information, see [Storage and SQL Server capacity planning and configuration (SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements).</span></span> 
 
### <a name="nsg-recommendations"></a><span data-ttu-id="51f77-184">NSG の推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-184">NSG recommendations</span></span>

<span data-ttu-id="51f77-185">VM を含むサブネットごとに 1 つの NSG を用意して、サブネットを分離できるようにすることを推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-185">We recommend having one NSG for each subnet that contains VMs, to enable subnet isolation.</span></span> <span data-ttu-id="51f77-186">サブネットの分離を構成する場合は、各サブネットについて許可または拒否するインバウンド トラフィックまたはアウトバウンド トラフィックを定義する NSG ルールを追加します。</span><span class="sxs-lookup"><span data-stu-id="51f77-186">If you want to configure subnet isolation, add NSG rules that define the allowed or denied inbound or outbound traffic for each subnet.</span></span> <span data-ttu-id="51f77-187">詳しくは、「[ネットワーク セキュリティ グループによるネットワーク トラフィックのフィルタリング][virtual-networks-nsg]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-187">For more information, see [Filter network traffic with network security groups][virtual-networks-nsg].</span></span> 

<span data-ttu-id="51f77-188">ゲートウェイには NSG を割り当てないでください。割り当てると、ゲートウェイが機能を停止します。</span><span class="sxs-lookup"><span data-stu-id="51f77-188">Do not assign an NSG to the gateway subnet, or the gateway will stop functioning.</span></span> 

### <a name="storage-recommendations"></a><span data-ttu-id="51f77-189">記憶域の推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-189">Storage recommendations</span></span>

<span data-ttu-id="51f77-190">ファームの VM の記憶域構成は、オンプレミス デプロイで使用される適切なベスト プラクティスと一致する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-190">The storage configuration of the VMs in the farm should match the appropriate best practices used for on-premises deployments.</span></span> <span data-ttu-id="51f77-191">SharePoint サーバーではログ用に別のディスクが必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-191">SharePoint servers should have a separate disk for logs.</span></span> <span data-ttu-id="51f77-192">検索インデックス ロールをホストする SharePoint サーバーでは、検索インデックスを格納するために追加のディスク領域が必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-192">SharePoint servers hosting search index roles require additional disk space for the search index to be stored.</span></span> <span data-ttu-id="51f77-193">SQL Server の場合、標準プラクティスではデータとログを分離します。</span><span class="sxs-lookup"><span data-stu-id="51f77-193">For SQL Server, the standard practice is to separate data and logs.</span></span> <span data-ttu-id="51f77-194">データベース バックアップ ストレージにディスクをさらに追加し、[tempdb][tempdb] 用に別のディスクを使用します。</span><span class="sxs-lookup"><span data-stu-id="51f77-194">Add more disks for database backup storage, and use a separate disk for [tempdb][tempdb].</span></span>

<span data-ttu-id="51f77-195">最高の信頼性を得るには、[Azure Managed Disks][managed-disks] の使用を推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-195">For best reliability, we recommend using [Azure Managed Disks][managed-disks].</span></span> <span data-ttu-id="51f77-196">マネージド ディスクでは可用性セット内の VM のディスクが必ず分離されるため、単一障害点を回避できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-196">Managed disks ensure that the disks for VMs within an availability set are isolated to avoid single points of failure.</span></span> 

> [!NOTE]
> <span data-ttu-id="51f77-197">現在、この参照用アーキテクチャの Resource Manager テンプレートではマネージド ディスクは使用されていません。</span><span class="sxs-lookup"><span data-stu-id="51f77-197">Currently the Resource Manager template for this reference architecture does not use managed disks.</span></span> <span data-ttu-id="51f77-198">マネージド ディスクを使用するようにテンプレートを更新する予定です。</span><span class="sxs-lookup"><span data-stu-id="51f77-198">We are planning to update the template to use managed disks.</span></span>

<span data-ttu-id="51f77-199">SharePoint および SQL Server のすべての VM で Premium マネージド ディスクを使用してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-199">Use Premium managed disks for all SharePoint and SQL Server VMs.</span></span> <span data-ttu-id="51f77-200">マジョリティ ノード サーバー、ドメイン コントローラー、および管理サーバでは、Standard マネージド ディスクを使用できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-200">You can use Standard managed disks for the majority node server, the domain controllers, and the management server.</span></span> 

### <a name="sharepoint-server-recommendations"></a><span data-ttu-id="51f77-201">SharePoint Server の推奨事項</span><span class="sxs-lookup"><span data-stu-id="51f77-201">SharePoint Server recommendations</span></span>

<span data-ttu-id="51f77-202">SharePoint ファームを構成する前に、サービスごとに 1 つの Windows Server Active Directory サービス アカウントがあることを確認します。</span><span class="sxs-lookup"><span data-stu-id="51f77-202">Before configuring the SharePoint farm, make sure you have one Windows Server Active Directory service account per service.</span></span> <span data-ttu-id="51f77-203">このアーキテクチャでは、ロールごとに特権を分離するために、少なくとも次に示すドメインレベル アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-203">For this architecture, you need at a minimum the following domain-level accounts to isolate privilege per role:</span></span>

- <span data-ttu-id="51f77-204">SQL Server サービス アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-204">SQL Server Service account</span></span>
- <span data-ttu-id="51f77-205">セットアップ ユーザー アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-205">Setup User account</span></span>
- <span data-ttu-id="51f77-206">サーバー ファーム アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-206">Server Farm account</span></span>
- <span data-ttu-id="51f77-207">Search Service アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-207">Search Service account</span></span>
- <span data-ttu-id="51f77-208">コンテンツ アクセス アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-208">Content Access account</span></span>
- <span data-ttu-id="51f77-209">Web アプリ プール アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-209">Web App Pool accounts</span></span>
- <span data-ttu-id="51f77-210">サービス アプリ プール アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-210">Service App Pool accounts</span></span>
- <span data-ttu-id="51f77-211">キャッシュ スーパー ユーザー アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-211">Cache Super User account</span></span>
- <span data-ttu-id="51f77-212">キャッシュ スーパー リーダー アカウント</span><span class="sxs-lookup"><span data-stu-id="51f77-212">Cache Super Reader account</span></span>

<span data-ttu-id="51f77-213">最小 200 MB/s のディスク スループットのサポート要件を満たすには、必ず検索アーキテクチャを計画してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-213">To meet the support requirement for disk throughput of 200 MB per second minimum, make sure to plan the Search architecture.</span></span> <span data-ttu-id="51f77-214">「[SharePoint Server 2013 でエンタープライズ検索アーキテクチャを計画する][sharepoint-search]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-214">See [Plan enterprise search architecture in SharePoint Server 2013][sharepoint-search].</span></span> <span data-ttu-id="51f77-215">また、「[Best practices for crawling in SharePoint Server 2016][sharepoint-crawling](SharePoint Server 2016 のクロールのベスト プラクティス)」のガイドラインに従います。</span><span class="sxs-lookup"><span data-stu-id="51f77-215">Also follow the guidelines in [Best practices for crawling in SharePoint Server 2016][sharepoint-crawling].</span></span>

<span data-ttu-id="51f77-216">さらに、検索コンポーネント データを、パフォーマンスに優れている別の記憶域ボリュームまたはパーティションに格納します。</span><span class="sxs-lookup"><span data-stu-id="51f77-216">In addition, store the search component data on a separate storage volume or partition with high performance.</span></span> <span data-ttu-id="51f77-217">負荷を減らしてスループットを向上するには、オブジェクト キャッシュ ユーザー アカウントを構成します。これはこのアーキテクチャで必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-217">To reduce load and improve throughput, configure the object cache user accounts, which are required in this architecture.</span></span> <span data-ttu-id="51f77-218">Windows Server オペレーティング システム ファイル、SharePoint Server 2016 プログラム ファイル、および診断ログは、通常のパフォーマンスの 3 つの記憶域ボリュームまたはパーティションに分割します。</span><span class="sxs-lookup"><span data-stu-id="51f77-218">Split the Windows Server operating system files, the SharePoint Server 2016 program files, and diagnostics logs across three separate storage volumes or partitions with normal performance.</span></span> 

<span data-ttu-id="51f77-219">これらの推奨事項について詳しくは、「[SharePoint Server 2016 での初期展開の管理およびサービス アカウント][sharepoint-accounts]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-219">For more information about these recommendations, see [Initial deployment administrative and service accounts in SharePoint Server 2016][sharepoint-accounts].</span></span>


### <a name="hybrid-workloads"></a><span data-ttu-id="51f77-220">ハイブリッド ワークロード</span><span class="sxs-lookup"><span data-stu-id="51f77-220">Hybrid workloads</span></span>

<span data-ttu-id="51f77-221">この参照用アーキテクチャでは、[SharePoint ハイブリッド環境][sharepoint-hybrid]&mdash;として使用できる SharePoint Server 2016 ファームがデプロイされます。つまり、SharePoint Server 2016 を Office 365 SharePoint Online に拡張しています。</span><span class="sxs-lookup"><span data-stu-id="51f77-221">This reference architecture deploys a SharePoint Server 2016 farm that can be used as a [SharePoint hybrid environment][sharepoint-hybrid] &mdash; that is, extending SharePoint Server 2016 to Office 365 SharePoint Online.</span></span> <span data-ttu-id="51f77-222">Office Online Server がある場合は、「[Office Web Apps and Office Online Server supportability in Azure][office-web-apps](Azure での Office Web Apps および Office Online Server のサポート)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-222">If you have Office Online Server, see [Office Web Apps and Office Online Server supportability in Azure][office-web-apps].</span></span>

<span data-ttu-id="51f77-223">このデプロイにおける既定のサービス アプリケーションは、ハイブリッド ワークロードに対応するように設計されています。</span><span class="sxs-lookup"><span data-stu-id="51f77-223">The default service applications in this deployment are designed to support hybrid workloads.</span></span> <span data-ttu-id="51f77-224">SharePoint インフラストラクチャを変更せずに、SharePoint Server 2016 および Office 365 のすべてのハイブリッド ワークロードをこのファームにデプロイできます。ただし、1 つの例外として、Cloud Hybrid Search Service Application は、既存の検索トポロジをホストするサーバーにデプロイしないでください。</span><span class="sxs-lookup"><span data-stu-id="51f77-224">All SharePoint Server 2016 and Office 365 hybrid workloads can be deployed to this farm without changes to the SharePoint infrastructure, with one exception: The Cloud Hybrid Search Service Application must not be deployed onto servers hosting an existing search topology.</span></span> <span data-ttu-id="51f77-225">したがって、このハイブリッド シナリオに対応するためには、1 つ以上の検索ロール用の VM をファームに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-225">Therefore, one or more search-role-based VMs must be added to the farm to support this hybrid scenario.</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="51f77-226">SQL Server Always On 可用性グループ</span><span class="sxs-lookup"><span data-stu-id="51f77-226">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="51f77-227">SharePoint Server 2016 は Azure SQL Database を使用できないため、このアーキテクチャでは SQL Server 仮想マシンが使用されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-227">This architecture uses SQL Server virtual machines because SharePoint Server 2016 cannot use Azure SQL Database.</span></span> <span data-ttu-id="51f77-228">SQL Server の高可用性をサポートするために、Always On 可用性グループの使用を推奨します。これによって、一緒にフェールオーバーする一連のデータベースが指定され、それらのデータベースの可用性が向上して復旧可能になります。</span><span class="sxs-lookup"><span data-stu-id="51f77-228">To support high availability in SQL Server, we recommend using Always On Availability Groups, which specify a set of databases that fail over together, making them highly-available and recoverable.</span></span> <span data-ttu-id="51f77-229">この参照用アーキテクチャではデータベースはデプロイ時に作成されますが、手動で Always On 可用性グループを有効にして、SharePoint データベースを可用性グループに追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-229">In this reference architecture, the databases are created during deployment, but you must manually enable Always On Availability Groups and add the SharePoint databases to an availability group.</span></span> <span data-ttu-id="51f77-230">詳しくは、[可用性グループの作成と SharePoint データベースの追加][create-availability-group]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-230">For more information, see [Create the availability group and add the SharePoint databases][create-availability-group].</span></span>

<span data-ttu-id="51f77-231">また、リスナー IP アドレスのクラスターへの追加も推奨します。これは、SQL Server 仮想マシンの内部ロード バランサーのプライベート IP アドレスです。</span><span class="sxs-lookup"><span data-stu-id="51f77-231">We also recommend adding a listener IP address to the cluster, which is the private IP address of the internal load balancer for the SQL Server virtual machines.</span></span>

<span data-ttu-id="51f77-232">Azure で実行する SQL Server の推奨 VM サイズやその他のパフォーマンスの推奨事項について詳しくは、「[Performance best practices for SQL Server in Azure Virtual Machines][sql-performance]」(Azure 仮想マシンでの SQL Server のパフォーマンス ベスト プラクティス) をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-232">For recommended VM sizes and other performance recommendations for SQL Server running in Azure, see [Performance best practices for SQL Server in Azure Virtual Machines][sql-performance].</span></span> <span data-ttu-id="51f77-233">「[SharePoint Server 2016 ファーム内の SQL Server のベスト プラクティス][sql-sharepoint-best-practices]」の推奨事項にも従ってください。</span><span class="sxs-lookup"><span data-stu-id="51f77-233">Also follow the recommendations in [Best practices for SQL Server in a SharePoint Server 2016 farm][sql-sharepoint-best-practices].</span></span>

<span data-ttu-id="51f77-234">マジョリティ ノード サーバーはレプリケーション パートナーとは別のコンピューターに配置することを推奨します。</span><span class="sxs-lookup"><span data-stu-id="51f77-234">We recommend that the majority node server reside on a separate computer from the replication partners.</span></span> <span data-ttu-id="51f77-235">このサーバーによって、高度セーフティ モード セッションでセカンダリ パートナー サーバーが、自動フェールオーバーを開始するかどうかを認識できるようになります。</span><span class="sxs-lookup"><span data-stu-id="51f77-235">The server enables the secondary replication partner server in a high-safety mode session to recognize whether to initiate an automatic failover.</span></span> <span data-ttu-id="51f77-236">2 つのパートナーとは異なり、マジョリティ ノード サーバーはデータベースでは使用されず、自動フェールオーバーをサポートします。</span><span class="sxs-lookup"><span data-stu-id="51f77-236">Unlike the two partners, the majority node server doesn't serve the database but rather supports automatic failover.</span></span> 

## <a name="scalability-considerations"></a><span data-ttu-id="51f77-237">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="51f77-237">Scalability considerations</span></span>

<span data-ttu-id="51f77-238">既存のサーバーを拡張するには、VM サイズを変更するだけですみます。</span><span class="sxs-lookup"><span data-stu-id="51f77-238">To scale up the existing servers, simply change the VM size.</span></span> 

<span data-ttu-id="51f77-239">SharePoint Server 2016 の [MinRoles][minroles] 機能により、サーバーのロールに基づいてサーバーをスケールアウトできます。ロールからサーバーを削除することもできます。</span><span class="sxs-lookup"><span data-stu-id="51f77-239">With the [MinRoles][minroles] capability in SharePoint Server 2016, you can scale out servers based on the server's role and also remove servers from a role.</span></span> <span data-ttu-id="51f77-240">サーバーをロールに追加するときは、単独ロールのいずれか、または組み合わされたロールのいずれかを指定できます。</span><span class="sxs-lookup"><span data-stu-id="51f77-240">When you add servers to a role, you can specify any of the single roles or one of the combined roles.</span></span> <span data-ttu-id="51f77-241">ただし、サーバーを検索ロールに追加する場合は、PowerShell を使用して検索トポロジも再構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-241">If you add servers to the Search role, however, you must also reconfigure the search topology using PowerShell.</span></span> <span data-ttu-id="51f77-242">MinRoles を使用してロールを変換することもできます。</span><span class="sxs-lookup"><span data-stu-id="51f77-242">You can also convert roles using MinRoles.</span></span> <span data-ttu-id="51f77-243">詳しくは、「[SharePoint Server 2016 での MinRole サーバー ファームの管理][sharepoint-minrole]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-243">For more information, see [Managing a MinRole Server Farm in SharePoint Server 2016][sharepoint-minrole].</span></span>

<span data-ttu-id="51f77-244">SharePoint Server 2016 では自動スケーリングのための仮想マシン スケール セットの使用はサポートされないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-244">Note that SharePoint Server 2016 doesn't support using virtual machine scale sets for auto-scaling.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="51f77-245">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="51f77-245">Availability considerations</span></span>

<span data-ttu-id="51f77-246">この参照用アーキテクチャでは、Azure リージョン内の高可用性がサポートされます。ロールごとに少なくとも 2 つの VM が可用性セット内にデプロイされているためです。</span><span class="sxs-lookup"><span data-stu-id="51f77-246">This reference architecture supports high availability within an Azure region, because each role has at least two VMs deployed in an availability set.</span></span>

<span data-ttu-id="51f77-247">リージョン障害に対して保護するには、異なる Azure リージョンに別のディザスター リカバリー ファームを作成します。</span><span class="sxs-lookup"><span data-stu-id="51f77-247">To protect against a regional failure, create a separate disaster recovery farm in a different Azure region.</span></span> <span data-ttu-id="51f77-248">目標復旧時間 (RTO) と目標復旧時点 (RPO) によって設定の要件が決まります。</span><span class="sxs-lookup"><span data-stu-id="51f77-248">Your recovery time objectives (RTOs) and recovery point objectives (RPOs) will determine the setup requirements.</span></span> <span data-ttu-id="51f77-249">詳しくは、[SharePoint Server 2016 用のディザスター リカバリー戦略の選択][sharepoint-dr]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-249">For details, see [Choose a disaster recovery strategy for SharePoint 2016][sharepoint-dr].</span></span> <span data-ttu-id="51f77-250">セカンダリ リージョンは、プライマリ リージョンと "*ペアになっているリージョン*" であることが必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-250">The secondary region should be a *paired region* with the primary region.</span></span> <span data-ttu-id="51f77-251">広範囲にわたって障害が発生した場合は、すべてのペアで一方のリージョンの復旧が優先されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-251">In the event of a broad outage, recovery of one region is prioritized out of every pair.</span></span> <span data-ttu-id="51f77-252">詳しくは、「[ビジネス継続性とディザスター リカバリー (BCDR): Azure のペアになっているリージョン][paired-regions]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-252">For more information, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions][paired-regions].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="51f77-253">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="51f77-253">Manageability considerations</span></span>

<span data-ttu-id="51f77-254">サーバー、サーバー ファーム、およびサイトを運用して管理するには、SharePoint の運用の推奨プラクティスに従います。</span><span class="sxs-lookup"><span data-stu-id="51f77-254">To operate and maintain servers, server farms, and sites, follow the recommended practices for SharePoint operations.</span></span> <span data-ttu-id="51f77-255">詳しくは、[SharePoint Server 2016 の運用][sharepoint-ops]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-255">For more information, see [Operations for SharePoint Server 2016][sharepoint-ops].</span></span>

<span data-ttu-id="51f77-256">SharePoint 環境で SQL Server を管理する際に検討するタスクは、データベース アプリケーションで一般的に検討されるタスクとは異なる場合があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-256">The tasks to consider when managing SQL Server in a SharePoint environment may differ from the ones typically considered for a database application.</span></span> <span data-ttu-id="51f77-257">ベスト プラクティスは、すべての SQL データベースを週 1 回完全にバックアップし、さらに毎晩増分バックアップすることです。</span><span class="sxs-lookup"><span data-stu-id="51f77-257">A best practice is to fully back up all SQL databases weekly with incremental nightly backups.</span></span> <span data-ttu-id="51f77-258">トランザクション ログは 15 分間隔でバックアップします。</span><span class="sxs-lookup"><span data-stu-id="51f77-258">Back up transaction logs every 15 minutes.</span></span> <span data-ttu-id="51f77-259">別のプラクティスでは、SQL Server メンテナンス タスクをデータベースに実装し、組み込みの SharePoint メンテナンス タスクは無効にします。</span><span class="sxs-lookup"><span data-stu-id="51f77-259">Another practice is to implement SQL Server maintenance tasks on the databases while disabling the built-in SharePoint ones.</span></span> <span data-ttu-id="51f77-260">詳しくは、「[ストレージおよび SQL Server の容量計画と構成 ][sql-server-capacity-planning]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="51f77-260">For more information, see [Storage and SQL Server capacity planning and configuration][sql-server-capacity-planning].</span></span> 

## <a name="security-considerations"></a><span data-ttu-id="51f77-261">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="51f77-261">Security considerations</span></span>

<span data-ttu-id="51f77-262">SharePoint Server 2016 の実行に使用されるドメインレベル サービス アカウントでは、ドメイン参加や認証のプロセスで Windows Server AD ドメイン コントローラーが必要です。</span><span class="sxs-lookup"><span data-stu-id="51f77-262">The domain-level service accounts used to run SharePoint Server 2016 require Windows Server AD domain controllers for domain-join and authentication processes.</span></span> <span data-ttu-id="51f77-263">Azure Active Directory Domain Services はこの目的には使用できません。</span><span class="sxs-lookup"><span data-stu-id="51f77-263">Azure Active Directory Domain Services can't be used for this purpose.</span></span> <span data-ttu-id="51f77-264">イントラネットに既に配置されている Windows Server AD の ID インフラストラクチャを拡張するために、このアーキテクチャは、既存のオンプレミス Windows Server AD フォレストの 2 つの Windows Server AD レプリカ ドメイン コントローラーを使用します。</span><span class="sxs-lookup"><span data-stu-id="51f77-264">To extend the Windows Server AD identity infrastructure already in place in the intranet, this architecture uses two Windows Server AD replica domain controllers of an existing on-premises Windows Server AD forest.</span></span>

<span data-ttu-id="51f77-265">また、セキュリティ強化を計画するのは常に賢明です。</span><span class="sxs-lookup"><span data-stu-id="51f77-265">In addition, it's always wise to plan for security hardening.</span></span> <span data-ttu-id="51f77-266">他の推奨事項は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="51f77-266">Other recommendations include:</span></span>

- <span data-ttu-id="51f77-267">NSG にルールを追加してサブネットとロールを分離します。</span><span class="sxs-lookup"><span data-stu-id="51f77-267">Add rules to NSGs to isolate subnets and roles.</span></span>
- <span data-ttu-id="51f77-268">VM にパブリック IP アドレスを割り当てないでください。</span><span class="sxs-lookup"><span data-stu-id="51f77-268">Don't assign public IP addresses to VMs.</span></span>
- <span data-ttu-id="51f77-269">侵入の検出とペイロードの分析のためには、内部 Azure ロード バランサーではなくフロントエンド Web サーバーの前での、ネットワーク仮想アプライアンスの使用を検討します。</span><span class="sxs-lookup"><span data-stu-id="51f77-269">For intrusion detection and analysis of payloads, consider using a network virtual appliance in front of the front-end web servers instead of an internal Azure load balancer.</span></span>
- <span data-ttu-id="51f77-270">オプションとして、サーバー間のクリア テキスト トラフィックの暗号化のために IPsec ポリシーを使用します。</span><span class="sxs-lookup"><span data-stu-id="51f77-270">As an option, use IPsec policies for encryption of cleartext traffic between servers.</span></span> <span data-ttu-id="51f77-271">サブネットの分離も実行する場合は、IPsec トラフィックを許可するようにネットワーク セキュリティ グループ ルールを更新します。</span><span class="sxs-lookup"><span data-stu-id="51f77-271">If you are also doing subnet isolation, update your network security group rules to allow IPsec traffic.</span></span>
- <span data-ttu-id="51f77-272">VM 用のマルウェア対策エージェントをインストールします。</span><span class="sxs-lookup"><span data-stu-id="51f77-272">Install anti-malware agents for the VMs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="51f77-273">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="51f77-273">Deploy the solution</span></span>

<span data-ttu-id="51f77-274">このリファレンス アーキテクチャのデプロイについては、[GitHub][github] を参照してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-274">A deployment for this reference architecture is available on [GitHub][github].</span></span> <span data-ttu-id="51f77-275">デプロイ全体が完了するのに数時間かかる場合があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-275">The entire deployment can take several hours to complete.</span></span>

<span data-ttu-id="51f77-276">デプロイによってサブスクリプション内に作成されるリソース グループは、次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="51f77-276">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="51f77-277">ra-onprem-sp2016-rg</span><span class="sxs-lookup"><span data-stu-id="51f77-277">ra-onprem-sp2016-rg</span></span>
- <span data-ttu-id="51f77-278">ra-sp2016-network-rg</span><span class="sxs-lookup"><span data-stu-id="51f77-278">ra-sp2016-network-rg</span></span>

<span data-ttu-id="51f77-279">テンプレート パラメーター ファイルは、これらの名前を参照します。したがって、名前を変更する場合は、それに合わせてパラメーター ファイルも更新します。</span><span class="sxs-lookup"><span data-stu-id="51f77-279">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span> 

<span data-ttu-id="51f77-280">パラメーター ファイルには、ハードコーディングされたパスワードがさまざまな場所に含まれています。</span><span class="sxs-lookup"><span data-stu-id="51f77-280">The parameter files include a hard-coded password in various places.</span></span> <span data-ttu-id="51f77-281">デプロイする前にこれらの値を変更します。</span><span class="sxs-lookup"><span data-stu-id="51f77-281">Change these values before you deploy.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="51f77-282">前提条件</span><span class="sxs-lookup"><span data-stu-id="51f77-282">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a><span data-ttu-id="51f77-283">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="51f77-283">Deploy the solution</span></span> 

1. <span data-ttu-id="51f77-284">シミュレートされたオンプレミス ネットワークをデプロイするには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-284">Run the following command to deploy a simulated on-premises network.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. <span data-ttu-id="51f77-285">Azure VNet および VPN ゲートウェイをデプロイするには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-285">Run the following command to deploy the Azure VNet and the VPN gateway.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. <span data-ttu-id="51f77-286">Jumpbox、AD ドメイン コントローラー、SQL Server VM をデプロイするには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-286">Run the following command to deploy the jumpbox, AD domain controllers, and SQL Server VMs.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. <span data-ttu-id="51f77-287">フェールオーバー クラスターと可用性グループを作成するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-287">Run the following command to create the failover cluster and the availability group.</span></span> 

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy

5. Run the following command to deploy the remaining VMs.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

<span data-ttu-id="51f77-288">この時点では、SQL Server Always On 可用性グループ用の Web フロントエンドからロード バランサーへの TCP 接続を確立できることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="51f77-288">At this point, verify that you can make a TCP conection from the web front end to the load balancer for the SQL Server Always On availability group.</span></span> <span data-ttu-id="51f77-289">そのためには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-289">To do so, perform the following steps:</span></span>

1. <span data-ttu-id="51f77-290">Azure Portal を使用して、`ra-sp2016-network-rg` リソース グループで `ra-sp-jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="51f77-290">Use the Azure portal to find the VM named `ra-sp-jb-vm1` in the `ra-sp2016-network-rg` resource group.</span></span> <span data-ttu-id="51f77-291">これは、Jumpbox VM です。</span><span class="sxs-lookup"><span data-stu-id="51f77-291">This is the jumpbox VM.</span></span>

2. <span data-ttu-id="51f77-292">`Connect` をクリックして、VM に対するリモート デスクトップ セッションを開きます。</span><span class="sxs-lookup"><span data-stu-id="51f77-292">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="51f77-293">`azure1.json` パラメーター ファイルで指定したパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="51f77-293">Use the password that you specified in the `azure1.json` parameter file.</span></span>

3. <span data-ttu-id="51f77-294">リモート デスクトップ セッションから 10.0.5.4 にログインします。</span><span class="sxs-lookup"><span data-stu-id="51f77-294">From the Remote Desktop session, log into 10.0.5.4.</span></span> <span data-ttu-id="51f77-295">これは、`ra-sp-app-vm1` という名前の VM の IP アドレスです。</span><span class="sxs-lookup"><span data-stu-id="51f77-295">This is the IP address of the VM named `ra-sp-app-vm1`.</span></span>

4. <span data-ttu-id="51f77-296">VM で PowerShell コンソールを開き、`Test-NetConnection` コマンドレットを使用して、ロード バランサーに接続できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="51f77-296">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the load balancer.</span></span>

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

<span data-ttu-id="51f77-297">出力は次のようになります。</span><span class="sxs-lookup"><span data-stu-id="51f77-297">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

<span data-ttu-id="51f77-298">失敗した場合は、Azure Portal を使用して、`ra-sp-sql-vm2` という名前の VM を再起動します。</span><span class="sxs-lookup"><span data-stu-id="51f77-298">If it fails, use the Azure Portal to restart the VM named `ra-sp-sql-vm2`.</span></span> <span data-ttu-id="51f77-299">VM を再起動した後、`Test-NetConnection` コマンドを再実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-299">After the VM restarts, run the `Test-NetConnection` command again.</span></span> <span data-ttu-id="51f77-300">接続の正常な確立のため、VM が再起動した後、約 1 分待機しなければならない場合があります。</span><span class="sxs-lookup"><span data-stu-id="51f77-300">You may need to wait about a minute after the VM restarts for the connection to succeed.</span></span> 

<span data-ttu-id="51f77-301">デプロイを次のように完了します。</span><span class="sxs-lookup"><span data-stu-id="51f77-301">Now complete the deployment as follows.</span></span>

1. <span data-ttu-id="51f77-302">SharePoint ファームのプライマリ ノードをデプロイするには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-302">Run the following command to deploy the SharePoint farm primary node.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. <span data-ttu-id="51f77-303">SharePoint キャッシュ、検索、Web をデプロイするには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-303">Run the following command to deploy the SharePoint cache, search, and web.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. <span data-ttu-id="51f77-304">NSG ルールを作成するには、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="51f77-304">Run the following command to create the NSG rules.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a><span data-ttu-id="51f77-305">デプロイの検証</span><span class="sxs-lookup"><span data-stu-id="51f77-305">Validate the deployment</span></span>

1. <span data-ttu-id="51f77-306">[Azure Portal][azure-portal] で、`ra-onprem-sp2016-rg` リソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="51f77-306">In the [Azure portal][azure-portal], navigate to the `ra-onprem-sp2016-rg` resource group.</span></span>

2. <span data-ttu-id="51f77-307">リソースの一覧で、`ra-onpr-u-vm1` という名前の VM リソースを選択します。</span><span class="sxs-lookup"><span data-stu-id="51f77-307">In the list of resources, select the VM resource named `ra-onpr-u-vm1`.</span></span> 

3. <span data-ttu-id="51f77-308">「[仮想マシンへの接続][connect-to-vm]」の説明に従って、VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="51f77-308">Connect to the VM, as described in [Connect to virtual machine][connect-to-vm].</span></span> <span data-ttu-id="51f77-309">ユーザー名は `\onpremuser` です。</span><span class="sxs-lookup"><span data-stu-id="51f77-309">The user name is `\onpremuser`.</span></span>

5.  <span data-ttu-id="51f77-310">VM へのリモート接続が確立されたら、VM をブラウザーで開いて `http://portal.contoso.local` に移動します。</span><span class="sxs-lookup"><span data-stu-id="51f77-310">When the remote connection to the VM is established, open a browser in the VM and navigate to `http://portal.contoso.local`.</span></span>

6.  <span data-ttu-id="51f77-311">**[Windows セキュリティ]** ボックスで、ユーザー名として `contoso.local\testuser` を使用して SharePoint ポータルにログオンします。</span><span class="sxs-lookup"><span data-stu-id="51f77-311">In the **Windows Security** box, log on to the SharePoint portal using `contoso.local\testuser` for the user name.</span></span>

<span data-ttu-id="51f77-312">このログオンは、オンプレミス ネットワークで使用される Fabrikam.com ドメインから SharePoint ポータルで使用される contoso.local ドメインにトンネリングします。</span><span class="sxs-lookup"><span data-stu-id="51f77-312">This logon tunnels from the Fabrikam.com domain used by the on-premises network to the contoso.local domain used by the SharePoint portal.</span></span> <span data-ttu-id="51f77-313">SharePoint サイトが開くと、ルート デモ サイトが表示されます。</span><span class="sxs-lookup"><span data-stu-id="51f77-313">When the SharePoint site opens, you'll see the root demo site.</span></span>

<span data-ttu-id="51f77-314">**_この参照用アーキテクチャの共同作成者_** &mdash; Joe Davies、Bob Fox、Neil Hodgkinson、Paul Stork</span><span class="sxs-lookup"><span data-stu-id="51f77-314">**_Contributors to this reference architecture_** &mdash;  Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork</span></span>

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
