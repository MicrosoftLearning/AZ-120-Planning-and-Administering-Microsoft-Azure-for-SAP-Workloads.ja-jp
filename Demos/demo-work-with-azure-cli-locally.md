# デモ: Azure CLI をローカルで操作する

このデモでは、CLI をインストールして使用し、リソースを作成します。

## Windows に CLI を インストールする

MSI インストーラーを使用して Windows オペレーティング システムに Azure CLI をインストールします。

1. [MSI インストーラー](https://aka.ms/installazurecliwindows)をダウンロードして、ブラウザーのセキュリティ ダイアログ ボックスで 「**実行**」 をクリックします。
2. インストーラーで、ライセンス条項に同意し、「**インストール**」 をクリックします。
3. 「**ユーザー アカウント制御**」 ダイアログ ボックスで、「**はい**」 を選択します。

## Azure CLI インストールを確認する

Azure CLI は、Linux または Mac OS で Bash シェルを開くか、Windows でコマンド プロンプトまたは PowerShell から実行できます。

Azure CLI を起動し、バージョン チェックを実行してインストールを確認します。

```azurecli
az --version
 ```

> **注**: PowerShell から Azure CLI を実行すると、Windows コマンド プロンプトから Azure CLI を実行するよりもメリットがあります。PowerShell には、コマンド プロンプトよりも多くのタブ補完機能があります。

## Azure にログインする

ローカルの Azure CLI インストールを使用しているため、Azure コマンドを実行する前に認証する必要があります。これは、Azure CLI の**ログイン** コマンドを使用して実行します。

```azurecli
az login
```

通常、Azure CLI は既定のブラウザーを起動して Azure サインイン ページを開きます。これがうまくいかない場合は、コマンドラインの指示に従い、[https://aka.ms/devicelogin](https://aka.ms/devicelogin) で認証コードを入力します。

サインインが成功すると、Azure サブスクリプションに接続されます。

## リソース グループを作成する

新しい Azure サービスを作成する前に、新しいリソース グループの作成が必要な場合がよくあるため、リソース グループを使用して CLI から Azure リソースを作成する方法を例として示します。

Azure CLI **group create** コマンドは、リソース グループを作成します。名前と保存先を指定する必要があります。*名前*はサブスクリプション内で一意である必要があります。*保存先*によって、リソース グループのメタデータが格納される場所が決まります。保存先を指定するには、「米国西部」、「北ヨーロッパ」、「インド西部」 などの文字列を使用します。または、westus、northeurope、westindia など、相当する 1 つの単語を使用することもできます。構文の基本は以下のとおりです。

```azurecli
az group create --name <name> --location <location>
```

## リソース グループを確認する

多くの Azure リソースでは、Azure CLI には、リソースの詳細を表示するための **list** サブコマンドが用意されています。たとえば、Azure CLI **group list** コマンドは、Azure リソース グループを一覧表示します。これは、リソース グループの作成が成功したかどうかを確認するのに役立ちます。

```azurecli
az group list
```

より簡潔なビューを取得するには、出力を単純なテーブルとして書式設定できます。

```azurecli
az group list --output table
```

グループ リストに複数の項目がある場合は、**クエリ** オプションを追加すると戻り値をフィルタリングできます。次のコマンドを試してください。

```azurecli
az group list --query "[?name == '<rg name>']"
```

>**注:** JSON 要求の標準照会言語である **JMESPath** を使用してクエリを書式設定します。この強力なフィルター言語の詳細については、[http://jmespath.org/](http://jmespath.org/) を参照してください。
