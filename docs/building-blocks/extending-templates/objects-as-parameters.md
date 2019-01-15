---
title: Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する
description: Azure Resource Manager テンプレートの機能を拡張して、オブジェクトをパラメーターとして使用する方法について説明します。
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: f0826d8ed1ce446d295ebdacc66d8b0bef0b0dec
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111208"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する

[Azure Resource Manager テンプレートを作成する][azure-resource-manager-create-template]場合、テンプレートに直接リソース プロパティ値を指定するか、または、デプロイ時にパラメーターを定義して値を指定することができます。 小規模なデプロイでは各々のプロパティ値にパラメーターを使用しても問題ありませんが、デプロイあたり 255 個のパラメーターという制限があります。 大規模かつ複雑なデプロイになると、パラメーターが足りなくなる可能性があります。

この問題を解決する 1 つの方法が、値の代わりにオブジェクトをパラメーターとして使用することです。 これを行うには、テンプレートのパラメーターを定義し、デプロイ時に 1 つの値ではなく、JSON オブジェクトを指定します。 そして、テンプレート内の [`parameter()` 関数][azure-resource-manager-functions]とドット演算子を使用して、パラメーターのサブプロパティを参照します。

仮想ネットワーク リソースをデプロイする例を見てみましょう。 まず、テンプレートで `VNetSettings` パラメーターを指定し、`type` を `object` に設定します。

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```

次に、`VNetSettings` オブジェクトに値を指定してみましょう。

> [!NOTE]
> デプロイ時にパラメーター値を指定する方法については、「[Azure Resource Manager テンプレートの構造と構文の詳細][azure-resource-manager-authoring-templates]」の**パラメーター**のセクションを参照してください。

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

1 つのパラメーターが実際には次の 3 つのサブプロパティ、`name`、`addressPrefixes`、および`subnets` を指定することがわかります。 これらのサブプロパティの各々は、1 つの値または他のサブプロパティを指定します。 その結果、1 つのパラメーターが、仮想ネットワークのデプロイに必要なすべての値を指定します。

では、テンプレートの残りの部分を見て、`VNetSettings` オブジェクトがどのように使用されているか確認しましょう。

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

`VNetSettings` オブジェクトの値は、`parameters()` 関数を `[]` 配列インデクサーおよびドット演算子とともに使用して、仮想ネットワーク リソースに必要なプロパティに適用されています。 この方法は、リソースにパラメーター オブジェクトの値を静的に適用する場合に有効です。 ただし、デプロイ時にプロパティ値の配列を動的に割り当てる場合は、[コピー ループ][azure-resource-manager-create-multiple-instances]を使用します。 コピー ループを使用するには、リソースのプロパティ値の JSON 配列を指定します。すると、コピー ループが値をリソースのプロパティに動的に適用します。

動的なアプローチをする場合は、注意すべき問題が 1 つあります。 その問題を示すために、プロパティ値の典型的な配列を見てみましょう。 この例では、プロパティの値は変数に格納されています。 ここでは 2 つの配列があります&mdash; 1 つは `firstProperty` という名前、もう 1 つは `secondProperty` という名前です。

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

コピー ループを使用して、変数のプロパティにアクセスする方法を見てみましょう。

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

`copyIndex()` 関数がコピー ループの現在のイテレーションを返すので、それを 2 つの配列のそれぞれにインデックスとして同時に使用します。

これは、2 つの配列が同じ長さである場合は問題なく動作します。 もし間違って 2 つの配列の長さが異なっていると、問題が起きます&mdash;この場合、テンプレートはデプロイ時に検証に失敗します。 この問題は、1 つのオブジェクトにすべてのプロパティを含めることで回避できます。値がない場合に簡単に気づくことができるからです。 たとえば、`propertyObject` 配列の各要素が前からの `firstProperty` および `secondProperty` 配列の組み合わせであるような、別のパラメーター オブジェクトを見てみましょう。

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

配列内の 3 つ目の要素に気づいたでしょうか。 `number` プロパティがありませんが、このようにしてパラメーター値を作成すると、それが足りないことに簡単に気づくことができます。

## <a name="using-a-property-object-in-a-copy-loop"></a>コピー ループ内でプロパティ オブジェクトを使用する

この方法は、[serial copy loop][azure-resource-manager-create-multiple] と組み合わせる場合、特に、子リソースをデプロイするのにさらに便利です。

これを示すために、2 つのセキュリティ規則で[ネットワーク セキュリティ グループ (NSG)][nsg] をデプロイするテンプレートを見てみましょう。

最初に、パラメーターを見てみましょう。 テンプレートを見ると、`securityRules` という名前の配列を含む `networkSecurityGroupsSettings` という名前の 1 つのパラメーターを定義したことがわかります。 この配列には、セキュリティ規則の設定の数を指定する 2 つの JSON オブジェクトが含まれています。

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

ここで、テンプレートを見てみましょう。 最初の `NSG1` という名前のリソースは、NSG をデプロイしています。 2 番目の `loop-0` という名前のリソースは 2 つの関数を実行します。1 つは、NSG に対して `dependsOn` であるため、そのデプロイは `NSG1` が完了するまで開始されません。また、これはシーケンシャル ループの最初のイテレーションです。 3 番目のリソースは入れ子になったテンプレートで、最後の例のように、パラメーター値のオブジェクトを使用してセキュリティ規則をデプロイします。

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

`securityRules` 子リソースでのプロパティ値の指定方法を詳しく見てみましょう。 すべてのプロパティは `parameter()` 関数を使用して参照されているので、ドット演算子を使用して、イテレーションの現在値によってインデックス付けされた `securityRules` 配列を参照します。 最後に、別のドット演算子を使用して、オブジェクトの名前を参照します。

## <a name="try-the-template"></a>テンプレートを試行する

テンプレートの例は [GitHub][github] で入手できます。 テンプレートをデプロイするには、リポジトリを複製し、次の [Azure CLI][cli] コマンドを実行します。

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a>次の手順

- オブジェクト配列を反復処理するテンプレートを作成し、これを JSON スキーマに変換する方法を確認します。 「[Azure Resource Manager テンプレートでプロパティのトランスフォーマーとコレクターを実装する](./collector.md)」を参照してください

<!-- links -->

[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
