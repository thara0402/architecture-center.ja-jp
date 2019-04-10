---
title: 'CAF: 中小企業 – コスト管理の進化'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 中小企業 – コスト管理の進化
author: BrianBlanchard
ms.openlocfilehash: cded37b69d538016501d39e7c09e7095a74a96be
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245183"
---
# <a name="small-to-medium-enterprise-cost-management-evolution"></a><span data-ttu-id="97a1e-103">中小企業: コスト管理の進化</span><span class="sxs-lookup"><span data-stu-id="97a1e-103">Small-to-medium enterprise: Cost Management evolution</span></span>

<span data-ttu-id="97a1e-104">この記事では、ガバナンス MVP にコスト制御を追加することで物語を展開します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-104">This article evolves the narrative by adding cost controls to the governance MVP.</span></span>

## <a name="evolution-of-the-narrative"></a><span data-ttu-id="97a1e-105">物語の展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-105">Evolution of the narrative</span></span>

<span data-ttu-id="97a1e-106">導入は、ガバナンス MVP で定義されているコストの許容範囲指標を以上に拡大しました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-106">Adoption has grown beyond the cost tolerance indicator defined in the governance MVP.</span></span> <span data-ttu-id="97a1e-107">これは "ディザスター リカバリー" 用データ センターからの移行に対応しているため、良いことです。</span><span class="sxs-lookup"><span data-stu-id="97a1e-107">This is a good thing, as it corresponds with migrations from the "DR" datacenter.</span></span> <span data-ttu-id="97a1e-108">今回は支出の増加が、クラウド ガバナンス チームが時間をかける理由となっています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-108">The increase in spending now justifies an investment of time from the Cloud Governance team.</span></span>

### <a name="evolution-of-the-current-state"></a><span data-ttu-id="97a1e-109">現在の状態の展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-109">Evolution of the current state</span></span>

<span data-ttu-id="97a1e-110">この物語の前のフェーズでは、IT 部門がディザスター リカバリー用データ センターの 100% を廃止しました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-110">In the previous phase of this narrative, IT had retired 100% of the DR datacenter.</span></span> <span data-ttu-id="97a1e-111">アプリケーション開発チームと BI チームは、運用トラフィックに対する準備を終えていました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-111">The application development and BI teams were ready for production traffic.</span></span>

<span data-ttu-id="97a1e-112">その後、ガバナンスに影響を与える以下のような変更がありました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-112">Since then, some things have changed that will affect governance:</span></span>

- <span data-ttu-id="97a1e-113">移行チームが、運用環境のデータ センターから VM の移行を開始しました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-113">The migration team has begun migrating VMs out of the production datacenter.</span></span>
- <span data-ttu-id="97a1e-114">アプリケーション開発チームが CI/CD パイプライン経由で運用環境のアプリケーションをクラウドに積極的にプッシュしてます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-114">The application development teams is actively pushing production applications to the cloud through CI/CD pipelines.</span></span> <span data-ttu-id="97a1e-115">それらのアプリケーションは、ユーザーの要求によって事後対的にスケールできます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-115">Those applications can reactively scale with user demands.</span></span>
- <span data-ttu-id="97a1e-116">IT 部門内のビジネス インテリジェンス チームは、クラウドに多数の予測分析ツールを提供してきました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-116">The business intelligence team within IT has delivered a number of predictive analytics tools in the cloud.</span></span> <span data-ttu-id="97a1e-117">クラウドで集計されるデータの量は増加し続けています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-117">the volumes of data aggregated in the cloud continues to grow.</span></span>
- <span data-ttu-id="97a1e-118">こうした成長のすべてが、約束されたビジネス上の成果を支えています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-118">All of this growth supports committed business outcomes.</span></span> <span data-ttu-id="97a1e-119">ただし、コストが急増し始めました。</span><span class="sxs-lookup"><span data-stu-id="97a1e-119">However, costs have begun to mushroom.</span></span> <span data-ttu-id="97a1e-120">推定経費は予想より早く増加しています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-120">Projected budgets are growing faster than expected.</span></span> <span data-ttu-id="97a1e-121">CFO は、コストを管理するための改善されたアプローチを必要としています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-121">The CFO needs improved approaches to managing costs.</span></span>

