---

title: CentOS7 安装 Kubernetes1.18.1 并使用 Flannel
key: 2021-08-15
tags: kubernetes k8s kubeadm flannel
---

原文: [https://www.cnblogs.com/xiao987334176/p/12696740.html](https://www.cnblogs.com/xiao987334176/p/12696740.html)

手工搭建 Kubernetes 集群是一件很繁琐的事情，为了简化这些操作，就产生了很多安装配置工具，如 Kubeadm ，Kubespray，RKE 等组件，我最终选择了官方的 Kubeadm 主要是不同的 Kubernetes 版本都有一些差异，Kubeadm 更新与支持的会好一些。Kubeadm 是 Kubernetes 官方提供的快速安装和初始化 Kubernetes 集群的工具，目前的还处于孵化开发状态，跟随 Kubernetes 每个新版本的发布都会同步更新, 强烈建议先看下官方的文档了解下各个组件与对象的作用。

<!--more-->

- [https://kubernetes.io/docs/concepts/](https://kubernetes.io/docs/concepts/)
- [https://kubernetes.io/docs/setup/independent/install-kubeadm/](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
- [https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)

在创建 Kubernetes 集群时，阿里云容器服务提供两种网络插件：Terway 和 Flannel。

- Flannel：使用的是简单稳定的社区的Flannel CNI 插件，配合阿里云的VPC的高速网络，能给集群高性能和稳定的容器网络体验，但功能偏简单，支持的特性少，例如：不支持基于Kubernetes标准的Network Policy。
- Terway：是阿里云容器服务自研的网络插件，将阿里云的弹性网卡分配给容器，支持基于Kubernetes标准的NetworkPolicy来定义容器间的访问策略，支持对单个容器做带宽的限流。对于不需要使用Network Policy的用户，可以选择Flannel，其他情况建议选择 Terway。

因此，本文主要介绍 Flannel 的简单使用。

## 准备工作

> **注意** 必须在所有机器上执行

### 关闭防火墙

如果各个主机启用了防火墙，需要开放Kubernetes各个组件所需要的端口，可以查看Installing kubeadm中的”Check required ports”一节。 这里简单起见在各节点禁用防火墙：

```bash
systemctl stop firewalld
systemctl disable firewalld
```

### 禁用SELINUX

```bash
# 临时禁用
setenforce 0
# 永久禁用 
# 或者修改/etc/sysconfig/selinux
vim /etc/selinux/config
SELINUX=disabled
```

### 系统配置

```bash
sodu -i

cd /etc/sysctl.d/
touch k8s.conf

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

```

### 开启路由转发

```bash
cat /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
```

### 关闭swap

```bash
# 临时关闭
swapoff -a

sudo vim /etc/fstab
# 修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载（永久关闭swap，重启后生效）
# 注释掉以下字段
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

### 安装docker

> 参考 [https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl enable docker
sudo systemctl start docker 

sudo -i
cd /etc/docker
touch daemon.json
cat > daemon.json<<EOF
{
 "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://hub-mirror.c.163.com"
  ],
  "iptables" : false
}

EOF


```

### 修改主机名

```bash
hostnamectl set-hostname k8s-master
```

注意：主机名不能带下划线，只能带中划线
否则安装k8s会报错

> could not convert cfg to an internal cfg: nodeRegistration.name: Invalid value: "k8s_master": a DNS-1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*')

## 安装 kubeadm, kubelet, kubectl



> master, node1, node2 之上都必须执行！

### 修改 yum 安装源

```bash

sudo -i

cd /etc/yum.repos.d/
touch kubernetes.repo

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum -y update

## 此时只能安装 1.18.1, 
## 因为其他的版本, 就下载不下来相关的组件了.
## 所以前的安装的又特么的删掉了.
# yum -y install kubelet kubeadm kubectl 
# systemctl enable kubelet && systemctl start kubelet

# systemctl disable kubelet && systemctl stop kubelet
# yum -y remove kubelet kubeadm kubectl 

yum install -y kubelet-1.18.1-0 
yum install -y kubectl-1.18.1-0
yum install -y kubeadm-1.18.1-0 
## 
systemctl enable kubelet && systemctl start kubelet

```

> 以上就是 master 节点和 node 节点都要执行的操作！



### 初始化 Master 节点

> 登录到 master 主机上，运行初始化命令
>
> 注意： 修改 apiserver-advertise-address 为 master 节点的 IP

```bash
kubeadm init --kubernetes-version=1.18.1 \
--apiserver-advertise-address=192.168.3.201 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

... running with swap on is not supported. Please disable swap
swapoff -a



#### 
kubeadm reset 就可以解决这种问题

kubeadm init --kubernetes-version=1.18.1 \
--apiserver-advertise-address=192.168.56.109 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

```

注意**修改 apiserver-advertise-address 为 master 节点 ip**

参数解释：

```sh
–kubernetes-version: 用于指定k8s版本；
–apiserver-advertise-address：用于指定kube-apiserver监听的ip地址,就是 master本机IP地址。
–pod-network-cidr：用于指定Pod的网络范围； 10.244.0.0/16
–service-cidr：用于指定SVC的网络范围；
–image-repository: 指定阿里云镜像仓库地址
```

这一步很关键，由于kubeadm 默认从官网k8s.grc.io下载所需镜像，国内无法访问，因此需要通过–image-repository指定阿里云镜像仓库地址

集群初始化成功后返回如下信息：
记录生成的最后部分内容，此内容需要在其它节点加入Kubernetes集群时执行。
输出如下：

```sh
....

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.3.201:6443 --token 9ubtp9.btqe8ednga3w450p \
    --discovery-token-ca-cert-hash \
    sha256:b716f3d078a8ed62a6bc1380c1ea655622d0ef6743b86ba6a2d92144bff040d2 

```

注意: **保存好上面的信息**

### 配置kubectl工具

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 安装flannel

```bash
mkdir ~/k8s
cd k8s
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

如果 yml 中的 `"Network": "10.244.0.0/16"` 和 `kubeadm init xxx --pod-network-cidr` 不一样，就需要修改成一样的。不然可能会使得 Node 间 Cluster IP 不通。

由于我上面的 kubeadm init xxx --pod-network-cidr就 是 10.244.0.0/16。所以此 yaml 文件就不需要更改了。

### 获取镜像

> 注意: **这些镜像，也需要在node节点执行。**

查看 yaml 需要的镜像

```bash
cat kube-flannel.yml |grep image|uniq
        image: quay.io/coreos/flannel:v0.14.0
```

现在是 14.0 的版本了, 原文说去阿里云的 ACR 中下载, 但是 ACR 中没找到, 然后我去 docker hub 上找到了一个...

```bash
docker login
chengchao

docker pull xwjh/flannel:v0.14.0
docker tag xwjh/flannel:v0.14.0  quay.io/coreos/flannel:v0.14.0

docker tag xwjh/flannel:v0.14.0 chengchao/flannel:v0.14.0
docker push chengchao/flannel:v0.14.0

docker pull chengchao/flannel:v0.14.0
docker tag chengchao/flannel:v0.14.0  quay.io/coreos/flannel:v0.14.0

docker pull flannelcni/flannel:v0.17.0
docker tag flannelcni/flannel:v0.17.0 quay.io/coreos/flannel:v0.17.0
```

#### 加载flannel

```bash
kubectl apply -f kube-flannel.yml
```

### 设置开机启动

```bash
systemctl enable kubelet
```

安装命令补全:

```bash
yum install -y bash-completion

source < $(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
source  ~/.bashrc
```

## node 加入集群

> 在 nodes 的主机上执行！
>
> 

执行上面的 kubeadm join 命令

如果 token 过期:

原文链接：[https://blog.csdn.net/qq_38900565/article/details/102601527](https://blog.csdn.net/qq_38900565/article/details/102601527)

```bash

kubeadm token list
kubeadm token create

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

ed7ea5ae0c06f4ace9013e663b223e8da72e4e94e4dc657cfb1db68d777f3984

### !指定两个地方，token名和sha256
kubeadm join 192.168.3.201:6443 --token 9ubtp9.btqe8ednga3w450p \
     --discovery-token-ca-cert-hash \
     sha256:b716f3d078a8ed62a6bc1380c1ea655622d0ef6743b86ba6a2d92144bff040d2 
     
W0608 12:51:06.998161   25359 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.14. Latest validated version: 19.03
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.



```

设置开机启动:

```bash
systemctl enable kubelet

```

查看:

```bash
kubectl get nodes  -o wide

NAME                   STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
centos701.chaos.luxe   Ready    master   12m     v1.18.1   192.168.3.201   <none>        CentOS Linux 7 (Core)   3.10.0-1160.62.1.el7.x86_64   docker://20.10.14
centos702.chaos.luxe   Ready    <none>   3m6s    v1.18.1   192.168.3.202   <none>        CentOS Linux 7 (Core)   3.10.0-1160.62.1.el7.x86_64   docker://20.10.14
centos703              Ready    <none>   2m51s   v1.18.1   192.168.3.203   <none>        CentOS Linux 7 (Core)   3.10.0-1160.62.1.el7.x86_64   docker://20.10.14
```

## 部署 Dashboard UI

看看这里： [https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)



### Install

To deploy Dashboard, execute following command:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
```

Alternatively, you can install Dashboard using Helm as described at [`https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard`](https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard).

### 

### Access

To access Dashboard from your local workstation you must  create a secure channel to your Kubernetes cluster. Run the following  command:

```
kubectl proxy
```

Now access Dashboard at:

[`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/).

## Create An Authentication Token (RBAC)

To find out how to create sample user and log in follow [Creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md) guide.

**NOTE:**

- Kubeconfig Authentication method does not support external identity providers or certificate-based authentication.
- [Metrics-Server](https://github.com/kubernetes-sigs/metrics-server) has to be running in the cluster for the metrics and graphs to be available. Read more about it in [Integrations](https://github.com/kubernetes/dashboard/blob/master/docs/user/integrations.md) guide.







生成访问令牌

```bash
kubectl -n kube-system describe 
```





## 访问 Dashboard 用户界面

为了保护你的集群数据，默认情况下，Dashboard 会使用最少的 RBAC 配置进行部署。 当前，Dashboard 仅支持使用 Bearer 令牌登录。 要为此样本演示创建令牌，你可以按照 [创建示例用户](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user) 上的指南进行操作。





EOF

