---
title: 静的コンテンツ ホスティング パターン
titleSuffix: Cloud Design Patterns
description: 静的コンテンツを、クライアントに直接配信できるクラウド ベースのストレージ サービスにデプロイします。
keywords: 設計パターン
author: dragon119
ms.date: 01/04/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 719f0221ecc8d52267cba3136eec20dadef30b99
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483447"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="37a05-104">静的コンテンツ ホスティング パターン</span><span class="sxs-lookup"><span data-stu-id="37a05-104">Static Content Hosting pattern</span></span>

<span data-ttu-id="37a05-105">静的コンテンツを、クライアントに直接配信できるクラウド ベースのストレージ サービスにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="37a05-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="37a05-106">これにより、高額なコンピューティング インスタンスの必要性を低減できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="37a05-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="37a05-107">Context and problem</span></span>

<span data-ttu-id="37a05-108">通常、Web アプリケーションには静的コンテンツの要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="37a05-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="37a05-109">この静的コンテンツには、HTML ページのほか、クライアントから利用可能な画像やドキュメントが、HTML ページの一部 (インライン画像、スタイル シート、クライアント側 JavaScript ファイルなど) か、個別のダウンロード (PDF ドキュメントなど) として含まれています。</span><span class="sxs-lookup"><span data-stu-id="37a05-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="37a05-110">Web サーバーは動的レンダリングと出力キャッシュ用に最適化されていますが、静的コンテンツをダウンロードする要求を引き続き処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-110">Although web servers are optimized for dynamic rendering and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="37a05-111">このために消費される処理サイクルは、他の目的のために、より効果的に使用できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="37a05-112">解決策</span><span class="sxs-lookup"><span data-stu-id="37a05-112">Solution</span></span>

<span data-ttu-id="37a05-113">ほとんどのクラウド ホスティング環境では、いくつかのアプリケーションのリソースと静的なページをストレージ サービスに配置することができます。</span><span class="sxs-lookup"><span data-stu-id="37a05-113">In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service.</span></span> <span data-ttu-id="37a05-114">ストレージ サービスではこれらのリソースの要求を処理することができ、他の Web 要求を処理するコンピューティング リソースの負荷が軽減されます。</span><span class="sxs-lookup"><span data-stu-id="37a05-114">The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests.</span></span> <span data-ttu-id="37a05-115">クラウドでホストされるストレージのコストは、通常、多くのコンピューティング インスタンスのコストよりも安価です。</span><span class="sxs-lookup"><span data-stu-id="37a05-115">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="37a05-116">アプリケーションの一部をストレージ サービスでホストする場合の主な考慮事項は、アプリケーションのデプロイと、匿名ユーザーへの提供を意図していないリソースのセキュリティ保護に関連します。</span><span class="sxs-lookup"><span data-stu-id="37a05-116">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="37a05-117">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="37a05-117">Issues and considerations</span></span>

<span data-ttu-id="37a05-118">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="37a05-118">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="37a05-119">ホストされるストレージ サービスでは、ユーザーが静的リソースをダウンロードするためにアクセスできる HTTP エンドポイントを公開する必要があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-119">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="37a05-120">一部のストレージ サービスでは、SSL を必要とするストレージ サービス内のリソースをホストできるよう、HTTPS もサポートされています。</span><span class="sxs-lookup"><span data-stu-id="37a05-120">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="37a05-121">パフォーマンスと可用性を最大化するために、コンテンツ配信ネットワーク (CDN) を使用して、ストレージ コンテナーのコンテンツを世界各地の複数のデータ センターにキャッシュすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="37a05-121">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="37a05-122">ただし、CDN を使用するには、通常、料金を支払う必要があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-122">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="37a05-123">ストレージ アカウントは、データ センターに影響を与えるイベントが発生した場合の回復性を提供するために、既定で地理的にレプリケートされることが少なくありません。</span><span class="sxs-lookup"><span data-stu-id="37a05-123">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="37a05-124">つまり、IP アドレスが変更される可能性があります。ただし、URL はそのまま維持されます。</span><span class="sxs-lookup"><span data-stu-id="37a05-124">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="37a05-125">一部のコンテンツがストレージ アカウントに配置されており、その他のコンテンツがホストされているコンピューティング インスタンスにある場合、アプリケーションのデプロイや更新がより困難になります。</span><span class="sxs-lookup"><span data-stu-id="37a05-125">When some content is located in a storage account and other content is in a hosted compute instance, it becomes more challenging to deploy and update the application.</span></span> <span data-ttu-id="37a05-126">管理を容易にするには、デプロイを個別に行い、アプリケーションとコンテンツのバージョンを管理する必要が生じる場合があります (特に、静的コンテンツにスクリプト ファイルや UI コンポーネントが含まれている場合)。</span><span class="sxs-lookup"><span data-stu-id="37a05-126">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="37a05-127">ただし、静的リソースだけを更新すればよい場合は、アプリケーション パッケージを再デプロイすることなく、静的リソースだけをストレージ アカウントにアップロードすることもできます。</span><span class="sxs-lookup"><span data-stu-id="37a05-127">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="37a05-128">ストレージ サービスでは、カスタム ドメイン名の使用がサポートされない場合があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-128">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="37a05-129">その場合は、リンク内にリソースの完全 URL を指定する必要があります。これは、リンクを含んだ動的生成コンテンツから、リソースが別のドメインに配置されることがあるためです。</span><span class="sxs-lookup"><span data-stu-id="37a05-129">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="37a05-130">ストレージ コンテナーは、パブリック読み取りアクセス用に構成する必要があります。ただし、ユーザーがコンテンツをアップロードできないようにする必要があるため、パブリック書き込みアクセス用には構成しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="37a05-130">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span>

