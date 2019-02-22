---
title: CAF:Azure でのリソース アクセス管理
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure でのリソース アクセス管理の説明:Azure Resource Manager、サブスクリプション、リソース グループ、およびリソース
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902153"
---
# <a name="resource-access-management-in-azure"></a>Azure でのリソース アクセス管理

[クラウド ガバナンス](../overview.md)に関するページでは、リソース管理など、クラウド ガバナンスの 5 つの規範について概要を説明してします。  [リソース アクセス ガバナンスの概要](overview.md)に関するページでは、リソース アクセス管理がリソース管理の規範にどのように適合するかについても説明しています。 ガバナンス モデルを設計する方法について確認する前に、Azure のリソース アクセス管理の制御について理解することが重要です。 これらのリソース アクセス管理の制御の構成により、ご自身のガバナンス モデルの基礎が形成されます。

まずは、Azure でリソースがどのようにデプロイされるかを詳しく見ていきましょう。

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a>Azure リソースとは

Azure では、**リソース**という用語は、Azure によって管理されるエンティティを指します。 たとえば、仮想マシン、仮想ネットワーク、およびストレージ アカウントはすべて、Azure リソースと呼ばれます。

![リソースの図](../../_images/governance-1-9.png)
*図 1.リソース。*

## <a name="what-is-an-azure-resource-group"></a>Azure リソース グループとは

Azure の各リソースは、[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)に属している必要があります。 リソース グループは、複数のリソースを単一のエンティティとして管理できるようにまとめる、論理コンストラクトに過ぎません。 たとえば、[n 層アプリケーション](/azure/architecture/guide/architecture-styles/n-tier)のリソースなど、似たようなライフサイクルを共有するリソースを、グループとして作成または削除できます。

![リソースを含むリソース グループの図](../../_images/governance-1-10.png)
*図 2.リソース グループにはリソースが含まれる。*

リソース グループ、およびそのリソース グループに含まれているリソースは、Azure **サブスクリプション**に関連付けられます。

## <a name="what-is-an-azure-subscription"></a>Azure サブスクリプションとは

Azure サブスクリプションは、リソース グループとそのリソースをまとめる論理コンストラクトであるという点では、リソース グループに似ています。 しかし、Azure サブスクリプションは、Azure Resource Manager によって使用される制御にも関連付けられています。 これはどういう意味でしょうか。 Azure サブスクリプションと Azure Resource Manager の関係について確認するために、Azure Resource Manager を詳しく見てみましょう。

![Azure サブスクリプションの図](../../_images/governance-1-11.png)
*図 3.Azure サブスクリプション。*

## <a name="what-is-azure-resource-manager"></a>Azure Resource Manager とは

[Azure のしくみ](../../getting-started/what-is-azure.md)に関するページでは、Azure には、Azure のすべての機能を調整する多くのサービスを備えた "フロントエンド" が含まれることを説明しました。 これらのサービスの 1 つが [Azure Resource Manager](/azure/azure-resource-manager/) で、このサービスは、リソースを管理するために、クライアントによって使用される RESTful API をホストしています。

![Azure Resource Manager の図](../../_images/governance-1-12.png)
*図 4.Azure Resource Manager。*

