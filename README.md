# docker-compose を使った LNMP 環境のひな形

## コレは何？

docker-compose すれば PHP Web アプリの環境が作れる的なヤツ。開発環境を想定してコンテンツや PHP スクリプトはホスト側におく設定になってる。本番環境として使っても問題ない程度にセキュアな設定にしているつもり

## 使う前に

以下の点を変更すること

### FQDN

ブラウザからアクセスされる場合の FQDN。www.なんちゃらとか、開発環境なら localhost とか。

* nginx/conf.d/site.conf
  * 「server_name」の your_host_fqdn のところ
  * 「access_log」の your_host_fqdn_access.log のところ

#### プロジェクトのパス

コンテンツやらPHPスクリプトやらを置く場所。/var/www/なんちゃらとか、開発環境なら /home/user/なんちゃらとか。

* docker-compose.yml
  * 「php」の「volumes」の your_project_local_path のところ
  * 「nginx」の「volumes」の your_project_local_path のところ

### メール送信サーバー
* php/otherconf/esmtprc を編集する。Gmail 用になっているので、username と password に自分のものを設定する
* いちばん単純には、hostname に認証の必要ない MUA を設定して、他は全部コメントアウトすれば良いかもしんない

### ログ出力先

nginx とかのログ出力先をホスト側ディレクトリにしておく。エラーがおきてもわからないので。指定した先に nginx というサブディレクトリを事前に掘っておくこと。/var/log とか、開発環境なら /home/user/なんちゃらとか

* docker-compose.yml
  * 「nginx」の「volumes」の your_log_local_path のところ


### ヴァーチャルホストにする場合

* nginx/conf.d/site.conf を別名でコピーして、さっきの要領で your_host_fqdn、your_host_fqdn_access.log、your_project_local_path を書き換える
* 同じように、「root」の /var/www/webroot も適当に変更する
* docker-compose.yml の「php」および「nginx」の「volumes」の webroot も上の変更にあわせて追記する

### SSL/TLS サイトにする場合

* nginx/conf.d/site.conf の下の方でコメントアウトされている箇所を参考に。例として Let's Encrypt を使う想定になってる
* docker-compose.yml の「nginx」の「ports」でコメントアウトされてる 443 ポートを有効にする
* HSTS を有効にしたい場合は nginx/conf.d/000security.conf 下の方でコメントアウトされている箇所を参考に

### MySQL の root パスワード

docker-compose.yml で MYSQL_ROOT_PASSWORD として定義している。変更したければ「secret」を書き換える

## 実行方法

「YOUR_PROJECT_NAME」は適当に名前をつけて読み替えること

### 初回起動

```
$ docker-compose -p YOUR_PROJECT_NAME up -d
```

### 終了

```
$ docker-compose -p YOUR_PROJECT_NAME stop
```


### 2回目以降の起動

```
$ docker-compose -p YOUR_PROJECT_NAME start
```

### MySQL への CLI アクセス

mysql コンテナで mysql コマンドを使うとなぜだか日本語が通らないので、php コンテナから使う例

```
$ docker exec -it YOUR_PROJECT_NAME_php_1 mysql -uroot -psecret -hmysql mysql
```

## その他

ホスト環境は Debian または Ubuntu であることを想定している。変に uid とかをチェックしていなければ他の環境でも問題ないと思うけど、世の中にはそうい PHP アプリもあるので注意
