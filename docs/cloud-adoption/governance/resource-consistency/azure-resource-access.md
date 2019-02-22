---
title: CAF:Azure でのリソース アクセス管理
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure でのリソース アクセス管理の説明:Azure Resource Manager、サブスクリプション、リソース グループ、およびリソース
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902153"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="d57a5-103">Azure でのリソース アクセス管理</span><span class="sxs-lookup"><span data-stu-id="d57a5-103">Resource access management in Azure</span></span>

<span data-ttu-id="d57a5-104">[クラウド ガバナンス](../overview.md)に関するページでは、リソース管理など、クラウド ガバナンスの 5 つの規範について概要を説明してします。</span><span class="sxs-lookup"><span data-stu-id="d57a5-104">[Cloud Governance](../overview.md) outlines the five disciplines of Cloud Governance, which includes Resource Management.</span></span>  <span data-ttu-id="d57a5-105">[リソース アクセス ガバナンスの概要](overview.md)に関するページでは、リソース アクセス管理がリソース管理の規範にどのように適合するかについても説明しています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-105">[What is resource access governance](overview.md) furthers explains how resource access management fits into the resource management discipline.</span></span> <span data-ttu-id="d57a5-106">ガバナンス モデルを設計する方法について確認する前に、Azure のリソース アクセス管理の制御について理解することが重要です。</span><span class="sxs-lookup"><span data-stu-id="d57a5-106">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="d57a5-107">これらのリソース アクセス管理の制御の構成により、ご自身のガバナンス モデルの基礎が形成されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-107">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="d57a5-108">まずは、Azure でリソースがどのようにデプロイされるかを詳しく見ていきましょう。</span><span class="sxs-lookup"><span data-stu-id="d57a5-108">Begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="d57a5-109">Azure リソースとは</span><span class="sxs-lookup"><span data-stu-id="d57a5-109">What is an Azure resource?</span></span>

<span data-ttu-id="d57a5-110">Azure では、**リソース**という用語は、Azure によって管理されるエンティティを指します。</span><span class="sxs-lookup"><span data-stu-id="d57a5-110">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="d57a5-111">たとえば、仮想マシン、仮想ネットワーク、およびストレージ アカウントはすべて、Azure リソースと呼ばれます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-111">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="d57a5-112">![リソースの図](../../_images/governance-1-9.png)
*図 1.リソース。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-112">![Diagram of a resource](../../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="d57a5-113">Azure リソース グループとは</span><span class="sxs-lookup"><span data-stu-id="d57a5-113">What is an Azure resource group?</span></span>

<span data-ttu-id="d57a5-114">Azure の各リソースは、[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)に属している必要があります。</span><span class="sxs-lookup"><span data-stu-id="d57a5-114">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="d57a5-115">リソース グループは、複数のリソースを単一のエンティティとして管理できるようにまとめる、論理コンストラクトに過ぎません。</span><span class="sxs-lookup"><span data-stu-id="d57a5-115">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="d57a5-116">たとえば、[n 層アプリケーション](/azure/architecture/guide/architecture-styles/n-tier)のリソースなど、似たようなライフサイクルを共有するリソースを、グループとして作成または削除できます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-116">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="d57a5-117">![リソースを含むリソース グループの図](../../_images/governance-1-10.png)
*図 2.リソース グループにはリソースが含まれる。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-117">![Diagram of a resource group containing a resource](../../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="d57a5-118">リソース グループ、およびそのリソース グループに含まれているリソースは、Azure **サブスクリプション**に関連付けられます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-118">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="d57a5-119">Azure サブスクリプションとは</span><span class="sxs-lookup"><span data-stu-id="d57a5-119">What is an Azure subscription?</span></span>

<span data-ttu-id="d57a5-120">Azure サブスクリプションは、リソース グループとそのリソースをまとめる論理コンストラクトであるという点では、リソース グループに似ています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-120">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="d57a5-121">しかし、Azure サブスクリプションは、Azure Resource Manager によって使用される制御にも関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-121">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="d57a5-122">これはどういう意味でしょうか。</span><span class="sxs-lookup"><span data-stu-id="d57a5-122">What does this mean?</span></span> <span data-ttu-id="d57a5-123">Azure サブスクリプションと Azure Resource Manager の関係について確認するために、Azure Resource Manager を詳しく見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d57a5-123">Take a closer look at Azure Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="d57a5-124">![Azure サブスクリプションの図](../../_images/governance-1-11.png)
*図 3.Azure サブスクリプション。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-124">![Diagram of an Azure subscription](../../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="d57a5-125">Azure Resource Manager とは</span><span class="sxs-lookup"><span data-stu-id="d57a5-125">What is Azure Resource Manager?</span></span>

