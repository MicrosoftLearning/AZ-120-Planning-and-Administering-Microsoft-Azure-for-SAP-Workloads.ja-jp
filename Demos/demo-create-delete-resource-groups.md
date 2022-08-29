# <a name="demonstration-create-and-delete-resource-groups"></a>デモンストレーション: リソース グループの作成と削除

>**注**:リソースのロックを管理できるのは、所有者とユーザー アクセス管理者のロールだけです。

## <a name="manage-resource-groups-in-the-portal"></a>ポータルでのリソース グループの管理

1. Azure ポータルにアクセスします。
1. Create a resource group. Remember the name of this resource group. 
1. リソース グループの **[設定]** ブレードで、**[ロック]** を選択します。
1. To add a lock, select <bpt id="p1">**</bpt>Add<ept id="p1">**</ept>. If you want to create a lock at a parent level, select the parent. The currently selected resource inherits the lock from the parent. For example, you could lock the resource group to apply a lock to all its resources.
1. Give the lock a <bpt id="p1">**</bpt>name<ept id="p1">**</ept> and <bpt id="p2">**</bpt>lock type<ept id="p2">**</ept>. Optionally, you can add notes that describe the lock.
1. ロックを削除するには、省略記号 (...) を選択し、表示されるオプションから **[削除]** を選択します。

## <a name="manage-resource-groups-with-powershell"></a>PowerShell を使用してリソース グループを管理する

1. Cloud Shell にアクセスする。
2. リソース ロックを作成し、アクションを確認します。

    ```
        New-AzResourceLock -LockName <lockName> -LockLevel CanNotDelete -ResourceGroupName <resourceGroupName>
    ```

3. View resource lock information. Notice the LockId that will be used in the next step to delete the lock.

    ```
        Get-AzResourceLock
    ```

4. リソース ロックを削除し、アクションを確認します。 

    ```
        Remove-AzResourceLock -LockName <Name> -ResourceGroupName <Resource Group>
    ```

5. リソース ロックが削除されたことを確認します。


    ```
        Get-AzResourceLock
    ```

>**注:**  認定試験では、リソース ロックの設定、リソース グループ間でのリソースの移動、リソース グループの削除なども範囲に含まれます。