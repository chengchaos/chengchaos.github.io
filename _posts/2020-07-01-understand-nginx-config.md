---
title: 理解 Nginx 的配置文件
key: 2020-07-01
tags: nginx
---


Nginx 的配置文件使用了自定义的一套语法，完全可以把它理解成一个小型的变成语言。



<!--more-->

## 要点：

- 使用 `#` 开始一个注释行
- 使用单引号或者双引号定义字符串，允许使用斜杠  `\` 转义字符。
- 使用 `$var` 可以引用预定义的一些变量。
- 配置指令以分号结束，可以接收多个参数，使用空白字符分割。
- 配置块 （block）是特殊的配置指令，它有一个 `{}` 参数且无需分号结束，参数里面可以书写多个配置指令，配置块允许嵌套；
- 使用 `include` 指令包含其他配置文件，支持 `*` 通配符。
- 不能识别或者错误的配置指令会导致 Nginx 启动失败。


## 变量

变量是 Nginx 内部保存的运行时 HTTP/TCP 请求相关数据，可以在编写配置文件时任意引用，使得编写 Nginx 配置文件更像是在写程序。

在配置文件里使用变量需要以 `$` 开头，常见的变量：


- `$uri` 当钱请求的 URI 不包含 ？ 后面的参数
- `$args` ： 当前请求的参数
- `$arg_xxx` ： 当前请求里的某个参数，xxx 是参数的名字
- `$http_xxx` ： 当前请求里的 xxx 头对应的值
- `$sent_http_xxx` ： 返回给客户端的响应头部对应的值
- `$remote_addr` ： 客户端 IP 地址。

内置变量非常多，详细列表可以参考 Nginbx 的官方文档。

Nginx 也允许使用自定义变量，最常用的就是 `set` ：

```
set $max_size 10000; 
```


## HTTP 服务

### location 配置


location 指令定义 Web 服务的接口。

location 是一个配置块，除了 `{}` 外还有其他参数：

```
location [ = | ~ | ~* | ^~ ] uir {
  # ....
}
```

location 使用 uri 参数匹配 HTTP 请求里的 URI，默认是**前缀匹配**：

- `=` : RUI 必须完全匹配
- `~` : 大小写敏感正则匹配
- `~*` : 大小写不敏感正则匹配
- `^~` : 前缀匹配














EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>




