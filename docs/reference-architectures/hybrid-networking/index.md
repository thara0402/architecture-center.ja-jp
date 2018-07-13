---
title: オンプレミス ネットワークを Azure に接続するためのソリューションを選択する
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: 0cc07d3b7d45accf9f99ce32914b0ef065d62f32
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987480"
---
# <a name="connect-an-on-premises-network-to-azure"></a><span data-ttu-id="f4bec-103">オンプレミス ネットワークの Azure への接続</span><span class="sxs-lookup"><span data-stu-id="f4bec-103">Connect an on-premises network to Azure</span></span>

<span data-ttu-id="f4bec-104">この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。</span><span class="sxs-lookup"><span data-stu-id="f4bec-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="f4bec-105">各オプションには、さらに詳細な参照アーキテクチャが用意されています。</span><span class="sxs-lookup"><span data-stu-id="f4bec-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="f4bec-106">VPN 接続</span><span class="sxs-lookup"><span data-stu-id="f4bec-106">VPN connection</span></span>

<span data-ttu-id="f4bec-107">[VPN ゲートウェイ](/azure/vpn-gateway/vpn-gateway-about-vpngateways)は、Azure 仮想ネットワークとオンプレミスの場所の間で暗号化されたトラフィックを送信する仮想ネットワーク ゲートウェイの一種です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="f4bec-108">暗号化されたトラフィックは公共のインターネットを通過します。</span><span class="sxs-lookup"><span data-stu-id="f4bec-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="f4bec-109">このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。</span><span class="sxs-lookup"><span data-stu-id="f4bec-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="f4bec-110">**メリット**</span><span class="sxs-lookup"><span data-stu-id="f4bec-110">**Benefits**</span></span>

- <span data-ttu-id="f4bec-111">構成が簡単です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-111">Simple to configure.</span></span>

<span data-ttu-id="f4bec-112">**課題**</span><span class="sxs-lookup"><span data-stu-id="f4bec-112">**Challenges**</span></span>

- <span data-ttu-id="f4bec-113">オンプレミスの VPN デバイスが必要です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="f4bec-114">Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。</span><span class="sxs-lookup"><span data-stu-id="f4bec-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="f4bec-115">現在、Azure VPN Gateway 経由の VPN 接続では、最大で 200 Mbps の帯域幅がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="f4bec-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="f4bec-116">このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="f4bec-117">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="f4bec-117">**Reference architecture**</span></span>

- [<span data-ttu-id="f4bec-118">VPN ゲートウェイを使用するハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="f4bec-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

## <a name="azure-expressroute-connection"></a><span data-ttu-id="f4bec-119">Azure ExpressRoute 接続</span><span class="sxs-lookup"><span data-stu-id="f4bec-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="f4bec-120">[ExpressRoute](/azure/expressroute/) 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。</span><span class="sxs-lookup"><span data-stu-id="f4bec-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="f4bec-121">プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</span><span class="sxs-lookup"><span data-stu-id="f4bec-121">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="f4bec-122">このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="f4bec-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="f4bec-123">**メリット**</span><span class="sxs-lookup"><span data-stu-id="f4bec-123">**Benefits**</span></span>

- <span data-ttu-id="f4bec-124">接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。</span><span class="sxs-lookup"><span data-stu-id="f4bec-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="f4bec-125">要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="f4bec-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="f4bec-126">ただし、このオプションがない接続プロバイダーもあります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="f4bec-127">接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="f4bec-128">接続全体において、99.9% の可用性 SLA があります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-128">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="f4bec-129">**課題**</span><span class="sxs-lookup"><span data-stu-id="f4bec-129">**Challenges**</span></span>

- <span data-ttu-id="f4bec-130">セットアップが複雑になることがあります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-130">Can be complex to set up.</span></span> <span data-ttu-id="f4bec-131">ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="f4bec-132">このプロバイダーは、ネットワーク接続のプロビジョニングを行います。</span><span class="sxs-lookup"><span data-stu-id="f4bec-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="f4bec-133">オンプレミスの、高帯域幅のルーターが必要です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-133">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="f4bec-134">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="f4bec-134">**Reference architecture**</span></span>

- [<span data-ttu-id="f4bec-135">ExpressRoute を使用したハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="f4bec-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="f4bec-136">VPN のフェールオーバーを伴う ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="f4bec-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="f4bec-137">このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。</span><span class="sxs-lookup"><span data-stu-id="f4bec-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="f4bec-138">このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="f4bec-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="f4bec-139">**メリット**</span><span class="sxs-lookup"><span data-stu-id="f4bec-139">**Benefits**</span></span>

- <span data-ttu-id="f4bec-140">フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="f4bec-141">**課題**</span><span class="sxs-lookup"><span data-stu-id="f4bec-141">**Challenges**</span></span>

- <span data-ttu-id="f4bec-142">構成が複雑です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-142">Complex to configure.</span></span> <span data-ttu-id="f4bec-143">VPN 接続と ExpressRoute 回線の両方を設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="f4bec-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="f4bec-144">冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="f4bec-145">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="f4bec-145">**Reference architecture**</span></span>

- [<span data-ttu-id="f4bec-146">ExpressRoute と VPN フェールオーバーを使用したハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="f4bec-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a><span data-ttu-id="f4bec-147">ハブスポーク ネットワーク トポロジ</span><span class="sxs-lookup"><span data-stu-id="f4bec-147">Hub-spoke network topology</span></span>

<span data-ttu-id="f4bec-148">ハブスポーク ネットワーク トポロジは、ID やセキュリティなどのサービスを共有しながらワークロードを切り離す手法です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="f4bec-149">ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="f4bec-150">スポークはハブとピア接続する Vnet です。</span><span class="sxs-lookup"><span data-stu-id="f4bec-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="f4bec-151">共有サービスはハブにデプロイされ、個々のワークロードはスポークとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="f4bec-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>


<span data-ttu-id="f4bec-152">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="f4bec-152">**Reference architectures**</span></span>

- [<span data-ttu-id="f4bec-153">ハブスポーク トポロジ</span><span class="sxs-lookup"><span data-stu-id="f4bec-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="f4bec-154">共有サービスを含むハブスポーク</span><span class="sxs-lookup"><span data-stu-id="f4bec-154">Hub-spoke with shared services</span></span>](./shared-services.md)
