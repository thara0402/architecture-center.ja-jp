---
title: オンプレミスの Active Directory を Azure と統合するためのソリューションの選択。
description: オンプレミスの Active Directory を Azure と統合するためのリファレンス アーキテクチャを比較します。
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541659"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="27f35-103">オンプレミスの Active Directory を Azure と統合するためのソリューションの選択</span><span class="sxs-lookup"><span data-stu-id="27f35-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="27f35-104">この記事では、オンプレミスの Active Directory (AD) 環境を Azure ネットワークと統合するためのオプションを比較します。</span><span class="sxs-lookup"><span data-stu-id="27f35-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="27f35-105">各オプションについては、リファレンス アーキテクチャと、デプロイ可能なソリューションを示します。</span><span class="sxs-lookup"><span data-stu-id="27f35-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="27f35-106">多くの組織では、ユーザー、コンピューター、アプリケーション、またはセキュリティ境界内に含まれているその他のリソースに関連付けられた ID を認証するために、[Active Directory Domain Services (AD DS)][active-directory-domain-services] が使用されています。</span><span class="sxs-lookup"><span data-stu-id="27f35-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="27f35-107">通常、ディレクトリ サービスと ID サービスはオンプレミスでホストされますが、アプリケーションの一部をオンプレミスでホストし、一部を Azure でホストしている場合は、Azure からの認証要求をオンプレミスに送リ返す際に待機時間が発生することがあります。</span><span class="sxs-lookup"><span data-stu-id="27f35-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="27f35-108">この待機時間は、ディレクトリサービスと ID サービスを Azure 内で実装することにより減らすことができます。</span><span class="sxs-lookup"><span data-stu-id="27f35-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="27f35-109">Azure では、ディレクトリ サービスと ID サービスを Azure 内で実装する方法として、次の 2 つのソリューションを提供しています。</span><span class="sxs-lookup"><span data-stu-id="27f35-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="27f35-110">[Azure AD][azure-active-directory] を使用してクラウド上に Active Directory ドメインを作成し、それをオンプレミスの Active Directory ドメインに接続する。</span><span class="sxs-lookup"><span data-stu-id="27f35-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="27f35-111">オンプレミスのディレクトリは、[Azure AD Connect][azure-ad-connect] によって Azure AD と統合されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="27f35-112">AD DS をドメイン コントローラーとして実行する VM を Azure 内にデプロイすることで、既存のオンプレミス Active Directory インフラストラクチャを Azure に拡張する。</span><span class="sxs-lookup"><span data-stu-id="27f35-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="27f35-113">このアーキテクチャは、オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) が VPN または ExpressRoute 接続を使用して接続されている場合によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="27f35-114">このアーキテクチャは、いくつかのバリエーションで使用できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="27f35-115">Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させる。</span><span class="sxs-lookup"><span data-stu-id="27f35-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="27f35-116">オンプレミス フォレスト内のドメインによって信頼された個別のフォレストを、Azure 内に作成する。</span><span class="sxs-lookup"><span data-stu-id="27f35-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="27f35-117">Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートする。</span><span class="sxs-lookup"><span data-stu-id="27f35-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="27f35-118">以下の各セクションでは、これらのオプションについて詳しく説明します。</span><span class="sxs-lookup"><span data-stu-id="27f35-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="27f35-119">オンプレミス ドメインを Azure AD と統合する</span><span class="sxs-lookup"><span data-stu-id="27f35-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="27f35-120">Azure Active Directory (Azure AD) を使用して Azure 内にドメインを作成し、それをオンプレミスの AD ドメインにリンクします。</span><span class="sxs-lookup"><span data-stu-id="27f35-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="27f35-121">Azure AD ディレクトリは、オンプレミス ディレクトリの拡張機能ではありません。</span><span class="sxs-lookup"><span data-stu-id="27f35-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="27f35-122">同じオブジェクトと ID を含んだコピーです。</span><span class="sxs-lookup"><span data-stu-id="27f35-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="27f35-123">オンプレミスでこれらのアイテムに変更が加えられた場合、それらの変更は Azure AD にコピーされますが、Azure AD で加えられた変更はオンプレミス ドメインにはレプリケートされません。</span><span class="sxs-lookup"><span data-stu-id="27f35-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="27f35-124">オンプレミス ディレクトリを使用せずに Azure AD を使用することもできます。</span><span class="sxs-lookup"><span data-stu-id="27f35-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="27f35-125">その場合、Azure AD はオンプレミス ディレクトリからレプリケートされたデータを格納するのではなく、すべての ID 情報のプライマリ ソースとして機能します。</span><span class="sxs-lookup"><span data-stu-id="27f35-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="27f35-126">**メリット**</span><span class="sxs-lookup"><span data-stu-id="27f35-126">**Benefits**</span></span>

