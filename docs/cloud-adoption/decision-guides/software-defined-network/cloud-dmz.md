---
title: CAF:ソフトウェア定義ネットワーク - クラウド DMZ
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: このネットワーク アーキテクチャにより、オンプレミス ネットワークとクラウド ベース ネットワークの間のアクセスを制限することができます。
author: rotycenh
ms.openlocfilehash: a192541dcfb0f3d713f4139a2ab0541d0c7202db
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242813"
---
# <a name="software-defined-networks-cloud-dmz"></a><span data-ttu-id="0a218-103">ソフトウェア定義ネットワーク:クラウド DMZ</span><span class="sxs-lookup"><span data-stu-id="0a218-103">Software Defined Networks: Cloud DMZ</span></span>

<span data-ttu-id="0a218-104">クラウド DMZ ネットワーク アーキテクチャでは、仮想プライベート ネットワーク (VPN) を使用してネットワークに接続することで、オンプレミス ネットワークとクラウド ベース ネットワークとの間のアクセスを制限することができます。</span><span class="sxs-lookup"><span data-stu-id="0a218-104">The Cloud DMZ network architecture allows limited access between your on-premises and cloud-based networks, using a virtual private network (VPN) to connect the networks.</span></span> <span data-ttu-id="0a218-105">DMZ は、クラウド ベースのリソースからオンプレミス ネットワークへのアクセスを保護するために、クラウドにデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="0a218-105">A DMZ is deployed in the cloud to secure access to the on-premises network from cloud-based resources.</span></span>

![セキュリティ保護されたハイブリッド ネットワーク アーキテクチャ](../../../reference-architectures/dmz/images/dmz-private.png)

<span data-ttu-id="0a218-107">このアーキテクチャは、組織がクラウド ベースのワークロードとオンプレミスのワークロードの統合を開始したいが、完全に成熟したクラウド セキュリティ ポリシーを持たない、または 2 つの環境の間でセキュリティで保護された専用の WAN 接続を取得していないシナリオをサポートするように設計されています。</span><span class="sxs-lookup"><span data-stu-id="0a218-107">This architecture is designed to support scenarios where your organization wants to start integrating cloud-based workloads with on-premises workloads but may not have fully matured cloud security policies or acquired a secure dedicated WAN connection between the two environments.</span></span> <span data-ttu-id="0a218-108">その結果、オンプレミス サービスをセキュリティで確実に保護するために、クラウド ネットワークを非武装地帯のように扱う必要があります。</span><span class="sxs-lookup"><span data-stu-id="0a218-108">As a result, cloud networks should be treated like a demilitarized zone to ensure on-premises services are secure.</span></span>

<span data-ttu-id="0a218-109">DMZ は、ファイアウォールやパケット検査などのセキュリティ機能を実装するために、ネットワーク仮想アプライアンス (NVA) をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="0a218-109">The DMZ deploys network virtual appliances (NVAs) to implement security functionality such as firewalls and packet inspection.</span></span> <span data-ttu-id="0a218-110">オンプレミスとクラウド ベースのアプリケーションまたはサービスとの間を通過するトラフィックは、監査できるように DMZ を通過する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0a218-110">Traffic passing between on-premises and cloud-based applications or services must pass through the DMZ where it can be audited.</span></span> <span data-ttu-id="0a218-111">VPN 接続および DMZ ネットワークによって許可されるトラフィックを決定する規則は、IT セキュリティ チームが厳密に管理します。</span><span class="sxs-lookup"><span data-stu-id="0a218-111">VPN connections and the rules determining what traffic is allowed through the DMZ network are strictly controlled by IT security teams.</span></span>

## <a name="cloud-dmz-assumptions"></a><span data-ttu-id="0a218-112">クラウド DMZ の前提条件</span><span class="sxs-lookup"><span data-stu-id="0a218-112">Cloud DMZ assumptions</span></span>

<span data-ttu-id="0a218-113">クラウド DMZ のデプロイでは、以下のことを前提とします。</span><span class="sxs-lookup"><span data-stu-id="0a218-113">Deploying a Cloud DMZ assumes the following:</span></span>

