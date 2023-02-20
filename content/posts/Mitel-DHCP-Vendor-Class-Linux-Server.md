---
title: "Mitel DHCP Vendor Class Linux Server"
date: 2022-07-15T15:26:12+02:00
draft: false
tags:
- Mitel
- DHCP
- Linux
---

Short notes to remember on how to configure Vendor Classes on a linux DHCP server. I did use **isc-dhcp-server** for my test, **dhcpd.conf** file may contain following code:
```bash
option space AastraIPPhone;
option AastraIPPhone.ActivateVLANHeader code 08 = text;
option AastraIPPhone.VLAN-ID code 09 = unsigned integer 32;

class "AastraSIP" {
 match if ( (option vendor-class-identifier="AastraIPPhone6863i")
 or (option vendor-class-identifier="AastraIPPhone6865i")
 or (option vendor-class-identifier="AastraIPPhone6867i")
 or (option vendor-class-identifier="AastraIPPhone6869i")
 or (option vendor-class-identifier="AastraIPPhone6873i")
 or (option vendor-class-identifier="AastraIPPhone6920") );
 vendor-option-space AastraIPPhone;
 option AastraIPPhone.ActivateVLANHeader "Aastra Telecom  ";
 option AastraIPPhone.VLAN-ID 21;
}
```

Note that ulinke Windows in Linux we can enter everything as a normal string or decimal number (VLAN-ID) instead of converting them to Hex.

### Using a Wildcard (*) VCI Match
We can use a Wildcard VCI to match all Aastra/Mitel phones without the need to specify each type:
```bash
option space AastraIPPhone;
option AastraIPPhone.ActivateVLANHeader code 08 = text;
option AastraIPPhone.VLAN-ID code 09 = unsigned integer 32

class "AastraSIP" {
 match if substring (option vendor-class-identifier, 0, 13) = "AastraIPPhone";
 vendor-option-space AastraIPPhone;
 option AastraIPPhone.ActivateVLANHeader "Aastra Telecom  ";
 option AastraIPPhone.VLAN-ID 21;
}
```
