---
title: "Unify Openstage WL3 Wifi Phone"
date: 2022-01-21T21:40:43+01:00
draft: false
tags:
- Unify
- Ascom
- Aruba
---

During deployments with WIFI phones, Unify uses the Ascom i62wlan phones with their own branding (Did you know that they run Innovaphone firmware under the hood? ;-). Sometimes we deploy them in a network where the wifi wasn't deployed by us. The result can be very stressy because the finetuning / monitoring has to be done by a 3d party. Lucky for us Ascom provides interoperability reports and configuration steps for major vendors, it's located at:

[https://www.ascom-ws.com/AscomPartnerWeb/en/public-us/interop-i62wlan/](https://www.ascom-ws.com/AscomPartnerWeb/en/public-us/interop-i62wlan/)

### Aruba Access Points issues
We did notice very bad audio quality and drops, even one way audio when connecting the WL3 handset to an Aruba AP 505. I did a lot of traces regarding this issue.. but eventually we did found the solution in Ascom's documentation.
It's available here: [Download manual](/uploads/Aruba_App_Note_Ascom_8-4-0-1_i62_R1.pdf)

* For the 802.11d “Country Information” element to be broadcasted on non DFS channels its necessary to have 802.11k enabled. This is important for regions utilizing "world mode" regulatory domain,
**The default 11k profile enables “Quiet IE” to be broadcasted. This causes the Ascom device to function poorly with frequently disconnects. It is therefore critical to modify the 802.11k profile and disable “Quiet IE”. Refer to configuration settings in this document.**

The important step to notice is that you need to disable **RRM Quiet IE** on the SSID where you connect the WL3 handsets.
In Aruba Central Management you can find this under the WLAN advanced options (all roaming options are disabled in this prtscrn):

![SSID changes](/posts_images/WL3_disable_fast_roaming_SSID_Voice.png)
