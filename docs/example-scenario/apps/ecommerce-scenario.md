---
title: Azure の eコマース フロントエンド
description: Azure で eコマース サイトをホストします。
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 7baaf4d2986a00ab72b60a540bcd9d864893b109
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610823"
---
# <a name="an-e-commerce-front-end-on-azure"></a><span data-ttu-id="b1c90-103">Azure の eコマース フロントエンド</span><span class="sxs-lookup"><span data-stu-id="b1c90-103">An e-commerce front end on Azure</span></span>

<span data-ttu-id="b1c90-104">このサンプル シナリオでは、Azure の提供するサービスとしてのプラットフォーム (PaaS) ツールを使用した eコマース フロントエンドの実装について説明します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-104">This example scenario walks you through an implementation of an e-commerce front end using Azure platform as a service (PaaS) tools.</span></span> <span data-ttu-id="b1c90-105">eコマース Web サイトの多くが、季節性および時期的なトラフィックの変動に直面します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-105">Many e-commerce websites face seasonality and traffic variability over time.</span></span> <span data-ttu-id="b1c90-106">PaaS ツールを利用すれば、予想通りでも、予想外であっても、商品やサービスの需要が急増したときに、顧客や取引の増加に自動的に対処できます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-106">When demand for your products or services takes off, whether predictably or unpredictably, using PaaS tools will allow you to handle more customers and more transactions automatically.</span></span> <span data-ttu-id="b1c90-107">また、このシナリオでは、使用した容量分だけ支払うため、経済的にもクラウドの恩恵を受けられます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-107">Additionally, this scenario takes advantage of cloud economics by paying only for the capacity you use.</span></span>

<span data-ttu-id="b1c90-108">このドキュメントは、サンプル eコマース アプリケーションしての、*Relecloud Concerts* というオンライン コンサート発券プラットフォームをデプロイする際にまとめて使用されるさまざまな Azure PaaS コンポーネントと考慮事項について説明します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-108">This document will help you will learn about various Azure PaaS components and considerations used to bring together to deploy a sample e-commerce application, *Relecloud Concerts*, an online concert ticketing platform.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="b1c90-109">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="b1c90-109">Relevant use cases</span></span>

<span data-ttu-id="b1c90-110">その他の関連するユース ケース:</span><span class="sxs-lookup"><span data-stu-id="b1c90-110">Other relevant use cases include:</span></span>

* <span data-ttu-id="b1c90-111">さまざまなタイミングでユーザーの急増に対処できるように弾力性のあるスケーリングを必要とするアプリケーションを構築する。</span><span class="sxs-lookup"><span data-stu-id="b1c90-111">Building an application that needs elastic scale to handle bursts of users at different times.</span></span>
* <span data-ttu-id="b1c90-112">世界中のさまざまな Azure リージョンで高い可用性で運用されるよう設計されたアプリケーションを構築する。</span><span class="sxs-lookup"><span data-stu-id="b1c90-112">Building an application that is designed to operate at high availability in different Azure regions around the world.</span></span>

## <a name="architecture"></a><span data-ttu-id="b1c90-113">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="b1c90-113">Architecture</span></span>

![eコマース アプリケーションのサンプル シナリオ アーキテクチャ][architecture]

