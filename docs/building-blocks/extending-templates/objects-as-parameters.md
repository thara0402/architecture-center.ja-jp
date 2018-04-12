---
title: Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する
description: Azure Resource Manager テンプレートの機能を拡張して、オブジェクトをパラメーターとして使用する方法について説明します
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/06/2018
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="d1583-103">Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する</span><span class="sxs-lookup"><span data-stu-id="d1583-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="d1583-104">[Azure Resource Manager テンプレートを作成する][azure-resource-manager-create-template]場合、テンプレートに直接リソース プロパティ値を指定するか、または、デプロイ時にパラメーターを定義して値を指定することができます。</span><span class="sxs-lookup"><span data-stu-id="d1583-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="d1583-105">小規模なデプロイでは各々のプロパティ値にパラメーターを使用しても問題ありませんが、デプロイあたり 255 個のパラメーターという制限があります。</span><span class="sxs-lookup"><span data-stu-id="d1583-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="d1583-106">大規模かつ複雑なデプロイになると、パラメーターが足りなくなる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d1583-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="d1583-107">この問題を解決する 1 つの方法が、値の代わりにオブジェクトをパラメーターとして使用することです。</span><span class="sxs-lookup"><span data-stu-id="d1583-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="d1583-108">これを行うには、テンプレートのパラメーターを定義し、デプロイ時に 1 つの値ではなく、JSON オブジェクトを指定します。</span><span class="sxs-lookup"><span data-stu-id="d1583-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="d1583-109">そして、テンプレート内の [`parameter()` 関数][azure-resource-manager-functions]とドット演算子を使用して、パラメーターのサブプロパティを参照します。</span><span class="sxs-lookup"><span data-stu-id="d1583-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="d1583-110">仮想ネットワーク リソースをデプロイする例を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="d1583-111">まず、テンプレートで `VNetSettings` パラメーターを指定し、`type` を `object` に設定します。</span><span class="sxs-lookup"><span data-stu-id="d1583-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="d1583-112">次に、`VNetSettings` オブジェクトに値を指定してみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="d1583-113">デプロイ時にパラメーター値を指定する方法については、「[Azure Resource Manager テンプレートの構造と構文の詳細][azure-resource-manager-authoring-templates]」の**パラメーター**のセクションをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="d1583-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="d1583-114">1 つのパラメーターが実際には次の 3 つのサブプロパティ、`name`、`addressPrefixes`、および`subnets` を指定することがわかります。</span><span class="sxs-lookup"><span data-stu-id="d1583-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="d1583-115">これらのサブプロパティの各々は、1 つの値または他のサブプロパティを指定します。</span><span class="sxs-lookup"><span data-stu-id="d1583-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="d1583-116">その結果、1 つのパラメーターが、仮想ネットワークのデプロイに必要なすべての値を指定します。</span><span class="sxs-lookup"><span data-stu-id="d1583-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="d1583-117">では、テンプレートの残りの部分を見て、`VNetSettings` オブジェクトがどのように使用されているか確認しましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="d1583-118">`VNetSettings` オブジェクトの値は、`parameters()` 関数を `[]` 配列インデクサーおよびドット演算子とともに使用して、仮想ネットワーク リソースに必要なプロパティに適用されています。</span><span class="sxs-lookup"><span data-stu-id="d1583-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="d1583-119">この方法は、リソースにパラメーター オブジェクトの値を静的に適用する場合に有効です。</span><span class="sxs-lookup"><span data-stu-id="d1583-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="d1583-120">ただし、デプロイ時にプロパティ値の配列を動的に割り当てる場合は、[コピー ループ][azure-resource-manager-create-multiple-instances]を使用します。</span><span class="sxs-lookup"><span data-stu-id="d1583-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="d1583-121">コピー ループを使用するには、リソースのプロパティ値の JSON 配列を指定します。すると、コピー ループが値をリソースのプロパティに動的に適用します。</span><span class="sxs-lookup"><span data-stu-id="d1583-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="d1583-122">動的なアプローチをする場合は、注意すべき問題が 1 つあります。</span><span class="sxs-lookup"><span data-stu-id="d1583-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="d1583-123">その問題を示すために、プロパティ値の典型的な配列を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="d1583-124">この例では、プロパティの値は変数に格納されています。</span><span class="sxs-lookup"><span data-stu-id="d1583-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="d1583-125">ここでは 2 つの配列があります&mdash; 1 つは `firstProperty` という名前、もう 1 つは `secondProperty` という名前です。</span><span class="sxs-lookup"><span data-stu-id="d1583-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="d1583-126">コピー ループを使用して、変数のプロパティにアクセスする方法を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="d1583-127">`copyIndex()` 関数がコピー ループの現在のイテレーションを返すので、それを 2 つの配列のそれぞれにインデックスとして同時に使用します。</span><span class="sxs-lookup"><span data-stu-id="d1583-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="d1583-128">これは、2 つの配列が同じ長さである場合は問題なく動作します。</span><span class="sxs-lookup"><span data-stu-id="d1583-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="d1583-129">もし間違って 2 つの配列の長さが異なっていると、問題が起きます&mdash;この場合、テンプレートはデプロイ時に検証に失敗します。</span><span class="sxs-lookup"><span data-stu-id="d1583-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="d1583-130">この問題は、1 つのオブジェクトにすべてのプロパティを含めることで回避できます。値がない場合に簡単に気づくことができるからです。</span><span class="sxs-lookup"><span data-stu-id="d1583-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="d1583-131">たとえば、`propertyObject` 配列の各要素が前からの `firstProperty` および `secondProperty` 配列の組み合わせであるような、別のパラメーター オブジェクトを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="d1583-132">配列内の 3 つ目の要素に気づいたでしょうか。</span><span class="sxs-lookup"><span data-stu-id="d1583-132">Notice the third element in the array?</span></span> <span data-ttu-id="d1583-133">`number` プロパティがありませんが、このようにしてパラメーター値を作成すると、それが足りないことに簡単に気づくことができます。</span><span class="sxs-lookup"><span data-stu-id="d1583-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="d1583-134">コピー ループ内でプロパティ オブジェクトを使用する</span><span class="sxs-lookup"><span data-stu-id="d1583-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="d1583-135">この方法は、[serial copy loop][azure-resource-manager-create-multiple] と組み合わせる場合、特に、子リソースをデプロイするのにさらに便利です。</span><span class="sxs-lookup"><span data-stu-id="d1583-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="d1583-136">これを示すために、2 つのセキュリティ規則で[ネットワーク セキュリティ グループ (NSG)][nsg] をデプロイするテンプレートを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="d1583-137">最初に、パラメーターを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="d1583-138">テンプレートを見ると、`securityRules` という名前の配列を含む `networkSecurityGroupsSettings` という名前の 1 つのパラメーターを定義したことがわかります。</span><span class="sxs-lookup"><span data-stu-id="d1583-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="d1583-139">この配列には、セキュリティ規則の設定の数を指定する 2 つの JSON オブジェクトが含まれています。</span><span class="sxs-lookup"><span data-stu-id="d1583-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="d1583-140">ここで、テンプレートを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-140">Now let's take a look at our template.</span></span> <span data-ttu-id="d1583-141">最初の `NSG1` という名前のリソースは、NSG をデプロイしています。</span><span class="sxs-lookup"><span data-stu-id="d1583-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="d1583-142">2 番目の `loop-0` という名前のリソースは 2 つの関数を実行します。1 つは、NSG に対して `dependsOn` であるため、そのデプロイは `NSG1` が完了するまで開始されません。また、これはシーケンシャル ループの最初のイテレーションです。</span><span class="sxs-lookup"><span data-stu-id="d1583-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="d1583-143">3 番目のリソースは入れ子になったテンプレートで、最後の例のように、パラメーター値のオブジェクトを使用してセキュリティ規則をデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d1583-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="d1583-144">`securityRules` 子リソースでのプロパティ値の指定方法を詳しく見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="d1583-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="d1583-145">すべてのプロパティは `parameter()` 関数を使用して参照されているので、ドット演算子を使用して、イテレーションの現在値によってインデックス付けされた `securityRules` 配列を参照します。</span><span class="sxs-lookup"><span data-stu-id="d1583-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="d1583-146">最後に、別のドット演算子を使用して、オブジェクトの名前を参照します。</span><span class="sxs-lookup"><span data-stu-id="d1583-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="d1583-147">テンプレートを試行する</span><span class="sxs-lookup"><span data-stu-id="d1583-147">Try the template</span></span>

