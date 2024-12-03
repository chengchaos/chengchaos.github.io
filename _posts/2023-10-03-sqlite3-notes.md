---
title: SQLite3 Notes
key: 2023-10-03
tags: SQLite3 
---

[!Systemd](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016030701.gif)

> 本文主要记录了怎样快速使用 SQLite3，没有别的意思。

Search suggest: SQLite3

<!--more-->
## 0x01 Install

下载页面: [http://www.sqlite.org/download.html](http://www.sqlite.org/download.html)

## 0x02 使用

获取帮助

```sh
$ sqlite3
SQLite version 3.37.0 2021-12-09 01:34:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.




sqlite> .help

sqlite>.show
     echo: off
  explain: off
  headers: off
     mode: column
nullvalue: ""
   output: stdout
separator: "|"
    width:
sqlite>

```

格式化输出

您可以使用下列的点命令来格式化输出为本教程下面所列出的格式：

```sh
sqlite>.header on
sqlite>.mode column
sqlite>.timer on
sqlite>
```




## Appendix

以下是参考链接。

- [Runoob.com SQLite 教程](https://www.runoob.com/sqlite/sqlite-commands.html)

EOF
