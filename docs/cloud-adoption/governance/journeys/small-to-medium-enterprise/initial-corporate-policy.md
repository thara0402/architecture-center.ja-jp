---
title: CAF:中小企業 - ガバナンス戦略の背景にある初期の企業ポリシー
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: 中小企業 - ガバナンス戦略の背景にある初期の企業ポリシー
author: BrianBlanchard
ms.openlocfilehash: 4f49d0aae2c6ab5a5c8feba669cbbb904af2773b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901361"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="dcf51-103">中小企業:ガバナンス戦略の背景にある初期の企業ポリシー</span><span class="sxs-lookup"><span data-stu-id="dcf51-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="dcf51-104">次の企業ポリシーでは、この体験の出発点である初期のガバナンス ポジションが定義されています。</span><span class="sxs-lookup"><span data-stu-id="dcf51-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="dcf51-105">この記事では、初期段階のリスク、初期ポリシー ステートメント、ポリシー ステートメントを適用するための初期プロセスを定義します。</span><span class="sxs-lookup"><span data-stu-id="dcf51-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="dcf51-106">企業ポリシーは技術文書ではありませんが、多くの技術的判断の根拠になります。</span><span class="sxs-lookup"><span data-stu-id="dcf51-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="dcf51-107">[概要](./overview.md)で説明するガバナンス MVP は、最終的にこのポリシーから派生します。</span><span class="sxs-lookup"><span data-stu-id="dcf51-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="dcf51-108">ガバナンス MVP を実装する前に、組織はその目標とビジネス リスクに基づいて企業ポリシーを策定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="dcf51-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="dcf51-109">クラウド ガバナンス チーム</span><span class="sxs-lookup"><span data-stu-id="dcf51-109">Cloud Governance team</span></span>

<span data-ttu-id="dcf51-110">この物語のクラウド管理チームは、ガバナンスのニーズを認識している 2 人のシステム管理者で構成されます。</span><span class="sxs-lookup"><span data-stu-id="dcf51-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="dcf51-111">これからの数か月間、2 人は会社のクラウド プレゼンスのガバナンスをクリーンアップするジョブを引き継ぎ、クラウド管理者という肩書を与えられるようになります。</span><span class="sxs-lookup"><span data-stu-id="dcf51-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="dcf51-112">今後の進化で、この肩書は変わる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dcf51-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="dcf51-113">許容度指標</span><span class="sxs-lookup"><span data-stu-id="dcf51-113">Tolerance indicators</span></span>

<span data-ttu-id="dcf51-114">現在、リスクの許容度は高く、クラウド ガバナンスへの投資意欲は低い状態です。</span><span class="sxs-lookup"><span data-stu-id="dcf51-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="dcf51-115">このように、許容度指標は、時間とエネルギーの投資を増やす早期警告システムとして機能します。</span><span class="sxs-lookup"><span data-stu-id="dcf51-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="dcf51-116">以下の指標が観測された場合、ガバナンス戦略を進化させる必要があります。</span><span class="sxs-lookup"><span data-stu-id="dcf51-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="dcf51-117">コスト管理:クラウドへのデプロイ規模が 100 資産を超えている、または毎月の支出が 1,000 米ドルを超えている。</span><span class="sxs-lookup"><span data-stu-id="dcf51-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="dcf51-118">セキュリティ ベースライン:定義したクラウド導入計画に保護対象データが含まれている。</span><span class="sxs-lookup"><span data-stu-id="dcf51-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="dcf51-119">リソースの整合性:定義したクラウド導入計画にミッションクリティカルなアプリケーションが含まれている。</span><span class="sxs-lookup"><span data-stu-id="dcf51-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="dcf51-120">次の手順</span><span class="sxs-lookup"><span data-stu-id="dcf51-120">Next steps</span></span>

<span data-ttu-id="dcf51-121">この企業ポリシーにより、クラウド ガバナンス チームでは、導入のための基盤になるガバナンス MVP を実装する準備が整います。</span><span class="sxs-lookup"><span data-stu-id="dcf51-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="dcf51-122">次の手順はこの MVP の実装です。</span><span class="sxs-lookup"><span data-stu-id="dcf51-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="dcf51-123">ベスト プラクティスの説明</span><span class="sxs-lookup"><span data-stu-id="dcf51-123">Best practice explained</span></span>](./best-practice-explained.md)