# <a name="demonstration-create-an-alert-rule"></a>デモンストレーション: アラート ルールを作成する

## <a name="create-rule"></a>ルールを作成する

1. Azure Portal で、 **[監視]** をクリックします。 [モニター] ブレードでは、すべての監視設定とデータが 1 つのビューにまとめられています。
2. **[アラート]** をクリックして、 **[+ 新しいアラート ルール]** をクリックします。 ほとんどのリソース ブレードには [監視] の下のリソース メニューにもアラートが含まれるため、そこからアラートを作成することもできます。

## <a name="explore-alert-targets"></a>アラート ターゲットの詳細を確認する

1. ターゲット の下にある **[選択]** をクリックして、警告するターゲット リソースを選択します。 **サブスクリプション**と**リソースの種類**のドロップダウン リストを使用して、監視するリソースを検索します。 検索バーを使用して、リソースを検索することもできます。
2. 選択したリソースにアラートを作成できるメトリックがある場合は、右下の [使用可能なシグナル] にメトリックが表示されます。 メトリック アラートでサポートされているリソースの種類の完全な一覧については、こちらの記事をご覧ください。
3. 選択が完了したら、 **[完了]** をクリックします。

## <a name="explore-alert-conditions"></a>アラート条件を確認する

1. ターゲット リソースを選択した後、 **[条件の追加]** をクリックします。
2. リソースのためにサポートされているシグナルの一覧を監視し、アラートを作成するメトリックを選択します。
3. 必要に応じて、 [期間] と [集計] を調整して、メトリックを設定し直します。 メトリックにディメンションがある場合は、[ディメンション] テーブルが表示されます。 
4. 過去 6 時間のメトリックのグラフを監視します。 **[履歴の表示]** ドロップダウンを調整します。
5. **[アラート ロジック]** を定義します。 これにより、メトリック アラート ルールが評価するロジックが決定されます。
6. 静的しきい値を使用している場合、メトリック グラフが妥当なしきい値の決定に役立つことがあります。 動的しきい値を使用している場合は、メトリック グラフに最新のデータに基づいて計算されたしきい値が表示されます。
7. **[Done]** をクリックします。
8. 複雑なアラート ルールを監視する場合は、必要に応じて、別の条件を追加します。 

## <a name="explore-alert-details"></a>アラートの詳細を詳しく見る

1. **アラート ルール名**、**説明**、**重大度**などのアラートの詳細を入力します。
2. 既存のアクション グループを選択するか、新しいアクション グループを作成して、アラートにアクション グループを追加します。
3. **[完了]** をクリックして、メトリック アラート ルールを保存します。