---
lab:
  title: 01b - Azure VM 上に Windows クラスタリングを実装する
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 モジュール 1: IaaS for SAP on Azure の基盤を探る
# ラボ 1b:Azure VM 上に Windows クラスタリングを実装する

予想所要時間: 120 分

このラボのタスクすべては、Azure portal (PowerShell Cloud Shell セッションを含む) から実行されます  

   > **注**:Cloud Shell を使用しない場合は、ラボ仮想マシンに Az PowerShell モジュールがインストールされている必要があります ([**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi))。

ラボ ファイル: なし

## シナリオ
  
Adatum Corporation は、データベース管理システムとして SQL Server を使用して、Azure に SAP NetWeaver をデプロイする準備として、Windows Server 2022 が実行されている Azure VM にクラスタリングを実装するプロセスを確認したいと思っています。

## 目標
  
このラボを完了すると、次のことができるようになります。

-   高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure コンピューティング リソースをプロビジョニングします。

-   高可用性 SAP NetWeaver のデプロイをサポートするように、Windows Server 2022 を実行している Azure VM の オペレーティング システムを構成する。

-   高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure ネットワーク リソースをプロビジョニングします。

## 必要条件

-   このラボで使用する予定の Azure リージョンで、十分な数の Dsv2 および Dsv3 vCPU (1 vCPU の Standard_DS1_v2 VM 1台と、4 vCPU の Standard_D4s_v3 VM 4 台) が利用可能な Microsoft Azure サブスクリプション

-   Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター

