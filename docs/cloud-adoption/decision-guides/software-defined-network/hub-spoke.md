---
title: CAF:ソフトウェア定義ネットワーク - クラウド ネイティブ
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ネイティブの仮想ネットワーク サービスの説明
author: rotycenh
ms.openlocfilehash: e0ad6803f2ddc982ea0c42c59fdf2486e1710433
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901554"
---
# <a name="software-defined-networks-hub-and-spoke"></a><span data-ttu-id="fa59d-103">ソフトウェア定義ネットワーク:ハブ アンド スポーク</span><span class="sxs-lookup"><span data-stu-id="fa59d-103">Software Defined Networks: Hub and Spoke</span></span>

<span data-ttu-id="fa59d-104">ハブ アンド スポークのネットワーク モデルは、複数接続されている仮想ネットワークに、Azure ベースのクラウド ネットワーク インフラストラクチャを編成します。</span><span class="sxs-lookup"><span data-stu-id="fa59d-104">The hub and spoke networking model organizes your Azure-based cloud network infrastructure into multiple connected virtual networks.</span></span> <span data-ttu-id="fa59d-105">このモデルを使用すると、一般的な通信やセキュリティの要件をより効率的に管理し、潜在的なサブスクリプションの制限に対処できます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-105">This model allows you to more efficiently manage common communication or security requirements and deal with potential subscription limitations.</span></span>

<span data-ttu-id="fa59d-106">ハブ アンド スポークのモデルで、*ハブ*は、外部接続を管理したり、複数のワークロードによって使用されるサービスをホストしたりするための中心的な場所として機能する仮想ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="fa59d-106">In the hub and spoke model, the *hub* is a virtual network that acts as a central location for managing external connectivity and hosting services used by multiple workloads.</span></span> <span data-ttu-id="fa59d-107">*スポーク*は、ワークロードをホストし、[仮想ネットワーク ピアリング](/virtual-network/virtual-network-peering-overview)を介して中央のハブに接続する仮想ネットワークです。</span><span class="sxs-lookup"><span data-stu-id="fa59d-107">The *spokes* are virtual networks that host workloads and connect to the central hub through [virtual network peering](/virtual-network/virtual-network-peering-overview).</span></span>

<span data-ttu-id="fa59d-108">ワークロード スポーク ネットワークを出入りするすべてのトラフィックは、ハブ ネットワークを介してルーティングされます。そのハブ ネットワークで、集中管理された IT 規則またはプロセスによってトラフィックをルーティング、検査、またはその他の方法で管理できます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-108">All traffic passing in or out of the workload spoke networks is routed through the hub network where it can be routed, inspected, or otherwise managed by centrally managed IT rules or processes.</span></span>

<span data-ttu-id="fa59d-109">このモデルは、次の問題に対処することを目的としています。</span><span class="sxs-lookup"><span data-stu-id="fa59d-109">This model aims to address the following issues:</span></span>

- <span data-ttu-id="fa59d-110">コストの削減と管理の効率。</span><span class="sxs-lookup"><span data-stu-id="fa59d-110">Cost savings and management efficiency.</span></span> <span data-ttu-id="fa59d-111">複数のワークロードで共有できるサービス (ネットワーク仮想アプライアンス (NVA) や DNS サーバーなど) を 1 か所に集めることで、IT は複数のワークロードにわたって過剰なリソースと管理作業を最小限にすることができます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-111">Centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location allows IT to minimize redundant resources and management effort across multiple workloads.</span></span>
- <span data-ttu-id="fa59d-112">サブスクリプションの制限の克服。</span><span class="sxs-lookup"><span data-stu-id="fa59d-112">Overcoming subscriptions limits.</span></span> <span data-ttu-id="fa59d-113">大規模なクラウド ベースのワークロードでは、単一の Azure サブスクリプション内で許可されるリソースよりも多くのリソースの使用が求められる場合があります ([サブスクリプションの制限](/azure/azure-subscription-service-limits)をご覧ください)。</span><span class="sxs-lookup"><span data-stu-id="fa59d-113">Large cloud-based workloads may require the use of more resources than are allowed within a single Azure subscription (see [subscription limits](/azure/azure-subscription-service-limits)).</span></span> <span data-ttu-id="fa59d-114">さまざまなサブスクリプションから中央のハブへのワークロード仮想ネットワークのピアリングで、こうした制限を克服できます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-114">Peering workload virtual networks from different subscriptions to a central hub can overcome these limits.</span></span>
- <span data-ttu-id="fa59d-115">懸念事項の分離。</span><span class="sxs-lookup"><span data-stu-id="fa59d-115">Separation of concerns.</span></span> <span data-ttu-id="fa59d-116">中央の IT チームとワークロード チームの間で個々 のワークロードをデプロイする機能。</span><span class="sxs-lookup"><span data-stu-id="fa59d-116">The ability to deploy individual workloads between central IT teams and workloads teams.</span></span>

<span data-ttu-id="fa59d-117">次の図は、集中管理されたハイブリッド接続を含むハブ アンド スポークのアーキテクチャの例を示しています。</span><span class="sxs-lookup"><span data-stu-id="fa59d-117">The following diagram shows an example hub and spoke architecture including centrally managed hybrid connectivity.</span></span>

![ハブスポークのネットワーク アーキテクチャ](../../../reference-architectures/hybrid-networking/images/hub-spoke.png)

