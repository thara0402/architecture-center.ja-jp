---
title: "クラウドの設計パターン"
description: "Microsoft Azure のクラウド設計パターン"
keywords: Azure
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="c1de3-104">クラウドの設計パターン</span><span class="sxs-lookup"><span data-stu-id="c1de3-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="c1de3-105">これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションをクラウドに構築するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="c1de3-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="c1de3-106">パターンごとに、そのパターンで対処する問題、パターンの適用に関する考慮事項、Microsoft Azure に基づいた例を説明します。</span><span class="sxs-lookup"><span data-stu-id="c1de3-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="c1de3-107">ほとんどのパターンには、Azure でのパターンの実装方法を示すコード サンプルまたはスニペットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="c1de3-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="c1de3-108">ただし、パターンのほとんどは、ホストが Azure か他のクラウド プラットフォームかにかかわらず、分散システムに関連しています。</span><span class="sxs-lookup"><span data-stu-id="c1de3-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="c1de3-109">クラウドにおける問題点</span><span class="sxs-lookup"><span data-stu-id="c1de3-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="c1de3-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="c1de3-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="c1de3-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="c1de3-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="c1de3-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="c1de3-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="c1de3-113">パターンのカタログ</span><span class="sxs-lookup"><span data-stu-id="c1de3-113">Catalog of patterns</span></span>

| <span data-ttu-id="c1de3-114">パターン</span><span class="sxs-lookup"><span data-stu-id="c1de3-114">Pattern</span></span> | <span data-ttu-id="c1de3-115">概要</span><span class="sxs-lookup"><span data-stu-id="c1de3-115">Summary</span></span> |
| ------- | ------- |
<span data-ttu-id="c1de3-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="c1de3-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>