---
layout: post
title: x86主机搭建家庭智能路由系统 ---- Proxmox虚拟化实现一机多用
categories: [软路由, virtualization]
description: 
keywords: proxmox, virtualization
---

### Proxmox VE简介
[Proxmox VE](http://pve.proxmox.com/wiki/Main_Page)(Proxmox Virtual Environment) 是一款完全开源虚拟化管理平台，可以管理QEMU/KVM虚拟机和LXC容器。事实上它只是一个前端管理界面，虚拟化技术由KVM和LXC提供。


### 安装Proxmox VE
首先到官网下载Promox VE的镜像文件。
下载地址：https://www.proxmox.com/en/downloads/item/proxmox-ve-4-4-iso-installer

下载完成后，使用dd命令或者USBWriter将镜像内容写入U盘,制作引导盘。
`dd if=proxmox-ve_4.4-eb2d6f1e-2.iso of=/dev/sdc bs=4m`

开始安装前，先用网线连接x86主机和路由器（目的是为了我的笔记本可以访问Proxmox的web界面），然后插入U盘进行引导，出现如下安装界面：
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309174740156-1975372171.png)

按照提示,分别设置root密码，IP地址，直至安装完成并重启，安装完成后的界面如下。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309174851672-1879544313.png)


### 配置Proxmox VE
启动Proxmox VE后会提示访问网址，使用笔记本访问 https://192.168.1.100:8006 ,并输入 root/刚刚设置的密码 进行登录。
我到这里时遇到了第一个坑，打不开网页。仔细检查后发现，Proxmox默认只开启了第一块网卡，而我的主机装有两块网卡，并且连接路由器的网线插到了第二块网卡上，解决办法是将网线插到主板自带的第一块网卡，或者更改网络配置，启用第二块网卡并设置默认路由。

成功登录后界面如下，默认支持中文。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309223118641-2003905449.png)

首先在配置中创建一块虚拟网卡，桥接我的第二张物理网卡，点击左侧“节点”--“System”--“网络”，创建一块“vmbr1”桥接到“eth1”，重启使配置生效。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309223358375-1545041836.png)


### 安装虚拟机
Proxmox支持两种类型的虚拟机，管理界面右上角的“创建虚拟机”会创建KVM虚拟机，“创建CT”则是创建OpenVZ虚拟机。

此处使用KVM虚拟机，创建虚拟机前，需要先将ISO镜像文件上传到服务器中，点击左侧“存储”--“local”--“内容”，上传ISO文件。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309223812672-1604382146.png)

点击右上角“创建虚拟机”，然后输入一个名字，我这里使用“pfSense”。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309224125141-1804444602.png)

下一步，根据需要选择操作系统、IOS文件、硬盘大小、CPU核心数、内存大小以及网络，注意选择网络时只可以选择一块网卡，但可以完成后在虚拟机的硬件配置中添加另一块网卡。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309224537750-324818030.png)

所有配置完成后，点击页面上部的“启动”，虚拟机就跑起来了。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309225106453-257642440.png)


### 总结
通过实践，Proxmox VE的易用性还是很高的，主要功能都可以在web中管理，安装虚拟机也非常方便。

性能方面，开机禁用所有虚拟机占用660M内存，CPU不足1%（CPU图中左侧的峰值是我重启前的数据），个人感觉内存占用比较大，考虑到Proxmox还跑了个Debian和Java这个内存占用也还能接受。
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170309230105672-1985455593.png)

好了，文章至此结束，下一篇中，我会配置pfSense作为软路由进行拨号上网。
