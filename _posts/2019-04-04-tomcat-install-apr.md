---
title: Tomcat 8.5 安装 ARP
key: 20190404
tags: tomcat apr
---

参考： https://tomcat.apache.org/tomcat-8.5-doc/apr.html



<!--more-->

```bash
$ sudo yum -y install openssl-devel apr apr-devel apr-util apr-util-devel apr-util-openssl
$ cd bin
$ tar -xzvf tomcat-native.tar.gz 
$ cd tomcat-native
$ cd tomcat-native-1.2.21-src/native/
$ sudo ./configure --with-java-home=/usr/java/default --with-apr=/usr/bin/apr-1-config
$ sudo make 
$ sudo make install


----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/apr/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
```


<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
