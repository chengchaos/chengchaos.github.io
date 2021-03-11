---
title: CentOS 7 安装 MySQL 8
key: 2021-02-09
tags: linux centos7 mysql8
---

总™忘！

<!--more-->

## 使用 yum 安装

去 MySQL 官方网站，就是下面的这个连接地址：

[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)

下载：

[https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)

```bash
wget -c https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
sudo rpm -ihv mysql80-community-release-el7-3.noarch.rpm
sudo yum -y update

```

但是在阿里云的主机上连不上这个服务。

## 使用 tar 包安装

```bash

sudo yum remove mysql-libs
sudo yum install libaio


sudo rpm -ihv mysql-community-common-8.0.23-1.el7.x86_64.rpm
sudo rpm -ihv mysql-community-client-plugins-8.0.23-1.el7.x86_64.rpm
sudo rpm -ihv mysql-community-libs-8.0.23-1.el7.x86_64.rpm
sudo rpm -ihv mysql-community-client-8.0.23-1.el7.x86_64.rpm
sudo rpm -ihv mysql-community-server-8.0.23-1.el7.x86_64.rpm

sudo vim /etc/my.cnf
```

## 修改配置

在 my.cnf 添加如下内容：

```ini
cat my.cnf
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html
[client]
port                            = 3306

[mysql]
prompt                          = \u@\h [\d]>\_
no_auto_rehash
# default-character-set = utf8mb4 会导致控制台不能输入中文
# default-character-set           = utf8mb4


[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

default_authentication_plugin  = mysql_native_password

character-set-client-handshake = FALSE
character-set-server           = utf8mb4
collation-server               = utf8mb4_unicode_ci
#init_connect                   = 'SET NAMES utf8mb4'

```

启动服务以及修改密码创建用户。

```bash
systemctl enable mysqld
systemctl start mysqld

cat /var/log/mysqld.log  | grep root
mysql -u root -p

mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> alter user 'root'@'localhost' identified with mysql_native_password by '{您的密码}';
Query OK, 0 rows affected (0.01 sec)

mysql> create user '{新用户名}'@'%' identified with mysql_native_password by '{他的密码}';
Query OK, 0 rows affected (0.00 sec)

mysql> grant all privileges on *.* to 'modern'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit;
Bye

mysql> create database sina default character set utf8mb4 collate utf8mb4_unicode_ci;
```

## MySQL 的优化

