---
title: SSH 端口转发
key: 2020-08-25
tags: ssh ssh-tunnel
---

SSH端口转发也被称作 SSH 隧道([SSH Tunnel](http://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html))，因为它们都是通过SSH登陆之后，在**SSH客户端**与**SSH服务端**之间建立了一个隧道，从而进行通信。SSH 隧道是非常安全的，因为 SSH 是通过加密传输数据的(SSH 全称为 Secure Shell)。

<!--more-->

SSH 有三种端口转发模式:

- 本地端口转发(Local Port Forwarding)
- 远程端口转发(Remote Port Forwarding)
- 动态端口转发(Dynamic Port Forwarding)

对于本地/远程端口转发，两者的方向恰好相反。**动态端口转发**则可以用于科学上网。

在本文所有示例中，本地主机 A1 为 SSH 客户端，远程云主机 B1 为 SSH 服务端。从 A1 主机通过 SSH 登录 B1 主机，指定不同的端口转发选项(**-L、-R和-D**)，即可在 A1 与 B1 之间建立 SSH 隧道，从而进行不同的端口转发。

## 本地端口转发

### 应用场景:

远程云主机 B1 运行了一个服务，端口为 3000，本地主机 A1 需要访问这个服务。

示例为一个简单的 Node.js 服务:

```js
var http = require('http');

var server = http.createServer(function(request, response) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    response.end("Hello Fundebug\n");
});

server.listen(3000);
```

假设云主机 B1 的IP为 **103.59.22.17**，则该服务的访问地址为: http://103.59.22.17:3000 

### 为啥需要本地端口转发呢？ 

一般来讲，云主机的防火墙默认只打开了 22 端口，如果需要访问 3000 端口的话，需要修改防火墙。为了保证安全，防火墙需要配置允许访问的 IP 地址。但是，本地公网 IP 通常是网络提供商动态分配的，是不断变化的。这样的话，防火墙配置需要经常修改，就会很麻烦。



### 什么是本地端口转发？

所谓本地端口转发，就是将发送到本地端口的请求，转发到目标端口。这样，就可以通过访问本地端口，来访问目标端口的服务。使用 `-L` 属性，就可以指定需要转发的端口，语法是这样的:

```
-L [本地网卡地址]:<本地端口>:<目标地址>:<目标端口>
```

通过**本地端口转发**，可以将发送到本地主机 A1 端口 2000 的请求，转发到远程云主机 B1 的 3000 端口。

```bash
# 在本地主机A1登陆远程云主机B1，并进行本地端口转发
$ ssh -L localhost:2000:localhost:3000 -N  root@103.59.22.17
```


这样，在本地主机 A1 上可以通过访问 http://localhost:2000 来访问远程云主机 B1 上的Node.js服务。

选项：
- `-f` 后台启用，即认证之后 ssh 自动以后台运行，不再输出信息
- `-n` 将 stdio 重定向到 /dev/null 与 -f 配合使用
- `-N` 不打开远程shell，处于等待状态(不执行脚本或命令，即通知 sshd 不运行设定的 shell 通常与 -f 连用)
- `-g` 启用网关功能
- `-T` 不分配 TTY 只做代理用
- `-q` 安静模式，不输出 **错误/警告** 信息

实际上，`-L` 选项中的本地网卡地址是可以省略的，这时表示 2000 端口绑定了本地主机 A1 的所有网卡：

```bash
# 在本地主机A1登陆远程云主机 B1，并进行本地端口转发。2000端口绑定本地所有网卡
$ ssh -L 2000:localhost:3000 root@103.59.22.17
```

若有一台本地主机 A2 能够访问 A1，则 A2 也可以通过 A1 访问远程远程云主机 B1 上的 Node.js 服务。

另外，`-L` 选项中的目标地址也可以是其他主机的地址。假设远程云主机 B2 的局域网 IP 地址为 192.168.59.100，则可以这样进行端口转发:

```bash
# 在本地主机A1登陆远程云主机 B1，并进行本地端口转发。
# 请求被转发到远程云主机 B2 上
$ ssh -L 2000:192.168.59.100:3000 root@103.59.22.17
```

若将 Node.js 服务运行在远程云主机 B2 上，则发送到 A1 主机 2000 端口的请求，都会被转发到B2主机上。

## 远程端口转发

###  应用场景:

本地主机 A1 运行了一个服务，端口为 3000，远程云主机 B1 需要访问这个服务。

将前文的 Node.js 服务运行在本地，在本地就可以通过 http://localhost:3000 访问该服务。

### 为啥需要远程端口转发呢？

通常，本地主机是没有独立的公网 IP 的，它与同一网络中的主机共享一个 IP。没有公网 IP，云主机是无法访问本地主机上的服务的。

###  什么是远程端口转发？

所谓远程端口转发，就是**将发送到远程端口的请求，转发到目标端口**。这样，就可以通过访问远程端口，来访问目标端口的服务。使用 `-R` 属性，就可以指定需要转发的端口，语法是这样的:

```
-R [远程网卡地址]:<远程端口>:<目标地址>:<目标端口>
```

这时，通过**远程端口转发**，可以将发送到远程云主机 B1 端口 2000 的请求，转发到本地主机 A1 端口 3000。

```bash
# 在本地主机A1登陆远程云主机B1，并进行远程端口转发
$ ssh -R localhost:2000:localhost:3000 root@103.59.22.17
```

这样，在远程云主机A1可以通过访问 http://localhost:2000 来访问本地主机的服务。

```bash
# 在远程云主机B1访问本地主机A1上的Node.js服务
$ curl http://localhost:2000
Hello Fundebug
```
同理，**远程网卡地址**可以省略，**目标地址**也可以是其他主机地址。假设本地主机 A2 的局域网IP地址为192.168.0.100。

```bash
# 在本地主机A1登陆远程云主机B1，并进行远程端口转发
$ ssh -R 2000:192.168.0.100:3000 root@103.59.22.17
```

若将 Node.js 服务运行在本地主机 A2 上，则发送到远程云主机 A1 端口 2000 的请求，都会被转发到 A2 主机上。

## 动态端口转发

### 应用场景:

远程云主机 B1 运行了多个服务，分别使用了不同端口，本地主机 A1 需要访问这些服务。

### 为啥需要动态端口转发呢？

一方面，由于防火墙限制，本地主机 A1 并不能直接访问远程云主机 B1 上的服务，因此需要进行端口转发；另一方面，为每个端口分别创建本地端口转发非常麻烦。

### 什么是动态端口转发？

对于**本地端口转发**和**远程端口转发**，都存在两个一一对应的端口，分别位于SSH的客户端和服务端，而**动态端口转发**则只是绑定了一个**本地端口**，而**目标地址:目标端口**则是不固定的。**目标地址:目标端口**是由发起的请求决定的，比如，请求地址为 `192.168.1.100:3000`，则通过SSH转发的请求地址也是 `192.168.1.100:3000`。

```
-D 本地网卡地址:本地端口
```

这时，通过**动态端口转发**，可以将在本地主机A1发起的请求，转发到远程主机B1，而由B1去真正地发起请求。


```bash
# 在本地主机A1登陆远程云主机B1，并进行动态端口转发
$ ssh -D localhost:2000 root@103.59.22.17
```

而在本地发起的请求，需要由Socket代理([Socket Proxy](https://en.wikipedia.org/wiki/SOCKS))转发到SSH绑定的2000端口。以 Firefox 浏览器为例，配置 Socket代理需要找到**首选项**>**高级**>**网络**>**连接**->**设置**:

![image](https://upload-images.jianshu.io/upload_images/1881763-ad18bc3a236066ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样的话，Firefox 浏览器发起的请求都会转发到 2000 端口，然后通过 SSH 转发到真正地请求地址。若 Node.js 服务运行在远程云主机 B1 上，则在 Firefox 中访问[localhost:3000](localhost:3000) 即可以访问。如果主机 B1 能够访问外网的话，则可以科学上网……

## 链式端口转发

**本地端口转发**与**远程端口转发**结合起来使用，可以进行链式转发。假设 A 主机在公司，B 主机在家，C 主机为远程云主机。A 主机上运行了前文的 Node.js 服务，需要在B 主机上访问该服务。由于 A 和 B 不在同一个网络，且 A 主机没有独立公共 IP 地址，所以无法直接访问服务。

通过本地端口转发，将发送到 B 主机 3000 端口的请求，转发到远程云主机 C 的 2000 端口。

```bash
# 在B主机登陆远程云主机C，并进行本地端口转发
$ ssh -L localhost:3000:localhost:2000 root@103.59.22.17
```

通过远程端口转发，将发送到远程云主机 C 端口 2000 的请求，转发到 A 主机的 3000 端口。

```bash
# 在A主机登陆远程云主机C，并进行远程端口转发
$ ssh -R localhost:2000:localhost:3000 root@103.59.22.17
```

这样，在主机B可以通过访问[http://localhost:3000](http://localhost:3000)来访问主机A上的服务。


## [](https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/#参考链接 "参考链接")参考链接

*   [SSH PortForwarding](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding?action=fullsearch&value=linkto%3A%22SSH%2FOpenSSH%2FPortForwarding%22&context=180)
*   [SSH隧道的原理和实现](http://www.pchou.info/linux/2015/11/01/ssh-tunnel.html)

## SSH 跳过 HostKeyChecking，不用输入 yes

SSH 跳过输入ssh 跳过 RSA key fingerprint 输入 yes/no

在配置大量的节点之间需要 ssh 连通的时候，如果自动复制很多节点，都需要输入 yes，两两节点之间都要互通一次，这样会造成很大的麻烦

**解决1；** 修改配置文件/etc/ssh/ssh_config, 找到  

```
# StrictHostKeyChecking ask
```
修改为：

```
StrictHostKeyChecking  no
```


**解决2：**  添加参数 `–o`  【o=option】

```
$ ssh root@192.168.25.133 -o "StrictHostKeyChecking no"`
$ scp -o "StrictHostKeyChecking no" newfile.txt  root@192.168.25.133:/root
```



参考：

- https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/


EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





