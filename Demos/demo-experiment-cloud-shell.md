# <a name="demonstration-experiment-with-the-cloud-shell"></a>デモンストレーション: Cloud Shell を試す

## <a name="configure-the-cloud-shell"></a>Cloud Shell を設定する

1. **Azure portal** にアクセスします。
2. 上部バナーの **Cloud Shell** アイコンをクリックします。
3. On the Welcome to the Shell page, notice your selections for Bash or PowerShell. Select <bpt id="p1">**</bpt>PowerShell<ept id="p1">**</ept>.
4. The Azure Cloud Shell requires an Azure file share to persist files. As you have time, click Learn more to obtain information about the Cloud Shell storage and the associated pricing.
5. **[サブスクリプション]** を選択し、**[ストレージの作成]** をクリックします。 

## <a name="experiment-with-azure-powershell"></a>Azure PowerShell を使用してみる

1. ストレージが作成され、アカウントが初期化されるのを待ちます。
2. PowerShell プロンプトで、**Get-AzSubscription** と入力してサブスクリプションを表示します。
3. **Get-AzResourceGroup** と入力して、リソース グループ情報を表示します。

## <a name="experiment-with-the-bash-shell"></a>Bash シェルを使用してみる

1. ドロップダウンを使用して **Bash** シェルに切り替え、選択を確認します。
2. At the Bash shell prompt, type <bpt id="p1">**</bpt>az account list<ept id="p1">**</ept> to view your subscriptions. Also, try tab completion. 
3. **az resource list** と入力して、リソース情報を表示します。

## <a name="experiment-with-the-cloud-editor"></a>Cloud Editor を使用してみる

1. To use the Cloud Editor, type <bpt id="p1">**</bpt>code .<ept id="p1">**</ept>. You can also select the curly braces icon. 
2. Select a file from the left navigation pane. For example, <bpt id="p1">**</bpt>.profile<ept id="p1">**</ept>.
3. エディターの上部バナーに、設定 (テキスト サイズとフォント) とファイルのアップロード/ダウンロードの選択肢があることに注意してください。
4. [保存]、[エディターを閉じる]、および [ファイルを開く] の右端にある省略記号 ( **...** リソース グループ:) に注意してください。
5. 時間が許す限り試し、その後 Cloud Editor を**閉じます**。 
6. Cloud Shell を閉じます。