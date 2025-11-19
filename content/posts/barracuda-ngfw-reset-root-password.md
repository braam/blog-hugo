---
title: "Barracuda Ngfw Reset Root Password"
date: 2024-04-15T09:49:14+02:00
draft: false
tags:
- Barracuda
---

When you don't know the root password of the Barracuda Cloudgen Firewall we can reset the root password without losing it active configuration. We have following scenarios:

## 1) Firewall connected to Control-Center
## 2) Low-privilege user access
## 3) Standalone firewall physical access

___

&nbsp;
### 1) Firewall connected to Control-Center
We will overwrite the current unknown root password with a known password, by using a cluster repository.
If no cluster repository exists, then you need to create an cluster. Right click on the cluster name and select **Create Repository**.
![Barracuda_Repository_Cluster](/posts_images/barracuda-cc-box-rootpass_01.png)

### * Copy Administrative Settings
The next step would be to copy the existing Administrative Settings to the Cluster Repository. Go to the box and right click on Administrative Settings and chose **Copy To Cluster Repository...**.
Give it a name like "AdminSetNewRootPassword".

### * Edit AdminSetNewRootPassword Repository
Now you can edit the newly created AdminSetNewRootPassword Cluster Repository and fill-in a new root password. By copying the settings from the box first will keep all other settings intact.

![Barracuda_Repository_Cluster](/posts_images/barracuda-cc-box-rootpass_02.png)

### * Copy Administrative Settings from Cluster AdminSetNewRootPassword Repository
Now the final step would be to copy back the settings from the cluster repository to the box. **Right click the Administrative Settings** and choose **lock** first, after that right click again on the Administrative Settings
and choose **Copy From Cluster Repository...** 
The Control Center will now apply the new root password to the box by clicking **Activate**.

&nbsp;
&nbsp;
### 2) Low-privilege user access
If we only have low-privilege user access, this does mean we don't know the current root password. Then we can use a trick, to reset the current root password. We'll be using a cronjob created with the Firewall Admin tool.

Tasks added with the System Scheduler will be run as root.. so following command will change the password for the root account with the password we provide.
```
echo 'root:ngf1r3wall' | sudo chpasswd
```

Navigate to **CONFIGURATION** > **Configuration Tree** > **Box** > **Advanced Configuration** > **System Scheduler**
![Barracuda_System-Scheduler](/posts_images/barracuda-system-scheduler.png)

Go to the **Generic Schedule** and copy/paste following snippet:

```
CONFDEF box/boxother/boxcron partial 8.3

[job_reset-root-passw]
GDESC = 
DESCRIPTION = Resets the root password to the factory default password.
COMMANDS[0] = echo 'root:ngf1r3wall' | sudo chpasswd
COMMANDS[1] = /opt/phion/bin/hwtool -a 1
MINUTES = every
MINLIST = 0
MPERIOD = 5
HOURS = list
HOURLIST = 0
HPERIOD = 12
MONTHDAY = list
MDLIST = 14
MDPERIOD = 14
WEEKDAY = list
WDLIST = 6
WDPERIOD = 2
MONTH = list
MONTHLIST = 6
MONTHPERIOD = 6
```

![Barracuda_Generic_Schedule](/posts_images/barracuda-reset-root-system-scheduler.png)

If you can see the box physicaly then you'll hear ring the bell with the second command. After that you can remove the scheduled task again. Otherwise remotely just wait 5min. before removing the job.

&nbsp;
&nbsp;
### 3) Standalone firewall physical access
If you do have physical access towards the box, you'll need a console cable or a monitor with keyboard to be able to reset the root password. We'll connect with a console cable with **baud rate 19200**.
We'll abusing grub.conf to force a root shell, and then we'll chroot into the barracuda system.

Connect the console cable and reboot the firewall, when you do see the grub entries, press the **e** button on your keyboard.
The grub password is: **ph10n** (We found this password in the firmware / working box in the location: /opt/phion/config/active/bootloader.conf)

In the grub config go to the Linux line and at the end (ctrl+e) add **rd.break**
Proceed with ctrl+x and save the changes.

Now you'll get a root shell, in this root shell proceed with following commands:
```
switch_root:/# mount -o remount,rw /sysroot
switch_root:/# chroot /sysroot
switch_root:/# echo 'root:ngf1r3wall' | sudo chpasswd
switch_root:/# exit
switch_root:/# exit
```
