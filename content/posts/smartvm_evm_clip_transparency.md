---
title: "SmartVM EVM Clip Transparency"
date: 2022-01-28T22:23:27+01:00
draft: false
tags:
- Unify
- OSBiz
- CLIP
---

When using IVR through SmartVM CLIP transparency isn't working out of the box. You can get it working using this config:
Create a virtual station (expert mode only).
Create a call forwarding rule and assign it as internal to this virtual station. Use the targets:

1) \*
2) Target with mobility user / virtual station forwarded through *1 / group with team as member
3) No Entry
4) No Entry
CF starts after: 5 secs.

The call will ring 5 sec. on the virtual station, then proceed to the actual target including the CLIP.
In the IVR, set the target to the virtual station extension.
