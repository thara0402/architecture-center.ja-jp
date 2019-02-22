---
title: CAF:デジタル資産とは
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: デジタル資産とは
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: d23bdbdd98226e8b9cdb9bbb5fefa5a918a498d8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898427"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-what-is-a-digital-estate"></a><span data-ttu-id="5bf2b-103">CAF:デジタル資産とは</span><span class="sxs-lookup"><span data-stu-id="5bf2b-103">CAF: What is a digital estate?</span></span>

<span data-ttu-id="5bf2b-104">現代のどの企業にも、何らかの形式のデジタル資産があります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-104">Every modern company has some form of digital estate.</span></span> <span data-ttu-id="5bf2b-105">デジタル資産は、物理的な資産と同様に、形のある、所有されているいくつかの資産への抽象的な参照です。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-105">Much like a physical estate, a digital estate is an abstract reference to a number of tangible, owned assets.</span></span> <span data-ttu-id="5bf2b-106">デジタル資産では、これらの資産は、仮想マシン (VM)、サーバー、アプリケーション、データなどで構成されます。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-106">In a digital estate, those assets are comprised of virtual machines (VMs), servers, applications, data, and so on.</span></span> <span data-ttu-id="5bf2b-107">基本的に、デジタル資産は、ビジネス プロセスやサポートしている操作の機能を向上させる IT 資産のコレクションです。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-107">Essentially, a digital estate is the collection of IT assets that power business processes and supporting operations.</span></span>

<span data-ttu-id="5bf2b-108">デジタル資産の重要性は、デジタル変革の取り組みの計画および実行中に最も明らかになります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-108">The importance of a digital estate is most obvious during planning and execution of Digital Transformation efforts.</span></span> <span data-ttu-id="5bf2b-109">変革の取り組み中、デジタル資産は、クラウド戦略チームがビジネス成果をリリース計画や技術的な取り組みにどのようにマッピングするかを示します。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-109">During transformation journeys, the digital estate is how Cloud Strategy teams map the business outcomes to release plans and technical efforts.</span></span> <span data-ttu-id="5bf2b-110">それらはすべて、組織が今日所有するデジタル資産のインベントリおよび測定で始まります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-110">That all starts with an inventory and measurement of the digital assets the organization owns today.</span></span>

## <a name="how-can-a-digital-estate-be-measured"></a><span data-ttu-id="5bf2b-111">デジタル資産はどのように測定できるか</span><span class="sxs-lookup"><span data-stu-id="5bf2b-111">How can a digital estate be measured?</span></span>

<span data-ttu-id="5bf2b-112">デジタル資産の測定は、望ましいビジネス成果によって異なります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-112">The measurement of a digital estate changes depending on the desired business outcomes.</span></span>

- <span data-ttu-id="5bf2b-113">**インフラストラクチャの移行**。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-113">**Infrastructure migrations**.</span></span> <span data-ttu-id="5bf2b-114">組織が内向き指向であり、コスト、運用プロセス、俊敏性、または最適化操作のその他の側面を最適化しようとしている場合、デジタル資産は VM、サーバー、およびサポートしているワークロードに焦点を絞ります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-114">When an organization is inward facing and seeks to optimize cost, operational processes, agility, or other aspects of optimizing operations, the digital estate focuses on VMs, servers, and supporting workloads.</span></span>

- <span data-ttu-id="5bf2b-115">**アプリケーションのイノベーション**。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-115">**Application innovation**.</span></span> <span data-ttu-id="5bf2b-116">顧客中心の変革の場合は、焦点が少し異なります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-116">For customer-obsessed transformations, the lens is a bit different.</span></span> <span data-ttu-id="5bf2b-117">顧客をサポートするアプリケーション、API、およびトランザクション データに焦点を置く必要があります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-117">The focus should be placed in the applications, APIs, and transactional data that support the customers.</span></span> <span data-ttu-id="5bf2b-118">VM やネットワーク アプライアンスには多くの場合、それほど焦点は置かれません。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-118">VMs and network appliances are often of less focus.</span></span>

