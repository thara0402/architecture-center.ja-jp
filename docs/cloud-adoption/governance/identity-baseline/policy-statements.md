---
title: CAF:ID ベースラインのサンプル ポリシー ステートメント
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: ID ベースラインのサンプル ポリシー ステートメント
author: BrianBlanchard
ms.openlocfilehash: 5fad9265b9c048ee502c7e084ddd03faa0ad3e23
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902033"
---
# <a name="identity-baseline-sample-policy-statements"></a><span data-ttu-id="57356-103">ID ベースラインのサンプル ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="57356-103">Identity Baseline sample policy statements</span></span>

<span data-ttu-id="57356-104">個々のクラウド ポリシー ステートメントは、リスクの評価プロセス時に識別された特定のリスクに対処するためのガイドラインとなります。</span><span class="sxs-lookup"><span data-stu-id="57356-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="57356-105">これらのステートメントでは、リスクとそれらのリスクに対処する計画の簡潔な概要を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="57356-106">各ステートメント定義には、以下の情報を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="57356-107">技術的なリスク - このポリシーで対処されるリスクの概要。</span><span class="sxs-lookup"><span data-stu-id="57356-107">Technical risk - A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="57356-108">ポリシー ステートメント - ポリシー要件の端的な概要説明。</span><span class="sxs-lookup"><span data-stu-id="57356-108">Policy statement - A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="57356-109">設計オプション - 実践的な推奨事項、仕様、または IT チームおよび開発者がポリシーの実装時に使用できるその他のガイダンス。</span><span class="sxs-lookup"><span data-stu-id="57356-109">Design options - Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="57356-110">次のサンプル ポリシー ステートメントは、ID に関連する複数の一般的なビジネス リスクに対処し、組織のニーズに対応するポリシー ステートメントのドラフトを作成するときに参照する例として提供されます。</span><span class="sxs-lookup"><span data-stu-id="57356-110">The following sample policy statements address a number of common identity-related business risks, and are provided as examples for you to reference when drafting policy statements to address your own organization's needs.</span></span> <span data-ttu-id="57356-111">これらのサンプルは、規制的であることを意図しておらず、特定のリスクに対処するために複数のポリシー オプションが存在する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="57356-111">These examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any particular risk.</span></span> <span data-ttu-id="57356-112">ビジネス チームおよび IT チームと密接に連携して、リスクの一意のセットに最適なポリシー ソリューションを識別します。</span><span class="sxs-lookup"><span data-stu-id="57356-112">Work closely with business and IT teams to identify the best policy solutions for your unique set of risks.</span></span>

## <a name="lack-of-access-controls"></a><span data-ttu-id="57356-113">アクセス制御の欠如</span><span class="sxs-lookup"><span data-stu-id="57356-113">Lack of access controls</span></span>

<span data-ttu-id="57356-114">**技術的なリスク**:不十分な、またはアドホックのアクセス制御設定により、機密リソースまたはミッション クリティカルなリソースへの未承認アクセスのリスクが生じる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="57356-114">**Technical risk**: Insufficient or ad-hoc access control settings can introduce risk of unauthorized access to sensitive or mission-critical resources.</span></span>

<span data-ttu-id="57356-115">**ポリシー ステートメント**:クラウドにデプロイされるすべての資産は、現在のガバナンス ポリシーによって承認された ID とロールを使用して制御する必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-115">**Policy statement**: All assets deployed to the cloud should be controlled using identities and roles approved by current governance policies.</span></span>

<span data-ttu-id="57356-116">**使用可能な設計オプション**:[Azure Active Directory の条件付きアクセス](/azure/active-directory/conditional-access/overview)は、Azure の既定のアクセス制御メカニズムです。</span><span class="sxs-lookup"><span data-stu-id="57356-116">**Potential design options**: [Azure Active Directory conditional access](/azure/active-directory/conditional-access/overview) is the default access control mechanism in Azure.</span></span>

## <a name="overprovisioned-access"></a><span data-ttu-id="57356-117">オーバープロビジョニングのアクセス</span><span class="sxs-lookup"><span data-stu-id="57356-117">Overprovisioned access</span></span>

<span data-ttu-id="57356-118">**技術的なリスク**:ユーザーおよびグループがその責任範囲を超えてリソースを制御した結果、無許可の変更が行われ、障害またはセキュリティの脆弱性につながる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="57356-118">**Technical risk**: Users and groups with control over resources beyond their area of responsibility can result in unauthorized modifications leading to outages or security vulnerabilities.</span></span>

<span data-ttu-id="57356-119">**ポリシー ステートメント**:次のポリシーが実装されます:</span><span class="sxs-lookup"><span data-stu-id="57356-119">**Policy statement**: The following policies will be implemented:</span></span>

