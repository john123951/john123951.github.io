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


#### elasticsearch-job-cron

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    |
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, optional)
```