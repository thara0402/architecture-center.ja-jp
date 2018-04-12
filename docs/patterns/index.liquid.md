---
title: クラウド設計パターン
description: Microsoft Azure のクラウド設計パターン
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a>クラウド設計パターン

[!INCLUDE [header](../../_includes/header.md)]

これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションをクラウドに構築するために役立ちます。

パターンごとに、そのパターンで対処する問題、パターンの適用に関する考慮事項、Microsoft Azure に基づいた例を説明します。 ほとんどのパターンには、Azure でのパターンの実装方法を示すコード サンプルまたはスニペットが含まれています。 ただし、パターンのほとんどは、ホストが Azure か他のクラウド プラットフォームかにかかわらず、分散システムに関連しています。

## <a name="problem-areas-in-the-cloud"></a>クラウドにおける問題点

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>パターンのカタログ

| パターン | まとめ |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
