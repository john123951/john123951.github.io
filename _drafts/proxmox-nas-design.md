---
layout: post
title: 自建家用NAS计划 ---- 设计篇
categories: [NAS, pve, virtualization]
description: 
keywords: NAS, design
---

### 需求分析
近期用U盘拷贝资料到公司，拷来拷去发现总是分不清哪些文件是最新的，用网盘又不安全，由此产生了搭建私有NAS实现文件共享的想法。
整理了一下，我的要求并不高：
- 统一的存储中心，家里或户外可以随时访问
- 数据要保证安全，不能出现文件丢失的情况
- 简单的影音中心，保存学习视频的播放进度
- 成本不要太高，功耗也不要太高
[图片]


### 方案比较
- [群晖](https://demo.synology.cn/zh-cn) 国人体验最好，不过价格昂贵
- [万由 U-NAS](http://www.u-nas.cn/products-uns-os.html) 体验也很好，操作系统可以免费下载
- [OpenMediaVault](https://www.openmediavault.org/) 国外的一款开源NAS，基于Debian
- [FreeNAS](https://www.freenas.org/) 国外的一款NAS，基于FreeBSD
- [XigmaNAS(NAS4Free)] 国外的一款NAS，基于FreeBSD

群晖和万由的系统，可以在官网进行体验。OMV、FreeNAS、XigmaNAS系统可以去官网查看截图。
网络上各系统比较的文章较多，在这里就不过多赘述了，请大家自行搜索。


### 最终方案
上面的几种方案都可以满足我的需求，考虑到实际情况，
一方面，家里的Proxmox性能过剩，正好利用起来，不再添置新设备了，
另一方面，由于个人不太喜欢群晖和万由的Web桌面体验，仅仅只需要SMB及其他服务的管理界面就够了，
所以在此选择 [OpenMediaVault] 作为NAS系统，移动端的APP也使用开源方案替代。

软件列表
- SMB：用于Windows共享
- FTP：用于手机数据同步
- Duplicati：用于数据备份
- Nextcloud：个人云盘
- Plex：个人影音中心
- Emby：另一款影音中心
- Aria2Ng：下载器+Web界面
- FolderSync：手机照片同步
- 

[图片]