- <span data-ttu-id="37a05-131">匿名でのアクセスを許可しないリソースについては、バレット キーやトークンを使用してアクセスを制御することを検討してください。</span><span class="sxs-lookup"><span data-stu-id="37a05-131">Consider using a valet key or token to control access to resources that shouldn't be available anonymously.</span></span> <span data-ttu-id="37a05-132">詳細については、「[Valet Key pattern](./valet-key.md)」 (バレット キー パターン) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="37a05-132">See the [Valet Key pattern](./valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="37a05-133">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="37a05-133">When to use this pattern</span></span>

<span data-ttu-id="37a05-134">このパターンは次の目的に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="37a05-134">This pattern is useful for:</span></span>

- <span data-ttu-id="37a05-135">静的リソースを含んだ Web サイトやアプリケーションのホスティング コストを最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="37a05-135">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="37a05-136">静的コンテンツと静的リソースのみで構成される Web サイトのホスティング コストを最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="37a05-136">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="37a05-137">ホスティング プロバイダーのストレージ システムの機能によっては、完全に静的な Web サイトをストレージ アカウントで完全にホストできる場合もあります。</span><span class="sxs-lookup"><span data-stu-id="37a05-137">Depending on the capabilities of the hosting provider's storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="37a05-138">静的リソースと静的コンテンツを、他のホスト環境やオンプレミス サーバーで実行されているアプリケーション用に公開する。</span><span class="sxs-lookup"><span data-stu-id="37a05-138">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="37a05-139">ストレージ アカウントのコンテンツを世界各地の複数のデータ センターにキャッシュするコンテンツ配信ネットワークを使用して、コンテンツを複数の地理的位置に配置する。</span><span class="sxs-lookup"><span data-stu-id="37a05-139">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="37a05-140">コストと帯域幅の使用状況を監視する。</span><span class="sxs-lookup"><span data-stu-id="37a05-140">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="37a05-141">静的コンテンツの一部またはすべてに個別のストレージ アカウントを使用すると、ホスティング コストやランタイム コストとの分離がより簡単になります。</span><span class="sxs-lookup"><span data-stu-id="37a05-141">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="37a05-142">このパターンは、次の状況では有効でない場合があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-142">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="37a05-143">静的コンテンツをクライアントに配信する前に、アプリケーションで何らかの処理を実行する必要がある。</span><span class="sxs-lookup"><span data-stu-id="37a05-143">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="37a05-144">たとえば、ドキュメントにタイムスタンプを追加する必要がある場合などです。</span><span class="sxs-lookup"><span data-stu-id="37a05-144">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="37a05-145">静的コンテンツの量が非常に少ない。</span><span class="sxs-lookup"><span data-stu-id="37a05-145">The volume of static content is very small.</span></span> <span data-ttu-id="37a05-146">これらのコンテンツを個別のストレージから取得するためのオーバーヘッドが、コンピューティング リソースから分離することのコスト メリットを上回る恐れがあります。</span><span class="sxs-lookup"><span data-stu-id="37a05-146">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="37a05-147">例</span><span class="sxs-lookup"><span data-stu-id="37a05-147">Example</span></span>

<span data-ttu-id="37a05-148">Azure Storage では、ストレージ コンテナーから直接静的コンテンツを提供できます。</span><span class="sxs-lookup"><span data-stu-id="37a05-148">Azure Storage supports serving static content directly from a storage container.</span></span> <span data-ttu-id="37a05-149">ファイルは匿名のアクセス要求を通じて提供されます。</span><span class="sxs-lookup"><span data-stu-id="37a05-149">Files are served through anonymous access requests.</span></span> <span data-ttu-id="37a05-150">既定では、ファイルにはサブドメインが `core.windows.net` の URL (`https://contoso.z4.web.core.windows.net/image.png` など) があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-150">By default, files have a URL in a subdomain of `core.windows.net`, such as `https://contoso.z4.web.core.windows.net/image.png`.</span></span> <span data-ttu-id="37a05-151">カスタム ドメイン名を構成し、Azure CDN を使用して、HTTPS 経由でファイルにアクセスすることができます。</span><span class="sxs-lookup"><span data-stu-id="37a05-151">You can configure a custom domain name, and use Azure CDN to access the files over HTTPS.</span></span> <span data-ttu-id="37a05-152">詳細については、「[Azure Storage での静的 Web サイト ホスティング](/azure/storage/blobs/storage-blob-static-website)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="37a05-152">For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span></span>

