---
title: "Copy From Android With Terminal"
date: 2022-02-07T14:45:51+01:00
draft: false
tags:
- Linux
- Android
---

Go-mtpfs is a simple FUSE-based filesystem written in Go language for mounting Android devices as a MTP device. By using go-mtpfs we can copy from android to our pc by using the terminal.

### Configuration
Appropriate users need to be in the plugdev group:
```bash
sudo gpasswd -a <USER_NAME> plugdev
```

### Usage
**Mount**:
```bash
mkdir /media/braam/MyAndroid
go-mtpfs /media/braam/MyAndroid &
```
_**Note:**_ If go-mtpfs is not ran in the background (with & at the end), another terminal will be needed to browse the device and unmount the device (when finished).

**Unmount**:
```bash
fusermount -u /media/braam/MyAndroid
```
When the device is unmount, go-mtpfs will quit.
