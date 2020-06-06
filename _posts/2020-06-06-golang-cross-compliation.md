---
title: Golang Corss Compliation
key: 2020-06-06
tags: go golang
---



![golang-gopher.png](https://upload-images.jianshu.io/upload_images/1881763-8909e92cdc183a2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Golang 支持交叉编译，在一个平台上生成另一个平台的可执行程序。



<!--more-->






### Golang支持的平台和版本

```bash
$ go tool dist list
```

其实 Golang 的交叉编译非常简单，只需要在编译前指定系统和 CPU 架构，基本不会有任何问题，编译出来讲文件拷贝到对应平台就能跑。


### Mac 下编译 Linux 和 Windows 64位可执行程序

```bash
$ CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```


### Linux 下编译 Mac 和 Windows 64位可执行程序

```bash
$ CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
$ CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
```

### Windows 下编译 Mac 和 Linux 64位可执行程序

```bat
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go

SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

- GOOS：指示目标平台的操作系统（darwin、freebsd、linux、windows
- GOARCH：指示目标平台的体系架构（386、amd64、arm）



Golang 语言的交叉编译的坑，主要就坑在 CGO 上！**所以一般都禁用它**，想用就参考这个页面：

https://blog.csdn.net/Three_dog/article/details/94640507



本文主要参考： 

https://blog.csdn.net/panshiqu/article/details/53788067



---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)
