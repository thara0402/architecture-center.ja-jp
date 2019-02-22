---
title: CAF:セキュリティ ベースラインのサンプル ポリシー ステートメント
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: セキュリティ ベースラインのサンプル ポリシー ステートメント
author: BrianBlanchard
ms.openlocfilehash: ac40e022f8dff0c3021c04cd9d6ae42d28afaf1f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901382"
---
# <a name="security-baseline-sample-policy-statements"></a><span data-ttu-id="fa560-103">セキュリティ ベースラインのサンプル ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="fa560-103">Security Baseline sample policy statements</span></span>

<span data-ttu-id="fa560-104">個々のクラウド ポリシー ステートメントは、リスクの評価プロセスで識別された特定のリスクに対処するためのガイドラインとなります。</span><span class="sxs-lookup"><span data-stu-id="fa560-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="fa560-105">これらのステートメントでは、リスクとそれらのリスクに対処する計画の簡潔な概要を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="fa560-106">各ステートメント定義には、以下の情報を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="fa560-107">技術的なリスク - このポリシーで対処されるリスクの概要。</span><span class="sxs-lookup"><span data-stu-id="fa560-107">Technical risk - A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="fa560-108">ポリシー ステートメント - ポリシー要件の端的な概要説明。</span><span class="sxs-lookup"><span data-stu-id="fa560-108">Policy statement - A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="fa560-109">技術的オプション - 実行可能な推奨事項、仕様、または IT チームおよび開発者がポリシーの実装時に使用できるその他のガイダンス。</span><span class="sxs-lookup"><span data-stu-id="fa560-109">Technical options - Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="fa560-110">次のサンプル ポリシー ステートメントでは、セキュリティに関連する複数の一般的なビジネス リスクに対処し、組織のニーズに対応する実際のポリシー ステートメントのドラフトを作成するときに参照する例として提供されます。</span><span class="sxs-lookup"><span data-stu-id="fa560-110">The following sample policy statements address a number of common security-related business risks, and are provided as examples for you to reference when drafting actual policy statements addressing your own organization's needs.</span></span> <span data-ttu-id="fa560-111">これらの例は、規制的であることを意図しておらず、単一の特定されたリスクに対処するために複数のポリシー オプションが存在する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-111">These examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any single identified risk.</span></span> <span data-ttu-id="fa560-112">ビジネス チーム、セキュリティ チーム、IT チームと密接に連携して、特定のセキュリティ リスクに最適なポリシー ソリューションを識別します。</span><span class="sxs-lookup"><span data-stu-id="fa560-112">Work closely with business, security, and IT teams to identify the best policy solutions for your particular security risks.</span></span>  

## <a name="asset-classification"></a><span data-ttu-id="fa560-113">資産の分類</span><span class="sxs-lookup"><span data-stu-id="fa560-113">Asset classification</span></span>

<span data-ttu-id="fa560-114">**技術的なリスク**:資産がミッション クリティカルとして、または機密データを含むものとして正しく識別されていない場合、十分な保護を受けられず、データのリークやビジネスの中断につながる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-114">**Technical risk**: Assets that are not correctly identified as mission-critical or involving sensitive data may not receive sufficient protections, leading to potential data leaks or business disruptions.</span></span>

<span data-ttu-id="fa560-115">**ポリシー ステートメント**:デプロイされたすべての資産を、重大性とデータ分類によってカテゴライズする必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-115">**Policy statement**: All deployed assets must be categorized by criticality and data classification.</span></span> <span data-ttu-id="fa560-116">クラウド ガバナンス チームとアプリケーション所有者は、クラウドにデプロイする前に、分類をレビューする必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-116">Classifications must be reviewed by the Cloud Governance team and the application owner before deployment to the cloud.</span></span>

<span data-ttu-id="fa560-117">**使用可能な設計オプション**:[リソースタグ付け標準](../../decision-guides/resource-tagging/overview.md)を確立し、IT スタッフによってそれがデプロイされるすべてのリソースに [Azure リソース タグ](/azure/azure-resource-manager/resource-group-using-tags)を使用して一貫して適用されるようにします。</span><span class="sxs-lookup"><span data-stu-id="fa560-117">**Potential design option**: Establish [resource tagging standards](../../decision-guides/resource-tagging/overview.md) and ensure IT staff apply them consistently to any deployed resources using [Azure resource tags](/azure/azure-resource-manager/resource-group-using-tags).</span></span>

