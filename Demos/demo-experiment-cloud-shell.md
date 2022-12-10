# <a name="demonstration-experiment-with-the-cloud-shell"></a>デモンストレーション: Cloud Shell を試す

## <a name="configure-the-cloud-shell"></a>Cloud Shell を設定する

1. **Azure portal** にアクセスします。
2. 上部バナーの **Cloud Shell** アイコンをクリックします。
3. 「Shell へようこそ」ページで、Bash または PowerShell の選択内容を確認します。 **[PowerShell]** を選択します。
4. Azure Cloud Shell はファイルを保持するために Azure ファイル共有が必要です。 時間がある場合は、「詳細情報」をクリックして、Cloud Shell ストレージと関連する価格に関する情報を取得します。
5. **[サブスクリプション]** を選択し、**[ストレージの作成]** をクリックします。 

## <a name="experiment-with-azure-powershell"></a>Azure PowerShell を使用してみる

1. ストレージが作成され、アカウントが初期化されるのを待ちます。
2. PowerShell プロンプトで、**Get-AzSubscription** と入力してサブスクリプションを表示します。
3. **Get-AzResourceGroup** と入力して、リソース グループ情報を表示します。

## <a name="experiment-with-the-bash-shell"></a>Bash シェルを使用してみる

1. ドロップダウンを使用して **Bash** シェルに切り替え、選択を確認します。
2. Bash シェル プロンプトで、**az account list** と入力してサブスクリプションを表示します。 また、タブの補完を試してみてください。 
3. **az resource list** と入力して、リソース情報を表示します。

## <a name="experiment-with-the-cloud-editor"></a>Cloud Editor を使用してみる

1. Cloud Editor を使用するには、**code** と入力します。 中かっこアイコンを選択することもできます。 
2. 左側のナビゲーション ペインからファイルを選択します。 たとえば、 **.profile** です。
3. エディターの上部バナーに、設定 (テキスト サイズとフォント) とファイルのアップロード/ダウンロードの選択肢があることに注意してください。
4. [保存]、[エディターを閉じる]、および [ファイルを開く] の右端にある省略記号 ( **...** リソース グループ:) に注意してください。
5. 時間が許す限り試し、その後 Cloud Editor を**閉じます**。 
6. Cloud Shell を閉じます。