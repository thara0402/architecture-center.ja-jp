---
title: CAF:暗号化
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure 移行のコア サービスとしての暗号化について説明します。
author: rotycenh
ms.openlocfilehash: 660206d57ded9a93d73c57ba9cb8058020d87525
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901454"
---
# <a name="encryption-decision-guide"></a><span data-ttu-id="70037-103">暗号化決定ガイド</span><span class="sxs-lookup"><span data-stu-id="70037-103">Encryption decision guide</span></span>

<span data-ttu-id="70037-104">データを暗号化すると、データが不正アクセスから保護されます。</span><span class="sxs-lookup"><span data-stu-id="70037-104">Encrypting data protects it against unauthorized access.</span></span> <span data-ttu-id="70037-105">暗号化ポリシーを適切に実装すると、クラウドベースのワークロードにセキュリティのレイヤーが追加され、組織やネットワークの内外両方からの攻撃者や他の承認されていないユーザーに対する保護が提供されます。</span><span class="sxs-lookup"><span data-stu-id="70037-105">Properly implemented encryption policy provides additional layers of security for your cloud-based workloads and guards against attackers and other unauthorized users from both inside and outside your organization and networks.</span></span>

<span data-ttu-id="70037-106">リソースの暗号化は一般に望ましいことですが、待ち時間が長くなってリソースの全体的な使用量が増加するコストも伴います。</span><span class="sxs-lookup"><span data-stu-id="70037-106">While encrypting resources is generally desirable, encryption does have costs that can increase latency and overall resource usage.</span></span> <span data-ttu-id="70037-107">要求の厳しいワークロードでは、暗号化とパフォーマンスの適切なバランスが重要です。</span><span class="sxs-lookup"><span data-stu-id="70037-107">For demanding workloads, striking the correct balance between encryption and performance is essential.</span></span>

![暗号化 オプションを最も簡単なものから最も複雑なものまでプロットし、その下に整合するジャンプ リンクを示す](../../_images/discovery-guides/discovery-guide-encryption.png)

