# <a name="demonstration-create-storage-accounts"></a>デモ: ストレージ アカウントを作成する

## <a name="create-a-storage-account-in-the-portal"></a>ポータルでストレージ アカウントを作成する

1.  In the Azure portal, select <bpt id="p1">**</bpt>All services<ept id="p1">**</ept>. In the list of resources, type Storage Accounts. As you begin typing, the list filters based on your input. Select <bpt id="p1">**</bpt>Storage Accounts<ept id="p1">**</ept>.
2.  表示された [ストレージ アカウント] ウィンドウで **[追加]** を選択します。
3.  ストレージ アカウントを作成する **[サブスクリプション]** を選択します。
4.  Under the Resource group field, select <bpt id="p1">**</bpt>Create new<ept id="p1">**</ept>. Enter a name for your new resource group.
5.  Enter a <bpt id="p1">**</bpt>name<ept id="p1">**</ept> for your storage account. The name you choose must be unique across Azure. The name also must be between 3 and 24 characters in length, and can include numbers and lowercase letters only.
6.  ストレージ アカウントの **[場所]** を選択するか、既定の場所を使用します。
7.  以下のフィールドは既定値に設定されたままにします。

     * デプロイ モデル: **Resource Manager**
     * パフォーマンス: **Standard**
     * アカウント種類:**ストレージ V2(汎用 v2)**
     * レプリケーション:**ローカル冗長ストレージ (LRS)**
     * アクセス層:**ホット**

8.  **[確認および作成]** を選択して、ストレージ アカウントの設定を確認し、アカウントを作成します。
9.  **［作成］** を選択します

## <a name="create-a-storage-account-using-powershell"></a>PowerShell を使用したストレージ アカウントの作成

Azure Portal で **[すべてのサービス]** を選択します。

```PowerShell
Get-AzLocation | select Location 
$location = "westus" 
$resourceGroup = "storage-demo-resource-group" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "storagedemo" -Location $location -SkuName Standard_LRS -Kind StorageV2 
```

## <a name="create-a-storage-account-using-azure-cli"></a>Azure CLI を使用してストレージ アカウントを作成する

リソースの一覧で「ストレージ アカウント」と入力します。

```PowerShell
az group create --name storage-resource-group --location westus 
az account list-locations --query "[].{Region:name}" --out table 
az storage account create --name storagedemo --resource-group storage-resource-group --location westus --sku Standard_LRS --kind StorageV2 
```