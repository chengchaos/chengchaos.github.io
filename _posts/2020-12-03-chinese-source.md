---
title: 国内源
key: 2020-12-03
tags: maven python ubuntu centos
---

如下

<!--more-->

## 阿里云

[https://developer.aliyun.com/mirror/](https://developer.aliyun.com/mirror/)

## Python Pip3

- 清华：[https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple)
- 阿里云：[http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/)
- 中国科技大学 [https://pypi.mirrors.ustc.edu.cn/simple/](https://pypi.mirrors.ustc.edu.cn/simple/)
- 华中理工大学：[http://pypi.hustunique.com/](http://pypi.hustunique.com/)
- 山东理工大学：[http://pypi.sdutlinux.org/](http://pypi.sdutlinux.org/)
- 豆瓣：[http://pypi.douban.com/simple/](http://pypi.douban.com/simple/)

> 新版 ubuntu 要求使用 https 源，要注意。

### 临时使用

可以在使用pip的时候加参数`-i https://pypi.tuna.tsinghua.edu.cn/simple`
例如：

```bash
pip install -i http://mirrors.aliyun.com/pypi/simple/ tensorflow
```

这样就会从阿里云这边的镜像去安装tensorflow库。

### 永久使用

在Linux下, 修改 `~/.pip/pip.conf` (没有就创建一个文件夹及文件。文件夹要加“.”，表示是隐藏文件夹)

内容如下：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

windows下，直接在user目录中创建一个pip目录，再新建文件 pip.ini。（例如：C:\Users\WQP\pip\pip.ini）内容同上。

## Pipenv

编辑项目使用的 Pipfile 文件，修改 url 属性的指向地址。

```ini
[[source]]
#url = "https://pypi.org/simple"
url = "https://mirrors.aliyun.com/pypi/simple"
```

### 可选的源

- 阿里云：[https://mirrors.aliyun.com/pypi/simple](https://mirrors.aliyun.com/pypi/simple)
- 豆瓣：[https://pypi.douban.com/simple](https://pypi.douban.com/simple/)
- 清华大学：[https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple/)
- 中国科学技术大学：[https://pypi.mirrors.ustc.edu.cn/simple](https://pypi.mirrors.ustc.edu.cn/simple/)





## CentOS

```sh
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 根据版本下载
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo

# 目前没有 8
wget -O /etc/yum.repos.d/epel-6.repo https://mirrors.aliyun.com/repo/epel-6.repo
wget -O /etc/yum.repos.d/epel-6.repo https://mirrors.aliyun.com/repo/epel-7.repo

yum clean all
yum makecache

```

转自：

- [python pip 使用国内源](https://www.jianshu.com/p/dfbb90995a2c)

EOF

---

Power by TeXt.
