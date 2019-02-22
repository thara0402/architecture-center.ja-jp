---
title: CAF:アーキテクチャの決定ガイド
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド導入フレームワークのアーキテクチャの決定ガイドについて説明します。
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901958"
---
# <a name="architectural-decision-guides"></a><span data-ttu-id="8b560-103">アーキテクチャの決定ガイド</span><span class="sxs-lookup"><span data-stu-id="8b560-103">Architectural decision guides</span></span>

<span data-ttu-id="8b560-104">クラウド導入フレームワークのアーキテクチャの決定ガイドでは、クラウド ガバナンス設計ガイダンスを作成する際に役立つパターンとモデルについて説明します。</span><span class="sxs-lookup"><span data-stu-id="8b560-104">The architectural decision guides in the Cloud Adoption Framework describe patterns and models that help when creating cloud governance design guidance.</span></span> <span data-ttu-id="8b560-105">各決定ガイドでは、クラウド デプロイの中心的なインフラストラクチャ コンポーネントの 1 つに焦点を当て、特定のクラウド デプロイのシナリオをサポートすることを目的として考えられるパターンまたはモデルの一覧を紹介します。</span><span class="sxs-lookup"><span data-stu-id="8b560-105">Each decision guide focuses on one core infrastructure component of cloud deployments and lists potential patterns or models intended to support specific cloud deployment scenarios.</span></span>

<span data-ttu-id="8b560-106">組織のクラウド ガバナンスの制定を始めるときに、"アクションにつながるガバナンス体験" が提供するベースラインのロードマップを利用できます。</span><span class="sxs-lookup"><span data-stu-id="8b560-106">When you begin to establish cloud governance for your organization,  actionable governance journeys provide a baseline roadmap.</span></span> <span data-ttu-id="8b560-107">ただし、このような体験で提供される前提条件は、組織の要件や優先順位を反映していない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="8b560-107">However, these journeys make assumptions about requirements and priorities that may not reflect those of your organization.</span></span>
<span data-ttu-id="8b560-108">これらの決定ガイドは、サンプル設計ガイダンスで行われたアーキテクチャ設計の選択をお客様の要件に合わせるために役立つ代替のパターンやモデルを提供することで、サンプルのガバナンス体験を補完します。</span><span class="sxs-lookup"><span data-stu-id="8b560-108">These decision guides supplement the sample governance journeys by providing alternative patterns and models that help you align the architectural design choices made in the example design guidance with your own requirements.</span></span>

## <a name="design-guidance-categories"></a><span data-ttu-id="8b560-109">設計ガイダンスのカテゴリ</span><span class="sxs-lookup"><span data-stu-id="8b560-109">Design guidance categories</span></span>

<span data-ttu-id="8b560-110">次の各カテゴリは、すべてのクラウド デプロイの基盤となるテクノロジを表しています。</span><span class="sxs-lookup"><span data-stu-id="8b560-110">Each of the following categories represents a foundational technology of all cloud deployments.</span></span> <span data-ttu-id="8b560-111">サンプル設計体験のうち、お客様の要件に合わない各選択について、以下の関連セクションのオプションを確認し、お客様のニーズに適したパターンまたはモデルを選択してください。</span><span class="sxs-lookup"><span data-stu-id="8b560-111">For each choice in the example design journeys that doesn’t match your requirements, examine the options in the relevant section below to choose a pattern or model better suited to your needs.</span></span>

<span data-ttu-id="8b560-112">[サブスクリプション](./subscriptions/overview.md):組織の所有権、請求、管理機能に合わせて、クラウド デプロイのサブスクリプション設計とアカウント構造を計画します。</span><span class="sxs-lookup"><span data-stu-id="8b560-112">[Subscriptions](./subscriptions/overview.md): Plan your cloud deployment's subscription design and account structure to match your organization's ownership, billing, and management capabilities.</span></span>

<span data-ttu-id="8b560-113">[ID](./identity/overview.md):クラウドベースの ID サービスを既存の ID リソースと統合して、IT 環境内のアクセス制御をサポートします。</span><span class="sxs-lookup"><span data-stu-id="8b560-113">[Identity](./identity/overview.md): Integrate cloud-based identity services with your existing identity resources to support access control within your IT environment.</span></span>

