---
lab:
  title: 06a - Azure Center for SAP solutions (ACSS) のデプロイのための前提条件の概要
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# AZ 1006 モジュール: Azure で SAP ワークロードをサポートするためのインフラストラクチャを設計して実装する
# AZ-1006 1 日コース ラボ: Azure Center for SAP solutions (ACSS) を使用した SAP ワークロードのデプロイのための前提条件の概要

予測される所要時間:100 分

AZ-1006 1 日コース ラボのすべてのタスクは Azure portal から実行します

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Center for SAP solutions を使用して、Azure に SAP ワークロードをデプロイするための前提条件を実装する

## 手順

### 演習 1: Azure Center for SAP solutions を使用して、Azure に SAP ワークロードをデプロイするための前提条件を実装する

期間:60 分

この演習では、Azure Center for SAP solutions を使用して、Azure に SAP ワークロードをデプロイするための前提条件を確認して実装します。 これには次のアクティビティが含まれます。

- Azure Storage アクセスのために Azure Center for SAP solutions によってデプロイ時に使用される Microsoft Entra ユーザー割り当てマネージド ID を作成する。
- デプロイに含まれるすべての Azure Virtual Machines をホストする Azure 仮想ネットワークを作成する。
- インターネットから Azure VM への接続をセキュリティで保護するための Azure Bastion リソースを作成する。
- デプロイ用に使用される Azure Center for SAP solutions に関連付けられている Azure Storage 汎用 v2 アカウントを作成する
- Azure サブスクリプションと Azure Storage 汎用 v2 アカウントへのデプロイ アクセスの実行に使用される Microsoft Entra ユーザー割り当てマネージド ID を付与する
- SAP トランスポート ディレクトリの実装に使用される Azure Premium ファイル共有アカウントを作成する
- デプロイをホストする仮想ネットワークのサブネットからの送信アクセスを制限するために使用される、ネットワーク セキュリティ グループ (NSG) を作成して構成する。
- Azure Center for SAP solutions デプロイの一部としての SAP ソフトウェアのインストールに使用する Azure Virtual Machine (VM) を作成する。
- Azure Bastion を使用して Azure VM に接続し、SAP ソフトウェアのインストール用に構成する。
- ラボでプロビジョニングした Azure リソースをすべて削除する。

これらのアクティビティは、この演習の以下のタスクに対応します。

- タスク 1:Microsoft Entra ユーザー割り当てマネージド ID を作成する
- タスク 2:Azure 仮想ネットワークを作成する
- タスク 3:Azure Bastion リソースを作成する
- タスク 4:Azure Storage 汎用 v2 アカウントを作成する
- タスク 5:Microsoft Entra ユーザー割り当てマネージド ID の認可を構成する
- タスク 6:Azure Premium ファイル共有アカウントを作成する
- タスク 7:ネットワーク セキュリティ グループを作成して構成する
- タスク 8:Azure 仮想マシンを作成します。
- タスク 9:Azure Virtual Machine を構成する
- タスク 10:Azure リソースを削除する

#### タスク 1:Microsoft Entra ユーザー割り当てマネージド ID を作成する

このタスクでは、Azure Storage アクセスのために Azure Center for SAP solutions によってデプロイ時に使用される Microsoft Entra ユーザー割り当てマネージド ID を作成します。

