---
title: 在 Debian 12 中编译安装 Python 3
key: 2025-08-25
tags: debian linux compile install python
---

> Search suggest: debian linux compile install python 编译 安装

Debian 12 中的默认安装的 Python 版本是 3.11 . 我们的项目用的什么版本都有， 至少从 3.8 到 3.10 的都有， 为了便于调试，我这里记录怎样收到安装 Python 3.8 到 3.10 的版本

以前还记过一个在 centos 中安装的笔记，[在这里](2023-08-21-install-python3-in-centos.md) 不过， 现在 Centos 已经 game over 了。 

<!--more-->

## 0x01 安装依赖

```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev libreadline-dev libsqlite3-dev libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev tk-dev libffi-dev
```

## 0x02 下载

```bash
# 下载 Python 3.8 的源代码包：
# https://www.python.org/downloads/source/
# wget https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tgz
sudo wget -c https://www.python.org/ftp/python/3.8.20/Python-3.8.20.tgz
sudo wget -c https://www.python.org/ftp/python/3.9.23/Python-3.9.23.tgz
sudo wget -c https://www.python.org/ftp/python/3.10.18/Python-3.10.18.tgz
sudo wget -c https://www.python.org/ftp/python/3.11.12/Python-3.11.12.tgz
```

## 0x03 编译

```bash
# 解压源代码包：
sudo tar -zxvf Python-3.8.12.tgz

# 进入解压后的目录：
cd Python-3.8.12

# 配置 Python 的编译选项：
sudo ./configure --enable-optimizations

# 编译并安装 Python：

sudo make -j 4
sudo make altinstall

# 注意：使用 make altinstall 命令而不是 make install 命令，这样可以避免覆盖系统自带的 Python 版本。
# 安装完成后，可以使用以下命令检查 Python 版本：

python3.8 --version
```


如果输出类似于 Python 3.8.12 的版本信息，则说明 Python 3.8 已经成功安装在你的 Debian 系统中了。
