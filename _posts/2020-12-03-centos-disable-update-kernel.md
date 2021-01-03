---
title: CentOS 7 进制更新内核
key: 2020-12-03
tags: centos7 kernel yum 
---

如下

<!--more-->
编辑配置文件：

```bash
vim /etc/yum.conf
```

2/ 添加如下配置：

```bash
exclude=kernel*
```

3/ 更新缓存：

```bash
yum clean all
yum update
```

转自：

- [CentOS 7 禁止更新内核](https://ilouis.cn/centos/disable_kernel_update.html)

EOF

---

Power by TeXt.
