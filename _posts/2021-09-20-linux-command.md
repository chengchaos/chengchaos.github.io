---
title: Linux 常用名利
key: 2021-09-20
tags: linux tar
---

- tar 压缩/解压缩命令详解
- 获取进程的 PID

<!--more-->

## tar 压缩/解压缩命令详解

### 命令

- `-c` 创建压缩包
- `-x, --extract, --get` 解压缩包
- `-t, --list` 查看压缩包中的内容
- `-r, --append` 向压缩包末尾追加文件
- `-u, --update` 更新压缩包中的文件

这五个是独立的命令，压缩或解压都要用到其中一个，可以和别的命令连用但只能用其中一个。

下面的参数是根据需要在压缩或解压档案时可选的。

### 参数

`-z`：有gzip属性的

`-j`：有bz2属性的

`-Z`：有compress属性的

`-v`：显示所有过程

`-O`：将文件解开到标准输出

参数 `-f` 是必须的

`-f`: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

```bash
# tar -cf all.tar *.jpg 这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
# tar -rf all.tar *.gif 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。 
# tar -uf all.tar logo.gif 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。 
# tar -tf all.tar 这条命令是列出all.tar包中所有文件，-t是列出文件的意思 
# tar -xf all.tar 这条命令是解出all.tar包中所有文件，-x是解开的意思
```

### 示例

查看

`tar -tf aaa.tar.gz` 在不解压的情况下查看压缩包的内容

压缩

`tar –cvf jpg.tar *.jpg` //将目录里所有jpg文件打包成tar.jpg

`tar –czf jpg.tar.gz *.jpg` //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz

`tar –cjf jpg.tar.bz2 *.jpg` //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2

`tar –cZf jpg.tar.Z *.jpg`   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z

解压

`tar –xvf file.tar` //解压 tar包

`tar -xzvf file.tar.gz` //解压tar.gz

`tar -xjvf file.tar.bz2`   //解压 tar.bz2tar –xZvf file.tar.Z //解压tar.Z

总结

1、`*.tar` 用 `tar –xvf` 解压

2、`*.gz` 用 `gzip -d` 或者 `gunzip` 解压

3、`*.tar.gz` 和 `*.tgz` 用 `tar –xzf` 解压

4、`*.bz2` 用 `bzip2 -d` 或者用 `bunzip2` 解压

5、`*.tar.bz2` 用 `tar –xjf` 解压

6、`*.Z` 用 `uncompress` 解压

7、`*.tar.Z` 用 `tar –xZf` 解压

## 获取进程的 PID

### 交互式 Bash Shell 获取进程 pid

在已知进程名(name)的前提下，交互式 Shell 获取进程 pid 有很多种方法，典型的通过 grep 获取 pid 的方法为（这里添加 -v grep是为了避免匹配到 grep 进程）：

`ps -ef | grep "name" | grep -v grep | awk '{print $2}'`

或者不使用 grep（这里名称首字母加[]的目的是为了避免匹配到 awk 自身的进程）：

`ps -ef | awk '/[n]ame/{print $2}'`

如果只使用 x 参数的话则 pid 应该位于第一位：

`ps x | awk '/[n]ame/{print $1}'`

最简单的方法是使用 pgrep：

`pgrep -f name`

如果需要查找到 pid 之后 kill 掉该进程，还可以使用 pkill：

`pkill -f name`

如果是可执行程序的话，可以直接使用 pidof

`pidof name`

### 获取 Shell 脚本自身进程 PID

这里涉及两个指令：

1. `$$` ：当前 Shell 进程的 pid
2. `$!` ：上一个后台进程的 pid

可以使用这两个指令来获取相应的进程 pid。例如，如果需要获取某个正在执行的进程的 pid（并写入指定的文件）：

```bash
myCommand && pid=$!
myCommand & echo $! >/path/to/pid.file
```

> 注意，在脚本中执行 $! 只会显示子 Shell 的后台进程 pid，如果子 Shell 先前没有启动后台进程，则没有输出。

### 查看指定进程是否存在

在获取到 pid 之后，还可以根据 pid 查看对应的进程是否存在（运行），这个方法也可以用于 kill 指定的进程。

```bash
if ps -p $PID > /dev/null
then
   echo "$PID is running"
   # Do something knowing the pid exists, i.e. the process with $PID is running
fi
```

## sed

- `-r, --regexp-extended` use extended regular expressions in the script.
- `-i[SUFFIX], --in-place[=SUFFIX]` edit files in place (makes backup if SUFFIX supplied)
- `-c, --copy` use copy instead of rename when shuffling files in -i mode

## 压缩文件到指定目录

```bash
#!/usr/bin/env bash

BASE_DIR=$(cd $(dirname $0) && pwd)
BASE_INPUT="/home/chengchao/temp/somelog"
BASE_OUTPUT="/home/chengchao/temp/somegz"

echo "BASE_DIR=> $BASE_DIR"
echo "BASE_INPUT => $BASE_INPUT"
echo "BASE_OUTPUT => $BASE_OUTPUT"

cd $BASE_INPUT
echo "chenge dir to $BASE_INPUT"

for mydir in $(ls -d */ | sed -r "s/(.*)\//\1/g"); do
        cd $mydir
        echo "    chenge dir to $mydir"
        for myfile in $(ls *.log) ; do
                echo "        $mydir / $myfile"
                FILE_YEAR=$(echo $myfile | sed -r "s/.*([[:digit:]]{4})-([[:digit:]]{2})-([[:digit:]]{2}).*/\1/g")
                FILE_MONTH=$(echo $myfile | sed -r "s/.*([[:digit:]]{4})-([[:digit:]]{2})-([[:digit:]]{2}).*/\2/g")
                TARGET_DIR="${BASE_OUTPUT}/${mydir}/${FILE_YEAR}/${FILE_MONTH}"
                mkdir -p ${TARGET_DIR}
                tar -zcvf ${TARGET_DIR}/${myfile}.tar.gz ${myfile} --remove-files
                echo "        tar ${myfile} ... is $?"
        done
        cd -
done

```

EOF
