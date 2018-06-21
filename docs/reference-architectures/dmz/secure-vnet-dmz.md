---
title: Azure とインターネットの間の DMZ の実装
description: インターネットにアクセスするセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する方法。
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270406"
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="e01cd-103">Azure とインターネットの間の DMZ</span><span class="sxs-lookup"><span data-stu-id="e01cd-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="e01cd-104">次のリファレンス アーキテクチャは、オンプレミスのネットワークを Azure に拡張してインターネット トラフィックも受け入れる、セキュリティ保護されたハイブリッド ネットワークを示しています。</span><span class="sxs-lookup"><span data-stu-id="e01cd-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> 

<span data-ttu-id="e01cd-105">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="e01cd-105">[![0]][0]</span></span> 

<span data-ttu-id="e01cd-106">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="e01cd-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="e01cd-107">このリファレンス アーキテクチャは、「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているアーキテクチャを拡張したものです。</span><span class="sxs-lookup"><span data-stu-id="e01cd-107">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="e01cd-108">オンプレミスのネットワークからのトラフィックを処理するプライベート DMZ に加え、インターネット トラフィックを処理するパブリック DMZ が追加されています。</span><span class="sxs-lookup"><span data-stu-id="e01cd-108">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="e01cd-109">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="e01cd-109">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="e01cd-110">ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="e01cd-110">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="e01cd-111">オンプレミスとインターネットからの着信トラフィックをルーティングする Azure インフラストラクチャ。</span><span class="sxs-lookup"><span data-stu-id="e01cd-111">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="e01cd-112">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="e01cd-112">Architecture</span></span>

<span data-ttu-id="e01cd-113">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="e01cd-114">**パブリック IP アドレス (PIP)**。</span><span class="sxs-lookup"><span data-stu-id="e01cd-114">**Public IP address (PIP)**.</span></span> <span data-ttu-id="e01cd-115">パブリック エンドポイントの IP アドレス。</span><span class="sxs-lookup"><span data-stu-id="e01cd-115">The IP address of the public endpoint.</span></span> <span data-ttu-id="e01cd-116">インターネットに接続されている外部ユーザーは、このアドレスを介してシステムにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-116">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="e01cd-117">**ネットワーク仮想アプライアンス (NVA)**</span><span class="sxs-lookup"><span data-stu-id="e01cd-117">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="e01cd-118">このアーキテクチャには、インターネットから発信されるトラフィック用の独立した NVA プールが含まれています。</span><span class="sxs-lookup"><span data-stu-id="e01cd-118">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="e01cd-119">**Azure ロード バランサー**。</span><span class="sxs-lookup"><span data-stu-id="e01cd-119">**Azure load balancer**.</span></span> <span data-ttu-id="e01cd-120">インターネットから着信するすべての要求は、ロード バランサーを通過してパブリック DMZ 内の NVA に分散されます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-120">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="e01cd-121">**パブリック DMZ のインバウンド サブネット**。</span><span class="sxs-lookup"><span data-stu-id="e01cd-121">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="e01cd-122">このサブネットは、Azure ロード バランサーからの要求を受け入れます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-122">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="e01cd-123">着信要求は、パブリック DMZ 内のいずれかの NVA に渡されます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-123">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="e01cd-124">**パブリック DMZ のアウトバウンド サブネット**。</span><span class="sxs-lookup"><span data-stu-id="e01cd-124">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="e01cd-125">NVA によって承認された要求は、このサブネットを通過して、Web 層の内部ロード バランサーに送られます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-125">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="e01cd-126">Recommendations</span><span class="sxs-lookup"><span data-stu-id="e01cd-126">Recommendations</span></span>

<span data-ttu-id="e01cd-127">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-127">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="e01cd-128">これらの推奨事項には、優先される特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-128">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="e01cd-129">NVA の推奨事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-129">NVA recommendations</span></span>

