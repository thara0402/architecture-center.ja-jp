---
title: メインフレームの移行:通説と事実
description: メインフレームで現在実行されているシステムに関してメインフレーム環境から Azure にアプリケーションを移行します。Azure は可用性が高く、拡張可能なインフラストラクチャであることが証明されています。
author: njray
ms.date: 12/27/2018
ms.openlocfilehash: bcad01ec044d2d802b055e328a9496aae7b33311
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901474"
---
# <a name="mainframe-myths-and-facts"></a><span data-ttu-id="e9182-103">メインフレームに関する通説と事実</span><span class="sxs-lookup"><span data-stu-id="e9182-103">Mainframe myths and facts</span></span>

<span data-ttu-id="e9182-104">メインフレームはコンピューティングの歴史において際立っており、非常に特殊なワークロードに対して価値があることに変わりはありません。</span><span class="sxs-lookup"><span data-stu-id="e9182-104">Mainframes figure prominently in the history of computing and remain viable for highly specific workloads.</span></span> <span data-ttu-id="e9182-105">メインフレームは実績のあるプラットフォームであり、長い歴史を持つ運用手順によって信頼性が高く堅牢な環境が構成されているということに、異論はほとんどありません。</span><span class="sxs-lookup"><span data-stu-id="e9182-105">Most agree that mainframes are a proven platform with long-established operating procedures that make them reliable, robust environments.</span></span> <span data-ttu-id="e9182-106">ソフトウェアは使用状況に基づいて実行され、MIPS (百万命令/秒) の単位で測定されて、配賦のために広範な使用状況レポートを利用できます。</span><span class="sxs-lookup"><span data-stu-id="e9182-106">Software runs based on usage, measured in million instructions per second (MIPS), and extensive usage reports are available for charge backs.</span></span>

<span data-ttu-id="e9182-107">メインフレームの信頼性、可用性、処理能力はほとんど神話の域に達しています。</span><span class="sxs-lookup"><span data-stu-id="e9182-107">The reliability, availability, and processing power of mainframes have taken on almost mythical proportions.</span></span> <span data-ttu-id="e9182-108">Azure に最も適しているメインフレーム ワークロードを評価するには、最初に事実から通説を区別する必要があります。</span><span class="sxs-lookup"><span data-stu-id="e9182-108">To evaluate the mainframe workloads that are most suitable for Azure, you first want to distinguish the myths from the reality.</span></span>

## <a name="myth-mainframes-never-go-down-and-have-a-minimum-of-five-9s-of-availability"></a><span data-ttu-id="e9182-109">通説:メインフレームはダウンすることがなく、最低でもファイブ ナインの可用性である</span><span class="sxs-lookup"><span data-stu-id="e9182-109">Myth: Mainframes never go down and have a minimum of five 9s of availability</span></span>

<span data-ttu-id="e9182-110">メインフレームのハードウェアとオペレーティング システムは、信頼性が高く安定性しているものと見なされます。</span><span class="sxs-lookup"><span data-stu-id="e9182-110">Mainframe hardware and operating systems are viewed as reliable and stable.</span></span> <span data-ttu-id="e9182-111">しかし、実際には、メンテナンスと再起動のためのダウンタイムをスケジュールする必要があります (イニシャル プログラム ロードまたは IPL と呼ばれます)。</span><span class="sxs-lookup"><span data-stu-id="e9182-111">But the reality is that downtime must be scheduled for maintenance and reboots (referred to as initial program loads or IPLs).</span></span> <span data-ttu-id="e9182-112">これらのタスクを考慮すると、多くの場合、メインフレーム ソリューションの可用性はツー ナインまたはスリー ナインに近く、これはハイエンドの Intel ベース サーバーと同等です。</span><span class="sxs-lookup"><span data-stu-id="e9182-112">When these tasks are considered, a mainframe solution often has closer to two or three 9s of availability, which is equivalent to that of high-end, Intel-based servers.</span></span>

<span data-ttu-id="e9182-113">また、メインフレームは他のサーバーと同じように災害に対して脆弱であり、この種の障害に対処するために無停電電源 (UPS) システムを必要とします。</span><span class="sxs-lookup"><span data-stu-id="e9182-113">Mainframes also remain as vulnerable to disasters as any other servers do, and require uninterruptible power supply (UPS) systems to handle these types of failures.</span></span>

