---
title: 静的コンテンツ ホスティング
description: 静的コンテンツを、クライアントに直接配信できるクラウド ベースのストレージ サービスにデプロイします。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 450d0c4c08098c1ba48e4c0dac3d058a46e3709b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428213"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="75d42-104">静的コンテンツ ホスティング パターン</span><span class="sxs-lookup"><span data-stu-id="75d42-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="75d42-105">静的コンテンツを、クライアントに直接配信できるクラウド ベースのストレージ サービスにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="75d42-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="75d42-106">これにより、高額なコンピューティング インスタンスの必要性を低減できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="75d42-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="75d42-107">Context and problem</span></span>

<span data-ttu-id="75d42-108">通常、Web アプリケーションには静的コンテンツの要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="75d42-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="75d42-109">この静的コンテンツには、HTML ページのほか、クライアントから利用可能な画像やドキュメントが、HTML ページの一部 (インライン画像、スタイル シート、クライアント側 JavaScript ファイルなど) か、個別のダウンロード (PDF ドキュメントなど) として含まれています。</span><span class="sxs-lookup"><span data-stu-id="75d42-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="75d42-110">Web サーバーは、効率的な動的ページ コード実行や出力キャッシュを通じて、要求を最適化するようにチューニングされていますが、それでも、静的コンテンツをダウンロードする要求は処理する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="75d42-111">このために消費される処理サイクルは、他の目的のために、より効果的に使用できる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="75d42-112">解決策</span><span class="sxs-lookup"><span data-stu-id="75d42-112">Solution</span></span>

<span data-ttu-id="75d42-113">ほとんどのクラウド ホスティング環境では、アプリケーションのリソースや静的ページの一部をストレージ サービスに配置することで、コンピューティング インスタンスの必要性を最小化 (たとえば、インスタンスを小さくしたり、インスタンス数を減らすなど) することができます。</span><span class="sxs-lookup"><span data-stu-id="75d42-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="75d42-114">クラウドでホストされるストレージのコストは、通常、多くのコンピューティング インスタンスのコストよりも安価です。</span><span class="sxs-lookup"><span data-stu-id="75d42-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="75d42-115">アプリケーションの一部をストレージ サービスでホストする場合の主な考慮事項は、アプリケーションのデプロイと、匿名ユーザーへの提供を意図していないリソースのセキュリティ保護に関連します。</span><span class="sxs-lookup"><span data-stu-id="75d42-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="75d42-116">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="75d42-116">Issues and considerations</span></span>

<span data-ttu-id="75d42-117">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="75d42-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="75d42-118">ホストされるストレージ サービスでは、ユーザーが静的リソースをダウンロードするためにアクセスできる HTTP エンドポイントを公開する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="75d42-119">一部のストレージ サービスでは、SSL を必要とするストレージ サービス内のリソースをホストできるよう、HTTPS もサポートされています。</span><span class="sxs-lookup"><span data-stu-id="75d42-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="75d42-120">パフォーマンスと可用性を最大化するために、コンテンツ配信ネットワーク (CDN) を使用して、ストレージ コンテナーのコンテンツを世界各地の複数のデータ センターにキャッシュすることを検討してください。</span><span class="sxs-lookup"><span data-stu-id="75d42-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="75d42-121">ただし、CDN を使用するには、通常、料金を支払う必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="75d42-122">ストレージ アカウントは、データ センターに影響を与えるイベントが発生した場合の回復性を提供するために、既定で地理的にレプリケートされることが少なくありません。</span><span class="sxs-lookup"><span data-stu-id="75d42-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="75d42-123">つまり、IP アドレスが変更される可能性があります。ただし、URL はそのまま維持されます。</span><span class="sxs-lookup"><span data-stu-id="75d42-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="75d42-124">一部のコンテンツをストレージ アカウントに配置し、その他のコンテンツをコンピューティング インスタンスでホストする場合は、アプリケーションのデプロイや更新がより困難になります。</span><span class="sxs-lookup"><span data-stu-id="75d42-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="75d42-125">管理を容易にするには、デプロイを個別に行い、アプリケーションとコンテンツのバージョンを管理する必要が生じる場合があります (特に、静的コンテンツにスクリプト ファイルや UI コンポーネントが含まれている場合)。</span><span class="sxs-lookup"><span data-stu-id="75d42-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="75d42-126">ただし、静的リソースだけを更新すればよい場合は、アプリケーション パッケージを再デプロイすることなく、静的リソースだけをストレージ アカウントにアップロードすることもできます。</span><span class="sxs-lookup"><span data-stu-id="75d42-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="75d42-127">ストレージ サービスでは、カスタム ドメイン名の使用がサポートされない場合があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="75d42-128">その場合は、リンク内にリソースの完全 URL を指定する必要があります。これは、リンクを含んだ動的生成コンテンツから、リソースが別のドメインに配置されることがあるためです。</span><span class="sxs-lookup"><span data-stu-id="75d42-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="75d42-129">ストレージ コンテナーは、パブリック読み取りアクセス用に構成する必要があります。ただし、ユーザーがコンテンツをアップロードできないようにする必要があるため、パブリック書き込みアクセス用には構成しないようにしてください。</span><span class="sxs-lookup"><span data-stu-id="75d42-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="75d42-130">匿名でのアクセスを許可しないリソースについては、バレット キーやトークンを使用してアクセスを制御することを検討してください。詳細については、「[Valet Key pattern](valet-key.md)」(バレット キー パターン) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="75d42-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="75d42-131">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="75d42-131">When to use this pattern</span></span>

