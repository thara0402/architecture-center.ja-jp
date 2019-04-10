---
title: CAF:リソースの整合性のサンプル ポリシー ステートメント
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: リソースの整合性のサンプル ポリシー ステートメント
author: BrianBlanchard
ms.openlocfilehash: 1336d349825b2b87d072f70b99d94dca57249bda
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241963"
---
# <a name="resource-consistency-sample-policy-statements"></a><span data-ttu-id="a5502-103">リソースの整合性のサンプル ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="a5502-103">Resource Consistency sample policy statements</span></span>

<span data-ttu-id="a5502-104">個々のクラウド ポリシー ステートメントは、リスクの評価プロセスで識別された特定のリスクに対処するためのガイドラインとなります。</span><span class="sxs-lookup"><span data-stu-id="a5502-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="a5502-105">これらのステートメントでは、リスクとそれらのリスクに対処する計画の簡潔な概要を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="a5502-106">各ステートメント定義には、以下の情報を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="a5502-107">技術的なリスク - このポリシーで対処されるリスクの概要。</span><span class="sxs-lookup"><span data-stu-id="a5502-107">Technical risk - A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="a5502-108">ポリシー ステートメント - ポリシー要件の端的な概要説明。</span><span class="sxs-lookup"><span data-stu-id="a5502-108">Policy statement - A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="a5502-109">設計オプション - 実行可能な推奨事項、仕様、または IT チームおよび開発者がポリシーの実装時に使用できるその他のガイダンス。</span><span class="sxs-lookup"><span data-stu-id="a5502-109">Design options - Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="a5502-110">次のサンプル ポリシー ステートメントは、リソースの整合性に関連する複数の一般的なビジネス リスクに対処し、組織のニーズに対応するポリシー ステートメントのドラフトを作成するときに参照する例として提供されます。</span><span class="sxs-lookup"><span data-stu-id="a5502-110">The following sample policy statements address a number of common Resource Consistency-related business risks, and are provided as examples for you to reference when drafting policy statements to address your own organization's needs.</span></span> <span data-ttu-id="a5502-111">これらのサンプルは、規制的であることを意図しておらず、特定のリスクに対処するために複数のポリシー オプションが存在する可能性があることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="a5502-111">Note that these examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any particular risk.</span></span> <span data-ttu-id="a5502-112">ビジネス チームおよび IT チームと密接に連携して、リスクの一意のセットに最適なポリシー ソリューションを識別します。</span><span class="sxs-lookup"><span data-stu-id="a5502-112">Work closely with business and IT teams to identify the best policy solutions for your unique set of risks.</span></span>

## <a name="tagging"></a><span data-ttu-id="a5502-113">タグ付け</span><span class="sxs-lookup"><span data-stu-id="a5502-113">Tagging</span></span>

<span data-ttu-id="a5502-114">**技術的なリスク**:適切なメタデータのタグ付けが、デプロイされたリソースに関連付けられていない場合、IT 運用チームは、必要な SLA、ビジネス操作に対する重要性、または運用コストに基づいて、リソースのサポートまたは最適化の優先順位を決定できません。</span><span class="sxs-lookup"><span data-stu-id="a5502-114">**Technical risk**: Without proper metadata tagging associated with deployed resources, IT Operations cannot prioritize support or optimization of resources based on required SLA, importance to business operations, or operational cost.</span></span> <span data-ttu-id="a5502-115">これにより、IT リソースが不適切に割り当てられて、インシデントの解決が遅れる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-115">This can result in mis-allocation of IT resources and potential delays in incident resolution.</span></span>

<span data-ttu-id="a5502-116">**ポリシー ステートメント**:次のポリシーが実装されます:</span><span class="sxs-lookup"><span data-stu-id="a5502-116">**Policy statement**: The following policies will be implemented:</span></span>