## <a name="myth-mainframes-have-limitless-scalability"></a><span data-ttu-id="e9182-114">通説:メインフレームのスケーラビリティは無限である</span><span class="sxs-lookup"><span data-stu-id="e9182-114">Myth: Mainframes have limitless scalability</span></span>

<span data-ttu-id="e9182-115">メインフレームのスケーラビリティは、顧客情報管理システム (CICS) などのシステム ソフトウェアのキャパシティ、およびメインフレームのエンジンとストレージの新しいインスタンスのキャパシティに依存します。</span><span class="sxs-lookup"><span data-stu-id="e9182-115">A mainframe’s scalability depends on the capacity of its system software, such as the customer information control system (CICS), and the capacity of new instances of mainframe engines and storage.</span></span> <span data-ttu-id="e9182-116">メインフレームを使用する一部の大企業では、パフォーマンスのために CICS をカスタマイズしており、そうしないと、利用可能な最大のメインフレームの機能を超えています。</span><span class="sxs-lookup"><span data-stu-id="e9182-116">Some large companies that use mainframes have customized their CICS for performance, and have otherwise outgrown the capability of the largest available mainframes.</span></span>

## <a name="myth-intel-based-servers-are-not-as-powerful-as-mainframes"></a><span data-ttu-id="e9182-117">通説:Intel ベースのサーバーはメインフレームほど強力ではない</span><span class="sxs-lookup"><span data-stu-id="e9182-117">Myth: Intel-based servers are not as powerful as mainframes</span></span>

<span data-ttu-id="e9182-118">新しいコア密度の Intel ベースのシステムのコンピューティング キャパシティはメインフレームに匹敵します。</span><span class="sxs-lookup"><span data-stu-id="e9182-118">The new core-dense, Intel-based systems have as much compute capacity as mainframes.</span></span>

## <a name="myth-the-cloud-cannot-accommodate-mission-critical-applications-for-large-companies-such-as-financial-institutions"></a><span data-ttu-id="e9182-119">通説:クラウドでは、金融機関などの大企業のミッション クリティカルなアプリケーションに対応できない</span><span class="sxs-lookup"><span data-stu-id="e9182-119">Myth: The cloud cannot accommodate mission-critical applications for large companies, such as financial institutions</span></span>

<span data-ttu-id="e9182-120">クラウド ソリューションでは要求が満たされない例がまれにありますが、通常はアプリケーションのアルゴリズムを分散できないためです。</span><span class="sxs-lookup"><span data-stu-id="e9182-120">Although there may be some isolated instances where cloud solutions fall short, it is usually becuase the application algorithms cannot be distributed.</span></span> <span data-ttu-id="e9182-121">これらの少数の例は例外であって、ルールではありません。</span><span class="sxs-lookup"><span data-stu-id="e9182-121">These few examples are the exceptions, not the rule.</span></span>

## <a name="summary"></a><span data-ttu-id="e9182-122">まとめ</span><span class="sxs-lookup"><span data-stu-id="e9182-122">Summary</span></span>

<span data-ttu-id="e9182-123">比較すると、Azure では、メインフレームの同等の機能を格段に低いコストで提供できる、代替プラットフォームが提供されます。</span><span class="sxs-lookup"><span data-stu-id="e9182-123">By comparison, Azure offers  an alternative platform that is capable of delivering equivalent mainframe functionality and features, and at a much lower cost.</span></span> <span data-ttu-id="e9182-124">さらに、クラウドのサブスクリプションに基づく使用量によるコスト モデルの総保有コスト (TCO) は、メインフレーム コンピューターよりはるかに低コストです。</span><span class="sxs-lookup"><span data-stu-id="e9182-124">In addition, the total cost of ownership (TCO) of the cloud’s subscription-based, usage-driven cost model is far less expensive than mainframe computers.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e9182-125">次の手順</span><span class="sxs-lookup"><span data-stu-id="e9182-125">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="e9182-126">メインフレームから Azure に切り替える</span><span class="sxs-lookup"><span data-stu-id="e9182-126">Make the Switch from Mainframes to Azure</span></span>](migration-strategies.md)
