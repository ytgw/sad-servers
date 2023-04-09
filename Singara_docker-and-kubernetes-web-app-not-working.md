# Docker and Kubernetes web app not working.


## Scenario
"Singara": Docker and Kubernetes web app not working.


## Level
Hard


## Type
Fix


## Access
Public


## Description
There's a k3s Kubernetes install you can access with kubectl.
The Kubernetes YAML manifests under /home/admin have been applied.
The objective is to access from the host the "webapp" web server deployed and find what message it serves (it's a name of a town or city btw).
In order to pass the check, the webapp Docker container should not be run separately outside Kubernetes as a shortcut.


## Description(日本語)
kubectlでアクセスできるk3s Kubernetesのインストールがあります。
home/admin以下にKubernetesのYAMLマニフェストが適用されています。
目的は、ホストからデプロイされた「webapp」Webサーバーにアクセスし、それがどんなメッセージを提供しているかを調べることです（町名や都市名ですbtw）。
チェックをパスするためには、webappのDockerコンテナをショートカットとしてKubernetesの外で別途実行してはいけない。


## Test
curl localhost:8888 returns a value from the webapp deployed Kubernetes pod.


## Time to Solve
20 minutes.


## OS
Debian 11


## 回答

### 現状確認

```bash
# 接続テスト
curl localhost:8888
# 出力結果
# curl: (7) Failed to connect to localhost port 8888: Connection refused
```

```sudo netstat -pan -A inet,inet6 | grep -i listen```の出力結果は下記の通りで、8888をリッスンしているプロセスはない。
```txt
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:6444          0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:10256         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:10258         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      612/k3s server
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      607/sshd: /usr/sbin
tcp        0      0 127.0.0.1:10010         0.0.0.0:*               LISTEN      834/containerd
tcp6       0      0 :::10250                :::*                    LISTEN      612/k3s server
tcp6       0      0 :::6443                 :::*                    LISTEN      612/k3s server
tcp6       0      0 :::6767                 :::*                    LISTEN      577/sadagent
tcp6       0      0 :::8080                 :::*                    LISTEN      576/gotty
tcp6       0      0 :::22                   :::*                    LISTEN      607/sshd: /usr/sbin
```

Kubernetes YAML manifestsの確認結果は以下の通り。
```bash
cd /home/admin/

ls
# 出力結果
# agent  deployment.yml  namespace.yml  nodeport.yml
```

deployment.ymlの内容は以下の通り。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: web
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp
        imagePullPolicy: Always
        ports:
        - containerPort: 8880
```

namespace.ymlの内容は以下の通り。
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: web
```

nodeport.ymlの内容は以下の通り。
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: web
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: webapp
  ports:
    - port: 80
      targetPort: 8888
      nodePort: 30007
```


### kubectlでの確認

```kubectl --help```の出力結果の抜粋は以下の通り。

```txt
kubectl controls the Kubernetes cluster manager.

...

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources

...

Troubleshooting and Debugging Commands:
  describe        Show details of a specific resource or group of resources
  logs            Print the logs for a container in a pod
  attach          Attach to a running container
  exec            Execute a command in a container
  port-forward    Forward one or more local ports to a pod

...

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

```kubectl get -h```の出力結果の抜粋は以下の通り。

```txt
Display one or many resources.

...

Examples:
  # List all pods in ps output format
  kubectl get pods

...

Options:
    -A, --all-namespaces=false:
        If present, list the requested object(s) across all namespaces. Namespace in current context is ignored even
        if specified with --namespace.

...

Usage:
  kubectl get
[(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns|custom-columns-file|wide]
(TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

```kubectl get pods --all-namespaces```の出力結果は以下の通り。

```txt
NAMESPACE     NAME                                      READY   STATUS             RESTARTS        AGE
kube-system   helm-install-traefik-crd-nml28            0/1     Completed          0               203d
kube-system   helm-install-traefik-z54r4                0/1     Completed          2               203d
web           webapp-deployment-666b67994b-5sffz        0/1     ImagePullBackOff   0               203d
kube-system   coredns-b96499967-scfhc                   1/1     Running            7 (203d ago)    203d
kube-system   local-path-provisioner-7b7dc8d6f5-r8777   1/1     Running            7 (203d ago)    203d
kube-system   metrics-server-668d979685-gkzrn           1/1     Running            11 (203d ago)   203d
kube-system   svclb-traefik-7cfc151c-hqm5p              2/2     Running            8 (203d ago)    203d
kube-system   traefik-7cd4fcff68-g8t6k                  1/1     Running            7 (203d ago)    203d
kube-system   svclb-traefik-7cfc151c-m6842              2/2     Running            2 (109s ago)    202d
```

```kubectl options```を実行すると、namespaceを指定する方法が記載されている。
```kubectl -n web get all```の出力結果は以下の通り。

```txt
NAME                                     READY   STATUS             RESTARTS   AGE
pod/webapp-deployment-666b67994b-5sffz   0/1     ImagePullBackOff   0          203d

NAME                     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/webapp-service   NodePort   10.43.35.97   <none>        80:30007/TCP   203d

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-deployment   0/1     1            0           203d

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-deployment-666b67994b   1         1         0       203d
```

```kubectl -n web describe all```の出力結果の抜粋は以下の通り。

```txt
...

Events:
  Type     Reason          Age                  From               Message
  ----     ------          ----                 ----               -------
  Normal   Scheduled       203d                 default-scheduler  Successfully assigned web/webapp-deployment-666b67994b-5sffz to ip-10-0-0-17
  Normal   Pulling         203d (x4 over 203d)  kubelet            Pulling image "webapp"
  Warning  Failed          203d (x4 over 203d)  kubelet            Failed to pull image "webapp": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/webapp:latest": failed to resolve reference "docker.io/library/webapp:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed          203d (x4 over 203d)  kubelet            Error: ErrImagePull
  Warning  Failed          203d (x6 over 203d)  kubelet            Error: ImagePullBackOff
  Normal   BackOff         203d (x7 over 203d)  kubelet            Back-off pulling image "webapp"
  Normal   SandboxChanged  203d                 kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   BackOff         203d (x6 over 203d)  kubelet            Back-off pulling image "webapp"
  Warning  Failed          203d (x6 over 203d)  kubelet            Error: ImagePullBackOff
  Normal   Pulling         203d (x4 over 203d)  kubelet            Pulling image "webapp"
  Warning  Failed          203d (x4 over 203d)  kubelet            Failed to pull image "webapp": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/webapp:latest": failed to resolve reference "docker.io/library/webapp:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed          203d (x4 over 203d)  kubelet            Error: ErrImagePull

...
```

imageをpullできずエラーになっていることがわかった。


### image pullエラーの修正

```bash
sudo docker pull webapp
# 出力結果は下記の通りで、手動でpullしても失敗する。
# Using default tag: latest
# Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

ping docker.io
# 出力結果は下記の通りで、Sad Serversは外と通信できない。
# PING docker.io (52.3.144.121) 56(84) bytes of data.
# ^C
# --- docker.io ping statistics ---
# 4 packets transmitted, 0 received, 100% packet loss, time 3061ms

sudo docker image ls
# 出力結果は下記の通りで、docker image registry用のイメージがある。
# REPOSITORY   TAG        IMAGE ID       CREATED        SIZE
# webapp       latest     9c082e2983bc   6 months ago   135MB
# python       3.7-slim   c1d0bab51bbf   6 months ago   123MB
# registry     2          3a0f7b0a13ef   8 months ago   24.1MB

# docker image registryを起動。
# 5000というポートはdocker image inspect registry:2で見つけられる。
sudo docker run -d -p 5000:5000 registry:2

# タグをつける
sudo docker tag webapp localhost:5000/webapp
sudo docker image ls
# REPOSITORY              TAG        IMAGE ID       CREATED        SIZE
# webapp                  latest     9c082e2983bc   6 months ago   135MB
# localhost:5000/webapp   latest     9c082e2983bc   6 months ago   135MB
# python                  3.7-slim   c1d0bab51bbf   6 months ago   123MB
# registry                2          3a0f7b0a13ef   8 months ago   24.1MB

# imageの登録(push)
sudo docker push localhost:5000/webapp

# デプロイmanifestsの修正
nano deployment.yml
# 以下は修正内容抜粋
# before
# image: webapp
# after
# image: localhost:5000/webapp

# 修正後のmanifestsの適用
kubectl apply -f deployment.yml
```

```kubectl -n web get all```の出力結果は以下の通り。
```txt
NAME                                     READY   STATUS        RESTARTS   AGE
pod/webapp-deployment-666b67994b-5sffz   0/1     Terminating   0          203d
pod/webapp-deployment-5b98dcc989-2hbc6   1/1     Running       0          23s

NAME                     TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/webapp-service   NodePort   10.43.35.97   <none>        80:30007/TCP   203d

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-deployment   1/1     1            1           203d

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-deployment-5b98dcc989   1         1         1       23s
replicaset.apps/webapp-deployment-666b67994b   0         0         0       203d
```

### コンテナポート番号の修正

```bash
# デプロイmanifestの修正
nano deployment.yml
# 以下は修正内容抜粋
# before
# containerPort: 8880
# after
# containerPort: 8888

# 修正後のmanifestsの適用
kubectl apply -f deployment.yml

# ポートフォワーディングの修正
kubectl -n web port-forward deployments/webapp-deployment 8888
# 出力結果は以下の通り
# Forwarding from 127.0.0.1:8888 -> 8888
# Forwarding from [::1]:8888 -> 8888


# 接続確認(別ウィンドウで実施する)
curl localhost:8888
# 出力結果は以下の通り
# Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch
```
