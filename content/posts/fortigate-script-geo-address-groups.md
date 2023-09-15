---
title: "Fortigate Script Geo Address Groups"
date: 2023-09-15T10:45:09+02:00
draft: false
tags:
- Fortigate
---

When configuring new object addresses/groups through the WEB UI it could become timeconsuming. That's why I did paste here the CLI commands I do often use to enable GEO blocking with the firewall. These addresses/groups 
can then be used within policies/VPN service to whitelist countries to be able to establish a SSL-VPN connection.

This is currently only for Europe regions and will looks like the following image:
![Fortigate-addresses-GEO](/posts_images/fortigate_geolocation01.png)
![Fortigate-groups-GEO](/posts_images/fortigate_geolocation02.png)

```
config firewall address
    edit "Geolocation_Belgium"
        set type geography
        set country "BE"
    next
    edit "Geolocation_Netherlands"
        set type geography
        set country "NL"
    next
    edit "Geolocation_Germany"
        set type geography
        set country "DE"
    next
    edit "Geolocation_Denmark"
        set type geography
        set country "DK"
    next
    edit "Geolocation_Sweden"
        set type geography
        set country "SE"
    next
    edit "Geolocation_Finland"
        set type geography
        set country "FI"
    next
    edit "Geolocation_Ireland"
        set type geography
        set country "IE"
    next
    edit "Geolocation_UK"
        set type geography
        set country "GB"
    next
    edit "Geolocation_Luxembourg"
        set type geography
        set country "LU"
    next
    edit "Geolocation_France"
        set type geography
        set country "FR"
    next
    edit "Geolocation_Austria"
        set type geography
        set country "AT"
    next
    edit "Geolocation_Slovenia"
        set type geography
        set country "SI"
    next
    edit "Geolocation_Lithuania"
        set type geography
        set country "LT"
    next
    edit "Geolocation_Estonia"
        set type geography
        set country "EE"
    next
    edit "Geolocation_Latvia"
        set type geography
        set country "LV"
    next
    edit "Geolocation_Poland"
        set type geography
        set country "PL"
    next
    edit "Geolocation_Czech-Republic"
        set type geography
        set country "CZ"
    next
    edit "Geolocation_Slovakia"
        set type geography
        set country "SK"
    next
    edit "Geolocation_Hungary"
        set type geography
        set country "HU"
    next
    edit "Geolocation_Romania"
        set type geography
        set country "RO"
    next
    edit "Geolocation_Bulgaria"
        set type geography
        set country "BG"
    next
    edit "Geolocation_Greece"
        set type geography
        set country "GR"
    next
    edit "Geolocation_Italy"
        set type geography
        set country "IT"
    next 
    edit "Geolocation_Spain"
        set type geography
        set country "ES"
    next
    edit "Geolocation_Portugal"
        set type geography
        set country "PT"
    next
    edit "Geolocation_Cyprus"
        set type geography
        set country "CY"
    next
    edit "Geolocation_Malta"
        set type geography
        set country "MT"
    next
    end
config firewall addrgrp
   edit "Geolocation_Nord-Europe"
        set member "Geolocation_Denmark" "Geolocation_Finland" "Geolocation_Sweden"
    next
    edit "Geolocation_West-Europe"
        set member "Geolocation_Austria" "Geolocation_Belgium" "Geolocation_France" "Geolocation_Germany" "Geolocation_Ireland" "Geolocation_Luxembourg" "Geolocation_Netherlands" "Geolocation_Slovenia" "Geolocation_UK"
    next
    edit "Geolocation_East-Europe"
        set member "Geolocation_Bulgaria" "Geolocation_Czech-Republic" "Geolocation_Estonia" "Geolocation_Hungary" "Geolocation_Latvia" "Geolocation_Lithuania" "Geolocation_Poland" "Geolocation_Romania" "Geolocation_Slovakia"
    next
    edit "Geolocation_South-Europe"
        set member "Geolocation_Cyprus" "Geolocation_Greece" "Geolocation_Italy" "Geolocation_Malta" "Geolocation_Portugal" "Geolocation_Spain"
    next
    end
```