- <span data-ttu-id="0a218-114">セキュリティ チームが、オンプレミスとクラウド ベースのセキュリティ要件とポリシーを完全には調整していない。</span><span class="sxs-lookup"><span data-stu-id="0a218-114">Your security teams have not fully aligned on-premises and cloud-based security requirements and policies.</span></span>
- <span data-ttu-id="0a218-115">クラウド ベース ワークロードで、オンプレミスまたはサード パーティのネットワークでホストされているサービスへのアクセスを制限する必要がある。または、オンプレミス環境内のユーザーやアプリケーションでクラウド ホスト リソースへのアクセスを制限する必要がある。</span><span class="sxs-lookup"><span data-stu-id="0a218-115">Your cloud-based workloads require limited access to services hosted on your on-premises or third-party networks, or your users or applications in your on-premises environment need limited access to cloud-hosted resources.</span></span>
- <span data-ttu-id="0a218-116">オンプレミスのネットワークとクラウド プロバイダーの間に VPN 接続を実装することが、企業ポリシー、規制要件、または技術的な互換性の問題によって妨げられていない。</span><span class="sxs-lookup"><span data-stu-id="0a218-116">Implementing a VPN connection between your on-premises networks and cloud provider is not prevented by corporate policy, regulatory requirements, or technical compatibility issues.</span></span>
- <span data-ttu-id="0a218-117">ワークロードが、サブスクリプションのリソース制限をバイパスするために複数のサブスクリプションを必要としないか、複数のサブスクリプションを必要とするが、複数のサブスクリプションに分散するリソースによって使用される接続と共有サービスの中央管理を必要としない。</span><span class="sxs-lookup"><span data-stu-id="0a218-117">Your workloads either do not require multiple subscriptions to bypass subscription resource limits, or they involve multiple subscriptions but don't require central management of connectivity or shared services used by resources spread across multiple subscriptions.</span></span>

<span data-ttu-id="0a218-118">クラウド DMZ 仮想ネットワーク アーキテクチャの実装を考える際、クラウド導入チームは、次の問題を考慮する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0a218-118">Your Cloud Adoption team should consider the following issues when looking at implementing a Cloud DMZ virtual networking architecture:</span></span>

- <span data-ttu-id="0a218-119">オンプレミス ネットワークをクラウド ネットワークに接続すると、セキュリティ要件がより複雑になります。</span><span class="sxs-lookup"><span data-stu-id="0a218-119">Connecting on-premises networks with cloud networks increases the complexity of your security requirements.</span></span> <span data-ttu-id="0a218-120">クラウド ネットワークとオンプレミス環境の間の接続がセキュリティで保護されていても、クラウド リソースを確実に保護する必要があります。</span><span class="sxs-lookup"><span data-stu-id="0a218-120">Even though the connection between cloud networks and the on-premises environment are secured, you still need to ensure cloud resources are secured.</span></span>
- <span data-ttu-id="0a218-121">クラウドDMZ アーキテクチャは、接続をセキュリティでさらに保護し、オンプレミスとクラウド ネットワークの間でセキュリティ ポリシーを調整しながら、本格的なハイブリッド ネットワーク アーキテクチャを幅広く採用できるようにするための足がかりとして広く使用されています。</span><span class="sxs-lookup"><span data-stu-id="0a218-121">The Cloud DMZ architecture is commonly used as a stepping stone while connectivity is further secured and security policy aligned between on-premises and cloud networks, allowing a broader adoption of a full-scale hybrid networking architecture.</span></span>

## <a name="learn-more"></a><span data-ttu-id="0a218-122">詳細情報</span><span class="sxs-lookup"><span data-stu-id="0a218-122">Learn more</span></span>

<span data-ttu-id="0a218-123">Azure プラットフォームでのクラウド DMZ の実装の詳細については、次の記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="0a218-123">See the following for more information about the implementing a Cloud DMZ in the Azure platform.</span></span>

- <span data-ttu-id="0a218-124">[Azure とオンプレミス データ センター間に DMZ を実装する](../../../reference-architectures/dmz/secure-vnet-hybrid.md)。</span><span class="sxs-lookup"><span data-stu-id="0a218-124">[Implement a DMZ between Azure and your on-premises datacenter](../../../reference-architectures/dmz/secure-vnet-hybrid.md).</span></span> <span data-ttu-id="0a218-125">この記事では、安全なハイブリッド ネットワーク アーキテクチャを Azure に実装する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="0a218-125">This article discusses how to implement a secure hybrid network architecture in Azure.</span></span>
