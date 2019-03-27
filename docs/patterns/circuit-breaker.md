---
title: サーキット ブレーカー パターン
titleSuffix: Cloud Design Patterns
description: リモート サービスまたはリソースとの接続時の修正に要する時間が一定しないエラーを処理します。
keywords: 設計パターン
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 7cc84b3c14ea277aa82643f3141f0693ec702a49
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249317"
---
# <a name="circuit-breaker-pattern"></a>サーキット ブレーカー パターン

リモート サービスまたはリソースとの接続時に、復旧に要する時間が一定しないエラーを処理します。 これにより、アプリケーションの安定性と回復性を向上させることができます。

## <a name="context-and-problem"></a>コンテキストと問題

分散環境では、リモートのリソースやサービスへの呼び出しは、低速なネットワーク接続、タイムアウト、または、リソースが過剰にコミットされたり一時的に使用できなくなったりするといった一時的なエラーのために失敗する可能性があります。 このようなエラーは、通常は短時間で自動的に修正され、堅牢なクラウド アプリケーションでは、[再試行パターン](./retry.md)などの方法を使用して、これらのエラーを処理する準備が整っている必要があります。

ただし、エラーが予期しないイベントによるものであることや、修正にかなり時間がかかる可能性があることも考えられます。 このようなエラーの重大度は、部分的な接続の損失からサービスの完全な障害に至るまで、さまざまな可能性があります。 このような状況では、成功する可能性の低い操作を、アプリケーションが連続して無意味に再試行することがあります。そうではなくアプリケーションは、操作が失敗したことをすぐに受け入れて、そのエラーを適切に処理する必要があります。

さらに、サービスが非常にビジーな状態の場合は、システムの一部分のエラーが障害の連鎖につながる可能性があります。 たとえば、サービスを呼び出す操作は、タイムアウトを実装し、サービスがこの期間内に応答できない場合はエラー メッセージを返信するよう構成することもできます。 ただし、この方法では、タイムアウト期間が過ぎるまで、同じ操作に対する多数の同時要求がブロックされる可能性があります。 これらのブロックされた要求が、重要なシステム リソース (メモリ、しきい値、データベース接続など) をとどめてしまう可能性があります。 その結果、これらのリソースが使い尽くされて、同じリソースを使用する必要がある、関連性のない可能性のあるシステムの他の部分の障害を引き起こす可能性があります。 このような場合は、操作がすぐに失敗し、成功する可能性がある場合はサービスの呼び出しを試みるだけのほうが望ましいでしょう。 短いタイムアウトを設定すると、この問題を解決しやすくなることもあります。ただし、最終的にサービスへの要求が成功するとしても、ほとんど常に操作が失敗するような短いタイムアウトにすべきではないことに注意してください。

## <a name="solution"></a>解決策

