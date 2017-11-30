---
title: "Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する"
description: "Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する方法について説明します"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a><span data-ttu-id="b71ed-103">Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する</span><span class="sxs-lookup"><span data-stu-id="b71ed-103">Implement a property transformer and collector in an Azure Resource Manager template</span></span>

<span data-ttu-id="b71ed-104">「[Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する][objects-as-parameters]」では、リソース プロパティ値をオブジェクトに格納し、デプロイ時にリソースに適用する方法を学びました。</span><span class="sxs-lookup"><span data-stu-id="b71ed-104">In [use an object as a parameter in an Azure Resource Manager template][objects-as-parameters], you learned how to store resource property values in an object and apply them to a resource during deployment.</span></span> <span data-ttu-id="b71ed-105">これはパラメーターの管理には非常に便利な方法ですが、テンプレートで使用するたびに、オブジェクトのプロパティをリソースのプロパティにマップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b71ed-105">While this is a very useful way to manage your parameters, it still requires you to map the object's properties to resource properties each time you use it in your template.</span></span>

<span data-ttu-id="b71ed-106">これを回避するには、オブジェクト配列を反復処理してリソースで想定されている JSON スキーマに変換するような、プロパティ変換とコレクターのテンプレートを実装します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-106">To work around this, you can implement a property transform and collector template that iterates your object array and transforms it into the JSON schema expected by the resource.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="b71ed-107">この方法では、Resource Manager のテンプレートと関数について深い理解があることが必要です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-107">This approach requires that you have a deep understanding of Resource Manager templates and functions.</span></span>

<span data-ttu-id="b71ed-108">[ネットワーク セキュリティ グループ (NSG)][nsg] をデプロイする例で、プロパティのコレクターとトランスフォーマーの実装方法を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-108">Let's take a look at how we can implement a property collector and transformer with an example that deploys a [network security group (NSG)][nsg].</span></span> <span data-ttu-id="b71ed-109">次の図は、これらのテンプレートとテンプレート内のリソースとの間のリレーションシップを示しています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-109">The diagram below shows the relationship between our templates and our resources within those templates:</span></span>

![プロパティのコレクターとトランスフォーマーのアーキテクチャ](../_images/collector-transformer.png)

<span data-ttu-id="b71ed-111">この**呼び出し元テンプレート**には 2 つのリソースが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-111">Our **calling template** includes two resources:</span></span>
* <span data-ttu-id="b71ed-112">**コレクター テンプレート**を呼び出す、テンプレートのリンク。</span><span class="sxs-lookup"><span data-stu-id="b71ed-112">a template link that invokes our **collector template**.</span></span>
* <span data-ttu-id="b71ed-113">デプロイする NSG リソース。</span><span class="sxs-lookup"><span data-stu-id="b71ed-113">the NSG resource to deploy.</span></span>

<span data-ttu-id="b71ed-114">この**コレクター テンプレート**には 2 つのリソースが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-114">Our **collector template** includes two resources:</span></span>
* <span data-ttu-id="b71ed-115">**アンカー** リソース。</span><span class="sxs-lookup"><span data-stu-id="b71ed-115">an **anchor** resource.</span></span>
* <span data-ttu-id="b71ed-116">コピー ループ内で変換テンプレートを呼び出す、テンプレートのリンク。</span><span class="sxs-lookup"><span data-stu-id="b71ed-116">a template link that invokes the transform template in a copy loop.</span></span>

<span data-ttu-id="b71ed-117">この**変換テンプレート**には 1 つのリソースが含まれています: **メイン テンプレート**で `source` JSON を NSG リソースで想定されている JSON スキーマに変換する変数を持つ、空のテンプレート。</span><span class="sxs-lookup"><span data-stu-id="b71ed-117">Our **transform template** includes a single resource: an empty template with a variable that transforms our `source` JSON to the JSON schema expected by our NSG resource in the **main template**.</span></span>

## <a name="parameter-object"></a><span data-ttu-id="b71ed-118">パラメーター オブジェクト</span><span class="sxs-lookup"><span data-stu-id="b71ed-118">Parameter object</span></span>

