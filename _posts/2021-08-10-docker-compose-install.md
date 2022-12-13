·---
title: docker-compose安装 (2020最新版)
key: 2021-08-15
tags: docker docker-compose
---

https://blog.csdn.net/weixin_45444133/article/details/110588884

docker-compose 的官方文档: [https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)

## 安装 docker-compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## 安装 Harbor

下载镜像包. [https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

```bash
wget -c https://github.com/goharbor/harbor/releases/download/v2.3.1/harbor-offline-installer-v2.3.1.tgz
tar -zxvf harbor-offline-installer-v2.3.1.tgz


sudo -i
cd /etc/docker
touch daemon.json
cat > daemon.json <<EOF
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
exit
docker load -i harbor.v2.3.1.tar.gz

####### 创建 harbor.yml 并编辑修改.
####### 先去掉 https 的部分.
sudo ./prepare
sudo ./install.sh
```

推镜像前要打一个标签

```bash
docker tag openjdk:7-jre harbor.vm241.local:8080/micro-service/openjdk:7-jre
docker push


insecure registries:
harbor.vm241.local:8080


# cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://0nth4654.mirror.aliyuncs.com"],
  "insecure-registries": ["harbor.vm241.local"]
}



docker pull chengchao/openjdk_8-jdk-alpine:v1
docker tag chengchao/openjdk_8-jdk-alpine:v1 harbor.vm241.local/test/alpine-openjdk8:v1
docker push harbor.vm241.local/test/alpine-openjdk8:v1

[vmuser@vm241 ~]$ docker login http://harbor.vm241.local

test
Test12345678

#### 推送命令:
## 在项目中标记镜像： 
docker tag SOURCE_IMAGE[:TAG] harbor.vm241.local/test/REPOSITORY[:TAG]
## 推送镜像到当前项目： 
docker push harbor.vm241.local/test/REPOSITORY[:TAG]
```

openjdk:8-alpine

[vmuser@vm241 ~]$ docker pull openjdk:8-alpine

docker file

```dockerfile
From alpine
MAINTAINER ×××debiaobiao
RUN apk update && 
 apk add openjdk8 curl busybox tzdata && 
 cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && 
 echo Asia/Shanghai > /etc/timezone && 
 apk del tzdata && 
 rm -rf /tmp/* /var/cache/apk/*

```

我的构建历史:

```sh
2018/1/10 上午5:10
ADD file:093f0723fa46f6cdbd6f7bd146448bb70ecce54254c35701feeceb956414622f in /
2018/1/10 上午5:10
CMD ["/bin/sh"]
2018/1/10 下午12:48
ENV LANG=C.UTF-8
2018/1/10 下午12:48
RUN { echo '#!/bin/sh'; echo 'set -e'; echo; echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"'; } > /usr/local/bin/docker-java-home && chmod +x /usr/local/bin/docker-java-home
2018/1/10 下午12:50
ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
2018/1/10 下午12:50
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
2018/1/10 下午12:50
ENV JAVA_VERSION=8u151
2018/1/10 下午12:50
ENV JAVA_ALPINE_VERSION=8.151.12-r0
2018/1/10 下午12:51
RUN set -x && apk add --no-cache openjdk8="$JAVA_ALPINE_VERSION" && [ "$JAVA_HOME" = "$(docker-java-home)" ]
2018/5/26 上午9:40
/bin/sh
```

## CICD 和 DevOps

### 来源

- 编译失败
- 手动发布
- 运行异常
- 手动部署


提交, 触发 hook 编译一遍
增加静态代码检查

[https://github.com/liuyi01/imooc-docs/blob/master/gitlab-install.md](https://github.com/liuyi01/imooc-docs/blob/master/gitlab-install.md)

docker docker 安装 gitlab

```bash
ssh-keygen -t rsa -c 'chengchaos@outlook.com'
```

安装 jenkins

安装 pipeline 插件

安装 git
安装 java
安装 maven

构建一个流水线

```sh
local - push -> git

  -> web hook ->

  项目, setting -> integrations ->webhooks
    url (回调地址)
    事件: push
```

jekins 创建 job pipeline

构建触发器, 触发远程构建

jinkins 系统, 全局安全配置  CSRF Protection
allow anonymous ...

```grovvy
#!groovy
pipeline {
    agent any
    environment {
        REPOSITORY = "ssh://git@gitlab..."
        MODULE="user-edge-service"
    }

    stages {
        stage('获取代码') {
            steps {
                echo "start fetch code from git:${REPOSITORY}"
                deleteDir()
                git "${REPOSITORY}"
            }
        }
        stage('编译') {
            steps {
                echo "start compile"
                sh "mvn -U -pl ${MODULE} -am clean package -D"
                git "${REPOSITORY}"
            }
        }
        stage('构建镜像'){
            steps {
                echo "start build image"
                sh "${SCRIPT_PATH}/build-images.sh ${MODULE}"
            }
        }
        stage('发布系统'){
            steps {
                echo "start build image"
                sh "${SCRIPT_PATH}/deploy.sh ${MODULE}"
            }
        }
    }
}

```

build-image.sh

```bash
#!/usr/bin/env bash
MODULE=$1
TIME=`date "+%Y%m%d%H%M"`
GIT_REVISION=`git log -1 --pretty=format:"%h"`
IMAGE_NAME=hub.vm241.local:8080/micro-service/${MODULE}:${TIME}_${GIT_REVISION}

cd ${MODULE}

docker build -t ${IMAGE_NAME} .
cd -
docker push ${IMAGE_NAME}
echo "${IMAGE_NAME}" > IMAGE_NAME
```

deploy.sh

```bash
#!/usr/bin/env bash

IMAGE=`cat IMAGE_NAME`
echo "update image to : ${IMAGE}
kubectl set image deployments/uer-service-deployment user-edge-service=${IMAGE}

DEPLOYMENT=$1
MODULE=$2
kubectl set image deployments/${DEPLOYMENT} ${MODULE}=${IMAGE}
```