### <a name="evolution-of-the-future-state"></a><span data-ttu-id="97a1e-122">将来の状態の展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-122">Evolution of the future state</span></span>

<span data-ttu-id="97a1e-123">コストの監視およびレポートが、クラウド ソリューションに追加されます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-123">Cost monitoring and reporting is to be added to the cloud solution.</span></span> <span data-ttu-id="97a1e-124">現在も IT 部門がコスト情報センターとしての役割を果たしています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-124">IT is still serving as a cost clearing house.</span></span> <span data-ttu-id="97a1e-125">これは、クラウド サービスの支払いが、引き続き IT 調達に分類されることを意味します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-125">This means that payment for cloud services continues to come from IT procurement.</span></span> <span data-ttu-id="97a1e-126">しかしレポートでは、クラウド コストを費やしている機能に直接運用経費を結び付ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-126">However, reporting should tie direct operational expenses to the functions that are consuming the cloud costs.</span></span> <span data-ttu-id="97a1e-127">このモデルは、クラウド会計に対する "真相暴露" モデルと呼ばれています。</span><span class="sxs-lookup"><span data-stu-id="97a1e-127">This model is referred to as "Show Back" model to cloud accounting.</span></span>

<span data-ttu-id="97a1e-128">現在や将来の状態が変わると、新しいポリシー ステートメントを必要とする新しいリスクにさらされることになります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-128">The changes to current and future state expose new risks that will require new policy statements.</span></span>

## <a name="evolution-of-tangible-risks"></a><span data-ttu-id="97a1e-129">具体的なリスクの展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-129">Evolution of tangible risks</span></span>

<span data-ttu-id="97a1e-130">**予算管理**: セルフ サービス機能には、新しいプラットフォームでは過度の予期しないコストが生じるリスクがつきものです。</span><span class="sxs-lookup"><span data-stu-id="97a1e-130">**Budget control**: There is an inherent risk that self-service capabilities will result in excessive and unexpected costs on the new platform.</span></span> <span data-ttu-id="97a1e-131">コストを監視し、継続的なコスト リスクを軽減するためのガバナンス プロセスを実施し、計画した予算との継続的な整合を確保する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-131">Governance processes for monitoring costs and mitigating ongoing cost risks must be in place to ensure continued alignment with the planned budget.</span></span>

<span data-ttu-id="97a1e-132">このビジネス リスクは、いくつかの技術的リスクへと拡大する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-132">This business risk can be expanded into a few technical risks:</span></span>

- <span data-ttu-id="97a1e-133">実際のコストが計画を超過する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-133">Actual costs might exceed the plan.</span></span>
- <span data-ttu-id="97a1e-134">ビジネス条件は変化します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-134">Business conditions change.</span></span> <span data-ttu-id="97a1e-135">そうしたときには、ビジネス機能が予想より多くのクラウド サービスを消費する必要がある場合が存在し、それが異常な支出につながります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-135">When they do, there will be cases when a business function needs to consume more cloud services than expected, leading to spending anomalies.</span></span> <span data-ttu-id="97a1e-136">こうした余分な支出は、計画に必要な調整ではなく、古くて役に立たないものと見なされるリスクがあります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-136">There is a risk that this extra spending will be considered overages, as opposed to a necessary adjustment to the plan.</span></span>
- <span data-ttu-id="97a1e-137">システムがオーバープロビジョニングされて、余分なコストの支出となる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-137">Systems could be overprovisioned, resulting in excess spending.</span></span>

## <a name="evolution-of-the-policy-statements"></a><span data-ttu-id="97a1e-138">ポリシー ステートメントの展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-138">Evolution of the policy statements</span></span>

