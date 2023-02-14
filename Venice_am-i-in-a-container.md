# Am I in a container?


## Scenario
"Venice": Am I in a container?


## Level
Medium


## Description
Try and figure out if you are inside a container (like a Docker one for example) or inside a Virtual Machine (like in the other scenarios).


## Description(日本語)
コンテナ内（例えばDockerのような）か、仮想マシン内（他のシナリオのような）か、試してみてください。


## Test
This scenario doesn't have a test (hence also no "Check My Solution" either).


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答

```
ps aux
# systemd含め、複数プロセスあり
# 出力結果
# USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root           1  1.1  2.1 100920  9884 ?        Ss   12:57   0:01 /sbin/init
# root          22  0.0  1.5  26728  7024 ?        Ss   12:57   0:00 /lib/systemd/systemd-journald
# message+      47  0.0  0.8   8268  4040 ?        Ss   12:57   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
# root          49  0.0  1.2  13404  5620 ?        Ss   12:57   0:00 /lib/systemd/systemd-logind
# root          71  0.0  2.2 1230092 10444 ?       S<sl 12:57   0:00 /usr/local/gotty --permit-write --reconnect --max-connection 5 bash
# root          83  0.0  0.7   6056  3560 pts/0    S<s  12:58   0:00 bash
# root          85  0.0  0.6   8652  3132 pts/0    R<+  12:58   0:00 ps aux

env
# containerのような環境変数なし
# 出力結果
# SHELL=/bin/sh
# PWD=/
# LOGNAME=root
# HOME=/root
# LANG=C.UTF-8
# INVOCATION_ID=9105e90f9b37488083a2a55b08f3aa50
# TERM=xterm-256color
# USER=root
# SHLVL=1
# JOURNAL_STREAM=8:12269
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# _=/usr/bin/env

cat /proc/1/environ
# containerという文字があるので、コンテナ内
# 出力結果
# PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binTERM=xtermcontainer=youlookedHOME=/rootHOSTNAME=ip-172-31-40-9
```