<span data-ttu-id="75d42-132">このパターンは次の目的に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="75d42-132">This pattern is useful for:</span></span>

- <span data-ttu-id="75d42-133">静的リソースを含んだ Web サイトやアプリケーションのホスティング コストを最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="75d42-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="75d42-134">静的コンテンツと静的リソースのみで構成される Web サイトのホスティング コストを最小限に抑える。</span><span class="sxs-lookup"><span data-stu-id="75d42-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="75d42-135">ホスティング プロバイダーが提供するストレージ システムの機能によっては、完全に静的な Web サイトをストレージ アカウントで丸ごとホストできる場合もあります。</span><span class="sxs-lookup"><span data-stu-id="75d42-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="75d42-136">静的リソースと静的コンテンツを、他のホスト環境やオンプレミス サーバーで実行されているアプリケーション用に公開する。</span><span class="sxs-lookup"><span data-stu-id="75d42-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="75d42-137">ストレージ アカウントのコンテンツを世界各地の複数のデータ センターにキャッシュするコンテンツ配信ネットワークを使用して、コンテンツを複数の地理的位置に配置する。</span><span class="sxs-lookup"><span data-stu-id="75d42-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="75d42-138">コストと帯域幅の使用状況を監視する。</span><span class="sxs-lookup"><span data-stu-id="75d42-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="75d42-139">静的コンテンツの一部またはすべてに個別のストレージ アカウントを使用すると、ホスティング コストやランタイム コストとの分離がより簡単になります。</span><span class="sxs-lookup"><span data-stu-id="75d42-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="75d42-140">このパターンは、次の状況では有効でない場合があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="75d42-141">静的コンテンツをクライアントに配信する前に、アプリケーションで何らかの処理を実行する必要がある。</span><span class="sxs-lookup"><span data-stu-id="75d42-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="75d42-142">たとえば、ドキュメントにタイムスタンプを追加する必要がある場合などです。</span><span class="sxs-lookup"><span data-stu-id="75d42-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="75d42-143">静的コンテンツの量が非常に少ない。</span><span class="sxs-lookup"><span data-stu-id="75d42-143">The volume of static content is very small.</span></span> <span data-ttu-id="75d42-144">これらのコンテンツを個別のストレージから取得するためのオーバーヘッドが、コンピューティング リソースから分離することのコスト メリットを上回る恐れがあります。</span><span class="sxs-lookup"><span data-stu-id="75d42-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="75d42-145">例</span><span class="sxs-lookup"><span data-stu-id="75d42-145">Example</span></span>