- <span data-ttu-id="57356-120">ミッション クリティカルなアプリケーションまたは保護対象データに関わるあらゆるリソースに、最小特権アクセス モデルが適用されます。</span><span class="sxs-lookup"><span data-stu-id="57356-120">A least privilege access model will be applied to any resources involved in mission-critical applications or protected data.</span></span>
- <span data-ttu-id="57356-121">アクセス許可の昇格は例外とするべきであり、クラウド ガバナンス チームはそのような例外をすべて記録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-121">Elevated permissions should be an exception, and any such exceptions must be recorded with the Cloud Governance team.</span></span> <span data-ttu-id="57356-122">例外は定期的に監査されます。</span><span class="sxs-lookup"><span data-stu-id="57356-122">Exceptions will be audited regularly.</span></span>

<span data-ttu-id="57356-123">**使用可能な設計オプション**:[知る必要性](https://wikipedia.org/wiki/Need_to_know)と[最小特権セキュリティ](https://wikipedia.org/wiki/Principle_of_least_privilege)の各原則に基づいてアクセスを制限するロールベースのアクセス制御 (RBAC) 戦略を、[Azure Identity Management のベスト プラクティス](/azure/security/azure-security-identity-management-best-practices)を参考にして実装します。</span><span class="sxs-lookup"><span data-stu-id="57356-123">**Potential design options**: Consult the [Azure Identity Management best practices](/azure/security/azure-security-identity-management-best-practices) to implement a role-based access control (RBAC) strategy that restricts access based on the [need to know](https://wikipedia.org/wiki/Need_to_know) and [least privilege security](https://wikipedia.org/wiki/Principle_of_least_privilege) principles.</span></span>

## <a name="lack-of-shared-management-accounts-between-on-premises-and-the-cloud"></a><span data-ttu-id="57356-124">オンプレミスとクラウドの間で共有される管理アカウントの欠如</span><span class="sxs-lookup"><span data-stu-id="57356-124">Lack of shared management accounts between on-premises and the cloud</span></span>

<span data-ttu-id="57356-125">**技術的なリスク**:オンプレミスの Active Directory にアカウントを持つ IT 管理スタッフが、クラウド リソースへの十分なアクセス権限を持たないことにより、運用またはセキュリティの問題を効率的に解決できない場合があります。</span><span class="sxs-lookup"><span data-stu-id="57356-125">**Technical risk**: IT management or administrative staff with accounts on your on-premises Active Directory may not have sufficient access to cloud resources may not be able to efficiently resolve operational or security issues.</span></span>

<span data-ttu-id="57356-126">**ポリシー ステートメント**:オンプレミスの Active Directory インフラストラクチャ内にあり、昇格された特権を持つすべてのグループを、承認済みの RBAC ロールにマップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-126">**Policy statement**: All groups in the on-premises Active Directory infrastructure that have elevated privileges should be mapped to an approved RBAC role.</span></span>

<span data-ttu-id="57356-127">**使用可能な設計オプション**:クラウド ベースの Azure Active Directory とオンプレミスの Active Directory の間でハイブリッド ID ソリューションを実装し、必要なオンプレミス グループを、各自の作業を行うために必要な RBAC ロールに追加します。</span><span class="sxs-lookup"><span data-stu-id="57356-127">**Potential design options**: Implement a hybrid identity solution between your cloud-based Azure Active Directory and your on-premise Active Directory, and add the required on-premises groups to the RBAC roles necessary to do their work.</span></span>

## <a name="weak-authentication-mechanisms"></a><span data-ttu-id="57356-128">弱い認証メカニズム</span><span class="sxs-lookup"><span data-stu-id="57356-128">Weak authentication mechanisms</span></span>

<span data-ttu-id="57356-129">**技術的なリスク**:基本的なユーザー/パスワードの組み合わせなど、安全性が不十分なユーザー認証方法を使用した ID 管理システムは、パスワードの侵害またはハッキングにつながる可能性があり、セキュリティで保護されたクラウド システムへの未承認アクセスの大きなリスクをもたらします。</span><span class="sxs-lookup"><span data-stu-id="57356-129">**Technical risk**: Identity management systems with insufficiently secure user authentication methods, such as basic user/password combinations, can lead to compromised or hacked passwords, providing a major risk of unauthorized access to secure cloud systems.</span></span>

<span data-ttu-id="57356-130">**ポリシー ステートメント**:すべてのアカウントは、セキュリティで保護されたリソースにログインする際、多要素認証 (MFA) 方式を使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-130">**Policy statement**: All accounts are required to login to secured resources using a multi-factor authentication (MFA) method.</span></span>

<span data-ttu-id="57356-131">**使用可能な設計オプション**:Azure Active Directory の場合、ユーザー承認プロセスの一部として [Azure Multi-Factor Authentication](/azure/active-directory/authentication/concept-mfa-howitworks) を実装します。</span><span class="sxs-lookup"><span data-stu-id="57356-131">**Potential design options**: For Azure Active Directory, implement [Azure Multi-Factor Authentication](/azure/active-directory/authentication/concept-mfa-howitworks) as part of your user authorization process.</span></span>

## <a name="isolated-identity-providers"></a><span data-ttu-id="57356-132">孤立した ID プロバイダー</span><span class="sxs-lookup"><span data-stu-id="57356-132">Isolated identity providers</span></span>

<span data-ttu-id="57356-133">**技術的なリスク**:互換性のない ID プロバイダーにより、顧客またはその他のビジネス パートナーとリソースまたはサービスを共有できなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="57356-133">**Technical risk**: Incompatible identity providers can result in the inability to share resources or services with customers or other business partners.</span></span>

<span data-ttu-id="57356-134">**ポリシー ステートメント**:カスタマー認証が必要なアプリケーションをデプロイする場合、内部ユーザー用のプライマリ ID プロバイダーと互換性のある承認済み ID プロバイダーを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-134">**Policy statement**: Deployment of any applications that require customer authentication must use an approved identity provider that is compatible with the primary identity provider for internal users.</span></span>

<span data-ttu-id="57356-135">**使用可能な設計オプション**:社内の ID プロバイダーと顧客の ID プロバイダーの間に [Azure Active Directory とのフェデレーション](/azure/active-directory/hybrid/whatis-fed)を実装します。</span><span class="sxs-lookup"><span data-stu-id="57356-135">**Potential design options**: Implement [Federation with Azure Active Directory](/azure/active-directory/hybrid/whatis-fed) between your internal and customer identity providers.</span></span>

## <a name="identity-reviews"></a><span data-ttu-id="57356-136">ID レビュー</span><span class="sxs-lookup"><span data-stu-id="57356-136">Identity reviews</span></span>

<span data-ttu-id="57356-137">**技術的なリスク**:時間の経過と共に、ビジネスの変化、新しいクラウド デプロイの追加、またはその他のセキュリティの懸念事項により、セキュリティで保護されたリソースへの未承認アクセスのリスクが高まる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="57356-137">**Technical risk**: Over time business change, the addition of new cloud deployments or other security concerns can increase the risks of unauthorized access to secure resources.</span></span>

<span data-ttu-id="57356-138">**ポリシー ステートメント**:クラウド ガバナンスのプロセスには、クラウド資産の構成によって阻止する必要がある悪意のあるアクターや使用パターンを識別するために、ID 管理チームによる四半期ごとのレビューを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="57356-138">**Policy statement**: Cloud Governance processes must include quarterly review with identity management teams to identify malicious actors or usage patterns that should be prevented by cloud asset configuration.</span></span>

<span data-ttu-id="57356-139">**使用可能な設計オプション**:ガバナンスのチーム メンバーと、ID サービスの管理を担当する IT スタッフの両方を含む四半期ごとのセキュリティ レビュー会議を設定します。</span><span class="sxs-lookup"><span data-stu-id="57356-139">**Potential design options**: Establish a quarterly security review meeting that includes both governance team members and IT staff responsible for managing identity services.</span></span> <span data-ttu-id="57356-140">既存のセキュリティ データとメトリックのレビューを行って、現在の ID 管理のポリシーとツールのギャップを確かめ、新しいリスクを軽減するようにポリシーを更新します。</span><span class="sxs-lookup"><span data-stu-id="57356-140">Review existing security data and metrics to establish gaps in current identity management policy and tooling, and update policy to mitigate any new risks.</span></span>

## <a name="next-steps"></a><span data-ttu-id="57356-141">次の手順</span><span class="sxs-lookup"><span data-stu-id="57356-141">Next steps</span></span>

<span data-ttu-id="57356-142">手始めに、この記事で説明されているサンプルを使用して、クラウドの導入計画に合致する特定のビジネス リスクに対処するポリシーを作成します。</span><span class="sxs-lookup"><span data-stu-id="57356-142">Use the samples mentioned in this article as a starting point to develop policies that address specific business risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="57356-143">ID ベースラインに関連する独自のカスタム ポリシー ステートメントを作成するには、[ID ベースライン テンプレート](template.md)をダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="57356-143">To begin developing your own custom policy statements related to Identity Baseline, download the [Identity Baseline template](template.md).</span></span>

<span data-ttu-id="57356-144">この規範の導入を促進するには、ご使用の環境に最も合う[アクションにつながるガバナンス体験](../journeys/overview.md)を選択します。</span><span class="sxs-lookup"><span data-stu-id="57356-144">To accelerate adoption of this discipline, choose the [Actionable Governance Journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="57356-145">その後、設計を変更して、特定の企業ポリシーの決定を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="57356-145">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="57356-146">アクションにつながるガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="57356-146">Actionable Governance Journeys</span></span>](../journeys/overview.md)