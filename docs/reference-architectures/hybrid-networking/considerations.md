---
title: オンプレミス ネットワークを Azure に接続するためのソリューションを選択する
description: オンプレミス ネットワークを Azure に接続するための参照アーキテクチャを比較します。
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541051"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="a2f99-103">オンプレミス ネットワークを Azure に接続するためのソリューションを選択する</span><span class="sxs-lookup"><span data-stu-id="a2f99-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="a2f99-104">この記事では、オンプレミス ネットワークを Azure Virtual Network (VNet) に接続するためのオプションを比較します。</span><span class="sxs-lookup"><span data-stu-id="a2f99-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="a2f99-105">各オプションの参照アーキテクチャおよびデプロイ可能なソリューションが提供されます。</span><span class="sxs-lookup"><span data-stu-id="a2f99-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="a2f99-106">VPN 接続</span><span class="sxs-lookup"><span data-stu-id="a2f99-106">VPN connection</span></span>

<span data-ttu-id="a2f99-107">ご利用の仮想プライベートネットワーク (VPN) を使用して、IPSec VPN トンネルを介してオンプレミス ネットワークを Azure VNet に接続します。</span><span class="sxs-lookup"><span data-stu-id="a2f99-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="a2f99-108">このアーキテクチャは、オンプレミス ハードウェアとクラウド間のトラフィックが軽量であると考えられるハイブリッド アプリケーション、または待機時間よりも柔軟性とクラウドの処理能力を優先させる場合に適しています。</span><span class="sxs-lookup"><span data-stu-id="a2f99-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="a2f99-109">**メリット**</span><span class="sxs-lookup"><span data-stu-id="a2f99-109">**Benefits**</span></span>

- <span data-ttu-id="a2f99-110">構成が簡単です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-110">Simple to configure.</span></span>

<span data-ttu-id="a2f99-111">**課題**</span><span class="sxs-lookup"><span data-stu-id="a2f99-111">**Challenges**</span></span>

- <span data-ttu-id="a2f99-112">オンプレミスの VPN デバイスが必要です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="a2f99-113">Microsoft は各 VPN Gateway の 99.9% の可用性を保証していますが、この SLA は、VPN Gateway のみが対象であり、ゲートウェイへのネットワーク接続は含まれていません。</span><span class="sxs-lookup"><span data-stu-id="a2f99-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="a2f99-114">現在、Azure VPN Gateway 経由の VPN 接続では、最大で 200 Mbps の帯域幅がサポートされています。</span><span class="sxs-lookup"><span data-stu-id="a2f99-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="a2f99-115">このスループットを超えると予想される場合は、ご利用の Azure Virtual Network を複数の VPN 接続に分割する必要がある可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="a2f99-116">**[詳細][vpn]**</span><span class="sxs-lookup"><span data-stu-id="a2f99-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="a2f99-117">Azure ExpressRoute 接続</span><span class="sxs-lookup"><span data-stu-id="a2f99-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="a2f99-118">ExpressRoute 接続は、サード パーティ製接続プロバイダーを経由する、プライベートの専用接続を使用します。</span><span class="sxs-lookup"><span data-stu-id="a2f99-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="a2f99-119">プライベート接続は、ご利用のオンプレミス ネットワークを Azure に拡張します。</span><span class="sxs-lookup"><span data-stu-id="a2f99-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="a2f99-120">このアーキテクチャは、高度なスケーラビリティを必要とする、大規模な、ミッション クリティカルのワークロードを実行するハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="a2f99-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="a2f99-121">**メリット**</span><span class="sxs-lookup"><span data-stu-id="a2f99-121">**Benefits**</span></span>

- <span data-ttu-id="a2f99-122">接続プロバイダーに応じて、最大 10 Gbpsの より高い帯域幅を使用できます。</span><span class="sxs-lookup"><span data-stu-id="a2f99-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="a2f99-123">要求が低い期間のコストを削減できる、帯域幅の動的スケーリングがサポートされます。</span><span class="sxs-lookup"><span data-stu-id="a2f99-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="a2f99-124">ただし、このオプションがない接続プロバイダーもあります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="a2f99-125">接続プロバイダーによっては、組織は各国のクラウドに直接アクセスできる場合があります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="a2f99-126">接続全体において、99.9% の可用性 SLA があります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="a2f99-127">**課題**</span><span class="sxs-lookup"><span data-stu-id="a2f99-127">**Challenges**</span></span>

- <span data-ttu-id="a2f99-128">セットアップが複雑になることがあります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-128">Can be complex to set up.</span></span> <span data-ttu-id="a2f99-129">ExpressRoute 接続の作成には、サード パーティ製接続プロバイダーの操作が必要です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="a2f99-130">このプロバイダーは、ネットワーク接続のプロビジョニングを行います。</span><span class="sxs-lookup"><span data-stu-id="a2f99-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="a2f99-131">オンプレミスの、高帯域幅のルーターが必要です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="a2f99-132">**[詳細][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="a2f99-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="a2f99-133">VPN のフェールオーバーを伴う ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="a2f99-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="a2f99-134">このオプションは、前の 2 つのオプションを組み合わせたもので、ExpressRoute を通常の条件で使用しますが、ExpressRoute 回線に接続の切断がある場合は、VPN 接続にフェールオーバーします。</span><span class="sxs-lookup"><span data-stu-id="a2f99-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="a2f99-135">このアーキテクチャは、ExpressRoute のより高い帯域幅および可用性の高いネットワーク接続を必要とする、ハイブリッド アプリケーションに適しています。</span><span class="sxs-lookup"><span data-stu-id="a2f99-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="a2f99-136">**メリット**</span><span class="sxs-lookup"><span data-stu-id="a2f99-136">**Benefits**</span></span>

- <span data-ttu-id="a2f99-137">フェールバック接続はより低い帯域幅ネットワークとなりますが、ExpressRoute 回線が切断されても可用性が高いままとなります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="a2f99-138">**課題**</span><span class="sxs-lookup"><span data-stu-id="a2f99-138">**Challenges**</span></span>

- <span data-ttu-id="a2f99-139">構成が複雑です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-139">Complex to configure.</span></span> <span data-ttu-id="a2f99-140">VPN 接続と ExpressRoute 回線の両方を設定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a2f99-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="a2f99-141">冗長ハードウェア (VPN アプライアンス)、および有料の冗長 Azure VPN Gateway 接続が必要です。</span><span class="sxs-lookup"><span data-stu-id="a2f99-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="a2f99-142">**[詳細][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="a2f99-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md