---
ms.openlocfilehash: 8253f093bc365b521180798165190e879b37a9af
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857595"
---
# <a name="demonstration-create-a-virtual-machine-with-powershell"></a>デモンストレーション: PowerShell を使用して仮想マシンを作成する

このデモでは、PowerShell を使用して仮想マシンを作成します。

## <a name="create-the-virtual-machine"></a>仮想マシンの作成

>**注:** Cloud Shell またはローカル バージョンの PowerShell を使用できます。 

1. Cloud Shell を起動します。
2. このコードを実行します。

```
    # create a resource group
    New-AzResourceGroup -Name myResourceGroup -Location EastUS

    # create the virtual machine 
    # when prompted, provide a username and password to be used as the logon credentials for the VM
    New-AzVm `
        -ResourceGroupName "myResourceGroup" `
        -Name "myVM" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress" `
        -OpenPorts 80,3389
```

## <a name="verify-the-machine-creation-in-the-portal"></a>ポータルでのマシンの作成を確認する

1. ポータルにアクセスし、仮想マシンを表示します。
2. **myVM** が作成されたことを確認します。
3. VM の設定を確認します。 
4. これは、新しい VNet およびサブネットの Windows マシンであることに注意してください。 
5. コマンドがマシンを起動したのに注意してください。 
6. この時点で、ポータルまたは PowerShell を使用して変更を加えることができます。 

## <a name="connect-to-the-virtual-machine"></a>仮想マシンへの接続

1. マシンのパブリック IP アドレスを取得します。

```
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```
2. ローカル マシンから RDP セッションを作成します。 IP アドレスは、実際の VM のパブリック IP アドレスに置き換えてください。 このコマンドは、cmd ウィンドウから実行されます。

```
mstsc /v:publicIpAddress
```

3. プロンプトが表示された場合は、マシンのログイン資格情報を入力します。 必ず **別のアカウントを使用** してください。 ユーザー名を localhost\username として入力し、仮想マシン用に作成したパスワードを入力して **[OK]** を選択します。 サインイン処理中に証明書の警告が表示される場合があります。 接続を作成するには、**はい** または  **続行** を選択します。
4. 終了したら、VM への RDP 接続を閉じます。
5. リソースをクリーンアップする。 数分かかるこの処理により、リソースグループと仮想マシンが削除されます。

```
Remove-AzResourceGroup -Name myResourceGroup 
```