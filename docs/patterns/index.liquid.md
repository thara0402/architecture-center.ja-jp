---
title: クラウド設計パターン
description: Microsoft Azure のクラウド設計パターン
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848259"
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="cd235-104">クラウド設計パターン</span><span class="sxs-lookup"><span data-stu-id="cd235-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="cd235-105">これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションをクラウドに構築するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="cd235-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="cd235-106">パターンごとに、そのパターンで対処する問題、パターンの適用に関する考慮事項、Microsoft Azure に基づいた例を説明します。</span><span class="sxs-lookup"><span data-stu-id="cd235-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="cd235-107">ほとんどのパターンには、Azure でのパターンの実装方法を示すコード サンプルまたはスニペットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="cd235-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="cd235-108">ただし、パターンのほとんどは、ホストが Azure か他のクラウド プラットフォームかにかかわらず、分散システムに関連しています。</span><span class="sxs-lookup"><span data-stu-id="cd235-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="cd235-109">クラウドにおける問題点</span><span class="sxs-lookup"><span data-stu-id="cd235-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="cd235-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="cd235-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="cd235-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="cd235-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="cd235-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="cd235-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="cd235-113">パターンのカタログ</span><span class="sxs-lookup"><span data-stu-id="cd235-113">Catalog of patterns</span></span>

| <span data-ttu-id="cd235-114">パターン</span><span class="sxs-lookup"><span data-stu-id="cd235-114">Pattern</span></span> | <span data-ttu-id="cd235-115">まとめ</span><span class="sxs-lookup"><span data-stu-id="cd235-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="cd235-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="cd235-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
