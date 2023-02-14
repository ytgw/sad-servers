# what is writing to this log file?

## Scenario
"Saint John": what is writing to this log file?

## Level
Easy

## Description
A developer created a testing program that is continuously writing to a log file /var/log/bad.log and filling up disk.
You can check for example with tail -f /var/log/bad.log.
This program is no longer needed. Find it and terminate it.

## Test
The log file hasn't changed in the last 6 seconds: find /var/log/bad.log -mmin -0.1 (You don't need to know the details of this command).

## Time to Solve
10 minutes.

## OS
Ubuntu 22.04 LTS

## 回答
lsofを使って該当ファイルに書き込んでいるプロセスを特定し、それを止める。

```
$ tail -f /var/log/bad.log
2022-11-20 08:15:21.480677 token: 1167384895
2022-11-20 08:15:22.481925 token: 1389063089
2022-11-20 08:15:23.483059 token: 748719531
2022-11-20 08:15:24.484194 token: 1679048409
2022-11-20 08:15:25.486096 token: 75959157
2022-11-20 08:15:26.487245 token: 675139284
2022-11-20 08:15:27.488394 token: 412421226
2022-11-20 08:15:28.489534 token: 778789483
2022-11-20 08:15:29.490663 token: 406853619
2022-11-20 08:15:30.492809 token: 1136151369
2022-11-20 08:15:31.494009 token: 1740370725
2022-11-20 08:15:32.495203 token: 2122296816
2022-11-20 08:15:33.496392 token: 1164478634
2022-11-20 08:15:34.497574 token: 426091271
^C

$ lsof /var/log/bad.log
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
badlog.py 595 ubuntu    3w   REG  259,1    13085 67701 /var/log/bad.log

$ kill 595
$ lsof /var/log/bad.log
$ find /var/log/bad.log -mmin -0.1
```
