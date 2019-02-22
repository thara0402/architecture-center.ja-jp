---
title: CAF:ソフトウェア定義ネットワーク - PaaS のみ
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ベースのネットワーク機能のための PaaS のみモデルについて説明します
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901726"
---
# <a name="software-defined-networks-paas-only"></a><span data-ttu-id="598f8-103">ソフトウェア定義ネットワーク:PaaS のみ</span><span class="sxs-lookup"><span data-stu-id="598f8-103">Software Defined Networks: PaaS-only</span></span>

<span data-ttu-id="598f8-104">サービスとしてのプラットフォーム (PaaS) リソースを実装すると、デプロイ プロセスによって、負荷分散、ポートのブロック、他の PaaS サービスへの接続など、そのネットワークに対する少数の制御を持つ仮の基礎ネットワークが自動的に作成されます。</span><span class="sxs-lookup"><span data-stu-id="598f8-104">When you implement a platform as a service (PaaS) resource, the deployment process automatically creates an assumed underlying network with a limited number of controls over that network, including load balancing, port blocking, and connections to other PaaS services.</span></span>

<span data-ttu-id="598f8-105">Azure では、いくつかの PaaS リソースの種類を仮想ネットワークに[デプロイ](/azure/virtual-network/virtual-network-for-azure-services)または[接続](/azure/virtual-network/virtual-network-service-endpoints-overview)して、このようなリソースを既存の仮想ネットワーク インフラストラクチャと統合できます。</span><span class="sxs-lookup"><span data-stu-id="598f8-105">In Azure, several PaaS resource types can be [deployed into](/azure/virtual-network/virtual-network-for-azure-services) or [connected to](/azure/virtual-network/virtual-network-service-endpoints-overview) a virtual network, allowing these resources to integrate with your existing virtual networking infrastructure.</span></span> <span data-ttu-id="598f8-106">しかし、PaaS リソースがネイティブで提供するこのような既定のネットワーク機能のみに依存する PaaS のみのネットワーク アーキテクチャは、多くの場合、ワークロード要件を満たすのに十分です。</span><span class="sxs-lookup"><span data-stu-id="598f8-106">However, in many cases a PaaS only networking architecture, relying only on these default networking capabilities natively provided by PaaS resources, is sufficient to meet workload requirements.</span></span>

<span data-ttu-id="598f8-107">PaaS のみのネットワーク アーキテクチャを検討している場合は、必要な前提条件がお客様の要件と一致していることを確認してください。</span><span class="sxs-lookup"><span data-stu-id="598f8-107">If you are considering a PaaS only networking architecture, be sure you validate that the required assumptions align with your requirements.</span></span>

## <a name="paas-only-assumptions"></a><span data-ttu-id="598f8-108">PaaS のみの前提条件</span><span class="sxs-lookup"><span data-stu-id="598f8-108">PaaS-only assumptions</span></span>

<span data-ttu-id="598f8-109">PaaS のみのネットワーク アーキテクチャのデプロイは、以下を前提としています。</span><span class="sxs-lookup"><span data-stu-id="598f8-109">Deploying a PaaS-only networking architecture assumes the following:</span></span>

- <span data-ttu-id="598f8-110">デプロイされるアプリケーションはスタンドアロン アプリケーションであるか、他の PaaS リソースにのみ依存しています。</span><span class="sxs-lookup"><span data-stu-id="598f8-110">The application being deployed is a standalone application OR is dependent on only other PaaS resources.</span></span>
- <span data-ttu-id="598f8-111">お客様の IT 運用チームは、スタンドアロンの PaaS アプリケーションの管理、構成、デプロイをサポートするためにツール、トレーニング、プロセスを更新できます。</span><span class="sxs-lookup"><span data-stu-id="598f8-111">Your IT operations teams can update their tools, training, and processes to support management, configuration, and deployment of standalone PaaS applications.</span></span>
- <span data-ttu-id="598f8-112">PaaS アプリケーションは、IaaS リソースを含む、より広範なクラウド移行作業には含まれません。</span><span class="sxs-lookup"><span data-stu-id="598f8-112">The PaaS application is not part of a broader cloud migration effort that will include IaaS resources.</span></span>

<span data-ttu-id="598f8-113">このような前提条件は、PaaS のみのネットワークのデプロイに合わせた最低限の資格条件です。</span><span class="sxs-lookup"><span data-stu-id="598f8-113">These assumptions are minimum qualifiers aligned to deploying a PaaS-only network.</span></span> <span data-ttu-id="598f8-114">この方法は単一アプリケーションのデプロイの要件と一致する可能性がありますが、クラウド導入チームは次のような長期的な問題を検討する必要があります。</span><span class="sxs-lookup"><span data-stu-id="598f8-114">While this approach may align with the requirements of a single application deployment, your Cloud Adoption Team should examine these long-term questions:</span></span>

- <span data-ttu-id="598f8-115">このデプロイは、PaaS 以外の他のリソースへのアクセスを必要とする範囲または規模に拡大する予定ですか。</span><span class="sxs-lookup"><span data-stu-id="598f8-115">Will this deployment expand in scope or scale to require access to other non-PaaS resources?</span></span>
- <span data-ttu-id="598f8-116">現在のソリューション以外に他の PaaS のデプロイは計画されていますか。</span><span class="sxs-lookup"><span data-stu-id="598f8-116">Are other PaaS deployments planned beyond the current solution?</span></span>
- <span data-ttu-id="598f8-117">組織は将来の他のクラウド移行の計画を立てていますか。</span><span class="sxs-lookup"><span data-stu-id="598f8-117">Does the organization have plans for other future cloud migrations?</span></span>

<span data-ttu-id="598f8-118">このような問題に対する答えは、チームが PaaS のみの選択肢を選ぶことを妨げるものではありませんが、最終的な決定を下す前に検討する必要があります。</span><span class="sxs-lookup"><span data-stu-id="598f8-118">The answers to these questions would not preclude a team from choosing a PaaS only option but should be considered before making a final decision.</span></span>
