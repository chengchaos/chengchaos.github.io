---
title: Linux 下将文件打包、压缩并分割成指定大小
key: 2022-08-26
tags: linux tar split
---

## tar 压缩一个文件

```bash
[chengchao@c7h00 ~]$ ls
jdk
[chengchao@c7h00 ~]$ tar -czvf jdk.tar.gz jdk
jdk/
jdk/jdk-8u301-windows-x64.exe
[chengchao@c7h00 ~]$ ls
jdk  jdk.tar.gz

```

说明: 将 jdk 目录打包并压缩为 jdk.tar.gz 文件.

<!--more-->

文件太大需要分割:

```bash

[chengchao@c7h00 ~]$ split -b 30M -d -a 1 jdk.tar.gz jdk.tar.gz.
[chengchao@c7h00 ~]$ ls -lh
total 335M
drwxrwxr-x 2 chengchao chengchao   39 Aug 26 17:04 jdk
-rw-rw-r-- 1 chengchao chengchao 168M Aug 26 17:04 jdk.tar.gz
-rw-rw-r-- 1 chengchao chengchao  30M Aug 26 17:08 jdk.tar.gz.0
-rw-rw-r-- 1 chengchao chengchao  30M Aug 26 17:08 jdk.tar.gz.1
-rw-rw-r-- 1 chengchao chengchao  30M Aug 26 17:08 jdk.tar.gz.2
-rw-rw-r-- 1 chengchao chengchao  30M Aug 26 17:08 jdk.tar.gz.3
-rw-rw-r-- 1 chengchao chengchao  30M Aug 26 17:08 jdk.tar.gz.4
-rw-rw-r-- 1 chengchao chengchao  18M Aug 26 17:08 jdk.tar.gz.5

```

说明:

- `-b 30M` 表示设置每个分割包的大小，单位还是可以 `k`
- `-d` 参数指定生成的分割包后缀为数字的形式
- `-a x` 来设定序列的长度(默认值是2)，这里设定序列的长度为1

EOF

## 压缩并分割

以上两部可以合并为一步

```bash
[chengchao@c7h00 ~]$ tar -czv jdk | split -b 30M -d -a1 - jdk.tar.gz.
jdk/
jdk/jdk-8u301-windows-x64.exe
[chengchao@c7h00 ~]$ ls -lh
total 168M
drwxrwxr-x 2 chengchao chengchao  39 Aug 26 17:04 jdk
-rw-rw-r-- 1 chengchao chengchao 30M Aug 26 17:13 jdk.tar.gz.0
-rw-rw-r-- 1 chengchao chengchao 30M Aug 26 17:13 jdk.tar.gz.1
-rw-rw-r-- 1 chengchao chengchao 30M Aug 26 17:13 jdk.tar.gz.2
-rw-rw-r-- 1 chengchao chengchao 30M Aug 26 17:13 jdk.tar.gz.3
-rw-rw-r-- 1 chengchao chengchao 30M Aug 26 17:13 jdk.tar.gz.4
-rw-rw-r-- 1 chengchao chengchao 18M Aug 26 17:13 jdk.tar.gz.5
```

说明: 其中 `-` 参数表示将所创建的文件输出到标准输出上

## 解压缩

```bash
mkdir jdk2
tar -zxvf jdk.tar.gz -C jdk2
```

## 解压缩分割后的文件

```bash
[chengchao@c7h00 ~]$ cat jdk.tar.gz.* | tar -zxv -C jdk2
jdk/
jdk/jdk-8u301-windows-x64.exe

```

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
