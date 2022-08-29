# <a name="demonstration-explore-actve-directory-users-and-groups"></a>デモ:Active Directory のユーザーとグループを詳しく見る

>**注**: サブスクリプションによっては、[Active Directory] ブレードの一部の領域を利用できない場合があります。

## <a name="determine-domain-information"></a>ドメイン情報を決定する

1. Azure portal にアクセスし、**[Azure Active Directory]** ブレードに移動します。
2. Make a note of your available domain name. For example, usergmail.onmicrosoft.com.

## <a name="explore-user-accounts"></a>ユーザー アカウントを詳しく見る

1. **[ユーザー]** ブレードを選択します。
2. **[ 新規ユーザー]** を選択します。 
3. **[新しいゲスト ユーザー]** を作成する選択に注意してください。
4. Create a <bpt id="p1">**</bpt>New user<ept id="p1">**</ept>. Replace your domain. 

    + **名前**: *Chris Green*
    + **アドレス**: *chris@your ドメイン*
    + **プロファイル情報**:姓と名を入力。 
    + **ディレクトリ ロール** - *ユーザー*。

5. **[ユーザー設定]** ブレードをレビューします。
6. **[監査ログ]** ブレードをレビューします。

## <a name="explore-group-accounts"></a>グループ アカウントを詳しく見る

1. **[グループ]** ブレードを選択します。
2. **[新しいグループ]** を追加します。 

    + **グループの種類**: *セキュリティ*
    + **グループ名**: *マネージャー*
    + **メンバーシップの種類**:*割り当て済み*
    + **メンバー**:グループに*Chris Green*を追加する。 

3. **[設定]** の下で **[全般]** ブレードをレビューします。
4. **[アクティビティ]** の下で **[監査ログ]** ブレードをレビューします。

## <a name="explore-powershell-for-group-management"></a>グループ管理のための PowerShell を詳しく見る

1. **開発者**という新しいグループを作成します。

    ```
    New-AzADGroup -DisplayName Developers -MailNickname Developers
    ```

2. **Developers** グループ **ObjectId** を取得します。

    ```
    Get-AzADGroup
    ```

3. 追加するメンバーのユーザー **ObjectId** を取得します。

    ```
    Get-AzADUser
    ```

4. Add the user to the group. Replace <bpt id="p1">**</bpt>groupObjectId<ept id="p1">**</ept> and <bpt id="p2">**</bpt>userObjectId<ept id="p2">**</ept>.

    ```
    Add-AzADGroupMember -MemberUserPrincipalName ""myemail@domain.com"" -TargetGroupDisplayName ""MyGroupDisplayName""
    ```

5. Verify the members of the group. Replace <bpt id="p1">**</bpt>groupObjectId<ept id="p1">**</ept>.

    ```
    Get-AzADGroupMember -GroupDisplayName "MyGroupDisplayName"
    ```