---
title: "Aruba Vsf Stacking"
date: 2022-07-04T08:48:40+02:00
draft: false
tags:
- Aruba
---

Short notes about creating a VSF (Virtual Switching Framework) stacking with Aruba 2930F switches.
More information can be found in a by Aruba published best practices document, located [here](https://community.arubanetworks.com/HigherLogic/System/DownloadDocumentFile.ashx?DocumentFileKey=fdfe116d-c9bf-4136-9c83-73217f9b2646).

Using VSF we do not need backplane stacking, but instead we can use the switch ports to create the stackingring.

### Create a 2 member stack
Enable VSF on the first switch.
```bash
vsf enable domain X
(automatically reboot switch)
vsf member 1 link 1 1/49-1/50
vsf member 1 link 1 name "I-Link1_1"
vsf member 1 link 2 name "I-Link1_2"
```

(add member 2 in stack)
```bash
vsf member 2 type JL262A
vsf member 2 link 1 2/49-2/50
vsf member 2 link 1 name "I-Link2_1"
vsf member 2 link 2 name "I-Link2_2"
```

Now you'll need to connect the cables that link2 --> link to get the ring.


&nbsp;
### Create 3 or more members stack
Enable VSF on the first switch.
```bash
vsf enable domain X
(automatically reboot switch)
vsf member 1 link 1 1/25
vsf member 1 link 1 name "I-Link1_1"
vsf member 1 link 2 1/26
vsf member 1 link 2 name "I-Link1_2"
```

(add member 2 in stack)
```bash
vsf member 2 type JL255A
vsf member 2 link 1 2/25
vsf member 2 link 1 name "I-Link2_1"
vsf member 2 link 2 2/26
vsf member 2 link 2 name "I-Link2_2"
```

(add member 3 in stack)
```bash
vsf member 3 type JL255A
vsf member 3 link 1 3/25
vsf member 3 link 1 name "I-Link3_1"
vsf member 3 link 2 3/26
vsf member 3 link 2 name "I-Link3_2"
```

(add member 4 in stack)
```bash
vsf member 4 type JL255A
vsf member 4 link 1 4/25
vsf member 4 link 1 name "I-Link4_1"
vsf member 4 link 2 4/26
vsf member 4 link 2 name "I-Link4_2"
```

Now you'll need to connect the cables that link2 --> link to get the ring.

**As from version 16.06 you can stack up to eight 2930F members in one stack.**
