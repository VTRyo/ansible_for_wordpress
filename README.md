# Ansibleによるインフラ自動化入門

このAnsibleコードをforkして実行すると、wordpressサーバのベース部分が自動構築されます。

## 自動化箇所
* よくつかうコマンドのインストール
* hostnameの変更
* タイムゾーンの変更
* ユーザ作成
* SFTPグループの作成
* visudo
* authorized_key登録
* WEB：h2oのセットアップ
* APP: php7（php-fpm）
* DB: mysql5.7
* zabbix-agent
* cronのセット

実行後、対象のサーバのミドルウェアは起動状態になっています。<br>
自身のIPでアクセスをするとテストページが表示されます。

## 手動で実行する箇所
* wordpressのインストールとセットアップ
* mysqlの初期セットアップ
* サーバ証明書の取得

## 使い方
### group_varsに変数定義する
各々で変数を再定義してください。
* server_hostname
* Domain_name
* allow_ip1 （webserverのアクセスコントロール）
* allow_ip2 （webserverのアクセスコントロール）

### hostsに対象サーバを再定義する
IP、ドメイン名など、環境に合わせて適切にhostsを定義してください。

### ユーザパスワードの記述
roles/common/tasks/main.ymlの46行目にユーザパスワード「xxxxx」を任意のパスワードに変更してください。

### 公開鍵の記述
roles/common/tasks/main.ymlの最終行、""内に公開鍵を記述してください。

### h2oの/files/bintray-tatsushid-h2o-rpm.repoはamazon Linux用です
baseurlがcentosとamazon Linuxとで異なりますので、環境に合わせて変更してください。

### PHP7のmain.ymlはamazon Linux用です
main.yml内にて--disablerepo=amzn-mainを記述しています。EC2でremiを使用するには、amzon Linux固有のリポジトリを明示的に”使用しない”宣言をします。<br>
そのため、対象のサーバがamazon Linuxではない場合は、--disablerepo=amzn-mainの記述を削除してください。

### mysqlはインストール以降の設定は手動です
mysql_secure_installationは手動実行してください。<br>
※mysql用ログインパスワードは/root/.my.cnfに出力されるよう、Ansibleで処理をしています。

### h2oのテンプレートでは標準でssl設定をコメントアウトしています。
証明書はletsencryptで取得した場合のパスを記述しています。<br>
ssl設定を有効にする場合は証明書を取得後に有効にしてください。


## 実行コマンド
### ansible DryRun実行
```
ansible-playbook -i hosts wordpress.yml --check
```

### ansible 本番実行
```
ansible-playbook -i hosts wordpress.yml
```

### ansible 鍵とログインユーザを指定して実行
```
ansible-playbook --private-key=<key> -i hosts -u <user> wordpress.yml --check
```

## 補足
### .sshファイル
対象のサーバのIPや鍵の設定を予めしておくことで、Ansibleをより快適に使用できます。

### init.dのスクリプトはh2oの起動スクリプト
デフォルトの起動スクリプトを使用するとrestartしない不具合が起こるため、こちらを使用のオススメします。

### crontabに記述するコマンドの意味
wordpressを構築後、コンテンツバックアップは「backwpup」プラグインを使用する想定でこのcron設定をしています。<br>
必要がない場合はwordpress.ymlからcronをコメントアウトしてください。

### wordpressのインストールを自動化しない理由
ansibleで最も重要視すべきは冪等性です。同じコマンドを何度打っても、同じ結果に収束する必要があります。wordpressはコンテンツの展開をするまでがサーバ側でのインストール準備になります。ansibleを打つ度にwordpressが展開されてしまうと、毎回初期化される危険に晒されます。そのため、ansibleの管理化から除外しました。

### zabbix-agentはzabbix-serverがいないと稼働しません
今回ansibleに記述しているのはzabbix-agent側の設定です。<br>
zabbix監視を行う場合は、zabbix-serverの構築が必要です。

### zabbix-agentについて
tasks/main.yml序盤のセクションにあるrpmインストールやrpm keyのインストールは、DryRunではスキップされるため、実際に本番適用させる必要があります。<br>
本コードはすでに何度も本番環境で実行された実績があります。
