---
title: Azure Resource Manager テンプレートのリソースを更新する
description: Azure Resource Manager テンプレートの機能を拡張して、リソースを更新する方法について説明します。
author: petertay
ms.date: 10/31/2018
ms.openlocfilehash: 927826283163b2ae45575035168d6238de98dc00
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113418"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートのリソースを更新する

デプロイ中にリソースの更新が必要になるシナリオはいくつかあります。 このシナリオは、他の依存リソースが作成されるまで、リソースの一部のプロパティを指定できないときに発生する可能性があります。 たとえば、ロード バランサーのバックエンド プールを作成する場合、仮想マシン (VM) のネットワーク インターフェイス (NIC) をバックエンド プールに含めるために、その NIC を更新することがあります。 そして、Resource Manager でデプロイ中のリソース更新がサポートされている間、エラーが発生しないように、また、デプロイが更新として確実に処理されるようにするには、テンプレートを適切に設計する必要があります。

最初に、テンプレートでそのリソースを 1 回参照してリソースを作成し、その後同じ名前でそのリソースを参照し、後でそのリソースを更新します。 しかし、テンプレート内で同じ名前のリソースが 2 つあると、Resource Manager は例外をスローします。 このエラーが発生しないようにするには、更新されたリソースを、サブテンプレートとしてリンクまたは挿入されている 2 番目のテンプレートで、`Microsoft.Resources/deployments` リソース タイプを使用して指定します。

次に、入れ子になっているテンプレートに、変更する既存のプロパティの名前を指定するか、追加するプロパティの新しい名前を指定する必要があります。 元のプロパティと、そのプロパティの元の値も指定しなければなりません。 元のプロパティと値を指定しないと、新しいリソースを作成する必要があるものと Resource Manager からみなされ、元のリソースは削除されます。

## <a name="example-template"></a>テンプレートの例

これを実証するテンプレートの例を見てみましょう。 このテンプレートでは、`firstSubnet` という名前のサブネットを含む `firstVNet` という名前の仮想ネットワークをデプロイします。 そして、`nic1` という名前の仮想ネットワーク インターフェイス (NIC) をデプロイし、サブネットに関連付けます。 その後、`secondSubnet` という名前の 2 番目のサブネットを追加することにより、`updateVNet` という名前のデプロイ リソースに、`firstVNet` リソースを更新する入れ子になったテンプレートが追加されます。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

最初に、`firstVNet` リソースのリソース オブジェクトを見てみましょう。 入れ子になったテンプレートで `firstVNet` の設定を再度指定しますのでご注意ください。この理由は、Resource Manager では同じテンプレート内での同じデプロイ名を使用することは許可されず、入れ子になったテンプレートは別のテンプレートと見なされるためです。 `firstSubnet` リソースに値を再度指定することにより、リソース マネージャーは既存のリソースを削除する代わりに更新し、再度デプロイします。 最後に、`secondSubnet` の新しい設定は、この更新中に選択されます。

## <a name="try-the-template"></a>テンプレートを試行する

テンプレートの例は [GitHub][github] で入手できます。 テンプレートをデプロイするには、次の [Azure CLI][cli] コマンドを実行します。

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

デプロイが完了したら、ポータルで指定したリソース グループを開きます。 `firstVNet` という名前の仮想ネットワークと、`nic1` という名前の NIC が表示されます。 `firstVNet`、`subnets` の順にクリックします。 最初に作成された `firstSubnet` のほか、`updateVNet` リソースで追加された `secondSubnet` が表示されます。

![元のサブネットと、更新されたサブネット](../_images/firstVNet-subnets.png)

次に、リソース グループに戻り、`nic1`、`IP configurations` の順にクリックします。 `IP configurations` セクションで、`subnet` が `firstSubnet (10.0.0.0/24)` に設定されています。

![nic1 IP 構成設定](../_images/nic1-ipconfigurations.png)

元の `firstVNet` が、再作成ではなく更新されています。 `firstVNet` が再作成されているとしたら、`nic1` は `firstVNet` に関連付けられていません。

## <a name="next-steps"></a>次の手順

* パラメーター値の有無などの条件に基づいてリソースをデプロイする方法を確認します。 「[Azure Resource Manager テンプレートのリソースを条件付きでデプロイする](./conditional-deploy.md)」を参照してください。

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