* <span data-ttu-id="27f35-127">クラウド上の AD インフラストラクチャを保守維持する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="27f35-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="27f35-128">Azure AD の管理と保守は、すべて Microsoft によって行われます。</span><span class="sxs-lookup"><span data-stu-id="27f35-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="27f35-129">Azure AD では、オンプレミスで使用できるのと同じ ID 情報が提供されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="27f35-130">認証は Azure 内で実行されるので、外部のアプリケーションやユーザーがオンプレミス ドメインに接続する必要性が減ります。</span><span class="sxs-lookup"><span data-stu-id="27f35-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="27f35-131">**課題**</span><span class="sxs-lookup"><span data-stu-id="27f35-131">**Challenges**</span></span>

* <span data-ttu-id="27f35-132">ID サービスはユーザーとグループに制限されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="27f35-133">サービス アカウントやコンピューター アカウントを認証することはできません。</span><span class="sxs-lookup"><span data-stu-id="27f35-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="27f35-134">Azure AD ディレクトリの同期を維持するために、オンプレミス ドメインとの接続を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="27f35-135">Azure AD 経由の認証を可能にするために、アプリケーション コードの再記述が必要になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="27f35-136">**[詳細を読む...][aad]**</span><span class="sxs-lookup"><span data-stu-id="27f35-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="27f35-137">Azure 内の AD DS をオンプレミス フォレストに参加させる</span><span class="sxs-lookup"><span data-stu-id="27f35-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="27f35-138">AD ドメイン サービス (AD DS) サーバーを Azure にデプロイします。</span><span class="sxs-lookup"><span data-stu-id="27f35-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="27f35-139">Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させます。</span><span class="sxs-lookup"><span data-stu-id="27f35-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="27f35-140">Azure AD で現在実装されていない AD DS 機能を使用する必要がある場合には、このオプションを検討してください。</span><span class="sxs-lookup"><span data-stu-id="27f35-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="27f35-141">**メリット**</span><span class="sxs-lookup"><span data-stu-id="27f35-141">**Benefits**</span></span>

* <span data-ttu-id="27f35-142">オンプレミスで使用できるのと同じ ID 情報にアクセスできます。</span><span class="sxs-lookup"><span data-stu-id="27f35-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="27f35-143">ユーザー、サービス、およびコンピューター アカウントを、オンプレミスと Azure で認証できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="27f35-144">個別の AD フォレストを管理する必要がありません。</span><span class="sxs-lookup"><span data-stu-id="27f35-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="27f35-145">Azure 内のドメインは、オンプレミス フォレストに属することができます。</span><span class="sxs-lookup"><span data-stu-id="27f35-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="27f35-146">オンプレミスのグループ ポリシー オブジェクトによって定義されたグループ ポリシーを、Azure 内のドメインに適用できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="27f35-147">**課題**</span><span class="sxs-lookup"><span data-stu-id="27f35-147">**Challenges**</span></span>

