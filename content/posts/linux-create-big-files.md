---
title: "Linux Create Big Files"
date: 2024-10-04T11:34:26+02:00
draft: false
tags:
- Linux
---

Sometimes it's needed to created a dummy file, it would also be handy to be able to give the filesize you want it to be. Following command is almost possible on all linux distributions:

```
$> fallocate -l 1G onegigabytefile.text
```