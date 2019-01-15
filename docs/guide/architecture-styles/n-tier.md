---
title: n 層アーキテクチャのスタイル
titleSuffix: Azure Application Architecture Guide
description: Azure での n 層アーキテクチャのメリット、課題、ベスト プラクティスについて説明します。
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 4e8aae0032d20df05e1b16a47fda4afa720ed0d9
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110307"
---
# <a name="n-tier-architecture-style"></a><span data-ttu-id="b5ca5-103">n 層アーキテクチャのスタイル</span><span class="sxs-lookup"><span data-stu-id="b5ca5-103">N-tier architecture style</span></span>

<span data-ttu-id="b5ca5-104">n 層アーキテクチャは、アプリケーションを**論理レイヤー**と**物理層**に分離します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-104">An N-tier architecture divides an application into **logical layers** and **physical tiers**.</span></span>

![n 層アーキテクチャ スタイルの論理図](./images/n-tier-logical.svg)

<span data-ttu-id="b5ca5-106">レイヤーは役割を切り離し、依存関係を管理する方法です。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-106">Layers are a way to separate responsibilities and manage dependencies.</span></span> <span data-ttu-id="b5ca5-107">各レイヤーには特定の役割があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-107">Each layer has a specific responsibility.</span></span> <span data-ttu-id="b5ca5-108">上位レイヤーは下位レイヤーのサービスを使用できますが、その逆はありません。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-108">A higher layer can use services in a lower layer, but not the other way around.</span></span>

<span data-ttu-id="b5ca5-109">各層は物理的に分離されており、個別のマシンで実行されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-109">Tiers are physically separated, running on separate machines.</span></span> <span data-ttu-id="b5ca5-110">層は、別の層を直接呼び出したり、非同期メッセージング (メッセージ キュー) を使用したりできます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-110">A tier can call to another tier directly, or use asynchronous messaging (message queue).</span></span> <span data-ttu-id="b5ca5-111">各レイヤーはそれぞれの層でホストされることがありますが、それは必須ではありません。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-111">Although each layer might be hosted in its own tier, that's not required.</span></span> <span data-ttu-id="b5ca5-112">複数のレイヤーが同じ層でホストされることもあります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-112">Several layers might be hosted on the same tier.</span></span> <span data-ttu-id="b5ca5-113">層を物理的に分離することで拡張性と回復性が向上しますが、ネットワーク通信が追加されると待機時間も増加します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-113">Physically separating the tiers improves scalability and resiliency, but also adds latency from the additional network communication.</span></span>

<span data-ttu-id="b5ca5-114">従来の 3 層アプリケーションには、プレゼンテーション層、中間層、データベース層があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-114">A traditional three-tier application has a presentation tier, a middle tier, and a database tier.</span></span> <span data-ttu-id="b5ca5-115">中間層はオプションです。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-115">The middle tier is optional.</span></span> <span data-ttu-id="b5ca5-116">複雑なアプリケーションでは、層が 3 層よりも多くなることがあります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-116">More complex applications can have more than three tiers.</span></span> <span data-ttu-id="b5ca5-117">上記の図は、2 つの中間層があり、異なる機能領域をカプセル化するアプリケーションを示しています。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-117">The diagram above shows an application with two middle tiers, encapsulating different areas of functionality.</span></span>

<span data-ttu-id="b5ca5-118">n 層アプリケーションは、**クローズド レイヤー アーキテクチャ**にすることも、**オープン レイヤー アーキテクチャ**にすることもできます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-118">An N-tier application can have a **closed layer architecture** or an **open layer architecture**:</span></span>

- <span data-ttu-id="b5ca5-119">クローズド レイヤー アーキテクチャでは、レイヤーは直下のレイヤーのみを呼び出すことができます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-119">In a closed layer architecture, a layer can only call the next layer immediately down.</span></span>
- <span data-ttu-id="b5ca5-120">オープン アーキテクチャでは、レイヤーは下位のどのレイヤーでも呼び出すことができます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-120">In an open layer architecture, a layer can call any of the layers below it.</span></span>

