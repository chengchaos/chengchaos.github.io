---
title: openssl 、nginx生成配置自签名证书
key: 2020-07-16
tags: openssl ssl nginx 
---

![openssl.png](http://qiniu.chaos.luxe/openssl.png)

参考：

- <https://www.cnblogs.com/ishen/p/12216681.html>

在 nginx 使用 https 协议需要配置证书，通过 CA 机构获取的证书是收费的，出于研究测试的话可以通过 openssl 自己制作证书，使用 openssl 制作证书如下：

- (1)生成CA根证书
- (2)生成服务器证书请求
- (3)通过CA根证书和服务器证书请求生成服务器证书

服务器证书生成后，便可以在nginx进行配置

<!--more-->

## openssl

openssl是一个用于生成密钥、公钥，证书，以及进行证书签名的工具。

### 配置

使用 openssl 之前，需要对其进行配置，设置好证书的存放目录，序列 ID 的存放位置。基本默认的是遏制都已经做好了，只需要更改一下 dir 的值即可。

CentOS 7 中有个 openssl 的示例文件： /etc/pki/tls/openssl.cnf

```bash
sudo cp /etc/pki/tls/openssl.cnf /etc/ssl/openssl.cnf
```

然后 openssl.cnf 中 `[ CA_default ]` 描述的文件结构，创建对应的文件目录和文件

```bash
$ sudo -i
# mkdir -pv /etc/pki/CA/{certs,crl,newcerts,private}
# touch /etc/pki/CA/{serial,index.txt}

```

文件说明：

```bash
# tree /etc/pki
/etc/pki
├── CA
│   ├── certs          (已颁发的证书保存目录)
│   ├── crl              (证书撤销列表存放目录)
│   ├── index.txt     (数据库索引文件，记录着一些已签署的证书信息)
│   ├── newcerts    (新签署证书的保存目录)
│   ├── private        (存放CA私钥的目录)
│   └── serial        （当前证书序列号）
```

### 指定证书编号

根证书是用来生成服务器证书的，证书之间是存在链式关系，当信任根证书时，由其衍生出来的证书都会被信任。

从根证书开始每一个证书都有一个对应的编号，是通过serial的值来进行维护的，首先指明证书的开始编号

```bash
# echo 01 >> serial
```

### 生成 CA 私钥

使用 `umask 077` 使得之后生成文件的默认权限为 077，使用 openssl 工具生成 4096 位的 rsa 秘钥，该秘钥存放在 `/etc/pki/CA/private/cakey.pem`。

```bash
# umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096
```

### 生成 CA 证书

使用刚生成的私钥生成CA证书，CA 证书保存在 /`etc/pki/CA/cacert.pem`。

- `req` : 这是一个大命令，提供生成证书请求文件，验证证书，和创建根CA
- `-new` : 表示新生成一个证书请求
- `-x509` : 直接输出证书
- `-key` : 生成证书请求时用到的私钥文件
- `-out`：输出文件

```bash
# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem \
  -subj /C=CN/ST=Beijing/L=Beijing/O=Futuremove/OU=GA/CN=GA \
  -out /etc/pki/CA/cacert.pem -days 3650
```

这个生成CA证书的命令会让人迷惑，因为生成证书其实一般需要经过三个步骤

1. 生成秘钥 `xxx.pem`
2. 通过秘钥 `xxx.pem` 生成证书请求文件 `xxx.csr`
3. 通过证书请求文件 `xxx.csr` 生成最终的证书 `xxx.crt`

但是以下的命令将2和3杂糅在了一起

```bash
# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem \
  -subj /C=CN/ST=Beijing/L=Beijing/O=Futuremove/OU=GA/CN=GA \
  -out /etc/pki/CA/cacert.pem -days 3650
```

其等价于：

```bash
### 生成证书请求文件
# openssl req -new -key /etc/pki/CA/private/cakey.pem \
  -subj /C=CN/ST=Beijing/L=Beijing/O=Futuremove/OU=GA/CN=GA \
  -out /etc/pki/CA/req1.csr
### -in 使用证书请求文件生成证书，-signkey 指定私钥，这是一个还没搞懂的参数
# openssl x509 -req -in /etc/pki/CA/req1.csr \
  -signkey /etc/pki/CA/private/cakey.pem \
  -out /etc/pki/CA/cacert1.pem -days 3650

```

## 生成服务器端证书

### 生成服务器端私钥 **(*.pem)**

首先我创建并进入了 ~/https 目录，然后生成服务端的**私钥**

```bash
$ openssl genrsa -out https.pem 4096
Generating RSA private key, 4096 bit long modulus
..........................................................................++
...........................................................................................................++

```

### 生成服务器端证书请求 **(*.csr)**

```bash
$ openssl req -new -key https.pem \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=Futuremove/CN=web1.chaos.luxe/emailAddress=chengchaos@outlook.com" \
  -days 365 -out https.csr 

```

这里使用 `-subj` 可以预先完成证书请求者信息的填写，但要注意，填入的信息中 C 和 ST 和 L 和 Ｏ 一定要与签署的根证书一样，如果忘记了根证书乱填了什么，可以通过如下指令进行查询:

```bash

$ openssl x509 -in 根证书的路径+名字 -noout -subject
$ openssl x509 -in /etc/pki/CA/cacert.pem -noout -subject
subject= /C=CN/ST=Beijing/L=Beijing/O=Futuremove/OU=GA/CN=GA
```

### 生成服务器端证书 **(*.cert)**

执行以下命令之后就能得到证书文件https.cert，这就是用于发送给客户端的证书。

```bash
$ openssl ca -in https.csr -out https.crt -days 365
$ sudo openssl ca -in https.csr -out https.crt -days 365
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Jul 16 12:38:09 2020 GMT
            Not After : Jul 16 12:38:09 2021 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = Beijing
            organizationName          = Futuremove
            commonName                = web1.chaos.luxe
            emailAddress              = chengchaos@outlook.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                6B:8E:28:40:53:3E:16:70:3D:5E:B0:47:56:76:57:1C:20:6A:7F:4B
            X509v3 Authority Key Identifier:
                keyid:D8:68:74:A0:10:BC:23:E3:09:14:B7:5B:74:3E:20:7E:5B:73:1C:92

Certificate is to be certified until Jul 16 12:38:09 2021 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

```

## 安装到 nginx 上

```bash

server {
    listen 443 ssl;
 server_name web1.chaos.luxe;
 
 ssl_certificate cert/https.crt;
 ssl_certificate_key cert/https.pem;
 
 ssl_session_cache shared:SSL:1m;
 ssl_session_timeout 5m;
 
 ssl_ciphers HIGH:!aNULL:!MD5;
 ssl_prefer_server_ciphers on;
 
 location / {
     root /works/htdocs;
  index index.html index.htm;
 }
}
```

## 其他

除了服务端证书和CA根证书之外，还有一种类型叫做客户端证书，这个的作用是用来验证客户的身份，在极少数的情况下会用到，比如说网银限制客户在某台机器上进行登陆，很久之前银行提供的u盾的作用就是为了提供客户端证书而存在的。总之，证书的作用就是用来验证身份。

## 原文参考文档

使用 openssl 生成证书（含openssl详解）：<https://blog.csdn.net/gengxiaoming7/article/details/78505107>
理解服务器证书CA&& SSL: <https://blog.csdn.net/weixin_41830501/article/details/81128968>
使用openssl生成证书(详细): <https://blog.csdn.net/gengxiaoming7/article/details/78505107>
证书的签发和通信过程: <https://www.cnblogs.com/handsomeBoys/p/6556336.html>
自签名根证书和客户端证书的制作: <https://blog.csdn.net/ilytl/article/details/52450334>
openssl指令说明： <https://www.cnblogs.com/gordon0918/p/5409286.html>

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
