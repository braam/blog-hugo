---
title: "Wifi Internet Sharing Over Ethernet"
date: 2022-06-23T12:34:03+02:00
draft: false
tags:
- Linux
---
I needed to be able to have internet on a PC with ethernet only, the problem was the switch was way to far and I didn't have a long UTP cable. So I was looking for a way to share the laptop WIFI connection over the laptop ethernet port to the second PC. 

This can also be handy to give a firewall temporary internet through DHCP if no internet is available. The setup would become so, that your mobile will do tethering and you connect your laptop to that hotspot.

&nbsp;
### Setup ethernet bridge
Edit the **network connections** on your Linux pc and double click the **Wired Connection**. 
Click on **IPV4 Settings** and change the Method to **Share to other computers**.

![network connections](/posts_images/wifi-ethernet-sharing-01.png)

Now just connect the 2 computers and you are done.

**NOTE**: If both computers have a Wired Gigabit Ethernet NIC then you can actually connect them directly since most 1000BASE-T have automatic MDI/MDI-X. If not just connect them to your switch, router, hub, whatever and that is all.

&nbsp;
### Connect Client PC
You can enable static IP configuration or go ahead with the defaults, when Linux Mint is sharing the network adapter it automatically enables a DHCP server. So make sure the client pc is using DHCP for network connection.
