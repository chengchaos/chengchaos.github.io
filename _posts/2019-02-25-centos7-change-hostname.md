---
title: CentOS 7 修改主机名称
key: 2019-02-25
tags: centos7 hostname
---

CentOS 7 主机修改 hostname。

<!--more-->

## 方法 1：使用 `hostnamectl` 命令

```sh
[root@bogon ~]# hostnamectl set-hostname c02
```

## 方法 2：修改  `/etc/hostname` 文件

```sh
[root@bogon ~]# vim /etc/hostname

```
## CentOS 7 添加 EPEL 存储库



[https://fedoraproject.org/wiki/EPEL](https://fedoraproject.org/wiki/EPEL)



```sh
sudo yum install epel-release
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```



以上。

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
