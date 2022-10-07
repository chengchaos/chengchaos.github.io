---

title: bash URL encode && decode
key: 2021-10-04
tags: bash url_encode url_decode
---

这是我在尝试用 bash 编写 cgi 程序的时候遇到的问题，记录在这里。

<!--more-->

## 0x01 URLEncode

URLEncode 又叫 [Percent-encoding](http://en.wikipedia.org/wiki/Percent-encoding)，是非常常见的一种编码方式。

但它绝不是仅仅用在 URL 里面，事实上，表单提交，协议头参数传递，甚至远程过程调用，都有该编码方式的用武之地。虽然它名字叫 URLEncoding，但它绝不是仅仅用在 URL 里面，事实上，表单提交，协议头参数传递，甚至远程过程调用，都有该编码方式的用武之地。

URLEncode 实现很简单，但在 Bash 这样表现能力严重受限的语言里面，真要做起来就不是那么容易了，因此大家一般都会用 Bash 调用一些别的语言的 one-liner 啥的，曲线救国一下。那么，我们能不能不调用任何别的编程语言，仅仅只用标准命令，实现一个 URLEncode 呢？经过多次失败的尝试，查阅多方面资料后，滇狐终于发现了这样的解决方案：

```bash
#!/usr/local/bin/bash

export LANG=C

if [ $# -lt 1 ]; then
  echo Usage: urlencode string
  exit 1
fi

arg="$1"
i="0"
while [ "$i" -lt ${#arg} ]; do
    c=${arg:$i:1}
    if echo "$c" | grep -q '[a-zA-Z/:_\.\-]'; then
        echo -n "$c"
    elif [ "$c" = " " ]; then
        printf "+"
    else
        printf "%%%X" "'$c'"
    fi
    i=$((i+1))
done
```

中间部分的代码非常简单，相信大家都能看明白，主要讲解的是两条小秘技。

第一条秘技是，其实在 Bash 里是有对字符串取子串的操作的，格式是：`${变量:下标:长度}`。求字符串长度也很简单：`${#变量}` 就是。有了求子串和求字符串长度的语句，写一个小循环，以字符为单位遍历一个字符串是很简单的操作，没有任何难度。

第二条秘技是，Bash 取子串操作是以字符为单位的，在 UTF-8 环境下，一个字符可能包含多个字节，这给 URLEncode 带来很大困难。为了解决这个问题，我们在脚本一开头处搞了个小诡计：把 LANG 强行设置为 C。这样一来，Bash 就无法知道我们当前处于什么 locale 下，也就无法把字节拼装成为字符了，于是我们就如愿以偿地获得了每个字节，接下来要做编码那就是轻而易举的事情了。

总之，这是 URLEncode 的一个非常低效率的实现，但不管怎么样，至少功能是没问题的，先屯着，说不定什么时候就会用得着呢，对吧？

> 原文： <http://edyfox.codecarver.org/html/bash_url_encode.html>

这个程序在我的 MAC 上执行是有问题的，但是先放在这里以后调整一下。

## 0x02 URL_Decode

```bash
function urldecode() {
  # urldecode <string>
  local url_encoded="${1//+/ }"
  printf '%b' "${url_encoded//%/\\x}"
}
```

emmmmmn, 这个原理目前还不知道，不过这个是好用的。

> 原文： <https://blog.longwin.com.tw/2017/12/bash-shell-curl-send-urlencode-args-2017/>

EOF