<span data-ttu-id="97a1e-139">ポリシーに対する以下の変更は、新しいリスクを軽減して実施を導く助けとなります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-139">The following changes to policy will help mitigate the new risks and guide implementation.</span></span>

1. <span data-ttu-id="97a1e-140">すべてのクラウド コストは、クラウド ガバナンス チームが毎週、計画を基準にして監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-140">All cloud costs should be monitored against plan on a weekly basis by the governance team.</span></span> <span data-ttu-id="97a1e-141">クラウド コストと計画の間の差異に関するレポートは、毎月、IT リーダーと財務部門で共有する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-141">Reporting on deviations between cloud costs and plan is to be shared with IT leadership and finance monthly.</span></span> <span data-ttu-id="97a1e-142">クラウド コストと計画のすべての更新は、毎月、IT リーダーと財務部門で確認する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-142">All cloud costs and plan updates should be reviewed with IT leadership and finance monthly.</span></span>
2. <span data-ttu-id="97a1e-143">すべてのコストは、責任の所在を明らかにするため、特定の職務に割り当てる必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-143">All costs must be allocated to a business function for accountability purposes.</span></span>
3. <span data-ttu-id="97a1e-144">クラウド資産は、最適化の機会を探して継続的に監視する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-144">Cloud assets should be continually monitored for optimization opportunities.</span></span>
4. <span data-ttu-id="97a1e-145">クラウド ガバナンス ツールでは、資産のサイズ変更オプションを、承認済みの構成リストに限定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-145">Cloud Governance tooling must limit Asset sizing options to an approved list of configurations.</span></span> <span data-ttu-id="97a1e-146">このツールにより、すべての資産がコスト監視ソリューションによって発見可能で、追跡されるようにする必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-146">The tooling must ensure that all assets are discoverable and tracked by the cost monitoring solution.</span></span>
5. <span data-ttu-id="97a1e-147">デプロイの計画時には、運用ワークロードのホスティングに関連付けられている必須のクラウド リソースがあれば、それを文書化する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-147">During deployment planning, any required cloud resources associated with the hosting of production workloads should be documented.</span></span> <span data-ttu-id="97a1e-148">このドキュメントは、予算の精度を高め、より高価なオプションの使用を防ぐ追加の自動化を準備するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-148">This documentation will help refine budgets and prepare additional automation to prevent the use of more expensive options.</span></span> <span data-ttu-id="97a1e-149">このプロセス中には、予約インスタンスやライセンス コストの削減など、クラウド プロバイダーが提供するさまざまな割引ツールについて検討する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-149">During this process consideration should be given to different discounting tools offered by the cloud provider, such as reserved instances or license cost reductions.</span></span>
6. <span data-ttu-id="97a1e-150">すべてのアプリケーション所有者は、クラウド コストをより適切に管理するために、ワークロードの最適化実習のトレーニングに参加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-150">All application owners are required to attend trained on practices for optimizing workloads to better control cloud costs.</span></span>

## <a name="evolution-of-the-best-practices"></a><span data-ttu-id="97a1e-151">ベスト プラクティスの展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-151">Evolution of the best practices</span></span>

<span data-ttu-id="97a1e-152">記事のこのセクションでは、ガバナンス MVP の設計を進化させて、新しい Azure ポリシーと Azure Cost Management の実装を含めます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-152">This section of the article will evolve the governance MVP design to include new Azure policies and an implementation of Azure Cost Management.</span></span> <span data-ttu-id="97a1e-153">これら 2 つの設計変更が一体となって、新しい企業ポリシー ステートメントが満たされます。</span><span class="sxs-lookup"><span data-stu-id="97a1e-153">Together, these two design changes will fulfill the new corporate policy statements.</span></span>