<span data-ttu-id="b71ed-119">[パラメーターとしてのオブジェクト][objects-as-parameters]に関するページにある、`securityRules` パラメーター オブジェクトを使用します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-119">We'll be using our `securityRules` parameter object from [objects as parameters][objects-as-parameters].</span></span> <span data-ttu-id="b71ed-120">この**変換テンプレート**は、`securityRules` 配列内の各々のオブジェクトを、**呼び出し元テンプレート**の NSG リソースで想定されている JSON スキーマに変換します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-120">Our **transform template** will transform each object in the `securityRules` array into the JSON schema expected by the NSG resource in our **calling template**.</span></span>

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

<span data-ttu-id="b71ed-121">最初に**変換テンプレート**を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-121">Let's look at our **transform template** first.</span></span>

## <a name="transform-template"></a><span data-ttu-id="b71ed-122">変換テンプレート</span><span class="sxs-lookup"><span data-stu-id="b71ed-122">Transform template</span></span>

<span data-ttu-id="b71ed-123">この**変換テンプレート**には、**コレクター テンプレート**から渡される 2 つのパラメーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-123">Our **transform template** includes two parameters that are passed from the **collector template**:</span></span> 
* <span data-ttu-id="b71ed-124">`source` は、プロパティ配列からプロパティ値オブジェクトの 1 つを受け取るオブジェクトです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-124">`source` is an object that receives one of the property value objects from the property array.</span></span> <span data-ttu-id="b71ed-125">この例では、`"securityRules"` 配列からの各オブジェクトが一度に 1 つ渡されます。</span><span class="sxs-lookup"><span data-stu-id="b71ed-125">In our example, each object from the `"securityRules"` array will be passed in one at a time.</span></span>
* <span data-ttu-id="b71ed-126">`state` は、以前の変換のすべてを連結した結果を受け取る配列です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-126">`state` is an array that receives the concatenated results of all the previous transforms.</span></span> <span data-ttu-id="b71ed-127">これは、変換された JSON のコレクションです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-127">This is the collection of transformed JSON.</span></span>

<span data-ttu-id="b71ed-128">パラメーターは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="b71ed-128">Our parameters look like this:</span></span>

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

<span data-ttu-id="b71ed-129">このテンプレートではまた、`instance` という名前の変数を定義します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-129">Our template also defines a variable named `instance`.</span></span> <span data-ttu-id="b71ed-130">これは `source` オブジェクトを、必須の JSON スキーマに実際に変換します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-130">It performs the actual tranform of our `source` object into the required JSON schema:</span></span>

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

<span data-ttu-id="b71ed-131">最後に、テンプレートの `output` が、`state` パラメーターの収集された変換を `instance` 変数によって実行される現在の変換と連結します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-131">Finally, the `output` of our template concatenates the collected transforms of our `state` parameter with the current transform performed by our `instance` variable:</span></span>

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

<span data-ttu-id="b71ed-132">次に**コレクター テンプレート**で、パラメーター値をどのように渡しているか見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-132">Next, let's take a look at our **collector template** to see how it passes in our parameter values.</span></span>

## <a name="collector-template"></a><span data-ttu-id="b71ed-133">コレクター テンプレート</span><span class="sxs-lookup"><span data-stu-id="b71ed-133">Collector template</span></span>