- <span data-ttu-id="a5502-117">デプロイされた資産には、コスト、重要度、SLA、および環境の値で、タグを付ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-117">Deployed assets should be tagged with the following values: cost, criticality, SLA, and environment.</span></span>
- <span data-ttu-id="a5502-118">ガバナンス ツールでは、コスト、重要度、SLA、アプリケーション、環境に関連するタグ付けが検証される必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-118">Governance tooling must validate tagging related to cost, criticality, SLA, application, and environment.</span></span> <span data-ttu-id="a5502-119">すべての値を、ガバナンス チームが管理する定義済みの値に合わせる必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-119">All values must align to predefined values managed by the governance team.</span></span>

<span data-ttu-id="a5502-120">**使用可能な設計オプション**:Azure では、[標準的な名前と値のメタデータ タグ](/azure/azure-resource-manager/resource-group-using-tags)が、ほとんどのリソースの種類でサポートされています。</span><span class="sxs-lookup"><span data-stu-id="a5502-120">**Potential design options**: In Azure, [standard name-value metadata tags](/azure/azure-resource-manager/resource-group-using-tags) are supported on most resource types.</span></span> <span data-ttu-id="a5502-121">リソース作成の一部として特定のタグを適用するには、[Azure Policy](/azure/governance/policy/overview) を使用します。</span><span class="sxs-lookup"><span data-stu-id="a5502-121">[Azure Policy](/azure/governance/policy/overview) is used to enforce specific tags as part of resource creation.</span></span>

## <a name="ungoverned-subscriptions"></a><span data-ttu-id="a5502-122">管理されていないサブスクリプション</span><span class="sxs-lookup"><span data-stu-id="a5502-122">Ungoverned subscriptions</span></span>

<span data-ttu-id="a5502-123">**技術的なリスク**:サブスクリプションと管理グループを勝手に作成すると、ガバナンス ポリシーによって適切に対象とされない、クラウド資産の切り離されたセクションが発生します。</span><span class="sxs-lookup"><span data-stu-id="a5502-123">**Technical risk**: Arbitrary creation of subscriptions and management groups can lead to isolated sections of your cloud estate that are not properly subject to your governance policies.</span></span>

<span data-ttu-id="a5502-124">**ポリシー ステートメント**:ミッション クリティカルなアプリケーションまたは保護対象データ向けに新しいサブスクリプションまたは管理グループを作成する場合、クラウド ガバナンス チームによるレビューを受ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-124">**Policy statement**: Creation of new subscriptions or management groups for any mission-critical applications or protected data will require a review from the Cloud Governance team.</span></span> <span data-ttu-id="a5502-125">承認された変更は、適切なブループリントの割り当てに統合されます。</span><span class="sxs-lookup"><span data-stu-id="a5502-125">Approved changes will be integrated into a proper blueprint assignment.</span></span>

<span data-ttu-id="a5502-126">**使用可能な設計オプション**:組織の [Azure 管理グループ](/azure/governance/management-groups/)への管理アクセスを、サブスクリプションの作成とアクセス制御プロセスを制御する承認されたガバナンス チームのメンバーのみにロックダウンします。</span><span class="sxs-lookup"><span data-stu-id="a5502-126">**Potential design options**: Lock down administrative access to your organizations [Azure management groups](/azure/governance/management-groups/) to only approved governance team members who will control the subscription creation and access control process.</span></span>

## <a name="manage-updates-to-virtual-machines"></a><span data-ttu-id="a5502-127">仮想マシンに対する更新プログラムを管理する</span><span class="sxs-lookup"><span data-stu-id="a5502-127">Manage updates to virtual machines</span></span>

<span data-ttu-id="a5502-128">**技術的なリスク**:最新の更新プログラムとソフトウェアの修正プログラムによって最新の状態になっていない仮想マシン (VM) は、セキュリティやパフォーマンスの問題に対して脆弱であり、サービスが中断される可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-128">**Technical risk**: Virtual machines (VMs) that are not up-to-date with the latest updates and software patches are vulnerable to security or performance issues, which can result in service disruptions.</span></span>

