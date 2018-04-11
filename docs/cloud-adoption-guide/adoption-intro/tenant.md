---
title: 'ガイダンス: Azure AD テナント設計'
description: 基本のクラウド導入戦略の一環である Azure テナント設計のガイダンス
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="ecb3f-103">ガイダンス: Azure AD テナント設計</span><span class="sxs-lookup"><span data-stu-id="ecb3f-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="ecb3f-104">Azure AD テナントでは、1 つ以上の [Azure サブスクリプション](subscription-explainer.md)に使用されるデジタル ID サービスおよび名前空間を提供しています。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="ecb3f-105">基本の導入の枠組みに従っている場合、[Azure AD テナントの取得方法][how-to-get-aad-tenant]については既に習得済みです。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="ecb3f-106">設計上の考慮事項</span><span class="sxs-lookup"><span data-stu-id="ecb3f-106">Design considerations</span></span>

- <span data-ttu-id="ecb3f-107">基本導入ステージでは、単一の Azure AD テナントから開始できます。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="ecb3f-108">所属する組織に既存の Office 365 サブスクリプションまたは Azure サブスクリプションがある場合は、使用できる Azure AD テナントを既にお持ちです。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="ecb3f-109">これらのサブスクリプションのいずれもない場合は、[Azure AD テナントの取得方法][how-to-get-aad-tenant]で詳細を確認できます。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="ecb3f-110">中間および高度の導入ステージでは、オンプレミス ディレクトリを Azure AD と同期または連動させる方法について学習します。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="ecb3f-111">これにより、オンプレミスのデジタル ID を Azure AD で使用できるようになります。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="ecb3f-112">ただし、基本ステージでは、単一の Azure AD テナントの ID のみを持つ新規ユーザーを追加することになります。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="ecb3f-113">これらの ID を管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="ecb3f-114">たとえば、新規 Azure AD ユーザーの登録、Azure リソースにもうアクセスするつもりがない Azure AD ユーザーの登録解除、およびその他のユーザー アクセス許可への変更があります。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ecb3f-115">次の手順</span><span class="sxs-lookup"><span data-stu-id="ecb3f-115">Next Steps</span></span>

* <span data-ttu-id="ecb3f-116">Azure AD テナントを入手したので、次は[ユーザーの追加方法][azure-ad-add-user]について学習します。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="ecb3f-117">Azure AD テネントに 1 人または複数の新規ユーザーを追加し終えたら、次のステップとして [Azure サブスクリプション](subscription-explainer.md)について学習します。</span><span class="sxs-lookup"><span data-stu-id="ecb3f-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