<span data-ttu-id="b71ed-134">この**コレクター テンプレート**には 3 つのパラメーターが含まれています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-134">Our **collector template** includes three parameters:</span></span>
* <span data-ttu-id="b71ed-135">`source` は完全なパラメーター オブジェクトの配列です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-135">`source` is our complete parameter object array.</span></span> <span data-ttu-id="b71ed-136">**呼び出し元テンプレート**によって渡されます。</span><span class="sxs-lookup"><span data-stu-id="b71ed-136">It's passed in by the **calling template**.</span></span> <span data-ttu-id="b71ed-137">これは**変換テンプレート**にある `source` パラメーターと同じ名前ですが、すでにお気づきのように 1 つの大きな違いがあります。これは完全な配列ですが、**変換テンプレート**には一度にこの配列の 1 要素のみを渡しています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-137">This has the same name as the `source` parameter in our **transform template** but there is one key difference that you may have already noticed: this is the complete array, but we only pass one element of this array to the **transform template** at a time.</span></span>
* <span data-ttu-id="b71ed-138">`transformTemplateUri` は、この**変換テンプレート**の URI です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-138">`transformTemplateUri` is the URI of our **transform template**.</span></span> <span data-ttu-id="b71ed-139">テンプレートが再利用できるように、ここではパラメーターとして定義しています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-139">We're defining it as a parameter here for template reusability.</span></span>
* <span data-ttu-id="b71ed-140">`state` は、**変換テンプレート**に渡す、最初は空である配列です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-140">`state` is an initially empty array that we pass to our **transform template**.</span></span> <span data-ttu-id="b71ed-141">コピー ループが完了すると、変換されたパラメーター オブジェクトのコレクションを格納します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-141">It stores the collection of transformed parameter objects when the copy loop is complete.</span></span>

<span data-ttu-id="b71ed-142">パラメーターは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="b71ed-142">Our parameters look like this:</span></span>

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

<span data-ttu-id="b71ed-143">次に、`count` という名前の変数を定義します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-143">Next, we define a variable named `count`.</span></span> <span data-ttu-id="b71ed-144">その値は、`source` パラメーター オブジェクトの配列の長さです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-144">Its value is the length of the `source` parameter object array:</span></span>

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

<span data-ttu-id="b71ed-145">お察しのようにコピー ループでのイテレーションの数に使用しています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-145">As you might suspect, we use it for the number of iterations in our copy loop.</span></span>

<span data-ttu-id="b71ed-146">ここで、リソースを見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-146">Now let's take a look at our resources.</span></span> <span data-ttu-id="b71ed-147">2 つのリソースを定義します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-147">We define two resources:</span></span>
* <span data-ttu-id="b71ed-148">`loop-0` はコピー ループの 0 から始まるリソースです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-148">`loop-0` is the zero-based resource for our copy loop.</span></span>
* <span data-ttu-id="b71ed-149">`loop-` は `copyIndex(1)` 関数の結果と連結され、リソースに、`1` で始まるイテレーションに基づく一意の名前を生成します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-149">`loop-` is concatenated with the result of the `copyIndex(1)` function to generate a unique iteration-based name for our resource, starting with `1`.</span></span>

<span data-ttu-id="b71ed-150">リソースは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="b71ed-150">Our resources look like this:</span></span>

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

<span data-ttu-id="b71ed-151">入れ子になったテンプレート内の**変換テンプレート**に渡しているパラメーターを詳しく見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-151">Let's take a closer look at the parameters we're passing to our **transform template** in the nested template.</span></span> <span data-ttu-id="b71ed-152">`source` パラメーターは `source` パラメーター オブジェクトの配列にある現在のオブジェクトを渡していることを思い出してください。</span><span class="sxs-lookup"><span data-stu-id="b71ed-152">Recall from earlier that our `source` parameter passes the current object in the `source` parameter object array.</span></span> <span data-ttu-id="b71ed-153">`state` パラメーターは、コレクションが行われる場所です。なぜなら、コピー ループの前のイテレーションの出力を受け取るからです&mdash;`reference()` 関数が `copyIndex()` 関数をパラメーターなしで使用して、前にリンクされたテンプレート オブジェクトの `name` を参照すること&mdash;そして、現在のイテレーションに渡すことに注目してください。</span><span class="sxs-lookup"><span data-stu-id="b71ed-153">The `state` parameter is where the collection happens, because it takes the output of the previous iteration of our copy loop&mdash;notice that the `reference()` function uses the `copyIndex()` function with no parameter to reference the `name` of our previous linked template object&mdash;and passes it to the current iteration.</span></span>

