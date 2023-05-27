# sad-servers
Troubleshoot and make a sad server happy!

[SadServers - Troubleshooting Linux Servers](https://sadservers.com/)の回答を記載するリポジトリ。


## シナリオ一覧

| # | シナリオ名 | 成功か |
| :-- | :-- | :-- |
| 1 | ["Saint John": what is writing to this log file?](SaintJohn_what-is-writing-to-this-log-file.md) | o |
| 2 | ["Saskatoon": counting IPs.](Saskatoon_counting-ips.md) | o |
| 3 | ["Santiago": Find the secret combination](Santiago_find-the-secret-combination.md) | o |
| 4 | ["Manhattan": can't write data into database.](Manhattan_cant-write-data-into-database) | o |
| 5 | ["Tokyo": can't serve web file](Tokyo_cant-serve-web-file.md) | o |
| 6 | ["Cape Town": Borked Nginx](CapeTown_borked-nginx.md) | o |
| 7 | ["Salta": Docker container won't start.](Salta_docker-container-wont-start.md) | o |
| 8 | ["Venice": Am I in a container?](Venice_am-i-in-a-container.md) | o |
| 9 | ["Oaxaca": Close an Open File](Oaxaca_close-an-open-file.md) |  |
| 10 | ["Melbourne": WSGI with Gunicorn](Melbourne_wsgi-with-gunicorn.md) |  |
| 11 | ["Lisbon": etcd SSL cert troubles](Lisbon_etcd-ssl-cert-troubles.md) |  |
| 12 | "Kihei": Surely Not Another Disk Space Scenario |  |
| 13 | ["Jakarta": it's always DNS.](Jakarta_its-always-dns.md) |  |
| 14 | ["Bern": Docker web container can't connect to db container.](Bern_docker-web-container-cant-connect-to-db-container.md) |  |
| 15 | ["Karakorum": WTFIT – What The Fun Is This?](Karakorum_wtfit–what-the-fun-is-this.md) |  |
| 16 | ["Singara": Docker and Kubernetes web app not working.](Singara_docker-and-kubernetes-web-app-not-working.md) |  |
| 17 | ["Hong-Kong": can't write data into database.](Hong-Kong_cant-write-data-into-database.md) |  |
| 18 | ["Pokhara": SSH and other sshenanigans](Pokhara_ssh-and-other-sshenanigans.md) |  |
| 19 | ["Roseau": Hack a Web Server](Roseau_hack-a-web-server.md) |  |
| 20 | "Belo-Horizonte": A Java Enigma |  |


## General Instructions
You have full (root) access to a real Linux server (an ephemeral Virtual Machine) via an SSH session.
Do whatever is necessary to fix the problem described so that a test passes within the alloted time.
After that time the VM will be terminated.

As a constraint, from the servers you cannot go out to the Internet.
DNS is available locally.
Also do not interfere with services unrelated to the issue described that are needed for SadServers to work properly, specifically the services running on ports :2020 and :6767.

If you are stuck or not sure what to do, you can click on the "Next Clue / Solution" button and it will display a new hint (and the previous ones) until it reveals the solution.
Close the clue window and click "Next Clue / Solution" again to get a new clue.

Once you think you have fixed the issue, click on the "Check My Solution" button to verify it as per the given Test.


## General Instructions(機械翻訳)
あなたは、実際の Linux サーバー（一時的な仮想マシン）に SSH セッションでフルアクセス（root）することができます。
割り当てられた時間内にテストに合格するように、説明されている問題を修正するために必要なことを何でもしてください。
時間が経過すると、VMは終了します。

制約として、このサーバからインターネットに出ることはできません。
DNSはローカルで利用可能です。
また、SadServersが正常に動作するために必要な、説明した問題とは関係のないサービス、特にポート2020とポート6767で動作しているサービスを妨害しないでください。

行き詰まったときや、どうすればいいかわからないときは、「次のヒント/解決策」ボタンをクリックすると、解決策がわかるまで新しいヒント（と前のヒント）が表示されます。
ヒントウィンドウを閉じて、もう一度「Next Clue / Solution」をクリックすると、新しいヒントが表示されます。

問題が解決したと思ったら、「Check My Solution」ボタンをクリックし、与えられたテスト通りに解決したかどうかを確認します。
