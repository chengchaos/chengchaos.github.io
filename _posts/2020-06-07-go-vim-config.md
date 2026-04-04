---
title: 使用 Vim 开发 Golang
key: 2020-06-07
tags: vim go 
---

记录如何使用 Vim 快速开发 Golang 的步骤

<!--more-->

## 安装 Vundle.vim 插件

### 安装

[https://github.com/VundleVim/Vundle.vim](https://github.com/VundleVim/Vundle.vim)

```bash
mkdir -p ~/.vim/bundle
cd ~/.vim/bundle/
git clone git@github.com:VundleVim/Vundle.vim.git
```

### 配置

编辑 `~/.vimrc` 文件（如果没有就创建）,在文件顶部添加有关 Vundle.vim 的配置：

```sh
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```

保存后关闭，然后在打开，在 vim 窗口中执行命令：

```sh
:PluginInstall
```

Vundle 开始安装插件，安装完成后现实 `Done!`

## 安装 vim-go 插件

[https://github.com/fatih/vim-go](https://github.com/fatih/vim-go)

编辑 `~/.vimrc` 文件，在 `bundle#being` 和 `bundle#end` 中间增加一行：

```sh
Plugin 'fatih/vim-go'
```

保存后关闭，然后在打开，在 vim 窗口中执行命令：

```sh
:PluginInstall
```

Vundle 开始安装插件，安装完成后现实 `Done!`

## 安装 go.tools Binaries

vim-go 安装说明中提到所有必要的 binary 需要先安装好，比如 gocode、godef、goimports等。

通过在 vim 环境中执行：

```sh
:GoInstallBinaries
```

这些 vim-go 依赖的二进制工具将会自动被下载，并被安装到 `$GOBIN` 下或 `$GOPATH/bin` 下。（这个工具需要依赖 git 或 hg，需要提前安装到你的 OS 中。）

编辑 go 文件，新起一行输入 `fmt.`，然后 `ctrl+x` , `ctrl+o`，Vim 会弹出补齐提示下拉框，不过并非实时跟随的那种补齐，这个补齐是由 gocode 提供的。

- 输入一行代码：time.Sleep(time.Second)，执行 `:GoImports` ，Vim会自动导入time包。
- 将光标移到 Sleep 函数上，执行 `:GoDef` 或命令模式下敲入gd，Vim会打开`$GOROOT/src/time/sleep.go` 中 的 Sleep 函数的定义。执行 `:b 1`返回到hellogolang.go。
- 执行 `:GoLint`，运行 golint 在当前 Go 源文件上。
- 执行 `:GoDoc`，打开当前光标对应符号的 Go 文档。
- 执行 `:GoVet`，在当前目录下运行 `go vet` 在当前Go源文件上。
- 执行 `:GoRun`，编译运行当前 main package。
- 执行 `:GoBuild`，编译当前包，这取决于你的源文件，`GoBuild` 不产生结果文件。
- 执行 `:GoInstall`，安装当前包。
- 执行 `:GoTest`，测试你当前路径下的 `*_test.go` 文件。
- 执行 `:GoCoverage`，创建一个测试覆盖结果文件，并打开浏览器展示当前包的情况。
- 执行 `:GoErrCheck`，检查当前包种可能的未捕获的 errors。
- 执行 `:GoFiles`，显示当前包对应的源文件列表。
- 执行 `:GoDeps`，显示当前包的依赖包列表。
- 执行 `:GoImplements`，显示当前类型实现的interface列表。
- 执行 `:GoRename [to]`，将当前光标下的符号替换为 `[to]`。

## 其他插件

### YCM

YCM 现在使用 Python3 了，。

[https://github.com/ycm-core/YouCompleteMe](https://github.com/ycm-core/YouCompleteMe)

### ultisnips

```sh
Plugin 'SirVer/ultisnips'
```

[https://github.com/SirVer/ultisnips](https://github.com/SirVer/ultisnips)

添加以下配置到 `~/.vimrc` 文件中：

```sh
" Trigger configuration. Do not use <tab> if you use https://github.com/Valloric/YouCompleteMe.
let g:UltiSnipsExpandTrigger="<tab>"
let g:UltiSnipsJumpForwardTrigger="<c-b>"
let g:UltiSnipsJumpBackwardTrigger="<c-z>"

```

### molokai theme

```bash
mkdir -p ~/.vim/colors
cd ~/.vim/colors
touch molokai
```

复制这个文件到上面创建的文集中：

[https://github.com/fatih/molokai/blob/master/colors/molokai.vim](https://github.com/fatih/molokai/blob/master/colors/molokai.vim)

在.vimrc添加一行：

```sh
colorscheme molokai
```

- 参考文档： [go-vim配置](https://www.cnblogs.com/chris-cp/p/5846640.html)

.

---

Power by TeXt.
