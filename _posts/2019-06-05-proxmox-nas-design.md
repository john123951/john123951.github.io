---
layout: post
title: 自建家用NAS计划 ---- 设计篇
categories: [NAS, pve, virtualization, 自建家用NAS计划]
description: 
keywords: NAS, design
---

### 需求分析
家庭里的“NAS”用了有一段时间了，都是基于Docker跑的零散的服务，随着时间的推移，使用起来越来越不顺手，由此产生了搭建一套真正NAS系统，实现文件共享的想法。
整理了一下，我的要求并不高：
- 统一的存储中心，家里或户外可以随时访问
- 数据要保证安全，不能出现文件丢失的情况
- 简单的影音中心，保存学习视频的播放进度
- 成本不要太高，功耗也不要太高

![](/images/blog/2019-06-05-proxmox-nas-design/nas.png)


### 方案比较
- [群晖](https://demo.synology.cn/zh-cn) 国人体验最好，不过价格昂贵
- [万由 U-NAS](http://www.u-nas.cn/products-uns-os.html) 体验也很好，操作系统可以免费下载
- [OpenMediaVault](https://www.openmediavault.org/) 国外的一款NAS系统，基于Debian
- [FreeNAS](https://www.freenas.org/) 国外的一款NAS系统，基于FreeBSD
- [XigmaNAS(NAS4Free)](https://www.xigmanas.com/) 国外的一款NAS系统，基于FreeBSD

群晖和万由的系统，可以在官网进行体验。OMV、FreeNAS、XigmaNAS系统可以去官网查看截图。
网络上各系统比较的文章较多，在这里就不过多赘述了，请大家自行搜索。


### 最终方案
上面的几种方案虽然都可以满足我的需求，但考虑到实际情况，
一方面，家里的Proxmox性能过剩，正好可以利用起来，省去添置新设备，
另一方面，由于个人不太喜欢群晖和万由的Web桌面体验，仅仅只需要SMB及其他服务的管理界面就够了，
所以在此选择 [OpenMediaVault] 作为NAS系统，移动端的APP也使用开源方案替代。

服务列表
- SMB：用于Windows共享
- FTP：用于手机数据同步
- WebDAV: 用于与其他应用集成
- Duplicati：数据自动备份
- Nextcloud：私有云盘
- Plex：家族影音中心
- Emby：另一款影音中心
- Aria2Ng：下载器+Web界面
- FolderSync：手机照片同步

![](/images/blog/2019-06-05-proxmox-nas-design/nas-omv.png)


自此，NAS大体轮廓已经确定，下一步进行数据存储平台的搭建。