次の図は、[PowerShell](/powershell/azure/overview)、[Azure portal](https://portal.azure.com)、および [Azure コマンド ライン インターフェイス (CLI)](/cli/azure) という 3 つのクライアントを示しています。

![Azure Resource Manager API に接続する Azure クライアントの図](../../_images/governance-1-13.png)
*図 5.Azure クライアントが Azure Resource Manager RESTful API に接続する。*

これらのクライアントは、RESTful API を使用して Azure Resource Manager に接続しますが、Azure Resource Manager には、リソースを直接管理する機能が含まれていません。 代わりに、Azure ではリソースの種類のほとんどに、独自の[**リソース プロバイダー**](/azure/azure-resource-manager/resource-group-overview#terminology)があります。

![Azure リソース プロバイダー](../../_images/governance-1-14.png)
*図 6.Azure リソース プロバイダー。*

クライアントが特定のリソースを管理するように要求すると、Azure Resource Manager は要求を完了するために、そのリソースの種類のリソース プロバイダーに接続します。 たとえば、クライアントが仮想マシン リソースを管理するように要求した場合、Azure Resource Manager は、**Microsoft.Compute** リソース プロバイダーに接続します。

![Azure Resource Manager から Microsoft.Compute リソース プロバイダーへの接続](../../_images/governance-1-15.png)
*図 7.Azure Resource Manager は、クライアント要求で指定されたリソースを管理する **Microsoft.Compute** リソース プロバイダーに接続します。*

Azure Resource Manager が、仮想マシン リソースを管理するために、サブスクリプションとリソース グループの両方の識別子を指定するようクライアントに要求しています。

Azure Resource Manager のしくみがわかったので、Azure サブスクリプションが、Azure Resource Manager によって使用される制御にどのように関連付けられるかという説明に戻りましょう。 Azure Resource Manager によってリソースの管理要求が実行される前に、一連の制御がチェックされます。

最初の制御は、要求が必ず検証済みユーザーによって行われていることです。また、ユーザー ID 機能を提供するために、Azure Resource Manager は [Azure Active Directory (Azure AD)](/azure/active-directory/) との間に信頼関係を確保します。

![Azure Active Directory](../../_images/governance-1-16.png)
*図 8.Azure Active Directory。*

Azure AD では、ユーザーが**テナント**にセグメント化されています。 テナントとは、通常は組織に関連付けられる Azure AD の安全な専用インスタンスを表す論理コンストラクターです。 各サブスクリプションが Azure AD テナントと関連付けられています。

![サブスクリプションと関連付けられている Azure AD テナント](../../_images/governance-1-17.png)
*図 9.サブスクリプションと関連付けられている Azure AD テナント。*

特定のサブスクリプションでリソースを管理するためのクライアント要求ごとに、関連付けられている Azure AD テナント内にユーザーがアカウントを持っている必要があります。

次の制御では、要求を行うための十分なアクセス許可がユーザーにあることが確認されます。 [ロールベースのアクセス制御 (RBAC)](/azure/role-based-access-control/) を使用して、アクセス許可がユーザーに割り当てられます。

![RBAC ロールに割り当てられたユーザー](../../_images/governance-1-18.png)
*図 10.テナントの各ユーザーに 1 つ以上の RBAC ロールが割り当てらている。*

RBAC ロールでは、特定のリソースに対してユーザーが適用できる一連のアクセス許可が指定されます。 ロールがユーザーに割り当てられると、これらのアクセス許可が適用されます。 たとえば、[組み込み**所有者**ロール](/azure/role-based-access-control/built-in-roles#owner)を使用すると、ユーザーはリソースに対して任意のアクションを実行できます。

次の制御は、[Azure リソース ポリシー](/azure/governance/policy/)に適うように指定されている設定で、要求が許可されることのチェックです。 Azure リソース ポリシーでは、特定のリソースに対して許可される操作が指定されます。 たとえば、Azure リソース ポリシーを使用して、ユーザーが特定の種類の仮想マシンのみデプロイできるように指定できます。

![Azure リソース ポリシー](../../_images/governance-1-19.png)
*図 11.Azure リソース ポリシー。*

次の制御は、要求が [Azure サブスクリプションの制限](/azure/azure-subscription-service-limits)を超えていないことのチェックです。 たとえば、サブスクリプションあたりのリソース グループ数は、980 グループに制限されています。 追加のリソース グループをデプロイする要求を受け取ったときに、制限に達している場合、その要求は拒否されます。

![Azure リソース制限](../../_images/governance-1-20.png)
*図 12.Azure リソース制限。*

最後の制御は、要求が、サブスクリプションに関連付けられている財務コミットメント内に収まっていることのチェックです。 たとえば、仮想マシンのデプロイ要求の場合は、サブスクリプションに十分な支払い情報があることが Azure Resource Manager によって確認されます。

![サブスクリプションに関連付けられている財務コミットメント](../../_images/governance-1-21.png)
*図 13.財務コミットメントはサブスクリプションに関連付けられています。*

## <a name="summary"></a>まとめ

この記事では、Azure Resource Manager を使用して、リソース アクセスが Azure でどのように管理されるかを説明しました。

## <a name="next-steps"></a>次の手順

Azure でのリソース アクセスの管理方法については理解できました。次は、これらのサービスを使用して、[単純なワークロード](governance-simple-workload.md)または[複数チーム](governance-multiple-teams.md)向けのガバナンス モデルを設計する方法を説明します。

> [!div class="nextstepaction"]
> [ガバナンスの概要](../overview.md)
