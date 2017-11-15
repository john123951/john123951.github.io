---
layout: wiki
title: crontab
categories: linux
description: crontab 格式。
keywords: linux, cron, crontab
---

#### crontab格式

```
#+------------ Minute            (range: 0-59)
#| +---------- Hour              (range: 0-23)
#| | +-------- Day of the Month  (range: 1-31)
#| | | +------ Month of the Year (range: 1-12)
#| | | | +---- Day of the Week   (range: 1-7, 1 standing for Monday)
#| | | | | +-- Year              (range: 1900-3000)
#| | | | | | 
#* * * * * *
```