<span data-ttu-id="b1c90-115">このシナリオでは、eコマース サイトからのチケット購入に対応します。シナリオのデータ フローを次に示します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-115">This scenario covers purchasing tickets from an e-commerce site, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="b1c90-116">Azure Traffic Manager により、ユーザーの要求が、Azure App Service でホストされている eコマース サイトにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-116">Azure Traffic Manager routes a user's request to the e-commerce site hosted in Azure App Service.</span></span>
2. <span data-ttu-id="b1c90-117">Azure CDN は、静的なイメージとコンテンツをユーザーに提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-117">Azure CDN serves static images and content to the user.</span></span>
3. <span data-ttu-id="b1c90-118">ユーザーが Azure Active Directory B2C テナントを介してアプリケーションにサインインします。</span><span class="sxs-lookup"><span data-stu-id="b1c90-118">User signs in to the application through an Azure Active Directory B2C tenant.</span></span>
4. <span data-ttu-id="b1c90-119">ユーザーが Azure Search を使用してコンサートを検索します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-119">User searches for concerts using Azure Search.</span></span>
5. <span data-ttu-id="b1c90-120">Web サイトによって、Azure SQL Database からコンサートの詳細がプルされます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-120">Web site pulls concert details from Azure SQL Database.</span></span> 
6. <span data-ttu-id="b1c90-121">Web サイトから、Blob Storage にある購入済みチケットの画像を参照します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-121">Web site refers to purchased ticket images in Blob Storage.</span></span>
7. <span data-ttu-id="b1c90-122">データベース クエリの結果が、パフォーマンス向上のため、Azure Redis Cache にキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-122">Database query results are cached in Azure Redis Cache for better performance.</span></span>
8. <span data-ttu-id="b1c90-123">ユーザーが送信したチケット注文とコンサート レビューがキューに配置されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-123">User submits ticket orders and concert reviews, which are placed in the queue.</span></span>
9. <span data-ttu-id="b1c90-124">Azure Functions によって、注文の支払いとコンサート レビューが処理されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-124">Azure Functions processes order payment and concert reviews.</span></span>
10. <span data-ttu-id="b1c90-125">Cognitive Services ではコンサート レビューが分析され、センチメント (肯定的または否定的) が判別されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-125">Cognitive services provide an analysis of the concert review to determine the sentiment (positive or negative).</span></span>
11. <span data-ttu-id="b1c90-126">Application Insights が、Web アプリケーションの正常性を監視するためのパフォーマンス メトリックを提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-126">Application Insights provides performance metrics for monitoring the health of the web application.</span></span>

### <a name="components"></a><span data-ttu-id="b1c90-127">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="b1c90-127">Components</span></span>

