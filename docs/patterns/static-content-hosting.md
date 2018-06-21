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
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541691"
---
# <a name="static-content-hosting-pattern"></a>静的コンテンツ ホスティング パターン

[!INCLUDE [header](../_includes/header.md)]

静的コンテンツを、クライアントに直接配信できるクラウド ベースのストレージ サービスにデプロイします。 これにより、高額なコンピューティング インスタンスの必要性を低減できる可能性があります。

## <a name="context-and-problem"></a>コンテキストと問題

通常、Web アプリケーションには静的コンテンツの要素が含まれています。 この静的コンテンツには、HTML ページのほか、クライアントから利用可能な画像やドキュメントが、HTML ページの一部 (インライン画像、スタイル シート、クライアント側 JavaScript ファイルなど) か、個別のダウンロード (PDF ドキュメントなど) として含まれています。

Web サーバーは、効率的な動的ページ コード実行や出力キャッシュを通じて、要求を最適化するようにチューニングされていますが、それでも、静的コンテンツをダウンロードする要求は処理する必要があります。 このために消費される処理サイクルは、他の目的のために、より効果的に使用できる可能性があります。

## <a name="solution"></a>解決策

ほとんどのクラウド ホスティング環境では、アプリケーションのリソースや静的ページの一部をストレージ サービスに配置することで、コンピューティング インスタンスの必要性を最小化 (たとえば、インスタンスを小さくしたり、インスタンス数を減らすなど) することができます。 クラウドでホストされるストレージのコストは、通常、多くのコンピューティング インスタンスのコストよりも安価です。

アプリケーションの一部をストレージ サービスでホストする場合の主な考慮事項は、アプリケーションのデプロイと、匿名ユーザーへの提供を意図していないリソースのセキュリティ保護に関連します。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

- ホストされるストレージ サービスでは、ユーザーが静的リソースをダウンロードするためにアクセスできる HTTP エンドポイントを公開する必要があります。 一部のストレージ サービスでは、SSL を必要とするストレージ サービス内のリソースをホストできるよう、HTTPS もサポートされています。

- パフォーマンスと可用性を最大化するために、コンテンツ配信ネットワーク (CDN) を使用して、ストレージ コンテナーのコンテンツを世界各地の複数のデータ センターにキャッシュすることを検討してください。 ただし、CDN を使用するには、通常、料金を支払う必要があります。

- ストレージ アカウントは、データ センターに影響を与えるイベントが発生した場合の回復性を提供するために、既定で地理的にレプリケートされることが少なくありません。 つまり、IP アドレスが変更される可能性があります。ただし、URL はそのまま維持されます。

- 一部のコンテンツをストレージ アカウントに配置し、その他のコンテンツをコンピューティング インスタンスでホストする場合は、アプリケーションのデプロイや更新がより困難になります。 管理を容易にするには、デプロイを個別に行い、アプリケーションとコンテンツのバージョンを管理する必要が生じる場合があります (特に、静的コンテンツにスクリプト ファイルや UI コンポーネントが含まれている場合)。 ただし、静的リソースだけを更新すればよい場合は、アプリケーション パッケージを再デプロイすることなく、静的リソースだけをストレージ アカウントにアップロードすることもできます。

- ストレージ サービスでは、カスタム ドメイン名の使用がサポートされない場合があります。 その場合は、リンク内にリソースの完全 URL を指定する必要があります。これは、リンクを含んだ動的生成コンテンツから、リソースが別のドメインに配置されることがあるためです。

- ストレージ コンテナーは、パブリック読み取りアクセス用に構成する必要があります。ただし、ユーザーがコンテンツをアップロードできないようにする必要があるため、パブリック書き込みアクセス用には構成しないようにしてください。 匿名でのアクセスを許可しないリソースについては、バレット キーやトークンを使用してアクセスを制御することを検討してください。詳細については、「[Valet Key pattern](valet-key.md)」(バレット キー パターン) を参照してください。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の目的に役立ちます。

- 静的リソースを含んだ Web サイトやアプリケーションのホスティング コストを最小限に抑える。

- 静的コンテンツと静的リソースのみで構成される Web サイトのホスティング コストを最小限に抑える。 ホスティング プロバイダーが提供するストレージ システムの機能によっては、完全に静的な Web サイトをストレージ アカウントで丸ごとホストできる場合もあります。