<span data-ttu-id="e01cd-130">インターネットから発信されるトラフィックとオンプレミスから発信されるトラフィックには異なる NVA セットを使用します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-130">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="e01cd-131">両方にただ 1 つの NVA セットを使用すると、2 つのネットワーク トラフィック セットの間にセキュリティ境界が提供されないため、セキュリティ リスクが発生します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-131">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="e01cd-132">異なる NVA を使用すると、セキュリティ規則のチェックの複雑さが軽減され、規則と着信ネットワーク要求の対応が明確になります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-132">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="e01cd-133">片方の NVA セットはインターネット トラフィックのみの規則を実装し、他方の NVA セットはオンプレミスのトラフィックのみの規則を実装します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-133">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="e01cd-134">第 7 層の NVA を含めて、アプリケーションの接続を NVA レベルで終了し、バックエンド層との互換性を維持します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-134">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="e01cd-135">これにより、バックエンド層からの応答トラフィックが NVA を介して返される対称接続性が保証されます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-135">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="e01cd-136">パブリック ロード バランサーの推奨事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-136">Public load balancer recommendations</span></span>

<span data-ttu-id="e01cd-137">スケーラビリティと可用性のために、パブリック DMZ の NVA を[可用性セット][availability-set]にデプロイし、[インターネットに接続するロード バランサー][load-balancer]を使用して、インターネットからの要求を可用性セット内の NVA に分散します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-137">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="e01cd-138">インターネット トラフィックに必要なポートのみで要求を受け入れるようにロード バランサーを構成します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-138">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="e01cd-139">たとえば、インバウンドの HTTP 要求をポート 80 に制限し、インバウンドの HTTPS 要求をポート 443 に制限します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-139">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="e01cd-140">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-140">Scalability considerations</span></span>

<span data-ttu-id="e01cd-141">初期のアーキテクチャでは、パブリック DMZ 内に単一の NVA が必要である場合でも、最初からパブリック DMZ の前にロード バランサーを配置しておくことをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-141">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="e01cd-142">これにより、将来必要になった場合に、複数の NVA を簡単にスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-142">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="e01cd-143">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-143">Availability considerations</span></span>

<span data-ttu-id="e01cd-144">インターネットに接続するロード バランサーは、パブリック DMZ の インバウンド サブネット内の各 NVA が[正常性プローブ][lb-probe]を実装することを要求します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-144">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="e01cd-145">エンドポイントで応答しない正常性プローブは使用不可であるとみなされ、ロード バランサーは、同じ可用性セット内の他の NVA に要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-145">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="e01cd-146">すべての NVA が 応答しない場合、アプリケーションは失敗するため、正常な NVA インスタンスの数が定義されたしきい値を下回った場合に DevOps アラートを生成するように構成された監視機能を用意することが重要であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-146">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="e01cd-147">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-147">Manageability considerations</span></span>

<span data-ttu-id="e01cd-148">パブリック DMZ 内の NVA に対するすべての監視と管理は、管理サブネット内のジャンプボックスによって実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-148">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="e01cd-149">「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているように、アクセスを制限するために、オンプレミスのネットワークからゲートウェイ経由でジャンプボックスに至る単一のネットワーク ルートを定義します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-149">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="e01cd-150">オンプレミスのネットワークから Azure へのゲートウェイ接続がダウンした場合でも、パブリック IP アドレスをデプロイしてジャンプボックスに追加し、インターネットからログインすることで、ジャンプボックスにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-150">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="e01cd-151">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="e01cd-151">Security considerations</span></span>

<span data-ttu-id="e01cd-152">このリファレンス アーキテクチャは、複数のレベルのセキュリティを実装します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-152">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="e01cd-153">インターネットに接続するロード バランサーは、アプリケーションで必要なポートでのみ、パブリック DMZ のインバウンド サブネット内の NVA に要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-153">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="e01cd-154">パブリック DMZ のインバウンド サブネットとアウトバウンド サブネットの NSG 規則は、NSG 規則に適合しない要求をブロックすることで、NVA への侵入を防ぎます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-154">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="e01cd-155">NVA の NAT ルーティング構成は、ポート 80 とポート 443 で着信した要求を Web 層のロード バランサーに送信しますが、それ以外のすべてのポートの要求を無視します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-155">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="e01cd-156">すべてのポートに着信したすべての要求をログに記録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-156">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="e01cd-157">このログを定期的に監査して、特に予期されたパラメーターの範囲外にある要求の有無を調べます。そのような要求は、侵入の試みを示唆している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-157">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="solution-deployment"></a><span data-ttu-id="e01cd-158">ソリューションのデプロイ</span><span class="sxs-lookup"><span data-stu-id="e01cd-158">Solution deployment</span></span>

