---
title: "QNAP Data Recovery RAID_1 Single Disk"
date: 2022-06-27T08:52:06+02:00
draft: false
tags:
- Linux
- QNAP
- Data-Recovery
---

These are my notes to try to recover data from a RAID-1 disk used in a QNAP NAS, you can follow the whole process here and hopefully also recover your own data. We do have a healthy disk from a RAID-1 setup in our hands, but it's only one disk from two. Please note that when you hear your disk making a "clicking" sound then you can not follow this write-up, because then you'll probably have a hardware failing disk and that's not the purpose for this blogpost.


### 1) Mounting disk in Linux
First step is to attach the disk to a linux device, you can go ahead with any distro you like or try this ubuntu live cd, available from here: [https://ubuntu.com/tutorials/try-ubuntu-before-you-install](https://ubuntu.com/tutorials/try-ubuntu-before-you-install)
QNAP uses mdadm for creating software RAID devices, so let's start by examiming the disk using fdisk:
```bash
$> sudo fdisk -l

Disk /dev/sdb: 2,75 TiB, 3000592982016 bytes, 5860533168 sectors
Disk model: EFRX-68AX9N0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: AEFC8B50-1DBC-46F6-844F-60D7BD2704AB

Dispositivo      Start       Fine    Settori   Size Tipo
/dev/sdb1           40    1060289    1060250 517,7M Microsoft basic data
/dev/sdb2      1060296    2120579    1060284 517,7M Microsoft basic data
/dev/sdb3      2120584 5842744109 5840623526   2,7T Microsoft basic data <<<<<<
/dev/sdb4   5842744112 5843804399    1060288 517,7M Microsoft basic data
/dev/sdb5   5843804408 5860511999   16707592     8G Microsoft basic data
```

We are interested in the biggest partition. This partition is holding our data. (/dev/sdb3)

Next, let's check if the RAID metadata is still intact:
```bash
sudo mdadm -E /dev/sdb3

/dev/sdb3:
          Magic : a92b4efc
        Version : 1.0
    Feature Map : 0x0
     Array UUID : 77976160:8f893135:eb6cdd71:733bb188
           Name : 1
  Creation Time : Mon Jun 22 01:07:33 2020
     Raid Level : raid1
   Raid Devices : 2

 Avail Dev Size : 5840623240 (2785.03 GiB 2990.40 GB)
     Array Size : 2920311616 (2785.03 GiB 2990.40 GB)
  Used Dev Size : 5840623232 (2785.03 GiB 2990.40 GB)
   Super Offset : 5840623504 sectors
   Unused Space : before=0 sectors, after=264 sectors
          State : clean
    Device UUID : a997883d:504f6cc2:bd13a2e1:3c1e2e92

    Update Time : Mon Aug 24 12:11:56 2020
  Bad Block Log : 512 entries available at offset -8 sectors
       Checksum : 7fb23406 - correct
         Events : 708


   Device Role : Active device 1
   Array State : .A ('A' == active, '.' == missing, 'R' == replacing)
```

We notice that the RAID-1 is still intact and was like always build with two devices.

If you do have mdadm already installed then probably your RAID device is already discovered and created, let's check this with the following command:
```bash
$> cat /proc/mdstat

Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md127 : active raid1 sdb3[1] <<<<<<
      2920311616 blocks super 1.0 [2/1] [_U]
      
md322 : active (auto-read-only) raid1 sdb5[1]
      7235136 blocks super 1.0 [2/1] [_U]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md256 : active (auto-read-only) raid1 sdb2[1]
      530112 blocks super 1.0 [2/1] [_U]
      bitmap: 0/1 pages [0KB], 65536KB chunk

md9 : active (auto-read-only) raid1 sdb1[1]
      530048 blocks super 1.0 [64/1] [_U]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md13 : active (auto-read-only) raid1 sdb4[1]
      458880 blocks super 1.0 [64/1] [_U]
      bitmap: 1/1 pages [4KB], 65536KB chunk

unused devices: <none>
```

If your RAID device is not started already, then we can try to enable it with the following command. 

Note: Choose a free number for the RAID the device (checking output in above command to determine free number).

```bash
$> sudo mdadm -A -R /dev/md127 /dev/sdb3
mdadm: /dev/md127 has been started with 1 drive.
```

Now that the RAID device is enabled, let's verify LVM2 and grab volume/logical drive:
```bash
$> sudo pvscan 
  PV /dev/md127   VG vg288           lvm2 [<2,72 TiB / 0    free]
  Total: 1 [<2,72 TiB] / in use: 1 [<2,72 TiB] / in no VG: 0 [0   ]

$> sudo vgdisplay 
  --- Volume group ---
  VG Name               vg288
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  21
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <2,72 TiB
  PE Size               4,00 MiB
  Total PE              712966
  Alloc PE / Size       712966 / <2,72 TiB
  Free  PE / Size       0 / 0
  VG UUID               E9eFWd-WRmR-C8W6-YSu4-tSgA-1WnT-P4eXqX
   
$> sudo pvdisplay 
  --- Physical volume ---
  PV Name               /dev/md127
  VG Name               vg288
  PV Size               <2,72 TiB / not usable <1,78 MiB
  Allocatable           yes (but full)
  PE Size               4,00 MiB
  Total PE              712966
  Free PE               0
  Allocated PE          712966
  PV UUID               lkTOn4-W5Ni-kjaJ-5ok0-DjbA-h5cc-1xtN0J

$> sudo lvdisplay 
  --- Logical volume ---
  LV Path                /dev/vg288/lv544
  LV Name                lv544
  VG Name                vg288
  LV UUID                4KMo6U-AfNV-amSL-Th2U-UZoZ-qKQW-n0EtKz
  LV Write Access        read/write
  LV Creation host, time Qnap, 2020-06-22 01:07:36 +0200
  LV Status              NOT available
  LV Size                <27,90 GiB
  Current LE             7142
  Segments               2
  Allocation             inherit
  Read ahead sectors     8192
   
  --- Logical volume ---
  LV Path                /dev/vg288/lv1
  LV Name                lv1
  VG Name                vg288
  LV UUID                CYGw0J-01iU-e9vv-GxZe-nWg9-kPL4-DOn3mP
  LV Write Access        read/write
  LV Creation host, time Qnap, 2020-06-22 01:07:47 +0200
  LV Status              NOT available
  LV Size                2,69 TiB
  Current LE             705824
  Segments               1
  Allocation             inherit
  Read ahead sectors     8192
```

Note:  If, after the command pvscan appear the following error 'WARNING: PV /dev/md127 in VG vg288 is using an old PV header, modify the VG to update' do the following command :
```bash
$> sudo vgck --updatemetadata vg288
```
We are ready now, to activate the VolumeGroup:
```bash
$> sudo vgchange -ay vg288
```
**HERE WE DID ENCOUNTER FOLLOWING ERROR:**
```bash
WARNING: Unrecognised segment type tier-thin-pool
WARNING: Unrecognised segment type thick
WARNING: Unrecognised segment type flashcache
LV tp1, segment 1 invalid: does not support flag ERROR_WHEN_FULL. for tier-thin-pool segment.
```
> This error is due the fact that QNAP users MD for RAID, and then uses a segment type, on LVM2, named tier_thin_pool. On top of all that, you'll find an ext4 filesystem. This segment type was created by QNAP and never released to the public as the license GPLv2 demands. Hopefully they'll release it in the future.

So what can we do next?
- Place your disk inside a working QNAP unit and mount the drive there.
- Run a QNAP Virtual Machine and mount the disk there.

I went for the VM approach as I did not want to use my current working NAS.


&nbsp;
### 2) Running QNAP VM
I went googling for this and found following interesting article: [https://xpenology.com/forum/topic/27543-qnap-auf-eigener-hardware/](https://xpenology.com/forum/topic/27543-qnap-auf-eigener-hardware/)

I'll not cover the setup details for this VM, you can follow this excellent youtube video: [https://youtu.be/ZdsK5f1qSDw](https://youtu.be/ZdsK5f1qSDw)

After this you can attach your disk through USB to the QNAP VM and boot it up (probably run vmwareplayer as root for this).

For easier acces, enable SSH access on the QNAP by login in to the WBM portal and go to Control Panel --> Telenet/SSH --> enable.

Let's mount the RAID device now in QNAP:
```bash
[#] fdisk -l
--> /dev/sdb3 partition is our Data partition.
[#] mdadm -A -R /dev/md11 /dev/sdb3
--> started with 1 drive
[#] cat /proc/mdstat
--> md11 started
[#] pvscan
--> ERROR: No matching physical volumes found.
// Lets fix this by re-enabling the cache to determine new volumgroups
[#] pvscan --cache
[#] vgdisplay --cache
  --- Volume group ---
  VG Name               vg1
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  278
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               2.72 TiB
  PE Size               4.00 MiB
  Total PE              712966
  Alloc PE / Size       712966 / 2.72 TiB
  Free  PE / Size       0 / 0
  VG UUID               fQPPkx-x13R-6YOq-l4u4-oUMU-d2T0-TMcLBF
[#] vgchange -ay vg1
--> ERROR:
    sbin/pdata_tools_8192 failed: 1
    thin_check with block_size = 8192 failed
    /sbin/pdata_tools_4096 failed: 1
    thin_check with block_size = 4096 failed
      Check of pool vg1/tp1 failed (status:1). Manual repair required!
    /sbin/pdata_tools_8192 failed: 1
      thin_check with block_size = 8192 failed
    /sbin/pdata_tools_4096 failed: 1
    thin_check with block_size = 4096 failed
      1 logical volume(s) in volume group "vg1" now active
// Lets fix this with a dirty workaround, to disbale thin checking completely.
[#] vi /etc/lvm/lvm.conf
    -> Find line with: thin_check_executable = "/sbin/thin_check"   <-- remove the executable, so line will be = ""
[#] vgchange -ay vg1
--> all volumes started
[#] lvdisplay 
  --- Logical volume ---
  LV Path                /dev/vg1/lv544
  LV Name                lv544
  VG Name                vg1
  LV UUID                mECHNc-e2a4-ROkf-X3Az-eqHR-gWCr-Kiv3xg
  LV Write Access        read/write
  LV Creation host, time Nas-jpel, 2017-10-06 14:25:57 +0200
  LV Status              available
  # open                 0
  LV Size                27.85 GiB
  Current LE             7129
  Segments               1
  Allocation             inherit
  Read ahead sectors     8192
  Block device           253:0
   
  --- Logical volume ---
  LV Name                tp1
  VG Name                vg1
  LV UUID                eSiAR1-2ioE-p2ay-Gxas-eOCD-3Etl-8l5Djv
  LV Write Access        read/write
  LV Creation host, time Nas-jpel, 2017-10-06 14:25:58 +0200
  LV Pool metadata       tp1_tmeta
  LV Pool data           tp1_tierdata_0
  LV Status              available
  # open                 2
  LV Size                2.63 TiB
  Allocated pool data    64.74%
  Allocated pool chunks  3568946
  Allocated metadata     0.15%
  Current LE             689053
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:5
   
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                jNsi9d-vNNp-vOB0-d3mW-chj5-pBc8-1efhZF
  LV Write Access        read/write
  LV Creation host, time Nas-jpel, 2017-10-06 14:26:15 +0200
  LV Pool name           tp1
  LV Status              available
  # open                 1
  LV Size                2.63 TiB
  Mapped size            64.77%
  Mapped sectors         3654600704
  Current LE             688768
  Segments               1
  Allocation             inherit
  Read ahead sectors     8192
  Block device           253:7

//We need /dev/vg1/lv1 volume, lets check what filetype this is:
[#] blkid /dev/vg1/lv1
  /dev/vg1/lv1: LABEL="DataVol1" UUID="126d3858-bd8e-4c9c-807b-a0c128e7b653" TYPE="ext4" 
//Finally mount it
[#] mkdir /mnt/recovered
[#] mount -t ext4 /dev/vg1/lv1 /mnt/recovered
```

Now we can copy all files with scp to a fresh new harddrive.

Optional you can afterwards format the NAS disk, by deleting all partitions:
```bash
$> sudo fdisk /dev/sdb
fdisk: d  (delete all partitions)
fdisk: p  (verify no partitions exists anymore)
fdisk: w  (save changes to disk an quit)
```

You can now format the disk and copy back all the data you recovered with scp or use the disk for something else.






