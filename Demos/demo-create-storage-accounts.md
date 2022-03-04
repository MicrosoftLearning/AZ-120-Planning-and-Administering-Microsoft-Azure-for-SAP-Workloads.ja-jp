---
ms.openlocfilehash: 0106fd5b2837ad870b148f4ecebb4dcc690b89ca
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857597"
---
# <a name="demonstration-create-storage-accounts"></a>デモ: ストレージ アカウントを作成する

## <a name="create-a-storage-account-in-the-portal"></a>ポータルでストレージ アカウントを作成する

1.  Azure Portal で **[すべてのサービス]** を選択します。 リソースの一覧で「ストレージ アカウント」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[ストレージ アカウント]** を選択します。
2.  表示された [ストレージ アカウント] ウィンドウで **[追加]** を選択します。
3.  ストレージ アカウントを作成する **[サブスクリプション]** を選択します。
4.  [リソース グループ] フィールドの下の **[新規作成]** を選択します。 新しいリソース グループの名前を入力します。
5.  ストレージ アカウントの **名前** を入力します。 選択する名前は Azure 全体で一意である必要があります。 また、名前の長さは 3 から 24 文字とし、数字と小文字のみを使用できます。
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

PowerShell を使用してストレージ アカウントを作成するには、次のコードを使用します。 要件に合わせてストレージの種類と名前を入れ替えます。

```PowerShell
Get-AzLocation | select Location 
$location = "westus" 
$resourceGroup = "storage-demo-resource-group" 
New-AzResourceGroup -Name $resourceGroup -Location $location 
New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "storagedemo" -Location $location -SkuName Standard_LRS -Kind StorageV2 
```

## <a name="create-a-storage-account-using-azure-cli"></a>Azure CLI を使用してストレージ アカウントを作成する

Azure CLI を使用して、ストレージ アカウントを作成するには、次のコードを使用します。 要件に合わせてストレージの種類と名前を変更します。

```PowerShell
az group create --name storage-resource-group --location westus 
az account list-locations --query "[].{Region:name}" --out table 
az storage account create --name storagedemo --resource-group storage-resource-group --location westus --sku Standard_LRS --kind StorageV2 
```