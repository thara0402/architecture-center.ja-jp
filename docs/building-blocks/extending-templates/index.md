---
title: "Azure Resource Manager テンプレート機能の拡張"
description: "Azure Resource Manager テンプレート機能を拡張する方法のヒントとテクニックについて説明します。"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 33ae6850ffa5b28108f30475804be5347859f0c3
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/17/2018
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Azure Resource Manager テンプレート機能の拡張

2016 年に Microsoft patterns & practices チームは、リソースのデプロイを簡略化する目標で、一連の Azure Resource Manager [テンプレートの構成要素](https://github.com/mspnp/template-building-blocks/wiki)を作成しました。 各構成要素には事前構築済みのテンプレートのセットが含まれ、個別のパラメーター ファイルによって指定されるリソースのセットをデプロイします。

構成要素テンプレートは、組み合わせることで大規模で複雑なデプロイを作成できるように設計されています。 たとえば、Azure に仮想マシンをデプロイするには、仮想ネットワーク、ストレージ アカウントおよび他のリソースが必要です。 [仮想ネットワークの構成要素テンプレート](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1))は、仮想ネットワークとサブネットをデプロイします。 [仮想マシンの構成要素テンプレート](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1))は、ストレージ アカウント、ネットワーク インターフェイスおよび実際の VM をデプロイします。 つまり、スクリプトまたはテンプレートを作成して、これら両方の構成要素テンプレートを対応するパラメーター ファイルと一緒に呼び出すと、1 回の操作で完全なアーキテクチャをデプロイすることができます。

p&p は、構成要素テンプレートを開発する際に、Azure Resource Manager のテンプレート機能を拡張するいくつかの方法を考案しています。 このシリーズでは、これらの方法の一部について説明し、ユーザーが自分のテンプレートに取り入れることができるようにします。

> [!NOTE]
> これらの記事では、Azure Resource Manager テンプレートについて十分に理解していると仮定します。