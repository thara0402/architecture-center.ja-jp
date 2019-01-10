---
title: データ管理のパターン
titleSuffix: Cloud Design Patterns
description: データ管理はクラウド アプリケーションの重要な要素であり、品質属性のほとんどに影響します。 通常、パフォーマンス、スケーラビリティ、または可用性の理由から、データは複数のサーバーにまたがってさまざまな場所でホストされます。これによって、広範な課題が生じることがあります。 たとえば、データの整合性を維持する必要があります。また、通常はさまざまな場所にあるデータを同期する必要があります。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: ff6d5703af64ddd8b012b588ddfe810da0b6630c
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009187"
---
# <a name="data-management-patterns"></a><span data-ttu-id="fb741-106">データ管理のパターン</span><span class="sxs-lookup"><span data-stu-id="fb741-106">Data Management patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="fb741-107">データ管理はクラウド アプリケーションの重要な要素であり、品質属性のほとんどに影響します。</span><span class="sxs-lookup"><span data-stu-id="fb741-107">Data management is the key element of cloud applications, and influences most of the quality attributes.</span></span> <span data-ttu-id="fb741-108">通常、パフォーマンス、スケーラビリティ、または可用性の理由から、データは複数のサーバーにまたがってさまざまな場所でホストされます。これによって、広範な課題が生じることがあります。</span><span class="sxs-lookup"><span data-stu-id="fb741-108">Data is typically hosted in different locations and across multiple servers for reasons such as performance, scalability or availability, and this can present a range of challenges.</span></span> <span data-ttu-id="fb741-109">たとえば、データの整合性を維持する必要があります。また、通常はさまざまな場所にあるデータを同期する必要があります。</span><span class="sxs-lookup"><span data-stu-id="fb741-109">For example, data consistency must be maintained, and data will typically need to be synchronized across different locations.</span></span>

|                        <span data-ttu-id="fb741-110">Pattern</span><span class="sxs-lookup"><span data-stu-id="fb741-110">Pattern</span></span>                         |                                                                  <span data-ttu-id="fb741-111">まとめ</span><span class="sxs-lookup"><span data-stu-id="fb741-111">Summary</span></span>                                                                  |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
|            [<span data-ttu-id="fb741-112">キャッシュ アサイド</span><span class="sxs-lookup"><span data-stu-id="fb741-112">Cache-Aside</span></span>](../cache-aside.md)            |                                            <span data-ttu-id="fb741-113">オンデマンドでデータをデータ ストアからキャッシュに読み込みます</span><span class="sxs-lookup"><span data-stu-id="fb741-113">Load data on demand into a cache from a data store</span></span>                                             |
|                   [<span data-ttu-id="fb741-114">CQRS</span><span class="sxs-lookup"><span data-stu-id="fb741-114">CQRS</span></span>](../cqrs.md)                   |                    <span data-ttu-id="fb741-115">個別のインターフェイスを使用して、データを更新する操作から、データを読み取る操作を分離します。</span><span class="sxs-lookup"><span data-stu-id="fb741-115">Segregate operations that read data from operations that update data by using separate interfaces.</span></span>                     |
|         [<span data-ttu-id="fb741-116">イベント ソース</span><span class="sxs-lookup"><span data-stu-id="fb741-116">Event Sourcing</span></span>](../event-sourcing.md)         |               <span data-ttu-id="fb741-117">追加専用のストアを使用して、ドメイン内のデータに実行されるアクションを記述する一連のイベントすべてを記録します。</span><span class="sxs-lookup"><span data-stu-id="fb741-117">Use an append-only store to record the full series of events that describe actions taken on data in a domain.</span></span>               |
|            [<span data-ttu-id="fb741-118">テーブルのインデックス作成</span><span class="sxs-lookup"><span data-stu-id="fb741-118">Index Table</span></span>](../index-table.md)            |                         <span data-ttu-id="fb741-119">クエリによって頻繁に参照されるデータ ストア内のフィールドにインデックスを作成します。</span><span class="sxs-lookup"><span data-stu-id="fb741-119">Create indexes over the fields in data stores that are frequently referenced by queries.</span></span>                          |
|      [<span data-ttu-id="fb741-120">具体化されたビュー</span><span class="sxs-lookup"><span data-stu-id="fb741-120">Materialized View</span></span>](../materialized-view.md)      | <span data-ttu-id="fb741-121">データの形式が必要なクエリ操作に適していない場合に、1 つ以上のデータ ストアのデータの事前設定されたビューを生成します。</span><span class="sxs-lookup"><span data-stu-id="fb741-121">Generate prepopulated views over the data in one or more data stores when the data isn't ideally formatted for required query operations.</span></span> |
|               [<span data-ttu-id="fb741-122">シャーディング</span><span class="sxs-lookup"><span data-stu-id="fb741-122">Sharding</span></span>](../sharding.md)               |                                    <span data-ttu-id="fb741-123">データ ストアを水平方向のパーティションまたはシャードのセットに分割します。</span><span class="sxs-lookup"><span data-stu-id="fb741-123">Divide a data store into a set of horizontal partitions or shards.</span></span>                                     |
| [<span data-ttu-id="fb741-124">静的コンテンツ ホスティング</span><span class="sxs-lookup"><span data-stu-id="fb741-124">Static Content Hosting</span></span>](../static-content-hosting.md) |                   <span data-ttu-id="fb741-125">静的なコンテンツを、クライアントに直接配信できるクラウドベースのストレージ サービスにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="fb741-125">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span>                    |
|              [<span data-ttu-id="fb741-126">バレット キー</span><span class="sxs-lookup"><span data-stu-id="fb741-126">Valet Key</span></span>](../valet-key.md)              |                 <span data-ttu-id="fb741-127">特定のリソースまたはサービスへの限定的な直接アクセスをクライアントに提供する、トークンまたはキーを使用します。</span><span class="sxs-lookup"><span data-stu-id="fb741-127">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                 |
