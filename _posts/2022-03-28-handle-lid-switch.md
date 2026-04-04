---
title: 设置合上笔记本盖子不休眠
key: 2022-03-28
tags: linux switch
---

编辑文件：

```bash
sudo gedit /etc/systemd/logind.conf
```

<!--more-->

```bash
#HandlePowerKey 按下电源键后的行为，默认 power off
#HandleSleepKey 按下挂起键后的行为，默认 suspend
#HandleHibernateKey 按下休眠键后的行为，默认 hibernate
#HandleLidSwitch 合上笔记本盖后的行为，默认 suspend
# （改为ignore；即合盖不休眠）在原文件中，还要去掉前面的#
```

然后将其中的：
```bash
#HandleLidSwitch=suspend
```

改成下面，记得去 `#` 号：
```bash
HandleLidSwitch=ignore
```

最后重启服务

```bash
service systemd-logind restart
systemctl restart systemd-logind.service 
```

> 注：在Ubuntu18.04和Ubuntu16.04笔记本电脑，下面测试可以使用。

## 参考(照抄)

- [https://blog.csdn.net/xiaoxiao133/article/details/82847936](https://blog.csdn.net/xiaoxiao133/article/details/82847936)

---

If you like TeXt, don't forget to give me a star. :star2:

[![Star This Project](https://img.shields.io/github/stars/kitian616/jekyll-TeXt-theme.svg?label=Stars&style=social)](https://github.com/kitian616/jekyll-TeXt-theme/)
