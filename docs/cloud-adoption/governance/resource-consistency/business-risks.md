---
title: CAF:リソースの整合性の目的とビジネス上のリスク
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: リソースの整合性の目的とビジネス上のリスク
author: alexbuckgit
ms.openlocfilehash: 19e0d761e4afa3473099bde2edc960c8b9eadb79
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901665"
---
# <a name="resource-consistency-motivations-and-business-risks"></a><span data-ttu-id="5233d-103">リソースの整合性の目的とビジネス上のリスク</span><span class="sxs-lookup"><span data-stu-id="5233d-103">Resource Consistency motivations and business risks</span></span>

<span data-ttu-id="5233d-104">この記事では、お客様が通常、クラウド ガバナンス戦略でリソースの整合性の規範を採用する理由について説明します。</span><span class="sxs-lookup"><span data-stu-id="5233d-104">This article discusses the reasons that customers typically adopt a Resource Consistency discipline within a cloud governance strategy.</span></span> <span data-ttu-id="5233d-105">ポリシー ステートメントを促進する可能性のある潜在的なビジネス リスクの例をいくつかも挙げます。</span><span class="sxs-lookup"><span data-stu-id="5233d-105">It also provides a few examples of potential business risks that can drive policy statements.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="is-resource-consistency-relevant"></a><span data-ttu-id="5233d-106">リソース整合性の関連性について</span><span class="sxs-lookup"><span data-stu-id="5233d-106">Is Resource Consistency relevant?</span></span>

<span data-ttu-id="5233d-107">リソースとワークロードのデプロイに関しては、クラウドは従来のほとんどのオンプレミス データセンターよりも敏捷性と柔軟性を向上させます。</span><span class="sxs-lookup"><span data-stu-id="5233d-107">When it comes to deploying resources and workloads, the cloud offers increased agility and flexibility over most traditional on-premises datacenters.</span></span> <span data-ttu-id="5233d-108">ただし、このクラウド ベースの潜在的な利点は、クラウドの導入の成功を危うくする可能性のある潜在的な管理上の欠点と対になります。</span><span class="sxs-lookup"><span data-stu-id="5233d-108">However, these potential cloud-based advantages also come paired with potential management drawbacks that can seriously jeopardize the success of your cloud adoption.</span></span> <span data-ttu-id="5233d-109">デプロイしている資産は何か?</span><span class="sxs-lookup"><span data-stu-id="5233d-109">What assets have you deployed?</span></span> <span data-ttu-id="5233d-110">どのチームがどの資産を所有しているか?</span><span class="sxs-lookup"><span data-stu-id="5233d-110">What teams own what assets?</span></span> <span data-ttu-id="5233d-111">ワークロードをサポートしているリソースは十分であるか?</span><span class="sxs-lookup"><span data-stu-id="5233d-111">Do you have enough resources supporting a workload?</span></span> <span data-ttu-id="5233d-112">ワークロードが正常であるかどのように把握するか?</span><span class="sxs-lookup"><span data-stu-id="5233d-112">How do you know if workloads are healthy?</span></span>

<span data-ttu-id="5233d-113">リソースの整合性は、リソースが繰り返し一貫して、デプロイ、更新、構成されるようにし、サービスの中断を最小限に抑え、できるだけ短時間で修復されるようにするために不可欠です。</span><span class="sxs-lookup"><span data-stu-id="5233d-113">Resource Consistency is crucial to ensure that resources are deployed, updated, and configured consistently and repeatably, and that service disruptions are minimized and remedied in as little time as possible.</span></span>

<span data-ttu-id="5233d-114">リソースの整合性の規範は、クラウドのデプロイの運用側面に関連するビジネス リスクの識別と軽減に関係します。</span><span class="sxs-lookup"><span data-stu-id="5233d-114">The Resource Consistency discipline is concerned with identifying and mitigating business risks related to the operational aspects of your cloud deployment.</span></span> <span data-ttu-id="5233d-115">リソースの整合性には、アプリケーション、ワークロード、および資産のパフォーマンスの監視が含まれています。</span><span class="sxs-lookup"><span data-stu-id="5233d-115">Resource Consistency includes monitoring of applications, workloads, and asset performance.</span></span> <span data-ttu-id="5233d-116">スケール要求の充足、パフォーマンスのサービス レベル アグリーメント (SLA) 違反の修復、および自動修復によるパフォーマンス SLA 違反の事前回避を行うために必要なタスクも含まれています。</span><span class="sxs-lookup"><span data-stu-id="5233d-116">It also includes the tasks required to meet scale demands, remediate performance Service Level Agreement (SLA) violations, and proactively avoid performance SLA violations through automated remediation.</span></span>

<span data-ttu-id="5233d-117">初期テスト デプロイでは、リソースの整合性のニーズをサポートする簡単な名前付け基準およびタグ付け基準を採用する以外のことは必要ない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="5233d-117">Initial test deployments may not require much beyond adopting some cursory naming and tagging standards to support your Resource Consistency needs.</span></span> <span data-ttu-id="5233d-118">クラウドの導入が成熟し、より複雑でミッション クリティカルな資産を配置するにつれて、リソースの整合性の規範に投資する必要性が急速に高まります。</span><span class="sxs-lookup"><span data-stu-id="5233d-118">As your cloud adoption matures and you deploy more complicated and mission-critical assets, the need to invest in the Resource Consistency discipline increases rapidly.</span></span>

