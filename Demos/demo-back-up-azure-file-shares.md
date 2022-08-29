# <a name="demonstration-back-up-azure-file-shares"></a>デモ: Azure ファイル共有をバックアップする

このデモでは、Azure Portal でのファイル共有のバックアップについて詳しく見ていきます。

>**注:**  このデモには、Azure ファイル共有と、コンテナーが使用できるストレージ アカウントが必要です。 

## <a name="create-a-recovery-services-vault"></a>Recovery Services コンテナーを作成する

1. Azure portal で Recovery Services と入力し、**Recovery Services コンテナー**をクリックします。
3. **[追加]** をクリックします。
4. **名前**、**サブスクリプション**、**リソース グループ**、および**場所**を指定します。 
5. 新しいコンテナーは、ファイル共有と同じ場所にあることが必要です。 
5. Click <bpt id="p1">**</bpt>Create<ept id="p1">**</ept>. It can take several minutes for the Recovery Services vault to be created. Monitor the status notifications in the upper right-hand area of the portal. Once your vault is created, it appears in the list of Recovery Services vaults. 
6. 数分後にコンテナーが追加されない場合は、 **[更新]** をクリックします。

## <a name="configure-the-vault"></a>コンテナーの構成

1. Recovery Services コンテナーを開きます。 
2. **[バックアップ]** をクリックし、新しいバックアップ インスタンスを作成します。 
3. **[ワークロードはどこで実行されていますか?]** ドロップダウン メニューから、 **[Azure]** を選択します。
4. **[何をバックアップしますか?]** メニューから、 **[Azure FileShare]** を選択します。
5. [**バックアップ**] をクリックします。
6. From the list of Storage accounts, <bpt id="p1">**</bpt>select a storage account<ept id="p1">**</ept>, and click <bpt id="p2">**</bpt>OK<ept id="p2">**</ept>. Azure searches the storage account for files shares that can be backed up. If you recently added your file shares, allow a little time for the file shares to appear.
7. [ファイル共有] リストから、バックアップする**ファイル共有を 1 つ以上選択**し、 **[OK]** をクリックします。
8. On the Backup Policy page, choose <bpt id="p1">**</bpt>Create New backup policy<ept id="p1">**</ept> and provide Name, Schedule, and Retention information. Click <bpt id="p1">**</bpt>OK<ept id="p1">**</ept>.
9. バックアップの構成が完了したら、 **[バックアップを有効にする]** をクリックします。 

## <a name="explore-recovery-services-vault-information"></a>回復サービスのコンテナー情報を詳しく見る

1. Explore the <bpt id="p1">**</bpt>Backup items<ept id="p1">**</ept> blade. There is information on backed up items and replicated items.
2. Explore the <bpt id="p1">**</bpt>Backup policies<ept id="p1">**</ept> blade. You can add or delete backup policies. 
3. Explore the <bpt id="p1">**</bpt>Backup jobs<ept id="p1">**</ept> blade. Here you can review the status of your backup jobs.