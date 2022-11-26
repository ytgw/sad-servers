# counting IPs.


## Scenario
"Saskatoon": counting IPs.


## Level
Easy


## Description
There's a web server access log file at /home/admin/access.log.
The file consists of one line per HTTP request, with the requester's IP address at the beginning of each line.

Find what's the IP address that has the most requests in this file (there's no tie; the IP is unique).
Write the solution into a file /home/admin/highestip.txt.
For example, if your solution is "1.2.3.4", you can do echo "1.2.3.4" > /home/admin/highestip.txt


## Description(日本語)
Webサーバーのアクセスログファイルは、/home/admin/access.logにあります。
このファイルは、HTTPリクエストごとに1行で構成されており、各行の先頭にリクエスト元のIPアドレスが記載されています。

このファイルの中で最も多くのリクエストを持つIPアドレスは何かを見つけなさい(同点はない。IPは一意である)。
その解答をファイル /home/admin/highestip.txt に書き込んでください。
たとえば、あなたの解決策が "1.2.3.4" であれば、```echo "1.2.3.4" > /home/admin/highestip.txt```とします。


## Test
The SHA1 checksum of the IP address sha1sum /home/admin/highestip.txt is 6ef426c40652babc0d081d438b9f353709008e93 (just a way to verify the solution without giving it away.)


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答

```
less /home/admin/access.log
# 出力例
# 83.149.9.216 - - [17/May/2015:10:05:03 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
# 83.149.9.216 - - [17/May/2015:10:05:43 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-dashboard3.png HTTP/1.1" 200 171717 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
# 83.149.9.216 - - [17/May/2015:10:05:47 +0000] "GET /presentations/logstash-monitorama-2013/plugin/highlight/highlight.js HTTP/1.1" 200 26185 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
# 83.149.9.216 - - [17/May/2015:10:05:12 +0000] "GET /presentations/logstash-monitorama-2013/plugin/zoom-js/zoom.js HTTP/1.1" 200 7697 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"

# スペースで区切って1列目(IPアドレス)を表示する
cut --fields=1 --delimiter=' ' /home/admin/access.log
# 出力例
# 66.249.73.135
# 180.76.6.56
# 46.105.14.53

# uniqで重複行をカウントするが、その前処理にソートしないと連続していない重複行を同一のものと扱ってくれない
cut --fields=1 --delimiter=' ' /home/admin/access.log | sort | uniq --count
# 出力例
#      26 99.252.100.83
#       9 99.33.244.41
#       6 99.6.61.4

# カウント結果をソート
cut --fields=1 --delimiter=' ' /home/admin/access.log | sort | uniq --count | sort
# 出力例
#     273 75.97.9.59
#     357 130.237.218.86
#     364 46.105.14.53
#     482 66.249.73.135
```
