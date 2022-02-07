---
title: "Copy to Smb Share From Terminal"
date: 2022-02-07T14:41:12+01:00
draft: false
tags:
- Linux
---

Sometimes it's faster to copy files to a SMB share from within the terminal instead from using the GUI file manager. If you are running some desktop with shares already mounted by nautilus, caja or any other file manager, you could be using fuse (instead of smbclient).

You can find some mountpoints at:
```bash
ls -l /run/user/$UID/gvfs/
drwx------ 1 braam braam 0 Feb  2 10:04 smb-share:server=hostname,share=documents
```
Yes this is a mountpoint!

```bash
df -h /run/user/$UID/gvfs/*
Filesystem      Size   Used  Avail  Use% Mounted on
gvfsd-fuse      16.2T  3.6T  12.6T   59% /run/user/1000/gvfs
```

And you could use it as a regular filesystem to copy some files:
```bash
cp $HOME/myfile /run/user/$UID/gvfs/smb-share:server=hostname,share=documents/destpath/
```
