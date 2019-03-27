---
title: Azure Resource Manager テンプレートのリソースを条件付きでデプロイする
description: Azure Resource Manager テンプレートの機能を拡張し、パラメーターの値に応じて、条件付きでリソースをデプロイする方法について説明します。
author: petertay
ms.date: 10/30/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: f3d22c6437cdabcd93a781ecf7c99db5a570d7cf
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243803"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="9b58f-103">Azure Resource Manager テンプレートのリソースを条件付きでデプロイする</span><span class="sxs-lookup"><span data-stu-id="9b58f-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="9b58f-104">パラメーター値が存在するかどうかなどの条件に基づいて、リソースをデプロイするようにテンプレートを設計する必要があるシナリオがいくつかあります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="9b58f-105">たとえば、テンプレートは仮想ネットワークをデプロイしたり、ピアリング用の他の仮想ネットワークを指定するパラメーターを含んでいたりする可能性があります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="9b58f-106">ピアリングのパラメーター値を指定していない場合は、Resource Manager によってピアリング リソースをデプロイする必要はありません。</span><span class="sxs-lookup"><span data-stu-id="9b58f-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="9b58f-107">これを行うには、リソースの [condition 要素][azure-resource-manager-condition]を使用して、パラメーター配列の長さをテストします。</span><span class="sxs-lookup"><span data-stu-id="9b58f-107">To accomplish this, use the [condition element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="9b58f-108">長さが 0 の場合は、`false` が返されてデプロイが阻止されますが、0 より大きな値の場合は `true` が返され、デプロイが許可されます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="9b58f-109">テンプレートの例</span><span class="sxs-lookup"><span data-stu-id="9b58f-109">Example template</span></span>

<span data-ttu-id="9b58f-110">これを実証するテンプレートの例を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="9b58f-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="9b58f-111">このテンプレートでは [condition 要素][azure-resource-manager-condition]を使用して、`Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースのデプロイを制御します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-111">Our template uses the [condition element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="9b58f-112">このリソースは、同じリージョン内の 2 つの Azure Virtual Network 間でピアリングを作成します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="9b58f-113">テンプレートの各セクションを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="9b58f-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="9b58f-114">`parameters` 要素は、`virtualNetworkPeerings` という名前の単一パラメーターを定義します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```

<span data-ttu-id="9b58f-115">`virtualNetworkPeerings` パラメーターは `array` であり、次のスキーマがあります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
      "name": "firstVNet/peering1",
      "properties": {
          "remoteVirtualNetwork": {
              "id": "[resourceId('Microsoft.Network/virtualNetworks','secondVNet')]"
          },
          "allowForwardedTraffic": true,
          "allowGatewayTransit": true,
          "useRemoteGateways": false
      }
    }
]
```

<span data-ttu-id="9b58f-116">このパラメーターのプロパティが、[仮想ネットワークのピアリングに関連する設定][vnet-peering-resource-schema]を指定します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="9b58f-117">`resources` セクションで `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースを指定する場合に、これらのプロパティの値が提供されます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "firstVNet", "secondVNet"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```

<span data-ttu-id="9b58f-118">このテンプレートのこの部分では、2 つの処理が行われています。</span><span class="sxs-lookup"><span data-stu-id="9b58f-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="9b58f-119">最初に、デプロイされている実際のリソースは type が `Microsoft.Resources/deployments` のインライン テンプレートであり、実際に `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` をデプロイする独自のテンプレートが含まれています。</span><span class="sxs-lookup"><span data-stu-id="9b58f-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="9b58f-120">このインライン テンプレートに対する `name` は、`copyIndex()` の現在の繰り返しを接頭辞 `vnp-` で連結することによって一意になります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span>

<span data-ttu-id="9b58f-121">`condition` 要素は、`greater()` 関数が `true` として評価された場合にリソースが処理される必要があることを指定しています。</span><span class="sxs-lookup"><span data-stu-id="9b58f-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="9b58f-122">ここでは、`virtualNetworkPeerings` パラメーター配列が 0 よりも `greater()` であるかどうかをテストしています。</span><span class="sxs-lookup"><span data-stu-id="9b58f-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="9b58f-123">0 より大きい場合は、評価結果が `true` となり、`condition` が満たされます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="9b58f-124">それ以外の場合は、`false` となります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="9b58f-125">次に、`copy` ループを指定します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="9b58f-126">これは、`serial` ループであり、最後のリソースがデプロイされるまで待機している各リソースで、ループが順番に実行されることを意味しています。</span><span class="sxs-lookup"><span data-stu-id="9b58f-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="9b58f-127">`count` プロパティは、ループが繰り返される回数を指定します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="9b58f-128">このプロパティにはデプロイ対象のリソースを指定するパラメーター オブジェクトが含まれるため、これは通常、`virtualNetworkPeerings` 配列の長さに設定されます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="9b58f-129">ただしその場合、配列が空だと、存在しないプロパティにアクセスしようとしていることを Resource Manager が通知するため、検証が失敗します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="9b58f-130">しかし、これは回避できます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-130">We can work around this, however.</span></span> <span data-ttu-id="9b58f-131">必要な変数について説明します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-131">Let's take a look at the variables we'll need:</span></span>

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

<span data-ttu-id="9b58f-132">`workaround` 変数には、`true` および `false` という 2 つのプロパティが含まれます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="9b58f-133">`true` プロパティは、評価されて `virtualNetworkPeerings`パラメーター配列の値になります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="9b58f-134">`false` プロパティの評価結果は、Resource Manager での表示を想定している名前付きのプロパティも含めて空のオブジェクトになります。`false` は、`virtualNetworkPeerings` パラメーターと同様、実際には検証を満たす配列であることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="9b58f-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see &mdash; note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span>

<span data-ttu-id="9b58f-135">`peerings` 変数は `workaround` 変数をもう一度使用し、`virtualNetworkPeerings` パラメーター配列の長さが 0 より大きいかどうかをテストします。</span><span class="sxs-lookup"><span data-stu-id="9b58f-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="9b58f-136">0 より大きい場合、`string` は `true` として評価され、`workaround` 変数は `virtualNetworkPeerings` パラメーター配列として評価されます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-136">If it is, the `string` evaluates to `true` and the `workaround` variable evaluates to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="9b58f-137">それ以外の場合、評価結果が `false` となり、`workaround` 変数の評価結果によって配列の最初の要素が空のオブジェクトになります。</span><span class="sxs-lookup"><span data-stu-id="9b58f-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="9b58f-138">変数の問題について対処を行ったため、`Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースのデプロイを入れ子になったテンプレート内で指定し、`name` と `properties` を `virtualNetworkPeerings` パラメーター配列から渡すことができます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="9b58f-139">これは、リソースの `properties` 要素で入れ子にされた `template` 要素で確認できます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="9b58f-140">テンプレートを試行する</span><span class="sxs-lookup"><span data-stu-id="9b58f-140">Try the template</span></span>

<span data-ttu-id="9b58f-141">テンプレートの例は [GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="9b58f-141">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="9b58f-142">テンプレートをデプロイするには、次の [Azure CLI][cli] コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-142">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a><span data-ttu-id="9b58f-143">次の手順</span><span class="sxs-lookup"><span data-stu-id="9b58f-143">Next steps</span></span>

* <span data-ttu-id="9b58f-144">スカラー値ではなくオブジェクトを、テンプレート パラメーターとして使用します。</span><span class="sxs-lookup"><span data-stu-id="9b58f-144">Use objects instead of scalar values as template parameters.</span></span> <span data-ttu-id="9b58f-145">「[Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する](./objects-as-parameters.md)」を参照してください</span><span class="sxs-lookup"><span data-stu-id="9b58f-145">See [Use an object as a parameter in an Azure Resource Manager template](./objects-as-parameters.md)</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-manager-templates-resources#condition
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
