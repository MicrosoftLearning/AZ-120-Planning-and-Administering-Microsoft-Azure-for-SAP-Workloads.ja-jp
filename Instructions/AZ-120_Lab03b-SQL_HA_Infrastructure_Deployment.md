---
lab:
  title: 04b - Windows を実行する Azure VM に SAP アーキテクチャを実装する
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 モジュール 4:SAP on Azure のデプロイ
# ラボ 4b:課題: Windows を実行する Azure VM に SAP アーキテクチャを実装する

予測される所要時間:150 分

このラボのタスクすべては、Azure portal (PowerShell Cloud Shell セッションを含む) から実行されます  

   > **注**:Cloud Shell を使用しない場合は、ラボ仮想マシンに Az PowerShell モジュールがインストールされている必要があります ([**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi))。

ラボ ファイル: なし

## シナリオ
  
Azure に SAP NetWeaver をデプロイする準備として、Adatum Corporation は、Windows Server 2016 を実行する Azure VM での SAP NetWeaver の高可用性実装がわかるデモを実装したいと考えています。

## 目標
  
このラボを完了すると、次のことができるようになります。

-   高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

-   可用性の高い SAP NetWeaver デプロイをサポートするように、Windows を実行している Azure VM のオペレーティング システムを構成する

-   可用性の高い SAP NetWeaver のデプロイをサポートするために、Windows を実行している Azure VM にクラスタリングを構成する

## 必要条件

-   可用性ゾーンをサポートする Azure リージョンで、十分な数の Dsv3 vCPU (2 vCPU の Standard_D2s_v3 VM を 4 台、4 vCPU の Standard_D4s_v3 VM を 6 台) が利用可能な Microsoft Azure サブスクリプション

-   Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター


## 演習 1: 高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

期間:60 分

このエクササイズでは、Windows クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。 これには、同じ可用性セットで Windows Server 2016 を実行する Azure VM のペアを作成する必要があります。

### タスク 1:Azure Resource Manager テンプレートを使用して、高可用性な Active Directory ドメイン コントローラを実行する 2 組の Azure VM をデプロイする

1. ラボ コンピューターから Web ブラウザーを起動し、**https://portal.azure.com** にある Azure portal に移動します。

1. プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Azure portal のインターフェイスで、**[リソースの作成]** をクリックします。

1. **[新規]** ブレードから、新しい **Template deployment (カスタム テンプレートを使用したデプロイ)** の作成を開始します

1. **[カスタム デプロイ]** ブレードの、**[クイックスタート テンプレート (免責事項)]** ドロップダウン リストで、エントリ **application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones** を選択し、**[テンプレートの選択]** をクリックします。

    > **注**:または、<https://github.com/Azure/azure-quickstart-templates> の Azure クイック スタート テンプレート ページに移動して、**2 つの新しい Windows VM、新しい AD フォレスト、ドメイン、2 つの DC を別々の可用性ゾーンに作成する**という名前のテンプレートを検索し、**[Azure にデプロイ]** ボタンをクリックしてデプロイを開始することもできます。

