---
title: Azure とインターネットの間の DMZ の実装
titleSuffix: Azure Reference Architectures
description: インターネットにアクセスするセキュリティ保護されたハイブリッド ネットワーク アーキテクチャを Azure に実装する方法。
author: telmosampaio
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: 80125626d0c79888445bc7828577846bcce9fc67
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488224"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a><span data-ttu-id="2e97f-103">Azure とインターネットの間の DMZ の実装</span><span class="sxs-lookup"><span data-stu-id="2e97f-103">Implement a DMZ between Azure and the Internet</span></span>

<span data-ttu-id="2e97f-104">次のリファレンス アーキテクチャは、オンプレミスのネットワークを Azure に拡張してインターネット トラフィックも受け入れる、セキュリティ保護されたハイブリッド ネットワークを示しています。</span><span class="sxs-lookup"><span data-stu-id="2e97f-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> <span data-ttu-id="2e97f-105">[**このソリューションをデプロイします**](#deploy-the-solution)。</span><span class="sxs-lookup"><span data-stu-id="2e97f-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

> [!NOTE]
> <span data-ttu-id="2e97f-106">このシナリオは、クラウドベースのネットワーク セキュリティ サービスである [Azure Firewall](/azure/firewall/) を使用して実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-106">This scenario can also be accomplished using [Azure Firewall](/azure/firewall/), a cloud-based network security service.</span></span>

![セキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](./images/dmz-public.png)

<span data-ttu-id="2e97f-108">"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"</span><span class="sxs-lookup"><span data-stu-id="2e97f-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="2e97f-109">このリファレンス アーキテクチャは、「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているアーキテクチャを拡張したものです。</span><span class="sxs-lookup"><span data-stu-id="2e97f-109">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="2e97f-110">オンプレミスのネットワークからのトラフィックを処理するプライベート DMZ に加え、インターネット トラフィックを処理するパブリック DMZ が追加されています。</span><span class="sxs-lookup"><span data-stu-id="2e97f-110">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network.</span></span>

<span data-ttu-id="2e97f-111">このアーキテクチャの一般的な用途は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="2e97f-111">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="2e97f-112">ワークロードの一部がオンプレミスで、一部が Azure で実行されるハイブリッド アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="2e97f-112">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="2e97f-113">オンプレミスとインターネットからの着信トラフィックをルーティングする Azure インフラストラクチャ。</span><span class="sxs-lookup"><span data-stu-id="2e97f-113">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="2e97f-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="2e97f-114">Architecture</span></span>

<span data-ttu-id="2e97f-115">アーキテクチャは、次のコンポーネントで構成されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-115">The architecture consists of the following components.</span></span>

- <span data-ttu-id="2e97f-116">**パブリック IP アドレス (PIP)**。</span><span class="sxs-lookup"><span data-stu-id="2e97f-116">**Public IP address (PIP)**.</span></span> <span data-ttu-id="2e97f-117">パブリック エンドポイントの IP アドレス。</span><span class="sxs-lookup"><span data-stu-id="2e97f-117">The IP address of the public endpoint.</span></span> <span data-ttu-id="2e97f-118">インターネットに接続されている外部ユーザーは、このアドレスを介してシステムにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-118">External users connected to the Internet can access the system through this address.</span></span>
- <span data-ttu-id="2e97f-119">**ネットワーク仮想アプライアンス (NVA)**</span><span class="sxs-lookup"><span data-stu-id="2e97f-119">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="2e97f-120">このアーキテクチャには、インターネットから発信されるトラフィック用の独立した NVA プールが含まれています。</span><span class="sxs-lookup"><span data-stu-id="2e97f-120">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
- <span data-ttu-id="2e97f-121">**Azure ロード バランサー**。</span><span class="sxs-lookup"><span data-stu-id="2e97f-121">**Azure load balancer**.</span></span> <span data-ttu-id="2e97f-122">インターネットから着信するすべての要求は、ロード バランサーを通過してパブリック DMZ 内の NVA に分散されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-122">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="2e97f-123">**パブリック DMZ のインバウンド サブネット**。</span><span class="sxs-lookup"><span data-stu-id="2e97f-123">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="2e97f-124">このサブネットは、Azure ロード バランサーからの要求を受け入れます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-124">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="2e97f-125">着信要求は、パブリック DMZ 内のいずれかの NVA に渡されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-125">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="2e97f-126">**パブリック DMZ のアウトバウンド サブネット**。</span><span class="sxs-lookup"><span data-stu-id="2e97f-126">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="2e97f-127">NVA によって承認された要求は、このサブネットを通過して、Web 層の内部ロード バランサーに送られます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-127">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="2e97f-128">Recommendations</span><span class="sxs-lookup"><span data-stu-id="2e97f-128">Recommendations</span></span>

