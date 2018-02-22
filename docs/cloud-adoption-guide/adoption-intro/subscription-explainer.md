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
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="03d72-103">説明: Azure サブスクリプションとは</span><span class="sxs-lookup"><span data-stu-id="03d72-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="03d72-104">「[Azure Active Directory テナントとは](tenant-explainer.md)」の説明の記事で、組織のデジタル ID は Azure Active Directory テナントに保存されることを説明しました。</span><span class="sxs-lookup"><span data-stu-id="03d72-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="03d72-105">また、Azure は Azure Active Directory を信頼して、リソースの作成、読み取り、更新、または削除のユーザー要求を認証していることも説明しました。</span><span class="sxs-lookup"><span data-stu-id="03d72-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="03d72-106">リソースへのこれらの操作へのアクセスを制限する必要がある理由についても、本質的に理解したうえで、認証または承認されていないユーザーのリソースへのアクセスを防止します。</span><span class="sxs-lookup"><span data-stu-id="03d72-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="03d72-107">しかし、これらのリソースの操作には、作成が許可されるユーザーまたはユーザー グループの数やリソースを実行する際のコストなど、組織が制御したいと考える他の要素があります。</span><span class="sxs-lookup"><span data-stu-id="03d72-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="03d72-108">Azure では、**サブスクリプション**という名前で、この制御を実装しています。</span><span class="sxs-lookup"><span data-stu-id="03d72-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="03d72-109">サブスクリプションは、ユーザーとそれらのユーザーによって作成されたリソースをグループ化します。</span><span class="sxs-lookup"><span data-stu-id="03d72-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="03d72-110">それらの各リソースが、特定のリソースで[全体に対する制限][subscription-service-limits]を決定する要因となります。</span><span class="sxs-lookup"><span data-stu-id="03d72-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="03d72-111">組織では、サブスクリプションを使用してユーザーごと、チームごと、プロジェクトごとのコストやリソース作成を管理するなど、他にもさまざまな戦略を利用できます。</span><span class="sxs-lookup"><span data-stu-id="03d72-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="03d72-112">これらの戦略については、中間および高度の導入ステージに関する記事で説明します。</span><span class="sxs-lookup"><span data-stu-id="03d72-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="03d72-113">次の手順</span><span class="sxs-lookup"><span data-stu-id="03d72-113">Next steps</span></span>

* <span data-ttu-id="03d72-114">Azure サブスクリプションについて学習したので、最初の Azure リソースを作成する前に[サブスクリプションの作成](subscription.md)に関する詳細を確認してください。</span><span class="sxs-lookup"><span data-stu-id="03d72-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
