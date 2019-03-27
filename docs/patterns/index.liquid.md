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
ms.openlocfilehash: 4229531366f1b0c3257384694cf4358da9e63177
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241483"
---
# <a name="cloud-design-patterns"></a>クラウド設計パターン

[!INCLUDE [header](../../_includes/header.md)]

これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションをクラウドに構築するために役立ちます。

パターンごとに、そのパターンで対処する問題、パターンの適用に関する考慮事項、Microsoft Azure に基づいた例を説明します。 ほとんどのパターンには、Azure でのパターンの実装方法を示すコード サンプルまたはスニペットが含まれています。 ただし、パターンのほとんどは、ホストが Azure か他のクラウド プラットフォームかにかかわらず、分散システムに関連しています。

## <a name="problem-areas-in-the-cloud"></a>クラウドにおける問題点

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a>パターンのカタログ

| Pattern | まとめ |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