* <span data-ttu-id="b1c90-128">[Azure CDN][docs-cdn] が、待機時間を減らすために、ユーザーに近い場所から、キャッシュされた静的なコンテンツを提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-128">[Azure CDN][docs-cdn] delivers static, cached content from locations close to users to reduce latency.</span></span>
* <span data-ttu-id="b1c90-129">[Azure Traffic Manager][docs-traffic-manager] によって、さまざまな Azure リージョンにおけるサービス エンドポイントのユーザー トラフィックの分散が制御されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-129">[Azure Traffic Manager][docs-traffic-manager] controls the distribution of user traffic for service endpoints in different Azure regions.</span></span>
* <span data-ttu-id="b1c90-130">[App Services - Web Apps][docs-webapps] により Web アプリケーションがホストされ、自動スケーリングと高可用性が可能になります。インフラストラクチャの管理は不要です。</span><span class="sxs-lookup"><span data-stu-id="b1c90-130">[App Services - Web Apps][docs-webapps] hosts web applications allowing autoscale and high availability without having to manage infrastructure.</span></span>
* <span data-ttu-id="b1c90-131">[Azure Active Directory B2C][docs-b2c] は ID 管理サービスです。このサービスを使用すると、アプリケーションでの顧客のサインアップ、サインイン、およびプロファイル管理の方法をカスタマイズし、制御できます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-131">[Azure Active Directory - B2C][docs-b2c] is an identity management service that enables customization and control over how customers sign up, sign in, and manage their profiles in an application.</span></span>
* <span data-ttu-id="b1c90-132">[Storage キュー][docs-storage-queues]には、アプリケーションからアクセスできるキュー メッセージが多数格納されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-132">[Storage Queues][docs-storage-queues] stores large numbers of queue messages that can be accessed by an application.</span></span>
* <span data-ttu-id="b1c90-133">[Functions][docs-functions] は、インフラストラクチャを管理せずに、アプリケーションをオンデマンドで実行できるようにするサーバーレス コンピューティング オプションです。</span><span class="sxs-lookup"><span data-stu-id="b1c90-133">[Functions][docs-functions] are serverless compute options that allow applications to run on-demand without having to manage infrastructure.</span></span>
* <span data-ttu-id="b1c90-134">[Cognitive Services - 感情分析][docs-sentiment-analysis]は機械学習 API を使用して、開発者が、アプリケーションに感情認識や映像検出、顔認識、音声認識、視覚認識、音声理解、言語理解など、インテリジェントな機能を簡単に追加できるようにします。</span><span class="sxs-lookup"><span data-stu-id="b1c90-134">[Cognitive Services - Sentiment Analysis][docs-sentiment-analysis] uses machine learning APIs and enables developers to easily add intelligent features – such as emotion and video detection; facial, speech, and vision recognition; and speech and language understanding – into applications.</span></span>
* <span data-ttu-id="b1c90-135">[Azure Search][docs-search] はサービスとしての検索クラウド ソリューションで、多彩な検索機能を、Web、モバイル、およびエンタープライズ アプリケーションのプライベートな異種コンテンツに提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-135">[Azure Search][docs-search] is a search-as-a-service cloud solution that provides a rich search experience over private, heterogenous content in web, mobile, and enterprise applications.</span></span>
* <span data-ttu-id="b1c90-136">[Storage Blob][docs-storage-blobs] は、テキスト データ、バイナリ データなど、大量の非構造化データを格納するために最適化されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-136">[Storage Blobs][docs-storage-blobs] are optimized to store large amounts of unstructured data, such as text or binary data.</span></span>
* <span data-ttu-id="b1c90-137">[Redis Cache][docs-redis-cache] は、アクセス頻度が高いデータを、アプリケーションに近い場所にある高速ストレージに一時的にコピーすることで、バックエンドのデータストアに大きく依存するシステムのパフォーマンスとスケーラビリティを向上させます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-137">[Redis Cache][docs-redis-cache] improves the performance and scalability of systems that rely heavily on back-end data stores by temporarily copying frequently accessed data to fast storage located close to the application.</span></span>
* <span data-ttu-id="b1c90-138">[SQL Database][docs-sql-database] は、リレーショナル データ、JSON、空間、XML などの構造をサポートする、Microsoft Azure における汎用リレーショナル データベース管理サービスです。</span><span class="sxs-lookup"><span data-stu-id="b1c90-138">[SQL Database][docs-sql-database] is a general-purpose relational database managed service in Microsoft Azure that supports structures such as relational data, JSON, spatial, and XML.</span></span>
* <span data-ttu-id="b1c90-139">[Application Insights][docs-application-insights] は、パフォーマンスやユーザビリティを継続的に向上させるうえで役立つように設計されています。これを実現するために、アプリでのユーザーの操作内容を把握しやすいように、組み込みの分析ツールを使用してパフォーマンスの異常が自動検出されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-139">[Application Insights][docs-application-insights] is designed to help you continuously improve performance and usability by automatically detecting performance anomalies through built-in analytics tools to help understand what users do with an app.</span></span>

### <a name="alternatives"></a><span data-ttu-id="b1c90-140">代替手段</span><span class="sxs-lookup"><span data-stu-id="b1c90-140">Alternatives</span></span>

<span data-ttu-id="b1c90-141">他にもさまざまなテクノロジを、顧客向けの大規模な eコマース用アプリケーションの構築に使用できます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-141">Many other technologies are available for building a customer facing application focused on e-commerce at scale.</span></span> <span data-ttu-id="b1c90-142">これらは、アプリケーションのフロントエンドとデータ層の両方を対象としています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-142">These cover both the front end of the application as well as the data tier.</span></span>

<span data-ttu-id="b1c90-143">Web 層と機能に関するその他のオプションは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="b1c90-143">Other options for the web tier and functions include:</span></span>