<span data-ttu-id="b5ca5-121">クローズド レイヤー アーキテクチャでは、レイヤー間の依存関係が制限されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-121">A closed layer architecture limits the dependencies between layers.</span></span> <span data-ttu-id="b5ca5-122">ただし、レイヤーが次のレイヤーに要求をただ渡すだけでも、不要なネットワーク トラフィックが発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-122">However, it might create unnecessary network traffic, if one layer simply passes requests along to the next layer.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="b5ca5-123">このアーキテクチャを使用する状況</span><span class="sxs-lookup"><span data-stu-id="b5ca5-123">When to use this architecture</span></span>

<span data-ttu-id="b5ca5-124">n 層アーキテクチャは通常、サービスとしてのインフラストラクチャ (IaaS) アプリケーションとして実装され、各層は個別の VM セットで実行されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-124">N-tier architectures are typically implemented as infrastructure-as-service (IaaS) applications, with each tier running on a separate set of VMs.</span></span> <span data-ttu-id="b5ca5-125">ただし、n 層アプリケーションは純粋な IaaS である必要はありません。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-125">However, an N-tier application doesn't need to be pure IaaS.</span></span> <span data-ttu-id="b5ca5-126">多くの場合、アーキテクチャの一部、特にキャッシュ、メッセージング、データ ストレージに管理サービスを使用するのが有用です。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-126">Often, it's advantageous to use managed services for some parts of the architecture, particularly caching, messaging, and data storage.</span></span>

<span data-ttu-id="b5ca5-127">n 層アーキテクチャで以下の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-127">Consider an N-tier architecture for:</span></span>

- <span data-ttu-id="b5ca5-128">シンプルな Web アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-128">Simple web applications.</span></span>
- <span data-ttu-id="b5ca5-129">最小限のリファクタリングでオンプレミス アプリケーションを Azure に移行。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-129">Migrating an on-premises application to Azure with minimal refactoring.</span></span>
- <span data-ttu-id="b5ca5-130">オンプレミス アプリケーションとクラウド アプリケーションの統合開発。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-130">Unified development of on-premises and cloud applications.</span></span>

<span data-ttu-id="b5ca5-131">n 層アーキテクチャは従来のオンプレミス アプリケーションで非常によく使用されているため、既存のワークロードを Azure に移行するのにとても適しています。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-131">N-tier architectures are very common in traditional on-premises applications, so it's a natural fit for migrating existing workloads to Azure.</span></span>

## <a name="benefits"></a><span data-ttu-id="b5ca5-132">メリット</span><span class="sxs-lookup"><span data-stu-id="b5ca5-132">Benefits</span></span>

- <span data-ttu-id="b5ca5-133">クラウドとオンプレミス間、またクラウド プラットフォーム間での移植性。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-133">Portability between cloud and on-premises, and between cloud platforms.</span></span>
- <span data-ttu-id="b5ca5-134">ほとんどの開発者が短時間で習得。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-134">Less learning curve for most developers.</span></span>
- <span data-ttu-id="b5ca5-135">従来のアプリケーション モデルからの自然な進化。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-135">Natural evolution from the traditional application model.</span></span>
- <span data-ttu-id="b5ca5-136">異機種混在環境 (Windows と Linux) でも利用可能</span><span class="sxs-lookup"><span data-stu-id="b5ca5-136">Open to heterogeneous environment (Windows/Linux)</span></span>

## <a name="challenges"></a><span data-ttu-id="b5ca5-137">課題</span><span class="sxs-lookup"><span data-stu-id="b5ca5-137">Challenges</span></span>

- <span data-ttu-id="b5ca5-138">データベースで CRUD 操作のみを行う中間層で終わらせるのが簡単なため、有効な作業を行うことなく待機時間が増えます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-138">It's easy to end up with a middle tier that just does CRUD operations on the database, adding extra latency without doing any useful work.</span></span>
- <span data-ttu-id="b5ca5-139">モノリシックな設計により、機能の個別のデプロイが回避されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-139">Monolithic design prevents independent deployment of features.</span></span>
- <span data-ttu-id="b5ca5-140">IaaS アプリケーションの管理は、管理サービスのみを使用するアプリケーションよりも手がかかります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-140">Managing an IaaS application is more work than an application that uses only managed services.</span></span>
- <span data-ttu-id="b5ca5-141">大規模なシステムでネットワークのセキュリティを管理するのは難しいことがあります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-141">It can be difficult to manage network security in a large system.</span></span>

