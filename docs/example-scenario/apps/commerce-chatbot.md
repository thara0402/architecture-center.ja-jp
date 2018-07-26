---
title: Azure でのホテル予約用の会話型チャットボット
description: Azure Bot Service、Cognitive Services と LUIS、Azure SQL Database、Application Insights を使用して、商取引アプリケーション用の会話型チャットボットを構築するための実証済みのシナリオ。
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: b664faf20d806824c2581346aaa592b0d74207da
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060865"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a><span data-ttu-id="02e2e-103">Azure でのホテル予約用の会話型チャットボット</span><span class="sxs-lookup"><span data-stu-id="02e2e-103">Conversational chatbot for hotel reservations on Azure</span></span>

<span data-ttu-id="02e2e-104">このシナリオ例は、会話型チャットボットをアプリケーションに統合する必要がある企業に適用されます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-104">This example scenario is applicable to businesses that need integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="02e2e-105">このシナリオでは、顧客が Web アプリケーションまたはモバイル アプリケーションを使用して空室状況を確認し、宿泊施設を予約できる C# チャットボットをホテル チェーンで使用します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-105">In this scenario, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="02e2e-106">シナリオ例では、顧客がホテルの空室状況を確認して部屋を予約する方法、レストランのテイクアウト メニューを表示して料理を注文をする方法、写真を検索してプリントを注文する方法を提供します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-106">Example scenarios include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="02e2e-107">従来、企業は顧客の要求に対応するために、カスタマー サービス エージェントを雇用してトレーニングする必要があり、顧客は担当者が支援できるようになるまで待つ必要がありました。</span><span class="sxs-lookup"><span data-stu-id="02e2e-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="02e2e-108">Bot Service と Language Understanding Service や Speech API サービスなどの Azure サービスを使用することで、企業は自動化されたスケーラブルなボットを使って顧客を支援し、注文や予約を処理できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="02e2e-109">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="02e2e-109">Related use cases</span></span>

<span data-ttu-id="02e2e-110">次のユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-110">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="02e2e-111">レストランのテイクアウト メニューを表示し、料理を注文する</span><span class="sxs-lookup"><span data-stu-id="02e2e-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="02e2e-112">ホテルの空室状況を確認し、部屋を予約する</span><span class="sxs-lookup"><span data-stu-id="02e2e-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="02e2e-113">利用可能な写真を検索し、プリントを注文する</span><span class="sxs-lookup"><span data-stu-id="02e2e-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="02e2e-114">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="02e2e-114">Architecture</span></span>

![会話型チャットボットに関与する Azure コンポーネントのアーキテクチャの概要][architecture]

<span data-ttu-id="02e2e-116">このシナリオでは、ホテルのコンシェルジュとして機能する会話型ボットに対応できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-116">This scenario covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="02e2e-117">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="02e2e-117">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="02e2e-118">顧客がモバイル アプリまたは Web アプリを使用してチャットボットにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="02e2e-119">Azure Active Directory B2C (Business 2 Customer) を使用してユーザーが認証されます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="02e2e-120">ユーザーは Bot Service と対話して、ホテルの空室状況に関する情報を要求します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="02e2e-121">Cognitive Services が自然言語の要求を処理して、顧客のコミュニケーションを理解します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="02e2e-122">ユーザーが結果に満足すると、ボットが SQL Database で顧客の予約を追加または更新します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="02e2e-123">Application Insights がプロセス全体を通じてランタイム テレメトリを収集し、ボットのパフォーマンスと使用状況の情報を提供して DevOps チームを支援します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="02e2e-124">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="02e2e-124">Components</span></span>

* <span data-ttu-id="02e2e-125">[Azure Active Directory][aad-docs]: Microsoft が提供する、マルチテナントに対応したクラウドベースのディレクトリおよび ID 管理サービスです。</span><span class="sxs-lookup"><span data-stu-id="02e2e-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="02e2e-126">Azure AD では、Google、Facebook、Microsoft アカウントなどの外部 ID を使用して個人を識別できる B2C コネクタがサポートされています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="02e2e-127">[App Service][appservice-docs]: インフラストラクチャを管理することなく、任意のプログラミング言語で Web アプリケーションを構築し、ホストすることができます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="02e2e-128">[Bot Service][botservice-docs]: インテリジェント ボットの構築、テスト、デプロイ、管理を行うツールを提供します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="02e2e-129">[Cognitive Services][cognitive-docs]: 自然なコミュニケーション手段を通じて、見る、聞く、話す、理解する、ユーザーのニーズを解釈することが可能なインテリジェントなアルゴリズムを使用できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="02e2e-130">[SQL Database][sqldatabase-docs]: SQL Server エンジンの互換性を提供するフル マネージド リレーショナル クラウド データベース サービスです。</span><span class="sxs-lookup"><span data-stu-id="02e2e-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="02e2e-131">[Application Insights][appinsights-docs]: チャットボットなどのアプリケーションのパフォーマンスを監視できる、拡張可能な Application Performance Management (APM) サービスです。</span><span class="sxs-lookup"><span data-stu-id="02e2e-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="02e2e-132">代替手段</span><span class="sxs-lookup"><span data-stu-id="02e2e-132">Alternatives</span></span>

