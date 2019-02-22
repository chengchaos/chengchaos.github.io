---
title: Ubuntu 使用 LVM
key: 20190119
tags: Linux Ubuntu LVM
---

那台跟了我很久的古老的笔记本，ThinkPad T420i，后来一直使用 Ubuntu。我给他添加了一块固态硬盘，下面是如何将另外一块硬盘挂载的笔记。


<!--more-->


## 硬盘分区

首先，将硬盘物理分区转换成 Linux LVM 使用的物理区段。

```bash
chengchao@t420i:~$ sudo -i
root@t420i:~# fdisk -l
...
Disk /dev/sdb: 232.9 GiB, 250059350016 bytes, 488397168 sectors
...
root@t420i:~# fdisk /dev/sdb
命令(输入 m 获取帮助)： n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
分区号 (1-4, default 1): 
First sector (2048-488397167, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-488397167, default 488397167): 

Created a new partition 1 of type 'Linux' and of size 232.9 GiB.

命令(输入 m 获取帮助)： p
Disk /dev/sdb: 232.9 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x475000bd

设备       启动 Start    末尾    扇区   Size Id 类型
/dev/sdb1        2048 488397167 488395120 232.9G 83 Linux

命令(输入 m 获取帮助)： t
Selected partition 1
Partition type (type L to list all types): L

 0  空              24  NEC DOS         81  Minix / 旧 Linu bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux 交换 / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux 扩展      c7  Syrinx         
 5  扩展            41  PPC PReP Boot   86  NTFS 卷集       da  非文件系统数据 
 6  FAT16           42  SFS             87  NTFS 卷集       db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux 纯文本    de  Dell 工具      
 8  AIX             4e  QNX4.x 第2部分  8e  Linux LVM       df  BootIt         
 9  AIX 可启动      4f  QNX4.x 第3部分  93  Amoeba          e1  DOS 访问       
 a  OS/2 启动管理器 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad 休 ea  Rufus alignment
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs        
 f  W95 扩展 (LBA)  54  OnTrackDM6      a6  OpenBSD         ee  GPT            
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/
11  隐藏的 FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC  
12  Compaq 诊断     5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      
14  隐藏的 FAT16 <3 61  SpeedStor       ab  Darwin 启动     f4  SpeedStor      
16  隐藏的 FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS 次要       
17  隐藏的 HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    
18  AST 智能睡眠    65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 
1b  隐藏的 W95 FAT3 70  DiskSecure 多启 bb  Boot Wizard 隐  fd  Linux raid 自动
1c  隐藏的 W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        
1e  隐藏的 W95 FAT1 80  旧 Minix        be  Solaris 启动    ff  BBT            
Partition type (type L to list all types): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

命令(输入 m 获取帮助)： p
Disk /dev/sdb: 232.9 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x475000bd

设备       启动 Start    末尾    扇区   Size Id 类型
/dev/sdb1        2048 488397167 488395120 232.9G 8e Linux LVM

命令(输入 m 获取帮助)： w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

## 安装 lvm2

```bash
root@t420i:~# apt install lvm2
root@t420i:~# systemctl enable lvm2-lvmetad.service
root@t420i:~# systemctl enable lvm2-lvmetad.socket
root@t420i:~# systemctl start lvm2-lvmetad.service
root@t420i:~# systemctl start lvm2-lvmetad.socket

```

### 1 使用分区创建真实的物理卷

```bash
root@t420i:~# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
```

### 2 创建卷组

```bash
root@t420i:~# vgcreate Vol1 /dev/sdb1
  Volume group "Vol1" successfully created
root@t420i:~# vgdisplay
```

### 3 创建逻辑卷

```bash
root@t420i:~# lvcreate -l 100%FREE -n lvchaos Vol1
  Logical volume "lvchaos" created
root@t420i:~# lvdisplay
```

### 4 创建文件系统

```bash
root@t420i:~# mkfs.ext4 /dev/Vol1/lvchaos
mke2fs 1.42.s13 (17-May-2015)
Creating filesystem with 61048832 4k blocks and 15269888 inodes
Filesystem UUID: 8f328065-54d4-444a-b491-6db6e224311d
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (32768 blocks): 完成
Writing superblocks and filesystem accounting information: 完成 
root@t420i:~# mkdir /work
root@t420i:~# mount /dev/Vol1/lvchaos /work
    
```

### 5 编辑 /etc/fstab 文件

```bash
/dev/Vol1/lvchaos       /work   ext4    rw,noatime      0       0
```

### 6 其他命令

| 命令 | 功能 |
| ---- | ----|
| vgchange | 激活或禁用卷组 |
| vgremove | 删除卷组 |
| vgextend | 将物理卷添加到卷组中 |
| vgreduce | 从卷组中删除物理卷 |
| lvextend | 增加逻辑卷大小 |
| lvreduce | 减小逻辑卷大小  |














If you like TeXt, don't forget to give me a star :star2:.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