- 静的リソースと静的コンテンツを、他のホスト環境やオンプレミス サーバーで実行されているアプリケーション用に公開する。

- ストレージ アカウントのコンテンツを世界各地の複数のデータ センターにキャッシュするコンテンツ配信ネットワークを使用して、コンテンツを複数の地理的位置に配置する。

- コストと帯域幅の使用状況を監視する。 静的コンテンツの一部またはすべてに個別のストレージ アカウントを使用すると、ホスティング コストやランタイム コストとの分離がより簡単になります。

このパターンは、次の状況では有効でない場合があります。

- 静的コンテンツをクライアントに配信する前に、アプリケーションで何らかの処理を実行する必要がある。 たとえば、ドキュメントにタイムスタンプを追加する必要がある場合などです。

- 静的コンテンツの量が非常に少ない。 これらのコンテンツを個別のストレージから取得するためのオーバーヘッドが、コンピューティング リソースから分離することのコスト メリットを上回る恐れがあります。

## <a name="example"></a>例

Azure Blob ストレージに配置された静的コンテンツは、Web ブラウザーから直接アクセスできます。 Azure では、クライアントにパブリックに公開できるストレージ上で、HTTP ベースのインターフェイスを提供しています。 たとえば、Azure Blob ストレージ コンテナー内のコンテンツは、次の形式の URL を使用して公開されます。

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


コンテンツをアップロードする際には、ファイルやドキュメントを保持するための、1 つ以上の BLOB コンテナーを作成する必要があります。 新しいコンテナーの既定のアクセス許可はプライベートです。クライアントがコンテンツにアクセスできるようにするには、これをパブリックに変更する必要があります。 コンテンツを匿名アクセスから保護する必要がある場合は、[バレット キー パターン](valet-key.md)を実装できます。これによりユーザーは、有効なトークンを提示しないとリソースをダウンロードできなくなります。

> Blob ストレージに関する情報と、その使用方法やアクセス使用については、「[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)」(Blob サービスの概念) を参照してください。

各ページ内のリンクはリソースの URL を指しており、クライアントはストレージ サービスからそれらのリソースに直接アクセスします。 次の図は、アプリケーションの静的部分をストレージ サービスから直接配信する様子を示しています。

![図 1 - アプリケーションの静的部分をストレージ サービスから直接配信する](./_images/static-content-hosting-pattern.png)


クライアントに配信されるページ内のリンクでは、BLOB コンテナーやリソースの完全 URL を指定する必要があります。 たとえば、パブリック コンテナー内の画像へのリンクを含んだページには、次の HTML コードが含まれる可能性があります。

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> リソースがバレット キー (Azure の共有アクセス署名など) を使用して保護されている場合は、リンクの URL にその署名が含まれている必要があります。

[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) では、静的リソース用の外部ストレージの使用方法を示す、StaticContentHosting というソリューションが公開されています。 StaticContentHosting.Cloud プロジェクトには、ストレージ アカウントを指定する構成ファイルと、静的コンテンツを保持するコンテナーが含まれています。

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

StaticContentHosting.Web プロジェクトの Settings.cs ファイル内にある `Settings` クラスには、 これらの値を抽出し、クラウド ストレージ アカウントのコンテナー URL を含んだ文字列値を構築するメソッドが含まれています。

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

StaticContentUrlHtmlHelper.cs ファイル内の `StaticContentUrlHtmlHelper` クラスは、渡された URL が ASP.NET のルート パス文字 (~) で始まる場合に、クラウド ストレージ アカウントへのパスを含んだ URL を生成する、`StaticContentUrl` というメソッドを公開します。

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

Views\Home フォルダー内の Index.cshtml ファイルには、`StaticContentUrl` メソッドを使用して `src` 属性の URL を作成する画像要素が含まれています。

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

- このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) から入手できます。
- 「[Valet Key pattern](valet-key.md)」(バレット キー パターン)。 ターゲット リソースの利用を匿名ユーザーに許可しない場合は、静的コンテンツを保持するストアに対してセキュリティを実装する必要があります。 トークンやキーを使用して、特定のリソースやサービス (クラウド ホスト型ストレージ サービスなど) に対するクライアントの直接アクセスを制限する方法について説明しています。
- Infosys ブログ内の「[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html)」(静的 Web サイトを Azure 上にデプロイするための効率的な方法)。
- [Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)(Blob サービスの概念)