<span data-ttu-id="fa59d-119">ハブ アンド スポークのアーキテクチャは、ハイブリッド ネットワーク アーキテクチャと共によく使用され、複数のワークロード間で共有されるオンプレミス環境への集中管理された接続を提供します。</span><span class="sxs-lookup"><span data-stu-id="fa59d-119">The hub and spoke architecture is often used alongside the hybrid networking architecture, providing a centrally managed connection to your on-premises environment shared between multiple workloads.</span></span> <span data-ttu-id="fa59d-120">このシナリオでは、ワークロードとオンプレミスの間を移動するすべてのトラフィックはハブを通過します。そのハブで、トラフィックを管理および保護できます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-120">In this scenario, all traffic traveling between the workloads and on-premises passes through the hub where it can be managed and secured.</span></span>

## <a name="hub-and-spoke-assumptions"></a><span data-ttu-id="fa59d-121">ハブ アンド スポークの前提条件</span><span class="sxs-lookup"><span data-stu-id="fa59d-121">Hub and spoke assumptions</span></span>

<span data-ttu-id="fa59d-122">ハブ アンド スポークの仮想ネットワーク アーキテクチャの実装は、以下のことを前提とします。</span><span class="sxs-lookup"><span data-stu-id="fa59d-122">Implementing a hub and spoke virtual networking architecture assumes the following:</span></span>

- <span data-ttu-id="fa59d-123">クラウドのデプロイには、開発、テスト、実稼働などの個別の作業環境でホストされるワークロードが含まれ、それらはすべて DNS やディレクトリ サービスなどの一連の共通サービスに依存します。</span><span class="sxs-lookup"><span data-stu-id="fa59d-123">Your cloud deployments will involve workloads hosted in separate working environments, such as development, test, and production, that all rely on a set of common services such as DNS or directory services.</span></span>
- <span data-ttu-id="fa59d-124">ワークロードは相互に通信する必要はありませんが、共通の外部通信と共有サービスの要件があります。</span><span class="sxs-lookup"><span data-stu-id="fa59d-124">Your workloads do not need to communicate with each other but have common external communications and shared services requirements.</span></span>
- <span data-ttu-id="fa59d-125">ワークロードには、単一の Azure サブスクリプション内で使用できるリソースよりも多くのリソースが必要です。</span><span class="sxs-lookup"><span data-stu-id="fa59d-125">Your workloads require more resources than are available within a single Azure subscription.</span></span>
- <span data-ttu-id="fa59d-126">ワークロード チームには、外部接続に対する集中的なセキュリティ管理を維持しながら、自身のリソースに対する委任管理権限を付与する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa59d-126">You need to provide workload teams with delegated management rights over their own resources while maintaining central security control over external connectivity.</span></span>

## <a name="global-hub-and-spoke"></a><span data-ttu-id="fa59d-127">グローバルなハブ アンド スポーク</span><span class="sxs-lookup"><span data-stu-id="fa59d-127">Global hub and spoke</span></span>

<span data-ttu-id="fa59d-128">ハブ アンド スポークのアーキテクチャの実装は、通常、ネットワーク間の待機時間を最小限に抑えるために、同じ Azure リージョンにデプロイされた仮想ネットワークを使用して行われます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-128">Hub and spoke architectures are commonly implemented with virtual networks deployed to the same Azure Region to minimize latency between networks.</span></span> <span data-ttu-id="fa59d-129">ただし、世界規模で展開している大規模組織では、可用性、ディザスター リカバリー、または規制の要件のために、複数のリージョンにわたってワークロードをデプロイすることが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="fa59d-129">However, large organizations with global reach may need to deploy workloads across multiple regions for availability, disaster recovery, or regulatory requirements.</span></span> <span data-ttu-id="fa59d-130">ハブ アンド スポーク モデルは、Azure の[グローバル仮想ネットワーク ピアリング](/azure/virtual-network/virtual-network-peering-overview)の使用により、集中管理と共有サービスをリージョン全体に拡張し、世界中に分散したワークロードをサポートすることができます。</span><span class="sxs-lookup"><span data-stu-id="fa59d-130">Through the use of Azure [global virtual network peering](/azure/virtual-network/virtual-network-peering-overview), the hub and spoke model can extend centralized management and shared services across regions to support workloads distributed across the world.</span></span>

## <a name="learn-more"></a><span data-ttu-id="fa59d-131">詳細情報</span><span class="sxs-lookup"><span data-stu-id="fa59d-131">Learn more</span></span>

<span data-ttu-id="fa59d-132">ハブ アンド スポークのネットワークを Azure に実装する方法の例については、Azure の参照アーキテクチャのサイトで次の例をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="fa59d-132">For examples of how to implement hub and spoke networks on Azure, see the following examples on the Azure Reference Architectures site:</span></span>

- [<span data-ttu-id="fa59d-133">Azure にハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="fa59d-133">Implement a hub-spoke network topology in Azure</span></span>](../../../reference-architectures/hybrid-networking/hub-spoke.md)
- [<span data-ttu-id="fa59d-134">Azure に共有サービスを含むハブスポーク ネットワーク トポロジを実装する</span><span class="sxs-lookup"><span data-stu-id="fa59d-134">Implement a hub-spoke network topology with shared services in Azure</span></span>](../../../reference-architectures/hybrid-networking/shared-services.md)
