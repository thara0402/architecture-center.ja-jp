---
title: 'CAF: セキュリティ ベースラインの導入理由とビジネス リスク'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: セキュリティ ベースラインの導入理由とビジネス リスク
author: BrianBlanchard
ms.openlocfilehash: 8407ed358e5862e466176096ee6a82ad792027cb
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58247313"
---
# <a name="security-baseline-motivations-and-business-risks"></a><span data-ttu-id="35be5-103">セキュリティ ベースラインの導入理由とビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="35be5-103">Security Baseline motivations and business risks</span></span>

<span data-ttu-id="35be5-104">この記事では、クラウド ガバナンス戦略において、お客様が通常はセキュリティ ベースライン規範を導入する理由について説明します。</span><span class="sxs-lookup"><span data-stu-id="35be5-104">This article discusses the reasons that customers typically adopt a Security Baseline discipline within a cloud governance strategy.</span></span> <span data-ttu-id="35be5-105">また、ポリシー ステートメントを追いやる可能性のある潜在的なビジネス リスクの例をいくつか示します。</span><span class="sxs-lookup"><span data-stu-id="35be5-105">It also provides a few examples of potential business risks that can drive policy statements.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="is-a-security-baseline-relevant"></a><span data-ttu-id="35be5-106">セキュリティ ベースラインの関連性</span><span class="sxs-lookup"><span data-stu-id="35be5-106">Is a Security Baseline relevant?</span></span>

<span data-ttu-id="35be5-107">セキュリティは、いかなる IT 組織にとっても重要な懸念事項です。</span><span class="sxs-lookup"><span data-stu-id="35be5-107">Security is a key concern for any IT organization.</span></span> <span data-ttu-id="35be5-108">クラウドのデプロイは、従来のオンプレミス データ センターでホストされるワークロードと同じセキュリティ リスクの大半と直面しています。</span><span class="sxs-lookup"><span data-stu-id="35be5-108">Cloud deployments face many of the same security risks as workloads hosted in traditional on-premises datacenters.</span></span> <span data-ttu-id="35be5-109">ただし、ワークロードを格納して実行する物理的なハードウェアの直接的所有権がないというパブリック クラウド プラットフォームの性質上、クラウドのセキュリティには独自のポリシーとプロセスが必要であることを意味します。</span><span class="sxs-lookup"><span data-stu-id="35be5-109">However, the nature of public cloud platforms, with a lack of direct ownership of the physical hardware storing and running your workloads, means cloud security requires its own policy and processes.</span></span>

<span data-ttu-id="35be5-110">クラウド セキュリティ ガバナンスと従来のセキュリティ ポリシーを区別する主な事柄の 1 つは、リソースを作成できる容易さです。これは、デプロイの前にセキュリティが考慮されていない場合、脆弱性の増加につながる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="35be5-110">One of the primary things that set cloud security governance apart from traditional security policy is the ease with which resources can be created, potentially adding vulnerabilities if security isn't considered before deployment.</span></span> <span data-ttu-id="35be5-111">[ソフトウェア定義ネットワーク (SDN)](../../decision-guides/software-defined-network/overview.md) などのテクノロジで提供される、急速に変化するクラウド ベースのネットワーク トポロジを想定した柔軟性は、予期しない方法で、ネットワークの全体としての攻撃対象を簡単に変えてしまう可能性もあります。</span><span class="sxs-lookup"><span data-stu-id="35be5-111">The flexibility that technologies like [software defined networking (SDN)](../../decision-guides/software-defined-network/overview.md) provide for rapidly changing your cloud-based network topology can also easily modify your overall network attack surface in unforeseen ways.</span></span> <span data-ttu-id="35be5-112">クラウド プラットフォームには、オンプレミス環境では常に可能であるとは限らない方法でセキュリティ機能を高めることができるツールと機能も用意されています。</span><span class="sxs-lookup"><span data-stu-id="35be5-112">Cloud platforms also provide tools and features that can improve your security capabilities in ways not always possible in on-premises environments.</span></span>

<span data-ttu-id="35be5-113">セキュリティ ポリシーとプロセスへの投資額は、実際のクラウド デプロイの性質に大きく左右されます。</span><span class="sxs-lookup"><span data-stu-id="35be5-113">The amount you invest into security policy and processes will depend a great deal on the nature of your cloud deployment.</span></span> <span data-ttu-id="35be5-114">テスト用の初期デプロイであれば最も基本的なセキュリティ ポリシーのみを導入するのに対して、ミッション クリティカルなワークロードには必然的に、複雑で広範なセキュリティ ニーズへの対処が伴います。</span><span class="sxs-lookup"><span data-stu-id="35be5-114">Initial test deployments may only need the most basic of security policies in place, while a mission-critical workload will entail addressing complex and extensive security needs.</span></span> <span data-ttu-id="35be5-115">すべてのデプロイで、いずれかのレベルの規範に取り組む必要があります。</span><span class="sxs-lookup"><span data-stu-id="35be5-115">All deployments will need to engage with the discipline at some level.</span></span>

