---
lab:
  title: 04a - Linux を実行する Azure VM に SAP アーキテクチャを実装する
  module: Module 04 - Deploy SAP on Azure
ms.openlocfilehash: 477438705acbd5fc3c0ac796353e77ef8a352dc5
ms.sourcegitcommit: 2d98b3c8cdd6f7b2b1a9a43868559bef227a5266
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/19/2022
ms.locfileid: "145179694"
---
# <a name="az-120-module-4-deploy-sap-on-azure"></a>AZ 120 モジュール 4:SAP on Azure のデプロイ
# <a name="lab-4a-implement-sap-architecture-on-azure-vms-running-linux"></a>ラボ 4a:Linux を実行する Azure VM に SAP アーキテクチャを実装する

予測される所要時間:100 分

このラボのタスクすべては、Azure portal (Bash Cloud Shell セッションを含む) から実行されます  

   > **注**:Cloud Shell を使用しない場合は、ラボ仮想マシンに Azure CLI がインストールされている必要があります ([ **https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows))。

ラボ ファイル: なし

## <a name="scenario"></a>シナリオ
  
Azure に SAP NetWeaver をデプロイする準備として、Adatum Corporation は、Linux の SUSE ディストリビューションを実行する Azure VM での SAP NetWeaver の高可用性実装がわかるデモを実装したいと考えています。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

-   高可用性な SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

-   可用性の高い SAP NetWeaver のデプロイをサポートするように、Linux を実行している Azure VM の OS を構成する

-   可用性の高い SAP NetWeaver のデプロイをサポートするように、Linux を実行している Azure VM でクラスタリングを構成する

## <a name="requirements"></a>要件

-   可用性ゾーンをサポートする Azure リージョン内で、十分な数の Dsv3 vCPU (2 vCPU の Standard_D2s_v3 VM を 4 台、4 vCPU の Standard_D4s_v3 VM を 2 台) が利用可能な Microsoft Azure サブスクリプション

-   Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター


## <a name="exercise-1-provision-azure-resources-necessary-to-support-highly-available-sap-netweaver-deployments"></a>演習 1: 高可用性の SAP NetWeaver のデプロイをサポートするために必要な Azure リソースをプロビジョニングする

期間: 30 分

この演習では、Linux クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。 これには、同じ可用性セットで Linux SUSE を実行する Azure VM のペアを作成する必要があります。

### <a name="task-1-create-a-virtual-network-that-will-host-a-highly-available-sap-netweaver-deployment"></a>タスク 1:可用性の高い SAP NetWeaver のデプロイをホストする仮想ネットワークを作成します。

1.  ラボ コンピューターから Web ブラウザーを起動し、 https://portal.azure.com から Azure portal に移動します

