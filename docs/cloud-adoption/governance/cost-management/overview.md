---
title: CAF:Cost Management の規範
description: クラウド ガバナンスと関連した Cost Management の説明
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: a86b1a75da57cec9c9aaf88abb70049f70731561
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243703"
---
# <a name="cost-management-discipline-overview"></a>Cost Management の規範の概要

Cost Management とは、[CAF ガバナンス モデル](../overview.md)の [5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 多くのお客様にとって、クラウド テクノロジを採用する際には、管理コストが大きな問題になります。 パフォーマンスに対する要求、導入ペース、およびクラウド サービス コストのバランスをとることが困難な場合があります。 これは、クラウド テクノロジを実装する大規模なビジネス変革の際に特に該当します。 このセクションでは、クラウド ガバナンス戦略の一環として Cost Management 規範を策定する方法について概要を説明します。  

> [!NOTE]
> Cost Management ガバナンスは、IT 関連コストに関する組織の財務管理に関連する既存のビジネス チーム、会計実務、および手順に代わるものではありません。 この規範の主な目的は、IT 支出に関連したクラウド関連の潜在的なリスクを識別し、クラウドのデプロイとその管理を担当するビジネス チームと IT チームにリスク軽減のガイダンスを提供することです。 ガバナンス ポリシーおよびプロセスの策定時には、行っている計画と確認のプロセスに、関係するビジネ ス チームと IT スタッフを関与させるようにしてください。

このガイドの主な対象読者は、組織のクラウド アーキテクトおよびクラウド ガバナンス チームのその他のメンバーです。 ただし、この規範を使用した決定、ポリシー、プロセスには、ビジネス チームと IT チームの該当メンバー (特にクラウド ベースのワークロードの所有、管理、支払いを担当するリーダー) が関与して、議論する必要があります。

## <a name="policy-statements"></a>ポリシー ステートメント

実践的なポリシー ステートメントと、その結果のアーキテクチャの要件は、Cost Management の規範の基盤となります。 ポリシー ステートメントのサンプルを参照するには、[Cost Management のポリシー ステートメント](./policy-statements.md)に関する記事をご覧ください。 これらのサンプルは、お客様の組織のガバナンス ポリシーの開始点として使用できます。

> [!CAUTION]
> このサンプル ポリシーは、よくあるお客様の体験をもとにしています。 特定のクラウド ガバナンスの要件にこれらのポリシーが合うように調整するには、次の手順を実行し、お客様独自のビジネス ニーズに合うポリシー ステートメントを作成してください。

## <a name="developing-cost-management-governance-policy-statements"></a>Cost Management のガバナンス ポリシー ステートメントの開発

次の 6 つの手順を使用して、環境内でコストを管理するためのガバナンス ポリシーを定義できます。

<!-- markdownlint-disable MD033 -->

<ul  class="panelContent cardsE">
<li style="display: flex; flex-direction: column;">
    <a href="./template.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-template.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Cost Management テンプレート</h3>
                        <p class="x-hidden-focus">Cost Management の規範を文書化するテンプレートをダウンロードする</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li><li style="display: flex; flex-direction: column;">
    <a href="./business-risks.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-risks.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>ビジネス リスク</h3>
                        <p class="x-hidden-focus">一般的に Cost Management の規範に関係する動機およびリスクを理解できます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./metrics-tolerance.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-metrics.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>インジケーターとメトリック</h3>
                        <p class="x-hidden-focus">Cost Management の規範に投資する最適なタイミングであるかどうかを知るインジケーターです。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./compliance-processes.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-enforce.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>ポリシー準拠プロセス</h3>
                        <p class="x-hidden-focus">Cost Management の規範でのポリシー準拠を確保するための推奨の手順です。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./discipline-improvement.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-maturity.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>成熟度</h3>
                        <p class="x-hidden-focus">クラウドの導入の段階とクラウド管理の成熟度を調整します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./toolchain.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-toolchain.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>ツールチェーン</h3>
                        <p class="x-hidden-focus">Cost Management の規範をサポートするために実装できる Azure サービスです。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>次の手順

特定の環境でビジネス上のリスクを評価する方法の概要を学びます。

> [!div class="nextstepaction"]
> [ビジネス上のリスクについての理解](./business-risks.md)

<!-- markdownlint-enable MD033 -->
