---
title: ダッシュボードを使用して Azure Databricks のメトリックを視覚化する
description: Grafana ダッシュボードをデプロイして Azure Databricks でのパフォーマンスを監視する方法
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 36fcd93f6ca757e8e750d0fcbbdf0311c08560b0
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887830"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a>ダッシュボードを使用して Azure Databricks のメトリックを視覚化する

この記事では、パフォーマンスの問題について Azure Databricks ジョブを監視するように Grafana ダッシュ ボードを設定する方法について説明します。

[Azure Databricks](/azure/azure-databricks/) は、ビッグ データ分析ソリューションと人工知能 (AI) ソリューションの高速な開発とデプロイを容易にする、高速、強力、かつ協調的な [Apache Spark](https://spark.apache.org/) ベースの分析サービスです。 監視は、実稼働環境で Azure Databricks ワークロードを運用するうえで重要な構成要素です。 最初の手順は、分析のためにメトリックをワークスペースに収集することです。 Azure では、ログ データを管理するための最良のソリューションは [Azure Monitor](/azure/azure-monitor/) です。 Azure Databricks は、Azure Monitor へのログ データの送信をネイティブにサポートしていませんが、[この機能のためのライブラリ](https://github.com/mspnp/spark-monitoring)が [Github](https://github.com) にあります。

このライブラリにより、Azure Databricks サービスのメトリックおよび Apache Spark 構造化ストリーミング クエリ イベントのメトリックのログ記録が可能になります。 このライブラリを Azure Databricks クラスターに正常にデプロイしたら、さらに、実稼働環境の一部としてデプロイできる [Grafana](https://granfana.com) ダッシュボードのセットをデプロイすることができます。

![ダッシュボードのスクリーンショット](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a>前提条件

[Github リポジトリ](https://github.com/mspnp/spark-monitoring)を複製し、[デプロイ手順に従って](./configure-cluster.md)、Azure Log Analytics ワークスペースにログを送信するように Azure Databricks ライブラリ用に Azure Monitor のログ記録を構築して構成します。

## <a name="deploy-the-azure-log-analytics-workspace"></a>Azure Log Analytics ワークスペースをデプロイする

Azure Log Analytics ワークスペースをデプロイするには、以下の手順に従ってください。

1. `/perftools/deployment/loganalytics` ディレクトリに移動します。
1. **logAnalyticsDeploy.json** Azure Resource Manager テンプレートをデプロイします。 Resource Manager テンプレートのデプロイの詳細については、「[Azure Resource Manager テンプレートと Azure CLI を使用したリソースのデプロイ][rm-cli]」をご覧ください。 このテンプレートには、次のパラメーターがあります。

    * **location**:Log Analytics ワークスペースとダッシュボードがデプロイされるリージョン。
    * **serviceTier**:ワークスペースの価格レベル。 有効値の一覧については、[こちら][sku]を参照してください。
    * **dataRetention** (省略可能):Log Analytics ワークスペースにログ データが保持される日数。 既定値は 30 日間です。 価格レベルが `Free` の場合、データ保持は 7 日間でなければなりません。
    * **workspaceName** (省略可能):ワークスペースの名前。 指定しない場合、テンプレートが名前を生成します。

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

このテンプレートはワークスペースを作成し、さらに、ダッシュボードで使用される定義済みクエリのセットも作成します。

## <a name="deploy-grafana-in-a-virtual-machine"></a>Grafana を仮想マシンにデプロイする

Grafana はオープン ソース プロジェクトです。これをデプロイすると、Azure Monitor 用の Grafana プラグインを使用して、Azure Log Analytics ワークスペースに格納されている時間系列メトリックを視覚化することができます。 Grafana は仮想マシン (VM) 上で実行され、ストレージ アカウント、仮想ネットワーク、およびその他のリソースを必要とします。 Bitnami で認定された Grafana イメージと関連リソースを使用して仮想マシンをデプロイするには、以下の手順に従ってください。

1. Azure CLI を使用して、Grafana の Azure Marketplace イメージの使用条件に同意します。

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. GitHub リポジトリのローカル コピーの `/spark-monitoring/perftools/deployment/grafana` ディレクトリに移動します。
1. **grafanaDeploy.json** Resource Manager テンプレートを以下のようにデプロイします。

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

デプロイが完了すると、Grafana の Bitnami イメージが仮想マシンにインストールされます。

## <a name="update-the-grafana-password"></a>Grafana のパスワードを更新する

セットアップ プロセスの一環として、Grafana のインストール スクリプトは、**admin** ユーザー用の一時パスワードを出力します。 サインインするためには、この一時パスワードが必要です。 一時パスワードを取得するには、次の手順に従ってください。  

1. Azure ポータルにログインします。  
1. リソースがデプロイされたリソース グループを選択します。
1. Grafana がインストールされた VM を選択します。 デプロイ テンプレートで既定のパラメーター名を使用した場合、VM 名は **sparkmonitoring-vm-grafana** で始まります。
1. **[サポート + トラブルシューティング]** セクションで **[ブート診断]** をクリックして、[ブート診断] ページを開きます。
1. [ブート診断] ページで **[シリアル ログ]** をクリックします。
1. 次の文字列を検索します。"Setting Bitnami application password to" (Bitnami アプリケーションのパスワードを次のように設定する)。
1. パスワードを安全な場所にコピーします。

次に、次の手順に従って Grafana の管理者パスワードを変更します。

1. Azure portal で、VM を選択して **[概要]** をクリックします。
1. パブリック IP アドレスをコピーします。
1. Web ブラウザーを開き、次の URL に移動します:`http://<IP addresss>:3000`。
1. Grafana ログイン画面で、ユーザー名として **admin** と入力し、前の手順で取得した Grafana のパスワードを使用します。
1. ログインしたら、**[Configuration]** (歯車のアイコン) を選択します。
1. **[Server Admin]** を選択します。
1. **[User]** タブで、**[admin]** ログインを選択します。
1. パスワードを変更します。

## <a name="create-an-azure-monitor-data-source"></a>Azure Monitor データ ソースを作成する

1. Grafana で Log Analytics ワークスペースへのアクセスを管理できるようにするサービス プリンシパルを作成します。 詳細については、「[Azure CLI で Azure サービス プリンシパルを作成する](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)」をご覧ください。

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. 次のコマンドの出力にある appId、password、および tenant の値をメモします。

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. 前述の説明に従って Grafana にログインします。 **[Configuration]** (歯車のアイコン) を選択し、次に **[Data Sources]** を選択します。
1. **[Data Sources]** タブで、**[Add data source]** をクリックします。
1. データ ソースの種類として **[Azure Monitor]** を選択します。
1. **[Settings]** セクションで、**[Name]** テキストボックスにデータ ソースの名前を入力します。
1. **[Azure Monitor API Details]** セクションで、次の情報を入力します。

    * サブスクリプション ID:お使いの Azure サブスクリプション ID。
    * テナント ID:前のテナント ID。
    * クライアント ID:前の "appId" の値。
    * クライアント シークレット: 前の "password" の値。

1. **[Azure Log Analytics API Details]** セクションで、**[Same Details as Azure Monitor API]** チェックボックスにチェックマークを入れます。
1. **[Save & Test]** をクリックします。 Log Analytics のデータ ソースが正しく構成されている場合は、成功メッセージが表示されます。

## <a name="create-the-dashboard"></a>ダッシュボードを作成する

次の手順に従って、Grafana にダッシュボードを作成します。

1. GitHub リポジトリのローカル コピーの `/perftools/dashboards/grafana` ディレクトリに移動します。
1. 次のスクリプトを実行します。

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    このスクリプトの出力は、**SparkMonitoringDash.json** という名前のファイルです。

1. Grafana ダッシュボードに戻り、**[Create]** (プラス記号のアイコン) を選択します。
1. **[インポート]** を選択します。
1. **[Upload .json File]** をクリックします。
1. 手順 2 で作成した **SparkMonitoringDash.json** ファイルを選択します。
1. **[Options]** セクションの **[ALA]** で、先ほど作成した Azure Monitor データ ソースを選択します。
1. **[インポート]** をクリックします。

## <a name="visualizations-in-the-dashboards"></a>ダッシュ ボードでの視覚化

Azure Log Analytics ダッシュボード と Grafana ダッシュ ボードの両方に、時系列の視覚化のセットが含まれています。 各グラフは、Apache Spark の[ジョブ](https://spark.apache.org/docs/latest/job-scheduling.html)、ジョブのステージ、および各ステージを形成するタスクに関連したメトリック データの時系列プロットです。

次のような視覚化があります。

### <a name="job-latency"></a>ジョブの待機時間

この視覚化は、ジョブの実行の待機時間を示すもので、ジョブの全体的なパフォーマンスの粗いビューです。 開始から完了までのジョブの実行時間が表示されます。 ジョブの開始時刻はジョブの送信時刻と同じではないことに注意してください。 待機時間は、クラスター ID とアプリケーション ID によってインデックスが付けられたジョブの実行のパーセンタイル (10%、30%、50%、90%) として表されます。

### <a name="stage-latency"></a>ステージの待機時間

この視覚化は、クラスターごと、アプリケーションごと、および個々のステージごとの各ステージの待機時間を示します。 この視覚化は、実行速度が遅い特定のステージを識別するのに役立ちます。

### <a name="task-latency"></a>タスクの待機時間

この視覚化は、タスクの実行の待機時間を示します。 待機時間は、クラスター、ステージ名、およびアプリケーションごとのタスクの実行のパーセンタイルとして表されます。

### <a name="sum-task-execution-per-host"></a>ホストごとのタスクの実行の合計

この視覚化は、クラスターで実行されているホストごとのタスクの実行の待機時間の合計を示します。 ホストごとのタスクの実行の待機時間を表示すると、他のホストよりも全体的なタスクの待機時間がずっと長いホストを識別できます。 これは、タスクが非効率的または不均等にホストに分散されていたことを意味する場合があります。

### <a name="task-metrics"></a>タスクのメトリック

この視覚化は、特定のタスクの実行の実行メトリックのセットを示します。 これらのメトリックには、データ シャッフルのサイズと時間、シリアライズ操作と逆シリアライズ操作の時間などが含まれます。 メトリックの完全なセットについては、パネルの Log Analytics クエリをご覧ください。 この視覚化は、タスクを形成する操作を理解したり、各操作のリソース使用量を特定したりするのに役立ちます。 グラフの急上昇は、調査が必要なコストのかかる操作を表しています。

### <a name="cluster-throughput"></a>クラスターのスループット

この視覚化は、クラスターおよびアプリケーションごとに実行される作業量を表すためにクラスターとアプリケーションによってインデックスが付けられた作業項目の概要です。 これは、1 分間の増分で、クラスター、アプリケーション、およびステージごとに完了したジョブ、タスク、およびステージの数を示します。 

### <a name="streaming-throughputlatency"></a>ストリーミングのスループットと待機時間

この視覚化は、構造化ストリーミング クエリに関連付けられたメトリックに関連しています。 このグラフは、1 秒あたりの入力行数と 1 秒あたりの処理行数を示しています。 ストリーミング メトリックは、アプリケーションごとでも表されます。 これらのメトリックは、構造化ストリーミング クエリが処理される際に OnQueryProgress イベントが生成されるときに送信されます。そして、この視覚化ではストリーミング待機時間は、クエリ バッチを実行するために要した時間 (ミリ秒) で表わされます。

### <a name="resource-consumption-per-executor"></a>Executor ごとのリソース使用量

次は、特定の種類のリソースと、それが各クラスターで Executor ごとにどのように消費されているかを示すダッシュボードの視覚化のセットです。 これらの視覚化は、Executor ごとのリソース使用量の外れ値を識別するのに役立ちます。 たとえば、特定の Executor の作業割り振りにスキューがある場合、クラスターで実行されている他の Executor との関連でリソース使用量が上昇します。 これは、Executor のリソース使用量の急上昇によって識別できます。

### <a name="executor-compute-time-metrics"></a>Executor のコンピューティング時間のメトリック

次は、Executor のコンピューティング時間全体に対する Executor のシリアライズ時間、逆シリアライズ時間、CPU 時間、および Java 仮想マシン時間の割合を示すダッシュボードの視覚化のセットです。 これは、これらの 4 つの各メトリックが Executor の処理全体に占める割合を視覚的に示します。

### <a name="shuffle-metrics"></a>シャッフルのメトリック

最後の視覚化のセットは、Executor 全体の構造化ストリーミング クエリに関連付けられたデータ シャッフル メトリックを示します。 これには、ファイル システムが使用されているクエリでの読み取りシャッフル バイト数、書き込みシャッフル バイト数、シャッフル メモリ、およびディスク使用量が含まれます。

## <a name="next-steps"></a>次の手順

> [!div class="nextstepaction"]
> [パフォーマンスのボトルネックのトラブルシューティング](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object