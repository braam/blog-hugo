---
title: "Aruba AOS-CX Cheatsheet"
date: 2023-09-06T08:39:19+02:00
draft: false
tags:
- Aruba
---

This cheatsheet is used for remembering commands I do use quite often when configuring Aruba AOS-CX switches. **Please note that you always have to first upgrade to the latest firmware version**, because not all
commands are available in older versions. You can download firmware files through [https://asp.arubanetworks.com/downloads](https://asp.arubanetworks.com/downloads)


### Configuration context
```
conf t
```

### Change the password for user admin
```
user admin password plaintext <password>
```

### Add Extra Administrator user
```
user <username> group administrators password plaintext <password>
 -default groups:
  -administrators   (full)
  -operators        (basic)
  -auditors         (read only)
```

### Change hostname for switch
```
hostname SWITCH01
```

### Changing the console banner (motd)
```
banner motd $
```

### Config DNS servers
```
ip dns server-address 9.9.9.9
ip dns server-address 1.1.1.1
```

### Config NTP servers
```
ntp server be.pool.ntp.org iburst version 4 prefer
ntp server time.google.com 
ntp enable
```

### Upgrade Firmware
```
copy tftp://192.10.12.0/ss.10.00.0002.swi primary   (using tftp)
copy primary usb:/11.10.00.0002.swi                 (using usb drive)
boot system primary
```

### Enable SSH server
```
ssh host-key rsa bits 2048
ssh server vrf default
```

### SNMPv3 configuration
```
snmp-server snmpv3-only
snmp-server vrf default
snmpv3 context BKMmonitor vrf default
snmpv3 security-level auth-privacy
snmpv3 user RWnetworks auth sha auth-pass plaintext XXXXXXXX priv aes priv-pass plaintext XXXXXXXX access-level rw
snmpv3 user RWnetworks context BKMmonitor
```

### Backup Running-Config
```
copy running-config tftp://10.100.0.5/sw_startup.cfg cli vrf mgmt
```

### Factory Reset Switch
```
(when connected to aruba central, first fire this command: aruba-central support-mode enable)
erase all zeroize
```

### Create VLAN
```
vlan x
	name vlan-name
```

### Join VLAN
```
interface 1/1/1-1/1/10 (or lag x)
	vlan access x
	vlan trunk native x
	vlan trunk allowed x,x		(also allow native vlan!!)
```

### Remove VLAN
```
no vlan x
```

### Enable LLDP voice on VLAN
```
vlan x
	voice
```

### Mgmnt IP address
```
interface vlan x
	ip address x.x.x.x/x
	(or interface mgmt is dedicated port, defaults to DHCP)
```

### Add IP Routes
```
ip route 0.0.0.0/0 x.x.x.x
```

### Enter Description On Port
```
interface x
	description UPLINK_FW01-Port-X
```

### Disable/Enable interface
```
interface x
	shutdown
	no shutdown
```

### Create LAG/LACP (best practice to use multiplies from 2, --> 2,4 of 8)
```
interface lag x
int 1/1/1,1/1/2
	lag x
	lacp mode active (optional for lacp protocol, otherwise just a 'bond')
	lacp rate slow
```

### Configure SpanningTree Ports (defaults to MSTP)
For user/workstation/endpoint ports (defaults to admin-network, is good for AP's/uplinks).
```
interface x
	spanning-tree port-type admin-edge 
	spanning-tree bpdu-guard
```

### Configure loop-protect on interfaces (port/lag)
```
interface x
	loop-protect
#Configure loop protect on vlan
loop-protect vlan x
```

### Configure DHCP-snooping
(firewall/pabx/ap's/servers/uplinks trust, all others untrust (default behavior))
```
dhcpv4-snooping
interface x
	dhcpv4-snooping trust
```

### Configure stacking VSF (only Aruba 6200 > and max 8 switches in stack.)
(you can use OOBM ports as failover connection, so the primary and secondary are directly connected to this port, following command is needed: vsf split-detect mgmt)
2-members STACK:
```
-1st switch:
    vsf member 1
    link 1/1/27
-2nd switch (login to this switch first, do not connect to 1st switch at this point):
    vsf member 1
    link 1 1/1/27
    vsf renumber-to 2
    vsf secondary-member 2 (set second switch as secondary/standby)
	(2nd switch will reboot, after that you can make the stacking uplinks to 1st switch)
```
More then 2 members STACK:
```
-1st switch:
    vsf member 1
    link 1 1/1/27
	link 2 1/1/28
-2nd switch (login to this switch first, do not connect to 1st switch at this point):
    vsf member 1
    link 1 1/1/27
	link 2 1/1/28
    vsf renumber-to 2
    vsf secondary-member 2 (set second switch as secondary/standby)
	(2nd switch will reboot, after that you can make the stacking uplinks to 1st switch)
-3d switch (login to this switch first, do not connect to 2nd switch at this point):
    vsf member 1
    link 1 1/1/27
	link 2 1/1/28
    vsf renumber-to 3
    (3d switch will reboot, after that you can make the stacking uplinks to 2nd switch)
```
As always make the ring with link1 <-> link2.