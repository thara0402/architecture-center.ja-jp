---
title: "AWS プロフェッショナルのための Azure"
description: "Microsoft Azure アカウント、プラットフォーム、およびサービスの基本を理解します。 AWS プラットフォームと Azure プラットフォームの主要な類似点と相違点についても説明します。 AWS の経験を Azure でご活用ください。"
keywords: "AWS エキスパート, Azure との比較, AWS との比較, Azure と AWS の違い, Azure と Aws"
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: ac96110e3fe69b4bb69714e18fd0f193208bc244
ms.sourcegitcommit: 744ad1381e01bbda6a1a7eff4b25e1a337385553
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2018
---
# <a name="azure-for-aws-professionals"></a><span data-ttu-id="1bb97-106">AWS プロフェッショナルのための Azure</span><span class="sxs-lookup"><span data-stu-id="1bb97-106">Azure for AWS Professionals</span></span>

<span data-ttu-id="1bb97-107">この記事は、アマゾン ウェブ サービス (AWS) エキスパートが Microsoft Azure のアカウント、プラットフォーム、およびサービスの基本を理解するために役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-107">This article helps Amazon Web Services (AWS) experts understand the basics of Microsoft Azure accounts, platform, and services.</span></span> <span data-ttu-id="1bb97-108">AWS プラットフォームと Azure プラットフォームの主要な類似点と相違点についても説明しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-108">It also covers key similarities and differences between the AWS and Azure platforms.</span></span>

<span data-ttu-id="1bb97-109">学習内容:</span><span class="sxs-lookup"><span data-stu-id="1bb97-109">You'll learn:</span></span>

* <span data-ttu-id="1bb97-110">アカウントとリソースが Azure でどのように構成されているか。</span><span class="sxs-lookup"><span data-stu-id="1bb97-110">How accounts and resources are organized in Azure.</span></span>
* <span data-ttu-id="1bb97-111">利用可能なソリューションが Azure でどのように構造化されているか。</span><span class="sxs-lookup"><span data-stu-id="1bb97-111">How available solutions are structured in Azure.</span></span>
* <span data-ttu-id="1bb97-112">主要な Azure サービスが AWS サービスとどのように異なっているか。</span><span class="sxs-lookup"><span data-stu-id="1bb97-112">How the major Azure services differ from AWS services.</span></span>

<span data-ttu-id="1bb97-113">Azure と AWS のさまざまな機能は時間の経過と共に独立して構築されたため、各機能の実装と設計には重要な相違点があります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-113">Azure and AWS built their capabilities independently over time so that each has important implementation and design differences.</span></span>

## <a name="overview"></a><span data-ttu-id="1bb97-114">概要</span><span class="sxs-lookup"><span data-stu-id="1bb97-114">Overview</span></span>

<span data-ttu-id="1bb97-115">AWS と同じように、Microsoft Azure も、コンピューティング、ストレージ、データベース、およびネットワーク サービスを中心に構築されています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-115">Like AWS, Microsoft Azure is built around a core set of compute, storage, database, and networking services.</span></span> <span data-ttu-id="1bb97-116">多くの場合、両方のプラットフォームが提供する製品とサービスは、基本的には同等です。</span><span class="sxs-lookup"><span data-stu-id="1bb97-116">In many cases, both platforms offer a basic equivalence between the products and services they offer.</span></span> <span data-ttu-id="1bb97-117">AWS と Azure の両方で、Windows または Linux ホストに基づく可用性が高いソリューションを構築できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-117">Both AWS and Azure allow you to build highly available solutions based on Windows or Linux hosts.</span></span> <span data-ttu-id="1bb97-118">したがって、Linux と OSS のテクノロジを使用する開発に慣れていれば、両方のプラットフォームでその仕事を行うことができます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-118">So, if you're used to development using Linux and OSS technology, both platforms can do the job.</span></span>

<span data-ttu-id="1bb97-119">2 つのプラットフォームの機能は似ていますが、それらの機能を提供するリソースの構成は、しばしば異なっています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-119">While the capabilities of both platforms are similar, the resources that provide those capabilities are often organized differently.</span></span> <span data-ttu-id="1bb97-120">ソリューションを構築するために必要なサービス間の一対一のリレーションシップは、常に明確であるとは限りません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-120">Exact one-to-one relationships between the services required to build a solution are not always clear.</span></span> <span data-ttu-id="1bb97-121">さらに、特定のサービスが片方のプラットフォームでのみ提供されている場合があります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-121">There are also cases where a particular service might be offered on one platform, but not the other.</span></span> <span data-ttu-id="1bb97-122">[Azure と AWS の対応するサービスの一覧](services.md)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-122">See [charts of comparable Azure and AWS services](services.md).</span></span>

## <a name="accounts-and-subscriptions"></a><span data-ttu-id="1bb97-123">アカウントとサブスクリプション</span><span class="sxs-lookup"><span data-stu-id="1bb97-123">Accounts and subscriptions</span></span>

