---
title: OpenSUSE Leap 15.2 更新源
key: 2020-08-06
tags: linux opensuse aliyun 
---

如题

<!--more-->

## Step 1

禁用所有的软件源

```bash
$ sudo zypper mr -da
```

## Step2 添加新源

```bash

sudo -i
zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/leap/15.2/repo/oss/ aliyun-openSUSE-Leap-15.2-oss
zypper addrepo -f http://mirrors.aliyun.com/opensuse/distribution/leap/15.2/repo/non-oss/ aliyun-openSUSE-Leap-15.2-non-oss
zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/leap/15.2/oss/ aliyun-openSUSE-Update-Leap-15.2-oss
zypper addrepo -f http://mirrors.aliyun.com/opensuse/update/leap/15.2/non-oss/ aliyun-openSUSE-Update-Leap-15.2-non-oss

```
## Step3 刷新

```bash
$ sudo zypper ref

```

参考：

- https://blog.csdn.net/frdevolcqzyxynjds/article/details/104850686




EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