- <span data-ttu-id="5bf2b-119">**データ主導のイノベーション**。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-119">**Data-driven innovation**.</span></span> <span data-ttu-id="5bf2b-120">今日のデジタル主導の市場では、データの強力な基礎なくして、新しい製品またはサービスを発売することは困難です。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-120">In today's digitally driven market, it's difficult to launch a new product or service without a strong foundation in data.</span></span> <span data-ttu-id="5bf2b-121">破壊的な変革の間は、組織全体にわたるデータのサイロにより多くの焦点が置かれます。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-121">During disruptive transformation, the focus is more on the silos of data across the organization.</span></span>

<span data-ttu-id="5bf2b-122">組織が変革の最も重要な形式を理解すると、デジタル資産計画ははるかに管理しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-122">Once an organization understands the most important form of transformation, digital estate planning becomes much easier to manage.</span></span>

> [!TIP]
> <span data-ttu-id="5bf2b-123">各種類の変革は、3 つのビューのいずれでも測定できます。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-123">Each type of transformation could be measured with any of the three views.</span></span> <span data-ttu-id="5bf2b-124">さらに、企業が 3 つの変革をすべて並列に実行することが一般的です。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-124">Further, it is common for companies to execute all three transformations in parallel.</span></span> <span data-ttu-id="5bf2b-125">リーダーシップおよびクラウド戦略チームを、ビジネスの成功にとって最も重要な変革に関して堅固に整合させることを強くお勧めします。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-125">It is highly suggested that the leadership and Cloud Strategy team be firmly aligned regarding the transformation that is most important for business success.</span></span> <span data-ttu-id="5bf2b-126">その理解が、複数のイニシアチブにわたる共通の言語およびメトリックのための基盤として機能します。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-126">That understanding will serve as the basis for common language and metrics across multiple initiatives.</span></span>

## <a name="how-can-a-financial-model-be-updated-to-reflect-the-digital-estate"></a><span data-ttu-id="5bf2b-127">デジタル資産を反映するように財務モデルをどのように更新できるか</span><span class="sxs-lookup"><span data-stu-id="5bf2b-127">How can a financial model be updated to reflect the digital estate?</span></span>

<span data-ttu-id="5bf2b-128">デジタル資産の分析によって、クラウド導入アクティビティが促進されます。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-128">An analysis of the digital estate will drive cloud adoption activities.</span></span> <span data-ttu-id="5bf2b-129">また、クラウド原価計算モデルを提供することによって財務モデルにも通知され、それによりさらに投資収益率 (ROI) が向上します。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-129">It will also inform financial models by providing cloud costing models, which will in turn drive the return on investment (ROI).</span></span>

<span data-ttu-id="5bf2b-130">デジタル資産の分析を完了するには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-130">To complete the digital estate analysis, perform the following steps:</span></span>

1. [<span data-ttu-id="5bf2b-131">分析アプローチを決定する</span><span class="sxs-lookup"><span data-stu-id="5bf2b-131">Determine analysis approach</span></span>](approach.md)
1. [<span data-ttu-id="5bf2b-132">現在の状態のインベントリを収集する</span><span class="sxs-lookup"><span data-stu-id="5bf2b-132">Collect current state inventory</span></span>](inventory.md)
1. [<span data-ttu-id="5bf2b-133">デジタル資産内の資産を合理化する</span><span class="sxs-lookup"><span data-stu-id="5bf2b-133">Rationalize the assets in the digital estate</span></span>](rationalize.md)
1. [<span data-ttu-id="5bf2b-134">価格を計算するために資産をクラウド サービスに整合させる</span><span class="sxs-lookup"><span data-stu-id="5bf2b-134">Align assets to cloud offerings to calculate pricing</span></span>](calculate.md)

<span data-ttu-id="5bf2b-135">財務モデルや移行のバックログを、合理化され、価格が決定された資産を反映するように変更できます。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-135">Financial models and migration backlogs can be modified to reflect the rationalized and priced estate.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5bf2b-136">次の手順</span><span class="sxs-lookup"><span data-stu-id="5bf2b-136">Next steps</span></span>

<span data-ttu-id="5bf2b-137">デジタル資産計画を開始するには、使用するアプローチを決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="5bf2b-137">Before digital estate planning can begin, you must determine what approach to use.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="5bf2b-138">デジタル資産計画の手法</span><span class="sxs-lookup"><span data-stu-id="5bf2b-138">Approaches to digital estate planning</span></span>](approach.md)