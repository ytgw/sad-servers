# can't write data into database.


## Scenario
"Manhattan": can't write data into database.


## Level
Level: Medium


## Description
Your objective is to be able to insert a row in an existing Postgres database.
The issue is not specific to Postgres and you don't need to know details about it (although it may help).

Helpful Postgres information: it's a service that listens to a port (:5432) and writes to disk in a data directory, the location of which is defined in the data_directory parameter of the configuration file /etc/postgresql/14/main/postgresql.conf.
In our case Postgres is managed by systemd as a unit with name postgresql.


## Description(日本語)
あなたの目的は、既存の Postgres データベースに行を挿入できるようにすることです。
この問題はPostgresに特有なものではないので、それについて詳細を知る必要はありません（役に立つかもしれませんが）。

Postgresはポート(:5432)をリッスンし、データディレクトリのディスクに書き込むサービスです。その場所は、設定ファイル/etc/postgresql/14/main/postgresql.confのdata_directoryパラメータで定義されています。
この例では、Postgresはpostgresqlという名前のユニットとしてsystemdによって管理されています。


## Test
(from default admin user) sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt

Should return:INSERT 0 1


## Time to Solve
20 minutes.


## OS
Debian 10


## 回答

```
# データを挿入できないことを確かめる
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
# 期待出力
# psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
#         Is the server running locally and accepting connections on that socket?


# 該当のサービスの状態を調べる
systemctl status postgresql
# 期待出力
# ● postgresql.service - PostgreSQL RDBMS
#    Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
#    Active: active (exited) since Sun 2022-11-27 16:40:03 UTC; 3min 50s ago
#   Process: 692 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
#  Main PID: 692 (code=exited, status=0/SUCCESS)
#
# Nov 27 16:40:03 ip-172-31-37-97 systemd[1]: Starting PostgreSQL RDBMS...
# Nov 27 16:40:03 ip-172-31-37-97 systemd[1]: Started PostgreSQL RDBMS.

# ExecStart=/bin/trueとなっており起動スクリプトが違うのが気になる
# サービスが起動しているように見える


# 該当のサービスのログを調べる
journalctl -u postgresql
# 期待出力
# -- Logs begin at Sun 2022-11-27 17:28:38 UTC, end at Sun 2022-11-27 17:31:03 UTC. --
# Nov 27 17:29:33 ip-172-31-41-116 systemd[1]: Starting PostgreSQL RDBMS...
# Nov 27 17:29:33 ip-172-31-41-116 systemd[1]: Started PostgreSQL RDBMS.

# エラーが無いように見える


# 関連するエラーがないかシステムログを調べる
cat /var/log/syslog | grep postgresql
# 期待出力
# Nov 27 17:31:03 ip-172-31-41-116 postgresql@14-main[852]: Error: /usr/lib/postgresql/14/bin/pg_ctl /usr/lib/postgresql/14/bin/pg_ctl start -D /opt/pgdata/main -l /var/log/postgresql/postgresql-14-main.log -s -o  -c config_file="/etc/postgresql/14/main/postgresql.conf"  exited with status 1:
# Nov 27 17:31:03 ip-172-31-41-116 postgresql@14-main[852]: 2022-11-27 17:31:03.786 UTC [857] FATAL:  could not create lock file "postmaster.pid": No space left on device
# Nov 27 17:31:03 ip-172-31-41-116 postgresql@14-main[852]: pg_ctl: could not start server
# Nov 27 17:31:03 ip-172-31-41-116 postgresql@14-main[852]: Examine the log output.
# Nov 27 17:31:03 ip-172-31-41-116 systemd[1]: postgresql@14-main.service: Can't open PID file /run/postgresql/14-main.pid (yet?) after start: No such file or directory
# Nov 27 17:31:03 ip-172-31-41-116 systemd[1]: postgresql@14-main.service: Failed with result 'protocol'.

# No space left on deviceというエラーがある


# ディスク使用可能領域を調べる
df -h
# 期待出力
# Filesystem       Size  Used Avail Use% Mounted on
# udev             224M     0  224M   0% /dev
# tmpfs             47M  1.5M   46M   4% /run
# /dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
# tmpfs            233M     0  233M   0% /dev/shm
# tmpfs            5.0M     0  5.0M   0% /run/lock
# tmpfs            233M     0  233M   0% /sys/fs/cgroup
# /dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
# /dev/nvme0n1     8.0G  8.0G   28K 100% /opt/pgdata

# /opt/pgdataのディスク使用率が100%になっている


# 該当のディレクトリを調べる
ls -al /opt/pgdata/
# 期待出力
# total 8285624
# drwxr-xr-x  3 postgres postgres         82 May 21  2022 .
# drwxr-xr-x  3 root     root           4096 May 21  2022 ..
# -rw-r--r--  1 root     root             69 May 21  2022 deleteme
# -rw-r--r--  1 root     root     7516192768 May 21  2022 file1.bk
# -rw-r--r--  1 root     root      967774208 May 21  2022 file2.bk
# -rw-r--r--  1 root     root         499712 May 21  2022 file3.bk
# drwx------ 19 postgres postgres       4096 May 21  2022 main


# バックアップファイルを削除する
rm /opt/pgdata/file*.bk
ls -al /opt/pgdata/
# 期待出力
# total 12
# drwxr-xr-x  3 postgres postgres   34 Nov 27 17:35 .
# drwxr-xr-x  3 root     root     4096 May 21  2022 ..
# -rw-r--r--  1 root     root       69 May 21  2022 deleteme
# drwx------ 19 postgres postgres 4096 May 21  2022 main


# ディスク使用可能領域が増えたか調べる
df -h
# 期待出力
# Filesystem       Size  Used Avail Use% Mounted on
# udev             224M     0  224M   0% /dev
# tmpfs             47M  1.5M   46M   4% /run
# /dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
# tmpfs            233M     0  233M   0% /dev/shm
# tmpfs            5.0M     0  5.0M   0% /run/lock
# tmpfs            233M     0  233M   0% /sys/fs/cgroup
# /dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
# /dev/nvme0n1     8.0G   91M  8.0G   2% /opt/pgdata


# サービスを起動して状態を調べる
systemctl start postgresql
systemctl status postgresql
# 期待出力
# ● postgresql.service - PostgreSQL RDBMS
#    Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
#    Active: active (exited) since Sun 2022-11-27 17:29:33 UTC; 7min ago
#   Process: 646 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
#  Main PID: 646 (code=exited, status=0/SUCCESS)
#
# Nov 27 17:29:33 ip-172-31-41-116 systemd[1]: Starting PostgreSQL RDBMS...
# Nov 27 17:29:33 ip-172-31-41-116 systemd[1]: Started PostgreSQL RDBMS.


# データが挿入できるか調べる
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
# 期待出力
# INSERT 0 1
```


PostgreSQLのsystemdのサービスファイル(/lib/systemd/system/postgresql.service)は下記になっており、ExecStartやExecReloadがtrueコマンドになっていて不適切な気がするが、これで問題なく動いていることが気になる。

```
# systemd service for managing all PostgreSQL clusters on the system. This
# service is actually a systemd target, but we are using a service since
# targets cannot be reloaded.

[Unit]
Description=PostgreSQL RDBMS

[Service]
Type=oneshot
ExecStart=/bin/true
ExecReload=/bin/true
RemainAfterExit=on

[Install]
WantedBy=multi-user.target
```
