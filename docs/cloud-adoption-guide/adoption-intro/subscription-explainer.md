---
title: "説明: Azure サブスクリプションとは"
description: "Azure サブスクリプション、アカウント、およびプランについて説明します。"
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a>説明: Azure サブスクリプションとは

「[Azure Active Directory テナントとは](tenant-explainer.md)」の説明の記事で、組織のデジタル ID は Azure Active Directory テナントに保存されることを説明しました。 また、Azure は Azure Active Directory を信頼して、リソースの作成、読み取り、更新、または削除のユーザー要求を認証していることも説明しました。 

リソースへのこれらの操作へのアクセスを制限する必要がある理由についても、本質的に理解したうえで、認証または承認されていないユーザーのリソースへのアクセスを防止します。 しかし、これらのリソースの操作には、作成が許可されるユーザーまたはユーザー グループの数やリソースを実行する際のコストなど、組織が制御したいと考える他の要素があります。 

Azure では、**サブスクリプション**という名前で、この制御を実装しています。 サブスクリプションは、ユーザーとそれらのユーザーによって作成されたリソースをグループ化します。 それらの各リソースが、特定のリソースで[全体に対する制限][subscription-service-limits]を決定する要因となります。

組織では、サブスクリプションを使用してユーザーごと、チームごと、プロジェクトごとのコストやリソース作成を管理するなど、他にもさまざまな戦略を利用できます。 これらの戦略については、中間および高度の導入ステージに関する記事で説明します。 

## <a name="next-steps"></a>次の手順

* Azure サブスクリプションについて学習したので、最初の Azure リソースを作成する前に[サブスクリプションの作成](subscription.md)に関する詳細を確認してください。

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
