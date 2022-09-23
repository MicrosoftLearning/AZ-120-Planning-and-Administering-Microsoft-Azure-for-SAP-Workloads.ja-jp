# <a name="demonstration-create-an-azure-policy"></a>デモ: Azure のポリシーを作成する

## <a name="assign-a-policy"></a>ポリシーを割り当てる

1. Launch the Azure Policy service in the Azure portal by clicking <bpt id="p1">**</bpt>All services<ept id="p1">**</ept>, then searching for and selecting <bpt id="p2">**</bpt>Policy<ept id="p2">**</ept>. This service is under <bpt id="p1">**</bpt>Management and Governance<ept id="p1">**</ept>.
2. Select <bpt id="p1">**</bpt>Assignments<ept id="p1">**</ept> on the left side of the Azure Policy page. An assignment is a policy that has been assigned to take place within a specific scope.
3. [ポリシー - 割り当て] ページの上部で **[ポリシーの割り当て]** を選択します。
4. ポリシーの割り当てが適用されるリソースまたはリソースのグループを決定する**スコープ**に注目してください。
5. Select the <bpt id="p1">**</bpt>Policy definition ellipsis<ept id="p1">**</ept> to open the list of available definitions. Take some time to review the built-in policy definitions.
6. Azure portal 上で **[すべてのサービス]** をクリックし、 **[ポリシー]** を検索して選択し、Azure Policy サービスを起動します。
7. **[管理対象 ID の作成]** はオフのままにします。 
8. **[割り当て]** をクリックしてポリシーを作成します。

## <a name="create-and-assign-an-initiative-definition"></a>イニシアチブ定義の作成と割り当て

1. Azure Policy ページの左側にある [作成] の下の **[定義]** を選択します。
2. ページの上部にある **[+ イニシアティブ定義]** を選択して [イニシアティブ定義] ページを開きます。
3. **名前**と**説明**を入力します。
4. **新しいカテゴリを作成**します。
5. 右側のパネルから **[SQL Server バージョン 12.0 を必須とする]** ポリシーを **追加**します。
6. 選択したポリシーを 1 つ追加します。
7. 変更内容を**保存**し、イニシアティブ定義をサブスクリプションに**割り当て**ます。

## <a name="check-for-compliance"></a>コンプライアンスを確認する

1. [Azure Policy サービス] ページに戻ります。
2. **[コンプライアンス]** を選択します。
3. ポリシーのステータスと定義を確認します。 