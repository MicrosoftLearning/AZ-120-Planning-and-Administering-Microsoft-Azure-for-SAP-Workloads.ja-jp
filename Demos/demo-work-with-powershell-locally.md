# <a name="demonstration-work-with-powershell-locally"></a>デモンストレーション: PowerShell をローカルで使用する

In this demonstration, we will install Azure Az PowerShell module. The Az module is available from a global repository called the <bpt id="p1">*</bpt>PowerShell Gallery<ept id="p1">*</ept>. You can install the module onto your local machine through the <bpt id="p1">**</bpt>Install-Module<ept id="p1">**</ept> command. You need an elevated PowerShell shell prompt to install modules from the PowerShell Gallery. 

## <a name="install-the-az-module"></a>Az モジュールをインストールする

1. **[スタート]** メニューを開き、「**Windows PowerShell**」と入力します。
2. **Windows PowerShell** アイコンを右クリックし、 **[管理者として実行する]** を選択します。
3. **[ユーザー アカウント制御]** ダイアログで、**[はい]** を選択します。
4. Type the following command, and then press Enter. This command installs the module for all users by default. (It's controlled by the scope parameter.) AllowClobber overwrites the previous PowerShell module. 

    ```
    Install-Module -Name Az -AllowClobber
    ```

## <a name="install-nuget-if-needed"></a>NuGet をインストールする (必要な場合)

1. インストールした NuGet バージョンによっては、最新バージョンのダウンロードとインストールを求めるプロンプトが表示される場合があります。
2. プロンプトが表示された場合は、NuGet プロバイダーをインストールしてインポートします。

## <a name="trust-the-repository"></a>リポジトリを信頼する

1. このデモでは、Az PowerShell モジュールをインストールします。

    ```
    You are installing the modules from an untrusted repository. If you trust this repository, change its
    InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
    'PSGallery'?
    ```

2. プロンプトに従い、モジュールをインストールします。 

## <a name="connect-to-azure-and-view-your-subscription-information"></a>Azure に接続し、サブスクリプション情報を表示する

1. Azure にログインする

    ```
    Login-AzAccount
    ```

2. プロンプトが表示されたら、認証情報を入力します。
3. サブスクリプション情報を確認します。

    ```
    Get-AzSubscription
    ```

## <a name="create-resources"></a>リソースを作成する

1. Az モジュールは、*PowerShell ギャラリー*と呼ばれるグローバル リポジトリから入手できます。

    ```
    New-AzResourceGroup -name <name> -location <location>
    ```

2. リソース グループを確認します。 
  
    ```
    Get-AzResourceGroup
    ```

3. **Install-Module** コマンドを使用して、モジュールをローカル マシンにインストールします。 

    ```
    Remove-AzResourceGroup -Name Test
    ```