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

><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: Running Azure CLI from PowerShell has some advantages over running Azure CLI from the Windows command prompt. PowerShell provides more tab completion features than the command prompt.

## <a name="login-to-azure"></a>Azure にログインする

Because you're working with a local Azure CLI installation, you'll need to authenticate before you can execute Azure commands. You do this by using the Azure CLI <bpt id="p1">**</bpt>login<ept id="p1">**</ept> command:

```azurecli
az login
```

Azure CLI will typically launch your default browser to open the Azure sign-in page. If this doesn't work, follow the command-line instructions and enter an authorization code at <bpt id="p1">[</bpt><ph id="ph1">https://aka.ms/devicelogin</ph><ept id="p1">](https://aka.ms/devicelogin)</ept>.

サインインに成功すると、ご利用の Azure サブスクリプションに接続されます。

## <a name="create-a-resource-group"></a>リソース グループを作成する

新しい Azure サービスを作成する前に新しいリソース グループを作成しなければならないことが頻繁にあります。そこで、例としてリソース グループを使用し、CLI から Azure リソースを作成する方法について示します。

Azure CLI <bpt id="p1">**</bpt>group create<ept id="p1">**</ept> command creates a resource group. You must specify a name and location. The <bpt id="p1">*</bpt>name<ept id="p1">*</ept> must be unique within your subscription. The <bpt id="p1">*</bpt>location<ept id="p1">*</ept> determines where the metadata for your resource group will be stored. You use strings like "West US", "North Europe", or "West India" to specify the location; alternatively, you can use single word equivalents, such as westus, northeurope, or westindia. The core syntax is:

```azurecli
az group create --name <name> --location <location>
```

## <a name="verify-the-resource-group"></a>リソース グループを確認する

For many Azure resources,  Azure CLI provides a <bpt id="p1">**</bpt>list<ept id="p1">**</ept> subcommand to view resource details. For example, the Azure CLI <bpt id="p1">**</bpt>group list<ept id="p1">**</ept> command lists your Azure resource groups. This is useful to verify whether resource group creation was successful:

```azurecli
az group list
```

さらに簡潔なビューを得るために、出力をシンプルな表として書式設定できます。

```azurecli
az group list --output table
```

If you have several items in the group list, you can filter the return values by adding a <bpt id="p1">**</bpt>query<ept id="p1">**</ept> option. Try this command:

```azurecli
az group list --query "[?name == '<rg name>']"
```

><bpt id="p1">**</bpt>Note:<ept id="p1">**</ept> You format the query using <bpt id="p2">**</bpt>JMESPath<ept id="p2">**</ept>, which is a standard query language for JSON requests. Learn more about this powerful filter language at <bpt id="p1">[</bpt><ph id="ph1">http://jmespath.org/</ph><ept id="p1">](http://jmespath.org/)</ept>.