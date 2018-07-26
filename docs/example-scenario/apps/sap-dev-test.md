---
title: 開発/テスト ワークロード用の SAP
description: 開発/テスト環境の SAP シナリオ
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060963"
---
# <a name="sap-for-devtest-workloads"></a><span data-ttu-id="b19ee-103">開発/テスト ワークロード用の SAP</span><span class="sxs-lookup"><span data-stu-id="b19ee-103">SAP for dev/test workloads</span></span>

<span data-ttu-id="b19ee-104">この例では、Azure で Windows または Linux 環境の SAP NetWeaver の開発/テスト実装を実行する方法に関するガイダンスを提供します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-104">This example provides guidance for how to run a dev/test implementation of SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="b19ee-105">使用するデータベースは AnyDB で、これはサポートされている任意の DBMS を表す SAP 用語 (SAP HANA 以外) です。</span><span class="sxs-lookup"><span data-stu-id="b19ee-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="b19ee-106">このアーキテクチャは、運用環境以外を対象に設計されているため、単一の仮想マシン (VM) のみでデプロイされ、サイズは組織のニーズに合わせて変更できます。</span><span class="sxs-lookup"><span data-stu-id="b19ee-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="b19ee-107">運用ユース ケースについては、以下で使用できる SAP リファレンス アーキテクチャを確認してください。</span><span class="sxs-lookup"><span data-stu-id="b19ee-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="b19ee-108">[AnyDB 向けの SAP NetWeaver][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="b19ee-108">[SAP netweaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="b19ee-109">[SAP S/4Hana][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="b19ee-109">[SAP S/4Hana][sap-hana]</span></span>
* <span data-ttu-id="b19ee-110">[SAP on Azure L インスタンス][sap-large]</span><span class="sxs-lookup"><span data-stu-id="b19ee-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="b19ee-111">関連するユース ケース</span><span class="sxs-lookup"><span data-stu-id="b19ee-111">Related use cases</span></span>

<span data-ttu-id="b19ee-112">次のユース ケースについて、このシナリオを検討してください。</span><span class="sxs-lookup"><span data-stu-id="b19ee-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="b19ee-113">重要度、生産性が低い SAP ワークロード (サンドボックス、開発、テスト、品質保証)</span><span class="sxs-lookup"><span data-stu-id="b19ee-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="b19ee-114">重要度が低い SAP Business One ワークロード</span><span class="sxs-lookup"><span data-stu-id="b19ee-114">Non-critical SAP business one workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="b19ee-115">アーキテクチャ</span><span class="sxs-lookup"><span data-stu-id="b19ee-115">Architecture</span></span>

![ダイアグラム](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

<span data-ttu-id="b19ee-117">このシナリオでは、1 つの仮想マシンにおける 1 つの SAP システム データベースと SAP アプリケーション サーバーのプロビジョニングに対応できます。シナリオのデータ フローを次に示します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-117">This scenario covers the provision of a single SAP system database and SAP application Server on a single virtual machine, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="b19ee-118">プレゼンテーション層の顧客が、SAP GUI、またはオンプレミスの他のユーザー インターフェイス (Internet Explorer、Excel、または他の Web アプリケーション) を使用して、Azure ベースの SAP システムにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="b19ee-118">Customers from the Presentation Tier use their SAP gui, or other user interfaces (Internet Explorer, Excel or other web application) on premise to access the Azure based SAP system.</span></span>
2. <span data-ttu-id="b19ee-119">確立された Express Route を使用して接続が提供されます。</span><span class="sxs-lookup"><span data-stu-id="b19ee-119">Connectivity is provided through the use of the established Express Route.</span></span> <span data-ttu-id="b19ee-120">Express Route は、Azure の Express Route ゲートウェイで終了します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-120">The Express Route is terminated in Azure at the Express Route Gateway.</span></span> <span data-ttu-id="b19ee-121">ネットワーク トラフィックが、Express Route ゲートウェイを介してゲートウェイ サブネット、アプリケーション層スポーク サブネットの順にルーティングされ ([ハブ スポーク][hub-spoke] パターンを参照)、さらにネットワーク セキュリティ ゲートウェイ経由で SAP アプリケーションの仮想マシンにルーティングされます。</span><span class="sxs-lookup"><span data-stu-id="b19ee-121">Network traffic routes through the Express Route gateway to the Gateway Subnet and from the gateway subnet to the Application Tier Spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="b19ee-122">ID 管理サーバーは、認証サービスを提供します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-122">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="b19ee-123">ジャンプ ボックスは、ローカル管理機能を提供します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-123">The Jump Box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="b19ee-124">コンポーネント</span><span class="sxs-lookup"><span data-stu-id="b19ee-124">Components</span></span>

* <span data-ttu-id="b19ee-125">[リソース グループ](/azure/azure-resource-manager/resource-group-overview#resource-groups)は、Azure リソースの論理コンテナーです。</span><span class="sxs-lookup"><span data-stu-id="b19ee-125">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) is a logical container for Azure resources.</span></span>
* <span data-ttu-id="b19ee-126">[仮想ネットワーク](/azure/virtual-network/virtual-networks-overview)は、Azure 内のネットワーク通信の基盤です</span><span class="sxs-lookup"><span data-stu-id="b19ee-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) is the basis of network communications within Azure</span></span>
* <span data-ttu-id="b19ee-127">[仮想マシン](/azure/virtual-machines/windows/overview)である Azure Virtual Machines では、Windows または Linux サーバーを使用して、セキュリティで保護された高スケールな仮想化インフラストラクチャをオンデマンドで構築できます</span><span class="sxs-lookup"><span data-stu-id="b19ee-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server</span></span>
* <span data-ttu-id="b19ee-128">[Express Route](/azure/expressroute/expressroute-introduction) を利用すると、接続プロバイダーが提供するプライベート接続を介して、オンプレミスのネットワークを Microsoft クラウドに拡張できます。</span><span class="sxs-lookup"><span data-stu-id="b19ee-128">[Express Route](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="b19ee-129">[ネットワーク セキュリティ グループ](/azure/virtual-network/security-overview)を使用すると、仮想ネットワーク内のリソースへのネットワーク トラフィックを制限できます。</span><span class="sxs-lookup"><span data-stu-id="b19ee-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="b19ee-130">ネットワーク セキュリティ グループには、ソースまたはターゲット IP アドレス、ポート、およびプロトコルを基に、受信/送信ネットワーク トラフィックを許可または拒否するセキュリティ規則の一覧が含まれています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 

## <a name="considerations"></a><span data-ttu-id="b19ee-131">考慮事項</span><span class="sxs-lookup"><span data-stu-id="b19ee-131">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="b19ee-132">可用性</span><span class="sxs-lookup"><span data-stu-id="b19ee-132">Availability</span></span>

 <span data-ttu-id="b19ee-133">Microsoft は 1 つの VM インスタンスに対してサービス レベル アグリーメント (SLA) を提供します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-133">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="b19ee-134">Virtual Machines の Microsoft Azure サービス レベル アグリーメントの詳細については、「[Virtual Machines の SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)」を参照してください</span><span class="sxs-lookup"><span data-stu-id="b19ee-134">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="b19ee-135">スケーラビリティ</span><span class="sxs-lookup"><span data-stu-id="b19ee-135">Scalability</span></span>

<span data-ttu-id="b19ee-136">スケーラブルなソリューションの設計に関する一般的なガイダンスについては、Azure アーキテクチャ センターの[スケーラビリティのチェックリスト][scalability]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b19ee-136">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="b19ee-137">セキュリティ</span><span class="sxs-lookup"><span data-stu-id="b19ee-137">Security</span></span>

<span data-ttu-id="b19ee-138">セキュリティで保護されたソリューションの設計に関する一般的なガイダンスについては、「[Azure のセキュリティのドキュメント][security]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="b19ee-138">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="b19ee-139">回復性</span><span class="sxs-lookup"><span data-stu-id="b19ee-139">Resiliency</span></span>

<span data-ttu-id="b19ee-140">回復性に優れたソリューションの設計に関する一般的なガイダンスについては、「[回復性に優れた Azure 用アプリケーションの設計][resiliency]」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="b19ee-140">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="b19ee-141">価格</span><span class="sxs-lookup"><span data-stu-id="b19ee-141">Pricing</span></span>

<span data-ttu-id="b19ee-142">このシナリオの実行コストを調べてください。すべてのサービスがコスト計算ツールで事前構成されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-142">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span>  <span data-ttu-id="b19ee-143">特定のユース ケースについて価格の変化を確認するには、予想されるトラフィックに合わせて該当する変数を変更します。</span><span class="sxs-lookup"><span data-stu-id="b19ee-143">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="b19ee-144">取得するトラフィックの量に基づいて、次の 4 つのサンプル コスト プロファイルが用意されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-144">We have provided four sample cost profiles based on amount of traffic you expect to get:</span></span>

|<span data-ttu-id="b19ee-145">サイズ</span><span class="sxs-lookup"><span data-stu-id="b19ee-145">Size</span></span>|<span data-ttu-id="b19ee-146">SAP</span><span class="sxs-lookup"><span data-stu-id="b19ee-146">SAPs</span></span>|<span data-ttu-id="b19ee-147">VM の種類</span><span class="sxs-lookup"><span data-stu-id="b19ee-147">VM Type</span></span>|<span data-ttu-id="b19ee-148">Storage</span><span class="sxs-lookup"><span data-stu-id="b19ee-148">Storage</span></span>|<span data-ttu-id="b19ee-149">Azure 料金計算ツール</span><span class="sxs-lookup"><span data-stu-id="b19ee-149">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="b19ee-150">Small</span><span class="sxs-lookup"><span data-stu-id="b19ee-150">Small</span></span>|<span data-ttu-id="b19ee-151">8000</span><span class="sxs-lookup"><span data-stu-id="b19ee-151">8000</span></span>|<span data-ttu-id="b19ee-152">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="b19ee-152">D8s_v3</span></span>|<span data-ttu-id="b19ee-153">P20 x 2、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="b19ee-153">2xP20, 1xP10</span></span>|[<span data-ttu-id="b19ee-154">Small</span><span class="sxs-lookup"><span data-stu-id="b19ee-154">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="b19ee-155">Medium</span><span class="sxs-lookup"><span data-stu-id="b19ee-155">Medium</span></span>|<span data-ttu-id="b19ee-156">16000</span><span class="sxs-lookup"><span data-stu-id="b19ee-156">16000</span></span>|<span data-ttu-id="b19ee-157">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="b19ee-157">D16s_v3</span></span>|<span data-ttu-id="b19ee-158">P20 x 3、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="b19ee-158">3xP20, 1xP10</span></span>|[<span data-ttu-id="b19ee-159">Medium</span><span class="sxs-lookup"><span data-stu-id="b19ee-159">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="b19ee-160">Large</span><span class="sxs-lookup"><span data-stu-id="b19ee-160">Large</span></span>|<span data-ttu-id="b19ee-161">32000</span><span class="sxs-lookup"><span data-stu-id="b19ee-161">32000</span></span>|<span data-ttu-id="b19ee-162">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="b19ee-162">E32s_v3</span></span>|<span data-ttu-id="b19ee-163">P20 x 3、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="b19ee-163">3xP20, 1xP10</span></span>|[<span data-ttu-id="b19ee-164">Large</span><span class="sxs-lookup"><span data-stu-id="b19ee-164">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="b19ee-165">Extra Large</span><span class="sxs-lookup"><span data-stu-id="b19ee-165">Extra Large</span></span>|<span data-ttu-id="b19ee-166">64000</span><span class="sxs-lookup"><span data-stu-id="b19ee-166">64000</span></span>|<span data-ttu-id="b19ee-167">M64s</span><span class="sxs-lookup"><span data-stu-id="b19ee-167">M64s</span></span>|<span data-ttu-id="b19ee-168">P20 x 4、P10 x 1</span><span class="sxs-lookup"><span data-stu-id="b19ee-168">4xP20, 1xP10</span></span>|[<span data-ttu-id="b19ee-169">Extra Large</span><span class="sxs-lookup"><span data-stu-id="b19ee-169">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

<span data-ttu-id="b19ee-170">注: 価格はガイドであり、VM とストレージのコストのみを示しています (ネットワーク、バックアップ ストレージ、データの受信/送信料金は含まれません)。</span><span class="sxs-lookup"><span data-stu-id="b19ee-170">Note: pricing is a guide and only indicates the VMs and storage costs (excludes, networking, backup storage and data ingress/egress charges).</span></span>

* <span data-ttu-id="b19ee-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): 小規模なシステムは、VM の種類 D8s_v3 (8 個の vCPU)、32 GB の RAM、200 GB の一時ストレージ、および 2 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32GB RAM and 200GB temp storage, additionally two 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="b19ee-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): 中規模なシステムは、VM の種類は D16s_v3 (16 個の vCPU)、64 GB の RAM、400 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64GB RAM and 400GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="b19ee-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): 大規模なシステムは、VM の種類 E32s_v3 (32 個の vCPU)、256 GB の RAM、512 GB の一時ストレージ、および 3 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256GB RAM and 512GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="b19ee-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): 非常に大規模なシステムは、VM の種類 M64s (64 個の vCPU)、1024 GB の RAM、2000 GB の一時ストレージ、および 4 つの 512 GB と 1 つの 128 GB Premium Storage ディスクで構成されています。</span><span class="sxs-lookup"><span data-stu-id="b19ee-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024GB RAM and 2000GB temp storage, additionally four 512GB and one 128GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="b19ee-175">Deployment</span><span class="sxs-lookup"><span data-stu-id="b19ee-175">Deployment</span></span>

<span data-ttu-id="b19ee-176">上記のシナリオに似た基盤インフラストラクチャをデプロイするには、デプロイ ボタンを使用してください</span><span class="sxs-lookup"><span data-stu-id="b19ee-176">To deploy the underlying infrastructure similar to the scenario above, please use the deploy button</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="b19ee-177">\* SAP はインストールされないため、インフラストラクチャを手動で構築した後、インストールする必要があります。</span><span class="sxs-lookup"><span data-stu-id="b19ee-177">\* SAP will not be installed, you'll need to do this after the infrastructure is built manually.</span></span>

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke