---
title: 'CAF: データ分類とは'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: データ分類とは
author: BrianBlanchard
ms.openlocfilehash: bb40745b02e4ad6d7faa7054bf62443964f6784b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245143"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a>データ分類とは

これは、データ分類の一般的なトピックに関する入門記事です。 データ分類は、すべてのガバナンスにおいて非常に一般的な開始点です。

## <a name="business-risks-and-governance"></a>ビジネス上のリスクとガバナンス

ほとんどの組織で、ガバナンスに投資する主な理由は、次の 3 つのビジネス上のリスクに絞り込むことができます。

* データ侵害に関連する責任
* 障害によるビジネスの中断
* 計画されていない、または予期しない支出

これらの 3 つのビジネス上のリスクには多くの種類があります。 ただし、上記が最も一般的である傾向があります。

## <a name="understand-then-mitigate"></a>理解してから軽減する

リスクを軽減するには、その前にリスクを理解する必要があります。 データ侵害の責任の場合は、データの分類から理解を始めます。 データ分類は、デジタル資産内のすべての資産にメタ データ特性を関連付けるプロセスです。それにより、その資産に関連付けられているデータの種類を識別します。

Microsoft では、クラウドへの移行やデプロイの潜在的な候補として識別されている資産はすべて、データ分類、ビジネス上の重要度、および請求責任を記録するためのメタデータを文書化しておくことを推奨しています。 分類のこれら 3 つのポイントが、リスクの理解と軽減に大いに役立ちます。

## <a name="microsofts-data-classification"></a>Microsoft のデータ分類

Microsoft が使用する分類の一覧を以下に示します。 業界や既存のセキュリティ要件に応じて、データ分類の標準は、組織内に既に存在する場合があります。 標準が存在しない場合は、デジタル資産とリスク プロファイルをよりよく理解するために、このサンプル分類を使用することをお勧めします。  

* **ビジネス以外の場合:** Microsoft に属していない私生活のデータ
* **パブリック:** 自由に利用でき、公開用に承認されているビジネス データ
* **全般:** 一般の人々を対象としていないビジネス データ
* **機密:** 過剰に共有された場合に Microsoft に損害を与える可能性のあるビジネス データ
* **極秘:** 過剰に共有された場合に Microsoft に広範な損害を与える可能性のあるビジネス データ

## <a name="tagging-data-classification-in-azure"></a>Azure でのデータ分類のタグ付け

すべてのクラウド プロバイダーは、あらゆる資産に関するメタデータを記録するためのメカニズムを提供する必要があります。 メタデータは、クラウドでの資産の管理に不可欠です。 Azure の場合、リソース タグはメタデータのストレージの推奨される方法です。 Azure でのリソースのタグ付けの詳細については、[タグを使用した Azure リソースの整理](/azure/azure-resource-manager/resource-group-using-tags)に関する記事をご覧ください。

## <a name="next-steps"></a>次の手順

いずれかの「アクションにつながるガバナンス体験」の際にデータ分類を適用します。

> [!div class="nextstepaction"]
> [アクションにつながるガバナンス体験の開始](../journeys/overview.md)
