# <a name="demonstration-explore-azure-blob-storage"></a>デモ: Azure Blob Storage を調べる

>**注**:このデモでは、ストレージ アカウントが必要です。

## <a name="create-a-container"></a>コンテナーの作成

1. Azure portal で、ストレージ アカウントに移動します。
2. ストレージ アカウントの左側のメニューで、**[Blob service]** セクションまでスクロールしてから、**[BLOB]** を選択します。
3. **[+ コンテナー]** ボタンを選択します。
4. Type a <bpt id="p1">**</bpt>Name<ept id="p1">**</ept> for your new container. The container name must be lowercase, must start with a letter or number, and can include only letters, numbers, and the dash (-) character. 
5. Set the level of public access to the container. The default level is Private (no anonymous access).
6. **[OK]** を選択してコンテナーを作成します。

## <a name="upload-a-block-blob"></a>ブロック BLOB をアップロードする

1. Azure Portal で、前のセクションで作成したコンテナーに移動します。
2. Select the container to show a list of blobs it contains. Since this container is new, it won't yet contain any blobs.
3. **[アップロード]** ボタンを選択して、コンテナーに BLOB をアップロードします。
4. **[詳細設定]** セクションを展開します。
5. **[認証タイプ]**、**[BLOB タイプ]**、**[ブロック サイズ]**、**[フォルダーへのアップロード]** 機能に注意してください。
6. 既定の **[認証タイプ]** は、SAS であることに注意してください。
4. ローカル ファイル システムを参照して、ブロック BLOB としてアップロードするファイルを見つけて、**[アップロード]** を選択します。
5. Upload as many blobs as you like in this way. You'll observe that the new blobs are now listed within the container.

## <a name="download-a-block-blob"></a>ブロック BLOB をダウンロードする

ブロック BLOB をダウンロードして、ブラウザーで表示したり、ローカル ファイル システムに保存したりできます。 

1. 前のセクションでアップロードした BLOB の一覧に移動します。
2. ダウンロードする BLOB を右クリックし、 **[ダウンロード]** を選択します。

>**注**:時間がある場合は、Azure Storage Explorer を使用して、Blob Storage を管理する方法を学習してください。 