1.  プロンプトが表示された場合は、このラボおよびサブスクリプションに関連付けられた Azure AD テナントのグローバル管理者ロールで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

    > **注**:現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1.  [Cloud Shell] ペインで次のコマンドを実行して、可用性ゾーンをサポートする Azure リージョンと、このラボのリソースを作成する場所を指定します (`<region>` を、可用性ゾーンをサポートする Azure リージョンの名前に置き換えます)。

    ```cli
    LOCATION='<region>'
    ```

    > **注**:リソースのデプロイには、**米国東部** または **米国東部 2** リージョンの使用を検討してください。 

    > **注**:Azure リージョンに適切な表記を使用してください (**US East** ではなく **eastus** など、スペースを含まない短い名前)

    > **注**:可用性ゾーンをサポートする Azure リージョンを特定するには、[https://docs.microsoft.com/en-us/azure/availability-zones/az-region](https://docs.microsoft.com/en-us/azure/availability-zones/az-region) を参照してください

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_NAME`」の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

    ```cli
    RESOURCE_GROUP_NAME='az12003a-sap-RG'
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、指定したリージョンにリソースグループを作成します。

    ```cli
    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、作成したリソース グループ内に 1 つのサブネットを有する仮想ネットワークを作成します:

    ```cli
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  Cloud Shell ペインで次のコマンドを実行して、新しく作成した仮想ネットワークにあるサブネットのリソース ID を特定します。

    ```cli
    az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv
    ```

1.  結果の値をクリップボードにコピーします。 これは、次のタスクで必要になります。

### <a name="task-2-deploy-azure-resource-manager-template-provisioning-azure-vms-running-linux-suse-that-will-host-a-highly-available-sap-netweaver-deployment"></a>タスク 2:可用性の高い SAP NetWeaver のデプロイをホストする Linux SUSE が稼働する Azure VM をプロビジョニングするために Azure Resource Manager テンプレートをデプロイする

1.  ラボ コンピューターでブラウザーを起動し、[ **https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/sap/sap-3-tier-marketplace-image-md**](https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/sap/sap-3-tier-marketplace-image-md) を参照します

    > **注**:Microsoft Edge またはサード パーティのブラウザーを使用してください。 Internet Explorer は使用しないでください。

1.  **SAP NetWeaver 3-tier compatible template using a Marketplace image - MD** (Marketplace イメージを使った SAP NetWeaver 3 層構成互換テンプレート - MD) というタイトルのページで、 **「Azure にデプロイ」** をクリックします。 これにより、Azure portal に自動的にリダイレクトされ、**SAP NetWeaver 3-tier (managed disk)** ブレードが表示されます。

1.  **SAP NetWeaver 3-tier (managed disk)** ブレードで、 **「テンプレートの編集」** .を選択します。

1.  **「テンプレートの編集」** ブレードで、次の変更を適用して **「保存」** を選択します。

    -   **197** の行で、`"dbVMSize": "Standard_E8s_v3",` を `"dbVMSize": "Standard_D4s_v3",` に置き換えます

1.  **[SAP NetWeaver 3 層 (マネージド ディスク)]** ブレードで、以下の設定を使用してデプロイを開始します。

    -   サブスクリプション: *Azure サブスクリプションの名前。*

    -   リソース グループ: *前のタスクで使用したリソース グループの名前*

    -   場所: *この演習の最初のタスクで指定したものと同じ Azure リージョン*

    -   SAP システム ID:**I20**

    -   スタック タイプ:**ABAP**

    -   OS タイプ:**SLES 12**

    -   Dbtype:**HANA**

    -   Sap システム サイズ:**デモ**

    -   システム可用性:**HA**

    -   管理者ユーザー名: **受講生**

    -   認証タイプ: **パスワード**

    -   管理者パスワードまたはキー:**Pa55w.rd1234**

    -   サブネット ID: *前のタスクでクリップボードにコピーした値*

    -   可用性ゾーン:**1、2**

    -   場所: **[resourceGroup().location]**

    -   _artifacts Location (成果物の場所): **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/**

    -   _アーティファクト ロケーション SAS トークン: *空白のままにする*

1.  デプロイが完了するのを待たずに、代わりに次のタスクに進みます。 

    > **注**:CustomScriptExtension コンポーネントのデプロイ中に **競合** エラー メッセージが表示され、デプロイが失敗した場合は、次の手順を使用してこの問題を修復します。

       - Azure portal の「**デプロイ**」ブレードで、デプロイの詳細を確認し、CustomScriptExtension のインストールが失敗した VM を特定します

       - Azure portal で、前の手順で特定した VM のブレードに移動し、「**拡張機能**」を選択し、「**拡張機能**」ブレードから CustomScript 拡張機能を削除します

       - Azure portal で、 **[az12003a-sap-RG]** リソース グループ ブレードに移動し、 **[デプロイ]** を選択して、失敗したデプロイへのリンクを選択します。 **[再デプロイ]** を選択し、ターゲット リソース グループ (**az12003a-sap-RG**) を選択して、ルート アカウントのパスワードを指定します (**Pa55w.rd1234**)。


### <a name="task-3-deploy-a-jump-host"></a>タスク 3:ジャンプ ホストをデプロイする

   > **注**:前のタスクでデプロイした Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1.  ラボ コンピューターの Azure portal で、 **「+ リソースの作成」** をクリックします。

1.  「**新規**」 ブレードから、**Windows Server 2019 Datacenter** イメージを基に新規 Azure VM の作成を開始します。

1.  Azure VM を次の設定でプロビジョニングします (その他の設定は既定値のままにします)。

    -   サブスクリプション: *Azure サブスクリプションの名前。*

    -   リソース グループ: *"新しいリソース グループの名前"* **az12003a-dmz-RG**

    -   仮想マシン名: **az12003a-vm0**

    -   リージョン: *このエクササイズの最初のタスクで Azure VM をデプロイしたのと同じ Azure リージョン*

    -   可用性オプション:**インフラストラクチャ冗長は必要ありません**

    -   イメージ:**Windows Server 2019 Datacenter - Gen2**

    -   サイズ:**Standard D2s_v3** または類似のもの

    -   ユーザー名: **Student**

    -   パスワード: **Pa55w.rd1234**

    -   パブリック インバウンド ポート: **[選択したポートを許可する]**

    -   受信ポートを選択 **RDP (3389)**

    -   Windows ライセンスを持っていますか?:"**いいえ**"

    -   OS ディスク タイプ:**Standard HDD**

    -   仮想ネットワーク: **az12003a-sap-vnet**

    -   サブネット: **bastionSubnet(10.3.255.0/24)** という名前の新しいサブネット

    -   パブリック IP: **az12003a-vm0-ip** *"という名前の新規 IP アドレス"*

    -   NIC ネットワーク セキュリティ グループ:**Basic**

    -   パブリック インバウンド ポート: **[選択したポートを許可する]**

    -   受信ポートを選択 **RDP (3389)**

    -   高速ネットワーキング:"**オフ**"

    -   この仮想マシンを既存の負荷分散ソリューションの背後に配置します。"**いいえ**"

    -   ブート診断:**無効化**

    -   OS ゲスト診断:"**オフ**"

    -   システム割り当てマネージド ID:"**オフ**"

    -   AAD 資格情報を使用してログインする (プレビュー):"**オフ**"

    -   自動シャットダウンを有効にする:"**オフ**"

    -   バックアップを有効にする:"**オフ**"

    -   パッチ オーケストレーション オプション **手動更新**

    -   拡張機能: **なし**

    -   タグ:**なし**

1.  プロビジョニングが完了するまで待ちます。 通常、これには数分かかります。

> **Result**:この演習を完了することにより、高可用性 SAP NetWeaver のデプロイをサポートするために必要となる Azure リソースのプロビジョニングを行いました。


## <a name="exercise-2-configure-azure-vms-running-linux-to-support-a-highly-available-sap-netweaver-deployment"></a>演習 2: 高可用性の SAP NetWeaver のデプロイをサポートするように、Linux を実行する Azure VM を構成する

期間: 30 分

このエクササイズでは、可用性の高い SAP NetWeaver に対応するように、SUSE Linux Enterprise Server を実行している Azure VM 上のオペレーティングシステムを構成します。

### <a name="task-1-configure-networking-of-the-database-tier-azure-vms"></a>タスク 1:データベース層 Azure VM のネットワークを構成します。

   > **注**:このタスクを開始する前に、前のエクササイズで開始したテンプレートのデプロイが正常に完了したことを確認します。 

1.  ラボ コンピューターの Azure portal で、**i20-db-0** Azure VM のブレードに移動します。

1.  **i20-db-0** ブレードから、**ネットワーク** ブレードに移動します。 

1.  **i20-vm0 - ネットワーク** ブレードから、i20-vm0 のネットワーク インターフェイスに移動します。 

1.  i20-db-0 のネットワーク インターフェイスのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスを **10.3.0.20** に設定し、割り当てを「**静的**」に変更して変更を保存します。

1.  Azure portal で、**i20-db-1** Azure VM ブレードに移動します。

1.  **[i20-db-1]** ブレードから、その **[ネットワーク]** ブレードに移動します。 

1.  **i20-db-1 - ネットワーク** ブレードから、i20-db-1 のネットワーク インターフェイスに移動します。 

1.  i20-db-1 のネットワーク インターフェイスのブレードから、その [IP 構成] ブレードに移動し、そこから **[ipconfig1]** ブレードを表示します。

1.  **ipconfig1** ブレードで、プライベート IP アドレスを **10.3.0.21** に設定し、割り当てを **静的** に変更して保存します。


### <a name="task-2-connect-to-the-database-tier-azure-vms"></a>タスク 2:Azure VM のデータベース層に接続します。

1.  ラボ コンピューターの Azure portal で、**az12003a-vm0** ブレードに移動します。

1.  **az12003a-vm0** ブレードから、リモート デスクトップ経由で Azure VM az12003a-vm0 に接続します。 

1.  az12003a-vm0 への RDP セッションのサーバー マネージャーで **ローカル サーバー** ビューに移動し、 **「IE セキュリティ強化の構成」** をオフにします。

1.  az12003a-vm0 への RDP セッション内で、[ **https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から PuTTY をダウンロードしてインストールします。

1.  PuTTY を使用して、SSH 経由で **i20-db-0** Azure VM に接続します。 セキュリティ警告を確認し、プロンプトが表示されたら、次の認証情報を入力します。

    -   ログイン: **受講生**

    -   パスワード: **Pa55w.rd1234**

1.  PuTTY を使用して、同じ認証情報を使って SSH 経由で **i20 db-1** Azure VM に接続します。


### <a name="task-3-examine-the-storage-configuration-of-the-database-tier-azure-vms"></a>タスク 3:データベース層 Azure VM のストレージ構成を確認する。

1.  PuTTY SSH セッション～i20-db-0 Azure VM 内で、次のコマンドを実行して権限を昇格させます: 

    ```
    sudo su -
    ```

1.  パスワードの入力を求めるメッセージが表示されたら、 **「Pa55w.rd1234」** と入力し、**Enter** キーを押します。 

1.  i20-db-0 への SSH セッションで、次を実行することにより、SAP HANA 関連ボリュームすべて ( **/usr/sap**、 **/hana/shared**、 **/hana/backup**、 **/hana/data**、 **/hana/logs** を含む) が適切にマウントされていることを確認します。

    ```
    df -h
    ```

