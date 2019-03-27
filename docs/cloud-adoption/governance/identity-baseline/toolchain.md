---
title: CAF:Azure での ID ベースライン ツール
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Azure での ID ベースライン ツール
author: BrianBlanchard
ms.openlocfilehash: 81b0fa9cfee597da98d8b983fb155eac82d97bf8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243673"
---
# <a name="identity-baseline-tools-in-azure"></a>Azure での ID ベースライン ツール

[ID ベースライン](overview.md)は、[5 つのクラウド ガバナンス規範](../governance-disciplines.md)のうちの 1 つです。 この規範では、アプリケーションまたはワークロードをホストするクラウド プロバイダーに関係なく、ユーザー ID の一貫性と継続性を保証するポリシーを確立する方法に焦点を当てています。

ハイブリッド ID の検出ガイドには、次のツールが含まれています。

**Active Directory (オンプレミス):** Active Directory は、ユーザーの資格情報を保存および検証するために企業で使用されることが最も多い ID プロバイダーです。

**Azure Active Directory:** サービスとしてのソフトウェア (SaaS)。Active Directory と同等の機能を持ち、オンプレミスの Active Directory とのフェデレーションが可能です。

**Active Directory (IaaS):** Azure の仮想マシンで実行されている Active Directory アプリケーションのインスタンス。

ID は IT セキュリティのコントロール プレーンです。 したがって認証は、クラウドに対する組織のアクセスの保護です。 組織には、セキュリティを強化し、クラウド アプリを侵入者から保護する ID コントロール プレーンが必要です。

## <a name="cloud-authentication"></a>クラウド認証

正しい認証方法の選択は、クラウドにアプリを移行しようとしている組織にとって最大の関心事です。

この方法を選ぶと、Azure AD がユーザーのサインイン プロセスを処理します。 シームレスなシングル サインオン (SSO) と組み合わせることで、ユーザーは資格情報を再入力しなくてもクラウド アプリにサインインできます。 クラウド認証では、2 つのオプションから選ぶことができます。

**Azure AD のパスワード ハッシュ同期**:Azure AD でオンプレミスのディレクトリ オブジェクトの認証を有効にする最も簡単な方法です。 この方法は、オンプレミスのサーバーがダウンした場合のバックアップ フェールオーバー認証方法として、任意の方法と一緒に使用することもできます。

**Azure AD パススルー認証**:1 つ以上のオンプレミス サーバーで実行されているソフトウェア エージェントを使用して、Azure AD 認証サービスに永続的なパスワード検証を提供します。

> [!NOTE]
> オンプレミスのユーザー アカウントの状態、パスワード ポリシー、およびサインイン時間をすぐに適用するセキュリティ要件のある企業では、パススルー認証方法を検討してください。

**フェデレーション認証:**

この方法を選ぶと、Azure AD は別の信頼された認証システム (オンプレミスの Active Directory フェデレーション サービス (AD FS) や、信頼できるサード パーティのフェデレーション プロバイダーなど) に、ユーザーのパスワードを検証する認証プロセスを引き渡します。

[Azure Active Directory 用の適切な認証方法の選択](/azure/security/azure-ad-choose-authn)に関する記事には、組織に最適なソリューションを選択するためのデシジョン ツリーが含まれています。

次の表は、このガバナンス規範をサポートするポリシーとプロセスを成熟させるのに役立つネイティブ ツールの一覧です。

<!-- markdownlint-disable MD033 -->

