# counting IPs.


## Scenario
"Saskatoon": counting IPs.


## Level
Easy


## Type
Do


## Access
Public


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

```tail /home/admin/access.log```でログファイルを確認する。
出力結果は下記の通り。
```
66.249.73.135 - - [20/May/2015:21:05:11 +0000] "GET /blog/tags/xsendevent HTTP/1.1" 200 10049 "-" "Mozilla/5.0 (iPhone; CPU iPhone OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A5376e Safari/8536.25 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
198.46.149.143 - - [20/May/2015:21:05:29 +0000] "GET /blog/geekery/disabling-battery-in-ubuntu-vms.html?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+semicomplete%2Fmain+%28semicomplete.com+-+Jordan+Sissel%29 HTTP/1.1" 200 9316 "-" "Tiny Tiny RSS/1.11 (http://tt-rss.org/)"
198.46.149.143 - - [20/May/2015:21:05:34 +0000] "GET /blog/geekery/solving-good-or-bad-problems.html?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+semicomplete%2Fmain+%28semicomplete.com+-+Jordan+Sissel%29 HTTP/1.1" 200 10756 "-" "Tiny Tiny RSS/1.11 (http://tt-rss.org/)"
82.165.139.53 - - [20/May/2015:21:05:15 +0000] "GET /projects/xdotool/ HTTP/1.0" 200 12292 "-" "-"
100.43.83.137 - - [20/May/2015:21:05:01 +0000] "GET /blog/tags/standards HTTP/1.1" 200 13358 "-" "Mozilla/5.0 (compatible; YandexBot/3.0; +http://yandex.com/bots)"
63.140.98.80 - - [20/May/2015:21:05:28 +0000] "GET /blog/tags/puppet?flav=rss20 HTTP/1.1" 200 14872 "http://www.semicomplete.com/blog/tags/puppet?flav=rss20" "Tiny Tiny RSS/1.11 (http://tt-rss.org/)"
63.140.98.80 - - [20/May/2015:21:05:50 +0000] "GET /blog/geekery/solving-good-or-bad-problems.html?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+semicomplete%2Fmain+%28semicomplete.com+-+Jordan+Sissel%29 HTTP/1.1" 200 10756 "-" "Tiny Tiny RSS/1.11 (http://tt-rss.org/)"
66.249.73.135 - - [20/May/2015:21:05:00 +0000] "GET /?flav=atom HTTP/1.1" 200 32352 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"
180.76.6.56 - - [20/May/2015:21:05:56 +0000] "GET /robots.txt HTTP/1.1" 200 - "-" "Mozilla/5.0 (Windows NT 5.1; rv:6.0.2) Gecko/20100101 Firefox/6.0.2"
46.105.14.53 - - [20/May/2015:21:05:15 +0000] "GET /blog/tags/puppet?flav=rss20 HTTP/1.1" 200 14872 "-" "UniversalFeedParser/4.2-pre-314-svn +http://feedparser.org/"
```

```bash
# スペースで区切って1列目(IPアドレス)を表示する
cut --fields=1 --delimiter=' ' /home/admin/access.log
# 下記は出力結果抜粋
# 66.249.73.135
# 180.76.6.56
# 46.105.14.53

# uniqで重複行をカウントするが、その前処理にソートしないと連続していない重複行を同一のものと扱ってくれない
cut --fields=1 --delimiter=' ' /home/admin/access.log | sort | uniq --count
# 下記は出力結果抜粋
#      26 99.252.100.83
#       9 99.33.244.41
#       6 99.6.61.4

# カウント結果をソート
cut --fields=1 --delimiter=' ' /home/admin/access.log | sort | uniq --count | sort
# 下記は出力結果抜粋
#     273 75.97.9.59
#     357 130.237.218.86
#     364 46.105.14.53
#     482 66.249.73.135

# 結果の書き込み
echo "66.249.73.135" > /home/admin/highestip.txt
```
