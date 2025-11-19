---
title: "Barracuda Phibstest"
date: 2023-01-31T14:23:26+01:00
draft: false
tags:
 - Barracuda
---

When using different authenticationschemes within the Barracuda NGFirewall we can troubleshoot/test these authentication methods using the **phibstest tool**.

### As an example, testing MSAD
Make sure the NGF DNS server is pointed towards the DC.
```bash
>$ phibstest i authscheme=msad user=test.gebruiker
type=userinfo sub=-1 id=2 ver=1 res=Success timeout=5: Got User Info
groups = CN=VPN-Gebruikers,OU=SCE,DC=SCE,DC=local
metadirattr = 
user = test.gebruiker
[2023-01-31 14:22 CET] [-standard-] [-Barracuda Networks-]
```

You can easily see the groups to which an user belongs to, be aware that when configuring the MSAD authentication scheme that you do not filter out some specific groups or you'll end up with some empty results.


