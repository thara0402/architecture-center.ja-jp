---
title: オンプレミス ネットワークの Azure への接続
titleSuffix: Azure Reference Architectures
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 6172866b08197b0ca1cd3aabb3c14c01b4f06f9c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343443"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="7a5f2-103">オンプレミス ネットワークを Azure に接続するためのソリューションを選択する</span><span class="sxs-lookup"><span data-stu-id="7a5f2-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="7a5f2-104">この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="7a5f2-105">各オプションには、さらに詳細な参照アーキテクチャが用意されています。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="7a5f2-106">VPN 接続</span><span class="sxs-lookup"><span data-stu-id="7a5f2-106">VPN connection</span></span>

<span data-ttu-id="7a5f2-107">[VPN ゲートウェイ](/azure/vpn-gateway/vpn-gateway-about-vpngateways)は、Azure 仮想ネットワークとオンプレミスの場所の間で暗号化されたトラフィックを送信する仮想ネットワーク ゲートウェイの一種です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="7a5f2-108">暗号化されたトラフィックは公共のインターネットを通過します。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="7a5f2-109">このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

### <a name="benefits"></a><span data-ttu-id="7a5f2-110">メリット</span><span class="sxs-lookup"><span data-stu-id="7a5f2-110">Benefits</span></span>

- <span data-ttu-id="7a5f2-111">構成が簡単です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-111">Simple to configure.</span></span>

### <a name="challenges"></a><span data-ttu-id="7a5f2-112">課題</span><span class="sxs-lookup"><span data-stu-id="7a5f2-112">Challenges</span></span>

- <span data-ttu-id="7a5f2-113">オンプレミスの VPN デバイスが必要です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="7a5f2-114">Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="7a5f2-115">現在、Azure VPN Gateway 経由の VPN 接続では、最大で 1.25 Gbps の帯域幅がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="7a5f2-116">このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="7a5f2-117">参照アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-117">Reference architecture</span></span>

- [<span data-ttu-id="7a5f2-118">VPN ゲートウェイを使用するハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="7a5f2-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a><span data-ttu-id="7a5f2-119">Azure ExpressRoute 接続</span><span class="sxs-lookup"><span data-stu-id="7a5f2-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="7a5f2-120">[ExpressRoute](/azure/expressroute/) 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="7a5f2-121">プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-121">The private connection extends your on-premises network into Azure.</span></span>

<span data-ttu-id="7a5f2-122">このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span>

### <a name="benefits"></a><span data-ttu-id="7a5f2-123">メリット</span><span class="sxs-lookup"><span data-stu-id="7a5f2-123">Benefits</span></span>

- <span data-ttu-id="7a5f2-124">接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="7a5f2-125">要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="7a5f2-126">ただし、このオプションがない接続プロバイダーもあります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="7a5f2-127">接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="7a5f2-128">接続全体において、99.9% の可用性 SLA があります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-128">99.9% availability SLA across the entire connection.</span></span>

### <a name="challenges"></a><span data-ttu-id="7a5f2-129">課題</span><span class="sxs-lookup"><span data-stu-id="7a5f2-129">Challenges</span></span>

- <span data-ttu-id="7a5f2-130">セットアップが複雑になることがあります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-130">Can be complex to set up.</span></span> <span data-ttu-id="7a5f2-131">ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="7a5f2-132">このプロバイダーは、ネットワーク接続のプロビジョニングを行います。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="7a5f2-133">オンプレミスの、高帯域幅のルーターが必要です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-133">Requires high-bandwidth routers on-premises.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="7a5f2-134">参照アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-134">Reference architecture</span></span>

- [<span data-ttu-id="7a5f2-135">ExpressRoute を使用したハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="7a5f2-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="7a5f2-136">VPN のフェールオーバーを伴う ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="7a5f2-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="7a5f2-137">このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="7a5f2-138">このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span>

### <a name="benefits"></a><span data-ttu-id="7a5f2-139">メリット</span><span class="sxs-lookup"><span data-stu-id="7a5f2-139">Benefits</span></span>

- <span data-ttu-id="7a5f2-140">フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

### <a name="challenges"></a><span data-ttu-id="7a5f2-141">課題</span><span class="sxs-lookup"><span data-stu-id="7a5f2-141">Challenges</span></span>

- <span data-ttu-id="7a5f2-142">構成が複雑です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-142">Complex to configure.</span></span> <span data-ttu-id="7a5f2-143">VPN 接続と ExpressRoute 回線の両方を設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="7a5f2-144">冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="7a5f2-145">参照アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-145">Reference architecture</span></span>

- [<span data-ttu-id="7a5f2-146">ExpressRoute と VPN フェールオーバーを使用したハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="7a5f2-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a><span data-ttu-id="7a5f2-147">ハブスポーク ネットワーク トポロジ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-147">Hub-spoke network topology</span></span>

<span data-ttu-id="7a5f2-148">ハブスポーク ネットワーク トポロジは、ID やセキュリティなどのサービスを共有しながらワークロードを切り離す手法です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="7a5f2-149">ハブは、オンプレミス ネットワークへの主要な接続ポイントとして機能する Azure の仮想ネットワーク (VNet) です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="7a5f2-150">スポークはハブとピア接続する Vnet です。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="7a5f2-151">共有サービスはハブにデプロイされ、個々のワークロードはスポークとしてデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="7a5f2-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>

### <a name="reference-architectures"></a><span data-ttu-id="7a5f2-152">参照用アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-152">Reference architectures</span></span>

- [<span data-ttu-id="7a5f2-153">ハブスポーク トポロジ</span><span class="sxs-lookup"><span data-stu-id="7a5f2-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="7a5f2-154">共有サービスを含むハブスポーク</span><span class="sxs-lookup"><span data-stu-id="7a5f2-154">Hub-spoke with shared services</span></span>](./shared-services.md)