![アプリケーションの静的部分をストレージ サービスから直接配信する](./_images/static-content-hosting-pattern.png)

<span data-ttu-id="37a05-154">静的 Web サイト ホスティングでは、匿名アクセスでファイルを使用できるようにします。</span><span class="sxs-lookup"><span data-stu-id="37a05-154">Static website hosting makes the files available for anonymous access.</span></span> <span data-ttu-id="37a05-155">ファイルにアクセスできるユーザーを制御する必要がある場合は、Azure BLOB ストレージにファイルを格納し、[共有アクセス署名](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)を生成してアクセスを制限できます。</span><span class="sxs-lookup"><span data-stu-id="37a05-155">If you need to control who can access the files, you can store files in Azure blob storage and then generate [shared access signatures](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) to limit access.</span></span>

<span data-ttu-id="37a05-156">クライアントに配信されるページ内のリンクでは、リソースの完全 URL を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-156">The links in the pages delivered to the client must specify the full URL of the resource.</span></span> <span data-ttu-id="37a05-157">リソースがバレット キー (共有アクセス署名など) を使用して保護されている場合、この署名は URL に含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="37a05-157">If the resource is protected with a valet key, such as a shared access signature, this signature must be included in the URL.</span></span>

<span data-ttu-id="37a05-158">静的リソース用の外部ストレージの使用方法を示すサンプル アプリケーションは、[GitHub][sample-app] で入手できます。</span><span class="sxs-lookup"><span data-stu-id="37a05-158">A sample application that demonstrates using external storage for static resources is available on [GitHub][sample-app].</span></span> <span data-ttu-id="37a05-159">このサンプルでは、ストレージ アカウントを指定する構成ファイルと、静的コンテンツを保持するコンテナーを使用します。</span><span class="sxs-lookup"><span data-stu-id="37a05-159">This sample uses configuration files to specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="37a05-160">StaticContentHosting.Web プロジェクトの Settings.cs ファイル内にある `Settings` クラスには、 これらの値を抽出し、クラウド ストレージ アカウントのコンテナー URL を含んだ文字列値を構築するメソッドが含まれています。</span><span class="sxs-lookup"><span data-stu-id="37a05-160">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="37a05-161">StaticContentUrlHtmlHelper.cs ファイル内の `StaticContentUrlHtmlHelper` クラスは、渡された URL が ASP.NET のルート パス文字 (~) で始まる場合に、クラウド ストレージ アカウントへのパスを含んだ URL を生成する、`StaticContentUrl` というメソッドを公開します。</span><span class="sxs-lookup"><span data-stu-id="37a05-161">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="37a05-162">Views\Home フォルダー内の Index.cshtml ファイルには、`StaticContentUrl` メソッドを使用して `src` 属性の URL を作成する画像要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="37a05-162">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="37a05-163">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="37a05-163">Related patterns and guidance</span></span>

- <span data-ttu-id="37a05-164">[静的コンテンツ ホスティングのサンプル][sample-app]。</span><span class="sxs-lookup"><span data-stu-id="37a05-164">[Static Content Hosting sample][sample-app].</span></span> <span data-ttu-id="37a05-165">このパターンを示すサンプル アプリケーション。</span><span class="sxs-lookup"><span data-stu-id="37a05-165">A sample application that demonstrates this pattern.</span></span>
- <span data-ttu-id="37a05-166">「[Valet Key pattern](./valet-key.md)」(バレット キー パターン)。</span><span class="sxs-lookup"><span data-stu-id="37a05-166">[Valet Key pattern](./valet-key.md).</span></span> <span data-ttu-id="37a05-167">ターゲット リソースの使用を匿名ユーザーに許可しない場合は、このパターンを使用して直接アクセスを制限します。</span><span class="sxs-lookup"><span data-stu-id="37a05-167">If the target resources aren't supposed to be available to anonymous users, use this pattern to restrict direct access.</span></span>
- <span data-ttu-id="37a05-168">[Azure 上のサーバーレス Web アプリケーション](../reference-architectures/serverless/web-app.md)。</span><span class="sxs-lookup"><span data-stu-id="37a05-168">[Serverless web application on Azure](../reference-architectures/serverless/web-app.md).</span></span> <span data-ttu-id="37a05-169">サーバーレス Web アプリを実装するために、Azure Functions で静的 Web サイトのホスティングを使用する参照アーキテクチャ。</span><span class="sxs-lookup"><span data-stu-id="37a05-169">A reference architecture that uses static website hosting with Azure Functions to implement a serverless web app.</span></span>

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
