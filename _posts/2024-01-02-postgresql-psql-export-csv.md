---
title: PostgreSQL 导入导出 CSV 文件
key: 2024-01-02
tags: postgre psql csv
---

2024 年上班的第一天，貌似一切又都回到了当初，上一天班开会半天，然后 debug 半天。午饭都没有吃上，晚上又开始做 Audi 的东西……

中午媳妇给我发了儿子在家里的儿童游乐场玩游戏的两个视频，看着他玩的非常开心，我心里多少好受了一些，家里有游乐场，又有妈妈姥姥哥哥姐姐陪他玩，看起来比呆在我身边要好。 

今天记录一下 PostgreSQL 数据用 `\copy` 命令或者 `COPY` 语句实现 CSV 文件的导入导出。

<!--more-->

## 0x01 导入


### 一般情况

```sql
-- 使用语句
COPY <table_name> FROM '/path/to/csv/file.csv' WITH CSV HEADER;
-- 使用命令
\copy <table_name> from '/path/to/csv/file.csv' with csv header;

```

- `WITH CSV HEADER` 指定文件格式为 CSV 且首行为标题。
- 使用 `COPY` 语句导入时必须使用管理员账户或者有 `pg_read_server_files` 的权限，
- 使用 `\copy` 命令不受权限影响。

使用管理员账户给当前用户授权

```bash
# 以 postgres 的身份运行 psql
sudo -u postgres psql
postgres=# GRANT pg_read_server_files TO chengchao;
```

### 自定义分隔符

```sql
-- 使用语句
COPY <table_name> FROM '/path/to/csv/file.csv' WITH CSV DELIMITER '|';
-- 使用命令
\copy <table_name> from '/path/to/csv/file.csv' with csv delimiter '|';
```

### 指定导入的列

如果 csv 文件中的列比表中的少， 例如没有 id, 假设 PostgreSQL 使用自增的 ID：

```sql
-- 使用语句
COPY <table_name(col1, col2)> FROM '/path/to/csv/file.csv' WITH CSV DELIMITER '|';
-- 使用命令
\copy <table_name(col1, col2)> from '/path/to/csv/file.csv' with delimiter '|';
```


## 0x02 导出

### 导出全部内容和标题

```sql
-- 使用语句
COPY <table_name> to '~/file.csv' WITH CSV HEADER;
-- 使用命令
\copy <table_name> to '~/file.csv' with csv header;
```

- 对于 `COPY` 语句，需要用户有 `pg_write_server_files` 权限

授权

```bash
sudo -u postgres psql
postgres=# GRANT pg_write_server_files TO chengchao;
```

### 导出部分字段，自定义分隔符，并且不要标题

```sql
\copy <table(col1, col2) to '~/file.csv' with csv delimiter '|';
```

### 导出 SELECT 查询结果

```sql
\copy (SELECT col1, col2 from table_name) to '~/file.csv' with csv header;
```

## 参考和抄袭:

- [https://blog.csdn.net/jiang_huixin/article/details/119813106](https://blog.csdn.net/jiang_huixin/article/details/119813106)


