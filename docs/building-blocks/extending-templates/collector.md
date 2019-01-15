---
title: Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する
description: Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する方法について説明します。
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: 1a6a01ee513609132d8522a79ccb81b7938651b5
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113809"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する

「[Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する][objects-as-parameters]」では、リソース プロパティ値をオブジェクトに格納し、デプロイ時にリソースに適用する方法を学びました。 これはパラメーターの管理には非常に便利な方法ですが、テンプレートで使用するたびに、オブジェクトのプロパティをリソースのプロパティにマップする必要があります。

これを回避するには、オブジェクト配列を反復処理してリソースで想定されている JSON スキーマに変換するような、プロパティ変換とコレクターのテンプレートを実装します。

> [!IMPORTANT]
> この方法では、Resource Manager のテンプレートと関数について深い理解があることが必要です。

[ネットワーク セキュリティ グループ (NSG)][nsg] をデプロイする例で、プロパティのコレクターとトランスフォーマーの実装方法を見てみましょう。 次の図は、これらのテンプレートとテンプレート内のリソースとの間のリレーションシップを示しています。

![プロパティのコレクターとトランスフォーマーのアーキテクチャ](../_images/collector-transformer.png)

この**呼び出し元テンプレート**には 2 つのリソースが含まれています。

- **コレクター テンプレート**を呼び出す、テンプレートのリンク。
- デプロイする NSG リソース。

この**コレクター テンプレート**には 2 つのリソースが含まれています。

- **アンカー** リソース。
- コピー ループ内で変換テンプレートを呼び出す、テンプレートのリンク。

この**変換テンプレート**には 1 つのリソースが含まれています: **メイン テンプレート**で `source` JSON を NSG リソースで想定されている JSON スキーマに変換する変数を持つ、空のテンプレート。

## <a name="parameter-object"></a>パラメーター オブジェクト

[パラメーターとしてのオブジェクト][objects-as-parameters]に関するページにある、`securityRules` パラメーター オブジェクトを使用します。 この**変換テンプレート**は、`securityRules` 配列内の各々のオブジェクトを、**呼び出し元テンプレート**の NSG リソースで想定されている JSON スキーマに変換します。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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

最初に**変換テンプレート**を見てみましょう。

## <a name="transform-template"></a>変換テンプレート

この**変換テンプレート**には、**コレクター テンプレート**から渡される 2 つのパラメーターが含まれています。

- `source` は、プロパティ配列からプロパティ値オブジェクトの 1 つを受け取るオブジェクトです。 この例では、`"securityRules"` 配列からの各オブジェクトが一度に 1 つ渡されます。
- `state` は、以前の変換のすべてを連結した結果を受け取る配列です。 これは、変換された JSON のコレクションです。

パラメーターは次のようになります。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

このテンプレートではまた、`instance` という名前の変数を定義します。 これは `source` オブジェクトを、必須の JSON スキーマに実際に変換します。

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

最後に、テンプレートの `output` が、`state` パラメーターの収集された変換を `instance` 変数によって実行される現在の変換と連結します。

```json
    "resources": [],
    "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

次に**コレクター テンプレート**で、パラメーター値をどのように渡しているか見てみましょう。

## <a name="collector-template"></a>コレクター テンプレート

この**コレクター テンプレート**には 3 つのパラメーターが含まれています。

- `source` は完全なパラメーター オブジェクトの配列です。 **呼び出し元テンプレート**によって渡されます。 これは**変換テンプレート**にある `source` パラメーターと同じ名前ですが、すでにお気づきのように 1 つの大きな違いがあります。これは完全な配列ですが、**変換テンプレート**には一度にこの配列の 1 要素のみを渡しています。
- `transformTemplateUri` は、この**変換テンプレート**の URI です。 テンプレートが再利用できるように、ここではパラメーターとして定義しています。
- `state` は、**変換テンプレート**に渡す、最初は空である配列です。 コピー ループが完了すると、変換されたパラメーター オブジェクトのコレクションを格納します。

パラメーターは次のようになります。

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
```

次に、`count` という名前の変数を定義します。 その値は、`source` パラメーター オブジェクトの配列の長さです。

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

お察しのようにコピー ループでのイテレーションの数に使用しています。

ここで、リソースを見てみましょう。 2 つのリソースを定義します。

- `loop-0` はコピー ループの 0 から始まるリソースです。
- `loop-` は `copyIndex(1)` 関数の結果と連結され、リソースに、`1` で始まるイテレーションに基づく一意の名前を生成します。

リソースは次のようになります。

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
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

入れ子になったテンプレート内の**変換テンプレート**に渡しているパラメーターを詳しく見てみましょう。 `source` パラメーターは `source` パラメーター オブジェクトの配列にある現在のオブジェクトを渡していることを思い出してください。 `state` パラメーターは、コレクションが行われる場所です。なぜなら、コピー ループの前のイテレーションの出力を受け取るからです&mdash;`reference()` 関数が `copyIndex()` 関数をパラメーターなしで使用して、前にリンクされたテンプレート オブジェクトの `name` を参照すること&mdash;そして、現在のイテレーションに渡すことに注目してください。

最後に、テンプレートの `output` は、**変換テンプレート**の最後のイテレーションの `output` を返します。

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```

**変換テンプレート**の最後のイテレーションの `output` を**呼び出し元テンプレート**に返すのは、それを `source` パラメーターに格納していたように見えるので、直感に反するかもしれません。 しかし、これは変換されたプロパティ オブジェクトの完全な配列を持つ**変換テンプレート**の最後のイテレーションであることを思い出してください。それこそが返すものです。

最後に、**呼び出し元テンプレート**から**コレクター テンプレート**を呼び出す方法を見てみましょう。

## <a name="calling-template"></a>呼び出し元テンプレート

この**呼び出し元テンプレート**は、`networkSecurityGroupsSettings` という名前の単一パラメーターを定義します。

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

次に、テンプレートは `collectorTemplateUri` という名前の単一の変数を定義します。

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

予想通り、これはリンクされたテンプレート リソースによって使用される**コレクター テンプレート**の URI です。

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('collectorTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

**コレクター テンプレート**に 2 つのパラメーターを渡します。

- `source` はプロパティ オブジェクトの配列です。 この例では、`networkSecurityGroupsSettings` パラメーターです。
- `transformTemplateUri` は**コレクター テンプレート**の URI で定義した変数です。

最後に、`Microsoft.Network/networkSecurityGroups` リソースで、`collector` がリンクされたテンプレート リソースの `output` をその `securityRules` プロパティに直接割り当てます。

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('collector').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('collector').outputs.result.value]"
      }

  }
```

## <a name="try-the-template"></a>テンプレートを試行する

テンプレートの例は [GitHub][github] で入手できます。 テンプレートをデプロイするには、リポジトリを複製し、次の [Azure CLI][cli] コマンドを実行します。

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example4-collector
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example4-collector/deploy.json \
    --parameters deploy.parameters.json
```

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
