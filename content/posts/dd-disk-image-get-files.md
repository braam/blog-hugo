---
title: "[dd] Disk Image Get Files"
date: 2022-01-28T16:43:20+01:00
draft: false
tags:
- Linux
---

### Create disk image with dd and get files from it
This post describes my expierences while creating an entire image from a disk and how we can recover files from this backup image.

#### Create an entire disk image with dd
```bash
$ dd if=/dev/sdb of=/path/to/backup.img bs=64M conv=sync,noerror status=progress
```
or compress the image (less size):
```bash
$ dd if=/dev/sdb bs=64M conv=sync,noerror status=progress | gzip -c > /path/to/backup.img.gz
```

&nbsp;
#### Mount specific partition
We can recover specific files from the image, without cloning it back to a disk. It's necessary to know on which partition your files are located.
Run **fdisk** -l to see all available partitions:
```bash
$ braam@ntbk-mate-hp:/media/braam/Boot/SVEN$ sudo fdisk -l sven
[sudo] password for braam: 
The backup GPT table is corrupt, but the primary appears OK, so that will be used.
Disk sven: 465.8 GiB, 500108689408 bytes, 976774784 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 9FEB6EC8-FE81-4342-9751-19BE2D92E583
Device     Start       End   Sectors   Size Type
sven1       2048   1230847   1228800   600M Windows recovery environment
sven2    1230848   1845247    614400   300M EFI System
sven3    1845248   2107391    262144   128M Microsoft reserved
sven4    2107392 941957119 939849728 448.2G Microsoft basic data
sven5  941957120 976773119  34816000  16.6G Windows recovery environment
```

We are interested at partition 4, we see that the partition starts at offset: **2107392**
To mount this partition run command:
```bash
$ sudo mount -o offset=$((2107392*512)) /media/braam/Boot/SVEN/sven /mnt/iso/
[sudo] password for braam: 
The disk contains an unclean file system (0, 0).
Metadata kept in Windows cache, refused to mount.
Falling back to read-only mount because the NTFS partition is in an
unsafe state. Please resume and shutdown Windows fully (no hibernation
or fast restarting.)
```
**512** comes frome sector 512 bytes and sven is the image name it's in this example without an extension.
You can see that it's only readonly mounted, but for recovering some files this isn't important.
