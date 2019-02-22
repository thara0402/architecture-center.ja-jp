---
title: 'CAF: CISO の準備状況'
description: CISO がクラウドに向けた準備ができる方法
author: BrianBlanchard
ms.date: 10/03/2018
ms.openlocfilehash: cedb86488304e2fc84897e1da373730768adce66
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902169"
---
# <a name="ciso-cloud-readiness-guide"></a>CISO のクラウド準備状況に関するガイド

Azure クラウド導入フレームワーク (CAF) などの Microsoftによるガイダンスは、このドキュメントのサポート対象である何千社もの企業に特有のセキュリティ制約を、特定したり左右したりするように位置付けられていません。 クラウドに移行する場合、最高情報セキュリティ責任者や最高情報セキュリティ オフィス (CISO) の役割は、クラウド テクノロジに取って代わられるものではありません。 まったく逆に、CISO と CISO オフィスは、よりしっかりと根付いた、統合された存在となります。 このガイドは、読者が CISO のプロセスに精通しており、クラウド変革を実現するためにそれらのプロセスを最新化しようとしていることを前提としています。

クラウドの導入により、従来の IT 環境ではあまり検討されなかったサービスが実現します。 セルフサービスのデプロイや自動化されたデプロイは、一般に、従来は運用環境のデプロイには結び付けられていなかったアプリケーション開発チームやその他の IT チームによって実行されます。 組織によっては、事業の構成単位が同様に、セルフ サービスの機能を持つことになります。 これは、オンプレミス環境では必要でなかった、新しいセキュリティ要件が生じるきっかけとなる可能性があります。 セキュリティの一元化はより困難であり、セキュリティは多くの場合、事業部門と IT 部門の間で責任を共有することになります。 この記事は、そうしたアプローチに向けた CISO の準備と、増分ガバナンスへの取り組みの助けになります。

## <a name="how-can-the-ciso-prepare-for-the-cloud"></a>CISO がクラウドに向けた準備ができる方法

ほとんどのポリシーと同様に、組織内のセキュリティとガバナンスのポリシーは自然に拡大する傾向があります。 セキュリティ インシデントの発生時には、ユーザーへの通知を行い、繰り返しが起きる可能性を減らすためのポリシーがまとめられます。 自然ではあっても、このアプローチでは、ポリシーの肥大化と技術的な依存関係が生じます。 クラウド変革の体験では、ポリシーを最新化してリセットする独自の機会を作ります。 どの変革体験を準備しているときでも、CISO は[ポリシー レビュー](./what-is-a-cloud-policy-review.md)で主要な利害関係者としての役割を果たすことで、即時の測定可能な価値を作り出すことができます。

そのようなレビューにおける CISO の役割は、既存のポリシーやコンプライアンスの制約と、クラウド プロバイダーの改善されたセキュリティ体制の間で、あぶなげないバランスを生み出すことです。 この進行状況はさまざまな形式で測定される場合があり、多くの場合、クラウド プロバイダーに安全に移管できるセキュリティ ポリシーの数で測定されます。

**セキュリティ リスクの移転**: サービスは、サービスとしてのインフラストラクチャ (IaaS) ホスティング モデルに移行されるため、ハードウェアのプロビジョニングに関して企業が引き受ける直接的なリスクは減少します。 リスクは解消されたのではなく、クラウド ベンダーに移転されました。 ハードウェアのプロビジョニングに対するクラウド ベンダーのアプローチによって、セキュリティで保護された繰り返し可能なプロセスと同じレベルのリスク軽減が提供されるのであれば、ハードウェア プロビジョニングの実行リスクは企業ポリシーから取り除かれます。 これで、それらのプロセスを検証するための新しいポリシーで置き換えることもできますが、実行のリスクが再割り当てされて、全体のセキュリティ リスクは減少しています。

ソリューションがさらに "スタックを上に" 移動して、サービス としてのプラットフォーム (PaaS) やサービスとしてのソフトウェア (SaaS) モデルを組み込と、より多くのリスクの軽減、移転、置き換えが可能になります。 リスクが安全にクラウド プロバイダーに移されると、セキュリティ ポリシーやその他のコンプライアンス ポリシーを実行、監視、および適用するコストも安全に削減できます。

