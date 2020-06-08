---
title: Docker 国内镜像
key: 2020-06-09
tags: docker 
---





Docker domestic images ... 是这么写的吗？



<!--more-->



![dockerhero.jpg](https://upload-images.jianshu.io/upload_images/1881763-489ccadbb1e791c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Docker 官方针对中国区推出了镜像加速服务。通过 Docker 官方镜像加速，国内用户能够以更快的下载速度和更强的稳定性访问最流行的 Docker 镜像。

### 如何使用

Docker 中国官方镜像加速可通过 registry.docker-cn.com 访问。目前该镜像库只包含流行的公有镜像，而私有镜像仍需要从美国镜像库中拉取。

可以使用以下命令直接从该镜像加速地址进行拉取。

```bash
docker pull registry.docker-cn.com/library/ubuntu:16.04
```
> 注:除非您修改了Docker守护进程的–registry-mirror参数,否则您将需要完整地指定官方镜像的名称。例如，library/ubuntu、library/redis、library/nginx。


### 给Docker守护进程配置加速器

国内很多云服务商都提供了国内加速器服务，例如：

- [网易云加速器 https://hub-mirror.c.163.com](https://www.163yun.com/help/documents/56918246390157312)
- [阿里云加速器(需登录账号获取)](https://cr.console.aliyun.com/cn-hangzhou/mirrors)

> 由于镜像服务可能出现宕机，建议同时配置多个镜像。各个镜像站测试结果请到 [docker-practice/docker-registry-cn-mirror-test](https://github.com/docker-practice/docker-registry-cn-mirror-test/actions) 查看。
>
> 国内各大云服务商均提供了 Docker 镜像加速服务，建议根据运行 Docker 的云平台选择对应的镜像加速服务，具体请参考官方文档。

对于使用 [systemd](https://www.freedesktop.org/wiki/Software/systemd/) 的系统，请在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）

> 为了避免运行 Docker 是使用端口映射导致的在防火墙上开孔，又增加了 `"iptables":false` 
>
> 

```json
{
 "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://hub-mirror.c.163.com"
  ],
  "iptables" : false
}
```
### 检查加速器是否生效

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
$ sudo systemctl status docker
$ sudo docker info
...
 Registry Mirrors:
  https://hub-mirror.c.163.com/
  https://registry.cn-beijing.aliyuncs.com/
  https://registry.cn-hangzhou.aliyuncs.com/
  https://registry.aliyuncs.com/
```

参考：

- 镜像加速器 [https://yeasy.gitbook.io/docker_practice/install/mirror](https://yeasy.gitbook.io/docker_practice/install/mirror)
- [https://www.jianshu.com/p/84b6fe281b4d](https://www.jianshu.com/p/84b6fe281b4d)
- [http://www.yunweipai.com/archives/20727.html?utm_source=tuicool&utm_medium=referral](http://www.yunweipai.com/archives/20727.html?utm_source=tuicool&utm_medium=referral)
- [https://docs.docker.com/](https://docs.docker.com/)

.

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
