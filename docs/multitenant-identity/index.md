---
title: "マルチテナント アプリケーションの ID 管理"
description: "マルチテナント アプリケーションでの認証、承認、および ID 管理のベスト プラクティス。"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="manage-identity-in-multitenant-applications"></a>マルチテナント アプリケーションでの ID 管理

この一連の記事では、Azure AD を認証と ID 管理のために使用する際のマルチテナントのベスト プラクティスについて説明します。

[![GitHub](../_images/github.png) サンプル コード][sample application]

マルチテナント アプリケーションを構築する場合、最初の課題の 1 つは、すべてのユーザーがテナントに属しているようになるために、ユーザー ID を管理することです。 次に例を示します。

* ユーザーは、その組織の資格情報でサインインします。
* ユーザーは、その組織のデータへのアクセス権を持つ必要がありますが、他のテナントに属しているデータは必要ありません。
* 組織は、アプリケーションにサインアップし、そのメンバーにアプリケーション ロールを割り当てることができます。

Azure Active Directory (Azure AD) には、これらのシナリオのすべてをサポートするいくつかの優れた機能があります。

この一連の記事に付け加えるために、マルチテナント アプリケーションの完全な[エンドツーエンド実装][sample application]を作成しました。 記事には、アプリケーションの作成プロセスで学習した内容が反映されています。 アプリケーションを開始するには、[「Run the Surveys application」][running-the-app] (Surveys アプリケーションの実行) を参照してください。

## <a name="introduction"></a>はじめに

たとえば、クラウドでホストされるエンタープライズ SaaS アプリケーションを作成するとします。 当然ながら、アプリケーションにはユーザーがいます。

![ユーザー](./images/users.png)

ただし、ユーザーは組織に所属しています。

![組織のユーザー](./images/org-users.png)

例: Tailspin は、SaaS アプリケーションへのサブスクリプションを販売しています。 Contoso と Fabrikam がアプリにサインアップします。 Alice (`alice@contoso`) がサインインすると、アプリケーションは Alice が Contoso に属していることを認識します。

* Alice は Contoso データのアクセス権を持っている*べきです*。
* Alice は Fabrikam データのアクセス権を持っている*べきではありません*。

このガイダンスでは、マルチテナント アプリケーションで [Azure Active Directory][AzureAD] (Azure AD) を使用してサインインと認証を処理し、ユーザー ID を管理する方法を説明します。

## <a name="what-is-multitenancy"></a>マルチテナントとは
"*テナント*" はユーザーのグループです。 SaaS アプリケーションのテナントとは、アプリケーションのサブスクライバーまたは顧客です。 "*マルチテナント*" は、複数のテナントがアプリの同じ物理インスタンスを共有するアーキテクチャです。 テナントは物理リソース (VM や記憶域など) を共有しますが、各テナントにはアプリの論理インスタンスが付与されます。

通常、アプリケーション データは、テナント内のユーザー間で共有されますが、他のテナントとは共有されません。

![マルチテナント](./images/multitenant.png)

このアーキテクチャを、各テナントが専用の物理インスタンスを持つシングルテナント アーキテクチャと比較します。 シングルテナント アーキテクチャでは、アプリの新しいインスタンスを起動してテナントを追加します。

![シングル テナント](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>マルチテナントと水平スケーリング
クラウドでスケーリングを達成するには、物理インスタンスを追加する方法が一般的です。 これは "*水平スケーリング*" または "*スケーリング アウト*" と呼ばれます。たとえば、Web アプリがあるとします。 多くのトラフィックを処理する場合、サーバー VM を追加し、ロード バランサーの背後に配置します。 各 VM では、別物理インスタンスの Web アプリを実行します。

![Web サイトの負荷分散](./images/load-balancing.png)

任意の要求を任意のインスタンスにルーティングできます。 その結果、システムは 1 つの論理インスタンスとして機能します。 ユーザーに影響を与えることなく、VM を増減することができます。 このアーキテクチャでは、各物理インスタンスがマルチ テナントであり、インスタンスを追加することで拡張できます。 1 つのインスタンスが停止しても、テナントに影響は及びません。

## <a name="identity-in-a-multitenant-app"></a>マルチテナント アプリの ID
マルチテナント アプリの場合、テナントのコンテキストでユーザーを考慮する必要があります。

**認証**

* ユーザーは、組織の資格情報を使用してアプリにサインインします。 ユーザーがアプリの新しいユーザー プロファイルを作成する必要はありません。
* 同じ組織内のユーザーは、同じテナントに属します。
* ユーザーがサインインすると、アプリケーションユーザーが属するテナントを認識します。

**承認**

* ユーザーのアクション (リソースの表示など) を承認する場合、アプリはユーザーのテナントを考慮する必要があります。
* ユーザーには、アプリケーション内のロール ("Admin"、"Standard User" など) が割り当てられていることがあります。 ロールの割り当ては、SaaS プロバイダーではなく、顧客が管理する必要があります。

**例:** Contoso の従業員である Alice は、ブラウザーでアプリケーションを開き、[ログイン] ボタンをクリックします。 ログイン画面にリダイレクトされ、Alice は会社の資格情報 (ユーザー名とパスワード) を入力します。 この時点で、 `alice@contoso.com`としてアプリにログインされます。 アプリケーションは、Alice がこのアプリケーションの管理者であることも認識します。 Alice は管理者なので、Contoso に属するすべてのリソースの一覧を表示できます。 ただし、自分のテナント内でのみ管理者なので、Fabrikam のリソースは表示できません。

このガイドでは、ID 管理のために Azure AD を使用することに注目します。

* ここでは、顧客が Azure AD (Office 365 テナントや Dynamics CRM テナントなど) にユーザー プロファイルを保存しているとします。
* オンプレミスの Active Directory (AD) がある顧客は、[Azure AD Connect][ADConnect] を使用してオンプレミスの AD を Azure AD と同期できます。

オンプレミスの AD がある顧客が (会社の IT ポリシーなどの理由で) Azure AD Connect を使用できない場合、SaaS プロバイダーは、Active Directory Federation Services (AD FS) を介して顧客の AD と連携できます。 このオプションについては、「 [顧客の AD FS とのフェデレーション]」を参照してください。

このガイドでは、データのパーティション分割、テナントごとの構成など、マルチテナントの他の側面について考慮していません。

[**次へ**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[顧客の AD FS とのフェデレーション]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