* <span data-ttu-id="02e2e-133">[Microsoft Speech API][speech-api] を使用して、顧客とボット間のインターフェイスを変更できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="02e2e-134">[QnA Maker][qna-maker] を使用すると、FAQ などの半構造化コンテンツから、ボットに知識をすばやく追加できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="02e2e-135">[Translator Text][translator] は、ボットに多言語サポートを簡単に追加するために検討できるサービスです。</span><span class="sxs-lookup"><span data-stu-id="02e2e-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="02e2e-136">考慮事項</span><span class="sxs-lookup"><span data-stu-id="02e2e-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="02e2e-137">可用性</span><span class="sxs-lookup"><span data-stu-id="02e2e-137">Availability</span></span>

<span data-ttu-id="02e2e-138">このシナリオでは、Azure SQL Database を使用して顧客の予約を格納します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-138">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="02e2e-139">SQL Database には、ゾーン冗長データベース、フェールオーバー グループ、geo レプリケーションなどの機能が用意されています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="02e2e-140">詳細については、[Azure SQL Database の可用性機能][sqlavailability-docs]に関するセクションをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="02e2e-141">可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-141">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="02e2e-142">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="02e2e-142">Scalability</span></span>

<span data-ttu-id="02e2e-143">このシナリオでは、Azure App Service を使用します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-143">This scenario uses Azure App Service.</span></span> <span data-ttu-id="02e2e-144">App Service により、ボットを実行するインスタンスの数を自動的にスケーリングできます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="02e2e-145">この機能を使用することで、Web アプリケーションやチャットボットに対する顧客の要求に対応できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="02e2e-146">自動スケールの詳細については、アーキテクチャ センターの[自動スケールのベスト プラクティス][autoscaling]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the architecture center.</span></span>

<span data-ttu-id="02e2e-147">スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="02e2e-148">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="02e2e-148">Security</span></span>

<span data-ttu-id="02e2e-149">このシナリオでは、Azure Active Directory B2C (Business 2 Consumer) を使用してユーザーを認証します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-149">This scenario uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="02e2e-150">AAD B2C を使用することで、機密性の高い顧客のアカウント情報や資格情報がチャットボットによって保存されることはありません。</span><span class="sxs-lookup"><span data-stu-id="02e2e-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="02e2e-151">詳細については、[Azure Active Directory B2C の概要][aadb2c-docs]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="02e2e-152">Azure SQL Database に格納される情報は、Transparent Data Encryption (TDE) を使用して保存時に暗号化されます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="02e2e-153">また、SQL Database では、クエリおよび処理中にデータを暗号化する Always Encrypted も提供します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="02e2e-154">SQL Database のセキュリティの詳細については、[Azure SQL Database のセキュリティとコンプライアンス][sqlsecurity-docs]に関するセクションをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="02e2e-155">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="02e2e-156">回復性</span><span class="sxs-lookup"><span data-stu-id="02e2e-156">Resiliency</span></span>

<span data-ttu-id="02e2e-157">このシナリオでは、Azure SQL Database を使用して顧客の予約を格納します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-157">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="02e2e-158">SQL Database には、ゾーン冗長データベース、フェールオーバー グループ、geo レプリケーション、自動バックアップなどの機能が用意されています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="02e2e-159">これらの機能により、メンテナンス イベントやシステム停止が発生した場合に、アプリケーションを実行し続けることができます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-159">These features allow your application to continue running in the event of a maintenance event or outage.</span></span> <span data-ttu-id="02e2e-160">詳細については、[Azure SQL Database の可用性機能][sqlavailability-docs]に関するセクションをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="02e2e-161">アプリケーションの正常性を監視するために、このシナリオでは Application Insights を使用します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-161">To monitor the health of your application, this scenario uses Application Insights.</span></span> <span data-ttu-id="02e2e-162">Application Insights により、チャットボットのカスタマー エクスペリエンスや可用性に影響を及ぼすパフォーマンスの問題についてアラートを生成し、対応できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="02e2e-163">詳細については、「[Application Insights とは何か?][appinsights-docs]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="02e2e-164">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="02e2e-165">シナリオのデプロイ</span><span class="sxs-lookup"><span data-stu-id="02e2e-165">Deploy the scenario</span></span>

