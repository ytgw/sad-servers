# etcd SSL cert troubles


## Scenario
"Lisbon": etcd SSL cert troubles


## Level
Medium


## Type
Fix


## Description
There's an etcd server running on https://localhost:2379, get the value for the key "foo", ie etcdctl get foo or curl https://localhost:2379/v2/keys/foo


## Description(日本語)
https://localhost:2379 で etcd サーバーが動作しています。
キー "foo" の値を取得するには、etcdctl get foo または curl https://localhost:2379/v2/keys/foo を実行します。


## Test
etcdctl get foo returns bar.


## Time to Solve
20 minutes.


## OS
Debian 11


## 回答

### 時刻の修正
```bash
# テストの実行
etcdctl get foo
# 出力結果は以下の通りで、現時刻が未来の時刻になっている。
# Error:  client: etcd cluster is unavailable or misconfigured; error #0: x509: certificate has expired or is not yet valid: current time 2024-03-09T11:30:39Z is after 2023-01-30T00:02:48Z
#
# error #0: x509: certificate has expired or is not yet valid: current time 2024-03-09T11:30:39Z is after 2023-01-30T00:02:48Z

# 時刻の確認
date
# 出力結果
# Sat Mar  9 11:31:38 UTC 2024

# 時刻の変更
sudo date --set '2023-01-01T00:00:00Z'
# 出力結果
# Sun Jan  1 00:00:00 UTC 2023

# 再度テスト
etcdctl get foo
# 出力結果
# Error:  client: response is invalid json. The endpoint is probably not valid etcd cluster endpoint.

# curlでテスト
curl https://localhost:2379/v2/keys/foo
# 出力結果
# <html>
# <head><title>404 Not Found</title></head>
# <body>
# <center><h1>404 Not Found</h1></center>
# <hr><center>nginx/1.18.0</center>
# </body>
# </html>
```

期待するサービスに接続されていない。

### NATの修正
```bash
# プロセスの確認(出力は省略。プロセスやサービスは起動済み)
ps auxf | grep etcd
systemctl status etcd.service

# ポートの確認
sudo netstat -pan -A inet,inet6 | grep -i listen
```

出力結果は以下の通りで、etcdは2379をリッスンしており、nginxは別ポートをリッスンしている。
```
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      578/etcd
tcp        0      0 127.0.0.1:2380          0.0.0.0:*               LISTEN      578/etcd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      606/sshd: /usr/sbin
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      610/nginx: master p
tcp6       0      0 :::6767                 :::*                    LISTEN      572/sadagent
tcp6       0      0 :::8080                 :::*                    LISTEN      571/gotty
tcp6       0      0 :::22                   :::*                    LISTEN      606/sshd: /usr/sbin
```

```bash
# NATの確認
sudo iptables -t nat -L --line-number
```

出力結果は以下の通りで、2379ポートを443に転送している。
```
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination
1    DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
1    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:2379 redir ports 443
2    DOCKER     all  --  anywhere            !ip-127-0-0-0.us-east-2.compute.internal/8  ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination
1    MASQUERADE  all  --  ip-172-17-0-0.us-east-2.compute.internal/16  anywhere

Chain DOCKER (2 references)
num  target     prot opt source               destination
1    RETURN     all  --  anywhere             anywhere
```

```bash
# NATルールの削除
sudo iptables -t nat -D OUTPUT 1

# NATルールの確認
sudo iptables -t nat -L --line-number
```

出力結果は以下の通り。
```
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination
1    DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
1    DOCKER     all  --  anywhere            !ip-127-0-0-0.us-east-2.compute.internal/8  ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination
1    MASQUERADE  all  --  ip-172-17-0-0.us-east-2.compute.internal/16  anywhere

Chain DOCKER (2 references)
num  target     prot opt source               destination
1    RETURN     all  --  anywhere             anywhere
```

```bash
# テスト
etcdctl get foo
# 出力結果
# bar

# curlでテスト
curl https://localhost:2379/v2/keys/foo
# 出力結果
# {"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
```
