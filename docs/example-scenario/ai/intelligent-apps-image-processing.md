---
title: 保険金請求イメージの分類
titleSuffix: Azure Example Scenarios
description: ご使用の Azure アプリケーションに画像処理を組み込みます。
author: david-stanford
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-intelligent-apps-image-processing.png
ms.openlocfilehash: 03ef9d15ec9bf64dc743657e9d1e7a7e275c5a8e
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908326"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Azure での保険金請求イメージの分類

このシナリオは、イメージを処理する必要があるビジネスに関連があります。

応用の可能性としては、ファッション Web サイトのイメージの分類、保険金請求のテキストおよびイメージの分析、ゲームのスクリーンショットからの利用統計情報の把握などが挙げられます。 従来、企業では、機械学習モデルで専門知識を開発したうえで、そのモデルをトレーニングし、最終的にカスタム プロセスによってイメージを実行し、イメージからデータを取得していました。

Computer Vision API、Azure Functions などの Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減できるほか、既に Microsoft で開発済みの、Cognitive Services でのイメージ処理に関する専門知識を活用することができます。 この例では、特にイメージ処理ユース ケースを扱っています。 別の AI ニーズがある場合は、一連の [Cognitive Services](/azure/#pivot=products&panel=ai) について検討してください。

## <a name="relevant-use-cases"></a>関連するユース ケース

その他の関連するユース ケース:

- ファッション Web サイトの画像の分類。
- ゲームのスクリーンショットの利用統計情報の分類。

## <a name="architecture"></a>アーキテクチャ

![イメージ分類のアーキテクチャ][architecture]

このシナリオでは、Web またはモバイル アプリケーションのバックエンド コンポーネントに対応できます。 シナリオのデータ フローは次のとおりです。

1. API レイヤーは、Azure Functions を使用して構築されています。 これらの API により、アプリケーションでイメージをアップロードし、Cosmos DB からデータを取得できます。
2. API 呼び出しでアップロードされたイメージが Blob Storage に格納されます。
3. 新しいファイルを Blob Storage に追加すると、EventGrid 通知がトリガーされ、Azure 関数に送信されます。
4. Azure Functions により、新しくアップロードされたファイルへのリンクが、分析のために Computer Vision API に送信されます。
5. Computer Vision API からデータが返されたら、Azure Functions によって、Cosmos DB のエントリに分析の結果とイメージ メタデータが保持されます。

### <a name="components"></a>コンポーネント

- [Computer Vision API](/azure/cognitive-services/computer-vision/home) は、Cognitive Services スイートに含まれ、各イメージに関する情報の取得に使用されます。
- [Azure Functions](/azure/azure-functions/functions-overview) は、Web アプリケーションにバックエンド API を、また、アップロードされたイメージにイベント処理を提供します。
- [Event Grid](/azure/event-grid/overview) は、新しいイメージが Blob Storage にアップロードされたときに、イベントをトリガーします。 その後、イメージは Azure 関数で処理されます。
- [Blob Storage](/azure/storage/blobs/storage-blobs-introduction) には、Web アプリケーションにアップロードされたすべてのイメージ ファイルと、Web アプリケーションによって使用される任意の静的ファイルが格納されます。
- [Cosmos DB](/azure/cosmos-db/introduction) には、Computer Vision API からの処理結果を含め、アップロードされた各イメージに関するメタデータが格納されます。

## <a name="alternatives"></a>代替手段

- [Custom Vision Service](/azure/cognitive-services/custom-vision-service/home)。 Computer Vision API は、一連の[分類ベースのカテゴリ][cv-categories]を返します。 Computer Vision API によって返されていない情報を処理する必要がある場合は、Custom Vision Service を検討してください。これにより、カスタム イメージ分類子を構築できます。
- [Azure Search](/azure/search/search-what-is-azure-search)。 特定の条件を満たすイメージを検索するために、ご自身のユース ケースでメタデータにクエリを実行する必要がある場合は、Azure Search を検討してください。 現在、プレビュー中の[コグニティブ検索](/azure/search/cognitive-search-concept-intro)では、このワークフローがシームレスに統合されます。

## <a name="considerations"></a>考慮事項

### <a name="scalability"></a>スケーラビリティ

このシナリオの例で使用されるコンポーネントの大半が、自動スケーリングされるマネージド サービスです。 注目すべき例外がいくつかあります。Azure Functions のインスタンス数は最大 200 個に制限されています。 この限界を超えてスケーリングする場合は、複数のリージョンまたはアプリ プランを検討してください。

Cosmos DB は、プロビジョニング済み要求ユニット (RU) の点からいうと、自動スケーリングされません。 ご自身の要件の推定に関するガイダンスについては、Microsoft ドキュメントの[要求ユニット](/azure/cosmos-db/request-units)に関するページをご覧ください。 Cosmos DB でスケーリングのメリットを十分に活用するには、Cosmos DB での[パーティション キー](/azure/cosmos-db/partition-data)のしくみを理解してください。

NoSQL データベースでは、可用性、スケーラビリティ、および分断性と引き換えに、一貫性 (CAP 定理という意味で) が犠牲になることがよくあります。 ただし、このシナリオの例では、キーと値のデータ モデルが使用され、ほとんどの操作が本質的にアトミックであるため、トランザクションの一貫性が必要になることはほとんどありません。 追加のガイダンスについては、Azure アーキテクチャ センターの「[適切なデータ ストアの選択](../../guide/technology-choices/data-store-overview.md)」を参照してください。 実装で高い一貫性が必要な場合は、Cosmos DB で[整合性レベルを選択する](/azure/cosmos-db/consistency-levels)ことができます。

スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

[Azure リソース用のマネージド ID][msi] は、自分のアカウントの内部にある他のリソースへのアクセスを提供するために使用された後、Azure Functions に割り当てられます。 これらの ID の必要なリソースへのアクセスのみを許可して、余分なものがお使いの関数 (およびご自身の顧客) に公開されないようにします。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

このシナリオのすべてのコンポーネントが管理されているため、すべてについて、リージョン レベルの回復性が自動的に確保されます。

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。

トラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています (すべてのイメージのサイズが 100 KB であると想定しています)。

- [Small][small-pricing]: この価格例は、1 か月あたり &lt;5,000 件のイメージの処理に対応します。
- [Medium][medium-pricing]: この価格例は、1 か月あたり 500,000 件のイメージの処理に対応します。
- [Large][large-pricing]: この価格例は、1 か月あたり 50,000,000 件のイメージの処理に対応します。

## <a name="related-resources"></a>関連リソース

ガイド付きラーニング パスについては、「[Azure でサーバーレス Web アプリを作成する][serverless]」を参照してください。

このシナリオの例を運用環境にデプロイする前に、[Azure Functions のパフォーマンスと信頼性を最適化する][functions-best-practices]ための推奨プラクティスをご確認ください。

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