* <span data-ttu-id="b1c90-144">[Service Fabric][docs-service-fabric] - 細かな制御でクラスター全体にデプロイして実行することでメリットを得られる分散コンポーネントの構築に重点を置いたプラットフォーム。</span><span class="sxs-lookup"><span data-stu-id="b1c90-144">[Service Fabric][docs-service-fabric] - A platform focused around building distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="b1c90-145">Service Fabric を使ってコンテナーをホストすることもできます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-145">Service Fabric can also be used to host containers.</span></span>
* <span data-ttu-id="b1c90-146">[Azure Kubernetes Service][docs-kubernetes-service] - マイクロサービス アーキテクチャの実装として使用できる、コンテナー ベースのソリューションを構築およびデプロイするためのプラットフォーム。</span><span class="sxs-lookup"><span data-stu-id="b1c90-146">[Azure Kubernetes Service][docs-kubernetes-service] - A platform for building and deploying container-based solutions that can be used as one implementation of a microservices architecture.</span></span> <span data-ttu-id="b1c90-147">これにより、アプリケーションのさまざまなコンポーネントの俊敏性が実現し、オンデマンドで個別にスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-147">This allows for agility of different components of the application to be able to scale independently on demand.</span></span>
* <span data-ttu-id="b1c90-148">[Azure Container Instances][docs-container-instances] - コンテナーを短いライフサイクルですばやくデプロイし、実行する手段。</span><span class="sxs-lookup"><span data-stu-id="b1c90-148">[Azure Container Instances][docs-container-instances] - A way of quickly deploying and running containers with a short lifecycle.</span></span> <span data-ttu-id="b1c90-149">このコンテナーは、メッセージの処理や計算の実行など、簡単な処理ジョブを実行するためにデプロイされ、完了するとすぐにプロビジョニングが解除されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-149">Containers here are deployed to run a quick processing job such as processing a message or performing a calculation and then deprovisioned as soon as they are complete.</span></span>
* <span data-ttu-id="b1c90-150">[Service Bus][service-bus] は、Storage キューの代わりに使用できます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-150">[Service Bus][service-bus] could be used in place of Storage Queue's.</span></span>

<span data-ttu-id="b1c90-151">データ層の他のオプションを次に示します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-151">Other options for the data tier include:</span></span>

* <span data-ttu-id="b1c90-152">[Cosmos DB](/azure/cosmos-db/introduction): Microsoft のグローバル分散型マルチモデル データベースです。</span><span class="sxs-lookup"><span data-stu-id="b1c90-152">[Cosmos DB](/azure/cosmos-db/introduction): Microsoft's globally distributed, multi-model database.</span></span> <span data-ttu-id="b1c90-153">このサービスは、Mongo DB、Cassandra、Graph データ、シンプルなテーブル ストレージなど、他のデータ モデルを実行するためのプラットフォームを提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-153">This service provides a platform to run other data models such as Mongo DB, Cassandra, Graph data, or simple table storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="b1c90-154">考慮事項</span><span class="sxs-lookup"><span data-stu-id="b1c90-154">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="b1c90-155">可用性</span><span class="sxs-lookup"><span data-stu-id="b1c90-155">Availability</span></span>

* <span data-ttu-id="b1c90-156">クラウド アプリケーションを構築するときは、[可用性のための標準的な設計パターン][design-patterns-availability]の利用を検討してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-156">Consider leveraging the [typical design patterns for availability][design-patterns-availability] when building your cloud application.</span></span>
* <span data-ttu-id="b1c90-157">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]で可用性に関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="b1c90-157">Review the availability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>
* <span data-ttu-id="b1c90-158">可用性に関する追加の考慮事項については、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-158">For additional considerations concerning availability, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="b1c90-159">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="b1c90-159">Scalability</span></span>

* <span data-ttu-id="b1c90-160">クラウド アプリケーションを構築するときは、[スケーラビリティのための標準的な設計パターン][design-patterns-scalability]に注意してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-160">When building a cloud application be aware of the [typical design patterns for scalability][design-patterns-scalability].</span></span>
* <span data-ttu-id="b1c90-161">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]でスケーラビリティに関する考慮事項を確認します</span><span class="sxs-lookup"><span data-stu-id="b1c90-161">Review the scalability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>
* <span data-ttu-id="b1c90-162">スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-162">For other scalability topics, see the [scalability checklist][scalability] available in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="b1c90-163">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="b1c90-163">Security</span></span>

