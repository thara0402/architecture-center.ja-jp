---
title: CAF:大企業 - ガバナンス戦略の背景にある初期の企業ポリシー
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: 大企業 - ガバナンス戦略の背景にある初期の企業ポリシー。
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902009"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="50c8a-103">大企業:ガバナンス戦略の背景にある初期の企業ポリシー</span><span class="sxs-lookup"><span data-stu-id="50c8a-103">Large enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="50c8a-104">次の企業ポリシーは、この体験の出発点である初期のガバナンス ポジションを定義しています。</span><span class="sxs-lookup"><span data-stu-id="50c8a-104">The following corporate policy defines the initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="50c8a-105">この記事では、初期段階のリスク、初期ポリシー ステートメント、ポリシー ステートメントを適用するための初期プロセスを定義します。</span><span class="sxs-lookup"><span data-stu-id="50c8a-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="50c8a-106">企業ポリシーは技術文書ではありませんが、多くの技術的判断の根拠になります。</span><span class="sxs-lookup"><span data-stu-id="50c8a-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="50c8a-107">[概要](./overview.md)で説明するガバナンス MVP は、最終的にこのポリシーから派生します。</span><span class="sxs-lookup"><span data-stu-id="50c8a-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="50c8a-108">ガバナンス MVP を実装する前に、組織はその目標とビジネス リスクに基づいて企業ポリシーを策定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="50c8a-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="50c8a-109">クラウド ガバナンス チーム</span><span class="sxs-lookup"><span data-stu-id="50c8a-109">Cloud Governance team</span></span>

<span data-ttu-id="50c8a-110">CIO は最近、個人情報ポリシーとミッションクリティカル ポリシーの歴史を理解し、それらのポリシーを変更した場合の影響をレビューするために、IT ガバナンス チームとの会議を開きました。</span><span class="sxs-lookup"><span data-stu-id="50c8a-110">The CIO recently held a meeting with the IT Governance team to understand the history of the PII and mission-critical policies and review the impact of changing those policies.</span></span> <span data-ttu-id="50c8a-111">IT と企業にとってのクラウドの可能性全般についても議論しました。</span><span class="sxs-lookup"><span data-stu-id="50c8a-111">She also discussed the overall potential of the cloud for IT and the company.</span></span>

<span data-ttu-id="50c8a-112">会議後、IT ガバナンス チームの 2 人のメンバーが、クラウド計画の取り組みを調査およびサポートするための許可を求めました。</span><span class="sxs-lookup"><span data-stu-id="50c8a-112">After the meeting, two members of the IT Governance team requested permission to research and support the cloud planning efforts.</span></span> <span data-ttu-id="50c8a-113">ガバナンスの必要性とシャドウ IT を制限する機会を認識していた IT ガバナンス責任者はこのアイデアを支持しました。</span><span class="sxs-lookup"><span data-stu-id="50c8a-113">Recognizing the need for governance and an opportunity to limit shadow IT, the Director of IT Governance supported this idea.</span></span> <span data-ttu-id="50c8a-114">このような経緯で、クラウド ガバナンス チームが誕生しました。</span><span class="sxs-lookup"><span data-stu-id="50c8a-114">With that, the Cloud Governance team was born.</span></span> <span data-ttu-id="50c8a-115">このチームはこれから数か月間、クラウドの検証過程で経験する多くの失敗について、ガバナンスの観点からクリーンアップを継承していきます。</span><span class="sxs-lookup"><span data-stu-id="50c8a-115">Over the next several months, they will inherit the cleanup of many mistakes made during exploration in the cloud from a governance perspective.</span></span> <span data-ttu-id="50c8a-116">これにより "クラウド管理人" の呼び名を得ることになります。</span><span class="sxs-lookup"><span data-stu-id="50c8a-116">This will earn them the moniker of Cloud Custodians.</span></span> <span data-ttu-id="50c8a-117">今後の進化の中で、この体験におけるチームの役割は刻々と変わっていきます。</span><span class="sxs-lookup"><span data-stu-id="50c8a-117">In later evolutions, this journey will show how their roles change over time.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="50c8a-118">許容度指標</span><span class="sxs-lookup"><span data-stu-id="50c8a-118">Tolerance indicators</span></span>

<span data-ttu-id="50c8a-119">現在、リスク許容度は高く、クラウド ガバナンスへの投資意欲は低い状態です。</span><span class="sxs-lookup"><span data-stu-id="50c8a-119">The current risk tolerance is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="50c8a-120">このように、許容度指標は、時間とエネルギーの投資を誘発する早期警告システムとして機能します。</span><span class="sxs-lookup"><span data-stu-id="50c8a-120">As such, the tolerance indicators act as an early warning system to trigger the investment of time and energy.</span></span> <span data-ttu-id="50c8a-121">以下の指標が観測された場合、ガバナンス戦略を進化させることが賢明です。</span><span class="sxs-lookup"><span data-stu-id="50c8a-121">If the following indicators are observed, it would be wise to evolve the governance strategy.</span></span>

- <span data-ttu-id="50c8a-122">コスト管理:クラウドへのデプロイ規模が 1,000 資産を超えている、または毎月の支出が 10,000 米ドルを超えている。</span><span class="sxs-lookup"><span data-stu-id="50c8a-122">Cost Management: Scale of deployment exceeds 1,000 assets to the cloud, or monthly spending exceeds $10,000 USD per month.</span></span>
- <span data-ttu-id="50c8a-123">ID ベースライン:レガシまたはサード パーティの多要素認証 (MFA) 要件を持つアプリケーションが含まれている。</span><span class="sxs-lookup"><span data-stu-id="50c8a-123">Identity Baseline: Inclusion of applications with legacy or third-party multifactor authentication (MFA) requirements.</span></span>
- <span data-ttu-id="50c8a-124">セキュリティ ベースライン:定義したクラウド導入計画に保護対象データが含まれている。</span><span class="sxs-lookup"><span data-stu-id="50c8a-124">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="50c8a-125">リソースの整合性:定義したクラウド導入計画にミッションクリティカルなアプリケーションが含まれている。</span><span class="sxs-lookup"><span data-stu-id="50c8a-125">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="50c8a-126">次の手順</span><span class="sxs-lookup"><span data-stu-id="50c8a-126">Next steps</span></span>

<span data-ttu-id="50c8a-127">この企業ポリシーにより、クラウド ガバナンス チームでは、導入のための基盤になるガバナンス MVP を実装する準備が整います。</span><span class="sxs-lookup"><span data-stu-id="50c8a-127">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="50c8a-128">次の手順はこの MVP の実装です。</span><span class="sxs-lookup"><span data-stu-id="50c8a-128">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="50c8a-129">ベスト プラクティスの説明</span><span class="sxs-lookup"><span data-stu-id="50c8a-129">Best practice explained</span></span>](./best-practice-explained.md)