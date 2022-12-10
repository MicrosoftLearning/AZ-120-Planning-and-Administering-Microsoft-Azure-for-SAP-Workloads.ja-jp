# <a name="demonstration-create-and-delete-resource-groups"></a>デモンストレーション: リソース グループの作成と削除

>**注**:リソースのロックを管理できるのは、所有者とユーザー アクセス管理者のロールだけです。

## <a name="manage-resource-groups-in-the-portal"></a>ポータルでのリソース グループの管理

1. Azure ポータルにアクセスします。
1. リソース グループを作成します。 このリソース グループの名前を覚えておいてください。 
1. リソース グループの **[設定]** ブレードで、**[ロック]** を選択します。
1. ロックを追加するには、**[追加]** を選択します。 親レベルでロックを作成する場合は、親を選択します。 現在選択されているリソースは、ロックを親から継承します。 たとえば、リソース グループをロックすることで、グループのリソースすべてにロックを適用することができます。
1. ロックに**名前**と**ロックの種類**を指定します。 必要に応じて、ロックについて説明したメモを追加することができます。
1. ロックを削除するには、省略記号 (...) を選択し、表示されるオプションから **[削除]** を選択します。

## <a name="manage-resource-groups-with-powershell"></a>PowerShell を使用してリソース グループを管理する

1. Cloud Shell にアクセスする。
2. リソース ロックを作成し、アクションを確認します。

    ```
        New-AzResourceLock -LockName <lockName> -LockLevel CanNotDelete -ResourceGroupName <resourceGroupName>
    ```

3. リソース ロック情報を表示します。 ロックを削除するために、次の手順で使用する LockId に注意してください。

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