---
title: 'CAF: ソフトウェア定義ネットワーク - クラウド ネイティブ'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ネイティブの仮想ネットワーク サービスの説明
author: rotycenh
ms.openlocfilehash: c6200491bc9ba35a9f00e0003e51716b58628980
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242153"
---
# <a name="software-defined-networks-cloud-native"></a><span data-ttu-id="63b61-103">ソフトウェア定義ネットワーク: クラウド ネイティブ</span><span class="sxs-lookup"><span data-stu-id="63b61-103">Software Defined Networks: Cloud native</span></span>

<span data-ttu-id="63b61-104">クラウド ネイティブ仮想ネットワークは、仮想マシンなどの IaaS リソースをクラウド プラットフォームにデプロイするときに必要です。</span><span class="sxs-lookup"><span data-stu-id="63b61-104">A cloud native virtual network is a required when deploying IaaS resources such as virtual machines to a cloud platform.</span></span> <span data-ttu-id="63b61-105">Web のような外部ソースから仮想ネットワークへのアクセスは、明示的にプロビジョニングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="63b61-105">Access to virtual networks from external sources, similar to the web, need to be explicitly provisioned.</span></span> <span data-ttu-id="63b61-106">これらの種類の仮想ネットワークでは、サブネット、ルーティング規則、および仮想ファイアウォール、およびトラフィック管理デバイスの作成がサポートされます。</span><span class="sxs-lookup"><span data-stu-id="63b61-106">These types of virtual networks support the creation of subnets, routing rules, and virtual firewall and traffic management devices.</span></span>

<span data-ttu-id="63b61-107">クラウド ネイティブ仮想ネットワークには、クラウドでホストされるワークロードをサポートするために、組織のオンプレミスまたはクラウド以外のリソースへの依存関係はありません。</span><span class="sxs-lookup"><span data-stu-id="63b61-107">A cloud native virtual network has no dependencies on your organization's on-premises or other non-cloud resources to support the cloud-hosted workloads.</span></span> <span data-ttu-id="63b61-108">必要なリソースはすべて、仮想ネットワーク自体またはマネージド PaaS オファリングを使用してプロビジョニングされます。</span><span class="sxs-lookup"><span data-stu-id="63b61-108">All required resources are provisioned either in the virtual network itself or by using managed PaaS offerings.</span></span>

## <a name="cloud-native-assumptions"></a><span data-ttu-id="63b61-109">クラウド ネイティブの前提条件</span><span class="sxs-lookup"><span data-stu-id="63b61-109">Cloud native assumptions</span></span>

<span data-ttu-id="63b61-110">クラウド ネイティブ仮想ネットワークのデプロイでは、以下のことが前提とされます。</span><span class="sxs-lookup"><span data-stu-id="63b61-110">Deploying a cloud native virtual network assumes the following:</span></span>

- <span data-ttu-id="63b61-111">仮想ネットワークにデプロイするワークロードに、オンプレミス ネットワーク内からのみアクセスできるアプリケーションまたはサービスへの依存関係はありません。</span><span class="sxs-lookup"><span data-stu-id="63b61-111">The workloads you deploy to the virtual network have no dependencies on applications or services that are accessible only from inside your on-premises network.</span></span> <span data-ttu-id="63b61-112">パブリック インターネット経由でアクセスできるエンドポイントを提供する場合を除き、オンプレミスで内部的にホストされるアプリケーションとサービスは、クラウド プラットフォームでホストされるリソースから使用することができません。</span><span class="sxs-lookup"><span data-stu-id="63b61-112">Unless they provide endpoints accessible over the public Internet, applications and services hosted internally on-premises are not usable by resources hosted on a cloud platform.</span></span>
- <span data-ttu-id="63b61-113">ワークロードの ID 管理とアクセス制御は、クラウド プラットフォームの ID サービスまたはクラウド環境でホストされている IaaS サーバーに依存します。</span><span class="sxs-lookup"><span data-stu-id="63b61-113">Your workload's identity management and access control depends on the cloud platform's identity services or IaaS servers hosted in your cloud environment.</span></span> <span data-ttu-id="63b61-114">オンプレミスまたはその他の外部の場所でホストされている ID サービスに直接接続する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="63b61-114">You will not need to directly connect to identity services hosted on-premises or other external locations.</span></span>
- <span data-ttu-id="63b61-115">ID サービスは、オンプレミス ディレクトリによってシングル サインオン (SSO) をサポートする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="63b61-115">Your identity services do not need to support single sign-on (SSO) with on-premises directories.</span></span>