<span data-ttu-id="02e2e-166">このシナリオは、最も重視される分野を検討するために、次の 3 つのコンポーネントに分かれています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-166">This scenario is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="02e2e-167">[インフラストラクチャ コンポーネント](#deploy-infrastructure-components)。</span><span class="sxs-lookup"><span data-stu-id="02e2e-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="02e2e-168">Azure Resource Manger テンプレートを使用して、コア インフラストラクチャ コンポーネント (App Service、Web App、Application Insights、ストレージ アカウント、SQL Server およびデータベース) をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="02e2e-169">[Web App チャットボット](#deploy-web-app-chatbot)。</span><span class="sxs-lookup"><span data-stu-id="02e2e-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="02e2e-170">Azure CLI を使用して、Bot Service および Language Understanding and Intelligent Services (LUIS) アプリと共にボットをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="02e2e-171">[サンプル C# チャットボット アプリケーション](#deploy-chatbot-c-application-code)。</span><span class="sxs-lookup"><span data-stu-id="02e2e-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="02e2e-172">Visual Studio を使用して、ホテル予約 C# アプリケーションのサンプル コードを確認し、ボットを Azure にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="02e2e-173">**前提条件:** </span><span class="sxs-lookup"><span data-stu-id="02e2e-173">**Prerequisites.**</span></span> <span data-ttu-id="02e2e-174">既存の Azure アカウントが必要です。</span><span class="sxs-lookup"><span data-stu-id="02e2e-174">You must have an existing Azure account.</span></span> <span data-ttu-id="02e2e-175">Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="02e2e-176">インフラストラクチャ コンポーネントをデプロイする</span><span class="sxs-lookup"><span data-stu-id="02e2e-176">Deploy infrastructure components</span></span>

<span data-ttu-id="02e2e-177">Azure Resource Manager テンプレートを使用してインフラストラクチャ コンポーネントをデプロイするには、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="02e2e-178">**[Deploy to Azure]\(Azure にデプロイ\)** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="02e2e-179">Azure portal でテンプレートのデプロイが開くまで待ってから、次の手順を実行します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="02e2e-180">リソース グループを**新規作成**し、テキスト ボックスに名前 (例: *myCommerceChatBotInfrastructure*) を指定します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="02e2e-181">**[場所]** ドロップダウン ボックスでリージョンを選択します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="02e2e-182">SQL Server 管理者アカウントのユーザー名とセキュリティで保護されたパスワードを入力します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="02e2e-183">使用条件を確認し、**[上記の使用条件に同意する]** をオンにします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="02e2e-184">**[購入]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="02e2e-185">デプロイが完了するまで数分かかります。</span><span class="sxs-lookup"><span data-stu-id="02e2e-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="02e2e-186">Web App チャットボットをデプロイする</span><span class="sxs-lookup"><span data-stu-id="02e2e-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="02e2e-187">チャットボットを作成するには、Azure CLI を使用します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="02e2e-188">次の例では、Bot Service 用 CLI 拡張機能をインストールし、リソース グループを作成して、Application Insights を使用するボットをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="02e2e-189">メッセージが表示されたら、Microsoft アカウントを認証し、ボットが Bot Service と Language Understanding and Intelligent Services (LUIS) アプリに自身を登録できるようにします。</span><span class="sxs-lookup"><span data-stu-id="02e2e-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="02e2e-190">チャットボット C# アプリケーション コードをデプロイする</span><span class="sxs-lookup"><span data-stu-id="02e2e-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="02e2e-191">サンプル C# アプリケーションは GitHub で入手できます。</span><span class="sxs-lookup"><span data-stu-id="02e2e-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="02e2e-192">コマース ボット C# サンプル</span><span class="sxs-lookup"><span data-stu-id="02e2e-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="02e2e-193">このサンプル アプリケーションには、Azure Active Directory 認証コンポーネント、および Cognitive Services の Language Understanding and Intelligent Services (LUIS) コンポーネントとの統合が含まれています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="02e2e-194">アプリケーションでは、シナリオを構築してデプロイするために Visual Studio が必要です。</span><span class="sxs-lookup"><span data-stu-id="02e2e-194">The application requires Visual Studio to build and deploy the scenario.</span></span> <span data-ttu-id="02e2e-195">AAD B2C と LUIS アプリの構成に関する追加情報については、GitHub リポジトリのドキュメントをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="02e2e-196">価格</span><span class="sxs-lookup"><span data-stu-id="02e2e-196">Pricing</span></span>

<span data-ttu-id="02e2e-197">このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-197">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="02e2e-198">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="02e2e-199">チャットボットが処理すると予想されるメッセージの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="02e2e-199">We have provided three sample cost profiles based on the amount of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="02e2e-200">[Small][small-pricing]: 1 か月あたり 1 万件未満のメッセージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-200">[Small][small-pricing]: this correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="02e2e-201">[Medium][medium-pricing]: 1 か月あたり 50 万件未満のメッセージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-201">[Medium][medium-pricing]: this correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="02e2e-202">[Large][large-pricing]: 1 か月あたり 1,000 万件未満のメッセージの処理に対応します。</span><span class="sxs-lookup"><span data-stu-id="02e2e-202">[Large][large-pricing]: this correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="02e2e-203">関連リソース</span><span class="sxs-lookup"><span data-stu-id="02e2e-203">Related Resources</span></span>

<span data-ttu-id="02e2e-204">Azure Bot Service の活用方法に関する一連のガイド付きチュートリアルについては、ドキュメントの[チュートリアル ノード][botservice-docs]をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="02e2e-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887