## <a name="best-practices"></a><span data-ttu-id="b5ca5-142">ベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="b5ca5-142">Best practices</span></span>

- <span data-ttu-id="b5ca5-143">自動スケールを使用して負荷の変化に対応する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-143">Use autoscaling to handle changes in load.</span></span> <span data-ttu-id="b5ca5-144">「[自動スケールのベスト プラクティス][autoscaling]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-144">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="b5ca5-145">非同期メッセージングを使用して層を分離する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-145">Use asynchronous messaging to decouple tiers.</span></span>
- <span data-ttu-id="b5ca5-146">半静的なデータをキャッシュする。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-146">Cache semi-static data.</span></span> <span data-ttu-id="b5ca5-147">[キャッシュのベスト プラクティス][caching]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-147">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="b5ca5-148">[SQL Server Always On 可用性グループ][sql-always-on]のようなソリューションを使用して、高可用性のデータベース層を構成する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-148">Configure database tier for high availability, using a solution such as [SQL Server Always On Availability Groups][sql-always-on].</span></span>
- <span data-ttu-id="b5ca5-149">フロント エンドとインターネットの間に Web アプリケーション ファイアウォール (WAF) を配置する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-149">Place a web application firewall (WAF) between the front end and the Internet.</span></span>
- <span data-ttu-id="b5ca5-150">各層をそれぞれのサブネットに配置し、サブネットをセキュリティ境界として使用する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-150">Place each tier in its own subnet, and use subnets as a security boundary.</span></span>
- <span data-ttu-id="b5ca5-151">中間層からの要求のみを許可して、データ層へのアクセスを制限する。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-151">Restrict access to the data tier, by allowing requests only from the middle tier(s).</span></span>

## <a name="n-tier-architecture-on-virtual-machines"></a><span data-ttu-id="b5ca5-152">仮想マシンでの n 層アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="b5ca5-152">N-tier architecture on virtual machines</span></span>

<span data-ttu-id="b5ca5-153">このセクションでは、VM で実行される推奨の n 層アーキテクチャについて説明します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-153">This section describes a recommended N-tier architecture running on VMs.</span></span>

![n 層アーキテクチャの物理図](./images/n-tier-physical.png)

<span data-ttu-id="b5ca5-155">各層は 2 つ以上の VM で構成され、可用性セットまたは VM スケール セットに配置されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-155">Each tier consists of two or more VMs, placed in an availability set or VM scale set.</span></span> <span data-ttu-id="b5ca5-156">VM を複数配置すると、1 つの VM が失敗した場合の回復性があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-156">Multiple VMs provide resiliency in case one VM fails.</span></span> <span data-ttu-id="b5ca5-157">層内の VM 間で要求を分散させるには、ロード バランサーが使用されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-157">Load balancers are used to distribute requests across the VMs in a tier.</span></span> <span data-ttu-id="b5ca5-158">プールに VM を追加することで、層を水平方向にスケールできます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-158">A tier can be scaled horizontally by adding more VMs to the pool.</span></span>

<span data-ttu-id="b5ca5-159">各層はそれぞれのサブネット内にも配置されますが、それは、その内部 IP アドレスが同じアドレス範囲内になることを意味します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-159">Each tier is also placed inside its own subnet, meaning their internal IP addresses fall within the same address range.</span></span> <span data-ttu-id="b5ca5-160">そのため、ネットワーク セキュリティ グループ (NSG) ルールを適用して個別の層にテーブルをルーティングするのが容易になります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-160">That makes it easy to apply network security group (NSG) rules and route tables to individual tiers.</span></span>

