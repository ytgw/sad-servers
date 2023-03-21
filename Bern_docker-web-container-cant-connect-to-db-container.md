# Docker web container can't connect to db container.


## Scenario
"Bern": Docker web container can't connect to db container.


## Level
Hard


## Type
Fix


## Access
Public


## Description
There are two Docker containers running, a web application (Wordpress or WP) and a database (MariaDB) as back-end, but if we look at the web page, we see that it cannot connect to the database.
curl -s localhost:80 |tail -4 returns:

```html
<body id="error-page"> <div class="wp-die-message"><h1>Error establishing a database connection</h1></div></body> </html>
```

This is not a Wordpress code issue (the image is :latest with some network utilities added).
What you need to know is that WP uses "WORDPRESS_DB_" environment variables to create the MySQL connection string.
See the ./html/wp-config.php WP config file for example (from /home/admin).


## Description(日本語)
Dockerコンテナが2つ動いていて、Webアプリケーション（Wordpress、WP）とバックエンドとしてデータベース（MariaDB）があるのですが、Webページを見ると、データベースに接続できないことがわかります。
curl -s localhost:80 |tail -4 が返ってきます。

```html
<body id="error-page"> <div class="wp-die-message"><h1>Error establishing a database connection</h1></div></body> </html>
```

これはWordpressのコードの問題ではありません（画像はネットワークユーティリティを追加した:最新版です）。
知っておくべきことは、WPは「WORDPRESS_DB_」環境変数を使用してMySQL接続文字列を作成することです。
例えば、./html/wp-config.php WP設定ファイル（/home/adminから）をご覧ください。


## Test
sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping.
The wordpress container is able to connect to the database in the mariadb container and returns mysqld is alive.


## Time to Solve
20 minutes.


## OS
Debian 11


## 回答

### DBへの接続
```bash
# アクセステスト
curl -s localhost:80 |tail -4
# 以下は出力抜粋
# <h1>Error establishing a database connection</h1>

# wordpress内からのDB接続テスト
sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping
```

出力結果は下記の通りで、mysqlというホストがないと出る。
```
mysqladmin: connect to server at 'mysql' failed
error: 'Unknown MySQL server host 'mysql' (-2)'
Check that mysqld is running on mysql and that the port is 3306.
You can check this by doing 'telnet mysql 3306'
```

```bash
# コンテナの状態確認
sudo docker ps -a
# 出力は下記の通りで、localhost:3306からDBへアクセス可能
# CONTAINER ID   IMAGE            COMMAND                  CREATED        STATUS         PORTS                    NAMES
# 6ffb084b515c   wordpress:sad    "docker-entrypoint.s…"   7 months ago   Up 2 minutes   0.0.0.0:80->80/tcp       wordpress
# 0eef97284c44   mariadb:latest   "docker-entrypoint.s…"   7 months ago   Up 2 minutes   0.0.0.0:3306->3306/tcp   mariadb

# ホストからDBへアクセス
mysql -u root -ppassword -h localhost
# 出力は下記の通りで、ソケット経由のアクセスになっている。
# ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/run/mysqld/mysqld.sock' (2)

# IPアドレス指定で、ホストからDBへアクセス
mysql -u root -ppassword -h 127.0.0.1
```

出力は下記の通りで、DBへアクセス可能。
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.8.3-MariaDB-1:10.8.3+maria~jammy mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

```bash
# wordpressの/etc/hostsを確認
sudo docker exec wordpress cat /etc/hosts
```

出力は下記の通りで、mysqlというホスト名とIPアドレスが対応づいていない。
```
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      6ffb084b515c
```

### wordpressコンテナにDBホストを追加

```bash
# DBのIPアドレスを取得
sudo docker inspect mariadb | grep -i ipaddress
# 出力結果から172.17.0.2と分かる。

# wordpressの実行時オプションを確認
sudo docker inspect wordpress
# 出力結果は省略

# wordpressコンテナの削除
sudo docker stop wordpress
sudo docker rm wordpress

# wordpressコンテナの起動
sudo docker run --name wordpress -v "html:/var/www/html" -p "80:80" --add-host "mysql:172.17.0.2" -d wordpress:sad
```

### 修正確認

```bash
# wordpress内からのDB接続テスト
sudo docker exec wordpress mysqladmin -h mysql -u root -ppassword ping
# 出力結果
# mysqld is alive

# curlでの接続テスト
curl -s localhost:80 |tail -4
# 出力なしだが、echo $?で確認するとエラーはない。
```