1.  i20-db-1 Azure VM の前の手順を繰り返します。


### <a name="task-4-enable-cross-node-password-less-ssh-access"></a>タスク 4:クロスノード パスワードレス SSH アクセスを有効にする

1.  i20-db-0 への SSH セッションで、次を実行してパスフレーズ不要の SSH キーを生成します。

    ```
    ssh-keygen
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-1 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  vi エディターで、i20-db-0 で生成したキーを貼り付けます。

1.  変更を保存してエディターを閉じます。

1.  SSH セッション～i20-db-1 内で、次のコマンドを実行してパスフレーズ不要の SSH キーを生成します。

    ```
    ssh-keygen
    ```

1.  プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行してキーを表示します。 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1.  キーの値をクリップボードにコピーします。

1.  i20-db-0 への SSH セッションで、次を実行して、vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します。

    ```
    vi /root/.ssh/authorized_keys
    ```

1.  vi エディターで、新しい行から開始する i20-db-1 で生成したキーを貼り付けます。

1.  変更を保存してエディターを閉じます。

1.  i20-db-0 への SSH セッションで構成が成功したことを確認するには、次を実行して、i20-db-0 から i20-db-1 への SSH セッションを **root** として確立します。 

    ```
    ssh root@i20-db-1
    ```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、「`yes`」 と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  次の手順を実行して、i20-db-0 から i20-db-1 への SSH セッションを閉じます。 

    ```
    exit
    ```

1.  i20-db-1 への SSH セッションで、次の手順を実行して、i20-db-1 から i20-db-0 への SSH セッションを **root** として確立します。 

    ```
    ssh root@i20-db-0
    ```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、「`yes`」 と入力し、**Enter** キーを押します。 

1.  パスワードの入力が求められないことを確認します。

1.  以下を実行して、i20-db-1 から i20-db-0 への SSH セッションを閉じます。 

    ```
    exit
    ```

### <a name="task-5-add-yast-packages-update-the-linux-operating-system-and-install-ha-extensions"></a>タスク 5:YaST パッケージを追加し、Linux オペレーティング システムを更新し、HA 拡張機能をインストールする

1.  i20-db-0 への SSH セッションで、次を実行して YaST を起動します。

    ```
    yast
    ```

1.  **YaST コントロール センター** で、 **[ソフトウェア] -\> [アドオン製品]** を選択し、**Enter** キーを押します。 これにより **Package Manager** が読み込まれます。

1.  **インストールされたアドオン製品** 画面で、**パブリック クラウド モジュール** が既にインストールされていることを確認します。 次に、**F9** を 2 回押してシェル プロンプトに戻ります。

1.  i20-db-0 への SSH セッションで、次を実行してオペレーティング システムを更新します (プロンプトが表示されたら、**y** と入力し、**Enter** キーを押します)。

    ```
    zypper update
    ```

1.  i20-db-0 への SSH セッションで、次を実行してクラスター リソースで必要なパッケージをインストールします (プロンプトが表示されたら、**y** と入力し、**Enter** キーを押します)。

    ```
    zypper in socat
    ```

1.  i20-db-0 への SSH セッションで、次のコマンドを実行して、クラスター リソースに必要な azure-lb コンポーネントをインストールします。

    ```
    zypper in resource-agents
    ```

1.  i20-db-0 への SSH セッションで、次を実行して、vi エディターでファイル **/etc/systemd/system.conf** を開きます。

    ```
    vi /etc/systemd/system.conf
    ```

1.  vi エディターで、`#DefaultTasksMax=512` を `DefaultTasksMax=4096` に置き換えます。 

    > **注**:場合によっては、Pacemaker が多数のプロセスを作成し、その数がデフォルトの制限値に達してフェールオーバーをトリガーすることがあります。 この変更により、許可されるプロセスの最大数が増加します。

