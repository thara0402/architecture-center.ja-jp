---
title: 'Azure の導入: 基本'
description: 企業が Azure を導入するのに必要な基本レベルの知識について説明します。
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229169"
---
# <a name="adopting-microsoft-azure-foundational"></a>Microsoft Azure の導入: 基本

クラウド テクノロジを初めて導入する組織では、導入をどこから始めたらよいのかを決めるのが難しい場合があります。 基本となる導入ステージの目標は、この出発点を提供することにあります。 組織の各個人がこのステージに対処する経験を積んでおけば、シンプルなワークロードのためのコンピューティング リソースを Azure にデプロイするうえで必要なすべての知識とスキルが身につきます。 

> [!NOTE]
> このガイドでは、アプリケーション開発については取り上げません。 Azure でのアプリケーション開発の詳細については、「[Azure アプリケーション アーキテクチャ ガイド](/azure/architecture/guide/)」を参照してください。

ガイドのこのステージは、組織内の次のペルソナを対象としています。

- "*財務:*" Azure の財務コミットメントの所有者。請求、チャージバックなど、リソース使用量のコストを追跡するためのポリシーおよび手順の開発を担当します。
- "*中央 IT:*" リソースの管理、アクセスなど、組織のクラウド リソース管理のほか、ワークロードの正常性や監視を担当します。
- "*ワークロード所有者:*" 開発者、テスト担当者、ビルド エンジニアなど、Azure へのワークロードのデプロイに関与するすべての開発ロールです。

## <a name="section-1-azure-basics"></a>セクション 1: Azure の基本

この導入セクションは、"*財務*" と "*中央 IT*" の各ペルソナを対象とします。 このセクションの重点は、[クラウド ガバナンスの概念](governance-explainer.md)の学習に備え、[Azure のしくみ](azure-explainer.md)についての基本的理解を得ることにあります。 また、組織の "*ワークロード所有者*" がこのコンテンツを確認すると、リソース アクセスの管理方法を理解するうえで役立つ場合があります。

## <a name="section-2-governance-design-guide"></a>セクション 2: ガバナンス設計ガイド

Azure のしくみとクラウド ガバナンスの基本を理解したら、Azure 導入の最初のステップとして、Azure における[リソース アクセス管理](azure-resource-access.md)について学習します。 この記事では、リソース アクセス要求を行うための Azure サービスと、これらの要求の検証に使用されるコントロールについて説明します。

次のステップでは、単一のチームの[ガバナンス モデルの設計](governance-how-to.md)方法を学習します。 この記事では、既に説明した、リソース アクセス管理サービスとコントロールを構成する方法について説明します。

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>セクション 3: 基本的なリソース アクセス管理モデルの実装

導入の最後のステップでは、前に設計したガバナンス モデルを実装する方法について説明します。 

まず、お客様の組織には Azure アカウントが必要です。 お客様の組織の既存の[マイクロソフトエンタープライズ契約](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx)に Azure が含まれていない場合は、前払いによる年額コミットメントを行うことで Azure を追加できます。 詳細については、「[Azure のエンタープライズ向けライセンス](https://azure.microsoft.com/pricing/enterprise-agreement/)」を参照してください。 

Azure アカウントが作成されたら、組織の 1 人を Azure **アカウント所有者**に指定します。 Azure Active Directory (Azure AD) テナントは、そのとき既定で作成されます。 Azure **アカウント所有者**は、**ワークロード所有者**となる組織内ユーザーの[ユーザー アカウントを作成する](/azure/active-directory/add-users-azure-active-directory)必要があります。 

次に、Azure **アカウント所有者**は、[サブスクリプションを作成](https://docs.microsoft.com/partner-center/create-a-new-subscription)し、これに [Azure AD テナントを関連付ける](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory)必要があります。

サブスクリプションを作成し、これに Azure AD テナントを関連付けたので、最後に、[**ワークロード所有者**を、組み込みの**所有者**ロールを持つサブスクリプションに追加](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal)します。

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>セクション 4: 基本的なワークロード アーキテクチャを Azure にデプロイする

このセクションの対象ユーザーは、"*ワークロード所有者*" ペルソナです。 "*ワークロード所有者*" は、このワークロードの計算およびネットワークの要件を定義し、これらの要件を満たす適切なリソースを選択して Azure にデプロイします。 

基本的な導入ステージでは、ワークロード所有者は、基本的な Web アプリケーション、または仮想ネットワーク (VNet) と仮想マシン (VM) を選択できます。 Azure のさまざまな種類のコンピューティング オプションの詳細については、[Azure コンピューティング オプションの概要](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)に関するページをご覧ください。

選択したコンピューティング オプションに関係なく、各デプロイでは**リソース グループ**が必要です。 **ワークロード所有者**が、[リソース グループを作成する](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy)必要があります。 デプロイの一環として、**ワークロード所有者**がリソース グループの名前を指定します。 以下のセクションでは、次の名前を使用します。

### <a name="basic-web-application-paas"></a>基本的な Web アプリケーション (PaaS)

基本的な Web アプリケーションの場合、[Web Apps のドキュメント](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json)に記載されている 5 分間のクイック スタートのいずれかを選択し、その手順に従います。 

> [!NOTE]
> 一部のクイック スタートでは、既定でリソース グループがデプロイされます。 その場合、**ワークロード所有者**が明示的にリソース グループを作成する必要はありません。 それ以外の場合は、上記で作成したリソース グループに Web アプリケーションをデプロイします。

シンプルなワークロードをデプロイしたら、Azure への[基本的な Web アプリケーション](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)のデプロイに関する実証済みプラクティスの詳細を学習できます。

### <a name="single-windows-or-linux-vm-iaas"></a>単一の Windows または Linux VM (IaaS)

仮想マシンで実行されるシンプルなワークロードの場合、最初のステップとして、仮想ネットワークをデプロイします。 仮想マシン、ロード バランサー、ゲートウェイなど、Azure のすべての IaaS リソースには、仮想ネットワークが必要です。 [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) について学習したら、[ポータルを使用して仮想ネットワークを Azure にデプロイする](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 Azure portal で仮想ネットワークの設定を指定するときに、上記で作成したリソース グループの名前を指定します。

次のステップでは、単一の Windows VM または Linux VM のどちらをデプロイするかを決定します。 Windows VM の場合は、[ポータルを使用して Windows VM を Azure にデプロイする](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 ここでも、Azure portal で仮想マシンの設定を指定するときに、上記で作成したリソース グループの名前を指定します。

手順に従って VM をデプロイしたら、[Azure での Windows VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)について学習できます。 Linux VM の場合は、[ポータルを使用して Linux VM を Azure にデプロイする](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 [Azure での Linux VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)の詳細を参照することもできます。

## <a name="next-steps"></a>次の手順

クラウドに備えるための次のステージは[**中級ステージ**](../intermediate-stage/overview.md)です。 中級ステージでは、複数のワークロードを実行しているオンプレミス ネットワークを拡張する方法と、複数のチーム用のガバナンス モデルについて学習します。