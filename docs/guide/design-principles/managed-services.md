---
title: 管理対象サービスの使用
titleSuffix: Azure Application Architecture Guide
description: なるべく、サービスとしてのインフラストラクチャ (IaaS) ではなく、サービスとしてのプラットフォーム (PaaS) を使用します。
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485810"
---
# <a name="use-managed-services"></a><span data-ttu-id="20358-103">管理対象サービスの使用</span><span class="sxs-lookup"><span data-stu-id="20358-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="20358-104">なるべく、サービスとしてのインフラストラクチャ (IaaS) ではなくサービスとしてのプラットフォーム (PaaS) を使用する</span><span class="sxs-lookup"><span data-stu-id="20358-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="20358-105">IaaS は部品箱のようなサービスです。</span><span class="sxs-lookup"><span data-stu-id="20358-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="20358-106">何でも構築できますが、自分で組み立てる必要があります。</span><span class="sxs-lookup"><span data-stu-id="20358-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="20358-107">管理対象サービスのほうが、構成や管理が簡単です。</span><span class="sxs-lookup"><span data-stu-id="20358-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="20358-108">VM をプロビジョニングしたり、Vnet を設定したり、修正プログラムや更新プログラムを管理する手間が省けるだけでなく、VM 上でのソフトウェア実行に関するその他すべてのオーバーヘッドが解消されます。</span><span class="sxs-lookup"><span data-stu-id="20358-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="20358-109">たとえば、アプリケーションでメッセージ キューを使用する必要があるとします。</span><span class="sxs-lookup"><span data-stu-id="20358-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="20358-110">その場合、RabbitMQ などのツールを使用して VM 上に独自のメッセージング サービスを設定することもできますが、</span><span class="sxs-lookup"><span data-stu-id="20358-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="20358-111">Azure Service Bus なら、信頼性のあるメッセージング機能が既にサービスとして提供されていますし、設定もより簡単です。</span><span class="sxs-lookup"><span data-stu-id="20358-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="20358-112">Service Bus 名前空間を作成し (これはデプロイ スクリプトの一部として実行できます)、 クライアント SDK を使用して Service Bus を呼び出すだけです。</span><span class="sxs-lookup"><span data-stu-id="20358-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span>

<span data-ttu-id="20358-113">もちろん、アプリケーションの要件によっては、IaaS アプローチのほうが適している場合もあるでしょう。</span><span class="sxs-lookup"><span data-stu-id="20358-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="20358-114">ただし、アプリケーションが IaaS ベースの場合でも、管理対象サービスを導入したほうが妥当な部分がないかどうかを確認するようにしてください。</span><span class="sxs-lookup"><span data-stu-id="20358-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="20358-115">具体的には、キャッシュ、キュー、データ ストレージなどについて確認を行ってください。</span><span class="sxs-lookup"><span data-stu-id="20358-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="20358-116">以下を実行する代わりに...</span><span class="sxs-lookup"><span data-stu-id="20358-116">Instead of running...</span></span> | <span data-ttu-id="20358-117">以下の使用を検討してください...</span><span class="sxs-lookup"><span data-stu-id="20358-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="20358-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="20358-118">Active Directory</span></span> | <span data-ttu-id="20358-119">Azure Active Directory Domain Services</span><span class="sxs-lookup"><span data-stu-id="20358-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="20358-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="20358-120">Elasticsearch</span></span> | <span data-ttu-id="20358-121">Azure Search</span><span class="sxs-lookup"><span data-stu-id="20358-121">Azure Search</span></span> |
| <span data-ttu-id="20358-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="20358-122">Hadoop</span></span> | <span data-ttu-id="20358-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="20358-123">HDInsight</span></span> |
| <span data-ttu-id="20358-124">IIS</span><span class="sxs-lookup"><span data-stu-id="20358-124">IIS</span></span> | <span data-ttu-id="20358-125">App Service</span><span class="sxs-lookup"><span data-stu-id="20358-125">App Service</span></span> |
| <span data-ttu-id="20358-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="20358-126">MongoDB</span></span> | <span data-ttu-id="20358-127">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="20358-127">Cosmos DB</span></span> |
| <span data-ttu-id="20358-128">Redis</span><span class="sxs-lookup"><span data-stu-id="20358-128">Redis</span></span> | <span data-ttu-id="20358-129">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="20358-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="20358-130">SQL Server</span><span class="sxs-lookup"><span data-stu-id="20358-130">SQL Server</span></span> | <span data-ttu-id="20358-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="20358-131">Azure SQL Database</span></span> |
