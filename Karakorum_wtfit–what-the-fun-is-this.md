# WTFIT – What The Fun Is This?


## Scenario
"Karakorum": WTFIT – What The Fun Is This?


## Level
Hard


## Type
Fix


## Access
Public


## Description
There's a binary at /home/admin/wtfit that nobody knows how it works or what it does ("what the fun is this").
Someone remembers something about wtfit needing to communicate to a service in order to start.
Run this wtfit program so it doesn't exit with an error, fixing or working around things that you need but are broken in this server.
(Note that you can open more than one web "terminal").


## Description(日本語)
/home/admin/wtfitにバイナリがあるのですが、これがどう動くのか、何をするのか、誰も知りません（「これは何が面白いんだ」）。
誰かが、wtfitが起動するためにサービスと通信する必要があることを何か覚えている。
このwtfitプログラムがエラーで終了しないように実行し、必要だけどこのサーバーでは壊れているものを修正したり、回避したりします。
(複数のウェブ「ターミナル」を開くことができることに注意してください)。


## Test
Running /home/admin/wtfit returns OK.


## Time to Solve
20 minutes.


## OS
Debian 11


## 回答

### バイナリファイルへの実行権限付与

```bash
# お試しの実行
/home/admin/wtfit
# 出力結果
# bash: /home/admin/wtfit: Permission denied

# 実行権限確認
ls -l /home/admin/wtfit
# 出力結果
# -rw-r--r-- 1 admin admin 6381234 Sep 13  2022 /home/admin/wtfit

# 実行権限付与
chmod u+x /home/admin/wtfit
# 出力結果
# bash: /usr/bin/chmod: Permission denied

# chmodの実行権限確認
ls -l /usr/bin/chmod
# 出力結果
# -rw-r--r-- 1 root root 64448 Sep 24  2020 /usr/bin/chmod

# chmod自身に実行権限を付与
sudo python3 -c "import os; os.chmod('/usr/bin/chmod', 0o755)"
ls -l /usr/bin/chmod
# 出力結果
# -rwxr-xr-x 1 root root 64448 Sep 24  2020 /usr/bin/chmod

# バイナリファイルへの実行権限付与
chmod u+x /home/admin/wtfit

# バイナリファイルの実行
/home/admin/wtfit
# 出力結果
# ERROR: can't open config file
```

### 読み込みファイルの作成

```bash
# バイナリファイルの実行
/home/admin/wtfit
# 出力結果
# ERROR: can't open config file

# 開こうとしているファイルの確認
strace -f /home/admin/wtfit 2>&1 | grep open
# 出力結果
# openat(AT_FDCWD, "/sys/kernel/mm/transparent_hugepage/hpage_pmd_size", O_RDONLY) = 3
# [pid   868] openat(AT_FDCWD, "/home/admin/wtfitconfig.conf", O_RDONLY|O_CLOEXEC <unfinished ...>
# [pid   868] <... openat resumed>)       = -1 ENOENT (No such file or directory)
# [pid   868] write(1, "ERROR: can't open config file\n", 30ERROR: can't open config file

# ファイルの作成
touch /home/admin/wtfitconfig.conf

# バイナリファイルの実行
/home/admin/wtfit
# 出力結果
# ERROR: can't connect to server
```

### 通信先サーバーの起動

```bash
# 接続しようとしているサーバーの確認
strace -f /home/admin/wtfit 2>&1 | grep connect
# 出力結果
# [pid   884] connect(3, {sa_family=AF_INET, sin_port=htons(7777), sin_addr=inet_addr("127.0.0.1")}, 16 <unfinished ...>
# [pid   884] <... connect resumed>)      = -1 EINPROGRESS (Operation now in progress)
# [pid   884] write(1, "ERROR: can't connect to server\n", 31 <unfinished ...>
# [pid   886] nanosleep({tv_sec=0, tv_nsec=3000}, ERROR: can't connect to server

# リッスンポートの確認
sudo netstat -pan -A inet,inet6 | grep -i listen
# 出力結果は省略するが7777をリッスンしているプロセスはない

# 通信内容の確認
sudo tcpdump -i lo port 7777
# 別ターミナルでバイナリファイルを実行時の出力結果抜粋
# 09:13:14.576549 IP localhost.36212 > localhost.7777: Flags [S], seq 1280819725, win 65495, options [mss 65495,sackOK,TS val 3674072197 ecr 0,nop,wscale 6], length 0
# 09:13:14.576557 IP localhost.7777 > localhost.36212: Flags [R.], seq 0, ack 1280819726, win 0, length 0

# TCPサーバーの起動
nc -l -p 7777
# 別ターミナルでバイナリファイルを実行する
# 出力結果は以下の通りで、HTTPリクエストを送っている
# GET / HTTP/1.1
# Host: localhost:7777
# User-Agent: Go-http-client/1.1
# Accept-Encoding: gzip
#
# ^C

# HTTPサーバーの起動
python3 -m http.server 7777
# 別ターミナルでバイナリファイルを実行する
# 出力結果
# Serving HTTP on 0.0.0.0 port 7777 (http://0.0.0.0:7777/) ...
# 127.0.0.1 - - [24/Mar/2023 09:25:30] "GET / HTTP/1.1" 200 -

# バイナリファイルの実行
/home/admin/wtfit
# 出力結果
# OK
```