## <a name="data-encryption"></a><span data-ttu-id="fa560-118">データの暗号化</span><span class="sxs-lookup"><span data-stu-id="fa560-118">Data encryption</span></span>

<span data-ttu-id="fa560-119">**技術的なリスク**:保管中に、保護されたデータが流出するリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="fa560-119">**Technical risk**: There is a risk of protected data being exposed during storage.</span></span>

<span data-ttu-id="fa560-120">**ポリシー ステートメント**:すべての保護対象データは暗号化した状態で保存される必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-120">**Policy statement**: All protected data must be encrypted when at rest.</span></span>

<span data-ttu-id="fa560-121">**使用可能な設計オプション**:Azure プラットフォームで保存データの暗号化を行う方法については、「[Azure の暗号化の概要](/azure/security/security-azure-encryption-overview)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="fa560-121">**Potential design option**: See the [Azure encryption overview](/azure/security/security-azure-encryption-overview) article for a discussion of how data at rest encryption is performed on the Azure platform.</span></span>  

## <a name="network-isolation"></a><span data-ttu-id="fa560-122">ネットワークの分離</span><span class="sxs-lookup"><span data-stu-id="fa560-122">Network isolation</span></span>

<span data-ttu-id="fa560-123">**技術的なリスク**:ネットワークとネットワーク内のサブネット間の接続における潜在的な脆弱性により、データのリークや、ミッション クリティカルなサービスの中断が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-123">**Technical risk**: Connectivity between networks and subnets within networks introduces potential vulnerabilities that can result in data leaks or disruption of mission-critical services.</span></span>

<span data-ttu-id="fa560-124">**ポリシー ステートメント**:保護対象データが含まれているネットワーク サブネットは、他のサブネットから分離する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-124">**Policy statement**: Network subnets containing protected data must be isolated from any other subnets.</span></span> <span data-ttu-id="fa560-125">保護対象データ サブネット間のネットワーク トラフィックを定期的に監査します。</span><span class="sxs-lookup"><span data-stu-id="fa560-125">Network traffic between protected data subnets is to be audited regularly.</span></span>

<span data-ttu-id="fa560-126">**使用可能な設計オプション**:Azure では、ネットワークとサブネットの分離は [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) によって管理されます。</span><span class="sxs-lookup"><span data-stu-id="fa560-126">**Potential design option**: In Azure, network and subnet isolation is managed through [Azure Virtual Networks](/azure/virtual-network/virtual-networks-overview).</span></span>

## <a name="secure-external-access"></a><span data-ttu-id="fa560-127">セキュリティで保護された外部アクセス</span><span class="sxs-lookup"><span data-stu-id="fa560-127">Secure external access</span></span>

<span data-ttu-id="fa560-128">**技術的なリスク**:パブリック インターネットからワークロードへのアクセスを許可すると、承認されていないデータの漏洩やビジネスの中断の原因になる侵入のリスクが発生します。</span><span class="sxs-lookup"><span data-stu-id="fa560-128">**Technical risk**: Allowing access to workloads from the public internet introduces a risk of intrusion resulting in unauthorized data exposure or business disruption.</span></span>

<span data-ttu-id="fa560-129">**ポリシー ステートメント**:保護対象データが含まれているどのサブネットにも、パブリック インターネット経由で、またはデータセンターをまたいで直接アクセスすることはできません。</span><span class="sxs-lookup"><span data-stu-id="fa560-129">**Policy statement**: No subnet containing protected data can be directly accessed over public internet or across datacenters.</span></span> <span data-ttu-id="fa560-130">それらのサブネットへのアクセスは、中間サブネットワークを介してルーティングされる必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-130">Access to those subnets must be routed through intermediate subnet works.</span></span> <span data-ttu-id="fa560-131">それらのサブネットへのアクセスはすべて、パケットのスキャンおよびブロック機能を実行できるファイアウォール ソリューション経由で行われる必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-131">All access into those subnets must come through a firewall solution capable of performing packet scanning and blocking functions.</span></span>

