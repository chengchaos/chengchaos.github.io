---
title: 清理 Docker 的日志
key: 20181226
tags: docker 
---

我有一台服务器上面部署了多个docker容器， 并且每个docker容器都往stderr中源源不断的输出日志，导致今天磁盘被占满了。搜索了一下，docker官方网站上提供了一篇解决方案的[文章](https://success.docker.com/article/no-space-left-on-device-error)。

<!--more-->

Docker容器在启动/重启的时候会往/var/lib/docker中写东西，如果你在启动docker容器遇到No space left on device的问题，可以按照下面的步骤进行清理相关的日志操作。

## 步骤 1， 查找巨大的日志文件



对 `/var/lib/docker/containers` 下的文件夹进行排序，看看哪个容器占用了太多的磁盘空间

```bash
$ du -d1 -h /var/lib/docker/containers | sort -h
```

上面的命令会按照升序的方式对于容器文件夹进行排序，并列出容器文件夹的大小:

```bash	
[root@dbl14195 testnet]# du -d1 -h /var/lib/docker/containers | sort -h
36K /var/lib/docker/containers/4d91f92dd7604216f2e9e123572e9a80d7bbee3d8c8ce7be2ed241c572816fb6
40K /var/lib/docker/containers/374aa0ba92b37d829012282ff15c1bb838d95dedb54589874c4285991be2d4aa
40K /var/lib/docker/containers/7cfdbc453b2f7109b52e974061754266e6cdc0ccaee62ab4a5887865113e1144
40K /var/lib/docker/containers/84ee24989ad52383c6e99eaa4d236e600095aa7f855e81fbafe10416b75ceefb
40K /var/lib/docker/containers/aeced3ef3e23df27e52f65743bb05448b46a2c660acc5b0aab12604e060779b4
40K /var/lib/docker/containers/c36722baf0d2e1c22b7dde9979665ab62cd8ab85c3f1d0f427bb7a34e0fd977a
44K /var/lib/docker/containers/62477b332d18e192d70c7420435d47a379e6bbd8de13da8a8762e0fd95b341ca
44K /var/lib/docker/containers/78da0cf9743b6940fabbbd8c574b99dc5deb642fa998a8f819a6c6978fc875d7
44K /var/lib/docker/containers/9f63daf7caa7c469385bed4b178fbfe662e15b8c569c6644081af090f8e40426
44K /var/lib/docker/containers/e2d1286119a45aac7e58d6dac6e4b44b1d1288799b735943be45abed50244e56
56K /var/lib/docker/containers/ebd1bd211a1b9d02bb39bfb80eec3d0960a5b25e18f54d7371781ec456e7a1e8
176K /var/lib/docker/containers/1fe0a241e5ce9726c547c68739793633f9dd906768a36fe80e8fb80373aa3bfb
17M /var/lib/docker/containers/ac30e68d454b37d22b3964053a2b52ba043baa1add13556a09c0e3e05589104f
25M /var/lib/docker/containers/872ca4e3d005594591ca2df0e832d36eef448981ab2820c69df4ff1399f8423e
25M /var/lib/docker/containers/bd49a0a0368b99a9f69981d8b921ea1830957451577b635a07d5425d48e1144b
30M /var/lib/docker/containers/8f732390a020a6ef647fabb04da32c87d6341b72ac2af6bb4a1cf5743fda54db
88M /var/lib/docker/containers/648e883aa0a93f696f64e4ab76434657f4845769fe1eaaad49c2dc1d7960f2b0
171M /var/lib/docker/containers/8de7ff9f0276586a6ab346c2be1c9dc879bbb0d795fa7776c1d8d1568ea2794a
354M /var/lib/docker/containers
```

## 步骤 2，选择你要清理的容器进行清理


```bash	
$ cat /dev/null > /var/lib/docker/containers/container_id/container_log_name
```

上述命令会清空对应的日志,如:

```bash	
cat /dev/null > /var/lib/docker/containers/374aa0ba92b37d829012282ff15c1bb838d95dedb54589874c4285991be2d4aa/374aa0ba92b37d829012282ff15c1bb838d95dedb54589874c4285991be2d4aa-json.log
```

## 步骤 3，限制日志文件的大小

启动容器时，可以通过参数设置日志文件的大小、日志文件的格式。

```bash	
docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash
```


命令参数说明

```bash
    du :  estimate file space usage 
    -d, --max-depth=N : 目录深度
    -h, --human-readable : 人类可读
```


参考原文： [Docker日志太多导致磁盘占满](https://colobu.com/2018/10/22/no-space-left-on-device-for-docker/)

- 


If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
