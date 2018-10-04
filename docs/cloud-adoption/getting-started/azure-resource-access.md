---
title: 'エンタープライズ クラウドの導入: Azure でのリソース アクセス管理'
description: 'Azure のリソース アクセス管理コンストラクトの説明: Azure Resource Manager、サブスクリプション、リソース グループ、およびリソース'
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: c26b01d207bc4eb100653a826a053b394d03d1e6
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428705"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a><span data-ttu-id="b0b7c-103">エンタープライズ クラウドの導入: Azure でのリソース アクセス管理</span><span class="sxs-lookup"><span data-stu-id="b0b7c-103">Enterprise Cloud Adoption: Resource access management in Azure</span></span>

<span data-ttu-id="b0b7c-104">[クラウド リソース ガバナンス](what-is-governance.md)に関するページでは、ガバナンスとは、組織の目標と要件を満たすために、Azure リソースの使用を継続的に管理、監視、および監査するプロセスを意味することを説明しました。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="b0b7c-105">ガバナンス モデルを設計する方法について確認する前に、Azure のリソース アクセス管理の制御について理解することが重要です。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="b0b7c-106">これらのリソース アクセス管理の制御の構成により、ご自身のガバナンス モデルの基礎が形成されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="b0b7c-107">まずは、Azure でリソースがどのようにデプロイされるかを詳しく見ていきましょう。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-107">Let's begin by taking a closer look at how are resources are deployed in Azure.</span></span> 

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="b0b7c-108">Azure リソースとは</span><span class="sxs-lookup"><span data-stu-id="b0b7c-108">What is an Azure resource?</span></span>

<span data-ttu-id="b0b7c-109">Azure では、**リソース**という用語は、Azure によって管理されるエンティティを指します。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="b0b7c-110">たとえば、仮想マシン、仮想ネットワーク、およびストレージ アカウントはすべて、Azure リソースと呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

![](../_images/governance-1-9.png)   
<span data-ttu-id="b0b7c-111">*図 1: リソース。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-111">*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="b0b7c-112">Azure リソース グループとは</span><span class="sxs-lookup"><span data-stu-id="b0b7c-112">What is an Azure resource group?</span></span>

<span data-ttu-id="b0b7c-113">Azure の各リソースは、[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)に属している必要があります。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="b0b7c-114">リソース グループは、複数のリソースを単一のエンティティとして管理できるようにまとめる、論理コンストラクトに過ぎません。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="b0b7c-115">たとえば、[n 層アプリケーション](/azure/architecture/guide/architecture-styles/n-tier)のリソースなど、似たようなライフサイクルを共有するリソースを、グループとして作成または削除できます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span> 

![](../_images/governance-1-10.png)   
<span data-ttu-id="b0b7c-116">*図 2: リソース グループにはリソースが含まれる。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-116">*Figure 2. A resource group contains a resource.*</span></span> 

<span data-ttu-id="b0b7c-117">リソース グループ、およびそのリソース グループに含まれているリソースは、Azure **サブスクリプション**に関連付けられます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span> 

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="b0b7c-118">Azure サブスクリプションとは</span><span class="sxs-lookup"><span data-stu-id="b0b7c-118">What is an Azure subscription?</span></span>

<span data-ttu-id="b0b7c-119">Azure サブスクリプションは、リソース グループとそのリソースをまとめる論理コンストラクトであるという点では、リソース グループに似ています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="b0b7c-120">ただし、Azure サブスクリプションも、Azure Resource Manager によって使用される制御に関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-120">However, an Azure subscription is also associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="b0b7c-121">これはどういう意味でしょうか。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-121">What does this mean?</span></span> <span data-ttu-id="b0b7c-122">Azure サブスクリプションと Azure Resource Manager の関係について確認するために、Azure Resource Manager を詳しく見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-122">Let's take a closer look at Azure resource manager to learn about the relationship between it and an Azure subscription.</span></span>

![](../_images/governance-1-11.png)   
<span data-ttu-id="b0b7c-123">*図 3: Azure サブスクリプション。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-123">*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="b0b7c-124">Azure Resource Manager とは</span><span class="sxs-lookup"><span data-stu-id="b0b7c-124">What is Azure resource manager?</span></span>

