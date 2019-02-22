---
title: CAF:データ分類とは
description: データ分類とは
author: BrianBlanchard
ms.date: 2/11/2019
ms.openlocfilehash: 07268e7242d92ac2581bf28b378a3c43d166620c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901629"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a><span data-ttu-id="4c16b-103">データ分類とは</span><span class="sxs-lookup"><span data-stu-id="4c16b-103">What is data classification?</span></span>

<span data-ttu-id="4c16b-104">これは、データ分類の一般的なトピックに関する入門記事です。</span><span class="sxs-lookup"><span data-stu-id="4c16b-104">This is an introductory article on the general topic of Data Classification.</span></span> <span data-ttu-id="4c16b-105">データ分類は、すべてのガバナンスにおいて非常に一般的な開始点です。</span><span class="sxs-lookup"><span data-stu-id="4c16b-105">Data classification is a very common starting point for all governance.</span></span>

## <a name="business-risks-and-governance"></a><span data-ttu-id="4c16b-106">ビジネス上のリスクとガバナンス</span><span class="sxs-lookup"><span data-stu-id="4c16b-106">Business risks and governance</span></span>

<span data-ttu-id="4c16b-107">ほとんどの組織で、ガバナンスに投資する主な理由は、次の 3 つのビジネス上のリスクに絞り込むことができます。</span><span class="sxs-lookup"><span data-stu-id="4c16b-107">In most organizations, the primary reasons for investing in governance can be reduced to three business risks:</span></span>

* <span data-ttu-id="4c16b-108">データ侵害に関連する責任</span><span class="sxs-lookup"><span data-stu-id="4c16b-108">Liability associated with data breaches</span></span>
* <span data-ttu-id="4c16b-109">障害によるビジネスの中断</span><span class="sxs-lookup"><span data-stu-id="4c16b-109">Interruption to the business from outages</span></span>
* <span data-ttu-id="4c16b-110">計画されていない、または予期しない支出</span><span class="sxs-lookup"><span data-stu-id="4c16b-110">Unplanned or unexpected spending</span></span>

<span data-ttu-id="4c16b-111">これらの 3 つのビジネス上のリスクには多くの種類があります。</span><span class="sxs-lookup"><span data-stu-id="4c16b-111">There are many variants of these three business risks.</span></span> <span data-ttu-id="4c16b-112">しかし、上記が最も一般的である傾向があります。</span><span class="sxs-lookup"><span data-stu-id="4c16b-112">However, the tend to be the most common.</span></span>

## <a name="understand-then-mitigate"></a><span data-ttu-id="4c16b-113">理解してから軽減する</span><span class="sxs-lookup"><span data-stu-id="4c16b-113">Understand then mitigate</span></span>

<span data-ttu-id="4c16b-114">リスクを軽減するには、その前にリスクを理解する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4c16b-114">Before any risk can be mitigated, it must be understood.</span></span> <span data-ttu-id="4c16b-115">データ侵害の責任の場合は、データの分類から理解を始めます。</span><span class="sxs-lookup"><span data-stu-id="4c16b-115">In the case of data breach liability, that understanding starts with data classification.</span></span> <span data-ttu-id="4c16b-116">データ分類は、デジタル資産内のすべての資産にメタ データ特性を関連付けるプロセスです。それにより、その資産に関連付けられているデータの種類を識別します。</span><span class="sxs-lookup"><span data-stu-id="4c16b-116">Data classification is the process of associating a meta data characteristic to every asset in a digital estate, which identifies the type of data associated with that asset.</span></span>

<span data-ttu-id="4c16b-117">Microsoft では、クラウドへの移行やデプロイの潜在的な候補として識別されている資産はすべて、データ分類、ビジネス上の重要度、および請求責任を記録するためのメタデータを文書化しておくことを推奨しています。</span><span class="sxs-lookup"><span data-stu-id="4c16b-117">Microsoft suggests that any asset which has been identified as a potential candidate for migration or deployment to the cloud should have documented meta data to record the data classification, business criticality, and billing responsibility.</span></span> <span data-ttu-id="4c16b-118">分類のこれら 3 つのポイントが、リスクの理解と軽減に大いに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="4c16b-118">These three points of classification can go a long way to understanding and mitigating risks.</span></span>

## <a name="microsofts-data-classification"></a><span data-ttu-id="4c16b-119">Microsoft のデータ分類</span><span class="sxs-lookup"><span data-stu-id="4c16b-119">Microsoft's data classification</span></span>

