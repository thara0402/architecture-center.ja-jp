<!-- TEMPLATE FILE - DO NOT ADD METADATA -->

## <a name="policy-statements"></a><span data-ttu-id="15597-101">ポリシー ステートメント</span><span class="sxs-lookup"><span data-stu-id="15597-101">Policy statements</span></span>

<span data-ttu-id="15597-102">以下のポリシー ステートメントにより、定義されたリスクを軽減するために必要な要件が確立されます。</span><span class="sxs-lookup"><span data-stu-id="15597-102">The following policy statements establish the requirements needed to mitigate the defined risks.</span></span> <span data-ttu-id="15597-103">これらのポリシーでは、MVP ガバナンスに対する機能要件が定義されています。</span><span class="sxs-lookup"><span data-stu-id="15597-103">These policies define the functional requirements for the governance MVP.</span></span> <span data-ttu-id="15597-104">それぞれは、ガバナンス MVP の実装で表されます。</span><span class="sxs-lookup"><span data-stu-id="15597-104">Each will be represented in the implementation of the governance MVP.</span></span>

<span data-ttu-id="15597-105">デプロイの高速化:</span><span class="sxs-lookup"><span data-stu-id="15597-105">Deployment Acceleration:</span></span>

- <span data-ttu-id="15597-106">すべての資産は、定義されているグループ化とタグ付けの戦略に従って、グループ化およびタグ付けされる必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-106">All assets must be grouped and tagged according to defined grouping and tagging strategies.</span></span>
- <span data-ttu-id="15597-107">すべての資産では、承認済みのデプロイ モデルを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-107">All assets must use an approved deployment model.</span></span>
- <span data-ttu-id="15597-108">クラウド プロバイダーに対するガバナンス基盤が確立された後、すべてのデプロイ ツールは、ガバナンス チームによって定義されたツールとの互換性を備えている必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-108">Once a governance foundation has been established for a cloud provider, any deployment tooling must be compatible with the tools defined by the governance team.</span></span>

<span data-ttu-id="15597-109">ID ベースライン:</span><span class="sxs-lookup"><span data-stu-id="15597-109">Identity Baseline:</span></span>

- <span data-ttu-id="15597-110">クラウドにデプロイされるすべての資産は、現在のガバナンス ポリシーによって承認された ID とロールを使用して制御する必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-110">All assets deployed to the cloud should be controlled using identities and roles approved by current governance policies.</span></span>
- <span data-ttu-id="15597-111">オンプレミスの Active Directory インフラストラクチャ内にあり、昇格された特権を持つすべてのグループを、承認済みの RBAC ロールにマップする必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-111">All groups in the on-premises Active Directory infrastructure that have elevated privileges should be mapped to an approved RBAC role.</span></span>

<span data-ttu-id="15597-112">セキュリティ ベースライン:</span><span class="sxs-lookup"><span data-stu-id="15597-112">Security Baseline:</span></span>

- <span data-ttu-id="15597-113">クラウドにデプロイされるすべての資産は、承認済みのデータ分類を持つ必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-113">Any asset deployed to the cloud must have an approved data classification.</span></span>
- <span data-ttu-id="15597-114">セキュリティとガバナンスに関する十分な要件を承認して実装できるようになるまで、保護対象レベルのデータと識別された資産をクラウドにデプロイすることはできません。</span><span class="sxs-lookup"><span data-stu-id="15597-114">No assets identified with a protected level of data may be deployed to the cloud, until sufficient requirements for security and governance can be approved and implemented.</span></span>
- <span data-ttu-id="15597-115">最低限のネットワーク セキュリティ要件を検証して管理できるようになるまで、クラウド環境は非武装地帯と見なされ、他のデータ センターまたは内部ネットワークと同様の接続要件を満たす必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-115">Until minimum network security requirements can be validated and governed, cloud environments are seen as a demilitarized zone and should meet similar connection requirements to other data centers or internal networks.</span></span>

<span data-ttu-id="15597-116">コスト管理:</span><span class="sxs-lookup"><span data-stu-id="15597-116">Cost Management:</span></span>

- <span data-ttu-id="15597-117">追跡目的のため、すべての資産は、コア ビジネス機能の 1 つのアプリケーション所有者に割り当てる必要があります。</span><span class="sxs-lookup"><span data-stu-id="15597-117">For tracking purposes, all assets must be assigned to an application owner within one of the core business functions.</span></span>
- <span data-ttu-id="15597-118">コストに関する問題が発生したときは、追加のガバナンス要件が財務チームによって確立されます。</span><span class="sxs-lookup"><span data-stu-id="15597-118">When cost concerns arise, additional governance requirements will be established with the Finance team.</span></span>

<span data-ttu-id="15597-119">リソースの整合性:</span><span class="sxs-lookup"><span data-stu-id="15597-119">Resource Consistency:</span></span>

- <span data-ttu-id="15597-120">このステージではミッション クリティカルなワークロードはデプロイされないため、管理対象となる SLA、パフォーマンス、BCDR の要件はありません。</span><span class="sxs-lookup"><span data-stu-id="15597-120">Because no mission-critical workloads are deployed at this stage, there are no SLA, performance, or BCDR requirements to be governed.</span></span>
- <span data-ttu-id="15597-121">ミッション クリティカルなワークロードをデプロイする時点で、IT 運用部門によって追加のガバナンス要件が確立されます。</span><span class="sxs-lookup"><span data-stu-id="15597-121">When mission-critical workloads are deployed, additional governance requirements will be established with IT operations.</span></span>

## <a name="processes"></a><span data-ttu-id="15597-122">処理</span><span class="sxs-lookup"><span data-stu-id="15597-122">Processes</span></span>

<span data-ttu-id="15597-123">これらのガバナンス ポリシーの継続的な監視しと適用に対して、予算は割り当てられていません。</span><span class="sxs-lookup"><span data-stu-id="15597-123">No budget has been allocated for ongoing monitoring and enforcement of these governance policies.</span></span> <span data-ttu-id="15597-124">そのため、クラウド ガバナンス チームは何らかのアドホックな手段を使用してポリシー ステートメントへの準拠を監視します。</span><span class="sxs-lookup"><span data-stu-id="15597-124">Because of that, the Cloud Governance team has some ad hoc ways to monitor adherence to policy statements.</span></span>

- <span data-ttu-id="15597-125">**教育**:クラウド ガバナンス チームは、これらのポリシーをサポートするガバナンス体験において、クラウド導入チームの教育に時間を費やしています。</span><span class="sxs-lookup"><span data-stu-id="15597-125">**Education**: The Cloud Governance team is investing time to educate the cloud adoption teams on the governance journeys that support these policies.</span></span>
- <span data-ttu-id="15597-126">**デプロイ**のレビュー:資産をデプロイする前に、クラウド ガバナンス チームはクラウド導入チームと共にガバナンス体験を確認します。</span><span class="sxs-lookup"><span data-stu-id="15597-126">**Deployment** reviews: Before deploying any asset, the Cloud Governance team will review the governance journey with the cloud adoption teams.</span></span>
