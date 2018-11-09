---
title: Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する
description: Azure Resource Manager テンプレートの機能を拡張して、オブジェクトをパラメーターとして使用する方法について説明します
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: c1955823b3474efa0abea1d9634add5f13d02eda
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251891"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="45226-103">Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する</span><span class="sxs-lookup"><span data-stu-id="45226-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="45226-104">[Azure Resource Manager テンプレートを作成する][azure-resource-manager-create-template]場合、テンプレートに直接リソース プロパティ値を指定するか、または、デプロイ時にパラメーターを定義して値を指定することができます。</span><span class="sxs-lookup"><span data-stu-id="45226-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="45226-105">小規模なデプロイでは各々のプロパティ値にパラメーターを使用しても問題ありませんが、デプロイあたり 255 個のパラメーターという制限があります。</span><span class="sxs-lookup"><span data-stu-id="45226-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="45226-106">大規模かつ複雑なデプロイになると、パラメーターが足りなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="45226-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="45226-107">この問題を解決する 1 つの方法が、値の代わりにオブジェクトをパラメーターとして使用することです。</span><span class="sxs-lookup"><span data-stu-id="45226-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="45226-108">これを行うには、テンプレートのパラメーターを定義し、デプロイ時に 1 つの値ではなく、JSON オブジェクトを指定します。</span><span class="sxs-lookup"><span data-stu-id="45226-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="45226-109">そして、テンプレート内の [`parameter()` 関数][azure-resource-manager-functions]とドット演算子を使用して、パラメーターのサブプロパティを参照します。</span><span class="sxs-lookup"><span data-stu-id="45226-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="45226-110">仮想ネットワーク リソースをデプロイする例を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="45226-111">まず、テンプレートで `VNetSettings` パラメーターを指定し、`type` を `object` に設定します。</span><span class="sxs-lookup"><span data-stu-id="45226-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="45226-112">次に、`VNetSettings` オブジェクトに値を指定してみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="45226-113">デプロイ時にパラメーター値を指定する方法については、「[Azure Resource Manager テンプレートの構造と構文の詳細][azure-resource-manager-authoring-templates]」の**パラメーター**のセクションを参照してください。</span><span class="sxs-lookup"><span data-stu-id="45226-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

<span data-ttu-id="45226-114">1 つのパラメーターが実際には次の 3 つのサブプロパティ、`name`、`addressPrefixes`、および`subnets` を指定することがわかります。</span><span class="sxs-lookup"><span data-stu-id="45226-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="45226-115">これらのサブプロパティの各々は、1 つの値または他のサブプロパティを指定します。</span><span class="sxs-lookup"><span data-stu-id="45226-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="45226-116">その結果、1 つのパラメーターが、仮想ネットワークのデプロイに必要なすべての値を指定します。</span><span class="sxs-lookup"><span data-stu-id="45226-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="45226-117">では、テンプレートの残りの部分を見て、`VNetSettings` オブジェクトがどのように使用されているか確認しましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
<span data-ttu-id="45226-118">`VNetSettings` オブジェクトの値は、`parameters()` 関数を `[]` 配列インデクサーおよびドット演算子とともに使用して、仮想ネットワーク リソースに必要なプロパティに適用されています。</span><span class="sxs-lookup"><span data-stu-id="45226-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="45226-119">この方法は、リソースにパラメーター オブジェクトの値を静的に適用する場合に有効です。</span><span class="sxs-lookup"><span data-stu-id="45226-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="45226-120">ただし、デプロイ時にプロパティ値の配列を動的に割り当てる場合は、[コピー ループ][azure-resource-manager-create-multiple-instances]を使用します。</span><span class="sxs-lookup"><span data-stu-id="45226-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="45226-121">コピー ループを使用するには、リソースのプロパティ値の JSON 配列を指定します。すると、コピー ループが値をリソースのプロパティに動的に適用します。</span><span class="sxs-lookup"><span data-stu-id="45226-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="45226-122">動的なアプローチをする場合は、注意すべき問題が 1 つあります。</span><span class="sxs-lookup"><span data-stu-id="45226-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="45226-123">その問題を示すために、プロパティ値の典型的な配列を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="45226-124">この例では、プロパティの値は変数に格納されています。</span><span class="sxs-lookup"><span data-stu-id="45226-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="45226-125">ここでは 2 つの配列があります&mdash; 1 つは `firstProperty` という名前、もう 1 つは `secondProperty` という名前です。</span><span class="sxs-lookup"><span data-stu-id="45226-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

<span data-ttu-id="45226-126">コピー ループを使用して、変数のプロパティにアクセスする方法を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