参考知乎《MySQL配置性能优化》：[MySQL配置性能优化](https://zhuanlan.zhihu.com/p/30075280)  [https://zhuanlan.zhihu.com/p/30075280](https://zhuanlan.zhihu.com/p/30075280)

### 1、mysql一些主要配置项介绍

- innodb_buffer_pool_size

这是你安装完InnoDB后第一个应该设置的选项。缓冲池是数据和索引缓存的地方：这个值越大越好，这能保证你在大多数的读取操作时使用的是内存而不是硬盘。如果是纯数据库，可以设置到机器内存 70%（官网建议）.

- innodb_log_file_size

这是redo日志的大小。redo日志被用于确保写操作快速而可靠并且在崩溃时恢复。一直到MySQL 5.1，它都难于调整，因为一方面你想让它更大来提高性能，另一方面你想让它更小来使得崩溃后更快恢复。

- max_connections

如果你经常看到‘Too many connections’错 误，是因为max_connections的值太低了。这非常常见因为应用程序没有正确的关闭数据库连接，你需要比默认的151连接数更大的值。 max_connection值被设高了(例如1000或更高)之后一个主要缺陷是当服务器运行1000个或更高的活动事务时会变的没有响应。在应用程序 里使用连接池或者在MySQL里使用进程池有助于解决这一问题。

- innodb_flush_log_at_trx_commit

默认值为1，表示 InnoDB 完全支持 ACID 特性。当你的主要关注点是数据安全的时候这个值是最合适的，比如在一个主节点上。但是对于磁盘（读写）速度较慢的系统，它会带来很巨大的开销，因为每次将改变 flush 到 redo 日志都需要额外的 fsyncs。将它的值设置为2会导致不太可靠 （unreliable）因为提交的事务仅仅每秒才 flush 一次到 redo 日志，但对于一些场景是可以接受的，比如对于主节点的备份节点这个值是可以接受的。如果值为 0 速度就更快了，但在系统崩溃时可能丢失一些数据：只适用于备份节点。

- innodb_log_buffer_size

这项配置决定了为尚未执行的事务分配的缓存。其默认值（1MB）一般来说已经够用了，但是如果你的事务中包含有二进制大对象或者大文本字段的话，这点缓存很快就会被填满并触发额外的 I/O 操作。看看 Innodb_log_waits 状态变量，如果它不是 0，增加 innodb_log_buffer_size。

- query_cache_size/query_cache_type

query cache（查询缓存）是一个众所周知的瓶颈，甚至在并发并不多的时候也是如此。 最佳选项是将其从一开始就停用，设置 `query_cache_size = 0`（现在MySQL 5.6的默认值）并利用其他方法加速查询：优化索引、增加拷贝分散负载或者启用额外的缓存（比如 memcache 或 redis ）。如果你已经为你的应用启用了 query cache 并且还没有发现任何问题，query cache 可能对你有用。这时如果你想停用它，那就得小心了。

- innodb_lock_wait_timeout

innodb 引擎，当存在锁竞争时等待的时间。

### 2、配置上做一些优化

**当然下面是我们服务器的优化，因服务器配置不同，读写等进行不同配置优化。**

1．`innodb_flush_log_at_trx_commit` 设置为 0，提升 MySQL 写性能，但是如果出现宕机，存在丢失数据的风险，如果可以，最好修改为 2，然后使用读写分离等手段来提升 MySQL 整体性能。

2．调整了 MySQL 的事务隔离级别，由默认的 Repeatable read 调整到 Read committed 。

3．修改了 `innodb_lock_wait_timeout`，由默认的 50 to 30  。

4．修改 `innodb_log_file_size`，5M to 512M，对于写入负载高的场景，参考值：64~512MB。

5．修改 `innodb_buffer_pool_size`，原先是 8G，换新服务器后修改为 24G，可以通过缓存命中率来判断这个值是否够。

6．修改 `query_cache_type` 为 2，默认情况下不使用 MySQL 缓存，除非显示指定。

## 在 Windows 上安装

先去下载 [https://dev.mysql.com/downloads/file/?id=501136](https://dev.mysql.com/downloads/file/?id=501136)

解压缩。

新建一个 data 目录，一个 my.ini 文件。

```ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=C:\\works\\server\\mysql-8.0.23-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\\works\\server\\mysql-8.0.23-winx64\\data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10
# 服务端使用的字符集默认为UTF8
character-set-server           = utf8mb4
collation-server               = utf8mb4_unicode_ci
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8
```

```bat
➜  mysql-8.0.23-winx64 .\bin\mysqld.exe --initialize --user=mysql --console 
2021-03-11T06:01:10.477189Z 0 [System] [MY-013169] [Server] C:\works\server\mysql-8.0.23-winx64\bin\mysqld.exe (mysqld 8.0.23) initializing of server in progress as process 26636
2021-03-11T06:01:10.505445Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-03-11T06:01:11.356352Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-03-11T06:01:13.098050Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: JUL?=wM7s;jK
➜  mysql-8.0.23-winx64 .\bin\mysqld.exe --console
2021-03-11T06:02:55.071024Z 0 [System] [MY-010116] [Server] C:\works\server\mysql-8.0.23-winx64\bin\mysqld.exe (mysqld 8.0.23) starting as process 59052
2021-03-11T06:02:55.106239Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-03-11T06:02:55.548667Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-03-11T06:02:55.703004Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060
2021-03-11T06:02:55.895620Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2021-03-11T06:02:55.897703Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2021-03-11T06:02:55.996668Z 0 [System] [MY-010931] [Server] C:\works\server\mysql-8.0.23-winx64\bin\mysqld.exe: ready for connections. Version: '8.0.23'  socket: ''  port: 3306  MySQL Community Server - GPL.

```

### 安装为系统服务

- 使用命令： `mysqld -install` 进行服务的添加 。
- 使用命令： `net start mysql` 启动服务 。

- 使用命令： `mysql -u root -p` 进行登录数据库，这时提示需要密码，然后就是用你上面的密码登录 。
- 登录成功后，使用命令： `ALTER USER root@localhost IDENTIFIED  BY '123456';` 修改密码为：123456 。

### 赠送：8.0之前版本，忘记密码修改方法

进入 bin 目录：`mysqld --skip-grant-tables`

重新在开一个cmd窗口，进入 bin 目录：mysql 就进入登陆状态了

- 5.7.22修改密码语句：`update user set authentication_string=password('123456') where user='root' and host='localhost';`
- 5.6.修改密码语句：`update user set password=password('123456') where user='root' and host='localhost';`  (我没有实验过，网上都是这么写的)

## Redis （赠送的)

修改 /etc/sysctl.conf 文件添加如下内容：

```conf

# for redis:
vm.overcommit_memory = 1  # default 0
net.core.somaxconn = 1024 # default 128
```

修改 /etc/rc.local 文件添加

```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

```

EOF

---

Power by TeXt.
