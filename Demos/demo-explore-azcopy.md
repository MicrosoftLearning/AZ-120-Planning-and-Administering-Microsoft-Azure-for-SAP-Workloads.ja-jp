# <a name="demonstration-explore-azcopy"></a>デモ: AzCopy を調べる

## <a name="download-azcopy"></a>AzCopy をダウンロードする

First, download the AzCopy V10 executable file to any directory on your computer. AzCopy V10 is just an executable file, so there's nothing to install.

- [Windows 64 ビット](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Windows 32 ビット](https://aka.ms/downloadazcopy-v10-windows-32bit) (zip)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

These files are compressed as a zip file (Windows and Mac) or a tar file (Linux). To download and decompress the tar file on Linux, see the documentation for your Linux distribution.

## <a name="run-azcopy"></a>AzCopy を実行する

For convenience, consider adding the directory location of the AzCopy executable to your system path for ease of use. That way you can type <ph id="ph1">`azcopy`</ph> from any directory on your system.

AzCopy ディレクトリをご自分のパスに追加しないことを選択した場合、実際の AzCopy 実行可能ファイルの場所にディレクトリを変更し、Windows PowerShell コマンド プロンプトで「`azcopy`」または「`.\azcopy`」と入力する必要があります。

まず、お使いのコンピューター上の任意のディレクトリに AzCopy V10 実行可能ファイルをダウンロードします。 

## <a name="authorize-azcopy"></a>AzCopy を承認する

認証資格情報は、Azure Active Directory (AD) または Shared Access Signature (SAS) トークンを使用して提供できます。

次の表をガイドとして使用してください。

| ストレージの種類 | 現在サポートされている認証方法 |
|--|--|
|**Blob Storage** | Azure AD および SAS |
|**BLOB ストレージ (階層型名前空間)** | Azure AD および SAS |
|**File Storage** | SAS のみ |

### <a name="option-1-use-azure-active-directory"></a>オプション 1: Azure Active Directory を使用する

AzCopy V10 は単に実行可能ファイルなので、インストールするものはありません。  

> <bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> In the current release, if you plan to copy blobs between storage accounts, you'll have to append a SAS token to each source URL. You can omit the SAS token only from the destination URL. For examples, see <bpt id="p1">[</bpt>Copy blobs between storage accounts<ept id="p1">](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10#transfer-data)</ept>.

Azure AD を使用してアクセスを承認するには、「[AzCopy と Azure Active Directory (Azure AD) を使用して BLOB へのアクセスを承認する](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-authorize-azure-active-directory)」を参照してください。

### <a name="option-2-use-a-sas-token"></a>オプション 2:SAS トークンを使用する

AzCopy コマンドで使用する各コピー元または各コピー先の URL に SAS トークンを追加できます。

This example command recursively copies data from a local directory to a blob container. A fictitious SAS token is appended to the end of the container URL.

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

SAS トークンの詳細とその取得方法については、「[Shared Access Signatures (SAS) を使用して Azure Storage リソースへの制限付きアクセスを許可する](https://docs.microsoft.com/azure/storage/common/storage-sas-overview)」を参照してください。

> <bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> The <bpt id="p2">[</bpt>Secure transfer required<ept id="p2">](storage-require-secure-transfer.md)</ept> setting of a storage account determines whether the connection to a storage account is secured with Transport Layer Security (TLS). This setting is enabled by default.   

## <a name="explore-the-help"></a>ヘルプを確認する

1. コマンドの一覧を表示するには、「`azcopy -h`」と入力し、ENTER キーを押します。
2. 特定のコマンドの情報を知るには、コマンドの名前 (例: `azcopy list -h`) を含めて、Enter キーを押してください。

![インライン ヘルプ](Images/azcopy-inline-help.png)

## <a name="download-a-blob-from-blob-storage-to-the-file-system"></a>BLOB Storage からファイル システムへ BLOB をダウンロードする

>**注:**
>- This example requires an Azure storage account with a blob container and blob file. You will also need to capture parameters in a text editor like Notepad.
>- This example encloses path arguments with single quotes (''). Use single quotes in all command shells except for the Windows Command Shell (cmd.exe). If you're using a Windows Command Shell (cmd.exe), enclose path arguments with double quotes ("") instead of single quotes ('').

1. Azure ポータルにアクセスします。
2. ダウンロードしたい BLOB を使用してストレージ アカウントにアクセスします。
3. 目的の BLOB へドリルダウンし、ファイルの**プロパティ**を表示します。
4. これらのファイルは、zip ファイル (Windows および Mac) または tar ファイル (Linux) として圧縮されます。
5. Linux 上で tar ファイルをダウンロードして圧縮を解除するには、お使いの Linux ディストリビューションのドキュメントを参照してください。
6. 値を使用してコマンドを作成します。

    ```
    azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'
    ```
    
7. エラーが発生した場合は、注意深く読んで修正します。
8. BLOB がローカル ディレクトリにダウンロードされたことを確認します。 

## <a name="upload-files-to-azure-blob-storage"></a>Azure Blob Storage にファイルをアップロードする

>**注:**
>- この例は前の例の続きで、ファイルを含むローカル ディレクトリが必要です。
>- This example encloses path arguments with single quotes (''). Use single quotes in all command shells except for the Windows Command Shell (cmd.exe). If you're using a Windows Command Shell (cmd.exe), enclose path arguments with double quotes ("") instead of single quotes ('').

1. コマンドの *<local-file-path>* ソースは、ファイルを含むローカル ディレクトリになります。 
2. The <bpt id="p1">*</bpt>https://&lt;storage-account-name&gt;.<ph id="ph1">&lt;blob or dfs&gt;</ph>.core.windows.net/&lt;container-name&gt;/&lt;blob-name&gt;<ept id="p1">*</ept> destination for the command will the blob URL used in the previous example. Be sure to remove the filename, just include the storage account and container. 
3. 値を使用してコマンドを作成します。

    ```
    azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'
    ```

5. エラーが発生した場合は、注意深く読んで修正します。
6. ローカル ファイルが Azure コンテナーにコピーされたことを確認します。 
7. サブディレクトリとパターンの一致を再帰処理するスイッチがあることに注意してください。
