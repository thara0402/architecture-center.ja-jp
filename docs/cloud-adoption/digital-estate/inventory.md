---
title: デジタル資産のインベントリ データを収集する
titleSuffix: Enterprise Cloud Adoption
description: デジタル資産のインベントリを作成する方法
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: c8c2671a76ab09ab3f817872fb9d78cf84d7daa5
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179656"
---
# <a name="enterprise-cloud-adoption-gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="a5422-103">エンタープライズ クラウドの導入:デジタル資産のインベントリ データを収集する</span><span class="sxs-lookup"><span data-stu-id="a5422-103">Enterprise Cloud Adoption: Gather inventory data for a digital estate</span></span>

<span data-ttu-id="a5422-104">インベントリを作成することは、[デジタル資産計画](overview.md)の最初の手順です。</span><span class="sxs-lookup"><span data-stu-id="a5422-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="a5422-105">このプロセスでは、後で分析や合理化が可能なように、特定のビジネス機能をサポートする IT 資産の一覧が収集されます。</span><span class="sxs-lookup"><span data-stu-id="a5422-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="a5422-106">この記事では、計画の必要上、分析にはボトムアップの手法を採用することが最適だと想定しています。</span><span class="sxs-lookup"><span data-stu-id="a5422-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="a5422-107">詳細については、[デジタル資産計画の手法](./approach.md)についての記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="a5422-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="how-can-a-digital-estate-be-inventoried"></a><span data-ttu-id="a5422-108">デジタル資産はどのようにインベントリ化できるか</span><span class="sxs-lookup"><span data-stu-id="a5422-108">How can a digital estate be inventoried?</span></span>

<span data-ttu-id="a5422-109">デジタル資産をサポートするインベントリは、目的としているデジタル変革と、それに対応する変革の過程に応じて変わります。</span><span class="sxs-lookup"><span data-stu-id="a5422-109">The inventory supporting a digital estate changes depending on the desired Digital Transformation and corresponding Transformation Journey.</span></span>

- <span data-ttu-id="a5422-110">事業の変革:事業の変革の際には、多くの場合、すべての VM とサーバーの一元的な一覧を作成できるスキャン ツールからインベントリを収集することが推奨されます。</span><span class="sxs-lookup"><span data-stu-id="a5422-110">Operational Transformation: During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="a5422-111">一部のツールでは、ネットワーク マッピングや依存関係も作成できます。これは、ワークロードの配置を定義するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="a5422-111">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="a5422-112">増分型の変革:増分型の変革でのインベントリは、顧客と共に始まります。</span><span class="sxs-lookup"><span data-stu-id="a5422-112">Incremental Transformation: Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="a5422-113">開始から終了まで顧客のエクスペリエンスをマッピングすることが、良い開始点となります。</span><span class="sxs-lookup"><span data-stu-id="a5422-113">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="a5422-114">そのマップをアプリケーション、API、データ、およびその他の資産に対応付けることで、分析用の詳細なインベントリが作成されます。</span><span class="sxs-lookup"><span data-stu-id="a5422-114">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="a5422-115">中断を伴う変革:中断を伴う変革では、製品やサービスに重点が置かれます。</span><span class="sxs-lookup"><span data-stu-id="a5422-115">Disruptive Transformation: Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="a5422-116">そうした観点から、インベントリには、市場や必要とされる機能に中断をもたらす機会のマッピングが含められます。</span><span class="sxs-lookup"><span data-stu-id="a5422-116">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="a5422-117">インベントリの精度と完全性</span><span class="sxs-lookup"><span data-stu-id="a5422-117">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="a5422-118">インベントリは、初回の実施時に完全であることはめったにありません。</span><span class="sxs-lookup"><span data-stu-id="a5422-118">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="a5422-119">クラウド戦略チームのさまざまなメンバーが、利害関係者とパワー ユーザーの足並みを揃えさせて、インベントリを検証することを強くお勧めします。</span><span class="sxs-lookup"><span data-stu-id="a5422-119">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="a5422-120">可能であれば、ネットワークと依存関係の分析などのその他のツールを利用すると、トラフィックの送信先になっているがインベントリに含まれていない資産を識別できます。</span><span class="sxs-lookup"><span data-stu-id="a5422-120">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a5422-121">次の手順</span><span class="sxs-lookup"><span data-stu-id="a5422-121">Next steps</span></span>

<span data-ttu-id="a5422-122">インベントリの作成と検証が終わったら、インベントリを合理化できます。</span><span class="sxs-lookup"><span data-stu-id="a5422-122">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="a5422-123">インベントリの合理化は、デジタル資産計画の次の手順です。</span><span class="sxs-lookup"><span data-stu-id="a5422-123">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="a5422-124">デジタル資産を合理化する</span><span class="sxs-lookup"><span data-stu-id="a5422-124">Rationalize the digital estate</span></span>](rationalize.md)