---
title: Linux 编写 Bash 脚本作弊条 (001)
key: 2022-09-14
tags: Linux Bash
---

As the title says.

<!--more-->

## 0x01 脚本调试

有时即使某些​​命令​​运行失败，bash 可能继续去执行脚本，这样就影响到脚本的其余部分（会最终导致逻辑错误）。用下面的行的方式在遇到​ ​命令​​失败时来退出脚本执行：

```bash
#/usr/bin/env bash
set -xeuo pipefail
# -x 在执行每一个命令之前把经过变量展开之后的命令打印出来。
# 这个对于 debug 脚本、输出 Log 时非常有用。正式运行的脚本也可以不加。
# -e 遇到一个命令失败（返回码非零）时，立即退出
### set -o errexit
### set -e
# 如果有未设置的变量脚本退出执行
### set -o nounset
### set -u

```

参数解释

**`-x`** : 在执行每一个命令之前把经过变量展开之后的命令打印出来。

这个对于 debug 脚本、输出 Log 时非常有用。正式运行的脚本也可以不加。

**`-e`** : 遇到一个命令失败（返回码非零）时，立即退出。或者 `set -o errexit`

bash 跟其它的脚本语言最大的不同点之一，应该就是遇到异常时继续运行下一条命令。这在很多时候会遇到意想不到的问题。加上 `-e` ，会让 bash 在遇到一个命令失败时，立即退出。

如果有时确实需要忽略个别命令的返回码，可以用 `|| true` 。如：

```bash
some_cmd || true   ## 即使 some_cmd 失败仍然继续运行
some_cmd || RET=$? ## 或者收集返回码供后面代码判断
```

**`-u`** : 试图使用未定义的变量，就立即退出。或者 `set -o nounset`

如果在 bash 里使用一个未定义的变量，默认是会展开成一个空串。有时这种行为会导致问题，比如：

```bash
rm -rf $mydir/data
```

如果 mydir 变量因为某种原因没有赋值，这条命令就会变成 `rm -rf /data` 。这就悲剧了, 使用 `-u` 可以避免这种情况。

但有时候在已经设置了 `-u` 后，某些地方还是希望能把未定义变量展开为空串，可以这样写：

```bash
${SOME_VAR:-} ## BASH 
```

展开语法参考连接: <https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html>

**`-o pipefail`** : 只要管道中的一个子命令失败，整个管道命令就失败。

`pipefail` 与 `-e` 结合使用的话，就可以做到管道中的一个子命令失败，就退出脚本。

## 静态变量

静态变量不会改变；它的值一旦在脚本中定义后不能被修改：

```bash
readonly passwd_file="/etc/passwd"
readonly group_file="/etc/group"
```

环境变量用大写字母命名，而自定义变量用小写.

## 防止重复运行

在一些场景中，我们通常不希望一个脚本有多个实例在同时运行。比如用 crontab 周期性运行脚本时，有时不希望上一个轮次还没运行完，下一个轮次就开始运行了。这时可以用 `flock` 命令来解决。 `flock` 通过文件锁的方式来保证独占运行，并且还有一个好处是进程退出时，文件锁也会自动释放，不需要额外处理。

用法 1：假设你的入口脚本是 myscript.sh，可以新建一个脚本，通过 flock 来运行它：

```bash
# flock --wait 超时时间 -e 锁文件 -c "要执行的命令"
# 例如：
flock --wait 5 -e "lock_myscript" -c "bash myscript.sh"
```

用法 2：也可以在原有脚本里使用 flock。可以把文件打开为一个文件描述符，然后使用 flock 对它上锁（flock 可以接受文件描述符参数）。

```bash
exec 123<>lock_myscript   # 把lock_myscript打开为文件描述符123
flock  --wait 5  123 || { echo 'cannot get lock, exit'; exit 1; }
```

## 意外退出时杀掉所有子进程

我们的脚本通常会启动好多子脚本和子进程，当父脚本意外退出时，子进程其实并不会退出，而是继续运行着。如果脚本是周期性运行的，有可能发生一些意想不到的问题。

在 stackoverflow 上找到的一个方法，原理就是利用 trap 命令在脚本退出时 kill 掉它整个进程组。把下面的代码加在脚本开头区，实测管用：

```bash
trap "trap - SIGTERM && kill -- -$$" SIGINT SIGTERM EXIT
```

不过如果父进程是用 SIGKILL (kill -9) 杀掉的，就不行了。因为 SIGKILL 时，进程是没有机会运行任何代码的。

## timeout 限制运行时间

有时候需要对命令设置一个超时时间。这时可以使用 timeout 命令，用法很简单：

```bash
timeout 600s some_command arg1 arg2
```

命令在超时时间内运行结束时，返回码为 0，否则会返回一个非零返回码。

timeout 在超时时默认会发送 TERM 信号，也可以用 -s 参数让它发送其它信号。

## 连续管道时，考虑使用 tee 将中间结果落盘，以便查问题

有时候我们会用到把好多条命令用管道串在一起的情况。如 cmd1 | cmd2 | cmd3 | ...这样会让问题变得难以排查，因为中间数据我们都看不到。

如果改成这样的格式：

```bash
cmd1 > out1.dat
cat out1 | cmd2 > out2.dat
cat out2 | cmd3 > out3.dat
```

性能又不太好，因为这样 cmd1, cmd2, cmd3 是串行运行的，这时可以用 tee 命令：

```bash
cmd1 | tee out1.dat | cmd2 | tee out2.dat | cmd3 > out3.dat
```

## 带返回值的函数

函数最神奇的功能之一是它们允许将数据从一个函数传递到另一个函数。它在多种情况下很有用。查看下一个示例。

```bash
#!/bin/bash

function Greet() {
    str="你好$name，是什么把你带到Linux公社?"
    echo $str
}

echo "->你叫什么名字？"
read name

val=$(Greet)
echo -e "-> $val"

```

## 使用管道连接函数

虽然每个人都同意需要使用 `stdin` , 但是这里缺少的答案是 `/dev/stdin` 文件的实际用法.

```bash
jc_hms() {
    declare -i i=${1:-$(</dev/stdin>)};
    declare hr=$(($i/3600)) min=$(($i/60%60)) sec=$(($i%60))
    printf "%02d:%02d:%02d\n" "$hr" "$min" "$sec";
}
jc_hms 7800
02:10:00
echo 7800 | jc_hms
02:10:00
```

## cURL 入门

避免 curl 命令显示进度信息: `curl URL --silent`

显示进度条: `curl URL -o index.html --progress`

将下载数据写入文件: `curl URL --silent -o [文件名]`

断点续传: `curl -c - URL`

Referrer: `curl --referer REFERRER_URL TARGET_URL`

携带 Cookie: `curl URL --cookie "user=chengchao;pass=hack`

将 Cookie 存成文件: `curl URL --cookie-jar cookie_file`

HTTP Header: `curl -H "Host:www.slynux.org" -H "Accept-language: en" URL`

限定可用的带宽: `curl URL --limit-rate 20k`

进行认证:

```bash
curl -u user:pass URL
curl -u user URL # 提示后输入密码
```

只打印头部信息:

```bash
curl -I URL
curl --head
```

## 文件切分/合并

```bash
split [-b <字节/K/M/G/T> ] [-<行数>] [-l 行数] [-a <后缀长度>] [-d 使用数字后缀]
## 合并 
## Linux
cat xaa xab > king_of_ring_merge.avi
## Windows
copy /b xaa + xab  king_of_ring_merge.avi
```

EOF