<span data-ttu-id="35be5-116">セキュリティ ベースライン規範の対象は、セキュリティ リスクからクラウド デプロイを保護するために配置できる企業ポリシーと手動プロセスです。</span><span class="sxs-lookup"><span data-stu-id="35be5-116">The Security Baseline discipline covers the corporate policies and manual processes that you can put in place to protect your cloud deployment against security risks.</span></span>

> [!NOTE]
><span data-ttu-id="35be5-117">セキュリティ ベースラインのコンテキストで [ID ベースライン](../identity-baseline/overview.md)について理解し、それがアクセス制御にどう関係するかを理解することは大切ですが、[クラウド ガバナンスの 5 つの規範](../overview.md)では、[ID ベースライン](../identity-baseline/overview.md)はセキュリティ ベースラインとは別の独自の規範であると見なしています。</span><span class="sxs-lookup"><span data-stu-id="35be5-117">While it is important to understand [Identity Baseline](../identity-baseline/overview.md) in the context of Security Baseline and how that relates to Access Control, the [Five Disciplines of Cloud Governance](../overview.md) calls out [Identity Baseline](../identity-baseline/overview.md) as its own discipline, separate from Security Baseline.</span></span>

## <a name="business-risk"></a><span data-ttu-id="35be5-118">ビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="35be5-118">Business risk</span></span>

<span data-ttu-id="35be5-119">セキュリティ ベースライン規範では、セキュリティ関連の中心的ビジネス リスクへの取り組みを試みます。</span><span class="sxs-lookup"><span data-stu-id="35be5-119">The Security Baseline discipline attempts to address core security-related business risks.</span></span> <span data-ttu-id="35be5-120">会社と協力してこれらのリスクを特定し、クラウドのデプロイを計画して実装するときに、各リスクに関連性がないかを監視します。</span><span class="sxs-lookup"><span data-stu-id="35be5-120">Work with your business to identify these risks and monitor each of them for relevance as you plan for and implement your cloud deployments.</span></span>

<span data-ttu-id="35be5-121">リスクは組織によって異なりますが、クラウド ガバナンス チームでの検討の開始点として利用できる一般的なセキュリティ関連リスクには、以下のものがあります。</span><span class="sxs-lookup"><span data-stu-id="35be5-121">Risks will differ between organization, but the following serve as common security-related risks that you can use as a starting point for discussions within your Cloud Governance team:</span></span>

- <span data-ttu-id="35be5-122">**データ侵害**。</span><span class="sxs-lookup"><span data-stu-id="35be5-122">**Data breach**.</span></span> <span data-ttu-id="35be5-123">クラウドでホストされている機密データの不注意による開示や損失は、顧客の喪失、契約上の問題、法的責任につながる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="35be5-123">Inadvertent exposure or loss of sensitive cloud-hosted data can lead to losing customers, contractual issues, or legal consequences.</span></span>
- <span data-ttu-id="35be5-124">**サービスの中断**。</span><span class="sxs-lookup"><span data-stu-id="35be5-124">**Service disruption**.</span></span> <span data-ttu-id="35be5-125">セキュリティが確保されていないインフラストラクチャによる停止やその他のパフォーマンスの問題は、通常業務を中断させ、生産性や商機の逸失に至る可能性があります。</span><span class="sxs-lookup"><span data-stu-id="35be5-125">Outages and other performance issues due to insecure infrastructure interrupts normal operations and can result in lost productivity or lost business.</span></span>

## <a name="next-steps"></a><span data-ttu-id="35be5-126">次の手順</span><span class="sxs-lookup"><span data-stu-id="35be5-126">Next steps</span></span>

<span data-ttu-id="35be5-127">[クラウド管理テンプレート](./template.md)を使用して、現在のクラウド導入計画によって生じる可能性の高いビジネス リスクを文書化します。</span><span class="sxs-lookup"><span data-stu-id="35be5-127">Using the [Cloud Management Template](./template.md), document business risks that are likely to be introduced by the current cloud adoption plan.</span></span>

<span data-ttu-id="35be5-128">現実的なビジネス リスクについての理解が確実に得られたら、リスクに対する企業の許容範囲と、その許容範囲を監視するための指標と主なメトリックについて文書化することが、次の手順となります。</span><span class="sxs-lookup"><span data-stu-id="35be5-128">Once an understanding of realistic business risks is established, the next step is to document the business's tolerance for risk and the indicators and key metrics to monitor that tolerance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="35be5-129">指標、メトリック、リスクの許容範囲について理解する</span><span class="sxs-lookup"><span data-stu-id="35be5-129">Understand indicators, metrics, and risk tolerance</span></span>](./metrics-tolerance.md)
