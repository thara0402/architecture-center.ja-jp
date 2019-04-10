---
title: CAF:クラウド ガバナンス戦略の実装
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Azure 向けの Microsoft Cloud 導入フレームワーク (CAF) を使用してクラウド ガバナンス戦略を実装する方法について説明します。
author: BrianBlanchard
ms.date: 01/03/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 9da73502ad3a6243572128b5bf72399aad648353
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241254"
---
# <a name="implement-a-cloud-governance-strategy"></a>クラウド ガバナンス戦略の実装

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
ビジネス プロセスやテクノロジ プラットフォームに対するすべての変更は、ビジネス上のリスクを伴います。 クラウド カストディアンと呼ばれることもあるメンバーで構成されるクラウド ガバナンス チームの任務は、導入やイノベーションの取り組みをできるだけ途切れさせずに、こうしたリスクを軽減することです。<br/><br/>ただし、クラウド ガバナンスには、技術的な実装以上のことが求められます。 企業のナラティブや企業ポリシーの些細な変更により、導入の取り組みに大幅な影響が及ぼされる場合があります。 実装の前に、企業ポリシーの定義において IT の今後を見据えることが重要です。<br/><br/>
                </div>
            </div>
        </div>
    </div>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../_images/operational-transformation-govern-highres.png" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize">
            <div class="cardPadding" style="padding-bottom:10px;">
                <div class="card" style="padding-bottom:10px;">
                    <div class="cardText" style="padding-left:0px;">
<img src="../_images/operational-transformation-govern-highres.png" alt="Diagram of the CAF governance model: Corporate policy and governance disciplines">
<br>
<i>図 1: 企業ポリシーとクラウド ガバナンスの 5 つの規範</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="define-corporate-policy"></a>企業ポリシーの定義

企業ポリシーの定義では、クラウド プラットフォームにかかわらず、ビジネス上のリスクを特定して軽減することに注力します。 正常なクラウド ガバナンス戦略は、健全な企業ポリシーを定義することから開始します。 以下の 3 段階のプロセスでは、そのようなポリシーの反復開発について示します。

<!-- markdownlint-disable MD033 -->

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/understanding-business-risk.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/business-risk.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ビジネス リスク</h3>
                        <p>現在のクラウド導入計画とデータ分類を調査し、ビジネス上のリスクを特定します。 企業と協力し、リスク許容度とリスク軽減コストのバランスを取ります。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/define-policy.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/corporate-policy.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ポリシーとコンプライアンス</h3>
                        <p>リスク許容度を評価し、クラウドの導入とリスク軽減のための最も影響が少ないポリシーを通知します。 業界によっては、サードパーティのコンプライアンスが最初のポリシー作成に影響を及ぼします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/processes.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/enforcement.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>処理</h3>
                        <p>導入の速度とイノベーションのアクティビティにより、自然とポリシー違反が発生します。 関連プロセスの実行により、ポリシーへの準拠を監視して強制できるようにします。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>次の手順

健全なクラウド ガバナンス戦略は、ビジネス リスクを理解することから始まります。

> [!div class="nextstepaction"]
> [ビジネス リスクの理解](./policy-compliance/understanding-business-risk.md)
