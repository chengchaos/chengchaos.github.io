---
title: CentOS 7 安装 MySQL 8
key: 2021-02-09
tags: linux centos7 mysql8
---

总(Teme)忘！

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

修改 my.cnf 添加如下内容：

```ini
[mysql]
default_authentication_plugin=mysql_native_password
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

```

## Redis

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
