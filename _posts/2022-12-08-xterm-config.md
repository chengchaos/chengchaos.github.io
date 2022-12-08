---
title: Xtrem 配置
key: 2022-12-08
tags: xtrem xrdb Xresources
---

> 建议关键字 Xtrem xrdb Xresources

Omitted ...

<!--more-->

## 0x01 示例

英文叹号开头的是注释

```sh
!Xft 设置
Xft.dpi: 96
Xft.antialias: true
Xft.rgba: rgb
Xft.hinting: true
Xft.hintstyle: hintslight

!URxvt.preeditType:Root
xterm*termName:xterm-256color
! 中文字体
!xterm*faceName: Liberation Mono:size=11:antialias=false
!xterm*font: 7x13
! monofur 字体
!xterm*faceName: monofur:antialias=true:pixelsize=15
! Inconsolata 字体
!xterm*faceName: Inconsolata:antialias=true:pixelsize=14
xterm*faceName: Cascadia Code:antialias=true:pixelsize=14
!xterm*faceName: DejaVu Sans Mono:style=Book:antialias=false:pixelSize=9
!xterm*faceName: UbuntuMono-R:antialias=true:pixelSize=8
!xterm*faceNameDoublesize: 方正兰亭黑_GBK:antialias=true:pixelsize=12
xterm*faceNameDoublesize: Microsoft YaHei Mono:antialias=true:pixelsize=13
!xterm*faceSize: 8
!xterm*locale:zh_CN.UTF-8
xterm*locale:true
xterm*cjkWidt:true
!xterm*cjkWidt:false
!xterm*inputMethod:ibus

! 前景色和背景色
xterm*foreground:rgb:b2/b2/b2
xterm*background:rgb:08/08/08
!xterm*foreground:rgb:0a/0a/0a
!xterm*background:rgb:08/08/08

! 行间距
xterm*scaleHeight: 1.01

! 初始窗口大小
!xterm*geometry:120x60
xterm*VT100.geometry: 100x32+48+50
! fix alt key input
xterm*eightBitInput: false
xterm*altSendsEscape: true
xterm*scrollBar: true
! 动态颜色
xterm*dynamicColors:true
!-- Tango color scheme
*xterm*color0: #2e3436
*xterm*color1: #cc0000
*xterm*color2: #4e9a06
*xterm*color3: #c4a000
*xterm*color4: #3465a4
*xterm*color5: #75507b
*xterm*color6: #0b939b
*xterm*color7: #d3d7cf
*xterm*color8: #555753
*xterm*color9: #ef2929
*xterm*color10: #8ae234
*xterm*color11: #fce94f
*xterm*color12: #729fcf
*xterm*color13: #ad7fa8
*xterm*color14: #00f5e9
*xterm*color15: #eeeeec
! Scrolling
Xterm*saveLines: 4096
xterm*VT100.Translations: #override\n\
    Ctrl Shift <Key>V: insert-selection(CLIPBOARD)\n\
    Ctrl Shift <Key>C: copy-selection(CLIPBOARD)\n\
    Shift <Key>Up: scroll-back(1,line)\n\
    Shift <Key>Down: scroll-forw(1,line)\n\
    Ctrl <Key>Insert: copy-selection(CLIPBOARD)\n\
    Shift <Key>Insert: insert-selection(CLIPBOARD)\n
!    Ctrl <Key>Insert: copy-selection(SECONDARY)\n\
!    Shift <Key>Insert: insert-selection(SECONDARY)\n
!!XTerm*VT100.translations: #override\n\
    Shift <Key>Up: scroll-back(1,line) \n\
    Shift <Key>Down: scroll-forw(1,line)
    

```

保存为 `~/.Xresources` 文件

执行 `$ xrdb ~/.Xresources` 使配置生效

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>


