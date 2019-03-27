---
title: 'CAF: ソフトウェア定義ネットワーク - ハイブリッド ネットワーク'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: ハイブリッド ネットワークで、クラウドの仮想ネットワークをオンプレミスのリソースに接続できるようにする方法を説明します
author: rotycenh
ms.openlocfilehash: 02d181db0ae9baef3b453b8623d212b624f6b16a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243363"
---
# <a name="software-defined-networks-hybrid-network"></a><span data-ttu-id="01447-103">ソフトウェア定義ネットワーク: ハイブリッド ネットワーク</span><span class="sxs-lookup"><span data-stu-id="01447-103">Software Defined Networks: Hybrid network</span></span>

<span data-ttu-id="01447-104">ハイブリッド クラウドのネットワーク アーキテクチャでは、ExpressRoute やその他の接続方法など、ネットワークに直接接続する専用 WAN 接続を使用して、仮想ネットワークがオンプレミスのリソースやサービスにアクセスすること、およびその逆のことが可能です。</span><span class="sxs-lookup"><span data-stu-id="01447-104">The hybrid cloud network architecture allows virtual networks to access your on-premises resources and services and vice versa, using a Dedicated WAN connection such as ExpressRoute or other connection method to directly connect the networks.</span></span>

![ハイブリッド ネットワーク](../../../reference-architectures/hybrid-networking/images/expressroute.png)

<span data-ttu-id="01447-106">クラウド ネイティブの仮想ネットワーク アーキテクチャ上に構築されるハイブリッド仮想ネットワークは、当初に作成されたときには切り離されています。</span><span class="sxs-lookup"><span data-stu-id="01447-106">Building on the cloud native virtual network architecture, a hybrid virtual network is isolated when initially created.</span></span> <span data-ttu-id="01447-107">仮想ネットワーク内のリソースをターゲットとしている他の受信トラフィックはすべて明示的に許可される必要がありますが、オンプレミス環境への接続を追加することで、オンプレミス ネットワークとの間のアクセスを付与します。</span><span class="sxs-lookup"><span data-stu-id="01447-107">Adding connectivity to the on-premises environment grants access to and from the on-premises network, although all other inbound traffic targeting resources in the virtual network need to be explicitly allowed.</span></span> <span data-ttu-id="01447-108">アクセスを制限する仮想ファイアウォール デバイスとルーティング規則を使用して接続をセキュリティで保護できます。または、クラウド ネイティブのルーティング機能を使用するか、ネットワーク仮想アプライアンス (NVA) を配置してトラフィックを管理することで、2 つのネットワーク間でどのサービスがアクセス可能であるかを正確に指定することができます。</span><span class="sxs-lookup"><span data-stu-id="01447-108">You can secure the connection using virtual firewall devices and routing rules to limit access or you can specify exactly what services can be accessed between the two networks using cloud-native routing features or deploying network virtual appliances (NVAs) to manage traffic.</span></span>

<span data-ttu-id="01447-109">ハイブリッド ネットワーク アーキテクチャでは VPN 接続がサポートされていますが、一般に、パフォーマンスが高くセキュリティが向上するため、ExpressRoute などの専用 WAN 接続が優先されます。</span><span class="sxs-lookup"><span data-stu-id="01447-109">Although the hybrid networking architecture supports VPN connections, dedicated WAN connections like ExpressRoute are generally preferred due to higher performance and increased security.</span></span>

## <a name="hybrid-assumptions"></a><span data-ttu-id="01447-110">ハイブリッドの前提条件</span><span class="sxs-lookup"><span data-stu-id="01447-110">Hybrid assumptions</span></span>

<span data-ttu-id="01447-111">ハイブリッド仮想ネットワークのデプロイでは、以下のことが前提とされます。</span><span class="sxs-lookup"><span data-stu-id="01447-111">Deploying a hybrid virtual network assumes the following:</span></span>

