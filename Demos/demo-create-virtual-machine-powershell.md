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
2. **myVM**が作成されたことを確認します。
3. VM の設定を確認します。 
4. これは、新しい VNet およびサブネットの Windows マシンであることに注意してください。 
5. コマンドがマシンを起動したのに注意してください。 
6. この時点で、ポータルまたは PowerShell を使用して変更を加えることができます。 

## <a name="connect-to-the-virtual-machine"></a>仮想マシンへの接続

1. マシンのパブリック IP アドレスを取得します。

```
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```
2. Create an RDP session from your local machine. Replace the IP address with the public IP address of your VM. This command runs from a cmd window.

```
mstsc /v:publicIpAddress
```

3. When prompted, provide your login credentials for the machine. Be sure to <bpt id="p1">**</bpt>Use a different account<ept id="p1">**</ept>. Type the username as localhost\username, enter password you created for the virtual machine, and then select <bpt id="p1">**</bpt>OK<ept id="p1">**</ept>. You may receive a certificate warning during the sign-in process. Select <bpt id="p1">**</bpt>Yes<ept id="p1">**</ept> or <bpt id="p2">**</bpt>Continue<ept id="p2">**</ept> to create the connection
4. 終了したら、VM への RDP 接続を閉じます。
5. Clean up your resources. This will take a few minutes and remove the resource group and virtual machine.

```
Remove-AzResourceGroup -Name myResourceGroup 
```