1. <span data-ttu-id="97a1e-154">Azure Cost Management を実装します</span><span class="sxs-lookup"><span data-stu-id="97a1e-154">Implement Azure Cost Management</span></span>
    1. <span data-ttu-id="97a1e-155">サブスクリプション パターンとリソース整合性の規範に合致する適切なアクセス スコープを確立します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-155">Establish the right scope of access to align with the subscription pattern and the Resource Consistency discipline.</span></span> <span data-ttu-id="97a1e-156">以前の記事で定義したガバナンス MVP と整合させることを前提にすると、このためには、高レベルのレポート作成を実行するクラウド ガバナンスチームのために、**登録アカウントのスコープ**へのアクセスが必要です。</span><span class="sxs-lookup"><span data-stu-id="97a1e-156">Assuming alignment with the governance MVP defined in prior articles, this requires **Enrollment Account Scope** access for the Cloud Governance team executing on high-level reporting.</span></span> <span data-ttu-id="97a1e-157">ガバナンスの外部にあるその他のチームには、**リソース グループのスコープ**へのアクセスが必要な場合があります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-157">Additional teams outside of governance may require **Resource Group Scope** access.</span></span>
    2. <span data-ttu-id="97a1e-158">Azure Cost Management で予算を編成します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-158">Establish a budget in Azure Cost Management.</span></span>
    3. <span data-ttu-id="97a1e-159">初期の推奨事項を確認して対応します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-159">Review and act on initial recommendations.</span></span> <span data-ttu-id="97a1e-160">レポート作成をサポートするための繰り返しのプロセスを用意します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-160">Have a recurring process to support reporting.</span></span>
    4. <span data-ttu-id="97a1e-161">初期用と繰り返し用の両方について、Azure Cost Management のレポートを構成して実行します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-161">Configure and execute Azure Cost Management Reporting, both initial and recurring.</span></span>
2. <span data-ttu-id="97a1e-162">Azure Policy を更新します</span><span class="sxs-lookup"><span data-stu-id="97a1e-162">Update Azure Policy</span></span>
    1. <span data-ttu-id="97a1e-163">タグ付け、管理グループ、サブスクリプション、およびリソース グループの値を監査して、差異がないか特定します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-163">Audit the tagging, management group, subscription, and resource group values to identify any deviation.</span></span>
    2. <span data-ttu-id="97a1e-164">SKU サイズ オプションを確定して、デプロイ計画ドキュメントに記載されている SKU にデプロイを制限します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-164">Establish SKU size options to limit deployments to SKUs listed in deployment planning documentation.</span></span>

## <a name="conclusion"></a><span data-ttu-id="97a1e-165">まとめ</span><span class="sxs-lookup"><span data-stu-id="97a1e-165">Conclusion</span></span>

<span data-ttu-id="97a1e-166">ガバナンス MVP にこれらのプロセスと変更を加えることで、コスト ガバナンスに関連するリスクの多くを軽減する助けとなります。</span><span class="sxs-lookup"><span data-stu-id="97a1e-166">Adding these processes and changes to the governance MVP helps mitigate many of the risks associated with cost governance.</span></span> <span data-ttu-id="97a1e-167">同時に、コスト管理に必要な可視性、説明責任、および最適化が実現します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-167">Together, they create the visibility, accountability, and optimization needed to control costs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="97a1e-168">次の手順</span><span class="sxs-lookup"><span data-stu-id="97a1e-168">Next steps</span></span>

<span data-ttu-id="97a1e-169">クラウドの導入が進み続けてより高いビジネス価値が提供されるのにつれて、リスクとクラウド ガバナンスのニーズも高度化します。</span><span class="sxs-lookup"><span data-stu-id="97a1e-169">As cloud adoption continues to evolve and deliver additional business value, risks and cloud governance needs will also evolve.</span></span> <span data-ttu-id="97a1e-170">これを体験している架空の企業にとっての次の手順は、このガバナンス投資を利用して複数のクラウドを管理することです。</span><span class="sxs-lookup"><span data-stu-id="97a1e-170">For the fictional company in this journey, the next step is using this governance investment to manage multiple clouds.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="97a1e-171">マルチクラウドの展開</span><span class="sxs-lookup"><span data-stu-id="97a1e-171">Multi-cloud evolution</span></span>](./multi-cloud-evolution.md)
