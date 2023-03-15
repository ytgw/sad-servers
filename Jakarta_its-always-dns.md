# it's always DNS.


## Scenario
"Jakarta": it's always DNS.


## Level
Hard


## Type
Fix


## Access
Public


## Description
Can't ping google.com.
It returns ping: google.com: Name or service not known.
Expected is being able to resolve the hostname.
(Note: currently the VMs can't ping outside so there's no automated check for the solution).


## Description(日本語)
google.comにpingを打てない。
ping:google.comを返します。名前またはサービスが不明です。
期待されるのは、ホスト名を解決できることです。
(注：現在、VMは外部にpingを打てないので、解決のための自動チェックはありません)。


## Test
ping google.com should return something like PING google.com (172.217.2.46) 56(84) bytes of data.


## Time to Solve
20 minutes.


## OS
Ubuntu 22.04 LTS


## 回答

### DNSの確認
```bash
# check ping
ping google.com
# 出力結果
# ping: google.com: Name or service not known

# check dig
dig google.com
```

出力結果は下記の通りで、DNSからIPアドレスを取得できている。
```
; <<>> DiG 9.18.1-1ubuntu1.1-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30476
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             102     IN      A       142.250.190.46

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Wed Mar 15 12:21:17 UTC 2023
;; MSG SIZE  rcvd: 55
```

### /etc/nsswitch.confの編集

```bash
# pingは失敗するがdigの名前解決はできる場合の対処法
# https://tex2e.github.io/blog/linux/nsswitch
cat /etc/nsswitch.conf
```

結果は下記の通り。

```
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

```bash
# /etc/nsswitch.confの編集
# hostsの行の最後にdnsと追記する。
sudo nano /etc/nsswitch.conf

# 編集結果の確認
cat /etc/nsswitch.conf
```

結果は下記の通り。

```
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

```bash
# pingの確認
ping google.com
# 出力結果は下記の通りで、このVMでは外との通信はできない。
# PING google.com (142.250.190.46) 56(84) bytes of data.
# ^C
# --- google.com ping statistics ---
# 2 packets transmitted, 0 received, 100% packet loss, time 1016ms
```
