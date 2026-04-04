---
title: git 修改远程的 URL
key: 2020-11-06
tags: git remote set-url
---

git remote set-url 指令需要传递两个参数：

- remote name。例如，origin 或者 upstream
- new remote url。例如，git@github.com:USERNAME/OTHERREPOSITORY.git

<!--more-->

例如：从 SSH 切换到 HTTPS 的远程 URL

1. 打开终端

2. 切换到你项目的工作目录

3. 列出 remotes，是为了得到你想要改变的 remote 的名字

```bash
xxxxxx@xxxxxx:~/workspace/goal$ git remote -v
origin    git@github.com:xxxxxx/SpringBoot.git (fetch)
origin    git@github.com:xxxxxx/SpringBoot.git (push)
```

1. 设置新 URL

使用 `git remote set-url` 命令从 SSH 到 HTTPS 的远程 URL

```bash
xxxxxx@xxxxxx:~/workspace/goal$ git remote set-url origin https://github.com/xxxxxx/SpringBoot.git
```

1. 验证是否改变成功

```bash
xxxxxx@xxxxxx:~/workspace/goal$ git remote -v
origin    https://github.com:xxxxxx/SpringBoot.git (fetch)
origin    https://github.com:xxxxxx/SpringBoot.git (push)
```

参考：

- [git-修改远程的URL](https://www.cnblogs.com/yandufeng/p/6423821.html)

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
