---
layout: post
title: 自建家用NAS计划 ---- Linux RAID & LVM 数据存储篇
categories: [NAS, pve, virtualization, openmediavault]
description: 
keywords: NAS, design
---

> [Proxmox VE 让硬盘休眠的办法](https://blog.myds.cloud/archives/proxmox-ve-spin-down-hard-disk.html)


### 解决 Proxmox VE 无法硬盘休眠问题
经过搜索资料，PVE下硬盘无法休眠是状态服务（pvestatd）不停读取硬盘状态的原因。状态服务（pvestatd）负责将硬盘状态等信息显示在首页的管理界面，同时与服务器群组交换状态数据。
解决办法就是将这个服务停掉（pvestatd），后果就是首页不会再更新状态了，比如cpu，网卡负载等状态信息。

1. 硬盘未休眠
`smartctl -i -n standby /dev/sdb`
```shell
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-4.15.18-13-pve] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Model Family:     Western Digital Blue
Device Model:     WDC WD5000AAKX-00ERMA0
Serial Number:    WD-WCC2E1HRZV9E
LU WWN Device Id: 5 0014ee 26239bfa0
Firmware Version: 15.01H15
User Capacity:    500,107,862,016 bytes [500 GB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    7200 rpm
Device is:        In smartctl database [for details use: -P show]
ATA Version is:   ATA8-ACS (minor revision not indicated)
SATA Version is:  SATA 3.0, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Thu May 30 15:45:52 2019 CST
SMART support is: Available - device has SMART capability.
SMART support is: Enabled
Power mode is:    ACTIVE or IDLE
```

2. 关闭pvestatd服务
`pvestatd stop && systemctl disable pvestatd`

3. 检查服务状态
`pvestatd status`
```shell
stopped
```

4. 立即休眠硬盘
`hdparm -y /dev/sdb` 或 `hdparm -Y /dev/sdb`
```
/dev/sdb:
 issuing sleep command
```

5. 再次检查硬盘是否休眠
`smartctl -i -n standby /dev/sdb`
```shell
smartctl 6.6 2016-05-31 r4324 [x86_64-linux-4.15.18-13-pve] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

Device is in STANDBY mode, exit(2)
```

5. 配置自动休眠

6. 查看硬盘温度
`hddtemp /dev/sdb`
