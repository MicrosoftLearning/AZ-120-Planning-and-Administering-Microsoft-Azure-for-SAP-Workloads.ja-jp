---
ms.openlocfilehash: f7dfd65b73dd2e48b37b7d4e26ed86a4a82463ae
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857582"
---
# <a name="demonstration-work-with-powershell-locally"></a>デモンストレーション: PowerShell をローカルで使用する

このデモでは、Az PowerShell モジュールをインストールします。 Az モジュールは、*PowerShell ギャラリー* と呼ばれるグローバル リポジトリから入手できます。 **Install-Module** コマンドを使用して、モジュールをローカル マシンにインストールします。 PowerShell ギャラリーからモジュールをインストールする場合、昇格された PowerShell シェル プロンプトが必要です。 

## <a name="install-the-az-module"></a>Az モジュールをインストールする

1. **[スタート]** メニューを開き、「**Windows PowerShell**」と入力します。
2. **Windows PowerShell** アイコンを右クリックし、 **[管理者として実行する]** を選択します。
3. **[ユーザー アカウント制御]** ダイアログで、**[はい]** を選択します。
4. 次のコマンドを入力して、Enter キーを押します。 このコマンドは、既定ではすべてのユーザーにモジュールをインストールします。 (スコープ パラメーターによって制御されます。)AllowClobber は、以前の PowerShell モジュールを上書きします。 

    ```
    Install-Module -Name Az -AllowClobber
    ```

## <a name="install-nuget-if-needed"></a>NuGet をインストールする (必要な場合)

1. インストールした NuGet バージョンによっては、最新バージョンのダウンロードとインストールを求めるプロンプトが表示される場合があります。
2. プロンプトが表示された場合は、NuGet プロバイダーをインストールしてインポートします。

## <a name="trust-the-repository"></a>リポジトリを信頼する

1. 既定では、PowerShell ギャラリーは、PowerShellGet 用の信頼できるリポジトリとしては構成されません。 PowerShell ギャラリーを初めて使用する場合、プロンプトが表示されます。

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

1. 新しいリソース グループを作成します。 必要に応じて、別の保存先を指定します。 *名前* はサブスクリプション内で一意である必要があります。 *保存先* によって、リソース グループのメタデータが格納される場所が決まります。 "米国西部"、"北ヨーロッパ"、"インド西部" などの文字列を使用して場所を指定するか、westus、northeurope、westindia など、同じものを意味する 1 つの単語を使用できます。 中心的な構文は次のようになります。

    ```
    New-AzResourceGroup -name <name> -location <location>
    ```

2. リソース グループを確認します。 
  
    ```
    Get-AzResourceGroup
    ```

3. リソース グループを削除します。 プロンプトが表示されたら確認します。 

    ```
    Remove-AzResourceGroup -Name Test
    ```