---
title: CAF:合理化の 5 R
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: デジタル資産を合理化するために使用できるオプションを確認します。
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 25987e790df5c82ad2c8296f1be1dabc949f1629
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243303"
---
# <a name="the-5-rs-of-rationalization"></a><span data-ttu-id="6989d-103">合理化の 5 R</span><span class="sxs-lookup"><span data-stu-id="6989d-103">The 5 Rs of rationalization</span></span>

<span data-ttu-id="6989d-104">クラウドの合理化は、クラウド内の各資産の移行または最新化に対する最適なアプローチを決定するために資産を評価するプロセスです。</span><span class="sxs-lookup"><span data-stu-id="6989d-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="6989d-105">合理化のプロセスの詳細については、「[デジタル資産とは](overview.md)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="6989d-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="6989d-106">ここに示す "合理化の 5 R" は、合理化のための最も一般的なオプションについて説明しています。</span><span class="sxs-lookup"><span data-stu-id="6989d-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="6989d-107">リホスト</span><span class="sxs-lookup"><span data-stu-id="6989d-107">Rehost</span></span>

<span data-ttu-id="6989d-108">リホストの取り組みは、"リフト アンド シフト" とも呼ばれ、アーキテクチャ全体への変更を最小限に抑えて、現在の状態の資産を選択されたクラウド プロバイダーに移動します。</span><span class="sxs-lookup"><span data-stu-id="6989d-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="6989d-109">一般的な促進要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-109">Common drivers could include:</span></span>

* <span data-ttu-id="6989d-110">CapEx の削減</span><span class="sxs-lookup"><span data-stu-id="6989d-110">Reduce CapEx</span></span>
* <span data-ttu-id="6989d-111">データセンターの占有領域の解放</span><span class="sxs-lookup"><span data-stu-id="6989d-111">Free up datacenter space</span></span>
* <span data-ttu-id="6989d-112">クラウド ROI の迅速化</span><span class="sxs-lookup"><span data-stu-id="6989d-112">Quick cloud ROI</span></span>

<span data-ttu-id="6989d-113">定量分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6989d-114">VM サイズ (CPU、メモリ、ストレージ)</span><span class="sxs-lookup"><span data-stu-id="6989d-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6989d-115">依存関係 (ネットワーク トラフィック)</span><span class="sxs-lookup"><span data-stu-id="6989d-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6989d-116">資産の互換性</span><span class="sxs-lookup"><span data-stu-id="6989d-116">Asset compatibility</span></span>

<span data-ttu-id="6989d-117">定性分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6989d-118">変更の許容範囲</span><span class="sxs-lookup"><span data-stu-id="6989d-118">Tolerance for change</span></span>
* <span data-ttu-id="6989d-119">ビジネスの優先順位</span><span class="sxs-lookup"><span data-stu-id="6989d-119">Business priorities</span></span>
* <span data-ttu-id="6989d-120">重要なビジネス イベント</span><span class="sxs-lookup"><span data-stu-id="6989d-120">Critical business events</span></span>
* <span data-ttu-id="6989d-121">プロセスの依存関係</span><span class="sxs-lookup"><span data-stu-id="6989d-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="6989d-122">リファクター</span><span class="sxs-lookup"><span data-stu-id="6989d-122">Refactor</span></span>

<span data-ttu-id="6989d-123">サービスとしてのプラットフォーム (PaaS) オプションは、多くのアプリケーションに関連した運用コストを削減できます。</span><span class="sxs-lookup"><span data-stu-id="6989d-123">Platform as a service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="6989d-124">PaaS ベースのモデルに適合するように、アプリケーションを少しリファクタリングすることが適切である場合があります。</span><span class="sxs-lookup"><span data-stu-id="6989d-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="6989d-125">リファクターはまた、アプリケーションが新しいビジネス機会を実現できるようにコードをリファクタリングするアプリケーション開発プロセスも指します。</span><span class="sxs-lookup"><span data-stu-id="6989d-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="6989d-126">一般的な促進要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-126">Common drivers could include:</span></span>

