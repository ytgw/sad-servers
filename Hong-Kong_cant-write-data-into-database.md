# can't write data into database.


## Scenario
"Hong-Kong": can't write data into database.


## Level
Hard


## Type
Fix


## Access
Public


## Description
(Similar to "Manhattan" scenario but harder).
Your objective is to be able to insert a row in an existing Postgres database.
The issue is not specific to Postgres and you don't need to know details about it (although it may help).

Postgres information:
it's a service that listens to a port (:5432) and writes to disk in a data directory, the location of which is defined in the data_directory parameter of the configuration file /etc/postgresql/14/main/postgresql.conf.
In our case Postgres is managed by systemd as a unit with name postgresql.


## Description(日本語)
(マンハッタン」のシナリオに似ているが、より難しい）。
あなたの目的は、既存のPostgresデータベースに行を挿入できるようにすることです。
この問題はPostgresに特化したものではないので、詳細については知る必要はありません（役に立つかもしれませんが）。

Postgresの情報です：
これは、ポート(:5432)をリッスンし、データディレクトリのディスクに書き込むサービスで、その場所は設定ファイル /etc/postgresql/14/main/postgresql.conf の data_directory パラメータで定義します。
この場合、Postgresはsystemdによってpostgresqlという名前のユニットとして管理されています。


## Test
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt

Should return:INSERT 0 1


## Time to Solve
20 minutes.


## OS
Debian 10
```bash

```


## 回答

### postgresqlサービスの状態確認
```bash
# データ挿入の実施
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
# 出力結果
# psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
#         Is the server running locally and accepting connections on that socket?

# サービスの状態確認
systemctl list-units | grep postgresql
# 出力結果
#    postgresql.service                                                       loaded active exited    PostgreSQL RDBMS
# ● postgresql@14-main.service                                               loaded failed failed    PostgreSQL Cluster 14-main
#    system-postgresql.slice                                                  loaded active active    system-postgresql.slice
```

各サービスの状態を確認する。

```systemctl status postgresql```の出力結果は下記の通り。
```
● postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
   Active: active (exited) since Mon 2023-05-01 12:14:44 UTC; 2min 38s ago
  Process: 611 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 611 (code=exited, status=0/SUCCESS)

May 01 12:14:44 ip-172-31-25-11 systemd[1]: Starting PostgreSQL RDBMS...
May 01 12:14:44 ip-172-31-25-11 systemd[1]: Started PostgreSQL RDBMS.
```

```systemctl status postgresql@14-main```の出力結果は下記の通り。
```
● postgresql@14-main.service - PostgreSQL Cluster 14-main
   Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
   Active: failed (Result: protocol) since Mon 2023-05-01 12:14:44 UTC; 2min 54s ago
  Process: 568 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 14-main start (code=exited, status=1/FAILURE)

May 01 12:14:43 ip-172-31-25-11 systemd[1]: Starting PostgreSQL Cluster 14-main...
May 01 12:14:44 ip-172-31-25-11 postgresql@14-main[568]: Error: /opt/pgdata/main is not accessible or does not exist
May 01 12:14:44 ip-172-31-25-11 systemd[1]: postgresql@14-main.service: Can't open PID file /run/postgresql/14-main.pid (yet?) after start: No such file or directory
May 01 12:14:44 ip-172-31-25-11 systemd[1]: postgresql@14-main.service: Failed with result 'protocol'.
May 01 12:14:44 ip-172-31-25-11 systemd[1]: Failed to start PostgreSQL Cluster 14-main.
```

```systemctl status system-postgresql.slice```の出力結果は下記の通り。
```
● system-postgresql.slice
   Loaded: loaded
   Active: active since Mon 2023-05-01 12:14:38 UTC; 3min 24s ago
    Tasks: 0
   Memory: 2.7M
   CGroup: /system.slice/system-postgresql.slice

May 01 12:14:44 ip-172-31-25-11 postgresql@14-main[568]: Error: /opt/pgdata/main is not accessible or does not exist
Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.
```

```ls -al /opt/pgdata/```の出力結果は下記の通り。
```
total 8
drwxr-xr-x 2 postgres postgres 4096 May 21  2022 .
drwxr-xr-x 3 root     root     4096 May 21  2022 ..
```

/opt/pgdata/mainが存在しないことが問題であることがわかった。
ディレクトリを間違って削除したか、デバイスのマウントをしていないかなどが原因の可能性がある。


### ストレージデバイスのマウント

その他エラーが発生していないかの確認のため、```journalctl -p err```を実行。
結果は下記の通りで、/dev/xvdbと接続できていないことがわかる。
```
-- Logs begin at Mon 2023-05-01 12:14:38 UTC, end at Mon 2023-05-01 12:17:01 UTC. --
May 01 12:14:38 ip-172-31-25-11 kernel: ena 0000:00:05.0: LLQ is not supported Fallback to host mode policy.
May 01 12:14:44 ip-172-31-25-11 systemd[1]: Failed to start PostgreSQL Cluster 14-main.
May 01 12:16:09 ip-172-31-25-11 systemd[1]: Timed out waiting for device /dev/xvdb.
```

```df -h```の実行結果は下記の通りで、/dev/xvdbは存在しない。
```
Filesystem       Size  Used Avail Use% Mounted on
udev             224M     0  224M   0% /dev
tmpfs             47M  1.5M   45M   4% /run
/dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
tmpfs            233M     0  233M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            233M     0  233M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
```

```fdisk -l```でディスクの確認した。
結果は下記の通りで、/dev/xvdbはなく、代わりに/dev/nvme0n1が存在していることがわかった。
```
Disk /dev/nvme0n1: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: Amazon Elastic Block Store
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/nvme1n1: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk model: Amazon Elastic Block Store
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 56507B41-5D22-1349-81AD-C628BA074922

Device           Start      End  Sectors  Size Type
/dev/nvme1n1p1  262144 16777182 16515039  7.9G Linux filesystem
/dev/nvme1n1p14   2048     8191     6144    3M BIOS boot
/dev/nvme1n1p15   8192   262143   253952  124M EFI System

Partition table entries are not in disk order.
```

ストレージデバイスのマウント設定は/etc/fstabにあるので、```cat /etc/fstab```で確認。
結果は下記の通りで、存在しない/dev/xvdbを/opt/pgdataにマウントしようとしていることがわかる。
```
# /etc/fstab: static file system information
UUID=5db68868-2d70-449f-8b1d-f3c769ec01c7 / ext4 rw,discard,errors=remount-ro,x-systemd.growfs 0 1
UUID=72C9-F191 /boot/efi vfat defaults 0 0
/dev/xvdb /opt/pgdata xfs defaults,nofail 0 0
```

```bash
# /dev/xvdbから/dev/nvme0n1に変更
nano /etc/fstab

# 設定の反映
systemctl daemon-reload

# デバイスのマウント
mount /dev/nvme0n1 /opt/pgdata
```

```df -h```の実行結果は下記の通りで、無事マウントできたが、使用量が100%になっている。
```
Filesystem       Size  Used Avail Use% Mounted on
udev             224M     0  224M   0% /dev
tmpfs             47M  1.5M   45M   4% /run
/dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
tmpfs            233M     0  233M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            233M     0  233M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
/dev/nvme0n1     8.0G  8.0G   28K 100% /opt/pgdata
```

### ストレージ不足の解消

```bash
# サービスの再起動
systemctl restart postgresql

# データ挿入テストの実行
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
# 出力結果
# psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
#         Is the server running locally and accepting connections on that socket?
```

```systemctl status postgresql```の結果は下記の通り。
```
● postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
   Active: active (exited) since Mon 2023-05-01 12:48:27 UTC; 40s ago
  Process: 1359 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 1359 (code=exited, status=0/SUCCESS)

May 01 12:48:27 ip-172-31-25-11 systemd[1]: Starting PostgreSQL RDBMS...
May 01 12:48:27 ip-172-31-25-11 systemd[1]: Started PostgreSQL RDBMS.
```

```systemctl status postgresql@14-main```の結果は下記の通りで、デバイスの容量が足りない。
```
● postgresql@14-main.service - PostgreSQL Cluster 14-main
   Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
   Active: failed (Result: protocol) since Mon 2023-05-01 12:48:27 UTC; 50s ago
  Process: 1353 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 14-main start (code=exited, status=1/FAILURE)

May 01 12:48:26 ip-172-31-25-11 systemd[1]: Starting PostgreSQL Cluster 14-main...
May 01 12:48:27 ip-172-31-25-11 postgresql@14-main[1353]: Error: /usr/lib/postgresql/14/bin/pg_ctl /usr/lib/postgresql/14/bin/pg_ctl start -D /opt/pgdata/main -l /var/log/postgresql/postgresql-14-main.log -s -o  -c config_file="/etc/po
May 01 12:48:27 ip-172-31-25-11 postgresql@14-main[1353]: 2023-05-01 12:48:26.992 UTC [1358] FATAL:  could not create lock file "postmaster.pid": No space left on device
May 01 12:48:27 ip-172-31-25-11 postgresql@14-main[1353]: pg_ctl: could not start server
May 01 12:48:27 ip-172-31-25-11 postgresql@14-main[1353]: Examine the log output.
May 01 12:48:27 ip-172-31-25-11 systemd[1]: postgresql@14-main.service: Can't open PID file /run/postgresql/14-main.pid (yet?) after start: No such file or directory
May 01 12:48:27 ip-172-31-25-11 systemd[1]: postgresql@14-main.service: Failed with result 'protocol'.
May 01 12:48:27 ip-172-31-25-11 systemd[1]: Failed to start PostgreSQL Cluster 14-main.
```

```df -h```の実行結果は下記の通りで、使用量が100%になっている。
```
Filesystem       Size  Used Avail Use% Mounted on
udev             224M     0  224M   0% /dev
tmpfs             47M  1.5M   45M   4% /run
/dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
tmpfs            233M     0  233M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            233M     0  233M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
/dev/nvme0n1     8.0G  8.0G   28K 100% /opt/pgdata
```

```ls -al /opt/pgdata/```の結果は下記の通りで、バックアップファイルのサイズが大きい。
```
total 8285624
drwxr-xr-x  3 postgres postgres         82 May 21  2022 .
drwxr-xr-x  3 root     root           4096 May 21  2022 ..
-rw-r--r--  1 root     root             69 May 21  2022 deleteme
-rw-r--r--  1 root     root     7516192768 May 21  2022 file1.bk
-rw-r--r--  1 root     root      967774208 May 21  2022 file2.bk
-rw-r--r--  1 root     root         499712 May 21  2022 file3.bk
drwx------ 19 postgres postgres       4096 May 21  2022 main
```

```rm /opt/pgdata/file*.bk```で削除後に、```df -h```を実行。
結果は下記の通りで容量不足が解消した。
```
Filesystem       Size  Used Avail Use% Mounted on
udev             224M     0  224M   0% /dev
tmpfs             47M  1.5M   45M   4% /run
/dev/nvme1n1p1   7.7G  1.2G  6.1G  17% /
tmpfs            233M     0  233M   0% /dev/shm
tmpfs            5.0M     0  5.0M   0% /run/lock
tmpfs            233M     0  233M   0% /sys/fs/cgroup
/dev/nvme1n1p15  124M  278K  124M   1% /boot/efi
/dev/nvme0n1     8.0G   91M  8.0G   2% /opt/pgdata
```

```bash
# サービスの再起動
systemctl restart postgresql

# データ挿入
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
# 出力結果
# INSERT 0 1
```