<span data-ttu-id="fa560-132">**使用可能な設計オプション**:Azure では、[パブリック インターネットとクラウドベースのネットワークの間に DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz) をデプロイすることで、パブリック エンドポイントをセキュリティ保護します。</span><span class="sxs-lookup"><span data-stu-id="fa560-132">**Potential design option**: In Azure, secure public endpoints by deploying a [DMZ between the public internet and your cloud-based network](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).</span></span>

## <a name="ddos-protection"></a><span data-ttu-id="fa560-133">DDoS 保護</span><span class="sxs-lookup"><span data-stu-id="fa560-133">DDoS protection</span></span>

<span data-ttu-id="fa560-134">**技術的なリスク**:分散型サービス拒否 (DDoS) 攻撃により、業務が中断する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-134">**Technical risk**: Distributed denial of service (DDoS) attacks can result in a business interruption.</span></span>

<span data-ttu-id="fa560-135">**ポリシー ステートメント**:パブリックにアクセス可能なすべてのネットワーク エンドポイントに、自動化された DDoS リスク軽減メカニズムをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="fa560-135">**Policy statement**: Deploy automated DDoS mitigation mechanisms to all publicly accessible network endpoints.</span></span>

<span data-ttu-id="fa560-136">**使用可能な設計オプション**:[Azure DDoS Protection](/azure/virtual-network/ddos-protection-overview) を使用して、DDoS 攻撃による中断を最小限に抑えます。</span><span class="sxs-lookup"><span data-stu-id="fa560-136">**Potential design option**: Use [Azure DDoS Protection](/azure/virtual-network/ddos-protection-overview) to minimize disruptions caused by DDoS attacks.</span></span>

## <a name="secure-on-premises-connectivity"></a><span data-ttu-id="fa560-137">オンプレミスの接続をセキュリティ保護する</span><span class="sxs-lookup"><span data-stu-id="fa560-137">Secure on-premises connectivity</span></span>

<span data-ttu-id="fa560-138">**技術的なリスク**:パブリック インターネット経由でのクラウド ネットワークとオンプレミスの間の暗号化されていないトラフィックは、傍受に対して脆弱であり、データ漏洩のリスクが生まれます。</span><span class="sxs-lookup"><span data-stu-id="fa560-138">**Technical risk**: Unencrypted traffic between your cloud network and on-premises over the public internet is vulnerable to interception, introducing the risk of data exposure.</span></span>

<span data-ttu-id="fa560-139">**ポリシー ステートメント**:オンプレミスとクラウド ネットワークの間のすべての接続は、セキュリティで保護されて暗号化された VPN 接続または専用のプライベート WAN リンクを通して行う必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-139">**Policy statement**: All connections between the on-premises and cloud networks must take place either through a secure encrypted VPN connection or a dedicated private WAN link.</span></span>

<span data-ttu-id="fa560-140">**使用可能な設計オプション**:Azure では、ExpressRoute または Azure VPN を使用して、オンプレミスとクラウド ネットワークの間のプライベート接続を確立します。</span><span class="sxs-lookup"><span data-stu-id="fa560-140">**Potential design option**: In Azure, use ExpressRoute or Azure VPN to establish private connections between your on-premises and cloud networks.</span></span>

## <a name="network-monitoring-and-enforcement"></a><span data-ttu-id="fa560-141">ネットワークの監視と適用</span><span class="sxs-lookup"><span data-stu-id="fa560-141">Network monitoring and enforcement</span></span>

<span data-ttu-id="fa560-142">**技術的なリスク**:ネットワークの構成を変更すると、新たな脆弱性やデータ漏洩のリスクが発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-142">**Technical risk**: Changes to network configuration can lead to new vulnerabilities and data exposure risks.</span></span>

<span data-ttu-id="fa560-143">**ポリシー ステートメント**:ガバナンス ツールでは、セキュリティ ベースライン チームによって定義されたネットワーク構成要件を監査し、実施する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-143">**Policy statement**: Governance tooling must audit and enforce network configuration requirements defined by the Security Baseline team.</span></span>

