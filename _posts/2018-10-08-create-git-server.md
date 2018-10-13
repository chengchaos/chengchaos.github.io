---
title: Create-Git-Server
key: 20181008
tags: Git
---



搭建Git服务器需要准备一台运行Linux的机器，强烈推荐用Ubuntu或Debian，这样，通过几条简单的apt命令就可以完成安装。

<!--more-->

https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000


第 1 步，安装 `git`：


```bash
$ sudo apt install git
```

第 2 步，创建一个 git 用户，用来运行 git 服务：


```bash
$ sudo adduser git
```

第 3 步，创建证书登录：


收集所有需要登录的用户的公钥，就是他们自己的 `id_rsa.pub` 文件，把所有公钥导入到 `/home/git/.ssh/authorized_keys` 文件里，一行一个。


第 4 步，初始化 Git 仓库：


先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：

```bash
$ sudo git init --bare simple.git
```

Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：

```bash
$ sudo chown -R git:git sample.git
```


第 5 步，禁用 shell 登录：


出于安全考虑，第二步创建的 git 用户不允许登录 shell，这可以通过编辑 `/etc/passwd` 文件完成。找到类似下面的一行：

```bash
git:x:1001:1001:,,,:/home/git:/bin/bash
```

修改为：

```bash
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

这样，git 用户可以正常通过 ssh 使用 git，但无法登录 shell，因为我们为 git 用户指定的 git-shell 每次一登录就自动退出。


第 6 步，克隆远程仓库：


现在，可以通过 `git clone` 命令克隆远程仓库了，在各自的电脑上运行：

```bash
$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
```

剩下的推送就简单了。

---

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
