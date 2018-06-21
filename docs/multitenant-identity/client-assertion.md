---
title: クライアント アサーションを使用した Azure AD からのアクセス トークンの取得
description: クライアント アサーションを使用して Azure AD からのアクセス トークンを取得する方法について説明します。
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540283"
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a><span data-ttu-id="2c426-103">クライアント アサーションを使用した Azure AD からのアクセス トークンの取得</span><span class="sxs-lookup"><span data-stu-id="2c426-103">Use client assertion to get access tokens from Azure AD</span></span>

<span data-ttu-id="2c426-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="2c426-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="background"></a><span data-ttu-id="2c426-105">バックグラウンド</span><span class="sxs-lookup"><span data-stu-id="2c426-105">Background</span></span>
<span data-ttu-id="2c426-106">OpenID Connect で承認コード フローまたはハイブリッド フローを使用すると、クライアントはアクセス トークンの承認コードを交換します。</span><span class="sxs-lookup"><span data-stu-id="2c426-106">When using authorization code flow or hybrid flow in OpenID Connect, the client exchanges an authorization code for an access token.</span></span> <span data-ttu-id="2c426-107">クライアントは、この手順中にサーバーに対して自身を認証する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2c426-107">During this step, the client has to authenticate itself to the server.</span></span>

![クライアント シークレット](./images/client-secret.png)

<span data-ttu-id="2c426-109">クライアントを認証する方法の 1 つは、クライアント シークレットの使用です。</span><span class="sxs-lookup"><span data-stu-id="2c426-109">One way to authenticate the client is by using a client secret.</span></span> <span data-ttu-id="2c426-110">[Tailspin Surveys][Surveys] アプリケーションは、クライアント シークレットを使用するように既定で構成されます。</span><span class="sxs-lookup"><span data-stu-id="2c426-110">That's how the [Tailspin Surveys][Surveys] application is configured by default.</span></span>

<span data-ttu-id="2c426-111">次に、クライアントから IDP に対して、アクセス トークンを要求する要求の例を示します。</span><span class="sxs-lookup"><span data-stu-id="2c426-111">Here is an example request from the client to the IDP, requesting an access token.</span></span> <span data-ttu-id="2c426-112">`client_secret` パラメーターに注目してください。</span><span class="sxs-lookup"><span data-stu-id="2c426-112">Note the `client_secret` parameter.</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

<span data-ttu-id="2c426-113">シークレットは単なる文字列なので、値が漏えいしないように注意する必要があります。</span><span class="sxs-lookup"><span data-stu-id="2c426-113">The secret is just a string, so you have to make sure not to leak the value.</span></span> <span data-ttu-id="2c426-114">ベスト プラクティスは、クライアント シークレットをソース制御とは別に保存することです。</span><span class="sxs-lookup"><span data-stu-id="2c426-114">The best practice is to keep the client secret out of source control.</span></span> <span data-ttu-id="2c426-115">Azure にデプロイするときは、[アプリ設定][configure-web-app]にシークレットを保存します。</span><span class="sxs-lookup"><span data-stu-id="2c426-115">When you deploy to Azure, store the secret in an [app setting][configure-web-app].</span></span>

<span data-ttu-id="2c426-116">ただし、Azure サブスクリプションにアクセス権を持つ全員がアプリ設定を表示できます。</span><span class="sxs-lookup"><span data-stu-id="2c426-116">However, anyone with access to the Azure subscription can view the app settings.</span></span> <span data-ttu-id="2c426-117">さらに、シークレットをソース制御 (たとえばデプロイ スクリプトなど) に組み入れたり、電子メールで共有したりするなどの誘惑は常にあります。</span><span class="sxs-lookup"><span data-stu-id="2c426-117">Further, there is always a temptation to check secrets into source control (e.g., in deployment scripts), share them by email, and so on.</span></span>

