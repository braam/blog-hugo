---
title: "Speeddials With Pauze"
date: 2022-01-28T16:14:04+01:00
draft: false
tags:
- Unify
- OSBiz
---

To program a "dial pause" and DTMF changeover for suffix dialing of DTMF characters (e.g., for controlling voicemail boxes), you can use the Repdial "P-key" or "#" (pound) key.

The repdial key for the customer must be set as:
```
<dial_number><dtmf_digit><pause_digit><access_code> e.g. 008007728477#P2210344
```
You can add as many P (pauzes) as you wish.
