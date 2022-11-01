
---

title: FreeBSD 更换 PKG 源

key: 2021-10-07

tags: FreeBSD pkg

---

FreeBSD 有四类源，pkg、ports、portsnap、update。http://freebsd.cn 暂不可用。
对于失去安全支持的版本，如 FreeBSD 9.0 是没有 pkg 源可用的，只能使用当时的 ports 编译安装软件。
本文对于一个源列出了多个镜像站，无需全部配置，只需选择其一即可。
目前境内没有官方镜像站，以下均为非官方镜像站。

<!--more-->

## 0x01 PKG 源

pkg 源提供二进制安装包.
pkg 的下载路径是 /var/cache/pkg/
FreeBSD 中 pkg 源分为系统级和用户级两个源.不建议直接修改 /etc/pkg/FreeBSD.conf,因为该文件会随着基本系统的更新而发生改变.

创建用户级源目录:

```bash
# mkdir -p /usr/local/etc/pkg/repos
```

### 网易开源镜像站

创建用户级源文件:

```bash
# ee /usr/local/etc/pkg/repos/163.conf
```

写入以下内容:

```js
163: {
    url: "pkg+http://mirrors.163.com/freebsd-pkg/${ABI}/quarterly",
    mirror_type: "srv",
    signature_type: "none",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
}
FreeBSD: { enabled: no }
```

### 中国科学技术大学开源软件镜像站

创建用户级源文件:

```bash
# ee /usr/local/etc/pkg/repos/ustc.conf
```

写入以下内容:

```js
ustc: {
    url: "pkg+http://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/quarterly",
    mirror_type: "srv",
    signature_type: "none",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
}

FreeBSD: { enabled: no }
```

### 南京大学开源镜像站

```bash
# ee /usr/local/etc/pkg/repos/nju.conf
```

写入以下内容:

```js
nju: {  
    url: "pkg+http://mirrors.nju.edu.cn/freebsd-pkg/${ABI}/quarterly",  
    mirror_type: "srv",  
    signature_type: "none",  
    fingerprints: "/usr/share/keys/pkg",  
    enabled: yes
}

FreeBSD: { enabled: no }
```


### http://FreeBSD.cn

```bash
# ee /usr/local/etc/pkg/repos/freebsdcn.conf
```

写入以下内容:

```js
freebsdcn: {  
    url: "pkg+http://pkg.freebsd.cn/${ABI}/quarterly",  
    mirror_type: "srv",  
    signature_type: "none",  
    fingerprints: "/usr/share/keys/pkg",  
    enabled: yes
}

FreeBSD: { enabled: no }
```

## ports 源

提供源码方式安装软件的包管理器

ports 下载路径是 `/usr/ports/distfiles`

### 北京交通大学自由与开源软件镜像站

创建或修改文件 `# ee /etc/make.conf`

写入以下内容:

```sh
MASTER_SITE_OVERRIDE?=http://mirror.bjtu.edu.cn/reverse/freebsd-pkg/ports-distfiles/
```

### 网易开源镜像站

创建或修改文 `件# ee /etc/make.conf`

写入以下内容:

```sh
MASTER_SITE_OVERRIDE?=http://mirrors.163.com/freebsd-ports/distfiles/
```

### 中国科学技术大学开源软件镜像站

创建或修改文件 `# ee /etc/make.conf`

写入以下内容:

```sh
MASTER_SITE_OVERRIDE?=http://mirrors.ustc.edu.cn/freebsd-ports/distfiles/
```

### http://FreeBSD.cn

创建或修改文件 `# ee /etc/make.conf`

写入以下内容:

```sh
MASTER_SITE_OVERRIDE?=http://ports.freebsd.cn/ports-distfiles/
```

## portsnap 源

打包的 ports文件

### 北京交通大学自由与开源软件镜像站

编辑portsnap配置文件 `# ee /etc/portsnap.conf `

将 `SERVERNAME=portsnap.FreeBSD.org` 修改为 `SERVERNAME=freebsd-portsnap.mirror.bjtulug.org`

获取 portsnap 更新

```sh
# portsnap fetch extract
```

故障排除:

Snapshot appears to have been created more than one day into the future!

(Is the system clock correct?)

Cowardly refusing to proceed any further.

需要同步时间。

```sh
ntpdate ntp.api.bz
```

### http://FreeBSD.cn （暂不可用）

编辑 portsnap 配置文件 `# ee /etc/portsnap.conf`

将 `SERVERNAME=portsnap.FreeBSD.org` 修改为 `SERVERNAME=portsnap.FreeBSD.cn`

### freebsd-update 源:提供基本系统更新

注意：只有一级架构的 release 版本才提供该源。也就是说 current 和 stable 是没有的。关于架构的支持等级说明请看：
 
<https://www.freebsd.org/platforms/>

### 北京交通大学自由与开源软件镜像站

编辑 `# ee /etc/freebsd-update.conf` 文件:

将 `ServerName update.FreeBSD.org` 修改为 `ServerName freebsd-update.mirror.bjtulug.org`

例:从 FreeBSD 12 升级到 13.0

```bash
# freebsd-update -r 13.0-RELEASE upgrade
```

### http://FreeBSD.cn （暂不可用）

编辑 `# ee /etc/freebsd-update.conf` 文件:

将 `ServerName update.FreeBSD.org` 修改为 `ServerName update.FreeBSD.cn`


> 原文： <https://www.cnblogs.com/FreeBSD-CN/p/15751253.html>

EOF