<span data-ttu-id="b5ca5-161">Web 層とビジネス層はステートレスです。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-161">The web and business tiers are stateless.</span></span> <span data-ttu-id="b5ca5-162">どの VM も、その層へのあらゆる要求を処理できます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-162">Any VM can handle any request for that tier.</span></span> <span data-ttu-id="b5ca5-163">データ層は、レプリケートされたデータベースで構成されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-163">The data tier should consist of a replicated database.</span></span> <span data-ttu-id="b5ca5-164">Windows の場合は、SQL Server で Always On 可用性グループを使用して高可用性を実現することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-164">For Windows, we recommend SQL Server, using Always On Availability Groups for high availability.</span></span> <span data-ttu-id="b5ca5-165">Linux の場合は、Apache Cassandra などの、レプリケーションをサポートするデータベースを選択します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-165">For Linux, choose a database that supports replication, such as Apache Cassandra.</span></span>

<span data-ttu-id="b5ca5-166">ネットワーク セキュリティ グループ (NSG) により、各層へのアクセスを制限します。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-166">Network Security Groups (NSGs) restrict access to each tier.</span></span> <span data-ttu-id="b5ca5-167">たとえば、データベース層ではビジネス層からのアクセスのみが許可されます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-167">For example, the database tier only allows access from the business tier.</span></span>

<span data-ttu-id="b5ca5-168">詳細について、また配置可能な Resource Manager テンプレートについては、次の参照アーキテクチャをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-168">For more details and a deployable Resource Manager template, see the following reference architectures:</span></span>

- <span data-ttu-id="b5ca5-169">[Run Windows VMs for an N-tier application (n 層アプリケーションの Windows VM を実行する)][n-tier-windows]</span><span class="sxs-lookup"><span data-stu-id="b5ca5-169">[Run Windows VMs for an N-tier application][n-tier-windows]</span></span>
- <span data-ttu-id="b5ca5-170">[Run Linux VMs for an N-tier application (n 層アプリケーションの Linux VM を実行する)][n-tier-linux]</span><span class="sxs-lookup"><span data-stu-id="b5ca5-170">[Run Linux VMs for an N-tier application][n-tier-linux]</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="b5ca5-171">追加の考慮事項</span><span class="sxs-lookup"><span data-stu-id="b5ca5-171">Additional considerations</span></span>

- <span data-ttu-id="b5ca5-172">n 層アーキテクチャは 3 層に制限されているわけではありません。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-172">N-tier architectures are not restricted to three tiers.</span></span> <span data-ttu-id="b5ca5-173">複雑なアプリケーションの場合は、さらに層が増えるのが一般的です。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-173">For more complex applications, it is common to have more tiers.</span></span> <span data-ttu-id="b5ca5-174">その場合は、レイヤー 7 ルーティングを使用して特定の層に要求をルートすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-174">In that case, consider using layer-7 routing to route requests to a particular tier.</span></span>

- <span data-ttu-id="b5ca5-175">層は拡張性、信頼性、セキュリティの境界です。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-175">Tiers are the boundary of scalability, reliability, and security.</span></span> <span data-ttu-id="b5ca5-176">これらの領域で異なる要件を持つ各サービスには、個別の層を使用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-176">Consider having separate tiers for services with different requirements in those areas.</span></span>

- <span data-ttu-id="b5ca5-177">自動スケールには、VM Scale Sets を使用してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-177">Use VM Scale Sets for autoscaling.</span></span>

- <span data-ttu-id="b5ca5-178">大幅にリファクタリングせずに管理サービスを使用できる、アーキテクチャ内の場所を探してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-178">Look for places in the architecture where you can use a managed service without significant refactoring.</span></span> <span data-ttu-id="b5ca5-179">具体的には、キャッシュ、メッセージング、ストレージ、データベースを確認してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-179">In particular, look at caching, messaging, storage, and databases.</span></span>

- <span data-ttu-id="b5ca5-180">セキュリティを高めるために、アプリケーションの前にネットワーク DMZ を配置してください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-180">For higher security, place a network DMZ in front of the application.</span></span> <span data-ttu-id="b5ca5-181">DMZ には、ファイアウォールやパケット検査などのセキュリティ機能を実装する、ネットワーク仮想アプライアンス (NVA) が含まれています。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-181">The DMZ includes network virtual appliances (NVAs) that implement security functionality such as firewalls and packet inspection.</span></span> <span data-ttu-id="b5ca5-182">詳細については、[ネットワーク DMZ 参照アーキテクチャ][dmz]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-182">For more information, see [Network DMZ reference architecture][dmz].</span></span>

