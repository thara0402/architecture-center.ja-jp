---
title: クラウド設計パターン
titleSuffix: Azure Architecture Center
description: Microsoft Azure のクラウド設計パターン
keywords: Azure
author: dragon119
ms.date: 12/10/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="cloud-design-patterns"></a><span data-ttu-id="8bc6c-104">クラウド設計パターン</span><span class="sxs-lookup"><span data-stu-id="8bc6c-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="8bc6c-105">これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションをクラウドに構築するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="8bc6c-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="8bc6c-106">パターンごとに、そのパターンで対処する問題、パターンの適用に関する考慮事項、Microsoft Azure に基づいた例を説明します。</span><span class="sxs-lookup"><span data-stu-id="8bc6c-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="8bc6c-107">ほとんどのパターンには、Azure でのパターンの実装方法を示すコード サンプルまたはスニペットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="8bc6c-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="8bc6c-108">ただし、パターンのほとんどは、ホストが Azure か他のクラウド プラットフォームかにかかわらず、分散システムに関連しています。</span><span class="sxs-lookup"><span data-stu-id="8bc6c-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="8bc6c-109">クラウドにおける問題点</span><span class="sxs-lookup"><span data-stu-id="8bc6c-109">Problem areas in the cloud</span></span>

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
<span data-ttu-id="8bc6c-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="8bc6c-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="8bc6c-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="8bc6c-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="8bc6c-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="8bc6c-112">{%- endfor %}</span></span>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a><span data-ttu-id="8bc6c-113">パターンのカタログ</span><span class="sxs-lookup"><span data-stu-id="8bc6c-113">Catalog of patterns</span></span>

| <span data-ttu-id="8bc6c-114">Pattern</span><span class="sxs-lookup"><span data-stu-id="8bc6c-114">Pattern</span></span> | <span data-ttu-id="8bc6c-115">まとめ</span><span class="sxs-lookup"><span data-stu-id="8bc6c-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="8bc6c-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="8bc6c-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
