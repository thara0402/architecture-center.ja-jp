---
title: "Key Vault を使用したアプリケーション シークレットの保護"
description: "Key Vault サービスを使用してアプリケーション シークレットを格納する方法"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 45d1564c255f2450f68c5e92ebe0d7de0c40ae31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2017
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="3c72f-103">Azure Key Vault を使用したアプリケーション シークレットの保護</span><span class="sxs-lookup"><span data-stu-id="3c72f-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="3c72f-104">[![GitHub](../_images/github.png) サンプル コード][sample application]</span><span class="sxs-lookup"><span data-stu-id="3c72f-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="3c72f-105">次のようなアプリケーション設定は機密情報であり、保護する必要があることは一般的です。</span><span class="sxs-lookup"><span data-stu-id="3c72f-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="3c72f-106">データベース接続文字列</span><span class="sxs-lookup"><span data-stu-id="3c72f-106">Database connection strings</span></span>
* <span data-ttu-id="3c72f-107">パスワード</span><span class="sxs-lookup"><span data-stu-id="3c72f-107">Passwords</span></span>
* <span data-ttu-id="3c72f-108">暗号化キー</span><span class="sxs-lookup"><span data-stu-id="3c72f-108">Cryptographic keys</span></span>

<span data-ttu-id="3c72f-109">セキュリティのベスト プラクティスとして、ソース管理でこれらの機密情報を保存しないでください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="3c72f-110">ソースコード リポジトリがプライベートであっても、機密情報が漏洩しやすくなります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="3c72f-111">また、これは機密情報が一般に公開されることから保護するだけではありません。</span><span class="sxs-lookup"><span data-stu-id="3c72f-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="3c72f-112">大規模なプロジェクトでは、実稼働の機密情報にアクセスできる開発者およびオペレーターを制限する必要がある場合があります </span><span class="sxs-lookup"><span data-stu-id="3c72f-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="3c72f-113">(テスト環境または開発環境の設定は異なります)。</span><span class="sxs-lookup"><span data-stu-id="3c72f-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="3c72f-114">より安全なオプションは、これらの機密情報を [Azure Key Vault][KeyVault] に保存することです。</span><span class="sxs-lookup"><span data-stu-id="3c72f-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="3c72f-115">Key Vault とは、暗号化キーやその他の機密情報を管理するための、クラウドでホストされているサービスです。</span><span class="sxs-lookup"><span data-stu-id="3c72f-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="3c72f-116">この記事では、Key Vault を使用して、アプリケーションの構成設定を格納する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-116">This article shows how to use Key Vault to store configuration settings for you app.</span></span>

<span data-ttu-id="3c72f-117">[Tailspin Surveys][Surveys] アプリケーションでは、次の設定が機密情報です。</span><span class="sxs-lookup"><span data-stu-id="3c72f-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="3c72f-118">データベース接続文字列。</span><span class="sxs-lookup"><span data-stu-id="3c72f-118">The database connection string.</span></span>
* <span data-ttu-id="3c72f-119">Redis 接続文字列。</span><span class="sxs-lookup"><span data-stu-id="3c72f-119">The Redis connection string.</span></span>
* <span data-ttu-id="3c72f-120">Web アプリケーションのクライアント シークレット。</span><span class="sxs-lookup"><span data-stu-id="3c72f-120">The client secret for the web application.</span></span>

<span data-ttu-id="3c72f-121">Surveys アプリケーションは、次の場所から構成設定を読み込みます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="3c72f-122">appsettings.json ファイル</span><span class="sxs-lookup"><span data-stu-id="3c72f-122">The appsettings.json file</span></span>
* <span data-ttu-id="3c72f-123">[ユーザー シークレット ストア][user-secrets] (開発環境のみ、テスト目的)</span><span class="sxs-lookup"><span data-stu-id="3c72f-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="3c72f-124">ホスティング環境 (Azure Web アプリのアプリ設定)</span><span class="sxs-lookup"><span data-stu-id="3c72f-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="3c72f-125">Key Vault (有効な場合)</span><span class="sxs-lookup"><span data-stu-id="3c72f-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="3c72f-126">これらはそれぞれ、以前の設定を上書きするため、Key Vault に格納されているすべての設定が優先されます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="3c72f-127">既定では、Key Vault の構成プロバイダーは無効になっています。</span><span class="sxs-lookup"><span data-stu-id="3c72f-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="3c72f-128">これは、アプリケーションをローカルで実行する場合には必要ありません。</span><span class="sxs-lookup"><span data-stu-id="3c72f-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="3c72f-129">運用環境のデプロイで有効にします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="3c72f-130">起動時に、アプリケーションはすべての登録済み設定プロバイダーからの設定を読み取り、これらの設定を使用して、厳密に型指定されたオプションのオブジェクトを設定します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="3c72f-131">詳細については、[オプションと構成オブジェクトの使用][options]に関する記事をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="3c72f-132">Surveys アプリケーションでの Key Vault の設定</span><span class="sxs-lookup"><span data-stu-id="3c72f-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="3c72f-133">前提条件:</span><span class="sxs-lookup"><span data-stu-id="3c72f-133">Prerequisites:</span></span>

