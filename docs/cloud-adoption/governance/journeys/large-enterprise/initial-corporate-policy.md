---
title: 'CAF: 大企業 - ガバナンス戦略の背景にある初期の企業ポリシー'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 大企業 - ガバナンス戦略の背景にある初期の企業ポリシー。
author: BrianBlanchard
ms.openlocfilehash: 51b709b3ae6d704a229a08611a3cbc0da974c6c2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241160"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a>大企業:ガバナンス戦略の背景にある初期の企業ポリシー

次の企業ポリシーは、この体験の出発点である初期のガバナンス ポジションを定義しています。 この記事では、初期段階のリスク、初期ポリシー ステートメント、ポリシー ステートメントを適用するための初期プロセスを定義します。

> [!NOTE]
>企業ポリシーは技術文書ではありませんが、多くの技術的判断の根拠になります。 [概要](./overview.md)で説明するガバナンス MVP は、最終的にこのポリシーから派生します。 ガバナンス MVP を実装する前に、組織はその目標とビジネス リスクに基づいて企業ポリシーを策定する必要があります。

## <a name="cloud-governance-team"></a>クラウド ガバナンス チーム

CIO は最近、PII とミッションクリティカル ポリシーの歴史を理解し、それらのポリシーを変更した場合の影響をレビューするために、IT ガバナンス チームとの会議を開きました。 IT と企業にとってのクラウドの可能性全般についても議論しました。

会議後、IT ガバナンス チームの 2 人のメンバーが、クラウド計画の取り組みを調査およびサポートするための許可を求めました。 ガバナンスの必要性とシャドウ IT を制限する機会を認識していた IT ガバナンス責任者はこのアイデアを支持しました。 このような経緯で、クラウド ガバナンス チームが誕生しました。 このチームはこれから数か月間、クラウドの検証過程で経験する多くの失敗について、ガバナンスの観点からクリーンアップを継承していきます。 これにより "クラウド管理人" の呼び名を得ることになります。 今後の進化の中で、この体験におけるチームの役割は刻々と変わっていきます。

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a>許容度指標

現在、リスク許容度は高く、クラウド ガバナンスへの投資意欲は低い状態です。 このように、許容度指標は、時間とエネルギーの投資を誘発する早期警告システムとして機能します。 以下の指標が観測された場合、ガバナンス戦略を進化させることが賢明です。

- コスト管理:クラウドへのデプロイ規模が 1,000 資産を超えている、または毎月の支出が 10,000 米ドルを超えている。
- ID ベースライン:レガシまたはサード パーティの多要素認証 (MFA) 要件を持つアプリケーションが含まれている。
- セキュリティ ベースライン:定義したクラウド導入計画に保護対象データが含まれている。
- リソースの整合性:定義したクラウド導入計画にミッションクリティカルなアプリケーションが含まれている。

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a>次の手順

この企業ポリシーにより、クラウド ガバナンス チームでは、導入のための基盤になるガバナンス MVP を実装する準備が整います。 次の手順はこの MVP の実装です。

> [!div class="nextstepaction"]
> [ベスト プラクティスの説明](./best-practice-explained.md)