---
title: 在 CentOS 中编译安装 Python 3
key: 2023-08-21
tags: linux python install
---

> Suggest search： linux python install ceotos 编译

linux 中编译 python 3 还是挺方便的， 这里是一个笔记。

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
$ ./configure --prefix=/opt/apps/python-3.11.3/
......

If you want a release build with all stable optimizations active (PGO, etc),
please run ./configure --enable-optimizations

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

## 0x03 参考和抄袭

- <https://blog.csdn.net/witton/article/details/109352577>
- <https://www.cnblogs.com/shangping/p/10756718.html>

EOF


