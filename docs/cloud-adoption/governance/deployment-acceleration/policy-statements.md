---
title: CAF:デプロイ高速化のサンプル ポリシー ステートメント
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: デプロイ高速化のサンプル ポリシー ステートメント
author: alexbuckgit
ms.openlocfilehash: 4f7d59e6653c29db03f966d1c7105524b72586fc
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901825"
---
# <a name="deployment-acceleration-sample-policy-statements"></a><span data-ttu-id="61b6b-103">デプロイ高速化のサンプル ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="61b6b-103">Deployment Acceleration sample policy statements</span></span>

<span data-ttu-id="61b6b-104">個々のクラウド ポリシー ステートメントは、リスクの評価プロセスで識別された特定のリスクに対処するためのガイドラインとなります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="61b6b-105">これらのステートメントでは、リスクとそれらのリスクに対処する計画の簡潔な概要を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="61b6b-106">各ステートメントの定義には、以下の情報を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="61b6b-107">**技術的なリスク。**</span><span class="sxs-lookup"><span data-stu-id="61b6b-107">**Technical risk.**</span></span> <span data-ttu-id="61b6b-108">このポリシーが対処するリスクの概要です。</span><span class="sxs-lookup"><span data-stu-id="61b6b-108">A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="61b6b-109">**ポリシー ステートメント。**</span><span class="sxs-lookup"><span data-stu-id="61b6b-109">**Policy statement.**</span></span> <span data-ttu-id="61b6b-110">ポリシーの要件の端的な概要説明です。</span><span class="sxs-lookup"><span data-stu-id="61b6b-110">A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="61b6b-111">**設計オプション。**</span><span class="sxs-lookup"><span data-stu-id="61b6b-111">**Design options.**</span></span> <span data-ttu-id="61b6b-112">実践的な推奨事項、仕様、または IT チームおよび開発者がポリシーの実装時に使用できるその他のガイダンス。</span><span class="sxs-lookup"><span data-stu-id="61b6b-112">Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="61b6b-113">次のサンプル ポリシー ステートメントは、構成に関連する複数の一般的なビジネス リスクに対処し、組織のニーズに対応するポリシー ステートメントのドラフトを作成するときに参照する例として提供されます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-113">The following sample policy statements address a number of common configuration-related business risks, and are provided as examples for you to reference when drafting policy statements to address your own organization's needs.</span></span> <span data-ttu-id="61b6b-114">これらのサンプルは、規制的であることを意図しておらず、特定のリスクに対処するために複数のポリシー オプションが存在する可能性があることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="61b6b-114">Note that these examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any particular risk.</span></span> <span data-ttu-id="61b6b-115">ビジネス チームおよび IT チームと密接に連携して、リスクの一意のセットに最適なポリシー ソリューションを識別します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-115">Work closely with business and IT teams to identify the best policy solutions for your unique set of risks.</span></span>

## <a name="reliance-on-manual-deployment-or-configuration-of-systems"></a><span data-ttu-id="61b6b-116">システムの手動デプロイまたは構成への依存</span><span class="sxs-lookup"><span data-stu-id="61b6b-116">Reliance on manual deployment or configuration of systems</span></span>

<span data-ttu-id="61b6b-117">**技術的なリスク**:デプロイまたは構成時に人による介入に頼ると、、ヒューマン エラーが発生する可能性が増え、システムのデプロイと構成の再現性と予測可能性が低下します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-117">**Technical risk**: Relying on human intervention during deployment or configuration increases the likelihood of human error and reduces the repeatability and predictability of system deployments and configuration.</span></span> <span data-ttu-id="61b6b-118">また、 一般にシステム リソースのデプロイが遅くなります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-118">It also typically leads to slower deployment of system resources.</span></span>

<span data-ttu-id="61b6b-119">**ポリシー ステートメント**:クラウドにデプロイされるすべての資産は、可能な限りテンプレートまたは自動化スクリプトを使用してデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-119">**Policy statement**: All assets deployed to the cloud should be deployed using templates or automation scripts whenever possible.</span></span>

