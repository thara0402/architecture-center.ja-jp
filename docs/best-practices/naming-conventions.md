---
title: Azure リソースの名前付け規則
description: Azure リソースの名前付け規則。 仮想マシン、ストレージ アカウント、ネットワーク、仮想ネットワーク、サブネット、その他の Azure エンティティに名前を付ける方法
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 7a94e7b3a54f48a8b1996415e194ecacb4261399
ms.sourcegitcommit: fdcacbfdc77370532a4dde776c5d9b82227dff2d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/24/2018
ms.locfileid: "49962978"
---
# <a name="naming-conventions"></a>名前付け規則

[!INCLUDE [header](../_includes/header.md)]

この記事は、Azure リソースの名前付け規則と制限事項の概要、および名前付け規則に関する推奨事項のベースライン セットを示します。  これらの推奨事項は、各自のニーズを満たす独自の規則を作るうえでの叩き台として利用できます。

Microsoft Azure におけるどのリソースの名前の選択も、以下の理由から重要です。

* 後で名前を変更するのは困難である。
* 名前は、そのリソースの種類の要件を満たす必要がある。

一貫性のある名前付け規則を使用することで、リソースが見つけやすくなります。 また、ソリューション内のリソースのロールを表すこともできます。

名前付け規則で成功を収めるうえで鍵となるのは、アプリケーションおよび組織全体で規則を策定し、準拠していくことです。

## <a name="naming-subscriptions"></a>サブスクリプションへの名前付け
Azure サブスクリプションに名前を付ける際、詳細な名前にしておくと、各サブスクリプションのコンテキストと用途がはっきりわかるようになります。  多数のサブスクリプションが利用される環境で作業する場合は、共通の名前付け規則に従うことで、わかりやすさが増します。

サブスクリプションに名前を付ける際に推奨されるパターンは次のとおりです。

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* "Company" は、通常、各サブスクリプションで同じにします。 ただし、一部の企業には、組織構造内に子会社があります。 こうした企業は、中央の IT グループによって管理されている場合があります。 このような場合は、親会社の名前 (*Contoso*) と子会社の名前 (*Northwind*) の両方を指定することで区別できます。
* "Department" は組織内の名前で、個人グループが含まれます。 名前空間内でのこの項目は省略してもかまいません。
* "Product Line" は部署内で遂行される職務や製品の個別の名前です。 一般に、内部向けのサービスやアプリケーションでは省略してもかまいません。 ただし、簡単に区別および識別できる必要のある外部向けのサービスには使用することを強くお勧めします (課金レコードの明確な区別などのため)。
* "Environment" は、"Dev"、"QA"、"Prod" など、アプリケーションまたはサービスのデプロイ ライフサイクルを示す名前です。

| [会社] | 学科 | Product Line または Service | 環境 | フルネーム |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Production |Contoso SocialGaming AwesomeService Production |
| Contoso |SocialGaming |AwesomeService |Dev |Contoso SocialGaming AwesomeService Dev |
| Contoso |IT |InternalApps |Production |Contoso IT InternalApps Production |
| Contoso |IT |InternalApps |Dev |Contoso IT InternalApps Dev |

より大規模な企業でサブスクリプションを整理する方法の詳細については、「[Azure エンタープライズ スキャフォールディング - 規範的なサブスクリプション ガバナンス][scaffold]」を参照してください。

## <a name="use-affixes-to-avoid-ambiguity"></a>接辞による明確化

Azure のリソースに名前を付けるときは、リソースの種類とコンテキストを識別するために、一般的なプレフィックスまたはサフィックスを使用することをお勧めします。  種類、メタデータ、コンテキストに関する情報はいずれもプログラムで利用できますが、一般的な接辞を適用することで、視覚的に区別しやすくなります。  名前付け規則に接辞を導入する場合は、接辞を名前の先頭に付ける (プレフィックス) のか、末尾に付ける (サフィックス) のかを明確に指定することが重要です。

たとえば、計算エンジンをホストするサービスの名前として考えられる名前を 2 つ、次に示します。