<span data-ttu-id="2e97f-129">ほとんどのシナリオには、次の推奨事項が適用されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-129">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="2e97f-130">これらの推奨事項には、オーバーライドする特定の要件がない限り、従ってください。</span><span class="sxs-lookup"><span data-stu-id="2e97f-130">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="nva-recommendations"></a><span data-ttu-id="2e97f-131">NVA の推奨事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-131">NVA recommendations</span></span>

<span data-ttu-id="2e97f-132">インターネットから発信されるトラフィックとオンプレミスから発信されるトラフィックには異なる NVA セットを使用します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-132">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="2e97f-133">両方にただ 1 つの NVA セットを使用すると、2 つのネットワーク トラフィック セットの間にセキュリティ境界が提供されないため、セキュリティ リスクが発生します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-133">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="2e97f-134">異なる NVA を使用すると、セキュリティ規則のチェックの複雑さが軽減され、規則と着信ネットワーク要求の対応が明確になります。</span><span class="sxs-lookup"><span data-stu-id="2e97f-134">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="2e97f-135">片方の NVA セットはインターネット トラフィックのみの規則を実装し、他方の NVA セットはオンプレミスのトラフィックのみの規則を実装します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-135">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="2e97f-136">第 7 層の NVA を含めて、アプリケーションの接続を NVA レベルで終了し、バックエンド層との互換性を維持します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-136">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="2e97f-137">これにより、バックエンド層からの応答トラフィックが NVA を介して返される対称接続性が保証されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-137">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="2e97f-138">パブリック ロード バランサーの推奨事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-138">Public load balancer recommendations</span></span>

<span data-ttu-id="2e97f-139">スケーラビリティと可用性のために、パブリック DMZ の NVA を[可用性セット][availability-set]にデプロイし、[インターネットに接続するロード バランサー][load-balancer]を使用して、インターネットからの要求を可用性セット内の NVA に分散します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-139">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>

<span data-ttu-id="2e97f-140">インターネット トラフィックに必要なポートのみで要求を受け入れるようにロード バランサーを構成します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-140">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="2e97f-141">たとえば、インバウンドの HTTP 要求をポート 80 に制限し、インバウンドの HTTPS 要求をポート 443 に制限します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-141">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="2e97f-142">スケーラビリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-142">Scalability considerations</span></span>

<span data-ttu-id="2e97f-143">初期のアーキテクチャでは、パブリック DMZ 内に単一の NVA が必要である場合でも、最初からパブリック DMZ の前にロード バランサーを配置しておくことをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-143">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="2e97f-144">これにより、将来必要になった場合に、複数の NVA を簡単にスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-144">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="2e97f-145">可用性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-145">Availability considerations</span></span>

<span data-ttu-id="2e97f-146">インターネットに接続するロード バランサーは、パブリック DMZ の インバウンド サブネット内の各 NVA が[正常性プローブ][lb-probe]を実装することを要求します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-146">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="2e97f-147">エンドポイントで応答しない正常性プローブは使用不可であるとみなされ、ロード バランサーは、同じ可用性セット内の他の NVA に要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-147">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="2e97f-148">すべての NVA が 応答しない場合、アプリケーションは失敗するため、正常な NVA インスタンスの数が定義されたしきい値を下回った場合に DevOps アラートを生成するように構成された監視機能を用意することが重要であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="2e97f-148">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="2e97f-149">管理容易性に関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-149">Manageability considerations</span></span>

<span data-ttu-id="2e97f-150">パブリック DMZ 内の NVA に対するすべての監視と管理は、管理サブネット内のジャンプボックスによって実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2e97f-150">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="2e97f-151">「[Azure とオンプレミスのデータセンターの間の DMZ の実装][implementing-a-secure-hybrid-network-architecture]」に記載されているように、アクセスを制限するために、オンプレミスのネットワークからゲートウェイ経由でジャンプボックスに至る単一のネットワーク ルートを定義します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-151">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="2e97f-152">オンプレミスのネットワークから Azure へのゲートウェイ接続がダウンした場合でも、パブリック IP アドレスをデプロイしてジャンプボックスに追加し、インターネットからログインすることで、ジャンプボックスにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-152">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="2e97f-153">セキュリティに関する考慮事項</span><span class="sxs-lookup"><span data-stu-id="2e97f-153">Security considerations</span></span>

<span data-ttu-id="2e97f-154">このリファレンス アーキテクチャは、複数のレベルのセキュリティを実装します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-154">This reference architecture implements multiple levels of security:</span></span>

- <span data-ttu-id="2e97f-155">インターネットに接続するロード バランサーは、アプリケーションで必要なポートでのみ、パブリック DMZ のインバウンド サブネット内の NVA に要求を送信します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-155">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
- <span data-ttu-id="2e97f-156">パブリック DMZ のインバウンド サブネットとアウトバウンド サブネットの NSG 規則は、NSG 規則に適合しない要求をブロックすることで、NVA への侵入を防ぎます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-156">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
- <span data-ttu-id="2e97f-157">NVA の NAT ルーティング構成は、ポート 80 とポート 443 で着信した要求を Web 層のロード バランサーに送信しますが、それ以外のすべてのポートの要求を無視します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-157">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="2e97f-158">すべてのポートに着信したすべての要求をログに記録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2e97f-158">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="2e97f-159">このログを定期的に監査して、特に予期されたパラメーターの範囲外にある要求の有無を調べます。そのような要求は、侵入の試みを示唆している可能性があります。</span><span class="sxs-lookup"><span data-stu-id="2e97f-159">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="2e97f-160">ソリューションのデプロイ方法</span><span class="sxs-lookup"><span data-stu-id="2e97f-160">Deploy the solution</span></span>

