---
layout: post
title: 自建家用NAS计划 ---- Linux RAID & LVM 数据存储篇
categories: [NAS, pve, virtualization, openmediavault]
description: 
keywords: NAS, design
---

> [Proxmox VE 让硬盘休眠的办法](https://blog.myds.cloud/archives/proxmox-ve-spin-down-hard-disk.html)

### 数据安全性

### 数据备份方案

codes -- pc -- gogs -- github
photos -- mobile -- ftp -- encrypt -- gzip -- baidu

### 解决 Proxmox VE 无法硬盘休眠问题
PVE下默认启动的状态服务（pvestatd）会不停读取硬盘信息，导致硬盘休眠后马上被唤醒。
解决办法是将这个服务停掉（pvestatd），后果就是首页不会再更新状态了，比如cpu，网卡负载等状态信息。

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