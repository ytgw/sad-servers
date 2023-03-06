# WSGI with Gunicorn


## Scenario
"Melbourne": WSGI with Gunicorn


## Level
Medium


## Type
Fix


## Description
There is a Python WSGI web application file at /home/admin/wsgi.py, the purpose of which is to serve the string "Hello, world!".
This file is served by a Gunicorn server which is fronted by an nginx server (both servers managed by systemd).
So the flow of an HTTP request is: Web Client (curl) -> Nginx -> Gunicorn -> wsgi.py.
The objective is to be able to curl the localhost (on default port :80) and get back "Hello, world!", using the current setup.


## Description(日本語)
Python WSGI Web アプリケーションファイルが /home/admin/wsgi.py にあり、その目的は "Hello, world!" という文字列を提供することである。
このファイルは、nginxサーバによってフロントされるGunicornサーバによって提供されます（両方のサーバはsystemdによって管理されます）。
つまり、HTTPリクエストの流れはこうだ。Web Client (curl) -> Nginx -> Gunicorn -> wsgi.py.
目標は、現在の設定で、localhost (デフォルトのポート :80) をcurlして "Hello, world!" を返せるようにすることです。


## Test
curl -s http://localhost returns Hello, world! (serving the wsgi.py file via Gunicorn and Nginx)


## Time to Solve
20 minutes.


## OS
Debian 11


## 回答

### ポートのリッスン状態の確認
```bash
# テストの実行
curl http://localhost
# 出力結果
# curl: (7) Failed to connect to localhost port 80: Connection refused
```

```bash
# ポートのリッスン状態の確認
netstat -pan -A inet,inet6 | grep -i listen
```

以下は出力結果だが、80でリッスンしているNginxがいない。
```
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::6767                 :::*                    LISTEN      602/sadagent
tcp6       0      0 :::8080                 :::*                    LISTEN      601/gotty
tcp6       0      0 :::22                   :::*                    LISTEN      -
```


### Nginxの起動
```bash
# Nginxの状態確認
systemctl status nginx.service
```

以下は出力結果。
```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)
```

```bash
# Nginxの起動
sudo systemctl start nginx.service
systemctl status nginx.service
```

以下は出力結果。
```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-03-06 14:48:12 UTC; 4s ago
       Docs: man:nginx(8)
    Process: 910 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 911 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 912 (nginx)
      Tasks: 3 (limit: 524)
     Memory: 11.1M
        CPU: 35ms
     CGroup: /system.slice/nginx.service
             ├─912 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─913 nginx: worker process
             └─914 nginx: worker process

Mar 06 14:48:08 ip-172-31-38-220 systemd[1]: Starting A high performance web server and a reverse proxy server...
Mar 06 14:48:12 ip-172-31-38-220 systemd[1]: Started A high performance web server and a reverse proxy server.
```

```bash
# テストの実行
curl http://localhost
```

以下は出力結果。
```html
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
```


### Nginxの設定変更
```bash
# configの確認
less /etc/nginx/nginx.conf
```

以下は出力結果の抜粋。
```
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```

```bash
ls /etc/nginx/conf.d/
# 出力なし

ls /etc/nginx/sites-enabled/
# 出力結果
# default

# 設定確認
less /etc/nginx/sites-enabled/default
```

以下は出力結果。
```
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.socket;
    }
}
```

```bash
# socketファイルの確認
ps auxf | grep gunicorn
admin        952  0.0  0.1   5268   704 pts/0    S<+  14:51   0:00      \_ grep gunicorn
admin        609  0.0  4.7  27480 22024 ?        Ss   14:45   0:00 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
admin        679  0.0  4.1  27560 19400 ?        S    14:45   0:00  \_ /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi

# サービスの確認
systemctl status gunicorn.service
```

以下は出力結果。
```
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-03-06 14:45:27 UTC; 8min ago
TriggeredBy: ● gunicorn.socket
   Main PID: 609 (gunicorn)
      Tasks: 2 (limit: 524)
     Memory: 17.1M
        CPU: 333ms
     CGroup: /system.slice/gunicorn.service
             ├─609 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
             └─679 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi

Mar 06 14:45:27 ip-172-31-38-220 systemd[1]: Started gunicorn daemon.
Mar 06 14:45:28 ip-172-31-38-220 gunicorn[609]: [2023-03-06 14:45:28 +0000] [609] [INFO] Starting gunicorn 20.1.0
Mar 06 14:45:28 ip-172-31-38-220 gunicorn[609]: [2023-03-06 14:45:28 +0000] [609] [INFO] Listening at: unix:/run/gunicorn.sock (60>
Mar 06 14:45:28 ip-172-31-38-220 gunicorn[609]: [2023-03-06 14:45:28 +0000] [609] [INFO] Using worker: sync
Mar 06 14:45:28 ip-172-31-38-220 gunicorn[679]: [2023-03-06 14:45:28 +0000] [679] [INFO] Booting worker with pid: 679
```

```bash
# 設定ファイルの変更
# http://unix:/run/gunicorn.socket を http://unix:/run/gunicorn.sock に変更
sudo nano /etc/nginx/sites-enabled/default
less /etc/nginx/sites-enabled/default
```

以下は出力結果。
```
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

```bash
# Nginxの再起動
sudo systemctl restart nginx.service

# テスト
curl http://localhost
# 何も返ってこない

# socketでのテスト
curl --unix-socket /run/gunicorn.sock http://localhost
# 何も返ってこない
```


### アプリケーションの修正
```bash
# gunicornサービスの確認
less /etc/systemd/system/gunicorn.service
```

以下は出力結果。
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=admin
Group=admin
WorkingDirectory=/home/admin
ExecStart=/usr/local/bin/gunicorn \
          --bind unix:/run/gunicorn.sock \
          wsgi
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
# アプリの確認
less /home/admin/wsgi.py
```

以下は出力結果。
```
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ('Content-Length', '0'), ])
    return [b'Hello, world!']

```

```bash
# HTTPヘッダーのContent-Lengthが0になっているものを修正。
nano /home/admin/wsgi.py
less /home/admin/wsgi.py
```

以下は出力結果。
```
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ])
    return [b'Hello, world!']

```

```bash
# gunicornサービスの再起動
sudo systemctl restart gunicorn.service

# socketでのテスト
curl --unix-socket /run/gunicorn.sock http://localhost
Hello, world!

# テスト
curl -s http://localhost
Hello, world!
```