<span data-ttu-id="8b560-114">[ポリシーの適用](./policy-enforcement/overview.md):クラウドにデプロイするリソースとワークロードについて組織のポリシー ルールを定義し、適用します。</span><span class="sxs-lookup"><span data-stu-id="8b560-114">[Policy Enforcement](./policy-enforcement/overview.md): Define and enforce organizational policy rules for resources and workloads that you deploy to the cloud.</span></span>

<span data-ttu-id="8b560-115">[リソースの整合性](./resource-consistency/overview.md):クラウドベースのリソースのデプロイと編成を、リソース管理とポリシー要件を強化するように確実に調整します。</span><span class="sxs-lookup"><span data-stu-id="8b560-115">[Resource Consistency](./resource-consistency/overview.md): Ensure that deployment and organization of your cloud-based resources align to enforce resource management and policy requirements.</span></span>

<span data-ttu-id="8b560-116">[リソースのタグ付け](./resource-tagging/overview.md):クラウドベースのリソースを整理して、請求モデル、クラウドの会計処理方法、管理、アクセス制御、および運用効率をサポートします。</span><span class="sxs-lookup"><span data-stu-id="8b560-116">[Resource Tagging](./resource-tagging/overview.md): Organize your cloud-based resources to support billing models, cloud accounting approaches, management, access control, and operational efficiency.</span></span> <span data-ttu-id="8b560-117">リソースのタグ付けには、一貫した体系的な名前付け規則とメタデータ スキームが必要です。</span><span class="sxs-lookup"><span data-stu-id="8b560-117">Resource tagging requires a consistent and well-organized naming and metadata scheme.</span></span>

<span data-ttu-id="8b560-118">[ソフトウェア定義ネットワーク](./software-defined-network/overview.md):迅速なデプロイと仮想化ネットワーク機能の変更を使用して、セキュリティで保護されたワークロードをクラウドにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="8b560-118">[Software Defined Networks](./software-defined-network/overview.md): Deploy secure workloads to the cloud using rapid deployment and modification of virtualized networking capabilities.</span></span> <span data-ttu-id="8b560-119">ソフトウェア定義ネットワーク (SDN) は、アジャイル ワークフローをサポートし、リソースを分離し、クラウドベースのシステムを既存の IT インフラストラクチャと統合することができます。</span><span class="sxs-lookup"><span data-stu-id="8b560-119">Software-defined networks (SDNs) can support agile workflows, isolate resources, and integrate cloud-based systems with your existing IT infrastructure.</span></span>

<span data-ttu-id="8b560-120">[暗号化](./encryption/overview.md):クラウド デプロイ内のセキュリティにおいて重要な側面である暗号化を使用して、機密データをセキュリティで保護します。</span><span class="sxs-lookup"><span data-stu-id="8b560-120">[Encryption](./encryption/overview.md): Secure your sensitive data using encryption, an important aspect of security within a cloud deployment.</span></span>

<span data-ttu-id="8b560-121">[ログとレポート](./log-and-report/overview.md):クラウドベースのリソースによって生成されたログ データを監視します。</span><span class="sxs-lookup"><span data-stu-id="8b560-121">[Logs and Reporting](./log-and-report/overview.md): Monitor log data generated by cloud-based resources.</span></span> <span data-ttu-id="8b560-122">データを分析することで、ワークロードの運用、保守、およびポリシーの適用状況の正常性に関する分析情報が得られます。</span><span class="sxs-lookup"><span data-stu-id="8b560-122">Analyzing data provides health-related insights into the operations, maintenance, and policy enforcement status of workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8b560-123">次の手順</span><span class="sxs-lookup"><span data-stu-id="8b560-123">Next steps</span></span>

<span data-ttu-id="8b560-124">サブスクリプションとアカウントがクラウド デプロイのベースとしてどのように機能するかについて学びます。</span><span class="sxs-lookup"><span data-stu-id="8b560-124">Learn how subscriptions and accounts serve as the base of a cloud deployment.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="8b560-125">サブスクリプションの設計</span><span class="sxs-lookup"><span data-stu-id="8b560-125">Subscriptions design</span></span>](subscriptions/overview.md)
