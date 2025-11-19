---
title: "Linux Disk Space Analyse"
date: 2023-09-07T14:15:11+02:00
draft: false
tags:
- Linux
---

When disk space is almost full, then we need to identify which big files we can remove to make again free space. We can do this easily from the linux terminal:

1) Check which disk is almost full
```
$> sudo df -h
```

2) Change directory to the mount point and start sorting the folders by their size.
```
$> mount (to find out mount point drive)
$> cd / (assuming our drive is mounted at /)
$> du -sh * | sort -h
```
Proceed with changing directory and above command, until you find the files you want to remove.

&nbsp;
### 3d-party product
Ncdu is a disk usage analyzer with an ncurses interface. It is designed to find space hogs on a remote server where you don’t have an entire graphical setup available, but it is a useful tool even on regular desktop systems. Ncdu aims to be fast, simple and easy to use, and should be able to run in any minimal POSIX-like environment with ncurses installed.
More info [https://dev.yorhel.nl/ncdu](https://dev.yorhel.nl/ncdu)

Install it easily:
```
$> sudo apt install ncdu
```

