# デモ: ストレージ アカウントを作成する

## ポータルでストレージ アカウントを作成する

1.  Azure portal で、「**すべてのサービス**」 を選択します。リソースのリストに、ストレージ アカウントを入力します。入力し始めると、入力に基づいてリストがフィルター処理されます。「**ストレージ アカウント**」 を選択します。
2.  表示される ストレージ アカウント ウィンドウで、「**追加**」 を選択します。
3.  ストレージ アカウントを作成する 「**サブスクリプション**」 を選択します。
4.  リソース グループ フィールドで、「**新規作成**」 を選択します。新しいリソース グループの名前を入力します。
5.  ストレージ アカウントの**名前**を入力します。選択する名前は、Azure 全体で一意である必要があります。名前の長さは 3 ～ 24 文字で、数字と小文字のみを含めることができます。
6.  ストレージ アカウントの 「**場所**」 を選択するか、既定の場所を使用します。
7.  これらのフィールドは既定値に設定したままにします。

     * デプロイ　モデル: **リソース マネージャー**
     * パフォーマンス：**標準**
     * アカウント種類：**ストレージ V2(汎用 v2)**
     * レプリケーション：**ローカル リダンダント ストレージ (LRS)**
     * アクセス層: **ホット**

8.  「**レビューと作成**」 を選択して、ストレージ アカウントの設定を確認し、アカウントを作成します。
9.  **作成** を選択します。

## PowerShell を使用して、ストレージ アカウントを作成する

PowerShell を使用してストレージ アカウントを作成するには、次のコードを使用します。要求に合わせてストレージの種類と名前を入れ替えます。

```PowerShell
Get-AzLocation | select Location 
$location = "westus" 
$resourceGroup = "storage-demo-resource-group" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "storagedemo" -Location $location -SkuName Standard_LRS -Kind StorageV2 
```

## Azure CLI を使用してストレージ アカウントを作成します

Azure CLI を使用して、ストレージ アカウントを作成するには、次のコードを使用します。要求に合わせてストレージの種類と名前を変更します。

```PowerShell
az group create --name storage-resource-group --location westus 
az account list-locations --query "[].{Region:name}" --out table 
az storage account create --name storagedemo --resource-group storage-resource-group --location westus --sku Standard_LRS --kind StorageV2 
```
