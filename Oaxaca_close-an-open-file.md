# Close an Open File


## Scenario
"Oaxaca": Close an Open File


## Level
Medium


## Description
The file /home/admin/somefile is open for writing by some process.
Close this file without killing the process.


## Description(日本語)
home/admin/somefile ファイルは、あるプロセスによって書き込み可能な状態で開かれています。
プロセスを停止させずに、このファイルを閉じてください。


## Test
lsof /home/admin/somefile returns nothing.


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答

```
# オープンしているプロセスの確認
# 77というファイルディスクリプタを使っている
lsof /home/admin/somefile
# 出力結果
# COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
# bash    796 admin   77w   REG  259,1        0 272875 /home/admin/somefile

# オープンしているプロセスのファイルディスクリプタの詳細確認
ls -al /proc/796/fd
# 出力結果
# total 0
# dr-x------ 2 admin admin  0 Feb 15 12:48 .
# dr-xr-xr-x 9 admin admin  0 Feb 15 12:48 ..
# lrwx------ 1 admin admin 64 Feb 15 12:48 0 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 1 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 2 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 255 -> /dev/pts/0
# l-wx------ 1 admin admin 64 Feb 15 12:48 77 -> /home/admin/somefile

# ファイルのクローズ
# http://site.m-bsys.com/linux/redirect-sample
# https://zenn.dev/mom0tomo/articles/1ec9a644daabcf
# https://man7.org/linux/man-pages/man1/exec.1p.html
exec 77>&-

# クローズされたか確認
ls -al /proc/796/fd
# 出力結果
# total 0
# dr-x------ 2 admin admin  0 Feb 15 12:48 .
# dr-xr-xr-x 9 admin admin  0 Feb 15 12:48 ..
# lrwx------ 1 admin admin 64 Feb 15 12:48 0 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 1 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 2 -> /dev/pts/0
# lrwx------ 1 admin admin 64 Feb 15 12:48 255 -> /dev/pts/0

# クローズされたか確認
lsof /home/admin/somefile

# なお~/.bashrcの最終行でファイルを開いている。
# exec 77> /home/admin/somefile
```
