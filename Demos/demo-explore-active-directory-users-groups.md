# デモ: Active Directory のユーザーとグループを詳しく見る

> **注**: サブスクリプションによっては、Active Directory ブレードのすべての領域を利用できるわけではありません。

## ドメイン情報を決定する

1. Azure portal にアクセスし、「**Azure Active Directory**」ブレードに移動します。
2. 使用可能なドメイン名をメモします。例: usergmail.onmicrosoft.com。

## ユーザー アカウントを詳しく見る

1. 「**ユーザー**」ブレードを選択します。
2. 「**新しいユーザー**」を選択します。 
3. 「**新しいゲスト ユーザー**」を作成する選択に注意してください。
4. 「**新しいユーザー**」 を作成します。ドメインを置き換え。 

    + 「**名前**」: *Chris Green*
    + **住所**: *chris@your domain*
    + **プロファイル情報**: 姓と名を入力。 
    + **ディレクトリ ロール** - *ユーザー*。

5. 「**ユーザー設定**」 ブレードをレビューします。
6. 「**監査ログ**」 ブレードをレビューします。

## グループ アカウントを詳しく見る

1. 「**グループ**」ブレードを選択します。
2. 「**新しいグループ**」を追加します。 

    + 「**グループ タイプ**」: *セキュリティ*
    + 「**グループ名**」: *マネージャー*
    + 「**メンバーシップタイプ**」: *Assigned*
    + 「**メンバー**」: グループに*Chris Green*を追加する。 

3. 「**設定**」 の下で 「**全般**」 ブレードをレビューします。
4. 「**アクティビティ**」 の下で 「**監査ログ**」 ブレードをレビューします。

## グループ管理のためのPowerShellを詳しく見る

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

4. グループにユーザーを追加します。**groupObjectId** および **userObjectId** を置き換えます。

    ```
    Add-AzADGroupMember -MemberUserPrincipalName ""myemail@domain.com"" -TargetGroupDisplayName ""MyGroupDisplayName""
    ```

5. グループのメンバーを確認します。**groupObjectId** を置き換えます。

    ```
    Get-AzADGroupMember -GroupDisplayName "MyGroupDisplayName"
    ```