> **注**: リソースのデプロイ用に選択した Azure リージョンが可用性ゾーンをサポートしていることを確認します。 このようなリージョンの一覧については、(https://docs.microsoft.com/en-us/azure/availability-zones/az-overview) を参照してください。 **米国東部**または**米国東部 2** の使用を検討してください。

## 演習 1: 高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure コンピューティング リソースをプロビジョニングする

期間:50 分

この演習では、Windows Server 2022 を実行している Azure VM でフェールオーバー クラスタリングを構成するために必要な Azure インフラストラクチャのコンピューティング コンポーネントをデプロイします。 これには、Active Directory ドメイン コントローラーのペアをデプロイした後、Windows Server 2022 を実行する Azure VM のペアをデプロイすることが含まれます。 VM の各ペアは、同じ仮想ネットワーク内の個別の可用性ゾーンに配置されます。 ドメイン コントローラーのデプロイを自動化するには、<https://aka.ms/az120-1bdeploy> から利用可能な Azure Resource Manager クイック スタート テンプレートを使用します

### タスク 1: Bicep テンプレートを使用して、高可用性 Active Directory ドメイン コントローラを実行する Azure VM のペアをデプロイする

1. ラボ コンピューターから Web ブラウザーを起動し、 **https://portal.azure.com** から Azure portal に移動します。

1. プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注**:現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. Cloud Shell ペインで、次のコマンドを実行して、高可用性 Active Directory ドメイン コントローラーを実行している Azure VM のペアのデプロイに使用する Bicep テンプレートをホストするリポジトリのシャロー クローンを作成し、現在のディレクトリをそのテンプレートとそのパラメーター ファイルの場所に設定します。

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$rgName` の値を `az12001b-ad-RG` に設定します。

    ```
    $rgName = 'az12001b-ad-RG'
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、変数 `$location` の値を、可用性ゾーンをサポートし、ラボ VM をデプロイする Azure リージョンの名前に設定します (`<Azure_region>` プレースホルダーはそのリージョンの名前に置き換えます)。

    ```
    $location = '<Azure_region>'
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、選択した Azure リージョンに **az12001b-ad-RG** という名前のリソースグループを作成します。

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. Cloud Shell ペインで、次のコマンドを実行して変数 `$deploymentName` の値を設定します。

    ```
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、管理ユーザー アカウントの名前とそのパスワードを設定します (`<username>` と`<password>` プレースホルダーは、それぞれ管理ユーザー アカウントの名前とそのパスワードの値に置き換えます)。

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **注**: パスワードが、Windows を実行している Azure VM のデプロイに適用される複雑さの要件を満たしていることを確認します (小文字と大文字、数字、特殊文字を含む 12 文字以上の長さ)。

1. Cloud Shell ペインで、次のコマンドを実行してデプロイを行います。

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. コマンドの出力を確認し、エラーと警告が含まれていないことを確かめます。 メッセージが表示されたら、**Enter** キーを押してデプロイを続行します。

    > **注**:デプロイには約 30 分かかります。 デプロイが完了するまで待ってから、次のタスクに進みます。

    > **注**:ステートメント `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found` を含むエラーでデプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

    - Azure portal で、**adBDC** VM のブレードに移動し、左側の縦型ナビゲーション メニューの **[設定]** セクションで **[拡張機能とアプリケーション]** を選択し、**[拡張機能とアプリケーション]** ペインで **[PrepareBDC] (BDC の準備)** を選択し、**[Prepare BDC] (BDC の準備)** ペインで **[アンインストール]** を選択します。 

    - **adBDC** VM のブレードに戻り、Azure VM を再起動します。

    - **[az1201b-ad-RG]** ブレードに移動し、左側の縦型ナビゲーション メニューの **[設定]** セクションで、**[展開]** を選択します。

    - **[az1201b-ad-RG] \| [展開]** ブレードで、名前が **az1201b** というプレフィックスで始まるデプロイを選択し、デプロイ ブレードで **[再デプロイ]** を選択します。

    - **[Custom deployment] (カスタム デプロイ)** ブレードの **[Admin Password] (管理者パスワード)** テキスト ボックスに、元のデプロイ時に使用したものと同じパスワードを入力し、**[確認 + 作成]** を選択し、**[作成する]** を選択します。

    - デプロイが完了するのを待たず、代わりに次のタスクに進みます。 再デプロイには約 3 分かかります。

### タスク 2: Windows Server 2022 を実行している Azure VM のペアを別の可用性ゾーンにデプロイする

1. ラボ コンピューターから、Azure portal の **[仮想マシン]** ブレードに移動し、 **[+ 作成]** をクリックし、ドロップダウン メニューから **[Azure 仮想マシン]** を選択します。

1. **[仮想マシンの作成]** ブレードから、次の設定を使用して **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM のプロビジョニングを開始します (その他はすべて既定値のままにします)。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "新しいリソース グループの名前" **az12001b-cl-RG**** |
    | **仮想マシン名** | **az12001b-cl-vm0** |
    | **リージョン** | "前のタスクで Azure VM をデプロイしたのと同じ Azure リージョン"** |
    | **可用性オプション** | **可用性ゾーン** |
    | **可用性ゾーン** | **ゾーン 1** |
    | **イメージ** | **[Windows Server 2022 Datacenter: Azure Edition - Gen2]** を ''選択'' します** |
    | **[サイズ]** | **Standard D4s v3** |
    | **ユーザー名** | ''この演習の前半で Bicep テンプレートをデプロイするときに指定したのと同じユーザー名''** |
    | **パスワード** | ''この演習の前半で Bicep テンプレートをデプロイするときに指定したのと同じパスワード''** |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **既存の Windows Server ライセンスを使用しますか?** | **No** |
    | **OS ディスクの種類** | **Premium SSD** |
    | **Virtual Network** | **adVNET** |
    | **サブネット名** | **clSubnet** "という名前の新しいサブネット"** |
    | **サブネットのアドレス範囲** | **10.0.1.0/24** |
    | **パブリック IP アドレス** | **az12001b-cl-vm0-ip** "という名前の新しい IP アドレス"** |
    | **NIC ネットワーク セキュリティ グループ** | **Basic**  |
    | **高速ネットワークの有効化** | **オン** |
    | **負荷分散オプション** | **なし** |
    | **システム割り当てマネージド ID を有効にする** | "**オフ**" |
    | **Azure AD でログインする** | "**オフ**" |
    | **自動シャットダウンを有効にする** | "**オフ**" |
    | **パッチ オーケストレーション オプション** | **手動更新** |
    | **ブート診断** | **無効にする** |
    | **拡張機能** | *なし* |
    | **タグ** | *なし* |

1. プロビジョニングが完了するのを待たず、次の手順に進みます。

1. 次の設定で別の **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM をプロビジョニングします。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "このタスクで最初の **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM をデプロイするときに使用したリソース グループの名前"** |
    | **仮想マシン名** | **az12001b-cl-vm1** |
    | **リージョン** | "このタスクで最初の **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM をデプロイするときに使用したものと同じ Azure リージョン"** |
    | **可用性オプション** | **可用性ゾーン** |
    | **可用性ゾーン** | **ゾーン 2** |
    | **イメージ** | **[Windows Server 2022 Datacenter: Azure Edition - Gen2]** を ''選択'' します** |
    | **[サイズ]** | **Standard D4s v3** |
    | **ユーザー名** | ''この演習の前半で Bicep テンプレートをデプロイするときに指定したのと同じユーザー名''** |
    | **パスワード** | ''この演習の前半で Bicep テンプレートをデプロイするときに指定したのと同じパスワード''** |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **既存の Windows Server ライセンスを使用しますか?** | **No** |
    | **OS ディスクの種類** | **Premium SSD** |
    | **Virtual Network** | **adVNET** |
    | **サブネット名** | **clSubnet** |
    | **パブリック IP アドレス** | **az12001b-cl-vm1-ip** "という名前の新規 IP アドレス"** |
    | **NIC ネットワーク セキュリティ グループ** | **Basic**  |
    | **高速ネットワークの有効化** | **オン** |
    | **負荷分散オプション** | **なし** |
    | **Azure AD でログインする** | "**オフ**" |
    | **自動シャットダウンを有効にする** | "**オフ**" |
    | **パッチ オーケストレーション オプション** | **手動更新** |
    | **ブート診断** | **無効にする** |
    | **拡張機能** | *なし* |
    | **タグ** | *なし* |

1. プロビジョニングが完了するまで待ちます。 通常、これには数分かかります。

### タスク 3:Azure VM ディスクを作成して構成する

1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

1. [Cloud Shell] ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM にアタッチする 4 個のマネージド ディスクの最初のセットを作成します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM にアタッチする 4 個のマネージド ディスクの 2 番目のセットを作成します。

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. Azure portal で、前のタスクでプロビジョニングした最初の Azure VM (**az12001b-cl-vm0**) のブレードに移動します。

1. **az12001b-cl-vm0** ブレードから **az12001b-cl-vm0 - Disks** ブレードに移動します。

1. 次の設定のデータ ディスクを **az12001b-cl-vm0 - Disks** ブレードから az12001b-cl-vm0 に接続します。
   
   | 設定 | 値 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **ディスク名** | **az12001b-cl-vm0-DataDisk0** |
   | **リソース グループ** | "前のタスクで **Windows Server 2022 Datacenter** Azure VM のペアをデプロイするときに使用したリソース グループの名前"** |
   | **ホスト キャッシュ** | **読み取り専用** |

1. 前の手順を繰り返して、プレフィックスが **az12001b-cl-vm0-DataDisk** である残りの 3 つのディスク (合計 4 つ) を接続します。 ディスク名の最後の文字と一致する LUN 番号を割り当てます。 最後のディスク (LUN **3**) については、ホスト キャッシュを**なし**に設定します。

1. 変更を保存します。 

1. Azure portal で、前のタスクでプロビジョニングした 2 番目の Azure VM (**az12001b-cl-vm1**) のブレードに移動します。

1. **[az12001b-cl-vm1]** ブレードから、**[az12001b-cl-vm1 - ディスク]** ブレードに移動します。

1. **[az12001b-cl-vm1 - ディスク]** ブレードから、次の設定を持つデータ ディスクを az12001b-cl-vm1 に接続します。
     
   | 設定 | 値 |
   |   --    |  --   |
   | **LUN** | **0** |
   | **ディスク名** | **az12001b-cl-vm1-DataDisk0** |
   | **リソース グループ** | "前のタスクで **Windows Server 2022 Datacenter** Azure VM のペアをデプロイするときに使用したリソース グループの名前"** |
   | **ホスト キャッシュ** | **読み取り専用** |

1. 前の手順を繰り返して、プレフィックスが **az12001b-cl-vm1-DataDisk** である残りの 3 つのディスク (合計 4 つ) を接続します。 ディスク名の最後の文字と一致する LUN 番号を割り当てます。 最後のディスク (LUN **3**) については、ホスト キャッシュを**なし**に設定します。

1. 変更を保存します。 

> **Result**:この演習を完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## 演習 2: 高可用性 SAP NetWeaver のインストールをサポートするように、Windows Server 2022 を実行する Azure VM のオペレーティング システムを構成する

期間:40 分

### タスク 1: Windows Server 2022 Datacenter VM を Active Directory ドメインに参加させる。

   > **注**:このタスクを開始する前に、前の演習の最後のタスクで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1. Azure portal で、このラボの最初の演習で自動的にプロビジョニングされた **adVNET** という仮想ネットワークのブレードに移動します。

1. **[adVNET - DNS サーバー]** ブレードを表示します。仮想ネットワークは、このラボの最初の演習でデプロイされたドメイン コントローラーに割り当てられたプライベート IP アドレスを DNS サーバーとして構成されます。

1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

1. Cloud Shell ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、前の演習でプロビジョニングした **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、前の演習の 2 番目のタスクでデプロイした Windows Server 2022 Azure VM を **adatum.com** Active Directory ドメインに参加させます (`<username>` と `<password>` プレースホルダーは、このラボの最初の演習で Bicep テンプレートをデプロイするときに指定した管理ユーザー アカウントの名前とパスワードに置き換えます)。

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. 次のタスクを進める前に、スクリプトが完了するのを待ちます。


### タスク 2: 高可用性 SAP NetWeaver のインストールをサポートするように、Windows Server 2022 を実行している Azure VM のストレージを構成する。

1. Azure portal で、このラボの最初の演習でプロビジョニングした **az12001b-cl-vm0** という仮想マシンのブレードに移動します。

1. リモート デスクトップを使って、**az12001b-cl-vm0** ブレードから仮想マシン ゲスト オペレーティング システムに接続します。 認証を求められたら、このラボの最初の演習で Bicep テンプレートをデプロイするときに指定した管理ユーザー アカウントの資格情報を指定します。 

    > **注**: オペレーティング システム レベルのアカウントではなく、必ず、**ADATUM** ドメイン アカウントを使用してサインインしてください (つまり、ユーザー名の前に **ADATUM\\** プレフィックスが付いていることを確かめます)。

1. az12001b-cl-vm0 への RDP セッション内で、サーバー マネージャーの **[ファイルとストレージ サービス]** -> **[サーバー]** ノードに移動します。 

1. **記憶域プール** ビューに移動し、前の演習で Azure VM に接続したすべてのディスクが表示されていることを確認します。

1. **新しい記憶域プール ウィザード**を使用して、次の設定で新しい記憶域プールを作成します。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **Data Storage Pool** |
    | **物理ディスク** | "最初の 3 つの LUN 番号 (0 から 2) に対応するディスク番号を持つ 3 つのディスクを選択し、それらの割り当てを" **[自動]** "に設定します"** |

    > **注**:**Chassis** 列のエントリを使って、**LUN** 番号を識別します。

1. **新しい仮想ディスク ウィザード**を使用して、次の設定で新しい仮想ディスクを作成します。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **仮想ディスク名** | **Data Virtual Disk** |
    | **記憶域のレイアウト** | **簡易** |
    | **Provisioning** | **修正済み** |
    | **[サイズ]** | **最大サイズ** |

1. **新規ボリューム ウィザード**を使用して、次の設定で新しいボリュームを作成します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サーバーとディスク** | ''既定値を受け入れます''** |
    | **[サイズ]** | ''既定値を受け入れます''** |
    | **ドライブ文字** | **M** |
    | **ファイル システム** | **ReFS** |
    | **アロケーション ユニット サイズ** | **既定値** |
    | **ボリューム ラベル** | **データ** |

1. **[記憶域プール]** ビューに戻り、**新しい記憶域プール ウィザード**を使用して、次の設定で新しい記憶域プールを作成します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **ログ記憶域プール** |
    | **物理ディスク** | "4 つのディスクのうち最後の 1 つを選び、その割り当てを" **[自動]** "に設定します"** |

1. **新しい仮想ディスク ウィザード**を使用して、次の設定で新しい仮想ディスクを作成します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **仮想ディスク名** | **Log Virtual Disk** |
    | **記憶域のレイアウト** | **簡易** |
    | **Provisioning** | **修正済み** |
    | **[サイズ]** | **最大サイズ** |

1. **新規ボリューム ウィザード**を使用して、次の設定で新しいボリュームを作成します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サーバーとディスク** | ''既定値を受け入れます''** |
    | **[サイズ]** | ''既定値を受け入れます''** |
    | **ドライブ文字** | **L** |
    | **ファイル システム** | **ReFS** |
    | **アロケーション ユニット サイズ** | **既定値** |
    | **ボリューム ラベル** | **ログ** |

1. このタスクの前の手順を繰り返して、az12001b-cl-vm1 に記憶域を設定します。

### タスク 3: 高可用性 SAP NetWeaver のインストールをサポートするように、Windows Server 2022 を実行している Azure VM でフェールオーバー クラスタリングの構成を準備する。

1. az12001b-cl-vm0 への RDP セッション内で、Windows PowerShell ISE セッションを開始し、次のコマンドを実行して、az12001b-cl-vm0 と az12001b-cl-vm1 の両方にフェールオーバー クラスタリングとリモート管理ツールの機能をインストールします。

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注**:これにより、両方の Azure VM でゲスト オペレーティングシステムが再起動されます。

1. ラボ コンピューターで Azure portal を開き、**[+ リソースの作成]** をクリックします。

1. **[新規]** ブレードから、次の設定を使用して新しい**ストレージ アカウント**の作成を開始します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"** |
    | **リソース グループ** | ''前の演習でプロビジョニングした **Windows Server 2022 Datacenter** Azure VM のペアを含むリソース グループの名前''** |
    | **Storage account name (ストレージ アカウント名)** | ''3 から 24 の文字と数字で構成される一意の名前''** |
    | **場所** | ''前の演習で Azure VM をデプロイしたのと同じ Azure リージョン''** |
    | **パフォーマンス** | **Standard** |
    | **冗長性** | **ローカル冗長ストレージ (LRS)** |
    | **接続方法** | **パブリック エンドポイント (すべてのネットワーク)** |
    | **REST API 操作の安全な転送を必須にする** | **有効** |
    | **大きいファイルの共有** | **Disabled** |
    | **BLOB、コンテナー、ファイルの論理的な削除** | **Disabled** |
    | **階層構造の名前空間** | **Disabled** |

### タスク 4: 高可用性 SAP NetWeaver のインストールをサポートするように、Windows Server 2022 を実行している Azure VM のフェールオーバー クラスタリングを構成する。

1. Azure portal で、このラボの最初の演習でプロビジョニングした **az12001b-cl-vm0** という仮想マシンのブレードに移動します。

1. リモート デスクトップを使って、**az12001b-cl-vm0** ブレードから仮想マシン ゲスト オペレーティング システムに接続します。 認証を求められたら、このラボの最初の演習で Bicep テンプレートをデプロイするときに指定した管理ユーザー アカウントの資格情報を指定します。

1. az12001b-cl-vm0 への RDP セッションで、サーバー マネージャーの **[ツール]** メニューから、**Active Directory 管理センター**を起動します。

1. Active Directory 管理センターで、adatum.com ドメインのルートに**クラスター**という名前の新しい組織単位を作成します。

1. Active Directory 管理センターで、**az12001b-cl-vm0** および **az12001b-cl-vm1** のコンピューター アカウントを**コンピューター** コンテナーから**クラスター**組織単位に移動します。

1. az12001b-cl-vm0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. az12001b-cl-vm0 への RDP セッションから、**Active Directory 管理センター** コンソールに切り替えます。

1. Active Directory 管理センターで、**[クラスター]** 組織単位に移動し、**[プロパティ]** ウィンドウを表示します。 

1. **[クラスター]** 組織単位の **[プロパティ]** ウィンドウで、**[拡張機能]** セクションに移動し、**[セキュリティ]** タブを表示します。 

1. **[セキュリティ]** タブで **[詳細]** ボタンをクリックして、**[Advanced Security Settings for Clusters]\(クラスターの詳細なセキュリティ設定\)** ウィンドウを開きます。 

1. **[クラスターの詳細なセキュリティ設定]** ウィンドウの **[アクセス許可]** タブで、**[追加] **をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[プリンシパルの選択]** をクリックします

1. **[ユーザー、サービス アカウントまたはグループの選択]** ダイアログ ボックスで、**[オブジェクトの種類]** をクリックし、**[コンピューター]** エントリの横にあるチェックボックスを有効にし、**[OK]** をクリックします。 

1. **[ユーザー、コンピューター、サービス アカウント、またはグループの選択]** ダイアログ ボックスに戻り、**[選択するオブジェクト名を入力する]** に「**az12001b-cl-cl0**」と入力し、**[OK]** をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[種類]** ドロップダウン リストに **[許可]** が表示されるようにします。 次に、**[適用先]** ドロップダウン リストで、**[このオブジェクトとすべての子オブジェクト]** を選択します。 **[アクセス許可]** リストで、**[コンピューター オブジェクトの作成]** チェック ボックスと **[コンピューター オブジェクトの削除]** チェック ボックスをオンにし、**[OK]** を 2 回クリックします。

1. Windows PowerShell ISE セッション内で、次の実行して Az PowerShell モジュールをインストールします。

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して Azure AD 資格情報を使用して認証します。

    ```
    Add-AzAccount
    ```

    > **注**:プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して、新しいクラスターの Cloud Witness Quorum を設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. 結果の構成を確認するには、az12001b-cl-vm0 への RDP セッションのサーバー マネージャーで、**[ツール]** メニューから **[フェールオーバー クラスター マネージャー]** を起動します。

1. **[フェールオーバー クラスター マネージャー]** コンソールで、**az12001b-cl-cl0** クラスター構成 (ノードを含む)、監視設定およびネットワーク設定を確認します。 クラスターには共有ストレージがないことに注意してください。

1. az12001b-cl-vm0 への RDP セッションを終了します。

> **結果**: この演習を完了すると、高可用性 SAP NetWeaver のインストールをサポートするように、Windows Server 2022 を実行している Azure VM のオペレーティング システムを構成することができます


## 演習 3: 高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure ネットワーク リソースをプロビジョニングする

期間: 30 分

この演習では、SAP NetWeaver のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。

### タスク 1:負荷分散のセットアップが容易に行えるよう Azure VM を構成します。

   > **注**:Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1. ラボ コンピューターで、Azure portal の Azure VM **az12001b-cl-vm0** ブレードに移動します。 

1. **az12001b-cl-vm0** ブレードから、ネットワーク アダプターに関連付けられたパブリック IP アドレス **az12001b-cl-vm0-ip** のブレードに移動します。

1. **az12001b-cl-vm0-ip** ブレードで、パブリック IP アドレスをネットワーク インターフェイスから関連付けを解除し、削除します。

1. Azure portal で、Azure VM **az12001b-cl-vm1** のブレードに移動します。 

1. **[az12001b-cl-vm1]** ブレードから、ネットワーク アダプターに関連付けられているパブリック IP アドレス **az12001b-cl-vm1-ip** のブレードに移動します。

1. **[az12001b-cl-vm1-ip]** ブレードで、パブリック IP アドレスをネットワーク インターフェイスから関連付け解除し、削除します。

1. Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1. **az12001a-vm0** ブレードから、**ネットワーク** ブレードに移動します。 

1. **az12001a-vm0 - Networking** ブレードから、az12001a-vm0 のネットワーク インターフェイスに移動します。 

1. az12001a-vm0 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

1. Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。

1. **[az12001a-vm1]** ブレードから、**[ネットワーク]** ブレードに移動します。 

1. **[az12001a-vm1 - ネットワーク]** ブレードから、az12001a-vm1 のネットワーク インターフェイスに移動します。 

1. az12001a-vm1 のネットワーク インターフェイスのブレードから、その [IP 構成] ブレードに移動し、そこから **[ipconfig1]** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

### タスク 2:受信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal で、**[+リソースの作成]** をクリックします。

1. **新規**ブレードから、次の設定を使って Azure ロード バランサーの新規作成を開始します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"** |
    | **リソース グループ** | ''このラボの最初の演習でプロビジョニングした **Windows Server 2022 Datacenter** Azure VM のペアを含むリソース グループの名前''** |
    | **名前** | **az12001b-cl-lb0** |
    | **リージョン** | このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン* |
    | **SKU** | **Standard** |
    | **Type** | **内部** |
    | **フロントエンド IP 名** | **frontend-ip1** |
    | **Virtual Network** | **adVNET** |
    | **サブネット** | **clSubnet** |
    | **IP アドレスの割り当て** | **静的** |
    | **IP アドレス (IP address)** | **10.0.1.240** |
    | **可用性ゾーン** | **ゾーン冗長** |

1. ロード バランサーのプロビジョニングが完了するのを待って、Azure portal のブレードに移動します。

1. **az12001b-cl-lb0** ブレードから、次の設定でバックエンド プールを追加します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **az12001b-cl-lb0-bepool** |
    | **Virtual Network** | **adVNET** |
    | **バックエンド プールの構成** | **IP アドレス (IP address)** |
    | **IP アドレス (IP address)** | **10.0.1.4** リソース名 **az1201b-cl-vm0** |
    | **IP アドレス (IP address)** | **10.0.1.5** リソース名 **az1201b-cl-vm1** |

1. **az12001b-cl-lb0** ブレードから、次の設定で正常性プローブを追加します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **az12001b-cl-lb0-hprobe** |
    | **プロトコル** | **TCP** |
    | **ポート** | **59999** |
    | **間隔** | **5** *秒* |
    | **異常のしきい値** | **2** *個の連続エラー* |

1. **az12001b-cl-lb0** ブレードから、次の設定でネットワーク負荷分散ルールを追加します。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **az12001b-cl-lb0-lbruletcp1433** |
    | **IP バージョン** | **IPv4** |
    | **フロントエンド IP アドレス** | **10.0.1.240 (LoadBalancerFrontEnd)** |
    | **HA ポート** | **Disabled** |
    | **プロトコル** | **TCP** |
    | **ポート** | **1433** |
    | **バックエンド ポート** | **1433** |
    | **バックエンド プール** | **az12001b-cl-lb0-bepool (2 台の仮想マシン)** |
    | **正常性プローブ** | **az12001b-cl-lb0-hprobe (TCP:59999)** |
    | **セッション永続化** | **なし** |
    | **アイドル タイムアウト (分)** | **4** |
    | **TCP リセット** | **Disabled** |
    | **フローティング IP (ダイレクト サーバー リターン)** | **有効** |

### タスク 3:送信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal の Cloud Shell で PowerShell セッションを開始します。 

1. Cloud Shell ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初の演習でプロビジョニングした **Windows Server 2022 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer を作成します。

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. [Cloud Shell] ペインを閉じます。

1. Azure portal で、Azure ロード バランサー **az12001b-cl-lb1** のプロパティを表示するブレードに移動します。

1. **[az12001b-cl-lb1]** ブレードで、**[バックエンド プール]** をクリックします。

1. **az12001b-cl-lb1 - Backend pools** ブレードで、**az12001b-cl-lb1-bepool** をクリックします。

1. **[az12001b-cl-lb1-bepool]** ブレードで、次の設定を指定し、**[保存]** をクリックします。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **Virtual Network** | **adVNET (4 VM)** |
    | **仮想マシン** | **az12001b-cl-vm0**  IP アドレス: **ipconfig1** |
    | **仮想マシン** | **az12001b-cl-vm1**  IP アドレス: **ipconfig1** |

1. **[az12001b-cl-lb1]** ブレードで、**[正常性プローブ]** をクリックします。

1. **az12001b-cl-lb1 - Health probes** ブレードから、次の設定で正常性プローブを追加します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **az12001b-cl-lb1-hprobe** |
    | **プロトコル** | **TCP** |
    | **ポート** | **80** |
    | **間隔** | **5** *秒* |
    | **異常のしきい値** | **2** *個の連続エラー* |

1. **[az12001b-cl-lb1]** ブレードで、**[負荷分散規則]** をクリックします。

1. **az12001b-cl-lb1 - Load balancing rules** ブレードから、次の設定でネットワーク負荷分散ルールを追加します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **名前** | **az12001b-cl-lb1-lbharule** |
    | **IP バージョン** | **IPv4** |
    | **フロントエンド IP アドレス** | *既定値を受け入れる* |
    | **HA ポート** | **Disabled** |
    | **プロトコル** | **TCP** |
    | **ポート** | **80** |
    | **バックエンド ポート** | **80** |
    | **バックエンド プール** | **az12001b-cl-lb1-bepool (2 台の仮想マシン)** |
    | **正常性プローブ** | **az12001b-cl-lb1-hprobe (TCP:80)** |
    | **セッション永続化** | **なし** |
    | **アイドル タイムアウト (分)** | **4** |
    | **TCP リセット** | **Disabled** |
    | **フローティング IP (ダイレクト サーバー リターン)** | **Disabled** |

### タスク 4:ジャンプ ホストをデプロイする

   > **注**:2 台のクラスター化された Azure VM にはインターネットから直接アクセスできなくなったため、ジャンプ ホストとして機能する Windows Server 2022 Datacenter を実行する Azure VM をデプロイします。 

1. ラボ コンピューターから、Azure portal の **[仮想マシン]** ブレードに移動し、 **[+ 作成]** をクリックし、ドロップダウン メニューから **[Azure 仮想マシン]** を選択します。

1. **[仮想マシンの作成]** ブレードから、次の設定を使用して **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM のプロビジョニングを開始します。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | ''このラボの最初の演習でプロビジョニングした **Windows Server 2022 Datacenter: Azure Edition - Gen2** Azure VM のペアを含むリソース グループの名前''** |
    | **仮想マシン名** | **az12001b-vm2** |
    | **リージョン** | ''このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン''** |
    | **可用性オプション** | **インフラストラクチャ冗長は必要ありません** |
    | **イメージ** | **[Windows Server 2022 Datacenter: Azure Edition - Gen2]** を ''選択'' します** |
    | **[サイズ]** | **Standard DS1 v2*** または同様のもの* |
    | **ユーザー名** | ''このラボの最初の演習で Bicep テンプレートをデプロイするときに指定したのと同じユーザー名''** |
    | **パスワード** | ''このラボの最初の演習で Bicep テンプレートをデプロイするときに指定したのと同じパスワード''** |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **既存の Windows Server ライセンスを使用しますか?** | **No** |
    | **OS ディスクの種類** | **Standard HDD** |
    | **Virtual Network** | **adVNET** |
    | **サブネット名** | **bastionSubnet** "という名前の新しいサブネット"** |
    | **サブネットのアドレス範囲** | **10.0.255.0/24** |
    | **パブリック IP アドレス** | **az12001b-vm2-ip** "という名前の新しい IP アドレス"** |
    | **NIC ネットワーク セキュリティ グループ** | **Basic**  |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **高速ネットワークの有効化** | "**オフ**" |
    | **負荷分散オプション** | **なし** |
    | **システム割り当てマネージド ID を有効にする** | "**オフ**" |
    | **Azure AD でログインする** | "**オフ**" |
    | **自動シャットダウンを有効にする** | "**オフ**" |
    | **ブート診断** | **無効にする** |
    | **OS のゲスト診断を有効にする** | "**オフ**" |
    | **拡張機能** | *なし* |
    | **タグ** | *なし* |

1. プロビジョニングが完了するまで待ちます。 通常、これには数分かかります。

1. RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1. az12001b-vm2 への RDP セッション内で、プライベート IP アドレス (10.0.1.4 と 10.0.1.5) を介して az12001b-vm0 と az12001b-vm1 の両方に SSH セッションを確立できることを確認します。 

> **Result**:この演習を完了すると、可用性の高い SAP NetWeaver のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます

## 演習 4: ラボ リソースを削除する

期間: 10 分

この演習では、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1:Cloud Shell を開く

1. ポータルの上部にある **[Cloud Shell]** アイコンをクリックして Cloud Shell ペインを開き、シェルとして PowerShell を選択します。

1. Cloud Shell ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初の演習でプロビジョニングした **Windows Server 2022 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupNamePrefix = 'az12001b-'
    ```

1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成したすべてのリソース グループをリストアップします。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。 これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. Cloud Shell ペインで次のコマンドを実行して、新しく作成したサブネットのリソース ID を特定します。

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. [Cloud Shell] ペインを閉じます。

> **Result**:この演習を完了すると、このラボで使用したリソースが削除されます。
