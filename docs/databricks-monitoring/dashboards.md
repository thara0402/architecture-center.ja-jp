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
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a><span data-ttu-id="ac58a-103">ダッシュボードを使用して Azure Databricks のメトリックを視覚化する</span><span class="sxs-lookup"><span data-stu-id="ac58a-103">Use dashboards to visualize Azure Databricks metrics</span></span>

<span data-ttu-id="ac58a-104">この記事では、パフォーマンスの問題について Azure Databricks ジョブを監視するように Grafana ダッシュ ボードを設定する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-104">This article shows how to set up a Grafana dashboard to monitor Azure Databricks jobs for performance issues.</span></span>

<span data-ttu-id="ac58a-105">[Azure Databricks](/azure/azure-databricks/) は、ビッグ データ分析ソリューションと人工知能 (AI) ソリューションの高速な開発とデプロイを容易にする、高速、強力、かつ協調的な [Apache Spark](https://spark.apache.org/) ベースの分析サービスです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-105">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful, and collaborative [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="ac58a-106">監視は、実稼働環境で Azure Databricks ワークロードを運用するうえで重要な構成要素です。</span><span class="sxs-lookup"><span data-stu-id="ac58a-106">Monitoring is a critical component of operating Azure Databricks workloads in production.</span></span> <span data-ttu-id="ac58a-107">最初の手順は、分析のためにメトリックをワークスペースに収集することです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-107">The first step is to gather metrics into a workspace for analysis.</span></span> <span data-ttu-id="ac58a-108">Azure では、ログ データを管理するための最良のソリューションは [Azure Monitor](/azure/azure-monitor/) です。</span><span class="sxs-lookup"><span data-stu-id="ac58a-108">In Azure, the best solution for managing log data is [Azure Monitor](/azure/azure-monitor/).</span></span> <span data-ttu-id="ac58a-109">Azure Databricks は、Azure Monitor へのログ データの送信をネイティブにサポートしていませんが、[この機能のためのライブラリ](https://github.com/mspnp/spark-monitoring)が [Github](https://github.com) にあります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-109">Azure Databricks does not natively support sending log data to Azure monitor, but a [library for this functionality](https://github.com/mspnp/spark-monitoring) is available in [Github](https://github.com).</span></span>

<span data-ttu-id="ac58a-110">このライブラリにより、Azure Databricks サービスのメトリックおよび Apache Spark 構造化ストリーミング クエリ イベントのメトリックのログ記録が可能になります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-110">This library enables logging of Azure Databricks service metrics as well as Apache Spark structure streaming query event metrics.</span></span> <span data-ttu-id="ac58a-111">このライブラリを Azure Databricks クラスターに正常にデプロイしたら、さらに、実稼働環境の一部としてデプロイできる [Grafana](https://granfana.com) ダッシュボードのセットをデプロイすることができます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-111">Once you've successfully deployed this library to an Azure Databricks cluster, you can further deploy a set of [Grafana](https://granfana.com) dashboards that you can deploy as part of your production environment.</span></span>

![ダッシュボードのスクリーンショット](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a><span data-ttu-id="ac58a-113">前提条件</span><span class="sxs-lookup"><span data-stu-id="ac58a-113">Prequisites</span></span>

<span data-ttu-id="ac58a-114">[Github リポジトリ](https://github.com/mspnp/spark-monitoring)を複製し、[デプロイ手順に従って](./configure-cluster.md)、Azure Log Analytics ワークスペースにログを送信するように Azure Databricks ライブラリ用に Azure Monitor のログ記録を構築して構成します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-114">Clone the [Github repository](https://github.com/mspnp/spark-monitoring) and [follow the deployment instructions](./configure-cluster.md) to build and configure the Azure Monitor logging for Azure Databricks library to send logs to your Azure Log Analytics workspace.</span></span>

## <a name="deploy-the-azure-log-analytics-workspace"></a><span data-ttu-id="ac58a-115">Azure Log Analytics ワークスペースをデプロイする</span><span class="sxs-lookup"><span data-stu-id="ac58a-115">Deploy the Azure Log Analytics workspace</span></span>

<span data-ttu-id="ac58a-116">Azure Log Analytics ワークスペースをデプロイするには、以下の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-116">To deploy the Azure Log Analytics workspace, follow these steps:</span></span>

1. <span data-ttu-id="ac58a-117">`/perftools/deployment/loganalytics` ディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-117">Navigate to the `/perftools/deployment/loganalytics` directory.</span></span>
1. <span data-ttu-id="ac58a-118">**logAnalyticsDeploy.json** Azure Resource Manager テンプレートをデプロイします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-118">Deploy the **logAnalyticsDeploy.json** Azure Resource Manager template.</span></span> <span data-ttu-id="ac58a-119">Resource Manager テンプレートのデプロイの詳細については、「[Azure Resource Manager テンプレートと Azure CLI を使用したリソースのデプロイ][rm-cli]」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-119">For more information about deploying Resource Manager templates, see [Deploy resources with Resource Manager templates and Azure CLI][rm-cli].</span></span> <span data-ttu-id="ac58a-120">このテンプレートには、次のパラメーターがあります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-120">The template has the following parameters:</span></span>

    * <span data-ttu-id="ac58a-121">**location**:Log Analytics ワークスペースとダッシュボードがデプロイされるリージョン。</span><span class="sxs-lookup"><span data-stu-id="ac58a-121">**location**: The region where the Log Analytics workspace and dashboards are deployed.</span></span>
    * <span data-ttu-id="ac58a-122">**serviceTier**:ワークスペースの価格レベル。</span><span class="sxs-lookup"><span data-stu-id="ac58a-122">**serviceTier**: Rhe workspace pricing tier.</span></span> <span data-ttu-id="ac58a-123">有効値の一覧については、[こちら][sku]を参照してください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-123">See [here][sku] for a list of valid values.</span></span>
    * <span data-ttu-id="ac58a-124">**dataRetention** (省略可能):Log Analytics ワークスペースにログ データが保持される日数。</span><span class="sxs-lookup"><span data-stu-id="ac58a-124">**dataRetention** (optional): The number of days log data is retained in the Log Analytics workspace.</span></span> <span data-ttu-id="ac58a-125">既定値は 30 日間です。</span><span class="sxs-lookup"><span data-stu-id="ac58a-125">The default value is 30 days.</span></span> <span data-ttu-id="ac58a-126">価格レベルが `Free` の場合、データ保持は 7 日間でなければなりません。</span><span class="sxs-lookup"><span data-stu-id="ac58a-126">If the pricing tier is `Free`, the data retention must be 7 days.</span></span>
    * <span data-ttu-id="ac58a-127">**workspaceName** (省略可能):ワークスペースの名前。</span><span class="sxs-lookup"><span data-stu-id="ac58a-127">**workspaceName** (optional): A name for the workspace.</span></span> <span data-ttu-id="ac58a-128">指定しない場合、テンプレートが名前を生成します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-128">If not specified, the template generates a name.</span></span>

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

<span data-ttu-id="ac58a-129">このテンプレートはワークスペースを作成し、さらに、ダッシュボードで使用される定義済みクエリのセットも作成します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-129">This template creates the workspace and also creates a set of predefined queries that are used by by dashboard.</span></span>

## <a name="deploy-grafana-in-a-virtual-machine"></a><span data-ttu-id="ac58a-130">Grafana を仮想マシンにデプロイする</span><span class="sxs-lookup"><span data-stu-id="ac58a-130">Deploy Grafana in a virtual machine</span></span>

<span data-ttu-id="ac58a-131">Grafana はオープン ソース プロジェクトです。これをデプロイすると、Azure Monitor 用の Grafana プラグインを使用して、Azure Log Analytics ワークスペースに格納されている時間系列メトリックを視覚化することができます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-131">Grafana is an open source project you can deploy to visualize the time series metrics stored in your Azure Log Analytics workspace using the Grafana plugin for Azure Monitor.</span></span> <span data-ttu-id="ac58a-132">Grafana は仮想マシン (VM) 上で実行され、ストレージ アカウント、仮想ネットワーク、およびその他のリソースを必要とします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-132">Grafana executes on a virtual machine (VM) and requires a storage account, virtual network, and other resources.</span></span> <span data-ttu-id="ac58a-133">Bitnami で認定された Grafana イメージと関連リソースを使用して仮想マシンをデプロイするには、以下の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-133">To deploy a virtual machine with the bitnami certified Grafana image and associated resources, follow these steps:</span></span>

1. <span data-ttu-id="ac58a-134">Azure CLI を使用して、Grafana の Azure Marketplace イメージの使用条件に同意します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-134">Use the Azure CLI to accept the Azure Marketplace image terms for Grafana.</span></span>

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. <span data-ttu-id="ac58a-135">GitHub リポジトリのローカル コピーの `/spark-monitoring/perftools/deployment/grafana` ディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-135">Navigate to the `/spark-monitoring/perftools/deployment/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="ac58a-136">**grafanaDeploy.json** Resource Manager テンプレートを以下のようにデプロイします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-136">Deploy the **grafanaDeploy.json** Resource Manager template as follows:</span></span>

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

<span data-ttu-id="ac58a-137">デプロイが完了すると、Grafana の Bitnami イメージが仮想マシンにインストールされます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-137">Once the deployment is complete, the bitnami image of Grafana is installed on the virtual machine.</span></span>

## <a name="update-the-grafana-password"></a><span data-ttu-id="ac58a-138">Grafana のパスワードを更新する</span><span class="sxs-lookup"><span data-stu-id="ac58a-138">Update the Grafana password</span></span>

<span data-ttu-id="ac58a-139">セットアップ プロセスの一環として、Grafana のインストール スクリプトは、**admin** ユーザー用の一時パスワードを出力します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-139">As part of the setup process, the Grafana installation script outputs a temporary password for the **admin** user.</span></span> <span data-ttu-id="ac58a-140">サインインするためには、この一時パスワードが必要です。</span><span class="sxs-lookup"><span data-stu-id="ac58a-140">You need this temporary password to sign in.</span></span> <span data-ttu-id="ac58a-141">一時パスワードを取得するには、次の手順に従ってください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-141">To obtain the temporary password, follow these steps:</span></span>  

1. <span data-ttu-id="ac58a-142">Azure ポータルにログインします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-142">Log in to the Azure portal.</span></span>  
1. <span data-ttu-id="ac58a-143">リソースがデプロイされたリソース グループを選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-143">Select the resource group where the resources were deployed.</span></span>
1. <span data-ttu-id="ac58a-144">Grafana がインストールされた VM を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-144">Select the VM where Grafana was installed.</span></span> <span data-ttu-id="ac58a-145">デプロイ テンプレートで既定のパラメーター名を使用した場合、VM 名は **sparkmonitoring-vm-grafana** で始まります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-145">If you used the default parameter name in the deployment template, the VM name is prefaced with **sparkmonitoring-vm-grafana**.</span></span>
1. <span data-ttu-id="ac58a-146">**[サポート + トラブルシューティング]** セクションで **[ブート診断]** をクリックして、[ブート診断] ページを開きます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-146">In the **Support + troubleshooting** section, click **Boot diagnostics** to open the boot diagnostics page.</span></span>
1. <span data-ttu-id="ac58a-147">[ブート診断] ページで **[シリアル ログ]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-147">Click **Serial log** on the boot diagnostics page.</span></span>
1. <span data-ttu-id="ac58a-148">次の文字列を検索します。"Setting Bitnami application password to" (Bitnami アプリケーションのパスワードを次のように設定する)。</span><span class="sxs-lookup"><span data-stu-id="ac58a-148">Search for the following string: "Setting Bitnami application password to".</span></span>
1. <span data-ttu-id="ac58a-149">パスワードを安全な場所にコピーします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-149">Copy the password to a safe location.</span></span>

<span data-ttu-id="ac58a-150">次に、次の手順に従って Grafana の管理者パスワードを変更します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-150">Next, change the Grafana administrator password by following these steps:</span></span>

1. <span data-ttu-id="ac58a-151">Azure portal で、VM を選択して **[概要]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-151">In the Azure portal, select the VM and click **Overview**.</span></span>
1. <span data-ttu-id="ac58a-152">パブリック IP アドレスをコピーします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-152">Copy the public IP address.</span></span>
1. <span data-ttu-id="ac58a-153">Web ブラウザーを開き、次の URL に移動します:`http://<IP addresss>:3000`。</span><span class="sxs-lookup"><span data-stu-id="ac58a-153">Open a web browser and navigate to the following URL: `http://<IP addresss>:3000`.</span></span>
1. <span data-ttu-id="ac58a-154">Grafana ログイン画面で、ユーザー名として **admin** と入力し、前の手順で取得した Grafana のパスワードを使用します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-154">At the Grafana log in screen, enter **admin** for the user name, and use the Grafana password from the previous steps.</span></span>
1. <span data-ttu-id="ac58a-155">ログインしたら、**[Configuration]** (歯車のアイコン) を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-155">Once logged in, select **Configuration** (the gear icon).</span></span>
1. <span data-ttu-id="ac58a-156">**[Server Admin]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-156">Select **Server Admin**.</span></span>
1. <span data-ttu-id="ac58a-157">**[User]** タブで、**[admin]** ログインを選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-157">On the **Users** tab, select the **admin** login.</span></span>
1. <span data-ttu-id="ac58a-158">パスワードを変更します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-158">Update the password.</span></span>

## <a name="create-an-azure-monitor-data-source"></a><span data-ttu-id="ac58a-159">Azure Monitor データ ソースを作成する</span><span class="sxs-lookup"><span data-stu-id="ac58a-159">Create an Azure Monitor data source</span></span>

1. <span data-ttu-id="ac58a-160">Grafana で Log Analytics ワークスペースへのアクセスを管理できるようにするサービス プリンシパルを作成します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-160">Create a service principal that allows Grafana to manage access to your Log Analytics workspace.</span></span> <span data-ttu-id="ac58a-161">詳細については、「[Azure CLI で Azure サービス プリンシパルを作成する](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-161">For more information, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span></span>

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. <span data-ttu-id="ac58a-162">次のコマンドの出力にある appId、password、および tenant の値をメモします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-162">Note the values for appId, password, and tenant in the output from this command:</span></span>

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. <span data-ttu-id="ac58a-163">前述の説明に従って Grafana にログインします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-163">Log into Grafana as described earlier.</span></span> <span data-ttu-id="ac58a-164">**[Configuration]** (歯車のアイコン) を選択し、次に **[Data Sources]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-164">Select **Configuration** (the gear icon) and then **Data Sources**.</span></span>
1. <span data-ttu-id="ac58a-165">**[Data Sources]** タブで、**[Add data source]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-165">In the **Data Sources** tab, click **Add data source**.</span></span>
1. <span data-ttu-id="ac58a-166">データ ソースの種類として **[Azure Monitor]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-166">Select **Azure Monitor** as the data source type.</span></span>
1. <span data-ttu-id="ac58a-167">**[Settings]** セクションで、**[Name]** テキストボックスにデータ ソースの名前を入力します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-167">In the **Settings** section, enter a name for the data source in the **Name** textbox.</span></span>
1. <span data-ttu-id="ac58a-168">**[Azure Monitor API Details]** セクションで、次の情報を入力します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-168">In the **Azure Monitor API Details** section, enter the following information:</span></span>

    * <span data-ttu-id="ac58a-169">サブスクリプション ID:お使いの Azure サブスクリプション ID。</span><span class="sxs-lookup"><span data-stu-id="ac58a-169">Subscription Id: Your Azure subscription ID.</span></span>
    * <span data-ttu-id="ac58a-170">テナント ID:前のテナント ID。</span><span class="sxs-lookup"><span data-stu-id="ac58a-170">Tenant Id: The tenant ID from earlier.</span></span>
    * <span data-ttu-id="ac58a-171">クライアント ID:前の "appId" の値。</span><span class="sxs-lookup"><span data-stu-id="ac58a-171">Client Id: The value of "appId" from earlier.</span></span>
    * <span data-ttu-id="ac58a-172">クライアント シークレット: 前の "password" の値。</span><span class="sxs-lookup"><span data-stu-id="ac58a-172">Client Secret: The value of "password" from earlier.</span></span>

1. <span data-ttu-id="ac58a-173">**[Azure Log Analytics API Details]** セクションで、**[Same Details as Azure Monitor API]** チェックボックスにチェックマークを入れます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-173">In the **Azure Log Analytics API Details** section, check the **Same Details as Azure Monitor API** checkbox.</span></span>
1. <span data-ttu-id="ac58a-174">**[Save & Test]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-174">Click **Save & Test**.</span></span> <span data-ttu-id="ac58a-175">Log Analytics のデータ ソースが正しく構成されている場合は、成功メッセージが表示されます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-175">If the Log Analytics data source is correctly configured, a success message is displayed.</span></span>

## <a name="create-the-dashboard"></a><span data-ttu-id="ac58a-176">ダッシュボードを作成する</span><span class="sxs-lookup"><span data-stu-id="ac58a-176">Create the dashboard</span></span>

<span data-ttu-id="ac58a-177">次の手順に従って、Grafana にダッシュボードを作成します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-177">Create the dashboards in Grafana by following these steps:</span></span>

1. <span data-ttu-id="ac58a-178">GitHub リポジトリのローカル コピーの `/perftools/dashboards/grafana` ディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-178">Navigate to the `/perftools/dashboards/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="ac58a-179">次のスクリプトを実行します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-179">Run the following script:</span></span>

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    <span data-ttu-id="ac58a-180">このスクリプトの出力は、**SparkMonitoringDash.json** という名前のファイルです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-180">The output from the script is a file named **SparkMonitoringDash.json**.</span></span>

1. <span data-ttu-id="ac58a-181">Grafana ダッシュボードに戻り、**[Create]** (プラス記号のアイコン) を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-181">Return to the Grafana dashboard and select **Create** (the plus icon).</span></span>
1. <span data-ttu-id="ac58a-182">**[インポート]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-182">Select **Import**.</span></span>
1. <span data-ttu-id="ac58a-183">**[Upload .json File]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-183">Click **Upload .json File**.</span></span>
1. <span data-ttu-id="ac58a-184">手順 2 で作成した **SparkMonitoringDash.json** ファイルを選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-184">Select the **SparkMonitoringDash.json** file created in step 2.</span></span>
1. <span data-ttu-id="ac58a-185">**[Options]** セクションの **[ALA]** で、先ほど作成した Azure Monitor データ ソースを選択します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-185">In the **Options** section, under **ALA**, select the Azure Monitor data source created earlier.</span></span>
1. <span data-ttu-id="ac58a-186">**[インポート]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="ac58a-186">Click **Import**.</span></span>

## <a name="visualizations-in-the-dashboards"></a><span data-ttu-id="ac58a-187">ダッシュ ボードでの視覚化</span><span class="sxs-lookup"><span data-stu-id="ac58a-187">Visualizations in the dashboards</span></span>

<span data-ttu-id="ac58a-188">Azure Log Analytics ダッシュボード と Grafana ダッシュ ボードの両方に、時系列の視覚化のセットが含まれています。</span><span class="sxs-lookup"><span data-stu-id="ac58a-188">Both the Azure Log Analytics and Grafana dashboards include a set of time-series visualizations.</span></span> <span data-ttu-id="ac58a-189">各グラフは、Apache Spark の[ジョブ](https://spark.apache.org/docs/latest/job-scheduling.html)、ジョブのステージ、および各ステージを形成するタスクに関連したメトリック データの時系列プロットです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-189">Each graph is time-series plot of metric data related to an Apache Spark [job](https://spark.apache.org/docs/latest/job-scheduling.html), stages of the job, and tasks that make up each stage.</span></span>

<span data-ttu-id="ac58a-190">次のような視覚化があります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-190">The visualizations are as follows:</span></span>

### <a name="job-latency"></a><span data-ttu-id="ac58a-191">ジョブの待機時間</span><span class="sxs-lookup"><span data-stu-id="ac58a-191">Job latency</span></span>

<span data-ttu-id="ac58a-192">この視覚化は、ジョブの実行の待機時間を示すもので、ジョブの全体的なパフォーマンスの粗いビューです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-192">This visualization shows execution latency for a job, which is a coarse view on the overall peformance of a job.</span></span> <span data-ttu-id="ac58a-193">開始から完了までのジョブの実行時間が表示されます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-193">Displays the job execution duration from start to completion.</span></span> <span data-ttu-id="ac58a-194">ジョブの開始時刻はジョブの送信時刻と同じではないことに注意してください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-194">Note that the job start time is not the same as the job submission time.</span></span> <span data-ttu-id="ac58a-195">待機時間は、クラスター ID とアプリケーション ID によってインデックスが付けられたジョブの実行のパーセンタイル (10%、30%、50%、90%) として表されます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-195">Latency is represented as percentiles (10%, 30%, 50%, 90%) of job execution indexed by cluster ID and application ID.</span></span>

### <a name="stage-latency"></a><span data-ttu-id="ac58a-196">ステージの待機時間</span><span class="sxs-lookup"><span data-stu-id="ac58a-196">Stage latency</span></span>

<span data-ttu-id="ac58a-197">この視覚化は、クラスターごと、アプリケーションごと、および個々のステージごとの各ステージの待機時間を示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-197">The visualization shows the latency of each stage per cluster, per application, and per individual stage.</span></span> <span data-ttu-id="ac58a-198">この視覚化は、実行速度が遅い特定のステージを識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-198">This visualization is useful for identifying a particular stage that is running slowly.</span></span>

### <a name="task-latency"></a><span data-ttu-id="ac58a-199">タスクの待機時間</span><span class="sxs-lookup"><span data-stu-id="ac58a-199">Task latency</span></span>

<span data-ttu-id="ac58a-200">この視覚化は、タスクの実行の待機時間を示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-200">This visualization shows task execution latency.</span></span> <span data-ttu-id="ac58a-201">待機時間は、クラスター、ステージ名、およびアプリケーションごとのタスクの実行のパーセンタイルとして表されます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-201">Latency is represented as a percentile of task execution per cluster, stage name, and application.</span></span>

### <a name="sum-task-execution-per-host"></a><span data-ttu-id="ac58a-202">ホストごとのタスクの実行の合計</span><span class="sxs-lookup"><span data-stu-id="ac58a-202">Sum Task Execution per host</span></span>

<span data-ttu-id="ac58a-203">この視覚化は、クラスターで実行されているホストごとのタスクの実行の待機時間の合計を示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-203">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="ac58a-204">ホストごとのタスクの実行の待機時間を表示すると、他のホストよりも全体的なタスクの待機時間がずっと長いホストを識別できます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-204">Viewing task execution latency per host identifies hosts that have much higher overall task latency than other hosts.</span></span> <span data-ttu-id="ac58a-205">これは、タスクが非効率的または不均等にホストに分散されていたことを意味する場合があります。</span><span class="sxs-lookup"><span data-stu-id="ac58a-205">This may mean that tasks have been inefficiently or unevenly distributed to hosts.</span></span>

### <a name="task-metrics"></a><span data-ttu-id="ac58a-206">タスクのメトリック</span><span class="sxs-lookup"><span data-stu-id="ac58a-206">Task metrics</span></span>

<span data-ttu-id="ac58a-207">この視覚化は、特定のタスクの実行の実行メトリックのセットを示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-207">This visualization shows a set of the execution metrics for a given task's execution.</span></span> <span data-ttu-id="ac58a-208">これらのメトリックには、データ シャッフルのサイズと時間、シリアライズ操作と逆シリアライズ操作の時間などが含まれます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-208">These metrics include the size and duration of a data shuffle, duration of serialization and deserialization operations, and others.</span></span> <span data-ttu-id="ac58a-209">メトリックの完全なセットについては、パネルの Log Analytics クエリをご覧ください。</span><span class="sxs-lookup"><span data-stu-id="ac58a-209">For the full set of metrics, view the Log Analytics query for the panel.</span></span> <span data-ttu-id="ac58a-210">この視覚化は、タスクを形成する操作を理解したり、各操作のリソース使用量を特定したりするのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-210">This visualization is useful for understanding the operations that make up a task and identifying resource consumption of each operation.</span></span> <span data-ttu-id="ac58a-211">グラフの急上昇は、調査が必要なコストのかかる操作を表しています。</span><span class="sxs-lookup"><span data-stu-id="ac58a-211">Spikes in the graph represent costly operations that should be investigated.</span></span>

### <a name="cluster-throughput"></a><span data-ttu-id="ac58a-212">クラスターのスループット</span><span class="sxs-lookup"><span data-stu-id="ac58a-212">Cluster throughput</span></span>

<span data-ttu-id="ac58a-213">この視覚化は、クラスターおよびアプリケーションごとに実行される作業量を表すためにクラスターとアプリケーションによってインデックスが付けられた作業項目の概要です。</span><span class="sxs-lookup"><span data-stu-id="ac58a-213">This visualization is a high level view of work items indexed by cluster and application to represent the amount of work done per cluster and application.</span></span> <span data-ttu-id="ac58a-214">これは、1 分間の増分で、クラスター、アプリケーション、およびステージごとに完了したジョブ、タスク、およびステージの数を示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-214">It shows the number of jobs, tasks, and stages completed per cluster, application, and stage in one minute increments.</span></span> 

### <a name="streaming-throughputlatency"></a><span data-ttu-id="ac58a-215">ストリーミングのスループットと待機時間</span><span class="sxs-lookup"><span data-stu-id="ac58a-215">Streaming Throughput/Latency</span></span>

<span data-ttu-id="ac58a-216">この視覚化は、構造化ストリーミング クエリに関連付けられたメトリックに関連しています。</span><span class="sxs-lookup"><span data-stu-id="ac58a-216">This visualzation is related to the metrics associated with a structured streaming query.</span></span> <span data-ttu-id="ac58a-217">このグラフは、1 秒あたりの入力行数と 1 秒あたりの処理行数を示しています。</span><span class="sxs-lookup"><span data-stu-id="ac58a-217">The graphs shows the number of input rows per second and the number of rows processed per second.</span></span> <span data-ttu-id="ac58a-218">ストリーミング メトリックは、アプリケーションごとでも表されます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-218">The streaming metrics are also represented per application.</span></span> <span data-ttu-id="ac58a-219">これらのメトリックは、構造化ストリーミング クエリが処理される際に OnQueryProgress イベントが生成されるときに送信されます。そして、この視覚化ではストリーミング待機時間は、クエリ バッチを実行するために要した時間 (ミリ秒) で表わされます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-219">These metrics are sent when the OnQueryProgress event is generated as the structured streaming query is processed and the visualization represents streaming latency as the amount of time, in milliseconds, taken to execute a query batch.</span></span>

### <a name="resource-consumption-per-executor"></a><span data-ttu-id="ac58a-220">Executor ごとのリソース使用量</span><span class="sxs-lookup"><span data-stu-id="ac58a-220">Resource consumption per executor</span></span>

<span data-ttu-id="ac58a-221">次は、特定の種類のリソースと、それが各クラスターで Executor ごとにどのように消費されているかを示すダッシュボードの視覚化のセットです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-221">Next is a set of visualizations for the dashboard show the particular type of resource and how it is consumed per executor on each cluster.</span></span> <span data-ttu-id="ac58a-222">これらの視覚化は、Executor ごとのリソース使用量の外れ値を識別するのに役立ちます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-222">These visualizations help identify outliers in resource consumption per executor.</span></span> <span data-ttu-id="ac58a-223">たとえば、特定の Executor の作業割り振りにスキューがある場合、クラスターで実行されている他の Executor との関連でリソース使用量が上昇します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-223">For example, if the work allocation for a particular executor is skewed, resource consumption will be elevated in relation to other executors running on the cluster.</span></span> <span data-ttu-id="ac58a-224">これは、Executor のリソース使用量の急上昇によって識別できます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-224">This can be identified by spikes in the resource consumption for an executor.</span></span>

### <a name="executor-compute-time-metrics"></a><span data-ttu-id="ac58a-225">Executor のコンピューティング時間のメトリック</span><span class="sxs-lookup"><span data-stu-id="ac58a-225">Executor compute time metrics</span></span>

<span data-ttu-id="ac58a-226">次は、Executor のコンピューティング時間全体に対する Executor のシリアライズ時間、逆シリアライズ時間、CPU 時間、および Java 仮想マシン時間の割合を示すダッシュボードの視覚化のセットです。</span><span class="sxs-lookup"><span data-stu-id="ac58a-226">Next is a set of visualizations for the dashboard that show the ratio of executor serialize time, deserialize time, CPU time, and Java virtual machine time to overall executor compute time.</span></span> <span data-ttu-id="ac58a-227">これは、これらの 4 つの各メトリックが Executor の処理全体に占める割合を視覚的に示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-227">This demonstrates visually how much each of these four metrics are contributing to overall executor processing.</span></span>

### <a name="shuffle-metrics"></a><span data-ttu-id="ac58a-228">シャッフルのメトリック</span><span class="sxs-lookup"><span data-stu-id="ac58a-228">Shuffle metrics</span></span>

<span data-ttu-id="ac58a-229">最後の視覚化のセットは、Executor 全体の構造化ストリーミング クエリに関連付けられたデータ シャッフル メトリックを示します。</span><span class="sxs-lookup"><span data-stu-id="ac58a-229">The final set of visualizations show the data shuffle metrics associated with a structured streaming query across all executors.</span></span> <span data-ttu-id="ac58a-230">これには、ファイル システムが使用されているクエリでの読み取りシャッフル バイト数、書き込みシャッフル バイト数、シャッフル メモリ、およびディスク使用量が含まれます。</span><span class="sxs-lookup"><span data-stu-id="ac58a-230">These include shuffle bytes read, shuffle bytes written, shuffle memory, and disk usage in queries where the file system is used.</span></span>

## <a name="next-steps"></a><span data-ttu-id="ac58a-231">次の手順</span><span class="sxs-lookup"><span data-stu-id="ac58a-231">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="ac58a-232">パフォーマンスのボトルネックのトラブルシューティング</span><span class="sxs-lookup"><span data-stu-id="ac58a-232">Troubleshoot performance bottlenecks</span></span>](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object