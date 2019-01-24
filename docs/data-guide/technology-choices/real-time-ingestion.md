---
title: リアルタイム メッセージ取り込みテクノロジの選択
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 9f787a0de5db97f5c0a5651b510e49762fbc44b9
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483004"
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a><span data-ttu-id="a36f9-102">Azure でのリアルタイム メッセージ取り込みテクノロジの選択</span><span class="sxs-lookup"><span data-stu-id="a36f9-102">Choosing a real-time message ingestion technology in Azure</span></span>

<span data-ttu-id="a36f9-103">リアルタイム処理は、リアルタイムでキャプチャされて最小限の待機時間で処理されるデータ ストリームを処理します。</span><span class="sxs-lookup"><span data-stu-id="a36f9-103">Real time processing deals with streams of data that are captured in real-time and processed with minimal latency.</span></span> <span data-ttu-id="a36f9-104">多くのリアルタイム処理ソリューションには、メッセージのためのバッファーとして機能し、スケールアウト処理、信頼性の高い配信、その他のメッセージ キューのセマンティクスをサポートするメッセージ インジェスト ストアが必要です。</span><span class="sxs-lookup"><span data-stu-id="a36f9-104">Many real-time processing solutions need a message ingestion store to act as a buffer for messages, and to support scale-out processing, reliable delivery, and other message queuing semantics.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-for-real-time-message-ingestion"></a><span data-ttu-id="a36f9-105">リアルタイム メッセージ取り込みのオプション</span><span class="sxs-lookup"><span data-stu-id="a36f9-105">What are your options for real-time message ingestion?</span></span>

<!-- markdownlint-enable MD026 -->

