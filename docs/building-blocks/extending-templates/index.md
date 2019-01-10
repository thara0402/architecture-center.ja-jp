---
title: Azure Resource Manager テンプレート機能の拡張
description: Azure Resource Manager テンプレート機能を拡張する方法のヒントやコツについて説明します。
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 725013df3d0551060a9f504da7a97cdc370f956a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111378"
---
# <a name="extend-azure-resource-manager-template-functionality"></a><span data-ttu-id="08b03-103">Azure Resource Manager テンプレート機能の拡張</span><span class="sxs-lookup"><span data-stu-id="08b03-103">Extend Azure Resource Manager template functionality</span></span>

<span data-ttu-id="08b03-104">2016 年に Microsoft patterns & practices チームは、リソースのデプロイを簡略化する目標で、一連の Azure Resource Manager [テンプレートの構成要素](https://github.com/mspnp/template-building-blocks/wiki)を作成しました。</span><span class="sxs-lookup"><span data-stu-id="08b03-104">In 2016, the Microsoft patterns & practices team created a set of Azure Resource Manager [template building blocks](https://github.com/mspnp/template-building-blocks/wiki) with the goal of simplifying resource deployment.</span></span> <span data-ttu-id="08b03-105">各構成要素には事前構築済みのテンプレートのセットが含まれ、個別のパラメーター ファイルによって指定されるリソースのセットをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="08b03-105">Each building block contains a set of pre-built templates that deploy sets of resources specified by separate parameter files.</span></span>

<span data-ttu-id="08b03-106">構成要素テンプレートは、組み合わせることで大規模で複雑なデプロイを作成できるように設計されています。</span><span class="sxs-lookup"><span data-stu-id="08b03-106">The building block templates are designed to be combined together to create larger and more complex deployments.</span></span> <span data-ttu-id="08b03-107">たとえば、Azure に仮想マシンをデプロイするには、仮想ネットワーク、ストレージ アカウントおよび他のリソースが必要です。</span><span class="sxs-lookup"><span data-stu-id="08b03-107">For example, deploying a virtual machine in Azure requires a virtual network, storage accounts, and other resources.</span></span> <span data-ttu-id="08b03-108">[仮想ネットワークの構成要素テンプレート](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1))は、仮想ネットワークとサブネットをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="08b03-108">The [virtual network building block template](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) deploys a virtual network and subnets.</span></span> <span data-ttu-id="08b03-109">[仮想マシンの構成要素テンプレート](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1))は、ストレージ アカウント、ネットワーク インターフェイスおよび実際の VM をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="08b03-109">The [virtual machine building block template](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) deploys storage accounts, network interfaces, and the actual VMs.</span></span> <span data-ttu-id="08b03-110">つまり、スクリプトまたはテンプレートを作成して、これら両方の構成要素テンプレートを対応するパラメーター ファイルと一緒に呼び出すと、1 回の操作で完全なアーキテクチャをデプロイすることができます。</span><span class="sxs-lookup"><span data-stu-id="08b03-110">You can then create a script or template to call both building block templates with their corresponding parameter files to deploy a complete architecture with one operation.</span></span>

<span data-ttu-id="08b03-111">p&p は、構成要素テンプレートを開発する際に、Azure Resource Manager のテンプレート機能を拡張するいくつかの方法を考案しています。</span><span class="sxs-lookup"><span data-stu-id="08b03-111">While developing the building block templates, p&p designed several concepts to extend Azure Resource Manager template functionality.</span></span> <span data-ttu-id="08b03-112">このシリーズでは、これらの方法の一部について説明し、ユーザーが自分のテンプレートに取り入れることができるようにします。</span><span class="sxs-lookup"><span data-stu-id="08b03-112">In this series, we will describe several of these concepts so you can use them in your own templates.</span></span>

> [!NOTE]
> <span data-ttu-id="08b03-113">これらの記事では、Azure Resource Manager テンプレートについて十分に理解していると仮定します。</span><span class="sxs-lookup"><span data-stu-id="08b03-113">These articles assume you have an advanced understanding of Azure Resource Manager templates.</span></span>