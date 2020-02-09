---
layout: post
title: 自建家用NAS计划 ---- OpenMediaVault搭建家庭数据中心
categories: [NAS, pve, virtualization, openmediavault, 自建家用NAS计划]
description: some word here
keywords: keyword1, keyword2
---


#### 安装 OpenMediaVault

默认账号是admin 密码是openmediavault

omv-extras.org


#### 存储
qm set 100 -virtio1 /dev/raid1VG/raid1
qm set 100 -virtio2 /dev/storageVG/storage




