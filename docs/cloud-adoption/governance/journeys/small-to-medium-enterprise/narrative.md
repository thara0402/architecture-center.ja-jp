---
title: CAF:中小企業 - ガバナンス戦略の背景にある初期の物語
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: この物語では、中小企業のガバナンス体験のユース ケースを組み立てます。
author: BrianBlanchard
ms.openlocfilehash: 6173b01f310169017761110d6641acfa51d45df8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902209"
---
# <a name="small-to-medium-enterprise-the-narrative-behind-the-governance-strategy"></a><span data-ttu-id="c560a-103">中小企業:ガバナンス戦略の背景にある物語</span><span class="sxs-lookup"><span data-stu-id="c560a-103">Small-to-medium enterprise: The narrative behind the governance strategy</span></span>

<span data-ttu-id="c560a-104">以下の物語では、[中小企業のガバナンス体験](./overview.md)のユース ケースについて説明します。</span><span class="sxs-lookup"><span data-stu-id="c560a-104">The following narrative describes the use case for the [small-to-medium enterprise governance journey](./overview.md).</span></span> <span data-ttu-id="c560a-105">この体験を実行する前に、この物語に反映されている前提条件とその理由を理解しておくことが重要です。</span><span class="sxs-lookup"><span data-stu-id="c560a-105">Before implementing the journey, it’s important to understand the assumptions and reasoning that are reflected in this narrative.</span></span> <span data-ttu-id="c560a-106">そうすれば、ガバナンス戦略をお客様の組織の体験に合わせることができます。</span><span class="sxs-lookup"><span data-stu-id="c560a-106">Then you can better align the governance strategy to your own organization’s journey.</span></span>

## <a name="back-story"></a><span data-ttu-id="c560a-107">背景</span><span class="sxs-lookup"><span data-stu-id="c560a-107">Back story</span></span>

<span data-ttu-id="c560a-108">取締役会は、いくつかの方法で事業を活性化する計画からその年を始めました。</span><span class="sxs-lookup"><span data-stu-id="c560a-108">The board of directors started the year with plans to energize the business in several ways.</span></span> <span data-ttu-id="c560a-109">市場シェアを獲得するために顧客エクスペリエンスを改善するリーダーシップを推し進めています。</span><span class="sxs-lookup"><span data-stu-id="c560a-109">They are pushing leadership to improve customer experiences to gain market share.</span></span> <span data-ttu-id="c560a-110">また、会社を業界の思想的リーダーと位置づけることになる新しい製品とサービスを求めています。</span><span class="sxs-lookup"><span data-stu-id="c560a-110">They are also pushing for new products and services that will position the company as a thought leader in the industry.</span></span> <span data-ttu-id="c560a-111">さらに、無駄を減らし、不必要なコストを削減する取り組みも並行して始めました。</span><span class="sxs-lookup"><span data-stu-id="c560a-111">They also initiated a parallel effort to reduce waste and cut unnecessary costs.</span></span> <span data-ttu-id="c560a-112">威圧的ではありますが、取締役会の行動とリーダーシップは、この取り組みが将来の成長に最大限の資本を集中させていることを示します。</span><span class="sxs-lookup"><span data-stu-id="c560a-112">Though intimidating, the actions of the board and leadership show that this effort is focusing as much capital as possible on future growth.</span></span>

<span data-ttu-id="c560a-113">過去には、会社の CIO はこのような戦略的な話し合いから除外されていました。</span><span class="sxs-lookup"><span data-stu-id="c560a-113">In the past, the company’s CIO has been excluded from these strategic conversations.</span></span> <span data-ttu-id="c560a-114">ただし、将来のビジョンは本質的にテクノロジの成長と関連しているので、このような大きな計画を指導するための会議には IT 部門の席が用意されます。</span><span class="sxs-lookup"><span data-stu-id="c560a-114">However, because the future vision is intrinsically linked to technical growth, IT has a seat at the table to help guide these big plans.</span></span> <span data-ttu-id="c560a-115">現在、IT 部門は新しい方法を提示することが期待されています。</span><span class="sxs-lookup"><span data-stu-id="c560a-115">IT is now expected to deliver in new ways.</span></span> <span data-ttu-id="c560a-116">チームはこのような変更に実際には備えておらず、学習曲線に苦労する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="c560a-116">The team isn’t really prepared for these changes and is likely to struggle with the learning curve.</span></span>

## <a name="business-characteristics"></a><span data-ttu-id="c560a-117">ビジネスの特性</span><span class="sxs-lookup"><span data-stu-id="c560a-117">Business characteristics</span></span>