<span data-ttu-id="4c16b-120">Microsoft が使用する分類の一覧を以下に示します。</span><span class="sxs-lookup"><span data-stu-id="4c16b-120">The following is a list of classifications Microsoft uses.</span></span> <span data-ttu-id="4c16b-121">業界や既存のセキュリティ要件に応じて、データ分類の標準は、組織内に既に存在する場合があります。</span><span class="sxs-lookup"><span data-stu-id="4c16b-121">Depending on your industry or existing security requirements, data classifications standards may already exist within your organization.</span></span> <span data-ttu-id="4c16b-122">標準が存在しない場合は、デジタル資産とリスク プロファイルをよりよく理解するために、このサンプル分類を使用することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="4c16b-122">If no standard exists, we welcome you to use this sample classification, to help you better understand your digital estate and risk profile.</span></span>  

* <span data-ttu-id="4c16b-123">**ビジネス以外の場合:** Microsoft に属していない私生活のデータ</span><span class="sxs-lookup"><span data-stu-id="4c16b-123">**Non-Business:** Data from your personal life that does not belong to Microsoft</span></span>
* <span data-ttu-id="4c16b-124">**パブリック:** 自由に利用でき、公開用に承認されているビジネス データ</span><span class="sxs-lookup"><span data-stu-id="4c16b-124">**Public:** Business data that is freely available and approved for public consumption</span></span>
* <span data-ttu-id="4c16b-125">**全般:** 一般の人々を対象としていないビジネス データ</span><span class="sxs-lookup"><span data-stu-id="4c16b-125">**General:** Business data that is not meant for a public audience</span></span>
* <span data-ttu-id="4c16b-126">**機密:** 過剰に共有された場合に Microsoft に損害を与える可能性のあるビジネス データ</span><span class="sxs-lookup"><span data-stu-id="4c16b-126">**Confidential:** Business data that could cause harm to Microsoft if over-shared</span></span>
* <span data-ttu-id="4c16b-127">**極秘:** 過剰に共有された場合に Microsoft に広範な損害を与える可能性のあるビジネス データ</span><span class="sxs-lookup"><span data-stu-id="4c16b-127">**Highly Confidential:** Business data that would cause extensive harm to Microsoft if over-shared</span></span>

## <a name="tagging-data-classification-in-azure"></a><span data-ttu-id="4c16b-128">Azure でのデータ分類のタグ付け</span><span class="sxs-lookup"><span data-stu-id="4c16b-128">Tagging data classification in Azure</span></span>

<span data-ttu-id="4c16b-129">すべてのクラウド プロバイダーは、あらゆる資産に関するメタデータを記録するためのメカニズムを提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="4c16b-129">Every cloud provider should offer a mechanism for recording metadata about any asset.</span></span> <span data-ttu-id="4c16b-130">メタデータは、クラウドでの資産の管理に不可欠です。</span><span class="sxs-lookup"><span data-stu-id="4c16b-130">Metadata is vital to managing assets in the cloud.</span></span> <span data-ttu-id="4c16b-131">Azure の場合、リソース タグはメタデータのストレージの推奨される方法です。</span><span class="sxs-lookup"><span data-stu-id="4c16b-131">In the case of Azure, resource tags are the suggested approach for metadata storage.</span></span> <span data-ttu-id="4c16b-132">Azure でのリソースのタグ付けの詳細については、[タグを使用した Azure リソースの整理](/azure/azure-resource-manager/resource-group-using-tags)に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="4c16b-132">For additional information on resource tagging in Azure, see the article on [Using tags to organize your Azure resources](/azure/azure-resource-manager/resource-group-using-tags).</span></span>

## <a name="next-steps"></a><span data-ttu-id="4c16b-133">次の手順</span><span class="sxs-lookup"><span data-stu-id="4c16b-133">Next steps</span></span>

<span data-ttu-id="4c16b-134">いずれかの「アクションにつながるガバナンス体験」の際にデータ分類を適用します。</span><span class="sxs-lookup"><span data-stu-id="4c16b-134">Apply data classifications during one of the actionable governance journeys.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="4c16b-135">アクションにつながるガバナンス体験の開始</span><span class="sxs-lookup"><span data-stu-id="4c16b-135">Begin an Actionable Governance Journeys</span></span>](../journeys/overview.md)
