---
title: 'Azure の導入: 基本'
description: 企業が Azure を導入するのに必要な基本レベルの知識について説明します。
author: petertay
ms.openlocfilehash: 3f522d1662849d651423d8022ad152c64692b823
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290477"
---
# <a name="adopting-azure-foundational"></a>Azure の導入: 基本

Azure の導入は、企業の組織としての成熟における最初のステージです。 このステージの終わりには、組織に所属する人々が Azure に簡単なワークロードをデプロイできるようになります。

以下の一覧に、基本導入ステージを完了するためのタスクを示します。 一覧は手順に沿って示しているので、各タスクを順番に完了してください。 事前にタスクを完了していた場合は、一覧の次のタスクに進んでください。 

1. Azure 内部について: 
    - **説明**:[ Azure のしくみ](azure-explainer.md)
    - **説明**: [クラウド リソース ガバナンスとは](governance-explainer.md)
2. Azure のエンタープライズ デジタル ID について: 
    - **説明:** [Azure Active Directory テナントとは](tenant-explainer.md)
    - **方法:** [Azure Active Directory テナントを取得する](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **ガイダンス:** [Azure AD テナント設計ガイダンス](tenant.md)
    - **方法:** [Azure Active Directory に新しいユーザーを追加する](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Azure のサブスクリプションについて: 
    - **説明:** [Azure サブスクリプションとは](subscription-explainer.md)
    - **ガイダンス:** [Azure サブスクリプション設計](subscription.md)
4. Azure でのリソース管理について:  
    - **説明:** [Azure Resource Manager とは](resource-manager-explainer.md)
    - **説明:** [Azure リソース グループとは](resource-group-explainer.md)
    - **説明:** [Azure でのリソース アクセスについて](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **方法:** [Azure Portal を使用して Azure リソース グループを作成する](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **ガイダンス:** [Azure リソース グループ設計ガイダンス](resource-group.md)
    - **ガイダンス:** [Azure リソースの名前付け規則](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. 基本の Azure アーキテクチャのデプロイ: 
    - 「[Azure コンピューティング オプションの概要](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)」で、サービスとしてのインフラストラクチャ (IaaS) やサービスとしてのプラットフォーム (PaaS) など、さまざまな種類の Azure コンピューティング オプションについて確認してください。
    - さまざまな種類の Azure コンピューティング オプションを理解したので、次は Azure の最初のリソースとして、Web アプリケーション (PaaS) または仮想マシン (IaaS) のどちらかを選択します。
    - PaaS: サービスとしてのプラットフォームへの導入: 
        - **方法:** [基本的な Web アプリケーションを Azure にデプロイする](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **ガイダンス:** [基本的な Web アプリケーション](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)の Azure へのデプロイに関する実証済みプラクティス
    - IaaS: 仮想ネットワークへの導入: 
        - **説明:** [Azure 仮想ネットワーク](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **方法:** [ポータルを使用して仮想ネットワークを Azure にデプロイする](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IaaS: 単一の仮想マシン (VM) ワークロードのデプロイ (Windows および Linux):
        - **方法:** [ポータルを使用して Windows VM を Azure にデプロイする](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **ガイダンス:** [Azure での Windows VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **方法:** [ポータルを使用して Linux VM を Azure にデプロイする](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **ガイダンス:** [Azure での Linux VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
