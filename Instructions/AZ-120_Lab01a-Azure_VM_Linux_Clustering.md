---
ms.openlocfilehash: 719d5988e5f36bef026d781ba3f5f82893e77efc
ms.sourcegitcommit: 3d7a4cc1ab7f6dcb68f259736361cfe76302b595
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/24/2022
ms.locfileid: "140872846"
---
# <a name="az-120-module-2-explore-the-foundations-of-iaas-for-sap-on-azure"></a>AZ 120 モジュール 2:IaaS for SAP on Azure の基盤を探る
# <a name="lab-1a-implement-linux-clustering-on-azure-vms"></a>ラボ 1a:Azure VM での Linux クラスタリングの実装

予測される所要時間:90 分

このラボのタスクすべては、Azure portal (Bash Cloud Shell セッションを含む) から実行されます  

   > **注**:Cloud Shell を使用しない場合は、ラボ仮想マシンに Azure CLI がインストールされ ([ **https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows))、[ **https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html** ](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から使用可能な PuTTY などの SSH クライアントが含まれている必要があります。

ラボ ファイル: なし

## <a name="scenario"></a>シナリオ
  
Adatum Corporation は、Azure に SAP HANA をデプロイする準備として、Linux の SUSE ディストリビューションが実行されている Azure VM にクラスタリングを実装するプロセスを確認したいと思っています。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行う

- 可用性の高い SAP HANA のインストールをサポートするために、Linux を実行している Azure VM のオペレーティング システムを構成する

- 可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行う

## <a name="requirements"></a>必要条件

- 使用可能な DSv3 vCPU (4 x 4) と DSv2 (1 x 1) vCPU の十分な数を持つ Microsoft Azure サブスクリプション

- Azure Cloud Shell に対応した Web ブラウザーと Azure へのアクセスが可能なラボ コンピューター

## <a name="exercise-1-provision-azure-compute-resources-necessary-to-support-highly-available-sap-hana-deployments"></a>演習 1: 高可用性の SAP HANA のデプロイをサポートするために必要な Azure コンピューティング リソースをプロビジョニングする

期間: 30 分

この演習では、Linux クラスタリングの構成に必要な Azure インフラストラクチャ コンピューティング コンポーネントをデプロイします。 これには、同じ可用性セットで Linux SUSE を実行する Azure VM のペアを作成する必要があります。

### <a name="task-1-deploy-azure-vms-running-linux-suse"></a>タスク 1:Linux SUSE を実行している Azure VM をデプロイする

1. ラボ コンピューターから Web ブラウザーを起動し、 https://portal.azure.com から Azure portal に移動します

1. プロンプトが表示された場合は、このラボで使用する Azure サブスクリプションの所有者または共同作成者のロールを使用して、職場、学校または個人の Microsoft アカウントを使用してログインします。

1. Azure portal で、Azure portal ページの上部にある **「リソース、サービス、およびドキュメントの検索」** テキスト ボックスを使用して、 **「近接配置グループ」** ブレードを検索してナビゲートし、 **「近接配置グループ」** ブレードで **「+ 作成」** を選択します。

1. **「近接配置グループの作成」** ブレードの **「基本」** タブで次の設定を指定し、 **「Review + create」** を選択します。

   - サブスクリプション: *Azure サブスクリプションの名前。*

   - リソース グループ: リソース グループ: "*新しいリソース グループの名前*" **az12001a-RG**

   > **注**:リソースのデプロイには、**米国東部** または **米国東部 2** リージョンの使用を検討してください。 

   - リージョン: *Azure VM をデプロイできる Azure リージョン*

   - 近接配置グループ名: **az12001a-ppg**

1. **「近接配置グループの作成」** ブレードの **「Review + create」** タブで、「**作成**」を選択します。

   > **注**:プロビジョニングが完了するまで待ちます。 これに要する時間は 1 分未満です。

1. Azure portal で、Azure portal ページの上部にある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスを使用して **[仮想マシン]** ブレードを検索して移動し、 **[仮想マシン]** ブレードで **[+ 作成]** を選択し、ドロップダウン メニューで **[Azure 仮想マシン]** を選択します。

1. **[仮想マシンの作成]** ブレードの **[基本]** タブで、次の設定を指定して **[次へ:ディスク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - サブスクリプション: *Azure サブスクリプションの名前。*

   - リソース グループ: *このタスクで前に使用したリソース グループ名*

   - 仮想マシン名 = **az12001a-vm0**

   - リージョン: *近接配置グループを作成したときに選択したものと同じ Azure リージョン*

   - 可用性オプション:**可用性セット**

   - 可用性セット: "*2 つの障害ドメインと 5 つの更新ドメインを持つ*" **az12001a-avset** "*という名前の新規可用性セット*"

   - イメージ:**SUSE Enterprise Linux for SAP 12 SP5 - BYOS - Gen 1**
   
   > **注**:イメージを検索するには、 **「イメージの選択」** ブレードで **「すべてのイメージを表示」** リンクをクリックし、検索テキスト ボックスに **「SUSE Enterprise Linux for SAP 12 BYOS」** を入力し、結果のリストで、 **「SUSE Enterprise Linux for SAP 12 SP5 - BYOS」** を選択します。

   - Azure Spot インスタンス:"**いいえ**"

   - サイズ:**Standard D4s v3**

   - 認証の種類: **パスワード**

   - ユーザー名: **学生**

   - パスワード: **Pa55w.rd1234**

1. **[仮想マシンの作成]** ブレードの **[ディスク]** タブで、次の設定を指定して **[次へ:ネットワーク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - OS ディスク タイプ:**Premium SSD**

   - 暗号化の種類: **(既定) プラットフォーム マネージド キーを使用した保存時の暗号化**

1. **[仮想マシンの作成]** ブレードの **[ネットワーク]** タブで、次の設定を指定して **[次へ:管理 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 仮想ネットワーク: **az12001a-RG-vnet** "*という名前の新規仮想ネットワーク*"

   - アドレス空間：**192.168.0.0/20**

   - サブネット名: **subnet-0**

   - サブネット アドレス範囲:**192.168.0.0/24**

   - パブリック IP アドレス: **az12001a-vm0-ip** "*という名前の新規 IP アドレス*"

   - NIC ネットワーク セキュリティ グループ:**詳細**

   > **注**:このイメージは NSG ルールを事前に構成しました

   - 高速ネットワーキング:**基準**

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。"**いいえ**"

1. **[仮想マシンの作成]** ブレードの **[管理]** タブで、次の設定を指定して **[次へ:詳細 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 基本プランを有効にする (無料):"**いいえ**"

   > **注**:この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断:**マネージド ストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断:"**オフ**"

   - システム割り当てマネージド ID:"**オフ**"

   - 自動シャットダウンを有効にする:"**オフ**"

1. 「**仮想マシンの作成**」ブレードの「**詳細**」タブで、次の設定を指定して **「Review + create」** を選択します (既定値を他のユーザーのままにします)。

   - 近接配置グループ: **az12001a-ppg**

1. 「**仮想ネットワークの作成**」ブレードの「**Review + create**」タブで、「**作成**」を選択します。

   > **注**:プロビジョニングが完了するまで待ちます。 これは 3 分もかかりません。

1. Azure portal で、Azure portal ページの上部にある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスを使用して **[仮想マシン]** ブレードを検索して移動し、 **[仮想マシン]** ブレードで **[+ 作成]** を選択し、ドロップダウン メニューで **[Azure 仮想マシン]** を選択します。

1. **[仮想マシンの作成]** ブレードの **[基本]** タブで、次の設定を指定して **[次へ:ディスク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - サブスクリプション: *Azure サブスクリプションの名前。*

   - リソース グループ: *このタスクで前に使用したリソース グループ名*

   - 仮想マシン名: **az12001a-vm1**

   - リージョン: *最初の Azure VM を作成したときに選択したものと同じ Azure リージョン*

   - 可用性オプション:**可用性セット**

   - 可用性セット: **az12001a-avset**

   - イメージ:**SUSE Enterprise Linux for SAP 12 SP5 - BYOS - Gen 1**
   
   > **注**:イメージを検索するには、 **「イメージの選択」** ブレードで **「すべてのイメージを表示」** リンクをクリックし、検索テキスト ボックスに **「SUSE Enterprise Linux for SAP 12 BYOS」** を入力し、結果のリストで、 **「SUSE Enterprise Linux for SAP 12 SP5 - BYOS」** を選択します。

   - Azure Spot インスタンス:"**いいえ**"

   - サイズ:**Standard D4s v3**

   - 認証の種類: **パスワード**

   - ユーザー名: **学生**

   - パスワード: **Pa55w.rd1234**

1. **[仮想マシンの作成]** ブレードの **[ディスク]** タブで、次の設定を指定して **[次へ:ネットワーク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - OS ディスク タイプ:**Premium SSD**

   - 暗号化の種類: **(既定) プラットフォーム マネージド キーを使用した保存時の暗号化**

1. **[仮想マシンの作成]** ブレードの **[ネットワーク]** タブで、次の設定を指定して **[次へ:管理 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット: **subnet-0 (192.168.0.0/24)**

   - パブリック IP アドレス: **az12001a-vm1-ip** "*という名前の新規 IP アドレス*"

   - NIC ネットワーク セキュリティ グループ:**詳細**

   > **注**:このイメージは NSG ルールを事前に構成しました

   - 高速ネットワーキング:**基準**

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。"**いいえ**"

1. **[仮想マシンの作成]** ブレードの **[管理]** タブで、次の設定を指定して **[次へ:詳細 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 基本プランを有効にする (無料):"**いいえ**"

   > **注**:この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断:**マネージド ストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断:"**オフ**"

   - システム割り当てマネージド ID:"**オフ**"

   - 自動シャットダウンを有効にする:"**オフ**"

1. 「**仮想マシンの作成**」ブレードの「**詳細**」タブで、次の設定を指定して **「Review + create」** を選択します (既定値を他のユーザーのままにします)。

   - 近接配置グループ: **az12001a-ppg**

1. 「**仮想ネットワークの作成**」ブレードの「**Review + create**」タブで、「**作成**」を選択します。

   > **注**:プロビジョニングが完了するまで待ちます。 これは 3 分もかかりません。


### <a name="task-2-create-and-configure-azure-vms-disks"></a>タスク 2:Azure VM ディスクを作成して構成する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

   > **注**:現在の Azure サブスクリプションで Cloud Shell を初めて起動する場合は、Azure ファイル共有を作成して Cloud Shell ファイルを永続化するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_NAME`」の値を、前のタスクでプロビジョニングしたリソースを含むリソース グループ名に設定します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM に接続する 8 個のマネージド ディスクの最初のセットを作成します。

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした 2 番目の Azure VM に接続する 8 個のマネージド ディスクの 2 番目のセットを作成します。

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Azure portal で、前のタスクでプロビジョニングした最初の Azure VM (**az12001a-vm0**) のブレードに移動します。

1. **[az12001a-vm0]** ブレードから、 **[az12001a-vm0 \| ディスク]** ブレードに移動します。

1. **[az12001a-vm0 \| ディスク]** ブレードで、 **[既存のディスクを接続する]** を選択して、次の設定を持つデータ ディスクを az12001a-vm0 に接続します。

   - LUN:**0**

   - ディスク名: **az12001a-vm0-DataDisk0**

   - リソース グループ: *このタスクで前に使用したリソース グループ名*

   - ホスト キャッシュ:**読み取り専用**

1. 前の手順を繰り返して、プレフィックス **az12001a-vm0-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。 ディスク名の最後の文字と一致する LUN 番号を割り当てます。 LUN **1** を使用するディスクのホスト キャッシュを **読み取り専用** に設定し、残りのすべてのディスクのホスト キャッシュを **なし** に設定します。

1. 変更を保存します。 

1. Azure portal で、前のタスクでプロビジョニングした 2 番目の Azure VM (**az12001a-vm1**) のブレードに移動します。

1. **[az12001a-vm1]** ブレードから、 **[az12001a-vm1 \| ディスク]** ブレードに移動します。

1. **[az12001a-vm1 \| ディスク]** ブレードから、次の設定を持つデータ ディスクを az12001a-vm1 に接続します。

   - LUN:**0**

   - ディスク名: **az12001a-vm1-DataDisk0**

   - リソース グループ: *このタスクで前に使用したリソース グループ名*

   - ホスト キャッシュ:**読み取り専用**

1. 前の手順を繰り返して、プレフィックス **az12001a-vm1-DataDisk** を使用して残りの 7 つのディスク (合計 8 つ) を接続します。 ディスク名の最後の文字と一致する LUN 番号を割り当てます。 LUN **1** を使用するディスクのホスト キャッシュを **読み取り専用** に設定し、残りのすべてのディスクのホスト キャッシュを **なし** に設定します。

1. 変更を保存します。 

> **Result**:この演習を完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure コンピューティング リソースのプロビジョニングを行うことができます。


## <a name="exercise-2-configure-operating-system-of-azure-vms-running-linux-to-support-a-highly-available-sap-hana-installation"></a>演習 2: 高可用性の SAP HANA のインストールをサポートするように、Linux を実行する Azure VM のオペレーティング システムを構成する

期間: 30 分

この演習では、SAP HANA のクラスタ化されたインストールに対応するように、SUSE Linux Enterprise Server を実行している Azure VM 上の OS とストレージを構成します。

### <a name="task-1-connect-to-azure-linux-vms"></a>タスク 1:Azure Linux VM に接続する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_NAME`」の値を、前の演習でプロビジョニングしたリソースを含むリソース グループ名に設定します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Cloud Shell ペインで、次のコマンドを実行して、前のタスクでデプロイした最初の Azure VM のパブリック IP アドレスを特定します。

   ```cli
   PIP=$(az network public-ip show --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-ip --query ipAddress --output tsv)
   ```

1. Cloud Shell ペインで次のコマンドを実行して前の手順で特定した IP アドレスに SSH セッションを確立します。

   ```cli
   ssh student@$PIP
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、「`yes`」 と入力し、**Enter** キーを押します。 

1. パスワードの入力を求めるメッセージが表示されたら、「`Pa55w.rd1234`」と入力し、**Enter** キーを押します。 

1. 「Cloud Shell」ツールバーの「**新しいセッションを開く**」アイコンをクリックして、別の Cloud Shell Bash セッションを開きます。

1. 新しく開いた Cloud Shell Bash セッションで、このタスクのすべての手順を繰り返して、IP アドレス **az12001a-vm0-ip** を介して **az12001a-vm1** Azure VM に接続します。


### <a name="task-2-configure-storage-of-azure-vms-running-linux"></a>タスク 2:Linux を実行している Azure VM のストレージを構成する

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次のコマンドを実行して特権を昇格します。 

   ```cli
   sudo su -
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次のコマンドを実行して、新しく接続されたデバイスとその LUN 番号の間のマッピングを特定します。
   
   ```cli
   lsscsi
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して (8 つのうち) 6 つのデータ ディスクの物理ボリュームを作成します。
   
   ```cli
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してボリューム グループを作成します。
   
   ```cli
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームを作成します。

   ```cli
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **注**:各ボリューム グループに 1 つの論理ボリュームを作成しています

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームをフォーマットします。

   ```cli
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **注**:SUSE Linux Enterprise Server 12 以降では、XFS メタデータの自動チェックサム、ファイル タイプ サポート、およびファイルごとのアクセス制御リスト数の制限を提供する、XFS ファイル システムの新しいディスク上フォーマット (v5) を使用するオプションがあります。 YaST を使用して XFS ファイル システムを作成すると、新しいフォーマットが自動的に適用されます。 互換性上の理由から古い形式で XFS ファイル システムを作成するには、`-m crc=1` オプションを指定しないで mkfs.xfs コマンドを使用します。 

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdi** ディスクをパーティション化します。

   ```cli
   fdisk /dev/sdi
   ```

1. プロンプトが表示されたら、順番に「`n`」、「`p`」、「`1`」 (毎回 **Enter** キーを押す) と入力し **Enter** キーを 2 回押します。次に「`w`」と入力して書き込みを完了します。

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して **/dev/sdj** ディスクをパーティション化します。

   ```cli
   fdisk /dev/sdj
   ```

1. プロンプトが表示されたら、順番に「`n`」、「`p`」、「`1`」 (毎回 **Enter** キーを押す) と入力し **Enter** キーを 2 回押します。次に「`w`」と入力して書き込みを完了します。

1. [Cloud Shell] ペインの az12001a-vm0 への SSH セッションで、新しく作成したパーティションを書式設定します (「`y`」と入力し、確認を求められたら **Enter** キーを押します)。

   ```cli
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してマウント ポイントの役割を果たすディレクトリを作成します。

   ```cli
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して論理ボリュームの ID を表示します。

   ```cli
   blkid
   ```

   > **注**: **/dev/sdi** ( **/hana/shared** に使用) と **dev/sdj** ( **/usr/sap** に使用) を含む、新しく作成したボリューム グループおよびパーティションに関連付けられた **UUID** 値を特定します。


1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行して vi エディターで **/etc/fstab** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /etc/fstab
   ```

1. エディターで、次のエントリを **/etc/fstab** に追加します (ここで、`\<UUID of /dev/vg\_hana\_data-hana\_data\>`、`\<UUID of /dev/vg\_hana\_log-hana\_log\>`、`\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`、`\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>`、`\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>` は、前の手順で特定した ID を表します)。

   ```cli
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. 変更を保存してエディターを閉じます。

1. Cloud Shell ペインの SSH セッション ～ az12001a-vm0 内で、次のコマンドを実行して新しいボリュームをマウントします。

   ```cli
   mount -a
   ```

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してマウントが行われたことを確認します。

   ```cli
   df -h
   ```

1. az12001a-vm1 への Cloud Shell Bash セッションに切り替え、このタスクのすべての手順を繰り返して、**az12001a-vm1** でストレージを構成します。


### <a name="task-3-enable-cross-node-password-less-ssh-access"></a>タスク 3:クロスノード パスワードレス SSH アクセスを有効にする

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

   ```cli
   ssh-keygen -tdsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. [Cloud Shell] ペインの az12001a-vm1 への SSH セッションで、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、az12001a-vm0 で生成したキーを貼り付けます。

1. 変更を保存してエディターを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm1 内で、次のコマンドを実行してパスフレーズ不要の SSH キーを生成します。

   ```cli
   ssh-keygen -tdsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. az12001a-vm0 への SSH セッションを含む [Cloud Shell] ペインに切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を作成します (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、az12001a-vm1 で生成したキーを貼り付けます。

1. 変更を保存してエディターを閉じます。

1. Cloud Shell ペインの az12001a-vm0 への SSH セッションで、次を実行してパスフレーズレス SSH キーを生成します。

   ```cli
   ssh-keygen -t rsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. **az12001a-vm1** への SSH セッションを含む [Cloud Shell] ペインに切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、新しい行から開始し、az12001a-vm0 で生成したキーを貼り付けます。

1. 変更を保存してエディターを閉じます。

1. Cloud Shell ペインの SSH セッション～az12001a-vm1 内で、次のコマンドを実行してパスフレーズ不要の SSH キーを生成します。

   ```cli
   ssh-keygen -t rsa
   ```

1. プロンプトが表示されたら、**Enter** を 3 回押してから、次を実行して公開キーを表示します。 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. キーの値をクリップボードにコピーします。

1. az12001a-vm0 への SSH セッションを含む [Cloud Shell] ペインに切り替え、次を実行して vi エディターでファイル **/root/.ssh/authorized\_keys** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. エディター ウィンドウで、新しい行から開始し、az12001a-vm1 で生成したキーを貼り付けます。

1. 変更を保存してエディターを閉じます。

1. [Cloud Shell] ペインの az12001a-vm0 への SSH セッションで、次を実行して vi エディターでファイル **/etc/ssh/sshd\_config** を開きます (他のエディターを自由に使用できます)。

   ```cli
   vi /etc/ssh/sshd_config
   ```

1. **/etc/ssh/sshd\_config** ファイルで、**PermitRootLogin** および **AuthorizedKeysFile** エントリを検索し、次のように構成します (必要に応じて先頭の **#** 文字を削除します)。

   ```cli
   PermitRootLogin yes
   AuthorizedKeysFile  /root/.ssh/authorized_keys
   ```

1. 変更を保存してエディターを閉じます。

1. [Cloud Shell] ウィンドウの az12001a-vm0 への SSH セッションで、次を実行して sshd デーモンを再起動します。

   ```cli
   systemctl restart sshd
   ```

1. az12001a-vm1 で前の 4 つの手順を繰り返します。

1. 構成が成功したことを確認するには、Cloud Shell ペインの SSH セッションで az12001a-vm0 に対して、次を実行して az12001a-vm0 から az12001a-vm1 への SSH セッションを **root** として確立します。 

   ```cli
   ssh root@az12001a-vm1
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、「`yes`」 と入力し、**Enter** キーを押します。 

1. パスワードの入力が求められないことを確認します。

1. 次のコマンドを実行して、az12001a-vm0 から az12001a-vm1 までの SSH セッションを閉じます。 

   ```cli
   exit
   ```

1. 次を 2 回実行して、az12001a-vm0 からサインアウトします。

   ```cli
   exit
   ```

1. 構成が成功したことを確認するには、 「Cloud Shell」 ペインの SSH セッションで az12001a-vm1 に対して、次を実行して az12001a-vm1 から az12001a-vm0 への SSH セッションを **root** として確立します。 

   ```cli
   ssh root@az12001a-vm0
   ```

1. 接続を続行するかどうかを確認するメッセージが表示されたら、「`yes`」 と入力し、**Enter** キーを押します。 

1. パスワードの入力が求められないことを確認します。

1. 次を実行して、az12001a-vm1 から az12001a-vm0 への SSH セッションを閉じます。 

   ```cli
   exit
   ```

1. 次を 2 回実行して、az12001a-vm1 からサインアウトします。

   ```cli
   exit
   ```

> **Result**:この演習を完了すると、可用性の高い SAP HANA インストールをサポートするように、Linux を実行している Azure VM のオペレーティング システムを構成することができます。


## <a name="exercise-3-provision-azure-network-resources-necessary-to-support-highly-available-sap-hana-deployments"></a>演習 3: 高可用性の SAP HANA のデプロイをサポートするために必要な Azure ネットワーク リソースをプロビジョニングする

期間: 30 分

この演習では、SAP HANA のクラスター化されたインストールに対応するために Azure Load Balancers を実装します。


### <a name="task-1-configure-azure-vms-to-facilitate-load-balancing-setup"></a>タスク 1:負荷分散のセットアップが容易に行えるよう Azure VM を構成します。

   > **注**:Stardard SKU の Azure Load Balancer のペアを設定するので、まず、負荷分散バックエンド プールとして機能する 2 つの Azure VM のネットワーク アダプターに関連付けられたパブリック IP アドレスを削除する必要があります。

1. Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1. **[az12001a-vm0]** ブレードから **[az12001a-vm0 \| ネットワーク]** ブレードに移動し、 **[az12001a-vm0 \| ネットワーク]** ブレードで、そのネットワーク アダプターに関連付けられているパブリック IP アドレス **az12001a-vm0-ip** を表すエントリを選択します。

1. **「az12001a-vm0-ip」** ブレードで、 **「切り離し」** を選択してパブリック IP アドレスをネットワーク インターフェイスから切り離し、次に **「削除」** を選択して削除します。

1. Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。

1. **[az12001a-vm1]** ブレードから **[az12001a-vm1 \| ネットワーク]** ブレードに移動し、 **[az12001a-vm1 \| ネットワーク]** ブレードで、そのネットワーク アダプターに関連付けられているパブリック IP アドレス **az12001a-vm1-ip** を表すエントリを選択します。

1. **[az12001a-vm1-ip]** ブレードで、 **[切り離し]** を選択してパブリック IP アドレスをネットワーク インターフェイスから切り離し、次に **[削除]** を選択して削除します。

1. Azure portal で、**az12001a-vm0** Azure VM のブレードに移動します。

1. **[az12001a-vm0]** ブレードから、 **[az12001a-vm0 \| ネットワーク]** ブレードに移動します。 

1. **[az12001a-vm0 \| ネットワーク]** ブレードで、az12001a-vm0 のネットワーク インターフェイスを表すエントリを選択します。 

1. az12001a-vm0 のネットワーク インターフェースのブレードから、その IP 構成ブレードに移動し、そこから **ipconfig1** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。

1. Azure portal で、**az12001a-vm1** Azure VM のブレードに移動します。

1. **[az12001a-vm1]** ブレードから、 **[az12001a-vm1 \| ネットワーク]** ブレードに移動します。 

1. **[az12001a-vm1 \| ネットワーク]** ブレードから、az12001a-vm1 のネットワーク インターフェイスに移動します。 

1. az12001a-vm1 のネットワーク インターフェイスのブレードから、その [IP 構成] ブレードに移動し、そこから **[ipconfig1]** ブレードを表示します。

1. **ipconfig1** ブレードで、プライベート IP アドレスの割り当てを **Static** に設定し、変更を保存します。


### <a name="task-2-create-and-configure-azure-load-balancers-handling-inbound-traffic"></a>タスク 2:受信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal で、Azure ポータル ページの上部にある **「リソース、サービス、およびドキュメントの検索」** テキスト ボックスを使用して、 **「ロード バランサー」** ブレードを検索して移動し、 **「ロード バランサー」** ブレードで **「+ 追加」** を選択します。

1. 「**ロード バランサ―の作成**」ブレードの「**基本**」タブで、次の設定を指定して **「Review + create」** を選択します (他の設定は既定値のままにします)。

   - サブスクリプション: *Azure サブスクリプションの名前。*

   - リソース グループ: *このラボで前に使用したリソース グループ名*

   - 名前: **az12001a-lb0**

   - リージョン: *このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン*

   - SKU:**Standard**
   
   - 型: **内部**

1. **[次へ: フロントエンド IP の構成]** をクリックします。 **[フロントエンド IP の構成]** タブで、 **[フロントエンド IP 構成の追加]** をクリックし、 **[追加]** をクリックします。
   - 名前: frontend1
   
   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット: **subnet-0**

   - IP アドレスの割り当て:**静的**

   - IP アドレス：**192.168.0.240**

   - 可用性ゾーン:**ゾーン冗長**

1. **[確認と作成]** を選択し、次に **[作成]** を選択します。

   > **注**:ロード バランサーがプロビジョニングされるまで待ちます。 これに要する時間は 1 分未満です。 

1. Azure portal で、新しくプロビジョニングされた **az12001a-lb0** という名前のロード バランサ―のプロパティを表示するブレードに移動します。 

1. **「az12001a lb0」** ブレードで、 **「バックエンド プール」** を選択し、 **「+ 追加」** を選択し、 **「バックエンド プールを追加」** で次の設定を指定します (他は既定の設定のままにします)。

   - 名前: **az12001a-lb0-bepool**

   - IP バージョン:**IPv4** を選択します。

   - 仮想マシン: **az12001a-vm0** IP 構成: **ipconfig1 (192.168.0.4)**

   - 仮想マシン: **az12001a-vm1** IP 構成: **ipconfig1 (192.168.0.5)**

1. **「az12001a lb0」** ブレードで、 **「正常性プローブ」** を選択して **「追加」** を選択し、 **「正常性プローブの追加」** ブレードで次の設定を指定します (他は既定値のままにします)。

   - 名前: **az12001a-lb0-hprobe**

   - プロトコル: **TCP**

   - ポート:**62500**

   - 間隔:**5** *秒*

   - 異常しきい値:**2** *個の連続エラー*

1. **「az12001a lb0」** ブレードで、 **「負荷分散規則」** を選択し、 **「+ 追加」** を選択し、 **「負荷分散規則を追加」** で、次の設定を指定します (他は既定のままにします)。

   - 名前: **az12001a-lb0-lbruleAll**

   - IP バージョン: **IPv4**

   - フロントエンド IP アドレス:**192.168.0.240 (LoadBalancerFrontEnd)**

   - HA Ports:**有効**

   - バックエンド プール: **az12001a-lb0-bepool (2 台の仮想マシン)**

   - 正常性プローブ: **az12001a-lb0-hprobe (TCP:62500)**

   - セッション永続化: **なし**

   - アイドル タイムアウト (分):**4**

   - TCP リセット **Disabled**

   - フローティング IP (ダイレクト サーバー リターン):**有効**

### <a name="task-3-create-and-configure-azure-load-balancers-handling-outbound-traffic"></a>タスク 3:送信トラフィックを処理する Azure Load Balancers を作成して構成する

1. Azure portal の Cloud Shell で Bash セッションを開始します。 

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_NAME`」の値を、このラボの最初の演習でプロビジョニングしたリソースを含むリソース グループ名に設定します。

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer で使用するパブリック IP アドレスを作成します。

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer を作成します。

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. Cloud Shell ペインで次のコマンドを実行して、2 番目の Load Balancer のアウトバウンド規則を作成します。

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. [Cloud Shell] ペインを閉じます。

1. Azure portal で、新しくデプロイされた Azure Load Balancer **az12001a-lb** のプロパティを表示するブレードに移動します。

1. **az12001a-lb1** ブレードで、「**バックエンド プール**」をクリックします。

1. **[az12001a-lb1 \| バックエンド プール]** ブレードで、 **[az12001a-lb1-bepool]** をクリックします。

1. **az12001a-lb1-bepool** ブレードで、次の設定を指定し、「**保存**」をクリックします。

   - 仮想ネットワーク: **az12001a-rg-vnet (2 VM)**

   - 仮想マシン: **az12001a-vm0** IP 構成: **ipconfig1 (192.168.0.4)**

   - 仮想マシン: **az12001a-vm1**  IP 構成: **ipconfig1 (192.168.0.5)**

### <a name="task-4-deploy-a-jump-host"></a>タスク 4:ジャンプ ホストをデプロイする

   > **注**:2 つのクラスター化された Azure VM にはインターネットから直接アクセスできないため、ジャンプ ホストとして機能する Windows Server 2019 Datacenter を実行する Azure VM をデプロイします。 

1. ラボ コンピューターから Azure portal で、Azure portal ページの上部にある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスを使用して **[仮想マシン]** ブレードを検索して移動し、 **[仮想マシン]** ブレードで **[+ 作成]** を選択し、ドロップダウン メニューで **[Azure 仮想マシン]** を選択します。

1. **[仮想マシンの作成]** ブレードの **[基本]** タブで、次の設定を指定して **[次へ:ディスク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - サブスクリプション: *Azure サブスクリプションの名前。*

   - リソース グループ: *このラボで前に使用したリソース グループ名*

   - 仮想マシン名: **az12001a-vm2**

   - リージョン: *このラボの最初の演習で Azure VM をデプロイしたのと同じ Azure リージョン*

   - 可用性オプション:**インフラストラクチャ冗長は必要ありません**

   - イメージ:**Windows Server 2019 Datacenter - Gen 2**

   - サイズ:**Standard DS1 v2** _ または同様のもの_

   - Azure Spot インスタンス:"**いいえ**"

   - ユーザー名: **Student**

   - パスワード: **Pa55w.rd1234**

   - パブリック インバウンド ポート: **[選択したポートを許可する]**

   - [Selected inbound ports](選択された受信ポート): **RDP (3389)**

   - 既存の Windows サーバー ライセンスを使用しますか?"**いいえ**"

1. **[仮想マシンの作成]** ブレードの **[ディスク]** タブで、次の設定を指定して **[次へ:ネットワーク >]** を選択します (その他の設定はすべて既定値のままにします)。

   - OS ディスク タイプ:**Standard HDD**

   - 暗号化の種類: **(既定) プラットフォーム マネージド キーを使用した保存時の暗号化**

1. **[仮想マシンの作成]** ブレードの **[ネットワーク]** タブで、次の設定を指定して **[次へ:管理 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 仮想ネットワーク: **az12001a-RG-vnet**

   - サブネット名: **subnet-0 (192.168.0.0/24)**

   - パブリック IP アドレス: **az12001a-vm2-ip** "*という名前の新規 IP アドレス*"

   - NIC ネットワーク セキュリティ グループ:**Basic**

   - パブリック インバウンド ポート: **[選択したポートを許可する]**

   - 受信ポートを選択 **RDP (3389)**

   - 高速ネットワーキング:"**オフ**"

   - この仮想マシンを既存の負荷分散ソリューションの背後に配置します。"**いいえ**"

1. **[仮想マシンの作成]** ブレードの **[管理]** タブで、次の設定を指定して **[次へ:詳細 >]** を選択します (その他の設定はすべて既定値のままにします)。

   - 基本プランを有効にする (無料):"**いいえ**"

   > **注**:この設定は、Azure Security Center プランを既に選択している場合は使用できません。

   - ブート診断:**マネージド ストレージ アカウントで有効にする (推奨)**

   - OS ゲスト診断:"**オフ**"

   - システム割り当てマネージド ID:"**オフ**"

   - 自動シャットダウンを有効にする:"**オフ**"

   - バックアップを有効にする:"**オフ**"

   - ゲスト OS の更新:**手動更新**

1. 「**仮想マシンの作成**」ブレードの「**詳細**」タブで、 **「Review + create」** を選択します (他の設定はすべて既定値のままにします)。

1. 「**仮想ネットワークの作成**」ブレードの「**Review + create**」タブで、「**作成**」を選択します。

   > **注**:プロビジョニングが完了するまで待ちます。 これは 3 分もかかりません。

1. RDP 経由で新しくプロビジョニングされた Azure VM に接続します。 

1. az12001a-vm2 への RDP セッション内で、[ **https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html** ](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) から PuTTY をダウンロードします。

1. プライベート IP アドレス (それぞれ 192.168.0.4 および 192.168.0.5) を介して az12001a-vm0 と az12001a-vm1 の両方に SSH セッションを確立できることを確認します。 

> **Result**:この演習を完了すると、可用性の高い SAP HANA のデプロイをサポートするために必要となる Azure ネットワーク リソースのプロビジョニングを行うことができます。


## <a name="exercise-4-remove-lab-resources"></a>演習 4: ラボ リソースを削除する

期間: 10 分

この演習では、このラボでプロビジョニングしたリソースすべてを削除します。

#### <a name="task-1-open-cloud-shell"></a>タスク 1:Cloud Shell を開く

1. ポータルの上部にある「**Cloud Shell**」アイコンをクリックして Cloud Shell ペインを開き、シェルとして Bash を選択します。

1. [Cloud Shell] ペインで次のコマンドを実行して、変数「`RESOURCE_GROUP_PREFIX`」の値を、このラボでプロビジョニングしたリソースを含むリソース グループ名のプレフィックスに設定します。

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
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
