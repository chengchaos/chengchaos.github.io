---
title: Ubuntu 切换 Xfce4
key: 2026-01-25
tags: Ubnut Xfce4
---

在 Ubuntu 22.04 中，可以通过以下步骤从 Gnome 桌面切换到 Xfce4 桌面 ...


<!--more-->


1. 打开终端(Ctrl + Alt + T)
1. 安装 Xfce4：输入命令 `sudo apt install xfce4` 并回车
1. 安装 LightDM(登录器)：输入命令 `sudo apt install lightdm` 并回车
1. 选择 LightDM 作为默认登录器：输入命令 `sudo dpkg-reconfig lightdm` 并回车
1. 重启电脑，在登录界面选择 Xfce4 桌面登录即可。

[原文链接]<https://blog.csdn.net/weixin_42581003/article/details/129602454>
















