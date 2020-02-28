---
layout: post
title: 自建家用NAS计划 ---- Linux RAID & LVM 数据存储篇
categories: [NAS, pve, virtualization, openmediavault]
description: 
keywords: NAS, design
---

### 启动过程中卡在random: crng init done
> https://unix.stackexchange.com/questions/442698/when-i-log-in-it-hangs-until-crng-init-done
```
apk add haveged
rc-update add haveged
```


### Google Auth
> https://wiki.alpinelinux.org/wiki/Two_Factors_Authentication_With_OpenSSH
`apk add google-authenticator openssh-server-pam`

`cat /etc/ssh/sshd_config`
```
# google
#AuthenticationMethods password publickey,keyboard-interactive
ChallengeResponseAuthentication yes
PermitRootLogin no
UsePAM yes
```

`cat /etc/pam.d/sshd || touch /etc/pam.d/sshd`
```
account     include			base-account
password    include			base-password

auth        required       pam_env.so
auth        required       pam_unix.so nullok_secure
auth        required       pam_nologin.so successok

auth        required       pam_unix.so nullok try_first_pass
auth        include        google-authenticator
```

Everyone run command: `google-authenticator`