<span data-ttu-id="a5502-129">**ポリシー ステートメント**:ガバナンス ツールでは、すべてのデプロイ済み VM に対して自動更新が有効になるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-129">**Policy statement**: Governance tooling must enforce that automatic updates are enabled on all deployed VMs.</span></span> <span data-ttu-id="a5502-130">違反は運用管理チームがレビューし、運用ポリシーに従って修復する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-130">Violations must be reviewed with operational management teams and remediated in accordance with operations policies.</span></span> <span data-ttu-id="a5502-131">自動的に更新されない資産は、IT 運用チームが担当するプロセスに含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-131">Assets that are not automatically updated must be included in processes owned by IT Operations.</span></span>

<span data-ttu-id="a5502-132">**使用可能な設計オプション**:Azure でホストされた VM では、[Azure Automation の Update Management ソリューション](/azure/automation/automation-update-management)を使用して、一貫した更新プログラム管理を提供できます。</span><span class="sxs-lookup"><span data-stu-id="a5502-132">**Potential design options**: For Azure hosted VMs, you can provide consistent update management using the [Update Management solution in Azure Automation](/azure/automation/automation-update-management).</span></span>

## <a name="deployment-compliance"></a><span data-ttu-id="a5502-133">デプロイのコンプライアンス</span><span class="sxs-lookup"><span data-stu-id="a5502-133">Deployment compliance</span></span>

<span data-ttu-id="a5502-134">**技術的なリスク**:クラウド ガバナンス チームによって十分に調査されていないデプロイ スクリプトと自動化ツールを使用すると、リソースのデプロイがポリシーに違反する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-134">**Technical risk**: Deployment scripts and automation tooling that is not fully vetted by the Cloud Governance team can result in resource deployments that violate policy.</span></span>

<span data-ttu-id="a5502-135">**ポリシー ステートメント**:次のポリシーが実装されます:</span><span class="sxs-lookup"><span data-stu-id="a5502-135">**Policy statement**: The following policies will be implemented:</span></span>

- <span data-ttu-id="a5502-136">デプロイされた資産の継続的なガバナンスを保証するために、デプロイ ツールはクラウド ガバナンス チームの承認を受ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-136">Deployment tooling must be approved by the Cloud Governance team to ensure ongoing governance of deployed assets.</span></span>
- <span data-ttu-id="a5502-137">デプロイ スクリプトのメンテナンスは、クラウド ガバナンス チームが定期的なレビューと監査のためにアクセスできる中央リポジトリで行われる必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-137">Deployment scripts must be maintained in central repository accessible by the Cloud Governance team for periodic review and auditing.</span></span>

<span data-ttu-id="a5502-138">**使用可能な設計オプション**:[Azure Blueprints](/azure/governance/blueprints/) を一貫して使用して自動化されたデプロイを管理すると、組織のガバナンス標準とポリシーに準拠している Azure リソースを一貫してデプロイできます。</span><span class="sxs-lookup"><span data-stu-id="a5502-138">**Potential design options**: Consistent use of [Azure Blueprints](/azure/governance/blueprints/) to manage automated deployments allows consistent deployments of Azure resources that adhere to your organization's governance standards and policies.</span></span>

## <a name="monitoring"></a><span data-ttu-id="a5502-139">監視</span><span class="sxs-lookup"><span data-stu-id="a5502-139">Monitoring</span></span>

<span data-ttu-id="a5502-140">**技術的なリスク**:監視の実装が不適切な場合、または監視のインストルメント化が一貫していない場合は、ワークロードの正常性の問題またはその他のポリシー コンプライアンス違反を検出できない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-140">**Technical risk**: Improperly implemented or inconsistently instrumented monitoring can prevent the detection of workload health issues or other policy compliance violations.</span></span>

<span data-ttu-id="a5502-141">**ポリシー ステートメント**:次のポリシーが実装されます:</span><span class="sxs-lookup"><span data-stu-id="a5502-141">**Policy statement**: The following policies will be implemented:</span></span>

