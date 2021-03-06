---
title: CAF:クラウド ポリシーの確認とは
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ポリシーの確認とは
author: BrianBlanchard
ms.openlocfilehash: 2e50c6bac0118db1b6a223244cf8efc43a4ab438
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242083"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-a-cloud-policy-review"></a>クラウド ポリシーの確認とは

**クラウド ポリシーの確認**は、クラウドにおける[ガバナンスの成熟](../overview.md)に向けた最初の手順です。 このプロセスの目的は、既存の社内 IT ポリシーを最新化することです。 完了すると、更新されたポリシーによって、クラウド ベースのリソースに対して同等レベルのリスク軽減の効果が得られます。 この記事では、クラウド ポリシーの確認プロセスとその重要性について説明します。

## <a name="why-perform-a-cloud-policy-review"></a>クラウド ポリシーの確認を行う理由

ほとんどの企業は、管理ポリシーと整合するプロセスの実行を通じて IT を管理しています。 小規模な企業では、それらのポリシーは根拠があいまいで、プロセスが大まかに定義されている場合があります。 ビジネスが大企業へと成長するにつれ、ポリシーとプロセスはより明確に文書化され、一貫して実行される傾向があります。

企業において企業 IT ポリシーが成熟するにつれ、過去の技術的な決定への依存が管理ポリシーに組み込まれていく傾向があります。 たとえば、ディザスター リカバリーのプロセスにオフサイトのテープ バックアップを義務付けるポリシーが含まれていることがよくあります。 これを含めることは、1 つのタイプのテクノロジ (テープ バックアップ) への依存が前提となりますが、そのタイプは最適なソリューションではなくなっている場合があります。

クラウドの変換により、過去のレガシ ポリシーの決定を再検討するための自然な変曲点がもたらされます。 技術的な機能と既定のプロセスは、継承リスクと同様にクラウドで大きく変化します。 前の例を使用すると、テープ バックアップ ポリシーは 1 つの場所にデータを保持することによる単一障害点のリスクに端を発していたため、ビジネスはこのリスクを軽減して、リスク プロファイルを最小限に抑える必要があります。 クラウドのデプロイでは、はるかに低い目標復旧時間 (RTO) で同じリスク軽減を実現するいくつかの選択肢があります。 次に例を示します。

- クラウド ネイティブのソリューションでは、SQL Azure データベースの geo レプリケーションを有効にできる場合があります。
- ハイブリッド ソリューションでは、Azure Site Recovery を使用して、IaaS ワークロードを複数のデータセンターにレプリケートできる場合があります。

クラウドの変換を実行する際には、クラウドの導入チームが利用できるツール、サービス、およびプロセスの多くをポリシーで管理することがよくあります。 それらのポリシーがレガシ テクノロジに基づいている場合、変更を推進するためのチームの取り組みを妨げる可能性があります。 最悪の場合、回避策を有効にするために、重要なポリシーが移行チームによって完全に無視されます。 どちらも許容できる結果ではありません。

## <a name="the-cloud-policy-review-process"></a>クラウド ポリシーの確認のプロセス

クラウド ポリシーの確認では、既存の IT ガバナンスと IT セキュリティ ポリシーを次の[クラウド ガバナンスの 5 つの規範](../overview.md)と整合させます。[Cost Management](../cost-management/overview.md)、[セキュリティ ベースライン](../security-baseline/overview.md)、 [ID ベースライン](../identity-baseline/overview.md)、[リソースの整合性](../resource-consistency/overview.md)、[デプロイ高速化](../deployment-acceleration/overview.md)。

確認プロセスは、これらの規範ごとに次の手順に従います。

1. 特定の規範に関連する既存のオンプレミス ポリシーを確認し、2 つの主要なデータ ポイント (レガシの依存、識別されたビジネス上のリスク) を探します。
2. 次の簡単な質問を行ってビジネス上の各リスクを評価します。「クラウド モデルにビジネス上のリスクがまだ存在しますか?」
3. リスクが依然として存在する場合、技術的なソリューションではなく、必要な軽減策を文書化して、ポリシーを書き換えます。
4. 更新されたポリシーをクラウド導入チームと共に確認して、必要な軽減策に対する可能なソリューションを理解します。

## <a name="example-of-a-policy-review-for-a-legacy-policy"></a>レガシ ポリシーにおけるポリシーの確認の例

プロセスの例を示すために、前のセクションのテープ バックアップ ポリシーをもう一度使用します。

- 企業ポリシーでは、すべての実稼働システムに対してオフサイト テープ バックアップが義務付けられています。 このポリシーでは、対象となる次の 2 つのデータ ポイントを確認できます。
  - テープ バックアップ ソリューションへのレガシの依存
  - 実稼働機器と同じ物理的な場所にバックアップを保存することに関連して想定されるビジネス上のリスク。
- リスクはまだ存在しますか? はい。 クラウドでも、1 つの施設に依存することでリスクが発生します。 オンプレミスのソリューションに存在していたリスクよりも、このリスクがビジネスに影響を与える可能性は低いですが、リスクは依然として存在します。
- ポリシーを書き換えます。 データ センター全体の障害の場合は、障害から 24 時間以内に別のデータセンターと別の地理的な場所に実稼働システムを復元する手段が必要です。
- クラウド導入のチームと確認します。 実装されるソリューションに応じて、このリソース整合性ポリシーに準拠する複数の手段があります。

## <a name="tools-to-help-create-modern-policies"></a>最新のポリシーを作成するためのツール

最新のポリシーの作成時間を短縮するために、クラウド ガバナンスの 5 つの規範それぞれに一連のサンプル ポリシーが用意されています。 これらのサンプル ポリシーはそれぞれ、次の 3 つの設計上の前提のうちの 1 つを開始点とします。

- **クラウド ネイティブ**:デプロイされるソリューションはクラウド ネイティブであり、最小限の構成で、Azure にある既定のソリューションを活用できます。
- **エンタープライズ**:デプロイされるソリューションは複雑であり、ハイブリッド クラウドのデプロイ モデルが必要です。 これには、特定のガバナンス規範のより複雑な実装が必要です。
- **クラウド設計原則 (CDP) の準拠**:デプロイされるソリューションは、CDP で定義されているアーキテクチャの軸に準拠している必要があり、さらに高度なガバナンスが必要です。  

各規範について、サンプル ポリシーをこれらの各レベルで作成する必要があります。 各サンプルは、企業環境内で考察や対話をもたらすように意図されてします。 これらのサンプルは、適切に構成された企業 IT ポリシーに代わるものとして使用することを目的としたものではないことに注意してください。