<span data-ttu-id="2c426-118">セキュリティを強化するために、クライアント シークレットではなく、[クライアント アサーション]をご利用いただけます。</span><span class="sxs-lookup"><span data-stu-id="2c426-118">For additional security, you can use [client assertion] instead of a client secret.</span></span> <span data-ttu-id="2c426-119">クライアント アサーションの場合、クライアントは X.509 証明書を使用して、クライアントから送信されたトークン要求を証明します。</span><span class="sxs-lookup"><span data-stu-id="2c426-119">With client assertion, the client uses an X.509 certificate to prove the token request came from the client.</span></span> <span data-ttu-id="2c426-120">クライアント証明書は Web サーバーにインストールされています。</span><span class="sxs-lookup"><span data-stu-id="2c426-120">The client certificate is installed on the web server.</span></span> <span data-ttu-id="2c426-121">一般的に、クライアント シークレットが誤って暴露されないようにするよりも、証明書に対するアクセス権を制限する方が簡単です。</span><span class="sxs-lookup"><span data-stu-id="2c426-121">Generally, it will be easier to restrict access to the certificate, than to ensure that nobody inadvertently reveals a client secret.</span></span> <span data-ttu-id="2c426-122">Web アプリで証明書を構成する方法の詳細については、「[Using Certificates in Azure Websites Applications (Azure Websites アプリケーションでの証明書の使用)][using-certs-in-websites]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2c426-122">For more information about configuring certificates in a web app, see [Using Certificates in Azure Websites Applications][using-certs-in-websites]</span></span>

<span data-ttu-id="2c426-123">クライアント アサーションを使用したトークン要求を次に示します。</span><span class="sxs-lookup"><span data-stu-id="2c426-123">Here is a token request using client assertion:</span></span>

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

<span data-ttu-id="2c426-124">`client_secret` パラメーターは使用されなくなったことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="2c426-124">Notice that the `client_secret` parameter is no longer used.</span></span> <span data-ttu-id="2c426-125">代わりに、クライアント証明書を使用して署名された JWT トークンは `client_assertion` パラメーターに含まれます。</span><span class="sxs-lookup"><span data-stu-id="2c426-125">Instead, the `client_assertion` parameter contains a JWT token that was signed using the client certificate.</span></span> <span data-ttu-id="2c426-126">`client_assertion_type` パラメーターでは、アサーションの種類 (この例では JWT トークン) を指定します。</span><span class="sxs-lookup"><span data-stu-id="2c426-126">The `client_assertion_type` parameter specifies the type of assertion &mdash; in this case, JWT token.</span></span> <span data-ttu-id="2c426-127">サーバーが JWT トークンを検証します。</span><span class="sxs-lookup"><span data-stu-id="2c426-127">The server validates the JWT token.</span></span> <span data-ttu-id="2c426-128">JWT トークンが無効の場合、トークン要求からエラーが返されます。</span><span class="sxs-lookup"><span data-stu-id="2c426-128">If the JWT token is invalid, the token request returns an error.</span></span>

> [!NOTE]
> <span data-ttu-id="2c426-129">X.509 証明書は、クライアント アサーションの唯一の形式ではありませんが、Azure AD でサポートされているため、この記事では取り上げています。</span><span class="sxs-lookup"><span data-stu-id="2c426-129">X.509 certificates are not the only form of client assertion; we focus on it here because it is supported by Azure AD.</span></span>
> 
> 

<span data-ttu-id="2c426-130">実行時に、Web アプリケーションは証明書ストアから証明書を読み取ります。</span><span class="sxs-lookup"><span data-stu-id="2c426-130">At run time, the web application reads the certificate from the certificate store.</span></span> <span data-ttu-id="2c426-131">証明書は Web アプリと同じコンピューターにインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="2c426-131">The certificate must be installed on the same machine as the web app.</span></span>

<span data-ttu-id="2c426-132">Surveys アプリケーションには、Azure AD からトークンを取得する [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) メソッドに渡すことができる、[ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) を作成するヘルパー クラスが含まれています。</span><span class="sxs-lookup"><span data-stu-id="2c426-132">The Surveys application includes a helper class that creates a [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate) that you can pass to the [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) method to acquire a token from Azure AD.</span></span>

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

<span data-ttu-id="2c426-133">Surveys アプリケーションでのクライアント アサーションの設定については、「[Use Azure Key Vault to protect application secrets (Azure Key Vault を使用したアプリケーション シークレットの保護)][key vault]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="2c426-133">For information about setting up client assertion in the Surveys application, see [Use Azure Key Vault to protect application secrets ][key vault].</span></span>

<span data-ttu-id="2c426-134">[**次へ**][key vault]</span><span class="sxs-lookup"><span data-stu-id="2c426-134">[**Next**][key vault]</span></span>

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[クライアント アサーション]: https://tools.ietf.org/html/rfc7521
[client assertion]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
