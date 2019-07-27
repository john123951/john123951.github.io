---
layout: post
title: 自建家用NAS计划 ---- Linux RAID & LVM 数据存储篇
categories: [NAS, pve, virtualization, openmediavault]
description: 
keywords: NAS, design
---

### 数据存储策略
首先将存储区域划分为两部分：安全存储区域、普通存储区域
- 安全存储区域：存储重要数据，定时自动备份，保证数据不丢失。例如：照片、文档类
- 普通存储区域：存储普通数据，数据丢失不会产生太大影响。例如：下载的视频、压缩包类

由于目前存储的需求不大，故只安装了两块500G硬盘，每块硬盘划出100G空间做RAID1，剩下400G+400G=800G空间存储普通文件。
[图片]


### 搭建RAID1
| RAID 类型 | 说明 |
| --------- | ---- |

#### 第一步：划分磁盘分区
将 `/dev/sdb` `/dev/sdc` 两块硬盘均按以下方式划分分区。

```
# 创建第一个分区：100G RAID，依次输入以下命令
# n 创建新分区
# p 主分区
# +100G 空间大小
# t 改变分区类型
# fd Linux RAID类型
fdisk /dev/sdb

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-976773167, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-976773167, default 976773167): +100G

Created a new partition 1 of type 'Linux' and of size 100 GiB.

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): fd
Changed type of partition 'Linux' to 'Linux raid autodetect'.
```
```
# 剩余空间创建第二个分区，输入 `n` 命令后依次回车
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (209717248-976773167, default 209717248):
Last sector, +sectors or +size{K,M,G,T,P} (209717248-976773167, default 976773167):

Created a new partition 2 of type 'Linux' and of size 365.8 GiB.
```
```
# 输入 `p` 命令查看创建好的分区
Command (m for help): p

Disk /dev/sdb: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xfbc4f911

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sdb1            2048 209717247 209715200   100G fd Linux raid autodetect
/dev/sdb2       209717248 976773167 767055920 365.8G 83 Linux

# 输入 `w` 保存修改并退出
```

#### 组建 Linux RAID1
> https://www.howtoing.com/manage-software-raid-devices-in-linux-with-mdadm/

```
# 使用 `/dev/sdb` 和 `/dev/sdc` 创建RAID1
mdadm --create  /dev/md0 --level=1 --chunk=32 /dev/sd[bc]1

# 查询Raid 状态
mdadm --detail /dev/md0
```


### LVM


### 虚拟机连接存储卷


codes -- pc -- gogs -- github
photos -- mobile -- ftp -- encrypt -- gzip -- baidu

### 解决 Proxmox VE 无法硬盘休眠问题
> [Proxmox VE 让硬盘休眠的办法](https://blog.myds.cloud/archives/proxmox-ve-spin-down-hard-disk.html)
> [hdparm 服务](https://wiki.archlinux.org/index.php/Hdparm)
PVE下默认启动的状态服务（pvestatd）会不停读取硬盘信息，导致硬盘休眠后马上被唤醒。
解决办法是将这个服务停掉（pvestatd），后果就是首页不会再更新状态了，比如cpu，网卡负载等状态信息。

**
修改/etc/lvm/lvm.conf文件
设置use_lvmetad = 1 或者 global_filter = [ "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|", "r|/dev/disk/by-id/ata-WDC*|" ]
**
global_filter = [ "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|" "r|/dev/mapper/.*-(vm|base)--[0-9]+--disk--[0-9]+|", "r|/dev/md.*|", "r|/dev/raid1VG.*|", "r|/dev/storageVG.*|", "r|/dev/sd[bc]|"]


1. 硬盘未休眠
`hdparm -C /dev/sdb` 或 `smartctl -i -n standby /dev/sdb`
```shell
/dev/sdb:
 drive state is:  active/idle

/dev/sdc:
 drive state is:  active/idle
```

2. 关闭pvestatd服务
`pvestatd stop && systemctl disable pvestatd && pvestatd status`

3. 立即休眠硬盘
`hdparm -y /dev/sdb` 或 `hdparm -Y /dev/sdb`
```
/dev/sdb:
 issuing sleep command
```

4. 再次检查硬盘是否休眠
`hdparm -C /dev/sdb` 或 `smartctl -i -n standby /dev/sdb`
```shell
/dev/sdb:
 drive state is:  standby
```

5. 配置自动休眠

6. 查看硬盘温度
`hddtemp /dev/sdb`

### 禁用LVM扫描
```
# vi /etc/lvm/lvm.conf
use_lvmetad = 1

pvscan --cache
```



### Nextcloud
七、最重要的一步，使用OCC命令更新文件索引
以下内容都是在/unas/apps/nextcloud/web目录下执行的，如果没有进入该目录，请看回上面第六大点。

1、使用OCC命令更新文件索引。
occ有三个用于管理Nextcloud中文件的命令：

files files:cleanup                #清除文件缓存 
files:scan                         #重新扫描文件系统 
files:transfer-ownership           #将所有文件和文件夹都移动到另一个文件夹
我们需要使用：

files:scan
来扫描新文件。

格式: files:scan [-p|--path="..."] [-q|--quiet] [-v|vv|vvv --verbose] [--all] [user_id1] ... [user_idN]
参数: user_id #扫描所指定的用户（一个或多个，多个用户ID之间要使用空格分开）的所有文件
​​​​​​​选项: --path #限制扫描路径 --all #扫描所有已知用户的所有文件 --quiet #不输出统计信息 --verbose #在扫描过程中显示正在处理的文件和目录 --unscanned #仅扫描以前未扫描过的文件
以下是一个具体的命令示例：
sudo -u www-data php occ files:scan --all   #扫描所有用户的所有文件
--------------------- 
作者：icarus666 
来源：CSDN 
原文：https://blog.csdn.net/icarus666/article/details/87939420 
版权声明：本文为博主原创文章，转载请附上博文链接！