1.  変更を保存してエディターを閉じます。

1.  i20-db-0 への SSH セッションで、次を実行して構成の変更をアクティブ化します。

    ```
    systemctl daemon-reload
    ```

1. i20-db-0 への SSH セッションで、次を実行してフェンス エージェント パッケージをインストールします。

    ```
    zypper install fence-agents
    ```

1. i20-db-0 への SSH セッションで、次を実行してフェンス エージェントで必要な Azure Python SDK パッケージをインストールします (プロンプトが表示されたら、**y** と入力し、**Enter** キーを押します)。

    ```
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    zypper install python-azure-mgmt-compute
    ```

1. i20-db-1 でこのタスクの前の手順を繰り返します。

> **Result**:このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Linux を実行している Azure VM のオペレーティング システムを構成することができます

## <a name="exercise-3-configure-clustering-on-azure-vms-running-linux-to-support-a-highly-available-sap-netweaver-deployment"></a>演習 3: 高可用性の SAP NetWeaver のデプロイをサポートするように、Linux を実行する Azure VM 上のクラスタリングを構成する

期間: 30 分

このエクササイズでは、可用性の高い SAP NetWeaver のデプロイをサポートするために、Linux を実行している Azure VM にクラスタリングを構成します。

### <a name="task-1-configure-clustering"></a>タスク 1:クラスタリングを構成する

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションから次を実行して、i20-db-0 上の HA クラスターの構成を開始します。

    ```
    ha-cluster-init -u
    ```