1. ラボ コンピューターから Web ブラウザーを起動し、`https://portal.azure.com` の Azure portal に移動して、このラボで使用する Azure サブスクリプションの所有者のロールを持つ Microsoft アカウントまたは Microsoft Entra ID アカウントを使用して認証します。
1. Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[マネージド ID]** を検索して選択します。
1. **[マネージド ID]** ページで、 **[+ 作成]** を選択します。
1. **[ユーザー割り当てマネージド ID]** ページの **[基本]** タブで、以下の設定を指定した後、 **[確認と作成]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
`   |リソース グループ| 新しい **acss-infra-RG** を選択または作成する|
   |リージョン|ACSS デプロイ用に使用する Azure リージョンの名前|
   |名前|**acss-infra-MI**|

1. **[確認]** タブで、検証プロセスが完了するのを待って、 **[作成]** を選択します。

   >**注**:プロビジョニング プロセスが完了するのを待たずに、次のタスクに進んでください。 プロビジョニングにかかるのは、わずか数秒です。

   >**注**:この後のタスクの 1 つでは、Azure Center for SAP solutions を使用した SAP ソフトウェアのインストールに対応するために、SAP インストール メディアをホストするストレージ アカウントへのマネージド ID のアクセスを承認します。

#### タスク 2:仮想ネットワークを作成する

このタスクでは、デプロイに含まれるすべての Azure Virtual Machines をホストする Azure 仮想ネットワークを作成します。 さらに、仮想ネットワーク内に次のサブネットを作成します。

- AzureFirewallSubnet - Azure Firewall のデプロイを目的としています
- AzureBastionSubnet - Azure Bastion のデプロイを目的としています
- dmz - SAP ソフトウェアのデプロイに使用される Azure VM のデプロイを目的としています
- app - SAP アプリケーションと SAP セントラル サービス インスタンス サーバーをホストすることを目的としています
- db - SAP データベース層をホストすることを目的としています

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[仮想ネットワーク]** を検索して選択します。 
1. **[仮想ネットワーク]** ページで、**[+ 作成]** を選択します。
1. **[仮想ネットワークの作成]** ページの **[基本]** タブで、以下の設定を指定し、 **[次へ]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |仮想ネットワーク名|**acss-infra-VNET**|
   |リージョン|この演習の前のタスクで使用したのと同じ Azure リージョンの名前|

1. **[セキュリティ]** タブで、既定の設定をそのまま使用し、 **[次へ]** を選択します。

   >**注**: この時点で Azure Bastion と Azure Firewall の両方をプロビジョニングできますが、仮想ネットワークが作成されてから、それらを個別にプロビジョニングします。

1. **[IP アドレス]** タブで、以下のサブネット設定を指定した後、**[確認 + 作成]** を選択します。

   |設定|値|
   |---|---|
   |IP アドレス空間|**10.0.0.0/16 (65,536 個のアドレス)**|

1. サブネットの一覧でごみ箱アイコンを選択して、**既定**のサブネットを削除します。
1. **+ サブネットの追加** を選択します。
1. **[サブネットの追加]** ペインで、以下の設定を指定してから、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|Value|
   |---|---|
   |サブネットの目的|**Azure Firewall**|
   |開始アドレス|**10.0.0.0**|

   >**注**:これにより、サブネットに **AzureFirewallSubnet** という名前が自動的に割り当てられ、そのサイズが **/26 (64 個のアドレス)** に設定されます。

1. **+ サブネットの追加** を選択します。
1. **[サブネットの追加]** ペインで、以下の設定を指定してから、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|値|
   |---|---|
   |名前|**dmz**|
   |開始アドレス|**10.0.0.128**|
   |サイズ|**/26 (64 個のアドレス)**|

   >**注**:このサブネットは、Azure Center for SAP solutions を使用した SAP ソフトウェアのインストールに使う Azure VM をホストするために使用されます。

1. **+ サブネットの追加** を選択します。
1. **[サブネットの追加]** ペインで、以下の設定を指定してから、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|Value|
   |---|---|
   |サブネットの目的|**Azure Bastion**|
   |開始アドレス|**10.0.1.0**|
   |サイズ|**/24 (256 アドレス)**|

   >**注**:これにより、サブネットに **AzureBastionSubnet** という名前が自動的に割り当てられます。

1. **+ サブネットの追加** を選択します。
1. **[サブネットの追加]** ペインで、以下の設定を指定してから、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|値|
   |---|---|
   |名前|**app**|
   |開始アドレス|**10.0.2.0**|
   |サイズ|**/24 (256 アドレス)**|

1. **+ サブネットの追加** を選択します。
1. **[サブネットの追加]** ペインで、以下の設定を指定してから、**[追加]** を選択します (他の設定は既定値のまま)。

   |設定|値|
   |---|---|
   |名前|**db**|
   |開始アドレス|**10.0.3.0**|
   |サイズ|**/24 (256 アドレス)**|

1. **[IP アドレス]** タブで、**[確認および作成]** を選択します。
1. **[確認および作成]** タブで、検証プロセスが完了するのを待ってから、**[作成]** を選択します。

   >**注**:プロビジョニング プロセスが完了するのを待たずに、次のタスクに進んでください。 プロビジョニングにかかるのは、わずか数秒です。

#### タスク 3:Azure Bastion リソースを作成する

このタスクでは、インターネットから Azure VM への接続をセキュリティで保護するための Azure Bastion リソースを作成します。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[Bastion]** を検索して選択します。 
1. **[Bastion]** ページで、 **[+ 作成]** を選択します。
1. **[Bastion]** ページの **[基本]** タブで、以下の設定を指定し、**[次へ: 詳細 >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |名前|**acss-infra-BASTION**|
   |リージョン|この演習で前に使用したのと同じ Azure リージョンの名前|
   |レベル|**Basic**|
   |インスタンス数|**2**|
   |仮想ネットワーク|**acss-infra-VNET**|
   |Subnet|**AzureBastionSubnet (10.0.1.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |パブリック IP アドレス名|**acss-bastion-PIP**|

1. **[詳細]** タブで、変更を加えずに利用可能な設定を確認した後、**[確認および作成]** を選択します。
1. **[確認および作成]** タブで、検証プロセスが完了するのを待ってから、**[作成]** を選択します。

   >**注**:プロビジョニングが完了するのを待たずに、次のタスクに進んでください。 プロビジョニングは 5 分ほどかかる場合があります。

#### タスク 4:Azure Storage 汎用 v2 アカウントを作成する

このタスクでは、デプロイ用に使用される Azure Center for SAP solutions に関連付けられている Azure Storage 汎用 v2 アカウントを作成します。 このストレージ アカウントは、Azure Center for SAP solutions を使用した SAP ソフトウェアのインストールに対応するために、SAP インストール メディアをホストするために使用されます。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[ストレージ アカウント]** を検索して選択します。
1. **[ストレージ アカウント]** ページで **[+ 作成]** を選択します。
1. **[ストレージ アカウントの作成]** ページの **[基本]** タブで、以下の設定を指定し、 **[次へ: 詳細 >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |ストレージ アカウント名|文字と数字で構成される、長さが 3 から 24 のグローバルに一意の名前|
   |リージョン|この演習で前に使用したのと同じ Azure リージョンの名前|
   |パフォーマンス|**Standard**|
   |冗長性|**geo 冗長ストレージ (GRS)**|
   |リージョン可用性のイベントでデータへの読み取りアクセスを利用できるようにする|無効|

1. **[詳細]** タブで、利用可能なオプションを確認し、既定値をそのまま使用し **[次へ: ネットワーク >]** を選択します。
1. **[ネットワーク]** タブで、次のアクションを実行してから、**[確認]** を選択します。

   1. **[選択した仮想ネットワークと IP アドレスからのパブリック アクセスを有効にする]** を選択します。
   1. **[仮想ネットワーク]** セクションで、このラボで使用する Azure サブスクリプションの名前が **[仮想ネットワーク サブスクリプション]** ドロップダウン リストに表示されていることを確認します。
   1. **[仮想ネットワーク]** セクションの **[仮想ネットワーク]** ドロップダウン リストで、**[acss-infra-VNET]** を選択します。
   1. **[サブネット]** ドロップダウン リストで、**app**、**db**、**dmz** の各サブネットを選択します。

1. **[確認]** タブで、検証プロセスが完了するのを待って、 **[作成]** を選択します。

   >**注**:プロビジョニング プロセスが完了するまで待ちます。 プロビジョニングには 1 分かからないはずです。

1. **[デプロイが完了しました]** ページで、**[リソースに移動]** を選択します。
1. ストレージ アカウント ページの左側にある縦型ナビゲーション メニューの **[データ ストレージ]** セクションで、**[コンテナー]** を選択します。
1. **[+ コンテナー]** を選択します。
1. **[新しいコンテナー]** ペインの **[名前]** テキスト ボックスに「**sapbits**」と入力し、**[作成]** を選択します。

   >**注**:**sapbits** コンテナーは、SAP インストール メディアをホストします。

#### タスク 5:Microsoft Entra ユーザー割り当てマネージド ID の認可を構成する

このタスクでは、Azure ロールベースのアクセス制御 (RBAC) のロールの割り当てを使用して、Microsoft Entra ユーザー割り当てマネージド ID を付与します。 マネージド ID は、Azure サブスクリプションと、前のタスクで作成した Azure Storage 汎用 v2 アカウントへのデプロイ アクセスを実行するために使用されます。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[マネージド ID]** を検索して選択します。
1. [マネージド ID] ページで、**[acss-infra-MI]** エントリを選択します。
1. **[acss-infra-MI]** ページの左側にある縦型ナビゲーション メニューで、**[Azure ロールの割り当て]** を選択します。
1. **[Azure ロール割り当て]** ページで、 **[+ ロール割り当ての追加 (プレビュー)]** を選択します。
1. **[+ ロール割り当ての追加 (プレビュー)]** ペインで、以下の設定を指定し、 **[保存]** を選択します。

   |設定|値|
   |---|---|
   |Scope|**サブスクリプション**|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |ロール|**Azure Center for SAP solutions のサービス ロール**|

1. **[Azure ロール割り当て]** ページに戻り、 **[+ ロール割り当ての追加 (プレビュー)]** を選択します。
1. **[+ ロール割り当ての追加 (プレビュー)]** ペインで、以下の設定を指定し、 **[保存]** を選択します。

   |設定|値|
   |---|---|
   |Scope|**Storage**|
   |サブスクリプション|このラボで使用する Azure サブスクリプションの名前|
   |リソース|前のタスクで作成した Azure Storage アカウントの名前|
   |役割|**Reader and Data Access**|

#### タスク 6:Azure Premium ファイル共有アカウントを作成する

このタスクでは、SAP トランスポート ディレクトリの実装に使用される Azure Premium ファイル共有アカウントを作成します。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[ストレージ アカウント]** を検索して選択します。
1. **[ストレージ アカウント]** ページで **[+ 作成]** を選択します。
1. **[ストレージ アカウントの作成]** ページの **[基本]** タブで、以下の設定を指定し、 **[次へ: 詳細 >]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使う Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |ストレージ アカウント名|文字と数字で構成される、長さが 3 から 24 のグローバルに一意の名前|
   |リージョン|この演習で前に使用したのと同じ Azure リージョンの名前|
   |パフォーマンス|**Premium**|
   |Premium アカウントの種類|**ファイル共有**|
   |冗長性|**ゾーン冗長ストレージ (ZRS)**|

1. **[詳細設定]** タブで、**[REST API 操作の安全な転送を必須にする]** 設定を無効にして、**[次へ: ネットワーク >]** を選択します。

   >**注**:NFS プロトコルは暗号化をサポートせず、代わりにネットワーク レベルのセキュリティに依存します。 NFS が機能するためには、この設定を無効にする必要があります。

1. **[ネットワーク]** タブで、次のアクションを実行してから、**[確認]** を選択します。

   1. **[選択した仮想ネットワークと IP アドレスからのパブリック アクセスを有効にする]** を選択します。
   1. **[仮想ネットワーク]** セクションで、このラボで使用する Azure サブスクリプションの名前が **[仮想ネットワーク サブスクリプション]** ドロップダウン リストに表示されていることを確認します。
   1. **[仮想ネットワーク]** セクションの **[仮想ネットワーク]** ドロップダウン リストで、**[acss-infra-VNET]** を選択します。
   1. **[サブネット]** ドロップダウン リストで、**app**、**db**、**dmz** の各サブネットを選択します。

   >**注**:一般に、内部リソースへのアクセスを境界サブネットから許可しないようにします。 この場合、このようにする唯一の理由は、このラボの後半でこのアクセスを検証できるようにすることです。

1. **[確認]** タブで、検証プロセスが完了するのを待って、 **[作成]** を選択します。

   >**注**:プロビジョニング プロセスが完了するまで待ちます。 プロビジョニングには 1 分かからないはずです。

1. **[デプロイが完了しました]** ページで、**[リソースに移動]** を選択します。
1. ストレージ アカウント ページの左側にある縦型メニューの **[データ ストレージ]** セクションで、**[ファイル共有]** を選択し、**[+ ファイル共有]** を選択します。
1. **[新しいファイル共有]** ページの **[基本]** タブで、以下の設定を指定した後、**[確認および作成]** を選択します。

   |設定|値|
   |---|---|
   |名前|**trans**|
   |プロビジョニング容量|**128**|
   |プロトコル|**NFS**|
   |ルート スカッシュ|*ルート スカッシュなし**|

1. **[確認および作成]** タブで、検証プロセスが完了するのを待ってから、**[作成]** を選択します。

   >**注**:ファイル共有のプロビジョニングが完了するまで待ちます。 プロビジョニングにかかるのは、わずか数秒です。

1. **[この NFS 共有への Linux からの接続]** ページの **[Linux ディストリビューションの選択]** ドロップダウン リストで、**[SUSE]** を選択し、この NFS 共有をマウントするサンプル コマンドを確認します。

#### タスク 7:ネットワーク セキュリティ グループを作成して構成する

このタスクでは、デプロイをホストする仮想ネットワークのサブネットからの送信アクセスを制限するために使用されるネットワーク セキュリティ グループ (NSG) を作成して構成します。 これを実現するには、インターネットへの接続をブロックしますが、以下のサービスへの接続を明示的に許可します。

- SUSE または Red Hat 更新インフラストラクチャ エンドポイント
- Azure Storage
- Azure Key Vault
- Microsoft Entra ID
- Azure Resource Manager

>**注**:一般に、NSG ではなく Azure Firewall を使用して SAP デプロイのネットワーク接続をセキュリティで保護することを検討する必要があります。 このラボでは、両方のオプションについて説明します。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[ネットワーク セキュリティ グループ]** を検索して選択します。
1. [**ネットワーク セキュリティ グループ**] ページで、 **[+ 作成]** を選択します。
1. **[ネットワーク セキュリティ グループの作成]** ページの **[基本]** タブで、以下の設定を指定した後、 **[確認と作成]** を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使う Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |名前|**acss-infra-NSG**|
   |リージョン|この演習で前に使用したのと同じ Azure リージョンの名前|

1. **[確認と作成]** タブで、検証プロセスが完了するのを待って、 **[作成]** を選択します。

   >**注**:プロビジョニング プロセスが完了するまで待ちます。 プロビジョニングには 1 分かからないはずです。

1. **[デプロイが完了しました]** ページで、**[リソースに移動]** を選択します。

   >**注**: 既定では、ネットワーク セキュリティ グループの組み込みルールは、すべての送信トラフィック、同じ仮想ネットワーク内のすべてのトラフィック、およびピアリングされた仮想ネットワーク間のすべてのトラフィックを許可します。 セキュリティの観点から、この既定の動作を制限することを検討してください。 提示された構成では、インターネットと Azure への送信接続が制限されます。 NSG 規則を使用して、仮想ネットワーク内の接続を制限することもできます。

1. **[acss-infra-NSG]** ページの左側にある縦型ナビゲーション メニューの **[設定]** セクションで、**[送信セキュリティ規則]** を選択します。
1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   >**注**:Red Hat 更新インフラストラクチャ エンドポイントへの接続を明示的に許可するには、次の規則を追加する必要があります。

   >**注**: RHEL に使用する IP アドレスを特定するには、「[インフラストラクチャ デプロイのためのネットワークの準備](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)」を参照してください

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |宛先|**IP アドレス**|
   |宛先 IP アドレス/CIDR 範囲|**13.91.47.76、40.85.190.91、52.187.75.218、52.174.163.213、52.237.203.198**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**300**|
   |名前|**AllowAnyRHELOutbound**|
   |説明|**RHEL 更新インフラストラクチャ エンドポイントへの送信接続を許可する**|

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   >**注**:SUSE 更新インフラストラクチャ エンドポイントへの接続を明示的に許可するには、次の規則を追加する必要があります。

   >**注**: SUSE に使用する IP アドレスを識別するには、「[インフラストラクチャ デプロイのためのネットワークの準備](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)」を参照してください

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |宛先|**IP アドレス**|
   |宛先 IP アドレス/CIDR 範囲|**52.188.224.179、52.186.168.210、52.188.81.163、40.121.202.140**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**305**|
   |名前|**AllowAnySUSEOutbound**|
   |説明|**SUSE 更新インフラストラクチャ エンドポイントへの送信接続を許可する**|

   >**注**:Azure Storage への接続を明示的に許可するには、次の規則を追加する必要があります。

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |到着地|**サービス タグ**|
   |宛先サービス タグ|**Storage**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**400**|
   |名前|**AllowAnyCustomStorageOutbound**|
   |説明|**Azure Storage への送信接続を許可する**|

   >**注**:**Storage** サービス タグは、**Storage.EastUS** などのリージョン固有のものに置き換えることができます。

   >**注**:Azure Key Vault への接続を明示的に許可するには、次の規則を追加する必要があります。

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |到着地|**サービス タグ**|
   |宛先サービス タグ|**AzureKeyVault**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**500**|
   |名前|**AllowAnyCustomKeyVaultOutbound**|
   |説明|**Azure Key Vault への送信接続を許可する**|

   >**注**:Microsoft Entra ID への接続を明示的に許可するには、次の規則を追加する必要があります。

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |到着地|**サービス タグ**|
   |宛先サービス タグ|**AzureActiveDirectory**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**600**|
   |名前|**AllowAnyCustomEntraIDOutbound**|
   |説明|**Microsoft Entra ID への送信接続を許可する**|

   >**注**:Azure Resource Manager への接続を明示的に許可するには、次の規則を追加する必要があります。

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |到着地|**サービス タグ**|
   |宛先サービス タグ|**AzureResourceManager**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**許可**|
   |Priority|**700**|
   |名前|**AllowAnyCustomARMOutbound**|
   |説明|**Azure Resource Manager への送信接続を許可する**|

   >**注**:インターネットへの接続を明示的にブロックするには、最後の規則を追加する必要があります。

1. **[acss-infra-NSG \| 送信セキュリティ規則]** ページで、**[+ 追加]** を選択します。
1. **[送信セキュリティ規則の追加]** ペインで、以下の設定を指定し、**[追加]** を選択します。

   |設定|値|
   |---|---|
   |source|**[任意]**|
   |ソース ポート範囲|*|
   |到着地|**サービス タグ**|
   |宛先サービス タグ|**Internet**|
   |サービス|**カスタム**|
   |宛先ポート範囲|*|
   |プロトコル|**[任意]**|
   |アクション|**Deny**|
   |Priority|**1000**|
   |名前|**DenyAnyCustomInternetOutbound**|
   |説明|**インターネットへの送信接続を拒否する**|

   >**注**:最後に、SAP デプロイをホストする仮想ネットワークの関連サブネットに NSG を割り当てる必要があります。

1. **[送信セキュリティ規則の追加]** ペインの左側にある縦型ナビゲーション メニューの **[設定]** セクションで、**[サブネット]** を選択します。
1. **[acss-infra-NSG \| サブネット]** ページで、**[+ 関連付け]** を選択します。
1. **[サブネットの関連付け]** ペインの **[仮想ネットワーク]** ドロップダウン リストで、**[acss-intra-VNET (acss-infra-RG)]** を選択し、**[サブネット]** ドロップダウン リストで **[app]** を選択して、**[OK]** を選択します。
1. **[サブネットの関連付け]** ペインの **[仮想ネットワーク]** ドロップダウン リストで、**[acss-intra-VNET (acss-infra-RG)]** を選択し、**[サブネット]** ドロップダウン リストで **[db]** を選択した後、**[OK]** を選択します。

#### タスク 8:Azure 仮想マシンを作成します。

このタスクでは、Azure Center for SAP solutions デプロイの一部としての SAP ソフトウェアのインストールに使用する Azure Virtual Machine (VM) を作成します。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[仮想マシン]** を検索して選択します。
1. **[仮想マシン]** ページで、**[+ 作成]** を選択し、ドロップダウン メニューの **[Azure Virtual Machine]** を選択します。
1. **[仮想マシンの作成]** ページの **[基本]** タブで、以下の設定を指定して **[次へ: ディスク >]** を選択します (その他の設定はすべて既定値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使う Azure サブスクリプションの名前|
   |リソース グループ|**acss-infra-RG**|
   |仮想マシン名|**acss-infra-vm0**|
   |リージョン|この演習で前に使用したのと同じ Azure リージョンの名前|
   |可用性オプション|**インフラストラクチャ冗長は必要ありません**|
   |セキュリティの種類|**トラステッド起動の仮想マシン**|
   |Image|**Ubuntu Server 20.04 LTS - x64 Gen2**|
   |VMアーキテクチャ|**x64**|
   |Azure Spot 割引で実行します|disabled|
   |サイズ|**Standard_B2ms**|
   |認証の種類|**パスワード**|
   |ユーザー名|任意の有効なユーザー名|
   |Password|任意の複雑なパスワード|
   |パブリック受信ポート|**なし**|
   
    > **注**:指定したユーザー名とパスワードを覚えておいてください。 このラボで後ほど必要になります。

1. **[ディスク]** タブで、既定値をそのまま使用し、**[次へ: ネットワーク >]** を選択します。
1. **[ネットワーク]** タブで、以下の設定を指定し、**[次へ: 管理 >]** を選択します (その他の設定はすべて既定値のままにします)。

   |設定|値 |
   |---|---|
   |仮想ネットワーク|**acss-infra-VNET**|
   |Subnet|**dmz**|
   |パブリック IP アドレス|**なし**|
   |NIC ネットワーク セキュリティ グループ|**なし**|
   |VM が削除された場合に NIC を削除する|有効|
   |負荷分散オプション|**なし**|

1. **[管理]** タブで、すべての設定を既定値のままにして、**[次へ: 監視 >]** を選択します
1. **[監視]** タブで、**[ブート診断]** を **[無効]** に 設定し、**[次へ： 詳細 >]** を選択します (その他の設定はすべて既定値のままにします)
1. **[詳細]** タブで、**[確認および作成]** を選択します (すべての設定を既定値のままにします)。
1. **[仮想マシンの作成]** メニューの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注**:プロビジョニングが完了するまで待ちます。 プロビジョニングには 3 分ほどかかる場合があります。

#### タスク 9:Azure VM を構成する

このタスクでは、Azure Bastion を使用して Azure VM に接続し、SAP ソフトウェアのインストール用に構成します。 

> **注**:このタスクを開始する前に、Azure Bastion のプロビジョニングが完了していることを確認します。

> **注**:Web ブラウザーでポップアップ ウィンドウがブロックされていないことを確認し、ブロックされている場合は、ポップアップ ブロック機能を無効にします。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウの **[検索]** テキスト ボックスで、**[仮想マシン]** を検索して選択します。 
1. **[仮想マシン]** ページで、**[acss-infra-vm0]** エントリを選択します。
1. **[acss-infra-vm0]** ページのツール バーで **[接続]** を選択し、ドロップダウン メニューで **[Connect via Bastion]\(Bastion 経由で接続\)** を選択します。
1. **[acss-infra-vm0 \| Bastion]** ページで、**[認証の種類]** が **[VM パスワード]** に設定されていることを確認し、**[ユーザー名]** と **[パスワード]** のテキスト ボックスに、Azure VM のプロビジョニング時に設定したユーザー名とパスワードを入力し、**[新しいブラウザー タブで開く]** チェック ボックスがオンになっていることを確認して、**[接続]** を選択します。

   > **注**:これにより、別の Web ブラウザー ウィンドウ タブが開き、Azure VM で実行されているシェル セッションが表示されます。

   > **注**:SAP インストール メディアのアップロード用に Ubuntu サーバーを準備するには、Azure CLI をインストールします。

1. 新しく開いたブラウザー タブのシェル セッション内で、次のコマンドを実行して Azure CLI をインストールします。

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

1. シェル セッション内で、次のコマンドを実行して PIP3 をインストールします。

   ```bash
   sudo apt install python3-pip
   ```

1. シェル セッション内で、次のコマンドを実行して Ansible 2.13.19 をインストールします。

   ```bash
   sudo pip3 install ansible-core==2.13.9
   ```

1. シェル セッション内で、次のコマンドを実行して Ansible Galaxy コレクション モジュールをインストールします。

   ```bash
   sudo ansible-galaxy collection install ansible.netcommon:==5.0.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.posix:==1.5.1 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.utils:==2.9.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.windows:==1.13.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install community.general:==6.4.0 -p /opt/ansible/collections
   ```

1. シェル セッション内で次のコマンドを実行して、GitHub から SAP オートメーション サンプル リポジトリをクローンします。

   ```bash
   git clone https://github.com/Azure/SAP-automation-samples.git
   ```

1. シェル セッション内で次のコマンドを実行して、GitHub から SAP オートメーション リポジトリをクローンします。

   ```bash
   git clone https://github.com/Azure/sap-automation.git
   ```

1. シェル セッション内で、次のコマンドを実行してセッションを終了します。

   ```bash
   logout
   ```

1. メッセージが表示されたら、**［閉じる］** を選択します。

#### タスク 10:Azure リソースを削除する

このタスクでは、このラボでプロビジョニングされたすべての Azure リソースを削除します。

1. ラボのコンピューターで、Azure portal が表示されている Web ブラウザー ウィンドウで **[Cloud Shell]** アイコンを選択して Cloud Shell ペインを開きます。 必要に応じて、**[Bash]** を選択して Bash シェル セッションを開始します。 

   > **注**:このラボで使用する Azure サブスクリプションで Cloud Shell を起動するのが初めての場合は、Cloud Shell ファイルを永続化するための Azure ファイル共有を作成するように求められます。 その場合は、既定値に設定すると、自動的に生成されたリソース グループ内にストレージ アカウントが作成されます。

1. Cloud Shell ペインで次のコマンドを実行して、リソース グループ **acss-infra-RG** とそのリソースすべてを削除します。

   ```cli
   az group delete --name 'acss-infra-RG' --no-wait --yes
   ```

   > **注**:コマンドは非同期的に実行されます (これは `--nowait` パラメータによって決定されます)。そのため、シェル プロンプトは呼び出した直後に表示されますが、リソース グループとそのリソースが実際に削除されるまでに数分かかります。

1. [Cloud Shell] ペインを閉じます。
