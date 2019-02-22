---
title: 'CAF: 設計ガイドをポリシーと整合させる。'
description: 設計ガイドをポリシーと整合させる方法
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901297"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a>設計ガイドをポリシーと整合させる方法

[特定されたリスク](understanding-business-risk.md)に基づいて[クラウド ポリシーを定義](define-policy.md)したら、IT スタッフと開発者が参照する、これらのポリシーに合致した実践的なガイダンスを作成する必要があります。 クラウド ガバナンス設計ガイドの素案を作成することで、5 つのガバナンス規範それぞれに対して作成されるポリシー ステートメントに基づいた構造上、技術上、プロセス上の選択肢を指定することができます。

クラウド ガバナンス設計ガイドでは、実際のポリシー要件に最適なクラウド デプロイのコア インフラストラクチャ コンポーネントそれぞれについて、アーキテクチャの選択肢と設計パターンを確立する必要があります。 これらと共に、こうした設計上の各決定事項をサポートするテクノロジ、ツール、およびプロセスの概要説明を含める必要があります。

リスク分析とポリシー ステートメントは、ある程度クラウド プラットフォームに依存しない場合もありますが、設計ガイドでは、IT および開発部門が必要とするプラットフォーム固有の実装の詳細を提供する必要があります。 設計上の決定を下し、ガイダンスを提供するときには、選択したプラットフォームのアーキテクチャ、ツール、および、機能に焦点を合わせます。

クラウド設計ガイドでは、各インフラストラクチャ コンポーネントに関連する技術的詳細の一部を考慮に入れる必要がありますが、ガイドは広範囲にわたる技術ドキュメントや仕様書となるものではありません。 ガイドでは、すべてのポリシー ステートメントが網羅されていて、スタッフが容易に理解して参照できる形式で、設計の決定内容が明確に書かれていることを確認します。

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a>実践的ガバナンス体験の利用

クラウド導入のために Azure プラットフォームを使用する予定の場合は、CAF が、CAF ガバナンス モデルの増分型アプローチについて説明している[ガバナンス体験](../journeys/overview.md)を提供しています。 ストーリー型のこれらの体験では、ビジネス リスク、許容の要件、ポリシー ステートメントを含む一般的な導入シナリオが取り上げられていています。これらが、ガバナンスにおける "実用最小限の製品" (MVP) を作り出すことになります。 これらの体験は、実世界で顧客が経験する Azure でのクラウド導入プロセスの全体を表しています。

どのクラウド導入にも固有の目標、優先度、課題がありますが、これらのサンプルは、実際のポリシーをガイダンスに移し替えるための適切なテンプレートとなるはずです。 開始点として、実際の状況に最も近いシナリオを選択し、ポリシーの具体的なニーズに合うように変更します。

## <a name="next-steps"></a>次の手順

設計ガイダンスを用意したので、ポリシー コンプライアンスを確保するためのポリシー準拠プロセスを確立します。

> [!div class="nextstepaction"]
> [ポリシー準拠プロセス](processes.md)