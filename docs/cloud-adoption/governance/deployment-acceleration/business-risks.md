---
title: 'CAF: デプロイ高速化を促進する動機とビジネス リスク'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: クラウド ガバナンス戦略の一環としての、デプロイ高速化の規範について説明します。
author: alexbuckgit
ms.openlocfilehash: 854b1fd420de605a665922e9b207e6aecbfab2f0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901278"
---
# <a name="deployment-acceleration-motivations-and-business-risks"></a><span data-ttu-id="da435-103">デプロイ高速化の動機とビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="da435-103">Deployment Acceleration motivations and business risks</span></span>

<span data-ttu-id="da435-104">この記事では、お客様が一般的に、クラウド ガバナンス戦略の中のデプロイ高速化の規範を採用する理由について説明します。</span><span class="sxs-lookup"><span data-stu-id="da435-104">This article discusses the reasons that customers typically adopt a Deployment Acceleration discipline within a cloud governance strategy.</span></span> <span data-ttu-id="da435-105">ポリシー ステートメントを追いやるビジネス リスクの例もいくつか示します。</span><span class="sxs-lookup"><span data-stu-id="da435-105">It also provides a few examples of business risks that drive policy statements.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="is-deployment-acceleration-relevant"></a><span data-ttu-id="da435-106">デプロイ高速化の関連性</span><span class="sxs-lookup"><span data-stu-id="da435-106">Is Deployment Acceleration relevant?</span></span>

<span data-ttu-id="da435-107">オンプレミス システムは、多くの場合、ベースライン イメージまたはインストール スクリプトを使用してデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="da435-107">On-premises systems are often deployed using baseline images or installation scripts.</span></span> <span data-ttu-id="da435-108">通常は追加の構成が必要で、複数の手順やユーザーの介入が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="da435-108">Additional configuration is usually necessary, which may involve multiple steps or human intervention.</span></span> <span data-ttu-id="da435-109">これらの手動プロセスではエラーが発生しやすく、多くの場合 "構成ドリフト" に至るため、時間のかかるトラブルシューティングや修復のタスクが必要になります。</span><span class="sxs-lookup"><span data-stu-id="da435-109">These manual processes are error-prone and often result in "configuration drift", requiring time-consuming troubleshooting and remediation tasks.</span></span>

<span data-ttu-id="da435-110">ほとんどの Azure リソースは、Azure portal からのデプロイと手動での構成が可能です。</span><span class="sxs-lookup"><span data-stu-id="da435-110">Most Azure resources can be deployed and configured manually via the Azure portal.</span></span> <span data-ttu-id="da435-111">管理対象のリソースがいくつかあるだけの場合は、このアプローチで十分な可能性があります。</span><span class="sxs-lookup"><span data-stu-id="da435-111">This approach may be sufficient for your needs when only have a few resources to manage.</span></span> <span data-ttu-id="da435-112">ただし、クラウド資産が拡大するにつれて、組織はクラウド リソースのデプロイを自動化し、Azure が提供するスケール、フェールオーバー、およびディザスター リカバリーの機能を活用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="da435-112">However, as your cloud estate grows, your organization should automate the deployment of your cloud resources to take advantage of the scaling, failover, and disaster recovery capabilities that Azure provides.</span></span> <span data-ttu-id="da435-113">DevOps または DevSecOps のアプローチを採用することは、多くの場合、デプロイを管理する最適な方法です。</span><span class="sxs-lookup"><span data-stu-id="da435-113">Adopting a DevOps or DevSecOps approach is often the best way to manage your deployments.</span></span>

<span data-ttu-id="da435-114">確固としたデプロイ高速化の計画によって、クラウド リソースが確実に、正しく一貫してデプロイ、更新、構成されてその状態が維持されるようにします。</span><span class="sxs-lookup"><span data-stu-id="da435-114">A robust Deployment Acceleration plan ensures that your cloud resources are deployed, updated, and configured correctly and consistently, and remain that way.</span></span> <span data-ttu-id="da435-115">デプロイ高速化戦略の成熟度は、[コスト管理戦略](../cost-management/overview.md)においても非常に重要な要因になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="da435-115">The maturity of your Deployment Acceleration strategy can also be a significant factor in your [Cost Management strategy](../cost-management/overview.md).</span></span> <span data-ttu-id="da435-116">クラウド リソースの自動プロビジョニングおよび構成を行うと、需要が少なかったり期間限定であったりするときに、スケール ダウンやリソース割り当ての解除をすることができます。そうすれば、必要なときにだけリソースの料金を支払うことができます。</span><span class="sxs-lookup"><span data-stu-id="da435-116">Automated provisioning and configuration of your cloud resources allows you to scale down or deallocate resources when demand is low or time-bound, so you only pay for resources as you need them.</span></span>

