# <a name="demonstration-work-with-azure-cli-locally"></a>デモンストレーション: Azure CLI をローカルで使用する

このデモでは、CLI をインストールして使用し、リソースを作成します。

## <a name="install-the-cli-on-windows"></a>Windows に CLI を インストールする

MSI インストーラーを使用して Windows オペレーティング システムに Azure CLI をインストールします。

1. [MSI インストーラー](https://aka.ms/installazurecliwindows)をダウンロードして、ブラウザーのセキュリティ ダイアログ ボックスで **[実行]** をクリックします。
2. インストーラーで、ライセンス条項に同意し、**[インストール]** をクリックします。
3. **[ユーザー アカウント制御]** ダイアログで、**[はい]** を選択します。

## <a name="verify-azure-cli-installation"></a>Azure CLI のインストールの確認

Azure CLI は、Linux または Mac OS で Bash シェルを開くか、Windows でコマンド プロンプトまたは PowerShell から実行できます。

Azure CLI を起動し、バージョン チェックを実行してインストールを確認します。

```azurecli
az --version
 ```

>**注**:PowerShell から Azure CLI を実行する場合、Windows コマンド プロンプトから Azure CLI を実行するよりもメリットがあります。 PowerShell には、コマンド プロンプトよりも多くのタブ補完機能があります。

## <a name="login-to-azure"></a>Azure にログインする

ローカルの Azure CLI インストールを操作するため、Azure コマンドを実行する前に認証を行う必要があります。 これは、Azure CLI の**ログイン** コマンドを使用して実行します。

```azurecli
az login
```

通常、Azure CLI は既定のブラウザーを起動して Azure サインイン ページを開きます。 これでうまくいかない場合、コマンドラインの指示に従い、[https://aka.ms/devicelogin](https://aka.ms/devicelogin) で承認コードを入力します。

サインインに成功すると、ご利用の Azure サブスクリプションに接続されます。

## <a name="create-a-resource-group"></a>リソース グループを作成する

新しい Azure サービスを作成する前に新しいリソース グループを作成しなければならないことが頻繁にあります。そこで、例としてリソース グループを使用し、CLI から Azure リソースを作成する方法について示します。

Azure CLI **group create** コマンドは、リソース グループを作成します。 名前と場所を指定する必要があります。 *名前*はサブスクリプション内で一意である必要があります。 *保存先*によって、リソース グループのメタデータが格納される場所が決まります。 "米国西部"、"北ヨーロッパ"、"インド西部" などの文字列を使用して場所を指定するか、westus、northeurope、westindia など、同じものを意味する 1 つの単語を使用できます。 中心的な構文は次のようになります。

```azurecli
az group create --name <name> --location <location>
```

## <a name="verify-the-resource-group"></a>リソース グループを確認する

多くの Azure リソースでは、Azure CLI には、リソースの詳細を表示するための **list** サブコマンドが用意されています。 たとえば、Azure CLI **group list** コマンドによって Azure リソース グループが一覧表示されます。 これは、リソース グループの作成が成功したかどうかを確認するのに役立ちます。

```azurecli
az group list
```

さらに簡潔なビューを得るために、出力をシンプルな表として書式設定できます。

```azurecli
az group list --output table
```

グループ リストに複数の項目がある場合は、**クエリ** オプションを追加すると戻り値をフィルタリングできます。 次のコマンドを試してください。

```azurecli
az group list --query "[?name == '<rg name>']"
```

>**注:**  JSON 要求の標準クエリ言語である **JMESPath** を使用して、クエリの書式を設定します。 この強力なフィルター言語の詳細については、[http://jmespath.org/](http://jmespath.org/) を参照してください。