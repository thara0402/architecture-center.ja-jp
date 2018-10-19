---
title: Azure での SAP ワークロード向けの開発/テスト環境
description: SAP ワークロード向けの開発/テスト環境を構築します。
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: b47e4cb527d3e4ecd74bee7bcf08f2794da56d6c
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876801"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="9f191-103">Azure での SAP ワークロード向けの開発/テスト環境</span><span class="sxs-lookup"><span data-stu-id="9f191-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="9f191-104">この例は、Windows または Linux 環境の SAP NetWeaver の開発/テスト環境を Azure 上に構築する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="9f191-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="9f191-105">使用するデータベースは AnyDB で、これはサポートされている任意の DBMS を表す SAP 用語 (SAP HANA 以外) です。</span><span class="sxs-lookup"><span data-stu-id="9f191-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="9f191-106">このアーキテクチャは、運用環境以外を対象に設計されているため、単一の仮想マシン (VM) のみでデプロイされ、サイズは組織のニーズに合わせて変更できます。</span><span class="sxs-lookup"><span data-stu-id="9f191-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="9f191-107">運用ユース ケースについては、以下で使用できる SAP リファレンス アーキテクチャを確認してください。</span><span class="sxs-lookup"><span data-stu-id="9f191-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="9f191-108">[AnyDB 向けの SAP NetWeaver][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="9f191-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="9f191-109">[SAP S/4HANA][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="9f191-109">[SAP S/4HANA][sap-hana]</span></span>
* <span data-ttu-id="9f191-110">[SAP on Azure L インスタンス][sap-large]</span><span class="sxs-lookup"><span data-stu-id="9f191-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="9f191-111">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="9f191-111">Relevant use cases</span></span>

<span data-ttu-id="9f191-112">次のユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="9f191-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="9f191-113">重要度、生産性が低い SAP ワークロード (サンドボックス、開発、テスト、品質保証)</span><span class="sxs-lookup"><span data-stu-id="9f191-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="9f191-114">重要度が低い SAP ビジネス ワークロード</span><span class="sxs-lookup"><span data-stu-id="9f191-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="9f191-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="9f191-115">Architecture</span></span>

![SAP ワークロードの開発/テスト環境のアーキテクチャ図](media/architecture-sap-dev-test.png)

<span data-ttu-id="9f191-117">このシナリオは、単一の仮想マシンに単一の SAP システム データベースと SAP アプリケーション サーバーをプロビジョニングする方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="9f191-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="9f191-118">このシナリオのデータ フローは次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="9f191-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="9f191-119">お客様は、SAP ユーザー インターフェイスまたは他のクライアント ツール (Excel、Web ブラウザー、またはその他の Web アプリケーション) を使用して、Azure ベースの SAP システムにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="9f191-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="9f191-120">確立された ExpressRoute を使用して接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="9f191-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="9f191-121">ExpressRoute 接続は、Azure の ExpressRoute ゲートウェイが終端です。</span><span class="sxs-lookup"><span data-stu-id="9f191-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="9f191-122">ネットワーク トラフィックが、ExpressRoute ゲートウェイを介してゲートウェイ サブネット、アプリケーション層スポーク サブネットの順にルーティングされ ([ハブスポーク][hub-spoke] パターンを参照)、さらにネットワーク セキュリティ ゲートウェイ経由で SAP アプリケーションの仮想マシンにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="9f191-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="9f191-123">ID 管理サーバーは、認証サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="9f191-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="9f191-124">ジャンプ ボックスは、ローカル管理機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="9f191-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="9f191-125">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="9f191-125">Components</span></span>

* <span data-ttu-id="9f191-126">[仮想ネットワーク](/azure/virtual-network/virtual-networks-overview)は、Azure 内のネットワーク通信の基盤です</span><span class="sxs-lookup"><span data-stu-id="9f191-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
* <span data-ttu-id="9f191-127">[仮想マシン](/azure/virtual-machines/windows/overview)である Azure Virtual Machines では、Windows または Linux サーバーを使用して、セキュリティで保護された高スケールな仮想化インフラストラクチャをオンデマンドで構築できます。</span><span class="sxs-lookup"><span data-stu-id="9f191-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
* <span data-ttu-id="9f191-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) を利用すると、接続プロバイダーが提供するプライベート接続を介して、オンプレミスのネットワークを Microsoft クラウドに拡張できます。</span><span class="sxs-lookup"><span data-stu-id="9f191-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="9f191-129">[ネットワーク セキュリティ グループ](/azure/virtual-network/security-overview)を使用すると、仮想ネットワーク内のリソースへのネットワーク トラフィックを制限できます。</span><span class="sxs-lookup"><span data-stu-id="9f191-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="9f191-130">ネットワーク セキュリティ グループには、ソースまたはターゲット IP アドレス、ポート、およびプロトコルを基に、受信/送信ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。</span><span class="sxs-lookup"><span data-stu-id="9f191-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 
* <span data-ttu-id="9f191-131">[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)は、Azure リソースの論理コンテナーとして機能します。</span><span class="sxs-lookup"><span data-stu-id="9f191-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="9f191-132">考慮事項</span><span class="sxs-lookup"><span data-stu-id="9f191-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="9f191-133">可用性</span><span class="sxs-lookup"><span data-stu-id="9f191-133">Availability</span></span>

 <span data-ttu-id="9f191-134">Microsoft は 1 つの VM インスタンスに対してサービス レベル アグリーメント (SLA) を提供します。</span><span class="sxs-lookup"><span data-stu-id="9f191-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="9f191-135">Virtual Machines の Microsoft Azure サービス レベル アグリーメントの詳細については、「[Virtual Machines の SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)」を参照してください</span><span class="sxs-lookup"><span data-stu-id="9f191-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="9f191-136">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="9f191-136">Scalability</span></span>

<span data-ttu-id="9f191-137">スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9f191-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="9f191-138">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="9f191-138">Security</span></span>

<span data-ttu-id="9f191-139">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="9f191-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="9f191-140">回復性</span><span class="sxs-lookup"><span data-stu-id="9f191-140">Resiliency</span></span>

<span data-ttu-id="9f191-141">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="9f191-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="9f191-142">価格</span><span class="sxs-lookup"><span data-stu-id="9f191-142">Pricing</span></span>

<span data-ttu-id="9f191-143">このシナリオの実行コストを調べることができるよう、すべてのサービスが料金計算ツールで事前構成されています。以下に例を示します。</span><span class="sxs-lookup"><span data-stu-id="9f191-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="9f191-144">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="9f191-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="9f191-145">受信するトラフィックの量に基づいて、次の 4 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="9f191-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="9f191-146">サイズ</span><span class="sxs-lookup"><span data-stu-id="9f191-146">Size</span></span>|<span data-ttu-id="9f191-147">SAP</span><span class="sxs-lookup"><span data-stu-id="9f191-147">SAPs</span></span>|<span data-ttu-id="9f191-148">VM の種類</span><span class="sxs-lookup"><span data-stu-id="9f191-148">VM Type</span></span>|<span data-ttu-id="9f191-149">Storage</span><span class="sxs-lookup"><span data-stu-id="9f191-149">Storage</span></span>|<span data-ttu-id="9f191-150">Azure 料金計算ツール</span><span class="sxs-lookup"><span data-stu-id="9f191-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="9f191-151">Small</span><span class="sxs-lookup"><span data-stu-id="9f191-151">Small</span></span>|<span data-ttu-id="9f191-152">8000</span><span class="sxs-lookup"><span data-stu-id="9f191-152">8000</span></span>|<span data-ttu-id="9f191-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="9f191-153">D8s_v3</span></span>|<span data-ttu-id="9f191-154">P20 x 2、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="9f191-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="9f191-155">Small</span><span class="sxs-lookup"><span data-stu-id="9f191-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="9f191-156">Medium</span><span class="sxs-lookup"><span data-stu-id="9f191-156">Medium</span></span>|<span data-ttu-id="9f191-157">16000</span><span class="sxs-lookup"><span data-stu-id="9f191-157">16000</span></span>|<span data-ttu-id="9f191-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="9f191-158">D16s_v3</span></span>|<span data-ttu-id="9f191-159">P20 x 3、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="9f191-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="9f191-160">Medium</span><span class="sxs-lookup"><span data-stu-id="9f191-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="9f191-161">Large</span><span class="sxs-lookup"><span data-stu-id="9f191-161">Large</span></span>|<span data-ttu-id="9f191-162">32000</span><span class="sxs-lookup"><span data-stu-id="9f191-162">32000</span></span>|<span data-ttu-id="9f191-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="9f191-163">E32s_v3</span></span>|<span data-ttu-id="9f191-164">P20 x 3、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="9f191-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="9f191-165">Large</span><span class="sxs-lookup"><span data-stu-id="9f191-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="9f191-166">Extra Large</span><span class="sxs-lookup"><span data-stu-id="9f191-166">Extra Large</span></span>|<span data-ttu-id="9f191-167">64000</span><span class="sxs-lookup"><span data-stu-id="9f191-167">64000</span></span>|<span data-ttu-id="9f191-168">M64s</span><span class="sxs-lookup"><span data-stu-id="9f191-168">M64s</span></span>|<span data-ttu-id="9f191-169">P20 x 4、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="9f191-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="9f191-170">Extra Large</span><span class="sxs-lookup"><span data-stu-id="9f191-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="9f191-171">この価格は、単に VM とストレージの料金を示す目安です。</span><span class="sxs-lookup"><span data-stu-id="9f191-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="9f191-172">ネットワーク、バックアップ ストレージ、データ イングレス/エグレスの料金は含まれていません。</span><span class="sxs-lookup"><span data-stu-id="9f191-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

* <span data-ttu-id="9f191-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): 小規模なシステムは、VM の種類 D8s_v3 (8 個の vCPU)、32 GB の RAM、200 GB の一時ストレージ、および 2 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="9f191-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="9f191-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): 中規模なシステムは、VM の種類 D16s_v3 (16 個の vCPU)、64 GB の RAM、400 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="9f191-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="9f191-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): 大規模なシステムは、VM の種類 E32s_v3 (32 個の vCPU)、256 GB の RAM、512 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="9f191-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="9f191-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): 超大規模なシステムは、VM の種類 M64s (64 個の vCPU)、1,024 GB の RAM、2,000 GB の一時ストレージ、および 4 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="9f191-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="9f191-177">Deployment</span><span class="sxs-lookup"><span data-stu-id="9f191-177">Deployment</span></span>

<span data-ttu-id="9f191-178">次をクリックして、このシナリオの基盤となるインフラストラクチャをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="9f191-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> <span data-ttu-id="9f191-179">このデプロイでは、SAP と Oracle はインストールされません。</span><span class="sxs-lookup"><span data-stu-id="9f191-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="9f191-180">それらのコンポーネントは、個別にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="9f191-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
