---
title: Solr 创建 core
key: 2021-12-14
tags: solr
---



有两种方法可以创建 Solr 的 Core。

<!--more-->

## 第一种方法

使用命令行工具

```bash
cd /Users/chengchao/apps/solr
bin/solr create -c chaos
ls server/solr
README.txt chaos      chengchao  configsets filestore  new_core   solr.xml   userfiles  zoo.cfg
```

打开浏览器，访问 Solr 的地址： http://localhost:8983/solr/#/



## 第二种方法

1， 在 server/solr 下面创建一个新的文件夹， 假设名字叫 `server/solr/cheng`。

2， 复制 `server/solr/configsets/_default/conf` 目录到 `server/solr/cheng` 目录中

3， 登录到 Solr 的页面管理工具，执行：

- 点击左侧的 Core Admin 链接
- 点击上面的 Add Core 按钮
- 在弹出层中输入名称和目录（这里就是 cheng)
- 点击 Add Core 按钮



照猫画虎的猫：

- [https://www.cnblogs.com/116970u/p/10403713.htm](https://www.cnblogs.com/116970u/p/10403713.htm)