サーキット ブレーカー パターンは、Michael Nygard 氏がその著書である「[Release It!](https://pragprog.com/book/mnee/release-it)」によって広めたもので、アプリケーションが失敗する可能性のある操作を繰り返し試行するのを防ぐことができます。 障害が長期間にわたると判断している間に、エラーが修正されたり CPU サイクルが浪費されたりするのを待つことなく、続行することができます。 サーキット ブレーカー パターンでは、エラーが解決されたかどうかをアプリケーションで検出することもできます。 問題が解決済みのように見える場合、アプリケーションは操作の呼び出しを試みることができます。

> サーキット ブレーカー パターンの目的は、再試行パターンとは異なります。 再試行パターンでは、アプリケーションが成功を見込んで操作を再試行することができます。 サーキット ブレーカー パターンは、失敗する可能性がある操作をアプリケーションが実行しないようにします。 アプリケーションは、サーキット ブレーカーによって操作を呼び出す再試行パターンを使用することで、これら 2 つのパターンを組み合わせることができます。 ただし、再試行ロジックは、サーキット ブレーカーによって返されるすべての例外から大きな影響を受け、エラーが一時的なものではないことが示されると、再試行回数を破棄します。

サーキット ブレーカーは、失敗する可能性のある操作のプロキシとして機能します。 プロキシは、最近発生した障害の数を監視し、その情報を使用して、操作を続行できるか、すぐに例外を返すだけにするかを決定します。

プロキシは、電路遮断器の機能を模倣する、次の状態のステート マシンとして実装できます。

- **クローズド**:アプリケーションからの要求は、操作にルーティングされます。 プロキシは最近のエラーの数のカウントを保持し、操作への呼び出しが成功しなかった場合、プロキシはこのカウントを増分します。 最近のエラーの数が指定された期間内に指定されたしきい値を超えると、プロキシは**オープン**状態になります。 プロキシはこの時点でタイムアウト タイマーを開始し、このタイマーの期限が切れると、プロキシは**ハーフオープン**状態になります。

    > タイムアウト タイマーの目的は、エラーの原因となった問題を解決するための時間をシステムに与えて、その後でアプリケーションが操作の実行を再度試みられるようにすることです。

- **未解決**: アプリケーションからの要求はすぐに失敗し、アプリケーションに例外が返されます。

- **ハーフオープン**:アプリケーションからの限られた数の要求が、操作のパス スルーと呼び出しを許可されます。 これらの要求が成功した場合、以前にエラーの原因となった障害は既に修正されている見なされ、サーキット ブレーカーは**クローズド**状態に切り替わります (エラー カウンターはリセットされます)。 どの要求が失敗しても、サーキット ブレーカーはエラーがまだ存在していると見なすため、**オープン**状態に戻り、障害から復旧するためのさらに長い時間をシステムに与えるために、タイムアウト タイマーを再起動します。

    > **ハーフオープン**状態は、復旧中のサービスに突然大量の要求が送信されないようにするために役立ちます。 サービスの復旧中は、復旧が完了するまでは限られた量の要求に対応できますが、復旧の進行中は大量の作業によってサービスがタイムアウトになったり、再び失敗したりする可能性があります。

![サーキット ブレーカーの状態](./_images/circuit-breaker-diagram.png)

この図では、**クローズド**状態で使用されるエラー カウンターは時間ベースです。 これは定期的な間隔で自動的にリセットされます。 これにより、一時的なエラーが発生した場合に、サーキット ブレーカーが**オープン**状態に入らないようにすることができます。 サーキット ブレーカーを**オープン**状態にトリップするエラーのしきい値に達するのは、指定された期間の間に指定された数のエラーが発生したときにのみです。 **ハーフオープン**状態で使用されるカウンターは、操作の呼び出しに成功した試行の数を記録します。 サーキット ブレーカーは、指定された数の連続した操作の呼び出しが成功した後、**クローズド**状態に戻ります。 呼び出しが失敗すると、サーキット ブレーカーは**オープン**状態に入り、次に**ハーフオープン**状態に入ったときに成功カウンターがリセットされます。

> システムの復旧は、障害が発生したコンポーネントを復元または再起動するか、ネットワーク接続を修復することで、外部的に処理される可能性があります。

サーキット ブレーカー パターンにより、エラーからのシステムの復旧中の安定性が保たれ、パフォーマンスに対する影響が最小限に抑えられます。 これにより、失敗する可能性がある操作の要求を、操作がタイムアウトになるまで待機したり返さないようにしたりするのではなく、すぐに拒否することによって、システムの応答時間を維持することができます。 状態が変化するたびにサーキット ブレーカーでイベントが発生する場合は、この情報を使用して、サーキット ブレーカーによって保護されているシステムの一部の正常性を監視したり、サーキット ブレーカーが**オープン**状態にトリップするときに管理者に警告したりできます。

このパターンはカスタマイズ可能で、可能性のあるエラーの種類に応じて適用できます。 たとえば、サーキット ブレーカーにタイムアウト タイマーの増加を適用できます。 サーキット ブレーカーを最初の数秒間**オープン**状態にし、その後でエラーが解決されていない場合は、タイムアウトを数分に延ばすといったこともできます。 場合によっては、エラーを返し例外を発生させる**オープン**状態ではなく、アプリケーションにとって意味がある既定値を返すほうが有益な場合もあります。

## <a name="issues-and-considerations"></a>問題と注意事項

このパターンの実装方法を決めるときには、以下の点に注意してください。

**例外処理**。 サーキット ブレーカーによって操作を呼び出すアプリケーションは、操作が使用不可になった場合に発生する例外を処理するよう準備する必要があります。 例外の処理方法はアプリケーション固有になります。 たとえば、アプリケーションは、その機能を一時的に低下させたり、同じタスクの実行や同じデータの取得を試みる代替の操作を呼び出したり、ユーザーに例外を報告して後でもう一度やり直すよう求めたりすることがあります。

**例外の種類**。 要求は多くの理由で失敗する可能性があり、エラーによっては他のエラーよりもより重大な種類のエラーを示すものもあります。 たとえば、リモート サービスがクラッシュし、回復するのに数分かかるために要求が失敗することも、一時的に過負荷になっているサービスによるタイムアウトのために要求が失敗することもあります。 サーキット ブレーカーは、発生する例外の種類を確認し、その例外の性質に応じて戦略を調整できる場合があります。 たとえば、完全に使用不可になっているサービスによるエラーの数と比べると、サーキット ブレーカーを**オープン**状態にトリップするにはより多くのタイムアウト例外が必要な場合があります。

**ログの記録**。 サーキット ブレーカーは、管理者が操作の正常性を監視できるよう、すべての失敗した要求 (および成功した可能性のある要求) のログを記録します。

**回復性**。 サーキット ブレーカーは、保護する操作の、可能性の高い復旧パターンに一致するよう構成してください。 たとえば、サーキット ブレーカーが長期間**オープン**状態のままの場合は、エラーの原因が解決されているとしても、例外が発生する可能性があります。 同様に、**オープン**状態から**ハーフオープン**状態に切り替えるのが早すぎると、サーキット ブレーカーが変動し、アプリケーションの応答時間を短くすることがあります。

**失敗した操作のテスト**。 **オープン**状態では、タイマーを使用して**ハーフオープン**状態に切り替えるタイミングを決定する代わりに、サーキット ブレーカーでリモート サービスまたはリソースに定期的に ping を実行して、もう一度使用可能になるかどうかを確認できます。 この ping は、以前に失敗した操作の呼び出しを試みる形で行われることも、「[正常性エンドポイント監視パターン](./health-endpoint-monitoring.md)」に記載されている、サービスの正常性のテスト専用のリモート サービスによって提供される、特殊な操作を使用することもあります。

**手動オーバーライド**。 障害が発生する操作の復旧時間に極端な幅があるシステムでは、手動リセットのオプションを用意して、管理者がサーキット ブレーカーをクローズして失敗カウンターをリセットできるようにすることをお勧めします。 同様に、サーキット ブレーカーで保護されている操作が一時的に使用できなくなる場合は、管理者が強制的にサーキット ブレーカーを**オープン**状態にし、タイムアウト タイマーを再起動することもできます。

**コンカレンシー**。 アプリケーションの多数の同時実行インスタンスが、同じサーキット ブレーカーにアクセスすることもできます。 この実装は、同時要求をブロックしたり、操作へのそれぞれの呼び出しに過剰なオーバーヘッドを加えたりしません。

**リソースの区別**。 複数の基になる独立したプロバイダーがある可能性がある場合、1 つの種類のリソースに単一のサーキット ブレーカーを使用するときは注意してください。 たとえば、複数のシャードが含まれているデータ ストアでは、あるシャードで一時的な問題が発生しているときに、別のシャードには完全にアクセス可能であることがあります。 このようなシナリオのエラー応答が統合されると、アプリケーションは、エラーの可能性が高いときでも一部のシャードへのアクセスを試みる一方、他のシャードへのアクセスは、成功する可能性があってもブロックすることがあります。

**高速なサーキット ブレーク**。 エラー応答には、サーキット ブレーカーをすぐにトリップさせ、最短時間トリップしたままのするのに十分な情報が含まれていることがあります。 たとえば、過負荷になっている共有リソースからのエラー応答で、即時の再試行が推奨されず、代わりに、アプリケーションが数分後にもう一度試みることを示されることがあります。

> [!NOTE]
> サービスは、クライアントが調整中の場合は HTTP 429 (要求が多すぎます) を返し、サービスが現在使用可能でない場合は HTTP 503 (サービス利用不可) を返すことがあります。 応答には、予想される遅延時間などの追加情報を含めることができます。

**失敗した要求の再現**。 **オープン**状態では、単にすぐに失敗させるのではなく、サーキット ブレーカーでジャーナルへの各要求の詳細を記録して、リモート リソースやサービスが使用可能になったときにこれらの要求が再生されるように準備することもできます。

**外部サービスでの不適切なタイムアウト**。 サーキット ブレーカーは、タイムアウト時間が長く構成されている外部のサービスで失敗する操作からは、アプリケーションを完全に保護できない場合があります。 タイムアウト時間が長すぎると、操作が失敗したことをサーキット ブレーカーが示す前に、サーキット ブレーカーを実行するスレッドが長期間ブロックされることがあります。 この時点では、他の多くのアプリケーション インスタンスもサーキット ブレーカーによってサービスを呼び出そうとして、すべてが失敗するまで多数のスレッドを占有することがあります。

## <a name="when-to-use-this-pattern"></a>このパターンを使用する状況

このパターンは次の目的で使用します。

- この操作が失敗する可能性が高い場合に、アプリケーションがリモート サービスを呼び出そうとしたり、共有リソースにアクセスしようとしたりしないようにするため。

このパターンは次の場合は推奨されません。

- メモリ内データ構造など、アプリケーションのローカルのプライベート リソースへのアクセスを処理する場合。 この環境では、サーキット ブレーカーを使用すると、システムにオーバーヘッドが加わることがあります。
- 例外をアプリケーションのビジネス ロジックで処理する代わりとして。

## <a name="example"></a>例

Web アプリケーションでは、一部のページに外部サービスから取得されたデータが設定されます。 システムが実装しているキャッシュが最小限の場合、これらのページへのほとんどのヒットで、サービスへのラウンド トリップが発生します。 Web アプリケーションからサービスへの接続は、タイムアウト期間 (通常 60 秒) で構成され、サービスが時間内に応答しない場合は、各 web ページ内のロジックがサービスを使用不可と見なし、例外をスローします。

ただし、サービスが失敗し、システムが非常にビジー状態な場合は、例外が発生するまで最大 60 秒間、ユーザーを強制的に待機させます。 最終的にはメモリ、接続、スレッドなどのリソースが使い果たされ、他のユーザーは、サービスからデータを取得するページにアクセスしていない場合でもシステムに接続できなくなります。

Web サーバーをさらに追加して負荷分散を実装することでシステムをスケーリングすると、リソースが使い果たされるのは遅れますが、ユーザーの要求は応答しないままになり、Web サーバーは最終的にリソース不足になるため、問題は解決しません。

サービスに接続し、サーキット ブレーカーでデータを取得するロジックをラップすれば、この問題が解決して、サービスのエラーをより効率よく処理できる場合があります。 それでもユーザーの要求は失敗しますが、より速く失敗し、リソースはブロックされません。

`CircuitBreaker` クラスは、次のコードに示されている `ICircuitBreakerStateStore` インターフェイスを実装するオブジェクトに、サーキット ブレーカーに関する状態情報を保持します。

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

`State` プロパティは、サーキット ブレーカーの現在の状態を示し、`CircuitBreakerStateEnum` 列挙型の定義に従って**オープン**、**ハーフオープン**、または**クローズド**のいずれかになります。 `IsClosed` プロパティは、サーキット ブレーカーがクローズドの場合は true ですが、オープンまたはハーフオープンの場合は false になります。 `Trip` メソッドは、サーキット ブレーカーの状態をオープン状態に切り替えて、例外が発生した日時と共に、状態の変化の原因となった例外を記録します。 `LastException` および `LastStateChangedDateUtc` プロパティはこの情報を返します。 `Reset` メソッドはサーキット ブレーカーをクローズし、`HalfOpen` メソッドはサーキット ブレーカーをハーフオープンに設定します。

この例の `InMemoryCircuitBreakerStateStore` クラスには、`ICircuitBreakerStateStore` インターフェイスの実装が含まれています。 `CircuitBreaker` クラスは、このクラスのインスタンスを作成して、サーキット ブレーカーの状態を保持します。

`CircuitBreaker` クラスの `ExecuteAction` メソッドは、`Action` デリゲートとして指定された操作をラップします。 サーキット ブレーカーがクローズドの場合は、`ExecuteAction` が `Action` デリゲートを呼び出します。 操作が失敗すると、例外ハンドラーが `TrackException` を呼び出し、それによってサーキット ブレーカーの状態がオープンに設定されます。 次のコード例はこのフローを示しています。

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

次の例では、サーキット ブレーカーがクローズドではない場合に実行されるコード (前の例で省略されているもの) を示します。 これは、サーキット ブレーカーが `CircuitBreaker` クラスのローカルの `OpenToHalfOpenWaitTime` フィールドで指定されている時間より長い期間の間オープンだったかどうかをチェックします。 このような場合、`ExecuteAction` メソッドは、サーキット ブレーカーをハーフオープンに設定してから、`Action` デリゲートによって指定されている操作を実行しようとします。

操作が成功すると、サーキット ブレーカーはクローズド状態にリセットされます。 操作が失敗すると、オープン状態に再度トリップされて例外が発生した時刻が更新されるため、サーキット ブレーカーは、さらに長い時間待機してから操作の実行を再度試みます。

サーキット ブレーカーが `OpenToHalfOpenWaitTime` 値未満の短い時間しかオープンでなかった場合、`ExecuteAction` メソッドは単に `CircuitBreakerOpenException` 例外をスローし、サーキット ブレーカーがオープン状態に遷移した原因となったエラーを返します。

さらに、ロックを使用して、サーキット ブレーカーがハーフオープンの間に操作への同時呼び出しの実行を試みないようにします。 同時に操作を呼び出す試みは、サーキット ブレーカーがオープンであるかのように処理され、後述のように例外で失敗します。

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
        }
        catch (Exception ex)
        {
          // If there's still an exception, trip the breaker again immediately.
          this.stateStore.Trip(ex);

          // Throw the exception so that the caller knows which exception occurred.
          throw;
        }
        finally
        {
          if (lockTaken)
          {
            Monitor.Exit(halfOpenSyncObject);
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

`CircuitBreaker` オブジェクトを使用して操作を保護するために、アプリケーションは `CircuitBreaker` クラスのインスタンスを作成し、実行する操作をパラメーターとして指定して `ExecuteAction` メソッドを呼び出します。 サーキット ブレーカーがオープンであるために操作が失敗する場合は、`CircuitBreakerOpenException` 例外をキャッチするようにアプリケーションを準備する必要があります。 次に例を示します。

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a>関連のあるパターンとガイダンス

このパターンを実装する場合は、次のパターンも役に立つことがあります。

- [再試行パターン](./retry.md)。 失敗した操作を透過的に再試行することで、サービスまたはネットワーク リソースに接続しようとする際に、予測される一時的な障害をアプリケーションがどのように処理できるかを説明しています。

- [正常性エンドポイントの監視パターン](./health-endpoint-monitoring.md)。 サーキット ブレーカーは、サービスによって公開されるエンドポイントに要求を送信することで、サービスの正常性をテストできることがあります。 サービスは、その状態を示す情報を返す必要があります。
