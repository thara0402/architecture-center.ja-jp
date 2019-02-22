---
title: 'CAF: 設計ガイドをポリシーと整合させる。'
description: 設計ガイドをポリシーと整合させる方法
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901297"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a><span data-ttu-id="40cb8-103">設計ガイドをポリシーと整合させる方法</span><span class="sxs-lookup"><span data-stu-id="40cb8-103">How do you align design guides with policy?</span></span>

<span data-ttu-id="40cb8-104">[特定されたリスク](understanding-business-risk.md)に基づいて[クラウド ポリシーを定義](define-policy.md)したら、IT スタッフと開発者が参照する、これらのポリシーに合致した実践的なガイダンスを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="40cb8-104">After you've [defined cloud policies](define-policy.md) based on your [identified risks](understanding-business-risk.md), you'll need to generate actionable guidance that aligns with these policies for your IT staff and developers to refer to.</span></span> <span data-ttu-id="40cb8-105">クラウド ガバナンス設計ガイドの素案を作成することで、5 つのガバナンス規範それぞれに対して作成されるポリシー ステートメントに基づいた構造上、技術上、プロセス上の選択肢を指定することができます。</span><span class="sxs-lookup"><span data-stu-id="40cb8-105">Drafting a cloud governance design guide allows you to specify specific structural, technological, and process choices based on the policy statements you generated for each of the five governance disciplines.</span></span>

<span data-ttu-id="40cb8-106">クラウド ガバナンス設計ガイドでは、実際のポリシー要件に最適なクラウド デプロイのコア インフラストラクチャ コンポーネントそれぞれについて、アーキテクチャの選択肢と設計パターンを確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="40cb8-106">A cloud governance design guide should establish the architecture choices and design patterns for each of the core infrastructure components of cloud deployments that best meet your policy requirements.</span></span> <span data-ttu-id="40cb8-107">これらと共に、こうした設計上の各決定事項をサポートするテクノロジ、ツール、およびプロセスの概要説明を含める必要があります。</span><span class="sxs-lookup"><span data-stu-id="40cb8-107">Alongside these you should provide a high-level explanation of the technology, tools, and processes that will support each of these design decisions.</span></span>

<span data-ttu-id="40cb8-108">リスク分析とポリシー ステートメントは、ある程度クラウド プラットフォームに依存しない場合もありますが、設計ガイドでは、IT および開発部門が必要とするプラットフォーム固有の実装の詳細を提供する必要があります。</span><span class="sxs-lookup"><span data-stu-id="40cb8-108">Although your risk analysis and policy statements may, to some degree, be cloud platform agnostic, your design guide should provide platform-specific implementation details that your IT and dev.</span></span> <span data-ttu-id="40cb8-109">設計上の決定を下し、ガイダンスを提供するときには、選択したプラットフォームのアーキテクチャ、ツール、および、機能に焦点を合わせます。</span><span class="sxs-lookup"><span data-stu-id="40cb8-109">Focus on the architecture, tools, and features of your chosen platform when making design decision and providing guidance.</span></span>

<span data-ttu-id="40cb8-110">クラウド設計ガイドでは、各インフラストラクチャ コンポーネントに関連する技術的詳細の一部を考慮に入れる必要がありますが、ガイドは広範囲にわたる技術ドキュメントや仕様書となるものではありません。</span><span class="sxs-lookup"><span data-stu-id="40cb8-110">While cloud design guides should take into account some of the technical details associated with each infrastructure component, they are not meant to be extensive technical documents or specifications.</span></span> <span data-ttu-id="40cb8-111">ガイドでは、すべてのポリシー ステートメントが網羅されていて、スタッフが容易に理解して参照できる形式で、設計の決定内容が明確に書かれていることを確認します。</span><span class="sxs-lookup"><span data-stu-id="40cb8-111">Make sure your guides address all of your policy statements and clearly state design decisions in a format easy for staff to understand and reference.</span></span>

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a><span data-ttu-id="40cb8-112">実践的ガバナンス体験の利用</span><span class="sxs-lookup"><span data-stu-id="40cb8-112">Using the actionable governance journeys</span></span>

<span data-ttu-id="40cb8-113">クラウド導入のために Azure プラットフォームを使用する予定の場合は、CAF が、CAF ガバナンス モデルの増分型アプローチについて説明している[ガバナンス体験](../journeys/overview.md)を提供しています。</span><span class="sxs-lookup"><span data-stu-id="40cb8-113">If you're planning to use the Azure platform for your cloud adoption, the CAF provides [governance journeys](../journeys/overview.md) illustrating the incremental approach of the CAF governance model.</span></span> <span data-ttu-id="40cb8-114">ストーリー型のこれらの体験では、ビジネス リスク、許容の要件、ポリシー ステートメントを含む一般的な導入シナリオが取り上げられていています。これらが、ガバナンスにおける "実用最小限の製品" (MVP) を作り出すことになります。</span><span class="sxs-lookup"><span data-stu-id="40cb8-114">These narrative journeys cover a range of common adoption scenarios, including the business risks, tolerance requirements, and policy statements that went into creating a governance minimum viable product (MVP).</span></span> <span data-ttu-id="40cb8-115">これらの体験は、実世界で顧客が経験する Azure でのクラウド導入プロセスの全体を表しています。</span><span class="sxs-lookup"><span data-stu-id="40cb8-115">These journeys represent a synthesis of real-world customer experience of the cloud adoption process in Azure.</span></span>

<span data-ttu-id="40cb8-116">どのクラウド導入にも固有の目標、優先度、課題がありますが、これらのサンプルは、実際のポリシーをガイダンスに移し替えるための適切なテンプレートとなるはずです。</span><span class="sxs-lookup"><span data-stu-id="40cb8-116">While every cloud adoption has unique goals, priorities, and challenges, these samples should provide a good template for converting your policy into guidance.</span></span> <span data-ttu-id="40cb8-117">開始点として、実際の状況に最も近いシナリオを選択し、ポリシーの具体的なニーズに合うように変更します。</span><span class="sxs-lookup"><span data-stu-id="40cb8-117">Pick the closest scenario to your situation as a starting point, and mold it to fit your specific policy needs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="40cb8-118">次の手順</span><span class="sxs-lookup"><span data-stu-id="40cb8-118">Next steps</span></span>

<span data-ttu-id="40cb8-119">設計ガイダンスを用意したので、ポリシー コンプライアンスを確保するためのポリシー準拠プロセスを確立します。</span><span class="sxs-lookup"><span data-stu-id="40cb8-119">With design guidance in place, establish policy adherence processes to ensure policy compliance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="40cb8-120">ポリシー準拠プロセス</span><span class="sxs-lookup"><span data-stu-id="40cb8-120">Policy adherence processes</span></span>](processes.md)