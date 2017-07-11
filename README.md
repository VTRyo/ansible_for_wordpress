# Ansibleによるインフラ自動化入門

## 使い方
### group_varsに変数定義する
各々で変数を再定義してください。
* server_hostname
* Domain_name

### mysqlはインストール以降の設定は手動です

### wordpressはDocumentRootへの展開以降は手動です

### zabbix-agentはzabbix-serverがいないと稼働しません

### 公開鍵の記述
roles/common/tasks/main.ymlの最終行、""内に公開鍵を記述してください。

## ansible DryRun実行
```
ansible-playbook -i hosts wordpress.yml --check
```

## ansible 本番実行
```
ansible-playbook -i hosts wordpress.yml
```

## ミドルウェア
* h2o
* mysql5.7
* php7
* zabbix-agent

## 補足
### .sshファイル
対象のサーバのIPや鍵の設定を予めしておくことで、Ansibleをより快適に使用できます。

### init.dのスクリプトはh2oの起動スクリプトです。
デフォルトの起動スクリプトを使用するとrestartしない不具合が起こるため、こちらを使用のオススメします。

### PHP7のmain.ymlはamazon Linux用です。
main.yml内にて--disablerepo=amzn-mainを記述しています。EC2でremiを使用するには、amzon Linux固有のリポジトリを明示的に”使用しない”宣言をします。<br>
そのため、対象のサーバがamazon Linuxではない場合は、--disablerepo=amzn-mainの記述を削除してください。

### zabbix-agentについて
tasks/main.yml序盤のセクションにあるrpmインストールやrpm keyのインストールは、DryRunではスキップされるため、実際に本番適用させる必要があります。<br>
本コードはすでに何度も本番環境で実行された実績があります。
