# デモ: ファイル共有とスナップショットを作成および管理する

> **注**: これらの手順では、ストレージ アカウントが必要です。 

## ファイル共有を作成してファイルをアップロードする

1. ストレージ アカウントにアクセスし、「**ファイル**」 をクリックします。
2. 「**+ ファイル共有**」 をクリックして、新しいファイル共有に**名前** と**クォータ**を付与します。
3. ファイル共有の作成後、ファイルを**アップロード**します。 
4. **ディレクトリの追加**、**共有の削除**、および**クォータ**を編集する機能に注目してください。

## スナップショットを管理する

1. ファイル共有にアクセスします。
1. 「**スナップショットの作成**」 を選択します。
1. 「**スナップショットの表示**」 を選択し、スナップショットが作成されたことを確認します。
1. スナップショットをクリックし、アップロードしたファイルが含まれていることを確認します。
1. スナップショットの一部であるファイルをクリックし、「**ファイル プロパティ**」 を確認します。 
1. スナップショット ファイルの**ダウンロード**および**復元**の選択に注目してください。 
1. ファイル共有にアクセスし、以前にアップロードしたファイルを削除します。
1. ファイルをスナップショットから**復元**します。 
 
## ファイル共有を作成する (PowerShell)

1. ストレージ アカウントとキーのコンテキストを作成します。コンテキストは、ストレージ アカウント名とアカウント キーをカプセル化します。

    ```PowerShell
    $storageContext = New-AzStorageContext storage-account-name storage-account-key
    ```

2. ファイル共有を作成。ファイル共有の名前はすべて小文字にする必要があります。

    ```PowerShell
    $share = New-AzStorageShare logs -Context $storageContext
    ```

## ファイル共有をマウントする (PowerShell)

1. 通常の (つまり、昇格されていない) PowerShell セッションから次のコマンドを実行して、Azure ファイル共有をマウントします。**your-resource-group-name**、**your-storage-account-name**、**your-file-share-name**、および **desired-drive-letter** を適切な情報に置き換えることを忘れないでください。

    ```PowerShell
    $resourceGroupName = "your-resource-group-name"
    $storageAccountName = "your-storage-account-name"
    $fileShareName = "your-file-share-name"

    # These commands require you to be logged into your Azure account, run Login-AzAccount if you haven't
    # already logged in.
    $storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
    $storageAccountKeys = Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName
    $fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object { 
        $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
    }

    if ($fileShare -eq $null) {
        throw [System.Exception]::new("Azure file share not found")
    }

    # The value given to the root parameter of the New-PSDrive cmdlet is the host address for the storage account, 
    # storage-account.file.core.windows.net for Azure Public Regions. $fileShare.StorageUri.PrimaryUri.Host is 
    # used because non-Public Azure regions, such as sovereign clouds or Azure Stack deployments, will have different 
    # hosts for Azure file shares (and other storage resources).
    $password = ConvertTo-SecureString -String $storageAccountKeys[0].Value -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "AZURE\$($storageAccount.StorageAccountName)", $password
    New-PSDrive -Name desired-drive-letter -PSProvider FileSystem -Root "\\$($fileShare.StorageUri.PrimaryUri.Host)\$($fileShare.Name)" -Credential $credential -Persist
    ```

2. 終了したら、次のコマンドを実行してファイル共有をマウント解除できます。

    ```PowerShell
    Remove-PSDrive -Name desired-drive-letter
    ```
