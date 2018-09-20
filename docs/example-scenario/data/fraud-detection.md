---
title: Azure におけるリアルタイムでの不正検出
description: Azure Event Hubs と Stream Analytics を使用して、不正行為をリアルタイムで検出する実証済みのシナリオ。
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: d80fab460938cceeb84f3ed2ecd97e9e149f8e2d
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389130"
---
# <a name="real-time-fraud-detection-on-azure"></a>Azure におけるリアルタイムでの不正検出

このサンプル シナリオは、不正なトランザクションやその他の異常なアクティビティを検出するために、リアルタイムでデータを分析する必要がある組織に関連します。

考えられる用途としては、不正なクレジット カード アクティビティや携帯電話の通話の特定などが挙げられます。 従来のオンライン分析システムでは、異常なアクティビティを特定するためのデータの変換や分析に何時間もかかる場合があります。

Event Hubs、Stream Analytics などのフル マネージド Azure サービスを使用すると、企業はサーバーを個別に管理する必要がなくなり、コストを削減し、クラウド規模のデータ インジェストとリアルタイム分析で Microsoft の専門知識を活用することができます。 このシナリオでは、特に不正行為の検出を扱っています。 データ分析について他のニーズがある場合は、使用可能な [Azure Analytics サービス][product-category]の一覧を確認する必要があります。

このサンプルは、広範なデータ処理アーキテクチャと戦略の一部です。 全体的なアーキテクチャのこの側面における別のオプションについては、この記事の後半で説明します。

## <a name="related-use-cases"></a>関連するユース ケース

次のユース ケースについて、このシナリオを検討してください。

* 電気通信シナリオで不正な携帯電話呼び出しを検出する。
* 金融機関向けに不正なクレジット カード トランザクションを特定する。
* 小売または eコマースのシナリオで不正購入を特定する。

## <a name="architecture"></a>アーキテクチャ

![リアルタイム不正検出シナリオの Azure コンポーネント アーキテクチャの概要][architecture-diagram]

このシナリオでは、リアルタイム分析パイプラインのバックエンド コンポーネントに対応できます。 シナリオのデータ フローは次のとおりです。

1. 携帯電話呼び出しメタデータが、ソース システムから Azure Event Hubs インスタンスに送信されます。 
2. Stream Analytics ジョブが開始され、イベント ハブ ソースを介してデータが受信されます。
3. Stream Analytics ジョブによって定義済みクエリが実行されます。これにより、入力ストリームが変換され、不正なトランザクションのアルゴリズムに基づいて分析されます。 このクエリでは、タンブリング ウィンドウを使って、ストリームが個別のテンポラル ユニットにセグメント化されます。
4. Stream Analytics ジョブにより、検出された不正な呼び出しを表す変換済みストリームが、Azure Blob Storage の出力シンクに書き込まれます。

### <a name="components"></a>コンポーネント

* [Azure Event Hubs][docs-event-hubs] はリアルタイム ストリーミング プラットフォームであり、毎秒数百万のイベントを受け取って処理できるイベント インジェスト サービスです。 Event Hubs では、分散されたソフトウェアやデバイスから生成されるイベント、データ、またはテレメトリを処理および格納できます。 このシナリオでは、Event Hubs が、すべての電話呼び出しメタデータを受け取り、不正行為に関する分析を実行します。
* [Azure Stream Analytics][docs-stream-analytics] は、デバイスおよび他のデータ ソースからの大量のデータ ストリームを分析できるイベント処理エンジンです。 また、データ ストリームから情報を抽出し、パターンやリレーションシップを特定することもできます。 これらのパターンでは、その他のダウンストリーム アクションをトリガーできます。 このシナリオでは、Stream Analytics によって、Event Hubs からの入力ストリームが変換され、不正な呼び出しが特定されます。
* このシナリオでは、[Blob Storage][docs-blob-storage] を使って、Stream Analytics ジョブの結果が格納されます。

## <a name="considerations"></a>考慮事項

### <a name="alternatives"></a>代替手段

リアルタイム メッセージ インジェスト、データ ストレージ、ストリーム処理、分析データのストレージ、および分析とレポート作成では、使用できるテクノロジが多数あります。 これらのオプションやその機能、主要な選択条件の概要については、Azure データ アーキテクチャ ガイドの[ビッグ データ アーキテクチャのリアルタイム処理](/azure/architecture/data-guide/technology-choices/real-time-ingestion)に関するページをご覧ください。

