---
layout: wiki
title: Proxmox
categories: proxmox
description: -
keywords: proxmox
---

### LXC & fuse
>>> https://myatus.com/p/quick-note-fuse-inside-proxmox-lxc-container/
Proxmox’ LXC containers do not have the /dev/fuse device created automatically.
A quick way of doing that is by adding the following two lines to the container’s configuration on the host node (in `/etc/pve/lxc/<$container_id>.conf`):
```
lxc.autodev: 1
lxc.hook.autodev: sh -c "mknod -m 0666 ${LXC_ROOTFS_MOUNT}/dev/fuse c 10 229"
```

### LXC & loop
`/etc/pve/lxc/<$container_id>.conf`
```
lxc.aa_profile = unconfined
lxc.cgroup.devices.allow = b 7:* rwm
lxc.cgroup.devices.allow = c 10:237 rwm
```