- [<span data-ttu-id="a36f9-106">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="a36f9-106">Azure Event Hubs</span></span>](/azure/event-hubs/)
- [<span data-ttu-id="a36f9-107">Azure IoT Hub</span><span class="sxs-lookup"><span data-stu-id="a36f9-107">Azure IoT Hub</span></span>](/azure/iot-hub/)
- [<span data-ttu-id="a36f9-108">HDInsight 上の Kafka</span><span class="sxs-lookup"><span data-stu-id="a36f9-108">Kafka on HDInsight</span></span>](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a><span data-ttu-id="a36f9-109">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="a36f9-109">Azure Event Hubs</span></span>

<span data-ttu-id="a36f9-110">[Azure Event Hubs](/azure/event-hubs/) は高度にスケーラブルなデータ ストリーミング プラットフォームであり、毎秒数百万のイベントを受け取って処理できるイベント インジェスト サービスでもあります。</span><span class="sxs-lookup"><span data-stu-id="a36f9-110">[Azure Event Hubs](/azure/event-hubs/) is a highly scalable data streaming platform and event ingestion service, capable of receiving and processing millions of events per second.</span></span> <span data-ttu-id="a36f9-111">Event Hubs では、分散されたソフトウェアやデバイスから生成されるイベント、データ、またはテレメトリを処理および格納できます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-111">Event Hubs can process and store events, data, or telemetry produced by distributed software and devices.</span></span> <span data-ttu-id="a36f9-112">イベント ハブに送信されたデータは、任意のリアルタイム分析プロバイダーやバッチ処理/ストレージ アダプターを使用して、変換および保存できます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-112">Data sent to an event hub can be transformed and stored using any real-time analytics provider or batching/storage adapters.</span></span> <span data-ttu-id="a36f9-113">Event Hubs は大規模で待機時間の短いパブリッシュ/サブスクライブ機能を提供するため、ビッグ データのシナリオに適しています。</span><span class="sxs-lookup"><span data-stu-id="a36f9-113">Event Hubs provides publish-subscribe capabilities with low latency at massive scale, which makes it appropriate for big data scenarios.</span></span>

## <a name="azure-iot-hub"></a><span data-ttu-id="a36f9-114">Azure IoT Hub</span><span class="sxs-lookup"><span data-stu-id="a36f9-114">Azure IoT Hub</span></span>

<span data-ttu-id="a36f9-115">[Azure IoT Hub](/azure/iot-hub/) は、何百万もの IoT デバイスとクラウドベースのバックエンド間で、セキュリティで保護された信頼性のある双方向通信を実現する、管理されたサービスです。</span><span class="sxs-lookup"><span data-stu-id="a36f9-115">[Azure IoT Hub](/azure/iot-hub/) is a managed service that enables reliable and secure bidirectional communications between millions of IoT devices and a cloud-based back end.</span></span>

<span data-ttu-id="a36f9-116">IoT Hub の機能は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="a36f9-116">Feature of IoT Hub include:</span></span>

- <span data-ttu-id="a36f9-117">device-to-cloud および cloud-to-device 通信における複数のオプション。</span><span class="sxs-lookup"><span data-stu-id="a36f9-117">Multiple options for device-to-cloud and cloud-to-device communication.</span></span> <span data-ttu-id="a36f9-118">これらのオプションには、一方向メッセージング、ファイル転送、要求/応答メソッドなどがあります。</span><span class="sxs-lookup"><span data-stu-id="a36f9-118">These options include one-way messaging, file transfer, and request-reply methods.</span></span>
- <span data-ttu-id="a36f9-119">他の Azure サービスへのメッセージ ルーティング。</span><span class="sxs-lookup"><span data-stu-id="a36f9-119">Message routing to other Azure services.</span></span>
- <span data-ttu-id="a36f9-120">デバイス メタデータと同期状態情報用にクエリ実行可能なストア。</span><span class="sxs-lookup"><span data-stu-id="a36f9-120">Queryable store for device metadata and synchronized state information.</span></span>
- <span data-ttu-id="a36f9-121">デバイスごとのセキュリティ キーまたは X.509 証明書を使用した、セキュリティ保護された通信とアクセス制御。</span><span class="sxs-lookup"><span data-stu-id="a36f9-121">Secure communications and access control using per-device security keys or X.509 certificates.</span></span>
- <span data-ttu-id="a36f9-122">デバイス接続イベントおよびデバイス ID 管理イベントの監視。</span><span class="sxs-lookup"><span data-stu-id="a36f9-122">Monitoring of device connectivity and device identity management events.</span></span>

<span data-ttu-id="a36f9-123">メッセージ取り込みの観点からは、IoT Hub は Event Hubs に似ています。</span><span class="sxs-lookup"><span data-stu-id="a36f9-123">In terms of message ingestion, IoT Hub is similar to Event Hubs.</span></span> <span data-ttu-id="a36f9-124">ただし、メッセージ取り込みだけでなく IoT デバイス接続の管理向けに設計されています。</span><span class="sxs-lookup"><span data-stu-id="a36f9-124">However, it was specifically designed for managing IoT device connectivity, not just message ingestion.</span></span> <span data-ttu-id="a36f9-125">詳しくは、「[Azure IoT Hub と Azure Event Hubs の比較](/azure/iot-hub/iot-hub-compare-event-hubs)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a36f9-125">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).</span></span>

## <a name="kafka-on-hdinsight"></a><span data-ttu-id="a36f9-126">HDInsight 上の Kafka</span><span class="sxs-lookup"><span data-stu-id="a36f9-126">Kafka on HDInsight</span></span>

<span data-ttu-id="a36f9-127">[Apache Kafka](https://kafka.apache.org/) はオープン ソースの分散ストリーム プラットフォームで、リアルタイムのデータ パイプラインとストリーミング アプリケーションの構築に使用できます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-127">[Apache Kafka](https://kafka.apache.org/) is an open-source distributed streaming platform that can be used to build real-time data pipelines and streaming applications.</span></span> <span data-ttu-id="a36f9-128">Kafka は、名前付きデータ ストリームへの公開および購読ができる、メッセージ キューと同様のメッセージ ブローカー機能も提供しています。</span><span class="sxs-lookup"><span data-stu-id="a36f9-128">Kafka also provides message broker functionality similar to a message queue, where you can publish and subscribe to named data streams.</span></span> <span data-ttu-id="a36f9-129">水平方向のスケーラビリティとフォールト トレランスを備え、きわめて高速です。</span><span class="sxs-lookup"><span data-stu-id="a36f9-129">It is horizontally scalable, fault-tolerant, and extremely fast.</span></span> <span data-ttu-id="a36f9-130">[HDInsight 上の Kafka](/azure/hdinsight/kafka/apache-kafka-get-started) では、管理された、拡張性の高い、高可用性のサービスとして Kafka が Azure で提供されます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-130">[Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) provides a Kafka as a managed, highly scalable, and highly available service in Azure.</span></span>

<span data-ttu-id="a36f9-131">Kafka の一般的なユース ケースは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="a36f9-131">Some common use cases for Kafka are:</span></span>

- <span data-ttu-id="a36f9-132">**メッセージング**。</span><span class="sxs-lookup"><span data-stu-id="a36f9-132">**Messaging**.</span></span> <span data-ttu-id="a36f9-133">Kafka はパブリッシュ/サブスクライブのメッセージ パターンをサポートするため、メッセージ ブローカーとしてよく使用されます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-133">Because it supports the publish-subscribe message pattern, Kafka is often used as a message broker.</span></span>
- <span data-ttu-id="a36f9-134">**アクティビティの追跡**。</span><span class="sxs-lookup"><span data-stu-id="a36f9-134">**Activity tracking**.</span></span> <span data-ttu-id="a36f9-135">Kafka ではレコードの受信順序のログ記録が提供されるため、Web サイトでのユーザー アクションなどのアクティビティの追跡と再現に使用することができます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-135">Because Kafka provides in-order logging of records, it can be used to track and re-create activities, such as user actions on a web site.</span></span>
- <span data-ttu-id="a36f9-136">**集計**。</span><span class="sxs-lookup"><span data-stu-id="a36f9-136">**Aggregation**.</span></span> <span data-ttu-id="a36f9-137">ストリーム処理を使用して異なるストリームからの情報を集計し、情報をまとめて運用データに一元化することができます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-137">Using stream processing, you can aggregate information from different streams to combine and centralize the information into operational data.</span></span>
- <span data-ttu-id="a36f9-138">**変換**。</span><span class="sxs-lookup"><span data-stu-id="a36f9-138">**Transformation**.</span></span> <span data-ttu-id="a36f9-139">ストリーム処理を使用して入力された複数のトピックからのデータを結合し、1 つまたは複数の出力トピックに変換することができます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-139">Using stream processing, you can combine and enrich data from multiple input topics into one or more output topics.</span></span>

## <a name="key-selection-criteria"></a><span data-ttu-id="a36f9-140">主要な選択条件</span><span class="sxs-lookup"><span data-stu-id="a36f9-140">Key selection criteria</span></span>

<span data-ttu-id="a36f9-141">選択を絞り込むには、以下の質問に答えることから開始します。</span><span class="sxs-lookup"><span data-stu-id="a36f9-141">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="a36f9-142">IoT デバイスと Azure の間の双方向通信が必要ですか。</span><span class="sxs-lookup"><span data-stu-id="a36f9-142">Do you need two-way communication between your IoT devices and Azure?</span></span> <span data-ttu-id="a36f9-143">必要な場合は、IoT Hub を選びます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-143">If so, choose IoT Hub.</span></span>

- <span data-ttu-id="a36f9-144">個々のデバイスのアクセスを管理し、特定のデバイスへのアクセスを取り消せる必要がありますか。</span><span class="sxs-lookup"><span data-stu-id="a36f9-144">Do you need to manage access for individual devices and be able to revoke access to a specific device?</span></span> <span data-ttu-id="a36f9-145">答えが「はい」の場合は、IoT Hub を選びます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-145">If yes, choose IoT Hub.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="a36f9-146">機能のマトリックス</span><span class="sxs-lookup"><span data-stu-id="a36f9-146">Capability matrix</span></span>

<span data-ttu-id="a36f9-147">次の表は、機能の主な相違点をまとめたものです。</span><span class="sxs-lookup"><span data-stu-id="a36f9-147">The following tables summarize the key differences in capabilities.</span></span>

<!-- markdownlint-disable MD033 -->

| | <span data-ttu-id="a36f9-148">IoT Hub</span><span class="sxs-lookup"><span data-stu-id="a36f9-148">IoT Hub</span></span> | <span data-ttu-id="a36f9-149">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="a36f9-149">Event Hubs</span></span> | <span data-ttu-id="a36f9-150">HDInsight 上の Kafka</span><span class="sxs-lookup"><span data-stu-id="a36f9-150">Kafka on HDInsight</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="a36f9-151">クラウドからデバイスへの通信</span><span class="sxs-lookup"><span data-stu-id="a36f9-151">Cloud-to-device communications</span></span> | <span data-ttu-id="a36f9-152">[はい]</span><span class="sxs-lookup"><span data-stu-id="a36f9-152">Yes</span></span> | <span data-ttu-id="a36f9-153">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-153">No</span></span> | <span data-ttu-id="a36f9-154">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-154">No</span></span> |
| <span data-ttu-id="a36f9-155">デバイスによって開始されたファイル アップロード</span><span class="sxs-lookup"><span data-stu-id="a36f9-155">Device-initiated file upload</span></span> | <span data-ttu-id="a36f9-156">[はい]</span><span class="sxs-lookup"><span data-stu-id="a36f9-156">Yes</span></span> | <span data-ttu-id="a36f9-157">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-157">No</span></span> | <span data-ttu-id="a36f9-158">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-158">No</span></span> |
| <span data-ttu-id="a36f9-159">デバイスの状態情報</span><span class="sxs-lookup"><span data-stu-id="a36f9-159">Device state information</span></span> | [<span data-ttu-id="a36f9-160">デバイス ツイン</span><span class="sxs-lookup"><span data-stu-id="a36f9-160">Device twins</span></span>](/azure/iot-hub/iot-hub-devguide-device-twins) | <span data-ttu-id="a36f9-161">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-161">No</span></span> | <span data-ttu-id="a36f9-162">いいえ </span><span class="sxs-lookup"><span data-stu-id="a36f9-162">No</span></span> |
| <span data-ttu-id="a36f9-163">プロトコルのサポート</span><span class="sxs-lookup"><span data-stu-id="a36f9-163">Protocol support</span></span> | <span data-ttu-id="a36f9-164">MQTT、AMQP、HTTPS <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="a36f9-164">MQTT, AMQP, HTTPS <sup>1</sup></span></span> | <span data-ttu-id="a36f9-165">AMQP、HTTPS</span><span class="sxs-lookup"><span data-stu-id="a36f9-165">AMQP, HTTPS</span></span> | [<span data-ttu-id="a36f9-166">Kafka プロトコル</span><span class="sxs-lookup"><span data-stu-id="a36f9-166">Kafka Protocol</span></span>](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| <span data-ttu-id="a36f9-167">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="a36f9-167">Security</span></span> | <span data-ttu-id="a36f9-168">デバイス ID ごと。取り消し可能なアクセス制御。</span><span class="sxs-lookup"><span data-stu-id="a36f9-168">Per-device identity; revocable access control.</span></span> | <span data-ttu-id="a36f9-169">共有アクセス ポリシー。パブリッシャー ポリシーを介した限られた取り消し。</span><span class="sxs-lookup"><span data-stu-id="a36f9-169">Shared access policies; limited revocation through publisher policies.</span></span> | <span data-ttu-id="a36f9-170">SASL を使用した認証。プラグ可能な承認。サポートされている外部認証サービスとの統合。</span><span class="sxs-lookup"><span data-stu-id="a36f9-170">Authentication using SASL; pluggable authorization; integration with external authentication services supported.</span></span> |

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="a36f9-171">[1] [Azure IoT プロトコル ゲートウェイ](/azure/iot-hub/iot-hub-protocol-gateway)をカスタム ゲートウェイとして使用して、IoT Hub に対してプロトコルを適用することもできます。</span><span class="sxs-lookup"><span data-stu-id="a36f9-171">[1] You can also use [Azure IoT protocol gateway](/azure/iot-hub/iot-hub-protocol-gateway) as a custom gateway to enable protocol adaptation for IoT Hub.</span></span>

<span data-ttu-id="a36f9-172">詳しくは、「[Azure IoT Hub と Azure Event Hubs の比較](/azure/iot-hub/iot-hub-compare-event-hubs)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="a36f9-172">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).</span></span>
