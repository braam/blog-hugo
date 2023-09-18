---
title: "Alpha+manual Tricks"
date: 2023-09-18T14:48:29+02:00
draft: false
tags:
- Alpha+
---

Configuring the Alpha+ module can be challenging espacilly when using the DTMF codes instead of the programmer software. I did had a case when a button was connected to the Alpha+ module, when someone pressed the button a call should start.
But the important thing was that the call should not be longer then 20seconds. We tried of course first to program this behaviour in the PABX itself but did didn't work. So we changed a setting in the Alpha+ box.
The customer didn't want to answer the call, but just had to be notified when someone is at the door.

When looking in the manual, we can use topic 4 and 5.
![Alpha+_config_01](/posts_images/alpha+_config_01.png)

It's important to note that but codes 6 & 7 are needed, if you configure only code 6 it won't work!

So for 20seconds the code becomes:
```
<6 00 02>
<7 00 02>
```

The full manual is available here: [Download manual](/uploads/alpha+NL.pdf)





