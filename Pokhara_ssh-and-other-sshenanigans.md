# SSH and other sshenanigans


## Scenario
"Pokhara": SSH and other sshenanigans


## Level
Hard


## Type
Fix


## Access
Public


## Description
A user client was added to the server, as well as their SSH public key.
The objective is to be able to SSH locally (there's only one server) as this user client using their ssh keys.
This is, if as root you change to this user sudo su; su client, you should be able to login with ssh: ssh localhost.


## Description(日本語)
サーバーにユーザークライアントと、そのSSH公開鍵が追加されました。
目的は、このユーザークライアントとして、彼らのsshキーを使ってローカルにSSHできるようにすることです（サーバーは1台しかありません）。
つまり、rootとしてこのユーザー sudo su; su clientに変更すると、ssh: ssh localhostでログインできるようになるはずです。


## Test
As user admin: sudo -u client ssh client@localhost 'pwd' returns /home/client


## Time to Solve
30 minutes.


## OS
Debian 11


## 回答

### known_hostsの修正

テストコマンド```sudo -u client ssh client@localhost 'pwd'```を実行した。
結果は下記の通りで、SSHサーバーのホストキーが変更されていることがわかった。
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:YW7npS3A8AKzO8qieal8Dk0yGtJlH/cvJ+Q60pQjjj0.
Please contact your system administrator.
Add correct host key in /home/client/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/client/.ssh/known_hosts:1
  remove with:
  ssh-keygen -f "/home/client/.ssh/known_hosts" -R "localhost"
ECDSA host key for localhost has changed and you have requested strict checking.
Host key verification failed.
```

下記コマンドで保存されているホストキーを削除する。

```bash
sudo -u client ssh-keygen -f "/home/client/.ssh/known_hosts" -R "localhost"
```

### SSHサーバーのDenyUsersリスト設定の修正

再度テストコマンド```sudo -u client ssh client@localhost 'pwd'```を実行した。
```client@localhost: Permission denied (publickey).```という結果だった。

```sudo -u client ssh -vvv client@localhost 'pwd'```で詳細結果を見ても分からなかった。

そのため、```sudo systemctl status sshd```でサーバーの状態を確認した。
結果の抜粋は下記の通りで、DenyUsersリストに含まれていることが分かった。
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-05-02 05:58:00 UTC; 8min ago

...

May 02 06:06:08 ip-172-31-34-210 sshd[946]: User client from 127.0.0.1 not allowed because listed in DenyUsers
May 02 06:06:08 ip-172-31-34-210 sshd[946]: Connection closed by invalid user client 127.0.0.1 port 52278 [preauth]
```

下記の通り、/etc/ssh/sshd_configおよび/etc/ssh/sshd_config.d/*.confを確認および修正した。
```bash
ls -al /etc/ssh/

# /etc/ssh/sshd_configの確認したが怪しい点はなし
less /etc/ssh/sshd_config

# /etc/ssh/sshd_config.d/*.confの確認したところ、sad.confというファイルを発見
ls -al /etc/ssh/sshd_config.d/

# /etc/ssh/sshd_config.d/sad.confの確認
cat /etc/ssh/sshd_config.d/sad.conf
# 出力結果は下記の通りでclientがDenyUsersリストに含まれている
# DenyUsers client

sudo rm /etc/ssh/sshd_config.d/sad.conf

sudo systemctl restart sshd
```

### 秘密鍵のパーミッションの修正

```sudo -u client ssh client@localhost 'pwd'```を実行した。
結果は下記の通りで、秘密鍵のパーミッションが不適切で、他のユーザーから見えるようになっている。
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/client/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/home/client/.ssh/id_rsa": bad permissions
client@localhost: Permission denied (publickey).
```

```sudo ls -al /home/client/.ssh/```の結果は以下の通りで、オーナー以外も読み取り権限がついている。
```
total 24
drwx------ 2 client client 4096 May  2 05:59 .
drwxr-xr-x 4 client client 4096 Feb  5 22:49 ..
-rw------- 1 client client  569 Feb  6 00:21 authorized_keys
-rw-r--r-- 1 client client 2610 Feb  5 22:33 id_rsa
-rw-r--r-- 1 client client  569 Feb  5 22:49 id_rsa.pub
-rw-r--r-- 1 client client  222 May  2 05:59 known_hosts
```

```sudo chmod 600 /home/client/.ssh/id_rsa```で修正。

### アカウント有効期限の修正

SSH接続を試すと下記結果となった。
```
Your account has expired; please contact your system administrator.
Connection closed by 127.0.0.1 port 22
```

[こちら](https://www.kakistamp.com/entry/2018/08/07/013647)を参考に修正。

```sudo chage --list client```の結果は下記の通りで1970年が有効期限になっている。
```
Last password change                                    : Feb 05, 2023
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Jan 01, 1970
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

```sudo chage -E -1 client```で修正後に、再度```sudo chage --list client```で確認。
```
Last password change                                    : Feb 05, 2023
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

### プロセス数の上限修正

SSH接続を実行すると```exec request failed on channel 0```という表示が出た。

[こちら](https://linux.just4fun.biz/?Linux%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A/shell+request+failed+on+channel+0+%E3%81%AE%E5%AF%BE%E5%BF%9C%E6%96%B9%E6%B3%95)を参考に修正する。

```sudo -u client ulimit -u```を実行すると```sudo: ulimit: command not found```となりclientでは実行できなかった。

```less /etc/security/limits.conf```で上限を確認した。
結果の抜粋は下記の通りでプロセス数の上限が0になっている。
```
client          hard    nproc           0
```

該当する行をコメントアウトする。

### ログインシェルの修正

SSH接続を実行すると```This account is currently not available.```という表示が出た。

[こちら](https://qiita.com/riekure/items/27e07258a5a3ac4bd3fa)を参考にする。

/etc/passwdを確認すると、client:x:1001:1001::/home/client:/usr/sbin/nologinという行がある。
こちらをclient:x:1001:1001::/home/client:/bin/bashに変更する。

SSH接続をテストを実行すると下記の通りを接続できた。
```bash
sudo -u client ssh client@localhost 'pwd'
# 出力結果
# /home/client
```