<span data-ttu-id="63b61-116">クラウド ネイティブ仮想ネットワークに外部の依存関係はありません。</span><span class="sxs-lookup"><span data-stu-id="63b61-116">Cloud native virtual networks have no external dependencies.</span></span> <span data-ttu-id="63b61-117">このため、そのデプロイと構成は簡単になっていて、その結果として、このアーキテクチャは多くの場合、実験やその他の小規模な自己完結型または反復処理型のデプロイにとって最適な選択肢となっています。</span><span class="sxs-lookup"><span data-stu-id="63b61-117">This makes them simple to deploy and configure, and as a result this architecture is often the best choice for experiments or other smaller self-contained or rapidly iterating deployments.</span></span>

<span data-ttu-id="63b61-118">クラウド導入チームが、クラウド ネイティブ仮想ネットワーク アーキテクチャについて検討するときに考慮する必要のあるその他の問題には、以下が含まれます。</span><span class="sxs-lookup"><span data-stu-id="63b61-118">Additional issues your Cloud Adoption Team should consider when discussing a cloud native virtual networking architecture include:</span></span>

- <span data-ttu-id="63b61-119">オンプレミスのデータ センターで実行するように設計された既存のワークロードでは、ストレージや認証サービスなどのクラウド ベースの機能を活用するために、大がかりな変更が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="63b61-119">Existing workloads designed to run in an on-premises datacenter may need extensive modification to take advantage of cloud-based functionality, such as storage or authentication services.</span></span>
- <span data-ttu-id="63b61-120">クラウド ネイティブのネットワークは、クラウド プラットフォーム管理ツールを通じてのみ管理されるので、時間の経過とともに、管理とポリシーが既存の IT 標準とは異なる状況になる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="63b61-120">Cloud native networks are managed solely through the cloud platform management tools, and therefore may lead to management and policy divergence from your existing IT standards as time goes on.</span></span>

## <a name="learn-more"></a><span data-ttu-id="63b61-121">詳細情報</span><span class="sxs-lookup"><span data-stu-id="63b61-121">Learn more</span></span>

<span data-ttu-id="63b61-122">Azure プラットフォームでのクラウド ネイティブ仮想ネットワークの詳細については、以下を参照してください。</span><span class="sxs-lookup"><span data-stu-id="63b61-122">See the following for more information about cloud native virtual networking in the Azure platform.</span></span>

- <span data-ttu-id="63b61-123">[Azure Virtual Network: 攻略ガイド](/azure/virtual-network/virtual-network-vnet-plan-design-arm)。</span><span class="sxs-lookup"><span data-stu-id="63b61-123">[Azure Virtual Network: How-to guides](/azure/virtual-network/virtual-network-vnet-plan-design-arm).</span></span> <span data-ttu-id="63b61-124">新しく作成される Azure の仮想ネットワークは、既定でクラウド ネイティブとなります。</span><span class="sxs-lookup"><span data-stu-id="63b61-124">Newly created Azure Virtual Networks are cloud-native by default.</span></span> <span data-ttu-id="63b61-125">これらのガイドを使用すると、仮想ネットワークの設計とデプロイの計画に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="63b61-125">Use these guides to help plan the design and deployment of your virtual networks.</span></span>
- <span data-ttu-id="63b61-126">[サブスクリプションの制限: ネットワーク](/azure/azure-subscription-service-limits?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits) を選択します。</span><span class="sxs-lookup"><span data-stu-id="63b61-126">[Subscription limits: Networking](/azure/azure-subscription-service-limits?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits).</span></span> <span data-ttu-id="63b61-127">1 つのサブスクリプション内には、1 つの仮想ネットワークと、接続されたリソースのみが存在できます。これらはサブスクリプションの制限によってバインドされています。</span><span class="sxs-lookup"><span data-stu-id="63b61-127">Any single virtual network and connected resources can only exist within a single subscription, and are bound by subscription limits.</span></span>
