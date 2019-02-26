---
title: Redis-安装
key: 20190226
tags: redis
---

生产环境中的 Redis 都是安装在 Linux 环境中的。

<!--more-->

## CentOS 7

安装依赖：

```
	$ sudo yum install gcc gcc-c++
```

解压缩，然后编译：

```
	$ make PREFIX=/opt/soft/redis install #安装到指定目录中
```

如果出现类似 ` jemalloc/jemalloc.h: No such file or directory` 这样的错误：

一个办法是：

```
	$ make MALLOC=libc  
```

另一个：

```

	$ yum search jemalloc      
	Loaded plugins: fastestmirror
	Loading mirror speeds from cached hostfile
	 * base: mirrors.aliyun.com
	 * extras: mirrors.aliyun.com
	 * updates: mirrors.tuna.tsinghua.edu.cn
	=================================================== Matched: jemalloc ====================================================
	memkind.x86_64 : User Extensible Heap Manager
	memkind-devel.x86_64 : Memkind User Extensible Heap Manager development lib and tools
	$ sudo yum install memkind-devel
	$ make
```

## Ubuntu

1 编译


```bash
    $ sudo apt-get install build-essentail
    $ mkdir -p /work/redis
    $ cd /work/redis
    $ wget http://download.redis.io/releases/redis-4.0.1.tar.gz
    $ tar zxvf redis-xxx.gz
    $ cd redis-xxx.gz
    $ mkdir -p /work/redis/conf/
    $ cp redis.conf /work/redis/conf/
    $ cd deps
    $ make hiredis lua jemalloc linenoise
    $ cd ..
    $ make
    ...
    It's a good idea to run 'make test' ;)
    ...
    $ make PREFIX=/work/redis install

```

2 直接安装

```base
    $ sudo apt-get update
    $ sudo apt-get install redis-server
    $ which redis-server
```

## 关闭

```bash
    $ kill `pidof redis-server`
    $ cd /work/redis
    $ bin/redis-cli shutdown
    $ /etc/init.d/redis-server stop
```


差不多就酱紫。

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
