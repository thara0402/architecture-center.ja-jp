---
title: 説明 - Azure リソース グループとは
description: リソース グループの内部 Azure 関数について説明します。
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2018
ms.locfileid: "29062080"
---
# <a name="what-is-an-azure-resource-group"></a>Azure リソース グループとは

「[Azure Resource Manager とは](resource-manager-explainer.md)」の説明の記事では、リソースの作成、読み取り、更新、または削除のために呼び出しが実行されるとき、Azure Resource Manager では**リソース グループ ID** を必要とすることを説明しました。 このリソース グループ ID とは、**リソース グループ**のことを指します。 リソース グループとは単に、Azure Resource Manager がリソースをグループ化するために適用する ID です。 このリソース グループ ID によって、Azure Resource Manager はこの ID を共有するリソース グループに対して操作を実行できるようになります。

たとえば、ユーザーは、特定のリソース ID を含めずにリソース グループ ID を指定して、Azure Resource Manager の RESTful API に対する**削除**の呼び出しを行うことができます。 Azure Resource Manager は、指定されたリソース グループ ID ですべてのリソースの内部 Azure データベースを照会し、RESTful API を呼び出して各リソースを削除します。

リソース グループに、別のサブスクリプションからのリソースを含めることはできません。 これは、テナント ID とサブスクリプション ID 間に一対多のリレーションシップがあるためです。複数のサブスクリプションで同じテナントを信頼して認証および承認を提供することは可能ですが、各サブスクリプションでは 1 つのテナントしか信頼できません。 また、サブスクリプション ID とリソース グループ ID 間にも一対多のリレーションシップがあります。複数のリソース グループが同じサブスクリプションに属することは可能ですが、各リソース グループは 1 つのサブスクリプションにしか属することができません。 最後に、リソース グループ ID とリソース ID 間に一対多のリレーションシップがあります。1 つのリソース グループに複数のリソースを含めることは可能ですが、各リソースは 1 つのリソース グループにしか属することができません。

## <a name="next-steps"></a>次の手順

* Azure リソース グループについて学習したので、次は[リソースへのアクセス制限について](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)基本的な知識を備えてください。 基本の導入ステージの一部ではありませんが、中間導入ステージにおいて重要になります。 その後、[最初のリソース グループを作成](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)して、[Azure リソース グループの設計ガイダンス](resource-group.md)を確認できます。
