---
title: Azure 仮想データセンター
description: Microsoft Azure 仮想データセンターのリソース
keywords: Azure
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
---

# <a name="azure-virtual-datacenter-and-the-enterprise-control-plane"></a>Azure 仮想データセンターとエンタープライズ コントロール プレーン

Azure 仮想データセンターは、既存のセキュリティやネットワーク ポリシーを順守しながら、Azure のクラウド プラットフォーム機能を最大限に活用できる手法です。 IT 企業や IT 事業部門がエンタープライズ ワークロードをクラウドにデプロイする際は、ガバナンスと開発者の俊敏性のバランスを取る必要があります。 Azure 仮想データセンターでは、ガバナンスに重点を置きつつ、このバランスを実現するモデルを提供します。
 
## <a name="resources"></a>リソース
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="https://aka.ms/VDC/Concepts"><img src="../_images/virtual-datacenter.svg" alt="Virtual Datacenter eBook" /></a></td>
    <td>
        <h3><a href="https://aka.ms/VDC/Concepts">Azure 仮想データセンター:概念</a></h3>
        <p>この電子書籍では、既存のセキュリティとネットワーク ポリシーを順守しながら、エンタープライズ ワークロードを Azure クラウド プラットフォームにデプロイする方法について説明します。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="/azure/networking/networking-virtual-datacenter"><img src="./images/vdc-network.png" alt="Network Perspective" /></a></td>
    <td>
        <h3><a href="networking-virtual-datacenter.md">Azure 仮想データセンター:ネットワーク パースペクティブ</a></h3>
        <p>このオンライン記事では、多くのお客様がクラウドへの一括移行を検討するときに直面するアーキテクチャのスケーリング、パフォーマンス、セキュリティに関する問題の解決に役立てることができるネットワーク パターンと設計の概要について説明します。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="https://aka.ms/VDC/Lift"><img src="./images/vdc-lift-and-shift.png" alt="Lift and Shift Guide" /></a></td>
    <td>
        <h3><a href="https://aka.ms/VDC/Lift">Azure 仮想データセンター:リフト アンド シフト ガイド</a></h3>
        <p>このホワイトペーパーでは、大企業の IT スタッフや意思決定者がリフト アンド シフト メソッドを使用して、Azure へのアプリケーションとサーバーの移行方法を特定および計画するために使用できるプロセスについて説明します。これにより、追加の開発コストを最小限に抑えながらクラウドのホスティング オプションを最適化できます。</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="https://aka.ms/VDC/Deck"><img src="./images/vdc-deck.png" alt="Presentation Deck" /></a></td>
    <td>
        <h3><a href="https://aka.ms/VDC/Deck">Azure 仮想データセンター:プレゼンテーション デッキ</a></h3>
        <p>このプレゼンテーション デッキでは、Azure 仮想データセンターのガイダンスとツールについて説明します。 ここでは、VDC の目標、顧客要因、Azure リージョン、VDC の自動化要素、先進的で信頼される Azure VDC について触れ、最後に CIO へのガイダンスを中心としたアクション プランを示します。 また、サポート情報とトレーニング情報も提供します。</p>
    </td>
</tr>
</table>

## <a name="what-is-the-azure-virtual-datacenter"></a>Azure 仮想データセンターとは

ワークロードをクラウドにデプロイするには、既存のデータセンターに対する信頼と同程度に、クラウドへの信頼を高めて維持する必要があります。 Azure 仮想データセンターのガイダンスの最初のモデルは、仮想インフラストラクチャに対するロックダウン方式を通じてこのニーズを満たすように設計されています。 この方法ですべてのユーザーに対応できるわけではありません。 これは、大企業の IT 部門がオンプレミスのインフラストラクチャを Azure のパブリック クラウドに拡張する際に指針となるよう特別に考えられたものです。 この方法は、信頼されたデータセンター拡張モデルと呼ばれます。 時間の経過と共に、仮想データセンターからの、セキュリティで保護された直接のインターネット アクセスが可能なものも含め、その他いくつかのモデルが提供される予定です。

<img src="./images/vdc-components.svg" alt="Virtual Datacenter components" style="max-width:700px;"/>

Azure 仮想データセンターは、ID、暗号化、ソフトウェアによるネットワーク制御、コンプライアンス (ログおよびレポートを含む) という 4 つのコンポーネントによって可能になります。

Azure 仮想データセンターのこのモデルでは、分離ポリシーの適用により、クラウドを既知の物理データセンターのようにすることができるため、必要なレベルのセキュリティと信頼を達成できます。 どの大企業の IT チームもその重要性を認める 4 つのコンポーネント (ソフトウェアによるネットワーク制御、暗号化、ID 管理、および Azure プラットフォームの基になるコンプライアンス標準と認定資格) によって、これが可能になります。 この 4 つは、仮想データセンターが、既存のインフラストラクチャへの投資に対する確かな拡張となるための鍵です。


引き続き、電子書籍「<a href="https://aka.ms/VDC/eBook">Azure 仮想データセンター: 概念</a>」をお読みください。