- <span data-ttu-id="01447-112">オンプレミス システムと直接通信するためにクラウド ベースの仮想ネットワークを信頼できるようにするため、IT セキュリティ チームが、オンプレミスとクラウド ベースのネットワーク セキュリティ ポリシーを整合済みであること。</span><span class="sxs-lookup"><span data-stu-id="01447-112">Your IT security teams have aligned on-premises and cloud-based network security policy to ensure cloud-based virtual networks can be trusted to communicated directly with on-premises systems.</span></span>
- <span data-ttu-id="01447-113">クラウド ベース ワークロードには、オンプレミスまたはサード パーティのネットワークでホストされているストレージ、アプリケーション、サービスへのアクセスが必要です。または、オンプレミス環境のユーザーやアプリケーションが、クラウドでホストされているリソースにアクセスできる必要があります。</span><span class="sxs-lookup"><span data-stu-id="01447-113">Your cloud-based workloads require access to storage, applications, and services hosted on your on-premises or third-party networks, or your users or applications in your on-premises need access to cloud-hosted resources.</span></span>
- <span data-ttu-id="01447-114">オンプレミスのリソースに依存している既存のアプリケーションやサービスを移行する必要があるが、それらの依存関係を解消するために再開発にリソースを費やしたくはない。</span><span class="sxs-lookup"><span data-stu-id="01447-114">You need to migrate existing applications and services that depend on on-premises resources, but don't want to expend the resources on redevelopment to remove those dependencies.</span></span>
- <span data-ttu-id="01447-115">オンプレミス ネットワークとクラウド プロバイダーの間に VPN 接続または専用 WAN 接続を実装することが、企業ポリシー、法規制上の要件、または技術的互換性の問題によって妨げられていない。</span><span class="sxs-lookup"><span data-stu-id="01447-115">Implementing a VPN or dedicated WAN connection between your on-premises networks and cloud provider is not prevented by corporate policy, regulatory requirements, or technical compatibility issues.</span></span>
- <span data-ttu-id="01447-116">ワークロードでは、サブスクリプションのリソース制限を回避するために複数のサブスクリプションが必要とされない。または、ワークロードには複数のサブスクリプションが必要だが、複数のサブスクリプションにわたるリソースによって使用される接続や共有サービスを一元管理する必要はない。</span><span class="sxs-lookup"><span data-stu-id="01447-116">Your workloads either do not require multiple subscriptions to bypass subscription resource limits, OR your workloads involve multiple subscriptions but do not require central management of connectivity or shared services used by resources spread across multiple subscriptions.</span></span>

<span data-ttu-id="01447-117">クラウド導入チームは、ハイブリッド仮想ネットワーク アーキテクチャの実装を検討する際に、以下の問題について考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="01447-117">Your Cloud Adoption team should consider the following issues when looking at implementing a hybrid virtual networking architecture:</span></span>

- <span data-ttu-id="01447-118">オンプレミス ネットワークをクラウド ネットワークに接続すると、セキュリティ要件がより複雑になります。</span><span class="sxs-lookup"><span data-stu-id="01447-118">Connecting on-premises networks with cloud networks increases the complexity of your security requirements.</span></span> <span data-ttu-id="01447-119">外部の脆弱性と、ハイブリッド環境の両方の側からの未承認アクセスに対して、両方のネットワークをセキュリティで保護する必要があります。</span><span class="sxs-lookup"><span data-stu-id="01447-119">Both networks need to be secured against external vulnerabilities and unauthorized access from both sides of the hybrid environment.</span></span>
- <span data-ttu-id="01447-120">ハイブリッド クラウド環境内でワークロードの数とサイズをスケールすると、ルーティングとトラフィック管理の複雑さが大幅に高まります。</span><span class="sxs-lookup"><span data-stu-id="01447-120">Scaling the number and size of workloads within a hybrid cloud environment can add significant complexity to routing and traffic management.</span></span>
- <span data-ttu-id="01447-121">組織全体で一貫したガバナンスを維持するためには、互換性のある管理とアクセス制御のポリシーを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="01447-121">You will need to develop compatible management and access control policies to maintain consistent governance throughout your organization.</span></span>

## <a name="learn-more"></a><span data-ttu-id="01447-122">詳細情報</span><span class="sxs-lookup"><span data-stu-id="01447-122">Learn more</span></span>

<span data-ttu-id="01447-123">Azure プラットフォームでのハイブリッド ネットワークの詳細については、次の記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="01447-123">See the following for more information about hybrid networking in the Azure platform.</span></span>

- <span data-ttu-id="01447-124">[ハイブリッド ネットワーク参照アーキテクチャ](../../../reference-architectures/hybrid-networking/expressroute.md)。</span><span class="sxs-lookup"><span data-stu-id="01447-124">[Hybrid network reference architecture](../../../reference-architectures/hybrid-networking/expressroute.md).</span></span> <span data-ttu-id="01447-125">Azure のハイブリッド仮想ネットワークでは、ExpressRoute 回線または Azure VPN のいずれかを使用して、仮想ネットワークを、組織に既にある Azure 以外のホスト型 IT 資産に接続します。</span><span class="sxs-lookup"><span data-stu-id="01447-125">Azure hybrid virtual networks use either an ExpressRoute circuit or Azure VPN to connect your virtual network with your organization's existing non-Azure hosted IT assets.</span></span> <span data-ttu-id="01447-126">この記事では、Azure 内にハイブリッド ネットワークを作成するためのオプションについて説明しています。</span><span class="sxs-lookup"><span data-stu-id="01447-126">This article discusses the options for creating a hybrid network in Azure.</span></span>
