---
title: オンプレミスの Active Directory と Azure の統合
titleSuffix: Azure Reference Architectures
description: オンプレミスの Active Directory を Azure と統合するための参照アーキテクチャを比較します。
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 706cb63a65ce521e72ebc41a997dc900afacaab9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343970"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>オンプレミスの Active Directory を Azure と統合するためのソリューションの選択

この記事では、オンプレミスの Active Directory (AD) 環境を Azure ネットワークと統合するためのオプションを比較します。 各オプションには、さらに詳細な参照アーキテクチャが用意されています。

多くの組織では、ユーザー、コンピューター、アプリケーション、またはセキュリティ境界内に含まれているその他のリソースに関連付けられた ID を認証するために、Active Directory Domain Services (AD DS) が使用されています。 通常、ディレクトリ サービスと ID サービスはオンプレミスでホストされますが、アプリケーションの一部をオンプレミスでホストし、一部を Azure でホストしている場合は、Azure からの認証要求をオンプレミスに送リ返す際に待機時間が発生することがあります。 この待機時間は、ディレクトリサービスと ID サービスを Azure 内で実装することにより減らすことができます。

Azure では、ディレクトリ サービスと ID サービスを Azure 内で実装する方法として、次の 2 つのソリューションを提供しています。

- [Azure AD][azure-active-directory] を使用してクラウド上に Active Directory ドメインを作成し、それをオンプレミスの Active Directory ドメインに接続する。 オンプレミスのディレクトリは、[Azure AD Connect][azure-ad-connect] によって Azure AD と統合されます。

- AD DS をドメイン コントローラーとして実行する VM を Azure 内にデプロイすることで、既存のオンプレミス Active Directory インフラストラクチャを Azure に拡張する。 このアーキテクチャは、オンプレミス ネットワークと Azure 仮想ネットワーク (VNet) が VPN または ExpressRoute 接続を使用して接続されている場合によく使用されます。 このアーキテクチャは、いくつかのバリエーションで使用できます。

  - Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させる。
  - オンプレミス フォレスト内のドメインによって信頼された個別のフォレストを、Azure 内に作成する。
  - Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートする。

以下の各セクションでは、これらのオプションについて詳しく説明します。

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>オンプレミス ドメインを Azure AD と統合する

Azure Active Directory (Azure AD) を使用して Azure 内にドメインを作成し、それをオンプレミスの AD ドメインにリンクします。

Azure AD ディレクトリは、オンプレミス ディレクトリの拡張機能ではありません。 同じオブジェクトと ID を含んだコピーです。 オンプレミスでこれらのアイテムに変更が加えられた場合、それらの変更は Azure AD にコピーされますが、Azure AD で加えられた変更はオンプレミス ドメインにはレプリケートされません。

オンプレミス ディレクトリを使用せずに Azure AD を使用することもできます。 その場合、Azure AD はオンプレミス ディレクトリからレプリケートされたデータを格納するのではなく、すべての ID 情報のプライマリ ソースとして機能します。

**メリット**

- クラウド上の AD インフラストラクチャを保守維持する必要がありません。 Azure AD の管理と保守は、すべて Microsoft によって行われます。
- Azure AD では、オンプレミスで使用できるのと同じ ID 情報が提供されます。
- 認証は Azure 内で実行されるので、外部のアプリケーションやユーザーがオンプレミス ドメインに接続する必要性が減ります。

**課題**

- ID サービスはユーザーとグループに制限されます。 サービス アカウントやコンピューター アカウントを認証することはできません。
- Azure AD ディレクトリの同期を維持するために、オンプレミス ドメインとの接続を構成する必要があります。
- Azure AD 経由の認証を可能にするために、アプリケーション コードの再記述が必要になる場合があります。

**参照アーキテクチャ**

- [オンプレミスの Active Directory ドメインと Azure Active Directory を統合する][aad]

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>Azure 内の AD DS をオンプレミス フォレストに参加させる

AD ドメイン サービス (AD DS) サーバーを Azure にデプロイします。 Azure 内にドメインを作成し、それをオンプレミスの AD フォレストに参加させます。

Azure AD で現在実装されていない AD DS 機能を使用する必要がある場合には、このオプションを検討してください。

**メリット**

- オンプレミスで使用できるのと同じ ID 情報にアクセスできます。
- ユーザー、サービス、およびコンピューター アカウントを、オンプレミスと Azure で認証できます。
- 個別の AD フォレストを管理する必要がありません。 Azure 内のドメインは、オンプレミス フォレストに属することができます。
- オンプレミスのグループ ポリシー オブジェクトによって定義されたグループ ポリシーを、Azure 内のドメインに適用できます。

**課題**

- 独自の AD DS サーバーとドメインをクラウド上にデプロイし、管理する必要があります。
- クラウド上のドメイン サーバーとオンプレミスで実行しているサーバーの間に、一定の同期待機時間が発生する可能性があります。

**参照アーキテクチャ**

- [Active Directory Domain Services (AD DS) を Azure に拡張する][ad-ds]

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>個別のフォレストを使用して AD DS を Azure にデプロイする

AD ドメイン サービス (AD DS) サーバーを Azure にデプロイしますが、オンプレミス フォレストとは分離された、個別の Active Directory [フォレスト][ad-forest-defn]を作成します。 このフォレストは、オンプレミス フォレスト内のドメインによって信頼されます。

このアーキテクチャは、クラウドで保持されているオブジェクトと ID のセキュリティ分離を維持しつつ、個々 のドメインをオンプレミスからクラウドに移行する場合によく使用されます。

**メリット**

- オンプレミスの ID と、分離さた Azure 専用の ID を実装できます。
- オンプレミスの AD フォレストから Azure へのレプリケートを行う必要がありません。

**課題**

- オンプレミス ID を Azure 内で認証するために、オンプレミス AD サーバーへの余分なネットワーク ホップが必要になります。
- 独自の AD DS サーバーとフォレストをクラウド上にデプロイし、フォレスト間の適切な信頼関係を確立する必要があります。

**参照アーキテクチャ**

- [Azure での Active Directory Domain Services (AD DS) リソース フォレストの作成][ad-ds-forest]

## <a name="extend-ad-fs-to-azure"></a>AD FS を Azure に拡張する

Azure で実行されているコンポーネントのフェデレーション認証と承認を実行するために、Active Directory フェデレーション サービス (AD FS) のデプロイを Azure にレプリケートします。

このアーキテクチャは通常、次の目的のために使用されます。

- パートナー組織のユーザーを認証および承認する。
- 組織のファイアウォールの外部で実行されている Web ブラウザーからユーザーを認証できるようにする。
- ユーザーが、承認済みの外部デバイス (モバイル デバイスなど) から接続できるようにする。

**メリット**

- 要求に対応したアプリケーションを利用できます。
- 外部のパートナーを認証用に信頼することができます。
- 多数の認証プロトコルとの互換性を確保できます。

**課題**

- 独自の AD DS、AD FS、および AD FS Web アプリケーション プロキシ サーバーを、Azure にデプロイする必要があります。
- このアーキテクチャは、構成が複雑になる場合があります。

**参照アーキテクチャ**

- [Active Directory フェデレーション サービス (AD FS) を Azure に拡張する][adfs]

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