<span data-ttu-id="d57a5-126">[Azure のしくみ](../../getting-started/what-is-azure.md)に関するページでは、Azure には、Azure のすべての機能を調整する多くのサービスを備えた "フロントエンド" が含まれることを説明しました。</span><span class="sxs-lookup"><span data-stu-id="d57a5-126">In [how does Azure work?](../../getting-started/what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="d57a5-127">これらのサービスの 1 つが [Azure Resource Manager](/azure/azure-resource-manager/) で、このサービスは、リソースを管理するために、クライアントによって使用される RESTful API をホストしています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-127">One of these services is [Azure Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="d57a5-128">![Azure Resource Manager の図](../../_images/governance-1-12.png)
*図 4.Azure Resource Manager。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-128">![Diagram of Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span></span>

<span data-ttu-id="d57a5-129">次の図は、[PowerShell](/powershell/azure/overview)、[Azure portal](https://portal.azure.com)、および [Azure コマンド ライン インターフェイス (CLI)](/cli/azure) という 3 つのクライアントを示しています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-129">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command-line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="d57a5-130">![Azure Resource Manager API に接続する Azure クライアントの図](../../_images/governance-1-13.png)
*図 5.Azure クライアントが Azure Resource Manager RESTful API に接続する。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-130">![Diagram of Azure clients connecting to the Azure Resource Manager API](../../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Azure Resource Manager RESTful API.*</span></span>

<span data-ttu-id="d57a5-131">これらのクライアントは、RESTful API を使用して Azure Resource Manager に接続しますが、Azure Resource Manager には、リソースを直接管理する機能が含まれていません。</span><span class="sxs-lookup"><span data-stu-id="d57a5-131">While these clients connect to Azure Resource Manager using the RESTful API, Azure Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="d57a5-132">代わりに、Azure ではリソースの種類のほとんどに、独自の[**リソース プロバイダー**](/azure/azure-resource-manager/resource-group-overview#terminology)があります。</span><span class="sxs-lookup"><span data-stu-id="d57a5-132">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="d57a5-133">![Azure リソース プロバイダー](../../_images/governance-1-14.png)
*図 6.Azure リソース プロバイダー。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-133">![Azure resource providers](../../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="d57a5-134">クライアントが特定のリソースを管理するように要求すると、Azure Resource Manager は要求を完了するために、そのリソースの種類のリソース プロバイダーに接続します。</span><span class="sxs-lookup"><span data-stu-id="d57a5-134">When a client makes a request to manage a specific resource, Azure Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="d57a5-135">たとえば、クライアントが仮想マシン リソースを管理するように要求した場合、Azure Resource Manager は、**Microsoft.Compute** リソース プロバイダーに接続します。</span><span class="sxs-lookup"><span data-stu-id="d57a5-135">For example, if a client makes a request to manage a virtual machine resource, Azure Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="d57a5-136">![Azure Resource Manager から Microsoft.Compute リソース プロバイダーへの接続](../../_images/governance-1-15.png)
*図 7.Azure Resource Manager は、クライアント要求で指定されたリソースを管理する **Microsoft.Compute** リソース プロバイダーに接続します。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-136">![Azure Resource Manager connecting to the Microsoft.Compute resource provider](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="d57a5-137">Azure Resource Manager が、仮想マシン リソースを管理するために、サブスクリプションとリソース グループの両方の識別子を指定するようクライアントに要求しています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-137">Azure Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="d57a5-138">Azure Resource Manager のしくみがわかったので、Azure サブスクリプションが、Azure Resource Manager によって使用される制御にどのように関連付けられるかという説明に戻りましょう。</span><span class="sxs-lookup"><span data-stu-id="d57a5-138">Now that you have an understanding of how Azure Resource Manager works, return to the discussion of how an Azure subscription is associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="d57a5-139">Azure Resource Manager によってリソースの管理要求が実行される前に、一連の制御がチェックされます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-139">Before any resource management request can be executed by Azure Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="d57a5-140">最初の制御は、要求が必ず検証済みユーザーによって行われていることです。また、ユーザー ID 機能を提供するために、Azure Resource Manager は [Azure Active Directory (Azure AD)](/azure/active-directory/) との間に信頼関係を確保します。</span><span class="sxs-lookup"><span data-stu-id="d57a5-140">The first control is that a request must be made by a validated user, and Azure Resource Manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

<span data-ttu-id="d57a5-141">![Azure Active Directory](../../_images/governance-1-16.png)
*図 8.Azure Active Directory。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="d57a5-142">Azure AD では、ユーザーが**テナント**にセグメント化されています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-142">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="d57a5-143">テナントとは、通常は組織に関連付けられる Azure AD の安全な専用インスタンスを表す論理コンストラクターです。</span><span class="sxs-lookup"><span data-stu-id="d57a5-143">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="d57a5-144">各サブスクリプションが Azure AD テナントと関連付けられています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-144">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="d57a5-145">![サブスクリプションと関連付けられている Azure AD テナント](../../_images/governance-1-17.png)
*図 9.サブスクリプションと関連付けられている Azure AD テナント。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-145">![An Azure AD tenant associated with a subscription](../../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="d57a5-146">特定のサブスクリプションでリソースを管理するためのクライアント要求ごとに、関連付けられている Azure AD テナント内にユーザーがアカウントを持っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="d57a5-146">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="d57a5-147">次の制御では、要求を行うための十分なアクセス許可がユーザーにあることが確認されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-147">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="d57a5-148">[ロールベースのアクセス制御 (RBAC)](/azure/role-based-access-control/) を使用して、アクセス許可がユーザーに割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-148">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="d57a5-149">![RBAC ロールに割り当てられたユーザー](../../_images/governance-1-18.png)
*図 10.テナントの各ユーザーに 1 つ以上の RBAC ロールが割り当てらている。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-149">![Users assigned to RBAC roles](../../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="d57a5-150">RBAC ロールでは、特定のリソースに対してユーザーが適用できる一連のアクセス許可が指定されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-150">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="d57a5-151">ロールがユーザーに割り当てられると、これらのアクセス許可が適用されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-151">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="d57a5-152">たとえば、[組み込み**所有者**ロール](/azure/role-based-access-control/built-in-roles#owner)を使用すると、ユーザーはリソースに対して任意のアクションを実行できます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-152">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="d57a5-153">次の制御は、[Azure リソース ポリシー](/azure/governance/policy/)に適うように指定されている設定で、要求が許可されることのチェックです。</span><span class="sxs-lookup"><span data-stu-id="d57a5-153">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="d57a5-154">Azure リソース ポリシーでは、特定のリソースに対して許可される操作が指定されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-154">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="d57a5-155">たとえば、Azure リソース ポリシーを使用して、ユーザーが特定の種類の仮想マシンのみデプロイできるように指定できます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-155">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="d57a5-156">![Azure リソース ポリシー](../../_images/governance-1-19.png)
*図 11.Azure リソース ポリシー。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-156">![Azure resource policy](../../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="d57a5-157">次の制御は、要求が [Azure サブスクリプションの制限](/azure/azure-subscription-service-limits)を超えていないことのチェックです。</span><span class="sxs-lookup"><span data-stu-id="d57a5-157">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="d57a5-158">たとえば、サブスクリプションあたりのリソース グループ数は、980 グループに制限されています。</span><span class="sxs-lookup"><span data-stu-id="d57a5-158">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="d57a5-159">追加のリソース グループをデプロイする要求を受け取ったときに、制限に達している場合、その要求は拒否されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-159">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="d57a5-160">![Azure リソース制限](../../_images/governance-1-20.png)
*図 12.Azure リソース制限。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-160">![Azure resource limits](../../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="d57a5-161">最後の制御は、要求が、サブスクリプションに関連付けられている財務コミットメント内に収まっていることのチェックです。</span><span class="sxs-lookup"><span data-stu-id="d57a5-161">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="d57a5-162">たとえば、仮想マシンのデプロイ要求の場合は、サブスクリプションに十分な支払い情報があることが Azure Resource Manager によって確認されます。</span><span class="sxs-lookup"><span data-stu-id="d57a5-162">For example, if the request is to deploy a virtual machine, Azure Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="d57a5-163">![サブスクリプションに関連付けられている財務コミットメント](../../_images/governance-1-21.png)
*図 13.財務コミットメントはサブスクリプションに関連付けられています。*</span><span class="sxs-lookup"><span data-stu-id="d57a5-163">![A financial commitment associated with a subscription](../../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="d57a5-164">まとめ</span><span class="sxs-lookup"><span data-stu-id="d57a5-164">Summary</span></span>

<span data-ttu-id="d57a5-165">この記事では、Azure Resource Manager を使用して、リソース アクセスが Azure でどのように管理されるかを説明しました。</span><span class="sxs-lookup"><span data-stu-id="d57a5-165">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d57a5-166">次の手順</span><span class="sxs-lookup"><span data-stu-id="d57a5-166">Next steps</span></span>

<span data-ttu-id="d57a5-167">Azure でのリソース アクセスの管理方法については理解できました。次は、これらのサービスを使用して、[単純なワークロード](governance-simple-workload.md)または[複数チーム](governance-multiple-teams.md)向けのガバナンス モデルを設計する方法を説明します。</span><span class="sxs-lookup"><span data-stu-id="d57a5-167">Now that you understand how to manage resource access in Azure, move on to learn how to design a governance model [for a simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d57a5-168">ガバナンスの概要</span><span class="sxs-lookup"><span data-stu-id="d57a5-168">An overview of governance</span></span>](../overview.md)
