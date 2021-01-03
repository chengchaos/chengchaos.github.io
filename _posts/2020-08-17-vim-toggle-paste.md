---
title: Vim 的 Paste 模式
key: 2020-08-17
tags: vim paste
---

vim 开启了自动缩进以后，在粘贴文字，会一直缩进，导致格式很乱。

<!--more-->

可以在命令模式输入

```sh
:set paste
```

这时候在进行粘贴，就没有问题了。

进入 paste 模式执行 `:set paste` 他做了以下事情：

```sh
textwidth 设置为 0
wrapmargin 设置为 0
set noai set nosi softtabstop 设置为 0
revins 重置
ruler 重置
showmatch 重置
formatoptions 使用空值
lisp 选项值不变，但被禁用
indentexpr 选项值不变，但被禁用
cindent 选项值不变，但被禁用
```

在命令模式下输入以下命令退出 paste 模式：

```sh
:set nopaste
```

另外，有一个切换 paste 的开关选项叫 `pastetoggle`。

可以通过它来绑定一个快捷键，实现一键激活/退出 paste 模式。

```sh
:set pastetoggle=<F5>
```

参考：

- [https://blog.csdn.net/weixin_44648216/article/details/103788877](https://blog.csdn.net/weixin_44648216/article/details/103788877)
- [https://vim.fandom.com/wiki/Toggle_auto-indenting_for_code_paste](https://vim.fandom.com/wiki/Toggle_auto-indenting_for_code_paste)

-- EOF --

Power by TeXt.