<span data-ttu-id="75d42-146">Azure Blob ストレージに配置された静的コンテンツは、Web ブラウザーから直接アクセスできます。</span><span class="sxs-lookup"><span data-stu-id="75d42-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="75d42-147">Azure では、クライアントにパブリックに公開できるストレージ上で、HTTP ベースのインターフェイスを提供しています。</span><span class="sxs-lookup"><span data-stu-id="75d42-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="75d42-148">たとえば、Azure Blob ストレージ コンテナー内のコンテンツは、次の形式の URL を使用して公開されます。</span><span class="sxs-lookup"><span data-stu-id="75d42-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="75d42-149">コンテンツをアップロードする際には、ファイルやドキュメントを保持するための、1 つ以上の BLOB コンテナーを作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="75d42-150">新しいコンテナーの既定のアクセス許可はプライベートです。クライアントがコンテンツにアクセスできるようにするには、これをパブリックに変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="75d42-151">コンテンツを匿名アクセスから保護する必要がある場合は、[バレット キー パターン](valet-key.md)を実装できます。これによりユーザーは、有効なトークンを提示しないとリソースをダウンロードできなくなります。</span><span class="sxs-lookup"><span data-stu-id="75d42-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="75d42-152">Blob ストレージに関する情報と、その使用方法やアクセス使用については、「[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)」(Blob サービスの概念) を参照してください。</span><span class="sxs-lookup"><span data-stu-id="75d42-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="75d42-153">各ページ内のリンクはリソースの URL を指しており、クライアントはストレージ サービスからそれらのリソースに直接アクセスします。</span><span class="sxs-lookup"><span data-stu-id="75d42-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="75d42-154">次の図は、アプリケーションの静的部分をストレージ サービスから直接配信する様子を示しています。</span><span class="sxs-lookup"><span data-stu-id="75d42-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![図 1 - アプリケーションの静的部分をストレージ サービスから直接配信する](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="75d42-156">クライアントに配信されるページ内のリンクでは、BLOB コンテナーやリソースの完全 URL を指定する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="75d42-157">たとえば、パブリック コンテナー内の画像へのリンクを含んだページには、次の HTML コードが含まれる可能性があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="75d42-158">リソースがバレット キー (Azure の共有アクセス署名など) を使用して保護されている場合は、リンクの URL にその署名が含まれている必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="75d42-159">[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) では、静的リソース用の外部ストレージの使用方法を示す、StaticContentHosting というソリューションが公開されています。</span><span class="sxs-lookup"><span data-stu-id="75d42-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="75d42-160">StaticContentHosting.Cloud プロジェクトには、ストレージ アカウントを指定する構成ファイルと、静的コンテンツを保持するコンテナーが含まれています。</span><span class="sxs-lookup"><span data-stu-id="75d42-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="75d42-161">StaticContentHosting.Web プロジェクトの Settings.cs ファイル内にある `Settings` クラスには、 これらの値を抽出し、クラウド ストレージ アカウントのコンテナー URL を含んだ文字列値を構築するメソッドが含まれています。</span><span class="sxs-lookup"><span data-stu-id="75d42-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="75d42-162">StaticContentUrlHtmlHelper.cs ファイル内の `StaticContentUrlHtmlHelper` クラスは、渡された URL が ASP.NET のルート パス文字 (~) で始まる場合に、クラウド ストレージ アカウントへのパスを含んだ URL を生成する、`StaticContentUrl` というメソッドを公開します。</span><span class="sxs-lookup"><span data-stu-id="75d42-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="75d42-163">Views\Home フォルダー内の Index.cshtml ファイルには、`StaticContentUrl` メソッドを使用して `src` 属性の URL を作成する画像要素が含まれています。</span><span class="sxs-lookup"><span data-stu-id="75d42-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="75d42-164">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="75d42-164">Related patterns and guidance</span></span>

- <span data-ttu-id="75d42-165">このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) から入手できます。</span><span class="sxs-lookup"><span data-stu-id="75d42-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="75d42-166">「[Valet Key pattern](valet-key.md)」(バレット キー パターン)。</span><span class="sxs-lookup"><span data-stu-id="75d42-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="75d42-167">ターゲット リソースの利用を匿名ユーザーに許可しない場合は、静的コンテンツを保持するストアに対してセキュリティを実装する必要があります。</span><span class="sxs-lookup"><span data-stu-id="75d42-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="75d42-168">トークンやキーを使用して、特定のリソースやサービス (クラウド ホスト型ストレージ サービスなど) に対するクライアントの直接アクセスを制限する方法について説明しています。</span><span class="sxs-lookup"><span data-stu-id="75d42-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="75d42-169">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)(Blob サービスの概念)</span><span class="sxs-lookup"><span data-stu-id="75d42-169">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)</span></span>
