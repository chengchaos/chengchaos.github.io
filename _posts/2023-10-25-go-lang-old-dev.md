---
title: Windows XP 开发 go 语言程序
tags: go go-lang
article_header:
  type: cover
  image:
    src: /screenshot.jpg
---

A Post with Header Image, See [Page layout](https://tianqi.name/jekyll-TeXt-theme/samples.html#page-layout) for more examples.

<!--more-->

Go 在 1.11 版本中增加了模块管理功能 mod。 

Go 1.10 之后就不再支持 Window XP 了。使用 Go 1.10 就需要继续使用 GOPATH 的方式进行包管理。

## 原生模块管理

Go 1.10 使用原生的模块管理方式即 GOPATH，这种模式下需要注意以下：

- 所有的第三方包都已源码方式放到 `GOPATH/src` 目录下面。
- 使用 `go get` 安装的第三方包就是下载到这个目录中。
- 我们在创建自己的项目的时候， 也需要将我们的项目放在这个目录下， 注意我们的项目中的各个包才能够正确的相互调用。

## 实战

创建环境变量 GOPATH

%USERPROFILE%\go

cd %USERPROFILE%
mkdir go

注意：项目目录一定要保存在 GOPATH 中的 src 目录下。


GoLand 配置：

- New Project .. Go (GOPATH)
- 手动配置 GOPATH
- 取消使用系统环境中的目录

## Vendor 模块管理

在 1.5 之后 Golang 支持使用 Vendor 作为模块管理， Vendor 优点：

- 项目里可以使用与 GOPATH 不同版本的第三方库。
- 可以系统的管理目前项目中用的的第三方库。

Vender 的管理方式与 npm 很相似，通过维护一个名为 `vendor.json` 的文件，文件内维护第三方库的信息。

Vender 会在项目中创建一个名为 `vendor` 的目录， 用于存放三方包， 也就是项目自己的 GOPATH。

由于 go 11.0 之后使用了 mod ， Vendor 已经停止维护， 最后一个版本是 1.9.0.

<https://github.com/kardianos/govendor/tree/v1.0.9>

```bash
#### 安装 Vendor
go get -u github.com/kardianos/govendor

govendor.exe -version

```

> 注意， Vendor 的使用同样需要我们将项目放置在	GOPATH/src/ 目录中。

```bash
govendor init
govendor fetch github.com/natefinch/lumberjack@v2.0.0
```

一个新的 Vendor 进行管理的项目, 可以使用下面的命令下载相关依赖包.

```bash
govendor sync
govendor fetch -insecure [ 你的项目 HTTP git 地址 ]
```

缺包

Vendor 默认情况下只会拉去目的地址的跟路径下的文件, 如果三方库含有复杂的包结构, 会出现问题, 因此我们需要拉去整个项目.

> 使用 `tree` 参数表示拉取整个项目.

```bash
govendor fetch -tree [ 你的项目 git 地址 ]
```
你的项目 HTTP git 地址]

