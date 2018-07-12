---
title: 基本的な Web アプリケーション
description: Microsoft Azure で実行する基本的な Web アプリケーションの推奨アーキテクチャ。
author: MikeWasson
ms.date: 12/12/2017
cardTitle: Basic web application
ms.openlocfilehash: bc8cf9b5c66fc451d097cbc992ecb9a249645dce
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/10/2018
ms.locfileid: "37958842"
---
# <a name="basic-web-application"></a>基本的な Web アプリケーション
[!INCLUDE [header](../../_includes/header.md)]

この参照アーキテクチャは、[Azure App Service][app-service] と [Azure SQL Database][sql-db] を使用する Web アプリケーションを対象とした一連の実証済みプラクティスを示しています。 [**以下のソリューションをデプロイします。**](#deploy-the-solution)

![[0]][0]

"*このアーキテクチャの [Visio ファイル][visio-download]をダウンロードします。*"

## <a name="architecture"></a>アーキテクチャ 

> [!NOTE]
> このアーキテクチャでは、アプリケーション開発については説明しません。また、特定のアプリケーション フレームワークを前提としていません。 目的は、さまざまな Azure サービスがどのように組み合わせるかを理解することです。
>
>

アーキテクチャには次のコンポーネントがあります。

* **リソース グループ**。 [リソース グループ](/azure/azure-resource-manager/resource-group-overview)は、Azure リソースの論理コンテナーです。

* **App Service アプリ**。 [Azure App Service][app-service] は、クラウド アプリケーションを作成およびデプロイするための完全に管理されたプラットフォームです。     

* **App Service プラン**。 [App Service プラン][app-service-plans]は、アプリをホストする管理された仮想マシン (VM) を提供します。 プランに関連付けられているすべてのアプリが同じ VM インスタンスで実行されます。

* **デプロイ スロット**。  [デプロイ スロット][deployment-slots]を使用すると、デプロイをステージングし、運用環境のデプロイとスワップできます。 このようにして、運用環境に直接デプロイするのを回避します。 具体的な推奨事項については、[管理容易性](#manageability-considerations)に関するセクションをご覧ください。

* **IP アドレス**。 App Service アプリには、パブリック IP アドレスとドメイン名があります。 ドメイン名は `azurewebsites.net` のサブドメインです (`contoso.azurewebsites.net` など)。  

* **Azure DNS**。 [Azure DNS][azure-dns] は、DNS ドメインのホスティング サービスであり、Microsoft Azure インフラストラクチャを使用した名前解決を提供します。 Azure でドメインをホストすることで、その他の Azure サービスと同じ資格情報、API、ツール、課金情報を使用して DNS レコードを管理できます。 カスタム ドメイン名 (`contoso.com` など) を使用するには、カスタム ドメイン名を IP アドレスにマップする DNS レコードを作成します。 詳細については、[Azure App Service でのカスタム ドメイン名の構成][custom-domain-name]に関するページをご覧ください。  

* **Azure SQL データベース**。 [SQL Database][sql-db] は、クラウドのサービスとしてのリレーショナル データベースです。 SQL Database は、そのコード ベースを Microsoft SQL Server データベース エンジンと共有しています。 アプリケーションの要件に応じて、[Azure Database for MySQL](/azure/mysql) または [Azure Database for PostgreSQL](/azure/postgresql) を使用することもできます。 これらは、それぞれオープン ソースの MySQL Server および Postgres データベース エンジンに基づく、完全に管理されたデータベース サービスです。

* **論理サーバー**。 Azure SQL Database では、論理サーバーがデータベースをホストします。 論理サーバーごとに複数のデータベースを作成できます。

* **Azure Storage**。 診断ログを格納する BLOB コンテナーを持つ Azure ストレージ アカウントを作成します。

* **Azure Active Directory** (Azure AD)。 Azure AD または他の認証 ID プロバイダーを使用します。

## <a name="recommendations"></a>Recommendations

実際の要件は、ここで説明するアーキテクチャとは異なる場合があります。 このセクションに記載されている推奨事項は原案として使用してください。

### <a name="app-service-plan"></a>App Service プラン
Standard レベルまたは Premium レベルを使用してください。どちらもスケールアウト、自動スケール、および Secure Sockets Layer (SSL) をサポートしています。 各レベルが、コア数とメモリが異なる複数の "*インスタンス サイズ*" をサポートしています。 レベルまたはインスタンス サイズは、プランの作成後に変更できます。 App Service プランの詳細については、「[App Service の価格][app-service-plans-tiers]」を参照してください。

アプリが停止されても、App Service プランのインスタンスは課金の対象となります。 使用していないプラン (テスト デプロイなど) は必ず削除してください。

### <a name="sql-database"></a>SQL Database
[V12 バージョン][sql-db-v12]の SQL Database を使用してください。 SQL Database は、Basic、Standard、および Premium [サービス レベル][sql-db-service-tiers]をサポートしており、各レベルに、[データベース トランザクション ユニット (DTU)][sql-dtu] で測定された複数のパフォーマンス レベルが存在します。 キャパシティ プランニングを実行し、要件に合うレベルとパフォーマンス レベルを選択してください。

### <a name="region"></a>リージョン
ネットワークの待機時間を最小限に抑えるには、App Service プランと SQL Database を同じリージョンでプロビジョニングします。 一般的には、ユーザーに最も近いリージョンを選択してください。

リソース グループにもリージョンがあり、これによりデプロイ メタデータの格納場所が指定されます。 リソース グループとそのリソースは同じリージョンに配置してください。 これにより、デプロイ時の可用性が向上する可能性があります。 

## <a name="scalability-considerations"></a>スケーラビリティに関する考慮事項

Azure App Service の主な利点は、負荷に応じてアプリケーションをスケーリングできることです。 アプリケーションのスケーリングを計画する場合の考慮事項を次に示します。

### <a name="scaling-the-app-service-app"></a>App Service アプリのスケーリング

App Service アプリをスケーリングする方法は 2 つあります。

* "*スケールアップ*" とは、インスタンス サイズを変更することです。 インスタンス サイズにより、各 VM インスタンスのメモリ、コア数、およびストレージが決まります。 手動でスケールアップするには、インスタンス サイズまたはプラン レベルを変更します。  

* "*スケールアウト*" とは、負荷の増加に対応するためにインスタンスを追加することです。 価格レベルごとに、インスタンスの最大数が設定されています。 

  自分でインスタンス数を変更して手動でスケールアウトすることも、[自動スケール][web-app-autoscale]を使用して、スケジュールやパフォーマンス メトリックに基づいて、インスタンスが Azure によって自動的に追加または削除されるように指定することもできます。 各スケール操作は迅速に、通常は数秒以内で行われます。 

  自動スケールを有効にするには、インスタンスの最小数と最大数を定義する自動スケール "*プロファイル*" を作成します。 プロファイルのスケジュールを設定できます。 たとえば、平日と週末で別個のプロファイルを作成できます。 必要に応じて、どのタイミングでインスタンスを追加または削除するかを指定するルールをプロファイルに適用します  (CPU 使用率が 5 分にわたって 70% を超えた場合にインスタンスを 2 つ追加する、など)。
  
Web アプリのスケーリングに関する推奨事項:

* スケールアップとスケールダウンはできるだけ行わないでください。これによりアプリケーションの再起動がトリガーされる場合があるためです。 代わりに、通常の負荷でパフォーマンスの要件を満たすレベルとサイズを選択したうえで、インスタンスをスケールアウトして、トラフィック量の変化に対処します。    
* 自動スケールを有効にします。 アプリケーションに予測可能な通常のワークロードがある場合は、プロファイルを作成することで、事前にインスタンス数のスケジュールを設定します。 ワークロードを予測できない場合は、ルール ベースの自動スケールを使用して負荷の変化に対応します。 両方のアプローチを組み合わせることができます。
* 一般的に、CPU 使用率は自動スケールのルールのメトリックとして適切です。 ただし、アプリケーションのロード テストを行い、潜在的なボトルネックを特定したうえで、そのデータに基づいて自動スケールのルールを設定する必要があります。  
* 自動スケールのルールには、"*クールダウン*" 期間が含まれます。これは、スケール アクションの完了後、新しいスケール アクションが開始するまでの待機時間です。 クールダウン期間により、スケーリングが再度行われる前に、システムを安定させることができます。 インスタンスを追加する場合はクールダウン期間を短くし、インスタンスを削除する場合はクールダウン期間を長くします。 たとえば、インスタンスを追加するときは 5 分に、インスタンスを削除するときは 60 分に設定します。 負荷の高い場所に新しいインスタンスを追加して過度のトラフィックを処理し、段階的にスケールバックすることをお勧めします。

### <a name="scaling-sql-database"></a>SQL Database のスケーリング
SQL Database に高いサービス レベルまたはパフォーマンス レベルが必要な場合は、アプリケーションのダウンタイムを伴わずに個々のデータベースをスケールアップできます。 詳細については、[SQL Database のオプションとパフォーマンス: 各サービス レベルで使用できる内容][sql-db-scale]に関するページをご覧ください。

## <a name="availability-considerations"></a>可用性に関する考慮事項
この資料の作成時点では、App Service のサービス レベル アグリーメント (SLA) は 99.95% です。また、Basic、Standard、および Premium レベルの SQL Database の SLA は 99.99% です。 

> [!NOTE]
> App Service の SLA は、1 つのインスタンスおよび複数のインスタンスの両方に適用されます。  
>
>

### <a name="backups"></a>バックアップ
データ損失が発生した場合は、SQL Database により、ポイントインタイム リストアと geo リストアが提供されます。 こうした機能はすべてのレベルで使用でき、自動的に有効になっています。 バックアップをスケジュールまたは管理する必要はありません。 

- ポイントインタイム リストアを使用して、データベースを以前の特定の時点に戻すことで[人為的ミスから復旧][sql-human-error]できます。 
- geo リストアを使用して、データベースを geo 冗長バックアップから復元することで[サービス停止から復旧][sql-outage-recovery]できます。 

詳細については、[SQL Database を使用したクラウド ビジネス継続性とデータベース ディザスター リカバリー][sql-backup]に関するページをご覧ください。

App Service により、ご使用のアプリケーション ファイルで[バックアップおよび復元][web-app-backup]機能をご利用いただけます。 ただし、バックアップ ファイルに含まれるアプリ設定はプレーンテキストであり、これには接続文字列などのシークレットが含まれる可能性があることに注意してください。 App Service のバックアップ機能を使用して SQL データベースをバックアップすることは避けてください。このバックアップにより、データベースが SQL .bacpac にエクスポートされ、[DTU][sql-dtu] を消費するためです。 代わりに、上で説明した SQL Database のポイントインタイム リストアを使用します。

## <a name="manageability-considerations"></a>管理容易性に関する考慮事項
運用、開発、およびテスト環境それぞれに対して個別のリソース グループを作成してください。 これにより、デプロイの管理、テスト デプロイの削除、およびアクセス権の割り当てを行いやすくなります。

リソースをリソース グループに割り当てるときは、以下の事項を検討してください。

* ライフサイクル。 一般的に、同じライフサイクルのリソースは、同じリソース グループに追加します。
* アクセス。 [ロールベースのアクセス制御][rbac] (RBAC) を使用して、アクセス ポリシーをグループのリソースに適用します。
* 課金。 リソース グループのロールアップ コストを表示できます。  

詳細については、「[Azure Resource Manager の概要](/azure/azure-resource-manager/resource-group-overview)」を参照してください。

### <a name="deployment"></a>デプロイ
デプロイには 2 つの手順が含まれます。

1. Azure リソースのプロビジョニング。 この手順には [Azure Resource Manager テンプレート][arm-template]を使用することをお勧めします。 テンプレートにより、PowerShell または Azure コマンド ライン インターフェイス (CLI) を使用したデプロイが自動化しやすくなります。
2. アプリケーションのデプロイ (コード、バイナリ、およびコンテンツ ファイル)。 ローカル Git リポジトリからのデプロイ、Visual Studio の使用、クラウド ベースのソース管理からの継続的デプロイなど、複数のオプションがあります。 [Azure App Service へのアプリのデプロイ][deploy]に関するページをご覧ください。  

App Service アプリには、`production` という名前のデプロイ スロットが必ず 1 つ含まれます。これは実稼働の運用サイトを表します。 更新プログラムをデプロイするステージング スロットを作成することをお勧めします。 ステージング スロットを使用する利点は次のとおりです。

* デプロイが成功していることを確認してから、それを運用環境にスワップできます。
* ステージング スロットにデプロイすることで、運用環境にスワップする前に、すべてのインスタンスを確実にウォームアップできます。 ウォームアップとコールドスタートに時間がかかるアプリケーションは多数あります。

また、3 番目のスロットを作成して、前回正常起動時のデプロイを保持することをお勧めします。 ステージングと運用をスワップしたら、これまでの運用環境のデプロイ (現在ステージング スロットにある) を、前回正常起動時デプロイ用のスロットに移動します。 このようにして、後で問題を見つけた場合に、前回正常起動時のバージョンにすばやく戻ることができます。

![[1]][1]

前のバージョンに戻す場合は、データベース スキーマのすべての変更に下位互換性があることを確認します。

同じ App Service プラン内のすべてのアプリが同じ VM インスタンスを共有するため、運用環境のデプロイのスロットをテスト用に使用しないでください。 たとえば、ロード テストは実稼働の運用サイトの機能を低下させる可能性があります。 代わりに、運用とテスト用に別々の App Service プランを作成します。 テスト デプロイを別のプランに入れて、運用バージョンから切り離してください。

### <a name="configuration"></a>構成
構成設定を[アプリ設定][app-settings]として格納します。 Resource Manager テンプレート内で、または PowerShell を使用して、アプリ設定を定義します。 実行時、アプリケーションはアプリ設定を環境変数として使用できます。

パスワード、アクセス キー、または接続文字列を、ソース管理にチェックインしないでください。 代わりに、これらをパラメーターとして、デプロイ スクリプトに渡します。スクリプトは、こうした値をアプリ設定として格納します。

デプロイ スロットをスワップすると、アプリ設定は既定でスワップされます。 運用環境とステージングで異なる設定が必要な場合は、スワップされないスロット固有のアプリ設定を作成できます。

### <a name="diagnostics-and-monitoring"></a>診断および監視
アプリケーションのログ記録や Web サーバーのログ記録を含む、[診断のログ記録][diagnostic-logs]を有効にします。 Blob Storage を使用するようにログ記録を構成します。 パフォーマンス上の理由から、診断ログ用に別のストレージ アカウントを作成します。 ログとアプリケーション データに同じストレージ アカウントを使用しないでください。 ログ記録に関する詳細なガイダンスについては、[監視と診断のガイダンス][monitoring-guidance]に関するページをご覧ください。

[New Relic][new-relic]、[Application Insights][app-insights] などのサービスを使用して、負荷がかかった状態のアプリケーションのパフォーマンスと動作を監視してください。 Application Insights の[データ速度の制限][app-insights-data-rate]に気を付けてください。

[Visual Studio Team Services][vsts] などのツールを使用して、ロード テストを行います。 クラウド アプリケーションのパフォーマンス分析の概要については、「[Performance Analysis Primer (パフォーマンス分析の手引き)][perf-analysis]」を参照してください。

アプリケーションのトラブルシューティングのヒント:

* Azure Portal の[トラブルシューティング ブレード][troubleshoot-blade]を使用して、一般的な問題の解決策を見つけます。
* [ログ ストリーミング][web-app-log-stream]を有効にして、ほぼリアルタイムでログ情報を監視します。
* [Kudu ダッシュボード][kudu]には、アプリケーションを監視およびデバッグするためのツールがいくつか用意されています。 詳細については、[知っておくべき Azure Websites のオンライン ツール][kudu]に関するページ (ブログ記事) を参照してください。 Kudu ダッシュボードには Azure Portal からアクセスできます。 アプリのブレードを開き、<strong>[ツール]</strong>、<strong>[Kudu]</strong> の順にクリックします。
* Visual Studio を使用する場合、デバッグとトラブルシューティングのヒントについては、「[Visual Studio を使用した Azure App Service のトラブルシューティング][troubleshoot-web-app]」を参照してください。

## <a name="security-considerations"></a>セキュリティに関する考慮事項
このセクションでは、この記事で説明している Azure サービスに固有のセキュリティの考慮事項について説明します。 これはセキュリティ上のベスト プラクティスを網羅した一覧ではありません。 その他のセキュリティの考慮事項については、[Azure App Service でのアプリのセキュリティ保護][app-service-security]に関するページをご覧ください。

### <a name="sql-database-auditing"></a>SQL Database 監査
監査により、規定遵守を維持したり、ビジネス上の懸念やセキュリティ違反の疑いを示す差異や異常に対する分析情報を得たりすることが容易になります。 「[SQL Database 監査の使用][sql-audit]」を参照してください。

### <a name="deployment-slots"></a>デプロイ スロット
デプロイ スロットごとにパブリック IP アドレスがあります。 開発チームと DevOps チームのメンバーのみがこうしたエンドポイントにアクセスできるように、[Azure Active Directory ログイン][aad-auth]を使用して、非運用スロットをセキュリティで保護します。

### <a name="logging"></a>ログの記録
ユーザーのパスワードなど、不正アクセスにつながる情報はログに記録しないようにします。 このような詳細情報は、データを格納する前に除外してください。   

### <a name="ssl"></a>SSL
App Service アプリには、追加コストなしで、`azurewebsites.net` のサブドメインに SSL エンドポイントが含まれています。 SSL エンドポイントには、`*.azurewebsites.net` ドメインのワイルドカード証明書が含まれます。 カスタム ドメイン名を使用する場合は、カスタム ドメインに合致する証明書を指定する必要があります。 最も簡単な方法は、Azure Portal から直接に証明書を購入することです。 他の証明機関から証明書をインポートすることもできます。 詳細については、「[Azure App Service の SSL 証明書を購入して構成する][ssl-cert]」を参照してください。

セキュリティのベスト プラクティスとして、アプリは HTTP 要求をリダイレクトすることで HTTPS を強制する必要があります。 これをアプリケーション内で実装するか、[Azure App Service でのアプリの HTTPS の有効化][ssl-redirect]に関するページの説明に従って、URL 書き換え規則を使用できます。

### <a name="authentication"></a>認証
Azure AD、Facebook、Google、Twitter などの ID プロバイダー (IDP) を使用して認証することをお勧めします。 認証フローには OAuth 2 または OpenID Connect (OIDC) を使用してください。 Azure AD には、ユーザーとグループの管理、アプリケーション ロールの作成、オンプレミス ID の統合、およびバックエンド サービス (Office 365、Skype for Business など) の使用に関する機能が用意されています。

ユーザーのログインおよび資格情報をアプリケーションで直接管理するのは避けてください。これにより、攻撃対象領域が発生する可能性があるからです。  少なくとも、メール確認、パスワード復元、および多要素認証が必要です。また、パスワードの強度を検証し、パスワード ハッシュを安全に格納する必要があります。 大規模な ID プロバイダーはこれをすべて処理し、セキュリティ プラクティスを常に監視し、強化しています。

[App Service 認証][app-service-auth]を使用して、OAuth/OIDC 認証フローを実装することを検討してください。 App Service 認証の利点は次のとおりです。

* 構成が簡単。
* シンプルな認証シナリオではコードが不要。
* ユーザーに代わって、OAuth アクセス トークンを使用してリソースを消費できるよう、委任承認をサポート。
* 組み込みのトークン キャッシュを提供。

App Service 認証の制限は次のとおりです。  

* カスタマイズのオプションに制限がある。
* ログイン セッションごとに、委任承認が 1 つのバックエンド リソースに制限される。
* 複数の IDP を使用する場合、ホーム領域検出のための組み込みメカニズムがない。
* マルチテナント シナリオでは、アプリケーションがトークン発行者を検証するロジックを実装する必要がある。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法
このアーキテクチャの Resource Manager テンプレートの例については、[GitHub を参照してください][paas-basic-arm-template]。

PowerShell を使用してテンプレートをデプロイするには、次のコマンドを実行します。

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

詳細については、[Azure Resource Manager テンプレートを使用したリソースのデプロイ][deploy-arm-template]に関するページをご覧ください。

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "基本的な Azure Web アプリケーションのアーキテクチャ"
[1]: ./images/paas-basic-web-app-staging-slots.png "運用環境およびステージング環境のデプロイ用スロットのスワップ"