* <span data-ttu-id="27f35-148">独自の AD DS サーバーとドメインをクラウド上にデプロイし、管理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="27f35-149">クラウド上のドメイン サーバーとオンプレミスで実行しているサーバーの間に、一定の同期待機時間が発生する可能性があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="27f35-150">**[詳細を読む...][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="27f35-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="27f35-151">個別のフォレストを使用して AD DS を Azure にデプロイする</span><span class="sxs-lookup"><span data-stu-id="27f35-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="27f35-152">AD ドメイン サービス (AD DS) サーバーを Azure にデプロイしますが、オンプレミス フォレストとは分離された、個別の Active Directory [フォレスト][ad-forest-defn]を作成します。</span><span class="sxs-lookup"><span data-stu-id="27f35-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="27f35-153">このフォレストは、オンプレミス フォレスト内のドメインによって信頼されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="27f35-154">このアーキテクチャは、クラウドで保持されているオブジェクトと ID のセキュリティ分離を維持しつつ、個々 のドメインをオンプレミスからクラウドに移行する場合によく使用されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="27f35-155">**メリット**</span><span class="sxs-lookup"><span data-stu-id="27f35-155">**Benefits**</span></span>

* <span data-ttu-id="27f35-156">オンプレミスの ID と、分離さた Azure 専用の ID を実装できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="27f35-157">オンプレミスの AD フォレストから Azure へのレプリケートを行う必要がありません。</span><span class="sxs-lookup"><span data-stu-id="27f35-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="27f35-158">**課題**</span><span class="sxs-lookup"><span data-stu-id="27f35-158">**Challenges**</span></span>

* <span data-ttu-id="27f35-159">オンプレミス ID を Azure 内で認証するために、オンプレミス AD サーバーへの余分なネットワーク ホップが必要になります。</span><span class="sxs-lookup"><span data-stu-id="27f35-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="27f35-160">独自の AD DS サーバーとフォレストをクラウド上にデプロイし、フォレスト間の適切な信頼関係を確立する必要があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="27f35-161">**[詳細を読む...][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="27f35-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="27f35-162">AD FS を Azure に拡張する</span><span class="sxs-lookup"><span data-stu-id="27f35-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="27f35-163">Azure で実行されているコンポーネントのフェデレーション認証と承認を実行するために、Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートします。</span><span class="sxs-lookup"><span data-stu-id="27f35-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="27f35-164">このアーキテクチャは通常、次の目的のために使用されます。</span><span class="sxs-lookup"><span data-stu-id="27f35-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="27f35-165">パートナー組織のユーザーを認証および承認する。</span><span class="sxs-lookup"><span data-stu-id="27f35-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="27f35-166">組織のファイアウォールの外部で実行されている Web ブラウザーからユーザーを認証できるようにする。</span><span class="sxs-lookup"><span data-stu-id="27f35-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="27f35-167">ユーザーが、承認済みの外部デバイス (モバイル デバイスなど) から接続できるようにする。</span><span class="sxs-lookup"><span data-stu-id="27f35-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="27f35-168">**メリット**</span><span class="sxs-lookup"><span data-stu-id="27f35-168">**Benefits**</span></span>

* <span data-ttu-id="27f35-169">要求に対応したアプリケーションを利用できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="27f35-170">外部のパートナーを認証用に信頼することができます。</span><span class="sxs-lookup"><span data-stu-id="27f35-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="27f35-171">多数の認証プロトコルとの互換性を確保できます。</span><span class="sxs-lookup"><span data-stu-id="27f35-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="27f35-172">**課題**</span><span class="sxs-lookup"><span data-stu-id="27f35-172">**Challenges**</span></span>

* <span data-ttu-id="27f35-173">独自の AD DS、AD FS、および AD FS Web アプリケーション プロキシ サーバーを、Azure にデプロイする必要があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="27f35-174">このアーキテクチャは、構成が複雑になる場合があります。</span><span class="sxs-lookup"><span data-stu-id="27f35-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="27f35-175">**[詳細を読む...][adfs]**</span><span class="sxs-lookup"><span data-stu-id="27f35-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