* <span data-ttu-id="3c72f-134">[Azure Resource Manager コマンドレット][azure-rm-cmdlets]をインストールします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="3c72f-135">「[Run the Surveys application (Surveys アプリケーションの実行)][readme]」の説明に従って、Surveys アプリケーションを構成します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="3c72f-136">手順の概要は次のとおりです。</span><span class="sxs-lookup"><span data-stu-id="3c72f-136">High-level steps:</span></span>

1. <span data-ttu-id="3c72f-137">テナントで管理者ユーザーを設定します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="3c72f-138">クライアント証明書を設定します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="3c72f-139">Key Vault を作成します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-139">Create a key vault.</span></span>
4. <span data-ttu-id="3c72f-140">作成した Key Vault に構成設定を追加します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="3c72f-141">Key Vault を有効にするコードをコメント解除します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="3c72f-142">アプリケーションのユーザー シークレットを更新します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="3c72f-143">管理者ユーザーを設定する</span><span class="sxs-lookup"><span data-stu-id="3c72f-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="3c72f-144">Key Vault を作成するには、Azure サブスクリプションを管理できるアカウントを使用する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="3c72f-145">また、Key Vault からの読み取りを承認する任意のアプリケーションを、そのアカウントと同じテナントに登録する必要があります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="3c72f-146">この手順では、Surveys アプリケーションが登録されているテナントのユーザーとしてサインインした状態で、Key Vault を作成できることを確認します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="3c72f-147">Surveys アプリケーションが登録されている Azure AD テナント内に管理者ユーザーを作成します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="3c72f-148">[Azure Portal][azure-portal] にログインします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="3c72f-149">アプリケーションが登録されている Azure AD テナントを選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="3c72f-150">**[その他のサービス]** > **[セキュリティ + ID]** > **[Azure Active Directory]** > **[ユーザーとグループ]** > **[すべてのユーザー]** の順にクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="3c72f-151">ポータルの上部にある **[新しいユーザー]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="3c72f-152">フィールドに入力し、ユーザーを **[全体管理者]** ディレクトリ ロールに割り当てます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="3c72f-153">**Create** をクリックしてください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-153">Click **Create**.</span></span>

