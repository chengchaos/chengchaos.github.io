---
title: Bask 如何判断一个变量是否为空
key: 2020-01-14
tags: Bash 
---





编写 Shell 脚本时，判断一个变量是否是已经赋值了的。



<!--more-->



## 判断方法



### 变量通过双引号扩起来



```bash
param=
if [ ! -n "$param" ]; then
    echo "is null"
else
    echo "Not is null"
fi
```





### 直接通过变量判断



```bash
param=
if [ ! $param ]; then
    echo "is null"
else
    echo "Not is null"
fi
```





### 使用 test 命令



```bash
param=
if test -z "$param"; then
    echo "is not set!"
else
    echo "is set"
fi
```



###  和空串比较



```bash
param=
if [ "$param" = "" ]; then
    echo "is not set!"
else
    echo "is set"
fi
```



就酱



<< EOF >>

If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
