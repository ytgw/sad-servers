# Docker container won't start.


## Scenario
"Salta": Docker container won't start.


## Level
Medium


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
```
curl localhost:8888  # these are not the droids you're looking for

sudo lsof -i:8888  # sudo: lsof: command not found
sudo netstat -apn | grep 8888
# 出力例
# tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      600/nginx: master p
# tcp        0      0 127.0.0.1:33752         127.0.0.1:8888          TIME_WAIT   -
# tcp6       0      0 :::8888                 :::*                    LISTEN      600/nginx: master p

sudo kill 600
sudo netstat -apn | grep 8888
# 出力例
# tcp        0      0 127.0.0.1:33752         127.0.0.1:8888          TIME_WAIT   -
```

### Dockerのtypo修正
```
sudo docker ps -a
# 出力例
# CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS                    PORTS     NAMES
# 124a4fb17a1c   app       "docker-entrypoint.s…"   3 months ago   Exited (1) 3 months ago             elated_taussig

sudo docker logs elated_taussig  # Error: Cannot find module '/usr/src/app/serve.js'
sudo docker rm elated_taussig
sudo docker image ls
# 出力例
# REPOSITORY   TAG           IMAGE ID       CREATED         SIZE
# app          latest        1d782b86d6f2   3 months ago    124MB
# node         15.7-alpine   706d12284dd5   22 months ago   110MB

cd /home/admin/app/
ls
cat Dockerfile

# Dockerfileの修正
# serve.js -> server.js
nano Dockerfile

sudo docker build -t app .
sudo docker run --rm --name app -p 8888:8880 -d app
curl localhost:8888  # curl: (56) Recv failure: Connection reset by peer
sudo docker logs app # Server Started on: 8888
sudo docker kill app
```


### Dockerfileのポート修正
```
cat server.js  # port = process.env.PORT || 8888

# Dockerfileの修正
# EXPOSE 8880 -> EXPOSE 8888
nano Dockerfile

sudo docker build -t app .
sudo docker run --rm --name app -p 8888:8888 -d app

# 結果確認
curl localhost:8888  # Hello World!
```
