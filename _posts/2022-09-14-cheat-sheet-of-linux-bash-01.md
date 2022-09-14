---
title: Linux 编写 Bash 脚本作弊条 (001)
key: 2022-09-14
tags: Linux Bash
---

As the title says.

<!--more-->

## 0x01 脚本设置

1, 当运行失败时使脚本退出

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

如果 mydir 变量因为某种原因没有赋值，这条命令就会变成 `rm -rf /data` 。这就悲剧了, 使用 `-u 可以避免这种情况。

但有时候在已经设置了-u 后，某些地方还是希望能把未定义变量展开为空串，可以这样写：

```bash
${SOME_VAR:-} ## BASH 
```

展开语法参考连接: <https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html>

**`-o pipefail`** : 只要管道中的一个子命令失败，整个管道命令就失败。

`pipefail` 与 `-e` 结合使用的话，就可以做到管道中的一个子命令失败，就退出脚本。

## 0x02 静态变量

静态变量不会改变；它的值一旦在脚本中定义后不能被修改：

```bash
readonly passwd_file="/etc/passwd"
readonly group_file="/etc/group"
```

环境变量用大写字母命名，而自定义变量用小写.

## 0x03 防止重复运行

在一些场景中，我们通常不希望一个脚本有多个实例在同时运行。比如用 crontab 周期性运行脚本时，有时不希望上一个轮次还没运行完，下一个轮次就开始运行了。这时可以用 `flock` 命令来解决。 `flock` 通过文件锁的方式来保证独占运行，并且还有一个好处是进程退出时，文件锁也会自动释放，不需要额外处理。

用法 1：假设你的入口脚本是 myscript.sh，可以新建一个脚本，通过 flock 来运行它：

```bash
# flock --wait 超时时间 -e 锁文件 -c "要执行的命令"
# 例如：
flock --wait 5 -e "lock_myscript"  -c "bash myscript.sh"
```

用法 2：也可以在原有脚本里使用 flock。可以把文件打开为一个文件描述符，然后使用 flock 对它上锁（flock 可以接受文件描述符参数）。

```bash
exec 123<>lock_myscript   # 把lock_myscript打开为文件描述符123
flock  --wait 5  123 || { echo 'cannot get lock, exit'; exit 1; }
```
