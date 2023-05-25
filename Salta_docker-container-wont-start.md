# Docker container won't start.


## Scenario
"Salta": Docker container won't start.


## Level
Medium


## Type
Fix


## Access
Public


## Description
There's a "dockerized" Node.js web application in the /home/admin/app directory.
Create a Docker container so you get a web app on port :8888 and can curl to it.
For the solution to be valid, there should be only one running Docker container.


## Description(日本語)
home/admin/app ディレクトリに "dockerized" Node.js web アプリケーションがあります。
Docker コンテナを作成し、ポート :8888 でウェブアプリケーションを取得し、それに curl できるようにします。
このソリューションを有効にするには、実行中のDockerコンテナが1つだけである必要があります。


## Test
curl localhost:8888 returns Hello World! from a running container.


## Time to Solve
15 minutes.


## OS
Debian 11


## 回答

### 8888ポートを使っている別プロセスの停止
```bash
# 接続確認
curl localhost:8888
# 出力結果は下記の通りで、別のプロセスが該当ポートを利用している
# these are not the droids you're looking for

# 該当ポートを使用しているプロセスの特定
sudo netstat -pan -A inet,inet6 | grep 8888
# 出力結果は下記の通りで、nginxがポートを使用している
# tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      618/nginx: master p
# tcp        0      0 127.0.0.1:58362         127.0.0.1:8888          TIME_WAIT   -
# tcp6       0      0 :::8888                 :::*                    LISTEN      618/nginx: master p

# nginxがsystemdで管理されているか確認
systemctl status nginx.service
# 出力結果は省略するが、状態はActiveでプロセスIDもnetstatで調べたものと一致した

# nginxの停止
sudo systemctl stop nginx.service

# 停止されたか確認
sudo netstat -pan -A inet,inet6 | grep 8888
# 出力なし
```

### Dockerfileのtypo修正
```bash
# コンテナの起動状態を確認
sudo docker ps -a
# 出力結果は下記の通りで起動中のものはない。
# CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS                    PORTS     NAMES
# 124a4fb17a1c   app       "docker-entrypoint.s…"   3 months ago   Exited (1) 3 months ago             elated_taussig

# イメージのビルド
cd /home/admin/app/
sudo docker build -t sad .
# 出力結果は省略するがエラーなくビルドできた
```

コンテナの起動のため、```sudo docker run --rm -p 8888:8880 sad```を実行した。
なお、ポート番号はDockerfileのEXPOSEを参考に8880にしている。
出力結果は下記の通りで、エラーが発生している。
```
node:internal/modules/cjs/loader:928
  throw err;
  ^

Error: Cannot find module '/usr/src/app/serve.js'
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:925:15)
    at Function.Module._load (node:internal/modules/cjs/loader:769:27)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:76:12)
    at node:internal/main/run_main_module:17:47 {
  code: 'MODULE_NOT_FOUND',
  requireStack: []
}
```

Dockerfileを確認すると```CMD [ "node", "serve.js" ]```という記述があるが、```serve.js```でなく```server.js```のtypoである。
エディタで修正し再度ビルドし、```sudo docker run --rm -p 8888:8880 sad```でコンテナを起動する。
```Server Started on: 8888```と出力され、エラーが発生なくなった。

### Dockerfileのポート修正
```bash
# dockerを起動したウィンドウとは別の新しいウィンドウで実施

# 再度接続確認
curl localhost:8888
# 出力結果
# curl: (56) Recv failure: Connection reset by peer

# コンテナの起動状況確認
sudo docker ps -a
# 出力結果は下記の通りで特に問題は見つからない。
# CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                    PORTS                                       NAMES
# d9104e2769c8   sad       "docker-entrypoint.s…"   52 seconds ago   Up 51 seconds             0.0.0.0:8888->8880/tcp, :::8888->8880/tcp   distracted_goldberg
# 124a4fb17a1c   app       "docker-entrypoint.s…"   8 months ago     Exited (1) 8 months ago                                               elated_taussig

# ポート使用状況の確認
sudo netstat -pan -A inet,inet6 | grep 8888
# 出力結果は下記の通りで特に問題は見つからない
# tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      1165/docker-proxy
# tcp6       0      0 :::8888                 :::*                    LISTEN      1171/docker-proxy
```

アプリケーションスクリプト(/home/admin/app/server.js)を確認すると、```port = process.env.PORT || 8888```という行がある。
これはDockerfileの```EXPOSE 8880```と食い違っている。
Dockerfileを```EXPOSE 8880```から```EXPOSE 8888```にエディタで変更する。
```bash
# 起動中のコンテナを停止
sudo docker stop distracted_goldberg

# Dockerfileの修正
# EXPOSE 8880 -> EXPOSE 8888
nano Dockerfile

# イメージのビルド
sudo docker build -t sad .

# コンテナの起動
sudo docker run --rm -p 8888:8888 sad
# 出力結果
# Server Started on: 8888
```

接続確認のためdockerを起動したウィンドウとは別の新しいウィンドウで```curl localhost:8888```を実行すると期待通り```Hello World!```と出力された。
