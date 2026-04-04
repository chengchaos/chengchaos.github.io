---
title: 在 CentOS 中编译安装 Python 3
key: 2023-08-21
tags: centos linux compile install python
---

> Search suggest: centos linux compile install python 编译 安装

现在看在linux 中编译 python 3 还是挺方便的， 这里是一个笔记。

<!--more-->

## 0x01 依赖

```bash
yum -y install gcc patch libffi-devel python-devel zlib-devel bzip2-devel openssl-devel ncurses-devel  sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

## 0x02 下载并编译


0, **下载和编译**

```bash
$ wget -c https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tar.xz
$ tar -xvf Python-3.11.4.tar.xz
$ cd Python-3.11.4

## 下面这个命令可以安装, 但是在 CentOS 7 上会有缺失 ssl 模块的问题.
$ ./configure --prefix=/opt/apps/python-3.11.4/
## 下面的这个命令, 编译不成功, 具体的原因不详, 有人说是要把过去的编译文件清掉
## 即现执行 make clean , 但是测试还是报错.
##另外 --with-ssl 命令无效了. 
./configure --enable-optimizations \
    --prefix=/opt/apps/python-3.11.4/ \
    --with-ssl
## 所以下面的命令是可以成功的. 但是要提前安装高版本的 openssl 库.
./configure --prefix=/opt/apps/python-3.11.4/

......

If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations

$ make clean
$ make
$ sudo make install
......

Processing /tmp/tmpevph_e2t/setuptools-65.5.0-py3-none-any.whl
Processing /tmp/tmpevph_e2t/pip-23.1.2-py3-none-any.whl
Installing collected packages: setuptools, pip
  WARNING: The scripts pip3 and pip3.11 are installed in '/opt/apps/python-3.11.3/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed pip-23.1.2 setuptools-65.5.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

```
## 0x03 openssl

今天在 CentOS 7 上使用 virtualenv 的时候，出现了错误：ModuleNotFoundError：No module name '_ssl',但是我的系统是安装了 openssl 的 1.0.1 的，查了网络上的信息发现，Python3.1 以后的版本，需要openssl1.0.2+，或者 Libressl2.6.4+, Python 3.10 以后不支持 OpenSSL 1.0.2 或者 LibreSSL.

下载 openssl-1.1.1+

下载地址

- 参考: <https://www.openssl.org/>
- <https://www.openssl.org/source/openssl-1.1.1w.tar.gz>


```bash
$ yum list | grep openssl 
openssl.x86_64                             1:1.0.2k-26.el7_9      @updates      
openssl-devel.x86_64                       1:1.0.2k-26.el7_9      @updates      
openssl-libs.x86_64                        1:1.0.2k-26.el7_9      @updates      
openssl11-libs.x86_64                      1:1.1.1k-5.el7         @epel         
anope-openssl.x86_64                       2.0.14-1.el7           epel          
apr-util-openssl.x86_64                    1.5.2-6.el7_9.1        updates       
globus-gsi-openssl-error.x86_64            4.4-1.el7              epel          
globus-gsi-openssl-error-devel.x86_64      4.4-1.el7              epel          
globus-gsi-openssl-error-doc.noarch        4.4-1.el7              epel          
globus-openssl-module.x86_64               5.2-1.el7              epel          
globus-openssl-module-devel.x86_64         5.2-1.el7              epel          
globus-openssl-module-doc.noarch           5.2-1.el7              epel          
libknet1-crypto-openssl-plugin.x86_64      1.20-1.el7             epel          
openssl-devel.i686                         1:1.0.2k-26.el7_9      updates       
openssl-libs.i686                          1:1.0.2k-26.el7_9      updates       
openssl-perl.x86_64                        1:1.0.2k-26.el7_9      updates       
openssl-pkcs11.x86_64                      0.4.10-1.el7           epel          
openssl-static.i686                        1:1.0.2k-26.el7_9      updates       
openssl-static.x86_64                      1:1.0.2k-26.el7_9      updates       
openssl098e.i686                           0.9.8e-29.el7.centos.3 base          
openssl098e.x86_64                         0.9.8e-29.el7.centos.3 base          
openssl11.x86_64                           1:1.1.1k-5.el7         epel          
openssl11-devel.x86_64                     1:1.1.1k-5.el7         epel          
openssl11-static.x86_64                    1:1.1.1k-5.el7         epel          
rh-mongodb32-golang-github-10gen-openssl-devel.noarch
rh-mongodb32-golang-github-10gen-openssl-unit-test.x86_64
rh-ruby24-rubygem-openssl.x86_64           2.0.9-92.el7           centos-sclo-rh
rh-ruby25-rubygem-openssl.x86_64           2.1.2-9.el7            centos-sclo-rh
rh-ruby26-rubygem-openssl.x86_64           2.1.2-121.el7          centos-sclo-rh
rh-ruby27-rubygem-openssl.x86_64           2.1.2-130.el7          centos-sclo-rh
rubygem-openssl_cms_2_0_0.x86_64           0.0.2-1.20140212git7fea071.el7
rubygem-openssl_cms_2_0_0-doc.noarch       0.0.2-1.20140212git7fea071.el7
xmlsec1-openssl.i686                       1.2.20-7.el7_4         base          
xmlsec1-openssl.x86_64                     1.2.20-7.el7_4         base          
xmlsec1-openssl-devel.i686                 1.2.20-7.el7_4         base          
xmlsec1-openssl-devel.x86_64               1.2.20-7.el7_4         base    

