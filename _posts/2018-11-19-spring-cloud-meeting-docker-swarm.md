---
title: Spring cloud 遇见 Docker Swarm
key: 20181119
tags: spring-cloud docker-swarm 
---

如何在 docker swarm 中部署 spring cloud，这个问题耽误了很久。

<!--more-->

后来终于有一个周六，可以专心的搞这个事情了。


## 测试环境

首先安装 VirtualBox 虚，再虚拟出两个 CentOS 7 的虚拟机。

- 第一台主机名： vbc701, IP: 192.168.1.11
- 第二台主机名： vbc702, IP: 192.168.1.12

然后分别安装 Docker，允许内网互通，允许内网 ssh 登录，安装 Java 环境 …… Sonds boring.


## 创建 docker swarm

设置 vbc701 为主节点：

```bash
$ docker swarm init
Swarm initialized: current node (pa5hfbqf0gqhpwkjddl3jd20b) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-50fbj4ukn58gr5onytzd30i5ir6nxd0rn3a1rtnux32eu68ffk-afkp5dr185zgkab4k8mgtlhuv 192.168.1.11:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

添加一个 overlay 网络，这个比较重要的一步。

```bash
$ docker network create -d overlay springcloud-overlay
```

到 vbc702 主机上，输入以下命令加入 Swarm 中：

```bash
$ docker swarm join --token SWMTKN-1-50fbj4ukn58gr5onytzd30i5ir6nxd0rn3a1rtnux32eu68ffk-afkp5dr185zgkab4k8mgtlhuv 192.168.1.11:2377
This node joined a swarm as a worker.
```


## spring cloud 的关键配置

创建了一个 Spring Cloud 的 demo，有 3 个微服务，其中 Eureka 对外保留 8761 端口用于注册，serv1 服务对外暴露 8101 端口，serv2 服务对外暴露 8102 端口。他们都注册到 Eureka 上，通过 serv1 的 `/hello-serv2` 调用 serv2 服务上的 hello


Eureka 的配置文件，重点是 **`EUREKA_SERVER_ADDRESS1`** 这个变量，将来使用环境变量的方式传递到容器中。

```yaml

######################################## 测试环境配置 test
# 测试环境
spring:
  profiles: test
eureka:
  server:
    # 测试时关闭自我保护机制，保证不可用服务及时踢出
    enable-self-preservation: false
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${EUREKA_SERVER_ADDRESS1}:8761/eureka/

```

SERV1 和 SERV2 的配置，同样 **`EUREKA_SERVER_ADDRESS1`** 和 **`EUREKA_SERVER_ADDRESS2`** 两个变量也将使用环境变量的方式传递到容器中。

另外设置 `preferred-networks: none` 是为了避免 spring boot 在容器中启动后，不能获取正确的 IP 地址。


```yaml

######################################## 测试环境配置 test

spring:
  profiles: test
  cloud:
    inetutils:
      preferred-networks: none

eureka:
  client:
    serviceUrl:
      defaultZone: http://${EUREKA_SERVER_ADDRESS1}:8761/eureka/,http://${EUREKA_SERVER_ADDRESS2}:8761/eureka/
  instance:
    prefer-ip-address: true

```
## docker stack 部署文件

*docker-eureka.yml 文件：*

```yaml
version: "3"
services:
  eureka1:
    image: tsp/tsp-registry-center
    networks:
      springcloud-overlay:
        aliases:
          - euraka
    ports:
      - "8761:8761"
    environment:
      - EUREKA_SERVER_ADDRESS1=http://eureka2:8761/eureka/
  eureka2:
    image: tsp/tsp-registry-center
    networks:
      springcloud-overlay:
        aliases:
          - eureka
    ports:
      - "8762:8761"
    environment:
      - EUREKA_SERVER_ADDRESS1=http://eureka1:8761/eureka/
networks:
  springcloud-overlay:
    external:
      name: springcloud-overlay
```


*docker-serv1.yml 文件：*

```yaml
version: "3"
services:
  serv1:
    image: tsp/serv1
    deploy:
      replicas: 2
    ports:
      - "8101:8101"
    networks:
      - springcloud-overlay
    environment:
      - EUREKA_SERVER_ADDRESS1=eureka1
      - EUREKA_SERVER_ADDRESS2=eureka2
    
networks:
  springcloud-overlay:
    external:
      name: springcloud-overlay
```


*docker-serv2.yml 文件：*

```yaml
version: "3"
services:
  serv2:
    image: tsp/serv2
    deploy:
      replicas: 2
    ports: 
      - "8102:8102"
    networks:
      - springcloud-overlay
    environment:
      - EUREKA_SERVER_ADDRESS1=eureka1
      - EUREKA_SERVER_ADDRESS2=eureka2
networks:
  springcloud-overlay:
    external:
      name: springcloud-overlay
```

重点是 **`external`** 的设置，这样这些微服务才可以在同一个网络内，才可以互相通信。




## 执行命令

```bash
$ docker stack deploy -c docker-eureka.yml eureka
$ docker stack deploy -c docker-serv1.yml serv1
$ docker stack deploy -c docker-serv2.yml serv2
```

这样就可以了。

用于没有资源搭建镜像的私服，镜像是通过 `scp` 和 `ssh` 传递到另外一台主机上的 :( 。

```bash
$ cat build-serv1.sh 
#!/usr/bin/env bash
bash ../serv1/src/main/script/build.sh
docker save tsp/serv1 -o serv1.tar 
scp serv1.tar vbc702:~/
ssh vbc702 docker load -i ~/serv1.tar
rm -f serv1.tar
$ 
```



---

参考文档： [Docker Swarm运行Spring Cloud应用（二）：Eureka高可用](https://blog.csdn.net/qq_32440951/article/details/79692086)

- 


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