* SvcCalculationEngine (プレフィックス)
* CalculationEngineSvc (サフィックス)

接辞では、特定のリソースを説明するさまざまな特徴を示すことができます。 一般的に使用される例を次に示します。

| 特徴 | 例 | メモ |
| --- | --- | --- |
| 環境 |dev、prod、QA |リソースの環境を識別 |
| Location |uw (米国西部)、ue (米国東部) |リソースの展開先のリージョンを識別 |
| インスタンス |01、02 |複数の名前付きインスタンスが存在するリソースの場合 (Web サーバーなど)。 |
| 製品またはサービス |service |リソースがサポートする製品、アプリケーション、サービスを識別 |
| Role |sql、web、messaging |関連付けられているリソースのロールを識別 |

企業またはプロジェクトの特定の名前付け規則を策定する際には、一般的な接辞のセットと接辞の位置 (サフィックスまたはプリフィックス) を選択することが重要です。

## <a name="naming-rules-and-restrictions"></a>名前付け規則と制限事項

Azure のリソースまたはサービスの種類ごとに、名前付けに関する制限とスコープのセットが強制され、名前付け規則またはパターンはすべて、必須の名前付け規則とスコープに従う必要があります。  たとえば、VM の名前は DNS 名にマップされますが (このため、Azure 全体で一意である必要があります)、VNET の名前のスコープは、それが作成されたリソース グループに制限されます。

一般的に、特殊文字 (`-` または `_`) は、名前の先頭または末尾には使用しないでください。 こうした文字があると、ほとんどの検証規則でエラーが発生します。

### <a name="general"></a>全般

| エンティティ | Scope (スコープ) | Length | 大文字小文字の区別 | 有効な文字 | 推奨パターン | 例 |
| --- | --- | --- | --- | --- | --- | --- |
|リソース グループ |サブスクリプション |1-90 |大文字と小文字は区別されない |[こちら](/rest/api/resources/resourcegroups/createorupdate)に記載されている正規表現と一致している英数字、アンダースコア、かっこ、ハイフン、ピリオド (末尾を除く)、および Unicode 文字。  |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|可用性セット |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、アンダースコア、ハイフン |`<service-short-name>-<context>-as` |`profx-sql-as` |
|タグ |関連付けられたエンティティ |512 (名前)、256 (値) |大文字と小文字は区別されない |英数字 |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>コンピューティング

| エンティティ | Scope (スコープ) | Length | 大文字小文字の区別 | 有効な文字 | 推奨パターン | 例 |
| --- | --- | --- | --- | --- | --- | --- |
|仮想マシン |リソース グループ |1 ～ 15 (Windows)、1 ～ 64 (Linux) |大文字と小文字は区別されない |英数字とハイフン |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Function App | グローバル |1 ～ 60 |大文字と小文字は区別されない |英数字とハイフン |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Azure の仮想マシンには、仮想マシン名とホスト名の 2 つの個別の名前があります。 ポータルで VM を作成するときは、ホスト名と仮想マシン リソース名の両方に同じ名前が使用されます。 上記の制限はホスト名を対象としています。 実際のリソース名の最大文字数は 64 文字です。

### <a name="storage"></a>Storage

| エンティティ | Scope (スコープ) | Length | 大文字小文字の区別 | 有効な文字 | 推奨パターン | 例 |
| --- | --- | --- | --- | --- | --- | --- |
|ストレージ アカウント名 (データ) |グローバル |3 ～ 24 |小文字 |英数字 |`<globally unique name><number>` (ストレージ アカウントの名前付けのために関数を使用して一意の GUID を計算) |`profxdata001` |
|ストレージ アカウント名 (ディスク) |グローバル |3 ～ 24 |小文字 |英数字 |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| コンテナー名 |ストレージ アカウント |3 ～ 63 |小文字 |英数字とハイフン |`<context>` |`logs` |
|BLOB 名 | コンテナー |1 ～ 1,024 |大文字小文字は区別される |任意の URL 文字 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|キュー名 |ストレージ アカウント |3 ～ 63 |小文字 |英数字とハイフン |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|テーブル名 | ストレージ アカウント |3 ～ 63 |大文字と小文字は区別されない |英数字 |`<service short name><context>` |`awesomeservicelogs` |
|ファイル名 | ストレージ アカウント |3 ～ 63 |小文字 | 英数字 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | グローバル |3 ～ 24 |小文字 | 英数字 |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>ネットワーク

