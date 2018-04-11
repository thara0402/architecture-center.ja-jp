---
title: Tailspin Surveys アプリケーションについて
description: Tailspin Surveys アプリケーションの概要
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="the-tailspin-scenario"></a>Tailspin シナリオ

[![GitHub](../_images/github.png) サンプル コード][sample application]

Tailspin は、Surveys という名前の SaaS アプリケーションを開発している仮の会社です。 このアプリケーションを使用すると、オンライン アンケートを作成および発行することができます。

* 組織はアプリケーションにサインアップできます。
* 組織がサインアップした後、ユーザーはその組織の資格情報でアプリケーションにサインインできます。
* ユーザーはアンケートを作成、編集、および発行することができます。

> [!NOTE]
> アプリケーションを開始するには、「[Run the Surveys application (Surveys アプリケーションの実行)]」を参照してください。
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a>ユーザーがアンケートを作成、編集、および表示できる
認証されたユーザーは、自身が作成した、または自身が共同作成者の権限を持つすべてのアンケートを表示し、新しいアンケートを作成することができます。 ユーザーが組織 ID `bob@contoso.com` でサインインしていることに注意してください。

![アンケート アプリ](./images/surveys-screenshot.png)

このスクリーンショットには、[アンケートの編集] ページが表示されます。

![アンケートの編集](./images/edit-survey.png)

ユーザーは、同じテナント内で他のユーザーによって作成されたアンケートを表示することもできます。

![テナントのアンケート](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a>アンケートの所有者が共同作成者を招待できる
ユーザーがアンケートを作成する場合は、アンケートの共同作成者に他のユーザーを招待できます。 共同作成者はアンケートを編集することはできますが、削除または公開することはできません。  

![共同作成者の追加](./images/add-contributor.png)

ユーザーが他のテナントから共同作成者を追加できます。これにより、リソースのテナント間共有が可能になります。 このスクリーンショットでは、Bob (`bob@contoso.com`) が自身で作成したアンケートに、Alice (`alice@fabrikam.com`) を共同作成者として追加しています。

Alice がログインすると、[共同作成できるアンケート] の下にアンケートが表示されます。

![アンケートの共同作成者](./images/contributor.png)

Alice が Contoso テナントのゲストとしてではなく、自分のテナントにサインインすることに注意してください。 Alice に付与されているのは、そのアンケートの共同作成者のアクセス許可だけです &mdash; Contoso テナントから他のアンケートを表示することはできません。

## <a name="architecture"></a>アーキテクチャ
Surveys アプリケーションは、Web フロント エンドおよび Web API バックエンドで構成されます。 両方とも [ASP.NET Core] を使用して実装されます。

Web アプリケーションでは、Azure Active Directory (Azure AD) を使用して、ユーザーを認証します。 また、Web アプリケーションでは Azure AD を呼び出して、Web API の OAuth 2 アクセス トークンを取得します。 アクセス トークンは、Azure Redis Cache でキャッシュされます。 キャッシュを使用すると、複数のインスタンスで同じトークンのキャッシュ (たとえば、サーバー ファーム) を共有できます。

![アーキテクチャ](./images/architecture.png)

[**次へ**][authentication]

<!-- Links -->

[authentication]: authenticate.md

[Run the Surveys application (Surveys アプリケーションの実行)]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
