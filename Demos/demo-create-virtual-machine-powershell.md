# デモ: PowerShell で仮想マシンを作成する

このデモでは、PowerShell を使用して仮想マシンを作成します。

## 仮想マシンを作成する

>**注:** Cloud Shell またはローカル バージョンの PowerShell を使用できます。 

1. Cloud Shell を起動します。
2. このコードを実行します｡

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

## ポータルでのマシンの作成を確認する

1. ポータルにアクセスし、仮想マシンを表示します。
2. **myVM**が作成されたことを確認します。
3. VM の設定を確認します。 
4. これは、新しい VNet およびサブネットの Windows マシンであることに注意してください。 
5. コマンドがマシンを起動したのに注意してください。 
6. この時点で、ポータルまたは PowerShell を使用して変更を加えることができます。 

## 仮想マシンに接続する

1. マシンのパブリック IP アドレスを取得します。

```
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```
2. ローカル マシンから RDP セッションを作成します。IP アドレスを VM のパブリック IP アドレスに置き換えます。このコマンドは、cmd ウィンドウから実行されます。

```
mstsc /v:publicIpAddress
```

3. プロンプトが表示された場合は、マシンのログイン資格情報を入力します。必ず **別のアカウントを使用**してください。localhost\username としてユーザー名を入力して、仮想マシン用に作成したパスワードを入力し 「**OK**」 を選択します。サインイン プロセス中に証明書の警告が表示されることがあります。**はい**または**続行**を選択して接続を作成します
4. 完了したら、VM への RDP 接続を閉じます。
5. リソースをクリーンアップします。数分かかるこの処理により、リソースグループと仮想マシンが削除されます。

```
Remove-AzResourceGroup -Name myResourceGroup 
```
