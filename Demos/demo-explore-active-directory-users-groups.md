---
ms.openlocfilehash: 86c9487720b5a2ee7c2f08c5c2600a9a6f4843e8
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857594"
---
# <a name="demonstration-explore-actve-directory-users-and-groups"></a>デモ:Active Directory のユーザーとグループを詳しく見る

>**注**:サブスクリプションによっては、「Active Directory」ブレードのすべての領域を利用できない場合があります。

## <a name="determine-domain-information"></a>ドメイン情報を決定する

1. Azure portal にアクセスし、「**Azure Active Directory**」ブレードに移動します。
2. 使用可能なドメイン名をメモします。 例: usergmail.onmicrosoft.com。

## <a name="explore-user-accounts"></a>ユーザー アカウントを詳しく見る

1. 「**ユーザー**」ブレードを選択します。
2. **[ 新規ユーザー]** を選択します。 
3. 「**新しいゲスト ユーザー**」を作成する選択に注意してください。
4. **[新しいユーザー]** を作成します。 ドメインを置き換え 

    + **名前**: *Chris Green*
    + **アドレス**: *chris@your ドメイン*
    + **プロファイル情報**:姓と名を入力。 
    + **ディレクトリ ロール** - *ユーザー*。

5. **[ユーザー設定]** ブレードをレビューします。
6. **[監査ログ]** ブレードをレビューします。

## <a name="explore-group-accounts"></a>グループ アカウントを詳しく見る

1. 「**グループ**」ブレードを選択します。
2. 「**新しいグループ**」を追加します。 

    + **グループの種類**: *セキュリティ*
    + **グループ名**: *マネージャー*
    + **メンバーシップの種類**:*割り当て済み*
    + **メンバー**:グループに *Chris Green* を追加する。 

3. **[設定]** の下で **[全般]** ブレードをレビューします。
4. **[アクティビティ]** の下で **[監査ログ]** ブレードをレビューします。

## <a name="explore-powershell-for-group-management"></a>グループ管理のための PowerShell を詳しく見る

1. **開発者** という新しいグループを作成します。

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

4. グループにユーザーを追加します。 **groupObjectId** および **userObjectId** を置き換えます。

    ```
    Add-AzADGroupMember -MemberUserPrincipalName ""myemail@domain.com"" -TargetGroupDisplayName ""MyGroupDisplayName""
    ```

5. グループのメンバーを確認します。 **groupObjectId** を置き換えます。

    ```
    Get-AzADGroupMember -GroupDisplayName "MyGroupDisplayName"
    ```