<span data-ttu-id="61b6b-120">**使用可能な設計オプション**:[Azure Resource Manager テンプレート](/azure/azure-resource-manager/resource-group-overview#template-deployment)で、リソースを Azure にデプロイするための、コードとしてのインフラストラクチャ手法が提供されます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-120">**Potential design options**: [Azure Resource Manager templates](/azure/azure-resource-manager/resource-group-overview#template-deployment) provides an infrastructure-as-code approach to deploying your resources to Azure.</span></span> <span data-ttu-id="61b6b-121">[Azure の構成要素](https://github.com/mspnp/template-building-blocks/wiki)で、コマンドライン ツールと、Azure リソースのデプロイの簡略化を意図した Resource Manager テンプレートのセットが提供されます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-121">The [Azure Building Blocks](https://github.com/mspnp/template-building-blocks/wiki) provide a command-line tool and set of Resource Manager templates designed to simplify deployment of Azure resources.</span></span>

## <a name="lack-of-visibility-into-system-issues"></a><span data-ttu-id="61b6b-122">システムの問題の可視化の欠如</span><span class="sxs-lookup"><span data-stu-id="61b6b-122">Lack of visibility into system issues</span></span>

<span data-ttu-id="61b6b-123">**技術的なリスク**:ビジネス システムの監視と診断が不十分であると、システム障害が発生する前に運用担当者が問題を識別して修復することができず、障害を適切に解決するために必要な時間が大幅に増大する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-123">**Technical risk**: Insufficient monitoring and diagnostics for business systems prevent operations personnel from identifying and remediating issues before a system outage occurs, and can significantly increase the time needed to properly resolve an outage.</span></span>

<span data-ttu-id="61b6b-124">**ポリシー ステートメント**:次のポリシーが実装されます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-124">**Policy statement**: The following policies will be implemented:</span></span>

- <span data-ttu-id="61b6b-125">主要なメトリックと診断方法がすべての実稼働システムとコンポーネントに対して識別されます。また、監視ツールと診断ツールがそれらのシステムに適用され、運用担当者によって定期的に監視されます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-125">Key metrics and diagnostics measures will be identified for all production systems and components, and monitoring and diagnostic tools will be applied to these systems and monitored regularly by operations personnel.</span></span>
- <span data-ttu-id="61b6b-126">運用では、ステージングや QA などの非運用環境で監視ツールと診断ツールを使用して、運用環境でシステムの問題が発生する前にそれらの問題を識別することを検討します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-126">Operations will consider using monitoring and diagnostic tools in non-production environments such as Staging and QA to identify system issues before they occur in the production environment.</span></span>

<span data-ttu-id="61b6b-127">**使用可能な設計オプション**:[Azure Monitor](/azure/azure-monitor/) には Log Analytics と Application Insights も含まれており、テレメトリを収集して分析するためのツールが提供されます。このツールは、アプリケーションの実行状況を把握し、それらのアプリケーションに影響を与える問題およびそれらのアプリケーションが依存するリソースを事前に識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-127">**Potential design options**: [Azure Monitor](/azure/azure-monitor/), which also includes Log Analytics and Application Insights, provides tools for collecting and analyzing telemetry to help you understand how your applications are performing and proactively identify issues affecting them and the resources they depend on.</span></span>

## <a name="configuration-security-reviews"></a><span data-ttu-id="61b6b-128">構成セキュリティ のレビュー</span><span class="sxs-lookup"><span data-stu-id="61b6b-128">Configuration security reviews</span></span>

<span data-ttu-id="61b6b-129">**技術的なリスク**:時間が経つにつれて、新しいセキュリティ上の脅威または懸念により、セキュリティで保護されたリソースへの不正アクセスのリスクが高まる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-129">**Technical risk**: Over time, new security threats or concerns can increase the risks of unauthorized access to secure resources.</span></span>

<span data-ttu-id="61b6b-130">**ポリシー ステートメント**:クラウド ガバナンスのプロセスには、クラウド資産の構成によって阻止する必要がある悪意のあるアクターや使用パターンを識別するために、構成管理チームとの四半期ごとのレビューを含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="61b6b-130">**Policy statement**: Cloud Governance processes must include quarterly review with configuration management teams to identify malicious actors or usage patterns that should be prevented by cloud asset configuration.</span></span>

<span data-ttu-id="61b6b-131">**使用可能な設計オプション**:ガバナンスのチーム メンバーと、クラウド アプリケーションとリソースの構成を担当する IT スタッフの両方が参加する、四半期ごとのセキュリティ レビュー会議を設定します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-131">**Potential design options**: Establish a quarterly security review meeting that includes both governance team members and IT staff responsible for configuration cloud applications and resources.</span></span> <span data-ttu-id="61b6b-132">既存のセキュリティ データとメトリックのレビューを行って、現在のデプロイ高速化のポリシーとツールのギャップを明らかにし、新しいリスクを軽減するようにポリシーを更新します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-132">Review existing security data and metrics to establish gaps in current Deployment Acceleration policy and tooling, and update policy to mitigate any new risks.</span></span>

## <a name="next-steps"></a><span data-ttu-id="61b6b-133">次の手順</span><span class="sxs-lookup"><span data-stu-id="61b6b-133">Next steps</span></span>

<span data-ttu-id="61b6b-134">この記事で説明されているサンプルを開始点として使用し、クラウドの導入計画に合致する特定のビジネス上のリスクに対処するポリシーを作成します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-134">Use the samples mentioned in this article as a starting point to develop policies that address specific business risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="61b6b-135">ID 管理に関連する独自のカスタム ポリシー ステートメントを作成するには、[ID ベースライン テンプレート](template.md)をダウンロードします。</span><span class="sxs-lookup"><span data-stu-id="61b6b-135">To begin developing your own custom policy statements related to identity management, download the [Identity Baseline template](template.md).</span></span>

<span data-ttu-id="61b6b-136">この規範の導入を促進するには、ご使用の環境に最も合致する[アクションにつながるガバナンス体験](../journeys/overview.md)を選択します。</span><span class="sxs-lookup"><span data-stu-id="61b6b-136">To accelerate adoption of this discipline, choose the [Actionable Governance Journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="61b6b-137">次に、設計を変更して、特定の企業ポリシーの決定を組み込みます。</span><span class="sxs-lookup"><span data-stu-id="61b6b-137">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="61b6b-138">アクションにつながるガバナンス体験</span><span class="sxs-lookup"><span data-stu-id="61b6b-138">Actionable Governance Journeys</span></span>](../journeys/overview.md)