---
title: Azure でのマイクロサービスの構築
description: Azure でのマイクロサービス アーキテクチャの設計、構築、および操作
ms.date: 03/07/2019
layout: LandingPage
ms.topic: landing-page
---

# <a name="building-microservices-on-azure"></a>Azure でのマイクロサービスの構築

<!-- markdownlint-disable MD033 -->

<img src="../_images/microservices.svg" style="float:left; margin-top:8px; margin-right:8px; max-width: 80px; max-height: 80px;"/>

マイクロサービスは、回復性があり、単独でのデプロイが可能で、迅速に展開できる非常にスケーラブルなアプリケーションを構築するための一般的なアーキテクチャ スタイルです。 しかし、マイクロサービス アーキテクチャが成功するには、アプリケーションを設計および構築するのためのさまざまなアプローチが必要です。

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./introduction.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>マイクロサービスとは</h3>
                        <p>マイクロサービスと他のアーキテクチャの違いと、それらを使用すべき場面</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../guide/architecture-styles/microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>マイクロサービス アーキテクチャ スタイル</h3>
                        <p>マイクロサービス アーキテクチャ スタイルの概要</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="examples-of-microservices-architectures"></a>マイクロサービス アーキテクチャの例

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/infrastructure/service-fabric-microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Service Fabric を使用したモノリシック アプリケーションの分解</h3>
                        <p>ASP.NET Web サイトをマイクロサービスに分解する反復的なアプローチ。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/data/ecommerce-order-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Azure でのスケーラブルな注文処理</h3>
                        <p>マイクロサービスを介して実装される関数型プログラミング モデルを使用した注文処理。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="build-a-microservices-application"></a>マイクロサービス アプリケーションの構築

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./model/domain-analysis.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>ドメイン分析を使用したマイクロサービスのモデル化</h3>
                        <p>マイクロサービスの設計時によくあるいくつかの問題を回避するために、ドメイン分析を使用してマイクロサービス境界を定義します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Azure Kubernetes Services (AKS) の参照アーキテクチャ</h3>
                        <p>この参照アーキテクチャには、ほとんどのデプロイの出発点にすることができる基本的な AKS 構成が示されています。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Azure Service Fabric の参照アーキテクチャ</h3>
                        <p>この参照アーキテクチャには、ほとんどのデプロイの出発点にすることができる推奨構成が示されています。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>マイクロサービス アーキテクチャの設計</h3>
                        <p>これらの記事では、Azure Kubernetes Service (AKS) が使用されるリファレンス実装に基づいて、マイクロサービス アプリケーションを構築する方法について詳しく説明します。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/patterns.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>設計パターン</h3>
                        <p>マイクロサービスに役立つ一連の設計パターン。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="operate-microservices-in-production"></a>運用環境でのマイクロサービスの操作

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./logging-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>ログ記録と監視</h3>
                        <p>マイクロサービス アーキテクチャの分散特性によって、ログ記録と監視が特に重要になります。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./ci-cd.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>継続的インテグレーションとデプロイ</h3>
                        <p>継続的インテグレーションと継続的デリバリー (CI/CD) は、マイクロサービスで成功を収めるための鍵です。</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>
