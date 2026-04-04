---
title: 删除 IIS 中 Header 里面的 Server Header
key: 2020-09-10
tags: IIS Header Server
---

如果不考虑使用第三方控件实现，可以用PowerShell的方式：


<!--more-->

```
Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter "system.webServer/security/requestFiltering" -name "removeServerHeader" -value "True"
```

大概是修改IIS的配置：

C:\Windows\System32\inetsrv\config\applicationHost.config

```xml
<system.webServer>
　　<security>
　　　　 <requestFiltering removeServerHeader="true">
　　　　　　//...
           <requestFiltering>
      <security>
<system.webServer>
```


参考：

- [https://www.cnblogs.com/wybin6412/p/12924938.html](https://www.cnblogs.com/wybin6412/p/12924938.html)


EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