* <span data-ttu-id="6989d-127">より迅速で、より短い更新</span><span class="sxs-lookup"><span data-stu-id="6989d-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="6989d-128">コードの移植性</span><span class="sxs-lookup"><span data-stu-id="6989d-128">Code portability</span></span>
* <span data-ttu-id="6989d-129">クラウド効率の向上 (リソース、速度、コスト)</span><span class="sxs-lookup"><span data-stu-id="6989d-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="6989d-130">定量分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6989d-131">アプリケーション資産のサイズ (CPU、メモリ、ストレージ)</span><span class="sxs-lookup"><span data-stu-id="6989d-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6989d-132">依存関係 (ネットワーク トラフィック)</span><span class="sxs-lookup"><span data-stu-id="6989d-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6989d-133">ユーザーのトラフィック (ページ ビュー、ページ上の時間、読み込み時間)</span><span class="sxs-lookup"><span data-stu-id="6989d-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6989d-134">開発プラットフォーム (言語、データ プラットフォーム、中間層サービス)</span><span class="sxs-lookup"><span data-stu-id="6989d-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6989d-135">定性分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6989d-136">継続的なビジネス投資</span><span class="sxs-lookup"><span data-stu-id="6989d-136">Continued business investments</span></span>
* <span data-ttu-id="6989d-137">増え続けるオプション/タイムライン</span><span class="sxs-lookup"><span data-stu-id="6989d-137">Bursting options/timelines</span></span>
* <span data-ttu-id="6989d-138">ビジネス プロセスの依存関係</span><span class="sxs-lookup"><span data-stu-id="6989d-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="6989d-139">再設計</span><span class="sxs-lookup"><span data-stu-id="6989d-139">Rearchitect</span></span>

<span data-ttu-id="6989d-140">一部の古いアプリケーションは、そのアプリケーションの構築時に行われたアーキテクチャ上の決定のために、クラウド プロバイダーと互換性がありません。</span><span class="sxs-lookup"><span data-stu-id="6989d-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="6989d-141">この場合は、変革の前に、そのアプリケーションの再設計が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="6989d-141">In these cases, the application may need to be rearchitected before transformation.</span></span>

<span data-ttu-id="6989d-142">別の場合として、クラウドと互換性はあるが、クラウド ネイティブの利点がないアプリケーションは、クラウド ネイティブ アプリケーションになるようにソリューションを再設計することによって、コスト効率と運用効率を生み出す可能性があります。</span><span class="sxs-lookup"><span data-stu-id="6989d-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="6989d-143">一般的な促進要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-143">Common drivers could include:</span></span>

* <span data-ttu-id="6989d-144">アプリケーションの規模と俊敏性</span><span class="sxs-lookup"><span data-stu-id="6989d-144">Application scale and agility</span></span>
* <span data-ttu-id="6989d-145">新しいクラウド機能のより容易な導入</span><span class="sxs-lookup"><span data-stu-id="6989d-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="6989d-146">テクノロジ スタックの混在</span><span class="sxs-lookup"><span data-stu-id="6989d-146">Mix of technology stacks</span></span>

<span data-ttu-id="6989d-147">定量分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6989d-148">アプリケーション資産のサイズ (CPU、メモリ、ストレージ)</span><span class="sxs-lookup"><span data-stu-id="6989d-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6989d-149">依存関係 (ネットワーク トラフィック)</span><span class="sxs-lookup"><span data-stu-id="6989d-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6989d-150">ユーザーのトラフィック (ページ ビュー、ページ上の時間、読み込み時間)</span><span class="sxs-lookup"><span data-stu-id="6989d-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6989d-151">開発プラットフォーム (言語、データ プラットフォーム、中間層サービス)</span><span class="sxs-lookup"><span data-stu-id="6989d-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6989d-152">定性分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6989d-153">増大するビジネス投資</span><span class="sxs-lookup"><span data-stu-id="6989d-153">Growing business investments</span></span>
* <span data-ttu-id="6989d-154">運用コスト</span><span class="sxs-lookup"><span data-stu-id="6989d-154">Operational costs</span></span>
* <span data-ttu-id="6989d-155">潜在的なフィードバック ループおよび DevOps 投資</span><span class="sxs-lookup"><span data-stu-id="6989d-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="6989d-156">再構築</span><span class="sxs-lookup"><span data-stu-id="6989d-156">Rebuild</span></span>

