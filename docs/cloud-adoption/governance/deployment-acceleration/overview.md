---
title: CAF:デプロイ高速化の規範の概要
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ガバナンスに関連するデプロイ高速化の説明です。
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 8a9cd359f631708d07ab601c4488b969dc8294a0
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246593"
---
# <a name="caf-deployment-acceleration-discipline-overview"></a>CAF:デプロイ高速化の規範の概要

デプロイ高速化とは、[CAF ガバナンス モデル](../overview.md)の [5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 この規範は、資産の構成またはデプロイを管理するためのポリシーを確立する方法に焦点を当てています。 クラウド ガバナンスの 5 つの規範のうち、デプロイ高速化には、デプロイ、構成のアラインメント、およびスクリプトの再利用が含まれます。 これは手動のアクティビティによる場合と、完全に自動化された DevOps アクティビティによる場合があります。 どちらの場合も、ポリシーはほとんど同じままです。 この規範が成熟するにつれて、クラウド ガバナンス チームは、再利用可能な資産の適用によってデプロイを加速し、クラウド導入に対する障壁を取り除くことで、DevOps およびデプロイ戦略におけるパートナーとしての役割を果たすことができます。

この記事では、クラウド ソリューションの計画、構築、導入、運用の各フェーズで企業が経験するデプロイ高速化プロセスについて概説します。 1 つのドキュメントですべてのビジネスのすべての要件を説明することはできません。 そのため、この記事の各セクションでは、推奨される最小および潜在的なアクティビティについて概説します。 これらのアクティビティの目標は、お客様が[ポリシーの MVP](../policy-compliance/overview.md#minimum-viable-product-mvp-for-policy) を構築する一方、[増分型のポリシー](../policy-compliance/overview.md#incremental-policy-growth)進化のフレームワークを確立できるように支援することです。 クラウド ガバナンス チームは、デプロイ高速化の地位を向上させるために、これらのアクティビティに投資する金額を決定する必要があります。

> [!NOTE]
> デプロイ高速化の規範は、クラウドベースのリソースを効率的にデプロイおよび構成しているお客様の組織の既存 IT チーム、プロセス、および手順にとって代わるものではありません。 この規範は、クラウド上のご利用のリソースを管理する責任がある IT スタッフが、ビジネス上の潜在的なリスクを特定し、リスクを軽減するときの指針となることを主な目的としています。 ガバナンス ポリシーおよびプロセスの策定時には、行っている計画と確認のプロセスに、関係する IT チームを関与させるようにしてください。

このガイドの主な対象読者は、組織のクラウド アーキテクトおよびクラウド ガバナンス チームのその他のメンバーです。 ただし、この規範を使用した決定、ポリシー、プロセスには、ビジネス チームと IT チームの該当メンバー (特にクラウド ベースのワークロードのデプロイおよび構成を担当するリーダー) が関与して、議論する必要があります。

## <a name="policy-statements"></a>ポリシー ステートメント

実践的なポリシー ステートメントと、その結果のアーキテクチャの要件は、デプロイ高速化の規範の基盤となります。 ポリシー ステートメントのサンプルを参照するには、[デプロイ高速化のポリシー ステートメント](./policy-statements.md)に関する記事をご覧ください。 これらのサンプルは、お客様の組織のガバナンス ポリシーの開始点として使用できます。

> [!CAUTION]
> このサンプル ポリシーは、よくあるお客様の体験をもとにしています。 特定のクラウド ガバナンスの要件にこれらのポリシーが合うように調整するには、次の手順を実行し、お客様独自のビジネス ニーズに合うポリシー ステートメントを作成してください。

## <a name="developing-deployment-acceleration-governance-policy-statements"></a>デプロイ高速化のガバナンス ポリシー ステートメントの開発

次の 6 つの手順を使用すると、クラウド環境内のリソースのデプロイおよび構成を管理するガバナンス ポリシーを定義できます。

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsE">
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
                        <h3>デプロイ高速化のテンプレート</h3>
                        <p class="x-hidden-focus">デプロイ高速化の規範を文書化するためのテンプレートをダウンロードします。</p>
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
                        <p class="x-hidden-focus">一般的にデプロイ高速化の規範に関係する動機およびリスクを理解できます。</p>
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
                        <p class="x-hidden-focus">デプロイ高速化の規範に投資する最適なタイミングであるかどうかを知るインジケーターです。</p>
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
                        <p class="x-hidden-focus">デプロイ高速化の規範でのポリシー準拠を確保するための推奨の手順です。</p>
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
                        <p class="x-hidden-focus">デプロイ高速化の規範をサポートするために実装できる Azure サービスです。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>次の手順

特定の環境で[ビジネス上のリスク](./business-risks.md)を評価する方法の概要です。

> [!div class="nextstepaction"]
> [ビジネス上のリスクについての理解](./business-risks.md)

<!-- markdownlint-enable MD033 -->