<span data-ttu-id="fa560-144">**使用可能な設計オプション**:Azure では、[Azure Network Watcher](/azure/network-watcher/network-watcher-monitoring-overview) を使用してネットワーク アクティビティを監視でき、[Azure Security Center](/azure/security-center/security-center-network-recommendations) はセキュリティの脆弱性を識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="fa560-144">**Potential design option**: In Azure, network activity can be monitored using [Azure Network Watcher](/azure/network-watcher/network-watcher-monitoring-overview), and [Azure Security Center](/azure/security-center/security-center-network-recommendations) can help identify security vulnerabilities.</span></span> <span data-ttu-id="fa560-145">Azure Policy を使用すると、セキュリティ チームによって定義された制限に従って、ネットワーク リソースとリソース構成ポリシーを制限することができます。</span><span class="sxs-lookup"><span data-stu-id="fa560-145">Azure Policy allows you to restrict network resources and resource configuration policy according to limits defined by the security team.</span></span>

## <a name="security-review"></a><span data-ttu-id="fa560-146">セキュリティ レビュー</span><span class="sxs-lookup"><span data-stu-id="fa560-146">Security review</span></span>

<span data-ttu-id="fa560-147">**技術的なリスク**:時間の経過と共に、新しいセキュリティ脅威や攻撃の種類が発生し、クラウド リソースが流出または中断するリスクが高くなります。</span><span class="sxs-lookup"><span data-stu-id="fa560-147">**Technical risk**: Over time, new security threats and attack types emerge, increasing the risk of exposure or disruption of your cloud resources.</span></span>

<span data-ttu-id="fa560-148">**ポリシー ステートメント**:クラウドのデプロイに影響を及ぼす可能性があるトレンドおよび潜在的悪用をセキュリティ チームが定期的にレビューし、クラウドで使用されるセキュリティ ベースライン ツールの更新を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fa560-148">**Policy statement**: Trends and potential exploits that could affect cloud deployments should be reviewed regularly by the security team to provide updates to Security Baseline tooling used in the cloud.</span></span>

<span data-ttu-id="fa560-149">**使用可能な設計オプション**:関連する IT およびガバナンス チームのメンバーが参加するセキュリティ レビュー会議を定期的に行います。</span><span class="sxs-lookup"><span data-stu-id="fa560-149">**Potential design option**: Establish a regular security review meeting that includes relevant IT and governance team members.</span></span> <span data-ttu-id="fa560-150">既存のセキュリティ データとメトリックのレビューを行って、現在のポリシーとセキュリティ ベースライン ツールのギャップを確かめ、新しいリスクを軽減するようにポリシーを更新します。</span><span class="sxs-lookup"><span data-stu-id="fa560-150">Review existing security data and metrics to establish gaps in current policy and Security Baseline tooling, and update policy to mitigate any new risks.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fa560-151">次の手順</span><span class="sxs-lookup"><span data-stu-id="fa560-151">Next steps</span></span>

<span data-ttu-id="fa560-152">手始めに、この記事で説明されているサンプルを使用して、クラウドの導入計画に合致する特定のセキュリティ リスクに対処するポリシーを作成します。</span><span class="sxs-lookup"><span data-stu-id="fa560-152">Use the samples mentioned in this article as a starting point to develop policies that address specific security risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="fa560-153">セキュリティ ベースラインに関連する独自のカスタム ポリシー ステートメントを作成するには、[セキュリティ ベースライン テンプレート](template.md)をダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="fa560-153">To begin developing your own custom policy statements related to Security Baseline, download the [Security Baseline template](template.md).</span></span>

<span data-ttu-id="fa560-154">この規範の導入を促進するには、ご使用の環境に最も合う[アクションにつながるガバナンス体験](../journeys/overview.md)を選択します。</span><span class="sxs-lookup"><span data-stu-id="fa560-154">To accelerate adoption of this discipline, choose the [Actionable Governance Journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="fa560-155">その後、設計を変更して、特定の企業ポリシーの決定を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="fa560-155">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="fa560-156">アクションにつながるガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="fa560-156">Actionable Governance Journeys</span></span>](../journeys/overview.md)
