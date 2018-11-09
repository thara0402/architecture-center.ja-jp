---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 11/03/2018
ms.openlocfilehash: cb074272b8d00a71223d8c5755ef8cde3a3f2592
ms.sourcegitcommit: 225251ee2dd669432a9c9abe3aa8cd84d9e020b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2018
ms.locfileid: "50981982"
---
# <a name="decision-tree-for-azure-compute-services"></a>Azure コンピューティング サービスのデシジョン ツリー

Azure では、複数の方法でお使いのアプリケーション コードをホストできます。 "*コンピューティング*" という用語は、アプリケーションがそこで実行されるコンピューティング リソースのホスティング モデルを指します。 次のフローチャートは、お使いのアプリケーションのコンピューティング サービスを選択するうえで役立ちます。 このフローチャートは、推奨事項を導き出すための一連の主要な意思決定基準を示しています。 

**このフローチャートを原案として使用します。** すべてのアプリケーションには固有の要件があるため、推奨事項は原案として使用してください。 その後、詳細な評価を実行し、次の点を確認します。
 
- 機能セット
- [サービスの制限](/azure/azure-subscription-service-limits)
- [コスト](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [リージョン別の提供状況](https://azure.microsoft.com/global-infrastructure/services/)
- 開発者のエコシステムとチームのスキル
- [コンピューティングの比較表](./compute-comparison.md)

アプリケーションが複数のワークロードで構成されている場合は、それぞれのワークロードを個別に評価します。 完全なソリューションに、複数のコンピューティング サービスに組み込むことができます。

Azure でコンテナーをホストする際のオプションの詳細については、 https://azure.microsoft.com/overview/containers/ を参照してください。

## <a name="flowchart"></a>フローチャート

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a>定義

- **リフト アンド シフト:** アプリケーションの再設計やコード変更なしで、ワークロードをクラウドに移行する戦略です。 "*リホスト*" とも呼ばれます。 詳細については、「[Azure Migration Center](https://azure.microsoft.com/migration/)」を参照してください。

- **クラウド用に最適化:** アプリケーションをリファクタリングすることでクラウドネイティブの機能を利用して、クラウドに移行する戦略です。

## <a name="next-steps"></a>次の手順

考慮する必要がある追加条件については、「[Azure コンピューティング サービスを選択するための条件](./compute-comparison.md)」を参照してください。