|考慮事項|パスワード ハッシュ同期 + シームレス SSO|パススルー認証 + シームレス SSO|AD FS とのフェデレーション|
|:-----|:-----|:-----|:-----|
|認証が行われる場所|クラウド内|クラウド内で、オンプレミスの認証エージェントとのセキュリティで保護されたパスワード検証の交換後|オンプレミス|
|プロビジョニング システム以外のオンプレミスのサーバーの要件: Azure AD Connect|なし|追加の認証エージェントごとに 1 つのサーバー|2 つ以上の AD FS サーバー<br><br>境界/DMZ ネットワークに 2 つ以上の WAP サーバー|
|プロビジョニング システム以外のオンプレミスのインターネットおよびネットワークの要件|なし|認証エージェントを実行しているサーバーからの[発信インターネット アクセス](/azure/active-directory/hybrid/how-to-connect-pta-quick-start)|境界の WAP サーバーへの[着信インターネット アクセス](/windows-server/identity/ad-fs/overview/ad-fs-requirements)<br><br>境界の WAP サーバーから AD FS サーバーへの着信ネットワーク アクセス<br><br>ネットワークの負荷分散|
|SSL 証明書の要件|いいえ |いいえ |はい|
|正常性の監視ソリューション|必要なし|エージェントの状態は [Azure Active Directory 管理センター](/azure/active-directory/hybrid/tshoot-connect-pass-through-authentication)によって提供される|[Azure AD Connect Health](/azure/active-directory/hybrid/how-to-connect-health-adfs)|
|会社のネットワーク内のドメインに参加しているデバイスからクラウドのリソースへのユーザーのシングル サインオン|[シームレス SSO](/azure/active-directory/hybrid/how-to-connect-sso) を使用して実行|[シームレス SSO](/azure/active-directory/hybrid/how-to-connect-sso) を使用して実行|はい|
|サポートされているサインインの種類|UserPrincipalName + パスワード<br><br>[シームレス SSO](/azure/active-directory/hybrid/how-to-connect-sso) を使用した Windows 統合認証<br><br>[代替ログイン ID](/azure/active-directory/hybrid/how-to-connect-install-custom)|UserPrincipalName + パスワード<br><br>[シームレス SSO](/azure/active-directory/hybrid/how-to-connect-sso) を使用した Windows 統合認証<br><br>[代替ログイン ID](/azure/active-directory/hybrid/how-to-connect-pta-faq)|UserPrincipalName + パスワード<br><br>sAMAccountName + パスワード<br><br>Windows 統合認証<br><br>[証明書とスマート カード認証](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[代替ログイン ID](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|Windows Hello for Business のサポート|[キー信頼モデル](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Intune での証明書信頼モデル](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[キー信頼モデル](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Intune での証明書信頼モデル](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[キー信頼モデル](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[証明書信頼モデル](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|多要素認証のオプション|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[条件付きアクセスを使用するカスタム コントロール*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[条件付きアクセスを使用するカスタム コントロール*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Azure MFA](/azure/multi-factor-authentication/)<br><br>[Azure MFA サーバー](/azure/active-directory/authentication/howto-mfaserver-deploy)<br><br>[サード パーティの MFA](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[条件付きアクセスを使用するカスタム コントロール*](/azure/active-directory/conditional-access/controls#custom-controls-1)|
|サポートされるユーザー アカウントの状態|無効なアカウント<br>(最大 30 分の遅延)|無効なアカウント<br><br>アカウントのロックアウト<br><br>アカウント期限切れ<br><br>パスワード期限切れ<br><br>サインイン時間|無効なアカウント<br><br>アカウントのロックアウト<br><br>アカウント期限切れ<br><br>パスワード期限切れ<br><br>サインイン時間|
|条件付きアクセスのオプション|[Azure AD 条件付きアクセス](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Azure AD 条件付きアクセス](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Azure AD 条件付きアクセス](/azure/active-directory/active-directory-conditional-access-azure-portal)<br><br>[AD FS の要求規則](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|サポートされる従来のプロトコルのブロック|[はい](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[はい](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[はい](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|サインイン ページのロゴ、イメージ、説明のカスタマイズ可能性|[Azure AD Premium を使用して可能](/azure/active-directory/customize-branding)|[Azure AD Premium を使用して可能](/azure/active-directory/customize-branding)|[はい](/azure/active-directory/connect/active-directory-aadconnect-federation-management#customlogo)|
|サポートされる高度なシナリオ|[Smart Password Lockout](/azure/active-directory/active-directory-secure-passwords)<br><br>[漏洩した資格情報レポート](/azure/active-directory/active-directory-reporting-risk-events)|[Smart Password Lockout](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-smart-lockout)|複数サイトの低待機時間の認証システム<br><br>[AD FS エクストラネットのロックアウト](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[サード パーティの ID システムとの統合](/azure/active-directory/connect/active-directory-aadconnect-federation-compatibility)|

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> Azure AD の条件付きアクセスでのカスタム コントロールは、現時点ではデバイスの登録をサポートしていません。

## <a name="next-steps"></a>次の手順

[ハイブリッド ID デジタル変換フレームワーク](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?LCID=EN-US)に関する記事では、これらの各コンポーネントを選択および統合するためのさまざまな組み合わせとソリューションについて説明しています。

[Azure AD Connect ツール](https://aka.ms/aadconnectwiz)は、オンプレミスのディレクトリを Azure AD と統合するために役立ちます。
