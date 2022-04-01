---
ms.openlocfilehash: 18555523da5c295a9e0d961f9339e48fffbc6ae1
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857593"
---
# <a name="demonstration-explore-azcopy"></a>デモ: AzCopy を調べる

## <a name="download-azcopy"></a>AzCopy をダウンロードする

まず、お使いのコンピューター上の任意のディレクトリに AzCopy V10 実行可能ファイルをダウンロードします。 AzCopy V10 は単に実行可能ファイルなので、インストールするものはありません。

- [Windows 64 ビット](https://aka.ms/downloadazcopy-v10-windows) (zip)
- [Windows 32 ビット](https://aka.ms/downloadazcopy-v10-windows-32bit) (zip)
- [Linux x86-64](https://aka.ms/downloadazcopy-v10-linux) (tar)
- [macOS](https://aka.ms/downloadazcopy-v10-mac) (zip)

これらのファイルは、zip ファイル (Windows および Mac) または tar ファイル (Linux) として圧縮されます。 Linux 上で tar ファイルをダウンロードして圧縮を解除するには、お使いの Linux ディストリビューションのドキュメントを参照してください。

## <a name="run-azcopy"></a>AzCopy を実行する

利便性のため、AzCopy 実行可能ファイルのディレクトリの場所をご自分のシステム パスに追加して使いやすくすることを検討してください。 そうすると、ご使用のシステム上にある任意のディレクトリから「`azcopy`」を入力できます。

AzCopy ディレクトリをご自分のパスに追加しないことを選択した場合、実際の AzCopy 実行可能ファイルの場所にディレクトリを変更し、Windows PowerShell コマンド プロンプトで「`azcopy`」または「`.\azcopy`」と入力する必要があります。

ご自分の Azure Storage アカウントの所有者であっても、データへのアクセス許可が自動的に割り当てられるわけではありません。 AzCopy を使用して意味のある動作を行う前に、ストレージ サービスに認証資格情報を提供する方法を決定する必要があります。 

## <a name="authorize-azcopy"></a>AzCopy を承認する

認証資格情報は、Azure Active Directory (AD) または Shared Access Signature (SAS) トークンを使用して提供できます。

次の表をガイドとして使用してください。

| ストレージの種類 | 現在サポートされている認証方法 |
|--|--|
|**Blob Storage** | Azure AD および SAS |
|**BLOB ストレージ (階層型名前空間)** | Azure AD および SAS |
|**File Storage** | SAS のみ |

### <a name="option-1-use-azure-active-directory"></a>オプション 1: Azure Active Directory を使用する

このオプションは、BLOB ストレージでのみ使用できます。 Azure Active Directory を使用すると、各コマンドに SAS トークンを追加する代わりに、資格情報を 1 回入力するだけで済みます。  

> **注:**  現在のリリースでは、ストレージ アカウント間で BLOB をコピーする場合は、各ソース URL に SAS トークンを追加する必要があります。 コピー先 URL からのみ、SAS トークンを省略できます。 例については、「[ストレージ アカウント間で BLOB をコピーする](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-v10#transfer-data)」をご覧ください。

Azure AD を使用してアクセスを承認するには、「[AzCopy と Azure Active Directory (Azure AD) を使用して BLOB へのアクセスを承認する](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-authorize-azure-active-directory)」を参照してください。

### <a name="option-2-use-a-sas-token"></a>オプション 2:SAS トークンを使用する

AzCopy コマンドで使用する各コピー元または各コピー先の URL に SAS トークンを追加できます。

この例のコマンドでは、ローカル ディレクトリから BLOB コンテナーにデータが繰り返しコピーされます。 架空の SAS トークンが、コンテナー URL の末尾に追加されます。

```azcopy
azcopy copy "C:\local\path" "https://account.blob.core.windows.net/mycontainer1/?sv=2018-03-28&ss=bjqt&srt=sco&sp=rwddgcup&se=2019-05-01T05:01:17Z&st=2019-04-30T21:01:17Z&spr=https&sig=MGCXiyEzbtttkr3ewJIh2AR8KrghSy1DGM9ovN734bQF4%3D" --recursive=true
```

SAS トークンの詳細とその取得方法については、「[Shared Access Signatures (SAS) を使用して Azure Storage リソースへの制限付きアクセスを許可する](https://docs.microsoft.com/azure/storage/common/storage-sas-overview)」を参照してください。

> **注:**  ストレージ　アカウントの[安全な転送が必須](storage-require-secure-transfer.md)の設定は、ストレージ　アカウントへの接続がトランスポート層セキュリティ (TLS) で保護されているかどうかを決定します。 既定では、この設定は有効になっています。   

## <a name="explore-the-help"></a>ヘルプを確認する

1. コマンドの一覧を表示するには、「`azcopy -h`」と入力し、ENTER キーを押します。
2. 特定のコマンドの情報を知るには、コマンドの名前 (例: `azcopy list -h`) を含めて、Enter キーを押してください。

![インライン ヘルプ](Images/azcopy-inline-help.png)

## <a name="download-a-blob-from-blob-storage-to-the-file-system"></a>BLOB Storage からファイル システムへ BLOB をダウンロードする

>**注:**
>- この例では、BLOB コンテナーと BLOB ファイルを含む Azure ストレージ アカウントが必要になります。 また、Notepad などのテキスト エディターにパラメーターをキャプチャする必要があります。
>- この例では、パス引数を単一引用符 ('') で囲んでいます。 Windows コマンド シェル (cmd.exe) を除き、すべてのコマンド シェルで単一引用符を使用します。 Windows コマンド シェル (cmd.exe) を使用している場合は、単一引用符 ('') ではなく、二重引用符 ("") でパス引数を囲みます。

1. Azure ポータルにアクセスします。
2. ダウンロードしたい BLOB を使用してストレージ アカウントにアクセスします。
3. 目的の BLOB へドリルダウンし、ファイルの **プロパティ** を表示します。
4. **[URL]** の情報をコピーします。 これは、ソース パス *https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>* になります。
5. ローカル宛先ディレクトリを検索します。 これは、 *<local-file-path>* 値になります。 ファイル名も必要です。
6. 値を使用してコマンドを作成します。

    ```
    azcopy copy 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-path>' '<local-file-path>'
    ```
    
7. エラーが発生した場合は、注意深く読んで修正します。
8. BLOB がローカル ディレクトリにダウンロードされたことを確認します。 

## <a name="upload-files-to-azure-blob-storage"></a>Azure Blob Storage にファイルをアップロードする

>**注:**
>- この例は前の例の続きで、ファイルを含むローカル ディレクトリが必要です。
>- この例では、パス引数を単一引用符 ('') で囲んでいます。 Windows コマンド シェル (cmd.exe) を除き、すべてのコマンド シェルで単一引用符を使用します。 Windows コマンド シェル (cmd.exe) を使用している場合は、単一引用符 ('') ではなく、二重引用符 ("") でパス引数を囲みます。

1. コマンドの *<local-file-path>* ソースは、ファイルを含むローカル ディレクトリになります。 
2. コマンドの *https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>* のコピー先は、前の例で使用された BLOB の URL になります。 ファイル名を削除し、ストレージ アカウントとコンテナーのみを含めます。 
3. 値を使用してコマンドを作成します。

    ```
    azcopy copy '<local-file-path>' 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>/<blob-name>'
    ```

5. エラーが発生した場合は、注意深く読んで修正します。
6. ローカル ファイルが Azure コンテナーにコピーされたことを確認します。 
7. サブディレクトリとパターンの一致を再帰処理するスイッチがあることに注意してください。
