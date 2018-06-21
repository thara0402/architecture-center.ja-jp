---
title: CQRS
description: 個別のインターフェイスを使用して、データを更新する操作とデータを読み取る操作を分離します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: c2832aa806909c6f0aab8b6345ffb8162eb59903
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/07/2018
ms.locfileid: "33811051"
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a>コマンド クエリ責務分離 (CQRS) パターン

[!INCLUDE [header](../_includes/header.md)]

個別のインターフェイスを使用して、データを更新する操作とデータを読み取る操作を分離します。 これにより、パフォーマンス、スケーラビリティ、セキュリティを最大化できます。 高い柔軟性でシステムの進化に経時的に対応し、更新コマンドによってドメイン レベルでマージ競合が発生するのを防ぐことができます。

## <a name="context-and-problem"></a>コンテキストと問題

従来のデータ管理システムでは、コマンド (データの更新) とクエリ (データの要求) は、1 つのデータ リポジトリ内のエンティティの同じセットに対して実行されます。 これらのエンティティは、リレーショナル データベース (SQL Server など) における 1 つ以上のテーブル内の行のサブセットである可能性があります。

これらのシステムでは通常、作成、読み取り、更新、削除 (CRUD) の操作はすべて、同じエンティティ表現に適用されます。 たとえば、顧客を表すデータ転送オブジェクト (DTO) は、データ アクセス層 (DAL) によってデータ ストアから取得され、画面に表示されます。 ユーザーが (データ バインドによって) DTO の一部のフィールドを更新すると、DTO は DAL によってデータ ストアに保存されます。 読み取り操作と書き込み操作に同じ DTO が使用されます。 次の図は、従来の CRUD アーキテクチャを示しています。

