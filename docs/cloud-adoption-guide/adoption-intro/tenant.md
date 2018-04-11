---
title: 'ガイダンス: Azure AD テナント設計'
description: 基本のクラウド導入戦略の一環である Azure テナント設計のガイダンス
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>ガイダンス: Azure AD テナント設計

Azure AD テナントでは、1 つ以上の [Azure サブスクリプション](subscription-explainer.md)に使用されるデジタル ID サービスおよび名前空間を提供しています。 基本の導入の枠組みに従っている場合、[Azure AD テナントの取得方法][how-to-get-aad-tenant]については既に習得済みです。 

## <a name="design-considerations"></a>設計上の考慮事項

- 基本導入ステージでは、単一の Azure AD テナントから開始できます。 所属する組織に既存の Office 365 サブスクリプションまたは Azure サブスクリプションがある場合は、使用できる Azure AD テナントを既にお持ちです。 これらのサブスクリプションのいずれもない場合は、[Azure AD テナントの取得方法][how-to-get-aad-tenant]で詳細を確認できます。 
- 中間および高度の導入ステージでは、オンプレミス ディレクトリを Azure AD と同期または連動させる方法について学習します。 これにより、オンプレミスのデジタル ID を Azure AD で使用できるようになります。 ただし、基本ステージでは、単一の Azure AD テナントの ID のみを持つ新規ユーザーを追加することになります。 これらの ID を管理する必要があります。 たとえば、新規 Azure AD ユーザーの登録、Azure リソースにもうアクセスするつもりがない Azure AD ユーザーの登録解除、およびその他のユーザー アクセス許可への変更があります。

## <a name="next-steps"></a>次の手順

* Azure AD テナントを入手したので、次は[ユーザーの追加方法][azure-ad-add-user]について学習します。 Azure AD テネントに 1 人または複数の新規ユーザーを追加し終えたら、次のステップとして [Azure サブスクリプション](subscription-explainer.md)について学習します。

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
