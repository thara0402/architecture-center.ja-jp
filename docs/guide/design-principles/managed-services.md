---
title: 管理対象サービスの使用
description: なるべく、サービスとしてのインフラストラクチャ (IaaS) ではなくサービスとしてのプラットフォーム (PaaS) を使用する
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: f6777a19e126a8a7f64be05dfad9bc503d27b1c3
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325776"
---
# <a name="use-managed-services"></a>管理対象サービスの使用

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>なるべく、サービスとしてのインフラストラクチャ (IaaS) ではなくサービスとしてのプラットフォーム (PaaS) を使用する

IaaS は部品箱のようなサービスです。 何でも構築できますが、自分で組み立てる必要があります。 管理対象サービスのほうが、構成や管理が簡単です。 VM をプロビジョニングしたり、Vnet を設定したり、修正プログラムや更新プログラムを管理する手間が省けるだけでなく、VM 上でのソフトウェア実行に関するその他すべてのオーバーヘッドが解消されます。

たとえば、アプリケーションでメッセージ キューを使用する必要があるとします。 その場合、RabbitMQ などのツールを使用して VM 上に独自のメッセージング サービスを設定することもできますが、 Azure Service Bus なら、信頼性のあるメッセージング機能が既にサービスとして提供されていますし、設定もより簡単です。 Service Bus 名前空間を作成し (これはデプロイ スクリプトの一部として実行できます)、 クライアント SDK を使用して Service Bus を呼び出すだけです。 

もちろん、アプリケーションの要件によっては、IaaS アプローチのほうが適している場合もあるでしょう。 ただし、アプリケーションが IaaS ベースの場合でも、管理対象サービスを導入したほうが妥当な部分がないかどうかを確認するようにしてください。 具体的には、キャッシュ、キュー、データ ストレージなどについて確認を行ってください。

| 以下を実行する代わりに... | 以下の使用を検討してください... |
|-----------------------|-------------|
| Active Directory | Azure Active Directory Domain Services |
| Elasticsearch | Azure Search |
| Hadoop | HDInsight |
| IIS | App Service |
| MongoDB | Cosmos DB |
| Redis | Azure Redis Cache |
| SQL Server | Azure SQL Database |