1. **[可用性ゾーンを使用して 2 つの DC を持つ新しい AD ドメインを作成する]** というラベルの付いたブレードで、次の設定を指定し、**[確認および作成]** をクリックした後、**[作成]** をクリックしてデプロイを開始します。
     
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "新しいリソース グループの名前" **az12003b-ad-RG**** |
    | **場所** | "Azure VM をデプロイできる Azure リージョン"** |
    | **管理ユーザー名** | **Student** |
    | **場所** | "上記で指定したのと同じ Azure リージョン"** |
    | **パスワード** | *任意の 12 文字以上の長さの任意の複雑なパスワード* |
    | **ドメイン名** | **adatum.com** |
    | **DnsPrefix** | *一意の有効な DNS プレフィックスを使用します* |
    | **VM サイズ** | **Standard D2s_v3** |
    | **_artifacts の場所** | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/** |
    | **Sas トークンとしての _artifacts の場所** | *空白のままにする* |

    > **注**: デプロイ時に指定したパスワードを忘れないでください。 このラボで後ほど必要になります。

    > **注**:リソースのデプロイには、**米国東部**または**米国東部 2** リージョンの使用を検討してください。 
    
    > **注**:デプロイには約 35 分かかります。 次のタスクを進める前に、展開が完了するのを待ちます。

    > **注**:CustomScriptExtension コンポーネントのデプロイ中に**競合**エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal の **[デプロイ]** ブレードで、デプロイの詳細を確認し、CustomScriptExtension のインストールが失敗した VM を特定します

       - Azure portal で、前の手順で特定した VM のブレードに移動し、**[拡張機能]** を選択し、**[拡張機能]** ブレードから CustomScript 拡張機能を削除します

       - Azure portal で、**az12003b-sap-RG** リソース グループ ブレードに移動し、**[デプロイ]** を選択し、失敗したデプロイへのリンクを選択して **[再デプロイ]** を選択し、ターゲット リソース グループ (**az12003b-sap-RG**) を選択して、ルート アカウントのパスワードを指定します (**Pa55w.rd1234**)。

1. デプロイが完了したら、Azure portal で **adPDC** 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの **[操作]** セクションで **[コマンドの実行]** を選択し、**[コマンド スクリプトの実行]** ペインの **[PowerShell スクリプト]** テキスト ボックスに次のスクリプトを入力し、**[実行]** ボタンを選択します。

    ```
    New-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters\' -Name 'DisabledComponents' -Value 0xffffffff -PropertyType 'DWord'
    Restart-Computer -Force
    ```

1. **adPDC** 仮想マシンが再び実行されるまで待ち、**adBDC** 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの **[操作]** セクションで **[コマンドの実行]** を選択し、**[コマンド スクリプトの実行]** ペインの **[PowerShell スクリプト]** テキスト ボックスに次のスクリプトを入力し、**[実行]** ボタンを選択します。

    ```
    New-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters\' -Name 'DisabledComponents' -Value 0xffffffff -PropertyType 'DWord'
    Restart-Computer -Force
    ```
    
1. **adBDC** 仮想マシンが再び実行されるまで待ち、**adPDC** 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの **[操作]** セクションで **[コマンドの実行]** を選択し、**[コマンド スクリプトの実行]** ペインの **[PowerShell スクリプト]** テキスト ボックスに次のスクリプトを入力し、**[実行]** ボタンを選択します。

    ```
    repadmin /syncall /APeD
    ```
    
1. **adBDC** 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの **[操作]** セクションで **[コマンドの実行]** を選択し、**[コマンド スクリプトの実行]** ペインの **[PowerShell スクリプト]** テキスト ボックスに次のスクリプトを入力し、**[実行]** ボタンを選択します。

    ```
    repadmin /syncall /APeD
    ```
    
    > **注**: これらの追加の手順では、この場合の名前解決の問題が発生する IPv6 を無効にし、その後、2 つのドメイン コントローラー間で強制的にレプリケーションを行います。  

### タスク 2:可用性の高い SAP NetWeaver デプロイと S2D クラスターを実行する Azure VM をホストするサブネットをプロビジョニングする。

1. Azure Portal で、**az12003b-ad-RG** リソース グループのブレードに移動します。

1. **[az12003b-ad-RG]** リソース グループ ブレードのリソースのリストで、**adVNET** 仮想ネットワークを検索し、そのエントリをクリックして **[adVNET]** ブレードを表示します。

1. **[adVNET]** ブレードから、**[adVNET - サブネット]** ブレードに移動します。 

1. **[adVNET - サブネット]** ブレードから、次の設定を使用して新しいサブネットを作成します。

    -   名前: **sapSubnet**

    -   アドレス範囲 (CIDR ブロック):**10.0.1.0/24**

1. **[adVNET - サブネット]** ブレードから、次の設定を使用して新しいサブネットを作成します。

    -   名前: **s2dSubnet**

    -   アドレス範囲 (CIDR ブロック):**10.0.2.0/24**

1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注**:現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. [Cloud Shell] ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1. [Cloud Shell] ウィンドウで次のコマンドを実行して、前のタスクで作成した仮想ネットワークを特定します。

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1. Cloud Shell ペインで次のコマンドを実行して、新しく作成したサブネットのリソース ID を特定します。

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1. 結果の値をクリップボードにコピーします。 これは、次のタスクで必要になります。

### タスク 3:可用性の高い SAP NetWeaver のデプロイをホストする Windows Server 2016 を実行する Azure VM をプロビジョニングする Azure Resource Manager テンプレートをデプロイする

1. ラボ コンピューターの Azure portal で、**[Template deployment (カスタム テンプレートを使用したデプロイ)]** を検索して選択します。

1. **[カスタム デプロイ]** ブレードの **[クイックスタート テンプレート (免責事項)]** ボックスの一覧で、「**application-workloads/sap/sap-3-tier-marketplace-image-md**」と入力し、**[テンプレートの選択]** をクリックします。

    > **注**:Microsoft Edge またはサード パーティのブラウザーを使用してください。 Internet Explorer は使用しないでください。

1. **[SAP NetWeaver 3-tier (managed disk)]** ブレードで、**[テンプレートの編集]** を選択します。

1. **[テンプレートの編集]** ブレードで、次の変更を適用して **[保存]** を選択します。

    -   **197** の行で、`"dbVMSize": "Standard_E8s_v3",` を `"dbVMSize": "Standard_D4s_v3",` に置き換えます

1. **[SAP NetWeaver 3 層 (マネージド ディスク)]** ブレードに戻り、次の設定を指定して、**[確認および作成]** をクリックした後、**[作成]** をクリックしてデプロイを開始します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "新しいリソース グループの名前" **az12003b-sap-RG**** |
    | **場所** | "この演習の最初のタスクで指定したものと同じ Azure リージョン"** |
    | **SAP システム ID** | **I20** |
    | **スタックの種類** | **ABAP** |
    | **OS の種類** | **Windows Server 2016 Datacenter** |
    | **Dbtype** | **SQL** |
    | **SAP システム サイズ** | **デモ** |
    | **システム可用性** | **HA** |
    | **管理ユーザー名** | **Student** |
    | **認証の種類** | **password** |
    | **管理者パスワードまたはキー** | *このラボで前に指定したのと同じパスワード* |
    | **サブネット ID** | "前のタスクでクリップボードにコピーした値"** |
    | **可用性ゾーン** | **1、2** |
    | **場所** | **[resourceGroup().location]** |
    | **_artifacts の場所** | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/** |
    | **Sas トークンとしての _artifacts の場所** | *空白のままにする* |

1. デプロイが完了するのを待たずに、代わりに次のタスクに進みます。 

### タスク 4:スケールアウト ファイル サーバー (SOFS) クラスターをデプロイする

このタスクでは、[**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md) で利用可能な GitHub の Azure Resource Manager クイックスタート テンプレートを使用して、SAP ASCS サーバーのファイル共有をホストするスケールアウト ファイル サーバー (SOFS) クラスターをデプロイします。 

1. ラボ コンピューターでブラウザーを起動し、[**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md) を参照します。 

    > **注**:Microsoft Edge またはサード パーティのブラウザーを使用してください。 Internet Explorer は使用しないでください。

1. "**マネージド ディスクを使用して、Windows Server 2016 で記憶域スペース ダイレクト (S2D) スケールアウト ファイル サーバー (SOFS) を作成する**" というタイトルのページで、**[Azure にデプロイ]** をクリックします。 これにより、ブラウザーが Azure portal に自動的にリダイレクトされ、**[カスタム デプロイ]** ブレードが表示されます。

1. **[カスタム デプロイ]** ブレードから、次の設定を指定し、**[確認および作成]** をクリックした後、**[作成]** をクリックしてデプロイを開始します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "新しいリソース グループの名前" **az12003b-s2d-RG**** |
    | **リージョン** | "このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン"** |
    | **Name Prefix** | **i20** |
    | **VM サイズ** | **Standard D4s\_v3** |
    | **高速ネットワークの有効化** | **true** |
    | **イメージ SKU** | **2016-Datacenter-Server-Core** |
    | **VM カウント** | **2** |
    | **VM ディスク サイズ** | **128** |
    | **VM ディスク カウント** | **3** |
    | **既存のドメイン名** | **adatum.com** |
    | **管理ユーザー名** | **Student** |
    | **管理者パスワード** | *このラボで前に指定したのと同じパスワード* |
    | **既存の仮想ネットワーク RG 名** | **az12003b-ad-RG** |
    | **既存の仮想ネットワーク名** | **adVNet** |
    | **既存のサブネット名** | **s2dSubnet** |
    | **Sofs Name** | **sapglobalhost** |
    | **共有名** | **sapmnt** |
    | **スケジュールされた更新日** | **日曜日** |
    | **スケジュールされた更新時間** | **3:00 AM** |
    | **リアルタイム マルウェア対策有効** | **false** |
    | **スケジュールされたマルウェア対策有効** | **false** |
    | **スケジュールされたマルウェア対策時間** | **120** |
    | **_artifacts の場所** | **https://raw.githubusercontent.com/polichtm/301-storage-spaces-direct-md/master** |
    | **Sas トークンとしての _artifacts の場所** | **既定値のままにします** |

1. デプロイには約 20 分間かかります。 デプロイが完了するのを待たずに、代わりに次のタスクに進みます。

    > **注**:i20-s2d-1/s2dPrep or i20-s2d-0/s2dPrep コンポーネントのデプロイ中に**競合**エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal で、i20-s2d-0 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの [操作] セクションで [コマンドの実行] を選択し、[コマンド スクリプトの実行] ペインの [PowerShell スクリプト] テキスト ボックスに次のスクリプトを入力し、[実行] ボタンを選択します。

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```

       - i20-s2d-01 仮想マシンのブレードに移動し、縦型ナビゲーション メニューの [操作] セクションで [コマンドの実行] を選択し、[コマンド スクリプトの実行] ペインの [PowerShell スクリプト] テキスト ボックスに次のスクリプトを入力し、[実行] ボタンを選択します。

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```
       
       - 現在のタスクの手順を初めからもう一度実行します

### タスク 5:ジャンプ ホストをデプロイする

   > **注**:前のタスクでデプロイした Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2016 Datacenter を実行する Azure VM をデプロイします。 

1. ラボ コンピューターで Azure portal インターフェイスを開き、**[リソースを作成]** をクリックします。

1. **[新規]** ブレードから、**Windows Server 2019 Datacenter -Gen1** イメージを基に新規 Azure VM の作成を開始します。

1. 次の設定を使用して Azure VM をプロビジョニングします:
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"**  |
    | **リソース グループ** | "新しいリソース グループの名前" **az12003b-dmz-RG**** |
    | **仮想マシン名** | **az12003b-vm0** |
    | **リージョン** | "このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン"** |
    | **可用性オプション** | **インフラストラクチャ冗長は必要ありません** |
    | **イメージ** | *select* **Windows Server 2019 Datacenter - Gen2** |
    | **[サイズ]** | **Standard D2s_v3** |
    | **ユーザー名** | **Student** |
    | **パスワード** | *このラボで前に指定したのと同じパスワード* |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **既存の Windows Server ライセンスを使用しますか?** | **No** |
    | **OS ディスクの種類** | **Standard HDD** |
    | **Virtual Network** | **adVNET** |
    | **サブネット名** | **dmzSubnet** "という名前の新しいサブネット"** |
    | **サブネットのアドレス範囲** | **10.0.255.0/24** |
    | **パブリック IP アドレス** | **az12003b-vm0-ip** "という名前の新しい IP アドレス"** |
    | **NIC ネットワーク セキュリティ グループ** | **Basic**  |
    | **パブリック インバウンド ポート** | **[選択したポートを許可する]** |
    | **選択された受信ポート** | **RDP (3389)** |
    | **高速ネットワークの有効化** | "**オフ**" |
    | **負荷分散オプション** | **なし** |
    | **システム割り当てマネージド ID を有効にする** | "**オフ**" |
    | **Azure AD でログインする** | "**オフ**" |
    | **自動シャットダウンを有効にする** | "**オフ**" |
    | **パッチ オーケストレーションのオプション** | **手動更新** |
    | **ブート診断** | **無効にする** |
    | **OS のゲスト診断を有効にする** | "**オフ**" |
    | **拡張機能** | *なし* |
    | **タグ** | *なし* |
   
1. プロビジョニングが完了するまで待ちます。 通常、これには数分かかります。

> **Result**:この演習を完了することにより、高可用性 SAP NetWeaver のデプロイをサポートするために必要となる Azure リソースのプロビジョニングを行いました。


## 演習 2: 高可用性の SAP NetWeaver のデプロイをサポートするように、Windows を実行する Azure VM のオペレーティング システムを構成する

期間:60 分

この演習では、可用性の高い SAP NetWeaver に対応するように、Windows Server を実行している Azure VM 上のオペレーティング システムを構成します。

### タスク 1:Windows Server 2016 Azure VM を Active Directory ドメインに結合します。

   > **注**:このタスクを開始する前に、前のエクササイズで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1. Azure Portal で、このラボの最初のエクササイズで自動的にプロビジョニングされた **adVNET** という名前の仮想ネットワークのブレードに移動します。

1. **[adVNET - DNS サーバー]** ブレードを表示します。仮想ネットワークは、このラボの最初の演習でデプロイされたドメイン コントローラーに割り当てられたプライベート IP アドレスを DNS サーバーとして構成されます。

1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

1. [Cloud Shell] ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Cloud Shell ウィンドウで次のコマンドを実行して、前の演習の 3 番目のタスクでデプロイした Windows Server Azure VM を adatum.com Active Directory ドメインに結合します。

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### タスク 2:データベース層 Azure VM のストレージ構成を確認する。

1. ラボ コンピューターの Azure portal で、**az12003b-vm0** ブレードに移動します。

1. **[az12003b-vm0]** ブレードから、リモート デスクトップ経由で Azure VM az12003b-vm0 に接続します。 プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **学生**

    -   パスワード: *このラボで前に指定したのと同じパスワード*

1. az12003b-vm0 への RDP セッションから、リモート デスクトップを使用して **i20-db-0.adatum.com** Azure VM に接続します。 プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン:**ADATUM\\Student**

    -   パスワード: *このラボで前に指定したのと同じパスワード*

1. リモート デスクトップを使用して、同じ認証情報で **i20-db-1.adatum.com** Azure VM に接続します。

1. i20-db-0.adatum.com への RDP セッション内で、サーバー マネージャーでファイル サービスと記憶域サービスを使用してディスク構成を確認します。 データベースファイルとログファイルのストレージを提供するために、ボリュームマウントを介して単一のデータディスクが構成されていることに注意してください。 

1. i20-db-1.adatum.com への RDP セッション内で、サーバー マネージャーでファイル サービスと記憶域サービスを使用してディスク構成を確認します。 データベースファイルとログファイルのストレージを提供するために、ボリュームマウントを介して単一のデータディスクが構成されていることに注意してください。 


### タスク 3:可用性の高い SAP NetWeaver インストールをサポートするために、Windows Server 2016 を実行している Azure VM でフェールオーバー クラスタリングの構成を準備します。

1. i20-db-0.adatum.com への RDP セッション内で、Windows PowerShell ISE セッションを開始し、それぞれ ASCS および SQL サーバー クラスターのノードとなる ASCS サーバーおよび DB サーバーのペアで以下を実行して、フェールオーバー クラスタリングおよびリモート管理ツール機能をインストールします。

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **注**:これにより、Azure VM 4 つすべてのゲストオペレーティングシステムが再起動されます。

1. ラボ コンピューターで Azure portal を開き、**[+ リソースの作成]** をクリックします。

1. **[新規]** ブレードから、次の設定を使用して新しい**ストレージ アカウント**の作成を開始します。
    
    | 設定 | 値 |
    |   --    |  --   |
    | **サブスクリプション** | "Azure サブスクリプションの名前"** |
    | **リソース グループ** | "可用性の高い SAP NetWeaver のデプロイをホストする Azure VM をデプロイしたリソース グループの名前"** |
    | **Storage account name (ストレージ アカウント名)** | ''3 から 24 の文字と数字で構成される一意の名前''** |
    | **場所** | ''前の演習で Azure VM をデプロイしたのと同じ Azure リージョン''** |
    | **パフォーマンス** | **Standard** |
    | **冗長性** | **ローカル冗長ストレージ (LRS)** |
    | **接続方法** | **パブリック エンドポイント (すべてのネットワーク)** |
    | **REST API 操作の安全な転送を必須にする** | **有効** |
    | **大きいファイルの共有** | **Disabled** |
    | **BLOB、コンテナー、ファイルの論理的な削除** | **Disabled** |
    | **階層構造の名前空間** | **Disabled** |


### タスク 4:可用性の高い SAP NetWeaver のデータベース階層のインストールをサポートするために、Windows Server 2016 を実行している Azure VM のフェールオーバー クラスタリングを構成します。

1. 必要に応じて、az12003b-vm0 への RDP セッションから、リモート デスクトップを使用して **i20-db-0.adatum.com** Azure VM に再接続します。 プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン:**ADATUM\\Student**

    -   パスワード: *このラボで前に指定したのと同じパスワード*

1. i20-db-0.adatum.com への RDP セッションのサーバー マネージャーで **[ローカル サーバー]** ビューに移動し、**[IE セキュリティ強化の構成]** をオフにします。

1. i20-db-0.adatum.com への RDP セッションで、サーバー マネージャーの **[ツール]** メニューから、**[Active Directory 管理センター]** を起動します。

1. Active Directory 管理センターで、adatum.com ドメインのルートに**クラスター**という名前の新しい組織単位を作成します。

1. Active Directory 管理センターで、i20-db-0 および i20-db-1 の**コンピューター** アカウントをコンピューター コンテナーから**クラスター**組織単位に移動します。

1. i20-db-0 への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1. i20-db-0.adatum.com への RDP セッション内で、**[Active Directory 管理センター]** コンソールに切り替えます。

1. Active Directory 管理センターで、**[クラスター]** 組織単位に移動し、**[プロパティ]** ウィンドウを表示します。 

1. **[クラスター]** 組織単位の **[プロパティ]** ウィンドウで、**[拡張機能]** セクションに移動し、**[セキュリティ]** タブを表示します。 

1. **[セキュリティ]** タブで **[詳細]** ボタンをクリックして、**[Advanced Security Settings for Clusters]\(クラスターの詳細なセキュリティ設定\)** ウィンドウを開きます。 

1. **[クラスターの詳細なセキュリティ設定]** ウィンドウの **[アクセス許可]** タブで、**[追加] **をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[プリンシパルの選択]** をクリックします

1. **[ユーザー、サービス アカウントまたはグループの選択]** ダイアログ ボックスで、**[オブジェクトの種類]** をクリックし、**[コンピューター]** エントリの横にあるチェックボックスを有効にし、**[OK]** をクリックします。 

1. **[ユーザー、コンピューター、サービス アカウント、またはグループの選択]** ダイアログ ボックスに戻り、**[選択するオブジェクト名を入力する]** に「**az12003b-db-cl0**」と入力し、**[OK]** をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[種類]** ドロップダウン リストに **[許可]** が表示されるようにします。 次に、**[適用先]** ドロップダウン リストで、**[このオブジェクトとすべての子オブジェクト]** を選択します。 **[アクセス許可]** リストで、**[コンピューター オブジェクトの作成]** チェック ボックスと **[コンピューター オブジェクトの削除]** チェック ボックスをオンにし、**[OK]** を 2 回クリックします。

1. Windows PowerShell ISE セッション内で、次の実行して Az PowerShell モジュールをインストールします。

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して Azure AD 資格情報を使用して認証します。

    ```
    Add-AzAccount
    ```

    > **注**:プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して、変数 `$resourceGroupName` の値を、前のタスクでプロビジョニングしたストレージ アカウントを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して、新しいクラスターの Cloud Witness Quorum を設定します。

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. 結果の構成を確認するには、i20-db-0.adatum.com への RDP セッションのサーバー マネージャーで、**[ツール]** メニューから**フェールオーバー クラスター マネージャー**を起動します。

1. **フェールオーバー クラスター マネージャー** コンソールで、**az12003b-db-cl0** クラスター構成 (ノードを含む)、監視設定およびネットワーク設定を確認します。 クラスターには共有ストレージがないことに注意してください。


### タスク 6:可用性の高い SAP NetWeaver の ASCS 階層のインストールをサポートするために、Windows Server 2016 を実行している Azure VM のフェールオーバー クラスタリングを構成します。

> **注**:このタスクを開始する前に、演習 1 のタスク 4 で開始した S2D クラスターのデプロイが正常に完了していることを確認します。

1. az12003b-vm0 への RDP セッションから、リモート デスクトップを使用して **i20-ascs-0.adatum.com** Azure VM に接続します。 プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン:**ADATUM\\Student**

    -   パスワード: *このラボで前に指定したのと同じパスワード*

1. i20-ascs-0.adatum.com への RDP セッションのサーバー マネージャーで **[ローカル サーバー]** ビューに移動し、**[IE セキュリティ強化の構成]** をオフにします。

1. i20-ascs-0.adatum.com への RDP セッションで、サーバー マネージャーの **[ツール]** メニューから、**Active Directory 管理センター**を起動します。

1. Active Directory 管理センターで、**[コンピューター]** コンテナーに移動します。 

1. Active Directory 管理センターで、i20-ascs-0 および i20-ascs-1 のコンピューター アカウントを**コンピューター** コンテナーから**クラスター**組織単位に移動します。

1. i20-ascs-0.adatum.com への RDP セッションで、Windows PowerShell ISE セッションを開始し、次を実行して新しいクラスターを作成します。

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1. i20-ascs-0.adatum.com への RDP セッション内で、**[Active Directory 管理センター]** コンソールに切り替えます。

1. Active Directory 管理センターで、**[クラスター]** 組織単位に移動し、**[プロパティ]** ウィンドウを表示します。 

1. **[クラスター]** 組織単位の **[プロパティ]** ウィンドウで、**[拡張機能]** セクションに移動し、**[セキュリティ]** タブを表示します。 

1. **[セキュリティ]** タブで **[詳細]** ボタンをクリックして、**[Advanced Security Settings for Clusters]\(クラスターの詳細なセキュリティ設定\)** ウィンドウを開きます。 

1. **[コンピューターの高度セキュリティ設定]\(Advanced Security Settings for Computers\)** ウィンドウの **[アクセス許可]** タブで、**[追加]** をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[プリンシパルの選択]** をクリックします

1. **[ユーザー、サービス アカウントまたはグループの選択]** ダイアログ ボックスで、**[オブジェクトの種類]** をクリックし、**[コンピューター]** エントリの横にあるチェックボックスを有効にし、**[OK]** をクリックします。 

1. **[ユーザー、コンピューター、サービス アカウント、またはグループの選択]** ダイアログ ボックスに戻り、**[選択するオブジェクト名を入力する]** に「**az12003b-ascs-cl0**」と入力し、**[OK]** をクリックします。

1. **[クラスターのアクセス許可エントリ]** ウィンドウで、**[種類]** ドロップダウン リストに **[許可]** が表示されるようにします。 次に、**[適用先]** ドロップダウン リストで、**[このオブジェクトとすべての子オブジェクト]** を選択します。 **[アクセス許可]** リストで、**[コンピューター オブジェクトの作成]** チェック ボックスと **[コンピューター オブジェクトの削除]** チェック ボックスをオンにし、**[OK]** を 2 回クリックします。

1. Windows PowerShell ISE セッション内で、次の実行して Az PowerShell モジュールをインストールします。

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して Azure AD 資格情報を使用して認証します。

    ```
    Add-AzAccount
    ```

    > **注**:プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して、変数 `$resourceGroupName` の値を、このエクササイズでプロビジョニングしたストレージ アカウントを含むリソース グループ名に設定します。

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Windows PowerShell ISE セッション内で、次のコマンドを実行して、クラスターの Cloud Witness Quorum を設定します。

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. 結果の構成を確認するには、i20-ascs-0.adatum.com への RDP セッションのサーバー マネージャーで、**[ツール]** メニューから**フェールオーバー クラスター マネージャー**を起動します。

1. **[フェールオーバー クラスター マネージャー]** コンソールで、**az12003b-ascs-cl0** クラスター構成 (ノードを含む)、監視設定およびネットワーク設定を確認します。 クラスターには共有ストレージがないことに注意してください。


### タスク 7:\\\\GLOBALHOST\\sapmnt 共有にアクセス許可を設定する

このタスクでは、**\\\\GLOBALHOST\\sapmnt** 共有に共有レベルのアクセス許可を設定します。

> **注**:既定では、フル コントロールのアクセス許可は ADATUM\Student アカウントにのみ付与されます。 

1. i20-ascs-0.adatum.com へのリモート デスクトップ セッション内で、**[Windows PowerShell ISE]** ウィンドウから、以下を実行します。

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### タスク 8:SAP NetWeaver ASCS およびデータベース コンポーネントをインストールするためのオペレーティング システムの前提条件を構成する

1. i20-ascs-0.adatum.com へのリモート デスクトップ セッションで、Windows PowerShell ISE セッションから次を実行して、SAP ASCS コンポーネントのインストールと仮想名の使用を調整するのに必要なレジストリ エントリを構成します。

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Result**:このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Windows を実行している Azure VM のオペレーティング システムを構成することができます。


## 演習 3: ラボ リソースを削除する

期間: 10 分

この演習では、このラボでプロビジョニングしたリソースすべてを削除します。

#### タスク 1:Cloud Shell を開く

1. ポータルの上部にある **[Cloud Shell]** アイコンをクリックして Cloud Shell ペインを開き、シェルとして PowerShell を選択します。

1. [Cloud Shell] ペインで次のコマンドを実行して、変数 `$resourceGroupName` の値を、このラボの最初の演習でプロビジョニングした **Windows Server 2019 Datacenter** Azure VM のペアを含むリソース グループ名に設定します。

    ```
    $resourceGroupNamePrefix = 'az12003b-'
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