## <a name="business-risk"></a><span data-ttu-id="5233d-119">ビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="5233d-119">Business risk</span></span>

<span data-ttu-id="5233d-120">リソースの整合性の規範は、基本的な運用上のビジネス リスクに対処しようとします。</span><span class="sxs-lookup"><span data-stu-id="5233d-120">The Resource Consistency discipline attempts to address core operational business risks.</span></span> <span data-ttu-id="5233d-121">お客様の業務チームと IT チームと協力して、こうしたリスクを識別し、クラウドのデプロイを計画して実装したら、妥当性について各リスクを監視します。</span><span class="sxs-lookup"><span data-stu-id="5233d-121">Work with your business and IT teams to identify these risks and monitor each of them for relevance as you plan for and implement your cloud deployments.</span></span>

<span data-ttu-id="5233d-122">リスクは組織によって異なりますが、クラウド ガバナンス チーム内でディスカッションの開始点として使用できる一般的なリスクとして以下を利用できます。</span><span class="sxs-lookup"><span data-stu-id="5233d-122">Risks will differ between organization, but the following serve as common risks that you can use as a starting point for discussions within your Cloud Governance team:</span></span>

- <span data-ttu-id="5233d-123">**不要な運用コスト**。</span><span class="sxs-lookup"><span data-stu-id="5233d-123">**Unnecessary operational cost**.</span></span> <span data-ttu-id="5233d-124">古いリソースまたは未使用のリソース、または低需要時にオーバープロビジョニングされたリソースは、不必要な運用コストを増加させます。</span><span class="sxs-lookup"><span data-stu-id="5233d-124">Obsolete or unused resources, or resources that are overprovisioned during times of low demand, add unnecessary operational costs.</span></span>
- <span data-ttu-id="5233d-125">**プロビジョニング不足のリソース** 。</span><span class="sxs-lookup"><span data-stu-id="5233d-125">**Underprovisioned resources**.</span></span> <span data-ttu-id="5233d-126">予想よりも高い需要が発生したリソースは、クラウド リソースが需要によって圧迫されるため、ビジネスの中断を招く可能性があります。</span><span class="sxs-lookup"><span data-stu-id="5233d-126">Resources that experience higher than anticipated demand can result in business disruption as cloud resources are overwhelmed by demand.</span></span>
- <span data-ttu-id="5233d-127">**管理の非効率性**。</span><span class="sxs-lookup"><span data-stu-id="5233d-127">**Management inefficiencies**.</span></span> <span data-ttu-id="5233d-128">一貫性のある名前付けおよびタグ付けのメタデータがリソースに関連付けられていないと、IT スタッフが管理タスクに関するリソースを探したり、資産に関連する所有権および会計情報を特定したりするのが困難になります。</span><span class="sxs-lookup"><span data-stu-id="5233d-128">Lack of consistent naming and tagging metadata associated with resources can lead to IT staff having difficulty finding resources for management tasks or identifying ownership and accounting information related to assets.</span></span> <span data-ttu-id="5233d-129">その結果、コストが増大し、サービスの中断やその他の運用上の問題に対する IT の対応が遅くなる可能性がある管理上の非効率が生じます。</span><span class="sxs-lookup"><span data-stu-id="5233d-129">This results in management inefficiencies that can increase cost and slow IT responsiveness to service disruption or other operational issues.</span></span>
- <span data-ttu-id="5233d-130">**ビジネスの中断**。</span><span class="sxs-lookup"><span data-stu-id="5233d-130">**Business Interruption**.</span></span> <span data-ttu-id="5233d-131">組織で確立されたサービス レベル アグリーメント (SLA) の違反となるサービス中断は、事業の喪失やその他の財務上の影響を会社にもたらす可能性があります。</span><span class="sxs-lookup"><span data-stu-id="5233d-131">Service disruptions that result in violations of your organization's established Service Level Agreements (SLAs) can result in loss of business or other financial impacts to your company.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5233d-132">次の手順</span><span class="sxs-lookup"><span data-stu-id="5233d-132">Next steps</span></span>

<span data-ttu-id="5233d-133">[クラウド管理テンプレート](./template.md)使用して、現在のクラウド導入計画によって生じる可能性の高いビジネス上のリスクを文書化します。</span><span class="sxs-lookup"><span data-stu-id="5233d-133">Using the [Cloud Management Template](./template.md), document business risks that are likely to be introduced by the current cloud adoption plan.</span></span>

<span data-ttu-id="5233d-134">現実的なビジネス上のリスクについて確実に理解したら、次に、リスクに対するビジネスの許容値とその許容値を監視するインジケーターおよびキー メトリックについて文書化します。</span><span class="sxs-lookup"><span data-stu-id="5233d-134">Once an understanding of realistic business risks is established, the next step is to document the business's tolerance for risk and the indicators and key metrics to monitor that tolerance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="5233d-135">インジケーター、メトリック、およびリスク許容値の理解</span><span class="sxs-lookup"><span data-stu-id="5233d-135">Understand indicators, metrics, and risk tolerance</span></span>](./metrics-tolerance.md)
