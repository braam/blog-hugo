---
title: "Connecting a Broadcasting System"
date: 2022-01-28T16:00:34+01:00
draft: false
tags:
- Unify
- OSBiz
---

My Customer wanted to connect an broadcasting system to the PABX, the following we need:
* STRB: the amplifier switch needed to be closed otherwise the input won't triggered.
* Analog position in PABX.

&nbsp;
### Setup analog port in PABX
You need to setup an normal SLAV port in the PABX, through this port the speech will blown in the aplifiers input. It's important to set the port type to **Loudspeaker**.

![analog port](/posts_images/amplifier_broadcast_analogport.png)

&nbsp;
### Connecting the STRB
Actually at the customer it's an STRBR so we can just terminate an RJ45 connecter in the middle (PIN 4 and 5). This will be an NO contact. The function of the actuator needs to be changed to **"Amplifier Speaker"**. It's important to assing the chosen analog port as "Assigned Station" Also change the "Switching Time" to **1 * 100 ms**. This will interrupt the NO contact immediate after you finished the broadcast by hanging up the phone.
***Hint**: This setting can only applied by Hipath Manager E.*

![actuators](/posts_images/amplifier_broadcast_strb.png)

&nbsp;
### Amplifier input
Connect the pair for NO contact correctly as the diagram shows, also connect the analog position to + and - screw block, the polarity doesn't matter. On most amplifiers you'll need to set the switch for input to **MIC** instead of **Line**. If the signal is to silent you can set the extra switch to level up the signal by +18V.

![actuators](/posts_images/amplifier_broadcast_amp.jpg)
