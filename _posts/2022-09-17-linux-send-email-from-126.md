---
title: Linux 使用 126 邮箱发邮件
key: 2022-09-17
tags: Linux email mail.126.com
---

昨天看到一个Linux 使用 126 邮箱发邮件的文章.

<!--more-->

## 0x01 安装 mailx

因为我的服务器上跑的是 OpenSUSE , 所以我用  zypper 装.

```bash
chengchao@web1:~/bin> sudo zypper install mailx
[sudo] password for chengchao:
Retrieving repository 'openSUSE-15.3-Oss' metadata ..................................[done]
Building repository 'openSUSE-15.3-Oss' cache .......................................[done]
Retrieving repository 'openSUSE-15.3-Update-Sle' metadata ...........................[done]
Building repository 'openSUSE-15.3-Update-Sle' cache ................................[done]
Loading repository data...
Warning: Repository 'openSUSE-15.3-Update-Oss' appears to be outdated. Consider using a different mirror or server.
Reading installed packages...
Resolving package dependencies...

The following NEW package is going to be installed:
  mailx

1 new package to install.
Overall download size: 320.4 KiB. Already cached: 0 B. After the operation, additional
548.3 KiB will be used.
Continue? [y/n/v/...? shows all options] (y): y
Retrieving package mailx-12.5-3.3.1.x86_64            (1/1), 320.4 KiB (548.3 KiB unpacked)
Retrieving: mailx-12.5-3.3.1.x86_64.rpm .............................................[done]

Checking for file conflicts: ........................................................[done]
(1/1) Installing: mailx-12.5-3.3.1.x86_64 ...........................................[done]
```

## 0x02 开通 126 邮箱的 smtp 和授权码

登录 126 邮箱, 在"设置"中开启 POP3/SMTP 服务.

## 0x03 配置 mailx

编辑 /etc/mail.rc 文件

```bash

chengchao@web1:~> cat /etc/mail.rc
set asksub append dot save crt=20
ignore Received Message-Id Resent-Message-Id Status Mail-From Return-Path Via

set from=<name>@126.com ## 发件人信息
set smtp=smtp.126.com:994 ## smtp 信息
set smtp-auth-user=<name>@126.com ## 登录账号
set smtp-auth-password=**** ## 授权码, 不是账号的密码!
set smtp-auth=login  ## 邮件认证方式
set ssl-verify=ignore
set nss-config-dir=/home/chengchao/.certs
```

网上的帖子到这里就配置结束了, 但是我测试了发不出去, 提示:

```bash
chengchao@web1:~> could not connect: Connection timed out
"/home/chengchao/dead.letter" 40/2057
. . . message not sent.
```

在另一篇[帖子](https://cloud.tencent.com/developer/article/1416520)中提到 :

> **到目前为止，如果不是云主机的话，已经可以实现发送邮件了。若是云主机，则需要下面的操作**

## 0x04 增加 SSL

上面配置的是简单的使用25端口的SMTP发送邮件的功能，一般情况下我们使用这个就足够了，这个办法在网上也很多配置说明，这里就不再浪费时间了，下面我们讲重点，使用TSL发送邮件；

前面说了，阿里云把25端口封了，去申请解封也比较麻烦，于是就想到了用TSL方式，绕过25端口发送邮件；

TSL也就是使用SSL加密的方式,使用465或者其他端口来发送邮件，现在大部分邮箱都支持SSL，具体SSL的端口地址，也可以查百度，这里是以126邮箱为准,126邮箱使用的是465或者994端口；

下面是详细的配置过程：

1. 软件要求：openssl、mailx 12.0以上；
2. 既然使用的是SSL协议，那当然是要有证书的了，下面是获取证书的操作；

```bash
mkdir -p ~/.certs/
echo -n | openssl s_client -connect smtp.126.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/qq.crt
certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -L -d ~/.certs
```

其实我只执行了第一行的命令就好使了

```bash
echo -n | openssl s_client -connect smtp.126.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/qq.crt
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root CA
verify return:1
depth=1 C = US, O = DigiCert Inc, CN = GeoTrust RSA CN CA G2
verify return:1
depth=0 C = CN, ST = zhejiang, L = hangzhou, O = "NetEase (Hangzhou) Network Co., Ltd", CN = *.126.com
verify return:1
DONE
chengchao@web1:~> cat .certs/qq.crt
-----BEGIN CERTIFICATE-----
略...
-----END CERTIFICATE-----

```

3. 证书配置好了，下面我们就要来配置mail.rc配置文件了，和最开始的不同，这里我们就需要配置和TSL相关的东西了

```bash
set smtp=smtps://smtp.126.com:465
set ssl-verify=ignore
set nss-config-dir=/root/.certs
```

其实就多了几个配置 stmp 前面加了 stmps://指定协议类型，后面加上端口号；

启动 SSL 协议 ，下面指定 SSL 证书所在目录，就这样。

## 0x05 发一个邮件

```bash
echo "hello: " | mail -v -s test c.b.cheng@myemailaddress.domain
```

发邮件的三种方式:

1.命令行:

```bash
mailx -s '主题' 邮件地址
```

回车后输入内容, 最后按 `Ctrl + D` 发送.

2.管道符:

```bash
echo "邮件正文" | mail -s "主题" 邮件地址
```

3.文件:

```bash
mail -s "主题" 邮件地址 < /tmp/content.txt
```

## 0x06 参考链接

- <https://www.jianshu.com/p/83f806914eff>
- <https://blog.csdn.net/cunjiu9486/article/details/109074085>
- <https://cloud.tencent.com/developer/article/1416520>

EOF
