---
layout: post
title: x86主机搭建家庭智能路由系统 ---- Webmin搭建家庭数据中心
categories: [软路由, virtualization]
description: some word here
keywords: webmin
---

搭建共享服务器、NAS（网络附加存储），并通过Web方式管理Samba等服务。


### 起因
过年时从家里的旧电脑上拆下一块机械硬盘（NTFS格式且含有资料），回来后发现笔记本想读硬盘里的资料是件很麻烦的事。
一个简单的解决方法是，某宝买一个"硬盘底座"就好了，不过想到家里的小主机一直闲着也是闲着，本着物尽其用的原则，准备自己动手搭一台NAS服务器。

设想中的NAS服务器应该有以下特征：
1. 能读取物理磁盘上的资料。
2. 能在内网中提供文件共享（Samba、NFS、AFP）服务，方便其他设备读取资料。
3. 能提供文件备份服务，定期备份手机相册。

动手尝试了几款出名的NAS系统：[FreeNAS](http://freenas.org)、[Rockstor](http://rockstor.com)，都不太符合我的需求，此类产品为了保证数据的安全性，大部分使用ZFS系统，并限制其他格式，经过尝试虽然还是可以挂载NTFS格式，但转念一想，为什么我要绕一大圈来"破解"系统本来限制的功能？我想要的仅仅只是一个文件共享以及对应的Web管理界面而已。

明确了需求后，我尝试寻找一些带Web管理的操作系统，最后，我找到了本文的主角：[Webmin](http://webmin.com/)。

### Webmin简介
[Webmin](http://webmin.com/)是目前功能最强大的基于Web的Unix系统管理工具。管理员通过浏览器访问Webmin的各种管理功能并完成相应的管理动作。目前Webmin支持绝大多数的Unix系统，这些系统除了各种版本的linux以外还包括：AIX、HPUX、Solaris、Unixware、Irix和FreeBSD等。
(内容摘自百度百科)

简单来说，Webmin提供了一个Web管理平台，使得常见的应用都可以在GUI界面中进行配置，代替那些难记的命令。

这东西简直太符合我的要求了，只需在CentOS上装个Samba，再加个管理界面，NAS就成了。
我不在乎数据的安全性，也不用管那些RAID，磁盘坏了就坏了吧。
我需要的仅仅是一个数据的暂存区，文件在局域网内共享，并且定时将特定数据上传到云盘就好了。

### Proxmox中创建虚拟机
搭建平台前，首先要在Proxmox中创建一台虚拟机，考虑到我只需要跑几个服务，我选择了轻量级容器LXC。
第一次使用LXC，要先下载模板，选择存储中的"local"--"内容"--"模板"，根据需要下载对应模板。

![](/images/blog/2017-05-27-route-webmin/proxmox-createvm.png)

有了模板之后，点击Proxmox控制台右上角“创建CT”，设置root密码，选择要使用的模板，CPU，内存等配置。

![](/images/blog/2017-05-27-route-webmin/proxmox-createvm2.png)

虚拟机创建好了，还需要将物理磁盘进行映射，通过阅读[官方文档](https://pve.proxmox.com/wiki/Linux_Container)，在Host机上执行以下命令：
*注意：LXC容器不可以挂载NFS和块设备*

```language
pct set 104 -mp0 /dev/disk/by-id/ata-WDC_WD5000AAKX-001CA0_WD-WMAYUM449361-part1,mp=/storage
pct set 104 -mp0 /dev/disk/by-id/ata-WDC_WD5000AAKX-001CA0_WD-WMAYUM449361-part1(windows)
```

如果使用KVM，也可以映射物理磁盘（[官方文档](https://pve.proxmox.com/wiki/Physical_disk_to_kvm)），使用以下命令：
```bash
# 将104换成虚拟机ID，by-id后名称替换为自己硬盘
qm set 104 -virtio2 /dev/disk/by-id/ata-WDC_WD5000AAKX-001CA0_WD-WMAYUM449361
```

查看虚拟机信息，可以看到磁盘或者分区已经挂载好了。

![](/images/blog/2017-05-27-route-webmin/proxmox-hardware.png)

**这里虽然成功挂载了，实际使用时会提示Read-Only file system，解决方法是在宿主机安装ntfs-3g：**
```language
yum install ntfs-3g
```

### 安装Webmin
Webmin提供了[多种安装包](http://webmin.com/download.html),其中当然包换CentOS，打开虚拟机shell，使用命令下载RPM包并安装：
```language
cd /tmp
curl -O http://prdownloads.sourceforge.net/webadmin/webmin-1.840-1.noarch.rpm	#注意是大写字母O
yum install webmin-1.840-1.noarch.rpm
```

![](/images/blog/2017-05-27-route-webmin/proxmox-console.png)

安装完成后，访问 https://webmin:10000 就可以看到Webmin的真容了。

![](/images/blog/2017-05-27-route-webmin/webmin-dashboard.png)

### 安装Windows共享
开启Windows共享需要安装Samba，在虚拟机的shell中执行命令：
`yum install samba`

安装完成后，点击Webmin中的"Refresh Modules"，就可以看到新添加的Samba模块了。

![](/images/blog/2017-05-27-route-webmin/webmin-servers.png)

#### 添加Samba用户
选择Samba模块中的"Convert Users"，将Linux系统中用户转换为Samba用户。
"Samba Users"中可以修改用户信息。

![](/images/blog/2017-05-27-route-webmin/webmin-samba-users.png)

#### 添加Samba共享
点击"Create a new file share"来添加一个共享，勾选"Available"和"Browseable"。

编辑添加好的共享，选择"File Permissions"--"Force Unix user"，输入"root"，以保证有足够的权限访问该文件夹。

![](/images/blog/2017-05-27-route-webmin/webmin-samba-share.png)

![](/images/blog/2017-05-27-route-webmin/webmin-samba-edit-share.png)

![](/images/blog/2017-05-27-route-webmin/webmin-samba-permissions.png)

配置好后，点击Samba模块下方"Start Samba Servers"开启服务，Windows中就可以顺利访问共享了。

### 安装Linux共享、Mac共享
Linux共享，使用NFS。
Mac共享，使用Netatalk。

Netatalk与Webmin整合需要手动安装模块。
https://sourceforge.net/projects/netatalk/files/Webmin/
https://github.com/Netatalk/webmin-module

Linux、Mac共享，只需安装上述两个包，而后在GUI界面中进行配置，这里就不再阐述。

### 总结
Webmin提供了另一种操作Unix系统的方式，使其可以在Web端管理服务器，甚至支持移动设备（虽然小屏幕看起来乱糟糟的）。
这篇文章只是抛块砖，大家可以发挥更多的想象力，让程序给我们提供便利。

下次我将介绍手机同步软件FolderSync及云盘同步软件，打造自动备份系统，让丢照片成为过去。