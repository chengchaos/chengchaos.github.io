---
title: Sublime 使用笔记
key: 2022-11-18
tags: sublime
---

Omitted ...

<!--more-->

## 0x01 安装 package control

### 简单方法：

最简单的安装方法是通过 Sublime Text 控制台。 可通过 ctrl +\` 快捷方式或 View &gt; Show &gt; Console 菜单访问控制台。 打开后，将适用于您的 Sublime Text 版本的 Python 代码粘贴到控制台中。

Sublime Text 3粘贴如下代码

```python
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by) 
```

Sublime Text 2粘贴如下代码

```python
import urllib2,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation') 
```

运行报错需要运行代理全局模式，代理也不行的话可以用手动下载完整安装包的方法完成安装

### 手动方法：

点击 Preferences &gt; Browse Packages … menu 在 packages 文件夹的上层目录有一个 Installed Packages 

目录点击链接下载 [Package Control.sublime-package](https://link.zhihu.com/?target=https%3A//packagecontrol.io/Package%2520Control.sublime-package) 

复制到 Installed Packages 目录

重启软件 

## 0x01 安装 package

去这里 [Package Control](https://packagecontrol.io/) 搜索


参考：

- [Sublime Text 3 安装Package Control](https://zhuanlan.zhihu.com/p/70914472)

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