<span data-ttu-id="c560a-118">この会社のビジネス プロファイルは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="c560a-118">The company has the following business profile:</span></span>

- <span data-ttu-id="c560a-119">すべての営業と業務は 1 つの国内で行われ、グローバルな顧客の割合は高くありません。</span><span class="sxs-lookup"><span data-stu-id="c560a-119">All sales and operations reside in a single country, with a low percentage of global customers.</span></span>
- <span data-ttu-id="c560a-120">事業は単一の事業部門として運営され、営業、マーケティング、業務、IT などの職務に合わせて予算が調整されます。</span><span class="sxs-lookup"><span data-stu-id="c560a-120">The business operates as a single business unit, with budget aligned to functions, including Sales, Marketing, Operations, and IT.</span></span>
- <span data-ttu-id="c560a-121">事業部門は、IT の大部分を資本の流出またはコスト センターと考えています。</span><span class="sxs-lookup"><span data-stu-id="c560a-121">The business views most of IT as a capital drain or a cost center.</span></span>

## <a name="current-state"></a><span data-ttu-id="c560a-122">現在の状態</span><span class="sxs-lookup"><span data-stu-id="c560a-122">Current state</span></span>

<span data-ttu-id="c560a-123">この会社の IT およびクラウド運用の現在の状態は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="c560a-123">Here is the current state of the company’s IT and cloud operations:</span></span>

- <span data-ttu-id="c560a-124">IT 部門は 2 つのホスト型インフラストラクチャ環境を運用しています。</span><span class="sxs-lookup"><span data-stu-id="c560a-124">IT operates two hosted infrastructure environments.</span></span> <span data-ttu-id="c560a-125">1 つ目の環境には運用資産が含まれています。</span><span class="sxs-lookup"><span data-stu-id="c560a-125">One environment contains production assets.</span></span> <span data-ttu-id="c560a-126">2 つ目の環境にはディザスター リカバリーと一部の開発/テスト資産が含まれています。</span><span class="sxs-lookup"><span data-stu-id="c560a-126">The second environment contains disaster recovery and some dev/test assets.</span></span> <span data-ttu-id="c560a-127">これらの環境は、2 つの異なるプロバイダーによってホストされます。</span><span class="sxs-lookup"><span data-stu-id="c560a-127">These environments are hosted by two different providers.</span></span> <span data-ttu-id="c560a-128">IT 部門は、これら 2 つのデータセンターをそれぞれ Prod と DR と呼びます。</span><span class="sxs-lookup"><span data-stu-id="c560a-128">IT refers to these two datacenters as Prod and DR respectively.</span></span>
- <span data-ttu-id="c560a-129">IT 部門は、エンドユーザーのすべての電子メール アカウントを Office 365 に移行することからクラウドを始めました。</span><span class="sxs-lookup"><span data-stu-id="c560a-129">IT entered the cloud by migrating all end-user email accounts to Office 365.</span></span> <span data-ttu-id="c560a-130">この移行は 6 か月前に完了しました。</span><span class="sxs-lookup"><span data-stu-id="c560a-130">This migration was completed six months ago.</span></span> <span data-ttu-id="c560a-131">他の IT 資産は、クラウドにはほとんどデプロイされていません。</span><span class="sxs-lookup"><span data-stu-id="c560a-131">Few other IT assets have been deployed to the cloud.</span></span>
- <span data-ttu-id="c560a-132">アプリケーション開発チームは、クラウドのネイティブ機能について学ぶために開発/テストの予算内で取り組んでいます。</span><span class="sxs-lookup"><span data-stu-id="c560a-132">The application development teams are working in a dev/test capacity to learn about cloud native capabilities.</span></span>
- <span data-ttu-id="c560a-133">ビジネス インテリジェンス (BI) チームは、クラウド内のビッグ データと新しいプラットフォーム上のデータのキュレーションを実験しています。</span><span class="sxs-lookup"><span data-stu-id="c560a-133">The business intelligence (BI) team is experimenting with big data in the cloud and curation of data on new platforms.</span></span>
- <span data-ttu-id="c560a-134">この会社には、顧客の個人を特定できる情報 (PII) と財務データをクラウドでホストすることはできないという、明確に定義されたポリシーがあります。これにより、現在のデプロイでミッションクリティカル アプリケーションが制限されます。</span><span class="sxs-lookup"><span data-stu-id="c560a-134">The company has a loosely defined policy stating that customer personally identifiable information (PII) and financial data cannot be hosted in the cloud, which limits mission-critical applications in the current deployments.</span></span>
- <span data-ttu-id="c560a-135">IT への投資は、主に資本経費 (CapEx) によって管理されています。</span><span class="sxs-lookup"><span data-stu-id="c560a-135">IT investments are controlled largely by capital expense (CapEx).</span></span> <span data-ttu-id="c560a-136">このような投資は毎年計画されます。</span><span class="sxs-lookup"><span data-stu-id="c560a-136">Those investments are planned yearly.</span></span> <span data-ttu-id="c560a-137">過去数年間で、基本的なメンテナンス要件を越える投資はほとんどありませんでした。</span><span class="sxs-lookup"><span data-stu-id="c560a-137">In the past several years, investments have included little more than basic maintenance requirements.</span></span>

