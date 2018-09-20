---
title: Azure での保険金請求イメージの分類
description: イメージ処理を Azure アプリケーションに組み込む実証済みのシナリオ。
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 0ca0b46e83219afc5e22c2ac6467bf4be945c97a
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389164"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Azure での保険金請求イメージの分類

このシナリオの例は、イメージを処理する必要があるビジネスに適用できます。

応用の可能性としては、ファッション Web サイトのイメージの分類、保険金請求のテキストおよびイメージの分析、ゲームのスクリーンショットからの利用統計情報の把握などが挙げられます。 従来、企業では、機械学習モデルで専門知識を開発したうえで、そのモデルをトレーニングし、最終的にカスタム プロセスによってイメージを実行し、イメージからデータを取得していました。

Computer Vision API、Azure Functions などの Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減できるほか、既に Microsoft で開発済みの、Cognitive Services でのイメージ処理に関する専門知識を活用することができます。 この例では、特にイメージ処理ユース ケースを扱っています。 別の AI ニーズがある場合は、一連の [Cognitive Services][cognitive-docs] について検討してください。

## <a name="related-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* ファッション Web サイトのイメージを分類する。
* ゲームのスクリーンショットの利用統計情報を分類する。

## <a name="architecture"></a>アーキテクチャ

![インテリジェントなアプリのアーキテクチャ - Computer Vision][architecture-computer-vision]

このシナリオでは、Web またはモバイル アプリケーションのバックエンド コンポーネントに対応できます。 シナリオのデータ フローは次のとおりです。

1. Azure Functions が、API レイヤーとして機能します。 これらの API により、アプリケーションでイメージをアップロードし、Cosmos DB からデータを取得できます。

2. API 呼び出しでアップロードされたイメージが Blob Storage に格納されます。

3. 新しいファイルを Blob Storage に追加すると、EventGrid 通知がトリガーされ、Azure 関数に送信されます。

4. Azure Functions により、新しくアップロードされたファイルへのリンクが、分析のために Computer Vision API に送信されます。

5. Computer Vision API からデータが返されたら、Azure Functions によって、Cosmos DB のエントリに分析の結果とイメージ メタデータが保持されます。

### <a name="components"></a>コンポーネント

* [Computer Vision API][computer-vision-docs] は、Cognitive Services スイートに含まれ、各イメージに関する情報の取得に使用されます。

* [Azure Functions][functions-docs] は、Web アプリケーションにバックエンド API を、また、アップロードされたイメージにイベント処理を提供します。

* [Event Grid][eventgrid-docs] は、新しいイメージが Blob Storage にアップロードされたときに、イベントをトリガーします。 その後、イメージは Azure 関数で処理されます。

* [Blob Storage][storage-docs] には、Web アプリケーションにアップロードされたすべてのイメージ ファイルと、Web アプリケーションによって使用される任意の静的ファイルが格納されます。

* [Cosmos DB][cosmos-docs] には、Computer Vision API からの処理結果を含め、アップロードされた各イメージに関するメタデータが格納されます。

## <a name="alternatives"></a>代替手段

* [Custom Vision Service][custom-vision-docs]。 Computer Vision API は、一連の[分類ベースのカテゴリ][cv-categories]を返します。 Computer Vision API によって返されていない情報を処理する必要がある場合は、Custom Vision Service を検討してください。これにより、カスタム イメージ分類子を構築できます。

* [Azure Search][azure-search-docs]。 特定の条件を満たすイメージを検索するために、ご自身のユース ケースでメタデータにクエリを実行する必要がある場合は、Azure Search を検討してください。 現在、プレビュー中の[コグニティブ検索][cognitive-search]では、このワークフローがシームレスに統合されます。

## <a name="considerations"></a>考慮事項

### <a name="scalability"></a>スケーラビリティ

このシナリオの例で使用されるコンポーネントの大半が、自動スケーリングされるマネージド サービスです。 注目すべき例外がいくつかあります。Azure Functions のインスタンス数は最大 200 個に制限されています。 この限界を超えてスケーリングする場合は、複数のリージョンまたはアプリ プランを検討してください。

Cosmos DB は、プロビジョニング済み要求ユニット (RU) の点からいうと、自動スケーリングされません。  ご自身の要件の推定に関するガイダンスについては、Microsoft ドキュメントの[要求ユニット][request-units]に関するページをご覧ください。 Cosmos DB でスケーリングのメリットを十分に活用するには、[パーティション キー][partition-key]についても確認する必要があります。

NoSQL データベースでは、可用性、スケーラビリティ、および分断性と引き換えに、一貫性 (CAP 定理という意味で) が犠牲になることがよくあります。  ただし、このシナリオの例では、キーと値のデータ モデルが使用され、ほとんどの操作が本質的にアトミックであるため、トランザクションの一貫性が必要になることはほとんどありません。 追加のガイダンスについては、Azure アーキテクチャ センターの「[適切なデータ ストアの選択](../../guide/technology-choices/data-store-overview.md)」を参照してください。

スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

[マネージド サービス ID][msi] (MSI) は、ご自身のアカウントの内部にあり、ご自身の Azure Functions に割り当てられている他のリソースへのアクセスを提供するときに使用されます。 これらの ID の必要なリソースへのアクセスのみを許可して、余分なものがお使いの関数 (およびご自身の顧客) に公開されないようにします。  

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

このシナリオのすべてのコンポーネントが管理されているため、すべてについて、リージョン レベルの回復性が自動的に確保されます。

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

トラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています (すべてのイメージのサイズが 100 KB であると想定しています)。

* [Small][pricing]: この価格例は、1 か月あたり &lt;5,000 件のイメージの処理に対応します。
* [Medium][medium-pricing]: この価格例は、1 か月あたり 500,000 件のイメージの処理に対応します。
* [Large][large-pricing]: この価格例は、1 か月あたり 50,000,000 件のイメージの処理に対応します。

## <a name="related-resources"></a>関連リソース

このシナリオのガイド付きラーニング パスについては、「[Build a serverless web app in Azure (Azure でのサーバーレス Web アプリの構築)][serverless]」を参照してください。  

このシナリオの例を運用環境にデプロイする前に、Azure Functions の[ベスト プラクティス][functions-best-practices]に関するページをご確認ください。

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data