<span data-ttu-id="70037-109">ジャンプ先:[キーの管理](#key-management) | [データの暗号化](#data-encryption) | [詳細情報](#learn-more)</span><span class="sxs-lookup"><span data-stu-id="70037-109">Jump to: [Key management](#key-management) | [Data encryption](#data-encryption) | [Learn more](#learn-more)</span></span>

<span data-ttu-id="70037-110">クラウド暗号化戦略を決定するときの転換点では、企業のポリシーとコンプライアンスの要件に焦点が当てられます。</span><span class="sxs-lookup"><span data-stu-id="70037-110">The inflection point when determining a cloud encryption strategy focuses on corporate policy and compliance mandates.</span></span>

<span data-ttu-id="70037-111">クラウド環境に暗号化を実装する方法は複数あり、コストと複雑さが異なります。</span><span class="sxs-lookup"><span data-stu-id="70037-111">There are multiple ways to implement encryption in a cloud environment, with varying cost and complexity.</span></span> <span data-ttu-id="70037-112">企業のポリシーとサード パーティのコンプライアンスは、暗号化戦略を計画するときの最も大きな要因です。</span><span class="sxs-lookup"><span data-stu-id="70037-112">Corporate policy and third-party compliance are the biggest drivers when planning an encryption strategy.</span></span> <span data-ttu-id="70037-113">ほとんどのクラウドベースのソリューションでは、保存時と転送中のデータを暗号化するための標準的なメカニズムが提供されます。</span><span class="sxs-lookup"><span data-stu-id="70037-113">Most cloud-based solutions provide standard mechanisms for encrypting data, whether at rest or in transit.</span></span> <span data-ttu-id="70037-114">一方、標準化されたシークレットとキーの管理、使用する暗号化、データ固有の暗号化など、より厳しい制御が要求されるポリシーとコンプライアンス要件の場合は、おそらく、複雑なソリューションの実装が必要になります。</span><span class="sxs-lookup"><span data-stu-id="70037-114">However, for policies and compliance requirements that demand tighter controls, such as standardized secrets and key management, encryption in-use, or data specific encryption, you will likely need to implement a complex solution.</span></span>

## <a name="key-management"></a><span data-ttu-id="70037-115">キー管理</span><span class="sxs-lookup"><span data-stu-id="70037-115">Key management</span></span>

<span data-ttu-id="70037-116">最新のキー管理システムでは、保護を強化するためハードウェア セキュリティ モジュール (HSM) を使用したキーの格納のサポートが提供されている必要があります。</span><span class="sxs-lookup"><span data-stu-id="70037-116">Modern key management systems should offer support for storing keys using hardware security modules (HSMs) for increased protection.</span></span> <span data-ttu-id="70037-117">そのため、組織で暗号化キー、重要なパスワード、接続文字列、その他の IT の機密情報を作成して保管できるためには、キー管理システムが不可欠です。</span><span class="sxs-lookup"><span data-stu-id="70037-117">Thus, a key management system is critical to your organization's ability to create and store cryptographic keys, important passwords, connection strings, and other IT confidential information.</span></span>

<span data-ttu-id="70037-118">次の表では、クラウドへの移行を計画するときのために、安全で管理しやすいクラウド デプロイを作成するのに不可欠な、暗号化キー、証明書、シークレットを保存して管理する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="70037-118">When planning a cloud migration, the following table describes how you can store and manage encryption keys, certificates, and secrets, which are critical for creating secure and manageable cloud deployments:</span></span>

| <span data-ttu-id="70037-119">質問</span><span class="sxs-lookup"><span data-stu-id="70037-119">Question</span></span> | <span data-ttu-id="70037-120">クラウド ネイティブ</span><span class="sxs-lookup"><span data-stu-id="70037-120">Cloud Native</span></span> | <span data-ttu-id="70037-121">ハイブリッド</span><span class="sxs-lookup"><span data-stu-id="70037-121">Hybrid</span></span> | <span data-ttu-id="70037-122">オンプレミス</span><span class="sxs-lookup"><span data-stu-id="70037-122">On-premises</span></span> |
|---------------------------------------------------------------------------------------------------------------------------------------|--------------|--------|-------------|
| <span data-ttu-id="70037-123">組織では一元的なキーとシークレットの管理が行われていませんか</span><span class="sxs-lookup"><span data-stu-id="70037-123">Does your organization lack centralized key and secret management?</span></span>                                                                    | <span data-ttu-id="70037-124">はい</span><span class="sxs-lookup"><span data-stu-id="70037-124">Yes</span></span>          | <span data-ttu-id="70037-125">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-125">No</span></span>     | <span data-ttu-id="70037-126">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-126">No</span></span>          |
| <span data-ttu-id="70037-127">デバイスに対するキーとシークレットの作成はオンプレミスのハードウェアに制限する必要がある一方で、それらのキーはクラウドで使用されますか</span><span class="sxs-lookup"><span data-stu-id="70037-127">Will you need to limit the creation of keys and secrets to devices to your on-premises hardware, while using these keys in the cloud?</span></span> | <span data-ttu-id="70037-128">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-128">No</span></span>           | <span data-ttu-id="70037-129">はい</span><span class="sxs-lookup"><span data-stu-id="70037-129">Yes</span></span>    | <span data-ttu-id="70037-130">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-130">No</span></span>          |
| <span data-ttu-id="70037-131">組織にはキーとシークレットがオフサイトに格納されるのを禁止するルールまたはポリシーがありますか</span><span class="sxs-lookup"><span data-stu-id="70037-131">Does your organization have rules or policies in place that would prevent keys and secrets from being stored offsite?</span></span>                | <span data-ttu-id="70037-132">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-132">No</span></span>           | <span data-ttu-id="70037-133">いいえ </span><span class="sxs-lookup"><span data-stu-id="70037-133">No</span></span>     | <span data-ttu-id="70037-134">はい</span><span class="sxs-lookup"><span data-stu-id="70037-134">Yes</span></span>         |

### <a name="cloud-native"></a><span data-ttu-id="70037-135">クラウド ネイティブ</span><span class="sxs-lookup"><span data-stu-id="70037-135">Cloud native</span></span>

<span data-ttu-id="70037-136">クラウド ネイティブのキー管理では、すべてのキーとシークレットはクラウドベースのコンテナーで生成、管理、格納されます。</span><span class="sxs-lookup"><span data-stu-id="70037-136">With cloud native key management, all keys and secrets are generated, managed, and stored in a cloud-based vault.</span></span> <span data-ttu-id="70037-137">このアプローチでは、キーの管理に関連する多くの IT タスクが簡略化されます。</span><span class="sxs-lookup"><span data-stu-id="70037-137">This approach simplifies many IT tasks related to key management.</span></span>

<span data-ttu-id="70037-138">クラウド ネイティブのキー管理の前提条件:クラウド ネイティブのキー管理システムの使用には、次の前提条件があります。</span><span class="sxs-lookup"><span data-stu-id="70037-138">Cloud native key management assumptions: Using a cloud native key management system assumes the following:</span></span>

- <span data-ttu-id="70037-139">組織のシークレットとキーの作成、管理、ホストについて、クラウドのキー管理ソリューションを信頼します。</span><span class="sxs-lookup"><span data-stu-id="70037-139">You trust the cloud key management solution with creating, managing, and hosting your organization's secrets and keys.</span></span>
- <span data-ttu-id="70037-140">クラウドのキー管理システムにアクセスするために暗号化サービスまたはシークレットへのアクセスに依存するすべてのオンプレミス アプリケーションとサービスを有効にします。</span><span class="sxs-lookup"><span data-stu-id="70037-140">You enable all on-premises applications and services that rely on accessing encryption services or secrets to access the cloud key management system.</span></span>

### <a name="hybrid-bring-your-own-key"></a><span data-ttu-id="70037-141">ハイブリッド (Bring Your Own Key)</span><span class="sxs-lookup"><span data-stu-id="70037-141">Hybrid (bring your own key)</span></span>

<span data-ttu-id="70037-142">Bring Your Own Key のアプローチでは、オンプレミス環境内の専用 HSM ハードウェアでキーを生成した後、クラウド リソースで使用するために、セキュリティで保護されたクラウドのキー管理システムにキーを転送します。</span><span class="sxs-lookup"><span data-stu-id="70037-142">With a bring-your-own-key approach, you generate keys on dedicated HSM hardware within your on-premises environment, then transfer the keys to a secure cloud key management system for use with cloud resources.</span></span>

<span data-ttu-id="70037-143">ハイブリッドのキー管理の前提条件:ハイブリッドのキー管理システムの使用には、次の前提条件があります。</span><span class="sxs-lookup"><span data-stu-id="70037-143">Hybrid key management assumptions: Using a hybrid key management system assumes the following:</span></span>

- <span data-ttu-id="70037-144">キーとシークレットをホストおよび使用するために、クラウド プラットフォームの基盤のセキュリティおよびアクセス制御インフラストラクチャを信頼します。</span><span class="sxs-lookup"><span data-stu-id="70037-144">You trust the underlying security and access control infrastructure of the cloud platform for hosting and using your keys and secrets.</span></span>
- <span data-ttu-id="70037-145">規制や組織方針によって、組織のシークレットとキーの作成と管理をオンプレミスで行うことが求められます。</span><span class="sxs-lookup"><span data-stu-id="70037-145">You are required by regulatory or organizational policy to keep the creation and management of your organization's secrets and keys on-premises.</span></span>

### <a name="on-premises-hold-your-own-key"></a><span data-ttu-id="70037-146">オンプレミス (Hold Your Own Key)</span><span class="sxs-lookup"><span data-stu-id="70037-146">On-premises (hold your own key)</span></span>

<span data-ttu-id="70037-147">特定のシナリオでは、規制、方針、または技術的な理由により、パブリック クラウド サービスによって提供されるキー管理システムにキーを格納できない可能性があります。</span><span class="sxs-lookup"><span data-stu-id="70037-147">In certain scenarios, there may be regulatory, policy, or technical reasons why you can't store keys on a key management system provided by a public cloud service.</span></span> <span data-ttu-id="70037-148">このような場合は、オンプレミスのハードウェアを使用してキーを保持し、クラウドベースのリソースが暗号化のためにこれらのキーにアクセスできるメカニズムをプロビジョニングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="70037-148">In these cases, you must maintain keys using on-premises hardware, and provision a mechanism to allow cloud-based resource to access these keys for encryption purposes.</span></span> <span data-ttu-id="70037-149">Hold Your Own Key アプローチと互換性のないクラウド サービスがあるかもしれないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="70037-149">Note that a hold your own key approach may not be compatible with all cloud services.</span></span>

<span data-ttu-id="70037-150">オンプレミスのキー管理の前提条件:オンプレミスのキー管理システムの使用には、次の前提条件があります。</span><span class="sxs-lookup"><span data-stu-id="70037-150">On-premises key management assumptions: Using an on-premises key management system assumes the following:</span></span>

- <span data-ttu-id="70037-151">規制や組織方針によって、組織のシークレットとキーの作成、管理、ホストをオンプレミスで行うことが求められます。</span><span class="sxs-lookup"><span data-stu-id="70037-151">You are required by regulatory or organizational policy to keep the creation, management, and hosting of your organization's secrets and keys on-premises.</span></span>
- <span data-ttu-id="70037-152">暗号化サービスまたはシークレットへのアクセスに依存するすべてのクラウドベースのアプリケーションまたはサービスが、オンプレミスのキー管理システムにアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="70037-152">Any cloud-based applications or services that rely on accessing encryption services or secrets can access the on-premises key management system.</span></span>

## <a name="data-encryption"></a><span data-ttu-id="70037-153">データの暗号化</span><span class="sxs-lookup"><span data-stu-id="70037-153">Data encryption</span></span>

<span data-ttu-id="70037-154">暗号化ポリシーを計画するときは、暗号化ニーズが異なる複数のデータの状態が存在することを考慮します。</span><span class="sxs-lookup"><span data-stu-id="70037-154">There are several different states of data with different encryption needs to consider when planning your encryption policy:</span></span>

| <span data-ttu-id="70037-155">データの状態</span><span class="sxs-lookup"><span data-stu-id="70037-155">Data state</span></span> | <span data-ttu-id="70037-156">データ</span><span class="sxs-lookup"><span data-stu-id="70037-156">Data</span></span> |
|-----|-----|
| <span data-ttu-id="70037-157">転送中のデータ</span><span class="sxs-lookup"><span data-stu-id="70037-157">Data in transit</span></span> | <span data-ttu-id="70037-158">内部ネットワーク トラフィック、インターネット接続、データ センターまたは仮想ネットワーク間の接続</span><span class="sxs-lookup"><span data-stu-id="70037-158">Internal network traffic, internet connections, connections between datacenters or virtual networks</span></span> |
| <span data-ttu-id="70037-159">保存データ</span><span class="sxs-lookup"><span data-stu-id="70037-159">Data at rest</span></span>    | <span data-ttu-id="70037-160">データベース、ファイル、仮想ドライブ、PaaS ストレージ</span><span class="sxs-lookup"><span data-stu-id="70037-160">Databases, files, virtual drives, PaaS storage</span></span> |
| <span data-ttu-id="70037-161">使用中のデータ</span><span class="sxs-lookup"><span data-stu-id="70037-161">Data in use</span></span>     | <span data-ttu-id="70037-162">RAM または CPU キャッシュに読み込まれたデータ</span><span class="sxs-lookup"><span data-stu-id="70037-162">Data loaded in RAM or in CPU caches</span></span> |

### <a name="data-in-transit"></a><span data-ttu-id="70037-163">転送中のデータ</span><span class="sxs-lookup"><span data-stu-id="70037-163">Data in transit</span></span>

<span data-ttu-id="70037-164">転送中のデータとは、内部のリソース間、データ センター間や外部のネットワーク間、またはインターネット上を移動しているデータです。</span><span class="sxs-lookup"><span data-stu-id="70037-164">Data in transit is data moving between resources on the internal, between datacenters or external networks, or over the internet.</span></span>

<span data-ttu-id="70037-165">転送中のデータの暗号化は、通常、トラフィックに対して SSL/TLS プロトコルを要求することによって行われます。</span><span class="sxs-lookup"><span data-stu-id="70037-165">Encrypting data in transit is usually done by requiring SSL/TLS protocols for traffic.</span></span> <span data-ttu-id="70037-166">クラウドでホストされたリソースと外部ネットワークまたはパブリック インターネットの間を移動するトラフィックは、常に暗号化する必要があります。</span><span class="sxs-lookup"><span data-stu-id="70037-166">Traffic transiting between your cloud-hosted resources to external network or the public internet should always be encrypted.</span></span> <span data-ttu-id="70037-167">PaaS リソースでも、一般に、既定でトラフィックに対して SSL/TLS の暗号化が適用されます。</span><span class="sxs-lookup"><span data-stu-id="70037-167">PaaS resources generally also enforce SSL/TLS encryption to traffic by default.</span></span> <span data-ttu-id="70037-168">会社の仮想ネットワーク内でホストされている IaaS リソース間のトラフィックに対して暗号化を適用するかどうかは、クラウド導入チームとワークロード所有者が決定することであり、通常は推奨されます。</span><span class="sxs-lookup"><span data-stu-id="70037-168">Whether you enforce encryption for traffic between IaaS resources hosted inside your virtual networks is a decision for your Cloud Adoption Team and workload owner and is generally recommended.</span></span>

<span data-ttu-id="70037-169">**転送中のデータの暗号化の前提条件**。</span><span class="sxs-lookup"><span data-stu-id="70037-169">**Encrypting data in transit assumptions**.</span></span> <span data-ttu-id="70037-170">転送中データに対する適切な暗号化ポリシーの実装では、次のことが想定されます。</span><span class="sxs-lookup"><span data-stu-id="70037-170">Implementing proper encryption policy for data in transit assumes the following:</span></span>

- <span data-ttu-id="70037-171">クラウド環境内のパブリックにアクセスできるすべてのエンドポイントは、SSL/TLS プロトコルを使用してパブリック インターネットで通信します。</span><span class="sxs-lookup"><span data-stu-id="70037-171">All publicly accessible endpoints in your cloud environment will communicate with the public internet using SSL/TLS protocols.</span></span>
- <span data-ttu-id="70037-172">クラウド ネットワークを、パブリック インターネット経由でオンプレミスまたは他の外部ネットワークと接続するときは、暗号化された VPN プロトコルを使用します。</span><span class="sxs-lookup"><span data-stu-id="70037-172">When connecting cloud networks with on-premises or other external network over the public internet, use encrypted VPN protocols.</span></span>
- <span data-ttu-id="70037-173">ExpressRoute などの専用 WAN 接続を使用して、クラウド ネットワークをオンプレミスや他の外部ネットワークと接続するときは、オンプレミスの VPN または他の暗号化アプライアンスと、クラウド ネットワークにデプロイされた対応する仮想 VPN または暗号化アプライアンスを、ペアにして使用します。</span><span class="sxs-lookup"><span data-stu-id="70037-173">When connecting cloud networks with on-premises or other external network using a dedicated WAN connection such as ExpressRoute, you will use a VPN or other encryption appliance on-premises paired with a corresponding virtual VPN or encryption appliance deployed to your cloud network.</span></span>
- <span data-ttu-id="70037-174">IT スタッフが見ることのできるトラフィック ログまたは他の診断レポートに含めるべきでない機密データがある場合は、仮想ネットワーク内のリソース間のすべてのトラフィックを暗号化します。</span><span class="sxs-lookup"><span data-stu-id="70037-174">If you have sensitive data that shouldn't be included in traffic logs or other diagnostics reports visible to IT staff, you will encrypt all traffic between resources in your virtual network.</span></span>

### <a name="data-at-rest"></a><span data-ttu-id="70037-175">保存データ</span><span class="sxs-lookup"><span data-stu-id="70037-175">Data at rest</span></span>

<span data-ttu-id="70037-176">保存データは、積極的に移動または処理されていないすべてのデータを表し、ファイル、データベース、仮想マシンのドライブ、PaaS のストレージ アカウント、または同様の資産が含まれます。</span><span class="sxs-lookup"><span data-stu-id="70037-176">Data at rest represents any data not being actively moved or processed, including files, databases, virtual machine drives, PaaS storage accounts, or similar assets.</span></span> <span data-ttu-id="70037-177">保存データの暗号化は、外部ネットワーク侵入、悪意のある内部ユーザー、または偶発的な解放からの不正アクセスに対して、仮想デバイスまたはファイルを保護します。</span><span class="sxs-lookup"><span data-stu-id="70037-177">Encrypting stored data protects virtual devices or files against unauthorized access either from external network penetration, rogue internal users, or accidental releases.</span></span>

<span data-ttu-id="70037-178">一般に、PaaS ストレージとデータベース リソースでは、既定で暗号化が適用されます。</span><span class="sxs-lookup"><span data-stu-id="70037-178">PaaS storage and database resources generally enforce encryption by default.</span></span> <span data-ttu-id="70037-179">IaaS の仮想リソースは、キー管理システムに格納されている暗号化キーを使用して、仮想ディスクの暗号化により保護できます。</span><span class="sxs-lookup"><span data-stu-id="70037-179">IaaS virtual resources can be secured through virtual disk encryption using cryptographic keys stored in your key management system.</span></span>

<span data-ttu-id="70037-180">保存データの暗号化には、保護対象の厳密なデータをいっそう細かく制御できる、列レベルと行レベルの暗号化のような高度なデータベースの暗号化技術も含まれます。</span><span class="sxs-lookup"><span data-stu-id="70037-180">Encryption for data at rest also encompasses more advanced database encryption techniques, such as column-level and row level encryption, which provides much more control over exactly what data is being secured.</span></span>

<span data-ttu-id="70037-181">ポリシーとコンプライアンスの全体的な要件、格納されているデータの機密性、およびワークロードのパフォーマンス要件に基づいて、暗号化を必要とする資産を決定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="70037-181">Your overall policy and compliance requirements, the sensitivity of the data being stored, and the performance requirements of your workloads should determine which assets require encryption.</span></span>

<span data-ttu-id="70037-182">**保存データの暗号化の前提条件**。</span><span class="sxs-lookup"><span data-stu-id="70037-182">**Encrypting Data at Rest Assumptions**.</span></span> <span data-ttu-id="70037-183">保存データの暗号化では、次のことが想定されます。</span><span class="sxs-lookup"><span data-stu-id="70037-183">Encrypting data at rest assumes the following:</span></span>

- <span data-ttu-id="70037-184">格納するデータは、公開を前提としたものではありません。</span><span class="sxs-lookup"><span data-stu-id="70037-184">You are storing data that is not meant for public consumption.</span></span>
- <span data-ttu-id="70037-185">ワークロードは、ディスク暗号化による待ち時間の増加を受け入れることができます。</span><span class="sxs-lookup"><span data-stu-id="70037-185">Your workloads can accept the added latency cost of disk encryption.</span></span>

### <a name="data-in-use"></a><span data-ttu-id="70037-186">使用中のデータ</span><span class="sxs-lookup"><span data-stu-id="70037-186">Data in use</span></span>

<span data-ttu-id="70037-187">使用中のデータの暗号化には、RAM や CPU キャッシュなどの非永続的ストレージ内のデータのセキュリティ保護が含まれます。</span><span class="sxs-lookup"><span data-stu-id="70037-187">Encryption for data in use involves securing data in nonpersistent storage, such as RAM or CPU caches.</span></span> <span data-ttu-id="70037-188">完全なメモリの暗号化などのテクノロジや、Intel の Secure Guard Extensions (SGX) などのエンクレーブ テクノロジーを使用します。</span><span class="sxs-lookup"><span data-stu-id="70037-188">Use of technologies such as full memory encryption, enclave technologies, such as Intel's Secure Guard Extensions (SGX).</span></span> <span data-ttu-id="70037-189">これには、安全で信頼できる実行環境を作成するために使用できる同形暗号化などの暗号化技術も含まれます。</span><span class="sxs-lookup"><span data-stu-id="70037-189">This also includes cryptographic techniques, such as homomorphic encryption that can be used to create secure, trusted execution environments.</span></span>

<span data-ttu-id="70037-190">**使用中のデータの暗号化の前提条件**。</span><span class="sxs-lookup"><span data-stu-id="70037-190">**Encrypting data in use assumptions**.</span></span> <span data-ttu-id="70037-191">使用中のデータの暗号化では、次のことが想定されます。</span><span class="sxs-lookup"><span data-stu-id="70037-191">Encrypting data in use assumes the following:</span></span>

- <span data-ttu-id="70037-192">常に (RAM や CPU レベルでも)、データの所有権を、基になるクラウド プラットフォームから切り離しておく必要があります。</span><span class="sxs-lookup"><span data-stu-id="70037-192">You are required to maintain data ownership separate from the underlying cloud platform at all times, even at the RAM and CPU level.</span></span>

## <a name="learn-more"></a><span data-ttu-id="70037-193">詳細情報</span><span class="sxs-lookup"><span data-stu-id="70037-193">Learn more</span></span>

<span data-ttu-id="70037-194">Azure プラットフォームでの暗号化とキー管理の詳細については、次の記事を参照してください。</span><span class="sxs-lookup"><span data-stu-id="70037-194">See the following for more information about encryption and key management in the Azure platform.</span></span>

- <span data-ttu-id="70037-195">「[Azure の暗号化の概要](/azure/security/security-azure-encryption-overview)」。</span><span class="sxs-lookup"><span data-stu-id="70037-195">[Azure encryption overview](/azure/security/security-azure-encryption-overview).</span></span> <span data-ttu-id="70037-196">Azure で暗号化を使用して保存データと転送中のデータの両方が保護される方法について詳細に説明されています。</span><span class="sxs-lookup"><span data-stu-id="70037-196">A detailed description of how Azure uses encryption to secure both data at rest and data in transit.</span></span>
- <span data-ttu-id="70037-197">[Azure Key Vault](/azure/key-vault/key-vault-overview)。</span><span class="sxs-lookup"><span data-stu-id="70037-197">[Azure Key Vault](/azure/key-vault/key-vault-overview).</span></span> <span data-ttu-id="70037-198">Key Vault は、Azure 内の暗号化キー、シークレット、証明書を格納および管理するための主要なキー管理システムです。</span><span class="sxs-lookup"><span data-stu-id="70037-198">Key Vault is the primary key management system for storing and managing cryptographic keys, secrets, and certificates within Azure.</span></span>
- <span data-ttu-id="70037-199">「[Confidential computing in Azure (Azure での Confidential Computing)](/solutions/confidential-compute)」。</span><span class="sxs-lookup"><span data-stu-id="70037-199">[Confidential computing in Azure](/solutions/confidential-compute).</span></span> <span data-ttu-id="70037-200">Azure の Confidential Computing イニシアチブでは、信頼できる実行環境または使用中のデータを保護するための他の暗号化メカニズムを作成するためのツールとテクノロジが提供されます。</span><span class="sxs-lookup"><span data-stu-id="70037-200">Azure's confidential computing initiative provides tools and technology to create trusted execution environments or other encryption mechanisms to secure data in use.</span></span>

## <a name="next-steps"></a><span data-ttu-id="70037-201">次の手順</span><span class="sxs-lookup"><span data-stu-id="70037-201">Next steps</span></span>

<span data-ttu-id="70037-202">ソフトウェア定義ネットワークでクラウドのデプロイに仮想化されたネットワーク機能が提供される方法について学習します。</span><span class="sxs-lookup"><span data-stu-id="70037-202">Learn how Software Defined Networks provide virtualized networking capabilities for cloud deployments.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="70037-203">デプロイに最も適したソフトウェア定義ネットワーク パターン</span><span class="sxs-lookup"><span data-stu-id="70037-203">Which Software Defined Network pattern is best for my deployment?</span></span>](../software-defined-network/overview.md)
