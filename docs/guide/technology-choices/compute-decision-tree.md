---
title: Azure コンピューティング サービスのデシジョン ツリー
description: コンピューティング サービスを選択するためのフローチャート
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: e601dcb653ed1809ea3f9bbda8db8b40efb460a5
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/29/2018
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

![](../images/compute-decision-tree.svg)

