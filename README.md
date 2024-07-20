# sample-cfn

`templates/bastion.yml` は EC2 インスタンスを作成する CloudFormation テンプレートです。

ローカルから EC2 インスタンスに SSH 接続して、Terraform の開発と実行などの作業を行うことを想定しています。

## 使い方

### 0. リポジトリのクローン

1. このリポジトリをクローンします。

    ```bash
    git clone https://github.com/KyoheiNakamura/sample-cfn.git
    ```

### 1. 踏み台サーバーの作成

> [!NOTE]
> AWS コンソールで作業します。

#### 1-1. CloudFormation スタックの作成

1. サービスから `CloudFormation` を選択します。

    ![alt text](./docs/images/aws-cfn-select.png)

2. `スタックの作成` をクリックします。

    ![alt text](./docs/images/aws-cfn-create.png)

3. `テンプレートファイルのアップロード` を選択し、`templates/bastion.yml` をアップロードします。

    ![alt text](./docs/images/aws-cfn-create-template.png)

4. `スタック名` を入力し、`Env` に適切な環境を選択します。

    ![alt text](./docs/images/aws-cfn-create-stack.png)

5. `次へ` をクリックして、スタックが正常に作成されることを確認します。

    ![alt text](./docs/images/aws-cfn-create-confirm.png)

#### 1-2. 踏み台サーバーのパブリック DNS のメモ

1. [1. 踏み台サーバーの作成](#1-踏み台サーバーの作成) で作成した bastion スタックの出力から、踏み台サーバーのパブリック DNS (`EC2BastionPublicDnsName`) の値をメモします。

    ![alt text](./docs/images/aws-cfn-bastion-output-ec2.png)

#### 1-3. SSH プライベートキーのメモ

1. [1. 踏み台サーバーの作成](#1-踏み台サーバーの作成) で作成した bastion スタックの出力から、SSH プライベートキー (`KeyPairBastionLocation`) の値をメモします。

    ![alt text](./docs/images/aws-cfn-bastion-output-keypair.png)

2. サービスから `パラメータストア` を選択します。

    ![alt text](./docs/images/aws-ssm-ps-select.png)

3. 1 でメモした `KeyPairBastionLocation` の値を選択します。

    ![alt text](./docs/images/aws-ssm-ps-select-keypair.png)

4. `復号化された値を表示` をクリックし、`プライベートキー` をコピーしてメモします。

    ![alt text](./docs/images/aws-ssm-ps-save-keypair.png)

### 2. SSH 接続

> [!NOTE]
> ローカルで作業します。

#### 2-1. SSH 接続の設定

[1-3. SSH プライベートキーのメモ](#1-3-ssh-プライベートキーのメモ) でメモしたプライベートキーをローカルに保存し、踏み台サーバーに SSH 接続します。

1. `~/.ssh/bastion.pem` にプライベートキーを保存して、パーミッションを変更します。

    ```bash
    echo "{プライベートキー}" > ~/.ssh/bastion.pem && chmod 400 ~/.ssh/bastion.pem
    ```

2. `~/.ssh/config` に以下の設定を追加します。

    HostName には [1-2. 踏み台サーバーのパブリック DNS のメモ](#1-2-踏み台サーバーのパブリック-dns-のメモ) でメモした値を入力します。

    ```bash
    echo "
    # 踏み台サーバー
    Host bastion
      HostName {踏み台サーバーのパブリック DNS}
      User ec2-user
      IdentityFile ~/.ssh/bastion.pem
      ServerAliveInterval 60
      ServerAliveCountMax 120" >> ~/.ssh/config
    ```

#### 2-2. ターミナルからの SSH 接続

1. SSH 接続をします。

    ```bash
    ssh bastion
    ```

    <details>

    <summary>ログ</summary>

    ```log
    ~
    ❯ ssh bastion
       ,     #_
       ~\_  ####_        Amazon Linux 2023
      ~~  \_#####\
      ~~     \###|
      ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
       ~~       V~' '->
        ~~~         /
          ~~._._/
             _/_/
           _/m/'
    Last login: Fri Jul 19 05:00:32 2024 from xxx.xxx.xxx.xxx
    [ec2-user@ip-10-0-240-231 ~]$
    ```

    </details>

### 3. VSCode での開発環境の構築

> [!NOTE]
> ローカルで作業します。

#### 3-1. VSCode からの SSH 接続

VSCode から SSH 接続するための設定を行います。

1. VSCode に `Remote - SSH` 拡張機能をインストールします。

    ```bash
    code --install-extension ms-vscode-remote.remote-ssh
    ```

    左側のアイコンから拡張機能を検索してインストールすることもできます。

    ![alt text](./docs/images/vscode-remote-ssh-install.png)

2. 左下の `><` アイコンをクリックするかコマンドパレットを開いて、`Remote-SSH: Connect to Host...` を選択し、SSH Host に `bastion` を選択します。

    ![alt text](./docs/images/vscode-remote-ssh-connect.png)

3. 左下の `><` アイコンの右側に `SSH: bastion` と表示されていることを確認します。

    ![alt text](./docs/images/vscode-remote-ssh-connected.png)

#### 3-2. Dev Container の立ち上げ

> [!NOTE]
> VSCode で `bastion` に SSH 接続した状態で作業します。

1. VSCode に `Remote - Containers` 拡張機能をインストールします。

    ```bash
    code --install-extension ms-vscode-remote.remote-containers
    ```

    左側のアイコンから拡張機能を検索してインストールすることもできます。

    ![alt text](./docs/images/vscode-remote-container-install.png)

2. 左下の `><` アイコンをクリックするかコマンドパレットを開いて、`Remote-Containers: Reopen in Container` を選択します。

    ![alt text](./docs/images/vscode-remote-container-reopen.png)

3. 左下の `><` アイコンの右側に `Dev Container: Sample CFn @  bastion` と表示されていることを確認します。

    ![alt text](./docs/images/vscode-remote-container-running.png)

#### 3-3. 推奨拡張機能のインストール

1. コマンドパレットを開いて、`Extensions: Show Recommended Extensions` を選択して、推奨の拡張機能を検索できます。

    ![alt text](./docs/images/vscode-extensions-recommended-show.png)

    左側のアイコンから拡張機能を `@recommended` で検索することもできます。

    ![alt text](./docs/images/vscode-extensions-recommended-search.png)

2. 左側のアイコンから拡張機能を `@recommended` で検索して、インストールアイコンをクリックすることでまとめてインストールできます。

    ![alt text](./docs/images/vscode-extensions-recommended-install-icon.png)
