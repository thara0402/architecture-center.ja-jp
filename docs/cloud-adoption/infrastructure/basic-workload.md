---
title: 'エンタープライズ クラウドの導入: 基本的なワークロードのデプロイ'
description: 基本的なワークロードを Azure にデプロイする方法について説明します
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: e615ba33fb713278a3695057e61d99c92b72b3f2
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389317"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a>エンタープライズ クラウドの導入: 基本的なワークロードのデプロイ

**ワークロード** という言葉は、通常、アプリケーション、サービスなどの機能の任意の単位を定義するものとして、認識されています。 ワークロードを、サーバーにデプロイされているコード成果物という観点だけでなく、その他の必要な任意のサービスという観点から考えるのです。 オンプレミス アプリケーションまたはサービスついては、この定義が便利です。しかし、クラウドでは、さらに踏み込む必要があります。

クラウドでのワークロードに含まれるのは、すべての成果物だけではありません。クラウド リソースも含まれています。 クラウド リソースをこの定義に含める理由は、コードとしてのインフラストラクチャと呼ばれる概念です。 [Azure のしくみ](../getting-started/what-is-azure.md)に関するページで確認したように、Azure ではリソースがオーケストレーター サービスによってデプロイされます。 オーケストレーター サービスでは、この機能は Web API を使用して公開され、この Web API は、Powershell、Azure コマンド ライン インターフェイス (CLI)、Azure portal など、複数のツールを使用して呼び出すことができます。 つまり、リソースは、アプリケーションに関連付けられているコード成果物と共に格納できる、コンピューターが読み取り可能なファイルで指定できます。

これにより、コード成果物および必要なクラウド リソースという観点からワークロードを定義でき、さらにはワークロードを分離することができます。 ワークロードは、リソースが整理されている方法、ネットワーク トポロジ、またはその他の属性によって分離できます。 ワークロードを分離する目的は、ワークロード固有のリソースをチームに関連付け、これらのリソースのあらゆる側面を個別に管理できるようにすることです。 こうすることで、複数のチームが Azure でリソース管理サービスを共有する際に、お互いのリソースを誤って削除または変更してしまうのを防ぐことができます。

また、この分離により、DevOps という概念も有効になります。 DevOps にはソフトウェア開発プラクティスが含まれます。これには上記のソフトウェア開発と IT 操作の両方が含まれますが、可能な限り自動化が使用されます。 DevOps の原則の 1 つは、継続的インテグレーションと継続的デリバリー (CI/CD) と呼ばれます。 継続的インテグレーションは、開発者がコード変更をコミットするたびに実行される自動ビルド プロセスを意味します。一方、継続的デリバリーは、開発環境 (テストの場合)、運用環境 (最終デプロイの場合) など、さまざまな環境へのこのコードの自動デプロイ プロセスを意味します。

## <a name="basic-workload"></a>基本的なワークロード

**基本的なワークロード**は、通常、単一の Web アプリケーション、または仮想マシン (VM) を含む仮想ネットワーク (VNet) として定義されます。 

> [!NOTE]
> このガイドでは、アプリケーション開発については取り上げません。 Azure でのアプリケーション開発の詳細については、「[Azure アプリケーション アーキテクチャ ガイド](/azure/architecture/guide/)」を参照してください。

ワークロードが Web アプリケーションか VM かに関係なく、これらのデプロイには**リソース グループ**が必要です。 リソース グループを作成するアクセス許可を持つユーザーは、次の手順に従う前に、リソース グループを作成する必要があります。

## <a name="basic-web-application-paas"></a>基本的な Web アプリケーション (PaaS)

基本的な Web アプリケーションの場合、[Web Apps のドキュメント](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json)に記載されている 5 分間のクイック スタートのいずれかを選択し、その手順に従います。 

> [!NOTE]
> 一部のクイック スタートでは、既定でリソース グループがデプロイされます。 その場合、明示的にリソース グループを作成する必要はありません。 それ以外の場合は、上記で作成したリソース グループに Web アプリケーションをデプロイします。

シンプルなワークロードをデプロイしたら、Azure への[基本的な Web アプリケーション](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)のデプロイに関する実証済みプラクティスの詳細を学習できます。

## <a name="single-windows-or-linux-vm-iaas"></a>単一の Windows または Linux VM (IaaS)

仮想マシンで実行されるシンプルなワークロードの場合、最初のステップとして、仮想ネットワークをデプロイします。 仮想マシン、ロード バランサー、ゲートウェイなど、Azure のすべての IaaS リソースには、仮想ネットワークが必要です。 [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json) について学習したら、[ポータルを使用して仮想ネットワークを Azure にデプロイする](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 Azure portal で仮想ネットワークの設定を指定するときに、上記で作成したリソース グループの名前を指定します。

次のステップでは、単一の Windows VM または Linux VM のどちらをデプロイするかを決定します。 Windows VM の場合は、[ポータルを使用して Windows VM を Azure にデプロイする](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 ここでも、Azure portal で仮想マシンの設定を指定するときに、上記で作成したリソース グループの名前を指定します。

手順に従って VM をデプロイしたら、[Azure での Windows VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)について学習できます。 Linux VM の場合は、[ポータルを使用して Linux VM を Azure にデプロイする](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)手順に従います。 [Azure での Linux VM の実行に関する実証済みプラクティス](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)の詳細を参照することもできます。