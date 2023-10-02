---
title: Kudu 本地安装部署指北
key: 2023-03-08
tags: linux centos7 kudu install
---

> Suggest search： linux centos7 kudu install 安装 kudu安装

使用 CentOS 7 进行说明，详细信息见官方文档中的[安装指南](https://kudu.apache.org/docs/installation.html)。

官网：<https://kudu.apache.org/docs/>

<!--more-->

## 0x10 Kudu 原理

### 0x11概述

Kudu是一个类似Hbase的列式存储分布式数据库，它的定位介于 HDFS 和 HBase 之间。

- HDFS：使用列式存储，适合 OLAP，不支持单条纪录级别的更新操作，随机读写性能差，不适合用来作实时查询。
- HBase：可以高效地随机读写，适用 OLTP，但是不适用于 SQL，且大数据量时读性能较差。

Kudu 就是结合了两者的优点，平衡了两者的缺点。从而同时在 OLTP 和 OLAP 中都提供较好的性能。这样就无需为了解决以上两者的缺点而搭建两种架构。

### 0x12 基本概念

- Table：是一张表，具有全局的主键，一张 table 可以分为多个段，即 tablet。
- Tablet：一个 tablet 是一张表连续的一个段，类似于关系型数据库的 partition 分区。
- Tablet server：存储 tablet，并且向客户端提供读取数据的服务。对于指定的 tablet，有一个 server 作为 leader，基余 server 作为 follower 副本。只有 leader 处理写请求，follower 负责和 leader 同步数据，并且提供读服务。
- Master：主要用于管理元数据，监听 tserver(tablet server) 的状态。当 client 发出请求时，其先对请求做校验，再分配 tserver 给 client 进行请求。

## 0x20 Kudu 安装

### 0x21 前置条件

硬件

- 至少一个主机来作为 Kudu 的 Master 节点，Master 节点数必须是奇数，推荐配置 1 个或者 3 个。
- 至少一个主机来作为 tablet 节点，如果需要备份复制，那么至少需要 3 个服务器 
> 2k 个数量的 Master 节点和 2k-1 个节点的容错等级是一样的。4 个节点和 3 个节点一样，只能容忍一个错误。2 个节点则不能容错。（这也是为什么官方推荐 1 个或 3 个节点的原因） 
操作系统

- RHEL 7, RHEL 8, CentOS 7, CentOS 8, Ubuntu 18.04 (bionic), Ubuntu 20.04 (focal)
- macOS 10.13 (High Sierra), macOS 10.14 (Mojave), macOS 10.15 (Catalina)
- A kernel and filesystem that support hole punching. Hole punching is the use of the fallocate(2) system call with the FALLOC_FL_PUNCH_HOLE option set. See [troubleshooting hole punching](https://kudu.apache.org/docs/troubleshooting.html#req_hole_punching) for more information.
- ntp or chrony.
- xfs/ext4 文件系统
- Although not a strict requirement, it’s highly recommended to use nscd to cache both DNS name resolution and static name resolution. See [troubleshooting slow DNS lookups](https://kudu.apache.org/docs/troubleshooting.html#slow_dns_nscd) for more information.
- 硬盘至少 50G 以上的可用空间， 如果想完全编译，至少 120G 以上可用空间。
- If solid state storage is available, storing Kudu WALs on such high-performance media may significantly improve latency when Kudu is configured for its highest durability levels.
- C++17 的编译器 （GCC4.8）
- JDK 8 is required to build Kudu, but a JRE is not required at runtime except for tests.

### 0x22 CentOS7 中安装

RHEL or CentOS 7.0 or later is required to build Kudu from source. To build on a version older than 8.0, the Red Hat Developer Toolset must be installed (in order to have access to a C++17 capable compiler).

0, System inforation

```bash
Last login: Sun Aug 13 11:39:03 2023 from 192.168.1.54
[chengchao@matrix002 ~]$ uname -a
Linux matrix002 3.10.0-1160.95.1.el7.x86_64 #1 SMP Mon Jul 24 13:59:37 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
[chengchao@matrix002 ~]$ df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G  8.5M  3.9G   1% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   50G  6.9G   44G  14% /
/dev/sda1               1014M  207M  808M  21% /boot
/dev/mapper/centos-home  107G  346M  107G   1% /home
tmpfs                    783M     0  783M   0% /run/user/1000
[chengchao@matrix002 ~]$ cat /etc/*rele*
CentOS Linux release 7.9.2009 (Core)
Derived from Red Hat Enterprise Linux 7.9 (Source)
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

CentOS Linux release 7.9.2009 (Core)
CentOS Linux release 7.9.2009 (Core)
cpe:/o:centos:centos:7
[chengchao@matrix002 ~]$ gcc --version
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44)
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```


1, Install the prerequisite libraries, if they are not installed.

```bash
$ sudo yum install autoconf automake cyrus-sasl-devel cyrus-sasl-gssapi \
  cyrus-sasl-plain flex gcc gcc-c++ gdb git java-1.8.0-openjdk-devel \
  krb5-server krb5-workstation libtool make openssl-devel patch \
  pkgconfig redhat-lsb-core rsync unzip vim-common which
```

2, If building on RHEL or CentOS older than 8.0, install the Red Hat Developer Toolset. Below are the steps required for CentOS. If you are on RHEL, follow their documentation [here](https://developers.redhat.com/products/developertoolset/hello-world).

```bash
$ sudo yum install centos-release-scl-rh
$ sudo yum install devtoolset-8
```

3, 可选的编译依赖略过了，需要的可以看官网。

4， Clone the Git repository and change to the new kudu directory.

```bash
$ cd  /works
$ mkdir git-repo
$ cd git-repo
$ git clone https://github.com/apache/kudu
Cloning into 'kudu'...
remote: Enumerating objects: 146422, done.
remote: Counting objects: 100% (20818/20818), done.
remote: Compressing objects: 100% (970/970), done.
remote: Total 146422 (delta 20063), reused 19948 (delta 19819), pack-reused 125604
Receiving objects: 100% (146422/146422), 78.90 MiB | 17.47 MiB/s, done.
Resolving deltas: 100% (109364/109364), done.

```

5, 使用 `build-if-necessary.sh` 脚本可以构建任何缺失的第三方依赖。需要使用到Python，本文中安装的是 python 3.6。

```bash
$ sudo yum -y install python3
$ python3 --version
Python 3.6.8
$ cd kudu
$ pwd
/works/git-repo/kudu
$ ls
build-support   cmake_modules      docker  examples  kubernetes   NOTICE.txt  README.adoc     src         version.txt
CMakeLists.txt  CONTRIBUTING.adoc  docs    java      LICENSE.txt  python      RELEASING.adoc  thirdparty  www
$ build-support/enable_devtoolset.sh thirdparty/build-if-necessary.sh
```

6, 
Build Kudu, using the utilities installed in the previous step. Choose a build directory for the intermediate output, which can be anywhere in your filesystem except for the kudu directory itself. Notice that the devtoolset must still be specified, else you’ll get `cc1plus: error: unrecognized command line option "-std=c++17"`.

```bash
mkdir -p build/release
cd  build/release
../../build-support/enable_devtoolset.sh \
  ../../thirdparty/installed/common/bin/cmake \
  -DCMAKE_BUILD_TYPE=release \
  -DNO_TESTS=1 \
../..
make -j4

```

`--DNO_TEST=1` 参数的作用是在编译时跳过测试工作的编译，否则编译结果将十分巨大。

> 安装时可以使用以下参数来跳过指定的组件：
>	- `-DKUDU_CLIENT_INSTALL=OFF` 跳过 kudu 客户端的安装
>	- `-DKUDU_TSERVER_INSTALL=OFF` 跳过 tserver 的安装
>	- `-DKUDU_MASTER_INSTALL=OFF` 跳过 master的 安装
> 假如某个服务器只作为tserver使用，那么可以跳过master的安装以减少磁盘资源占用并加快安装过程。
> If you need to install only a subset of Kudu executables, you can set the following cmake flags to OFF in order to skip any of the executables.
> - KUDU_CLIENT_INSTALL (set to OFF to skip installing /usr/local/bin/kudu executable)
> - KUDU_TSERVER_INSTALL (set to OFF to skip installing /usr/local/sbin/kudu-tserver executable)
> - KUDU_MASTER_INSTALL (set to OFF to skip installing /usr/local/sbin/kudu-master executable)
> E.g., use the following variation of cmake command if you need to install only Kudu client libraries and headers:
> ```
> ../../build-support/enable_devtoolset.sh \
> ../../thirdparty/installed/common/bin/cmake \
> -DKUDU_CLIENT_INSTALL=OFF \
> -DKUDU_MASTER_INSTALL=OFF \
> -DKUDU_TSERVER_INSTALL=OFF
> -DCMAKE_BUILD_TYPE=release ../..
> ```

编译过程非常耗时。够喝一壶的了。

6.1 编译报错的调整

编译报错，因此需要手动安装 gradle。 [官网](https://gradle.org/releases/) 下载， 此时的版本是 gradle-8.2.1-bin.zip 

```bash
$ mkdir -p /usr/local/jvm
$ unzip gradle-8.2.1-bin.zip
$ sudo mv gradle-8.2.1 /usr/local/jvm 
$ sudo ln -s /usr/local/jvm/gradle-8.2.1 /usr/local/jvm/gradle
$ sudo mv g
```

修改环境变量：

```bash
vim ~/.bashrc

GRADLE_HOME='/usr/local/jvm/gradle'; export GRADLE_HOME
PATH=$GRADLE_HOME/bin:$PATH

export PATH

source ~/.bashrc 
```

再次报错， 不能访问 raw.githubusercontent.com

```bash
curl: (7) Failed connect to raw.githubusercontent.com:443; Connection refused

```
修改 /etc/hosts 文件， 添加： 

```
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```

7, 完成编译后，构建可执行文件，库文件和头文件。进入上一步编译的输出目录，运行下面的命令进行构建。用 `DESTDIR=xxx` 指定自己想要输出的位置，如果不指定则默认输出到 `/usr/local/` 目录下面。

```
sudo make DESTDIR=/home/kudu install
```

Running `sudo make install` installs the following:

- kudu-tserver and kudu-master executables in /usr/local/sbin
- Kudu command line tool in /usr/local/bin
- Kudu client library in /usr/local/lib64/
- Kudu client headers in /usr/local/include/kudu

The default installation directory is `/usr/local`. You can customize it through the DESTDIR environment variable.

```bash
sudo make DESTDIR=/opt/kudu install
Install the project...
-- Install configuration: "RELEASE"
-- Installing: /opt/kudu/usr/local/lib64/libkudu_client.so.0.1.0
-- Installing: /opt/kudu/usr/local/lib64/libkudu_client.so.0
-- Installing: /opt/kudu/usr/local/lib64/libkudu_client.so
-- Installing: /opt/kudu/usr/local/include/kudu/client/callbacks.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/client.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/columnar_scan_batch.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/hash.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/resource_metrics.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/row_result.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/scan_batch.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/scan_predicate.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/schema.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/shared_ptr.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/stubs.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/value.h
-- Installing: /opt/kudu/usr/local/include/kudu/client/write_op.h
-- Installing: /opt/kudu/usr/local/include/kudu/common/partial_row.h
-- Installing: /opt/kudu/usr/local/include/kudu/util/kudu_export.h
-- Installing: /opt/kudu/usr/local/include/kudu/util/int128.h
-- Installing: /opt/kudu/usr/local/include/kudu/util/monotime.h
-- Installing: /opt/kudu/usr/local/include/kudu/util/slice.h
-- Installing: /opt/kudu/usr/local/include/kudu/util/status.h
-- Installing: /opt/kudu/usr/local/share/doc/kuduClient/examples/CMakeLists.txt
-- Installing: /opt/kudu/usr/local/share/doc/kuduClient/examples/example.cc
-- Installing: /opt/kudu/usr/local/share/doc/kuduClient/examples/non_unique_primary_key.cc
-- Installing: /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientTargets.cmake
-- Installing: /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientTargets-release.cmake
-- Installing: /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientConfig.cmake
-- Munging kudu client targets in /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientConfig.cmake
-- Munging kudu client targets in /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientTargets-release.cmake
-- Munging kudu client targets in /opt/kudu/usr/local/share/kuduClient/cmake/kuduClientTargets.cmake
-- Installing: /opt/kudu/usr/local/sbin/kudu-master
-- Installing: /opt/kudu/usr/local/bin/kudu
-- Installing: /opt/kudu/usr/local/sbin/kudu-tserver
```

## 0x30 Kudu 的配置

我们可以通过在命令行启动时传递参数来对 Kudu 进行配置，当参数过多时，也可在启动时使用  `–-flagfile=<file>` 参数来引用配置文件。更奇妙地是，在配置文件中还可以通过 `--flagfile=<file>` 来再次引用另一个配置文件，这样配置参数就相当灵活。

Master 和 Tablet 节点的配置可以放在同一个配置文件中，它们会自动识别哪些是属于自己的配置参数。配置参数的格式为 `-–flag=xxx` ，前缀可以用一个或者二个 `-` 都 OK。

### 0x31 配置目录

每个节点都需要手动配置目录。

- `--fs_wal_dir`：用来配置 Kudu 的写前日志 WAL（Write-Ahead Log）的输出目录。
- `--fs_metadata_dir`：配置 Kudu 每个 tablet 的元数据的存放目录 > 建议这些目录放在一个高性能的、有高带宽的磁盘上，例如SSD。如果--fs_metadata_dir没有指定，那么元数据会放在WAL的输出目录。
- `--fs_data_dirs`：指定 Kudu 用于存放数据块的目录。可以通过 `,` 分隔添加多个目录，Kudu 会把数据平均地存在这些目录中，如果不指定该参数，也是会把数据块放在 WAL 的输出目录中。
> `--fs_wal_dir` 和 `--fs_metadata_dir` 可以设置为 `--fs_data_dirs` 给出的目录列表中的某一个目录，但是不能是任何目录列表中目录的子目录。

每一个目录都只能被一个 Kudu 进程使用，所以多个 Kudu 进程要分别设置属于自己的各种目录。否则启动可能会失败。

以上是一些最基本的设置，更多关于 Master 节点的配置可在官方网站查阅：[kudu-Master 配置项](https://kudu.apache.org/docs/configuration_reference.html#kudu-master_supported) .

### 0x33 目录规划

```bask
### master 元数据目录
mkdir -p /works/kudu_data/master_data
### table 数据目录
mkdir -p /works/kudu_data/tserver_data
### 日志目录
mkdir -p /works/kudu_data/logs_data

### 配置文件
mkdir -p /works/kudu_data/etc
cd  /works/kudu_data/etc
touch master.gflagfile
vim master.gflagfile
--fromenv=rpc_bind_addresses
--fromenv=log_dir

--fs_wal_dir=/works/kudu_data/master_data
--fs_data_dirs=/works/kudu_data/master_data
--log_dir=/works/kudu_data/master_data/logs

touch tserver.gflagfile
vim tserver.gflagfile
--fromenv=rpc_bind_addresses
--fromenv=log_dir

--fs_wal_dir=/works/kudu_data/tserver_data
--fs_data_dirs=/works/kudu_data/tserver_data
--log_dir=/works/kudu_data/tserver_data/logs
--tserver_master_addrs=192.168.1.251:7051
```

启动

```bash
$ cd /opt/kudu/usr/local
$ sbin/kudu-master -flagfile /works/kudu_data/etc/master.gflagfile 

E20230814 15:28:40.917841 22109 master_main.cc:42] Service unavailable: RunMasterServer() failed: Cannot initialize clock: Error reading clock. Clock considered unsynchronized
### 还记得要安装 ntpd 服务么？ 
$ yum search ntp
$ sudo yum -y install ntp
$ ntpq -p
$ sudo sudo ntpdate -u  119.28.183.184
### 高版本的 ntpd 需要增加下面的配置到 ntp.confs
$ sudo -i
# vim /etc/ntp.conf
server 127.127.1.0 iburst
fudge 127.127.1.0 stratum 8
:wq
# systemctl restart ntpd
# exit
$ 
$ sbin/kudu-master --flagfile=/works/kudu_data/etc/master.gflagfile 

```

UI ： 

http://192.168.1.251:8051/

Static pages not available. Configure KUDU_HOME or use the --webserver_doc_root flag to fix page styling.

```bash
$ nohup sbin/kudu-master --flagfile=/works/kudu_data/etc/master.gflagfile &
$ nohup sbin/kudu-tserver --flagfile=/works/kudu_data/etc/tserver.gflagfile &


### 0x33 配置文件

```sh
# tserver配置

--fs_wal_dir=/home/kudu/kudu_data/tserver/wal
--fs_data_dirs=/home/kudu/kudu_data/tserver/data
--fs_metadata_dir=/home/kudu/kudu_data/tserver/metadata
--log_dir=/home/kudu/kudu_data/tserver/logs
--tserver_master_addrs=112.124.30.113:7051
--webserver_doc_root=/home/kudu/kudu/www

# master 配置
--fs_wal_dir=/home/kudu/kudu_data/master/wal
--fs_data_dirs=/home/kudu/kudu_data/master/data
--fs_metadata_dir=/home/kudu/kudu_data/master/metadata
--log_dir=/home/kudu/kudu_data/master/logs
--webserver_doc_root=/home/kudu/kudu/www
--trusted_subnets=0.0.0.0/0

```

master 和 tserver 的前四个配置是上面提到的基本配置参数。

`--webserver_doc_root` 是为了能在 8051 端口看到 Kudu 的 Master 节点的一些详细信息（tserver 节点的默认端口是 8050）。需要指定静态文件的路径，否则在访问 8051 会报错：

```sh
Static pages not available. Configure KUDU_HOME or use the --webserver_doc_root flag to fix page styling.
```

由报错信息可知，也可以通过设置 `KUDU_HOME` 环境变量来解决，但是笔者在实践时这一方法失效了，所以采取了第 2 种方法。同时这个 WebUI 的访问端口也可以通过 `–-webserver_port` 参数来修改。

tserver 节点需要额外配置一个 `--tserver_master_addrs` 参数，指定一串 master 节点地址的数组。用于将 tserver 连接到 mastser 节点。

`-–trusted_subnets` 参数设置在 master 上，为了让 master 节点能够接受 tserver 节点的连接请求，这里设为 `0.0.0.0/0` 接受所有的 IP。当然也可以只单独添加其他节点的 IP。

启动后进入master节点的8051端口，点击 Tablet Servers 可以看到节点已经连上






## 0x04 参考

- [Kudu 部署指南](https://seyoatda.github.io/tech/kudu-%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97/)
EOF


