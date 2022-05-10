---
ms.openlocfilehash: 64a17311a61ca8622ac5e6203ec5b63a0c169f57
ms.sourcegitcommit: 30dae3c49fe96a790479d08844a71fcb7851aa46
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/15/2022
ms.locfileid: "139867887"
---

# <a name="az-120-lab-prerequisites"></a>AZ 120:ラボの前提条件

## <a name="vcpu-core-requirements"></a>vCPU コア要件

-   このコースの最後のラボを終了するには、このラボにデプロイされる Azure VM が存在する可用性ゾーンをサポートする Azure リージョンで利用可能な、少なくとも 28 の vCPUを持つ Microsoft Azure サブスクリプションが必要です。

    -   4 x Standard_DS1_v2 (各 1 vCPU) = 4

    -   6 x Standard_D4s_v3 (各 4 vCPU) = 24

    > **注**:リソースのデプロイには、**米国東部** または **米国東部 2** リージョンの使用を検討してください。

    > **注**:可用性ゾーンをサポートする Azure リージョンを特定するには、[https://docs.microsoft.com/en-us/azure/availability-zones/az-overview ]<https://docs.microsoft.com/en-us/azure/availability-zones/az-overview> を参照してください

このコースの最初の 3 つのラボの vCPU 要件は低いですが、クォータを増やすプロセスには時間がかかる場合があるため (クォータの増加要求が同じ営業日に完了する場合もあります)、ラボ全体の要件を満たせるようクォータの増加を要求するようお勧めします。

## <a name="before-the-hands-on-lab"></a>実践ラボの前

期間:30 分

### <a name="task-1-validate-sufficient-number-of-vcpu-cores"></a>タスク 1:vCPU コアが十分であることを確認する

1.  <http://portal.azure.com> にある Azure portal で、 

1.  Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注**:現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1.  Azure portal の **[Cloud Shell]** ペインにある PowerShell プロンプトで、次を実行します。`<Azure_region>` は、このラボで使用する対象の Azure リージョンを指定します (例: `eastus`)。

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **注**:Azure リージョンの名前を識別するには、**Cloud Shell** の Bash プロンプトで `(Get-AzLocation).Location` を実行します
   
1.  前の手順で実行したコマンドの出力の現在の値と制限エントリを確認し、Azure リージョンに十分な数の vCPU があることを確認します。

1.  vCPU の数が十分でない場合は、Azure portal でサブスクリプション ブレードに戻り、 **「使用状況 + クォータ」** をクリックします。 

1.  サブスクリプションの **「使用状況 + クォータ」** ブレードで、 **「増量のリクエスト」** をクリックします。

1.  **[問題の説明]** ブレードで、次のように指定します。

    -   問題の種類:**サービスとサブスクリプションの制限 (クォータ)**

    -   サブスクリプション: このラボで使用する Azure サブスクリプションの名前

    -   クォータの種類:**コンピューティング/VM (コア/vCPU) サブスクリプション制限の増加**

1. **[クォータの管理]** をクリックします。

1. [場所] ドロップダウン メニューを使用して、使用する予定の Azure リージョンに結果をフィルター処理します。 **[米国東部]** または **[米国東部 2]** を使用することをお勧めします。

1. **Standard DSv3 Family vCPUs** のクォータの種類を見つけて、[編集] の鉛筆アイコンを選択します。

1. [新しい制限] フィールドで、**40** を指定し、 **[保存して続行]** をクリックします。

1. **Total Regional vCPUs** クォータの種類を見つけて、[鉛筆の編集] を選択します。

1. [新しい制限] フィールドで、**40** を指定し、 **[保存して続行]** をクリックします。

   > **注**:クォータの引き上げ要求は自動的に承認される必要があります。 要求が拒否された場合は、「サポート チケットを開いてクォータの引き上げを要求する」に進みます。
