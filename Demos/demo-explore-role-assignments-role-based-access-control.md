# <a name="demonstration-explore-role-assignments-and-role-based-access-control"></a>デモ: ロールの割り当てとロールベースのアクセス制御を調べる

## <a name="locate-access-control-blade"></a>アクセス制御ブレードを探す

1. Access the Azure portal, and select a resource group. Make a note of what resource group you use. 
2. **[アクセス制御 (IAM)]** ブレードを選択します。 
3. このブレードは、アクセスを制御できるように、さまざまなリソースで使用できます。

## <a name="review-role-permissions"></a>ロールのアクセス許可を確認する

1. **[ロール]** タブ (上部) を選択します。
2. 使用できる多数の組み込みロールを確認します。
3. ロールをダブルクリックし、**[アクセス許可]** (上部) を選択します。
4. そのロールの**読み取り、書き込み、削除**のアクションが表示されるまで、ロールの詳細表示を続行します。
5. **[アクセス制御 (IAM)]** ブレードに戻ります。

## <a name="add-a-role-assignment"></a>ロールの割り当てを追加する

1. **[ロールの割り当ての追加]** を選択します。 

    + **ロール**:*所有者*
    + **Select**:*マネージャー*
    + 変更内容を**保存**します。 

2. **[アクセスの確認]** を選択します。
3. Chris Green を**検索**します。
4. 彼はマネージャー グループの一部であり、所有者でもあります。 
5. **割り当てを拒否**することもできます。 

## <a name="explore-powershell-commands"></a>PowerShell コマンドについて調べる

1. Azure Cloud Shell を開きます。
2. [PowerShell] ドロップダウンを選択します。
3. ロール定義を一覧表示します。

    ```PowerShell
    Get-AzRoleDefinition | FT Name, Description
    ```

4. ロールのアクションを一覧表示します。

    ```PowerShell
    Get-AzRoleDefinition owner | FL Actions, NotActions
    ```

5. ロールの割り当てを一覧表示します。

    ```PowerShell
    Get-AzRoleAssignment -ResourceGroupName <resource group name>
    ```