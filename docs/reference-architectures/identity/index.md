---
title: オンプレミスの Active Directory と Azure の統合
titleSuffix: Azure Reference Architectures
description: オンプレミスの Active Directory を Azure と統合するための参照アーキテクチャを比較します。
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 905dedda6de1a107f55b2f7651441780a685aea7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119865"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="d8ad4-103">オンプレミスの Active Directory を Azure と統合するためのソリューションの選択</span><span class="sxs-lookup"><span data-stu-id="d8ad4-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="d8ad4-104">この記事では、オンプレミスの Active Directory (AD) 環境を Azure ネットワークと統合するためのオプションを比較します。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="d8ad4-105">各オプションには、さらに詳細な参照アーキテクチャが用意されています。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="d8ad4-106">多くの組織では、ユーザー、コンピューター、アプリケーション、またはセキュリティ境界内に含まれているその他のリソースに関連付けられた ID を認証するために、Active Directory Domain Services (AD DS) が使用されています。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="d8ad4-107">通常、ディレクトリ サービスと ID サービスはオンプレミスでホストされますが、アプリケーションの一部をオンプレミスでホストし、一部を Azure でホストしている場合は、Azure からの認証要求をオンプレミスに送リ返す際に待機時間が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="d8ad4-108">この待機時間は、ディレクトリサービスと ID サービスを Azure 内で実装することにより減らすことができます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="d8ad4-109">Azure では、ディレクトリ サービスと ID サービスを Azure 内で実装する方法として、次の 2 つのソリューションを提供しています。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="d8ad4-110">[Azure AD][azure-active-directory] を使用してクラウド上に Active Directory ドメインを作成し、それをオンプレミスの Active Directory ドメインに接続する。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="d8ad4-111">オンプレミスのディレクトリは、[Azure AD Connect][azure-ad-connect] によって Azure AD と統合されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="d8ad4-112">AD DS をドメイン コントローラーとして実行する VM を Azure 内にデプロイすることで、既存のオンプレミス Active Directory インフラストラクチャを Azure に拡張する。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="d8ad4-113">このアーキテクチャは、オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) が VPN または ExpressRoute 接続を使用して接続されている場合によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="d8ad4-114">このアーキテクチャは、いくつかのバリエーションで使用できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="d8ad4-115">Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させる。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="d8ad4-116">オンプレミス フォレスト内のドメインによって信頼された個別のフォレストを、Azure 内に作成する。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="d8ad4-117">Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートする。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="d8ad4-118">以下の各セクションでは、これらのオプションについて詳しく説明します。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="d8ad4-119">オンプレミス ドメインを Azure AD と統合する</span><span class="sxs-lookup"><span data-stu-id="d8ad4-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="d8ad4-120">Azure Active Directory (Azure AD) を使用して Azure 内にドメインを作成し、それをオンプレミスの AD ドメインにリンクします。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="d8ad4-121">Azure AD ディレクトリは、オンプレミス ディレクトリの拡張機能ではありません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="d8ad4-122">同じオブジェクトと ID を含んだコピーです。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="d8ad4-123">オンプレミスでこれらのアイテムに変更が加えられた場合、それらの変更は Azure AD にコピーされますが、Azure AD で加えられた変更はオンプレミス ドメインにはレプリケートされません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="d8ad4-124">オンプレミス ディレクトリを使用せずに Azure AD を使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="d8ad4-125">その場合、Azure AD はオンプレミス ディレクトリからレプリケートされたデータを格納するのではなく、すべての ID 情報のプライマリ ソースとして機能します。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="d8ad4-126">**メリット**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-126">**Benefits**</span></span>

- <span data-ttu-id="d8ad4-127">クラウド上の AD インフラストラクチャを保守維持する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="d8ad4-128">Azure AD の管理と保守は、すべて Microsoft によって行われます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="d8ad4-129">Azure AD では、オンプレミスで使用できるのと同じ ID 情報が提供されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="d8ad4-130">認証は Azure 内で実行されるので、外部のアプリケーションやユーザーがオンプレミス ドメインに接続する必要性が減ります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="d8ad4-131">**課題**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-131">**Challenges**</span></span>

- <span data-ttu-id="d8ad4-132">ID サービスはユーザーとグループに制限されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="d8ad4-133">サービス アカウントやコンピューター アカウントを認証することはできません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="d8ad4-134">Azure AD ディレクトリの同期を維持するために、オンプレミス ドメインとの接続を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
- <span data-ttu-id="d8ad4-135">Azure AD 経由の認証を可能にするために、アプリケーション コードの再記述が必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="d8ad4-136">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-136">**Reference architecture**</span></span>

