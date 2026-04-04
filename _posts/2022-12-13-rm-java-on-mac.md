---
title: Mac 彻底卸载 Java
key: 2022-12-13
tags: java MAC
---

> Suggest search： rm java on mac Java MAC

使用了 sdkman 以后就不想用原来手动安装的 java 了。

<!--more-->

## 0x01 执行

```bash
### 1、移除 JavaAppletPlugin.plugin 与 JavaControlPanel.prefpane
sudo rm -fr /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin 
sudo rm -fr /Library/PreferencesPanes/JavaControlPanel.prefpane

### 2、查找jdk
ls /Library/Java/JavaVirtualMachines/ 


### 3、删除jdk
sudo rm -rf /Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk

```

## 0x02 参考链接

- [Mac 卸载Java「建议收藏」](https://javaforall.cn/144486.html)

EOF