<span data-ttu-id="45226-127">`copyIndex()` 関数がコピー ループの現在のイテレーションを返すので、それを 2 つの配列のそれぞれにインデックスとして同時に使用します。</span><span class="sxs-lookup"><span data-stu-id="45226-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="45226-128">これは、2 つの配列が同じ長さである場合は問題なく動作します。</span><span class="sxs-lookup"><span data-stu-id="45226-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="45226-129">もし間違って 2 つの配列の長さが異なっていると、問題が起きます&mdash;この場合、テンプレートはデプロイ時に検証に失敗します。</span><span class="sxs-lookup"><span data-stu-id="45226-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="45226-130">この問題は、1 つのオブジェクトにすべてのプロパティを含めることで回避できます。値がない場合に簡単に気づくことができるからです。</span><span class="sxs-lookup"><span data-stu-id="45226-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="45226-131">たとえば、`propertyObject` 配列の各要素が前からの `firstProperty` および `secondProperty` 配列の組み合わせであるような、別のパラメーター オブジェクトを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

<span data-ttu-id="45226-132">配列内の 3 つ目の要素に気づいたでしょうか。</span><span class="sxs-lookup"><span data-stu-id="45226-132">Notice the third element in the array?</span></span> <span data-ttu-id="45226-133">`number` プロパティがありませんが、このようにしてパラメーター値を作成すると、それが足りないことに簡単に気づくことができます。</span><span class="sxs-lookup"><span data-stu-id="45226-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="45226-134">コピー ループ内でプロパティ オブジェクトを使用する</span><span class="sxs-lookup"><span data-stu-id="45226-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="45226-135">この方法は、[serial copy loop][azure-resource-manager-create-multiple] と組み合わせる場合、特に、子リソースをデプロイするのにさらに便利です。</span><span class="sxs-lookup"><span data-stu-id="45226-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="45226-136">これを示すために、2 つのセキュリティ規則で[ネットワーク セキュリティ グループ (NSG)][nsg] をデプロイするテンプレートを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="45226-137">最初に、パラメーターを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="45226-138">テンプレートを見ると、`securityRules` という名前の配列を含む `networkSecurityGroupsSettings` という名前の 1 つのパラメーターを定義したことがわかります。</span><span class="sxs-lookup"><span data-stu-id="45226-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="45226-139">この配列には、セキュリティ規則の設定の数を指定する 2 つの JSON オブジェクトが含まれています。</span><span class="sxs-lookup"><span data-stu-id="45226-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

<span data-ttu-id="45226-140">ここで、テンプレートを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-140">Now let's take a look at our template.</span></span> <span data-ttu-id="45226-141">最初の `NSG1` という名前のリソースは、NSG をデプロイしています。</span><span class="sxs-lookup"><span data-stu-id="45226-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="45226-142">2 番目の `loop-0` という名前のリソースは 2 つの関数を実行します。1 つは、NSG に対して `dependsOn` であるため、そのデプロイは `NSG1` が完了するまで開始されません。また、これはシーケンシャル ループの最初のイテレーションです。</span><span class="sxs-lookup"><span data-stu-id="45226-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="45226-143">3 番目のリソースは入れ子になったテンプレートで、最後の例のように、パラメーター値のオブジェクトを使用してセキュリティ規則をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="45226-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

<span data-ttu-id="45226-144">`securityRules` 子リソースでのプロパティ値の指定方法を詳しく見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="45226-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="45226-145">すべてのプロパティは `parameter()` 関数を使用して参照されているので、ドット演算子を使用して、イテレーションの現在値によってインデックス付けされた `securityRules` 配列を参照します。</span><span class="sxs-lookup"><span data-stu-id="45226-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="45226-146">最後に、別のドット演算子を使用して、オブジェクトの名前を参照します。</span><span class="sxs-lookup"><span data-stu-id="45226-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="45226-147">テンプレートを試行する</span><span class="sxs-lookup"><span data-stu-id="45226-147">Try the template</span></span>

<span data-ttu-id="45226-148">テンプレートの例は [GitHub][github] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="45226-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="45226-149">テンプレートをデプロイするには、リポジトリを複製し、次の [Azure CLI][cli] コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="45226-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="45226-150">次の手順</span><span class="sxs-lookup"><span data-stu-id="45226-150">Next steps</span></span>

- <span data-ttu-id="45226-151">オブジェクト配列を反復処理するテンプレートを作成し、これを JSON スキーマに変換する方法を確認します。</span><span class="sxs-lookup"><span data-stu-id="45226-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="45226-152">「[Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する](./collector.md)」を参照してください</span><span class="sxs-lookup"><span data-stu-id="45226-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>


<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