<span data-ttu-id="2e97f-161">これらの推奨事項を実装する参照アーキテクチャのデプロイは、[GitHub][github-folder] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-161">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2e97f-162">前提条件</span><span class="sxs-lookup"><span data-stu-id="2e97f-162">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="2e97f-163">リソースのデプロイ</span><span class="sxs-lookup"><span data-stu-id="2e97f-163">Deploy resources</span></span>

1. <span data-ttu-id="2e97f-164">参照アーキテクチャ GitHub リポジトリの `/dmz/secure-vnet-dmz` フォルダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-164">Navigate to the `/dmz/secure-vnet-dmz` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="2e97f-165">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="2e97f-166">次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-166">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="2e97f-167">オンプレミスと Azure ゲートウェイに接続する</span><span class="sxs-lookup"><span data-stu-id="2e97f-167">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="2e97f-168">この手順では、2 つのローカル ネットワーク ゲートウェイを接続します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-168">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="2e97f-169">Azure portal で、作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-169">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="2e97f-170">`ra-vpn-vgw-pip` という名前のリソースを探し、**[概要]** ブレードに表示されている IP アドレスをコピーします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-170">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="2e97f-171">`onprem-vpn-lgw` という名前のリソースを探します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-171">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="2e97f-172">**[構成]** ブレードをクリックします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-172">Click the **Configuration** blade.</span></span> <span data-ttu-id="2e97f-173">**[IP アドレス]** に手順 2 の IP アドレスを貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-173">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![[IP アドレス] フィールドのスクリーンショット](./images/local-net-gw.png)

5. <span data-ttu-id="2e97f-175">**[保存]** をクリックし、処理が完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-175">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="2e97f-176">完了までに約 5 分かかります。</span><span class="sxs-lookup"><span data-stu-id="2e97f-176">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="2e97f-177">`onprem-vpn-gateway1-pip` という名前のリソースを探します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-177">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="2e97f-178">**[概要]** ブレードに表示されている IP アドレスをコピーします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-178">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="2e97f-179">`ra-vpn-lgw` という名前のリソースを探します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-179">Find the resource named `ra-vpn-lgw`.</span></span>

8. <span data-ttu-id="2e97f-180">**[構成]** ブレードをクリックします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-180">Click the **Configuration** blade.</span></span> <span data-ttu-id="2e97f-181">**[IP アドレス]** に手順 6 の IP アドレスを貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-181">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="2e97f-182">**[保存]** をクリックし、処理が完了するまで待ちます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-182">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="2e97f-183">接続を確認するには、各ゲートウェイの **[接続]** ブレードに移動します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-183">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="2e97f-184">状態が **[接続済み]** であることを確認します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-184">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="2e97f-185">ネットワーク トラフィックが Web 階層に到達していることを確認する</span><span class="sxs-lookup"><span data-stu-id="2e97f-185">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="2e97f-186">Azure portal で、作成したリソース グループに移動します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-186">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="2e97f-187">パブリック DMZ の前面にあるロード バランサーである `pub-dmz-lb` という名前のリソースを探します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-187">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span>

3. <span data-ttu-id="2e97f-188">**[概要]** ブレードからパブリック IP アドレスをコピーし、このアドレスを Web ブラウザーで開きます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-188">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="2e97f-189">既定の Apache2 サーバーのホーム ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-189">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="2e97f-190">プライベート DMZ の前面にあるロード バランサーである `int-dmz-lb` という名前のリソースを探します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-190">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="2e97f-191">**[概要]** ブレードからプライベート IP アドレスをコピーします。</span><span class="sxs-lookup"><span data-stu-id="2e97f-191">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="2e97f-192">`jb-vm1` という名前の VM を見つけます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-192">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="2e97f-193">**[接続]** をクリックし、リモート デスクトップを使用して VM に接続します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-193">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="2e97f-194">ユーザー名とパスワードは、onprem.json ファイルに指定されています。</span><span class="sxs-lookup"><span data-stu-id="2e97f-194">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="2e97f-195">リモート デスクトップ セッションから Web ブラウザーを開き、手順 4 の IP アドレスに移動します。</span><span class="sxs-lookup"><span data-stu-id="2e97f-195">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="2e97f-196">既定の Apache2 サーバーのホーム ページが表示されます。</span><span class="sxs-lookup"><span data-stu-id="2e97f-196">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
