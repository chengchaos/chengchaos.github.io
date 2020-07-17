---
title: Netty 和 SSL
key: 2020-07-16
tags: netty openssl 
---

![openssl.png](http://qiniu.chaos.luxe/openssl.png)

参考： 
- https://blog.csdn.net/chuorena/article/details/77622235
- https://www.cnblogs.com/f-ck-need-u/p/6091027.html

## SSL常用认证方式介绍

- 单向认证
- 双向认证
- CA认证

<!--more-->

### SSL单向认证

单向认证只需客户端验证服务端，即客户端只需要认证服务端的合法性，服务端不需要。

这种认证方式适用 Web 应用，因为 Web 应用的用户数目广泛，且无需在通讯层对用户身份进行验证，一般都在应用逻辑层来保证用户的合法登入。但如果是企业应用对接，情况就不一样，可能会要求对客户端(相对而言)做身份验证。这时就需要做SSL双向认证。

### SSL双向认证

双向认证顾名思义，服务端也需要认证客户端的合法性，这就意味着**客户端的自签名证书需要导入服务端的数字证书仓库**。

采用这种方式会不太便利，一但客户端或者服务端修改了秘钥和证书，就需要重新进行证书交换，对于调试和维护工作量非常大，并且由于 agent 数目不确定，动态增加 agent 的时候需要平台和 agent 双方互相加入相应各自的证书。

### CA认证

CA 认证的好处是只要服务端和客户端只需要将 CA 证书导入各自的 keystore，客户端和服务端只需判断这些证书是 CA 签名过的即可，这也是蜂鸟平台内部采用的认证方式

##

### 根证书的生成

```bash

$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout root.key -out root.crt \
  -subj /C=CN/ST=Beijing/L=Beijing/O=Futuremove/OU=GA/CN=GA \
  -config /etc/pki/tls/openssl.cnf

```

### 服务端证书和密钥

```bash
$ keytool -genkey -alias server -keypass 123456 \
  -validity 1825 -keyalg RSA -keystore gateway.keystore \
  -keysize 2048 -storepass 123456 \
  -dname "CN=GA, OU=GA, O=Futuremove, L=Beijing, ST=Beijing, C=CN"

```

EOF
---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