また、より複雑な不正検出アルゴリズムを、Azure のさまざまな機械学習サービスで生成することもできます。 これらのオプションの概要については、「[Azure データ アーキテクチャ ガイド](../../data-guide/index.md)」の[機械学習のテクノロジの選択](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)に関するページをご覧ください。

### <a name="availability"></a>可用性

Azure Monitor には、さまざまな Azure サービスにわたって監視するための統合ユーザー インターフェイスが用意されています。 詳細については、[Microsoft Azure での監視](/azure/monitoring-and-diagnostics/monitoring-overview)に関するページをご覧ください。 Event Hubs と Stream Analytics は両方とも、Azure Monitor に統合されています。 

他の可用性に関する考慮事項については、Azure アーキテクチャ センターの「[可用性のチェックリスト][availability]」を参照してください。

### <a name="scalability"></a>スケーラビリティ

このシナリオのコンポーネントは、ハイパースケール インジェストと超並列リアルタイム分析を実現するように設計されています。 Azure Event Hubs は高度にスケーラブルで、毎秒数百万のイベントを、短い待機時間で受け取って処理できます。  Event Hubs では、スループット ユニット数が、使用量のニーズに合わせて[自動的にスケールアップ](/azure/event-hubs/event-hubs-auto-inflate)できます。 Azure Stream Analytics では、多くのソースからの大量のストリーミング データを分析できます。 Stream Analytics をスケールアップするには、ご自身のストリーミング ジョブを実行するために割り当てられている[ストリーミング ユニット](/azure/stream-analytics/stream-analytics-streaming-unit-consumption)数を増やします。

スケーラブルなシナリオの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。

### <a name="security"></a>セキュリティ

Azure Event Hubs では、Shared Access Signature (SAS) トークンとイベント パブリッシャーの組み合わせに基づく[認証とセキュリティ モデル][docs-event-hubs-security-model]によって、データが保護されます。 イベント パブリッシャーは Event Hub の仮想エンドポイントを定義します。 パブリッシャーは、Event Hub にメッセージを送信するためにのみ使用できます。 パブリッシャーからメッセージを受信することはできません。

セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。

### <a name="resiliency"></a>回復性

回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。

## <a name="deploy-the-scenario"></a>シナリオのデプロイ

このシナリオをデプロイするには、こちらの[ステップ バイ ステップのチュートリアル][tutorial]に従います。このチュートリアルでは、シナリオの各コンポーネントを手動でデプロイする方法を示しています。 また、このチュートリアルは、サンプル電話呼び出しメタデータを生成し、そのデータをイベント ハブ インスタンスに送信するための .NET クライアント アプリケーションも提供します。

## <a name="pricing"></a>価格

このシナリオの実行コストを調べることができるように、すべてのサービスがコスト計算ツールで事前構成されています。 特定のユース ケースについて価格の変化を確認するには、予想されるデータ ボリュームに合わせて該当する変数を変更します。

取得するトラフィックの量に基づいて、次の 3 つのサンプル コスト プロファイルが用意されています。

* [Small][small-pricing]: 1 標準ストリーミング ユニットで 1 か月あたり 100万件のイベントを処理します。
* [Medium][medium-pricing]: 5 標準ストリーミング ユニットで 1 か月あたり 1 億件のイベントを処理します。
* [Large][large-pricing]: 20 標準ストリーミング ユニットで 1 か月あたり 9 億 9,900 万件のイベントを処理します。

## <a name="related-resources"></a>関連リソース

不正検出のシナリオがさらに複雑な場合は、機械学習モデルを活用できます。 Machine Learning Server を使用して構築されたシナリオについては、[Machine Learning Server を使用した不正検出][r-server-fraud-detection]に関するページをご覧ください。 Machine Learning Server を使用した他のソリューション テンプレートについては、[データ サイエンスのシナリオとソリューション テンプレート][docs-r-server-sample-solutions]に関するページをご覧ください。 Azure Data Lake Analytics を使用したソリューションの例については、「[Using Azure Data Lake and R for Fraud Detection (不正検出に Azure Data Lake および R を使用する)][technet-fraud-detection]」を参照してください。  

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture-diagram]: ./media/architecture-diagram-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

