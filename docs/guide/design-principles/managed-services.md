---
title: "管理対象サービスの使用"
description: "なるべく、サービスとしてのインフラストラクチャ (IaaS) ではなくサービスとしてのプラットフォーム (PaaS) を使用する"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 7156c073db3e047fb38e031309ddb637a9e44c02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
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


