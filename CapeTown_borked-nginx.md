# Borked Nginx


## Scenario
"Cape Town": Borked Nginx


## Level
Medium


## Description
There's an Nginx web server installed and managed by systemd.
Running curl -I 127.0.0.1:80 returns curl: (7) Failed to connect to localhost port 80: Connection refused , fix it so when you curl you get the default Nginx page.


## Description(日本語)
Nginxのウェブサーバーがインストールされていて、systemdで管理されています。
curl -I 127.0.0.1:80 を実行すると curl: (7) Failed to connect to localhost port 80: Connection refused と返ってきます。
curl すると Nginx のデフォルトページが表示されるように直します。


## Test
curl -Is 127.0.0.1:80|head -1 returns HTTP/1.1 200 OK


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答
### 設定ファイルの修正

```
# Nginxに接続できるか確認 → 接続できない
curl -I 127.0.0.1:80

# ファイアウォールに問題ないか確認 → 問題ない
sudo iptables -L
```


```
# サービスが起動しているか確認 → 起動していない
# 「unexpected ";" in /etc/nginx/sites-enabled/default:1」のエラーあり
systemctl status nginx.service
```

以下が期待出力。
```
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Mon 2022-11-28 14:05:51 UTC; 4min 39s ago
    Process: 584 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 31ms

Nov 28 14:05:50 ip-172-31-37-146 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Nov 28 14:05:51 ip-172-31-37-146 nginx[584]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
Nov 28 14:05:51 ip-172-31-37-146 nginx[584]: nginx: configuration file /etc/nginx/nginx.conf test failed
Nov 28 14:05:51 ip-172-31-37-146 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Nov 28 14:05:51 ip-172-31-37-146 systemd[1]: nginx.service: Failed with result 'exit-code'.
Nov 28 14:05:51 ip-172-31-37-146 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
```


```
# エラーの設定ファイルを修正
# /etc/nginx/sites-enabled/defaultの1行目の;をコメントアウト
sudo nano /etc/nginx/sites-enabled/default

# サービスの起動
sudo systemctl start nginx.service
systemctl status nginx.service
```

以下が期待出力。
```
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-11-29 15:03:18 UTC; 7s ago
    Process: 826 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 827 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 828 (nginx)
      Tasks: 2 (limit: 524)
     Memory: 2.4M
        CPU: 54ms
     CGroup: /system.slice/nginx.service
             ├─828 nginx: master process /usr/sbin/nginx
             └─829 nginx: worker process

Nov 29 15:03:18 ip-172-31-33-150 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Nov 29 15:03:18 ip-172-31-33-150 nginx[826]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 29 15:03:18 ip-172-31-33-150 nginx[826]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 29 15:03:18 ip-172-31-33-150 systemd[1]: Started The NGINX HTTP and reverse proxy server.
```


```
# Nginxに接続できるか確認 → 接続できない
curl -I 127.0.0.1:80
```

以下が期待出力。
```
HTTP/1.1 500 Internal Server Error
Server: nginx/1.18.0
Date: Tue, 29 Nov 2022 15:03:43 GMT
Content-Type: text/html
Content-Length: 177
Connection: close
```


### Too many open filesエラーの修正

```
# Nginxのログを確認 → 「Too many open files」のエラーあり
tail /var/log/nginx/error.log
```

以下が期待出力。
```
2022/09/11 16:39:11 [emerg] 5875#5875: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/09/11 16:54:26 [emerg] 5931#5931: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/09/11 16:55:00 [emerg] 5961#5961: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/09/11 17:02:07 [emerg] 6066#6066: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/09/11 17:07:03 [emerg] 6146#6146: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/11/29 15:01:09 [emerg] 575#575: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/11/29 15:03:18 [alert] 828#828: socketpair() failed while spawning "worker process" (24: Too many open files)
2022/11/29 15:03:18 [emerg] 829#829: eventfd() failed (24: Too many open files)
2022/11/29 15:03:18 [alert] 829#829: socketpair() failed (24: Too many open files)
2022/11/29 15:03:43 [crit] 829#829: *1 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "HEAD / HTTP/1.1", host: "127.0.0.1"
```


```
# 該当のPIDを調べる
ps ax | grep nginx | grep worker
# 期待出力
    825 ?        S      0:00 nginx: worker process

# エラーの詳細確認 → 該当プロセスのMax open filesが10になっている
cat /proc/825/limits
```


以下が期待出力。
```
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             1748                 1748                 processes
Max open files            10                   10                   files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       1748                 1748                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```


```
# Nginxの該当設定の確認 → 設定されていない
grep -rl 'worker_rlimit_nofile' /etc/nginx/


# systemdのサービスファイルを確認 → LimitNOFILE=10となっている
cat /etc/systemd/system/nginx.service
```

以下が期待出力。
```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
LimitNOFILE=10

[Install]
WantedBy=multi-user.target
```


```
# 設定変更(LimitNOFILE=10をコメントアウト)し再起動
sudo nano /etc/systemd/system/nginx.service
sudo systemctl daemon-reload
sudo systemctl restart nginx.service


# Nginxに接続できるか確認 → 接続できる
curl -Is 127.0.0.1:80|head -1
# 期待出力
# HTTP/1.1 200 OK
```