<span data-ttu-id="1bb97-124">Azure サービスは、組織の規模とニーズに応じて、さまざまな価格オプションで購入できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-124">Azure services can be purchased using several pricing options, depending on your organization's size and needs.</span></span> <span data-ttu-id="1bb97-125">詳細については、[価格の概要](https://azure.microsoft.com/pricing/)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-125">See the [pricing overview](https://azure.microsoft.com/pricing/) page for details.</span></span>

<span data-ttu-id="1bb97-126">[Azure サブスクリプション](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)は、リソースと割り当てられた所有者をグループ化したものです。所有者は課金とアクセス許可の管理を担当します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-126">[Azure subscriptions](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) are a grouping of resources with an assigned owner responsible for billing and permissions management.</span></span> <span data-ttu-id="1bb97-127">特定の AWS アカウントの下に作成されたすべてのリソースはそのアカウントに関連付けられる AWS とは異なり、サブスクリプションは所有者アカウントとは無関係に存在し、必要に応じて新しい所有者に再割り当てすることができます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-127">Unlike AWS, where any resources created under the AWS account are tied to that account, subscriptions exist independently of their owner accounts, and can be reassigned to new owners as needed.</span></span>

<span data-ttu-id="1bb97-128">![AWS アカウントと Azure サブスクリプションの構造と所有権の比較](./images/azure-aws-account-compare.png "AWS アカウントと Azure サブスクリプションの構造と所有権の比較")</span><span class="sxs-lookup"><span data-stu-id="1bb97-128">![Comparison of structure and ownership of AWS accounts and Azure subscriptions](./images/azure-aws-account-compare.png "Comparison of structure and ownership of AWS accounts and Azure subscriptions")</span></span>
<br/><span data-ttu-id="1bb97-129">*AWS アカウントと Azure サブスクリプションの構造と所有権の比較*</span><span class="sxs-lookup"><span data-stu-id="1bb97-129">*Comparison of structure and ownership of AWS accounts and Azure subscriptions*</span></span>
<br/><br/>

<span data-ttu-id="1bb97-130">サブスクリプションには、3 種類の管理者アカウントが割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-130">Subscriptions are assigned three types of administrator accounts:</span></span>

-   <span data-ttu-id="1bb97-131">**アカウント管理者** - サブスクリプションの所有者であり、このアカウントに対して、サブスクリプションで使用されたリソースの課金が行われます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-131">**Account Administrator** - The subscription owner and the account billed for the resources used in the subscription.</span></span> <span data-ttu-id="1bb97-132">アカウント管理者は、サブスクリプションの所有権を譲渡することでのみ変更できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-132">The account administrator can only be changed by transferring ownership of the subscription.</span></span>

-   <span data-ttu-id="1bb97-133">**サービス管理者** - このアカウントには、サブスクリプション内にリソースを作成して管理する権限がありますが、課金は行われません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-133">**Service Administrator** - This account has rights to create and manage resources in the subscription, but is not responsible for billing.</span></span> <span data-ttu-id="1bb97-134">既定では、アカウント管理者とサービス管理者は同じアカウントに割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-134">By default, the account administrator and service administrator are assigned to the same account.</span></span> <span data-ttu-id="1bb97-135">アカウント管理者は、サブスクリプションの技術面と運用面を管理するサービス管理者アカウントに別のユーザーを割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-135">The account administrator can assign a separate user to the service administrator account for managing the technical and operational aspects of a subscription.</span></span> <span data-ttu-id="1bb97-136">サービス管理者はサブスクリプションごとに 1 人だけ存在します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-136">There is only one service administrator per subscription.</span></span>

-   <span data-ttu-id="1bb97-137">**共同管理者** - 1 つのサブスクリプションに複数の共同管理者アカウントを割り当てることができます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-137">**Co-administrator** - There can be multiple co-administrator accounts assigned to a subscription.</span></span> <span data-ttu-id="1bb97-138">共同管理者はサービス管理者を変更できませんが、それ以外は、サブスクリプションのリソースとユーザーを完全に制御できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-138">Co-administrators cannot change the service administrator, but otherwise have full control over subscription resources and users.</span></span>

<span data-ttu-id="1bb97-139">サブスクリプション レベルの下で、特定のリソースに対してユーザー ロールと個々のアクセス許可を割り当てることもできます。これは、AWS で IAM ユーザーとグループに対してアクセス許可を付与することに似ています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-139">Below the subscription level user roles and individual permissions can also be assigned to specific resources, similarly to how permissions are granted to IAM users and groups in AWS.</span></span> <span data-ttu-id="1bb97-140">Azure では、すべてのユーザー アカウントは、Microsoft アカウントまたは組織アカウント (Azure Active Directory を通して管理されるアカウント) に関連付けられます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-140">In Azure all user accounts are associated with either a Microsoft Account or Organizational Account (an account managed through an Azure Active Directory).</span></span>

<span data-ttu-id="1bb97-141">AWS アカウントと同じように、サブスクリプションには既定のサービスのクォータと制限があります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-141">Like AWS accounts, subscriptions have default service quotas and limits.</span></span> <span data-ttu-id="1bb97-142">これらの制限の完全な一覧については、「[Azure サブスクリプションとサービスの制限、クォータ、制約](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-142">For a full list of these limits, see [Azure subscription and service limits, quotas, and constraints](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).</span></span>
<span data-ttu-id="1bb97-143">これらの制限は、[管理ポータルでのサポート リクエストの申し込み](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)によって、最大値まで増やすことができまです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-143">These limits can be increased up to the maximum by [filing a support request in the management portal](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).</span></span>

### <a name="see-also"></a><span data-ttu-id="1bb97-144">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-144">See also</span></span>

-   [<span data-ttu-id="1bb97-145">Azure 管理者ロールを追加または変更する方法</span><span class="sxs-lookup"><span data-stu-id="1bb97-145">How to add or change Azure administrator roles</span></span>](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [<span data-ttu-id="1bb97-146">Azure の請求書と毎日の使用状況データをダウンロードする方法</span><span class="sxs-lookup"><span data-stu-id="1bb97-146">How to download your Azure billing invoice and daily usage data</span></span>](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a><span data-ttu-id="1bb97-147">リソース管理</span><span class="sxs-lookup"><span data-stu-id="1bb97-147">Resource management</span></span>

<span data-ttu-id="1bb97-148">Azure では、"リソース" という用語を AWS と同じように使用しています。つまり、すべてのコンピューティング インスタンス、ストレージ オブジェクト、ネットワーク デバイス、プラットフォームで作成または構成できるその他のエンティティを意味します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-148">The term "resource" in Azure is used in the same way as in AWS, meaning any compute instance, storage object, networking device, or other entity you can create or configure within the platform.</span></span>

<span data-ttu-id="1bb97-149">Azure リソースは、2 つのモデル ([Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) と従来の Azure [クラシック デプロイメント モデル](/azure/azure-resource-manager/resource-manager-deployment-model)) のどちらかでデプロイして管理されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-149">Azure resources are deployed and managed using one of two models: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview), or the older Azure [classic deployment model](/azure/azure-resource-manager/resource-manager-deployment-model).</span></span>
<span data-ttu-id="1bb97-150">すべての新しいリソースは、Resource Manager モデルを使用して作成されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-150">Any new resources are created using the Resource Manager model.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="1bb97-151">リソース グループ</span><span class="sxs-lookup"><span data-stu-id="1bb97-151">Resource groups</span></span>

<span data-ttu-id="1bb97-152">Azure と AWS の両方に、VM、ストレージ、仮想ネットワーク デバイスなどのリソースを整理する "リソース グループ" と呼ばれるエンティティがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-152">Both Azure and AWS have entities called "resource groups" that organize resources such as VMs, storage, and virtual networking devices.</span></span> <span data-ttu-id="1bb97-153">ただし、[Azure リソース グループ](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)と AWS リソース グループは同等ではありません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-153">However, [Azure resource groups](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) are not directly comparable to AWS resource groups.</span></span>

<span data-ttu-id="1bb97-154">AWS では、1 つのリソースを複数のリソース グループにタグ付けできますが、Azure のリソースは常に 1 つのリソース グループに関連付けられます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-154">While AWS allows a resource to be tagged into multiple resource groups, an Azure resource is always associated with one resource group.</span></span> <span data-ttu-id="1bb97-155">あるリソース グループ内に作成されたリソースを別のグループに移動できますが、リソースは一度に 1 つのリソース グループ内にのみ存在できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-155">A resource created in one resource group can be moved to another group, but can only be in one resource group at a time.</span></span> <span data-ttu-id="1bb97-156">リソース グループは、Azure Resource Manager によって使用される基本グループです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-156">Resource groups are the fundamental grouping used by Azure Resource Manager.</span></span>

<span data-ttu-id="1bb97-157">リソースは、[タグ](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)を使用して整理することもできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-157">Resources can also be organized using [tags](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).</span></span>
<span data-ttu-id="1bb97-158">タグは、サブスクリプションのリソースを、リソース グループのメンバーシップに関係なくグループ化できるようにするキーと値のペアです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-158">Tags are key-value pairs that allow you to group resources across your subscription irrespective of resource group membership.</span></span>

### <a name="management-interfaces"></a><span data-ttu-id="1bb97-159">管理インターフェイス</span><span class="sxs-lookup"><span data-stu-id="1bb97-159">Management interfaces</span></span>

<span data-ttu-id="1bb97-160">Azure では、さまざまな方法でリソースを管理できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-160">Azure offers several ways to manage your resources:</span></span>

-   <span data-ttu-id="1bb97-161">[Web インターフェイス](https://azure.microsoft.com/documentation/articles/resource-group-portal/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-161">[Web interface](https://azure.microsoft.com/documentation/articles/resource-group-portal/).</span></span>
    <span data-ttu-id="1bb97-162">AWS Dashboard と同じように、Azure ポータルは、Azure リソース用の完全な Web ベースの管理インターフェイスを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-162">Like the AWS Dashboard, the Azure portal provides a full web-based management interface for Azure resources.</span></span>

-   <span data-ttu-id="1bb97-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).</span></span>
    <span data-ttu-id="1bb97-164">Azure Resource Manager REST API は、Azure ポータルで使用できる機能の大半に、プログラムによってアクセスできるようにします。</span><span class="sxs-lookup"><span data-stu-id="1bb97-164">The Azure Resource Manager REST API provides programmatic access to most of the features available in the Azure portal.</span></span>

-   <span data-ttu-id="1bb97-165">[コマンド ライン](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-165">[Command Line](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).</span></span>
    <span data-ttu-id="1bb97-166">Azure CLI 2.0 ツールは、Azure リソースを作成して管理できるコマンド ライン インターフェイスを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-166">The Azure CLI 2.0 tool provides a command-line interface capable of creating and managing Azure resources.</span></span> <span data-ttu-id="1bb97-167">Azure CLI は [Windows、Linux、および Mac OS X](https://aka.ms/azurecli2) で使用できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-167">Azure CLI is available for [Windows, Linux, and Mac OS](https://aka.ms/azurecli2).</span></span>

-   <span data-ttu-id="1bb97-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).</span></span>
    <span data-ttu-id="1bb97-169">PowerShell 用の Azure モジュールによって、スクリプトを使用した自動管理タスクを実行できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-169">The Azure modules for PowerShell allow you to execute automated management tasks using a script.</span></span> <span data-ttu-id="1bb97-170">PowerShell は [Windows、Linux、および Mac OS X](https://github.com/PowerShell/PowerShell) で使用できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-170">PowerShell is available for [Windows, Linux, and Mac OS](https://github.com/PowerShell/PowerShell).</span></span>

-   <span data-ttu-id="1bb97-171">[テンプレート](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-171">[Templates](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).</span></span>
    <span data-ttu-id="1bb97-172">Azure Resource Manager テンプレートは、AWS CloudFormation サービスのような JSON テンプレート ベースのリソース管理機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-172">Azure Resource Manager templates provide similar JSON template-based resource management capabilities to the AWS CloudFormation service.</span></span>

<span data-ttu-id="1bb97-173">どのインターフェイスでも、Azure でリソースの作成、デプロイ、または変更を行うための中心はリソース グループです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-173">In each of these interfaces, the resource group is central to how Azure resources get created, deployed, or modified.</span></span> <span data-ttu-id="1bb97-174">これは、CloudFormation のデプロイ時に AWS リソースをグループ化するために "スタック" が果たす役割に似ています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-174">This is similar to the role a "stack" plays in grouping AWS resources during CloudFormation deployments.</span></span>

<span data-ttu-id="1bb97-175">これらのインターフェイスの構文と構造は AWS とは異なっていますが、同等の機能を備えています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-175">The syntax and structure of these interfaces are different from their AWS equivalents, but they provide comparable capabilities.</span></span> <span data-ttu-id="1bb97-176">さらに、AWS で使用される多数のサード パーティ製の管理ツール ([Hashicorp の Terraform](https://www.terraform.io/docs/providers/azurerm/) や [Netflix Spinnaker](http://www.spinnaker.io/) など) を Azure でも使用できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-176">In addition, many third party management tools used on AWS, like [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) and [Netflix Spinnaker](http://www.spinnaker.io/), are also available on Azure.</span></span>

### <a name="see-also"></a><span data-ttu-id="1bb97-177">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-177">See also</span></span>

-   [<span data-ttu-id="1bb97-178">Azure リソース グループのガイドライン</span><span class="sxs-lookup"><span data-stu-id="1bb97-178">Azure resource group guidelines</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a><span data-ttu-id="1bb97-179">リージョンとゾーン (高可用性)</span><span class="sxs-lookup"><span data-stu-id="1bb97-179">Regions and zones (high availability)</span></span>

<span data-ttu-id="1bb97-180">障害が及ぼす影響の範囲はさまざまです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-180">Failures can vary in the scope of their impact.</span></span> <span data-ttu-id="1bb97-181">たとえば、ディスク障害などの一部のハードウェア障害が、1 台のホスト コンピューターに影響を及ぼすことがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-181">Some hardware failures, such as a failed disk, may affect a single host machine.</span></span> <span data-ttu-id="1bb97-182">障害が発生したネットワーク スイッチが、サーバー ラック全体に影響する場合もあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-182">A failed network switch could affect a whole server rack.</span></span> <span data-ttu-id="1bb97-183">データ センターの停電など、データ センター全体を中断させる障害はまれです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-183">Less common are failures that disrupt a whole data center, such as loss of power in a data center.</span></span> <span data-ttu-id="1bb97-184">ほとんど発生しませんが、リージョン全体が利用できなくなることもあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-184">Rarely, an entire region could become unavailable.</span></span>

<span data-ttu-id="1bb97-185">アプリケーションの回復性を実現するための 1 つの手段として、冗長性があります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-185">One of the main ways to make an application resilient is through redundancy.</span></span> <span data-ttu-id="1bb97-186">ただし、この冗長性は、アプリケーションの設計時に計画しなければなりません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-186">But you need to plan for this redundancy when you design the application.</span></span> <span data-ttu-id="1bb97-187">また、必要な冗長性のレベルはビジネス要件によって異なります。リージョン障害に備えるために、すべてのアプリケーションにリージョン間での冗長性が必要だとは限りません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-187">Also, the level of redundancy that you need depends on your business requirements &mdash; not every application needs redundancy across regions to guard against a regional outage.</span></span> <span data-ttu-id="1bb97-188">一般的に、冗長性と信頼性を高めると、コストと複雑さが増すというトレードオフがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-188">In general, there is a tradeoff between greater redundancy and reliability versus higher cost and complexity.</span></span>  

<span data-ttu-id="1bb97-189">AWS では、リージョンは、2 つ以上の可用性ゾーンに分割されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-189">In AWS, a region is divided into two or more Availability Zones.</span></span> <span data-ttu-id="1bb97-190">可用性ゾーンは、地理的地域内に物理的に分離されたデータ センターに対応します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-190">An Availability Zone corresponds with a physically isolated datacenter in the geographic region.</span></span> <span data-ttu-id="1bb97-191">Azure には、**可用性セット**、**可用性ゾーン**、**ペアのリージョン**など、あらゆる障害レベルでアプリケーションの冗長性を確保するための機能が複数用意されています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-191">Azure has a number of features to make an application redundant at every level of failure, including **availability sets**, **availability zones**, and **paired regions**.</span></span> 

![](../resiliency/images/redundancy.svg)

<span data-ttu-id="1bb97-192">各オプションを以下の表に示します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-192">The following table summarizes each option.</span></span>

| &nbsp; | <span data-ttu-id="1bb97-193">可用性セット</span><span class="sxs-lookup"><span data-stu-id="1bb97-193">Availability Set</span></span> | <span data-ttu-id="1bb97-194">可用性ゾーン</span><span class="sxs-lookup"><span data-stu-id="1bb97-194">Availability Zone</span></span> | <span data-ttu-id="1bb97-195">ペアのリージョン</span><span class="sxs-lookup"><span data-stu-id="1bb97-195">Paired region</span></span> |
|--------|------------------|-------------------|---------------|
| <span data-ttu-id="1bb97-196">障害の範囲</span><span class="sxs-lookup"><span data-stu-id="1bb97-196">Scope of failure</span></span> | <span data-ttu-id="1bb97-197">ラック</span><span class="sxs-lookup"><span data-stu-id="1bb97-197">Rack</span></span> | <span data-ttu-id="1bb97-198">データセンター</span><span class="sxs-lookup"><span data-stu-id="1bb97-198">Datacenter</span></span> | <span data-ttu-id="1bb97-199">リージョン</span><span class="sxs-lookup"><span data-stu-id="1bb97-199">Region</span></span> |
| <span data-ttu-id="1bb97-200">要求のルーティング</span><span class="sxs-lookup"><span data-stu-id="1bb97-200">Request routing</span></span> | <span data-ttu-id="1bb97-201">Load Balancer</span><span class="sxs-lookup"><span data-stu-id="1bb97-201">Load Balancer</span></span> | <span data-ttu-id="1bb97-202">クロスゾーン ロード バランサー</span><span class="sxs-lookup"><span data-stu-id="1bb97-202">Cross-zone Load Balancer</span></span> | <span data-ttu-id="1bb97-203">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="1bb97-203">Traffic Manager</span></span> |
| <span data-ttu-id="1bb97-204">ネットワーク待ち時間</span><span class="sxs-lookup"><span data-stu-id="1bb97-204">Network latency</span></span> | <span data-ttu-id="1bb97-205">非常に低い</span><span class="sxs-lookup"><span data-stu-id="1bb97-205">Very low</span></span> | <span data-ttu-id="1bb97-206">低</span><span class="sxs-lookup"><span data-stu-id="1bb97-206">Low</span></span> | <span data-ttu-id="1bb97-207">中～高</span><span class="sxs-lookup"><span data-stu-id="1bb97-207">Mid to high</span></span> |
| <span data-ttu-id="1bb97-208">仮想ネットワーク</span><span class="sxs-lookup"><span data-stu-id="1bb97-208">Virtual networking</span></span>  | <span data-ttu-id="1bb97-209">VNet</span><span class="sxs-lookup"><span data-stu-id="1bb97-209">VNet</span></span> | <span data-ttu-id="1bb97-210">VNet</span><span class="sxs-lookup"><span data-stu-id="1bb97-210">VNet</span></span> | <span data-ttu-id="1bb97-211">リージョン間 VNet ピアリング (プレビュー)</span><span class="sxs-lookup"><span data-stu-id="1bb97-211">Cross-region VNet peering (preview)</span></span> |

### <a name="availability-sets"></a><span data-ttu-id="1bb97-212">可用性セット</span><span class="sxs-lookup"><span data-stu-id="1bb97-212">Availability sets</span></span> 

<span data-ttu-id="1bb97-213">ディスクやネットワーク スイッチの障害など、ローカライズされたハードウェアの障害から保護するには、可用性セットに 2 つ以上の VM をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="1bb97-213">To protect against localized hardware failures, such as a disk or network switch failing, deploy two or more VMs in an availability set.</span></span> <span data-ttu-id="1bb97-214">可用性セットは、電源とネットワーク スイッチを共有する 2 つ以上の "*障害ドメイン*" で構成されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-214">An availability set consists of two or more *fault domains* that share a common power source and network switch.</span></span> <span data-ttu-id="1bb97-215">可用性セットの VM は複数の障害ドメインに分散されるため、ある障害ドメインがハードウェア障害の影響を受けた場合は、ネットワーク トラフィックを他の障害ドメインの VM にルーティングできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-215">VMs in an availability set are distributed across the fault domains, so if a hardware failure affects one fault domain, network traffic can still be routed the VMs in the other fault domains.</span></span> <span data-ttu-id="1bb97-216">可用性セットの詳細については、「[Azure での Windows 仮想マシンの可用性の管理](/azure/virtual-machines/windows/manage-availability)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-216">For more information about Availability Sets, see [Manage the availability of Windows virtual machines in Azure](/azure/virtual-machines/windows/manage-availability).</span></span>

<span data-ttu-id="1bb97-217">VM インスタンスが可用性セットに追加されると、それらには[更新ドメイン](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)も割り当てられます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-217">When VM instances are added to availability sets, they are also assigned an [update domain](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/).</span></span> <span data-ttu-id="1bb97-218">更新ドメインは、計画済みメンテナンス イベントが同時に設定される VM グループです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-218">An update domain is a group of VMs that are set for planned maintenance events at the same time.</span></span> <span data-ttu-id="1bb97-219">複数の更新ドメインに VM を分散させることで、予定された更新と修正イベントが特定の時点でこれらの VM のサブセットのみに作用することが保証されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-219">Distributing VMs across multiple update domains ensures that planned update and patching events affect only a subset of these VMs at any given time.</span></span>

<span data-ttu-id="1bb97-220">アプリケーション内のインスタンスのロール別に可用性セットを編成して、各ロールで 1 つのインスタンスが操作可能であることを保証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-220">Availability sets should be organized by the instance's role in your application to ensure one instance in each role is operational.</span></span> <span data-ttu-id="1bb97-221">たとえば、3 層構造の Web アプリケーションでは、フロント エンド層、アプリケーション層、およびデータ層に個別の可用性セットを作成します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-221">For example, in a three-tier web application, create separate availability sets for the front-end, application, and data tiers.</span></span>

<span data-ttu-id="1bb97-222">![各アプリケーション ロール用の Azure 可用性セット](./images/three-tier-example.png "各アプリケーション ロール用の Azure 可用性セット")</span><span class="sxs-lookup"><span data-stu-id="1bb97-222">![Azure availability sets for each application role](./images/three-tier-example.png "Availability sets for each application role")</span></span>

### <a name="availability-zones-preview"></a><span data-ttu-id="1bb97-223">可用性ゾーン (プレビュー)</span><span class="sxs-lookup"><span data-stu-id="1bb97-223">Availability zones (preview)</span></span>

<span data-ttu-id="1bb97-224">[可用性ゾーン](/azure/availability-zones/az-overview)とは、Azure リージョン内の物理的に独立したゾーンのことです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-224">An [Availability Zone](/azure/availability-zones/az-overview) is a physically separate zone within an Azure region.</span></span> <span data-ttu-id="1bb97-225">可用性ゾーンはそれぞれ異なる電源、ネットワーク、および冷却装置を持ちます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-225">Each Availability Zone has a distinct power source, network, and cooling.</span></span> <span data-ttu-id="1bb97-226">可用性ゾーンに VM をデプロイすると、データセンター全体の障害からアプリケーションを保護するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-226">Deploying VMs across availability zones helps to protect an application against datacenter-wide failures.</span></span> 

### <a name="paired-regions"></a><span data-ttu-id="1bb97-227">ペアになっているリージョン</span><span class="sxs-lookup"><span data-stu-id="1bb97-227">Paired regions</span></span>

<span data-ttu-id="1bb97-228">リージョンの障害からアプリケーションを保護するために、複数のリージョンにアプリケーションをデプロイし、[Azure Traffic Manager][traffic-manager] を使用してインターネット トラフィックを異なるリージョンに分散できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-228">To protect an application against a regional outage, you can deploy the application across multiple regions, using [Azure Traffic Manager][traffic-manager] to distribute internet traffic to the different regions.</span></span> <span data-ttu-id="1bb97-229">各 Azure リージョンは別のリージョンとペアになります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-229">Each Azure region is paired with another region.</span></span> <span data-ttu-id="1bb97-230">これが[リージョン ペア][paired-regions]になります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-230">Together, these form a [regional pair][paired-regions].</span></span> <span data-ttu-id="1bb97-231">ブラジル南部を除き、リージョン ペアは、税および法の執行を目的としたデータ常駐要件を満たすために同じ地理的場所に配置されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-231">With the exception of Brazil South, regional pairs are located within the same geography in order to meet data residency requirements for tax and law enforcement jurisdiction purposes.</span></span>

<span data-ttu-id="1bb97-232">可用性ゾーン (物理的に独立しているが、比較的近接する地理的地域に配置されている場合があるデータセンター) とは異なり、ペアになっているリージョンは、通常は少なくとも 300 マイル離れています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-232">Unlike Availability Zones, which are physically separate datacenters but may be in relatively nearby geographic areas, paired regions are usually separated by at least 300 miles.</span></span> <span data-ttu-id="1bb97-233">その目的は、大規模災害がペアになっているリージョンの片方のみに影響を与えることを保証することです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-233">This is intended to ensure larger scale disasters only impact one of the regions in the pair.</span></span> <span data-ttu-id="1bb97-234">近接するペアにデータベースとステージ サービスのデータを同期するように設定して、プラットフォームの更新プログラムが、ペアになっているリージョンの片方にのみ同時にロールアウトされるように構成できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-234">Neighboring pairs can be set to sync database and storage service data, and are configured so that platform updates are rolled out to only one region in the pair at a time.</span></span>

<span data-ttu-id="1bb97-235">Azure の [geo 冗長ストレージ](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)は、適切なペアになっているリージョンに自動的にバックアップされます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-235">Azure [geo-redundant storage](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) is automatically backed up to the appropriate paired region.</span></span> <span data-ttu-id="1bb97-236">他のすべてのリソースでは、ペアになっているリージョンを使用した完全に冗長なソリューションの作成は、両方のリージョンにソリューションの完全なコピーを作成することを意味します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-236">For all other resources, creating a fully redundant solution using paired regions means creating a full copy of your solution in both regions.</span></span>


### <a name="see-also"></a><span data-ttu-id="1bb97-237">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-237">See also</span></span>

-   [<span data-ttu-id="1bb97-238">Azure の仮想マシンのリージョンと可用性について</span><span class="sxs-lookup"><span data-stu-id="1bb97-238">Regions and availability for virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [<span data-ttu-id="1bb97-239">Azure アプリケーションの高可用性</span><span class="sxs-lookup"><span data-stu-id="1bb97-239">High availability for Azure applications</span></span>](../resiliency/high-availability-azure-applications.md)

-   [<span data-ttu-id="1bb97-240">Azure アプリケーションのディザスター リカバリー</span><span class="sxs-lookup"><span data-stu-id="1bb97-240">Disaster recovery for Azure applications</span></span>](../resiliency/disaster-recovery-azure-applications.md)

-   [<span data-ttu-id="1bb97-241">Azure での Linux 仮想マシンに対する計画的なメンテナンス</span><span class="sxs-lookup"><span data-stu-id="1bb97-241">Planned maintenance for Linux virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a><span data-ttu-id="1bb97-242">サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-242">Services</span></span>

<span data-ttu-id="1bb97-243">プラットフォーム間のすべてのサービスの対応の一覧については、[AWS と Azure のサービスの完全比較マトリックス](https://aka.ms/azure4aws-services)を参照してください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-243">Consult the [complete AWS and Azure service comparison matrix](https://aka.ms/azure4aws-services) for a full listing of how all services map between platforms.</span></span>

<span data-ttu-id="1bb97-244">すべての Azure 製品とサービスがすべてのリージョンで使用できるわけではありません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-244">Not all Azure products and services are available in all regions.</span></span> <span data-ttu-id="1bb97-245">[リージョン別の製品](https://azure.microsoft.com/regions/services/)に関するページを参照してください。</span><span class="sxs-lookup"><span data-stu-id="1bb97-245">Consult the [Products by Region](https://azure.microsoft.com/regions/services/) page for details.</span></span> <span data-ttu-id="1bb97-246">Azure の各製品またはサービスのアップタイム保証とダウンタイム クレジット ポリシーについては、「[サービス レベル アグリーメント](https://azure.microsoft.com/support/legal/sla/)」ページで確認できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-246">You can find the uptime guarantees and downtime credit policies for each Azure product or service on the [Service Level Agreements](https://azure.microsoft.com/support/legal/sla/) page.</span></span>

<span data-ttu-id="1bb97-247">この後のセクションでは、一般的に使用される機能とサービスの AWS プラットフォームと Azure プラットフォームでの違いについて、簡単に説明します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-247">The following sections provide a brief explanation of how commonly used features and services differ between the AWS and Azure platforms.</span></span>

### <a name="compute-services"></a><span data-ttu-id="1bb97-248">Compute Services</span><span class="sxs-lookup"><span data-stu-id="1bb97-248">Compute services</span></span>

#### <a name="ec2-instances-and-azure-virtual-machines"></a><span data-ttu-id="1bb97-249">EC2 インスタンスと Azure 仮想マシン</span><span class="sxs-lookup"><span data-stu-id="1bb97-249">EC2 Instances and Azure virtual machines</span></span>

<span data-ttu-id="1bb97-250">AWS インスタンスのタイプと Azure 仮想マシンのサイズは、似たような方法で分類できますが、RAM、CPU、およびストレージの機能に違いがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-250">Although AWS instance types and Azure virtual machine sizes breakdown in a similar way, there are differences in the RAM, CPU, and storage capabilities.</span></span>

-   [<span data-ttu-id="1bb97-251">Amazon EC2 インスタンスのタイプ</span><span class="sxs-lookup"><span data-stu-id="1bb97-251">Amazon EC2 Instance Types</span></span>](https://aws.amazon.com/ec2/instance-types/)

-   [<span data-ttu-id="1bb97-252">Azure の仮想マシンのサイズ (Windows)</span><span class="sxs-lookup"><span data-stu-id="1bb97-252">Sizes for virtual machines in Azure (Windows)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [<span data-ttu-id="1bb97-253">Azure の仮想マシンのサイズ (Linux)</span><span class="sxs-lookup"><span data-stu-id="1bb97-253">Sizes for virtual machines in Azure (Linux)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

<span data-ttu-id="1bb97-254">AWS の秒単位の課金とは異なり、Azure のオンデマンド VM は分単位で課金されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-254">Unlike AWS' per second billing, Azure on-demand VMs are billed by the minute.</span></span>

<span data-ttu-id="1bb97-255">Azure には EC2 スポット インスタンスと専用ホストに相当するものはありません。</span><span class="sxs-lookup"><span data-stu-id="1bb97-255">Azure has no equivalent to EC2 Spot Instances or Dedicated Hosts.</span></span>

#### <a name="ebs-and-azure-storage-for-vm-disks"></a><span data-ttu-id="1bb97-256">EBS と VM ディスク用の Azure Storage</span><span class="sxs-lookup"><span data-stu-id="1bb97-256">EBS and Azure Storage for VM disks</span></span>

<span data-ttu-id="1bb97-257">Blob ストレージに存在する[データ ディスク](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)によって、Azure VM の持続性のあるデータ ストレージが提供されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-257">Durable data storage for Azure VMs is provided by [data disks](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) residing in blob storage.</span></span> <span data-ttu-id="1bb97-258">これは、EC2 インスタンスが Elastic Block Store (EBS) にディスク ボリュームを格納することに似ています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-258">This is similar to how EC2 instances store disk volumes on Elastic Block Store (EBS).</span></span> <span data-ttu-id="1bb97-259">[Azure の一時ストレージ](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)も、EC2 Instance Storage と同じ待機時間の短い一時的な読み取り/書き込みストレージ (短期ストレージとも呼ばれます) を VM に提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-259">[Azure temporary storage](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) also provides VMs the same low-latency temporary read-write storage as EC2 Instance Storage (also called ephemeral storage).</span></span>

<span data-ttu-id="1bb97-260">高パフォーマンスのディスク IO は、[Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage) を使用してサポートされます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-260">Higher performance disk IO is supported using [Azure premium storage](https://docs.microsoft.com/azure/storage/storage-premium-storage).</span></span>
<span data-ttu-id="1bb97-261">これは、AWS が提供する Provisioned IOPS ストレージ オプションに似ています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-261">This is similar to the Provisioned IOPS storage options provided by AWS.</span></span>

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a><span data-ttu-id="1bb97-262">Lambda、Azure Functions、Azure Web-Jobs、および Azure Logic Apps</span><span class="sxs-lookup"><span data-stu-id="1bb97-262">Lambda, Azure Functions, Azure Web-Jobs, and Azure Logic Apps</span></span>

<span data-ttu-id="1bb97-263">[Azure Functions](https://azure.microsoft.com/services/functions/) は、サーバーレスなオンデマンド コードを提供するという点で、AWS Lambda に相当します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-263">[Azure Functions](https://azure.microsoft.com/services/functions/) is the primary equivalent of AWS Lambda in providing serverless, on-demand code.</span></span>
<span data-ttu-id="1bb97-264">ただし、Lambda の機能は、他の Azure サービスとも重複しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-264">However, Lambda functionality also overlaps with other Azure services:</span></span>

-   <span data-ttu-id="1bb97-265">[WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - スケジュールされた、または継続的に実行されるバック グラウンド タスクを作成できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-265">[WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - allow you to create scheduled or continuously running background tasks.</span></span>

-   <span data-ttu-id="1bb97-266">[Logic Apps](https://azure.microsoft.com/services/logic-apps/) - 通信、統合、およびビジネス ルール管理サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-266">[Logic Apps](https://azure.microsoft.com/services/logic-apps/) - provides communications, integration, and business rule management services.</span></span>

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a><span data-ttu-id="1bb97-267">自動スケール、Azure VM のスケーリング、および Azure App Service の自動スケール</span><span class="sxs-lookup"><span data-stu-id="1bb97-267">Autoscaling, Azure VM scaling, and Azure App Service Autoscale</span></span>

<span data-ttu-id="1bb97-268">Azure の自動スケールは、2 つのサービスによって処理されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-268">Autoscaling in Azure is handled by two services:</span></span>

-   <span data-ttu-id="1bb97-269">[VM スケール セット](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - 同一の VM セットをデプロイして管理できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-269">[VM scale sets](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - allow you to deploy and manage an identical set of VMs.</span></span> <span data-ttu-id="1bb97-270">インスタンスの数は、パフォーマンスのニーズに基づいて自動スケールできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-270">The number of instances can autoscale based on performance needs.</span></span>

-   <span data-ttu-id="1bb97-271">[App Service の自動スケール](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - Azure App Service ソリューションを自動スケールする機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-271">[App Service Autoscale](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - provides the capability to autoscale Azure App Service solutions.</span></span>


#### <a name="container-service"></a><span data-ttu-id="1bb97-272">Container Service</span><span class="sxs-lookup"><span data-stu-id="1bb97-272">Container Service</span></span>
<span data-ttu-id="1bb97-273">[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) は、Docker Swarm、Kubernetes、または DC/OS を介して管理されている Docker コンテナーをサポートします。</span><span class="sxs-lookup"><span data-stu-id="1bb97-273">The [Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) supports Docker containers managed through Docker Swarm, Kubernetes, or DC/OS.</span></span>

#### <a name="other-compute-services"></a><span data-ttu-id="1bb97-274">その他のコンピューティング サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-274">Other compute services</span></span> 


<span data-ttu-id="1bb97-275">Azure には、AWS に直接相当するものがないいくつかのコンピューティング サービスがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-275">Azure offers several compute services that do not have direct equivalents in AWS:</span></span>

-   <span data-ttu-id="1bb97-276">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 仮想マシンのスケーラブルなコレクション全体にわたってコンピューティング集中型の作業を管理できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-276">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - allows you to manage compute-intensive work across a scalable collection of virtual machines.</span></span>

-   <span data-ttu-id="1bb97-277">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 拡張性の高い[マイクロサービス](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) ソリューションを開発してホストするためのプラットフォームです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-277">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - platform for developing and hosting scalable [microservice](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) solutions.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-278">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-278">See also</span></span>

-   [<span data-ttu-id="1bb97-279">ポータルを使用して Azure に Linux VM を作成する</span><span class="sxs-lookup"><span data-stu-id="1bb97-279">Create a Linux VM on Azure using the Portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [<span data-ttu-id="1bb97-280">Azure リファレンス アーキテクチャ: Azure での Linux VM の実行</span><span class="sxs-lookup"><span data-stu-id="1bb97-280">Azure Reference Architecture: Running a Linux VM on Azure</span></span>](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [<span data-ttu-id="1bb97-281">Get started with Node.js web apps in Azure App Service (Azure App Service で Node.js Web アプリの使用を開始する)</span><span class="sxs-lookup"><span data-stu-id="1bb97-281">Get started with Node.js web apps in Azure App Service</span></span>](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [<span data-ttu-id="1bb97-282">Azure リファレンス アーキテクチャ: 基本的な Web アプリケーション</span><span class="sxs-lookup"><span data-stu-id="1bb97-282">Azure Reference Architecture: Basic web application</span></span>](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [<span data-ttu-id="1bb97-283">初めての Azure 関数の作成</span><span class="sxs-lookup"><span data-stu-id="1bb97-283">Create your first Azure Function</span></span>](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a><span data-ttu-id="1bb97-284">Storage</span><span class="sxs-lookup"><span data-stu-id="1bb97-284">Storage</span></span>

#### <a name="s3ebsefs-and-azure-storage"></a><span data-ttu-id="1bb97-285">S3/EBS/EFS と Azure Storage</span><span class="sxs-lookup"><span data-stu-id="1bb97-285">S3/EBS/EFS and Azure Storage</span></span>

<span data-ttu-id="1bb97-286">AWS プラットフォームでは、クラウド ストレージは主に 3 つのサービスに分類されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-286">In the AWS platform, cloud storage is primarily broken down into three services:</span></span>

-   <span data-ttu-id="1bb97-287">**Simple Storage Service (S3)** - 基本的なオブジェクト ストレージ。</span><span class="sxs-lookup"><span data-stu-id="1bb97-287">**Simple Storage Service (S3)** - basic object storage.</span></span> <span data-ttu-id="1bb97-288">インターネットでアクセス可能な API を介してデータを利用できるようにします。</span><span class="sxs-lookup"><span data-stu-id="1bb97-288">Makes data available through an Internet accessible API.</span></span>

-   <span data-ttu-id="1bb97-289">**Elastic Block Storage (EBS)** - 1 台の VM によるアクセス向きのブロック レベルのストレージ。</span><span class="sxs-lookup"><span data-stu-id="1bb97-289">**Elastic Block Storage (EBS)** - block level storage, intended for access by a single VM.</span></span>

-   <span data-ttu-id="1bb97-290">**Elastic File System (EFS)** - 数千の EC2 インスタンスの共有ストレージとして使用するためのファイル ストレージ。</span><span class="sxs-lookup"><span data-stu-id="1bb97-290">**Elastic File System (EFS)** - file storage meant for use as shared storage for up to thousands of EC2 instances.</span></span>

<span data-ttu-id="1bb97-291">Azure Storage では、サブスクリプションに関連付けられた[ストレージ アカウント](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)を使用して、次のストレージ サービスを作成して管理できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-291">In Azure Storage, subscription-bound [storage accounts](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) allow you to create and manage the following storage services:</span></span>

-   <span data-ttu-id="1bb97-292">[Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 任意の種類のテキスト データやバイナリ データを格納します (ドキュメント、メディア ファイル、アプリケーション インストーラーなど)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-292">[Blob storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - stores any type of text or binary data, such as a document, media file, or application installer.</span></span> <span data-ttu-id="1bb97-293">BLOB ストレージをプライベート アクセス用に設定したり、インターネットに公開してコンテンツを共有したりできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-293">You can set Blob storage for private access or share contents publicly to the Internet.</span></span> <span data-ttu-id="1bb97-294">BLOB ストレージの目的は、AWS の S3 と EBS の両方の目的と同じです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-294">Blob storage serves the same purpose as both AWS S3 and EBS.</span></span>
-   <span data-ttu-id="1bb97-295">[Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 構造型データセットを格納します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-295">[Table storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - stores structured datasets.</span></span> <span data-ttu-id="1bb97-296">Table Storage は、NoSQL キー属性データ ストアであるため、開発が迅速化され、大量のデータにすばやくアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-296">Table storage is a NoSQL key-attribute data store that allows for rapid development and fast access to large quantities of data.</span></span> <span data-ttu-id="1bb97-297">AWS の SimpleDB サービスと DynamoDB サービスに似ています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-297">Similar to AWS' SimpleDB and DynamoDB services.</span></span>

-   <span data-ttu-id="1bb97-298">[Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - クラウド サービスのコンポーネント間のワークフロー処理と通信のためのメッセージング機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-298">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - provides messaging for workflow processing and for communication between components of cloud services.</span></span>

-   <span data-ttu-id="1bb97-299">[File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 標準的なサーバー メッセージ ブロック (SMB) プロトコルを使用するレガシー アプリケーション用の共有ストレージを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-299">[File storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - offers shared storage for legacy applications using the standard server message block (SMB) protocol.</span></span> <span data-ttu-id="1bb97-300">File Storageは、AWS プラットフォームの EFS と同じように使用されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-300">File storage is used in a similar manner to EFS in the AWS platform.</span></span>
 
#### <a name="glacier-and-azure-storage"></a><span data-ttu-id="1bb97-301">Glacier と Azure Storage</span><span class="sxs-lookup"><span data-stu-id="1bb97-301">Glacier and Azure Storage</span></span> 

<span data-ttu-id="1bb97-302">[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) は、AWS Glacier ストレージ サービスに相当します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-302">[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) is comparable to AWS Glacier storage service.</span></span> <span data-ttu-id="1bb97-303">これは、少なくとも 180 日格納され、数時間の取得待ち時間が許容される、アクセス頻度が非常に少ないデータを対象としています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-303">It is intended for rarely accessed data that is stored for at least 180 days and can tolerate several hours of retrieval latency.</span></span> 

<span data-ttu-id="1bb97-304">アクセス頻度が少ないが、アクセス時にすぐに使用できる必要があるデータについては、[Azure クール BLOB ストレージ層](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier)のストレージを使用すると、標準の BLOB ストレージよりもコストを抑えることができます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-304">For data that is infrequently accessed but must be available immediately when accessed, [Azure Cool Blob Storage tier](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) provides cheaper storage than standard blob storage.</span></span> <span data-ttu-id="1bb97-305">このストレージ層は、AWS S3 (Infrequent Access ストレージ サービス) に相当します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-305">This storage tier is comparable to AWS S3 - Infrequent Access storage service.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-306">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-306">See also</span></span>

-   [<span data-ttu-id="1bb97-307">Microsoft Azure Storage のパフォーマンスとスケーラビリティに対するチェック リスト</span><span class="sxs-lookup"><span data-stu-id="1bb97-307">Microsoft Azure Storage Performance and Scalability Checklist</span></span>](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [<span data-ttu-id="1bb97-308">Azure Storage セキュリティ ガイド</span><span class="sxs-lookup"><span data-stu-id="1bb97-308">Azure Storage security guide</span></span>](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [<span data-ttu-id="1bb97-309">パターンとプラクティス: コンテンツ配信ネットワーク (CDN) のガイダンス</span><span class="sxs-lookup"><span data-stu-id="1bb97-309">Patterns & Practices: Content Delivery Network (CDN) guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a><span data-ttu-id="1bb97-310">ネットワーク</span><span class="sxs-lookup"><span data-stu-id="1bb97-310">Networking</span></span>

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a><span data-ttu-id="1bb97-311">Elastic Load Balancing、Azure Load Balancer、および Azure Application Gateway</span><span class="sxs-lookup"><span data-stu-id="1bb97-311">Elastic Load Balancing, Azure Load Balancer, and Azure Application Gateway</span></span>

<span data-ttu-id="1bb97-312">Azure には、Elastic Load Balancing に相当する 2 つのサービスがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-312">The Azure equivalents of the two Elastic Load Balancing services are:</span></span>

-   <span data-ttu-id="1bb97-313">[Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - AWS Classic Load Balancer と同じ機能を提供し、複数の VM のトラフィックをネットワーク レベルで分散します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-313">[Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - provides the same capabilities as the AWS Classic Load Balancer, allowing you to distribute traffic for multiple VMs at the network level.</span></span> <span data-ttu-id="1bb97-314">フェールオーバー機能も提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-314">It also provides failover capability.</span></span>

-   <span data-ttu-id="1bb97-315">[Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - AWS Application Load Balancer に相当するアプリケーション レベルのルール ベースのルーティングを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-315">[Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - offers application-level rule-based routing comparable to the AWS Application Load Balancer.</span></span>

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a><span data-ttu-id="1bb97-316">Route 53、Azure DNS、および Azure Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="1bb97-316">Route 53, Azure DNS, and Azure Traffic Manager</span></span>

<span data-ttu-id="1bb97-317">AWS の Route 53 では、DNS 名管理と DNS レベルのトラフィック ルーティング サービスと、フェールオーバー サービスの両方を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-317">In AWS, Route 53 provides both DNS name management and DNS-level traffic routing and failover services.</span></span> <span data-ttu-id="1bb97-318">Azure では、これは 2 つのサービスで処理されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-318">In Azure this is handled through two services:</span></span>

-   <span data-ttu-id="1bb97-319">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/) は、ドメインと DNS の管理を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-319">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/) provides domain and DNS management.</span></span>

-   <span data-ttu-id="1bb97-320">[Traffic Manager][traffic-manager] は、DNS レベルのトラフィック ルーティング、負荷分散、およびフェールオーバー機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-320">[Traffic Manager][traffic-manager] provides DNS level traffic routing, load balancing, and failover capabilities.</span></span>

#### <a name="direct-connect-and-azure-expressroute"></a><span data-ttu-id="1bb97-321">Direct Connect と Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="1bb97-321">Direct Connect and Azure ExpressRoute</span></span>

<span data-ttu-id="1bb97-322">Azure では、[ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) サービスを介して、同様のサイト間専用接続を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-322">Azure provides similar site-to-site dedicated connections through its [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) service.</span></span> <span data-ttu-id="1bb97-323">ExpressRoute では、専用のプライベート ネットワーク接続を使用して、ローカル ネットワークを Azure のリソースに直接接続できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-323">ExpressRoute allows you to connect your local network directly to Azure resources using a dedicated private network connection.</span></span> <span data-ttu-id="1bb97-324">Azure では、従来の[サイト間 VPN 接続](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)も低コストで提供しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-324">Azure also offers more conventional [site-to-site VPN connections](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) at a lower cost.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-325">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-325">See also</span></span>

-   [<span data-ttu-id="1bb97-326">Azure Portal を使用した仮想ネットワークの作成</span><span class="sxs-lookup"><span data-stu-id="1bb97-326">Create a virtual network using the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [<span data-ttu-id="1bb97-327">Azure Virtual Network の計画と設計</span><span class="sxs-lookup"><span data-stu-id="1bb97-327">Plan and design Azure Virtual Networks</span></span>](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [<span data-ttu-id="1bb97-328">Azure のネットワーク セキュリティに関するベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="1bb97-328">Azure Network Security Best Practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a><span data-ttu-id="1bb97-329">データベース サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-329">Database services</span></span>

#### <a name="rds-and-azure-relational-database-services"></a><span data-ttu-id="1bb97-330">RDS と Azure リレーショナル データベース サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-330">RDS and Azure relational database services</span></span>

<span data-ttu-id="1bb97-331">Azure には、AWS の Relational Database Service (RDS) に相当するリレーショナル データベース サービスが何種類かあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-331">Azure provides several different relational database services that are the equivalent of AWS' Relational Database Service (RDS).</span></span>

-   [<span data-ttu-id="1bb97-332">SQL Database</span><span class="sxs-lookup"><span data-stu-id="1bb97-332">SQL Database</span></span>](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [<span data-ttu-id="1bb97-333">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="1bb97-333">Azure Database for MySQL</span></span>](https://docs.microsoft.com/azure/mysql/overview)
-   [<span data-ttu-id="1bb97-334">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="1bb97-334">Azure Database for PostgreSQL</span></span>](https://docs.microsoft.com/azure/postgresql/overview)

<span data-ttu-id="1bb97-335">[SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/)、[Oracle](https://azure.microsoft.com/campaigns/oracle/)、[MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) などの他のデータベース エンジンは、Azure VM インスタンスを使用してデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-335">Other database engines such as [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/), and [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) can be deployed using Azure VM Instances.</span></span>

<span data-ttu-id="1bb97-336">AWS RDS のコストは、インスタンスが使用するハードウェア リソース (CPU、RAM、ストレージ、ネットワーク帯域幅など) の量によって決まります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-336">Costs for AWS RDS are determined by the amount of hardware resources that your instance uses, like CPU, RAM, storage, and network bandwidth.</span></span> <span data-ttu-id="1bb97-337">Azure データベース サービスでは、コストは、データベースのサイズ、同時接続数、およびスループット レベルによって決まります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-337">In the Azure database services, cost depends on your database size, concurrent connections, and throughput levels.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-338">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-338">See also</span></span>

-   [<span data-ttu-id="1bb97-339">Azure SQL Database のチュートリアル</span><span class="sxs-lookup"><span data-stu-id="1bb97-339">Azure SQL Database Tutorials</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [<span data-ttu-id="1bb97-340">Azure ポータルを使用して Azure SQL Database の geo レプリケーションを構成する</span><span class="sxs-lookup"><span data-stu-id="1bb97-340">Configure geo-replication for Azure SQL Database with the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [<span data-ttu-id="1bb97-341">Cosmos DB の概要: NoSQL JSON Database</span><span class="sxs-lookup"><span data-stu-id="1bb97-341">Introduction to Cosmos DB: A NoSQL JSON Database</span></span>](https://azure.microsoft.com/documentation/articles/documentdb-introduction/)

-   [<span data-ttu-id="1bb97-342">Node.js から Azure Table Storage を使用する方法</span><span class="sxs-lookup"><span data-stu-id="1bb97-342">How to use Azure Table storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a><span data-ttu-id="1bb97-343">セキュリティと ID</span><span class="sxs-lookup"><span data-stu-id="1bb97-343">Security and identity</span></span>

#### <a name="directory-service-and-azure-active-directory"></a><span data-ttu-id="1bb97-344">ディレクトリ サービスと Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="1bb97-344">Directory service and Azure Active Directory</span></span>

<span data-ttu-id="1bb97-345">Azure では、ディレクトリ サービスを次の製品に分割しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-345">Azure splits up directory services into the following offerings:</span></span>

-   <span data-ttu-id="1bb97-346">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - クラウドベースのディレクトリおよび ID 管理サービスです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-346">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - cloud based directory and identity management service.</span></span>

-   <span data-ttu-id="1bb97-347">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - パートナーによって管理される ID から会社のアプリケーションにアクセスできるようにします。</span><span class="sxs-lookup"><span data-stu-id="1bb97-347">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - enables access to your corporate applications from partner-managed identities.</span></span>

-   <span data-ttu-id="1bb97-348">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - コンシューマー向けアプリケーションに対するシングル サインオンおよびユーザー管理をサポートするサービスです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-348">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - service offering support for single sign-on and user management for consumer facing applications.</span></span>

-   <span data-ttu-id="1bb97-349">[Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - ドメイン コントローラー サービスをホストし、Active Directory と互換性のあるドメイン参加とユーザー管理機能を提供できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-349">[Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - hosted domain controller service, allowing Active Directory compatible domain join and user management functionality.</span></span>

#### <a name="web-application-firewall"></a><span data-ttu-id="1bb97-350">Web アプリケーション ファイアウォール</span><span class="sxs-lookup"><span data-stu-id="1bb97-350">Web application firewall</span></span>

<span data-ttu-id="1bb97-351">[Application Gateway Web アプリケーション ファイアウォール](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)に加え、[Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) などのサード パーティ製の [Web アプリケーション ファイアウォールも使用できます](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)。</span><span class="sxs-lookup"><span data-stu-id="1bb97-351">In addition to the [Application Gateway Web Application Firewall](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), you can also [use web application firewalls](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) from third-party vendors like [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-352">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-352">See also</span></span>

-   [<span data-ttu-id="1bb97-353">Microsoft Azure セキュリティの概要</span><span class="sxs-lookup"><span data-stu-id="1bb97-353">Getting started with Microsoft Azure security</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [<span data-ttu-id="1bb97-354">Azure の ID 管理とアクセス制御セキュリティのベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="1bb97-354">Azure Identity Management and access control security best practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a><span data-ttu-id="1bb97-355">アプリケーションとメッセージング サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-355">Application and messaging services</span></span>

#### <a name="simple-email-service"></a><span data-ttu-id="1bb97-356">Simple Email Service</span><span class="sxs-lookup"><span data-stu-id="1bb97-356">Simple Email Service</span></span>

<span data-ttu-id="1bb97-357">AWS では、通知、トランザクション、またはマーケティングに関する電子メールを送信するための Simple Email Service (SES) を提供しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-357">AWS provides the Simple Email Service (SES) for sending notification, transactional, or marketing emails.</span></span> <span data-ttu-id="1bb97-358">Azure では、[Sendgrid](https://sendgrid.com/partners/azure/) などのサード パーティ製のソリューションによって電子メール サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-358">In Azure, third-party solutions like [Sendgrid](https://sendgrid.com/partners/azure/) provide email services.</span></span>

#### <a name="simple-queueing-service"></a><span data-ttu-id="1bb97-359">Simple Queueing Service</span><span class="sxs-lookup"><span data-stu-id="1bb97-359">Simple Queueing Service</span></span>

<span data-ttu-id="1bb97-360">AWS Simple Queueing Service (SQS) は、AWS プラットフォーム内のアプリケーション、サービス、およびデバイスを接続するためのメッセージング システムを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-360">AWS Simple Queueing Service (SQS) provides a messaging system for connecting applications, services, and devices within the AWS platform.</span></span> <span data-ttu-id="1bb97-361">Azure には、同様の機能を提供する 2 つのサービスがあります。</span><span class="sxs-lookup"><span data-stu-id="1bb97-361">Azure has two services that provide similar functionality:</span></span>

-   <span data-ttu-id="1bb97-362">[Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Azure プラットフォーム内のアプリケーション コンポーネント間で通信できるようにするクラウド メッセージング サービスです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-362">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - a cloud messaging service that allows communication between application components within the Azure platform.</span></span>

-   <span data-ttu-id="1bb97-363">[Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - アプリケーション、サービス、およびデバイスを接続するためのより堅牢なメッセージング システムです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-363">[Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - a more robust messaging system for connecting applications, services, and devices.</span></span> <span data-ttu-id="1bb97-364">Service Bus は、関連する [Service Bus Relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it) を使用して、リモートでホストされているアプリケーションとサービスにも接続できます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-364">Using the related [Service Bus relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it), Service Bus can also connect to remotely hosted applications and services.</span></span>

#### <a name="device-farm"></a><span data-ttu-id="1bb97-365">Device Farm</span><span class="sxs-lookup"><span data-stu-id="1bb97-365">Device Farm</span></span>

<span data-ttu-id="1bb97-366">AWS Device Farm は、デバイス間のテスト サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-366">The AWS Device Farm provides cross-device testing services.</span></span> <span data-ttu-id="1bb97-367">Azure では、[Xamarin Test Cloud](https://www.xamarin.com/test-cloud) によって、モバイル デバイス用のデバイス間フロントエンド テストを提供しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-367">In Azure, [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) provides similar cross-device front-end testing for mobile devices.</span></span>

<span data-ttu-id="1bb97-368">フロントエンド テストだけでなく、[Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) によって、Linux 環境と Windows 環境向けのバックエンド テスト リソースも提供しています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-368">In addition to front-end testing, the [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) provides back end testing resources for Linux and Windows environments.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-369">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-369">See also</span></span>

-   [<span data-ttu-id="1bb97-370">Node.js から Queue ストレージを使用する方法</span><span class="sxs-lookup"><span data-stu-id="1bb97-370">How to use Queue storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [<span data-ttu-id="1bb97-371">Service Bus キューの使用方法</span><span class="sxs-lookup"><span data-stu-id="1bb97-371">How to use Service Bus queues</span></span>](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a><span data-ttu-id="1bb97-372">分析とビッグ データ</span><span class="sxs-lookup"><span data-stu-id="1bb97-372">Analytics and big data</span></span>

<span data-ttu-id="1bb97-373">[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) は、大量のデータを取得、整理、分析、および視覚化することを目的とする Azure の製品とサービスのパッケージです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-373">[The Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) is Azure's package of products and services designed to capture, organize, analyze, and visualize large amounts of data.</span></span> <span data-ttu-id="1bb97-374">Cortana スイートは、次のサービスで構成されています。</span><span class="sxs-lookup"><span data-stu-id="1bb97-374">The Cortana suite consists of the following services:</span></span>

-   <span data-ttu-id="1bb97-375">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Hadoop、Spark、Storm、または HBase を含む、管理された Apache ディストリビューションです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-375">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - managed Apache distribution that includes Hadoop, Spark, Storm, or HBase.</span></span>

-   <span data-ttu-id="1bb97-376">[Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - データ オーケストレーションおよびデータ パイプライン機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="1bb97-376">[Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - provides data orchestration and data pipeline functionality.</span></span>

-   <span data-ttu-id="1bb97-377">[SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 大規模リレーショナル データ ストレージです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-377">[SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - large-scale relational data storage.</span></span>

-   <span data-ttu-id="1bb97-378">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - ビッグ データ分析ワークロード用に最適化された大規模ストレージです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-378">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - large-scale storage optimized for big data analytics workloads.</span></span>

-   <span data-ttu-id="1bb97-379">[Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - データの予測分析を構築して適用するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-379">[Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - used to build and apply predictive analytics on data.</span></span>

-   <span data-ttu-id="1bb97-380">[Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - リアルタイムのデータ分析を行います。</span><span class="sxs-lookup"><span data-stu-id="1bb97-380">[Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - real-time data analysis.</span></span>

-   <span data-ttu-id="1bb97-381">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - Data Lake Store と連携するように最適化された大規模分析サービスです。</span><span class="sxs-lookup"><span data-stu-id="1bb97-381">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - large-scale analytics service optimized to work with Data Lake Store</span></span>

-   <span data-ttu-id="1bb97-382">[PowerBI](https://powerbi.microsoft.com/) - データの視覚化を促進するために使用されます。</span><span class="sxs-lookup"><span data-stu-id="1bb97-382">[PowerBI](https://powerbi.microsoft.com/) - used to power data visualization.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-383">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-383">See also</span></span>

-   [<span data-ttu-id="1bb97-384">Cortana Intelligence ギャラリー</span><span class="sxs-lookup"><span data-stu-id="1bb97-384">Cortana Intelligence Gallery</span></span>](https://gallery.cortanaintelligence.com/)

-   [<span data-ttu-id="1bb97-385">Microsoft ビッグ データ ソリューションを理解する</span><span class="sxs-lookup"><span data-stu-id="1bb97-385">Understanding Microsoft big data solutions</span></span>](https://msdn.microsoft.com/library/dn749804.aspx)

-   [<span data-ttu-id="1bb97-386">Azure Data Lake と Azure の HDInsight ブログ</span><span class="sxs-lookup"><span data-stu-id="1bb97-386">Azure Data Lake & Azure HDInsight Blog</span></span>](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a><span data-ttu-id="1bb97-387">モノのインターネット</span><span class="sxs-lookup"><span data-stu-id="1bb97-387">Internet of Things</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-388">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-388">See also</span></span>

-   [<span data-ttu-id="1bb97-389">Azure IoT Hub を使ってみる</span><span class="sxs-lookup"><span data-stu-id="1bb97-389">Get started with Azure IoT Hub</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [<span data-ttu-id="1bb97-390">IoT Hub と Event Hubs の比較</span><span class="sxs-lookup"><span data-stu-id="1bb97-390">Comparison of IoT Hub and Event Hubs</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a><span data-ttu-id="1bb97-391">モバイル サービス</span><span class="sxs-lookup"><span data-stu-id="1bb97-391">Mobile services</span></span>

#### <a name="notifications"></a><span data-ttu-id="1bb97-392">通知</span><span class="sxs-lookup"><span data-stu-id="1bb97-392">Notifications</span></span>

<span data-ttu-id="1bb97-393">Notification Hubs は SMS または電子メール メッセージの送信をサポートしていないため、これらの配信の種類では、サード パーティが提供するサービスが必要です。</span><span class="sxs-lookup"><span data-stu-id="1bb97-393">Notification Hubs do not support sending SMS or email messages, so third-party services are needed for those delivery types.</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-394">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-394">See also</span></span>

-   [<span data-ttu-id="1bb97-395">Android アプリの作成</span><span class="sxs-lookup"><span data-stu-id="1bb97-395">Create an Android app</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [<span data-ttu-id="1bb97-396">Azure Mobile Apps での認証および承認</span><span class="sxs-lookup"><span data-stu-id="1bb97-396">Authentication and Authorization in Azure Mobile Apps</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [<span data-ttu-id="1bb97-397">Azure Notification Hubs によるプッシュ通知の送信</span><span class="sxs-lookup"><span data-stu-id="1bb97-397">Sending push notifications with Azure Notification Hubs</span></span>](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a><span data-ttu-id="1bb97-398">管理と監視</span><span class="sxs-lookup"><span data-stu-id="1bb97-398">Management and monitoring</span></span>

#### <a name="see-also"></a><span data-ttu-id="1bb97-399">関連項目</span><span class="sxs-lookup"><span data-stu-id="1bb97-399">See also</span></span>
-   [<span data-ttu-id="1bb97-400">監視と診断のガイダンス</span><span class="sxs-lookup"><span data-stu-id="1bb97-400">Monitoring and diagnostics guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [<span data-ttu-id="1bb97-401">Azure Resource Manager テンプレートを作成するためのベスト プラクティス</span><span class="sxs-lookup"><span data-stu-id="1bb97-401">Best practices for creating Azure Resource Manager templates</span></span>](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [<span data-ttu-id="1bb97-402">Azure Resource Manager のクイックスタート テンプレート</span><span class="sxs-lookup"><span data-stu-id="1bb97-402">Azure Resource Manager Quickstart templates</span></span>](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a><span data-ttu-id="1bb97-403">次の手順</span><span class="sxs-lookup"><span data-stu-id="1bb97-403">Next steps</span></span>

-   [<span data-ttu-id="1bb97-404">AWS と Azure のサービスの完全比較マトリックス</span><span class="sxs-lookup"><span data-stu-id="1bb97-404">Complete AWS and Azure service comparison matrix</span></span>](https://aka.ms/azure4aws-services)

-   [<span data-ttu-id="1bb97-405">対話型 Azure プラットフォームの全体像</span><span class="sxs-lookup"><span data-stu-id="1bb97-405">Interactive Azure Platform Big Picture</span></span>](http://azureplatform.azurewebsites.net/)

-   [<span data-ttu-id="1bb97-406">Azure を使ってみる</span><span class="sxs-lookup"><span data-stu-id="1bb97-406">Get started with Azure</span></span>](https://azure.microsoft.com/get-started/)

-   [<span data-ttu-id="1bb97-407">Azure ソリューション アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="1bb97-407">Azure solution architectures</span></span>](https://azure.microsoft.com/solutions/architecture/)

-   [<span data-ttu-id="1bb97-408">Azure リファレンス アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="1bb97-408">Azure Reference Architectures</span></span>](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [<span data-ttu-id="1bb97-409">パターンとプラクティス: Azure のガイダンス</span><span class="sxs-lookup"><span data-stu-id="1bb97-409">Patterns & Practices: Azure Guidance</span></span>](https://azure.microsoft.com/documentation/articles/guidance/)

-   [<span data-ttu-id="1bb97-410">無料オンライン コース: AWS エキスパート向け Microsoft Azure</span><span class="sxs-lookup"><span data-stu-id="1bb97-410">Free Online Course: Microsoft Azure for AWS Experts</span></span>](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/