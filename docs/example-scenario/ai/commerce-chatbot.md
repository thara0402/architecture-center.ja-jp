---
title: ホテル予約用の会話型チャットボット
titleSuffix: Azure Example Scenarios
description: Azure Bot Service を使用して商取引アプリケーション用の会話型チャットボットを構築します。
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: a8540f61a8c5ec500147dc04dc94f3ea6742e6f3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487000"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a>Azure でのホテル予約用の会話型チャットボット

このシナリオ例は、会話型チャットボットをアプリケーションに統合する必要がある企業に適用されます。 このシナリオでは、顧客が Web アプリケーションまたはモバイル アプリケーションを使用して空室状況を確認し、宿泊施設を予約できる C# チャットボットをホテル チェーンで使用します。

顧客がホテルの空室状況を確認して部屋を予約したり、レストランのテイクアウト メニューを表示して料理を注文したり、写真を検索してプリントを注文したりするために使用できます。 従来、企業は顧客の要求に対応するために、カスタマー サービス エージェントを雇用してトレーニングする必要があり、顧客は担当者が支援できるようになるまで待つ必要がありました。

Bot Service と Language Understanding Service や Speech API サービスなどの Azure サービスを使用することで、企業は自動化されたスケーラブルなボットを使って顧客を支援し、注文や予約を処理できます。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- レストランのテイクアウト メニューの表示と料理の注文
- ホテルの空室状況の確認と部屋の予約
- 利用可能な写真の検索とプリントの注文

## <a name="architecture"></a>アーキテクチャ

![会話型チャットボットに関与する Azure コンポーネントのアーキテクチャの概要][architecture]

このシナリオでは、ホテルのコンシェルジュとして機能する会話型ボットに対応できます。 このシナリオのデータ フローは次のとおりです。

1. 顧客がモバイル アプリまたは Web アプリを使用してチャットボットにアクセスします。
2. Azure Active Directory B2C (Business 2 Customer) を使用してユーザーが認証されます。
3. ユーザーは Bot Service と対話して、ホテルの空室状況に関する情報を要求します。
4. Cognitive Services が自然言語の要求を処理して、顧客のコミュニケーションを理解します。
5. ユーザーが結果に満足すると、ボットが SQL Database で顧客の予約を追加または更新します。
6. Application Insights がプロセス全体を通じてランタイム テレメトリを収集し、ボットのパフォーマンスと使用状況の情報を提供して DevOps チームを支援します。

### <a name="components"></a>コンポーネント

- [Azure Active Directory][aad-docs]: Microsoft が提供する、マルチテナントに対応したクラウドベースのディレクトリおよび ID 管理サービスです。 Azure AD では、Google、Facebook、Microsoft アカウントなどの外部 ID を使用して個人を識別できる B2C コネクタがサポートされています。
- [App Service][appservice-docs]: インフラストラクチャを管理することなく、任意のプログラミング言語で Web アプリケーションを構築し、ホストすることができます。
- [Bot Service][botservice-docs]: インテリジェント ボットの構築、テスト、デプロイ、管理を行うツールを提供します。
- [Cognitive Services][cognitive-docs]: 自然なコミュニケーション手段を通じて、見る、聞く、話す、理解する、ユーザーのニーズを解釈することが可能なインテリジェントなアルゴリズムを使用できます。
- [SQL Database][sqldatabase-docs]: SQL Server エンジンの互換性を提供するフル マネージド リレーショナル クラウド データベース サービスです。
- [Application Insights][appinsights-docs]: チャットボットなどのアプリケーションのパフォーマンスを監視できる、拡張可能な Application Performance Management (APM) サービスです。

### <a name="alternatives"></a>代替手段

- [Microsoft Speech API][speech-api] を使用して、顧客とボット間のインターフェイスを変更できます。
- [QnA Maker][qna-maker] を使用すると、FAQ などの半構造化コンテンツから、ボットに知識をすばやく追加できます。
- [Translator Text][translator] は、ボットに多言語サポートを簡単に追加するために検討できるサービスです。

## <a name="considerations"></a>考慮事項

### <a name="availability"></a>可用性

このシナリオでは、Azure SQL Database を使用して顧客の予約を格納します。 SQL Database には、ゾーン冗長データベース、フェールオーバー グループ、geo レプリケーションなどの機能が用意されています。 詳細については、[Azure SQL Database の可用性機能][sqlavailability-docs]に関するセクションをご覧ください。

可用性に関する他のトピックについては、Azure アーキテクチャ センターの[可用性のチェックリスト][availability]を参照してください。

### <a name="scalability"></a>スケーラビリティ

このシナリオでは、Azure App Service を使用します。 App Service により、ボットを実行するインスタンスの数を自動的にスケーリングできます。 この機能を使用することで、Web アプリケーションやチャットボットに対する顧客の要求に対応できます。 自動スケールの詳細については、Azureアーキテクチャ センターの[自動スケールのベスト プラクティス][autoscaling]に関する記事をご覧ください。

スケーラビリティに関する他のトピックについては、Azure アーキテクチャ センターの「[スケーラビリティのチェックリスト][scalability]」をご覧ください。