<span data-ttu-id="d1583-148">このテンプレートを実験する場合は、次の手順に従います。</span><span class="sxs-lookup"><span data-stu-id="d1583-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="d1583-149">Azure Portal に移動し、**+** アイコンを選択して、**[テンプレートのデプロイ]** のリソースの種類を検索しそれを選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="d1583-150">**[テンプレートのデプロイ]** ページに移動して、**[作成]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="d1583-151">このボタンにより、**[カスタム デプロイ]** ブレードが開きます。</span><span class="sxs-lookup"><span data-stu-id="d1583-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="d1583-152">**[テンプレートの編集]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="d1583-153">空のテンプレートを削除します。</span><span class="sxs-lookup"><span data-stu-id="d1583-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="d1583-154">サンプル テンプレートをコピーして、右のウィンドウに貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="d1583-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="d1583-155">**[保存]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="d1583-156">**[カスタム デプロイ]** ウィンドウに戻り、**[パラメーターの編集]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="d1583-157">**[パラメーターの編集]** ブレードで、既存のテンプレートを削除します。</span><span class="sxs-lookup"><span data-stu-id="d1583-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="d1583-158">上記のサンプル パラメーター テンプレートをコピーして、貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="d1583-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="d1583-159">**[保存]** を選択します。これにより、**[カスタム デプロイ]** ブレードに戻ります。</span><span class="sxs-lookup"><span data-stu-id="d1583-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="d1583-160">**[カスタム デプロイ]** ブレードで、サブスクリプションを選択し、新しいリソース グループを作成するか、既存のリソース グループを使用して、場所を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="d1583-161">使用条件を確認し、**[同意する]** チェックボックスをオンにします。</span><span class="sxs-lookup"><span data-stu-id="d1583-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="d1583-162">**[購入]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="d1583-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d1583-163">次の手順</span><span class="sxs-lookup"><span data-stu-id="d1583-163">Next steps</span></span>

* <span data-ttu-id="d1583-164">これらの手法を拡張して、[プロパティ オブジェクトのトランスフォーマーとコレクター ](./collector.md)を実装することができます。</span><span class="sxs-lookup"><span data-stu-id="d1583-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="d1583-165">トランスフォーマーとコレクターの手法はより一般的なもので、テンプレートからリンクすることができます。</span><span class="sxs-lookup"><span data-stu-id="d1583-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="d1583-166">この手法は、[テンプレート構成ブロックのプロジェクト](https://github.com/mspnp/template-building-blocks)と [Azure 参照アーキテクチャ](/azure/architecture/reference-architectures/)で実装することもできます。</span><span class="sxs-lookup"><span data-stu-id="d1583-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="d1583-167">ここでのテンプレートを確認して、この手法をどのように実装したか確認することができます。</span><span class="sxs-lookup"><span data-stu-id="d1583-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg