---
title: "Linux Brother Scanner"
date: 2022-02-07T14:05:21+01:00
draft: false
tags:
- Linux
- Brother
---

Brother MFC models do have good linux support, you can download drivers from there webpage [https://www.brother.com](https://www.brother.com). Everything went smooth untill I did some changes on my netwerk, I did change the DHCP range and suddenly I was not able to scan pages anymore. I could still print documents, but scanning was failing..

I found out that printing is done using the mDNS name from the printer and scanning is done using IP address.. the IP address is not changing when re-adding the scanner, so we can change it directly in the config file:

```
/etc/opt/brother/scanner/brscan4/brsanenetdevice4.cfg
```
