---
title: 国内源
key: 2020-12-03
tags: maven python ubuntu centos
---

如下

<!--more-->

## Python Pip3



- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 阿里云：http://mirrors.aliyun.com/pypi/simple/
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 华中理工大学：http://pypi.hustunique.com/
- 山东理工大学：http://pypi.sdutlinux.org/
- 豆瓣：http://pypi.douban.com/simple/



> 新版 ubuntu 要求使用 https 源，要注意。

### 临时使用

可以在使用pip的时候加参数`-i https://pypi.tuna.tsinghua.edu.cn/simple`
例如：

```bash
pip install -i http://mirrors.aliyun.com/pypi/simple/ tensorflow
```

这样就会从阿里云这边的镜像去安装tensorflow库。

### 永久使用

在Linux下, 修改 ` ~/.pip/pip.conf` (没有就创建一个文件夹及文件。文件夹要加“.”，表示是隐藏文件夹)

内容如下：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```



windows下，直接在user目录中创建一个pip目录，再新建文件 pip.ini。（例如：C:\Users\WQP\pip\pip.ini）内容同上。





转自：

- [python pip 使用国内源](https://www.jianshu.com/p/dfbb90995a2c)

EOF

---

Power by TeXt.
