---
title: CentOS 7 升级内核
key: 2020-11-18
tags: linux centos7 kernel
---

The kernel is the core of an operating system. The Linux kernel is the monolithic Unix-like kernel of the Linux computer operating system. It was created by Linux Torvalds, and all Linux distributions including Ubuntu, CentOS and Debian are based on this kernel - the Linux kernel.

<!--more-->

In this tutorial, I will show you how to upgrade the CentOS 7 kernel to the latest version. We will use a precompiled kernel from the ELRepo repository. By default CentOS 7 uses the kernel 3.10. In this manual we will install the latest stable kernel version 5.0.11.

## What is the ELRepo

ELRepo is a community-based repository for Enterprise Linux and supports for RedHat Enterprise (RHEL) and other distribution based on it (CentOS, Scientific, and Fedora).

ELRepo has the focus on packages related to hardware, including filesystem drivers, graphic drivers, network drivers, sound card drivers, webcam, and others.

## What we will do

- Update and Upgrade CentOS 7
- Checking the Kernel Version
- Add ELRepo Repository
- Install New Kernel Version
- Configure Grub2
- Remove Old Kernel

## 第 1 步，更新系统

升级内核之前要作得第一件事事将系统中的所有包都更新到最新版本。使用以下命令：

```bash
# yum -y update
```

然后安装 yum plugin 使用比较块的镜像

```bash
# yum -y install yum-plugin-fastestmirror
Loaded plugins: fastestmirror, langpacks, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

Loading mirror speeds from cached hostfile
 * epel: mirror.lzu.edu.cn
Package yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch already installed and latest version
Nothing to do
```

## 第 2 步，检查内核版本

使用以下命令：

```bash
# cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

```

还可以使用 `uname` 命令：

```bash
# uname -snr
Linux peer1 3.10.0-1062.12.1.el7.x86_64
```

## 第 3 步，添加 ELRepo 仓库

升级内核前，我们需要添加一个新的仓库 -- ELRepo repository。

添加 ELRepo gpg key 到系统：

```bash
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

使用 `rpm` 命令添加新的 ELRepo repository：

```bash
# rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
Retrieving https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:elrepo-release-7.0-3.el7.elrepo  ################################# [100%]
```

命令执行完成后，检查一下 repository 在系统中是否事 enable 状态，并确认 ELREpo 在列表中：

```bash
# yum repolist
Loaded plugins: fastestmirror, langpacks, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

Loading mirror speeds from cached hostfile
 * elrepo: mirror.rackspace.com
 * epel: mirror.lzu.edu.cn
elrepo                                                                                                  | 2.9 kB  00:00:00
elrepo/primary_db                                                                                       | 481 kB  00:00:00
repo id                                      repo name                                                                   status
base/7/x86_64                                CentOS-7 - Base                                                             10,070
docker-ce-stable/x86_64                      Docker CE Stable - x86_64                                                       82
elrepo                                       ELRepo.org Community Enterprise Linux Repository - el7                         130
epel/x86_64                                  Extra Packages for Enterprise Linux 7 - x86_64                              13,470
extras/7/x86_64                              CentOS-7 - Extras                                                              413
openlogic/7/x86_64                           CentOS-7 - openlogic packages for x86_64                                        22
updates/7/x86_64                             CentOS-7 - Updates                                                           1,134
repolist: 25,321
```

## 第 4 步，安装 CentOS 的新内核

使用 `yum` 命令安装 ELRepo 内核：

```bash
# yum --enablerepo=elrepo-kernel install kernel-ml
```

**`--enablerepo`** 是一个可选项，用于启用 CentOS 系统中指定的仓库。默认情况下， `elrepo` 仓库是 enabled 的，但是 `elrepo-kernel` 不是。

可以检查系统中的可用仓库，使用下面的命令：

```bash
# yum repolist all
```

## 第 5 步，配置 CentOS 7 的 Grub2

使用 `awk` 命令检查 Grub2 中所有可用的内核：

```bash
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.9.8-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1127.19.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1127.13.1.el7.x86_64) 7 (Core)
3 : CentOS Linux (3.10.0-1062.12.1.el7.x86_64) 7 (Core)
4 : CentOS Linux (3.10.0-1062.9.1.el7.x86_64) 7 (Core)
5 : CentOS Linux (3.10.0-957.27.2.el7.x86_64) 7 (Core)
```

此时就可以看到所有的内核版本了。

现在可以配置系统启动使用的默认内核，这里我们使用 5.9.8 当作默认内核：

```bash
# grub2-set-default 0
```

0 - it's from the awk command on the top. Kernel 5.0.11 = 0, and Kernel 3.10 = 1. When you want to back to the old kernel, you can change the value of the grub2-set-default command to 1.

接下来，生成 grub2 配置，使用 `gurb2-mkconfig` 命令：

```bash
#  grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.9.8-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.9.8-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.19.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.19.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.13.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.13.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1062.12.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1062.12.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1062.9.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1062.9.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-957.27.2.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-957.27.2.el7.x86_64.img
done
```

重启系统：

```bash
# reboot
```

重启后，检查内核版本：

```bash
# uname -srn
```

## 第 6 步，移除旧版本的内核（可选）

This is an optional step that is useful to get more free space. In this step, I will show you how to remove an old kernel from your CentOS 7 system. This should be done when you have a more than 3 or 5 kernel versions installed on the server.

For this purpose, we need to install the yum-utils utility from the repository.

```bash
# yum install yum-utils
```

Now clean your old kernel with command below.

```bash
# package-cleanup --oldkernels
```

That means you've only 2 or 3 Kernel versions installed. If you have more than 3 versions installed, the command will automatically remove the old kernel from your system.

CentOS 7 Kernel has been updated to the latest stable using ELRepo Kernel Version.

参考：

- [How to Upgrade the Linux Kernel on CentOS 7](https://www.howtoforge.com/tutorial/how-to-upgrade-kernel-in-centos-7-server/#:~:text=By%20default%20CentOS%207%20uses,11)
- [https://elrepo.org/](https://elrepo.org/)
- [https://wiki.centos.org/HowTos/Grub2](https://wiki.centos.org/HowTos/Grub2)

EOF

---

Power by TeXt.