* <span data-ttu-id="b1c90-164">必要に応じて、[セキュリティのための標準的な設計パターン][design-patterns-security]を利用することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-164">Consider leveraging the [typical design patterns for security][design-patterns-security] where appropriate.</span></span>
* <span data-ttu-id="b1c90-165">適切な [App Service Web アプリケーションのリファレンス アーキテクチャ][app-service-reference-architecture]のセキュリティに関する考慮事項を確認します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-165">Review the security considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture].</span></span>
* <span data-ttu-id="b1c90-166">[セキュリティで保護された開発ライフサイクル][secure-development] プロセスに従って、開発コストを削減しながら、開発者がより安全なソフトウェアを構築し、セキュリティ コンプライアンス要件に対応できるようにします。</span><span class="sxs-lookup"><span data-stu-id="b1c90-166">Consider following a [secure development lifecycle][secure-development] process to help developers build more secure software and address security compliance requirements while reducing development cost.</span></span>
* <span data-ttu-id="b1c90-167">[Azure PCI DSS コンプライアンス][pci-dss-blueprint]を実現するためのブループリント アーキテクチャを確認してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-167">Review the blueprint architecture for [Azure PCI DSS compliance][pci-dss-blueprint].</span></span>

### <a name="resiliency"></a><span data-ttu-id="b1c90-168">回復性</span><span class="sxs-lookup"><span data-stu-id="b1c90-168">Resiliency</span></span>

* <span data-ttu-id="b1c90-169">アプリケーションの一部が使用できない場合は、[サーキット ブレーカー パターン][circuit-breaker]を利用して、グレースフル エラー処理を提供することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-169">Consider leveraging the [circuit breaker pattern][circuit-breaker] to provide graceful error handling should one part of the application not be available.</span></span>
* <span data-ttu-id="b1c90-170">[回復性のための標準的な設計パターン][design-patterns-resiliency]を確認し、必要に応じて、これらを実装することを検討します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-170">Review the [typical design patterns for resiliency][design-patterns-resiliency] and consider implementing these where appropriate.</span></span>
* <span data-ttu-id="b1c90-171">Azure アーキテクチャ センターでは、多数の [App Service に関する推奨プラクティス][resiliency-app-service]を確認できます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-171">You can find a number of [recommended practices for App Service][resiliency-app-service] in the Azure Architecture Center.</span></span>
* <span data-ttu-id="b1c90-172">データ層にはアクティブ [geo レプリケーション][sql-geo-replication]を、イメージおよびキューには [geo 冗長][storage-geo-redudancy]ストレージを使用することを検討します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-172">Consider using active [geo-replication][sql-geo-replication] for the data tier and [geo-redundant][storage-geo-redudancy] storage for images and queues.</span></span>
* <span data-ttu-id="b1c90-173">[回復性][resiliency]の詳細については、Azure アーキテクチャ センターの関連記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b1c90-173">For a deeper discussion on [resiliency][resiliency], see the relevant article in the Azure Architecture Center.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="b1c90-174">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="b1c90-174">Deploy the scenario</span></span>

<span data-ttu-id="b1c90-175">このシナリオをデプロイするには、こちらの[ステップ バイ ステップのチュートリアル][end-to-end-walkthrough]に従います。このチュートリアルでは、各コンポーネントを手動でデプロイする方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-175">To deploy this scenario, you can follow this [step-by-step tutorial][end-to-end-walkthrough] demonstrating how to manually deploy each component.</span></span> <span data-ttu-id="b1c90-176">また、このチュートリアルでは、シンプルなチケット購入アプリケーションを実行する .NET サンプル アプリケーションも提供します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-176">This tutorial also provides a .NET sample application that runs a simple ticket purchasing application.</span></span> <span data-ttu-id="b1c90-177">さらに、ほとんどの Azure リソースのデプロイを自動化する Resource Manager テンプレートもあります。</span><span class="sxs-lookup"><span data-stu-id="b1c90-177">Additionally, there is a Resource Manager template to automate the deployment of most of the Azure resources.</span></span>

## <a name="pricing"></a><span data-ttu-id="b1c90-178">価格</span><span class="sxs-lookup"><span data-stu-id="b1c90-178">Pricing</span></span>

<span data-ttu-id="b1c90-179">このシナリオの実行コストを調べてください。すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-179">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="b1c90-180">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-180">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="b1c90-181">取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-181">We have provided three sample cost profiles based on amount of traffic you expect to get:</span></span>

