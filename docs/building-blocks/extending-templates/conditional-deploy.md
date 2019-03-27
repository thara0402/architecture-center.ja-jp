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
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートのリソースを条件付きでデプロイする

パラメーター値が存在するかどうかなどの条件に基づいて、リソースをデプロイするようにテンプレートを設計する必要があるシナリオがいくつかあります。 たとえば、テンプレートは仮想ネットワークをデプロイしたり、ピアリング用の他の仮想ネットワークを指定するパラメーターを含んでいたりする可能性があります。 ピアリングのパラメーター値を指定していない場合は、Resource Manager によってピアリング リソースをデプロイする必要はありません。

これを行うには、リソースの [condition 要素][azure-resource-manager-condition]を使用して、パラメーター配列の長さをテストします。 長さが 0 の場合は、`false` が返されてデプロイが阻止されますが、0 より大きな値の場合は `true` が返され、デプロイが許可されます。

## <a name="example-template"></a>テンプレートの例

これを実証するテンプレートの例を見てみましょう。 このテンプレートでは [condition 要素][azure-resource-manager-condition]を使用して、`Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースのデプロイを制御します。 このリソースは、同じリージョン内の 2 つの Azure Virtual Network 間でピアリングを作成します。

テンプレートの各セクションを見てみましょう。

`parameters` 要素は、`virtualNetworkPeerings` という名前の単一パラメーターを定義します。

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

`virtualNetworkPeerings` パラメーターは `array` であり、次のスキーマがあります。

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

このパラメーターのプロパティが、[仮想ネットワークのピアリングに関連する設定][vnet-peering-resource-schema]を指定します。 `resources` セクションで `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースを指定する場合に、これらのプロパティの値が提供されます。

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

このテンプレートのこの部分では、2 つの処理が行われています。 最初に、デプロイされている実際のリソースは type が `Microsoft.Resources/deployments` のインライン テンプレートであり、実際に `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` をデプロイする独自のテンプレートが含まれています。

このインライン テンプレートに対する `name` は、`copyIndex()` の現在の繰り返しを接頭辞 `vnp-` で連結することによって一意になります。

`condition` 要素は、`greater()` 関数が `true` として評価された場合にリソースが処理される必要があることを指定しています。 ここでは、`virtualNetworkPeerings` パラメーター配列が 0 よりも `greater()` であるかどうかをテストしています。 0 より大きい場合は、評価結果が `true` となり、`condition` が満たされます。 それ以外の場合は、`false` となります。

次に、`copy` ループを指定します。 これは、`serial` ループであり、最後のリソースがデプロイされるまで待機している各リソースで、ループが順番に実行されることを意味しています。 `count` プロパティは、ループが繰り返される回数を指定します。 このプロパティにはデプロイ対象のリソースを指定するパラメーター オブジェクトが含まれるため、これは通常、`virtualNetworkPeerings` 配列の長さに設定されます。 ただしその場合、配列が空だと、存在しないプロパティにアクセスしようとしていることを Resource Manager が通知するため、検証が失敗します。 しかし、これは回避できます。 必要な変数について説明します。

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

`workaround` 変数には、`true` および `false` という 2 つのプロパティが含まれます。 `true` プロパティは、評価されて `virtualNetworkPeerings`パラメーター配列の値になります。 `false` プロパティの評価結果は、Resource Manager での表示を想定している名前付きのプロパティも含めて空のオブジェクトになります。`false` は、`virtualNetworkPeerings` パラメーターと同様、実際には検証を満たす配列であることに注意してください。

`peerings` 変数は `workaround` 変数をもう一度使用し、`virtualNetworkPeerings` パラメーター配列の長さが 0 より大きいかどうかをテストします。 0 より大きい場合、`string` は `true` として評価され、`workaround` 変数は `virtualNetworkPeerings` パラメーター配列として評価されます。 それ以外の場合、評価結果が `false` となり、`workaround` 変数の評価結果によって配列の最初の要素が空のオブジェクトになります。

変数の問題について対処を行ったため、`Microsoft.Network/virtualNetworks/virtualNetworkPeerings` リソースのデプロイを入れ子になったテンプレート内で指定し、`name` と `properties` を `virtualNetworkPeerings` パラメーター配列から渡すことができます。 これは、リソースの `properties` 要素で入れ子にされた `template` 要素で確認できます。

## <a name="try-the-template"></a>テンプレートを試行する

テンプレートの例は [GitHub][github] で入手できます。 テンプレートをデプロイするには、次の [Azure CLI][cli] コマンドを実行します。

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a>次の手順

* スカラー値ではなくオブジェクトを、テンプレート パラメーターとして使用します。 「[Azure Resource Manager テンプレートのパラメーターとしてオブジェクトを使用する](./objects-as-parameters.md)」を参照してください

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-manager-templates-resources#condition
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
