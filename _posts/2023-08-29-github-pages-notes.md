---
title: 使用 Jekyll 创建 GitHub Pages 站点笔记
key: 2023-08-29
tags: github pages Jekyll
---

> Search suggest: centos linux compile install python 编译 安装

可以使用 Jekyll 在新仓库或现有仓库中创建 GitHub Pages 站点。

<!--more-->

这里列出了一些有用的参考链接, 有空的时候 (是什么时候?) 在回来弄吧。

- [jekyll 安装](https://jekyllcn.com/docs/installation/)
- [使用 Jekyll 创建 GitHub Pages 站点](https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)
- [Quickstart for GitHub Pages](https://docs.github.com/en/pages/quickstart)

## 0x01 测试 Hexo

参考: [docs](https://hexo.io/zh-cn/docs/)

```bash
chengchao@web1:~$  node -v
v16.18.1
chengchao@web1:~$ npm -v
8.19.2
chengchao@web1:~$ git --version
git version 2.35.3

```

1, 使用 npm 安装 Hexo

```bash
sudo npm install -g hexo-cli
## 对于熟悉 npm 的进阶用户，可以仅局部安装 hexo 包。
npm install hexo
```

安装以后，可以使用以下两种方式执行 Hexo：

```bash
npx hexo <command>
### Linux 用户可以将 Hexo 所在的目录下的 node_modules 添加到环境变量之中即可直接使用 hexo <command>：
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
```

2, 建站

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

动手试一试?

```bash
cd /works
mkdir hexo
hexo init  hexo
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
INFO  Install dependencies
INFO  Start blogging with Hexo!
ls -l hexo/
total 124
-rw-r--r--   1 chengchao users     0 Aug 31 10:33 _config.landscape.yml
-rw-r--r--   1 chengchao users  2441 Aug 31 10:33 _config.yml
drwxr-xr-x 210 chengchao users 12288 Aug 31 10:33 node_modules
-rw-r--r--   1 chengchao users   615 Aug 31 10:33 package.json
-rw-r--r--   1 chengchao users 91348 Aug 31 10:33 package-lock.json
drwxr-xr-x   2 chengchao users  4096 Aug 31 10:33 scaffolds
drwxr-xr-x   3 chengchao users  4096 Aug 31 10:33 source
drwxr-xr-x   2 chengchao users  4096 Aug 31 10:33 themes
```

3, 配置

参考这里: [https://hexo.io/zh-cn/docs/configuration](https://hexo.io/zh-cn/docs/configuration)

4, 指令

参考这里: [https://hexo.io/zh-cn/docs/commands](https://hexo.io/zh-cn/docs/commands)

动手试一试? 

```bash
hexo new 'post title with whitespace'
INFO  Validating config
INFO  Created: /works/hexo/source/_posts/post-title-with-whitespace.md
hexo generate
```

5, 部署到 ngix

```bash

    location /blog/ {
        proxy_pass http://blog;
    }

    location /hexo/ {
        alias /works/hexo/public/;
    }

```

this access policy has taken effect, thank you very much.

EOF