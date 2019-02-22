---
title: CAF:リソースの整合性の規範の概要
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: リソースの整合性の規範の概要
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ce29efab1502d59513528ab4b640173346aee516
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901604"
---
# <a name="caf-resource-consistency-discipline-overview"></a>CAF:リソースの整合性の規範の概要

リソースの整合性とは、[CAF ガバナンス モデル](../overview.md)の [5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 これは、環境、アプリケーションまたはワークロードの運用管理に関係するポリシーを規定する方法に関する規範です。 IT 業務チームは、多くの場合、アプリケーション、ワークロード、および資産のパフォーマンスの監視を行います。 また、スケーリングの要求、パフォーマンスのサービス レベル アグリーメント (SLA) 違反の修復、および自動修復によるパフォーマンス SLA 違反の事前回避の対応作業も通常行っています。 クラウド ガバナンスの 5 つの規範のうちのリソースの整合性とは、IT 業務部が検出でき、復旧のソリューションに含まれ、繰り返し可能な操作手順に組み込むことができる方法で、リソースが常に構成されていることを保証する規範です。

> [!NOTE]
> リソースの整合性のガバナンスは、クラウドベースのリソースを効率的に管理しているお客様の組織の既存 IT チーム、プロセス、および手順にとって代わるものではありません。 この規範は、クラウド上のご利用のリソースを管理する責任がある IT スタッフが、ビジネス上の潜在的なリスクを特定し、リスクを緩和するときの指針となることを主な目的としています。 ガバナンス ポリシーおよびプロセスの開発時には、行っている計画と確認のプロセスに必ず関係する IT チームを含めるようにしてください。

CAF のこのセクションでは、お客様のクラウド ガバナンス戦略の一環として、リソースの整合性の規範を開発する方法を説明します。 このガイドの主な対象読者は、お客様の組織のクラウド アーキテクトおよびお客様のクラウド ガバナンス チームのその他のメンバーです。 ただし、この規範を使用した決定、ポリシーおよびプロセスには、お客様の組織のリソースの整合性ソリューションを導入および管理する責任のある IT チームの該当メンバーが関与して、議論する必要があります。

お客様の社内にリソースの整合性の専門家がいない場合、この規範の一環として外部コンサルタントを雇用することを検討してください。 また、ご利用のクラウドベースの資産を最善の方法で整理、トラッキングおよび最適化する相談のために、[Microsoft コンサルティング サービス](https://www.microsoft.com/enterprise/services)、[Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) クラウド導入サービス、または外部のその他のクラウド導入のエキスパートを雇用することを検討してください。

## <a name="policy-statements"></a>ポリシー ステートメント

実践的なポリシー ステートメントと、その結果のアーキテクチャの要件は、リソースの整合性の規範の基盤となります。 ポリシー ステートメントのサンプルを参照するには、[リソースの整合性ポリシー ステートメント](./policy-statements.md)に関する記事を参照してください。 これらのサンプルは、お客様の組織のガバナンス ポリシーの第一歩となります。

> [!CAUTION]
> このサンプル ポリシーは、よくあるお客様の体験をもとにしています。 特定のクラウド ガバナンスの要件にこれらのポリシーが合うように調整するには、次の手順を実行し、お客様独自のビジネス ニーズに合うポリシー ステートメントを作成してください。

## <a name="developing-resource-consistency-governance-policy-statements"></a>リソースの整合性のガバナンス ポリシー ステートメントの開発

次の 6 つの手順は、リソースの整合性のガバナンスを開発する場合の、例と可能性のあるオプションです。 各手順は、お客様のクラウド ガバナンス チームと関連するビジネス、およびお客様の組織全体の IT チームが、リソースの整合性のリスクを緩和するのに必要なポリシーやプロセスを規定する話し合いのための第一歩として使用します。

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
                        <h3>リソースの整合性テンプレート</h3>
                        <p class="x-hidden-focus">リソースの整合性の規範を文書化するテンプレートをダウンロードできます。</p>
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
                        <p class="x-hidden-focus">リソースの整合性の規範に通常関係する、動機およびリスクを理解できます。</p>
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
                        <p class="x-hidden-focus">リソースの整合性の規範に投資する最適なタイミングであるかどうかを知るインジケーターです。</p>
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
                        <p class="x-hidden-focus">リソースの整合性の規範がポリシーに準拠するようにするための推奨の手順です。</p>
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
                        <p class="x-hidden-focus">リソースの整合性の規範をサポートするために実装できる Azure サービスです。</p>
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
