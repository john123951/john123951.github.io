---
layout: post
title: 自建家用NAS计划 ---- 设计篇
categories: [NAS, pve, virtualization, 自建家用NAS计划]
description: 
keywords: NAS, design
---

### 需求分析
家庭里有一台小主机一直客串NAS的角色，平常做着挂机下载，放放电影的活。
由于各个服务都是零散地搭建起来的，导致登录地址和账户密码也很零散。

随着时间的推移，这台“NAS”使用起来就越来越不顺手，
由此产生了搭建一套真正NAS系统，统一管理存储的想法。

整理了一下需求，我的要求并不高：
- 统一的存储中心，家里或户外可以随时访问
- 数据要保证安全，不能出现文件丢失的情况
- 简单的影音中心，保存学习视频的播放进度
- 硬件成本别太高，运行时的功耗也不要太高

![](/images/blog/2019-06-05-proxmox-nas-design/nas.png)


### NAS平台比较
- [群晖](https://demo.synology.cn/zh-cn) 国人体验最好，不过买硬件才能用软件，价格昂贵
- [万由 U-NAS](http://www.u-nas.cn/products-uns-os.html) 体验也很好，操作系统可以免费下载
- [OpenMediaVault](https://www.openmediavault.org/) 国外的一款NAS系统，基于Debian，免费下载
- [FreeNAS](https://www.freenas.org/) 国外的一款NAS系统，基于FreeBSD，免费下载
- [XigmaNAS(NAS4Free)](https://www.xigmanas.com/) 国外的一款NAS系统，基于FreeBSD，免费下载

群晖和万由的系统，均可以在官网进行体验。
OMV、FreeNAS、XigmaNAS系统官网可以查看截图。

总体来说，
群晖、万由的系统更符合国人操作习惯且自带移动端APP。
OpenMediaVault 基于Debian，需要使用者对Linux比较熟悉。
FreeNAS、XigmaNAS 默认使用ZFS文件系统，对内存要求较高（建议8G）。


### 最终方案
综合考虑后，
一方面，群晖的性价比较低，不打算掏钱去购置硬件（但是群晖系统是真的优秀），
另一方面，个人也不太喜欢群晖和万由的Web桌面体验，喜欢简单直接，能对系统的控制度大一些，关闭不常用的服务，
所以最终选择 [OpenMediaVault] 作为NAS系统，移动端的APP也使用开源方案替代。

简单介绍下使用的服务组件：
- SMB：文件共享协议，用于Windows系统访问共享文件
- FTP：文件传输协议，用于上传下载文件
- WebDAV: 用于与第三方应用集成
- Duplicati：数据自动备份
- Nextcloud：优秀的私有云盘
- Plex：优秀的家庭影音中心
- Emby：另一款影音中心，胜在开源
- Aria2Ng：远程下载器+Web界面
- FolderSync：手机端APP，配合SMB或FTP进行照片同步

![](/images/blog/2019-06-05-proxmox-nas-design/nas-omv.png)


### 硬件配置

| 推荐配置 | 推荐配置信息                    | 参考价格                   |
| -------- | ------------------------------- | -------------------------- |
| 主板     | 华擎J3455-ITX                   | 529元                      |
| CPU      | J3455 四核                      | -                          |
| 内存     | 威刚（ADATA）DDR3L 1600 4GB * 2 | 135 * 2 = 270元            |
| 固态硬盘 | 闪迪（SanDisk）120GB            | 149元                      |
| 机械硬盘 | 西部数据 500G * 2               | 60 * 2 = 120元（某鱼价格） |
| 电源     | 金河田电源 200W                 | 99元                       |
| 机箱     | JONSBO乔思伯全铝ITX机箱C2       | 155元                      |
| 合计     | -                               | 1322元                     |


| 新手尝鲜 | 尝鲜配置信息      | 参考价格                   |
| -------- | ----------------- | -------------------------- |
| 套餐     | 星际蜗牛D款       | 300元（某鱼价格）          |
| 机械硬盘 | 西部数据 500G * 2 | 60 * 2 = 120元（某鱼价格） |
| 合计     | -                 | 420元                      |



OK，本篇主要梳理清楚NAS需求，并对使用的软件进行选型。
下一篇文章开始动手，搭建数据存储平台。