| エンティティ | Scope (スコープ) | Length | 大文字小文字の区別 | 有効な文字 | 推奨パターン | 例 |
| --- | --- | --- | --- | --- | --- | --- |
|Virtual Network (VNet) |リソース グループ |2 ～ 64 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<service short name>-vnet` |`profx-vnet` |
|サブネット |親 VNet |2 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<descriptive context>` |`web` |
|ネットワーク インターフェイス |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<vmname>-nic<num>` |`profx-sql1-vm1-nic1` |
|ネットワーク セキュリティ グループ |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|ネットワーク セキュリティ グループの規則 |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<descriptive context>` |`sql-allow` |
|パブリック IP アドレス |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<vm or service name>-pip` |`profx-sql1-vm1-pip` |
|Load Balancer |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<service or role>-lb` |`profx-lb` |
|負荷分散規則の構成 |Load Balancer |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<descriptive context>` |`http` |
|Azure Application Gateway |リソース グループ |1 ～ 80 |大文字と小文字は区別されない |英数字、ハイフン、アンダースコア、ピリオド |`<service or role>-agw` |`profx-agw` |
|Traffic Manager プロファイル |リソース グループ |1 ～ 63 |大文字と小文字は区別されない |英数字、ハイフン、ピリオド |`<descriptive context>` |`app1` |

### <a name="containers"></a>Containers

| エンティティ | Scope (スコープ) | Length | 大文字小文字の区別 | 有効な文字 | 推奨パターン | 例 |
| --- | --- | --- | --- | --- | --- | --- |
|Container Registry | グローバル |5 ～ 50 |大文字と小文字は区別されない | 英数字 |`<service short name>registry` |`app1registry` |


## <a name="organize-resources-with-tags"></a>タグによるリソースの整理

Azure Resource Manager では、コンテキストを識別し、自動化を合理化するために、任意のテキスト文字列を使用したエンティティへのタグ付けがサポートされています。  たとえば、`"sqlVersion"="sql2014ee"` タグでは、SQL Server 2014 Enterprise Edition を実行している VM を識別できます。 タグは、コンテキストを明確化するために、選択した名前付け規則と共に使用してください。

> [!TIP]
> タグには、リソース グループをまたいで適用できるため、異なるデプロイ間でエンティティどうしを関連付けることができるという利点もあります。

各リソースまたはリソース グループには、最大で **15** 個のタグを設定できます。 タグ名は 512 文字まで、タグ値は 256 文字までに制限されます。

リソースへのタグ付けの詳細については、[タグを使用した Azure リソースの整理](/azure/azure-resource-manager/resource-group-using-tags/)に関するページをご覧ください。

一般的なタグ付けの使用例は次のとおりです。

* **課金**: リソースをグループ化し、課金またはチャージバック コードに関連付けます。
* **サービスのコンテキストの識別**: 一般的な操作とグループ化のために、リソース グループ間でリソースのグループを識別します。
* **アクセス制御とセキュリティ コンテキスト**: ポートフォリオ、システム、サービス、アプリ、インスタンスなどに基づいて管理者のロールを識別します。

> [!TIP]
> タグは早い段階から頻繁に設定してください。  データを集め終えてから改良を加えるよりも、ベースラインのタグ付けのスキームを設けておいて、徐々に調整していく方が適切です。

以下に、一般的なタグ付けのアプローチの例を示します。

| タグ名 | キー | 例 | Comment (コメント) |
| --- | --- | --- | --- |
| 請求先/内部チャージバック ID |billTo |`IT-Chargeback-1234` |内部 I/O または課金コード |
| オペレーターまたは直接責任者 (DRI) |managedBy |`joe@contoso.com` |エイリアスまたは電子メール アドレス |
| プロジェクト名 |projectName |`myproject` |プロジェクトまたは製品ラインの名前 |
| プロジェクトのバージョン |projectVersion |`3.4` |プロジェクトまたは製品ラインのバージョン |
| 環境 |環境 |`<Production, Staging, QA >` |環境の識別子 |
| レベル |レベル |`Front End, Back End, Data` |レベルまたはロール/コンテキストの識別 |
| データ プロファイル |dataProfile |`Public, Confidential, Restricted, Internal` |リソースに格納されているデータの機密性 |

## <a name="tips-and-tricks"></a>ヒントとコツ

一部のリソースの種類については、名前付けと規則にさらに注意が必要になる場合があります。

### <a name="virtual-machines"></a>仮想マシン

特に、大規模なトポロジでは、仮想マシンの名前を工夫することで、各マシンのロールと用途の識別が容易になり、スクリプトを作成する際の予測可能性が高まります。

### <a name="storage-accounts-and-storage-entities"></a>ストレージ アカウントとストレージ エンティティ

ストレージ アカウントには、VM のディスクのサポートと、BLOB、キュー、テーブルへのデータの格納という 2 つの主要な用途があります。  VM ディスクに使用されるストレージ アカウントについては、親 VM の名前に関連付けるという名前付け規則に従う必要があります (また、ハイエンド VM SKU 向けに複数のストレージ アカウントが必要になる可能性があるため、数値のサフィックスも適用してください)。

> [!TIP]
> ストレージ アカウントについては、データ用であるか、ディスク用であるかにかかわらず、複数のストレージ アカウントを活用できるような名前付け規則に従う必要があります (つまり、必ず数値のサフィックスを使用してください)。

Azure ストレージ アカウントの BLOB データにアクセスするためのカスタム ドメイン名を構成できます。 Blob service の既定のエンドポイントは https://\<name\>.blob.core.windows.net です。

ただし、カスタム ドメイン (www.contoso.com など) をストレージ アカウントの BLOB エンドポイントにマップしている場合、ユーザーはそのドメインを使って、ストレージ アカウントの BLOB データにもアクセスできます。 たとえば、カスタム ドメイン名を使用すると、`https://mystorage.blob.core.windows.net/mycontainer/myblob` に、`https://www.contoso.com/mycontainer/myblob` としてアクセスできます。