- <span data-ttu-id="a5502-142">ミッション クリティカルなアプリケーションや保護データに関連するすべての資産がリソース減耗と最適化のための監視対象となっていることを、ガバナンス ツールが検証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-142">Governance tooling must validate that all assets related to mission-critical applications or protected data are included in monitoring for resource depletion and optimization.</span></span>
- <span data-ttu-id="a5502-143">ミッション クリティカルなアプリケーションや保護データのすべてについて適切なレベルのログ データが収集されていることを、ガバナンス ツールが検証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-143">Governance tooling must validate that the appropriate level of logging data is being collected for all mission-critical applications or protected data.</span></span>

<span data-ttu-id="a5502-144">**使用可能な設計オプション**:[Azure Monitor](/azure/azure-monitor/overview) は Azure プラットフォームでの既定の監視サービスであり、リソースをデプロイするときに [Azure Blueprints](/azure/governance/blueprints/) を使用することで一貫した監視を適用できます。</span><span class="sxs-lookup"><span data-stu-id="a5502-144">**Potential design options**: [Azure Monitor](/azure/azure-monitor/overview) is the default monitoring service in the Azure platform, and consistent monitoring can be enforced through the use of [Azure Blueprints](/azure/governance/blueprints/) when deploying resources.</span></span>

## <a name="disaster-recovery"></a><span data-ttu-id="a5502-145">障害復旧</span><span class="sxs-lookup"><span data-stu-id="a5502-145">Disaster recovery</span></span>

<span data-ttu-id="a5502-146">**技術的なリスク**:リソースの障害、削除、または破損により、ミッション クリティカルなアプリケーションまたはサービスが中断し、機密データが失われる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-146">**Technical risk**: Resource failure, deletions, or corruption can result in disruption of mission-critical applications or services and the loss of sensitive data.</span></span>

<span data-ttu-id="a5502-147">**ポリシー ステートメント**:すべてのミッション クリティカルなアプリケーションおよび保護されたデータには、障害やシステム エラーのビジネスへの影響を最小限に抑えるため、バックアップと復旧のソリューションを実装する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a5502-147">**Policy statement**: All mission-critical applications and protected data must have backup and recovery solutions implemented to minimize business impact of outages or system failures.</span></span>

<span data-ttu-id="a5502-148">**使用可能な設計オプション**:Azure Site Recovery サービスでは、ビジネス継続性とディザスター リカバリー (BCDR) のシナリオでの停止時間を最小限に抑えることを目的とする、バックアップ、復旧、およびレプリケーションの機能が提供されています。</span><span class="sxs-lookup"><span data-stu-id="a5502-148">**Potential design options**: The [Azure Site Recovery] service provides backup, recovery, and replication capabilities intended to minimize outage duration in business continuity and disaster recovery (BCDR) scenarios.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a5502-149">次の手順</span><span class="sxs-lookup"><span data-stu-id="a5502-149">Next steps</span></span>

<span data-ttu-id="a5502-150">手始めに、この記事で説明されているサンプルを使用して、クラウドの導入計画に合致する特定のビジネス リスクに対処するポリシーを作成します。</span><span class="sxs-lookup"><span data-stu-id="a5502-150">Use the samples mentioned in this article as a starting point to develop policies that address specific business risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="a5502-151">リソースの整合性に関連する独自のカスタム ポリシー ステートメントの開発を始めるには、[リソースの整合性テンプレート](template.md)をダウンロードしてください。</span><span class="sxs-lookup"><span data-stu-id="a5502-151">To begin developing your own custom policy statements related to Resource Consistency, download the [Resource Consistency template](template.md).</span></span>

<span data-ttu-id="a5502-152">この規範の導入を促進するには、ご使用の環境に最も合う[アクションにつながるガバナンス体験](../journeys/overview.md)を選択します。</span><span class="sxs-lookup"><span data-stu-id="a5502-152">To accelerate adoption of this discipline, choose the [actionable governance journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="a5502-153">その後、設計を変更して、特定の企業ポリシーの決定を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="a5502-153">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="a5502-154">アクションにつながるガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="a5502-154">Actionable Governance Journeys</span></span>](../journeys/overview.md)
