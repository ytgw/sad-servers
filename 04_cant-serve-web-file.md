# can't serve web file


## Scenario
"Tokyo": can't serve web file


## Level
Medium


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

```
# 状況確認
cat /var/www/html/index.html
# 期待出力
# hello sadserver

curl 127.0.0.1:80
# 何も出力されない

lsof -i:80
# 期待出力
# COMMAND PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# apache2 626     root    4u  IPv6  16966      0t0  TCP *:http (LISTEN)
# apache2 774 www-data    4u  IPv6  16966      0t0  TCP *:http (LISTEN)
# apache2 775 www-data    4u  IPv6  16966      0t0  TCP *:http (LISTEN)


# ファイアウォール
iptables -L
# 期待出力
# Chain INPUT (policy ACCEPT)
# target     prot opt source               destination
# DROP       tcp  --  anywhere             anywhere             tcp dpt:http
#
# Chain FORWARD (policy ACCEPT)
# target     prot opt source               destination
#
# Chain OUTPUT (policy ACCEPT)
# target     prot opt source               destination

iptables -I INPUT -p tcp --dport 80 -j ACCEPT
# iptables --flush
# iptables -P INPUT DROP  # これをするとコンソールにアクセスできなくなる
# iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# iptables -A INPUT -i lo -j ACCEPT
# iptables -A INPUT -p tcp --dport 80 -j ACCEPT

iptables -L
# 期待出力
# Chain INPUT (policy ACCEPT)
# target     prot opt source               destination
# ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
# DROP       tcp  --  anywhere             anywhere             tcp dpt:http
#
# Chain FORWARD (policy ACCEPT)
# target     prot opt source               destination
#
# Chain OUTPUT (policy ACCEPT)
# target     prot opt source               destination


# 状況確認
curl 127.0.0.1:80
# 期待出力
# <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
# <html><head>
# <title>403 Forbidden</title>
# </head><body>
# <h1>Forbidden</h1>
# <p>You don't have permission to access this resource.</p>
# <hr>
# <address>Apache/2.4.52 (Ubuntu) Server at 127.0.0.1 Port 80</address>
# </body></html>


# 権限確認
ls -l /var/www/html/index.html
# 期待出力
# -rw------- 1 root root 16 Aug  1 00:40 /var/www/html/index.html

chown www-data:www-data /var/www/html/index.html
ls -l /var/www/html/index.html
# 期待出力
# -rw------- 1 www-data www-data 16 Aug  1 00:40 /var/www/html/index.html


# 状況確認
curl 127.0.0.1:80
# hello sadserver
```
