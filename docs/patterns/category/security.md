---
title: セキュリティのパターン
titleSuffix: Cloud Design Patterns
description: セキュリティとは、設計された用途以外の悪意あるアクションや偶発的なアクションを防ぎ、情報の漏えいや損失を防ぐシステムの機能です。 クラウド アプリケーションは、信頼できるオンプレミス境界の外側でインターネット上に存在します。一般に公開されていることも多く、信頼できないユーザーが利用する場合もあります。 アプリケーションの設計とデプロイでは、悪意のある攻撃から保護し、承認したユーザーのみにアクセスを制限し、機密データを保護する必要があります。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 259d4bd5178ca8ab85df0cd03e5268c5a7845a3a
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009442"
---
# <a name="security-patterns"></a><span data-ttu-id="a42c0-106">セキュリティのパターン</span><span class="sxs-lookup"><span data-stu-id="a42c0-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="a42c0-107">セキュリティとは、設計された用途以外の悪意あるアクションや偶発的なアクションを防ぎ、情報の漏えいや損失を防ぐシステムの機能です。</span><span class="sxs-lookup"><span data-stu-id="a42c0-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="a42c0-108">クラウド アプリケーションは、信頼できるオンプレミス境界の外側でインターネット上に存在します。一般に公開されていることも多く、信頼できないユーザーが利用する場合もあります。</span><span class="sxs-lookup"><span data-stu-id="a42c0-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="a42c0-109">アプリケーションの設計とデプロイでは、悪意のある攻撃から保護し、承認したユーザーのみにアクセスを制限し、機密データを保護する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a42c0-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="a42c0-110">Pattern</span><span class="sxs-lookup"><span data-stu-id="a42c0-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="a42c0-111">まとめ</span><span class="sxs-lookup"><span data-stu-id="a42c0-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="a42c0-112">フェデレーション ID</span><span class="sxs-lookup"><span data-stu-id="a42c0-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="a42c0-113">外部の ID プロバイダーに認証を委任します。</span><span class="sxs-lookup"><span data-stu-id="a42c0-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="a42c0-114">ゲートキーパー</span><span class="sxs-lookup"><span data-stu-id="a42c0-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="a42c0-115">専用のホスト インスタンスを使用して、アプリケーションとサービスを保護します。このホスト インスタンスは、クライアントと、アプリケーションまたはサービスとの間でブローカーとして機能し、要求を検証して不適切な部分を除去します。その後、クライアントと、アプリケーションまたはサービスとの間で要求とデータを渡します。</span><span class="sxs-lookup"><span data-stu-id="a42c0-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="a42c0-116">バレット キー</span><span class="sxs-lookup"><span data-stu-id="a42c0-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="a42c0-117">特定のリソースまたはサービスへの限定的な直接アクセスをクライアントに提供する、トークンまたはキーを使用します。</span><span class="sxs-lookup"><span data-stu-id="a42c0-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