## <a name="business-risk"></a><span data-ttu-id="da435-117">ビジネス リスク</span><span class="sxs-lookup"><span data-stu-id="da435-117">Business risk</span></span>

<span data-ttu-id="da435-118">デプロイ高速化の規範は、以下のビジネス リスクへの対処を試みるものです。</span><span class="sxs-lookup"><span data-stu-id="da435-118">The Deployment Acceleration discipline attempts to address the following business risks.</span></span> <span data-ttu-id="da435-119">クラウドの導入時に、関連性がないか以下の各項目を監視します。</span><span class="sxs-lookup"><span data-stu-id="da435-119">During cloud adoption, monitor each of the following for relevance:</span></span>

- <span data-ttu-id="da435-120">**サービスの中断**。</span><span class="sxs-lookup"><span data-stu-id="da435-120">**Service disruption**.</span></span> <span data-ttu-id="da435-121">予測可能かつ繰り返し可能なデプロイ プロセスがないか、システム構成に対する管理されていない変更のため、通常業務が中断され、生産性や商機の逸失に至る可能性があります。</span><span class="sxs-lookup"><span data-stu-id="da435-121">Lack of predictable repeatable deployment processes or unmanaged changes to system configurations can disrupt normal operations and can result in lost productivity or lost business.</span></span>
- <span data-ttu-id="da435-122">**コストの超過**。</span><span class="sxs-lookup"><span data-stu-id="da435-122">**Cost overruns**.</span></span> <span data-ttu-id="da435-123">システム リソースの構成において予期しない変更があると、問題の根本原因を特定することがより困難になり、開発、運用、および保守のコストが上昇する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="da435-123">Unexpected changes in configuration of system resources can make identifying root cause of issues more difficult, raising the costs of development, operations, and maintenance.</span></span>
- <span data-ttu-id="da435-124">**組織の非効率性**。</span><span class="sxs-lookup"><span data-stu-id="da435-124">**Organizational inefficiencies**.</span></span> <span data-ttu-id="da435-125">開発、運用、およびセキュリティのチーム間に障壁があると、クラウド テクノロジの効果的な導入と、統合されたクラウド ガバナンス モデルの開発に、多数の問題が引き起こされる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="da435-125">Barriers between development, operations, and security teams can cause numerous challenges to effective adoption of cloud technologies and the development of a unified cloud governance model.</span></span>

## <a name="next-steps"></a><span data-ttu-id="da435-126">次の手順</span><span class="sxs-lookup"><span data-stu-id="da435-126">Next steps</span></span>

<span data-ttu-id="da435-127">[クラウド管理テンプレート](./template.md)を使用して、現在のクラウド導入計画によって生じる可能性の高いビジネス リスクを文書化します。</span><span class="sxs-lookup"><span data-stu-id="da435-127">Using the [Cloud Management Template](./template.md), document business risks that are likely to be introduced by the current cloud adoption plan.</span></span>

<span data-ttu-id="da435-128">現実的なビジネス リスクについての理解が確実に得られたら、リスクに対する企業の許容範囲と、その許容範囲を監視するための指標と主なメトリックについて文書化することが、次の手順となります。</span><span class="sxs-lookup"><span data-stu-id="da435-128">Once an understanding of realistic business risks is established, the next step is to document the business's tolerance for risk and the indicators and key metrics to monitor that tolerance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="da435-129">メトリック、指標、およびリスクの許容範囲</span><span class="sxs-lookup"><span data-stu-id="da435-129">Metrics, indicators, and risk tolerance</span></span>](./metrics-tolerance.md)
