---
title: 'CAF: 中小企業 - ガバナンス戦略の背景にある初期の企業ポリシー'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 中小企業 - ガバナンス戦略の背景にある初期の企業ポリシー
author: BrianBlanchard
ms.openlocfilehash: 2133145c9933ad450ea3cc55ecd68b8a667df783
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640024"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a>中小企業: ガバナンス戦略の背景にある初期の企業ポリシー

次の企業ポリシーでは、この体験の出発点である初期のガバナンス ポジションが定義されています。 この記事では、初期段階のリスク、初期ポリシー ステートメント、ポリシー ステートメントを適用するための初期プロセスを定義します。

> [!NOTE]
>企業ポリシーは技術文書ではありませんが、多くの技術的判断の根拠になります。 [概要](./overview.md)で説明するガバナンス MVP は、最終的にこのポリシーから派生します。 ガバナンス MVP を実装する前に、組織はその目標とビジネス リスクに基づいて企業ポリシーを策定する必要があります。

## <a name="cloud-governance-team"></a>クラウド ガバナンス チーム

この物語のクラウド管理チームは、ガバナンスのニーズを認識している 2 人のシステム管理者で構成されます。 これからの数か月間、2 人は会社のクラウド プレゼンスのガバナンスをクリーンアップするジョブを引き継ぎ、クラウド管理者という肩書を与えられるようになります。 今後の進化で、この肩書は変わる可能性があります。

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a>許容度指標

現在、リスクの許容度は高く、クラウド ガバナンスへの投資意欲は低い状態です。 このように、許容度指標は、時間とエネルギーの投資を増やす早期警告システムとして機能します。 以下の指標が観測された場合、ガバナンス戦略を進化させる必要があります。

- コスト管理:クラウドへのデプロイ規模が 100 資産を超えている、または毎月の支出が 1,000 米ドルを超えている。
- セキュリティ ベースライン:定義したクラウド導入計画に保護対象データが含まれている。
- リソースの整合性:定義したクラウド導入計画にミッションクリティカルなアプリケーションが含まれている。

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a>次の手順

この企業ポリシーにより、クラウド ガバナンス チームでは、導入のための基盤になるガバナンス MVP を実装する準備が整います。 次の手順はこの MVP の実装です。

> [!div class="nextstepaction"]
> [ベスト プラクティスの説明](./best-practice-explained.md)