- <span data-ttu-id="d8ad4-137">[オンプレミスの Active Directory ドメインと Azure Active Directory を統合する][aad]</span><span class="sxs-lookup"><span data-stu-id="d8ad4-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="d8ad4-138">Azure 内の AD DS をオンプレミス フォレストに参加させる</span><span class="sxs-lookup"><span data-stu-id="d8ad4-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="d8ad4-139">AD ドメイン サービス (AD DS) サーバーを Azure にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="d8ad4-140">Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="d8ad4-141">Azure AD で現在実装されていない AD DS 機能を使用する必要がある場合には、このオプションを検討してください。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="d8ad4-142">**メリット**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-142">**Benefits**</span></span>

- <span data-ttu-id="d8ad4-143">オンプレミスで使用できるのと同じ ID 情報にアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="d8ad4-144">ユーザー、サービス、およびコンピューター アカウントを、オンプレミスと Azure で認証できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="d8ad4-145">個別の AD フォレストを管理する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="d8ad4-146">Azure 内のドメインは、オンプレミス フォレストに属することができます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="d8ad4-147">オンプレミスのグループ ポリシー オブジェクトによって定義されたグループ ポリシーを、Azure 内のドメインに適用できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="d8ad4-148">**課題**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-148">**Challenges**</span></span>

- <span data-ttu-id="d8ad4-149">独自の AD DS サーバーとドメインをクラウド上にデプロイし、管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="d8ad4-150">クラウド上のドメイン サーバーとオンプレミスで実行しているサーバーの間に、一定の同期待機時間が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="d8ad4-151">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-151">**Reference architecture**</span></span>

- <span data-ttu-id="d8ad4-152">[Active Directory Domain Services (AD DS) を Azure に拡張する][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="d8ad4-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="d8ad4-153">個別のフォレストを使用して AD DS を Azure にデプロイする</span><span class="sxs-lookup"><span data-stu-id="d8ad4-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="d8ad4-154">AD ドメイン サービス (AD DS) サーバーを Azure にデプロイしますが、オンプレミス フォレストとは分離された、個別の Active Directory [フォレスト][ad-forest-defn]を作成します。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="d8ad4-155">このフォレストは、オンプレミス フォレスト内のドメインによって信頼されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="d8ad4-156">このアーキテクチャは、クラウドで保持されているオブジェクトと ID のセキュリティ分離を維持しつつ、個々 のドメインをオンプレミスからクラウドに移行する場合によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="d8ad4-157">**メリット**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-157">**Benefits**</span></span>

- <span data-ttu-id="d8ad4-158">オンプレミスの ID と、分離さた Azure 専用の ID を実装できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="d8ad4-159">オンプレミスの AD フォレストから Azure へのレプリケートを行う必要がありません。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="d8ad4-160">**課題**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-160">**Challenges**</span></span>

- <span data-ttu-id="d8ad4-161">オンプレミス ID を Azure 内で認証するために、オンプレミス AD サーバーへの余分なネットワーク ホップが必要になります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="d8ad4-162">独自の AD DS サーバーとフォレストをクラウド上にデプロイし、フォレスト間の適切な信頼関係を確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="d8ad4-163">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-163">**Reference architecture**</span></span>

- <span data-ttu-id="d8ad4-164">[Azure での Active Directory Domain Services (AD DS) リソース フォレストの作成][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="d8ad4-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="d8ad4-165">AD FS を Azure に拡張する</span><span class="sxs-lookup"><span data-stu-id="d8ad4-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="d8ad4-166">Azure で実行されているコンポーネントのフェデレーション認証と承認を実行するために、Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートします。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="d8ad4-167">このアーキテクチャは通常、次の目的のために使用されます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="d8ad4-168">パートナー組織のユーザーを認証および承認する。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="d8ad4-169">組織のファイアウォールの外部で実行されている Web ブラウザーからユーザーを認証できるようにする。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="d8ad4-170">ユーザーが、承認済みの外部デバイス (モバイル デバイスなど) から接続できるようにする。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-170">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="d8ad4-171">**メリット**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-171">**Benefits**</span></span>

- <span data-ttu-id="d8ad4-172">要求に対応したアプリケーションを利用できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="d8ad4-173">外部のパートナーを認証用に信頼することができます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="d8ad4-174">多数の認証プロトコルとの互換性を確保できます。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="d8ad4-175">**課題**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-175">**Challenges**</span></span>

- <span data-ttu-id="d8ad4-176">独自の AD DS、AD FS、および AD FS Web アプリケーション プロキシ サーバーを、Azure にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="d8ad4-177">このアーキテクチャは、構成が複雑になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="d8ad4-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="d8ad4-178">**参照アーキテクチャ**</span><span class="sxs-lookup"><span data-stu-id="d8ad4-178">**Reference architecture**</span></span>

- <span data-ttu-id="d8ad4-179">[Active Directory フェデレーション サービス (AD FS) を Azure に拡張する][adfs]</span><span class="sxs-lookup"><span data-stu-id="d8ad4-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