<span data-ttu-id="e01cd-159">これらの推奨事項を実装する参照アーキテクチャのデプロイは、[GitHub][github-folder] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-159">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> <span data-ttu-id="e01cd-160">リファレンス アーキテクチャは、次の手順に従って、Windows または Linux Vm のいずれにもデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-160">The reference architecture can be deployed either with Windows or Linux VMs by following the directions below:</span></span>

1. <span data-ttu-id="e01cd-161">下のボタンをクリックしてください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-161">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="e01cd-162">Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-162">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="e01cd-163">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-public-dmz-network-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-163">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="e01cd-164">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-164">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="e01cd-165">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-165">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="e01cd-166">**[OS の種類]** ボックスの一覧の **[Windows]** または **[linux]**. を選択します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-166">Select the **Os Type** from the drop down box, **windows** or **linux**.</span></span>
   * <span data-ttu-id="e01cd-167">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-167">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="e01cd-168">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-168">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="e01cd-169">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-169">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="e01cd-170">下のボタンをクリックしてください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-170">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="e01cd-171">Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-171">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="e01cd-172">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[新規作成]** を選択し、テキスト ボックスに「`ra-public-dmz-wl-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-172">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-wl-rg` in the text box.</span></span>
   * <span data-ttu-id="e01cd-173">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-173">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="e01cd-174">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-174">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="e01cd-175">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-175">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="e01cd-176">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-176">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="e01cd-177">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-177">Wait for the deployment to complete.</span></span>
7. <span data-ttu-id="e01cd-178">下のボタンをクリックしてください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-178">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. <span data-ttu-id="e01cd-179">Azure Portal でリンクが開いたら、いくつかの設定に値を入力する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e01cd-179">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="e01cd-180">**リソース グループ**の名前はパラメーター ファイルで既に定義されているため、**[既存のものを使用]** を選択し、テキスト ボックスに「`ra-public-dmz-network-rg`」と入力します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-180">The **Resource group** name is already defined in the parameter file, so select **Use Existing** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="e01cd-181">**[場所]** ボックスの一覧でリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-181">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="e01cd-182">**[Template Root Uri (テンプレート ルート URI)]** または **[Parameter Root Uri (パラメーター ルート URI)]** ボックスは編集しないでください。</span><span class="sxs-lookup"><span data-stu-id="e01cd-182">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="e01cd-183">使用条件を確認し、**[上記の使用条件に同意する]** チェック ボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-183">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="e01cd-184">**[購入]** ボタンをクリックします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-184">Click the **Purchase** button.</span></span>
9. <span data-ttu-id="e01cd-185">デプロイが完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="e01cd-185">Wait for the deployment to complete.</span></span>
10. <span data-ttu-id="e01cd-186">パラメーター ファイルには、すべての VM のハードコーディングされた管理者のユーザー名とパスワードが含まれているため、この両方をすぐに変更することを強くお勧めします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-186">The parameter files include hard-coded administrator user name and password for all VMs, and it is strongly recommended that you immediately change both.</span></span> <span data-ttu-id="e01cd-187">デプロイの各 VM で、Azure ポータルで VM を選択し、**[サポート + トラブルシューティング]** ブレードで **[パスワードのリセット]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="e01cd-187">For each VM in the deployment, select it in the Azure portal and then click  **Reset password** in the **Support + troubleshooting** blade.</span></span> <span data-ttu-id="e01cd-188">**[モード]** ボックスの一覧の **[パスワードのリセット]** を選択し、新しい**ユーザー名**と**パスワード**を選択します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-188">Select **Reset password** in the **Mode** drop down box, then select a new **User name** and **Password**.</span></span> <span data-ttu-id="e01cd-189">**[更新]** ボタンをクリックして保存します。</span><span class="sxs-lookup"><span data-stu-id="e01cd-189">Click the **Update** button to save.</span></span>


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "セキュリティ保護されたハイブリッド ネットワーク アーキテクチャ"