<span data-ttu-id="b71ed-154">最後に、テンプレートの `output` は、**変換テンプレート**の最後のイテレーションの `output` を返します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-154">Finally, the `output` of our template returns the `output` of the last iteration of our **transform template**:</span></span>

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
<span data-ttu-id="b71ed-155">**変換テンプレート**の最後のイテレーションの `output` を**呼び出し元テンプレート**に返すのは、それを `source` パラメーターに格納していたように見えるので、直感に反するかもしれません。</span><span class="sxs-lookup"><span data-stu-id="b71ed-155">It may seem counterintuitive to return the `output` of the last iteration of our **transform template** to our **calling template** because it appeared we were storing it in our `source` parameter.</span></span> <span data-ttu-id="b71ed-156">しかし、これは変換されたプロパティ オブジェクトの完全な配列を持つ**変換テンプレート**の最後のイテレーションであることを思い出してください。それこそが返すものです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-156">However, remember that it's the last iteration of our **transform template** that holds the complete array of transformed property objects, and that's what we want to return.</span></span>

<span data-ttu-id="b71ed-157">最後に、**呼び出し元テンプレート**から**コレクター テンプレート**を呼び出す方法を見てみましょう。</span><span class="sxs-lookup"><span data-stu-id="b71ed-157">Finally, let's take a look at how to call the **collector template** from our **calling template**.</span></span>

## <a name="calling-template"></a><span data-ttu-id="b71ed-158">呼び出し元テンプレート</span><span class="sxs-lookup"><span data-stu-id="b71ed-158">Calling template</span></span>

<span data-ttu-id="b71ed-159">この**呼び出し元テンプレート**は、`networkSecurityGroupsSettings` という名前の単一パラメーターを定義します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-159">Our **calling template** defines a single parameter named `networkSecurityGroupsSettings`:</span></span>

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

<span data-ttu-id="b71ed-160">次に、テンプレートは `collectorTemplateUri` という名前の単一の変数を定義します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-160">Next, our template defines a single variable named `collectorTemplateUri`:</span></span>

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

<span data-ttu-id="b71ed-161">予想通り、これはリンクされたテンプレート リソースによって使用される**コレクター テンプレート**の URI です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-161">As you would expect, this is the URI for the **collector template** that will be used by our linked template resource:</span></span>

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

<span data-ttu-id="b71ed-162">**コレクター テンプレート**に 2 つのパラメーターを渡します。</span><span class="sxs-lookup"><span data-stu-id="b71ed-162">We pass two parameters to the **collector template**:</span></span>
* <span data-ttu-id="b71ed-163">`source` はプロパティ オブジェクトの配列です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-163">`source` is our property object array.</span></span> <span data-ttu-id="b71ed-164">この例では、`networkSecurityGroupsSettings` パラメーターです。</span><span class="sxs-lookup"><span data-stu-id="b71ed-164">In our example, it's our `networkSecurityGroupsSettings` parameter.</span></span>
* <span data-ttu-id="b71ed-165">`transformTemplateUri` は**コレクター テンプレート**の URI で定義した変数です。</span><span class="sxs-lookup"><span data-stu-id="b71ed-165">`transformTemplateUri` is the variable we just defined with the URI of our **collector template**.</span></span>

<span data-ttu-id="b71ed-166">最後に、`Microsoft.Network/networkSecurityGroups` リソースで、`collector` がリンクされたテンプレート リソースの `output` をその `securityRules` プロパティに直接割り当てます。</span><span class="sxs-lookup"><span data-stu-id="b71ed-166">Finally, our `Microsoft.Network/networkSecurityGroups` resource directly assigns the `output` of the `collector` linked template resource to its `securityRules` property:</span></span>

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a><span data-ttu-id="b71ed-167">次のステップ</span><span class="sxs-lookup"><span data-stu-id="b71ed-167">Next steps</span></span>

* <span data-ttu-id="b71ed-168">この手法は、[テンプレート構成ブロックのプロジェクト](https://github.com/mspnp/template-building-blocks)と [Azure 参照アーキテクチャ](/azure/architecture/reference-architectures/)で実装されています。</span><span class="sxs-lookup"><span data-stu-id="b71ed-168">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="b71ed-169">これらを使用して、独自のアーキテクチャを作成したり、この参照アーキテクチャのいずれかをデプロイしたりできます。</span><span class="sxs-lookup"><span data-stu-id="b71ed-169">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
