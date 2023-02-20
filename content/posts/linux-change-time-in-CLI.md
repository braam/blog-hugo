---
title: "Linux Change Time in CLI"
date: 2022-02-01T11:32:57+01:00
draft: false
tags:
- Linux
- OSBiz
---

The Openscape Business S (server editions) run in SLES linux, time for these systems is crucial! Sometimes we only have access with SSH and not with the GUI, this note is to remember on how to quickly change the time from the terminal.

```
date +%T -s "10:11:12"
```
