---
title: 删除 MAC 启动台 lanuchpad 中的无效图标
key: 2019-06-26
tags: MAC
---

在 Mac 上曾经用过 Eclipse， 后来升级为新版本以后，就删除了老版本，但是在启动台（LanuchPad）中还保留有图标。

怎么办？

<!--more-->


在网上找到了一个解决办法：重置启动台

```bash
defaults write com.apple.dock ResetLaunchPad -bool true
killall Dock
```


参考原文： [删除mac启动台launchpad中的无效图标，重置启动台](https://www.pythontab.com/html/2018/ITzixun_0929/1358.html)

<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
