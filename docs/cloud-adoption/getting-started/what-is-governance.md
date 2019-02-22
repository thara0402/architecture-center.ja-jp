---
title: 'CAF: クラウド リソース ガバナンスとは'
description: Azure でのクラウド リソース ガバナンスの説明
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: ec8b0b04ac8a4782c215359cf907c3c092ae2f4d
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897951"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a><span data-ttu-id="d9f2e-103">クラウド リソース ガバナンスとは</span><span class="sxs-lookup"><span data-stu-id="d9f2e-103">What is cloud resource governance?</span></span>

<span data-ttu-id="d9f2e-104">[Azure のしくみ](what-is-azure.md)に関するページでは、Azure は、仮想化されたハードウェアとソフトウェアをユーザーの代わりに実行する、サーバーとネットワーク ハードウェアのコレクションであることを説明しました。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-104">In [how does Azure work?](what-is-azure.md), you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users.</span></span> <span data-ttu-id="d9f2e-105">Azure を使用すると、必要に応じてリソースを容易に作成、読み取り、更新、および削除できるようにすることで、組織の開発および IT 部門の俊敏性を高めることが可能です。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-105">Azure enables your organization's development and IT departments to be agile by making it easy to create, read, update, and delete resources as needed.</span></span>

<span data-ttu-id="d9f2e-106">ただし、開発者が無制限にリソースにアクセスできるようにすると、俊敏性は高まりますが、意図しないコストが発生する場合があります。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-106">However, while giving unrestricted resource access to developers can make them very agile, it can also lead to unintended cost consequences.</span></span> <span data-ttu-id="d9f2e-107">たとえば、テスト用に一連のリソースのデプロイを承認された開発チームが、テスト完了後に、そのリソースを削除し忘れることがあります。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-107">For example, a development team might be approved to deploy a set of resources for testing but forget to delete them when testing is complete.</span></span> <span data-ttu-id="d9f2e-108">これらのリソースのコストは、承認が無効になったり不要になったりしても発生し続けます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-108">These resources will continue to accrue costs even though their use is no longer approved or necessary.</span></span>

<span data-ttu-id="d9f2e-109">この問題を解決するのが、リソース アクセス **ガバナンス**です。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-109">The solution to this problem is resource access **governance**.</span></span> <span data-ttu-id="d9f2e-110">ガバナンスとは、組織の目標と要件を満たすために、Azure リソースの使用を継続的に管理、監視、および監査するプロセスを意味します。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-110">Governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span>

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<span data-ttu-id="d9f2e-111">これらの目標や要件は組織によって異なるため、1 つのアプローチですべてのガバナンス対応するのは不可能です。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-111">These goals and requirements are unique to each organization so it's not possible to have a one-size-fits-all approach to governance.</span></span> <span data-ttu-id="d9f2e-112">正確には、Azure では、主要ガバナンス ツールとして**ロール ベースのアクセス制御 (RBAC)** と**リソース ポリシー**の 2 つが実装され、これらのツールを使用してガバナンス モデルを設計することは、それぞれの組織の責任です。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-112">Rather, Azure implements two primary governance tools, **role based access control (RBAC)**, and **resource policy**, and it's up to each organization to design their governance model using them.</span></span>

<span data-ttu-id="d9f2e-113">RBAC ではロールが定義され、そのロールによって、ロールに割り当てられているユーザーの機能が定義されます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-113">RBAC defines roles, and roles define the capabilities for a user that is assigned the role.</span></span> <span data-ttu-id="d9f2e-114">たとえば、**所有者**ロールでは、リソースのすべての機能 (作成、読み取り、更新、および削除) が有効になりますが、**閲覧者**ロールでは、読み取り機能しか有効になりません。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-114">For example, the **owner** role enables all capabilites (create, read, update, and delete) for a resource, while the  **reader** roles enables only the read capability.</span></span> <span data-ttu-id="d9f2e-115">広範なスコープでロールを定義して、多くの種類のリソースに適用することも、スコープを狭くして適用対象を少なくすることもできます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-115">Roles can be defined with a broad scope that applies to many resources types, or a narrow scope that applies to a few.</span></span>

<span data-ttu-id="d9f2e-116">リソース ポリシーでは、リソース作成のルールが定義されます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-116">Resource policies define rules for resource creation.</span></span> <span data-ttu-id="d9f2e-117">たとえば、リソース ポリシーによって、VM の SKU を、事前に承認された特定のサイズに制限できます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-117">For example, a resource policy can limit the SKU of a VM to a particular pre-appproved size.</span></span> <span data-ttu-id="d9f2e-118">または、リソースの作成が要求されたときに、リソース ポリシーによって、タグとコスト センターの追加を強制できます。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-118">Or, a resource policy can enforce the addition of a tag with a cost center when the request is made to create the resource.</span></span>

<span data-ttu-id="d9f2e-119">これらのツールを構成するときは、ガバナンスと組織の機敏性のバランスを考慮することが重要です。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-119">When configuring these tools, an important consideration is balancing governance versus organizational agility.</span></span> <span data-ttu-id="d9f2e-120">つまり、ガバナンス ポリシーの制限が厳しいほど、開発者や IT 作業者の俊敏性が低下します。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-120">That is, the more restrictive your governance policy, the less agile your developers and IT workers become.</span></span> <span data-ttu-id="d9f2e-121">これは、ガバナンス ポリシーの制限が厳しいと、開発者がフォームを入力するか、ガバナンス チームの担当者にメールして、リソースを手動で作成しなければならない、といった手動による操作が必要になる場合があるためです。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-121">This is because a restrictive goverance policy may require more manual steps, such as requiring a developer to fill out a form or send an email to a person on the governance team to manually create a resource.</span></span> <span data-ttu-id="d9f2e-122">ガバナンス チームの能力には限りがあり、バックログが発生する可能性があります。これにより、リソースが作成されるのを待っている開発チームの生産性が低下します。また、リソースが削除されるのを待っている場合は、その間に不要なリソース コストが発生します。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-122">The goverance team has finite capabilities and may become backlogged, resulting in unproductive development teams waiting for their resources to be created and unneeded resources accruing costs while they wait to be deleted.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d9f2e-123">次の手順</span><span class="sxs-lookup"><span data-stu-id="d9f2e-123">Next steps</span></span>

<span data-ttu-id="d9f2e-124">クラウド リソース ガバナンスの概念を理解したので、Azure でリソース アクセスを管理する方法について確認します。</span><span class="sxs-lookup"><span data-stu-id="d9f2e-124">Now that you understand the concept of cloud resource goverance, learn more about how resource access is managed in Azure.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d9f2e-125">Azure でのリソース アクセスについて確認する</span><span class="sxs-lookup"><span data-stu-id="d9f2e-125">Learn about resource access in Azure</span></span>](azure-resource-access.md)