$ cat INSTALL
...
If you want to just get on with it, do:
...
  on Unix (again, this includes Mac OS/X):

    $ ./config
    $ make
    $ make test
    $ make install
# sudo -i
# cd /usr/local/lib64
# ln -s /usr/local/lib64/libssl.so.1.1 libssl.so
# exit
```

1， **设置环境变量**

略。

2, **安装 pipenv**

```bash
[chengchao@matrix002 python-3]$ pip3 install pipenv
  faulting to user installation because normal site-packages is not writeable
WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
......

[chengchao@matrix002 ~]$ mkdir -p ~/.pip
[chengchao@matrix002 ~]$ touch ~/.pip/pip.conf
[chengchao@matrix002 ~]$ vim ~/.pip/pip.conf

[global]
index-url = http://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host = mirrors.aliyun.com

[chengchao@matrix002 python-3]$ pip3 install pipenv
......
[notice] A new release of pip is available: 23.1.2 -> 23.2.1
[notice] To update, run: python3.11 -m pip install --upgrade pip
[chengchao@matrix002 ~]$ python3.11 -m pip install -upgrade pip

Usage:   
  /opt/apps/python-3/bin/python3.11 -m pip install [options] <requirement specifier> [package-index-options] ...
  /opt/apps/python-3/bin/python3.11 -m pip install [options] -r <requirements file> [package-index-options] ...
  /opt/apps/python-3/bin/python3.11 -m pip install [options] [-e] <vcs project url> ...
  /opt/apps/python-3/bin/python3.11 -m pip install [options] [-e] <local project path> ...
  /opt/apps/python-3/bin/python3.11 -m pip install [options] <archive url/path> ...

no such option: -u
[chengchao@matrix002 ~]$ python3.11 -m pip install --upgrade pip
Defaulting to user installation because normal site-packages is not writeable
Looking in indexes: http://mirrors.aliyun.com/pypi/simple/
Requirement already satisfied: pip in /opt/apps/python-3/lib/python3.11/site-packages (23.1.2)
Collecting pip
  Downloading http://mirrors.aliyun.com/pypi/packages/50/c2/e06851e8cc28dcad7c155f4753da8833ac06a5c704c109313b8d5a62968a/pip-23.2.1-py3-none-any.whl (2.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 574.8 kB/s eta 0:00:00
Installing collected packages: pip
Successfully installed pip-23.2.1                                                       
```

## appendix 参考和抄袭

- <https://blog.csdn.net/witton/article/details/109352577>
- <https://www.cnblogs.com/shangping/p/10756718.html>

EOF