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

### 运行初始化命令

```bash
kubeadm init --kubernetes-version=1.18.1 \
--apiserver-advertise-address=10.10.10.241 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

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

kubeadm join 10.10.10.241:6443 --token io1pa7.pp3wbn8i4hbi7ktn \
    --discovery-token-ca-cert-hash sha256:e9dd69f71312e61280d6d3a257f811135b43dea7e62fd667478bb776af16737d

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
mkdir k8s
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

执行上面的 kubeadm join 命令

如果 token 过期:

原文链接：[https://blog.csdn.net/qq_38900565/article/details/102601527](https://blog.csdn.net/qq_38900565/article/details/102601527)

```bash

kubeadm token list
kubeadm token create

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

ed7ea5ae0c06f4ace9013e663b223e8da72e4e94e4dc657cfb1db68d777f3984

### !指定两个地方，token名和sha256

kubeadm join 192.168.1.200:6443 \
  --token wxvdun.vec7m9cu4ru3hngg \
  --discovery-token-ca-cert-hash sha256:ed7ea5ae0c06f4ace9013e663b223e8da72e4e94e4dc657cfb1db68d777f3984 



```

设置开机启动:

```bash
systemctl enable kubelet

```

查看:

```bash
kubectl get nodes  -o wide
```

EOF
