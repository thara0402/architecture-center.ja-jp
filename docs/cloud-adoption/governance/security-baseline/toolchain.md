---
title: 'CAF: Azure でのセキュリティ ベースライン ツール'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure で改善されたセキュリティ ベースラインを利用できるツールの説明
author: BrianBlanchard
ms.openlocfilehash: b316626c8ad717514f7f592abefa0f33a92afdca
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901589"
---
# <a name="security-baseline-tools-in-azure"></a>Azure でのセキュリティ ベースライン ツール

[セキュリティ ベースライン](overview.md)は、[5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 この規範では、ネットワーク、資産、そして最も重要なクラウド プロバイダーのソリューションに存在するデータを保護するポリシーを確立する方法に重点を置いています。 クラウド ガバナンスの 5 つの規範のうち、セキュリティ ベースラインにはデジタル資産とデータの分類が含まれます。 また、これには、リスク、ビジネスの許容範囲、およびデータ、資産、ネットワークのセキュリティに関連付けられたリスク軽減戦略のドキュメントも含まれます。 技術的な観点から、これは、[暗号化](../../decision-guides/encryption/overview.md)、[ネットワーク要件](../../decision-guides/software-defined-network/overview.md)、[ハイブリッド ID 戦略](../../decision-guides/identity/overview.md)、および[リソース グループ](../../decision-guides/resource-consistency/overview.md)全体でセキュリティ ポリシーの[適用を自動化](../../decision-guides/policy-enforcement/overview.md)するツールに関する決定にも関わります。

セキュリティ ベースラインをサポートするポリシーとプロセスを成熟させるのに役立つ Azure ツールの一覧を次に示します。

|                                                            | [Azure portal](https://azure.microsoft.com/features/azure-portal/) / [Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Azure Key Vault](/azure/key-vault)  | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) | [Azure Policy](/azure/governance/policy/overview) | [Azure Security Center](/azure/security-center/security-center-intro) | [Azure Monitor](/azure/azure-monitor/overview) |
|------------------------------------------------------------|---------------------------------|-----------------|----------|--------------|-----------------------|---------------|
| アクセスの制御をリソースとリソースの作成に適用する   | はい                             | いいえ               | はい      | いいえ            | いいえ                     | いいえ             |
| 仮想ネットワークをセキュリティで保護する                                    | はい                             | いいえ               | いいえ        | はい          | いいえ                     | いいえ             |
| 仮想ドライブを暗号化する                                     | いいえ                               | はい             | いいえ        | いいえ            | いいえ                     | いいえ             |
| PaaS ストレージとデータベースを暗号化する                         | いいえ                               | はい             | いいえ        | いいえ            | いいえ                     | いいえ             |
| ハイブリッドの ID サービスを管理する                            | いいえ                               | いいえ               | はい      | いいえ            | いいえ                     | いいえ             |
| リソースの許可される型を制限する                         | いいえ                               | いいえ               | いいえ        | はい          | いいえ                     | いいえ             |
| geo リージョンの制限を適用する                          | いいえ                               | いいえ               | いいえ        | はい          | いいえ                     | いいえ             |
| ネットワークとリソースのセキュリティ正常性を監視する          | いいえ                               | いいえ               | いいえ        | いいえ            | 可能                    | はい           |
| 悪意のあるアクティビティを検出する                                  | いいえ                               | いいえ               | いいえ        | いいえ            | 可能                    | はい           |
| 事前に脆弱性を検出する                        | いいえ                               | いいえ               | いいえ        | いいえ            | はい                   | いいえ             |
| バックアップとディザスター リカバリーを構成する                     | はい                             | いいえ               | いいえ        | いいえ            | いいえ                     | いいえ             |

セキュリティ ツールとサービスの完全なリストについては、「[Azure で利用できるセキュリティ サービスとテクノロジ](/azure/security/azure-security-services-technologies)」を参照してください。

また、セキュリティ ベースラインのアクティビティを促進するために、顧客がサードパーティ製のツールを使用することもきわめて一般的です。 詳細については、「[Azure Security Center でのセキュリティ ソリューションの統合](/azure/security-center/security-center-partner-integration)」の記事を参照してください。

セキュリティ ツールに加えて、[Microsoft Trust Center](https://www.microsoft.com/trustcenter/guidance/risk-assessment) には、広範なガイダンス、レポート、および移行計画のプロセスの一環として、リスク評価の実行に役立つ関連するドキュメントが含まれます。
