---
title: openssl 、nginx生成配置自签名证书
key: 2020-07-16
tags: openssl ssl nginx 
---

以前摘抄过一个 [openssl 、nginx生成配置自签名证书](2020-07-16-nginx-with-ssl.md) 可以对照着看.

<!--more-->

## 具体操作

```bash
yum -y install mod_ssl openssl httpd
mkdir /etc/httpd/ca
cat /etc/httpd/conf.d/ssl.conf
### 创建自己的CA证书
openssl req -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 1000 -out ca.crt

### 生成CA证书签名请求
openssl req -newkey rsa:4096 -nodes -sha256 -keyout apache.xyc.com.key -out apache.xyc.com.csr

### 生成注册主机的证书
openssl x509 -req -days 365 -in apache.xyc.com.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apache.xyc.com.crt

### 使用 apache.xyc.com.crt 和 apache.xyc.com.key 这两个文件.
```

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