<span data-ttu-id="6989d-157">一部のシナリオでは、アプリケーションを引き継ぐために克服する必要のある差分が大きすぎて、それ以上の投資を正当化できない場合があります。</span><span class="sxs-lookup"><span data-stu-id="6989d-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="6989d-158">これは、ビジネスのニーズを満たすために使用されるアプリケーションに特に当てはまりますが、現在はサポートされていないか、または今日のビジネス プロセスの実行方法と整合していません。</span><span class="sxs-lookup"><span data-stu-id="6989d-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="6989d-159">この場合は、[クラウド ネイティブな](https://azure.microsoft.com/overview/cloudnative/)アプローチに整合させるための新しいコード ベースが作成されます。</span><span class="sxs-lookup"><span data-stu-id="6989d-159">In this case, a new code base is created to align with a [cloud native](https://azure.microsoft.com/overview/cloudnative/) approach.</span></span>

<span data-ttu-id="6989d-160">一般的な促進要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-160">Common drivers could include:</span></span>

* <span data-ttu-id="6989d-161">イノベーションの促進</span><span class="sxs-lookup"><span data-stu-id="6989d-161">Accelerate innovation</span></span>
* <span data-ttu-id="6989d-162">アプリ構築の迅速化</span><span class="sxs-lookup"><span data-stu-id="6989d-162">Build apps faster</span></span>
* <span data-ttu-id="6989d-163">運用コストの削減</span><span class="sxs-lookup"><span data-stu-id="6989d-163">Reduce operational cost</span></span>

<span data-ttu-id="6989d-164">定量分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6989d-165">アプリケーション資産のサイズ (CPU、メモリ、ストレージ)</span><span class="sxs-lookup"><span data-stu-id="6989d-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6989d-166">依存関係 (ネットワーク トラフィック)</span><span class="sxs-lookup"><span data-stu-id="6989d-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6989d-167">ユーザーのトラフィック (ページ ビュー、ページ上の時間、読み込み時間)</span><span class="sxs-lookup"><span data-stu-id="6989d-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6989d-168">開発プラットフォーム (言語、データ プラットフォーム、中間層サービス)</span><span class="sxs-lookup"><span data-stu-id="6989d-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6989d-169">定性分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="6989d-170">エンド ユーザー満足度の低下</span><span class="sxs-lookup"><span data-stu-id="6989d-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="6989d-171">機能によって制限されるビジネス プロセス</span><span class="sxs-lookup"><span data-stu-id="6989d-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="6989d-172">潜在的なコスト、エクスペリエンス、または収益増加</span><span class="sxs-lookup"><span data-stu-id="6989d-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="6989d-173">Replace</span><span class="sxs-lookup"><span data-stu-id="6989d-173">Replace</span></span>

<span data-ttu-id="6989d-174">ソリューションは一般に、現時点で使用可能な最高のテクノロジとアプローチを使用して実装されます。</span><span class="sxs-lookup"><span data-stu-id="6989d-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="6989d-175">場合によっては、サービスとしてのソフトウェア (SaaS) アプリケーションが、ホストされるアプリケーションに必要なすべての機能を満足できることがあります。</span><span class="sxs-lookup"><span data-stu-id="6989d-175">In some cases, software as a service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="6989d-176">これらのシナリオでは、ワークロードを将来の置き換え用に予定し、変革の取り組みから実質的に削除することもできます。</span><span class="sxs-lookup"><span data-stu-id="6989d-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="6989d-177">一般的な促進要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-177">Common drivers could include:</span></span>

* <span data-ttu-id="6989d-178">業界のベスト プラクティスに関する標準化</span><span class="sxs-lookup"><span data-stu-id="6989d-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="6989d-179">ビジネス プロセス主導のアプローチの導入の促進</span><span class="sxs-lookup"><span data-stu-id="6989d-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="6989d-180">競争力のある差別化または利点を生み出すアプリケーションへの開発投資の再割り当て</span><span class="sxs-lookup"><span data-stu-id="6989d-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="6989d-181">定量分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6989d-182">一般的な運用コストの削減</span><span class="sxs-lookup"><span data-stu-id="6989d-182">General operating cost reductions</span></span>
* <span data-ttu-id="6989d-183">VM サイズ (CPU、メモリ、ストレージ)</span><span class="sxs-lookup"><span data-stu-id="6989d-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6989d-184">依存関係 (ネットワーク トラフィック)</span><span class="sxs-lookup"><span data-stu-id="6989d-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6989d-185">廃棄される資産</span><span class="sxs-lookup"><span data-stu-id="6989d-185">Assets to be retired</span></span>

<span data-ttu-id="6989d-186">定性分析の要因には、次のものが含まれます。</span><span class="sxs-lookup"><span data-stu-id="6989d-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6989d-187">現在のアーキテクチャと SaaS ソリューションの間のコスト メリット分析</span><span class="sxs-lookup"><span data-stu-id="6989d-187">Cost benefit analysis of the current architecture versus a SaaS solution</span></span>
* <span data-ttu-id="6989d-188">ビジネス プロセス マップ</span><span class="sxs-lookup"><span data-stu-id="6989d-188">Business process maps</span></span>
* <span data-ttu-id="6989d-189">データ スキーマ</span><span class="sxs-lookup"><span data-stu-id="6989d-189">Data schemas</span></span>
* <span data-ttu-id="6989d-190">カスタムまたは自動化プロセス</span><span class="sxs-lookup"><span data-stu-id="6989d-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="6989d-191">次の手順</span><span class="sxs-lookup"><span data-stu-id="6989d-191">Next steps</span></span>

<span data-ttu-id="6989d-192">これらの合理化の 5 R はまとめて、各アプリケーションの将来の状態に関連した合理化の決定を行うためにデジタル資産に適用できます。</span><span class="sxs-lookup"><span data-stu-id="6989d-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6989d-193">デジタル資産とは</span><span class="sxs-lookup"><span data-stu-id="6989d-193">What is a digital estate?</span></span>](overview.md)