- <span data-ttu-id="b5ca5-183">高可用性のために、可用性セットに複数の NVA を配置してください。外部のロード バランサーを使用して、インターネット要求をインスタンス間で分散させてください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-183">For high availability, place two or more NVAs in an availability set, with an external load balancer to distribute Internet requests across the instances.</span></span> <span data-ttu-id="b5ca5-184">詳細については、「[Deploy highly available network virtual appliances (高可用性ネットワーク仮想アプライアンスのデプロイ)][ha-nva]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-184">For more information, see [Deploy highly available network virtual appliances][ha-nva].</span></span>

- <span data-ttu-id="b5ca5-185">アプリケーション コードを実行している VM には、RDP または SSH から直接アクセスできないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-185">Do not allow direct RDP or SSH access to VMs that are running application code.</span></span> <span data-ttu-id="b5ca5-186">その代わりに、オペレーターは要塞ホストとも呼ばれる JumpBox にログインする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-186">Instead, operators should log into a jumpbox, also called a bastion host.</span></span> <span data-ttu-id="b5ca5-187">これは、管理者が他の VM への接続に使用するネットワーク上の VM です。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-187">This is a VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="b5ca5-188">JumpBox には、承認されたパブリック IP アドレスからの RDP または SSH のみを許可する NSG があります。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-188">The jumpbox has an NSG that allows RDP or SSH only from approved public IP addresses.</span></span>

- <span data-ttu-id="b5ca5-189">サイト間の仮想プライベート ネットワーク (VPN) または Azure ExpressRoute を使用して、Azure 仮想ネットワークをお客様のオンプレミス ネットワークに拡張できます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-189">You can extend the Azure virtual network to your on-premises network using a site-to-site virtual private network (VPN) or Azure ExpressRoute.</span></span> <span data-ttu-id="b5ca5-190">詳細については、[ハイブリッド ネットワーク 参照アーキテクチャ][hybrid-network]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-190">For more information, see [Hybrid network reference architecture][hybrid-network].</span></span>

- <span data-ttu-id="b5ca5-191">お客様の組織が Active Directory を使用して ID を管理している場合、Active Directory 環境を Azure VNet に拡張できます。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-191">If your organization uses Active Directory to manage identity, you may want to extend your Active Directory environment to the Azure VNet.</span></span> <span data-ttu-id="b5ca5-192">詳細については、[ID 管理参照アーキテクチャ][identity]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-192">For more information, see [Identity management reference architecture][identity].</span></span>

- <span data-ttu-id="b5ca5-193">VM の Azure SLA で提供されるよりも高い可用性が必要な場合は、アプリケーションを 2 つのリージョンにレプリケートして、Azure Traffic Manager を使用してフェールオーバーを行います。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-193">If you need higher availability than the Azure SLA for VMs provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="b5ca5-194">詳細については、[複数リージョンでの Windows VM の実行][ multiregion-windows]、または[複数リージョンでの Linux VM の実行][multiregion-linux]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b5ca5-194">For more information, see [Run Windows VMs in multiple regions][multiregion-windows] or [Run Linux VMs in multiple regions][multiregion-linux].</span></span>

[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[dmz]: ../../reference-architectures/dmz/index.md
[ha-nva]: ../../reference-architectures/dmz/nva-ha.md
[hybrid-network]: ../../reference-architectures/hybrid-networking/index.md
[identity]: ../../reference-architectures/identity/index.md
[multiregion-linux]: ../../reference-architectures/virtual-machines-linux/multi-region-application.md
[multiregion-windows]: ../../reference-architectures/virtual-machines-windows/multi-region-application.md
[n-tier-linux]: ../../reference-architectures/virtual-machines-linux/n-tier.md
[n-tier-windows]: ../../reference-architectures/virtual-machines-windows/n-tier.md
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server