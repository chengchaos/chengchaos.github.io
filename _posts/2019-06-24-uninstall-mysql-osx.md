---
title: Mac OS X下完全卸载MySQL
key: 2019-06-24
tags: MAC OSX MySQL
---

Mac OS X 下删除 MySQL 是一件非常麻烦的事情，很多时候都不能完全删除，最终导致 MySQL 在 Mac 下的使用非常麻烦。下面我将介绍 MySQL 如何完全卸载的方法。


MySQL的卸载一般使用终端的方式操作（安装包中有安装文件，但是没有卸载文件，只能通过终端命令的方式卸载）。



<!--more-->

##  命令：

```bash
$ sudo rm -rf /usr/local/mysql*
$ sudo rm -rf /Library/StartupItems/MySQLCOM
$ sudo rm -rf /Library/PreferencePanes/MySQL.prefPane
$ rm -rf ~/Library/PreferencePanes/My*
$ sudo rm -rf /Library/Receipts/mysql*
$ sudo rm -rf /Library/Receipts/MySQL*
$ sudo rm -rf /var/db/receipts/com.mysql.*
```

最后打开系统偏好设置，最下方MySQL图标消失。




Good luck!

Write by Jimmy.li



参考： [Mac OS X下完全卸载MySQL](https://blog.csdn.net/u012721519/article/details/55002626)





---------------------
作者：Jimmy.li 
来源：CSDN 
原文：https://blog.csdn.net/u012721519/article/details/55002626 
版权声明：本文为博主原创文章，转载请附上博文链接！

https://www.cnblogs.com/zhaof/p/8244542.html)





<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
