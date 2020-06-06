---
title: MySQL-Change-charset
key: 20181015
tags: MySQL8 utf8mb4
---





MySQL 修改默认字符集和排序规则……

<!--more-->



## ERROR 1820 (HY000)

新装的 mysql8

```sql
mysql> show databases;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> alter user user() identified by "NpdKM0Eky/_f"
    -> ;
Query OK, 0 rows affected (0.04 sec)
```



## 一、查看当前数据库字符集和排序规则




```bash
-- 查看字符集:
root@localhost [(none)]> SHOW VARIABLES LIKE 'collation_%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_0900_ai_ci |
| collation_database   | utf8mb4_0900_ai_ci |
| collation_server     | utf8mb4_0900_ai_ci |
+----------------------+--------------------+
3 rows in set (0.00 sec)
-- 查看排序规则：
root@localhost [(none)]> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8mb4                          |
| character_set_connection | utf8mb4                          |
| character_set_database   | utf8mb4                          |
| character_set_filesystem | binary                           |
| character_set_results    | utf8mb4                          |
| character_set_server     | utf8mb4                          |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/share/mysql/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.00 sec)
```

## 二、修改 MySQL 的 my.cnf/my.ini 文件




```bash

[client]
port                            = 3306
socket                          = /tmp/mysql.sock
default-character-set           = utf8mb4

[mysql]
prompt                          = \u@\h [\d]>\_
no_auto_rehash
default-character-set           = utf8mb4

[mysqld]
init_connect                    = 'SET collation_connection = utf8mb4_general_ci'
init_connect                    = 'SET NAMES utf8mb4'
character_set_server            = utf8mb4
collation-server                = utf8mb4_general_ci
skip-character-set-client-handshake
```

再次查看：

```bash
root@localhost [(none)]> show variables like 'collation_%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_general_ci |
| collation_database   | utf8mb4_general_ci |
| collation_server     | utf8mb4_general_ci |
+----------------------+--------------------+
3 rows in set (0.01 sec)

root@localhost [(none)]> show variables like 'character%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8mb4                          |
| character_set_connection | utf8mb4                          |
| character_set_database   | utf8mb4                          |
| character_set_filesystem | binary                           |
| character_set_results    | utf8mb4                          |
| character_set_server     | utf8mb4                          |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/share/mysql/charsets/ |
+--------------------------+----------------------------------+
8 rows in set (0.01 sec)

root@localhost [(none)]>
root@localhost [(none)]> quit
Bye
```

以上。

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
