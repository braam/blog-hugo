---
title: "Fasttel Doorphone Tricks"
date: 2022-01-28T16:27:43+01:00
draft: false
tags:
- Fasttel
---

### Fasttel Doorphone Doorcontact
Sometimes you've to connect a Fasttel doorphone to an existing doorcontact. The problem is that the doorcontact has no external power, so what can you do? I figured out that you can use the power connected to the doorphone to feed the doorcontact. Please be aware that not all doorcontacts will work by this trick, also PoE powered doorphones will no work because you can't split the power. *(more over this below)*

First check that the doorcontact works by using an simple 9V battery connected to both wires. You can then connect the cables like this (PIN 11+12 is a NO contact):

![fasttel doorcontact power bridge](/posts_images/fasttel_doorphone_doorcontact_power.jpeg)


### Fasttel IP Doorphone (PoE)
A Fasttel IP doorphone can use PoE as the main power, but sometimes we see that there is only one UTP cable available at the door. So we cannot feed the doorcontact with this, but wait.. we can connect the UTP as 100Mbps instead of 1Gbps. This will save us two pairs as for 100Mbps only 2 pairs is needed, PoE is still working with those two pairs. The other two pairs can be used to trigger the doorcontact.

![100Mbps UTP pinout](/posts_images/100Mbps_utp.png)
