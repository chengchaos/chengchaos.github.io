---
title: iPhone 使用 MAC 上的 Socks 代理
key: 2020-06-12
tags: iPhone Mac socks
---



**Step 1.** Make sure the SOCKS tunnel on your work computer allows LAN connections so your iPhone/iPod Touch can connect to it.



```bash
$ ssh -N -g -D 1080 user@domain.com
```



**Step 2.** Create a text file and insert the following code:



```
function FindProxyForURL(url, host)
{ 
     return "SOCKS 192.168.xx.xx:yyyy";
}
```



*Replace the x's with your IP and the y's with the port you used after the -D in your SSH command*

**Step 3.** Save the text file as a Proxy Auto-Config (PAC) file to a web accessible place with a **.pac** extension.

*If you're reading this chances are you know how to serve a file over HTTP on your work LAN, so I won't delve into that.*



**Step 4.** Finally, on your iPhone/iPod Touch, go to **Settings &gt; Wifi** and click the blue arrow to the right of your work network. Scroll to  the bottom, click Auto and type in the address to your PAC file (e.g. http://192.168.xx.xx/mysupersecretproxy.pac).

<!--more-->

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>