![全体管理者ユーザー](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="3c72f-155">このユーザーを、サブスクリプションの所有者として割り当てます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="3c72f-156">ハブ メニューで、**[サブスクリプション]**を選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="3c72f-157">この管理者がアクセスするサブスクリプションを選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="3c72f-158">サブスクリプション ブレードで、**[アクセス制御 (IAM)]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="3c72f-159">**[追加]**をクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-159">Click **Add**.</span></span>
4. <span data-ttu-id="3c72f-160">**[ロール]** で、**[所有者]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="3c72f-161">所有者として追加するユーザーの電子メール アドレスを入力します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="3c72f-162">ユーザーを選択して **[保存]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="3c72f-163">クライアント証明書を設定する</span><span class="sxs-lookup"><span data-stu-id="3c72f-163">Set up a client certificate</span></span>
1. <span data-ttu-id="3c72f-164">次のとおり、PowerShell スクリプト [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] を実行します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="3c72f-165">`Subject` パラメーターに任意の名前を入力します ("surveysapp" など)。</span><span class="sxs-lookup"><span data-stu-id="3c72f-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="3c72f-166">このスクリプトによって、自己署名証明書が生成され、"Current User/Personal" 証明書ストアに保存されます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="3c72f-167">スクリプトの出力は、JSON フラグメントです。</span><span class="sxs-lookup"><span data-stu-id="3c72f-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="3c72f-168">この値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-168">Copy this value.</span></span>

2. <span data-ttu-id="3c72f-169">[[Azure Portal]][azure-portal] で、ポータルの右上隅にあるアカウントを選択して、Survey アプリケーションが登録されているディレクトリに移動します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="3c72f-170">**[Azure Active Directory]** > **[アプリの登録]** > [Surveys] の順に選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="3c72f-171">**[マニフェスト]** をクリックし、**[編集]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="3c72f-172">スクリプトの出力を `keyCredentials` プロパティに貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="3c72f-173">次のようになります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="3c72f-174">**[Save]** をクリックします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-174">Click **Save**.</span></span>  

7. <span data-ttu-id="3c72f-175">手順 3 - 6 を繰り返し、同じ JSON フラグメントを Web API (Surveys.WebAPI) のアプリケーション マニフェストに追加します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="3c72f-176">PowerShell ウィンドウから次のコマンドを実行して、証明書の拇印を取得します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="3c72f-177">`[subject]` には、PowerShell スクリプトの Subject に指定した値を使用します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="3c72f-178">拇印は "Cert Hash(sha1)" の下に表示されます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="3c72f-179">この値をコピーします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-179">Copy this value.</span></span> <span data-ttu-id="3c72f-180">拇印は後で使用します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="3c72f-181">Key Vault を作成します</span><span class="sxs-lookup"><span data-stu-id="3c72f-181">Create a key vault</span></span>
1. <span data-ttu-id="3c72f-182">次のとおり、PowerShell スクリプト [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] を実行します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="3c72f-183">資格情報の入力を求められたら、先ほど作成した Azure AD ユーザーとしてサインインします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="3c72f-184">このスクリプトにより、新しいリソース グループと、そのリソース グループ内に新しい Key Vault が作成されます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="3c72f-185">次のように SetupKeyVault.ps を再び実行します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="3c72f-186">次のパラメーター値を設定します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="3c72f-187">key vault name = 前の手順で Key Vault に指定した名前。</span><span class="sxs-lookup"><span data-stu-id="3c72f-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="3c72f-188">Surveys app ID = Surveys Web アプリケーションのアプリケーション ID。</span><span class="sxs-lookup"><span data-stu-id="3c72f-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="3c72f-189">Surveys.WebApi app ID = Surveys.WebAPI アプリケーションのアプリケーション ID。</span><span class="sxs-lookup"><span data-stu-id="3c72f-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="3c72f-190">例:</span><span class="sxs-lookup"><span data-stu-id="3c72f-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="3c72f-191">このスクリプトでは、Web アプリと Web API が Key Vault からシークレットを取得することを承認します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="3c72f-192">詳細については、「[Azure Key Vault の概要](/azure/key-vault/key-vault-get-started/)」をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="3c72f-193">作成した Key Vault に構成設定を追加する</span><span class="sxs-lookup"><span data-stu-id="3c72f-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="3c72f-194">次のように SetupKeyVault.ps を実行します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="3c72f-195">ここで、</span><span class="sxs-lookup"><span data-stu-id="3c72f-195">where</span></span>
   
   * <span data-ttu-id="3c72f-196">key vault name = 前の手順で Key Vault に指定した名前。</span><span class="sxs-lookup"><span data-stu-id="3c72f-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="3c72f-197">Redis DNS name = Redis Cache インスタンスの DNS 名。</span><span class="sxs-lookup"><span data-stu-id="3c72f-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="3c72f-198">Redis access key = Redis Cache インスタンスのアクセス キー。</span><span class="sxs-lookup"><span data-stu-id="3c72f-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="3c72f-199">この時点で、Key Vault にシークレットが正常に保存されたかどうかをテストすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="3c72f-200">次の PowerShell コマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="3c72f-201">もう一度 SetupKeyVault.ps を実行して、データベース接続文字列を追加します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="3c72f-202">ここでは、 `<<DB connection string>>` がデータベース接続文字列の値です。</span><span class="sxs-lookup"><span data-stu-id="3c72f-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="3c72f-203">ローカル データベースでテストする場合は、Tailspin.Surveys.Web/appsettings.json ファイルから接続文字列をコピーします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="3c72f-204">これを行う場合、必ず二重の円記号 ('\\\\') を 1 つの円記号に変更してください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="3c72f-205">2 つのバックスラッシュは、JSON ファイルにおけるエスケープ文字です。</span><span class="sxs-lookup"><span data-stu-id="3c72f-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="3c72f-206">例:</span><span class="sxs-lookup"><span data-stu-id="3c72f-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="3c72f-207">Key Vault を有効にするコードをコメント解除する</span><span class="sxs-lookup"><span data-stu-id="3c72f-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="3c72f-208">Tailspin.Surveys ソリューションを開きます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="3c72f-209">Tailspin.Surveys.Web/Startup.cs で、次のコード ブロックを見つけてコメントを解除します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="3c72f-210">Tailspin.Surveys.Web/Startup.cs で、`ICredentialService` を登録するコードを探します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="3c72f-211">`CertificateCredentialService` を使用する行のコメントを解除し、`ClientCredentialService` を使用する行をコメントにします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="3c72f-212">この変更により、[クライアント アサーション][client-assertion]を使用する Web アプリが OAuth アクセス トークンを取得できるようになります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="3c72f-213">クライアント アサーションを使用すると、OAuth クライアント シークレットは不要になります。</span><span class="sxs-lookup"><span data-stu-id="3c72f-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="3c72f-214">代わりに、Key Vault にクライアント シークレットを格納することができます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="3c72f-215">ただし、Key Vault とクライアント アサーションの両方がクライアント証明書を使用するため、Key Vault を有効にする場合は、クライアント アサーションも有効にすることをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="3c72f-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="3c72f-216">ユーザー シークレットを更新する</span><span class="sxs-lookup"><span data-stu-id="3c72f-216">Update the user secrets</span></span>
<span data-ttu-id="3c72f-217">ソリューション エクスプローラーで Tailspin.Surveys.Web プロジェクトを右クリックし、 **[ユーザー シークレットの管理]**を選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="3c72f-218">secrets.json ファイルで、既存の JSON を削除し、次のコードを貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "ClientSecret": "[Surveys web app client secret]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

<span data-ttu-id="3c72f-219">角かっこ ([ ]) 内のエントリを適切な値に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="3c72f-220">`AzureAd:ClientId`: Surveys アプリケーションのクライアント ID。</span><span class="sxs-lookup"><span data-stu-id="3c72f-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="3c72f-221">`AzureAd:ClientSecret`: Azure AD で Surveys アプリケーションを登録したときに生成したキー。</span><span class="sxs-lookup"><span data-stu-id="3c72f-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="3c72f-222">`AzureAd:WebApiResourceId`: Azure AD で Surveys.WebAPI アプリケーションの作成時に指定したアプリ ID URI。</span><span class="sxs-lookup"><span data-stu-id="3c72f-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="3c72f-223">`Asymmetric:CertificateThumbprint`: クライアント証明書の作成時に以前入手した証明書の拇印。</span><span class="sxs-lookup"><span data-stu-id="3c72f-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="3c72f-224">`KeyVault:Name`: Key Vault の名前。</span><span class="sxs-lookup"><span data-stu-id="3c72f-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="3c72f-225">`Asymmetric:ValidationRequired` が false になっているのは、前に作成した証明書がルート証明機関 (CA) によって署名されていないためです。</span><span class="sxs-lookup"><span data-stu-id="3c72f-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="3c72f-226">運用時には、ルート CA によって署名された証明書を使用して、`ValidationRequired` を true に設定してください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="3c72f-227">更新した secrets.json ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="3c72f-228">次に、ソリューション エクスプローラーで、Tailspin.Surveys.WebApi プロジェクトを右クリックし、 **[ユーザー シークレットの管理]**を選択します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="3c72f-229">既存の JSON を削除し、次のコードを貼り付けます。</span><span class="sxs-lookup"><span data-stu-id="3c72f-229">Delete the existing JSON and paste in the following:</span></span>

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="3c72f-230">角かっこ ([ ]) 内のエントリを置き換え、secrets.json ファイルを保存します。</span><span class="sxs-lookup"><span data-stu-id="3c72f-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="3c72f-231">Web API の場合は、Surveys アプリケーションではなく、Surveys.WebAPI アプリケーションのクライアント ID を使用してください。</span><span class="sxs-lookup"><span data-stu-id="3c72f-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="3c72f-232">[**次へ**][adfs]</span><span class="sxs-lookup"><span data-stu-id="3c72f-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
