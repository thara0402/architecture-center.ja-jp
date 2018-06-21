---
title: 外部構成ストア
description: アプリケーション展開パッケージの構成情報を一元管理される場所に移動します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
ms.locfileid: "24542283"
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="dc25b-104">外部構成ストア パターン</span><span class="sxs-lookup"><span data-stu-id="dc25b-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="dc25b-105">アプリケーション展開パッケージの構成情報を一元管理される場所に移動します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="dc25b-106">こうすることで、構成データの管理と制御が簡単になり、構成データをアプリケーションとアプリケーション インスタンス全体で共有できるようになります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="dc25b-107">コンテキストと問題</span><span class="sxs-lookup"><span data-stu-id="dc25b-107">Context and problem</span></span>

<span data-ttu-id="dc25b-108">多くのアプリケーション ランタイム環境には、アプリケーションと共にデプロイされるファイルに保持される構成情報が含まれています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="dc25b-109">場合によっては、これらのファイルを編集して、デプロイ後にアプリケーションの動作を変更することができます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="dc25b-110">ただし、構成を変更するには、アプリケーションを再デプロイする必要があります。また、その結果、許容できないダウンタイムや他の管理上のオーバーヘッドが発生することもよくあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="dc25b-111">ローカル構成ファイルでも構成は単一のアプリケーションに制限されていますが、複数のアプリケーションで構成設定を共有することが適している場合もあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="dc25b-112">たとえば、データベース接続文字列、UI テーマ情報、関連するアプリケーション セットに使用されるキューとストレージの URL があります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="dc25b-113">アプリケーションの複数の実行インスタンス全体でローカル構成の変更を管理することが困難になります。クラウドホスト型のシナリオの場合は特に困難です。</span><span class="sxs-lookup"><span data-stu-id="dc25b-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="dc25b-114">インスタンスは異なる構成設定を使用することになりますが、更新プログラムはデプロイされます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="dc25b-115">さらに、アプリケーションとコンポーネントの更新プログラムでは、構成スキーマの変更が必要になることがあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="dc25b-116">多くの構成システムは、異なるバージョンの構成情報をサポートしていません。</span><span class="sxs-lookup"><span data-stu-id="dc25b-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="dc25b-117">解決策</span><span class="sxs-lookup"><span data-stu-id="dc25b-117">Solution</span></span>

