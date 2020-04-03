---
layout: post
title: 谈谈如何使用docker，搭建一台“群晖”
categories: [NAS, unRAID, docker, 自建家用NAS计划]
description:
keywords: docker, 群晖, NAS
---

## 写在前面
点开这篇文章的朋友，我相信应该已经尝试过很多NAS系统了，例如OMV、FreeNAS、群晖。。。

大部分人在折腾过后，还是义无反顾地重回群晖的怀抱，即使明知道自己是个黑户，说不定哪天会被官方制裁，落入地狱。。。但架不住群晖的生态确实完善，各种APP都有，平常用起来顺手啊。 

开源软件就比不上群晖吗？恰恰相反，大部分开源软件制作的都要更加精良。只不过并未组合成为群晖那样的功能整体，下面就来详细说说如何让它们发挥1+1>2的价值，安排的明明白白。。。 


## 总体设计和使用体验
先看一下总体设计与最终效果。

[!nas_docker_software_simple](../images/blog/2020-02-27-docker-software/nas_docker_software_simple.jpg)

[!heimdall_dashboard](../images/blog/2020-02-27-docker-software/heimdall_dashboard.png)


### 说说我的使用体验
【居家中使用】

我喜欢将上图的统一入口，保存到收藏夹，需要时顺手一点就开，

我会把NAS挂载为“网络磁盘”，这样就可以像操作本地硬盘一样操作里面的文件，

需要下载大型文件时，就打开aria2或qBittorrent进行离线下载，小文件则直接保存到“网络磁盘”，

需要看电影时，打开Jellyfin  


最最重要的是，登录使用的都是同样的账户密码，用户在“统一认证”中管理（作者下篇文章会详细介绍统一认证）。


【公司时使用】

我在公司时，一般会将重要的工作文件进行同步，防止文件意外丢失，

具体做法是使用Nextcloud客户端的同步功能，保证工作效率的同时，备份文件到NAS，

必要时，通过wireguard拨号回内网，访问其他服务。


【手机端使用】

手机端我目前只有备份照片这一个需求，使用nextcloud的APP可以轻松完成备份。


## 开始搭建系统
搭建整个【系统】唯一的前提条件是docker环境！！！如果没有docker可以不用往下看了。。。 

本文提到的所有服务均运行在docker容器中，所以无论哪个平台都可以使用，群晖、、OMV（等.等.等.等）。。。

搭建的过程并不算复杂，下面会详细地对配置进行说明，相信小白也一定可以看明白。



先看个总览，整个系统的搭建列表。。。 

[!​服务列表](../images/blog/2020-02-27-docker-software/unRAID_docker_list.png)


作者目前使用的是unRAID，所以本文就已unRAID进行演示说明，

如果使用其他的平台，建议用docker-compose进行部署，但注意一定要配置好linux文件权限，否则是用不了的。。。

为了方便大家，作者整理了本文相关的docker-compose配置模板，详情见这里： [配置模板(Github)](https://github.com/htpcassistant/htpc-docker-compose)。

`简单说明一下，unRAID是一款国外的NAS系统，感兴趣的可以自己了解一下。`



### 应用市场
首先在【Plugins】里安装应用市场，填入以下地址后，点击【Install】。

`https://raw.githubusercontent.com/Squidly271/community.applications/master/plugins/community.applications.plg`

### mariadb
> mariadb是一款开源数据库软件，主要为nextcloud提供配置信息的存储服务。

安装方法：应用市场

镜像名称：linuxserver/mariadb

配置说明：【MYSQL_ROOT_PASSWORD】项设置为数据库root用户密码，其他保持默认。

[mariadbmariadb]

### nextcloud
> nextcloud属于文件私有云，搭建一个自己的云盘。

安装方法：应用市场

镜像名称：linuxserver/nextcloud

配置说明：【/data】目录配置为所有用户保存文件的根目录，也可以像作者一样单独配置某用户的目录，例如【/data/sweet】配置为【/mnt/user/personal/homes/sweet/】，其他保持默认。

[nextcloud](../images/blog/2020-02-27-docker-software/nextcloud.png)
界面演示：

[界面演示](../images/blog/2020-02-27-docker-software/nextcloud_preview.png)

### jellyfin
> jellyfin是一款开源的影音播放软件，最有用的是根据影片信息，自己从网上下载回来影片海报。

安装方法：应用市场

镜像名称：linuxserver/jellyfin

配置说明：【/movies】目录配置为自己的影片库，其他保持默认。

[jellyfin](../images/blog/2020-02-27-docker-software/jellyfin.png)
界面演示：

[jellyfin](../images/blog/2020-02-27-docker-software/jellyfin_preview.png)

### aria2
> aria2是一款下载工具，易于和BaiduExporter及PanDownload等第三方软件集成。

安装方法：手动安装

镜像名称：john123951/aria2-with-webui

配置说明：【/download】目录配置为下载保存到的文件夹，【SECRET】配置为RPC秘钥，其他保持默认。

[aria2](../images/blog/2020-02-27-docker-software/aria2.png)
界面演示：

[aria2](../images/blog/2020-02-27-docker-software/aria2_preview.png)

### qBittorrent
> qBittorrent是一款BT下载工具，作者推荐的这个版本屏蔽了迅雷吸血，增加tracker资源服务器。

安装方法：手动安装

镜像名称：superng6/qbittorrentee

配置说明：【/downloads】目录配置为下载保存到的文件夹，其他保持默认。

[qbittorrentee](../images/blog/2020-02-27-docker-software/qBittorrent.png)
界面演示：

[qBittorrent](../images/blog/2020-02-27-docker-software/qBittorrent_preview.png)

### chronos
> chronos是一款可以定时执行python脚本的工具，使用者需要会一些简单的python，作者常使用它自动签到、监控商品价格。

安装方法：应用市场

镜像名称：simsemand/chronos

配置说明：全部保持默认即可。

[chronos](../images/blog/2020-02-27-docker-software/chronos.png)
界面演示：

[chronos](../images/blog/2020-02-27-docker-software/chronos_preview.png)


## 尾言
本篇主要介绍了如何整合各个服务的思路，并没有深究其中很多细节，一则是站内有许多保姆级教程，二则实操部分的乐趣也想留给大家自己体验。

下一篇文章中，我会进行更深入的整合，打通各服务的登录账号，使用“认证中心”管理用户状态。

---
本文提到的docker-compose文件：https://github.com/htpcassistant/htpc-docker-compose