1.  プロンプトが表示されたら、次の情報を入力します。

    -   続行しますか (y/n)?: **y**

    -   リング0 [10.3.0.20] のアドレス:**ENTER**

    -   ring0 [5405] のポート:**ENTER**

    -   SBD を使用しますか (y/n)?: **n**

    -   仮想 IP アドレスを構成しますか (y/n)?: **n**

    > **注**:クラスタリングのセットアップでは、パスワードが **Linux** に設定された **hacluster** アカウントが生成されます。 これについては、タスクの後半で変更します。

1.  az12003a-vm0 への RDP セッション内で、i20-db-1 への PuTTY ベースの SSH セッションから次を実行して、i20-db-1 から i20-db-0 上に HA クラスターを結合します。

    ```
    ha-cluster-join
    ```

1.  プロンプトが表示されたら、次の情報を入力します。

    -   続行しますか (y/n)? **y**

    -   既存のノードの IP アドレスまたはホスト名 (例:192.168.1.1) \[\]: **i20-db-0**

    -   リング0 [10.3.0.21] のアドレス:**ENTER**

1.  i20-db-0 への PuTTY ベースの SSH セッションから、次を実行して、**hacluster** アカウントのパスワードを **Pa55w.rd1234** に設定します (プロンプトが表示されたら新しいパスワードを入力します)。 

    ```
    passwd hacluster
    ```

