---
title: Docker 容器内访问宿主记时报"No route to host"
key: 20181017
tags: Docker
---



Docker 容器内访问宿主记时报"No route to host"

<!--more-->

修复方式请按顺序运行以下命令：

```bash
# nmcli connection modify docker0 connection.zone trusted
# systemctl stop NetworkManager.service
# firewall-cmd --permanent --zone=trusted --change-interface=docker0
# systemctl start NetworkManager.service
# nmcli connection modify docker0 connection.zone trusted
# systemctl restart docker.service
```


---

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