この機能の構成の詳細については、「[BLOB ストレージ エンドポイントのカスタム ドメイン名の構成](/azure/storage/storage-custom-domain-name/)」を参照してください。

BLOB、コンテナー、テーブルへの名前付けの詳細については、次の一覧を参照してください。

* [コンテナー、BLOB、メタデータの名前付けと参照](https://msdn.microsoft.com/library/dd135715.aspx)
* [キューとメタデータの名前付け](https://msdn.microsoft.com/library/dd179349.aspx)
* [テーブルの名前付け](https://msdn.microsoft.com/library/azure/dd179338.aspx)

BLOB 名には任意の文字の組み合わせを含めることができますが、URL の予約文字は適切にエスケープする必要があります。 BLOB 名の末尾をピリオド (.)、スラッシュ (/)、それらの連続または組み合わせたものにしないでください。 通常、スラッシュは、 **仮想** ディレクトリの区切り記号です。 BLOB 名では、バックスラッシュ (\\) を使わないでください。 クライアント API では許容されるものの、正常にハッシュできず、署名が一致しなくなります。

ストレージ アカウントまたはコンテナーの名前は、作成後に変更することはできません。 新しい名前を使用する場合は、削除したうえで、新たに作成する必要があります。

> [!TIP]
> 新しいサービスまたはアプリケーションの開発に着手する前に、すべてのストレージ アカウントと種類の名前付け規則を策定することをお勧めします。

<!-- links -->

[scaffold]: /azure/architecture/cloud-adoption/appendix/azure-scaffold
