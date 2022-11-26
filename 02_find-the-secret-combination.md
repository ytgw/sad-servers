# Find the secret combination


## Scenario
"Santiago": Find the secret combination


## Level
Easy


## Description
Alice the spy has hidden a secret number combination, find it using these instructions:

1. Find the number of lines where the string Alice occurs in *.txt files in the /home/admin directory
1. There's a file where "Alice" appears exactly once.
In the line after that ocurrence there's a number.

Write both numbers consecutively as one (no new line or spaces) to the solution file (eg if the first number from 1) is 11 and the second 22, you can do echo -n 11 > /home/admin/solution; echo 22 >> /home/admin/solution).


## Test
Running md5sum /home/admin/solution returns d80e026d18a57b56bddf1d99a8a491f9(just a way to verify the solution without giving it away.)


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答

```
# Aliceを含むテキストファイルを見つける
grep 'Alice' --count /home/admin/*.txt
# 期待出力
# /home/admin/11-0.txt:398
# /home/admin/1342-0.txt:1
# /home/admin/1661-0.txt:12
# /home/admin/84-0.txt:0

# 合計行数のカウント
# https://qiita.com/tofu511/items/3ecf9c5361d08b5c6eae
grep 'Alice' --count --no-filename /home/admin/*.txt | awk 'BEGIN { sum = 0 } { sum += $0 } END { print sum }'
# 期待出力
# 411

# Aliceが1度だけ書かれているファイルをlessで表示
# /Alice(現在より後方の検索)もしくは?Alice(現在より前方を検索)と入力
# Aliceの次の行にある数字が156とわかる
less /home/admin/1342-0.txt

# 回答ファイルに出力
echo -n 411 > /home/admin/solution
echo 156 >> /home/admin/solution
md5sum /home/admin/solution
```
