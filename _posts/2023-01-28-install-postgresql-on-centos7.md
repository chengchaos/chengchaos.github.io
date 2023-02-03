---
title: 在CentOS 7上安装&配置PostgreSQL 12
key: 2023-01-28
tags: CentOS7 PostgreSQL
---

> Suggest search： PostgreSQL CentOS7

> 转载： <https://cloud.tencent.com/developer/article/1592808>

本文主要内容
- PostgreSQL 12 安装(yum)
- PostgreSQL 12 基础配置
- PostgreSQL 12 远程访问配置
- PostgreSQL 基础管理


<!--more-->

## 0x01 PostgreSQL安装

1、导入yum源

```bash
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

1、安装PostgreSQL服务

```bash
sudo yum install -y postgresql12 postgresql12-server
```

> [注释] 安装PostgreSQL 11 命令就是 `yum install postgresql11 postgresql11-server` 安装 PostgreSQL 9.5 命令就是 `yum install postgresql95 postgresql95-server1 依此类推 ...`

2、初始化数据库

```bash
sudo /usr/pgsql-12/bin/postgresql-12-setup initdb 
#Initializing database ... OK
```

3、启动PostgreSQL服务

```bash
#启动PostgreSQL服务
sudo systemctl start postgresql-12

#设置PostgreSQL服务为开机启动
sudo systemctl enable postgresql-12
```


> 9.x 版本的服务名是postgresql-9.x

## 0x02 修改postgres账号密码

PostgreSQL 安装成功之后，会默认创建一个名为 `postgres` 的 Linux 用户，初始化数据库后，会有名为 `postgres` 的数据库，来存储数据库的基础信息，例如用户信息等等，相当于 MySQL 中默认的名为 mysql 数据库。

postgres 数据库中会初始化一名超级用户 postgres。

为了方便我们使用postgres账号进行管理，我们可以修改该账号的密码。


1、进入PostgreSQL命令行
通过su命令切换linux用户为postgres会自动进入命令行

```bash
sudo su - postgres
```

2、启动SQL Shell

```bash
psql
```

3、修改密码

```sql
ALTER USER postgres WITH PASSWORD 'NewPassword';
```

## 0x03 配置远程访问

1、开放端口

```bash
sudo firewall-cmd --add-port=5432/tcp --permanent
sudo firewall-cmd --reload
```

2、修改IP绑定

```bash
# 修改配置文件
vi /var/lib/pgsql/12/data/postgresql.conf

# 将监听地址修改为 *
# 默认listen_addresses配置是注释掉的，所以可以直接在配置文件开头加入该行
listen_addresses='*'
```

3、允许所有IP访问

```bash
 ####### 修改配置文件
vi /var/lib/pgsql/12/data/pg_hba.conf

 ####### 在文件尾部加入
host  all  all 0.0.0.0/0 md5
```

4、重启PostgreSQL服务

```bash
 ####### 重启PostgreSQL服务
sudo systemctl restart postgresql-12
```

配置完成后即可使用客户端进行连接。

## 0x04 PostgreSQL Shell 常用语法示例
启动SQL shell：

```bash
sudo su - postgres
psql
```

1、数据库相关语法示例

```sql
-- 创建数据库
CREATE DATABASE mydb;

-- 查看所有数据库
\l

-- 切换当前数据库
\c mydb

-- 创建表
CREATE TABLE test(id int,body varchar(100));

-- 查看当前数据库下所有表
\d
```

2、用户与访问授权语法示例

```sql
-- 新建用户
CREATE USER test WITH PASSWORD 'test';

-- 赋予指定账户指定数据库所有权限
GRANT ALL PRIVILEGES ON DATABASE mydb TO test;

-- 移除指定账户指定数据库所有权限
REVOKE ALL PRIVILEGES ON DATABASE mydb TO test

-- 权限代码：SELECT、INSERT、UPDATE、DELETE、TRUNCATE、REFERENCES、TRIGGER、CREATE、CONNECT、TEMPORARY、EXECUTE、USAGE
```

## 0x05 备注
1、相关阅读
- <https://www.postgresql.org/download/>
- <https://www.runoob.com/postgresql/postgresql-privileges.html>


EOF