## <a name="future-state"></a><span data-ttu-id="c560a-138">将来的な状態</span><span class="sxs-lookup"><span data-stu-id="c560a-138">Future state</span></span>

<span data-ttu-id="c560a-139">今後の数年間で、次のような変化が予想されます。</span><span class="sxs-lookup"><span data-stu-id="c560a-139">The following changes are anticipated over the next several years:</span></span>

- <span data-ttu-id="c560a-140">CIO は、将来的な状態の目標を見越して PII と財務データに関するポリシーを見直しています。</span><span class="sxs-lookup"><span data-stu-id="c560a-140">The CIO is reviewing the policy on PII and financial data to allow for the future state goals.</span></span>
- <span data-ttu-id="c560a-141">アプリケーション開発チームと BI チームは、顧客エンゲージメントと新製品のビジョンに基づいて、今後 24 か月間でクラウドベースのソリューションを運用環境にリリースしたいと考えています。</span><span class="sxs-lookup"><span data-stu-id="c560a-141">The application development and BI teams want to release cloud-based solutions to production over the next 24 months based on the vision for customer engagement and new products.</span></span>
- <span data-ttu-id="c560a-142">今年、IT チームは、2,000 個の VM をクラウドに移行することで、DR データセンターのディザスター リカバリー ワークロードを廃止する予定です。</span><span class="sxs-lookup"><span data-stu-id="c560a-142">This year, the IT team will finish retiring the disaster recovery workloads of the DR datacenter by migrating 2,000 VMs to the cloud.</span></span> <span data-ttu-id="c560a-143">これにより、今後 5 年間で推定 2,500 万米ドルのコスト削減が見込まれます。</span><span class="sxs-lookup"><span data-stu-id="c560a-143">This is expected to produce an estimated $25M USD cost savings over the next five years.</span></span>
    <span data-ttu-id="c560a-144">![今後 5 年間で 2,500 万米ドルのコスト削減を示すオンプレミスのコストと Azure のコストの比較](../../../_images/governance/calculator-small-to-medium-enterprise.png)</span><span class="sxs-lookup"><span data-stu-id="c560a-144">![On-premise costs versus Azure costs demonstrating a return of $25M USD over the next five years](../../../_images/governance/calculator-small-to-medium-enterprise.png)</span></span>
- <span data-ttu-id="c560a-145">この会社は、割り当てられている CapEx を IT 部門内の運用経費 (OpEx) として再配分することで、IT 投資の方法を変えることを計画しています。</span><span class="sxs-lookup"><span data-stu-id="c560a-145">The company plans to change how it makes IT investments by repositioning the committed CapEx as an operational expense (OpEx) within IT.</span></span> <span data-ttu-id="c560a-146">この変更により、コスト管理が強化され、IT 部門は他の計画されている作業を加速できます。</span><span class="sxs-lookup"><span data-stu-id="c560a-146">This change will provide greater cost control and enable IT to accelerate other planned efforts.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c560a-147">次の手順</span><span class="sxs-lookup"><span data-stu-id="c560a-147">Next steps</span></span>

<span data-ttu-id="c560a-148">この会社は、ガバナンスの実装を形作るための企業ポリシーを策定しました。</span><span class="sxs-lookup"><span data-stu-id="c560a-148">The company has developed a corporate policy to shape the governance implementation.</span></span> <span data-ttu-id="c560a-149">企業ポリシーによって、多くの技術的な決定は変わります。</span><span class="sxs-lookup"><span data-stu-id="c560a-149">The corporate policy drives many of the technical decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="c560a-150">初期の企業ポリシーの確認</span><span class="sxs-lookup"><span data-stu-id="c560a-150">Review the initial corporate policy</span></span>](./initial-corporate-policy.md)
