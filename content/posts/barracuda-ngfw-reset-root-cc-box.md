---
title: "Barracuda Ngfw Reset Root Cc Box"
date: 2023-09-19T10:37:56+02:00
draft: false
tags:
- Barracuda
---

When you don't know the root password of the Barracuda Cloudgen Firewall and the firewall is connected to a ControlCenter, then you can easily reset the password without loosing any configuration.
Proceed like below:

### 1) Create Cluster Repository
If no cluster repository exists, then you need to create an cluster. Right click on the cluster name and select **Create Repository**.
![Barracuda_Repository_Cluster](/posts_images/barracuda-cc-box-rootpass_01.png)

&nbsp;
### 2) Copy Administrative Settings
The next step would be to copy the existing Administrative Settings to the Cluster Repository. Go to the box and right click on Administrative Settings and chose **Copy To Cluster Repository...**.
Give it a name like "AdminSetNewRootPassword".

&nbsp;
### 3) Edit AdminSetNewRootPassword Repository
Now you can edit the newly created AdminSetNewRootPassword Cluster Repository and fill-in a new root password. By copying the settings from the box first will keep all other settings intact.

![Barracuda_Repository_Cluster](/posts_images/barracuda-cc-box-rootpass_02.png)

&nbsp;
### 4) Copy Administrative Settings from Cluster AdminSetNewRootPassword Repository
Now the final step would be to copy back the settings from the cluster repository to the box. **Right click the Administrative Settings** and choose **lock** first, after that right click again on the Administrative Settings
and choose **Copy From Cluster Repository...** 
The Control Center will now apply the new root password to the box by clicking **Activate**.
