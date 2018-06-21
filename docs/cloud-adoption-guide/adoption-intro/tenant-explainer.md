---
title: '説明: Azure Active Directory テナントとは'
description: Azure でサービスとしての ID (IDaaS) を提供する Azure Active Directory の内部機能について説明します。
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062040"
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>説明: Azure Active Directory テナントとは

[Azure のしくみ](azure-explainer.md)を説明した記事では、Azure は、ユーザーの代わりに視覚化されたハードウェアおよびソフトウェアを実行する、サーバーとネットワークのコレクションであることを説明しました。 また、これらのサーバーの一部は分散型オーケストレーションのアプリケーションを実行して、Azure リソースの作成、読み取り、更新、および削除を管理していることも説明しました。

しかし、お気づきのように、Azure では、任意のユーザーが、リソースに対してこれらのいずれかの操作を実行できるように許可しているわけではありません。 Azure は、**Azure Active Directory** (Azure AD) という信頼済みのデジタル ID サービスを使用して、これらの操作へのアクセスを制限しています。 Azure AD では、ユーザー名、パスワード、プロファイル データなどの情報を格納しています。 Azure AD ユーザーは、**テナント**にセグメント化されています。 テナントとは、通常は組織に関連付けられる Azure AD の安全な専用インスタンスを表す論理コンストラクターです。

テナントを作成するには、Azure では**特権アカウント**が必要です。 この特権アカウントは、Azure アカウントまたはエンタープライズ契約のどちらかに関連付けられています。 これらは、両方とも課金の構造体であり、Azure AD には格納されません。これらのアカウントは、安全性の高い課金データベースに格納されます。 

テナントが作成されたら、そのテナントに対して**テナント ID** が生成され、安全性の高い　内部 Azure AD データベースに保存されます。 特権アカウントの所有者は、Azure Portal にログインして、新しく作成された Azure AD テナントにユーザーを追加できます。 

ほとんどの企業には既に、少なくとも 1 つの ID 管理サービス (通常は Active Directory Domain Services (AD DS)) があります。 Azure AD は AD DS からユーザー ID を同期または連動させることができるので、企業は 2 つの環境で個別に ID を管理する必要はありません。 このことは、デジタル ID の中間および高度の導入ステージに関する記事で、詳しく説明しています。

## <a name="next-steps"></a>次の手順

* Azure AD テナントについて学習したので、基本の導入ステージの最初のステップとして、次は [Azure Active Directory テナントの取得方法][how-to-get-aad-tenant]について学びます。 その後、[Azure AD テナントの設計ガイダンス](tenant.md)を確認してください。

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json