---
title: OpenSUSE 安装 PostgreSQL 12
key: 2021-11-15
tags: PostgreSQL OpenSUSE
---



自从 MySQL 被 Oracle 收购以后，[PostgreSQL ](https://www.postgresql.org/) 逐渐成为开源关系型数据库的首选。

本文介绍 PostgreSQL 的安装和基本用法，供初次使用者上手。以下内容基于 OpenSUSE 操作系统，其他操作系统实在没有精力兼顾，但是大部分内容应该普遍适用。

<!--more-->

## 安装

```bash
chengchao@web1:~:) sudo zypper search postgresql
chengchao@web1:~:) sudo zypper install postgresql12-server
chengchao@web1:~:) sudo systemctl status postgresql.service 
chengchao@web1:~:) sudo systemctl start postgresql.service 
```

正常情况下，安装完成后，PostgreSQL 服务器会自动在本机的 5432 端口开启。

如果还想安装图形管理界面，可以运行下面命令，但是本文不涉及这方面内容。

```bash
chengchao@web1:~:) zypper search pgadmin
Loading repository data...
Warning: Repository 'openSUSE-15.1-Update-Non-Oss' appears to be outdated. Consider using a different mirror or server.
Warning: Repository 'openSUSE-15.1-Update-Oss' appears to be outdated. Consider using a different mirror or server.
Reading installed packages...

S | Name         | Summary                                   | Type
--+--------------+-------------------------------------------+-----------
  | pgadmin4     | Management tool for PostgreSQL            | package
  | pgadmin4-doc | Documentation for pgAdmin4                | package
  | pgadmin4-web | Web package for pgAdmin4                  | package
  | phpPgAdmin   | Administration of PostgreSQL over the web | package
  | phpPgAdmin   | Administration of PostgreSQL over the web | srcpackage
chengchao@web1:~:) sudo zypper install pgadmin4
```



## 添加新用户和数据库

初次安装后，默认生成一个名为 postgres 的数据库和一个名为 postgres 的数据库用户。这里需要注意的是，同时还生成了一个名为 postgres 的 Linux 系统用户。

下面，我们使用 postgres 用户，来生成其他用户和新数据库。好几种方法可以达到这个目的，这里介绍两种。

### 第一种方法，使用PostgreSQL控制台。

首先，新建一个 Linux 新用户，可以取你想要的名字，这里为 dbuser。

```bash
sudo adduser dbuser
```

然后，切换到postgres用户。

```bash
sudo su - postgres
```

下一步，使用psql命令登录PostgreSQL控制台。

```bash
postgres@web1:~> psql
psql (12.5)
Type "help" for help.

postgres=# 
```



这时相当于系统用户 postgres 以同名数据库用户的身份，登录数据库，这是不用输入密码的。如果一切正常，系统提示符会变为 ··`postgres=#"`，表示这时已经进入了数据库控制台。以下的命令都在控制台内完成。

第一件事是使用\password命令，为postgres用户设置一个密码。

```bash
postgres=# \password postgres
Enter new password: 
Enter it again: 
postgres=# 
```

第二件事是创建数据库~~用户~~角色 dbuser（刚才创建的是Linux系统用户），并设置密码。

```bash
postgres=# DROP ROLE dbuser;
DROP ROLE
postgres=# CREATE ROLE dbuser WITH LOGIN CREATEDB PASSWORD '654321';
```

第三件事是创建用户数据库，这里为exampledb，并指定所有者为dbuser。

```bash
postgres=# CREATE DATABASE exampledb OWNER dbuser;
```

第四件事是将exampledb数据库的所有权限都赋予dbuser，否则dbuser只能登录控制台，没有任何数据库操作权限。

```bash
postgres=# GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
```

最后，使用 `\q` 命令退出控制台（也可以直接按ctrl+D）。

```bash
postgres=# \q
```

### 第二种方法，使用shell命令行。

添加新用户和新数据库，除了在 PostgreSQL 控制台内，还可以在 shell 命令行下完成。这是因为PostgreSQL 提供了命令行程序 `createuser` 和 `createdb`。以新建用户 dbuser2 和数据库exampledb2 为例。

首先，创建数据库用户 dbuser2，并指定其为超级用户。

```bash
chengchao@web1:~:) sudo useradd -m dbuser2
chengchao@web1:~:) sudo -u postgres createuser --superuser dbuser2
```

然后，登录数据库控制台，设置 dbuser2 用户的密码，完成后退出控制台。

```bash
chengchao@web1:~:) sudo -u postgres psql
psql (12.5)
Type "help" for help.

postgres=# \password dbuser2
Enter new password: 
Enter it again: 
postgres=# \q
```

接着，在 shell 命令行下，创建数据库 exampledb2，并指定所有者为 dbuser2。

```bash
chengchao@web1:~:) sudo -u postgres createdb -O dbuser2 exampledb2
```

添加新用户和新数据库以后，就要以新用户的名义登录数据库，这时使用的是 psql 命令

```bash
chengchao@web1:~:) sudo su - dbuser2
dbuser2@web1:~> psql -d exampledb2
psql (12.5)
Type "help" for help.

exampledb2=# create table tb_user (id int, name varchar(20));
CREATE TABLE
exampledb2=# insert into tb_user values (1, 'chengchao');
INSERT 0 1
exampledb2=# select * from tb_user ;
 id |   name    
----+-----------
  1 | chengchao
(1 row)

exampledb2=# 
```



## Java 

增加 pom 的依赖

```xml
        <!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.3.1</version>
        </dependency>
```

写个简单的代码测一下：

```java

    /**
     * 修改配置 pg_hba.conf 添加：
     * # IPv4 local connections:
     * host  all  all  0.0.0.0/0  md5
     * 修改配置 postgresql.conf 添加：
     * listen_addresses = '0.0.0.0'
     *
     * @param args
     * @throws ClassNotFoundException
     */
    public static void main(String[] args) throws ClassNotFoundException {
        Class.forName("org.postgresql.Driver");
        String sql = "select * from tb_user";
        try (Connection conn = DriverManager
                    .getConnection("jdbc:postgresql://47.114.98.28:5432/exampledb2",
                            "dbuser2", "654321");
            Statement stmt = conn.createStatement();
            ResultSet resultSet = stmt.executeQuery(sql) ) {

            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                System.out.println("id : "+ id +", name: "+ name);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



## Go

待补充。



so so .

## 参考

- [PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)
- [openSUSE 安装 PostgreSQL](https://segmentfault.com/a/1190000010032424)
- [Java连接PostgreSQL数据库](https://www.yiibai.com/postgresql/postgresql_java.html)

EOF

