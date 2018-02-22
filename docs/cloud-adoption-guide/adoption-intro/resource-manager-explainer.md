---
title: "説明: Azure Resource Manager とは"
description: "Azure Resource Manager の内部機能について説明します。"
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-azure-resource-manager"></a>説明: Azure Resource Manager とは

「[Azure のしくみ](azure-explainer.md)」の説明では、Azure の内部アーキテクチャについて説明しました。 このアーキテクチャには、内部の Azure サービスを管理する分散アプリケーションをホストするフロント エンドが含まれています。

Azure フロント エンドには、Azure Resource Manager と呼ばれるサービスが含まれています。 Azure Resource Manager は、作成から削除まで、Azure でホストされるリソースのライフ サイクルを担います。 Powershell、Azure コマンドライン インターフェイス、SDK を使用して Azure Resource Manager を操作する方法はたくさんありますが、これらの各ツールは単純に、Azure Resource Manager でホストされる RESTful API に対するラッパーです。

Azure Resource Manager が提供する RESTful API は、一連の**リソース プロバイダー**に対する一貫性のあるインターフェイスです。 リソース プロバイダーとは単純に、Azure でリソースの作成、読み取り、更新、および削除を行う Azure サービスです。 実際、RESTful API には、これらの各機能に対するメソッドが含まれます。 

RESTful API では、該当のユーザー、**サブスクリプション ID** に加え、新しく **リソース グループ ID** に対するアクセス トークンを必要とします。 リソース グループについては、[リソース グループの説明](resource-group-explainer.md)の記事をご覧ください。 Azure Resource Manager では、アクセス トークンの一部としてエンコードされる**テナント ID** も必要になります。 

リソースを作成する有効な API 呼び出しが受信されると、Azure Resource Manager は指定した領域内の容量を検出し、必要なファイルをステージングの場所にコピーします。 その後、要求はラック内のファブリック コントローラーに送信され、ファブリック コントローラーがリソースを割り当てます。 ファブリック コントローラーは、新しく作成されたリソースの**リソース ID** と共に、成功または失敗の通知で要求に応答します。 これらの 4 つの ID は Azure に内部的に保存され、デプロイされたリソースの一意の識別子として機能します。

## <a name="next-steps"></a>次の手順

* Azure Resource Manager の内部機能について理解したので、[リソース グループについて](resource-group-explainer.md)を学習して、最初のリソース グループの作成に役立ててください。