![従来の CRUD アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

従来の CRUD 設計は、限られたビジネス ロジックだけをデータ操作に適用する場合に適しています。 開発ツールで提供されるスキャフォールディング メカニズムにより、データ アクセス コードを非常に迅速に作成することができ、必要に応じてカスタマイズできます。

ただし、従来の CRUD アプローチには、いくつかの短所があります。

- 読み取りと書き込みのデータ表現が一致しないことがよくあります。具体的には、操作の一部としては必要ないものの、正しく更新しなければならない追加の列やプロパティなどです。

- 複数のアクターが同じデータ セットに対して操作を並列で実行するコラボレーション ドメインのデータ ストアでレコードがロックされている場合、データ競合のリスクがあります。 また、オプティミスティック ロックを使用している場合、同時更新による更新の競合のリスクもあります。 システムの複雑さとスループットの増加に伴って、これらのリスクも増加します。 さらに、従来のアプローチは、データ ストアとデータ アクセス層への負荷、および情報を取得するために必要なクエリの複雑さによって、パフォーマンスに悪影響を及ぼす可能性があります。

- 各エンティティは読み取りと書き込みの両方の操作の対象となり、誤ったコンテキストでデータが公開されることがあるため、セキュリティとアクセス許可の管理が複雑化する可能性があります。

> CRUD アプローチの制限の詳細については、[余裕がある場合にのみ CRUD を使用する方法](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/)に関するページをご覧ください。

## <a name="solution"></a>解決策

コマンド クエリ責務分離 (CQRS) は、別のインターフェイスを使用することで、データを更新する操作 (コマンド) からデータを読み取る操作 (クエリ) を分離するパターンです。 つまり、クエリと更新にそれぞれ異なるデータ モデルを使用します。 絶対条件ではありませんが、次の図に示すようにモデルを分離できます。

![基本的な CQRS アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

CQRS ベースのシステムでデータのクエリと更新の分離モデルを使用すると、CRUD ベースのシステムで使用される単一データ モデルと比べて、設計と実装が簡単になります。 ただし、CRUD 設計とは異なり、CQRS コードはスキャフォールディング メカニズムを使用して自動的に生成できないという短所があります。

データを読み取るためのクエリ モデルとデータを書き込むための更新モデルは、おそらく SQL ビューを使用するか、即時にプロジェクションを生成することにより、同じ物理ストアにアクセスできます。 ただし、パフォーマンス、スケーラビリティ、セキュリティを最大化するため、次の図に示すように、データを異なる物理ストアに分けるのが一般的です。

![読み取りストアと書き込みストアを分けた CQRS アーキテクチャ](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

書き込みストアの読み取り専用レプリカを読み取りストアにすることも、読み取りストアと書き込みストアをまったく別の構造にすることもできます。 読み取りストアの読み取り専用レプリカを複数使用すると、読み取り専用レプリカがアプリケーション インスタンスの近くに配置されている分散シナリオでは特に、クエリのパフォーマンスとアプリケーション UI の応答性が大幅に向上します。 一部のデータベース システム (SQL Server) には、可用性を最大化するためのフェールオーバー レプリカなどの追加機能があります。

読み取りストアと書き込みストアを分離することにより、それぞれの負荷に合わせて適切にスケーリングすることもできます。 たとえば、読み取りストアには通常、書き込みストアよりはるかに高い負荷が発生します。

クエリ/読み取りモデルに非正規化データが含まれている場合 (「[具体化されたビュー パターン](materialized-view.md)」を参照)、パフォーマンスが最大化されるのは、アプリケーションの各ビューのデータを読み取るときと、システム内のデータを照会する実行するときです。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

- データ ストアを読み取り操作用と書き込み操作用という別々の物理ストアに分けると、システムのパフォーマンスとセキュリティは向上する一方で、回復性と最終的な整合性の観点からは複雑さが増す可能性があります。 読み取りモデル ストアは、書き込みモデル ストアへの変更を反映させるために更新する必要がありますが、ユーザーが古い読み取りデータに基づいた要求をいつ発行したのかを検出するのは容易ではなく、この場合、操作を完了できません。

    > 最終的な整合性の詳細については、「[Data Consistency Primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)」を参照してください。

- システムの最も重要な、限られたセクションに CQRS を適用することを検討してください。

- 最終的な整合性をデプロイする一般的な方法は、イベント ソーシングと CQRS を併用することです。これにより、書き込みモデルは、コマンドの実行によって発生するイベントの追加専用ストリームになります。 これらのイベントは、読み取りモデルとして機能する具体化されたビューの更新に使用されます。 詳細については、「[Event Sourcing and CQRS (イベント ソーシングと CQRS)](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs)」を参照してください。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の状況で使用します。

- 同じデータに対して複数の操作が並列で実行されるコラボレーション ドメイン。 CQRS を使用すると、同じ種類のデータに見えるものを更新する場合でも、ドメイン レベルでマージの競合を最小化するのに十分な細分性でコマンドを定義できます (発生したすべての競合をコマンドでマージできます)。

- 一連の手順として、または複雑なドメイン モデルを使用して、複雑なプロセスがユーザーにされるタスクベースのユーザー インターフェイス。 既にドメインベースの設計 (DDD) 手法を使い慣れているチームにも有用です。 書き込みモデルには、ビジネス ロジック、入力の検証、ビジネスの検証を使用する完全なコマンド処理スタックがあり、その書き込みモデル内で各集計 (データ変更の単位として処理される関連オブジェクトの各クラスター) についてすべてが一貫していることを保証します。 読み取りモデルにはビジネス ロジックや検証スタックはなく、ビュー モデルで使用する DTO を返すだけです。 読み取りモデルは、最終的には書き込みモデルと一致します。

- データ読み取りのパフォーマンスを、データ書き込みのパフォーマンスとは別に細かく調整する必要があるシナリオ (特に、読み取り/書き込みの比率が非常に高く、水平スケーリングが必要な場合)。 たとえば、多くのシステムで、読み取り操作の回数は書き込み操作の回数の数倍になります。 これに対応できるように、読み取りモデルをスケール アウトし、書き込みモデルは 1 つまたは少数のインスタンスでのみ実行することを検討してください。 書き込みモデルのインスタンス数を少なくすることは、マージ競合の発生を最小化するうえでも役立ちます。

- 1 つの開発者チームが書き込みモデルの一部である複雑なドメイン モデルに注力し、もう 1 つのチームが読み取りモデルとユーザー インターフェイスに注力できるシナリオ。

- システムが時間の経過に伴って進化し、モデルの複数のバージョンを含むようになることが予測されるシナリオや、ビジネス ルールが定期的に変更されるシナリオ。

- 他のシステムとの統合 (特にイベント ソーシングとの組み合わせ)。この場合、1 つのサブシステムの一時的なエラーは、他のサブシステムの可用性に影響しません。

次の状況では、このパターンはお勧めしません。

- ドメインやビジネス ルールが単純な場合。

- 単純な CRUD スタイルのユーザー インターフェイスと、関連するデータ アクセス操作で十分な場合。

- システム全体にまたがる実装の場合。 CQRS が役立つ全体的なデータ管理シナリオに特有のコンポーネントもあるものの、必要ない場合には煩雑さが増してしまう可能性があります。

## <a name="event-sourcing-and-cqrs"></a>イベント ソーシングと CQRS

CQRS パターンは、イベント ソーシング パターンと共によく使用されます。 CQRS ベースのシステムは、個別の読み取りデータ モデルと書き込みデータ モデルを使用します。これらはそれぞれ関連するタスクに合わせて調整されており、多くの場合、物理的に分離されたストアに存在します。 [イベント ソーシング](event-sourcing.md) パターンと共に使用される場合、イベントのストアは書き込みモデルであり、公式の情報ソースです。 CQRS ベースのシステムの読み取りモデルは、データの具体化されたビュー (通常は高度に非正規化されたビュー) を提供します。 これらのビューは、アプリケーションのインターフェイスとディスプレイの要件に合わせて調整されており、ディスプレイとクエリの両方のパフォーマンスを最大化するのに役立ちます。

ある時点での実際のデータではなく、イベントのストリームを書き込みストアとして使用することにより、単一の集計での更新の競合を回避し、パフォーマンスとスケーラビリティを最大化します。 イベントを使用して、読み取りストアへのデータ入力に使用されるデータの具体化されたビューを非同期的に生成できます。

イベント ストアは公式の情報ソースであるため、システムが進化したり読み取りモデルの変更が必要になったりした場合に、具体化されたビューを削除し、過去のすべてのイベントを再生して最新の状態の新しい表現を作成することができます。 具体化されたビューは、実質的にはデータの持続的な読み取り専用キャッシュです。

CQRS とイベント ソーシング パターンを組み合わせて使用する場合、次の点を考慮してください。

- 書き込みストアと読み取りストアが分離しているすべてのシステムと同様に、このパターンに基づくシステムは、最後の段階にならないと一貫性が確保されません。 イベントの生成とデータ ストアの更新の間には、いくらかの遅延があります。

- イベントを開始および処理し、クエリや読み取りモデルで必要な適切なビューやオブジェクトをアセンブルまたは更新するようにコードを作成する必要があるため、このパターンでは複雑さが増します。 CQRS パターンをイベント ソーシング パターンと併用すると複雑さが増すため、実装が難しくなる可能性があり、システム設計に別のアプローチが必要になります。 ただし、イベント ソーシングを使用するとドメインのモデル化が容易になります。また、データの変更の目的が保持されるため、ビューの再構築や新規作成も容易になります。

- 特定のエンティティまたはエンティティのコレクションのイベントを再生または処理することにより、データの読み取りモデルまたはプロジェクションで使用する具体化されたビューを生成すると、大量の処理時間とリソース使用量が必要になる可能性があります。 これは特に、長期にわたる値の合計や解析が必要な場合に当てはまります。関連するすべてのイベントの検証が必要な場合があるためです。 この問題を解決するには、スケジュールされた間隔 (発生した特定のアクションの合計数、エンティティの現在の状態など) でデータのスナップショットを実装します。

## <a name="example"></a>例

次のコードは、読み取りモデルと書き込みモデルに異なる定義を使用する CQRS 実装の例から抽出したものです。 モデル インターフェイスは、基になるデータ ストアの機能に影響しません。また、進化することができ、インターフェイスどうしが分離しているため個別に微調整もできます。

次のコードは、読み取りモデルの定義を示しています。

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

ユーザーは製品を評価することができます。 そのためには、次のコードに示すように、アプリケーション コードで `RateProduct` コマンドを使用します。

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

システムは `ProductsCommandHandler` クラスを使用して、アプリケーションから送信されたコマンドを処理します。 クライアントは通常、キューなどのメッセージング システムを使用して、ドメインにコマンドを送信します。 コマンド ハンドラーはこれらのコマンドを受け入れ、ドメイン インターフェイスのメソッドを呼び出します。 各コマンドの細分性は、要求の競合が発生する可能性が少なくなるように設計されています。 次のコードは、`ProductsCommandHandler` クラスのアウトラインを示しています。

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

次のコードは、書き込みモデルからの `IProductsDomain` インターフェイスを示しています。

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

ドメイン内で意味を持つメソッドが `IProductsDomain` インターフェイスにどのように含まれているかにも注意してください。 通常、これらのメソッドは CRUD 環境では `Save` や `Update` などの汎用的な名前を持ち、DTO が唯一の引数です。 CQRS アプローチは、この組織のビジネスと在庫管理システムのニーズを満たすように設計できます。

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

このパターンを実装する場合、次のパターンとガイダンスが役に立ちます。

- CQRS と他のアーキテクチャ スタイルとの比較については、「[アーキテクチャ スタイル](/azure/architecture/guide/architecture-styles/)」と「[CQRS アーキテクチャのスタイル](/azure/architecture/guide/architecture-styles/cqrs)」を参照してください。

- [Data consistency primer (データ整合性入門)](https://msdn.microsoft.com/library/dn589800.aspx)。 CQRS パターン使用時に読み取りデータ ストアと書き込みデータ ストアの間の最終的な整合性が原因で通常発生する問題と、これらの問題の解決方法について説明します。

- [データのパーティション分割のガイダンス](https://msdn.microsoft.com/library/dn589795.aspx)。 CQRS パターンで使用される読み取りデータ ストアと書き込みデータ ストアを、個別に管理およびアクセス可能なパーティションに分割して、スケーラビリティの向上、競合の削減、パフォーマンスの最適化を図る方法について説明します。

- [イベント ソーシング パターン](event-sourcing.md)。 イベント ソーシングを CQRS パターンと共に使用して、複雑なドメインでのタスクを簡略化しながら、パフォーマンス、スケーラビリティ、応答性を向上させる方法について、さらに詳しく説明します。 補正アクションを有効にできる完全な監査証跡と履歴を保持しながら、トランザクション データの整合性を提供する方法についても説明します。

- [具体化されたビュー パターン](materialized-view.md)。 CQRS 実装の読み取りモデルには、書き込みモデル データの具体化されたビューを含めることができます。また、読み取りモデルは具体化されたビューの生成に使用できます。

- パターンとプラクティスのガイド「[CQRS Journey (CQRS の旅)](http://aka.ms/cqrs)」。 具体的には、「[Introducing the Command Query Responsibility Segregation Pattern (コマンド クエリ責務分離パターンの概要)](https://msdn.microsoft.com/library/jj591573.aspx)」でパターンとそのパターンが役立つ状況について説明します。「[Epilogue: Lessons Learned (エピローグ: 得られた教訓)](https://msdn.microsoft.com/library/jj591568.aspx)」は、このパターンを使用したときに発生する問題の一部を理解するのに役立ちます。

- Martin Fowler の投稿「[CQRS](http://martinfowler.com/bliki/CQRS.html)」では、パターンの基本と他の有用なリソースへのリンクを紹介しています。

- [Greg Young の投稿](http://codebetter.com/gregyoung/)では、CQRS パターンのさまざまな側面について説明しています。