<span data-ttu-id="b0b7c-125">[Azure のしくみ](what-is-azure.md)に関するページでは、Azure には、Azure のすべての機能を調整する多くのサービスを備えた "フロントエンド" が含まれることを説明しました。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="b0b7c-126">これらのサービスの 1 つが [Azure Resource Manager ](/azure/azure-resource-manager/)で、このサービスは、リソースを管理するために、クライアントによって使用される RESTful API をホストしています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-126">One of these services is [Azure resource manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span> 

![](../_images/governance-1-12.png)   
<span data-ttu-id="b0b7c-127">*図 4: Azure Resource Manager。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-127">*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="b0b7c-128">次の図は、[Powershell](/powershell/azure/overview)、[Azure portal](https://portal.azure.com)、および [Azure コマンド ライン インターフェイス (CLI)](/cli/azure) の 3 つのクライアントを示しています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-128">The following figure shows three clients: [Powershell](/powershell/azure/overview), [the Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

![](../_images/governance-1-13.png)   
<span data-ttu-id="b0b7c-129">*図 5: Azure クライアントが Azure Resource Manager RESTful API に接続する。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-129">*Figure 5. Azure clients connect to the Azure resource manager RESTful API.*</span></span>

<span data-ttu-id="b0b7c-130">これらのクライアントは、RESTful API を使用して Azure Resource Manager に接続しますが、Azure Resource Manager には、リソースを直接管理する機能が含まれていません。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-130">While these clients connect to Azure resource manager using the RESTful API, Azure resource manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="b0b7c-131">代わりに、Azure ではリソースの種類のほとんどに、独自の[**リソース プロバイダー**](/azure/azure-resource-manager/resource-group-overview#terminology)があります。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span> 

![](../_images/governance-1-14.png)   
<span data-ttu-id="b0b7c-132">*図 6: Azure リソース プロバイダー。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-132">*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="b0b7c-133">クライアントが特定のリソースを管理するように要求すると、Azure Resource Manager は要求を完了するために、そのリソースの種類のリソース プロバイダーに接続します。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-133">When a client makes a request to manage a specific resource, Azure resource manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="b0b7c-134">たとえば、クライアントが仮想マシン リソースを管理するように要求した場合、Azure Resource Manager は、**Microsoft.compute** リソース プロバイダーに接続します。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-134">For example, if a client makes a request to manage a virtual machine resource, Azure resource manager connects to the **Microsoft.compute** resource provider.</span></span> 

![](../_images/governance-1-15.png)   
<span data-ttu-id="b0b7c-135">*図 7: Azure Resource Manager が、クライアント要求で指定されたリソースを管理する **Microsoft.compute** リソース プロバイダーに接続する。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-135">*Figure 7. Azure resource manager connects to the **Microsoft.compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="b0b7c-136">Azure Resource Manager が、仮想マシン リソースを管理するために、サブスクリプションとリソース グループの両方の識別子を指定するようクライアントに要求しています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-136">Azure resource manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span> 

<span data-ttu-id="b0b7c-137">Azure Resource Manager のしくみがわかったので、Azure サブスクリプションが、Azure Resource Manager によって使用される制御にどのように関連付けられるかという説明に戻りましょう。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-137">Now that you have an understanding of how Azure resource manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="b0b7c-138">Azure Resource Manager によってリソースの管理要求が実行される前に、一連の制御がチェックされます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-138">Before any resource management request can be executed by Azure resource manager, a set of controls are checked.</span></span> 

<span data-ttu-id="b0b7c-139">最初の制御は、要求が必ず検証済みユーザーによって行われていることです。また、ユーザー ID 機能を提供するために、Azure Resource Manager は [Azure Active Directory (Azure AD)](/azure/active-directory/) との間に信頼関係を確保します。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-139">The first control is that a request must be made by a validated user, and Azure resource manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

![](../_images/governance-1-16.png)   
<span data-ttu-id="b0b7c-140">*図 8: Azure Active Directory。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-140">*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="b0b7c-141">Azure AD では、ユーザーが**テナント**にセグメント化されています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="b0b7c-142">テナントとは、通常は組織に関連付けられる Azure AD の安全な専用インスタンスを表す論理コンストラクターです。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="b0b7c-143">各サブスクリプションが Azure AD テナントと関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-143">Each subscription is associated with an Azure AD tenant.</span></span>

![](../_images/governance-1-17.png)   
<span data-ttu-id="b0b7c-144">*図 9: サブスクリプションと関連付けられている Azure AD テナント。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-144">*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="b0b7c-145">特定のサブスクリプションでリソースを管理するためのクライアント要求ごとに、関連付けられている Azure AD テナント内にユーザーがアカウントを持っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span> 

<span data-ttu-id="b0b7c-146">次の制御では、要求を行うための十分なアクセス許可がユーザーにあることが確認されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="b0b7c-147">[ロールベースのアクセス制御 (RBAC)](/azure/role-based-access-control/) を使用して、アクセス許可がユーザーに割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

![](../_images/governance-1-18.png)   
<span data-ttu-id="b0b7c-148">*図 10: テナントの各ユーザーに 1 つ以上の RBAC ロールが割り当てらている。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-148">*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="b0b7c-149">RBAC ロールでは、特定のリソースに対してユーザーが適用できる一連のアクセス許可が指定されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="b0b7c-150">ロールがユーザーに割り当てられると、これらのアクセス許可が適用されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="b0b7c-151">たとえば、[組み込み**所有者**ロール](/azure/role-based-access-control/built-in-roles#owner)を使用すると、ユーザーはリソースに対して任意のアクションを実行できます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="b0b7c-152">次の制御は、[Azure リソース ポリシー](/azure/governance/policy/)に適うように指定されている設定で、要求が許可されることのチェックです。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="b0b7c-153">Azure リソース ポリシーでは、特定のリソースに対して許可される操作が指定されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="b0b7c-154">たとえば、Azure リソース ポリシーを使用して、ユーザーが特定の種類の仮想マシンのみデプロイできるように指定できます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

![](../_images/governance-1-19.png)   
<span data-ttu-id="b0b7c-155">*図 11: Azure リソース ポリシー。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-155">*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="b0b7c-156">次の制御は、要求が [Azure サブスクリプションの制限](/azure/azure-subscription-service-limits)を超えていないことのチェックです。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="b0b7c-157">たとえば、サブスクリプションあたりのリソース グループ数は、980 グループに制限されています。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="b0b7c-158">追加のリソース グループをデプロイする要求を受け取ったときに、制限に達している場合、その要求は拒否されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

![](../_images/governance-1-20.png)   
<span data-ttu-id="b0b7c-159">*図 12: Azure リソース制限。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-159">*Figure 12. Azure resource limits.*</span></span> 

<span data-ttu-id="b0b7c-160">最後の制御は、要求が、サブスクリプションに関連付けられている財務コミットメント内に収まっていることのチェックです。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="b0b7c-161">たとえば、仮想マシンのデプロイ要求の場合は、サブスクリプションに十分な支払い情報があることが Azure Resource Manager によって確認されます。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-161">For example, if the request is to deploy a virtual machine, Azure resource manager verifies that the subscription has sufficient payment information.</span></span>

![](../_images/governance-1-21.png)   
<span data-ttu-id="b0b7c-162">*図 13: 財務コミットメントはサブスクリプションに関連付けられています。*</span><span class="sxs-lookup"><span data-stu-id="b0b7c-162">*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="b0b7c-163">まとめ</span><span class="sxs-lookup"><span data-stu-id="b0b7c-163">Summary</span></span>

<span data-ttu-id="b0b7c-164">この記事では、Azure Resource Manager を使用して、リソース アクセスが Azure でどのように管理されるかを説明しました。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-164">In this article, you learned about how resource access is managed in Azure using Azure resource manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b0b7c-165">次の手順</span><span class="sxs-lookup"><span data-stu-id="b0b7c-165">Next steps</span></span>

<span data-ttu-id="b0b7c-166">Azure でのリソース アクセスの管理方法については理解できました。次は、これらのサービスを使用して、[単一チーム](../governance/governance-single-team.md)または[複数チーム](../governance/governance-multiple-teams.md)向けのガバナンス モデルを設計する方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="b0b7c-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model [for a single team](../governance/governance-single-team.md) or [multiple teams](../governance/governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="b0b7c-167">ガバナンスの概要</span><span class="sxs-lookup"><span data-stu-id="b0b7c-167">An overview of governance</span></span>](../governance/overview.md)
