# <a name="demonstration-connect-to-linux-virtual-machines"></a>デモ: Linux 仮想マシンに接続する

このデモでは、Linux マシンを作成して SSL でアクセスします。

>**注:**  接続が機能するようにポート 22 が開いていることを確認します。 

## <a name="create-the-ssh-keys"></a>SSH キーの作成

1. Download the PuTTY tool. This will include <bpt id="p1">[</bpt>PuTTYgen<ept id="p1">](https://putty.org/)</ept>. 
2. インストールされたら、**PuTTYgen** プログラムを見つけて開きます。
3. **[パラメーター]** オプション グループで、**[RSA]** を選択します。
4. **[生成]** ボタンをクリックします。
5. マウスをウィンドウ内の空白領域の辺りに移動して、ある程度のランダム性を生成します。
6. **[認可されたキー ファイルに貼り付けるための公開キー]** のテキストをコピーします。
7. Optionally you can specify a <bpt id="p1">**</bpt>Key passphrase<ept id="p1">**</ept> and then <bpt id="p2">**</bpt>Confirm passphrase.<ept id="p2">**</ept> You will be prompted for the passphrase when you authenticate to the VM with your private SSH key. Without a passphrase, if someone obtains your private key, they can sign in to any VM or service that uses that key. We recommend you create a passphrase. However, if you forget the passphrase, there is no way to recover it.
8. **[秘密キーを保存]** をクリックします。
9. Choose a location and filename and click <bpt id="p1">**</bpt>Save<ept id="p1">**</ept>. You will need this file to access the VM. 

## <a name="create-the-linux-machine-and-assign-the-public-ssh-key"></a>Linux マシンを作成し、SSH 公開キーを割り当てる

1. Azure portal で選択した Linux マシンを作成します。
2. **[認証の種類]** として (**[パスワード]** ではなく) **[SSH 公開キー]** を選択します。
3. **[ユーザー名]** を指定します。
4. Paste the public SSH key from PuTTY into the <bpt id="p1">**</bpt>SSH public key<ept id="p1">**</ept> text area. Ensure the key validates with a checkmark. 
5. Create the VM. Wait for it to deploy.
6. 実行中の VM にアクセスします。 
7. **[概要]** ブレードから、**[接続]** をクリックします。
8. ユーザーとパブリック IP アドレスを含むログイン情報を書き留めておきます。

## <a name="access-the-server-using-ssh"></a>SSH を使用してサーバーにアクセスする

1. **PuTTY** ツールを開きます。
2. 「**username@publicIpAddress**」と入力します。ここで、username は VM を作成するときに割り当てた値であり、publicIpAddress は Azure portal から取得した値です。
3. **[ポート]** として **22** を指定します。
4. **[接続の種類]** オプション グループで **[SSH]** を選択します。
5. [カテゴリ] パネルで **[SSH]** に移動し、**[認証]** をクリックします。
6. **[認証用の秘密キー ファイル]** の横にある **[参照]** ボタンをクリックします。
7. SSH キーを生成したときに保存された秘密キー ファイルに移動し、**[開く]** をクリックします。
8. PuTTY のメイン画面から、**[開く]** をクリックします。
9. これで、サーバーのコマンド ラインに接続されます。 