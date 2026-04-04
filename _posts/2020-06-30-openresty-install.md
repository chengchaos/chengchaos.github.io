---
title: OpenResty 的安装
key: 2020-06-30
tags: OpenResty
---



OpenResty 源码可以在官网下载：

https://www.openresty.org

也可以下载已经编译好的二进制版本。


<!--more-->

## 安装

**这里记录在 CentOS 中的安装**


首先添加 `openresty` yum 仓库：

```bash
# add the yum repo:
$ wget https://openresty.org/package/centos/openresty.repo
$ sudo mv openresty.repo /etc/yum.repos.d/
# update the yum index:
$ sudo yum check-update
$ sudo yum install openresty
$ sudo yum install openresty-resty
$ sudo yum install openresty-opm


```

默认情况下，将会安装在 ： `/usr/local/openresty/` 目录中。

`bin` 目录中包括以下常见命令：

- `openresty` : 启动 OpenResty 服务 （我觉得它就是 nginx）
- `opm` : 组件管理工具，用来只能装各种功能组件
- `resty` : 命令行工具，可以直接执行 Lua 程序。
- `restydoc` : 参考手册。

`lualib` 目录中存放着 OpenResty 自带的 Lua 组件，例如 `lua_cjson`,  `lua_core` 等。


在 CentOS 7 中，默认安装了名为 openresty 的系统服务：

```bash

$ sudo systemctl start openresty
$ sudo systemctl status openresty
● openresty.service - The OpenResty Application Platform
   Loaded: loaded (/usr/lib/systemd/system/openresty.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-06-30 19:39:01 CST; 4min 6s ago
  Process: 21231 ExecStart=/usr/local/openresty/nginx/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 21229 ExecStartPre=/usr/local/openresty/nginx/sbin/nginx -t (code=exited, status=0/SUCCESS)
 Main PID: 21234 (nginx)
   CGroup: /system.slice/openresty.service
           ├─21234 nginx: master process /usr/local/openresty/nginx/sbin/ngin...
           └─21235 nginx: worker process

Jun 30 19:39:01 web1.chaos.luxe systemd[1]: Starting The OpenResty Applicati....
Jun 30 19:39:01 web1.chaos.luxe nginx[21229]: nginx: the configuration file ...k
Jun 30 19:39:01 web1.chaos.luxe nginx[21229]: nginx: configuration file /usr...l
Jun 30 19:39:01 web1.chaos.luxe systemd[1]: Started The OpenResty Applicatio....
Hint: Some lines were ellipsized, use -l to show in full.


```

## Further


### 使用  resty 执行

**在命令行中执行**

```bash
$ resty -e 'print("Hello OpenResty")'

```

**使用脚本执行**

```bash
$ cat hello.resty
#!/usr/bin/env resty
# use resty 
print("hello openresty")

$ ./hello.resty 
```

> 脚本文件支持传递命令行参数，参数存储在表 `arg` 中，使用  `arg[N]` 的方式可以访问参数：

```bash
$cat args.resty
#!/usr/bin/env resty
-- 得到参数的数量
local n = #arg
print("args count is ", n)

for i = 1, n do
    print("arg ", i, ":", arg[i])
end
$
$ ./args.resty a b c 1 3 5
args count is 6
arg 1:a
arg 2:b
arg 3:c
arg 4:1
arg 5:3
arg 6:5

```

**使用 restydoc**

OpenResty 附带了非常完善的用户参考手册  restydoc， 功能类似 man。例如：

```bash
$ restydoc nginx
$ restydoc luajit
$ restydoc opm
$ restydoc -s proxy_pass 
```

其中 `-s` 参数用来指定搜索手册里的小结名。


对于使用 opm 安装的组件，需要使用 `-r` 参数指定安装目录，例如：

```bash
$ restydoc -r /usr/local/openresty/site -s lua-resty-http
```



EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





