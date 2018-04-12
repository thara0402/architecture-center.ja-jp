---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 530844a0d3b1256cec807e7bad509a40dca304f6
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="azure-application-architecture-guide"></a>Azure アプリケーション アーキテクチャ ガイド

このガイドは、Azure においてスケーラブルで回復力がある高可用性のアプリケーションを設計するための体系化された方法を示します。 顧客エンゲージメントから学んだ実証済みのプラクティスに基づいています。

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>はじめに

クラウドによって、アプリケーションを設計する方法が変わりつつあります。 アプリケーションは、モノリスではなく、小さな分散サービスに分解されます。 これらのサービスは、API を介して、あるいは非同期メッセージングまたはイベントを使用して通信します。 アプリケーションは、需要に応じて新しいインスタンスを追加して水平方向に拡張します。 

このような傾向により新たな課題が生まれました。 アプリケーションの状態が分散しています。 操作は並列かつ非同期的に実行されます。 障害が発生しても、システム全体としては回復力があることが必要です。 デプロイは自動的に行い予測可能であることが必要です。 監視とテレメトリは、システムの状態を把握するために重要です。 Azure アプリケーション アーキテクチャ ガイドは、このような変化に対応できるように支援します。 

<table>
<thead>
    <tr><th>従来のオンプレミス</th><th>先進的なクラウド</th></tr>
</thead>
<tbody>
<tr><td>モノリシック、集中管理<br/>
予測可能なスケーラビリティの設計<br/>
リレーショナル データベース<br/>
強力な一貫性<br/>
直列および同期の処理<br/>
障害 (MTBF) を回避する設計<br/>
不定期の大規模な更新<br/>
手動管理<br/>
スノーフレーク サーバー</td>
<td>
分解、分散化<br/>
柔軟なスケーラビリティの設計<br/>
多機種の持続性 (記憶域テクノロジの混在)<br/>
最終的な一貫性<br/>
並列および非同期の処理<br/>
障害 (MTTR) のための設計<br/>
頻繁な小規模の更新<br/>
自動自己管理<br/>
イミュータブル インフラストラクチャ<br/>
</td>
</tbody>
</table>

このガイドは、アプリケーションの設計者、開発者、および運用チームを対象としています。 個々 の Azure サービスを使用するためのハウツー ガイドではありません。 このガイドを読むことで、Azure クラウド プラットフォームでの構築に適用できるアーキテクチャのパターンやベスト プラクティスを理解できます。 また、[このガイドの電子ブック バージョン][ebook]をダウンロードすることもできます。

## <a name="how-this-guide-is-structured"></a>本書の構成

Azure アプリケーション アーキテクチャ ガイドは、アーキテクチャや設計から実装まで、一連のステップで編成されます。 ステップごとに、アプリケーション アーキテクチャの設計に役立つ補助的なガイダンスがあります。

**[アーキテクチャ スタイル][arch-styles]**。 最初の意思決定ポイントがすべての基盤になります。 どのような種類のアーキテクチャを構築しますか。 マイクロサービス アーキテクチャ、従来の n 層アプリケーション、またはビッグ データ ソリューションも考えられます。 アーキテクチャ スタイルを 7 つ特定しました。 それぞれに利点と課題があります。

> &#10148; [Azure 参照アーキテクチャ][ref-archs]では、Azure での推奨デプロイを、スケーラビリティ、可用性、管理性、およびセキュリティに関する考慮事項と共に示します。 ほとんどにはデプロイ可能な Resource Manager テンプレートも含まれています。

**[テクノロジの選択][technology-choices]**。 早い段階で 2 つのテクノロジの選択を決定する必要があります。アーキテクチャ全体に影響が及ぶためです。 これらは、コンピューティングと記憶域のテクノロジの選択です。 "*コンピューティング*" という用語は、アプリケーションが実行するコンピューティング リソースのホスティング モデルを指します。 記憶域にはデータベースの他に、メッセージ キュー、キャッシュ、IoT データ、非構造化ログ データ、およびアプリケーションが記憶域に格納するその他すべての記憶域が含まれます。 

> &#10148; [コンピューティング オプション][compute-options]と[記憶域オプション][storage-options]では、コンピューティングと記憶域のサービスを選択するために各基準が詳しく比較されます。

**[設計原則][design-principles]**。 設計プロセス全体を通じて、ここで示す 10 個の設計原則に留意してください。 

> &#10148; [ベスト プラクティス][best-practices]の記事では、自動スケール、キャッシュ、データのパーティション分割、API 設計などについて具体的なガイダンスが提供されます。   

**[要点][pillars]**。 成功するクラウド アプリケーションは、ソフトウェア品質の 5 つの要点、すなわちスケーラビリティ、可用性、回復力、管理性、およびセキュリティに重点的に取り組みます。 

> &#10148; [設計レビューのチェックリスト][checklists] を使用し、品質に関するこれらの要点に基づいて設計をレビューします。 

**[クラウドの設計パターン][patterns]**。 これらの設計パターンは、信頼性の高い、スケーラブルで安全なアプリケーションを Azure に構築するために役立ちます。 各パターンでは、問題、その問題を解決するパターン、Azure に基づく例を説明します。

> &#10148; クラウド設計パターンの完全なカタログは[こちら](../patterns/index.md)を参照してください。


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

