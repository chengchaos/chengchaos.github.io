---
title: kubernetes simple notes
key: 2022-06-27
tags: minikube suse
---

Mninkube 是一个构建单节点集群的工具。

<!--more-->

## 安装 Minikube 

Minikube 是一个二进制文件，可以安装在 MAC、Linux 和 Windows 中。

访问这里： [https://github.com/kubernetes/minikube](https://github.com/kubernetes/minikube)

安装： [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

阿里云： [https://developer.aliyun.com/article/221687](https://developer.aliyun.com/article/221687)



### 1 安装 kubectl 

参考： [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl version --client --output=yaml    

```

### 2 安装 minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube

```

### 3 启动 minikube

```bash
minikube start --image-mirror-country='cn'
```

### 4 出现问题就重来

```bash
minikube delete --all
rm -rf ~/.minikube

minikube start \
    --image-mirror-country=cn \
    --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
    --registry-mirror=https://xxx.mirror.aliyuncs.com \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.9.0.iso

```

**一次都没有安装成功， 那就放弃吧。 也不是必须要用这个东东。**



## Kubectl

部署应用程序最简单的方式是使用 `kubectl run` 命令。此命令可以创建必要的组件且无需使用 json 或者 yaml 文件。

### run 一个 pod

```bash
kubectl run <pod-name> \
  --image=<image-name> \
  --port <pord> \

kubectl run my-kubia \
  --image='registry.cn-hangzhou.aliyuncs.com/chengchaos/kubia' \
  --port=8280 \
  --image-pull-policy='Never'
pod/my-kubia created
```



### apply 一个 rc

现在 `--generator=run/v1` 命令已经弃用， k8s文档：https://kubernetes.io/zh/docs/reference/kubectl/conventions/#%E7%94%9F%E6%88%90%E5%99%A8

所以，现在只能用 YAML 文件的方式来启动一个 ReplicationController 了，

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 2
  selector:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
        - name: kubia
          image: registry.cn-hangzhou.aliyuncs.com/chengchaos/kubia
          ports:
            - containerPort: 8280
      imagePullSecrets:
        - name: chengchaos-aliyun
```

### expose 一个 svc

要创建服务，需要告诉 Kubernetes 对外暴露之前创建的 replication controller。

```bash
kubectl expose rc kubia \
  --type=LoadBalancer \
  --name kubia-http

kubectl expose rc kubia --name kubia-http
service/kubia-http exposed
kubectl get svc
NAME                TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
cloud4-company-rc   ClusterIP   10.1.250.58   <none>        10001/TCP   14d
kubernetes          ClusterIP   10.1.0.1      <none>        443/TCP     19d
kubia-http          ClusterIP   10.1.84.96    <none>        8280/TCP    65s
curl http://10.1.84.96:8280
You'v hit kubia-m78ph
```

### scale 一个 rc

```bash
kubectl scale rc kubia --replicas=3
```

## Pod

因为 Pod 是 Kubernetes 中最为重要的核心概念，其他对象都是在管理、暴露 Pod 或者被 Pod 使用。

容器是被设计为每个容器只运行一个进程（除非进程本身产生子进程）。如果在单个容器中运行多个不相关的进程，那么保持所有进程运行、管理它们的日志等将会是我们的责任。

### 使用 YAML 或者 JSON 描述文件创建 Pod

```bash
kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
kubia-rc-88fqp   1/1     Running   0          12m

kubectl get pod kubia-rc-88fqp -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2022-06-28T07:26:18Z"
  generateName: kubia-rc-
  labels:
    app: kubia
# ... 略
```

### 一个简单的 YAML 描述文件



```yaml
apiVersion: v1 # 藐视文件遵循 v1 版本的 kubernetes api
kind: Pod      # 描述一个 Pod
metadata:
	name: kubia-manual # Pod 的名称
spec:
	containers:
  - image: xxx/kubia
  	name: kubia
  	ports:
  	- containerPort: 8280
  		protocol: TCP
```

> 在pod定义中指定端口纯粹是展示性的（informational）。忽略它们对于客户端是否可以通过端口连接到pod不会带来任何影响。如果容器通过绑定到地址0.0.0.0的端口接收连接，那么即使端口未明确列出在podspec中，其他pod也依旧能够连接到该端口。但明确定义端口仍是有意义的，在端口定义下，每个使用集群的人都可以快速查看每个pod对外暴露的端口。此外，我们将在本书的后续内容中看到，明确定义端口还允许你为每个端口指定一个名称，这样一来更加方便我们使用。

### explain 解释属性

```bash
kubectl explain pods
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

### create 创建一个 Pod

```bash
[chengchao@centos701 tmp]$ cat kubia-normal.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: kubia-normal
spec:
  containers:
  - name: kubia-normal
    image: registry.cn-hangzhou.aliyuncs.com/chengchaos/kubia
    ports:
    - containerPort: 8280
    imagePullPolicy: IfNotPresent
[chengchao@centos701 tmp]$ kubectl create -f kubia-normal.yaml 
pod/kubia-normal created
[chengchao@centos701 tmp]$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
kubia-normal     1/1     Running   0          5s
kubia-rc-x6blp   1/1     Running   0          4m29s
```

> `kubectl create -f` 命令用于从 yaml 或 json 文件中创建任何资源。

### logs 查看日志

```bash
[chengchao@centos701 tmp]$ kubectl logs kubia-normal -c kubia-normal -f
Kubia server starting ...

```

如果 pod 中有多个容器， 使用 -c 加名称来指定。

### port-forward 对 pod 进行端口转发

如果想要在不通过 service 的情况下与某个特定的 pod 进行通信（出于调试或其他原因）,Kubernetes 将允许我们配置端口转发到该 pod。可以通过 `kubectl port-forward` 命令完成上述操作。 

```bash
kubectl port-forward kubia-normal 7777:8280
```

### 使用标签（labels）组织 Pod

标签是一种简单却强大的 Kubernetes 特性，可以组织所有 Kubernetes 的资源。

标签是可以附加到资源上的任意键值对。只要标签的 key 在资源内是唯一的， 一个资源便可以拥有多个标签。

- app ： 指定 pod 是属于哪个应用，组件或者微服务。
- rel : 显示在 pod 中运行的应用程序版本是 stable， beta 或者是 canary。

> 金丝雀发布指的是在部署新版本时， 先让一小部分用户体验新版本以观察新版本的表现，然后在决定是否向所有用户进行推广，这样可以防止暴露有问题的版本给过多的用户。

#### 创建 pod 时指定标签

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
spec:
  containers:
  - name: kubia-normal
    image: registry.cn-hangzhou.aliyuncs.com/chengchaos/kubia
    ports:
    - containerPort: 8280
		  protocol: TCP
    imagePullPolicy: IfNotPresent
```



`kubectl get pods` 命令默认不回列出任何标签， 可以使用 `--show-labels` 选项查看

```bash
[chengchao@centos701 tmp]$ kubectl create -f kubia-manual.yaml 
pod/kubia-manual-v2 created
[chengchao@centos701 tmp]$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
kubia-manual-v2   1/1     Running   0          5s
kubia-rc-x6blp    1/1     Running   0          64m
[chengchao@centos701 tmp]$ kubectl get pods --show-labels
NAME              READY   STATUS    RESTARTS   AGE   LABELS
kubia-manual-v2   1/1     Running   0          11s   creation_method=manual,env=prod
kubia-rc-x6blp    1/1     Running   0          64m   app=kubia
[chengchao@centos701 tmp]$ 
[chengchao@centos701 tmp]$ kubectl get pods -L creation_method
NAME              READY   STATUS    RESTARTS   AGE   CREATION_METHOD
kubia-manual-v2   1/1     Running   0          3m    manual
kubia-rc-x6blp    1/1     Running   0          66m   
[chengchao@centos701 tmp]$ kubectl get pods -L creation_method,env
NAME              READY   STATUS    RESTARTS   AGE     CREATION_METHOD   ENV
kubia-manual-v2   1/1     Running   0          3m13s   manual            prod
kubia-rc-x6blp    1/1     Running   0          67m   
```

### label 修改现有 Pod 的标签

```bash
[chengchao@centos701 tmp]$ kubectl label pod kubia-manual-v2 owner=chengchao
pod/kubia-manual-v2 labeled
[chengchao@centos701 tmp]$ kubectl get pods --show-labels
NAME              READY   STATUS    RESTARTS   AGE     LABELS
kubia-manual-v2   1/1     Running   0          6m30s   creation_method=manual,env=prod,owner=chengchao
kubia-rc-x6blp    1/1     Running   0          70m     app=kubia
$ kubectl label pod kubia-manual-v2 owner=chengchao2 --overwrite
pod/kubia-manual-v2 labeled
[chengchao@centos701 tmp]$ kubectl get pods --show-labels
NAME              READY   STATUS    RESTARTS   AGE    LABELS
kubia-manual-v2   1/1     Running   0          103m   creation_method=manual,env=prod,owner=chengchao2
kubia-rc-x6blp    1/1     Running   0          167m   app=kubia
```

### 使用标签选择器列出 Pod

```bash
[chengchao@centos701 tmp]$ kubectl get pods -l owner=chengchao2
NAME              READY   STATUS    RESTARTS   AGE
kubia-manual-v2   1/1     Running   0          105m

[chengchao@centos701 tmp]$ kubectl get pods -l 'env'
NAME              READY   STATUS    RESTARTS   AGE
kubia-manual-v2   1/1     Running   0          105m

[chengchao@centos701 tmp]$ kubectl get pods -l '!env'
NAME             READY   STATUS    RESTARTS   AGE
kubia-rc-x6blp   1/1     Running   0          170m
```

> - `'!env'` 没有 env 的标签
> - ·‘creation_method!=manual’· 带有 creation_method 的标签，但值等于 manual.
> - `‘env in (prod,devel)’` 带有 env 的标签，值是 prod 或 devel 的
> - `'env notin (prod,devel)'` 带有 env 的标签，但是值不是 prod 或 devel 的



### 调度 pod 到一个节点



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - name: kubia-normal
    image: registry.cn-hangzhou.aliyuncs.com/chengchaos/kubia
    ports:
    - containerPort: 8280
    imagePullPolicy: IfNotPresent
```



```bash
kubectl label node centos702.chaos.luxe gpu=true

```

### annotate 添改注解

注解类似 label 也是键值对，注解可以容纳更多信息。

向 Kubernetes 引入新特性时，通常也会使用注解。一般来说，新功能的 alpha 和 beta 版本不会向 API 对象引入任何新字段，因此使用的是注解而不是字段，一旦所需的 API 更改变得清晰并得到所有相关人员的认可，就会引入新的字段并废弃相关注解。

```bash
[chengchao@centos701 tmp]$ kubectl annotate pod kubia-manual-v2 chaos.luxe/sa='foo bar'
pod/kubia-manual-v2 annotated

[chengchao@centos701 tmp]$ kubectl describe pod kubia-manual-v2
Name:         kubia-manual-v2
Namespace:    default
Priority:     0
Node:         centos702.chaos.luxe/192.168.3.202
Start Time:   Tue, 28 Jun 2022 18:51:05 +0800
Labels:       creation_method=manual
              env=prod
Annotations:  chaos.luxe/sa: foo bar
Status:       Running
IP:           10.244.1.9
IPs:
  IP:  10.244.1.9
# 略 ……
```

> chaos.luxe/sa='foo bar' 这个注解前面的 chaos.luxe/ 其实是类似命名空间的一种方式，避免无意间覆盖存在的值。

### namespace 对资源进行分组

Kubernetes 命名空间简单地为对象名称提供了一个作用域。此时我们并不会将所有资源都放在同一个命名空间中，而是将它们组织到多个命名空间中，这样可以允许我们多次使用相同的资源名称（跨不同的命名空间）。

在使用多个 namespace 的前提下，我们可以将包含大量组件的复杂系统拆分为更小的不同组，这些不同组也可以用于在多租户环境中分配资源，将资源分配为生产、开发和QA环境，或者以其他任何你需要的方式分配资源。资源名称只需在命名空间内保持唯一即可，因此两个不同的命名空间可以包含同名的资源。虽然大多数类型的资源都与命名空间相关，但仍有一些与它无关，其中之一便是全局且未被约束于单一命名空间的节点资源。

```bash
# 使用 kubectl get ns 命令列出所有的命名空间
[chengchao@centos701 tmp]$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   4h1m
kube-node-lease   Active   4h1m
kube-public       Active   4h1m
kube-system       Active   4h1m
# 也可以使用 -n 代替 --namespace
[chengchao@centos701 tmp]$ kubectl get po --namespace kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
coredns-7ff77c879f-58r92                       1/1     Running   0          4h2m
coredns-7ff77c879f-xn2rt                       1/1     Running   0          4h2m
etcd-centos701.chaos.luxe                      1/1     Running   0          4h2m
kube-apiserver-centos701.chaos.luxe            1/1     Running   0          4h2m
kube-controller-manager-centos701.chaos.luxe   1/1     Running   0          4h2m
kube-proxy-pg2fc                               1/1     Running   0          4h2m
kube-proxy-rlmxx                               1/1     Running   0          4h2m
kube-proxy-szndx                               1/1     Running   0          4h2m
kube-scheduler-centos701.chaos.luxe            1/1     Running   0          4h2m
```

#### 创建命名空间

```bash
[chengchao@centos701 tmp]$ cat custom-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: custom-namespace

[chengchao@centos701 tmp]$ kubectl create -f custom-namespace.yaml 
namespace/custom-namespace created

[chengchao@centos701 tmp]$ kubectl get ns
NAME               STATUS   AGE
custom-namespace   Active   14s
default            Active   4h5m
kube-node-lease    Active   4h5m
kube-public        Active   4h5m
kube-system        Active   4h5m
# 也可以使用 create namespace 命令创建
[chengchao@centos701 tmp]$ kubectl create namespace custom-namespace2
namespace/custom-namespace2 created
```

> 命名空间不允许包含点 `.` 。

如果像在命名空间中创建资源，可以选择在 metadata 字段中添加一个 `namespace: custom-namespace` 的属性。也可以在使用 `kubectl create` 命令创建资源时指定明明空间

```bash
[chengchao@centos701 tmp]$ kubectl create -f kubia-manual.yaml -n custom-namespace
pod/kubia-manual-v2 created
[chengchao@centos701 tmp]$ kubectl get pods -n custom-namespace
NAME              READY   STATUS    RESTARTS   AGE
kubia-manual-v2   1/1     Running   0          5s

[chengchao@centos701 tmp]$ kubectl port-forward kubia-manual-v2 7777:8280 -n custom-namespace
Forwarding from 127.0.0.1:7777 -> 8280
Forwarding from [::1]:7777 -> 8280
Handling connection for 7777
Handling connection for 7777
Handling connection for 7777
^C
[chengchao@centos701 tmp]$ kubectl port-forward kubia-manual-v2 7777:8280 
Error from server (NotFound): pods "kubia-manual-v2" not found
[chengchao@centos701 tmp]$ 
```

> 尽管命名空间将对象分隔到不同的组，只允许你对属于特定命名空间的对象进行操作，但实际上命名空间之间并不提供对正在运行的对象的任何隔离。
>
> 例如，你可能会认为当不同的用户在不同的命名空间中部署 pod 时，这些 pod 应该彼此隔离，并且无法通信，但事实却并非如此。命名空间之间是否提供网络隔离取决于 Kubernetes 所使用的网络解决方案。当该解决方案不提供命名空间间的网络隔离时，如果命名空间 foo 中的某个 pod 知道命名空间 bar 中 pod 的 IP 地址，那它就可以将流量（例如 HTTP 请求）发送到另一个 pod。

### delete 删除 Pod

```bash
[chengchao@centos701 tmp]$ kubectl delete pod kubia-manual-v2  -n custom-namespace
pod "kubia-manual-v2" deleted
```

> 在删除 pod 的过程中，实际上我们在指示 Kubernetes 终止该 pod 中的所有容器。Kubernetes 向进程发送一个 SIGTERM 信号并等待一定的秒数（默认为30），使其正常关闭。如果它没有及时关闭，则通过 SIGKILL 终止该进程。因此，为了确保你的进程总是正常关闭，进程需要正确处理 SIGTERM 信号。

- 删除多个 pod `kubectl delete pod pod1 pod2`
- 指定标签 `kubectl delete pod -l creation_method=manual`
- 删除金丝雀 `kubectl delete po -l rel=canary`
- 删除所有 pod ： `kubectl delete po --all`

### delete 删除 namespace 

也会删除里面的所有 pod

```bash
kubectl delete namespace custom-namespace2
```

删除所有资源

```bash
kubectl delete all --all
```



---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)