### <a name="security"></a>セキュリティ

このシナリオでは、Azure Active Directory B2C (Business 2 Consumer) を使用してユーザーを認証します。 AAD B2C を使用することで、機密性の高い顧客のアカウント情報や資格情報がチャットボットによって保存されることはありません。 詳細については、[Azure Active Directory B2C の概要][aadb2c-docs]に関する記事をご覧ください。

Azure SQL Database に格納される情報は、Transparent Data Encryption (TDE) を使用して保存時に暗号化されます。 また、SQL Database では、クエリおよび処理中にデータを暗号化する Always Encrypted も提供します。 SQL Database のセキュリティの詳細については、[Azure SQL Database のセキュリティとコンプライアンス][sqlsecurity-docs]に関するセクションをご覧ください。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

このシナリオでは、Azure SQL Database を使用して顧客の予約を格納します。 SQL Database には、ゾーン冗長データベース、フェールオーバー グループ、geo レプリケーション、自動バックアップなどの機能が用意されています。 これらの機能により、メンテナンス イベントやシステム停止が発生した場合に、アプリケーションを実行し続けることができます。 詳細については、[Azure SQL Database の可用性機能][sqlavailability-docs]に関するセクションをご覧ください。

アプリケーションの正常性を監視するために、このシナリオでは Application Insights を使用します。 Application Insights により、チャットボットのカスタマー エクスペリエンスや可用性に影響を及ぼすパフォーマンスの問題についてアラートを生成し、対応できます。 詳細については、「[Application Insights とは何か?][appinsights-docs]」をご覧ください。

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

このシナリオは、最も重視される分野を検討するために、次の 3 つのコンポーネントに分かれています。

- [インフラストラクチャ コンポーネント](#deploy-infrastructure-components)。 Azure Resource Manger テンプレートを使用して、コア インフラストラクチャ コンポーネント (App Service、Web App、Application Insights、ストレージ アカウント、SQL Server およびデータベース) をデプロイします。
- [Web App チャットボット](#deploy-web-app-chatbot)。 Azure CLI を使用して、Bot Service および Language Understanding and Intelligent Services (LUIS) アプリと共にボットをデプロイします。
- [サンプル C# チャットボット アプリケーション](#deploy-chatbot-c-application-code)。 Visual Studio を使用して、ホテル予約 C# アプリケーションのサンプル コードを確認し、ボットを Azure にデプロイします。

### <a name="prerequisites"></a>前提条件

既存の Azure アカウントが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

### <a name="walk-through"></a>チュートリアル

Resource Manager テンプレートを使用してインフラストラクチャ コンポーネントをデプロイするには、次の手順を実行します。

<!-- markdownlint-disable MD033 -->

1. **[Deploy to Azure]\(Azure にデプロイ\)** をクリックします。<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure portal でテンプレートのデプロイが開くまで待ってから、次の手順を実行します。
   - リソース グループを**新規作成**し、テキスト ボックスに名前 (例: *myCommerceChatBotInfrastructure*) を指定します。
   - **[場所]** ドロップダウン ボックスでリージョンを選択します。
   - SQL Server 管理者アカウントのユーザー名とセキュリティで保護されたパスワードを入力します。
   - 使用条件を確認し、**[上記の使用条件に同意する]** をオンにします。
   - **[購入]** をクリックします。

<!-- markdownlint-enable MD033 -->

デプロイが完了するまで数分かかります。

### <a name="deploy-web-app-chatbot"></a>Web App チャットボットをデプロイする

チャットボットを作成するには、Azure CLI を使用します。 次の例では、Bot Service 用 CLI 拡張機能をインストールし、リソース グループを作成して、Application Insights を使用するボットをデプロイします。 メッセージが表示されたら、Microsoft アカウントを認証し、ボットが Bot Service と Language Understanding and Intelligent Services (LUIS) アプリに自身を登録できるようにします。

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

### <a name="deploy-chatbot-c-application-code"></a>チャットボット C# アプリケーション コードをデプロイする

サンプル C# アプリケーションは GitHub で入手できます。

- [コマース ボット C# サンプル](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

このサンプル アプリケーションには、Azure Active Directory 認証コンポーネント、および Cognitive Services の Language Understanding and Intelligent Services (LUIS) コンポーネントとの統合が含まれています。 アプリケーションでは、シナリオを構築してデプロイするために Visual Studio が必要です。 AAD B2C と LUIS アプリの構成に関する追加情報については、GitHub リポジトリのドキュメントをご覧ください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

チャットボットが処理すると予想されるメッセージの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。

- [Small][small-pricing]: この価格例は、1 か月あたり 10,000 未満のメッセージの処理に対応します。
- [Medium][medium-pricing]: この価格例は、1 か月あたり 500,000 未満のメッセージの処理に対応します。
- [Large][large-pricing]: この価格例は、1 か月あたり 10,000,000 未満のメッセージの処理に対応します。

## <a name="related-resources"></a>関連リソース

Azure Bot Service に関する一連のガイド付きチュートリアルについては、ドキュメントの[チュートリアル セクション][botservice-docs]をご覧ください。

<!-- links -->

[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
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