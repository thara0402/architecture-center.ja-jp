---
title: CAF:Azure のリソース整合性ツール
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure のリソース整合性ツール
author: BrianBlanchard
ms.openlocfilehash: 68503289f60fbb3682264ff39546ca7b7700cef5
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901926"
---
# <a name="resource-consistency-tools-in-azure"></a>Azure のリソース整合性ツール

[リソースの整合性](overview.md)は、[5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 これは、環境、アプリケーションまたはワークロードの運用管理に関係するポリシーを規定する方法に関する規範です。 クラウド ガバナンスの 5 つの規範のうち、"リソースの整合性" にアプリケーション、ワークロード、および資産のパフォーマンスの監視が含まれます。 スケール要求の充足、パフォーマンスの SLA 違反の修復、および自動修復によるパフォーマンス SLA 違反の事前回避を行うために必要なタスクも含まれています。

このガバナンス規範をサポートするポリシーとプロセスを成熟させるのに役立つ Azure ツールの一覧を次に示します。

|    | [Azure Portal](https://azure.microsoft.com/features/azure-portal/)  | [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Azure Blueprint](/azure/governance/blueprints/overview) | [Azure Automation](/azure/automation/automation-intro) | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) |
|---------|---------|---------|---------|---------|---------|
| リソースのデプロイ                             | はい | はい | はい | はい | いいえ   |
| リソースの管理                             | はい | はい | はい | はい | いいえ   |
| テンプレートを使用してリソースをデプロイする             | いいえ   | はい | いいえ   | はい | いいえ   |
| 調整された環境のデプロイ          | いいえ   | いいえ   | はい | いいえ   | いいえ   |
| リソース グループを定義する                       | はい | はい | はい | いいえ   | いいえ   |
| ワークロードとアカウントの所有者を管理する           | はい | はい | はい | いいえ   | いいえ   |
| リソースへの条件付きアクセスを管理する       | はい | はい | はい | いいえ   | いいえ   |
| RBAC ユーザーを構成する                         | はい | いいえ   | いいえ   | いいえ   | はい |
| ロールとアクセス許可をリソースに割り当てる | はい | はい | はい | いいえ   | はい |
| リソース間の依存関係を定義する        | いいえ   | 可能  | はい | いいえ   | いいえ   |
| アクセス制御を適用する                         | はい | はい | はい | いいえ   | はい |
| 可用性とスケーラビリティを評価する          | いいえ   | いいえ   | いいえ   | はい | いいえ   |
| タグをリソースに適用する                      | はい | はい | はい | いいえ   | いいえ   |
| Azure Policy の規則を割り当てる                    | はい | はい | はい | いいえ   | いいえ   |
| ディザスター リカバリーのためにリソースを計画する         | はい | はい | はい | いいえ   | いいえ   |
| 自動修復を適用する                  | いいえ   | いいえ   | いいえ   | はい | いいえ   |
| 請求の管理                               | はい | いいえ   | いいえ   | いいえ   | いいえ   |

リソースの整合性に関するこれらのツールおよび機能と共に、デプロイされたリソースのパフォーマンスと正常性の問題を監視する必要があります。 [Azure Monitor](/azure/azure-monitor/overview) は Azure の既定の監視およびレポート ソリューションです。 Azure Monitor にはクラウド リソースの監視に使用できるいくつかの個別機能があり、次の一覧は、一般的な監視要件に対応するための機能を示します。

|                                                    | [Azure Portal](https://azure.microsoft.com/features/azure-portal/) | [Application Insights](/azure/application-insights/app-insights-overview) | [Log Analytics](/azure/azure-monitor/log-query/log-query-overview) | [Azure Monitor Rest API](/rest/api/monitor/) |
|----------------------------------------------------|--------------|----------------------|---------------|------------------------|
| 仮想マシンの利用統計情報をログに記録する                 | いいえ            | いいえ                    | はい           | いいえ                      |
| 仮想ネットワークの利用統計情報をログに記録する              | いいえ            | いいえ                    | はい           | いいえ                      |
| PaaS サービスの利用統計情報をログに記録する                   | いいえ            | いいえ                    | はい           | いいえ                      |
| アプリケーションの利用統計情報をログに記録する                     | いいえ            | はい                  | いいえ             | いいえ                      |
| レポートとアラートを構成する                       | はい          | いいえ                    | いいえ             | はい                    |
| 定期的なレポートまたはカスタム分析をスケジュールする        | いいえ            | いいえ                    | いいえ             | いいえ                      |
| ログとパフォーマンス データを視覚化および分析する     | はい          | いいえ                    | いいえ             | いいえ                      |
| オンプレミスまたはサード パーティ製の監視ソリューションと統合する     | いいえ            | いいえ                    | いいえ             | はい                    |

デプロイを計画するときには、ログ データの格納場所と、クラウド ベースの[レポート サービスと監視サービス](../../decision-guides/log-and-report/overview.md)を既存のプロセスとツールに統合する方法を検討する必要があります。

> [!NOTE]
> 組織では、ワークロードとリソースの監視にサード パーティ製の DevOps ツールも使用します。 詳細については、「[DevOps ツールの統合](https://azure.microsoft.com/products/devops-tool-integrations/)」を参照してください。

# <a name="next-steps"></a>次の手順

Azure で[ポリシー定義](/azure/governance/policy/)の作成、割り当て、管理を行う方法について説明します。
