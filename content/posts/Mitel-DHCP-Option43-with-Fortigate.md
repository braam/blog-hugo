---
title: "Mitel DHCP Option43 With Fortigate"
date: 2022-07-15T14:20:27+02:00
draft: false
tags:
- Mitel
- Fortigate
- DHCP
---

I was assigned the task to figure out howto assign the VLAN ID to Mitel/Aastra phones on a Fortigate with Option43.
Here is my write-up, I did following pdf as reference: [https://www.telefonanlage-shop.de/downloads/Mitel_6800_6900_SIP_Phone_Admin_Guide_5.0_SP2.pdf](https://www.telefonanlage-shop.de/downloads/Mitel_6800_6900_SIP_Phone_Admin_Guide_5.0_SP2.pdf)

![Option43-String](/posts_images/mitel-fortigate-option43_01.png)

### Apply string on Fortigate
You can now apply the option43 in the Fortigate DHCP server:
![Option43-String](/posts_images/mitel-fortigate-option43_02.png)

I did notice when the phone reboots, the VLAN is still remembered, so probably you can overwrite it with another VLAN when you change the Option43 string or you'll need to factory reset the Phone when you want to remove the VLAN tag.
