---
title: CAF:クラウド ガバナンスの 5 つの規範
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: CAF 内のガバナンス コンテンツの概要
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 80bdfc6f55b76fc3ae57d8fc25ce68c385eaa009
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241683"
---
# <a name="the-five-disciplines-of-cloud-governance"></a>クラウド ガバナンスの 5 つの規範

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
ビジネス プロセスまたは技術プラットフォームの変更にはリスクが伴います。 そのメンバーがクラウド管理人と呼ばれることもあるクラウド ガバナンス チームの任務は、これらのリスクを軽減し、導入やイノベーションの取り組みをできるだけ途切れさせないようにすることです。<br/><br/>CAF ガバナンス モデルでは、<a href="#corporate-policy">企業ポリシーの策定</a>と<a href="#disciplines-of-cloud-governance">クラウド ガバナンスの規範</a>に重点を置くことによって、(選択したクラウド プラットフォームに関係なく) これらの決定を導きます。 <a href="#actionable-journeys">アクションにつながる設計ガイド</a>は、Azure サービスを使用してこのモデルを実証します。 この記事は、CAF ガバナンス モデルの 5 つの規範のランディング ページとして機能します。
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
<i>図 1: 企業ポリシーとクラウド ガバナンスの 5 つの規範のダイアグラム</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="disciplines-of-cloud-governance"></a>クラウド ガバナンスの規範

個々のクラウド プロバイダーの枠を超えた共通のガバナンス規範が存在し、これは、ポリシーを周知したりツールチェーンを調整したりするためのガイドとして機能します。 これらの規範は、クラウド プロバイダーの枠を超えて適切なレベルで企業ポリシーを自動化および実施することに関する意思決定の指針です。

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsA">
<li style="display: flex; flex-direction: column;">
    <a href="./cost-management/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/cost-management.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cost Management</h3>
                        <p>コストはクラウド ユーザーの最大の関心事です。 すべてのクラウド プラットフォームに対応した、コスト管理のためのポリシーを策定します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./security-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/security-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>セキュリティ ベースライン</h3>
                        <p>セキュリティは複雑で個人的なトピックであり、会社ごとに独特です。 セキュリティ要件が確立されると、クラウド ガバナンスのポリシーと実施により、それらの要件がネットワーク、データ、資産の構成全体に適用されます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./identity-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/identity-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ID ベースライン</h3>
                        <p>ID 要件の適用が一貫性を欠いた場合、侵害のリスクが高まるおそれがあります。 ID ベースラインの規範は、クラウド導入の取り組み全体で ID が一貫して適用されることを保証する方法に焦点を当てています。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./resource-consistency/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/resource-consistency.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>リソースの整合性</h3>
                        <p>クラウドの運用はリソース構成の一貫性に依存します。 ガバナンス ツールを使用して、一貫した方法でリソースを構成し、オンボーディング、ドリフト、検出可能性、および復旧に関連したリスクを軽減することができます。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./deployment-acceleration/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/deployment-acceleration.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>デプロイ高速化</h3>
                        <p>デプロイと構成の集中化、標準化、一貫性のためのアプローチによってガバナンスのプラクティスを改善します。 クラウド ベースのガバナンス ツールからそれらを利用できれば、デプロイ アクティビティの迅速化に寄与するクラウドの要因となります。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->