* <span data-ttu-id="b1c90-182">[Small][small-pricing]: この価格例は、最小の運用レベル インスタンスの構築に必要なコンポーネントを表します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-182">[Small][small-pricing]: This pricing example represents the components necessary to build the out for a minimum production level instance.</span></span> <span data-ttu-id="b1c90-183">ここでは、1 か月あたり数千人の少人数のユーザーを想定しています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-183">Here we are assuming a small number of users, numbering only in a few thousand per month.</span></span> <span data-ttu-id="b1c90-184">アプリでは標準の Web アプリの単一インスタンスが使用されており、自動スケーリングを有効にするにはこれで十分です。</span><span class="sxs-lookup"><span data-stu-id="b1c90-184">The app is using a single instance of a standard web app that will be enough to enable autoscaling.</span></span> <span data-ttu-id="b1c90-185">その他のコンポーネントはそれぞれ Basic レベルにスケーリングされ、最小限のコストで、SLA をサポートし、運用レベルのワークロードを処理できるだけの容量が確保されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-185">Each of the other components are scaled to a basic tier that will allow for a minimum amount of cost but still ensure that there is SLA support and enough capacity to handle a production level workload.</span></span>
* <span data-ttu-id="b1c90-186">[Medium][medium-pricing]: この価格例は、中規模サイズのデプロイの指標となるコンポーネントを表します。</span><span class="sxs-lookup"><span data-stu-id="b1c90-186">[Medium][medium-pricing]: This pricing example represents the components indicative of a moderate size deployment.</span></span> <span data-ttu-id="b1c90-187">ここでは、1 か月間に約 100,000 人のユーザーがシステムを使用することを推定しています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-187">Here we estimate approximately 100,000 users using the system over the course of a month.</span></span> <span data-ttu-id="b1c90-188">予想されるトラフィックは、中程度の Standard レベルによって単一アプリ サービス インスタンスで処理されます。</span><span class="sxs-lookup"><span data-stu-id="b1c90-188">The expected traffic is handled in a single app service instance with a moderate standard tier.</span></span> <span data-ttu-id="b1c90-189">また、中レベルのコグニティブおよび検索サービスが計算ツールに追加されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-189">Additionally, moderate tiers of cognitive and search services are added to the calculator.</span></span>
* <span data-ttu-id="b1c90-190">[Large][large-pricing]: この価格例は、高度のスケーリングを想定したアプリケーションを表します。このアプリケーションでは、1 か月あたり数百万人のユーザーが数テラバイトのデータをやり取りします。</span><span class="sxs-lookup"><span data-stu-id="b1c90-190">[Large][large-pricing]: This pricing example represents an application meant for high scale, at the order of millions of users per month moving terabytes of data.</span></span> <span data-ttu-id="b1c90-191">このレベルの使用状況では、トラフィック マネージャーがアクセスする複数のリージョンにデプロイされた、高パフォーマンスの Premium レベルの Web アプリが必要です。</span><span class="sxs-lookup"><span data-stu-id="b1c90-191">At this level of usage high performance, premium tier web apps deployed in multiple regions fronted by traffic manager is required.</span></span> <span data-ttu-id="b1c90-192">データは、ストレージ、データベース、および CDN で構成され、これらはテラバイト データ用に構成されています。</span><span class="sxs-lookup"><span data-stu-id="b1c90-192">Data consists of the following: storage, databases, and CDN, are configured for terabytes of data.</span></span>

## <a name="related-resources"></a><span data-ttu-id="b1c90-193">関連リソース</span><span class="sxs-lookup"><span data-stu-id="b1c90-193">Related resources</span></span>

* <span data-ttu-id="b1c90-194">[複数リージョンの Web アプリケーション向けリファレンス アーキテクチャ][multi-region-web-app]</span><span class="sxs-lookup"><span data-stu-id="b1c90-194">[Reference Architecture for Multi-Region Web Application][multi-region-web-app]</span></span>
* <span data-ttu-id="b1c90-195">[eShopOnContainers の参照用の例][microservices-ecommerce]</span><span class="sxs-lookup"><span data-stu-id="b1c90-195">[eShop on Containers Reference Example][microservices-ecommerce]</span></span>

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
