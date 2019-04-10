---
title: CAF:セキュリティ ベースライン規範の概要
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: セキュリティ ベースライン規範の概要
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 189fbd93371055f6aec63323988138f5bf90fd30
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247283"
---
# <a name="caf-security-baseline-discipline-overview"></a>CAF:セキュリティ ベースライン規範の概要

セキュリティ ベースラインとは、[CAF ガバナンス モデル](../overview.md)の [5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 セキュリティはあらゆる IT のデプロイに含まれる構成要素であり、クラウドによって固有のセキュリティの懸案事項が発生します。 多くの企業は、クラウドの変革を検討する際に、機密データの保護を組織の最優先事項にするという規制の要件に従います。 クラウド環境に対する潜在的なセキュリティの脅威を特定し、このような脅威に対処するためのプロセスと手順を確立することは、あらゆる IT セキュリティ チームやサイバー セキュリティ チームにとって優先事項です。 こうした要件の成熟につれて、技術的な要件とセキュリティ上の制約がクラウド環境に一貫して適用されるようにセキュリティ ベースライン規範で確保します。

> [!NOTE]
> セキュリティ ベースラインのガバナンスは、クラウドにデプロイされているリソースをセキュリティで保護するためにお客様の組織が使用している既存の IT チーム、プロセス、および手順にとって代わるものではありません。 この規範は、セキュリティ インフラストラクチャの責任を持つ IT スタッフが、セキュリティに関連するビジネス上のリスクを特定し、リスクを緩和するときの指針となることを主な目的としています。 ガバナンス ポリシーおよびプロセスの開発時には、行っている計画と確認のプロセスに必ず関係する IT チームを含めるようにしてください。

この記事では、クラウド ガバナンス戦略の一環としてセキュリティ ベースライン規範を開発する方法について概要を説明します。 このガイドの主な対象読者は、お客様の組織のクラウド アーキテクトおよびお客様のクラウド ガバナンス チームのその他のメンバーです。 ただし、この規範から出てくる決定、ポリシー、プロセスには、IT およびセキュリティ チームの該当メンバー (特にネットワーク、暗号化、および ID サービスの実装を担当する技術リーダー) が関与して、議論する必要があります。

適切なセキュリティの決定を下すことは、クラウド デプロイの成功と幅広いビジネスの成功にとって重要です。 お客様の組織内にサイバーセキュリティのエキスパートがいない場合は、この規範の一環として外部のセキュリティ コンサルタントを雇用することを検討してください。 また、この規範に関連する懸案事項を相談するために、[Microsoft コンサルティング サービス](https://www.microsoft.com/enterprise/services)、[Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack/) クラウド導入サービス、または外部のその他のクラウド導入のエキスパートを雇用することを検討してください。

## <a name="policy-statements"></a>ポリシー ステートメント

実践的なポリシー ステートメントと、その結果のアーキテクチャの要件は、セキュリティ ベースラインの規範の基盤となります。 ポリシー ステートメントのサンプルを参照するには、[セキュリティ ベースライン ポリシー ステートメント](./policy-statements.md)に関する記事を参照してください。 これらのサンプルは、お客様の組織のガバナンス ポリシーの第一歩となります。

> [!CAUTION]
> このサンプル ポリシーは、よくあるお客様の体験をもとにしています。 特定のクラウド ガバナンスの要件にこれらのポリシーが合うように調整するには、次の手順を行い、お客様独自のビジネス ニーズに合うポリシー ステートメントを作成してください。

## <a name="developing-security-baseline-governance-policy-statements"></a>セキュリティ ベースラインのガバナンス ポリシー ステートメントの開発

次の 6 つの手順は、セキュリティ ベースラインのガバナンスを開発する場合の、例と可能性のあるオプションです。 各手順は、お客様のクラウド ガバナンス チーム内と、お客様の組織全体の影響を受ける業務、IT、およびセキュリティ チームとで、セキュリティに関連するリスクを緩和するために必要なポリシーやプロセスを規定する話し合いの第一歩として使用します。

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
                        <h3>セキュリティ ベースライン テンプレート</h3>
                        <p class="x-hidden-focus">セキュリティ ベースラインの規範を文書化するテンプレートをダウンロードできます。</p>
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
                        <p class="x-hidden-focus">一般的にセキュリティ ベースラインの規範に関係する動機およびリスクを理解できます。</p>
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
                        <p class="x-hidden-focus">セキュリティ ベースラインの規範に投資する最適なタイミングであるかどうかを知るインジケーターです。</p>
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
                        <p class="x-hidden-focus">セキュリティ ベースラインの規範によりポリシーのコンプライアンスがサポートされるようにするための推奨の手順です。</p>
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
                        <p class="x-hidden-focus">セキュリティ ベースラインの規範をサポートするために実装できる Azure サービスです。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>次の手順

特定の環境でビジネス上のリスクを評価する方法の概要を学びます。

> [!div class="nextstepaction"]
> [ビジネス上のリスクについての理解](./business-risks.md)