**成長の思考様式**: 変更がおそろしいのはビジネス面だけでなく、技術の実装者たちにとっても同様な場合があります。 CISO が組織における成長の思考様式を先導している場合、そうした自然なおそれは、安全性とポリシー コンプライアンスへの関心の高まりで置き換えられることがわかっています。 成長の思考様式で、変革体験の 1 つである[ポリシー レビュー](./what-is-a-cloud-policy-review.md)や単純な実装レビューに取り掛かると、チームはすばやく動くことができますが、適正で管理しやすいリスク プロファイルを犠牲にすることはありません。

## <a name="resources-for-the-chief-information-security-officer"></a>最高情報セキュリティ責任者のためのリソース

成長の思考様式で[ポリシー レビュー](./what-is-a-cloud-policy-review.md)に取り掛かるためには、クラウドに関する知識が必須です。 以下のリソースは、CISO が Microsoft の Azure プラットフォームのセキュリティ体制をより深く理解するのに役立ちます。

セキュリティ プラットフォームのリソース: 

* [セキュリティ開発サイクル、内部監査](https://www.microsoft.com/sdl/)
* [必須のセキュリティ トレーニング、バックグラウンド チェック](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx)
* [侵入テスト、不正侵入検出、DDoS、監査、ログ記録](https://www.microsoft.com/trustcenter/Security/AuditingAndLogging)
* [最新のデータ センター](https://www.microsoft.com/cloud-platform/global-datacenters)、物理的なセキュリティ、[セキュリティで保護されたネットワーク](/azure/security/security-network-overview)
* [クラウドにおける Microsoft Azure のセキュリティ対応に関する資料 (PDF)](http://aka.ms/SecurityResponsePaper)

プライバシーと管理: 

* [データの常時管理](https://www.microsoft.com/trustcenter/Privacy/You-own-your-data)
* [データ保管場所の管理](https://www.microsoft.com/trustcenter/Privacy/Where-your-data-is-located)
* [条件に応じたアクセス権の付与](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)
* [法執行機関への対応](https://www.microsoft.com/trustcenter/Privacy/Responding-to-govt-agency-requests-for-customer-data)
* [厳格なプライバシー基準](https://www.microsoft.com/TrustCenter/Privacy/We-set-and-adhere-to-stringent-standards)

コンプライアンス: 

* [トラスト センター](https://www.microsoft.com/trustcenter/default.aspx)
* [共通管理ハブ](https://www.microsoft.com/trustcenter/Common-Controls-Hub)
* [クラウド サービス向けデリジェンス チェックリスト](https://www.microsoft.com/trustcenter/Compliance/Due-Diligence-Checklist)
* [サービス、場所、および業界ごとのコンプライアンス](https://www.microsoft.com/trustcenter/Compliance/default.aspx)

透明性: 

* [Microsoft が Azure サービスで顧客データの安全性を確保する方法](https://www.microsoft.com/trustcenter/Transparency/default.aspx)
* [Microsoft が Azure サービスでデータの保管場所を管理する方法](http://azuredatacentermap.azurewebsites.net/)
* [データにアクセスできるユーザーとその条件](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)
* [Microsoft が Azure サービスで顧客データの安全性を確保する方法](https://www.microsoft.com/trustcenter/Transparency/default.aspx)
* [Azure サービスの証明書確認、Transparency Hub](https://www.microsoft.com/trustcenter/Compliance/default.aspx)

## <a name="next-steps"></a>次の手順

どのガバナンス戦略においても、行動を起こす最初の手順は[ポリシー レビュー](./what-is-a-cloud-policy-review.md)です。 [ポリシーとコンプライアンス](./overview.md)に関するページは、ポリシー レビュー時に役立つガイドになる可能性があります。

> [!div class="nextstepaction"]
> [ポリシー レビューを準備する](./what-is-a-cloud-policy-review.md)