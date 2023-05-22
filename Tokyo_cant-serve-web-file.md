# can't serve web file


## Scenario
"Tokyo": can't serve web file


## Level
Medium


## Type
Fix

## Access
Public


## Description
There's a web server serving a file /var/www/html/index.html with content "hello sadserver" but when we try to check it locally with an HTTP client like curl 127.0.0.1:80, nothing is returned.
This scenario is not about the particular web server configuration and you only need to have general knowledge about how web servers work.


## Description(日本語)
コンテンツ "hello sadserver" を含むファイル /var/www/html/index.html を提供しているウェブサーバーがありますが、curl 127.0.0.1:80 のような HTTP クライアントでローカルにそれを確認しようとすると、何も返されません。
このシナリオは、特定のWebサーバーの構成に関するものではないので、Webサーバーがどのように動作するかについての一般的な知識があれば大丈夫です。


## Test
curl 127.0.0.1:80 should return: hello sadserver


## Time to Solve
15 minutes.


## OS
Ubuntu 22.04 LTS


## 回答

状況確認のため、```curl 127.0.0.1:80```で接続テストを実行したが、何も出力されず、Ctrl-Cで終了した。
プロセスが立ち上がっていないか、ファイアウォールでブロックされている可能性がある。

```lsof -i:80```で接続先のポートをリッスンしているプロセスが起動しているか確認する。
出力結果は下記の通りで、プロセスは起動している。
```
COMMAND PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2 619     root    4u  IPv6  16725      0t0  TCP *:http (LISTEN)
apache2 769 www-data    4u  IPv6  16725      0t0  TCP *:http (LISTEN)
apache2 770 www-data    4u  IPv6  16725      0t0  TCP *:http (LISTEN)
```

ファイアウォールの確認のため、```iptables --list```を実行する。
出力結果は下記の通りで、HTTP(80)のINPUT通信がDROPするようになっている。
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  anywhere             anywhere             tcp dpt:http

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

```iptables --flush```を実行しルールを初期化する。
その後、変更結果確認のため、```iptables --list```を実行する。
出力結果は下記の通りで、HTTP(80)のINPUT通信をDROPするルールがなくなった。
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

状況確認のため、```curl 127.0.0.1:80```で接続テストを実行した。
出力は下記の通りで、パーミッションエラーになっている。
```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.52 (Ubuntu) Server at 127.0.0.1 Port 80</address>
</body></html>
```


```ls -al /var/www/html```で権限を確認する。
出力結果は下記の通りで、index.htmlはroot以外読み取りできなくなっている。
```
total 12
drwxr-xr-x 2 root root 4096 Aug  1  2022 ./
drwxr-xr-x 3 root root 4096 Aug  1  2022 ../
-rw------- 1 root root   16 Aug  1  2022 index.html
```

```chown www-data:www-data -R /var/www/html/```でオーナーを変更する。
ユーザとグループ名は```lsof -i:80```で表示されたプロセスの実行ユーザ名にしている。
変更結果の確認のため```ls -al /var/www/html```を実行すると、下記が出力された。
```
total 12
drwxr-xr-x 2 www-data www-data 4096 Aug  1  2022 ./
drwxr-xr-x 3 root     root     4096 Aug  1  2022 ../
-rw------- 1 www-data www-data   16 Aug  1  2022 index.html
```

```curl 127.0.0.1:80```で接続テストを実行すると、下記の通り出力され、無事に接続できた。
```
hello sadserver
```
