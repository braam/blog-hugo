---
title: "Compiling on Raspberrypi Memory Issue"
date: 2022-02-07T13:35:29+01:00
draft: false
tags:
- Raspberry-Pi
- Linux
---

When compiling on the raspberrypi I came along the following error, the compiling job did not end succesfully:

```bash
g++: fatal error: Killed signal terminated program cc1plus
```

I did a quick google search and it was quite obvious that this was concering to memory, the problem occurs when your system runs out of memory. In this case rather than the whole system falling over, the operating systems runs a process to score each process on the system. The one that scores the highest gets killed by the operating system to free up memory. If the process that is killed is cc1plus, gcc (perhaps incorrectly) interprets this as the process crashing and hence assumes that it must be a compiler bug. But it isn't really, the problem is the OS killed cc1plus, rather than it crashed.

We can confirm this by running dmesg command:
```bash
[11388.072160] Out of memory: Killed process 1075 (cc1plus) total-vm:241960kB, anon-rss:120700kB, file-rss:0kB, 
shmem-rss:0kB, UID:0 pgtables:242kB oom_score_adj:0
```

How can we fix this? As we cannot add more RAM to the rpi?? We could create a bigger SWAP partition are even better we can create a _**swapfile**_, so we do not have to alter the partition table!

&nbsp;
### Create swapfile
Create an empty file (1K * 4M = 4 GiB), this could take some time..
```bash
sudo mkdir -v /var/cache/swap
cd /var/cache/swap
sudo dd if=/dev/zero of=swapfile bs=1K count=4M
sudo chmod 600 swapfile
```

Convert newly created file into a swap space file.
```bash
sudo mkswap swapfile
```

Enable file for paging and swapping.
```bash
sudo swapon swapfile
```

Verify by: swapon -s or top:
```bash
top -bn1 | grep -i swap
```
Should display line like: KiB Swap:  4194300 total,  4194300 free

To disable, use command:
```bash
sudo swapoff /var/cache/swap/swapfile
```

&nbsp;
### Optional - Mount swapfile on startup
Add it into fstab file to make it persistent on the next system boot.
```bash
echo "/var/cache/swap/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab
```

Re-test swap file on startup by:
```bash
sudo swapoff swapfile
sudo swapon -va
```
_**Note:**_ Above commands re-checks the syntax of fstab file, otherwise your Linux could not boot up properly.