<span data-ttu-id="dc25b-118">外部ストレージに構成情報を格納し、構成設定の読み取りと更新をすばやく効率的に行うために使用できるインターフェイスを用意します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="dc25b-119">外部ストアの種類は、アプリケーションのホスティングおよびランタイム環境によって変わります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="dc25b-120">クラウドホスト型シナリオでは、一般的にクラウドベースのストレージ サービスですが、ホスト型データベースや他のシステムの場合もあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="dc25b-121">構成情報のために選択するバッキング ストアには、一貫した使いやすいアクセスを提供するインターフェイスがあるものをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="dc25b-122">正しく入力され、構造化された形式で情報を公開します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="dc25b-123">実装では、必要に応じてユーザーのアクセスを承認して、構成データを保護するようにします。また、複数バージョンの構成 (それぞれの複数リリース バージョンを含め、開発、ステージング、運用など) のストレージを許容できるように柔軟にします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="dc25b-124">多くの組み込み構成システムは、アプリケーションの起動時にデータを読み取り、データをメモリ内にキャッシュして高速なアクセスを提供し、アプリケーション参照に対する影響を最小限に抑えます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="dc25b-125">使用するバッキング ストアの種類とそのストアの待機時間によっては、外部構成ストア内にキャッシュ メカニズムを実装することが役立つ場合があります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="dc25b-126">詳細については、「 [Caching Guidance (キャッシュのガイダンス)](https://msdn.microsoft.com/library/dn589802.aspx)」を参照してください。</span><span class="sxs-lookup"><span data-stu-id="dc25b-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="dc25b-127">この図は、省略可能なローカル キャッシュがある外部構成ストア パターンの概要を示しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![省略可能なローカル キャッシュがある外部構成ストア パターンの概要。](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a><span data-ttu-id="dc25b-129">問題と注意事項</span><span class="sxs-lookup"><span data-stu-id="dc25b-129">Issues and considerations</span></span>

<span data-ttu-id="dc25b-130">このパターンの実装方法を決めるときには、以下の点に注意してください。</span><span class="sxs-lookup"><span data-stu-id="dc25b-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="dc25b-131">許容できるパフォーマンス、高可用性、堅牢性を提供し、アプリケーション保守と管理プロセスの一部としてバックアップすることができるバッキング ストアを選択します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="dc25b-132">クラウドホスト型アプリケーションの場合、これらの要件を満たすには、通常、クラウド ストレージ メカニズムの使用がお勧めです。</span><span class="sxs-lookup"><span data-stu-id="dc25b-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="dc25b-133">保持できる情報の種類に柔軟性を持たせるようにバッキング ストアのスキーマを設計します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="dc25b-134">型指定されたデータ、設定のコレクション、複数バージョンの設定、使用するアプリケーションに必要なその他の機能など、すべての構成要件に備えるようにします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="dc25b-135">要件の変更に応じて追加の設定をサポートできるように、拡張しやすいスキーマにします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="dc25b-136">バッキング ストアの物理機能、構成情報を格納する方法と関連付ける方法、およびパフォーマンスに対する影響を検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="dc25b-137">たとえば、構成情報を含む XML ドキュメントの保存には、個々の設定を読み取るために、ドキュメントを解析できる構成インターフェイスまたはアプリケーションが必要です。</span><span class="sxs-lookup"><span data-stu-id="dc25b-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="dc25b-138">設定の更新は複雑になりますが、設定のキャッシュによって、読み取りパフォーマンスの低下と相殺することができます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="dc25b-139">構成インターフェイスで、構成設定の範囲と継承の制御を許可する方法を検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="dc25b-140">たとえば、構成設定の範囲を組織、アプリケーション、およびコンピューター レベルに設定する要件が考えられます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="dc25b-141">必要に応じて、異なる範囲に対するアクセスの制御の委任をサポートし、個々のアプリケーションが設定を上書きすることを禁止または許可します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="dc25b-142">構成インターフェイスで、型指定された値、コレクション、キー/値のペア、プロパティ バッグなど、必要な形式で構成データを公開できるようにします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="dc25b-143">設定にエラーが含まれる場合、または設定がバッキング ストアに存在しない場合の、構成ストア インターフェイスの動作方法を検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="dc25b-144">既定の設定とログ エラーを返すことが適切な場合もあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="dc25b-145">また、構成設定のキーや名前の大文字と小文字の区別、バイナリ データの保存と処理、null 値または空の値の処理方法などの側面も検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="dc25b-146">構成データを保護して、適切なユーザーとアプリケーションにのみアクセスを許可する方法を検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="dc25b-147">これは構成ストア インターフェイスの機能が考えられますが、適切なアクセス許可なしでバッキング ストアのデータに直接アクセスできないようにすることも必要です。</span><span class="sxs-lookup"><span data-stu-id="dc25b-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="dc25b-148">構成データの読み取りと書き込みに必要なアクセス許可は、厳密に分離します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="dc25b-149">また、構成設定の一部またはすべてを暗号化する必要があるかどうか、それを構成ストア インターフェイスにどのように実装するかについても検討します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="dc25b-150">一元的に保存されている構成で、実行時にアプリケーションの動作を変更する構成は非常に重要です。アプリケーション コードのデプロイと同じメカニズムを使用して、デプロイ、更新、および管理することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="dc25b-151">たとえば、複数のアプリケーションに影響する可能性がある変更は、その構成を使用するすべてのアプリケーションにその変更が適切できることを確認するために、完全なテストと段階的なデプロイ手法を使用して実施する必要があります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="dc25b-152">管理者が 1 つのアプリケーションを更新するように設定を編集する場合、同じ設定を使用する他のアプリケーションに悪影響が出る可能性があります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="dc25b-153">アプリケーションが構成情報をキャッシュしている場合、構成が変更されたときにアプリケーションは通知を受ける必要があります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="dc25b-154">その情報が定期的に自動更新され、すべての変更が取得 (および実行) されるように、キャッシュされている構成データに対して有効期限ポリシーを実装することができます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="dc25b-155">このパターンを使用する状況</span><span class="sxs-lookup"><span data-stu-id="dc25b-155">When to use this pattern</span></span>

<span data-ttu-id="dc25b-156">このパターンは次の場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-156">This pattern is useful for:</span></span>

- <span data-ttu-id="dc25b-157">複数のアプリケーションとアプリケーション インスタンス間で構成設定を共有する場合、または標準の構成を複数のアプリケーションとアプリケーション インスタンス全体に適用する必要がある場合。</span><span class="sxs-lookup"><span data-stu-id="dc25b-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="dc25b-158">画像や複雑なデータ型の格納など、必要なすべての構成設定をサポートしない標準の構成システム。</span><span class="sxs-lookup"><span data-stu-id="dc25b-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="dc25b-159">一元的に保存されている設定の一部またはすべてをアプリケーションで上書きできるようにする場合など、アプリケーションの一部の設定の補完的なストアとして。</span><span class="sxs-lookup"><span data-stu-id="dc25b-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="dc25b-160">複数のアプリケーションの管理を簡易化する方法として。また、必要に応じて、構成ストアに対する一部またはすべての種類のアクセスをログに記録して、構成設定の使用を監視するため。</span><span class="sxs-lookup"><span data-stu-id="dc25b-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="dc25b-161">例</span><span class="sxs-lookup"><span data-stu-id="dc25b-161">Example</span></span>

<span data-ttu-id="dc25b-162">Microsoft Azure でホストされるアプリケーションの場合、構成情報を外部に保存する一般的な選択肢は、Azure Storage を使用することです。</span><span class="sxs-lookup"><span data-stu-id="dc25b-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="dc25b-163">Azure Storage は回復力があり、パフォーマンスが高く、自動フェールオーバーで 3 回レプリケートされるので、高可用性を実現できます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="dc25b-164">Azure Table Storage には、値に柔軟なスキーマを使用できる機能を持つキー/値のストアが用意されています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="dc25b-165">Azure Blob ストレージには、個別の名前の BLOB に任意の種類のデータを保持できる階層型でコンテナーベースのストアが用意されています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="dc25b-166">次の例は、構成情報の格納と公開に使用する構成ストアを BLOB ストレージに実装する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="dc25b-167">次のコードのように、構成情報を保持できるように `BlobSettingsStore` クラスで BLOB ストレージを抽象化し、`ISettingsStore` インターフェイスを実装します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="dc25b-168">このコードは、[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) から入手できる _ExternalConfigurationStore_ ソリューションの _ExternalConfigurationStore.Cloud_ プロジェクトで提供されています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="dc25b-169">このインターフェイスでは、構成ストアに保持される構成設定の取得と更新に使用されるメソッドを定義しています。また、いずれかの構成設定が最近変更されたかどうかを検出するために使用できるバージョン番号が含まれています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="dc25b-170">`BlobSettingsStore` クラスは、バージョン管理を実装するために BLOB の `ETag` プロパティを使用しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="dc25b-171">`ETag` プロパティは、BLOB が書き込まれるたびに自動的に更新されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="dc25b-172">設計により、この単純なソリューションは、すべての構成設定を型指定された値ではなく文字列値として公開します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="dc25b-173">`ExternalConfigurationManager` クラスは、`BlobSettingsStore` オブジェクトのラッパーを提供します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="dc25b-174">アプリケーションはこのクラスを使用して、構成情報を格納および取得できます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="dc25b-175">このクラスは、Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) ライブラリを使用し、`IObservable` インターフェイスを実装して、構成に変更が加えられた場合はその変更を公開します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="dc25b-176">`SetAppSetting` メソッドを呼び出して設定が変更された場合は、`Changed` イベントが発生し、このイベントのサブスクライバーすべてに通知されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="dc25b-177">すべての設定は、すばやくアクセスできるように、`ExternalConfigurationManager` クラス内の `Dictionary` オブジェクトにもキャッシュされます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="dc25b-178">構成設定の取得に使用された `GetSetting` メソッドで、キャッシュからデータを読み取ります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="dc25b-179">キャッシュに設定が見つからなかった場合は、代わりに `BlobSettingsStore` オブジェクトから取得されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="dc25b-180">`GetSettings` メソッドは `CheckForConfigurationChanges` メソッドを呼び出して、BLOB ストレージの構成情報が変更されたかどうかを検出します。</span><span class="sxs-lookup"><span data-stu-id="dc25b-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="dc25b-181">この処理のために、バージョン番号が確認され、`ExternalConfigurationManager` オブジェクトに保持されている最新バージョン番号が比較されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="dc25b-182">1 つ以上の変更が発生した場合、`Changed` イベントが発生し、`Dictionary` オブジェクトにキャッシュされている構成設定は更新されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="dc25b-183">これは、[キャッシュアサイド パターン](cache-aside.md)のアプリケーションです。</span><span class="sxs-lookup"><span data-stu-id="dc25b-183">This is an application of the [Cache-Aside pattern](cache-aside.md).</span></span>

<span data-ttu-id="dc25b-184">次のコード サンプルは、`Changed` イベント、`GetSettings` メソッド、および `CheckForConfigurationChanges` メソッドの実装方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> <span data-ttu-id="dc25b-185">`ExternalConfigurationManager` クラスにも、`Environment` というプロパティがあります。</span><span class="sxs-lookup"><span data-stu-id="dc25b-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="dc25b-186">このプロパティは、ステージングや運用など、さまざまな環境で実行されるアプリケーションの多様な構成をサポートします。</span><span class="sxs-lookup"><span data-stu-id="dc25b-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="dc25b-187">`ExternalConfigurationManager` オブジェクトは `BlobSettingsStore` オブジェクトに対して、すべての変更を照会するクエリを定期的に実行することもできます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="dc25b-188">次のコードでは、`StartMonitor` メソッドは、すべての変更を検出できる間隔で `CheckForConfigurationChanges` を呼び出し、前述のように `Changed` イベントを発生させます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

<span data-ttu-id="dc25b-189">`ExternalConfigurationManager` クラスは、以下のように `ExternalConfiguration` クラスによるシングルトン インスタンスとしてインスタンス化されます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

<span data-ttu-id="dc25b-190">次のコードは、_ExternalConfigurationStore.Cloud_ プロジェクトの `WorkerRole` クラスから引用されたものです。</span><span class="sxs-lookup"><span data-stu-id="dc25b-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="dc25b-191">アプリケーションが設定を読み取るために `ExternalConfiguration` クラスを使用する方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

<span data-ttu-id="dc25b-192">次のコードも `WorkerRole` クラスから引用されたものです。アプリケーションが構成イベントにサブスクライブする方法を示しています。</span><span class="sxs-lookup"><span data-stu-id="dc25b-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="dc25b-193">関連のあるパターンとガイダンス</span><span class="sxs-lookup"><span data-stu-id="dc25b-193">Related patterns and guidance</span></span>

- <span data-ttu-id="dc25b-194">このパターンを示すサンプルは [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) から入手できます。</span><span class="sxs-lookup"><span data-stu-id="dc25b-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
