---
title: "Unify OSBiz sen_r Pass"
date: 2022-02-11T22:18:40+01:00
draft: false
tags:
- Unify
- OSBiz
- Pentest
---

New challenge accepted to retrieve the default sen_r password that Openscape Business systems use to retrieve the new software files from Unify's server. The files are located at:
[https://download.gsi-ms.com/sen_s/openscape_business/v2/osbiz_v2_ocab.tar](https://download.gsi-ms.com/sen_s/openscape_business/v2/osbiz_v2_ocab.tar)

I did use my trick to log every 1 sec. all the running processes to determine which process is responsible for the download.
```bash
while true; do ps auxw >> ps.txt; date >> ps.txt; sleep 1; done
```
I did found following interesting process: **swuObserver** it looks like the swuObserver process logs his action in following location:
```
/mnt/persistent/trace_log/log/swUpgrade/swUpgrade.log
```

We can retrieve the interesting part from the log with this command:
```bash
OSbizS:~ # cat /mnt/persistent/trace_log/log/swUpgrade/swUpgrade.log | grep "Author"
2020/03/09 12:45:13  swuSystem.c:548  pid=47873  libcurl: 003c: Authorization: Basic c2VuX3I6JFNlbkBSMg==
```

As you can see the Authorization Basic header is encoded with Base64, let's decrypt it:
```bash
OSbizS:~ # echo c2VuX3I6JFNlbkBSMg== | base64 -d
sen_r:$Sen@R2
```
sen_r is the username, everything after the semicolon is the password.

We could probably used tcpdump for this aswell:
```bash
OSbizS:~ # tcpdump -i any -w - | strings | grep -i "Authorization: Basic"
```

**Note**: That this will only work with SLES11, when trying this on SLES12 the Authorization Basic header is not used anymore.

**Note**: When colleagues used their own credentials to transfer files from the company download webserver.. those credentials could also be intercepted.