1.  i20-db-1 の前の手順を繰り返します。

### <a name="task-2-review-corosync-configuration"></a>タスク 2:corosync 構成の確認

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションから次を実行して、 **/etc/corosync/corosync.conf** ファイルを開きます。

    ```
    vi /etc/corosync/corosync.conf
    ```

1.  vi エディターで、`transport: udpu` エントリとセクション `nodelist` に注意してください。
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1.  vi エディターで、エントリ `token: 5000` を `token: 30000` に置き換えます。

    > **注**:この変更により、メモリを保持したメンテナンスが可能になります。 詳細については、[Azure での仮想マシンのメンテナンスに関する Microsoft ドキュメント](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates#maintenance-that-doesnt-require-a-reboot)を参照してください。

1.  変更を保存してエディターを閉じます。

1.  i20-db-1 の前の手順を繰り返します。


### <a name="task-3-identify-the-value-of-the-azure-subscription-id-and-the-azure-ad-tenant-id"></a>タスク 3:Azure サブスクリプション ID と Azure AD テナント ID の値を特定する

1.  ラボ コンピューターのブラウザー ウィンドウから Azure portal ( **https://portal.azure.com** ) を開き、サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つユーザー アカウントを使用してサインインしていることを確認します。

1.  Azure portal の Cloud Shell で Bash セッションを開始します。 

1.  Cloud Shell ペインで、次のコマンドを実行して、Azure サブスクリプションの ID と対応する Azure AD テナントの ID を特定します。

    ```cli
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1.  結果の値を Notepad にコピーします。 これは、次のタスクで必要になります。


### <a name="task-4-create-an-azure-ad-application-for-the-stonith-device"></a>タスク 4:STONITH デバイス用の Azure AD アプリケーションの作成

1.  Azure portal で、 [Azure Active Directory] ブレードに移動します。

1.  Azure Active Directory ブレードから、**アプリの登録** ブレードに移動し、 **「+ 新規登録」** をクリックします。

1.  **「アプリケーションの登録」** ブレードで、次の設定を指定し、 **「登録」** をクリックします。

    -   名前: **Stonith app**

    -   サポートされるアカウントの種類:**この組織のディレクトリ内のアカウントのみ**

1.  **Stonith アプリ** ブレードで、**アプリケーション (クライアント) ID** の値を Notepad にコピーします。 これは、このエクササイズの後半で **login_id** として参照します。

1.  **Stonith アプリ** ブレードで、 **「証明書とシークレット」** をクリックします。

1.  **Stonith アプリ - 証明書とシークレット** ブレードで、 **「+ 新しいクライアント シークレット」** をクリックします。

1.  **[クライアント シークレットの追加]** ペインの **[説明]** テキスト ボックスで、 **[有効期限]** セクションに「**STONITH app key**」と入力し、既定の「**推奨:6 か月**」のままにして、 **[追加]** をクリックします。

1.  結果の **値** をメモ帳にコピーします (このエントリは、 **[追加]** をクリックした後 1 度だけ表示されます)。 これは、このエクササイズの後半で **パスワード** として参照します。


### <a name="task-5-grant-permissions-to-azure-vms-to-the-service-principal-of-the-stonith-app"></a>タスク 5:STONITH アプリのサービス プリンシパルに Azure VM へのアクセス許可を付与する 

1.  Azure portal で、 **i20-db-0** Azure VM のブレードに移動します。

1.  **i20-db-0** ブレードから、**i20-db-0 - アクセスの制御 (IAM)** ブレードを表示します。

1.  **i20-db-0 - アクセスの制御 (IAM)** ブレードから、次の設定でロールの割り当てを追加します。

    -   ロール:**Virtual Machine Contributor**

    -   アクセスの割り当て先:**Azure AD のユーザー、グループ、サービス プリンシパル**

    -   選択:**Stonith app**

1.  前の手順を繰り返して、Stonith アプリの仮想マシン共同作成者ロールを **i20-db-1** Azure VM に割り当てます。


### <a name="task-6-configure-the-stonith-cluster-device"></a>タスク 6:STONITH クラスター デバイスを構成する 

1.  az12003a-vm0 への RDPセッション 内で、PuTTY ベースの SSH セッションを i20-db-0 に切り替えます。

1.  az12003a-vm0 への RDP セッション内で、i20-db-0 への PuTTY ベースの SSH セッションで、次のコマンドを実行します (`subscription_id`、`tenant_id`、`login_id,`、`password` のプレースホルダーを、必ず演習 3 のタスク 4 で特定した値に置き換えてください)。

    ```
    crm configure property stonith-enabled=true

    crm configure property concurrent-fencing=true

    crm configure primitive rsc_st_azure stonith:fence_azure_arm \
      params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password" \
      pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 \
      op monitor interval=3600 timeout=120

    sudo crm configure property stonith-timeout=900
    ```

### <a name="task-7-review-clustering-configuration-on-azure-vms-running-linux-by-using-hawk"></a>タスク 7:Hawk を使用して Linux を実行している Azure VM のクラスタリング構成を確認する

1.  az12003a-vm0 への RDP セッション内で、Internet Explorer を起動し、 **https://i20-db-0:7630** に移動します。 SUSE Hawk サインイン ページが表示されます。

   > **注**:**このサイトはセキュリティで保護されていません** のメッセージを無視します。

1.  SUSE Hawk サインイン ページで、次の認証情報を使用してログインします。

    -   ユーザー名: **hacluster**

    -   パスワード: **Pa55w.rd1234**

1.  クラスターの状態が正常であることを確認します。 2 つのクラスター ノードの 1 つが不完全であることを示すメッセージが表示される場合は、Azure portal からそのノードを再起動します。

> **Result**:このエクササイズを完了すると、可用性の高い SAP NetWeaver デプロイをサポートするように、Linux を実行している Azure VM にクラスタリングを構成することができます。


## <a name="exercise-4-remove-lab-resources"></a>演習 4: ラボ リソースを削除する

期間: 10 分

この演習では、このラボでプロビジョニングしたリソースすべてを削除します。

#### <a name="task-1-open-cloud-shell"></a>タスク 1:Cloud Shell を開く

1. ポータルの上部にある「**Cloud Shell**」アイコンをクリックして Cloud Shell ペインを開き、シェルとして Bash を選択します。

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_PREFIX`」の値を、このラボでプロビジョニングしたリソースを含むリソース グループ名のプレフィックスに設定します。

    ```cli
    RESOURCE_GROUP_PREFIX='az12003a-'
    ```

1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成したすべてのリソース グループをリストアップします。

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
    ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。 このリソース グループとそのすべてのリソースは、次のタスクで削除されます。

#### <a name="task-2-delete-resource-groups"></a>タスク 2: リソース グループを削除する

1. Cloud Shell ペインで次のコマンドを実行して、リソース グループとそのリソースを削除します。

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. [Cloud Shell] ペインを閉じます。

> **Result**:この演習を完了すると、このラボで使用したリソースが削除されます。
